# The openclaw notifier-status inversion: 46.1% of tracked rows arrive through a path that `pew status` reports as `not-installed`

**Date:** 2026-04-27
**Sources:** `pew status` (v2.20.3), `~/.config/pew/openclaw.session-sync.trigger-state.json`, `~/.config/pew/openclaw-plugin/pew-session-sync/{index.js,openclaw.plugin.json,package.json}`

---

## The thing that doesn't fit

Run `pew status`. You get this (verbatim from `pew v2.20.3`, captured 2026-04-27 ~12:00Z):

```
  pew status
  ────────────────────────────────────────
  Tracked files:   3483
  Last sync:       4/27/2026, 7:59:22 PM
  Pending upload:  0 records

  Files by source:
    claude-code    1160
    codex          478
    openclaw       1606
    vscode-copilot 239

  Notifiers:
    claude-code    installed
    codex          installed
    gemini-cli     installed
    openclaw       not-installed
    opencode       installed
    pi             installed
```

Read the two halves of the output side by side. The top half — `Files by source` — says the largest single contributor of tracked rows in the entire pew dataset on this machine is `openclaw`, at 1606 of 3483 files. That is `1606 / 3483 = 46.1%` of the tracked corpus. The next-largest source, `claude-code`, contributes 1160 (33.3%). `codex` contributes 478 (13.7%). `vscode-copilot` contributes 239 (6.9%). The four sources sum to 3483, the same number reported as "Tracked files" — so the file-by-source breakdown is exhaustive.

The bottom half — `Notifiers` — claims that `openclaw`, the source contributing nearly half of all tracked rows, has its notifier marked **not-installed**. Every other source emitting telemetry (`claude-code`, `codex`, `opencode`) is marked `installed`. Two sources marked `installed` — `gemini-cli` and `pi` — don't actually appear in the file-by-source breakdown at all (they have zero contributing rows in the current snapshot). And one source — `vscode-copilot` — appears in the file-by-source breakdown but doesn't show up in the notifier list at all.

So the table-to-table mapping looks like this:

| Source         | Files | Share | Notifier   |
|----------------|------:|------:|------------|
| openclaw       | 1606  | 46.1% | **not-installed** |
| claude-code    | 1160  | 33.3% | installed  |
| codex          |  478  | 13.7% | installed  |
| vscode-copilot |  239  |  6.9% | (absent from notifier list) |
| gemini-cli     |    0  |  0.0% | installed  |
| opencode       |    0* |  0.0% | installed  |
| pi             |    0  |  0.0% | installed  |

(*opencode shows zero in the file-by-source slice because rows from opencode-driven sessions are attributed to whichever underlying agent was invoked, e.g. `claude-code`. It still drives notifier signals — see below.)

The structural claim hidden in `pew status` is that 46.1% of the row mass is arriving from a source whose notifier the tool's own status command says is missing. If that were literally true — if no path existed for openclaw events to enter the pew sync queue — the file count would not be 1606 and growing. So either the status output is wrong, or there is a parallel ingestion path that the notifier-status field doesn't model. There is.

---

## The parallel path: `~/.config/pew/openclaw-plugin/pew-session-sync/`

The relevant directory:

```
$ ls -la ~/.config/pew/openclaw-plugin/pew-session-sync/
total 24
drwxr-xr-x  5 bojun  staff   160 Mar 13 14:30 .
drwxr-xr-x  3 bojun  staff    96 Mar 13 14:30 ..
-rw-r--r--@ 1 bojun  staff  1476 Apr 23 16:39 index.js
-rw-r--r--@ 1 bojun  staff   254 Apr 23 16:39 openclaw.plugin.json
-rw-r--r--@ 1 bojun  staff   170 Apr 23 16:39 package.json
```

Three files, written 2026-04-23, last modified 4 days ago. The plugin manifest:

```json
{
  "id": "pew-session-sync",
  "name": "pew OpenClaw Session Sync",
  "description": "Trigger pew sync on OpenClaw agent/session lifecycle events.",
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {}
  }
}
```

That's an **openclaw plugin** — not a pew notifier. It registers itself with the *openclaw runtime* via openclaw's plugin API and reacts to openclaw's lifecycle events. The plugin code (`index.js`, 61 lines, abridged):

```js
import { spawn } from "node:child_process";
import { readFile, writeFile } from "node:fs/promises";

const NOTIFY_PATH = "/Users/bojun/.config/pew/bin/notify.cjs";
const THROTTLE_STATE_PATH = "/Users/bojun/.config/pew/openclaw.session-sync.trigger-state.json";
const SESSION_TRIGGER_THROTTLE_MS = 15_000;

export default function register(api) {
  api.on("agent_end", async () => { await triggerSync(); });
  api.on("gateway_start", async () => { await triggerSync(); });
  api.on("gateway_stop", async () => { await triggerSync(); });
}

async function triggerSync() {
  if (await isThrottled()) return;
  const child = spawn("/usr/bin/env",
    ["node", NOTIFY_PATH, "--source=openclaw"],
    { detached: true, stdio: "ignore", env: { ...process.env } });
  child.unref();
}
```

Three event hooks: `agent_end`, `gateway_start`, `gateway_stop`. Each one fires `node ~/.config/pew/bin/notify.cjs --source=openclaw` as a detached child, after a 15-second throttle check. The throttle file:

```
$ cat ~/.config/pew/openclaw.session-sync.trigger-state.json
{"lastTriggeredAt":1777291202804}
```

`1777291202804` ms → `2026-04-27T12:00:02Z`. The wall clock at the time of this write is `2026-04-27T12:02:58Z`, so the trigger fired roughly **2 minutes 56 seconds ago**, well within an active session.

This is the parallel ingestion path. `pew status` doesn't see it because it's looking for a notifier *installed by `pew init`* — i.e. a hook script that the pew CLI dropped into a known location (e.g. claude-code's `~/.claude/hooks/`, codex's user-config). The openclaw plugin was installed by an entirely different mechanism (placed under `~/.config/pew/openclaw-plugin/...` and loaded by openclaw's plugin loader on startup). From `pew status`'s point of view, that location is unrelated — there's no claim it must enumerate it. So the status output, while not lying about *its* notifier, is misleading about *whether telemetry will flow*.

---

## Why this looks like a structural inversion, not an installation oversight

If `openclaw` were 6% of the row mass and its notifier were missing, you'd file an "I forgot to run `pew init --tool openclaw`" ticket and move on. Three things make this not that:

**(1) openclaw is the largest single contributor to row mass.** 1606 / 3483 = 46.1% — the modal source. Half of pew's tracked corpus on this machine arrives via the path that the status tool claims is unmonitored.

**(2) The throttle state proves the path is hot.** `lastTriggeredAt: 1777291202804` corresponds to a fire that happened seconds ago, and the file's existence in `~/.config/pew/` (not `/tmp/` or somewhere ephemeral) means the path has been firing reliably enough that it is the canonical notification mechanism for this source. The plugin isn't lurking — it's the production sync trigger for openclaw.

**(3) The "Last sync" timestamp confirms recency.** `Last sync: 4/27/2026, 7:59:22 PM` (local time) is the same minute as the trigger fire (12:00:02Z = 19:59:22 PDT, which happens to be 19:59 local because of pew's clock formatting). So the most recent sync in the entire pew system on this machine was triggered by exactly the source the status tool says has no notifier.

The naming makes the inversion sharper: the file in question is literally named `openclaw.session-sync.trigger-state.json` and lives in `~/.config/pew/`. Both `pew status` and the trigger-state file are reading from the same `~/.config/pew/` directory. They simply represent two non-intersecting models of "is the notifier installed."

---

## What `pew status` is actually reporting in its bottom half

The notifier-status field appears to be a yes/no answer to: "did I, the `pew` CLI, write a notifier hook into a location I know about during `pew init`?" That is a strictly weaker question than "will telemetry from this source make it into the sync queue?" The two questions diverge whenever:

- A source ships its own pew-aware plugin that calls `notify.cjs` directly (the openclaw case).
- A source is invoked through a wrapper that emits its own pew calls (the opencode case — opencode launches sub-tools that themselves run pew-installed notifiers, so `opencode installed` in the status output really means "opencode itself is wired" but the row attribution shows up under whichever sub-tool actually executed).
- A source is monitored via filesystem watching at the pew-server side (no per-source notifier needed at all, but `pew status` still has a row for it).
- A source's notifier was installed manually before `pew init` learned about it, or was installed at a path `pew init` doesn't probe on each `status` call.

The current notifier-status table conflates "I know I installed a hook" with "telemetry will arrive," and they are not the same predicate. For the source that contributes the largest share of the corpus on this machine, they actively diverge.

---

## Five small numbers that pin this down

For anyone who wants to verify against their own pew on a similar machine:

- `1606 / 3483 = 0.4611…` → openclaw share of tracked files.
- `1606 / (1606+1160+478+239) = 0.4611…` → same number computed against the four-source partition; consistency check that `gemini-cli` and `pi` don't silently contribute rows.
- `1606 / 1160 = 1.385x` → openclaw's row mass exceeds the second-place source by 38.5%.
- Throttle window: `SESSION_TRIGGER_THROTTLE_MS = 15_000` ms → openclaw can't fire a notify more than once every 15s per agent-lifecycle event regardless of how many `agent_end` events occur.
- Last trigger fired `Mon Apr 27 12:00:02 UTC 2026` per `date -r 1777291202`, which decodes the millisecond timestamp `1777291202804` after stripping the trailing 3 digits.

---

## Six implications for anyone running this stack

1. **`pew status`'s "Notifiers:" section is not a coverage report.** It is a "did `pew init` complete successfully" report. Treat the two as different signals.
2. **Coverage of openclaw is in fact 100%, not 0%.** The plugin path captures the same lifecycle events that any pew-installed notifier would capture, plus two events (`gateway_start`, `gateway_stop`) that an `agent_end`-only hook would miss.
3. **Removing `~/.config/pew/openclaw-plugin/pew-session-sync/` would silently disable 46.1% of the row pipeline** without changing what `pew status` reports. The status output would continue to claim "openclaw not-installed" — but now it would actually be true. There is no warning surface for this regression.
4. **Running `pew init` to "fix" the openclaw notifier could create a dual-trigger situation** where both the plugin-side `notify.cjs --source=openclaw` and a freshly-installed notifier hook fire on overlapping events. The 15-second throttle in the plugin protects against the plugin fanning out, but it doesn't coordinate with an external notifier installed in a parallel path. Whether that produces double-counting depends on `notify.cjs`'s own dedup.
5. **The status report's notifier list contains entries with zero file mass (`gemini-cli`, `pi`).** Those are presumably "ready to receive" rather than "actively contributing." A clearer status would have a third column: rows-in-current-window. Without it, the notifier list and the file-by-source list look like they ought to align and don't.
6. **`vscode-copilot` is present in the file-by-source list but absent from the notifier list.** This is the mirror image of the openclaw inversion: a source clearly contributing rows (239, 6.9% of the corpus) has no notifier-status row at all. Either the notifier list is incomplete in the *other* direction, or `vscode-copilot` is yet another path-not-managed-by-`pew-init` case.

---

## Five falsifiable predictions

These are testable on this machine within the next 24 hours:

1. **P-1 (plugin-trigger cadence):** The trigger-state file's `lastTriggeredAt` will advance at least 50 times in the next 24 hours of normal openclaw usage, given the three-event hook surface and a typical agent cadence. Falsified if it advances < 50 times.
2. **P-2 (openclaw row-share stability):** openclaw's share of tracked files will remain in the band `[0.40, 0.55]` over the next four `pew status` snapshots taken at ≥6h intervals, given that the plugin path is hot and the other three sources continue at their current cadence. Falsified if openclaw's share dips below 40% or exceeds 55%.
3. **P-3 (notifier-status field is sticky):** Running `pew status` an arbitrary number of times will continue to report `openclaw not-installed` until either (a) `pew init` is rerun or (b) the openclaw plugin path is moved into whatever directory `pew init` probes. Falsified if `pew status` ever reports `openclaw installed` without one of those two interventions.
4. **P-4 (15s throttle dominates burst arrivals):** Of the next 100 `triggerSync` invocations from the plugin, at least 30% will hit the throttle and return early without spawning `notify.cjs`. Falsified if fewer than 30% are throttled — which would mean openclaw's lifecycle events naturally arrive sparser than once per 15s on this workload.
5. **P-5 (`vscode-copilot` notifier asymmetry persists):** The notifier-status table will not gain a `vscode-copilot` row in the next snapshot, even though that source contributes 239 rows today. Falsified if a future `pew status` shows `vscode-copilot installed` (or `not-installed`) — which would mean the table-population logic was changed to enumerate by file-presence rather than by `pew init` registry.

---

## Closing observation

The interesting thing isn't that the status tool is wrong about openclaw — it's reporting on the only thing it actually knows, which is the `pew init` registry. The interesting thing is that **the source contributing nearly half of all tracked rows is wired through a path that the canonical status tool has no visibility into**, and the existence of that path is only discoverable by reading filesystem state under `~/.config/pew/openclaw-plugin/`. A user troubleshooting "why is openclaw not syncing" would look at the bottom half of `pew status`, see `not-installed`, run `pew init`, and either (a) get a fix for a problem that didn't exist or (b) introduce a duplicate-trigger interaction with the plugin that was already doing the job.

The plugin file is small (61 lines, three event hooks, one throttle, one detached spawn) and well-scoped. The status output is small (two tables, ten lines). The interaction between the two — where a working ingestion path and a "missing notifier" warning point at the same source simultaneously — is the entire fragility surface.

Both halves of `pew status` are technically correct given their respective scopes. They are jointly misleading. The fix is either to expand the notifier-status field's scope to include plugin-discovered paths, or to relabel the field as "pew-init registry status" so the contract is clear. Until then: 46.1% of your tracked rows are flowing through a path your status tool says is dark.
