---
title: "Designing a deterministic dispatcher: family rotation, tie-breaks, and the parallel-tick problem"
date: 2026-04-24
tags: [dispatcher, automation, agents, concurrency, jsonl]
est_reading_time: 11 min
---

## The problem

I have an autonomous dispatcher that fires on a cron, picks a "family" of work to do (posts, tools catalog, retros, glossary), spawns a sub-agent, and writes the result to `history.jsonl`. For about three weeks it ran fine on a single tick at a time. Then I added a second cron entry — a faster one — to test whether the dispatcher could handle bursty load. Within an hour I had two ticks colliding on the same family, two sub-agents writing posts to the same date slug, and one beautiful little merge conflict where both branches had appended a different line to `history.jsonl` with the same `tick_id`.

The interesting part of that failure is not the conflict. The interesting part is that I had been running the dispatcher for weeks under the comforting illusion that it was "deterministic" when in fact it was deterministic only under serialization. The moment two ticks could be in flight at once, the design assumptions I never wrote down started leaking out as bugs. This post is about what those assumptions were, what I changed, and why I now think a dispatcher is mostly a coordination artifact rather than a scheduler.

## The setup

The dispatcher is a single Python file, ~400 lines, invoked by a launchd job every 30 minutes. It does roughly this:

1. Read `state/history.jsonl` to figure out which family ran most recently.
2. Apply a rotation policy to pick the next family.
3. Spawn a sub-agent (a separate `opencode` process with a tightly scoped prompt and a hard time budget) to do the work.
4. Wait for the sub-agent to return a single JSON line.
5. Append that line to `history.jsonl`, plus a tick envelope (start time, end time, exit code, family chosen, repo touched).

That's it. There is no daemon, no shared memory, no lock manager, no message broker. Each tick is a one-shot process. The state of the system lives entirely in `history.jsonl` and the git repos the sub-agents touch.

## What I tried

### Attempt 1: round-robin on a fixed family list

The first version of the rotation policy was the most obvious thing: keep a fixed list of families, and pick whichever one is "next" after the most recent entry in `history.jsonl`. If the last tick wrote `posts`, the next tick writes `tools`, then `retros`, then `glossary`, then back to `posts`.

This worked perfectly under serialized ticks. Under parallel ticks it produced an interesting failure: both ticks read `history.jsonl` at roughly the same moment, both saw the same "most recent family," both picked the same "next family," and both spawned a sub-agent for it. The sub-agents touched the same repo, sometimes the same file. One of them committed; the other tried to commit the same change on top and produced either an empty commit or a merge conflict that the sub-agent had no idea how to resolve.

The root cause was not the rotation policy. The root cause was that I had treated `history.jsonl` as if it were the source of truth for "what is happening right now," when in fact it is only the source of truth for "what has finished." There is a window — the duration of a sub-agent — where the system has decided what to do but has not yet recorded that decision. In that window, a second tick reading `history.jsonl` sees a stale world.

### Attempt 2: a lock file

The instinct here is to reach for a lock. I tried it. I added a `state/dispatcher.lock` file with `flock`-style semantics: a tick acquires the lock at the start, releases it at the end. If a second tick can't acquire the lock within 5 seconds, it exits with a no-op.

This worked, in the sense that it prevented collisions. But it converted the dispatcher from a parallel system into a serialized one. Under bursty load, ticks queued up behind the lock, and the entire point of having a faster cron disappeared. The 30-minute cron and the 10-minute cron started behaving identically, because the 10-minute cron spent most of its time blocked on the 30-minute cron's slow sub-agent.

It also introduced a new failure mode: stuck locks. If a sub-agent crashed or got OOM-killed, the lock file remained. The next tick would block, then the next, then the next, and by the time I noticed, I had a queue of dead ticks and no work being done. I added a stale-lock detector keyed on PID liveness, and that worked, but at this point I was just rebuilding a lock manager in shell, badly.

### Attempt 3: pessimistic family claim, written before the work starts

The next attempt was to write to `history.jsonl` *before* the work, not after. The tick would append a `claim` record with the chosen family, then do the work, then append a `result` record. A second tick reading `history.jsonl` would see the claim and pick a different family.

This is closer to right, but it has a subtle problem: appending to `history.jsonl` from two processes at roughly the same time is not atomic in the way I wanted. POSIX guarantees that an `O_APPEND` write of less than `PIPE_BUF` bytes is atomic with respect to interleaving, which means the bytes don't get scrambled. It does *not* guarantee that two readers between the two writes see a consistent ordering. More importantly, both ticks could read `history.jsonl`, see no claim, decide on the same family, and *then* both write claims. The claim record solves the "what finished" gap but not the "who decided first" gap.

I could have layered a lock on top of the claim, but at that point I was solving the same problem twice.

### Attempt 4: repo-disjoint family assignment

The version that actually works exploits a property of the work itself: each family touches a different repo. `posts` writes to `ai-native-notes`. `tools` writes to `ai-native-tools`. `retros` writes to `retros`. `glossary` writes to `glossary`. None of the families share files.

Once you notice that, the entire concurrency problem dissolves. Two ticks running in parallel are safe as long as they pick *different* families. The git repos themselves act as the partitioning structure. There is no shared mutable state between, say, a `posts` sub-agent and a `tools` sub-agent — they will never touch the same file, never produce a merge conflict, never need to coordinate.

So the rotation policy became: read `history.jsonl`, find the families currently in flight (claimed but not yet completed), and pick from the set of remaining families. If two ticks read `history.jsonl` at the same time and see the same "in flight" set, the *worst* thing that happens is they both pick the same family — but I can detect that collision after the fact, because the second one will write a result that conflicts with the first, and I can have it bail out gracefully.

In practice the collision happens roughly never, because the "in flight" window is on the order of 10 minutes and the cron interval is on the order of 10 minutes. The race window is small enough that I haven't seen a collision in two weeks of running both crons.

## What worked

The current dispatcher logic, in pseudocode:

```python
def pick_family(history_path: Path, families: list[str]) -> str | None:
    entries = read_jsonl(history_path)
    in_flight = {e["family"] for e in entries
                 if e.get("kind") == "claim"
                 and not has_matching_result(entries, e["tick_id"])}
    available = [f for f in families if f not in in_flight]
    if not available:
        return None  # everything's busy, this tick is a no-op

    # tie-break: prefer the family least recently completed
    last_completed = {f: last_completion_ts(entries, f) for f in available}
    return min(available, key=lambda f: last_completed.get(f, 0))
```

Three things matter here:

1. **The tie-break is deterministic.** Two ticks reading the same `history.jsonl` will pick the same family, but only if the available set has exactly one element. If there are two or more available families, ties are broken by "least recently completed," which is itself derived from `history.jsonl` — so two parallel ticks with the same view of history will pick the same family only if they agree on the ordering, which means the conflict is detectable and not silent.

2. **`history.jsonl` is the only source of truth.** There is no in-memory state between ticks. There is no daemon. There is no scheduler thread. The dispatcher is stateless and idempotent across restarts. If you delete the lock file, nothing breaks. There is no lock file. If you `kill -9` a sub-agent mid-work, the next tick reads `history.jsonl`, sees a claim with no matching result older than some threshold (I use 30 minutes), treats it as abandoned, and proceeds.

3. **Families are partitioned by repo.** This is the key design constraint. If I ever add a family that needs to touch a repo another family already touches, the design breaks and I will have to introduce real locking. So far I have resisted that temptation by being honest about what each family is for. When I was tempted to make a `cross-repo-summary` family, I instead made it a *post-processing* step that runs after a regular tick, reads `history.jsonl`, and writes its own summary repo. It is not a family. It is downstream.

## The parallel-tick problem, named

I have started calling this the **parallel-tick problem**, and I think it shows up in any cron-driven automation that grows past one cron entry. The shape is:

- You have N independent units of work.
- They share some resource (a file, a repo, an API quota, a database table).
- Each tick decides what to do based on observable state.
- Two ticks can be in flight at once.
- The decision-making code reads state that is stale by the duration of the previous in-flight tick.

The standard solution is a lock. Locks work but they serialize. The better solution, where it's available, is to **partition the work so that decisions made in parallel cannot collide**. Repo-disjointness is one way to do that. Topic-disjoint queues are another. Sharded customer IDs are another. The thing they have in common is that the partitioning structure is part of the *problem*, not part of the dispatcher.

If you can't find a partitioning structure in your problem, you genuinely do need a lock — but you should be suspicious that you have not looked hard enough.

## Tie-breaks deserve their own paragraph

I kept underestimating how important deterministic tie-breaking is. In the lock-file version of the dispatcher, ties didn't matter, because only one tick was deciding at a time. In the partitioned version, ties matter constantly, because two ticks may be deciding at the same time and you want them to agree (so they don't both pick the same family) or to disagree predictably (so one of them deterministically loses and bails out).

The cleanest tie-break I've found is "least recently completed family from the available set." It has three nice properties:

1. It is a pure function of `history.jsonl` — no clocks, no PIDs, no random.
2. It tends to balance work across families over time, which is what I want anyway.
3. When ties are genuinely tied (e.g., two families have never been completed), the ordering of `families` itself is the final tie-break, which is deterministic and reviewable.

What I avoided was tie-breaks that depend on the tick's own state — PID, timestamp, hostname. Those produce *non-deterministic* tie-breaks, which means two parallel ticks can both think they "won" the tie. Then you're back to needing a lock.

## What `history.jsonl` is and isn't

It took me a while to be honest about what `history.jsonl` is. It is not a queue. It is not a message bus. It is not a scheduler. It is an **append-only log of events that have happened**, where "events" are claims (a tick decided to do X) and results (a tick finished doing X, with this outcome). It is the only persistent state in the dispatcher.

What `history.jsonl` is *not* good at:

- It is not good at "what is happening right now," because there is always a window between a claim and a result during which the world is in flight.
- It is not good at coordinating two parties, because there is no read-modify-write primitive across processes.
- It is not good at fast lookups, because every read is a full scan. (For my volumes — a few hundred entries — this is fine. At a million entries I would need an index.)

What it *is* very good at:

- Auditing. Every decision the dispatcher has ever made is in there.
- Replay. I can re-derive the dispatcher's view of the world from any point by truncating and re-reading.
- Cross-process visibility. Any tool, any sub-agent, any human can `cat` it and see exactly what is going on.

The lesson, which keeps showing up in my work: **append-only logs are extremely good at recording what happened, and extremely bad at coordinating what should happen next.** If you treat them as the latter, you will rebuild a lock manager in shell.

## What I would do differently

I would have started with the partitioning question. The first version of the dispatcher was a round-robin over a list of strings, with no awareness that those strings corresponded to disjoint repos. If I had noticed the partitioning structure on day one, I would have skipped the entire lock-file detour and saved myself a week.

The deeper lesson is that the dispatcher is not really a scheduler. It is a coordination artifact between a cron timer, a set of sub-agents, and a set of repos. The "coordination" part is interesting; the "scheduling" part is trivially `min(available, key=last_completed)`. I was treating the trivial part as the hard part, and the hard part as a footnote.

## Links

- POSIX `O_APPEND` atomicity guarantees (read the actual man page for `write(2)`, not the Stack Overflow summary)
- The launchd `KeepAlive` and `StartInterval` keys in `man launchd.plist`
- The CRDT literature on partitioning vs. locking — this dispatcher is essentially a degenerate CRDT where the conflict-free property comes from disjoint state, not from merge functions
