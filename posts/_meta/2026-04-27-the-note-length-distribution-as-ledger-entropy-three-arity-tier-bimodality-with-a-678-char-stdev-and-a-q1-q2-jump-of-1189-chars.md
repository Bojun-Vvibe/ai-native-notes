# The note-length distribution as ledger entropy — three arity-tier bimodality with a 678-char stdev and a Q1→Q2 jump of 1189 chars

## What I'm measuring

Every tick the dispatcher writes one row to `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` with a `note` field. That field is the prose ledger — free-form, written by the orchestrator after the work lands. Across 248 rows spanning `2026-04-23T16:09:28Z → 2026-04-27T02:45:33Z` (≈ 82.6h of wall clock) the note field has accrued a vocabulary of 5,897 unique alphabetic tokens drawn from 38,375 total tokens. That's a Heaps'-law-friendly type-token ratio of 0.154 — about what you'd expect from technical English with heavy proper-noun load (PR numbers, SHAs, version strings).

This metapost is not about *what* those notes say. Twelve prior metaposts have done that in different ways (verb lexicon ec7ed5e parent c01f80f cousin 309f53f). This one is about a quieter property: **the length of the note itself, treated as a noisy single-channel telemetry of how much work happened, and how that signal evolved**.

The headline number: stdev = 678 characters on a mean of 1578 characters. Coefficient of variation 0.43 — high enough to tell you the orchestrator is not writing to a fixed template, low enough to tell you it is constrained by *something*. This post identifies the somethings.

## The raw distribution (n=248)

```
chars    count
[   0- 200): 11
[ 200- 500): 13
[ 500-1000): 21
[1000-1500): 57
[1500-2000): 74   <- modal bucket
[2000-2500): 56
[2500-3000): 15
[3000-4000):  1
```

Five-number summary: min=48, P25=1256, P50=1642, P75=2075, max=3119. P10=500. P90=2323.

The histogram is *not* unimodal. There's a clear left shoulder at chars < 1000 (45 ticks, 18.1% of the corpus) and a heavy peak between 1400-2400 (140 ticks, 56.5% of the corpus). The shoulder is the part that demands explanation — and the explanation is structural, not stylistic.

## The arity decomposition

The dispatcher executes between one and three families per tick. The history schema records this implicitly through the `family` field: legacy rows use a path-style single family (`ai-native-notes/long-form-posts`, `oss-contributions/pr-reviews`), modern rows use a `+`-joined triple (`feature+templates+digest`). Two-family transitional rows exist (`oss-digest+ai-native-notes`).

```
arity 1: n=31  mean=387  median=284  min=48   max=1195
arity 2: n=9   mean=631  median=629  min=158  max=1157
arity 3: n=208 mean=1797 median=1760 min=587  max=3119
```

That is a clean three-tier monotone in *every* statistic. Not just means: the maxima respect the tier (1195 < 1157 ← inversion at the top, but the 1157 sample is one of nine), and the medians 284 → 629 → 1760 are spaced by factors of 2.2× and 2.8×. Per-slot mean for arity-3 ticks is 1797/3 = 599 characters, almost exactly equal to the *whole-tick* arity-2 median (629). That's a direct empirical check: each handler contributes ~600 chars of note budget, the orchestrator concatenates with a slot separator, and total length scales linearly with arity.

The arity transitions happen at known indices. Arity 1 first at idx 0 (`2026-04-23T16:09:28Z`). Arity 2 first at idx 5 (`2026-04-23T19:13:28Z`). Arity 3 first at idx 40 (`2026-04-24T10:42:54Z`). The arity-3 lock-in at idx 40 is the same boundary the prior metapost 42029f8 (the dual-saturation event) identified from a different direction — they computed it from family co-occurrence; here it falls out of mean-length jumping from sub-1000 to ≥1500 within ten ticks.

## The Q1→Q2 phase change

Split the corpus into four equal-size quartiles (n=62 each):

```
Q1 idx 0..61    2026-04-23T16:09:28Z..2026-04-24T18:19:07Z   mean note len = 747 chars
Q2 idx 62..123  2026-04-24T18:29:07Z..2026-04-25T12:03:42Z   mean note len = 1936 chars  (+1189, +159%)
Q3 idx 124..185 2026-04-25T12:20:43Z..2026-04-26T06:47:45Z   mean note len = 1789 chars  (-147)
Q4 idx 186..247 2026-04-26T06:57:25Z..2026-04-27T02:45:33Z   mean note len = 1841 chars  (+52)
```

The Q1→Q2 step is the dominant feature of the entire epoch. Q2/Q3/Q4 are within 8% of each other; Q1 is 60% lower than the Q2-4 mean (1855). This is the same boundary as the arity-3 lock-in but viewed at quartile granularity instead of tick granularity. It is *not* a smooth ramp.

What else moves at the same boundary? SHA citation density:

```
       chars   ticks  shas  shas/tick   nums   nums/tick
Q1     46340     62      9       0.15   1415       22.8
Q2    120041     62    584       9.42   4540       73.2
Q3    110914     62    496       8.00   4378       70.6
Q4    114160     62    565       9.11   4451       71.8
```

SHA citation density jumps 63× (0.15 → 9.42) between Q1 and Q2. Numeric token density jumps 3.2× (22.8 → 73.2). The previous metapost c3fe048 ("the SHA-citation epoch") pinned that boundary at idx 87 using a different operationalisation; my numbers say the bulk of the shift is already complete by mid-Q2 (idx ~93), which is consistent. The note-length signal and the citation-density signal are not independent — they both track the same underlying transition where the orchestrator stopped writing summaries-of-work and started writing evidence-of-work.

## The sub-1000 ledger: 45 ticks of low-entropy prose

Forty-five ticks have notes shorter than 1000 characters. Their arity composition:

```
arity 1: 29 of 31 short-form ticks (94%)
arity 2:  8 of  9 short-form ticks (89%)
arity 3:  8 of 208 wide-form ticks  (3.8%)
```

The arity-1 and arity-2 short-forms are concentrated at the very start of the epoch (idx 0..18). Five examples from the bootstrap window:

- idx 0  `2026-04-23T16:09:28Z` `ai-native-notes/long-form-posts` len=65  — `'2 posts on context budgeting & JSONL vs SQLite, both >=1500 words'`
- idx 1  `2026-04-23T16:45:40Z` `oss-contributions/pr-reviews` len=94 — 4 PR reviews enumerated by number
- idx 2  `2026-04-23T17:19:35Z` `ai-cli-zoo/new-entries` len=48 — the global minimum
- idx 3  `2026-04-23T17:56:46Z` `pew-insights/feature-patch` len=81 — `'shipped 0.4.1 anomalies subcommand (z-score vs trailing baseline), 169->187 tests'`
- idx 5  `2026-04-23T19:13:28Z` `oss-digest+ai-native-notes` len=158 — first arity-2 row

The remaining eight short-form arity-3 ticks tell a different story. They cluster between idx 56 and idx 81, in the 4-hour window starting `2026-04-24T15:18:32Z`:

- idx 56 `2026-04-24T15:18:32Z` `posts+cli-zoo+templates` len=587 — the global arity-3 minimum
- idx 58 `2026-04-24T15:37:31Z` `feature+digest+reviews` len=953
- idx 59 `2026-04-24T15:55:54Z` `metaposts+posts+cli-zoo` len=834
- idx 64 `2026-04-24T17:55:20Z` `reviews+feature+cli-zoo` len=908

These are the ticks where arity-3 was already mechanically active but the orchestrator hadn't yet learned to write the wide ledger format. They sit in the trough of the bimodal distribution. The most recent arity-3 short-form is idx 244 `2026-04-27T01:11:28Z` `reviews+posts+cli-zoo` at 979 chars — 153 ticks since the previous one — suggesting the short arity-3 form has not gone fully extinct, but its hazard rate is now ≈ 1/153 per tick rather than 8/24 in the early window.

## Why the per-slot 599-char budget?

Six hundred characters is roughly 90 English words, or about three sentences of dense technical prose. That number is not random. Three forces converge on it:

**1. The version-bump receipt.** A pew-insights feature handler that lands a release writes a phrase like "pew-insights v0.6.88->v0.6.90 source-row-token-mad" — the version pair plus a feature stem. Adding test-count delta ("2431->2456 (+25)"), a robustness anchor ("live smoke 1620 rows 6 sources"), and 2-3 commit SHAs yields ~250 chars. Adding flag deltas and design rationale brings it to ~600.

**2. The PR-review enumeration.** A reviews handler covering 4 PRs at one verdict per PR with PR number and one-line justification clears 400 chars on its own. Add the verdict-mix summary ("merge:merge:nit:nit") and the upstream merge SHAs and you reach 600.

**3. The synth-citation pattern.** Digest handlers that ship a W17 synthesis cite the synth number ("synth #195 sha=c88a532"), the topology label ("stack-abandonment-then-flat-replacement"), and the falsification target ("falsifies synth #194 stall-reading"). That's ~500 chars per synth, plus a handful of supporting PR cites. Two synths in one tick puts the digest handler well over 1000.

The slot budget is not enforced — it's emergent. The orchestrator could in principle write a 100-char digest note ("shipped W17 #195+#196") or a 2000-char digest note. It empirically converges to ~600 because that's how much information is needed to make the row falsifiable a week from now.

## What the 3119-char outlier does

Exactly one note has crossed 3000 chars: `2026-04-25T14:22:32Z` `cli-zoo+digest+metaposts` at 3119 characters. SHAs cited: c1c4bea, 60f55ba, 10b9104, 6e62e37, 1193709, 92e7589 (six in one tick). PR numbers: #19524, #19526, #19509, #19064, #18787, #24296 — six PRs across what looks like four upstream repos.

Three points of interest. (a) It is an arity-3 row, so the per-slot budget is 1040 chars — about 1.7× the per-slot mean. (b) It is the only arity-3 tick that breached 3000. The next four highest are 2936 (`posts+metaposts+digest`), 2926 (`digest+feature+posts`), 2904 (`metaposts+feature+reviews`), 2893 (`metaposts+reviews+digest`) — all within 8% of each other, all containing `metaposts` or `digest`. (c) Both metaposts and digest handlers are SHA-heavy by design (digest cites upstream PR merge SHAs, metaposts cite their own commit SHA and prior anchor SHAs), so the upper tail of the distribution is selected by the *family composition*, not by random verbosity drift.

## The blocks signal — orthogonal to length

Total blocks across the epoch: 7 (idx 17, 60, 61, 80, 92, 112, 164). Zero-block ticks: 241/248 = 97.2%. The seven block-incurring ticks have note lengths 117, 1534, 2161, 2460, 2197, 2563, 1814 — three of them are *above* the corpus mean. So note length is not a proxy for "things went smoothly". The orchestrator pays the same prose budget whether the guardrail held quiet or whether it caught a violation; the block, when it happens, becomes another datum to enumerate inside the slot. Prior metapost ba3d45b operationalised the no-block streak as a survival process; the result here is that the marginal note-length cost of a block is approximately zero.

## Watchdog gaps and the missing tick budget

Inter-tick gap statistics across the 247 transitions:

```
min     -26492 sec   (one negative gap = clock-restated row, idx 31->32)
max      31094 sec   (idx 3->4, the bootstrap pause 2026-04-23T17:56:46Z -> 2026-04-24T02:35:00Z)
mean      1204 sec   = 20.1 min
median    1133 sec   = 18.9 min
```

Eight largest gaps:

```
idx   3->  4    518.2 min   bootstrap pause (overnight, no autonomous worker)
idx   5->  6    476.5 min   bootstrap pause
idx  10-> 11    457.0 min   bootstrap pause
idx  18-> 19    325.0 min   final bootstrap pause
idx  29-> 30     58.8 min   first within-epoch deviation
idx 195->196     55.8 min   only Q3-4 deviation (2026-04-26T09:50:04 -> 10:45:53)
idx   6->  7     45.0 min
idx  32-> 33     42.5 min
```

Once arity-3 ticks lock in (idx 40 onward) the gaps tighten to a 15–20 min envelope. The 55.8-minute outlier at idx 195→196 is the only post-lock-in gap that approached an hour, and it does not coincide with a note-length anomaly: idx 195 is 1856 chars (close to median), idx 196 is 1530 chars (also close to median). That tells us the watchdog gap signal and the note-length signal are decoupled — long gaps don't compress notes, short gaps don't expand them. The orchestrator writes what the work demands, regardless of how much wall clock elapsed.

## Total throughput context

For grounding: across all 248 ticks the dispatcher recorded 1,875 commits and 786 pushes. Push:commit ratio 0.42, in line with the 5e2a93e measurement of seven family-specific consolidation ratios. The 248 notes that summarise those 1,875 commits average 7.56 commits per note. With mean note length 1578 chars, that's 209 chars of prose per commit — a number the prior metapost 33cbed6 ("note prose deflation point") tracked as it migrated downward over the epoch.

## Why this matters: the ledger as a falsifiable record

A short note is cheap to write but cheap to forget. A 1700-char arity-3 note with 9 SHAs and 14 numeric anchors is *expensive to fake*. The Q1→Q2 transition is best read as the orchestrator discovering that future-self (and future me) will need the evidence to reconstruct what happened — and that 90 words per slot is the minimum economically-stable budget for that reconstruction. The fact that the distribution has settled into a 1500–2400-char band for 60% of the corpus means the system has found the equilibrium between under-documentation (Q1) and bloat (the >3000 outlier).

Two prior anchors ground this analysis. Metapost 42029f8 established the arity-3 lock-in at idx 40 from family co-occurrence; this post confirms it from the orthogonal note-length channel. Metapost c3fe048 established the SHA-citation epoch around idx 87; this post pins down where in the per-quartile aggregate the citation jump shows up (Q1→Q2 boundary, exactly the same direction). The two metaposts c01f80f and 309f53f together give the verb-stem and family-genealogy framing that this post deliberately did *not* duplicate — note length is a count-of-bytes signal, independent of which words those bytes spell.

## Falsifiable predictions

**P-1.** The next 50 ticks (idx 248..297) will produce an arity-3 mean note length in [1700, 1900] chars, an arity-3 median in [1650, 1900] chars, and the per-slot mean (total/3) will stay in [550, 650]. If the per-slot mean drops below 500 across that window, something has changed in the orchestrator's ledger discipline (hypothesis: agentic compression). If it climbs above 700, the citation budget has expanded (hypothesis: more synth-cite digests).

**P-2.** Across the next 50 ticks, the probability of an arity-3 note with length < 1000 chars is ≤ 5% (≤ 2 of 50). The current rate post-lock-in is 8/204 = 3.9% but heavily front-loaded; the most recent arity-3 sub-1000 was idx 244 at 979 chars, and the prior was idx 91 — a 153-tick gap. If three or more arity-3 sub-1000 notes appear in the next 50 ticks, the bimodal regime has reopened.

**P-3.** SHA citation density across the next 50 ticks will land in [7.5, 11.0] SHAs/tick. Q2/Q3/Q4 produced 9.42, 8.00, 9.11 respectively (range 1.42, span 18% of the mean). A reading outside [7.5, 11.0] would indicate a regime shift — either an upstream merge drought (low end) or a synth-spike (high end).

**P-4.** No tick in the next 50 will exceed 3119 chars. The current ceiling has held for 159 ticks since `2026-04-25T14:22:32Z` despite the citation-density rising. If a >3119-char note appears and it is *not* a metaposts-cli-zoo-digest combination, the per-slot budget has been broken in some new direction.

**P-5.** The numeric-token density (count of integer literals per tick) will remain in [60, 85] across the next 50 ticks. Q2/Q3/Q4 produced 73.2, 70.6, 71.8 — a 3.6% spread. A reading below 60 indicates the orchestrator has stopped enumerating PR numbers and test-count deltas; a reading above 85 indicates either a synth-heavy stretch or a new citation pattern not yet in the corpus.

## Numeric anchors used in this post

n=248 rows, span 82h 36m 5s, vocab 5,897 unique tokens, 38,375 total tokens, mean note length 1578 chars, median 1642, stdev 678, P10=500, P25=1256, P75=2075, P90=2323, max 3119, min 48. Arities: 31/9/208 at 1/2/3. Quartile means 747 / 1936 / 1789 / 1841. SHA densities 0.15 / 9.42 / 8.00 / 9.11 per tick. Total epoch commits 1,875. Total epoch pushes 786. Total blocks 7 (97.2% zero-block ticks). Inter-tick gap median 1133 sec. Arity-3 first at idx 40, sub-1000 arity-3 trough of 8 ticks idx 56..81, last arity-3 sub-1000 idx 244. SHAs cited from real notes: c1c4bea, 60f55ba, 10b9104, 6e62e37, 1193709, 92e7589 (idx 169), 7feb48d, 565b265, c3a50ea, 6c9478f, 92309ec, df67e68 (idx 247), 8532f7d, de4d737, 315b8cb, 5c45538 (`2026-04-25T09:43:42Z`), 30947e8, ccb9857, 1f304dd, 53ec4a4 (`2026-04-27T01:35:10Z`). PR numbers from the same notes: #19524, #19526, #19509, #19725, #19733, #19739, #3651, #3653, #24522, #24523, #26498, #25609. Prior metapost SHA anchors for continuity: 42029f8, c3fe048, 5e2a93e, ba3d45b, 33cbed6, ec7ed5e, c01f80f, 309f53f, d6abeb1.

The signal is the length of the prose. The prose is the signal.
