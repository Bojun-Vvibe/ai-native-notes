# The Fanout Asymmetry: claude=1.17, codex=3.00, opencode=7.00 — Three Harnesses, One Session Schema, Three Different Conversational Physics

Pull the 9,804-row session ledger out of `~/.config/pew/session-queue.jsonl`, filter to `kind: human` (7,484 rows), then collapse each session to a single number: assistant messages divided by user messages. Then bucket by `source`. The medians are not close. They are not even on the same order of magnitude in shape:

```
claude-code:  n=1108  zero-user=0   ratio med=1.17  p90=1.97  p99=2.00  max=2
codex:        n=478   zero-user=0   ratio med=3.00  p90=6.00  p99=8.50  max=10
opencode:     n=5898  zero-user=451 ratio med=7.00  p90=29.00 p99=63.00 max=111
```

Same schema. Same `session_key / source / kind / user_messages / assistant_messages / total_messages` shape. Same human at the keyboard, often the same human, often calling the same model — the top ten fanout opencode sessions are *all* `claude-opus-4.7`, the same weights `claude-code` is calling. And yet the harnesses produce conversational structures that look like three different species. This post is about why that ratio is the most useful single number you can extract from a session ledger, and what it tells you about how the harness — not the model, not the user — defines the protocol.

## What the ratio is actually measuring

`assistant_messages / user_messages` is, naïvely, "how many turns the assistant takes per user turn." For a chat-shaped REPL with strict alternation — user, assistant, user, assistant — the ratio is 1.0. For the median `claude-code` session, it is 1.17, which is exactly what you'd expect from a strictly-alternating REPL where occasionally the assistant gets a follow-up because of a tool result or a system reminder. The p99 is **2.00**, capped. The max is **2**. Claude Code's session-ledger physics says: the assistant is allowed to speak twice in a row, sometimes, and never more.

`codex` is bumped up. Median 3.00, p90 6.00, max 10. That's a tool-loop harness: each user turn typically expands into one reasoning message, one tool call message, one tool result acknowledgement, etc., counted as separate assistant messages in the ledger. The cap at 10 suggests the harness has either a hard turn budget or a strong convergence prior — it doesn't go off and chain ten thousand tool calls per user prompt.

`opencode` is in a different regime entirely. Median 7.00. p90 **29**. p99 **63**. **Max 111.** And — crucially — 451 of 5,898 human sessions (7.6%) have *zero* user messages despite being labeled `kind: human`. The other two harnesses have zero such sessions. Opencode's max of 111 means: one user prompt, one hundred and eleven assistant messages, 1,191 seconds of wall clock, all on `claude-opus-4.7`. The user said one thing. The assistant said one hundred and eleven things back. That's the agent loop running long.

## Same model, different ratios — so it's not the model

Pull the top ten fanout opencode sessions and look at the model field. Every single one is `claude-opus-4.7`:

```
ratio=111.0 u=1 a=111 dur=1191s model=claude-opus-4.7
ratio=103.0 u=1 a=103 dur=725s  model=claude-opus-4.7
ratio=102.0 u=1 a=102 dur=1188s model=claude-opus-4.7
ratio=100.0 u=1 a=100 dur=974s  model=claude-opus-4.7
ratio=93.0  u=1 a=93  dur=878s  model=claude-opus-4.7
ratio=88.0  u=1 a=88  dur=998s  model=claude-opus-4.7
ratio=85.0  u=1 a=85  dur=685s  model=claude-opus-4.7
ratio=85.0  u=1 a=85  dur=618s  model=claude-opus-4.7
ratio=84.0  u=1 a=84  dur=560s  model=claude-opus-4.7
ratio=83.0  u=1 a=83  dur=1182s model=claude-opus-4.7
```

Meanwhile the median `claude-code` session, also calling `claude-opus-4.7`, has a fanout of **1.17**. Same weights. Same provider. Same prompt cache infrastructure on the back end. The ratio jumped by ~6x. The only thing that changed is the harness: which CLI is wrapping the model, which protocol is counting messages, which loop is deciding when to ask the user a question versus when to take another autonomous step.

This is the single most important observation you can make about agentic CLIs in 2026: **the conversational shape is owned by the harness, not the model.** When people compare "Claude vs Codex vs whatever-else" by quality, they are mostly comparing harnesses pretending to be models. The model is a function — text in, text out. Everything else — when to call a tool, when to summarize, when to compact the context, when to ask for confirmation, when to give up and return — lives in the shell around it.

## The 451 zero-user opencode "human" sessions

In the same dataset, 451 opencode sessions are labeled `kind: human` but have `user_messages == 0`. None of `claude-code` or `codex` produces this. What is it?

The most likely explanation is the agentic continuation pattern: a session resumes from a previous one with no fresh user input — the user clicks "continue" or the harness auto-resumes a stalled task — and the entire continuation is assistant-side work (tool calls, reasoning, file edits) that finishes before the user types anything new. The ledger snapshot then gets written with `user_messages: 0` and a non-zero `assistant_messages`. From the schema's perspective the session is "human" because it has a human-shaped session key, but from a turn-taking perspective there's no human turn in the window.

7.6% of opencode "human" sessions are this shape. For `claude-code`, it's 0%. For `codex`, also 0%. The harnesses with stricter alternation simply don't produce this artifact. Opencode's looser, longer, more autonomous loop produces it as a natural consequence.

## Why this matters for billing, eval, and product design

### Billing

If you bill per assistant message, opencode users will look ~6x more expensive than claude-code users on the same model. If you bill per user message, opencode users will look ~6x cheaper. Neither number is wrong; they're measuring different protocols on top of the same weights. A reasonable per-session cost normalization needs to look at *both* axes (user messages and assistant messages), or at token mass directly, not at message count. The 9,804-row ledger here uses neither — it uses message counts as a structural signal, not a cost signal — but anyone building a usage dashboard on top of these numbers needs to know that `messages` is a harness-defined unit, not a workload-defined one.

### Eval

If your eval harness measures "tool-calling ability" by counting assistant messages per user turn, you've accidentally measured *the harness*, not the model. Two evals running against the same model through different CLIs will produce systematically different message-shape numbers, and the differences will dwarf any actual model-quality delta. This is why benchmarks that don't fix the harness are largely measuring the harness.

### Product design

The `claude-code` cap at p99=2.00 is a product decision, not a model limitation. Someone decided that the assistant should generally take one turn per user turn, with rare exceptions. The `opencode` p99 of 63 is also a product decision — someone decided autonomous tool loops are the point and the user shouldn't have to tap "continue" 60 times. Both are coherent products. They are not the same product. They produce session ledgers that share a schema and nothing else.

## The codex shape: tool-loop with a budget

Codex sits in the middle: median 3.00, p90 6.00, max 10. This is the signature of a tool-using harness with a hard turn budget. Each user prompt typically expands to: one reasoning message, one or two tool calls, one or two tool result digestions, one synthesis. That's 3-6 assistant ledger entries, depending on how the harness chooses to chunk the trace. The hard cap at 10 says: there is a turn-budget enforced somewhere, and it triggers around 10 assistant turns per user turn, after which the loop returns control even if it isn't done.

This is a defensible product choice. It bounds the worst case, makes per-prompt billing predictable, and prevents the assistant from going off and writing a novel when the user asked for a one-line fix. But it produces an entirely different conversational shape than opencode's "let it run until it converges or hits 1,191 seconds of wall time" approach. Neither is wrong. They are different decisions about who owns the loop.

## What to do with this number

The fanout ratio is the single cheapest characterization of an agentic CLI. From one number you can read off:

- **Strict alternation** (~1.0): chat REPL, user-driven.
- **1.x to 2.x** (capped p99): chat REPL with occasional follow-ups; harness enforces a tight turn budget.
- **3-6 typical with caps in low double digits**: tool-using harness with a turn budget.
- **median in single digits, p99 in 60+, max in 100+**: autonomous loop, harness lets the agent run.
- **non-trivial fraction of sessions with `user_messages == 0`**: the harness supports continuation without a fresh prompt, which means it's autonomous enough that "session" stops being synonymous with "conversation."

If you're building a session ledger of your own, capture both `user_messages` and `assistant_messages` separately, and capture them per source (per harness, per CLI). The marginal storage cost is two integers per session row. The information yield is enormous: it lets you separate workload (what the user asked for) from protocol (how the harness chose to do it).

## A note on the "human" label

The 9,804 rows split as `human: 7,484` and `automated: 2,320`. The automated class is exclusively `openclaw` (and only `openclaw` — no other source produces a single automated session). Every single automated session has `user_messages == 0`. The total assistant messages from automated sessions is 356,983 — call it 357k assistant messages with literally zero human turns triggering them. That's a separate post (and there are reasons to write it), but it's worth noting here because it changes how you should read the "human" label on the opencode rows. The human-vs-automated distinction in this schema is *binary at the source level*, not at the per-session level. Opencode produces 100% human-labeled sessions even when 7.6% of them have no user message. Openclaw produces 100% automated-labeled sessions even though they look structurally identical to those 7.6% opencode sessions in many ways. The label is a source-tag, not an inference.

This means the fanout ratio for `opencode` is computed over a population that includes some sessions that are arguably automated continuations. If you wanted a stricter "user-driven only" filter, you could exclude `user_messages == 0` rows. Doing so, the medians don't move much (claude-code: 1.17 → 1.17, codex: 3.00 → 3.00, opencode: 7.00 → 7.00), because the zero-user opencode sessions are mostly *short* — the median continues to be dominated by the bulk of the distribution. But the means do move, and the long tail (p99=63, max=111) is essentially intact regardless of how you filter. The shape is robust.

## The 6x harness premium

The headline number is the median ratio: 1.17 vs 3.00 vs 7.00. The opencode median is roughly 6x the claude-code median on the same models against the same humans. That ratio is the harness premium — it is what the agentic loop costs in additional assistant turns per user turn. For 5,898 opencode sessions at median ratio 7.00 vs a counterfactual ratio of 1.17, the implied "extra" assistant turns are roughly 5,898 × (7 − 1.17) ≈ 34,400 additional assistant messages produced because the harness chose autonomy over alternation. At the long tail, those extra turns are where most of the wall-clock time and most of the token budget go.

If you're measuring agent productivity by output produced per user prompt, opencode wins by a multiplier. If you're measuring it by output produced per assistant turn, opencode loses by the same multiplier. The harness is the variable. Pick the metric that matches what you actually want the harness to do.

## Closing

One number, one ledger column, one collapse function — `assistant_messages / user_messages` — and three CLIs that share a schema diverge into three distinct species. Median 1.17 vs 3.00 vs 7.00 on the same models is not a model story. It is a harness story. If you only ever pull one statistic out of an agentic-CLI session ledger, pull this one.
