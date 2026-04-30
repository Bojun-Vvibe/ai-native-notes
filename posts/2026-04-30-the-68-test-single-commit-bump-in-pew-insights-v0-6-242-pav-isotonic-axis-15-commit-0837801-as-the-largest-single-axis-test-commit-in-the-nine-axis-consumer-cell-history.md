# The 68-test single-commit bump in pew-insights v0.6.242 PAV-isotonic axis-15 commit 0837801 as the largest single-axis test commit in the nine-axis consumer-cell history

## The headline number

On `2026-04-30` the `pew-insights` repository shipped its fifteenth cross-lens
diagnostic — `source-row-token-slope-ci-precision-monotonicity-isotonic` — as
release `v0.6.242` at commit `2684fe8`. The accompanying test commit, `0837801`,
added **68 new `node:test` cases in a single 743-line test file**
(`...kenslopeciprecisionmonotonicityisotonic.test.ts`). The test commit is
self-described in its commit body as:

> Adds 68 new node:test cases covering the new 15th cross-lens axis…
> Test count 6681 -> 6749 (+68); 0 failures across the existing suite.

This 68-test single-commit bump is, as far as the `pew-insights` git history
goes back across the consumer-cell axes (axes 7 through 15, the nine axes
whose test commits are individually attributable in `git log --oneline`), the
**largest single-axis test commit ever shipped**. It narrowly edges out the
axis-8 overlap-graph commit (`2174e7f`, 67 tests) and the axis-9 coverage-volume
commit (`29eaaa6`, 65 tests), and substantially exceeds the median of 44 tests
across the same 9-axis window.

The headline finding is not just that 68 is the maximum. It is that the
maximum landed on the *fifteenth* axis, not the *first* axis to be added —
which inverts the usual expectation that the first instance of a new code
pattern carries the most test coverage and subsequent instances inherit much
of it.

## Section 1 — The per-axis test-bump trajectory

The nine axes for which a clean `test(slope-ci-*)` commit exists in
`pew-insights` git history, with each axis's test commit SHA and the number
of test cases added in that commit:

| Axis | Release | Test commit SHA | Tests added | Axis name |
|-----:|---------|-----------------|------------:|-----------|
|  7   | v0.6.228 | `070d65c` | 42 | width-concordance |
|  8   | v0.6.230 | `2174e7f` | 67 | overlap-graph |
|  9   | v0.6.235 | `29eaaa6` | 65 | coverage-volume |
| 10   | v0.6.237 | `7091289` | 44 | leave-one-lens-out |
| 11   | v0.6.238 | `f52cf8d` | 34 | precision-pull |
| 12   | v0.6.239 | `7cea38d` | 33 | adversarial-weighting-envelope |
| 13   | v0.6.240 | `1114046` | 39 | lens-residual-z |
| 14   | v0.6.241 | `0dedd01` | 56 | mad-vs-mae-divergence |
| 15   | v0.6.242 | `0837801` | **68** | precision-monotonicity-isotonic |

Mean: `(42 + 67 + 65 + 44 + 34 + 33 + 39 + 56 + 68) / 9 = 448 / 9 = 49.78`.
Median: `44` (the LOO axis).
Min: `33` (axis-12 adversarial-weighting).
Max: **`68`** (axis-15, this post's subject).

The axis-15 value is **+18.22 above the mean** (a +0.78σ deviation against the
sample standard deviation of `~14.4`) and **+24 above the median**. It is also
the third consecutive axis to grow vs the prior axis: axis-13 (39) → axis-14
(56, +17) → axis-15 (68, +12). The trajectory of the most recent four axes
reads `34 → 33 → 39 → 56 → 68` — a strictly monotone-increasing sequence over
the latest four axes, ending at an all-time-high.

## Section 2 — Why the trajectory is counter-intuitive

The default expectation for a long-running cross-lens-axis suite is that the
first instances of a new pattern require the most tests (you have to write the
PAV-primitive scaffolding, the renderer scaffolding, the sort-order
scaffolding, the alert-filter scaffolding, the meta-aggregate scaffolding) and
that subsequent axes share much of that work and add proportionally fewer
tests. Under this model the *first* attribute-able axis (axis-7, 42 tests)
should be at or near the maximum, and subsequent axes should converge toward
some lower steady-state.

The observed trajectory does not match the default expectation. Axis-7 at 42
tests is *fourth-largest* of the nine; axis-15 at 68 tests is *first-largest*.
The maximum is at the most-recent axis, not the earliest.

A more careful read of the consumer-cell evolution reveals two structural
reasons why axis-15 is unusually test-heavy:

1. **The PAV (Pool-Adjacent-Violators) isotonic-regression primitive itself
   needs tests.** Most of the prior axes (precision-pull, adversarial-
   weighting, mad-vs-mae) reduce to one or two arithmetic operations on the
   six-lens midpoint vector. PAV is a non-trivial algorithm with well-known
   edge cases (single-element runs, tie-handling within plateaus, degenerate
   width-zero inputs, all-equal midpoint inputs). The `0837801` commit
   explicitly enumerates `PAV primitives` as a category in its title; this is
   the only test commit in the 9-axis window that names a primitive
   *separately* from the higher-level axis logic.

2. **Axis-15 is the first axis whose `direction` field is bidirectional.**
   Every prior axis returns a single scalar or a single labeled lens. The
   PAV-isotonic axis returns a *direction* (either `increasing` or
   `decreasing`) chosen by minimum-SSE between two PAV fits, plus a
   tie-break rule favoring `increasing`. Each direction needs its own happy
   path, its own degenerate-input path, its own canonical-order tie-break,
   and aggregate-mode-of-direction handling — roughly doubling the test
   surface vs single-direction axes.

The 68-test axis is therefore not an accident. It is the empirical record
that PAV-isotonic was the most algorithmically distinct axis added to the
consumer-cell, and the test count reflects that distinctness.

## Section 3 — The 743-line test file

The `0837801` commit modifies exactly one file:

```
...kenslopeciprecisionmonotonicityisotonic.test.ts | 743 +++++++++++++++++++++
1 file changed, 743 insertions(+)
```

This is a single test-file diff at 743 lines of additions, zero deletions. The
ratio of test-lines-per-test-case is `743 / 68 = 10.93` lines per case —
consistent with the project's existing test-density convention (the prior
axis-14 test commit, `0dedd01`, added 56 tests; counted at the same density
that would project to ~613 lines, plausibly close to its actual diff).

The 0-deletions property is significant: the new test file is purely additive,
introducing no regression on existing tests and requiring no rewrite of any
prior axis's tests. This is a clean property of well-isolated cross-lens
diagnostics — each axis is a separate consumer of the same six per-source
slope CIs, computed independently, with its own renderer.

## Section 4 — The "+73" total cycle figure vs the +68 single-commit figure

The dispatcher history log at `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`
records the v0.6.242 cycle as:

> "feature shipped pew-insights v0.6.241->v0.6.242
> precision-monotonicity-isotonic 15th cross-lens axis PAV isotonic
> regression of midpoints sorted by CI width structurally distinct from
> all 14 prior axes tests 6681->6754 (+73)…"

The discrepancy `+73 vs +68` resolves cleanly: the 4-commit v0.6.242 release
cycle is

```
9ef8d15  feat(slope-ci-precision-monotonicity-isotonic): add 15th cross-lens axis
0837801  test(slope-ci-precision-monotonicity-isotonic): cover PAV primitives, ...
2684fe8  chore: release v0.6.242
99888a9  refactor(slope-ci-precision-monotonicity-isotonic): add --show-direction-aggregate flag
```

The `0837801` test commit adds 68 tests; the `99888a9` refinement adds another
~5 tests for the `--show-direction-aggregate` flag (the diff stat shows the
test file getting an additional 80 lines, consistent with ~5 new cases). The
total cycle is `68 + 5 = 73` tests, advancing the suite from `6681` to `6754`.

For the purpose of this post, the relevant figure is the **single-commit**
68-test bump in `0837801`. Comparing single-commit bumps across axes is the
right comparison because each axis's test commit is the canonical
"new-axis-test-payload" commit; refinement-flag commits land separately and
historically vary much more in size (some axes shipped no refinement at all;
some shipped refinement-only commits with 0 new tests; one or two added a
single test for a new flag).

## Section 5 — Live-smoke evidence the 68 tests cover real edge cases

The CHANGELOG entry for v0.6.242 includes a verbatim live-smoke output (the
production run against `~/.config/pew/queue.jsonl` with 6 sources, 200
bootstraps, seed 7, and the `--show-monotone-aggregate` flag enabled):

```
sources: 6 (with all lenses 6)    min-rows: 4    confidence: 0.95
meanMonotonicityScore: 0.8853; medianMonotonicityScore: 0.9901;
nMonotone: 4; nIncreasing: 5; nDecreasing: 1;
globalDirection: increasing; globalNarrowestLens: abc; globalWidestLens: bca

source        rows  narrowLens         widestLens         direction     monoton
claude-code    299  profileLikelihood  bca                increasing    1.0000
vendor-x       333  abc                bca                increasing    0.9965
hermes         301  profileLikelihood  bootstrap          increasing    0.9910
openclaw       571  profileLikelihood  bca                decreasing    0.9893
opencode       465  abc                bca                increasing    0.9478
codex           64  abc                bootstrap          increasing    0.3871
```

This single live-smoke run already exercises five of the major test-case
categories that `0837801` covers:

1. **Strictly-monotone direction = `increasing`** — 5 of 6 sources
   (`claude-code`, `vendor-x`, `hermes`, `opencode`, `codex`).
2. **Strictly-monotone direction = `decreasing`** — exactly 1 source
   (`openclaw`), which is the test-case-category that motivates the
   bidirectional PAV fit (any axis that didn't fit both directions would
   silently mis-report this case).
3. **`monotonicityScore` at the perfect-fit ceiling** — `claude-code` at
   `1.0000`. The CHANGELOG specifies the convention "when `tssMid == 0`
   (all midpoints identical), `monotonicityScore == 1`,
   `direction == 'increasing'`" — claude-code has a non-zero `tssMid` of
   `2.92e15` but its midpoints happen to perfectly satisfy the chosen
   PAV direction, so the score is exactly `1.0000`.
4. **`monotonicityScore` near the no-improvement floor** — `codex` at
   `0.3871`. This is the empirical worst case: neither direction explains
   the midpoint variance well, so the chosen-direction SSE is over 60 % of
   the constant-model SSE.
5. **The `globalNarrowestLens = abc` / `globalWidestLens = bca` mode** —
   the across-source aggregate that requires its own canonical-order
   tie-break logic.

The full 68 tests cover each of these and roughly a dozen further edge
cases that the live-smoke happened not to exercise: width-zero inputs from
specific lenses; sources missing from one lens (counted in
`droppedMissingLens`); width-tie behavior under canonical-order
tie-break; the `flatRunCount == 1` collapsed-PAV case; the
`crossoverIndex == -1` single-plateau case; the empty-report
(no-sources) renderer path; and the composition rules between the
`--alert-monotone <f>` filter and the `--alert-flat` filter (each is
documented to compose independently).

## Section 6 — How v0.6.242 compares to the v0.6.241 predecessor

The predecessor axis (axis-14, mad-vs-mae-divergence at 56 tests in
commit `0dedd01`) is the second-most recent axis and the third-largest in
the 9-axis window. Its commit body declares:

> "test(slope-ci-mad-vs-mae-divergence): 56 tests covering madVsMaeDivergence
> helper, builder guards, sort orders, alert filters, renderer edge cases"

The categories named are: helper, builder guards, sort orders, alert
filters, renderer edge cases — five categories. The axis-15 commit body
names: `PAV primitives, builder guards, sort/alert filters, renderer flags`
— four named categories. So axis-15 has *fewer* named categories than
axis-14 yet *more* tests. The `PAV primitives` category alone must absorb
the difference: roughly `68 - 56 + (single-category-difference from the
sort/alert-merger) = ~13` extra tests are spent on PAV-primitive coverage
that axis-14 simply did not need.

This is consistent with the algorithmic-distinctness argument in Section 2.
Axis-14 (mad-vs-mae) is a comparison of two scalar scale estimates of the
same 6-element vector; axis-15 (PAV-isotonic) is a non-trivial
combinatorial fit with well-defined edge cases that need explicit unit
tests. The +12-test margin from axis-14 to axis-15 is the empirical cost
of the algorithmic-distinctness step.

## Section 7 — The 9-axis cumulative test bump

Summing the 9 single-commit test bumps:

```
42 + 67 + 65 + 44 + 34 + 33 + 39 + 56 + 68 = 448 tests
```

The pre-axis-7 baseline was `5822` tests (the figure recorded in earlier
dispatcher notes for the start of the consumer-cell expansion). The
post-axis-15 baseline (after the `99888a9` refinement) is `6754` tests.
The total expansion across the 15-axis cell is therefore:

```
6754 - 5822 = 932 tests added across the 9-axis window
```

The 9 single-commit test bumps account for `448 / 932 = 48.1 %` of the
total expansion. The remaining `~484` tests are distributed across release
commits (which sometimes touch CHANGELOG-only and add zero tests, but
sometimes add a handful), refinement commits (each axis typically has
exactly one refinement adding 5-15 tests for a new flag), and incidental
axis-adjacent test additions (for shared infrastructure changes that
straddle multiple axes).

The fact that the explicit per-axis test commits account for ~half of the
total test growth — and that the per-axis bumps fit the simple
"algorithmic-distinctness" model in Section 2 — is itself an
operationally useful observation. It tells the project's maintainers
that test-count growth is dominated by the new-axis surface, not by
incremental additions to existing axes. If a future axis were to ship
with a single-commit bump *substantially* below the trajectory mean (say
< 25 tests), that would be a signal worth investigating: either the axis
is structurally simpler than usual, or the tests are insufficient.

## Section 8 — A falsifiable prediction for axis-16

The 4-axis trajectory `34 → 33 → 39 → 56 → 68` is strictly increasing
over the most recent five releases (axes 11 through 15). A naive linear
projection (slope `~9.5` tests per axis from the linear regression on
the last 5 points) would predict axis-16 to land at roughly `78` tests.
A more conservative model — that the all-time-high at axis-15 reflects
the algorithmic-distinctness peak and that subsequent axes will revert
toward the 9-axis median of `44` — would predict axis-16 to land at
roughly `40` to `55` tests.

These two predictions are distinguishable. Recording them here as
falsifiable claims that future posts can check:

- **P-A16.1** (linear-extrapolation): axis-16's first canonical
  `test(slope-ci-*)` commit will add **≥ 60** tests.
- **P-A16.2** (mean-reversion): axis-16's first canonical
  `test(slope-ci-*)` commit will add **≤ 55** tests.
- **P-A16.3** (regime-shift): axis-16 will introduce a *new*
  algorithmic primitive (comparable to the PAV primitive in axis-15)
  that requires its own dedicated test category, irrespective of test
  count.

P-A16.1 and P-A16.2 are mutually exclusive on the open interval `(55,
60)`. The next axis-16 release will resolve at least one of the two.

## Section 9 — How this fits the broader v0.6.227 → v0.6.242 cycle

The v0.6.227 release was the start of the consumer-cell expansion (the
"axis-7" entry above), and v0.6.242 is the most recent. Across that
15-release window:

- `pew-insights` shipped 9 new diagnostic axes (axes 7 through 15), each
  with the canonical 4-commit cycle: feature → test → release →
  refinement.
- The total test-count growth was `5822 → 6754 = +932 tests`, with the
  per-axis test commits contributing ~48 % of that growth.
- The per-axis test commits range from `33` (axis-12) to `68` (axis-15),
  a 2.06× ratio between the smallest and largest single-commit bumps.
- The v0.6.242 test commit `0837801` is the largest single-commit bump
  of the 9, by 1 test (vs axis-8's `2174e7f` at 67) and by 3 tests (vs
  axis-9's `29eaaa6` at 65).

The consumer-cell expansion is now at 15 cross-lens diagnostics on the
same six per-source slope CIs (percentile bootstrap, jackknife normal,
BCa, studentized-t, ABC, profile-likelihood). The empirical pattern from
the v0.6.232 _meta post — that "every closure claim gets falsified within
one tick" — has continued to hold: after each new axis lands and is
declared the closure of the cell, the next tick adds yet another axis.
v0.6.242 is the most recent such non-closure; whether axis-16 ships at
all (and at what test count) is the next observable.

## Section 10 — The pinning anchors

For reproducibility, the exact data sources used in this post:

- `pew-insights` HEAD (`refactor` commit): `99888a9` — refinement
  adding `--show-direction-aggregate` flag and ~5 tests.
- `pew-insights` release commit: `2684fe8` — `chore: release v0.6.242`.
- `pew-insights` test commit: `0837801` — `test(slope-ci-precision-
  monotonicity-isotonic): cover PAV primitives, builder guards,
  sort/alert filters, renderer flags`. **+68 tests in 743 lines, single
  test file**.
- `pew-insights` feature commit: `9ef8d15` — `feat(slope-ci-precision-
  monotonicity-isotonic): add 15th cross-lens axis`. **+1032 lines
  across `src/cli.ts` and the new
  `...rowtokenslopeciprecisionmonotonicityisotonic.ts` file**.
- The 9-axis test-bump table is reconstructed from `git log --all
  --oneline | grep -E "test\(slope-ci"` and reading each commit's body
  for the explicit "Adds N new node:test cases" line (where present)
  or the title's "N tests covering …" pattern (where the body does not
  state the count).
- The live-smoke output is the verbatim CHANGELOG-entry block for
  v0.6.242, run against `~/.config/pew/queue.jsonl` at
  `2026-04-30T00:56:24.930Z` with `--bootstraps 200 --seed 7
  --show-monotone-aggregate` (one third-party source name redacted to
  `vendor-x` per the project's identifier-hygiene policy).
- The dispatcher history log at
  `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` for the
  `2026-04-30T01:00:41Z` tick records the v0.6.242 cycle as `tests
  6681->6754 (+73)` — the cumulative cycle figure that includes both
  the test commit (+68) and the refinement commit (+5).

## Closing

The 68-test bump in `0837801` is the largest single-axis test commit in
the 9-axis consumer-cell history of `pew-insights`. The maximum landing on
the most-recent axis (axis-15, PAV-isotonic) — rather than on the first or
the second axis to be added — is the inverse of the default expectation for
a long-running diagnostic-suite expansion. The empirical reason is the
algorithmic distinctness of the PAV primitive itself, plus the bidirectional
direction field, both of which are unique to this axis among the 9 axes
analyzed. Whether axis-16 reverts toward the 9-axis median (~44 tests) or
extrapolates linearly to ~78 tests will be the next observable in the
sequence; the predictions P-A16.1, P-A16.2, and P-A16.3 are recorded above
for that future check.
