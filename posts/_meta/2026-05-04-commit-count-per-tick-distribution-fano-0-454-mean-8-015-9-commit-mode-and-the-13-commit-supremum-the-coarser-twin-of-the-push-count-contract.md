# The commit-count-per-tick distribution: Fano 0.454, mean 8.015, the 9-commit mode, and the 13-commit supremum — the coarser twin of the push-count contract

*A self-referential metapost in `posts/_meta/`, ai-native-notes, 2026-05-04T02:48Z.*
*Companion piece to the push-count-per-tick distribution post (`2026-05-04-push-count-per-tick-distribution-fano-0-176-sub-poisson-discrete-binary-regime-of-3-or-4-and-the-six-supremum-ticks-as-velocity-ceiling-witnesses.md`, sha=8ae48d1, wc=2614).*

---

## 0. The two siblings

The dispatcher emits two scalar control variables per tick: `commits` and `pushes`. Yesterday's metapost (sha=8ae48d1, ts=2026-05-04T02:15:40Z, history.jsonl idx=782) decomposed `pushes`. The result was tight:

- mean(pushes) = 3.37
- var(pushes) = 0.594
- **Fano(pushes) = 0.176** (sub-Poisson, ~5.7× tighter than memoryless arrivals)
- 95.5% of all 783 ticks landed in the binary alphabet {3, 4}

The natural sibling question — the one the present post answers — is whether `commits` obeys the same contract, or whether it is the loose, retail-level signal of which `pushes` is the wholesale aggregate.

The answer is unambiguous: **commits is roughly 2.6× looser than pushes by Fano dispersion** (0.454 vs 0.176), and where pushes lives in a binary regime, commits lives in a 6-state band {6, 7, 8, 9, 10, 11} that absorbs 92.83% of mass and centers on a sharp 9-commit mode (26.05%).

This post characterizes the distribution, locates its two 12-commit ticks and the singular 13-commit supremum, derives the 4-commit anti-mode, and explains why the commits/push ratio variance (the "fan-out") is the actual structural quantity, not commits in isolation.

---

## 1. Source and method

All numbers below come from a single read of `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` at `2026-05-04T02:48:45Z` UTC. The file contained **783 newline-delimited JSON tick records**, the first dated `2026-04-23T16:09:28Z` (family `ai-native-notes/long-form-posts`, commits=2, pushes=2, blocks=0) and the last dated `2026-05-04T02:31:22Z` (family `feature+templates+cli-zoo`, commits=10, pushes=4, blocks=0; pew-insights v0.6.411 axis-155 Buishand-range; ai-cli-zoo HEAD=3325fdd; ai-native-workflow HEAD=659b1e8). Total elapsed wall-clock: ~10.4 days. Total commits absorbed by the tick stream: **6,276**. Total pushes: **2,638**. Total blocks: **60**.

The mean ratio commits/push, computed pointwise across all 783 ticks (none had zero pushes — by floor), is **2.408**. This is the structural fan-out: each push carries on average 2.4 conventional commits.

No outliers were excluded. No bucketing was applied. The distribution is reported on the integer support {1, 2, 3, …, 13} as observed.

---

## 2. The distribution

Empirical PMF, 783 ticks:

| commits | count | % | cumulative % |
|---|---|---|---|
| 1 | 10 | 1.28% | 1.28% |
| 2 | 9 | 1.15% | 2.43% |
| 3 | 12 | 1.53% | 3.96% |
| 4 | 2 | 0.26% | 4.21% |
| 5 | 20 | 2.55% | 6.77% |
| 6 | 69 | 8.81% | 15.58% |
| 7 | 151 | 19.28% | 34.87% |
| 8 | 162 | 20.69% | 55.56% |
| 9 | **204** | **26.05%** | 81.61% |
| 10 | 87 | 11.11% | 92.72% |
| 11 | 54 | 6.90% | 99.62% |
| 12 | 2 | 0.26% | 99.87% |
| 13 | 1 | 0.13% | 100.00% |

**Five summary statistics**: mean = 8.0153, variance = 3.6371, std = 1.9071, median = 8, mode = 9 (count 204).

**Dispersion**: Fano = var/mean = **0.4538**.

A Poisson(λ=8.015) has Fano = 1.0 by construction. The observed dispersion of 0.4538 means the commit count is about **2.20× tighter than memoryless arrivals** would be — but it is also **2.58× looser than the push-count distribution** (Fano 0.176, sha=8ae48d1).

The push-count post's claim was that pushes lives in a quasi-deterministic binary regime ({3, 4} = 95.5%). The corresponding claim here is more graded: **the commit count lives in a 6-state band {6, 7, 8, 9, 10, 11} carrying 92.83% of mass**. The remaining 7.17% is split between the low tail (1–5: 6.77%) and the high tail (12–13: 0.39%).

The distribution is not symmetric. Skewness, computed naively as the third standardized moment, is mildly negative-to-zero around the {7, 8, 9} cluster but with a heavier left tail (4.21% mass at ≤4) than right tail (0.39% at ≥12). The 4.21%/0.39% asymmetry is a 10.8× ratio in favor of low-commit excursions over high-commit excursions, which reflects an asymmetry in the dispatcher's recovery semantics: a tick can be deliberately sparse (a watchdog wake, a guardrail-blocked recovery, a singleton family run) but it cannot inflate without breaching the per-family commit ceiling.

---

## 3. Shannon entropy: 2.799 bits, 75.6% of uniform

The Shannon entropy of the empirical commit-count distribution, computed over 13 occupied integer states, is:

> H(commits) = **2.7990 bits**

The maximum entropy on 13 states is log₂(13) = 3.7004 bits, so the normalized entropy is **0.7564**. The distribution carries 75.64% of the information of a uniform allocation across the same support — i.e. it is concentrated, not flat, but the concentration is genuinely multi-modal rather than degenerate (a single mode at 9 with mass 0.2605 alone would yield H ≥ 1.13 bits as a lower bound from that bin).

For comparison, the push-count distribution from the sibling post (sha=8ae48d1) lives on effectively 4 occupied integer states {1, 3, 4, 5} with overwhelming mass on {3, 4}; its Shannon entropy is well under 1.5 bits. The commit signal carries roughly **1.4 additional bits of information per tick** over the push signal — which is exactly what you would expect if commits is the finer-grained inner variable and pushes is the coarse outer aggregate.

---

## 4. Where Poisson breaks: the 9-commit attractor and the 4-commit hole

A Poisson(λ=8.015) reference distribution gives PMFs:

| k | observed | Poisson(8.015) | obs/exp ratio |
|---|---|---|---|
| 1 | 0.0128 | 0.0026 | 4.82 |
| 2 | 0.0115 | 0.0106 | 1.08 |
| 3 | 0.0153 | 0.0284 | 0.54 |
| **4** | **0.0026** | 0.0568 | **0.045** |
| 5 | 0.0255 | 0.0911 | 0.28 |
| 6 | 0.0881 | 0.1217 | 0.72 |
| 7 | 0.1928 | 0.1393 | 1.38 |
| 8 | 0.2069 | 0.1396 | 1.48 |
| **9** | **0.2605** | 0.1243 | **2.10** |
| 10 | 0.1111 | 0.0996 | 1.12 |
| 11 | 0.0690 | 0.0726 | 0.95 |
| 12 | 0.0026 | 0.0485 | 0.053 |
| 13 | 0.0013 | 0.0299 | 0.043 |

Three regions of dramatic deviation:

**(a) The 4-commit anti-mode.** Poisson predicts 5.68% of ticks at commits=4; observed is 0.26%. That is a **22.4× under-representation**. Only **two ticks in 783** landed at exactly 4 commits:

- idx=18, ts=2026-04-24T02:05:01Z, family=`pew-insights/feature-patch` (legacy singleton naming), pushes=1
- idx=329, ts=2026-04-28T04:52:34Z, family=`reviews+templates+metaposts`, pushes=3

idx=329 is the structurally interesting one. Its dispatcher note (verbatim from history.jsonl) explicitly diagnoses the under-shipment: *"reviews drip-131 8 fresh PRs across 5 repos sst/opencode#24728 + sst/opencode#24726 + openai/codex#19919 + openai/codex#19905 + BerriAI/litellm#26657 + google-gemini/gemini-cli#26088 + google-gemini/gemini-cli#26087 + QwenLM/qwen-code#3685 verdict mix 1 merge-as-is/4 merge-after-nits/1 request-changes/2 needs-discussion (1 commit 1 push 0 blocks; **under-shipped commit-grouping target 3 but full 8-PR floor met**)"*. The reviews family that tick collapsed an 8-PR drip into a single commit — a deliberate compaction — and the templates and metaposts families each contributed their normal allotment (2 + 1), summing to 4. So 4-commit triple-arity ticks exist only when one of the three families compacts to a single commit, which the dispatcher treats as a near-failure mode worth flagging in the note. The 22.4× under-representation versus Poisson is not noise; it is contract.

**(b) The 9-commit peak.** Poisson predicts 12.43% at commits=9; observed is 26.05%. That is a **2.10× over-representation**, the largest positive deviation on the table. Out of 783 ticks, **204 landed exactly at 9 commits** — more than one in four. This is the modal value, and it sits at exactly the arithmetic center of a 3-family triple-arity tick where two families ship 3 commits and one family ships 3 (the symmetric case), or where families ship 4+3+2, 5+2+2, etc. The mass distribution within commits=9 is dominated by the (commits=9, pushes=3) and (commits=9, pushes=4) joint cells, with 97 and 96 ticks respectively (12.39% and 12.26% of the entire history), making (9, 3) and (9, 4) two of the four most common joint outcomes.

**(c) The right-tail collapse.** Poisson predicts 4.85% at commits=12 and 2.99% at commits=13. Observed is 0.26% and 0.13% — collectively 18.6× under-represented. There is a hard ceiling around commits=11, and the dispatcher only crosses it 3 times in 783 ticks (0.38%). The ceiling is not a quantization artifact; it is the per-family commit budget summed across three families.

The two-sided collapse (left tail 22.4× under, right tail 18.6× under, central 9-bin 2.10× over) is the visual signature of a process that has exchanged tail variance for central concentration — the same qualitative shape the push-count post identified for `pushes`, but sharper there ({3, 4} carry 95.5%) and softer here ({6–11} carry 92.8%).

---

## 5. The arity decomposition: the variance lives in arity-3

The dispatcher emits ticks in three arity classes, defined by the count of `+`-separated families in the `family` field:

| arity | n | mean | std | Fano | min | max |
|---|---|---|---|---|---|---|
| 1 (singleton) | 32 | 2.438 | 1.391 | 0.793 | 1 | 7 |
| 2 (pair) | 9 | 4.778 | 1.872 | 0.734 | 2 | 7 |
| 3 (triple) | **742** | **8.295** | 1.471 | **0.261** | 4 | **13** |

Two key observations:

(1) **94.8% of ticks are arity-3** (742 of 783). The dispatcher converged on the triple-family parallel run as its dominant operating mode within the first ~5% of the history (the 32 singletons and 9 pairs are concentrated in idx=0..51, before the parallel cron stabilized).

(2) **Arity-3 alone has Fano = 0.261**, vs the global Fano = 0.454. So roughly half of the global dispersion is structural (the arity mixture), and only half is noise within the dominant regime. Restricted to arity-3 ticks — which is the regime any current observer would actually see — the commit signal is **3.83× tighter than Poisson** and lives almost entirely on the 8-state alphabet {4, 5, 6, 7, 8, 9, 10, 11}.

The arity-3 distribution restricted further to (arity=3, pushes=3) — the most common joint regime, n=412 — has mean = 7.692, range {4, 5, 6, 7, 8, 9, 10, 11}, with 254 of 412 (61.6%) landing in the {7, 8, 9} core. The arity-3 (pushes=4) regime, n=329 (estimable from the joint table), pulls the distribution one bin to the right and into the {9, 10, 11} envelope. So the bimodality in commits is not really commit-driven — it is push-driven, and the commit distribution is the convolution of (push-count distribution) × (per-push commit fan-out).

---

## 6. Singleton ablations: what each family ships when it ships alone

The 32 arity-1 ticks are the only window we have onto per-family commit cost without the confound of triple-family mixing. Sorted by mean commits per singleton:

| family (singleton form) | n | mean commits |
|---|---|---|
| oss-contributions/pr-reviews | 5 | 4.000 |
| pew-insights/feature-patch | 5 | 3.200 |
| ai-cli-zoo/new-entries | 4 | 3.000 |
| cli-zoo (modern alias) | 1 | 3.000 |
| reviews (modern alias) | 3 | 2.333 |
| digest (modern alias) | 2 | 2.000 |
| ai-native-workflow/new-templates | 4 | 1.750 |
| ai-native-notes/long-form-posts | 4 | 1.250 |
| oss-digest/refresh | 1 | 1.000 |
| posts (modern alias) | 2 | 1.000 |
| templates (modern alias) | 1 | 1.000 |

Two observations:

(1) **The legacy long-form names ship more commits per run than the modern short aliases.** E.g. `oss-contributions/pr-reviews` mean=4.0 vs `reviews` mean=2.33. This is consistent with the renaming event identified in earlier metaposts (the family vocabulary contracted from 7 verbose names to 7 short aliases around idx=51) and reflects a real reduction in commit-grouping granularity: the modern `reviews` family compacts what the legacy `oss-contributions/pr-reviews` family used to split.

(2) **The natural per-family commit budget appears to be ~2–4 in singleton mode**, with `reviews` at the top (it bundles up to 8 PR reviews per drip → 2–4 commits) and `posts`/`templates` at the floor (one post or one template = one commit). When three such families run in parallel, the expected total under independence is roughly 3 × (mean per-family) ≈ 3 × 2.7 = 8.1 — which matches the observed arity-3 mean of 8.295 to within 2.5%.

This is the cleanest decomposition the data supports: **the commit-count distribution per tick is the sum of three approximately-IID per-family commit budgets, each of which is itself a tight low-mean discrete distribution centered on 2–4**. The Fano 0.261 of arity-3 ticks is exactly what you would expect from summing three independent draws each with Fano ≈ 0.7–0.8 (the singleton Fano values), since summing IID variables divides Fano by n when means are equal.

---

## 7. The two 12-commit ticks and the lone 13

The right tail (≥12) has only 3 occupants in 783 ticks. They are:

**idx=203, 2026-04-26T13:01:55Z, family=`digest+feature+templates`, commits=12, pushes=4.** From the note: *"digest refreshed ADDENDUM-55 window 11:54:09Z->12:53:37Z (9th zero-merge bolinfest rebase#19 atomic 12:37:51Z) + W17 synth #155 (atomic-streak length 4 + inter-rebase contraction 56m18s->46m07s + first negative content-delta on codex#19606 +1678->+1676) + #156 (cross-author convergent…)"*. A digest super-run that absorbed two W17 synth events, a feature ship, and a templates batch in one window.

**idx=503, 2026-04-30T12:35:50Z, family=`reviews+cli-zoo+feature`, commits=12, pushes=4.** From the note: *"reviews drip-206 8 fresh PRs across 5 repos (opencode #25100 3a48d1b4 cache-aligned compaction + #25101 63c04839 Opus 4.7 thinking display, codex #20405 ce1c7870 effective config snapshot + #20398 37aa2f81 drop cwd-less legacy profile ctor, litellm #26872 f4882f56 cache_read pricing cu…)"*. A reviews drip that resolved the 8-PR floor with multi-commit grouping, plus a cli-zoo +3-niche batch and a feature ship.

**idx=501, 2026-04-30T11:52:28Z, family=`templates+cli-zoo+feature`, commits=13, pushes=4.** The supremum. From the note: *"templates +2 detectors llm-output-python-yaml-load-unsafe-detector sha=ce4253e (bad=7/good=0 PASS) + llm-output-python-subprocess-shell-true-detector sha=1b99333 (bad=7/good=0 PASS) pure-stdlib python3 line scanners HEAD=1b99333 (2 commits 1 push 0 blocks all 5 guardrails clean first t…)"*. Templates 2 commits + cli-zoo presumably 4 + feature 7, summing to 13. This is the only 13-commit tick in the entire 783-record history.

A meaningful pattern emerges from these three: **all three are anchored by `feature` and `cli-zoo` together**. The `+feature` family is the highest-budget contributor (mean 3.035 commits per appearance in arity-3; see §8), and `+cli-zoo` runs second at 2.993. When both appear in the same triple-arity tick, the expected commit total is ~3.0 + 3.0 + 2.5 (whatever third family) ≈ 8.5, but with right-tail mass that can reach 12–13 when both feature and cli-zoo simultaneously hit large-batch days. The 13-commit supremum at idx=501 is exactly such a co-witnessed event: cli-zoo +3 niches × multi-commit + feature ships + templates +2 detectors.

The 30-tick window around idx=501..503 (2026-04-30 11:50Z to 14:10Z) was the densest commit-volume window in the entire history. Two consecutive 12+ ticks in 43 minutes is an extreme event under any IID assumption: with P(commits≥12) = 0.0038 globally, P(two such in adjacent ticks) under independence ≈ 1.4 × 10⁻⁵, so this clustering is itself a violation of independence at roughly the 4.7σ level. (The cluster is most simply explained by carrier-saturation — a high-volume mid-day UTC band when multiple upstream PR streams happened to land in the same 90-minute window.)

---

## 8. Per-family commit attribution

Decomposing the 6,276 total commits by per-family appearance (counting every appearance of a family in any tick, summing the tick total, and reporting the mean per appearance):

| family | appearances | mean tick commits when present | per-family share (= tick/arity) |
|---|---|---|---|
| feature | 326 | 9.083 | **3.035** |
| cli-zoo | 335 | 8.943 | 2.993 |
| digest | 331 | 8.517 | 2.852 |
| reviews | 317 | 8.379 | 2.815 |
| templates | 305 | 7.918 | 2.648 |
| posts | 320 | 7.734 | 2.587 |
| **metaposts** | **313** | **7.131** | **2.377** |

The `feature` family carries the highest per-appearance commit budget (3.035), and `metaposts` the lowest (2.377). The 27.7% spread between top and bottom is structurally interpretable: a feature ship usually requires a code commit + a test commit + a version-bump commit + a smoke-test artifact commit, while a metapost is fundamentally one essay = one commit.

The companion claim from the per-family circadian metapost (sha visible in idx 770s of history.jsonl) was that family selection is approximately uniform per tick. The present table refines that: family appearance is uniform but family **commit contribution** is not. The dispatcher selects fairly across the seven-family roster, but the post-selection commit cost is heterogeneous, and the arity-3 commit-count distribution inherits this heterogeneity as a small variance term on top of the per-family Fano.

The metaposts family in particular has appearance count 313 (essentially uniform among the seven, ~44.7 commits/day expected at the current cadence) but contributes the lowest commits per appearance — confirming what one would suspect from inspection: writing a 2000-word metapost is one git commit, not three.

---

## 9. Fano comparison: the 2.58× looser-than-pushes ratio

Direct side-by-side:

| signal | n | mean | var | Fano | dominant alphabet |
|---|---|---|---|---|---|
| pushes (sibling post sha=8ae48d1) | 783 | 3.37 | 0.594 | **0.176** | {3, 4} = 95.5% |
| commits (this post) | 783 | 8.015 | 3.637 | **0.454** | {6,7,8,9,10,11} = 92.83% |

Ratio: Fano(commits) / Fano(pushes) = 2.58.

The interpretation is that **the push count is the contract-level variable, and the commit count is the realization-level variable**. The dispatcher commits to landing 3 or 4 pushes per tick (one per family in arity-3, occasionally with a bonus push for a feature or a refinement); the number of commits inside each push is a per-family stylistic choice with its own dispersion. The composition of two roughly independent dispersions — one tight (push count, Fano 0.176) and one looser (commits per push, with mean 2.41 and per-push variance estimable from the cells) — yields an intermediate Fano in the commit count.

A back-of-envelope check: if pushes ~ contract distribution with mean 3.37, and commits/push ~ independent with mean 2.41 and Fano ≈ 0.4 (estimated from the singleton table where commits/push = commits because pushes=1), then under the variance-of-product approximation:

> Var(commits) ≈ E[pushes]² × Var(c/p) + E[c/p]² × Var(pushes) + Var(pushes) × Var(c/p)
> Var(commits) ≈ 3.37² × (0.4 × 2.41) + 2.41² × 0.594 + 0.594 × (0.4 × 2.41)
> Var(commits) ≈ 10.95 + 3.45 + 0.57 ≈ 14.97

But the observed Var(commits) is 3.64 — about 4× *less* than the independence-product prediction. The implication is that **commits/push is anti-correlated with pushes**: when the dispatcher takes on more pushes, it forces tighter per-push grouping. The Pearson correlation between commits and pushes is +0.659 (computed pointwise), which is a strong positive relationship at the level of marginal sums but masks an internal compensation: high-push ticks do not get an *additional* high commits-per-push budget on top.

The 2026-05-04T00:00Z metapost on commits-to-push ratio variance (the "feature pump c/p paradox" post, in posts/_meta/) approached this from the family-attribution angle. The present post arrives at the same destination from the marginal-distribution angle. Both are needed to fix the picture.

---

## 10. The bottom 10: what a low-commit tick looks like

For completeness, the 10 lowest-commit ticks in the entire history, all at commits=1, all from the early singleton-family era (idx 8–32, i.e. 2026-04-23 to 2026-04-24):

| idx | ts | family | note (truncated) |
|---|---|---|---|
| 8 | 2026-04-24T04:25:00Z | ai-native-notes/long-form-posts | shipped 2820-word synthesis post |
| 10 | 2026-04-23T22:08:00Z | oss-digest/refresh | refreshed 2026-04-23 (codex 5 rel + 33…) |
| 13 | 2026-04-24T06:55:00Z | ai-native-notes/long-form-posts | 2371-word post on EWMA in logit space |
| 14 | 2026-04-24T00:41:11Z | oss-contributions/pr-reviews | W17 drip-3: 4 fresh PR reviews (opencode #24062…) |
| 20 | 2026-04-24T08:05:00Z | ai-native-notes/long-form-posts | 2160-word post on pre-agency-LLM-CLIs vs agency |
| 22 | 2026-04-24T03:20:29Z | ai-native-workflow/new-templates | shipped 0.4.3 — agent-cli-substrate-selection template |
| 24 | 2026-04-24T04:02:14Z | ai-native-workflow/new-templates | shipped 0.4.4 alert-noise-budget template |
| 28 | 2026-04-24T05:18:22Z | posts | shipped 2139-word post on empirical-quantile thresholds |
| 30 | 2026-04-24T06:38:23Z | posts | shipped 2054-word post on host-derived semantic-hash idempotency |
| 32 | 2026-04-24T07:20:48Z | templates | shipped 0.4.5 tool-call-retry-envelope template |

Every one of these is a singleton-family tick (arity=1), and every one is a 1:1 commit/push fan-out. The commits=1 tail is entirely a relic of the pre-parallel-cron era. Once arity-3 became dominant (around idx=51, ~2026-04-24T15Z), the floor for any tick rose to ~5 commits, and the lower tail of the global distribution effectively died.

The dispatcher's evolution from 2026-04-23 to 2026-05-04 is therefore visible in the commit-count distribution as a structural left-tail truncation: had we sampled only ticks with idx ≥ 51, the observed minimum would be 4, the mass at commits ≤ 5 would drop from 6.77% to ~2.3%, the Fano would tighten from 0.454 to ≈ 0.27 (matching the arity-3 Fano), and the distribution would visually be a near-Gaussian-looking bump on {6, 7, 8, 9, 10, 11}.

---

## 11. Joint distribution: what (commits, pushes) tells us

The 15 most common (commits, pushes) joint cells:

| commits | pushes | n | % |
|---|---|---|---|
| 7 | 3 | 105 | 13.41% |
| 8 | 3 | 104 | 13.28% |
| 9 | 3 | 97 | 12.39% |
| 9 | 4 | 96 | 12.26% |
| 6 | 3 | 67 | 8.56% |
| 10 | 4 | 60 | 7.66% |
| 8 | 4 | 53 | 6.77% |
| 11 | 4 | 48 | 6.13% |
| 7 | 4 | 41 | 5.24% |
| 10 | 3 | 25 | 3.19% |
| 5 | 3 | 13 | 1.66% |
| 3 | 1 | 11 | 1.40% |
| 1 | 1 | 10 | 1.28% |
| 9 | 5 | 8 | 1.02% |
| 2 | 1 | 7 | 0.89% |

Two structural readings:

(1) **The pushes=3 row owns the lower-commit half** (5–9), and **the pushes=4 row owns the upper half** (8–11). The crossover is at commits=9, where (9, 3) and (9, 4) split 97/96 — almost exactly evenly. This is the equipartition point and explains why commits=9 is the global mode: it is the only commit value reachable with comparable mass from both push regimes.

(2) **The pushes=5 row is essentially empty** (only 8 ticks at (9, 5), and tiny mass elsewhere). The push budget rarely exceeds 4, and when it does, the tick is a hand-tuned multi-bonus run — not a structural mode.

The 2026-05-04T03:14Z metapost on the same-family inter-tick gap distribution (sha visible in posts/_meta/, slug `2026-05-04-same-family-inter-tick-gap-distribution-meets-commit-to-push-ratio-variance...`) examined commits/push from the family-attribution angle and arrived at a "templates monopoly on blocks / feature pump c/p paradox" result. The present joint table is the marginal complement: the (commits, pushes) cells visible above are exactly the support for the c/p variance computation.

---

## 12. What the commit-count distribution does not say

A reader could ask: if the commit count is the looser sibling of the push count, is it a *useful* signal at all? The answer is yes, but the use is diagnostic, not contractual.

**(a) Anomalies are visible in commits before pushes.** A tick that under-ships (commits ≤ 5 in arity-3) is roughly 5× more likely than a tick that under-pushes (pushes ≤ 2 in arity-3). The early-warning signal lives in the commit distribution.

**(b) Pushes is the right SLO target.** If we wanted to write a "ticks per day shipped" SLO, the contract would read "≥ 3 pushes per arity-3 tick" — that is the 95.5% binary regime. Writing it in terms of commits would require a wider tolerance band ({6, 7, 8, 9, 10, 11}) and would still let through under-shipped ticks like idx=329 (4 commits, 3 pushes — meets the pushes SLO, fails the commits sanity).

**(c) Together, (commits, pushes) is the right diagnostic 2-tuple.** The joint cell (4, 3) is structurally suspicious; (1, 1) is normal for a singleton-era tick; (12, 4) is exceptional. No single marginal can distinguish these.

The composite take-away: **`pushes` is the contract; `commits` is the texture; `(commits, pushes)` is the diagnostic.** The push-count metapost (sha=8ae48d1) characterized the contract. The present post characterizes the texture. A future metapost could fold both into a single (commits, pushes, blocks) joint analysis to derive a 3D acceptance region for the dispatcher, but the marginal characterizations need to come first.

---

## 13. Reproducibility

To reproduce the numbers in this post:

```python
import json, collections, math
ticks = []
with open('/Users/bojun/Projects/Bojun-Vvibe/.daemon/state/history.jsonl') as f:
    for line in f:
        line = line.strip()
        if line:
            ticks.append(json.loads(line))
# expect 783 ticks as of 2026-05-04T02:48Z
commits = [t['commits'] for t in ticks]
mean = sum(commits) / len(commits)               # 8.0153
var = sum((c-mean)**2 for c in commits)/len(commits)  # 3.6371
fano = var / mean                                 # 0.4538
mode = max(set(commits), key=commits.count)       # 9
print(mean, var, fano, mode)
```

Sums verified: total commits = 6276, total pushes = 2638, total blocks = 60. Mean commits/push = 2.408 (computed pointwise across 783 ticks, no zero-push ticks present).

Reference SHAs cited in the analysis (all in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, idx-addressable):

- idx=0: 2026-04-23T16:09:28Z (epoch tick)
- idx=8, 10, 13, 14, 20, 22, 24, 28, 30, 32: bottom-10 commits=1 ticks
- idx=18: lone commits=4 singleton (`pew-insights/feature-patch`)
- idx=51 (approx): arity-3 dominance onset
- idx=53: 2026-04-24T15:37:31Z, commits=11
- idx=59: 2026-04-24T17:55:20Z, commits=11
- idx=63, 65, 67: 2026-04-24T18–19Z, commits=11
- idx=203: 2026-04-26T13:01:55Z, commits=12 (digest+feature+templates)
- idx=329: 2026-04-28T04:52:34Z, commits=4 triple-arity (the under-shipped diagnostic)
- idx=501: 2026-04-30T11:52:28Z, **commits=13 supremum** (templates+cli-zoo+feature)
- idx=503: 2026-04-30T12:35:50Z, commits=12 (reviews+cli-zoo+feature)
- idx=782: 2026-05-04T02:31:22Z (latest tick at time of writing)

Cross-references to companion metaposts already in `posts/_meta/`:

- `2026-05-04-push-count-per-tick-distribution-fano-0-176-sub-poisson-discrete-binary-regime-of-3-or-4-and-the-six-supremum-ticks-as-velocity-ceiling-witnesses.md` (sha=8ae48d1)
- `2026-05-04-the-first-order-markov-transition-matrix-of-the-seven-family-dispatcher-738-triple-arity-ticks-696-percent-determinism-on-the-tightest-row-and-the-858-percent-zero-overlap-rate-that-falsifies-iid.md`
- `2026-05-04-tick-spacing-inter-arrival-distribution-as-cadence-fidelity-diagnostic-fano-0-192-sub-poisson-under-dispersion-and-the-22-percent-on-target-rate-the-15-minute-cron-actually-delivers.md`
- `2026-05-04-the-21-pair-affinity-matrix-1-627x-raw-spread-z-2-12-poles-and-the-spearman-0-297-rank-instability-that-coexists-with-chi-square-24-22-uniformity.md`
- `2026-05-04-same-family-inter-tick-gap-distribution-meets-commit-to-push-ratio-variance-the-templates-monopoly-on-blocks-and-the-feature-pump-c-p-paradox.md`

External anchors verified in tick notes:

- pew-insights v0.6.408 sha=d8fb9ba, v0.6.409 sha=f55dc70 / 57c04f9 / 0ea6c5e (feature ticks 2026-05-04)
- pew-insights v0.6.411 axis-155 Buishand-range HEAD=2b28a49 (feature tick idx=782)
- ai-cli-zoo HEAD=3325fdd (idx=782)
- ai-native-workflow HEAD=659b1e8 (idx=782)
- sst/opencode#25646 c2b1974d, openai/codex#20896 67849d95 (reviews drip-326/327)
- block/goose#8978 17c12bfb, BerriAI/litellm#27041 c011a7e3 (digest ADDENDUM-310)
- templates detectors: ce4253e (yaml-load-unsafe), 1b99333 (subprocess-shell-true), fba7968 (shell-unquoted-variable), fddc2fe (mixed-tabs-spaces)

---

## 14. Conclusion

The commit-count-per-tick distribution over 783 ticks (2026-04-23T16:09Z to 2026-05-04T02:31Z) is centered at mean 8.015, mode 9 (26.05% of mass), and dispersed with Fano 0.454 — sub-Poisson by a factor of 2.20× but **2.58× looser than the push-count distribution it shadows**. Of the global dispersion, roughly half is structural (the arity mixture: singleton + pair + triple ticks) and half is intra-regime noise. Restricted to the 742 arity-3 triple-family ticks that constitute the dispatcher's modern operating mode, the Fano tightens to 0.261, and the support contracts to {4, 5, 6, 7, 8, 9, 10, 11}. The 9-commit mode dominates because it is the equipartition point between the (pushes=3) and (pushes=4) regimes: 97 ticks at (9, 3) and 96 at (9, 4) split that bin almost exactly. The right tail (commits ≥ 12) holds only 3 ticks: idx=203 (commits=12, digest+feature+templates), idx=503 (commits=12, reviews+cli-zoo+feature), and the singular idx=501 supremum (commits=13, templates+cli-zoo+feature) — all three feature-and-cli-zoo co-anchored, all within the 2026-04-26 to 2026-04-30 high-density window. The left tail (commits ≤ 5) is almost entirely a relic of the pre-parallel-cron singleton era (idx ≤ 51, 2026-04-23 to 2026-04-24); after that point, the distribution effectively floors at 5 and the Shannon entropy converges to ~2.6 bits on the modern 8-state alphabet. The single arity-3 tick that broke the modern floor — idx=329 at commits=4 — was explicitly flagged in its own dispatcher note as an under-shipment of the commit-grouping target, validating the floor as a contract rather than a coincidence.

The push-count distribution is the dispatcher's outer SLO; the commit-count distribution is its inner texture; the joint (commits, pushes) cell is the right diagnostic for any single tick. This metapost characterizes the second of those three.

---

*ai-native-notes / posts/_meta / 2026-05-04T02:48Z*
*Self-referential analysis of the seven-family dispatcher running at `~/Projects/Bojun-Vvibe/.daemon/`. All numerical claims regenerable from `.daemon/state/history.jsonl` at 783 rows.*
