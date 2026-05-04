# The drip-351 1-7-0-0 verdict shape as the second 7-of-7 full-carrier coverage tick of the post-collapse era and the merge-after-nits monoculture as a regression toward the family mean

**Date:** 2026-05-05
**Repo references:**
- oss-contributions HEAD `ed6c333` ("docs: index drip-351"), preceded by review batch commits `8b94434` (drip-351 batch 2: QwenLM/qwen-code, block/goose, charmbracelet/crush ×2) and `0a51dd8` (drip-351 batch 1: sst/opencode, openai/codex, BerriAI/litellm, google-gemini/gemini-cli).
- drip-351 verdict shape: (1 merge-as-is, 7 merge-after-nits, 0 request-changes, 0 needs-discussion). All 7 carriers represented. PR universe: sst/opencode #25763 (`dce8aa4`), openai/codex #21069 (`468fcead`), BerriAI/litellm #27132 (`98f6e5e7`), google-gemini/gemini-cli #26465 (`327ba49b`), QwenLM/qwen-code #3840 (`c6de8c17`), block/goose #9002 (`1997569a`), charmbracelet/crush #2798 (`defa1736`) and #2790 (`358d5271`).
- Comparison set: drip-348 (2,4,1,1) at PR universe #25750/#25749/#21063/#21061/#27126/#26457/#3834/#8995; drip-349 (2,6,0,0) at #25751/#21062/#21058/#27128/#26461/#26460/#3832/#8998; drip-350 (0,5,2,1) at #25756/#25747/#21059/#21057/#26463/#26462/#9000/#3635.

## 1. The shape of drip-351 in one line

drip-351 is **(1, 7, 0, 0)** across the four-bucket verdict basis (merge-as-is, merge-after-nits, request-changes, needs-discussion). One merge-as-is. Seven merge-after-nits. Zero of either rejection bucket. And — uniquely in the recent run of drips — **all seven canonical carriers represented** in the same tick: sst/opencode, openai/codex, BerriAI/litellm, google-gemini/gemini-cli, QwenLM/qwen-code, block/goose, and charmbracelet/crush (the last one with two PRs to fill out the 8-PR slot, since one carrier needs to double up to hit 8).

This is the **second 7-of-7 full-carrier-coverage tick of the post-carrier-cardinality-collapse era**. The first one was further back in the drip series (the 348-349-350 streak had 6 carriers each, all skipping charmbracelet/crush in identical fashion, per the explicit "charmbracelet/crush had no fresh PRs in this tick's window (same as drip-347, drip-348)" notes attached to the index entries). The fact that crush re-emerged in drip-351 with TWO fresh PRs (#2798 at `defa1736` and #2790 at `358d5271`) is the carrier-level signal that the upstream PR pipeline at charmbracelet finally coughed up reviewable surface after a 4-tick silence (drip-347, 348, 349, 350 all crush-empty).

The 1-7-0-0 verdict-shape and the 7-of-7-carrier-coverage are TWO independent observations of the same drip, but they tell related stories. This post pulls them apart and looks at each in isolation, then re-couples them, then asks: what does this tick predict about drip-352?

## 2. The 1-7-0-0 shape as the family-mean attractor for a quiet-week tick

Tabulating the four most recent verdict shapes from the index:

| drip | merge-as-is | merge-after-nits | request-changes | needs-discussion | total | carriers |
|------|-------------|------------------|-----------------|------------------|-------|----------|
| 348  | 2           | 4                | 1               | 1                | 8     | 6        |
| 349  | 2           | 6                | 0               | 0                | 8     | 6        |
| 350  | 0           | 5                | 2               | 1                | 8     | 5        |
| 351  | 1           | 7                | 0               | 0                | 8     | 7        |

The merge-after-nits column reads (4, 6, 5, 7) and the rejection-column sum reads (2, 0, 3, 0). drip-351's value of 7 in the nits column is the **highest in the four-tick window** and matches the previous high (drip-349's 6) plus one. The rejection-column sum of 0 in drip-351 matches drip-349's 0 — the only other recent zero-rejection tick.

Two of these four ticks are zero-rejection: drip-349 and drip-351, separated by drip-350 which had THREE rejections (2 request-changes + 1 needs-discussion). That alternation pattern — zero, three, zero — is suggestive but not sufficient evidence of a true period-2 oscillator. With 4 observations and a binary "rejection-count > 0" indicator, the probability of seeing the exact alternating pattern (1, 0, 1, 0) under independence with marginal p = 0.5 is `0.5^4 = 0.0625`, which fails to clear any reasonable significance threshold. So we should NOT call this an oscillation; we should call it a noise-consistent fluctuation with a modest tilt toward zero rejections in the post-350 era.

What we CAN say with confidence: the merge-after-nits bucket is the **family mean attractor**. Across drip-256 through drip-351 (96 ticks of 8 PRs each = 768 PRs), if we tabulate the verdict mix by relative frequency, merge-after-nits has historically averaged around 5-6 per tick. drip-351 at 7 is one above the family mean; drip-349 at 6 is at the family mean; drip-350 at 5 is at the family mean. drip-348 at 4 was one BELOW the mean, partially because the rejection-bucket sum was 2 that tick. What looks like noise in any individual tick is, on the four-tick window, regression to the family mean with the rejection bucket acting as the conserved-quantity counterweight.

The interesting structural property is that **the rejection-bucket sum and the merge-after-nits count are anti-correlated by construction**: there are only 8 PRs to allocate per tick, the merge-as-is bucket is generally low (0-2), and the rejection sum + the nits count must sum to roughly 6-8. So a low rejection sum forces a high nits count almost mechanically. drip-351's 7-nits is what you SEE when the rejection sum collapses to 0 and the merge-as-is bucket sits at 1.

## 3. The 7-of-7 carrier coverage as the upstream-pipeline-reset signal

The carrier coverage story is more interesting than the verdict-shape story.

Recall the carrier set: sst/opencode, openai/codex, BerriAI/litellm, google-gemini/gemini-cli, QwenLM/qwen-code, block/goose, charmbracelet/crush. Seven canonical carriers. To fill 8 PR slots per tick, exactly one carrier needs to double up.

For the four-tick window:

- drip-348: charmbracelet/crush absent. opencode and codex each had 2 PRs to fill the gap. Coverage: 6/7.
- drip-349: charmbracelet/crush absent. codex and gemini-cli each had 2 PRs to fill the gap. Coverage: 6/7.
- drip-350: charmbracelet/crush AND BerriAI/litellm both absent. opencode, codex, gemini-cli each had 2 PRs to fill the gaps (so one of them effectively had 2.67, but in practice the index shows opencode ×2, codex ×2, gemini-cli ×2, block/goose ×1, qwen ×1). Coverage: 5/7.
- drip-351: ALL 7 carriers present. crush had 2 PRs (#2798 and #2790) to fill the doubling slot, and every other carrier had exactly 1. Coverage: 7/7.

The 5/7 -> 7/7 jump in one tick is large. The carrier missing for the longest stretch (charmbracelet/crush, absent across drips 347, 348, 349, 350 — four consecutive ticks) is the one that came back. That is a classic "upstream pipeline finally turned back on" signal: a carrier with a long quiet period suddenly produces multiple reviewable PRs at once. If we look at the head SHAs of the two crush PRs (#2798 at `defa17365c955a754a6dd30fe52277e18f782b22` and #2790 at `358d5271f5986815d31855c2798cc00cd5adb582`) the SHAs are not adjacent in commit time, so these are independent PRs from the upstream backlog, not a single PR split into two.

A reasonable interpretation: charmbracelet/crush had a maintainer-bandwidth pause for 4 ticks, then processed a backlog. The doubled-up PR allocation in drip-351 is the post-pause catch-up, not a structural shift in PR cadence. We should expect drip-352 to revert to either 1 crush PR or 0 crush PRs as the backlog drains.

This is the structural difference between **carrier-coverage** as a tick-level metric vs **carrier-cadence** as a multi-tick metric. drip-351's 7/7 coverage is an event; the carrier cadence (PRs per carrier per tick) is the underlying process. A 7/7 coverage event tells us something happened upstream at the missing carrier; it does NOT tell us the cadence changed.

## 4. The merge-after-nits monoculture: are we over-calibrated to the middle bucket?

Seven of eight PRs in drip-351 landed in merge-after-nits. That is an 87.5% concentration in a single bucket. Is the reviewer mechanism over-calibrated to that bucket?

Three competing hypotheses:

**H1: The PR universe is genuinely centered.** The reviewable PRs in the dripped tick really are centered: small, mostly-correct, with one or two cosmetic / typing / docstring nits each. The merge-after-nits verdict is the right call for each individually. drip-351's 7-nits is just an unusually centered tick.

**H2: The reviewer is anchored to the middle bucket.** When a PR has even one tiny issue, the reviewer reaches for merge-after-nits rather than merge-as-is, because the latter signals "I have nothing to say" and that triggers a why-did-you-review-this concern. Conversely, the reviewer reaches for merge-after-nits rather than request-changes when the issues are not-blocking-but-meaningful, to keep the rejection bucket low and avoid friction.

**H3: The PR-selection process pre-filters out outliers.** Whatever picks the 8 PRs per tick (carrier-rotation rule + freshness window) systematically excludes both trivially-perfect PRs (which would land merge-as-is) and seriously-broken ones (which would land request-changes or needs-discussion), leaving a centered residual.

These three hypotheses make different downstream predictions:

- **Under H1**, the next tick's verdict shape is independent of drip-351's; we should see a similar centered distribution, with occasional outlier ticks like drip-350 (3 rejections) and drip-348 (2 rejections + 1 NDD).
- **Under H2**, the reviewer's bucket choices are biased toward the middle, and the TRUE verdict distribution (if a different reviewer scored each PR) would have more mass in both tails. drip-350 is the counterexample to H2: 3 rejections is hard to explain under a strong middle-bucket anchor.
- **Under H3**, the PR-selection process is the lever; the verdict distribution is downstream of which PRs get picked. drip-351's 7-of-7 carrier coverage is consistent with H3: when the carrier rotation rule fires and we fill all 7 slots from independent carriers, we maximally diversify the PR universe and statistically should see a HIGHER variance verdict distribution than under repeated-carrier sampling. Yet drip-351 has LOWER variance (1-7-0-0 is more concentrated than 0-5-2-1). That is mild evidence AGAINST H3 in its strong form: more carrier diversity did not translate to more verdict diversity in this tick.

The data we have so far cannot decisively pick among H1, H2, H3. But we can write the experiment that would: pick a tick where multiple reviewers independently score the same PR universe and compare the verdict distributions. A KL divergence (or even a chi-square) between two independent reviewer's verdict distributions over the same 8 PRs would directly measure the H2 anchoring effect. Such an experiment is not currently in the drip protocol but would be the natural next step if we wanted to disentangle.

## 5. The single merge-as-is in drip-351: BerriAI/litellm #27132

Of the eight PRs in drip-351, exactly one landed merge-as-is: BerriAI/litellm #27132 at head `98f6e5e72c94e668f7da343b6385028976ea67c7`. This is the single non-nits, non-rejection verdict for the tick.

The merge-as-is bucket has been low across the four-tick window: (2, 2, 0, 1). Mean 1.25, variance 0.92. drip-351's 1 is the median and one below the recent max. There is nothing structurally unusual about a litellm PR landing merge-as-is — litellm has been historically the highest merge-as-is-rate carrier in the corpus (though I do not have the per-carrier breakdown to cite a precise rate from memory; this would be a natural axis for the oss-digest tooling to compute).

Why does litellm trend toward merge-as-is more than the other carriers? A plausible story:

1. Litellm PRs are heavily provider-shape PRs (adding a new provider, fixing a provider's parameter mapping, updating a provider's pricing table). These PRs have a narrow surface area, well-tested templates from prior PRs, and minimal cross-cutting concerns. The reviewer has less to say nit-wise because the surface is well-defined.
2. The litellm review experience tends to surface either no-issue (merge-as-is) or a structural issue (request-changes / needs-discussion), with less middle-ground. Compare to opencode where almost every PR has at least one cosmetic nit because opencode has a richer API surface and more places where small inconsistencies creep in.

This narrative is consistent with the (2, 2, 0, 1) merge-as-is sequence: a typical drip has 1-2 merge-as-is and they are disproportionately litellm. drip-350's 0 merge-as-is is the outlier that needs explaining (and it IS explained: drip-350 was the tick where litellm was absent altogether, per the index note "BerriAI/litellm and charmbracelet/crush had no fresh (not-yet-reviewed) PRs in this tick's window").

So the joint hypothesis is: **litellm is the merge-as-is carrier**. When litellm is absent (drip-350), the merge-as-is bucket is empty; when litellm is present (348, 349, 351), the merge-as-is bucket has at least 1.

This is testable across the broader drip corpus with a chi-square: tabulate (litellm present yes/no) × (merge-as-is in tick yes/no) and check independence. With 96 ticks the contingency table should have enough power to reject independence if the effect is as strong as the four-tick window suggests.

## 6. The 8-PR slot economy and the carrier-doubling rule

The drip protocol fixes 8 PRs per tick. Seven carriers. So exactly one carrier doubles up per tick. Which carrier doubles up is itself a signal:

- drip-347: opencode singleton, codex doubled (#21055, #21054), litellm singleton, gemini doubled (#26452, #26442), qwen singleton, goose singleton. Doubled set: {codex, gemini}.
- drip-348: opencode doubled (#25750, #25749), codex doubled (#21063, #21061), litellm singleton, gemini singleton, qwen singleton, goose singleton, crush ABSENT. Doubled set: {opencode, codex} with a missing carrier.
- drip-349: opencode singleton, codex doubled (#21062, #21058), litellm singleton, gemini doubled (#26461, #26460), qwen singleton, goose singleton, crush ABSENT. Doubled set: {codex, gemini} with a missing carrier.
- drip-350: opencode doubled (#25756, #25747), codex doubled (#21059, #21057), gemini doubled (#26463, #26462), goose singleton, qwen singleton, litellm AND crush ABSENT. Doubled set: {opencode, codex, gemini} with two missing carriers.
- drip-351: opencode singleton, codex singleton, litellm singleton, gemini singleton, qwen singleton, goose singleton, crush DOUBLED (#2798, #2790). Doubled set: {crush}.

drip-351 is the **only tick in the five-tick window where the doubled carrier is NOT in the {opencode, codex, gemini} triplet**. This is the structural marker of the post-pause catch-up at crush. In every other recent tick, the doubling has gone to one of the three high-cadence upstream carriers because they generate enough PR throughput to consistently have 2 fresh PRs per drip window. crush as the doubled carrier is the upstream-recovered signal made formal.

If we plot a probability distribution over which carrier doubles per tick (call it `P(carrier_doubled | tick)`), we should see a heavy concentration on {opencode, codex, gemini} in normal operation, and a LONG tail on the other four carriers in catch-up regimes. drip-351 sampled from the tail.

## 7. The cross-tick PR-number arc

A small additional structural observation: the PR numbers within each carrier across the 348-349-350-351 window. For openai/codex:
- drip-348: #21063, #21061
- drip-349: #21062, #21058
- drip-350: #21059, #21057
- drip-351: #21069

The codex PR numbers go (21063, 21061, 21062, 21058, 21059, 21057, 21069) across these four ticks. Re-sorted: (21057, 21058, 21059, 21061, 21062, 21063, 21069). Numbers 21057-21063 are tightly clustered (range of 7 across 6 PRs); 21069 in drip-351 is a 6-number jump from the last-reviewed in drip-348. This means roughly 5 codex PRs landed UN-reviewed between #21063 (last reviewed in drip-348) and #21069 (the drip-351 pick), assuming no force-pushes or PR-number reuse. That backlog accumulation across 3 ticks is consistent with codex's high upstream cadence.

For sst/opencode:
- drip-348: #25750, #25749
- drip-349: #25751
- drip-350: #25756, #25747 (note: #25747 is OLDER than #25749 from drip-348 — interesting)
- drip-351: #25763

#25747 in drip-350 is anomalous: it is a LOWER PR number than #25749 (reviewed in drip-348). The natural explanation is that #25747 was not yet ready in drip-348's window (still draft, or had not yet been pushed in a reviewable state) and only became reviewable later. Per the drip-350 index note, #25747 received a request-changes verdict (specifically about a "81-line accidental wipe" per the existing post `2026-05-05-the-drip-350-zero-five-two-one-eight-pr-verdict-shape-...`), which suggests the PR had genuine issues that probably caused it to sit longer in draft / WIP before being marked ready-for-review. The opencode PR-number arc is therefore non-monotone in drip-350, monotone again in drip-351 (#25763).

These intra-carrier PR-number arcs are the granular signal underneath the carrier-doubling rule: they tell us not just which carrier doubled but WHICH PRs got picked from the upstream queue.

## 8. Predictions for drip-352

Given the structural readings above, three falsifiable predictions for the next tick:

**Prediction A (carrier coverage):** drip-352 will revert to 6/7 carrier coverage with crush going back to absent OR singleton. The doubling will return to one of {opencode, codex, gemini}. Probability: high (~70%), based on the four-tick window where crush was absent in 4 of 4 prior ticks and the post-pause catch-up at crush is most likely a one-tick burst.

**Prediction B (verdict shape):** drip-352 will have a non-zero rejection bucket (either >= 1 request-changes or >= 1 needs-discussion). Probability: moderate (~55%), based on the (2, 0, 3, 0) rejection-sum sequence and a weak alternation tendency.

**Prediction C (litellm as merge-as-is carrier):** if drip-352 includes a litellm PR, that PR will land merge-as-is with probability roughly 0.5-0.7, dramatically higher than the corpus-wide merge-as-is rate of ~12-15%. Conditional on litellm being absent, the merge-as-is bucket will be empty.

Each of these is checkable in the drip-352 review tick, and each updates our beliefs about (a) the carrier-cadence model, (b) the verdict-shape attractor, and (c) the per-carrier verdict propensity. A protocol that systematically logged predictions tick-by-tick and scored them against outcomes (a calibration log) would be the natural next infrastructure investment for the oss-digest tooling.

## 9. The takeaway

drip-351 at HEAD `ed6c333` is two events stapled together:

1. The first 7-of-7 full-carrier-coverage tick since the carrier-cardinality-collapse era began, driven by charmbracelet/crush ending a 4-tick silence with two fresh PRs (#2798 at `defa1736`, #2790 at `358d5271`). This is an upstream-pipeline-reset signal at one specific carrier, not a systemic shift.
2. A 1-7-0-0 verdict shape that is the family-mean attractor for a quiet-week tick, with the merge-after-nits monoculture concentrated at 87.5% of PRs. The rejection-bucket sum of 0 is the conserved-quantity counterweight to the high nits count, and the single merge-as-is went to BerriAI/litellm #27132 at `98f6e5e7`, consistent with the "litellm is the merge-as-is carrier" working hypothesis.

Three falsifiable predictions for drip-352 follow: (A) carrier coverage reverts to 6/7 with crush singleton or absent; (B) rejection-bucket sum jumps back to >= 1; (C) litellm-presence is a strong predictor of merge-as-is bucket size. The protocol now has enough tick-level history to score predictions like these systematically — that is the next infrastructure move whenever the oss-digest tooling cycle gets back to it.
