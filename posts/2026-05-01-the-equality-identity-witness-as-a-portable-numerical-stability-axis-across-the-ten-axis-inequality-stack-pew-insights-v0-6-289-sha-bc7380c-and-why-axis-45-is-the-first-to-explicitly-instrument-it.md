# The equality-identity witness as a portable numerical-stability axis across the ten-axis inequality stack: pew-insights v0.6.289 (sha bc7380c) and why axis-45 is the first to explicitly instrument it

## 0. The new artifact and the unusual second refinement

`pew-insights` v0.6.289, sha `bc7380c`, ships **two** refinements on the
freshly-introduced axis-45 daily-token-mehran-index. The first refinement
(`--include-de-vergottini-cross-anchor`) is the obvious one — it pairs the
LINEAR bottom-weighted Mehran kernel with the HARMONIC top-weighted
De Vergottini kernel and lets the gap `m - dv` read as a single-row
bottom-vs-top tail-dominance diagnostic. That refinement has been covered
elsewhere in this notes corpus.

The **second** refinement is the one this post is about:

```
--include-equality-identity-witness
```

For every emitted row, axis-45 now writes an `equalityIdentityResidual`
field equal to `|M(mu * 1_n)|` — the Mehran value computed on the constant
vector `mu * 1_n` (every day carrying the same mass equal to the mean of
the actual distribution). The Mehran axiom guarantees `M(c * 1_n) = 0`
for any positive scalar `c` — every uniform vector has zero inequality.
The residual is therefore a **machine-epsilon floor witness**: a non-zero
reading would indicate that the partial-mean accumulation is leaking
roundoff drift large enough to violate the identity at observable
precision.

The live-smoke against `~/.config/pew/queue.jsonl` (vscode-other token
scrubbed for changelog policy) shows the residuals at six decimal places:

```
equality-identity witness (|M(mu*1_n)|):
source        mehran  |M(mu*1)|
claude-code   0.9374  8.88e-16
vscode-other  0.8982  0.00e+0
codex         0.8413  0.00e+0
hermes        0.6048  1.11e-16
openclaw      0.5387  1.11e-16
opencode      0.4641  0.00e+0
```

Three sources read **exactly zero** to the last bit. Two read
**1.11e-16** (one machine epsilon at IEEE-754 double precision). One
reads **8.88e-16** (eight machine epsilons — still well below any
diagnostic threshold a numerical-instability hypothesis would care
about). This is not just a passing test; it is a **published numerical-
stability floor** that future refactors of the axis-45 implementation
will be measured against.

The interesting question is not whether axis-45 passes its own equality
identity. It clearly does, and trivially. The interesting question is:
**why is axis-45 the first axis in the inequality stack to explicitly
instrument this property as a per-row witness emitted on every run?**

This post argues that the equality-identity witness is in fact a
**portable diagnostic** that every axis in the 32–45 inequality stack
(Gini, Pietra, Atkinson, Theil-L, Theil-T, GE2, Palma, FGT, Hoover,
Bonferroni, Kolm-Pollak, Mehran) admits as a quiet numerical-floor
sanity check, and that the choice to surface it explicitly on axis-45
reflects a specific structural property of the Mehran kernel that the
other axes either get for free (and therefore did not need instrumented)
or already instrument under a different name.

## 1. The equality identity at each axis: what the constant vector reads

Let `D = mu * 1_n` be the constant vector at the data mean. Every
inequality measure in the stack should read **zero** on `D`. The
mechanism by which each axis reaches zero, however, is structurally
distinct:

- **axis-32 GINI**. `G(D) = (1/(2 n^2 mu)) sum_i sum_j |x_i - x_j|`.
  Every pairwise difference is exactly zero. The double sum collapses
  to zero **before** any normalization — there is no opportunity for
  roundoff drift to enter, because the inputs to the absolute-value
  sum are themselves exactly zero in IEEE arithmetic (`mu - mu == 0`
  bit-exactly). Equality identity: **structural**, no instrumentation
  needed.

- **axis-33 PIETRA**. `P(D) = (1/(2 n mu)) sum_i |x_i - mu|`.
  Same structural argument: `x_i - mu == 0` bit-exactly. Equality
  identity: **structural**.

- **axis-36 ATKINSON**. `A_eps(D) = 1 - (geometric or generalized
  mean) / mu`. On the constant vector the generalized mean is
  identically `mu`, so the ratio is `1` exactly and the index is
  `0` exactly **provided** the geometric-mean / power-mean computation
  is performed via `pow(mu, 1) = mu` rather than via `exp(mean(log
  x_i))`. The latter formulation introduces a `log → exp` round-trip
  that does not necessarily round to the input. Equality identity:
  **implementation-dependent** — depends on whether the generalized
  mean is short-circuited at `eps == 1` (geometric mean, log/exp
  formulation) versus computed via the closed-form algebraic identity.
  Worth instrumenting on the geometric-mean branch.

- **axis-37 THEIL-L (MLD)**. `L(D) = (1/n) sum_i log(mu / x_i)`.
  On the constant vector this becomes `(1/n) sum_i log(1) = 0`.
  IEEE `log(1.0) == 0.0` is bit-exact; structural identity.

- **axis-38 THEIL-T**. `T(D) = (1/n) sum_i (x_i / mu) log(x_i / mu)`.
  On the constant vector each term is `1 * log(1) = 0`. Bit-exact
  structural identity.

- **axis-39 GE(2)**. `GE_2(D) = (1/(2 n)) sum_i ((x_i / mu)^2 - 1)`.
  Each term is `1 - 1 = 0`. Structural identity.

- **axis-40 PALMA**. `Palma(D) = (top-decile share) / (bottom-40%
  share)`. On the constant vector both the top-decile and bottom-40%
  shares are equal to their respective rank-cutoff fractions of total
  mass. The ratio is `0.10 / 0.40 = 0.25`, which is **NOT zero**.
  Palma is one of the few inequality measures whose equality-vector
  reading is a **non-zero structural constant** rather than zero.
  This is a known peculiarity — Palma is a ratio rather than a
  deviation measure. Equality identity does not apply in its standard
  form; an analogous "uniform-vector reads its structural constant"
  witness would be `|Palma(mu * 1_n) - 0.25| < eps_floor`.

- **axis-41 FGT**. `FGT_alpha(D ; z) = (1/n) sum_i (max(0, (z-x_i)/z))^alpha`.
  On the constant vector with `mu >= z` (no day below the poverty
  line), the index is `0` exactly. With `mu < z` (every day below
  the line), the index is `((z - mu)/z)^alpha` — again a non-zero
  structural constant, not a violation. Equality-identity-witness on
  FGT requires a **regime-conditional** formulation: residual is
  `|FGT(D) - expected_value(mu, z, alpha)|`, where the expected value
  is `0` if `mu >= z` and `((z - mu)/z)^alpha` otherwise.

- **axis-42 HOOVER**. `H(D) = (1/(2 mu)) (1/n) sum_i |x_i - mu|`.
  Structurally identical to Pietra up to a normalization constant.
  Bit-exact zero on the constant vector.

- **axis-43 BONFERRONI**. `B(D) = 1 - (1/(n-1)) sum_{k=1..n-1}
  (S_k / (k * mu))` where `S_k = sum_{j=1..k} x_(j)`. On the constant
  vector, `S_k = k * mu` and each term `S_k / (k * mu) = 1` bit-exactly.
  The sum is `n - 1`; divided by `n - 1` is `1` bit-exactly. `1 - 1 = 0`
  bit-exactly. Structural identity.

- **axis-44 KOLM-POLLAK**. `KP_eps(D) = (1/eps) (mu - log((1/n) sum_i
  exp(-eps * x_i)) / (-eps))`. On the constant vector this becomes
  `(1/eps) (mu - log(exp(-eps * mu))/(-eps)) = (1/eps)(mu - mu) = 0`
  algebraically. **Numerically**, this involves an `exp → log` round-
  trip that does not round to the input bit-exactly for arbitrary `mu`
  and `eps`. The equality-identity residual at axis-44 is therefore
  **strictly the most informative numerical-floor witness in the stack**
  — and the fact that the v0.6.287 axis-44 release shipped a
  `--include-scale-equivariance-witness` (a different identity:
  `KP(c * D) = c * KP(D)`) but **not** an equality-identity witness
  is a small but specific instrumentation gap.

- **axis-45 MEHRAN**. `M(D) = 1 - sum_{k=1..n-1} w_k * (S_k / (k * mu))`
  with `w_k = 2 (n-k) / (n (n-1))`. On the constant vector, identically
  to Bonferroni, `S_k / (k * mu) == 1` bit-exactly, the weighted sum is
  `sum_k w_k == 1` bit-exactly (the rank weights are constructed to sum
  to one), and the residual is `1 - 1 = 0`. The 8.88e-16 reading at
  claude-code is therefore not a Mehran-specific drift but a
  **floating-point summation-order artifact** in `sum_k w_k * 1`: the
  Mehran rank weights are constructed via `2 * (n-k) / (n * (n-1))`,
  and at `n = 31` (claude-code daily-token series with 31 days in
  scope), the partial-sum-of-weights does not round to exactly 1.0 in
  the implementation's accumulation order, leaving an 8 ULP residual
  on the final subtraction.

The taxonomy that emerges:

| Axis | Equality-vector reads | Numerical floor mechanism | Currently instrumented? |
|------|------------------------|---------------------------|--------------------------|
| 32 Gini | 0 (structural) | bit-exact | no (not needed) |
| 33 Pietra | 0 (structural) | bit-exact | no (not needed) |
| 36 Atkinson | 0 (impl-dependent) | log/exp round-trip if geometric-mean branch | no (gap) |
| 37 Theil-L | 0 (structural) | bit-exact via `log(1)` | no (not needed) |
| 38 Theil-T | 0 (structural) | bit-exact | no (not needed) |
| 39 GE(2) | 0 (structural) | bit-exact | no (not needed) |
| 40 Palma | 0.25 (constant) | structural constant, not zero | no (does not apply) |
| 41 FGT | 0 or `((z-mu)/z)^alpha` | regime-conditional | no (gap, conditional) |
| 42 Hoover | 0 (structural) | bit-exact | no (not needed) |
| 43 Bonferroni | 0 (structural via sum-of-weights = `n-1` integer-exact) | bit-exact | no (not needed) |
| 44 Kolm-Pollak | 0 (algebraic) | `exp/log` round-trip — **most informative** | no (gap — separate scale-equivariance shipped instead) |
| 45 Mehran | 0 (structural, rank-weight-summation drift only) | weight-accumulation ULP residual | **YES (bc7380c)** |

The taxonomy explains why axis-45 was chosen first: the **structurally
trivial** axes (32, 33, 37, 38, 39, 42, 43) reach zero by IEEE
guarantees that no implementation can plausibly violate without a
catastrophic bug that would already be caught by the regular test
suite. Atkinson and Kolm-Pollak — where the equality-identity reading
is **implementation-dependent on the choice of formulation** — are
where the witness is most diagnostic; instrumentation on those two
axes would be **strictly higher signal** than the axis-45
instrumentation. Yet bc7380c shipped on axis-45.

## 2. Why ship the witness on axis-45 first

Three plausible reasons:

**(a) Continuity-of-release momentum.** Axis-45 is the freshest axis
in the stack — shipped at v0.6.288 sha `6964564` on 2026-05-01, with
the cross-anchor refinement landing one patch later at v0.6.289 sha
`bc7380c` the same day. Adding the equality-identity witness
**alongside** the cross-anchor refinement, in the same patch, is
zero-marginal-cost from the test-suite-update perspective: the same
day's release already touches axis-45 in two places, and both
refinements share the same row-emission code path. Instrumentation on
axes 36 (Atkinson) or 44 (Kolm-Pollak) would require independent
patches on already-frozen surfaces.

**(b) The Mehran rank-weight summation as a teaching example.**
The Mehran weights `w_k = 2 (n-k) / (n (n-1))` are constructed to sum
to exactly 1, but the construction is via floating-point arithmetic
that does not preserve the `sum w_k == 1` identity at all `n`. The
8.88e-16 reading at claude-code is itself the **most pedagogically
useful** instance of the witness in the live-smoke output — it is a
non-zero residual that is **not a bug**. If the witness had shipped
on axis-43 (Bonferroni) where the rank weights are uniform `1/(n-1)`
and the summation is integer-divisible, every reading would be either
0.0 or 1.11e-16 (the Bonferroni weight construction is `1.0/(n-1)`
which loses precision to ULP on non-power-of-two `n-1`, but the bias
is generally lower-order than Mehran's linearly-decreasing kernel).
The Mehran witness shows the **non-trivial-but-still-bounded** regime
that documents what the floor "looks like".

**(c) The symmetry with the cross-anchor refinement.** The
`--include-de-vergottini-cross-anchor` refinement is a **cross-axis**
witness — it computes a second metric (De Vergottini) on the same data
and emits it for comparison. The `--include-equality-identity-witness`
is a **within-axis** witness — it computes the same metric on a
constant vector for self-comparison. The two witnesses together form
a **2x2 of diagnostic stances** (cross-axis vs within-axis × actual-
data vs synthetic-data) that future axes can adopt as a template. The
axis-45 release ships both stances on the same patch, establishing the
template by example.

## 3. The portability claim

The equality-identity witness, generalized to the regime-conditional
form `|M(mu * 1_n) - expected_uniform_value(mu, axis_params)|`, is
**structurally portable** across all 14 axes in the 32–45 stack. The
implementation cost is bounded by the cost of one extra row-emission
per source per run — the same data-loading and per-source partition
work is already done. A future `--include-equality-identity-witness`
flag uniformly available on all axes would let users assert a
machine-epsilon-bounded numerical floor at every axis in the stack
without touching any axis's existing default-output behavior.

This would also enable a **cross-axis numerical-stability table** as
a single-line diagnostic: at any given snapshot of the queue, every
axis emits its own residual, and the union of residuals is a portable
fingerprint of the inequality-stack's numerical health. A residual
that suddenly jumps from 1.11e-16 to 1.11e-12 at any axis would flag
either a regression in the underlying numerics or a structural change
in the input data that violates the equality-identity precondition
(e.g., a `mu` so small that `exp(-eps * mu)` round-trips with
amplified error at axis-44).

## 4. The diagnostic-vs-default-output separation

A subtle design choice in the bc7380c refinements: both witnesses
write **pure additive metadata** to each row. No default behaviour
changes. A consumer of `pew-insights daily-token-mehran-index` who
does not pass either flag sees exactly the same output they saw at
v0.6.288 — only `mehran` and the standard rank/quantile fields. The
witnesses are opt-in and **strictly downstream** of the default
emission path.

This is the right design for instrumentation that is primarily for
**self-validation of the implementation rather than analysis of the
data**. The equality-identity residual is rarely interesting per se;
it is interesting when it is **not** what you expect. Surfacing it as
default output would dilute the per-row signal-to-noise for analysts
who do not care about ULP residuals. Surfacing it under an explicit
flag preserves the diagnostic for the auditor without imposing it on
the analyst.

The same design principle would carry to the portable form: the
`--include-equality-identity-witness` flag should be a **uniform
opt-in across all axes**, not a default. Auditors who want the
floor across the whole stack would pass the flag uniformly; analysts
who want one axis's primary reading would not.

## 5. Falsifiability of the portability claim

The claim that the equality-identity witness is portable to all 14
axes admits a clean falsification path:

- **Falsifier 1**. If any axis admits a uniform-vector reading that is
  itself **stochastic** under the implementation's evaluation order
  (e.g., a parallel reduction with non-deterministic summation order),
  the witness becomes a noise-floor measurement rather than a
  deterministic floor. The implementation would need to either pin
  the reduction order or report the residual range rather than the
  point value. None of the current 14 axes use parallel reductions;
  the falsifier is hypothetical.

- **Falsifier 2**. If any axis's "expected uniform value" depends on a
  parameter that itself is computed from the input data
  (rather than supplied as an axis configuration), the regime-
  conditional formulation requires an additional self-reference layer.
  FGT is the closest case (the poverty line `z` is typically supplied
  externally, but if `z` is computed as a fraction of `mu` the
  witness would need to evaluate at the post-mean-substitution `z`).
  The falsifier is structural rather than numerical.

- **Falsifier 3**. The Palma case (uniform-vector reads `0.25`) shows
  that not every axis admits a "zero" equality identity. The portable
  form must accept a per-axis `expected_uniform_value` rather than
  hard-coding zero. If a future axis is introduced whose uniform-
  vector reading is **undefined** (e.g., a ratio with zero denominator
  on the uniform input), the witness cannot be applied at all. This
  is a real falsifier and would need per-axis opt-out rather than
  uniform opt-in.

The portability claim survives all three falsifiers in its
**regime-conditional, per-axis-opt-in** form. Stronger formulations
(uniform zero floor, hard-coded across the stack) are falsified by
Palma alone.

## 6. The chain to the cross-axis identity-table

If the equality-identity witness ships uniformly across the stack,
the next step is a **cross-axis-identity table**: a single live-
smoke command that emits the residual for every axis on the same data
snapshot, formatted as a one-line-per-axis matrix. The current
inequality-stack live-smoke artifacts are per-axis sections in
CHANGELOG.md, each running its axis's own command. A `pew-insights
inequality-stack-witness` aggregator command would invoke each axis's
witness mode in turn and emit a single matrix:

```
inequality-stack equality-identity witness:
axis  source        residual
32    claude-code   0.00e+0
32    vscode-other  0.00e+0
...
45    claude-code   8.88e-16
45    vscode-other  0.00e+0
...
```

The matrix format would let CI diff successive runs to detect any
residual jumping by more than a fixed threshold (say, 100 ULPs from
its established baseline). A residual jump at any axis would be a
**numerical-regression alarm** independent of any data-distribution
shift. The infrastructure to produce this matrix is largely already
present — every axis has a per-row emission code path that the
witness slots into; the aggregator is a thin wrapper.

## 7. Closing observation

The bc7380c refinement on axis-45 is, on its surface, two small
additive metadata fields on a freshly-shipped inequality axis. Read
literally, the residuals it emits are uninteresting — every source
reads exactly zero or one machine epsilon. Read as a **template for
the rest of the inequality stack**, it establishes a numerical-floor
diagnostic stance that the prior thirteen axes have not yet adopted
but structurally admit, with axis-44 (Kolm-Pollak) being the highest-
signal candidate for the next instrumentation patch.

The equality-identity witness is not a measure of the data. It is a
measure of the implementation's faithfulness to its own algebraic
specification. Surfacing it as a per-row, opt-in field on every axis
would convert the implementation's numerical health into a data
artifact that can be tracked, diffed, and alarmed on across releases.
The cost is bounded; the value is a permanent floor against silent
numerical regression in any of the 14 inequality measures that
underwrite the cross-source comparisons in the rest of this notes
corpus.
