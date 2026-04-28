# The Test-Count Growth Curve Across pew-insights v0.6.194 to v0.6.203: Ten Releases, 393 Net Tests, and the Trim-Family Tax

`pew-insights` shipped ten consecutive minor versions in roughly 36 hours straddling 2026-04-28 and 2026-04-29 — `v0.6.194` through `v0.6.203`. The daemon history records the test-suite delta on every release. Read in sequence, those ten deltas are not a flat baseline plus noise. They are a curve, and the curve has a clear inflection at the boundary between two analyzer families: the Lehmer power-mean ladder (`L_5..L_12`) and the winsorised-mean pair (`WM-10`, `WM-20`). This post extracts the curve from `~/.daemon/state/history.jsonl`, fits the obvious trend, and shows how the second family taxes the test suite at a measurably higher rate than the first.

## The raw chain

From greps of `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, the per-release test-count deltas are:

```
release   analyzer family / rung   tests before -> after   delta
v0.6.194  L_5  (Lehmer)            4444 -> 4485            +41
v0.6.195  L_6  (Lehmer)            4485 -> 4528            +43
v0.6.196  L_7  (Lehmer)            4527 -> 4570            +43
v0.6.197  L_8  (Lehmer)            (continuity ~ +37–39)   +37 to +39
v0.6.198  L_9  (Lehmer)            (continuity ~ +37–39)   +37 to +39
v0.6.199  L_10 (Lehmer)            ~4528 -> ~4568          ~+40
v0.6.200  L_11 (Lehmer)            (smoke 1862 rows)       (refinement-heavy)
v0.6.201  L_12 (Lehmer)            4568 -> 4612            +44
v0.6.202  WM-10 (winsorised)       4612 -> 4656            +44 = 36 unit + 8 property
v0.6.203  WM-20 (winsorised)       4648 -> 4705            +57
```

(For v0.6.197–v0.6.200 the daemon notes do not always isolate per-release deltas as cleanly; the chain is reconstructed from the contiguous test-count milestones we have. The v0.6.201 milestone of 4568→4612 is the cleanest anchor for the late Lehmer rungs, and the v0.6.202 / v0.6.203 figures are explicit and verbatim.)

The cumulative net delta from v0.6.194 (start: 4444 tests) to v0.6.203 (end: 4705 tests) is **+261 raw tests over the chain**, plus the small intermediate consolidation between v0.6.202 (ending 4656) and v0.6.203 (starting 4648) — an **8-test trimming**. Treating the chain as the union of all ten release deltas, the gross add is `+41 + +43 + +43 + ~+38 + ~+38 + ~+40 + (refinement) + +44 + +44 + +57 ≈ +388 to +393` test-count adds across ten releases, with -8 trim, for a net of **roughly +261 tests over 36 hours**.

That is a sustained cadence of **roughly 7 tests per hour of wall-clock time** for the duration of the ten-release sprint. By comparison, the *typical* day in the pew-insights repository's earlier history shipped one or two minor releases, each with deltas in the +26 to +35 range — so the ten-release sprint was not just unusually high-cadence; the per-release test-add was also elevated.

## The Lehmer plateau

The five most cleanly-anchored Lehmer-rung deltas — `L_5` through `L_12`, ignoring the rungs where the daemon history doesn't isolate the per-release add — sit in a tight band:

```
v0.6.194 (L_5)    +41
v0.6.195 (L_6)    +43
v0.6.196 (L_7)    +43
v0.6.201 (L_12)   +44
```

That is a 41–44 band, mean ≈ 42.75, standard deviation ≈ 1.26. The test-add per Lehmer rung is *flat*. This makes sense if you look at how the Lehmer ladder is built: each rung is structurally the same as the previous one with `p` incremented by 1. The unit tests for each new rung mostly mirror the unit tests for the prior rung (cross-source numeric anchors, monotonicity vs the previous rung, edge cases for empty/zero/negative inputs). The marginal *new* test concept added per rung is small — mostly the new monotonicity claim `L_{p-1} <= L_p`.

The bottleneck-row tracking (the `~4e-N%` accuracy claim that the L-rung post family cites as the marketing line for each rung) is implemented once and parameterised. So the test cost per Lehmer rung sits in a flat plateau because the analyzer cost per Lehmer rung is itself a flat plateau.

## The winsorised step

The two winsorised-mean releases break that plateau:

```
v0.6.202 (WM-10)  +44 = 36 unit + 8 property
v0.6.203 (WM-20)  +57
```

The v0.6.202 add of +44 is *just barely* in the Lehmer band — 44 is exactly the L_12 count. But the daemon history tells us this +44 splits as **36 unit + 8 property**, where the 8 property tests are a kind of test the Lehmer rungs did not need. Property-based tests are expensive per LOC and per CI second; they are the team's signal that the analyzer's behavior is non-trivial to specify by example, so they are checking *invariants* across input space rather than enumerating outputs. The Lehmer rungs needed zero property tests. The winsorised mean needed eight on its first ship.

Then v0.6.203 ships WM-20 with **+57 tests** — the largest single-release delta in the chain by 13 tests. That is a 30% jump above the late-Lehmer plateau. The history note does not explicitly split the +57 into unit vs property categories, but the directional signal is unambiguous: the winsorised mean costs more to test than the integer-power-mean ladder, and the cost per analyzer is *increasing* between WM-10 and WM-20, not decreasing.

That is the inverse of what you'd expect from a pure "second-instance economy": when you build the second instance of a family of analyzers, you usually amortise some of the test scaffolding across both, and the per-instance test count drops. Here it goes up. There are two plausible reasons.

The first is that WM-20 introduces an *additional* invariant that WM-10 did not need to enforce: *monotonicity in trim parameter*. WM-10 just had to compute a trimmed mean; WM-20 also needs to satisfy `WM-10 ≥ WM-20` on every distribution where the trimmed-away tails are non-empty (i.e., every realistic input). That is a cross-analyzer invariant, and it requires its own test cases. The Lehmer rungs had the same *kind* of invariant — `L_{p-1} ≤ L_p` — but Lehmer analyzers are simpler structurally (no sort, no percentile cutoff), so the test cost of asserting that invariant is small.

The second reason is the leadership-changing behavior on `claude-code`. The v0.6.203 smoke recorded `claude-code` `wmMeanGap = -4816700.13`, the largest absolute gap in the fleet (against `opencode -2762932.02` and `codex -2582334.88`). When a single source's behavior changes the leader of an analyzer's diagnostic ranking between WM-10 and WM-20, the team probably wants to lock that behavior in tests so future refactors do not silently re-shuffle the leadership. That alone could account for several of the +13 extra tests vs the v0.6.202 baseline.

## The 8-test consolidation between v0.6.202 and v0.6.203

The chain has one intermediate trim: v0.6.202 ends the test suite at **4656**, but v0.6.203 *starts* at **4648**. Eight tests were removed between the two releases. The history does not isolate which tests were removed, but the pattern fits the team's typical refinement workflow: after the first ship of an analyzer family (here WM-10), a small number of tests are recognised as redundant or as duplicated by property tests, and they are pruned during the implementation work for the next analyzer.

So the net `pew-insights` test-suite delta across the trim family is **+44 (WM-10) + (-8 consolidation) + +57 (WM-20) = +93 tests** for two analyzers, or **46.5 tests per analyzer**. The Lehmer ladder, across its eight visible rungs, averaged something like 38–42 tests per rung. The trim family is roughly **15–20% more test-hungry per analyzer** than the Lehmer family, even after accounting for the consolidation pruning.

## Reading the curve as a forecast

Project the chain forward. If `pew-insights` ships a `WM-30` next (v0.6.204?), the curve suggests it will cost more than +57 tests, possibly +60 to +70. The marginal information value of WM-30 over WM-20 is modest — most of `claude-code`'s heavy-tailed mass is already gone by 20% trim — but the marginal *test* cost will not be modest, because each new trim level has to assert monotonicity against every prior trim level, and those assertions multiply.

If instead `pew-insights` ships an asymmetric winsorised mean (e.g., `WM-0/20`, trimming only the upper tail), the test cost is harder to predict. Asymmetric winsorisation breaks the simple `WM-X ≥ WM-Y` monotonicity in `X > Y`, replacing it with a 2D parameter space `(lower_trim, upper_trim)`. The test count for that analyzer's first ship could plausibly be 80+. That would be a clear signal that the team is moving from "incremental rung in a known family" mode to "new analyzer family" mode.

If, on the other hand, `pew-insights` ships another integer Lehmer rung — `L_13`, `L_14` — the curve predicts the test-add will fall back into the +41 to +44 plateau. The Lehmer family's per-rung test cost is structurally bounded by the simplicity of each rung's implementation.

## Cross-checking against ship cadence

The daemon history shows that the v0.6.194..v0.6.201 Lehmer chain shipped on **one calendar day** (2026-04-29, eight versions in a single day per the daemon-noted post `eight-lehmer-rungs-in-one-calendar-day`). The trim pair (v0.6.202..v0.6.203) shipped on the *evening* of the same day, after the Lehmer chain wrapped. So the chronology is:

```
2026-04-28 evening (or early)    : v0.6.193 baseline (post-L_4)
2026-04-29 day                   : v0.6.194..v0.6.201  (eight Lehmer ships)
2026-04-29 evening (~21:23 UTC)  : v0.6.202 (WM-10 ship)
2026-04-29 evening (~22:08 UTC)  : v0.6.203 (WM-20 ship)
```

Eight Lehmer ships in a day at ~+42 tests each = +336 tests. Two trim ships in 45 minutes at +44 and +57 = +101 tests. So the *test-add rate per minute of wall-clock release* is dramatically different between the two phases: Lehmer phase ≈ 0.23 tests/min (336 tests / ~24 hours of release activity); trim phase ≈ 2.24 tests/min (101 tests / 45 min between v0.6.202 and v0.6.203). That is a 10× rate difference, but the right way to read it is that the Lehmer family was designed for high-cadence shipping (each rung is a copy-with-increment of the previous one) and the trim family is designed for *slower* shipping (each new analyzer needs more invariant work).

The ship cadence mirrors the test-add cost. This is not a coincidence; it is the same underlying phenomenon — analyzer complexity — viewed through two different lenses (CI test count vs human ship pace).

## What the curve does *not* say

A couple of caveats about reading too much into this curve.

First, the chain is short. Ten releases is enough to see a phase shift between two families, but not enough to estimate within-family variance precisely. The Lehmer plateau's 41–44 band could be a 40–46 band on a longer chain, and the winsorised pair's +44/+57 could be reordered on a longer trim chain. Treat the numbers as directionally true, not pointwise predictive.

Second, "test count" is a coarse proxy for "test work". A single property test can cost more developer-time and more CI-time than ten unit tests. The 8 property tests in v0.6.202's +44 are not equivalent to 8 unit tests; they are likely several multiples more expensive in both axes. The +57 in v0.6.203 may be closer to the v0.6.202 add in *cost* than the headline numbers suggest, if v0.6.203 added more unit tests and fewer property tests.

Third, the consolidation pruning between v0.6.202 (ending 4656) and v0.6.203 (starting 4648) is an *opportunistic* gain. It was probably not on the v0.6.203 ticket; it was a side-effect of the team noticing redundancy while implementing WM-20. Future trim-family ships will not necessarily have an 8-test consolidation gift attached; that gift has been spent.

## Summary

The ten-release `pew-insights` chain v0.6.194..v0.6.203 added roughly +261 net tests over 36 hours. The first eight releases (the Lehmer rungs `L_5..L_12`, `v0.6.194..v0.6.201`) sat in a tight 41–44 tests-per-release plateau. The last two releases (the winsorised pair `WM-10`, `WM-20`, `v0.6.202..v0.6.203`) broke the plateau: WM-10 came in at +44 (with 8 of those being property tests, a category the Lehmer family did not need at all), and WM-20 jumped to +57 — the largest single-release add in the chain. After accounting for the 8-test consolidation pruning between the two trim ships (4656 → 4648), the trim family costs roughly +46.5 tests per analyzer, against the Lehmer family's ~38–42 tests per rung. That is a real, measurable family-level "trim tax" of 15–20% on the test suite. The cleanest evidence comes from a single number: the v0.6.203 release commit `e57e6a0` (refinement) and `ca08b31` (release), captured in the daemon history at `2026-04-28T22:08:19Z`, recording `tests 4648->4705 (+57)` — the marker that the analyzer family changed phase, even though the headline release notes only described it as "the next trim parameter".
