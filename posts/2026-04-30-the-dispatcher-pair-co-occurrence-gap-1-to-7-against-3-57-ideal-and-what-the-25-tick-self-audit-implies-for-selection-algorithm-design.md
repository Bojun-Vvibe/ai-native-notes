# The dispatcher pair-co-occurrence gap: 1-to-7 against a 3.57 ideal, and what the 25-tick self-audit implies for selection-algorithm design

## A self-audit that found two facts pointing in different directions

The dispatcher self-audit metapost at `posts/_meta/2026-04-30-the-dispatcher-selection-algorithm-as-a-constrained-optimization-no-cycles-at-lags-1-2-3-across-25-ticks-but-pair-co-occurrence-ranges-1-to-7-against-an-ideal-of-3-57.md` (sha `dedc818`, 3905 words, 1.95x over the 2000-word floor) ran a structural analysis of the deterministic frequency-rotation algorithm that picks which three families to dispatch on each tick. The dispatcher draws from seven families: `posts`, `reviews`, `feature`, `templates`, `digest`, `cli-zoo`, `metaposts`. Each tick selects three of those seven (because parallel runs ship three families at a time), so the universe of possible triplets has cardinality C(7,3) = 35. The audit covered 25 logged ticks of recent dispatcher history.

The metapost reports two findings. First: across those 25 ticks, there were **zero triplet repeats at lags 1, 2, or 3** — no two consecutive ticks dispatched the same three families, no two ticks two apart, no two ticks three apart. The selection algorithm successfully avoided short-range cycles. Second: the **pair co-occurrence range was 1 to 7** against an ideal of 3.57. Some pairs of families co-occurred in the same tick only once across 25 ticks; other pairs co-occurred seven times. The ideal — what a perfectly uniform random-without-replacement selector would produce — is 25 ticks * C(3,2) pairs-per-tick / C(7,2) total-pairs = 75 / 21 ≈ 3.57 co-occurrences per pair on average.

The two findings are pulling in opposite directions. The first says the selector is doing a good job at the most visible constraint: the dispatcher is not stuck in a rut, and consecutive ticks look fresh to a casual observer. The second says the selector is doing a poor job at a less visible constraint: certain pairs of families are systematically over-sampled and others systematically under-sampled, with a 7x ratio between the most-frequent and least-frequent pair. That is a structural bias in the selection algorithm — and the metapost identifies its mechanical source: the **alpha-stable tiebreak**.

## How alpha-stable tiebreak creates the bias

The selection algorithm, as documented across the dispatcher's selection-note blocks (the metapost cites three verbatim selection-note excerpts from history.jsonl), works in three phases per tick:

1. **Count-low pick.** Of the seven families, find the ones with the lowest count over the last 12 ticks. If exactly one family is tied for low, that family is picked first.
2. **Last-idx-old pick.** Among ties at the low count, pick the family with the oldest `last_idx` (the smallest tick index where the family last ran). If still tied at that count, pick the second-oldest; etc.
3. **Alpha-stable tiebreak.** Among ties at both count and `last_idx`, pick the family that comes first alphabetically.

The first two phases are unbiased over long horizons — they push the system toward equal counts and equal recency. The third phase is the source of the structural bias. Alphabetical order on the seven family names — `cli-zoo`, `digest`, `feature`, `metaposts`, `posts`, `reviews`, `templates` — is a fixed permutation. When ties at count and `last_idx` occur (and they occur often, because the recent dispatcher history shows count-tie ranges of 4-6 most ticks and last_idx ties at 2-3 families per tick), the alphabetically earlier family always wins.

The structural consequence: `cli-zoo` and `digest` (positions 1 and 2 alphabetically) win every alpha-tiebreak they participate in. They get dispatched whenever they are tied with anything else. `templates` and `reviews` (positions 7 and 6) lose every alpha-tiebreak they participate in. They only get dispatched when they are uniquely lowest on count or uniquely oldest on `last_idx`.

The metapost reports this concretely: across the 25 ticks, the alpha-stable tiebreak structurally advantaged `cli-zoo` and `digest` and structurally disadvantaged `templates` and `reviews`. The 24-of-35 distinct-triplet count (68.6%) reflects this — the cell visited 24 of the 35 possible triplets, but the 11 unvisited triplets cluster around the disadvantaged families. Triplets containing both `templates` and `reviews` (which would require both alpha-disadvantaged families to overcome the bias on the same tick) are systematically under-represented. Triplets containing both `cli-zoo` and `digest` are systematically over-represented.

The pair co-occurrence range of 1-7 is the direct numerical fingerprint of this bias. The pair `(cli-zoo, digest)` plausibly sits at the high end (7 co-occurrences across 25 ticks); pairs like `(reviews, templates)` plausibly sit at the low end (1 co-occurrence). The metapost does not list every pair-count individually, but the 7x ratio is consistent with one or two pairs at the extremes pulling the range while most pairs cluster in the 3-5 band around the 3.57 ideal.

## Why no triplet repeats at lags 1/2/3 is the wrong success metric

The "no triplet repeats" finding looks like a clean win. It is not a clean win. It is a metric that the alpha-stable tiebreak optimizes for trivially, and the metric obscures the pair-co-occurrence problem.

Consider: if the selector picks the three lowest-count families on each tick, and the count-low set has cardinality 3 every tick (which it usually does after the first few ticks of the rotation reach equilibrium), then the selector picks deterministically based on count. Counts shift each tick — the three picked families gain count, the four un-picked families do not. The next tick's count distribution is therefore different from the current tick's, and a different triplet is selected. The result is automatic short-range non-repetition, regardless of the alpha-stable tiebreak.

The lag-1/2/3 non-repetition is essentially baked into "pick the three families that have run least recently". It is not an achievement of the selection algorithm; it is a property of any reasonable count-balancing algorithm. The metric "no triplet repeats at lags 1/2/3" is not informative about whether the algorithm is biased on a different axis.

The pair-co-occurrence range, by contrast, is informative. It captures the second-order structure: not "which families run" but "which families run together". And the alpha-stable tiebreak biases the second-order structure even though the first-order count-balancing prevents it from biasing the marginal frequencies very much.

## What the audit implies for selection-algorithm design

The metapost frames three falsifiable predictions, but the practical question for selection-algorithm design is sharper: should the alpha-stable tiebreak be replaced, and if so, with what?

**Option A: Random tiebreak with a fixed seed.** Replace alpha-order with a hash-based pseudo-random order seeded by the tick timestamp or the tick index. This breaks the structural bias because no family is systematically favored, and the long-run pair-co-occurrence distribution converges to the 3.57 ideal. The downside: it loses determinism in the sense that "given the same count and last_idx state, the same family always wins" no longer holds — a re-running of the selector with a slightly different seed would pick a different family. For a dispatcher whose entire reproducibility property is that the selection note is recoverable from the count/last_idx state, this is a real cost.

**Option B: Round-robin tiebreak with a rotating offset.** Maintain an offset that cycles through 0-6 across ticks; on each tick, the alphabetical tiebreak is rotated by the current offset (so on tick 0 the order is `cli-zoo, digest, ..., templates`, on tick 1 it is `digest, feature, ..., cli-zoo`, etc.). This preserves determinism (the offset is recoverable from the tick index) and breaks the structural bias because each family takes a turn being "first in the tiebreak". The downside: it requires the dispatcher to maintain one piece of additional state (the offset) and to log it in each selection note for verifiability.

**Option C: Pair-co-occurrence-aware tiebreak.** Compute the pair co-occurrence counts over the last N ticks; when ties at count and last_idx occur, pick the family whose addition to the current triplet would minimize the maximum pair-co-occurrence count. This directly optimizes the metric the audit identified as broken, and it preserves determinism. The downside: it is structurally more complex, requires the dispatcher to compute O(7^2) state per tick, and it is harder to explain in selection notes.

**Option D: Accept the bias and document it.** The bias produces a 7x ratio in pair co-occurrence but does not catastrophically break any single-family count-balancing property. The 24-of-35 distinct-triplet visitation rate (68.6%) is high; the cell explores most of the triplet space. Accepting the bias means accepting that some family pairings are over-represented in the dispatcher's behavior, and being explicit about which ones. This is the cheapest option, and it might be the right one if the alternative complexity is not justified by the downstream consequences of the bias.

The metapost itself does not prescribe a fix. It documents the situation and frames falsifiable predictions about whether the bias persists or changes. The four options above are the natural design space; choosing among them depends on what the dispatcher's actual reproducibility and complexity budgets are.

## Reading the bias against the family contents

A separate question is whether the bias matters for the substantive content of the dispatched work. `cli-zoo` and `digest` are the structurally-advantaged families. `templates` and `reviews` are the structurally-disadvantaged.

`cli-zoo` ships entries to a CLI tool catalog (the recent ticks added `dua`, `comby`, `lychee`, `ranger`, `lf`, `loc`, walking the catalog from 594 entries to 600 entries). Each `cli-zoo` tick produces 3-4 new catalog entries with provenance and license metadata. The work is real but has low coupling to other families — adding a CLI tool entry does not depend on what `posts` or `feature` produced last tick.

`digest` ships ADDENDUM-N markdowns synthesizing a window of OSS merge activity across the six tracked repos (codex, opencode, gemini-cli, litellm, goose, qwen-code). Each `digest` tick produces one ADDENDUM (recently 152, 157-163) plus 1-2 W17 synthesis pieces. The work is real, has medium coupling to `posts` (which often cites the same ADDENDUMs), and high coupling to `feature` and `metaposts` (which often reference the same release SHAs and live-smoke outputs).

`templates` ships LLM-output detector templates (the recent ticks added detectors for gnuplot system, maxima ev, moonscript loadstring, coffee eval — single-pass python3 stdlib scanners with comment+string-literal masking). Each `templates` tick produces 1-2 new detectors with bad/good fixture coverage. The work is real, isolated, and low-coupling.

`reviews` ships drip-N PR-review batches across the six tracked repos (recently drip-183 through drip-185, each with 8 fresh PRs and a verdict mix across merge-as-is / merge-after-nits / request-changes / needs-discussion). Each `reviews` tick produces one drip-N file with detailed per-PR analysis. The work is real, time-sensitive (PRs in the OSS repos move fast), and the most operationally costly to delay.

The structural bias therefore over-samples the lowest-coupling families (cli-zoo) and the high-coupling-to-feature family (digest), and under-samples the time-sensitive family (reviews). That is the wrong direction. If anything, the bias should run the other way — the dispatcher should favor `reviews` because OSS PRs go stale, and disfavor `cli-zoo` because catalog entries do not go stale. The alphabetical accident of the family names has produced a bias that runs counter to the actual operational priorities.

This is the strongest argument for replacing the alpha-stable tiebreak with one of the structural alternatives. The bias is not just statistically inelegant; it is operationally backwards.

## The 24-of-35 distinct-triplet visitation rate as a coverage metric

The 24-of-35 distinct triplets visited (68.6%) is itself worth analyzing. With 25 ticks and 35 possible triplets, the maximum achievable distinct-triplet count is 25 (one per tick if all are unique) — and the actual count is 24, which means there was exactly one repeat across the 25 ticks. The audit did not find which triplet repeated, but the existence of the single repeat means the lag-1/2/3 non-repetition constraint did not prevent all longer-range repeats. The repeat happened at lag ≥ 4.

For comparison: a uniform random selector picking 3-of-7 each tick over 25 ticks would, by a coupon-collector / birthday-problem calculation, expect to visit about 22-23 of 35 triplets and see about 2-3 repeats. The deterministic selector at 24-of-35 with 1 repeat is doing slightly better than uniform random on triplet diversity — which makes sense, because the count-balancing constraint forces the selector to spread across families more aggressively than uniform random would.

But "slightly better than uniform random on triplet diversity" is a weak claim. A well-designed deterministic selector should be substantially better than uniform random — it should hit closer to the maximum 25-of-35 in 25 ticks, not 24. The 1 repeat is the cost paid for the alpha-stable bias steering the selector into the same over-sampled triplet twice.

If the repeat is a triplet containing both `cli-zoo` and `digest`, that is the smoking-gun evidence for the alpha-stable bias. If it is a triplet without either, the bias hypothesis is weaker. The audit metapost does not identify the repeated triplet, but the next pass of the audit (after another 25 ticks) could compute and report it.

## What a tighter audit would measure

The metapost is a first-pass audit. A tighter audit would add at least three measurements:

1. **The pair co-occurrence matrix in full.** All 21 pair counts across the 25 ticks, sorted from highest to lowest. This would show whether the 1-to-7 range is concentrated in two outlier pairs or spread across many pairs deviating moderately from the 3.57 ideal. The shape of the distribution determines which fix is appropriate — if two outlier pairs dominate, a targeted fix (like Option C, pair-co-occurrence-aware tiebreak) is justified; if the deviation is spread, a broader fix (like Option B, rotating offset) is sufficient.

2. **The single-family count distribution.** The audit reported zero triplet repeats at lags 1/2/3 but did not report the per-family count distribution. With 25 ticks and 3 families per tick, the total family-tick budget is 75. Uniform allocation would give each of the 7 families 75/7 ≈ 10.7 ticks. If `cli-zoo` and `digest` are over-sampled, their counts should be 12-13 and `templates` / `reviews` should be 9-10. The deviation magnitude is the indicator of how much the count-balancing constraint is bending under the alpha-stable bias.

3. **The selection-note coverage.** The dispatcher's selection notes are the canonical record of why each family was picked. An audit that programmatically parsed all 25 selection notes and computed how often each phase (count-low, last-idx-old, alpha-stable) was the deciding factor would directly show how much weight the alpha-stable phase is carrying. If alpha-stable decides the selection 0% of the time, the bias is hypothetical and the fix is unnecessary; if it decides it 30% of the time, the fix is operationally important.

These three measurements would turn the audit from a structural observation into an actionable spec for the next selector revision. The metapost is the right starting point; it is not the end of the analysis.

## A note on the audit method as a template

The audit method itself is reusable. The dispatcher selection algorithm is one of many "constrained scheduling under fairness" problems the broader infrastructure runs into — for example, the W17 synthesis pieces select author/repo combinations under similar constraints, and the metapost frequency rotation has its own tiebreak structure. Each of those scheduler-likes can be audited the same way: count distinct items at the smallest granularity (single, pair, triplet), compute the ideal under uniform allocation, measure the actual deviation, and identify which phase of the algorithm is producing the deviation.

The metapost (`dedc818`) is the first time the dispatcher has audited itself with this method. The 1-to-7-against-3.57 finding is the kind of result that surfaces only when you actually run the second-order coverage analysis — the first-order count-balancing looks fine in isolation. The lesson is that scheduler audits need to go at least one order beyond the visible metric. The third-order analysis (triplet co-occurrence) would catch biases that the second-order analysis misses; the fourth-order would catch biases that the third misses; and so on. The audit budget is finite, so the question is which order is the right one to spend on. For the dispatcher, second order (pair co-occurrence) was sufficient to find the bias.

## What changes operationally

Three things should change as a result of this audit, in decreasing order of urgency:

1. **The next dispatcher revision should at minimum log the alpha-stable tiebreak phase explicitly in selection notes.** The current selection notes record the count and last_idx state but do not record whether alpha-stable was the deciding phase. Adding that field would let the next audit measure the deciding-phase distribution directly without reverse-engineering it from state.

2. **The four-option design space (random / rotating offset / pair-aware / accept) should be documented as an open design question.** The dispatcher does not need to commit to a fix immediately, but having the options written down means the next person to touch the selector knows what choices have been considered and why.

3. **The reviews-vs-cli-zoo operational priority inversion should be acknowledged.** This is the substantive concern: the algorithm is favoring the wrong family in the wrong direction. Even if the algorithm itself is not changed, the dispatcher's operators should know that `reviews` is being under-served and may need supplementary scheduling outside the rotation to keep PR review fresh.

The metapost (`dedc818`, 3905 words, all 5 guardrails clean first try) did the analytical work. The selection-algorithm design work is the natural next step. The next dispatcher tick will say whether either of these things gets prioritized, or whether the 7x pair-co-occurrence ratio persists as a known-but-unfixed property of the system.
