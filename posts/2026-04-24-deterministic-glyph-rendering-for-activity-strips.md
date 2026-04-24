# Deterministic Glyph Rendering For Activity Strips

An activity strip is the one-character-per-day visualization you have seen on every contribution graph since GitHub shipped the green squares. One glyph, one bucket of time, one color. They look trivial. They are not. The first three implementations I wrote of this were all wrong in different ways, and the wrongness only showed up when I ran the same data through the renderer twice and got two different strips. Determinism is the whole game with these things, and most of the way you achieve it is by being ruthless about three details: UTC bucketing, the color rule, and the tie-break when two events land in the same bucket.

This post is about how to make the strip the same every time, why each of those three details matters more than it looks, and what the failure modes are when you get them wrong. I am going to use a thirty-day strip as the running example because that is what I render most often, but the same logic applies to ninety-day, year, or hour-of-day variants.

## The setup

You have a list of events with timestamps. You want to render a string of N glyphs, each glyph representing one day, where the glyph encodes some property of the events that fell into that day's bucket. Empty days get a placeholder. Days with events get a glyph that depends on the events. Often there is color too, ANSI escapes wrapping each glyph.

The output is a single line of text. It needs to be the same single line every time you call the function with the same input. That is the only requirement, and it is the requirement that everyone misses.

## Detail one: UTC bucketing, no exceptions

The single biggest source of nondeterminism in activity strips is timezone handling. The renderer runs on machine A in `America/Los_Angeles`. The same renderer runs on machine B in `Asia/Tokyo`. The same input data, the same code, two different strips. An event at `2026-04-24T07:00:00Z` lands in the April 24 bucket on machine B and the April 23 bucket on machine A. Now your CI is checking the strip into a snapshot test and it fails depending on which laptop runs it.

The fix is to bucket in UTC, always, with no user-facing setting. Take the timestamp, convert to UTC, truncate to the day, that is the bucket key. The strip is "the last N UTC days ending at the most recent UTC midnight that has fully elapsed." Not "the last N days in your local timezone." Not "the last N days in the timezone of the data source." UTC.

This will feel wrong to users in non-UTC timezones who know they wrote code at 11pm local on Tuesday and want to see the contribution show up under Tuesday, not Wednesday. You have two options. Option one: tell them no, the strip is in UTC, here is a tooltip that explains it. Option two: render two strips, one in UTC for snapshot stability and one in local time for human consumption, and never let the local-time one cross a process boundary. The local-time strip is allowed to be machine-dependent because it is for the human running the process, not for any other consumer.

Do not, under any circumstance, let the local timezone leak into the UTC strip. The most common way this happens is using the host's `time.Now()` to compute the strip's right edge:

```go
end := time.Now().Truncate(24 * time.Hour) // wrong
```

`time.Now()` returns local time. `Truncate` truncates relative to the zero time, which is UTC, but the conversion of the local clock to a duration-since-zero produces a different number depending on the offset from UTC. The strip's right edge becomes timezone-dependent. The fix:

```go
end := time.Now().UTC().Truncate(24 * time.Hour)
```

The `UTC()` call before `Truncate` is load-bearing. It is also the kind of call that gets removed during a "cleanup" pass by someone who does not know it matters. Add a comment. Add a test that runs with `TZ=America/Los_Angeles` and `TZ=Asia/Tokyo` and asserts identical output.

The right edge of the strip is also load-bearing. "The most recent UTC midnight that has fully elapsed" means if you run the script at `2026-04-24T00:30:00Z`, the right edge is `2026-04-24T00:00:00Z` and the strip ends today. If you run it at `2026-04-23T23:30:00Z`, the right edge is `2026-04-23T00:00:00Z` and the strip ends yesterday. The strip changes within a thirty-minute window depending on which side of UTC midnight you call it. This is correct behavior — the bucket for "today" is not yet closed — but it surprises people. Document it.

## Detail two: the color rule

The color rule is the function from `events_in_this_bucket -> glyph_with_color`. It needs to be a pure function. Same input, same output. No randomness, no time-of-day, no machine state.

The naive color rule is a histogram bucketing on count: 0 events is empty, 1-3 events is light, 4-10 events is medium, 11+ is heavy. This is fine until your event distribution shifts and suddenly every day is "heavy." The thresholds are wrong now. You re-tune them, the strip changes meaning, snapshot tests break, downstream consumers who were eyeballing the strip for trends are confused.

A better color rule is data-driven: compute the strip's quantiles over the visible window, then bucket each day relative to those quantiles. Day below the 25th percentile of nonzero days is light, between 25th and 75th is medium, above 75th is heavy. This auto-adjusts as the user's activity scales, which is what you want for a relative-intensity visualization.

But — and this is the part that bites you — the quantile-based rule is only deterministic if the quantile computation itself is deterministic, and quantiles on small samples have ties. If you have thirty days and twenty of them have exactly one event, the 25th percentile and the 75th percentile are both 1, and your bucketing rule degenerates. Every nonzero day becomes "medium." The strip looks uniform. The user complains that the visualization is broken.

The fix is a small one: when the quantile boundaries collapse, fall back to a fixed-threshold rule. Or better, use a method like "above the median is hot, at the median is warm, below is cool, zero is empty" which only has three nonzero buckets and is robust to collapsed quantiles. Pick one, document the fallback, write the test that constructs the collapsed-quantile case and asserts the strip still varies.

The other thing about the color rule: it has to be deterministic across runs that see different windows of the same data. If today's strip uses the 25th percentile of the last thirty days and tomorrow's strip uses the 25th percentile of the last thirty days, those two percentiles are different numbers because the window has shifted by one day. This is fine — the strip is supposed to be a sliding-window view — but it means yesterday's "heavy" day might be today's "medium" day even though the day's underlying data did not change. If you render strips into a long-lived log and expect them to be comparable across runs, you have a problem. The fix is to either (a) render the strip with a fixed reference window's quantiles instead of the visible window's, or (b) render the strip with absolute thresholds. Either is fine; mixing them is not.

## Detail three: tie-break when two events land in the same bucket

If your glyph encodes a single property derived from one event — say, the kind of event, where you have several kinds and each gets a different glyph — and a bucket contains events of multiple kinds, you need a tie-break rule. "What glyph does Tuesday get if Tuesday had a `commit`, a `push`, and a `revert`?"

There are three reasonable rules and you must pick one and stick to it.

**Rule A: priority order.** Each kind has a fixed priority, the highest-priority kind wins. `revert` beats `push` beats `commit`. The strip shows reverts loudly even on days with lots of normal commits. This is good for surfacing exceptional events.

**Rule B: count-weighted majority.** The kind with the most events wins. Ties within the bucket broken by priority. This is good for showing the dominant activity.

**Rule C: earlier wins.** The earliest event in the bucket determines the glyph. This is good when the order of events within a day is meaningful — first commit of the day, first failure of the day.

I have shipped all three in different contexts and rule C is the one I reach for most often, for a non-obvious reason: it is the only one that is stable when events arrive late. Rules A and B both depend on the full set of events for the bucket. If a late-arriving event for yesterday's bucket comes in after you have already rendered yesterday's glyph, the glyph might change. With rule C, the earliest event determines the glyph forever — late arrivals can only be later, never earlier, so the glyph is stable from the first time it is rendered.

That property — "once a bucket has any event, the glyph is stable against future additions" — is enormously valuable for any system that renders strips repeatedly and caches them. With rule A or rule B, you have to invalidate yesterday's glyph any time a new event arrives for yesterday's bucket. With rule C, you only have to render a bucket the first time it transitions from empty to nonempty, and never again. That is a real performance and correctness win.

The "earlier wins" rule does have a failure mode: if your event timestamps are not monotonic — say, the data source gives you events out of order — you might render the glyph based on the first event you happened to see, not the actually-earliest event. The fix is to always sort by timestamp before bucketing. Do not skip the sort because "the source emits in order." Sources lie.

## Putting it together: the rendering function

Here is the deterministic skeleton in pseudo-Go. It is shorter than you expect:

```go
func renderStrip(events []Event, days int) string {
    // 1. Right edge: most recent UTC midnight that fully elapsed.
    end := time.Now().UTC().Truncate(24 * time.Hour)
    start := end.AddDate(0, 0, -days+1)

    // 2. Sort events by timestamp ascending. Required for "earlier wins".
    sort.Slice(events, func(i, j int) bool {
        return events[i].At.Before(events[j].At)
    })

    // 3. Bucket: map[day_index] -> first event in that day, plus count.
    type bucket struct {
        first Event
        count int
    }
    buckets := make([]bucket, days)
    for _, e := range events {
        utcDay := e.At.UTC().Truncate(24 * time.Hour)
        if utcDay.Before(start) || !utcDay.Before(end.AddDate(0, 0, 1)) {
            continue
        }
        idx := int(utcDay.Sub(start) / (24 * time.Hour))
        if buckets[idx].count == 0 {
            buckets[idx].first = e
        }
        buckets[idx].count++
    }

    // 4. Compute quantile thresholds over nonzero buckets, with collapse fallback.
    counts := make([]int, 0, days)
    for _, b := range buckets {
        if b.count > 0 { counts = append(counts, b.count) }
    }
    sort.Ints(counts)
    p50 := percentile(counts, 0.5)

    // 5. Render: one glyph per bucket.
    var sb strings.Builder
    for _, b := range buckets {
        if b.count == 0 {
            sb.WriteString(emptyGlyph)
            continue
        }
        glyph := glyphForKind(b.first.Kind) // earlier-wins
        intensity := "warm"
        if b.count > p50 { intensity = "hot" } else if b.count < p50 { intensity = "cool" }
        sb.WriteString(colorize(glyph, intensity))
    }
    return sb.String()
}
```

A few things to notice:

**The right edge is computed from `time.Now()` and is the only nondeterministic input.** Everything else is a pure function of `events` and the right edge. For testing, parameterize the right edge — pass it in instead of computing it inside. This lets snapshot tests pin the strip to a known right edge and not flake at midnight.

**The bucket-membership check uses half-open intervals.** `utcDay.Before(start)` for the lower bound but `!utcDay.Before(end.AddDate(0,0,1))` for the upper, which is "utcDay is before tomorrow," i.e., utcDay is at most today. Using `<= end` would silently allow events on today's date even though today's bucket has not closed yet. Using `< end` would silently exclude today's bucket entirely. The half-open form is the only one that gives you exactly N buckets ending at the right edge.

**The first-wins tie-break is implemented by checking `count == 0` before assigning `first`.** This is the entire mechanism. The events list is already sorted ascending by time, so the first event encountered for a bucket is the earliest. Do not be tempted to "optimize" this by checking timestamps inside the loop — the sort already did that work, and adding a per-event timestamp comparison is both slower and easier to get wrong.

**The intensity rule uses `>` and `<` with `p50`, not `>=` and `<=`.** This means days exactly at the median are "warm." If you used `>=` for hot, the median day would flip to "hot" depending on which side of the comparison the median value happened to fall on, which depends on your percentile interpolation method. The strict-inequality form makes the median bucket always "warm" regardless of interpolation, which is the desired behavior.

## What goes wrong if you skip the discipline

I have seen all of these in production:

A strip that renders fine on the dev's machine and as a string of question marks on the CI server, because the CI server is in UTC and the strip's right edge is computed in local time, and the offset pushes the strip's window into the future where there are no events.

A strip that subtly changes shape every time it is regenerated, because the color rule depends on the visible window's quantiles and the visible window slides forward each run. The user complains that "yesterday looks different today than it did yesterday." It does, because yesterday's bucket is now compared against a different set of peer days.

A strip that flickers between two states when an event arrives for a previous day. Rule A or B with caching, no invalidation. The fix is either rule C or correct invalidation; teams often pick "render fresh every time" as the workaround, which is fine until the strip is being rendered ten thousand times a second.

A strip that has duplicate glyphs because the same event was bucketed twice — once by its UTC timestamp and once by its local-time timestamp, due to a code path that mixed the two. This one is the worst because it looks subtly wrong rather than obviously wrong, and you do not notice until someone points it out.

A strip with off-by-one days at the right edge, because someone "fixed" the bucket-membership check by changing `!utcDay.Before(end.AddDate(0,0,1))` to `utcDay.Before(end)` to "remove the awkward double-negative." The double-negative was load-bearing.

## The test set

Snapshot tests for the rendered string are necessary but not sufficient. The real tests are the ones that exercise the determinism contract:

1. Render with `TZ=UTC`, render with `TZ=America/Los_Angeles`, render with `TZ=Asia/Tokyo`, assert all three are identical.
2. Render twice with the same input but with `time.Now()` mocked to two times that are within the same UTC day, assert identical output.
3. Render with events presented in time order, render with events shuffled, assert identical output.
4. Render with one bucket containing two events of different kinds, assert the glyph matches the kind of the earlier event.
5. Render with all bucket counts equal, assert the strip is uniform and does not crash on the collapsed-quantile case.
6. Render with empty events, assert a strip of all-empty glyphs of the correct length.
7. Render with events outside the window, assert they are silently dropped and do not affect any glyph.

Seven tests. None of them need a mock framework. All of them catch a real bug I have hit. The whole suite runs in milliseconds.

## Closing

Activity strips look like a five-minute job. They are a five-minute job if you do not care about determinism. The moment they get checked into a snapshot test, or rendered into a log that is diffed across runs, or cached anywhere, the determinism becomes the entire contract. UTC bucketing, a color rule with documented behavior on collapsed quantiles, and "earlier wins" as the tie-break are the three disciplines that make the strip stable. Skip any one of them and you will spend an afternoon next quarter chasing a strip that "sometimes looks different" — and the diff will be one or two glyphs and the cause will be three layers removed from where the strip is rendered.
