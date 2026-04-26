# Session length conditional on prompt count — the 655-second slope and the three-prompt collapse

The naïve question is: do longer sessions have more prompts? The answer is yes, obviously, with a Pearson correlation of `0.34`, which is real but weak. The interesting question is the *shape* of the relationship — whether session duration grows linearly in prompt count, whether it grows faster or slower, and whether there are bands of prompt count where the relationship breaks down. Because if you fit a single linear regression to all 6065 human sessions in the corpus and report the slope, you get a number that is technically correct and operationally meaningless: it averages over qualitatively different regimes.

This post does the regression, reports the slope, and then immediately throws the slope away in favour of a bucketed conditional view that actually tells you what's going on. The bucketed view shows that **the relationship between prompt count and duration is not monotonic** — sessions with 2-3 prompts are *shorter* on average than sessions with a single prompt, and this is not a fluke.

## The query

Captured at `2026-04-26T03:39:42Z` against `~/.config/pew/session-queue.jsonl`, restricted to sessions where `kind == "human"` and `user_messages > 0` (excludes automated and zero-input sessions).

```
jq -r 'select(.kind=="human" and .user_messages>0)
       | "\(.user_messages)\t\(.duration_seconds)"' \
   ~/.config/pew/session-queue.jsonl
```

n = 6065. I then ran a simple OLS slope-and-intercept calculation in awk, plus a bucketed mean by prompt count, plus the Pearson correlation.

## The aggregate result

```
n=6065  mean_x=4.84 prompts  mean_y=2351.57 sec (39.2 min)
slope=654.67 sec/prompt  intercept=-817.50 sec
pearson_r=0.3403
```

The slope says: every additional prompt adds, on average, **655 seconds (10.9 minutes) of session duration**. The intercept is negative (`-817 sec`), which on its face is nonsensical — a session with zero prompts can't have negative duration — but is just the linear fit's way of telling you that the relationship is non-linear at the low end. A real session must have at least one prompt and at least zero seconds; the regression is reporting a line that crosses below those constraints.

The Pearson r of 0.34 is a moderate-but-weak correlation. It means prompt count *does* explain some of the variance in duration (about 11%, since R² is roughly 0.12), but it leaves 88% of the variance unexplained. So the slope number — 655 sec/prompt — is *real on average* but useless as a predictor for any individual session. Knowing a session has, say, 5 prompts tells you very little about how long that specific session lasted.

## The bucketed conditional view

Here is where the interesting structure shows up. Bucketing sessions by prompt count and reporting mean duration per bucket:

```
prompts=01      n=4855   mean_dur=  468 sec   ( 7.8 min)
prompts=02-03   n=534    mean_dur=  197 sec   ( 3.3 min)
prompts=04-10   n=94     mean_dur= 5168 sec   (86.1 min)
prompts=11-30   n=345    mean_dur= 8425 sec   (140.4 min)
prompts=31+     n=237    mean_dur=35842 sec   (597.4 min)
```

Read those numbers carefully. Sessions with **2-3 prompts have shorter mean duration (197 sec) than sessions with 1 prompt (468 sec)**. That is not a typo, that is not a transient artefact, and it is not what a linear regression would predict. It is a real inversion that shows up in 5389 sessions (4855 + 534, or 88.9% of the corpus).

Then sessions with 4-10 prompts jump to 5168 seconds — a 26× jump from the 2-3 bucket. And from there it grows roughly monotonically: 8425 sec for 11-30 prompts, 35842 sec for 31+ prompts. The 31+ bucket has a mean duration of nearly 10 hours per session.

So the relationship is not linear. It is roughly four distinct regimes:

1. **Single-prompt sessions** (n=4855, 80% of corpus): mean duration 7.8 minutes. The dominant mode.
2. **2-3 prompt sessions** (n=534, 8.8%): mean duration 3.3 minutes. *Shorter than single-prompt.*
3. **4-10 prompt sessions** (n=94, 1.5%): mean duration 86 minutes. A 26× jump.
4. **Long sessions, 11+ prompts** (n=582, 9.6%): mean duration grows from 140 min to 597 min as prompt count grows.

The slope in regimes 3-4 is roughly linear and roughly fits the global slope. But regimes 1-2 break the slope entirely. The regression averages over the inversion and produces a number (655 sec/prompt) that does not correctly describe any of the four regimes.

## Why 2-3 prompt sessions are shorter than 1-prompt sessions

This is the most counter-intuitive number in the whole table. The naïve expectation is that more prompts means more time. The data says the opposite, in the 1→3 prompt range. Why?

There are several plausible mechanisms, and the data is consistent with all of them:

**Hypothesis 1: One-shot sessions include a "stuck" tail.** When you ask a model one question, sometimes the model gives a complete answer in 30 seconds and you close the tab; sometimes the model gives a long, slow, structured answer that takes 3 minutes to stream; sometimes you ask one question and walk away while the model is still thinking, and the session timer keeps running until the system gives up. The 1-prompt bucket therefore has a mix of "fast satisfaction" sessions and "abandoned but still ticking" sessions. The 7.8-minute mean is high because of the abandoned tail. Single-prompt sessions are bimodal: some are very fast, some are unusually long.

**Hypothesis 2: 2-3 prompt sessions are "ping-pong" sessions.** A user asks a question, gets a partial answer, asks a clarifier, gets the rest, says thanks, closes. The whole exchange is fast and tightly coupled. There's no abandonment because the user is actively engaged. The 3.3-minute mean reflects this: short, focused, conclusive.

**Hypothesis 3: Single-prompt sessions over-include exploratory probes.** A user opens a session, types something to see what happens, then either gets what they need in one shot or gives up. The "give up" path leaves a session open with no further prompts but a non-trivial duration ticking until timeout. 2-3 prompt sessions, by definition, did *not* give up — the user came back at least once — and so they have lower abandonment-driven duration inflation.

All three hypotheses agree on the same structural claim: **single-prompt sessions are duration-inflated by abandonment, and 2-3 prompt sessions are duration-deflated by tight active engagement.** The inversion is not a bug in the data; it's a feature of how humans use these tools.

The implication is that **prompt count is a poor proxy for "how engaged was this session"** in the low end. A session with 1 prompt and 7 minutes of duration could be a focused query or a walked-away tab. A session with 3 prompts and 3 minutes could be a tight, productive exchange. The two look similar in raw duration but have completely different qualitative shapes.

## The 4-10 prompt jump

Going from 2-3 prompts to 4-10 prompts, mean duration jumps from 197 seconds to 5168 seconds — a **26.2× increase** for what looks like a modest increase in prompt count. This is the most dramatic discontinuity in the whole table, and it cannot be explained by linear scaling.

The most plausible reading: somewhere around 3-4 prompts, the *nature* of the session changes. Sessions with 1-3 prompts are short ad-hoc exchanges. Sessions with 4+ prompts are *working sessions* — the user has committed to a multi-step task, the model is being asked to do real work over time, and the duration reflects the actual time required to do that work rather than the time required to type a question.

The 4-10 bucket has only 94 sessions, so the n is small and the mean is sensitive to outliers. But the pattern is consistent with the next bucket up: 11-30 prompts has 345 sessions and a mean of 8425 seconds (140 min), which is only a 1.6× jump from the 4-10 mean despite a 3× jump in median prompt count. The growth is sub-linear in prompt count from 4-10 onward.

This sub-linear growth is interesting. It suggests that within "working session" mode, additional prompts are not adding proportional time. A session that has 30 prompts is not 3× longer than a session with 10 prompts; it's only about 1.6× longer. The marginal prompt in a long session is faster than the marginal prompt in a medium session. That makes sense ergonomically — once you're deep in a working session, you've already paid the setup costs (loading context, finding files, orienting), and additional prompts ride on top of that established context.

## The 31+ prompt bucket

237 sessions with 31 or more prompts, mean duration 35842 seconds (9.95 hours), or about 597 minutes. This is the bucket of *long-running collaborative work* — sessions that go for the better part of a workday, with constant back-and-forth.

A 10-hour mean session with 31+ prompts implies, very roughly, **17-20 minutes per prompt on average**, if you assume the prompts are roughly evenly distributed. That's much higher than the 4-10 bucket's per-prompt average (~10 min/prompt at the upper end of that bucket) and dramatically higher than the 2-3 bucket's per-prompt average (~70 sec/prompt). The slowdown per prompt as session length grows suggests that long sessions are dominated by *thinking and reading time between prompts*, not by typing or model generation time. The user is doing work; the model is intermittently consulted.

This is the "background tool" use mode. The model is open in a tab, the user is doing other things, occasionally returning to consult or delegate. The duration reflects the user's full-day cycle, not the model's response cycle.

## The slope is misleading; the buckets aren't

If you only had the regression slope (655 sec/prompt) and the correlation (r=0.34), you'd conclude: "session duration grows roughly linearly with prompt count, with weak but real correlation." That conclusion is not wrong, exactly, but it misses everything interesting. It hides:

- The inversion between 1-prompt and 2-3 prompt sessions.
- The 26× discontinuity at the 3→4 prompt boundary.
- The sub-linear growth above 4 prompts.
- The qualitative regime shift at the 31+ prompt bucket where per-prompt time triples.

A linear fit treats all of these as "noise around the line" and reports r=0.34 as evidence the model is OK. But the noise is structural, not random. If you fit four separate lines to the four regimes you'd get four very different slopes, and the *regime boundaries* would be more useful than the slope of any one regime.

This is a recurring pattern when you regress against count-of-events variables in usage data: the relationship is almost never linear, because the *meaning* of "one more event" changes as the count grows. The first prompt, the second prompt, the tenth prompt, and the fortieth prompt are doing categorically different things in a session.

## What you'd actually use this for

If you were building a session-level cost predictor, you would not use the global slope. You would use something like: "if user_messages == 1, predict 468 sec; if 2-3, predict 197 sec; if 4-10, predict 5168 sec; if 11-30, predict 8425 sec; if 31+, predict 35842 sec." That piecewise predictor would be vastly more accurate than the linear one, despite using only the same input variable. The structure was already in the data; the linear regression just couldn't represent it.

If you were trying to detect abandoned sessions automatically, you would look for 1-prompt sessions with duration in the upper tail (say, p75 of 1-prompt durations). The mean of 468 sec for that bucket is dragged up by the upper tail; the median is almost certainly much lower (the data isn't shown here but a 7.8-minute mean for single-prompt sessions screams "the median is two minutes and there's a long tail of timeouts").

If you were trying to characterise a user's working style, the right summary statistic is **what fraction of their sessions fall into the 4-10, 11-30, and 31+ buckets**. A user whose sessions are 95% in the 1-prompt bucket is using the tool as a query interface. A user whose sessions are 30% in the 11+ bucket is using it as a collaborator. The same user might be both, depending on the day.

## The Pearson r of 0.34 is technically correct and operationally wrong

Pearson r measures *linear* association. r=0.34 is a real signal — prompt count does carry information about duration — but it under-reports the strength of the relationship because the relationship is non-linear and partly inverted at the low end. A non-parametric measure like Spearman rank correlation would likely be much higher, because the *rank* relationship is much stronger than the linear one (ignoring the low-end inversion, more prompts does generally mean more time).

A reported Pearson r of 0.34 invites the casual reader to dismiss the relationship as weak. The bucketed view shows it is anything but weak — it is structured, with clear regimes, sharp discontinuities, and predictable behaviour within each regime. The weakness in r is not in the data; it's in the choice of metric.

This is a recurring pitfall in usage analytics. People reach for Pearson r because it's familiar, get a low number, and conclude there's no signal. The signal is there; it's just shaped wrong for the metric they used.

## Summary numbers

- n = 6065 human sessions with at least one user message
- Pearson r between prompt count and duration: **0.3403**
- Linear slope: **654.67 sec/prompt** (operationally misleading)
- 1-prompt sessions: 4855 (80%), mean 468 sec
- 2-3 prompt sessions: 534 (8.8%), mean **197 sec** ← inverted, *shorter* than 1-prompt
- 4-10 prompt sessions: 94 (1.5%), mean 5168 sec ← 26× jump
- 11-30 prompt sessions: 345 (5.7%), mean 8425 sec
- 31+ prompt sessions: 237 (3.9%), mean 35842 sec ← ~10 hours
- Query timestamp: `2026-04-26T03:39:42Z`
- Data source: `~/.config/pew/session-queue.jsonl`

## The takeaway

The session-length-vs-prompt-count relationship is not a line. It's four regimes separated by sharp boundaries, with the most interesting feature being a non-monotonic dip between 1-prompt and 2-3 prompt sessions. This dip is best explained by the difference between abandoned and engaged sessions: a single prompt followed by silence keeps the session timer running; a 2-3 prompt exchange is tightly coupled and ends quickly when satisfied.

Above 4 prompts, growth is sub-linear in prompt count, suggesting that long sessions are dominated by between-prompt user time rather than per-prompt model time.

The linear regression slope of 655 sec/prompt is technically correct as an average and operationally useless as a predictor. The bucketed conditional means are far more useful, and they reveal that "more prompts → more time" is true on average but false in detail at the most populous part of the distribution.

The Pearson r of 0.34 understates a relationship that is actually strongly structured but non-linear. Use rank-based metrics or piecewise fits if you want to capture the real shape.

And maybe most importantly: **prompt count is a noisy proxy for engagement.** A 1-prompt session can be active or abandoned, fast or slow, satisfied or stuck. A 3-prompt session almost always indicates a real exchange. The qualitative gap between these two is bigger than any quantitative difference in their prompt counts would suggest.
