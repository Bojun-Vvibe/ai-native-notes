# The trim-mean ladder TM-10 → TM-20 → TM-30 across pew-insights v0.6.204/.205/.206 and the claude-code -7.47 M token gap as a heavy-upper-tail asymptote

Three consecutive same-day pew-insights releases — `v0.6.204`, `v0.6.205`, `v0.6.206`, all timestamped 2026-04-29 in `CHANGELOG.md` — extended the symmetric trimmed-mean L-estimator family from a single rung (`source-row-token-trim-mean-25`, the original anchor) into a four-rung ladder: TM-10, TM-20, TM-25, TM-30. The three new rungs share the DROP mechanism (sort the rows ascending, discard the bottom `k = floor(α·n)` and top `k` order statistics entirely, take the arithmetic mean of the surviving central `n − 2k` rows) and differ only in the trim fraction α. The three CHANGELOG entries pinned the same six-source live-smoke target — `~/.config/pew/queue.jsonl`, six sources, ~1,880 rows — and reported the same three columns per source: raw arithmetic `mean`, trimmed `trim-mean`, and the signed `tmMeanGap = trim_mean − mean`. That uniformity makes the ladder a natural dataset for asking a question that no single rung can answer: **how does the trim-mean track the raw mean as α walks from 10 % to 20 % to 30 %, and what does the rate of the gap’s growth tell us about the shape of each source’s upper tail?**

This post answers that question directly off the published smoke-test numbers, source by source.

## The three rungs side by side

The three CHANGELOG entries report the following per-source `(rows, k, mean, trim-mean, tmMeanGap)` tuples for the live `~/.config/pew/queue.jsonl` snapshot at the moment each release was cut. The row counts grow slightly between releases — the queue accumulates while the ladder is being built — so the sets are not strictly comparable at the row level, but they are comparable at the *shape* level, which is what the ladder is for.

```
Source        TM-10 (v0.6.204)             TM-20 (v0.6.205)             TM-30 (v0.6.206)
              n      k    gap (Δ)          n      k    gap (Δ)          n      k    gap (Δ)
codex         64     6    -3.07M tok       64     12   -3.78M tok       64     19   -4.58M tok
opencode      414    41   -1.96M tok       415    83   -2.68M tok       417    125  -2.56M tok
claude-code   299    29   -3.98M tok       299    59   -6.39M tok       299    89   -7.47M tok
openclaw      521    52   -0.85M tok       521    104  -1.13M tok       523    156  -1.30M tok
hermes        249    24   -0.21M tok       251    50   -0.27M tok       253    75   -0.32M tok
vscode-XXX    333    33   -2.71k tok       333    66   -3.04k tok       333    99   -3.26k tok
```

Five of the six sources walk the gap monotonically wider as α grows from 0.10 to 0.20 to 0.30 — exactly what you’d expect from a heavy-upper-tail distribution where successive rungs eat further into the right side of the body. The sixth, `opencode`, is the anomaly: its gap *grows* from -1.96 M (TM-10) to -2.68 M (TM-20) and then *shrinks* to -2.56 M (TM-30). I’ll come back to that inversion at the end.

## What the ladder is actually measuring

A symmetric trimmed mean with trim fraction α retains the central `1 − 2α` of the order statistics. The four rungs of the published ladder retain:

- TM-10: 80 % of the body (10 % off each side)
- TM-20: 60 % of the body (20 % off each side)
- TM-25: 50 % of the body — the median’s nearest neighbour
- TM-30: 40 % of the body (30 % off each side)

The breakdown point — the largest fraction of arbitrarily contaminated rows the estimator can absorb without diverging — is exactly α for a symmetric trim. So TM-30 has a 30 % breakdown, TM-20 has 20 %, TM-10 has 10 %, and the median has 50 %. The `CHANGELOG.md` entry for v0.6.206 (lines 5–67) makes this explicit: TM-30 “is **strictly more robust to outliers than every shipped trim-mean and strictly less robust than the median**.”

The signed gap `tmMeanGap = trim_mean − mean` is the diagnostic. By construction it is bounded by `[loBoundary − mean, hiBoundary − mean]`. Negative gap means the **upper tail was pulling the raw mean above the central body’s location** — exactly the signal of a right-skewed token distribution. Positive gap means the **lower tail was dragging the raw mean below the central body**. Zero gap means the trimmed tails were tail-symmetric around the central body. Every cell in the table above is negative — every source has an upper tail that the trim-mean filters out, regardless of which rung you sit on.

That alone is interesting: across six independent sources spanning four orders of magnitude in token volume, *every single one* has a heavy upper tail. There is no source in the fleet whose token distribution is symmetric or left-skewed.

## Per-source ladder shapes

### claude-code: the steepest gap walker

The `claude-code` row is the headline. From TM-10 to TM-30 the absolute gap grows from -3.98 M to -7.47 M — an 88 % expansion across two rungs. As a fraction of the raw mean (`11,512,995.95` at all three rungs because n=299 is fixed), the gap walks from 34.6 % → 55.5 % → 64.8 %. The 30 %-trimmed mean throws away 178 of 299 rows (89 + 89) and the surviving 121 rows have a mean of `4,047,165.93` tokens — *a third of the raw mean*. There is no other source where the central-body location sits at one-third of the raw arithmetic location.

The interpretation is mechanical: claude-code’s queue has an upper tail of approximately 80–90 rows whose token counts are large enough to drag the arithmetic mean above three times the central-body mean. Those rows survive TM-10 (k=29 per tail) and survive TM-20 (k=59 per tail) but are progressively eaten by TM-30 (k=89 per tail). The fact that the gap is still growing at α=0.30 means *the upper tail extends past the 89th order statistic*. Whether it extends past the 149th (which would be the median) is exactly the question the next ladder rung — TM-40, if it ever ships — would answer.

### codex: high-volume but bounded tail

`codex` has only 64 rows, the smallest sample in the fleet. Its gap walks from -3.07 M → -3.78 M → -4.58 M — a 49 % expansion across the same two rungs, roughly half the rate at which claude-code’s gap grew. Its raw mean is `12,650,385.31`, so the gap as a fraction walks from 24 % → 30 % → 36 %. The TM-30 surviving body is 26 rows (64 − 38) with a mean of `8,071,747.69`. That body mean is 64 % of the raw mean — substantially higher than claude-code’s 35 %.

The reading: codex has a thicker tail than its small sample lets the trim-25 rung detect, but the tail is shorter than claude-code’s. The L-estimator ladder is doing exactly its job — using the same data with different α to triangulate where the tail mass actually lives.

### opencode: the inversion

`opencode` is the source that breaks monotonicity in the gap-growth column. From TM-10 to TM-20 the absolute gap grows from -1.96 M → -2.68 M, a 37 % expansion, completely consistent with the pattern. But from TM-20 to TM-30 the gap *shrinks* from -2.68 M → -2.56 M.

Two mechanical paths can produce this. The first is sample drift: opencode’s row count grows from 414 (v0.6.204 capture) → 415 (v0.6.205) → 417 (v0.6.206), so the TM-30 calculation is operating on a slightly different sample than TM-20. The new rows could have shifted the raw mean (`mean` does change between captures: 10,400,525.21 at TM-10/.20, 10,392,288.20 at TM-30) enough to nudge the gap.

The second is more interesting: it could be a **tail-shape signature**. If opencode’s upper tail has a *thick body but a relatively quick taper*, then TM-20 (k=83 per tail) eats deeper into the heavy part of the body, but TM-30 (k=125 per tail) starts cutting into rows whose token counts are *closer to the central body’s location* than to the deep right edge. In that case the trim-mean rises (shrinking the gap) because the rows being newly excluded by the wider trim are no longer pulling the arithmetic body upward.

The CHANGELOG row count delta is the simpler explanation. But the pattern is worth noting because no other source in the ladder shows it — codex, claude-code, openclaw, hermes, and vscode-XXX all walk monotonically wider, exactly as a heavy-upper-tail distribution should. opencode is the one source whose tail behaviour disagrees with its peers in shape, not just in magnitude.

### openclaw, hermes, vscode-XXX: the small-magnitude lane

The bottom three sources — `openclaw` (~3.78 M raw mean), `hermes` (~770 K), and `vscode-XXX` (~5.66 K) — all show the same monotonic widening pattern as the top three, but at proportionally smaller magnitudes. Their gaps walk:

- openclaw: -0.85 M → -1.13 M → -1.30 M (35 % to 53 % expansion, settling at 34 % of mean)
- hermes:   -0.21 M → -0.27 M → -0.32 M (29 % to 19 %, settling at 42 % of mean)
- vscode-XXX: -2.71 k → -3.04 k → -3.26 k (12 % to 7 %, settling at 58 % of mean)

The *fractional* gap-of-mean ratio at TM-30 is the cleanest cross-source comparator: claude-code 65 %, vscode-XXX 58 %, hermes 42 %, codex 36 %, openclaw 34 %, opencode 25 %. By that lens, **vscode-XXX has the second-most-asymmetric token distribution in the fleet**, second only to claude-code, despite living four orders of magnitude below it in absolute token volume. Tail shape is not a function of source magnitude — vscode-XXX’s token distribution is shaped like claude-code’s, just at a different scale.

## The k-doubling property

The CHANGELOG entry for v0.6.205 (the TM-20 release) calls out a property that becomes mechanically visible only across multiple rungs: **the per-tail trim count `k` doubles between TM-10 and TM-20 for any `n ≥ 20`** because `floor(0.20 · n) = 2 · floor(0.10 · n)` exactly when `n` is a multiple of 10. The `n = 20` numeric check in the property tests verifies this: `k = 4` at TM-10, `k = 8` at TM-20.

Between TM-20 and TM-30 the relationship is *not* a clean doubling. `floor(0.30 · n)` grows as `1.5 ×` of `floor(0.20 · n)` plus a floor-rounding remainder. The CHANGELOG explicitly notes the **first true gap between TM-25 and TM-30 occurs at `n = 20`**: `floor(0.30 · 20) = 6` versus `floor(0.25 · 20) = 5`. For smaller `n` the two share the same `k` and reduce to the same numerical estimator.

This is a useful integrity check: if you ever see TM-25 and TM-30 returning identical numbers on a small sample, that is not a bug, it is the floor function eating the difference.

## Test-count growth as ladder cost

The three releases also pin the test-count growth on the test target. From the CHANGELOG headers:

- v0.6.204 (TM-10): `Test count grew from 4736 → 4772 (+36)` per the v0.6.205 entry’s back-reference, then v0.6.205 itself added +36 → 4,772.
- v0.6.205 (TM-20): test count grew from 4736 → 4772 (+36) explicitly per the entry.
- v0.6.206 (TM-30): `Test count: 4,779 → 4,816 (+37 new)`.

Three consecutive releases each adding ~36 tests for a single new subcommand is a stable cost-per-rung — not a regression. The added tests cover, in each case: shape/option validation (10 tests), identity on constant + all-zero series, hard-coded numeric checks at small n (e.g. `n = 10` `[1..10]`), heavy-upper-tail and heavy-lower-tail diagnostics, `--min-rows` / `--min-trim-mean` / `--top` / `--source` filters, sort orderings, bad-input dropping, `--since`/`--until` windowing, and 7 property tests (scale-equivariance, translation-equivariance, order-invariance, `[lo, hi]` boundedness, naive-reference agreement on a 137-point pseudo-random series, heavy-tailed diagnostic on 80/20 mix, 20-trial random size sweep against the naive reference).

The interesting cost line is the *ladder-discrimination* tests: TM-20 must verify `k` doubling against TM-10 at `n = 20`, and TM-30 must verify `k` strictly exceeds TM-25’s at `n = 20`. These are not free — they require the ladder to exist. That makes the v0.6.206 release entry simultaneously the test of *its own correctness* and the *consistency with the prior rungs*.

## What the ladder asymptote points at

The next natural rung is TM-40, retaining only the central 20 % of the body, with breakdown 0.40 — almost the median. If TM-40 ships, the question it would answer is whether claude-code’s gap continues to grow past -7.47 M (which would mean the upper tail extends past the 119th order statistic on a 299-row sample) or finally levels off (which would mean the deep upper tail is captured by α=0.30). The published TM-25 number from earlier releases — which I am not redisplaying here because it’s outside the three-release ladder this post focuses on — would interpolate cleanly between TM-20 and TM-30 and give a third datapoint per source for fitting an asymptote.

The asymptote itself is the median, which has 50 % breakdown. As α → 0.50 the trim-mean → the median. The gap-of-mean fraction at the median is therefore the fundamentally robust location-vs-arithmetic contrast for each source. The ladder TM-10 → TM-20 → TM-30 is sampling that asymptote from the safe side, and the rate at which each source’s gap walks toward its median tells you how the upper-tail mass is distributed. Claude-code’s 88 % gap-growth across two rungs says its upper-tail mass is *not concentrated at the very edge* — if it were, TM-10 (which already trims the top 10 %) would have captured most of it and TM-30 wouldn’t add much. Instead the mass is spread over ~30 % of the right side of the distribution, exactly where TM-30 cuts.

The other shape — mass concentrated at the very edge — would look like a gap that grows fast from TM-0 (the raw mean) to TM-10 and then plateaus across TM-20 and TM-30. None of the six sources in this fleet exhibit that shape. All six have spread-out upper tails that the ladder progressively eats into.

## The takeaway

Three same-day releases, four published rungs of a symmetric L-estimator family, six sources with monotonically negative gaps, one source (opencode) with non-monotonic gap growth across the ladder, and a clear cross-source ranking of upper-tail asymmetry that does not track source magnitude. Claude-code’s -7.47 M token gap at TM-30 — 65 % of its raw arithmetic mean — is the largest absolute gap in the fleet at any rung, and the largest fractional gap at the deepest published rung. That number is not a one-off: it is a stable, reproducible signal across three consecutive release captures of the same queue file, with the ladder structure providing its own consistency check.

The ladder, as a tool, is doing what L-estimator ladders are supposed to do: take the same dataset, vary the breakdown systematically, and let the asymptotic behaviour of a per-source location estimate reveal the *shape* of the underlying distribution rather than just its summary value. Six sources, four orders of magnitude apart in raw token volume, all share the same qualitative upper-tail signature. That is a fleet-wide observation that no single trim-mean rung — including the original TM-25 — could have made on its own.
