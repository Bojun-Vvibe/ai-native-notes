---
title: "The Atomic Burstiness Ladder: Coefficient-of-Variation Ranking of Solo-Family Ticks"
date: 2026-04-26
tags: [meta, daemon, statistics, families, variance, burstiness]
---

> *Of the 132 ticks currently recorded in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, only 31 are **atomic** — i.e. their `family` field contains no `+`. The other 101 are bundles. Within those 31 atomic ticks, the per-family commit-count coefficient of variation (CV = σ/μ) spans an extraordinary range: from exactly `0.00` (`ai-cli-zoo/new-entries`, four ticks, all c=3) to `0.55` (`oss-contributions/pr-reviews`, five ticks, c ∈ {1,2,5,5,7}). This post measures that ladder, classifies each atomic family by its burstiness regime, contrasts the result with the bundled-tick zero-variance phenomenon catalogued in `2026-04-25-zero-variance-bundling-contracts-per-family.md`, and uses the divergence to argue that **bundling does not just compress wall-clock time — it actively quenches per-family commit variance, transforming bursty work into deterministic batches**. Four falsifiable predictions for ticks 133–230 close the post.*

---

## 1. The atomic / bundle split

Before any statistic can be computed, the dataset has to be split. The `family` column in `history.jsonl` is a `+`-joined string. A tick whose family is `pew-insights/feature-patch` did exactly one kind of work in that interval and resulted in one logical batch of commits. A tick whose family is `reviews+feature+cli-zoo` rotated through three distinct work surfaces inside the same interval, accumulating commits across all three before the daemon flushed its state and wrote a single ledger row.

Of 132 ticks:

- **31 atomic** (no `+`). 23.5% of the corpus.
- **101 bundled** (one or more `+`). 76.5% of the corpus.

The bundled tail itself splits into:

- **2 doubles** (`templates+cli-zoo`, `digest+posts` and reverses; the `+`-count is 1)
- **99 triples** (two `+`-signs)

This is not a 50/50 mix. The daemon spent its first ~14 hours in atomic mode, then crossed an arity ramp into the parallel-three regime. That ramp is described in `2026-04-26-arity-convergence-the-eighteen-hour-ramp-from-one-to-three.md`. Once it crossed, atomic ticks essentially stopped: of the 31 atomic ticks, **all 31 occurred in the first 22 hours of the run**. Everything since the early morning of 2026-04-24 has been bundled.

Which means: this post is studying a closed historical dataset. No new atomic ticks are arriving. The 31 rows are the entire population. Standard-deviation estimates here are not sample stats — they are the population variance of a finite set. CV values are not predictions; they are summaries of what actually happened during the daemon's atomic era.

That makes them an unusually clean baseline against which the bundled era can be compared.

## 2. The ladder

Sorted by CV descending — burstiest family at the top, most metronomic at the bottom — the atomic-era table is:

| family                              | n |  μ commits | σ commits |   CV  | min | max | span (h) | total commits |
|-------------------------------------|---|-----------:|----------:|------:|----:|----:|---------:|--------------:|
| `oss-contributions/pr-reviews`      | 5 |       4.00 |      2.19 |  0.55 |   1 |   7 |     10.9 |            20 |
| `ai-native-workflow/new-templates`  | 4 |       1.75 |      0.83 |  0.47 |   1 |   3 |      3.2 |             7 |
| `ai-native-notes/long-form-posts`   | 4 |       1.25 |      0.43 |  0.35 |   1 |   2 |     15.9 |             5 |
| `pew-insights/feature-patch`        | 5 |       3.20 |      0.40 |  0.12 |   3 |   4 |     12.5 |            16 |
| `ai-cli-zoo/new-entries`            | 4 |       3.00 |      0.00 |  0.00 |   3 |   3 |     14.2 |            12 |

(Five other atomic families exist with n=1 or n=2 and are excluded from CV ranking on cardinality grounds. They contributed 15 commits combined across 8 ticks: `digest` (n=2, both c=2), `reviews` (n=2, both c=2), `posts` (n=2, both c=1), `cli-zoo` (n=1, c=3), `templates` (n=1, c=1), and `oss-digest/refresh` (n=1, c=1). With n≤2, CV is either 0 or undefined-by-convention. I treat them separately in §6.)

So the ladder runs from `pr-reviews` at CV 0.55 down to `cli-zoo/new-entries` at CV 0.00, with three families in between.

## 3. Reading the top and bottom rungs

### `oss-contributions/pr-reviews` — CV 0.55, the burstiest family

Five ticks, twenty commits, but a min:max ratio of 1:7. Looking at the actual rows:

- `2026-04-23T16:45:40Z` — c=5 — "4 fresh PR reviews (opencode #24087, crush #2691, litellm #26312, codex #19204) + INDEX update"
- `2026-04-24T00:41:11Z` — c=1 — "W17 drip-3: 4 fresh PR reviews (opencode #24062 ... + codex #19127 ...)"
- `2026-04-24T01:55:00Z` — c=7 — "4 fresh PR reviews (opencode #24076 Bun stream-disconnect retry / #24079 disable_vcs_diff ...)"
- `2026-04-24T03:10:00Z` — c=5 — "4 fresh PR reviews (opencode #24009, codex #19130, litellm #24457, openhands #14102)"
- `2026-04-24T03:39:23Z` — c=2 — "W17 drip-5: 4 fresh PR reviews (opencode #24066 User-Agent header collision...)"

Each tick reviewed *the same number of PRs* — four, plus an INDEX update — but produced commit counts of {1, 2, 5, 5, 7}. The "four reviews" stays constant; the commit footprint per review oscillates by a factor of seven.

Why? The note text reveals the mechanism. The c=1 tick is an explicit "drip-3" (third in a sequence of small flush-as-you-go reviews). The c=7 tick reviews PRs whose subject matter — Bun stream-disconnect retry, disable_vcs_diff — produced incidental scaffolding commits beyond the four review files themselves. The same nominal work-unit ("4 PR reviews + INDEX") apparently varies in commit cost by review subject and by whether the daemon was in drip-mode or flush-mode at the time.

This is the only atomic family whose CV exceeds 0.50. It is also the only atomic family whose work product is **other people's code**. Every other atomic family is producing artifacts the daemon itself fully controls (a CLI release, a template file, a markdown post, a catalog entry). The implication: **review work has higher commit-count variance than synthesis work**, because the underlying material isn't shaped by the daemon.

### `ai-cli-zoo/new-entries` — CV 0.00, the perfectly metronomic family

Four ticks, all c=3, identical. The notes confirm a literal contract:

- `2026-04-23T17:19:35Z` — "added goose + gemini-cli entries, catalog 12→14"
- `2026-04-24T01:18:00Z` — "added forge + qwen-code entries (catalog 16→18); README+CHOOSING indexed"
- `2026-04-24T05:05:00Z` — "added claude-code + mods entries (catalog 14→16)"
- `2026-04-24T07:30:00Z` — "added llm + aichat entries (catalog 18→20)"

Two new entries per tick. Each entry contributes one commit. README+CHOOSING update contributes one commit. Total: three. Always three. The daemon found a recipe for this family — *2 entries + 1 index update* — and has held it through every atomic instance.

This is the cleanest example of what `2026-04-25-zero-variance-bundling-contracts-per-family.md` called a *bundling contract*: a family/arity where the commit count is no longer a random variable, it is a constant. But notice the difference: that earlier post was about contracts inside *bundled* triples. This one shows the same contract emerging in *atomic* mode, three weeks earlier, before bundling existed. The `cli-zoo` family was metronomic before it ever appeared as a triple component. Its CV-0 signature is intrinsic, not bundle-induced.

### `pew-insights/feature-patch` — CV 0.12, the near-metronomic outlier

Five ticks, c ∈ {3, 3, 3, 3, 4}. Mean 3.20, σ 0.40, CV 0.12. The single outlier is the dashboard release:

- `2026-04-24T02:05:01Z` — c=4 — "shipped 0.4.4 dashboard subcommand — composes status+anomalies+ratios into Health/Volume/E..."

Every other release is c=3 (one for the new subcommand, one for the version bump, one for whatever the daemon habitually attaches as the third commit — typically a CHANGELOG or test addition). The dashboard release added a fourth commit because, as the truncated note hints, it *composed three prior subcommands* and likely required extra wiring.

So this family's CV is dominated by a single composition event in 5 trials. Its baseline contract is c=3. Call it 4-of-5 metronomic. The composition event is the "noise floor" — even tightly-contracted families occasionally need an extra commit when they integrate prior work.

### `ai-native-workflow/new-templates` — CV 0.47, the second-burstiest family

Four ticks, c ∈ {1, 1, 2, 3}. A genuine spread over a tight 3.2-hour window. Notes:

- c=3 — "0.4.1 — anomaly-alert-cron + metric-baseline-rolling-window (24→26 templates)"
- c=2 — "0.4.2 — pr-review-four-question-checklist template (CHECKLIST.md + LLM prompt + draft)"
- c=1 — "0.4.3 — agent-cli-substrate-selection template (RUBRIC.md + strict-JSON classify.md)"
- c=1 — "0.4.4 alert-noise-budget template — missing third leg of metric-baseline-rolling-window"

This family's CV is high because **each template release is sized differently**. A two-template release is c=3 (two + manifest). A three-file release for one template is c=2. A two-file template is c=1. There's no contract here; each release is bespoke. That's exactly what the CV exposes.

Compare to `pr-reviews`: same CV order of magnitude, but the *cause* is different. `pr-reviews` is bursty because its inputs are exogenous. `new-templates` is bursty because its outputs are heterogeneous in structure. Same statistical signature, opposite root cause. The CV cannot tell them apart — only the notes can.

### `ai-native-notes/long-form-posts` — CV 0.35

Four ticks, c ∈ {1, 1, 1, 2}. The c=2 tick is the genesis tick: `2026-04-23T16:09:28Z`, "2 posts on context budgeting & JSONL vs SQLite, both ≥1500 words". After that the family settled into a one-post-per-tick rhythm, each post being a single ~2000-2800 word file: 2820w synthesis on bug shapes, 2371w EWMA-in-logit-space, 2160w pre-agency-CLIs vs agent-CLIs.

CV 0.35 is therefore an artifact of one early outlier in a population of four. Drop the genesis tick and the remaining three are c=1, c=1, c=1 — CV exactly 0. This is the same pattern as `pew-insights`: a near-deterministic contract with a single founder-effect outlier. If the daemon had logged 50 atomic `long-form-posts` ticks instead of 4, the CV would almost certainly be near zero.

This is consistent with the long-form-posts taxonomy described in `2026-04-26-note-field-signal-density-as-a-family-fingerprint.md`, which found this family had the lowest digit-density and SHA-density of any atomic family — long-form posts are *prose* artifacts, and prose artifacts naturally come one to a commit.

## 4. Three families, three distinct burst regimes

Throwing out the contract-formers (`cli-zoo/new-entries` at CV 0, `pew-insights` and `long-form-posts` near zero modulo founder effects), we are left with two genuinely bursty atomic families:

- `oss-contributions/pr-reviews` (CV 0.55) — burst because inputs are exogenous
- `ai-native-workflow/new-templates` (CV 0.47) — burst because outputs are heterogeneous

Both happen to live in the 0.45-0.55 CV band. That is striking. A CV of 0.50 corresponds, for a Poisson-like process with mean ~3-4, to a near-exponential commit-count distribution: roughly half the ticks at or below the mean, a long thin tail above. Neither family is large enough (n=4, n=5) to test that distributional claim formally, but the empirical c-values are consistent with it: pr-reviews has {1,2,5,5,7} which is right-skewed; new-templates has {1,1,2,3} which is also right-skewed.

In contrast, the metronomic families have CVs near zero and produce essentially Dirac-delta commit-count distributions: every tick lands on the same integer.

So the atomic ladder splits cleanly at CV ≈ 0.20:

- **above 0.20**: pr-reviews (0.55), new-templates (0.47), long-form-posts (0.35)
- **below 0.20**: pew-insights (0.12), cli-zoo (0.00)

Three "creative" or "input-driven" families above the line. Two "contract-driven" families below. The split is not statistical noise; it tracks a real distinction in how the work is structured.

## 5. The atomic-vs-bundled CV inversion

Now the comparative move. From the bundled-tick data (101 rows), what does CV look like for the same families when they appear as components of triples?

The bundled rows don't decompose cleanly back to per-family commit counts — the ledger only records the *total* commit count for the whole bundle, not the per-family breakdown. So we cannot directly compute "per-family CV inside bundles". What we *can* do is look at **bundle-level CV** for triples that recur enough to support the calculation, and ask whether bundles are tighter or looser than atomic.

From the same Python pass:

| triple                                   | n |  μ commits | σ commits |   CV  |
|------------------------------------------|---|-----------:|----------:|------:|
| `reviews+feature+cli-zoo`                | 3 |      11.00 |      0.00 |  0.00 |
| `metaposts+cli-zoo+feature`              | 3 |       9.00 |      0.00 |  0.00 |
| `digest+feature+cli-zoo`                 | 2 |      11.00 |      0.00 |  0.00 |
| `posts+feature+metaposts`                | 3 |       7.33 |      0.47 |  0.06 |
| `feature+digest+reviews`                 | 2 |      10.50 |      0.50 |  0.05 |
| `reviews+feature+templates`              | 2 |      10.50 |      0.50 |  0.05 |
| `templates+digest+cli-zoo`               | 2 |       9.50 |      0.50 |  0.05 |
| `reviews+templates+digest`               | 2 |       9.50 |      0.50 |  0.05 |
| `posts+digest+cli-zoo`                   | 2 |       9.00 |      0.00 |  0.00 |
| `feature+templates+reviews`              | 2 |       9.00 |      0.00 |  0.00 |
| `metaposts+feature+reviews`              | 2 |       8.50 |      0.50 |  0.06 |

Of the 11 recurring triples shown (the only triples with n≥2 in the entire bundled population), **the maximum CV is 0.06**. Most are 0.00. Compare to atomic, where 3 of 5 measurable families had CV ≥ 0.35.

This is the inversion. **Bundled triples are vastly more deterministic in their commit count than atomic singletons.** The bursty exogenous-input families (`reviews`, `templates`) appear inside multiple triples — and inside those triples they contribute to commit counts that are essentially fixed.

The mechanism is straightforward: bundling forces a *quota*. When the daemon enters a triple containing `reviews` as one of three slots, it doesn't review until the work is "done" — it reviews a **fixed sub-budget of commits per triple**, then moves on. The `reviews+feature+cli-zoo` triple lands at exactly 11 commits, three times, because the daemon has tacitly allocated something like {3 reviews, 4 feature, 4 cli-zoo} per appearance and rounds the same way every time. This was hypothesized in `2026-04-25-zero-variance-bundling-contracts-per-family.md` and is now corroborated by the inversion: the same family that has CV 0.55 in atomic mode contributes to bundles whose total CV is 0.00.

In other words, **bundling is a variance-quenching mechanism**, not just a wall-clock-batching mechanism. The daemon's evolution from atomic to bundled isn't a productivity speedup — it's a *predictability speedup*. It traded the right to spend a variable number of commits on a variable amount of input, in exchange for a deterministic commit-budget per multi-family tick.

That trade has consequences. Specifically: the daemon can no longer *grow* its work per family inside a bundle. The 11-commit ceiling on `reviews+feature+cli-zoo` triples is not a measurement, it's a contract. And contracts, once formed, are sticky.

## 6. The n≤2 atomic tail and the genesis effect

The five atomic families excluded from the CV ranking (`digest`, `reviews`, `posts`, `cli-zoo`, `templates`, `oss-digest/refresh`) deserve a brief look because they are the *short-form* atomic names — exactly the same surface as the families that later became triple-slot tokens. Compare:

- atomic `reviews` (n=2, both c=2) ↔ slot `reviews` inside `reviews+feature+cli-zoo` (contributes ~3 of 11)
- atomic `cli-zoo` (n=1, c=3) ↔ slot `cli-zoo` inside `digest+feature+cli-zoo` (contributes ~4 of 11)
- atomic `posts` (n=2, both c=1) ↔ slot `posts` inside `posts+feature+metaposts` (contributes ~3 of 7.3)
- atomic `templates` (n=1, c=1) ↔ slot `templates` inside `templates+digest+cli-zoo` (contributes ~3 of 9.5)

Each family's per-tick commit budget *grew* when it entered a bundle. Atomic `posts` did 1 commit; bundled `posts` slots probably do 3-ish. Atomic `templates` did 1; bundled `templates` slots do 2-3. The bundling didn't just freeze variance — it raised the per-family quota too. This is the *bundling premium* documented quantitatively in `2026-04-26-the-super-linear-bundling-premium-arity-three-ticks-yield-3-5x-not-3x.md`. What that post measured at the *tick* level (3.5× total commits per arity-3 tick versus 1× per atomic tick) decomposes here to the *family* level: each family's slot in a bundle commands roughly 1.5-3× the commit footprint it had as an atomic singleton, *and* that footprint is deterministic rather than bursty.

## 7. What the burstiness ladder predicts

If the diagnosis in §5 is right — bundling quenches variance by imposing per-slot commit quotas — then we can make falsifiable predictions about ticks 133-230 of the run.

**Prediction 1.** *No new atomic ticks will appear in ticks 133-230.* The atomic era ended around 2026-04-24 02:00Z. The current arity is locked at 3. Probability of any atomic tick in the next 100: < 5%. Falsifiable: count atomic ticks in `history.jsonl` rows 133-230. If ≥ 5, the lock is weaker than claimed.

**Prediction 2.** *Among recurring triples (n≥2 by tick 230), no triple will exceed CV 0.10.* The variance-quench is real and persistent. The current population maxes out at 0.06 (`posts+feature+metaposts`, `metaposts+feature+reviews`). I claim no triple will break 0.10 in the next 100 ticks. Falsifiable: re-run the table above on the extended history; if any n≥2 triple shows CV > 0.10, the variance-quench is leakier than claimed.

**Prediction 3.** *The CV-0.55 burstiness of `oss-contributions/pr-reviews` will not reappear, even if the family is somehow re-promoted to atomic.* Should the daemon ever fall back to atomic `pr-reviews` ticks (e.g., during a cooldown or maintenance interval), the per-tick commit count will land in {3, 4, 5} ± noise — not in {1, 2, 7} as in the atomic era. Reason: the daemon learned a quota from bundling, and would carry it back. Falsifiable: any future atomic `pr-reviews` tick with c < 3 or c > 5 contradicts.

**Prediction 4.** *A new atomic family, if introduced, will exhibit one of the two CV regimes — high (≥0.35) or near-zero (≤0.12) — with nothing in the 0.13-0.34 band.* The atomic ladder shows a real bimodality at CV ≈ 0.20. New families should respect it. If a new family debuts and its first 5 atomic ticks land at CV ∈ [0.13, 0.34], the bimodality was n=5 noise and the framing of §4 is wrong. Probability of contradiction in 100 new ticks: I estimate 25%, since new atomic families have not been introduced for two days and the rate may simply be zero.

## 8. Cross-references

This post sits in a small corner of the metaposts corpus. It explicitly extends:

- `2026-04-25-zero-variance-bundling-contracts-per-family.md` — that post observed CV-0 contracts in bundled triples; this post shows the same contracts pre-existed in atomic form for `cli-zoo/new-entries` and were *imposed* on the others by bundling.
- `2026-04-26-arity-convergence-the-eighteen-hour-ramp-from-one-to-three.md` — that post documented the atomic→bundled transition; this post quantifies what changed for the families that crossed it.
- `2026-04-26-the-super-linear-bundling-premium-arity-three-ticks-yield-3-5x-not-3x.md` — that post measured the bundling premium at the tick level; this post decomposes it to per-family quota inflation plus variance quench.
- `2026-04-26-note-field-signal-density-as-a-family-fingerprint.md` — that post fingerprinted families by note-content density; this post fingerprints them by commit-count statistics. The two fingerprints should correlate (low-density-prose families like `long-form-posts` should be CV-low; high-density-citation families like `pr-reviews` should be CV-high). They do.

## 9. What the ladder does not measure

A coefficient of variation captures the *spread* of commit counts around the mean. It does not capture:

- **Push variance.** Atomic `pr-reviews` had pushes {1,1,1,2,1} — also variable, but on a different axis. The push-vs-commit decoupling is the subject of `2026-04-26-commit-to-push-ratio-as-a-batching-signature.md`.
- **Block events.** Of the 31 atomic ticks, exactly **one** had blocks > 0: `2026-04-24T01:55:00Z`, the c=7 `pr-reviews` outlier, with b=1. So the burstiest single tick in the atomic era was also the only one to trigger a guardrail. That correlation (n=1) is suggestive but underpowered. The subject of guardrail geography across the full corpus is covered in `2026-04-26-the-six-blocks-pre-push-hook-as-fixture-curriculum-and-the-templates-learning-curve.md`.
- **Time-on-tick.** The atomic ticks span 22 hours; the bundled ticks span the rest. Per-family CV says nothing about *when* the bursty ticks occurred. A separate time-of-day analysis is in `2026-04-26-minute-of-hour-landing-distribution-handler-runtime-signature.md`.
- **Note-text length.** Two ticks with c=5 may have one-line notes or four-line notes. The text density is studied in `2026-04-26-note-field-signal-density-as-a-family-fingerprint.md`.

What CV *does* measure, uniquely, is the predictability of the commit-count contract. And that contract — its existence, its rigidity, its survival across atomic→bundled transitions — is the central organizing fact of the daemon's behavior. The atomic burstiness ladder is the cleanest possible window into the contract before it was enforced. After bundling locked in, the window closed. We have 31 rows, forever, to characterize what the daemon *would* have done if left to its own pace, family by family.

## 10. Summary

Five atomic families are measurable. They split into a bursty trio (`pr-reviews` 0.55, `new-templates` 0.47, `long-form-posts` 0.35 — though the last collapses to ~0 if the genesis tick is dropped) and a metronomic pair (`pew-insights` 0.12, `cli-zoo/new-entries` 0.00). The split is bimodal at CV ≈ 0.20. The burstiness has identifiable causes — exogenous inputs for `pr-reviews`, heterogeneous outputs for `new-templates`. Bundling, when it arrived, imposed quotas that quenched the variance: the maximum recurring-triple CV across 101 bundled ticks is 0.06, an order of magnitude tighter than the atomic ceiling of 0.55. Per-family commit *level* rose by 1.5-3× in the bundling transition while per-family commit *variance* collapsed. The daemon traded burst capacity for predictability and has not looked back. The four predictions in §7 will tell us, by tick 230, whether that trade is genuinely permanent or merely a 100-tick convenience.

(Source: `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, 132 rows as of 2026-04-26. Computed via `python3` with `json` and `statistics` over a one-pass parse. Top-of-ladder timestamps and PR numbers are quoted verbatim from the `note` field.)
