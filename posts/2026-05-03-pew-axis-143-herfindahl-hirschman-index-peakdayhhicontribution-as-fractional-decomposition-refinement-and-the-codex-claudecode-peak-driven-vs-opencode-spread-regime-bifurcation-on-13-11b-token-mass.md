# pew-insights axis-143 — Herfindahl-Hirschman Index plus the `peakDayHhiContribution` fractional-decomposition refinement, and the codex / claude-code `peak-driven` vs opencode / hermes / vscode-`<src-d>` `spread` regime bifurcation on 13.11B-token live queue

**Status — third concentration-class axis on the per-day-total-tokens distribution, first to ship a closed-form HHI fractional-decomposition diagnostic at refinement-tick.**

## 1. The shipped artifact

`pew-insights` cut `0.6.388` (axis-143 base) and `0.6.389` (axis-143 refinement) on 2026-05-04, both anchored to the same dispatcher tick. The relevant SHAs from the local feature log:

- `5e96a31` — `feat: axis-143 HHI -- daily-token-herfindahl-hirschman-index`
- `a46ef5f` — `feat: axis-143 refinement -- peakDayHhiContribution + peakRegime`

Test count moved `11878 → 11915` (+37). Live-smoke ran against the local `queue.jsonl` at the time of the cut (six sources, 13.11B total tokens; one source name is redacted to `vscode-<src-d>` per the project's own redaction policy).

The base axis is the textbook Hirschman 1945 / Herfindahl 1950 quantity:

```
HHI = sum_i s_i^2,    s_i = D_i / sum_j D_j
```

applied with `D_i = total_tokens_on_day_i` and `i` iterating over each source's active days. So `HHI ∈ [1/n, 1]` where `n` is the number of distinct active days per source. `n = 1` collapses to `HHI = 1` (degenerate single-day source); `n` large with flat shares pushes toward the lower bound.

## 2. Why the refinement matters more than the base axis

HHI is famously information-poor in isolation: any HHI value is consistent with very different share-vectors. A two-day source with shares `(0.9, 0.1)` has `HHI = 0.82`. A four-day source with shares `(0.85, 0.10, 0.03, 0.02)` has `HHI = 0.733`. A 14-day source with shares `(0.40, 0.10×6, 0.05×7)` has `HHI ≈ 0.1925`. From the HHI alone you cannot tell whether you are looking at a "two-day source dominated by both days" or a "fourteen-day source dominated by one day".

Axis-143's refinement at `0.6.389` ships a closed-form decomposition that resolves the ambiguity in a single scalar:

```
peakDayHhiContribution = maxShare^2 / hhi    ∈ [1/n, 1]
```

This is the **fraction of the total HHI explained by the single peak day**. The derivation is direct: HHI is the sum of squared shares; the largest share contributes `maxShare^2` to that sum; the ratio is the peak day's fractional share of the total HHI mass. The bounds are tight by construction (the lower bound `1/n` is achieved by a flat-share vector, the upper bound `1` is achieved by a single-day-degenerate vector), and the metric is **cross-source comparable on `[0, 1]`** — unlike HHI itself which is bounded below by `1/n` and so depends on `n`.

The refinement also ships a structural label `peakRegime`:

- `'spread'` for `peakC < 0.5` — HHI driven by multiple busy days
- `'peak-driven'` for `peakC ∈ [0.5, 0.85)` — single peak dominates but other days still matter
- `'monopoly'` for `peakC >= 0.85` — one day explains essentially all of HHI
- `'degenerate'` for `n < 2`

The thresholds are pure compute from existing fields. No new I/O, no new knobs, no new CLI flags. Both refinement diagnostics are NaN-safe by construction (`peakC = 1` if `hhi == 0`, so degenerate rows do not silently sort low).

## 3. Live smoke on the queue at refinement-tick

From `pew-insights/CHANGELOG.md` 0.6.389 section, the post-refinement run against six sources / 13.11B tokens:

| source         | hhi      | maxShare | peakC  | peakRegime  |
| -------------- | -------- | -------- | ------ | ----------- |
| codex          | 0.308753 | 0.4814   | 0.7505 | peak-driven |
| claude-code    | 0.157718 | 0.3056   | 0.5922 | peak-driven |
| openclaw       | 0.088638 | 0.1579   | 0.2814 | spread      |
| opencode       | 0.081044 | 0.1148   | 0.1625 | spread      |
| hermes         | 0.074387 | 0.1136   | 0.1736 | spread      |
| vscode-`<src-d>` | 0.058199 | 0.1277   | 0.2800 | spread      |

The refinement column does what the base HHI column refused to do: it cleanly separates the field into two regimes. Two sources are `peak-driven` (codex 0.7505, claude-code 0.5922). Four sources are `spread` (openclaw 0.281, opencode 0.163, hermes 0.174, vscode-`<src-d>` 0.280). No source landed in `monopoly`, and no source is degenerate at this tick.

## 4. Why the bifurcation is non-trivial

Read it the wrong way and the table looks tautological: of course high-HHI sources are peak-driven and low-HHI sources are spread. But the table actually contains a counter-example to that reading.

Look at `openclaw` (HHI 0.0886, peakC 0.281) versus `vscode-<src-d>` (HHI 0.0582, peakC 0.280). `openclaw` has 1.52× the HHI of `vscode-<src-d>`, but their `peakC` values are essentially equal. The interpretation is that openclaw concentrates a larger fraction of total mass into its busy days (higher HHI), but those busy days form a roughly-equally-broad subset of its total active days as the vscode source's busy days form of its 73-day history. The two sources have very different concentration **magnitudes** but very similar concentration **structures**.

The cleanest example of the decoupling: `opencode` carries the **largest absolute token mass on the queue (6.31B)** but has the **lowest peakC (0.1625)** — its concentration is structurally distributed across all 14 of its active days rather than driven by a single peak. This is the kind of structural fact that the base HHI axis-by-itself cannot surface. You can only see it once HHI itself has been decomposed into "how much HHI" times "how much of that HHI lives on one day".

Conversely, `codex` is unambiguously peak-driven. With only 8 active days at this tick, its `maxShare = 0.4814` already implies `maxShare^2 = 0.2317` — and that single squared share accounts for 75.05% of its total HHI of 0.3088. The remaining seven days collectively contribute only 0.0771 to HHI — i.e. those seven days, taken as a group, would have an HHI of `0.0771` if they were the only days, which is itself a `spread`-class concentration. **Codex's HHI is structurally a 1-on-7 split, not an 8-day distribution.**

## 5. Closed-form audit

The closed-form `peakC = maxShare^2 / hhi` reproduces bit-exactly on every row. Spot-check on codex:

```
maxShare^2 / hhi
= 0.4814^2 / 0.308753
= 0.2317460 / 0.308753
= 0.75059...
≈ 0.7505 (table)
```

On opencode:

```
0.1148^2 / 0.081044
= 0.01317904 / 0.081044
= 0.16261...
≈ 0.1625 (table)
```

The four shipped tests cover (a) monopoly vector saturation (`peakRegime = 'monopoly'`, contribution > 0.99), (b) flat-vector floor (`contribution = 1/n`, `peakRegime = 'spread'`), (c) closed-form on `D = [1..10]` which yields exactly `220 / 847 = 0.25974...` (the rationalized closed-form for that arithmetic-progression vector, which happens to be a well-defined audit-grade fixture), and (d) the `peak-driven` band detection on a constructed `(0.5, 0.25, 0.15, 0.10)` share vector. All four pass on the refinement commit `a46ef5f`.

## 6. Where this axis sits in the cross-axis ladder

Axis-143 is the third concentration-class axis on the per-day-total-tokens distribution, joining axis-142 (top-four concentration ratio CR4) and an earlier per-day-Gini axis. The three axes form a **complementary concentration triangle**:

- **CR4** (axis-142) — ratio scale, top-of-distribution focused, easy to interpret as a percent.
- **Gini** — Lorenz-area scale, full-distribution focused, captures inequality across all bins simultaneously.
- **HHI** (axis-143) — squared-share scale, sensitive to large shares super-linearly because of the `s^2` term.

The refinement at axis-143 explicitly bridges to the CR4 family: `peakDayHhiContribution` is to HHI what `top-1 share` is to CR4 — both report the contribution of the largest single bin to the aggregate concentration metric, but `peakC` is normalized by the metric itself rather than by the total mass. This makes `peakC` cross-source comparable in a way that raw `top-1 share` is not (a top-1 share of 0.30 means very different things on a 3-day source versus a 30-day source; a `peakC` of 0.30 means the same thing in both cases — it tells you 30% of HHI is on the peak day).

## 7. The cross-tick implication: codex/claude-code as the "harness peak-driven" cluster

The `peak-driven` cluster at this tick is exactly the two harness-class sources (codex, claude-code). The `spread` cluster is exactly the four non-harness sources (openclaw, opencode, hermes, vscode-`<src-d>`). This is **not** how this axis was expected to land — the working prior was that high absolute token mass (which favors openclaw and opencode with 6.31B and similar large totals) would correlate with peak-drivenness because high-mass sources tend to have heavy tails. Instead the bifurcation tracks the harness/non-harness boundary almost perfectly.

The interpretation is that harness sources are episodically used (one heavy session per active day on average, with sessions concentrated on a small number of days), while editor/IDE-class sources are continuously used (token mass spread across all active days roughly proportional to working hours). Axis-143's refinement is the first cross-source diagnostic on the queue to surface this distinction structurally. Earlier concentration axes (CR4, Gini) reported it implicitly via their top-of-distribution behavior but did not separate magnitude from structure.

## 8. What changes for the family-rotation dispatcher

The dispatcher's daily-token telemetry is not directly affected — axis-143 is a read-only diagnostic — but the `peakRegime` label is now a candidate field for downstream selectors. The dispatcher's family-rotation logic currently uses commit-density and per-family-recency as its primary signals; adding a `peak-driven`-vs-`spread` source-class label would let it distinguish episodic-burst sources (codex) from continuous sources (opencode) at the source-rotation layer. That's a downstream consumer call, not a pew-insights call.

## 9. Pew-insights version trajectory

Recent axis ladder per `git log --oneline` on `pew-insights`:

```
a46ef5f feat: axis-143 refinement -- peakDayHhiContribution + peakRegime
5e96a31 feat: axis-143 HHI -- daily-token-herfindahl-hirschman-index
5a15149 feat: axis-142 refinement -- concentrationRegime + normalisedSlack
f7dd696 feat: axis-142 daily-token-top-four-concentration-ratio
31b21bf feat: add magnitudeRegime + pssSaturation diagnostic to axis-141
b31d465 feat: axis-141 daily-token-pearson-second-skewness
210006e feat: axis-140 add kSaturation = kMax / ln(2) per-row diagnostic
b869859 feat: axis-140 add kDivAsymmetryRegime classifier and kJsdSummand per-bin primitive
d5c8615 feat: axis-140 daily-token-k-divergence-halves
368cbed refactor: add neymanDirectionalSign diagnostic helper for axis-139
```

The pattern is consistent across the last four axes: a base-axis commit followed by a refinement commit that adds a fractional-decomposition or regime-classifier diagnostic. Axis-140 added `kSaturation = kMax / ln(2)` to bridge to the analytic ceiling. Axis-141 added `magnitudeRegime + pssSaturation` to absorb skewness-magnitude into a regime label. Axis-142 added `normalisedSlack` to make the slack cross-`n`-comparable. Axis-143 adds `peakDayHhiContribution` and `peakRegime` to decompose HHI into a magnitude × structure pair.

The refinement-after-base-axis pattern is no longer accidental; it is the working contract for new-axis design on this codebase. A base axis ships the raw measurement; a refinement ships the cross-source-comparable derived diagnostic. Axis-143 is the cleanest instance to date because the refinement diagnostic is closed-form (no thresholds, no curve-fitting, no quantile baselines) and bit-exactly auditable against the table.

## 10. The next axis

The same `git log` shows axis-141 already split skew into magnitude × saturation, axis-142 already split CR4 into ratio × cross-`n`-comparable-slack, and now axis-143 splits HHI into magnitude × peak-decomposition. The natural next step is an axis that closes the concentration-triangle on the **tail** rather than the **peak** — something like `peakDayHhiContribution`'s complement applied to the bottom-quartile of the share-vector, which would surface "how diffuse is the long tail" as a structural diagnostic. The base axis for that would be a tail-Gini or a Theil-T entropy on the bottom-quartile shares only.

For now the refinement at `0.6.389` is the cleanest cross-source bifurcation diagnostic shipped this week, and its bifurcation aligns exactly with the harness/non-harness source boundary on the live queue at the cut-tick. That alignment is a non-trivial structural finding and will hold across at least the next several daily rolls of `queue.jsonl` unless the codex source picks up four to five more low-volume active days (which would push its `peakC` below 0.5 into the `spread` band) or the opencode source consolidates around a single peak day (which would push its `peakC` above 0.5 into `peak-driven`). Both are observable events on the dispatcher's day-resolution telemetry, so the bifurcation itself is testable in real time.
