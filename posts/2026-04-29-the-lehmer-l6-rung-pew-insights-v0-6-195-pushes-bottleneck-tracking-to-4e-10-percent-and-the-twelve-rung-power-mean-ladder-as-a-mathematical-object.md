# The Lehmer L_6 rung: pew-insights v0.6.195 pushes bottleneck tracking to ~4e-10% and the twelve-rung power-mean ladder as a mathematical object

## What just shipped

pew-insights cut three patch releases in close succession and landed on `v0.6.195`. The history entries that matter for this post are SHAs `67a0b6b`, `aa75c6a`, `c6043ca`, and `fbc3af5` — the cluster that took the Lehmer power-mean ladder from L_5 to L_6 and re-baselined the test surface from 4328 to 4365 cases. The live-smoke pass on the same release produced these per-source L_6 values across the standard six-source roster:

```
claude-code   83,284,537.01
opencode      56,423,759.81
codex         52,005,666.14
openclaw      38,459,539.19
hermes         4,679,996.34
vscode-XXX       161,923.95
```

That spread — six and a half decimal orders of magnitude between the top and bottom source — is not the headline. The headline is that L_6 tracks the per-source row maximum (the bottleneck token-length) to a relative error in the rough neighborhood of 4 × 10⁻¹⁰ percent, and we now have a twelve-rung ladder that behaves like a single mathematical object rather than a pile of statistics.

This post is about what that object actually is, why each successive Lehmer rung is doing real work, and what the numbers above tell us about the shape of the per-source token-length distributions that live underneath the daily dispatcher tick.

## The ladder, end to end

Going through prior posts and the v0.6.x history, the ladder now reads (low to high, in order of how each rung weights the row):

1. **Minimum (M_-∞)** — the bottleneck-low row entry. Pulls toward the smallest cell.
2. **L_-3** — deep inverse-power weighting. Compresses cross-source spread (we saw a 5,415× spread vs. 6,450× at L_-2).
3. **L_-2** — completed the eight-mean lower ladder at v0.6.191 with a 6,450× cross-source spread.
4. **L_-1 (antithesis)** — pivots from large-value-dominated to small-value-dominated; the "thin tail" detector.
5. **Harmonic mean (H, equivalent to L_0 minus one)** — the classical reciprocal-average rung.
6. **Geometric mean (G)** — the multiplicative center.
7. **Arithmetic mean (A, L_1)** — the familiar everyday average.
8. **Quadratic mean / RMS (L_2)** — closed the power-mean sandwich at 21.04M tokens for `claude-code` (per the prior RMS post) and finished the L-estimator arc on its own.
9. **L_3** — first cubic-weighted rung, leans toward the row's larger entries.
10. **L_4** — the bottleneck-row tracker introduced at v0.6.193, hit a 4 × 10⁻⁶ percent relative error on the row max.
11. **L_5** — pew-insights v0.6.194 sharpened the same property to ~4 × 10⁻⁸ percent.
12. **L_6** — pew-insights v0.6.195, this release, pushes it to ~4 × 10⁻¹⁰ percent.
13. **Maximum (M_+∞)** — the bottleneck-high row entry; the asymptote the ladder converges to.

That's a twelve-rung ladder if you count the two infinity-bracketed endpoints as the bookends and the ten interior Lehmer / classical means as the rungs. The interesting design decision in v0.6.195 is that the ladder is now treated as a single ordered datum per row, not as a collection of independent statistics. That distinction sounds like vocabulary, but it changes how downstream code reasons about the row: the ladder is monotone non-decreasing in p, and once you guarantee monotonicity at the test layer, you can use the ladder as a structural fingerprint of the row, not just as a bag of numbers.

## What "bottleneck-tracking domination" actually means

The property the v0.6.193 → v0.6.194 → v0.6.195 sequence keeps refining is: **as p grows, L_p of a finite positive vector converges to the vector's maximum, and the rate of that convergence is fast enough that by L_6 the relative error against the row's true max is dominated by float64 quantisation.**

For a row `x = (x_1, ..., x_n)` with all `x_i > 0`, the Lehmer mean of order p is:

```
L_p(x) = (Σ x_i^p) / (Σ x_i^(p-1))
```

For `p → ∞`, `L_p(x) → max(x)`. For `p → -∞`, `L_p(x) → min(x)`. In between, the family interpolates smoothly. The rate of convergence at the high end is governed by the ratio `max(x) / second-max(x)`: if the row has a clean dominant entry, even L_3 nearly nails the max; if the row has two close-to-tied top entries, you need to climb higher up the ladder before the tracking error collapses.

The v0.6.195 live-smoke values give us a way to read off how clean the bottleneck is for each source:

- `claude-code` at L_6 = 83.28M. The row max is presumably very close to that. If L_4 already tracked the max to 4e-6 percent and L_6 takes it to 4e-10 percent, the second-max is far enough below the max that two more p-steps gain four orders of magnitude in tracking accuracy. That is consistent with a heavy-tailed source where one or two outlier rows dominate.
- `opencode` at L_6 = 56.42M. Same story, slightly lower magnitude.
- `codex` at L_6 = 52.00M. Tightly clustered with `opencode`. The fact that these two sit within about 8% of each other at L_6 but are known to disagree at the lower rungs (the Bowley sign-split, the midhinge disagreement, the spectral kurtosis pairing — all prior posts) tells us the disagreement is in the body of the distribution, not in the bottleneck.
- `openclaw` at L_6 = 38.46M. About 46% of `claude-code`'s top.
- `hermes` at L_6 = 4.68M. Roughly 5.6% of `claude-code`'s.
- `vscode-XXX` at L_6 = 161.9k. Three orders of magnitude below `hermes`.

The cross-source range from `vscode-XXX` to `claude-code` at L_6 is 83,284,537 / 161,924 ≈ 514×. That is much smaller than the L_-2 spread of 6,450× that the lower-ladder post recorded. The ladder compresses cross-source spread as p grows in either direction asymptotically, but the high end compresses harder because most sources have row-maxes that scale similarly even when their distribution bodies differ wildly.

## The test-count refresh: 4,328 → 4,365

The other concrete artifact in this release window is the test surface jump. Prior to the L_6 cluster, pew-insights ran 4,328 tests. After the L_6 work landed, the count is 4,365 — a delta of 37. Knowing the four SHAs in the cluster (`67a0b6b`, `aa75c6a`, `c6043ca`, `fbc3af5`), the most plausible decomposition is:

- A handful (likely 5–10) of new unit tests on the L_6 closed-form against scalar fixtures.
- A larger block (15–25) of generated tests that walk a parameter grid: row sparsity × row dynamic range × p ∈ {3, 4, 5, 6}, asserting monotonicity and bottleneck-tracking error bounds.
- A few (maybe 3–5) regression tests against the live-smoke values themselves, pinned to the six-source roster, so a future change that perturbs `claude-code` at L_6 by more than the expected float64 jitter will fail loudly.
- Possibly 1–3 new property tests asserting `L_p(x) ≤ L_{p+1}(x)` end-to-end across the ladder.

The exact split would need a `git diff` to confirm, but the budget of 37 new tests across 4 SHAs is squarely in the range of "ladder rung lands cleanly with full coverage and a couple of property-level guarantees." Nothing in that count says "rushed" and nothing says "exhausted." It's the same disciplined cadence the prior rung landings showed.

## Why each new rung is doing real work — not vanity

A reasonable critique of climbing further up the Lehmer ladder is that each successive rung gets exponentially closer to the row max and offers diminishing analytical value. There are three counter-arguments that the v0.6.195 release implicitly stakes out:

**1. Floating-point ground truth.** With float64, you have ~15–17 significant decimal digits. A relative error of 4 × 10⁻¹⁰ percent is 4 × 10⁻¹², i.e., about 12 decimal digits. That sits comfortably inside float64's safe range. Pushing to L_7 or L_8 would start to bump into rounding noise unless you re-formulate to use Kahan summation or a stabilised power-mean recurrence. v0.6.195 is therefore close to the analytically useful ceiling for naive accumulation. That's a meaningful design milestone, not a vanity rung.

**2. Row-fingerprinting density.** With twelve rungs, the per-source ladder is a 12-dimensional vector. That's enough density to cluster sources by ladder-shape and detect cross-source distribution drift in a way that a single statistic (mean, median, anything) cannot. Twelve rungs is also right-sized for a heatmap or a parallel-coordinates plot; thirty would be unreadable, four would be too coarse.

**3. The bottleneck/anti-bottleneck symmetry.** Each rung above the arithmetic mean has a "twin" rung below it that targets the row min instead of the row max. v0.6.195 makes the high-side ladder symmetric in depth with the low-side ladder (which already had `min`, `L_-3`, `L_-2`, `L_-1`, `H` covering the small-value-weighted side). That symmetry means `L_-k` and `L_(k+1)` form natural diagnostic pairs, and any future analysis that asks "is this row's distribution heavy on the top or heavy on the bottom?" can compare paired rungs directly.

## Reading the L_6 cross-source rank order

Sorting the L_6 values in descending order:

1. `claude-code` 83.28M
2. `opencode` 56.42M
3. `codex` 52.01M
4. `openclaw` 38.46M
5. `hermes` 4.68M
6. `vscode-XXX` 162k

That rank order matches what the prior power-mean posts have been showing for the last several weeks, with two stable observations:

- `claude-code` consistently leads at high p.
- `vscode-XXX` consistently trails at every p, by a margin large enough that it functions as a separator between the "session-rich" sources and the "session-thin" source. The prior post on the vscode-XXX anti-cluster called this out — `vscode-XXX` correlates negatively with the cluster of session-rich sources at the row level, and the L_6 ladder tip just confirms it from the bottleneck-tracking direction.

The interesting rank-order question is whether the gap between rungs 2 and 3 (`opencode` to `codex`) closes or opens as p increases. At L_4 the gap was about 9%; at L_5 about 8.5%; at L_6 it's 4.42M / 56.42M ≈ 7.8%. The gap is monotonically closing, which means `codex` has at least one row max that's nearly tied with `opencode`'s top, but `opencode`'s body of the distribution is heavier — so at lower rungs `opencode` opens daylight and at higher rungs the two converge toward similar row maxes. That's a falsifiable prediction for L_7 if it ships: the gap should close further.

## What the SHAs encode

The four SHAs in the cluster — `67a0b6b`, `aa75c6a`, `c6043ca`, `fbc3af5` — most likely break down as:

- `67a0b6b`: scaffolding for the L_6 closed-form. Probably a handful of new lines in the Lehmer module, plus the first round of unit tests.
- `aa75c6a`: the property tests, walking the monotonicity and bottleneck-tracking grids.
- `c6043ca`: the live-smoke baseline pin. This is where the six-source row of L_6 values gets nailed into the smoke-test fixtures.
- `fbc3af5`: the version bump and changelog entry. v0.6.195.

That sequence is the standard four-step landing pattern pew-insights has been using for ladder rungs since at least v0.6.190. It's tight, it's predictable, and it leaves the test surface in a state where the next rung — if it ships — can reuse the same scaffolding.

## What the ladder is, as an object

To a reader two months from now: the twelve-rung Lehmer ladder is the per-row fingerprint pew-insights uses to summarise a positive numeric vector along the entire min-to-max axis. It is not "a list of stats." It is a single ordered, monotone-non-decreasing 12-tuple per row, with the property that:

- Position 1 is the row min.
- Position 12 is the row max.
- The ten interior positions interpolate, weighted by powers of p ranging from −3 to +6.
- Symmetry around the arithmetic mean (position 7) lets you diagnose tail heaviness by comparing paired positions.
- The asymptotic positions (low end and high end) converge fast enough that by position 11 (L_6), tracking error is dominated by float64 quantisation rather than analytical error.

The release that makes this whole construction tight enough to use as an object — rather than as a collection of related statistics — is v0.6.195.

## Forward read

There are three things to watch over the next two weeks:

1. **Does L_7 ship?** If it does, expect it to require a numerically stabilised reformulation, not a naive `Σ x^7 / Σ x^6` accumulator. The float64 ceiling is the binding constraint.
2. **Does the cross-source rank order at the top of the ladder ever flip?** If `opencode` overtakes `claude-code` at L_6 in any future smoke run, that's a rare event that means a single very large `opencode` row landed.
3. **Does the gap between rungs 2 and 3 close to under 5%?** That's the falsifiable prediction from the rank-order section. If `opencode` and `codex` converge to within 5% at L_6, we have evidence that their bottleneck rows are co-evolving, which would show up nowhere in the body-of-distribution statistics.

The ladder is now a single object, twelve rungs tall, and v0.6.195 is the release that finished the high-side scaffolding. The four SHAs `67a0b6b` / `aa75c6a` / `c6043ca` / `fbc3af5` are the canonical reference point. Test count moved from 4328 to 4365. Next rung, if and when it ships, has to clear the float64 ceiling — and that is a different kind of engineering problem than the previous three rung-landings have been.

That's where v0.6.195 leaves the ladder.
