---
title: "The kurtosis tail-class divides tools: 74.78 vs 1.67 and the fourth moment as a harness fingerprint"
date: 2026-04-27
tags: [pew-insights, kurtosis, statistics, harness-fingerprint, row-distribution, fourth-moment]
---

Most discussions of "how spiky is this CLI's token usage" stop at the second moment. People reach for variance, or its scale-free cousin the coefficient of variation, declare a tool "bursty," and move on. The third moment — skewness — sometimes shows up when somebody notices that the mean is to the right of the median and asks why. The fourth moment almost never gets read out loud. That is a mistake, because the fourth moment is the only place where the *shape of the tail relative to the body* survives. CV tells you the spread relative to the mean. Skewness tells you which side that spread leans toward. Kurtosis tells you whether the spread is built out of "many medium-sized deviations" or "a thin body plus a few catastrophes," and those two regimes have completely different operational implications even when CV and skewness agree.

Per pew-insights `source-row-token-kurtosis` @2026-04-27T02:23:48.806Z, the per-source Fisher excess kurtosis g2 of per-row total_tokens splits the six-source fleet across nearly two orders of magnitude:

- vscode-assistant: g2 = **74.7811**, n=333, mean=5,662.84, stddev=14,933.73
- openclaw: g2 = **23.0571**, n=430, mean=4,342,232.90, stddev=4,955,173.35
- opencode: g2 = **5.3532**, n=324, mean=10,390,321.56, stddev=13,339,300.70
- claude-code: g2 = **4.9976**, n=299, mean=11,512,995.95, stddev=17,605,167.00
- hermes: g2 = **3.8984**, n=168, mean=870,008.29, stddev=988,968.55
- codex: g2 = **1.6733**, n=64, mean=12,650,385.31, stddev=14,252,148.98

For reference points the metric definition embeds in its own help text: a Normal distribution has g2=0, a Laplace distribution has g2=3, an Exponential has g2=6. Anything above 6 is heavier-tailed than exponential. The vscode-assistant figure of 74.78 is not a normal distribution with some outliers. It is not even an exponential with some outliers. It is a regime where roughly 95+% of rows are tightly clustered near a body that is two to three orders of magnitude smaller than the largest single observation, and the fourth-moment integral is being carried almost entirely by a handful of monster rows.

## Why the fourth moment is the right tool here

Excess kurtosis g2 = m4/m2² − 3 raises every standardised deviation to the fourth power before averaging. That weighting is brutal: a deviation of size 5σ contributes 625 units to the numerator, while a deviation of size 1σ contributes 1. Five rows at 5σ outweigh three thousand rows at 1σ. This is the property you want when the operational question is "how often does this tool do something so token-heavy that the average underestimates the worst case by an order of magnitude," because that question is fundamentally about tail mass — and the tail is exactly what the fourth power amplifies.

CV (a second-moment statistic, dimensionless because variance is in units²) treats every deviation linearly in σ-space and so a workload that produces ten 3σ rows looks the same as one that produces one 9.5σ row plus a tighter body. They are very different workloads. The first one is a noisy but predictable producer; the second one is a thin-body, rare-shock producer where the median tells you nothing about the budget.

Skewness g1 (third moment) tells you the *direction* of the asymmetry but is only weakly sensitive to how extreme that asymmetry gets — once you are clearly right-skewed, doubling the magnitude of the largest observation barely moves g1. The fourth moment absorbs that magnitude doubling much more aggressively. So when the question is *how rare and how large* the heavy tail is, kurtosis is the correct lens.

A practical rule of thumb that emerges from this: if you are sizing a token budget or a rate limit headroom against a tool whose g2 is in the tens, you do not size against the mean and you do not size against p99. You size against the worst observed row and add a multiple, because the fourth-moment integral is telling you that worst-row dominance is not noise — it is the load model.

## What the ladder is actually showing

Read top-to-bottom, the kurtosis ladder is a fingerprint of *which kind of harness is talking to the model*. The 74.78 → 1.67 spread is not noise; the cross-source ratio of nearly 45× is too large to be measurement variance over n in the hundreds. The two ends of the ladder represent two different population structures of rows.

**Top of the ladder (g2 ≫ 10): IDE-style assistants and polling daemons.** vscode-assistant at 74.78 and openclaw at 23.06 share a structural property: most invocations are tiny. An IDE-side completion or a polling daemon overwhelmingly issues short prompts that fit in a few hundred or a few thousand tokens, because the harness deliberately keeps prompts cheap to keep latency low. But every so often — when the user opens a giant file, when a session is resumed with a long history, when a polling pass picks up a fat queue — a single row blows out by a factor of 50× to 200× over the modal row. The fourth moment then explodes because (50)⁴ = 6.25 million and there is no comparable mass below to dilute it. vscode-assistant's mean of 5,663 against a stddev of 14,934 already tells you the body is small in absolute terms and that even the stddev is dominated by tail; g2 = 74.78 confirms that the tail is not just heavier than Normal, it is heavier than exponential by an order of magnitude.

**Middle of the ladder (g2 ≈ 4–6): full-context coding harnesses.** opencode (5.35), claude-code (5.00), and hermes (3.90) are all close to the Laplace-to-Exponential band. These are tools that send fat prompts every turn — full file context, tool definitions, conversation history, system prompt — so the *modal* row is already large, and the heaviest rows are only a few times larger than the modal, not fifty times. The body of the distribution carries enough mass that the fourth-moment integral cannot be dominated by a handful of rows. You see this in the means: claude-code's mean of 11.5M and opencode's mean of 10.4M are within striking distance of their stddevs (17.6M and 13.3M respectively), which is the signature of a distribution where body and tail are on the same scale.

**Bottom of the ladder (g2 ≈ 1.7): codex.** codex at 1.67 with n=64 is the lightest-tailed distribution in the fleet — flatter than Laplace, between Normal and Laplace. With n=64 the kurtosis estimate is necessarily noisier than the larger samples, but the magnitude is far enough from the leptokurtic neighbours that the ranking is meaningful. What it says operationally: codex rows arrive in a relatively narrow band relative to their mean. The mean is 12.65M and the stddev is 14.25M, so CV is around 1.13 — high in absolute terms but the *shape* of the deviations is closer to symmetric than to heavy-tailed. That is consistent with a harness that is configured to send a stable prompt envelope every turn, with reasoning expansion roughly in proportion. Whether that configuration is intentional or just a small-sample artifact is a question for a longer window, but the fourth moment is the fastest way to surface "this tool's row distribution looks like noise around a level" versus "this tool's row distribution looks like a body plus rare disasters."

## The independence of kurtosis from CV (and why it matters)

Kurtosis is mathematically *independent* of CV. You can hold the mean and stddev fixed and change g2 by orders of magnitude by redistributing mass between the body and the tail. This is why the kurtosis ladder cuts orthogonally to anything CV can tell you.

Consider opencode (g2=5.35, CV ≈ 1.28) and claude-code (g2=5.00, CV ≈ 1.53). claude-code has higher CV — its rows are more spread relative to the mean — but its kurtosis is lower than opencode's. The interpretation is that claude-code's higher CV comes from a *thicker body* (more medium-large rows) rather than a heavier tail. opencode's lower CV plus higher kurtosis says the body is tighter but the tail, when it fires, is sharper. Two tools that look "similar" on a CV-ranked dashboard reveal opposite tail geometries on a kurtosis-ranked dashboard.

The same orthogonality principle separates vscode-assistant from openclaw. Both have g2 ≫ 10. But vscode-assistant's body is at scale 10³–10⁴ tokens while openclaw's body is at scale 10⁶ tokens — three orders of magnitude apart. CV would put both in "high spike" territory; kurtosis additionally tells you that vscode-assistant's spikes happen against a much thinner background, so each spike is structurally more disruptive to a budget calibrated on body-mean. Operationally: if you set a per-row budget alert for vscode-assistant at 10× the mean, you would catch the spikes; the same multiple on openclaw would catch nothing because that source's tail rows live at maybe 2–3× the mean.

## What to do with this

1. **Replace per-tool flat token caps with kurtosis-aware caps.** A tool whose g2 is in the seventies cannot be safely capped at a small multiple of its mean; the multiple has to be sized against the worst observed row plus a margin, because the body is irrelevant to where the tail lives. A tool whose g2 is below 2 can be capped at a small multiple of the mean with much higher confidence that the cap will not fire on routine operation.

2. **Rank dashboards by g2 in addition to CV.** The two metrics are independent, and shipping only one of them means losing half the information about the row distribution. CV-only dashboards systematically under-rank thin-body, fat-tail tools because their CV is moderate while their g2 is enormous.

3. **Treat g2 ≫ 6 as a flag for "rare-shock" workload class.** Anything heavier than Exponential is a workload where the worst hour in a month dominates the budget for the month. That is operationally different from "noisy steady" workloads (g2 around 0–3), and the on-call playbook should differ accordingly: thin-body rare-shock workloads need *outlier alerting on individual rows*; noisy steady workloads need *windowed mean alerting*. The same alert configured on both types is going to either miss the rare-shock tool's actual problem or fire constantly on the steady tool's noise.

4. **Use the ladder as a regression test on harness design changes.** If a harness rolls out a "send less context per turn" optimisation, you would expect g2 to *rise* (body shrinks, tail unchanged → ratio amplifies). If it rolls out "always send full context," you would expect g2 to *fall* (body inflates to meet the tail). A flat g2 across a harness redesign that claimed to change context behaviour is a signal that the change did not actually take effect, or that it took effect in the wrong direction. CV alone cannot do this regression test because CV moves under both kinds of change.

The fourth moment is unfashionable because it is sensitive to outliers, and a generation of intro-stats teaching has trained engineers to treat outlier-sensitivity as a defect. For describing a token-row distribution where the operational cost of the worst row is exactly what the budget cares about, that "defect" is the entire point. Read your kurtosis. The ladder it produces — 74.78 down to 1.67 across a six-source fleet observed at 2026-04-27T02:23:48.806Z — is a one-number-per-tool fingerprint that no second-moment statistic can reproduce, and the ranking it produces aligns more cleanly with operational risk than any spread metric on its own.
