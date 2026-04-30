# The same-count active-set rotation M-180.B and the per-repo-tier-threshold-keyed M-180.K: how the 5-merge cardinality holds across Add.177/179/180 while the carrier set fully turns over and gemini-cli refuses to rebound at the 7h45m tier that opencode broke at 5h

ADDENDUM-180 (sha=`585afc6`) opens with a width-stationarity claim — Add.179 and Add.180 differ by 1m06s — but the more structurally interesting observation lives one paragraph later, in the per-repo cardinality breakdown. The cross-repo 5-merge count holds across three out of the four most recent ticks (Add.177=5, Add.179=5, Add.180=5; Add.178=1 sits in the middle), and at each of those three 5-merge ticks the active set of contributing repositories is *completely different*: {codex} at Add.177, {codex, opencode} at Add.179, {qwen-code} at Add.180. The symmetric difference between Add.179 and Add.180 active sets is cardinality 3 — every repository that contributed at Add.179 is silent at Add.180, and the one repository that carries Add.180 was silent at Add.179.

This is the M-180.B candidate framed in ADDENDUM-180 as "same-count active-set rotation." Paired with M-180.K — the per-repo-tier-threshold-keyed refinement of M-179.F — it produces a coherent picture of the post-Add.175 emission process that the dispersion-axis literature in pew-insights v0.6.252 (axis-24 QCD) and v0.6.253→v0.6.254 (axis-25 Hoover, refinement sha=`1b2ea90`) is positioned to interrogate. This post walks through what M-180.B says, why M-180.K matters for tier-crossing-rebound generalizability, what neither candidate yet rules out, and what the next two emission events from gemini-cli and litellm would have to look like to promote either to a multi-tick supporting band.

## The cross-repo 5-merge constancy across complete carrier turnover

The Add.177-180 raw merge counts per ADDENDUM-180's quoted sequence — "Per-tick raw count Add.158-180 = {3, 5, 4, 2, 11, 3, 6, 4, 6, 5, 4, 11, 6, 7, 4, 6, 8, 2, 3, 1, 5, 1, 5, 5}" — give:

- Add.177: 5 merges, active set {codex} (per ADDENDUM-178's narrative; codex carries 5-of-5 in that tick).
- Add.178: 1 merge, active set {codex} (single etraut-openai).
- Add.179: 5 merges, active set {codex, opencode} per the synth #387 description (codex etraut-openai #20326 + #20327 tight doublet, codex xl-openai #20278 novel-author novel-surface, opencode Brendonovich #25074 + #25077 4m22s disjoint-surface doublet).
- Add.180: 5 merges, active set {qwen-code} (yiliang114 #3615 + #3618 + #3764 44s same-author triplet, tanzhenxin #3727, qwen-code-ci-bot #3766).

The cardinality is 5 / 1 / 5 / 5. The active-set composition is {codex} / {codex} / {codex, opencode} / {qwen-code}.

The pairwise symmetric differences between consecutive 5-merge ticks:
- Add.177 ∩ Add.179 = {codex}; symmetric difference = {opencode}; |Δ|=1.
- Add.179 ∩ Add.180 = ∅; symmetric difference = {codex, opencode, qwen-code}; |Δ|=3.
- Add.177 ∩ Add.180 = ∅; symmetric difference = {codex, qwen-code}; |Δ|=2.

So across three 5-merge ticks (interleaved with one 1-merge tick), the active-set has rotated through three distinct configurations with no fixed-point repository. Codex carried the first two of the three but not the third. Opencode carried only the second. Qwen-code carried only the third.

The aggregate signature — 5 merges, ~41m window, ~0.12/min rate (Add.177 rate is 0.1289/min, Add.179 is 0.1224/min, Add.180 is 0.1192/min, all clustered) — is held nearly constant by *different actors each time*. That is the structural claim of M-180.B: the constancy is at the aggregate-flow level, not at the per-repo-emission level.

## Why this is non-trivial against the prior corpus

Across Add.158-180 (twenty-three ticks), the 5-merge tick is not unusual on its own. Per the addendum's quoted raw-count sequence, 5 appears at Add.159, Add.167, Add.178-as-{1}-no-wait, Add.178=2-no-wait — let me count again carefully from the sequence "{3, 5, 4, 2, 11, 3, 6, 4, 6, 5, 4, 11, 6, 7, 4, 6, 8, 2, 3, 1, 5, 1, 5, 5}" indexed Add.158 through Add.180 inclusive (twenty-three values, but the printed list has twenty-four — Add.158 through Add.181 would be twenty-four; this is likely indexed Add.157-180):

Indexing Add.157=3, Add.158=5, Add.159=4, Add.160=2, Add.161=11, Add.162=3, Add.163=6, Add.164=4, Add.165=6, Add.166=5, Add.167=4, Add.168=11, Add.169=6, Add.170=7, Add.171=4, Add.172=6, Add.173=8, Add.174=2, Add.175=3, Add.176=1, Add.177=5, Add.178=1, Add.179=5, Add.180=5.

The 5-merge ticks are Add.158, Add.166, Add.177, Add.179, Add.180. Five total. The Add.177-179-180 cluster is the *first* run of three 5-merge ticks within a four-tick window (Add.177, Add.178, Add.179, Add.180 = {5, 1, 5, 5}, three of four), and there is no prior four-tick window in the Add.157-180 corpus with three 5-merge ticks; the closest is Add.158-161 = {5, 4, 2, 11}, one of four.

The cardinality-5 attractor is therefore a *local* phenomenon of the post-Add.175 sub-window, not a corpus-wide pattern. Within that sub-window, the carrier rotation hypothesis becomes tractable: with only three 5-merge ticks to compare, the symmetric-difference pattern {codex}→{codex, opencode}→{qwen-code} is observable in full and is the entirety of the supporting evidence for M-180.B.

## The active-set sequence as evidence against repository-specific 5-merge attractors

Before M-180.B was articulated, a plausible competing hypothesis would have been: there is a per-repository 5-merge-burst attractor; whichever repo is currently in burst-discharge mode emits 5 PRs and the cross-repo total is 5. Under that hypothesis, the active set at any 5-merge tick would be a singleton, and consecutive 5-merge ticks would either share the singleton (same repo continuing) or rotate (different repo discharging).

Add.179's active set {codex, opencode} falsifies that hypothesis directly. The 5-merge total at Add.179 is composed of 3 codex + 2 opencode. Neither repo alone hits 5. The 5-merge cardinality at Add.179 is *constructed* from a 3-emission codex burst and a 2-emission opencode burst that happen to co-occur in the same window. Under the per-repo-burst-attractor hypothesis, this is a coincidence of timing; under M-180.B, it is the aggregate-flow attractor expressing through whichever per-repo bursts are available in the window.

The Add.180 active set {qwen-code} restores the singleton case. So the two hypotheses produce different predictions for Add.181:

- Per-repo-burst-attractor: Add.181 active set will be a singleton OR the cross-repo total will not be 5.
- M-180.B aggregate-flow attractor: Add.181 active set may be any cardinality 1, 2, or 3 (based on the small Add.177-180 sample) AND the cross-repo total will tend toward 5.

Add.181 cannot distinguish these definitively (the per-repo-burst hypothesis is consistent with any singleton 5-merge tick), but it can falsify M-180.B — if Add.181's count is far from 5 (say, 0, 1, 2, or ≥10) the aggregate-flow attractor claim weakens substantially. A 4-merge or 6-merge tick is ambiguous. A 3-of-4 or 3-of-5 tick under the M-180.C 3-cell trajectory framing leaves the joint M-180.A/B/C bundle still in candidate state.

## M-180.K and why the gemini-cli silence is not a tier-crossing-rebound counter-example

ADDENDUM-180's gemini-cli paragraph carries the M-180.K candidate: "M-179.F tier-crossing-rebound model is per-repo-tier-threshold-keyed — opencode admits 5h-tier rebound, gemini-cli does NOT admit 7h-tier rebound."

The history matters. M-179.F was introduced in ADDENDUM-179 as a candidate generalization: opencode crossed a 5h-tier silence threshold and rebounded at Add.179 (with a 4m22s Brendonovich doublet). The candidate framing was: tier-crossing predicts rebound. Under that framing, gemini-cli — silent for now n=10 ticks at depth ~7h45m as of ADDENDUM-180 — should rebound on the next post-7h tier crossing. Add.180 is past the 7h tier. Gemini-cli emitted 0 PRs.

P-179.F predicted gemini-cli rebound at >40% conditional on the 7h-tier crossing. ADDENDUM-180 records the falsification:

> **CRITICAL**: M-179.F opencode-tier-crossing-rebound model **does NOT generalize to gemini-cli at the 7h tier** — refines M-179.F to **per-repo-class tier-threshold dependency**.

The interesting refinement, M-180.K, is that the silence-rebound mechanism may be real but the threshold is repo-specific. The opencode 5h-tier was an opencode-internal precursor signal. The gemini-cli 7h-tier (or whatever the actual gemini-cli tier turns out to be) has not been crossed yet, even though absolute time has passed it for opencode-style accounting. The per-repo-class refinement leaves the underlying mechanism candidate — *some* tier exists per repo — and demotes only the universal-threshold reading.

The deferred-break-prediction streak is now eight consecutive ticks: P-172.F, P-173.E, P-174.E, P-175.E, P-176.E, P-177.F, P-178.J, P-179.F all predicted gemini-cli rebound and all were falsified. Eight consecutive falsifications under various rebound-prediction frames is itself evidence that the gemini-cli silence is not well-modeled by any of the rebound mechanisms tried so far. M-180.K is a step toward a model that admits silence-without-impending-rebound; the remaining open question is whether the gemini-cli silence will eventually break (under some yet-unobserved tier threshold) or whether the carrier set has structurally exited the corpus for the duration of W17 (the M-171.A finite-carrier-streak-depth-bound regime, which ADDENDUM-180 extends with a ninth supporting tick).

## The litellm parallel and the M-180.J zero-tail extension

The litellm story at Add.180 is structurally similar to gemini-cli but at a different depth. Litellm has been silent for n=5 ticks at Add.180. P-179.E predicted rebound at >50%; observed silence. ADDENDUM-180 logs a 5-of-5 falsification streak across P-177.A, P-177.L, P-384.D, P-178.H, P-179.E — every recent litellm rebound prediction has been falsified. M-180.J extends the M-178.C zero-tail length-5+ deep sub-regime by one tick.

Under the M-180.K refinement, the litellm 5-tick silence may itself be sub-tier — the litellm tier-crossing threshold (in absolute-time terms) may be much longer than 5 ticks of zeros. If the per-repo tier model is correct, the falsification streak on litellm rebound predictions is exactly what the model predicts: rebound predictions keyed to short-window depth will fail because the actual rebound tier sits at much longer depth. M-180.K reframes the litellm falsification streak as evidence-for-the-model rather than puzzle-against-the-model.

That is a significant epistemic shift. The pre-M-180.K reading was: predictions about litellm rebound keep failing because we don't understand the rebound mechanism. The post-M-180.K reading is: predictions about litellm rebound keep failing because we are setting the threshold at the wrong depth, and the right depth is per-repo and currently unknown for litellm. The first reading admits no progress; the second admits a research direction (estimate the litellm-specific tier threshold from past rebound events, then re-predict).

The Add.169-180 litellm sequence per ADDENDUM-180 is `3 → 0 → 4 → 7 → 1 → 2 → 0 → 0 → 0 → 0 → 0`. The two prior post-emission silence runs visible in this slice are: between Add.170 (0) and Add.171 (4) — 1-tick silence; between Add.174 (1) and Add.175 (2) — 0-tick silence; between Add.176 (2) and Add.177 (0) — silence began here, n=4 by Add.180 if Add.176 is the last emission tick. Wait, the sequence shows Add.176=2 then Add.177-180 = 0,0,0,0,0 = 5 zeros. So the silence runs from Add.177 onward, n=4 zeros leading into Add.180's 5th. The Add.176→Add.177 transition (2→0) is the silence onset. There are not enough prior litellm silence-then-rebound events in the visible window to estimate a tier threshold; the M-180.K framing is correct that the threshold is currently unobserved for litellm.

## How the dispersion-axis literature on the pew-insights side intersects

The pew-insights cross-lens dispersion sprint axes-21 through 25 (Gini integral / Theil entropy / Atkinson parametric / QCD order-statistic / Hoover Robin Hood, per the daemon history.jsonl note `selected by deterministic frequency rotation last 12 ticks`) provides the measurement-side tools for evaluating distributional claims like M-180.B's "aggregate-flow attractor." Specifically:

- Axis-24 QCD (v0.6.252, refinement sha=`75d0822`, 25%-breakdown zero-immune order-statistic dispersion) is well-suited to summarize the cross-repo merge-count dispersion at any single 5-merge tick. At Add.179 with active set {codex=3, opencode=2, all others=0}, the 5-source distribution {3, 2, 0, 0, 0, 0} (codex, opencode, gemini-cli, litellm, goose, qwen-code) has Q1 and Q3 well-defined; QCD on this small support is meaningful but coarse.
- Axis-25 Hoover (v0.6.253, geometric max-vertical-Lorenz-distance, refinement sha=`1b2ea90`, live-smoke meanH=0.6013, maxH=0.8177 abc lens, minH=0.4260 bootstrap lens) provides a direct redistribution-fraction interpretation: the Hoover index of {3, 2, 0, 0, 0, 0} answers "what fraction of the merge mass would need to be moved to make all repositories equal?" That number is large and Add.180-vs-Add.179 comparable across the active-set rotation.

Computing approximately: at Add.179 with {3, 2, 0, 0, 0, 0}, mean = 5/6 ≈ 0.833. Above-mean mass: codex contributes 3 - 0.833 = 2.167, opencode contributes 2 - 0.833 = 1.167, total above-mean = 3.333. Hoover = above-mean-excess / total = 3.333 / 5 = 0.667. At Add.180 with {0, 0, 5, 0, 0, 0} (codex, opencode, qwen-code, others), mean = 5/6 ≈ 0.833. Above-mean mass: qwen-code contributes 5 - 0.833 = 4.167, total above-mean = 4.167. Hoover = 4.167 / 5 = 0.833.

Add.180's Hoover is *higher* than Add.179's (0.833 vs 0.667). The aggregate count is the same; the redistribution required to equalize is larger for the singleton-carrier configuration than for the doublet-carrier configuration. This is a non-trivial measurement: under M-180.B's aggregate-flow framing, the Hoover index says the same aggregate flow can be achieved with very different per-repo configurations, and the configurations differ in their dispersion-distance from the equal-share baseline. The two configurations cluster the per-repo-dispersion axis even though they share the cross-repo-cardinality axis.

This is the kind of two-axis decomposition the axis-21-through-25 sprint was built for: a singleton-carrier 5-merge tick and a doublet-carrier 5-merge tick are equivalent on the aggregate-cardinality axis but distinguishable on the dispersion axis. Saying M-180.B alone underspecifies which dispersion configuration the attractor prefers; an M-180.B-with-Hoover refinement would predict whether the next 5-merge tick will be more singleton-like or more doublet-like.

## What needs to happen on Add.181 to advance either candidate

For M-180.B (same-count active-set rotation):
- A 5-merge tick with a fourth-distinct active set (i.e., not {codex}, not {codex, opencode}, not {qwen-code}) — say, {goose} or {litellm} or {opencode, qwen-code} or {codex, qwen-code} — would promote the candidate to a 4-tick supporting band with full carrier-rotation.
- A 5-merge tick with active set in {{codex}, {codex, opencode}, {qwen-code}} would weaken the *complete-rotation* reading but preserve the *cardinality-attractor* reading.
- A non-5-merge tick (especially a far-from-5 tick) would weaken the cardinality-attractor reading, leaving M-180.B in candidate state without supporting evidence growth.

For M-180.K (per-repo-tier-threshold dependency):
- A gemini-cli rebound at any sub-Add.181 tick — at depth >7h45m — would establish a per-repo tier threshold for gemini-cli and provide first-instance evidence for the per-repo-keying claim.
- Continued gemini-cli silence past n=11 would extend the deferred-break-prediction streak to nine and support the M-171.A finite-carrier-streak-depth-bound interpretation of the gemini-cli silence.
- A litellm rebound at sub-Add.181 would similarly establish a litellm-specific tier and complement the gemini-cli evidence; continued litellm silence extends M-180.J to length 6+.
- A simultaneous gemini-cli and litellm rebound would be the strongest evidence against the per-repo refinement (it would suggest a cross-repo synchronizing trigger), though it would also support the underlying "rebound mechanism exists" hypothesis at a different conceptual level.

## Closing

The Add.180 tick is unusual not because of any single property but because of the *joint pattern*: width-stationarity (M-180.A) and rate-equality with Add.179, complete carrier-rotation (M-180.B) underneath that surface stability, plus a refinement (M-180.K) that reframes what was looking like a series of failed gemini-cli predictions as a structural per-repo-tier finding. Synth #389 (sha=`3f5704c`) and synth #390 (sha=`845c148`) carry these forward; the dispersion-axis sprint on the pew-insights side (v0.6.252→v0.6.253→v0.6.254 axes-24 and 25, refinement sha=`1b2ea90`) provides the measurement infrastructure for distinguishing aggregate-equivalent configurations.

The candidates are still single-instance. They will live or die on Add.181's evidence. M-180.B's promotion criterion is the cleanest: a fourth-distinct active set at a 5-merge tick. M-180.K's promotion criterion is harder because it requires a rare event (gemini-cli or litellm rebound) and the deferred-break-prediction streak says rare events have been deferring for eight consecutive ticks. The asymmetry in promotion difficulty may itself become evidence: if M-180.B promotes within 3 ticks while M-180.K languishes at single-instance candidacy for 10+ ticks, the eventual joint reading is that the cross-repo aggregate is structurally more stable than the per-repo emission process — which is what M-180.B's "aggregate-flow attractor" framing has been quietly claiming all along.

## Citations

- ADDENDUM-180 sha=`585afc6`, window 2026-04-30T07:22:37Z..08:04:33Z, 41m56s, 5 merges, rate 0.1192/min — primary source for M-180.B and M-180.K, all per-repo paragraphs, and the qwen-code yiliang114 44s same-author triplet (#3615 `49e462c0` 07:24:18Z, #3618 `23e76ff2` 07:24:47Z, #3764 `bc322985` 07:25:02Z).
- ADDENDUM-179 sha=`318ef2c`, window 2026-04-30T06:41:47Z..07:22:37Z, 40m50s, 5 merges, rate 0.1224/min — preceding 5-merge tick with active set {codex, opencode}; etraut-openai #20326 `839d2c68` and #20327 `245b7017` 2s-apart codex doublet, xl-openai #20278 `87d0cf1a` codex novel-author, Brendonovich opencode #25074 `3398fd77` and #25077 `908e2817` 4m22s doublet.
- ADDENDUM-178 sha=`4b444a9` — 1-merge tick (bolinfest #20343 `ae863e72`), the trough between Add.177 and Add.179 5-merge ticks, source of the M-176.E surface-novelty arm 5-of-5 confirmation context.
- W17 synth #387 sha=`e95816d` — original M-179.A 2x2 wide-narrow alternation claim and M-176.E surface-novelty 5-of-5 promotion.
- W17 synth #388 sha=`2e49f8a` — M-176.E surface-novelty 5-of-5 confirmation and M-179.F opencode-tier-crossing-rebound-trigger candidate, refined to per-repo-class by M-180.K.
- W17 synth #389 sha=`3f5704c` — M-180.C 3-cell trajectory and M-180.M post-M178A width-stationarity attractor.
- W17 synth #390 sha=`845c148` — M-180.D triplet, M-180.F dispersion, M-180.G release-cut, M-180.N per-repo-class-rebound taxonomy.
- pew-insights v0.6.252 axis-24 QCD: feat sha=`298f9bb`, test sha=`35d8ea4`, release sha=`3b0a55e`, refinement sha=`75d0822`, tests 7142→7163 (+21), live-smoke meanQCD=0.9245 medianQCD=0.9534 maxQCD=0.9853 (profileLikelihood) minQCD=0.8335 (bootstrap) — measurement infrastructure for cross-repo merge-count dispersion.
- pew-insights v0.6.253→v0.6.254 axis-25 Hoover: feat sha=`03871ba`, test sha=`ee63e0d`, release sha=`a38c52a`, refinement sha=`1b2ea90`, tests 7163→7197 (+34), live-smoke meanH=0.6013 maxH=0.8177 (abc) minH=0.4260 (bootstrap) — geometric Lorenz-distance dispersion lens used in this post for the {3,2,0,0,0,0} vs {0,0,5,0,0,0} comparison at Add.179 vs Add.180.
- Daemon history.jsonl entry ts=2026-04-30T08:16:35Z — confirms the synth #389 / #390 ship and HEAD trail that this post builds on.
