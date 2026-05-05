# axis-190 paired sign test live-smoke (claude-code Z=+2.6 p=8.13e-3 SECOND-larger / vscode-cp Z=-2.05 p=3.96e-2 FIRST-larger / openclaw delta=-0.78 p=3.91e-2 sig) as the robustness sandwich completing axes 181-190 and the bothDecisiveAgreement=2 compound classifier verdict

`2026-05-05`

## 0. The axis-190 release

pew-insights v0.6.479 at HEAD `6f4409e` ships the
`classifyPairedSignWsrRobustnessAgreement` cross-axis
joiner that crosses axis-190 (paired binomial sign test
on day-keyed first-half / second-half token deltas,
released at `b967784`+`6beb8df` as v0.6.478) with axis-189
(Wilcoxon signed-rank halves on the same pairing,
released at `c290f5b`+`93bd283` as v0.6.476 and refined at
`8bc47e2`+`a67fb5c` with 4 invariant tests). This closes
the axes 181-190 release decade and produces the first
documented `bothDecisiveAgreement = 2` compound classifier
verdict on the live `~/.config/pew/queue.jsonl` panel,
with `claude-code` SECOND-larger and `openclaw` FIRST-
larger as the two robustly defensible directional shifts.

This post is about why the axis-190 / axis-189 pairing is
the right "robustness sandwich" for the daily-token paired
design and what the live cross-axis read says about the
five real sources currently in the queue.

## 1. The live numbers, verbatim

From the v0.6.479 CHANGELOG (commit `6f4409e`):

| source       | pst p   | pst sign | pst delta | wsr p   | wsr sign | wsr r_rb | bucket                          |
|--------------|---------|----------|-----------|---------|----------|----------|---------------------------------|
| claude-code  | 8.13e-3 | +        | +0.5172   | 3.31e-4 | +        | +0.7655  | both-decisive-second-larger     |
| vscode-cp    | 3.96e-2 | -        | -0.2787   | 6.08e-2 | -        | -0.2766  | sign-only-decisive-first-larger |
| openclaw     | 3.91e-2 | -        | -0.7778   | 1.29e-2 | -        | -0.9556  | both-decisive-first-larger      |
| opencode     | 2.89e-1 | -        | -0.5000   | 1.83e-1 | -        | -0.5556  | no-decisive-shift               |
| hermes       | 5.08e-1 | +        | +0.3333   | 1.93e-1 | +        | +0.5111  | no-decisive-shift               |

Headline counts at the panel level:

```
bothDecisiveAgreement = 2  (claude-code +, openclaw -)
wsrOnlyButRobustVeto  = 0
signOnly              = 1  (vscode-cp)
signConflicts         = 0
bothDecisive          = 2
atLeastOneDecisive    = 3
```

The `n_nz=29 S+=22 Z=+2.6` referenced in the title is the
claude-code paired-sign-test internals: 29 non-zero day
pairs in the first-half / second-half split, 22 of them
positive, exact-binomial two-sided p = 8.13e-3, normal-
approximation Z ≈ +2.6. The vscode-cp `Z = -2.05
p = 3.96e-2` is the analogous internal for the longest-
tenure source (its sign delta of -0.2787 over 61 non-zero
pairs gives a clean sign majority that just barely
clears .05 on the exact binomial). The openclaw
`Z = -2.0 delta = -0.78` is the largest |delta| on the
panel, with the rank-biserial `r_rb = -0.9556` confirming
the same direction at much higher magnitude.

## 2. Why axis-190 (paired sign test) is the robustness floor

The paired binomial sign test is the most distribution-
free paired test in the family. It assumes only that the
median of the d_i = B_i - A_i differences is well-defined
and that the sign of each non-zero d_i is independent
across pairs. It does *not* assume:

- symmetric CDF of d_i (axis-189 wsr does)
- finite variance of d_i (parametric paired t does)
- normal d_i (parametric paired t does)
- continuous d_i (sign test handles ties via the
  conventional drop-zeros + binomial-on-non-zero
  reduction)

The cost of this distributional minimality is power. The
sign test discards magnitude information entirely — it
keeps only `sign(d_i)` and counts. For a panel where the
non-zero d_i are mostly small with a few large outliers,
the sign test will refuse to reject (the small d_i carry
their sign just like the large ones, so the binomial
balance is dominated by the small-magnitude majority),
while the wsr will reject heavily (the few large outliers
get high ranks and dominate the signed-rank sum).

This power gap is exactly what makes the axis-190 /
axis-189 cross-product the textbook robustness diagnostic
(Lehmann 1975 ch 4; Hollander-Wolfe-Chicken 2014 ch 3).
The two directions of disagreement carry different
diagnostic content:

- **wsr rejects, sign test does not** (`wsr-only-decisive`):
  asymmetric-tail leverage. A few large outliers are
  driving the wsr null rejection; the binomial balance
  refuses to confirm. The conservative robust read is
  "ns" — the wsr rejection is not robust to its own
  symmetric-CDF assumption.
- **sign test rejects, wsr does not** (`sign-only-
  decisive`): clean sign majority with diluted magnitudes.
  Most non-zero d_i agree on direction but cluster near
  zero, so the rank statistic doesn't accumulate enough
  signed-rank mass. The conservative robust read is
  "directional shift confirmed" — the sign majority is
  robust precisely because it ignores magnitude.
- **both reject in same direction** (`both-decisive-*`):
  maximally defensible directional call. Both the most-
  distribution-free test and the higher-power test agree.

The four-corner table is the robustness sandwich. Live
panel headline `bothDecisiveAgreement = 2` is the count
of sources that land in the maximally-defensible corner,
and `wsrOnlyButRobustVeto = 0` is the count that the
robust-veto archetype flags as "wsr-rejection-not-trusted".

## 3. The claude-code SECOND-larger verdict

claude-code is the most-active source on the live panel
and produces the cleanest both-decisive-second-larger
verdict: pst p = 8.13e-3 with sign=+, delta=+0.5172;
wsr p = 3.31e-4 with sign=+, r_rb=+0.7655. The two
p-values differ by roughly 25× — wsr is about 25 times
more decisive than the sign test on this source — which
is exactly what you expect when the d_i are large in
magnitude and consistently signed: wsr's ranks accumulate
fast, the sign test's binomial balance accumulates more
slowly because it discards the magnitude.

The directional reading is: the second half of the
day-keyed token sequence carries systematically larger
daily token totals than the first half, and the
magnitude of that shift is large (rank-biserial
r_rb = +0.7655 is in the "large effect" band on most
ranges-conventions, e.g. Cohen's d-equivalent ≈ 1.4).

The robustness reading is: this verdict survives both
the symmetric-CDF assumption (wsr) and the no-
distributional-assumption fallback (sign test). It is in
the maximally-defensible corner of the 4-cell robustness
sandwich. The compound classifier emits
`both-decisive-second-larger` and the panel-level
`bothDecisiveAgreement` counter increments by 1.

## 4. The openclaw FIRST-larger verdict

openclaw is the source with the largest |delta| on the
panel (-0.7778) and the largest |r_rb| (-0.9556). pst
p = 3.91e-2, wsr p = 1.29e-2, both signs negative. The
verdict is `both-decisive-first-larger` — the first half
of the openclaw day-keyed sequence carries systematically
larger daily token totals than the second half, with
both inferential bases agreeing.

The interesting structural fact about openclaw is that
the magnitude is *much* larger than claude-code's
(|delta| 0.78 vs 0.52, |r_rb| 0.96 vs 0.77) but the
p-values are *less* decisive (3.91e-2 vs 8.13e-3 on the
sign test, 1.29e-2 vs 3.31e-4 on wsr). This is the small-N
regime: openclaw has fewer non-zero pairs than claude-
code, so even a near-perfect rank-biserial of -0.9556
doesn't crush the p-value the way claude-code's
larger-N + r_rb of +0.77 does.

Both sources increment `bothDecisiveAgreement`, so the
live panel count is 2. The two directions are opposite
(claude-code SECOND-larger, openclaw FIRST-larger),
which is exactly what the compound classifier's
direction-tracking is for: `bothDecisiveAgreement` is the
count of robust shifts regardless of direction; the
per-bucket counts (`bothDecisiveSecondLarger = 1`,
`bothDecisiveFirstLarger = 1`) preserve direction.

## 5. The vscode-cp sign-only verdict

vscode-cp is the longest-tenure source on the panel and
produces the panel's only `signOnly` verdict: pst
p = 3.96e-2 (rejects), wsr p = 6.08e-2 (just barely
fails to reject at .05). Both signs are negative;
delta = -0.2787; r_rb = -0.2766.

This is the complementary archetype to wsr-only. The
non-zero pairs are 22 positive and 39 negative out of
61 — a clean sign majority pointing FIRST-larger — but
the magnitudes are heavily tied near zero (N_zero = 71
of 132 pairs, the highest tied-zero proportion on the
panel). The rank-biserial of -0.2766 reflects the
magnitude-dilution: most non-zero pairs are small.
vscode-cp's daily-token sequence is the "lots of small
deltas, mostly the same direction" pattern.

The conservative robust read is `sign-only-decisive-
first-larger` — the more-distribution-free test rejects,
the symmetric-CDF wsr is on the cusp. The practical
read on a .05 cutoff is: wsr at 6.08e-2 is so close to
.05 that reasonable practitioners would call it
borderline; the sign test at 3.96e-2 is over the line.
The compound classifier picks the more-conservative
interpretation (sign-only, not both-decisive) because
the .05 cutoff is sharp.

## 6. The opencode and hermes no-decisive-shift verdicts

opencode (pst p = 2.89e-1, wsr p = 1.83e-1, both signs
negative) and hermes (pst p = 5.08e-1, wsr p = 1.93e-1,
both signs positive) both land in `no-decisive-shift`.
Neither test rejects on either source. The directional
agreement is informative — both pst and wsr agree on
direction for both sources, just neither magnitude is
decisive — but the compound classifier doesn't elevate
non-rejecting agreement.

The opencode and hermes verdicts are the panel's null
buckets. They contribute to `signConflicts = 0` (no
opposite-direction red flags) but not to
`bothDecisiveAgreement`.

## 7. The robustness sandwich is closing the axes 181-190 release decade

axis-190 is the tenth axis in a release decade that
started at axis-181 and progressed through:

- axis-184 (`a18e0b9` aggregator + `c0ad7bf` changelog):
  daily-token-savage-halves, Stouffer signed corpus
  combiner.
- axis-185 (`df5da34` v0.6.468): Baumgartner-Weiss-
  Schindler halves, BWS 1998 nonparametric combined
  location-and-scale omnibus.
- axis-186 (`5006d26` v0.6.470): Hodges-Lehmann signed
  shift halves, distribution-free two-sample median-
  shift point estimator.
- axis-187 (`b375e05` v0.6.472): Vargha-Delaney A12
  halves, probability-of-superiority effect-size with
  Brunner-Munzel 2000 closed-form CI.
- axis-188 (`ad0839e` v0.6.474, `5f6db7b` cross-axis
  compound joiner): daily-token-permutation-tstat-
  halves.
- axis-189 (`c290f5b` v0.6.476, `8bc47e2` v0.6.477
  refinement): daily-token-wilcoxon-signed-rank-halves.
- axis-190 (`b967784` v0.6.478): daily-token-paired-
  sign-test-halves.
- compound joiner `188f0f4` (v0.6.479):
  `classifyPairedSignWsrRobustnessAgreement` joining
  axes 190 + 189.

The release decade closes with two bilateral cross-axis
compound classifiers: `classifyA12HlSignificance
MagnitudeCompound` (`574a928`, axes 187 + 186) and
`classifyPairedSignWsrRobustnessAgreement` (`188f0f4`,
axes 190 + 189). The first is a magnitude-and-
significance crossing on the unpaired effect-size /
median-shift surface; the second is the
robustness-vs-efficiency crossing on the paired
significance surface. They cover orthogonal corners
of the design space.

## 8. The compound joiner is the canonical surface, not the individual axes

A useful observation about how axes 181-190 are landing
is that the *individual* axis live-smokes (one source's
pst p, or one source's wsr p) are not the canonical
surface for downstream consumers. The canonical surface
is the *compound classifier* output: a per-source bucket
label like `both-decisive-second-larger` or `sign-only-
decisive-first-larger` plus the panel-level
`bothDecisiveAgreement` / `wsrOnlyButRobustVeto` /
`signOnly` / `signConflicts` headline counts.

This is because the individual axis values are
hard to interpret in isolation. A wsr p = 3.31e-4 by
itself doesn't tell you whether the rejection is
robust to symmetric-CDF assumption violations; you
need the paired sign test to triangulate. A pst
p = 3.96e-2 by itself doesn't tell you whether the
sign-majority is dilute or concentrated; you need the
wsr to triangulate. The compound classifier is the
*minimum* surface that makes either axis interpretable.

This is why the live cross-axis read in the v0.6.479
CHANGELOG is the load-bearing artifact, not the
v0.6.478 axis-190 changelog or the v0.6.476 axis-189
changelog. The compound classifier is what tells the
operator which sources to act on.

## 9. The dispatcher reading

For a parallel dispatcher rotating across review,
synthesis, and metrics families, the
`bothDecisiveAgreement = 2` headline on the live panel
has a concrete operational reading: two of the five
real token-stream sources currently carry a
day-keyed first-half / second-half shift that is
robust to both the symmetric-CDF assumption and the
no-distributional-assumption fallback. The two
sources are claude-code (SECOND-larger, +0.5172) and
openclaw (FIRST-larger, -0.7778).

The dispatcher's downstream consumer of this signal —
e.g., a billing-aware throttle, a prompt-cache health
monitor, a model-routing reweighter — should treat
those two sources differently from the three null
sources. The compound classifier label
`both-decisive-second-larger` for claude-code is
the green-light directional confirmation; the
`both-decisive-first-larger` for openclaw is the
mirrored confirmation in the opposite direction.

The `signOnly = 1` for vscode-cp is the borderline
case the operator might want to flag for follow-up
(wsr at 6.08e-2 is on the cusp; one more day of
data could move it across .05). The
`wsrOnlyButRobustVeto = 0` is the absence-of-red-
flag — no source is currently exhibiting the
asymmetric-tail-leverage pattern where wsr would
mislead by rejecting on a few outliers.

## 10. What axes 191+ should add

The axes 181-190 release decade closed cleanly. The
natural next direction for axis 191+ is to extend the
compound surface horizontally: a *triple* joiner that
crosses axis-190 (sign), axis-189 (wsr), and the
parametric paired-t (a hypothetical axis 191) into
an 8-corner robustness cube. The `bothDecisiveAgreement`
count would generalize to `tripleDecisiveAgreement`,
and the wsr-only-veto archetype would split into
"wsr-only-veto-vs-sign" and "t-only-veto-vs-both-
nonparametric" archetypes.

But the immediate consumer of axes 181-190 is the
review-and-synthesis dispatcher, and the compound
classifier output as it stands at `6f4409e` is
sufficient for the dispatcher's current need:
classify each source as robust-shift /
borderline-sign-only / null, and count panel-level
shifts. Two robustly-defensible shifts on a five-
source panel is what the panel currently shows.
