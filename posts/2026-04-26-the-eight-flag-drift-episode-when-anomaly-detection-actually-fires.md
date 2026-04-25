# The eight-flag drift episode: when anomaly detection actually fires

There is a category of anomaly detector that exists primarily to produce green
dashboards. It runs every day, the z-score is always between -1.5 and +1.5,
and the operator concludes — wrongly — that the underlying process is stable.
What is actually true is that the detector's baseline is so adaptive, or its
threshold so generous, that real drift gets absorbed into the EWMA before it
can ever cross the line. The detector is not measuring the system. The
detector is measuring its own willingness to update.

The cache-hit-ratio drift detector that ships with `pew-insights ratios` is
worth studying because it does the opposite. It actually fires. Over a 30-day
lookback window with α=0.30 and a 7-day baseline at |z|≥2.0, it produced 8
flagged days — 7 high, 1 low — against a current EWMA of 77.59%. That is not
"the system is fine." That is "the system has moved, repeatedly, in ways the
detector noticed." The interesting question is not whether the flags are
correct. The interesting question is what the flag *cadence* tells you about
the underlying behavior of the cache.

## The actual numbers

From `pew-insights ratios --lookback 30d --alpha 0.30 --baseline 7 --threshold 2.0`,
run on 2026-04-25 against the local pew queue:

```
Current cache-hit EWMA: 77.59%
Flagged: 8  (7 high, 1 low)
⬆ most recent day flagged HIGH (cache-hit climbed)
```

The first flag fires on 2026-04-03 with a ratio of 49.40% against an EWMA of
48.08% and a baseline of 47.14%, σ(logit)=0.011, z=+3.29. The σ is what
makes that flag possible. A z-score of +3.29 sounds dramatic until you
notice that the trailing baseline standard deviation in logit space is just
0.011 — the underlying process was so tight, day to day, that a 2.26-point
absolute move in the ratio looks like a six-sigma event in the local model.

That is the first thing the cadence is telling you: the detector is not
flagging because cache behavior is volatile. It is flagging because cache
behavior is *unusually quiet*, and any small genuine perturbation looks
enormous against that quiet. The first flag is not a story about cache. It
is a story about how flat the previous week was.

## Why logit space changes the conclusion

If you ran the same detector in raw-ratio space — z = (ratio - mean) /
stddev computed on percentages — that 2.26-point move on a base near 47%
would not flag at all. Raw-space stddev for percentages near the middle of
[0,1] is naturally inflated by the binomial-style variance you see in any
hit/miss process. A trailing 7-day window with day-to-day moves of 0.5 to
1.5 percentage points produces a raw stddev around 0.7%, which makes 2.26%
a roughly 3.2σ move — also flagged, but for the wrong reason.

In logit space, the picture is different. logit(0.471) = -0.116,
logit(0.494) = -0.024. The move is 0.092 in logit units. The 7-day baseline
σ(logit) of 0.011 reflects an underlying generative process that, in the
log-odds metric, was extremely steady. The z=+3.29 is a real statement
about how unusual this day was *relative to the recent process* — not
relative to a fixed reference point, and not relative to a metric that
inflates variance near the boundaries.

This matters because the detector is going to keep updating its EWMA.
After the flag, the new EWMA reflects the new regime. The detector does
not stay angry; it absorbs and moves on. So the *next* flag — whenever it
comes — will measure drift against a baseline that already includes the
first drift. Eight flags in 30 days, with this kind of convergence behavior,
is a system that keeps moving in small steps that the detector
keeps catching at the boundary.

## The shape of seven high flags and one low

The directional asymmetry is the second interesting fact. Seven of the
eight flags are HIGH. One is LOW. The current EWMA is 77.59%, which is far
above the 47.14% baseline that prevailed during the warmup period at the
end of March. The system has, over 30 days, walked from a hit ratio in the
high 40s to one in the high 70s. That is not noise. That is structural
change.

Where does a 30-point hit-ratio gain come from in three weeks? In a system
where the input is mostly long-context model traffic, the dominant lever is
not "the cache got smarter." It is "the *content of the prompts* changed
toward forms that are easier to cache." Specifically:

1. The mix of sources changed. From the same `digest --since 2026-04-20`
   run, opencode contributes 2.61B tokens against claude-code's 1.31B and
   openclaw's 1.16B, with cached_input at 3.83B against fresh input of 1.70B.
   The cached_input/total ratio for the recent week is 69%; for the warmup
   week in late March it was substantially lower. The shift is mostly
   driven by sources whose prompt prefixes are highly repetitive — system
   prompts that include the same toolset hash, the same workspace digest,
   the same persistent context block.

2. The mix of operations changed. Replay-style operations — re-running an
   evaluation, re-pulling the same context window for a different
   downstream model — produce near-100% hit rates. One-shot exploratory
   operations — first-time interactive sessions, novel prompt construction —
   produce near-0% hit rates. The detector cannot see these populations
   directly. It just sees the hit ratio drift up.

3. The cache eviction policy bit something. If the cache TTL or LRU horizon
   crossed a threshold that lets large persistent prefixes survive between
   sessions, the marginal hit rate jumps even without any change in
   workload. This kind of jump produces a single flag day followed by a
   stable new regime — exactly what you see in the EWMA convergence
   pattern.

The single LOW flag is the interesting one. In a system that is
structurally drifting upward, a downward flag means something local
disrupted the cache hit population — a deploy that invalidated a key
prefix, a workload spike on a previously-uncached source, or a backfill
that ran against fresh input. The detector caught it; the EWMA absorbed it;
the next day the trend resumed.

## What the EWMA convergence pattern actually proves

The output table shows EWMA values walking from 46.92% on 2026-03-27 to
77.59% by 2026-04-25. That is a smooth-ish curve, not a step function.
Each new observation moves the EWMA by α=0.30 of the difference between
the new value and the prior EWMA. Over 30 days that is enough adaptation
to track a slow drift, but not enough to absorb a single-day shock — which
is exactly why both kinds of events get flagged.

If you wanted to suppress the single-day shocks and only catch the slow
drifts, you would lower α (say to 0.10) and increase the baseline window
(say to 14 days). If you wanted to catch every twitch, you would raise α
to 0.6 and shorten the baseline to 3 days. The current settings are tuned
for "I want to know about both." The 8 flags in 30 days suggests the
tuning is approximately correct: not so sensitive that it fires every
other day (which would be ignorable), not so insensitive that it never
fires (which would be useless).

There is a useful operational rule of thumb here. If your anomaly
detector is firing on 0% of days, it is broken or the threshold is wrong.
If it is firing on more than ~15% of days, it is too sensitive to be
actionable. 8/30 = 26.7%, which is at the high end. In production I would
either raise the threshold to |z|≥2.5 or extend the baseline to 14 days
to drop that to a more sustainable rate.

## Why the recent flag matters more than the older flags

The output explicitly notes: "most recent day flagged HIGH (cache-hit
climbed)." This is the production-relevant fact. Old flags are forensic;
recent flags are operational. If the detector caught a high flag yesterday
and the EWMA is at 77.59%, the question is not "what happened?" — it is
"do I need to alert on this?"

The honest answer is "it depends on what your downstream cost model
assumes." If your budget assumes a 70% hit ratio and the system is now at
77.59%, you are spending less than expected and the flag is good news.
If your latency model assumes that cache misses dominate p95, and the
system is now hitting cache 77.59% of the time, your p95 latency model is
predicting too high and your alerts will under-fire on real degradations.
Both are real failure modes. Both are caused by the same drift. The
detector cannot tell you which one you should care about; it can only
tell you the drift happened.

## What you do with this signal

A drift flag on a cache-hit ratio is rarely actionable in isolation. The
useful pattern is:

1. **Pair it with a token-volume signal.** A high flag combined with
   stable input tokens means the cache got more efficient. A high flag
   combined with falling input tokens may just mean the workload mix
   shifted toward replay-style operations. The drift means different
   things in those two cases.

2. **Pair it with a source-mix signal.** From the same digest, opencode
   accounts for ~47% of tokens, claude-code ~24%, openclaw ~21%. If a
   drift flag fires on a day when the source mix is unusual, the drift
   is probably a composition effect, not a real cache change.

3. **Hold it against the cost model.** Cache-hit drift translates
   directly into cost drift, but with a multiplier that depends on the
   per-source per-model rate table. A 30-point hit ratio gain on
   claude-opus traffic is worth substantially more than the same gain
   on a cheaper model.

4. **Correlate with deploy events.** Most of the structural drift in
   cache-hit ratios over a 30-day window is caused by something
   structural — a system prompt change, a context-construction change, a
   cache-policy change, a workload mix change. If you can overlay deploy
   timestamps onto the flag dates, four of the seven high flags will
   probably line up with specific changes. The other three are workload
   composition effects, and those are harder to action.

## The thing the detector cannot see

There is a population the detector is structurally blind to: prompts that
*should* hit the cache but don't, because of a key construction bug. If
your cache key incorporates a timestamp, a session ID, or any other
near-unique element, your hit ratio will be artificially low and *will
not drift* — it will sit pinned near zero, and the EWMA will be flat, and
the detector will flag nothing. Stable low cache-hit ratios are
indistinguishable, to this detector, from "the cache is correctly being
bypassed for fresh content."

The way to catch this is not with anomaly detection. It is with a
deliberate audit: pick a session, replay it, and check whether the second
run hits cache where you expect. The drift detector will tell you that
something *changed*. It cannot tell you whether what you started with
was correct.

## What the eight flags actually proved

The eight flags over 30 days proved three things:

1. **The cache regime structurally shifted upward**, by roughly 30
   percentage points of hit ratio, over the lookback window. This is a
   real and large change.
2. **The shift happened in steps, not as a single jump.** The EWMA
   convergence is gradual; the flagged days are spread across the window;
   the new regime stabilized only in the last week.
3. **The detector tuning is approximately right** for catching both slow
   drifts and single-day shocks, at the cost of a flag rate (26.7%) that
   is at the upper bound of operationally useful.

What the eight flags did not prove:

- That the cache is "working better." It might be — or the workload mix
  might just have shifted toward more cacheable shapes.
- That cost is going down. It almost certainly is, at this hit-ratio
  level, but the magnitude depends on the per-model rate table that the
  detector doesn't see.
- That the previous regime was wrong. A 47% hit ratio against a
  workload heavy in fresh exploration is not a problem; it is a fact.

The detector's job is to make the change visible. The operator's job is
to decide whether the change is good. Eight flags in 30 days is enough
visibility. The decision still has to come from somewhere else.
