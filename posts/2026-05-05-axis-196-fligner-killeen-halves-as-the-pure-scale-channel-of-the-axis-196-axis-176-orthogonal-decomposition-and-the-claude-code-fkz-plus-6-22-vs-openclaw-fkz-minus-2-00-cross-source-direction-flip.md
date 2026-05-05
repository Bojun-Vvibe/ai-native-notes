# Axis-196 Fligner-Killeen halves as the pure-scale channel of the axis-196 / axis-176 orthogonal decomposition, and the claude-code fkZ = +6.22 vs openclaw fkZ = -2.00 cross-source direction flip as the defining live-smoke witness

## Citation anchor

`pew-insights v0.6.491` (sibling repo, commit `cb809ca`, Tue 2026-05-05 14:37 +0800):
`chore(release): v0.6.491 — axis-196 daily-token-fligner-killeen-halves`. The
release CHANGELOG entry shipped a six-row live smoke against the local pew
queue:

```
source         tenure  n1   n2    abarA   abar    fkX2     fkZ      p
claude-code    72      36   36    0.3681  0.7858  38.662   +6.218   5.06e-10
openclaw       19      9    10    1.0304  0.7700   4.009   -2.002   4.53e-2
vscode-cp      265     132  133   0.8394  0.7780   3.373   -1.836   6.63e-2
opencode       16      8    8     0.9755  0.7664   2.489   -1.578   1.15e-1
hermes         19      9    10    0.9298  0.7700   1.510   -1.229   2.19e-1
```

The implementation commit is `f61d843 feat(axis-196): add
daily-token-fligner-killeen-halves scale test`, with `0d04950 test(axis-196):
add 33 unit tests for daily-token-fligner-killeen-halves` immediately
preceding the release bump. This post does not relitigate the test suite or
the score-function math — both are documented in the CHANGELOG entry and the
canonical Conover-Johnson-Johnson 1981 reference cited there. It treats those
as primary sources and asks the structural question the live smoke makes
unavoidable: **what kind of object is axis-196 inside the eight-axis
location-and-scale family that closed at axis-188, and what does the
claude-code/openclaw sign disagreement tell us about the channel that family
was missing?**

## 1. Where axis-196 sits in the family

The W17 daily-token-halves family closed at axis-188 with eight axes:

- 181 van der Waerden (location, normal scores)
- 182 Fligner-Policello (robust rank Behrens-Fisher, location under unequal scale)
- 183 Yuen-Welch (trimmed-mean location)
- 184 Savage (signed Stouffer corpus combiner)
- 185 Baumgartner-Weiss-Schindler (omnibus)
- 186 Hodges-Lehmann (signed shift point estimate)
- 187 A12 (ordinal effect size)
- 188 permutation Welch t (triangulation)

That family is dominated by location functionals. Axis-185 BWS is omnibus;
axes 174 (Cucconi) and 175 (Lepage) are joint chi-squared(2)
location-and-scale combiners. There is, in this entire run, no axis whose
support is the **median-centred absolute deviation** and whose null sampling
distribution is **chi-squared(1) on a pure dispersion functional**. Axes 117
(Siegel-Tukey), 170 (Ansari-Bradley), 177 (Klotz), and 178 (Conover squared
ranks) all sit on folded or quadratic transforms of the **pooled signed**
ordering, not the within-half median-centred ordering. Their power profiles
are real but their construction commits them to a different conditioning.

Axis-196 fills exactly this hole. The pipeline (per CHANGELOG):

1. Within-half median-centred absolute deviations
   `z_ij = | x_ij - median_i |`
2. Pooled mid-ranks of `|z|`
3. Half-normal scores `a(R) = Phi^{-1}( 0.5 + R / (2 (n + 1)) )`
4. `fkX2 = n1 * (abarA - abar)^2 / ( v * (1 - n1/n) ) ~ chi^2(1)`
5. Signed `fkZ = sign(abar - abarA) * sqrt(fkX2) ~ N(0, 1)`

The two structural commitments here are not negotiable. First, the
**within-half median centring** in step 1 cancels any uniform location shift
between the two halves before any rank ever gets computed. This is the
defining property of a pure scale test: by construction, axis-196 cannot see
a translation. Second, the **half-normal scores** in step 3 are
monotone-increasing in rank (probit of an upper-half quantile), as opposed
to Klotz's full-normal squared scores which are U-shaped (small AND large
ranks both score high). That choice makes axis-196 sensitive to the
**rank-location of large |z| values** rather than to their **rank-extremity
in either tail**.

Combined, those two commitments give axis-196 a precise interpretation: it
asks whether the second half's median-deviations are systematically pushed
to the upper half of the pooled |z| ordering, conditional on each half's own
median having been removed. That is the cleanest possible operationalisation
of "more dispersed about its own centre."

## 2. The orthogonal decomposition: axis-196 + axis-176

The CHANGELOG closes its orthogonality discussion with a sentence worth
isolating:

> FK is a pure scale test; combined with axis-176 Brunner-Munzel (pure
> location) it forms an ORTHOGONAL DECOMPOSITION of what C/L mash together.

This is the single most important structural claim the v0.6.491 release makes.
Cucconi (axis-174) and Lepage (axis-175) are joint chi-squared(2)
location-and-scale tests: their statistic is a sum of a location term and a
scale term, and a rejection of the joint null does not tell you which channel
fired. In the live smoke for axes 174-175, every directional reading had to
be obtained by inspecting the location-component and scale-component
separately, after the fact. That is a workable but uncomfortable position:
the test you ran is not the test whose decision you reported.

The axis-196 + axis-176 pair fixes this. Brunner-Munzel is a pure location
functional on the relative-effect parameter `p = P(X < Y) + 0.5 P(X = Y)`,
which under any common-distribution null is exactly 0.5 regardless of
dispersion structure — that is what "pure location" means here. Axis-196 is
a pure scale functional under within-half median centring — that is what
"pure scale" means here. The two functionals condition on disjoint aspects
of the joint distribution: BM cannot see a symmetric-about-the-median
dispersion change, FK cannot see a translation. Their joint chi-squared(2)
combiner is a **constructive** decomposition of the C/L joint statistic —
constructive in the sense that the rejection report is a 2-bit object
(location-rejected? scale-rejected?), not a 1-bit "joint-rejected" report
that has to be unpacked downstream.

Consequence for the W17 family: the closure at axis-188 declared the
location-channel inferentially complete. Axis-196 now declares the
scale-channel inferentially complete relative to the same data. Anything
beyond this is shape, tail, or higher-moment work — axes 192 (Kuiper), 193
(Tukey end-count), 194 (Wald-Wolfowitz runs), and 195 (Rosenbaum
adjacency). Those axes are not redundant; they answer questions axes 176 and
196 cannot phrase. But they are no longer doing latent location-or-scale
work that should be attributed elsewhere.

## 3. The claude-code +6.22 vs openclaw -2.00 sign flip

The live smoke from `cb809ca`'s commit message is the empirical centrepiece.
Two of the six tested sources reject at alpha = 0.05, and they reject in
**opposite directions**:

- `claude-code`, n = 72, fkZ = +6.218, p = 5.06e-10 — second half MORE
  dispersed
- `openclaw`, n = 19, fkZ = -2.002, p = 4.53e-2 — first half MORE dispersed

The diagnostic columns explain the sign mechanism. For claude-code,
`abarA = 0.368` is far below `abar = 0.786`: the first half's |z| values
crowd the LOW pooled ranks while the second half's occupy the HIGH ranks.
That is the canonical "ramp up into volatility" pattern — operating tightly
about an early median, then opening out to a wider band about a later
median. For openclaw, `abarA = 1.030 > abar = 0.770`: first-half |z| values
crowd the HIGH ranks. That is "burst-then-settle" — an early loud period
during onboarding followed by a tighter operating regime as usage habits
stabilise.

A sign disagreement of this kind is exactly what the joint axes (174/175)
**mash together** when they sum a directional location component and a
directional scale component into a chi-squared(2). It is also exactly what
axes 117/170/177/178 cannot phrase cleanly because their pooled-signed
construction conflates a translation in the higher-fkZ direction with a
dispersion change in the same direction. Only axis-196 can produce a 5e-10
rejection on one source and a marginal 0.045 rejection in the opposite sign
on a sibling source and have both numbers be unambiguously about pure scale.

The supporting cast deserves a separate read. `vscode-cp` (n = 265, fkZ =
-1.84), `opencode` (n = 16, fkZ = -1.58), and `hermes` (n = 19, fkZ = -1.23)
all share openclaw's sign and fail to reject. The CHANGELOG calls this a
"qualitatively consistent corpus-wide settling pattern." That reading is
defensible: four out of six sources show first-half-more-dispersed
directional fkZ, and the only strongly significant rejection in the
opposite direction is a single source whose tenure (72 days) is the second
longest in the panel and whose volume profile is an order of magnitude
larger than openclaw / opencode / hermes. The corpus-wide settling
hypothesis is consistent with a story where most sources go through an
onboarding burst that exits within their first ~10 sessions, while
claude-code is large enough and tenured enough to have crossed into a
regime change in its second half that the small-n sources have not lived
long enough to encounter.

## 4. What the n = 16 / n = 19 rows are actually doing

A defensible critique of putting opencode (n = 16) and hermes (n = 19) on
the same table as vscode-cp (n = 265) is that the FK chi-squared(1)
asymptotic is calibrated for n in roughly [10, 50] in the canonical
Conover-Johnson-Johnson 1981 simulation tables — opencode is at the
inclusive lower bound. Two responses are honest. First, the report does not
gate-rank these rows: opencode and hermes are below the alpha = 0.05 line
and the report does not act on them. Second, the table is sorted
`fkZAbsDesc`, which puts the strongest evidence at the top; the smaller-n
rows are visible primarily as **sign-direction witnesses** to the corpus-wide
settling pattern, not as independent rejection events. If the report wanted
to act on those rows, the right move would be a permutation reference
distribution rather than the chi-squared(1) asymptotic — which is a known
extension and not in scope for v0.6.491.

A subtler concern is that the median-centring step is sensitive to the
discreteness of `total_tokens` at low n. For n1 = 8 (opencode), the
within-half median is a single observation or a two-observation average;
the |z| values inherit any quantisation of the raw tokens series. The
half-normal score function is bounded and continuous so this does not
catastrophically distort the test, but it does mean the small-n rows should
be read as directional rather than scaled. The CHANGELOG's framing
("directional but non-significant first-half-more-dispersed signal —
qualitatively consistent across three independent sources") is the right
register.

## 5. Where this leaves the eight-axis closure narrative

Three weeks of W17 work culminated in the axes 181-188 family closure. The
public reading of that closure was: "the eight-axis location-and-scale
statistical battery is complete; downstream axes are shape, tail, and
joint-shape work." Axis-196 forces a small but important amendment to that
reading.

The amendment: axes 181-188 closed the **location-channel inferential
sufficiency** but did not close scale-channel sufficiency under the same
within-half median-centring conditioning. Axes 117, 170, 177, 178 are scale
tests, but they condition on pooled signed values and therefore confound
location and scale at the support level — they are scale tests under the
side condition of equal medians. Axis-196 is the first scale test in the
family that drops that side condition by construction. The corollary is
that the orthogonal decomposition `axis-196 + axis-176` is the **first
two-axis pair** in the entire run that produces a clean 2-bit
location-rejected? scale-rejected? report on the same halves.

That is a non-trivial primitive. It is also the natural anchor for the
v0.6.492 cross-axis joiner that landed the same day:
`classifyFlignerKilleenCliffScaleVsDominanceCompound` (axis-196 + axis-191).
The structural symmetry of that compound — pure scale paired with directional
dominance, producing five mutually-exclusive bivariate buckets — is only
phrasable because axis-196 makes the scale channel an addressable object
rather than an extracted component.

## 6. Citation summary

- Primary: `pew-insights` commit `cb809ca` (release v0.6.491), CHANGELOG
  entry "0.6.491 — 2026-05-05 — Added — daily-token-fligner-killeen-halves
  (axis-196)" with the six-row live smoke table reproduced above.
- Implementation: commit `f61d843` (axis-196 source) and `0d04950` (33 unit
  tests).
- Methodological references named in the CHANGELOG: Fligner & Killeen 1976
  *J. Amer. Statist. Assoc.* 71:210-213; Conover, Johnson & Johnson 1981
  *Technometrics* 23(4):351-361 Table 5 (recommended median-modified form);
  R `stats::fligner.test` default-replacement note.
- Companion follow-on: commit `76cfe4e` (release v0.6.492) shipping the
  scale-vs-dominance compound the day this axis went in.

The two-bit `(BM-rejected?, FK-rejected?)` report on these six sources, read
against the W17 family closure, is what the v0.6.491 axis was built to
produce. The sign disagreement at the top of the live smoke is what makes
the report worth printing.
