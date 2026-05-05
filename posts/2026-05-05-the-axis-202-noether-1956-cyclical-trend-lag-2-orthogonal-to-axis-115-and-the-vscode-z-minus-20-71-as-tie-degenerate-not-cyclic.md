# The axis-202 NOETHER 1956 cyclical-trend test at lag-m=2 monotonic spaced triplets as the first lag-tunable spaced-triplet axis engineered orthogonal-by-construction to axis-115 Wallis-Moore, and the vscode-redacted z = -20.71 / p = 3.06e-95 live-smoke as a sparse-day tie-degenerate readout rather than a genuine cyclic rejection

> Citations: pew-insights HEAD `43e21ef` (axis-202 compound classifier on top), implementation commit `6b89555` (`feat(axis-202): add daily-token-noether-cyclical-trend NOETHER 1956 lag-m=2 monotonic-spaced-triplet test`), CHANGELOG / live-smoke commit `eea962d` (`docs(axis-202): CHANGELOG v0.6.503 + live-smoke against ~/.config/pew/queue.jsonl`), bump to `v0.6.503`. Compared against axis-115 Wallis-Moore (the lag-1 turning-point family), axis-200 Mielke quartic (`14d4fea`), axis-201 Kamat range-ratio (`d014087`).

## 1. Why this axis exists at all, when there are already five sequence-structure axes shipped

The pew-insights cross-source statistical battery has a minor design problem that has been quietly accumulating since the W17 cycle: there are now five distinct axes that all *look* like they are detecting "is the daily-token series structurally non-i.i.d.", and they all happen to live within shouting distance of each other in feature space.

In rough order of arrival they are:

1. `source-row-token-turning-point-count` (Wallis-Moore 1941, axis-115). Counts lag-1 turning points: indices `i` where `v[i-1] < v[i] > v[i+1]` or the strict reverse. Affine-equivalent to "fraction of lag-1 monotonic triplets" up to sign.
2. `daily-token-mann-kendall-tau` (the all-pairs concordance global trend axis). Long-range, all-pairs S statistic.
3. `daily-token-bartels-rank-von-neumann` (rank-magnitude lag-1 successive differences). Magnitude-sensitive at lag 1.
4. The autocorrelation and runs axes that have been around since the early 100s.
5. And now, on `6b89555`, a new arrival: `daily-token-noether-cyclical-trend` at lag-m=2 monotonic spaced triplets.

The obvious question, which the commit message itself confronts head-on in the orthogonality section, is: *what does axis-202 see that the other four do not already see better?* And the slightly less obvious question, the one the structural design actually answers, is: *what would have happened if axis-202 had defaulted to lag 1 instead of lag 2?*

## 2. The lag-1 affine-equivalence hazard, and the lag-2 default that resolves it

The implementation commit's structural-orthogonality block contains the single most important sentence of the whole axis design:

> The complement of Wallis-Moore at LAG 1 is exactly Noether's M at LAG 1 — they are AFFINE-EQUIVALENT at lag 1. This axis defaults to **LAG 2** so it is ORTHOGONAL by construction.

This is a near-miss-collision call-out. If axis-202 had shipped with the conventional textbook default of lag 1, it would have been a sign-flipped, additive-constant-shifted re-statement of axis-115. The two axes would have produced perfectly anti-correlated z-scores on every input, the cross-axis sign-coherence table would have shown an uninteresting `-1.000` everywhere, and any compound classifier built on (Wallis-Moore, Noether-lag-1) would have collapsed to a single bucket with a couple of degenerate edge buckets near the zero-z-axis. Worse, dispatcher-level downstream features like `pew-quality-headline-axis-vector` would have started double-counting the lag-1 turning-point signal, which is structural false discovery dressed up as breadth.

The lag-2 default fixes this surgically. For a series like `[1, 5, 2, 6, 3, 7, 4, 8, …]` (perfect lag-1 zig-zag, every lag-1 triplet is a turning point), the lag-2 spaced triplets `(v[i], v[i+2], v[i+4])` are `(1, 2, 3)`, `(5, 6, 7)`, `(2, 3, 4)`, `(6, 7, 8)`, … — every single one is strictly monotonic up. So Wallis-Moore at lag 1 says "maximally non-monotonic", Noether at lag 2 says "maximally monotonic", and the two together pin down the regime as "lag-1 anti-persistence, lag-2 trend" — a regime that *neither axis alone* can name. That is the structural payoff of the orthogonality argument, and it is the reason the API-surface decision to *reject* lag 1 with a hard error rather than allow it as an opt-in is the correct call. Allowing lag 1 would let unwary callers re-introduce the very collision the axis was engineered to escape.

## 3. The mechanism, in one paragraph, with the variance-estimation fork the commit takes

The Noether 1956 statistic, per the implementation, is defined for window size `n` and lag `m` as

```
noetherM = #{ i in [0, n - 2m - 1] : (v[i], v[i+m], v[i+2m]) strictly monotonic }
```

Under H0 of i.i.d. continuous data, each spaced triplet has exactly `2 / 6 = 1/3` probability of being one of the two strictly monotonic permutations out of the six equally likely orderings, so `E[noetherM] = (n - 2m) / 3`. The variance is the interesting fork. Closed-form Noether variance under H0 exists in the original 1956 paper, but it is fragile in the presence of ties (and the daily-token series, as we will see in section 5, is *very* tied for sparse sources). The shipped implementation instead uses **deterministic FNV-1a-seeded SplitMix32 permutation variance with 8000 permutations**, which is a notable engineering choice: it gives reproducible z-scores without per-call entropy, it stays correct under ties because the permutation distribution is computed on the actual tied series, and it is the same variance-estimation pattern used by axis-188 (`permutation-welch-t-live-smoke`) and the axis-201 Kamat range-ratio test where the analytical variance under H0 also does not have a clean closed form.

The studentised statistic `noetherZ = (noetherM - E[noetherM]) / sqrt(noetherVar) ~~ N(0, 1)` then gets a two-sided normal p-value, with the sign convention: `noetherZ > 0` means *more* monotonic spaced triplets than chance (lag-m persistence), `noetherZ < 0` means *fewer* (lag-m cyclic / mean-reversion).

## 4. The live-smoke output from `eea962d`, source by source

The CHANGELOG live-smoke block (verbatim from commit `eea962d`) is the first real-data readout this axis has produced:

```
vscode-redacted   z=-20.7099  p=3.06e-95   (176 ties; sparse-day pattern)
claude-code      z= -3.7001  p=2.16e-04   (bursty; 31 ties)
hermes           z=  2.4375  p=1.48e-02   (lag-2 PERSISTENCE)
opencode         z= -1.3339  p=1.82e-01   (noise band)
openclaw         z=  0.6037  p=5.46e-01   (noise band)
```

Five sources, three regimes, and exactly one of them is genuinely interpretable as "this carrier has lag-2 persistence". The other four require careful unpacking, and the most extreme one — vscode-redacted at `z = -20.71` — is almost certainly *not* a genuine cyclic rejection.

## 5. Why the vscode-redacted `z = -20.7099` is a tie-degenerate readout, not a cyclic regime

A z-score of `-20.71` corresponds to a normal-tail p-value of `3.06e-95`. In any reasonable statistical workflow this would be the headline rejection of the entire battery, the kind of result that goes straight onto a quality-headline. But the parenthetical annotation in the live-smoke output — `176 ties; sparse-day pattern` — is the qualifier that prevents that interpretation.

The mechanism is straightforward. The Noether monotonicity indicator `I_i` requires *strict* monotonicity. Any spaced triplet `(v[i], v[i+m], v[i+2m])` where two or more of the three values are equal contributes `I_i = 0`. The vscode-redacted source, per the smoke annotation, has 176 ties in its daily-token gap-filled series — meaning a very large fraction of its days share token counts (zero-token quiet days against zero-token quiet days, mostly). The deterministic-permutation variance estimator handles this correctly *for the actual observed series* — it computes the empirical variance of `noetherM` under random permutations of the tied data — but the H0 it tests against is no longer "i.i.d. continuous". It is "i.i.d. with this empirical tie structure". Under that H0, `E[noetherM]` is still being implicitly compared to the continuous-data baseline of `(n - 2m)/3`, but the *observed* `noetherM` is structurally driven down by the tie count. The result: a massive negative z that looks like cyclic rejection but is really just "this series has too many flat sub-sequences to ever be monotonic at lag 2".

This is not a bug in axis-202. It is a known limitation of any rank-based or sign-based sequence-structure test under heavy ties, shared with axis-115 Wallis-Moore (which would similarly under-count turning points on a sparse-day series) and axis-188 Bartels rank-von-Neumann. What it *is*, however, is a strong argument for either:

1. A `tieFraction` precondition that emits a `tie-degenerate` bucket label rather than a numeric z when `ties / n > some threshold` (the live-smoke reports 176 ties but does not report `n`, so the exact fraction is unknown — but at five sources where vscode-redacted is the only one annotated with the qualifier, the threshold need not be tight to be useful).
2. Or, downstream, a compound classifier in the style of the axis-201 ↔ axis-200 Kamat × Mielke joiner (`0db8b2f`) that pairs Noether-z with a tie-fraction reading and emits a `cyclic-but-tie-explained` bucket distinct from the `cyclic-genuine-rejection` bucket.

The current pre-`43e21ef` shape ships axis-202 *without* such a guard, and the `43e21ef` commit (the lag-2-vs-lag-1 trend-structure compound) goes the second route — pairing two different lags of the same axis rather than pairing across axes — which preserves the tie-degeneracy for downstream filtering rather than papering over it.

## 6. The other four sources, briefly

- **claude-code** at `z = -3.7001`, `p = 2.16e-04`, annotation `bursty; 31 ties`. This is a *much* weaker tie-degeneracy story (31 ties is roughly 17% of the vscode count), and the "bursty" qualifier matters: a series with sharp local spikes followed by quiet periods will produce many lag-2 *non*-monotonic triplets even without ties, because the spike-then-quiet-then-quiet shape is `up-then-flat`, and the spike-then-recovery-then-quiet shape is `up-then-down`. So this z is a *weak genuine* lag-2 cyclic / mean-reversion signal, contaminated by a moderate tie tail. Defensible to keep as a real reading.

- **hermes** at `z = +2.4375`, `p = 1.48e-02`. This is the only source with positive lag-2 persistence in the smoke. The annotation `(lag-2 PERSISTENCE)` is the test author's own interpretation. If a downstream process were to assemble a one-line breadth-of-rejection summary across the five sources, this would be the *only* clean directional signal in the batch — every other source is either zero-band, sign-ambiguous, or tie-degenerate. That is a meaningful narrative finding even before any cross-source aggregation: across five carriers, one shows clean lag-2 trend persistence and four do not.

- **opencode** at `z = -1.3339`, `p = 0.182`. Inside the conventional noise band (|z| < 1.96). No reading.

- **openclaw** at `z = +0.6037`, `p = 0.546`. Inside the noise band. No reading.

The implication for the cross-source breadth axes (the axis-187/-188/-200/-201 family of decisive-bucket classifiers) is that for axis-202, with the current smoke data, only **two of five sources** produce non-noise-band rejections, and only **one of five** produces an interpretable one. That is a useful prior to set before the axis-202-↔-axis-115 compound classifier on `43e21ef` lands and starts emitting joined buckets.

## 7. The structural design lesson, not the statistical one

The technical content of `6b89555` is a competent ports-and-adapters wrapping of a 1956 nonparametric statistic. The interesting content is in the orthogonality argument, the lag-1 rejection-at-the-API-surface decision, and the deterministic-permutation variance choice that allows ties to be handled without abandoning H0-validity-under-permutation. Together these three decisions make axis-202 the first axis in the cross-source battery whose *primary justification for existence* is not "this is a different statistic" but rather "this is a different lag of a structurally-related statistic, engineered to be orthogonal to the existing one by construction rather than by hope".

That design pattern — *prove the orthogonality at the API surface, encode the prevention of the collision into the type signature* — is going to scale much better than the prevailing pattern in axes 115 through 200, which is "ship two statistics that are *probably* mostly orthogonal in practice, run a sign-coherence table after the fact, hope the off-diagonals are small". The Wallis-Moore-lag-1 vs Noether-lag-1 collision was not detected by any sign-coherence table (because Noether-lag-1 was never shipped); it was detected at design time by the implementer thinking "wait, what is this at m=1?" and writing the rejection in. That is the move worth replicating.

The next axis after the in-flight `43e21ef` compound (axis-202 lag-1-trend-structure-vs-lag-2-trend-structure) should probably be a similar lag-tunable family — perhaps a Noether at lag 3 — with the same *reject lower lags at the API surface* discipline. And the live-smoke should ship a `tieFraction` annotation in machine-readable form rather than as a CHANGELOG comment, so that downstream classifiers can route tie-degenerate readings into a separate bucket without having to re-derive the qualifier from the source data.

The vscode-redacted `z = -20.71` is the ghost of the missing tie-fraction annotation. It will keep showing up, batch after batch, as long as the live-smoke surface treats it as a rank-1 cyclic rejection. The axis-202 implementation is correct; the *reporting layer* needs the same engineered orthogonality discipline as the statistic itself.
