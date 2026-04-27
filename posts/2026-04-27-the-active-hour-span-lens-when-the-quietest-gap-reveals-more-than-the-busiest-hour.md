# The active-hour-span lens: when the quietest gap reveals more than the busiest hour

Every diurnal analytic in pew-insights so far has answered some variant of "where does the mass concentrate?" Centroids find the mean position on the circular clock. Top-K-mass-share lumps the busiest hours. Entropy measures spread without caring where the spread sits. They all start from the *positive* hours and ask how those positive hours are arranged.

`source-active-hour-span` does the inverse. It starts from the *negative* hours — the ones with zero observed token mass — and asks: what is the longest contiguous arc of silence on the 24-hour circular clock? Then it reports the complement: the smallest arc that covers every active hour. The output adds three numbers: `circularSpan` (= `24 - largestQuietGap`), `spanStartHour`, `spanEndHour`. The composition with `activeHours` produces a fourth: `spanDensity = activeHours / circularSpan`, the fraction of the cover arc that is actually positive.

This is a different question than entropy, and it surfaces different sources at the top of any sensible ranking.

## The numbers

From `pew-insights source-active-hour-span --json` against the same `~/.config/pew/queue.jsonl` snapshot generated at `2026-04-27T01:04:31.070Z`, on a corpus of `9,580,481,692` tokens across six sources (one redacted to `ide-assistant-A`; pew-insights repo SHA `4ce32be`, version `0.6.84`):

| source            | activeHours | circularSpan | spanStartHour | spanEndHour | largestQuietGap | spanDensity |
|-------------------|-------------|--------------|---------------|-------------|-----------------|-------------|
| claude-code       | 20          | 20           | 0             | 19          | 4               | 1.000       |
| opencode          | 24          | 24           | 0             | 23          | 0               | 1.000       |
| openclaw          | 24          | 24           | 0             | 23          | 0               | 1.000       |
| codex             | 16          | 16           | 1             | 16          | 8               | 1.000       |
| hermes            | 24          | 24           | 0             | 23          | 0               | 1.000       |
| ide-assistant-A   | 14          | 14           | 1             | 14          | 10              | 1.000       |

Two things jump out before any analysis. First, `spanDensity = 1.000` everywhere. That means every active hour for each of these six sources sits inside the smallest arc that contains them — there are no "holes" inside any source's working window. Second, the corpus splits cleanly into three groups by `largestQuietGap`: `0` (opencode, openclaw, hermes), `4` (claude-code), and the long-quiet pair `8` (codex) and `10` (ide-assistant-A).

That `largestQuietGap` column is the single most operationally useful number this analytic produces, and it tells a story that no other analytic in the pack tells the same way.

## What the gap actually means

The largest quiet gap is computed on the *circular* 24-hour axis. Hour 23 and hour 0 are adjacent. So a source that is positive on hours 22, 23, 0, 1, 2 and silent on hours 3..21 has a quiet gap of 19, not two short gaps split by the wrap. Conversely, a source positive on hours 1..16 and silent on hours 17..0 has a quiet gap of 8.

For each source above:

- **opencode, openclaw, hermes** — `largestQuietGap = 0`. Every hour has at least one positive bucket somewhere across the corpus. There is no defensible "off shift." Capacity has to be planned as 24-hour or accept arbitrary tail risk.
- **claude-code** — `largestQuietGap = 4`. The four-hour quiet block sits at hours 20..23 UTC inclusive (the `hourMass` array is zero on those bins). The active span is `[0, 19]`. This is the cleanest "there is a real off-shift" signal in the corpus.
- **codex** — `largestQuietGap = 8`. Eight consecutive zero-mass hours covering 17..0 UTC inclusive. The active span is `[1, 16]`. A third of the clock is structurally dead.
- **ide-assistant-A** — `largestQuietGap = 10`. Ten consecutive zero-mass hours from 15 onward through 0 UTC. Active span `[1, 14]`. Less than 60% of the clock ever sees a token.

The capacity-planning implication is direct: a process that needs to coexist with `claude-code` can safely take the `[20, 23]` window for noisy maintenance work. A process that needs to coexist with `codex` has an eight-hour evening-and-overnight UTC window. A process that wants the same window with `opencode` or `openclaw` has none.

## Why `spanDensity = 1` everywhere is meaningful

`spanDensity` is `activeHours / circularSpan`. For every source in this corpus it is exactly 1.000. That is not a coincidence and not a degeneracy of the metric — it tells you something specific about the shape of the data.

If `spanDensity = 1`, then between `spanStartHour` and `spanEndHour` (walking forward on the circular clock through the active arc), every hour is positive. There are no internal quiet hours. The work either happens within a contiguous window or it does not happen at all.

This is the *opposite* of what you would see for a workload with two daily peaks separated by an internal quiet block. A textbook example: a hypothetical source running European business hours then sleeping for six hours then running US business hours would have a positive arc that wraps through hour 0 with one or two internal zeros. Its `circularSpan` would be larger than its `activeHours`, and `spanDensity` would drop below 1. None of the six sources in the current corpus do this.

The interpretation: every source on this machine is operated as a single shift, not two. Even `claude-code`, with its 20 active hours and four-hour off-block, runs a single 20-hour push and a single four-hour break. There is no internal cron-fired silence in the middle of any source's working window.

## The relationship to `activeHours`, and why both numbers are returned

You might ask why `activeHours` and `circularSpan` are both reported when `spanDensity = 1` makes them numerically identical. The answer is that they are conceptually separate measurements, and they will diverge as soon as the corpus picks up a source with internal quiet hours. Returning both costs almost nothing on the JSON wire and protects the downstream consumer from having to recompute one from the other when the data changes.

The relationship in the present corpus:

- `claude-code`: `activeHours = 20`, `circularSpan = 20`, gap = 4. The cover arc is 20 hours, the gap is 4, totaling 24.
- `opencode`, `openclaw`, `hermes`: `activeHours = 24`, `circularSpan = 24`, gap = 0. The cover arc is the entire clock.
- `codex`: `activeHours = 16`, `circularSpan = 16`, gap = 8. Cover arc 16 hours, gap 8, total 24.
- `ide-assistant-A`: `activeHours = 14`, `circularSpan = 14`, gap = 10. Cover arc 14 hours, gap 10, total 24.

The arithmetic identity `circularSpan + largestQuietGap = 24` holds in every row. That is by construction — the cover arc and the largest quiet gap are complements on the circular clock — but it is worth checking against the numbers, because if a future change to the analytic produces a row where the identity fails, that row is a bug.

## Why `spanStart` and `spanEnd` matter on top of the gap

Two sources with the same `largestQuietGap` can have very different active windows depending on where the gap sits. `codex` and a hypothetical source with the same 8-hour gap covering hours 4..11 would both report `largestQuietGap = 8`, but the operational implications are completely different — one is "off in the evening," the other is "off in the morning."

The current corpus shows:

- `claude-code`: span `[0, 19]`, gap `[20, 23]`. The off-block is the late-evening UTC window. For US-centric operators that maps to afternoon-to-evening US Eastern; for European operators it is overnight.
- `codex`: span `[1, 16]`, gap `[17, 0]`. The off-block wraps through midnight UTC. The active window is morning-through-late-afternoon UTC.
- `ide-assistant-A`: span `[1, 14]`, gap `[15, 0]`. Tighter than codex by two hours on the back end. The active window ends mid-afternoon UTC.

Even between `codex` and `ide-assistant-A`, both narrow sources, the quiet gap geometries differ. `codex` is dark from 17 UTC; `ide-assistant-A` is dark from 15 UTC. A two-hour difference at the back end is meaningful when scheduling.

## The wrap problem, and why circular geometry is the right model

A naive linear computation of "longest run of zeros in the hourMass array" would give a wrong answer for any source whose quiet block wraps through hour 0. For `codex`, the linear-array view shows zeros at indices `[0, 17, 18, 19, 20, 21, 22, 23]` — eight zeros, but they appear as two runs of length 1 and 7 if you treat the array linearly. The circular view recognizes that index 23 is adjacent to index 0 and merges them into one run of length 8.

`source-active-hour-span` does this correctly. The `largestQuietGap = 8` for codex is the circular-merged value, not the linear maximum. Same for `ide-assistant-A` (gap 10, merged from indices `[0]` and `[15..23]`). For `claude-code` the situation is simpler — its four zeros at indices 20..23 do not wrap, so the circular and linear answers happen to agree.

This is the same circular geometry that `source-token-mass-hour-centroid` uses for its centroid calculation and that `source-dead-hour-count` uses for `longestDeadRun`. Without it, a source that sleeps from 22 UTC to 02 UTC would look like two separate two-hour gaps rather than one four-hour gap, and the operator would size capacity around the wrong window.

## What the analytic deliberately does not say

`source-active-hour-span` is silent on *mass*. The cover arc and the quiet gap are computed from the support set — the binary "is there any positive bucket here?" question — not from how much mass each hour carries. Two sources with identical `activeHours`, `circularSpan`, and `largestQuietGap` can have wildly different intra-window mass distributions. Pair the active-hour-span output with `source-hour-of-day-token-mass-entropy` to get both axes.

It is also silent on *chronology within the window*. A source whose `[1, 16]` window has been silent on hour 12 for the past month but was positive on hour 12 once during the corpus window will still report hour 12 as part of the active span. The analytic is conservative in this sense: it rolls up across the whole window. For per-day or per-week views you would slice the input with `--since` and `--until`.

It is silent on *cross-source overlap*. Knowing that `claude-code` runs `[0, 19]` and `codex` runs `[1, 16]` does not tell you how much they collide. The intersection of their active windows is `[1, 16]`, sixteen hours of potential collision. To measure the collision *intensity* you need to overlay their `hourMass` arrays — not their support sets.

And it is silent on *intra-hour structure*. UTC hours are wide. A source that fires for one minute at the top of every UTC hour and a source that fires uninterrupted for the full sixty minutes both score `activeHours = 24` and `largestQuietGap = 0`. To distinguish them you would need a finer time grain than the queue's hour bucket — which is outside the pew-insights bucket model entirely.

## Sort key choices

The analytic accepts `--sort` keys including `tokens` (default), `span` (`circularSpan` ascending — narrowest active windows first), `gap` (`largestQuietGap` descending — biggest off-blocks first), `density` (lowest first — most-illusory-coverage first), and `start` / `end` (`spanStartHour` / `spanEndHour` ascending). For a dispatcher trying to find the largest available evening window across the source pool, `--sort gap` is the natural call. It puts `ide-assistant-A` (10), `codex` (8), `claude-code` (4) at the top in that order and surfaces the windows worth claiming.

For an analyst trying to identify sources whose support sets overstate their active window, `--sort density` would be the choice — except that on this corpus every source has `spanDensity = 1` and the sort would be a tie-break on `tokens`. The density key starts to discriminate as soon as the corpus gains a source with internal quiet hours.

## How this composes with sibling analytics on the same axis

`source-dead-hour-count` reports `deadHours` (= 24 − `activeHours`), `liveHours`, `deadShare`, `longestDeadRun`, and `deadRunCount`. The `longestDeadRun` from that analytic equals `largestQuietGap` here — they are the same computation surfaced under two names because they answer two different questions ("how dead is this source?" vs "what is the active arc?"). `deadRunCount` is information not present in the active-hour-span analytic: it tells you whether the dead time is one block or several. For the present corpus, the `spanDensity = 1` rows imply `deadRunCount ≤ 1` for every source.

`source-active-hour-longest-run` reports the longest *contiguous* run of active hours. For sources with `spanDensity = 1`, that equals `activeHours` and `circularSpan`. Once a source gains internal silence, the longest active run drops below `circularSpan`, and the difference is exactly the size of the largest internal quiet block.

`source-token-mass-hour-centroid` answers "where on the clock is the mass centered?" The mean position on a circular axis is its own kind of summary. Combined with `source-active-hour-span`'s `[spanStartHour, spanEndHour]`, you get both a point and an interval. For `claude-code`: centroid near hour 9 with high resultant length, span `[0, 19]`. The centroid sits well inside the span and slightly past the midpoint — consistent with mass that piles on the early-morning UTC hours.

`source-hour-of-day-token-mass-entropy` answers "how concentrated is the mass within the active hours?" Combined with `source-active-hour-span`'s support measure, you get the spread within the window. For `openclaw`: span = full clock, entropy = 4.537 bits (near maximum). For `claude-code`: span = 20 hours, entropy = 4.092 bits (concentrated within the window).

The four analytics are complementary, not redundant. `source-active-hour-span` is the only one that names the *interval*. The others summarize over it.

## Operator takeaways for the present corpus

1. **`opencode`, `openclaw`, `hermes` cannot be planned around an off-block.** Every UTC hour shows positive mass. There is no quiet window to claim.
2. **`claude-code` has a small, defensible `[20, 23]` UTC off-block.** Four hours of silence, contiguous, no wrap. Workloads that need to share the same machine without colliding with `claude-code`'s active window can plan around this block.
3. **`codex` is the largest available off-block in the corpus: eight UTC hours from 17 through 0.** This is the natural slot for any nightly batch process that should not collide with codex's active window.
4. **`ide-assistant-A` has the largest off-block (ten hours, 15 through 0 UTC) but the smallest mass.** Useful as a guarantee — that source will not interfere with anything in the late-afternoon-through-overnight UTC window — but not useful as a capacity slot, because it carries only 1.89 M tokens out of the 9.58 B total.
5. **No source on the corpus has internal quiet hours.** Every source's working window is contiguous. The corpus has not yet picked up any workload that fires on a non-contiguous schedule. If one appears, `spanDensity` will be the first scalar to drop below 1 and will be the easiest signal to act on.

The active-hour-span analytic is the answer to the question "what is the longest stretch of clock I can claim?" — phrased on the circular axis, with the wrap handled correctly, and with the start and end of the active window named explicitly. Every other diurnal analytic on this axis answers a different question. None of them answers this one.

---

*Source data: `pew-insights source-active-hour-span --json` against `~/.config/pew/queue.jsonl` snapshot generated `2026-04-27T01:04:31.070Z`, repo SHA `4ce32be`, version `0.6.84`. Six sources, 9,580,481,692 total tokens, one redaction (`ide-assistant-A`).*
