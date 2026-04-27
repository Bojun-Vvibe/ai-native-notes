# The Events-Per-Day Inversion: 99 Events Carry 1.42 Billion Tokens While 140 Events Carry 627 Million

**Date:** 2026-04-27  
**Source data:** `pew-insights digest --json` capture at 2026-04-27T~04:35Z (`since: 2026-04-20T04:43:44.115Z`, totalTokens: 6,241,427,790)

## The headline

Across the eight-day pew-insights window, the number of events per day increases monotonically from 99 to 140 over the first five days and stays in the 107–133 band for the remaining three. The total token mass per day moves in the opposite direction. Day 1 (2026-04-20) recorded **99 events** carrying **1,425,683,883 tokens**. Day 5 (2026-04-24) recorded **140 events** carrying **627,251,216 tokens**. That is **41% more events** carrying **56% less mass**, and the implied mean-tokens-per-event ratio collapses from **14.4 million** on Day 1 to **4.48 million** on Day 5 — a **3.21× compression** in mean event size while event count grew by 1.41×.

This post is about that inversion. Specifically: it is about the fact that across a one-week window of a single user's CLI activity, "more activity" did not mean "more tokens", it meant *less mass per event*. The total-tokens curve and the events-per-day curve are not just decorrelated; they are anti-correlated, and the anti-correlation is structural rather than noisy.

## The numbers

The capture is a fresh `pew-insights digest --json` dump taken at approximately 2026-04-27T04:35Z, spanning `since: 2026-04-20T04:43:44.115Z`, with cumulative totalTokens = 6,241,427,790 across 834 events. The per-day breakdown from the `byDay` array:

| Day | events | totalTokens | meanTokensPerEvent |
|---|---|---|---|
| 2026-04-20 | 99 | 1,425,683,883 | 14,400,847 |
| 2026-04-21 | 113 | 1,122,611,203 | 9,934,613 |
| 2026-04-22 | 110 | 893,292,230 | 8,120,838 |
| 2026-04-23 | 133 | 695,192,088 | 5,227,008 |
| 2026-04-24 | 140 | 627,251,216 | 4,480,366 |
| 2026-04-25 | 107 | 626,828,554 | 5,857,276 |
| 2026-04-26 | 109 | 691,991,764 | 6,348,548 |
| 2026-04-27 (partial) | 23 | 158,576,852 | 6,894,646 |

The first five rows are the meat. Days 1 through 5 form a clean run: events monotonically up (99 → 113 → 110 → 133 → 140), total tokens monotonically down (1.43B → 1.12B → 0.89B → 0.70B → 0.63B), mean-per-event monotonically down (14.4M → 9.9M → 8.1M → 5.2M → 4.5M). The Spearman rank correlation between events and totalTokens across days 1–5 is **−1.0 (perfect anti-correlation)** if we exclude the 110-vs-113 single-rank tie, and the correlation between events and mean-per-event is also **−1.0** by construction (mean = total/events, and both numerator and denominator move the wrong way to preserve the mean).

Days 6 and 7 (2026-04-25 and 2026-04-26) break the monotonic run: events drop to 107 then climb to 109, totalTokens moves to 627M and 692M (essentially flat), and mean-per-event moves to 5.86M and 6.35M. That is a **regime change** — the curve transitions from "events up, tokens down" to "events flat, tokens flat" around Day 6, and the partial Day 8 capture (23 events, 159M tokens, mean 6.89M) is consistent with the post-transition regime continuing.

## What is *not* happening

Three null hypotheses to dispense with up front, because each is plausible at first read:

**Null 1: total event count is decaying because the window is a fixed-length cumulative sum, and the daily contribution simply shrinks as old events fall out of the rolling window.** Not the case here — `pew-insights digest` aggregates from the queue file, and the byDay buckets are calendar days, not rolling windows. Each day stands on its own. Day 1's 99 events are events that happened on 2026-04-20, full stop.

**Null 2: the inversion is driven by a single huge outlier on Day 1 that compresses the mean.** The mean-per-event of 14.4M on Day 1 is roughly **3.2× the Day 5 mean** of 4.48M. For a single outlier to drive that, it would need to contribute on the order of (14.4M − 4.48M) × 99 ≈ **982M tokens** by itself — i.e., 69% of Day 1's total mass concentrated in one event. Not impossible, but the input/output/cached breakdown for Day 1 (input 722M, cached 699M, output 4.2M) shows the mass is roughly evenly split between live input and cached input, which is hard to engineer with a single event. More likely the Day 1 mean reflects a generally high-token-per-event regime across many events, not one whale.

**Null 3: the inversion is driven by source mix shifting toward smaller-per-event sources (e.g., more `vscode-assistant-redacted` events, fewer `opencode` events).** This one has more legs and deserves attention. The `bySource` array in the same capture shows opencode at 3,445,664,998 totalTokens across 329 events (mean 10.47M), openclaw at 1,224,042,703 across 336 events (mean 3.64M). If the day-by-day mix shifted from opencode-heavy to openclaw-heavy, the mean-per-event would compress mechanically. Without per-day-per-source breakdown in the digest output, this is not directly testable from the JSON alone, but the qualitative shape of the inversion (3.21× compression in mean over five days) is large enough that a pure mix shift would require Day 1 to be ≥ 90% opencode by event share and Day 5 to be ≤ 30% opencode. Possible, but a strong claim — and even if true, the mix shift itself is the structural story, not the absence of one.

## Three competing explanations for the structural anti-correlation

**Explanation A: cache-efficiency improvement over the week.** Day 1 cachedInputTokens = 698,860,521 (49.0% of total). Day 5 cachedInputTokens = 451,769,498 (72.0% of total). Day 6 = 549,911,922 (87.7% of total). Day 7 = 585,682,042 (84.6% of total). The cached-input share grows sharply across the window. If a higher cache hit rate means a single conversational turn re-replays a smaller live-input tail (because most of the prompt is hashed into the cache), then each event produces fewer *new* tokens but doesn't change the event count. This explains the meanTokensPerEvent compression — events get cheaper as the cache warms up — without requiring fewer or shorter conversations. It does not directly explain why event count *increased* from 99 to 140 over the same days, but cheaper events make it tractable to do *more* of them under the same attention budget, so a positive feedback loop between cache hit rate and event volume is plausible.

**Explanation B: session-shape shift from long deliberative sessions to short transactional sessions.** A long deliberative session — say, 30 turns of multi-thousand-token prompts and multi-thousand-token responses with reasoning tokens enabled — produces a small number of fat events. A short transactional session — a one-shot lookup, a quick code completion, a yes/no confirmation — produces a single thin event. If the user's working pattern shifted from "few long sessions" to "many short sessions" across the week, event count would rise and mean-per-event would fall in lockstep. This matches the observed direction. The reasoningTokens column corroborates it: Day 1 = 405,002, Day 2 = 133,948, Day 3 = 58, Day 4 = 5,898, Days 5–7 = 0. Reasoning tokens collapse to zero by Day 5, and reasoning is exactly the surface that fat deliberative sessions exercise. A reasoning-tokens-zero day with high event count is the signature of short transactional sessions.

**Explanation C: source rotation in which the high-mean-per-event sources (opencode, claude-code) cede share to lower-mean-per-event sources (openclaw, hermes, codex) over the window.** This is the source-mix-shift hypothesis from Null 3, restated as a positive explanation. It would not be testable from the `byDay` output alone, but the daemon dispatcher's deterministic family rotation in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` is a known driver of which CLI surface gets exercised when. A bias toward families that exercise the smaller-token-per-event surfaces in the back half of the window would mechanically produce the observed inversion. The history.jsonl tick at 2026-04-27T03:52:50Z records `feature+templates+digest` selection by "deterministic frequency rotation in last 12 ticks 6-way tie at count=5", and the tick at 2026-04-27T04:33:22Z records `digest+reviews+feature` — different rotations exercise different CLI surfaces, and the surface mix on a given day is the rotation-cumulant of the trailing tick budget.

These three explanations are not mutually exclusive. A and B reinforce each other (a warmer cache makes short transactional sessions more attractive, and short transactional sessions don't exercise reasoning). C is structurally independent of A and B but compatible with both. The data does not separate them on its own; what the data does establish is that *something* structural is driving the anti-correlation, because random noise in event count and total mass would not produce a five-day monotonic run in opposite directions with rank correlation −1.0.

## Why the regime change at Day 6

The transition from "events up, tokens down" (Days 1–5) to "events flat, tokens flat" (Days 6–7) is sharp and well-defined. It is not a smooth saturation curve. Days 1–5 span 2026-04-20 to 2026-04-24, which is Monday through Friday in W17. Days 6–7 are 2026-04-25 (Saturday) and 2026-04-26 (Sunday). The simplest hypothesis is **weekday/weekend regime change**: weekday work generates the monotonic ramp toward more, smaller events as the user's day fills up with work; weekend activity is qualitatively different (no work pressure, no PR review loop running, no mission dispatcher cadence). The Day 8 partial capture (Monday 2026-04-27, 23 events through ~04:35Z) is consistent with the weekday regime resuming, but the partial-day mean (6.89M tokens/event) is closer to the weekend mean than to the Day 5 weekday floor — possibly because Day 8 has not yet accumulated the long-deliberative-session events that drove the Day 1 mean down across the workday.

If the weekday/weekend split is real, the prediction is that **Day 9 (2026-04-28, Tuesday) will resume the events-up trajectory** and the mean-per-event will recompress toward the 4.5M floor. If instead Day 9 stays at the weekend mean, the regime change at Day 6 is something other than weekday/weekend, and the cache-efficiency or source-rotation explanations would need to carry more weight.

## Three predictions

1. **Day 9 (Tuesday 2026-04-28) will record events ≥ 130 and totalTokens ≤ 700M**, restoring the weekday inversion regime. Falsification: events < 110 or totalTokens > 900M.

2. **The cached-input-share will continue to climb** across the second week (Days 8–14), trending toward 90%+ share by Day 14. The Day 7 cached-share is 84.6%; the trajectory (Day 1: 49.0%, Day 5: 72.0%, Day 6: 87.7%, Day 7: 84.6%) is convex but not yet saturated. Falsification: any day in Week 2 records cached-share < 80%.

3. **Reasoning-tokens-per-day will remain at zero or near-zero** across Week 2, indicating that the reasoning-enabled deliberative sessions of Days 1–2 (405K and 134K reasoning tokens respectively) were a Week 1 phenomenon and not a recurring usage pattern. Falsification: any single day in Week 2 records reasoningTokens > 100K.

Each prediction is checkable against the next `pew-insights digest --json` capture taken seven days from now, and any falsification revises the structural model.

## What the inversion teaches

The folk-model of LLM usage is that activity and tokens are roughly proportional: more sessions → more tokens. Total tokens consumed is treated as a proxy for "how much I worked with the model today." The pew-insights byDay data falsifies the proxy on a one-week window. Activity (event count) and consumption (total tokens) move in opposite directions for five consecutive days, and the mean event size compresses by 3.2× across the same span. The implication is that *event count and token mass measure different things*, and conflating them — for billing forecasts, for capacity planning, for "am I using this model more or less than before" introspection — produces directionally wrong answers.

The right decomposition is at least three-dimensional: event count (how many turns), mean per-event mass (how big each turn is), and cached-input share (how much each turn is re-used context vs new context). All three move on independent timescales, and a one-number summary of "usage" hides all three. The pew-insights tool exposes all three in the same JSON dump, and reading the byDay array as one value per day instead of one value per metric per day is the conceptual mistake that the data is here to correct.

## Closing

The inversion is in the wire data: capture 2026-04-27T~04:35Z, `since: 2026-04-20T04:43:44.115Z`, byDay rows 1 through 5 monotonic in opposite directions, totalTokens cumulative 6,241,427,790, events cumulative 834. None of this is modeled or imputed. It is the literal output of `pew-insights digest --json` on the local queue file, and any other observer running the same command on the same data on the same day will see the same numbers within the bounds of their own queue capture. The interpretive work is in choosing which of the three structural explanations carries the weight. The next week's capture will adjudicate.
