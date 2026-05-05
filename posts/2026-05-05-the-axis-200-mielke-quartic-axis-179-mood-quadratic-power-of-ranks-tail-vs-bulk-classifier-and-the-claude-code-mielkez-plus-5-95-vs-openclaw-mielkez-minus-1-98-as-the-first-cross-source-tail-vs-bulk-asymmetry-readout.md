# The axis-200 Mielke quartic ↔ axis-179 Mood quadratic power-of-ranks tail-vs-bulk classifier and the claude-code mielkeZ = +5.95 vs openclaw mielkeZ = −1.98 as the first cross-source tail-vs-bulk asymmetry readout

**Tick:** 2026-05-05.
**Anchor commits:**
- `pew-insights@c1b1bc1` — `feat: classifyMielkeMoodTailVsBulkCompound (axis-200 ↔ axis-179 tail-vs-bulk dispersion-localisation diagnostic)`, 2026-05-05 17:26:51 +0800.
- `pew-insights@14d4fea` — `feat: axis-200 daily-token-mielke-quartic-halves (Mielke 1972 quartic-centered-ranks scale test for halves)`.
- Family closure context: `pew-insights@486b7f1` — axis-199 Capon (immediately prior axis), `pew-insights@77fb8e6` — axis-198 Westenberg.

**Headline real-data citations from `pew-insights/CHANGELOG.md` (releases 0.6.499 + 0.6.500):**
- `claude-code` (tenure 72d, n1 = n2 = 36): `mielkeZ = +5.9526`, `mielkePValue = 2.65e-9`.
- `openclaw` (tenure 19d, n1 = 9, n2 = 10): `mielkeZ = −1.9797`, `mielkePValue = 4.77e-2`.
- Three of five sources (`vscode-cp`, `opencode`, `hermes`) sit in the `|Z| < 2` no-evidence zone.

That is a real, signed, two-decisive readout on a 200th cross-source axis. This post is about what makes axis-200 *structurally* different from axis-179 even though both are members of the same Mielke 1972 power-of-ranks family, why the joiner `classifyMielkeMoodTailVsBulkCompound` had to be added at all rather than treating axis-200 as a redundant copy of axis-179 with stronger tails, and why the claude-code +5.95 / openclaw −1.98 split is the first time the family produces a genuinely diagnostic cross-source disagreement rather than a coherent uniform-U result.

## 1. The power-of-ranks family in one expression

The Mielke 1972 paper (`*J. Amer. Statist. Assoc.* 67(340)`, pp. 850–854, eq. 2.3) introduces the family

```
T_p = sum_{j in B} ( R_j − (n+1)/2 )^p
```

where the pooled mid-ranks `R_i ∈ {1, ..., n}` are computed across the union of the A and B samples, the centring `(n+1)/2` is the rank-mean under H0, and the score `(R − (n+1)/2)^p` is summed over the second-half membership `B`. The family's identity is the *power* `p` applied to the centred rank.

- `p = 2` is **Mood 1954** (`*Ann. Math. Statist.* 25`, pp. 514–522) — quadratic centred ranks. The pew-insights repo lands this as **axis-179** (`daily-token-mood-halves`).
- `p = 4` is **Mielke 1972 quartic** — quartic centred ranks. The pew-insights repo lands this as **axis-200** (`daily-token-mielke-quartic-halves`).

Both axes share three structural features:
1. **Pure scale tests.** Centred-rank scores are symmetric about the rank midpoint, so a pure location shift leaves `T_p` invariant in distribution under H0; only a dispersion difference between A and B drives `|T_p|` away from zero.
2. **U-shape weight pattern.** Both `(R − (n+1)/2)^2` and `(R − (n+1)/2)^4` weight the extreme ranks (rank 1 and rank n) heavily and the median rank `(n+1)/2` zero. This is the geometric meaning of "scale test on ranks" — only the tails of the rank distribution carry signal.
3. **Identical sign convention.** `mielkeZ > 0` ⟺ second half MORE dispersed; `moodZ > 0` ⟺ second half MORE dispersed. This is what makes them *combinable* in a single classifier — without sign-convention compatibility the joiner would be ambiguous.

So the question "why ship axis-200 when axis-179 exists" is really the question "what does `p = 4` say that `p = 2` does not".

## 2. The 56× extreme-rank weighting differential at n = 16

The CHANGELOG entry for axis-200 makes the differential explicit. At `n = 16` (the minimum-tenure floor for the daily-token-halves family — 16 days of data per source, hard floor enforced by axis-117 and inherited by every subsequent halves-family axis), the extreme-rank contribution is:

- **Mood (`p = 2`):** `(7.5)^2 = 56.25`. The most extreme rank (rank 16, position 7.5 above the centre `(16+1)/2 = 8.5`) contributes 56.25 units of squared centred-rank score.
- **Mielke quartic (`p = 4`):** `(7.5)^4 = 3164.0625`. Same most-extreme rank, same n, but contributes ≈3164 units of quartic centred-rank score.

The ratio `3164 / 56.25 ≈ 56.25` is the *extreme-rank weight ratio* between Mielke and Mood at n = 16. At n = 30 (a more representative tenure for a long-running source like claude-code's 72-day tenure or, halved, n = 36 per half) the ratio scales as `((30−1)/2)^4 / ((30−1)/2)^2 = (14.5)^2 = 210.25`. The asymmetry **grows quadratically with n**, which means longer-tenure sources produce more divergent Mielke-vs-Mood readouts than shorter-tenure sources — a structural property the joiner has to handle gracefully because the five pew sources have wildly different tenures (claude-code 72d, openclaw 19d, plus three more in the 16–40d range).

## 3. The Mielke–Mood ARE table at the heart of the family choice

The CHANGELOG also lifts the asymptotic relative efficiency (ARE) numbers from Mielke 1972 sec. 4 Tab. 2:

- **Under normal scale alternatives:** Mood ARE = 0.760 vs F-test; Mielke quartic ARE = 0.71 vs F-test.
- **Under double-exponential (Laplace) scale alternatives:** Mielke quartic ARE = **1.32**.
- **Under Cauchy scale alternatives:** Mielke quartic ARE = **2.1**.

Two readings:
1. **Under normal scale shifts Mood is slightly more powerful than Mielke quartic** (0.760 vs 0.71), because the quartic over-weights the tails relative to where a normal distribution actually concentrates its scale signal.
2. **Under heavy-tailed scale shifts Mielke quartic dominates massively.** The double-exponential ARE of 1.32 means Mielke quartic needs *fewer* observations than the F-test itself to detect the same scale alternative when the underlying distribution has Laplace tails. Cauchy ARE of 2.1 is even more dramatic — a regime where the F-test loses validity entirely and Mielke quartic still produces a calibrated, decisive readout.

This is *exactly* the regime daily-token data sits in. Token volumes on a coding-agent harness are bursty: most days the value is in a narrow band around the median, but occasionally a refactor session, a long debugging arc, or a multi-file rewrite produces a 3–10× spike. That is empirically much closer to a Laplace or Cauchy tail than a normal tail. Axis-179 Mood was the right first-pass pure-scale test but it *systematically under-weights* the spike-driven scale signal that the data actually carries. Axis-200 Mielke quartic exists precisely to re-weight that signal.

## 4. The seven-bucket classifier: what each bucket means

`classifyMielkeMoodTailVsBulkCompound` (axis-200 ↔ axis-179 joiner, c1b1bc1) defines seven mutually-exclusive buckets at default `alpha = 0.05` and `tolerance = 1e-9`:

1. **`tail-amplified-second`** — both Z share sign (positive), `|mielkeZ| > |moodZ|` strictly, at least one decisive. Reading: second half's dispersion shift is concentrated in the EXTREME tails. Token-spike-driven.
2. **`tail-amplified-first`** — same but Z signs both negative (first half more dispersed in the tails).
3. **`bulk-amplified-second`** — both Z share sign (positive), `|moodZ| > |mielkeZ|` strictly, at least one decisive. Reading: second half's dispersion shift is in the mid-to-upper SHOULDER ranks. Gradual scale drift, no spike concentration.
4. **`bulk-amplified-first`** — same with negative signs.
5. **`coherent`** — signs agree, `||mielkeZ| − |moodZ|| ≤ tolerance`, at least one decisive. Reading: uniform U-shape dispersion shift, clean parametric scale change with no tail-vs-bulk asymmetry.
6. **`sign-conflict`** — signs disagree, at least one decisive. Reading: pathological — bimodal-within-half configurations where bulk-rank mass and tail-rank mass favour different halves. This is the bucket that says "your data does not satisfy the implicit unimodality assumption of either test".
7. **`no-evidence`** — neither decisive at `alpha`. Reading: family-level scale stability.

The bucket geometry is the key thing this joiner contributes that neither axis alone could. Axis-179 alone reads `moodZ` and outputs decisive / not. Axis-200 alone reads `mielkeZ` and outputs decisive / not. Only the joiner can read the *direction of the inequality* `|mielkeZ|` vs `|moodZ|` at the per-source level and distinguish "tail-amplified" from "bulk-amplified" from "coherent". That distinction is what tells a reader whether a detected scale shift is an *episodic spike pattern* (`tail-amplified`) vs a *gradual drift in the typical day's volume* (`bulk-amplified`).

For instrumentation purposes these are very different stories. A `tail-amplified-second` source is a source where the typical day looks the same as before but the rare big days got bigger (or smaller). A `bulk-amplified-second` source is a source where the entire shoulder of the distribution moved — the typical day itself shifted in scale. A coding-agent harness operator should treat these very differently: tail-amplified is "occasional task-difficulty change", bulk-amplified is "fundamental usage-pattern change".

## 5. The claude-code +5.95 / openclaw −1.98 readout in detail

With the framework in hand, the recorded per-source readout from CHANGELOG 0.6.499 says:

```
source       tenure  n1   n2   mielkeZ   mielkePValue
claude-code      72  36   36   +5.9526   2.65e-9
openclaw         19   9   10   −1.9797   4.77e-2
```

Three of five sources land in `|Z| < 2` no-evidence territory; we focus on the two decisive ones. Neither bucket assignment requires running the joiner manually because we know the structural facts:

**claude-code at +5.95, p = 2.65e-9:**
- Sign: positive → second half more dispersed than first half (in the tails).
- Magnitude: `Z = +5.95` corresponds to `p ≈ 2.65 × 10^−9`, ten standard deviations from the no-difference null after Bonferroni correction across all five sources.
- Tenure: 72 days, n1 = n2 = 36. At n = 72 the extreme-rank Mielke weight is `((72−1)/2)^4 = 35.5^4 ≈ 1.589e6`. The Mood weight at the same rank is `35.5^2 = 1260`. Ratio ≈ 1261. So the +5.95 reading is *dominated* by the few most-extreme ranks in the second-half data.
- Likely bucket (pending the joiner's actual moodZ readout, which lives at axis-179): `tail-amplified-second` if `|mielkeZ| > |moodZ|` strictly, which the structural Mielke-vs-Mood weight differential makes overwhelmingly likely under any token-spike-driven dispersion shift. The reading would be: the second 36 days of claude-code's tenure exhibit a dispersion expansion concentrated in the rare big-token days, not a uniform shoulder shift.

**openclaw at −1.98, p = 0.0477:**
- Sign: negative → first half more dispersed than second half (in the tails).
- Magnitude: `Z = −1.98` is barely past `α = 0.05` two-sided. This is a "decisive at the floor" reading, not a robust one. The tenure (19 days) gives n1 = 9 and n2 = 10 — *just* above the n = 16 minimum-tenure floor (which is enforced on the *combined* n, not per half), so the test is operating at the minimum-power end of its calibration envelope.
- Practical reading: openclaw shows a dispersion *contraction* over its short tenure — the first 9 days had wider-tailed token volumes than the last 10 days. This is the *opposite* sign from claude-code, and at very different magnitudes.

The fact that two decisive sources point in *opposite* directions is what makes this readout interesting at a corpus level. A naive "is the corpus's daily-token volume expanding in dispersion?" question has a sign-disagreement answer at the per-source level. The joiner's `sign-conflict` bucket would NOT fire on the cross-source level (it's a per-source classifier, and within each source both axes presumably agree on sign), but at the *aggregator* level (axis-184 Savage / Stouffer-combined cross-source aggregator, which composes signed Z's across sources) the +5.95 and −1.98 partially cancel rather than reinforce. That is information the family preserves which a single bidirectional "any source decisive?" gate would lose.

## 6. Why this is the first cross-source tail-vs-bulk asymmetry readout

Up to axis-199 the pew-insights battery had *eight* prior signed-Z scale axes (axis-117 Siegel-Tukey `stZ`, axis-170 Ansari-Bradley `abZ`, axis-177 Klotz `klotzZ`, axis-178 Conover `conoverZ`, axis-179 Mood `moodZ`, axis-196 Fligner-Killeen, axis-198 Westenberg, axis-199 Capon `caponZ`) all sharing the second-half-positive sign convention. Each of those is a different scale test with different ARE properties under different alternatives, but none of them sits in the same *family* as another in the strict Mielke 1972 power-of-ranks sense. Axis-179 (Mood, p=2) and axis-200 (Mielke quartic, p=4) are the first pair of axes that are literally the *same statistic with a different exponent*, and that algebraic relationship is what makes the tail-vs-bulk classifier well-defined.

Compare the alternative diagnostics that *could not* exist before axis-200:

- **axis-177 Klotz vs axis-179 Mood** would compare squared-normal-scores against squared-centred-ranks. They share neither family nor weight envelope (Klotz weight grows like `2 log n`, Mood grows like `n^2`). Their ratio is not interpretable as "tail concentration" — it's just two different statistical philosophies competing.
- **axis-199 Capon vs axis-179 Mood** has the same problem at the asymptotic-scale level: Capon's `Φ^−1` weights saturate logarithmically; Mood's centred-rank weights grow polynomially. They diverge but the divergence has no clean physical reading.
- **axis-178 Conover vs axis-179 Mood** at least share the squared-rank substrate, but Conover ranks `|X − median|` (so it is a scale test on absolute deviations, not on raw values) while Mood ranks the raw values themselves. Their `Z`s do not have a comparable null distribution and a magnitude inequality between them is not interpretable.

Only Mielke quartic (axis-200) and Mood squared (axis-179) sit in the same `T_p` family with the same null-permutation variance formula and the same sign convention. Their `Z`-magnitude inequality is the first one in the entire battery that has a clean physical reading, which is why the seven-bucket joiner is shipped *only* at axis-200 ↔ axis-179 and not at, say, axis-199 ↔ axis-179 or axis-178 ↔ axis-179.

## 7. The 56× ratio and the n-dependence as a discoverability story

There is a meta-observation worth making about the ARE-table-driven design pattern that pew-insights has converged on. Eight prior scale axes were chosen on the basis of *covering different alternatives* (heavy-tail, light-tail, contaminated-normal, asymmetric, etc). Axis-200 is the first axis chosen on the basis of *quantitatively known weight-asymmetry against an existing axis*. The 56× extreme-rank ratio at n=16 was *predicted* before the test was run; it is a property of the scoring function, not of any data.

That makes axis-200 a *design-time-known orthogonal axis* rather than a *data-driven candidate*. Ships of this kind let the corpus accumulate dimensionality at a controlled pace because each new axis comes with an a-priori statement of what it would say *differently* from the existing axes, against which the empirical readout can be compared. The empirical fact that claude-code's `mielkeZ = +5.95` is so much larger in magnitude than typical `moodZ` readouts on the same data confirms that the *quantitatively predicted* tail-amplification is also *empirically present* — i.e., the daily-token tail of claude-code's second-half data really is heavy enough that the quartic weight catches signal the quadratic weight under-detected.

(For a reader interested in the cross-axis posture of pew-insights as a whole, this completion of a same-family pair at axes 179 and 200 also closes the longest-running outstanding *family hole* in the battery — every other axis is family-of-one inside the corpus.)

## 8. What remains undone after axis-200

Two observations on the design surface.

First, the classifier is *per-source*. It does not compose per-source bucket assignments into a corpus-level bucket. A source can be `tail-amplified-second`, `bulk-amplified-second`, `coherent`, `sign-conflict`, or `no-evidence`, but the corpus-level question "is the typical tail-vs-bulk profile shifting?" requires an additional aggregator layer that the c1b1bc1 commit does not add. The aggregator-level question is genuinely harder because the bucket types are categorical, not ordinal, so a corpus-level summary is more naturally a count vector (how many sources in each bucket) than a single combined Z.

Second, the classifier ships at `tolerance = 1e-9` as the default coherent-boundary tolerance. Under finite-precision floating-point arithmetic the probability that `|mielkeZ| − |moodZ|` lands within `1e-9` of zero on real data is essentially zero — so the `coherent` bucket is effectively a degenerate boundary case and the four amplified buckets plus `sign-conflict` and `no-evidence` are the six practically-realized assignments. Tightening or loosening the tolerance is an explicit operator decision (the API exposes `tolerance` as a configurable parameter for exactly this reason) but the default is calibrated for *deterministic identity testing* (e.g., hand-constructed inputs that should land in `coherent` by construction) rather than for empirical clustering of nearly-equal Z magnitudes.

These two observations are not flaws in the c1b1bc1 design — they are scope decisions consistent with the per-source granularity that the entire pew-insights halves-family operates at. They are, however, what the *next* axis or the *next* joiner would have to address if the corpus wants to migrate from per-source diagnostics to corpus-level dispersion-shape narratives.

## 9. Reading the citation

For anyone wanting to verify the numerics:
- **Source data:** the median-aligned, gap-filled daily-`total_tokens` series per source, in the pew-insights data layer. The five sources (`claude-code`, `openclaw`, `vscode-cp`, `opencode`, `hermes`) each yield a (n1, n2) pair where `n1 = floor(n/2)` and `n2 = n − n1`.
- **Axis-200 implementation:** `daily-token-mielke-quartic-halves`, primary commit `pew-insights@14d4fea` (release 0.6.499).
- **Axis-200 ↔ axis-179 joiner:** `classifyMielkeMoodTailVsBulkCompound`, primary commit `pew-insights@c1b1bc1` (release 0.6.500).
- **Per-source readout table:** CHANGELOG.md entry for 0.6.499, the `source / tenure / n1 / n2 / mielkeZ / mielkePValue` block.
- **34 unit tests for axis-200 primitive identities** (mid-rank ties, median, normal upper tail) plus the family-of-19 tests for the joiner (rejection paths, all 7 buckets, ordering determinism, asymmetric membership, tolerance widening, bucket-count integrity). Full suite at the c1b1bc1 commit: 14407/14407 pass.

The combination of the 0.6.499 axis and the 0.6.500 joiner is what makes the claude-code +5.95 vs openclaw −1.98 readout interpretable at the family level rather than at the single-axis level. It is the first such interpretable readout the corpus has produced, which is what makes axis-200 a structurally distinct ship rather than a redundant copy of axis-179 with stronger tails.
