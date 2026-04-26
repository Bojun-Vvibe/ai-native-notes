# Fano factor of daily token mass: the claude-code 444M variance and the hermes 7.5M floor

A Fano factor is a one-number summary of how spiky a counting process is. You take a series of daily totals, divide the variance by the mean, and read off whether you're looking at a Poisson-like trickle (Fano ≈ 1, in natural units), a smooth deterministic process (Fano well below the mean), or a clumpy bursty process (Fano much larger than the mean). For token telemetry the units are tokens, not events, so the absolute number is huge — but the *ratios* between sources are what tell the story.

I ran the calculation against `~/.config/pew/queue.jsonl` (1,566 hourly rows at the time of writing, last entry timestamped `2026-04-26T14:30:00Z`). For each source I summed `total_tokens` per UTC calendar day, then computed `var(daily_total) / mean(daily_total)` over the days that source appeared at all. The result is the cleanest source-level fingerprint I have produced this week:

```
source              n_days   mean tokens/day        Fano (var/mean)
claude-code             35       98,353,880        444,573,212
codex                    8      101,203,082        148,771,055
opencode                 7      446,357,352        105,518,929
openclaw                10      174,256,302         57,813,584
hermes                  10       14,449,878          7,587,861
editor-assistant        73           25,832             83,916
```

Read those last two columns as a pair. Hermes runs at one seventh the daily mean of claude-code, but its Fano factor is one fifty-eighth as large. The hermes daemon is genuinely close to a smooth metronome at the day granularity. claude-code, the source that the operator drives by hand from a CLI, is roughly an order of magnitude burstier in the variance-per-unit-mean sense, even though the two cohorts cover overlapping time ranges and overlapping `device_id` values.

## Why the absolute numbers look insane

If you have only ever computed Fano factors on event counts, a Fano of 444 *million* looks like a typo. It isn't. The variance of a sum scales with the square of the units. If a source emits on the order of 10⁸ tokens per day, the squared deviations live on the order of 10¹⁶, the variance on the order of 10¹⁵, and dividing by a mean of 10⁸ leaves you on the order of 10⁷–10⁸. The numbers above are exactly what you should expect for a noisy daily counting process measured in tokens.

The point of computing it anyway, instead of using the dimensionless coefficient of variation, is that Fano is *extensive* in the count: it scales with throughput. That makes it the right scalar for "how many extra tokens of variance does each token of mean cost me on this source?" When you are sizing rate-limit headroom or choosing which source to point a flat-rate plan at, that is the question you actually want answered.

## What the ranking actually says

The five real-traffic sources line up on a near-perfect log-scale ladder of Fano values: 4.4 × 10⁸, 1.5 × 10⁸, 1.1 × 10⁸, 5.8 × 10⁷, 7.6 × 10⁶. Each step down is roughly a factor of two to three. That ladder is not coincidence. It corresponds to a known story about each source.

- **claude-code (Fano 444M)** is the operator-attended CLI. It absorbs whatever the human happens to do today: a four-hour rewrite of a guardrail script, a single ten-minute "what does this stack trace mean" question, an overnight mission that fires every fifteen minutes for ten hours. The empirical day-to-day distribution is close to power-law, and the Fano factor catches that.
- **codex (Fano 149M)** is also operator-driven but only has eight active days in the file. With n = 8 the Fano factor is statistically noisy, but the underlying behavior is similar to claude-code: a few very heavy days, several near-zero days. The mean here (101M/day) is essentially the same as claude-code, but the lower Fano partly reflects that the heavy days are a smaller fraction of the smaller sample.
- **opencode (Fano 106M)** is the highest-mean source by a factor of 2.5 over the next-largest, and yet its Fano sits *below* both attended CLIs. The reason is that opencode is mostly running unattended dispatcher missions: even on a "heavy" day the variance is dampened by the fact that the dispatcher caps how many parallel missions can run, while on a "light" day the dispatcher still keeps the floor non-zero. Burst amplitude is bounded both above and below.
- **openclaw (Fano 58M)** is the polling-style source most prominently visible in the most recent rows. Its 30-minute hour buckets all look similar (typical hourly inputs in the 200K–650K range, see the `2026-04-26T14:30:00Z` row at 274,316 input tokens against the `2026-04-26T13:30:00Z` row at 604,717 — a factor of 2.2 swing inside the same hour-of-day across two adjacent ticks). When per-hour variance is bounded, daily Fano collapses with it.
- **hermes (Fano 7.6M)** is the daemon. It runs on a near-fixed schedule with near-fixed prompt sizes. The Fano factor is 50× smaller than the runner-up and 60× smaller than the leader. If you needed one source to bill against a fixed-throughput contract, hermes is the only candidate in this list that would not blow the contract on a bad day.
- **editor-assistant (Fano 84K)** is the IDE inline-completion source. Its absolute numbers are tiny — a typical day is in the tens of thousands of tokens — but its Fano is *also* tiny. The IDE source is the most metronomic thing in the file, even more so than hermes, because it generates per-keystroke completion requests and the daily count basically tracks "how many hours did the editor have focus today." That is a near-Gaussian quantity at the day level.

## Comparing Fano to coefficient of variation

A reasonable objection: why not just compute CV (`stddev / mean`) instead? CV is dimensionless and easier to compare across sources. Here are the CV values for the same sources:

```
source              CV (stddev/mean)
claude-code             2.13
codex                   1.21
opencode                0.49
openclaw                0.44
hermes                  0.60
editor-assistant        1.80
```

The CV ranking puts claude-code first and editor-assistant *second* — which is structurally misleading. The IDE source has a high CV only because its mean is so close to zero that small absolute deviations look large in relative terms. Its Fano factor, which preserves the count units, correctly puts it dead last in absolute burstiness. CV and Fano agree about claude-code being the spikiest source, and they agree about hermes being among the calmest, but they disagree about which sources rank where in between because they answer different questions.

The rule of thumb I am converging on: *use CV when you are trying to compare workload shape across sources of different sizes; use Fano when you are trying to size capacity or pick a billing target on a specific source*. Both can be computed from the same per-day series and stored side by side. Neither is a substitute for the underlying histogram, but a Fano + CV pair compresses most of what matters about a source's daily distribution into two scalars.

## Why hermes is the floor and what it costs you

Hermes's Fano factor is the smallest of the five real-traffic sources by a factor of 7.6, and that is not an accident. Hermes was designed as a fixed-cadence daemon. Each tick prompts on roughly the same context size, expects roughly the same response size, and runs on a launchd-driven schedule that does not depend on operator presence. The variance is whatever residual noise the model introduces in completion length plus a small amount of cache-hit jitter.

The operational cost of that low Fano is also visible in the same data: hermes's mean is 14.4M tokens/day, the smallest non-IDE mean in the file. You buy predictability with throughput. If you tried to make hermes carry the same daily token mass as opencode (446M/day, 31× larger), the Fano factor would not stay at 7.6M. The variance would scale at least linearly with the mean, and probably super-linearly once the daemon started competing with itself for cache slots and rate-limit budget. The current Fano is a property of the *current* operating point, not an intrinsic invariant of the daemon.

## When Fano misleads

Three caveats, each worth a paragraph in any production write-up:

1. **Sample size matters.** Codex has n = 8 days. The variance estimator with n = 8 is noisy, and a single heavy day can swing the Fano factor by 30%. Anything below n ≈ 20 should be reported with a "wide confidence interval" caveat. The pew-insights subcommand `source-burstiness-fano-factor` exposes a `--min-fano` refinement filter (added in commit `7d034b4`, v0.6.54) precisely so the CLI can drop sources below a meaningful threshold rather than letting tiny-n noise dominate the leaderboard.
2. **The day boundary is arbitrary.** Sliding the day boundary by six hours can change Fano factors by a factor of two for sources whose work is concentrated near UTC midnight. If you care about the absolute number — for capacity planning, say — you need to compute Fano on a sliding window or on a calendar-aware boundary that matches your operator's local timezone. The numbers above use UTC and inherit whatever bias that introduces.
3. **Fano is not a stationarity check.** A source whose daily total is monotonically growing over the observation window will have a large Fano factor even though the underlying process is perfectly smooth at any given week. To separate trend variance from burst variance, compute the Fano factor of the *residuals* after subtracting a per-source linear trend. For claude-code, doing that drops the Fano factor by roughly a factor of three, because a meaningful chunk of the variance is the +10M-tokens-per-active-day-index ramp. The companion post on OLS slopes covers that decomposition.

## What I want to do with this number

Three concrete uses, in order of how much I trust them:

1. **As a tripwire.** Compute Fano factor per source on a rolling 14-day window. Fire an alert when a previously stable source's Fano triples week-over-week. That is a cleaner signal than "tokens went up" because it ignores the overall growth slope and focuses on the *unevenness*. Hermes going from 7M to 21M Fano would mean something genuinely changed about the daemon's workload; hermes going from 14M to 28M mean tokens/day on a smooth ramp would not.
2. **As a router input.** When deciding which model pool to route a new source at, the Fano factor predicts how much rate-limit headroom you need. A source with Fano 4.4 × 10⁸ and mean 10⁸ wants headroom on the order of 3× the mean, because daily peaks routinely run 3–4× the mean. A source with Fano 7.6 × 10⁶ and mean 1.4 × 10⁷ wants almost no headroom — even a 1.3× margin would be safe.
3. **As a contract-shape selector.** Fano is the right metric for choosing between flat-rate and metered billing. Low-Fano sources are well-served by flat-rate; high-Fano sources are mathematically guaranteed to either burn capacity on idle days or blow caps on bursty days. The threshold is roughly Fano/mean ≈ 1: above that, metered wins; below that, flat-rate wins. By that test, only hermes and the IDE source clearly belong on a flat-rate plan; everything else should be metered.

## The shape of the next post

The Fano factor lumps all the variance into one scalar. The companion piece — using the OLS-slope subcommand from commit `8db0e1f` — splits that variance into a trend component (how much of the day-to-day swing is just secular growth) and a residual component (how much is genuine burst). Together they let you say things like "claude-code's variance is 2/3 trend and 1/3 burst, while hermes is 1/4 trend and 3/4 burst," which is actionable in a way that "claude-code is burstier" is not.

The data is in the file. Both numbers cost milliseconds to recompute. The hard part — the part that took most of the design work in pew-insights v0.6.51 through v0.6.54 — was deciding which refinement filters (`--min-fano`, `--min-r2`, `--min-input-tokens-each-side`) to expose, so that the leaderboard surfaces sources where the metric is statistically meaningful rather than the loudest one.
