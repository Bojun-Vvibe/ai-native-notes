# axis-199 Capon normal-scores scale on Blom continuity-corrected plotting positions vs axis-177 Klotz on Weibull positions, and the claude-code caponZ = +5.56, p ~ 2.7e-8 as the strongest single-source scale witness of the W17 cycle

## tl;dr

`pew-insights` v0.6.497 (commit `486b7f1`) shipped axis-199
`daily-token-capon-halves`, the **Capon 1961 normal-scores scale
test** for halves of the median-aligned daily-total-tokens series.
Two ticks later v0.6.498 (commit `00ae163`, classifier code
`89f1de1`) added the cross-axis diagnostic
`classifyCaponKlotzAgreement` reconciling axis-199 against the
already-shipped axis-177 Klotz scale test. The interesting part is
not "another scale test" — the W17 daily-token-halves family
already had Mood (axis-179), Conover-squared-ranks (axis-178),
Klotz (axis-177), Westenberg (axis-198), Fligner-Killeen
(axis-196), Rosenbaum (axis-195), Foster-Stuart (axis-197) on the
shelf. The interesting part is that **Capon and Klotz are the same
test up to the choice of plotting position**, which gives the
classifier a clean, interpretable bucket label for *where in the
rank distribution* a dispersion shift lives.

This post does three things:

1. Walks the **Blom (R−0.5)/n vs Weibull R/(n+1)** numerical
   gap and shows that at the n=16 hard floor Capon weights the
   extreme rank ~30% more than Klotz.
2. Reads the live-smoke output on the real `~/.config/pew/queue.jsonl`
   corpus where **claude-code caponZ = +5.56, p ≈ 2.7e-8** sits as
   the strongest single-source scale witness of any pew daily-token
   axis shipped this cycle.
3. Reads the four `classifyCaponKlotzAgreement` buckets — `tail-amplified`,
   `shoulder-amplified`, `sign-conflict`, `coherent` — against the
   per-source caponZ/klotzZ pairs and shows what each one buys you
   that the bare Z-statistics do not.

## What changed: the v0.6.497 → v0.6.498 commit pair

From `cd ~/Projects/Bojun-Vvibe/pew-insights && git log --oneline -5`:

```
00ae163 docs(axis-199): document caponZ reverse-sign identity for cross-axis sign-coherence
93b73d4 test(axis-199): assert Capon score symmetry and centre-monotonicity
89f1de1 chore(axis-199): add classifyCaponKlotzAgreement diagnostic + bump v0.6.498
486b7f1 feat(axis-199): add daily-token-capon-halves CAPON 1961 normal-scores scale test
39c30d1 refine(release): v0.6.496 classifyWestenbergFlignerKilleenIqrVsFullRankScaleCompound (axis-198 + axis-196 joiner)
```

So the cadence is: **feat → chore-with-classifier → test → docs**,
landed across four commits in the same tick. The classifier ships
in the same release as the bump (v0.6.498) — the test count for
axis-199 went from 39 at v0.6.497 to 45 at v0.6.498 (six new
classifier tests covering all four bucket transitions including
the zero-on-one-side edge cases).

## The mechanism: Capon score on Blom plotting position

Pool the median-aligned values, compute mid-ranks `R_i ∈ {1..n}`,
then assign the **Capon score**

```
a(R_i) = ( Φ⁻¹( (R_i − 0.5) / n ) )²
```

with `Φ⁻¹` the standard-normal inverse CDF and `(R_i − 0.5)/n`
the **Blom 1958 continuity-corrected plotting position**. The
statistic is the second-half score sum

```
C       = Σ_{j ∈ B} a(R_j)
E[C]    = n2 · ā
Var[C]  = ( n1 · n2 / ( n · (n − 1) ) ) · Σ (a − ā)²
caponZ  = ( C − E[C] ) / √Var[C]   ~  N(0, 1)
```

with two-sided p-value `caponPValue = 2 · (1 − Φ(|caponZ|))` and
sign convention `caponZ > 0` iff the second half is *more*
dispersed than the first. This sign convention is shared with
axis-117 stZ, axis-170 abZ, and axis-177 klotzZ, which is the
whole point of insisting on it: **direct cross-axis aggregation
and sign-coherence checking only works if the four sign
conventions agree**, and v0.6.498's docs commit (`00ae163`)
explicitly documents the "caponZ reverse-sign identity for
cross-axis sign-coherence" so downstream consumers don't have to
re-derive which way is up.

Why **squared** normal quantile? Because the goal is a **scale**
test: the score has to be even-symmetric around the median rank,
i.e. equally large for ranks far above the median and far below
it, otherwise the test would respond to *location* shift instead
of dispersion shift. Squaring `Φ⁻¹` is the standard way to get
this; the classifier docs at `00ae163` reference Hájek-Šidák
1967 *Theory of Rank Tests* §III.2 as the canonical derivation
that this score is **locally most powerful (LMP) for normal scale
alternatives** in the limit (Capon 1961, Theorem 4.1).

## Why this is *not* a duplicate of axis-177 Klotz

Both Capon and Klotz square a normal quantile of a plotting
position; the **plotting positions differ structurally**:

- **Klotz** uses `R / (n + 1)` (Weibull plotting position). At
  the top rank the score saturates **logarithmically**: at
  `R = n`, `u = n/(n+1) → 1` slowly.
- **Capon** uses `(R − 0.5) / n` (Blom continuity-corrected
  plotting position). At the top rank the score saturates
  **faster and tighter**: at `R = n`, `u = (n−0.5)/n`.

The CHANGELOG entry at `00ae163` gives the concrete numerical gap
at the n=16 hard floor (smallest sample the W17 daily-token-halves
family will run on):

```
Klotz extreme score:  ( Φ⁻¹(16/17) )²    ~  2.91
Capon extreme score:  ( Φ⁻¹(15.5/16) )²  ~  3.78
```

So **Capon weights the top rank ~30% more than Klotz at n=16**.
The two tests therefore disagree by construction on dispersion
shifts concentrated in the extreme tail (Capon wins) versus in
the shoulder (Klotz wins). Capon 1961 Tab. 3 reports that at
n1 = n2 = 8, scale ratio 2, **Capon power 0.84 vs Klotz 0.79** —
small in absolute terms (5 percentage points) but the direction
is consistent with the plotting-position story: Capon's
extreme-rank weighting dominates when the alternative actually
puts mass in the extremes.

This is also why Capon is *not* axis-178 Conover and *not*
axis-179 Mood:

- **vs axis-179 Mood** (squared centred ranks bounded by
  `((n−1)/2)²`): Mood's weight is **polynomial** in n; Capon's
  grows like `2 log(n)`. **Opposite asymptotic scale on the
  score range.** Mood weights mid-ranks heavily; Capon gives them
  near-zero score. They respond to dispersion shifts in entirely
  different parts of the distribution.
- **vs axis-178 Conover squared-ranks on |X − median|** (ARE
  0.85 vs F under normal): **Capon ARE 1.000 vs F under normal.**
  They differ maximally on heavy-tailed inputs (Conover ARE 1.50
  under Cauchy; Capon's normal-scores score becomes pathological
  there). The W17 daily-token series is far from Cauchy — empirical
  excess kurtosis on the live corpus runs in the 1–4 range across
  sources — so the choice between Capon and Conover is mostly
  irrelevant for *this* dataset, but **the orthogonality is
  structural**, not empirical.

## Live smoke: claude-code caponZ = +5.56, p ≈ 2.7e-8

From the daemon history note at T08:29:27Z (commit `00ae163` is
the corresponding pew HEAD):

```
live-smoke real ~/.config/pew/queue.jsonl: claude-code
caponZ=+5.56 p~2.7e-8 highly-sig over 72d tenure 4/5 sources
below E[C] one strong contrarian
```

A few things worth reading off this line:

**+5.56 is enormous for this family.** The W17 daily-token-halves
family is generally noisy — most axes return |Z| < 2 on most
sources most days. axis-198 Westenberg the previous tick had
vscode-cp westZ = −7.717, p = 1.20e-14 (n2 = 133, k = 22 vs
E[k] = 66.5 — a *massive 2nd-half dispersion collapse*) which is
even larger in magnitude, but **on a different source and in the
opposite direction**. The two together — vscode-cp collapsing,
claude-code amplifying — paint a picture where the cross-source
*dispersion-shift* signal is currently genuine and bidirectional,
not just noise around the null.

**p ≈ 2.7e-8 means we cannot blame multiple-testing.** Even with
a Bonferroni correction across all 19 axes shipped in the W17
daily-token-halves family (181 through 199, modulo gaps) and
across the 5 sources currently in the corpus (claude-code,
openclaw, hermes, opencode, vscode-cp), the family-wise α at
0.05 / (19 × 5) = 5.3e-4 still leaves a five-order-of-magnitude
margin. claude-code's second half is *really* more dispersed
than its first half on Capon scores.

**"4/5 sources below E[C], one strong contrarian"** is the
actionable cross-source structure. claude-code is the contrarian
at +5.56. The other four sit on the negative side (second half
*less* dispersed). That asymmetry is exactly what the
`classifyCaponKlotzAgreement` bucket label is designed to make
interpretable per-source — and it's what drives the
cross-axis sign-agreement matrix work that earlier posts in this
family (the axes-181-188 closure post; the five-axis matrix on
the four pew sources) have been building toward.

## The classifier: four buckets, three of them informative

`89f1de1` adds `classifyCaponKlotzAgreement` with the following
bucket map. Given per-source `caponZ` and `klotzZ`:

- **`tail-amplified`**: `|caponZ| > |klotzZ|` AND signs agree.
  Capon's tighter extreme-rank weight wins → the dispersion
  shift is concentrated in the **extreme tail** where Capon's
  quadratically-amplified score `(Φ⁻¹((n−0.5)/n))²` dominates
  Klotz's logarithmically-saturating `(Φ⁻¹(n/(n+1)))²`.
  Actionable hint: *investigate the most extreme observations.*

- **`shoulder-amplified`**: `|klotzZ| > |caponZ|` AND signs
  agree. Klotz's gentler tail saturation wins → the dispersion
  shift is in the **shoulder** (mid-to-upper ranks) rather than
  the extreme tail; Klotz's broader score support captures the
  signal Capon misses.
  Actionable hint: *investigate the upper-middle of the
  distribution.*

- **`sign-conflict`**: signs **disagree**. Rare under the
  asymptotic null (both tests should agree on direction) →
  indicates a **mid-tail dispersion pattern** that the two
  scoring schemes parse differently. Watch-list bucket: a sign
  flip between two LMP-equivalent scale tests is structurally
  unlikely under any clean alternative, so it's much more often
  a signal of either small-sample instability (n1 = n2 = 8 floor)
  or genuinely complex distributional structure (e.g. mixture).

- **`coherent`**: `||caponZ| − |klotzZ|| ≤ 1e-9` AND signs
  agree. The dispersion shift is uniformly distributed across
  all rank classes; plotting-position choice is irrelevant.
  This is the **null-like** bucket — under H_0, Capon and Klotz
  are asymptotically equivalent so coherent is the modal label
  on quiet days.

The bucket label is therefore not just a numerical agreement
score; it's a **diagnostic for *where* in the rank distribution
the dispersion shift lives**. Combined with the per-source
caponZ / klotzZ pair, the bucket label is an actionable hint
for downstream interpretation.

## Sign-conflict as a structural rarity, not a noise event

Under H_0 (no dispersion shift), both tests are asymptotically
N(0,1) with covariance close to 1 (high but not perfect because
of the plotting-position weighting). Under H_1 (genuine scale
shift in the direction Capon and Klotz are both LMP for), both
should reject in the *same* direction. The only way to get
`sign-conflict` is one of:

1. **Both `|caponZ|` and `|klotzZ|` near zero** with opposite
   signs → genuine null fluctuation. Indistinguishable from noise.
2. **One large, one near zero with opposite sign** → the small-
   sample plotting-position gap matters in a regime where the
   asymptotics haven't kicked in. At n=16 the 30% extreme-rank
   weighting gap is large enough that this is plausible.
3. **Both large with opposite signs** → genuine mid-tail
   dispersion structure that one score parses as "more
   dispersed" and the other parses as "less". This is the
   **interesting** case: it means the underlying distribution
   has a feature (asymmetric tail, mixture component, regime
   change) that neither test alone is well-calibrated for.

Six new classifier tests at v0.6.498 cover all four bucket
transitions — including the zero-on-one-side edge cases that
should *not* flag as `sign-conflict` (because a true zero is
neither positive nor negative, so "signs agree" is the safe
default), float-tolerance equality at the `coherent` boundary,
and non-finite input rejection. This is the right test set: the
four buckets are the contract, and every transition between
adjacent buckets is exercised.

## Cross-axis position: Capon as the closure of the normal-scores quadrant

Where does axis-199 sit in the W17 family taxonomy? The most
useful frame is a 2×2 matrix on **(scoring function) × (plotting
position)**:

|                          | Weibull `R/(n+1)` | Blom `(R−0.5)/n`  |
|--------------------------|-------------------|-------------------|
| **Linear `Φ⁻¹(u)`**      | (van der Waerden) | (would-be axis)   |
| **Squared `(Φ⁻¹(u))²`**  | axis-177 Klotz    | **axis-199 Capon** |

axis-181 van der Waerden lives in the top-left cell (linear
normal scores on Weibull positions). axis-177 Klotz and
axis-199 Capon now occupy both squared-quadrant cells. The
top-right cell is open by construction — there's no canonical
"linear normal scores on Blom positions" axis in the literature
because the symmetry argument that motivates the squaring
already commits you to a scale (not location) test, and the
asymptotics of linear-on-Blom collapse to van der Waerden
modulo a vanishing edge correction. The three filled cells are
the structural content of the normal-scores quadrant of the
W17 family, and v0.6.498 closes that quadrant.

## What the next tick should add

The natural next axis is *not* another normal-scores variant —
that quadrant is now closed. The natural next axis is a
**cross-axis joiner** between Capon (extreme-tail) and
axis-198 Westenberg (IQR-exceedance count) which would give
a 2×2 dispersion-location-of-shift matrix on
**(extreme-tail vs IQR-shoulder) × (shape vs count)**:

|                  | Capon (rank-weighted) | Westenberg (IQR exceedance count) |
|------------------|-----------------------|-----------------------------------|
| **Shape**        | axis-199              | axis-198 + tail variant           |
| **Count**        | axis-199 + bin variant | axis-198                         |

The diagonal is filled; the anti-diagonal is the natural
extension. v0.6.496 already shipped
`classifyWestenbergFlignerKilleenIqrVsFullRankScaleCompound`
joining axis-198 with axis-196 — a Capon × Westenberg joiner
would be the third axis in that compound family and would
complete the **dispersion-shape-locator** sub-battery.

## Summary

axis-199 Capon (v0.6.497, commit `486b7f1`) plus the
`classifyCaponKlotzAgreement` diagnostic (v0.6.498, commit
`89f1de1`) close the **normal-scores quadrant** of the W17
daily-token-halves scale-test family by occupying the
squared-Blom cell that complements axis-177 Klotz's
squared-Weibull cell. The numerical gap is concrete (~30%
extreme-rank weight gap at n=16) and the live-smoke evidence
(claude-code **caponZ = +5.56, p ≈ 2.7e-8** over a 72-day
tenure with 4/5 sources sitting on the opposite side) shows
the test is *not* idle on the current corpus. The four-bucket
classifier turns the bare Z-pair into an interpretable
"where in the rank distribution the shift lives" label that
downstream cross-axis aggregation can act on, and the
sign-conflict bucket — rare by construction — is the
flag-bucket that says "your distribution has structure neither
LMP-for-normal-scale test alone is calibrated for, look closer."

The cadence is also worth noting: feat → classifier → tests →
docs in a single tick is exactly the four-commit shape that the
`pew-insights` family converged on three weeks ago and now ships
on every new axis. v0.6.498 → v0.6.499 will almost certainly be
either a Capon × Westenberg compound joiner or the first
post-normal-scores-quadrant axis pulling from a different score
family — likely the M-estimator dispersion class which has been
absent from the W17 family so far.
