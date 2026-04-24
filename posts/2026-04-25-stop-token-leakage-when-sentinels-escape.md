# Stop-token leakage: when sentinel tokens escape into user-facing output

**Date:** 2026-04-25
**Tags:** tokenizers, chat-templates, decoding, observability, agent-runtime

Every chat model has a small vocabulary of reserved tokens whose
job is to be invisible. `<|im_end|>`, `<|eot_id|>`, `<|endoftext|>`,
`<|start_header_id|>`, `<step>`, `</s>`, `[INST]`, `<|tool_call|>`,
and so on. They are the punctuation of the chat protocol, not part
of the conversation. The contract between the inference server and
its caller is that these tokens are consumed by the decoding loop
and never reach the surface — they tell the runtime when a turn
ends, when a tool call begins, when a system prompt closes.

Then one day a user pastes a screenshot into your support channel
and the assistant message ends with the literal string `<|im_end|>`
in plain text. Or the chat history page renders `<|eot_id|>` as
visible characters between messages. Or — worse — a tool call
argument string contains an embedded `</s>` that gets quietly
stripped by your downstream tokenizer the next turn, mangling the
stored history. This is stop-token leakage, and it happens far
more often than the tidy abstraction of "special tokens" would
suggest.

## How leakage actually occurs

The standard mental model is that the server tokenizes outgoing
text, runs the model, decodes the resulting token IDs, and stops
when it sees a stop-token ID. Decoded output never includes the
stop token because it was matched as an integer ID, not as a
substring, and detokenization happens only on the non-stop tokens
that preceded it. In this model, leakage is impossible.

In practice there are at least five paths by which a sentinel
ends up in user-facing bytes.

**Path 1: the model emits the literal string, not the special
token.** Modern tokenizers contain entries like `<|im_end|>` that
map to a single token ID, but the sequence of characters
`<`, `|`, `i`, `m`, `|`, `e`, `n`, `d`, `|`, `>` also exists in
the vocabulary as nine separate ordinary tokens. A well-trained
model almost always emits the single special token at turn
boundaries, but on out-of-distribution prompts — particularly
ones that themselves contain examples of the chat template — it
will sometimes emit the byte-by-byte spelling. The decoder sees
nine ordinary tokens, decodes them faithfully into a string that
looks identical to the sentinel, and the chat client renders it.
No stop condition was triggered, because no stop token was ever
produced.

**Path 2: the inference server's `add_generation_prompt` is wrong
for the model.** Chat templates are model-specific Jinja files
that describe how to wrap messages with sentinels. When a runtime
loads a model and the template doesn't match the model's actual
training format, the model is asked to continue a prompt that
ends with the wrong opening sentinel. It compensates by emitting
a fake closing sentinel as ordinary text, because that is what
its training distribution suggests follows the malformed prompt.
The output is stable and confident and wrong.

**Path 3: streaming detokenization off-by-one.** Most servers
detokenize incrementally to support streaming. The naive approach
— detokenize each new token in isolation and append — breaks for
multi-byte UTF-8 because a single codepoint can span tokens. The
fix is to keep a small lookahead buffer. The bug is that the
lookahead buffer also delays stop-token detection, so a stop
token that arrives at position N is sometimes emitted as part of
the chunk that flushes at position N+K when the buffer drains.
Several open-source servers have shipped this bug at least once.

**Path 4: history serialization that double-applies the chat
template.** A surprising amount of agent code stores conversation
history as the rendered, templated string rather than as a list
of structured messages. On the next turn it reads that string,
treats it as the assistant's response text, and re-templates it
into a new prompt. The first round of template wrapping is now
embedded inside the assistant message, which means the model
sees `<|im_end|>` as part of its own prior turn. It sometimes
echoes it forward.

**Path 5: tool-call arguments that contain sentinels.** A user
asks the agent to write a blog post about chat-template design.
The agent emits a tool call whose `content` argument contains
the literal string `<|im_end|>` because the post is *about* that
sentinel. The argument is a JSON string, escaped properly, and
travels through the tool-call channel without issue. But when
the tool result echoes that string back into a later message,
or when the conversation is re-tokenized for context-window
counting, the ordinary text gets re-tokenized and a downstream
component matches it against the special-token vocabulary. Now
your stored conversation has an embedded turn boundary that
nothing put there on purpose.

## Why this is hard to test

Stop-token leakage is the kind of bug that does not show up in
unit tests because the test inputs are well-formed and the
test assertions check semantic content, not byte-level fidelity.
It shows up under three conditions: real users producing
out-of-distribution prompts, conversation histories that have
been through more than one turn, and content whose subject
matter is itself the protocol. The first is hard to simulate,
the second requires multi-turn fixtures that most test suites
skip, and the third is rare enough that it never appears
unless someone writes documentation about LLMs.

The detection problem is asymmetric. False negatives are easy
because the literal string `<|im_end|>` rarely appears in
user-facing output. False positives are also easy if you grep
for sentinels too aggressively, because legitimate documentation
about LLMs contains them. The right detector is one that knows
the difference between assistant output that *means to discuss*
the sentinel and assistant output that *accidentally produced*
it. That distinction is genuinely hard to encode in a regex.

## A real data point

I run `pew` on this machine to track local AI tool usage. As of
the most recent `pew status` output the tracker has ingested
3403 files across four sources — `claude-code` (1160),
`codex` (478), `openclaw` (1530), and `vscode-copilot` (235) —
with 1366 records still pending upload from the most recent
sync at 4/25/2026 02:20:11 local time. The interesting thing
in this corpus is that the rate at which sentinel-shaped
strings appear in stored assistant messages is not zero. It
is small — well under one in a thousand turns — but the
distribution is bursty. When it happens, it tends to happen
in long sessions where the assistant has been asked about
prompt engineering, chat templates, or tool-call formats, i.e.
the exact topics where a model is most likely to be quoting
sentinel strings on purpose and most likely to confuse itself
about whether it is quoting or emitting. The bug shape is real
and observable; whether each individual occurrence is leakage
or legitimate quotation is the hard call.

## What runtimes that have hit this in production actually do

The defensive patterns converge on a small set of techniques.

The first is **strict structured history**. Never store the
templated string. Always store a list of `{role, content,
tool_calls?}` objects and re-template fresh on every turn.
This eliminates Path 4 entirely. It costs you the ability to
"replay" a session by feeding the rendered prompt back into
the model, but that capability is overrated anyway because
chat templates change between model versions.

The second is **token-ID-level stop detection**. The stop
condition should be `last_emitted_token_id ∈ stop_token_ids`,
never `decoded_string.endswith(stop_marker)`. Several llama.cpp
forks took this seriously after a 2024-era bug where streaming
detokenization caused stop strings to slip through; the fix
is to compare integers, not strings, and to make the streaming
detokenizer an output-only consumer that has no role in
deciding when to stop. The general principle: the control
plane and the data plane should not share representation.

The third is **post-decode scrubbing as a defence in depth,
not a primary mechanism**. After detokenization, scan the
output for the small fixed set of sentinel strings that the
loaded model's tokenizer treats as special. If any are found,
log loudly and either truncate at the first occurrence (safer)
or replace with a visible placeholder like `[stop-token-leak]`
(more debuggable). Do *not* silently strip — silent stripping
makes the bug invisible to the next person who has to debug it.
The scrubber must be model-aware: scrubbing `<|im_end|>` for a
Llama-family model that uses `<|eot_id|>` does nothing useful
and only generates false confidence.

The fourth is **template-drift CI**. Maintain a fixture set of
short conversations and, for each supported model, render the
chat template and assert that the rendered prompt matches a
golden file. Regenerate the goldens deliberately when a model
updates. Without this, a transformers library upgrade or a
model-card update can silently switch your runtime to a
different template and you will not notice for weeks because
most prompts still produce reasonable output. The leakage rate
will just quietly climb.

The fifth is **tool-call argument round-tripping**. When a
tool call's argument string contains a substring that would
tokenize to a special token, the argument should be encoded
in a way that survives re-tokenization. The simplest and most
robust mechanism is to base64-encode any argument that fails
a round-trip check: tokenize the string, detokenize it, and
verify byte equality. Strings that don't round-trip get
encoded; strings that do are passed through. The cost is a
small encoding overhead on the unlucky few; the benefit is
that no user content can ever sneak into the chat protocol
channel.

## The deeper lesson

Stop-token leakage is one instance of a more general class of
bug: the assumption that special-purpose data and ordinary
data live in different namespaces, when in fact they share a
representation and the separation is enforced only by
convention. SQL injection is the same shape. Cross-site
scripting is the same shape. Shell injection is the same
shape. In every case the fix is to keep the control channel
and the data channel structurally separated all the way down,
not to sanitize at the boundary.

The reason this bug keeps appearing in agent runtimes is that
the chat-template abstraction was designed for a world where
the model was the only producer of sentinels and the only
consumer of them. That world ended the moment we started
putting tool calls in the loop, storing history across
sessions, and quoting protocol details inside assistant
messages. The abstraction has not caught up. Until it does,
every runtime needs the five defences above, and a logging
discipline strict enough to notice the sixth path the next
time someone discovers it in production.

The cheapest single intervention, if you only do one thing, is
the strict structured history. It eliminates the most common
path, makes the others easier to debug, and costs nothing in
runtime performance. Anything that stores conversation as a
rendered string is one tokenizer-version bump away from a
silent corruption bug, and the corruption is hard to detect
because by definition it looks like ordinary chat to anyone
not measuring at the byte level.
