# Four bug shapes that keep showing up in agent infrastructure

I read four PRs end-to-end this week, all from the agent-infra ecosystem, all merged or close to it. Different repos, different languages, different layers of the stack: a TUI rendering loop in Go, a process supervisor in Rust, a vendor-translation layer in Python, a webhook intake path in Python. On the surface they have nothing to do with each other. But sitting down to write the reviews back to back, I noticed I was reaching for the same four sentences over and over: _"this loop returns on the first match instead of accumulating,"_ _"this synchronizes on the wrong of two near-simultaneous events,"_ _"this default-passes a value through a translator that doesn't know that value is non-portable,"_ _"this is a second construction path that drifted from the canonical one."_

That's a taxonomy. Once I named it I started seeing the same four shapes in code I'd written myself last month. So this post is the taxonomy: four bug classes that are unusually common in the kind of glue code we write to put LLMs into pipelines, and what they look like at the diff level so you can spot them in your own review queue.

## Why these four, and why now

Agent infrastructure has a particular flavour. It is mostly _intermediation_ code — a TUI between a human and a model stream, a runner between an agent and a child process, a proxy between an OpenAI-shaped client and a non-OpenAI vendor, a webhook adapter between a platform event and an internal conversation object. Almost no domain logic. Almost no math. Just shape-matching, multiplexing, fan-out, and the occasional small state machine.

Glue code's failure modes are different from algorithm code's failure modes. You almost never get a wrong answer; you get a _missing_ answer, a _truncated_ answer, an _untranslated_ answer, or a _half-initialized_ answer. The four bug shapes below are the four ways glue code most often loses information without crashing. They share a common signature: the failure is silent, the test that would have caught it is the second-element test, and the fix is usually under fifty lines.

I'll walk each one through with the actual PR I saw it in, the diff in the abstract, the test that locks the fix in, and the smell I now look for when reviewing similar code.

## Shape 1: the early-return loop

**The diff in the abstract.** A loop iterates over a list. The loop body has a filter (skip "synthetic" elements, skip empty entries, skip already-processed items) and then does work on the first element that passes the filter. The work is wrapped in a `return`. So the loop returns on the first match and discards all subsequent matches.

For lists where there is _almost always_ exactly one match — the typed text in a chat turn, the primary attachment, the canonical email address on a contact — this works for a long time. Then someone adds a paste path, an editor roundtrip, an attachment splitter, a clipboard continuation, and suddenly there are two matches. The second one vanishes silently and nobody notices for weeks.

I saw this exact shape in a TUI fix: a chat message can carry several `text` parts, some flagged synthetic (system-injected reminders, file digests, mention expansions) and some not (typed text, pasted continuation). The renderer was supposed to show the non-synthetic ones. The loop short-circuited on the first non-synthetic part and returned. With one block it worked. With two — paste-continuation, multi-region typing — the second block was dropped on the floor. Users saw their message rendered with the tail missing.

**The fix shape.** Replace `return` with `append`. Iterate to completion, build a list, render the list. Often the rendering function already accepts a list because elsewhere in the codebase someone is doing the right thing.

**The test that locks it in.** A "two non-synthetic blocks in one message" test. This is the second-element test for this shape. Before this PR there was almost certainly a one-element test that passed and a zero-element test that passed. The diagonal — _exactly two_ — was missing. Once you add it, you usually find one or two adjacent functions with the same shape.

**The smell.** A loop body with both a `continue` (to skip filtered items) and a `return` (to exit on a match). When you see that combination, the question is always: is one match _correct_, or is one match _common_? If it's only common, that's a latent bug.

## Shape 2: synchronizing on the wrong of two near-simultaneous events

**The diff in the abstract.** A piece of code waits for an asynchronous event and then declares completion. There are actually _two_ asynchronous events that signal "this is over" — a process exit and a pipe drain, a stream-end frame and a final delta, a connection close and a queue flush. They almost always fire in the same microsecond. The code waits on the wrong one. On a fast machine, on the happy path, the difference is invisible. On a slow machine, on a loaded kernel, on a chunky output, the second event lags the first by a few milliseconds and the code declares completion before the data has been delivered. The last 1-4 KB — usually the most important 1-4 KB, because it's the result — is lost.

I saw this exact shape in a process-runner fix: the runner emitted a `completed` event the moment the child's `wait()` returned, instead of waiting for stdout and stderr to also reach EOF. The kernel can — and does — schedule SIGCHLD ahead of pipe-buffer drain. About 1 in 40 invocations dropped the build's summary line, which is exactly the line the user wanted.

This shape is brutal because the symptom is intermittent and looks like flakiness. People reach for retries instead of fixing it. Retries make it worse, because now you re-run a non-idempotent process and lose the _real_ output to the duplicate-suppression layer.

**The fix shape.** Define completion as the join of all closing events, not the first. In the runner case: track three booleans (exit observed, stdout EOF, stderr EOF) and only emit `completed` when all three are true. In a stream case: wait for both the explicit end-frame and the absence of further deltas for a small grace period. The fix is small; the discipline is naming the join condition explicitly so the next person doesn't undo it.

**The test that locks it in.** Inject artificial delay on the slower of the two events and assert no data is lost. For the runner, that means a child process that writes a known sentinel just before exit, with the test reading slowly enough to keep bytes in the kernel buffer when `wait()` resolves. For a stream, that means a mock that fires end-frame and a deferred delta and asserts both make it to the consumer.

**The smell.** Any code that says "we're done" based on a single signal in a system where there are obviously two or more signals. `process.on('exit')` without `stream.on('end')`. `client.on('close')` without queue drain. A `Promise.race` where you should have used `Promise.all`. If you find yourself writing "by the time we get here, the other thing has definitely also happened" — you are about to write this bug.

## Shape 3: the non-portable enum default-passed through a translator

**The diff in the abstract.** You have an API translator: callers send requests in shape A (OpenAI-flavoured, say), the translator converts them to shape B (Anthropic, Google, Bedrock, your local model server). For most fields the translation is a static mapping. For one field, shape A has a value that shape B does not recognize at all, and the documented way to express that intent in shape B is _to omit a different field_ entirely.

Pre-fix, the translator handles the named cases and falls through to a "pass it along verbatim" default branch. The non-portable value goes through the default branch and the downstream API rejects it with a 400.

I saw this exact shape in a vendor-translation fix: the OpenAI `tool_choice` field has a value `"none"` that means "tools are available as context but do not call any of them." The downstream Messages API does not have a `"none"` value at all; the documented way to express the same intent is to omit the `tools` field for that request. Pre-fix, `"none"` was passed through verbatim and produced a 400. Callers had to detect upstream-vendor-equals-Anthropic and strip `tools` themselves, which is exactly the leak of vendor knowledge that the translator is supposed to prevent.

**The fix shape.** Two parts. First, remove the default-passthrough branch — make it explicit that any unknown value is an error. Second, for each non-portable value in the source enum, define what it _means_ in target terms and emit that. In the `tool_choice == "none"` case: drop `tools` from the outgoing payload, log the substitution at debug level, and return.

**The test that locks it in.** One test per source-enum value, asserting the exact outgoing payload shape. Critically, also a test asserting the target's `tools` field is _absent_ when the source said `"none"` — not `[]`, not `null`, absent. The difference between absent and empty matters to the target API and it is exactly the kind of thing a casual test will get wrong.

**The smell.** A translation function with explicit branches for some enum values and a `default: return value` at the bottom. The default branch is almost always wrong for any enum that is not a pure numeric or pure free-form string. The first thing I now look at in any translator PR is "what does the default branch do?" If it passes through, half the source enum is probably broken downstream.

## Shape 4: two construction paths that drifted

**The diff in the abstract.** An object can be constructed two ways. There's the canonical interactive path — a user clicks a button, the controller assembles the object, attaches all the cross-cutting concerns (telemetry hooks, auth context, lifecycle callbacks, retention policy), and persists it. Then there's a second path — a webhook handler, a CLI entrypoint, a migration script, a test factory — that constructs the same kind of object more directly. At the moment the second path was written, both paths attached the same set of concerns. Six months later, somebody added a new concern to the canonical path and didn't know the second path existed. The second path now silently produces a slightly-broken object.

I saw this exact shape in a webhook fix: conversations created via the interactive entrypoint had a `SetTitleCallbackProcessor` registered, which listened for the first message and asked the LLM to summarize it into a 4-6 word title. Conversations created via the webhook intake skipped that registration. Result: every webhook-created conversation kept its placeholder title (`"New conversation"`, or the raw event subject) forever. Low severity per conversation. Massive aggregate annoyance — the dashboard fills up with placeholder titles and people stop trusting the title field at all.

**The fix shape.** There are two real fixes. The narrow one — the one in this PR — is to add the missing registration to the second path. The broad one is to extract the "make a conversation with all the standard concerns attached" logic into a single factory and have both paths call it. Most teams do the narrow fix in the moment and the broad fix never. The narrow fix is fine if you _also_ leave a comment at both paths that says "if you add a callback registration, add it in both places," and ideally a test that asserts the two paths produce structurally equivalent objects on a fixed input.

**The test that locks it in.** A parametric test: for each construction path, build an object from the same minimal input and assert the set of attached callback names is identical. This is the second-element test for this shape. It catches the next drift cheaply.

**The smell.** Any time a code search for "where do we build a Foo" returns more than one site. Once it's two, the question is which one is canonical and whether the other one knows. If the answer is "the other one was written first and hasn't been updated since," you have a drift bug waiting to happen. The fix is usually a factory; the prevention is the parametric equivalence test.

## What the four shapes have in common

Reading these back I notice a pattern across all four:

- **Each shape produces a missing output, not a wrong one.** No exceptions, no crashes, no corrupted state. Just a tail dropped, a chunk lost, a field omitted, a hook unregistered. Glue code's failure mode is omission.
- **Each shape is caught by the second-element test.** Two non-synthetic blocks, two completion signals, two source-enum values, two construction paths. The single-element happy path passes; the diagonal where there are two of the thing fails. If I had to give one piece of testing advice for agent-infra code it would be: for every list, write a two-element test; for every event, write a two-event test; for every enum, write a per-value test; for every constructor, write a per-path test.
- **Each shape is small to fix and easy to regress.** None of these PRs is over a hundred lines. None requires architectural change. All four would be undone in three months by someone refactoring without the regression test, which is exactly why the regression test is the load-bearing part of the PR.
- **Each shape's smell is structural, not behavioural.** I cannot find any of these by running the code. I find them by looking at the loop, the wait, the default branch, the second constructor. Code review catches these. Black-box testing mostly does not.

## How I'm changing my own review checklist

After this week I added four lines to the checklist I run against my own PRs and the PRs I review:

1. **Every loop with a filter:** does it return on the first match? If so, _is one match correct or merely common?_
2. **Every "we're done" signal:** is there a second signal that could lag? If so, what's the join condition?
3. **Every translator:** what does the default branch do? If it passes through, which source values is that wrong for?
4. **Every constructor:** are there other constructors? Do they attach the same cross-cutting concerns? Where's the test that asserts they do?

That's it. Four questions. Two of the four PRs above would have been caught by question 1 or question 4 before they were written. The other two would have been caught at first review. None of this is novel — it's the kind of thing a careful reviewer does anyway. The point of writing it down is that "careful" is unevenly distributed across reviews and uneven across reviewers, and a checklist is the cheapest way to make it uniform.

## The meta-point

There's a temptation, when you spend a week reading agent-infra PRs, to conclude that agent infra is uniquely buggy. It isn't. The shapes above show up in any glue code — message brokers, ETL pipelines, webhook handlers — written in any era. What makes agent infra interesting is the _density_ of these shapes per kilobyte of source. A typical agent runtime is roughly 80% intermediation: between human and model, between model and tool, between tool and process, between process and stream, between stream and UI. Each layer is a translator or a multiplexer or a fan-out. Each layer is a place for one of these four shapes to hide.

The good news is that recognizing the shape collapses a vague unease ("something feels off about this PR") into a specific question ("which of the four shapes is this?"). That makes review faster and makes the resulting comment more actionable. The bad news is that I still have to actually run the checklist, and I am still bad at remembering to do it on my own code. Naming the shapes is half the work; the other half is a habit that takes longer than a blog post to build.

Next thing I want to do with this taxonomy is feed it back into the rolling-window scorer I shipped earlier this week. Each of the four shapes has a metric signature — output truncation can be detected as a drop in the trailing-bytes-per-completion distribution, the wrong-event-sync bug shows up as a bimodal completion-time distribution, the translator default-passthrough shows up as a 4xx-rate spike on a specific upstream, and the construction-path drift shows up as a divergence in the per-path field-set entropy. None of those are subtle once you know what to look for. The hard part, again, was naming what to look for.

If you do agent-infra reviews, try the four-question checklist for a week and tell me what's missing. I expect a fifth shape to surface within ten reviews. When it does I'll add it here.
