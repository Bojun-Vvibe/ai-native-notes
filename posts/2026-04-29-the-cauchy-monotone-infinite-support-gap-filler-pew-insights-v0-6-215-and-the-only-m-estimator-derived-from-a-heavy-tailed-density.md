---
title: The Cauchy monotone / infinite-support gap-filler — pew-insights v0.6.215 and the only M-estimator in the suite derived from a heavy-tailed location-scale density
date: 2026-04-29
---

The pew-insights M-estimator-of-location suite landed across seven
releases. Huber arrived at `0.6.209` (commit `6142b2a`), Tukey
biweight at `0.6.210` (commit `a0bf65a`), Hampel at `0.6.211`,
Andrews sine at `0.6.212`, Welsch at `0.6.213`, Theil-Sen and
related slope work at `0.6.214`, and then Cauchy at `0.6.215`.
Six location-M-estimator lenses in five consecutive minor versions,
all targeting the same per-row `total_tokens` column on the same
real local `~/.config/pew/queue.jsonl`. By `0.6.214` the reasonable
question was whether the suite needed a sixth one. The Cauchy
release answers that with a structural argument, not a tuning-curve
one: there was a hole in the family that none of the previous five
filled, and the hole is exactly the cell `(monotone, infinite
support)`. Cauchy fills that cell uniquely.

This post is about what that cell means, why it was empty, what
shape Cauchy has that the others don't, and what the `0.6.215`
live-smoke run reveals about how a heavy-tailed-density-derived
estimator behaves on a six-source production corpus.

## The 2-by-2 table the suite is implicitly building

The CHANGELOG block at `pew-insights/CHANGELOG.md:137-144` lays out
the six lenses on two axes — class (`monotone` vs `redescender`)
and support (`compact` vs `infinite`) — and the pre-`0.6.215`
state of the table is striking once you read it that way:

| Lens (version) | Class | Support | Tail psi behavior |
|---|---|---|---|
| Huber (v0.6.209, c=1.345) | monotone | infinite | clips to constant `±c` |
| Tukey (v0.6.210, c=4.685) | redescender | **compact** | exactly 0 past `±c` |
| Hampel (v0.6.211, knots 1.7/3.4/8.5) | redescender | **compact** | piecewise-linear, 0 past 8.5 |
| Andrews (v0.6.212, A=1.339) | redescender | **compact** | sine wave, 0 past `±Aπ` |
| Welsch (v0.6.213, c=2.9846) | redescender | infinite | Gaussian decay → 0 |
| **Cauchy (v0.6.215, c=2.3849)** | **monotone** | **infinite** | **decays like c²/z, never 0** |

Read the four cells of the (class × support) cross. Before
`0.6.215`:

- `(monotone, infinite)`: Huber. One occupant.
- `(monotone, compact)`: empty (and rightly empty — a monotone psi
  that hard-clips at `±c` is exactly Huber; if you go past clipping
  by sending psi to zero past `±c` you lose monotonicity by
  definition).
- `(redescender, compact)`: Tukey, Hampel, Andrews. Three occupants.
- `(redescender, infinite)`: Welsch. One occupant.

The redescender row is rich. The monotone row had exactly one
member. And that one member, Huber, has a very specific
pathological behavior at the tail: psi clips to a constant `±c`,
which means a contaminated row with `|z| = 100c` contributes the
same influence to the estimating equation as a clean row with
`|z| = c`. Bounded influence, but not vanishing influence. The
suite had no monotone lens with vanishing tail influence.

Cauchy lands exactly there. Its psi is monotone — it never
increases past a peak, then turns around (so it is not a
redescender by the strict definition that psi changes sign or
returns to zero) — but its tail behavior is `psi_C(z) = z / (1 +
(z/c)^2) ~ c^2 / z` for large `|z|`. Influence vanishes like
`1/z`. Slowly. Heavy-tailed-slowly. But it vanishes, where Huber's
plateaus.

That is the cell Cauchy fills. Not "another redescender". Not "a
softer Huber". A genuinely new cell in the (class × support × tail
behavior) cube, occupied for the first time at `0.6.215`.

## Why this is the only M-estimator in the suite derived from a density

Every M-estimator can be read as a maximum-likelihood estimator
under some assumed error density: `rho(z) = -log f(z)` up to a
constant. The CHANGELOG flags this explicitly at lines 148-152:
the Cauchy weight `w(z) = 1 / (1 + (z/c)^2)` is the un-normalized
standard Cauchy / Lorentzian density at `z/c`, and `rho_C(z) =
(c^2 / 2) * ln(1 + (z/c)^2)` is the negative log of that density.
So the Cauchy M-estimator is the maximum-likelihood location
estimator assuming Cauchy-distributed errors — a heavy-tailed
location-scale family with no finite mean and no finite variance.

None of the previous five lenses come from this construction in any
clean way. Huber's rho is constructed to be the minimax location
estimator in a contamination neighborhood of the normal, not from a
density per se. Tukey biweight, Hampel, Andrews, and Welsch are
designed-for-purpose redescender shapes — their rho functions are
chosen to give specific weight-curve behavior, and you can back out
an implied density only awkwardly (Tukey's `(1 - (z/c)^2)^2` weight
inside compact support corresponds to a density that is exactly
zero past `±c`, which is not a useful generative model of anything
in nature). Welsch's `exp(-(z/c)^2)` weight backs out to a Gaussian
density on the residual, but Welsch is shipped as a redescender
shape, not as "Gaussian MLE" — that would be ordinary least
squares.

Cauchy, in contrast, is what you get if you write down the Cauchy
density, take its log, drop the constant, and solve the score
equation. It is the only lens in the suite that comes from
"assume the residuals follow this real heavy-tailed density"
rather than from "design a weight curve with these robustness
properties." The mechanical consequence is that Cauchy will give
the right answer on data that is actually Cauchy-distributed (no
finite mean, no LLN, no CLT), in a way that none of the others
will.

## The live-smoke table at v0.6.215

The CHANGELOG ships a six-source live-smoke run at
`pew-insights/CHANGELOG.md:182-199`. Reproduced here, sorted by the
`cauchy` column descending, with the gap columns made explicit:

```
source           rows  mean         median      cauchy      mad         scale       iter  conv       core  tail  far  cauchy-mean   cauchy-median
codex            64    12650385.31  7132861.00  9634840.85  6506096.00  9645952.36  17    converged  57    7     0    -3015544.46   +2501979.85
opencode         429   10405853.48  7958860.00  7926913.20  4653113.00  6898715.66  12    converged  393   36    0    -2478940.28      -31946.80
claude-code      299   11512995.95  3319967.00  4331249.77  3103438.00  4601164.06  16    converged  232   53    14   -7181746.18    +1011282.77
openclaw         535   3728319.36   2310411.00  2702181.74  1333458.00  1976987.79  16    converged  475   53     7   -1026137.63     +391770.74
hermes           265   754720.61    432134.00   496311.43   264124.00   391590.83   16    converged  217   47     1    -258409.18      +64177.43
vscode-redacted  333   5662.84      2319.00     2717.65     1736.00     2573.80     15    converged  289   37     7      -2945.20         +398.65
```

Several patterns to call out, because they are direct empirical
fingerprints of the monotone-infinite-support shape:

**Sign of `cauchyMeanGap` is uniformly negative across all six
sources.** The Cauchy estimate sits below the mean on every source.
This is the signature of right-tail contamination: every one of the
six sources has a heavy upper tail that drags the mean above the
robust center. `claude-code` shows the largest absolute gap at
`-7,181,746` tokens (the mean is 11.5M, the Cauchy estimate is
4.3M); `vscode-redacted` shows the smallest at `-2,945` tokens but
on a much smaller scale. The sign uniformity is the same six-of-six
strict pattern that Tukey-minus-Huber showed at `0.6.210` versus
`0.6.209`. It is not a coincidence — it is the right tail of the
fleet's per-row token distribution showing up in every robust lens
in the same direction.

**Sign of `cauchyMedianGap` is mixed.** Five of six sources are
positive (Cauchy lands above the median), one (`opencode`) is
slightly negative (`-31,946` on a median of 7.96M, a 0.4 %
deviation). This is the diagnostic for whether a source's
distribution is roughly symmetric around its median. `opencode`
is the most-symmetric source by this measure — the Cauchy
estimate is essentially indistinguishable from the median.
`claude-code` is the least-symmetric: Cauchy lands `+1,011,282`
above the median, meaning the inter-median region is asymmetric
and the M-estimator is being pulled up toward the heavier upper
side even after robust downweighting.

**`farTailRows` count is bounded but never zero on
`claude-code`.** The three-bucket partition (`coreRows`,
`tailRows`, `farTailRows`) keyed on Cauchy weight magnitude shows
how many rows the Cauchy weight pushes below `w = 0.05`.
`claude-code` puts 14 rows in that far-tail bucket — the largest
count in the cohort. But "far tail" under Cauchy still means
strictly positive weight: `psi_C` never reaches zero at any finite
`z`. The CHANGELOG flags this at line 178-180: the `zero-weight`
termination branch is reachable defensively but mathematically
impossible for Cauchy at finite `z`, because `1 + (z/c)²` is
always `≥ 1`, so the denominator never hits zero. Compare to
Welsch at `0.6.213`, where Gaussian-fast decay can underflow IEEE
double precision and cause real `w = 0` rows.

**Convergence is fast and uniform.** Every source converges in
`≤ 17` IRLS iterations (codex 17, opencode 12, claude-code 16,
openclaw 16, hermes 16, vscode-redacted 15). No `max-iter`
terminations, no `zero-weight` terminations. The 1,925 total rows
across six sources, zero rows dropped, all converged — this is
what the CHANGELOG quietly reports at line 190 with
`dropped: 0 bad hour_start, 0 bad total_tokens, 0 negative
total_tokens, 0 by source filter, 0 below min-rows, 0 below
min-cauchy, 0 below top cap`. A clean run.

## What the three-bucket partition is keyed on, and why

Tukey/Hampel/Andrews use compact support, so their natural
three-bucket partition is keyed on whether a row's residual is
inside the "fully weighted" inner region, in the "tapering" middle
region, or past the cutoff (where `w = 0` exactly). Welsch uses
infinite support but the bucketing in `0.6.213` was keyed on
weight thresholds because there is no compact-support cutoff to
work with.

Cauchy inherits Welsch's problem: there is no `w = 0` cutoff to
bucket on. So `0.6.215` keeps the weight-magnitude convention but
ties the thresholds to physically meaningful points on the
Lorentzian:

- `coreRows` is `w ≥ 0.5`, equivalently `|z| ≤ c`. This is the
  half-power knee of the Lorentzian — exactly the point where the
  weight equals half its peak value of 1.0.
- `tailRows` is `0.05 ≤ w < 0.5`, equivalently
  `c < |z| ≤ c·√19 ≈ 4.36c`.
- `farTailRows` is `w < 0.05`, equivalently `|z| > 4.36c`.

The arithmetic for the tail boundary is just inverting the weight
function: `w = 1 / (1 + (z/c)²) = 0.05` gives `(z/c)² = 19`, so
`|z| = c·√19 ≈ 4.36c`. With `c = 2.3849` and the typical
per-source `scale ≈ MAD/0.6745`, "far-tail" means a row whose
residual is more than about `4.36 · 2.3849 · scale ≈ 10.4 · scale`
away from `mu`. On `claude-code`, `scale = 4,601,164.06`, so the
far-tail boundary sits at residuals of about `47.9 M` tokens away
from the Cauchy center of `4.3 M` — i.e., rows with
`total_tokens` either above ~52 M or below ~ -43 M (the latter
impossible for a token count, so all 14 far-tail rows on
`claude-code` are extreme-large). That is consistent with the
heavy upper-tail signature already visible in `cauchyMeanGap`.

## What this lens lets you say that the others can't

Concretely, the six location lenses now answer different
questions:

- Huber: "What is the most plausible center if I trust everything
  inside `±1.345·scale` and clip everything outside?"
- Tukey: "What is the most plausible center if I treat anything
  past `±4.685·scale` as not data?"
- Hampel: "What is the center under a piecewise-linear soft-then-
  hard cutoff?"
- Andrews: "What is the center under a smooth sinusoidal taper
  with compact support?"
- Welsch: "What is the center if every row contributes Gaussian-
  decaying weight with no hard cutoff?"
- Cauchy: "What is the center if I assume the residuals are
  literally Cauchy-distributed?"

The Cauchy answer is qualitatively different from the others in
two ways. First, it is an MLE under a real-named heavy-tailed
distribution, not a designed-for-purpose weight curve. Second, it
gives every row strictly positive weight, however small, which
means no row is ever "thrown away" — including the row that the
others might be silently zero-weighting. On `claude-code`, the 14
`farTailRows` are still in the estimating equation under Cauchy;
under Tukey or Andrews, rows past the compact-support cutoff are
mathematically removed and the estimate is computed over the
surviving subset.

This matters operationally because "is this row in the model"
becomes a continuous question under Cauchy and a discrete question
under the redescenders with compact support. If you are
diagnosing why two location estimates differ on the same source,
the Cauchy estimate gives you one less branch to chase: there is
no row-rejection step, so any difference is a difference in shape
function, not a difference in which rows participated.

## Closing observation: the fleet's right-tail signature is
remarkably stable

Across the seven location-M-estimator releases (`0.6.209` through
`0.6.215`), the same fact keeps showing up in every live-smoke
run: `<lens>MeanGap < 0` for all six sources, every time. Huber
showed it. Tukey showed it more strongly. Hampel, Andrews,
Welsch, and now Cauchy all show it. The right-tail-contamination
fingerprint of the production queue is so strong that it survives
every robust shape function the suite throws at it. The mean is
above the robust center on every source, every release. Cauchy at
`0.6.215` is the latest data point in a now-very-consistent series
— and because Cauchy is the gentlest of the six (monotone,
infinite support, slowest tail decay), the magnitude of
`cauchyMeanGap` is generally the smallest of the six lenses on
the same source. Tukey's gap is bigger because Tukey throws rows
out; Cauchy's gap is smaller because Cauchy keeps everything but
discounts it slowly. Both agree on the sign. That sign agreement,
six lenses by six sources, 36 cells, all negative, is the most
robust empirical claim the suite has produced so far.

The release notes for `0.6.215` describe the lens as filling a
hole in the M-estimator zoo. The live-smoke confirms the
mechanical claim — monotone, infinite support, vanishing-but-
never-zero tail — and adds the empirical confirmation that the
fleet's heavy-upper-tail signature is reproducible across all six
shapes the suite now ships. The hole is filled; the picture is
consistent; the next obvious lens is no longer in the
location-M-estimator family but in the slope/trend family, which
is exactly where `0.6.214` and `0.6.216` go.
