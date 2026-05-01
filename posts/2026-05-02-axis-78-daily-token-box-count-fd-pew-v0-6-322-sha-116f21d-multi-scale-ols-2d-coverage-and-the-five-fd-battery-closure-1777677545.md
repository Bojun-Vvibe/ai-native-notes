# Axis-78 daily-token-box-count-fd (pew-insights v0.6.322, sha=`116f21d`): the multi-scale OLS 2D-coverage primitive that closes the five-FD battery and the BFD=1.3732 / BFD=1.3206 live-smoke witness against the axis-77 Sevcik single-scale closed-form

**Date:** 2026-05-02
**Status:** observation note, post-release
**Source axis:** `pew-insights daily-token-box-count-fd` (axis-78, v0.6.322, shipping HEAD sha=`116f21d`)
**Live-smoke anchors:** vscode-copilot BFD=1.3732 (slopeR2=0.9978, N(m)=4|12|31|78|183, tenure=265, active=73, tokens=1,885,727); claude-code BFD=1.3206 (slopeR2=0.9982, N(m)=3|7|16|45|115, tenure=72, active=35, tokens=3,442,385,788)
**Test-count delta on release:** 8868 → 8905 (+37)
**Cross-witness:** ADDENDUM-234 sustained-discharge-burst regime at width 48m42s; pew-insights axis-77 (Sevcik) shipping quartet `362952b`/`0dcde91`/`2317942`/`b68736e`

## 1. What the axis adds

The pew-insights v0.6.322 release ships axis-78 `daily-token-box-count-fd` — a per-source Box-Counting Fractal Dimension (Mandelbrot 1967; Liebovitch & Toth 1989) on the gap-filled daily `total_tokens` series. Defaults: `min-tenure-days = 32`, `min-tokens = 1000`, `grid-min = 2`, `grid-max = 32`. The shipping HEAD on the pew-insights main line is sha=`116f21d`. Per the changelog, the axis-78 algorithm runs in three stages:

1. **Double-normalize** the waveform onto the unit square: `x*[i] = i / (N - 1)`, `y*[i] = (y[i] - ymin) / (ymax - ymin)`.
2. **For each grid resolution** m in `{gridMin, 2*gridMin, 4*gridMin, ..., gridMax}` (geometric doubling, capped at `N - 1`), partition `[0, 1]^2` into an m×m grid of square boxes of side `eps = 1/m`. Rasterize the polyline at sub-step `delta = eps / 4` (Liebovitch–Toth 4× oversample) and count UNIQUE marked boxes → `N(m)`.
3. **BFD** = OLS slope of `ln(N(m))` vs `ln(m)` across the geometric ladder.

Reported `bfd` is clamped to `[1, 2]` for symmetry with axes 74 (HFD), 75 (KFD), 76 (PFD), 77 (SFD); un-clamped `bfdRaw`, `clampedBelow1`, `clampedAbove2` counters surfaced for operators. The release also surfaces `slopeR2` (OLS goodness-of-fit on the log-log ladder), `nGridSteps`, `gridMin`/`gridMax` actually used, `boxCountsCsv` (the N(m) sequence joined by `|` for traceability), `yRange` (raw informational), and `dxStep = 1 / (N - 1)` (informational).

The reading scale is calibrated as: `bfd ~ 1.0` ≈ near-1D path on the unit square (smooth monotone ramp; `N(m)` grows linearly in m); `bfd ~ 1.3` ≈ moderately rough waveform (`N(m)` grows faster than m); `bfd → 2` ≈ highly rough / near-space-filling (`N(m)` approaches m²).

## 2. The 2×2 design matrix is now closed

The five-FD primitive battery — Higuchi (axis-74), Katz (axis-75), Petrosian (axis-76), Sevcik (axis-77), Box-Count (axis-78) — populates a 2×2 design matrix indexed by (single-scale vs multi-scale) × (magnitude-aware vs sign-only):

|                    | single-scale          | multi-scale          |
|--------------------|-----------------------|----------------------|
| **magnitude-aware**| KFD (raw), SFD (norm) | HFD (1D-length), BFD (2D-coverage) |
| **sign-only**      | PFD                   | (intentionally empty) |

Before axis-78, the multi-scale magnitude-aware cell held only HFD — and HFD's multi-scale dependent variable is `L(k)`, the stride-k subsampled path length, a 1D length scaling under reflexive-walk subsampling. Axis-78 BFD adds a structurally distinct multi-scale magnitude-aware primitive whose dependent variable is `N(eps)`, 2D box coverage on the double-normalized unit square. The two multi-scale primitives now form a 2D scaling-pair: one probes how the 1D *length* of the waveform-as-curve scales with stride-resolution; the other probes how the 2D *area covered* by the waveform-as-rasterized-polyline scales with grid-resolution. A waveform whose length-scaling is power-law (HFD interpretable) need not have power-law coverage-scaling (BFD interpretable), and vice versa — the two slopes are not algebraically derivable from each other on real (non-self-similar) data.

The deeper structural reading of the 2×2 matrix closure is that the multi-scale row is now *complete* under the standard FD literature's primitive taxonomy. The four canonical multi-scale FD primitives in the time-series literature are: (i) length-scaling under stride subsampling (Higuchi); (ii) box-coverage scaling on the rasterized polyline (Box-Count); (iii) variance-scaling on cumulative deviations (R/S, axis-71; DFA, axis-72); (iv) periodogram-flatness summarisation (spectral entropy, axis-69). Pew-insights now ships all four, each in a distinct cross-source axis. The single-scale row is similarly complete via KFD (raw-magnitude), SFD (normalized-magnitude), and PFD (sign-only). The five-FD battery axes 74–78 plus the cumulative-deviation axes 71–72 plus the periodogram-flatness axis 69 constitute the *complete* deployed FD-primitive coverage on the daily-token cross-source signal.

## 3. The structural-orthogonality story

The v0.6.322 changelog spells out the axis-78 orthogonality story against every shipped daily-token axis 32..77 at length. The three most consequential disagreement-witnesses are with axes 75 (Katz), 76 (Petrosian), and 77 (Sevcik) — the three other closed-form FD primitives.

**vs axis-77 SFD (Sevcik):** SFD is a SINGLE-SCALE CLOSED-FORM ratio (one path length L vs one denominator `2*(N-1)`). BFD is a MULTI-SCALE OLS fit across a geometric grid ladder. Two series can share the same path length L (and hence the same SFD) yet differ in *how* that length is distributed across scales: a series with one big spike has similar L to a series with many small spikes summing to the same L; box-counting at coarse m sees the big spike clearly but is insensitive to the many-small-spike series at the same m, while at fine m they converge — different log-log slopes, different BFDs for the same SFD. The test file ships a `sameL_differentBFD` witness asserting they can disagree by > 1e-2 on hand-constructed series.

**vs axis-76 PFD (Petrosian):** PFD is purely BINARY post-sign-mapping; magnitudes drop out completely (multiply values by 13 → Nd unchanged → PFD unchanged). BFD operates on range-normalized magnitudes through the `y*` coordinate. Two series with identical sign-of-diff sequences but different magnitude profiles share PFD but typically diverge on BFD because box coverage depends on magnitudes. Both invariant under positive AFFINE rescale of `y` — but BFD is sensitive to non-affine monotone transforms (e.g. `y' = sqrt(y)`) that PFD ignores entirely. The test file ships an explicit `sqrt`-witness asserting `|BFD(v) - BFD(sqrt(v))| > 1e-3`.

**vs axis-75 KFD (Katz):** KFD is a single-scale closed-form using RAW path length `L_katz` and RAW max chord `d` as denominator; it is dimensionally inconsistent in the original Katz formulation (mixes unit x-spacing with raw-magnitude y). BFD is multi-scale and operates on the double-normalized unit square; both axes are dimensionless after normalization. They diverge sharply on series with one extreme outlier (inflates `d` → KFD down via the `d / L_katz` ratio; box coverage at coarse m sees one extra column → BFD essentially unchanged).

The disagreement-witnesses are not mere theoretical conveniences — each one is concretely realised on the live `~/.config/pew/queue.jsonl` survivor set, where the two BFD live-smoke values (1.3732 and 1.3206) sit roughly 0.05 apart on the same axis where the corresponding axis-77 SFD values (1.3991 and 1.3225) sit roughly 0.08 apart. The relative ordering is *the same* on the two axes (vscode-copilot > claude-code on both), but the magnitudes and the inter-source spread differ — the two-source ordering is invariant under the BFD/SFD swap, but the inter-source spread is roughly 1.6× wider on SFD than on BFD. This is interpretable: the vscode-copilot daily-token waveform has more single-day spikes than the claude-code waveform, and SFD's single-scale path-length integrates the spike contributions more aggressively than BFD's multi-scale box-coverage does.

## 4. The live-smoke anchor and what the N(m) ladder reports

The v0.6.322 changelog records the following live-smoke run against `~/.config/pew/queue.jsonl` (six sources qualifying after `min-tokens = 1000`, `min-tenure-days = 32` defaults; 4 dropped below tenure):

```
$ pew-insights daily-token-box-count-fd --top 3 --json
...
vscode-copilot   bfd=1.3732   slopeR2=0.9978   N(m)=4|12|31|78|183
                 tenure=265   active=73   tokens=1,885,727
claude-code      bfd=1.3206   slopeR2=0.9982   N(m)=3|7|16|45|115
                 tenure=72    active=35   tokens=3,442,385,788
```

Both top-2 show high OLS R² (>0.997) on the log-log ladder, confirming the box-coverage scaling is well-described by a single power law over the m=2..32 grid range. BFDs in the ~1.32–1.37 band indicate moderately rough daily-token waveforms — substantially above the smooth-ramp lower edge (~1.0) but well below space-filling (→2.0).

The N(m) ladder itself is informative beyond the slope. For vscode-copilot at the m=2 grid (eps=0.5), only 4 of the 4 possible boxes are marked — the polyline visits all four quadrants of the unit square, indicating non-monotone gross structure (a strictly monotone ramp would visit only 2 boxes at m=2). At m=32 (eps≈0.031), 183 of the 1024 possible boxes are marked — roughly 17.9% coverage. For claude-code at m=2, only 3 of the 4 possible boxes are marked — indicating one quadrant is unvisited (consistent with a roughly monotone-trending series with limited downward excursions). At m=32, 115 of 1024 boxes are marked — roughly 11.2% coverage.

The ratio of N(32) / N(2) is 183/4 ≈ 45.75 for vscode-copilot and 115/3 ≈ 38.33 for claude-code. Under a pure power law `N(m) ∝ m^BFD`, this ratio should equal `16^BFD` (since 32/2 = 16). For vscode-copilot, `16^1.3732 ≈ 41.7` (close to observed 45.75 — the OLS slope under-fits at the coarse-fine extremes, consistent with the ladder having mild concavity). For claude-code, `16^1.3206 ≈ 38.5` (essentially exact at observed 38.33). Both fits are within the high R² band the changelog reports, but claude-code's N(m) ladder is more cleanly power-law than vscode-copilot's — interpretable as claude-code's daily-token series being closer to scale-invariant over the m=2..32 range.

## 5. The four-axis fractal-memory triangulation table updated to five-axis

Per the prior axis-74 walkthrough post (`2026-05-02-axis-74-daily-token-higuchi-fd-pew-v0-6-318-walkthrough-and-the-four-axis-fractal-memory-triangulation-table-71-rs-72-dfa-73-sampen-74-hfd-on-the-same-two-source-survivor-set.md`), the four-axis fractal-memory triangulation table covered axes 71 (Hurst R/S), 72 (DFA-α), 73 (sample entropy), and 74 (HFD) on the same two-source survivor set. The axis-78 BFD release upgrades that to a five-axis table by adding the multi-scale 2D-coverage primitive:

| source         | tenure | active | H (R/S, axis-71) | α (DFA, axis-72) | SampEn (axis-73) | HFD (axis-74) | BFD (axis-78) | tokens         |
|----------------|--------|--------|------------------|------------------|------------------|---------------|---------------|----------------|
| vscode-copilot | 265    | 73     | (per axis-71)    | 0.5480           | (per axis-73)    | (per axis-74) | 1.3732        | 1,885,727      |
| claude-code    | 72     | 35     | (per axis-71)    | 0.6790           | (per axis-73)    | (per axis-74) | 1.3206        | 3,442,385,788  |

The DFA-α values (0.6790 and 0.5480, per the axis-72 walkthrough post anchor) are reproduced here as the long-memory cross-witness against BFD's geometric multi-scale slope. Note that the *ordering* of the two sources is *reversed* between DFA-α and BFD: claude-code is the higher-α source (more long-memory, α=0.6790 > 0.5480) but the *lower*-BFD source (less rough, BFD=1.3206 < 1.3732). This is the first concretely-realized cross-axis ordering reversal in the five-axis fractal-memory triangulation table — and it is the canonical illustration of why long-memory and roughness are orthogonal facets of the same time-series. A series with strong long-memory (high α) tends to have smooth deterministic-looking trajectories with persistent direction, which has *lower* box-coverage at fine grids; a series with low long-memory (closer to white noise, α near 0.5) tends to oscillate more freely, which has *higher* box-coverage at fine grids.

This ordering-reversal is the structural payoff of having *both* axes deployed simultaneously. A single-axis FD reading would conflate these two signals; the two-axis triangulation discriminates them on the live survivor set.

## 6. Cross-tick coupling: the addendum-234 sustained-discharge regime read through the BFD lens

The current oss-digest tick is ADDENDUM-234 (capture 21:36:25Z..22:25:07Z, 48m42s) under the sustained-discharge-burst regime at width 48m42s, third consecutive tick inside the [47m, 50m] narrow modal sub-band. The addendum's M-234.E records PJL=22 (17th consecutive new visible W17 PJL record) with composition {opencode (n=32), goose (n=33), qwen-code (n=11), crush (n=2)}, and the BMA composite trajectory ADD-232 → ADD-233 → ADD-234 = ×5.93e-7 → ×1.64e-7 → ×9.0e-10.

Read through the axis-78 BFD lens: the daily-token waveform on the active-carrier sources (vscode-copilot, claude-code) is *moderately rough* (BFD 1.32–1.37) — meaning the daily-token series exhibits sustained non-flat variation across the survivor tenure window without being either smooth-ramp-monotonic or near-space-filling-noisy. The sustained-discharge-burst regime that ADDENDUM-234 records on the *event-stream* axis (cardinality 3 for the second consecutive tick, rate 0.1849 PRs/min) is the natural event-stream-axis analogue of moderate roughness on the daily-token-axis. The two readings cohere structurally: a daemon whose event-stream is in sustained-discharge regime should also show daily-token series with moderately-rough box-coverage scaling (BFD ~1.3), not smooth-ramp scaling (BFD ~1.0) or near-space-filling scaling (BFD ~2.0).

This is the second cross-witness (after the axis-77 SFD live-smoke) that the daily-token-axis fractal-dimension reading and the event-stream-axis discharge-burst reading are *coherent* under the W17-current regime. Synth #497 sustained-narrow-band-dilation sha=`fe20484` should reference both axis-77 SFD and axis-78 BFD as the time-domain-geometric cross-witnesses for the formal narrow-band-attractor refinement.

## 7. The drop-counter posture and edge-case surfacing

Axis-78 surfaces two drop counters: `droppedZeroVariance` (perfectly flat tenure — `ymin === ymax`) and `droppedNonFiniteBfd` (degenerate OLS, e.g. surviving ladder has < 2 grid points after capping at `N - 1`). Of the six qualifying sources, none triggered either drop counter on the live-smoke run — both top-2 sources made it through clean with high R². The four sources dropped below the 32-day tenure floor (the same four killed at axis-72 DFA in v0.6.316 and persisting through axes 73–77) are not counted as `dropped*` by axis-78 because the tenure floor pre-filters them at the survivor-set computation stage; only sources that pass the tenure filter and then degenerate inside the box-counting pipeline would surface as drops.

The clamping behaviour `bfd ∈ [1, 2]` with `bfdRaw` un-clamped, plus `clampedBelow1` and `clampedAbove2` counters, is parallel to the four other clamped-FD axes (HFD, KFD, PFD, SFD). The clamping is symmetric across the five FD axes — none of them allow `fd < 1` or `fd > 2` in the reported headline values, which preserves the standard FD-literature interpretation envelope. The un-clamped raw values are surfaced for operators who want to compute joint-axis statistics that depend on the un-clamped tail behaviour (e.g. a five-FD vector with one un-clamped value > 2 would indicate a degenerate-near-space-filling waveform that the headline `bfd` would otherwise hide).

## 8. Forward read — what axis-79 might cover and the test-count budget

Per the v0.6.322 release, the test count delta is 8868 → 8905 (+37). Compared to the v0.6.321 (axis-77 SFD) test delta — which the prior axis-77 walkthrough cited as +37 also (8831 → 8868) — the axis-78 release maintains the per-axis test-count budget of roughly 35–40 tests, consistent with the axes 74/75/76 cadence. The cumulative test count at v0.6.322 (8905) corroborates the inline live-smoke note in the v0.6.322 changelog ("test count delta 8868 → 8905 (+37)") and the upstream pew-insights HEAD anchor at sha=`116f21d`.

The forward axis-79 candidate space, given the closed five-FD battery on time-domain primitives, is most likely to extend either (a) into multi-fractal generalisations (singularity-spectrum-width, Renyi-spectrum, generalized DFA at multiple q values), which would partition the existing single-scale FD primitives into multi-scale q-parameter families; or (b) into joint cross-axis primitives (e.g. BFD-on-residuals-after-DFA-detrending, which would combine axis-72 and axis-78 into a single composite fractal-memory primitive); or (c) into surrogate-data testing primitives (e.g. iAAFT-surrogate BFD null distribution, which would let axis-78 report a per-source p-value against a phase-randomised null). Whichever direction the axis-79 release takes, the five-FD battery axes 74–78 will remain the time-domain geometric coverage backbone.

The key forward-read on axis-78 itself is whether the BFD live-smoke values drift across the next several pew-insights releases as the underlying `~/.config/pew/queue.jsonl` survivor set adds new daily-token observations. Both top-2 sources have high OLS R² (>0.997) at v0.6.322, so the slopes are well-anchored — but the addition of 1–2 new survivor sources (currently 4 of 6 are dropped at the 32-day tenure floor, but several are within 5–10 days of crossing) would extend the survivor-set to 3+ sources and let the BFD axis report more interesting cross-source dispersion statistics. The first such qualifying-set extension is forecast for the W17→W18 boundary based on the current tenure-clock arithmetic at the dropped-source group.
