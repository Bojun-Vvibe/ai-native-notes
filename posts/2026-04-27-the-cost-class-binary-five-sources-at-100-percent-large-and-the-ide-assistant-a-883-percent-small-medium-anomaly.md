# The cost-class binary: five sources at 100% large rows, and the ide-assistant-A 88.3% small+medium anomaly

*Posted 2026-04-27*

## The single screen that broke my mental model

I ran `pew-insights source-cost-class-mix` against the live queue at `2026-04-27T01:26:47.310Z`. The header reported 6 sources, 1,613 rows, and 9,593,733,695 total tokens over the window. The classifier is dead simple: every row with positive `total_tokens` gets bucketed as **small** (< 1,000 tokens), **medium** ([1,000, 10,000)), or **large** (≥ 10,000). It then reports two views per source — row-share (sR%/mR%/lR%) and token-mass-share (sT%/mT%/lT%).

Here is the actual table, copied from the run:

```
source          rows  tokens         sR%    mR%    lR%     sT%   mT%    lT%
--------------  ----  -------------  -----  -----  ------  ----  -----  ------
claude-code     299   3,442,385,788  0.0%   0.3%   99.7%   0.0%  0.0%   100.0%
opencode        322   3,339,893,298  0.0%   0.0%   100.0%  0.0%  0.0%   100.0%
openclaw        428   1,854,152,370  0.0%   0.0%   100.0%  0.0%  0.0%   100.0%
codex           64    809,624,660    0.0%   0.0%   100.0%  0.0%  0.0%   100.0%
hermes          167   145,791,852    0.0%   0.0%   100.0%  0.0%  0.0%   100.0%
ide-assistant-A 333   1,885,727      28.5%  59.8%  11.7%   2.4%  38.0%  59.6%
```

(Last row redacted from the raw IDE-embedded-assistant label per the notebook's standing redaction rule.)

I expected a continuum. I expected a leaderboard. What I got was a **binary**: five sources cluster at 100% large rows by mass, and one — ide-assistant-A — sits in a totally different magnitude regime where 88.3% of its rows (28.5% small + 59.8% medium) come in below 10,000 tokens. This is not a difference of degree. This is two different species of telemetry living in the same queue.

## Why the threshold matters

The thresholds are `smallMax=1,000` and `largeMin=10,000`. A 10,000-token row is roughly the size of a moderately long single conversation turn with a bit of tool output, or a small file read. It is not a heavy threshold. The fact that **claude-code, opencode, openclaw, codex, and hermes all post zero small rows and effectively zero medium rows** means every single observation in their queues clears that 10K bar.

Claude-code is the most striking: 299 rows total, 1 medium row (the rounding gives 0.3% in the lR% column, leaving 99.7% large), 0 small. Out of 299 wall-clock observations spanning months of usage, exactly one came in under 10K tokens. The other 298 all sit comfortably above. Opencode (322 rows), openclaw (428 rows), codex (64 rows), and hermes (167 rows) are all 100/100/100 — every single row, large.

This is what I mean by "binary". There is no smooth distribution of small to large work across these five tools. They are not occasionally cheap. They are *constitutionally* expensive on a per-row basis. The smallest row in the entire dataset for any of these five sources is still ≥ 10,000 tokens.

## What ide-assistant-A is actually doing

The ide-assistant-A row is the inverse:

- 333 rows (more than claude-code, more than opencode, more than codex+hermes combined)
- 1,885,727 total tokens — three orders of magnitude smaller than claude-code's 3.44 billion
- 28.5% small (< 1K tokens)
- 59.8% medium ([1K, 10K))
- 11.7% large (≥ 10K)

The token-mass split is even more telling: sT% 2.4%, mT% 38.0%, lT% 59.6%. So even though only 11.7% of rows are large, those large rows still carry 59.6% of the source's mass. The medium rows carry 38.0% of mass while accounting for 59.8% of rows. The small rows are essentially free — 28.5% of rows for 2.4% of mass.

This is a completion-engine signature, not an agent signature. Inline editor completions, hover/quickfix prompts, single-line refactor suggestions: these are all sub-1K token round trips by design. The medium tier is what you get when the completion expands to a multi-line block or a small refactor across a function. The 11.7% large tier is the long-tail — full-file rewrites, chat panel summarizations, "explain this codebase" queries.

The other five sources have **no completion tier at all**. They never make a sub-1K-token round trip. They never make a 1K–10K round trip either, except for claude-code's single observation. Whatever those tools are doing per row is structurally larger than what an inline IDE assistant does per row.

## The 1,884-to-1 mass ratio

Run the math on tokens per row:

- claude-code: 3,442,385,788 / 299 ≈ **11.51 million tokens per row**
- opencode: 3,339,893,298 / 322 ≈ **10.37 million tokens per row**
- openclaw: 1,854,152,370 / 428 ≈ **4.33 million tokens per row**
- codex: 809,624,660 / 64 ≈ **12.65 million tokens per row**
- hermes: 145,791,852 / 167 ≈ **873 thousand tokens per row**
- ide-assistant-A: 1,885,727 / 333 ≈ **5,663 tokens per row**

The ratio between the heaviest agent (codex at 12.65M tokens/row) and ide-assistant-A (5.66K tokens/row) is **2,235:1**. Even hermes — the lightest of the five "agents" — runs at **154:1** versus ide-assistant-A.

This is why a single global histogram of token sizes across all sources is essentially useless. The two populations don't overlap. ide-assistant-A's *largest* class (≥ 10K) starts where the other five sources' rows already live. There is no middle ground.

## Why this isn't just "ide-assistant-A makes more requests"

A naïve read would be "the IDE plugin fires off lots of small requests, the agents fire off fewer big ones, of course the per-row distribution differs". But that's not quite what's happening. ide-assistant-A has 333 rows. Claude-code has 299. Opencode has 322. The **row counts are within the same order of magnitude**. The difference isn't request frequency — it's request *shape*.

If you look at the smallest possible unit of work, ide-assistant-A and claude-code emit roughly the same number of observations. But each ide-assistant-A observation is a small completion or a small chat turn, and each claude-code observation is a multi-megaton conversation snapshot that includes the full prompt cache, the conversation history, a tool definition payload, and a generated response.

This suggests the queue's row grain is **not** "one user prompt → one assistant reply" uniformly across sources. For ide-assistant-A, that's roughly what a row is. For claude-code and similar, a row appears to be "one snapshot of an entire conversation state" — which is why even an idle re-snapshot lands above 10K tokens.

## The vendor-tenure cross-check

I cross-referenced this against `provider-tenure` in the same run window. Anthropic spans 6,499 hours, 615 active buckets, 7 distinct models, 6.87B tokens. OpenAI spans 6,042.5 hours, 686 active buckets, 6 models, 2.69B tokens. Google: 2,521.5 hours, 37 buckets, 1 model, 154,496 tokens. Unknown: 176.5 hours, 56 buckets, 1 model, 35,575,800 tokens.

The unknown bucket is interesting. It's only 176.5 hours of span (about a week), 56 buckets, 1 model, and 35.5M tokens. ide-assistant-A's lifetime is 1,885,727 tokens. That means the unknown provider — whatever it is — pushed roughly **18.9× more tokens in its first week than ide-assistant-A pushed in its entire dataset lifetime**. The IDE assistant is the smallest token producer by a wide margin, even though it's the third-largest by row count.

## Implications for cost modeling

If you build a cost model that estimates dollars per row using a single global mean, you will be wildly wrong. You'd have to maintain at least two separate cost-per-row priors — one for the agent population (where 100% of rows are large), one for the completion population (where 88.3% are small or medium). The thresholds in the `cost-class-mix` table are conveniently aligned to the breakpoints where pricing assumptions actually change: a 1K-token row is essentially free, a 10K-token row triggers cache-read pricing on most providers, and the long tail above 10K is where the bill actually accumulates.

For ide-assistant-A specifically, 59.6% of cost is concentrated in the 11.7% of rows that are large. So the operationally useful cost lever for that source is "what triggers the long-tail rows" — probably chat-panel queries and "explain this file" actions, not individual completions. For the other five sources, every row is a long-tail row, and there is no cheap mode to optimize toward.

## Implications for telemetry-row design

This dataset suggests the row schema is doing two different jobs at once. For ide-assistant-A, a row maps cleanly to a billable interaction. For the agents, a row maps to a snapshot of state, which inflates the token count by everything that's *along for the ride* — the conversation history, the tool registry, the system prompt, and the prompt cache. That's why the cost-class-mix collapses to "100% large" for those sources: the floor of a row is dominated by overhead, and the overhead alone exceeds the 10K threshold.

If you wanted a more honest distribution for the agent sources, you'd need to subtract the cached overhead and report only the *delta* tokens per row. That's a different metric — and it's why `cache-hit-ratio` and `source-cold-warm-row-ratio` exist as separate subcommands. But raw `source-cost-class-mix` is still useful precisely because it surfaces this binary so cleanly. You don't have to interpret anything. The 100/100/100 row of zeros across five sources is the entire finding.

## What to do with this

Three things:

1. **Stop using global token quantiles to set thresholds.** The p50 of "all rows" is meaningless when one population sits at 5K tokens and the other at 10M. Bucket by source first, then quantile within source.

2. **The `cost-class-mix` table is now a fingerprint.** Any future source that gets added to the queue can be classified instantly: is it 100% large (it's an agent, snapshotting whole conversations) or does it have a non-trivial small/medium share (it's an inline assistant)? The signature is visible from the first 50 rows.

3. **The 28.5% / 59.8% / 11.7% split for ide-assistant-A is itself a benchmark.** Other inline assistants, when added, should fall in roughly the same shape. If a future "completion engine" source is mostly large rows, that's a smell — either the source is mislabeled or it's snapshotting state instead of completing.

The whole finding lives in one row: `ide-assistant-A  333  1,885,727  28.5%  59.8%  11.7%  2.4%  38.0%  59.6%`. Run captured at 2026-04-27T01:26:47.310Z. Five sources at 100% large rows, one at 11.7%. That's the dataset's deepest stratification line, and it took a single subcommand to find it.
