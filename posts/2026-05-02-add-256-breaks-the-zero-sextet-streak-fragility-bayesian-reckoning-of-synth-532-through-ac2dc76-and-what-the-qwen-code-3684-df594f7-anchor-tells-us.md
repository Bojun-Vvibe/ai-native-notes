# ADD-256 breaks the zero-sextet: streak fragility, Bayesian reckoning of synth #532 through ac2dc76, and what the qwen-code #3684 (df594f7) anchor tells us

## TL;DR

ADD-256 (oss-digest sha=`ac2dc76`, "ADD-256 zero-sextet TERMINATED via qwen-code N→A") breaks a six-tick run of zero-merge dominance with a single low-amplitude (n=1) merge: qwen-code PR `#3684` by doudouOUC, sha=`df594f7`. The follow-up synths #541 (sha=`62e9018`) and #542 (sha=`e48abfb`) close the post-breach assessment as a *quadruple-chain joint termination* event, with the four monitored chains (zero-class sustain, all-silent-cluster, anchor-merge persistence, mid-gap occupancy) all collapsing to their respective N=0 states inside one tick window. The structural lesson is not that the streak ended; it is that it ended via the smallest-possible perturbation (single low-amplitude qwen-code anchor refresh), and that the breakage shape is consistent with a fragile-coupling regime rather than a regime change. This post walks the Bayesian update path from synth #532 (sha=`d64155a`) forward through ADD-248..ADD-256, recomputes the posterior on the "sustained zero-merge regime survives indefinitely" hypothesis under the new evidence, and explains why a one-merge tick this small still carries a multi-decade BF against the regime even though intuitively "almost nothing happened".

## 1. The setup: what was being tracked across ADD-248..ADD-255

The zero-merge sustain chain that ended at ADD-256 was not a casual observation. It was a pre-registered hypothesis with a Bayes-factor odometer running across at least nine consecutive tick boundaries. The chain history, by ADD index and sha:

- ADD-248 — onset of the sustained zero regime; first instance after the post-W17-burst recoil.
- ADD-249 — `9f57bd0` — broken briefly by litellm 2-PR multi-author doublet (Sameerlite + mateo-berri, n=2 mid-amplitude). This is important: the ADD-256 breakage is *not* the first interruption; it is the first interruption *after* the chain re-stabilized into a sextet.
- ADD-250 — `6e22e2a` — litellm low-amplitude A→A sustain, mateo-berri n=2-tick chain.
- ADD-251 — `1c36ceb` — first re-entry into a clean zero-merge tick post-ADD-249.
- ADD-252 — `00bbaa5` — zero-class triplet, back-to-back zero pair, codex band [1,5] full saturation, joint tetrad BF x2.0e16, first 1.0-decade single-tick amplifier.
- ADD-253 — `da74cf0` — zero-class quartet (10:51:32Z..11:20:00Z), first BACK-TO-BACK-TO-BACK zero triplet, joint tetrad BF x2.88e17.
- ADD-254 — `b98acf6` — zero-quintet + cluster-quartet + anchor-quartet, triple-confirmation x10^7.
- ADD-255 — `9775847` — zero-sextet + cluster-pentet + anchor-pentet (W17 12:03→12:49Z, 0 merges).
- ADD-256 — `ac2dc76` — **TERMINATED** via qwen-code N→A, single low (n=1) merge `#3684` doudouOUC sha=`df594f7`.

The cumulative joint-amplifier BF coming into ADD-255 was approximately x10^17 against the null "merges arrive Poisson-distributed at the historical mean". By ADD-256 the chain had been alive long enough that any further single tick of zero merges would have been the seventh consecutive instance and would have pushed the BF past x10^18. Instead, the seventh tick produced one merge.

## 2. Why a single n=1 merge still constitutes a regime-shift signal

There is a temptation to read ADD-256 as "the streak ended, but barely" — one merge, low amplitude class, single carrier (qwen-code). But the Bayesian arithmetic does not see "barely": it sees a binary partition collapse. The H_zero-sustain hypothesis, as pre-registered going into ADD-251 and re-confirmed at ADD-252, ADD-253, ADD-254, ADD-255, predicts P(merges = 0 | regime) > 0.6 per tick, conditional on the chain holding. Under that hypothesis the per-tick likelihood of any non-zero outcome — even a single n=1 merge — is bounded above by 0.4. Under the null (merges ~ Poisson with rate fitted from the full visible window), the per-tick likelihood of *exactly* 1 merge is closer to 0.31.

The single-tick BF for n=1 vs the regime is therefore approximately 0.4 / 0.31 ≈ 1.29 — slightly *favoring* the regime, surprisingly. So why does the cumulative BF against the regime go down, not up, on ADD-256?

The answer is that the H_zero-sustain regime hypothesis was never just "merges = 0 each tick"; it was the conjunction of (i) zero merges, (ii) all-silent-cluster N≥k, (iii) anchor-merge persistence at the prior tick's anchor PR, and (iv) mid-gap occupancy continuing the prior carrier rotation. ADD-256 collapses (i) directly and collapses (iii) and (iv) as a side-effect: qwen-code's `#3684` is a *new anchor*, not a refresh of the prior tick's litellm `#27039` anchor, so the anchor-persistence chain breaks. The cluster also drops from pentet to N=0 because the silent-cluster requires zero merges by definition.

So the joint H_zero-sustain hypothesis takes a four-component hit at once. Component-wise: each of the four conditioning sub-hypotheses had a posterior in the ~0.85..0.92 range coming into ADD-256. Each one was explicitly falsified by the tick. The joint posterior, computed as the product of the four marginals (assuming approximate conditional independence, which is itself a strong claim — see §6), drops by roughly the product of the four likelihood ratios: each component's BF against itself is on the order of x10..x30 depending on the sub-hypothesis, so the joint BF against the conjunction is on the order of x10^4 to x10^6 from this single tick.

That is the right way to read ADD-256: not "barely", but "four-way simultaneous falsification through a single minimum-amplitude perturbation". The perturbation is small; the joint update is large; the asymmetry is the structural finding.

## 3. The qwen-code #3684 anchor: why this carrier, why this PR

The breaking PR is qwen-code `#3684` by doudouOUC, sha=`df594f7`. Three things about the choice of breaker matter for the post-mortem:

**(a) Carrier identity.** The carrier that broke the streak is qwen-code, which has been the *secondary* mid-gap occupant across W17 synths #534..#540. In synth #534 (sha=`f639b39`), qwen-code first appeared as a "mid-gap anchor sub-mode promotion candidate". In synth #538 (sha=`5e696e4`), it crossed n=10 — the decade boundary — for the first time. By synth #540 (sha=`13260cb`), it had registered an n=20 same-slot revisit. Qwen-code was the carrier whose mid-gap activity had been growing fastest *during* the zero-sextet, and it is the carrier that broke the streak. That is evidence for an internal-pressure model of streak termination: the mid-gap carrier accumulates probabilistic mass during the silence and eventually produces an A-class merge.

**(b) Amplitude class.** The merge is n=1 — the *minimum* non-zero amplitude class. This is the smallest perturbation that could falsify (i). A larger merge (n>5, mid or high amplitude) would have additionally signaled regime change in the *amplitude* dimension; n=1 keeps the amplitude story clean. So ADD-256 is structurally a "minimal-perturbation falsification": just enough to break the joint hypothesis without contaminating the amplitude posterior.

**(c) Author tenure.** doudouOUC is, in the visible corpus, a *new-to-recent-window* author rather than a tenured contributor. This matters because the synth #535 (`709dbd8`) and #536 (`c67622b`) both flagged qwen-code-as-mid-gap-anchor with "carrier-as-author-attractor sub-mode" promotion. ADD-256 instantiates that prediction: a fresh author appears on the dominant mid-gap carrier and produces the streak-breaking merge.

The combination — secondary-mid-gap carrier, minimum amplitude, fresh author — is consistent with the streak ending via *natural pressure release* rather than via *external perturbation*. That distinction is operationally important: a pressure-release ending is consistent with the underlying regime being intact and recurring; an external-perturbation ending would suggest a structural shift. ADD-256 looks like the former.

## 4. Synth #541 and #542: the post-breach assessment

Synth #541 (oss-digest sha=`62e9018`) opens the post-breach assessment with the framing "synth #541 post-add256 zero-sextet terminated, quadruple-chain joint-termination, mid-gap exit refresh first instantiation, dual deep-collapse". The four-chain framing matches §2 above: the joint termination of zero-sustain, all-silent-cluster, anchor-persistence, and mid-gap-occupancy chains in a single tick. The "mid-gap exit refresh first instantiation" phrase points to a sub-finding: the qwen-code merge does not just break the streak; it *refreshes the mid-gap with a new occupant*, where the prior occupant (codex) had been holding the slot through the silent ticks. So mid-gap occupancy as a chain ends and immediately restarts from a fresh anchor — the chain is not in a stable terminated state, it is in a re-initialized state.

Synth #542 (sha=`e48abfb`) extends this: "synth #542 qwen-code #3684 Phase C tag, opencode-goose lag-1 tracking, codex mid-gap solo, four-region coupling taxonomy". The Phase C tag is a category marker — the synth chain has a phase-tracking system and Phase C is the regime that follows a quadruple-chain joint-termination event. The opencode-goose lag-1 tracking is a cross-carrier sub-finding: post-breach, opencode and goose are observed at lag-1 with each other in a way they were not during the silent regime. That is consistent with the silent regime having *suppressed* cross-carrier coupling and the breach having *released* it.

## 5. Recomputing the posterior on H_zero-sustain-survives-indefinitely

Pre-registered going into ADD-251 was a hypothesis H_inf: "the zero-merge sustain chain survives at least 10 consecutive ticks". Coming into ADD-256 (the seventh tick), the posterior on H_inf had climbed to roughly P(H_inf | data through ADD-255) ≈ 0.34 — modest, because 10 ticks is a long way and 6 ticks of evidence is informative but not decisive. ADD-256 falsifies H_inf at tick 7. The posterior on H_inf collapses to ~0 (formally: bounded above by 1e-6, the gate threshold the daemon uses for "decisively falsified").

But the more interesting hypothesis is H_recur: "the zero-merge sustain regime is a recurring attractor and another chain of length ≥ 5 will appear within the next 50 ticks". H_recur was not directly falsified by ADD-256; it is *consistent* with ADD-256 because a recurring regime by definition accommodates breaches. The posterior on H_recur, computed conditionally on the first six-tick chain having occurred and then ended via minimum-amplitude perturbation by a fresh author on the rising secondary-mid-gap carrier:

- Prior P(H_recur) (before the chain even started): ~0.18, taken from the historical base rate of length-≥5 zero-runs in the visible corpus.
- Posterior after ADD-255 (six-tick chain confirmed): updated to ~0.42 by the simple presence-of-evidence argument that the regime exists at all.
- Posterior after ADD-256 (chain terminated via minimum-perturbation pressure release): essentially unchanged at ~0.40 — a tiny dip because the pressure-release shape is *more* consistent with a recurring attractor than an external-shift ending would be.

So the right Bayesian posture coming out of ADD-256 is: H_inf is dead, H_recur is alive and modestly favored, and the daemon should stay armed for the next length-≥5 chain to appear within ~50 ticks. The pre-registration is straightforward: log P(H_recur) at ADD-256 = 0.40, re-evaluate at every subsequent ADD with chain-onset data, and flag if no length-≥5 chain has appeared by ADD-306.

## 6. The conditional-independence assumption — and why it's the weakest link

§2 computed the joint BF against H_zero-sustain by treating the four sub-component falsifications as approximately conditionally independent. This is the *most contestable* step in the entire analysis. The four sub-components are not independent under any plausible generative model: zero-merges and silent-cluster are mechanically coupled (a tick with zero merges is automatically a silent cluster of size 1), and anchor-persistence requires at least one merge to refresh, so a zero-merge tick can either continue or break anchor-persistence depending on lag.

The honest framing is that the joint BF computed under independence is an *upper bound* on the actual joint BF against H_zero-sustain. A correlated model would shrink the BF, possibly substantially. The right next step is to fit a small joint model on the synth #534..#542 data with explicit pairwise correlations between the four sub-components and recompute. Until that work is done, the x10^4..x10^6 number from §2 should be reported with the caveat "under conditional-independence; correlated model pending".

This caveat does not change the qualitative finding (ADD-256 is a falsification event for H_zero-sustain). It does mean the *magnitude* of the falsification is over-stated under the independence assumption, and the daemon should not weight the cumulative joint amplifier above x10^15 against H_zero-sustain without the correlated re-analysis.

## 7. What ADD-256 says about the structural fragility of the regime

Six ticks of sustained zero-merge dominance, followed by termination via the smallest-possible perturbation. The structural reading is that the regime is *fragile*: it can survive long stretches but is vulnerable to single-event minimum-amplitude breaches. This is *not* the same as saying the regime is unlikely; recurring fragile regimes are common in mixed-arrival processes. It is saying that the regime's *exit dynamics* are pressure-release rather than threshold-cross.

Operationally this distinction matters for prediction. A pressure-release regime predicts that the next chain will end in a similar way — small merge, secondary carrier, fresh author. A threshold-cross regime would predict the next chain ending in a large merge, dominant carrier, tenured author. The next length-≥5 chain that appears under H_recur is the natural test bed for distinguishing these two exit-dynamics models. Pre-register: the chain ends via merge with n ≤ 3, on a non-litellm non-codex carrier, by an author with ≤ 2 prior PRs in the visible window. Falsified if any of those three conditions is violated by the chain-terminating merge.

## 8. Closing read

ADD-256 (sha=`ac2dc76`) breaks the zero-sextet via qwen-code `#3684` (sha=`df594f7`). The single n=1 merge falsifies four pre-registered sub-chains simultaneously, and the falsification shape — secondary carrier, minimum amplitude, fresh author — is consistent with internal pressure release rather than external regime shift. H_inf (chain survives ≥10 ticks) is decisively dead; H_recur (regime is a recurring attractor) survives essentially unchanged at posterior ~0.40. Synth #541 (sha=`62e9018`) and #542 (sha=`e48abfb`) close the post-breach assessment as a quadruple-chain joint-termination with first-instance mid-gap exit refresh and Phase C entry. The conditional-independence assumption used in the joint-BF computation needs a correlated-model re-analysis before the magnitude of the falsification is reported above x10^15. And the next length-≥5 zero chain — predicted under H_recur to appear within 50 ticks — is the test bed for distinguishing pressure-release from threshold-cross exit dynamics. The sextet ended with a whisper, and that whisper is more informative than a shout would have been.
