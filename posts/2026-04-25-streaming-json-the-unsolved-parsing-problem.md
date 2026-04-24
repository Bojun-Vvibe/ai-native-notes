# Streaming JSON: the unsolved parsing problem at the agent boundary

**Date:** 2026-04-25
**Tags:** streaming, parsing, tool-calls, ux, agent-runtime

When a model emits a tool call, what arrives on the wire is not a
JSON object. It is a sequence of byte-fragments that, *if you wait
long enough*, eventually concatenate into something a strict JSON
parser will accept. Every agent runtime in the wild has been forced
to confront this problem, and every one of them has solved it
differently — usually by accident, usually with at least one
embarrassing failure mode they have not yet documented.

This post is about what "parsing JSON" actually means at the
streaming boundary between an LLM provider and an agent runtime,
why none of the obvious approaches work, and what the survivors
have converged on.

## Why we even stream JSON in the first place

The temptation is to say: just buffer until the response is
complete, then parse. For non-tool text this is fine; for tool
calls it is operationally fatal in two cases.

**Case 1 — long tool arguments.** Some tools (write_file, apply_patch,
shell_command with a long script) take arguments whose JSON
serialization is several thousand tokens. At a typical 50–80
tokens/sec generation rate, that is 30–90 seconds of dead air if
you wait for the closing brace before showing anything. Operators
hate dead air. They reach for the kill switch.

**Case 2 — early validation.** If the tool name is going to be
`shell_command` and the working directory is going to be
`/etc`, you would like to know that *now*, not after the model has
finished spelling out the rest of the argument blob. The earlier
the runtime can pattern-match a destructive call, the earlier it
can pop a confirmation prompt or short-circuit. This is what every
"approval gate" is doing under the hood.

Both cases force the runtime to *do something useful with a
half-finished JSON document*. And that turns out to be hard in
ways the JSON spec was explicitly designed to make hard.

## The four shapes of partial JSON

Concretely, an in-flight tool-call payload at any tick can be in
exactly one of four states:

1. **Syntactically complete and parseable.** This is the rare
   happy path that exists only at the final tick. Real agents see
   it for maybe one buffer in a hundred.
2. **Syntactically incomplete but unambiguous.** The string
   `{"path": "/tmp/foo.txt", "content": "hello wo` has not seen
   its closing quote, brace, or comma, but a human (and a smart
   parser) can tell exactly where it is in the grammar.
3. **Syntactically incomplete and ambiguous.** The string
   `{"path": "/tmp/foo` could be mid-string, or could be the
   parser's view *just before* a complete `"/tmp/foo"` arrives —
   you cannot tell without lookahead. Worse: `{"x": 1` could be
   mid-number (next char might be `2`) or complete-and-pending-
   comma. Numbers without a delimiter are the worst offenders.
4. **Provider-corrupted.** The model emits a stray newline inside
   a string, or two consecutive value fragments without a comma,
   or a UTF-8 boundary cuts a multibyte character in half. This
   one bites everyone eventually.

Strict `JSON.parse` rejects all of (2), (3), and (4). So no agent
runtime in production calls `JSON.parse` on a streaming buffer.
What they call instead is the interesting question.

## Strategy A: incremental tokenizer, no parser

The simplest approach: write a hand-rolled tokenizer that
recognizes JSON's terminal symbols (`{`, `}`, `[`, `]`, `:`, `,`,
`"`, escape sequences, number-start chars) but does *not* attempt
to validate grammar. Maintain a small state machine: in_string,
in_escape, in_number, expecting_key, expecting_value, depth. Emit
high-level events: `key_started`, `key_finished("path")`,
`value_started("string")`, `value_chunk("/tmp/fo")`,
`value_chunk("o.txt")`, `value_finished`, `object_finished`.

This is what most TUI agents that show "writing to /tmp/foo.txt
…" updates use under the hood. It handles state (1), (2), and (3)
cleanly because it never asserts grammatical completeness — it
only asserts "I have a definite key now" or "I have at least N
chars of this value." Numbers it just buffers and emits at the
next non-number char.

The cost is real: it is a parser you maintain forever. JSON has
escape sequences (`\uXXXX`, `\n`, `\t`, surrogate pairs) and the
moment you get one of them subtly wrong, an agent that
write_file's a UTF-16-surrogate-pair-containing emoji into
`/tmp/foo.txt` corrupts the file silently. Several of the OSS
agents have a regression-test fixture file just for this — a
single 1.4 KB JSON document containing every escape edge case
they have been bitten by.

## Strategy B: best-effort JSON repair, then strict parse

The other school: at every tick, take the buffer-so-far, pass it
through a **JSON repair** pass that closes open strings, balances
braces and brackets, deletes trailing commas, and invents a
plausible value for any half-typed number; then run strict
`JSON.parse` on the repaired output and hand the result to the
UI. On the next tick, throw the repaired version away and repair
the (now longer) raw buffer fresh.

This is genuinely surprising at first contact. The most popular
implementation strategy is roughly:

```js
function repair(buf) {
  let s = buf;
  // Strip incomplete escape at the very end.
  if (s.endsWith("\\")) s = s.slice(0, -1);
  // If we are inside a string, close it.
  const depth = scanDepth(s); // returns {strings, braces, brackets}
  if (depth.inString) s += '"';
  // Close any open arrays / objects in reverse open order.
  for (const ch of depth.openStack.reverse()) {
    s += ch === "{" ? "}" : "]";
  }
  // Repair dangling key:  {"path":  → {"path":null}
  s = s.replace(/"\s*:\s*([}\]])/g, '":null$1');
  // Repair dangling comma: ,}  →  }
  s = s.replace(/,\s*([}\]])/g, "$1");
  return s;
}
```

The advantage over Strategy A: you reuse the platform's strict
parser, which means escape handling, surrogate pairs, and Unicode
normalization are all somebody else's problem. The disadvantage:
**every tick you discard the previous parse**. For a 50 KB
write_file argument generating at 80 tokens/sec, you re-parse the
entire buffer maybe 200 times. Most runtimes that use this strategy
notice the CPU cost only after a user reports their fan spinning
during a long file edit, and then quietly add a "re-parse only on
structural-character arrival" optimisation — which works because
nothing visible changes when the model is mid-string anyway.

The other disadvantage is that "plausible values for half-typed
numbers" is a place where two reasonable engineers will disagree.
Is the in-flight `42` a complete value (followed by an as-yet-
unwritten comma) or a prefix of `4200000`? Strategy B has to
guess. Strategy A defers; the UI just doesn't show the value
until the delimiter arrives. For numbers used as IDs or budgets
this matters.

## Strategy C: schema-guided streaming

The third school says: forget generic JSON. We *know* the schema
of the tool call before it starts arriving — the tool definition
is in the system prompt, and the model is supposed to follow it.
So: keep a parser pinned to the *expected* shape, and use it to
disambiguate. If the schema says `arguments.path: string`, then
the moment the parser sees `"path":` it knows the next non-
whitespace char must open a string and everything until the
matching unescaped `"` is the path. Numbers without a closing
delimiter become unambiguous because the schema says where the
field ends.

This is what the most disciplined agent runtimes are moving
toward. It composes naturally with **structured-output mode**
(strict JSON-schema-conformant generation), where the provider
itself promises to emit only schema-conformant tokens. When that
promise holds, schema-guided streaming degenerates to "just slot
the bytes into the right field as they arrive" — no repair, no
guessing, no re-parsing.

The catch: not every provider supports strict structured output
for tool-call arguments yet, and even where they do, the runtime
has to fall back gracefully when the schema cannot be enforced
(because the operator authored a tool with a free-form
`additionalProperties: true`, say). Strategy C in practice means
"Strategy C if the schema is closed, Strategy A or B otherwise,"
which means the runtime ends up implementing all three.

## What 2026 actually looks like

A real data point: in week 17 of 2026, the digest covering
2026-04-20 through 2026-04-26 records 186 PRs merged against
`openai/codex` alone, with `rust-v0.123.0` shipping a permission-
profile + hooks-in-config arc and `rust-v0.122.0` through
`rust-v0.125.0-alpha.1` cutting in a roughly 3-day cadence. The
TUI side, `charmbracelet/crush` v0.62.0, made its headline
feature a **98% reduction in tool-description tokens by default**
(PR #2679) — a change that is materially about *how much JSON the
agent has to parse on every turn*, not just how much the model has
to read. PR #2671 in the same release added a `fetch`/`view`
truncation cap at 100 KB; that cap exists because their streaming
parser was eating CPU on multi-megabyte tool results.

On the routing side, `BerriAI/litellm` v1.83.11-nightly through
the v1.83.13-nightly + v1.83.7-stable.patch.1 cosign-signed pair
(both at commit `0112e53`) shipped adaptive routing in PR #26049,
per-team-member spend in #26195, and a Fireworks-streaming bug
report at issue #26326 about `<think>` tags leaking through. That
last one is the *exact* shape of failure mode this post is about:
a streaming parser saw `<think>` open mid-content and didn't know
whether it was reasoning content (skip) or assistant content
(keep). The provider's wire format and the runtime's parser had
drifted, and the only symptom was tags leaking to the user.

`anomalyco/opencode` v1.14.21 → v1.14.22 (released 2026-04-23
05:45Z and the desktop session-state fix in PR #24016) hit a
related problem: an Effect Schema migration sprint merged 9 PRs
back-to-back (#23244, #24005, #24019, #24024, #24027, #24029,
#24040, #24054, #24056) porting the entire tool framework and 18
built-in tools onto a single schema runtime — explicitly so that
*the same schema description that the model sees can also drive
the streaming parser*. That is Strategy C in production, named.

## The bug shapes you actually see

After enough time staring at production traces, the failure modes
cluster:

- **The half-emoji**: a multi-byte UTF-8 sequence is split across
  two SSE chunks. Strategy A handles it if the tokenizer buffers
  bytes-not-chars; Strategy B handles it if the repair function
  refuses to repair mid-codepoint; Strategy C handles it because
  the schema-aware parser knows the field is a string and just
  buffers until the next valid boundary.
- **The runaway string**: the model forgets a closing quote and
  generates 8 KB of "path content" before the runtime notices the
  schema says `path` is a relative-style string and probably
  shouldn't contain a newline. Some runtimes hard-cap individual
  string fields at, say, 32 KB and abort the tool call early; the
  truncation cap in Crush #2671 is the same idea applied to tool
  *output*.
- **The argument that never closes**: the model generates a
  perfectly well-formed object and then keeps going, emitting a
  second sibling key after what the parser thought was the end.
  This is rare but devastating: Strategy B will repair the buffer
  at every tick to a *different* parsed object as the model
  continues. Some runtimes pin the first complete-looking parse
  and discard subsequent edits; others treat the late edits as
  authoritative and stream a corrected UI. There is no consensus.
- **The double escape**: `\\n` vs `\n`. The model is happy to
  emit either. The runtime that displays the in-flight argument
  in a "writing to file …" preview must agree with the runtime
  that eventually writes the file, or the user sees `foo\nbar` in
  the preview and `foo<newline>bar` on disk. This is a UX bug
  that is technically not a parser bug but always gets blamed on
  the parser.

## A pragmatic recommendation

If you are writing the parser today, the cheapest defensible
choice is **Strategy A for the UI side** (an event-emitting
tokenizer that never lies about completeness, only ever about
"how much have I confirmed so far") and **Strategy B as a
periodic snapshot** for any consumer that wants a parsed object
(approval prompts, security gates, debug logs). Reserve Strategy
C for the path where you know the schema is closed and the
provider supports strict mode; treat it as an optimisation, not a
foundation. Always include the half-emoji and runaway-string
fixtures in your regression suite. Always cap individual string
fields at a number you wrote down in a config file, not a number
you decided was fine three years ago.

JSON is supposed to be the boring part of an agent. It is not.
It is a load-bearing parser sitting on a wire designed for HTTP
form posts, asked to produce real-time UI updates from a token-by-
token stream emitted by a model that occasionally forgets quotes.
The "unsolved" in this post's title is not hyperbole — every
runtime that ships agents has its own incomplete answer and its
own private list of bugs it has not yet found.
