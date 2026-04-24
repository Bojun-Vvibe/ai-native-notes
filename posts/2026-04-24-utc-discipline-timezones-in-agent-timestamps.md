# UTC Discipline: Timezones in Agent Timestamps

Every agent system I have ever debugged has, at some point, had a bug
caused by timezones. Not a complicated bug. A stupid bug. A "this thing
ran twice on Sunday because the cron was in local time and the log was in
UTC and we crossed the DST boundary" bug. A "the dashboard says the spike
happened at 3pm but the on-call got paged at 11pm" bug. A "the rate limit
window resets at midnight in whose timezone exactly" bug.

The fix is always the same: **UTC everywhere, conversion at the edge,
never in the middle**. The trouble is that nobody actually does this,
because the language standard library makes the wrong default the easy
default, the database makes the wrong default the easy default, the JSON
serializer makes the wrong default the easy default, and the human
operators want to read times in their local zone, which means somewhere
in the pipeline a conversion has to happen, and that conversion is where
all the bugs live.

This post is a tour of the specific places timezones bite agent systems,
the specific defenses that work, and the specific defenses that look
like they work but don't.

## Why agents are worse than normal services

A traditional web service has roughly two timestamp surfaces: the
database row and the user-visible UI. You normalize at the database
boundary (UTC in, UTC out, convert in the view layer) and you are done.

Agents have a lot more timestamp surfaces. Some I have had to debug:

- The wall-clock time the user sent the message
- The wall-clock time the agent received the message
- The wall-clock time the agent dispatched the first tool call
- The wall-clock time the tool call returned
- The wall-clock time the model started streaming
- The wall-clock time the model finished streaming
- The wall-clock time the response was committed to durable storage
- The wall-clock time the cron job that triggered this run was scheduled
- The wall-clock time the cron job actually fired
- The "logical" time embedded in the prompt as "today's date is..."
- The "logical" time the agent thinks it is, derived from a tool call
- The expiration time on a cached tool result
- The retention horizon for the conversation log
- The "next run after" time for a scheduled retry
- The token expiration time on every credential the agent uses

Each of these can independently be in UTC, in machine local time, in the
user's local time, in the time zone of the data center where the request
landed, or in some weird hybrid (timestamp in UTC, but date string in
local — I have shipped this bug at least three times).

When two of them get compared and they are in different zones, you get a
wrong answer. The wrong answer might be "this cache entry never expires"
or "this scheduled run never fires" or "this conversation looks like it
happened in the future" or "this rate limit window is 25 hours long for
one day a year." All of these are real bugs I have seen.

## The DST sinkhole

Daylight saving time is the canonical timezone bug, and yet every team
re-discovers it. The two weekends a year when the clock skips an hour or
repeats an hour, half of all naive scheduling code does the wrong thing.

Specifically: if you store a "next run time" as a local-time string like
`2026-03-08 02:30:00` and you live in a DST-observing zone, then on the
spring-forward weekend the time `02:30` does not exist. Most libraries
will silently move it to `01:30` or `03:30`. Some will throw. Some will
return null and your scheduler will wake up at noon by accident.

On fall-back weekend, `01:30` happens twice. If your scheduler is
"trigger at 01:30 local," you get triggered twice. If your scheduler is
deduplicating on the local-time string, you skip one. Either way, the
behavior is wrong, and it is wrong only twice a year, which means you
won't notice until a customer complains.

The fix is to never store local-time strings as scheduler keys. Store
the absolute UTC instant, schedule on the absolute UTC instant, and
convert to the user's local zone only when displaying. If the user said
"run at 9am every weekday in my zone," the scheduler should compute
"the next UTC instant such that when converted to the user's zone is a
weekday at 9am" and store that, recomputing after every fire. The
recomputation step is the part people skip and it is the part DST
breaks.

## The "today's date" problem

Agents often need to know what day it is. The naive way to give them
this information is to inject "Today's date is {date}" into the system
prompt at session start, where `{date}` is computed from the agent
runtime's clock.

The runtime's clock is in some timezone. If the runtime is running in a
container in a data center in Virginia, the timezone is probably UTC.
The user is probably not in UTC. So at 11pm Pacific on March 14, the
agent thinks it is March 15 because the runtime is in UTC. The user
asks "what's on my calendar today" and the agent returns tomorrow's
calendar. Or worse, the user asks "schedule this for tomorrow" and the
agent schedules it for two days from now in the user's frame.

This bug is worse than DST because it happens every day for users west
of UTC during the late evening, and every day for users east of UTC
during the early morning. The fix is to know the user's timezone
explicitly and inject the user-zone date, not the runtime-zone date.

If you don't know the user's zone, the right answer is to be explicit
about your uncertainty: inject the UTC date, mark it as UTC, and let
the agent ask the user when ambiguity matters. Injecting the runtime
zone date and pretending it is "today" is the worst option because it
silently produces wrong answers most of the year.

## The serialization layer

JSON does not have a date type. Every JSON timestamp is a string, and
every string format is a convention. The conventions in common use:

- ISO 8601 with explicit Z suffix: `2026-04-24T15:00:00Z`. UTC, explicit.
  This is the only correct choice. Use it everywhere.
- ISO 8601 with explicit offset: `2026-04-24T08:00:00-07:00`. Also
  correct, but introduces ambiguity about whether the offset is "where
  this event happened" or "where the writer of the timestamp lives." I
  avoid it for inter-service timestamps.
- ISO 8601 with no zone: `2026-04-24T15:00:00`. Naive, dangerous, and
  unfortunately the default of several popular serializers. Reject this
  on parse.
- Unix epoch seconds or milliseconds: `1745514000`. Unambiguous, no zone
  to get wrong. Slightly less human-readable. I use these for log lines
  and machine-to-machine, ISO 8601 for human-readable artifacts.
- Local-time strings like `04/24/2026 3:00 PM`: never. Reject on parse.

The trouble is that your stack probably has at least one library that
emits naive ISO strings. Find it and fix it. Set up a serialization
test that round-trips a known UTC timestamp through every layer and
asserts the output ends in `Z`. If the test fails, you have a layer
that strips the zone, and that layer is going to bite you eventually.

## The database layer

Most modern databases have a "timestamp with time zone" type and a
"timestamp without time zone" type. The "without time zone" type is a
trap. It stores the digits and discards the meaning. Two writers
inserting "the same" timestamp from different zones will get rows that
sort differently than they expect.

Always use the "with time zone" type. Always insert UTC values. The
combination means the database stores the absolute instant and the
human-readable form is whatever you want it to be at query time.

A lot of teams skip this and use `DATETIME` or naive `TIMESTAMP` and
"just always insert UTC by convention." This works until somebody
new joins the team, doesn't know the convention, inserts a local-time
value, and now you have rows that are wrong by 8 hours and you can't
tell which ones because the database doesn't remember.

If you are stuck with a database that doesn't have a tz-aware timestamp
type (looking at SQLite), store unix epoch milliseconds as an integer.
Integers don't have zones. They can't be wrong about zones. The cost is
that you can't run human-readable date queries directly against the
column, but you couldn't reliably do that with naive timestamps either.

## Token expiration and credential refresh

Agents authenticate to a lot of services. Every credential has an
expiration. The expiration is usually given as either an absolute
expiry timestamp or a relative "expires in N seconds" duration.

Relative durations are unambiguous and fine. Absolute timestamps are a
zone bug waiting to happen. If the auth provider returns
`expires_at: "2026-04-24 16:00:00"` with no zone, what does it mean?
Probably UTC, but "probably" is not "definitely," and a bug here means
your agent quietly stops working at some unpredictable hour.

The defense: when you get an absolute expiry, immediately convert it
to a relative duration ("expires in N seconds from when I received
this") and store the duration plus the receive time, both in UTC. When
you check "is this token still valid," compute the absolute UTC instant
of expiry from your stored values and compare to the current UTC
instant. Never compare a stored expiry against `datetime.now()` without
checking the zone of both sides.

## Cache TTL

Cache entries have a TTL. The TTL is usually a duration, which is fine.
But sometimes the cache entry has an "expires at" wall-clock timestamp,
either because the cache layer wants to know when to evict or because
the entry was created from a downstream response that included its own
expiry.

Two failure modes I have seen here:

- The cache server is in UTC. The application is in the data center's
  local zone. The application writes "expires at" using its local clock.
  The cache server compares against UTC. Entries either never expire or
  always look expired, depending on the sign of the offset.
- The expiry was set based on a downstream service's clock, and the
  downstream service's clock is skewed by 30 seconds. Entries near the
  expiry boundary flicker between valid and invalid depending on which
  clock you compare against.

The fix for the first is the universal fix: UTC everywhere, no
exceptions. The fix for the second is to never compare wall-clock
timestamps from different machines for things that need second-level
precision. If you need that precision, use a single authoritative
clock (the cache server's, usually) and store TTLs as durations
relative to that clock.

## Log correlation across services

When you debug an agent failure, you almost always need to correlate
logs from at least three places: the agent runtime, the model provider,
and one or more tool services. Each of these may log in a different
zone. If you have to mentally convert "11:42:17 PDT" into "18:42:17
UTC" to line up with the next log, you will eventually make an
arithmetic mistake during a high-stress incident and chase a phantom
bug for an hour.

The defense is operational, not technical: every log line emitted by
anything you control logs in UTC with explicit `Z` suffix. Every
dashboard renders in UTC by default with a toggle for local. Every
on-call runbook says "all times are UTC unless explicitly marked
otherwise." Every postmortem timestamp is UTC.

The exceptions you can't control — the third-party tool service that
logs in some other zone — should be normalized to UTC at ingest, with
the original timestamp preserved in a separate field for audit.

## "Logical" time inside the agent

Some agents have a concept of "the time the agent thinks it is" that
is independent of wall clock. For example, a replay-style agent that
re-executes a past conversation should think it is the time of the
original conversation, not the time of the replay, because otherwise
"today's date" in the prompt drifts and the replay is no longer
faithful.

This logical clock has to be passed explicitly through the call stack.
If any layer falls back to `datetime.now()` instead of the logical
clock, the replay diverges from the original. I have lost a day to this
class of bug more than once. The defense is to make the logical clock a
required parameter on every function that reads time, with no default
fallback. If a function needs time and didn't get a clock, it should
fail loudly, not silently consult the wall clock.

## Cron jobs and scheduled retries

The cron daemon runs in some timezone. By default on most systems it is
the system's local zone. If your system local zone is UTC, you are
fine. If it is anything else, you have a DST problem twice a year and a
"my dashboard says this ran at the wrong time" problem every day.

Configure your cron environment to UTC explicitly. Don't rely on the
system default. Set `TZ=UTC` in the environment for every scheduled job.
Verify that the timestamps the job logs are UTC by reading them after
the first run.

For agent retry schedulers in particular, store "next attempt at" as
absolute UTC instants. Compute the next instant by adding the retry
delay to the current UTC instant, not by adding to a local-time
representation and converting back. The latter is mathematically
equivalent until DST, at which point it is off by an hour, and the
off-by-an-hour retries fire either too early or too late depending on
the direction of the DST jump.

## Timezones in the prompt

If you tell the model "today is April 24, 2026," it will do arithmetic
on that date. If you tell it nothing, it will hallucinate a date based
on its training cutoff. If you tell it "the current time is
2026-04-24T15:00:00Z," it will mostly use UTC for arithmetic, which is
usually not what the user wants when they say "remind me in 30 minutes."

The pattern that has worked for me: inject both the UTC instant and
the user's zone explicitly, e.g. "Current UTC time:
2026-04-24T15:00:00Z. User's local timezone: America/Los_Angeles.
When the user refers to times, they mean their local zone unless
they specify otherwise. When you store times for later, store them as
UTC."

Then do the conversion in tool implementations, not in the model. The
model is bad at timezone arithmetic, especially around DST. The tool
implementation has access to a real timezone library and can do it
correctly. The model's job is to know which zone the user means, not
to do the arithmetic.

## A short checklist

- All internal timestamps are UTC.
- All serialized timestamps end in `Z`.
- All databases use tz-aware columns or integer epoch.
- All cron jobs run with `TZ=UTC` set explicitly.
- The agent knows the user's zone and uses it when formatting output.
- The agent does not do timezone arithmetic; tools do.
- All credential expiries are stored as duration-from-receive-time.
- All log dashboards default to UTC.
- The "today's date" injection uses the user's zone, not the runtime's.
- DST is tested explicitly: round-trip a timestamp on the spring-forward
  weekend through every layer of your stack and verify it survives.

## Closing

UTC is not a preference. It is the only zone that doesn't observe DST,
the only zone that doesn't change with the seasons, and the only zone
that two services in different parts of the world can agree on without
any side channel. Every other zone is a presentation concern. Mixing
the two — letting presentation zones leak into storage or computation
— produces a class of bug that is small most of the time and
catastrophic some of the time, and the catastrophic times correlate
with weekends and holidays when nobody is around to catch them quickly.

The discipline is dull. Adopt it anyway. The dull thing is the thing
that prevents the 3am page on the morning of November 1.
