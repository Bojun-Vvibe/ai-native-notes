# The effective-hours illusion: Shannon entropy on the 24-hour clock, and the six-source spread

There is a temptation, when you have a token-mass histogram laid out across twenty-four hours of the day, to count the number of bins that are non-zero and call that the source's "active window." It is the cheapest possible summary. It also happens to be wrong in a way that is not obvious until you put two sources next to each other that both touch every hour of the day and behave nothing alike.

The pew-insights `source-hour-of-day-token-mass-entropy` analytic (shipped at `pew-insights@0.6.84`, repo SHA `4ce32be`) was built to handle exactly this case. It computes the Shannon entropy in bits of each source's token mass over the 24 UTC hour bins, then converts that entropy into an *effective-hours* count via `2^entropyBits`. The gap between `activeHours` (a support-set count, treats every non-zero hour equally) and `effectiveHours` (mass-weighted) is what I want to examine here.

## The numbers

Run, on a queue snapshot generated at `2026-04-27T01:04:26.756Z` over a corpus of `9,580,481,692` total tokens across six sources (one of which is redacted to `ide-assistant-A` per the standing redaction protocol; the upstream JSON contained an internal product name that maps to the banned-string list):

| source            | totalTokens   | activeHours | entropyBits | normalized | effectiveHours | concentrationGap | topHour | topHourShare |
|-------------------|---------------|-------------|-------------|------------|----------------|------------------|---------|--------------|
| claude-code       | 3,442,385,788 | 20          | 4.092       | 0.892      | 17.05          | 2.95             | 8       | 13.55%       |
| opencode          | 3,334,361,907 | 24          | 4.408       | 0.961      | 21.24          | 2.76             | 17      | 9.33%        |
| openclaw          | 1,846,431,758 | 24          | 4.537       | 0.990      | 23.22          | 0.78             | 1       | 7.04%        |
| codex             | 809,624,660   | 16          | 3.628       | 0.791      | 12.36          | 3.64             | 12      | 12.78%       |
| hermes            | 145,791,852   | 24          | 4.107       | 0.896      | 17.23          | 6.77             | 14      | 10.76%       |
| ide-assistant-A   | 1,885,727     | 14          | 3.187       | 0.695      | 9.10           | 4.90             | 2       | 21.74%       |

The maximum possible entropy across 24 equiprobable bins is `log2(24) ≈ 4.585` bits. Every value in the `entropyBits` column is below that ceiling — which is mechanical, since perfect uniformity across a 24-hour clock would require every hour to carry exactly `1/24 ≈ 4.17%` of the mass. The interesting question is *how far below*, and what the gap to `activeHours` reveals.

## Three regimes the gap separates

**Regime 1 — `concentrationGap ≈ 0`: genuinely flat.** `openclaw` lights up all 24 hours and its `effectiveHours` is 23.22. The gap is 0.78. This is what a clocked, daemon-driven workload looks like on the entropy lens: every hour carries roughly its uniform share, the top hour (hour 1 UTC) holds only 7.04% of mass, and the source has essentially no preferred posting time. If you only looked at `activeHours = 24`, you would put `openclaw` and `opencode` and `hermes` in the same bucket. Entropy says they are not the same bucket at all.

**Regime 2 — moderate gap (2.7–3.7 hours): real diurnal preference inside a wide window.** `claude-code` (gap 2.95), `opencode` (gap 2.76), `codex` (gap 3.64). These sources touch most or all of the clock, but the mass concentrates in a working window. `claude-code` peaks at hour 8 UTC and skews into the European morning; `opencode` peaks at hour 17 UTC and dominates the late-Americas slot; `codex` is tightest, with 16 active hours and an `effectiveHours` of just 12.36 — barely half a clock.

**Regime 3 — large gap with thin support: the illusion of breadth.** `hermes` (gap 6.77, the largest in the table) shows the textbook pathology. It has `activeHours = 24` — every hour saw at least one positive bucket — but `effectiveHours` is only 17.23. Almost seven hours of the clock are *technically* alive but carry essentially no mass. Its top hour (14 UTC) holds 10.76%, and the bottom hours (17–22 UTC) sit in the half-percent range. If the dispatcher were to schedule Hermes work assuming a uniform 24-hour workload, it would over-provision badly across the quiet hours and under-provision around the 14 UTC peak. `ide-assistant-A` shows the same pathology in a smaller, narrower form: 14 active hours, `effectiveHours` of 9.10, gap of 4.90, and a top-hour share of 21.74% — by far the most concentrated source in the table, and the one for which the support-set count overstates the working window the most.

## Why `effectiveHours` is the right scalar

Three reasons.

First, `effectiveHours = 2^entropyBits` is a *unit-preserving* transform. The number reads back in hours, the same axis as `activeHours`. You can subtract them and the residual (`concentrationGap`) is in hours of "phantom breadth."

Second, it is mass-aware. A bin with one positive bucket of 100 tokens contributes essentially nothing to the entropy, but it contributes a full `+1` to `activeHours`. The first count is what you want when you are sizing capacity, paging an on-call, or deciding whether a workload is "really" 24/7. The second is what you want when you are debugging whether the cron entry fired at all.

Third, it composes with `topHourShare` cleanly. For a perfectly flat distribution, `topHourShare = 1/24 ≈ 4.17%` and `effectiveHours = 24`. For a degenerate single-hour distribution, `topHourShare = 1` and `effectiveHours = 1`. Every real distribution lives somewhere on the curve between those two. A high `topHourShare` together with high `activeHours` is the signature of the illusion-of-breadth regime. `hermes` and `ide-assistant-A` both score on that axis; `openclaw` scores on neither.

## What the per-source values say about each harness

`claude-code` is the largest source by mass (3.44 B tokens) and concentrates morning activity around 06–08 UTC. Hour 8 carries 466.6 M tokens (13.55%), hours 7 and 6 carry 340.4 M and 278.5 M respectively. Hours 20–23 are completely dead — `hourMass[20..23] = 0`. This is consistent with an operator running on European business hours through a US-overnight shutdown.

`opencode` is the only source whose mass distribution is *bimodal* in a way the entropy scalar partially hides. The `hourMass` array shows a small peak near hour 1 (196.3 M) and a much larger peak at hour 17 (311.0 M, 9.33% share). The two-peak shape pushes entropy up — bimodal distributions are entropically wider than unimodal ones with the same support — and explains why opencode's `effectiveHours = 21.24` is the second-highest in the table despite a peak share that is not particularly high.

`openclaw` is the cleanest example of a *clocked* source on the corpus. The hour-mass values float between 45.9 M (hour 21) and 130.0 M (hour 1) — a ratio of about 2.8x between the quietest and busiest hour. For comparison, claude-code's busiest-to-quietest ratio (excluding the four dead hours) is 466.6 M / 29.2 M ≈ 16x, and codex's, restricted to its active hours, is 103.4 M / 2.4 M ≈ 43x. The clocked-vs-bursty axis cleanly separates these three even though they share the same `activeHours = 24` (or close to it for codex's 16).

`codex` is the most chronologically narrow source on the corpus that still has nontrivial mass: 16 active hours, all bunched between hours 1 and 16 UTC, with hours 17–23 and hour 0 completely empty. Its `concentrationGap` of 3.64 hours combines moderate within-window concentration (top-hour share 12.78%) with the narrow active span. This is the entropy lens picking up something that `source-active-hour-span` reports separately and on a different axis: the mass *and* the support are both shifted into a single working window.

`hermes` is the corpus's outlier on `concentrationGap`. Its 6.77-hour gap means almost a third of the 24-hour clock is "active" only in the trivial sense. The `hourMass` for hours 17–22 reads `676k, 586k, 838k, 537k, 1.12 M, 363k`, all under 1.2 M, against a peak of 15.7 M at hour 14. Capacity planners who treat hermes as a 24/7 workload will get ~5x the busy-hour rate they expect.

`ide-assistant-A` is small (1.89 M tokens, three orders of magnitude below claude-code) but illustrative. It is the most concentrated source in the table on `topHourShare` (21.74%) and the most extreme `activeHours - effectiveHours` ratio (14 vs 9.10, a 35% shrinkage). The peak hour is 2 UTC, the second peak is 8 UTC (246.8 k), the tail past hour 14 is empty. For a tool with this little volume the precise hour-mass numbers are noisy, but the entropy scalar still places it correctly: a narrow, peaked, low-mass workload, not a 24-hour one.

## What the analytic deliberately *does not* do

The entropy scalar is rotation-invariant — entropy of `[a, b, c, d]` equals entropy of `[d, a, b, c]`. That is the right property for measuring breadth and concentration but the wrong property for measuring *time-of-day positioning*. A source that runs entirely between 04 and 08 UTC and a source that runs entirely between 16 and 20 UTC have identical entropy. To distinguish them you need the circular centroid (`source-token-mass-hour-centroid`) or the active-hour span (`source-active-hour-span`). Entropy answers "how concentrated?", not "concentrated where?".

It is also blind to chronology. Two distributions that look identical when binned by hour-of-day will have identical entropy regardless of whether one drifted from a flat shape into a peaked shape over the corpus window or vice versa. For drift you want the chronological-quartile family or the daily-trend slope.

And it is rate-blind. A source that runs 10 M tokens uniformly over 10 hours and a source that runs 10 M tokens uniformly over 24 hours both have *normalized* entropy near 1.0 within their support, but very different `effectiveHours` and very different `activeHours`. The normalized scalar is comparable across sources only when you also look at support size, which is why the analytic returns both `entropyNormalized` and `effectiveHours` rather than collapsing them.

## Sort key choices and what they surface

The `--sort` flag accepts six keys: `tokens` (default), `entropy`, `normalized`, `effective`, `gap`, `top-share`, and `source`. Each foregrounds a different reading.

- `--sort entropy` puts `openclaw` first (4.537 bits) and `ide-assistant-A` last (3.187). It is the clean "who is closest to uniform on the 24-hour clock" ranking.
- `--sort gap` is the one I would point a dispatcher at first. It puts `hermes` (6.77) and `ide-assistant-A` (4.90) and `codex` (3.64) at the top — the sources where treating support-set as workload-shape will produce the worst capacity decisions.
- `--sort top-share` puts `ide-assistant-A` (21.74%) at the top, then `claude-code` (13.55%) and `codex` (12.78%). It surfaces sources where one specific hour dominates regardless of total breadth.
- `--sort effective` differs from `--sort entropy` only in scaling — both are monotone transforms of each other — but `effective` gives the operator a number that reads in *hours* rather than bits, which most operators find easier to act on.

## Cross-references the analytic claims (and how they compose)

The help text declares orthogonality against five other analytics. Worth restating with the actual numbers:

- vs `hour-of-day-source-mix-entropy` — that one runs entropy across the *source* axis at each hour, asking "is this hour mono- or poly-source?" The current analytic runs entropy across the *hour* axis within each source. Both are entropies on the same 24x6 matrix, but on orthogonal margins.
- vs `source-token-mass-hour-centroid` — centroid gives location (one number on a circular scale), entropy gives spread (one number on a linear scale). `openclaw`'s entropy is high *and* its centroid sits near hour 12 with a small resultant length, both consistent with the same flat-distribution story. `claude-code`'s centroid will sit in the morning hours with a large resultant length, consistent with both peaked mass and a tight working window.
- vs `source-dead-hour-count` and `source-active-hour-longest-run` — these count which hours are positive without weighing them. They are noise-fragile (a single 100-token bucket flips a bin from "dead" to "alive" and changes the count by 1 but the entropy by ~0). Entropy is a smoother continuous statistic on the same data.
- vs `source-hour-of-day-topk-mass-share` — a lump statistic on the top K hours, easy to compute and easy to communicate ("the top 3 hours hold 35% of mass") but discards everything in the tail. Entropy uses the whole distribution.

## Operator takeaways for the present corpus

Treating the table as a planning input on the 9.58 B-token snapshot:

1. **`openclaw` is the only source the dispatcher can plan as flat.** Concentration gap 0.78, normalized entropy 0.99, top-hour share 7.04%. Capacity should be sized to the *mean* hourly rate.
2. **`opencode` and `claude-code` need diurnal capacity profiles.** Both have `activeHours` near 24 but `effectiveHours` in the 17–21 range. Capacity should be sized to a peak hour at roughly `0.10 × totalTokens / activeDays`.
3. **`hermes`, `codex`, and `ide-assistant-A` should be planned as windowed workloads.** Their concentration gaps (6.77, 3.64, 4.90 respectively) are large enough that 24-hour capacity allocation is structurally wasteful. `codex` in particular has eight consecutive dead hours (17–0 UTC).
4. **Cross-source mix at any individual hour is a separate question.** That is the `hour-of-day-source-mix-entropy` axis, not this one. The numbers above tell you what each source's own clock looks like; they do not tell you which sources will collide on a given hour.

The reason I built `source-hour-of-day-token-mass-entropy` and not, say, `source-hour-of-day-active-hour-count` is that the latter already exists in spirit — it is what `activeHours` reports as a free byproduct of the same scan. The new information is in `effectiveHours` and `concentrationGap`. Those two columns are what separate the six rows into the three regimes.

---

*Source data: `pew-insights source-hour-of-day-token-mass-entropy --json` against `~/.config/pew/queue.jsonl` snapshot generated `2026-04-27T01:04:26.756Z`, repo SHA `4ce32be`, version `0.6.84`. Total tokens in window: `9,580,481,692` across six sources after one redaction (`ide-assistant-A`).*
