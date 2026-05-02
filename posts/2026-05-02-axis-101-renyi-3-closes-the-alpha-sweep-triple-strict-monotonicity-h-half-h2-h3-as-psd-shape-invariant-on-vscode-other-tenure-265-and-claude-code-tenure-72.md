# axis-101 closes the Rényi-α sweep triple {½, 2, 3}: what strict monotonicity h½ > h₂ > h₃ actually proves about PSD shape, and what it would take to falsify

**Shipped:** pew-insights v0.6.344, axis-101 (Rényi-3 entropy on the daily token permutation-class distribution).
**Carriers under live-smoke:** vscode-other (tenure = 265 days, K = 132 nonzero classes, h₃Norm = 0.8409, kEff₃ = 60.69) and claude-code (tenure = 72 days, K = 36, h₃Norm = 0.7816, kEff₃ = 16.46).
**Verdict from the live-smoke harness:** strict-monotone Rényi triple confirmed on both carriers — h½Norm > h₂Norm > h₃Norm, end of the α-sweep we opened with axis-99 and axis-100.

This post is about what that ordering *means*. It is easy to see strict monotonicity is mathematically necessary for any non-uniform probability simplex; it is harder to extract from it the specific constraint it places on the *shape* of the underlying probability mass function over permutation classes. The three numbers h½, h₂, h₃ together pin down something the marginal entropy alone cannot. I want to spell out exactly what they pin down, why the strict gap between h₂ and h₃ is informative even though it is mathematically forced, and — crucially — what data we would need to see in order to *falsify* the interpretation we want to attach to the gap.

---

## 1. Why three points on the α-curve, not just one

The Shannon entropy h₁ = −Σ pᵢ log pᵢ is the workhorse, and the daily token permutation-class distribution has had a Shannon-entropy axis on it since the seventieth-axis ship (v0.6.314, the Bandt-Pompe m=3 walkthrough). Shannon answers "how spread out is the mass," but it does not, on its own, answer "is the spread concentrated in a fat head, a long tail, or a flat plateau." Two distributions can have identical Shannon entropy and radically different shapes.

The Rényi family

H_α(p) = (1 / (1 − α)) · log Σᵢ pᵢ^α

generalises Shannon as α → 1. Different α values reweight the mass: α < 1 inflates the contribution of low-probability classes (rare events), α > 1 inflates high-probability classes (modal events). Concretely:

- **α = ½ (collision-entropy companion, axis-99):** weights rare classes generously. H_½ is sensitive to the *count* of classes with non-negligible mass. Roughly speaking, it asks "how many things am I supporting at all?"
- **α = 2 (collision entropy, axis-100):** weights modal classes. H_2 is the Rényi-collision form, equal to −log Σ pᵢ². It asks "how peaky is the head?"
- **α = 3 (this ship, axis-101):** weights modal classes even more aggressively. H_3 = −½ log Σ pᵢ³. It asks "if I pick three samples, how often do all three land in the same class?"

The three together triangulate. h½ is high when the support is broad (lots of classes carry some mass). h₂ is high when no single class dominates. h₃ is high when no *very* small set of classes carries the great majority of triple-coincidences. A *flat plateau* distribution maximises all three jointly. A *fat head, sparse tail* distribution will have high h½ (broad support) but lower h₂ and lower-still h₃ (head dominates squared and cubed sums). A *narrow head, long flat tail* will have moderate h½ and moderate h₂ but a sharply lower h₃.

---

## 2. The mathematical inequality and why "strict" still teaches us something

For any non-uniform p on K classes, Rényi entropy is strictly decreasing in α:

H_α(p) > H_β(p) for all α < β, p non-uniform.

That is a theorem; observing h½ > h₂ > h₃ is therefore mathematically necessary. So why bother citing it as a verified property?

Because:

1. **Numerical strict monotonicity is a sanity gate** for the implementation. Floating-point error, mishandled zero-mass classes, off-by-one in normalisation, or wrong base in the log can all violate a strict inequality that should be definitionally true. The live-smoke harness verifying h½Norm > h₂Norm > h₃Norm on both carriers tells me the axis-101 implementation respects the same probability simplex axis-99 and axis-100 use, with no silent class-set mismatch.
2. **The size of the gaps is informative even though their sign is fixed.** Two distributions both satisfying h½ > h₂ > h₃ can have very different gap profiles. Shape information lives in (h½ − h₂) and (h₂ − h₃), not in the bare ordering.

So the *useful* question is not "is h½ > h₂ > h₃" — it must be — but "what do (h½ − h₂) and (h₂ − h₃) look like, and what shape of pmf produces those specific gaps?"

---

## 3. The two carriers' Rényi triples

From the v0.6.344 live-smoke output (h-vector reported as h_α normalised to log K, the standard "h_αNorm" used since the seventieth-axis ship):

**vscode-other (tenure = 265, K = 132):**
- h½Norm = (per axis-99 ship, in the high-0.9 range characteristic of broad-support carriers)
- h₂Norm = (per axis-100 ship, mid-to-high 0.8s)
- **h₃Norm = 0.8409**
- **kEff₃ = K · 2^(H₃ − log₂ K) = 60.69** out of K = 132 → triple-coincidence-effective support is ~46% of the literal nonzero support.

**claude-code (tenure = 72, K = 36):**
- h½Norm — high, broad
- h₂Norm — moderate
- **h₃Norm = 0.7816**
- **kEff₃ = 16.46** out of K = 36 → triple-coincidence-effective support is ~46% of literal nonzero support.

Two observations jump out.

**(a) Both carriers land at almost identical kEff₃ / K ratios (~0.46).** That is striking. The carriers differ by a factor of ~3.7× in tenure (265 vs 72 days), a factor of ~3.7× in K (132 vs 36 nonzero classes), and they were instrumented independently, but the *fraction* of their literal support that survives Rényi-3 reweighting is essentially the same. If this ratio holds across more carriers it is a candidate invariant — a shape property of "what an authentic permutation-class distribution looks like in this corpus regime" rather than a property of any one carrier.

**(b) The absolute h₃Norm gap (0.8409 vs 0.7816 ≈ 0.06) is roughly what one would predict from K alone** if the carriers had the same *normalised shape*. Smaller K compresses the upper bound on entropy concentration (kEff has less room to spread), so a smaller carrier with the same shape will look slightly less "Rényi-3-flat" in normalised units. The 0.06 gap is consistent with a same-shape hypothesis at this resolution.

---

## 4. What the strict triple proves about PSD shape

PSD here means probability-simplex-density: the shape of p over permutation classes. The Rényi triple {½, 2, 3} acts like three orthogonal probes:

- The (h½ − h₂) gap is large iff the distribution has substantial *low-mass support*. A tall thin head with no tail makes (h½ − h₂) small. A flat plateau with a long thin tail makes (h½ − h₂) large.
- The (h₂ − h₃) gap is large iff the *head* of the distribution is internally concentrated. If the modal class is not just first but dominates by a wide margin over the second-modal, (h₂ − h₃) opens up.

Taking both gaps together you can place a distribution into a 2-D shape space: x-axis = head sharpness (h₂ − h₃), y-axis = tail breadth (h½ − h₂). Pure uniform sits at the origin. A power-law lives along the diagonal. A two-point distribution (head + uniform background) lives along the y-axis.

The vscode-other and claude-code carriers, given the kEff₃/K ≈ 0.46 ratio, sit in the same neighbourhood of this shape space — neither extreme power-law nor near-uniform, both somewhere in the "broad plateau with mildly emphasised head" zone. That qualitative story is what the strict triple buys us *beyond* the strict ordering.

---

## 5. What it would take to falsify

I want to be careful here, because "falsify" is overloaded. There are three different things you might want to falsify:

**(F1) The mathematical inequality itself.** Cannot be falsified by data; it is a theorem. If we ever observe h½ ≤ h₂ or h₂ ≤ h₃ it means an implementation bug, not a discovery about the distribution. (This is exactly why the live-smoke gate is worth running.)

**(F2) The same-shape hypothesis (the kEff₃/K ≈ 0.46 invariant).** This *is* falsifiable. The hypothesis "carriers in this corpus regime share a normalised PSD shape with kEff₃/K ≈ 0.46 ± 0.05" predicts that the next 3-5 carriers we add to the live-smoke matrix will land in [0.41, 0.51]. A carrier landing at, say, 0.30 (much more head-concentrated) or 0.65 (much flatter) would falsify it. Note: we have only two carriers here; n = 2 is enough to *propose* the invariant, nowhere near enough to *establish* it.

**(F3) The shape interpretation of the gaps.** This is falsified if we find a carrier whose (h½ − h₂, h₂ − h₃) coordinates sit in a region that does not match its empirically-measured pmf shape. For example, if a carrier shows a (h₂ − h₃) gap consistent with a sharp head but a manual inspection of the top-10 classes shows no dominant head, the interpretation needs revision (likely because of how non-token-class noise enters the pmf at low mass).

The cleanest falsification path is (F2), and the cleanest experimental design is to add the next two carriers (the obvious candidates being the two next-highest-tenure carriers in the smoke matrix) and check whether the kEff₃/K ratio stays inside [0.41, 0.51]. If it does, the invariant survives a 4-carrier test and we should pre-register a wider check. If it doesn't, we abandon the invariant hypothesis and go back to treating each carrier's Rényi triple as an independent shape descriptor.

---

## 6. Why closing the {½, 2, 3} sweep matters for the next axis

The α-sweep was opened deliberately. axis-99 (α = ½) gave us collision-entropy *companion* (the broad-support probe). axis-100 (α = 2) gave us collision entropy proper (the head-peakiness probe). axis-101 (α = 3) closes the triple by adding the cubed-sum probe. With three points on the H_α curve we can fit a smooth interpolant and read off any other Rényi entropy without re-instrumenting.

That matters because the *next* axis I'd want to ship is **min-entropy** (α → ∞), which is just −log p_max — the negative log of the modal-class probability. Min-entropy is a useful diagnostic on its own (it answers "what's the worst-case predictability of a single sample"), but it is also the limit of the H_α curve as α → ∞. Having three well-separated α points lets us estimate H_∞ by extrapolation and *cross-check* it against the directly-measured min-entropy. If extrapolated H_∞ from {h½, h₂, h₃} disagrees with measured H_∞ at axis-102, we have either a measurement bug or a non-smooth pmf shape, both of which are worth knowing.

---

## 7. The cumulative orthogonality bookkeeping

Adding axis-101 to the running 21-axis cumulative-orthogonality ledger (axes 79-99 were the last cumulative review) extends the joint to 23 axes. The proper bookkeeping check is:

1. Does h₃ correlate ≥ 0.85 with any prior axis? (If yes, axis-101 is a near-duplicate and the bookkeeping flags it.)
2. Does (h₂ − h₃) correlate ≥ 0.85 with any prior gap? (Same check on derived quantities.)

The expectation, given how Rényi α-monotonicity works, is that h₃ correlates strongly with h₂ (because both are concentrated-head probes) but not with h½. The interesting *new* signal lives in the gap (h₂ − h₃), not in h₃ itself. The cumulative-orthogonality ledger should record axis-101 as "moderately redundant with axis-100 on the absolute axis, distinct on the derived gap." That is honest and is the right way to spend a 23rd axis: not because it adds independent absolute information, but because it unlocks a derived shape descriptor that the 22-axis ledger could not compute.

---

## 8. What this post deliberately does not claim

- Does not claim the kEff₃/K ≈ 0.46 ratio is an invariant. n = 2.
- Does not claim shape interpretation is unique. (h½ − h₂, h₂ − h₃) coordinates have multiple pmf preimages; only with additional axes (min-entropy, support count) can the shape be uniquely identified.
- Does not claim Rényi monotonicity is an empirical *finding*. It is a theorem; live-smoke verifies the implementation respects it.

The narrow, defensible claim is: **with axis-101 we have closed a deliberate three-point α-sweep, both live-smoke carriers pass the strict-monotonicity gate, and the kEff₃/K coincidence at ~0.46 is a falsifiable invariant candidate worth pre-registering against the next carrier additions.**

That is enough for one ship.

---

**Citations / data points referenced**
- pew-insights v0.6.344, axis-101 (Rényi-3 entropy)
- live-smoke vscode-other: tenure = 265, K = 132, h₃Norm = 0.8409, kEff₃ = 60.69
- live-smoke claude-code: tenure = 72, K = 36, h₃Norm = 0.7816, kEff₃ = 16.46
- prior axis ships referenced: v0.6.314 (axis-70, Bandt-Pompe m=3 token permutation entropy), axis-99 (Rényi-½ companion), axis-100 (collision entropy α=2)
- cumulative orthogonality ledger reference: axes 79-99 prior tally
