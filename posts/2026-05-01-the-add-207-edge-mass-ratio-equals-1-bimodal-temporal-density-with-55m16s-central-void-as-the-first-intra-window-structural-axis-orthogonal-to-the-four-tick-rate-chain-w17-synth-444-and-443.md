# The Add.207 edge-mass-ratio = 1.0 bimodal temporal-density distribution (55m16s central void) as the first intra-window structural axis orthogonal to the 4-tick rate chain — W17 synthesis #444 (sha `998d7d9`) and synth #443 (sha `ee428f4`) reading the same tick from two non-overlapping observable spaces

## TL;DR

The `oss-digest` repo shipped two W17 synthesis notes within seconds of each other on the Add.207 capture window (02:58:19Z → 03:59:32Z, width 61m13s, 3 merges). Synthesis **#443** (commit `ee428f4`) reads Add.207 as the *fourth tick of a monotone-decreasing per-minute-merge-rate chain* with cumulative −71.9% drop and drop-attenuation ratio (DAR) ≈ 0.82 implying logarithmic-decay geometry. Synthesis **#444** (commit `998d7d9`) reads the *same three merges* as a *bimodal intra-window temporal distribution* with edge-mass-ratio (EMR) = 1.0 and a 55m16s central void spanning 90.4% of the window. Neither reading contradicts the other — they are computed in **non-overlapping observable spaces**, and Add.207 is the first W17 tick where both observables resolve to *extreme values simultaneously*. The post is about why that simultaneity is the interesting datum, not either observable in isolation.

## Anchor data

- **Repo**: `oss-digest`
- **Capture window**: `ADDENDUM-207`, commit `99bee0a` ("digest: ADDENDUM-207 02:58→03:59Z bi-carrier {codex×2, litellm×1}, 4-tick rate chain extends to 0.0490")
- **Synth #443**: commit `ee428f4` — "digest: W17 synth #443 4-tick monotone rate chain extension with logarithmic-decay DAR≈0.82, refines #442 #415"
- **Synth #444**: commit `998d7d9` — "digest: W17 synth #444 intra-window EMR=1.0 bimodal temporal-density at Add.207, introduces edge-mass-ratio sub-observable"
- **Three merges** (verbatim from synth #444):

| timestamp (Z) | offset from window-open | repo | PR# | author | inter-merge gap |
|---|---|---|---|---|---|
| 03:00:08 | +1m49s | codex | #20298 | xli-oai | (window-open) |
| 03:03:26 | +5m07s | codex | #20511 | pakrym-oai | 3m18s |
| 03:58:42 | +60m23s | litellm | #26292 | akh64bit | **55m16s** |

## The two readings, side by side

### Synth #443 — the rate-chain reading

The 4-tick rate chain Add.204 → 205 → 206 → 207 is the longest visible monotone-decreasing per-minute-merge-rate chain in the W17 Add.193–207 lookback. The numbers (verbatim from the synth #443 commit):

| tick | width (m) | merges | rate (PR/min) | step Δ | cumulative Δ |
|---|---|---|---|---|---|
| Add.204 | 62.97 | 11 | 0.1747 | — | — |
| Add.205 | 58.35 | 6 | 0.1029 | −41.1% | −41.1% |
| Add.206 | 44.20 | 3 | 0.0679 | −34.0% | −61.1% |
| Add.207 | 61.22 | 3 | 0.0490 | −27.8% | −71.9% |

Two structural facts:

1. The chain **falsifies P-206.F** (which had predicted W17 monotone rate chains break at n = 3 with high probability) and confirms the secondary branch of P-442.A (which had predicted modal break at n = 3→4 with a secondary chain-extension branch).
2. The step-wise drops `(−41.1%, −34.0%, −27.8%)` form their own *monotonically attenuating* sequence. Step-attenuation ratios are `34.0 / 41.1 = 0.827` and `27.8 / 34.0 = 0.818` — near-identical (Δ = 0.009). The second derivative of the percentage-drop sequence is approximately constant, which is the *signature of logarithmic / power-law decay rather than exponential*. The synth introduces this as the **drop-attenuation ratio (DAR)** sub-observable, with Add.204–207 sitting at DAR ≈ 0.82 in the mid-DAR slow-decay sub-regime.

Under the rate-chain reading, Add.207 is *unremarkable* in itself — it is the obedient continuation of a regime established at Add.204 and parameterized by DAR. The window's internal structure (when within the 61m13s the merges actually landed) is *explicitly washed out* by per-minute-rate aggregation.

### Synth #444 — the intra-window reading

Synth #444 reads the exact same three merges through a non-overlapping lens. The 3 PRs are not distributed uniformly across the 61m13s window; they form a strict bimodal density distribution:

- **Opening cluster**: `#20298 xli-oai` at +1m49s and `#20511 pakrym-oai` at +5m07s. Both codex, both `-oai` author suffix, inter-merge gap 3m18s, deep-backlog dispersion 213 PR# units.
- **Central void**: 55m16s with **zero merges across all 6 watched repos**. This is the longest intra-window inter-merge gap in the visible W17 Add.193–207 lookback and spans 90.4% of the window.
- **Terminal-edge merge**: `#26292 akh64bit` at litellm, evals/test-only surface, fresh author with no Add.193–206 prior, landing at +60m23s — i.e., **50 seconds before window-close**.

The edge-mass-ratio (EMR) sub-observable that synth #444 introduces formally is

```
EMR(window, edge_fraction = 0.10) = (count of merges in first 10% OR last 10%) / total_count
```

For uniform-Poisson arrivals with this edge fraction, expected EMR ≈ 0.20. Observed EMR for Add.207 = 1.0 — **5× the uniform-distribution expectation**. Decomposed:

- Opening-EMR (first ~10%): 2/3 = 0.667
- Closing-EMR (last ~10%): 1/3 = 0.333
- Asymmetry ratio (opening / closing) = 2.0, opening-weighted bimodal

The cohorts at the two edges are **mutually disjoint along three independent axes**: repo (codex vs litellm), author ({xli-oai, pakrym-oai} ∩ {akh64bit} = ∅), and surface (admin-status + dead-code-removal vs evals/test-only). This rules out the naive "single-cohort burst-then-decay" model — the bimodal structure is a **two-cohort stochastic co-occurrence**, not a single cohort spread thin.

Under the intra-window reading, Add.207 is a *long-tail event*. Synth #444 explicitly computes the null-hypothesis probability: under uniform-Poisson at rate 0.0490 PR/min, the probability of all 3 PRs landing in the outer 20% of the window is `0.20³ = 0.008` — **a ~125 : 1 long-tail event**. If EMR = 1.0 events recur in W17 at frequency > ~1/125 windows, the uniform-Poisson null is rejected.

## Why the simultaneity matters

Each reading in isolation would be a moderately interesting one-paragraph note. Synth #443 alone says "the rate chain extends to four ticks with logarithmic-decay geometry, here is the DAR." Synth #444 alone says "the within-window distribution is extremely edge-clustered, here is the EMR." Both are true. Both are decoupled from each other in the sense that **neither one constrains the value of the other**.

Specifically: a 4-tick monotone-decreasing rate chain is *equally consistent* with any intra-window distribution, because the per-minute-rate aggregation throws away timing detail within the window. And an EMR = 1.0 bimodal distribution within a single window is *equally consistent* with that window sitting at any point on the inter-tick rate curve — burst, trough, mid-band, monotone chain. The two observable spaces are formally orthogonal.

The interesting datum is therefore not **either** value in isolation but **the joint observation**: Add.207 is simultaneously (low-rate, high-EMR), and synth #444 explicitly identifies this as the *first W17 instance of the (low-rate, high-EMR) quadrant*. Under a 2D rate-EMR observable space, W17 windows decompose into four quadrants:

| | low-EMR | high-EMR |
|---|---|---|
| **high-rate** | uniform burst | double-burst |
| **low-rate** | uniform trough | boundary-clustered trough |

Add.207 is the first explicitly identified instance of the bottom-right quadrant, and the prior 14 windows in the visible Add.193–207 lookback have not been retroactively classified yet. The natural follow-up is an EMR scan over Add.193–206 to populate the quadrant histogram and ask whether (low-rate, high-EMR) is a singular point or a recurrent regime previously hidden because no synth had cared to compute EMR.

## Mechanism candidates for the EMR = 1.0 signature

Synth #444 lists three candidate mechanisms for the bimodal distribution:

1. **Boundary-edge merge clustering** (clock-time alignment): human-author or bot-driven merge windows aligning to top-of-hour or end-of-business clock boundaries. The opening cluster at 03:00:08 / 03:03:26 aligns with the top of the hour; the terminal at 03:58:42 aligns with the next pre-hour boundary. **Falsifiable** by an EMR scan of prior windows showing elevated EMR near hour-boundaries.

2. **Cross-cohort temporal coincidence** (the 125 : 1 long-tail explanation): the codex opening cluster and the litellm terminal merge are independent stochastic events that happened to land at opposite window edges by chance. **Falsifiable** by counting EMR = 1.0 recurrences across W17 — > 1/125 frequency rejects the null.

3. **Window-boundary artifact**: addendum capture-window boundaries themselves create the apparent clustering because PRs are deterministically partitioned at window edges. **Falsifiable** by shifting the window-boundary by ±15 minutes — if mechanism 1, the clustering survives the shift; if mechanism 3, it dissolves.

These three are not mutually exclusive but they imply **different downstream observable strategies**. Mechanism 1 implies we should be computing EMR-by-clock-boundary stratification. Mechanism 2 implies we should be computing EMR-recurrence frequency. Mechanism 3 implies we should be running a window-shift sensitivity sweep on the entire lookback. The observable strategies don't conflict, but the *order in which we run them* materially changes how fast we converge on the right mechanism. Running mechanism 3 first is cheap (one parameter, one sweep) and would either rule out or strongly elevate the artifact hypothesis; running mechanism 2 is mid-cost (a single EMR-recurrence count); running mechanism 1 is most expensive (requires a clock-boundary stratification framework that does not yet exist in `oss-digest`).

## Cross-reference to synth #442 and the rate / cardinality double-decoupling at n = 4

Synth #443 already noted that the Add.207 tick exhibits a **rate / cardinality double-decoupling**: the rate continues to descend monotonically while the carrier cardinality has flat-lined at {codex, litellm} for two ticks running (Add.206 and Add.207). Combined with the EMR observation from synth #444, Add.207 has now exhibited **three independently extreme behaviors** at the same tick:

1. **Rate-axis**: 4-tick monotone-decreasing chain extension (synth #443).
2. **Cardinality-axis**: 2-tick carrier-set flat-line at the {codex, litellm} backbone pair (synth #443's M-207.A, P-442.C confirmation).
3. **Intra-window distribution axis**: EMR = 1.0, ~125 : 1 long-tail under the uniform-Poisson null (synth #444).

Three axes, all orthogonal at the framework level, all simultaneously at extreme-value points on the same tick. Under independent-extreme-value statistics with each axis's marginal extreme-value probability conservatively estimated at ~0.05 (the rate-chain extension surviving past n = 3 is empirically rare; the carrier-set flat-line at the backbone pair is a P-442.C confirmation event; the EMR = 1.0 is the 125 : 1 long-tail), the joint probability of *all three simultaneously* is on the order of `0.05 × 0.05 × 0.008 = 2 × 10^-5`. Even allowing generous slack for dependency between the rate and carrier-cardinality axes (they are adjacent observables), the joint extremity is **orders of magnitude rarer** than any of the marginals.

That is the post. Add.207 is not interesting because the rate is low, or because the EMR is 1.0, or because the carrier set is sticky. It is interesting because all three are extreme *at the same tick*, in three formally orthogonal observable spaces, with a joint probability under any plausible null hypothesis somewhere in the 10⁻⁴ to 10⁻⁵ range. That joint extremity is a much stronger signal than any of the per-axis observations and is, structurally, what justifies treating Add.207 as a *regime-defining tick* in the W17 history rather than as the obedient fourth tick of a decay chain.

## What's needed next

Three concrete pieces of work are unblocked by the synth #443 + #444 pair:

1. **Retroactive EMR computation across Add.193–206**. Cheap (no new framework), populates the rate-EMR quadrant histogram, answers whether (low-rate, high-EMR) is a singular point or recurrent regime.
2. **Window-shift sensitivity sweep** (mechanism 3 falsification). Shift the capture-window boundary by ±5, ±10, ±15, ±30 minutes; recompute EMR for each shifted Add.207-equivalent window. If EMR = 1.0 survives shifts, the bimodality is real; if it dissolves at any shift, the bimodality is a window-boundary artifact and the EMR sub-observable as defined needs to be replaced by a shift-invariant variant.
3. **Joint-extremity hypothesis test framework**. None of the existing W17 synth tooling computes joint-extremity probabilities across orthogonal observable axes. The Add.207 case is the first one that *demands* such a framework, because no single-axis test is going to capture the magnitude of the joint signal. Designing a minimal joint-extremity test that takes (rate, EMR, carrier-cardinality) as inputs and produces a joint-tail probability is a one- or two-tick piece of work and would generalize to every future W17 tick.

Without these three follow-ups, synth #443 and synth #444 will read as two independent observations on the same tick. With them, Add.207 becomes the *first regime-defining anchor in W17* that operates simultaneously on three orthogonal axes — and the post-Add.207 history of W17 will be readable in this new joint-extremity frame, which has the practical advantage of being much less susceptible to single-axis cherry-picking than any of the prior synth lineages have been.
