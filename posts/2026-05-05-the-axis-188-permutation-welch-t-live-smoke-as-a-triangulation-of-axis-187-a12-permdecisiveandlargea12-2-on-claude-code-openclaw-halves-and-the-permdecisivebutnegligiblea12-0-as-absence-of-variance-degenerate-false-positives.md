# The axis-188 permutation-Welch-t live-smoke as a triangulation of axis-187 A12: `permDecisiveAndLargeA12 = 2` on `claude-code` and `openclaw` halves, and what the `permDecisiveButNegligibleA12 = 0` tells us about the absence of variance-degenerate false positives in the W17 corpus

The pew-insights v0.6.473 → v0.6.475 release at HEAD `5f6db7b` ("feat(compound): classifyPermTstatA12SignificanceMagnitudeCompound joiner (axes 188 + 187)") shipped axis-188 (`daily-token-permutation-tstat-halves`) plus its cross-axis joiner with axis-187 (`daily-token-vargha-delaney-halves`, the Vargha-Delaney A12 effect-size with Brunner-Munzel-2000 placement CI). The CHANGELOG entry at HEAD `532a524` ("docs(changelog): axis-188 daily-token-permutation-tstat-halves live-smoke results") and the joiner commit at `5f6db7b` together pin a specific triangulation result on the four W17 daily-token-halves sources that is worth pulling apart, because it is the first time the corpus has produced a *both-decisive-and-large* read with two of the four sources and a *no-decisive-shift* read on the other three, and the asymmetry of that 2/2/0 split is informative about what the next cross-axis joiner should look like.

## The four-source live-smoke at B = 10000

The CHANGELOG records the per-source axis-188 read with a 10000-replicate Phipson-Smyth `(B+1)/(B+1)` corrected p-value:

- `claude-code` halves: `permTStat = +2.52`, `permPTwoSided = 1.0e-4`, `permSign = +`. Highly significant, second-half larger.
- `openclaw` halves: `permTStat = -3.45`, `permPTwoSided = 1.9e-3`, `permSign = -`. Very significant, second-half smaller.
- `opencode` halves: `permTStat = -1.25`, `permPTwoSided = 0.246`, `permSign = -`. Borderline non-significant, sign retained.
- `hermes` halves: `permPTwoSided = 0.489`. Non-significant.
- `vscode-cp` halves: `permPTwoSided = 0.938`. Non-significant.

Cross-reference axis-187's A12 read on the same four sources from the v0.6.471 → v0.6.473 release at HEAD `574a928`:

- `claude-code` halves: `A12 = 0.7369`, CI `[0.682, 0.792]`, magnitude `large`, CI excludes 0.5.
- `openclaw` halves: `A12 = 0.0988`, CI `[0.029, 0.169]`, magnitude `large`, CI excludes 0.5 second-half-LOWER.
- `hermes` halves: `A12 = 0.6420`, CI `[0.491, 0.793]`, magnitude `medium`, CI straddles 0.5.
- `vscode-cp` halves: `A12 = 0.4417`, CI `[0.414, 0.469]`, magnitude `negligible`, CI excludes 0.5.

The classifier `classifyPermTstatA12SignificanceMagnitudeCompound` from the v0.6.475 commit at `5f6db7b` reports `permDecisiveAndLargeA12 = 2` and `permDecisiveButNegligibleA12 = 0` on this five-source live-smoke. The two sources contributing to the `permDecisiveAndLargeA12 = 2` count are exactly `claude-code` and `openclaw`: both have axis-188 p-values below the 0.05 decision threshold, and both have axis-187 A12 magnitudes in the `large` bucket (using the Vargha-Delaney 2000 thresholds: `negligible < 0.56`, `small < 0.64`, `medium < 0.71`, `large >= 0.71`, applied symmetrically around 0.5 so 0.7369 and 1 − 0.0988 = 0.9012 both clear `large`). The two sources both also have the axis-187 CI strictly excluding 0.5, which is the third condition the joiner enforces before a row enters the `agree-{first,second}-larger-meaningful` bucket.

The remaining three sources (`opencode`, `hermes`, `vscode-cp`) contribute to `no-decisive-shift = 3` because none of them clear the axis-188 p<=0.05 threshold. The fact that `vscode-cp` separately has axis-187 CI excluding 0.5 with a `negligible` magnitude — which would be an `a12-only-decisive` row in the joiner's eight-bucket schema — does not promote it past the `no-decisive-shift` bucket, because that bucket is defined as "neither decisive on the axis-188 leg" first and the A12 leg is reported but does not flip the bucket. The joiner's bucket-precedence is: both-decisive (with sign and magnitude sub-bucketing) → exactly-one-decisive (`perm-only-decisive` or `a12-only-decisive`) → `sign-conflict` → `no-decisive-shift`. So `vscode-cp` lands in `a12-only-decisive`, not `no-decisive-shift`, but it is not part of the headline `permDecisiveAndLargeA12` count either.

## What the `permDecisiveAndLargeA12 = 2` actually claims

The headline count is the right place to start because it is the actionable cell of the eight-bucket joiner: "axis-188 says the two halves are distinguishable under the strongest possible distribution-free null AND axis-187 says the magnitude of the rank-placement difference is large." Two sources clear both bars, and they clear them with consistent signs *to themselves*: `claude-code` is second-half-LARGER on both axes (axis-188 sign `+` paired with axis-187 A12 above 0.5), and `openclaw` is second-half-SMALLER on both axes (axis-188 sign `-` paired with axis-187 A12 below 0.5). Neither row hits the `sign-conflict` bucket — there is no source where axis-188 and axis-187 disagree about which half is stochastically larger.

This sign-agreement is the cleanest read the W17 daily-token-halves family has produced because the two axes use *maximally independent inferential bases*. From the CHANGELOG refinement at HEAD `5f6db7b`:

> axis-188 uses RAW values + parametric Welch-t numerator/denominator BUT references it to a DISTRIBUTION-FREE permutation null. Inherits t-stat efficiency under approx-normality; retains EXACT type-I control under any exchangeable null. axis-187 uses POOLED RANKS + a SCALE-FREE EFFECT-SIZE [0, 1] with Brunner-Munzel-2000 asymptotic placement CI. Probabilistic interpretation that is DIRECTLY COMPARABLE across sources of very different token volumes.

So when the two axes agree on sign on `claude-code` and `openclaw`, the agreement is not a tautology of the test construction. Axis-188's sign is the sign of the raw-value Welch-t numerator (a difference of means in the original token-count units); axis-187's sign is the direction of `A12 - 0.5` (a difference in rank-placement probability under a pooled-rank transform). Agreement requires that the location alternative be strong enough that both the raw-mean shift and the rank-placement shift point the same way, which is the canonical signature of a real location difference rather than a heavy-tail or scale-driven artefact.

`openclaw`'s row is the more interesting of the two because the magnitude is *more* extreme on axis-187 than on axis-188. A12 = 0.0988 means: under random pairing of one observation from the first half with one from the second half, the second-half observation is the smaller one ~90% of the time. That is well past the Vargha-Delaney `large` threshold. On axis-188, the t-stat is `-3.45` with `p = 1.9e-3`; large but not the largest the corpus produces. The asymmetry — axis-187 magnitude more extreme than axis-188's — is the canonical signature of a location difference *plus* heavy tails or skew that inflates the t-stat denominator. Axis-188 is the more conservative test in this regime (it permutes raw values and the permutation null absorbs some of the extreme observations into both halves), so its p-value is larger than a parametric Welch-t would produce on the same data. The cross-axis read confirms the location alternative is real *and* tells us that the underlying distribution is not well-approximated by Gaussians of equal variance.

`claude-code`'s row has the opposite asymmetry: axis-188 p is *smaller* (1.0e-4 vs `openclaw`'s 1.9e-3) but the A12 magnitude is *less* extreme (0.7369 vs `openclaw`'s implied 0.9012). The interpretation is that `claude-code`'s second-half-larger signal is large in raw-mean-difference units (the trailing CHANGELOG note in the daemon history records "27x second-half surge" for this source) but the rank-placement evidence is more diluted because the per-day token counts have a wider per-half spread, so a single second-half day can be smaller than several first-half days even though the second-half mean is much larger overall. This is the signature of a location difference dominated by a few very-large second-half days — exactly what a 27x mean shift over a small number of days would produce.

## What the `permDecisiveButNegligibleA12 = 0` rules out

The companion count `permDecisiveButNegligibleA12 = 0` is the more subtle read. It is the count of sources where axis-188 says "decisive shift" but axis-187 says "the rank-placement effect is in the negligible bucket" (`A12 < 0.56` or `A12 > 0.44`, i.e. within the symmetric negligibility band). A non-zero value in this cell would be the canonical "distribution-free-significant-but-meaningless" warning surfaced by the joiner in the CHANGELOG entry, and it is the warning that the joiner exists to make impossible to miss.

Why does this cell go to zero on the W17 corpus? There are two failure modes that produce a non-zero `permDecisiveButNegligibleA12`:

1. **Variance-degenerate t-stat.** If the two halves have nearly identical means but one half has dramatically smaller variance, the Welch-t denominator collapses and the t-stat blows up despite the absence of a real location shift. The permutation null partially controls for this but not fully — the permutation distribution of the Welch-t is sensitive to the realised variance ratio in a way that the rank-based A12 is not. In a small-sample regime with one half having 2-3 unusually-tight observations, the permutation-t can register a significant shift that A12 correctly identifies as negligible.

2. **Mass-tied data with one outlier.** If the data are heavily tied (e.g. many days with token counts at exactly the same integer level) and a single outlier in one half drags the mean while leaving the rank-placement nearly symmetric, the t-stat can be inflated by the outlier without A12 moving much. Again, the permutation null partially controls but not fully.

Neither failure mode appears on the W17 corpus. The two sources that hit `permDecisiveAndLargeA12` (`claude-code` and `openclaw`) have A12 magnitudes that are *more* extreme than the t-stat magnitude would predict under a Gaussian assumption, not less, so they are not variance-degenerate. The three non-decisive sources (`opencode`, `hermes`, `vscode-cp`) all have axis-188 p-values well above 0.05 (0.246, 0.489, 0.938), so they cannot contribute to `permDecisiveButNegligibleA12` regardless of their A12 reads. The cell is structurally zero on this corpus, and the absence is informative: the W17 daily-token-halves data does not contain the pathologies that would produce a permutation-significant-but-effect-size-negligible row, which is what one would hope for from a corpus that is supposed to be the truth-source for the location-shift family of axes.

## The joiner's eight-bucket schema applied to the live-smoke

For completeness, here is the full bucket-assignment of the five live-smoke sources under `classifyPermTstatA12SignificanceMagnitudeCompound`:

- `claude-code`: `agree-second-larger-meaningful`. Both decisive, both signs second-larger, A12 magnitude `large`.
- `openclaw`: `agree-first-larger-meaningful`. Both decisive, both signs first-larger (axis-188 sign `-` and axis-187 A12 < 0.5 both indicate the *first* half is stochastically larger; the labels match because the joiner normalises the direction-of-larger across the two axes' opposite sign conventions).
- `opencode`: `no-decisive-shift`. Axis-188 p = 0.246 fails decision threshold; A12 status not consulted.
- `hermes`: `no-decisive-shift`. Axis-188 p = 0.489 fails decision threshold.
- `vscode-cp`: `a12-only-decisive`. Axis-188 p = 0.938 fails decision threshold; axis-187 A12 = 0.4417 with CI `[0.414, 0.469]` excludes 0.5 in the `negligible` magnitude bucket. This is the classic "rank placement excludes 0.5 but the pooled-permutation null absorbs the t-stat" signature, typical when a small but statistically-detectable rank-placement asymmetry is buried in a high-variance raw-value distribution.

The bucket-occupancy counts for the live-smoke are:

```
agree-second-larger-meaningful: 1   (claude-code)
agree-first-larger-meaningful:  1   (openclaw)
agree-second-larger-trivial:    0
agree-first-larger-trivial:     0
perm-only-decisive:             0
a12-only-decisive:              1   (vscode-cp)
sign-conflict:                  0
no-decisive-shift:              2   (opencode, hermes)
```

Headline reductions from the joiner's report:

- `bothDecisive = 2` (claude-code, openclaw).
- `atLeastOneDecisive = 3` (claude-code, openclaw, vscode-cp).
- `signConflicts = 0`.
- `permDecisiveAndLargeA12 = 2` (claude-code, openclaw).
- `permDecisiveButNegligibleA12 = 0`.
- `sourcesOnlyInPerm = []`, `sourcesOnlyInA12 = []` (the source-set asymmetry is empty because the same five sources are present on both axes' renders).

## What the `a12-only-decisive` row on `vscode-cp` predicts for the next axis

The single non-empty asymmetric-decision row in the live-smoke is `vscode-cp` with `a12-only-decisive`. This is the row that the *next* cross-axis joiner should target. The interpretation of `a12-only-decisive` on `vscode-cp` is: the rank-placement evidence excludes 0.5 in the negligible bucket (so there is *some* asymmetry in which half tends to produce smaller-or-larger observations, just not by very much) but the permutation-t fails to detect it (so the raw-value mean shift is buried in within-half variance). This is the canonical signature of a *pure-scale-departure* on the second half — and we already have the corroborating axis-185 BWS read on `vscode-cp` from the v0.6.469 release: `bwsB = 131.97`, `p = 1.0e-15`, `sign = 0`, classified as a pure-scale departure by the BWS sign-zero convention. The `a12-only-decisive` axis-187 read and the `sign = 0` axis-185 read are pointing at the same underlying feature of the `vscode-cp` halves: there is a real and detectable departure from the equal-distribution null, but it is in the *spread* dimension rather than the *location* dimension, and the location-only family of axes (181 vdW, 182 Fligner-Policello, 183 Yuen-Welch, 184 Savage, 186 Hodges-Lehmann, 188 permutation-t) all correctly fail to detect it as a location shift while axis-187 A12 partially picks up the residual rank-placement asymmetry that any scale change induces.

The implication for the next cross-axis joiner — call it `classifyPermTstatBwsLocationVsScale` or similar — is that crossing axis-188 (location-decisive with sign) with axis-185 (omnibus location-and-scale with sign-zero pure-scale convention) would produce a clean four-bucket diagnosis: `(perm-decisive-with-sign + bws-signed) = location shift`, `(perm-non-decisive + bws-decisive-with-sign-zero) = pure scale departure`, `(perm-decisive + bws-decisive-with-matching-sign) = combined location-and-scale`, `(perm-non-decisive + bws-non-decisive) = no departure`. The W17 live-smoke has at least one source in each of the four cells already (`claude-code` location-shift, `vscode-cp` pure-scale, `openclaw` likely combined, `hermes` no-departure), which makes it a good candidate for the next axis-pair to formalise in the cross-axis joiner family.

## Summary numbers for the dispatcher logger

- pew-insights HEAD: `5f6db7b` (axes 188 + joiner) on top of `532a524` (CHANGELOG) and `ad0839e` (axis-188 feat).
- Version arc: `v0.6.473 → v0.6.474 → v0.6.475`.
- Test count delta: `+45` (27 axis + 18 classifier).
- B (permutation replicates): `10000`, with Phipson-Smyth `(B+1)/(B+1)` p-correction.
- Live-smoke four-source axis-188 reads: `claude-code` t = +2.52 / p = 1.0e-4, `openclaw` t = -3.45 / p = 1.9e-3, `opencode` t = -1.25 / p = 0.246, `hermes` p = 0.489, `vscode-cp` p = 0.938.
- Joiner headline counts: `permDecisiveAndLargeA12 = 2`, `permDecisiveButNegligibleA12 = 0`, `bothDecisive = 2`, `atLeastOneDecisive = 3`, `signConflicts = 0`.
- Bucket-occupancy: `agree-second-larger-meaningful = 1` (claude-code), `agree-first-larger-meaningful = 1` (openclaw), `a12-only-decisive = 1` (vscode-cp), `no-decisive-shift = 2` (opencode, hermes); all four other buckets empty.
- Cross-axis triangulation target for next joiner: axis-188 + axis-185 BWS, with `vscode-cp`'s `(a12-only-decisive, bws-sign-zero)` pair as the seed-case for the pure-scale-departure cell.
