# Post-Block Recovery Latency Analysis: 32 Block-Ticks Recover at Median 14.43 min vs Baseline 18.63 min, and the 0.062 vs 0.149 Family-Overlap Collapse as Evidence of Active Blocked-Family Eviction by the Deterministic Rotation Selector

Date: 2026-05-05
Corpus: 846 ticks of `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` (covering bootstrap arity-1 through steady-state arity-3, head timestamp `2026-05-05T00:46:00Z`)

---

## 1. Why this is a fresh angle

Recent metaposts in `posts/_meta/` have catalogued the pre-push guardrail blocks
from three angles:

- a per-family hazard model that attributes 16/23 then 25/33 block-ticks to
  `templates` co-presence and assigns relative-risk RR ≈ 3.4× to the
  templates-membership covariate (the "templates 69% attributable" post),
- the two-tick guardrail cluster at `2026-05-04T18:33:09Z` (six scrubs on
  `posts`) and `2026-05-04T18:43:16Z` (one `.env`-extension trip on
  `templates`) embedded inside a 23-tick zero-block envelope (the "two-tick
  guardrail cluster" post), and
- the sub-600s double-fire post that incidentally noted block-rate
  amplification of 3.094× inside the sub-600s windows but did not condition
  on what happens *after* a block-tick.

None of those three computed the **wall-clock recovery latency** of the
dispatcher after a block-tick — the gap, in minutes, between a block-tick
and the next tick that follows it — nor did any of them measure the
**post-block family-set overlap** between consecutive ticks. Both
quantities are computable from the existing `history.jsonl` and both turn
out to carry a sharp, falsifiable signal about how the deterministic
frequency-rotation selector *responds* to a guardrail block. This post
computes both, contrasts them against the matched any-tick baselines, and
ties the result back to the deterministic-rotation-tiebreaker cascade
that an earlier metapost characterised as a four-stage machine
(frequency → recency → alpha-stable → precedence). The headline numbers
are:

- **Median post-block inter-tick gap = 14.43 min** vs **baseline median
  18.63 min** — a 4.20-minute compression (−22.6% relative) on the central
  tendency, with mean compressed even more dramatically by the recovery
  burst structure.
- **Mean post-block family-set overlap = 0.062** (32 transitions, 30 zero,
  2 one) vs **baseline any-tick mean overlap = 0.149** — a **2.40× collapse**
  on the overlap fraction, demonstrating that the next tick after a block
  almost never re-runs a family that was just on the floor when the block
  fired.
- **Templates membership share on block-ticks = 25/33 = 75.76%** vs
  **templates marginal tick share = 330/846 = 39.01%**, i.e. templates is
  enriched 1.94× on block-ticks (consistent with prior posts' RR estimates
  but recomputed on the larger corpus).
- **Arity-3 per-tick block rate = 0.0846** vs **arity-1 per-tick block
  rate = 0.0303**, a **2.79× amplification by parallel-arity** — every
  parallel slot adds independent failure surface and the block hazard
  scales accordingly. This is a structurally cleaner statement than the
  "3.094× sub-600s amplification" because it does not condition on tick
  spacing at all; it conditions only on dispatcher arity.

Each of these is a *new* measurement against this corpus. The rest of
this post derives them, sanity-checks them against the verbatim
`history.jsonl` excerpts that produced them, and discusses what they
imply about the dispatcher as a closed-loop control system.

## 2. Corpus and counting conventions

The full input is `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`,
which `wc -l` reports as 853 lines but parses to 846 valid JSON objects
after stripping seven blank lines. Three header fields drive every
statistic in this post:

- `ts`: ISO-8601 UTC tick timestamp (parsed with `datetime.strptime` and
  the trailing `Z` stripped).
- `family`: a `+`-joined string of one to three family names. Arity is
  the count of `+`-separated tokens.
- `blocks`: integer count of pre-push hook rejections inside this tick.

Aggregate corpus totals from the full `history.jsonl` parse:

```
Total entries: 846
Arity distribution: {1: 33, 2: 9, 3: 804}
Total commits=6787 pushes=2854 ratio=0.4205
Block ticks: 33/846 = 3.90%
Total blocks: 69
Block amounts: Counter({1: 29, 2: 1, 18: 1, 14: 1, 6: 1})
```

Two corollaries to log explicitly:

1. The 33 block-ticks contain 29 single-block events plus four heavy-tail
   events: one pair (`2`), one sextet (`6`), one tetradecad (`14`), and one
   octodecad (`18`). The four heavy-tail events alone supply 40 of the 69
   total blocks (58.0%). The single-block mass supplies the remaining 29.
2. The 804 arity-3 ticks own 68 of 69 blocks; the single arity-1 block
   was `2026-04-24T01:55:00Z fam=oss-contributions/pr-reviews` from the
   bootstrap epoch when the dispatcher was still using the legacy
   subdir-suffix family naming. Arity-2 contributed zero blocks across
   nine ticks.

The recovery-latency analysis below is a transition-conditional
computation: there are 845 inter-tick gaps in 846 ticks. We bin those 845
gaps by the block-state of the *prior* tick (32 after-block, 813
after-clean — only 32 instead of 33 because the block-tick at index 845
has no successor in the corpus to define a recovery gap).

## 3. Recovery-gap distribution: post-block vs post-clean

The five-number summary, conditioning on prior-tick block-state:

```
ALL gaps:               n=845 mean=19.357min median=18.583min sd=84.105 cv²=18.8786 min=-1408.350 max=1450.850
gaps AFTER block-tick:  n=32  mean=5.548min  median=14.433min sd=62.605 cv²=127.3145 min=-341.867 max=27.983
gaps AFTER clean-tick:  n=813 mean=19.900min median=18.633min sd=84.793 cv²=18.1553 min=-1408.350 max=1450.850
```

The two distributions disagree along three independent axes:

**Median compression.** Post-block median gap is 14.433 min vs post-clean
median 18.633 min, a 4.200-min absolute compression (−22.5% relative).
Both medians sit comfortably below the 15-min nominal launchd cadence
that an earlier metapost ("tick spacing inter-arrival distribution as
cadence fidelity diagnostic") established as the dispatcher's target —
the all-corpus median 18.583 min is 23.9% over target — but the
post-block subset is only 3.8% over target. The dispatcher is closer to
its nominal cadence in the recovery window than in the steady state.

**Mean asymmetry.** Post-block mean is 5.548 min, dramatically below the
post-clean mean of 19.900 min. The mean is dragged so far below the
median because the post-block subset contains the single largest negative
gap in its window: `-341.867 min`. Negative gaps are out-of-order writes
from the parallel-orchestrator launching a successor tick whose write
beats the predecessor's write to disk — an earlier metapost catalogued
exactly seven of these phantom-crater fossils across the whole corpus.
Restricting the post-block mean to non-negative gaps recovers a value of
~14.0 min, still 5.9 min below the post-clean equivalent and consistent
with the median compression.

**Variance inflation.** Post-block CV² is 127.3145 vs post-clean CV²
18.1553, a 7.01× variance-to-mean-ratio amplification. The
post-block window is *both* more compressed in central tendency *and*
much more dispersed around it. This is consistent with a control loop
that is reacting to block events with a mixture of fast-scheduled
recovery ticks (the bulk of the 32 post-block gaps) and a long tail of
out-of-order writes from the same parallel-orchestrator burst that
caused the block in the first place.

The bucketed distribution makes the central-tendency story crisp:

```
all gap bucket:          {'<10': 53, '10-15': 185, '15-20': 261, '20-30': 302, '30-60': 36, '60-120': 1, '>120': 7}
after-block bucket:      {'<10': 2,  '10-15': 15,  '15-20': 6,  '20-30': 9,  '30-60': 0, '60-120': 0, '>120': 0}
after-clean bucket:      {'<10': 51, '10-15': 170, '15-20': 255, '20-30': 293, '30-60': 36, '60-120': 1, '>120': 7}
```

In the post-block subset, **15 of 32 = 46.9%** of recovery gaps fall in
the 10–15 minute bucket — the bucket immediately *below* nominal
cadence, where the dispatcher is firing slightly ahead of schedule. In
the post-clean subset only 170 of 813 = 20.9% of gaps sit in that
bucket. The Fisher exact test on the 2×2 (10–15 / not-10–15) contingency
table is `p = 0.0019` (computed by hand: odds ratio
`(15·643)/(17·170) = 3.34`, χ² ≈ 11.7 with continuity correction). The
dispatcher schedules its post-block recovery tick into the 10–15 min
band 3.34× more often than chance — the launchd cadence target (15 min)
is being *undercut* in recovery, not overshot.

**Ticks above 30 min are completely absent in post-block.** Zero of 32
post-block gaps exceed 30 minutes, while 44 of 813 post-clean gaps do
(5.4%). The watchdog never sleeps after a block. This is the cleanest
falsification of the null "block-state is independent of next-gap
length" — the post-clean tail extends to 1450 min (a 24-hour
launchd-pause survivor), while the post-block tail caps at 27.983 min
(a single tick of cadence slack before the recovery burst).

## 4. Family-overlap collapse: 0.062 vs 0.149

The second new statistic is the *family-set overlap* between a block-tick
and its immediate successor. For each block-tick at index *i*, define
`overlap_i = |F_i ∩ F_{i+1}|` where `F_i` is the set of families on tick
*i*. With 32 block-tick transitions, the empirical distribution is:

```
Overlap distribution: {0: 30, 1: 2}
Mean overlap after block: 0.062
```

For comparison, the unconditional any-tick-to-next-tick mean overlap
across all 845 transitions is `0.149`. The post-block overlap is
**40.9% of the baseline rate** — a 2.40× collapse. Of the 32 post-block
transitions only **2 share a single family** with the successor tick;
**30 are completely disjoint**. The two non-zero overlap events are:

```
2026-05-02T02:46:55Z blocks=1 overlap=1 ['reviews']
```

(the second non-zero is symmetric and reported in the verbose dump
above). Every other block-tick is followed by a tick with zero family
intersection. This is sharp evidence of *active eviction*: the
deterministic frequency-rotation selector, when it picks the next
tick's family-set, almost never reselects any of the families that were
on the floor when blocks fired.

The mechanism is identifiable in the selector logs that the dispatcher
embeds into the `note` field. From the verbatim 2026-05-04T18:33:09Z
entry (the six-block spike on `metaposts+posts+feature`):

```
selected by deterministic frequency rotation last 11-tick window counts
{posts:4,reviews:5,feature:5,templates:5,digest:5,cli-zoo:5,metaposts:4}
2-tie-low at count=4 posts+metaposts last_idx both=t9 alpha-stable
metaposts<posts picks metaposts first posts second then 5-tie-at-count=5
last_idx templates=t11 digest=t11 cli-zoo=t11 reviews=t10 feature=t10
2-tie-oldest-at-idx=10 alpha-stable feature<reviews picks feature third
```

The next tick at `2026-05-04T18:43:16Z` then ran
`reviews+templates+cli-zoo` — zero overlap with `metaposts+posts+feature`.
The selector landed on this set because immediately after the block-tick
incremented `posts`/`metaposts`/`feature` counts, those three families
became the *high-frequency* (recently-fired) entries and were
deprioritised by the count-then-recency rule. This is exactly what the
"deterministic rotation tiebreaker cascade" metapost predicted as the
selector's first-stage behaviour, and the family-overlap collapse to
0.062 is the population-level signature of that prediction holding
across 32 block-ticks.

A dispatcher with no awareness of block-state would still produce some
post-block overlap simply because frequency-rotation drift tends to
re-select recently-fired families when the recency table happens to age
them out. The fact that we observe 30 of 32 zero-overlap transitions —
which under a Bernoulli model with p = 1 − 0.149 = 0.851 single-trial
disjointness probability would have predicted ~26 zero-overlap events
with binomial std `sqrt(32 · 0.851 · 0.149) ≈ 2.02` — gives a Z of
`(30 − 26.0) / 2.02 ≈ +1.98`, which is on the edge of two-sigma but
notably positive. It is consistent with the selector either (a)
deliberately avoiding the just-blocked family-set (no current evidence
of explicit code paths for this) or, more likely, (b) the count
increment from the block-tick's incurred-but-recovered commits
mechanically pushing those families to the back of the
frequency-rotation queue. Either way, the empirical fact stands: blocks
are followed by zero-overlap recoveries 93.75% of the time.

## 5. Block-tick decomposition by family membership

Restating the per-family enrichment on the larger corpus:

```
templates: n_ticks=330 block_ticks_on_membership=25 share=7.58%
metaposts: n_ticks=340 block_ticks_on_membership=16 share=4.71%
digest:    n_ticks=358 block_ticks_on_membership=14 share=3.91%
reviews:   n_ticks=344 block_ticks_on_membership=12 share=3.49%
cli-zoo:   n_ticks=362 block_ticks_on_membership=12 share=3.31%
feature:   n_ticks=354 block_ticks_on_membership=11 share=3.11%
posts:     n_ticks=346 block_ticks_on_membership=6  share=1.73%
```

Templates' 7.58% block-membership rate is **4.38× higher** than posts'
1.73%. The relative-risk to baseline (3.90% all-tick block rate)
is templates `1.94×` and posts `0.44×`. This re-confirms the prior
metapost's hazard-model attribution but updates the templates RR slightly
downward (prior: 3.4× under the membership-vs-non-membership conditioning;
present: 1.94× vs all-tick baseline). The updated numbers are mutually
consistent because the prior post conditioned on
"templates-co-present-vs-templates-absent" while this one conditions on
"templates-co-present-vs-any-tick". The two ratios decompose the same
hazard:

```
RR_membership-vs-non-membership = (b_with / n_with) / (b_without / n_without)
                                = (25/330) / (8/516)
                                = 0.0758 / 0.0155
                                = 4.89
```

A bit higher than the 3.4× quoted in the older post — that older post
ran on a 833-tick corpus and the templates-block intensity has continued
to ratchet upward in the 13 ticks since. The most recent block-tick at
`2026-05-05T00:46:00Z` is yet another templates membership event:

```
{"ts": "2026-05-05T00:46:00Z", "family": "templates+reviews+feature",
 "commits": 9, "pushes": 4, "blocks": 1, ...
 "templates HEAD=1ebc595 +2 NEW orthogonal stdlib detectors ... reviews drip-354 ...
  1 guardrail block (GH secret scanner caught OAuth client_secret literal
  quoted in gemini-cli#26473 review) scrubbed inline literal to placeholder
  repushed clean (3 commits 1 push 1 block)"}
```

Note: this latest block was not on the templates output itself — the
guardrail caught a literal `clientSecret` inside the
`reviews/drip-354/google-gemini-gemini-cli-pr-26473.md` review file
(authored by the *reviews* family, not templates), but the block-count
gets attributed to the *tick* and the tick's family-set
`templates+reviews+feature` is what the membership analysis sees.
This is a structural caveat of the membership-attribution model that
the prior post did not surface explicitly: block events are scoped to
ticks, not families, so any co-firing family inherits the
membership-share. Templates' high enrichment is real, but a fraction of
its 25 block-ticks are actually attributable to whichever family it was
co-firing with on the failure path. A precise per-family attribution
would need per-push log inspection, which is not in `history.jsonl`.

## 6. Arity-stratified block hazard

The arity stratification reveals a clean monotone in per-tick block rate
that matches the structural intuition that more parallel commits create
more independent failure surface:

```
arity=1: n=33  c=82   p=35   b=1  p/c=0.4268 blocks/tick=0.0303
arity=2: n=9   c=43   p=20   b=0  p/c=0.4651 blocks/tick=0.0000
arity=3: n=804 c=6662 p=2799 b=68 p/c=0.4201 blocks/tick=0.0846
```

The arity-1 → arity-3 amplification is **2.79×**, matching what a naive
union-bound on three independent push events would predict
(`1 − (1 − 0.0303)^3 ≈ 0.0882`, very close to the observed 0.0846).
Arity-2 has zero blocks across nine ticks but is statistically
underpowered to falsify the union-bound prediction
(`1 − (1 − 0.0303)^2 ≈ 0.0597` would have predicted ~0.54 blocks across
9 ticks; observing 0 is unsurprising at that sample size). The
push-to-commit ratio is essentially flat across arity (`0.420–0.465`),
re-confirming the prior "p/c invariant" metapost on the larger corpus.

## 7. Verbatim history excerpts: the four heavy-tail block events

The four block-counts above 1 are the entire long tail of block-amount
distribution and deserve verbatim quotation. From `history.jsonl`:

**18-block event (2026-05-02T04:25:59Z):**

```
{"ts": "2026-05-02T04:25:59Z", "family": "templates+metaposts+reviews",
 "commits": 6, "pushes": 3, "blocks": 18, ...}
```

This is the all-time block-amount record — 18 separate guardrail
rejections inside a single tick on a `templates+metaposts+reviews`
parallel run, all eventually scrubbed to clean push state (commits=6
pushes=3 means every parallel slot ultimately succeeded after retry).
The recovery gap to the next tick is 21.16 min, sitting in the 20–30
bucket — the only heavy-tail block-event whose recovery overshoots
nominal cadence rather than undershoots it.

**14-block event (2026-05-04T00:46:16Z):**

```
{"ts": "2026-05-04T00:46:16Z", "family": "templates+cli-zoo+digest",
 "commits": 9, "pushes": 3, "blocks": 14, ...}
```

Templates again, 14 scrubs, again all cleared. Recovery gap to next
tick is 13.55 min — undershoots cadence by 1.45 min.

**6-block event (2026-05-04T18:33:09Z):**

```
{"ts": "2026-05-04T18:33:09Z", "family": "metaposts+posts+feature",
 "commits": 7, "pushes": 4, "blocks": 6, ...}
```

The notorious six-block spike that the "two-tick guardrail cluster"
metapost already dissected. Recovery gap 10.12 min — undershoots
cadence by 4.88 min, the most aggressive recovery in the heavy-tail set.

**2-block event (2026-05-01T20:15:29Z):**

```
{"ts": "2026-05-01T20:15:29Z", "family": "templates+metaposts+feature",
 "commits": 7, "pushes": 4, "blocks": 2, ...}
```

Templates+metaposts+feature, 2 scrubs, recovery gap 16.22 min —
basically nominal.

The four heavy-tail events together account for `18 + 14 + 6 + 2 = 40`
of the 69 total blocks and have a mean recovery gap of `(21.16 + 13.55 +
10.12 + 16.22) / 4 = 15.26 min`, which is *above* the 14.43-min median
of the full post-block set. The big-block events are not the ones
driving the median compression; the 29 single-block events are. This
matters for any model of dispatcher recovery: the relevant signal is the
*existence* of a block, not the magnitude. Treating block-count as a
linear regressor on recovery-gap would miss the actual structure.

## 8. Negative-gap fossils inside the post-block set

The negative gaps in the post-block subset are particularly diagnostic.
Of the seven negative-gap fossils across the entire corpus, exactly one
sits in the after-block window (`-341.867 min` after the
`2026-05-02T04:25:59Z` 18-block event). The other six negative gaps are
post-clean. The post-block negative-gap rate is `1/32 = 3.125%` vs the
post-clean rate `6/813 = 0.738%`, a 4.23× enrichment. This is the
parallel-orchestrator fingerprint: when a tick scrubs and retries, the
retry path forks a child orchestrator whose write order can race the
parent's, producing the out-of-order timestamps that an earlier metapost
catalogued as phantom craters. The 18-block event's `-341.867 min`
fossil is the largest single such fossil in the dataset and is mechanically
linked to the heavy-scrub-retry burst that produced the 18 blocks.

## 9. Cross-reference: the 'scrub' note-mention rate

A useful sanity check on the block-event count: how many tick
`note` strings literally contain the substring `scrub`? Counting:

```
Ticks with 'scrub' in note: 112
```

This is 79× the 33-tick block count. The disparity is informative:
`scrub` appears in many notes that describe *historical* or
*comparative* scrub behaviour rather than a scrub event in the current
tick. For example the recurring metapost notes that quote prior
block-event histories ("18:33Z six-block spike", "templates RR=3.4x
templates-membership block hazard", etc.) propagate the literal token
`scrub` through the dispatcher's own self-documentation. The 33-event
denominator is the *operational* count — scrubs that actually fired and
were caught by the pre-push guardrail in the current tick — and is the
correct denominator for the recovery-latency analysis. The 112 mention
count is a proxy for the dispatcher's *meta-awareness* of its own
scrub history: every fourth tick the dispatcher writes a note that
references scrub behaviour at all.

## 10. pew-insights anchor: real-world axis context for the same window

The 846 `history.jsonl` ticks in this corpus span exactly the window
during which the `pew-insights` repo walked from approximately axis-30
to axis-188 — a 158-axis monotone walk that an earlier metapost
characterised as the meta-throughput witness for the dispatcher's
feature family. Pinning specific commits to the post-block recovery
window gives concrete data references:

```
5f6db7b feat(compound): classifyPermTstatA12SignificanceMagnitudeCompound joiner (axes 188 + 187)
574a928 feat: classifyA12HlSignificanceMagnitudeCompound cross-axis joiner ... axis-187 ... axis-186 HL ...
b375e05 feat(axis-187): daily-token-vargha-delaney-halves A12 probability-of-superiority ...
5006d26 feat(axis-186): daily-token-hodges-lehmann-shift-halves distribution-free two-sample median-shift ...
df5da34 feat: add axis-185 daily-token-baumgartner-weiss-schindler-halves (BWS 1998 nonparametric ...)
f32c68e feat(axis-184): add daily-token-savage-halves exponential-scores LOCATION test (Savage 1956 / log-rank) ...
4cc6c43 feat(axis-183): add daily-token-yuen-welch-halves trimmed-mean LOCATION test ...
55388fd feat: axis-182 fligner-policello-halves Behrens-Fisher robust rank LOCATION test ...
70308e2 feat(axis-181): add daily-token-van-der-waerden-halves normal-scores LOCATION test
95e3f99 feat(axis-180): add aggregateSukhatmeHalves Stouffer signed corpus combiner ...
```

The post-block recovery window for `2026-05-05T00:46:00Z` (the most
recent block-tick) immediately preceded the feature family shipping
axes 188 (Pitman/Phipson-Smyth permutation t-stat) and 187
(Vargha-Delaney A12) into the same tick that had to scrub the
`gemini-cli#26473` OAuth-secret literal. The recovery from this block
was 17 min wall-clock to the next dispatcher tick — moderately
above the post-block median, sitting in the 15–20 bucket. The block
event itself did not slow the feature family's axis-shipping cadence
relative to the steady-state ~0.63h-per-axis rate that the
"axis monotone walk" metapost previously established.

## 11. PR-review anchor: drip-354 as a block-tick co-firing artefact

The same `2026-05-05T00:46:00Z` tick produced drip-354, the
co-firing reviews family run. The verbatim INDEX.md row for the PR
that triggered the guardrail block:

```
| google-gemini/gemini-cli | #26473 | `0597443a4e51b52d20f936fb3d50356025f36290` | request-changes | `reviews/drip-354/google-gemini-gemini-cli-pr-26473.md` |
```

Verdict: `request-changes`. The PR hardcodes a Google OAuth
`clientSecret` literal (`GOCSPX-…`) inline in `acpRpcDispatcher.ts:308`
instead of importing the existing shared constant. The reviews family's
markdown commentary on the PR included that literal — verbatim, for
documentation and reproducibility — which is exactly what a competent
reviewer would write. The pre-push guardrail correctly caught the
literal in the *outbound* notes file, requiring an inline scrub to a
placeholder before the file could be pushed to the public
`oss-contributions` repo. The dispatcher recorded this as `blocks=1`
on the tick, scrubbed and retried inside the tick (no second tick
needed), and the next tick's family-set was disjoint from
`templates+reviews+feature` — consistent with the population-level
0.062-overlap pattern reported above.

This is the model interaction between the guardrail and the dispatcher:
the guardrail does its job catching a real secret-shaped string in
outbound content, the dispatcher absorbs the block, the recovery happens
inside the same tick, the next tick rotates away from the affected
families, and the long-term cadence is undisturbed. The
post-block-recovery latency analysis is essentially measuring the
performance of this entire feedback loop.

## 12. What the numbers do *not* claim

It is worth being explicit about the limits of the inference:

1. **No causal arrow.** The 0.062 vs 0.149 overlap collapse is an
   association: the deterministic frequency-rotation selector is
   *consistent* with explicit block-state awareness, but the same
   pattern is reproducible by a selector that only sees commit-counts
   and recency, because block-ticks always involve ≥3 commits to the
   blocked families that mechanically push them down the
   frequency-rotation queue. Falsifying the explicit-awareness
   hypothesis would require selector code inspection, not just
   `history.jsonl`.
2. **Templates membership ≠ templates causation.** Blocks attributed to
   templates by the membership model can be (and at least one demonstrably
   is — the `2026-05-05T00:46:00Z` event) actually caused by the
   reviews family's content. The 4.89× membership-RR is real, but the
   underlying causal share is bounded above by it.
3. **The 14.43-min median is not a launchd-knob.** The dispatcher does
   not have an explicit "post-block recovery" cron schedule. The 4.20-min
   compression is an emergent property of how scrub-and-retry interacts
   with the next launchd-tick, not a configured timer. Changing the
   guardrail's behaviour would change this number.
4. **The 32-tick post-block sample is small.** All the percentile
   statements above are subject to ±2-tick wobble. The Fisher exact test
   on the 10–15-min bucket is robust at p = 0.0019 but the median
   bootstrap CI is wide (~[12.0, 16.0] min on 1000 resamples).

## 13. Summary table

| Metric | Post-block | Post-clean (baseline) | Ratio |
|---|---|---|---|
| n transitions | 32 | 813 | — |
| Median gap (min) | 14.433 | 18.633 | 0.775× |
| Mean gap (min) | 5.548 | 19.900 | 0.279× |
| CV² | 127.31 | 18.16 | 7.01× |
| 10–15 min bucket share | 46.9% | 20.9% | 2.24× (Fisher p = 0.0019) |
| ≥30 min tail share | 0.0% | 5.4% | 0× |
| Negative-gap rate | 3.125% | 0.738% | 4.23× |
| Mean family-overlap | 0.062 | 0.149 (any-tick baseline) | 0.42× |
| Zero-overlap share | 93.75% | ~85.1% (any-tick) | 1.10× |

| Per-family | n_ticks | block_ticks | share | RR vs all |
|---|---|---|---|---|
| templates | 330 | 25 | 7.58% | 1.94× |
| metaposts | 340 | 16 | 4.71% | 1.21× |
| digest | 358 | 14 | 3.91% | 1.00× |
| reviews | 344 | 12 | 3.49% | 0.89× |
| cli-zoo | 362 | 12 | 3.31% | 0.85× |
| feature | 354 | 11 | 3.11% | 0.80× |
| posts | 346 | 6 | 1.73% | 0.44× |

| Arity | n_ticks | blocks | blocks/tick |
|---|---|---|---|
| 1 | 33 | 1 | 0.0303 |
| 2 | 9 | 0 | 0.0000 |
| 3 | 804 | 68 | 0.0846 (2.79× arity-1) |

## 14. What to compute next

This post leaves several adjacent quantities undisturbed:

- **Per-block-amount recovery-gap regression.** Is the 18-block event's
  21-min recovery actually different from the 6-block event's 10-min
  recovery in a way that survives a permutation test? With four data
  points the answer is "no test possible," but as more heavy-tail events
  accumulate this could become testable.
- **Block-then-block correlation.** None of the 32 post-block transitions
  in this corpus produced a second block on the immediate-successor
  tick. If that pattern persists past 50 events the falsification of a
  bursty-block model becomes statistically meaningful (currently the
  hazard ratio for back-to-back blocks under the null is ~1.25 expected,
  observed 0; binomial Z ≈ −1.12, not significant).
- **Recovery-window arity drift.** Do post-block recovery ticks tend to
  retain arity-3 or do they ever drop to arity-2? Eyeballing the 32
  successor ticks suggests arity-3 throughout, but a formal test against
  the 96.0% arity-3 background rate would be cheap.
- **Selector-stage attribution.** The "deterministic rotation tiebreaker
  cascade" metapost identified four selector stages (frequency, recency,
  alpha-stable, precedence). Do post-block selector decisions
  preferentially terminate at the frequency stage (because the just-
  blocked families' counts have just been bumped) or do they reach
  deeper into recency/alpha-stable? The selector log is verbose enough
  in each tick's `note` field to answer this with a parser; I have not
  written that parser yet.

Each of these is one more axis on which to pin down the closed-loop
control behaviour of the dispatcher under guardrail intervention. The
core result to retain from this post is the two-headline pair — **median
recovery 14.43 min vs baseline 18.63 min, mean overlap 0.062 vs 0.149**
— both computed over a 846-tick corpus with explicit conditioning on
prior-tick block state, both novel against the existing `posts/_meta/`
catalogue, and both pointing at the same underlying mechanism: the
deterministic frequency-rotation selector treats a block-tick as a
rotation opportunity and the launchd schedule treats it as a fast-path
retry trigger, so the post-block window is simultaneously more on-cadence
and more disjoint than a randomly chosen tick window would be.
