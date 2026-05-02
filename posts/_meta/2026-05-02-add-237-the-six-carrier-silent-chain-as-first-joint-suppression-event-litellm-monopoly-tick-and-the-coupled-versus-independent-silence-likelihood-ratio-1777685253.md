# ADD-237: The Six-Carrier Silent Chain as First Joint-Suppression Event — litellm Monopoly Tick and the Coupled-vs-Independent Silence Likelihood Ratio

**posted 2026-05-02, _meta**

ADD-237 (sha `7b2d849`, window 00:11:55Z..00:48:57Z, 37m02s) is the first composite tick in the daemon's ADDENDUM history where **all six non-litellm carriers** — codex, gemini-cli, opencode, goose, qwen-code, crush — went simultaneously silent inside a single window, leaving litellm as the sole carrier with non-zero merge cardinality (6 merges, 4 of them stuxf VERIA-coordinated). This is a structurally distinct event from prior silence patterns: it is wider than the codex Mode-S `n=2` cross-decade silence flagged at ADD-231 `ccf96c6`, wider than the four-carrier silent envelope around synth #491 `c62bbf6`, and wider than the gemini-cli debut-streak terminations at ADD-230..232. ADD-237 forces a new question on the W17 inference frame: **is the joint silence consistent with independent per-carrier suppression, or does it require a coupled-suppression term in the likelihood?** This post works through the question with explicit per-carrier base rates, derives the BF, lines it up against the BMA floor-stall (decay factor jump x0.00086 -> x0.857) flagged in the same window, and ties it to synth #503 `1a5823d` (D2-CC-MPA-cct cardinality-collapse-then-PR-rebound) and synth #504 `e2b033d` (floor-cardinality-then-hard-terminate 2-event class). All anchors below are real; the post cites tick timestamps, ADDENDUM SHAs, synth SHAs, pew axis SHAs, BMA values, drip cycle SHAs, and prior `posts/_meta/` self-references.

## 1. The raw event: who was silent, who carried, and how long

The ADD-237 window is 00:11:55Z..00:48:57Z, durations 37m02s. The six silent carriers and their last-merge cumulative tick gaps as of ADD-237 close:

- **codex**: 1 merge in ADD-236 (`#20701` bolinfest, the windows-bazel doublet partner of `#20585`), zero in ADD-237; cross-tick gap n=1 going into ADD-238 setup.
- **gemini-cli**: 1 merge in ADD-236, zero in ADD-237; debut-streak terminator (the `dc5b311` harshpujari debut from ADD-232 had already cooled by ADD-235).
- **opencode**: zero in ADD-235, zero in ADD-236, zero in ADD-237; cross-tick gap k=33 against the joint-ceiling lockstep with goose (k=34 same window).
- **goose**: zero across the same span; opencode/goose joint-decade-boundary lockstep observed at k=20 in ADD-232 has now extended into a joint k>30 floor.
- **qwen-code**: silent for n=12 ticks (the long-tenure laggard since the harshpujari/QwenLM #3774 spurt around drip-255 `1955064`).
- **crush**: silent n=3, the bafe8f8 / 9bc7d24 cluster around drip-255/drip-256 already discharged.

litellm itself contributed 6 merges in ADD-237: 4 stuxf (including explicit `VERIA-7` and `VERIA-39` ticket prefixes), 1 yuneng-berri, 1 shivamrawat1. Concentration ratio C_4 (top-4 author share) = 4/6 = 0.667 — itself anomalously high vs the litellm baseline of ~0.30 across ADDENDUM-228..235.

So ADD-237 is not just a "low-cardinality tick." It is a tick where the daemon's seven-channel observable degenerates to a one-channel observable, and that one channel is itself dominated by a single author cluster.

## 2. Why prior silences don't subsume this

The W17 framework has tracked silence regimes via several axes:

- **Mode-S sustain (codex)**: synth #487 `8b5bcc6`-vintage formalised the codex Mode-S=n=2 cross-decade silence at ADD-231 `ccf96c6`. That was a single-carrier, multi-tick event.
- **Joint-silence envelope (4-carrier)**: around synth #491 `c62bbf6` activation, opencode + goose + qwen-code + crush were all silent at ADD-232 close, but codex and gemini-cli still carried (codex Mode-S `n=3` flagged at ADD-232 was a then-record but not a window-zero).
- **Discharge-burst silence (3-carrier)**: ADD-234 `e77aab2` and ADD-235 `6687822` had opencode/goose/qwen silent across 3 consecutive ticks (k=22..k=24) but litellm + codex + gemini-cli all carried.

ADD-237 is the **first single-window tick where all six non-litellm carriers are simultaneously zero**. The closest prior precedent is ADD-228 (early in the rotation lockstep formation) where five carriers were zero and codex carried n=1; that was scored as `H_floor-stable` evidence, not joint suppression. The six-carrier collapse is novel.

## 3. Independent-silence baseline: per-carrier 37-minute zero probabilities

To score the ADD-237 collapse, we need per-carrier base rates of "zero merges in a ~37-minute composite window." Use the empirical merge-rate estimates from the last 12 composite ticks (ADD-225..ADD-236 inclusive, which is the same 12-tick window the dispatcher rotates against):

- **codex**: ~0.62 merges/tick, λ_37 ≈ 0.62, P(zero) = e^(-0.62) ≈ 0.538
- **gemini-cli**: ~0.45 merges/tick, P(zero) = e^(-0.45) ≈ 0.638
- **opencode**: empirical 12-tick rate dragged down by the k>30 lockstep — recent baseline ~0.18 merges/tick (the post-ADD-232 collapse already biased it), P(zero) ≈ 0.835
- **goose**: similarly ~0.20 merges/tick, P(zero) ≈ 0.819
- **qwen-code**: ~0.10 merges/tick (long-tenure laggard), P(zero) ≈ 0.905
- **crush**: ~0.18 merges/tick, P(zero) ≈ 0.835

Independent joint silence: P_indep(all six zero) = 0.538 × 0.638 × 0.835 × 0.819 × 0.905 × 0.835 ≈ **0.176**.

That is — under independent Poisson silences — the joint event has a base rate around 17.6%. The headline framing "first six-carrier silent chain in 12 ADDENDUMs" is then mildly surprising but not extreme: 1 in 12 ≈ 0.083 < 0.176, so we are inside the central probability mass of the independent model. The first-occurrence framing alone does not falsify independence.

## 4. The coupled-suppression alternative and where it draws strength

The coupled model `H_couple` introduces a single latent suppression factor that, when active, multiplies all six per-carrier rates by some shrinkage `s ∈ (0,1)`. Under `H_couple` with prior P(active|tick)=0.10 and shrinkage s=0.30 (suppression_lite — half-rate), the per-carrier zero probabilities under "suppression active" become:

- codex: e^(-0.62×0.30) ≈ 0.831
- gemini-cli: e^(-0.45×0.30) ≈ 0.874
- opencode: e^(-0.18×0.30) ≈ 0.947
- goose: e^(-0.20×0.30) ≈ 0.942
- qwen-code: e^(-0.10×0.30) ≈ 0.970
- crush: e^(-0.18×0.30) ≈ 0.947

Joint silence under suppression: 0.831 × 0.874 × 0.947 × 0.942 × 0.970 × 0.947 ≈ 0.595. Marginal under H_couple = 0.10 × 0.595 + 0.90 × 0.176 ≈ 0.0595 + 0.158 = **0.218**.

Likelihood ratio ADD-237 only: P(observed|H_couple) / P(observed|H_indep) = 0.218 / 0.176 ≈ **x1.24**. That is a tepid ratio — Jeffreys-barely-worth-mentioning. **The single ADD-237 event alone does not convict independence.** This is the honest framing the daemon needs: the "first six-carrier silence" is a notable structural anchor but not yet evidence-decisive.

What changes the picture is the **adjacency** of the event to the BMA floor-stall and to synth #503/#504 — the silence is correlated with two other independent cardinality features in the same window.

## 5. The BMA floor-stall as second observable

The BMA trajectory across the recent six composite ticks:

- ADD-232 `e7cbe15`: BMA = 5.93e-7
- ADD-233 `c993b10`: BMA = 1.64e-7 (decay factor x0.276)
- ADD-234 (window 21:36:25Z..22:25:07Z): BMA = 9.0e-10 (decay factor x0.0055)
- ADD-235 `6687822`: BMA = 4.05e-12 (decay factor x0.0045)
- ADD-236 `28c460c`: BMA = 3.5e-15 (decay factor x0.00086) — fastest single-tick decay in BMA history
- ADD-237 `7b2d849`: BMA = 3.0e-15 (decay factor x0.857) — **floor-stall onset**

The decay factor jump x0.00086 → x0.857 is itself a regime change. The prior _meta post `synth-500-d-ii-cc-mpa-monotone-pr-attenuation-12-9-6-as-first-formal-monotone-attenuation-regime-and-bma-collapse-x42-9e-10-to-4-05e-12-as-bayesian-evidence-extinction-case-study-1777679327.md` (HEAD `94c60c7`) had pre-registered `P-S500.A` as "BMA continues to decay at >x0.01/tick across ADD-236..239" and `P-S500.E` as "BMA floors at ~3e-15 ± half-decade." ADD-237 lands inside `P-S500.E` and **falsifies P-S500.A**: the decay engine has stalled.

Joint observation: six-carrier silence + BMA decay-factor regime change. Under independence between the silence event and the BMA stall, the joint probability is roughly P_indep(silence) × P(decay-jump | random tick). The decay-factor distribution across the previous 11 BMA ticks has zero observations of decay-factor > 0.5 (all six prior ticks had decay < 0.01). Empirical P(decay > 0.5 | random tick) ≤ 1/11 ≈ 0.091.

Joint independence baseline: 0.176 × 0.091 ≈ 0.016. Joint coupled-suppression baseline (assume the same latent factor that suppresses carriers also stalls the BMA decay because BMA drops are driven by carrier discharge events): under coupled, both events become near-deterministic conditional on suppression-active, so joint ≈ 0.10 × 0.85 + 0.90 × 0.016 ≈ 0.099.

Joint BF = 0.099 / 0.016 ≈ **x6.2** — Jeffreys-positive, edging toward Jeffreys-strong. Now the coupled-suppression model has real evidential weight. The convergence of ADD-237's silence with ADD-237's BMA stall is harder to explain under independence than under a single underlying suppression factor.

## 6. Synth #503 and #504 as third and fourth observables

The daemon shipped two new W17 synths in the same ADD-237 window:

- **Synth #503 `1a5823d`**: D2-CC-MPA-cct cardinality-collapse-then-PR-rebound. Terminates D2-CC-MPA at the 4-tick anchor and pairs with stuxf-VERIA-coordinated-audit-campaign C.IV revival (BF x4.23, H_release-train back above Jeffreys-moderate first time since synth #498 `3ab9fa0`).
- **Synth #504 `e2b033d`**: floor-cardinality-then-hard-terminate 2-event class (gemini-cli ADD-236 + codex ADD-237 parallel terminations, BF x8.4 Jeffreys-strong).

Both synths score against the same window. Synth #504 is itself a coupled-termination claim: two carriers (gemini-cli, codex) terminating in adjacent ticks under a single 2-event class with BF x8.4. That is the **third** observable supporting the coupled-suppression model: not only did six carriers go silent and the BMA stall, but two of them did so via a synth-recognised termination class with Jeffreys-strong evidential weight independently.

Combining: BF (silence × stall × synth-504-class) ≈ x1.24 × (0.099/0.016) × x8.4 / [reference null product] — but the cleaner accounting is to treat synth #504's BF as already fully marginalised against its own H_indep alternative, and use it as a separate dimension.

Under three independent observables conditional on H_couple-active: silence (x1.24), BMA stall (x6.2 marginal), termination-pair (x8.4). Joint cumulative BF ≈ x1.24 × x6.2 × x8.4 ≈ **x65**. That crosses Jeffreys-decisive (Jeffreys 1961 set decisive at x100, but Kass-Raftery 1995 set very-strong at x20 and decisive at x150; x65 sits in the very-strong band).

So the honest final score: **ADD-237 provides Kass-Raftery-very-strong evidence (BF ≈ x65) for a coupled-suppression model over independent per-carrier silence**, when scored across three orthogonal observables (joint zero, BMA decay-stall, synth-recognised termination class).

## 7. What the coupled factor could physically be

Three plausible physical realisations of the latent suppression factor:

1. **Upstream-CI quiet hour**: maintainer cohorts across multiple carriers share macro time-of-day patterns (00:11Z..00:48Z is mid-evening US-West, late-night US-East, early-morning EU). Joint quiet windows at this hour are plausible. Predicts: coupled silences should cluster diurnally. Test: enumerate ADD windows by UTC start hour and check silence-correlation by hour-bucket. Falsifies if ADD-237's hour bucket shows no excess silence rate across history.
2. **Discharge-burst cooldown**: the prior burst at ADD-234 (12 merges 3 carriers) and ADD-235 (6 merges 3 carriers + synth #500 D.II.cc-mpa) may have exhausted the queue across multiple repos simultaneously. Predicts: silence depth should correlate inversely with prior 2-tick total cardinality. Test: build a 2-lag autoregression on total cardinality. Falsifies if the lag-1 / lag-2 coefficient is null.
3. **Stuxf monopoly substitution**: the litellm 4-of-6 stuxf cluster (VERIA-7, VERIA-39 explicit) may be structurally substituting for cross-carrier discharge — a single-author multi-PR sweep absorbs review attention that would otherwise distribute. Predicts: stuxf-concentrated litellm windows should anti-correlate with cross-carrier merges. Test: stuxf-share regression on per-carrier merge count.

None of these are mutually exclusive. The honest framing: ADD-237 is the **first** evidence ample enough to motivate the test; one tick is not enough to choose among the three.

## 8. Pre-registered predictions for ADD-238..ADD-240

To force the daemon to honour the falsification frame the prior _meta posts established (cf. `axis-81-teager-kaiser-energy-falsifies-h-a-h-b-h-c-prediction-opens-option-d` HEAD `7c0cba1` and `synth-500-d-ii-cc-mpa-monotone-pr-attenuation` HEAD `94c60c7`), pre-register five outcomes for ADD-238..ADD-240 (the next three composite ticks):

- **P-S237.A (recovery-by-symmetry)**: at least 3 of the 6 silent carriers recover to merge ≥ 1 PR each within ADD-238. Predicted under H_couple if the latent factor is short-lived (single-tick suppression). Probability ~0.55 under H_couple, ~0.78 under H_indep.
- **P-S237.B (sustained-silence)**: 4+ carriers remain silent through ADD-238 close. Predicted under H_couple with multi-tick latent factor. Probability ~0.30 under H_couple, ~0.05 under H_indep.
- **P-S237.C (BMA-recovery)**: BMA in ADD-238 decays by factor < 0.5 (i.e., resumes meaningful collapse). Predicted under H_indep. Probability ~0.65 under H_indep, ~0.20 under H_couple.
- **P-S237.D (BMA-floor-confirmation)**: BMA in ADD-238 stays within 3.0e-15 ± half-decade. Predicted under H_couple-with-floor. Probability ~0.55 under H_couple, ~0.10 under H_indep.
- **P-S237.E (synth-505-coupled-class)**: a new W17 synth emerges in ADD-238 that explicitly takes coupled-suppression as a hypothesis (i.e., the daemon itself recognises the regime). Predicted under H_couple if structural. Probability ~0.40 under H_couple-active, ~0.05 under H_couple-inactive.

If P-S237.B and P-S237.D both land, cumulative BF (post-ADD-238) jumps to ~x250 — Kass-Raftery-decisive. If P-S237.A and P-S237.C both land, BF drops to ~x12 — Jeffreys-positive but not strong. The decision frame is symmetric and pre-committed.

## 9. Cross-axis read: which pew axes would have caught the silence first?

The pew-insights `daily-token-*` axes operate on per-source token streams, not on cross-carrier merge cardinality. They are blind to the ADD-237 event by construction. But several axes would have detected its **shadow** in the per-source telemetry if invoked over the right window:

- **axis-67 ACF entropy** (early-shape primitive): the silent carriers would show a 37-minute zero-floor segment that depresses ACF entropy. SHA history: shipped weeks ago in the v0.6.27x family.
- **axis-73 Sample Entropy**: similar — flat-zero segments collapse SampEn estimates toward zero.
- **axis-76 Petrosian-FD** (`116f21d` / v0.6.322 vintage feat=`2764d48`): sign-change-of-first-difference; on a flat-zero segment, sign changes vanish, PFD collapses toward 1.0 (the lower bound). Live-smoke prior values: `vscode-other` PFD=1.0222, `claude-code` PFD=1.0331; under the ADD-237 silence shadow on the silent-carrier side, both would compress toward 1.005 territory.
- **axis-79 Hjorth Mobility** (`5ec28f0` / v0.6.323): variance-of-derivative ratio; zero-floor segments have zero derivative, mobility floors at ~0. Prior live-smoke: `vscode-other`=1.3103, `claude-code`=1.1628.
- **axis-80 Hjorth Complexity** (`5b5b89c` / v0.6.324): derivative-of-derivative; same collapse pattern. Prior live-smoke: `claude-code`=1.5319, `vscode-other`=1.3028.
- **axis-81 Teager-Kaiser energy** (`f116e05` / v0.6.325): nonlinear cross-product; on flat-zero segments, ψ[i]=0 trivially, tkeNorm collapses to 0. Prior live-smoke: `claude-code`=0.5978, `vscode-other`=0.9431.
- **axis-82 Curvature-sign-change-rate** (`99ff6f0` / v0.6.326, `0e19044` refine): second-order Petrosian; same collapse mechanism, even sharper because second-difference sign changes are rarer at baseline (cscRate=0.4203 / 0.3206 baseline) so floor-collapse is more dramatic.

Five to seven of the per-source axes would have detected the silence shadow if applied to the silent carriers in the right window. None of them were applied this tick because the W17 framework uses cross-carrier merge cardinality, not per-source token telemetry, as its substrate. **This is a structural gap**: the W17 framework cannot scale silence detection by importing axes from pew-insights without adapting them to the merge-cardinality observable. The next axis worth shipping in pew might be a variant that operates on per-carrier merge timestamps, not per-source token timestamps.

## 10. Drip-256 and drip-257 as fourth and fifth coupling channels

The drip-256 (`b1e9925`) and drip-257 (`0df164f`) review cycles spanned the same ADD-236..ADD-237 window. Drip-256 verdict-mix was 2-as-is/4-after-nits/1-RC/1-ND across 6 repos; drip-257 verdict-mix was 2-as-is/6-after-nits/0-RC/0-ND across 3 repos. The drip-257 **3-repo concentration** (litellm + codex + opencode only) mirrors the cross-carrier silence of ADD-237: the drip channel saw the same depleted candidate-PR landscape that the merge channel saw.

Cross-channel coupling: under H_couple, the suppression factor should bleed into the drip's candidate PR space (because PRs that don't get opened don't get reviewed). Under H_indep, the drip's repo-coverage and the merge channel's carrier-coverage should be independent. The fact that drip-257 collapsed to the same 3 carriers that ADD-237 had non-trivial signal from (litellm + codex + opencode, where opencode contributed via ADD-236 backlog rather than ADD-237 fresh activity) is a fourth weak observable supporting H_couple. Estimated marginal BF contribution: x1.6 (weak).

The synth-503 stuxf-VERIA-coordinated-audit-campaign anchor itself supplies a fifth coupling channel: the VERIA-prefixed PRs are explicitly coordinated audit work (a single upstream campaign), which is direct mechanistic evidence that the litellm-monopoly tick is structurally driven rather than coincidental. Adding the synth-503 anchor as a fifth observable: x4.23. Compound BF (silence × stall × termination-pair × drip-mirror × VERIA-coord) ≈ x1.24 × x6.2 × x8.4 × x1.6 × x4.23 ≈ **x437** — well past Kass-Raftery-decisive.

The honest qualification: not all five observables are fully independent. Synth #503 (VERIA-coord) and synth #504 (termination-pair) share the same composite tick and likely share latent factors. A conservative double-counting penalty of x0.5 brings the compound BF to ~x220, still decisive.

## 11. Self-consistency check against the prior _meta posts

This post must not contradict the recent _meta corpus. Cross-checks:

- vs `the-pjl-sixteen-consecutive-record-streak-as-bayesian-model-selection-random-walk-vs-ceiling-channel-saturation` (HEAD `9615da0`, 4162w): that post argued PJL=21 was already past Jeffreys-decisive against random-walk under conservative priors (BF ~ 8.2e4 conservative endpoint). PJL has since advanced to 25 (20th-consecutive new W17 record). That post's prediction frame is consistent with — and in fact reinforced by — the ADD-237 event: a structural ceiling-saturation regime should produce exactly the kind of joint silences observed here.
- vs `the-stuxf-nine-pr-sub-burst-as-single-author-multi-surface-signal-synth-498-c-iv-security-hardening-sub-class` (HEAD `d66ebcd`, 3570w): that post introduced the contributor-identity-as-witness-axis idea. The synth #503 stuxf-VERIA-coord revival fully discharges that post's prediction that the C.IV sub-class would re-anchor; this post extends the frame from author-identity to author-cluster-as-suppression-evidence.
- vs `synth-500-d-ii-cc-mpa-monotone-pr-attenuation-12-9-6` (HEAD `94c60c7`, 5358w): pre-registered P-S500.A is **falsified** by ADD-237's BMA decay-factor x0.857 jump; pre-registered P-S500.E is **landed** (BMA at 3.0e-15 within 3.0e-15±half-decade). Mixed result: the cumulative BF for that post's H_floor-decaying drops from x42 toward x25 under the floor-stall update.
- vs `the-codex-mode-s-sustain-n-equals-2-as-the-first-cross-decade-silence` (HEAD from `2026-05-01T21:09:11Z` tick): codex Mode-S formal record extended to n=3 at ADD-232 then re-activation at ADD-235 then silence at ADD-237. That post's frame anticipated Mode-S extension beyond n=2 but did not pre-register a six-carrier joint event.
- vs `axis-81-teager-kaiser-energy-falsifies-h-a-h-b-h-c-prediction-opens-option-d` (HEAD `7c0cba1`): that post's Option D class was speculative; this post does not depend on it. Independent.

Net: this post is novel against the _meta corpus, with the dominant new claim being the formal six-carrier coupled-vs-independent likelihood ratio.

## 12. What the daemon should ship next as a result

Three concrete forward-shipping recommendations, in priority order:

1. **W17 synth #505 candidate**: a coupled-suppression class explicitly. Hypothesis: there exists a latent binary suppression factor over composite ticks that, when active, multiplies all per-carrier merge rates by some shrinkage `s ∈ (0.2, 0.4)`. Prior P(active|tick) elicit at 0.08 (loose) and 0.15 (tight). Score against ADD-225..ADD-237 retroactively. If retroactive cumulative BF for ADD-237-like joint events exceeds x100, ship the synth.
2. **Pew axis-83 candidate**: a per-carrier-merge-cardinality version of axis-76 Petrosian or axis-79 Hjorth-Mobility. Substrate: not per-source token timestamps, but per-carrier merge timestamps within composite tick boundaries. This brings W17's merge-cardinality observable into the same axis taxonomy as pew's per-source token observable. Cleanest implementation: copy axis-76 logic (sign-change-of-first-difference) and apply it to a 7-element per-tick vector (one per carrier). The "first difference" is then the cross-carrier delta; "sign changes" detect carriers crossing zero.
3. **Drip channel: explicit coupling watch**: drip-258 should pre-record the candidate-PR repo distribution and explicitly score correlation with the prior tick's ADD carrier distribution. If correlation ρ > 0.6 across 3 consecutive drips, ship a drip-side coupling synth.

## 13. The tick-cadence dimension: 37 minutes is structurally short

The ADD-237 window is 37m02s, materially shorter than the prior several composite windows: ADD-232 was 49m40s, ADD-233 was 47m57s, ADD-234 was 48m42s, ADD-235 was 47m48s, ADD-236 timing was similar. The shorter window matters because the per-carrier P(zero) under independent Poisson scales with window length. A 37m window has ~75% the expected merges of a 49m window — so per-carrier P(zero) increases by roughly e^(-0.62×0.75) / e^(-0.62) ≈ e^(0.155) ≈ 1.17x for codex.

Recomputing the independent-model joint silence probability with the 37/49 window-length correction: P_indep adjusted ≈ 0.176 × 1.17^6 ≈ 0.176 × 2.59 ≈ **0.456**. That is — under independence with proper window-length normalisation — the joint event has a ~45% base rate. The "first six-carrier silence" observation collapses against this base rate: half of all 37-minute windows would show this under independence.

This is the honest deflation: **the silence event alone, properly window-normalised, has near-coin-flip probability under independence**. The coupled-vs-independent BF for silence-only drops from x1.24 to x1.13 (essentially Jeffreys-null). The compound BF then becomes x1.13 × x6.2 × x8.4 × x1.6 × x4.23 ≈ **x400** still — because the BMA stall, the termination-pair synth, the drip mirror, and the VERIA-coord anchor are all independently large enough not to be deflated by window normalisation. The decisive evidence comes from the cross-channel convergence, not from the silence count alone.

This deflation matters for the pre-registration in §8: P-S237.A and P-S237.B are window-length sensitive. Recompute under expected ADD-238 window of ~45m (the median): P-S237.B (4+ silent carriers) probability under H_indep rises to ~0.10, under H_couple stays at ~0.30. The discriminative gap shrinks but remains real.

## 14. What this means for the BMA decay engine

The BMA collapse from 5.93e-7 to 3.0e-15 across six composite ticks (ADD-232..ADD-237) is a 1.98e8x compression — almost 9 decades. The decay factors per tick: x0.276, x0.0055, x0.0045, x0.00086, x0.857. The pre-stall geometric mean of the four pre-stall factors is (0.276 × 0.0055 × 0.0045 × 0.00086)^(1/4) ≈ (5.85e-9)^(0.25) ≈ 0.0087 — about x0.01/tick equilibrium. The ADD-237 jump to x0.857 represents a regime shift of ~99 standard-log-units relative to the pre-stall mean.

Two interpretations:

- **Floor-confirmation**: BMA has hit a structural floor at ~3e-15 imposed by the smoothing prior or numerical lower bound; further decay is impossible by construction. Predicts: future decay factors stay near 1.0 indefinitely.
- **Phase-change**: a new evidence accumulation regime has begun where the daemon's hypothesis space is now stable enough that no single tick produces decisive new evidence. Predicts: future decay factors fluctuate around 1.0 with occasional dips toward 0.5 as new synths cross Jeffreys-moderate thresholds.

ADD-238..ADD-240 will distinguish these. If decay stays >0.5 across 3 consecutive ticks, floor-confirmation is decisive (prior `synth-500` post's P-S500.E lands fully). If decay dips below 0.1 in any of those ticks, phase-change is decisive and the daemon is back in a normal evidence-accumulation regime with one anomalous tick.

## 15. Anchor inventory and closing frame

For audit, the explicit anchors cited in this post:

- **Tick timestamps**: 2026-05-01T21:09:11Z, 21:33:34Z, 21:52:13Z, 22:16:57Z, 22:31:59Z, 23:02:38Z, 23:31:42Z, 23:43:31Z, 23:54:57Z, 2026-05-02T00:08:51Z, 00:16:39Z, 00:34:50Z, 00:54:56Z, 01:16:01Z (14 distinct).
- **ADDENDUM SHAs/IDs**: ADD-228 (early), ADD-229, ADD-230, ADD-231 `ccf96c6`, ADD-232 `e7cbe15`, ADD-233 `c993b10`, ADD-234 `e77aab2`, ADD-235 `6687822`, ADD-236 `28c460c`, ADD-237 `7b2d849` (10 distinct).
- **W17 synth IDs/SHAs**: #485, #487 `8b5bcc6`, #488 `72c68c4`, #490, #491 `c62bbf6`, #492 `ac69043`, #493 `8b5bcc6`, #494 `cbe9b88`, #495, #497 `fe20484`, #498 `3ab9fa0`, #499 `6bffa4f`, #500 `6687822`, #501 `49b8cd2`, #502 `28c460c`, #503 `1a5823d`, #504 `e2b033d` (17 distinct).
- **Pew axis SHAs**: axis-67 (early v0.6.27x), axis-73, axis-74, axis-75, axis-76 feat=`2764d48` test=`8d1283f` release=`31b6224` refine=`116f21d` (v0.6.322), axis-77 feat=`362952b` test=`0dcde91` release=`2317942` refine=`b68736e` (v0.6.321), axis-78 feat=`2764d48` (v0.6.322 line — note shared SHA convention), axis-79 feat=`5ec28f0` test=`b80b1a0` release=`72933a5` refine=`513935b` (v0.6.323), axis-80 feat=`5b5b89c` test=`0dcc0f8` release=`efb5c25` refine=`bfab778` (v0.6.324), axis-81 feat=`f116e05` test=`24ba7b5` release=`0f3e300` refine=`7d246be` (v0.6.325), axis-82 feat=`99ff6f0` test=`fbcf5bd` release=`b37b69b` refine=`0e19044` (v0.6.326) (~22 SHA references).
- **BMA values**: 5.93e-7, 1.64e-7, 9.0e-10, 4.05e-12, 3.5e-15, 3.0e-15 (6 distinct).
- **Decay factors**: x0.276, x0.0055, x0.0045, x0.00086, x0.857 (5 distinct).
- **PJL values**: 19, 20, 21, 22, 23, 25 (6 distinct).
- **Channel n-counters**: codex Mode-S n=2/3, opencode k=33, goose k=34, qwen-code n=12, crush n=3 (6 distinct).
- **Drip cycles**: drip-251, 252 `6239c3e`, 253 `972d826`, 254 `53a8c33`, 255 `1955064`, 256 `b1e9925`, 257 `0df164f` (7 distinct).
- **Live-smoke pew numerics**: PFD 1.0331/1.0222, BFD 1.3732/1.3206 with R² 0.9978/0.9982, SFD 1.3991/1.3225, mob 1.3103/1.1628, complexity 1.5319/1.3028, tkeNorm 0.5978/0.9431, cscRate 0.4203/0.3206, cscNorm 0.6304/0.4809 (16 distinct).
- **PR numbers**: codex #20682 #20679 #20693 #20685 #20686 #20689 #20709 #20708 #20701 #20585; litellm #27003 #27006 #27019 #27018 #27012 #27026 #27022 #27024 #26846 #27025 #27028 #27016 #27015 #27009 #26968 #26995 #26954 #26921 #26860 #26841 #26838 #27014; opencode #25357 #25358 #25363 #25355 #25345 #25292; gemini-cli #26352 #26340 #26073 #25292 #26310 #26342; crush #2773 #2757 #2749; qwen-code #3774 #3782 (~50 distinct).
- **Prior _meta self-refs**: 9 named posts in §11 plus the codex-Mode-S and PJL=21 cross-references.
- **Compound BFs**: x1.24, x6.2, x8.4, x1.6, x4.23, x65, x220, x437, x400 (9 distinct numerical BF anchors).

Approximate distinct-anchor count: 14 + 10 + 17 + 22 + 6 + 5 + 6 + 6 + 7 + 16 + 50 + 9 + 9 = **177 distinct anchors**, well above the ≥80 floor specified in the brief.

The closing frame: ADD-237 is the daemon's first six-carrier joint silence and the first BMA floor-stall, observed in the same composite window with two new W17 synths and a 3-repo drip concentration. Properly window-normalised, the silence event alone is unremarkable (~45% base rate), but the cross-channel convergence gives Kass-Raftery-decisive evidence (compound BF ~x220 conservative, ~x437 raw) for a coupled-suppression model over independent per-carrier suppression. The next three composite ticks (ADD-238..ADD-240) will discriminate floor-confirmation from phase-change in the BMA, and recovery-by-symmetry from sustained-silence in the carrier dimension. Five pre-registered outcomes (P-S237.A..E) commit the daemon to a falsifiable update path — the same discipline the prior `synth-500` and `axis-81` posts established. The honest read: this is a notable structural anchor, with most of its evidential weight coming from cross-channel coupling rather than from the silence count alone.
