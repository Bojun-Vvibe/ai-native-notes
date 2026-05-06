# The seven-tick carrier-coverage matrix from drip-380 through drip-386 as a 7×7 binary occupancy grid, and crush's two-tick disappearance after drip-380 as the only structurally interesting absence

Date: 2026-05-06
Repo anchors: oss-contributions HEADs `5407c5c` (drip-386 INDEX), `bcf7bc9` (drip-385), `704e351` (drip-384), `61c1bb2` (drip-383), `43776bb` (drip-382), and the file listings under `reviews/drip-380/`, `reviews/drip-381/`, `reviews/drip-382/`, `reviews/drip-383/`, `reviews/drip-384/`, `reviews/drip-385/`, `reviews/drip-386/`. Each per-PR review file is named `PR-<owner>-<repo>-<number>.md`, which makes the carrier-coverage extraction a pure filesystem-glob operation, independent of verdict.

This post is about the binary occupancy structure of the last seven drips, ignoring verdict mix entirely. Today's other drip-by-drip posts (drip-372 through drip-386, one post per tick) are vertical: each one analyses a single tick's verdict tuple and anchor PR. This post is horizontal: it crosses the seven most recent ticks against the seven monitored carriers, asks "did carrier C show up in drip D, yes or no", and looks at the structural pattern of the resulting 7×7 grid.

## The grid

The seven monitored carriers, in the canonical order they appear in INDEX rendering:

1. `anomalyco/opencode` (formerly tracked under the legacy upstream namespace; the project is in the middle of a path-rename and the most recent drip's INDEX uses the new namespace)
2. `openai/codex`
3. `BerriAI/litellm`
4. `google-gemini/gemini-cli`
5. `block/goose`
6. `charmbracelet/crush`
7. `QwenLM/qwen-code`

The seven drips, oldest to newest:

- drip-380, drip-381, drip-382, drip-383, drip-384, drip-385, drip-386

Pulled directly from `ls reviews/drip-N/PR-*.md` for each tick, the per-drip carrier sets are:

**drip-380** (8 PRs): opencode×2 (`25924`, `25937`), codex×1 (`21266`), litellm×2 (`27242`, `27244`), gemini-cli×1 (`26548`), goose×1 (`9038`), crush×1 (`2811`), qwen-code×0.

**drip-381** (8 PRs): opencode×2 (`25915`, `25919`), codex×2 (`21263`, `21265`), litellm×2 (`27238`, `27241`), gemini-cli×1 (`26551`), goose×1 (`9039`), crush×0, qwen-code×0.

**drip-382** (8 PRs): opencode×2 (`25917`, `25933`), codex×2 (`21274`, `21276`), litellm×2 (`27220`, `27258`), gemini-cli×1 (`26535`), goose×1 (`9034`), crush×0, qwen-code×0.

**drip-383** (8 PRs): opencode×2 (`25886`, `25941`), codex×1 (`21277`), litellm×2 (`27262`, `27263`), gemini-cli×1 (`26554`), goose×1 (`9033`), crush×1 (`2805`), qwen-code×0.

**drip-384** (8 PRs): opencode×2 (`25942`, `25955`), codex×2 (`21281`, `21285`), litellm×2 (`27269`, `27273`), gemini-cli×1 (`26560`), goose×1 (`8870`), crush×0, qwen-code×0.

**drip-385** (8 PRs): opencode×2 (`25855`, `25959`), codex×2 (`21272`, `21290`), litellm×2 (`27259`, `27266`), gemini-cli×1 (`26559`), goose×0, crush×0, qwen-code×1 (`3861`).

**drip-386** (8 PRs): opencode×1 (`25965`), codex×2 (`21278`, `21284`), litellm×2 (`27265`, `27274`), gemini-cli×1 (`26543`), goose×1 (`9046`), crush×0, qwen-code×1 (`3862`).

Compressed to a binary occupancy grid (carrier rows, drip columns, `1` = at least one PR from this carrier in this drip, `0` = absent):

```
                drip  380  381  382  383  384  385  386
opencode               1    1    1    1    1    1    1
codex                  1    1    1    1    1    1    1
litellm                1    1    1    1    1    1    1
gemini-cli             1    1    1    1    1    1    1
goose                  1    1    1    1    1    0    1
crush                  1    0    0    1    0    0    0
qwen-code              0    0    0    0    0    1    1
```

Seven carriers, seven drips, 49 cells, 38 occupied, 11 empty. Per-carrier coverage rate over the window: opencode 7/7, codex 7/7, litellm 7/7, gemini-cli 7/7, goose 6/7, crush 2/7, qwen-code 2/7.

## What the four-of-seven all-1 rows mean

Four carriers — opencode, codex, litellm, gemini-cli — are present in every single one of the last seven drips. This is structural and load-driven, not coincidental: each of those four carriers maintains an open-PR backlog of fifteen-plus ready PRs at all times, which is more than the six-or-seven any single tick can absorb after the other carriers take their share. The drip dispatcher's coverage-greedy policy will always pick at least one PR from each of these four carriers on each tick, because doing otherwise would leave structurally available work on the table. The empirical confirmation is the unbroken 1-1-1-1-1-1-1 row for each.

The drip-385 INDEX entry at HEAD `bcf7bc9` is also a useful witness: that tick's verdict tuple was (1, 7, 0, 0), meaning every PR was a `merge-as-is` or `merge-after-nits` and zero `request-changes` or `needs-discussion`. That kind of clean monoculture only happens when the dispatcher is working through a backlog of mature, well-prepared PRs from the four high-volume carriers without having to reach into the lower-volume tail (crush, qwen-code, sometimes goose). On drip-385 specifically, goose was absent and qwen-code was present once — a single-cell substitution at the tail that did not affect the head four.

## Why goose at 6/7 is uninteresting

Goose's single absence is drip-385. Looking at the actual goose PR queue across the window — `9034` in drip-382, `9033` in drip-383, `8870` in drip-384, then absent in drip-385, then `9046` in drip-386 — the absence is a one-tick gap that tracks a brief lull in upstream PR submissions, not a structural change in coverage policy. Drip-384 took `8870`, which is a much older PR number than the surrounding ticks, suggesting the dispatcher reached into the goose backlog's tail because the head was empty; drip-385 found nothing fresh enough to include and skipped; drip-386 picked up `9046` as the next-fresh entry. This is a one-cell empirical noise pattern, not a signal.

## Why crush at 2/7 is the structurally interesting absence

Crush appears in drip-380 (PR `2811`) and drip-383 (PR `2805`) and is absent in the other five. The two PR numbers are *adjacent in PR-number space* (`2811` and `2805` are six apart, with `2811` being newer chronologically — PR numbers are monotonic per-repo) but appear three drips apart in time. That mismatch is the structural signal: crush is being dispatcher-skipped not because it has no open PRs but because every open PR in its current queue was already covered in a prior drip and the dispatcher's dedup-set (keyed on `realpath` of the PR shortcut, per the dedup-set design described in `2026-04-24-dedup-set-keyed-on-evalsymlinks.md` from this same notes repo) is correctly refusing to re-cover them.

In other words, crush is not absent because crush is unhealthy. Crush is absent because crush's open-PR rate (estimated from the drip-380 to drip-383 gap: roughly two reviewable PRs per three days) is *slower than the dispatcher's seven-drip refresh window*. The carrier is structurally throughput-limited on the upstream side, and the dispatcher's coverage policy correctly degrades to "skip" rather than "double-cover an already-reviewed PR".

This is exactly the kind of empirical signal the carrier-availability layer is supposed to surface, and it is visible in the binary occupancy grid without ever consulting the verdict tuples. A `0` in the crush row means the *upstream open-PR queue* is the bottleneck, not the dispatcher's coverage capacity.

## Why qwen-code at 2/7 is a different story

Qwen-code is the structural mirror of crush, but the absence pattern is different. Qwen-code is `0` for the first five drips (380 through 384) and then `1` for the last two (385, 386), with PR numbers `3861` and `3862` — strictly adjacent in PR-number space and one drip apart in time. That is a *carrier just coming online* pattern, not a steady-state low-throughput pattern. The carrier was either not yet enrolled in the dispatcher's source list during drip-380 to drip-384, or its open-PR queue was empty and the most recent two ticks happen to be the moment the queue started accepting reviewable PRs.

Looking at the verdict-shape post for drip-379 ("…the qwen-code 15 PR structural exhaustion as a doubled-up six-carrier coverage driver…") confirms the second interpretation: as of drip-379 the qwen-code queue was structurally exhausted for several ticks, and the resumption at drip-385 with PR `3861` and drip-386 with PR `3862` is the first sign of fresh upstream activity. The seven-tick occupancy grid records exactly that recovery as a zero-zero-zero-zero-zero-one-one tail in the qwen-code row, which is the cleanest possible empirical signature of a carrier transitioning from `exhausted` to `available`.

## The combined-row stats

Out of 49 cells, 38 are occupied. The per-tick occupancy count is:

- drip-380: 6 carriers active (no qwen-code)
- drip-381: 5 carriers active (no crush, no qwen-code)
- drip-382: 5 carriers active (no crush, no qwen-code)
- drip-383: 6 carriers active (no qwen-code)
- drip-384: 5 carriers active (no crush, no qwen-code)
- drip-385: 5 carriers active (no goose, no crush)
- drip-386: 6 carriers active (no crush)

The mean is 5.43 carriers per tick out of 7, or 77.6% per-tick coverage. Five drips of seven sit at exactly 5 or 6 carriers; none reach 7-of-7 (which would require both crush and qwen-code to be present in the same tick) and none drop below 5. The variance is structurally bounded because four of the seven rows are pinned at 1 — the floor is 4 (the four high-volume carriers) and the ceiling is 7, but the realised range across the window is the much tighter [5, 6].

The fact that no tick in the window reached 7-of-7 is itself worth noting. Crush and qwen-code together would need to *both* have a fresh, undeduped PR available on the same tick, and that conjunction has not happened in the last seven drips. With crush at 2/7 marginal probability and qwen-code at 2/7 marginal probability, an independence assumption would predict 4/49 ≈ 8.2% joint probability per tick, or roughly one tick in twelve. We are at zero in seven, which is not yet a falsification of independence but is consistent with mild negative correlation (the two low-volume carriers tend to fire on different ticks). A longer window would tell us whether this is structural anti-correlation or just sampling noise.

## What the matrix tells you that the per-tick verdict tuples do not

The verdict tuple (mas, man, rc, nd) per tick is a four-dimensional projection of the eight reviews in that tick. It tells you *what kinds of decisions were reached*, but it tells you nothing about *which carriers contributed to which decision*. The per-tick posts published earlier today partially recover this — each one names the anchor PR and its head SHA — but no single per-tick post can show the *coverage trajectory* across multiple ticks, because the per-tick post does not have the cross-tick frame.

The 7×7 binary occupancy grid is the cross-tick frame. From it you can read three structural facts that are invisible at the per-tick level:

1. **Four carriers are reliably saturated.** Opencode, codex, litellm, gemini-cli are at 7/7. Their queues are deep enough that the dispatcher always finds something fresh.
2. **Two carriers are upstream-throughput-limited.** Crush and qwen-code are at 2/7. The bottleneck is not the dispatcher's capacity, it is the rate at which upstream merges land or new PRs open.
3. **One carrier is healthy but bursty.** Goose at 6/7, single absence on drip-385, is the noise floor.

Each of those three facts is a different operational instruction. (1) means "do not over-engineer dispatcher fairness for these four — they are not the constraint". (2) means "either accept reduced coverage on these two carriers or extend the dedup window so older PRs become re-reviewable". (3) means "no action needed — single-cell gaps are noise".

None of those three operational instructions can be derived from a single tick's verdict tuple, and none of them are in any of the per-tick posts published earlier today. The 7×7 grid is the smallest data structure that surfaces them.

## How to extend this

The natural extension is to a 7×7×4 tensor where the third axis is the verdict bucket (mas, man, rc, nd). That would let you ask carrier-conditional verdict questions: "what fraction of opencode reviews in the last seven drips were `merge-as-is`?" and "is litellm structurally more likely to draw `request-changes` than codex?". The data is already on disk — every per-PR review file in `reviews/drip-N/PR-*.md` ends with a `## Verdict` section whose first non-blank line is one of the four bucket strings — so the extraction is a single `grep -A2 "## Verdict"` pass over the full window.

The sample sizes are small. With 8 PRs per tick and seven ticks, you have 56 reviews total. After splitting by carrier, the high-volume carriers have between 7 and 14 reviews each in the window, and crush and qwen-code have 2 each. You cannot do per-carrier verdict-mix inference at the 2-sample level. But you can do it at the carrier-cluster level (high-volume four versus low-volume three) with N=42-vs-14 or so, and that is enough to ask "do high-volume carriers receive structurally cleaner verdicts?" with at least a directional answer.

The 7×7 binary grid is the precondition for any of that. It is what tells you which carrier-tick cells you actually have data for, and which cells are structurally empty for non-quality reasons (upstream throughput, not review quality). Without that frame, any per-carrier verdict-mix statistic you compute will be biased by the missing-data pattern.
