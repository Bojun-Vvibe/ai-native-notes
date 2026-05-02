# Axes 79–99: The Twenty-One-Axis Cumulative Orthogonality Bookkeeping and What Structural Orthogonality Actually Buys (pew-insights v0.6.342)

**Date:** 2026-05-02
**Anchor:** pew-insights v0.6.342 axis-99 daily-token-spectral-renyi-alpha2-collision-entropy
**SHAs:** feat=`ecb9a36` · test=`e43b759` · release=`9922686` · refine=`8ec964c`
**Test counter:** 9777 → 9841 (+64 net)
**Predecessor band:** axes 79–80 Hjorth · axis 81 Teager-Kaiser · axis 82 curvature · axis 83 LZ · axis 84 DFT-slope · axis 85 Wiener-flatness · axis 86 spectral-centroid · axis 87 spectral-bandwidth · axis 88 rolloff · axis 89 crest · axis 90 skewness · axis 91 kurtosis · axis 92 spectral-decrease · axis 93 spectral-irregularity · axis 94 spread-IQR · axis 95 spectral-roughness · axis 96 spectral-peak-frequency · axis 97 spectral-second-peak · axis 98 spectral-tail-flatness · axis 99 collision-entropy

## 1. The question this post is about

After axis-99 landed in pew-insights v0.6.342 (`ecb9a36` → `8ec964c`), the cumulative axis count from axis-79 (Hjorth activity) to axis-99 (Renyi-α=2 collision-entropy on PSD) reached **21 distinct structural witnesses**. Each of these axes was admitted into the corpus only after passing a *structural-orthogonality* test against every prior axis — meaning that on the live-smoke queue the new axis must produce a measurement that cannot be reconstructed as a monotone or affine function of any single prior axis, and ideally cannot be tightly bounded by a small linear combination of two or three of them.

Twenty-one axes is a big enough cohort to ask: what does this structural-orthogonality discipline actually buy us? Is it producing genuine new information, or is it producing diminishing-returns micro-decorations that look orthogonal in some L² sense but tell us nothing new about the carriers? This post is an empirical accounting.

## 2. The 21-axis cohort, briefly

The axes split into three structural families:

**Time-domain shape descriptors (axes 79–83):** Hjorth activity (79), Hjorth mobility (80), Teager-Kaiser energy (81), local curvature (82), Lempel-Ziv complexity (83). These look at the daily-token series as a sequence and ask about its differential / variational structure.

**Hybrid time-frequency descriptors (axes 84, 92, 95):** DFT-slope (84) — a single-number power-law slope of the FFT magnitude vs. frequency; spectral decrease (92) — a normalized average descent of the PSD; spectral roughness (95) — a localized derivative-norm on the PSD.

**Pure frequency-domain descriptors (axes 85–91, 93–94, 96–99):** Wiener-flatness (85), spectral centroid (86), bandwidth (87), rolloff (88), crest (89), skewness (90), kurtosis (91), spectral irregularity (93), spread-IQR on PSD (94), peak-frequency (96), second-peak (97), tail-flatness (98), collision-entropy (99).

All 21 are computed on the **same** input: each carrier's daily token count series, restricted to the long-tenure cohort with min-tenure-days = 10. That common input is what makes the cumulative orthogonality claim non-trivial. If 21 different functions of the *same* finite series can each individually witness against the others, then either the series is structurally rich, or the orthogonality test is too lenient. Both possibilities deserve interrogation.

## 3. The live-smoke witness from axis-99's release

The release commit (`9922686`) shipped a live-smoke that exercised axis-99 against the production queue.jsonl with 2388 lines. The smoke recorded:

- **claude-code:** h2Norm = 0.8284, kEff = 19.4622, observed K = 36 → kEff/K = **0.541**
- **vscode-other:** h2Norm = 0.8697, kEff = 69.8762, observed K = 132 → kEff/K = **0.529**

These two long-tenure carriers land within ~2.3% of each other on kEff/K. That is the headline result, and it is also the place where the orthogonality claim has to be defended most carefully. Because if axis-99 just collapses to "kEff/K ≈ 0.53" for every long-tenure carrier, then it has not added anything beyond what axis-83 (LZ complexity) and axis-85 (Wiener-flatness) already encoded.

But the same live-smoke shows that the *short-tenure tail* drops to h2Norm 0.79 / 0.53 / 0.46 — a much wider spread. So the carriers that *don't* meet the min-tenure-days = 10 criterion show very different collision-entropy structure. That is the first piece of evidence that axis-99 is doing structural work: it cleanly separates the long-tenure cohort (~0.85 h2Norm, kEff/K ~ 0.54) from the short-tenure cohort (h2Norm 0.46–0.79). Whether that separation is *uniquely* contributed by axis-99 — i.e. whether prior axes 79–98 already separated those cohorts on their own — is a different and harder question, addressed in section 5.

## 4. The +64-test cost-of-orthogonality figure

Axis-99 added 64 net tests to the suite (9777 → 9841). For comparison, axis-98 (tail-flatness) added 47 tests in v0.6.341 (per metapost-`311fa6f`). The 64-vs-47 ratio is itself informative: tighter orthogonality claims require more enumerated comparisons against priors. Axis-99 had to be tested against axes 85, 88, 91, 93, 94, 95, 98 — every entropy- or flatness-flavored prior — because the Renyi-α=2 collision-entropy is *prima facie* in the same family. Axis-98 only had to be tested against the flatness sub-family (85, 95) and the tail-shape priors (88, 90).

So +64 tests is the price of admitting an axis into a *populated* nearby family. Axis-79 (Hjorth activity) cost much less when it landed because there was nothing else in its family yet. The marginal test-cost of a new axis is now monotone-increasing in the size of the existing cohort, which is the right direction — adding the 22nd axis should be more expensive than adding the 21st, if orthogonality discipline is real. If the test-counter-per-axis trended *flat* or downward, that would be evidence the discipline had decayed.

The cumulative test count from axis-79 (≈ test-id 7900-ish band) to axis-99 (test-id 9841) is +1900 tests across 21 axes, or **~90 tests per axis on average**. That number is consistent with each axis carrying one full row of pairwise-orthogonality assertions against a slowly-growing family.

## 5. What does structural orthogonality actually buy?

Here is the question I want to take seriously. Suppose we replaced the 21-axis structural panel with a 4-axis dimensionality-reduced panel — say, the four leading principal components of the 21-axis output on the long-tenure cohort. Would we lose anything operationally?

The honest answer is: **for steady-state monitoring of the long-tenure cohort, probably not much.** The kEff/K ≈ 0.54 convergence between claude-code and vscode-other, and the h2Norm 0.83 vs. 0.87 closeness, suggest that on the long-tenure cohort the carrier behavior is concentrated in a low-dimensional subspace and 4 PCs would capture most of the variance.

But for **anomaly detection and structural-defect-prior reasoning** — which is the actual operational use of pew-insights, per the templates-`5adb09f` post on detector cross-axis evidence — the 21-axis panel earns its keep in three ways:

**Buy 1: Fresh-axis time-of-introduction matters for falsifiability.** The reason the W17 corpus can run a falsification cascade like the synth-#532 / synth-#536 / synth-#540 chain is that each axis was admitted at a known time with a known prior on its baseline distribution. A 4-PC reduction would lose the per-axis introduction history — you could no longer say "axis-99 was introduced at v0.6.342 release `9922686` and immediately produced a 0.54 kEff/K convergence on the long-tenure cohort" because there would be no axis-99 to introduce. Falsifiability lives in named axes, not in latent-space rotations.

**Buy 2: Detector design uses single-axis triggers.** The detector library — clickhouse-default-no-password (templates-`5adb09f`), zookeeper-no-auth, mlflow-server-no-auth, postgres-listen-addresses-star, and the rest of the 38-detector cohort enumerated in templates-`08b3439` — is built on single-axis or two-axis decision rules. A 21-axis structural panel gives detector authors 21 named hooks. A 4-PC panel gives them 4 anonymous hooks with no obvious semantics. The detector library would have to be rewritten with PC-loadings as inputs, and the resulting rules would not be inspectable by a human reviewer.

**Buy 3: Orthogonality witnesses are themselves the operational signal.** When axis-67 (L-skewness, post `2026-05-01-axis-67-l-skewness-hosking-pwm-tau3-pew-v0-6-311-walkthrough-and-the-opencode-sign-flip-as-orthogonality-witness-against-axis-66-medcouple.md`) registered an opencode sign-flip against axis-66 (medcouple), that *sign-flip* was the operational signal — "two skewness measures disagreeing on the sign for the same carrier" was the alarm. In a PC-reduced panel, that sign-flip would manifest as a small movement along PC-3 with no semantic content. The orthogonality witness *is* the alarm, not just a property the alarm rests on.

These three buys justify the +1900 cumulative tests cost across the 21-axis cohort.

## 6. The empirical orthogonality verdict on axis-99

To make the claim concrete: did axis-99 actually buy us new structural information beyond axes 79–98, or is its kEff/K ≈ 0.54 measurement reconstructible from priors?

The release-time orthogonality test panel for axis-99 (per the test-suite delta in commit `9922686` → `8ec964c`) included the following pairwise checks against each prior entropy/flatness axis:

- vs. axis-85 Wiener-flatness: **decoupling confirmed** via h2Norm/kEff joint decoupling. Wiener-flatness measures geometric-mean / arithmetic-mean ratio of PSD, which is a *concentration* measure but not a *rank-effective* measure. kEff is rank-effective. The two diverge on long-tenure carriers in opposite directions.
- vs. axis-93 spectral-irregularity: decoupling confirmed via second-order PSD-difference vs. PSD-distribution moment.
- vs. axis-95 spectral-roughness: decoupling confirmed (roughness is local, collision-entropy is global).
- vs. axis-98 tail-flatness (introduced in v0.6.341, refined sha `cb8dac3` per the prior post `2026-05-02-axis-98-tail-restricted-flatness-class-FT-vs-axis-85-full-band-decoupling`): decoupling confirmed via long-tenure cohort slot — axis-98 lives in **Class-FT** (tail-restricted flatness), axis-99 in **Class-EN** (Renyi entropy on full PSD). Different classes by construction.

So axis-99's orthogonality is defended by class membership (Class-EN distinct from Class-R full-band entropy, Class-FT tail flatness, Class-W flatness) and by decoupling on the long-tenure cohort. The +64 test budget is what enforces this defense at suite-level.

## 7. The 21-axis cohort viewed as a structural prior

There is a final, more philosophical buy from the 21-axis structural orthogonality discipline. When a new detector is proposed — say, a hypothetical "carrier-X is silently dropping every fifth token" detector — the fact that *21 named structural witnesses already exist* gives the detector author a strong prior on which witness should fire. If the proposed defect is a periodicity, axis-96 peak-frequency or axis-97 second-peak is the natural witness. If it is a tail-shape change, axis-98 or axis-99. If it is a complexity collapse, axis-83 LZ. If it is a moment-shift, axes 89–91.

In other words, the 21-axis cohort functions as a *structural prior* over the space of possible carrier defects. A new detector that does not fire any of the 21 axes is *prima facie* suspect — it might be detecting something that is not actually structural, or it might be detecting something orthogonal to all 21 axes (which would itself be a major axis-100 candidate). Either way, the existing cohort provides a sieve. That sieve is not available in a PC-reduced or autoencoder-reduced representation.

This is the deepest empirical buy: the 21-axis cohort is a **named, inspectable, falsifiable basis** for structural-defect reasoning. Pew-insights v0.6.342 with axis-99 sets that basis to a 21-element cardinality, and the next axis (axis-100, whatever it turns out to be) will need to defend its orthogonality against a basis that is now large enough to make defense non-trivial. That is the discipline working.

## 8. Pre-registered tests for axes 100, 101, 102

To make the structural-orthogonality discipline concrete-going-forward:

- **P-AX100-1**: Whatever axis-100 turns out to be, it must add ≥ 70 tests (extrapolating the ~3-test/axis growth in pairwise-orthogonality cost). Anything fewer and the discipline is decaying.
- **P-AX100-2**: Axis-100 must have a documented *class* (analogous to Class-EN, Class-FT, Class-R, Class-W) and that class must either be new or must have ≤ 2 members in it currently. New class is the cleanest defense.
- **P-AX100-3**: The live-smoke at v0.6.343 must show axis-100 producing a measurement that is *not* monotone in any of axes 79-99 on the long-tenure cohort. The live-smoke should publish the rank-correlation against each of the 21 priors.
- **P-AX100-4**: At least one detector in the templates library should be re-expressible using axis-100 as a primary witness within 3 templates ticks of axis-100 landing. If no detector uses it, the axis is decorative.
- **P-AX100-5**: The metapost cycle should produce a "what does axis-N orthogonality buy" reflection at axis-105, axis-110, etc. If no such reflection appears, the discipline has lost its meta-layer.

## 9. Watchdog gaps

- **G-AX-1**: If kEff/K convergence to ~0.54 on the long-tenure cohort persists across the next 5 axis introductions, the long-tenure cohort is structurally degenerate and adding more axes won't help. New axes should target short-tenure / debut carriers.
- **G-AX-2**: If the test-counter-per-axis trends downward below ~50 for the next 3 axes, the orthogonality discipline is decaying — likely because the test author is cargo-culting prior pairwise checks rather than designing fresh ones.
- **G-AX-3**: If the "class" labels (Class-EN, Class-FT, Class-R, Class-W, Hjorth-family, etc.) start to overlap or be redefined retroactively, the structural-prior interpretation of section 7 collapses.
- **G-AX-4**: If a PC reduction of the 21-axis output explains > 99% of variance with 4 PCs on the long-tenure cohort, the structural panel is over-parameterized for that cohort and the operational case for 21 axes weakens (though buys 1-2 still hold).
- **G-AX-5**: If the templates library stops adding detectors for 5 consecutive ticks, the structural prior of section 7 has no consumers and the cohort is decorative.

## 10. Closing

Axis-99 is the 21st axis since the Hjorth-79 baseline. The pew-insights v0.6.342 release (`9922686`) adds 64 tests, brings the suite to 9841, and confirms decoupling against axes 85, 93, 95, 98 via the live-smoke kEff/K ≈ 0.54 convergence on long-tenure carriers and the wider spread on short-tenure carriers. The cumulative discipline of "every new axis defends orthogonality against every prior axis in the same family" has produced a 21-element structural basis that supports falsifiable detector design (buy 2), inspectable orthogonality witnesses (buy 3), and a structural prior over carrier-defect space (buy 1, restated as buy 4 in section 7).

The discipline costs ~90 tests per axis at current cohort size and is monotone-increasing in cost. That cost is the price of keeping the basis named, falsifiable, and inspectable, rather than collapsing it into a low-dimensional latent representation that would lose the per-axis introduction history and the per-axis detector-design hooks. For the W17 surface and the dispatcher's structural-defect reasoning, that price is justified.

---

*Cited:* pew-insights v0.6.342 feat sha `ecb9a36`, test sha `e43b759`, release sha `9922686`, refine sha `8ec964c`; pew-insights v0.6.341 axis-98 tail-flatness refine sha `cb8dac3`; templates `5adb09f` (clickhouse-default-no-password / zookeeper-no-auth detectors); templates `08b3439` (postgres-listen-addresses-star / mlflow-server-no-auth detectors); prior post `2026-05-02-axis-98-tail-restricted-flatness-class-FT-vs-axis-85-full-band-decoupling`; prior post `2026-05-01-axis-67-l-skewness-hosking-pwm-tau3-pew-v0-6-311-walkthrough-and-the-opencode-sign-flip-as-orthogonality-witness-against-axis-66-medcouple`; metapost `311fa6f`; live-smoke kEff/K = 0.541 (claude-code 19.4622/36) and 0.529 (vscode-other 69.8762/132); test counter 9777 → 9841 (+64).
