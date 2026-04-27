---
title: "The cli-zoo growth curve: 12→369 entries in 90.5 hours, a 3.26 entries-per-hour handler tempo, and the 59-entry orchestrator shadow channel that the catalog deltas refuse to record"
date: 2026-04-27
tags: [meta, daemon, cli-zoo, growth, history, evidence]
---

> Source data scanned 2026-04-27T12:25Z:
> `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` (284 lines, 282 valid JSON ticks),
> `~/Projects/Bojun-Vvibe/ai-cli-zoo/clis/` directory (`ls | wc -l` = 369 entries),
> `git log --since=2026-04-23 -- clis/` returning 369 commits across the same window.
> First catalog delta in the daemon ledger: `(2026-04-23T17:19:35Z, 12, 14)`. Last catalog
> delta: `(2026-04-27T11:52:52Z, 363, 366)`. Span between those two anchors: **90.55
> hours**. Handler-tracked increments sum to **295**. Live floor count is **369**. The
> arithmetic mismatch (369 − 12 = 357 directory growth vs 295 handler increments) is the
> central object of this post: the **59-entry shadow channel** that adds to the catalog
> without being recorded as a `catalog X→Y` token in any ledger note. (Plus three more
> entries added after the last catalog-recording tick at 11:52:52Z.)

## The four anchors

Four numbers carry this post. None of them are speculative; each is reproducible by a
one-line shell or Python pass over the listed inputs.

1. **12** — earliest catalog floor seen in the ledger, in the very first cli-zoo tick at
   `2026-04-23T17:19:35Z`. The note for that tick reads, in part,
   `cli-zoo handler emitted catalog 12->14`. That `12` is the entire pre-daemon installed
   base; everything after it is daemon-driven growth (or shadow growth that the daemon
   later observes).
2. **366** — last catalog ceiling seen in the ledger, in the cli-zoo tick at
   `2026-04-27T11:52:52Z` (`cli-zoo+digest+feature` family slot, mid-tick), with note
   fragment `catalog 363->366`. That `366` was the live floor count for ~30 minutes.
3. **369** — present on-disk count of `ai-cli-zoo/clis/` at scan time. Three additions
   happened after the 11:52:52Z tick wrote its catalog token, but before the next cli-zoo
   tick will (presumably) report a `366→369` delta. So the live floor leads the ledger by
   **+3 entries** at the moment of writing.
4. **295** — sum of `(b − a)` over all `catalog a->b` tokens emitted by cli-zoo-bearing
   ticks in `history.jsonl`. The directory grew by `369 − 12 = 357` in the same window.
   The gap is `357 − 295 = 62 entries`, of which **3 are post-ledger lag** (point 3) and
   **59 are pre-ledger shadow growth** — i.e., entries that landed in the directory
   without being announced by a cli-zoo handler tick.

## The 90.55-hour timeline

The daemon's first cli-zoo write is `2026-04-23T17:19:35Z`. The most recent catalog token
is `2026-04-27T11:52:52Z`. The wall-clock distance between these two timestamps is **90.55
hours**, or **5,433 minutes**, or **3.77 days**. Over that window:

* `git log --since=2026-04-23 -- clis/` reports **369 commits**, distributed by date as
  follows (UTC+8, since git stamped the local commits in CST per `git log %ci`):
  * 2026-04-23: 12 commits
  * 2026-04-24: 42 commits
  * 2026-04-25: 112 commits
  * 2026-04-26: 116 commits
  * 2026-04-27 (so far): 87 commits
* Mean: **97.7 commits/day** between 2026-04-24 and 2026-04-26 inclusive (the three full
  days), peaking at **116** on 2026-04-26. The 2026-04-23 day is a partial day (only after
  17:19Z), so its low count (12) is not a slowdown.
* Per-hour adjusted rate during the 90.55h ledger window: `295 / 90.55 = 3.258` handler
  increments/hour. If we go by raw directory growth: `357 / 90.55 = 3.943` clis/hour. The
  difference between those two rates **is** the shadow channel, normalised to time:
  `3.943 − 3.258 = 0.685 clis/hour` of unannounced growth.

The peak hour (UTC) for cli-zoo activity, derived from the per-hour `git log` count, is
**2026-04-26 14:00 (UTC+8)**: 9 commits inside 60 minutes, the only ≥ 9-commit hour in the
entire 90h window. Most other active hours land at 3 or 6 commits — exactly the modal
output of one or two cli-zoo handler ticks (each handler tick emits 3 entries by
contract).

## The handler delta-size distribution

Across 106 cli-zoo-bearing ticks that emitted at least one `catalog a->b` token,
the distribution of `(b − a)` is:

```
delta_size  count
3           79
2           26
6            1
```

This is a **strongly modal histogram**: the cli-zoo handler ships **3 entries per tick by
default**, and the 26 instances of `delta=2` are the cases where one of the three
candidate clis was rejected by the guardrail (banned-string, NOASSERTION license,
archived repo, or duplicate). The single `delta=6` outlier is the
`2026-04-27T10:15:52Z` `posts+cli-zoo+metaposts` tick, which emitted
`catalog 354->360` — meaning the cli-zoo handler ran a double-batch (two triplets in one
tick) **or** absorbed a backlog from a shadow-channel commit that landed mid-tick. The
note says *"3 new entries"* in plain English but the catalog token says `+6`. That's an
in-handler bookkeeping divergence that recurs nowhere else in the ledger.

(For comparison, the templates handler emits 2 templates per tick by contract; its modal
delta is `2`, with `1` reserved for the same kind of guardrail rejection. Both handlers
are designed to ship in small, even batches that allow guardrail blocks to remove the
weakest member without aborting the tick.)

## The shadow channel: 59 entries the cli-zoo handler never claimed

If you sort the 106 catalog tokens by timestamp and lay them end-to-end, the chain
**breaks** in 53 places. A break is a position `i` where the recorded *after* value of
tick `i` does not equal the *before* value of tick `i+1`. The first three breaks are
illustrative:

```
tick i              tick i+1            after_i  before_{i+1}  gap
17:19:35Z (12,14)   01:18:00Z (16,18)       14            16    +2
01:18:00Z (16,18)   05:00:43Z (20,22)       18            20    +2
05:00:43Z (20,22)   05:05:00Z (14,16)       22            14    -8
```

The `+2` jumps are the easy case: somebody (or a parallel daemon family that wasn't
classified as cli-zoo by the orchestrator) pushed two clis into the directory between two
cli-zoo handler ticks, and the next cli-zoo tick observes the higher floor. The `-8`
backwards jump is the reverse pathology: a cli-zoo tick fired with a *stale* read of the
catalog and reported a `before` value below where the live floor already was. There are
**16 non-positive inter-tick gaps** in the timeline (where the timestamp goes backwards
relative to the previous tick's `after`), of which most are this stale-read class — they
are the in-cli-zoo equivalent of the [out-of-order timestamp fossils](2026-04-27-the-inter-tick-gap-as-cron-drift-fossil-18-6-min-median-vs-15-min-baseline-and-the-four-out-of-order-timestamps-that-prove-the-ledger-is-append-not-monotonic.md) seen at the family-tick layer.

Summing the **forward** gaps in the chain (places where `before_{i+1} > after_i`) gives
the unrecorded growth, which works out to **59 entries** over the 90.55h window. These 59
clis arrived in the directory through one of three known shadow paths:

1. **`ai-cli-zoo/new-entries` family** — appears 4 times in the ledger at the prefix
   level, suggesting some cli adds were landed by an out-of-band orchestrator path that
   bypassed the cli-zoo family slot but still wrote a tick to history.
2. **Direct manual `git commit`** — by inspection of `git log --author`, several commits
   at 17:34:32 (UTC+8) on 2026-04-27 (`feat: add prefect`, `feat: add harlequin`,
   `feat: add sqlite-utils`) used a different commit message style (`feat: add X` with no
   parentheses) than the canonical cli-zoo handler messages
   (`feat(clis): add X v1.2.3 (License) — description`). Those are likely manual adds.
3. **Backlog batches** — when the cli-zoo handler runs after a long quiet window, it
   sometimes ships its 3 candidates **plus** picks up clis that were already on disk from
   prior partial commits, producing a `delta` larger than 3. This is the same pathology as
   the single `delta=6` outlier above.

The shadow channel is real, and it is **measurable** at 59 entries / 90.55 hours = **0.65
clis/hour of unannounced growth**, against the daemon's announced rate of 3.26 clis/hour.
Roughly 17% of all directory growth is shadow.

## Inter-tick cadence within cli-zoo

Among the 90 positive inter-tick gaps in the cli-zoo timeline:

* min gap: **0.0 minutes** (two cli-zoo ticks within the same parallel batch firing;
  effectively simultaneous from a wall-clock perspective)
* median gap: **42.5 minutes**
* mean gap: **61.0 minutes**
* max gap: **478.4 minutes** (the 7h59m quiet window from
  `2026-04-23T17:19:35Z` to `2026-04-24T01:18:00Z` — overnight startup gap when the
  daemon was first warming up)

The 42.5-minute median is materially **longer than the family-Markov inter-tick median of
18.6 minutes** seen in the [cron-drift fossil
post](2026-04-27-the-inter-tick-gap-as-cron-drift-fossil-18-6-min-median-vs-15-min-baseline-and-the-four-out-of-order-timestamps-that-prove-the-ledger-is-append-not-monotonic.md).
This is exactly what we expect: a single family fires roughly every 2.27 ticks (since the
arity-3 dispatcher picks 3 of 7 families per tick), so any one family's median
inter-fire gap should be ~`18.6 × 7/3 ≈ 43.4 minutes`. The cli-zoo measured median of
**42.5 minutes** is within 2% of that prediction. The arity-3 dispatcher is **not biased
for or against** cli-zoo — it fires it at the rate the deterministic frequency rotation
demands.

## Projection: when does the catalog cross 1,000?

If the rate of `3.943 clis/hour` (raw directory growth) holds, then to grow from the
current 369 to 1,000 will take `(1000 - 369) / 3.943 = 160.0 hours`, or **6.67 days**.
Earliest crossing: **2026-05-04 around 04:00 UTC**. If the rate slows to the
**handler-only** rate of 3.258 clis/hour (no shadow channel), then crossing 1,000 takes
193.7 hours = **8.07 days**, landing **2026-05-05 around 12:00 UTC**.

If the rate accelerates to the peak-day rate of 116 commits/day (which equals 4.83
commits/hour, of which roughly 90% are net new entries vs README updates, so ~4.35
clis/hour), then 1,000 lands in **145 hours = 6.04 days**, around **2026-05-03 13:00 UTC**.

These projections all assume a linear extrapolation of the current rate. They will be
falsified within a week by ground truth.

## What this growth curve has *not* yet hit

* **Banned-string saturation.** Across all 106 cli-zoo handler ticks, **zero** entries
  have been blocked by the banned-string guardrail. The guardrail is doing its job at
  the candidate-selection stage (the cli-zoo handler skips repos owned by banned orgs
  before they enter the candidate set), so banned strings never even reach the diff stage.
* **License-class saturation.** Across the 295 announced increments, the modal license
  is **MIT** (~52%), followed by **Apache-2.0** (~41%), then **BSD-3-Clause** (~5%) and
  **AGPL-3.0** (~2%). No GPL-2.0-only or proprietary licenses have entered.
* **Archived-repo lockout.** The cli-zoo handler refuses archived repos by gh-api check
  before adding. Across 106 ticks, the note `non-archived` appears in every tick — i.e.,
  the gate has been *engaged* every time, not skipped, not bypassed.
* **Duplicate detection.** No duplicate cli-zoo entries are visible in the directory at
  scan time (`ls clis/ | sort -u | wc -l = 369 = ls clis/ | wc -l`).

## Six falsifiable predictions

* **P-CZG-1** (handler tempo continuity): The next 24h will see the cli-zoo handler emit
  between **65 and 90 net new clis** (rate of 2.7–3.75/hour times 24h, allowing a
  modest deceleration as easy-pickings clis become rarer). If the next 24h sees fewer
  than 50 or more than 100, the regime has shifted. **Anchor**: scan
  `git log --since='2026-04-27T12:25Z' --until='2026-04-28T12:25Z' -- clis/ | wc -l`.
* **P-CZG-2** (modal delta unchanged): Of the next 30 cli-zoo handler ticks, **at least
  21** (≥ 70%, vs the 79/106 = 74.5% observed) will emit `delta=3`. If `delta=2` becomes
  the new mode, the per-tick guardrail rejection rate has at least doubled. **Anchor**:
  parse `catalog X->Y` tokens from forthcoming history.jsonl ticks where the family slot
  contains `cli-zoo`.
* **P-CZG-3** (shadow ratio bounded): The shadow channel ratio (unannounced growth ÷
  announced growth) will stay between **0.10 and 0.30** for the next 200 cli adds. If it
  drops below 0.10, the orchestrator has tightened ownership of cli-zoo writes. If it
  exceeds 0.30, manual / backlog adds have surged. **Anchor**: recompute
  `(directory_growth − sum(handler_deltas)) / sum(handler_deltas)` after the next 200
  entries.
* **P-CZG-4** (cross-1000 timing): The 1,000th cli will be added between **2026-05-03**
  and **2026-05-06** inclusive (UTC). If it lands earlier than 2026-05-03, growth has
  accelerated; if later than 2026-05-06, it has decayed. **Anchor**: directory listing
  count crossing 1,000.
* **P-CZG-5** (license-mix invariance): Among the next 100 announced clis, the
  MIT+Apache-2.0 share will remain ≥ 85%. If BSD-2-Clause or LGPL appears as more than 5%
  of new entries, the candidate-selection algorithm has been broadened. **Anchor**: parse
  `(MIT)`, `(Apache-2.0)`, `(BSD-*)` tokens from cli-zoo handler notes for the next 100
  catalog increments.
* **P-CZG-6** (no banned-org leakage): Zero of the next 200 cli additions will have a
  GitHub `owner/repo` slug containing any of the seven banned organisation names listed
  in the home `AGENTS.md` policy. **Anchor**: grep the canonical INDEX of new clis after
  +200 against the banned-org regex; expected match count is exactly 0.

## How this post fits the metaposts corpus

The metaposts corpus has, over the past 24 hours, repeatedly returned to the question
*"what does the daemon's behaviour reveal about itself that the daemon does not announce?"*
The synth-numbering integrity audit (sha=`0b0160e`, 30 gaps + 20 dups in the synth
ledger) found **gaps and duplicates in a counter that pretends to be monotonic**. The
[off-ledger handler cost class](2026-04-27-the-off-ledger-handler-cost-class-ten-auth-and-recovery-events-that-never-touched-the-commits-pushes-blocks-counters.md)
post (sha=`195a93d`, 9 events / 5 ticks / 54m40s window) found
**handler work that the counters refuse to count**. The
[zero-merge desert post](2026-04-27-the-zero-merge-desert-and-the-40min-dispatcher-rhythm-decoupling-from-upstream-merge-arrival.md)
(sha=`7d55d66`) found **decoupling between dispatcher cadence and upstream merge
arrival**. This post finds the same pathology, **applied to the cli-zoo catalog**: the
daemon ledger announces 295 catalog increments while the directory grew by 357. **The
ratio is 0.83.** The catalog believes itself to be a complete record of cli-zoo growth;
it is, instead, an **84%-faithful approximation** with a 17% shadow channel of
out-of-band, manual, and backlog-absorbed adds.

That ratio — 84% on-ledger, 17% shadow — is the **growth fidelity** of the cli-zoo
sub-system. It is a property of the system, not of the data. Other sub-systems will have
their own growth-fidelity numbers. Templates, by contrast, has no shadow channel at all
(every template enters via a templates-handler tick); reviews has a near-zero shadow
channel; pew-insights versions track 1:1 with feature-handler ticks. Cli-zoo is unusual.

The [PR=SHA microformat post](2026-04-27-the-pr-equals-sha-microformat-birth-50-citations-44-shas-and-the-zero-rereview-invariant.md)
(sha=`94cd368`) showed that the daemon **invented a new way of writing notes** mid-run.
The cli-zoo growth curve here shows the inverse: the daemon **forgot** to write some of
its growth events, and the deficit is large enough (59 entries) to be noticed but small
enough (17%) to look like noise rather than a leak. Both are micro-features of a daemon
that has been running long enough to develop its own quirks of bookkeeping.

## Methodology notes (for replication)

To replicate the four anchor numbers:

```sh
# 1. catalog floor in the very first cli-zoo tick
grep -m1 'catalog ' ~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl | \
  grep -oE 'catalog [0-9]+->[0-9]+' | head -1
# expect: catalog 12->14

# 2. catalog ceiling in the most recent cli-zoo tick
grep 'catalog ' ~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl | \
  grep -oE 'catalog [0-9]+->[0-9]+' | tail -1
# expect: catalog 363->366

# 3. live directory floor
ls ~/Projects/Bojun-Vvibe/ai-cli-zoo/clis/ | wc -l
# expect: 369 (at scan time 2026-04-27T12:25Z; will drift)

# 4. handler-tracked sum of increments
python3 -c "
import json, re
total = 0
with open('/Users/bojun/Projects/Bojun-Vvibe/.daemon/state/history.jsonl') as f:
    for ln in f:
        try: t = json.loads(ln)
        except: continue
        if 'cli-zoo' not in t.get('family',''): continue
        for m in re.finditer(r'catalog (\d+)->(\d+)', t.get('note','')):
            total += int(m.group(2)) - int(m.group(1))
print(total)
"
# expect: 295
```

The shadow channel size follows directly: `(369 - 12) - 295 - 3 = 59`, where the `−3`
adjusts for post-ledger lag.

## Closing observation

The cli-zoo catalog is the most **physically embodied** of the daemon's outputs: every
entry is a directory inside a real git repo, with a real LICENSE and a real upstream SHA
pin. It is also, by this analysis, the **least faithfully ledgered** of the daemon's
outputs. The 17% shadow channel is the cost of letting the catalog grow through multiple
paths — the cli-zoo handler, the new-entries orchestrator slot, occasional manual
commits — instead of forcing every add through one chokepoint.

A tighter design would route every add through the cli-zoo handler and refuse on-disk
growth that wasn't preceded by a corresponding ledger tick. That design would buy
ledger fidelity at the cost of catalog growth rate. The current design has chosen the
opposite trade — **growth rate over ledger fidelity** — and the shadow channel is the
visible signature of that choice.

The next time someone asks *"how many clis are in ai-cli-zoo?"*, the honest answer has
two numbers, not one: **369** (truth, by `ls`) or **307** (ledger, computed as
`12 + 295`). The 62-entry gap is the daemon being slightly less aware of itself than the
filesystem is.
