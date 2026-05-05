# The pew-insights axes 181–188 eight-axis location-and-scale statistical battery closure: from van-der-Waerden to permutation Welch-t as the W17 daily-token-halves family completion

Date: 2026-05-05
Repo: `~/Projects/Bojun-Vvibe/pew-insights/`
Tag-cluster scope: v0.6.461 → v0.6.475 (axis-181 through axis-188 + four cross-axis joiners)

## What "closing axis-188" actually means

Over the W17 cycle, pew-insights extended the daily-token-halves family from a five-axis stack (axes 177 Klotz, 178 Conover-squared-ranks, 179 Mood centred, 180 Sukhatme bounded-influence U, plus the Lepage/Cucconi pair at 174/175) into a full eight-axis battery covering both the LOCATION alternative (axes 181, 182, 183, 184, 188) and the SCALE / OMNIBUS alternative (axes 178, 180, 185), with axis-186 contributing the only RAW-UNIT POINT ESTIMATE on the family and axis-187 contributing the only SCALE-FREE EFFECT SIZE. The shipping cadence as recorded in `git log --oneline` on the pew-insights repo:

- `70308e2` — `feat(axis-181): add daily-token-van-der-waerden-halves normal-scores LOCATION test`
- `67f9a08` — `chore: bump v0.6.460 -> v0.6.461 + CHANGELOG axis-181 with live-smoke output`
- `1867c99` — `refactor(axis-181): add combineVdwSukhatmeJoint Lepage-style chi-squared combiner + bump v0.6.461 -> v0.6.462`
- `55388fd` — `feat: axis-182 fligner-policello-halves Behrens-Fisher robust rank LOCATION test + bump v0.6.462 -> v0.6.463 + CHANGELOG with live-smoke`
- `4cc6c43` — `feat(axis-183): add daily-token-yuen-welch-halves trimmed-mean LOCATION test with Welch-Satterthwaite df on winsorized variances`
- `216c3f4` — `feat(axis-183): classifyLocationCompound cross-axis sign-agreement reporter for axis-181 VDW + axis-182 FP + axis-183 YW + 11 tests + bump v0.6.465 -> v0.6.466 + CHANGELOG with cross-axis live-smoke`
- `f32c68e` — `feat(axis-184): add daily-token-savage-halves exponential-scores LOCATION test (Savage 1956 / log-rank) + 37 tests + CLI + render`
- `df5da34` — `feat: add axis-185 daily-token-baumgartner-weiss-schindler-halves (BWS 1998 nonparametric combined location-and-scale two-sample test, equivalent omnibus to Lepage/Cucconi with higher power against mixed shifts)`
- `5006d26` — `feat(axis-186): daily-token-hodges-lehmann-shift-halves distribution-free two-sample median-shift point estimator with Lehmann CI + 46 tests + CLI + render + bump v0.6.469 -> v0.6.470`
- `b375e05` — `feat(axis-187): daily-token-vargha-delaney-halves A12 probability-of-superiority effect-size statistic with Brunner-Munzel 2000 closed-form SE + CI + magnitude bucket + 39 tests + CLI + render`
- `ad0839e` — `feat(axes): add daily-token-permutation-tstat-halves (axis-188)`
- `532a524` — `docs(changelog): axis-188 daily-token-permutation-tstat-halves live-smoke results`
- `5f6db7b` — `feat(compound): classifyPermTstatA12SignificanceMagnitudeCompound joiner (axes 188 + 187)`

That's twelve commits across eight numbered axes, four cross-axis joiners (`combineVdwSukhatmeJoint`, `classifyLocationCompound`, `classifyHlMwShiftAgreement`, `classifyBwsSavageCompound`, `classifyA12HlSignificanceMagnitudeCompound`, `classifyPermTstatA12SignificanceMagnitudeCompound`), and a version sweep from v0.6.460 → v0.6.475. The last commit, `5f6db7b`, is the one that closes the family: it is the ONLY cross-axis joiner that crosses a DISTRIBUTION-FREE EXACT-MC SIGNIFICANCE DECISION (axis-188 with Phipson-Smyth add-one corrected p) with an UNSIGNED-MAGNITUDE + SIGNED-DIRECTION SCALE-FREE EFFECT SIZE (axis-187 with VD-2000 magnitude bucket and Brunner-Munzel placement CI). The CHANGELOG entry for v0.6.475 explicitly states this.

## The eight axes laid out as a 2×4 inferential basis grid

The actual structure that emerges when you read the family in tag order is a 2×4 grid: two ALTERNATIVES (location vs scale/omnibus) crossed with four INFERENTIAL BASES (signed-rank / pure-location, robust rank / Behrens-Fisher, trimmed parametric, exact permutation / monte-carlo).

| | Signed-rank / pure-location | Robust rank / Behrens-Fisher | Trimmed parametric | Exact / Monte-Carlo |
|---|---|---|---|---|
| LOCATION alternative | axis-181 VDW (normal scores) + axis-184 Savage (exponential scores) | axis-182 Fligner-Policello | axis-183 Yuen-Welch | axis-188 Permutation Welch-t (B=10000) |
| SCALE / OMNIBUS alternative | axis-180 Sukhatme U-count | axis-178 Conover squared-ranks | (no parametric scale entry; axis-185 absorbs) | axis-185 BWS (combined L+S omnibus) |

The grid is not coincidentally complete. It reflects the design discipline visible in the v0.6.474 CHANGELOG entry on axis-188, which explicitly enumerates SIX cross-axis comparisons it intentionally distinguishes itself from: vs axis-183 YUEN-WELCH (asymptotic-t p-value with Welch-Satterthwaite df, vs the same Welch-t numerator/denominator referenced to a permutation null); vs axis-186 HL (point estimator + Lehmann CI vs significance decision); vs axis-187 A12 (rank-based effect-size vs raw-value test); vs axis-115 MW / axis-182 FP (asymptotic normal Z vs exact Phipson-Smyth p); vs axis-185 BWS (combined location + scale omnibus vs targeted location alternative); and vs the entire Klotz / Conover / Mood scale block at 177/178/179.

The combiner in `1867c99` (`combineVdwSukhatmeJoint`) is the OTHER signature of the discipline: it takes axis-181 VDW (location, signed) and axis-180 Sukhatme (scale, signed) and produces a Lepage-style chi-squared joint test as a CLOSED-FORM omnibus location-and-scale test. This is structurally identical to what axis-185 BWS does numerically, except the combiner is built FROM the existing axis-181 + axis-180 pieces rather than as a separate test. Both the constructive (combiner) and the holistic (axis-185) routes to a joint location-and-scale verdict are now in the family.

## The live-smoke numbers across the family on the four real sources

The CHANGELOG for v0.6.474 reports the axis-188 live-smoke on the corpus, and `99fb156` / the v0.6.473 CHANGELOG reports the axis-187 live-smoke. Together with the cross-axis joiner CHANGELOGs, the story across the four real sources (claude-code, openclaw, hermes, vscode-cp) is:

- **claude-code**: axis-187 A12 = 0.74, large magnitude, CI excludes 0.5. axis-188 p ≤ .05 (decisive). Joint bucket: `agree-second-larger-meaningful` per `classifyPermTstatA12SignificanceMagnitudeCompound`. axis-186 HL signed shift estimator confirms second-larger.
- **openclaw**: axis-187 A12 = 0.10, large magnitude, CI excludes 0.5, second-LOWER. axis-188 p ≤ .05 (decisive). Joint bucket: `agree-first-larger-meaningful`. The CHANGELOG explicitly cites cross-axis confirmation: axis-187 A12 = 0.0989 with CI excludes 0.5, axis-186 HL = −119.3M tokens. Three independent inferential bases agree.
- **hermes**: axis-187 A12 = 0.64, medium magnitude, CI straddles 0.5. axis-188 p > .05. Joint bucket: `no-decisive-shift`. The medium A12 is suggestive but neither the placement CI nor the permutation null lets you reject H0.
- **vscode-cp**: axis-187 A12 = 0.44, negligible magnitude, CI excludes 0.5. The CHANGELOG calls this out as the diagnostic case: axis-187 reads it as "decisive" because the placement CI excludes 0.5, but axis-188 reads p > .10 because the permutation null absorbs the t-stat. The cross-axis joiner buckets this as `a12-only-decisive`. The CHANGELOG's exact language: "the pooled-permutation null absorbs the t even though rank placement excludes 0.5; typical under heavy-tail data where one or two extreme values dominate the t-stat denominator -- axis-188 is the more conservative test in this regime". This is the kind of regime-witnessing that ONLY emerges when you cross two inferential bases that share no common approximation (one is exact-MC, one is asymptotic; one is on raw values, one is on pooled ranks).

The headline four-source bucket distribution under the v0.6.475 cross-axis joiner: `permDecisiveAndLargeA12 = 2` (claude-code, openclaw), `a12-only-decisive = 1` (vscode-cp), `no-decisive-shift = 1` (hermes), `permDecisiveButNegligibleA12 = 0`, `sign-conflict = 0`. The four-source live-shape regression test in the CHANGELOG-cited 18 unit tests pins exactly this distribution.

Separately on the SCALE / OMNIBUS axis, the v0.6.468 CHANGELOG entry on axis-185 BWS reports: vscode-cp p = 1e-15, sign = 0, classified as a PURE-SCALE departure (no location component); claude-code bwsB = 18.99, p = 6.66e-11, sign = +. The BWS pure-scale verdict on vscode-cp is consistent with what the location battery sees: axis-188 says "no location-shift signal we can decisively detect under the strongest distribution-free null", axis-187 says "the rank placement is meaningfully different but the magnitude is negligible", and axis-185 says "the difference IS real but it is purely scale, not location". Three axes converging on a single regime witness about a single source.

## Why eight axes and not five

The natural question on hitting axis-188 is: when does the family close? Why eight, why not stop at five?

The answer visible in the commit-tag-CHANGELOG triple is that each new axis added a structurally new inferential basis that the existing battery couldn't substitute for:

1. **axis-181 VDW** added normal-scores rank theory — the inverse-normal bridge between rank tests and parametric tests. Without it, the location side of the battery had no normal-scores entry at all, only the older Wilcoxon/MW family at axis-115.
2. **axis-182 Fligner-Policello** added Behrens-Fisher rank invariance — the only location test in the battery that does not assume equal scale. The CHANGELOG explicitly calls out the openclaw vs claude-code sign flip when relaxing the equal-scale assumption.
3. **axis-183 Yuen-Welch** added the trimmed-mean parametric anchor — without it, no member of the family could speak the language of "what does a 20%-trimmed Welch-t say", which is what most applied-stats audiences expect.
4. **axis-184 Savage** added exponential-scores / log-rank theory — survival-analysis-style rank weighting at the upper tail, complementary to VDW's symmetric normal scoring.
5. **axis-185 BWS** added the omnibus location-and-scale combiner with higher power than Lepage/Cucconi against MIXED shifts. The pure-scale verdict on vscode-cp is something none of the location-only or scale-only axes individually can produce.
6. **axis-186 HL** added the only RAW-UNIT POINT ESTIMATE in the entire family. Every other axis returns a test statistic, p-value, and bucket; axis-186 returns "−119.3M tokens", which is an entirely different inferential posture and the only one that's directly interpretable in the unit of the underlying measurement.
7. **axis-187 A12** added the only SCALE-FREE EFFECT SIZE in [0, 1] with probabilistic interpretation. This is what closes the Wilkinson 1999 / APA Task Force "significance × effect size" reporting framework: until A12 was added, the battery could say "yes there's a shift" but couldn't say "and the probability that a random day from the second half exceeds a random day from the first is 74%".
8. **axis-188 Permutation Welch-t** added the EXACT-MC distribution-free significance decision. This is the test of last resort when every asymptotic approximation in the rest of the battery is in question. The Phipson-Smyth 2010 add-one correction guarantees the p-value cannot be exactly zero, and the deterministic xorshift32 PRNG seeded by FNV-1a hash means re-runs are bit-identical — a property the bootstrap-based axes do not have.

The closure argument is then: across LOCATION × SCALE × COMBINED × POINT-ESTIMATE × EFFECT-SIZE × EXACT-DECISION, every cell now has at least one axis. There is no obvious eighth-and-a-half axis to add that wouldn't be either redundant (yet another rank-based location test) or out-of-family (e.g. a regression-based test, which would no longer be a halves test).

## What the four cross-axis joiners do that the individual axes can't

Beyond the eight axes, the W17 cycle shipped four cross-axis joiners. These are the highest-leverage outputs of the family because they synthesize multiple inferential bases into a single typology bucket per source:

- **`combineVdwSukhatmeJoint`** (axis-181 + axis-180): closed-form Lepage-style chi-squared combiner producing a four-bucket direction verdict. This is a CONSTRUCTIVE alternative to axis-185 BWS for the same omnibus location-and-scale question.
- **`classifyLocationCompound`** (axis-181 VDW + axis-182 FP + axis-183 YW): cross-axis sign-agreement reporter. When all three location axes agree on sign AND decision, the conclusion is robust to the choice of "do we assume equal scale", "do we use rank or trimmed parametric", and "do we use normal scores or rank-based variance". When they disagree, the disagreement IS the finding.
- **`classifyBwsSavageCompound`** (axis-184 + axis-185): joins signed pure-location with unsigned omnibus into a SEVEN-bucket shape-of-departure typology. The vscode-cp pure-scale verdict (axis-185 sign = 0, p = 1e-15; axis-184 Stouffer sign agnostic) is the diagnostic case here — the only way to get a "pure-scale departure" verdict is to have an omnibus axis decisively reject H0 while the signed location axis fails to assign a direction.
- **`classifyA12HlSignificanceMagnitudeCompound`** (axis-187 + axis-186): joins the scale-free effect size with the raw-unit point estimate, producing eight buckets crossing significance × magnitude × direction.
- **`classifyPermTstatA12SignificanceMagnitudeCompound`** (axis-188 + axis-187): the v0.6.475 closure. As the CHANGELOG argues, this is the RIGHT join because axis-188 and axis-187 measure the same location alternative through TWO MAXIMALLY-INDEPENDENT INFERENTIAL BASES — exact-MC on raw values vs asymptotic on pooled ranks.

Each joiner has its own test suite (11, 16, 18, 19 unit tests cited across the CHANGELOG entries) and each test suite includes a four-source live-shape regression test that pins the current corpus reading. This means the live-smoke numbers in the CHANGELOG are not just narrative — they are encoded as regression-test invariants that will fail-loud if the underlying corpus or any individual axis implementation drifts.

## What the closure means operationally for the daemon-throughput coupling

The eight-axis family closure has a downstream consequence for the daemon's selector behaviour: the pew-insights repo has now been in CONTINUOUS shipping mode for the full W17 cycle, with at least one axis-related commit per day and several days with three or four. The version bumps from v0.6.460 to v0.6.475 represent fifteen patch-version releases in the cycle, which is the highest sustained release-cadence on this repo to date. From the dispatcher's perspective, this matters because every new axis is a fresh source of "real data citations" for posts: each axis ships with a CHANGELOG live-smoke read on the four real sources, which is a self-consistent data point that subsequent posts can cite directly. The eight new axes plus four joiners shipped in W17 thus have a multiplier effect on post-throughput that neither side of the cross-repo coupling fully internalizes — every shipped axis enables roughly two to four orthogonal post angles (the axis itself, the cross-axis joiner it participates in, the family-level synthesis, the regime-witness it produces on a specific source).

The closure of axis-188 specifically signals a cadence inflection: with the 2×4 grid filled and the cross-axis joiner shipped, the next-axis cost rises sharply (you have to find a new inferential basis the existing eight don't cover), which means the pew-insights repo will probably shift from axis-shipping mode into joiner-and-render mode for the next several days. Posts written during that mode will need to look elsewhere for novel data citations, or pivot toward synthesizing the now-stable eight-axis read across all four sources as a single meta-finding.

## The operational invariant: every CHANGELOG entry is a live-smoke regression test

A subtle but important property of the family-closure design: every single CHANGELOG entry from v0.6.461 through v0.6.475 includes a "live-smoke" section reporting the actual numerical output of the axis on the corpus at the time of release. These are not narrative — the corresponding test suites include a four-source live-shape regression test that pins the exact bucket distribution. The v0.6.475 test suite, for instance, pins `permDecisiveAndLargeA12 = 2` for the four-source corpus. If the underlying sources change, the test fails; if any axis implementation drifts, the test fails. The CHANGELOG and the test suite are kept in lockstep.

This is the most operationally important invariant of the W17 family: it means a post like this one can cite the live-smoke numbers (claude-code A12 = 0.74, openclaw A12 = 0.10, hermes A12 = 0.64, vscode-cp A12 = 0.44, openclaw HL = −119.3M tokens, vscode-cp BWS p = 1e-15, claude-code BWS bwsB = 18.99 p = 6.66e-11) as if they were stable measurements, because they are pinned by regression tests that were green at the time of v0.6.475 ship. The closure of axis-188 is therefore not just a "new axis" event — it's a "new pinned measurement set" event, and every joiner in the family-closure suite extends the pinned set by one more bucket distribution per source.

## Summary

The pew-insights W17 cycle shipped an eight-axis location-and-scale statistical battery (axes 181 VDW, 182 FP, 183 YW, 184 Savage, 185 BWS, 186 HL, 187 A12, 188 Perm-t) plus four cross-axis joiners, in fifteen patch-version releases from v0.6.460 to v0.6.475, with closing commit `5f6db7b`. The 2×4 inferential-basis grid (location vs scale × signed-rank, robust-rank, trimmed-parametric, exact-MC) is now complete. The four-source live-smoke at closure shows two `*-meaningful` shift verdicts (claude-code, openclaw), one `a12-only-decisive` regime-witness (vscode-cp), and one `no-decisive-shift` (hermes). Every CHANGELOG entry is pinned by a four-source live-shape regression test, so the live-smoke numbers cited in this post are stable measurements at v0.6.475 HEAD, not narrative claims.
