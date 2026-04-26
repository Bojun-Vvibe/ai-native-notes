# Day-of-Week Token Mass: The Friday Spike, the Monday Inversion, and Why Row Counts Lie

Bucket-row counts and token mass are not the same signal. Most fleet
dashboards quietly assume they are, because the row is the natural unit
of the underlying log file and the token is the natural unit of cost,
and people grab whichever is easier to compute. Today I want to show
what happens when you split a queue of hourly token-usage rows by
day-of-week and look at both axes together. The two distributions are
so far apart that they argue with each other about which day is the
busiest.

The data is `~/.config/pew/queue.jsonl`, 1,511 hourly buckets. Each row
has a `total_tokens`, an `hour_start` timestamp in UTC, plus
source/model fields. I bucketed by UTC day-of-week (the file spans
several weeks, so each DOW aggregates multiple weeks of rows), and the
shape that came out is one I have not seen anyone discuss in this
corpus: row count peaks on Monday, token mass peaks on Friday, and the
two curves are basically anti-correlated across the work week.

## The numbers

Query, run at `2026-04-26T02:43:45Z`:

```
jq -s -f /tmp/dow.jq ~/.config/pew/queue.jsonl
```

where `/tmp/dow.jq` is:

```
map(select(.hour_start != null and .total_tokens != null))
| map(. + {dow: (((.hour_start | sub("\\.[0-9]+Z$"; "Z") | strptime("%Y-%m-%dT%H:%M:%SZ") | mktime / 86400 | floor) + 4) % 7)})
| group_by(.dow)
| map({
    dow: .[0].dow,
    dow_name: (["Thu","Fri","Sat","Sun","Mon","Tue","Wed"][.[0].dow]),
    rows: length,
    total_tokens: (map(.total_tokens) | add),
    mean_tokens: ((map(.total_tokens) | add) / length | floor)
  })
| sort_by(.dow)
```

(The `+4` shift exists because `mktime / 86400 | floor` produces days
since the Unix epoch, and the Unix epoch was a Thursday. Modulo 7 then
maps day-since-epoch to a stable DOW index.)

Verbatim output:

```
[
  {"dow": 0, "dow_name": "Thu", "rows": 104, "total_tokens":  720131075, "mean_tokens": 6924337},
  {"dow": 1, "dow_name": "Fri", "rows": 214, "total_tokens": 2039860596, "mean_tokens": 9532058},
  {"dow": 2, "dow_name": "Sat", "rows": 229, "total_tokens": 1264614069, "mean_tokens": 5522332},
  {"dow": 3, "dow_name": "Sun", "rows": 257, "total_tokens": 1497445355, "mean_tokens": 5826635},
  {"dow": 4, "dow_name": "Mon", "rows": 267, "total_tokens":  907257660, "mean_tokens": 3397968},
  {"dow": 5, "dow_name": "Tue", "rows": 245, "total_tokens":  952209630, "mean_tokens": 3886569},
  {"dow": 6, "dow_name": "Wed", "rows": 195, "total_tokens": 1547754561, "mean_tokens": 7937202}
]
```

Total: 1,511 rows, 8.93 billion tokens summed across the entire DOW
breakdown. (The fleet headline of "around a billion tokens a day" is a
weekly average: 8.93B / 7 ≈ 1.28B per UTC day across this snapshot
window.)

## What is actually happening in this table

Look at the row count column first. It peaks on Monday at 267 buckets
and falls monotonically through the work week: 267 → 245 → 195 →
back up to 104 on Thursday because Thursday partial weeks are
underrepresented here. The weekend (Sat 229, Sun 257) is healthy but
not the peak.

Now look at the token mass column. It peaks on Friday at 2.04 billion
tokens — almost exactly twice the Monday total of 0.91 billion.
Wednesday is second at 1.55B. Sunday is third at 1.50B. Monday and
Tuesday — the row-count leaders — are the two lowest mass days of the
work week.

The mean-tokens-per-bucket column makes the inversion explicit. Friday
buckets are doing 9.53M tokens each on average. Monday buckets are
doing 3.40M. Same operator, same fleet, same model availability, same
tooling. The bucket-shape changes by a factor of 2.8 between Monday and
Friday for reasons that are not visible in the row count.

## Why row count and token mass disagree

A row in `queue.jsonl` is an hourly aggregate. It exists if and only if
some token-burning activity happened in that hour for that
source/model. It does not exist if you were idle. So the row count for
a given DOW is essentially a measure of **temporal coverage**: how many
distinct hours-of-the-day did you do something?

Token mass is a measure of **intensity**: when you did something, how
much of it did you do?

The two can move together (a busy day has both more covered hours and
more tokens per hour) or they can split apart, and on this fleet they
split apart hard. Monday has the most covered hours but the least
intensity per covered hour. Friday has fewer covered hours but vastly
more intensity per covered hour. So the question becomes: what kind of
work concentrates intensity into fewer hours, and what kind of work
spreads thin activity across more hours?

The honest answer for this corpus is that the two days are doing
different things.

Mondays look like exploration days. Many short, low-token sessions
spread across the day. The shape is consistent with a human at the
keyboard testing ideas, doing small refactors, reading code, asking
the agent diagnostic questions. Each individual hour has activity, but
no individual hour is dominated by a heavy job.

Fridays look like commit-and-burn days. Fewer hours of activity but
each hour is doing nearly three times the work of a Monday hour. The
shape is consistent with longer agent loops, batch jobs, the
"finish-this-feature-before-the-week-ends" sprint pattern that anyone
who has worked on a Friday recognises in themselves.

I cannot prove this from the row data alone — there is no
"work-mode" field in `queue.jsonl` and the snapshot window is short
enough that some of the unevenness could be a single project's
schedule rather than a true weekly rhythm. But the gap is large enough
(2.8× on mean-per-bucket) that the difference is not statistical noise.

## Wednesday: the second peak that does not fit the story

If the working theory were "intensity rises monotonically toward the
end of the week" then Wednesday would be a soft middle. It is not.
Wednesday clocks in at 7.94M mean tokens per bucket — second only to
Friday and well above the Mon/Tue/Sat/Sun cluster of 3.4M–5.8M.

There are two plausible explanations for the Wednesday hump and I do
not know which is right.

The first is that Wednesday is when long-running autonomous jobs that
were started earlier in the week are still grinding. If you launch a
multi-day reasoning loop on Monday afternoon, the bulk of its token
consumption lands on Tuesday and Wednesday, not on Monday when the row
that originally created the job is logged.

The second is that Wednesday is the natural mid-week deadline for
internal review cycles — code reviews, doc reviews, agent-driven PR
analysis batches that sweep through the week's accumulated diffs. The
shape of mid-week token spikes in any organisation that runs daily
standups is often Wed-shaped because Wed is when "the week" has enough
material to be worth processing in bulk.

The DOW data alone cannot distinguish these explanations. What it can
do is rule out one tempting story: "weekday vs weekend is the only
real boundary in this fleet." That story is wrong. Inside the workweek
there is more variance (3.40M Mon to 9.53M Fri = 2.8×) than there is
between weekend and weekday on average. The Sat/Sun mean of (5.52M +
5.83M) / 2 = 5.67M sits comfortably inside the Tue–Wed range. The
weekend is not a quiet time; it is a normal-intensity time with a
different row-count profile.

## What an alert built on this data would look like

The standard "tokens per day" alert is a single rolling number. If you
threshold on, say, "alert if today's token count is >2× the trailing
seven-day mean," you will generate a false positive every Friday and
miss every Monday slowdown. The trailing-seven-day mean is the wrong
baseline because the underlying distribution is not stationary across
DOW.

A DOW-aware baseline is straightforward to write. For each day,
compute the historical mean token mass for that DOW (over the trailing
N weeks), and threshold against that. Friday gets compared to Friday,
Monday to Monday. The same logic applies to row count: alert on
"unusually low row count for a Monday" rather than "unusually low row
count, period," because Mondays are supposed to have the highest row
count and a Monday with Tuesday-shaped row count is a real anomaly
that the unconditional alert would never catch.

This is just the standard advice for alerting on cyclical data, but
the size of the cycle here surprised me. I had been mentally treating
DOW as a small-amplitude effect on top of a roughly flat baseline. It
is not small. Friday is more than 2× Monday on token mass. Any single
threshold that fires on both will be either too tight for Monday or
too loose for Friday.

## What about the empty cells?

One detail worth flagging: Thursday has only 104 rows, the lowest in
the table. That is not because Thursdays are quiet on this fleet; it
is because the snapshot window happens to include fewer Thursdays than
other days. The mean-per-bucket column corrects for this — Thursday
mean of 6.92M tokens is healthy, third behind Friday and Wednesday —
but the total-tokens column is misleading for Thursday because it is a
sum over fewer weeks.

This is a generic hazard of DOW analysis on short windows. Always
report row count alongside totals. If you only quote the totals
column, Thursday will look like a low-volume day; if you only quote
the means, you lose the temporal-coverage signal that makes Monday
distinctive.

## The takeaway

Three numbers, captured at `2026-04-26T02:43:45Z`:

- Friday total: 2,039,860,596 tokens across 214 buckets (9.53M
  per bucket).
- Monday total: 907,257,660 tokens across 267 buckets (3.40M per
  bucket).
- Friday-to-Monday ratio: 2.25× on total mass, 0.80× on row count,
  2.80× on per-bucket intensity.

Row count and token mass are independent dimensions. They do not move
together inside the workweek for this fleet, and an alerting strategy
that conflates them — or that uses a single weekly baseline rather
than a DOW-conditional one — will be wrong in opposite directions on
opposite days. The Friday spike is real, the Monday floor is real, and
the Wednesday secondary peak is real enough to warrant its own
investigation. None of these are visible if you only look at one
column at a time.
