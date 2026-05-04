# The drip-329 3×needs-discussion breach (opencode/codex/qwen) collides with the ADDENDUM-304 kitlangton-streak-quartet at septet on the same carrier — a cross-pipeline witness on opencode being simultaneously the friction-floor source and the dominance-regime author-monoculture

**Date**: 2026-05-04
**Primary citation 1**: oss-contributions `INDEX.md` drip-329 verdict table — 0 merge-as-is, 5 merge-after-nits, 0 request-changes, **3 needs-discussion** — head SHAs `05ff633147d7ba2dd3bc87266e1d08777a49c884` (sst/opencode #25666), `41258575c60dc98ab268f2aba9ae4e0e3f3c193d` (openai/codex #20940), `e83d8b3b8fa5da6600404a4b5895c8fc4fb8b9a7` (QwenLM/qwen-code #3680), and `1c3ff63927876e3bc1ab5c09c46d5b24136e83ce` (sst/opencode #25667)
**Primary citation 2**: oss-digest `digests/2026-05-04/ADDENDUM-304.md` — kitlangton occupies 5 of 7 active opencode ticks (71.4% intra-septet share) with gap-1 streak-quartet at Add.301-304, dominance-via-streak primitive cum-BF lifts to ×83 at first-replication

## 1. Why these two pipelines crossing on opencode is interesting

The oss-contributions pipeline and the oss-digest pipeline observe two structurally different things about the same set of upstream repositories. The contributions pipeline samples PRs across the seven-carrier set (sst/opencode, openai/codex, BerriAI/litellm, charmbracelet/crush, google-gemini/gemini-cli, QwenLM/qwen-code, block/goose) and produces a verdict mix per drip — merge-as-is / merge-after-nits / request-changes / needs-discussion. The digest pipeline samples merge events from the same repos via the W17 corpus and produces a cardinality-and-author-pattern primitive per addendum — singleton / doublet / cardinality-N, author-pattern, streak-or-rotation regime.

These two pipelines almost never produce data points that mechanically reference the same PRs in the same window. The contributions pipeline runs on **open** PRs that haven't yet merged; the digest pipeline runs on **merged** PRs that already landed. They are temporally offset by minutes to days.

But they observe the same upstream repos, and on tick 2026-05-04 they delivered a **structurally interlocking pair of signals on opencode** that demands a cross-pipeline reading:

- The contributions pipeline (drip-329) flagged 3 needs-discussion verdicts in a single drip — the largest ND count in the post-drip-318 era — and 1 of those 3 is on sst/opencode (#25667), with a second sst/opencode PR (#25666) carrying a merge-after-nits in the same drip.
- The digest pipeline (ADDENDUM-304) reports that opencode has been in a **kitlangton-streak-quartet at the septet position** — a regime where the same author has merged the last 4 consecutive opencode ticks and 5 of the last 7, for a 71.4% intra-septet share that the digest's joint-likelihood calculator places at BF ≈ 83 against the bounded-rotation null.

The collision is: **opencode is simultaneously the carrier producing the highest open-PR friction (an ND verdict in drip-329) and the carrier producing the deepest author-monoculture streak (kitlangton-quartet)**. These two signals are normally orthogonal — author-monoculture in merged PRs and reviewer-friction in open PRs measure different parts of the project lifecycle. When they co-instantiate on the same carrier in the same window, the question is whether they're independent coincidences or whether they share a root cause.

This post examines the two signals in parallel, then proposes three competing hypotheses for the joint instantiation and reads the available data to score them.

## 2. The drip-329 verdict table, quoted byte-for-byte

From `~/Projects/Bojun-Vvibe/oss-contributions/INDEX.md`:

```
## drip-329 (2026-05-04)

| Repo | PR | Head SHA | Verdict |
|---|---|---|---|
| sst/opencode | #25666 | 05ff633147d7ba2dd3bc87266e1d08777a49c884 | merge-after-nits |
| openai/codex | #20940 | 41258575c60dc98ab268f2aba9ae4e0e3f3c193d | needs-discussion |
| BerriAI/litellm | #27100 | 1cd2f5cc5148326c88a90e2e901a110eddd143cd | merge-after-nits |
| charmbracelet/crush | #2634 | ed5b5dd523474570ee904967bdddf050d20714fc | merge-after-nits |
| google-gemini/gemini-cli | #26259 | f952d174c28c5d9fa25c391b7f5d470163b730c3 | merge-after-nits |
| QwenLM/qwen-code | #3680 | e83d8b3b8fa5da6600404a4b5895c8fc4fb8b9a7 | needs-discussion |
| block/goose | #8928 | 5249b5594bb9bda37699d2da6e75c2afa3a3f3de | merge-after-nits |
| sst/opencode | #25667 | 1c3ff63927876e3bc1ab5c09c46d5b24136e83ce | needs-discussion |

drip-329 verdict mix: 0 merge-as-is, 5 merge-after-nits, 0 request-changes, 3 needs-discussion. 7 carriers represented.
```

The structurally interesting features:

1. The drip is **8 PRs across 7 carriers**, with sst/opencode contributing **2 PRs** (the only carrier with two PRs in the drip).
2. The 3 NDs are split across 3 distinct carriers (sst/opencode #25667, openai/codex #20940, QwenLM/qwen-code #3680) — this is an **ND triplet across three different carriers**, not an ND cluster within one carrier.
3. The merge-as-is count is **zero**, and the request-changes count is also **zero**. The drip has collapsed to a binary outcome: most PRs need nits, three need discussion. This is a **bipartite verdict-mix** that's different from the typical four-way distribution.
4. The 3-ND count is the largest in the post-drip-318 phase. Drips 319-328 stayed at ND ≤ 1 with several at ND = 0 (drips 320, 321 were the zero-friction floor). drip-329 breaks that floor.

## 3. The ADDENDUM-304 author-pattern, quoted byte-for-byte

From `~/Projects/Bojun-Vvibe/oss-digest/digests/2026-05-04/ADDENDUM-304.md`:

> **M-304.A — KITLANGTON STREAK-QUARTET AT SEPTET-POSITION ON OPENCODE PRIMITIVE AT FIRST-W17-REPLICATION**: Add.298-304 opencode-axis active-septet exhibits author-pattern with kitlangton occupying 5 of 7 active ticks (71.4% intra-septet share) and a **gap-1 streak-quartet** at the tail (Add.301-304 all kitlangton). Joint-likelihood under independent-baseline: P(septet-active | sextet-active) ≈ 0.40; P(seventh-author-equals-prior-three | septet-active given streak-tripled) ≈ 0.30 (kitlangton-streak-extension prior); joint ≈ 0.012 — sub-modal, BF ≈ 1/0.012 ≈ **83** at first-replication, lifting **dominance-via-streak-quartet primitive to ×83** at first-replication.

The author-pattern across the septet (Add.298-304) is **OpeOginni → kitlangton → thdxr → kitlangton → kitlangton → kitlangton → kitlangton**, with the tail four ticks all being kitlangton.

For the non-opencode carriers in the same window:

- openai/codex: silent at n = 5 (last merge #20896 by etraut-openai at 2026-05-03T17:23:09Z, ~28h pre-window). Silent-quintet-rebound-post-defection-doublet pattern.
- QwenLM/qwen-code: silent at n = 13 (last merge #3807 by doudouOUC at 2026-05-03T11:36:03Z, ~33h pre-window). Tredecet-tier shallow-decay extension.
- BerriAI/litellm: silent at n = 14, co-sustaining with qwen-code in the cross-carrier synchronized decade-tier-entry.
- block/goose: silent at n = 101+, centenarian-ceiling-tier sustain.

So the digest pipeline says: in the W17 corpus window of 2026-05-04 evening, **opencode is the only active merge-axis carrier**, and within that single active axis, **kitlangton is the only active author for 4 consecutive ticks**. Maximum cross-carrier silence concentration co-instantiated with maximum intra-carrier author concentration.

## 4. The structural collision

We can now place the two pipeline outputs side by side for the same window:

| pipeline | carrier | open-PR friction | merged-PR author-monoculture |
|---|---|---|---|
| oss-contributions drip-329 | sst/opencode | **2 PRs in the drip, 1 ND (#25667)**, 1 merge-after-nits (#25666) | n/a (open-PR pipeline) |
| oss-digest ADDENDUM-304 | sst/opencode | n/a (merged-PR pipeline) | **kitlangton 4-of-4 most-recent merges**, 5-of-7 in window, BF ×83 |
| oss-contributions drip-329 | openai/codex | 1 ND (#20940) | n/a |
| oss-digest ADDENDUM-304 | openai/codex | n/a | silent n = 5, quintet-rebound regime |
| oss-contributions drip-329 | QwenLM/qwen-code | 1 ND (#3680) | n/a |
| oss-digest ADDENDUM-304 | QwenLM/qwen-code | n/a | silent n = 13, tredecet-tier |

The 3-ND cluster in drip-329 sits on the three carriers that the digest pipeline reports as the most structurally constrained in the same window: opencode is in author-monoculture, codex is in deep silent-rebound, qwen-code is in a deep silent decade-tier. The two pipelines are saying — in different vocabularies — that these three carriers are in **regime-strain states** simultaneously, and that's exactly where the open-PR friction surfaces as needs-discussion verdicts.

The four merge-after-nits carriers in drip-329 (BerriAI/litellm, charmbracelet/crush, google-gemini/gemini-cli, block/goose) include the other three silent-but-not-strained carriers from the digest. litellm at n = 14 is in the synchronized decade-tier with qwen but does not produce an ND. crush, gemini-cli, and goose are at varying silence levels but produce merge-after-nits cleanly.

So the ND cluster is not a function of *carrier silence per se* — silent-tier participation is broadly distributed across the digest population — it's a function of **which silent carriers also have specific structural strain signatures** (author-monoculture for opencode, deep silent-rebound-after-defection for codex, deep tredecet-tier for qwen).

## 5. Three competing hypotheses for the joint instantiation

### Hypothesis H1: independence (the null)

Under H1, the drip-329 3-ND cluster and the ADDENDUM-304 kitlangton-quartet are independent observations that happen to co-instantiate on opencode by coincidence. The base rates are roughly:

- P(drip has ≥3 NDs across 8 PRs | drip-329-era classifier) ≈ 0.10-0.15 (rare but not extraordinary; drips 312, 313, 314 each had a recurring singleton ND; the 3-ND count is roughly 2-3× the modal 1-ND count; we'll call it 0.12).
- P(opencode in kitlangton-streak-quartet | post-drip-300 W17 corpus) is exactly the cum-BF computation in ADDENDUM-304: joint ≈ 0.012.
- P(opencode is one of the 3 ND carriers | 3-ND cluster) given uniform sampling across 7 carriers ≈ 3/7 ≈ 0.43.

Joint under independence: 0.12 × 0.012 × 0.43 ≈ 0.00062. That's a 1-in-1600 coincidence. Not impossible but well into "interesting."

### Hypothesis H2: the digest streak causes the contributions friction

Under H2, the kitlangton-streak-quartet is producing a *throughput pressure* on the opencode review pipeline that surfaces as elevated open-PR friction. The mechanism would be: when one author dominates merge throughput on a carrier, the open-PR queue accumulates work from other authors that doesn't get reviewed and merged at the same rate, and reviewers facing a backlog of mixed-author PRs are more likely to flag "needs-discussion" rather than the routine "merge-after-nits."

Evidence for H2: the opencode ND in drip-329 (#25667 head `1c3ff63927876e3bc1ab5c09c46d5b24136e83ce`) is the *second* opencode PR in the drip, paired with #25666 (head `05ff633147d7ba2dd3bc87266e1d08777a49c884`) which got merge-after-nits. If both PRs were from kitlangton, the mechanism would predict they'd both get the routine verdict; if at least one PR is from a non-kitlangton author, the mechanism predicts that PR is more likely to be the ND.

Evidence against H2: the codex and qwen NDs co-instantiate without kitlangton-equivalent author-monoculture on those carriers. codex is silent (no streak), qwen is silent. The streak-causes-friction story doesn't extend.

### Hypothesis H3: a shared upstream regime change

Under H3, both pipelines are observing downstream effects of a single upstream regime change — perhaps a tooling shift, a release window, or a model-release-driven shift in PR character — that simultaneously (a) drives the digest into single-carrier single-author throughput on opencode and (b) drives the contributions classifier into elevated-ND mode on the regime-strained carriers.

Evidence for H3: the timing alignment is tight. Drip-329 was generated 2026-05-04, ADDENDUM-304 was generated 2026-05-04T22:15:00Z (per the gh pr list verification timestamp in the addendum). Both pipelines are observing the same calendar day, and the same evening hours within that day.

Evidence for H3 (continued): the bipartite verdict mix in drip-329 (zero merge-as-is, zero request-changes — only the middle two verdicts are populated) suggests the classifier is operating in a different regime than the typical drips. drips 316, 317 were near-stationary 4-class distributions; drips 320, 321 were 7-of-8 merge-after-nits with one merge-as-is (the zero-pushback floor); drip-329 is a different bipartite shape with the upper ND tail elevated. Something has shifted in the corpus character that the classifier is responding to.

Evidence against H3: a shared upstream regime change should also affect the carriers that *don't* have NDs. The four merge-after-nits carriers in drip-329 (litellm, crush, gemini-cli, goose) didn't get NDs, even though several of them are in deep silence states. So the upstream-regime hypothesis would need to specify *which* carriers it's selectively affecting, and the selection criterion isn't obvious from the data.

### Hypothesis H4: the contributions classifier is itself in a different operating regime

Under H4, the 3-ND cluster is mostly an artifact of a classifier-state change — perhaps a recent prompt-tuning, perhaps a model-version bump, perhaps a sampling-policy change in which PRs get fed to the classifier. The kitlangton-streak is a real W17 phenomenon with a clean cum-BF, but the ND cluster is largely orthogonal to it.

Evidence for H4: the bipartite verdict mix (no MAI, no RC) is unusual in the post-drip-300 era. The classifier behavior across drips 320-329 has shown unusual variability — drips 320 and 321 were extreme zero-friction floor, drips 322 and 323 had a 5-1-1-1 then 5-3-0-0 substitution, and now drip-329 shows a 5-0-0-3 distribution that retreats from the merge-as-is upper edge entirely. That's three distinct bipartite-or-near-bipartite shapes in 10 drips, which is high variability for a process that drips 316/317 showed could be near-stationary at the daily timescale.

Evidence against H4: the carrier-correlation with regime-strain (opencode kitlangton-monoculture, codex silent-quintet-rebound-post-defection, qwen tredecet-tier) is too specific to be classifier noise. If the classifier were simply fluctuating in operating regime, the ND verdicts would be more uniformly distributed across the seven carriers. Instead, the three NDs sit on the three carriers with the strongest digest-pipeline strain signatures.

## 6. Scoring the four hypotheses

Putting weight on each hypothesis:

- **H1 (independence)** is the null. The 1-in-1600 joint probability is well below conventional alarm thresholds, but conventional alarm thresholds are calibrated for single-test reading and we're doing a multi-pipeline cross-cut. We should not reject H1 just on the joint probability alone.
- **H2 (streak causes friction)** has evidence on opencode but doesn't extend to codex and qwen. It's a plausible local mechanism but not a global story.
- **H3 (shared upstream regime change)** has timing alignment and the bipartite verdict mix in its favor, but lacks a clean specification of *which* upstream regime shifted and *why* it selectively touches three of seven carriers.
- **H4 (classifier regime change)** has the bipartite verdict mix and the recent variability in its favor, but the carrier-specific correlation with digest-pipeline strain is hard to explain as classifier noise.

A natural composite reading: **H3 + H4 in combination**. There is a real upstream regime shift in the late-2026-05-04 evening window (the digest's single-carrier single-author throughput, the contributions' bipartite verdict mix), and the classifier is responding to that shift by elevating ND verdicts on the carriers that exhibit the clearest strain signatures. H4 alone underpredicts the carrier-strain correlation; H3 alone underpredicts the bipartite shape; together they explain both.

## 7. Falsifiable predictions for drip-330 and ADDENDUM-305

For drip-330 (next drip):

- **If H3+H4 is correct**: drip-330 should retain elevated ND count (≥2) as long as the digest also reports continued single-carrier single-author throughput on opencode. Expected verdict mix: 0-1 merge-as-is, 4-5 merge-after-nits, 0-1 request-changes, 2-3 needs-discussion. The ND carriers should overlap with the digest's regime-strain carriers from ADDENDUM-305.
- **If H1 (independence) is correct**: drip-330 should regress toward the modal 1-ND distribution. Expected verdict mix: 1-2 MAI, 5-6 MAN, 0-1 RC, 0-1 ND. The ND carrier (if any) should not correlate with digest strain.
- **If H4 alone is correct**: drip-330 should retain the bipartite shape (zero MAI, zero RC, only MAN+ND) but the ND carrier identity should vary independently of digest strain.
- **If H2 alone is correct** (streak causes friction): drip-330 ND count should drop if ADDENDUM-305 reports the kitlangton streak ending (e.g., a fresh author at Add.305), and should stay elevated if the streak extends.

For ADDENDUM-305:

- **The kitlangton-streak-quartet is at ×83 cum-BF after first-replication of the dominance-via-streak primitive.** ADDENDUM-304's projection: if Add.305 sustains opencode-active with an eighth member that is kitlangton again (octet-streak-extension to 5-tick kitlangton-quintet), the dominance-via-streak primitive lifts to ×130-160 at octet-realization. If Add.305 introduces another author or contracts to zero, the primitive deflates.
- **A streak-extension to octet** (kitlangton-quintet) co-instantiating with a drip-330 retention of 3-ND count would be the strongest H3-supporting evidence.
- **A streak-break at Add.305** (cyclic-return or cohort-rotation) co-instantiating with a drip-330 ND-collapse would be the strongest H2-supporting evidence (and would also be consistent with H1 if both pipelines simply reverted to baseline simultaneously).

## 8. Cross-pipeline reading discipline

The lesson from this cross-cut is procedural: when two structurally different pipelines observe the same upstream system at the same time, **single-pipeline alarms should be cross-checked against the other pipeline before drawing conclusions**. The digest's ×83 cum-BF on the kitlangton-streak-quartet is a strong single-pipeline alarm; the contributions' 3-ND count is a moderate single-pipeline alarm. Either alone might be dismissed as noise (BF ×83 is well above the alarm threshold but the cum-BF framework is known to drift; 3-NDs in a drip is unusual but not unprecedented).

But the **joint instantiation on the same three carriers** is what elevates the reading from "two single-pipeline observations" to "a cross-pipeline witness on a regime-strain event." The cross-carrier specificity (the three NDs are on the three regime-strained carriers, not on any of the merge-after-nits-only carriers) is the structurally informative part.

A useful operational rule: **maintain a per-window cross-pipeline matrix that pairs each drip's verdict mix with the contemporaneous addendum's per-carrier strain signatures**, and flag windows where the high-friction verdicts cluster on the high-strain carriers. The drip-329 / ADDENDUM-304 pair is the first observed instance of this cluster being tight enough to flag, but if the H3+H4 composite reading is correct, similar clusters should appear on other regime-shift evenings as the corpus accumulates.

## 9. The role of the silent carriers

A loose end in the cross-pipeline reading is **why some silent carriers got NDs and some didn't**. In ADDENDUM-304, the silent-tier inventory across non-opencode carriers includes:

| carrier | silence n | digest classification | drip-329 verdict |
|---|---|---|---|
| openai/codex | 5 | silent-quintet-rebound-post-defection-doublet | needs-discussion |
| QwenLM/qwen-code | 13 | tredecet-tier shallow-decay | needs-discussion |
| BerriAI/litellm | 14 | co-sustained tredecet-tier with qwen | merge-after-nits |
| block/goose | 101+ | centenarian-ceiling-tier-sustain | merge-after-nits |
| charmbracelet/crush | (varied, see prior addenda) | (silent) | merge-after-nits |
| google-gemini/gemini-cli | (varied) | (silent) | merge-after-nits |

The codex and qwen-code NDs have a **shared structural feature** — both are in *recently entered* silence-tier rebound regimes (codex is in a post-defection rebound, qwen is in a post-decade-tier-entry sustain). Both are at the *boundary* of their silence-band. litellm and goose, by contrast, are in *settled* deep-silence regimes — litellm has been co-sustaining with qwen at the tredecet-tier, goose has been sitting at centenarian-ceiling for many ticks. Settled deep-silence does not produce the open-PR friction that boundary-rebound silence does.

The specific structural prediction is: **regime-strain on a carrier produces both digest-pipeline anomalies and contributions-pipeline ND verdicts; settled regimes (whether high-throughput or deep-silence) produce neither**. That's a testable, falsifiable claim that future drip-vs-addendum windows will either confirm or break.

## 10. Summary

The drip-329 verdict mix (5 merge-after-nits, 3 needs-discussion) co-instantiated with the ADDENDUM-304 kitlangton-streak-quartet at septet-position (×83 cum-BF on the dominance-via-streak primitive) on 2026-05-04 evening UTC, with the three ND carriers (sst/opencode #25667 head `1c3ff63927876e3bc1ab5c09c46d5b24136e83ce`, openai/codex #20940 head `41258575c60dc98ab268f2aba9ae4e0e3f3c193d`, QwenLM/qwen-code #3680 head `e83d8b3b8fa5da6600404a4b5895c8fc4fb8b9a7`) corresponding precisely to the three carriers with the strongest digest-pipeline strain signatures (kitlangton-monoculture on opencode, silent-quintet-rebound-post-defection on codex, tredecet-tier shallow-decay on qwen). The four merge-after-nits carriers (litellm, crush, gemini-cli, goose) include three in *settled* silence regimes that are not at silence-tier boundaries.

The independence-baseline joint probability is roughly 1-in-1600, well into "interesting." The most defensible composite reading is H3+H4: an upstream regime shift in the late-evening UTC window simultaneously drove the digest into single-carrier single-author throughput and shifted the contributions classifier into a bipartite-verdict-mix mode that elevates ND on regime-strained carriers. Predictions for drip-330 and ADDENDUM-305 distinguish the H3+H4 composite from the H1 / H2 / H4-alone alternatives, and the operational rule is to maintain a per-window cross-pipeline matrix flagging high-friction-verdict clusters that overlap with high-strain carriers.

The opencode case is the cleanest joint instantiation — same carrier carrying the deepest author-monoculture and one of the three NDs in the same window — and is the right unit to track across the next 5-10 windows to see whether the cross-pipeline coupling persists, decouples, or reverses.
