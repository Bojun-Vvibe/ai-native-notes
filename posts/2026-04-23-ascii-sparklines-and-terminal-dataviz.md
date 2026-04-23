---
title: "ASCII sparklines and terminal dataviz for agent telemetry"
date: 2026-04-23
tags: [terminal, dataviz, observability, ascii, sparklines]
est_reading_time: 13 min
---

## TL;DR

A 40-character ASCII sparkline rendered in your terminal answers most of the dashboard questions you have about an agent fleet without the cost or latency of a real dashboard. You can build one in 12 lines of Python using Unicode block characters. Combined with a few other primitives (horizon charts, bar charts, sparkbars, ANSI color), you get a debugging surface that lives next to your shell prompt, updates in real time over `tail -f`, and ships in any environment that has a terminal. This post walks through the seven patterns I use daily for visualizing token usage, latency distributions, error rates, and queue depths, all from JSONL log files and all rendered in pure ASCII.

The bigger point: dashboards are good, but the friction of opening a browser tab is enough to make you not look. A `watch -n 1` over a script that prints sparklines is a dashboard you actually look at.

## Why this matters

For the first six months of running my agent stack, I had a real Grafana setup. Beautiful charts, alerting, the works. I looked at it maybe twice a week, when something was already on fire. The reason was friction: open browser, find tab, wait for queries, scroll. By the time I had a chart in front of me, I had usually already SSH'd into a box to grep logs.

The change that actually got me looking at metrics was when I added `pew status` (a custom shell function) that prints six sparklines in one terminal screen, sourced from the JSONL logs my agents write. I now look at it ten times a day. The Grafana setup still exists, for alerting and historical queries, but for "what is happening right now," the terminal won.

This post is the construction manual for that surface.

## Section 1: the basic sparkline

The Unicode block range `U+2581` to `U+2588` is exactly what we need: eight characters of increasing height (`▁▂▃▄▅▆▇█`). A sparkline is a string of these characters, one per data point, scaled to the data's range.

```python
BLOCKS = "▁▂▃▄▅▆▇█"

def sparkline(values, width=40):
    if not values:
        return " " * width
    # downsample if needed by averaging buckets
    if len(values) > width:
        bucket = len(values) / width
        sampled = []
        for i in range(width):
            lo, hi = int(i * bucket), int((i + 1) * bucket)
            chunk = values[lo:hi] or [values[lo] if lo < len(values) else 0]
            sampled.append(sum(chunk) / len(chunk))
        values = sampled
    lo, hi = min(values), max(values)
    if hi == lo:
        return BLOCKS[0] * len(values)
    span = hi - lo
    return "".join(BLOCKS[min(7, int((v - lo) / span * 8))] for v in values)

# example
import math
print(sparkline([math.sin(i / 5) for i in range(60)]))
```

That is the entire kernel. Everything in this post builds on it.

## Section 2: spark from JSONL with a single one-liner

The shape I use most: read the last N lines of a JSONL log, project a numeric field, render a sparkline. As a function:

```python
import json
from pathlib import Path

def spark_field(path, field, n=120, width=40):
    lines = Path(path).read_text().splitlines()[-n:]
    vals = []
    for l in lines:
        try:
            d = json.loads(l)
            vals.append(float(d.get(field, 0)))
        except (json.JSONDecodeError, ValueError, TypeError):
            continue
    return sparkline(vals, width=width)

print(spark_field("~/.cache/router.jsonl", "in_tok"))
```

For the router log from yesterday's post, this prints something like:

```
▁▁▂▂▃▃▃▄▄▄▅▅▆▆▇▇▇▆▅▄▃▂▂▁▁▂▃▄▅▆▇█▇▆▅▄▃▂▁
```

You can see traffic ramp, peak, decline, and ramp again in 40 characters. No axis labels, no legend, but enough to know "is this normal."

## Section 3: the labeled sparkline

A bare sparkline answers "shape," not "magnitude." Add the min, max, and last value as small labels and you have something useful in isolation:

```python
def labeled_spark(values, width=40, fmt="{:>5.0f}"):
    if not values:
        return f"{fmt.format(0)} [{' '*width}] {fmt.format(0)} (last: {fmt.format(0)})"
    s = sparkline(values, width)
    return f"{fmt.format(min(values))} [{s}] {fmt.format(max(values))} (last: {fmt.format(values[-1])})"

# example
print(labeled_spark([10, 12, 18, 30, 45, 50, 48, 38, 22, 14]))
```

Output:

```
   10 [▁▁▂▄▆▇▇▅▃▁] 50 (last: 14)
```

This is the form I default to in dashboards. Min, max, and last give you the three numbers you need; the bracket gives you the trend.

## Section 4: ANSI color for thresholds

When you have a target (latency under 500ms, fail rate under 2 percent), color the last value red or green based on whether it's in spec. Most terminals support ANSI 256-color since 2010.

```python
RED = "\033[31m"; GRN = "\033[32m"; YEL = "\033[33m"; RST = "\033[0m"

def color_for(v, good_max, warn_max):
    if v <= good_max: return GRN
    if v <= warn_max: return YEL
    return RED

def threshold_spark(values, good_max, warn_max, width=40):
    s = sparkline(values, width)
    last = values[-1] if values else 0
    c = color_for(last, good_max, warn_max)
    return f"[{s}] {c}{last:.0f}{RST}"

print(threshold_spark([120, 130, 145, 200, 380, 510, 620, 480, 410, 350], 400, 600))
```

The last value will be yellow because 350 is under the good threshold of 400; flip the inputs to see red. Three colors is plenty; more is noise.

## Section 5: the multi-line dashboard

A single sparkline is one metric. Six sparklines stacked is a dashboard. The trick is to make them line up: same width, same label column, same numeric column.

```python
def dashboard(rows):
    # rows: list of (label, values, fmt, good_max, warn_max)
    label_w = max(len(r[0]) for r in rows)
    out = []
    for label, values, fmt, good_max, warn_max in rows:
        s = sparkline(values, 40)
        last = values[-1] if values else 0
        c = color_for(last, good_max, warn_max) if good_max else ""
        rst = RST if c else ""
        out.append(f"{label:<{label_w}} [{s}] {c}{fmt.format(last)}{rst}")
    return "\n".join(out)

print(dashboard([
    ("input tok/min ", [100,120,140,160,200,180,160,140,120,100], "{:>6.0f}", 250, 500),
    ("output tok/min", [40,45,50,55,60,55,50,45,40,35],          "{:>6.0f}", 100, 200),
    ("p95 latency ms", [380,400,420,450,500,480,460,440,420,400], "{:>6.0f}", 500, 800),
    ("error rate %",   [0.5,0.6,0.8,1.2,1.5,1.0,0.8,0.5,0.4,0.3], "{:>6.2f}", 2.0, 5.0),
]))
```

Aligned output with colored last-value column, in 25 lines of total source. Wrap this in `watch -n 2` and you have a live dashboard:

```
watch -n 2 -c 'python3 -c "..."'
```

## Section 6: horizon charts for long time series

Sparklines lose detail when the data range is wide. A horizon chart slices the data into bands of equal value-width and stacks them in different colors, fitting more dynamic range into the same vertical space. The trick: use only one row of vertical space, render each band as its own sparkline-like glyph, overprint with color.

```python
HORIZON_BLOCKS = "▁▂▃▄▅▆▇█"
HORIZON_COLORS = ["\033[34m", "\033[36m", "\033[32m", "\033[33m", "\033[35m", "\033[31m"]

def horizon(values, bands=4, width=40):
    if not values: return ""
    if len(values) > width:
        bucket = len(values) / width
        values = [sum(values[int(i*bucket):int((i+1)*bucket)]) / max(1, int((i+1)*bucket)-int(i*bucket)) for i in range(width)]
    hi = max(values) or 1
    band_h = hi / bands
    out = []
    for v in values:
        band = min(bands - 1, int(v / band_h))
        within = (v - band * band_h) / band_h
        glyph = HORIZON_BLOCKS[min(7, int(within * 8))]
        color = HORIZON_COLORS[band % len(HORIZON_COLORS)]
        out.append(f"{color}{glyph}{RST}")
    return "".join(out)

print(horizon([i % 100 for i in range(200)], bands=4))
```

A horizon chart packs roughly 4x the dynamic range of a sparkline into the same row. I use it for token-spend rates that span four orders of magnitude (1 token to 10k tokens per call).

## Section 7: bar charts and sparkbars

For comparing categories rather than time series, the same block characters render horizontal bars:

```python
def hbar(values, labels, width=40, fmt="{:>6.0f}"):
    hi = max(values) or 1
    label_w = max(len(l) for l in labels)
    out = []
    for label, v in zip(labels, values):
        bar = "█" * int(v / hi * width)
        out.append(f"{label:<{label_w}} {bar} {fmt.format(v)}")
    return "\n".join(out)

print(hbar([12, 34, 56, 78, 23, 45], ["haiku","sonnet","opus","gpt-4o","mini","local"]))
```

For percentage breakdowns (token mix by model), a stacked sparkbar:

```python
def sparkbar(parts, labels, width=40):
    total = sum(parts) or 1
    out = ""
    for p, l in zip(parts, labels):
        n = int(p / total * width)
        out += l[0].upper() * n
    return out

print(sparkbar([30, 50, 20], ["haiku", "sonnet", "opus"]))
# HHHHHHHHHHHHSSSSSSSSSSSSSSSSSSSSOOOOOOOO
```

Crude, readable, fits in a single line.

## Section 8: the streaming dashboard, no `watch` needed

`watch` re-runs the script every interval. Sometimes you want a true live view that updates only when new data arrives. ANSI cursor controls let you redraw in place.

```python
import sys, time, json
from pathlib import Path

CLEAR = "\033[2J\033[H"

def stream(path, fields):
    path = Path(path)
    last_size = 0
    while True:
        sz = path.stat().st_size
        if sz != last_size:
            lines = path.read_text().splitlines()[-200:]
            sys.stdout.write(CLEAR)
            sys.stdout.write(f"streaming {path.name} ({len(lines)} recent rows)\n\n")
            for f in fields:
                vals = []
                for l in lines:
                    try: vals.append(float(json.loads(l).get(f, 0)))
                    except: pass
                sys.stdout.write(f"{f:<16} {labeled_spark(vals)}\n")
            sys.stdout.flush()
            last_size = sz
        time.sleep(1)

# stream("~/.cache/router.jsonl", ["in_tok", "out_tok", "dur_s"])
```

A `tail -f` for charts. The clear-and-redraw is jittery if updates are too frequent, so debouncing on file size change keeps it pleasant.

## Section 9: the patterns I have stopped using

A short list of dataviz primitives I tried and abandoned, with reasons.

- Box plots in ASCII: theoretically nice, but the glyphs do not align well in proportional fonts and the box-and-whisker shape is too big for a one-line metric. If you want distribution shape in one line, use a horizon chart instead.
- Heatmaps using background colors: visually busy, hard to interpret without a legend, and break in some terminals (especially over SSH from a different OS). Avoid.
- Animated spinners next to metrics: cute, distracting, and they hide the real change in the numbers. The data is the spinner.
- Unicode braille for high-density sparklines: looks great, but the braille glyphs are not consistently rendered across fonts. The block characters are universal.

## Section 10: the actual `pew status` script

For completeness, here is the trimmed version of the script I run as `pew status`:

```python
#!/usr/bin/env python3
import json
from pathlib import Path

LOG = Path.home() / ".cache" / "router.jsonl"
USAGE = Path.home() / ".cache" / "llm-usage.jsonl"

def spark(values, width=40):
    if not values: return " " * width
    BLOCKS = "▁▂▃▄▅▆▇█"
    lo, hi = min(values), max(values)
    if hi == lo: return BLOCKS[0] * len(values)
    return "".join(BLOCKS[min(7, int((v - lo) / (hi - lo) * 8))] for v in values[-width:])

def field(path, key, n=120):
    if not path.exists(): return []
    out = []
    for l in path.read_text().splitlines()[-n:]:
        try: out.append(float(json.loads(l).get(key, 0)))
        except: pass
    return out

def fmt(label, vals, unit=""):
    if not vals: return f"{label:<14} (no data)"
    return f"{label:<14} [{spark(vals)}] last={vals[-1]:.1f}{unit} max={max(vals):.1f}{unit}"

print(fmt("in tokens",   field(USAGE, "in"),     ""))
print(fmt("cache reads", field(USAGE, "cache_read"), ""))
print(fmt("out tokens",  field(USAGE, "out"),    ""))
print(fmt("call dur s",  field(LOG, "dur_s"),    "s"))
print(fmt("attempt",     field(LOG, "attempt"),  ""))
```

That is the dashboard. Ten lines of useful output, regenerated in under 50 ms, no browser, no network. The Grafana dashboards still exist for week-over-week trending. For the right-now question, this script is what I look at.

## Section 11: why the terminal wins for "right-now" observability

The browser-based dashboard is not bad. It is, in many ways, technically superior: it can do real interactive zoom, it can render a hundred series at once, it can drive alerting, it can be shared as a URL. None of that matters if you do not look at it. The terminal-based version wins on a different axis, which is friction, and friction is the thing that determines whether observability actually gets used.

Consider the path from "I noticed the agent felt slow" to "I have a chart in front of me." For the browser dashboard, the path is: switch context to browser, find the right tab, wait for the page to load, wait for queries to run against the time-series backend, possibly authenticate again, scroll to the relevant panel, adjust the time range. Even when each step is fast, the sum is 15 to 30 seconds, and the context switch out of the terminal is the expensive part. Every time I do this I lose my place in whatever I was doing.

The terminal version's path is: type three characters and hit return. The chart is in front of me in 50 milliseconds, in the same window I was already working in. There is no context switch. The dashboard is part of the shell, the same way `git status` is.

The cost of this is real. The terminal dashboard cannot do interactive drill-down. It cannot show a chart of arbitrary metrics from arbitrary time ranges. It cannot alert. For those use cases, the browser dashboard is the right tool. But "I want to glance at the system" is the most common observability use case by frequency, and the terminal wins it by an order of magnitude on friction. So that is where I optimized.

The pattern generalizes beyond observability. Any tool whose value depends on being looked at frequently should live where the user already is. That is usually the terminal. The browser is for tools whose value depends on shareability or interactivity, neither of which applies to a private, read-only dashboard.

## Section 12: what to log so the dashboard works

The dashboard is downstream of the log format. If the log is sloppy, the dashboard is sloppy. Three rules I now apply to every log file the agents write:

The first rule: one event per line, JSONL, no exceptions. Multi-line JSON is harder to tail, harder to grep, and harder to truncate safely. The line is the unit.

The second rule: every line has a `ts` field as a Unix timestamp in seconds, and a `kind` field naming the event type. The `ts` field lets time-based filtering be a one-liner. The `kind` field lets you split a single log into multiple sparklines without parsing the rest.

The third rule: numeric fields are flat, not nested. `{"in_tok": 1200}` not `{"usage": {"input_tokens": 1200}}`. The flat shape lets the field-extractor be a single `.get()` call instead of a path traversal. This sounds trivial; it is the difference between a 3-line dashboard script and a 30-line one.

Following these three rules, the dashboard scripts in this post extend to any new metric in two minutes: add a field at the log site, add a row to the dashboard. The lift is constant, not linear, in the number of metrics.

## References

- Unicode block elements: https://en.wikipedia.org/wiki/Block_Elements
- ANSI escape codes: https://en.wikipedia.org/wiki/ANSI_escape_code
- Tufte on sparklines: https://www.edwardtufte.com/tufte/posters
- Horizon charts (Heer et al.): https://idl.cs.washington.edu/papers/horizon
- `watch(1)` man page: https://man7.org/linux/man-pages/man1/watch.1.html
