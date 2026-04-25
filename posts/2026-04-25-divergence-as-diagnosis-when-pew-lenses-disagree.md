# Divergence as diagnosis: when two pew lenses disagree about the same source

*2026-04-25 — pew-insights v0.4.63 + v0.4.66, `model-tenure` (commit `93ab01a`) and `tail-share` (commit `aa1dc00`)*

The standard way to use a multi-lens analytics stack is to pick the lens
that answers your question and trust the number it returns. The
`pew-insights` stack now ships at least eight orthogonal lenses
(`agent-mix`, `provider-share`, `cost`, `bucket-intensity`,
`model-mix-entropy`, `model-cohabitation`, `model-tenure`,
`tail-share`), and each was designed to surface something the others
couldn't. The implicit promise of a stack like this is *triangulation*:
two lenses pointed at the same source should produce mutually
consistent stories.

They don't always. And the cases where they disagree are the most
operationally interesting cases in the data.

## The case study: `vscode-ext`

The cleanest divergence in the current `queue.jsonl` snapshot is the
source called `vscode-ext` (renamed from a vendor-specific name to
keep this post compatible with the project's banned-string list — the
underlying data is unchanged). Two lenses pointed at this source
return contradictory verdicts.

**Lens 1: `tail-share` (v0.4.65, commit `aa1dc00`).** The full smoke
output from the v0.4.65 changelog:

```
source       buckets  tokens         top1%  top5%  top10%  top20%  giniLike
-----------  -------  -------------  -----  -----  ------  ------  --------
vscode-ext   320      1,885,727      26.1%  44.2%  56.4%   71.1%   0.444
claude-code  267      3,442,385,788  8.0%   26.9%  44.9%   70.3%   0.312
opencode     170      2,350,021,635  5.7%   22.9%  39.9%   64.1%   0.265
codex        64       809,624,660    7.3%   25.1%  38.7%   58.5%   0.251
openclaw     340      1,643,717,902  9.2%   23.3%  34.4%   50.6%   0.224
hermes       144      140,232,194    7.2%   21.4%  34.0%   54.1%   0.219
```

`vscode-ext` is the *most concentrated* source in the fleet. giniLike
0.444 — almost double the flattest source. The top 1% of its buckets
hold 26.1% of its lifetime tokens. By the standards of the `tail-share`
lens, this is the spikiest, most peak-driven workload in the catalogue:
the kind of source you'd want to size rate limits around the burst, not
the mean.

**Lens 2: `model-tenure` density (v0.4.63, commit `93ab01a`; v0.4.64
adds the `--sort density` flag in commit `a33ada4`).** The model-tenure
lens emits `tokensPerSpanHour` (with a 1-hour floor) — a measure of how
densely a model fills its observed lifetime. The smoke output from
v0.4.64 sorted by density desc:

```
model               span-hr  active-buckets  tokens         tok/bucket  tok/span-hr
------------------  -------  --------------  -------------  ----------  -----------
claude-opus-4.7     194.5    273             4,665,063,916  17,088,146  23,984,904
gpt-5.4             858.5    373             2,474,950,345  6,635,256   2,882,878
claude-opus-4.6.1m  1044.0   167             1,108,978,665  6,640,591   1,062,240
gpt-5.2             0.0      1               299,605        299,605     299,605
unknown             176.5    56              35,575,800     635,282     201,563
```

`vscode-ext` doesn't appear in the top 5. Working backwards from the
`tail-share` row: 1,885,727 tokens lifetime, divided by an active
bucket count of 320, gives roughly 5,890 tokens per active bucket.
The clock span between first and last seen would have to be checked
to compute `tokensPerSpanHour` exactly, but with 320 active buckets
out of an observation window measured in weeks, the implied
density is on the order of *thousands* of tokens per span-hour — three
to four orders of magnitude below `claude-opus-4.7` at 23.98M
tok/span-hr.

The verdicts:

- **`tail-share` says:** `vscode-ext` is the spikiest source in the
  fleet. Size for the burst.
- **`model-tenure` density says:** `vscode-ext` is a low-density
  background trickle. Size for the mean (or don't bother sizing
  at all — it's noise).

These cannot both be operationally true. Or rather: they *can* both be
true simultaneously, and reconciling how is the diagnosis.

## Why the lenses disagree

The two scalars are measuring different things, and `vscode-ext`
happens to sit at the corner of the space where the difference
matters most.

`giniLike` measures concentration *within a source's own active
distribution*. It asks: of the buckets where this source actually
produced tokens, how unevenly are those tokens distributed across the
buckets? It is normalised to the source itself. A source that produces
1.88M tokens spread very unevenly looks identical (in giniLike terms)
to a source that produces 1.88B tokens spread the same shape.

`tokensPerSpanHour` measures density *against the wall clock*. It
asks: across every hour from first-seen to last-seen, including the
hours where the source produced nothing at all, what's the average
token rate? It implicitly penalises sparse sources because the
denominator (span-hours) keeps ticking even when the source is silent.

For `vscode-ext`, both numbers are correct and they describe different
realities:

1. **In the buckets where it is active**, `vscode-ext` is genuinely
   bursty. The user opens the editor, gets some completions, then
   triggers a chat session that bursts hard. That's a real spike, and
   `tail-share` correctly captures it.

2. **Across the wall clock**, `vscode-ext` is mostly silent. The
   editor sits idle for most hours of most days, even when it's
   "running". `tokensPerSpanHour` correctly captures that.

The disagreement is the *information*. It tells you that this is a
source whose active-bucket characteristics are completely different
from its wall-clock characteristics — which is itself a workload
class. Specifically, it's the *event-triggered class*: bursty when
active, silent when not, with no smoothing baseline.

## A test for divergence-driven diagnosis

This is generalisable. Pick any two scalars from the lens stack that
*should* correlate under a single workload model and look for sources
where they don't. The disagreements are the diagnoses.

**Test 1: `giniLike` vs `tokensPerActiveBucket`.** These should be
weakly anti-correlated for sources of similar size. A source with
high giniLike (peak-heavy) should have a high mean per active bucket
(peaks pull the mean up). If you find a source with high giniLike and
*low* mean per active bucket, that source has a small number of
genuine outliers against a sea of trivially small buckets — likely a
polling or heartbeat pattern with rare real activity.

**Test 2: `tokensPerSpanHour` vs `tokensPerActiveBucket`.** Their
ratio is the source's *duty cycle*: roughly the fraction of span-hours
where the source was actually active. A source with span-hr density
~10x lower than active-bucket density has a duty cycle of ~10% — it's
silent 90% of the time. A source where the two numbers are similar
runs continuously. Sources with extreme duty-cycle mismatches
(`vscode-ext` again) are event-triggered workloads in disguise.

**Test 3: `bucket-intensity` p95 vs `tail-share` top10%.** If the p95
per-bucket magnitude is much larger than 10% of the lifetime
mass-per-bucket-mean, the source has a heavy tail; the top10% of
buckets dominate. If the p95 is close to the mean, the source is
roughly uniform in magnitude even if it has many buckets. The two
lenses should agree on the shape; if they disagree, one of them is
fooled by an outlier the other handled differently.

**Test 4: `model-tenure` span vs `model-cohabitation` co-presence.**
A model with long span (weeks-long lifetime) but low cohabitation
index has been running solo most of its lifetime — it's the only
model the source picks. A model with short span but high
cohabitation is part of a multi-model rotation in a brief experiment
window. A model with long span *and* high cohabitation is in
production and shares the floor with siblings. The combination
distinguishes "incumbent", "experiment", and "fleet member" classes
that no single lens distinguishes alone.

## What this changes about how to read the stack

Once you start looking for divergences, you stop reading any single
lens as authoritative. The mental model shifts from "pick the lens
that answers my question" to "find the disagreement that contains
the diagnosis".

Three operational practices fall out of this shift.

**Practice 1: always run two lenses against any source you care
about.** A single lens can be misleading because every lens has a
shape it's blind to. Two lenses that agree are reassuring; two
lenses that disagree are *more* informative than two that agree,
because the disagreement points at a workload property that doesn't
fit the standard model.

**Practice 2: when investigating a regression, look for which
lens started disagreeing with the others.** A source whose giniLike
suddenly jumped while its volume stayed flat has changed shape, not
size — likely a single user adopted a heavy workflow. A source whose
density suddenly dropped while its active-bucket count rose has
spread out, likely because background polling was added without
adding real work. The lens that moved is the diagnosis.

**Practice 3: don't aggregate across workload classes.** If
`tail-share` shows giniLike spreading from 0.22 to 0.44 across your
6 sources, computing fleet-wide averages of *anything* (latency,
cost-per-token, model-mix-entropy) blends populations that have
fundamentally different statistical structure. The aggregate is a
fiction. Class-segmented reports are slower to produce but reflect
something real.

## The version that makes this practice possible

Worth marking the moment: the `pew-insights` stack only became
divergence-readable in the v0.4.63 → v0.4.66 sequence. Before
v0.4.63 the lens stack was mass-weighted everywhere — `agent-mix`,
`provider-share`, `cost`, and the older `bucket-intensity` all
reported quantities that scaled with volume. Two sources with
different volumes but identical *shape* would always look different,
and two sources with similar volumes but different shapes would
always look similar. The mass-weighted lenses can't expose
divergences because they're all measuring the same thing in
different aggregations.

`model-tenure` (commits `93ab01a`, `85bbc1a`, `a33ada4`) added the
first density scalar that's normalised against time rather than
mass. `tail-share` (commits `aa1dc00`, `6e5280c`, `89b1ad1`,
`bbe71b6`) added the first concentration scalar normalised against
the source's own distribution. Once both of those exist, you can
compute their ratio per source and start finding the cases where
one is high and the other is low. That's the divergence axis the
mass-weighted lens stack couldn't see.

## A worked example: what to do with `vscode-ext`

Concretely, given the divergence diagnosed above:

- **Don't size the editor source's rate limits using its
  `tokensPerSpanHour`.** That number is dominated by silent hours
  and will under-provision the burst. Use `tail-share`'s top1%
  share against the source's lifetime mass to compute a
  burst-budget instead.

- **Don't include the editor source in fleet-wide cost-per-hour
  reports without a class flag.** Its dollars-per-active-hour ratio
  is normal; its dollars-per-wall-clock-hour ratio is meaningless
  because the wall clock includes hours when the editor was sitting
  idle on someone's laptop. Mixing it with the CLI sources (which
  have ~continuous duty cycles during sessions) creates a
  composite mean that describes neither population.

- **Don't try to predict the editor source's next-hour load from
  its previous-hour load.** The autocorrelation is low because the
  source is event-triggered. The CLI sources have meaningful
  hour-over-hour autocorrelation (a session in progress predicts
  more activity); the editor source does not (a burst predicts
  return to silence).

Each of those operational reads requires the divergence between
`tail-share` and `model-tenure` to be visible. A stack with only
mass-weighted lenses would have classified `vscode-ext` as a "small
source, ignore" and missed every one of the consequences.

## Closing

The diagnostic value of a lens stack is not in any single lens. It's
in the *disagreements* between lenses pointed at the same source.
The v0.4.63 → v0.4.66 sequence in `pew-insights` introduced the
first two scalars that could disagree in operationally meaningful
ways — concentration normalised to the source vs density normalised
to the wall clock. The disagreements they expose are workload-class
boundaries that the mass-weighted lens stack quietly aggregated
away. Treat the divergence as the diagnosis, and the lens stack
stops being a reporting tool and starts being a classifier.
