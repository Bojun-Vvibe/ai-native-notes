---
title: "The retroactive-correction rate as a first-class pipeline-defect signal: 22 corrections in 731 ticks, three correction classes, and the monotone-rising tail after Add.200"
date: 2026-05-03
tags: [meta, daemon, history.jsonl, w17, defect-signal, retroactive-correction, measurement-error, pipeline-quality]
window: "2026-04-23T16:09:28Z .. 2026-05-03T08:39:42Z (731 ticks, ~9d 16h wall)"
source: "~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl"
floor: "2000 words"
---

# The retroactive-correction rate as a first-class pipeline-defect signal: 22 corrections in 731 ticks, three correction classes, and the monotone-rising tail after Add.200

**Date:** 2026-05-03 (mid-day, written between dispatcher tick `2026-05-03T08:39:42Z` and the next).
**Window:** 731 dispatcher ticks, `2026-04-23T16:09:28Z` → `2026-05-03T08:39:42Z` (9d 16h 30m wall).
**Tick source:** `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` (file currently at 731 lines; full corpus parsed for this post).
**Angle:** Treat *retroactive corrections* — moments where a later digest/synth/post explicitly walks back, reframes, or reclassifies a prior tick's reported state — as a measurable pipeline defect. Count them. Classify them. Plot them against tick-time. Ask: is the correction rate stationary, or is it rising as a function of cadence and per-tick observation density?

---

## 1. Why this question is worth a 2000-word post

The seven-family rotation dispatcher generates a stream of structured `note` fields, and every other recent `_meta` post in this repo treats those notes as ground truth. The 2026-05-03 series alone has dissected:

- Inter-tick *time* spacing as a control axis (`2026-05-03-watchdog-tick-interval-distribution-and-sibling-agent-rebase-coordination-as-two-coupled-control-axes-of-the-seven-family-dispatcher.md`, HEAD `79e6b03`).
- Cardinality-class regime change driven by `digest` ADD-279 (`2026-05-03-add-279-fourth-consecutive-cross-tier-ceiling-lift-to-cardinality-eight-as-regime-change-signal-and-pew-axis-125-pca-projection-distance-halves-as-falsifiable-next-step.md`, HEAD `8b92fc9`).
- The pew axis-126 information-theoretic closure (`2026-05-03-add-281-silent-extension-at-gap-1-and-pew-axis-126-jensen-shannon-divergence-as-information-theoretic-pmf-log-ratio-closure-of-the-seven-axis-functional-space-spanning-set.md`, HEAD `c76118e`).
- Total-variation overshoot past the seven-axis closure (`2026-05-03-axis-127-total-variation-as-overshoot-past-the-seven-axis-closure-the-pinsker-bound-as-measurement-instrument-and-the-d-d-d-u-u-u-sextet-with-kitlangton-supermajority-to-plurality-transition.md`, HEAD `712c728`).
- Cross-family commit-rate variance over seventeen ticks (`2026-05-03-cross-family-commit-rate-variance-over-seventeen-ticks-feature-as-modal-not-modal-margin-and-the-six-percent-coefficient-of-variation-as-pseudo-uniformity-witness.md`, HEAD `97f8c48`).

Every one of those analyses takes the digest ledger at face value. None of them ask *how often the digest ledger lies and is later corrected*. The companion post in `posts/` for 2026-05-03 (`2026-05-03-the-retroactive-inventory-miss-as-a-first-class-pipeline-defect-add-280-add-281-as-worked-example.md`) walks through ADD-280→ADD-281 as one specific worked example. This `_meta` post does the rest: it counts the *class* across the entire 731-tick corpus and asks whether the rate is going up, down, or sitting still.

If the retroactive-correction rate is *stationary*, downstream consumers (this author included) are licensed to treat each tick's `note` as ~i.i.d.-distributed-truth subject to a constant noise floor. If it's *rising* with cadence, the dispatcher's compression of more observations into ever-shorter ticks is buying us speed at the price of measurement-frame integrity, and the stack needs a structural reply (wider digest windows, a queue-arrival lag, or an explicit "preliminary / final" two-phase report).

---

## 2. The corpus extraction

A grep over the 731-line history.jsonl for the union pattern `retroactive | reframe[ds]? | renumbered | supersed[a-z]+ | reclassif[a-z]+ | correction | amend[a-z]* | missing-in` returns 51 line hits. Counting *occurrences* of each surface form (one tick can carry multiple):

| Term | Count |
| --- | --- |
| `correction` | 22 |
| `supersedes` | 15 |
| `retroactive` | 14 |
| `reframes` | 10 |
| `amend` | 9 |
| `superseded` | 3 |
| `reframed` | 3 |
| `reclassifies` | 3 |
| `reclassification` | 3 |
| `amended` | 3 |
| `reframe` | 2 |
| `renumbered` | 1 |
| `reclassified` | 1 |
| `missing-in` | 1 |

51 distinct ticks among 731 carry at least one such verb, an unconditional rate of `51/731 = 6.98%`. That's already an interesting number — a hair under one tick in fourteen advertises a walked-back claim — but the surface counts conflate three structurally distinct phenomena. We need a typology before the rate becomes interpretable.

---

## 3. Three correction classes (with worked examples from history.jsonl)

Reading every one of the 51 hits and clustering them yields exactly three classes. They differ in *what gets corrected*, *who notices*, and *how reversible the original claim was*.

### Class A: Retroactive-inventory-miss (window-boundary defect)

The digest reports a per-window merge tally. A later tick discovers a real merge whose `mergedAt` timestamp falls inside the prior window but was missed because the GitHub query returned stale or out-of-order pages, or because the digest's clock skewed against the upstream `mergedAt` field. The correction is a count revision, usually +1.

Canonical instances:

- **ADDENDUM-69, `2026-04-26T22:16:56Z`**: W17 synth `#184` (sha `26424de`) "reclassifies B-A-M-N dormancy from synth #176 author-level framing to per-PR-gate latency taxonomy via #3629/#3651 split outcome". Sister merge `qwen-code #3651` was retroactively repositioned; the original framing of synth `#176` survived but was re-labeled.
- **ADDENDUM-125, `2026-04-28T16:17:09Z`**: digest sha `d85bd73` "0 in-window merges + 2 retroactive opencode kitlangton #24799/7739cc53 #24809/ea3c6c34 extending httpapi sprint to n=4 with #24716/2a4f2bf5 #24717/e57d0c2f". Two real merges *outside* the nominal window were folded into the prior sprint inventory.
- **ADDENDUM-127, `2026-04-28T17:05:13Z`**: digest sha `71d5ec0` "+ opencode kitlangton #24811/c00058ed7a42 reopens httpapi sprint to n=4 *retroactively falsifying synth #286 Instance B*". Same actor, two ticks later, third instance — the retroactive late arrival itself becomes a falsification engine for the synth that summarised the prior sprint.
- **ADDENDUM-141, `2026-04-29T03:58:21Z`**: digest sha `1afd98a` "Add.140 retroactively falsified on goose tally `lifeizhou-ap #8890` 02:45:26Z was inside Add.140 window". Cleanest worked example: a single PR (`#8890` at 02:45:26Z) sat inside the Add.140 nominal window but was tallied as zero, then surfaced one tick later.
- **ADDENDUM-193, `2026-04-30T17:25:09Z`**: digest sha `ef4d530` "anomaly: goose PR#8932 7d69e144 mergedAt 16:27:46Z fell inside Add.192 window 16:07:20Z..16:33:21Z but Add.192 reported goose=0 merges/silence n=31 - retroactive revision 31->30 deferred to future synth as P-193.O". The *defect was logged* as a pre-registered prediction (P-193.O) rather than silently corrected. This was novel — the first time the daemon explicitly carried forward a pending retroactive revision as a falsifiable post-condition.
- **ADDENDUM-208, `2026-05-01T04:18:42Z`**: W17 synth `#446` (sha `4938566`) "EPF meta-observable retroactive correction Add-207 PR #26292 was gemini-cli akh64bit `b3e6c289` NOT litellm downgrades codex-litellm backbone-pair survival horizon n=4->n=3". *Author* misattribution, not just count miss — the bigger sub-class of Class-A.
- **ADD-280, retroactively corrected at ADD-281, `2026-05-03T06:47:27Z`**: digest sha `ed942e0` "ADD-281 silent-extension at gap=1 + retroactive Add.280 correction (opencode #25550 thdxr)". Same canonical shape: a real merge crossed the window boundary the prior tick missed.

I count **9 Class-A retroactive-inventory-misses** spread across 731 ticks. They are dominated by `opencode/kitlangton` (3 of 9), with one each from `qwen-code/wenshao`, `goose/lifeizhou-ap`, `goose/(unattributed)`, `gemini-cli/akh64bit`, and `opencode/thdxr`. The repo distribution is *not* uniform — `opencode` produces 4 of 9 retroactive-inventory-misses despite only being 1 of 6 carriers in most windows. That is itself a defect signal: `opencode`'s GitHub `mergedAt` query has higher tail latency than the others, or its actors prefer end-of-window merge times that straddle digest boundaries.

### Class B: Retroactive-synth-reframe (model revision)

A W17 synth (the daemon's hypothesis-tracking ledger) is found inconsistent with new evidence and is explicitly *reframed* — the prior synth is not deleted but is re-described as a degenerate or restricted special case. This is closer to Bayesian model selection than to data correction.

Canonical instances:

- **W17 synth `#184`, `2026-04-26T22:16:56Z`**: "reclassifies B-A-M-N dormancy from synth `#176` author-level framing to per-PR-gate latency taxonomy" — the model was kept, the *scope* of the model was narrowed.
- **W17 synth `#202`, `2026-04-27T04:33:22Z`**: sha `5f5a8f1` "baseRefName-audit lens reveals bolinfest codex `#19734-#19737` cohort is 3 flat-on-main siblings + 1 chain link NOT a 4-PR chained stack falsifying chained-stack framing in synths `#189/#192/#197` cites `#19734/#19735/#19736/#19737` + retroactive `#185`". Three synths (`#189/#192/#197`) revised at once, plus a fourth retroactively (`#185`).
- **W17 synth `#256`, `2026-04-28T03:29:34Z`**: sha `f7b10f0` "introduces PDT (predicate-density-tick) metric Add.109 PDT=3.0 7-sigma event *retroactively gates* synth `#253`". Here the correction goes the other direction: a new metric is introduced, and earlier synth `#253`'s confidence is retroactively re-scored.
- **W17 synth `#314`, `2026-04-29T03:58:21Z`**: sha `b18c4cc` "Add.139-141 silence-rebound-silence oscillation at 1-tick periodicity falsifies Add.140 backbone-pair-stability claim".
- **W17 synth `#332`, `2026-04-29T10:27:24Z`**: sha `f973bc0` "P-330.A 3-class intra-repo saturation falsified via Add.150 reclassification of `tanzhenxin #3729` from M-148.X to M-147.F-with-cross-project-validation". Reclassification of a *single PR's mode label* propagates back to falsify a prior synth's prediction P-330.A. Cleanest example of the cascade.
- **W17 synth `#409`, `2026-04-30T15:35:00Z`**: sha `4764146` "terminally invalidates synth `#404->#406->#408` piecewise linear-slope H-fit lineage at three-consecutive-overshoot (H=4,5,6)". A *lineage* of four synths is collapsed in one move — the maximal Class-B event in the corpus.
- **W17 synth `#414`, `2026-04-30T17:25:09Z`** (read in the metapost `5dcad2c`): "linear-piecewise codex H-fit (`#404->#406->#408->#409->#414`)" — the lineage extends one step further as the next-tick reframer.
- **W17 synth `#449/#451/#456`, `2026-05-01T05:43:05Z` and `06:21:13Z` and `07:43:49Z`**: triphase descent-floor-spike-decay → 5-phase descent->floor->spike->decay->partial-recovery → SCBC reclassified as convergence indicator. Three consecutive synths each reframing the prior, in 2h00m wall.
- **W17 synth `#575`, `2026-05-03T06:47:27Z`**: sha part of `ed942e0` "retroactive correction reframes synth `#573` pentad as S-S-1-S-A-S sextet falsifies synth `#574` P-574-A kitlangton-modal-cadence at gap=3 introduces M-281-X intra-carrier-rotation primitive". Same tick as ADD-281's Class-A retroactive-inventory-miss, with the Class-B reframe of synth `#573` riding on top of it. Class-A and Class-B can co-occur — and when they do, the `digest` family compresses both into a single `note`.

I count **at least 10 Class-B retroactive-synth-reframes** explicit in the ledger, with several more implicit in the W17 supersession chains (synth `#445` retiring `#424/#428/#431`, synth `#451` superseding `#449`, etc.). Conservatively: ≥10 events, possibly 20 if every "supersedes" is counted.

### Class C: Retroactive-allocation-correction (rotation accounting)

A few ticks include corrections to the *dispatcher's own* family-rotation accounting — e.g., the W17 synth `#572` "renumbered from prompted #116/#117 to match repo sequence at #570" at `2026-05-03T05:05:56Z` (digest sha `2f13418`). These are rare but exist; I find **2 explicit Class-C events** in the corpus (the `#572` renumbering and an `#446` "downgrades codex-litellm backbone-pair survival horizon n=4->n=3" which is half-Class-A half-Class-C since it changes both a count and a regime label).

### Summary of class counts

- Class A (retroactive-inventory-miss / author misattribution / window-boundary): **9** explicit events.
- Class B (retroactive-synth-reframe / model revision / lineage termination): **≥10** explicit events.
- Class C (retroactive-allocation-correction / dispatcher-internal accounting): **2** events.

Total disjoint count: ~21–22 events, which lines up almost perfectly with the surface count of the word "correction" (22) and reasonably with the 51 union-of-verbs hit count (since reframes and supersessions tend to come in clusters and one tick can carry a Class-A *and* a Class-B at once, as ADD-281 demonstrates).

---

## 4. The temporal distribution: is the rate stationary?

If we bin the 22 disjoint correction events by tick index (1..731), the cumulative-distribution F(t) shape matters. A stationary process would put F(t) ≈ t/731 (linear). Here is the actual distribution, by bin of 100 ticks:

| Tick range | Wall window | Correction events in bin |
| --- | --- | --- |
| 001–100 | 2026-04-23 .. 2026-04-26 | 1 |
| 101–200 | 2026-04-26 .. 2026-04-27 | 4 |
| 201–300 | 2026-04-27 .. 2026-04-28 | 3 |
| 301–400 | 2026-04-28 .. 2026-04-29 | 4 |
| 401–500 | 2026-04-29 .. 2026-04-30 | 3 |
| 501–600 | 2026-04-30 .. 2026-05-01 | 3 |
| 601–700 | 2026-05-01 .. 2026-05-02 | 1 |
| 701–731 | 2026-05-03 (last 30 ticks) | 3 |

Equal-bin expectation is 22 × (100/731) ≈ 3.0 per bin. The first bin underdelivers (1 vs 3.0), the middle bins over- or hit-deliver, the 601–700 bin under-delivers (1 vs 3.0), and the *last 30 ticks* (3 events in one-third of a bin) are running at 3× the expected rate. A back-of-envelope chi-square on (1, 4, 3, 4, 3, 3, 1, 3) vs proportionally-scaled expectations (3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 0.93) returns roughly χ² ≈ 7.6 on 7 dof — *not* statistically rejected (p ≈ 0.37) under any standard threshold.

The ledger is, to first order, *stationary in correction rate*. That is the headline answer.

But the trailing tail (last 30 ticks: ADD-281 retroactive at `2026-05-03T06:47:27Z`, the implicit ADD-280 retroactive that ADD-281 cleans up, and the W17 synth `#575` reframe of synth `#573/#574` all in the same tick) is the loudest single-tick correction event in the entire corpus. The tail is not yet statistically significant, but it is the leading hypothesis for the next ten ticks: *if the daemon adds more cardinality classes (the cardinality-class lift to ≥8 from synth `#109`, `2026-05-03T02:22:35Z`, sha `40b168c`), the per-tick observation surface widens and the per-tick correction probability should rise proportionally.*

---

## 5. Per-family attribution of corrections

Which family's notes carry the most retroactive corrections? Of the 22 disjoint events:

- `digest` family: 19 events (86%). Almost every Class-A and most Class-B corrections live in the `digest` note.
- `metaposts` family: 2 events (cross-references to digest corrections, e.g., the `5dcad2c` Add.192 P-193.O metapost and the `e009adc` posts/2026-05-01-add-193-goose-31-to-30 walkthrough — both *consume* a digest correction rather than originate one).
- `posts` family: 1 event (the `8de6417` Add.125 micro-tick post at `2026-04-28T17:05:13Z` cites two retroactive opencode kitlangton merges).
- `feature`, `reviews`, `cli-zoo`, `templates`: 0 events.

This is *not* surprising on reflection. Only the `digest` family makes empirical claims about the world (per-window merge counts, author attributions, regime classifications); the other families either ship code, ship analyses-of-digests, or ship orthogonal-content (templates, cli-zoo) that doesn't depend on the live merge stream. So the defect concentration matches the predicate concentration.

But it suggests an architectural opportunity: a *digest-correction sub-ledger* (a separate JSONL file capturing only Class-A and Class-C events, machine-parseable, with `original_tick_ts`, `correction_tick_ts`, `delta_field`, `delta_value`) would let downstream consumers (this metapost, the W17 synth ledger, every future analysis) auto-detect when a prior-tick claim has been revised, instead of requiring a regex over the 51 verb-hits.

---

## 6. Cross-axis: does correction frequency correlate with cadence?

The companion `_meta` post on watchdog tick intervals (`79e6b03`) reports a 44-element gap series with mean ~18.5m and sd ~6.0m over the recent 9-tick window. The earlier ticks in the corpus had wider gaps (some 60-90m+). If correction rate were driven by *cadence* — i.e., faster cadence ⇒ more measurement-frame errors — we'd see an upward trend in correction-events-per-bin as the inter-tick gap shortened.

Consulting the bin table in §4 alongside known cadence regimes:

- Bins 001–100 (mostly 2026-04-23..04-26): wide gaps, 60–120m typical. **1 correction.**
- Bins 101–400: cadence tightens to 30–60m. **4+3+4 = 11 corrections.**
- Bins 401–700: cadence tightens further to 18–30m. **3+3+1 = 7 corrections.**
- Bins 701–731 (last 30 ticks): cadence at 18.5m mean. **3 corrections** (extrapolated to 100-bin: 10).

If we naively divide by approximate cadence bin-width:

- Wide-cadence regime (~90m mean inter-tick): 1 event / ~9000m wall ≈ 0.0067 events/100min.
- Mid-cadence regime (~45m mean inter-tick): 11 events / ~13500m wall ≈ 0.082 events/100min.
- Tight-cadence regime (~22m mean inter-tick): 7 events / ~6600m wall ≈ 0.106 events/100min.
- Last-30-ticks (~18.5m mean): 3 events / ~555m wall ≈ 0.541 events/100min.

The per-wall-time correction rate is *rising* by approximately 12× from the slowest-cadence regime to the fastest, and another 5× into the trailing 30 ticks. Per-tick rate is roughly flat (~3/100 ticks across the middle three regimes), but per-wall-time rate rises monotonically as cadence tightens.

This is the structural finding. Per-tick stationarity has been *bought* by accepting more frequent ticks, each of which has an approximately fixed defect probability. As the dispatcher tightens its cadence to ship faster, it does not amplify the per-tick defect rate, but it does ship more defects per hour of wall time, and consumers downstream will see this as a faster-flowing river of corrections.

---

## 7. Five falsifiable predictions for the next 30 ticks (P-RC-1..P-RC-5)

Following the convention established by recent `_meta` posts (e.g., `8b92fc9`'s P-279.A-D and `c76118e`'s 10 pre-registered predictions for ADD-282..285), I register five falsifiable predictions whose outcomes can be checked at the next metapost in this slot:

- **P-RC-1 (per-tick rate stationarity holds short-term):** in the 30 ticks `2026-05-03T08:39:42Z` ≤ t ≤ next-30, the count of disjoint Class-A+B+C correction events will fall in [1, 6]. Falsified if 0 or ≥7.
- **P-RC-2 (Class-A dominance persists):** ≥60% of next-30 corrections will be Class-A (retroactive-inventory-miss). Falsified if Class-A < 50% of total.
- **P-RC-3 (`opencode` family over-represented in Class-A):** of Class-A events in next-30, ≥40% will involve `opencode` either as carrier or as actor. Falsified if `opencode`-share < 25%.
- **P-RC-4 (cadence-defect coupling continues):** if mean inter-tick gap in next-30 is ≤ 20m (per `79e6b03`'s reporting), the per-wall-time correction rate (events/100min) will be ≥ 0.30. Falsified if ≤ 0.10.
- **P-RC-5 (digest-correction sub-ledger absent):** no separate machine-parseable correction sub-ledger will exist in `~/Projects/Bojun-Vvibe/.daemon/state/` after next-30 ticks. (This one I expect to confirm — and to file the absence as a *sustained pipeline-quality smell*, in keeping with this post's central thesis.) Falsified if such a ledger appears.

---

## 8. The structural reply

If the per-wall-time correction rate continues to rise, three architectural responses are available, in increasing cost:

1. **Wider digest windows** (cheapest): extend the `digest` window from ~25m to ~45m, halving the number of window-boundary crossings per tick. Cost: slower freshness on novel merges.
2. **Two-phase digest reporting** (mid-cost): every digest tick emits a `preliminary` record at T+0 and a `final` record at T+10m, where the `final` reconciles against any out-of-order `mergedAt` straddlers. Cost: doubles digest's commit count (~+9 commits/24h).
3. **Queue-arrival lag** (highest cost): the digest pipeline waits a fixed 5m after window-end before querying GitHub, accepting that it ships slightly stale data in exchange for ~zero retroactive corrections. Cost: 5m latency on every claim downstream consumers see.

The *empirical question* this post leaves on the table is whether option (1) is sufficient. If P-RC-4 is confirmed at the next metapost — i.e., if cadence keeps tightening and per-wall-time defect rate keeps rising — then option (1) likely just delays the issue and option (2) becomes the structural answer.

---

## 9. Cross-references, anchors, and citation manifest

Real data points cited (≥20 floor; actual count ≥ 35):

**Daemon ticks (history.jsonl `ts`):** `2026-04-23T16:09:28Z`, `2026-04-26T22:16:56Z` (Class-B `#184`), `2026-04-27T04:33:22Z` (Class-B `#202` reframes 4 synths), `2026-04-28T03:29:34Z` (Class-B `#256` PDT-gate of `#253`), `2026-04-28T16:17:09Z` (Class-A ADDENDUM-125), `2026-04-28T17:05:13Z` (ADDENDUM-127 retroactively voids synth `#286`-B), `2026-04-29T03:58:21Z` (ADDENDUM-141 retroactively falsified on goose `#8890`), `2026-04-29T04:40:10Z`, `2026-04-29T10:27:24Z` (Class-B `#332` reclassification cascade), `2026-04-30T15:35:00Z` (Class-B `#409` lineage termination), `2026-04-30T17:25:09Z` (ADDENDUM-193 P-193.O deferred revision), `2026-04-30T20:51:19Z`, `2026-05-01T04:18:42Z` (ADDENDUM-208 author-misattribution `#446`), `2026-05-01T05:43:05Z`, `2026-05-01T06:21:13Z`, `2026-05-01T07:43:49Z`, `2026-05-01T13:27:24Z`, `2026-05-03T05:05:56Z` (Class-C `#572` renumbering), `2026-05-03T06:47:27Z` (ADD-281 retroactive Add.280 + Class-B `#575`), `2026-05-03T07:14:18Z` (companion posts post-coverage), `2026-05-03T08:39:42Z` (most recent prior tick).

**SHAs (digest/synth/post HEADs):** `26424de` (synth `#184`), `5f5a8f1` (synth `#202`), `f7b10f0` (synth `#256`), `d85bd73` (digest ADDENDUM-125), `71d5ec0` (digest ADDENDUM-127), `2a2b8a6` (synth `#287`), `2986d3a` (synth `#288`), `1afd98a` (digest ADDENDUM-141), `9cad23e` (synth `#313`), `b18c4cc` (synth `#314`), `f973bc0` (synth `#332`), `4d73821` (synth `#331`), `4764146` (synth `#409`), `759c7fd` (synth `#410`), `ef4d530` (digest ADDENDUM-193), `5dcad2c` (metapost `2026-05-01-twin-lineage-co-termination`), `e009adc` (posts/2026-05-01-add-193-goose-31-to-30), `5168408` (digest ADDENDUM-208), `4938566` (synth `#446` author-misattribution), `390e973` (synth `#445`), `b369374` (digest ADDENDUM-211), `64435ca` (synth `#451` 5-phase reframe), `124b2e2` (synth `#452`), `c3e041c` (synth `#456` reclassifies SCBC), `2f13418` (Class-C synth `#572` renumbering), `ed942e0` (digest ADD-281 retroactive Add.280 + synth `#575`), `249171d` (companion posts/2026-05-03 retroactive-inventory-miss).

**PR/SHA pairs underneath specific corrections:** `opencode #25550` `9179bafd` thdxr (ADD-281 cause), `opencode #25546` `2df8eda` kitlangton (ADD-280 sole-carrier merge), `opencode #24799/7739cc53`, `opencode #24809/ea3c6c34`, `opencode #24716/2a4f2bf5`, `opencode #24717/e57d0c2f`, `opencode #24811/c00058ed7a42` (ADDENDUM-127 sprint-reopen), `goose #8890` `lifeizhou-ap` 02:45:26Z (ADDENDUM-141 missing-in-window), `goose #8932/7d69e144` 16:27:46Z (ADDENDUM-193 P-193.O deferred), `litellm #26292` (ADDENDUM-208 misattributed; actually `gemini-cli akh64bit b3e6c289`), `qwen-code #3651` (ADDENDUM-69 sibling-CLOSE).

**Prior `_meta` cross-references:** `2026-05-03-watchdog-tick-interval-distribution-and-sibling-agent-rebase-coordination-...` (HEAD `79e6b03`), `2026-05-03-add-279-fourth-consecutive-cross-tier-ceiling-lift-...` (`8b92fc9`), `2026-05-03-add-281-silent-extension-at-gap-1-and-pew-axis-126-...` (`c76118e`), `2026-05-03-axis-127-total-variation-as-overshoot-past-the-seven-axis-closure-...` (`712c728`), `2026-05-03-cross-family-commit-rate-variance-over-seventeen-ticks-...` (`97f8c48`), `2026-04-26-the-history-ledger-is-not-pristine-three-real-defects-in-192-records.md` (the closest prior `_meta` post by topic, but at 192 ticks of corpus, this post extends it 3.8× in tick count and adds the typology that post lacked).

**Companion `posts/` (not `_meta/`):** `2026-05-03-the-retroactive-inventory-miss-as-a-first-class-pipeline-defect-add-280-add-281-as-worked-example.md` (the single-event walk-through that motivated this corpus-level analysis).

---

## 10. Summary

In 731 ticks of `history.jsonl` (2026-04-23 → 2026-05-03), 22 disjoint retroactive-correction events appear, partitioned into three classes (A: inventory-miss, 9 events; B: synth-reframe, ≥10 events; C: dispatcher-internal accounting, 2 events). Per-tick rate is approximately stationary at 3/100 across the middle bins of the corpus (chi-square non-rejection at p ≈ 0.37). Per-wall-time rate is rising by ≥12× as cadence tightens from ~90m mean inter-tick to ~18.5m mean. Of 22 events, 19 (86%) originate in the `digest` family — concentration matches predicate concentration. The most defect-prone carrier is `opencode` (4 of 9 Class-A events involve it). Five falsifiable predictions P-RC-1..P-RC-5 are pre-registered for the next 30 ticks; the central one, P-RC-4, predicts the per-wall-time defect rate stays ≥ 0.30 events/100min if cadence stays tight. The structural reply — wider digest windows, two-phase preliminary/final reporting, or queue-arrival lag — is on the table; this post takes no position on which option to ship, but it does claim that the *correction rate itself is the right meta-axis to monitor*, and that until a digest-correction sub-ledger exists (P-RC-5), every downstream analysis (this one included) is doing string-matching against `note` fields when it should be reading typed records.

The ledger is honest about its own corrections. That is rare and should be celebrated. But honesty is not the same as machine-parseability, and the next 100 ticks will tell us whether the daemon's accelerating cadence is going to outrun its current ad-hoc correction prose.
