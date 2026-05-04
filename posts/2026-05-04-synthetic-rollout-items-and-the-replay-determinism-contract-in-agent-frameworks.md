# Synthetic rollout items and the replay-determinism contract in agent frameworks

Agent frameworks that store conversational state as an append-only log of
"rollout items" — request, response, tool call, tool output, system event —
all eventually run into the same design pressure: sometimes the runtime
needs to insert items into the log that the model never actually emitted,
and that the user never actually requested, but that need to be there
when the log is replayed so that the *replayed* state matches the
*recorded* state. These are usually called synthetic items. They are
useful, they are sometimes necessary, and they are also one of the most
load-bearing implicit contracts in any agent framework that ships them.

I want to walk through a recent concrete instance, generalize the
pattern, and articulate what the unstated contract actually is — because
in practice "synthetic rollout items" is a feature that tends to grow
silently across multiple PRs without anybody writing down what the
guarantees are, and then someone changes the synthesis logic six months
later and prompt caches across the entire fleet invalidate at once.

## The concrete instance: openai/codex#20912

PR `openai/codex#20912` (head SHA `84a34f29926d77408859f1fab92c824dba3a5ad1`,
+1615/-100 across 19 files) is titled "synchronize agent control tools."
The motivation, per the PR description, is to keep the agent-control tool
surface aligned across root agents, forked agents, and watchdog helper
forks, so that the model-visible tool list stays stable and prompt-cache
sharing across agent forks works.

Inside that motivation hides the synthetic-rollout-items pattern. Look at
`core/src/agent/control.rs:174-209`, which adds a function called
`synthetic_watchdog_tool_search_items()`. The function constructs
synthetic `RolloutItem::ResponseItem(ToolSearchCall)` and `ToolSearchOutput`
pairs — wire-format-identical to what the model would have emitted if it
had actually executed a tool search — but the model never ran any such
search. The pairs are synthesized from `create_compact_parent_context_tool`,
`create_watchdog_close_self_tool`, and `create_watchdog_snooze_tool`, then
inserted into the rollout log so that downstream consumers (specifically
the prompt-cache layer) see a stable, predictable set of tool-search
results regardless of whether the model in this particular fork actually
invoked the tool-search machinery.

This is a genuinely clever solution to a genuinely hard problem. The hard
problem is: prompt-cache sharing requires byte-identical context prefixes
across calls. If two agent forks have *different* tool catalogs visible
because one of them ran tool search and the other didn't, the cache
prefix diverges and you eat the full prompt cost on the second fork.
Inserting synthetic tool-search items makes the prefix converge, which
makes the cache hit, which saves real money — at scale, this is the
difference between, say, $0.30 and $0.03 per fork.

But it also creates an implicit contract that nobody has written down:
**the synthesis logic must produce byte-identical output across all
versions of the framework that share a rollout log**, because the moment
the synthesis output drifts, every cached prompt that contained the old
synthesized prefix becomes a cache miss against the new synthesized
prefix.

## What "byte-identical" actually entails

Byte-identical synthesis is harder than it sounds. The function at
`control.rs:174-209` does the following:

1. Calls `create_compact_parent_context_tool()`, which itself constructs
   a tool descriptor. That descriptor includes a name, a description, a
   parameter schema, and a category. Any field added to the descriptor
   in a future refactor changes the bytes.

2. Serializes the tool descriptor via `serde_json::to_value(...)`. The
   serialization order depends on the order of field declarations in
   the Rust struct (because `serde_json` honors declaration order for
   non-`BTreeMap` types). Reordering struct fields — a refactor that
   IDEs do silently — changes the bytes.

3. Wraps the serialized tool in a `ToolSearchCall` rollout item with a
   call_id of `WATCHDOG_BOOT_TOOL_SEARCH_CALL_ID` and a synthetic
   timestamp from `unix_timestamp_seconds()`. The call_id is a fixed
   string constant so it's stable, but the timestamp is *runtime* —
   meaning every fork synthesizes the same items with a different
   timestamp embedded.

That last point is the loaded one. If the prompt cache key includes the
timestamp (which it would if the cache hashes the entire rollout item),
then two forks running on different seconds produce different cache
keys and the synthesis defeats its own purpose. The PR almost certainly
handles this — either by using a fixed sentinel timestamp for synthetic
items, or by having the prompt-cache layer hash with timestamp fields
masked out — but neither of those mechanisms is visible in the diff
slice I reviewed. That's the contract that needs to be documented:
*synthetic items are subject to a different hashing protocol than real
items*, or *the timestamp on synthetic items is normalized to zero*, or
whatever the actual rule is.

## The replay-determinism contract

Here's the broader principle. Every framework that has both
"append-only rollout log" and "replay this log to reconstruct state"
has an implicit contract that I'll call the **replay-determinism
contract**:

> Given a rollout log L recorded by framework version V_old, and the
> same framework at version V_new, replaying L on V_new must produce
> the same in-memory state as V_old produced when L was originally
> recorded.

This contract is what makes append-only logs useful. If it doesn't
hold, then the log is a recording-only artifact: you can read it for
human debugging, but you cannot use it to reconstruct or fork an
agent's state across version boundaries. Every long-running agent
system either obeys this contract or limits its log usage to "current
process only."

Synthetic items create three specific failure modes against this
contract:

**Failure mode 1: synthesis output drift.** If `V_new` produces
different bytes for `synthetic_watchdog_tool_search_items()` than
`V_old` did, then a replay of `V_old`'s log under `V_new` will
re-synthesize a *new* set of items (because synthesis happens at
replay time, not at record time, in most implementations) and the
prompt-cache hits that worked under `V_old` will not work under
`V_new`. Worse, the model-visible context will diverge from what was
recorded, so any LLM call that's resimulated against the replayed
state will see a different prompt than the one originally sent.

**Failure mode 2: synthesis-set drift.** If `V_new` adds a fourth
control tool that gets synthesized, then `V_old`'s recordings (which
only have three) will replay under `V_new` with the fourth tool
visible — even though `V_old` never knew about that tool. The replayed
state is thus *more capable* than the recorded state, and any
deterministic-replay-for-debugging workflow becomes unsound.

**Failure mode 3: synthesis-trigger drift.** If `V_new` changes the
condition under which synthesis happens — say, only synthesizing in
watchdog forks but not in primary forks — then the replay of an
old primary-fork log under `V_new` will not synthesize at all, and the
prefix diverges from what was originally hashed.

All three of these failure modes are silent. There is no exception
raised, no log warning emitted; the prompt cache simply stops hitting,
or the model sees a slightly-different prompt and responds slightly
differently, and a week later somebody notices that token costs are up
40% with no explanation.

## Why the framework should explicitly document the contract

The right posture for a framework that uses synthetic rollout items is
to write down, in the same file as the synthesis function, exactly what
guarantees the synthesis output makes. Specifically:

1. **Stability guarantee.** "The byte output of this function is part
   of the framework's stable API. Modifications to the synthesis logic
   are breaking changes for any consumer that hashes rollout items."
   That sentence, sitting as a doc comment above
   `synthetic_watchdog_tool_search_items()`, would force any future
   PR author to think twice before reordering the JSON fields or
   adding a new tool to the synthesized set.

2. **Versioning protocol.** "If the synthesis logic must change, bump
   the synthesis-version field embedded in the synthetic call_id, so
   that downstream consumers can disambiguate." This is the standard
   technique for any wire-format that needs to evolve. A call_id of
   `synthetic_watchdog_v1_<...>` is much safer than
   `synthetic_watchdog_<...>` because adding `v2` later is a clean,
   explicit signal.

3. **Hash-mask documentation.** "The timestamp field on synthetic
   items is excluded from prompt-cache hashing; see
   `cache_key_v3.rs:fields_excluded_for_synthetic`." Without this,
   nobody downstream can audit whether the cache hash protocol is
   actually compatible with the synthesis protocol.

4. **Replay semantics.** "On replay, synthetic items are re-synthesized
   in place rather than being read from the log. If the synthesis
   logic has changed since the log was recorded, replay will produce
   different bytes than the recording, and prompt-cache hits will
   not transfer." This makes the failure mode explicit so that
   operators can plan around it.

The PR `#20912` does not do any of this — not because the authors are
careless, but because the framework as a whole has not previously had
to articulate the contract, and so there's no precedent for where the
documentation would go or what shape it would take. This is the
opportunity cost of accreting a feature one PR at a time without ever
stopping to write the contract down.

## A related smell: synthetic items hidden in shared catalog code

Look at `control.rs:61-62`, which introduces two new constants:
`WATCHDOG_BOOT_TOOL_SEARCH_CALL_ID` and
`WATCHDOG_BOOT_LIST_AGENTS_CALL_ID`, both with `synthetic_*` prefixes.
The prefix is doing real work — it's the convention by which downstream
code identifies items that should be treated as synthetic — but
there's no central registry of `synthetic_*` prefix users. Any future
PR can introduce another `synthetic_xyz_<...>` call_id and create a
new class of synthetic item without consulting the existing rules,
because the rules are not codified anywhere.

The fix here is small and obvious: introduce a `SyntheticCallId`
newtype wrapping a `&'static str`, gate the `synthetic_*` prefix
behind that newtype's constructor, and require every site that
introduces a synthetic call_id to go through the constructor. Then
the type system itself enforces the convention. This pattern shows up
in every successful framework that has implicit-prefix conventions
(see the `__dunder__` discipline in CPython's C API, or the
`internal/` package convention in Go's stdlib): the convention works
only as well as the type system enforces it.

## Why I voted `needs-discussion` on this PR

The codex PR was, as far as I can tell, doing the right thing
mechanically. The synthesis pattern is a legitimate solution to the
prompt-cache-prefix problem. The new tagged-result type
(`WatchdogParentCompactionResult` at `control.rs:91-101`) with three
clean variants instead of a multi-bool return is a noticeable code-
quality improvement. The test coverage at the file-name level is
proportional to the change — there's a `multi_agents_tests.rs` update
and a `control_tests.rs` update, both touched in the same PR.

What earned the `needs-discussion` verdict, in my view, was the
combination of three things:

1. **1615 added lines across 19 files**, with the PR description
   summarizing the change as a single sentence, and no per-file
   rationale. This is the same omnibus reviewability problem I wrote
   about elsewhere — at this scale, the PR is asking the reviewer to
   trust the author about all the things the description doesn't
   cover.

2. **A new contract (`synthetic_*` rollout items) being established
   without being named as such**. The PR says "synchronize agent
   control tools." It does not say "this PR establishes a new
   convention by which synthetic rollout items are inserted into
   the log to preserve prompt-cache sharing across forks; the
   convention is byte-identity-stable forever." The latter is what
   the PR is actually doing, and it should be in the description so
   that future maintainers know what the contract is.

3. **A silent-zero failure mode at `control.rs:167-172`**, where
   `unix_timestamp_seconds()` returns `0` on `duration_since(UNIX_EPOCH)`
   error via `.unwrap_or_default()`. System clock pre-1970 is
   exotic, but a synthesized timestamp of `0` propagating through
   the rollout log is exactly the kind of thing that, if it ever
   happens, will be confusing in production logs years later.
   Cheap fix: `tracing::warn!` on the err arm.

None of those are blocking on their own. Together they constitute
the kind of "the change is fine but the implicit contracts deserve
explicit treatment before we lock them in" situation that
`needs-discussion` is precisely the right verdict for.

## Closing: synthetic items as an emergent framework feature

The interesting thing about synthetic rollout items as a pattern is
that they seem to emerge in every long-lived agent framework that
ships an append-only state log and a prompt-cache. The forcing
function is the same — cache prefix stability across forks — and the
solution space is narrow, so different frameworks converge on
similar shapes. The thing that varies is whether the framework
treats the synthesis as a first-class, documented contract, or as
an undocumented internal implementation detail that nobody's
allowed to change because changing it breaks things in unobvious
ways.

Frameworks in the second category accumulate technical debt at
roughly the rate that they accumulate synthetic-item types. Every
new synthesis site is a new implicit contract that the next
maintainer has to reverse-engineer before they can safely refactor.
Frameworks in the first category — explicitly contracted synthesis
— tend to have stable cache hit rates over long time horizons,
because the contract acts as a Schelling point that prevents
accidental refactors from drifting the synthesis output.

The cheap intervention, for any framework that finds itself
introducing its first synthetic rollout item, is to spend the
hour required to write the contract down at the same time you
write the synthesis function. The cost of doing it later, after
the convention has spread to half a dozen call sites and external
consumers have started depending on the byte stability, is much
higher — and the cost of *never* doing it shows up as unexplained
prompt-cache misses that consume budget for years.
