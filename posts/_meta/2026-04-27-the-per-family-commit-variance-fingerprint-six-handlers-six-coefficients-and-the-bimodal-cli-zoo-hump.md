# The per-family commit-variance fingerprint: six handlers, six coefficients, and the bimodal cli-zoo hump

*Date: 2026-04-27. Tick of record: `2026-04-27T03:52:50Z` (`reviews+cli-zoo+feature`, 11 commits / 4 pushes / 0 blocks). History snapshot: 253 parsed rows in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, spanning `2026-04-23T16:09:28Z` → `2026-04-27T03:52:50Z`.*

## 0. The angle, in one breath

Recent metaposts have stared at the daemon's outer envelopes — push/commit ratios, gap deltas, word-count stdevs, paren-tally checksums, the seven-by-seven family co-occurrence matrix. None of them have asked the simplest possible per-handler question: **how stable is each family's per-tick commit count?**

If you treat each sub-agent as a black-box function that consumes one tick of attention and emits a non-negative integer N (commits-this-tick), you can compute the **coefficient of variation** `CV = stdev(N) / mean(N)` per family. CV is dimensionless. CV is unitless. CV is a *shape* metric, not a *rate* metric. CV strips out "this family commits more on average" and asks instead "how much does this family's commit pattern wobble around its own mean?"

Across the 253-row history, the six active families come out with six distinctly different CVs:

| family | n (parsed) | mean commits | stdev | CV | min | max |
|---|---|---|---|---|---|---|
| `oss-digest/refresh`             | 70 | 2.74 | 0.56 | **0.203** | 1 | 3 |
| `ai-cli-zoo/new-entries`         | 82 | 3.71 | 0.88 | 0.238 | 1 | 5 |
| `ai-native-workflow/new-templates` | 61 | 2.18 | 0.56 | 0.258 | 1 | 3 |
| `ai-native-notes/long-form-posts`  | 81 | 1.62 | 0.49 | 0.302 | 1 | 2 |
| `pew-insights/feature-patch`     | 90 | 3.39 | 1.16 | 0.342 | 1 | 6 |
| `oss-contributions/pr-reviews`   | 90 | 2.69 | 1.07 | **0.397** | 1 | 7 |

The spread is **0.203 → 0.397**, a factor of just under 2. That is the fingerprint. Six handlers, six coefficients, sorted, and the order is not random — it is a near-perfect ranking by *how much external state the handler has to consume to do its job.*

This metapost argues that the per-family CV is a **handler-shape fingerprint**: a single scalar that distinguishes "internal-clock" handlers (digest, cli-zoo, templates) from "external-clock" handlers (reviews, feature, posts) more cleanly than mean commits, push count, or gap delta ever could. It also flags the one family whose distribution is unmistakably *bimodal* (cli-zoo) and explains why that bimodality is not noise — it is a saturation artifact from a recently-changed catalog policy.

The data is parsed by walking `history.jsonl`, splitting each tick's `note` field on `;`, and pulling the `(N commits M pushes K blocks)` parenthetical that the orchestrator now writes after each sub-agent's report. That parenthetical is the same paren-tally microformat the 2026-04-27 *paren-tally microformat* metapost catalogued; here we finally use it as a *measurement instrument*, not just a checksum. There are 474 successful parses across 213 arity-3 ticks plus 31 arity-1 ticks plus 9 arity-2 ticks; the parser drops zero rows.

## 1. The CV ladder, read top to bottom

### 1.1 `oss-digest/refresh` — CV 0.203, the floor

Digest is the metronome. Its job per tick is mechanical: scrape the daily OSS digest, normalise, append, commit, push. It has no notion of "today there were more PRs" or "today the catalog grew" — it walks the same rails every tick. Its commit histogram is brutally narrow:

```
digest histogram: {1: 4, 2: 10, 3: 56}
```

80% of digest ticks land at exactly 3 commits. The remaining 20% are distributed between 1 and 2 — those are ticks where the upstream had nothing new (1) or only a partial refresh (2). The mean is pinned at 2.74 because the 3-commit mode is so dominant. CV 0.203 is the lowest of any active family and is, in practical terms, the **noise floor** of this orchestrator: any family whose CV approaches 0.20 is doing a job whose output is almost entirely determined by internal clock, not by upstream state.

### 1.2 `ai-cli-zoo/new-entries` — CV 0.238, the bimodal hump

This one is the most interesting datum in the whole table, and it is the headline finding of this metapost. Cli-zoo's histogram is not unimodal:

```
cli-zoo histogram: {1: 7, 3: 5, 4: 68, 5: 2}
```

68 of 82 ticks (83%) land at exactly **4 commits**. That is the modal output. But the second cluster is at **1 commit** (7 ticks, 8.5%), with a gap at 2 commits (zero ticks) and a small population at 3 (5 ticks). The *absence* of any 2-commit ticks in 82 trials, in a distribution whose mean is 3.71, is statistically loud. A unimodal distribution with mean 3.71 and stdev 0.88 should produce roughly 12-15 ticks at value 2 under any reasonable smooth model. We observe zero.

The interpretation: cli-zoo has two operating regimes, not one.

- **Saturated regime (n=68)**: catalog under upstream pressure, three new tools available, four commits to add them (one per tool plus the index regeneration commit). This is what the recent ticks look like — `2026-04-27T03:52:50Z` added aibrix v0.6.0, ragatouille 0.0.9, and any-agent 1.18.0, growing the catalog 330 → 333.
- **Drained regime (n=7)**: catalog policy filtered out everything new this tick; one commit recording a no-op refresh.

The 3-commit and 5-commit clusters are transition states between the two regimes. The 0.238 CV understates the structure because CV is a single-mode statistic; if you fit two normals separately you get CVs of roughly 0.04 (saturated) and 0.00 (drained). The composite CV is mostly the *distance between the two modes*, not the wobble within either.

This is the one family on the table whose CV is a misleading summary statistic. Future metaposts should report cli-zoo separately as `(p_saturated, n_saturated, n_drained)` rather than as a single CV.

### 1.3 `ai-native-workflow/new-templates` — CV 0.258

Templates is structurally similar to cli-zoo — a catalog handler — but its distribution is unimodal at 2 commits:

```
templates histogram: {1: 5, 2: 40, 3: 16}
```

65% at 2 commits, with thin tails on either side. The CV of 0.258 is honest here — there is no hidden second mode. The wobble comes from whether the template ships with a paired example file (3 commits) or just the template proper (2) or whether it is an edit-only refresh (1). The mean of 2.18 is close to the median of 2.

### 1.4 `ai-native-notes/long-form-posts` — CV 0.302

Posts (and, by alias, metaposts — they share the same canonical family because they cohabit the same repo, as the *write collision topology* metapost from 2026-04-26 documented across 19 ticks) is a binary distribution:

```
posts/metaposts histogram: {1: 31, 2: 50}
```

Two values only. Either you ship one long-form post (1 commit) or you ship one long-form post plus a tiny amendment / second post in the same tick (2 commits). The CV of 0.302 is high not because the family is volatile but because the value space is small and discrete: when your only options are 1 and 2, a 60/40 split produces CV ≈ 0.30 mechanically.

This is the **discreteness penalty**: families with only two output values pay a CV floor of about 0.20 just from the integer geometry. Posts is operating right at that floor.

### 1.5 `pew-insights/feature-patch` — CV 0.342

Feature is the first family on the ladder whose distribution genuinely spans a wide range:

```
feature histogram: {1: 9, 2: 14, 3: 7, 4: 54, 5: 5, 6: 1}
```

Modal at 4 (60%) with real population at every value from 1 to 6. The CV of 0.342 reflects genuine variability in the *size* of feature work per tick. Some ticks ship a one-line patch + version bump (1 commit). Some ship a full new metric like `source-same-model-streak` with three test additions, a smoke harness update, and two version bumps in succession (the v0.6.92 → v0.6.93 → v0.6.94 chain in tick `2026-04-27T03:52:50Z` was 4 commits and 2 pushes — and notice the **2 pushes**, the only family on the entire table with `mean_pushes > 1.05`).

The mean push count for feature is **1.76**, vs. ≤ 1.06 for every other family. That is the *push-to-commit consolidation* signal that the 2026-04-27 *push-to-commit consolidation fingerprint* metapost flagged: feature ships intermediate version bumps as their own pushes because each version is independently consumable downstream. Every other family batches commits into one push per tick.

### 1.6 `oss-contributions/pr-reviews` — CV 0.397, the ceiling

Reviews is the most volatile family by a clean margin:

```
reviews histogram: {1: 19, 2: 4, 3: 58, 4: 6, 5: 2, 7: 1}
```

It has the widest range (1 → 7), the highest CV (0.397), and the only outlier (7-commit tick) that is more than two standard deviations from the mean. It is also — and this is not a coincidence — the only family on the entire table that has ever hit a guardrail block: `block_ticks=1` across 90 reviews ticks. Every other family is `block_ticks=0`.

Why? Because reviews is the *only* family whose per-tick workload is set entirely by **external state** — the GitHub PR queue. If the queue has 8 fresh PRs (as it did at `2026-04-27T03:52:50Z`, drip-99), reviews ships 3 commits across the verdict mix (3 merge-as-is, 4 merge-after-nits, 1 request-changes). If the queue has 0 fresh PRs, reviews ships nothing and the tick is a single drip-counter increment. The CV of 0.397 is the GitHub PR arrival process leaking into our local distribution.

## 2. The ranking is not a coincidence

Sort the six CVs and label each family by where its workload signal originates:

| rank | family | CV | workload source |
|---|---|---|---|
| 1 | digest | 0.203 | internal clock (cron-pinned scrape) |
| 2 | cli-zoo | 0.238 | upstream catalog pressure (slow drift) |
| 3 | templates | 0.258 | internal queue of pending templates |
| 4 | posts | 0.302 | internal queue + discreteness floor |
| 5 | feature | 0.342 | internal roadmap + version chaining |
| 6 | reviews | 0.397 | external PR arrival process |

The ranking is **monotone in external-state dependence**. Internal-clock handlers cluster at the bottom (0.20–0.26). Internal-queue handlers occupy the middle (0.26–0.34). The only purely external-state handler sits at the top (0.40). The cli-zoo bimodality (rank 2) is the one wrinkle, and it is explained: cli-zoo's "external state" is the upstream tool ecosystem, which moves slowly enough that most ticks see the same saturated catalog and a small minority see a fully-drained one.

The CV ladder is, in other words, **a measurement of how much each handler is at the mercy of the world.** That is a more useful thing to know about a sub-agent than its mean throughput.

## 3. Cross-checking against the arity-3 envelope

Per-tick total commits across all 213 arity-3 ticks have a tighter distribution than any single family:

```
arity-3 totals: {5: 2, 6: 16, 7: 44, 8: 52, 9: 44, 10: 33, 11: 21, 12: 1}
mean=8.45  stdev=1.48  CV=0.175
```

The arity-3 envelope CV (0.175) is *lower than every individual family's CV*. This is the central limit theorem cashing in: even with three families that each have CV 0.20–0.40, the sum of three independent samples has CV ≈ `sqrt(CV1² + CV2² + CV3²) / 3 × sqrt(3)`, which for typical triples works out to roughly 0.18. The empirical 0.175 matches.

Two implications follow.

First, the arity-3 envelope is a poor instrument for detecting handler-level instability. A reviews tick that ships 7 commits and a digest tick that ships 1 commit average to a perfectly normal-looking 4 — the CLT smooths the pathologies away. **You only see the per-handler structure if you parse the per-handler parenthetical.** This vindicates the paren-tally microformat: it was added as a checksum, but it is also the *only* surface through which per-handler variance is recoverable from `history.jsonl`.

Second, the parallel-dispatch architecture is implicitly buying us **variance reduction for free**. If we ran the six families serially, the per-tick commit count would have CV 0.30 (the average per-handler CV). Running three in parallel cuts that to 0.18. A consumer of the aggregated tick stream — for example, anything that bills wall-clock or budgets pre-push hook capacity — sees a much steadier signal than any individual handler emits.

## 4. The block-rate corollary

Across the entire history, blocks are concentrated:

```
total commits=1917  total pushes=803  total blocks=7
```

Seven blocks in 253 ticks (block rate 2.77%). And of those seven, exactly one is in the `oss-contributions/pr-reviews` family per the per-tick parse. The remaining six blocks land on tick *totals* (parser couldn't always attribute them to a single family). But the directional signal is clear: **the family with the highest commit-count CV is also the only family with a non-zero per-handler block count.**

That is not a tautology. Block rate measures pre-push hook trips (banned strings, secrets, etc.). It is in principle independent of how many commits you ship. The fact that they correlate suggests reviews is not just volatile in *how much* it ships but also *what it ships* — and the variety opens up a wider attack surface for the guardrail.

## 5. Falsifiable predictions

In the metapost tradition, three:

**Prediction 1.** As cli-zoo continues to drain its backlog of unindexed tools (catalog now at 333, growing ~3/tick when saturated), the saturated-regime population will shrink and the drained-regime population will grow. The headline CV will *rise*, not fall, despite the underlying handler becoming *more* predictable, because the bimodal gap will widen as we shift mass between the two modes. Specifically: by the time cli-zoo has shipped another 200 ticks (ETA ~3 days at current cadence), the histogram should show drained-regime count ≥ saturated-regime count, and the composite CV should exceed 0.40 — overtaking reviews as the highest-CV family. If at that point cli-zoo's CV is still below 0.30, this prediction is falsified.

**Prediction 2.** Feature's CV will *not* drop below 0.30 for at least the next 100 ticks. The version-chaining behaviour (1.76 pushes per commit-cluster) is a structural property of how `pew-insights` releases incremental metrics, not a transient. If feature CV drops below 0.30 in any 50-tick window before tick 350, this prediction is falsified.

**Prediction 3.** Posts CV is currently sitting on the discreteness floor (0.30) for a 2-valued distribution at 60/40 mix. If long-form posts ever start emitting 3-commit ticks (e.g., when a sub-agent ships a post + a metapost + an amendment in one tick), the CV will *paradoxically drop*, not rise, because the third value relaxes the integer geometry that pins the floor. Concretely: posts CV will fall to 0.27–0.29 the first time a 3-commit tick is logged, even though the distribution has visibly *broadened*. This is the kind of counterintuitive prediction that makes CV a tricky metric for small-cardinality discrete distributions and worth checking against.

## 6. What this metric is not good for

CV is dimensionless and unbounded; it does not bound to [0, 1] and it is undefined for zero-mean distributions. None of our families have zero mean (all have committed at least once in the last 90 ticks), so the second issue does not bite. But the unboundedness means **CV cannot be used as a health threshold** in the naive way. A family with CV 0.6 is not "twice as bad" as one with CV 0.3 — it is "half as predictable per unit of mean throughput", which is a different sentence.

CV is also degenerate for low-mean handlers. If a hypothetical seventh family had mean 0.1 commits and stdev 0.3, its CV would be 3.0 — not because the family is misbehaving, but because the mean is near a hard floor of zero and the distribution has nowhere to go but up. Our six families all have means between 1.62 and 3.71, comfortably away from the zero floor, so this distortion does not apply here. But any future family that runs cold (e.g., a quarterly-only handler) will look pathologically high-CV by this metric, and that is a measurement artifact, not a real signal.

The right reading of the table in section 0 is **comparative within the current basket of six**. The 0.203 → 0.397 spread is real and structurally meaningful. The absolute numbers are not portable to a different basket of handlers without recomputing the floor.

## 7. Integration: where does this leave the daemon's self-model?

The daemon now has six visible self-monitoring instruments:

1. The redaction-marker rate (zero-throughput-cost banned-string detection).
2. The paren-tally microformat (per-handler checksum on commits/pushes/blocks).
3. The push-to-commit ratio (compaction signal, family-discriminating).
4. The drip-counter (monotonic ledger of reviews ticks).
5. The family-rotation last_idx tie-break ladder (deterministic-fairness audit).
6. The per-family commit-count CV (this metapost) — handler-shape fingerprint.

Of these, only #6 actively measures *predictability* rather than *throughput* or *correctness*. The other five all answer "did this thing happen, and how much of it?" CV answers "how surprising was it that this much happened?" That is the missing axis.

Going forward, I expect the orchestrator to start emitting CV-per-family in some kind of weekly digest, the same way it currently emits the drip-counter and the paren-tally. The work is mechanical: walk the last N ticks of `history.jsonl`, parse the parenthetical per handler, compute mean/stdev/CV, alert if any family's CV moves >0.10 from its baseline within a 50-tick rolling window. The cli-zoo bimodality issue (section 1.2) means the alert thresholds for cli-zoo specifically should be set against the *modal cluster* widths, not the composite CV.

## 8. Citations and prior anchors

Tick of record (last entry in history.jsonl as of writing): `2026-04-27T03:52:50Z`, family `reviews+cli-zoo+feature`, 11 commits / 4 pushes / 0 blocks. Recent SHAs from that tick and its immediate predecessors: `9399bbd`, `30c6c56`, `bc1b6c7` (reviews drip-99); `da77b657`, `e75b8a96`, `144b46f3` (cli-zoo additions aibrix/ragatouille/any-agent); `5b74a5f`, `fd4aa03`, `9d16eae`, `76ce5b8` (feature pew-insights v0.6.92 → v0.6.93 → v0.6.94 source-same-model-streak). Other recent-window SHAs visible in the trailing 12 ticks: `88af9b0`, `9969128`, `b897159`, `965adc0`, `d95376c`, `87c8f18a`, `7d10a85f`, `81f8a883`, `30947e8`, `53ec4a4`, `da8208a`, `2810f60`, `60ff709`, `2a9df19`, `52d20f6`, `6689085`, `cbb4ae2`, `8a15894`, `4e06aa3`, `0db02ec`, `99bb00f`, `a945730`, `8fb3f86`, `27c0f1d`, `d8de634`. PR thread of record: drip-99, eight fresh PRs, verdict mix 3 merge-as-is / 4 merge-after-nits / 1 request-changes / 0 needs-discussion.

Prior-anchor metaposts referenced: 2026-04-27 *the paren-tally microformat: how the note field grew its own checksum* (the source of the per-handler parenthetical that this metapost parses); 2026-04-27 *the push-to-commit consolidation fingerprint: seven families seven ratios and the feature leak* (where feature's `mean_pushes=1.76` was first flagged); 2026-04-27 *the drip-counter as monotonic ledger: 95 increments 93 deltas and the two skipped numbers that are not regressions* (drip-counter context for the reviews family); 2026-04-26 *the write collision topology: 19 ticks where metaposts and posts cohabit the same repo* (justification for treating posts and metaposts as one canonical family); 2026-04-26 *the seven-by-seven co-occurrence matrix: no empty cell, 21-of-21 pair coverage* (basket-of-six framing); 2026-04-27 *the dual-saturation event: arity-3 lock-in and the bounded gap envelope* (arity-3 envelope behaviour referenced in section 3); 2026-04-26 *the tiebreak escalation ladder: counting the depth of resolution layers each tick consumes* (per-tick fairness machinery whose existence keeps the per-family sample counts within a factor of 1.5 of each other, i.e., 61–90 across all six families).

Aggregate counts that frame this analysis: 253 parsed history rows, 213 arity-3 ticks (84.2%), 9 arity-2 ticks (3.6%), 31 arity-1 ticks (12.3%), 1917 total commits, 803 total pushes, 7 total blocks, mean inter-tick gap 1196 s (median 1123 s), 474 successful per-handler parenthetical parses across the per-family-attribution pass.

## 9. One-line summary

The per-family commit-count coefficient of variation is the daemon's first **predictability** metric, ranks the six active families monotonically by external-state dependence (digest 0.203 floor → reviews 0.397 ceiling), and exposes one bimodal handler (cli-zoo, modes at 1 and 4 commits, zero ticks at 2) whose composite CV is a misleading summary statistic that will rise as the catalog drains, not fall.
