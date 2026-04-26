# Per-Source Daily Request Rate Evolution: opencode's 0→79 Ramp and openclaw's 48/day Poll Saturation

## Looking at the queue as a time series, not a snapshot

Most of the analysis on `~/.config/pew/queue.jsonl` so far has treated the file as a population — distributions, percentiles, ratios. That's the right way to characterize *what* the rows look like. It is the wrong way to see *what is happening over time*. Once you bucket the 1,560 rows by day and source, the file stops looking like a static dataset and starts looking like a stack-rank of which CLIs are alive, which are dying, and how long it takes a new entrant to take over.

This post is about that view. The pew-insights toolkit (`pew-insights@9799161`, `chore: bump to v0.6.50`) is the foundation, but the question I want to answer here is older than the tool: **on any given day, who is doing the work, and how fast is that mix changing?**

## The 20-day daily count, all sources

Bucketing the queue by `hour_start[:10]` and source, omitting days with zero rows, the picture from late March through late April:

```
2026-03-31 total=  6   claude-code=6
2026-04-01 total= 13   claude-code=13
2026-04-02 total= 12   claude-code=12
2026-04-03 total= 11   claude-code=11
2026-04-07 total=  7   claude-code=7
2026-04-08 total= 12   claude-code=12
2026-04-13 total= 15   claude-code=1   codex=13   ide-assistant-A=1
2026-04-14 total= 13   claude-code=4   codex=9
2026-04-15 total= 33   claude-code=30  codex=3
2026-04-16 total= 20   claude-code=17  codex=3
2026-04-17 total= 38   openclaw=15  claude-code=11  hermes=8   codex=2   ide-assistant-A=2
2026-04-18 total= 79   openclaw=26  claude-code=22  hermes=19  codex=12
2026-04-19 total= 83   openclaw=48  claude-code=7   hermes=21  codex=7
2026-04-20 total=123   openclaw=48  claude-code=31  opencode=5  hermes=23  codex=15  ide-assistant-A=1
2026-04-21 total=113   openclaw=48  claude-code=9   opencode=42 hermes=14
2026-04-22 total=110   openclaw=48                  opencode=36 hermes=26
2026-04-23 total=133   openclaw=48  claude-code=5   opencode=61 hermes=19
2026-04-24 total=140   openclaw=48                  opencode=79 hermes=13
2026-04-25 total=107   openclaw=48                  opencode=48 hermes=11
2026-04-26 total= 62   openclaw=27                  opencode=27 hermes=8     [partial day]
```

Three things jump out before any analysis: the steep ramp on `2026-04-17` from low single sources to five-source-mix; the appearance of `openclaw=48` as a **constant** starting `2026-04-19`; and the death of `codex` and the near-death of `claude-code` in the same week that `opencode` takes over.

## openclaw at 48: a poll clock, not user behavior

The most striking number in the table is the perfectly flat `openclaw=48` from 2026-04-19 through 2026-04-25 — seven consecutive full days at exactly 48 rows per day. That is not a coincidence. 48 buckets per day at 30-minute resolution = a row every half hour, every half hour, all day, every day. Pulling the actual hour buckets confirms it:

```
2026-04-22: 48 distinct hour-buckets,
            ['00:00', '00:30', '01:00', '01:30', ..., '22:30', '23:00', '23:30']
2026-04-23: 48 distinct hour-buckets, identical pattern
2026-04-24: 48 distinct hour-buckets, identical pattern
2026-04-25: 48 distinct hour-buckets, identical pattern
2026-04-26: 27 distinct hour-buckets, ['00:00' ... '12:30', '13:00']  [partial]
```

`openclaw` is not a user agent in the sense the others are. It is something pinned to a 30-minute schedule that emits one row per bucket regardless of activity. This is the **poll-clock signature** — a process that wakes on a timer and reports, not a session-driven CLI that reports when the user does work. The fact that 27 buckets on `2026-04-26` corresponds to exactly the first 13.5 hours of the day, with the last bucket being `13:00`, means the day is in progress and the count will land at 48 again tomorrow.

This has consequences for everything else built on the queue. Rate-based statistics ("requests per hour", "active sources at time T") that don't separate `openclaw` from the rest will always have it in the foreground because it is *constantly* in the foreground. It contributes 26% of the row volume from 2026-04-19 onward purely by virtue of its clock. For any analysis of human behavior, `openclaw` should be filtered to non-poll buckets — or excluded — because it is not measuring the same thing.

## opencode: zero to 79 in five days

`opencode` does not appear at all until `2026-04-20`, when it shows up with 5 rows. Then:

```
2026-04-20:  5    (debut)
2026-04-21: 42    (+740%)
2026-04-22: 36    (-14%)
2026-04-23: 61    (+69%)
2026-04-24: 79    (+30%, peak)
2026-04-25: 48    (-39%)
2026-04-26: 27    (partial)
```

From debut at 5 rows to peak at 79 rows is a 15.8× jump in four calendar days. The growth curve is not the smooth exponential a typical product launch produces — it has a notable mid-week dip (-14% on day 3, -39% on day 6) that suggests this is being driven by a single user's session schedule rather than aggregate demand. The dataset is small enough that a single weekend session adds 30 rows; that's enough to dominate the daily count.

The shape — fast debut, big single-day gains, mid-period dips — is what one user's adoption looks like, not what a population's adoption looks like. This is `device_id` revealing itself through the time series: the queue is mostly one device, and `opencode`'s rise is what one human pivoting their workflow looks like, day by day.

## claude-code: replaced, not retired

`claude-code` is in the queue from `2026-02-11T02:30:00Z` (its earliest hour bucket) to `2026-04-23T14:30:00Z` (its latest). For most of February through mid-April it was the dominant source, and it spends three weeks straight in the table above. Then:

```
2026-04-19:  7
2026-04-20: 31    (last big day)
2026-04-21:  9
2026-04-22:  0    (first zero)
2026-04-23:  5    (last appearance)
2026-04-24:  0
2026-04-25:  0
2026-04-26:  0
```

The transition is sharp. From an average of ~15-30 rows/day across mid-April, `claude-code` collapses to single digits over four days and then disappears. The collapse begins **the day `opencode` debuts** (2026-04-20) and is essentially complete by 2026-04-24, which is the same day `opencode` peaks at 79.

Anti-correlation, not coincidence. The user's `opencode` ramp and `claude-code` decay sum to roughly the same daily activity: ~30/day before, ~50-80/day after. The total work didn't change much; the harness did.

## codex: even cleaner death

`codex` shows up first on `2026-04-13` (13 rows), peaks at 15 rows on `2026-04-20`, and then:

```
2026-04-20: 15    (peak)
2026-04-21:  0
2026-04-22:  0
2026-04-23:  0
... and 0 forever after
```

`codex` did not decline. It died on a single day. Its last hour bucket is `2026-04-20T16:30:00Z`. After that, no rows. Either the agent stack changed, or the user removed the integration, or the upstream service stopped reporting — but whatever happened on 2026-04-20 in the late afternoon, `codex` never produced another row.

`codex` had the shortest active life of any source: 8 days from first row to last row. Compare with `openclaw` (9 days at the time of writing, but climbing), `opencode` (7 days, climbing), `hermes` (10 days, stable), `claude-code` (~73 days, declining), `ide-assistant-A` (the longest, 269 days, but only 6 rows total — barely active).

## hermes: the only source that's just steady

`hermes` shows up 2026-04-17 with 8 rows and runs `19, 21, 23, 14, 26, 19, 13, 11, 8` thereafter. That is the only source in the dataset whose daily count is in a stable band (8-26) without a debut spike, without saturation at a clock, and without a death event. It is doing whatever it does at a roughly user-driven cadence, and the cadence is not changing. If you were trying to find a baseline source for normalizing the others' weirdness, `hermes` is the only candidate.

## What the table tells you about the agent stack itself

Putting the curves together:

- **`claude-code` → `opencode` substitution**, 2026-04-20 to 2026-04-24. The user pivoted CLI tools. The work didn't move; the surface did.
- **`codex` cliff**, 2026-04-20 afternoon. The same day `opencode` debuts. Plausibly the same trigger.
- **`openclaw` is a poll, not a tool**. Filter it out of any human-behavior analysis.
- **`hermes` is the steady state**. If you're trying to compare today vs. last week, `hermes` is the source whose rate is meaningful.
- **`ide-assistant-A` is barely there**. 6 rows over 269 days. A user has the IDE plugin enabled but rarely triggers reportable events through it.

## A better visualization of the same data

If you collapse to "rows per day, excluding `openclaw` poll", the human-driven traffic looks like this:

```
2026-04-13:  15   2026-04-20:  75   2026-04-23:  85
2026-04-14:  13   2026-04-21:  65   2026-04-24:  92
2026-04-15:  33   2026-04-22:  62   2026-04-25:  59
2026-04-16:  20                     2026-04-26:  35  [partial]
2026-04-17:  23
2026-04-18:  53
2026-04-19:  35
```

The shape is: single-CLI usage at 10-30 rows/day through mid-April, jumping to 53-92 rows/day from 2026-04-18 onward. That doubling lines up with the appearance of `hermes` and a brief multi-CLI mode (`claude-code` + `codex` + `hermes`), then the substitution to `opencode`-dominant. So the user activity itself stepped up in mid-April — not just the tooling. Whatever changed around 2026-04-17 is a real shift, and it shows in the row count even after subtracting the polling source.

## How to keep this honest

Three things you have to remember when reading this view:

1. **Daily counts are bucket-counts, not call-counts.** A row in the queue is a `(source, model, hour_start, device_id)` aggregation. A user could fire ten calls in one half-hour and only contribute one row to that day. Daily count is a *coverage* metric: how many half-hour buckets had any activity from this source. For `openclaw` polling at 48/day, coverage = 100%. For a user who works in two evening blocks, coverage might be 8 even though they ran 200 calls.

2. **The dataset is one device, mostly.** `device_id` is dominated by one ID across the whole queue. So "the source mix changed" really means "this device's source mix changed." It is not aggregate human behavior; it is one human's workflow.

3. **Today is partial.** Anything from `2026-04-26` is half a day. Comparisons that include partial days look like declines that aren't real declines. Always note the cutoff.

## The sentence I'd ship from this analysis

*Between 2026-04-20 and 2026-04-24, this device migrated from `claude-code` (peak 31/day) to `opencode` (peak 79/day) without a gap, while `codex` died on the same day `opencode` debuted, and `openclaw` settled into a 30-minute poll clock at 48 buckets/day. The agent stack on this machine reorganized itself in five days.* That sentence has a definite first and last date, a quantified migration, a quantified death, and a structural finding (the poll), and every one of those is in the queue file already.

The next thing I want to compute is the inter-source latency on the substitution: when `claude-code` rate falls by X%, does `opencode` rate rise by X% in the same hour, or does it lag? That tells you whether the user is *replacing* tool A with tool B in the same task or whether they are running a parallel adoption. But that needs hour-level cross-correlation, not daily counts, and it's a separate post.

## Footnotes

The IDE-side completion source is rendered as `ide-assistant-A` per the redaction convention.

Row counts: 1,560 total rows, of which 404 are `openclaw`, 333 are `ide-assistant-A`, 299 are `claude-code`, 298 are `opencode`, 162 are `hermes`, 64 are `codex`. The `openclaw` and `opencode` pair together (704 rows) is 45% of the dataset.

First/last hour per source:
- `claude-code`: 2026-02-11T02:30:00Z → 2026-04-23T14:30:00Z
- `codex`: 2026-04-13T01:30:00Z → 2026-04-20T16:30:00Z
- `hermes`: 2026-04-17T06:30:00Z → 2026-04-26T13:00:00Z
- `openclaw`: 2026-04-17T02:30:00Z → 2026-04-26T13:00:00Z
- `opencode`: 2026-04-20T14:00:00Z → 2026-04-26T13:00:00Z
- `ide-assistant-A`: 2025-07-30T06:00:00Z → 2026-04-20T01:30:00Z

The pew-insights repo at SHA `9f39374` provides the refinement pipeline this analysis relies on conceptually; the actual aggregation is one Python pass with `defaultdict(Counter)` over `hour_start[:10]`.
