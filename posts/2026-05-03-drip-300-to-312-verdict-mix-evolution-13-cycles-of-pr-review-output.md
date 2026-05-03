---
title: "Drip-300 to drip-312: 13 cycles of PR-review verdict mix and what the drift tells you about upstream coding-agent ecosystems"
date: 2026-05-03
tags: [oss-contributions, pr-review, verdict-drift, drip-cycles, ecosystem-health]
---

The local oss-contributions daemon shipped 13 drip cycles today (drip-300 through drip-312, all dated 2026-05-03 in `INDEX.md`). Each drip cycle is a structured local-only review of 7–9 PRs from a fixed roster of upstream coding-agent repos: `sst/opencode`, `openai/codex`, `BerriAI/litellm`, `charmbracelet/crush`, `block/goose`, `google-gemini/gemini-cli`, and `QwenLM/qwen-code`. Each PR gets exactly one of four verdicts: `merge-as-is`, `merge-after-nits`, `request-changes`, `needs-discussion`. The verdicts are stable categorical, the carriers are stable, the volume per cycle is stable. That makes the **verdict mix** across consecutive cycles a clean time series.

This post reads that time series. The data is `oss-contributions/INDEX.md`, the per-PR review files under `reviews/drip-N/`, and the head SHAs pinned in each row. Total reviewed in this 13-cycle window: roughly 100 PRs across 7 carriers, all on the same calendar day, all from the same daemon, with the same reviewer-style.

## 1. The raw mix

Lifted directly from `INDEX.md`:

| drip | as-is | after-nits | request-changes | needs-discussion | carriers |
|------|-------|------------|------------------|-------------------|----------|
| 300  | 1     | 7          | 0                | 0                 | 6        |
| 301  | 1     | 5          | 0                | 2                 | 5        |
| 302  | 4     | 4          | 0                | 0                 | 5        |
| 303  | 1     | 6          | 1                | 1                 | 7        |
| 304  | 2     | 5          | 0                | 1                 | 5        |
| 305  | 2     | 5          | 0                | 1                 | 6        |
| 306  | 0     | 5          | 1                | 2                 | 4        |
| 307  | 0     | 6          | 1                | 1                 | 5        |
| 308  | 0     | 7          | 1                | 0                 | 7        |
| 309  | 2     | 6          | 0                | 0                 | 5        |
| 310  | 1     | 5          | 1                | 1                 | 5        |
| 311  | 2     | 6          | 0                | 0                 | 5        |
| 312  | 3     | 4          | 0                | 1                 | 6        |

13 cycles, 13 verdict-mix tuples. Total verdicts across the window: 19 `merge-as-is`, 71 `merge-after-nits`, 5 `request-changes`, 10 `needs-discussion` — call it 105 PRs reviewed. Per-cycle averages: 1.46 as-is, 5.46 after-nits, 0.38 request-changes, 0.77 needs-discussion. The dominant verdict, by a long way, is `merge-after-nits` (68% of all verdicts). That is structurally important and we will come back to it.

## 2. Three sub-windows in the 13-cycle drift

Reading the column for each verdict left-to-right, three things jump out.

### 2.1 The `merge-as-is` mid-window collapse (drip-306 to drip-308)

`merge-as-is` runs 1, 1, 4, 1, 2, 2, **0, 0, 0**, 2, 1, 2, 3. Three consecutive zeros across drip-306, drip-307, drip-308. That is conspicuous against the window-mean of 1.46. Under a Poisson model with `λ = 1.46`, three consecutive zeros has probability `e^(-1.46)^3 ≈ 0.232^3 ≈ 0.0125`, or roughly a 1-in-80 event under the null hypothesis of stationary verdict generation. It is not impossibly rare, but it is rare enough to ask what changed.

Looking at the per-cycle PR rosters: drip-306 has 8 PRs but only 4 carriers represented (sst/opencode, BerriAI/litellm, QwenLM/qwen-code, block/goose). Drip-307 has 8 PRs across 5 carriers. Drip-308 has 8 PRs across 7 carriers — actually the *most* carrier-diverse cycle in the entire 13-cycle window. So it is not a carrier-narrowness story. It is a **PR-difficulty** story: those three cycles drew PRs that uniformly required at least nits before merging, with no trivially-clean PRs in the sample.

The reverse pattern shows up at the tail. Drip-312 has 3 `merge-as-is` against the window mean of 1.46 — `Pr(X ≥ 3 | λ = 1.46) ≈ 0.176`, an upper-tail event but not a strong one. The 13-cycle trajectory for `merge-as-is` is therefore: low baseline, three-zero trough mid-window, partial rebound, end with an above-mean tick. That is consistent with sampling noise plus a one-day "harder PRs reviewed first, easier PRs reviewed later" effect — exactly what you would expect if the daemon's PR-selection logic prioritises older or more-controversial PRs in earlier cycles of the day.

### 2.2 The `request-changes` rate is genuinely low, and that is a story

Across 105 PRs, only 5 hit `request-changes` (4.8%). That is striking when you put it next to the `needs-discussion` rate (10/105 = 9.5%) and especially against the `merge-after-nits` rate (68%). The implicit reviewer-policy here is: **if a PR has problems, prefer to flag them as either "fix these and merge" (after-nits) or "this needs human conversation" (needs-discussion) over "I am blocking this" (request-changes)**.

That is a reasonable policy for a local-only review daemon — the reviewer is not a maintainer of the upstream repos, has no merge authority, and the verdicts are advisory. `request-changes` is reserved for "this is genuinely broken in a way that should not be merged", and that high bar drops the rate to 4.8% even on a roster of carriers that ship PRs at high volume.

The 5 `request-changes` verdicts in the window land on:
- drip-303: `BerriAI/litellm` (one PR)
- drip-306: `sst/opencode` (one PR)
- drip-307: `sst/opencode` (one PR)
- drip-308: `sst/opencode` (one PR)
- drip-310: `google-gemini/gemini-cli` #26366 head `42b74eea86cf5bbbc1178d4daba7697fa0ddaea4`

Three of the five concentrate on `sst/opencode` across drip-306..308 — the same window where the `merge-as-is` rate hit zero. That carrier evidently shipped a cluster of PRs with real defects in that mid-window, which is a structurally interesting signal: not just "the reviewer found nothing easy" but specifically "the reviewer found things that needed pushback" on one carrier in particular.

### 2.3 `needs-discussion` is the steady noise floor

`needs-discussion` runs 0, 2, 0, 1, 1, 1, 2, 1, 0, 0, 1, 0, 1. Mean 0.77, present in 8 of 13 cycles. The two-discussion cycles are drip-301 and drip-306. Drip-301 has them across `sst/opencode` and `BerriAI/litellm`; drip-306 across `BerriAI/litellm` and `QwenLM/qwen-code`. There is no carrier-clustering pattern in `needs-discussion` the way there is in `request-changes`.

Operationally, `needs-discussion` is the "this PR is doing something architecturally that I cannot judge without talking to a human about intent" verdict. The fact that it sits at ~10% across the window means roughly one in ten PRs hits a design question that the reviewer cannot answer alone. That is a reasonable rate — high enough to be useful as a signal, low enough that the reviewer is not just punting.

## 3. Carrier-level read

Pivoting on carrier across all 13 cycles in the window:

- **sst/opencode** appears in every cycle, usually 2–3 PRs per cycle. Verdict mix is heavily weighted toward `merge-after-nits` with the three `request-changes` clustered in drip-306..308. This is the carrier with the most velocity and also the most variance in PR quality, which tracks with the project's pace.
- **openai/codex** appears in 11 of 13 cycles. Mix is mostly `merge-after-nits` with occasional `merge-as-is`. Almost no `request-changes` or `needs-discussion` — the codex PRs in this window are uniformly "well-formed but needing nits."
- **BerriAI/litellm** appears in 8 of 13 cycles. Mix has both `request-changes` (drip-303) and `needs-discussion` (drip-301, drip-306, drip-312 #27087 `df04a955ea553d7e023415aaf07f41314ae9cbd0`) — this carrier draws the most "I need to talk to a human" verdicts in the window.
- **charmbracelet/crush** appears in 8 of 13 cycles. One `needs-discussion` (drip-310 #2738 `bad0d43f2470be7067f6b534d3c120715a4e8c4f`). Mostly clean.
- **google-gemini/gemini-cli** appears in 9 of 13 cycles. One `request-changes` (drip-310 #26366) and otherwise mostly `merge-after-nits`. The `request-changes` verdict on #26366 is the only such verdict on this carrier in the window.
- **block/goose** appears in 7 of 13 cycles. Mix is balanced between `merge-as-is` and `merge-after-nits`. No `request-changes`. One of the cleaner carriers in this window.
- **QwenLM/qwen-code** appears in 6 of 13 cycles. One `merge-as-is` in drip-312 (#3810 `aa3b30904f01f1bd096816317754d83d8e249b22`). No `request-changes`.

Cross-carrier read: the `request-changes` verdict is **carrier-correlated**, the `needs-discussion` verdict is **PR-correlated** (it lands wherever the PR is doing something architecturally surprising, regardless of carrier), and the `merge-as-is` rate is the carrier's "quality floor" — block/goose and QwenLM/qwen-code shipped a higher fraction of as-is-mergeable PRs in this window than the others.

## 4. Verdict transitions: drip-N → drip-(N+1)

A useful lens is the verdict-mix transition: how does cycle N's mix relate to cycle (N+1)'s mix? If the reviewer's verdicts were independent draws from a stationary distribution, consecutive cycles should look roughly i.i.d. — no autocorrelation in the verdict counts.

The actual data shows mild positive autocorrelation in `merge-after-nits` (cycles with high after-nits tend to be followed by cycles with high after-nits) and mild *negative* autocorrelation in `merge-as-is` between adjacent cycles. The clearest example is drip-300 → drip-301 → drip-302: `merge-as-is` goes 1 → 1 → 4. That is a sharp upper-tail step in drip-302, and it coincides with the lowest `needs-discussion` count of the early window. Drip-302 was the "easy day" of the early sub-window.

The transition from drip-302's 4 `merge-as-is` down to drip-303's 1 `merge-as-is` is the largest single-cycle drop in the window. Drip-303 also adds the only `request-changes` of the early sub-window and one `needs-discussion` — call it the regression cycle. The fact that drip-303 has 7 carriers represented (the most of the early sub-window) plus the worst verdict mix of the early sub-window is consistent with a "carrier-diversity costs verdict-quality" effect: pulling more carriers into a single cycle pulls in PRs the reviewer is less prepared for, which raises the rate of nits and discussion verdicts.

## 5. What the head-SHA pinning lets you check

Each row in `INDEX.md` pins the head SHA at the time of review — that is the entire mechanism for detecting upstream force-pushes. Two examples worth flagging from the window:

- `sst/opencode #25584` appears in **both** drip-309 (`8f5ef02e44c5c142e36a4feced6b95fb2490ee16`) and drip-312 (`30bc36f6f8cccad34cc6ed24caed3b58cd33d19f`). Different head SHAs across the same PR number. That is exactly the pattern the SHA-pinning is designed to catch: the PR was re-reviewed in a later cycle because the head ref had moved.
- `sst/opencode #25602` appears in drip-311 (`57bc4f257065d5c03b1b9c4bc2abf4af99bbdade`). The post-merge SHA on the same PR — captured in the parallel oss-digest ADDENDUM-295 for 2026-05-03 — is `5fdb3f1c92c16cae0f1952e8fc8414488102b9f4`. So between drip-311's review and the merge event captured in ADD-295, the head was force-pushed and rebased onto a different commit. The drip-311 review is structurally still valid — same code change, different parent — but the SHA pin makes that drift visible after the fact rather than invisible.

These two examples demonstrate the pinning's actual utility: not as an authentication mechanism, but as a **time-travel** mechanism. You can ask "what code did the reviewer actually look at?" and get an answer rather than a hand-wave. For a daemon shipping 100+ reviews per day, that is the difference between an audit-able trail and a vibe-only trail.

## 6. The 68% `merge-after-nits` ceiling

Coming back to the dominant verdict: 68% of all verdicts across 13 cycles are `merge-after-nits`. That is the ceiling — when the reviewer policy is "default to the most-permissive verdict that still flags an issue", `merge-after-nits` absorbs everything that is neither flawless nor genuinely concerning. Three implications:

1. **The verdict resolution is asymmetric**. The reviewer can sharpen "merge-as-is" by being stricter (any nit, no matter how trivial, downgrades to `merge-after-nits`). The reviewer cannot sharpen `merge-after-nits` from above without inventing a new verdict like `merge-after-significant-fixes`.
2. **The signal-to-noise ratio is concentrated in the 32% non-after-nits tail**. If you are scanning the daemon output for actionable items, the `merge-as-is` rows tell you "these PRs are clean, look at them last", the `request-changes` rows tell you "this carrier shipped something broken", and the `needs-discussion` rows tell you "this PR has a design question worth debating". The `merge-after-nits` rows are largely background noise unless you are the PR author or a maintainer of that carrier.
3. **The baseline rate matters for trend-detection**. A jump from 5/8 to 7/8 `merge-after-nits` in a single cycle is within sampling noise. A jump from 5/8 to 0/8 (with 8/8 in some other category) would be a regime change. The high baseline means the trend signal is in the *minority* verdicts, not in the majority verdict.

## 7. What the next 13 cycles should look like

Three predictions for drip-313 onward, based on the 13-cycle baseline:

- **`merge-after-nits` will sit between 4 and 7 per cycle**, mean ~5.5. A cycle outside that band is worth investigating.
- **`request-changes` will average <1 per cycle** with carrier-clustering (when it appears, it tends to appear on the same carrier across nearby cycles). Watch for two-in-a-row `request-changes` on the same carrier — that is the early signal of a carrier-quality regression.
- **`needs-discussion` will average ~1 per cycle** with no obvious carrier pattern. A cycle with ≥3 `needs-discussion` would be a strong signal of an unusually architecture-heavy PR roster.

If any of those three baselines drift across the next 13-cycle window, the drift is the story. Until then, the read on the drip-300..312 window is: stable reviewer policy, stable verdict-mix distribution, one carrier-clustered quality dip in drip-306..308, no other regime changes worth flagging.

## References

- `oss-contributions/INDEX.md`, drip-300 through drip-312 entries (all dated 2026-05-03).
- Per-PR review files under `oss-contributions/reviews/drip-N/`.
- Head-SHA pinning examples: `sst/opencode #25584` (`8f5ef02e44c5c142e36a4feced6b95fb2490ee16` in drip-309 vs `30bc36f6f8cccad34cc6ed24caed3b58cd33d19f` in drip-312); `sst/opencode #25602` (`57bc4f257065d5c03b1b9c4bc2abf4af99bbdade` in drip-311 vs post-merge `5fdb3f1c92c16cae0f1952e8fc8414488102b9f4` from oss-digest ADDENDUM-295).
- Cross-reference: oss-digest `ADDENDUM-295.md` for the same-day merge-event capture window 2026-05-03T15:09:34Z → 15:48:00Z.
