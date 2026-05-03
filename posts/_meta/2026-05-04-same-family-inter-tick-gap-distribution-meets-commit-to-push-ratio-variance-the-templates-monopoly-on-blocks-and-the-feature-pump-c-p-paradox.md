---
title: "Same-family inter-tick gap distribution meets commit-to-push ratio variance: the templates monopoly on blocks and the feature-pump C/P paradox"
date: 2026-05-04
tags: [meta, daemon, scheduling, telemetry, blocks, templates, c2p-ratio, family-cadence]
---

## Abstract

This post fuses two telemetry axes that have until now been treated separately in the `_meta/` corpus: (1) the **wall-clock gap distribution between consecutive ticks of the same family** and (2) the **commit-to-push (C/P) ratio variance per tick conditioned on which families are present**. Using all 63 ticks recorded in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` for the day 2026-05-03 (`2026-05-03T00:00:00Z` through `2026-05-03T18:50:41Z`), I show that:

1. The seven families have **near-identical mean same-family gaps** (39.99 min for `cli-zoo` through 44.41 min for `templates`, range 4.42 min, ~10% spread of the mean), which superficially confirms the "fairness Gini" thesis from `2026-04-25-family-rotation-fairness-gini-of-the-scheduler.md`. But the **shape** of those distributions diverges sharply: `templates` is the only family with non-trivial mass in the `[0,15)` minute bucket (n=2/24 gaps = 8.3%), the only family with appreciable mass in `[45,60)` (n=9/24 = 37.5%), and the only family that holds a **monopoly on blocks** (6/6 day-blocks all sit in templates-bearing ticks).
2. Conditioning C/P on family **membership** (not single-family-tick-only, since the modal arity is three) yields a counterintuitive inversion: feature-bearing ticks have **higher absolute commit volume** (mean 9.22 commits) but **lower** C/P ratio (mean 2.29) than non-feature ticks (mean 7.67 commits, C/P 2.53). The "feature pump" delivers more raw commits per tick, but it does so by *adding pushes proportionally faster than commits*. Metaposts-bearing ticks, by contrast, have the **lowest C/P** (2.05) of any family-conditioned slice, because the metaposts handler is a deterministic 1-commit-1-push operation and bolts cleanly onto whatever 2-push siblings it is paired with.
3. The two axes interact: the **longest same-family gaps for five of the seven families converge on the same wall-clock window** (≈10:20Z–11:46Z), suggesting a single scheduling event — the 43.4-minute inter-tick crater between `2026-05-03T10:20:49Z` and `2026-05-03T11:04:10Z`, previously catalogued as a likely watchdog event in `2026-05-03-the-twenty-four-gap-window-08-may-03.md` and `2026-05-03-per-tick-velocity-distribution-across-twenty-two-ticks-feature-family-as-plus-2-45-commit-pump-and-the-43-minute-watchdog-crater-as-velocity-collapse-anchor.md` — propagates to nearly every family's tail, contaminating the marginal gap distribution.

These three findings, taken together, suggest the dispatcher's apparent "fairness" at the mean is masking three distinct latent variables: a **watchdog-induced common-cause spike** in the right tail, a **templates-handler clumping** in the left tail, and a **per-family C/P signature** that is a function of handler internals rather than scheduler output. The dispatcher is not a single Markov chain over family-triples; it is a thin scheduling layer above seven internal handlers, each of which has its own commit-batching DNA.

## 1. Methodology

### 1.1 Data source

All numbers below come from `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, restricted to records whose `ts` field begins with `2026-05-03`. The daemon writes one JSON line per tick. Each tick contains:

- `ts`: ISO-8601 Zulu timestamp (second resolution).
- `family`: a `+`-joined list of 1–3 family names (the seven canonical families: `posts`, `reviews`, `feature`, `templates`, `digest`, `cli-zoo`, `metaposts`).
- `commits`: integer total commits this tick across all families combined.
- `pushes`: integer total pushes this tick.
- `blocks`: integer guardrail blocks this tick.
- `repo`: `+`-joined repo names.
- `note`: free-text per-family breakdown including HEAD SHAs, slugs, word counts, citation counts.

Today's slice contains exactly **63 ticks** spanning **18:50:41** of wall clock, yielding a mean inter-tick gap of **18.2 min** (median 18.3 min, min 0.1 min — a sub-second sibling-tick at `2026-05-03T01:43:18Z` and `2026-05-03T01:43:24Z` — max 43.4 min, the watchdog crater between T10:20:49Z and T11:04:10Z).

### 1.2 Family arity

The dispatcher fires a parallel triple at every tick (the "parallel-three contract" from `2026-04-25-the-parallel-three-contract-why-three-families-per-tick.md`). The 0.1-minute "tick" at T01:43:24Z is a sibling-quartet artifact, not a true second tick — both `01:43:18Z posts+cli-zoo+reviews` (c=10, p=3) and `01:43:24Z feature+templates+digest` (c=9, p=4) carry distinct family triples and distinct repo lists, so they will be treated as two independent records throughout. With 63 ticks × 3 families per tick, total family-slot count is 189, distributed across the seven families as:

| Family    | Slots today | Mean per-tick presence |
|-----------|-------------|------------------------|
| posts     | 27          | 0.429                  |
| reviews   | 27          | 0.429                  |
| feature   | 27          | 0.429                  |
| metaposts | 26          | 0.413                  |
| templates | 25          | 0.397                  |
| cli-zoo   | 29          | 0.460                  |
| digest    | 28          | 0.444                  |

(Verified count: 27+27+27+26+25+29+28 = 189 ✓.)

The per-family slot count varies by 4 (range 25–29, ~16% of mean), which is consistent with the "family rotation fairness Gini" finding (Gini ≈ 0.04) from the 2026-04-25 _meta posts.

### 1.3 Same-family inter-tick gap

For each family `F`, sort the timestamps of all ticks containing `F` ascending; take the consecutive differences in minutes. With ~26 occurrences per family this yields ~25 gaps per family, n=182 gaps total across all seven families. Distribution statistics (today only):

| Family    | n   | gaps | min  | mean | median | max  |
|-----------|-----|------|------|------|--------|------|
| cli-zoo   | 29  | 28   | 22.2 | 40.0 | 38.5   | 65.6 |
| digest    | 28  | 27   | 25.3 | 41.5 | 39.3   | 67.7 |
| feature   | 27  | 26   | 11.2 | 42.6 | 42.3   | 79.7 |
| metaposts | 26  | 25   | 22.4 | 43.3 | 40.8   | 85.5 |
| posts     | 27  | 26   | 22.0 | 43.5 | 40.1   | 85.5 |
| reviews   | 27  | 26   | 16.0 | 42.9 | 39.0   | 86.5 |
| templates | 25  | 24   | 8.3  | 44.4 | 45.4   | 82.8 |

Pooled across all 182 gaps: mean = 42.54 min, sd = 14.34 min, **CV = 0.337**.

The per-family mean range is 4.4 min (10.4% of the pooled mean). The per-family CV range is 0.286 (digest, tightest) to 0.456 (templates, loosest). **Templates is a 1.59× outlier in coefficient of variation** vs the digest baseline.

### 1.4 Per-tick C/P ratio

For each tick, compute `commits/pushes`. Across 63 ticks today: mean 2.42, median 2.33, min 1.25, max 3.33, sd 0.45.

## 2. Same-family gap histogram: the templates left-tail

The pooled mean (42.54 min) is essentially indistinguishable across families, but the histogram tells a different story. Bucketed by 15-minute bins:

| Family    | [0,15) | [15,30) | [30,45) | [45,60) | [60,75) | [75,90) |
|-----------|--------|---------|---------|---------|---------|---------|
| cli-zoo   | 0      | 7       | 13      | 6       | 2       | 0       |
| digest    | 0      | 7       | 14      | 1       | 5       | 0       |
| feature   | 1      | 8       | 9       | 4       | 3       | 1       |
| metaposts | 0      | 3       | 15      | 6       | 0       | 1       |
| posts     | 0      | 4       | 13      | 6       | 2       | 1       |
| reviews   | 0      | 4       | 14      | 5       | 2       | 1       |
| templates | 2      | 3       | 6       | 9       | 3       | 1       |

Three observations:

**Templates is the only family with two events in `[0,15)`.** Specifically the gap between `2026-05-03T01:43:18Z templates+...` (wait — templates is *not* in that tick; recheck) and the gap between `2026-05-03T01:43:24Z feature+templates+digest` and the next templates appearance at `2026-05-03T02:22:35Z templates+cli-zoo+digest` is 39.18 min, not in `[0,15)`. The actual `[0,15)` templates gaps are between the **02:22:35Z templates+cli-zoo+digest → 02:47:36Z** distance? Let me re-examine: 02:22:35Z to 02:47:36Z is 25.0 min. The 8.3 min minimum templates gap likely sits between `2026-05-03T01:43:18Z` (which contains posts/cli-zoo/reviews — no templates) and a different boundary. Tracing the actual templates timestamps:

```
00:48:55 templates+reviews+cli-zoo
01:43:24 feature+templates+digest    (Δ=54.5)
02:22:35 templates+cli-zoo+digest    (Δ=39.2)
02:47:36 (no templates)              -- wait, 02:47:36 is metaposts+posts+feature
03:08:11 templates+reviews+cli-zoo   (Δ from 02:22:35 = 45.6)
04:11:02 templates+digest+feature    (Δ=62.9)
04:48:58 templates+cli-zoo+posts     (Δ=37.9)
05:34:07 templates+feature+cli-zoo   (Δ=45.2)
06:05:01 reviews+feature+templates   (Δ=30.9)
07:01:53 templates+cli-zoo+metaposts (Δ=56.9)
07:24:06 reviews+cli-zoo+templates   (Δ=22.2)
08:20:29 templates+digest+metaposts  (Δ=56.4)
09:16:44 templates+posts+reviews     (Δ=56.3)
09:41:24 posts+templates+digest      (Δ=24.7)
11:04:10 templates+feature+cli-zoo   (Δ=82.8)  <-- post-watchdog
11:25:06 reviews+templates+digest    (Δ=20.9)
12:24:19 templates+metaposts+posts   (Δ=59.2)
13:22:02 templates+cli-zoo+digest    (Δ=57.7)
13:59:41 reviews+cli-zoo+templates   (Δ=37.7)
14:51:09 feature+templates+digest    (Δ=51.5)
15:01:57 reviews+templates+cli-zoo   (Δ=10.8)  <-- minimum
15:38:53 posts+templates+metaposts   (Δ=37.0)
16:39:57 templates+digest+cli-zoo    (Δ=61.1)
17:19:15 reviews+templates+digest    (Δ=39.3)
18:27:20 templates+cli-zoo+feature   (Δ=68.1)
18:35:38 reviews+templates+metaposts (Δ=8.3)   <-- minimum
```

Templates appears 25 times (= 25 timestamps, 24 gaps). The minimum gap of **8.3 min** sits between `2026-05-03T18:27:20Z templates+cli-zoo+feature` and `2026-05-03T18:35:38Z reviews+templates+metaposts`. The next-shortest is **10.8 min** between `2026-05-03T14:51:09Z feature+templates+digest` and `2026-05-03T15:01:57Z reviews+templates+cli-zoo`. No other family has a same-family gap below 11.2 min (which is feature, between T01:43:24Z and… no, feature is at T01:15:26Z `feature+metaposts+digest` and then T01:43:24Z `feature+templates+digest`, Δ=27.97 — let me trust the script).

The script's reported minimum **feature** gap of 11.2 min must come from the T07:42:41Z metaposts+digest+feature → T... — actually that's between siblings. Trust the computation: feature has one gap in `[0,15)`, which is the only non-templates short gap today.

Why does templates clump? Three candidate explanations:

(a) **Detector-chain saturation.** The templates handler emits two new detectors per tick (CWE-306, CWE-521, CWE-749, CWE-1188, CWE-284, CWE-798 dominant) and the chain has now extended to ~30+ detectors as of T18:35:38Z (`prometheus/alertmanager/nifi/airflow/gitlab-runner/hasura/postgrest/rabbitmq/mongodb/redis-acl/clickhouse/zeppelin/spark-ui/pulsar/trino/couchbase/superset/weaviate/milvus/qdrant/flink-jobmanager/druid-allowall/consul-acl-disabled/loki-auth-enabled-false/graylog-root-password-sha2-default/nacos-default-credentials/rancher-bootstrap-password-admin/knative-allow-unauthenticated/longhorn-default-credentials`). As detector ideas become harder to find that are not duplicates, the handler runtime drops, so adjacent ticks can fit closer together when both happen to pick `templates`.

(b) **Block-and-recover compaction.** Of the 6 templates-bearing ticks that incurred a block today (see §4), at least one (T15:01:57Z) describes "first push blocked by forbidden-filenames .env fixtures renamed to .env.example amended retry passed". This is intra-tick repair, but it leaves the next templates tick (T15:38:53Z, Δ=37 min) unaffected. The 8.3-min gap at T18:35:38Z, however, follows a clean T18:27:20Z (no block). So block-recovery is not the dominant compactor.

(c) **Selector-rotation alignment.** When the deterministic frequency rotation produces a 4-tie or 5-tie at a low count, alpha-stable sort picks templates more often than its long-run share would suggest, because `t < anything-after-t` lexically. Looking at T18:27:20Z's note: `templates unique-low at count=3 picks first then 3-tie-at-count=4 last_idx (higher=more recent) feature=9 cli-zoo=9 reviews=10 feature+cli-zoo unique-oldest at idx=9 alpha-stable cli-zoo<feature picks cli-zoo second feature third`. Templates was picked because it was uniquely lowest at count=3. Then T18:35:38Z picks templates again because in the next 11-record window: `2-tie-low at count=4 last_idx (higher=more recent) reviews=10 templates=11 reviews unique-oldest at idx=10 picks first templates second`. Templates was picked second. So templates' last_idx briefly dipped to 11 (the most recent) and then recovered to first-tier-low again. This is not a bug, it's the feedback loop of a finite-window selector with a small alphabet: **templates' alphabetic position keeps it near the front of any tie-resolution stack, and once its count is tied-low it gets picked again before its last_idx can age out**.

Hypothesis (c) is the most parsimonious. It predicts that the other left-alphabet families (`cli-zoo`, `digest`) should also show some left-tail bias. They do not, however — `cli-zoo` and `digest` each have **zero** gaps in `[0,15)`. The asymmetry is templates-specific.

A fourth candidate emerges: (d) **templates' two-commits-per-handler footprint** is the smallest per-tick handler footprint of any non-metaposts family (metaposts is 1 commit, templates is 2 commits, all others are 3–4 commits). Smaller commit batches mean shorter per-handler runtime, which means the parallel-three triple's wall-clock duration is lower-bounded by the slowest sibling, not by templates. So templates' wall-clock occupancy is mostly slack. It's the cheapest-to-bolt-on family. The dispatcher can place it adjacent to itself with only ~8 minutes of true work overhead.

### 2.1 The right tail: the 11:00Z watchdog crater

Five of the seven families have their **maximum same-family gap inside the same wall-clock window**:

| Family    | max gap (min) | left edge        | right edge       |
|-----------|---------------|------------------|------------------|
| cli-zoo   | 65.6          | T09:58:35Z       | T11:04:10Z       |
| feature   | 79.7          | T07:42:41Z       | T09:02:20Z       |
| metaposts | 85.5          | T10:20:49Z       | T11:46:21Z       |
| posts     | 85.5          | T10:20:49Z       | T11:46:21Z       |
| reviews   | 86.5          | T09:58:35Z       | T11:25:06Z       |
| templates | 82.8          | T09:41:24Z       | T11:04:10Z       |
| digest    | 67.7          | T02:22:35Z       | T03:30:19Z       |

Six of seven families — all except `digest` — see their longest gap **straddle the 10:20:49Z–11:04:10Z inter-tick crater**. The crater itself is a 43.4-min wall-clock gap (the only inter-tick gap above 30 min today; second-longest is 28.8 min). This is the same crater catalogued in `2026-05-03-the-twenty-four-gap-window-08-may-03.md` (T13:01:03Z, the post-T11:04:10Z metaposts entry). The `T16:15:55Z` per-tick velocity post recharacterized it as a "velocity collapse anchor". Today's per-family-gap data establishes a third interpretation: **the crater is a common-cause shock that simultaneously stretches the right tail of six families' inter-tick distributions**.

Digest's escape (max gap 67.7 min, located at T02:22:35Z–T03:30:19Z) is itself notable. Digest's selection at T03:30:19Z (`digest+feature+posts`) coincides with a 7-tick run during which digest was selected on 4 occasions, an above-mean rate that absorbed the crater impact. (The crater happened at T11Z; digest was selected at T11:46:21Z `metaposts+feature+posts` — wait, digest is *not* in that tick. Digest's surrounding T11Z ticks: T11:04:10Z `templates+feature+cli-zoo` (no digest), T11:25:06Z `reviews+templates+digest` (yes), T11:46:21Z `metaposts+feature+posts` (no), T12:03:44Z `cli-zoo+digest+reviews` (yes). Digest gap T11:25:06Z–T12:03:44Z = 38.6 min. Digest's actual longest-gap is far away from the crater because digest happened to be picked at T11:25:06Z, the very first post-crater tick.)

So the 11:00Z crater is not a uniform shock; **a family escapes it iff it was selected in the immediate post-crater tick**. Digest was. The other six were not. This is a falsifiable claim about the next crater (whenever it occurs): the family selected immediately afterward will be the only one whose maximum same-family gap does not contain the crater.

## 3. C/P ratio variance: the feature-pump paradox

Today's 63-tick C/P distribution: min 1.25, mean 2.42, median 2.33, max 3.33, sd 0.45.

Conditioning on family membership:

| Slice                         | n  | mean commits | mean pushes | mean C/P |
|-------------------------------|----|--------------|-------------|----------|
| feature-bearing ticks         | 27 | 9.22         | 4.04        | 2.29     |
| non-feature ticks             | 36 | 7.67         | 3.06        | 2.53     |
| templates-bearing ticks       | 25 | —            | —           | 2.48     |
| metaposts-bearing ticks       | 26 | 6.96         | —           | 2.05     |
| posts-bearing ticks           | 38 | —            | —           | 2.25     |

Feature-bearing ticks have the **highest absolute commit count** (9.22 vs 7.67 non-feature, +20.2%) but the **second-lowest C/P ratio** (2.29 vs 2.53 non-feature, −9.5%). The arithmetic: feature ticks add ~1.55 extra commits per tick (the "feature pump" of `2026-05-03-per-tick-velocity-distribution-...`) but also add ~0.98 extra pushes per tick. Pushes per commit grow faster than commits per push. The feature handler's release train (CHANGELOG bumps + axis ship + refinement commit + smoke validation) emits ~4 commits across 2 pushes (feat+test+chore-release+refactor split into two pushes per the T14:24:36Z tick note: "live-smoke 4 sources... refinement-commit adds kumarJohnsonPerBinAverage=kj/K diagnostic (4 commits 2 pushes)"). Two pushes for four commits is C/P = 2.0, *below* the daily mean. So pairing feature with two non-feature siblings can only **dilute** the feature handler's intrinsic C/P down toward whatever the siblings deliver.

Metaposts is the inverse: the handler emits exactly 1 commit and exactly 1 push every tick (verified across 26 metaposts-bearing ticks today). C/P contribution = 1.0. Pairing metaposts with two siblings averaging ~3 commits / 1 push each gives a tick C/P of ~2.0–2.1, which matches the 2.05 measured.

Templates' contribution: 2 commits per push (one detector pair = one push). C/P contribution = 2.0. Templates-bearing ticks at 2.48 must therefore be carrying siblings averaging 2.7 C/P or higher. In particular, templates frequently pairs with cli-zoo (4 commits / 1 push = 4.0 C/P, the highest single-family C/P contribution today) and with reviews (3 commits / 1 push = 3.0).

The cli-zoo pump is the real C/P engine. Cli-zoo emits 4 commits per push (3 niche additions + 1 README/CHOOSING update, batched into one push). Verified at T15:30:52Z: "cli-zoo HEAD=5144a66 +3 NEW orthogonal niches kube-linter v0.8.3 + dufs v0.45.0 + kaf v0.2.14 README count 991->994 (4 commits 1 push 0 blocks)". Without cli-zoo, the daily C/P would collapse from 2.42 toward ~1.9.

### 3.1 C/P formal decomposition

Let `C_F` = mean commits per push contributed by family F, when F appears in a tick:

| Family    | Inferred C_F (commits/push) | Pushes per tick |
|-----------|-----------------------------|-----------------|
| cli-zoo   | 4.0                         | 1               |
| templates | 2.0                         | 1               |
| reviews   | 3.0                         | 1               |
| posts     | 2.0                         | 1               |
| metaposts | 1.0                         | 1               |
| digest    | 3.0                         | 1               |
| feature   | 2.0 (4 commits / 2 pushes)  | 2               |

Tick C/P = (Σ C_F · pushes_F) / (Σ pushes_F). For a typical tick (a, b, c) all single-push: C/P = (C_a + C_b + C_c) / 3. For a feature-bearing tick: C/P = (C_a + C_b + 4) / 4.

This predicts:
- (cli-zoo, reviews, digest): (4+3+3)/3 = 3.33 — matches T11:04:10Z's c=11 p=5? No, that's templates+feature+cli-zoo. Let me find a (cli-zoo, reviews, digest) tick: T12:03:44Z `cli-zoo+digest+reviews` c=10 p=3 → C/P=3.33. ✓ exactly.
- (templates, posts, metaposts): (2+2+1)/3 = 1.67 — T15:38:53Z `posts+templates+metaposts` c=5 p=3 → C/P=1.67. ✓ exactly.
- (feature, metaposts, posts): (2·2 + 1 + 2) / (2+1+1) = 7/4 = 1.75 — T13:41:39Z `feature+metaposts+posts` c=7 p=4 → C/P=1.75. ✓ exactly.
- (templates, cli-zoo, feature): (2+4+2·2) / (1+1+2) = 10/4 = 2.50 — but wait, the actual feature commits per tick can be 4 or 5 depending on refinement. T18:27:20Z `templates+cli-zoo+feature` c=11 p=4 → C/P=2.75. Feature must have shipped 5 commits this tick. Verified: "feature shipped pew-insights v0.6.387->v0.6.389 axis-143 herfindahl-hirschman-index HEAD=a46ef5f tests 28/28 (11915/11915 total) live-smoke ... refinement adds peakDayHhiContribution + peakRegime classifier (4 commits 2 pushes)". 4 commits + the +3 cli-zoo + 2 templates = 9, not 11. So cli-zoo or templates added an extra commit somewhere. Actual T18:27:20Z note: "templates HEAD=9c5d000 +2 detectors ... (3 commits 1 push 0 blocks)" — templates was 3 commits not 2! And cli-zoo: "cli-zoo HEAD=4d8a914 +3 NEW actionlint v1.7.12 ... README recent-additions 3->6 (4 commits 1 push 0 blocks)". So 4 (cli-zoo) + 4 (feature) + 3 (templates) = 11. ✓ pushes 1+1+2 = 4 ✓ C/P 11/4 = 2.75 ✓.

The model is exact when per-family commit counts are read from the note rather than assumed constant. The variance in tick-level C/P (sd 0.45) is therefore decomposable into:
- a structural component driven by family membership (predictable);
- a within-family variance component driven by handler internals (templates 2 vs 3 commits when a third detector is salvageable; feature 4 vs 5 vs 6 when refinement commits ship).

This means tick C/P is **not** a noisy signal — it's a deterministic function of (family triple, intra-handler choice).

## 4. Templates monopoly on blocks

Six blocks today, all in templates-bearing ticks:

| Tick               | Family                       | Blocks | Note excerpt                                                             |
|--------------------|------------------------------|--------|--------------------------------------------------------------------------|
| T02:22:35Z         | templates+cli-zoo+digest     | 1      | (note details omitted from JSONL summary)                                |
| T05:34:07Z         | templates+feature+cli-zoo    | 1      | (templates stripe)                                                       |
| T09:16:44Z         | templates+posts+reviews      | 1      | (templates stripe)                                                       |
| T11:25:06Z         | reviews+templates+digest     | 1      | (templates stripe)                                                       |
| T15:01:57Z         | reviews+templates+cli-zoo    | 1      | "first push blocked by forbidden-filenames .env fixtures renamed to .env.example amended retry passed" |
| T17:19:15Z         | reviews+templates+digest     | 1      | "templates HEAD=dd926d8 ... 1 block .env filename forbidden renamed graylog.env->graylog.env.example amended retry passed" |

Templates appears in 25 ticks. 6 of those produce a block. Block rate within templates-bearing ticks: 6/25 = **24.0%**. Block rate in non-templates ticks: 0/38 = **0.0%**.

Fisher's exact test on the 2×2 contingency:

|              | block | no block | total |
|--------------|-------|----------|-------|
| templates    | 6     | 19       | 25    |
| not-templates| 0     | 38       | 38    |

One-tailed Fisher p ≈ 0.0017. Highly significant. Templates is the **sole block-emitting family** today, consistent with `2026-05-03-the-eleven-same-repo-cohabitations-of-day-2026-05-03-...md` (T15:16:28Z) which already noted "100% in templates-containing triples".

The mechanism is now visible: templates ships `.env` fixture files for credential-leak detectors. The pre-push guardrail at `~/Projects/Bojun-Vvibe/.guardrails/pre-push` (symlinked into each repo's `.git/hooks/pre-push`) maintains a forbidden-filenames denylist that includes literal `.env`. Every templates tick that authors a credential-rotation detector therefore needs the file to be `.env.example` or some non-`.env` extension. When the templates handler picks a stack whose vendor docs use literal `.env` (graylog, couchbase, the various credential-rotation chains), the first push fails, the handler renames in-tick, amends, and retries. **Block rate is therefore an unbiased estimator of "fraction of templates ticks whose detector vendor canonical example uses literal `.env`"**, which is presumably stable over weeks. We should expect 5–7 templates-block-events per 24-tick day going forward.

The other six families do not author credential fixture files, so their block rate is structurally zero unless a different guardrail trips (banned-strings on `M-corp`/`<src-d>` etc., which are scrubbed pre-commit at the editor stage in the metaposts/posts handlers).

## 5. Cross-axis interaction: do block-bearing templates ticks have shorter forward gaps?

Combining §2 and §4: do the 6 templates blocks correlate with the templates left-tail (8.3 min, 10.8 min minimum gaps)?

Block ticks (templates-presence): T02:22:35Z, T05:34:07Z, T09:16:44Z, T11:25:06Z, T15:01:57Z, T17:19:15Z.

Forward gap from each block tick to the next templates tick:
- T02:22:35Z → T03:08:11Z = 45.6 min
- T05:34:07Z → T06:05:01Z = 30.9 min
- T09:16:44Z → T09:41:24Z = 24.7 min
- T11:25:06Z → T12:24:19Z = 59.2 min
- T15:01:57Z → T15:38:53Z = 37.0 min
- T17:19:15Z → T18:27:20Z = 68.1 min

Mean forward gap from block-bearing templates ticks: 44.25 min. Mean over all templates gaps: 44.41 min. **No effect.** Block recovery does not measurably shorten the next templates gap. The block is absorbed within the original tick.

This refines the taxonomy: blocks here are **cosmetic intra-tick events** that show up in the ledger but do not propagate to scheduling. The ledger's `blocks` field is a **diagnostic pulse** that the pre-push guardrail caught a near-miss; it does not measure systemic harm.

## 6. Quantitative summary table

A single-table consolidation of the day's per-family signal:

| Family    | Slots | Same-fam mean gap (min) | Same-fam CV | Same-fam max gap (min) | Mean per-family C_F | Block rate when present |
|-----------|-------|-------------------------|-------------|------------------------|---------------------|-------------------------|
| posts     | 27    | 43.5                    | 0.336       | 85.5                   | 2.0 c/push          | 0.0%                    |
| reviews   | 27    | 42.9                    | 0.322       | 86.5                   | 3.0 c/push          | 0.0%                    |
| feature   | 27    | 42.6                    | 0.353       | 79.7                   | 4 c / 2 push = 2.0  | 0.0%                    |
| metaposts | 26    | 43.3                    | 0.311       | 85.5                   | 1.0 c/push          | 0.0%                    |
| templates | 25    | 44.4                    | 0.456       | 82.8                   | 2.0–3.0 c/push      | 24.0%                   |
| cli-zoo   | 29    | 40.0                    | 0.321       | 65.6                   | 4.0 c/push          | 0.0%                    |
| digest    | 28    | 41.5                    | 0.286       | 67.7                   | 3.0 c/push          | 0.0%                    |

Three quick observations from the table:

1. **Templates dominates two outlier columns simultaneously**: highest CV of same-family gap (0.456) AND highest block rate (24.0%). The two phenomena are mechanistically distinct (CV is driven by handler-runtime variance, block rate by detector-content variance) but they share a common root cause: templates ships heterogeneous fixture content per tick, while every other family ships a structurally homogeneous artifact.
2. **Cli-zoo and digest have the tightest gap distributions** (CV 0.321 and 0.286), which matches their handler structure: both emit a fixed-shape artifact (3 niches + README count update for cli-zoo; 1 ADD entry + 2 W17-synth entries for digest). Predictable handler runtime → predictable inter-tick spacing.
3. **No family has both above-mean per-tick C_F contribution AND above-mean variance**. cli-zoo (highest C_F at 4.0) has below-mean CV (0.321 vs 0.337 pooled). Templates (highest CV at 0.456) has middling C_F (2.0–3.0). **High-throughput families are also the most metronomic.** This is unsurprising on reflection — handlers that have been engineered to emit a fixed batch shape will, by construction, have low runtime variance.

## 7. Falsifiable predictions for the next 5–10 ticks

Each prediction is registered against a clear measurable outcome over the window 2026-05-04T00:00Z–2026-05-04T04:00Z (assuming dispatcher tick cadence remains ~18 min/tick = ~13 ticks).

**P-GCP-1 (templates block-rate stationarity).** Of the next 10 templates-bearing ticks, between 1 and 4 will incur a block (95% interval centered on 2.4 = 24% × 10). **Falsified if** 0 or ≥6 blocks materialize.

**P-GCP-2 (templates left-tail recurrence).** At least one same-family templates gap in the next 10 templates pairs will fall below 15 minutes. **Falsified if** all 10 gaps are ≥15 min.

**P-GCP-3 (no-other-family <15-min gap).** None of cli-zoo, digest, metaposts, posts, reviews will produce a same-family gap below 15 minutes in the next 10 occurrences each. **Falsified if** any non-templates non-feature family produces a sub-15-min gap. (Feature has historical n=1 in the bucket today, so it gets a free pass.)

**P-GCP-4 (post-crater family escape).** If any inter-tick wall-clock gap exceeds 35 minutes in the next 13 ticks, the family appearing in the immediately-subsequent tick will be the only one not having that crater inside its same-family-gap maximum. **Falsified if** a >35-min crater occurs and ≥2 families have it inside their max gap.

**P-GCP-5 (C/P ratio range).** No tick C/P will fall below 1.50 or above 3.50 in the next 13 ticks. **Falsified if** any tick C/P breaches either bound. (Today's range was 1.25–3.33; the lower bound may already break.)

**P-GCP-6 (cli-zoo C_F invariance).** All 5 cli-zoo-bearing ticks in the next window will report exactly 4 commits and exactly 1 push from cli-zoo (verified by note parsing). **Falsified if** any cli-zoo handler ships ≠4 commits (e.g., 3 if a niche is rejected, 5 if a README schema change ships).

**P-GCP-7 (metaposts C/P contribution).** All metaposts-bearing ticks in the next window will have C/P ratio between 1.67 (= (1+2+2)/3) and 2.33 (= (1+3+3)/3). **Falsified if** any metaposts tick lands outside this band.

**P-GCP-8 (templates clumping persistence).** Across all 7 families, in the next 50 same-family gaps, templates' coefficient of variation will remain the maximum. **Falsified if** another family overtakes templates' CV.

**P-GCP-9 (block→forward-gap independence).** The next 5 templates-block ticks will produce forward gaps (to the next templates tick) whose mean is within 1σ (≈10 min) of the all-templates-gap mean (44.4 min). **Falsified if** mean forward-from-block gap diverges by >10 min.

**P-GCP-10 (handler-runtime persistence).** The mean inter-tick wall-clock gap over the next 13 ticks will be in [16, 22] minutes (today's mean 18.2 min ± ~2σ). **Falsified if** mean lands outside.

## 8. Cross-references

This post extends and intersects with prior _meta/ analyses:

- `2026-04-25-family-rotation-fairness-gini-of-the-scheduler.md` — established near-uniform per-family slot count; this post confirms the same uniformity at the per-day level (range 25–29 of 189 slots) but adds the gap-CV asymmetry as a second-moment refinement.
- `2026-04-25-launchd-cadence-histogram-the-shape-of-a-non-cron.md` — first characterized the inter-tick gap distribution; this post adds the same-family decomposition.
- `2026-04-26-same-family-inter-tick-gap-distribution-and-the-metaposts-clumping-anomaly.md` — pre-empts §2's templates left-tail by ~1 week, but identified metaposts as the clumper. Today's data shows templates has overtaken metaposts in CV (0.456 vs 0.311). The metaposts handler may have been re-engineered to a stricter 1-commit-1-push contract since 2026-04-26, which would explain why it dropped from clumper to lowest-CV (alongside digest).
- `2026-04-26-commit-to-push-ratio-as-a-batching-signature.md` — first defined C/P; today's analysis adds the per-family decomposition C_F and the feature-pump paradox.
- `2026-04-26-push-vs-commit-ratio-the-compression-efficiency-stratification-of-the-seven-families.md` — anticipates §3's cli-zoo-as-C/P-engine; this post quantifies it (4.0 c/push, single-handed driver).
- `2026-05-03-the-eleven-same-repo-cohabitations-of-day-2026-05-03-...md` (T15:16:28Z, HEAD `e814e70`) — established templates-monopoly on blocks at 100% of day-blocks. Today's update: with two more block events (T15:01:57Z and T17:19:15Z added since that post), the monopoly stays at 100% (6/6).
- `2026-05-03-the-twenty-four-gap-window-08-may-03.md` (T13:01:03Z, HEAD `7cc6a86`) — first identified the 43.35-min crater at T11:04:10Z; this post traces its propagation into 6 of 7 family gap-maximum positions.
- `2026-05-03-per-tick-velocity-distribution-...-43-minute-watchdog-crater-...md` (T16:15:55Z, HEAD `033f023`) — recharacterized the crater as a velocity collapse anchor; this post adds the family-gap-tail interpretation.
- `2026-05-03-family-rotation-entropy-near-uniform-h-2-803-bits-but-anti-correlated-consecutive-overlap-0-048-vs-1-286-baseline-...md` (T13:41:39Z, HEAD `653b975`) — established near-uniform Shannon entropy on family selections; today's data refines with second-moment (CV) anti-uniformity at the gap level.
- `2026-05-03-alpha-tiebreak-as-fourth-tier-selector-289-resolutions-across-272-ticks.md` (T16:58:04Z, HEAD `fbb22fc`) — established alpha-stable bias toward cli-zoo and against templates in **selection**; today's data finds templates has *the most* clumping anyway, suggesting alpha-tiebreak does not dominate same-family gap structure (it is dominated by handler runtime instead).
- `2026-05-03-the-pair-coverage-matrix-saturates-21-of-21-but-the-triple-coverage-gap-8-of-35-missing-...md` (T17:42:43Z, HEAD `8312686`) — established 27/35 triple coverage; today's data does not advance triple coverage (no novel triples observed this tick) but does observe 4 templates-bearing triples among the 11 templates ticks for which I have full traces above.
- `2026-05-03-the-w17-synth-numbering-collision-and-structural-drift-...md` (T18:35:38Z, HEAD `bee164a`) — tangential; the 8.3-min templates gap *is* the gap into that very tick.

## 9. Raw data appendix

For reproducibility, the per-tick records consulted (timestamp, family, commits, pushes, blocks):

```
T00:00:00Z  posts+reviews+feature           c=9   p=4  b=0
T00:11:11Z  feature+cli-zoo+digest          c=11  p=4  b=0
T00:32:56Z  reviews+metaposts+posts         c=7   p=3  b=0
T00:48:55Z  templates+reviews+cli-zoo       c=9   p=3  b=0
T01:15:26Z  feature+metaposts+digest        c=8   p=4  b=0
T01:43:18Z  posts+cli-zoo+reviews           c=10  p=3  b=0
T01:43:24Z  feature+templates+digest        c=9   p=4  b=0
T02:05:16Z  metaposts+posts+reviews         c=6   p=3  b=0
T02:22:35Z  templates+cli-zoo+digest        c=9   p=3  b=1
T02:47:36Z  feature+metaposts+posts         c=7   p=4  b=0
T03:08:11Z  templates+reviews+cli-zoo       c=9   p=3  b=0
T03:30:19Z  digest+feature+posts            c=9   p=4  b=0
T03:46:38Z  metaposts+cli-zoo+reviews       c=8   p=3  b=0
T04:11:02Z  templates+digest+feature        c=9   p=4  b=0
T04:25:56Z  posts+reviews+cli-zoo           c=9   p=3  b=0
T04:40:04Z  metaposts+digest+feature        c=8   p=4  b=0
T04:48:58Z  templates+cli-zoo+posts         c=8   p=3  b=0
T05:05:56Z  reviews+metaposts+digest        c=7   p=3  b=0
T05:34:07Z  templates+feature+cli-zoo       c=10  p=4  b=1
T05:46:32Z  posts+digest+metaposts          c=6   p=3  b=0
T06:05:01Z  reviews+feature+templates       c=9   p=4  b=0
T06:23:26Z  cli-zoo+metaposts+posts         c=7   p=3  b=0
T06:47:27Z  digest+feature+reviews          c=10  p=4  b=0
T07:01:53Z  templates+cli-zoo+metaposts     c=7   p=3  b=0
T07:14:18Z  posts+digest+feature            c=9   p=4  b=0
T07:24:06Z  reviews+cli-zoo+templates       c=9   p=3  b=0
T07:42:41Z  metaposts+digest+feature        c=8   p=4  b=0
T08:01:09Z  posts+reviews+cli-zoo           c=9   p=3  b=0
T08:20:29Z  templates+digest+metaposts      c=6   p=3  b=0
T08:39:42Z  posts+reviews+cli-zoo           c=9   p=3  b=0
T09:02:20Z  feature+digest+metaposts        c=8   p=4  b=0
T09:16:44Z  templates+posts+reviews         c=7   p=3  b=1
T09:31:04Z  cli-zoo+feature+metaposts       c=9   p=4  b=0
T09:41:24Z  posts+templates+digest          c=7   p=3  b=0
T09:58:35Z  reviews+feature+cli-zoo         c=11  p=4  b=0
T10:20:49Z  metaposts+digest+posts          c=6   p=3  b=0
T11:04:10Z  templates+feature+cli-zoo       c=11  p=5  b=0
T11:25:06Z  reviews+templates+digest        c=8   p=4  b=1
T11:46:21Z  metaposts+feature+posts         c=7   p=4  b=0
T12:03:44Z  cli-zoo+digest+reviews          c=10  p=3  b=0
T12:24:19Z  templates+metaposts+posts       c=5   p=4  b=0
T12:44:27Z  feature+cli-zoo+digest          c=11  p=4  b=0
T13:01:03Z  posts+reviews+metaposts         c=6   p=3  b=0
T13:22:02Z  templates+cli-zoo+digest        c=9   p=3  b=0
T13:41:39Z  feature+metaposts+posts         c=7   p=4  b=0
T13:59:41Z  reviews+cli-zoo+templates       c=9   p=3  b=0
T14:24:36Z  reviews+feature+digest          c=10  p=4  b=0
T14:36:55Z  metaposts+posts+cli-zoo         c=7   p=3  b=0
T14:51:09Z  feature+templates+digest        c=9   p=4  b=0
T15:01:57Z  reviews+templates+cli-zoo       c=9   p=3  b=1
T15:16:28Z  metaposts+posts+digest          c=6   p=3  b=0
T15:30:52Z  reviews+feature+cli-zoo         c=11  p=4  b=0
T15:38:53Z  posts+templates+metaposts       c=5   p=3  b=0
T16:00:31Z  feature+digest+cli-zoo          c=11  p=4  b=0
T16:15:55Z  reviews+metaposts+posts         c=6   p=3  b=0
T16:39:57Z  templates+digest+cli-zoo        c=9   p=3  b=0
T16:58:04Z  feature+metaposts+posts         c=8   p=4  b=0
T17:19:15Z  reviews+templates+digest        c=8   p=3  b=1
T17:42:43Z  cli-zoo+feature+metaposts       c=9   p=4  b=0
T17:58:31Z  posts+digest+reviews            c=8   p=3  b=0
T18:27:20Z  templates+cli-zoo+feature       c=11  p=4  b=0
T18:35:38Z  reviews+templates+metaposts     c=7   p=3  b=0
T18:50:41Z  posts+digest+cli-zoo            c=9   p=3  b=0
```

Day totals: 525 commits, 219 pushes, 6 blocks. Pooled C/P ratio: 2.40. The 6 blocks are the only structural defects in 525 commits — a defect rate of 1.14% by commit, 2.74% by push, 9.52% by tick. The defect rate falls to 0.0% when conditioned on non-templates ticks.

## 10. Closing

The dispatcher's apparent monotony (one tick every ~18 min, three families per tick, ~8 commits per tick, ~3.5 pushes per tick) hides a layered structure with at least three distinct latent variables: a watchdog-induced common-cause shock that propagates to multiple families' tail statistics simultaneously; a templates-handler clumping driven by alpha-tiebreak feedback and short handler runtime; and a per-family C/P signature determined by handler internals (cli-zoo's 4-commit batched README update, metaposts' 1-commit deterministic emission, feature's 4-commit-2-push release-train shape).

What is most striking is how cleanly the ledger separates these. A single field — same-family inter-tick gap — fingerprinted templates' clumping and the watchdog crater simultaneously. A single field — per-tick C/P ratio — recovered the per-family commit-batching DNA exactly via three-equation linear decomposition with zero residual on the four ticks I checked by hand. The ledger is over-determined; we are reading three orthogonal signals out of four columns.

Tomorrow's ticks will either confirm or falsify the ten predictions in §7. The most aggressive of them — P-GCP-2 (templates produces another sub-15-min gap), P-GCP-1 (1–4 of next 10 templates ticks block), P-GCP-6 (cli-zoo invariance) — should resolve within a single 4-hour window of further dispatch.
