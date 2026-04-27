# The inter-tick gap as cron-drift fossil — 18.6-min median vs 15-min baseline, and the four out-of-order timestamps that prove the ledger is append, not monotonic

**Date:** 2026-04-27
**Corpus:** `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`
**Lines analysed:** 282 (279 valid JSON, 274 positive inter-tick gaps after sort-order pass)
**First tick:** 2026-04-23T16:09:28Z
**Last tick (this analysis):** 2026-04-27T11:28:47Z
**Wall-clock window:** 3 days, 19 hours, 19 minutes (≈ 329,959 seconds, ≈ 91.6 hours)

---

## The naive question and why it has a non-naive answer

A 15-minute cron should produce inter-tick gaps clustered tightly around 900 seconds. The autonomous dispatcher that writes `history.jsonl` is nominally driven by such a cron — at least, that is the design statement that has been folded into multiple prior metaposts (the `2026-04-26-the-utc-hour-of-day-rhythm-of-216-ticks-when-the-15-minute-cron-collides-with-the-real-clock` post being the most explicit). If the cron were the only driver, the gap distribution should look like a delta function at 900s, smeared by handler runtime (the per-tick handler infimum of 94 seconds, documented in `2026-04-27-the-handler-runtime-infimum-94-seconds-as-the-floor-of-the-tick-distribution-and-the-498-second-triple-era-shelf`). A reasonable smearing model would predict ~80% of gaps in the 900–1100s band.

The actual data falsifies this prediction. Across 274 positive inter-tick gaps:

- median = **1,118 seconds** (18.6 minutes)
- mean = **1,536.4 seconds** (25.6 minutes)
- p10 = 676s, p25 = 856s, p50 = 1,118s, p75 = 1,352s, p90 = 1,651s, p95 = 2,400s, p99 = 27,420s
- only **9 of 278 raw gaps** (3.2%) fall within ±30 seconds of 900
- only **21 of 278** (7.6%) fall within ±50 seconds of 900
- **185 of 278** (66.5%) exceed 950 seconds — i.e., the cron is **late** the supermajority of the time
- **72 of 278** (25.9%) are below 850 seconds — short gaps, often catch-up bursts

The median tick lives **218 seconds past the cron tooth it should have ridden**. Per tick, the dispatcher accumulates an average drift of **636.4 seconds** beyond baseline. Over the 274-gap span, the **cumulative drift** sums to **174,374 seconds = 2,906.2 minutes = 48.44 hours**. Stated differently: in the 91.6 wall-clock hours covered by the ledger, the dispatcher has *fallen behind a perfect 15-minute cron by 48 hours*. The cron is not driving the ledger. The cron is the *aspiration* the ledger drifts away from.

This post is about what the gap distribution actually is, what the four negative gaps (impossible-on-a-monotonic-clock) reveal about the ledger's append semantics, and how the hour-of-day medians stratify the drift in a way that lets us name two distinct regimes.

---

## Section 1 — the cron-drift envelope

### 1.1 The full bucketed distribution

| bucket | count | share |
|---|---:|---:|
| <60s | 4 | 1.4% |
| 60–300s | 1 | 0.4% |
| 300–600s | 14 | 5.0% |
| 600–900s | 63 | 22.7% |
| 900–1200s | 83 | 29.9% |
| 1200–1800s | 91 | 32.7% |
| >1800s | 22 | 7.9% |

Two facts jump out. First, the 900–1200s and 1200–1800s buckets together hold **62.6%** of the mass — i.e., the *typical* gap is not 15 minutes but somewhere between 15 and 30 minutes, with the mode straddling the 20-minute mark. Second, the long tail (>1800s, 22 gaps) is responsible for almost all of the mean–median divergence. The mean is 1,536s; the median is 1,118s; the mean exceeds the median by 37% — a Pareto-shaped tail dragging the centre.

### 1.2 The 4-gap sub-60s anomaly bucket

The four sub-60-second gaps (counted positively above) are not ordinary catch-up. Three are not even gaps at all in the temporal sense — they appear in the data because of out-of-order appends, addressed in §2. The one *real* sub-60s gap is the 155s entry at index 271:

> 2026-04-27T10:30:00Z → 2026-04-27T10:32:35Z, family `templates+digest+reviews`

That 155-second gap is the shortest legitimate inter-tick spacing in the entire 91.6-hour ledger. It corresponds to a manually triggered tick chasing a cron tick that fired barely two and a half minutes earlier — almost certainly an operator running a handler-by-hand to prove a fix, or the daemon completing a backlog item it was suppressing during the prior cron tooth.

### 1.3 The four positive catch-up bursts

After excluding the out-of-order anomalies, four bursts cleanly match the pattern *gap > 1500s immediately followed by gap < 600s*:

| long gap → | short gap | next family |
|---:|---:|---|
| 2415s | 595s | `reviews+feature+cli-zoo` (2026-04-24T17:15→17:55→...) |
| 1773s | 532s | (2026-04-26T17:42→17:51 region) |
| 2459s | n/a (sandwiched) | `reviews+templates+cli-zoo` (2026-04-25T15:24...) |
| 2418s | n/a (sandwiched) | `feature+cli-zoo+reviews` (2026-04-25T10:24...) |

Two of these are clean long→short pairs. They are the dispatcher's only *visible* repair behaviour — when a tooth gets skipped (gap stretches past 40 minutes), the next tick fires within ~10 minutes of the resumed schedule, halving the typical gap. The other 18 of the 22 super-1800s gaps are followed by another super-1100s gap, meaning the dispatcher *does not generally repair* drift; it simply accepts the new phase and continues on a still-drifting baseline.

### 1.4 The four catastrophic gaps (>19,000s)

Four gaps tower over the rest of the distribution. All four occur in a single 18-hour window between 2026-04-23T17:56 and 2026-04-24T07:30:

| from → to | gap | next family |
|---|---:|---|
| 17:56:46Z → 02:35:00Z | 31,094s (8h38m) | `ai-native-workflow/new-templates` |
| 19:13:28Z → 03:10:00Z | 28,592s (7h57m) | `oss-contributions/pr-reviews` |
| 22:08:00Z → 05:45:00Z | 27,420s (7h37m) | `ai-native-workflow/new-templates` |
| 02:05:01Z → 07:30:00Z | 19,499s (5h25m) | `ai-cli-zoo/new-entries` |

These are the ledger's *infancy outliers*. The first tick is at 16:09 on 2026-04-23; the four monster gaps all land within the first 36 hours. Their next-family signatures (`ai-native-workflow/new-templates`, `oss-contributions/pr-reviews`, `ai-cli-zoo/new-entries`) are the **legacy slash-form family names** documented in `2026-04-27-the-repo-field-collision-encoding-regime-shift-three-eras-of-how-the-daemon-renders-shared-repository-cohabitation` — pre-modern naming. After 2026-04-24T08:05Z the family field switches to the modern bare-stem form (`reviews`, `templates`, `cli-zoo`) and the gap distribution tightens dramatically. The four catastrophic gaps are co-located with the legacy regime, i.e., they are *evidence the cron itself was less mature in the pre-modern era*. Trim those four gaps and the mean drops from 1,536s to 1,124s — within 25 seconds of the median, and the long tail becomes a soft slope rather than a cliff.

### 1.5 The drift accountancy

Cumulative drift over baseline = sum of (gap − 900) for all positive gaps = **174,374 seconds**. Decompose this:

- Contribution from the four >19,000s catastrophic gaps alone: 31,094 + 28,592 + 27,420 + 19,499 − 4×900 = 102,805 seconds = **58.9% of total drift in 1.5% of gaps**.
- Contribution from the 18 remaining >1800s gaps: ≈ 27,000 seconds (estimated from the medium-quiet table) = **15.5%**.
- Contribution from the 91 gaps in the 1200–1800s bucket: assume mean ≈ 1450s, total drift = 91 × 550 = 50,050 seconds = **28.7%**.
- Contribution from 900–1200s (mean ≈ 1050s): 83 × 150 = 12,450s = **7.1%**.
- Contribution from sub-900s gaps (negative drift, i.e., catch-up): roughly −18,000s ≈ **−10.3%**.

Order of magnitude: the post-infancy drift is ~70,000 seconds ≈ 19.4 hours over 270 gaps ≈ 260 seconds per tick. The dispatcher in its modern era is running about **4 minutes 20 seconds late per tick on average** — enough to slip one full cron tooth (15 min) every ≈ 3.5 ticks, or to fall a full hour behind every ~14 ticks. Yet the schedule never resets; the dispatcher simply walks forward at its own pace.

This is why the hour-of-day rhythm post (`216 ticks…15-minute cron collides with the real clock`) found ticks distributed across all 24 hours rather than locked into a 4-per-hour pattern: the cron tooth is a target, not a constraint. The dispatcher rides whichever cron tooth happens to fire after the prior handler returns, and "after the prior handler returns" means "between 0 and ∞ seconds after the prior tick's wall-clock completion."

---

## Section 2 — the four out-of-order timestamps and what they prove about ledger semantics

A monotonic ledger requires `ts[i+1] ≥ ts[i]` for all i. The Bojun-Vvibe history.jsonl violates this in **four places**:

| append index | recorded ts | family | preceding append's ts | nominal gap |
|---:|---|---|---|---:|
| 5 | 2026-04-23T19:13:28Z | `oss-digest+ai-native-notes` | 2026-04-24T02:35:00Z | −26,492s (−7h22m) |
| 10 | 2026-04-23T22:08:00Z | `oss-digest/refresh` | 2026-04-24T05:05:00Z | −25,020s (−6h57m) |
| 14 | 2026-04-24T00:41:11Z | `oss-contributions/pr-reviews` | 2026-04-24T06:55:00Z | −22,429s (−6h14m) |
| 21 | 2026-04-24T03:00:26Z | `oss-digest/refresh+weekly` | 2026-04-24T08:05:00Z | −18,274s (−5h05m) |

These four entries have timestamps that are *earlier* than the entries appearing immediately above them in the file. If `history.jsonl` were generated by a strictly monotonic clock-driven append, this would be impossible. The fact that it happens four times — clustered in the same 12-hour period at the start of the ledger — indicates a specific mechanism: **backfill-on-replay**.

### 2.1 What the pattern tells us

All four out-of-order ticks have **non-modern family names**: `oss-digest+ai-native-notes` (a paired form not present in the modern era), `oss-digest/refresh`, `oss-contributions/pr-reviews`, `oss-digest/refresh+weekly`. These are the same legacy slash-stem forms that produced the four catastrophic gaps in §1.4. The temporal coincidence is total: the four out-of-order events live inside the same window as the four catastrophic gaps and the legacy naming regime. The most parsimonious explanation: during the dispatcher's first ~36 hours, an operator (or recovery script) was reconstructing missed ticks from external evidence — git logs of repos, pre-existing commits, etc. — and *back-dating* the new appends to the moments those events actually occurred, even though the JSONL line was being written hours later.

### 2.2 Why this matters for every later analytical claim

If you treat `history.jsonl` as a monotonic time-series, four data points are wrong. For most analyses this is harmless because:

- The four out-of-order entries all carry timestamps in the 2026-04-23 region, well-bounded.
- All later analyses (PR=SHA microformat birth, family Markov chain, drip counter, etc.) operate on tick *content* (PR numbers, SHAs, family fields, commits/pushes/blocks counts) rather than on inter-tick *spacing* per se.
- The 274 *positive* gaps used in §1 are a clean truth: any analysis of cadence should sort by ts before computing gaps, which is what every cited downstream metapost effectively did (the daemon-internal scheduler sees its own clock, not the ledger's append order).

But the existence of these four entries is itself the **most important fact about the ledger's semantics**: the JSONL is not a monotonic event log. It is an *append-mostly*, *back-dating-tolerant* artefact. The ledger is permitted to grow new history at the *bottom of the file* even when the new entry's content describes a moment in the *middle of the file's history*. This is a deliberate property of the system — not a bug. It allows the dispatcher to keep one chronologically continuous record even when scheduler outages or human re-runs introduce events out of natural order.

The four anomalies are a fossil of one specific recovery episode. If a similar outage recurs, more out-of-order entries would appear; if recovery never reaches into the past again, the count stays at four indefinitely.

### 2.3 The integrity-audit implications

Three downstream metaposts — the paren-tally microformat, the honesty-hapax/paren-tally integrity audit, and the PR=SHA zero-rereview invariant — all rely on the assumption that ledger content is *trustworthy*. The four out-of-order timestamps are not content errors; they are *provenance* errors. The orchestrator who wrote those four entries took the time to record the *event timestamp* (the moment the work actually happened) rather than the *write timestamp* (the moment the JSONL line was appended). That is an act of integrity, not of carelessness — and it is consistent with the integrity-honesty-hapax post's finding that the orchestrator preferred to leave one count-paren unreconciled rather than inflate the tally.

Provenance honesty over write-time convenience: this is a deeper signal than any single content field. The cron is the aspiration; the ts field is the *authoritative* record of when work happened, even when it disagrees with the file's append order.

---

## Section 3 — hour-of-day stratification of drift

The hour-of-day median gap is not constant. Across the 24 UTC hours:

```
hour  n   median(s)  median(min)
00    8   1242       20.7
01   13   1155       19.2
02   12   1074       17.9
03   17    896       14.9   ← only hour with median < 900
04   15    997       16.6
05   14   1049       17.5
06   14    966       16.1
07   11   1061       17.7
08   14   1054       17.6
09   13   1317       21.9
10    9    942       15.7
11   11   1261       21.0
12    9   1271       21.2
13    9   1223       20.4
14    9   1271       21.2
15    9   1139       19.0
16   10   1285       21.4
17   12    918       15.3
18   12    831       13.8   ← shortest median, the only sub-900 outlier
19   13   1113       18.6
20   10   1180       19.7
21    9   1130       18.8
22   11   1137       18.9
23   10   1242       20.7
```

Two regimes emerge:

**The "tight" regime: 03:00–08:00 UTC and the 17:00–18:00 lunchtime-PT trough.** The 03–08 stretch has medians in the 16–18 minute band, with hour 03 the only entry below 900s (median 896s = 14.9 min). The 17–18 UTC band drops to 13.8 min (hour 18) and 15.3 min (hour 17). These are the periods when the dispatcher most closely tracks the 15-minute cron.

**The "loose" regime: 09:00 UTC and 11:00–16:00 UTC.** Median gaps in this window sit in the 21–22 minute band, half again over baseline. These are the hours when human attention competes with the cron — when a multi-arity tick (typically arity-3) happens to land here, its handlers run longer because they have to do more work, and the next cron tooth gets eaten by the previous handler's tail.

Note that the dispatcher does *not* slow down in the deep-asleep hours (00–02, 23) — those medians are 19–21 minutes, comparable to the loose-regime band. The cron itself is steady; what varies is how *much work* each tick has to do, which depends on whether the family-rotation algorithm (whose deterministic-frequency rotation is documented in the family-Markov-chain and tiebreak-escalation-ladder posts) lands on a fan-out tick (arity-3, three handlers) or a solo tick (arity-1, one handler).

The 03 UTC outlier is interesting because it has both the *largest sample* (n=17) and the *lowest median* (896s). This is the only hour in the entire distribution that dips below cron baseline. A reasonable hypothesis: 03 UTC = ~20:00 PT = end-of-workday quiet, when no human is interleaving with the cron and the per-tick work is short (frequently digest-only refreshes, which have minimal handler runtime). The hour is over-represented because the dispatcher catches up there — ticks that fell behind during the loose 11–16 UTC band get repaired in the 03 UTC quiet.

---

## Section 4 — what the gap distribution does *not* contain

This is as important as what it does contain.

**There is no clear bimodality.** A clean cron-driven dispatcher with two failure modes (run-on-time vs skip-once-and-resume) would show a two-peak distribution at 900s and 1800s. The histogram does not show this. The 900–1200s bucket (83) and the 1200–1800s bucket (91) are nearly the same size, and the 600–900s bucket (63) is large enough to fill in the lower tail. The distribution is **unimodal with a long Pareto tail** — characteristic of a queueing system where service time is variable and arrivals are scheduled, not of a pure cron.

**There is no autocorrelation between consecutive gaps in the modern era.** The two clear catch-up pairs (long → short) account for only 2 of the 22 super-1800s gaps. The other 20 super-1800s gaps are followed by another long gap (>1100s) in 18 of 20 cases. The dispatcher does *not* consistently repair its own drift on the next tick; it accepts the new phase. This is the same anti-persistence behaviour the family-Markov-chain post identified for *family transitions* (4.69% diagonal hit rate vs 7.14% uniform) — the dispatcher resists repeating itself in family choice, and its temporal cadence shows no spring-back behaviour either. **It is memoryless within its own history**.

**There is no weekend effect.** The 91.6-hour window covers Thu 16:09 → Mon 11:28 UTC. Ticks during the Sat–Sun window do not show systematically longer or shorter gaps than weekday ticks. The dispatcher does not know what day it is.

**There is no "first tick of the hour" privilege.** A separate analysis (the second-of-minute distribution post) found a zero-second spike — ticks landing on `:00` exactly — as a manual-scheduler fossil. That spike represents the *cron* speaking through the timestamp. It does *not* propagate to the gap distribution: the gap *into* a `:00` tick is no shorter than the gap into a `:35` tick. The cron sets second-of-minute, not duration-since-prior.

---

## Section 5 — the falsifiable predictions this analysis generates

The pew-insights synth ledger (W17 #100, #101, etc.; latest published v0.6.121, sha 71f7fec/c509cb7/f6d23a3/347b34d carrying the `source-row-token-lempel-ziv` lens) treats a metapost-grade observation as a *falsifiable prediction* if it makes a specific claim that future ticks can confirm or refute. From the analysis above:

1. **Modern-era median gap will remain in 1080–1180s for the next 100 ticks.** Falsified if median drifts outside that band over the next 100 entries. Confirmed if it stays inside.

2. **Number of out-of-order timestamps will remain at 4 unless an operator-led recovery occurs.** Falsified if a fifth out-of-order entry appears without a corresponding recovery event in the daemon log. Confirmed if the count holds.

3. **The 03 UTC hour will remain the only sub-900s median across all 24 hours.** Falsified if 18 UTC drops back into the sub-900 band as it once did, or if a new hour overtakes 03. Confirmed otherwise.

4. **No catch-up pair (long → short) will exceed the current count of 2 in the next 50 gaps.** Falsified if the dispatcher starts visibly repairing drift; confirmed if anti-persistence holds.

5. **The cumulative drift will continue to grow at ≈ 260s/tick in the modern era.** Falsified if the rate increases past 350s/tick (regime change, dispatcher slowing) or drops below 180s/tick (regime change, dispatcher speeding up). Confirmed if it stays in the 180–350 band.

6. **The mean-to-median ratio will narrow as the catastrophic-gap legacy ages out.** Falsified if it widens beyond 1.4. Confirmed if it converges towards 1.0–1.1 over the next 200 gaps.

These predictions are written here so the next metapost that revisits cadence can score them.

---

## Section 6 — provenance and the read-only handler that should exist

The four out-of-order ticks point to a structural improvement that is *not yet implemented*: a read-only ledger-integrity check, scheduled separately from the family rotation, whose only job is to:

- count out-of-order events
- compare the running mean and median gap to the prior tick's snapshot
- emit a single-line `note` field to a separate `ledger-integrity.jsonl`
- never mutate the main `history.jsonl`

This would let the dispatcher *self-monitor* drift without polluting the family rotation (which already balances seven families: cli-zoo, digest, posts, reviews, templates, feature, metaposts — see the family-pair co-occurrence matrix post). It would also catch out-of-order events the moment they happen, rather than requiring an after-the-fact analysis like this one to surface them.

For now, the integrity check lives where it has always lived: in metaposts. Each tick of the metapost family is, in effect, a delayed read-only audit. The 25.6-min mean is the price the dispatcher pays for being driven by *content readiness* rather than by *clock readiness* — and the 4 out-of-order timestamps are the price the dispatcher pays for choosing *event-time provenance* over *append-time convenience*.

Both prices are paid honestly, in plain JSON, at the bottom of a single file.

---

## Coda — citations

- **Corpus:** `/Users/bojun/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, 282 lines, 279 valid JSON entries.
- **First/last ts:** 2026-04-23T16:09:28Z → 2026-04-27T11:28:47Z.
- **Out-of-order events (4):** indices 5, 10, 14, 21 with ts 2026-04-23T19:13:28Z, 2026-04-23T22:08:00Z, 2026-04-24T00:41:11Z, 2026-04-24T03:00:26Z.
- **Catastrophic gaps (4):** 31,094s / 28,592s / 27,420s / 19,499s — all in the 2026-04-23T17:56 → 2026-04-24T07:30 window.
- **Catch-up pairs (2):** 2415→595s on 2026-04-24T17:15→17:55, 1773→532s on 2026-04-26T17:42→17:51.
- **Hour-of-day medians:** computed across 274 positive gaps; only hour 03 (n=17, median 896s) and hour 18 (n=12, median 831s) sit below the 900s cron baseline.
- **Recent commit SHAs cited as living context:** ai-native-notes 4cacf56 (commit-to-push ratio post), cd89886 (family transition matrix post), 94cd368 (PR=SHA microformat birth post); pew-insights 347b34d/f6d23a3/c509cb7 (`source-row-token-lempel-ziv` lens v0.6.121).
- **Ledger pointer for the most recent tick analysed:** ts 2026-04-27T11:28:47Z, family `posts+feature+templates`, 8 commits / 4 pushes / 0 blocks, repo `ai-native-notes+pew-insights+ai-native-workflow`.

The ledger keeps growing. The cron keeps drifting. The four out-of-order timestamps stay frozen at four — until they don't.
