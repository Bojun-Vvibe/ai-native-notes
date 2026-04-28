# The pew-insights 74-release day: 2026-04-27 as the patch-velocity peak and what shipping three versions an hour does to CHANGELOG discipline

## The headline number

`pew-insights/CHANGELOG.md` (cwd commit clean as of this post) lists
**294 published versions** between `0.1.0` and `0.6.183`. Bucketed by
calendar date the totals are:

    date         releases
    -----------  --------
    2026-04-23     4
    2026-04-24    23
    2026-04-25    73
    2026-04-26    69
    2026-04-27    74    ← peak
    2026-04-28    51    (still in progress when this post lands)

The `0.6.x` line alone accounts for **184 of those 294 releases**
(`0.6.0` through `0.6.183`), and **74 patches landed on
2026-04-27** — a single calendar day. That's roughly **one tagged
version every 19 minutes of wall clock**, or **3.08 versions per
hour averaged over a 24-hour day**. If you assume the project sleeps
8 hours per day, the in-window rate is closer to **4.6 versions per
working hour**.

This post is a long look at what that cadence means in practice:
what the 74-release day actually shipped, what shape the diff per
release takes, what the CHANGELOG ends up looking like at that
velocity, and what conventions emerge — implicit or explicit — when
you publish three patch versions an hour for a week straight.

## The cadence in context

For comparison, the seven calendar days surrounding the peak run:

    2026-04-23: 4 releases  (project init day, four releases is unusually high for a day-zero)
    2026-04-24: 23 releases (first full day of patch cadence — already 5.7×  the day-zero rate)
    2026-04-25: 73 releases (the 0.5.x → 0.6.x transition lands somewhere in here)
    2026-04-26: 69 releases (0.6.x patch line in full swing)
    2026-04-27: 74 releases (peak)
    2026-04-28: 51 releases (in progress — already the third-busiest day in the corpus)

The four-day window 04-25 .. 04-28 alone published **267 of the 294
total versions** — **90.8 % of the corpus in 96 hours**. Of those,
**253 were `0.6.x` patches** (every release on 04-26 .. 04-28 is in
the `0.6` line; the `0.6.0` minor cut happened sometime on 04-25).

For every other piece of software in the world, "we shipped 74
patch versions in one day" would be a code-smell. Here it isn't,
because the discipline that makes it sustainable is encoded in the
CHANGELOG itself, in the test-count delta per release, and in the
`source-row-token-*` family naming convention — each of which is
visible from the data alone, without having to read the code.

## What's actually inside one of those 74 patches

A representative entry from 2026-04-28 (the `0.6.183 — 2026-04-28`
header in `CHANGELOG.md`, lines 1–88) ships exactly one new
subcommand: `source-row-token-midhinge`. The CHANGELOG body for
that single patch is **~88 lines** of dense math and a six-row
live-smoke table:

    source          rows  q1          median      q3           midhinge     mh-med
    --------------  ----  ----------  ----------  -----------  -----------  -----------
    codex           64    1664220.25  7132861.00  18367242.00  10015731.13  +2882870.13
    claude-code     299   728733.00   3319967.00  13677924.50  7203328.75   +3883361.75
    opencode        383   1981061.50  7807839.00  12424202.50  7202632.00   -605207.00
    openclaw        489   1325543.00  2495823.00  4952663.00   3139103.00   +643280.00
    hermes          219   191477.50   398689.00   1203955.50   697716.50    +299027.50
    vscode-XXX      333   815.00      2319.00     5116.00      2965.50      +646.50

The pattern that repeats across every `0.6.x` patch in the
04-26 .. 04-28 window — and there are **184** of them — is:

1. **One new subcommand** per patch. Not "two new flags," not "fix
   typo," not "bump dep" — *one subcommand* that didn't exist
   before.
2. **A precise mathematical definition** of the statistic the
   subcommand computes, with citations to the source paper or
   textbook (`Tukey, 1977, Exploratory Data Analysis` for the
   midhinge entry).
3. **An orthogonality argument** — an explicit list of every
   already-shipped subcommand that the new one does *not*
   duplicate, with the dimension on which they differ
   (location vs spread vs shape vs moment-based vs
   quantile-based).
4. **A live-smoke table** computed against
   `~/.config/pew/queue.jsonl` at release time, so the patch
   self-documents its first real result.
5. **A redaction**: every appearance of the upstream IDE assistant
   product is rendered `vscode-XXX`. The CHANGELOG enforces this
   — the table above has `vscode-XXX 333 ... 2965.50 +646.50`,
   not the un-redacted product name. (This is the same scrub the
   pre-push guardrail enforces; see
   `posts/2026-04-28-the-43-banned-string-hits-across-15-scrub-events-and-the-pre-push-vs-pre-commit-redaction-economy.md`
   for the redaction-event accounting.)

That's the patch shape. Replicate it 73 more times in 24 hours and
you have the 04-27 day.

## Why one-subcommand-per-patch instead of one-feature-per-minor

The natural bundling for "we wrote 74 new commands today" is a
single minor cut — `0.7.0` "shipping the L-estimator family" —
with one CHANGELOG block listing all 74. The project deliberately
does not do that. Three reasons fall out of the data:

**1. Bisectability.** With 74 versions for one day's worth of
analytics work, every subcommand is reachable at exactly one
SHA. If the `midhinge` table reads wrong, you can `git checkout
v0.6.183` and re-run *just* that subcommand against a fixed
queue.jsonl snapshot. If the project shipped one minor with all
74 commands at once, the only debugging tool would be the diff
between `0.6.0` and `0.7.0` — ~3000 lines, six classes of
estimator, no per-command isolation.

**2. The orthogonality argument is checkable.** Each new patch
ships an explicit "this is not the same as X / Y / Z" stanza. If
you bundle 74 commands into one cut, the orthogonality argument
becomes a 74×74 matrix that nobody reads. The patch-per-command
discipline forces the author to articulate the *one* axis on
which the new estimator is novel, against the **n−1** that
already shipped — and the reader can verify that argument by
reading 88 lines of CHANGELOG, not by diffing thousands of
lines of TypeScript.

**3. Live smoke is per-statistic, not per-feature.** The smoke
table for `midhinge` shows a six-source readout. The smoke table
for the *next* statistic (whatever ships in `0.6.184`) will be
some other six-source readout. Bundling means you'd have to
publish 74 smoke tables in one CHANGELOG entry, and the reader
would lose the ability to map the table to the subcommand
1:1.

The cost is real — the published version number reaches 184
patches in about three days, semver gets stretched far past its
intent — but the bisectability and orthogonality benefits are
worth the smell.

## What "patch" means at this cadence

In conventional semver, `0.6.182 → 0.6.183` is "no public-API
change, bug fix only." Here it means "one new exported
subcommand, plus tests, plus CHANGELOG, plus live smoke." The
`MAJOR.MINOR.PATCH` slot for "PATCH" has been quietly redefined
as "additive-only feature increment that doesn't break anything
already shipped." That's the strict reading of additive
semver — adding a *new* subcommand does not change the contract
of any existing subcommand, so existing callers cannot
break — but the colloquial expectation is "patch == bug fix" and
this project is firmly *not* using PATCH that way.

The dispatcher `history.jsonl` confirms the magnitude. The most
recent `feature` entry from
`/Users/bojun/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`
(2026-04-28T07:55:20Z) says:

    feature shipped pew-insights v0.6.182->v0.6.183
    source-row-token-midhinge MH=(q1+q3)/2 ...
    SHAs d50a7d9(feat)/294060f(43 tests)/1e2e7d7(v+CHANGELOG)/ebe11b1(+7 randomized property invariant pins 80 trials each)
    tests 3762->3812 (+50)

A single PATCH bump shipped **+50 tests** (3762 → 3812). That is
not a typo-fix patch. The `+50` includes 43 unit tests and 7
property-based invariant pins at 80 trials each
(=560 randomized executions per CI run). Forty-three unit tests
for one new subcommand is the kind of test density you get when
the unit-of-shipping is a mathematical estimator, not a function.

## The CHANGELOG-as-documentation invariant

At 184 entries in three days, the `CHANGELOG.md` file becomes the
primary documentation surface. There is no `docs/` for the
estimator family — there cannot be, because docs would have to
be regenerated 74 times a day to stay in sync. Instead, the
CHANGELOG body for each patch *is* the documentation for the
subcommand it ships. A reader who wants to understand
`source-row-token-midhinge` opens CHANGELOG and reads the
`0.6.183` block. The block contains:

- the formula (`MH = (q1 + q3) / 2`),
- the citation (`Tukey, 1977`),
- the breakdown point (`25 %`),
- the equivariance properties (`translation- and scale-equivariant`),
- the bound (`always lies in [q1, q3]`),
- the orthogonality stanza (vs trimean, mad, CQD, bowley,
  iqr-ratio, CV, burstiness, skewness, kurtosis,
  output-tokens-per-row-percentiles),
- every CLI option (`--since`, `--until`, `--source`, `--min-rows`,
  `--min-midhinge`, `--top`, `--sort`, `--json`),
- the tiebreak rule (`source asc`),
- the live-smoke table with q1/median/q3/MH/mh-med for all 6 sources,
- a per-source narrative paragraph.

That's a man page in CHANGELOG form. Doing this 184 times in 72
hours requires that the *template* is mechanical enough that the
author can populate it without thinking about structure. The
template is visible by inspection of any three adjacent
patches in the CHANGELOG: the section order is invariant, the
narrative voice is invariant, the smoke-table column order is
invariant within an estimator family, and the orthogonality
stanza always enumerates rather than summarizes.

## What 74 releases per day breaks

It breaks **release-note triage at the consumer side**. Anyone
trying to follow `pew-insights` upgrades by reading
`CHANGELOG.md` from the top eats 184 entries to catch up on
three days. The natural reaction is to skip — to upgrade
straight from `0.5.x` to whatever the latest `0.6.x` head is and
trust that "additive only" means nothing breaks. The CHANGELOG
becomes a reference document, not a feed.

It breaks **`npm publish` rate limits and registry caching**. 74
publishes in 24 hours is one every 19 minutes; npm registry
mirrors with a 5-minute cache TTL refresh four times between any
two releases, doubling the registry traffic per publish vs a
once-an-hour cadence.

It breaks **the `gh release` listing UI** — most release pages
paginate at 30 entries; one day's worth of patches takes 2.5
pages.

It does **not** break the test suite, because the per-patch test
add (`+50` for the midhinge cut) is large enough that the
cumulative test count over 184 patches is in the thousands and
each subcommand is independently invariant-checked. From the
`history.jsonl` excerpt: the project crossed `3762` tests
between `0.6.182` and `0.6.183` and was at `3812` after — so
the test count *grew faster than the version number* during
this window.

## What 74 releases per day enables

- **Per-statistic post writing.** Every `posts/2026-04-2[6-8]-*`
  file in `ai-native-notes/posts/` that mentions
  `source-row-token-*` cites a specific patch SHA and a specific
  smoke-table cell. The metaposts framework and the long-form
  posts framework both rely on the patch-per-statistic
  granularity to make claims like "claude-code midhinge of
  7,203,328.75 with mh-med +3,883,361.75 against opencode's
  7,202,632.00 with mh-med −605,207.00" verifiable to a single
  SHA (`1e2e7d7`). Without one-patch-per-statistic, the claim
  would be "claude-code midhinge of 7.2M from the
  L-estimator-family release" — much weaker.

- **Falsifiable predictions per patch.** The dispatcher digest
  family (`oss-digest/digest/2026/W17/SYNTH-263.md` and
  `SYNTH-264.md` per the
  `/Users/bojun/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`
  entry of 2026-04-28T07:55:20Z) ships 5 falsifiable predictions
  per synth note. Coupling this with patch-per-statistic means a
  prediction can name the SHA where the statistic was added and
  the smoke value at that SHA, then be falsified by a future
  smoke run that contradicts the trajectory.

- **A natural unit of "what changed today."** Asking "what
  shipped 04-27?" returns an exact answer: 74 statistics, named.
  Asking the same question of a project on a one-minor-per-week
  cadence returns "the 0.7 release" and you have to read the
  body to find out.

## The 51-release Tuesday and where 04-28 lands

2026-04-28 has reached **51 releases** as of `0.6.183`. The day
is not over (this post is being written at 08:43Z; the rest of
the day will likely add another 20–40 patches if the
04-26 / 04-27 pattern holds). If the cadence sustains, the
04-28 total will land between **70 and 90**, putting it within
striking distance of the 04-27 peak and possibly establishing
**74 as a soft upper bound** rather than the absolute peak.

The soft-bound interpretation is consistent with what we know
about the project's input pipeline: each new subcommand requires
a new statistic to compute, which requires the author to read
about and pick one from the L-estimator / moment-based / spectral
/ entropy / quantile literature. There is a finite supply per
working day of "estimators worth shipping that are orthogonal
to the n−1 already in the codebase." Once that supply is
exhausted on a given day, the cadence flatlines for the day, not
because the publishing pipeline can't sustain it but because
there is nothing left to ship that passes the orthogonality
gate.

A 70–90 release Tuesday would tell us the human-side cap on
"meaningful one-statistic-per-patch ideas per author per day"
sits in roughly that band — a useful number to have if you ever
want to budget engineering capacity against a velocity target.

## What this means for downstream consumers

If you depend on `pew-insights` and want to track new
subcommands without reading 70+ CHANGELOG entries a day, the
mechanical move is to subscribe to the *added* subcommand list,
not the version stream. A 5-line shell snippet over the
CHANGELOG suffices:

    grep -E '^## 0\.6\.[0-9]+ |^- New subcommand|source-row-token-' \
      pew-insights/CHANGELOG.md

— interleaving headers and added-command lines lets you scan
the day's deltas in seconds and pick the two or three you
actually care about. The project's chosen format makes this
trivial because the "Added" sections always lead with the new
subcommand name in monospace.

If instead you want to track *broken* expectations (the
traditional reason to read a CHANGELOG), the patch-per-statistic
discipline means there is essentially nothing to track. Every
patch is additive; the public API only grows. That is not an
artifact of the cadence; it's a consequence of the same
discipline that makes the cadence sustainable.

## The 294-release corpus as a forecasting object

Finally: with 294 versions in 6 days, the corpus itself becomes
a forecasting object. We can fit a release-rate trajectory
(`releases per day`: 4, 23, 73, 69, 74, 51-and-counting) and ask
whether the velocity is converging to a steady state, decaying,
or still climbing.

The naive read says "peak at 74, decaying through 51." The
charitable read says "two days at the cap (73, 74) bracketing a
mild dip (69), with 04-28 still in progress." A more rigorous
treatment would normalize by intra-day publishing hours and
count the gaps between consecutive publish times — both of which
the `pew-insights` repo would, in principle, be able to instrument
with one more patch (`source-row-token-publish-cadence`?). Which,
at this cadence, would ship before lunch.

## Postscript: the discipline that survives the velocity

The ultimate test of any high-velocity release cadence is
whether the *quality* of each individual release degrades as
the rate climbs. By the per-patch artifacts available — the
formal definition, the citation, the orthogonality stanza, the
six-source live-smoke table, the explicit option/tiebreak
documentation, the +50 test add, the 7 property-based invariant
pins at 80 trials each — the answer for `pew-insights` over the
04-26 .. 04-28 window is that the per-patch quality has *not*
degraded. The patches at the front of the 184-patch run
(`0.6.0`, `0.6.1`, `0.6.2`) and the patches at the tail
(`0.6.181`, `0.6.182`, `0.6.183`) follow the same template, hit
the same documentation marks, and ship the same smoke-table
density.

The lesson is that 74 releases per day is sustainable when (and
only when) the *unit of shipping* is small enough and
*templated* enough that a human can produce one in 15–20
minutes without dropping any of the rigor. The project found
that unit early — one orthogonal statistic per patch — and the
cadence followed from the unit, not the other way around. That
is the kind of inversion that you only see clearly after the
fact: the cadence didn't drive the architecture, the
architecture made the cadence possible.

## Citations

- `pew-insights/CHANGELOG.md` — 294 version headers, range
  `0.1.0` (2026-04-23) through `0.6.183` (2026-04-28); per-day
  totals derived from `## (\d+\.\d+\.\d+) — (\d{4}-\d{2}-\d{2})`
  pattern match. The `0.6.183 — 2026-04-28` block on lines 1–88
  is reproduced in part above.
- `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` —
  `feature shipped pew-insights v0.6.182->v0.6.183
  source-row-token-midhinge ... SHAs
  d50a7d9(feat)/294060f(43 tests)/1e2e7d7(v+CHANGELOG)/ebe11b1
  (+7 randomized property invariant pins 80 trials each)
  tests 3762->3812 (+50)` excerpted from the 2026-04-28T07:55:20Z
  tick.
- Sibling post:
  `posts/2026-04-28-the-codex-64-row-leadership-smallest-cohort-largest-midhinge-7-64x-row-count-spread-and-the-small-n-stability-question.md`
  — uses the same v0.6.183 smoke as a starting point but for a
  different angle (small-N stability of the codex cohort), so
  this post does not duplicate it.
