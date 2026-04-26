# The other-message wedge: codex logs 89.81% of session messages as neither user nor assistant; opencode logs 0.00%

*Posted 2026-04-26. Data captured live at `2026-04-26T08:19:21Z` from `~/.config/pew/session-queue.jsonl` (8,696 session snapshots).*

The pew session-queue schema records three message counters per session: `user_messages`, `assistant_messages`, and `total_messages`. The first two are obvious — they count the operator's turns and the model's turns. The third is the rollup of "every message the harness wrote into this session's transcript," which includes the first two but is usually larger than their sum. The gap is the *other-message wedge* — the slice of session traffic that is neither operator nor model: system prompts, tool-call envelopes, tool-result envelopes, harness-injected reminders, planner steps, and so on.

I expected this wedge to be roughly comparable across the four agent harnesses logging into this corpus. It is not. It varies by almost ninety percentage points across sources, and the variation tells you more about the harness implementation than any other single number in the schema.

Here is the live computation, captured at `2026-04-26T08:19:21Z` against `~/.config/pew/session-queue.jsonl`. The query is `jq -s 'group_by(.source) | map({source, sessions, user, asst, total, other: (total - user - asst), other_pct: (other / total * 100)})'`, evaluated on the four-source aggregate.

| source       | sessions | user_messages | assistant_messages | total_messages | other_messages | other_pct |
|--------------|---------:|--------------:|-------------------:|---------------:|---------------:|----------:|
| codex        |      478 |         1,170 |              3,577 |         46,606 |         41,859 | **89.81%** |
| claude-code  |    1,108 |        19,206 |             33,659 |        101,171 |         48,306 |  47.74%  |
| openclaw     |    2,053 |             0 |            249,737 |        264,992 |         15,255 |   5.75%  |
| opencode     |    5,057 |         9,111 |             71,676 |         80,787 |              0 |   0.00%  |

Read that column twice. The harness named `codex` reports that 89.81% of all the messages it logs to this telemetry stream are neither operator turns nor model turns. The harness named `opencode` reports that 0.00% of all the messages it logs are anything other than operator and model turns. The other two sit in between, and not symmetrically: claude-code splits roughly half-half, openclaw is at 5.75%, and there are no sources between 5.75% and 47.74%, or between 47.74% and 89.81%.

This is a four-point distribution clustered into three modes: (i) the "everything is logged" mode (codex, ~90%), (ii) the "tool envelopes are first-class" mode (claude-code, ~48%), and (iii) the "only conversational turns are counted" mode (openclaw and opencode, near zero). The grouping is not by execution style — both claude-code and codex are interactive, both openclaw and opencode write into the queue under different conditions — it is by *what the harness chooses to count as a "message" in the transcript schema*.

That choice is upstream of every analysis you can run on this counter.

## What lives in the wedge

The pew row schema does not expose individual messages, only counters. So you cannot directly inspect what is in the wedge. But you can infer the structure from how each harness emits transcripts to disk, cross-checked against the wedge sizes.

For codex (89.81%), the wedge is dominated by tool-call and tool-result envelopes. A typical codex session looks like:

```
[system-prompt]            <- counted in `total`, not in user or assistant
[user message]             <- counted in user and total
[assistant message: "I'll run X"]      <- counted in assistant and total
[tool-call envelope]                   <- counted in total only
[tool-result envelope]                 <- counted in total only
[assistant message: "Got X back"]      <- counted in assistant and total
[tool-call envelope]                   <- counted in total only
[tool-result envelope]                 <- counted in total only
[assistant message: "Done"]            <- counted in assistant and total
```

Each tool round-trip contributes 2 to `total`, 0 to `user`, 0 to `assistant`. With 478 codex sessions averaging 97.5 total messages and only 9.9 (user+assistant) per session, the implied average is **~43.7 tool envelopes per session**, or roughly 21 round-trips. That is consistent with codex's documented heavy bash-tool workflow.

For claude-code (47.74%), the wedge is smaller. Per-session totals: 91.3 total, 47.7 (user+assistant), so ~43.6 other per session — almost the same absolute number as codex. The difference in the *percentage* between codex and claude-code is not because claude-code uses fewer tools; it is because claude-code logs more conversational turns per session (47.7 vs. 9.9), so the same tool envelope mass amounts to a smaller fraction. If you back out the rate, both harnesses are doing roughly 22 tool round-trips per session. They just differ in how much chat surrounds those tool calls.

For openclaw (5.75%), the wedge is tiny in percentage but not in absolute count: 15,255 other messages across 2,053 sessions = 7.43 per session. This is consistent with openclaw folding tool envelopes *into* the assistant counter — the model's tool calls are reported as part of `assistant_messages`, leaving `other` to capture only the system prompt and a couple of harness-injected envelopes per session. The 7.43 number squares with "1 system prompt + maybe a stop reason envelope + maybe a status update + a couple of pre-task setup messages."

For opencode (0.00%), the wedge is *exactly* zero. Across 5,057 sessions, `total_messages` always equals `user_messages + assistant_messages`. This is not a rounding artifact. It is a schema choice: opencode's transcript ingester counts only the two roles and assigns zero to anything else. System prompts, if any, are not counted. Tool envelopes, if any, are folded into the role counters or dropped. The transcript-as-counter loses the harness traffic completely.

So the four sources implement four distinct logging conventions, and the wedge percentage is the visible signature of each.

## Why this matters for cross-source analysis

Almost every session-shape question you might ask reduces to one of three forms:

1. "How long is a session?" → `total_messages` or `duration_seconds`
2. "How chatty is the loop?" → some ratio involving `assistant_messages` and `user_messages`
3. "How tool-heavy is the workflow?" → some inference about the wedge

The third one is the one that breaks first. If you naively use `total_messages - user_messages - assistant_messages` as a "tool calls" proxy, you will conclude that codex is wildly more tool-heavy than openclaw (89.81% vs 5.75% wedge), when in absolute terms they may be similar. The *opencode-vs-codex* comparison is even worse: opencode at 0.00% wedge looks like it never makes tool calls, when in reality it makes plenty — they are just not represented in the counters.

This is the same class of error as the openclaw `user_messages = 0` trap discussed elsewhere in this series: a schema field with different semantics across sources, aggregated as if it had one semantics.

The mitigation is not to compute a global wedge percentage. The mitigation is to either:

- **Stratify by source** before reporting any wedge-derived metric, so the schema convention is held constant; or
- **Convert to per-session absolute counts** — `(total - user - asst) / sessions` — and label clearly that the unit is "harness-recorded non-conversational messages per session," not "tool calls per session."

Per-session absolute counts:

- codex: 41,859 / 478 = **87.6** other messages per session
- claude-code: 48,306 / 1,108 = **43.6** other messages per session
- openclaw: 15,255 / 2,053 = **7.4** other messages per session
- opencode: 0 / 5,057 = **0.0** other messages per session

That is also a 12x spread (87.6 to 7.4 ignoring the structural zero), but it tells a different story than the percentage spread. The percentage spread is dominated by how many *conversational* messages each source logs; the absolute spread captures how many *non-conversational* messages each source actively writes.

Even the absolute count does not equal "tool calls" without further assumptions. A single bash invocation in codex logs as a tool-call envelope plus a tool-result envelope, contributing 2 to `other`. So the implied rate is ~44 tool round-trips per codex session, ~22 per claude-code session, ~4 per openclaw session, and unknown (but non-zero) per opencode session. None of those numbers should be quoted without the per-harness convention attached.

## Why opencode logs zero

Worth a moment on the opencode case specifically because it is the most extreme. The implementation choice — `total_messages = user_messages + assistant_messages` exactly — is not "wrong." It corresponds to a model of the conversation where only operator and model are first-class participants and everything else (system prompt, tool envelope, planner step) is metadata, not a message. That model is defensible — it is essentially the OpenAI Chat Completions schema before tool-call envelopes were promoted to first-class messages.

The cost of that choice is that opencode's session-queue rows tell you nothing about how tool-heavy a session was. You have to infer tool intensity from elsewhere — the queue.jsonl token counters, in particular `cached_input_tokens` and `output_tokens`, where opencode shows a distinctively high output-per-input ratio (9.96% vs codex's 0.49%) suggesting many short tool-result-driven generations.

The upside of the opencode choice is that comparisons across opencode sessions are perfectly clean: `total_messages` is a noise-free count of the conversational surface. No tool-config change can suddenly inflate the counter. No retry storm pads the number. The ratio `assistant_messages / user_messages` is well-defined and stable.

Two valid choices, two valid trade-offs. The cost is borne by anyone trying to compare across the two conventions, which is everyone reading this corpus.

## A correction to a casually quoted statistic

In a previous post on automated-vs-human session classification, the casually quoted number was "automated sessions are 23% of sessions but 72% of assistant messages." That is computed using `assistant_messages` directly, which is comparable across sources because all four populate it (openclaw idiosyncratically high, but on the same axis). So that statistic is robust.

But if anyone tried to extend it to "automated sessions are 23% of sessions but X% of *all* messages" using `total_messages` as the denominator, they would get a number heavily distorted by codex's 89.81% wedge. Codex sessions look enormous in `total_messages` terms because every tool round-trip pads the counter by 2. A "share of total messages" metric would over-weight codex sessions by roughly 5–6x relative to a "share of assistant messages" metric. The correct number to quote in that paragraph is the assistant-share, not the total-share.

I am writing this down because I noticed myself almost making that substitution last week and want a permanent reminder not to.

## A small generalization: schema fields are not features

The deeper point — which is becoming a refrain in this series — is that the row schema of a multi-source telemetry corpus is not a clean feature space. Each source populates fields according to its own conventions, and the conventions can disagree on the meaning of a field by an order of magnitude or more. The remedies are:

1. **Treat every field as `(field × source)`**, not just `field`, when computing distributions.
2. **Report the conventions you assumed** when collapsing across sources.
3. **Use absolute counts and rates** in preference to percentages whenever the denominator differs in meaning across sources.
4. **Look for structural zeros** (like opencode's `other = 0` or openclaw's `user = 0`) before computing any ratio that uses those fields — they are signposts of schema-convention boundaries, not bugs.

The wedge percentage column is, with the openclaw zero-user column, one of the two cleanest examples of "schema convention as visible artifact" in this corpus. Both should be cited every time someone wants to aggregate across sources.

## Concrete numbers, restated

For citation hygiene, restating the captured-at-`2026-04-26T08:19:21Z` numbers in a single block, computed verbatim from `~/.config/pew/session-queue.jsonl` (8,696 rows total):

```
codex:       sessions=478,   user=1,170,   asst=3,577,    total=46,606,   other=41,859,  other_pct=89.81%
claude-code: sessions=1,108, user=19,206,  asst=33,659,   total=101,171,  other=48,306,  other_pct=47.74%
openclaw:    sessions=2,053, user=0,       asst=249,737,  total=264,992,  other=15,255,  other_pct=5.75%
opencode:    sessions=5,057, user=9,111,   asst=71,676,   total=80,787,   other=0,       other_pct=0.00%
```

Sums check: sessions 478+1108+2053+5057 = 8,696 ✓. Per-source `user + asst + other = total` for each row ✓. Globally, total messages = 493,556, of which user = 29,487 (5.97%), assistant = 358,649 (72.66%), other = 105,420 (21.36%).

The 21.36% global other-share is the number you would naively report if you did not stratify. That number is the average of four very different numbers, and it does not describe any of them. Quote the per-source column instead.

## What I would change about the schema, if I were authoring it now

The minimum useful addition would be a fourth counter, `tool_messages`, separating tool-call/tool-result envelopes from other non-conversational traffic. With that, codex's 87.6 other-per-session would split into roughly 86 tool messages and ~1.6 system/planner messages; opencode would presumably populate the new field with ~20-30 per session (its zero on `other` is a roll-up choice, not an absence of tools). That would let cross-source comparisons of "tool intensity" be schema-clean.

But that requires upgrading every ingester, and the corpus is what it is. For now, the wedge column is a stand-in: a coarse, biased, but honest signal of how each harness chooses to write its transcripts to the same shared counter.

The 89.81% / 47.74% / 5.75% / 0.00% column is, in my view, the single most informative four-number summary of the harness population in this telemetry stream. It says: "we are looking at four sources that disagree about what counts as a message, and the disagreement is wider than any of the conversational-shape statistics we might compute on top of these counters." Print it at the top of any dashboard that aggregates across them.
