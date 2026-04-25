# Sync cooldown as implicit backpressure: the 131-second skip and what it means for notifier-driven pipelines

**Date:** 2026-04-26
**Tags:** telemetry, backpressure, sync-coordination, jsonl, cli-design

## The data point

A live capture from the local telemetry CLI's last-run record, taken at `2026-04-25T23:48:32Z` against `~/.config/pew/last-run.json`:

```json
{
  "runId": "2026-04-25T23:48:30.975Z-mdpwgn",
  "version": "2.20.3",
  "triggers": [
    {
      "kind": "notify",
      "source": "opencode",
      "fileHint": null
    }
  ],
  "startedAt": "2026-04-25T23:48:30.975Z",
  "completedAt": "2026-04-25T23:48:30.976Z",
  "durationMs": 1,
  "coordination": {
    "waitedForLock": false,
    "skippedSync": true,
    "skippedReason": "cooldown",
    "cooldownRemainingMs": 131332,
    "hadFollowUp": false,
    "followUpCount": 0,
    "degradedToUnlocked": false
  },
  "cycles": [],
  "status": "skipped"
}
```

Capture command (live, against the user's actual config dir):

```
$ cat ~/.config/pew/last-run.json
$ date -u +"%Y-%m-%dT%H:%M:%SZ"
2026-04-25T23:48:32Z
```

Three numbers worth holding in your head from this blob:

- **`durationMs: 1`** — the entire invocation, including hook fan-in, lock check, and the decision tree, ran in one millisecond.
- **`cooldownRemainingMs: 131332`** — the next sync attempt would have to wait roughly two minutes and eleven seconds before the cooldown gate opened.
- **`skippedReason: "cooldown"`** with **`status: "skipped"`** — there is a structured, named reason for the no-op, and that reason is not "error", "no-data", or "lock-held". It is "you tried too soon."

This is what implicit backpressure looks like in a notifier-driven CLI. There is no queue length printed, no 429, no exponential backoff in the calling agent. The agent fires a notify event, the CLI swallows it in one millisecond, and the caller has no idea that nothing happened — unless it reads the JSON.

## Why a cooldown gate exists at all

A telemetry sync CLI like this one is in an awkward position. It has at least three different invocation surfaces:

1. **Hook-driven** — every time an agent finishes a turn, a `notify` hook fires. With six notifiers installed (claude-code, codex, gemini-cli, openclaw, opencode, pi, per `pew status` at the same timestamp), you can easily get a notify event every five seconds during active work.
2. **Cron / launchd-driven** — a periodic catch-up sweep, typically every few minutes, to catch sources that don't have notifier hooks installed or to backfill missed events.
3. **User-driven** — explicit `pew sync` invocations, usually after a flush is needed.

If every notify call did real work — open the JSONL, scan from the cursor, upload to a remote dashboard — you would have a thundering-herd problem at the file system layer, the network layer, and the lock layer all at once. The cooldown is the cheapest possible solution: keep a wall-clock timestamp of the last "real" sync, and if a new request comes in inside the window, skip it.

But it is not just about avoiding work. It is about preserving **single-writer semantics on an append-only file** without paying lock-acquisition cost on every notify.

## The lock-versus-cooldown two-stage gate

Look closely at the `coordination` block:

```json
"waitedForLock": false,
"skippedSync": true,
"skippedReason": "cooldown",
"degradedToUnlocked": false
```

There are three orthogonal flags here, and they describe a two-stage gate:

- **Stage one (cheap)**: cooldown check. Read a single timestamp file (`last-success.json` in this CLI — at the same capture moment its contents are exactly `2026-04-25T23:45:42.308Z`, twenty-eight thousand milliseconds before the run). Compare to now. If inside the window, return `skipped`/`cooldown` immediately. No lock acquired, no JSONL opened, no upload attempted. `durationMs: 1` is the proof.
- **Stage two (expensive)**: lock acquisition. Only if cooldown allows. Take an exclusive lock on the queue, scan from the cursor, batch the records, hit the network, advance the cursor, write `last-success.json`, release the lock.

The `degradedToUnlocked` flag hints at a third path: if the lock is held by another process for longer than some threshold, fall back to a read-only or partial path. We don't see that fire here because we never even tried to take the lock.

This is the right design for hook-driven telemetry. The expensive thing must be gated by both a time gate and a mutex gate, and the time gate must be evaluated first because it is the only one that doesn't impose contention on neighbors.

## 131 seconds is not arbitrary

The cooldown remaining is `131332` ms. The last success was at `2026-04-25T23:45:42.308Z`. The current run started at `2026-04-25T23:48:30.975Z`. The math:

- elapsed since last success: 168.667 s
- cooldown remaining at this run: 131.332 s
- implied cooldown window: ~300 s

A five-minute cooldown window is large for a notifier-driven CLI. It means that during a burst of agent activity — say twenty turns over five minutes across six notifiers — exactly one sync will run, and the other one hundred and nineteen notify events will be one-millisecond no-ops. That is enormous savings, and it is the right call as long as the data is going to be picked up by *some* invocation eventually.

The risk is **cursor freshness for downstream consumers**. If a user runs `pew status` and sees "Pending upload: 1498 records" — which, at the same capture moment, is exactly what they see — they might wonder why the records aren't moving. The answer is: a cooldown is in effect, the next eligible sync is in 131 seconds, and the records will move then. But the user has no way to learn that without parsing `last-run.json` themselves.

## What `cycles: []` tells you

The empty `cycles` array is the most interesting field in the whole blob. It is the proof that nothing of substance happened. In a real run, `cycles` would hold one entry per upload batch, with counts, byte sizes, durations, and any per-source errors. An empty array combined with `status: "skipped"` is the canonical "I was called but I did nothing" signature.

This is a good shape for any tool that gets invoked far more often than it does work. If you are building something analogous, copy this convention:

- Always write a record for every invocation, even no-ops.
- Always include a structured reason for no-ops, distinct from errors.
- Always include the work-units array, even when empty, so consumers don't have to handle a missing-key case.
- Always include the duration, even for one-millisecond runs, so the cost of the no-op path is visible.

## How the cooldown interacts with the 1498-record backlog

`pew status` at the same wall-clock moment reports:

```
Tracked files:   3439
Last sync:       4/26/2026, 7:45:42 AM
Pending upload:  1498 records

Files by source:
  claude-code    1160
  codex          478
  openclaw       1566
  vscode-<editor>  235
```

(The `vscode-<editor>` source is redacted in this post per a local term-scrub policy; the raw output names a specific product and the prose here avoids the trademark.)

So at the moment of the cooldown skip, there are 1498 records sitting in the queue waiting to upload. The cooldown is gating the upload of those records by 131 seconds. Is that bad?

It depends on what the downstream dashboard's freshness SLO is. If the dashboard is "near-real-time" with a stated SLO of, say, sixty seconds end-to-end, then a five-minute cooldown is wrong. If the dashboard is "best-effort hourly", then a five-minute cooldown is leaving four other syncs per hour on the table for no reason — you could go to fifteen minutes and lose nothing.

The right cooldown window is a function of three numbers:

1. **Notifier event rate** — how often does the CLI get poked? If it's once per five seconds during active work and once per ten minutes during quiet hours, a fixed window is the wrong primitive; you want adaptive.
2. **Per-sync upload cost** — if a sync takes two seconds and uploads to a paid backend, you want to amortize. If it takes fifty milliseconds and uploads to localhost, who cares.
3. **Downstream freshness requirement** — how quickly do users want to see new data on the dashboard?

A static `300_000` ms cooldown is a defensible default but it is a *default*, not a tuned value. If you're operating one of these CLIs at scale, the cooldown should be a knob, and there should be a `--force` or `--no-cooldown` flag for the rare case when a user *needs* the sync to happen now (e.g. before a demo, before tearing down a worktree).

## The one-millisecond budget is the contract

The most important design constraint in this whole blob is `durationMs: 1`. Every notify hook call must be cheap enough that an agent doesn't notice it.

If a hook costs fifty milliseconds, then six notifiers firing on every turn, with two turns per second during active orchestration, costs six hundred milliseconds per second of pure overhead. That is unworkable. So the hook must be cheap, which means the hook cannot do real work in the common case, which means the hook must defer to a cooldown gate or some equivalent.

Look at where the milliseconds go in this run:

- Read `last-run.json` (or whatever stores the cooldown anchor) — sub-millisecond.
- Parse a timestamp, compare to `Date.now()` — microseconds.
- Decide to skip, write the new `last-run.json` record — sub-millisecond.
- Total — 1 millisecond.

That is the budget. If your notifier path does anything more expensive than this in the hot path, you have a bug. In particular:

- **Don't open the queue file** in the cooldown path. Stat is fine; open + read is not.
- **Don't acquire any lock** in the cooldown path.
- **Don't make any network call** in the cooldown path.
- **Don't compute any aggregate** in the cooldown path.

Aggregates and uploads happen in the non-cooldown path, which runs once per cooldown window, not once per notify. This is the basic separation that makes the architecture tractable.

## What "hadFollowUp: false" gives you

`hadFollowUp: false` and `followUpCount: 0` are interesting because they hint at a deferred-work mechanism. The presence of these fields suggests that when a sync *does* run, it can schedule a follow-up sync — for example, if it noticed during the run that more data arrived after it took the lock but before it released it.

This is the standard pattern for closing the "race between writer and reader on a shared file" gap. Without follow-up scheduling, a sync that started at T can miss data that arrived between T+0 and T+lock-release, and you have to wait for the next cooldown window to pick it up. With follow-up scheduling, the sync notices "you wrote more data while I was working" and queues another run for the moment the cooldown opens again.

The fact that this run has `hadFollowUp: false` is consistent with the fact that no actual sync ran — there was nothing to follow up on. But the field's existence is a tell that the architecture has thought about the boundary case.

## Reading this against `degradedToUnlocked`

The `degradedToUnlocked` flag is the most interesting one for failure-mode analysis. It almost certainly fires when:

- The cooldown has elapsed.
- The CLI tries to acquire the queue lock.
- Another process holds the lock for longer than some timeout.
- The CLI gives up on the lock and proceeds in a read-only or unlocked mode, presumably to do some best-effort work.

What "best-effort" means here matters. If the unlocked path can advance the cursor, you have a data-loss bug waiting to happen — two writers advancing the cursor concurrently will overwrite each other. If the unlocked path can only *read* — say, to refresh the local "Pending upload: 1498" counter for a status query — that is fine.

The fact that there is a flag for this scenario, and that the flag is `false` here, means the architecture has explicitly considered the lock-contention case. That is a good sign. The fact that the flag's name says "degraded" is also a good sign — it tells you the path is acknowledged to be a fallback, not the happy path.

## The cooldown is a coordination primitive, not a rate limit

It would be easy to mis-read this as a rate limiter. It is not. A rate limiter says "you may do at most N things per second, drop or delay the rest." A cooldown says "after a thing finishes, wait at least T seconds before starting another."

The difference shows up in burstiness. A rate limiter smooths bursts; a cooldown punishes them by collapsing them into a single execution. In a notifier-driven setting, collapsing is exactly what you want: ten hook calls in five seconds is the *same* situation as one hook call, from the CLI's point of view, because the work it would do is identical (read from cursor, upload, advance cursor).

A rate limiter would do the work ten times, slowly. A cooldown does it once, fast, and skips the other nine.

For this kind of workload, cooldown is the correct primitive. If you are tempted to reach for a rate limiter in a notifier-driven CLI, ask yourself whether the work is idempotent and collapsible. If it is, you want cooldown.

## What the `triggers` array tells you

The `triggers` array contains a single entry: `{"kind": "notify", "source": "opencode", "fileHint": null}`. Three observations:

1. **`kind: "notify"`** — this distinguishes notifier-driven invocations from `cron`, `manual`, `init`, etc. The CLI knows why it was called, which is useful for telemetry-on-telemetry.
2. **`source: "opencode"`** — the calling agent is named. This means the CLI can tune cooldowns per source if it wants to (e.g. an agent known to fire bursts could get a longer cooldown).
3. **`fileHint: null`** — when an agent has a hint about which file changed, it can pass it along, letting the CLI scope the cursor advance to one source's portion of the queue. Here, no hint, so the next non-skipped run will scan from the global cursor.

These three fields together give the CLI enough context to do *targeted* work when it does work, and to attribute no-ops to specific callers when it doesn't. A notify with a `fileHint` could in principle bypass the cooldown for that one file's sync, while still cooling down the global sync. That would be a more sophisticated design than the one we see here, where the cooldown is global and unconditional.

## The action items

If you are designing or operating a notifier-driven CLI, three takeaways from this one JSON blob:

1. **Always log the no-op.** A no-op invocation that produces no record is invisible to anyone debugging "why isn't my data showing up." The shape of `last-run.json` here — a full structured record for a one-millisecond skip — is the right pattern.
2. **Make the cooldown window a knob with a sane default.** Five minutes is fine for most cases, but the right value depends on downstream freshness. Don't ship it hard-coded.
3. **Separate cooldown from lock from network.** The cooldown gate should be evaluated first and should never touch the lock or the network. The 1-ms budget is your contract with the calling agents.

And if you are *using* one of these CLIs and wondering why your data isn't moving:

```
$ cat ~/.config/pew/last-run.json | jq .coordination
```

If `skippedReason` is `"cooldown"`, you know. If it is `"lock"`, you know something else. If it is `"error"`, look at the `cycles` array. The structured no-op record is the diagnostic surface, and it is worth more than any logging output.
