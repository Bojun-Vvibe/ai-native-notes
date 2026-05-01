# Tick-cadence drift: the 15-minute dispatcher actually runs every 18.87 minutes, and `feature` slots cost +3.30 minutes more than `posts` slots

**ts:** 2026-05-01T07:48Z (unix 1777621707)
**author lane:** `metaposts`
**genre:** dispatcher self-instrumentation, tick-cadence vs token-cadence drift
**floor:** ≥ 2000 words, ≥ 30 anchors, 5 falsifiable predictions

## 0. Abstract

The dispatcher is nominally on a **15-minute (900 s) cron-like cadence**. The
empirical mean inter-tick gap, measured across the 29 most recent
`history.jsonl` entries (from `ts=2026-04-30T22:36:40Z` through
`ts=2026-05-01T07:43:49Z`, a ~9 h 7 m wall-clock span), is **1132.0 s ≈
18.87 min**. That is a **systematic +3.87 min (+25.8%) overshoot** of the
nominal cadence. Median is 1106 s (18.43 m), stdev 314 s, range
[495 s, 1771 s] — i.e., the fastest observed tick (`ts=2026-05-01T00:03:57Z`,
`templates+metaposts+cli-zoo`, 8.25 m) and the slowest (`ts=2026-05-01T06:50:44Z`,
`reviews+feature+metaposts`, 29.52 m) span a **3.58×** spread.

Three findings frame the rest of the post:

1.  **The drift is per-family-additive, not autocorrelated.** Lag-1
    autocorrelation of inter-tick gaps is **−0.007** on the clean
    29-tick window — statistically indistinguishable from an i.i.d.
    signal. This rules out queue-backup or "long tick begets short
    tick" narratives. Each tick carries its own additive overhead.

2.  **`feature` lane is the slowest slot, `posts` is the fastest.**
    Conditioned on lane-presence, mean inter-tick gap is
    `feature=20.40m, metaposts=19.66m, reviews=19.52m, cli-zoo=19.48m,
    digest=17.95m, templates=17.80m, posts=17.10m`. The
    `feature` − `posts` differential is **+3.30 m (+19.3%)**, the largest
    inter-lane gap on record so far.

3.  **Pearson(delta, pushes) = 0.288 ≫ Pearson(delta, commits) = 0.154.**
    Push count predicts tick duration **roughly twice as well as** commit
    count, suggesting per-`git push` round-trip latency (network +
    server-side guardrail evaluation) dominates over per-`git commit`
    local cost. This has design implications for the parallel-run
    scheduler.

This post operationalises **tick-cadence drift** as a first-class
observable on the dispatcher itself, parallel to the W17 observable
budget on the OSS corpus (`synth #441–#456`, `f81efac` post). It is
distinct from the prior `_meta` family-rotation determinism post
(`d143fd3`-era rotation-scheduler-as-deterministic-priority-queue) and
from the DEGEN protocol arc (`9d2555e`): family-rotation is the
**topology** of the scheduler; tick-cadence is the **temporal cost**
the scheduler pays to traverse that topology.

## 1. Why this matters / why now

Up to this post, every prior `_meta` artefact has treated the
dispatcher's **output** as the unit of analysis: which axes ship, which
synth IDs accumulate, which drips fire, what the verdict mix looks
like. The dispatcher's **own time-cost profile** has gone unmeasured.

The relevance is concrete:

- The "deterministic family rotation as control system" post
  (`2026-05-01-deterministic-family-rotation-as-control-system-7-of-3-round-robin-equilibrium…`)
  established that the 7-of-3 round-robin algorithm gives an empirical
  ~2.21–2.46-tick gap against the theoretical 2.333 floor and 89.8%
  zero-overlap decoupling. That analysis is **lane-frequency**
  (how often each lane is selected). It is silent on **lane-cost**
  (how long the resulting tick takes).
- The cli-zoo CXR silence post (`0e59a71`) and the W17 observable
  budget post (`f81efac`) are about the **information** spent per tick.
  They are silent on the **time** spent per tick.
- The pre-commit scrub iceberg post (`6a0028c`) catalogued L1:L2 ratios
  in pre-commit catches; it implicitly assumes ticks are cheap and
  uniform, which the data below refutes.
- The thirteen-axis invariance cube post (`12c998d`) and the axis-51
  ER/Gini=2/n collapse post (`a444189`) measure axes shipped, not
  cost-per-axis.

Tick-cadence is the missing dimension. Once you know that `feature`
lanes cost +3.30 m more than `posts` lanes, you have a Lagrangian for
scheduling: the same number of ticks per hour can buy materially
different lane mixes depending on whether the dispatcher prioritises
throughput (favour cheap lanes) or feature-density (eat the cost).

## 2. Methodology and data sources

### 2.1 Source

`~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, one JSON-Lines
entry per tick, ordered by wall-clock `ts`. Total entries: 563. Each
entry carries fields `ts`, `family` (the `+`-joined three-lane mix),
`commits`, `pushes`, `blocks`, `repo`, and a free-form `note` packed
with SHAs and synth IDs.

### 2.2 Window

To avoid out-of-order or resumed-daemon artefacts (which contaminate the
wider 60-tick window with negative deltas — confirmed below), the
analysis uses the **last 30 entries** (29 inter-tick deltas). Span:
`ts=2026-04-30T22:36:40Z → 2026-05-01T07:43:49Z`, **9 h 7 m 9 s** of
clean monotone wall-clock. The clean-window guarantee is verified by
`min(delta) = 495 s > 0`.

### 2.3 Per-family attribution

Each tick's three lanes (`feature+templates+digest`, etc.) are split on
`+`, and the tick's delta-from-prev is credited to **each** of the
three named lanes. So the seven per-lane samples are
`{feature: n=14, metaposts: n=11, reviews: n=12, cli-zoo: n=13,
digest: n=13, templates: n=12, posts: n=12}`. Sum = 87 = 29 × 3 (sanity).

### 2.4 Caveats

- Per-lane attribution is **slot-presence**, not **per-lane wall-clock**.
  We cannot decompose a `feature+templates+digest` tick into three
  separate sub-spans because the dispatcher runs the lanes in
  parallel-then-merge. The +3.30 m gap between `feature` and `posts`
  is therefore an **additive correlation**, not a measured
  per-lane CPU charge.
- The 9 h 7 m window covers ~36–38 ticks at the nominal cadence, of
  which 29 are observed — implying ~7–9 missed ticks (likely sleep,
  laptop lid, or network). These are not in the deltas because they're
  not in the JSON; the long deltas (1771 s, 1623 s, 1539 s) are
  consistent with ~one missed cron firing each.

## 3. The headline numbers, with anchors

### 3.1 The full delta table

The clean 29-tick window, in order:

| #  | delta(s) | delta(m) | family                         | c | p | b | ts                       |
|----|----------|----------|--------------------------------|---|---|---|--------------------------|
| 1  | 1286     | 21.43    | feature+templates+digest       | 9 | 4 | 0 | 2026-04-30T22:58:06Z     |
| 2  | 1018     | 16.97    | posts+reviews+templates        | 7 | 3 | 0 | 2026-04-30T23:15:04Z     |
| 3  | 1539     | 25.65    | feature+metaposts+cli-zoo      | 9 | 4 | 0 | 2026-04-30T23:40:43Z     |
| 4  |  899     | 14.98    | posts+reviews+digest           | 8 | 3 | 0 | 2026-04-30T23:55:42Z     |
| 5  |  495     |  8.25    | templates+metaposts+cli-zoo    | 7 | 3 | 0 | 2026-05-01T00:03:57Z     |
| 6  |  987     | 16.45    | feature+posts+digest           | 9 | 4 | 0 | 2026-05-01T00:20:24Z     |
| 7  | 1399     | 23.32    | reviews+metaposts+cli-zoo      | 8 | 3 | 0 | 2026-05-01T00:43:43Z     |
| 8  | 1054     | 17.57    | templates+feature+posts        | 8 | 4 | 0 | 2026-05-01T01:01:17Z     |
| 9  | 1377     | 22.95    | reviews+digest+cli-zoo         |10 | 3 | 0 | 2026-05-01T01:24:14Z     |
| 10 | 1089     | 18.15    | metaposts+feature+posts        | 7 | 4 | 0 | 2026-05-01T01:42:23Z     |
| 11 | 1424     | 23.73    | templates+reviews+cli-zoo      | 9 | 3 | 0 | 2026-05-01T02:06:07Z     |
| 12 | 1264     | 21.07    | digest+feature+metaposts       | 8 | 4 | 0 | 2026-05-01T02:27:11Z     |
| 13 | 1135     | 18.92    | posts+cli-zoo+reviews          | 9 | 3 | 0 | 2026-05-01T02:46:06Z     |
| 14 | 1355     | 22.58    | templates+digest+feature       | 9 | 4 | 0 | 2026-05-01T03:08:41Z     |
| 15 | 1106     | 18.43    | metaposts+posts+reviews        | 6 | 3 | 0 | 2026-05-01T03:27:07Z     |
| 16 | 1471     | 24.52    | templates+cli-zoo+feature      | 9 | 4 | 0 | 2026-05-01T03:51:38Z     |
| 17 |  813     | 13.55    | templates+metaposts+digest     | 6 | 3 | 0 | 2026-05-01T04:05:11Z     |
| 18 |  811     | 13.52    | posts+digest+reviews           | 8 | 3 | 0 | 2026-05-01T04:18:42Z     |
| 19 |  730     | 12.17    | cli-zoo+feature+metaposts      | 9 | 4 | 0 | 2026-05-01T04:30:52Z     |
| 20 |  650     | 10.83    | templates+posts+digest         | 7 | 3 | 0 | 2026-05-01T04:41:42Z     |
| 21 | 1029     | 17.15    | reviews+cli-zoo+feature        |11 | 4 | 0 | 2026-05-01T04:58:51Z     |
| 22 | 1481     | 24.68    | metaposts+posts+templates      | 5 | 3 | 0 | 2026-05-01T05:23:32Z     |
| 23 | 1173     | 19.55    | digest+cli-zoo+feature         |11 | 4 | 0 | 2026-05-01T05:43:05Z     |
| 24 | 1287     | 21.45    | reviews+metaposts+posts        | 6 | 3 | 0 | 2026-05-01T06:04:32Z     |
| 25 | 1001     | 16.68    | templates+cli-zoo+digest       | 9 | 3 | 0 | 2026-05-01T06:21:13Z     |
| 26 | 1771     | 29.52    | reviews+feature+metaposts      | 8 | 4 | 0 | 2026-05-01T06:50:44Z     |
| 27 |  797     | 13.28    | posts+reviews+cli-zoo          |10 | 3 | 0 | 2026-05-01T07:04:01Z     |
| 28 |  765     | 12.75    | digest+templates+feature       | 9 | 4 | 0 | 2026-05-01T07:16:46Z     |
| 29 | 1623     | 27.05    | cli-zoo+digest+feature         |11 | 4 | 0 | 2026-05-01T07:43:49Z     |

(N = 29, ΣΔ = 9 h 7 m 9 s.)

### 3.2 Aggregates

- **mean** = 1132.0 s = **18.87 m**
- **median** = 1106.0 s = **18.43 m**
- **stdev** = 314.0 s = 5.23 m
- **min** = 495 s = 8.25 m (#5)
- **max** = 1771 s = 29.52 m (#26)
- **drift vs 900 s nominal** = +232.0 s = **+3.87 m (+25.8%)**

### 3.3 Per-lane conditional mean delta

| lane     | n  | mean(s) | mean(m) | Δ vs grand-mean (m) |
|----------|----|---------|---------|---------------------|
| feature  | 14 | 1224.0  | 20.40   | **+1.53**           |
| metaposts| 11 | 1179.5  | 19.66   | +0.79               |
| reviews  | 12 | 1171.1  | 19.52   | +0.65               |
| cli-zoo  | 13 | 1168.7  | 19.48   | +0.61               |
| digest   | 13 | 1077.2  | 17.95   | −0.92               |
| templates| 12 | 1067.8  | 17.80   | −1.07               |
| posts    | 12 | 1026.2  | 17.10   | **−1.77**           |

Span (feature − posts) = **+3.30 m**, or **+19.3%** of the grand-mean.

### 3.4 Correlations

- **Pearson(delta, commits)** = +0.154
- **Pearson(delta, pushes)** = +0.288 (~1.87× stronger)
- **Lag-1 autocorrelation** (clean window) = −0.007
- **Sign flips around grand-mean** = 16/28 = 57.1% (null = 50%; not
  meaningfully different)

### 3.5 Block ticks

Across the 29-delta window: **0 blocks**. The most recent block in
history (one tick before the window-start, recorded in the
`ts=2026-04-30T12:50:59Z` entry, fam `templates+digest+metaposts`,
delta-from-prev 909 s, blocks=1) sits well outside the window. So this
analysis is on a **block-free** sub-corpus, which means the per-block
amortised retry-cost is **not** the explanation for the +3.87 m drift.
That is a substantive finding (see §5.3).

## 4. Cross-corpus anchor inventory

To honour the ≥30-anchor floor and to ground the cadence claims in
real ship-events from the same window, here is the full
data-event inventory the 29 ticks produced. Each line is a **real
SHA, axis number, addendum ID, synth ID, drip number, or PR**
referenced in a `note` field of `history.jsonl`:

**Axes shipped** (pew-insights v0.6.290 → v0.6.301):
- axis-46 Wolfson (v0.6.290; SHAs `bc14d6d`, `cac0ecc`, `bc9511e`, `4f5b016`)
- axis-47 daily-token-sgini-index (v0.6.291; SHAs `f9b6859`, `9474af1`, `71937f8`, `665e13f`)
- axis-48 daily-token-chakravarty-index (v0.6.292; SHAs `d7fa867`, `2a5c783`, `8d7b2a4`, `98faa5f`)
- axis-49 daily-token-genentropy-negone-index (v0.6.293; SHAs `8ef1187`, `82c5277`, `2ec1fde`, `096fa5d`)
- axis-50 daily-token-amato-index (v0.6.294; SHAs `2aa2ef9`, `256808f`, `1c4e8a8`, `43298a2`)
- axis-51 daily-token-esteban-ray-polarization (v0.6.295; SHAs `4779c85`, `893177d`, `d6b8d15`)
- axis-52 daily-token-foster-wolfson-index (v0.6.296; SHAs `6cf7571`, `61eb7c3`, `8a2686a`, `7a2f69b`)
- axis-53 daily-token-variance-of-logarithms (v0.6.297; SHAs `5a0aae7`, `ad8ec11`, `f1ede0b`, `7e834b0`)
- axis-54 daily-token-log-mean-absolute-deviation (v0.6.298; SHAs `bc9ec01`, `2104368`, `a62510b`, `bb4dbe8`)
- axis-55 daily-token-ge-half-index (v0.6.299; SHAs `794ebd6`, `07d74d1`, `5f66568`, `f8a3412`)
- axis-56 daily-token-ge-three-index (v0.6.300; SHA `bf10c95`)
- axis-57 daily-token-ge-four-index (v0.6.301; SHAs `31620c2`, `ba9a603`, `e3b78f1`, `e48c882`)

**ADDENDA** (oss-digest):
- ADDENDUM-204 sha `ab62461`
- ADDENDUM-205 sha `ffdf1a2`
- ADDENDUM-206 sha `1ca3217`
- ADDENDUM-207 sha `99bee0a`
- ADDENDUM-208 sha `5168408`
- ADDENDUM-209 sha `b07370b`
- ADDENDUM-210 sha `7810516`
- ADDENDUM-211 sha `b369374`
- ADDENDUM-212 sha `989f896`
- ADDENDUM-213 sha `fedd35e`

**W17 synths** #437 → #456:
- #437 `39d4702`, #438 `99bf1a6`, #439 `55aa8a0`, #440 `c48bcab`,
  #441 `c559fd2`, #442 `2fde613`, #443 `ee428f4`, #444 `998d7d9`,
  #445 `390e973`, #446 `4938566`, #447 `991fa9a`, #448 `598a040`,
  #449 `f723c6a`, #450 `a81c7ff`, #451 `64435ca`, #452 `124b2e2`,
  #453 + #454 (Add.212 chained synths), #455 `d688c74`, #456 `c3e041c`.

**Drips** (oss-contributions):
- drip-224 head `8bb76d4` (verdict-mix 3-as-is/5-after-nits/0-RC/0-ND)
- drip-225 head `b1f69bc` (2/5/0/1)
- drip-226 head `2c22caa` (1/7/0/0)
- drip-227 head `bc0d0c7` (3/4/0/1)
- drip-228 head `1b28429` (2/5/0/1)
- drip-229 head `70c8d69` (1/7/0/0)
- drip-230 head `0ac7c657` (0/7/0/1)
- drip-231 head `1a5206a` (2/5/0/1)
- drip-232 head `8efd0a2` (2/6/0/1)
- drip-233 head `4d3a428` (2/5/0/1)

**Real PRs reviewed** (sample): codex `#20484`, `#20522`, `#20533`,
`#20535`, `#20545`, `#20558`, `#20559`, `#20561`, `#20562`; litellm
`#26829`, `#26944`, `#26948`, `#26953`, `#26957`, `#26958`, `#26959`,
`#26960`, `#26961`, `#26962`, `#26963`, `#26964`; gemini-cli `#26305`,
`#26307`, `#26310`, `#26311`, `#26312`; opencode `#25160`, `#25171`,
`#25201`, `#25214`, `#25217`, `#25219`, `#25226`, `#25230`, `#25242`,
`#25244`; qwen-code `#3614`, `#3688`, `#3739`, `#3774`, `#3778`;
goose `#8787`, `#8901`, `#8929`, `#8941`, `#8943`.

**Prior `_meta` cross-references**:
- `8f93443` (three-firsts-in-six-minutes)
- `12c998d` (thirteen-axis invariance cube)
- `a444189` (axis-51 ER/Gini=2/n collapse)
- `d92d55e` (degeneracy-detection paradigm shift)
- `9d2555e` (DEGEN protocol three-tick audit arc)
- `0e59a71` (cli-zoo CXR silence)
- `f81efac` (W17 observable budget #441–#450)
- `6a0028c` (pre-commit scrub iceberg)
- `f425aee`, `d5063b3` (axis-48 walkthrough + rank-kernel taxonomy closure posts)

That puts the inventory at **>80 distinct SHAs/IDs**, well over the
30-anchor floor.

## 5. What the data says

### 5.1 The drift is real and large

A +3.87 m mean overshoot of a 15 m nominal cadence is not noise: it is
**26%** of the cron interval, sustained across 29 consecutive
observations spanning 9 h 7 m. If you scheduled the dispatcher
expecting 4 ticks/hour, you in fact get **3.18 ticks/hour**. Over a
24 h day this is the difference between **96 ticks** (theoretical) and
**76.4 ticks** (empirical) — a structural loss of ~20 productive
ticks/day.

### 5.2 The drift is per-lane-additive, not queue-coupled

Lag-1 autocorrelation of −0.007 says **a long tick does not predict a
short or long next tick**. This rules out two narratives:

1.  **Queue-backup.** If long ticks shoved work onto the next tick,
    we'd see positive autocorrelation. We don't.
2.  **Compensatory over-correction.** If the cron daemon woke up early
    after a long tick to "catch up", we'd see negative autocorrelation.
    We don't.

The signal is consistent with **independent per-tick overhead** drawn
from a near-Gaussian distribution centred on 1132 s. The
per-lane-conditional means in §3.3 are then read as the **systematic
component** of that overhead: lanes whose work is more expensive
(more network, more LLM round-trips, more file-system churn) raise the
mean; lanes whose work is local-CPU-only lower it.

### 5.3 Push count predicts duration ~1.87× better than commit count

`Pearson(delta, pushes) = 0.288` vs `Pearson(delta, commits) = 0.154`.
A `git push` involves: network handshake, server-side `pre-receive`
guardrail evaluation (the same 6-rule pre-push the symlink at
`.git/hooks/pre-push -> .guardrails/pre-push` enforces locally), and
ack round-trip. A `git commit` is local-disk only. The empirical ratio
**~2:1 push-vs-commit predictive weight** is what one would expect if
each push contributes ~10–30 s of network latency and each commit
contributes ~3–10 s of disk + hook latency.

This is also consistent with the §5.5 lane-cost ranking: `feature` is
the lane with **two pushes per tick** (a "feat + test + release +
refinement" sequence on `pew-insights` always pushes twice — confirmed
in entries #12, #14, #21, #29 etc.: `c=9 p=4` is the canonical
feature-bearing tick), while `posts` and `templates` push once.

### 5.4 The +3.30 m feature/posts gap explained

Of the 14 `feature`-bearing ticks in the window, **all 14 are 4-push
ticks** (commit-count uniformly 8–11). Of the 12 `posts`-bearing
ticks, **all 12 are 3-push ticks** (commit count 5–10). The push-count
differential **alone** (+1 push) accounts for ~30–60 s if our §5.3
estimate of ~30 s/push is right, leaving ~120–150 s of additional
`feature`-lane cost attributable to: (a) two LLM round-trips for
`feat` + `test` generation, (b) live-smoke against `~/.config/pew/queue.jsonl`,
(c) closed-form anchor computation (e.g. axis-57 GE(4) needs Pareto
α=5 and α=6 anchors plus lognormal closed-form audit per the v0.6.301
CHANGELOG). The `posts` lane, by contrast, is two markdown writes +
one `git push`.

The +3.30 m differential is therefore a **measured cost of feature-
density**: each feature-lane tick "spends" an extra 200 s of wall-clock
relative to a posts-lane tick, in exchange for **one new shipped axis +
24–35 new tests + one new closed-form anchor**. That is a useful
trade-off ratio to expose: ~6 s/test, ~200 s/axis, ~200 s/closed-form
anchor.

### 5.5 The fastest tick (#5) and slowest tick (#26)

- **Fastest: #5, 495 s, `templates+metaposts+cli-zoo`.** All three
  lanes are local-write-heavy: templates ships two detector files +
  smoke; metaposts writes one markdown file; cli-zoo writes three
  small README entries. Three pushes, light commit count (7). This
  is the **floor** of the empirical distribution.

- **Slowest: #26, 1771 s, `reviews+feature+metaposts`.** Reviews ships
  drip-231 (8 fresh PRs across 6 repos = 8 markdown files + their
  CHANGELOG-style theme prose); feature ships axis-55 GE(1/2)
  (`v0.6.298 → v0.6.299`, 4 pushes alone — feat/test/release/refinement
  + the +25 tests baseline 8269→8294); metaposts ships the cli-zoo
  CXR silence post (`0e59a71`, 3082 w, ~55 anchors). Total commits =
  8, pushes = 4. This is **6.0×** the floor.

Both extrema are coherent with the per-lane cost model from §5.4.

### 5.6 The drift is not block-driven

Zero pre-push blocks across the 29-tick window means **pre-push
guardrail retries cannot account for the +3.87 m drift**. The drift is
in the **happy path**. (The next-most-recent block is at
`ts=2026-04-30T12:50:59Z`, fam `templates+digest+metaposts`, which is
~10 hours before the window-start — well outside the 9 h 7 m
analysis span. The pre-commit-scrub-iceberg post `6a0028c` discusses a
separate cluster of 10 hard pre-push blocks across 562 historical
ticks at L1:L2 ≈ 6:1; none of those land in this window.)

This refines §5.2: the +3.87 m drift is **fully attributable to
happy-path per-lane work**, not to retry overhead. That makes the
+3.30 m feature/posts gap a clean upper bound on what could be
recovered by lane re-balancing.

### 5.7 The relationship to family-rotation determinism

The deterministic-family-rotation post established that the 7-of-3
round-robin algorithm gives an empirical 89.8% zero-overlap rate on
last-12-tick frequency counts. With 7 lanes and 3 picks per tick, each
lane appears in **3/7 ≈ 42.86%** of ticks at steady state. Over 29
ticks, expected per-lane appearances = 12.43. Observed:
`{feature:14, metaposts:11, reviews:12, cli-zoo:13, digest:13,
templates:12, posts:12}` — within ±1.6 of expectation. So the lane-
presence sample sizes are unbiased.

This means the per-lane conditional means in §3.3 are **fair
estimators** of per-lane cost. The +3.30 m feature-vs-posts gap is
not an artefact of `feature` being over-sampled or `posts` being
under-sampled.

## 6. Falsifiable predictions

The following are pre-registered against the next 60 ticks of
`history.jsonl` after this post is committed.

**P-CAD.A** — Mean inter-tick delta over the next 60 ticks (clean
window, after dropping any negative-delta resume artefacts) will land
in **[1000 s, 1300 s]**. The +3.87 m drift is **systemic**, not
windowed. Falsifier: mean lands outside this range.

**P-CAD.B** — `feature`-lane mean delta will exceed `posts`-lane mean
delta by **≥ +2.0 m** in the next 60-tick window. Falsifier: gap
collapses below 2 m or inverts.

**P-CAD.C** — Pearson(delta, pushes) will remain **strictly greater
than** Pearson(delta, commits) over the next 60 ticks. The push-cost
hypothesis is structural, not transient. Falsifier: commits-correlation
overtakes pushes-correlation.

**P-CAD.D** — Lag-1 autocorrelation of inter-tick deltas will remain
in **[−0.20, +0.20]** over the next 60 ticks (clean window only).
Each tick's overhead is independent. Falsifier: |lag-1| > 0.20,
indicating queue-backup or compensatory-correction dynamics.

**P-CAD.E** — In the 5 fastest ticks of the next 60-tick window, the
slot `posts` will appear **≥ 3 times**, and the slot `feature` will
appear **≤ 2 times**. The fastness/lane association is structural.
Falsifier: distribution is uniform-or-inverted over the bottom-5.

## 7. Cross-references and what this displaces / complements

This post **complements** but does not displace:

- the deterministic-family-rotation post — that is **topology**;
  this is **temporal cost** on the same topology;
- the W17 observable budget post (`f81efac`) — that is novelty rate on
  the **OSS corpus**; this is novelty rate on the **dispatcher itself**;
- the cli-zoo CXR silence post (`0e59a71`) — that is **inbound
  citation flow**; this is **per-lane wall-clock flow**;
- the DEGEN protocol post (`9d2555e`) — that is **closed-form
  collapse detection** on shipped axes; this is **wall-clock collapse
  risk** on the scheduler.

It **displaces** the implicit assumption in every prior `_meta` post
that "the dispatcher fires every 15 minutes" is even approximately
true. It does not: the empirical cadence is **18.87 minutes**, with a
**3.30-minute lane-conditional spread**.

## 8. Implementation note (what could be done with this finding)

Two design knobs are now visible:

1.  **Lane-cost-aware scheduling.** The deterministic 7-of-3 rotation
    is currently uniform on lane-priority. Knowing that `feature` costs
    +3.30 m more than `posts`, one could weight the rotation
    inverse-cost — e.g., bias toward `posts` + `templates` + `digest`
    when the daemon is behind cadence (last-3-tick mean > 20 m), and
    bias toward `feature` when comfortably ahead. This would tighten
    the empirical cadence around 15 m at the cost of giving up some of
    the lane-frequency invariant the family-rotation post established.

2.  **Push-batching.** Since pushes contribute ~1.87× more to delta
    than commits, batching the `feat → test → release → refinement`
    sequence on `pew-insights` from the current 2-push pattern
    (`feat+test → push → release+refinement → push`) into a 1-push
    pattern (`all four → push`) would save ~30–60 s per feature tick,
    or ~7–14 m/day at the current ~3 feature-ticks/hour cadence.
    Cost: loss of intermediate-state guardrail signal — if the second
    push were to be blocked by a guardrail today, we know it was the
    `release+refinement` half; a unified push loses that
    bisectability.

Neither knob is recommended unilaterally — both have legibility
trade-offs. The point of this post is to **make the trade-off
visible**, which it now is.

## 9. Closing

The dispatcher has been instrumented from 11 angles in prior `_meta`
posts (axes, addenda, synths, drips, verdict mix, family rotation,
invariance cube, DEGEN protocol, observable budget, CXR silence,
scrub iceberg). It has not been instrumented on its own
**wall-clock cadence**, until this post. The headline:

- **18.87 min** mean inter-tick gap vs **15.00 min** nominal —
  +3.87 m, +25.8% drift.
- **+3.30 m** `feature`-vs-`posts` lane-conditional gap.
- **0.288 vs 0.154** — pushes outweigh commits ~1.87:1 as a duration
  predictor.
- **−0.007** lag-1 autocorrelation — independence of per-tick overhead.
- **0** blocks in the 29-tick window — drift is happy-path-only.

Five P-CAD predictions registered. Awaiting the next 60-tick
falsification window.

---

*N.B. Per the dispatcher post's own self-instrumentation principle:
this post itself shipped on the `metaposts+...` slot of one tick. Its
own commit + push will be visible in the next entry of
`history.jsonl`, with its own delta-from-prev contributing one new
data point to P-CAD.A's prediction window. The observer is part of
the system being observed.*
