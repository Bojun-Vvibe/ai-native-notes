# The Cold-Start Signature: claude-code's First 5 Rows Run 3.0x Hot, ide-assistant-A Runs 6.5x Total but 0.7x on Output, and What "Source Onboarding" Looks Like in the Telemetry

*Posted 2026-04-26 — `ai-native-notes/posts/`*

## Frame

Sources don't show up to a queue at full intensity. The first call from a freshly-installed
harness is necessarily different from the 200th — there's a project to index, a token cache to
warm, a session shape to discover, a user who is still figuring out whether the tool is worth
paying attention to. That difference should be visible in the telemetry as a distinct
**cold-start signature** in the first few rows of each source.

This post asks one question: *if I take each source's first 5 rows and compare them against
that source's remaining rows, what does the per-source onboarding curve look like?* I will use
two metrics — average `total_tokens` and average `output_tokens` — and the framing pulls a
surprising decoupling out of the data.

## Source corpus

`~/.config/pew/queue.jsonl` has **1571 rows** at the time of writing. The rows split across six
distinct `source` values:

| source           | n   |
|------------------|----:|
| openclaw         | 409 |
| ide-assistant-A  | 333 |
| opencode         | 303 |
| claude-code      | 299 |
| hermes           | 163 |
| codex            |  64 |

(I write `ide-assistant-A` for the editor-integrated assistant source to keep house style
consistent with the rest of `ai-native-notes/posts/`.)

For each source I take the rows in the order they appear in the queue (which is chronological by
`hour_start`), pull the first 5 rows, and average their `total_tokens` and `output_tokens`. Then
I compute the same averages for the remaining rows of that source (n − 5) and report the ratio.

## The headline table

| source           | first-5 avg total | rest avg total | ratio | first-5 avg output | rest avg output | output ratio |
|------------------|------------------:|---------------:|------:|-------------------:|----------------:|-------------:|
| openclaw         |  5,476,933        |   4,256,573    | 1.29  |  19,759            | 11,712          | 1.69         |
| opencode         |  3,429,503        |  10,532,785    | 0.33  |  17,001            | 69,531          | 0.24         |
| claude-code      | 33,578,862        |  11,137,726    | **3.01**  |  96,338        | 39,616          | 2.43         |
| codex            | 17,567,536        |  12,233,678    | 1.44  |  40,097            | 31,264          | 1.28         |
| hermes           |  1,909,175        |     857,237    | 2.23  |  14,824            |  8,328          | 1.78         |
| ide-assistant-A  |     34,221        |       5,228    | **6.55**  |   2,417        |  3,424          | **0.71**     |

Three sources cold-start hot on **both** dimensions: claude-code, codex, hermes. One source
cold-starts hot on total but **cold on output**: ide-assistant-A. One source cold-starts
*ice-cold*: opencode. And openclaw is roughly steady-state at row 1.

That's six different shapes from six sources. Below I take each source one at a time and ask
*why*.

## claude-code: 3.01x on total, 2.43x on output

claude-code's first 5 hour-rows average **33.6M total tokens** vs **11.1M** for the remaining
294 rows — a 3.01x cold-start. The output side is a 2.43x burst (96k vs 40k). Both metrics
spike together; they correlate.

This is the most "intuitive" cold-start in the dataset: the harness lands, walks a project
tree, loads a deeply layered context, and emits a long response. A new claude-code session
typically begins with a "read these N files" pass that fills the context window aggressively
because there is no warm cache to lean on, then continues with a long answer. Both axes spike
because the user's first prompt is generally also the highest-leverage prompt of the session
("here is the codebase, what do you see").

Once the project is in cache and the user has narrowed scope, both averages drop. After row 5,
context loading is typically incremental — load one or two more files, not the whole tree.

Note also that claude-code owns 3 of the global top-10 output rows (see the companion post on
output-token Pareto concentration). The 2.43x output cold-start is consistent with that: the
first interactions of a session are precisely where the long output rows tend to land.

## opencode: 0.33x on total, 0.24x on output — the *opposite* of cold-start

opencode is the only source where the first 5 rows are **smaller** than the remaining 298 by a
factor of 3 on total and a factor of 4 on output. Average first-5 output is 17k tokens. Average
rest output is **69.5k**.

I want to call this an "anti-cold-start," but the more accurate read is that opencode's earliest
hour-rows in the queue are *not* its first real production runs — they are setup/probe traffic.
The data is consistent with the user (me) installing opencode, running a couple of small "is
this thing working" prompts, then closing the session. The "rest" of opencode's history is the
production traffic that came later, which dwarfs those probe rows.

This is a methodological note as much as a finding: "first 5 rows of a source" is not always
"first 5 calls of the cold-start phase." For opencode specifically, the first 5 rows of the
queue precede a multi-day gap before opencode showed up in earnest. For other sources, the first
5 rows are tighter against the burst of real use.

A more precise cold-start metric would be "first 5 rows after the first session boundary" — but
the queue does not carry a session_id, so I cannot directly identify session boundaries. The 5-row
heuristic is what I have.

## claude-code vs opencode: the same weights, opposite onboarding shapes

This is the most interesting comparison in the table. **claude-code and opencode both run
predominantly on `claude-opus-4.7`** (the same weight family that dominates the upper tail of
the queue's output distribution). They both are CLI/agent harnesses. They share orders of
magnitude on total token volume per row. And yet one cold-starts at 3.01x and the other at
0.33x.

The most plausible explanation: **the two harnesses have different "default project intake"
strategies**. claude-code defaults to scanning a working tree on session start; opencode defers
context loading to user-driven file references. So claude-code's row #1 looks like
"32M tokens of context for one paragraph of analysis"; opencode's row #1 looks like "200k
tokens for whatever the user pasted in." The user behavior pattern then asymmetrically inflates
claude-code's first-5 average and deflates opencode's.

If that's right, the ratio between these two cold-start factors (3.01 / 0.33 = ~9.1) is
essentially measuring the gap between an aggressive-by-default context strategy and a
lazy-by-default one. That is a useful artifact: it suggests cold-start ratio is a viable
**proxy for harness onboarding policy** even when you don't have access to the harness's
internals.

## codex: 1.44x on total, 1.28x on output — gentle onboarding

codex's first 5 rows run 1.44x hot on total, 1.28x on output. The smallest cold-start factor
among the "starts hot" cluster. With only n=64, the first 5 rows are 7.8% of the source's
entire history, so this number has the highest sampling variance of the table.

Still, the direction is clear: codex *does* warm up rather than cool down, but mildly. The
codex onboarding pattern in this queue looks like a measured "type a question, get a moderate
answer" style — no project-tree blast, no probe phase. First-5 output averages 40k vs 31k
steady-state. The harness behaves consistently.

## hermes: 2.23x on total, 1.78x on output

hermes is a routing/proxy layer in this dataset, not a generation engine. Yet it shows a clear
cold-start: first-5 average total of 1.9M vs 857k steady-state, and 14.8k output vs 8.3k.

That is more interesting than it first looks. If hermes were a pure pass-through, its
distribution should mirror its callers' distribution, smeared. But hermes' caller mix presumably
shifts over time — early on, the proxy may have been routed against by a handful of high-volume
clients, then later by many small ones. The 2.23x total ratio is consistent with that
story: the **first 5 hour-rows of hermes traffic happen to coincide with bursty upstream
clients** that did not persist as steady producers. That makes hermes' "cold-start" really a
signature of *its callers' onboarding*, refracted through the proxy.

This is the kind of source where cold-start ratio is structurally informative even if hermes
itself doesn't change behavior — because its volume tracks the union of upstream onboarding
curves.

## openclaw: 1.29x on total, 1.69x on output — basically steady-state at row 1

openclaw's first-5 averages are within 30% of its rest-of-corpus averages on total and within
70% on output. Compared to the other sources, this looks like an asymptotically flat curve.

That is consistent with openclaw being a long-running, daily-cadence source. It produces a
narrow band of token volume per hour-row, repeatedly, throughout the queue. There is no project
intake, no probe phase, no caller-mix shift. Row 1 looks like row 100. Row 100 looks like row
400.

It is also the source with the largest n in the queue (409 rows), so it is the source where the
first 5 rows are the smallest fraction of total history (1.2%). Cold-start influence on its
mean is structurally minimal.

## ide-assistant-A: 6.55x on total, 0.71x on output — the asymmetry

This is the row in the table that should make you stop reading and squint.

ide-assistant-A's first 5 rows run **6.55x** on total tokens (34k vs 5.2k steady-state) but
**0.71x** on output tokens (2,417 vs 3,424). Total balloons; output *contracts*.

The mechanical decomposition: `total = input + output + reasoning + cached_input` (with the
caveat that cached input is double-counted in some accountings — see earlier posts on the
token-accounting identity). For ide-assistant-A's cold start to spike total while output drops,
the **input/cached side must be carrying the weight**.

That is consistent with a specific harness pattern: when an editor-integrated assistant first
loads, it ingests the open file(s), surrounding context, project metadata, language-server
output, and possibly indexed documentation — then asks a small clarifying question or returns
a short suggestion. Total input swells. Output stays small because the user is in
inline-completion / short-suggestion mode, not chat mode. Once the editor session settles, input
shrinks (no more re-loading; cache is warm) but output rises modestly as the user asks for more
substantial generation.

Functionally, ide-assistant-A's cold-start is not an output-burst; it's an **input-burst with
output suppression**. The 6.55 / 0.71 split is the cleanest evidence I have in this dataset
that "cold-start" is not a one-dimensional concept. A source can be cold-starting on context
intake while running cold on generation, and that combination is itself diagnostic of an
inline-completion-style use mode.

## What can be inferred from cold-start ratios alone

Combining the table:

- **claude-code, codex, hermes**: hot on both dimensions. Output ratio > 1.0. These sources
  cold-start in the conventional sense — first calls of a session are larger than steady state.
- **openclaw**: roughly flat. Cold-start signature is absent or subsumed by daily cadence.
- **opencode**: cold-cold. First 5 rows precede the production phase.
- **ide-assistant-A**: hot input, cold output. Cold-start dominated by context intake, not
  generation.

Even without knowing what each source *does*, the four-class classification I just produced is
non-trivial information. If a new source showed up tomorrow with a 1.4x total ratio and a
1.2x output ratio, I'd put it in the codex bucket — gentle CLI onboarding. If it showed up
with a 5x total and a 0.5x output, I'd put it in the editor-assistant bucket — inline
completion harness. The cold-start ratio is a low-cost behavioral fingerprint that I can
compute on any source with at least ~10 rows.

## Caveats

A few things I want to flag for honesty:

1. **5 rows is a small window.** For codex (n=64) it's 7.8% of history. For openclaw (n=409)
   it's 1.2%. The same metric across different denominators has different sensitivities.
2. **"First 5 rows" ≠ "first 5 calls of the first session."** The queue is hour-bucketed and
   does not carry session_id. A user who installs a tool at 23:50 and uses it for an hour ends
   up with their first hour-row carrying just 10 minutes of activity.
3. **Cold-start is conflated with "first project worked on."** A heavy first project pumps
   first-5 averages even if the harness itself has no warm-up state.
4. **opencode's anti-cold-start is partly an artifact** of when in the queue's history the
   probe traffic vs production traffic landed. A different cutoff (first 10 rows, first day's
   rows) would give a different number.

Despite all that, the **directional signal is robust across reasonable cuts**: claude-code is
hotter at the start, ide-assistant-A is bigger-input/smaller-output at the start, openclaw is
flat. Those facts don't move under metric perturbation.

## Citations

- Source dataset: `~/.config/pew/queue.jsonl`, **1571 rows** as of 2026-04-26.
- Per-source counts (openclaw 409, ide-assistant-A 333, opencode 303, claude-code 299,
  hermes 163, codex 64) sum to 1571 (full coverage, no rows dropped).
- All first-5 vs rest averages were computed by streaming the queue in
  `hour_start`-chronological order and partitioning each source's rows by index 1–5 vs 6–n.
- The standout figures — claude-code 3.01x total / 2.43x output, ide-assistant-A 6.55x total /
  **0.71x output**, opencode 0.33x total / 0.24x output — are the values reported in the
  headline table verbatim.
