# Device share when N equals one — the singleton report as a meta-observation

Sometimes the most informative output of an analytical tool is the
output that looks like a degenerate case. You wire up a per-device
share-of-token-mass report, you point it at a real queue, and it
returns one row with `100.00%` in the share column. Your first
instinct is to say the report is broken, or unnecessary, or that
device-share is "not yet meaningful for this fleet." Your second
instinct, if you trust your instruments, should be to look at what
the singleton row is actually telling you. There is more in it than
the share column.

This note is about that. The data comes from `pew-insights
device-share` v0.4.48, run on the local queue at
`~/.config/pew/queue.jsonl` as of `2026-04-25T03:39:52.070Z`.

## The singleton row, verbatim

```
device_id                             tokens         share    rows   active hrs  models  sources  input          cached         output      cache%  first seen         last seen
------------------------------------  -------------  -------  -----  ----------  ------  -------  -------------  -------------  ----------  ------  -----------------  -----------------
a6aa6846-9de9-444d-ba23-279d86441eee  8,325,823,145  100.00%  1,407  877         15      6        3,338,890,430  4,950,808,195  35,025,886  148.3%  2025-07-30T06:00Z  2026-04-25T03:30Z
```

One row. One device. 100% share. Total tokens 8,325,823,145.
Drops: `0 bad hour_start, 0 zero-tokens, 0 empty device_id, 0
below min-tokens, 0 below top cap`. There is nothing missing,
nothing filtered, nothing pending. The fleet is one device.

If you stop reading after the share column you have learned
nothing. If you read the rest of the row you have learned a
surprising amount about what this device has been doing for the
last 269 days.

## What the row tells you that the share doesn't

The share column is the headline only because device-share was
designed to surface *concentration* — to detect the fleet where
one machine is producing 90% of the token mass and the other
fifteen are producing 10%. When the fleet is one machine, share
collapses to 100% and the column stops carrying information.
Everything else in the row is still meaningful, and several
columns are more meaningful precisely because there is no other
device to compare against.

**`rows: 1,407` and `active hrs: 877`.** Across 269 days
(2025-07-30 to 2026-04-25, the `first seen` to `last seen`
window), this device has been alive in 877 distinct UTC hours.
That is roughly `877 / (269 × 24) ≈ 13.6%` of all wall-clock
hours in the window. Not a server. Not a CI runner. Not a
24/7 background process. The duty cycle is exactly the duty
cycle of a single human's working device — a little less than
four hours of activity per calendar day on average, with the
real distribution almost certainly clustered into work-hour
spikes and dead nights.

The `1,407 / 877 ≈ 1.6 rows per active hour` ratio is also
informative. A row in the queue corresponds roughly to a tool
call or sync event, not to a turn. 1.6 rows per active hour
implies the device is mostly batching its activity — it logs
something, then is quiet for tens of minutes, then logs again.
This is not a chatty fleet client polling every minute. This
is a coarse-grained sync from a tool that aggregates locally
and reports periodically.

**`models: 15` and `sources: 6`.** The same device has touched
15 distinct normalised models across 6 distinct producer
sources. Pair this with the cohabitation finding that the
fleet has one binary-star pair (opus-4.7 × gpt-5.4 with
`cohabIndex = 0.678`) and a long tail of 13 other models, and
the picture sharpens: this single device is doing all the
multi-model work. There is no fleet diversity story to tell
because there is no fleet. The model and source diversity all
live inside this one machine. Whatever architectural complexity
you might attribute to "multi-tenant fleets" or "routing
policies" is in fact one user's setup talking to multiple
producers and multiple model endpoints from the same box.

**`input: 3.34B`, `cached: 4.95B`, `output: 35.0M`.** The
absolute numbers matter more than the share now. Output is
0.42% of input. That is the canonical "agent reading a lot,
writing a little" ratio — the machine spends 99.6% of its
token budget on context (prompts, tool outputs, file reads,
context-injection scaffolding) and 0.4% on the model's actual
generated reply. If you went looking for inefficiency, you
would look at the input column, not the output column.

**`cache%: 148.3%`.** This is the column that should make you
stop scrolling. The cache hit ratio is 148%, which means
`cached_input` exceeds `input`. There is a separate post on
why ratios above 100% are meaningful rather than buggy, but
the short version is: cached tokens here are *prompt-cache*
hits, billed at a discount, and the count of cache-hit tokens
is being measured against the count of new (non-cache) input
tokens. When the same context is replayed many times — long
agent sessions reusing the same system prompt and tool
definitions across dozens of turns — the cumulative cache hit
mass exceeds the cumulative new input mass, and you get a
ratio above 1.0. 148% is on the high end but not pathological.
It tells you that this device is doing *long sessions* with
*high prompt reuse*, which is consistent with a developer
running an agentic CLI that holds a multi-hour conversation
open with the same system prompt and the same tool registry.

**`first seen: 2025-07-30T06:00Z` and `last seen:
2026-04-25T03:30Z`.** Roughly nine months of continuous
existence. The 30-minute granularity on `last seen` reflects
that the device was active in the half-hour bucket ending at
03:30 UTC on the day of the report, which is the bucket
immediately before the report ran. There is no staleness; the
device is still live.

## The shape of "100% share, but not trivially so"

What this row is telling you, taken together, is that
device-share is being asked the wrong question. The interesting
report on a single-device queue is not "which device dominates"
— that is trivially answered. The interesting report is
"what does this device do per active hour," and almost every
column in the device-share row carries that signal:

- 877 active hours over 269 days → 13.6% duty cycle, consistent
  with a single human's working schedule.
- 1.6 rows per active hour → batched, coarse-grained sync, not
  per-call telemetry.
- 6 sources, 15 models inside one device → the diversity story
  lives inside the machine, not across machines.
- output / input ≈ 0.4% → context-heavy, generation-light,
  agent-shaped workload.
- cache% = 148% → long sessions with high prompt reuse, again
  agent-shaped.
- 9-month continuous activity window → not a one-off
  experiment; this is the device's normal operating mode.

The `100.00%` cell in the share column is not the headline.
It is the *boundary condition* that makes every other column
trustworthy: you do not have to wonder how much of any
property is coming from a noisy minority device, because there
is no minority device. Every aggregate is a per-device
statistic by construction.

## Why the singleton output is a *feature*, not a bug

A common reaction to a single-row device-share output is to
silently fold it into a heading and refuse to print the table:
"only one device, skipping report." That is the wrong
reaction, for three reasons.

First, the singleton row is a check on data integrity. The
`drops` line tells you `0 bad hour_start, 0 zero-tokens, 0
empty device_id, 0 below min-tokens, 0 below top cap`. If a
producer ever started emitting rows with empty device IDs —
because of a config bug, a sync without a logged-in user, a
provisioning slip — those rows would land in
`droppedEmptyDeviceId` and the singleton would suddenly be
joined by a "phantom device" or by a drop counter that did
not used to be nonzero. Suppressing the table on N=1 hides
the change in N. Printing it makes the boundary visible the
moment it moves.

Second, the singleton row is a baseline for cross-fleet
comparison. If at some later date a second device appears in
this queue — a backup laptop, a cloud sandbox, a teammate's
machine onboarded into the same telemetry stream — the new
device-share output will show two rows, and you will want the
historical singleton row to compare against. "How does the
new device look against the established device?" is a question
you can only answer if the established device has a paper
trail in the report's own format. Suppressing the table for
the period when N was 1 means the new comparison has nothing
to anchor to.

Third, the singleton row exposes the long axis of the device.
The 877 active hours, 15 models, 6 sources, and 148% cache
ratio numbers are all derived statistics that the device-share
report computes anyway. They are *more* useful when the
device is the whole fleet, because they describe the entire
queue's behaviour without any per-device aggregation noise.
On a 100-device fleet, those columns are weighted averages
that smooth over heterogeneity. On a 1-device fleet, they are
exact descriptions of one machine.

## The discipline this enforces

There is a small but real discipline in printing a one-row
table with `100.00%` in the share column. The discipline says:
the report is a description of the data as it stands, not a
statement about whether the question the report answers is
"interesting" right now. The interestingness of the question is
the operator's call, not the tool's. The tool's job is to
return an honest answer, with all the contextual columns
intact, even when the headline column has collapsed to its
trivial value.

That discipline is the same one that makes drop counters
useful (they print zeros when nothing was dropped, so you can
trust them when they print non-zeros), the same one that makes
floor reporting useful (display floors prevent the table from
silently hiding noise that the operator might want to see),
and the same one that should make `100.00%` device shares
useful — because the only way to detect when N moves from 1
to 2 is to have been printing the N=1 row all along.

## The data point in summary

One device. 1,407 rows. 877 active hours. 15 models. 6
sources. 8.33 billion tokens. 269-day window. Output is 0.4%
of input. Cache hit mass is 148% of new input mass. Duty cycle
is 13.6%. Coarse-grained sync, batched per ~38 minutes of
active time. Started 2025-07-30, still going as of 2026-04-25
03:30 UTC.

That is what one row of device-share output, with `100.00%` in
the share column, can tell you when you read the row instead
of the column. Whoever is operating this queue is operating it
from one machine, in long agent sessions, against a multi-source
multi-model setup, with extremely heavy context reuse, on a
schedule that looks human. None of that comes from the share
column. All of it comes from the columns that survive when
share collapses to its trivial value — which is the case for
keeping the report wired up even on the day N equals one.

The day N moves to 2 is the day the share column starts
carrying information again. Until then, the rest of the row
has been the report all along.
