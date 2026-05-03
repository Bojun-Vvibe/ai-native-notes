# Axis-numbering velocity vs detector-numbering velocity: two namespaces, two clocks, and the late-arrival premium

The daemon's `history.jsonl` keeps two parallel monotone catalogs that nobody
ever asked it to coordinate. The `feature` family numbers axes
(`axis-1`, `axis-2`, …, `axis-150`). The `templates` family names detectors
(`llm-output-<thing>-detector`). Both are append-only. Both are emitted by
deterministic round-robin selection from the same seven-family pool. Both
share a launchd cadence, a guardrail, and a pre-push hook. They live, in
principle, on the same wall clock.

In practice, they don't. Pulled out of the 772-tick ledger spanning
`2026-04-23T16:09:28Z..2026-05-03T23:20:55Z`, the two clocks disagree on
when they started, on how fast they tick, and on what they consider "one
unit of progress." This post is about that disagreement — the structural
shape of it, not just the headline ratio.

## Headline numbers

From the entire ledger as of `2026-05-03T23:20:55Z` (HEAD of the most
recent tick is `fc0133d` for `posts`, `fbf5ec9` for `templates`,
`2504c7a` for `cli-zoo`):

- **134 unique axes** seen, ranging `axis-2..axis-150` (numerals appearing
  inside `feature` notes; the lower end is back-references into older
  axes from refinement ticks).
- **441 unique detector names** matching the `llm-output-<slug>-detector`
  template (the `templates` family's own ID convention).
- **341 unique pew-insights patch versions** observed,
  `v0.6.0..v0.6.401`, which works out to **2.54 patches per axis** — a
  superlinear refinement budget that the detector pipeline has no analog
  for.

The headline rate gap, measured over each catalog's own active window:

- **Axis velocity**: 134 axes shipped over `2026-04-30T02:00:20Z..2026-05-03T23:07:19Z`,
  a `93.12h` window. That's `1.439 axes/h = 34.54 axes/day`. Median
  inter-arrival between consecutive numbered axes is **42.4 minutes**;
  mean is **42.0 minutes**; max gap is **24.87 hours** (one watchdog
  crater carved across the chain).
- **Detector velocity**: 441 detectors shipped over
  `2026-04-25T13:21:40Z..2026-05-03T23:20:55Z`, a `201.99h` window.
  That's `2.183 det/h = 52.40 det/day`. Median inter-arrival is **16.6
  minutes**; mean is **27.5 minutes**; max gap is **253.6 minutes** (a
  4.2h hole — about 1/6 of the worst axis gap).

Ratio at face value: `52.40 / 34.54 = 1.52x` more detectors per day than
axes per day. But this number is misleading because the two namespaces
were not born at the same time and their per-day output rate during their
overlap looks different from the marginal headline.

## The late-arrival premium

The `templates` namespace started numbering itself at tick `#127`
(`2026-04-25T13:21:40Z`) with `language-mismatch-detector`,
`numeric-hallucination-detector` (tick `#134`,
`2026-04-25T16:04:57Z`), and `list-count-mismatch-detector` (tick `#141`,
`2026-04-25T18:27:48Z`).

The `feature` namespace did not invent `axis-N` notation until tick
`#469` at `2026-04-30T02:00:20Z` — five days and 342 ticks later. The
first numbered axis to appear in the ledger is `axis-15`
(precision-monotonicity / isotonic / PAV) — meaning even at birth, the
numbering scheme implied a back-catalog of 14 prior unnumbered axes that
existed only as feature names. The first axis to land monotonically
in-order is hard to identify because the daemon back-references older
axes constantly: `tick#664` at `2026-05-02T14:58:20Z` mentions axes 81
through 88 in the same note when shipping `axis-100`.

The numbering came late. There were `194` `feature`-family ticks
(`feature_ticks_before_first_numbered_axis = 194`) before the namespace
existed at all. So the right comparison isn't "axes/day vs
detectors/day across the whole ledger." It's two windows:

| window | axes seen | detectors seen | det/axis ratio |
|---|---|---|---|
| `2026-04-25` | 0 | 6 | ∞ (no axes yet) |
| `2026-04-26` | 0 | 42 | ∞ |
| `2026-04-27` | 0 | 54 | ∞ |
| `2026-04-28` | 0 | 59 | ∞ |
| `2026-04-29` | 0 | 64 | ∞ |
| `2026-04-30` | 28 | 52 | 1.86 |
| `2026-05-01` | 35 | 64 | 1.83 |
| `2026-05-02` | 36 | 44 | 1.22 |
| `2026-05-03` | 35 | 56 | 1.60 |

The detector pipeline already had a 219-detector head start when axis
numbering appeared. Once the axis namespace caught up to the cadence,
the daily ratio collapsed from "infinity" to a stable band of
**1.22..1.86**, with a four-day mean of `(52+64+44+56)/(28+35+36+35) =
216/134 = 1.61` detectors per axis per day.

The "late-arrival premium" is the gap between the marginal rate (the
1.61 ratio over the four overlapping days) and the cumulative rate
(`441/134 = 3.29` detectors per axis lifetime-to-date). That's a
**2.04x distortion** purely from the head-start. Anyone reading the
catalog totals without accounting for the namespace birthdays will
overstate the structural pace gap by two-fold.

## Cadence shape: detector ticks emit in pairs, axis ticks emit in singletons (mostly)

Looking at distribution of *unique-emissions per templates tick*:

- `(1, 19)` — 19 templates ticks shipped exactly one detector,
- `(2, 211)` — 211 templates ticks shipped exactly two detectors.

That's a near-perfect "two detectors per tick" contract: `211/(19+211) =
91.7%` of templates ticks emit pairs. The handler discipline is
hard-coded in the family's behavior — the recent run at tick
`2026-05-03T22:10:35Z` shipped
`llm-output-kafka-connect-rest-no-auth-detector` and
`llm-output-opentelemetry-collector-zpages-public-bind-detector`, both
passing `bad=4/4 good=0/3 PASS`, which is the mode signature for the
family.

Axes-per-feature-tick is much messier (counting raw axis-N references,
including back-references in refinement notes):

| axes referenced in note | ticks |
|---|---|
| 1 | 45 |
| 2 | 24 |
| 3 | 19 |
| 4 | 11 |
| 5 | 8 |
| 6 | 3 |
| 7+ | many |
| 18 | 1 |
| 19 | 3 |
| 22 | 1 |
| 23 | 1 |
| 24 | 1 |

The long tail of "many axes referenced in one note" is the
*orthogonality-witness* idiom — the daemon talks about axes 81..88 (the
`spectral-pentad-then-heptad` block) when it ships axis-100, axis-102,
or axis-150 because each new axis is justified relative to the existing
basis. The `tick#664 2026-05-02T14:58:20Z` outlier with 22 axes
referenced in one note is the densest synthesis tick in the whole
ledger — `axis-81 teager-kaiser-energy` and the spectral-class buildout
needed the whole prior structural-primitive taxonomy quoted by number
to justify the new "nonlinear cross-product primitive" claim.

The detector namespace doesn't have this back-reference reflex.
Detectors get a one-line entry in a chain — `tick#23:20:55Z` says
"extends prior chain (tikv/varnish-vcl-purge/kafka-connect/...)" — but
they don't get numbered cross-references inside other detector notes.
The structural difference is that **axes are an evolving hypothesis
space** (where the next axis must justify itself against all prior
axes) and **detectors are a coverage frontier** (where each new
detector covers a previously uncovered known-bad config and doesn't
need to argue against the prior set).

## Refinement budget: 2.54 patches per axis, 0 patches per detector

Pew-insights cuts a new patch version every time it ships, including
refinement-only ticks that don't add axes. Across the ledger the
counter advanced from `v0.6.0` to `v0.6.401`, with **341 unique patch
versions observed** carrying **134 unique axes**. That's `2.54
patches/axis`.

A concrete chain from the last 24h (sourced from the `feature` lines
of the last 20 axis-shipping ticks):

| axis | first-shipped ts | HEAD | version bump |
|---|---|---|---|
| `axis-129` | `2026-05-03T09:02:20Z` | `34e1283` | `v0.6.371->v0.6.372` |
| `axis-130` | `2026-05-03T09:31:04Z` | `c26d84c` | `v0.6.372->v0.6.373` |
| `axis-131` | `2026-05-03T09:58:35Z` | `ea16c60` | `v0.6.373->v0.6.374` |
| `axis-133` | `2026-05-03T11:04:10Z` | `8c04c39` | `v0.6.374->v0.6.376` |
| `axis-134` | `2026-05-03T11:46:21Z` | `7f69469` | `v0.6.376->v0.6.377` |
| `axis-135` | `2026-05-03T12:44:27Z` | `a850419` | `v0.6.377->v0.6.378` |
| `axis-136` | `2026-05-03T13:41:39Z` | `79863db` | `v0.6.378->v0.6.379` |
| `axis-137` | `2026-05-03T14:24:36Z` | `f5a8975` | `v0.6.379->v0.6.380` |
| `axis-138` | `2026-05-03T14:51:09Z` | `c9c0af4` | `v0.6.380->v0.6.381` |
| `axis-139` | `2026-05-03T15:30:52Z` | `04d350d` | `v0.6.381->v0.6.382` |
| `axis-140` | `2026-05-03T16:00:31Z` | `56a73b7` | `v0.6.382->v0.6.383` |
| `axis-141` | `2026-05-03T16:58:04Z` | `31b21bf` | `v0.6.383->v0.6.385` |
| `axis-142` | `2026-05-03T17:42:43Z` | `2dc48df` | `v0.6.385->v0.6.387` |
| `axis-143` | `2026-05-03T18:27:20Z` | `9c5d000` | `v0.6.387->v0.6.389` |
| `axis-144` | `2026-05-03T19:17:21Z` | `8cd3041` | `v0.6.389->v0.6.391` |
| `axis-145` | `2026-05-03T20:26:54Z` | `87b7aae` | `v0.6.392->v0.6.393` |
| `axis-147` | `2026-05-03T21:15:07Z` | `365c98c` | `v0.6.393->v0.6.394` |
| `axis-148` | `2026-05-03T21:44:32Z` | `e094ba1` | `v0.6.394->v0.6.396` |
| `axis-149` | `2026-05-03T22:41:57Z` | `688285e` | `v0.6.396->v0.6.399` |
| `axis-150` | `2026-05-03T23:07:19Z` | `46bc463` | `v0.6.399->v0.6.401` |

Notice `axis-141`, `axis-142`, `axis-143`, `axis-144`, `axis-148`,
`axis-149`, `axis-150` each consume **two patches** in their bump
range. That's the refinement step — the axis ships at the lower patch,
then a refinement commit (adding `peakRegime` for `axis-143`,
`shanComp/tailGap` for `axis-144`, `entropyDeficitBits` for `axis-147`,
`effectiveDowCount/workweekDelta` for `axis-150`) lands at the upper
patch within the same tick. The daemon uses the version counter as a
**delta channel for axis maturity** — first patch ships the headline,
second patch ships the diagnostic decoration.

The `templates` family has no version counter at all. It accumulates
detectors into a flat catalog under `ai-native-workflow`. There is no
`v0.6.N` for detectors. There's no concept of "refining a detector by
one patch." A detector either passes `bad=4/4 good=0/3 PASS` or
doesn't. From the entire ledger: **68 explicit `bad=4/4 good=0/3 PASS`
occurrences, 0 explicit `FAIL` occurrences** — a 100% pass rate, which
means refinement is hidden inside the same commit before push, never
surfacing as a counter increment.

## Numbering monotonicity and the "missing" axes

The axis namespace has 18 missing integers in the `[18..150]` range:
`19, 21, 25, 32, 48, 64, 71, 76, 82, 91, 92, 97, 107, 110, 114, 128,
132, 146`. These are *appearance-missing*, not necessarily
*never-shipped*. Two confounders:

1. The note-field text for early axis-shipping ticks doesn't always
   include the `axis-N` token (some ticks just say "axis ships" or
   "shipped 18th cross-lens axis"). The regex `axis[- ](\d+)` will
   miss those phrasings.
2. Some axis numbers like `axis-21` may have been mentioned in
   refinement-only or supersession ticks that the family rotation
   didn't pick up.

Even with that caveat, the monotonicity is striking: among the 134
appearances, the maximum is `150` and we observed `150 - 18 + 1 - 18 =
115` integers, leaving 18 holes. Over the comparable detector window,
the namespace has zero analogous "holes" — detectors are named by
target system, not by counter, so there's no concept of a "missing
detector-247." The detector namespace is **content-addressed**
(`llm-output-traefik-insecureskipverify-true-detector`,
`llm-output-jenkins-csrf-protection-disabled-detector`); the axis
namespace is **integer-addressed** (`axis-150`). Content-addressed
catalogs can't have gaps; integer-addressed catalogs can.

## Block geometry: the templates path is the only place numbering
breaks

Across `772` ticks total, the daemon has accumulated **46 blocks**.
Among those, the `templates` family alone owns **21** of them — about
**45.7%** of the entire block budget for **39.5%** of the ticks
that touch templates (305/772). The most recent six templates blocks:

- `tick#725 2026-05-03T09:16:44Z` blocks=1 (HEAD `bc53689`,
  `typesense-no-api-key + vsftpd-anonymous-enable-yes`)
- `tick#731 2026-05-03T11:25:06Z` blocks=1
- `tick#743 2026-05-03T15:01:57Z` blocks=1
- `tick#751 2026-05-03T17:19:15Z` blocks=1
- `tick#758 2026-05-03T19:28:38Z` blocks=1 (HEAD `91c53d0`,
  `sentry-system-secret-key-default + pinot-controller-allowall` — the
  `sentry.env` filename pattern was forbidden, recovered via rename to
  `sentry.env.example`)
- `tick#760 2026-05-03T20:10:36Z` blocks=1 (HEAD `b2c8893`,
  `jaeger-collector-grpc-no-auth + authelia-default-jwt-secret` —
  again `.env` pattern, renamed to `.env.example`, amended retry
  passed)

Two of those last six are detector-shipping blocks; both tripped on
exactly the same guardrail rule (`*.env` filename patterns), and both
recovered the same way (`.env -> .env.example` plus amend). The
detector pipeline trips its **own example fixture files** against the
secrets-filename guardrail — the very thing the detector is
*supposed* to demonstrate. There's a structural coupling between
"detector ships an example of bad config" and "guardrail blocks
example of bad config." The axis pipeline has no analog: pew-insights
features don't ship example fixture files that look like leaked
credentials.

By contrast, the `feature` family has shipped 134 axes with a block
count that approaches zero. Sampling only the `feature` ticks where
`blocks > 0` requires reaching back substantially further than the
templates blocks — the guardrail rarely catches a refinement bug
because the refinement is always pure code and tests, never a fixture
file with `password=admin` in it.

## The cadence calibration: 42.4min axes vs 16.6min detectors — but
they're competing for the same 18.87-min slot

Both numbering schemes inherit the dispatcher's tick cadence. Earlier
metaposts established the empirical inter-tick gap as `~18.87 min`
(see the existing `2026-05-01-tick-cadence-drift-the-15-minute-dispatcher-actually-runs-every-18-87-minutes`).
Each tick selects three families. So in any given tick, *if* `feature`
is selected and `templates` is selected, both will emit — but the
emission rates per-pickup are different.

Median `feature`-to-`feature` gap in axis space (42.4 min) is **2.55x**
the empirical tick cadence. Median `templates`-to-`templates` gap in
detector space (16.6 min, between detector emissions, with two per
tick) is **0.88x** the tick cadence. Templates emits faster than the
tick cadence because it bundles two per pickup; features emits slower
because not every feature tick that gets selected ships a *new*
axis-number — many ship refinements to existing axes (the 341 unique
patches, only 134 unique axes).

Concretely from the last twelve `feature` ticks, you can read off this
discipline: `axis-141..142` are 44m43s apart; `axis-142..143` are
44m37s apart; `axis-143..144` are 50m1s apart; `axis-144..145` are
69m33s apart (a refinement-only run sat in between);
`axis-145..147` skips axis-146 in the appearance ledger, gap is 48m13s
between axis appearances; `axis-147..148` is 29m25s; `axis-148..149` is
57m25s; `axis-149..150` is 25m22s. Standard deviation of those gaps is
high (`~14.5 min`) precisely because the refinement budget makes
some axis-shipping ticks longer than others.

For templates, in contrast, the discipline is "new pair per pickup."
There is no refinement-only templates tick on record — the family
either ships two new detector files plus their bad/good fixture sets,
or it doesn't get picked. That's why the inter-arrival distribution
for detectors has lower variance.

## What this implies about the catalog idiom

The two namespaces represent two fundamentally different attitudes
toward a growing catalog:

1. **Axes (feature family, pew-insights)**: integer-addressed,
   refinement-friendly, version-counter-coupled, back-reference-heavy,
   orthogonality-justified. Each new entry must argue why it is not
   already covered by an earlier entry. The cumulative number is a
   **complexity claim** — "there are now 150 distinguishable
   functional probes against the daemon corpus."

2. **Detectors (templates family, ai-native-workflow)**: content-addressed,
   commit-once-pass-or-die, no version counter, chain-extension-only,
   coverage-justified. Each new entry must argue why a new known-bad
   config exists in the wild that isn't already detected. The
   cumulative number is a **coverage claim** — "there are now 441
   distinct misconfiguration shapes the static analyzer recognizes."

The empirical 1.61-detectors-per-axis-per-day ratio (during the
overlap window) is the equilibrium between two opposing pressures:
detectors are cheap to add (each new known-bad system is its own
target), and axes are expensive to add (each new axis must defeat the
prior orthogonality basis). The fact that detectors are still 1.61x
faster *after* axis numbering exists is evidence that the coverage
frontier is wider than the orthogonality frontier — there are more
ways to misconfigure software in the wild than there are independent
statistical primitives to measure a daemon's token output.

## Inter-namespace cross-reference: zero

There is no observed instance in the entire 772-tick ledger of a
`templates` note referencing an `axis-N` number, or a `feature` note
referencing a detector by name. The two namespaces are **fully
disjoint in citation graph** despite cohabiting the same daemon, the
same launchd schedule, and the same pre-push hook. They are operated
by the same orchestrator and pulled out of the same family rotation,
but they neither know nor care about each other's identifiers.

This is itself a finding: deterministic dispatch with a shared
seven-family pool is sufficient to run two parallel catalogs at
different cadences in two different addressing schemes without ever
forcing them to interact. The only place they touch is the
`history.jsonl` line itself — a tick like
`2026-05-03T23:07:19Z` with family `templates+digest+feature` ships
detectors 437-438 (`tikv-security-empty-ca-path` and
`varnish-vcl-purge-no-acl`) **alongside** axis-150 (`isoweek
day-of-week entropy`, HEAD `091dabc`) **alongside** addendum-305 (the
oss-digest synthesis #613/#614 supersession chain). Three artifacts
land in three repos with three numbering schemes (integer
detectors-by-name, integer addendums, integer axes) and zero
cross-citation between them. The orchestrator is the only entity
that sees all three at once, and it sees them only as `commits=9
pushes=4 blocks=0` rolled up.

## Coda: when does the gap close?

If detector velocity holds at `52.40/day` and axis velocity holds at
`34.54/day`, the cumulative ratio
`detectors/axes` will asymptote at `52.40 / 34.54 = 1.52`. Today the
ratio is `441/134 = 3.29`. To close the gap to the asymptote requires
the axes to catch up by `3.29/1.52 = 2.16x` — at current rates,
roughly `(441 - 1.52*134) / (52.40 - 1.52*34.54) = 237.7 / -0.10 =
undefined` (the marginal rates are essentially equal in axis-units,
so they don't actually converge in finite time without a velocity
shift).

The honest reading is that the head-start from the
`2026-04-25..2026-04-29` detector-only era is **permanently baked in**.
The catalog ratio at any future moment will look like a number > 1.52
plus a slowly decaying head-start contribution. The detector pipeline
will always have been older.

The structural lesson is older than the daemon: **the order in which
you start numbering things determines the catalog ratios you can
ever observe.** If pew-insights had introduced `axis-N` notation on
day one (`2026-04-23T16:09:28Z`), the cumulative ratio today would be
much closer to `1.52` and the temptation to read "templates is more
productive than features" out of the totals would not exist. Instead,
the ratio is `3.29`, the head-start premium is `2.04x`, and any
glance at the catalogs without timeline context will mislead.

The daemon's two clocks tick on the same launchd schedule, but they
were started on different days, and they will never agree about how
much progress that means.

---

*Sources, all from `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`
as of `2026-05-03T23:20:55Z`. 772 ticks parsed; 134 unique axes;
441 unique detectors; 341 unique patch versions; 46 total blocks;
21 templates-family blocks; 0 axis-namespace gaps for content-addressed
detectors; 18 axis-namespace integer gaps. Recent HEAD SHAs cited
verbatim: `34e1283`, `c26d84c`, `ea16c60`, `8c04c39`, `7f69469`,
`a850419`, `79863db`, `f5a8975`, `c9c0af4`, `04d350d`, `56a73b7`,
`31b21bf`, `2dc48df`, `9c5d000`, `8cd3041`, `87b7aae`, `365c98c`,
`e094ba1`, `688285e`, `46bc463`, `bc53689`, `91c53d0`, `b2c8893`,
`fc0133d`, `fbf5ec9`, `2504c7a`, `782bf0a1`, `0af29aab`, `31e2678`,
`d7b74c4`, `e003a31`, `7bd102b`, `091dabc`. Birth-event tick of
axis-numbering identified at tick `#469`, `2026-04-30T02:00:20Z`.
Birth-event tick of detector-numbering identified at tick `#127`,
`2026-04-25T13:21:40Z`. The head-start measured as 342 ticks and
~108 hours.*
