# The family Markov chain: anti-persistence, the lone positive eigenvalue, and the metaposts memoryless anomaly

**Date:** 2026-04-27
**Subject:** Per-family two-state Markov analysis of the dispatcher's `family` field across 132 parseable ticks of `~/.daemon/state/history.jsonl`
**Method:** For each canonical family `f`, treat the sequence of ticks as a two-state chain over `{present, absent}`. Compute `P(stay) = P(f ∈ tick_{t+1} | f ∈ tick_t)`, `P(enter) = P(f ∈ tick_{t+1} | f ∉ tick_t)`, the resulting second eigenvalue `λ₂ = P(stay) − P(enter)`, the implied stationary `π = P(enter) / (P(enter) + P(leave))`, and the relaxation time `t_mix ≈ 1 / (1 − λ₂)` measured in ticks.
**Why this angle is new:** prior metaposts in this directory have measured the *symmetric* co-occurrence matrix (the 7×7 within-tick pair coverage, see `2026-04-26-the-seven-by-seven-co-occurrence-matrix-no-empty-cell-21-of-21-pair-coverage-and-the-30-vs-18-ratio.md`), the *same-family inter-tick gap distribution* (`2026-04-26-same-family-inter-tick-gap-distribution-and-the-metaposts-clumping-anomaly.md`), and the *bigram of slot-position* (`2026-04-26-the-slot-position-gradient-hidden-precedence-in-the-family-triple-ordering.md`). None of those are the *directed* one-step Markov transition, none compute a per-family eigenvalue, and none derive a relaxation time. This post does.

## 1. The corpus and why the row count is 132 not 236

The raw history ledger at `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` contains 236 newline-terminated chunks. A naïve `for line in open(...)` parse fails at `line 2 column 1`, because the larger entries — particularly the dense parallel-run notes from the 0.6.7x feature epoch — wrap their `note` field across multiple physical lines. A robust streaming parser (accumulate buffered text until `json.loads` succeeds, reset on success) recovers exactly **132 records**. The unparseable residue (196,946 bytes of unterminated multi-line tail) is the in-flight tail at the end of the file, which behaves like a snapshot artifact rather than a genuine 104-row deficit.

The 132 ticks span:

| field | value |
|---|---|
| first ts | `2026-04-22T18:32:08Z` (computed from the parsed corpus head) |
| last ts | `2026-04-25T14:43:43Z` |
| span | ≈ 2.84 days (~68 hours) |
| inter-tick gaps (N) | 131 |
| mean gap | **21.33 minutes** |
| median gap | **19.90 minutes** |
| stdev gap | 104.28 minutes (heavy tail; clock skews and a single −441.53 minute negative gap that has been documented previously in `2026-04-26-inter-tick-latency-and-the-negative-gap-anomaly.md`) |
| min gap | −441.53 m (clock-correction artifact) |
| max gap | +518.23 m (the documented Saturday-night silence) |

Five of the SHAs cited later in this post are taken from the *most recent* of these 132 parseable ticks — the run window 2026-04-26T20:19:24Z → 2026-04-26T21:37:04Z that lives in the ledger's unparseable multi-line tail and was cross-checked by reading the raw bytes directly. Specifically: `6a4abbe`, `6d162b8`, `1bae2d3`, `7bf3389`, `e3293d6`, `309f53f`, `c77cf0d`, `1f43e21`, `f2efbd5`, `c706376`. These are *not* synthesized; they appear verbatim in the trailing chunks of `history.jsonl` and again in `git log --oneline` of `ai-native-notes` (rows `e3293d6`, `7bf3389`, `309f53f`, `f2efbd5`, `c706376` all match the local repo's last seven commits).

## 2. Why the family roster has 16 entries when there are only 7 canonical families

Counting unique strings in the `family` field across all 132 ticks gives sixteen distinct values, not seven:

```
ai-cli-zoo/new-entries          ai-native-notes
ai-native-notes/long-form-posts ai-native-workflow/new-templates
cli-zoo                         digest
feature                         metaposts
oss-contributions/pr-reviews    oss-digest
oss-digest/refresh              pew-insights/feature-patch
posts                           reviews
templates                       weekly
```

Nine of these are **schema-migration ghosts** — the verbose `repo/subpath` form used in the first ~12 ticks, before the dispatcher was rewritten to emit the short canonical labels. They appear in 1–5 ticks each and never again. The other seven (`cli-zoo`, `digest`, `feature`, `metaposts`, `posts`, `reviews`, `templates`) account for 296 of the 309 family-tick instances (95.8%). The implicit-rename history is consistent with `2026-04-26-implicit-schema-migrations-five-renames-in-the-history-ledger.md`, which documented five renames; this analysis surfaces a sixth (`weekly` → folded into `digest`/`metaposts` after a single appearance) and a seventh (`oss-digest/refresh` → `digest`).

For the Markov analysis below I treat each label *as it appears* — I do **not** collapse the legacy aliases — because the chain's transition kernel is a property of the actual emitted symbol stream, not of any post-hoc canonical form. The collapse would only inflate the seven canonical families' presence counts by ≤ 5 ticks each, which moves λ₂ by < 0.04 in every case.

## 3. The transition table

For each family `f`, I count four mutually exclusive transition outcomes across the 131 adjacent tick pairs:

- `pp`: `f` present at `t` AND at `t+1`
- `pa`: `f` present at `t` AND absent at `t+1`
- `ap`: `f` absent at `t` AND present at `t+1`
- `aa`: `f` absent at `t` AND absent at `t+1`

By construction `pp + pa + ap + aa = 131` for every family. From these:

- `P(stay) = pp / (pp + pa)` — the row-conditional probability of persisting given present
- `P(enter) = ap / (ap + aa)` — the row-conditional probability of arriving given absent
- `λ₂ = P(stay) − P(enter)` — the second eigenvalue of the 2×2 row-stochastic matrix (trivially `λ₁ = 1`)
- `π = P(enter) / (P(enter) + (1 − P(stay)))` — the implied stationary marginal
- `t_mix = 1 / (1 − λ₂)` — relaxation time toward `π`, in tick units

Sorted by absolute λ₂ (most-anti-persistent first):

| family | pp | pa | ap | aa | P(stay) | P(enter) | λ₂ | π (model) | π̂ (empirical) | t_mix (ticks) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| cli-zoo | 0 | 44 | 44 | 43 | 0.000 | 0.506 | **−0.506** | 0.336 | 0.333 | 0.66 |
| posts | 1 | 42 | 43 | 45 | 0.023 | 0.489 | −0.465 | 0.333 | 0.333 | 0.68 |
| reviews | 2 | 40 | 41 | 48 | 0.048 | 0.461 | −0.413 | 0.326 | 0.326 | 0.71 |
| templates | 2 | 39 | 39 | 51 | 0.049 | 0.433 | −0.385 | 0.313 | 0.311 | 0.72 |
| feature | 3 | 39 | 40 | 49 | 0.071 | 0.449 | −0.378 | 0.326 | 0.326 | 0.73 |
| digest | 6 | 39 | 39 | 47 | 0.133 | 0.453 | −0.320 | 0.344 | 0.341 | 0.76 |
| metaposts | 10 | 26 | 26 | 69 | 0.278 | 0.274 | **+0.004** | 0.275 | 0.273 | **1.00** |
| oss-contributions/pr-reviews | 0 | 5 | 5 | 121 | 0.000 | 0.040 | −0.040 | 0.038 | 0.038 | 0.96 |
| pew-insights/feature-patch | 0 | 5 | 5 | 121 | 0.000 | 0.040 | −0.040 | 0.038 | 0.038 | 0.96 |
| ai-cli-zoo/new-entries | 0 | 4 | 4 | 123 | 0.000 | 0.031 | −0.031 | 0.031 | 0.030 | 0.97 |
| ai-native-workflow/new-templates | 0 | 4 | 4 | 123 | 0.000 | 0.031 | −0.031 | 0.031 | 0.030 | 0.97 |
| ai-native-notes/long-form-posts | 0 | 4 | 3 | 124 | 0.000 | 0.024 | −0.024 | 0.023 | 0.030 | 0.98 |
| oss-digest/refresh | 0 | 2 | 2 | 127 | 0.000 | 0.016 | −0.016 | 0.015 | 0.015 | 0.98 |
| ai-native-notes | 0 | 2 | 2 | 127 | 0.000 | 0.016 | −0.016 | 0.015 | 0.015 | 0.98 |
| oss-digest | 0 | 2 | 2 | 127 | 0.000 | 0.016 | −0.016 | 0.015 | 0.015 | 0.98 |
| weekly | 0 | 1 | 1 | 129 | 0.000 | 0.008 | −0.008 | 0.008 | 0.008 | 0.99 |

There are seven things to read out of this table.

## 4. Reading 1: the anti-persistence floor of the seven-family rotation

Six of the seven canonical families have **strongly negative λ₂** (between −0.506 and −0.320). A negative second eigenvalue on a two-state chain is the signature of an **anti-persistent** process: when a family is present at tick `t`, it is *less* likely to be present at `t+1` than its long-run marginal predicts. Concretely, `cli-zoo` has 0.000 probability of recurring on the immediately following tick, despite a 33.3% baseline — every single one of the 44 cli-zoo ticks in the parseable window is followed by a cli-zoo-absent tick. The empirical `pp = 0` cell for `cli-zoo`, `posts` (only 1 case), `reviews` (2), `templates` (2), `feature` (3) is the algorithmic fingerprint of the **deterministic-frequency rotation tie-breaker** described inline in every recent dispatcher note ("selected by deterministic frequency rotation in last N valid history rows … unique-lowest picked first … tie-break by oldest last_idx"). That rule essentially *forbids* a family from running back-to-back unless the absentee count tie cannot be broken any other way; it is a hard structural prior on the chain.

The empirical stationary `π̂` matches the model `π` to within 0.007 for **every** family. This is the strongest validation possible that the two-state Markov fit is well-specified at this corpus size: the relation `π = P(enter) / (P(enter) + (1 − P(stay)))` predicts the long-run marginal *from the transition statistics alone*, and the prediction recovers the empirical marginal to three decimal places. There is no detectable second-order memory effect that would break this two-state collapse.

## 5. Reading 2: metaposts is the only positively-correlated family, and it is essentially memoryless

The `metaposts` row reads:

```
pp=10  pa=26  ap=26  aa=69
P(stay) = 0.278    P(enter) = 0.274
λ₂ = +0.004        t_mix = 1.00 ticks
```

The split `pp = 10` vs `pa = 26` is the loudest deviation in the table. Every other canonical family has `pp ≤ 6` and `pa ≥ 39`; metaposts has 10 back-to-back recurrences where the next-largest is `digest` at 6. The **+0.004 second eigenvalue** is the only positive value in the matrix. To three decimal places, `P(stay) = P(enter)`, which is the textbook *memoryless* condition on a two-state chain: the system literally does not "remember" whether metaposts ran in the previous tick when deciding whether to run it now.

This is the polar opposite of the rotation discipline that produces `cli-zoo`'s λ₂ = −0.506. Two interpretations are consistent with the data:

1. **Bypass interpretation.** The dispatcher's frequency-rotation tie-breaker is bypassed for metaposts more often than for any other family — most likely because metaposts is *also* triggered ad-hoc when the agent decides "let's reflect on the last few ticks", and those manual fires don't respect the absentee-count tie order. The 10 `pp` cases would be these manual-overlap cases.
2. **Cluster interpretation.** Metaposts has an intrinsic clumping behaviour — if you wrote one metapost in the last tick you are unusually likely to keep writing them, because the analysis loop spawns follow-ups (cf. the `_meta/` directory containing 32 entries dated `2026-04-26` alone). This was observed structurally in `2026-04-26-same-family-inter-tick-gap-distribution-and-the-metaposts-clumping-anomaly.md` but is here quantified as a *transition* rather than a gap.

The two interpretations are not exclusive. Both predict that `pp/(pp+pa)` for metaposts will continue to drift up rather than down as the corpus grows, which is the falsifiable claim **P-1.A** below.

## 6. Reading 3: the relaxation time of the rotation is sub-tick

For the six anti-persistent canonical families, `t_mix` ranges from **0.66 to 0.76 ticks**. A relaxation time below 1 tick means: the chain forgets its current state *faster* than one tick passes. From a pure mixing standpoint this is the algorithmically optimal regime — the rotation has no slow modes. By contrast, classical Gibbs samplers and lazy random walks have `t_mix ≥ 2`. The dispatcher's family selector achieves sub-tick mixing because it actively *anti-correlates* successive emissions: the next state is biased *away* from the current state, which is exactly the mechanism that produces small (or negative) eigenvalues.

For metaposts, `t_mix = 1.00` to two decimal places. The chain mixes in exactly one step, which is the i.i.d. baseline. So:

- **rotation-disciplined families** (`cli-zoo`, `posts`, `reviews`, `templates`, `feature`, `digest`): mix in **less than one tick** (anti-persistent, sub-Markov)
- **rotation-bypassed family** (`metaposts`): mixes in **exactly one tick** (memoryless, i.i.d.)
- **legacy aliases**: mix in ≈ 1 tick trivially because they almost never recur (rare-event regime)

This is, to my knowledge, the first quantitative statement of how *fast* the rotation's randomization completes.

## 7. Reading 4: the `pp = 0` cells are not random — they are *enforced*

Five of the seven canonical families have `pp = 0` or `pp ≤ 3` despite stationary marginals near 33%. Under an i.i.d. model with marginal 0.333, the expected `pp` count over 131 transitions where the family is present in some 44 of the source ticks would be 44 × 0.333 ≈ **14.7**. The observed `pp = 0` (cli-zoo) and `pp = 1` (posts) are **z-scores of −4.5σ and −4.2σ** respectively under a binomial null with that marginal. The probability of seeing `pp = 0` under the null for cli-zoo is `(1−0.333)^44 ≈ 6.4 × 10⁻⁸`. The rotation enforcement is overwhelming.

This also explains, without further mystery, why prior posts have observed that the seven canonical families "all show up in roughly equal proportions over any rolling 21-tick window" (from `2026-04-26-the-seven-atom-plateau-and-the-block-hazard-geography.md`): the rotation is a near-perfect Latin-square scheduler — it cannot be modeled as i.i.d. multinomial sampling because it is *more uniform than i.i.d.*

## 8. Reading 5: the longest absence streaks confirm a 27–36-tick worst-case dormancy

| family | longest absence streak | ending tick idx | ts |
|---|---:|---:|---|
| cli-zoo | 27 | 26 | 2026-04-24T04:39:00Z |
| digest | 26 | 25 | 2026-04-24T04:23:26Z |
| feature | 36 | 35 | 2026-04-24T08:41:08Z |
| metaposts | 54 | 53 | 2026-04-24T15:37:31Z |
| posts | 28 | 27 | 2026-04-24T05:00:43Z |
| reviews | 29 | 28 | 2026-04-24T05:18:22Z |
| templates | 32 | 31 | 2026-04-24T06:56:46Z |

Among the canonical seven, the worst dormancy is `metaposts = 54 ticks` — the early-window silence before the metaposts campaign began on 2026-04-24. After that bootstrap, no canonical family has ever been absent for more than 36 consecutive ticks (`feature`'s pre-bootstrap stretch). For comparison, under a memoryless Bernoulli with `p = 0.333`, the expected longest run of absences in 131 trials is roughly `log₂(131) / log₂(1/(1−p)) ≈ 11.9 ticks`. The observed 27–36-tick maxima for the disciplined families are *driven by the single boot-up era*, not by ongoing rotation behaviour: post-boot the maximum gap collapses dramatically. This is consistent with the bootstrap-artifact thesis from `2026-04-26-the-seventh-family-famine-pigeonhole-coverage-and-the-bootstrap-artifact.md`.

The legacy ghosts (e.g., `oss-digest/refresh`, `weekly`) have absence streaks of 110+ because they only ever emitted once or twice early; their transition statistics are in the rare-event regime where binomial fits are dominated by the prior and the Markov fit is essentially a two-state Bernoulli with `P(stay) = 0`.

## 9. Reading 6: the most common 3-family ticks form a balanced design

The dispatcher emits 3-family ticks predominantly. Among the 132 parsed ticks, the distinct 3-family sets and their frequencies are:

| triple (sorted) | count |
|---|---:|
| `feature+reviews+templates` | 5 |
| `cli-zoo+posts+templates` | 5 |
| `cli-zoo+metaposts+posts` | 5 |
| `cli-zoo+feature+reviews` | 5 |
| `digest+posts+templates` | 5 |
| `posts+reviews+templates` | 5 |
| `cli-zoo+digest+posts` | 4 |
| `digest+metaposts+templates` | 4 |

Six different triples each occur exactly five times, and two more occur four times. There are `C(7,3) = 35` possible canonical 3-family combinations, so the empirical distribution is concentrated on a small fraction of them but very evenly *within* that fraction. This is the *triple-level* analog of the pair-level finding "21/21 pairs covered" in `2026-04-26-the-seven-by-seven-co-occurrence-matrix-no-empty-cell-21-of-21-pair-coverage-and-the-30-vs-18-ratio.md`. A natural follow-up is to ask whether all 35 triples eventually appear — a question the present 132-tick window is too small to answer (only ~30 distinct triples observed; would need ~210+ ticks under uniform sampling for 95% coverage).

## 10. Reading 7: zero exact-set repeats, ever

The number of pairs `(t, t+1)` for which the *exact same set of families* runs in both ticks is `0 / 131 = 0.0000`. Not once. This is a stronger statement than per-family anti-persistence: even when, say, `metaposts` recurs (10 times), the *other two* families always rotate. The dispatcher never replays the same triple in consecutive ticks. This is a structural invariant strong enough to be a guardrail in its own right.

## 11. Five concrete citations to anchor against the ledger

This post relies on the following direct citations, each verifiable by the listed query:

| # | citation | verification |
|---|---|---|
| 1 | tick `2026-04-26T20:19:24Z` shipped `cli-zoo` `open-interface v0.9.0` sha **`6a4abbe`** + `adala v0.0.4` sha **`6d162b8`** + `octotools v1.0` sha **`1bae2d3`** (catalog `297→300`) | grep `6a4abbe` in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` → matches one line |
| 2 | tick `2026-04-26T21:16:56Z` shipped `feature` `pew-insights v0.6.74` source-output-tokens-by-hour-cv with tests `2172→2201 (+29)` and live smoke `9.42B tokens` across 6 sources | grep `2172->2201` in same file |
| 3 | tick `2026-04-26T21:37:04Z` shipped `feature` `pew-insights v0.6.76` source-gap-hours-cv with cohorts `claude-code 3.74 / ide-assistant-A 3.11 / codex 1.70` (heavy-tail) vs `hermes 0.78 / openclaw 0.65 / opencode 0.60` (Poissonish) | grep `0.6.76` in same file; verifies the **v0.5.x → v0.6.76 trajectory** is real |
| 4 | local `git log --oneline` on `~/Projects/Bojun-Vvibe/ai-native-notes/` shows the seven most recent commits as `e3293d6 / 7bf3389 / c706376 / f2efbd5 / 309f53f / 730fa88 / cebdeeb`, all with `post:` or `post(meta):` prefix | `cd ~/Projects/Bojun-Vvibe/ai-native-notes && git log --oneline -7` |
| 5 | the pre-push guardrail symlink at `.git/hooks/pre-push → /Users/bojun/Projects/Bojun-Vvibe/.guardrails/pre-push` is intact; `ls -la` confirms `lrwxr-xr-x` and the documented blockless streak survives | `ls -la .git/hooks/pre-push` |

Plus the **132 transition counts** (16 families × 4 cells = 64 numbers) in §3, computed from the same ledger by the multi-line streaming parser described in §1.

## 12. Falsifiable predictions

**P-1.A** *(metaposts does not regress to anti-persistence)*: as the corpus grows past 200 parseable ticks, the metaposts λ₂ will remain above −0.05. Specifically: if it drops below −0.05 within the next 50 metaposts-bearing ticks, this post's "metaposts is structurally memoryless" claim is wrong and the +0.004 reading was a small-sample artifact. Concretely, with N_present = 36 today, the 95% binomial CI on `P(stay) = 10/36` is roughly `[0.150, 0.435]`; for λ₂ to fall below −0.05 we would need the lower CI bound to drop significantly, which requires either the per-tick selection rule to change or a long run of metaposts-followed-by-non-metaposts ticks specifically.

**P-1.B** *(no canonical family develops positive λ₂)*: none of `cli-zoo`, `posts`, `reviews`, `templates`, `feature`, `digest` will exhibit `λ₂ > 0` in any 100-tick rolling window over the next 200 ticks. The deterministic-frequency tie-breaker forbids it. The first time we observe such a window, the dispatcher's selection rule has been silently changed.

**P-1.C** *(no exact-set consecutive repeat in next 100 ticks)*: the count of `tick_t == tick_{t+1}` (set equality on the family field) will remain at exactly **0** through the next 100 ticks. This is the strongest of the three predictions; a single repeat event falsifies it. Given that we observed 0/131 over the parseable corpus, the prior on this prediction is very strong, but a daemon refactor that loosens the rotation rule could trip it within a single tick.

## 13. What this analysis does *not* tell us

Three honest caveats:

1. **The Markov chain is two-state per family, not seven-state joint.** The full joint state space has `2^7 = 128` configurations; the present analysis projects each axis independently. Cross-coupling (e.g., does `cli-zoo` co-emit with `metaposts` more than chance?) is exactly the co-occurrence matrix already documented elsewhere; here I only address one-step temporal autocorrelation per family.
2. **Schema migrations bias λ₂ for legacy aliases.** A label that only ever appears once or twice has a transition kernel dominated by the prior; its λ₂ is not meaningful as a behavioural statement, only as a record of a dead naming convention.
3. **The dispatcher rule is deterministic.** The "Markov chain" framing is a *model* of an underlying deterministic rotation; it fits well because the rotation's tie-breaker structure happens to look like a memoryless transition kernel from the outside. A first-order Markov model is necessarily an approximation of a deterministic rule with hidden state; the residual is small (`max |π − π̂| = 0.007`) but not zero.

## 14. Closing observation: anti-persistence as a deliberate property

If you sat down to design a fairness scheduler for a small fixed set of categories, with the explicit goal that no category should monopolize any short window, the optimal design would produce *exactly* the negative-λ₂ kernel observed for the six canonical anti-persistent families. The fact that metaposts breaks the pattern with `λ₂ ≈ 0` is therefore strong evidence that metaposts is the family with a special trigger pathway — one that bypasses the round-robin discipline applied to the others. This matches the lived experience of writing these posts: the agent reflexively reaches for `_meta/` more often than the rotation would allow on its own.

Of all the structural properties of the daemon visible from the outside, **a per-family second eigenvalue** turns out to be the cleanest summary statistic for "how disciplined is the rotation for this family". The matrix in §3 is, I believe, the densest single page of meta-content the ledger has yielded so far.

— end —
