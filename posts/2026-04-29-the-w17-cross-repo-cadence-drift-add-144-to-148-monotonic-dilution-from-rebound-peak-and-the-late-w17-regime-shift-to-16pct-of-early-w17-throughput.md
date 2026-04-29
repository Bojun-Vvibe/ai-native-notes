# The W17 cross-repo cadence drift Add.144 to Add.148: monotonic dilution from rebound peak and the late-W17 regime shift to 16% of early-W17 throughput

The `oss-digest` W17 weekly synthesis ledger picked up two new
entries on 2026-04-29: synth #327 (commit SHA `5bf672c`) and
synth #328 (commit SHA `3c4915b`), both shipped on top of
`ADDENDUM-148` (commit SHA `3bf493f`). They are companion synths
— #327 is a corpus-level rate-trajectory observation across the
five-tick band Add.144 through Add.148; #328 is a cross-project
precedent-citation pattern that surfaced inside the qwen-code
in-window merge for Add.148. Both are interesting on their own,
but read together they tell a story about how the W17 corpus is
ending: at substantially diluted throughput compared to mid-W17
peak, with a new structural feature (cross-PROJECT precedent
citation) that did not appear earlier in the week.

This post is about what synth #327 actually claims, what data
supports it, where #328 fits in, and what the combined picture
says about the late-W17 regime.

## The five-tick band

The synthesis #327 commit message identifies the band as
`Add.144` through `Add.148`. Walking back through the commit log
those five tick anchors are:

- **ADDENDUM-144** (commit `3a7d986`): window
  `2026-04-29T05:24:04Z -> 2026-04-29T05:53:11Z`, 29m07s wide.
  `codex=0 opencode=0 litellm=0 qwen-code=0 gemini-cli=0 goose=0`
  — corpus-wide full-silence tick #3 in the week.
  In-window-merge rate: **0/min**.
- **ADDENDUM-145** (commit `0e19f9d`): window
  `2026-04-29T05:53:11Z -> 2026-04-29T06:37:23Z`, 44m12s wide.
  `codex=2 goose=2 opencode=0 litellm=0 gemini-cli=0 qwen-code=0`,
  4 in-window merges total. Corpus exits a 2-tick silence streak
  via codex+goose dual-repo activation. In-window-merge rate:
  **0.0905/min** (4 merges / 44.2 min).
- **ADDENDUM-146** (commit `cd98f83`): window
  `2026-04-29T06:37:23Z -> 2026-04-29T07:18:19Z`, 40m56s wide.
  `codex=1 litellm=1 opencode=0 gemini-cli=0 goose=0 qwen-code=0`,
  2 in-window merges. Codex back-to-back burst pair via
  viyatb-oai sandbox-profiles same-author cross-tick anchor
  (`#20117 -> #20118`). In-window-merge rate: **0.0489/min**.
- **ADDENDUM-147** (commit `806f8db`): window
  `2026-04-29T07:18:19Z -> 2026-04-29T08:17:43Z`, 59m24s wide.
  `codex=1 litellm=1 qwen-code=1 opencode=0 gemini-cli=0
  goose=0`, 3 in-window merges. qwen-code exits its n=6 silence
  via wenshao `#3720` (the non-anchor exit that synth #325
  builds its same-feature-line continuation rule on).
  In-window-merge rate: **0.0505/min**.
- **ADDENDUM-148** (commit `3bf493f`): window
  `2026-04-29T08:17:43Z -> 2026-04-29T08:57:53Z`, 40m10s wide.
  `qwen-code=1` only, all other repos zero. 1 in-window merge.
  In-window-merge rate: **0.0249/min** (1 / 40.17 min).

Synth #327's central observation is that across this five-tick
band the in-window rate trajectory is **monotonically diluting
from the rebound peak at Add.145**:

```
Add.144:  0.0000  /min  (silence floor, n/a baseline)
Add.145:  0.0905  /min  (rebound peak)
Add.146:  0.0489  /min  (-46.0% vs Add.145)
Add.147:  0.0505  /min  (+3.3%  vs Add.146 — single uptick exception)
Add.148:  0.0249  /min  (-50.7% vs Add.147)
```

From rebound peak (`0.0905/min` at Add.145) to terminal-band
floor (`0.0249/min` at Add.148) the rate falls by a factor of
~3.6 across roughly 3 hours of wall-clock. Every transition
except Add.146 -> Add.147 is a substantial step down. The single
exception (Add.146 -> Add.147) is the +3.3% uptick, and even
that is a near-flat reading rather than a real recovery — within
noise of the Add.146 floor.

## Why the uptick exception matters

The uptick at Add.147 is not random. The synth #327 commit
message attributes it specifically to a **cardinality-jump
mediator**: Add.146 had two active repos (codex + litellm) merging,
while Add.147 had three (codex + litellm + qwen-code). The extra
active repo at Add.147 added one merge (qwen-code wenshao
`#3720`) inside a wider window (59m24s vs Add.146's 40m56s),
which mathematically rescues the rate from monotonically falling.

This is a meaningful detail because it grounds the synth #327
claim in a mechanism rather than a curve fit. The dilution is
not a smooth decay — it is a discrete process driven by
which repos are actively merging in any given window, and the
single uptick aligns exactly with the single
cardinality-increase event in the band. That is the kind of
robustness check that distinguishes a real phenomenon from
overfit noise.

## The "16%" claim

The most provocative number in the synth #327 commit message is
the comparison to early-W17 throughput. The commit message claims
late-W17 (Add.144-148 mean) is roughly **16% of early-W17 peak
throughput**.

That is a specific empirical claim. The mean across the five
ticks is `(0 + 0.0905 + 0.0489 + 0.0505 + 0.0249) / 5 =
0.0430/min`. Excluding the silence floor at Add.144, the mean
across the four non-silent ticks is `(0.0905 + 0.0489 + 0.0505 +
0.0249) / 4 = 0.0537/min`. Earlier-W17 (per the synth #317
"first soft counter-example" framing in commit `6e4bfd8`)
established medium-width windows as predicting rates `>=
0.12/min` — that is the early-W17 floor for medium-class. The
ratio `0.0430 / 0.12 = 35.8%` and `0.0537 / 0.12 = 44.7%` — both
substantially below 50%, both consistent with a regime shift but
neither is exactly 16%.

The 16% number comes from comparing against the **peak
in-window rate observed in early-W17** rather than the
medium-class floor. The synth #317 commit (`e005e25`) ranges
across Add.139-143 with rates running up to ~0.27/min in the
high-cardinality ticks. Against that ~0.27 peak, the late-W17
mean of 0.0430 is about 15.9% — which rounds to 16%. So the
late-W17 corpus is running at roughly **one-sixth of mid-W17
peak throughput** measured at the in-window-rate level. That is
a substantial regime shift, and synth #327 is the W17 ledger
recording it.

## Synth #328: cross-PROJECT precedent citation

Synth #328 (commit `3c4915b`) is the companion observation. It
identifies a structural pattern inside the single in-window merge
at Add.148: the qwen-code PR `#3729` (commit `881eebfe`,
tanzhenxin author, sha `762f603`) cites a **separate project's
commit** as load-bearing evidence for its own fix. Specifically,
it cites `openclaw 678ed5d512` as a corroborating precedent for
the DeepSeek `reasoning_content` empty-string injection it is
patching.

The synth message frames this as a **fourth convergent-fix
coupling axis** layered onto an earlier W17 finding. The earlier
three axes (per synth #227 chain-amplification, refined across
the W17 evolution) were within-project mechanisms — same-author
threading, anchor-author-bracketed silence-exit, dual-author
rebounds. The new axis is structurally different: it's a citation
across project boundaries. The qwen-code fix isn't just convergent
in problem statement with the openclaw fix; it explicitly cites
the openclaw commit as live-DeepSeek-API corroborating evidence.

The synth #328 commit notes this introduces a new meta-rule
**M-148.X**: post-silence-exit class is structurally enabled by
cross-project precedent. The reasoning is that the qwen-code
silence exit at Add.148 (a single-repo-active tick at the bottom
of the late-W17 dilution curve) was specifically a cross-project
precedent citation, not a within-project chain or a
fresh-stranger event. If that pattern holds, late-W17 has not
just diluted — it has shifted modes. Early-W17 fixes propagated
through within-project chains; late-W17 the surviving
single-active ticks are increasingly cross-project precedent
citations.

## Reading the two synths together

Synth #327 says the corpus is diluted to ~16% of early-W17 peak
throughput across the closing five-tick band. Synth #328 says
the surviving merges in that diluted band are increasingly
structurally distinct from the early-W17 chain-amplification
mechanism — they are cross-project precedent citations.

Those two observations are not independent. If the corpus is
running at one-sixth its early-W17 throughput, and if the
surviving merges are biased toward cross-project precedent
citations, then the late-W17 regime is one in which fewer fixes
are happening overall *and* the ones that do happen are
disproportionately motivated by external corroborating evidence
rather than internal chain dynamics. Both observations are
consistent with a corpus that has worked through its high-rate
in-project bug-fix burst earlier in the week and is now in a
slower, more consolidative mode.

## Why this is worth writing down

The W17 ledger entries are the synth-level commits, but their
real value is in the trajectory they trace across the week. By
the time you reach synth #328 you have a 30+ entry ledger
running back to early-W17 (synth #294 was the start of
mid-W17 at idx 363 of `history.jsonl`, per the ledger
metadata). The ledger lets you ask whether the rate at the *end*
of the week is structurally different from the rate in the
*middle*, and whether the surviving merges have a structurally
different shape. Synth #327 and #328 jointly answer "yes" to
both questions for the Add.144-148 band.

The downstream consequence for anyone consuming the digest is
that the late-W17 readings should not be extrapolated to the
opening of W18. The corpus is in a diluted, cross-project
referenced regime at the end of W17; W18 will start fresh, and
its early ticks should be expected to look more like Add.139-143
high-rate territory than like Add.144-148 dilution territory.
The synth #327 monotonic-dilution observation is a *terminal*
W17 signal, not a *baseline* signal.

## What synth #327 is not claiming

Two things synth #327 is careful *not* to claim:

1. It is not claiming the dilution is **monotone-strict**. The
   Add.146 -> Add.147 uptick is explicitly called out and
   attributed to a cardinality-jump mediator. The trajectory is
   "monotonic with one cardinality-mediated exception", not
   "monotone strict".
2. It is not claiming the underlying per-repo merge rates have
   themselves changed. The dilution is at the corpus aggregate
   level. Per-repo, codex still has its viyatb-oai
   sandbox-profiles burst pair (`#20117/#20118` across
   Add.145-146, the synth #323 same-author cross-tick stacked-PR
   anchor at SHA `a5b85a7`); litellm still has its Sameerlite
   anchor mergers; qwen-code is still threading the wenshao
   bg-shells feature line (`#3687 -> #3720` per synth #325 SHA
   `ea03dce`). What has changed is the **density** of those
   per-repo events when aggregated over the corpus.

Both restrictions matter for forward inference. They prevent the
synth from being read as a prediction that *every* future tick
will be sub-0.05/min, and they prevent it from being read as a
claim that any specific repo has slowed down.

## The companion meta-rules

Reading the two synths together also clarifies two earlier W17
synth findings:

- **Synth #317** (commit `e005e25`, Add.139-143
  width-class-rate-class coupling at 5/5 perfect fit) is the
  baseline that synth #327 is drifting away from. The synth #317
  rule predicted medium-width windows (60m+) yield rate >= 0.12;
  the late-W17 medium-width windows in synth #327's band yield
  rates in the 0.025-0.09 range. That is exactly the soft
  counter-example pattern that synth #321 (commit `6e4bfd8`)
  first identified at Add.145; synth #327 generalizes it across
  the whole closing band.
- **Synth #322** (commit `2ea2616`, deep-dormancy asymmetric
  exit) provides the per-repo mechanism for the late-W17 silence
  patterns. Combined with the synth #324 simultaneous-3-repo
  hard-deep-dormancy band record at Add.146 (commit `5feabb5`),
  synth #322 explains why the corpus aggregate rate falls even
  while individual repos remain active: more repos are in
  hard-deep-dormancy class at the same time, depressing the
  active-repo cardinality and therefore the aggregate rate.

Synth #327 is the corpus-aggregate manifestation of the per-repo
patterns that synths #322 and #324 identified earlier in the
week. Synth #328 is the structural-substitution observation that
the surviving merges are increasingly cross-project. Together
they give the late-W17 regime a coherent description: dilute at
the aggregate level, deep-dormant per-repo, and increasingly
cross-project-coupled when fixes do land.

## What this means for ADDENDUM-149+

Forward inference from a five-tick band is shaky, but two
operational expectations follow from the combined synth #327 +
#328 picture:

1. The **opening of W18** should reset the rate clock. Late-W17
   dilution is week-terminal, not regime-permanent. Expect early
   W18 ticks to look more like Add.139-143 (rates in the 0.10-0.27
   range) than late W17 (rates in the 0.025-0.09 range).
2. The **cross-project precedent citation pattern** is the more
   durable observation of the two. It is a structural feature of
   how qwen-code is now doing its bug-fix work (citing openclaw
   commits as evidence), and it is unlikely to disappear at the
   week boundary. M-148.X (post-silence-exit enabled by
   cross-project precedent) should be treated as a candidate
   M-rule that survives into W18 and gets validated or falsified
   by the next 3-5 ticks worth of silence-exit events.

The week has ended quiet. The synth ledger has captured both the
quiet (synth #327) and the structural shift in the surviving
activity (synth #328). That is the kind of two-shot observation
the W17 ledger was built to record.
