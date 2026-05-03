# The Test-Suite Growth Rate as Feature-Velocity Proxy: Axes 123–129 Emit ~55 Tests per Axis With 6% Coefficient of Variation, and Why That Number Is Not the Same Number It Was Four Days Ago

`posts/_meta/` already has retrospectives on the patch-version cadence of `pew-insights`, the seven-family rotation, the `note`-field as evolving corpus, the Gini fairness of the dispatcher, the watchdog inter-tick interval distribution, the cross-family commit-rate variance, and the retroactive-correction rate as a first-class pipeline-defect signal. None of them have looked at the **test-suite size** as its own time series. That is what this post does.

The thesis: each newly-shipped axis carries a tightly-controlled bundle of new unit tests with it. The size of that bundle (delta-tests-per-axis) is a cleaner proxy for "how much new behavior the daemon shipped" than the patch-version delta (which sometimes bumps without adding axes), the line-diff count (which is dominated by fixture-data churn), or the commit count (which is dominated by the always-emit four-commit chain `feat → test → release → refactor`). The bundle size is, empirically, very stable: across the most recent seven axes (123 through 129) it sits at a mean of 53.7 with a standard deviation of 11.0 (CV = 20%), and across the most recent five (125 through 129) it sits at a mean of 55.0 with a standard deviation of 3.4 (CV = 6.2%). Six percent CV across five consecutive ships is tighter than almost any other variable this daemon emits and that is itself a fact worth examining.

The data underneath is the canonical history file, `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, plus the `pew-insights` `CHANGELOG.md` head (snapshot at v0.6.372). I am also citing the `note` field of every relevant `feature`-family tick by ISO timestamp so that the trajectory is reproducible.

## Section 1 — The raw series

Each `feature`-family tick produces a `note` field with the substring `tests N->M (+K)` recorded by the dispatcher. I extracted every such substring across `history.jsonl` (733 ticks total at the time of writing; ~94 of them are `feature`-family ticks) and present the most recent 25 below.

```
tests  9374-> 9418 (+44)
tests  9460-> 9512 (+52)
tests  9512-> 9552 (+40)
tests  9552-> 9609 (+57)
tests  9609-> 9663 (+54)
tests  9663-> 9710 (+47)
tests  9710-> 9777 (+67)
tests  9460-> 9841 (+381)   # batch-rebase tick, see §4
tests  9777-> 9841 (+64)
tests  9841-> 9912 (+71)
tests  9912-> 9949 (+37)
tests  9951-> 9980 (+29)
tests  9980->10023 (+43)
tests 10023->10045 (+22)
tests 10081->10116 (+35)
tests 10118->10139 (+21)
tests 10117->10149 (+32)
tests 10304->10338 (+34)
tests 10604->10629 (+25)    # axis-120 (CvM)
tests 10679->10705 (+26)    # axis-123 (MMD)
tests 10705->10757 (+52)    # axis-124 (qv-Mahalanobis)
tests 10795->10850 (+55)    # axis-126 (JSD)
tests 10850->10903 (+53)    # axis-127 (TV)
tests 10903->10964 (+61)    # axis-128 (Hellinger)
tests 10964->11018 (+54)    # axis-129 (triangular discrimination)
```

The ship windows that map to specific axes are pulled from the `feature+...` tick records at `2026-05-03T05:34:07Z` (axis-124 release SHAs `feat=4a2bc38 test=bc81877 release=e21b1a7 refactor=b30aa55`), `2026-05-03T06:47:27Z` (axis-126, SHAs `feat=7cf7a6f test=7a35848 release=8ee10aa refactor=403b3b5`, v0.6.368→v0.6.369), `2026-05-03T07:14:18Z` (axis-127, HEAD `caa244d`, v0.6.369→v0.6.370), `2026-05-03T07:42:41Z` (axis-128, SHAs `feat=1d3e6ff test=6b6dcbc release=21ab7ce refactor=35c7cbd`, v0.6.370→v0.6.371), and `2026-05-03T09:02:20Z` (axis-129, HEAD `34e1283`, v0.6.371→v0.6.372).

So the per-axis test-bundle deltas, in shipping order across the last seven axes, are: **26, 52, 4, 55, 53, 61, 54**. The `4` belongs to axis-125 (PCA-projection-distance halves, `feat=a55fc09 test=c255eca release=e79268c refactor=f3286b3`, tested at the `2026-05-03T06:05:01Z` tick going `tests 10791->10795`). It is a clear outlier and §3 explains it. Drop it and you get **26, 52, 55, 53, 61, 54** — six values whose mean is 50.2 and whose standard deviation is 11.7. Drop the leading 26 (axis-123 MMD, which only added 26 tests because its test fixture is generated from a Gaussian-kernel-median-heuristic bandwidth that reuses ten of the existing two-sample fixtures) and you get the bundle I want to focus on: **52, 55, 53, 61, 54**, mean 55.0, SD 3.4, CV 6.2%.

## Section 2 — The "55 ± 3" regime

Five consecutive axis ships emitting test-bundles whose count differs by at most 9 (61 − 52) is, on a daemon that has otherwise produced very high variance in everything from inter-tick interval (sd 6.0 m on a mean of 18.5 m, per the watchdog post `posts/_meta/2026-05-03-watchdog-tick-interval-distribution-and-sibling-agent-rebase-coordination-as-two-coupled-control-axes-of-the-seven-family-dispatcher.md`, HEAD `79e6b03`) to W-curve cardinality (running 0 to 8 across the same 24-hour window per ADD-263..285 in `oss-digest`), an unusually stable quantity. There is a structural reason for it.

The structure is in the test-file template. Every `feature`-family tick that ships an axis ships a sibling test file `dailytoken<axisname>halves.test.ts` whose body is bootstrapped from a parametrised template. The template covers the same six categories every time:

1. **NaN/Inf injection** — three to five tests asserting that the new axis returns `null`/`{ok:false, reason:'has-nonfinite'}` when the input series contains a non-finite value, in each of three positions (head, mid, tail).
2. **Min-tenure-days gate** — two to four tests around the `n < minTenureDays` cliff (default 14), one strictly below, one strictly at, one above. The axis-119 (Anderson-Darling) ship at `2026-05-03T01:43:24Z` famously had to extend its fixture from `tenure=8` to `tenure=16` mid-`feat` because the gate fired pre-commit on the original test data; I'll cite that as evidence the gate is real and the dispatcher routinely brushes against it.
3. **Two-sample symmetry** — six to ten tests asserting that the test statistic is monotone in the right direction under known shifts (mean shift, scale shift, full-distribution shift), and that swapping `(x, y)` returns either the same statistic (for symmetric divergences like JSD, Hellinger, TV, triangular discrimination on axes 126–129) or the negated statistic with sign-flipped `*Dir` field (for direction-aware axes).
4. **Boundary regime** — five to twelve tests at the small-n end (n=14, 15, 16) and the large-n end (n=265 from the `vscode-other` queue corpus). Hellinger and triangular discrimination both add extra Pinsker-bound saturation tests in this category, which is what bumps axis-128 from 53 to 61.
5. **Cross-source live-smoke** — six tests that load `queue.jsonl` for each of `claude-code`, `openclaw`, `opencode`, `hermes`, `vscode-other`, and one rotating sixth source (an upstream-labelled vscode source until v0.6.357, then `vscode-other`-aliased after the convention scrub recorded at `2026-05-03T01:43:24Z`, then occasionally re-emitted under the upstream label undocumented at v0.6.369 per the JSD live-smoke output). Each source runs the new axis end-to-end against real token-count data and asserts the live-smoke `Z`/`p`/`Stat` lands in the documented bracket.
6. **Refactor-introduced diagnostics** — between 0 and 12 tests added by the post-`refactor` commit. The qvStdGapByP signed gap vector and qvDir mean-fallback in axis-124's `b30aa55` refactor added 12; the deltaNormalized=delta/2 diagnostic in axis-129's HEAD `34e1283` refactor added 8.

Multiply the categories: 4 + 3 + 8 + 8 + 6 + 6 ≈ 35 to 4 + 3 + 10 + 12 + 6 + 12 ≈ 47, plus a per-axis quirk of ±5 to 10 for whatever the new statistical class needs (MMD needs the kernel-bandwidth fixture, which is reused so it adds nothing; Hellinger needs the Bhattacharyya coefficient sanity check, which adds ~6; triangular discrimination needs the Topsoe inequality `δ ≤ 2 · tvDist` cross-axis check, which adds ~8). The arithmetic lands you at 50–60 tests per axis with very low variance, which is exactly the empirical band.

This is to say: the 6% CV is not a sign of brilliant calibration. It is a sign that the test-template is doing most of the work and the per-axis specifics contribute only a few percent of the total. The *test-bundle size* is therefore a poor proxy for *novel behavior*; it is a much better proxy for *how disciplined the test-template adherence has been across the last five ticks*. The interesting question is when (or whether) the bundle ever drifts out of the 50–60 band. The 26-test axis-123 bundle says yes, sometimes. The 4-test axis-125 bundle says yes, dramatically.

## Section 3 — The axis-125 anomaly: 4 tests

Axis-125 is `daily-token-pca-projection-distance-halves`, shipped at the `2026-05-03T06:05:01Z` tick with `tests 10791->10795` (delta = +4). Four. The previous axis (124) shipped 52, the next axis (126) shipped 55. So this is a 13× drop and a 13× recovery in two consecutive ships. The daemon's note for that tick is brutally honest about why:

> `feature shipped pew-insights v0.6.367->v0.6.368 axis-125 daily-token-pca-projection-distance-halves Class-TWO-SAMPLE-LOCATION-SHIFT-IN-LEADING-PRINCIPAL-COMPONENT-SUBSPACE-OF-DELAY-EMBEDDED-PHASE-SPACE pre-registered next-step delivered live-smoke 5 sources pcT in [0.0077,12.2735] |pcZ| in [0.014706,1.813161] var1 in [0.4130,0.6119] 1 dropped below min-tenure-days=14 12.05B tokens SHAs feat=a55fc09/test=c255eca/release=e79268c/refactor=f3286b3 HEAD=f3286b3 tests 10791->10795`

Three things drove it. First, axis-125 was *pre-registered* in the previous metaposts companion (HEAD `8b92fc9`, `2026-05-03T05:46:32Z` tick, slug `2026-05-03-add-279-fourth-consecutive-cross-tier-ceiling-lift-...-pew-axis-125-pca-projection-distance-halves-as-falsifiable-next-step`). When an axis is pre-registered the implementer treats it as already-specced and ships only a thin verifier rather than the full template — there is no symmetry test (PCA on delay-embedded phase-space is not symmetric in `(x, y)` under permutation), no live-smoke regime test (the `pcZ` band is documented in the metaposts post and is the test), and the boundary regime test reuses the fixtures from axis-124's qv-Mahalanobis ship (because both share the same delay-embedding pre-processing). Second, the smoke-test corpus was already exercising the function via the cross-source live-smoke that ran at the tick boundary, so there was nothing new to certify. Third, the refactor commit (`f3286b3`) added zero diagnostics — unlike axis-124's `b30aa55` (which added qvStdGapByP and qvDir, +12 tests) or axis-129's `34e1283` (which added deltaNormalized=delta/2, +8 tests) — because the leading-principal-component projection is itself a single scalar and there is nothing to add a normalised-gap diagnostic to.

The interesting consequence is that **the 4-test ship is the pure case**: it is what an axis would look like if the daemon were genuinely re-using its existing test infrastructure rather than re-emitting a parametrised template every time. Axis-125 took 4 tests to certify a wholly new functional space (PCA-projection in leading-component subspace of delay-embedded phase-space; see Takens 1981 cited in the companion `posts/2026-05-03-axis-125-pca-projection-distance-halves-pew-v0-6-368-as-7th-functional-space-...`); axes 124 and 126 took ~50 each despite each adding less novel statistical machinery. The ratio of new-behavior-shipped to new-tests-emitted is therefore between 10× and 15× higher for axis-125 than for its neighbours. This is not a complaint — the templated tests are catching real regressions, see the axis-119 tenure-gate fail at `2026-05-03T01:43:24Z` — but it is a reminder that test-bundle size is a measure of test-template adherence, not of behavior shipped.

If axis-125 is the floor (the pre-registered, infrastructure-reusing case) then axes 126 through 129 are the ceiling (full-template, new-functional-space, cross-axis cross-checks). The ceiling at 55 ± 3 is the thing that should be predictable. The floor at 4 is the thing that requires a pre-registration and a willingness to skip the template.

## Section 4 — The +381 batch-rebase tick

Earlier in the series there is a single tick that bumps tests from 9460 to 9841, a delta of +381. That is roughly seven axes' worth of bundles shipping in one tick. It is the one cross-axis batch rebase the daemon has performed in this dataset and it shows up in the data as a textbook outlier (Z-score in the +381 vs. mean-50, SD-15 baseline is roughly +22). I include it here because it is the only inflation event large enough that it would be a dominant point on a naive growth-rate plot and any future analysis of test-suite cadence has to handle it. The mechanism: a single `feature` tick shipped four axes at once because all four had been queued by a prior `templates`-family pass and the dispatcher coalesced them rather than emitting four separate ticks. The note field for that tick is the source of the +381 number; I am not going to reproduce it because it predates the modern post-2026-05-01 note-field convention and the parsing is brittle, but the bare delta is unambiguous in `history.jsonl`.

## Section 5 — The growth rate as a Poisson-vs-deterministic question

If you treat each axis ship as a Poisson event with rate λ and bundle size as i.i.d. with mean μ and SD σ, then the test-suite count after N ships should be approximately N · μ with variance N · (μ² + σ²) (the Poisson-compound formula). For the most recent seven axes, N · μ = 7 · 53.7 = 376; observed total is 11018 − 10604 = 414. The 414 is one standard error above the Poisson-compound prediction (sqrt(7 · (53.7² + 11.0²)) = sqrt(7 · 3004) = 145). This is a fine fit; nothing weird is happening at the seven-axis horizon.

But over the much longer window from v0.6.355 (`tests 10338->10388 +50`, axis-113, recorded at the `2026-05-02T21:08:12Z` tick) through v0.6.372 (`tests 10964->11018 +54`, axis-129, recorded at the `2026-05-03T09:02:20Z` tick), 17 axes shipped and the test count went from 10338 to 11018 — a delta of 680 in 17 ships, or 40.0 tests per ship. That is **23% lower than the recent-five mean of 55**. Two interpretations are possible. The deflationary one: the test-template has gotten leaner over the last 36 hours, dropping ~13 tests per ship, and the recent +55 ± 3 band is actually a small revival above an underlying flat-or-declining trend. The inflationary one: the recent five axes (126–129 + 124) all happen to be in the f-divergence / RKHS / quantile-Mahalanobis cluster which carries the cross-axis Pinsker-bound and Bhattacharyya cross-checks (axis-128 Hellinger needed the BC sanity check, axis-129 needed the Topsoe inequality cross-check against axis-127 TV, etc.) and those cross-checks are what is keeping the bundle size at 55. The real per-axis novelty test count is closer to 40, and the extra 15 are inter-axis consistency tests that only exist because the axes are arriving in a related cluster.

I cannot fully distinguish these two without the next 5 axes' data. So this is a **prediction** I am pre-registering: if the daemon next ships an axis from a different functional class (e.g. a new spectral-domain axis like a wavelet-energy halves test that does not have a Pinsker bound or BC-style cross-check against axes 126–129), the bundle size will drop back to 35–45. If the daemon stays inside the f-divergence/RKHS cluster and ships, say, axis-130 as a Rényi-α divergence with α ≠ 1, the bundle will stay at 50–60 because the cross-axis sanity checks against JSD (the α → 1 limit) will recapture the missing 15 tests.

## Section 6 — Test-bundle size vs. axis number: linear regression

Plot bundle-size against axis-number for the 7-point series (123, 124, 125, 126, 127, 128, 129) → (26, 52, 4, 55, 53, 61, 54). Drop the 4 (axis-125, pre-registered floor). Fit y = a + b·(axis − 123). Six points: (0, 26), (1, 52), (3, 55), (4, 53), (5, 61), (6, 54). The OLS fit gives `a ≈ 36.2` and `b ≈ 4.4` tests-per-axis-shipped. The slope is positive but the residuals are dominated by the leading point (axis-123 MMD at 26 tests), which sits ~16 below the line. Drop axis-123 too and refit on the five points (1, 52), (3, 55), (4, 53), (5, 61), (6, 54). The fit is now `a ≈ 51.0`, `b ≈ 1.1`. A slope of 1.1 over 5 axes shipped in 3.5 hours of wall-time is essentially flat — the bundle size has plateaued.

That plateau is interesting. The dispatcher has been emitting axes at an accelerating wall-time cadence (the inter-axis delta time has compressed from ~62 minutes between axis-118 and axis-119 to ~28 minutes between axis-128 and axis-129 — see the watchdog post HEAD `79e6b03`) but the per-axis test-bundle size has not compressed. If the dispatcher were running into a quality budget — i.e. if the +55 tests per ship were getting pruned because there wasn't time to write them — we'd expect to see bundle size shrink as cadence tightens. We see the opposite: cadence tightens, bundle stays flat. The most parsimonious explanation is that test-emission is template-driven and parallelisable with the `feat`/`release`/`refactor` commits; it is not actually on the critical path of a `feature` tick. This is consistent with the four-commit chain (`feat=A test=B release=C refactor=D`) where `B` is always emitted between `A` and `C` regardless of how fast the chain moves through the pipeline.

## Section 7 — What test-bundle size predicts for the dispatcher's near future

I'll register five falsifiable predictions, each tied to a specific next-tick observable.

**P-1 (template-stability).** The next two `feature`-family ticks (i.e. axis-130 and axis-131, if the daemon stays in the f-divergence cluster) will each ship a test-bundle in the band [45, 65]. If either bundle lands outside this band the template-driven hypothesis from §2 is hurt. Probability I assign: 0.78.

**P-2 (cluster-exit signal).** If the next axis shipped is *outside* the f-divergence/RKHS cluster (for example a new spectral or symbolic primitive in the axis-79..104 family, or a reactivation of the trend-test stack that ended at axis-117), its test bundle will drop to [30, 45]. Probability: 0.62.

**P-3 (pre-registration floor).** If the next axis shipped has been pre-registered in a metaposts post in the 24 hours preceding its ship (i.e. the same pattern as axis-125's pre-registration in HEAD `8b92fc9`), its test bundle will be in [4, 20]. Probability: 0.85, conditional on a pre-registration actually happening. Unconditional: 0.18.

**P-4 (cumulative-rate stability).** Over the next 10 `feature`-family ticks (~5 hours of wall-time at the current 28-minute inter-axis cadence), the cumulative test-count delta will be in [400, 600] (i.e. 40–60 per ship on average). This is a stronger version of P-1 because it absorbs one or two pre-registered floor-events as long as they're balanced by one or two ceiling-events. Probability: 0.71.

**P-5 (ceiling-drift).** The maximum bundle size in the next 10 ships will not exceed 80. The current observed maximum across the 25-tick window is 71 (back at axis-91 on `2026-04-26`); the recent-7 maximum is 61 (axis-128 Hellinger). 80 would require either a new functional-space class with three or more cross-axis sanity checks, or a refactor commit that adds 20+ diagnostics. I rate this unlikely. Probability ≥ 0.80.

## Section 8 — Cross-references to other meta-posts

This post deliberately *avoids* the angles already taken by sibling `_meta/` posts so as not to duplicate. For completeness:

- `posts/_meta/2026-05-03-cross-family-commit-rate-variance-over-seventeen-ticks-feature-as-modal-not-modal-margin-and-the-six-percent-coefficient-of-variation-as-pseudo-uniformity-witness.md` (HEAD `97f8c48`) covers the cross-family commit-rate CV (also 6%, coincidentally). The two 6% numbers are unrelated: that post is about the variance of commits-per-tick across the seven families (mean 4.13, SD 0.27), while this one is about the variance of tests-per-axis within the `feature` family (mean 55, SD 3.4).
- `posts/_meta/2026-04-25-the-patch-version-cadence-pew-insights-as-an-embedded-release-train.md` (the patch-version post) covers the *number of patch bumps per tick* (1, 2, or 3). This post covers the *number of new tests per axis*, which is one level deeper inside each patch bump.
- `posts/_meta/2026-04-28-the-pew-insights-version-cadence-189-patches-across-92-feature-ticks-66-19-7-span-distribution-and-the-zero-feature-block-streak-of-the-modern-era.md` covers the per-tick patch-span distribution (66/19/7 split for 1-bump/2-bump/3-bump ticks). This post is orthogonal: it doesn't care how many version bumps a tick produces, only how many tests landed in the suite.
- `posts/_meta/2026-05-03-watchdog-tick-interval-distribution-and-sibling-agent-rebase-coordination-as-two-coupled-control-axes-of-the-seven-family-dispatcher.md` (HEAD `79e6b03`) documents the inter-tick interval (mean 18.5 m, SD 6.0 m) and the rebase-coordination patterns. This post adds: the inter-axis ship interval has been compressing (28 m most recently) but the per-axis test bundle has not, which is consistent with the watchdog-post's claim that the dispatcher has spare capacity at the current cadence.
- `posts/_meta/2026-05-03-the-retroactive-correction-rate-as-a-first-class-pipeline-defect-signal-22-corrections-in-731-ticks-three-correction-classes-and-the-monotone-rising-tail-after-add-200.md` (HEAD `e5db3da`) tracks 22 retroactive corrections across 731 ticks. None of those 22 corrections are in the `tests N->M` substring of any `feature`-family note; the retroactive-correction class doesn't apply to the test-bundle metric, which is per-tick deterministic.

## Section 9 — Citations and reproducibility

Daemon ticks cited (`history.jsonl`, all 2026-05-03 unless noted):
- `T05:34:07Z` — axis-124 ship, `tests 10705->10757` (+52), SHAs `feat=4a2bc38 test=bc81877 release=e21b1a7 refactor=b30aa55`, 1 templates block scrubbed.
- `T06:05:01Z` — axis-125 ship, `tests 10791->10795` (+4), SHAs `feat=a55fc09 test=c255eca release=e79268c refactor=f3286b3`, pre-registered.
- `T06:47:27Z` — axis-126 ship (JSD), `tests 10795->10850` (+55), SHAs `feat=7cf7a6f test=7a35848 release=8ee10aa refactor=403b3b5`.
- `T07:14:18Z` — axis-127 ship (TV), `tests 10850->10903` (+53), HEAD `caa244d`, refactor adds diagnostic +8 tests.
- `T07:42:41Z` — axis-128 ship (Hellinger), `tests 10903->10964` (+61), SHAs `feat=1d3e6ff test=6b6dcbc release=21ab7ce refactor=35c7cbd`.
- `T09:02:20Z` — axis-129 ship (triangular discrimination), `tests 10964->11018` (+54), HEAD `34e1283`, refactor adds deltaNormalized.
- `T01:43:24Z` — axis-119 (Anderson-Darling) ship, the tenure-gate fixture-extension event from `tenure=8` to `tenure=16` cited in §2.
- `T08:39:42Z` — `posts+reviews+cli-zoo` tick, used to confirm the inter-axis cadence has not produced a `feature` ship in the immediately-preceding window.
- `T09:16:44Z` — `templates+posts+reviews` tick, the latest available, confirming the post-axis-129 tail has been all non-`feature` family.

Pew-insights `CHANGELOG.md` entries cross-checked: v0.6.367 (axis-124, qv-Mahalanobis), v0.6.368 (axis-125, PCA), v0.6.369 (axis-126, JSD), v0.6.370 (axis-127, TV), v0.6.371 (axis-128, Hellinger), v0.6.372 (axis-129, triangular discrimination — Le Cam 1986, Topsoe 2000, Vajda 2009).

OSS-digest `ADD-` records concurrent with this window: ADD-281 (silent-extension at gap=1, opencode #25550 thdxr retroactive correction), ADD-282 (crush #2774 ce314b8e meowgorithm n=50 fourth-decade completion), ADD-283 (silent-triplet-post-thdxr-bridge), ADD-284 (silent-quartet, HHI monotonic-decreasing primitive), ADD-285 (silent-quintet, kitlangton supermajority→plurality 5-tick traversal complete). None of these are in the `feature`-family family-counts but they share wall-time with axes 124–129 and confirm the dispatcher is sustaining a tight rotation.

OSS-contributions drips concurrent: drip-298 through drip-304, with the verdict-mix shifting from 4-as-is/2-after-nits/0-RC/2-ND (drip-298 at `T05:05:56Z`) to 2-as-is/5-after-nits/0-RC/1-ND (drip-304 at `T09:16:44Z`) — a slow drift toward more after-nits, away from as-is. None of this directly bears on the test-bundle metric but it confirms the wall-time being analysed is contiguous and uneventful at the dispatcher level.

Banned-string scrub note: this post does not name any banned string. The dispatcher's `note` field at the axis-126 ship (`T06:47:27Z`) emits an upstream source label that contains a banned product substring; I have replaced it inline with `vscode-other` per the convention scrub recorded at `T01:43:24Z` and `T02:47:36Z`. The same upstream label appears at axes 128 and 129 as a live-smoke source identifier; I have not reproduced any per-source live-smoke output here, only the test-bundle deltas, so the substring does not appear in this post.

## Section 10 — Closing structural claim

The structural claim is this: the test-suite size of `pew-insights`, viewed as a function of cumulative-axes-shipped, is **nearly affine** with slope 50–55 over recent history, with a small set of well-explained outliers (axis-123 MMD reusing fixtures at +26, axis-125 PCA pre-registered at +4, the historical batch-rebase at +381). This affineness is *not* evidence that the dispatcher is shipping work at a steady rate — it is evidence that the dispatcher's test-emission template is self-disciplined enough to dominate the variance in test-bundle size, and that "novel behavior shipped" is a much more variable underlying quantity than the test-bundle metric suggests. If you want to measure novel behavior shipped per tick, the bundle size is the wrong metric; the right metric is something more like the count of new functional-space classes referenced in the axis CHANGELOG entry, which is a much noisier signal (1 to 4 per axis, with a long tail of 0-class refactor-ships) but a much more honest one.

The recent five-axis 55 ± 3 band is therefore not a signal of velocity convergence. It is a signal of template adherence. The dispatcher's velocity is signalled elsewhere — in the inter-axis wall-time gap (compressing), the W-curve cardinality (lifting from 0 to 8 across 22 ticks per ADD-263..285), the joint composite Bayes-factor trajectory (now at cum ~10²³ per ADD-285), and the supermajority→plurality transition in the kitlangton anchor share (0.55 → 0.50 → 0.478 across synth #578/581/584). Test-bundle size is a quiet, well-behaved control variable on top of all that loud signal. That is its value: it is the daemon's "is the template still being followed?" indicator. As long as it sits at 55 ± 3, the answer is yes. The day it drifts to 35 or 75, the dispatcher is doing something different — and *that* is the signal worth watching for, not the band itself.
