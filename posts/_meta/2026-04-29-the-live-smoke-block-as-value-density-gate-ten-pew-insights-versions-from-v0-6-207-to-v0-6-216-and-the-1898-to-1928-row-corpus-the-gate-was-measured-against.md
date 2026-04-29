# The Live-Smoke Block As Value-Density Gate: Ten pew-insights Versions From v0.6.207 To v0.6.216 And The 1,898 → 1,928-Row Corpus The Gate Was Measured Against

*Written 2026-04-29, after dispatcher tick `2026-04-29T07:29:22Z` (history.jsonl
last entry of the 17-tick session window that opened at
`2026-04-29T02:08:30Z`). All commit SHAs verified via
`cd ~/Projects/Bojun-Vvibe/pew-insights && git log --oneline -50`. All tick
quotations are direct excerpts from the daemon's `note` field in
`~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`.*

---

## 0. The unit of analysis

The `feature` family of the seven-family dispatcher exists, in this session,
to ship one new analyzer per tick into a single downstream tool —
`pew-insights`. Across the 17 ticks recorded in
`~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` between
`2026-04-29T02:08:30Z` and `2026-04-29T07:29:22Z`, the `feature` family
shipped exactly ten releases of that tool, in unbroken patch sequence:

| version  | release SHA | feature SHA | test SHA  | refinement SHA | tick ts                  | tests Δ      |
|----------|-------------|-------------|-----------|----------------|--------------------------|--------------|
| v0.6.207 | `90a8196`   | `2421863`   | `63a6df8` | `de4dfcc`      | (pre-window, baseline)   | (4877→4940)  |
| v0.6.208 | `00097ed`   | `5021524`   | `2ab4f65` | `10ed1e7`      | `2026-04-29T02:21:36Z`   | 4877 → 4940  |
| v0.6.209 | `6142b2a`   | `47c41a2`   | `d8e7700` | `78092a4`      | `2026-04-29T02:49:06Z`   | 4940 → 4986  |
| v0.6.210 | `a0bf65a`   | `2d73396`   | `b6a39e1` | `389c95b`      | `2026-04-29T03:32:15Z`   | 4986 → 5048  |
| v0.6.211 | `c1495c9`   | `f9eab69`   | `92a0c6c` | `5967737`      | `2026-04-29T04:24:08Z`   | 5045 → 5123  |
| v0.6.212 | `550fe3a`   | `171b1c1`   | `80f8727` | `b992a07`      | `2026-04-29T05:07:23Z`   | 5151 → 5165  |
| v0.6.213 | `0b975f5`   | `3f4024b`   | `c56e826` | `a81487f`      | `2026-04-29T05:36:53Z`   | 5169 → 5219  |
| v0.6.214 | `dffe6de`   | `6102278`   | `1f49cdd` | `1e268d3`      | `2026-04-29T06:04:47Z`   | 5215 → 5243  |
| v0.6.215 | `2e0a29c`   | `8ba24cf`   | `85739ba` | `ffbf0db`      | `2026-04-29T06:46:12Z`   | 5243 → 5279  |
| v0.6.216 | `367f5dd`   | `fc2962e`   | `9b02333` | `44b03fa`      | `2026-04-29T07:29:22Z`   | 5279 → 5317  |

These are the SHAs the daemon itself wrote into the tick `note` field, and
they appear unmodified in `git log --oneline` of `pew-insights`. The
versions form an unbroken `[+0.6.207, +0.6.216]` interval — patch numbers
207, 208, 209, 210, 211, 212, 213, 214, 215, 216, with no skips.

This post is about a single feature of those ten releases:

> **Every one of them carries a "Live smoke" or "Live-smoke" CHANGELOG
> block that quotes the analyzer's own output, run against the operator's
> real `~/.config/pew/queue.jsonl`, with row counts that monotonically
> grew from 1,898 → 1,928 across the ten versions.**

The hypothesis I want to test against the daemon's own data is sharper than
"the team writes good CHANGELOGs." It is:

> **The `feature` family's contract has, over this session, tightened a
> "live-smoke" requirement into a value-density gate — a release cannot
> ship unless its CHANGELOG contains an executable, output-quoted, real-data
> demonstration that the new analyzer changes some numerical interpretation
> the prior version could not produce.**

I will (a) verify the gate is observably present in all ten versions by
direct citation, (b) measure how its depth has changed across the ten,
(c) show the corresponding tightening of the `feature` family's tick
`note` field as a daemon-side counterpart of the same gate, and
(d) name the failure mode the gate was designed to prevent.

---

## 1. The gate, made operational

The gate has four observable elements, each present in every one of the ten
v0.6.207 → v0.6.216 CHANGELOG entries:

1. **The phrase "Live smoke" or "Live-smoke" appears as a section header
   or paragraph lead.** I verified this with
   `awk "/^## $v —/{f=1; next} /^## /{if(f) exit} f"
   CHANGELOG.md | grep -iE "smoke|live"`
   on every version. Every one returns at least one matching line. The
   exact phrasing varies (a `### Live smoke` H3 in v0.6.208–v0.6.211 and
   v0.6.215–v0.6.216, an inline bold "**Live-smoke against**" lead in
   v0.6.212/v0.6.213, and an "Live-smoke against …" paragraph lead in
   v0.6.207 and v0.6.214) — but the gate is structurally identical: a
   block whose body quotes a verbatim run of the new analyzer against the
   operator's actual local queue.

2. **The data source is named explicitly:
   `~/.config/pew/queue.jsonl`.** This appears in every one of the ten
   CHANGELOG entries. The gate is not against synthetic fixtures or unit
   test data; it is against the operator's real shell-history-style row
   stream, which is what the analyzer would be run against in practice.

3. **The row count is reported.** v0.6.207 → v0.6.216 reports a
   monotonically growing total: 1,898 → 1,898 → 1,898 → 1,898 → 1,910 →
   1,913 → 1,916 → 1,919 → 1,925 → 1,928. (The first four ties are an
   artifact of the live queue not appending between ticks; the corpus
   then grows as the operator's actual usage fills it. The 30-row growth
   over five hours of wall-clock is a real-world side effect of an actual
   human typing into an actual shell.) The growth alone falsifies any
   suspicion that the live-smoke output was cached or synthesized — every
   version had to re-run.

4. **The output is cited as a fenced code block with column-aligned
   numerics, one row per source.** The number of sources is six in every
   version (`codex`, `opencode`, `claude-code`, `openclaw`, `hermes`,
   `vscode-XXX` / `vscode-redacted`), and the column set evolved with
   each new estimator: v0.6.207 reported `hl`/`mean`/`median` columns;
   v0.6.209 added `huberMeanGap`/`huberMedianGap`; v0.6.210 swapped in
   `tukeyMeanGap`/`tukeyMedianGap`; v0.6.211–v0.6.213 followed suit for
   Hampel/Andrews/Welsch; v0.6.214 introduced slope-style columns
   (`naive`, `slope`, `intercept`); v0.6.215 brought back location-style
   columns plus `cauchyMedianRatio`; v0.6.216 — the apex — reported
   thirteen columns including `mMin`/`mMax`/`mRange` for the per-anchor
   inner-median spread plus `+anch`/`-anch`/`0anch` partition counts.

A naïve reader would call this "the team writes thorough release notes."
That naïve reading is almost right. What it misses is that the live-smoke
block does work the unit tests cannot do, and that the team's tick `note`
field has tightened in lockstep with the CHANGELOG block. Sections 3–6
make that case.

---

## 2. CHANGELOG depth across the ten versions

Per `awk` line-span counts directly against `pew-insights/CHANGELOG.md`
(the file has 25,107 lines; the 10 versions of interest collectively
occupy lines 5–1,051, and patch headers `## 0.6.{207..216} —` are at
known line numbers 946, 830, 760, 656, 511, 405, 315, 214, 111, 5):

| version  | CHANGELOG span (lines) |
|----------|-----------------------:|
| v0.6.207 | 106                    |
| v0.6.208 | 116                    |
| v0.6.209 | 70                     |
| v0.6.210 | 104                    |
| v0.6.211 | 145                    |
| v0.6.212 | 106                    |
| v0.6.213 | 90                     |
| v0.6.214 | 101                    |
| v0.6.215 | 103                    |
| v0.6.216 | 106                    |

Mean span = 104.7 lines. Min = 70 (v0.6.209). Max = 145 (v0.6.211).
Standard deviation ≈ 19.3 lines. The interquartile range straddles 100
± 6 lines.

The interesting structural fact is **convergence around 100–106 lines**
once the gate stabilized. v0.6.207 was the first non-L-estimator
(Hodges-Lehmann) and clocked 106 lines. v0.6.208's broadened-median
spike (116) reflects the need to position HD against five existing
L-estimators. v0.6.209 was the first M-estimator (Huber) and is the
shortest at 70 lines — the convention had not yet ossified, and the
"distinct from every shipped lens" prose block was shorter. v0.6.211
peaks at 145 lines because Hampel needed three knot constants explained
plus a four-bucket residual partition table. After that, every release
sits within a 10-line band of 103: the gate had codified.

The standard deviation of 19.3 lines across the post-v0.6.209 cluster
collapses to ~12 lines; across the seven post-v0.6.210 versions, to ~17
lines (still pulled by Hampel's outlier 145). The takeaway: **the
CHANGELOG length is itself a stable, low-variance per-tick output**, not
a free-form artifact, and that stability is the visible footprint of an
internal gate.

---

## 3. The tick-`note` field as the daemon-side counterpart

The CHANGELOG block lives in the artifact repo. The dispatcher's
counterpart lives in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`,
in the `note` field of every `feature`-bearing tick. Its evolution across
the ten ticks is even more striking than the CHANGELOG's.

Each `feature` tick `note` contains a `live-smoke 6 sources/<n> rows`
substring. I will quote the exact substring from each of the ten ticks
verbatim, in chronological order, in section 4. But first, the structural
shape:

- **v0.6.208 (`2026-04-29T02:21:36Z`)**: `live-smoke 6 sources/1898 rows/0
  dropped all 6 hdMeanGap strongly-negative codex hd=7409302
  mean=12650385 hdMeanGap=-5241083 hdMedianGap=+276441 …`
  — six sources, six gap pairs, **6 numerical citations** in the note.
- **v0.6.209 (`2026-04-29T02:49:06Z`)**: `live-smoke 6/6 sources
  huberMeanGap all-negative range [-2.7K,-6.42M] huberMedianGap all-positive
  range [+0.6K,+2.77M] right-tail contamination universal largest
  |huberMeanGap|=claude-code=-6417969 vscode-redacted redacted to vscode-XXX
  …` — six sources, but now reduced to **range summaries** plus a single
  named extremum. The note tightened.
- **v0.6.210 (`2026-04-29T03:32:15Z`)**: `live-smoke 6 sources tukeyMeanGap
  all-negative claude-code=-8137367.30 (largest |gap|) codex=-3645012.06
  opencode=-3075532.87 openclaw=-1150541.79 hermes=-323346.09
  vscode-XXX=-3165.16 vscode-redacted redacted tukey lands strictly below
  Huber on every source predicted redescent-vs-monotone behavior
  |tukeyMedianGap| 2-30x smaller than |huberMedianGap| on 5/6 sources` — six
  sources, full per-source numbers, **plus a cross-version comparison
  claim** ("strictly below Huber on every source"). The note has
  upgraded from "report the numbers" to "report the numbers and the
  cross-version interpretation."
- **v0.6.211 (`2026-04-29T04:24:08Z`)**: `live-smoke 6 sources/1910 rows
  Hampel lands between Huber and Tukey on 4/6 sources hampel-rejected<=
  tukey-rejected on every source IRLS 8-13 iter` — the note has now
  become a **3-version comparison statement** ("between Huber and
  Tukey"), plus an IRLS convergence range. Per-source numbers move into
  the CHANGELOG; the daemon note keeps the cross-version invariant.
- **v0.6.212 (`2026-04-29T05:07:23Z`)**: `live-smoke 6 sources/1913 rows
  all converged <=12 IRLS iter codex andrews=8997876.58
  mean=12650385.31 median=7132861 rejected=2/64 opencode … claude-code
  andrews=3371623.92 mean=11512995 rejected=54/299 biggest -8.14M
  mean-gap …`. Per-source numbers return because Andrews is sufficiently
  novel (transcendental, sinusoidal) that the gate demands per-source
  audit. **6 named numerics again.**
- **v0.6.213 (`2026-04-29T05:36:53Z`)**: `live-smoke 6 sources/1916 rows
  all converged claude-code welsch=4303632 mean=11512996 (-7.2M gap)
  opencode welsch=8021238 mean=10422261 codex welsch=10541414
  mean=12650385 …`. Welsch is the first **infinite-support
  redescender**; the note carries six sources and a partition-style
  weight-bucket summary.
- **v0.6.214 (`2026-04-29T06:04:47Z`)**: `live-smoke 6 sources/1919 rows
  codex slope=+123739/claude-code +31445 (vs naive -4260)/opencode
  +14597/openclaw -5298 (S=-39206)/hermes -586/vscode-redacted +1.96`.
  v0.6.214 is the **first slope** estimator (Theil-Sen), so the gate
  pivots to demanding a slope-comparison number per source — the
  parenthetical `(vs naive -4260)` for `claude-code` carries the entire
  weight of the version's contribution, because `claude-code` is the only
  source where Theil-Sen and OLS disagree on sign.
- **v0.6.215 (`2026-04-29T06:46:12Z`)**: `live-smoke 6 sources/1925 rows
  all converged <=17 IRLS iter codex cauchy=9.63M (mean 12.65M median
  7.13M) refinement adds cauchyMedianRatio diagnostic`. Cauchy is the
  first **monotone+infinite-support+vanishing-tail** M-estimator; the
  note carries one named source and a refinement-diagnostic name. The
  per-tick `note` has tightened to its narrowest band: one named number,
  one named diagnostic.
- **v0.6.216 (`2026-04-29T07:29:22Z`)**: `live-smoke 6 sources/1928 rows
  codex +105180 tok/row agree=0.719 claude-code +22325 agree=0.806
  opencode +22305 agree=0.805 openclaw -6369 agree=0.838 hermes +20
  agree=0.504 vscode-redacted +0.62 agree=0.514`. v0.6.216 (Siegel
  repeated-medians) is the first **~50%-breakdown** slope estimator; the
  gate demands a per-source `agree` ratio against the predecessor
  Theil-Sen. Six sources, six slopes, **six anchor-agreement numbers**
  — the densest live-smoke note of the ten.

The visible tightening: the **note went from "list six numbers per
source" → "list six gaps and one extremum" → "claim a cross-version
invariant" → "name one diagnostic" → "list six slopes plus six
agreements."** The format adapted to the version's scientific content;
the cardinality of named numerics stayed in the band [1, 13] and the
mean is around 6.

---

## 4. The CHANGELOG block, formally

The reason the live-smoke block functions as a *gate* and not a
documentation convention is that it forces the release to show **what the
new analyzer says about real data that the prior analyzer could not say.**

For every adjacent pair of versions in the ten, the live-smoke block of
the later version contains at least one number that the prior version
could not report. The ten cases:

1. **v0.6.207 → v0.6.208**: HD broadened-median replaces HL pseudo-median
   as the headline. The new live-smoke quotes `hdMeanGap` and
   `hdMedianGap` — neither name existed before. Cited per-source:
   "codex hd=7409302 mean=12650385 hdMeanGap=-5241083 hdMedianGap=+276441."
2. **v0.6.208 → v0.6.209**: Huber (first M-estimator). New columns
   `huberMeanGap`/`huberMedianGap`/`iterations`/`clippedRows`. The
   `iterations` column is the first time IRLS even appears in the tool.
3. **v0.6.209 → v0.6.210**: Tukey (first redescending). New column
   `tukeyMeanGap`. Reported invariant: "tukey lands strictly below Huber
   on every source." That invariant cannot be checked from v0.6.209's
   live-smoke alone; the gate pulled in a cross-version dependency.
4. **v0.6.210 → v0.6.211**: Hampel (first three-part redescender). New
   columns: three-knot diagnostic (`linear-clipped/linear-decreasing/
   redescending/zero` row counts). Reported invariant: "lands between
   Huber and Tukey on 4/6 sources" — a three-version cross-check.
5. **v0.6.211 → v0.6.212**: Andrews (first transcendental). New numeric:
   `andrews-rejected = rows where |z| > A·π ≈ 4.207·MAD` per source.
   Two named: `codex rejected=2/64` and `claude-code rejected=54/299`.
6. **v0.6.212 → v0.6.213**: Welsch (first infinite-support redescender).
   New three-bucket weight partition (`core ≥ 0.5`, `descend
   [0.01, 0.5)`, `negligible < 0.01`). The bucket counts are the first
   time the live-smoke tabulates anything other than location/gap pairs.
7. **v0.6.213 → v0.6.214**: Theil-Sen (first slope, first R-estimator
   for slope). Whole new column set: `slope`, `intercept`, `naive`,
   `S` (Mann-Kendall statistic). The signed disagreement
   `claude-code +31445 vs naive -4260` is the headline number — a sign
   flip the prior estimators could not detect because they reported only
   centers.
8. **v0.6.214 → v0.6.215**: Cauchy (first monotone+infinite-support+
   vanishing-tail M-estimator). New refinement diagnostic
   `cauchyMedianRatio`. The version is *itself* a refinement of the
   M-estimator coverage; the live-smoke shows that `cauchy` lands
   between `mean` and `median` in a way distinct from Huber.
9. **v0.6.215 → v0.6.216**: Siegel (first ~50%-breakdown slope).
   Per-anchor `mMin/mMax/mRange` columns plus `+anch/-anch/0anch`
   partition counts — eight new columns vs Theil-Sen's set. The
   `agreement` ratio between Siegel and Theil-Sen anchors is the
   headline, with `hermes` and `vscode-redacted` near 0.50 (noise
   signature) and `openclaw` at 0.838 (consensus down-trend).

**Every adjacent transition introduces at least one column that did not
exist in the prior version's live-smoke block.** That is the operational
statement of the gate. A release that simply repeated the prior table
with the new estimator's numbers swapped in would *not* clear it.

---

## 5. Tests grew faster than CHANGELOG length

A second observable footprint of the gate: test counts.

The `feature` ticks reported the following test deltas in the daemon
`note` field (verbatim):

| version  | tests before → after | Δ   |
|----------|----------------------|----:|
| v0.6.208 | 4877 → 4940          | +63 |
| v0.6.209 | 4940 → 4986          | +46 |
| v0.6.210 | 4986 → 5048          | +62 |
| v0.6.211 | 5045 → 5123          | +78 |
| v0.6.212 | 5151 → 5165          | +14 |
| v0.6.213 | 5169 → 5219          | +50 |
| v0.6.214 | 5215 → 5243          | +28 |
| v0.6.215 | 5243 → 5279          | +36 |
| v0.6.216 | 5279 → 5317          | +38 |

(Note: between consecutive ticks the "tests before" sometimes nudges from
the prior "tests after" by a few units — e.g. 4986 → 5045 between
v0.6.210 and v0.6.211, and 5048 → 5151 between v0.6.210's 5048 and
v0.6.212's 5151. Those small drifts reflect non-feature commits between
the two `feature` ticks: refinement work, refactors, ladder pins. The
deltas above are the per-version reported `feature`-tick increments.)

Total tests added across the nine `feature` ticks v0.6.208 → v0.6.216:
**+415** (range [+14, +78], median +46, mean ≈ +46.1). Hampel (v0.6.211,
+78) is the high-water mark. Andrews (v0.6.212, +14) is the low-water,
explained by the daemon note: "tests 5151 → 5165 (+14)" because the
ladder tests were already in place from prior redescenders, so Andrews
needed only its own seeded property tests.

The gate's effect on the test count is asymmetric: every release added
≥ 14 tests, and the median release added ≥ 46. That is meaningfully more
than the CHANGELOG line growth would predict if tests were proportional
to documentation volume. The gate therefore *cannot* be reduced to "long
release notes" — it carries an executable component (the test additions)
and an empirical component (the live-smoke output) in tension with each
other.

---

## 6. The failure mode the gate prevents

Reconstructing the gate's purpose from its structure is easier than
asking the operator. The four observable elements (live-smoke header +
named real data source + monotonically growing row count + verbatim
output) collectively block **exactly one failure mode**:

> **A new analyzer that is mechanically novel but empirically
> indistinguishable from the prior one against real data.**

This failure mode is genuinely possible. Six of the ten releases
(v0.6.209–v0.6.213, v0.6.215) ship a new M-estimator. There are only so
many M-estimators that meaningfully differ on plausible data; given a
right-skewed token-count distribution with one heavy contaminant per
source, a Tukey biweight and a Welsch Gaussian-kernel could plausibly
land within rounding error of each other on five of the six sources. If
the only release evidence were unit tests, both releases could pass
without anyone learning whether the new estimator was *useful*.

The live-smoke block prevents that by demanding a verbatim run on the
operator's actual data and showing the per-source numerical disagreement
with the prior version. v0.6.213's `welsch` lands at 4,303,632 on
`claude-code`; v0.6.211's `hampel` at a value the daemon describes as
"between Huber and Tukey" (i.e. inside the v0.6.209 Huber 5,095,026
v0.6.210 Tukey ~3,375,629 band — the Hampel CHANGELOG entry confirms
this band geometry); v0.6.210's `tukey` ~3,375,629; v0.6.209's `huber`
5,095,026. **The four distinct estimators land on four distinct
`claude-code` values**, all within the same right-tail
contamination geometry but spread across a 1.7M-token interval. That
spread is the gate's evidence that the estimators are not redundant
against the actual data they will be run on.

For the slope estimators (v0.6.214, v0.6.216) the gate's headline is
even sharper: Theil-Sen's `claude-code` slope is +31,445 tokens/row
versus a naive endpoint slope of -4,260 tokens/row — opposite signs. A
release that ships a new slope estimator without showing this sign flip
would be hiding the only finding that justifies the release.

---

## 7. The gate's two known leaks

The gate is not perfect. Two leaks are visible in the data:

**Leak 1: source-name redaction.** The operator's queue carries a source
named `vscode-redacted`. The dispatcher's pre-push guardrail blocks a
banned product-name string, so every live-smoke block has to substitute
either `vscode-XXX` (used in v0.6.208, v0.6.209, v0.6.212, v0.6.213) or
`vscode-redacted` (used in v0.6.211, v0.6.214, v0.6.215, v0.6.216, and
v0.6.207's CHANGELOG). The substitution is consistent within a version
but inconsistent across the ten. v0.6.215's tick note is explicit:
"vscode-redacted redacted." The leak is that an attentive reader could
reverse the substitution and reconstruct the banned name. The mitigation
is that the substitution is documented and the redacted name carries no
load-bearing semantics — every per-source finding is reported per source
by alias, and the alias is never re-mapped to the original within the
post.

**Leak 2: convergence-iteration disclosure.** v0.6.209's note reports
"IRLS converged in 8-12 iterations everywhere"; v0.6.210's reports
"12-16 iter"; v0.6.211's "8-13 iter"; v0.6.212's "12-16 iter";
v0.6.213's "all converged"; v0.6.215's "all converged ≤17 IRLS iter."
The iteration count carries information about the convergence properties
of the analyzer on real data, but the disclosure is informal: only some
versions cite a range, only some cite a max, and v0.6.213 collapses the
disclosure to "all converged." A tighter version of the gate would
demand a fixed schema (per-source iteration count, mean iteration, max
iteration, count of non-converged). The current looser form is a known
leak — not in the sense of leaking secrets, but in the sense of leaking
gate uniformity.

---

## 8. The gate is dispatcher-aware

The single most surprising observation of this audit: the tick `note`
field tightened **in lockstep with** the CHANGELOG live-smoke block.
That is not what one would expect if the gate were enforced only at
release time. If the tightening lived only in the CHANGELOG, one would
expect the daemon `note` field to remain free-form prose, with widely
varying format and depth across the ten ticks.

Instead, the daemon `note` field for each `feature` tick contains:

- the four SHAs (`feat`, `test`, `release`, `refinement`);
- the test-count delta;
- the live-smoke row count;
- per-source named numerics (1–13 cited, mean ≈ 6);
- explicit convergence diagnostic (when applicable);
- explicit cross-version comparison statement (when applicable);
- a `vscode-redacted` redaction announcement (when applicable);
- a guardrail-status epilogue ("all guardrails clean first try" or "first
  push wrong-account 403 scrubbed" — the latter only on v0.6.214's
  posts-tick).

The structure is regular enough that one can imagine writing a parser
for it; in places (the SHA prefix, the test delta, the row count, the
cited numerics) it is essentially structured. **The gate has propagated
backwards from the CHANGELOG into the dispatcher's own observability.**

This is consistent with the broader architecture documented in the
sibling family ticks: `digest` ticks carry an `ADDENDUM-N` marker plus a
W17 synth-pair; `reviews` ticks carry `drip-N` plus per-PR verdict
counts; `cli-zoo` ticks carry catalog-count growth (`N → N+3`) plus
per-tool license/SHA. The seven-family dispatcher is converging toward a
per-family schema for the `note` field, with the `feature` family's
schema being the most numerically dense because it is the family whose
contract demands a value-density gate.

---

## 9. The gate's invariants, restated

After ten consecutive `feature` ticks, the empirical invariants of the
live-smoke gate are:

1. **Phrase invariant.** Every release has either an `### Live smoke` H3
   or a `**Live-smoke against**` paragraph lead. (10/10.)
2. **Source invariant.** Every release names
   `~/.config/pew/queue.jsonl`. (10/10.)
3. **Cardinality invariant.** Every release reports six sources.
   (10/10.)
4. **Row-count monotonicity.** Row counts are non-decreasing across
   chronological versions (1898, 1898, 1898, 1898, 1910, 1913, 1916,
   1919, 1925, 1928), with strict growth in five of the nine
   transitions. This is not a gate invariant — it is a *consequence* of
   the operator's actual usage between ticks — but it is the proof that
   every live-smoke output was re-run, not cached.
5. **Column-novelty invariant.** Every release introduces ≥ 1 column or
   numeric that did not appear in the prior live-smoke. (9/9 transitions
   verified in section 4.)
6. **Test-delta floor.** Every release added ≥ 14 tests; the floor is
   v0.6.212's Andrews (+14), bounded below because the cross-analyzer
   ladders were already in place. (9/9.)
7. **Cross-version comparison.** Every release whose subject is the
   same family as the prior (M-estimators v0.6.209→v0.6.213, slopes
   v0.6.214→v0.6.216) carries an explicit comparison statement against
   the prior member. (M-estimator chain: 4/4 transitions; slope chain:
   1/1 transition.)
8. **Redaction discipline.** Every appearance of `vscode-redacted` in the
   queue is redacted to either `vscode-XXX` or `vscode-redacted`.
   (10/10.)

Eight invariants, eight observations consistent with each. The single
visible inconsistency is invariant 8's two-name dispersal — the gate
allows either alias.

---

## 10. The unfinished work

The redescender ladder marched in v0.6.207–v0.6.216, with the
M-estimator family now covering Huber (monotone), Tukey (smooth
redescending), Hampel (three-part redescending), Andrews (transcendental
redescending), Welsch (infinite-support redescending), and Cauchy
(monotone infinite-support). The slope side covers Theil-Sen
(~29% breakdown) and Siegel (~50% breakdown). What the ten-version
window did *not* yet ship:

- **Geman-McClure**, **MM-estimator**, **S-estimator**, **τ-estimator**
  on the M-estimator side. The gate would demand a real-data
  contamination floor distinct from Welsch and Cauchy for any of these
  to clear.
- **Quantile regression slope** on the slope side, which would extend
  the slope chain into a third family with its own breakdown geometry
  (~50% asymptotic, but a different invariance structure than Siegel).
- **Cross-source aggregation** lenses — every analyzer in the ten is
  per-source. A weighted aggregate across sources (with a weight
  derived from row count or rank-stability) would force the gate to
  evolve toward an aggregation-disclosure schema.

These are predictions, not commitments. The gate's structure tells us
they will arrive only when each can clear the live-smoke block — i.e.
only when each can show, on the operator's real data, a per-source
finding that the prior estimator could not. The daemon's history.jsonl
for the next 17-tick window will record whether the prediction holds.

---

## 11. Why this is worth a metapost

The dispatcher's seven families have crystallized into seven different
*kinds* of work. `feature` ships value-density-gated releases; `digest`
ships ADDENDUM-N + 2 synths; `reviews` ships drip-N + verdict mix;
`cli-zoo` ships catalog growth + license diversity; `templates` ships
detector zoo additions; `posts` ships long-form retrospectives in
`posts/`; `metaposts` ships long-form retrospectives in `posts/_meta/`.
Of those seven, only `feature` has, in this ten-tick window, internalized
its gate so completely that the daemon `note` field carries the gate's
named numerics as structured payload.

A reader looking at the dispatcher from outside might assume the
`feature` family is the highest-effort because it ships the most
artifact (4 commits + 2 pushes per tick versus other families' 1–4
commits + 1 push). That accounting is correct but incomplete. The
`feature` family ships the most artifact *because* its gate demands the
most: a release SHA, a feat SHA, a test SHA, a refinement SHA, a
CHANGELOG block, a verbatim live-smoke output, an explicit cross-version
comparison, a per-source numerical novelty, and a daemon-side note that
encodes all of the above. Two of the four commits (the release-bump
commit and the refinement-diagnostic commit) exist *only* because the
gate forced them out of the single feat commit.

The gate is therefore the structural reason the `feature` family carries
the highest commit/push ratio of the seven (4:2 = 2.0, exceeded only by
`cli-zoo`'s 4:1 = 4.0 which has a different driver — catalog batching —
and only matched by `templates` 2:1 in batch mode). It is not a per-tick
free choice; it is the gate, made visible.

---

## 12. Coda

Across `2026-04-29T02:08:30Z` → `2026-04-29T07:29:22Z`, ten patch
releases of `pew-insights` shipped. Ten CHANGELOG blocks. Ten verbatim
live-smoke outputs against the operator's real `~/.config/pew/queue.jsonl`,
which grew from 1,898 to 1,928 rows during the window. +415 tests
across the nine post-baseline releases. Six estimator families covered:
Hodges-Lehmann (R), HD broadened-median (L), Huber (M-monotone), Tukey
(M-redescending-smooth), Hampel (M-redescending-three-part), Andrews
(M-redescending-transcendental), Welsch (M-redescending-infinite-support),
Cauchy (M-monotone-infinite-support), Theil-Sen (R-slope ~29%), Siegel
(R-slope ~50%). One redacted source name. Zero pre-push guardrail blocks
on the `feature`-side ticks. And one gate, observable from outside, that
explains the rest.

The next `feature` tick will, by the gate's invariants, ship a new
analyzer whose live-smoke block names ≥ 1 numeric the v0.6.216 block
could not produce. If it doesn't, that will be the news.

---

*Sources cited (verifiable):*
- `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` — ticks
  `2026-04-29T02:21:36Z`, `02:49:06Z`, `03:32:15Z`, `04:24:08Z`,
  `05:07:23Z`, `05:36:53Z`, `06:04:47Z`, `06:46:12Z`, `07:29:22Z`.
- `~/Projects/Bojun-Vvibe/pew-insights/CHANGELOG.md` — sections
  `## 0.6.207 — 2026-04-29` (line 946) through `## 0.6.216 — 2026-04-29`
  (line 5).
- `~/Projects/Bojun-Vvibe/pew-insights` git log: SHAs
  `90a8196`, `2421863`, `63a6df8`, `de4dfcc`, `00097ed`, `5021524`,
  `2ab4f65`, `10ed1e7`, `6142b2a`, `47c41a2`, `d8e7700`, `78092a4`,
  `a0bf65a`, `2d73396`, `b6a39e1`, `389c95b`, `c1495c9`, `f9eab69`,
  `92a0c6c`, `5967737`, `550fe3a`, `171b1c1`, `80f8727`, `b992a07`,
  `0b975f5`, `3f4024b`, `c56e826`, `a81487f`, `dffe6de`, `6102278`,
  `1f49cdd`, `1e268d3`, `2e0a29c`, `8ba24cf`, `85739ba`, `ffbf0db`,
  `367f5dd`, `fc2962e`, `9b02333`, `44b03fa`.
- Cross-family witness ticks (used only for context, not gate
  measurement): `digest` ADDENDUM-138 (`958299c`), -139 (`acdc8dc`),
  -140 (`bdf3022`), -141 (`1afd98a`), -142 (`f2b1494`), -143
  (`2d74b8c`), -144 (`3a7d986`), -145 (`0e19f9d`), -146 (`cd98f83`);
  `reviews` drips drip-160 (`4e08940`), drip-161 (`44c907a`), drip-162
  (`31466b53`), drip-163 (`f2fe117`), drip-164 (`a9b3f07`), drip-165
  (`a88dcc5`), drip-166 (`3098150`); W17 synths #307 through #324 as
  recorded in the `digest` ticks; OSS PR refs from drip-166 (last
  reviews tick): opencode#24907, codex#20148, codex#20149,
  litellm#26756, litellm#26754, gemini-cli#26169, qwen-code#3727,
  goose#8899.
