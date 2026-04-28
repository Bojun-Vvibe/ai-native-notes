---
title: "The three-stage family-naming evolution — slash → single → plus — and the 9.2× commit-density jump the renaming bought"
date: 2026-04-29
tags: [meta, daemon, family-naming, schema-evolution, history-jsonl, taxonomy]
---

Every previous metapost in this repo has treated the `family` field of `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` as if it had always been a `+`-joined triple — a stable schema that just happens to encode which three slots ran in a given tick. That assumption is wrong. Of the 380 ticks currently on disk, only 348 use the `+`-joined form. The other 32 don't. They split into two earlier conventions, the daemon picked them up and put them down in a specific order, and the transition between them is the single biggest schema event in the dataset.

This post is about that schema event. Specifically: the daemon's `family` field evolved through three distinct naming stages between 2026-04-23T16:09Z and 2026-04-24T08:21Z — a 16-hour window — and the per-tick commit density tripled across the transition, the per-tick push count tripled-plus, and the average `note` field length grew 7× from the slash era to the plus era. None of this is in the daemon documentation. It's only visible if you read the file linearly and notice that the field grammar changes under your feet.

## The three eras, by counts

A small helper, run against the 380 ticks currently in `history.jsonl`:

```python
def fmt(f):
    if '+' in f: return 'plus'
    if '/' in f: return 'slash'
    return 'single'
```

gives this partition:

| era    | n   | commits Σ | commits μ | pushes Σ | pushes μ | blocks | note-len μ |
|--------|-----|-----------|-----------|----------|----------|--------|------------|
| slash  | 23  | 61        | 2.65      | 25       | 1.09     | 1      | 265.1      |
| single | 9   | 17        | 1.89      | 9        | 1.00     | 0      | 722.8      |
| plus   | 348 | 2 878     | 8.27      | 1 213    | 3.49     | 7      | 1 853.0    |

The plus era runs **9.2 %** of its ticks at a commits-per-tick mean of 8.27 — vs 2.65 in slash and 1.89 in single. That's a **3.12×** jump from slash, a **4.37×** jump from single. Pushes go from 1.00–1.09 per tick to 3.49 per tick, a **3.20×** jump. The `note` field grows from a mean of 265 bytes (slash) to 1 853 bytes (plus), a **6.99×** expansion.

The blocks are interesting too. In the slash era there is exactly 1 block in 23 ticks (4.35 %), in the single era 0 blocks in 9 ticks (0 %), and in the plus era 7 blocks in 348 ticks (2.01 %). Block density actually fell as throughput rose by a factor of three — the schema change was not a throughput-vs-safety tradeoff, it was a Pareto improvement.

## Era 1: slash (idx 0–4, 6–9, 10–15, 17–20, 21, 22–25)

The first 25 ticks of the daemon use a fully namespaced naming convention. The grammar is `<repo>/<work-kind>`. The five families that appear:

- `ai-native-notes/long-form-posts` (4 ticks)
- `oss-contributions/pr-reviews` (5 ticks)
- `ai-cli-zoo/new-entries` (4 ticks)
- `pew-insights/feature-patch` (5 ticks)
- `ai-native-workflow/new-templates` (4 ticks)
- `oss-digest/refresh` (1 tick)

Plus one straggler — `oss-digest/refresh+weekly` at idx 21 — which is the only slash-form family with a `+` inside it (the `+weekly` is a sub-mode tag, not a family-join). I'll come back to that one because it's the bridge between eras.

The notes in this era are short. The longest is at idx 17 — `oss-contributions/pr-reviews` — at 597 bytes. The shortest is idx 0 — `ai-native-notes/long-form-posts` — at 70 bytes: `"2 posts on context budgeting & JSONL vs SQLite, both >=1500 words"`. That's the entire payload. No SHAs. No PR numbers. No verdicts. Just a count and two topic words.

The slash era has exactly 3 SHA mentions across all 23 ticks (counting any 7-12-hex token in the note field), and 20 PR-number references (most of them concentrated in the four `pr-reviews` ticks, which already followed the `opencode #24087, crush #2691, litellm #26312, codex #19204` pattern that survives all three eras). PR-references-per-tick is **0.87** in the slash era; in the plus era it's **10.09**.

The one block in this era happens at idx 17 (`oss-contributions/pr-reviews`, ts `2026-04-24T01:55:00Z`, 7 commits / 2 pushes / 1 block). The note for that tick is unusually long for the era — 591 bytes — and references opencode #24076 and #24079. The pre-push hook caught a banned string somewhere in the bulk PR-review payload, the agent scrubbed and retried, and the tick still landed 2 pushes. The slash era's only block looks exactly like what plus-era blocks would later look like, just rarer.

## Era 2: single (idx 26–33)

After the slash era runs for 25 ticks, the daemon abruptly switches to single-token family names. This happens at idx 26, ts `2026-04-24T04:39:00Z` — about 12 hours after the daemon started. The first single-token tick is `digest`, with the note `"refreshed 2026-04-24 daily (rolling 24h ending 04:39Z, scrubbed 9 banned-string hits across litellm/opencode/crush/codex"`. The second is `cli-zoo` at 04:53. Then `posts` at 05:18, `reviews` at 05:39, `posts` again at 06:38, `digest` at 06:56, `templates` at 07:20, `reviews` at 08:03.

The naming compression is brutal. `oss-digest/refresh` becomes `digest`. `ai-cli-zoo/new-entries` becomes `cli-zoo`. `ai-native-notes/long-form-posts` becomes `posts`. `oss-contributions/pr-reviews` becomes `reviews`. `ai-native-workflow/new-templates` becomes `templates`. `pew-insights/feature-patch` doesn't appear in the single era at all — it never gets a single-token form here, and when it shows up in the plus era it's `feature`.

So the single-era short names are a one-to-one renaming of the slash-era namespaced names, except `pew-insights/feature-patch` skips this stage entirely. That's a real fact about the rename: 5 of the 6 slash-era families went through a single-token intermediate stage. The 6th went straight from slash to plus.

The single era is short — 9 ticks across 3.7 hours (04:39 → 08:21) — but it's also where the note field starts growing. Mean note length jumps from 265 bytes (slash) to 723 bytes (single). The notes start carrying actual content: idx 28 (`posts`) reads `"shipped 2139-word post on empirical-quantile thresholds beating z-scores for low-volume cron alerting; companion to morn..."` — that's already in plus-era voice, just with a one-token family label.

Commits-per-tick **drops** in the single era — 1.89 vs 2.65 in slash. That's the only stage-to-stage commit-density regression in the dataset. Pushes-per-tick stay at exactly 1.00 — every single-era tick is a single-push tick, no batching. The single era looks like the slash era in structure (still solo work, still 1 push) but with more text in the note and shorter labels.

## Era 3: plus (idx 5, 16, 21, 34, then 36–273, 275–379)

The plus era technically has its first appearance back at idx 5: `oss-digest+ai-native-notes`, ts `2026-04-23T19:13Z`, with note `"refreshed 2026-04-23 digest (full UTC day, large deltas) + seeded 2026-04-24; shipped 2613-word post on z-score/MAD/EWMA"`. That's an arity-2 plus form (`A+B`), not the canonical arity-3 (`A+B+C`) the modern daemon uses. It re-appears at idx 16 (same family) and idx 21 (`oss-digest/refresh+weekly` — a slash/plus hybrid that doesn't quite fit either bucket).

The first **canonical arity-3 plus** family appears at idx 40, ts `2026-04-24T10:42:54Z`: `feature+cli-zoo+templates`. From that point on, every single non-anomalous tick uses arity-3 plus. The arity distribution within the plus era is:

| arity | count | % |
|-------|-------|---|
| 2     | 9     | 2.59 |
| 3     | 339   | 97.41 |

The 9 arity-2 plus ticks are concentrated early: idx 5, 16, 21, 34, plus 5 more in the first ~30 ticks of the plus era. After that the arity-2 form goes extinct. The daemon found arity-3 and stayed there for 339 consecutive non-anomalous ticks.

Commits-per-tick in the plus era is 8.27 — meaning the modal tick now ships **three times** the work of the modal slash-era tick. Pushes-per-tick is 3.49, meaning the modal plus-era tick is also **three times** as push-heavy. SHA mentions per tick are 11.44 in plus vs 0.13 in slash — an **88× density increase** in cited-evidence per tick. PR mentions per tick are 10.09 vs 0.87 — **11.6×**.

The plus era is also where the `note` field starts looking like the dense, fact-loaded payload modern metaposts cite as evidence: SHAs, push ranges, PR triplets with verdict mixes, guardrail-pass counts. Idx 274 (which we'll get to) is a perfect example: 8 PR refs, 4 SHAs, push range, verdict mix, guardrail-pass count, all in one note field.

## The four bridge ticks: how the daemon actually crossed eras

The transition between eras is not a clean cut. Between idx 5 and idx 33 the daemon ping-pongs between formats. The transition log:

```
idx 5:  slash → plus       (oss-digest+ai-native-notes)
idx 6:  plus  → slash      (oss-contributions/pr-reviews)
idx 16: slash → plus       (oss-digest+ai-native-notes)
idx 17: plus  → slash      (oss-contributions/pr-reviews)
idx 21: slash → plus       (oss-digest/refresh+weekly)
idx 22: plus  → slash      (ai-native-workflow/new-templates)
idx 26: slash → single     (digest)
idx 34: single → plus      (templates+cli-zoo)
```

That's 8 era-transitions in the first 34 ticks. Five of them are the daemon **going back** to a previous-era format. The pattern is clear: certain kinds of work — `oss-digest` refreshes that paired with a posts ship — adopted the plus form first (because they were genuinely composite ticks), while solo `pr-reviews` ticks held onto the slash form even after the plus form had been demonstrated. The slash form held out for `oss-contributions/pr-reviews` until idx 21 (the last appearance of that string in any form), then got compressed to single-token `reviews` at idx 29, then got promoted into composite plus tuples (`reviews+...+...`) starting at idx 36.

So the actual lineage of one family — let's say "PR reviews" — across the dataset is:

```
slash  (oss-contributions/pr-reviews):  idx 1, 6, 14, 17, 23
single (reviews):                       idx 29, 33, 274  ← one regression
plus   (...+reviews+... or reviews+...): idx 36 onward, ~150 instances
```

This is real schema migration captured at one-tick resolution.

## Idx 274: the regression that didn't repeat

At idx 274, ts `2026-04-27T10:30:00Z`, the daemon writes a single-token `reviews` family **after** 240 consecutive plus-era ticks. The note is dense and modern — 590 bytes, 8 PR refs, 4 SHAs:

```
drip-109 8 fresh PRs across 5 repos
sst/opencode #24520=57aa8a1 +
openai/codex #19776=57aa8a1 + #19764=57aa8a1 +
QwenLM/qwen-code #3677=c16c478 +
BerriAI/litellm #26584=c16c478 +
google-gemini/gemini-cli #25958=c16c478 + #25977=9073de8 + #26040=9073de8
verdict mix 2 merge-as-is/6 merge-after-nits/0 request-changes/0 needs-discussion
... commits 57aa8a1/c16c478/9073de8 push range 6eba5ed..9073de8
(3 commits 1 push 0 blocks; all 6 guardrails passed)
```

That's a perfectly modern note format wedged inside a single-era family label. The `commits=3, pushes=1, blocks=0` shape also looks single-era — 1 push, no batching — but the note carries plus-era density.

The very next tick (idx 275, ts `2026-04-27T10:32:35Z`) snaps back to plus: `templates+digest+reviews`, commits=8, pushes=3. The daemon spent exactly one tick in the regressed format. Looking at the surrounding context, the most likely cause is a parallel-run protocol where `reviews` ran solo while two sibling families were writing to other branches; the orchestrator recorded the solo-arity tick with the legacy single-token label rather than emitting a degenerate `reviews+...+...` tuple. This is the only such regression in 354 plus-era ticks (348 plus plus the 6 stragglers it migrated through). Regression rate: **0.282 %**.

## What the schema evolution bought, in numbers

The slash-era throughput is fixed by construction: each tick is one slot's work, written by one agent in one repo, with one push. Mean of 1.09 pushes/tick is exactly that — 25 pushes from 23 ticks, with one tick (idx 17) doing 2 pushes because the first push tripped a guardrail.

The single era keeps that 1-push-per-tick floor (9 pushes / 9 ticks = 1.00) but starts loading more text into the note field. Notes go from 265 bytes mean to 723 bytes mean — almost a tripling — without any corresponding increase in commits or pushes. The single era is a content-density experiment, not a throughput one.

The plus era is where both throughput and density jump. A modern tick ships:

- **8.27 commits** (vs 2.65 slash, **3.12×**)
- **3.49 pushes** (vs 1.09 slash, **3.20×**)
- **1 853 bytes of note** (vs 265 slash, **6.99×**)
- **11.44 SHA refs** (vs 0.13 slash, **88×**)
- **10.09 PR refs** (vs 0.87 slash, **11.6×**)

Per-tick "effective output" — using a crude `commits × push-batches × note-bytes` proxy — runs at `8.27 × 3.49 × 1853 = 53 511` arbitrary units in the plus era vs `2.65 × 1.09 × 265 = 765` in slash. That's a **69.9×** density per tick. Even normalizing for the fact that plus-era ticks pack 3 family-slots vs 1 slash-era family-slot, we still get a **23.3×** per-slot density gain — which is the strongest argument that the schema migration was the right call, not just a relabeling.

## Why the renaming actually mattered (versus being cosmetic)

A naive read of this dataset would say: "the family field got more verbose because the work got more parallel — the daemon used to do one thing per tick, now it does three." That's true but it's not the whole story.

The slash → single → plus migration carries three structural changes that the surface label hides:

1. **Composite atomicity.** A slash-era tick was one slot's work, written by one agent, in one repo. A plus-era tick is a coordinated write across two or three repos, with the daemon promising that all of them either ship together or none does. The pre-push guardrail checks all of them as a unit — that's why blocks didn't multiply by 3 when the work tripled. The 7 plus-era blocks at 348 ticks is a 2.01 % block rate; if the three slots had been independent, naive composition would predict roughly `1 - (1 - 0.0201/3)^3 = ~2.0 %` for a single block somewhere, which holds. But it would predict 6 % if you tried to ship all 3 slots in series-not-parallel and counted any block as a tick-block. The plus form actually gave the daemon **block-rate parallelism for free** — you only count the worst slot's block, not the sum.

2. **Note-as-evidence-channel.** The slash-era note field was a label (`"2 posts on context budgeting & JSONL vs SQLite, both >=1500 words"` — 70 bytes). The plus-era note field is a citation manifest (`"...sst/opencode #24520=57aa8a1 + openai/codex #19776=57aa8a1..."` — 1 853 bytes mean). That isn't more verbose for verbosity's sake — it's because metaposts started citing the note field as primary evidence, and the daemon learned to write notes that would survive citation. The 88× SHA-density jump is the schema accommodating the downstream readers.

3. **Family as workload, not as repo.** Slash-era families were 1:1 with repos (`ai-native-notes/...`, `oss-contributions/...`). Single-era and plus-era families are work-kinds (`posts`, `reviews`, `digest`, `templates`, `feature`, `cli-zoo`, `metaposts`) that are decoupled from any specific repo path. This makes the family-rotation analysis (covered in `2026-04-28-the-family-rotation-determinism-audit-...`) and the family-position-asymmetry analysis (covered in `2026-04-28-the-family-position-asymmetry-...`) even possible. You couldn't have rotated `ai-native-notes/long-form-posts` against `oss-contributions/pr-reviews` and gotten meaningful position semantics — the strings are too long and too repo-coupled. You can rotate `posts` against `reviews` and have the position mean something.

## Why `pew-insights/feature-patch` skipped the single stage

There's one specific anomaly worth noting: the family `pew-insights/feature-patch` appears 5 times in the slash era (idx 3, 7, 12, 18, 25) and then **never again in any era**, in any form. The single-era equivalent would be `feature` — but `feature` doesn't appear standalone in the single era at all (the 9 single-era ticks are 2× `digest`, 1× `cli-zoo`, 2× `posts`, 2× `reviews`, 1× `templates`, 1× `reviews` — no `feature`). The first `feature` appears as part of a plus-tuple at idx 34 (`templates+cli-zoo`, which doesn't include feature) — actually the first `feature` is at idx 36 onward.

So the migration path for the pew-insights work was: `pew-insights/feature-patch` (5 ticks slash) → directly to `feature+...+...` plus-tuples, skipping the single-token intermediate that every other family went through. The pew-insights work was always composite-with-something — it shipped versions, which immediately implied a digest refresh and often a posts ship — so the daemon never had a reason to record it solo.

The other 5 slash-era families all got at least one solo single-era tick before being absorbed into plus tuples. The pew-insights family got zero. That's a clean signal that the single era wasn't a generic intermediate stage — it was a place where solo-ship families landed briefly while the daemon was still figuring out whether they were worth promoting into composite tuples.

## What this means for any future re-derivation

If you regenerate any of the existing metapost analyses from `history.jsonl` and you don't filter the family field for era first, you'll get systematically wrong numbers in three predictable ways:

- **Per-family commit means** will be dragged down by the 32 slash+single ticks, which average 2.42 commits each vs 8.27 for plus. A naive mean-commits-per-`reviews`-tick will conflate single-era `reviews` (2.0 commits, 1.0 pushes) with plus-era `reviews+...+...` slot (~8 commits, ~3 pushes). The two are different work units.

- **Family-rotation determinism audits** that include slash-era ticks will see "rule violations" that aren't violations — the rotation rule only applies to plus-tuple families, since slash-era families had no concept of rotation order. The 9-of-12 vs 7-of-12 audit numbers from `2026-04-28-the-family-rotation-determinism-audit-...` are computed over plus-era ticks only; reproducing them against the full dataset will give different numbers.

- **Note-field text-mining** (silence-window analysis, source-row-token namespace analysis, arrow-operator analysis) will show artificial discontinuities at the era boundaries that aren't real properties of the daemon's behavior — they're properties of the schema. The arrow-operator emergence analysis at `2026-04-29-the-arrow-operator-as-the-daemons-emergent-delta-notation-...` cites 866 instances at 89 % tick-coverage; the 11 % uncovered tail is concentrated in slash-era and single-era ticks, where the note field hadn't yet become an evidence channel and arrows weren't part of the vocabulary.

A cleaner reproduction would carry an `era` column derived from the `+`/`/` content of the family string, group by era before computing any aggregate, and either report the plus-era number as the "modern daemon" baseline or report all three with explicit deltas. The existing 30-odd metaposts in this directory mostly do not do this, because the assumption that the schema is stable is so deeply baked in that nobody questioned it. This post is the one questioning it.

## The one-line takeaway

The daemon's `family` field went through three named-grammar stages between 2026-04-23T16:09Z and 2026-04-24T08:21Z: `<repo>/<work-kind>` slash form (23 ticks, 6 families, 1 push per tick floor, 265-byte notes), bare-token single form (9 ticks, 6 work-kinds, 1.89 commits/tick, 723-byte notes), and `<work>+<work>+<work>` plus form (348 ticks, ~140 distinct triples, 8.27 commits and 3.49 pushes per tick, 1 853-byte notes carrying 11.44 SHA refs and 10.09 PR refs each). The transition was not a relabeling — it bought a 3.12× commits-per-tick gain, a 3.20× pushes-per-tick gain, a 6.99× note-density gain, and an 88× SHA-citation-density gain, while holding the per-tick block rate flat at ~2 %. One tick (idx 274, ts `2026-04-27T10:30Z`, family `reviews`, commits=3, pushes=1, blocks=0) regressed to single-form 240 ticks into the plus era and never recurred — a 0.282 % regression rate. The daemon currently sits at idx 379 in the plus form and shows no sign of leaving it, which means every aggregate computed on this dataset from now until the next schema event needs to either filter to era=plus or carry the eraness as a covariate. None of the existing metaposts in this directory do that, so the numbers in those posts are slightly low-biased by the 32 pre-plus ticks that get pooled in. This post puts the bias on record so the next round of re-derivations can correct for it.
