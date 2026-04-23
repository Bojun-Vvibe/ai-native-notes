---
title: "A reading protocol for understanding a 100k-LOC OSS repo with a coding agent in 30 minutes"
date: 2026-04-23
tags: [reading-protocol, oss, agents, codebase-comprehension]
est_reading_time: 8 min
---

The default move when a coding agent meets an unfamiliar repo is to grep around for whatever the user just asked about. This works for files. It does not work for understanding. Half an hour later you have a stack of disconnected snippets and no model of how the system actually fits together. The next question gets a worse answer than the first because the agent's context is full of low-signal noise.

I have a protocol I run with the agent before asking it any actual question about a new OSS codebase. It takes 20-30 minutes of agent time, costs a few thousand tokens, and produces a stable mental model that survives the next several hours of work. Below: the steps, the prompts I actually use, and what the output looks like.

## The protocol

Six steps, in this order. Each one has a fixed scope. The agent does not move on until the previous step's output is on disk.

1. Shallow clone, no history.
2. README plus tree, with a budget.
3. Find the entry point.
4. Trace the main loop.
5. Map the tool dispatcher (or equivalent extension surface).
6. Read the test layout.

The output of all six steps gets dumped into a single `NOTES.md` in the repo root, which is later fed back to the agent as part of any prompt about the repo. The notes file is the persistent context; the chat history is throwaway.

## Step 1: shallow clone

```bash
gh repo clone openhands-mentat/openhands -- --depth=1 --filter=blob:none
cd openhands
```

`--depth=1` skips the full history. `--filter=blob:none` defers blob downloads until they're actually accessed. On a 100k-LOC repo this brings the clone time from minutes to seconds and the disk footprint from hundreds of MB to a few MB. The history will be wrong if you ask the agent "when did this function get added," but for comprehension you don't need history yet.

If you do need to study evolution later, `git fetch --unshallow` is one command away.

## Step 2: README plus tree, with a budget

The prompt for the agent:

```
Read README.md, then run `tree -L 3 -I 'node_modules|dist|build|.git'` and
write to NOTES.md a single paragraph (no more than 200 words) covering:
- what this project does in one sentence
- the top-level directories and what each one is for
- the main entry point (filename and approximate location)

Do not read any other files yet. If the README is over 500 lines, summarize
only its first two sections. Do not invent purposes for directories you do
not understand; mark them as "unknown" instead.
```

Two things matter here. First, the budget — 200 words, no other files. This forces the agent to compress, not catalog. A catalog of every file is useless; a compressed paragraph is a navigation map. Second, the explicit "do not invent" — agents will confidently label every directory based on its name, and `lib/` could be source code, vendored dependencies, or a build artifact directory. The "mark as unknown" instruction creates a list of things to investigate, which is exactly what you want.

A real output for a project I read recently:

```
# NOTES.md

A terminal-based coding agent. Top-level layout:
- src/cli — entrypoint, argument parsing, top-level command dispatch
- src/agent — the main agent loop, message construction, model client
- src/tools — tool implementations (file ops, shell, search). Plugin-style.
- src/providers — model adapters (anthropic, openai, openrouter)
- src/mcp — MCP client for external tool servers
- tests — pytest, organized to mirror src/
- examples — runnable example sessions, useful for learning the API
- scripts — release and dev tooling. Unknown contents.
- assets — unknown.
Main entry: src/cli/main.py via console_script `og`.
```

This is enough to pick the next file. It is not enough to answer a real question. That's fine; it's step 2.

## Step 3: find the entry point

The agent reads exactly one file: the entry point identified in step 2. It writes a section to NOTES.md describing what happens when you invoke the CLI from cold start to "ready for first user input." Specifically: which functions are called in what order, where config is loaded from, where the agent loop starts.

Prompt:

```
Read src/cli/main.py. Append a section to NOTES.md titled "Cold start" that
traces, in order, every function call from invocation to the point where
the agent loop is ready to accept input. Include file:line references.
Do not read other files in this step; if you need to know what an imported
function does, write "TODO: read X" instead of guessing.
```

The TODO list at the end of step 3 is the work plan for step 4.

## Step 4: trace the main loop

Now the agent follows the TODOs. It reads the agent loop file, traces one full iteration: input is read, prompt is constructed, model is called, response is parsed, tool is dispatched, tool result is appended, loop repeats. NOTES.md gets a "Main loop" section with the same file:line discipline.

Prompt:

```
Read the files in your TODO list. Append a section "Main loop" to NOTES.md
that traces ONE iteration of the agent loop:
1. Where input enters
2. How the prompt is constructed (system + history + user)
3. How the model client is called and which model is selected
4. How the response is parsed (text vs tool calls)
5. How tools are dispatched
6. How results are appended back into history
For each step give file:line. If a step is unclear, write "unclear: <why>"
rather than inventing detail.
```

The "unclear" tag is load-bearing. It is the difference between a map you can trust and a map that's confidently wrong about a region you'll later visit and find missing.

## Step 5: map the tool dispatcher

Most agent codebases have one place where "the model said to call tool X" turns into actually invoking tool X. That place is the most important file in the repo for understanding capabilities. The agent reads it and writes a "Tools" section listing every tool the agent can call, what each one does, and what its inputs and outputs are.

Prompt:

```
Read src/tools/ (or whichever directory step 4 identified as the tool
implementation location). Append a section "Tools" to NOTES.md listing
every tool: name, one-sentence purpose, inputs, outputs. Do not include
implementation details. If there are more than 20 tools, list them all
but compress descriptions to a few words each. Prefer scanning a manifest
or registry file over reading each tool implementation in full.
```

The "scan the registry" hint is important. Reading every tool implementation is a 50k-token waste; almost every tool framework has a single registration point that lists all tools with metadata.

## Step 6: read the test layout

Tests are documentation written by people who had to actually make the code work. The agent looks at the test directory structure, picks the largest or most central-looking test file, and writes a section describing how the test harness works and what's covered.

Prompt:

```
Run `tree -L 2 tests/` and read the file with the highest line count under
tests/. Append a section "Tests" to NOTES.md with:
- the test framework in use
- the most-tested subsystem (the directory under tests/ with the most files)
- one example of how a test is structured (a 5-line code excerpt is fine)
- gaps: subsystems with little or no test coverage
```

Gaps in test coverage are gold. They tell you which parts of the code you should not trust, which is more useful than knowing which parts you can.

## What you have at the end

A NOTES.md of 600-1000 words covering: what the project is, top-level layout, cold-start path, main loop, tool surface, test layout, and an explicit list of unclear regions. Total agent time: 20-30 minutes. Total tokens: in my recent runs, between 8k and 15k input, 2k-4k output. Cost in real money: under a dollar with caching enabled.

This NOTES.md becomes the first attached file in every subsequent prompt about the repo. It is the persistent compressed context. The agent does not need to re-explore on every question.

## What this protocol does not give you

It does not give you correctness. The agent will get details wrong, label things confusingly, and miss subtleties. The protocol gives you a *navigable surface* so that when you ask the next question, the agent knows roughly where to look instead of grep-bombing the whole repo.

It also does not give you understanding of why the codebase is the way it is. For that you need the history (`git fetch --unshallow`, then read the merge commits and the issues they reference). That is a different protocol; out of scope here.

## What I would do differently

I used to skip step 2 and start with step 3. The agent would then misread the entry point because it had no idea which directory to look in. Step 2 takes two minutes and prevents an entire class of "I read the wrong file for ten minutes" failure. Do not skip it.

I also used to let the agent read more than one file per step. The output got better in the moment and worse over time, because each step was making decisions based on undigested earlier reads. Strict one-file-per-step is annoying and produces dramatically more useful notes.

## Links

- gh CLI clone docs: https://cli.github.com/manual/gh_repo_clone
- git partial clone reference: https://git-scm.com/docs/partial-clone
- tree(1) on most package managers, e.g. `brew install tree`
