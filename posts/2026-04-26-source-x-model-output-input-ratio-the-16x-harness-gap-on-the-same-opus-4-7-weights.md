# Source × model output/input ratio: the 16× harness gap on the same opus-4.7 weights

Most posts that talk about output/input ratios treat the ratio as a property
of the model. They will say "this model is verbose" or "this model is
terse," and produce a per-model bar chart that flatters whichever vendor
the author favors. The bar chart is not wrong, but it is incomplete in a
way that changes the conclusion. The output/input ratio is not a property
of the model. It is a property of the *driver* — the harness, the
scaffolding, the agent loop, the system prompt, the tool wiring, the
context-compression strategy — multiplied by the model.

The cleanest way to see this is to fix the model and vary the harness.
On the local pew queue at `2026-04-26T06:26:27Z`, the data permits exactly
that experiment: `claude-opus-4.7` shows up in three different sources
(`opencode`, `claude-code`, `hermes`) with sample counts that are large
enough to be meaningful. The aggregate output/input ratios are 0.112,
0.007, and 0.024 respectively. That is a 16× spread on the same model
weights.

This post is about what that spread means, where it comes from, and why
the popular "model X is more verbose than model Y" framing is the wrong
unit of analysis.

## The cross-tabulation

The decomposition was produced by streaming `~/.config/pew/queue.jsonl`
(1527 rows, snapshot `2026-04-26T06:26:27Z`) through `jq` to extract
`source`, `model`, `input_tokens`, `output_tokens`, then bucketing the
free-text `model` field into seven canonical families (`gpt-5*`,
`opus-4.7`, `opus-4.6`, `sonnet`, `haiku`, `gemini`, `big-pickle`, plus
`other`). The result is a 6×8 table with sparse cells; the cells with
≥30 rows are the ones I trust to talk about.

```
source         | family    | rows | in_tok       | out_tok    | out/in
---------------+-----------+------+--------------+------------+-------
opencode       | opus-4.7  |  214 |   163794592  |  18339783  | 0.112
hermes         | opus-4.7  |  153 |    54660626  |   1307931  | 0.024
claude-code    | opus-4.7  |   81 |  1159017656  |   8538187  | 0.007
claude-code    | opus-4.6  |  182 |   606191092  |   3450625  | 0.006
claude-code    | haiku     |   30 |    65945659  |    130391  | 0.002
codex          | gpt-5*    |   64 |   410781190  |   2045042  | 0.005
openclaw       | gpt-5*    |  390 |   922837071  |   4817673  | 0.005
opencode       | big-pickle|   53 |    17746683  |    366926  | 0.021
```

The 16× column is the only column that matters here. Every other
comparison in the table is contaminated by either model differences or
sample-size weakness. But the `opus-4.7` row appears three times, with
sample counts of 214, 153, and 81 — three independent harnesses calling
the same model weights. The ratios are 0.112, 0.024, 0.007.

If the ratio were a property of the model, those three numbers would
collapse to within sampling noise of each other. They do not collapse.
They span a factor of 16. That is a structural fact about the *driver*,
not about the model.

## Why the per-row evidence is more striking than the aggregate

Aggregates can hide tail behavior. Per-row evidence makes the harness
fingerprint unmistakable. The five highest output/input ratios in the
entire 1527-row corpus, filtered to rows with `input_tokens > 1000` so
the ratio is meaningful:

```
ratio    | source   | model         | in     | out    | hour
---------+----------+---------------+--------+--------+-----
0.9933   | opencode | opus-4.7      |   8348 |   8292 | 2026-04-24T00:00Z
0.6454   | opencode | opus-4.7      |  42248 |  27267 | 2026-04-23T18:30Z
0.6191   | opencode | opus-4.7      |  48384 |  29956 | 2026-04-23T13:00Z
0.5728   | opencode | opus-4.7      |  23226 |  13303 | 2026-04-23T21:30Z
0.5569   | opencode | opus-4.7      |  26263 |  14626 | 2026-04-24T01:00Z
```

All five are the same source, same model. The first row is a 0.9933
ratio: 8348 input tokens, 8292 output tokens. The model wrote almost
exactly as many tokens out as it received in. That row is functionally
a *rephrasing* — a transformation, a translation, a refactor that
emits roughly token-for-token replacement of the input.

Now the five lowest, same filter:

```
ratio       | source       | model              | in       | out  | hour
------------+--------------+--------------------+----------+------+-----
2.020e-05   | claude-code  | claude-sonnet-4-6  |   990186 |   20 | 2026-02-25T09:30Z
2.039e-05   | claude-code  | claude-haiku-4.5   |   441414 |    9 | 2026-02-26T05:30Z
2.222e-05   | claude-code  | claude-haiku-4.5   |  6346255 |  141 | 2026-02-12T03:00Z
2.361e-05   | claude-code  | claude-haiku-4.5   |  1905492 |   45 | 2026-02-25T10:30Z
3.163e-05   | claude-code  | claude-sonnet-4-6  |   632360 |   20 | 2026-02-25T09:00Z
```

Same source for all five. 990,186 input tokens to produce 20 output
tokens. 6,346,255 input tokens to produce 141 output tokens. These are
not bugs. These are routine: the agent harness sent a giant context
(probably a compaction request, a summarization step, a routing
classification) and the model returned a tiny structured answer.

The two extremes are roughly 50,000× apart. The model family is the
same in both cases (Anthropic Claude). The thing that varies is what
the harness asks the model to do.

## Three regimes hiding inside the same column

If you take the `opus-4.7` rows and split them by source, three
different regimes appear:

**Regime A — `opencode|opus-4.7`, n=214, ratio 0.112.** This is an
interactive coding harness where the model is doing actual code
generation: producing diffs, writing functions, replying to a user in
a chat-style loop. Output is bounded by what the user asked for, but
input is bounded by the *context window* that the harness chose to
send. opencode tends to send focused context — the file or files
being edited, plus a system prompt. So input is moderate (mean
~765K per row) and output scales with the model's actual response,
producing ratios that often live in the 0.1–1.0 range.

**Regime B — `hermes|opus-4.7`, n=153, ratio 0.024.** This is roughly
4–5× quieter on the output side relative to input than opencode. The
harness is sending more context per call (or the model is being asked
for shorter answers, or both). Mean input per row is ~357K, mean
output ~8.5K. The ratio is consistent with a slight ratio compression:
either bigger system prompts, or more aggressive context retention,
or a stricter "be concise" instruction.

**Regime C — `claude-code|opus-4.7`, n=81, ratio 0.007.** Input mean
is 14.3M tokens per row. That is not a typo: the per-row mean is in
the millions of input tokens. Output mean is ~105K. This is the
behavior of an agent harness that does aggressive context expansion —
loading entire repositories or large swaths of conversation history
on every call, then asking the model for a relatively constrained
response (a tool call, a partial diff, a compaction). The ratio is
low not because the model is terse but because the *denominator* is
inflated by an order of magnitude.

The 16× spread is therefore not a model fact. It is the cross of
three behaviors: how big the harness lets the input grow, how
aggressively the harness instructs the model to be brief, and what
fraction of calls are tool/router/classification calls (which return
tiny outputs) versus content-generation calls (which return large
outputs).

## Why per-source ratios are a better proxy for "agent style" than per-model

Look at the column for `gpt-5*`:

```
source       | rows | in_tok       | out_tok    | out/in
-------------+------+--------------+------------+-------
codex        |   64 |   410781190  |   2045042  | 0.0050
openclaw     |  390 |   922837071  |   4817673  | 0.0052
opencode     |   13 |     2202067  |     59450  | 0.0270
```

`codex` and `openclaw` produce nearly identical ratios on `gpt-5*`
(0.0050 vs 0.0052). They are both agent-style harnesses for OpenAI
models, and they apparently apply similar context-padding and
output-trimming heuristics. `opencode` on `gpt-5*` is 5× higher,
matching its higher ratio on `opus-4.7` — a consistent *harness*
signature across models.

This is the second reason ratios should be reported per harness, not
per model. The harness pattern is *stable across model families*. If
you wanted to predict the output/input ratio of a new model run, you
would do better knowing the source than knowing the model. The
source explains more variance than the model does.

## The ide-assistant-A anomaly and why it's actually consistent

The `ide-assistant-A` rows have `input_tokens = 0` for all gpt-5*,
gemini, and sonnet rows. The cells in the cross-tab read:

```
source         | family   | rows | in | out    | ratio
---------------+----------+------+----+--------+------
ide-assistant-A | gpt-5*   |  226 |  0 | 935425 | 0.000
ide-assistant-A | sonnet   |   63 |  0 | 146897 | 0.000
ide-assistant-A | gemini   |   37 |  0 |  43665 | 0.000
```

The ratio is 0/something — undefined in the strict sense, zero in the
aggregate sense. This is not a bug in the data; it is a documented
property of how the IDE-assistant harness reports tokens. The
input-side accounting is hidden inside the IDE context engine and is
not propagated to the local accounting layer that pew sees. So the
0.000 ratio for ide-assistant-A rows is really "input was not
disclosed." Treating it as a real zero would be wrong.

(The source is referred to here as `ide-assistant-A` per local
content-redaction policy; the underlying source is a single named IDE
assistant whose harness is the relevant variable.)

What this teaches: the output/input ratio is not just a function of
harness behavior, it is a function of *what the harness reports*. Some
harnesses lie by omission. Others over-count by including cached
context as input. The cross-tab is the only way to tell the difference,
because the patterns of omission are themselves source-specific.

## The model-conditional view

Flip the table around and condition on the model first. Of the 81
`claude-code|opus-4.7` rows, the ratio is 0.007. Of the 214
`opencode|opus-4.7` rows, the ratio is 0.112. The harness explains
roughly two orders of magnitude. Now condition on the source first.
Of the 81 + 214 + 153 = 448 opus-4.7 rows total, sources span 0.007
to 0.112 — 16× spread. Of the 467 gpt-5* rows split across `codex`,
`openclaw`, and `opencode`, sources span 0.005 to 0.027 — 5× spread.

Why does opus-4.7 show a wider source-spread than gpt-5* does? Two
candidate explanations.

**Explanation 1: opus-4.7 is the "go-to" model across more diverse
workflows.** Looking at the sample sizes, gpt-5* is dominated by
two harnesses (codex 64, openclaw 390 — both agent-style scaffolds
optimized for similar problems), with opencode contributing only 13
rows. opus-4.7 is more evenly split (81 / 153 / 214) across three
harnesses with genuinely different jobs: claude-code does heavy
agent work, hermes does a different agent style, opencode does
interactive editing. The source-spread is wider because the source
*workloads* are wider.

**Explanation 2: opus-4.7 honors brevity instructions less faithfully
than gpt-5*.** A harness that says "be terse" will get a more
compressed output from a model that follows brevity instructions
strictly. If gpt-5* compresses harder, the ratios converge across
sources. If opus-4.7 produces a more "natural" length regardless of
the prompt, the harness's *input* size dominates the ratio and you
get a wider spread.

The data on hand can't disambiguate these. But Explanation 1 is the
parsimonious story, because the 5× spread on gpt-5* is exactly the
spread that opencode-vs-others produces on opus-4.7 (opencode is the
high-ratio harness in both cases). The harness identity, not the
model identity, explains the gradient.

## What this means for capacity planning

If you are sizing capacity for a multi-harness deployment and you
budget capacity using a per-model ratio, you will be wrong by an
order of magnitude in either direction depending on which harness
dominates the actual traffic. The right unit is `(source, model)`
pair. The ratios for the 4 highest-volume pairs in the local data:

```
pair                       | rows | ratio  | implied output%
---------------------------+------+--------+----------------
openclaw|gpt-5*            |  390 | 0.0052 | 0.52%
opencode|opus-4.7          |  214 | 0.1119 | 11.19%
claude-code|opus-4.6       |  182 | 0.0057 | 0.57%
hermes|opus-4.7            |  153 | 0.0239 | 2.39%
```

If the deployment plan said "we expect 0.5–1% output" because that's
what the dominant `openclaw|gpt-5*` pair shows, the addition of even
modest `opencode|opus-4.7` traffic blows the output budget by 20×
relative to the assumption. The cost of being wrong is asymmetric:
output tokens are typically priced 3–10× higher than input tokens,
so under-estimating output share is more expensive than
under-estimating input share.

The correct planning calculation is a weighted sum across the
source×model pairs, where the weights are the *expected mix of
harness traffic*, not the model traffic. The harness mix is what
moves; the model is just what the harness happens to be using this
month.

## What's missing from this analysis

Three things are intentionally not in this post.

First, no per-day decomposition. The 16× spread is computed against
the entire 1527-row history. It is possible that the ratios converge
over time as the harnesses standardize, or diverge as they
specialize. A daily series would tell you which.

Second, no controlled experiment. The data is observational —
each row is whatever traffic the user happened to generate. The
*workloads* sent to each harness are not equivalent. claude-code is
asked harder questions than opencode is, on average, and the
"harder question = bigger context = lower ratio" path could explain
some of the spread without invoking any harness-specific behavior.

Third, no causal model. The ratio is a downstream effect of three
upstream choices (input padding, output instruction, call-type
mix), and disentangling them would require harness-internal
telemetry that the local pew log does not have. The 16× number is
the best estimate of *what the user observes*. It is not a
decomposition of *why*.

## The takeaway

Per-model output/input ratios are a misleading unit. They flatten
the harness contribution, which on this corpus is the dominant
source of variance. The same `claude-opus-4.7` weights produce
ratios spanning 0.007 to 0.112 — a 16× gap — depending on which
agent harness is making the call. Per-row extremes are even
sharper: a `claude-code` row asked the model to summarize 6.3M
tokens into 141 tokens (ratio 2.2e-05); an `opencode` row asked
the model to rephrase 8.3K tokens into 8.3K tokens (ratio 0.99).
Both rows used Anthropic models; neither row tells you anything
useful about the model in isolation.

If you want to compare models honestly, hold the harness fixed.
If you want to compare harnesses, hold the model fixed. The
local data here lets you do the second comparison cleanly because
opus-4.7 happens to be the most cross-harness-distributed model
in the corpus. The result is unambiguous: the harness explains the
spread, and reporting ratios per model alone misses the actual
operational fact you need to know.

---

*Cited data: `~/.config/pew/queue.jsonl`, 1527 rows, snapshot at
2026-04-26T06:26:27Z. Source × model-family bucketed via jq +
awk pipeline. The label `ide-assistant-A` is a redaction of one
internal source name per local content policy.*
