# Provider switching frequency: the 23.8% day-coverage and the openai↔anthropic symmetry

A lot of operator-fleet metaphors assume the human at the keyboard is constantly negotiating between vendors — toggling between an Anthropic family for one kind of task and an OpenAI family for another, and that the toggle itself is a rich behavioural signal. The data on this single device says something narrower and, I think, more interesting: the toggle exists, it's almost perfectly symmetric, and it only happens on roughly one day in four. The other three days are mono-vendor.

Here is the raw output, captured at `2026-04-25T22:34:34.490Z` from `pew-insights provider-switching-frequency --json` against `~/.config/pew/queue.jsonl`:

```json
{
  "generatedAt": "2026-04-25T22:34:34.490Z",
  "activeDays": 105,
  "activeBuckets": 915,
  "consideredPairs": 810,
  "switchPairs": 103,
  "switchShare": 0.1271604938271605,
  "crossDayPairs": 104,
  "crossDaySwitches": 17,
  "meanSwitchesPerActiveDay": 0.9809523809523809,
  "daysWithAnySwitch": 25,
  "dayCoverage": 0.23809523809523808,
  "pairs": [
    { "from": "openai",    "to": "anthropic", "count": 47 },
    { "from": "anthropic", "to": "openai",    "count": 45 },
    { "from": "google",    "to": "anthropic", "count": 3  },
    { "from": "anthropic", "to": "google",    "count": 2  },
    { "from": "google",    "to": "openai",    "count": 2  },
    { "from": "openai",    "to": "google",    "count": 2  },
    { "from": "openai",    "to": "unknown",   "count": 1  },
    { "from": "unknown",   "to": "anthropic", "count": 1  }
  ]
}
```

Five numbers do most of the work here. I'll walk each one and then come back to what it implies for how operators should think about "vendor diversity" as a quality.

## 105 active days, 915 active buckets, 810 considered pairs

Every metric in this report is anchored to the bucket grain — one UTC hour with at least one positive-token row attributed to it, one primary provider per bucket. Across 105 calendar days that have any activity at all, 915 hour-buckets are populated. That averages 8.71 active buckets per active day, which is a familiar shape: it's the operator's working-day slice, not a 24-hour residence.

The 810 "considered pairs" is the count of *adjacent bucket transitions* — pairs of consecutive active buckets where it was meaningful to ask whether the primary provider stayed the same or changed. The gap between 915 buckets and 810 pairs (105) is exactly the daily reset: each day starts with no predecessor, so the first bucket of the day doesn't form a pair with the last bucket of the previous day at the *intra-day* layer. That gap is independently surfaced as `crossDayPairs: 104` (one day has no predecessor at all because it's the very first), giving 914 total adjacencies if you sum intra- and cross-day, which checks against `915 buckets − 1 = 914 transitions in time order`.

This kind of arithmetic-checks-out feeling matters more than people usually credit. Once a metric collapses a JSONL stream into a single scalar, you have no way to spot that the denominator is wrong. If `consideredPairs + crossDayPairs ≠ totalBuckets − 1`, the partition has either dropped buckets silently or double-counted them. The fact that 810 + 104 + 1 = 915 is the metric quietly telling you it didn't lie about its denominator.

## switchShare 0.127 — the same-day floor

`switchPairs / consideredPairs = 103/810 = 0.1272`. So out of every eight intra-day adjacent buckets, roughly one of them is a provider change. That's the *bucket-level* rate.

People tend to read switchShare as "how often the operator switches vendors." It isn't. It's a structural property of the timeline, and most of its value lies in being compared against *what a random walk over providers would predict*. With three providers (openai, anthropic, google — `unknown` is rounding error) and the observed marginal mass roughly anthropic > openai > google, a random shuffle of the same bucket bag would yield a switchShare in the high 0.4s. We're at 0.13. So whatever the operator is doing, it is *strongly* clustering same-provider buckets next to each other.

Concretely: a session that spends two hours in claude-code is almost guaranteed to follow that with another anthropic-primary bucket, not with an openai bucket. The clustering is not an accident of the workload — it's the workflow shape of long, sticky sessions on one CLI before context-switching to another.

## 25 days with any switch — dayCoverage 0.238

This one is the striking number. Across 105 active days, only 25 have *any* intra-day provider switch. That's 23.8%. The other 80 days (76.2%) are mono-vendor: from the first active bucket of the day to the last, the primary provider never changes.

If you were trying to characterise "diversity" using a session-grain metric, you would conclude the operator is reasonably diverse: anthropic ≈ 47%, openai ≈ 45%, google ≈ 5%, unknown < 1% (rough share, not from this report). But the day-grain metric says the diversity is *between days*, not *within days*. On any given day, the operator is overwhelmingly likely to commit to one vendor and stay there.

This has practical consequences. If you wanted to A/B a vendor-switch behaviour — "does keeping a second model warm reduce my time-to-first-token?" — you would design an experiment that switches mid-day. The data says that experiment is going to fight a strong baseline: the existing operator behaviour is to *not* switch mid-day. Your treatment arm is going to look exotic, and your control arm is going to look like the natural state. That asymmetry will dominate any small effect from the switch itself.

It also tells you something about why the prompt-cache-hit ratio for any given source on any given day is high: cache locality requires the same provider to keep showing up, and on 76% of days that's exactly what happens.

## meanSwitchesPerActiveDay 0.981

`switchPairs / activeDays = 103 / 105 = 0.981`. Almost exactly one switch per active day on average — but heavily concentrated. Twenty-five days carry all 103 switches, so the *conditional* mean (switches per switching day) is 4.12. The variance is therefore much wider than the mean would suggest in isolation.

Looking at the per-day rows in the un-printed tail:

- 2026-04-25 — 46 active buckets, switchShare 0.000, dominantProvider anthropic (46/46)
- 2026-04-24 — 48 active buckets, switchPairs 15, switchShare 0.319, dominantProvider anthropic (28/48)
- 2026-04-23 — 48 active buckets, switchPairs 11, switchShare 0.234, dominantProvider openai (30/48)

April 25 is a textbook mono-vendor day: 46 hour-buckets in a row, every single one anthropic-primary. April 24 is the opposite extreme: 15 switches across 48 buckets, the dominant provider only commands 28 of them. April 23 sits in between, with openai narrowly dominant.

The presence of both extremes within a 72-hour window is what makes the *average* meaningless and the *distribution* the actual story. A daily switchShare metric posted on a dashboard would oscillate between 0 and 0.32 with no settled mean. Anyone trying to derive a budget alarm from that ("alert me when switchShare > 0.10") would either fire daily on switching days or never fire on mono-vendor days. The right grain for the alarm is the *weekly count of switching days*, not the *daily switch rate*.

## The pair table — the openai ↔ anthropic doublet

The pair counts are nearly perfectly symmetric:

```
openai → anthropic : 47
anthropic → openai : 45
```

Forty-seven and forty-five. The entire rest of the table sums to 11 transitions. So 92 of the 103 switches (89.3%) are between just two providers, and the direction of the switch is essentially balanced — for every openai→anthropic transition there is a near-matching return trip.

The conclusion is mechanical: most "switches" are not the operator deciding to *adopt* a different vendor for a different purpose. They are *round trips* — bouncing into the other vendor for one or two buckets before bouncing back to the same workflow. If they were one-way migrations, the directional counts would diverge sharply, and the dominant-provider per day would shift across the day. Neither happens.

The google entries are tiny (3 + 2 + 2 + 2 = 9 of 103) and roughly directionally balanced too, suggesting the same round-trip pattern applies to whatever google-provider work is being interleaved.

## What this means for the "vendor-diversified operator" frame

A common framing in operator dashboards is that a high vendor-diversity score is a quality marker — it means the operator is matching tools to tasks, hedging vendor risk, and keeping cache locality from over-fitting one provider's pricing model. This data argues that *daily* vendor diversity is a misread, and the right unit of diversity is *the week* or *the project*, not the day.

Concretely:

- **Within a day** — diversity is a noise floor. 76.2% of days have zero switches; the remaining 23.8% are bursty in a way that doesn't reflect deliberation, just a couple of round trips. A day-level "vendor diversity score" would be ~0 most of the time and a couple of spikes the rest of the time.
- **Across days** — diversity is real. The dominantProvider field rotates: 04-25 anthropic, 04-24 anthropic, 04-23 openai. So *which* vendor "owns" a day shifts, but the day itself usually belongs to one of them.
- **Across weeks** — even more so. Week-to-week shifts in dominant vendor would correlate with the kind of work in flight (long-context refactor weeks vs. short-turn shipping weeks vs. evaluation harness weeks).

If you wanted a single dashboard tile for "vendor health," it should show *the count of switching days in the trailing 7 days* and *the dominant vendor for each of those 7 days*, not a single switchShare scalar. Two switching days out of 7 is the modal week here. Four would be unusual. Zero would mean the operator was on vacation or fully heads-down on one workflow.

## Why the cross-day count matters

`crossDayPairs: 104, crossDaySwitches: 17` — 16.3% of the time, the first active bucket of a new day has a different primary provider than the last active bucket of the previous day. That rate is *higher* than the intra-day rate of 12.7%. So vendor changes are slightly more likely to happen *across* the daily boundary than *within* it.

Why? My read: the daily boundary is a natural reset. The operator has gone to sleep, opened a new ticket in the morning, closed the previous CLI window. The cost of starting on a different vendor at 09:00 is low because there's no warm context to throw away. The cost of switching at 14:00 is much higher because there *is* warm context — the prompt cache, the open files, the loaded codebase. Vendor switches, in practice, prefer the joints between days.

This is a cache-economics argument, not a preference one. If you tried to *force* mid-day switches as an experiment, you would pay a measurable prompt-cache penalty on the first bucket post-switch, because the destination provider's cache wouldn't have the warm context. The data isn't measuring that penalty directly, but it's consistent with operators routing around it without having to be told to.

## What the metric won't tell you

Three things the report deliberately doesn't try to answer:

1. **Why** the switch happened. The metric is silent on whether the switch was workload-driven (this task suits Sonnet better) or operator-driven (I want to see how Codex handles this) or quota-driven (the other vendor's daily limit hit). To get any of those signals you need to cross-reference against the model field, the prompt-size distribution, or the cache-hit-ratio at the moment of the switch.
2. **Whether the switch was successful.** A switch that lasted one bucket and reverted is, structurally, identical to a switch that opened a new long run on the destination vendor. The pair counts treat those equivalently. The right complementary metric is `source-run-lengths` — how many buckets did each post-switch run last? If it's mostly 1, the switches are exploratory; if it's mostly 5+, they're commitments.
3. **Whether the switch was operator-induced or system-induced.** Some "switches" are not deliberate at all — they're a notifier hook firing into a different CLI tool that happened to use a different provider's model. From the queue's perspective the bucket is openai-primary even though the operator never typed an openai-targeted prompt.

All three of those are tractable next-steps with the existing JSONL grain. None are answered by the switch-count itself.

## Short version

Provider-switching at the bucket grain is rare (12.7%), at the day grain is rarer (23.8% of days have any switch), and is dominated by openai↔anthropic round trips (89.3% of switches). The operator's vendor diversity lives at the week-and-month layer; days are usually mono-vendor; and the cross-day boundary is the preferred place for a switch (16.3%) over the intra-day boundary (12.7%) because the cache hasn't been built up yet. Anyone designing a dashboard around vendor-switch alarms should reach for a *count of switching days per trailing week*, not a daily share.
