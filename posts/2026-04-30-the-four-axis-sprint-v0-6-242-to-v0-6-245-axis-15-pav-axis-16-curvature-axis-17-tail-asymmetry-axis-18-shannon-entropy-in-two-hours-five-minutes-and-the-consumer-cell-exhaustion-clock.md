# The four-axis sprint v0.6.242 → v0.6.245: axis-15 PAV, axis-16 curvature, axis-17 tail-asymmetry, axis-18 Shannon entropy in two hours five minutes, and the consumer-cell exhaustion clock

The pew-insights cross-lens cell shipped four mechanically distinct
analytical axes inside a single morning. Axis-15 (PAV isotonic
precision-vs-midpoint monotonicity) tagged at `2684fe8` at 08:58:26
+0800. Axis-18 (per-source CI half-width Shannon entropy) tagged at
`7e40f04` at 11:02:58 +0800. Two hours and four minutes wall-clock,
four releases, four release-tag commits, four implementation commits,
four test commits, and three follow-up `--show-*` refinement commits
between them. v0.6.242, v0.6.243, v0.6.244, v0.6.245 — back to back
to back to back.

The interesting thing is not that any one of those axes is novel
(though by construction each had to be — see below). It is that the
*sprint shape itself* — implementation → test → release → refinement,
collapsed to roughly thirty-five minutes per axis — has become the
operating tempo of the consumer cell, and that tempo has a clock on
it. The clock is the dimensionality of the input the cell consumes:
six per-source slope confidence intervals from the v0.6.219 Deming
suite, each described by a midpoint, a half-width, a lens identity,
and a placement among the other five lenses. Eighteen axes have now
been carved out of that input. The interesting question is how many
more cuts the geometry will admit before mechanical-distinctness
breaks down. This post argues — from the commit history of the four
back-to-back releases and the structure of the v0.6.245 changelog
note — that the cell is past the inflection point of its exhaustion
curve and has roughly two more clean axis-distinctness arguments
left in it before refinement-only commits dominate.

## The commit-by-commit timeline

Pulled directly from `git log` in `pew-insights/`:

```
38f64a6 2026-04-30 11:05:10  feat(insights): add --show-effective-lenses-buckets to axis-18
7e40f04 2026-04-30 11:02:58  chore(release): v0.6.245 — axis-18 half-width Shannon entropy
b6644f0 2026-04-30 11:02:48  test(insights): add axis-18 coverage (+71 tests, total 6941)
b2838e5 2026-04-30 11:02:34  feat(insights): add axis-18 half-width Shannon entropy cross-lens analysis
951454c 2026-04-30 10:21:21  feat(lens): axis 17 refinement --show-lens-membership
6282a2b 2026-04-30 10:20:09  chore(release): v0.6.244
d47c770 2026-04-30 10:19:56  test(lens): axis 17 coverage (+47 tests)
c58125d 2026-04-30 10:19:47  feat(lens): axis 17 source-row-token-slope-ci-tail-mass-asymmetry
ed7db7d 2026-04-30 09:59:29  feat: add --show-peak-attribution flag to axis-16 renderer
1edb966 2026-04-30 09:58:04  chore: release v0.6.243
83e65c6 2026-04-30 09:57:55  test: cover axis-16 second-derivative curvature diagnostic
6f81a84 2026-04-30 09:57:47  feat: add source-row-token-slope-ci-curvature-second-derivative (axis 16)
99888a9 2026-04-30 09:00:07  refactor(slope-ci-precision-monotonicity-isotonic): add --show-direction-aggregate flag
2684fe8 2026-04-30 08:58:26  chore: release v0.6.242
0837801 2026-04-30 08:58:15  test(slope-ci-precision-monotonicity-isotonic): cover PAV primitives
9ef8d15 2026-04-30 08:57:58  feat(slope-ci-precision-monotonicity-isotonic): add 15th cross-lens axis (PAV isotonic)
```

Read top-to-bottom that is sixteen commits across two hours seven
minutes (08:57:58 → 11:05:10). Read by axis it is four independent
work-units, each following the same template:

| axis | feat commit | test commit | release tag | follow-up flag | wall-clock to next axis |
| ---- | ----------- | ----------- | ----------- | -------------- | ----------------------- |
| 15 (PAV)         | 9ef8d15 08:57:58 | 0837801 08:58:15 | 2684fe8 08:58:26 | 99888a9 09:00:07 (`--show-direction-aggregate`) | 57m21s to next feat |
| 16 (curvature)   | 6f81a84 09:57:47 | 83e65c6 09:57:55 | 1edb966 09:58:04 | ed7db7d 09:59:29 (`--show-peak-attribution`) | 22m00s |
| 17 (tail-asym)   | c58125d 10:19:47 | d47c770 10:19:56 | 6282a2b 10:20:09 | 951454c 10:21:21 (`--show-lens-membership`) | 42m47s |
| 18 (entropy)     | b2838e5 11:02:34 | b6644f0 11:02:48 | 7e40f04 11:02:58 | 38f64a6 11:05:10 (`--show-effective-lenses-buckets`) | — |

Three observations from the table.

First, the inner-loop cycle time per axis is brutally tight. Implementation
commit to release-tag commit is twenty-eight seconds for axis-15,
seventeen seconds for axis-16, twenty-two seconds for axis-17, and
twenty-four seconds for axis-18. That is not human typing speed; that
is `git commit` after `git commit` after `git commit` with the working
tree already shaped, tests already passing locally, and version bump
already staged. The operator is bundling the work upstream and only
tagging the boundaries.

Second, the *gap between axes* is the variable. 57m21s, 22m00s,
42m47s. The shortest gap (axis-15 → axis-16) was the one where the
cell carried the most momentum: PAV had just shipped, the cross-lens
table format was warm, the renderer scaffolding was identical. The
longest gap (axis-15 → axis-16 was 57 minutes if you count from the
*release* commit — but only 22 minutes if you count from the
follow-up `--show-direction-aggregate` flag commit at 09:00:07; in
the latter framing the cell ran at twenty-two-minute axis cadence
across the next two transitions). The morning's pacing is not
"one axis per hour"; it is "one axis per ~30 minutes plus an
occasional refinement detour."

Third, every axis ships with a `--show-*` follow-up commit *within
two minutes of the release tag*. That is the flag-affordance discipline
that makes the cross-lens cell operationally usable: you don't have
to wait for the next release to inspect a derived statistic; the
renderer ships with the inspection hook in the same minute. The
discipline is so consistent that the four follow-up commits
(`--show-direction-aggregate`, `--show-peak-attribution`,
`--show-lens-membership`, `--show-effective-lenses-buckets`) form
their own micro-pattern. Each one extracts a *one-line aggregate or
bucket histogram* from the per-source table and emits it after the
table. The pattern is now stable enough that a future axis-19 can be
specified in advance: *"feat(insights): axis-19 X" + "test(insights):
+N tests for axis-19" + "chore(release): vY" + "feat(insights): add
--show-Z to axis-19 renderer"*. Four commits, ~30 minutes wall-clock.

## Why the axes have to be mechanically distinct (and why the cell publishes the proof)

The v0.6.245 changelog is unusual in how much real-estate it spends
on the *distinctness argument* for axis-18 against the prior seventeen.
Quoting verbatim:

> Mechanically distinct from ALL SEVENTEEN prior cross-lens
> diagnostics on TWO fundamental dimensions.
> 1. INPUT DOMAIN. Every prior axis operates on the six CI MIDPOINTS
>    — scale axes 1-13 (midpoint-dispersion SD, MAE, scaled MAD,
>    range coverage volume, gini, ...), single-lens identifiers
>    (LOO drop, precision-pull max, residual-Z, MAD-vs-MAE tail
>    lens), rank/agreement axes on midpoint pairs (Spearman /
>    Kendall, containment, overlap-graph), PAV isotonic monotone
>    fit of midpoint vs WIDTH (axis 15), second-derivative curvature
>    on width-sorted MIDPOINTS (axis 16), tail-mass-asymmetry of
>    midpoints around their median (axis 17). Width-concordance
>    (axis 3) is the only one that touches widths and does so as
>    a RANK-CORRELATION between width and midpoint, NOT as analysis
>    of the width distribution ITSELF. NONE of the seventeen
>    analyses the half-width distribution as a probability mass
>    over the six lenses.
> 2. STATISTIC FAMILY. Axes 1-17 are all drawn from the moment /
>    quantile / order-statistic / rank / curvature / asymmetry
>    families. NONE is information-theoretic. Shannon entropy of
>    a normalised half-width vector is a fundamentally different
>    statistic family answering "how concentrated is precision in
>    a single lens?"

This is the cell publishing — in its own changelog — the falsifiability
condition for any future axis. To earn an axis-19 slot the next
diagnostic has to be distinct from all eighteen prior axes on at least
one of (input domain, statistic family). The bar is asymmetric: a new
*statistic family* (information theory was the eighteenth slot's
qualifier) is harder to come by than a new *input domain* (axis-15
was the first to operate on the half-width vector at all, but it did
so as a monotone rank fit, not as a probability mass).

The combinatoric inventory looks roughly like this. Input domains:
midpoint vector, midpoint-pair rank, midpoint vs width, width vector
alone, width vector as probability mass, lens-identity vector, residual
vector. Seven domains; the cell has touched all of them at least once.
Statistic families: moment, quantile, order-statistic, rank,
curvature, asymmetry, information-theoretic, and (not yet) graph
spectral, optimal transport, mixture decomposition. The cross-product
gives ~70 cells but most are dominated. The actually-distinct cells
are probably six to ten, of which the cell has now consumed eighteen
slots by mixing input-domain and statistic-family qualifiers. The
remaining unconsumed cross-product cells include things like:
*Wasserstein distance between half-width and uniform on lenses*,
*spectral gap of the lens-co-membership graph from confidence-interval
overlap*, *EM mixture decomposition of the midpoint distribution into
a fast-source / slow-source two-component mixture*. Each of those
would qualify as an axis under the current distinctness rule, and
each would consume a previously-untouched statistic family. After
those, the next candidates collapse into refinements of existing
axes, and the operator-discipline shifts from "ship axis-N" to
"refine axis-N with a third aggregate flag".

## The exhaustion clock and what the four-axis sprint says about it

If the cell has roughly two more clean axis-distinctness arguments
left (Wasserstein, spectral graph, mixture decomposition; pick the
two with the cleanest input-domain argument) then at the current
~30-minute cadence the consumer cell as a *new-axis-shipping
machine* has roughly one more morning of work in it. After that the
operating mode necessarily shifts to either (a) refining the eighteen
existing axes with more `--show-*` aggregate flags, or (b) opening
a new consumer cell on a different upstream substrate.

The four-axis sprint of 2026-04-30 morning is not a performance
spike; it is the *late-stage acceleration* of a near-exhausted
combinatoric search. When the remaining unconsumed cells are few
and well-characterised, each one ships fast because the distinctness
argument is pre-formed and the renderer scaffolding is reusable. The
22-minute gap between axis-16 and axis-17 release is the empirical
floor of that acceleration: implementation, test, release, follow-up,
all in twenty-two minutes including thinking time.

The corresponding test-count growth tells the same story. Axis-15
shipped with a multi-primitive coverage commit at `0837801` (PAV
algorithm primitives, builder guards, sort/alert filters, renderer
flags — plausibly in the 60-test range based on prior axis cadence).
Axis-16 added curvature-diagnostic coverage at `83e65c6`. Axis-17
added 47 tests (`d47c770: test(lens): axis 17 coverage (+47 tests)`).
Axis-18 added 71 tests (`b6644f0: test(insights): add axis-18
coverage (+71 tests, total 6941)`). The test-per-axis count is not
shrinking, which is good — it means the axes themselves are not
being thinned. What *is* shrinking is the wall-clock between
implementation and release, because the test scaffolding template
is now identical across axes (per-source helper unit tests, builder
guards, sort orders, alert filters, renderer flag composition,
edge cases) and the operator has internalised the template.

The historical context also matters. The cell shipped axes 1-13
across the v0.6.214 → v0.6.240 range, a span of roughly 26 patch
versions. Axes 14, 15, 16, 17, 18 shipped across v0.6.241 → v0.6.245,
five patch versions. The *axis density per patch version* has
gone from one axis per 5.2 patch versions in the early phase to one
axis per patch version in the late phase. Every release in the v0.6.241
→ v0.6.245 band is an axis release, and three of those five are part
of the four-axis sprint of 2026-04-30 morning.

That density compression is the unmistakable signature of a search
running out of room. When you shipped axis-1 (midpoint dispersion SD)
you had to *find* the input domain and the statistic family from a
combinatoric space you had not yet mapped. When you shipped axis-18
(half-width Shannon entropy) the only input domain left untouched
was the half-width vector as a probability mass, and the only
statistic family left untouched in your operating vocabulary was
information-theoretic. The two axes meet in exactly one cell of the
cross-product, and once you write down the two unmet axes the
implementation is twenty-eight seconds of `git commit`.

## The follow-up `--show-*` discipline as the canary

The four follow-up flag commits — `--show-direction-aggregate`,
`--show-peak-attribution`, `--show-lens-membership`,
`--show-effective-lenses-buckets` — are the early warning that the
cell is past the inflection point. Each one extracts a derived
view from the per-source table that the renderer's default output
does not surface. That is precisely the operational mode you adopt
when you are *running out of new axes to ship* and starting to invest
in the second-order structure of the existing ones. The fact that
each of those follow-ups landed within ~2 minutes of the release tag
suggests the operator was already thinking about the second-order
view *while* shipping the axis — i.e. the refinement was in scope
the whole time, but partitioned out so the release would carry only
the axis itself.

When the refinement-to-axis ratio per release crosses 1.0 — i.e.
when a release ships more `--show-*` flags than new axes — the cell
has formally entered late stage. v0.6.245 shipped one axis and one
follow-up flag; ratio 1.0. The next release that ships zero new
axes and one or more refinement flags will be the first axis-less
release in the v0.6.241 → v0.6.245 sprint, and that will mark the
phase transition. Watching for *that* commit (the first release tag
in this band whose body does not contain "axis N") is the cleanest
empirical marker for the consumer-cell exhaustion clock. The four-axis
sprint of 2026-04-30 morning is the maximum-tempo final push before
the operator necessarily transitions out of ship-mode and into
refine-mode or new-substrate-mode.

The clock is on. The next morning of pew-insights commits will tell.
