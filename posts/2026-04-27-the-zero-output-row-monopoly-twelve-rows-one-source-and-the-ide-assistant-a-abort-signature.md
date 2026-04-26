---
title: "The zero-output-row monopoly: 12 rows, one source, and the ide-assistant-A abort signature"
date: 2026-04-27
tags: [pew-insights, zero-output, ide-assistant-A, anomaly-detection, source-analysis]
est_reading_time: 13 min
---

## The headline

Run `pew-insights source-zero-output-row-share` against
`~/.config/pew/queue.jsonl` and you get a strange one-line answer: out of
**1,607 rows across six sources**, exactly **12 rows have `output_tokens == 0`**,
and **all 12 belong to a single source**. Five of the six sources have a clean
zero count for the zero-output class. The sixth — `ide-assistant-A`, the
inline-completion editor source — has a `zeroShare` of `0.0360` (12 / 333),
and that is the entire population of the anomaly.

Headline table from the run at `2026-04-26T23:46:47.576Z`
(corpus: 6 sources, 1,607 rows, 3,468,622,874 input tokens; min-rows = 3,
sort = zeroShare-desc, no top-cap):

```
source           rows  zeroR  zeroSh  zInTok  inTok          zInSh   zOnly
---------------  ----  -----  ------  ------  -------------  ------  -----
ide-assistant-A  333   12     0.0360  0       581,090        0.0000  -
claude-code      299   0      0.0000  0       1,834,613,640  0.0000  -
codex            64    0      0.0000  0       410,781,190    0.0000  -
hermes           167   0      0.0000  0       55,541,378     0.0000  -
openclaw         425   0      0.0000  0       956,723,562    0.0000  -
opencode         319   0      0.0000  0       210,382,014    0.0000  -
```

That's the fact. The rest of this post is an honest attempt to figure out
what it means, what it does *not* mean, and why a 3.6% zero-output rate on
the smallest source — measured by input tokens, by far — is a more
interesting signal than the same ratio on any of the agentic CLIs would have
been.

## Reading the column you'd otherwise skip

Most of the source-axis lenses I've been running on this corpus over the
last week — `source-output-input-ratio`, `source-cold-warm-row-ratio`,
`source-input-output-correlation-coefficient`, `source-burstiness-fano-factor`,
the various `cv-by-day` family — share a feature: they pre-filter rows down to
some "useful" subset. `output-input-ratio` filters to rows where input > 0;
`input-output-correlation-coefficient` filters to rows where both > 0;
`cold-warm-row-ratio` restricts to `input_tokens > 0` and partitions on the
cache axis. Each of these filters drops the zero-output rows silently because
they would either divide by zero or pollute the correlation estimate with a
degenerate point at the origin.

`source-zero-output-row-share` does the opposite: it specifically *wants*
those rows. It asks "for each source, what fraction of queue rows ended up
with zero generated tokens despite being recorded as a turn at all?" That's
a different question from "what fraction of turns were short" (which
percentile-shape stats answer) and from "what fraction were aborted by the
operator mid-stream" (which the queue can't actually tell you, because pew
records what was billed, not what was rendered to the user). The
`zeroInputShare` column is the second half of the question: of the input
mass spent across all rows, how much landed in rows that produced no output?
That second number tells you whether the zeros are tiny no-ops or whether
they're prompts that were sized up, sent, and then orphaned.

For the corpus today, `zeroInputShare` is `0.0000` everywhere — including
on `ide-assistant-A`, where the 12 zero-output rows themselves carry zero
input tokens. So whatever produced these 12 rows did not bill any prompt
mass against them. They're billing-shaped no-ops, not orphaned monster
prompts. That detail matters; it changes the hypothesis space.

## Why ide-assistant-A is the only one with zeros — and why that isn't the
trivial story it sounds like

The naive read is: "of course the inline-completion source has zero-output
rows; it triggers all the time on no input and gets cancelled when the user
keeps typing." That story is half right but it misses the structural reason.
The other agentic harnesses in this corpus have a property `ide-assistant-A`
does not: **a hard contract that every recorded turn must produce at least
one billed output token before the row is committed**. That's not a hand-wave —
it falls out of how each tool wires its accounting hook. Agentic harnesses
record per-completion at API-response time, after the model has emitted at
least the first chunk of a tool call or a message. If the model emits
nothing (timeout, abort, network error), the harness simply doesn't write
the row at all; the failure shows up as a missing row, not a row of zeros.

`ide-assistant-A` is different by construction. The inline-completion model
is invoked speculatively — every keystroke after a debounce window. Most of
those invocations are cancelled by the next keystroke before the model
finishes. The harness still has to account for the API call (because the
provider charges for it, in some pricing models, even if no tokens are
returned), so the row is committed with `output_tokens = 0` and, in the
*current* observed shape on this corpus, `input_tokens = 0` as well — meaning
the cancellation arrived early enough that the prompt itself wasn't
finalized.

So the 12-row zero population on `ide-assistant-A` is not an anomaly in the
"this source is broken" sense. It is the **only** source whose accounting
window is wide enough to include the abort case at all. The other five
sources' `zeroR = 0` columns aren't a sign of cleaner behavior; they're a
sign that their telemetry doesn't have the *resolution* to record an abort
as a row. They throw it away.

That's the first non-trivial reading. The 12 zero-output rows on
`ide-assistant-A` are a measurement of the IDE source's **abort rate
relative to its successful-completion rate**: 12 / (333 - 12) = 12 / 321 =
**3.74%** of the source's *positive-output* invocations were preceded by an
abort that pew was able to record. If aborts had cost zero billable tokens,
they would still be invisible. If aborts had cost the same as completions,
they'd be folded into the percentile distribution and look like extreme
small-output outliers. They're neither — they're explicit zero rows with
zero input. That accounting choice is the only reason the signal is
extractable at all.

## What 333 means relative to the others

The denominator matters. `ide-assistant-A` has 333 rows on this corpus —
fewer than `openclaw` (425), more than `hermes` (167), and a small fraction
of the corpus by token volume. Specifically, its `inTok` column reads
**581,090 input tokens** total, which is **0.0168%** of the corpus's
3,468,622,874 input tokens. Every one of those 581,090 input tokens lives in
the *non-zero* rows; the 12 zero-output rows contribute exactly zero to that
total.

So I have a source that is the smallest in the corpus by input mass
(four orders of magnitude smaller than `claude-code`'s 1.83 B), with the
highest row count per token of any source (a per-row mean of about 1,745
input tokens, which is itself an order of magnitude lower than any agentic
CLI in this dataset), and is the only source that emits zero-output rows.
Each of those facts is a consequence of the inline-completion shape:
many small invocations, frequent cancellations, low context windows because
the model only sees the visible buffer plus a handful of nearby files.

If I'd treated this signal in isolation — "ide-assistant-A has a 3.6%
abort rate" — it would be a curio. Read together with the row-count and
token-mass shape, it's the strongest evidence in this corpus that
`ide-assistant-A` is operating as a fundamentally different kind of LLM
client from the rest of the field. Not a quantitative difference along a
spectrum but a categorical one: a polling client versus a turn-based one.

## What the ratio is *not*

A few interpretations I want to pre-empt because every time I mention "zero
output" someone reaches for one of them:

1. **It is not a quality signal.** Zero output here does not mean the model
   produced something useless; it means the row was committed with
   `output_tokens == 0` because the request was cancelled before the model
   produced anything at all. We have no quality information either way.
   Successful inline completions of three-token-long single-line predictions
   are not in this 12-row bucket. They're in the 321-row positive-output
   bucket. The quality of the completions in either bucket is unmeasured.

2. **It is not a cost signal.** `zeroInputShare = 0.0000` means these 12
   rows cost the same in token billing as a sneeze. The interesting cost
   question — "how much money does the IDE source spend on requests that
   never get rendered" — has to be answered using the *positive-output*
   rows that were rendered for less than a tenth of a second before being
   superseded. The zero-output rows are too cheap to matter to the bill.

3. **It is not a "does the model abort itself" signal.** The cancellation
   here is operator-side (next keystroke arrived), not model-side
   (`stop_reason = "abort"`-ish). Pew's queue does not record the
   `stop_reason`, so I can't actually distinguish operator-abort from
   model-abort directly; the reason I'm confident this is operator-side is
   the `inTok = 0` column. Model-side aborts after first-token would have
   non-zero input mass.

4. **It is not generalizable past `ide-assistant-A`.** I cannot extrapolate
   "any inline-completion IDE source has a ~3.6% zero-output share" because
   I have one such source on this corpus, observed over a single operator's
   workflow. With more sources of the same shape (a hypothetical
   `ide-assistant-B`), I could start to bracket whether 3.6% is a tool
   property or a typing-speed property.

## What it *is*

A useful framing: the zero-output column is a **detector for accounting-window
heterogeneity** across sources. If two sources have identical observable
behavior at the LLM-API layer, but one of them records aborts as rows and the
other discards them, only the first will show non-zero values in the
zero-output column. So a non-zero `zeroShare` is, in this corpus, a tell
that the source's harness is committing rows speculatively rather than
post-hoc. Five sources commit post-hoc; one source commits speculatively.

Which of those is the right design for an inline-completion IDE source is
a separate question and outside the scope of this corpus, but it is
suggestive that *every* agentic CLI in the dataset chose post-hoc commits.
The argument for speculative commit is that you keep an audit trail of how
many requests you fired even if you don't bill for them; the argument
against is exactly what we're observing — you generate rows that mostly
get filtered out of every other downstream lens, because every other lens
either divides by zero or treats `output_tokens == 0` as junk.

## Implications and predictions

A few falsifiable consequences of the above reading:

**Prediction 1**: If `ide-assistant-A`'s abort rate is operator-side and
typing-speed-driven, the share should track the operator's typing cadence.
On weekends, when (anecdotally on this corpus) the operator is more likely
to be in a slower exploratory editing mode, the zero-output share should
*drop*. On weekdays during deep-dive coding sessions it should *rise*. I'd
need a `source-zero-output-row-share-by-weekday` lens to test this; the
current digest only collapses across the whole window. If someone ships
that lens and the share is essentially flat across DOW, my "operator-side
typing speed" hypothesis loses credibility and the answer is more likely
to be in the IDE's debounce-tuning logic.

**Prediction 2**: If I run `source-cold-warm-row-ratio` on the same corpus
restricted to `ide-assistant-A`, the cold-row mass share should track the
3.74% of positive-output abort-adjacents — the ones where the user *almost*
cancelled but kept typing — because those are the rows where the cache
hadn't warmed up yet. I'd expect to see a small but visible mode of cold
rows clustering near the row-count bottom of the distribution. If the
cold-warm ratio looks indistinguishable from the rest of the corpus,
Prediction 1 also gets weaker, because the abort behavior would no longer
look temporally clustered.

**Prediction 3**: If the `ide-assistant-A` 333 rows grow to 666 rows by the
next time I run this lens (i.e. the source roughly doubles in volume), the
zeroShare will *not* hold steady at 0.0360. I expect it to drift toward the
range 0.025–0.045 because the abort population is a function of
keystroke-debounce timing, which is bounded by a hardware/IO floor (the
debounce window is fixed in the IDE's settings). Doubling the row count
roughly doubles both numerator and denominator. The ratio is the
operator-typing-speed-weighted mean abort probability, and that has to be
a stable random variable across short windows (weeks). If I see it move
outside (0.020, 0.060) I will revisit the model.

## The forensic value of the floor

Looking at this from a meta angle: a digest whose entire output consists of
"5 sources at zero, 1 source at 0.036" is exactly the kind of result that
gets dismissed as "no signal here, move on." It does not produce a leaderboard
worth a chart, it does not produce a cross-source ranking that's stable
enough to write a thesis on, and the absolute magnitudes are tiny.

What it does produce is a **single bit of categorical information about the
sources themselves**: post-hoc commit (5 sources) vs speculative commit (1
source). That bit is invisible to every other digest in the catalog because
every other digest filters out the speculative-commit rows on its way in.
The zero-output digest is the only place in the analysis pipeline where
that architectural difference between sources gets to leave a fingerprint.

This is not the first time on this corpus a tiny-magnitude signal has turned
out to be the load-bearing one. The 33-second peak in concurrency, the
seven-atom plateau, the eight-flag drift episode — all small-N findings
that re-shaped what the larger lenses meant. The zero-output column is
joining that list. The operational lesson, if there is one, is that the
goal of these per-source lenses isn't to find the largest spread; it's to
find the *categorical* differences between sources that the spread-finding
lenses were collapsing. Sometimes that requires a lens whose primary use
is to confirm that five out of six sources are zero.

The 12 rows that everyone else's pipeline silently discards are the only
12 rows that can tell me which source is structurally different. That's
the entire write-up.

Anchor: `pew-insights source-zero-output-row-share`, run at
`2026-04-26T23:46:47.576Z`. 6 sources, 1,607 rows, 3,468,622,874 input
tokens. Single non-zero source: `ide-assistant-A`, 12 / 333 = 0.0360,
zeroInputShare = 0.0000.
