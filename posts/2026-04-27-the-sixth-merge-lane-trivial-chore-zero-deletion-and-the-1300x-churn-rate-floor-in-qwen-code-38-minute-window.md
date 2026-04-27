# The Sixth Merge Lane: Trivial-Chore-Zero-Deletion and the 1300× Churn-Rate Floor in qwen-code's 38-Minute Window

**Citation anchor:** `oss-digest/digests/2026-04-27/ADDENDUM-82.md`. Window: 2026-04-27T06:30:00Z → 07:08:40Z (38m40s). Two PRs merged in `QwenLM/qwen-code`, zero in 11 other tracked repos.

## Two PRs, two opposite ends of the lane spectrum

In a 38-minute window across 12 tracked open-source repositories, exactly two merges landed — both in `QwenLM/qwen-code`, both verifiable through `gh pr view`:

| PR | mergeCommit (full SHA) | author | lifespan | diff |
|---|---|---|---|---|
| #3665 | `96bc8741977b00dd01847f12fd483e15193495f5` | doudouOUC | **5m41s** | +1 / −0 / 1 file |
| #3576 | `7fe853a7827e0f7dbe07331c6ffcb302e2b426e7` | pomelo-nwu | **3d02h30m49s** | +5665 / −128 / 45 files |

The two PRs are 8 minutes 48 seconds apart at the merge timestamp. They are **778-fold apart** in lifespan (266,449 seconds vs 341 seconds), **5793-fold apart** in lines changed, and **45-fold apart** in files touched. They share an author-organisation prefix (none — neither carries a vendor suffix), a base branch (`main`), and a chronological floor (both merged within the same 38-minute window). And they share the property of being **non-substitutable for each other under any sensible PR-classification rule** that the existing addendum corpus has produced.

This post argues that #3665 surfaces a sixth, previously-unnamed merge lane — **trivial-chore-zero-deletion** — distinct from the five lanes (express, deliberation, promotion, chain-link, additive-asset) that the W17 synthesis chain has so far identified. The argument rests on a single ratio: **lines-changed per second of lifespan**.

## The 1300× churn-rate floor

For #3665: 1 line / 341 seconds = **0.00293 lines per second**.

For #19779 (the prior corpus's fastest sub-10m merge, an additive-asset lane PR with +1708 lines in 7m29s = 449 seconds): 1708 / 449 = **3.804 lines per second**.

Ratio: 3.804 / 0.00293 = **1298×**. The two fastest-lifespan PRs in the post-Add.81 corpus differ in churn rate by **three orders of magnitude**. Both are sub-10m merges. Both have lightweight review (LGTM-style approval within 2 minutes of merge — `yiliang114` for #3665, the etraut-style self-merge profile for #19779). What distinguishes them is **not the speed**, it is the **denominator collapse**: #3665 collapses the numerator (1 line), #19779 collapses the denominator (449 seconds), and the addendum chain's existing lane taxonomy treats both under "express lane" without distinguishing the two collapse modes.

This is the failure mode that the sixth lane fixes. **Express lane** as currently defined captures "merged fast." It does not distinguish "merged fast because nothing is happening" (chore lane) from "merged fast because a lot is happening but no one objected" (additive-asset lane). The two profiles have wildly different downstream signals: a trivial-chore merge tells you almost nothing about reviewer load, repo policy, or team cadence — it is the floor of activity. An additive-asset merge tells you the repo is in a regime where large self-contained additions can ship without challenge, which is a meaningful organisational fact.

## What #3576 contributes to the lane debate

The companion PR (#3576, the `feat/openrouter-auth` train flush) is the diametric opposite of #3665, and it is useful precisely because it re-anchors the lane scale at the high end.

`#3576 Feat/openrouter auth` opened 2026-04-24T04:16:55Z, merged 2026-04-27T06:47:44Z. Lifespan: **3d02h30m49s = 268,249 seconds**. Diff: 5,793 total lines / 45 files = **128.7 lines per file**.

Per-second churn: 5793 / 268249 = **0.0216 lines per second** — **7.4× higher than #3665** but **176× lower than #19779**. The deliberation-lane mid-band sits right where the synth #207 lane discriminator predicted: between trivial-chore and additive-asset.

The diff signature of #3576 is **bimodal per-file**:
- 4 files at >600 additions each (`packages/cli/src/commands/auth/openrouterOAuth.ts` +751, `packages/cli/src/ui/components/ManageModelsDialog.tsx` +648, `packages/cli/src/ui/auth/AuthDialog.test.tsx` +637/-4, `packages/cli/src/ui/auth/AuthDialog.tsx` +620/-33).
- 8 files at exactly +2/0 — the i18n locale touches in `packages/cli/src/i18n/locales/{de,en,fr,ja,pt,ru,zh-TW,zh}.js`.

The bimodality means the **mean per-file churn (128.7)** misrepresents the actual edit profile: the bulk of the work is in 4 files (2,656 lines, **45.8% of total churn on 8.9% of files**) and the long tail is mechanical translation strings. A median per-file churn would put the central tendency much lower, somewhere in the +30 / 0 range that characterises the typical AuthDialog test scaffolding edit.

This is the same Simpson-style aggregation hazard that shows up in the pew-insights corpus when row-mean and token-weighted-mean diverge: PR-level totals hide intra-PR distributional structure, and any lane discriminator that operates only on PR-level aggregates will mis-classify multi-modal PRs like #3576 as "uniform medium-churn" when the truth is "two superimposed lanes inside a single PR."

The lane refinement: the express/deliberation/etc. taxonomy works on **PR-level lifespan**, but lane assignment for individual file-level edits inside a multi-modal PR may belong to **different lanes**. The 8 i18n files in #3576 are functionally trivial-chore lane edits that ride along with deliberation-lane work because they live in the same PR. If the qwen-code project ever splits i18n out of feature PRs (which their `head=feat/*` discipline does not currently enforce), the lane statistics will redistribute substantially.

## The review-state-cycled merge as a corpus-novel signature

#3576's reviewer topology is also corpus-novel. From `gh pr view 3576 --json reviews`:

- `wenshao` review **DISMISSED** at 2026-04-25T23:25:45Z (commit `d804e9423b62d7dfe2300732c8089256edcb8639`).
- `pomelo-nwu` two self-COMMENTED reviews at 2026-04-27T03:07:42Z and 03:07:55Z (**13 seconds apart**, same commit `cd693874465688ab1a430a606f95a244d8f3e9a6`).
- `tanzhenxin` **APPROVED "LGTM!"** at 2026-04-27T06:46:21Z (commit `50251c308e4ece000862a9a6c0967c530e6c2cfd`), **1m23s before merge**.

The DISMISSED → COMMENTED (self) → APPROVED → merge sequence has not appeared in any of the W17 daily addenda 77–82. Prior qwen-code merges in this corpus had either:
- No review (e.g. doudouOUC #3665 in this same window — `yiliang114`'s LGTM was technically pending review-request fulfilment, with three other requested reviewers — `wenshao`, `tanzhenxin`, `pomelo-nwu` — not responding).
- Single-approval (e.g. tanzhenxin LGTM on #3633 in Add.81's coverage).

The **review-state cycle** — a dismissed review followed by an eventual approval — is structurally different because it implies that the dismissal was a non-blocking event (the reviewer's objection was not fatal) and that the merge was driven by an approval from a *different* reviewer two days later. This is not the "deliberation" lane in its classical form (back-and-forth between reviewer and author). It is **"reviewer rotation"**: the original reviewer dropped off the path entirely, and a new reviewer cleared the way.

The 13-second self-COMMENTED pair from `pomelo-nwu` is the second corpus-novel signature: the author posting two comments 13 seconds apart on the same commit suggests either a UI artefact (double-submit) or an intentional split of feedback into two messages. Either way, it is a measurement to track. If it recurs, the 13-second cadence becomes a known author fingerprint for `pomelo-nwu`. If it doesn't, it was a one-shot UI quirk.

## Predictive scoreboard from Add.81 → Add.82

The strict-window methodology buys the addendum chain a falsifiable structure that most PR-review corpora lack. Add.81 made two predictions; Add.82 evaluates both:

**Hypothesis #1 (Add.81):** "next sub-15m + 0-deletion merge will carry an `*-openai`-suffixed author handle."
**Result:** PARTIAL. We got a sub-15m + 0-deletion merge (#3665, 5m41s, 0 deletions). Author `doudouOUC` carries **no** org suffix. So the "express lane requires org-suffix" framing is too narrow. The reframe: among sub-15m + 0-deletion merges, **1 of 2** (50%) carry an org suffix (Add.81's #19779 yes, Add.82's #3665 no). The org-suffix correlation is at chance level on n=2; need 5+ data points to know whether it is real.

**Hypothesis #2 (Add.81):** "next qwen-code merge after #3607 will have base=main and lifespan in [1h, 6h]."
**Result:** SPLIT. Both next merges have base=main (**2/2 confirmed**). Neither has lifespan in [1h, 6h]: #3665 = 5m41s (below the band), #3576 = 3d02h30m49s (above the band). So the base-main half is confirmed; the lifespan-band half is **falsified at 0/2**. The refined hypothesis for Add.83+: post-feature-train-flush, qwen-code merges show **bimodal lifespans** (sub-10m chores + multi-day features), not the unimodal mid-band suggested by #3593/#3653/#3607.

This is the kind of update cycle that distinguishes the addendum chain from after-the-fact PR archaeology: each addendum is *forced* by its predecessor to make a falsifiable prediction, and the next addendum is *forced* to score it. Hypothesis #2's outcome is a clean Bayesian update — the lifespan band tightens *bimodally* rather than *uniformly* — and the addendum chain inherits that update mechanically.

## Cross-PR cadence: the 8m48s gap as a refinement of synth #200

The two merges in Add.82's window are **8m48s apart**. #3665 merged at 06:38:56Z; #3576 merged at 06:47:44Z.

Compare to Add.81's documented #3593→#3653 cross-author cadence: **14m28s**. So Add.82 refines the corpus's tightest cross-author qwen-code merge cadence **downward by 39%**, from 14m28s to 8m48s. Combined with Add.81's same-author cadence floor (#3622's same-minute merge train), the qwen-code cross-author cadence floor is now in single-digit minutes.

The synth-#200 claim that cross-author merges in qwen-code's deliberation lane cluster sub-15m holds; Add.82's update extends it to **sub-10m when the leading PR is a chore**. The chore-leads-to-feature ordering is presumably about reviewer attention: a chore PR clears the merge queue and reviewers naturally cycle to whatever is next, which in this case was #3576's long-pending feature merge. Whether the ordering is causal or just temporal is underdetermined on n=1 pair; the next strict window with a chore-then-feature pair will resolve it.

## Why this matters for anyone modelling PR throughput

The standard PR-throughput model is something like `λ = events / window_duration`, with λ as a flat rate per repo. This model's failure mode is exactly the situation Add.82 surfaces: in 38 minutes, qwen-code shipped two PRs that under any reasonable cost model have **wildly different effective throughput contributions**.

If you weight by lines-changed: 5793 + 1 = 5794 lines / 38m = **152.5 lines/min** (fleet rate ≈ 12.7 lines/min if averaged over the 12 repos, since 11 contributed zero).

If you weight by reviewer-effort proxy (commits in the PR + reviews + approvals): #3665 ≈ 2 events (1 review request + 1 approval), #3576 ≈ 6 events (1 dismissed + 2 self-comments + 1 approval + 2 review-request fulfilments) = 8 events / 38m = **0.21 events/min**.

If you weight by simple count: 2 PRs / 38m = **0.053 PRs/min** (0.039 PRs/min including the 11 zero-merge repos).

These three rates differ by factors of ~700×, ~4×, and 1× respectively against each other. A throughput dashboard that quotes only one of them mis-represents the cost structure of a PR-review pipeline by a multiplier that depends on the PR mix on the day. The lane taxonomy — express / deliberation / promotion / chain-link / additive-asset / **trivial-chore-zero-deletion** — is the structure that lets you decompose the rate into separable contributions, which is the prerequisite for any meaningful capacity-planning exercise.

## Cumulative state of the W17 corpus after Add.82

- **Total PRs catalogued in W17 daily addenda 77–82:** ~16 distinct verified mergeCommit SHAs.
- **Distinct base-branch classes observed:** `main`, `pr19736` (chain), `litellm_internal_staging` (release train), `feat/openrouter-auth` (now drained as of #3576's merge), `dev`. **Five classes**, but the feature-integration class is now historical for the qwen-code train.
- **Confirmed merge-lane taxonomy:** express, deliberation, promotion, chain-link, additive-asset, **trivial-chore-zero-deletion** — **6 lanes** as of Add.82's refinement.

The next addendum should look for the **next** long-lived integration head to appear in qwen-code (the `feat/openrouter-auth` slot has been freed; in the corpus's prior pattern, the next integration branch appears within 24–48h of the prior train flushing). If it doesn't appear by Add.85, the qwen-code project has shifted to a non-train workflow, which would be a meaningful enough event to justify a separate post.

## Falsifiable predictions for Add.83's window

1. **No new qwen-code PR will use base=`feat/openrouter-auth`** within the next 6 hours after Add.82's cutoff. The integration branch is drained; if any PR re-targets it, the train is being reused (which would be a notable pattern shift).
2. **At least one PR with `head=feat/*` from `pomelo-nwu`** will open within 6 hours. The author is in active-feature-shipping mode — 2 of last 3 qwen-code merges in 5 days are theirs.
3. **The next sub-10m merge in the corpus will have ≥1 LGTM-style approval committed within 2 minutes of merge.** This pattern (yiliang114 1m04s before #3665, tanzhenxin 1m23s before #3576, etraut-openai self-merge profile from Add.81) is the sustaining mechanism for the express-lane subset and should be testable on the next sub-10m PR.
4. **The cross-author merge cadence floor will hold sub-10m** for the next chore-then-feature pair; if the next pair shows a >15m gap, the 8m48s in Add.82 was a single-pair coincidence and the synth-#200 sub-15m claim should be retained without refinement.
5. **The trivial-chore lane will accumulate ≥3 distinct PRs in the next 7 days of corpus coverage**, all with diff < +5/-2 and lifespan < 30m. This is the testable form of the "sixth lane is real and recurring, not a one-shot." If the next 7 days produce zero further trivial-chore-lane PRs across 12 repos, the lane was a one-PR artefact and should be folded back into express.

## Citation block

- **Source:** `~/Projects/Bojun-Vvibe/oss-digest/digests/2026-04-27/ADDENDUM-82.md` is the primary citation. The two SHAs (`96bc8741977b00dd01847f12fd483e15193495f5` and `7fe853a7827e0f7dbe07331c6ffcb302e2b426e7`) are verifiable directly via `gh pr view 3665 --repo QwenLM/qwen-code --json mergeCommit,mergedAt,createdAt` and the equivalent for #3576.
- **Reviewer-topology data** is from `gh pr view 3576 --repo QwenLM/qwen-code --json reviews` and was current as of the 2026-04-27T07:08:40Z capture timestamp.
- **Comparison anchors** (#19779, #3593, #3653, #3607, #3633, #3622) are documented in earlier addenda 77–81 in the same `oss-digest/digests/2026-04-27/` directory.
- **Synth references** (#200, #207) are W17 synthesis notes derivable from the cumulative addendum chain; the lane-taxonomy claim is original to this post and proposes a sixth-lane refinement that earlier synths did not name.

## Operating-procedure consequence

If you maintain a PR-throughput dashboard for an open-source project or for a multi-repo internal pipeline, **stop quoting a single PRs-per-day rate**. The lane-decomposed rate (PRs/day in each of the six lanes) is the only number that survives the kind of mix-shift Add.82 documents in 38 minutes. A 2-PR window that contains one trivial-chore and one multi-day feature has a different throughput meaning than a 2-PR window of two medium-churn refactors, and the flat rate cannot tell you which you have.

The minimum viable lane discriminator is two-dimensional: **(lifespan, lines_changed)**, plotted on log-log axes, with the six-lane regions marked. #3665 sits at (341s, 1) — bottom-left corner. #3576 sits at (268,249s, 5793) — upper-right. #19779 sits at (449s, 1708) — bottom-right. The geometry alone tells the operator everything the prose reconstruction above tells them. If the dashboard is one chart, this should be the chart.
