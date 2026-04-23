---
title: "Why your AI agent should write to JSONL, not SQLite"
date: 2026-04-24
tags: [agents, storage, jsonl, sqlite, telemetry]
est_reading_time: 8 min
---

## The problem

The first time you ship an agent that produces telemetry — tool calls, token counts, decisions, errors — you reach for a database. SQLite, usually, because it's a file and feels light. You write a schema, you write some inserts, you write a query layer, and three weeks later you are debugging a locked database while a daemon spins on a retry loop and you can't open the file in any tool that doesn't speak SQL.

The mistake is treating agent telemetry like transactional data. It isn't. It's append-only event data with a tiny number of writers and a tiny number of readers, and the right primitive for that is a file with one JSON object per line. The fancy database is, almost always, a premature optimization that buys you nothing and costs you the ability to `tail -f` your agent's brain.

## The setup

The shape of agent telemetry is almost always:

- One process writes events. Sometimes two. Almost never more than four.
- Events are immutable once written. You don't update them, you append corrections.
- Reads are mostly "give me the last N events" or "give me everything from today and let me grep."
- Total volume is bounded. A daemon that ticks every fifteen minutes writes ~100 events per day. Even a busy multi-agent system writes maybe 10K events per day. That's 5–20 MB of JSONL, uncompressed.
- Concurrency requirements are essentially zero. A single writer per file, opened in append mode, is enough.

Compare to the assumed shape of "needs a database":

- Many concurrent writers
- Mutable rows
- Complex queries with joins and aggregations
- Volume that won't fit in memory
- Strong consistency requirements across multiple readers

Agent telemetry has none of those properties. Picking SQLite is picking a tool whose entire value proposition is solving problems you don't have, and the cost of that mismatch is paid in lock contention, schema drift, and tooling impedance.

## What I tried

- **Attempt 1: SQLite with a clean schema.** Tables for `ticks`, `tool_calls`, `tokens`, `errors`. Foreign keys. Indices. A nice little ORM layer in Python. Worked great for two weeks. Then I added a second daemon that wanted to read the same database while the first was writing. SQLite's default locking gave me `database is locked` errors during long-running writes. Switched to WAL mode. Better. Then I wanted to inspect the database from a second machine over SSH, and `sqlite3` wasn't installed there, and I couldn't `cat` the file. Annoying but survivable.

- **Attempt 2: SQLite with WAL + a small read API.** Wrote a tiny HTTP server on top of the SQLite database so other tools could query it without taking locks. This worked. It also meant my "telemetry system" was now a service with its own deployment, monitoring, and failure modes. The server crashed once and took the daemon's visibility down with it for an hour. The agent kept working, which was good; I just couldn't see what it was doing, which was bad.

- **Attempt 3: SQLite + JSONL mirror.** Wrote events to both SQLite (for "real" queries) and JSONL (for grep). Used the JSONL one for almost everything. Realized after a week that I hadn't written a single SQL query against the database in that week. Deleted the SQLite half. This is what's running now and has been for months.

- **Attempt 4 (counterfactual): JSONL from day one.** What I should have done. The total code is maybe twenty lines. There is no schema migration, because there is no schema. There is no lock contention, because writes are atomic appends of <8 KB lines (POSIX guarantees writes that small are atomic). There is no tooling impedance, because every tool on every Unix system can read JSONL.

## What worked

The append-only JSONL pattern, in full:

```python
import json, os, time
from pathlib import Path

LOG = Path.home() / "Projects/Bojun-Vvibe/.daemon/state/history.jsonl"
LOG.parent.mkdir(parents=True, exist_ok=True)

def emit(event_type: str, **fields) -> None:
    record = {
        "ts": time.strftime("%Y-%m-%dT%H:%M:%SZ", time.gmtime()),
        "pid": os.getpid(),
        "type": event_type,
        **fields,
    }
    line = json.dumps(record, separators=(",", ":"), sort_keys=True) + "\n"
    # Open-append-close per write. Slow? Yes, ~50µs. Don't care.
    with LOG.open("a", encoding="utf-8") as f:
        f.write(line)
```

That is the entire writer. Reading is whatever you want:

```bash
# last 20 events
tail -20 history.jsonl | jq .

# all errors today
grep '"type":"error"' history.jsonl | jq 'select(.ts | startswith("'"$(date -u +%Y-%m-%d)"'"))'

# token spend by hour
jq -r 'select(.type=="tick") | "\(.ts[0:13]) \(.tokens)"' history.jsonl \
  | awk '{h[$1]+=$2} END{for (k in h) print k, h[k]}' \
  | sort
```

No locks. No service. No schema migration. The file is grep-able, tail-able, scp-able, and human-readable. If you need indexed reads later — and you almost certainly won't — DuckDB will read JSONL natively (`SELECT * FROM read_json_auto('history.jsonl')`) without you having to convert anything.

Three operational rules make this scale further than you'd expect:

1. **Rotate by day, not by size.** `history-2026-04-24.jsonl`, `history-2026-04-25.jsonl`. Cheap. Makes "today's events" a path instead of a query.

2. **Keep records under 8 KB.** POSIX guarantees atomic appends below `PIPE_BUF`, which is 4 KB on macOS and 8 KB on most Linuxes. Stay under 4 KB if you want portability. If a record is bigger than that — say, a model response you want to log — write the big payload to a separate file and reference it by path.

3. **Never edit past records.** If you need to correct something, append a correction event with a `corrects: <prior_ts>` field. The history is the audit trail; the audit trail does not get retconned.

## Why it worked (or: my current best guess)

JSONL wins because it matches the actual access pattern of agent telemetry, and SQLite loses because it solves problems agent telemetry does not have.

The deeper reason is that agent telemetry sits in a particular spot in the design space: very write-heavy, very low-concurrency, very read-rare-but-read-from-many-tools. That combination is unusual. Most data systems assume either high concurrency (web apps) or high read volume (analytics). Agent telemetry assumes neither. The thing it does need — "any tool can read this without me writing a client" — is exactly what plaintext line-delimited formats give you for free.

There's also a second-order effect that took me a while to notice. Because JSONL is grep-able from a shell, I actually look at the data. SQLite required me to open a REPL or write a query, and that friction was enough that I'd go days without inspecting what the daemon was doing. With JSONL, `tail -f history.jsonl | jq .` is two seconds away in any terminal, and I check on the agents constantly. Visibility you don't use is visibility you don't have.

## When you actually do need a database

I want to be honest about the boundary. There are agent-shaped systems where JSONL is the wrong answer, and conflating them with the common case is how you end up with bad advice:

- **Shared mutable state across many agents.** If five workers need to claim items from a queue without double-claiming, you need real locking, and that means SQLite (with `BEGIN IMMEDIATE`) or a real queue like Redis or NATS. JSONL doesn't help you here, and pretending it does will produce subtle double-processing bugs that you'll find weeks later in production.
- **Aggregations over months of history.** If your common query is "p95 latency per tool over the last 90 days, grouped by hour, joined with the model the call was routed to," you'll want columnar storage. DuckDB on top of rotated JSONL files handles this surprisingly well up to maybe a hundred million rows, but past that point you'll want to flatten into Parquet and query with something that knows about predicate pushdown.
- **Strong audit requirements.** If you need cryptographic chains, signed records, or tamper-evident logs — anything that has to survive an adversarial reader — plain JSONL is not enough. You can layer hash chains on top, but the layering is not free and you should know going in that you're building a small append-only database, not just logging.

The thing none of those cases describe is "I have one daemon and I want to know what it did." For that case, JSONL wins, and the temptation to add a database is almost always the temptation to build infrastructure instead of building the thing the agent is supposed to do.

## What I would do differently

Start with JSONL. Move to SQLite only when a specific query becomes painful enough to justify a schema, and even then consider DuckDB-over-JSONL first. The rule of thumb I use now: if the data is write-once and read-rarely, plaintext beats databases on every axis that matters for an agent.

The other thing: pick the rotation policy on day one. Adding rotation later means writing a one-time migration script for a problem that didn't need to exist, and one-time migration scripts have a way of becoming permanent fixtures in the codebase that nobody wants to touch.

## Links

- SQLite, "Appropriate Uses For SQLite": https://www.sqlite.org/whentouse.html (note the "checklist for choosing the right database engine" — agent telemetry fails several items)
- DuckDB, "JSON Loading": https://duckdb.org/docs/data/json/overview
- POSIX `write()` atomicity discussion: https://pubs.opengroup.org/onlinepubs/9699919799/functions/write.html
- jq manual: https://jqlang.github.io/jq/manual/
