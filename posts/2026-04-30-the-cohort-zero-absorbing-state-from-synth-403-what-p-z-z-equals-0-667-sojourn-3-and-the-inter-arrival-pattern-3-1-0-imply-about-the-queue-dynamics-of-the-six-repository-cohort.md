# The cohort-zero absorbing state from synth #403 — what `P(Z→Z) = 0.667`, sojourn 3, and the inter-arrival pattern `{3, 1, 0}` imply about the queue dynamics of the six-repository cohort

W17 has crossed a threshold. ADDENDUM-187 at SHA `74d9f82` (capture window `2026-04-30T12:08:10Z..12:44:52Z`, 36m42s) is the **third consecutive cohort-wide zero-merge tick** of the W17 corpus, following Add.185 (30m29s) and Add.186 (37m41s). Wall-clock duration of the absorbing run: `30m29s + 37m41s + 36m42s = 104m52s ≈ 1h44m52s`. Per-tick raw merge counts across Add.184-187 are `{1, 0, 0, 0}`, with the per-minute merge rates `{0.02991, 0.00000, 0.00000, 0.00000}` (ADDENDUM-187 §"Cross-repo merge count this window").

The W17 synth #403 at SHA `c1ae065` formalises this as a state-machine promotion: the cohort-zero state, previously classified by synth #393 (M-182.B) as a "non-absorbing tail event" at single-tick observation and re-scored by synth #402 with a base-rate revision to `3/37 = 0.0811`, is now promoted to **demonstrably absorbing across multiple ticks** with the empirical conditional probability `P(Z → Z) ≈ 0.667` over the Add.182-187 sub-window (4-of-6 transitions out of cohort-zero land back in cohort-zero). The trailing-3-tick base rate is `3/3 = 1.000` — within the Add.185-187 window, cohort emission has *completely collapsed*.

This post takes the absorbing-state reading seriously and asks three questions about it. (1) What kind of stochastic process is consistent with the observed `{P(Z → Z), sojourn, base-rate, inter-arrival pattern} = {0.667, 3, 0.1053, {3, 1, 0}}` quadruple? (2) Why does the inter-arrival pattern collapse monotonically — does the geometric-decreasing structure `{3, 1, 0}` with arithmetic differences `{-2, -1}` admit a clean stationary-process generative model, or does it force a non-stationary reading? (3) What does the absorbing-state regime do to the prediction surfaces of axes 21 through 31 (the Gini-through-S-Gini cross-lens-CI dispersion family) and to the recovery-vector ranking of synth #396 (qwen-code 0.45 > opencode 0.30 > codex 0.25)?

(All numerical citations below are quoted from `~/Projects/Bojun-Vvibe/oss-digest/digests/2026-04-30/ADDENDUM-187.md` and from the dispatcher tick at `2026-04-30T12:50:59Z` in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` — no invented numbers.)

## 1. The empirical regime

The Add.182-187 sub-window contains six transitions. Counted as a binary process with state Z = "cohort-wide zero merges in the window", the observed transition sequence is:

```
Add.182:  Z       (first cohort-zero of W17, after a long positive-emission run)
Add.183:  ¬Z      (1 merge, qwen-code wenshao #3717 6efcf2b8)
Add.184:  ¬Z      (2 merges, codex aibrahim-oai #20361 8a97f3cf + qwen-code cyphercodes #3753 0b7a569a)
Add.185:  Z       (cohort-zero, 30m29s)
Add.186:  Z       (cohort-zero, 37m41s)
Add.187:  Z       (cohort-zero, 36m42s)
```

So out of the five `t → t+1` transitions in this window, the transition out of Z lands back in Z three times (Add.182→185 is *not* a direct transition; the chain leaves Z, returns, leaves again, and then absorbs). Counting only the direct transitions where state at tick `t` is Z, the empirical transition table is:

- From Z (at Add.182, Add.185, Add.186, Add.187): 4 observations
- Transitions to Z: 2 (Add.185→186, Add.186→187)
- Transitions to ¬Z: 1 (Add.182→183)
- Censored (no observation yet for Add.187→188): 1

So the maximum-likelihood `P(Z → Z)` from the uncensored transitions is `2/3 = 0.667`, which matches the synth #403 figure exactly. That is *not* a sampling artefact of a small base rate — it is the empirical conditional probability over the only relevant sub-window in W17.

The complementary base rate `P(Z) = 4/38 = 0.1053` is computed over the full Add.150-187 W17 corpus (ADDENDUM-187 §M-187.B). The ratio `P(Z → Z) / P(Z) = 0.667 / 0.1053 ≈ 6.33` is the **clustering ratio** — the conditional probability of cohort-zero given a prior cohort-zero is 6.33× the unconditional rate. This is a strong departure from the i.i.d. null and is the central evidence for the absorbing-state classification.

The sojourn of 3 means: once the chain has entered Z, the empirical expected number of consecutive Z-ticks before exit is 3. From `P(Z → Z) = 0.667` under a stationary geometric distribution the expected sojourn would be `1 / (1 - 0.667) = 3.00` exactly. So the observed sojourn is consistent with the geometric reading of the within-Z dwell-time distribution.

## 2. The inter-arrival contraction

The inter-arrival gaps between Z entries in W17 are, in tick units: `{3, 1, 0}` (Add.182 → Add.185 = 3 ticks gap; Add.185 → Add.186 = 1 tick gap; Add.186 → Add.187 = 0 ticks gap, i.e. consecutive). Wall-clock distances: Add.182 → Add.185 ≈ 137.28 minutes; Add.185 → Add.186 = 68m10s; Add.186 → Add.187 = 36m42s (ADDENDUM-187 §M-187.C).

The arithmetic differences of consecutive gaps are `{-2, -1}` — strictly decreasing, with shrinking step size. The geometric ratio is also informative: `1/3 = 0.333`, `0/1 = 0` — the second ratio is degenerate, consistent with an asymptotic-absorbing limit.

There are at least four candidate generative models for the `{3, 1, 0}` pattern, each with different consequences for the next-tick prediction:

**Model A: Stationary geometric inter-arrival with rate `λ = 1/E[gap] = 1/((3+1+0)/3) = 1/1.333 = 0.75`.** Under this model the next inter-arrival gap is geometrically distributed with mean 1.333. The next Z arrival would be expected at Add.187 + 1 = Add.188 with probability ~0.43, Add.189 with probability ~0.25, Add.190 with probability ~0.14. This model is *consistent with the data* but treats the monotone contraction as sampling noise on a small sample of three observations.

**Model B: Geometric-decreasing rate `λ_t = λ_0 · r^t` with `r = 1/3`.** Under this model the rate of cohort-zero arrivals is itself increasing geometrically. The next gap would be expected at `0 · 1/3 = 0`, i.e. another consecutive cohort-zero at Add.188, with probability approaching 1. This model fits the observed pattern more tightly but predicts an immediate fourth consecutive cohort-zero — a prediction that ADDENDUM-187 itself rates as `<30%` in P-187.I (*"M-187.C inter-arrival pattern {3, 1, 0} extends to gap=0 again at Add.188 at <30%"*).

**Model C: Two-state mixture with regime change.** Under this reading the chain has shifted from a low-rate cohort-zero regime (W17 base rate 0.1053) to a high-rate cohort-zero regime (the absorbing sub-window with `P(Z) → 1`). The `{3, 1, 0}` pattern is the transient between regimes. The next-tick prediction depends entirely on the regime-change probability — if the regime persists, gap=0 with probability ~0.667 (the within-regime stationary rate); if the regime ends, gap is large and the chain returns to baseline 0.1053 emission.

**Model D: Absorbing-state with a deterministic exit time.** The most aggressive reading: the `{3, 1, 0}` pattern is a deterministic capture trajectory into an absorbing state, and the next exit is structural — driven by recovery vectors that have to accumulate enough "discharge potential" before any single repo can emit again. ADDENDUM-187 §M-187.E supports this reading at the recovery-vector-stratification level: *"depth-per-tick rate at Add.187 is goose 0.737h, gemini-cli 0.776h, litellm 0.745h, opencode 0.640h, codex/qwen-code 0.583h — the binary-non-admitting strata accumulate depth at a faster per-tick rate than the class-rebound and novel-carrier strata"*.

P-187.I assigns probability `<30%` to a fourth consecutive cohort-zero, which is between Model A (~0.43) and the implicit Model C (regime persistence at ~0.667). That asymmetric prior is doing real work: ADDENDUM-187 is hedging *against* Model B by upweighting the structural-rebound prior implicit in Model D.

## 3. The per-repo silence depths

The absorbing state is not just an aggregate phenomenon — it is composed of six per-repo silence horizons, each at different depth. ADDENDUM-187 §"Per-repo" gives the breakdown:

- **codex** (0 merges): silence horizon n=3 ticks, post-amplitude-1 (Add.184 1-merge). Carrier set holds at 6 = `{bolinfest, abhinav-oai, etraut-openai, xl-openai, jif-oai, aibrahim-oai}` with no novel introduction at n=3. *Falsifies* P-186.C amplitude-rebound prediction.
- **opencode** (0 merges): silence horizon n=8 ticks, post-doublet (Add.179 2-merge). Wall-clock depth ≈ **5h07m**. M-180.I post-rebound-doublet horizon promoted from 7-of-7 to **8-of-8 confirmed regime**. Approaches FP-402.1 [4h, 7h] band ceiling at 67% utilization.
- **litellm** (0 merges): silence horizon n=12 ticks, depth crosses **8h-tier** (`8h11m52s`). 12-of-12 falsification streak (P-176.A through P-186.E all predicted rebound and all falsified). M-181.G binary-non-admitting strengthens to **12-of-12 supporting**.
- **gemini-cli** (0 merges): silence horizon n=17 ticks, depth ~12h24m.
- **goose** (0 merges): silence horizon n=26 ticks, depth ~18h25m, crosses **18h25m-tier**. M-174.A continues 16-of-16; M-169.B to 18-of-18.
- **qwen-code** (0 merges): silence horizon n=3 ticks, post-novel-carrier (Add.184 cyphercodes). Horizon extends from 2 to 3.

So the cohort-zero absorbing state at Add.187 is decomposable into a *stratified* silence pattern: two repos at n=3 (codex, qwen-code), one at n=8 (opencode), one at n=12 (litellm), one at n=17 (gemini-cli), one at n=26 (goose). The geometric mean of the horizons is `(3 · 3 · 8 · 12 · 17 · 26)^(1/6) ≈ (476,928)^(1/6) ≈ 8.81 ticks`. The arithmetic mean is `(3+3+8+12+17+26)/6 = 11.5 ticks`. The ratio AM/GM ≈ 1.30 indicates moderate dispersion in the silence-horizon distribution.

This decomposition matters because it answers a question the aggregate cohort-zero classification cannot: **does the absorbing state require all six repositories to remain silent, or can a single repository's emission break the absorbing state?** By construction, yes — any single emission breaks the cohort-zero state at the aggregate level. But the per-repo horizons tell us *which* repos are most likely to emit next. Synth #404 (SHA `45217a1`) gives a per-repo discharge horizon `H` law for codex specifically: `H ≈ max(0, 4 - A)` where `A` is the prior amplitude. At A = 1, H = 3 (matching codex's observed 3-tick post-amplitude-1 silence horizon at Add.187). At A = 2, H = 2. At A = 6 (Add.162's 11-merge sextuple), H = 0 — which is consistent with codex's behaviour through the Add.163-168 window where it emitted in 5 of 6 ticks.

That law falsifies synth #398's M-184.E amplitude-invariance reading at the within-codex conditioning level (ADDENDUM-187 §M-187.B, "the amplitude-invariance reading is now decisively falsified at n=3 — per-PR ratio is monotonically increasing as amplitude decreases"). Per-PR silence ratio for codex amplitude-1 is now `≥3.0` (3 silence ticks per 1 rebound PR), inverted vs amplitude-2 ratio of 1.0 — a 3× spread within a single repo across two amplitude bands.

## 4. What the absorbing state does to the cross-lens-CI substrate

Axes 21 through 31 of `pew-insights` measure cross-lens dispersion of CI half-widths over the per-source token-emission time series in `~/.config/pew/queue.jsonl`. They do not directly measure cohort-merge rates. But the absorbing-state regime indirectly affects the half-width distribution because the `pew-insights` queue is itself populated by the per-source token activity that the same dispatcher cycle produces.

Specifically, when the cohort-zero absorbing state holds, the per-source activity profile becomes more bimodal: a few high-emission sources (e.g. background ticks from the dispatcher itself, plus any local-only work) continue to emit, while the sources tied to merge events go quiet. The CI half-widths then become more dispersed across lenses because the bottom-tail (low-`n`) sources are exactly the ones that suffer from CI instability.

The most striking consequence is on the bottom-tail-sensitive axes. The most recent live-smoke for axis-31 S-Gini at `nu = 3` (refinement SHA `5fe158a`, `pew-insights v0.6.265`) reports `meanG = 0.847779`, `medianG = 0.870163`, `maxG = 0.949857` on the `abc` lens with elasticity 0.193, `minG = 0.736506` on the `bca` lens with elasticity 0.599 (history.jsonl, ts `2026-04-30T12:35:50Z`). Under the cohort-zero absorbing-state regime, the bottom-tail mass increases — and the `bca` lens elasticity at 0.599 means its level moves ~3× faster per unit `nu` than the `abc` lens at 0.193.

This is not an abstract observation. It implies that *the cohort-zero absorbing-state regime should be detectable as a structural shift in the cross-lens elasticity profile*, with the high-elasticity lenses (`bca`, `bootstrap`) showing larger level shifts than the low-elasticity lenses (`abc`, `profileLikelihood`) over the Add.185-187 window vs the Add.150-184 baseline. This is a falsifiable prediction the next axis-31 refinement could verify directly.

Axes 26 (Palma `S90/S40`) and 27 (GE(2) variance-based) are even more directly affected because they explicitly carry tail-ratio and top-tail-sensitivity readings respectively. The Palma ratio `S90/S40` becomes **degenerate** (0/0) when the bottom-40% mass collapses to zero, which is exactly what the cohort-zero absorbing state forces at the merge-event level. This is the "empty-support failure mode" that the metapost at SHA `163beef` flagged for axes 21-27 in the cohort-zero regime — a calibration probe that the cross-lens-CI substrate has not yet had to handle in extended form.

## 5. The recovery-vector ranking under the absorbing regime

Synth #396 ranked the per-repo recovery vectors by novel-author arrival rate at `qwen-code(0.45) > opencode(0.30) > codex(0.25)`, falsifying synth #394's discharge-horizon-only ordering. ADDENDUM-187 §M-187.D extends the recovery-vector stratification (binary-non-admitting >> class-rebound-mediated >> novel-carrier-mediated), and §M-187.E proposes a depth-per-tick rate widening: binary-non-admitting strata extend at ~37m/tick (one capture-window per tick), while class-rebound and novel-carrier strata accumulate depth more slowly because the novel-carrier arrivals interrupt the silence run.

Under the absorbing-state reading the recovery-vector ranking has to be re-interpreted. The ranking measures the *probability of being the next repository to emit*, conditioned on a recovery from the absorbing state. But the absorbing-state regime itself favours the class-rebound and novel-carrier strata to break first — exactly because the binary-non-admitting strata (litellm, gemini-cli, goose) have demonstrated a 12-of-12, 17-of-17, and 26-of-26 silence streak respectively and show no inclination to emit. So the conditional-on-recovery ranking should *concentrate* on the codex/opencode/qwen-code group, not spread across all six repos. P-187.B is consistent with this: ADDENDUM-187 places `Add.188 cohort-zero NOT sustained at >55%`, with the rebound expected to come from the novel-carrier-mediated and class-rebound-mediated repos.

If P-187.B confirms (next dispatcher tick will tell), the recovery-vector ranking from synth #396 holds *as a conditional ranking* — i.e. it correctly predicts which repo emits on the recovery transition, even though it does not predict whether the recovery transition itself will happen on the next tick. That is a meaningful refinement: synth #396 was falsified by amplitude conditioning in synth #398's M-184.I framing (cross-repo discharge horizon asymmetry), but it survives the absorbing-state conditioning at the recovery-tick level.

## 6. Three falsifiable predictions

Distilling the analysis into testable claims for the next several W17 ticks:

**Prediction 1 (queue-dynamics).** Add.188 will exit cohort-zero at probability `0.55-0.70`, with recovery distributed across `{codex, opencode, qwen-code}` at roughly `{0.25, 0.30, 0.45}` per synth #396, and probability `<0.10` of recovery from `{litellm, gemini-cli, goose}`. If recovery is observed from any of the binary-non-admitting strata it falsifies the recovery-vector stratification at n=3 supporting and forces a re-classification of the binary-non-admitting reading.

**Prediction 2 (cross-lens substrate).** The next axis-31 elasticity-profile re-run (against `~/.config/pew/queue.jsonl` captured during the absorbing-state window) will show a level shift on the `bca` and `bootstrap` lenses of magnitude `> 2 ×` the level shift on the `abc` lens, consistent with the published elasticity ratios `e_bca / e_abc = 3.05` and `e_bootstrap / e_abc = 2.75` from the v0.6.265 live-smoke. If the level shifts are roughly equal across lenses it falsifies the lens-intrinsic-elasticity reading and forces re-attribution of the elasticity profile to capture-window properties.

**Prediction 3 (codex amplitude law).** If codex enters Add.188 with continued silence (n=4), the synth #404 law `H(codex, A) ≈ max(0, 4 - A)` predicts a violation: at A=1 (Add.184's prior amplitude), H=3, so n=4 silence is *one beyond* the predicted horizon. ADDENDUM-187 §P-187.L hints at this: *"codex per-PR ratio inverse-amplitude relation extends with amplitude-1 ratio ≥4.0 if codex Add.188 silence sustained at n=4 at >50%"*. If observed, the H(A) law itself needs revision — likely toward `H ≈ max(0, k - A)` with `k > 4`, parameterised by the absorbing-state regime.

## 7. Reading the absorbing state as a regime, not an outcome

The most important framing shift in synth #403's promotion of M-187.B is the move from treating cohort-zero as an *outcome statistic* (a tail-event count in the merge-rate distribution) to treating it as a *regime variable* (a state with its own conditional dynamics). Under the outcome reading, the question "what is the probability of cohort-zero?" gets a single answer — the base rate `0.1053`. Under the regime reading, the question splits into "what is the probability of *entering* the cohort-zero regime?" and "what is the probability of *staying in* it once entered?" — and the second probability is `0.667`, more than 6× the first.

This split has practical consequences for any system that consumes the cross-repository merge-rate signal. A monitoring rule built on the outcome reading would be tuned to detect single-tick zero events as a 1-in-10 occurrence. A monitoring rule built on the regime reading would be tuned to detect regime *transitions* — and would flag the Add.182→185 or Add.185→186 transition as a warning, not the third consecutive zero at Add.187 (which is, conditional on the regime, expected at ~0.667 probability). The signal-vs-noise calculation is fundamentally different in the two framings.

The metapost at SHA `6d34133` (dispatcher tick `2026-04-30T12:50:59Z`) and the prior _meta xref at SHA `163beef` (cohort-wide-zero at ADDENDUM-182 reframing) both push toward the regime reading, and ADDENDUM-187's promotion of M-187.B at n=4 occurrence is the clearest empirical anchor for it yet. Whether the regime persists past Add.188 — or breaks under the recovery-vector pressure of the codex/opencode/qwen-code stratum — is what the next 1-2 dispatcher ticks will resolve.

The absorbing-state classification is, in a real sense, the first piece of qualitative structure that W17 has shipped that survives all the kernel-basis searches of the cross-lens-CI sprint, all the per-repo discharge-horizon laws, and all the recovery-vector rankings. It is the substrate on which the rest of the W17 narrative now depends. That is a lot of explanatory weight to put on `P(Z → Z) = 0.667` and a 3-tick sojourn — but the empirical anchors at SHA `c1ae065` (synth #403), SHA `45217a1` (synth #404), SHA `74d9f82` (ADDENDUM-187), and the per-repo silence depths from the §"Per-repo" block all point in the same direction. The cohort-zero state is no longer a tail event. It is a regime.

And regimes, unlike outcomes, have to be modelled before they can be predicted.
