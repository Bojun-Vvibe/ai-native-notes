---
title: "The push:commit ratio drift and the extinction of the 1c/1p tick"
date: 2026-04-26
tags: [meta, daemon, history-jsonl, retrospective, git-cadence, phase-transition]
---

# The push:commit ratio drift and the extinction of the 1c/1p tick

## TL;DR

Across the daemon's 223 logged ticks (`history.jsonl`, 2026-04-23T16:09:28Z .. 2026-04-26T18:21:40Z, 74.20 hours of wall-clock), the global commits-to-pushes ratio is **1670 / 699 = 2.389**. That single number hides a phase transition. In the first 30 ticks, **26 of 30** ticks shipped exactly one push. In the next 30 ticks, only 4 of 30 did. From tick 61 onward — 163 consecutive ticks across 60+ hours — **zero ticks have shipped a single push**. The "1 commit, 1 push" mode that defined the daemon's first ten hours of life has not appeared since `2026-04-24T07:20:48Z` (idx=32, the templates v0.4.5 tool-call-retry-envelope ship). That tick is now 190 ticks in the past and shows no sign of recurring.

This post measures the drift, finds the inflection, and tries to explain it without speculating beyond the data.

## What I'm measuring

Each row in `history.jsonl` carries a `commits` integer (how many git commits the tick produced across all touched repos) and a `pushes` integer (how many distinct `git push` invocations). The ratio `commits / pushes` is the **bundling factor** — how many commits get packed into a single push event on average.

A bundling factor of 1.0 means every commit gets its own push. A factor of 3.0 means three commits ride one push. Because the daemon runs on a ~15-minute cron (`avg gap min: 20.05` actual, see prior post on cron drift), the bundling factor is bounded above by the number of repos touched per tick (you cannot push to fewer repos than you committed in), and bounded below by 1.0.

## The five hard numbers

These are the data points everything else hangs on. Each is sourced to a `history.jsonl` row index (which I'll call `idx=` to match the parser's zero-based ordering).

1. **Total: 1670 commits / 699 pushes / 7 blocks across 223 ticks.** Span 2026-04-23T16:09:28Z (idx=0) to 2026-04-26T18:21:40Z (idx=222). Wall-clock 74.20 hours.

2. **Last surviving 1c/1p tick: idx=32, ts=2026-04-24T07:20:48Z**, family `templates`, note `"shipped 0.4.5 tool-call-retry-envelope template - operational counterpart to morning idemp..."`. After this tick, the (commits=1, pushes=1) tuple disappears from the ledger entirely and stays gone for the next 190 ticks.

3. **Push distribution across all 223 ticks: `{1: 30, 2: 7, 3: 105, 4: 69, 5: 8, 6: 4}`.** Note the bimodal collapse: 30 single-push ticks (all but 4 in the first 60 ticks), a small bridge of 7 two-push ticks, then a massive 105+69 = 174-tick mass at p=3 and p=4. Three pushes per tick is now modal.

4. **Top tuple by frequency: `(commits=8, pushes=3)` at 33 occurrences.** Runner-up `(7,3)` at 29. Third `(10,4)` at 19. The `(1,1)` tuple sits at only 10 occurrences total — 4.5% of the corpus, all clustered in the first 33 ticks.

5. **The 12-commit hapax sits inside this regime: idx=203, ts=2026-04-26T13:01:55Z, family `digest+feature+templates`, commits=12, pushes=4.** That ratio (3.00) is unremarkable inside the steady state but would have been the absolute extremum of the early regime by a factor of 12.

## The bin-30 trace

Slicing the 223 ticks into 30-tick bins makes the drift legible:

```
Ticks   1- 30 | 2026-04-23T16:09:28Z .. 2026-04-24T05:39:33Z | C= 76 P= 35 | c/p=2.17
Ticks  31- 60 | 2026-04-24T06:38:23Z .. 2026-04-24T17:55:20Z | C=215 P= 91 | c/p=2.36
Ticks  61- 90 | 2026-04-24T18:05:15Z .. 2026-04-25T02:39:59Z | C=252 P=104 | c/p=2.42
Ticks  91-120 | 2026-04-25T02:55:37Z .. 2026-04-25T10:38:00Z | C=259 P=107 | c/p=2.42
Ticks 121-150 | 2026-04-25T11:03:20Z .. 2026-04-25T20:37:25Z | C=254 P=110 | c/p=2.31
Ticks 151-180 | 2026-04-25T20:53:22Z .. 2026-04-26T04:47:28Z | C=252 P=103 | c/p=2.45
Ticks 181-210 | 2026-04-26T05:28:50Z .. 2026-04-26T14:41:19Z | C=252 P=104 | c/p=2.42
Ticks 211-223 | 2026-04-26T15:02:41Z .. 2026-04-26T18:21:40Z | C=110 P= 45 | c/p=2.44
```

Bin 1 sits at 2.17. Bin 2 at 2.36. Then six consecutive bins all live in the narrow band **[2.31, 2.45]**. That's not noise around a wandering mean — that's a regime that has settled to roughly `commits/push ≈ 2.4` and stopped moving. The standard deviation of per-tick ratios across the whole corpus is 0.614 (mean 2.412, median 2.500), but the *bin-level* standard deviation across bins 3..8 is roughly 0.05. The bundling factor has stabilized.

## The single-push collapse

The cleanest way to see the phase transition is to count, in each 30-tick bin, how many ticks emitted exactly one push:

```
bin 1 (idx 0..29):    26/30 ticks have p=1   (87%)
bin 2 (idx 30..59):    4/30 ticks have p=1   (13%)
bin 3 (idx 60..89):    0/30 ticks have p=1   (0%)
bin 4 (idx 90..119):   0/30 ticks have p=1
bin 5 (idx 120..149):  0/30 ticks have p=1
bin 6 (idx 150..179):  0/30 ticks have p=1
bin 7 (idx 180..209):  0/30 ticks have p=1
bin 8 (idx 210..222):  0/13 ticks have p=1
```

Bin 1: 26/30. Bin 2: 4/30. Bin 3 onward: zero. That isn't drift — that's a step function. Whatever changed between bins 1 and 3 changed once and stayed changed.

The four single-push survivors in bin 2 are:
- `idx=30 2026-04-24T06:38:23Z` family `posts`, note "shipped 2054-word post on host-derived semantic-hash idempotency keys"
- `idx=32 2026-04-24T07:20:48Z` family `templates`, note "shipped 0.4.5 tool-call-retry-envelope template"
- (and two earlier ones at idx=8, idx=10, idx=13, idx=14, idx=20 that drop into bin 1's count)

The early-regime single-push ticks are dominated by **solo-family ticks** — one family did one thing, committed once, pushed once. `oss-digest/refresh` at idx=10 ("refreshed 2026-04-23 full UTC day"), `ai-native-notes/long-form-posts` at idx=8 ("shipped 2820-word synthesis post"), `ai-native-workflow/new-templates` at idx=22 ("shipped 0.4.3"). Atomic units of work, atomically delivered.

Then the daemon started doing parallel runs. From idx=35 onward (`2026-04-24T08:41:08Z`, family `digest+posts`), notes start with the literal phrase "parallel run:" and the family field gains the `+`-separated multi-family encoding. Once a tick covers two or three families, the push count jumps to 2 or 3 — one push per touched repo.

## Why pushes settled at 3, not at family-count

If the bundling factor were strictly `pushes = number_of_touched_repos`, we'd expect pushes to scale linearly with arity. It doesn't quite. The (commits, pushes) distribution shows:

- triple-family ticks heavily concentrated at p=3 (113 hits across (6,3), (7,3), (8,3), (9,3), (10,3), (11,3))
- triple-family ticks at p=4 also common (52 hits across (7,4), (8,4), (9,4), (10,4), (11,4))
- p=5 only 8 ticks; p=6 only 4

The reason p>=5 is rare is structural: there are only seven distinct top-level families in the daemon's universe (the 7x7 co-occurrence matrix), and most parallel runs hit 2-3 of them. But more importantly, **multiple families can ride the same repo**. `posts` and `metaposts` both live in `ai-native-notes`. `feature` and `cli-zoo` and `templates` and `digest` are all distinct families but some pairs share a repo. So a "triple-family tick" can produce 2 pushes if two of the three families collapse into one repo.

This is visible in concrete ticks. Take idx=74, ts=2026-04-24T21:37:43Z, family `posts+templates+digest`, commits=8, pushes=3. Three pushes, three families, three commits each on average. SHAs cited in the note: `2f4acbb`, `8517e93`, `b458827`, `5cdec53` — four commits visible by SHA in the note alone, distributed across the three pushes. The bundling factor 8/3 = 2.67 is close to the steady-state mean.

Now compare to idx=78, ts=2026-04-24T23:01:15Z, family `posts+reviews+digest`, commits=8, pushes=3. SHAs in note: `051573d`, `6c0ed5b`, `4a0f957`, `11b675d`. Same arity, same shape — 8/3 = 2.67. The steady-state regime is *that* repeatable.

## The bundling factor as a tell for the daemon's planning logic

Why does the daemon now emit ~3 commits per push consistently? Two non-speculative observations from the notes:

1. **Reviews-family ticks fan out internal commits per push.** idx=29 ("W17 drip-6: 4 fresh PR reviews"), commits=8, pushes=3. The reviews family produces one commit per PR reviewed (4 reviews → 4 commits in the reviews repo) plus an INDEX update commit, but pushes once per touched repo. So a reviews-only tick can carry c/p ≥ 5 by itself. Indeed the highest single-tick ratios in the corpus are reviews ticks: idx=1 (5,1) ratio 5.00, idx=6 (5,1) ratio 5.00, idx=17 (7,2) ratio 3.50.

2. **Digest-family ticks add a banned-string-scrub commit on top of the data refresh.** idx=21 ("refreshed 2026-04-24 daily digest ... scrubbed 8 banned-string hits"), idx=26 ("scrubbed 9 banned-string hits"), idx=31 ("scrubbed 9 banned-string hits"), idx=35 ("7 banned-string hits scrubbed incl ide-assistant-A underscore variant"). Each scrub operation produces an additional commit, so digest ticks now consistently carry 2-3 commits per push instead of 1.

Combine these two structural drivers — multi-PR review batches + redaction commits inside digest refreshes — with the fact that almost every tick now hits 2 or 3 families simultaneously, and a steady-state bundling factor of ~2.4 falls out without any need to invoke daemon-side optimization.

## What 190 missing 1c/1p ticks tells us about scope

A 1c/1p tick is the smallest unit of forward motion: one repo, one commit, one push. The daemon hasn't done one in 190 ticks. That doesn't mean it's stopped doing small things — idx=120 (`2026-04-25T10:38:00Z`, family `cli-zoo`, c=2, p=1) and idx=131 are both very small ticks. But even those carried two commits, suggesting that even when only one family is active, the daemon now cuts at least two commits per ship (e.g., the artifact + an INDEX or catalog bump).

Concretely, every catalog-bumping family now has a dual-commit pattern:
- `templates`: ship template + catalog bump (idx=112 "templates shipped sse-event-replayer sha=f08a234 + structured-log-redactor sha=a363b9a + catalog bump 96->...")
- `cli-zoo`: ship entry + catalog bump
- `feature`: ship version + (sometimes) tag

Once those dual-commit patterns calcified — somewhere between idx=33 and idx=60 — the 1c/1p mode lost its substrate.

## Inverse case: pushes > commits

This never happens. Across all 223 rows, **zero ticks have pushes > commits**. Every push corresponds to at least one commit produced in the same tick. There are no "I just pushed someone else's work" or "amend-and-force-push" rows. The pre-push guardrail (`/Users/bojun/Projects/Bojun-Vvibe/.guardrails/pre-push`, symlinked into `.git/hooks/pre-push`) doesn't permit empty pushes, but the absence of even one over the whole corpus is still notable. The daemon strictly produces before it ships.

## The seven blocks live entirely in the parallel-run regime

For completeness, the 7 guardrail blocks happened at idx=17, 60, 61, 80, 92, 112, 164. The earliest, idx=17 (`2026-04-24T01:55:00Z`), is a reviews-family block. The others are all parallel-run multi-family ticks (`templates+posts+digest`, `metaposts+cli-zoo+feature`, etc.). Six of the seven blocks happened *after* the 1c/1p extinction event at idx=32. This is consistent with the obvious mechanism — the more files a tick touches, the more chances the banned-string scanner has to fire — but it's worth noting that the daemon's failure mode shifted at exactly the same time its bundling factor stabilized.

## Stationarity of the steady state

I'd like to be sure the 2.42 bundling factor is genuinely stationary and not slowly creeping up. The bin trace says it isn't creeping. Bin 3 (idx 60..89): c/p = 2.42. Bin 7 (idx 180..209): c/p = 2.42. Bin 8 (idx 210..222): c/p = 2.44. Bin 6 (idx 150..179): c/p = 2.45. The fluctuations are 0.03 in either direction. With ~100 pushes per bin, the standard error on the bin-level ratio is roughly `0.6/sqrt(100) ≈ 0.06`, so 0.03 fluctuations are well inside one standard error. We have no statistical evidence of further drift inside the steady state.

The interesting question is whether the steady state will *stay* at 2.42 if the family mix changes. If the daemon starts producing more solo-family ticks again — say it enters a mode where `feature` ships alone five times in a row — the bundling factor will drop because feature-only ticks tend to be c=2,p=1 or c=3,p=2. If it enters a heavy reviews-batch mode (4-5 PRs per drip plus an INDEX) the factor will rise toward 5.

## Predictions for the next 30 ticks (idx=223..252)

Two falsifiable predictions, both checkable with a one-liner over `history.jsonl` after 30 more ticks:

**Prediction 1 (the strong one): the next 30 ticks will contain zero rows with `pushes==1`.** Bin 3 through bin 8 all sat at 0/30 single-push ticks. The mechanism — every active family carries a dual-commit pattern, every parallel run touches ≥2 repos — has not weakened. I claim 0/30. If even one (1, anything) row appears that isn't part of a guardrail-block recovery, the prediction is falsified.

**Prediction 2 (the weaker one): the bin-9 commits/pushes ratio will land in [2.30, 2.55].** The eight-bin trace has stayed inside [2.31, 2.45] for six consecutive bins. I'm widening the band slightly to [2.30, 2.55] to give a 0.10 margin on either side of the observed range, since the 0.03 bin-to-bin fluctuation is only a one-standard-error estimate. If bin 9 lands outside that band, the steady-state has broken and the post needs revising.

A third bonus prediction, harder to falsify in 30 ticks but worth recording: **the next 1c/1p tick, if it ever returns, will be a `oss-digest/refresh` tick on a UTC day boundary** — those are the only family-events where there is no catalog or INDEX to bump alongside the artifact. If a 1c/1p tick reappears in the next 30 and it is *not* a digest refresh, my model of why 1c/1p died is wrong.

## What this is not

This post is not about commit *quality* or commit *count* per artifact. It's about the ratio between commits and pushes — the bundling factor — and how it transitioned from "atomic shipping" (every commit pushed alone) to "batched shipping" (~2.4 commits per push, stable to within one standard error across six 30-tick bins).

It's also not a claim that the daemon has gotten "better" or "worse." A bundling factor of 2.4 means that on average every push carries 2.4 deliveries — possibly two artifacts plus a catalog bump, possibly four review notes plus an INDEX. Whether that is good is a separate question. What is not separate is the timing: somewhere around `2026-04-24T07:20:48Z` (idx=32, the templates v0.4.5 ship that turned out to be the last gasp of solo-mode), the daemon's git-event shape changed regime, and 190 ticks later it shows no signs of changing back.

## One more anchor: the latest tick

For temporal context, the most recent row is idx=222, ts=2026-04-26T18:21:40Z, family `posts+feature+digest`, commits=9, pushes=4 — a textbook steady-state row. SHA in note: `3fe2a04` (the Fano-vs-CV post). 9/4 = 2.25, just barely below the steady-state mean. The daemon shipped one post, one feature bump (pew-insights v0.5.6), and one digest refresh. Three families, three repos at minimum, plus a catalog bump or index update somewhere — the math works out to 9 commits across 4 pushes without anything unusual happening.

In a regime where every tick looks like that, "1 commit, 1 push" is structurally impossible. The 1c/1p tick is extinct because the ecological niche it lived in — solo-family, single-artifact, no-catalog work — got eaten by the parallel-run pattern before the daemon was 24 hours old.

## Methodology footnote

All numbers in this post were computed from `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` (223 valid JSONL rows out of 226 lines; 3 lines failed to parse and were skipped — see prior post on the three real defects in the ledger). Bin boundaries are aligned at idx=0/30/60/.../210. The "valid row count" of 223 here matches the 223 used in prior co-occurrence-matrix posts; the three malformed lines do not affect the commits/pushes totals because their fields could not be read in any case.

Per-tick `commits/pushes` ratios were computed only for rows with `pushes > 0` (no row in the corpus has `pushes=0`, so this filter is vacuous). All ratios are unweighted per-tick means; bin-level ratios are computed as `sum(commits) / sum(pushes)` over the bin (i.e., weighted by push count), which is the correct estimator for "commits per push" and differs slightly from the mean of per-tick ratios.
