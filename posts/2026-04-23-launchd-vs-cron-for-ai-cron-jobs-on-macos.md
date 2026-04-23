---
title: "launchd vs cron for AI cron jobs on macOS: pick the boring one for the right reason"
date: 2026-04-23
tags: [macos, launchd, cron, scheduling, automation, agents]
est_reading_time: 13 min
---

## TL;DR

If you are running scheduled jobs on a Mac, launchd is the right answer roughly 90 percent of the time, even though cron still works and feels simpler. The reason is not "Apple says so." The reason is that launchd handles three things cron cannot: it survives missed intervals (laptop closed lid during the scheduled time), it owns the process supervision (auto-restart on crash, log rotation, environment), and it is the system that all other macOS automation already uses, so you can read your own jobs alongside the system's. Cron survives on macOS but is deprecated, undocumented, and silently breaks in subtle ways across OS upgrades. This post is the practical migration guide from cron to launchd for the kind of small AI-related cron jobs that pile up on a working developer's machine, with templates for the four patterns that cover most real workloads.

The framing: launchd is a worse user experience but a better operating model. Pay the upfront ergonomics cost and stop debugging weird "why didn't my job run" mysteries.

## Section 1: why this question keeps coming up

Cron is universal. Anyone who has used a Unix system for more than a year knows the syntax (`* * * * *`), knows where the crontab lives (`crontab -e`), and can write a five-line shell job in 30 seconds. Cron is the dictionary definition of "boring technology." The instinct, when you have a small Python script that needs to run every 15 minutes, is to reach for cron.

On macOS, that instinct is wrong, and the reasons take a while to discover.

The first reason: macOS laptops sleep. A cron job scheduled for 03:00 will not run if the lid is closed at 03:00. There is no make-up run; the slot is just missed. For a job that aggregates the day's data, missing the run means missing the day. For a job that pulls fresh API tokens, missing the run means a stale token until the next scheduled time, which often means tomorrow.

The second reason: cron's environment is minimal and mysterious. The PATH is short, no shell rc files are sourced, and tools installed via Homebrew are typically not on the PATH. Half the cron failures I have ever debugged on macOS came down to "command not found" because the script worked in my interactive shell but not in cron's environment. You can fix this by sourcing `~/.zshrc` from the cron command, but then you inherit interactive-shell-only quirks.

The third reason: cron has no logging by default. If your job crashes, the output goes to a system mail spool that you have to look up to find, which on modern macOS is increasingly inaccessible. You end up wrapping every cron job in `>> /tmp/job.log 2>&1`, which works but means every cron job has its own ad-hoc log convention.

launchd solves all three. Missed runs can be configured to fire on next wake. Environment is explicitly declared in the job's plist. stdout and stderr are redirected to file paths you control. The cost is the plist syntax, which is XML and verbose.

## Section 2: a minimal launchd job

The smallest useful launchd job is a plist file in `~/Library/LaunchAgents/` (for user-level jobs) or `/Library/LaunchDaemons/` (for system-level jobs that run as root). For AI cron jobs you almost always want the user-level path.

Here is a plist that runs a Python script every 15 minutes:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.example.token-rollup</string>

    <key>ProgramArguments</key>
    <array>
        <string>/opt/homebrew/bin/python3</string>
        <string>/Users/me/scripts/token-rollup.py</string>
    </array>

    <key>StartInterval</key>
    <integer>900</integer>

    <key>RunAtLoad</key>
    <true/>

    <key>StandardOutPath</key>
    <string>/Users/me/Library/Logs/token-rollup.out.log</string>
    <key>StandardErrorPath</key>
    <string>/Users/me/Library/Logs/token-rollup.err.log</string>

    <key>EnvironmentVariables</key>
    <dict>
        <key>PATH</key>
        <string>/opt/homebrew/bin:/usr/bin:/bin</string>
        <key>HOME</key>
        <string>/Users/me</string>
    </dict>
</dict>
</plist>
```

Save as `~/Library/LaunchAgents/com.example.token-rollup.plist`, then load it:

```bash
launchctl unload ~/Library/LaunchAgents/com.example.token-rollup.plist 2>/dev/null
launchctl load ~/Library/LaunchAgents/com.example.token-rollup.plist
launchctl list | grep com.example
```

The unload-then-load is the standard pattern for picking up edits. There is a newer `launchctl bootstrap`/`bootout` API on Catalina and later, but `load`/`unload` still works and is simpler.

Compared to the cron equivalent (`*/15 * * * * /opt/homebrew/bin/python3 /Users/me/scripts/token-rollup.py >> /Users/me/Library/Logs/token-rollup.log 2>&1`), the plist is much more verbose. The verbosity buys you control: the log paths are real files, not appended to a stream, and the PATH is explicit, not inherited from a half-defined shell.

## Section 3: pattern one, the wake-up survivor

The single most useful launchd feature for laptop users: jobs that fire at next wake when their scheduled time was missed. The flag is `StartCalendarInterval` plus a normal sleep schedule.

```xml
<key>StartCalendarInterval</key>
<dict>
    <key>Hour</key>
    <integer>3</integer>
    <key>Minute</key>
    <integer>0</integer>
</dict>
```

This says "fire at 03:00 every day." If the system was asleep at 03:00, launchd will fire the job as soon as the system wakes up, instead of skipping the slot. Combined with the `RunAtLoad` flag, this means the job runs when you boot or when you re-load it, regardless of when the next 03:00 happens to be.

Cron's `cron`/`anacron` distinction is the analog on Linux. macOS does not ship `anacron`. launchd's calendar intervals are the macOS equivalent.

For my own daily aggregation jobs, this changes the failure mode from "job silently missed because I closed my laptop" to "job runs the next morning when I open the lid." Same data, no manual intervention required.

## Section 4: pattern two, the long-running daemon

Some AI workloads are not "fire periodically" but "run continuously, restart if you crash." Examples: a queue worker that consumes from a local Redis-style queue and dispatches model calls, a webhook receiver that listens on a port and forwards to a model. These are daemons, not cron jobs, and launchd handles them naturally.

```xml
<key>KeepAlive</key>
<true/>

<key>RunAtLoad</key>
<true/>

<key>ThrottleInterval</key>
<integer>10</integer>
```

`KeepAlive=true` means "if this process exits for any reason, restart it." `ThrottleInterval=10` means "do not restart more often than once per 10 seconds" (so a crash loop does not eat the CPU). For a daemon, omit `StartInterval` and `StartCalendarInterval`; the process is meant to run continuously.

You can also make `KeepAlive` conditional on file existence or network reachability. The full grammar is in `man launchd.plist`. For most uses, `<true/>` is enough.

This is the pattern that makes launchd worth learning even if you never use the cron-replacement features. Process supervision on macOS without launchd means writing your own pid-file dance, which is reliably worse than just learning the plist syntax.

## Section 5: pattern three, the every-N-seconds tight loop

For jobs that need to run frequently (every 30 seconds, every minute), the `StartInterval` field accepts any positive integer in seconds. Cron's minimum granularity is 1 minute; launchd has no such limit.

```xml
<key>StartInterval</key>
<integer>30</integer>
```

This runs the job every 30 seconds. Be careful with this: if the job takes longer than 30 seconds to run, launchd will not start a second instance until the first one finishes. So you do not need to add your own mutex to prevent overlap; launchd does it.

For an AI workload, the example I have is a script that polls a job queue every 30 seconds and dispatches model calls for any pending items. The script is intentionally short-lived (drain the queue, exit). launchd ensures it runs every 30 seconds, never overlaps with itself, and restarts if it crashes mid-drain.

## Section 6: pattern four, the on-demand triggered job

A pattern I underused for years: launchd can trigger a job not on a schedule but on a filesystem event. If a directory has a new file in it, run the job. This is `WatchPaths`.

```xml
<key>WatchPaths</key>
<array>
    <string>/Users/me/Drop/inbound</string>
</array>

<key>RunAtLoad</key>
<false/>
```

When a new file appears in `~/Drop/inbound` (or an existing one is modified), the job fires. For an AI workload, this is the "drop a transcript here and have it summarized" pattern: my script picks up the file, sends it to the model, writes the summary back, and moves the original to an archive directory.

There is a subtlety: `WatchPaths` fires on any modification to the directory, including the script's own writes if they happen in the same directory. To avoid an infinite loop, the script should write outputs to a different directory than the one being watched. Common mistake; easy to fix once you know about it.

## Section 7: the things cron does that launchd makes harder

Honesty section. launchd is not strictly better at every dimension.

The first thing cron is better at: ad-hoc edits. `crontab -e` opens an editor on a single file with all your jobs; you edit a line, save, and you are done. With launchd, each job is its own plist file, and editing requires the unload-load dance. For a developer who edits cron jobs frequently, launchd's friction adds up.

The second thing cron is better at: terse syntax. `*/15 * * * *` is shorter than the equivalent plist by about 50 lines. For a one-off "run this every 15 minutes," the cron syntax is more proportional to the task.

The third thing cron is better at: portability. The cron syntax is the same on macOS, Linux, BSD, and Solaris. A plist is macOS-only. If you maintain a script that needs to run on multiple OSes, cron lets you have one config; launchd does not.

I have ended up using launchd for everything I run more than once a week and accepting cron for one-off experiments. The line is fuzzy. The principle is: if the job is worth keeping, port it to launchd.

## Section 8: a tool to make the plist generation less painful

The verbosity of plists is mostly boilerplate. A small Python script that generates plists from a YAML config makes the friction roughly equivalent to crontab editing.

```python
#!/usr/bin/env python3
"""Generate a launchd plist from a YAML config and load it."""
import sys, yaml, plistlib, subprocess, os
from pathlib import Path

def make_plist(cfg: dict) -> dict:
    out = {
        "Label": cfg["label"],
        "ProgramArguments": cfg["program"],
        "RunAtLoad": cfg.get("run_at_load", True),
    }
    if "interval" in cfg:
        out["StartInterval"] = cfg["interval"]
    if "calendar" in cfg:
        out["StartCalendarInterval"] = cfg["calendar"]
    if "keep_alive" in cfg:
        out["KeepAlive"] = cfg["keep_alive"]
    if "watch_paths" in cfg:
        out["WatchPaths"] = cfg["watch_paths"]
    if "env" in cfg:
        out["EnvironmentVariables"] = cfg["env"]
    log_dir = Path(cfg.get("log_dir", "~/Library/Logs")).expanduser()
    log_dir.mkdir(parents=True, exist_ok=True)
    name = cfg["label"].split(".")[-1]
    out["StandardOutPath"] = str(log_dir / f"{name}.out.log")
    out["StandardErrorPath"] = str(log_dir / f"{name}.err.log")
    return out

def install(cfg_path: str):
    cfg = yaml.safe_load(Path(cfg_path).read_text())
    plist = make_plist(cfg)
    target = Path(f"~/Library/LaunchAgents/{cfg['label']}.plist").expanduser()
    target.write_bytes(plistlib.dumps(plist))
    subprocess.run(["launchctl", "unload", str(target)], stderr=subprocess.DEVNULL)
    subprocess.run(["launchctl", "load", str(target)], check=True)
    print(f"installed {target}")

if __name__ == "__main__":
    install(sys.argv[1])
```

Usage:

```yaml
# token-rollup.yaml
label: com.example.token-rollup
program:
  - /opt/homebrew/bin/python3
  - /Users/me/scripts/token-rollup.py
interval: 900
env:
  PATH: /opt/homebrew/bin:/usr/bin:/bin
```

```bash
python3 install-job.py token-rollup.yaml
```

The YAML is roughly the size of a crontab line plus a few labels. The script handles the boilerplate. After you build this once, the cost difference between cron and launchd is essentially zero on the editing side, while you keep all the launchd benefits on the runtime side.

## Section 9: debugging launchd jobs that do not run

launchd is famously cryptic when jobs fail. Three diagnostic commands cover most issues.

```bash
launchctl list | grep my-job-label
```

The first column is the PID (a dash if not currently running), the second is the last exit status (0 for healthy, non-zero for the most recent failure). If you see a non-zero exit status repeatedly, the job is failing.

```bash
launchctl print gui/$(id -u)/com.example.my-job-label
```

This dumps the full state of the job: when it last ran, how many times it has been spawned, the configured environment, and any pending throttle delays. If the job is supposed to be running but isn't, this tells you why.

```bash
log show --predicate 'subsystem == "com.apple.xpc.launchd"' --last 1h
```

The unified log has launchd's own messages about loading, scheduling, and erroring on jobs. Useful when the job never starts in the first place.

After three years of running launchd jobs, the failure modes I have catalogued: missing executable on PATH (fix the env block), permissions error on log path (fix the path), Python import errors at startup (test the script standalone first), and a once-only "job is throttled, retry in N seconds" which resolved itself.

Cron's failures are usually the same set, but harder to find. launchd's verbosity at debug time pays back the verbosity at config time.

## Section 10: when I still use cron

For honesty: I have not banned cron from my machine. Three uses where I still write cron lines:

The first: jobs that need to run on multiple OSes. A single `crontab` file is portable; a plist is not.

The second: throwaway one-shots. If I am testing whether a script behaves correctly when run periodically, a cron line takes 10 seconds and a plist takes 5 minutes. For things I will delete next week, cron wins.

The third: jobs explicitly tied to user login (cron's `@reboot` syntax). launchd has a `RunAtLoad` flag but only fires when the job is loaded, which is at user login for user-level jobs. For "run when the user logs in," both work; cron is shorter to write.

Outside those three, launchd has been the right answer for years. The main reason I am writing this post now is that every six months I rediscover that someone else's "I scheduled an AI batch job and it never runs" problem is a missed cron interval, and the fix is the same every time: move it to launchd, pay the verbosity cost once, and stop debugging the same class of bug forever.

## References

- `launchd.plist(5)` man page: https://www.manpagez.com/man/5/launchd.plist/
- Apple's launchd guide (archived but accurate): https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/CreatingLaunchdJobs.html
- `launchctl(1)` man page: https://www.manpagez.com/man/1/launchctl/
- "Mac OS X and the missing batch jobs" (long-form blog post on cron pitfalls): https://launchd.info/
- Unified logging on macOS: https://developer.apple.com/documentation/os/logging
