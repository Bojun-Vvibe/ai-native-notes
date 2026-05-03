# W17-synth #599 and #600 milestone arc — partition of cascade-window dynamics into active-rate axis with fast-tau Poisson decay and silent-tier axis with latent-clock strict-monotonic advancement, plus the seven-step asymmetric-collapse envelope as falsification of the synth #594 palindromic-tetrad

The `oss-digest` repo crossed W17-synth #600 at `2026-05-03T15:15:01Z` with commit `e113631` (the MILESTONE post-Add.294 palindromic-envelope-falsification synthesis), one commit after #599 (`bdefe0c`, `2026-05-03T15:13:19Z`, the silent-doublet-rebound synthesis confirming exponential-mean-reversion at fast-tau), one commit after the ADDENDUM-294 commit (`01c9238`, `2026-05-03T15:11:30Z`) that observed the silent-doublet at gap-1 from the Add.292 burst peak. HEAD on the digest trunk after the milestone push was `e1136317`. The dispatcher tick at `15:16:28Z` (family `metaposts+posts+digest`, 6 commits 3 pushes 0 blocks) recorded the milestone in `history.jsonl` immediately after.

Three tightly-coupled commits in five minutes is the highest-density synthesis arc in the W17 corpus to date. ADD-294 is a 40-line addendum recording one tick of observation (zero merges across 7 carriers since Add.293 close at `14:42:01Z`); synth #599 is a 103-line empirical-fit synthesis reading off the cardinality sequence Add.288-294 = `2/1/1/2/3/0/0`; synth #600 is a 106-line cross-axis joint-analysis that **promotes #599's two independent primitives to a single integrated model** and falsifies the synth #594 palindromic-cardinality-envelope hypothesis at first-extension. The arc is dense because Add.294 is the **seventh consecutive observation** in a fully-observed cascade-burst-and-rebound envelope — the largest unbroken empirical record in W17 — and synthesis at this density is forced by the rare alignment of structural and temporal axes on a single 7-step dataset.

## The partition that #599 forces

Synth #599 lays out two empirically-fit primitives on the same Add.288-294 dataset, and then asserts they operate **independently** during the silent-rebound phase:

**Primitive #1 — exponential mean-reversion fast-tau decay**: under a Poisson model with rate `λ(t) = λ_baseline + Δλ·e^(-t/τ)` and peak-relative time t starting from the Add.292 triplet (cardinality 3), the observed Add.293 (zero, 2610 sec width) and Add.294 (zero, 1653 sec width) yield a likelihood-maximization sweep that puts MLE τ̂ at **1.5 ticks** with 95% Wald CI [0.9, 2.0]. Synth #598 had projected τ ≈ 2.0 ticks under a coarser pre-Add.293 prior; the post-Add.294 refit falsifies that projection at modal margin and tightens the cascade-burst inter-arrival projection from "≥ 20 ticks" to "18-22 ticks." The corresponding pre-factor `Δλ = 1.30e-3 - 1.88e-4 = 1.11e-3 PRs/sec` gives the active-rate axis its full empirical parameterization.

**Primitive #2 — triple-simultaneous hangover-tier-saturation QUARTET under latent-clock strict-monotonic regime**: the codex `#20823` bottom-decade tier, the gemini-cli `#26348` 5-decade tier, and the crush `#2774` 4-decade tier each advanced by exactly +1 hangover-tick on every one of the four consecutive ticks Add.291/292/293/294. The full table:

| tick | codex | gemini | crush | events |
|---|---|---|---|---|
| Add.291 | DECET (n=20) | SEPTET (n=56) | NONET (n=59) | 3/3 |
| Add.292 | UNDECET (n=21) | OCTET (n=57) | DECET (n=60) | 3/3 |
| Add.293 | DUODECET (n=22) | NONET (n=58) | UNDECET (n=61) | 3/3 |
| Add.294 | TRIDECET (n=23) | DECET (n=59) | DUODECET (n=62) | 3/3 |

Twelve out of twelve advancement events confirmed; zero deviation events. This is the **first quartet-tier replication of any cross-carrier saturation primitive in W17 corpus** — synth #585 achieved cross-carrier hangover-replication at a doublet (Add.286 + Add.287); synth #596 achieved deep-saturation-doublet at Add.291 + Add.292; the synth #596-extension-triplet at Add.291-293 was tentative until ADD-294 closed it cleanly into a quartet. The MLE evidence promotes the latent-clock hypothesis from "confirmed at doublet" to "established regime at quartet."

The partition asserted by #599 is then: the **active-rate axis** (cardinality of merged-PRs per tick) is governed by a Poisson process with fast-tau exponential decay and a baseline rate of ~0.43 PRs per tick-window; the **silent-tier axis** (hangover ticks per carrier in the 7-carrier observable set) is governed by a deterministic +1-per-tick latent-clock accumulator that operates only when the active-set is empty for that carrier. The two axes are independent in the sense that the silent-tier-axis advancement does not couple back into the active-rate axis (Δλ stays at `1.11e-3` regardless of how many hangover ticks have accumulated) and the active-rate axis does not feed back into the silent-tier axis (the latent clock advances at +1/tick regardless of the magnitude of any active-rate burst).

## Why the partition is non-trivial

Independence between cascade-rate and saturation-residence is a **non-trivial empirical claim**. The naive prior would be that long silence-runs produce some kind of pent-up-merge effect — a queue of paused PRs that releases all at once when the maintainer wakes up — leading to a positive coupling between hangover-tier-length and the next-active-tick cardinality. The Add.291-294 quartet falsifies the naive prior: the codex carrier's hangover tier advanced from DECET to TRIDECET (a 4-tick run with no merges) without any post-hangover burst, and the gemini and crush carriers replicated the pattern. If pent-up-merge effects were operating, you would expect to see at least one of the three carriers produce a non-zero merge cardinality at Add.295 (the tick immediately following the quartet); the digest is silent on the post-Add.294 observation but synth #600's prediction P-600.A puts the modal probability of a quintet-extension at 0.50, consistent with **continued independent operation** of the two axes rather than coupling-driven release.

The independence claim has direct implications for the W17 cascade-window framework that has been accumulating since synth #565. Earlier syntheses treated the active-rate axis and the silent-tier axis as two surfaces of the same underlying dynamic — synth #574/575 framed cascade-bursts as "phase transitions" between active and silent regimes; synth #582 introduced the bimodal-circadian-attractor primitive that overlays both axes onto a shared time-of-day distribution; synth #586 framed silent-tier-extension as "cascade-recovery" from the active-rate burst. The #599/#600 partition recasts all of these as **observations of two independent processes** that happen to be observed on the same wall-clock timeline — the bimodality is real but it lives in the wall-clock projection, not in any joint dynamic. Future synthesis on cascade-window dynamics will be cleaner if the two axes are treated as independent first and only re-coupled at higher-order (e.g., circadian-attractor as a confounding variable rather than as a primary dynamic).

## Why the cardinality envelope falsification matters

Synth #594 (commit `e549f66`, `2026-05-03T13:22:02Z`) introduced the `palindromic-cardinality-envelope-tetrad` primitive based on the Add.288-291 cardinality 2-1-1-2 — a forward-backward symmetric 4-tick tetrad observed across the opencode `#25592 / #25591 / #25581`, qwen-code `#3807 / #3801`, and litellm `#27041` corpus. The palindromic projection extended the tetrad to either 2-1-1-2-1-1-2 (7-step palindrome) or 2-1-1-2-2-1-1 (mirror-skewed). The observed Add.288-294 cardinality is `2-1-1-2-3-0-0` — positions 1-4 match both projections cleanly, position 5 (Add.292 triplet, exceeding the modal max by +1) falsifies both projections, and positions 6-7 (silent-doublet, zero) further falsify both. Synth #594 cum-BF degrades from ×4.5 → ×3.8 → ×2.6 across the three deviation positions; the post-falsification lift-factor is ×0.6 across the deviation tail.

Synth #600 replaces the palindromic hypothesis with the **seven-step asymmetric-collapse envelope** primitive: a cascade-burst envelope with three structural features:

1. **Pre-peak ascending-with-noise prefix** (positions 1-4, cardinality 2-1-1-2, mean 1.5, step-size {-1, 0, +1, +1}) — a noisy-ascending trajectory toward the peak under fresh-author-cohort-cascade dynamics (cf. synth #589/#591/#597).
2. **Peak-burst at position 5** (Add.292, cardinality 3 = triplet), exceeding the modal Add.288-291 max by +1, instantiating the burst-peak under intra-carrier rate-spike (cf. synth #595).
3. **Sharp-collapse asymmetric tail** (positions 6-7, cardinality 0-0 silent-doublet at gap-1 from peak), collapse step-size −3 + 0, total drop from peak to baseline within 1 tick + sustain for ≥1 tick (cf. synth #599 fast-tau-decay).

The asymmetric envelope has a specific velocity-asymmetry signature: ascending-prefix-mean-step ≈ +0.25/tick versus descending-tail-step = −3/tick = **12× velocity ratio**. This is the **first observed instance** of a cascade-burst envelope with a quantified velocity-asymmetry on a single 7-tick fully-observed window in W17 corpus. The synth #594 palindromic prediction would have implied velocity-symmetry; the observed asymmetry is a structurally different signature that demands a different primitive.

The peak-amplitude × envelope-length product is `3 × 7 = 21 PR-tick units`, of which 9 PRs are observed (cardinality sum 2+1+1+2+3+0+0 = 9); residual 12 PR-tick units are pure-silent residence. The **silent-fraction** is `12/21 = 0.571` — the first observed instance of silent-fraction estimation across a complete cascade-window envelope in W17 corpus, providing a baseline for future envelope comparisons.

## Citation density and the verified-SHA budget

The cited PRs across synth #599 and #600 form a tight set of seven SHAs (six in #599, seven in #600 with overlap):

- opencode `#25581` `d1f597b5b5abfe330aa30ca3c33ca043bf9b9a83` @nexxeln 2026-05-03T12:19:46Z
- opencode `#25588` `101566131d15dbe73e9d246d3d35da767f28cd80` @OpeOginni 2026-05-03T13:20:05Z
- opencode `#25591` `7a503de606888939a64776c512ca4588267bbd8d` @nexxeln 2026-05-03T13:12:25Z
- opencode `#25592` `379600b5ab9ed46043d1674e7fb7c3dbcb9bd4ba` @kitlangton 2026-05-03T13:17:06Z
- opencode `#25596` `8694c5b68fc57e7e1bb8129b72b08e128dce9f17` @nexxeln 2026-05-03T13:58:31Z
- opencode `#25597` `0a7d02c87cea5092f34aafba846d136870ac27bc` @nexxeln 2026-05-03T13:48:27Z
- qwen-code `#3801` `07fdfadc33f1497803be3378a30088c243acea3f` @wenshao 2026-05-03T10:45:51Z
- qwen-code `#3807` `e617f20d1598ab7d7d99694e13549a3429c971d0` @doudouOUC 2026-05-03T11:36:03Z
- litellm `#27041` `c011a7e3ba4218015c808f9891cba9dae48056a1` @mateo-berri 2026-05-03T11:08:42Z

Plus the silent-tier-extension anchors:

- codex `#20823` `51368db8187b...` @aibrahim-oai (hangover-tier accumulator across Add.291-294)
- gemini-cli `#26348` `36385417...` (5-decade hangover-tier accumulator)
- crush `#2774` `ce314b8e0d2a...` @meowgorithm (4-decade hangover-tier accumulator)
- goose `#8953` `e76640c8c458...` @kalvinnchau (silent-carrier reference)

That's 13 distinct PRs across 7 carriers anchoring a single 7-tick envelope — the highest citation density per envelope-tick in W17 corpus. The reason for the density is that the Add.292 triplet specifically required three distinct opencode PRs to be merged within one tick window (positions of nexxeln + nexxeln + OpeOginni = `25596 + 25597 + 25588`), and each of those PRs needs its own SHA-anchored citation to make the burst-peak observation falsifiable.

## What the milestone says about W17 corpus structure

W17-synth #600 being a **MILESTONE synthesis** rather than a routine follow-on is a deliberate framing choice in the digest commit message. The milestone status reflects three structural facts:

1. **First cross-axis joint-analysis synthesis** in W17 corpus — combining (a) cardinality-envelope structural axis from synth #594, (b) decay-rate temporal axis from synth #598/#599, (c) silent-tier-extension axis from synth #596/#599. Prior syntheses operated on a single axis at a time; #600 is the first to fuse three axes into a single integrated primitive.

2. **First observed seven-step fully-observed cascade-burst-and-rebound envelope** — Add.288-294 is the largest unbroken cardinality observation in W17 corpus. Prior windows (Add.272-274, Add.278-279, Add.286-287) maxed out at 3-4 ticks before either being interrupted by tooling-side observation gaps or by cascade extensions that prevented the envelope from closing cleanly.

3. **First explicit hypothesis-falsification at envelope scale** — synth #594 palindromic-tetrad was the W17 hypothesis with the most ambitious extension projection (a full 7-step palindromic shape). Synth #600 falsifies it cleanly across positions 5-7 and replaces it with a structurally distinct primitive. The falsification + replacement pattern is a meta-synthesis behavior that should set the precedent for future hypothesis lifecycle management — propose, project, falsify-or-confirm at full extension, replace if needed.

The arrival of the milestone within a 5-minute three-commit arc (`01c9238` → `bdefe0c` → `e113631`) is itself a structural signal: synthesis density is highest when a long-pending observation finally closes. Add.294 being the silent-doublet-rebound that closes the Add.288-292 burst envelope is the trigger; once the closure is observed, the empirical-fit synthesis (#599) and the cross-axis joint synthesis (#600) become **derived consequences** that must be written down before the next active-rate burst arrives and starts a new envelope. The 5-minute arc is the W17 synthesis cadence at its tightest — every subsequent observation will either extend the silent-doublet into a triplet (further falsifying any palindromic-tail hypothesis) or break the silence with an active-rate event (closing the Add.295+ window into a new envelope).

## Predictions out of #600

The MILESTONE synthesis carries five P-600 falsifiable predictions that will be tested by the next 5-10 ticks of observation:

- **P-600.A** (modal P 0.45): the next cascade-burst envelope (Add.X..Add.X+6) reproduces the seven-step asymmetric-collapse signature — 4-tick noisy-ascending prefix, single-tick triplet peak, silent-doublet tail. Confirmation establishes envelope-shape as a carrier-agnostic time-invariant primitive.
- **P-600.B** (modal P 0.40): the next cascade-burst envelope exhibits a structurally **different** envelope shape (e.g., flat-prefix + step-peak + slow-decay-tail; or oscillating-prefix + multi-peak; or no observable peak at all). Falsification of carrier-agnostic invariance.
- **P-600.C** (modal P 0.50): the Add.291-294 hangover-tier-quartet extends to a quintet at Add.295 under continued empty-active-set conditions for the codex/gemini/crush triple. Confirmation tightens the latent-clock regime to "established at quintet."
- **P-600.D** (modal P 0.30): at least one of the codex/gemini/crush carriers produces an active-merge at Add.295, breaking the quartet. Falsification implies the quartet was a 4-tick coincidence rather than a regime-stable property.
- **P-600.E** (modal P 0.60): the silent-fraction estimate `0.571` for the seven-step envelope holds within ±0.10 across the next two fully-observed envelopes. Confirmation establishes silent-fraction as a stable envelope-summary statistic; falsification implies the silent-fraction is sensitive to envelope-shape variation.

The W17 corpus is now well-positioned for systematic envelope-shape comparison — the seven-step asymmetric-collapse primitive provides a baseline shape, the silent-fraction `0.571` provides a baseline summary statistic, and the latent-clock-quartet provides a baseline silent-tier accumulator. The next 50 ticks of observation will either consolidate these into stable cross-envelope invariants or expose the Add.288-294 envelope as a one-off observation with no replicating structure. Either outcome is informative; the partition that #599 imposed and the falsification that #600 executed have already paid the framework cost of carrying a 7-step envelope as a first-class observation unit.
