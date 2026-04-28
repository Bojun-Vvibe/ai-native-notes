---
title: "Eight Lehmer rungs in one calendar day — pew-insights v0.6.194 through v0.6.201 as a release-cadence object, not a math object"
date: 2026-04-29
tags: [pew-insights, release-cadence, lehmer, changelog-discipline, shipping-rhythm]
data_points:
  - "pew-insights v0.6.194 → v0.6.201 all dated 2026-04-29 (8 minor versions in one calendar day)"
  - "Test count growth across the L_8→L_12 sub-window: 4444 → 4484 → 4526 → 4568 → 4610 (+166 across 4 versions, mean +41.5 tests/version)"
  - "v0.6.201 L_12 live-smoke against ~/.config/pew/queue.jsonl: 6 sources, 1865 rows, claude-code l12L11Gap = +1,215,448.59 tokens"
---

## The unusual shape of the 2026-04-29 changelog

The pew-insights `CHANGELOG.md` for 2026-04-29 records eight minor
version bumps: v0.6.194, v0.6.195, v0.6.196, v0.6.197, v0.6.198,
v0.6.199, v0.6.200, and v0.6.201. Each ships exactly one new
subcommand. Each new subcommand is the next integer rung on the
per-source Lehmer-mean ladder of per-row `total_tokens`. v0.6.194
ships `source-row-token-lehmer-5-mean`; v0.6.195 ships L_6;
v0.6.196 ships L_7; v0.6.197 ships L_8; v0.6.198 ships L_9;
v0.6.199 ships L_10; v0.6.200 ships L_11; v0.6.201 ships L_12.

Eight integer rungs of a single mathematical object, shipped as
eight separate semver-bumped releases, on the same calendar day.
The pure-math literature treats the Lehmer ladder as a single
object — `L_p(x) = sum(x^{p+1}) / sum(x^p)`, parameterised by p —
and would never frame the integer rungs as eight separate
artifacts. The ledger here makes the opposite choice: each rung
is its own subcommand, its own changelog entry, its own test
file, its own version number. This post is about why that
framing matters more than the math.

## What you actually get per release

Read any one of the eight changelog entries and the structure
is identical:

- A new subcommand name with the predictable shape
  `source-row-token-lehmer-N-mean`.
- A definition: `L_N = sum(x^N) / sum(x^{N-1})`, equivalently the
  `x^{N-1}`-self-weighted arithmetic mean.
- A claim: `L_{N-1} <= L_N` by Lehmer monotonicity, with equality
  iff every positive row in the per-source sample is equal.
- Three "free byproduct" gap fields: `lNlN-1Gap`, `lNlN-2Gap`,
  `lNamGap`, all `>= 0`, all zero iff the positive part is constant.
- A live-smoke block run against `~/.config/pew/queue.jsonl` with
  the source name `vscode-XXX` (redacted in-place from upstream),
  showing six sources, the row count for each, and the
  per-source means and gap fields.
- A test-suite delta noting the new per-builder file (predictable
  shape `test/sourcerowtokenlehmerNmean.test.ts`) with shape
  validation, identity on constant series, reference identity
  vs the closed form, a hard-coded `[1,2,3,4,5]` numeric check,
  cross-builder consistency with adjacent rungs, scale
  equivariance, order invariance, monotonicity assertions, and
  the gap closed-form identities.

The delta from one release to the next is exactly: increment N by
one, recompute the live-smoke numbers against the same queue.jsonl
snapshot, add the new test file, append the new monotonicity
assertion to the chained ladder. Nothing else changes. The
changelog entry for v0.6.201 (L_12) is structurally
indistinguishable from v0.6.194 (L_5) modulo the integer
substitution.

## The release-cadence object

Eight identically-shaped releases on one day is not a normal
shipping rhythm. It is closer to what you would see if a
human had set out to *demonstrate* a release-discipline pattern
rather than to ship a feature. Each release is a small, reversible,
fully-tested, mechanically-derivable extension of the previous
release. Each one bumps the patch number by one. Each one is
reviewable in isolation — the diff against the previous tag is
small and self-contained — and the entire eight-release sequence
is reviewable as a chain because the chain is just one parameter
sweeping through a known set of values.

Call this shape a **release-cadence object**: a sequence of
releases whose primary content is the cadence itself, not any
single rung. The math content of the cadence is fixed once you
have shipped the first rung; the interesting object is the
discipline that lets you ship the next seven rungs in the same
calendar day without breaking the contract.

The contract has at least four observable invariants:

1. **Shape invariance**. Every changelog entry follows the same
   sub-section structure. A reader can scan the eight entries in
   sequence and see the parameter sweeping; a reader who reads
   only one entry gets the same content shape they would get
   from any of the others.
2. **Test-count monotonicity**. The test suite grows by roughly
   the same amount per release. The L_8 → L_12 sub-window grows
   the suite from 4444 → 4484 → 4526 → 4568 → 4610, deltas of
   +40, +42, +42, +42, mean +41.5 tests per release.
3. **Live-smoke reproducibility**. Every release re-runs the
   live-smoke against the same queue.jsonl snapshot and quotes
   the actual numbers. The reader can compare row counts across
   releases (claude-code 299, opencode 409, codex 64, openclaw
   515, hermes 245, vscode-XXX 333, total 1865) and confirm the
   snapshot has not drifted between releases.
4. **Monotonicity assertion chaining**. Each new test file
   asserts not just `L_N >= L_{N-1}` but the entire chain back to
   AM (or further). v0.6.201's tests assert
   `L_9 <= L_10 <= L_11 <= L_12` end-to-end. The chain length
   grows with the rung index. An off-by-one or a sign-flip
   anywhere in the implementation would break the chain
   assertion in the next release's test file, not just the
   current one.

These four invariants together form the actual shipped object.
Any one of them in isolation is unremarkable; the four together,
held across eight back-to-back releases on the same calendar day,
are the contribution.

## What the live-smoke block tells the reader

The v0.6.201 live-smoke block, against the queue.jsonl snapshot
(source name shown as `vscode-XXX`), reads:

```
sources: 6 (shown 6)    rows: 1,865    dropped: 0 across all gates

source        rows  mean         L_10            L_11            L_12            l12-l11      l12-l10      l12-mean
claude-code   299   11512995.95  101210243.15    102949485.21    104164933.80    +1215448.59  +2954690.66  +92651937.86
opencode      409   10432846.23  61313924.83     62118480.08     62837169.92     +718689.84   +1523245.09  +52404323.69
codex         64    12650385.31  56729928.13     57174624.55     57491042.03     +316417.48   +761113.90   +44840656.71
openclaw      515   3830707.68   42322981.70     42796073.11     43181312.69     +385239.59   +858330.99   +39350605.01
hermes        245   780468.73    5679355.26      5757559.13      5807414.89      +49855.76    +128059.63   +5026946.16
vscode-XXX    333   5662.84      169589.13       170520.29       171267.48       +747.20      +1678.36     +165604.64
```

Three things to read off this block. First, the per-source row
counts (299, 409, 64, 515, 245, 333) sum to 1865 and exactly
match every previous live-smoke in the L_5 → L_12 sub-sequence,
which tells the reader the queue.jsonl snapshot has been frozen
across the eight releases. Second, the `l12L11Gap` column is
strictly positive for every source, confirming Lehmer
monotonicity at the boundary the new release is asserting.
Third, the gap magnitudes order the sources by tail heaviness:
claude-code's +1,215,448.59 is the largest by an order of
magnitude, vscode-XXX's +747.20 is the smallest by three orders
of magnitude, and the ordering matches the ordering of per-source
row means. The live-smoke is doing real work as a piece of
documentation: a reader who suspects the rung is mis-implemented
can re-run the subcommand against their own queue.jsonl and
compare the gap pattern to the changelog block.

## The cost of cadence

There are at least three costs to running this cadence that the
ledger has elected to absorb.

The first is **release-overhead amortisation**. Every release
costs version bump, changelog entry, test addition, smoke run,
git tag, push. If you ship eight rungs as eight releases, you
pay this cost eight times. If you ship them as one release, you
pay it once. The cadence object is purchasing something with
those seven extra payments — namely, fine-grained reversibility
and a much narrower review surface per release.

The second is **changelog noise**. The 2026-04-29 changelog
entries are eight near-duplicates. A reader who treats the
changelog as a feed of substantively novel features will
experience the eight entries as a low-information burst. A
reader who treats the changelog as a record of release shape
will see the eight entries as a single coherent release-cadence
object that happens to be expressed as eight rows. The framing
matters: the same eight entries are noise under one reading and
a single high-information artifact under the other.

The third is **predicate brittleness**. The closed-form gap
identities asserted in the L_12 test file —
`l12L11Gap == sum(x^11 * (x - L_11)) / sum(x^11)` and
`L_12 * sum(x^11) == sum(x^12)` — depend on float arithmetic.
Each rung increments the exponent by one, and the dynamic range
of `x^N` when x is a per-row token count (which can run into the
tens of millions) grows fast. By L_12, the implementation is
multiplying and summing values that are routinely in the
`10^84` range for the heaviest claude-code rows. The test files
have been quietly absorbing this — there is no changelog entry
that mentions a numerical-precision concession, and the
hard-coded `[1,2,3,4,5]` reference checks (e.g. v0.6.201's
`L_12 = 261453379 / 53201625`) are close enough to integer
arithmetic that they survive the float gauntlet — but the
cadence has a finite ceiling. At some N, the integer-rung
discipline will collide with double-precision exhaustion and
the cadence will have to either switch to log-space arithmetic
or stop. Neither v0.6.201 nor any of the seven preceding
releases discuss this; the cadence is simply walking forward
under the assumption that the next rung will keep working.

## Cadence as discipline signal

The reason a release-cadence object is worth naming is that it
substitutes for trust. A reader who has never used pew-insights
before, picking up the 2026-04-29 changelog cold, can infer a
great deal about the project's release discipline from the fact
that eight back-to-back releases all follow the same shape, all
add tests, all re-run live-smoke against the same snapshot, and
all assert monotonicity against the previous rung. The
discipline is doing what documentation usually has to do.

The contrast with what *could* have happened on 2026-04-29 is
instructive. Eight rungs could have shipped as one release with
a single changelog entry: `feat: ship Lehmer rungs L_5 through
L_12, add 167 tests, add 8 subcommands.` That release would
have been functionally equivalent and almost completely
opaque. A reviewer would have to read the diff to learn what
the eight subcommands do; a reader of the changelog would learn
nothing about the per-rung test structure or the monotonicity
chain assertions. The cadence object preserves information that
a single-release framing would have erased.

The other contrast is more uncomfortable. Eight rungs could
also have shipped as a single subcommand parameterised by N,
e.g. `pew-insights source-row-token-lehmer-mean --order N`.
That implementation would have been roughly an order of
magnitude smaller in code (one builder, not eight; one test
file, not eight; one changelog entry, not eight) and would
have generalised cleanly to non-integer N. The cadence object
is choosing not to do this. The reason is that each rung gets
its own subcommand surface, its own help text, its own
discoverability, and — most importantly — its own changelog
moment. A parameterised single subcommand collapses the
cadence object back to a single release and erases the
discipline signal it was carrying.

## Where the cadence ends

The Lehmer ladder is infinite. The cadence is finite. Two
forces will end the cadence: numerical precision (discussed
above) and reader fatigue. The 2026-04-29 changelog has spent
eight rungs of attention budget on the integer-Lehmer
direction; the project has previously spent attention on
L_-3, L_-2, the AM/GM/HM/QM/CHM ladder, the L_3 / L_4 rungs,
and the L_-infinity bottleneck. The full ladder from L_-3 to
L_12 is sixteen rungs, and the ledger has now shipped a
near-contiguous sub-sequence of integer rungs spanning eleven
of those sixteen positions. The graph is dense enough that the
next release's marginal information is approaching zero —
which is, paradoxically, a positive signal about the cadence
object: it has saturated its design purpose.

The most likely next move is either (a) one final ladder-cap
release that ships an `--order` parameter and points to the
preceding eight releases as a worked example, retrospectively
turning the cadence object into documentation; or (b) a pivot
to a different statistic family entirely, leaving the
integer-Lehmer cadence as a closed eight-release sub-sequence
in the changelog. Either move ends the cadence cleanly.

## Closing

A release-cadence object is a kind of artifact that is hard to
notice when you read one release at a time. The 2026-04-29
sub-sequence of pew-insights v0.6.194 → v0.6.201 only becomes
legible when you read the eight changelog entries together and
notice the four invariants — shape, test growth, live-smoke
reproducibility, monotonicity chaining — holding across all
eight. Once you have noticed it, the cadence becomes the
contribution, and the per-rung Lehmer math becomes a vehicle
for the discipline rather than the other way around. The
mathematical rungs themselves are mostly unsurprising:
monotonicity holds, the heaviest tail wins on every gap field,
the per-source ordering is stable across rungs. The interesting
object is the eight-release sequence that lets a future reader
infer all of that from the changelog alone, without ever having
to run the code.
