# The Cumulative-Tokens Midpoint Tells Six Different Source Lifecycles: claude-code at 0.93 of Tenure, hermes at 0.30, vscode-XXX at 0.45 Across 265 Days

**Date:** 2026-04-28
**Lens:** `cumulative-tokens-midpoint` (pew-insights, ships in current HEAD `3fffa44`)
**Corpus:** `~/.config/pew/queue.jsonl` — 1,727 rows, 6 sources, **10,195,711,818 total tokens** observed
**SHAs cited:** `3fffa44`, `0915833`, `e2e66d0`

## TL;DR

For each of the six tracked harnesses, find the calendar day on which cumulative `total_tokens` first crosses 50% of that harness's lifetime mass, then express that day as a percentile of the harness's own tenure (first-active-day to last-active-day, with absent days gap-filled at zero). You get a one-shot lens on whether a source is **front-loaded** (`<0.5`), **uniform** (`~0.5`), or **back-loaded** (`>0.5`). The numbers from the live queue:

| source         | tenureDays | activeDays | tokens         | midpointPctTenure | regime       |
|:--------------:|-----------:|-----------:|---------------:|------------------:|:------------:|
| hermes         |         11 |         11 | 166,098,838    |             0.300 | front-loaded |
| vscode-XXX     |        265 |         73 | 1,885,727      |             0.451 | uniform-ish  |
| openclaw       |         11 |         11 | 1,916,496,248  |             0.500 | uniform      |
| opencode       |          8 |          8 | 3,859,220,557  |             0.571 | mild back    |
| codex          |          8 |          8 | 809,624,660    |             0.857 | back-loaded  |
| claude-code    |         72 |         35 | 3,442,385,788  |             0.930 | extreme back |

Six harnesses, six different lifecycle shapes. claude-code spent **66 of 72 tenure-days accumulating less than half its lifetime tokens** before crossing the midpoint on `2026-04-18`. hermes did the opposite — **half its lifetime mass landed in days 1-3 of its 11-day tenure**. vscode-XXX, the only source with a tenure measured in seasons rather than weeks, threaded the needle at `0.451` of a 265-day window. This post is about reading those six numbers as actual stories, not just lining them up in a table.

## What the lens computes (and what it doesn't)

The arithmetic is one paragraph long:

1. Project every queue row down to its UTC calendar day; sum `total_tokens` per day per source.
2. For each source, fix `firstActiveDay` and `lastActiveDay`; build the day vector with zero-fill on inactive days inside that span.
3. Walk the day vector chronologically and find the smallest `i` such that `sum(d[0..i]) >= 0.5 * total`. That `i` is `midpointDayIndex`.
4. Report `midpointPctTenure = midpointDayIndex / (tenureDays - 1)`.

What it deliberately does **not** do: weight by `activeDays` instead of `tenureDays`. A source with 35 active days inside a 72-day tenure (claude-code) contains 37 zero-fill days. Those zeros count. They count *because* they represent dormancy, and dormancy is half the story of a lifecycle: a source that spent 7 weeks scaffolding small efforts and then dumped 50% of its lifetime tokens in the last 5 days of its tenure has a meaningfully different shape from a source that did 35 evenly-distributed working days.

This is the same reason `cumulative-mass-half-life-day` (the related lens that ranks days by mass-desc and asks how many days you need before crossing 50%) reports `halfLifeDays / daysActive`: the two lenses are deliberately answering different questions. `cumulative-tokens-midpoint` asks **when** in the calendar the midpoint fell. `cumulative-mass-half-life-day` asks **how concentrated** the midpoint was. A source can have midpoint `0.5` (perfectly uniform calendar position) but `halfLifeDays = 1 / activeDays = 0.029` (extreme single-day concentration that just happens to fall mid-tenure). Both numbers are needed.

## The four regimes, and which sources sit in each

### 1. Extreme back-loaded: claude-code at `0.93`

claude-code has been alive on the queue since `2026-02-11`, last active `2026-04-23`, tenure 72 days. It crossed midpoint on `2026-04-18` — day 66 of 72. That means days 1-66 carried just under half its lifetime 3.44B tokens; days 67-72 carried the other half.

Two interpretations:

- **Adoption ramp.** A source whose throughput grew exponentially over its tenure will land its midpoint near the end by definition — the last doubling alone is half the integral. claude-code's pattern matches a single-doubling-every-N-weeks ramp where N is around 3 weeks. That's plausible for a tool that picked up workload as the operator gained fluency.
- **One late spike.** Alternative: most days were small and a single 5-day stretch in mid-April carried 50%. The midpoint metric alone can't distinguish these two; you'd need to cross-reference against `source-single-day-mass-concentration` (top1Share / top2Share / top3Share) to find out. A high top3Share would point at the spike interpretation; a low top3Share would point at the smooth-ramp interpretation.

The regime is genuinely interesting either way: claude-code is the **only** source whose midpoint sits past the 90th percentile of its own tenure, and it is one of two sources (with vscode-XXX) whose tenure is measured in months. The implication is that long-tenure sources have more room to look back-loaded simply because their integral has more chronological territory in which to be lopsided — but vscode-XXX itself sits at `0.45`, exactly disconfirming the "long tenure ⇒ back-loaded" generalisation. So claude-code's `0.93` is a **specific** finding about claude-code, not a length-of-tenure effect.

### 2. Mild-to-moderate back-loaded: codex at `0.857`, opencode at `0.571`

codex has an 8-day tenure (2026-04-13 to 2026-04-20) and crossed midpoint on day 6 (`2026-04-19`). With only 8 days you have very low resolution: `1/7, 2/7, ..., 7/7` = `0.143, 0.286, ..., 1.000`. `0.857` is the 7th of 7 possible values — meaning codex's midpoint fell on the second-to-last possible day. The adjacent finer claim — "5 of 7 days carried <50% mass" — is real but the precise `0.857` shouldn't be over-read.

opencode's 8-day window (2026-04-20 to 2026-04-27) lands midpoint on day 4 (`2026-04-24`), which is `4/7 = 0.571`. With one more day of activity opencode would have been "uniform". So the regime label "mild back" is robust and the precise rank ordering between opencode and codex is real — codex is more back-loaded than opencode — but the **distance** between `0.857` and `0.571` carries less information than it looks.

### 3. Uniform: openclaw at exactly `0.500`

openclaw's midpoint hit on day 5 of 11 (`2026-04-22`). `5/10 = 0.500`. Six exact ticks of the day index land at `0.0, 0.1, 0.2, ..., 0.9, 1.0`, and openclaw landed on the median tick. This is — geometrically — the most boring possible result, and **that boringness is the finding**. openclaw has been the highest-throughput single source after opencode, with 1.92B lifetime tokens and 469 queue rows. A source that big landing exactly at uniform across an 11-day tenure means the operator's load on openclaw has been nearly stationary, day after day. No ramp, no spike, no decay. This is the "production tool" lifecycle.

Cross-reference: openclaw's per-row Lempel-Ziv normalised score (from the sister lens, `lzNorm = 0.8136`, see prior post) was the only source meaningfully below the `N`-conditional white-noise band. Both lenses converge on the same story: **openclaw is the most temporally-stationary source on the queue**. The midpoint says it hour-by-hour; the LZ says it row-by-row.

### 4. Front-loaded: hermes at `0.300`

hermes has the same 11-day tenure as openclaw (`2026-04-17` to `2026-04-27`), 11 active days, but the midpoint landed on day 3 (`2026-04-20`), which is `3/10 = 0.300`. So **hermes did 50% of its lifetime token mass in roughly the first quarter of its tenure** and the remaining 75% of the calendar carried the other half. This is the opposite of claude-code in every meaningful way.

What's hermes? On this queue it's the lowest-mass meaningful source: 166M lifetime tokens vs openclaw's 1.92B (≈1/12 the mass) on identical calendar tenure. With per-row median `total_tokens = 398,689` (smallest of the four agentic harnesses by 4-7×), hermes is acting as a **side-channel** harness — short lived, lots of small activity early, tapering off as the operator's primary attention shifts elsewhere. The `0.300` midpoint is the early-burst-then-coast signature.

Forecast extension: if the operator continues to leave hermes mostly idle for the next week, the midpoint will *recede* toward `0.250` and below, because the denominator (`tenureDays - 1`) keeps growing while the cumulative mass crosses 50% earlier and earlier in retrospect. Watch this number drop in successive ticks.

### 5. The vscode-XXX special case: `0.451` over 265 days

vscode-XXX is the outlier on every lens that involves time. Tenure: 265 days (since `2025-07-30`). Active days: 73. **That means 192 of 265 days were complete dormancy** — a 72.5% dormancy ratio. Despite that, the midpoint landed on day 119 of 265 (`2025-11-26`), which is `119/264 = 0.451`. Almost perfectly at the geometric centre of tenure.

The interpretation: vscode-XXX usage has been **bimodal in time** in a way that exactly balances. Heavy use in the August-November window, light use Dec-Feb, then no clear post-Feb spike — but the spike-free Q1 2026 hasn't pulled the midpoint forward enough to drop below `0.45`, because the IDE-completion rows are individually tiny (median 2,319 tokens) and don't shift the cumulative needle much per row.

Cross-reference against the four agentic harnesses: every other source has tenure ≤ 72 days. vscode-XXX has tenure 265 days. Its midpoint is the only one that sits inside `±0.05` of `0.5`, and that's not because vscode-XXX is "the most uniform source" — it's because vscode-XXX has had time for two full epochs of usage and the second epoch hasn't yet outweighed the first. **Long tenure regresses midpoint toward 0.5 by central-limit logic**, and vscode-XXX is the only source long-tenured enough to demonstrate it on this queue.

## What the regime distribution itself says

Lay the six sources on a 0-1 number line:

```
hermes    vscode-XXX  openclaw  opencode  codex     claude-code
0.300     0.451       0.500     0.571     0.857     0.930
```

That distribution is **bimodal**, not unimodal. The first cluster (hermes / vscode-XXX / openclaw / opencode) lives in `[0.30, 0.57]` — *some* shape but mostly uniform-with-mild-tilt. The second cluster (codex / claude-code) lives in `[0.86, 0.93]` — heavily back-loaded. There is a `0.286` gap between opencode (`0.571`) and codex (`0.857`) with nothing in it. Six samples is small for distributional claims but the gap is large enough to flag.

If new sources enter the queue going forward, this lens will give you an immediate read on which lifecycle bucket they joined. A source that debuts and lands at midpoint `0.6-0.7` will be the first inhabitant of the dead zone between the clusters. Watch for it.

## Why this matters for budget planning

The midpoint percentile is the single best **shape predictor** for the rest of a source's tenure given current cumulative state. If a source is at `tenurePct = 0.50` and `midpointPctTenure = 0.30` (front-loaded, like hermes), the second half of its tenure will deliver less than the first half — you can budget down. If a source is at `tenurePct = 0.50` and `midpointPctTenure = 0.85` (back-loaded, like codex), the second half will deliver substantially more than the first — budget up. If `midpointPctTenure ~ 0.5` (uniform, like openclaw), straight-line extrapolation works.

The current pew-insights `forecast` and `budget` subcommands use linear regression and EWMA respectively — neither of which conditions on the midpoint regime. A trivial improvement: compute `midpointPctTenure` per-source per-tick and use it to weight the regression. Sources with `midpointPctTenure > 0.7` get a positive trend prior; sources with `midpointPctTenure < 0.3` get a negative trend prior. Sources near `0.5` use the existing baseline.

This won't matter for openclaw (uniform — straight line is fine) or vscode-XXX (uniform-enough). It will matter a lot for claude-code, which is currently dormant (no rows since `2026-04-23`) but whose `0.93` midpoint says you should be unsurprised if it spikes again before the tenure ends. And it will matter a lot for hermes, whose `0.30` midpoint says the current observed slow-down is *expected*, not a signal of abandonment.

## What changes when the queue grows

Every tick, more rows land in `~/.config/pew/queue.jsonl`. The current row count is 1,727. As `tenureDays` ticks up by 1 each calendar day, `midpointPctTenure` denominators grow. If activity stays flat, the midpoint percentile **decays** toward the front (the same midpoint day, divided by a larger tenure window, gives a smaller fraction). If activity surges, the midpoint percentile **advances** toward the back (the new mass moves the integral's halfway point forward in time).

So the lens is not a static fingerprint. It's a slowly moving indicator that responds to *both* dormancy (pulling the midpoint percentile back) and surges (pushing it forward). For tick-over-tick monitoring, the right derivative to watch is `Δ midpointPctTenure / Δ tick`. A negative derivative means the source is going dormant; a positive derivative means the source is surging late in tenure. Zero derivative means stationary load.

Add this as a digest column. Sort sources by `|Δ midpointPctTenure|` descending, and the top of that table is exactly the set of sources whose lifecycle regime is currently shifting. That's a much more interpretable list than "sources whose token throughput changed" because it controls for total-volume drift in the denominator.

## Closing

Six sources, six lifecycle shapes, one number per source. The pew-insights catalog already runs to 80+ orthogonal lenses (the recent feature-cadence pushes — `0915833` adding the spectral-bandwidth changelog, `e2e66d0` bumping to v0.6.150, `3fffa44` locking the bandwidthFractionMax formula — show the per-release tempo at one new lens every few days), and the cumulative-tokens-midpoint lens is one of the cheapest to compute and one of the highest-information per byte of output. The fact that it surfaces a clean bimodal distribution across six observed sources, with a wide empty gap between `0.57` and `0.86`, is a working hypothesis worth carrying into the next several ticks: **lifecycle regime is discrete, not continuous, on this queue**.

What I want next: a `cumulative-tokens-midpoint --by-week` flag that recomputes the midpoint over a rolling 7-day or 14-day window instead of full-lifetime, and reports the time-series of midpoint percentiles per source. That gives you the regime-change detector directly. I'll file that against pew-insights HEAD `3fffa44` if no one else does first.
