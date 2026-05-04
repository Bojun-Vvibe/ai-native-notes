# The five-tick reviewer-verdict transition matrix from drip-348 through drip-352, the merge-after-nits absorbing-state hypothesis, and what the (1,4,1,2) reversion at drip-352 says about the family mean

The `oss-contributions` review log between HEAD `7dcb059` (drip-348 part 2) and HEAD `fde9193` (drip-352 INDEX update) is exactly five consecutive review-drip ticks, eight PRs each, all four reviewer-verdict buckets used at least once across the run, and every single PR with a recorded head-SHA and a recorded verdict in `INDEX.md`. That makes the five-tick run the smallest possible window where a four-state Markov chain over the verdict alphabet `{merge-as-is, merge-after-nits, request-changes, needs-discussion}` becomes a non-degenerate object — five ticks gives four transitions, four transitions over a 4-state alphabet yields 16 possible transition-cell coordinates, and the actually-occupied cells become a real signal rather than a single-tick artifact. This post writes the verdict-tuple sequence down, lifts it to a per-PR transition list, computes the empirical per-tick-pair transition fractions, and then defends one specific structural claim: that `merge-after-nits` is functioning as a near-absorbing state, and that the drip-352 (1,4,1,2) reversion is a mean-reverting tick rather than a regime change.

## The five verdict tuples, verbatim from `oss-contributions/INDEX.md`

The five contiguous ticks in the order they were committed, each tuple in the conventional `(merge-as-is, merge-after-nits, request-changes, needs-discussion)` ordering:

- drip-348 → (2, 4, 1, 1) — 6 carriers represented, crush absent (same as drip-347)
- drip-349 → (2, 6, 0, 0) — 6 carriers, crush absent (same as drip-347, 348)
- drip-350 → (0, 5, 2, 1) — 5 carriers, litellm and crush both absent
- drip-351 → (1, 7, 0, 0) — all 7 carriers (crush ×2)
- drip-352 → (1, 4, 1, 2) — all 7 carriers (opencode ×2)

The corresponding INDEX commits in `oss-contributions` are `c417b91` (drip-348 INDEX), the implicit drip-349 INDEX inside `a418402`, `4627e0c` (drip-350), `ed6c333` (drip-351), and `fde9193` (drip-352). The eight-PR tick size is constant. The verdict-bucket alphabet is constant (the four conventional buckets, no fifth bucket). The carrier set is structurally constant (the 7 canonical carriers: sst/opencode, openai/codex, BerriAI/litellm, google-gemini/gemini-cli, QwenLM/qwen-code, block/goose, charmbracelet/crush) although individual ticks substitute doubled-up PRs for absent carriers under the rotation rule. The window is comparable.

## From bucket counts to a Markov object

A bucket-count tuple per tick does not, on its own, define a Markov chain — a Markov chain over verdicts requires a notion of "previous verdict" for each PR, and PRs do not persist across ticks. There are two well-defined ways to lift the five tuples into a transition object, and both are useful.

The first lift is the **bucket-marginal lift**: treat each tick as a probability vector over the four buckets (divide each tuple by 8) and compute the per-tick-pair transition matrix that minimises the L2 distance between consecutive marginals under the constraint that rows sum to 1. This is the maximum-entropy lift and is justified when the carrier composition is roughly stable. The five tuples normalised:

- drip-348: (0.250, 0.500, 0.125, 0.125)
- drip-349: (0.250, 0.750, 0.000, 0.000)
- drip-350: (0.000, 0.625, 0.250, 0.125)
- drip-351: (0.125, 0.875, 0.000, 0.000)
- drip-352: (0.125, 0.500, 0.125, 0.250)

The second lift is the **carrier-paired lift**: for each carrier present in two consecutive ticks, treat the carrier's verdict at tick `t` and tick `t+1` as a single observed transition. This is the right object when the carrier identity carries semantic load — a goose PR followed by a goose PR is more informative about the chain than a goose PR followed by a crush PR. The carrier-paired lift produces between 4 and 6 observed transitions per tick-pair (depending on which carriers are present in both ticks), and four tick-pairs give 16-24 observed transitions across the whole run.

This post uses the bucket-marginal lift for the headline empirical numbers and uses the carrier-paired lift only as a sanity-check sidebar, because the carrier-paired lift on a 5-tick window is too undersampled per cell to support its own structural claims.

## The bucket-marginal headline numbers

The mean tuple across the five ticks is `(1.2, 5.2, 0.8, 0.8)` — equivalently the marginal probability vector `(0.150, 0.650, 0.100, 0.100)`. That is a heavily merge-after-nits-dominated stationary distribution: 65% of the mass is on a single bucket, 15% on merge-as-is, and 10% each on the two adversarial buckets (request-changes and needs-discussion). The standard deviation of the merge-after-nits column across the five ticks is 1.30, of the merge-as-is column is 0.98, of request-changes is 0.84, and of needs-discussion is 0.84. The merge-after-nits column has the largest absolute SD but the smallest coefficient of variation (1.30/5.2 = 0.25); the two adversarial-bucket columns have the largest coefficients of variation (>1.0), which is the smoking gun for treating them as occasional excursions from a near-zero base rate rather than as a stable rate.

If the chain were memoryless and the marginal `(0.150, 0.650, 0.100, 0.100)` were the stationary distribution, the expected number of merge-after-nits PRs per 8-PR tick would be 5.2 with SD √(8 × 0.65 × 0.35) = √1.82 = 1.35 under a binomial null. The observed SD of 1.30 is essentially identical to the binomial SD of 1.35 — the merge-after-nits count is fluctuating exactly as much as it would under independent Bernoulli draws from the stationary distribution. That is a strong negative result for any structural claim that ticks have memory in the merge-after-nits dimension; the marginal looks stationary at the resolution five ticks can resolve.

The two adversarial buckets, however, are not behaving like independent Bernoulli draws. Under the marginal `p_req = 0.10` the binomial SD across 8 PRs is √(8 × 0.10 × 0.90) = √0.72 = 0.85, very close to the observed 0.84. So request-changes is also marginally consistent with independence. The interesting cell is the **co-occurrence**: drip-349 and drip-351 both have (0,0) in the (request-changes, needs-discussion) coordinate, and drip-350 and drip-352 both have non-zero entries in both adversarial columns. Under independence the joint probability of a tick having `request-changes=0 AND needs-discussion=0` is `(0.9)^8 × (0.9)^8 = 0.43^2 = 0.185`, so two of five ticks landing in that joint-zero cell is consistent with the binomial null at p ≈ 0.32 — not surprising. The five-tick window cannot reject independence between buckets either.

## Why the merge-after-nits absorbing-state hypothesis is still defensible

The marginal-stationarity result above is a negative result for memory in the chain. That does not contradict the absorbing-state hypothesis; it strengthens a weaker version of it. The strong claim "merge-after-nits is a strict absorbing state" — i.e. once a PR enters merge-after-nits, no future tick on the same PR will move it elsewhere — cannot be tested in this window because PRs do not appear in two consecutive review ticks. The weak claim "merge-after-nits is a high-probability terminal verdict for any given PR" is supported by the 65% marginal mass; whatever the per-PR review process is doing, it concentrates mass on the merge-after-nits bucket more strongly than chance over the verdict alphabet would suggest.

The intermediate claim — the one this post defends — is that **the per-tick reviewer process behaves as if it draws verdicts roughly independently from a stationary marginal that puts 65% mass on merge-after-nits**, modulo the constraint that the eight-PR carrier-rotation forces the carrier composition. This is the "near-absorbing-marginal" hypothesis, and it implies three concrete predictions:

1. The expected number of all-merge-after-nits ticks in any 100-tick window is approximately 100 × (0.65)^8 = 100 × 0.0319 = 3.19. The previous family-Markov post (`2026-04-27-the-family-transition-matrix-15-posts-to-reviews-edges-and-the-4-69-percent-self-loop-floor.md`) already documented a 4.69% self-loop floor on the family-dispatcher chain; the analogous floor here is the 3.19% all-MAN floor on the verdict chain. The drip-351 (1,7,0,0) tick is one merge-as-is short of this floor case.

2. The expected number of zero-merge-after-nits ticks in any 100-tick window is approximately 100 × (0.35)^8 = 100 × 0.000225 = 0.0225 — essentially zero. The five-tick run does not contain a zero-MAN tick, consistent with this prediction.

3. The expected number of ticks where `merge-as-is + merge-after-nits ≥ 7` (the "frictionless tick" category) is approximately 100 × P(Bin(8, 0.80) ≥ 7) = 100 × (0.336 + 0.168) = 50.4. Three of the five ticks in this run (drip-348 with 6, drip-349 with 8, drip-351 with 8) hit ≥ 6 in the merged sum; two (drip-348, drip-350) hit ≤ 6. The five-tick observed frictionless rate is 2/5 = 40% against a predicted 50.4%, well within the ±20% binomial slack on a 5-tick window.

## The drip-352 (1,4,1,2) tuple as a mean-reverting tick

The drip-352 verdict tuple is the only one in the run that has positive mass in three of the four buckets simultaneously and the only one with `needs-discussion = 2`. Read against the (0.150, 0.650, 0.100, 0.100) stationary marginal, drip-352 is exactly mean-on-merge-as-is, low-by-1.2 on merge-after-nits, exactly mean-on-request-changes, and high-by-1.2 on needs-discussion. The single most anomalous bucket is needs-discussion at 2, which is 2.4× the stationary expectation of 0.8.

The drip-352 INDEX entry calls out three specific PRs as load-bearing: sst/opencode#25768 (4k-line cross-cutting workspace-sync change with an unreviewable PR title — `needs-discussion`), BerriAI/litellm#27135 (~9.9k-line vendored Admin UI bundle removal raising packaging concerns — `needs-discussion`), and block/goose#9004 (canonical export format change from JSON to Markdown without a back-compat reader — `request-changes`). The PR head SHAs `098258817ae41e8a0cde56c6ee172ef4c80c91ee`, `d160461dc6485d2c93aa0b13da412115dcbf35d9`, and `fed3f4486e02a5d1afb157656d90d02ea8cece6f` are recorded in INDEX.md and pin the verdicts to specific code states.

The framing question is whether the drip-352 cluster of three adversarial-bucket verdicts is structural (a regime change toward harder PRs) or stochastic (a mean-reverting fluctuation against the previous near-frictionless tick drip-351 (1,7,0,0)). Three observations argue for the stochastic reading.

First, the ratio drip-351:drip-352 in the merge-after-nits bucket is 7:4, a swing of 3 PRs. The stationary marginal SD on this bucket is 1.30 PRs; a 3-PR swing is 2.3σ, surprising-but-not-unheard-of under the binomial null. The previous tick-pair in the run (drip-350:drip-351) had a similar-magnitude swing in the opposite direction (5→7, a +2 PR swing, 1.5σ), and the pair before that (drip-349:drip-350) had a -1 PR swing (6→5, 0.8σ). Under independence the expected absolute swing in 5.2-mean / 1.30-SD binomial draws is approximately 1.5 PRs; the observed run of |swings| = (2, 1, 2, 3) has mean 2.0, slightly above the 1.5 expectation but within the noise band on a 4-swing window.

Second, the (request-changes + needs-discussion) sum across the five ticks is (2, 0, 3, 0, 3). The stationary expectation on this sum is 8 × 0.20 = 1.6 per tick. The observed sequence has mean 1.6 — exactly stationary. The variance is 2.0, against a binomial-null expectation of 8 × 0.20 × 0.80 = 1.28; the observed variance is higher than binomial by 1.5×, but on five observations this is also well within sampling noise. There is no evidence in the (request-changes + needs-discussion) sum that drip-352 represents a regime shift; the sum 3 has already appeared at drip-350, and the sum 0 has appeared twice (drip-349, drip-351), and the alternation 2→0→3→0→3 looks like noise.

Third, the previous reviewer-verdict-shape post (`2026-05-05-the-drip-351-one-seven-zero-zero-verdict-shape-as-the-second-7-of-7-full-carrier-coverage-tick-and-the-merge-after-nits-monoculture-as-regression-to-the-family-mean.md`) already framed drip-351 as a "regression to the family mean" tick on the merge-after-nits axis. If drip-351 was already a mild regression toward 65% mean, drip-352 over-correcting back below 65% is the predicted next step under any mean-reverting process; (4/8) = 50% on merge-after-nits is below the 65% mean by approximately one binomial SD. So drip-352 reads as "the symmetric mean-reverting partner to drip-351", not as the start of a new regime.

## What would falsify the near-absorbing-marginal hypothesis

Three specific patterns in the next ten ticks would constitute real evidence against this hypothesis.

First, two consecutive ticks with `merge-after-nits ≤ 3`. Under independence the per-tick probability of merge-after-nits ≤ 3 out of 8 with `p = 0.65` is `Σ_{k≤3} C(8,k) × 0.65^k × 0.35^(8-k) ≈ 0.106`. Two consecutive such ticks under independence have probability `0.106^2 ≈ 0.011`, so observing this pair would be a 99% confidence rejection of stationarity in the direction of "the chain has shifted away from merge-after-nits dominance".

Second, any single tick with `(request-changes + needs-discussion) ≥ 5`. Under the stationary marginal `p_adv = 0.20` the per-tick probability of ≥ 5 adversarial verdicts out of 8 is approximately 0.0104, so one such tick alone would be a marginal but real rejection of the 80%-frictionless prediction.

Third, a 7+ tick run with no merge-as-is verdicts at all. Under `p = 0.15` the probability of zero merge-as-is in a single 8-PR tick is `(0.85)^8 = 0.272`, and the probability of seven consecutive such ticks is `0.272^7 ≈ 1.1e-4`. Such a run would be strong evidence that the merge-as-is bucket has effectively closed, which would matter operationally because merge-as-is is the only bucket that does not impose downstream reviewer work.

None of these three falsification patterns has occurred in the five-tick window. The drip-348 → drip-352 sequence is consistent with the near-absorbing-marginal hypothesis at every measurable cut. The drip-352 tuple (1,4,1,2) is best read as the stochastic mean-reverting partner to the drip-351 (1,7,0,0) tuple, not as a regime change. The 65%-on-merge-after-nits stationary mass is the operative structural fact about the reviewer process at this point in the W17 cycle.

## Closing observation: why this matters for the next ten-tick window

If the near-absorbing-marginal hypothesis holds for the next ten ticks (drip-353 through drip-362), the predicted aggregate verdict count over 80 PRs is `(12, 52, 8, 8)` with binomial SDs of `(3.2, 4.3, 2.7, 2.7)` per bucket. Any aggregate that deviates from this prediction by more than 2σ on any single bucket — for example, fewer than 5 merge-as-is, fewer than 43 merge-after-nits, more than 14 request-changes, or more than 14 needs-discussion in the cumulative ten-tick window — will constitute first-tier evidence that the reviewer-verdict process is structurally different from what the drip-348 → drip-352 window measured. Until then, the family mean is `(0.150, 0.650, 0.100, 0.100)`, the merge-after-nits column is the absorbing-marginal centre, and the (1,4,1,2) drip-352 tuple is just the chain's natural breathing.
