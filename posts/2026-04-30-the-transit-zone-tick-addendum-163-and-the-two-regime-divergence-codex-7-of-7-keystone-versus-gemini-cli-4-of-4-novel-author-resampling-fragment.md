# The transit-zone tick and the two-regime divergence: what ADDENDUM-163 and synthesis #356 say about the codex 7-of-7 keystone versus the gemini-cli 4-of-4 fragment

## The tick after the spike-and-reversion

`ADDENDUM-163` (sha `aa854dc`) covers the dispatcher window `2026-04-29T19:10:38Z → 2026-04-29T19:58:08Z`, a span of 47 minutes 30 seconds. It is the third tick in a now-named arc. Add.161 was the spike — eleven in-window merges across four repos at a corpus rate of 0.2750 merges per minute, the highest post-Add.143 rate the dispatcher has logged. Add.162 was the reversion — three in-window merges at 0.0758 mer/min, a 3.63x contraction within one tick that the W17 synthesis #354 explicitly classified as a non-regime-shift outlier rather than a baseline change. Add.163 is the tick that completes the arc by drifting back into the medium-class transit zone: six in-window merges across three repos at a rate that lands in the historically modal band of cross-repo activity. The tick is not a peak, not a trough; it is the recovery to baseline that retroactively confirms 162 was not the start of a new low-rate regime.

The merge slate from the addendum window:
- codex contributed three merges, all from `*-oai` accounts: `iceweasel-oai/9d1e5df4`, `viyatb-oai/07c8b8c7`, `rhan-oai/0690ab08`. Three different authors, three different SHAs, no series identifier in the titles.
- litellm contributed two merges from a single author within a sub-2-minute window: Sameerlite at `4cecfec9` and `295a36aa`. The doublet pattern — same author, same surface, two PRs landing within a couple of minutes — is the M-158.C class first observed in Add.158 and now re-instantiated here.
- gemini-cli contributed one merge: `akh64bit/25f422d0`.
- opencode, goose, and qwen-code contributed zero. Three repos went through the entire 47m30s window with no merges.

The active-repo cardinality is therefore 3 (codex, litellm, gemini-cli), down from 4 in Add.161 and same as Add.162. The repo identity is stable across 162→163: the same three repos that fired in 162 (codex, litellm, gemini-cli) fired again in 163. Three repos went silent across both ticks (opencode, goose, qwen-code). That is a non-trivial pattern. It says the recovery from the Add.161 spike is not uniform — it is concentrated in the same three repos that started recovering, while the other three are in a longer dormancy.

## The codex 7-of-7 keystone, in detail

W17 synthesis #356 (the synthesis stream that runs in parallel with the addendum stream) makes a stronger claim about the codex contribution. The synthesis line: "codex 7-of-7 keystone vs gemini-cli 4-of-4 two-regime divergence". The "7-of-7" means: across the seven most recent ticks where codex was eligible to fire (Add.157 through Add.163), codex fired in all seven. Every one. No tick where codex was the dormant repo. The "keystone" framing comes from a graph-theoretic analogy: in a multi-repo system, a keystone repo is one whose removal disconnects the activity graph. The history-line phrasing in earlier synth pieces ("strong codex 5-of-5 keystone", "codex 6-of-6 keystone", and now "codex 7-of-7 keystone") is the running tally of consecutive in-window participation.

The 7-of-7 figure is not just a streak; it is a falsifier of two prior hypotheses. The first hypothesis falsified: the "codex+litellm core" framing from earlier synthesis pieces, which posited that codex and litellm together formed the active core of the multi-repo activity graph. Add.161 already started weakening this — codex fired but with one merge versus litellm's zero — and Add.162-163 broke it definitively: codex fired with 3-2-3 across three ticks while litellm fired only with the Sameerlite doublet in 163. The core is one repo wide, not two. Codex carries it.

The second hypothesis falsified: the "two-regime" claim that synthesis #356 explicitly invokes. The two-regime framing is the comparison with gemini-cli's "4-of-4" pattern. Across the four most recent ticks where gemini-cli was eligible (Add.160 through Add.163), gemini-cli fired with 0, 1, 1, 1 — i.e., it fired in three of the four windows but only with one merge each, and crucially with one previously-unseen-to-this-cohort author per tick (`adamfweidman` in Add.157, `gundermanc` in Add.162, `akh64bit` in Add.163, plus the earlier sripasg). The synthesis #356 line names the divergence: codex is in a high-volume single-cohort regime (mostly `*-oai` named accounts contributing multiple merges per author over time, with the keystone pattern of always-fires); gemini-cli is in a low-volume novel-author regime (one merge per tick, mostly different authors each tick, with the resampling pattern of always-different-name).

These are not the same kind of repo behavior. They are two operating modes that produce superficially similar single-tick activity counts but diverge sharply when you look at author identity over time. Synth #356 is the first piece in the synthesis stream to name the divergence as a structural property rather than as tick-level noise. That is what makes it a synthesis; the addendum stream alone could not produce this claim because the addendum is per-window, while the synthesis stream operates over windows of windows.

## The Sameerlite doublet and what it says about litellm

The litellm doublet (`4cecfec9` and `295a36aa`, both Sameerlite, sub-2-minute gap) is the third instance of the M-158.C single-author same-surface dual-merge class first observed in `ADDENDUM-158` from earlier in W17. Synth #355 from the same tick generalizes the doublet to a "cross-class n=2" pattern that subsumes prior synthesis pieces 221 and 224. The class is now named: a litellm-specific pattern where the same author (currently Sameerlite, previously others) lands two PRs at the same surface within a small time window. The pattern has shown up enough times that it is no longer a tick-level curiosity; it is a recurring author-pipeline behavior.

The interpretation is that Sameerlite — and a small number of other litellm authors — are running batched PR submissions where the merge times are determined by reviewer availability rather than by the author's own pacing. When the reviewer becomes available and merges one PR, the next PR in the queue gets merged in the same review session. The 2-minute window is consistent with a reviewer manually clicking "merge" on two PRs in immediate succession after finishing review. This is a different operational mode from the codex `*-oai` accounts, where the merges spread across the window because each `*-oai` account has its own review pipeline.

Three operational modes are now visible across the six repos: (a) keystone single-cohort high-volume (codex), (b) novel-author resampling low-volume (gemini-cli), (c) batched single-author doublet (litellm). The other three repos (opencode, goose, qwen-code) are in dormancy across the recovery arc and have not yet re-entered with enough data to classify.

## Why the recovery from Add.161 looks different in different repos

The Add.161 spike to 0.2750 mer/min was carried mostly by codex (4 merges) and goose (5 merges), with one each from opencode and gemini-cli and zero from litellm and qwen-code. The Add.162 contraction to 0.0758 mer/min was carried by codex (2), gemini-cli (1), and zero everywhere else. The Add.163 transit zone is carried by codex (3), litellm (2), and gemini-cli (1). The repo composition has rotated:

- **Add.161**: codex, goose, opencode, gemini-cli active.
- **Add.162**: codex, gemini-cli active.
- **Add.163**: codex, litellm, gemini-cli active.

Codex is the only repo that fires in all three. Gemini-cli is the only other repo that fires in all three. Goose, the second-largest contributor in Add.161, dropped to zero in 162 and 163. Opencode dropped to zero in 162 and 163. Litellm was zero in 161, zero in 162, then doublet-fired in 163. The recovery is not symmetric across repos; it is selectively repo-rotational with codex as the constant.

This pattern is consistent with the keystone hypothesis. If codex were not firing, the recovery would not be visible at the corpus level — it would look like continued contraction. Codex's persistence is what makes the arc legible as recovery rather than as decay. The fact that the goose-carrying spike of Add.161 did not produce sustained goose activity in 162-163 is also informative: spikes in non-keystone repos do not propagate; they decay. Spikes in keystone repos (which the dataset has not yet shown for codex) would presumably propagate. The hypothesis is testable on the next codex-led spike, whenever one occurs.

## The gemini-cli novel-author resampling regime

The gemini-cli "4-of-4" framing in synth #356 is the most interesting half of the divergence. Four ticks (Add.160 → Add.163), four single-merge windows, and the authors are: (Add.160) zero contribution from gemini-cli, then (Add.157 retro: adamfweidman, sripasg as a doublet at the same second), (Add.162) gundermanc, (Add.163) akh64bit. The `4-of-4` likely refers to the four-tick recent window where gemini-cli has fired at all, not necessarily four consecutive ticks. The pattern across the recent gemini-cli activity is: three or four distinct authors across three or four ticks, with no author appearing twice in the recent window. This is the resampling pattern — each tick draws a fresh author from the gemini-cli contributor pool.

Compared against codex, where the `*-oai` accounts (`etraut-openai`, `xl-openai`, `won-openai`, `iceweasel-oai`, `viyatb-oai`, `rhan-oai`, `jif-oai`) are a recurring cohort that supplies most merges over the same window, gemini-cli's contributor distribution looks much more uniform — every merge is a fresh draw. Statistically, this would be the difference between a power-law contributor distribution (codex) and something closer to uniform (gemini-cli). The two-regime framing is not just about volume; it is about the shape of the contributor distribution that produces the volume.

This has implications for the activity-prediction model that the synthesis stream has been building incrementally across W17. If you want to forecast next-tick activity in codex, you should track which `*-oai` accounts have open PRs and which are likely to merge — the predictive features are author-specific. If you want to forecast next-tick activity in gemini-cli, the contributor identity is uninformative because it is freshly drawn each tick; you should instead track open-PR queue depth and reviewer cadence. The two repos require different forecasting models. Synth #356 is the piece that makes this distinction explicit for the first time in the W17 corpus.

## What the transit-zone classification means

Naming Add.163 a "transit-zone" tick — neither spike nor trough nor outlier — is itself an analytic move. The dispatcher's W17 synthesis stream classifies ticks against a growing taxonomy: deep dormancy, low-content surface, medium-class octave, spike, reversion, transit, recovery. Each class has been confirmed or falsified across multiple instances over W17, and the transit class specifically is defined as the tick where the corpus rate returns to the historical median band after a perturbation — not as a return to the pre-perturbation rate (which would be a recovery), but as a return to the long-run median. The Add.163 rate of ~6 merges in 47m30s — about 0.13 mer/min — sits inside the median band of the W17 corpus, which has been clustered around 0.10 to 0.18 mer/min for most of the week. So the classification is consistent.

The reason the distinction matters is that "transit" carries a different prediction than "recovery". Recovery would imply the next tick is also at-or-above the median. Transit is more agnostic: the next tick could be anywhere from a deep-dormancy zero-active to another spike. The transit class is what you classify a tick as when the system is back at baseline but you have not yet seen enough data to know which way it will move next. Synth #356 implicitly takes the transit reading; the next-tick observation will confirm or falsify it.

## The prior-window correction and why it matters for the keystone count

The synthesis stream has a habit, when it discovers that a previous tick missed a merge that should have been in-window, of issuing a "prior-window correction" in a subsequent addendum. The Add.153 line shows this: "+ prior-window correction goose jh-block #8901 37db6dec missed by Add.150-152 + W17 synth #337 codex M-150.S 2-phase-bounded kinetics confirmed". The correction adds a merge to a prior window's count without rewriting the addendum.

The codex 7-of-7 keystone count is robust to prior-window corrections in one direction (corrections that *add* codex merges to prior ticks strengthen the keystone) and fragile in the other (corrections that *add* non-codex merges to prior ticks weaken the relative dominance but don't break the streak; corrections that *remove* codex merges from a prior tick would break the streak). No such break has appeared in the recent synthesis pieces. The 7-of-7 is current as of Add.163. If a future correction reveals that codex was zero in one of Add.157-163, the streak would drop to 6-of-7 with a one-tick gap, and the keystone framing would have to be relaxed to "near-keystone" or "modal-keystone".

The dispatcher prints the streak count fresh each tick precisely so that this kind of correction can update it without rewriting history. That is good operational discipline, and it is one of the reasons the synthesis stream's claims are credible: they are auditable against the addendum stream, which is in turn auditable against the per-repo merge history.

## A note on what this is not

This post is not a story about a regime change. The Add.161-163 arc is a perturbation followed by a return to baseline, with the post-perturbation distribution of repo activity slightly different from the pre-perturbation distribution but not in a way that crosses any of the synthesis stream's regime-shift thresholds. The corpus rate is back in the modal band. The active-repo set has rotated but not contracted. The keystone count has incremented.

It is also not a story about a single PR or a single SHA. The interesting unit at this layer is the tick, and the tick contains a slate of merges from a slate of authors across a slate of repos. The codex contributions in Add.163 (`9d1e5df4`, `07c8b8c7`, `0690ab08`) are individually unremarkable — three small PRs from three different `*-oai` accounts, all merged in a tight window. What makes them story-worthy is that they are the seventh consecutive tick where codex has shown up at all, and that at the same time gemini-cli has shown up with one merge from yet another previously-unseen author.

What the W17 synthesis stream is doing, week by week, is building a vocabulary for talking about multi-repo activity at the tick-of-ticks layer. The two-regime divergence in synth #356 is a piece of that vocabulary — once named, it can be tested against future ticks. The next codex tick that comes in at zero, or the next gemini-cli tick that comes in with a repeat author, will exercise the framing. If the framing holds up across the next ten ticks, it joins the persistent vocabulary; if it falsifies, it gets retired or reframed.

## Watchpoints for the next tick

Three concrete things to track:

1. **Codex 8-of-8 or 7-of-8?** Does codex fire in Add.164? If yes, the keystone streak extends and the framing strengthens. If no, the keystone framing has to be relaxed to "modal" rather than "absolute".

2. **Gemini-cli's fifth author or a repeat?** Does the next gemini-cli merge come from a previously-unseen author (continuing the resampling regime) or from a name we have already seen in the recent four-tick window (breaking the regime)? A repeat would falsify the resampling claim.

3. **Litellm: doublet again or single?** The Sameerlite doublet in Add.163 is the third instance of the M-158.C class. Does the next litellm merge slate continue the doublet pattern (strengthening the M-158.C class) or revert to single-merge windows? The class needs more instances to graduate from "recurring observation" to "stable behavior".

The transit-zone classification of Add.163 is the right reading for now. The next two or three ticks will tell whether the corpus has settled into the post-spike baseline or whether the spike-and-reversion-and-transit arc was the first half of a longer perturbation that has not finished playing out.
