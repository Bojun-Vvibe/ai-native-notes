---
title: "JSONL append-only pipelines: the boring backbone of agent telemetry"
date: 2026-04-23
tags: [jsonl, telemetry, observability, pipelines, data-engineering]
est_reading_time: 15 min
---

## TL;DR

Every interesting question I have ever asked about an agent fleet was answered by grepping a JSONL file. Not by querying a time-series database, not by opening Grafana, not by running a SQL query. Just `jq`, `awk`, and `grep` against a file that any process can append to without coordination. Append-only JSONL is the simplest durable pipeline shape that exists, it survives crashes, it survives concurrent writers, and it scales further than people expect, well into the gigabytes-per-day range, before you need anything more sophisticated. This post is the design rationale, the implementation patterns, the failure modes, and the day-to-day operating recipes for the JSONL pipelines that underpin the rest of my agent observability stack.

The header argument: choose the boring shape until it actually breaks, and the boring shape rarely breaks.

## Section 1: why append-only and why JSONL

A telemetry pipeline has a small number of hard requirements. It must accept writes from multiple processes without coordination, because the agents run independently. It must survive a process crash mid-write without corrupting the data already written. It must be queryable without standing up infrastructure. It must work over the next ten years without bit rot.

Append-only JSONL satisfies all four with no configuration. Multiple writers append to the same file using the OS's atomic write guarantees on individual lines (more on this below). A crash mid-write at worst loses the partial line, never corrupts a previous one. Queries are `jq` and friends. The format is plaintext UTF-8, the same format humans have been reading on terminals for fifty years.

Compare to the alternatives I tried before settling here:

- A Postgres table: requires a daemon, schema migrations, connection management, and you still need to dump it to a file when you want to grep. Ten times more moving parts for the same query.
- A time-series database (InfluxDB, Prometheus): great for numeric metrics, terrible for storing arbitrary structured events. The agent's tape (request/response pairs, tool calls with mixed-type arguments) does not fit cleanly.
- A logging-as-a-service vendor: vendor lock-in, monthly bill, and querying anything older than 7 days requires the paid tier. Plus, on a personal workload, the latency to run a query through the vendor's UI is higher than my entire `jq` pipeline.
- Per-event files (one file per event, named by ULID): scales beautifully on object storage, but on a local filesystem creates millions of small files that destroy `find` performance and exhaust inodes.

JSONL beat all of these for my workload. Not because it is sophisticated, but because it requires no decisions.

## Section 2: the atomic-write guarantee that makes this work

The non-obvious property of append-mode file writes on POSIX systems: a write of less than `PIPE_BUF` bytes (typically 4096 on Linux, 512 on macOS) to a file opened with `O_APPEND` is atomic with respect to other appenders. Concurrent writers will not interleave bytes within a single line, as long as each line fits in the atomic-write window.

This is documented in `man 2 write`:

> If the O_APPEND file status flag is set on the open file description, then a write(2) shall, prior to writing the data, set the file offset to the end of the file as an atomic step.

In practice, this means: if every JSONL line is under 512 bytes (macOS) or 4096 bytes (Linux), you can have N processes appending to the same file with no locks, no coordinators, no merge step, and the result is a valid JSONL file with all lines intact and in some order.

For lines longer than `PIPE_BUF`, atomicity is no longer guaranteed and you can get torn writes. Two practical defenses:

```python
import json, os

def safe_append(path: str, record: dict, max_bytes: int = 512) -> None:
    line = json.dumps(record, separators=(",", ":"), default=str) + "\n"
    if len(line) > max_bytes:
        # truncate string fields, mark truncated
        record = _shrink(record, max_bytes - 64)
        record["_truncated"] = True
        line = json.dumps(record, separators=(",", ":"), default=str) + "\n"
    with open(path, "a") as f:
        f.write(line)

def _shrink(rec: dict, budget: int) -> dict:
    out = {}
    for k, v in rec.items():
        if isinstance(v, str) and len(v) > 200:
            out[k] = v[:200] + "..."
        else:
            out[k] = v
    return out
```

The first defense: keep lines small by truncating string fields with a sentinel. The second defense, used only when you need to write large records: take a file lock for the duration of the write.

```python
import fcntl

def locked_append(path: str, record: dict) -> None:
    line = json.dumps(record, separators=(",", ":"), default=str) + "\n"
    with open(path, "a") as f:
        fcntl.flock(f.fileno(), fcntl.LOCK_EX)
        try:
            f.write(line)
        finally:
            fcntl.flock(f.fileno(), fcntl.LOCK_UN)
```

I default to `safe_append`. I use `locked_append` only for the small number of streams where individual records are intentionally large (transcript dumps, verbose model traces).

## Section 3: the writer side, in production

The full writer pattern I use for any new event stream:

```python
import json, os, time, threading
from pathlib import Path

class JsonlWriter:
    def __init__(self, path: str | Path, max_line_bytes: int = 4096):
        self.path = Path(path).expanduser()
        self.path.parent.mkdir(parents=True, exist_ok=True)
        self.max_line_bytes = max_line_bytes
        self._lock = threading.Lock()  # in-process serialization

    def write(self, kind: str, **fields) -> None:
        rec = {"ts": time.time(), "kind": kind, "pid": os.getpid(), **fields}
        line = json.dumps(rec, separators=(",", ":"), default=str) + "\n"
        if len(line) > self.max_line_bytes:
            rec = self._truncate(rec)
            line = json.dumps(rec, separators=(",", ":"), default=str) + "\n"
        with self._lock:
            with self.path.open("a", buffering=1) as f:
                f.write(line)

    def _truncate(self, rec: dict) -> dict:
        out = {"_truncated": True}
        for k, v in rec.items():
            if isinstance(v, str):
                out[k] = v[:300] + ("..." if len(v) > 300 else "")
            elif isinstance(v, (dict, list)):
                out[k] = json.dumps(v)[:300] + "..."
            else:
                out[k] = v
        return out

# usage
events = JsonlWriter("~/.cache/agent/events.jsonl")
events.write("model_call", model="haiku", in_tok=1200, out_tok=80)
```

The `buffering=1` is line-buffering, which flushes on every newline. This is the difference between "the dashboard updates within a second of the event" and "the dashboard updates when the kernel decides to flush, which could be never if the process exits abnormally." Line-buffering costs about 5 percent throughput vs default buffering, in exchange for crash safety. Worth it for telemetry.

## Section 4: the reader side, including `jq` cookbook

Most of my queries are one-liners. A non-comprehensive cookbook of patterns that come up weekly:

Count events by kind in the last hour:

```bash
jq -r --arg cutoff "$(date -v-1H +%s)" 'select(.ts > ($cutoff | tonumber)) | .kind' \
  ~/.cache/agent/events.jsonl | sort | uniq -c | sort -rn
```

Sum input tokens by model over the last 1000 events:

```bash
tail -1000 ~/.cache/agent/events.jsonl | \
  jq -r 'select(.kind == "model_call") | "\(.model)\t\(.in_tok)"' | \
  awk -F'\t' '{m[$1] += $2} END {for (k in m) printf "%s\t%d\n", k, m[k]}'
```

Find every event for a specific run id:

```bash
jq -c 'select(.run_id == "a1b2c3d4")' ~/.cache/agent/events.jsonl
```

Histogram of latency, ten buckets, last day:

```bash
jq -r --arg cutoff "$(date -v-1d +%s)" \
  'select(.ts > ($cutoff | tonumber) and .dur_s) | .dur_s' \
  ~/.cache/agent/events.jsonl | \
  python3 -c "
import sys
vals = sorted(float(l) for l in sys.stdin if l.strip())
if not vals: exit()
buckets = 10
lo, hi = vals[0], vals[-1]
step = (hi - lo) / buckets or 1
counts = [0] * buckets
for v in vals:
    b = min(buckets - 1, int((v - lo) / step))
    counts[b] += 1
for i, c in enumerate(counts):
    print(f'{lo + i*step:6.2f}-{lo + (i+1)*step:6.2f} {\"#\" * c}')
"
```

The pattern is: `tail` or `jq` to filter, `awk` or Python to aggregate. Two-stage pipelines. No SQL needed.

## Section 5: rotation without losing writers

A JSONL file grows forever if you let it. At my current event rate, the agent events file gains about 80 MB per day. After a month it is over 2 GB and `jq` queries take seconds rather than tens of milliseconds. You need rotation.

The naive approach (rename the file, start a new one) breaks active writers, who hold a file descriptor to the now-renamed file and continue writing to the renamed inode rather than the new file. The right approach is to either re-open the file on every write (expensive but trivial), or to use a sentinel signal that tells writers to re-open.

I use the first approach for low-rate streams and the second for high-rate ones. For low-rate, the writer above already does this implicitly because it uses a `with open(...)` per write. For high-rate, a long-lived file handle plus a SIGHUP handler:

```python
import signal

class RotatingWriter:
    def __init__(self, path):
        self.path = Path(path).expanduser()
        self.path.parent.mkdir(parents=True, exist_ok=True)
        self._open()
        signal.signal(signal.SIGHUP, lambda *a: self._reopen())

    def _open(self):
        self._f = self.path.open("a", buffering=1)

    def _reopen(self):
        try: self._f.close()
        except: pass
        self._open()

    def write(self, line: str):
        self._f.write(line + ("\n" if not line.endswith("\n") else ""))
```

The rotation script then becomes:

```bash
#!/bin/sh
# rotate.sh, run daily from launchd or cron
LOG=~/.cache/agent/events.jsonl
STAMP=$(date +%Y%m%d)
mv "$LOG" "$LOG.$STAMP"
gzip "$LOG.$STAMP" &
# signal long-running writers to reopen
pkill -HUP -f "agent-loop"
```

Aged-out compressed files are still queryable via `zcat | jq`. After 90 days, I delete them. The dashboard cares about recent data; long-term aggregates live in a separate, much smaller summary file written by a daily cron.

## Section 6: the daily summary roll-up

Querying a 30-day window of raw JSONL is slow. Querying 30 daily summary files is fast. The roll-up is one cron job:

```python
#!/usr/bin/env python3
import json, sys
from collections import defaultdict
from datetime import datetime, timezone
from pathlib import Path

LOG = Path.home() / ".cache" / "agent" / "events.jsonl"
SUMMARY = Path.home() / ".cache" / "agent" / "daily.jsonl"

def main():
    by_day = defaultdict(lambda: defaultdict(float))
    for line in LOG.read_text().splitlines():
        try: rec = json.loads(line)
        except: continue
        day = datetime.fromtimestamp(rec.get("ts", 0), tz=timezone.utc).strftime("%Y-%m-%d")
        if rec.get("kind") == "model_call":
            by_day[day]["calls"] += 1
            by_day[day]["in_tok"] += rec.get("in_tok", 0)
            by_day[day]["out_tok"] += rec.get("out_tok", 0)
    with SUMMARY.open("a") as f:
        for day, agg in sorted(by_day.items()):
            f.write(json.dumps({"day": day, **agg}) + "\n")

if __name__ == "__main__":
    main()
```

Note this is non-idempotent on its own (re-running adds duplicate rows). I keep idempotence by tracking the last-summarized timestamp in a sidecar file and only summarizing newer events. For the post I left that out to keep the example readable.

## Section 7: schema evolution without migrations

The risk of plaintext JSONL: there is no schema enforcement. A typo in a field name on day 60 silently produces a stream of events that no query knows about. Two practices have been enough to catch this:

The first practice: every writer goes through a single helper module that knows the field set for each event kind. Free-form `**kwargs` tempts mis-spelling; named writer functions per kind do not.

```python
def write_model_call(model, in_tok, out_tok, dur_s, run_id):
    events.write("model_call", model=model, in_tok=in_tok, out_tok=out_tok,
                 dur_s=dur_s, run_id=run_id)
```

The second practice: a daily check that every event's schema matches an expected shape, with deviations logged.

```python
EXPECTED = {
    "model_call": {"model", "in_tok", "out_tok", "dur_s", "run_id"},
}

def audit(path):
    issues = 0
    for line in Path(path).read_text().splitlines():
        rec = json.loads(line)
        kind = rec.get("kind")
        if kind in EXPECTED:
            missing = EXPECTED[kind] - rec.keys()
            extra = (rec.keys() - EXPECTED[kind]) - {"ts", "kind", "pid"}
            if missing or extra:
                print(f"{kind}: missing={missing} extra={extra}")
                issues += 1
    return issues
```

When the audit flags a new field, you have a decision to make: add it to the expected set (the new field is canonical) or remove it from the writer (it was a typo). Either way, the audit forces the question.

## Section 8: scale ceilings, and what to do when you hit them

JSONL on a local filesystem is fast until it isn't. The two ceilings I have hit:

The first: file size around 5 GB. `jq` slows from milliseconds to seconds on whole-file scans. The fix is rotation plus the daily summary, covered above. After rotation, no individual file exceeds 100 MB on my workload and `jq` stays in millisecond territory.

The second: write rate around 1000 events per second per file. At higher rates, even line-buffered append starts to bottleneck on the lock or on disk sync. The fix is to shard: write to one file per (process, day) and merge at read time. The reader then becomes `jq -s 'add'` over `cat shard1.jsonl shard2.jsonl ...`, which is still fast on a few hundred MB.

I have not personally hit the 10000 events/sec range. At that scale, you probably want a real streaming system (Kafka, NATS Jetstream, or a single-binary log shipper writing to S3). The transition cost is real but not enormous: the consumer side stays as `jq`-style queries on the materialized files; only the producer side moves.

## Section 9: the failure modes I have lived with

Three real incidents, and what they taught.

The first: a writer process running with a stale file handle after I manually `mv`'d the log file (forgetting that I had a long-running writer). The writer happily kept appending to the inode that was now `events.jsonl.20260315`, while my dashboard read from the new empty `events.jsonl`. Took an hour to notice. Fix: never `mv` log files manually; always go through the rotation script that signals SIGHUP.

The second: a writer crashed mid-line, leaving a partial line at the tail of the file. The next time `jq` ran, it failed parsing the entire file because of the one malformed line. Fix: read with `jq -c .` and a try/except, or pre-filter with `awk 'NF > 0 && /^\{.*\}$/'` to drop malformed lines.

The third: clock skew between two writer hosts produced events with `ts` going backwards on some queries. Most queries did not care, but the daily summary did. Fix: write `ts` as a monotonic-on-this-host value and add a separate `host` field, then dedup on `(ts, host, pid, kind)` in the rare cases where order matters.

None of these were hard to fix once diagnosed. All of them were diagnosed by looking at the file with `tail`, `head`, and `jq`. The file is the debugger.

## Section 10: why I have not migrated to anything fancier

A common piece of advice when you reach a few hundred MB of telemetry per day is to migrate to a "real" system. I have considered ClickHouse, DuckDB, Parquet on S3, and a few logging vendors. None of them have made a strong enough case to justify the migration cost on my workload, which sits at about 200 MB per day and 50 to 200 events per second peak.

What I would actually want from a fancier system is lower-latency complex queries and better long-term retention. JSONL plus daily summaries already handles the second; for the first, DuckDB over the rotated files would be the logical next step. DuckDB reads JSONL and Parquet natively, runs SQL over them in-process, and requires no daemon. When my queries get hairier than `jq` can handle, that is where I will go. For now, `jq | awk` continues to win.

The boring shape works for surprisingly long. The discipline is to actually wait until it stops working, rather than to switch off it because something newer exists.

## References

- POSIX `write(2)` atomicity: https://pubs.opengroup.org/onlinepubs/9699919799/functions/write.html
- JSONL specification: https://jsonlines.org/
- `jq` manual: https://jqlang.github.io/jq/manual/
- DuckDB JSON reader: https://duckdb.org/docs/data/json/overview
- Linux `PIPE_BUF` constant: https://man7.org/linux/man-pages/man7/pipe.7.html
