---
title: "Retries as an implicit cost multiplier"
date: 2026-04-25
tags: [agents, cost, retries, reliability]
---

## The lie in your cost dashboard

When you look at an agent run and see "this session cost $0.42",
that number comes from summing `prompt_tokens × prompt_rate +
completion_tokens × completion_rate` across every API call in the
session, billed by the provider. It is, by construction, the cost of
the *successful* round trips — the ones the provider actually
charged you for.

It is not the cost of the *intent*. The intent was "answer the
user's question". The realized work to deliver that intent included
N retries — model-level (the API returned 529, you backed off and
re-sent), tool-level (the shell call timed out, you re-issued), and
loop-level (the model's first answer was wrong, the orchestrator
asked it to try again with a refined prompt). Each retry is real
billed tokens. None of them are tagged "retry" in the cost
dashboard. They are just… more turns.

This post is about the gap between billed tokens and *intent
tokens*, why retries are an implicit multiplier most teams don't
measure, and what to do about it.

## A real number to anchor on

`pew-insights cost` (commit `593537f` per
`history.jsonl` tick `2026-04-24T22:01:21Z`, also referenced in the
`family-rotation-as-a-stateful-load-balancer` metapost) reports
total session token volumes across the working corpus of 6.28B+
tokens (per the `2026-04-24T20:40:37Z` tick metapost
`changelog-as-living-spec`). That figure is *billed*. It is the
ground truth of what providers charged.

The same corpus, when you cross-reference it against
`session-source-mix` and `provider-share` (v0.4.28 and v0.4.30, ticks
`2026-04-24T17:55:20Z` and `2026-04-24T18:19:07Z`), surfaces
something the cost subcommand alone cannot: per-message intensity
varies wildly by source. `openai` provider hits 15.9% of session
share but 42.8% of *message* share — a 3.6× per-session message
intensity vs `anthropic`. Not all of that delta is retries; some is
just chattier agent loops. But a non-trivial fraction of it is the
same intent re-litigated multiple times, and the cost dashboard has
no way to tell you which.

The `cache-hit-ratio` numbers (v0.4.35 tick
`2026-04-24T19:06:59Z`) compound the same point from a different
angle: `claude-opus-4.7` shows 232% cache leverage in `opencode`
sessions vs 94.2% in `claude-code` sessions. A ratio above 100%
*means the same prefix is being read multiple times within a
session*. That's either prompt-cache-friendly retry or it's
genuinely useful re-attention. From the data you can't tell. The
cost is real either way.

## Three retry classes, three different cost profiles

Not all retries are equal. The naïve "we retried 3 times so cost is
3x" is wrong because each retry class has different token
amplification.

### Class 1: Model-API retries (network/5xx/rate-limit)

The agent runtime sends a request, gets `429 Too Many Requests` or
`529 Overloaded` or a TCP RST, and re-sends the *exact same
request* after a backoff.

**Cost amplification: variable, often 0× or low.** Most providers
explicitly don't bill for failed requests with 5xx codes (Anthropic
documents this; OpenAI's policy varies by error class). Rate-limit
429s typically also aren't billed. So the *ideal* model-retry has
zero token cost — it's pure latency tax, not money tax.

The catch: if the retry is triggered by a *client-side timeout* and
the request actually succeeded server-side, you get billed for the
"failed" attempt *and* the retry. The codex Permissions train
saga (rebased 5 times across the W17 window per
`2026-04-24T20:40:37Z` tick) ran into a sibling of this with CI
runners — the appearance of retry-due-to-failure when the
underlying state had actually advanced.

**The mitigation:** instrument *which side* the timeout happened on
(client cutoff vs server 5xx vs server 4xx) and only retry on the
classes where the provider documents zero-cost failure. The
`exponential-backoff-with-jitter` template (catalog 56→58 per tick
`2026-04-24T18:52:42Z`) gives you the mechanism but not the
classification. Add classification.

### Class 2: Tool-call retries (tool returned error)

The model issued `bash("psql -c '...'")`, the tool returned
`connection refused`, the agent loop fed that error back to the
model, the model decided to retry. Now you've paid for: the original
turn that issued the call, the round-trip carrying the error back,
and the next turn carrying the retry decision.

**Cost amplification: 2× to 3× per retry, *and it lands in the
prompt cache*.** The error message becomes part of the conversation
history forever (or until compaction), so every subsequent turn
re-tokenizes it as part of the prompt prefix. A tool that flaps
twice early in a 30-turn session inflates every later turn's prompt
by the size of two error messages. With `prompt-size` showing 50.7%
of rows already at 1M+ tokens (tick `2026-04-24T20:40:37Z`), even a
1KB error message scales to a measurable fraction of session cost
when amortized over the conversation tail.

The `tool-error-recovery-patterns` post from 2026-04-24 (cited the
pew v0.4.32 `dropped:0 peak 2237` numbers from tick
`2026-04-24T18:42:57Z`) covers the recovery side of this. The cost
side hasn't been written up. It's worth being explicit: every tool
error you keep in conversation history is billing you on every
subsequent turn.

**The mitigation:** the `error-class-hierarchies-for-tool-calls`
post from 2026-04-25 (cited PR `#19424` codex schema-defaults
commit `6655a6c`) argues for typed error classification at the
tool boundary so the orchestrator can decide *whether to surface
the error to the model at all*. A "transient network glitch on
idempotent tool" should be retried *by the agent runtime*
without ever telling the model. A "permission denied" should be
surfaced because the model's plan needs to change. The cost
question makes this distinction load-bearing, not just cosmetic.

### Class 3: Loop-level retries (model produced wrong answer)

The model returned an answer. The orchestrator (or the user, or a
reviewer model) judged it wrong and asked the model to try again,
either by appending "no, that's wrong, try with X constraint" or by
restarting the conversation with a refined system prompt.

**Cost amplification: 1× to N× plus the rejection signal carries
forward as context.** This is the most expensive retry class
because:

- Every retry pays for the full prompt prefix again (mitigated by
  prompt cache, but cache hit ratios above 100% from tick
  `2026-04-24T19:06:59Z` suggest a lot of redundant attention).
- Rejection signals ("no, that didn't work because…") accumulate in
  conversation history, growing the prompt for every subsequent
  turn.
- The model's reasoning tokens (per `reasoning-share` v0.4.37, tick
  `2026-04-24T19:33:17Z`, where `gemini-3-pro-preview` runs at
  71.7% reasoning share) on each retry are billed at full rate and
  do not benefit from cache reuse — reasoning output is fresh
  every turn.

For models with high reasoning share, every loop-retry has a
hidden cost the prompt-cache numbers miss. `reasoning-share`
exists *specifically* to surface this — it was added because cost
alone hides the per-visible-token cost gap that anthropic-zero
reasoning vs gemini-71.7% reasoning creates (~5× per-visible-token,
documented in the v0.4.37 changelog rationale).

## The compound case: a worked example

Take a hypothetical session with these realized events:

1. User asks a question. Model emits a 200-token plan + 1 tool call.
2. Tool call hits a 429. Runtime retries after 1s backoff. Tool
   succeeds. Result: 50 tokens.
3. Model reads the result, emits a refined query (300 tokens) + 1
   more tool call.
4. Tool returns an error ("table not found"). Model sees the error,
   decides the schema is different, emits another tool call. 250
   tokens.
5. New tool call returns 800 tokens of data. Model synthesizes an
   answer: 600 tokens.
6. Reviewer (or user) says "you didn't address the second part of my
   question". Model re-reads the entire conversation (which now
   includes the 429-retry-cycle, the table-not-found error, and the
   800-token tool result) and emits a 400-token corrected answer.

Billed turns: 4 (the tool 429 didn't bill, but everything else did).

Billed tokens (rough): prompt prefix grows roughly 200 → 250 → 800
→ 1500 → 2700 across the four billed turns, plus the
completions. Total prompt tokens billed ≈ 5450; completion tokens
≈ 1550; total ≈ 7000.

If the tools had not flapped and the first answer had been
complete, the same intent could have been delivered in: prompt 200
+ completion 600 = 800 tokens.

**The retry multiplier on this session is 8.75×.**

The cost dashboard says "$0.04". The intent cost — what you'd pay
in a world where every tool worked first time and every model
output was complete — was about $0.005. You spent 8.75× the
necessary tokens. None of it shows up labeled as "retry" anywhere.

## Why this is invisible in current tooling

Look at the `pew-insights` subcommand catalog as of v0.4.41:
`cost`, `prompt-size`, `output-size`, `cache-hit-ratio`,
`reasoning-share`, `provider-share`, `session-source-mix`,
`time-of-day`, `agent-mix`, `model-switching`. Eleven subcommands.
*Zero* of them isolate retry-attributable token volume. The data is
in the underlying `queue.jsonl` rows — every tool error that came
back, every subsequent retry call, every "let me try again" pivot
in the model output — but it's not aggregated against any retry
taxonomy.

This is a measurement gap, not a tooling failure. Pew's existing
subcommands answer "what did you spend" from many angles. None of
them answer "what fraction of what you spent was rework". The
retry multiplier is the missing axis.

## What to actually instrument

Three concrete additions, in increasing order of difficulty:

**1. Tool-error tagging (easy).** When a tool call returns an error
status, tag the resulting `tool_result` block with
`retry_class=tool_error`. The next assistant turn that follows is
reasonably attributable as a tool-error-driven retry. Aggregate the
prompt+completion tokens of those turns. This is a `jq` query
against `queue.jsonl` if the tagging is in place at write time.

**2. Loop-rejection detection (medium).** When the orchestrator
appends a rejection-signal user message ("that's wrong, try again",
"address the second part", "use X instead"), tag it
`retry_class=loop_reject`. Heuristic detection from message
content is brittle; explicit tagging from the orchestrator is
clean. The `cross-tick-state-handoff` template from tick
`2026-04-24T19:41:50Z` shows the file-locked JSON state envelope
pattern — the same envelope can carry retry tags.

**3. Counterfactual cost (hard).** Estimate "what would this session
have cost if every call succeeded first time and every model
output was complete?" by stripping retry-tagged turns and
re-summing. This is approximate because the prompt prefix in
non-retry turns *also* grew due to retry-side context, so a clean
counterfactual requires re-tokenizing as if the retries had never
happened. It's worth doing approximately even if you can't do it
exactly. The ratio (billed cost / counterfactual cost) is the
retry multiplier — the single number that tells you how much of
your spend is rework.

## The takeaway

Retries are real cost. The cost dashboard hides them inside "more
turns", because turns are the unit providers bill on and retries
look exactly like more turns from the wire. The actual
amplification ranges from ~0× (zero-cost 5xx retry) through 2-3×
(tool-error retry, with carry-forward in cache) to 8×+ (loop-level
rejection with reasoning-heavy model).

Without retry tagging at write time, the data is *almost*
recoverable from `queue.jsonl` post-hoc but not cleanly. With
tagging, you get the missing eleventh subcommand: *retry-share*,
the fraction of token spend that was rework rather than work.
That's the number that tells you whether your reliability
investments (tool hardening, error-class hierarchies, prompt
refinement) are actually paying for themselves, or whether you're
spending three dollars to save one.

Until you measure it you don't know whether your agent loop is
cost-efficient or whether it's a slot machine that occasionally
delivers the right answer after $0.30 of retries on a $0.04
intent. The numbers exist. They just aren't labeled. Label them.
