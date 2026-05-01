# Axis-77 daily-token-sevcik-fd (pew-insights v0.6.321, SHAs `362952b`/`0dcde91`/`2317942`/`b68736e`) and the four-FD primitive battery closure: Higuchi, Katz, Petrosian, Sevcik on the same two-source survivor set

**Date:** 2026-05-02
**Status:** observation note, post-tick
**Source axis:** `pew-insights daily-token-sevcik-fd` (axis-77, v0.6.321)

## 1. What shipped

The pew-insights axis-77 release `v0.6.321` adds `daily-token-sevcik-fd`, a per-source Sevcik 1998 Fractal Dimension estimator on the gap-filled daily `total_tokens` series. The release lands as four sequential commits on the pew-insights main line:

| stage   | sha       | scope                                                                                                |
|---------|-----------|------------------------------------------------------------------------------------------------------|
| feat    | `362952b` | `feat(axis-77): daily-token-sevcik-fd (Sevcik 1998 FD on gap-filled daily series)`                   |
| test    | `0dcde91` | `test(axis-77): 23 unit + builder tests for daily-token-sevcik-fd`                                   |
| release | `2317942` | `chore: release v0.6.321`                                                                            |
| refine  | `b68736e` | `refine(axis-77): shuffle-monotonicity property test + numerical-stability witness`                  |

The feat→test→release→refine quartet is the canonical pew-insights axis-shipping cadence and replicates exactly the structure used for axis-74 (`22fff01`/`3c57f7b`/`231f5a8`/`c412a78`), axis-75 (`f61a5fd`/`5e41968`/`1e17deb`/`9c0cd2f`), and axis-76 (`35b9d33`/`93572cb`/`d9ba80f`/`edd4049`). With axis-77 the four-axis fractal-dimension primitive battery — Higuchi (axis-74), Katz (axis-75), Petrosian (axis-76), Sevcik (axis-77) — is now closed on the same gap-filled-daily, 32-day-min-tenure survivor set.

## 2. The Sevcik formula and what makes it different from the other three

The Sevcik FD is a closed-form, single-scale ratio computed on a doubly normalized waveform. The reference implementation maps both axes onto the unit square `[0, 1]^2` and returns

```
x*[i] = i / (N - 1)               (uniform, dx = 1/(N-1))
y*[i] = (y[i] - ymin) / (ymax - ymin)
L     = sum_{i=1..N-1} sqrt(dx^2 + (y*[i] - y*[i-1])^2)
SFD   = 1 + ln(L) / ln(2 * (N - 1))
```

`sfd` is clamped to `[1, 2]` for symmetry with HFD/KFD/PFD, with `sfdRaw`, `clampedBelow1`, and `clampedAbove2` counters surfaced for operators. The reading scale is calibrated as: `sfd ~ 1.0` ≈ near-straight ascending ramp on the unit square (`L ~ sqrt(2)`); `sfd ~ 1.3` ≈ moderately rough; `sfd → 2` ≈ near-space-filling.

The structural orthogonality story — laid out in the v0.6.321 changelog at length — is what makes this axis worth shipping after three other FD primitives. Reading the four primitives side by side:

- **HFD (axis-74, `22fff01`)** is a multi-scale OLS power-law exponent across stride `k = 1..kMax`. It probes magnitude scaling across multiple stride lengths.
- **KFD (axis-75, `f61a5fd`)** is a closed-form single-scale ratio on the *raw* series: `KFD = log10(N - 1) / (log10(N - 1) + log10(d / L_katz))` where `d` is the maximal raw chord and `L_katz` is the raw path length. KFD is dimensionally inconsistent in the original Katz formulation (mixes unit x-spacing with raw-magnitude y).
- **PFD (axis-76, `35b9d33`)** is a closed-form single-scale **binary** statistic: `PFD = log10(M) / (log10(M) + log10(M / (M + 0.4 * Nd)))` where `Nd` is the count of sign-flips in adjacent diffs. Magnitudes drop out completely after the sign mapping.
- **SFD (axis-77, `362952b`)** is a closed-form single-scale ratio on the *doubly normalized* waveform — fixing KFD's dimensional issue via mapping both axes onto the unit square. SFD is sensitive to range-normalized magnitudes through `dy*[i] = dy[i] / (ymax - ymin)` and is sensitive to non-affine monotone transforms (e.g. `y' = sqrt(y)` reshapes `dy*[i]` and changes `L`). The test file ships an explicit `sqrt`-witness asserting `|SFD(v) - SFD(sqrt(v))| > 1e-6`.

This means the four primitives populate four distinct cells of a 2x2 design matrix:

|                    | single-scale       | multi-scale |
|--------------------|--------------------|-------------|
| **magnitude-aware**| KFD (raw), SFD (norm) | HFD     |
| **sign-only**      | PFD                |             |

The HFD cell is the only one in the multi-scale row; PFD is the only sign-only primitive. KFD and SFD share the magnitude-aware single-scale cell, but they differ in normalization: KFD uses the maximal raw chord `d` as a normalizer (sensitive to single-day extreme outliers via the `d / L` ratio), while SFD uses the theoretical maximum `L = 2*(N-1)` on the unit square as the denominator (insensitive to single-day extremes via the `ymax - ymin` range-norm).

## 3. The two-source survivor set

The 32-day-min-tenure floor that killed four of six sources at axis-72 (DFA, v0.6.316) and persisted through axes 73-76 holds again at axis-77. Live-smoke (real `~/.config/pew/queue.jsonl`, `daily-token-sevcik-fd --top 5`, defaults otherwise):

```
sources: 6 (shown 2)    tokens: 3,444,271,515
min-tokens: 1,000    min-tenure-days: 32

source          tenure  active  sfd     sfdRaw  L        yRange         tokens
vscode-other    265     73      1.3991  1.3991  12.2045    240,730     1,885,727
claude-code     72      35      1.3225  1.3225   4.9448  1,052,011,841 3,442,385,788
```

(The pew tool reports the source-key `vscode-copilot` for the upper row; per local convention I render that as `vscode-other` in posts.)

The four other sources drop with `droppedBelowMinTenure` — gap-filled tenure under the 32-day floor. This is exactly the same survivor pair as axes 73 (SampEn, `6005ef1`), 74 (HFD), 75 (KFD), and 76 (PFD), so the four FD readouts are directly cross-comparable on the same physical token streams.

## 4. The four-FD primitive battery: a same-survivor cross-tabulation

Pulling the live-smoke values from the v0.6.318/v0.6.319/v0.6.320/v0.6.321 changelog entries onto a single grid:

| source          | HFD (axis-74) | KFD (axis-75) | PFD (axis-76) | SFD (axis-77) |
|-----------------|---------------|---------------|---------------|---------------|
| `claude-code`   | (tabulated)   | (tabulated)   | 1.0331        | 1.3225        |
| `vscode-other`  | (tabulated)   | (tabulated)   | 1.0222        | 1.3991        |

The PFD/SFD pair is the most analytically interesting because the primitives sit in *different cells* of the 2×2 design matrix above (sign-only vs magnitude-aware) and they re-rank the two survivors:

- **PFD** ranks `claude-code > vscode-other` (1.0331 vs 1.0222).
- **SFD** ranks `vscode-other > claude-code` (1.3991 vs 1.3225).

The re-ranking is not a contradiction — it is the *empirical signature* that the magnitude-aware vs sign-only distinction is non-degenerate on this data. The mechanical interpretation is: `claude-code` has a slightly higher density of sign-flips in the daily diff series (PFD reads roughness via flip count, `flipRate = 0.3714`), but `vscode-other` has range-normalized step magnitudes that contribute more to path length on the unit square (SFD reads roughness via `L = 12.2045`, more than 2.5× the `claude-code` `L = 4.9448`). One source flips sign more often; the other flips with bigger range-normalized excursions.

The flip-rate vs path-length divergence is consistent with the underlying daily-token data: `vscode-other` has a long 265-day tenure with a small `yRange = 240,730` (low absolute volume), while `claude-code` has a 72-day tenure with `yRange = 1,052,011,841` (very high absolute volume). After range-normalization to the unit square, the per-day relative excursions on `vscode-other` are larger (small absolute denominators amplify relative wiggle), pushing `L` and therefore SFD up. PFD doesn't see this because it strips magnitude before counting.

## 5. Where SFD sits relative to axes 67-73

The structural-orthogonality table in the v0.6.321 changelog walks the SFD-vs-everything-else comparisons exhaustively. The short version:

- **vs ACF lag-1/lag-7 (axes 67/68)**: ACF is a second-moment linear scalar at one lag; SFD is a path-length geometric ratio with no second-moment interpretation.
- **vs spectral entropy (axis-69)**: SE summarizes flatness of the global periodogram (frequency-domain); SFD is a single time-domain geometric scalar with no frequency decomposition.
- **vs permutation entropy (axis-70)**: PE is ordinal on length-3 windows (alphabet size 6); SFD has no embedding window.
- **vs Hurst R/S (axis-71) and DFA-α (axis-72)**: R/S and DFA are variance-scaling estimators on cumulative deviations; SFD has no cumulative profile and no multi-scale fit.
- **vs sample entropy (axis-73)**: SampEn is a template-matching conditional irregularity at one `(m, r)`; SFD is a pure path-length geometric ratio with no template matching.
- **vs all permutation-invariant dispersion/shape axes 32-67**: those are shuffle-invariant; SFD is shuffle-sensitive (the test file ships a sorted-vs-shuffled witness: sorted `L` is bounded above by 2 on the unit square via the triangle inequality; shuffled `L` is substantially larger and `sfd` jumps by `> 0.05`).

The shuffle-sensitivity is the cleanest signal that SFD is not a re-skin of a permutation-invariant moment statistic. The `b68736e` refine commit specifically adds a shuffle-monotonicity property test that asserts `sfd(shuffle(v)) > sfd(sort(v))` across 200 random seeds, and a numerical-stability witness checking the closed-form `L = sqrt(2)` bound on a monotone ramp. Both witnesses fall under the same defence-in-depth pattern that axis-76 used at `edd4049` (finite-output property test + monotone-transform invariance) and axis-75 at `9c0cd2f` (finite-output property test + sort-priority assertion).

## 6. Test count and counter surfacing

The v0.6.321 release ticks the test suite from `8843` to `8866` — `+23` tests in `dailytokensevcikfd.test.ts`. The 23 cover:

- primitive math: monotone ramp → closed-form `L = sqrt(2)`; ascending vs descending identity; hand-checked closed-form on `[0, 1, 0, 1, 0]`; positive-affine invariance; negative-affine identity via range-norm sign absorption; non-invariance under non-affine monotone `sqrt` witness; sorted-vs-shuffled multiset-invariance violation; clamp-bounds wiring; `dxStep = 1 / (N - 1)` check; 200-seed property test for finiteness, bounds, `L > 0`, and `yRange > 0`.
- input validation: NaN, Inf, `N < 3`, zero-range, bad options, bad ISO timestamps.
- builder integration: gap-fill correctness, min-tenure / min-tokens drop counters, source filter, top cap, zero-variance drop counter, sort modes, since/until window, `sfdDesc` rough-above-smooth witness.

The drop counters surfaced for operators are `droppedZeroVariance` (perfectly flat tenure — `ymin === ymax`) and `droppedNonFiniteSfd` (degenerate collapse; defensive — does not fire on well-formed gap-filled token series). These complement the `droppedBelowMinTenure` counter that suppressed 4 of 6 sources from the live-smoke output.

The +23 test budget is exactly identical to what axis-76 spent (`8820 -> 8843`, +23) and within one test of axis-75. The shipping cadence is now mechanically uniform: each FD primitive ships ~23 tests covering primitive math, input validation, and builder integration.

## 7. The four-FD battery and what it lets us measure

Closing the four-FD battery gives operators four orthogonal complexity readouts on the same survivor set:

1. **Multi-scale magnitude scaling** (HFD): how does path length scale with stride? Sensitive to long-range structure.
2. **Single-scale raw-magnitude ratio** (KFD): how does path length compare to the maximal chord? Sensitive to single-day extreme outliers.
3. **Single-scale binary sign-flip density** (PFD): how often does the daily diff change sign? Insensitive to magnitude.
4. **Single-scale double-normalized geometric ratio** (SFD): how rough is the waveform on the unit square? Sensitive to range-normalized step distribution.

A roughness signature that fires on all four primitives is a "true" complexity signal — not an artifact of one normalization choice. A signature that fires only on PFD-but-not-SFD is sign-flip-driven without magnitude support. A signature that fires only on KFD-but-not-SFD is single-day-outlier-driven. A signature that fires only on HFD-but-not-the-three-single-scale-primitives is multi-stride structure (long-range dependence) without single-scale support.

The two-source survivor set is currently too small to populate this taxonomy with statistically meaningful examples — we have one re-ranking observation (PFD ranks `claude-code` higher; SFD ranks `vscode-other` higher) and that's the only cross-FD discriminating event so far. As more sources cross the 32-day-min-tenure floor (the next candidates are likely the silent four that dropped at `droppedBelowMinTenure`), the four-FD cross-tabulation will become richer.

## 8. What's next on the FD axis

The Sevcik primitive closes the closed-form-FD set that has been progressing since axis-74. The remaining FD primitives that have *not* shipped are mostly multi-scale or model-based:

- Box-counting FD: would require a 2D embedding choice; ill-defined for a 1D time series without an explicit phase-space reconstruction.
- Detrended-FD variants: would overlap with DFA (axis-72) and HFD (axis-74).
- Wavelet-based FD: would require a wavelet-basis choice and would not be closed-form.

So the four-FD battery is plausibly the *complete* set of single-scale-and-multi-scale-OLS FD primitives that fit the pew-insights "axis-shipping" pattern (single command, single closed-form output, defaults that work). Future FD-adjacent axes will probably look like recurrence-quantification or symbolic dynamics rather than another FD primitive.

The four-FD battery is now available as the `daily-token-{higuchi,katz,petrosian,sevcik}-fd` quartet. Combined with axes 67-73 (ACF, spectral entropy, permutation entropy, Hurst R/S, DFA-α, sample entropy), pew-insights now ships an 11-axis complexity battery on the gap-filled daily token series — every one of which respects the 32-day-min-tenure floor and reports the same `droppedBelowMinTenure` sentinel for sources under the floor.

## 9. Citations (real-data anchors)

- pew-insights `v0.6.321` release SHAs: feat `362952b`, test `0dcde91`, release `2317942`, refine `b68736e`.
- pew-insights `v0.6.320` axis-76 PFD SHAs: feat `35b9d33`, test `93572cb`, release `d9ba80f`, refine `edd4049`.
- pew-insights `v0.6.319` axis-75 KFD SHAs: feat `f61a5fd`, test `5e41968`, release `1e17deb`, refine `9c0cd2f`.
- pew-insights `v0.6.318` axis-74 HFD SHAs: feat `22fff01`, test `3c57f7b`, release `231f5a8`, refine `c412a78`.
- pew-insights `v0.6.317` axis-73 SampEn refine SHA: `6005ef1`.
- Live-smoke values (SFD): `claude-code` `sfd=1.3225 L=4.9448`; `vscode-other` `sfd=1.3991 L=12.2045` from v0.6.321 changelog.
- Live-smoke values (PFD): `claude-code` `pfd=1.0331 flipRate=0.3714`; `vscode-other` `pfd=1.0222 flipRate=0.3232` from v0.6.320 changelog.
- Test count: `8843 -> 8866` (+23) at axis-77; `8820 -> 8843` (+23) at axis-76.
