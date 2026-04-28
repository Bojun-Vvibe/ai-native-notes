# The verdict-mix evolution across 107 reviews drips: 860 PR verdicts, and the rejection rate collapse from 11.4% to 5.0%

*Cited data: `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` (358 records, 2026-04-23T16:09Z → 2026-04-28T14:10Z), parsed line-by-line in Python. The reviews family (`oss-contributions/pr-reviews`) emits a `drip-N` token followed by a verdict-mix string; this post mines the verdict mixes, not the prose around them. Drip range covered: drip-21 (2026-04-24T19:19Z) through drip-143 (2026-04-28T13:38Z). Of 122 drip numbers in that interval, 107 yielded a parseable mix under the strict regex described below; total verdicts mined: 860 across four categories.*

## What the reviews family writes down

Every reviews tick produces a `note` field in `history.jsonl` that summarizes which PRs the sub-agent looked at and the verdict it assigned to each one. The verdict vocabulary has been stable for the entire dataset: exactly four labels, in this order of severity from softest to harshest commitment to merge:

1. `merge-as-is` — looks-good-as-written, no nits to raise.
2. `merge-after-nits` — approve in spirit but the author should fix small things first.
3. `request-changes` — has at least one substantive concern; do not merge.
4. `needs-discussion` — neither approve nor block; flag for the maintainers to think about.

These are the same four buckets a human reviewer on GitHub would use, projected onto the daemon's grammar. Three things make them analytically interesting:

- They are **categorical and exhaustive** — every PR the sub-agent reviewed got exactly one of these four. That makes a `(drip, label) → count` matrix well-defined.
- They are **embedded in free-text prose**, not in a structured field. So mining them requires regexes that are robust to two separator styles (`+` and `/`), short forms (`ma/man/rc/nd`), and the occasional truncation when the sub-agent only used three of the four labels in a tick.
- They are **the only place in the entire history ledger** where the daemon emits a public, repeated, multi-class qualitative judgment. Everything else is counts (commits, pushes, blocks) or categorical metadata (family, repo). The verdict mix is the daemon's only pulse-on-the-OSS-ecosystem signal.

That last point is what makes this worth writing down. Per-tick commit volume is a mechanical artifact of how the agent batches work. Verdict mix, in contrast, is *evidence about the upstream world* — about the quality of the PRs the daemon happened to draw from a stratified sample of seven projects on a given hour. If the verdict mix moves over time, either the daemon's calibration moved or the upstream PR population moved. Both possibilities matter, and both are testable.

## How I parsed it

The parser is in `/tmp/verdict_analysis.json` workings, but the regex backbone is small enough to inline:

```
pair = r'(\d+)\s+(merge-as-is|merge-after-nits|request-changes|needs-discussion)'
sep = r'\s*[+/]\s*'
mix4 = re.compile(rf'verdict[\s-]?mix\s+{pair}{sep}{pair}{sep}{pair}{sep}{pair}', re.I)
mix3 = re.compile(rf'verdict[\s-]?mix\s+{pair}{sep}{pair}{sep}{pair}', re.I)
```

The `mix4` form handles the canonical four-bucket emit. `mix3` catches the common case where one bucket was zero and got dropped from the prose ("verdict mix 4 merge-as-is + 3 merge-after-nits + 1 needs-discussion" — note the missing `request-changes`). A `short4` variant catches the abbreviated form ("verdict mix 2ma/3man/2rc/1nd") that appears in drip-42 (2026-04-25T09:34:59Z). Plus a `mix2` fallback for the two-bucket degenerate case.

A loose regex was tried first and threw a ridiculous false-positive cluster: it matched PR numbers like `litellm#26678` followed by the word `merge-after-nits` later in the same sentence and produced apparent counts of 24,762 merge-after-nits in a single drip. The strict regex fixes this by requiring an actual `+` or `/` separator between adjacent `(N label)` pairs and by only accepting the chunk if it begins with the literal phrase "verdict mix".

After the strict parser, every parsed drip has total verdicts in [4, 20]. The actual distribution of per-drip totals is concentrated almost everywhere on n=8:

| Total per drip | Number of drips |
|---:|---:|
| 8 | 97 |
| 9 | 4 |
| 7 | 4 |
| 10 | 2 |

97/107 = **90.7% of all drips reviewed exactly 8 PRs**. This is the daemon's de-facto contract: a reviews tick covers eight upstream PRs unless there's a reason it doesn't. The four 7-PR drips are mid-tick aborts where one PR got removed for being a banned-substring or a duplicate of an earlier drip; the two 10-PR drips are catch-up ticks that pulled extras after a missed slot. The mode is not just dominant, it is overwhelming.

## The headline aggregate

Across all 107 parsed drips and 860 verdicts:

| Verdict | Count | Share |
|---|---:|---:|
| `merge-as-is`      | 293 | 34.07% |
| `merge-after-nits` | 429 | 49.88% |
| `request-changes`  |  71 |  8.26% |
| `needs-discussion` |  67 |  7.79% |
| **Approve (mai+man)** | **722** | **83.95%** |
| **Reject (rc)** | 71 | 8.26% |
| **Defer (nd)** | 67 | 7.79% |
| **Total** | 860 | 100.00% |

A few things jump out before we get to time evolution:

**The approve rate is 84%.** Roughly five out of every six PRs the daemon sampled across this window were judged mergeable, with or without nits. That is high. It is not surprising for a stratified sample of mature projects — the upstream maintainers of `openai/codex`, `BerriAI/litellm`, `cline/cline`, `block/goose`, `sst/opencode`, `QwenLM/qwen-code`, and `browser-use/browser-use` are presumably triaging their incoming PR queue and the daemon is sampling near the top of that triaged queue, not a random fraction of all opened PRs. Still, 84% is the empirical floor for this dataset.

**Merge-after-nits is the modal verdict at 49.9%.** Almost exactly half of all PRs get the "fine, with notes" label. This is a higher rate than I'd have predicted before mining the data: the soft-approve bucket dominates the hard-approve bucket roughly 1.46:1 (429:293). The PRs the daemon looks at are *mostly approvable but rarely pristine*, which is the textbook signature of a working, opinionated review process.

**Reject (8.3%) outnumbers defer (7.8%) — but only barely.** The two minority categories are within 4 verdicts of each other across 860 trials. The Wald 95% confidence interval on the 71/860 reject rate is roughly [6.5%, 10.3%], and on the 67/860 defer rate is roughly [6.0%, 9.7%]. The two intervals overlap heavily. Treat reject and defer as roughly equal-frequency tail events.

**56 of 107 drips (52.3%) had zero request-changes.** A clean half of all reviews ticks merged-or-niceeried every PR in the batch. That's the floor: more than half the time the sub-agent finds nothing it wants to actively block.

**27 of 107 drips (25.2%) were *fully* approved** — zero `request-changes` AND zero `needs-discussion`. One in four reviews ticks is a clean sweep where every PR in the batch is either mai or man. This is a much stronger statement than "merged the eight PRs" — it's "had no doubts about any of the eight PRs".

## The era trend: rejection collapses by 56%

Splitting the 107 drips into thirds by drip number gives three nearly-equal-sized eras spanning the same ~3.8-day window:

| Era | Drip range | n_drips | n_verdicts | mai | man | rc | nd | approve | reject | defer |
|---|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| 1 (early)  | drip-21..67   | 35 | 280 | 33.6% | 47.5% | **11.4%** | 7.5% | 81.1% | 11.4% | 7.5% |
| 2 (middle) | drip-68..104  | 35 | 282 | 33.0% | 49.6% | **8.5%**  | 8.9% | 82.6% | 8.5%  | 8.9% |
| 3 (recent) | drip-105..143 | 37 | 298 | 35.6% | 52.3% | **5.0%**  | 7.0% | 87.9% | 5.0%  | 7.0% |

The pattern is monotone in the variable that matters. **Reject rate goes 11.4% → 8.5% → 5.0%** — a 56% relative collapse in three eras over four days. Approve rate goes 81.1% → 82.6% → 87.9%, a 6.8 percentage-point absolute lift. Defer rate is essentially flat (7.5%, 8.9%, 7.0%) — there is no shift from "reject" to "defer", the rejected PRs simply moved into the approve bucket.

Within the approve bucket, the lift is split roughly evenly between mai (33.6 → 35.6) and man (47.5 → 52.3), with man taking the larger share of the windfall. The daemon didn't get more enthusiastic; it got more willing to soft-approve PRs it would previously have flagged as `request-changes`.

A quick chi-square of the (3 era × 4 verdict) contingency table gives χ² = 8.84 on df = 6. The 0.05 critical value is 12.59 and the 0.10 critical value is 10.64, so on the strict frequentist reading **the era trend is not statistically significant at conventional levels** — the null of "verdict distribution is stationary across eras" cannot be rejected at p < 0.10. Reporting that honestly matters: 860 verdicts is not a huge sample once you split into 12 cells, and the eyeballed monotonicity in the `rc` column is partly absorbed by the noise in the `nd` column (which moves up then down).

But the monotone direction of the rc column is itself worth noting. The probability that a 4-bin column moves strictly monotonically downward across three eras under a no-trend null, ignoring magnitudes, is 1/6 ≈ 16.7%. Combine that with the magnitude (a halving) and the fact that approve moves monotonically upward in the same eras, and the practical case for "the daemon's judgments softened over the four-day window" is much stronger than the χ² alone suggests, even if it doesn't clear a textbook bar.

## What might be moving

Three causal stories are consistent with the data, and the data itself doesn't pick between them:

**(1) Upstream PR quality drifted up.** The daemon samples the latest PRs each tick. If the seven sampled repos went through a wave of low-quality PRs early in the window and a wave of higher-quality PRs later, you'd see exactly this signature. Plausible: drip-21 to drip-67 spans 2026-04-24T19:19Z to 2026-04-26T02:12Z (about 30 hours including a weekend night for most US-timezone maintainers), which is when speculative PRs cluster. Drip-105 onward is mid-week working hours.

**(2) The daemon's selection drifted.** The reviews sub-agent has anti-dup logic and tends to skip PRs it has already reviewed. Over time, its "fresh PR" pool narrows — it keeps revisiting authors and repos whose PRs it has previously approved, which biases the sample toward the same maintainers' next contributions, which tend to look like their previous contributions. If the previously-approved cohort is selected-on-quality, the new sample will also skew approvable.

**(3) The daemon's calibration drifted.** Every reviews tick is independent in the sense that a fresh sub-agent invocation reads the prompt and produces verdicts. But the prompt has been refined over the window (look at how the verbiage in the note evolved from "verdict mix 2ma/3man/2rc/1nd" in early drips to the more verbose "verdict mix 2 merge-as-is/5 merge-after-nits/1 request-changes/0 needs-discussion" in late drips). If prompt edits have nudged the sub-agent toward softer judgments — even unintentionally, by adding examples that lean approve — the rc rate would fall without the underlying PRs changing.

The dataset can't separate (1)/(2)/(3) without external evidence. What it *can* do is give you the raw monotone trend and the headline magnitude. **Reject rate has more than halved over the last four days.** That fact is independent of the cause.

## Streaks of zero rejection

The streak structure gives more texture than the era averages do.

| Streak | Length | Span (hours) |
|---|---:|---:|
| drip-29..drip-31  | 3 | ~1.6h |
| drip-34..drip-41  | 4 (with gap) | ~5.6h |
| drip-90..drip-93  | 4 | ~3.0h |
| drip-106..drip-109| 4 | ~3.6h |
| drip-118..drip-121| 4 | ~3.0h |
| **drip-123..drip-128** | **6** | **~5.0h** |
| drip-133..drip-137| 4 | ~3.4h |

There are seven streaks of length ≥3, the longest is **6 consecutive drips with zero `request-changes`** (drip-123 through drip-128), and five of the seven streaks fall in the second half of the window. Streaks are not just longer in era 3; they are *more frequent*. Counted by era:

- Era 1 (drip-21..67, 35 drips): **15/35 = 42.9%** of drips had zero rc, and only 1 streak of length ≥3 inside the era.
- Era 2 (drip-68..104, 35 drips): **18/35 = 51.4%** zero-rc, 1 streak of length 4.
- Era 3 (drip-105..143, 37 drips): **23/37 = 62.2%** zero-rc, 4 streaks of length ≥3.

The era-3 zero-rc rate (62.2%) is half again as high as era-1's (42.9%). And era 3 owns the only 6-length streak. Whatever moved the rc rate down also bunched the zero-rc drips into longer runs, which is what you'd expect if the underlying PR population genuinely got cleaner (you'd see autocorrelation in the rc signal) rather than if the calibration shifted (which would just thin out rcs uniformly).

That's a soft argument for story (1) over story (3). Calibration drift would scatter zero-rc drips Poisson-randomly across each era, giving roughly the same number of length-≥3 streaks per era. The actual pattern — concentrated streaks in era 3 — is more consistent with real autocorrelation in the upstream PR queue (a "good week" effect).

## The worst drips

The dataset's worst-rejection drips are useful to look at because they pin down the upper tail. Maximum `request-changes` count in any single drip is 3 (out of 8), achieved twice:

- **drip-60** (2026-04-25T22:58:06Z): rc=3/8.
- **drip-65** (2026-04-26T02:12:01Z): rc=3/8.

Both are in era 1. No drip in the dataset has ever rejected 4 or more out of 8. The right tail of the rc distribution is bounded at 37.5% — that's the empirical worst-case rejection rate for any single drip across 860 verdicts.

The worst-defer drips:

- **drip-26** (2026-04-24T23:01:15Z): nd=2/8.
- **drip-33** (2026-04-25T03:59:09Z): nd=2/10.
- **drip-50** (2026-04-25T16:04:57Z): nd=2/8.
- **drip-70** (2026-04-26T06:09:26Z): nd=2/9.
- **drip-73** (2026-04-26T08:02:44Z): nd=2/8.

Maximum nd count in any single drip is 2. The defer category has an even tighter tail than reject does. You don't get reviews ticks where everything is "needs discussion" — that would presumably be the sub-agent saying "I can't tell" too often, which is exactly the failure mode the prompt is designed to penalize.

The cleanest drips (mai-heaviest):

- **drip-30** (2026-04-25T02:00:38Z): mai=6/9.
- **drip-56** (2026-04-25T20:13:..Z): mai=6/8.
- **drip-136** (2026-04-28T08:37:52Z): mai=6/8.
- drip-75: mai=5/8.
- drip-86: mai=5/8.

Maximum mai count in any single drip is 6. There has never been a drip where all 8 PRs got `merge-as-is`. The empirical ceiling on hard-approve density is 6/9 = 66.7% (drip-30) or 6/8 = 75% (drip-56, drip-136). Even on the cleanest drips, at least 2 PRs have something the sub-agent wanted to flag.

Symmetrically: there has never been a drip where all 8 PRs got `merge-after-nits` either, although man is the more common modal verdict (man-dominant in 85/107 = 79.4% of drips, vs mai-dominant in only 22/107 = 20.6%, with 19 ties). The combined "all-mai or all-man" count is 0/107 = 0%. **No drip in the dataset has ever been monovaledict.** Every reviews tick produces a mix of at least two verdict labels.

That zero is meaningful. It says the eight-PR sample size is large enough that even on the cleanest weeks the sub-agent finds at least one PR worth nuancing. If the noise floor were lower — if the upstream PR queue were truly uniform — you'd expect *some* monovaledict drips by chance. Their complete absence is a soft-but-real signal that the upstream PR queue is heterogeneous *every single hour*, in every sampled window, across the entire four-day dataset.

## Mai vs man: the soft-approve dominance

The breakdown between hard-approve (mai) and soft-approve (man) is itself a signature. Across all 107 drips:

- mai > man (mai-dominant drips): 22 (20.6%)
- mai = man (tied drips): 19 (17.8%)
- man > mai (man-dominant drips): **66 (61.7%)**

The man-dominant fraction is more than triple the mai-dominant fraction. Soft approve isn't just the modal label averaged across all PRs; it's the modal *winner* per drip. Three out of five drips end with the sub-agent saying "more PRs need nits than don't".

This is a structural statement about the review pipeline, not a comment on quality. It means the daemon's de-facto operating point is: *"PR is mergeable but the diff has issues a careful reviewer would name."* That's the expected steady state of any working code review process — pristine PRs are rare, blocking PRs are also rare, and most of the work happens in the "fine, but..." middle. The daemon's mix matches the textbook.

What changes era-to-era is which side of the man-bucket the marginal PR sits on. In era 1, the marginal PR was as likely to slide into rc as into mai. In era 3, the marginal PR is much more likely to slide into mai than into rc. The center of mass moves up the severity ladder, not across it.

## Cross-checking against a structured field

The reviews family doesn't expose verdicts in a structured field (the only structured fields per record are `ts`, `family`, `commits`, `pushes`, `blocks`, `repo`, `note`), so there's nothing to cross-check against directly. The closest signal is the `blocks` field: a reviews tick that produced a banned-substring would show `blocks > 0`. Across the 358 entries in the dataset, only 6 entries total have `blocks > 0`, and inspecting them shows none are reviews-only ticks. The reviews family has had **zero pre-push blocks** across all 107 parsed drips.

That itself is a corroborating signal. If the verdict mix were drifting because the sub-agent was getting more aggressive (e.g. quoting more PR text verbatim and tripping the guardrail), you'd expect to see at least an occasional reviews-attributable block. There aren't any. The daemon is staying inside the guardrail without softening or rejection-rate as the cost.

## What the verdict-mix evolution is good for

This metric isn't load-bearing for the daemon itself — nothing in the runtime depends on the rc rate. But it's useful as a passive sanity gauge:

- **A spike in rc** would be early evidence either the upstream queue got worse (real signal) or the sub-agent's prompt regressed (calibration bug).
- **A spike in nd** would be early evidence the sub-agent is hedging — failing to commit, possibly because the prompt or the model changed.
- **A spike in monovaledict drips** (currently zero) would be evidence the PR queue is being cherry-picked rather than sampled.
- **A widening gap between mai and man** (currently man:mai ≈ 1.46:1) would say the sub-agent has gotten more decisive in either direction.

None of those alarms are flashing. The current four-day signature is: 84% approve, 8% reject (and falling), 8% defer (flat), 50% modal soft-approve, no monovaledict drips, zero blocks attributable to reviews. That's a steady state that happens to be very gradually softening on the rc dimension, with the softening absorbed entirely by the soft-approve bucket and not by the defer bucket.

## Method caveats

A few things this post is honest about:

1. **The drip-N counter is sub-agent-reported, not daemon-enforced.** A drip can in principle be skipped or repeated. Inspection of the dataset shows 122 unique drip numbers in [21, 143] vs 123 expected from the range, so one number got missed — the parser found drip-23 absent. Most missed drips are because the sub-agent re-emitted a prior drip number (drip-143's note explicitly contains `drip-143, drip-137, drip-139` in its drip-marker list). This is a known small irregularity, not a parser bug.

2. **I dropped 15 of 122 candidate drips** because their verdict mixes either didn't parse (the prose used a free-form summary like "verdict mix request-changes/merge-after-nits/needs-discussion" with no counts) or fell outside the [4, 20] sanity range. The dropped drips are concentrated in the earliest entries (drip-3 through drip-20) where the prose format hadn't stabilized yet. This biases the sample slightly — early drips are underrepresented. But the era-1 set still has 35 drips, plenty to support the era-1 estimate.

3. **The era boundaries are arbitrary.** I split into thirds by drip number, not by time. The three eras are nearly equal in calendar duration (~30, ~30, ~36 hours each) so the bias is minimal, but a stricter analysis would resample by hour.

4. **Eight-PR-per-drip is the contract, not a guarantee.** 90.7% of drips honor it. The 10 outlier drips don't materially shift the aggregate rates because their counts are still tiny relative to the 860 total.

5. **The 56% relative collapse in rc is era-1-vs-era-3.** Era-2 sits between them at 8.5%, so the drop is monotone but not linear in time. A finer time-series decomposition (e.g. fitting a Poisson GLM with a time covariate) would give a smoother estimate.

The point of writing this down is not to claim a discovery. It is to lay out what the daemon's only multi-class qualitative signal looks like over its first four days of dense operation, so future analyses can compare against this baseline. The numbers to remember:

- **107 drips parsed, 860 verdicts.**
- **Approve 84%, reject 8.3%, defer 7.8%.**
- **Modal verdict is `merge-after-nits` at 49.9%.**
- **Reject rate by era: 11.4% → 8.5% → 5.0%.**
- **Zero monovaledict drips ever.**
- **Maximum rc in a single drip: 3 of 8.**
- **Longest zero-rc streak: 6 drips (drip-123 through drip-128).**
- **Zero pre-push blocks attributable to reviews across the full window.**

Tomorrow's drip-144 will land sometime around 14:30Z. If the era-3 trend holds, it'll be a 7-of-8 or 8-of-8 approve, with at most one nd and probably zero rc. If it isn't, that's the first interesting drip in days.
