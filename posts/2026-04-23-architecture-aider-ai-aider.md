---
title: "Architecture deep-dive: Aider-AI/aider"
date: 2026-04-23
tags: [architecture, deep-dive, aider]
est_reading_time: 14 min
---

## TL;DR

- `aider` is a Python pair-programmer with a small surface (~13k LOC in `aider/`) that does three things very well: it knows your repo (tree-sitter + PageRank), it talks to any LLM (via litellm), and it edits files using a strict text-diff format that's easy for the model and easy to apply. There's no MCP, no tool registry, no async. The whole thing is a single-threaded blocking loop.
- The most interesting design decision is the **Coder subclass per edit format**. `EditBlockCoder`, `UDiffCoder`, `WholeFileCoder`, `PatchCoder` — each one ships its own system prompt, its own parser, and its own apply function. The shape of the model's response *is* a class. New format = new subclass; nothing else changes.
- What I'd steal: the **background cache-warming thread**. Aider sends a 1-token completion every ~5 minutes to keep the prompt-cache prefix alive. The cost is negligible; the savings on a long pairing session are large.

## Repo at a glance

- **Language**: Python 3, ~13k LOC under `aider/` (the runtime), ~6.9k of which sits in `aider/coders/`.
- **Top-level layout**: `aider/coders/` (one file per edit format + the base class), `aider/repomap.py` (the symbol-graph), `aider/repo.py` (git plumbing), `aider/sendchat.py` (the litellm wrapper), `aider/io.py` (terminal I/O, prompt_toolkit), `aider/commands.py` (the `/commands` system), `aider/main.py` (entry point + argparse).
- **Build**: setuptools, `pyproject.toml`, no compiled extensions of its own (depends on `tree-sitter`, `networkx`, `litellm`, `diskcache`, `grep_ast`, `pygments`).
- **Tests**: `pytest`, plus a benchmarking harness under `benchmark/` that runs against the Exercism Python exercises.

All references below are to commit `f09d706`.

## The agent loop

Aider's loop has three concentric circles. The outer one is REPL. The middle one is "send a message." The inner one is the **reflection loop** — what aider calls the iteration that happens when an edit fails to apply, the linter complains, or a test fails.

The outer REPL is in `Coder.run` ([`aider/coders/base_coder.py:876`](https://github.com/Aider-AI/aider/blob/f09d70659ae90a0d068c80c288cbb55f2d3c3755/aider/coders/base_coder.py#L876)):

```python
# aider/coders/base_coder.py:876-892
def run(self, with_message=None, preproc=True):
    try:
        if with_message:
            self.io.user_input(with_message)
            self.run_one(with_message, preproc)
            return self.partial_response_content
        while True:
            try:
                if not self.io.placeholder:
                    self.copy_context()
                user_message = self.get_input()
                self.run_one(user_message, preproc)
                self.show_undo_hint()
            except KeyboardInterrupt:
                self.keyboard_interrupt()
    except EOFError:
        return
```

Synchronous. Blocking on `get_input()` (which is `prompt_toolkit` under the hood). `Ctrl-C` is handled inline, not via signal handlers. There's no async runtime anywhere in aider — this is unusual for a 2026-vintage agent and it's a deliberate choice. Aider is fundamentally turn-based: the user types, the model talks, the model edits, the user types again. There's no streaming UI to update from a background task and no concurrent tool execution to manage.

The middle layer is `run_one` ([`base_coder.py:924`](https://github.com/Aider-AI/aider/blob/f09d70659ae90a0d068c80c288cbb55f2d3c3755/aider/coders/base_coder.py#L924)):

```python
# aider/coders/base_coder.py:924-947
def run_one(self, user_message, preproc):
    self.init_before_message()

    if preproc:
        message = self.preproc_user_input(user_message)
    else:
        message = user_message

    while message:
        self.reflected_message = None
        list(self.send_message(message))

        if not self.reflected_message:
            break

        if self.num_reflections >= self.max_reflections:
            self.io.tool_warning(f"Only {self.max_reflections} reflections allowed, stopping.")
            return

        self.num_reflections += 1
        message = self.reflected_message
```

This is the entire reflection mechanism. After every model response, `send_message` may set `self.reflected_message` to something. If it does, that string becomes the *next* user message and the loop runs again, capped at `max_reflections` (default 3). Reflections are triggered for:

- **Mentioned files not in chat**: model named a file that wasn't loaded → aider builds a "please add these files" prompt and reflects.
- **Edit failures**: SEARCH/REPLACE block didn't match → aider builds a "here are the failed blocks, here are similar lines" prompt and reflects.
- **Lint errors after edits**: linter found problems → aider reflects with the linter output if the user confirms.
- **Test failures**: tests fail after an edit → reflection with the test output.

The reflected_message is the only "memory" of failure that crosses iterations. There is no separate scratchpad, no agent state. The mechanism is laughably simple and it works because the model is the one doing the planning; aider just keeps feeding it the next constraint.

The inner core is `send_message` ([`base_coder.py:1419`](https://github.com/Aider-AI/aider/blob/f09d70659ae90a0d068c80c288cbb55f2d3c3755/aider/coders/base_coder.py#L1419)). At ~190 lines it's the second-largest method in the file, and it earns the size: it builds the prompt, calls the model with retry/backoff, captures the streamed response, parses it, applies edits, runs lint, runs tests, decides whether to reflect, and updates token accounting. The skeleton:

```python
# aider/coders/base_coder.py:1419-1460 (abridged)
def send_message(self, inp):
    self.cur_messages += [dict(role="user", content=inp)]
    chunks = self.format_messages()
    messages = chunks.all_messages()
    if not self.check_tokens(messages):
        return
    self.warm_cache(chunks)

    # ...spinner setup...

    retry_delay = 0.125
    litellm_ex = LiteLLMExceptions()
    try:
        while True:
            try:
                yield from self.send(messages, functions=self.functions)
                break
            except litellm_ex.exceptions_tuple() as err:
                ex_info = litellm_ex.get_ex_info(err)
                if ex_info.name == "ContextWindowExceededError":
                    exhausted = True
                    break
                if ex_info.retry:
                    retry_delay *= 2
                    if retry_delay > RETRY_TIMEOUT:
                        break
                    time.sleep(retry_delay)
                    continue
                break
            # ...
```

The retry strategy is a pure exponential backoff (`*= 2`) capped at `RETRY_TIMEOUT`. The classification of "should I retry this?" is delegated to `litellm`'s exception inspector — aider doesn't keep its own taxonomy. This is one of the under-appreciated benefits of leaning on an LLM gateway: the gateway does the per-provider error mapping so you don't have to.

After the model finishes, aider runs `apply_updates()` (which calls into the subclass's `get_edits` + `apply_edits`), and then the post-edit pipeline:

```python
# aider/coders/base_coder.py:1585-1618 (abridged)
edited = self.apply_updates()

if edited:
    self.aider_edited_files.update(edited)
    saved_message = self.auto_commit(edited)
    self.move_back_cur_messages(saved_message)

if edited and self.auto_lint:
    lint_errors = self.lint_edited(edited)
    self.auto_commit(edited, context="Ran the linter")
    self.lint_outcome = not lint_errors
    if lint_errors:
        ok = self.io.confirm_ask("Attempt to fix lint errors?")
        if ok:
            self.reflected_message = lint_errors
            return

shared_output = self.run_shell_commands()
# ...
if edited and self.auto_test:
    test_errors = self.commands.cmd_test(self.test_cmd)
    # ... reflect on test failures ...
```

Notice that aider commits *automatically* after every edit. It also commits *again* after the linter runs. The git history of an aider session looks like a series of tiny atomic commits — "feat: add factorial helper", "lint: format with black" — which is wonderful for `git bisect` after the fact and grates on engineers used to squashed PRs. The `move_back_cur_messages` call replaces the model's edit blocks in `cur_messages` with a "files have been edited" placeholder, so the next iteration doesn't re-paste 200 lines of SEARCH/REPLACE into context.

## Tool calling implementation

Aider does not use function-calling for editing in the default path. The model emits its edits as **literal text** in the streaming response, in a format the subclass knows how to parse. This is the most distinctive choice in the codebase and worth understanding before anything else.

For the default `EditBlockCoder` ([`aider/coders/editblock_coder.py`](https://github.com/Aider-AI/aider/blob/f09d70659ae90a0d068c80c288cbb55f2d3c3755/aider/coders/editblock_coder.py)) the format is a fenced block:

```
path/to/file.py
<<<<<<< SEARCH
old code that must match exactly
=======
new code
>>>>>>> REPLACE
```

Parsing happens in `find_original_update_blocks`, which walks the response with regexes for `<{5,9} SEARCH`, `=====+`, and `>{5,9} REPLACE`. Application happens in `do_replace` — a literal string replace, with a fallback that tries fuzzy matching when the exact SEARCH block doesn't appear in the file.

```python
# aider/coders/editblock_coder.py:41-74 (abridged)
def apply_edits(self, edits, dry_run=False):
    failed = []
    passed = []
    for edit in edits:
        path, original, updated = edit
        full_path = self.abs_root_path(path)
        new_content = None
        if Path(full_path).exists():
            content = self.io.read_text(full_path)
            new_content = do_replace(full_path, content, original, updated, self.fence)
        # If the edit failed, and this is not a "create a new file" with an
        # empty original, try patching any of the other files in the chat
        if not new_content and original.strip():
            for full_path in self.abs_fnames:
                content = self.io.read_text(full_path)
                new_content = do_replace(full_path, content, original, updated, self.fence)
                if new_content:
                    path = self.get_rel_fname(full_path)
                    break
        if new_content:
            if not dry_run:
                self.io.write_text(full_path, new_content)
            passed.append(edit)
        else:
            failed.append(edit)
```

When edits fail, aider doesn't just say "edit failed." It builds a structured error message that includes the exact failed block and `find_similar_lines` output (a `difflib.SequenceMatcher`-based "did you mean these lines?"). That message is set as `self.reflected_message` and the loop runs again. The model has been trained on enough of these failures that, given good "did you mean" hints, it usually fixes them on the next pass.

The other coders all follow the same shape but vary the format:

- `WholeFileCoder` — model emits the entire new contents of each file. Simplest, most token-expensive.
- `UDiffCoder` — model emits unified diffs.
- `PatchCoder` — model emits an OpenAI-style patch envelope.
- `EditBlockFunctionCoder` — same SEARCH/REPLACE format but wrapped in a function-call.

Each subclass carries its own `gpt_prompts` instance (e.g. `EditBlockPrompts`, `UDiffPrompts`) which holds the system prompt, the example conversations, and the system reminder. Switching edit formats means swapping the `Coder` class — nothing else.

The few things aider *does* expose as actual function-calls (when the model supports them) are control-plane: a function for "I want to add this file to the chat" and similar. They live in subclass `functions` attributes and are passed to `litellm.completion(... functions=...)`. The dispatch is trivial: parse the JSON args, call a Python method.

## Prompt construction & cache discipline

This is where aider shines compared to most of the agents I've read. The prompt is built up as a `ChatChunks` dataclass with **named, ordered segments** ([`aider/coders/chat_chunks.py`](https://github.com/Aider-AI/aider/blob/f09d70659ae90a0d068c80c288cbb55f2d3c3755/aider/coders/chat_chunks.py)):

```python
# aider/coders/chat_chunks.py:5-26
@dataclass
class ChatChunks:
    system: List = field(default_factory=list)
    examples: List = field(default_factory=list)
    done: List = field(default_factory=list)
    repo: List = field(default_factory=list)
    readonly_files: List = field(default_factory=list)
    chat_files: List = field(default_factory=list)
    cur: List = field(default_factory=list)
    reminder: List = field(default_factory=list)

    def all_messages(self):
        return (
            self.system
            + self.examples
            + self.readonly_files
            + self.repo
            + self.done
            + self.chat_files
            + self.cur
            + self.reminder
        )
```

The order is deliberate: most-stable at the top (system, examples), most-volatile at the bottom (cur, reminder). And the cache markers are placed at exactly the boundaries where the prefix is most likely to be reused:

```python
# aider/coders/chat_chunks.py:28-41
def add_cache_control_headers(self):
    if self.examples:
        self.add_cache_control(self.examples)
    else:
        self.add_cache_control(self.system)

    if self.repo:
        # this will mark both the readonly_files and repomap chunk as cacheable
        self.add_cache_control(self.repo)
    else:
        # otherwise, just cache readonly_files if there are any
        self.add_cache_control(self.readonly_files)

    self.add_cache_control(self.chat_files)
```

Three explicit cache breakpoints (Anthropic-style `cache_control: ephemeral`): one after the system prompt / examples, one after the repo map + read-only files, one after the chat files. The recent conversation (`cur`) and the system reminder are *not* cached — they change every turn.

And then there's the warming. `warm_cache` ([`base_coder.py:1340`](https://github.com/Aider-AI/aider/blob/f09d70659ae90a0d068c80c288cbb55f2d3c3755/aider/coders/base_coder.py#L1340)) starts a daemon thread that periodically pings the API to keep the prefix from expiring:

```python
# aider/coders/base_coder.py:1357-1381 (abridged)
def warm_cache_worker():
    while self.ok_to_warm_cache:
        time.sleep(1)
        if self.warming_pings_left <= 0:
            continue
        now = time.time()
        if now < self.next_cache_warm:
            continue

        self.warming_pings_left -= 1
        self.next_cache_warm = time.time() + delay  # delay = 5*60 - 5 by default

        kwargs = dict(self.main_model.extra_params) or dict()
        kwargs["max_tokens"] = 1
        try:
            completion = litellm.completion(
                model=self.main_model.name,
                messages=self.cache_warming_chunks.cacheable_messages(),
                stream=False,
                **kwargs,
            )
        except Exception as err:
            self.io.tool_warning(f"Cache warming error: {str(err)}")
            continue

        cache_hit_tokens = getattr(
            completion.usage, "prompt_cache_hit_tokens", 0
        ) or getattr(completion.usage, "cache_read_input_tokens", 0)
```

Anthropic's ephemeral cache TTL is 5 minutes; aider pings at 4:55. The completion requests `max_tokens=1` so the cost is one output token plus a cache *read* (the cheap part). The savings on the next real request — which gets to start with the entire prefix already cached — are vastly larger. This is the single best ROI optimization in the codebase.

The `cacheable_messages()` helper finds the rightmost message with a `cache_control` marker and returns everything up to and including it, so the warm-cache request doesn't waste tokens replaying the volatile bits.

## State / session management

Aider's state model is split between in-memory and on-disk, like codex's, but the on-disk side is git-native rather than a custom rollout format.

**In-memory**: `Coder` holds two message lists — `done_messages` (everything that's been "summarized" or moved out of active context) and `cur_messages` (the current turn). After every model response, `move_back_cur_messages(saved_message)` strips the verbose edit blocks out of `cur_messages` and replaces them with a short placeholder, so the prompt stays small.

**On-disk**: aider writes a chat history to `.aider.chat.history.md` and an input history to `.aider.input.history` in the project root. Edits are committed to git automatically, with attribution in the commit trailer (`# Aider edits:`). On restart with `--restore-chat-history`, aider replays the history file to rebuild `done_messages`.

**Compaction**: handled by `summarize_end()` in `format_chat_chunks` and the standalone `ChatSummary` class. Aider doesn't compact mid-turn the way codex does — it summarizes when the conversation *as a whole* exceeds a threshold, replacing older messages in `done_messages` with a model-generated summary. The compaction is itself an LLM call, but to a separate "weak model" (`weak_model`) that can be a smaller/cheaper checkpoint than the primary.

The interesting absent feature is any kind of branching or fork. Aider's chat is a single line; if you want to try two approaches you `/undo` and re-prompt. Codex (and OpenHands) both have multi-thread machinery; aider deliberately doesn't.

## Three patterns I'd steal

1. **Background cache-warming thread.** Already covered. ~30 lines of code; sub-cent cost per session; large savings on long sessions because every "real" message hits a fully cached prefix. Any agent that uses Anthropic's prompt cache should consider this.

2. **`ChatChunks` named segments + explicit cache breakpoints.** Most agents build prompts as opaque strings. Aider builds them as a struct of segments with stable order and known cache-eligibility. Adding a new context source means adding a field, not threading a parameter through five functions. Refactoring "what's cacheable" is a one-line change.

3. **Reflection via `self.reflected_message`.** A failed edit, a lint error, a test failure — all of them set the same field, which becomes the next user message. There is no "reflection state machine," no "agent strategy framework." Just a string. The simplicity is what makes it robust: every failure path either reflects or doesn't, and `max_reflections` caps the blast radius.

## Three pitfalls in this codebase

1. **`base_coder.py` is 2,485 lines and growing.** The `Coder` base class has accumulated every cross-cutting concern: prompt building, token accounting, retry, edit application, linting, testing, git committing, copy-pasting, file watching, and the reflection loop. New subclasses inherit all of it. A clean refactor would extract `EditPipeline`, `RepoState`, `ModelClient`, and `MessageFormatter` as separate collaborators, but the cost of breaking external subclasses (people fork aider) is real.

2. **Pure-text edit format means no schema validation.** The model can emit a SEARCH block that *almost* matches but has a one-character whitespace drift, and aider falls back to fuzzy matching. The fuzzy matcher is good but not perfect, and silent partial matches are scarier than loud failures. Function-call coders (`EditBlockFunctionCoder`) avoid this but are gated on the model supporting strict JSON output. I'd want a strict mode where any non-exact match becomes a reflection, full stop.

3. **Synchronous, single-threaded.** This is also a strength (simpler code, no race conditions), but it means aider can't do useful work while the model is streaming. There's no concurrent tool dispatch, no parallel file reads, no MCP. If the model is mid-stream and you realize you want to add a file, you have to wait. For a pair-programmer this is fine; for anything more agentic it's a wall.

## Comparison hook

Where aider keeps the "shape of the response" in the type system — `EditBlockCoder`, `UDiffCoder`, `PatchCoder` are different classes with different prompts and parsers — `openai/codex` keeps it in the **tool registry**. Codex has a single agent loop and a hundred possible tool combinations; aider has a hundred possible coder combinations and a small set of "tools" (mostly file I/O wrappers). Both work, but they optimize for different things: aider for cache stability and deterministic edits, codex for compositional surface area. If your agent is going to integrate with arbitrary MCP servers, you want codex's shape. If your agent is going to do one thing very well over and over, you want aider's.

## Reading order

For a new contributor, I'd read:

1. `aider/main.py` — entry point and argparse. Skim for the wiring.
2. `aider/coders/base_coder.py:876` — `run`, `run_one`, `init_before_message`. The skeleton.
3. `aider/coders/base_coder.py:1419` — `send_message`. The full turn lifecycle in one method.
4. `aider/coders/chat_chunks.py` — 64 lines, read in full. This is the prompt model.
5. `aider/coders/editblock_coder.py` — pick one edit format and follow it end-to-end. Editblock is the default and the simplest non-trivial one.
6. `aider/coders/editblock_prompts.py` — read the actual system prompt. It's short and the example messages are illuminating.
7. `aider/repomap.py:365` — `get_ranked_tags`. The PageRank-on-a-symbol-graph is aider's most original piece of engineering and it's worth understanding even if you don't plan to use it.
8. `aider/repo.py` — git plumbing. Mostly straightforward but there are nuances around dirty trees and pre-commit hooks.

Skip on first pass: `aider/gui.py`, `aider/help.py`, `aider/onboarding.py`, anything in `aider/queries/` (tree-sitter query files — read after `repomap.py` makes sense).
