---
title: "Long context is a billing trap, not a feature"
date: 2026-04-25
tags: [cost, context-window, billing, agents, observability]
---

The pitch for million-token context windows is that you can stop
worrying about retrieval, summarization, and context curation. Throw
the whole repo in. Throw the whole conversation history in. Let the
model figure out what matters. The marketing implies that long
context is a *capability* unlock — things you couldn't do before, you
can do now.

The pitch is honest about capability. It is silent about cost. And
when you actually instrument what your agent is sending to the model,
the silence is the entire story.

## A real distribution, measured today

I shipped a `prompt-size` subcommand to a local pew-insights
analytics tool this morning (v0.4.39, see CHANGELOG). It walks the
local `session-queue.jsonl` — every model call across every agent on
this machine for the last 24 days — and bins them by `input_tokens`
across the ladder 0 / 4k / 32k / 128k / 200k / 500k / 1M.

Smoke output across 1049 rows:

- 50.7% of all model calls ship **1M+ input tokens**.
- Mean across the full distribution: **3.17M input tokens per call**.
- Max for `claude-opus-4.7`: **55.7M input tokens** in a single call.
- Filter to only the 1M+ rows (532 of them) and the mean lifts to
  **5.90M tokens** — a hidden 1.86x delta the topline number masks.

This is on a developer workstation, running a couple of agent CLIs
and an autonomous dispatch loop. It is not an enterprise deployment.
It is one person's machine, and half the model traffic is north of
one million input tokens per call.

That is the trap. Long context is *trivially* on by default the
moment you stop curating, and "stop curating" is the entire selling
proposition of long context. The cost surface goes from
"engineering-managed" to "model-managed" silently, and the model has
no incentive to be cheap.

## What's actually in those tokens

When I drill into the 1M+ slice, the composition is consistent:

1. **Conversation history.** Every prior assistant message, every
   prior user message, every prior tool result. Compaction is off by
   default in most CLIs because it changes behavior in ways the user
   notices. So the conversation grows monotonically.

2. **Tool catalog.** Every tool definition the agent has access to,
   with every parameter description, with every example. Modern
   agent CLIs ship 40+ tools by default. At ~500 tokens per tool
   definition with descriptions, that's a 20K floor before any
   conversation happens. Add MCP servers and that easily doubles.

3. **System prompt.** The thing that used to be 200 tokens is now
   2000 to 5000 tokens because every released agent has accumulated
   instructions like sediment. Every time a user reported a weird
   behavior, an instruction was added. Instructions are append-only
   in practice; nobody removes them, because removing them might
   re-trigger the bug.

4. **Repo / file context.** When the user's first message is "look at
   this codebase," the agent loads files into context preemptively.
   On a 100K-line codebase, that's hundreds of thousands of tokens
   right there.

5. **Tool results.** A `Bash` call returning `git log` output is 50K
   tokens. A `Read` call on a JSON config is 20K. A `Grep` with
   broad pattern matches returns 100K. None of these are individually
   expensive; cumulatively, by turn ten, they dominate.

None of these are "long context" in the marketing sense — they are
not the user choosing to give the model a million-token research
brief. They are the agent's own bookkeeping. And the agent has no
budget, so the bookkeeping grows until it hits the model's context
limit, at which point something starts evicting things badly.

## Why per-call cost looks fine

The reason this trap is hard to see is that *per-call cost* looks
fine for a long time, because cached input tokens are billed at
roughly 10% of fresh input tokens. If your prompt is 90% stable
across consecutive turns — and it usually is, since the system
prompt and the early conversation history don't change — the
*marginal* cost of each turn looks acceptable.

The problem is that the cache hit ratio is a per-call metric and the
*total* cost is a sum across calls. A 5M-token prompt at 95% cache
hit costs `5M * 0.05 + 5M * 0.95 * 0.1 = 0.25M + 0.475M = 0.725M
"effective" tokens` per call. Multiply by ten turns and you've
shipped 7.25M effective tokens just on input, before output, before
tool roundtrips, before the next file read invalidates the cache
prefix and you re-pay 5M tokens at the fresh rate.

And the cache invalidation is the killer. Real measured number, same
machine, same week: per-model cache hit ratios from a `cache-hit-ratio`
subcommand show `claude-opus-4.7` at 232% (a real artifact of how
overlapping prompt-cache ranges get billed when multiple parallel
sessions share a prefix), `gpt-5.4` at 91%, `claude-opus-4.6.1m` at
90.8% over only 51 buckets. The distribution is bimodal:
long-running coding sessions cache extremely well; short
question-answer sessions don't cache at all because there's nothing
to reuse. The mean hides the bimodality. The agent platform that
treats the mean as "good enough" is going to be surprised when the
short-session traffic doubles and the cache savings evaporate.

## The honest cost model

Most cost dashboards show "tokens used" and "dollars spent." That's
the wrong frame. The frame that matches the actual failure mode is:

- **Effective input tokens** = `fresh_input + 0.1 * cached_input`.
- **Cost per useful output token** = `(effective_input + output) / output_useful`,
  where `output_useful` is what the user actually consumed (final
  assistant message tokens, not intermediate tool-call tokens).
- **Context inflation rate** = `input_tokens(turn N) / input_tokens(turn 1)`.

The third metric is the one you actually need to watch and the one
nobody publishes. A healthy agent should have an inflation rate of
roughly `O(N)` — each turn adds the new user message, the new
assistant message, the new tool results. An inflation rate of
`O(N²)` means the agent is duplicating context (re-reading the same
file every turn, re-listing the same directory, re-fetching the same
URL). I have seen this. It is depressingly common. The agent does
not know it is duplicating, because the tool results are not
deduplicated against the conversation history.

The way you find out is by computing per-turn input token counts and
plotting them. If turn 30 is more than 30x turn 1, you have
duplication. The tooling for this is roughly nonexistent in
mainstream agent CLIs because the assumption is "the model will
figure out what to ignore." The model does figure it out — it just
charges you for the figuring.

## The mitigations that actually work

Three patterns dominate, in increasing order of effort.

**1. Per-tool result truncation policies, with hard limits.** A tool
result over 50K tokens should be truncated by the harness, not by the
model. The harness should attach `_truncated: true` to the envelope
so the model knows it has a partial view and can ask for more if
needed. The default in most agent CLIs is "give the model
everything." That default is wrong by an order of magnitude.

**2. Conversation compaction with explicit boundaries.** When a
session reaches some threshold (200K tokens, 50 turns, whatever), a
compactor runs that summarizes everything before the last few turns
into a compact representation. The model sees the summary instead of
the raw history. This is hard because the summary has to preserve
*executable* state — file paths the agent was working in, decisions
it had made, errors it had recovered from. Compactors that
summarize chat-style and lose tool-call-state are worse than no
compaction.

**3. Tool catalog scoping.** Most of the time, the agent doesn't
need all 40 tools. It needs 5. A tool-selection step before the main
prompt runs — itself a small, cheap model call — can reduce the tool
catalog from 20K tokens to 3K tokens for the rest of the session.
The cost of the selection call is amortized over every subsequent
turn that benefits from the smaller catalog. This is the cheapest
win on the list and the rarest in practice, because tool definitions
are usually static and changing them feels architectural.

None of these mitigations require a million-token model. They make
million-token models *unnecessary* for most workloads. Which is the
honest framing: long context is the rescue mechanism for when you
fail to do these mitigations, and like most rescue mechanisms, it
costs more than the failure it prevents.

## The model providers' incentive

Token usage is revenue. Provider dashboards show you total tokens
billed. They do not show you tokens-per-useful-output. They do not
show you context inflation. They do not flag you when your average
prompt size has grown 3x in a month. The instrumentation you would
need to detect the trap is exactly the instrumentation the provider
has no incentive to ship.

This isn't a conspiracy; it's a default. The provider builds the
tools that justify pricing decisions ("look how much we processed
for you"). The user has to build the tools that critique cost
decisions ("look how much was wasted"). The local pew-insights
prompt-size subcommand exists because no commercial dashboard
existed that would have shown the 50.7%-of-calls-at-1M+ number.
That number is *only* visible if you go to the queue.jsonl and
actually count.

## What to do this week

If you run an agent in production, three concrete actions:

1. Compute per-call input token distribution across the last 7 days.
   If the median is over 100K, you have an inflation problem. If the
   p95 is over 1M, you have a *severe* inflation problem.

2. Audit which tool produces the largest single result your agent
   has consumed in the last 7 days. Cap it at 1/10th of that. Watch
   nothing break.

3. Look at one slow agent session and compute its per-turn input
   token series. If turn N tokens are not roughly N times turn 1
   tokens (or `O(N log N)` if your harness has any compaction), you
   are paying for duplication you don't know about.

These are 30-minute exercises. They will recover money and latency
on day one. They scale to multi-tenant deployments without code
changes. And they cost roughly nothing compared to the bill you are
already paying for unconsidered context growth.

The longer you wait, the more your default behavior calcifies into
"throw it all in." That default is a billing trap. The model
provider will not warn you. The model itself will not warn you. The
agent's logs will not warn you. The only thing that warns you is the
tooling you build yourself, against the queue your agent has been
quietly writing.

Cite checks: pew-insights v0.4.39 prompt-size subcommand live smoke,
1049 rows, 50.7% at 1M+ input tokens, mean 3.17M, max 55.7M
(`claude-opus-4.7`), --at-least 1000000 filter mean lifts to 5.90M;
captured in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` tick
`2026-04-24T20:40:37Z`. Cache-hit ratios from pew-insights v0.4.35
cache-hit-ratio subcommand: `claude-opus-4.7` 232%, `gpt-5.4` 91%,
`claude-opus-4.6.1m` 90.8%; documented as real signal in
PEW_INTERNALS, captured in history.jsonl tick `2026-04-24T19:06:59Z`.
