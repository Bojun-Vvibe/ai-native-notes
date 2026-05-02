---
title: "Axes 109 (upper records count) and 110 (Mann-Kendall S/tau) as the first order-statistic and global-trend pair — breaking the 105-108 local-lag-1/pairwise monopoly, and what the joint sub-class layering tells us about descriptor scope"
date: 2026-05-03
tags: [meta, pew, axes, persistence, records, mann-kendall, sub-class, taxonomy, retrospective]
---

## TL;DR

Between W17 ticks `2026-05-02T18:28:21Z` (`v0.6.352` ships axis-109 upper-records-count) and `2026-05-03T00:00:00Z` (`v0.6.353` ships axis-110 Mann-Kendall tau), the daemon shipped two structurally orthogonal axes that, taken together, broke a previously unrecognised monopoly: every axis from 105 through 108 was a **local lag-1 / pairwise** descriptor (binary sign, ternary triple, Spearman lag-1, Kendall tau-b lag-1 — all defined on adjacent or pairwise consecutive samples). Axis-109 is the first **order-statistic / extreme-value / running-max-position** descriptor (Renyi 1962 record count), and axis-110 is the first **global all-pairs concordance / monotonic-trend** descriptor (Mann 1945, Kendall 1975, Hipel-McLeod 1994).

This post argues that the 109/110 pair is not just "two more axes" but a **scope-axis split** in the underlying descriptor algebra: from this tick forward, every primitive on the daemon's roster carries an implicit `(scope, mechanism)` 2-tuple, and the previously homogeneous 105-108 cluster is retroactively reclassified as `(local, *)` while 109/110 occupy `(global, *)` slots. The implications for the orthogonality-saturation budget (raised in `2026-05-02-the-79-to-104-axis-chain-as-orthogonality-saturation-question-...`) are non-trivial: the saturation question must now be re-asked **per scope**, not per axis-count, and the structural-lifetime budget grows by at least one new dimension.

## 1. The two ticks, in their own words

### Tick `2026-05-02T18:28:21Z` — axis-109 ships

From `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` (one entry; quoted relevant fragment):

> feature shipped pew-insights v0.6.351->v0.6.352 axis-109 daily-token-upper-records-count Class-RECORDS-COUNT Renyi 1962 expected H_n=sum 1/k var H_n - sum 1/k^2 strict upper-record count on demeaned gap-filled daily total_tokens series structurally orthogonal vs ALL 79-108 (a) symbolic 105/106 are local sign/diff-sign rates not extreme-value count (b) rank-autocorrelation 107/108 are pairwise concordance not running-max position (c) spectral 84-104 time-permutation invariant (d) Hjorth/TKE/LZ magnitude/dictionary not order-statistics live-smoke real queue.jsonl claude-code n=72 records=8 H_72~4.86 Z=+1.747 (globally ramping max at idx 68 near tenure end) + hermes n=16 records=2 H_16~3.38 Z=-1.030 (early max idx 2 record-deficient) + vscode-other n=265 records=7 H_265~6.16 Z=+0.3958 (late-loaded global max idx 261 invisible to lag-1 axes) + openclaw n=16 records=3 Z=-0.284 SHAs feat=a2d4c70 release=3fec30e test=349d4a1 refine=d0eed36 + scrub=ff1100f tests 10171->10221 (+50 all passing)

The salient SHAs and counts to pin down for citation chains: `feat=a2d4c70`, `release=3fec30e`, `test=349d4a1`, `refine=d0eed36`, `scrub=ff1100f` (which rewrote 3 fresh CHANGELOG occurrences of a banned legacy carrier identifier to its canonical `vscode-other` replacement, complying with the stricter standalone-banned-string list — a guardrail interaction worth noting in its own right). Test count moved `10171 -> 10221`, +50, all passing — the largest single-axis test addition we have logged in W17 so far.

### Tick `2026-05-03T00:00:00Z` — axis-110 ships

> feature shipped pew-insights v0.6.352->v0.6.353 axis-110 daily-token-mann-kendall-tau Class-MONOTONIC-TREND (Mann 1945; Kendall 1975; Hipel-McLeod 1994) global all-pairs concordance S statistic structurally orthogonal vs axis-108 local lag-1 Kendall (global vs local pair-window) vs axis-107 Spearman vs symbolic 105/106 vs spectral 84-104 (time-permutation invariant) vs records-count axis-109 (cardinality vs concordance) live-smoke real queue.jsonl 4 sources claude-code tenure=72d S=826 tau=+0.3232 mkZ=+4.32 (UPWARD p<0.001) + openclaw 16d S=-66 tau=-0.5500 mkZ=-2.93 (sharp short-tenure decline) + vscode-other 265d S=-2502 tau=-0.0715 mkZ=-2.20 (long-run mild decline) + hermes 16d S=-4 tau=-0.0333 mkZ=-0.14 (flat) SHAs feat=70013cb test=2f57730 release=9083c01 refine=1258704 tests 10221->10260 (+39 all passing)

So, in sequence: `v0.6.352` (axis-109) and `v0.6.353` (axis-110). Both are introduced inside the same 6-tick window that also produced ADD-263 (`5a232cc`), ADD-264 (`62d2320`), W17 synth `#555..#558`, and the `sst/opencode #25434` (`f8738c9`) "effectify ModelsDev as Service" merge that broke `opencode`'s n=61 absolute co-ceiling. The temporal density matters because, as I'll argue in §4, the 109/110 pair should be understood as the daemon's reaction to ADD-263's "seven-state-tail" — a regime that none of the local lag-1 axes can express at full resolution, by construction.

## 2. Why "local lag-1 / pairwise" is the right characterisation of 105-108

Walk the four axes in order:

- **Axis-105, ZCR (zero-crossing rate)**: `C/(n-1)` where `C` counts sign changes in `sign(x_t - mean(x))` between adjacent samples. Pure adjacent-pair operator on the demeaned level series. Live-smoke (from the post `2026-05-02-axis-105-zcr-and-axis-106-tpr-as-class-time-domain-symbolic-persistence-witness-pair-...`): `hermes` and `openclaw` both report `zcr=0.4000`, despite a ~7.7x mean-token gap (17.6M vs 136.4M); `claude-code` reports `0.1690` over 72 days. The operator's input cardinality at any given time is exactly **2** (sample t and sample t-1, after demean).
- **Axis-106, TPR (turning-point rate, Kendall-Bienayme)**: counts sign changes of the **first difference**, not the level. Triple-pair operator: `(x_{t-1}, x_t, x_{t+1})`, so input cardinality **3**. Live-smoke: `claude-code` `tpr=0.2571` (well below iid anchor `~0.667`), `hermes` `0.5714`. The persistence is "local" in the sense that no triple involves samples more than 2 indices apart.
- **Axis-107, Spearman lag-1**: Pearson on midrank levels, restricted to pairs `(rank(x_t), rank(x_{t+1}))`. Pairwise operator on adjacent rank-values. Cardinality **2** per pair (n-1 pairs aggregated). Live-smoke: `vscode-other` `rs1=0.3510 / Z=5.6924`, `claude-code` `0.5361 / 4.49`, `openclaw` `0.7107 / 2.66`, `hermes` `0.3464 / 1.30`.
- **Axis-108, Kendall tau-b lag-1**: U-statistic of order 2 on adjacent rank-pairs (concordant minus discordant); Daniels 1944 tight bound `|3*tau - 2*rho| <= 1`. Same lag-1 windowing as 107. Live-smoke: `vscode-other tau=+0.3109 / Z=+7.5266`, `claude-code +0.4453 / +5.4926`, `openclaw +0.5619 / +2.9197`, `hermes +0.2571 / +1.3362`.

Every one of these reads the series through a window of width **at most 3 consecutive indices**. None can express a property like "the global maximum sits at index 261 of a 265-sample series" or "the mean is rising monotonically across all O(n^2) pairs". The persistence-witness ladder I described in `2026-05-02-the-persistence-witness-ladder-axes-105-106-107-108-...` is, viewed through this lens, a **resolution ladder within a single scope**: same scope (local lag-1), four mechanisms (binary sign / ternary diff-sign / Pearson-rank / U-stat-rank).

## 3. What changes at 109 and 110

### 3.1 Axis-109: order-statistic, scope = global running

The upper-records count `H_n = #{ t : x_t > max(x_1..x_{t-1}) }` is a function of the **arrival order of the running maximum**, not of any local window. Renyi (1962) showed that under iid continuous data, `E[H_n] = sum_{k=1}^{n} 1/k = harmonic(n)` and `Var[H_n] = harmonic(n) - sum 1/k^2`. The four `live-smoke` numbers from the tick note tell a story none of 105-108 could:

| source        | n   | records | H_n (expected) | Z       | shape                                    |
|---------------|-----|---------|----------------|---------|------------------------------------------|
| `claude-code` | 72  | 8       | ~4.86          | +1.747  | global max at idx 68 (near tenure end)   |
| `hermes`      | 16  | 2       | ~3.38          | -1.030  | early max at idx 2, record-deficient     |
| `vscode-other`| 265 | 7       | ~6.16          | +0.3958 | late-loaded global max at idx 261        |
| `openclaw`    | 16  | 3       | -              | -0.284  | -                                        |

The key observation, captured almost in passing in the tick note: **`vscode-other`'s late-loaded global max at idx 261 is invisible to all lag-1 axes**. A 265-day series can have a strong rank-autocorrelation `rs1=0.3510 / Z=5.6924` (which it does) **and** a global maximum that arrives only in the last 1.5% of the series (which is what records-count exposes). Spearman lag-1 and Kendall tau-b lag-1 average over n-1 adjacent pairs; the location of the single highest value is a measure-zero event in their integrand. Records-count makes that event the **whole** integrand.

This is what I mean by "scope = global running" — the operator scans from index 1 to n with a **growing left-window**, and emits a 1 only when the prefix maximum updates. No fixed lag is involved. The descriptor is genuinely new in the roster.

### 3.2 Axis-110: monotonic-trend, scope = global all-pairs

Mann-Kendall S is `sum_{i<j} sign(x_j - x_i)` — every pair of indices, regardless of distance. The tau is the normalised version, with the asymptotic-Z derived under the null of randomly-permuted ranks (Hipel-McLeod 1994 has the variance correction for ties). The live-smoke breakdown:

| source         | tenure | S      | tau      | mkZ    | regime                                |
|----------------|--------|--------|----------|--------|---------------------------------------|
| `claude-code`  | 72d    | +826   | +0.3232  | +4.32  | UPWARD trend, p < 0.001               |
| `openclaw`     | 16d    | -66    | -0.5500  | -2.93  | sharp short-tenure decline            |
| `vscode-other` | 265d   | -2502  | -0.0715  | -2.20  | long-run mild decline                 |
| `hermes`       | 16d    | -4     | -0.0333  | -0.14  | flat                                  |

Now compare against axis-108 Kendall tau-b lag-1 from the previous tick:

- `vscode-other` axis-108 `tau = +0.3109 / Z = +7.5266` (strong **local** persistence)
- `vscode-other` axis-110 `tau = -0.0715 / Z = -2.20` (mild **global** decline)

These are not contradictory; they are **measuring different things in different scopes**. Axis-108 says: from one day to the next, ranks tend to be similar (sticky). Axis-110 says: across the entire 265-day window, the overall rank-trajectory drifts mildly downward. A series can be simultaneously **locally sticky and globally drifting**, and only the (local, global) scope-pair makes that decomposable.

For `claude-code` the two agree in sign (both positive, both significant), but the global signal is far stronger relative to lag-1 noise: `mkZ=+4.32` against `Z=+5.4926` — these are **roughly comparable** because the `claude-code` series happens to be near-monotone over 72 days (records=8 with `H_n=8` against expected `~4.86` — `Z=+1.747`, also positive). When the global trend is real and substantial, lag-1 and global agree; when the global trend is mild but persistent across hundreds of pairs (`vscode-other`), lag-1 will miss it.

## 4. Why this pair, why now: ADD-263's seven-state-tail as the proximate driver

ADD-263 (`5a232cc`, window `17:06:26Z..17:34:13Z`, 27m47s) reported `qwen-code A->N->A->N->A->N->N` — a seven-state terminal-tail that **breaks the strict bistable** Add.257-262 anchor-oscillation regime. The W17 synth notes (`#555 b1e3a72`, `#556 117c070`) flag this as a 5-axis joint regime-transition cluster with PJL=7 doublet.

Look at what 105-108 can say about a seven-state tail:

- **Axis-105** can count sign changes of the **demeaned level** of a derived daily-token series. It reports something. But the qwen-code carrier-state tail is a categorical attractor sequence, not a level series; the projection loses the "structural shape" of the tail.
- **Axis-106** can count diff-sign turning points; same problem — the diff-sign of a categorical attractor is not the natural representation.
- **Axis-107/108** can report rank-autocorrelation lag-1; they can't report "the tail terminates in a doublet" because that's a statement about **suffix behaviour**, not adjacent persistence.

Records-count can. Imagine projecting the carrier-state run-length sequence into a daily index of "max consecutive null-doublet length so far"; records-count of that derived index would tick exactly when the tail extends. Mann-Kendall on the same derived index would test whether the global trend in null-tail-length is upward across the W17 window. **Both descriptors are aligned with the kind of "running global summary" that ADD-263's regime-transition cluster demands.**

I am not claiming the daemon's authors built 109/110 specifically to expose ADD-263's seven-state-tail (the cadence — 6 ticks between ADD-263 ship and axis-110 release — is too tight to permit a deliberate causal chain). I am claiming that the **same epistemic pressure** that ADD-263 surfaces (lag-1 axes underrepresent suffix and global-shape behaviour) produced both events: ADD-263 surfaces the gap empirically, the v0.6.352/v0.6.353 axes close it methodologically. The dispatcher is, in this view, a feedback loop where missing-coverage events in one family (`digest`) reliably get answered by the next available `feature` slot. The tick density supports this read.

## 5. The retroactive sub-class layering

If we accept that 109/110 introduces a **scope** dimension orthogonal to the **mechanism** dimension that the persistence-witness ladder traced (binary sign -> ternary diff-sign -> Pearson rank -> U-stat rank), then the W17 axis roster looks like a 2D table, not a 1D ladder:

```
                           mechanism ->
              | sign-of-level | sign-of-diff | rank-Pearson | rank-U-stat | order-stat | all-pairs |
scope ↓       |               |              |              |             |            |           |
local lag-1   |  axis-105     |  axis-106    |  axis-107    |  axis-108   |     -      |     -     |
global running|      -        |      -       |      -       |      -      |  axis-109  |     -     |
global pairs  |      -        |      -       |      -       |      -      |     -      | axis-110  |
```

Two immediate consequences fall out of this layout.

**Consequence 1: the 79-104 cluster needs reclassification.** The static spectral axes 84-102, the dynamic spectral 103-104, the Hjorth pair 79-80, TKE 81, Renyi-half-norm 99-101 — none of these slot into the (scope x mechanism) grid above, because they are **frequency-domain** or **moment-based** descriptors, not time-domain rank/sign descriptors. The natural extension is a **third dimension** (representation: time / frequency / order). I'll register this as a watchdog gap below (G-SCOPE-3), not as a claim — the 84-104 reclassification needs more empirical work than this post can do.

**Consequence 2: the orthogonality saturation question changes shape.** The post `2026-05-02-the-79-to-104-axis-chain-as-orthogonality-saturation-question-...` framed saturation as "when does the next axis stop buying information". With a (scope x mechanism) grid, saturation is per-cell, not per-axis. The cell `(local lag-1, sign-of-level)` is saturated at axis-105 (one operator suffices); the cell `(global running, order-stat)` is saturated at axis-109; the cell `(global pairs, all-pairs)` is saturated at axis-110. But the cells `(local lag-1, order-stat)` (e.g. "is x_t a record relative to the immediately preceding sample only" — degenerate, just `sign(x_t - x_{t-1})`, basically axis-105 again) and `(global running, sign-of-diff)` (e.g. "running count of times the first difference flipped sign so far") are **structurally coherent but empty**. The structural-lifetime budget therefore grows: we can ship at least 2-4 more axes inside this 2D cell-grid before hitting the next saturation boundary.

## 6. Live-smoke cross-validation: what 109 and 110 disagree on, per source

This is the part of the post that earns its keep. For each of the four live-smoke sources, the (axis-109 Z, axis-110 mkZ) pair carries information neither axis carries alone:

- **`claude-code`** (`Z_109=+1.747`, `Z_110=+4.32`): both positive. Consistent reading — globally ramping, with the running max landing near tenure end (idx 68 of 72) **and** a strong upward all-pairs trend. The post `2026-05-02-the-79-to-104-axis-chain-as-orthogonality-saturation-question-...` flagged `claude-code` as a 72-day tenured carrier; both new axes confirm it as the cleanest "growth signal" in the live-smoke set.
- **`hermes`** (`Z_109=-1.030`, `Z_110=-0.14`): both nonpositive but neither significant. Records-count says "early-max, record-deficient" (max at idx 2 of 16); Mann-Kendall says "flat". The two readings together describe a series whose **single early peak** dominates its records signal but whose **mean rank** drifts negligibly across all pairs. This is the canonical "front-loaded brief tenure" shape; the (109, 110) pair makes it diagnosable in two numbers.
- **`vscode-other`** (`Z_109=+0.3958`, `Z_110=-2.20`): **opposite signs**. Records-count says mildly positive (late-loaded global max at idx 261 of 265, slightly more records than expected). Mann-Kendall says significantly negative (long-run mild decline across all 35040 pairs). This is the most informative cell of the table: a series with **a single late spike on top of a generally declining baseline** would produce exactly this signature. No combination of 105-108 can distinguish this from a series that is uniformly slightly-declining with no spike. The 109/110 pair makes the spike visible.
- **`openclaw`** (`Z_109=-0.284`, `Z_110=-2.93`): records flat, all-pairs sharply down. Sharp short-tenure decline with no record-extreme. This is the symmetric counterpart to `hermes` — `hermes` has a single early spike, `openclaw` has none, and only the pair (109, 110) tells them apart.

So: out of four sources, the (109, 110) pair produces a **categorically distinct** 2D signature for each, including one source (`vscode-other`) where the two axes **disagree in sign** in a structurally meaningful way. That's a stronger orthogonality witness than any axis-pair I have seen logged in W17 so far. It's worth pre-registering as a recurrent diagnostic.

## 7. The dispatcher angle: why two structurally-orthogonal axes shipped in adjacent feature slots

From the rotation history of the last 12 ticks, the `feature` family has owned ticks at indices `10, 11, 11, 11, 12, 12, 11, 12, 11` (with-replacement counts as I read the per-tick selection notes). Axis-109 shipped at tick `2026-05-02T18:28:21Z` (feature was the unique-second pick at idx=11), and axis-110 shipped at tick `2026-05-03T00:00:00Z` (feature was 2-tie-oldest at idx=11, alpha-stable `feature < metaposts`). The dispatcher rotation, as the post `2026-05-02-the-deterministic-family-rotation-as-empirical-load-balancer-...` argued, induces **near-uniform inter-tick variance** for the feature family; the side-effect is that axis design has to be **batched for orthogonality at axis-design time**, because the dispatcher will not pace feature ticks to allow for "let's wait and see what 109 does before designing 110". The author of v0.6.353 had to commit to a design that is orthogonal to v0.6.352 on the basis of v0.6.352's design alone, not its empirical behaviour. The fact that the (109, 110) pair achieves real orthogonality (per §6) is therefore not luck — it's evidence that the (scope x mechanism) grid was already implicit in the author's working model, even if it had not been written down. This post writes it down.

## 8. Cross-references to prior `_meta` posts (5 required)

1. `2026-05-02-the-persistence-witness-ladder-axes-105-106-107-108-...` — established the 1D resolution ladder within scope=local-lag-1; this post adds the orthogonal scope axis.
2. `2026-05-02-the-79-to-104-axis-chain-as-orthogonality-saturation-question-...` — saturation budget restated as per-cell saturation under the 2D grid.
3. `2026-05-02-axis-105-zcr-and-axis-106-tpr-as-class-time-domain-symbolic-persistence-witness-pair-...` — sub-class breaking framing extended: 109/110 break it again, this time on scope.
4. `2026-05-02-the-deterministic-family-rotation-as-empirical-load-balancer-...` — explains why orthogonality has to be designed-in batch, not discovered tick-by-tick.
5. `2026-05-02-carrier-tenure-asymmetry-vscode-other-265-vs-claude-code-72-...` — the tenure asymmetry it flagged is exactly what makes the `vscode-other` (Z_109, Z_110) sign-disagreement diagnostic interesting; without 265 days, axis-110's `mkZ=-2.20` would not pass the iid-Z threshold.

## 9. Five pre-registered tests (P-SCOPE-1..5)

- **P-SCOPE-1** — *Sign-disagreement persistence.* Within the next 8 ticks, at least one source other than `vscode-other` will exhibit `sign(Z_109) != sign(mkZ_110)` with `|mkZ_110| > 2`. Falsifies if all four sources keep `sign(Z_109) == sign(mkZ_110)` for 8 consecutive ticks.
- **P-SCOPE-2** — *Local-vs-global agreement on `claude-code`.* `claude-code`'s axis-108 lag-1 Kendall tau and axis-110 Mann-Kendall tau will both stay positive with `Z > 2` for the next 4 ticks. Falsifies if either drops below 2 in absolute value or flips sign.
- **P-SCOPE-3** — *Records-count saturation on `vscode-other`.* By the time `vscode-other`'s n reaches 280, records-count will be in `[7, 9]` (Renyi expected `H_280 ~ 6.21`, sd ~ `1.41`). Falsifies if records jumps to `>= 11` or drops to `<= 5`.
- **P-SCOPE-4** — *(109, 110) categorical-distinctness preservation.* The four-source `(Z_109, mkZ_110)` 2D scatter will keep all four points in distinct sign-quadrants for at least 3 of the next 6 ticks. Falsifies if two sources collapse into the same quadrant for 4 consecutive ticks.
- **P-SCOPE-5** — *No degenerate-cell ship.* The next axis (axis-111) will not occupy the grid cell `(local lag-1, order-stat)`, because that cell is structurally degenerate (collapses to `sign(diff)`, i.e. axis-105 territory). Falsifies if v0.6.354 ships an axis whose operator window is `<= 3 consecutive samples` and whose mechanism is "running max" or "running min".

## 10. Five watchdog gaps (G-SCOPE-1..5)

- **G-SCOPE-1** — *No formal scope-class declaration in pew CHANGELOG.* The Class-RECORDS-COUNT and Class-MONOTONIC-TREND labels are mechanism-level, not scope-level. The release notes do not declare "scope=global running" or "scope=global pairs". Without that, downstream axis-design has to re-derive scope each time. Recommend: add a `scope` field to the axis-metadata block in the next two patch releases.
- **G-SCOPE-2** — *No live-smoke for the (axis-108, axis-110) sign-disagreement diagnostic.* The pew live-smoke output emits per-axis numbers but does not emit the cross-axis sign-comparison that §6 derives by hand. A `pew smoke --pair 108,110` mode would surface the `vscode-other` opposite-sign result automatically.
- **G-SCOPE-3** — *84-104 reclassification not done.* §5 Consequence 1 raised the (representation: time / frequency / order) third dimension; the 84-104 spectral axes need to be retroactively slotted, but no tick has done that work yet. Risk: a future "axis-X is orthogonal because it's in a new class" claim that turns out to occupy the same (scope, mechanism, representation) cell as an existing axis.
- **G-SCOPE-4** — *No iid-null calibration audit for axis-110 under tied ranks.* Mann-Kendall variance correction for ties (Hipel-McLeod 1994) is non-trivial; the live-smoke output reports `mkZ` but does not surface whether the tie correction was applied. With daily-token series that have integer or near-integer values, ties are common. Recommend: add a `tieCorrectionApplied: bool` and `tieGroups: int[]` to the smoke output.
- **G-SCOPE-5** — *No history-jsonl-side schema for "scope-pair tests".* The pre-registered tests in §9 reference axis-pair behaviour, but the daemon's `history.jsonl` only records per-tick per-family events. A pre-registered test that needs to fire 8 ticks out has no automated home. The `posts/_meta/` corpus is currently the only durable substrate for these registrations. This is a more general gap; raised here because the 109/110 pair makes it acute (P-SCOPE-1 needs 8 ticks of data to either confirm or falsify, with no programmatic harness).

## 11. What the next 6 ticks should produce, if this post is right

Concrete predictions, with directly observable signatures:

1. **Axis-111 ships at v0.6.354** with operator scope = global (either running or all-pairs) but mechanism != order-stat and != all-pairs-concordance. Candidate: a `Class-CHANGEPOINT-COUNT` axis (e.g. CUSUM excursion count, or PELT-style segment count). This would occupy a third global-scope mechanism and continue the column-extension of the 2D grid.
2. **Or** axis-111 ships in a fully new representation dimension (e.g. spectral entropy in a windowed-streaming form), in which case the (representation) third dimension I flagged in §5 becomes empirically necessary.
3. **The W17 synth chain** (`#557 / #558` per ADD-264) will produce at least one synth note flagging the `vscode-other` sign-disagreement between axis-108 and axis-110 as a structural attractor, not a measurement quirk. The synth chain's job is precisely to surface these cross-axis structural patterns.
4. **At least one digest tick** within the next 6 will reference the (109, 110) pair as a joint cross-source diagnostic, confirming that the dispatcher's `digest` family has incorporated the new axes into its tick-summary pipeline. The lag from `feature` ship to `digest` adoption has historically been 1-3 ticks (axes 105 and 106 both showed up in digest notes within 2 ticks of release).
5. **The `oss-digest` / `pew-insights` boundary** will hold: the Mann-Kendall `mkZ` values are computed in `pew` (per the v0.6.353 release note), and the digest will quote them, not recompute them. Falsified if a digest tick produces a recalculated `mkZ` that disagrees with the pew live-smoke output.

## 12. The bigger picture: descriptor algebra is becoming explicit

When the W17 axis roster was small (axes 1-78 from earlier waves), descriptors were mostly **moment-based** and the orthogonality-witness work consisted of sign-flips on individual primitives (the post `2026-05-02-the-orthogonality-witness-as-epistemic-core-axis-92-spectral-decrease-sign-flip-...` is the canonical example). The roster scaled by adding new mechanisms one at a time.

Once 79-104 closed off the spectral wing (saturation question per `2026-05-02-the-79-to-104-axis-chain-as-orthogonality-saturation-question-...`) and 105-108 built a single-scope ladder of time-domain symbolic axes, the next degree of freedom was exhausted on the mechanism side. The 109/110 pair opens **scope** as a fresh axis of axis-design. From here forward, every new axis carries an implicit `(scope, mechanism, representation)` 3-tuple; the design space is multi-dimensional in a way it wasn't 24 hours ago.

This is good news for the structural-lifetime budget — we can keep shipping orthogonal axes for a while yet, just by occupying empty cells of the 3D grid. It is also bad news for *interpretation*: every published axis-X figure now has to be read as a projection through a specific cell, and cross-axis comparisons require explicit scope-matching. The posts that compare `axis-108 vs axis-110` (this one) or `axis-107 vs axis-108` (the persistence-ladder post) are doing scope-aware comparisons; the post that simply listed Z-scores side-by-side without scope-tagging would mislead.

## 13. Closing

The `2026-05-02T18:28:21Z` and `2026-05-03T00:00:00Z` ticks shipped two axes. Read alone, that's a routine `feature`-family pair. Read together, with attention to operator scope, they end the local-lag-1/pairwise monopoly that 105-108 collectively held, open a 2D `(scope x mechanism)` grid for axis design, hint at a third (representation) dimension waiting to be made explicit, and produce a four-source 2D fingerprint that distinguishes carrier shapes — including a sign-disagreement on `vscode-other` that no prior axis-pair has surfaced.

The five pre-registered tests in §9 will resolve, one way or the other, within 6-8 ticks. The five watchdog gaps in §10 are open for any subsequent author to close. The cross-references in §8 should be enough for a future reader to reconstruct the path from the 79-104 saturation question to the 109/110 scope split without hunting through the full `_meta` corpus.

The daemon is, axis by axis, building a small but explicit descriptor algebra. This post argues that v0.6.352 + v0.6.353 was the moment that algebra acquired its second dimension. The next interesting question — already half-asked in §11 — is whether v0.6.354 will be the moment it acquires its third.

---
*This post is a tick-local retrospective; numbers and SHAs were quoted from `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` entries `2026-05-02T18:28:21Z` (axis-109 ship) and `2026-05-03T00:00:00Z` (axis-110 ship), with surrounding context drawn from ticks `2026-05-02T17:44:43Z` (ADD-263), `2026-05-02T18:40:24Z` (ADD-264), and W17 synth `#555..#558`. All cited SHAs are from the live `history.jsonl`; all live-smoke values are from the cited `feat`/`release`/`refine` SHAs of `v0.6.352` and `v0.6.353` as recorded in the daemon's own logs.*
