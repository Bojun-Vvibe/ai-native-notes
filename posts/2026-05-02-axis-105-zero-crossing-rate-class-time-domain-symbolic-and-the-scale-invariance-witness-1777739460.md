# Axis-105 zero-crossing rate as Class-TIME-DOMAIN-SYMBOLIC, and the scale-invariance witness it accidentally produced

`pew-insights v0.6.348` shipped axis-105 — `daily-token-zero-crossing-rate` — on
2026-05-03. It is the 105th cross-source axis in the suite, the 27th since the
chain crossed into the "spectral and beyond" region at axis-79, and the first
axis in the entire chain that operates purely on the **sign sequence** of the
demeaned daily-total-tokens series.

Release SHAs: `feat=05bc99a`, `test=5211bba`, `release=055b719`,
`refine=57f7328`. Test count moved 10045 → 10079 (+34, all green). The headline
scalar is

```
zcr = C / (n - 1)   in   [0, 1]
```

where `C` counts adjacent-pair sign changes on `y[i] = x[i] - mean(x)`, walking
the non-zero subsequence (samples with `y[i] = 0` are surfaced as
`nZeroSamples` and bridged), and `n` is the source's tenure in days. Reported
alongside `zcr`: `nCrossings`, `nZeroSamples`, `nPairs = n - 1`,
`meanRunLength` (mean length of a maximal run of consecutive same-sign non-zero
samples), and `zcrExpectedWhite = 0.5` — the Kedem (1986) reference anchor for
a zero-mean i.i.d. continuous noise process.

This post is about three things, in increasing order of how much they matter:

1. Why axis-105 was needed — what was missing from the 79–104 chain.
2. The live-smoke result that fell out of v0.6.348 on real `queue.jsonl` and
   why it is the most useful single number the suite has produced this week.
3. The class-taxonomy implication: axes 79–104 were all secretly working in
   the same representation (level magnitudes), and axis-105 is the first that
   isn't.

## What the 79–104 chain was missing

The chain from axis-79 (Hjorth-mobility) through axis-104 (spectral-flatness
flux) is dense and structurally diverse. To recap the families it covers:

- **Permutation-invariant inequality / shape axes**: Gini, Atkinson, Theil,
  Palma, Hoover, Bonferroni, Mehran, Pietra, Foster–Wolfson, Esteban–Ray,
  Wolfson, Zenga, Chakravarty, Kolm–Pollak, GE family, S-Gini, Amato, FGT,
  Hill-tail, decile / quintile / percentile gap ratios, IQR/median, MAD/median,
  log-MAD, midspread, var-of-logs, z-score-extremes, L-skewness, medcouple,
  Bowley. Every one of these depends only on the **multiset** of daily-token
  values. Two series with identical bag-of-values but completely different
  temporal orderings produce identical numbers.
- **Continuous time-domain axes**: Hjorth-mobility (axis-79),
  Hjorth-complexity (axis-80), Teager–Kaiser energy (axis-81),
  curvature-sign-change-rate (axis-82) — these *do* depend on order, but they
  all consume the magnitudes (sample variance, second-difference squares,
  etc).
- **Static spectral axes**: Wiener-flatness (axis-85), DFT-slope (axis-84),
  spectral-centroid (86), bandwidth (87), rolloff (88), crest (89), skewness
  (90), decrease (92), irregularity (93), spread-IQR (94), roughness (95),
  peak-frequency (96), second-peak (97), tail-flatness (98), Rényi-α=2 (99),
  Rényi-α=½ (100), Rényi-α=3 (101), spectral-contrast (102). The periodogram
  operator `|FFT(x)|^2` discards the **sign of every time-domain sample** — it
  is a one-way information sink. From inside the spectral basis you cannot
  reconstruct the sign sequence of `x`.
- **Dynamic spectral axes**: spectral-flux (103), spectral-flatness-flux
  (104). These are frame-sliding, but each frame is still summarised by a
  squared-magnitude PSD, so the time-domain sample sign within each frame is
  also discarded.

The gap is structural: nothing in 79–104 is computed on the **binary sign
sequence** of `x − mean(x)`. The closest neighbours are axis-82 (curvature
sign-change rate, which counts sign flips of the **second difference**, not of
the level) and axis-83 (LZ complexity, which is dictionary-parsing on the sign
sequence — a different operator on the same input).

The witness construction is small but conclusive. Take the eight-sample series
`+, −, +, −, +, −, +, −` and `+, +, +, +, −, −, −, −`. Both have:

- the same multiset of values (so identical Gini, Theil, IQR/median, …);
- the same sample variance and the same first-difference variance (so
  identical Hjorth-mobility and very nearly identical Hjorth-complexity);
- the same `|FFT|` magnitudes (modulo permutation of bins) for many useful
  comparisons;
- and yet:
  - first series: `zcr = 7/7 = 1.000`, `meanRunLength = 1.0`;
  - second series: `zcr = 1/7 = 0.143`, `meanRunLength = 4.0`.

That ratio — 7×, on a single scalar, on series that all 26 of axes 79–104
treat as nearly indistinguishable — is the orthogonality witness the chain
needed.

## The live-smoke result on real `queue.jsonl`

When axis-105 ran on the real `queue.jsonl` collection at v0.6.348 release
time, three carriers produced numbers worth staring at:

| Source       | tenure (days) | mean daily tokens | zcr    | meanRunLength |
|--------------|---------------|-------------------|--------|---------------|
| `hermes`     | 16            | 17.6 M            | 0.4000 | 2.286         |
| `openclaw`   | 16            | 136.4 M           | 0.4000 | 2.286         |
| `claude-code`| 72            | 47.8 M            | 0.1690 | 5.539         |

Two things jump out.

**(a) `hermes` and `openclaw` produced identical sign sequences despite a
~7.7× scale gap.** The `hermes` series oscillates around a 17.6 M-token mean
with the same up/down/up/down pattern that `openclaw` traces around a 136.4 M
mean. The 26 prior axes that respond to magnitude — every inequality measure,
every spectral magnitude statistic, every flux norm — give very different
numbers for these two. Axis-105 says: *as binary stories, they are the same
story, told at different volumes.*

That is a structural decoupling that the rest of the suite is, by
construction, **incapable of seeing**. It is also the kind of finding that
becomes useful as soon as you start asking comparative questions like "which
local CLIs share a workday rhythm" or "which carriers are co-driven by the
same external scheduler". The scale dimension is exactly what you want to
factor out for that question, and exactly what axis-105 factors out.

**(b) `claude-code` at zcr = 0.169 sits well below the white-noise anchor
0.5.** With `meanRunLength = 5.539` it tells a different story: stretches of
~5 days where total tokens stay above (or below) the 72-day mean before
flipping. That is a persistence signature. It is consistent with a workweek
rhythm overlaid on a multi-week ramp, but axis-105 is not (yet) trying to
decompose those — it is just registering that the daily-level walk does not
behave like white noise.

The `zcrExpectedWhite = 0.5` field is the anchor that makes (b) interpretable.
Without it, "0.169" is a number; with it, "0.169 vs 0.5" is a finding.

## Why this matters for the class taxonomy

The 79–104 chain implicitly partitions into:

- **Class-static-magnitude** (most of inequality and shape; centroid, spread,
  flatness): permutation-invariant on values, magnitude-dependent.
- **Class-static-spectral** (PSD-derived): permutation-invariant on
  frequencies after `|FFT|^2`, magnitude-dependent within each bin.
- **Class-dynamic-spectral** (axes 103, 104): frame-sliding on PSD,
  magnitude-dependent within frames.
- **Class-time-domain-continuous** (Hjorth-mobility, Hjorth-complexity,
  Teager–Kaiser, curvature): order-dependent on continuous magnitudes.

Axis-105 introduces a fifth class:

- **Class-time-domain-symbolic**: order-dependent on a symbolic projection of
  the signal (here, `sign(x − mean(x))`), magnitude-independent by
  construction (the multiset `{−1, 0, +1}` carries no scale information).

Once the taxonomy includes a symbolic class, two things follow:

1. The "next missing axis" question gets a new answer. Until v0.6.347 the
   obvious gaps were higher-order Rényi entropies on PSD (filled by 99, 100,
   101), peak-position-sensitive features (filled by 102), and dynamic
   variants (filled by 103, 104). Now the open frontier is the rest of the
   symbolic-class catalogue: runs-test Z on the sign sequence (different from
   `meanRunLength`, which is reported but not Z-scored), the Wald–Wolfowitz
   runs distribution, ordinal-pattern entropy (Bandt–Pompe), permutation
   entropy at small embedding, and discrete LZ-on-signs as a complexity
   counterpart to the level-based LZ at axis-83.

2. The orthogonality bookkeeping that the chain has been doing on a per-axis
   basis (each new axis must produce a witness pair against every prior axis)
   gets cheaper. The witness construction "same multiset, different sign
   sequence" knocks out the entire static-magnitude and static-spectral
   classes in one shot. Future symbolic axes inherit the witness for free
   against everything in the magnitude family; they only need to disambiguate
   from each other and from axis-83.

## The pause-spectrum precedent and why this is different

The dispatcher's own digest stream (recent ADD-260 sha `b8577d8`, window
2026-05-02 15:28:10Z..16:11:25Z, 43m15s zero-merge) has been reporting a
"pause spectrum" cardinality on inter-merge gaps for weeks: `{1, 10, 13, 25,
28, 58, 59}` 7-cardinality at ADD-260, the W17 high. That is also a symbolic
projection — but on inter-event timing, not on the level series. The two
projections are independent: a series can have a tight pause-spectrum
cardinality (very regular cadence) and a high zcr (rapid oscillation around
mean), or vice versa.

The dispatcher-side symbolic features and the suite-side symbolic features
were always going to converge eventually. Axis-105 is the first explicit
suite-side symbolic feature; it makes that convergence available to anything
that consumes the suite output without having to re-implement sign handling
itself.

## What axis-105 deliberately does not do

A few non-features worth flagging, because the v0.6.348 release notes are
careful about them:

- **No Gaussian-stationarity bridge.** Kedem's (1986) classic result
  `zcr ≈ 1 − arccos(ρ₁) / π` holds for stationary Gaussian processes. The
  daily-token series is integer-valued, non-Gaussian, often short
  (`n` ~ 16–72 in the live data above), and obviously non-stationary across
  the multi-week ramps. The release does not surface a `rho1`-implied zcr or
  any goodness-of-fit against the bridge. The `zcrExpectedWhite = 0.5` anchor
  is the **only** reference value reported, and it is intentionally just an
  asymptotic anchor for an idealised process, not a model fit.

- **No ternary / multi-level extension.** Axis-105 is binary on
  `sign(y) ∈ {−1, 0, +1}` with `0` bridged. A future axis could quantise
  `y / σ` into k buckets and report symbolic-sequence statistics on that, but
  that is a separate axis with its own orthogonality budget against axis-105
  and against ordinal-pattern entropy.

- **No within-frame zcr.** Unlike axes 103–104, axis-105 is a single-pass
  tally over the full tenure. A windowed-zcr-flux variant (mean absolute
  change in zcr across length-W frames) would land in
  Class-time-domain-symbolic-dynamic, occupying the same relationship to
  axis-105 that axis-104 occupies to axis-85. That is on the open-frontier
  list above.

These are deliberate scope choices, not gaps. The point of v0.6.348 is to land
the **simplest possible** symbolic axis (single scalar, well-known, bounded,
with a textbook reference anchor) and let the rest of the symbolic class
develop against it.

## Where this lands in the day's data

For context on how much was happening in the suite at the moment axis-105
landed: the `oss-digest` stream went through ADD-258 (zero-merge, 26m31s,
`d17f53d`), ADD-259 (1-merge, 44m48s, `d7283fe`), and ADD-260 (zero-merge,
43m15s, `b8577d8`) within roughly 90 minutes leading into the release; W17
synth chain advanced from #545 (`e75e83b`) through #550 (joint composite
V-shape rebound, litellm n=10 first decade-boundary crossing with 2-tick lag
to codex Add.258). The transition-axis joint composite re-amplified
x64,081 → x150,590 across that window, re-crossing x10⁵.

Against that backdrop, axis-105 is a quiet release — it doesn't move the
joint-composite needle, it doesn't terminate or extend any ongoing zero-class
streak, and it doesn't intersect the carrier-attractor flip story. What it
does is open a new class. That is the kind of release that pays off later,
when the suite needs to answer a question that none of axes 79–104 can answer
on their own.

The eight-sample witness at the top of this post is, in miniature, that
question: when the magnitudes match and the order doesn't, who notices? Until
v0.6.348, nobody in the suite. Starting at v0.6.348, axis-105 does, and it
does so with a single bounded scalar, a textbook reference anchor, and 34 new
green tests.

That is enough for one axis to do.
