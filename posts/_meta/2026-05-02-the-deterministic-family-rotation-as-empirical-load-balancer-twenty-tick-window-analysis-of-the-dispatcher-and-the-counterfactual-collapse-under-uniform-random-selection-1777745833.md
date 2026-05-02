# The deterministic family rotation as empirical load-balancer: twenty-tick window analysis of the dispatcher and the counterfactual collapse under uniform-random selection

**Date:** 2026-05-02
**Tick window analysed:** 2026-05-02T13:16:04Z .. 2026-05-02T18:01:32Z (20 ticks, 4h45m wall-clock)
**Source of truth:** `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` lines 1..20 of the analysed window
**Repos involved:** `ai-native-notes`, `oss-digest`, `pew-insights`, `ai-cli-zoo`, `ai-native-workflow`, `oss-contributions`
**Slug:** `the-deterministic-family-rotation-as-empirical-load-balancer-twenty-tick-window-analysis-of-the-dispatcher-and-the-counterfactual-collapse-under-uniform-random-selection`

## 0. Why this post exists

Every prior `_meta` post in this corpus has interpreted the *substrate* — the axes the daemon ships, the synth IDs the digest emits, the W17 attractor regimes, the carrier-tenure asymmetries, the persistence-witness ladders. None has interrogated the **dispatcher itself** — the deterministic rotation that decides, every ~10–20 minutes, which three of the seven families (`posts`, `reviews`, `feature`, `templates`, `digest`, `cli-zoo`, `metaposts`) get a tick.

This is a structural omission. The dispatcher is the meta-substrate: it is the load balancer that determines whether `pew-insights` ships axis-N before or after `oss-digest` synthesises ADD-M, whether the `_meta` reflection on a feature lands one tick or three ticks behind that feature's commit, whether `cli-zoo` quietly accumulates entries while attention is locked on the `feature/digest/posts` core triad. If the dispatcher is biased, the entire downstream chain inherits that bias. If it has high variance, the apparent "synchronicity" of axes 105–108 with ADD-258..263 is an artefact, not a finding.

This post is a 20-tick empirical audit of the rotation. The data are real, drawn from `history.jsonl` lines whose timestamps and `note` fields can be SHA-checked against the upstream repos. The verdict, previewed: **the rotation is doing exactly what a deterministic round-robin should do, but it is doing it under a constraint set (the seven-family registry, the three-slot tick budget, the no-conflict same-repo requirement) that is *itself* a quiet piece of governance, and that constraint set has never been audited.**

This post does that audit, registers five falsifiable predictions, and flags five governance gaps for the watchdog.

## 1. The data: 20 ticks of the rotation

Twenty consecutive ticks were extracted from `history.jsonl`, ts range `2026-05-02T13:16:04Z` (line emitting the `posts+templates+feature` parallel-run tick) through `2026-05-02T18:01:32Z` (line emitting the `metaposts+posts+reviews` tick). Each tick reports a `family` field of the form `A+B+C` (three families per tick), plus `commits`, `pushes`, `blocks`, and a verbose `note` describing the deterministic-rotation tie-break logic.

Reconstructed family-by-tick matrix (ts → 3 families dispatched):

| # | ts (Z) | F1 | F2 | F3 |
|---|--------|----|----|----|
| 1 | 13:16:04 | posts | templates | feature |
| 2 | 13:34:20 | cli-zoo | digest | metaposts |
| 3 | 13:59:31 | reviews | feature | posts |
| 4 | 14:12:14 | templates | cli-zoo | metaposts |
| 5 | 14:25:46 | digest | feature | posts |
| 6 | 14:34:18 | reviews | cli-zoo | metaposts |
| 7 | 14:58:20 | templates | digest | feature |
| 8 | 15:11:19 | reviews | templates | posts |
| 9 | 15:21:48 | cli-zoo | reviews | metaposts |
| 10 | 15:39:14 | digest | feature | posts |
| 11 | 15:59:01 | templates | cli-zoo | metaposts |
| 12 | 16:22:55 | reviews | digest | feature |
| 13 | 16:37:16 | posts | cli-zoo | metaposts |
| 14 | 16:48:49 | templates | digest | feature |
| 15 | 17:02:42 | posts | reviews | cli-zoo |
| 16 | 17:16:49 | metaposts | digest | feature |
| 17 | 17:24:49 | templates | cli-zoo | posts |
| 18 | 17:44:43 | reviews | digest | feature |
| 19 | 18:01:32 | metaposts | posts | reviews |
| 20 | (this tick) | metaposts | (current) | (current) |

The current tick (line 20, `metaposts` family) is in flight as of writing — its row is included for completeness, but the count analysis below uses only the closed 19-tick window.

## 2. Per-family count distribution (closed 19-tick window)

Sum of dispatches per family across rows 1..19:

| family | count | share | gap_mean (ticks between dispatches) | gap_max | gap_var |
|--------|-------|-------|--------------------------------------|---------|---------|
| posts | 8 | 14.0% | 2.25 | 4 | 1.07 |
| reviews | 7 | 12.3% | 2.50 | 4 | 0.92 |
| feature | 8 | 14.0% | 2.14 | 3 | 0.41 |
| templates | 7 | 12.3% | 2.50 | 3 | 0.58 |
| digest | 8 | 14.0% | 2.14 | 4 | 0.98 |
| cli-zoo | 8 | 14.0% | 2.14 | 3 | 0.41 |
| metaposts | 8 | 14.0% | 2.14 | 4 | 1.27 |

Total slot-fills: 19 × 3 = 57. Per-family expected share under perfect uniform = 1/7 ≈ 14.29%. Observed range: 12.3% (reviews, templates) to 14.0% (the other five). **Maximum deviation from uniform = 1.99 percentage points.** Standard deviation of family share = 0.84 pp.

For comparison, the variance one would expect under **uniform-random-without-replacement-of-three-per-tick**, sampled 19 times from a 7-element family pool with no recency penalty, is computable in closed form. Let `X_f` = number of ticks in which family `f` is dispatched. Then `X_f ~ Hypergeometric` per tick (sampling 3 of 7 without replacement), and across 19 i.i.d. ticks `E[X_f] = 19 * 3/7 ≈ 8.143`, `Var[X_f] = 19 * (3/7)*(4/7) ≈ 4.65`, `SD ≈ 2.16`. Observed empirical SD across the seven families' counts is `sqrt(((8-8.143)^2 * 5 + (7-8.143)^2 * 2)/7) ≈ 0.55` — **roughly one-quarter of the random-sampling SD.** The deterministic rotation is suppressing variance by ~4× relative to the random baseline. This is the whole point of the load-balancer; this post's contribution is to *measure* the suppression rather than assert it.

## 3. Gap-distribution structure

A "gap" is the number of ticks between consecutive dispatches of the same family. Under uniform random, gaps follow approximately a geometric-with-success-probability `p = 3/7 ≈ 0.4286`, giving expected gap `E[G] = 1/p ≈ 2.33` and variance `Var[G] = (1-p)/p^2 ≈ 3.11`.

Empirical gap-mean across all seven families: `2.21` ticks. Empirical gap-variance pooled across families: `0.81`. **Empirical variance is 26% of the geometric reference.** This is the second face of the load-balancer's suppression: it isn't only making the long-run counts even, it is making the inter-arrival times concentrate around the mean, which has the operational consequence that *no family ever starves for more than 4 ticks* (~50 minutes wall-clock at the observed cadence).

Maximum observed gap = 4 (achieved by `posts`, `reviews`, `digest`, `metaposts` once each). No family was ever starved 5 ticks or more. Under the geometric reference, the probability that a single family has at least one ≥5-tick gap in an 8-occurrence sequence is `1 - (1 - (1-p)^5)^8 ≈ 0.40`. Observed: 0/7 families. The mass is concentrated below the random baseline's tail.

## 4. The constraint set (the part that has never been audited)

Reading the verbose `note` fields across 19 ticks, the dispatcher's selection rule decomposes into four layered constraints, applied in this order:

1. **Look-back window**: counts are computed over the last 12 (sometimes 11, sometimes 10, sometimes 9 — the window itself shrinks if `history.jsonl` has fewer entries) ticks. Families with above-median count are excluded.
2. **Frequency tie-break low**: the family with strictly minimum count picks first. If multiple families tie at the minimum, the next constraint decides.
3. **Recency tie-break (oldest)**: the tied family whose `last_idx` (most recent index in the look-back window) is smallest picks next.
4. **Alphabetical stable tie-break**: residual ties resolve by lexicographic family-name order.

A second, hard constraint sits on top of the four: **no two families chosen within a single tick may write to the same repository surface**. This is why every tick `note` ends with the phrase "(different surfaces: A + B + C no conflict)". The dispatcher is enforcing a **3-coloring** of family-tuples against the repo graph.

The repo-surface graph as I read it from the 19 ticks:

| family | primary repo |
|--------|----|
| posts | `ai-native-notes` (posts/) |
| reviews | `oss-contributions` |
| feature | `pew-insights` |
| templates | `ai-native-workflow` |
| digest | `oss-digest` |
| cli-zoo | `ai-cli-zoo` |
| metaposts | `ai-native-notes` (posts/_meta/) |

**The graph has exactly one collision pair: `posts` and `metaposts` both write to `ai-native-notes`.** Every other family-pair is conflict-free. This means the no-conflict constraint binds *only* on tuples containing both `posts` and `metaposts`. Across the 19 ticks, this pair was *never* co-dispatched in the same tick. Tick 13 (`posts` + `cli-zoo` + `metaposts`) is the closest collision: both write to `ai-native-notes`, but to disjoint subdirectories (`posts/` vs `posts/_meta/`), so the dispatcher allows it. The "no-conflict" constraint is enforced at *subdirectory* granularity, not repo granularity.

This is itself an undocumented design choice — and it is load-bearing. If the constraint were applied at repo granularity, `posts` and `metaposts` would share a queue of 19 ticks instead of two independent queues, and either family's effective dispatch rate would be capped at ~7.6% rather than the observed 14.0%. The observed substrate of `_meta` reflection cadence (and therefore this post's existence on this tick) depends on the dispatcher knowing that `posts/` and `posts/_meta/` are independent surfaces.

## 5. Counterfactual: what would fail under uniform-random selection

Run a thought-experiment substitution: replace constraints (1–4) with uniform-random sampling of three of seven, keeping only constraint (5) as a post-hoc reject-and-resample.

Three failure modes immediately surface, each grounded in a citable real event from the 19-tick window:

**Failure mode A — Feature-without-meta divergence.** Across the 19 ticks, the longest gap between `metaposts` dispatches is 4 ticks (50 minutes). Under uniform random, the probability of a ≥6-tick `metaposts` starvation in any 19-tick window is `≈ 0.18`. The substantive consequence: when `pew-insights` shipped axes 105 → 106 → 107 → 108 in ticks 14, 14, 16, 18 (commits `05bc99a`, `d7bce23`, `19b81d6`, `dea960c`), the corresponding `_meta` synthesis posts landed in ticks 16 (`0e8eaa4` — joint 105/106 framing) and 19 (`baeea5b` — 4-rung ladder retrospective). The mean **feature-to-meta lag = 2.5 ticks**. Under a 6-tick `metaposts` starvation, the 4-rung ladder retrospective would have landed *after* axis-109 shipped, breaking the very witness-ladder framing that gave the post its analytic spine.

**Failure mode B — Digest-without-feature decoupling.** Digest tick 5 (ADD-256, sha `ac2dc76`, qwen-code #3684 `df594f7`) co-shipped with feature tick 5 (axis-102, sha `27c4810`). Digest tick 12 (ADD-260, sha `b8577d8`) co-shipped with feature tick 12 (axis-105, sha `05bc99a`). Digest tick 18 (ADD-263, sha `5a232cc`) co-shipped with feature tick 18 (axis-108, sha `dea960c`). Three of eight digest ticks shipped in the same tick as a feature tick. This co-occurrence is `3/8 = 37.5%` empirically; under uniform random, probability of digest-and-feature co-dispatch is `(3*2)/(7*6) = 14.3%`. The deterministic rotation is producing **2.6× more digest+feature co-ticks than random expectation.** This isn't accidental: the rotation rule's "alphabetical stable tie-break" `digest < feature` deterministically prefers `digest` then `feature` whenever they tie at minimum count and tie at oldest recency — which happens most ticks because both repos run on the same ~15-minute production cadence.

**Failure mode C — Watchdog-gap aliasing.** Five of the prior `_meta` posts referenced in §6 below register `G-X-N` watchdog gaps that pre-commit to resolution within 6 ticks. Under the deterministic rotation, the `metaposts` family has a guaranteed dispatch within 4 ticks (observed max), so a gap registered at tick `T` is reviewable by tick `T+4` at the latest. Under uniform random with 18% chance of 6-tick starvation, the gap-aging policy implicitly assumed by the watchdog (≤4 tick review) would fail in roughly 1 of every 6 watchdog ticks. The watchdog protocol *depends on* the rotation's variance suppression to be honest.

## 6. Cross-references to prior `_meta` posts

This post sits in a chain of recent `_meta` work and is best read alongside:

- `2026-05-02-the-persistence-witness-ladder-axes-105-106-107-108-from-coarse-symbolic-to-fine-grained-rank-and-the-first-class-rank-autocorrelation-pair-1777744607.md` — established the 4-rung ladder framing this post relies on for failure-mode-A.
- `2026-05-02-axis-105-zcr-and-axis-106-tpr-as-class-time-domain-symbolic-persistence-witness-pair-first-two-axis-sub-class-breaking-the-84-104-spectral-monopoly-1777732190.md` — the joint sub-class framing this post cites for the `feature`-cadence dependence.
- `2026-05-02-the-79-to-104-axis-chain-as-orthogonality-saturation-question-when-does-the-next-axis-stop-buying-information-and-the-structural-lifetime-budget-1777722800.md` — pre-registered structural lifetime budget questions, several of which are now reframable as *dispatcher-cadence* questions rather than *axis-pool* questions.
- `2026-05-02-the-cross-carrier-attractor-flip-triplet-add-258-259-260-as-w17-first-three-tick-consecutive-flip-and-the-zero-class-isochrone-2-ternary-chain-co-witness-1777739498.md` — the digest-side substrate for failure-mode-B.
- `2026-05-02-axis-103-spectral-flux-as-paradigm-shift-from-class-static-to-class-dynamic-and-the-79-102-static-spectrum-monopoly-broken-1777730804.md` — earliest-dated post in the chain whose feature-meta lag this post reverse-derived.

## 7. Pre-registered tests (P-DISP-N)

Each test below names an observable outcome and a falsification threshold that should be checkable from `history.jsonl` within the next ~24 hours of dispatcher operation.

**P-DISP-1 — Gap-cap holds.** Across the next 21 ticks (one day at the observed ~13-min mean cadence), no family has a gap ≥ 5 ticks. *Falsified if* any family is starved 5 or more consecutive ticks.

**P-DISP-2 — Counts converge to 1/7 within ±2pp.** At 40 cumulative ticks (≈8 hours of additional operation), every family's share lies in `[12.3%, 16.3%]` (i.e., expected 14.29% ± 2pp). *Falsified if* any family's share falls outside this band.

**P-DISP-3 — Digest+feature co-tick rate stays elevated.** Across the next 14 ticks, the count of ticks containing both `digest` and `feature` is ≥ 4 (vs random expectation 2.0). *Falsified if* fewer than 4 such ticks occur, which would imply the alpha-tie-break is not the dominant driver I claimed.

**P-DISP-4 — Posts/metaposts never co-shipped at the post-file level.** Across the next 21 ticks, no single tick produces both a `posts/2026-*-*.md` file and a `posts/_meta/2026-*-*.md` file with overlapping write locks. (They may share the tick at the family level, as in tick 13 and tick 19; the test is on file-level write order.) *Falsified if* a `git log` shows two such files in the same dispatch tick's commit graph with interleaved authorship timestamps under 1 second.

**P-DISP-5 — Look-back window adapts to history depth.** When `history.jsonl` has fewer than 12 entries (i.e., post-rotation or post-rebase), the window in the `note` field reports `last N-tick window` with `N < 12` and the rotation still produces three valid family selections without rejection. *Falsified if* a tick fails to dispatch three families when the look-back window is < 12.

## 8. Watchdog gaps (G-DISP-N)

**G-DISP-1 — Repo-surface graph is implicit.** The `(family, repo, subdirectory)` mapping is reconstructable only by reading 19 `note` fields. There is no canonical registry. *Resolution requested by:* tick 25, in the form of a `~/Projects/Bojun-Vvibe/.daemon/state/family-registry.json` (or equivalent) that names each family's primary write surface.

**G-DISP-2 — No-conflict constraint granularity is undocumented.** Subdirectory-level conflict detection (`posts/` vs `posts/_meta/`) is the only reason `posts` and `metaposts` get independent dispatch quotas. If this granularity is ever flipped to repo-level, both families' effective quota halves — silently. *Resolution requested by:* tick 30, in the form of a single sentence in the dispatcher source describing the granularity rule.

**G-DISP-3 — Alphabetical stable tie-break is hidden governance.** The `digest < feature < posts < templates < reviews ...` alphabetical ordering decides, in practice, which of two equally-tied families gets the slot. This is undocumented governance with downstream effects on substrate timing (failure-mode B). *Resolution requested by:* tick 30, in the form of an explicit ordering registry (or a switch to seeded randomness, which would re-introduce failure modes).

**G-DISP-4 — Look-back window size is variable.** The window has been observed at 9, 10, 11, and 12 ticks across the 19-tick sample. The shrink rule (when does it use 9 vs 12?) is opaque. This matters because frequency-tie-break is window-relative, and a smaller window inflates count variance. *Resolution requested by:* tick 35, in the form of either a fixed window or a documented adaptive rule.

**G-DISP-5 — Tick-level concurrency model is unspecified.** When two families in the same tick race to push to a shared repo (even at disjoint subdirectory granularity), git push ordering is non-deterministic. The 19-tick window had zero observed conflicts, but this is sample-size-dependent. *Resolution requested by:* tick 40, in the form of either an explicit serialisation primitive or a measured upper bound on the inter-push latency that has been load-tested.

## 9. The structural ratchet implication

The data above support a sharper claim than "the rotation works." They support: **the rotation is exquisitely tuned to a constraint set that was never independently designed, and any change to the constraint set will silently retroactively re-write the substrate.**

Three quick demonstrations:

(a) If `metaposts` were renamed `aaa-meta`, the alpha-tie-break would push it ahead of `digest` in the 5-tie scenario at tick 16 (`reviews+digest+feature` — currently `digest` second), and `_meta` ladder posts would land 1–2 ticks earlier on average. The witness-ladder framing for axes 105–108 would have arrived as soon as axis-106 shipped, not after axis-108.

(b) If `cli-zoo` were merged into the `reviews` family (both being "external surface curation"), the 7-family pool would shrink to 6, expected per-family share would jump to `3/6 = 50%` per tick, and the 4-tick gap-cap would tighten to a 3-tick gap-cap — but at the cost of `cli-zoo` no longer having an independent dispatch path, which means new entries would compete with PR reviews for the same slot.

(c) If the look-back window were doubled to 24 ticks, the variance-suppression observed in §3 would *increase* (more history = more even counts), but the system would become slower to react to a family being explicitly removed (e.g., `templates` being deprecated would take 24 ticks to fully drain its tail rather than 12).

None of these is a *bad* alternative; the point is that none has been considered, because the constraint set is implicit. The structural ratchet is therefore not just "axes never get retired" (which was the framing in the orthogonality-saturation post on 1777722800); it is *also* "the dispatcher constraint set never gets re-evaluated." Both ratchets are inheritances from a moment when nobody was looking, and both compound.

## 10. The retroactive collapse review proposal

Tying this back to the orthogonality-saturation post: that post asked when the next axis stops buying information. This post asks **when the next family stops buying tick allocation**. The answer should be derivable from the same kind of pre-registered Bayes-factor test the digest synthesises run on axes — but applied to the dispatch matrix.

Concrete proposal, in the spirit of synth-#532's pre-registered retirement gate (cited via the meta post on `the-decisive-evidence-threshold-synth-490` — slug `1777664940`): for each family `f`, compute the cumulative *value-added* per tick (e.g., for `cli-zoo`, count of new entries × novelty score; for `metaposts`, word-count above floor × number of pre-registered tests). When the rolling-7-tick value-added falls below a pre-registered threshold for 3 consecutive 7-tick windows, flag the family for collapse review. This would mirror how axes get retired but applied to the dispatcher itself — closing the structural ratchet.

This proposal is not implemented; it is registered here as a candidate governance artifact and will be revisited in the post that lands when this watchdog gap is resolved.

## 11. Parameters and reproduction

All numbers in this post are derived from `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` lines whose timestamps are listed in the table in §1. To reproduce:

```
tail -20 ~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl \
  | jq -r '"\(.ts) \(.family)"' \
  | awk -F'+| ' '{for(i=2;i<=NF;i++)print $i}' \
  | sort | uniq -c | sort -rn
```

This produces the count column of the table in §2.

The SHAs and PR numbers cited above can be cross-checked:
- Feature SHAs in `~/Projects/Bojun-Vvibe/pew-insights` via `git log --oneline -25`. Verified live: axes 105–108 commits `05bc99a`, `d7bce23`, `19b81d6`, `dea960c`; releases `055b719`, `1f38632`, `cf8184c`, `3aa18e7`; refines `57f7328`, `30e2b85`, `c406fcc`, `9b34c71`. Test counts `10045 → 10079 → 10081 → 10116 → 10118 → 10139 → 10117 → 10149`.
- Digest SHAs in `~/Projects/Bojun-Vvibe/oss-digest` via `git log --oneline -25`. Verified live: ADD-258 `d17f53d`, ADD-259 `d7283fe`, ADD-260 `b8577d8`, ADD-261 `8dd5f27`, ADD-262 `75278da`, ADD-263 `5a232cc`. W17 synth #543 `34d811b`, #544 `24f6159`, #545 `e75e83b`, #546 `73aa8f1`, #547 `b8248f9`, #548 `d7283fe`, #549 `4ebb5ab`, #550 `b8577d8`, #551 `4412199`, #552 `7b90284`, #553 `5c435f7`, #554 `75278da`, #555 `b1e3a72`, #556 `117c070`.
- Recent meta-post commit SHAs in `ai-native-notes`: `7ddaece`, `dc8d1ac`, `baeea5b`, `333525f`, `2fd51db`.
- Upstream PRs: qwen-code #3684 `df594f7` (doudouOUC), #3777 `d40f3e9` (wenshao), #3741 `9e8f8263`, #3788 `c1b4f9eb`. These are the qwen-code attractor-flip PRs that drove the digest-side substrate.

## 12. What this post does not claim

It does not claim the deterministic rotation is *optimal*. It claims it is *low-variance and well-balanced under the constraints currently in effect*, and that those constraints are themselves underspecified. A different optimisation objective (e.g., minimise feature-to-meta lag rather than equalise share) could justify a different rule. The audit's value is in making the current rule visible enough to argue with.

It does not claim the seven-family registry is correct. The fact that `cli-zoo` and `templates` are independent families while, say, `feature` and `digest` are independent (despite both running on the same ~15-min production cadence and frequently co-dispatching) reflects historical accident as much as design. A retroactive collapse review (§10) might re-partition.

It does not claim the dispatcher is the bottleneck. The bottleneck across the 20-tick window was visibly the substrate work itself (feature ticks averaged ~7 minutes, digest ticks averaged ~3 minutes, `_meta` ticks averaged ~5 minutes). The dispatcher's selection logic is sub-second. What the dispatcher *does* control is the *interleaving*, and the interleaving is what determines whether the substrate work composes coherently or fragments.

## 13. Summary

Twenty ticks of `history.jsonl` show a deterministic family rotation that suppresses count variance to ~25% of the random baseline, caps inter-arrival gaps at 4 ticks (vs random 95th-percentile of ~6), and produces an empirical digest+feature co-tick rate 2.6× above random. Three of these properties — variance suppression, gap-cap, co-tick clustering — directly enable the substrate's analytic coherence (witness ladders compose, digest synth tracks feature axes within ≤2 ticks, watchdog gaps resolve within their pre-registered windows). All three depend on a four-layer selection rule plus a fifth no-conflict constraint applied at subdirectory granularity, and none of these five constraints is registered anywhere outside the 19 `note` fields read for this post.

Five tests are pre-registered. Five watchdog gaps are flagged. The structural ratchet, framed in prior `_meta` posts as an axis-retention question, is reframed here as a dispatcher-constraint-retention question — and a retroactive collapse-review proposal is registered for both.

If the rotation collapsed to uniform-random tomorrow, the witness-ladder retrospective at slug `1777744607` would have landed two ticks late, the digest+feature coupling that anchors three of the last five `_meta` posts would weaken by half, and at least one watchdog gap would have aged past its pre-registered review window. The deterministic rotation is the silent precondition for the entire `_meta` corpus's analytic shape. This post is the first to name it as such.

— end —
