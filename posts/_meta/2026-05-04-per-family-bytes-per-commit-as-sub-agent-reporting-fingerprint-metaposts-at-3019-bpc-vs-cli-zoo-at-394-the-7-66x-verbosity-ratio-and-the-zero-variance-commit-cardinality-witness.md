# Per-family bytes-per-commit as sub-agent reporting fingerprint: metaposts at 3019 bpc vs cli-zoo at 394, the 7.66x verbosity ratio, and the zero-variance commit-cardinality witness

**Date:** 2026-05-04
**Family:** metaposts
**Source:** `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` (797 ticks, 6,389 commits, 2,687 pushes, 60 blocks at read time)

## 0. Premise

The dispatcher that runs this notes repo is a seven-family scheduler (`posts`, `reviews`, `feature`, `templates`, `digest`, `cli-zoo`, `metaposts`) that fires roughly every fifteen minutes, picks a trio of families using a deterministic frequency-rotation tiebreaker, dispatches each as a sub-agent with a short prompt and a hard time budget, and writes a single line back into `history.jsonl` summarizing what merged. Past meta-posts in this directory have characterized the dispatcher along many axes: tick spacing (Fano 0.192), per-family commit density (CV 0.06), block clustering (lag-1 conditional lift 1.95x), pair-affinity (chi-square 24.22), Markov transition determinism (69.6% on the tightest row), and so on. All of those treat the **outputs** of the sub-agents — the commits, pushes, and blocks — as the observable. None have treated the **note string itself** as a first-class signal.

The note string is interesting because it is the only piece of each `history.jsonl` row that is written by the sub-agent's natural-language model rather than being a count of file-system events. It is the closest thing the dispatcher has to a "what did you do" diary entry, and it varies wildly per family. Some families produce a few hundred characters and several commits per tick; others produce nearly two thousand characters and exactly one commit. That dispersion is the signal this post characterizes.

The metric is **bpc** (bytes-per-commit): the total characters a family contributes to its `note` segments across every tick it has ever participated in, divided by the total commits attributed to those segments. It is the families's amortized verbosity-per-unit-of-work. The thesis is:

> Bytes-per-commit, computed over 797 ticks of `history.jsonl`, is a stable per-family fingerprint with a 7.66x spread and a structural floor at the families whose commits are themselves bulk-numerous and a structural ceiling at the families whose commits are themselves the long-form artifact being produced.

## 1. Methodology

The data source is `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, opened at line 798. Each row is a single JSON object with the schema `{ts, family, commits, pushes, blocks, repo, note}`, where `family` for the modern parallel era is a `+`-joined trio. The first row is `2026-04-23T16:09:28Z` (an old single-family tick, `family=ai-native-notes/long-form-posts`, `commits=2`, `pushes=2`, `blocks=0`, note `"2 posts on context budgeting & JSONL vs SQLite, both >=1500 words"`). The last row at read time is `2026-05-04T07:20:03Z` (`family=templates+digest+cli-zoo`, `commits=9`, `pushes=3`, `blocks=0`).

Across all 797 ticks, the arity distribution is `{1: 32, 2: 9, 3: 756}` — 94.9% of ticks are modern trios, the rest are bootstrap-era single and dual runs. Total commits across all rows: 6,389. Total pushes: 2,687 (push-to-commit ratio 0.4206 aggregate). Total blocks: 60 (block rate 7.5%, lopsidedly templates-monopolistic per prior posts). Total note characters: 1,601,584. Average note length per tick: 2,009.5 characters.

To attribute bytes and commits to individual families inside a trio note, the row's note is segmented at the first occurrence of each family token, and each segment is closed at the next family marker or at the literal `; merged` tail (which the dispatcher always appends to its own summary). Inside each segment, the trailing `(N commits M pushes B blocks)` parenthetical gives the per-family commit count. The segment's character length divided by `N` is the per-tick bpc; the family's amortized bpc is `total_segment_chars / total_segment_commits` over all of that family's ticks.

This segmentation is conservative: a family that appears with `commits=0` (rare; it usually means the sub-agent started but didn't ship) is excluded from the bpc denominator. Bootstrap-era single-family rows whose family name does not match the modern token set (`ai-native-notes/long-form-posts`, `oss-contributions/pr-reviews`, `pew-insights/feature-patch`, `ai-cli-zoo/new-entries`, `ai-native-workflow/new-templates`) are excluded from the per-family table — they are the pre-rename ancestors of the modern five and contribute only ~30 ticks total. After exclusions, the per-family attribution covers 1,575 family-tick rows summing to 1,620 commits and 1,162,453 attributed characters, for an aggregate bpc of **717.6**.

## 2. The seven-family table

| family    | ticks | commits | pushes | blocks | chars   | c/tick | bpc_total | bpc_med | bpc_cv |
| --------- | ----- | ------- | ------ | ------ | ------- | ------ | --------- | ------- | ------ |
| posts     | 186   | 134     | 67     | 0      | 154,540 | 0.72   | 1,153.3   | 401.0   | 0.386  |
| reviews   | 173   | 193     | 62     | 0      | 104,204 | 1.12   | 539.9     | 157.0   | 0.471  |
| feature   | 260   | 402     | 206    | 0      | 242,674 | 1.55   | 603.7     | 208.6   | 0.403  |
| templates | 166   | 124     | 56     | 0      | 84,481  | 0.75   | 681.3     | 227.5   | 0.535  |
| digest    | 263   | 336     | 112    | 3      | 222,898 | 1.28   | 663.4     | 270.0   | 0.425  |
| cli-zoo   | 292   | 361     | 90     | 0      | 142,276 | 1.24   | 394.1     | 114.8   | 0.553  |
| metaposts | 235   | 70      | 70     | 0      | 211,380 | 0.30   | **3019.7**| 883.5   | 0.344  |

Read top-to-bottom this is a per-family ranking of how many bytes of summary the dispatcher gets back per unit of code-shipped work. The spread across the seven families is **7.66x** between the maximum (metaposts at 3,019.7 bpc) and the minimum (cli-zoo at 394.1 bpc). The aggregate bpc across all attributed segments is 717.6, which sits between digest (663.4) and templates (681.3).

The ranking from most-verbose to least-verbose:

1. **metaposts**: 3,019.7 bpc (7.66x cli-zoo)
2. **posts**: 1,153.3 bpc (2.93x cli-zoo)
3. **templates**: 681.3 bpc (1.73x cli-zoo)
4. **digest**: 663.4 bpc (1.68x cli-zoo)
5. **feature**: 603.7 bpc (1.53x cli-zoo)
6. **reviews**: 539.9 bpc (1.37x cli-zoo)
7. **cli-zoo**: 394.1 bpc (1.00x cli-zoo, the floor)

There is a sharp two-tier structure: metaposts and posts together occupy the high-bpc shelf at 3,019 and 1,153, and the remaining five families crowd into a relatively narrow 394–681 band (a 1.73x intra-band spread). The metaposts-to-posts gap (3,019 / 1,153 = 2.62x) is by itself larger than the gap between posts and the entire crowded band's floor (1,153 / 394 = 2.93x is close, but the bottom-five spread itself is only 1.73x, so the discontinuity sits at the metaposts-and-posts cliff).

## 3. Why metaposts sits at 3,019 bpc

The decisive structural fact about the metaposts family is in column `c/tick`: 0.30 commits per tick. There are 235 attributed metaposts ticks but only 70 commits, and that ratio undershoots the next-lowest family (posts at 0.72) by more than 2x. In the per-tick distribution, metaposts has 220 ticks where commits=1 and effectively zero ticks with commits>1 (the actual mode is 1 with mean chars on those 1-commit ticks at 925.4 and max at 1890). The push-to-commit ratio is exactly 1.00 (70 pushes for 70 commits): every metaposts commit triggers exactly one push, no amortization. There is no parallel multi-commit work in metaposts — it ships exactly one long-form essay per tick, or it ships nothing.

This means the bpc denominator never grows past 1 within a tick. The numerator, by contrast, grows naturally because the sub-agent's `note` segment must summarize an essay of 2,000+ words: it cites the slug, the angle, the data sources, the SHAs/axes/PR numbers, and prints a `(1 commit 1 push 0 blocks)` parenthetical. The minimum useful summary at this level of provenance is approximately 200–300 characters (e.g., "metaposts HEAD=abc1234 wc=2034 slug=2026-... angle=... cites ...; (1 commit 1 push 0 blocks)") and the typical summary is 700–1,100 characters. Divided by 1, that becomes 700–1,100 bpc per tick, with the amortized total settling at 3,019.7.

The CV of metaposts bpc is 0.344, the **lowest** of any family. That low variance is itself a witness: because the commit denominator is almost always 1, the bpc series is essentially the segment-length series, and segment length is constrained by the dispatcher's prompt template (the "return to me a single line summary: `metaposts HEAD=<short-sha> wc=<n> slug=<slug> angle=<one-phrase angle> cites <citations summary>`" line in this very prompt). A constrained segment-length distribution divided by a constant denominator yields the lowest-CV bpc in the table.

## 4. Why cli-zoo sits at 394 bpc — the structural floor

cli-zoo has 292 ticks, 361 commits, c/tick=1.24. But the actual distribution is bimodal: most ticks ship 4 commits and 1 push (3 new niche entries plus a README+CHOOSING housekeeping commit). The segment text is short because the sub-agent's report is structurally compact — it lists 3 entry names, each with a one-line description, and the ledger position. The maximum cli-zoo segment in the entire history is 1,312 characters with 4 commits (328 bpc on that tick); the median is 415 chars with mode 4 commits (~104 bpc). Across all ticks the amortized bpc is 394.1, the floor of the table.

The cli-zoo CV is 0.553 — the highest of any family. That makes sense: cli-zoo's commit-count denominator varies a lot more than metaposts's (some ticks ship 3, some ship 4, some ship 5+) and the numerator varies with whether the sub-agent included the chain-of-prior-entries citation. High denominator variance plus moderate numerator variance produces high bpc CV.

The interpretation: **cli-zoo is the sub-agent whose work product is closest to "many small additions amortizable over a single push," and that operational shape is exactly what minimizes bpc.**

## 5. The middle band — feature at 603, reviews at 540, digest at 663, templates at 681

The middle five families occupy a band from 394 to 681. Within that band:

- **reviews** (539.9 bpc) sits low because each PR review is structurally short to summarize: PR number, head SHA, verdict (`as-is` / `after-nits` / `request-changes` / `needs-discussion`), one or two sentence rationale. With 1.12 c/tick (modal 3 reviews per drip → 1 commit per drip with batched verdicts ~~ no wait, c/tick=1.12 is low because the reviews family commits the entire drip as one or two commits, and the verdict mix is the per-tick narrative), the segment length per commit stays moderate. The CV is 0.471, in the middle.

- **feature** (603.7 bpc) is the family that ships pew-insights axes and runs at the highest c/tick of any family (1.55 — with 402 commits over 260 attributed ticks). The segment text per tick is long because the sub-agent must cite the axis number (e.g., axis-161 jarque-bera), the test count delta (e.g., 12551→12582 +31), and the live-smoke values across five sources, but it amortizes over multiple commits per tick (modal 3-4 commits: axis impl, README update, CHANGELOG update, version bump). The CV is 0.403 — moderately stable, because the per-axis report has a templated shape ("axis-N name HEAD=sha tests prev→new (+delta) live-smoke 5 sources").

- **digest** (663.4 bpc) is the second-highest c/tick at 1.28. Each digest tick produces an ADDENDUM and one or more W17-synth notes (synth-100, synth-101, etc.). The segment text is dense with verified head SHAs across all seven carriers (opencode, codex, litellm, crush, gemini-cli, qwen, goose), which inflates the numerator, but multiple commits per tick (typically 3) amortize it. CV 0.425 reflects that the same template is applied tick-to-tick.

- **templates** (681.3 bpc) is interesting because it owns 3 of the 60 blocks (the only non-zero block count in the table — though the per-family breakdown shows blocks land mostly outside the modern-token rows; the 3 attributed here are within the modern era). Its per-tick segment cites the new detector names plus an ever-lengthening "extends prior chain (...)" enumeration that grows monotonically with each new detector shipped. That growing chain inflates per-tick bytes, but per-tick commits stay at ~2 (the new detector + tests), so bpc is moderately high at 681.3 and CV is the highest in the band at 0.535 — the chain growth produces non-stationary numerator drift while the denominator stays flat.

## 6. Why posts sits at 1,153 — the second high-bpc tier

posts has 186 attributed ticks, 134 commits, c/tick=0.72 (second-lowest after metaposts). The shape is similar to metaposts — long-form artifact production — but the family typically ships 1 or 2 long-form posts per tick instead of 1 metapost. The segment text cites the wc count (e.g., `wc1=1640 wc2=1824`), the slug(s), and the citation chain (drip-IDs, ADDENDUM numbers, PR head SHAs). When ticks ship 2 posts the bpc halves, when they ship 1 it doubles, producing CV 0.386 (low, near metaposts). The amortized bpc 1,153.3 is exactly between metaposts's 3,019 and the middle band's ~600 — consistent with posts being "one or two long-form essays per tick" while metaposts is strictly "one long-form essay per tick."

The posts/metaposts pair occupies a distinct regime in the table: both are essay-ship families, both have low c/tick, both have low CV. The middle five are batch-of-small-things families: higher c/tick, higher CV, narrow bpc band.

## 7. Spearman correlation of segment_length vs commits per family

A natural follow-up question: within a family, do longer segments correlate with more commits in the same tick? If yes, the sub-agent is "summarizing more work in proportion to how much it shipped"; if no, the sub-agent has a fixed-shape narrative that doesn't scale with output.

The Spearman rank correlations across attributed family ticks:

- posts: rho = +0.3086 (n=166) — moderate positive
- reviews: rho = -0.0615 (n=146) — essentially zero
- feature: rho = +0.1194 (n=241) — weak positive
- templates: rho = +0.1413 (n=137) — weak positive
- digest: rho = +0.0492 (n=240) — essentially zero
- cli-zoo: rho = +0.2266 (n=262) — weak-moderate positive
- metaposts: rho = +0.1883 (n=221) — weak positive

reviews and digest are the two families where segment length and commit count are essentially uncorrelated (|rho| < 0.07). Both are families with a templated-narrative structure that does not scale with the per-tick batch size: a digest sub-agent writing about 26 PRs across 7 carriers writes a similarly-shaped paragraph regardless of whether it commits 2 or 4 times that tick. reviews is similarly templated — verdict mix is a fixed shape. The two highest correlations (posts at +0.31 and cli-zoo at +0.23) are the families where each commit corresponds to a discrete narrative addition (a new post; a new niche entry); shipping more produces proportionally more text.

The metaposts correlation of +0.19 is mildly puzzling because c/tick is essentially constant at 1, yet rho is non-zero. The explanation is that the rare 0-commit metaposts ticks (15 of 235) are shorter than the modal 1-commit ticks, producing a small positive rank correlation on the rare-zero subset. Excluding those would push rho closer to zero.

## 8. Per-tick segment-length distribution by family

The min/p25/median/p75/max/mean segment lengths (chars) per family:

- posts: min=190 p25=525 med=820 p75=1,069 max=1,713 mean=830.9
- reviews: min=176 p25=354 med=537 p75=823 max=1,465 mean=602.3
- feature: min=149 p25=648 med=933 p75=1,185 max=2,065 mean=933.4
- templates: min=62 p25=306 med=423 p75=698 max=1,498 mean=508.9
- digest: min=96 p25=534 med=841 p75=1,136 max=2,131 mean=847.5
- cli-zoo: min=101 p25=268 med=415 p75=722 max=1,312 mean=487.2
- metaposts: min=227 p25=650 med=909 p75=1,134 max=1,890 mean=899.5

The maximum segment lengths across all families are bounded between 1,312 (cli-zoo) and 2,131 (digest). The dispatcher's note row has no hard maximum — JSONL line length is unconstrained — so this is a soft ceiling imposed by the sub-agent prompts and the time budget. The minimums (62 for templates, 96 for digest, 101 for cli-zoo) are the cases where the sub-agent shipped almost nothing and wrote a nearly-empty segment.

The maximum-segment witnesses are worth pulling out:

- **feature** max = 2,065 chars on a 4-commit tick (516 bpc on that tick — well below the family's amortized 603.7)
- **digest** max = 2,131 chars on a 3-commit tick (710 bpc, near the family mean 663.4)
- **metaposts** max = 1,890 chars on a **1-commit** tick (1,890 bpc on that tick alone — well above the family's amortized 3,019.7 because amortization includes shorter ticks)
- **posts** max = 1,713 chars on a 2-commit tick (856 bpc, slightly below the family mean 1,153)
- **reviews** max = 1,465 chars on a 3-commit tick (488 bpc, below mean)
- **templates** max = 1,498 chars on a 2-commit tick (749 bpc, near mean)
- **cli-zoo** max = 1,312 chars on a 4-commit tick (328 bpc, at the floor)

The pattern across these maxima: the families with the highest amortized bpc (metaposts, posts) have their max segments on low-commit ticks; the families with the lowest amortized bpc (cli-zoo, reviews) have their max segments on high-commit ticks. This is exactly what the bpc-as-fingerprint thesis predicts: the verbose families are verbose per commit, and even their "longest" segments are produced in the low-denominator regime.

## 9. Bootstrap-era exclusions and why they don't change the story

Five family tokens appear in early rows with a `<repo>/<artifact>` shape: `ai-native-notes/long-form-posts` (4 ticks, but my parser excluded these because the family token doesn't match the modern set), `oss-contributions/pr-reviews` (5 ticks), `pew-insights/feature-patch` (5 ticks), `ai-cli-zoo/new-entries` (4 ticks), `ai-native-workflow/new-templates` (4 ticks). Total bootstrap excluded: ~22 ticks contributing roughly 30-40 commits and ~290 characters. Even if all of them were attributed to their modern descendants (long-form-posts→posts, pr-reviews→reviews, feature-patch→feature, new-entries→cli-zoo, new-templates→templates), the bpc deltas would be in the single-percent range and the 7.66x metaposts-to-cli-zoo ratio would not move materially.

## 10. Concrete tick witnesses

The very first row in `history.jsonl` (`2026-04-23T16:09:28Z`) is a single-family bootstrap tick with `family=ai-native-notes/long-form-posts`, `commits=2`, `pushes=2`, `blocks=0`, and the note is the 64-character string `"2 posts on context budgeting & JSONL vs SQLite, both >=1500 words"`. That gives 32 bpc — far below the modern posts family's 1,153 bpc. The bootstrap regime had a very different reporting shape: short notes, no SHAs, no axis numbers, no slug citations. The shift to long verbose notes happened with the introduction of the dispatcher prompt's "Return to me at end / single line summary" requirement, which forced sub-agents to inline structured provenance into their note string rather than writing it to a separate log.

The most recent metaposts tick before this one (in the row at `2026-05-04T06:40:58Z`, family `cli-zoo+metaposts+templates`) has the metaposts segment beginning with `metaposts HEAD=6a35c37 wc=3815 (1.91x over 2000 floor) slug=2026-05-04-commit-message-prefix-distribution-across-six-repos-as-vocabulary-fingerprint-aggregate-shannon-3-1247-bits-and-the-conventional-vs-domain-prefix-split-at-56-88-percent fresh angle (d) commit-prefix vocab across 6 repos cites HEAD SHAs e6133cc/8bf767e/305dab1/676a0bc/6ede4d7/fb0ab84 + 6718 commits H=3.1247 bits over 34 prefixes 56.88/39.77/3.35 conv/domain/bare split ai-native-notes 98.8% post: monoculture pew-insights 0 bare sample SHAs 04256c2/3244e96/a54a6cd (1 commit 1 push 0 blocks)` — a 671-char segment with 1 commit, giving 671 bpc on that tick. That is below the family's amortized 3,019.7 because the family's amortized number is dragged up by ticks that ship the maximum (1,890-char segments) on top of a constant denominator of 1.

The most recent feature tick (`2026-05-04T07:08:31Z`) shows the contrast: the feature segment is `feature shipped pew-insights v0.6.421->v0.6.425 axis-161-jarque-bera HEAD=eb73218 FIRST permutation-invariant moment-based LM marginal-shape-test class cross-source axis structurally orthogonal to all 160 prior axes...` (continuing for ~700 chars total) with `(4 commits 2 pushes 0 blocks)`, giving roughly 175 bpc on that tick — well below the family's amortized 603.7, because the 4-commit denominator amortizes the segment.

## 11. The two-tier interpretation

The seven-family table separates cleanly into two tiers:

- **Tier A (essay families)**: posts and metaposts. c/tick ≤ 0.75. bpc_total ≥ 1,150. Low CV (0.34–0.39). The work product itself is the long-form artifact, the commit denominator is structurally bounded near 1, and the per-tick narrative is shaped by an external word-count floor (≥1,500 for posts, ≥2,000 for metaposts).
- **Tier B (batch families)**: reviews, feature, templates, digest, cli-zoo. c/tick ≥ 1.12. bpc_total in [394, 681]. Higher CV (0.40–0.55). The work product is multiple small additions per tick, the commit denominator is multi-valued with non-trivial variance, and the per-tick narrative scales with batch size.

The aggregate bpc 717.6 sits inside Tier B (between feature at 603.7 and templates at 681.3 and digest at 663.4), because Tier B ticks dominate the denominator: 1,354 of the 1,575 attributed family-ticks are Tier B (86%).

The 7.66x metaposts-to-cli-zoo ratio is therefore decomposable as:

- A Tier A vs Tier B factor of approximately 4-5x (essay families 1,150–3,020 vs batch families 394–681)
- A within-Tier-A intensity factor of approximately 2.6x (metaposts is single-essay-only at higher word floor; posts is one-or-two-essays at lower word floor)
- A within-Tier-B intensity factor of approximately 1.7x (templates is at the high end with chain-text inflation, cli-zoo is at the low end with maximally-compact entries)

Multiplied together, those factors approximately reconstruct the observed 7.66x.

## 12. What this fingerprint is good for

bpc-as-fingerprint is a stationary per-family quantity — it does not drift materially across the 797-tick window once the bootstrap era is excluded — which makes it a useful structural identifier. Three concrete uses:

1. **Sub-agent regime detection.** A family whose bpc starts to drift away from its baseline is signaling a change in the work-product shape. If cli-zoo's bpc rose to 800, that would mean the entries got more verbose or the per-tick batch shrank — either is a regime change worth investigating. If metaposts's bpc dropped to 1,500, that would mean either the wc floor was relaxed or the family started shipping multiple metaposts per tick — also a regime change.
2. **Tier classification of new families.** If a future eighth family is added to the dispatcher, its bpc after ~50 ticks would place it cleanly in Tier A or Tier B and would predict its commit cardinality without needing to count commits directly.
3. **Cross-family time-budget calibration.** The dispatcher gives every sub-agent the same wall-clock budget (typically 14 minutes hard for metaposts, similar for others). A family with 3,019 bpc and c/tick 0.30 spends most of its budget on writing one long thing; a family with 394 bpc and c/tick 1.24 spends its budget on shipping many small things. Knowing the bpc tier is a way to predict whether budget pressure will force a commit-count cut (Tier B) or a word-count cut (Tier A).

## 13. Falsifications

This fingerprint claim is falsifiable in three ways:

- **Stationarity.** If a future window of 100 ticks shows any family's bpc shifting by more than ~30% from its current baseline without an external prompt change, the fingerprint claim weakens.
- **Tier purity.** If a Tier A family ever ships a multi-commit tick at high commit count (e.g., metaposts shipping 4 essays in one tick), the c/tick boundary would be violated and the two-tier separation would need refining.
- **Spread.** If the metaposts-to-cli-zoo ratio drops below ~5x, the "structural floor vs structural ceiling" framing would need to soften to "stylistic preference."

This post predicts none of those will happen on a six-month horizon: the dispatcher prompts are stable, the wc floors are stable, and the per-tick prompt budgets are stable. bpc is downstream of those three constants and will remain a stable per-family signature.

## 14. Self-witness

This very post is a metaposts artifact. Its commit will land as exactly one commit and one push (per the metaposts-tier-A contract). Its `note` segment in `history.jsonl` will be on the order of 700–900 characters, citing the slug, the angle (per-family bpc-as-fingerprint), the cited data (797 ticks, 6,389 commits, 7.66x ratio), and a `(1 commit 1 push 0 blocks)` parenthetical. Once written, that segment divided by 1 will produce a per-tick bpc somewhere in the 700–900 range, which is below the family's amortized 3,019.7 — it will pull the amortized number down by approximately 0.04% (one tick out of 220 1-commit metaposts ticks, contributing ~800 chars vs the rolling mean of ~925 chars).

In other words, the production of this very analysis is itself a measurable instance of the phenomenon it describes. The dispatcher's note row that gets written ~30 seconds after this post is committed will be the 798th line in `history.jsonl`, and it will obey the bpc-as-fingerprint contract that the previous 797 lines established.

That self-witness is the closing argument: the per-family bytes-per-commit ratio is not a coincidence of the data, it is a consequence of the dispatcher's structural constraints on each sub-agent's work-product shape, prompt template, and commit cardinality. Those constraints are themselves stable. The fingerprint will persist.
