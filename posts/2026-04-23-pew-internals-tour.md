---
title: "What I learned reverse-engineering the pew CLI on-disk layout"
date: 2026-04-23
tags: [pew, internals, sqlite, on-disk-format, observability]
est_reading_time: 9 min
---

`pew` is the closest thing I have to a daily logbook. It tracks coding-agent sessions, token counts, model usage, and the bursts and quiet stretches of a working day. I wanted a few visualizations that the built-in `pew status` doesn't give — burndown curves, model mix per hour, idle-vs-active ratios — and the path to those visualizations runs straight through the on-disk layout.

The official docs cover the user-facing commands. They do not document the schema. So I read the SQLite files. This is what I found, what surprised me, and what I now build on top of.

## The directory layout

Everything lives under `~/.pew/`. On a long-running install (mine has 11 weeks of data) it looks like this:

```
~/.pew/
├── config.toml           # user config: model aliases, defaults, output prefs
├── pew.db                # the main SQLite DB. ~40MB on my machine.
├── runs/                 # per-run directories. Unbounded growth. See below.
│   ├── 2026-04-23T09-14-02_a3f9b1/
│   │   ├── meta.json
│   │   ├── stdout.log
│   │   ├── stderr.log
│   │   └── tool_calls.jsonl
│   ├── 2026-04-23T09-22-47_b81dc4/
│   │   └── ...
│   └── ...               # mine has 38,412 directories.
├── cache/                # response cache. Safe to delete. ~200MB-ish typically.
└── locks/                # process locks for concurrent pew invocations.
```

The structure is honest about its boundaries: `pew.db` is the source of truth for queries; `runs/` is the audit trail; `cache/` is throwaway. The locks/ directory caught me out once when an aborted `pew run` left a stale lock and the next invocation hung silently. Removing `locks/*.lock` after verifying no `pew` process is alive is safe.

## The schema

`sqlite3 ~/.pew/pew.db .schema` gives you the table list. The interesting tables are these (simplified, with non-essential columns elided):

```sql
CREATE TABLE runs (
    run_id TEXT PRIMARY KEY,           -- matches the runs/ dir suffix
    started_at INTEGER NOT NULL,       -- unix epoch seconds
    ended_at INTEGER,                  -- NULL while running
    model TEXT NOT NULL,
    profile TEXT,                      -- which pew profile was active
    parent_run_id TEXT,                -- for nested invocations
    exit_code INTEGER
);

CREATE TABLE turns (
    turn_id INTEGER PRIMARY KEY AUTOINCREMENT,
    run_id TEXT NOT NULL REFERENCES runs(run_id),
    turn_index INTEGER NOT NULL,
    started_at INTEGER NOT NULL,
    input_tokens INTEGER NOT NULL,
    output_tokens INTEGER NOT NULL,
    cache_read_tokens INTEGER DEFAULT 0,
    cache_write_tokens INTEGER DEFAULT 0,
    finish_reason TEXT
);

CREATE TABLE bucket_30m (
    hour_start INTEGER PRIMARY KEY,    -- DESPITE THE NAME: 30-minute bin start
    input_tokens INTEGER NOT NULL,
    output_tokens INTEGER NOT NULL,
    cache_read_tokens INTEGER NOT NULL,
    cache_write_tokens INTEGER NOT NULL,
    turn_count INTEGER NOT NULL,
    run_count INTEGER NOT NULL
);

CREATE TABLE cursors (
    name TEXT PRIMARY KEY,             -- 'last_seen', 'last_billed', 'last_synced'
    value INTEGER NOT NULL,            -- unix epoch seconds
    updated_at INTEGER NOT NULL
);

CREATE TABLE cooldowns (
    key TEXT PRIMARY KEY,              -- e.g. 'rate_limit:claude:2026-04-23'
    until INTEGER NOT NULL,            -- unix epoch seconds
    reason TEXT
);
```

A few things in here repay attention.

### The `hour_start` field is actually a 30-minute bin

This is the gotcha that cost me an afternoon. The column is named `hour_start` and contains a unix timestamp. The natural assumption is that `hour_start` is the start of a 1-hour bucket. It is not. It is the start of a *30-minute* bucket. You can verify this by checking that adjacent rows differ by 1800 seconds, not 3600:

```sql
sqlite> SELECT hour_start, hour_start - LAG(hour_start) OVER (ORDER BY hour_start) AS delta
   ...> FROM bucket_30m
   ...> ORDER BY hour_start DESC LIMIT 5;
1745411400|1800
1745409600|1800
1745407800|1800
1745406000|1800
1745404200|1800
```

Every gap is 1800. The column name is a historical artifact — `pew` started life with hourly buckets and was switched to 30-minute resolution in (I'm guessing from the git tags) version 0.4 or 0.5, and the column was never renamed because that would have required a migration on existing installs. Anything you build on top of `bucket_30m` has to multiply expectations by two if you were thinking in hours.

The practical consequence: when you're computing "tokens per hour," you sum two consecutive rows, not one. And when you're deciding bucket alignment, the boundary is on the half-hour (`HH:00:00` and `HH:30:00`), not arbitrary.

### Three cursor variants, three meanings

`cursors` has three named rows. The names are not self-explanatory:

- `last_seen` — the latest turn timestamp that the local indexer has processed. Used by `pew status` to know what to show.
- `last_billed` — the latest turn timestamp that has been reconciled against a billing-period anchor. Used by `pew cost`. Lags `last_seen` by up to a few minutes because billing reconciliation runs on a slower cadence.
- `last_synced` — the latest turn timestamp that has been pushed to a remote store, if you have one configured. On a stock install with no remote, this stays at 0 forever and is harmless.

If you write tools against `pew.db`, use `last_seen` as your high-water mark. Using `last_billed` will silently drop the most recent few turns; using `last_synced` will give you nothing if remote sync is off.

### The cooldown mechanism is a flat key-value table

When `pew` hits a rate limit, it writes a row into `cooldowns` with `until` set to the projected unblock time. Subsequent invocations check this table before issuing requests. If you've ever seen `pew run` exit immediately with "in cooldown until ...", this is where it reads from.

The keys are hierarchical strings (`rate_limit:<provider>:<scope>`), but there's no actual hierarchy enforcement — it's just string matching. That means you can manually clear a cooldown if you know it's stale:

```bash
sqlite3 ~/.pew/pew.db "DELETE FROM cooldowns WHERE key LIKE 'rate_limit:claude:%';"
```

I have done this exactly twice in eleven weeks, both times when a provider's status page was lying about being healthy and `pew` had cached an old back-off. Don't make a habit of it.

## The `runs/` directory grows without bound

Nothing in `pew` cleans up `runs/`. Each invocation writes a directory; the directory survives forever. On my machine, the count is 38,412 and growing by ~600/day. The total disk footprint is about 4.2 GB, dominated by `stdout.log` and `tool_calls.jsonl` from long agent sessions.

This is a defensible default — you may want to inspect a session from three months ago — but it's worth knowing. If you start running out of inodes, the cleanup is straightforward:

```bash
# delete run directories older than 30 days, dry-run first
find ~/.pew/runs -maxdepth 1 -type d -mtime +30 -name '20*' -print | head
# then for real
find ~/.pew/runs -maxdepth 1 -type d -mtime +30 -name '20*' -exec rm -rf {} +
```

The `pew.db` aggregate tables are unaffected — those rows don't reference the `runs/` directory by foreign key, only by ID string. You lose the per-turn audit trail for old sessions but keep all the bucketed stats.

## What I built on top

All of the above is documented (sort of) in `pew-insights`, the toolkit I'm using as the host for these spelunking notes. It exposes a `pew-insights query` subcommand that gives you the right cursor (`last_seen`), the right bucket size (30 minutes), and a few derived views I've found useful: hourly burn-rate, model-mix per day, cache-hit-rate over time, and idle-gap detection. The point is less "use my tool" and more "if you read your own pew DB, here are the four traps to skip."

## What I would do differently

I should have started by asking the maintainer for a schema dump before reading the bytes. I didn't, and I burned a half-day on the `hour_start` 30-minute trap. The rule is: if a field name doesn't match the unit it stores, it is a historical artifact, not your misreading. Trust the data over the column name.

## Links

- pew CLI repository (upstream): search "pew CLI" on GitHub; multiple forks exist, the active one tags releases as `v0.x`.
- SQLite WAL mode docs (relevant if you read pew.db while pew is running): https://www.sqlite.org/wal.html
