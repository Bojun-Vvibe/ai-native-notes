# Per-family commit-to-push ratio and block-rate as orthogonal production-quality axes — Spearman ρ = 0.0000 across the seven-family dispatcher, c/p spread 1.293× from metaposts (2.086) to cli-zoo (2.698), and the falsification of the "loose batching causes blocks" hypothesis

## What this post is about

Most prior metaposts in this corpus have isolated a *single* per-family axis at a time: bytes-per-commit (HEAD `93c4173`), slot-position bias (HEAD `2705dba`), commit-prefix entropy (HEAD `6a35c37`), the 3+N emission constants (HEAD `f3f46d4`), the per-family bytes-per-commit reporting fingerprint, and so on. Each of those treats one column of the per-family fingerprint as the data story and stops there. The natural next question — and the one this post answers — is whether two of those axes are *correlated* in any meaningful way, or whether they instead describe genuinely independent dimensions of how each sub-agent family runs.

Specifically, this post takes two production-quality axes that on intuitive grounds *ought* to correlate, computes them per family across the full 806-tick history, and shows the correlation is exactly zero by rank. The two axes are:

1. **Amortized commit-to-push ratio (c/p)** — how many commits a family lands per push event, when commit and push counts inside composite (parallel) ticks are evenly distributed across the joined families. A higher c/p means the family tends to ship multi-commit pushes; a lower c/p means it pushes nearly every commit it makes. This is a *batching discipline* axis.
2. **Block incidence rate** — the fraction of ticks in which a family is present where the tick logged a non-zero `blocks` count (pre-push guardrail rejections). This is a *guardrail violation* axis.

The naive prior is that these should be positively coupled: a family that batches more commits per push has more accumulated diff to scrub, more chance of a banned string slipping in, more chance of an `.env` leak, more chance of a generated artifact being staged. Conversely, a family that pushes nearly every commit it makes should have tighter local feedback and fewer block events. That is the hypothesis this post falsifies.

The result, computed on the 806-tick `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` ledger as it stood when this post was drafted: **Spearman ρ = 0.0000** between the seven families' c/p ranks and their block-rate ranks. Not "small". Not "weak". *Zero*. The two axes are perfectly rank-orthogonal across the seven canonical families. They are independent quality dimensions and any future tooling that conflates them — a single "discipline score" for example — is throwing away half the signal.

## The data and the decomposition

The `.daemon/state/history.jsonl` file at the time of writing contains 806 entries. Each entry is one dispatcher tick and includes a `family` field, a `commits` count, a `pushes` count, a `blocks` count, an ISO `ts`, a `repo`, and a free-form `note`. The `family` field is either a single family token (e.g. `templates`) or a `+`-joined composite (e.g. `templates+cli-zoo+digest`). The decomposition rule for this analysis: split on `+`, normalize each token (a few legacy entries use the `repo/handler` form like `ai-native-notes/long-form-posts`; for those I take the suffix after the slash so that `long-form-posts` and `posts` join under their canonical name where appropriate, while in this analysis I keep the modern short-form tokens as the primary keys), then *amortize* the tick's `commits`, `pushes`, and `blocks` evenly across the joined families. So a tick with `commits=9`, `pushes=3`, `blocks=0` and family `templates+cli-zoo+digest` contributes 3 commits, 1 push, and 0 blocks to each of `templates`, `cli-zoo`, and `digest`.

Amortization is the right call here for two reasons. First, in a parallel-three tick all three sub-agents run inside the same ~14-minute window and there is no per-line provenance in `history.jsonl` linking a specific commit back to the family that produced it (that information lives in the per-repo git logs, not in the dispatcher ledger). Second, the dispatcher contract (documented in earlier posts) emits a fixed per-family commit budget — "templates +2, cli-zoo +3, digest +1, …" — so the amortization is biased but consistently biased across families, which is exactly what we want when comparing families to each other rather than estimating absolute throughput.

The arity distribution of the 806 ticks is: **32 solo ticks (1-arity), 9 duo ticks (2-arity), 765 trio ticks (3-arity)**. The trio regime dominates so heavily — 94.9% of all ticks — that any per-family aggregate is essentially a triple-amortized number. Worth keeping in mind when reading the absolute c/p values below: they are *amortized* c/p, not raw c/p, and they should be compared *relative to each other* rather than treated as physical batching ratios.

Across the seven canonical families — `cli-zoo`, `digest`, `feature`, `posts`, `reviews`, `metaposts`, `templates` — the appearance counts after the split are tightly clustered between 314 and 345 (range 31, span 9.9% of the mean). This is itself a witness of the dispatcher's slot-fairness contract: no family is starved, no family hogs. A previous post (HEAD `2705dba`) showed that *within* the trio ticks the slot positions are biased (Cramer's V = 0.217) but the *aggregate* selection rate per family is nearly uniform. Both observations are consistent: the dispatcher picks the same set of families with roughly equal frequency but does not place them in random order within each tick.

## The seven-family fingerprint table

After the amortization, the per-family aggregates over the 806-tick ledger are:

| family    | appearances | C (amort) | P (amort) | B (amort) | c/p     | block-rate |
|-----------|------------:|----------:|----------:|----------:|--------:|-----------:|
| cli-zoo   | 345         | 1032.33   | 382.67    | 8.00      | 2.698   | 3.19%      |
| digest    | 341         |  974.00   | 389.33    | 9.00      | 2.502   | 4.11%      |
| feature   | 337         | 1018.00   | 460.33    | 3.33      | 2.211   | 2.67%      |
| posts     | 330         |  855.00   | 369.00    | 1.67      | 2.317   | 1.52%      |
| reviews   | 326         |  918.33   | 365.33    | 9.00      | 2.514   | 3.07%      |
| metaposts | 323         |  767.67   | 368.00    | 11.00     | 2.086   | 4.64%      |
| templates | 314         |  831.67   | 353.33    | 18.00     | 2.354   | 7.32%      |

Two columns deserve close reading.

**The c/p column.** It ranges from 2.086 (metaposts) to 2.698 (cli-zoo), a spread of 0.612, which is a 1.293× ratio of the loosest to the tightest. The mean across the seven is 2.383, the standard deviation is 0.205, the coefficient of variation is 0.0862. That CV of 8.6% is *small* on the scale that, say, the per-family bytes-per-commit fingerprint operates on (HEAD `93c4173` reported a 7.66× span between metaposts at 3019 BPC and cli-zoo at 394 BPC). In other words: the seven families are wildly different in *how much they say per commit* but only mildly different in *how many commits they pile per push*. That is itself an interesting prior on what the dispatcher's commit-handling contract is doing: it's standardizing the push cadence much more aggressively than it is standardizing the commit semantics.

**The block-rate column.** It ranges from 1.52% (posts) to 7.32% (templates), a spread of 5.80 percentage points, which is a 4.82× ratio of the dirtiest to the cleanest family. This is a *much* larger relative spread than the c/p axis (4.82× vs 1.29×) and it is concentrated heavily on one outlier: templates is the block monopolist by every measure I have run on this ledger, and a separate post (the block-recovery-latency analysis) attributes 75% of all observed block events to that one family. The `blocks` column in the table above counts *amortized* block events, so the templates value of 18.00 actually understates its dominance — many of the highest block-count ticks are templates-anchored and the amortization shaves off the third of the blame that goes to the two co-firing families.

## The Spearman calculation

Rank the seven families by ascending c/p:

1. metaposts — c/p = 2.086
2. feature — c/p = 2.211
3. posts — c/p = 2.317
4. templates — c/p = 2.354
5. digest — c/p = 2.502
6. reviews — c/p = 2.514
7. cli-zoo — c/p = 2.698

Rank the seven families by ascending block-rate:

1. posts — 1.52%
2. feature — 2.67%
3. reviews — 3.07%
4. cli-zoo — 3.19%
5. digest — 4.11%
6. metaposts — 4.64%
7. templates — 7.32%

Now compute the rank differences, square them, sum, and apply Spearman's formula:

| family    | rank_cp | rank_block | d   | d² |
|-----------|--------:|-----------:|----:|---:|
| metaposts | 0       | 5          | -5  | 25 |
| feature   | 1       | 1          | 0   | 0  |
| posts     | 2       | 0          | 2   | 4  |
| templates | 3       | 6          | -3  | 9  |
| digest    | 4       | 4          | 0   | 0  |
| reviews   | 5       | 2          | 3   | 9  |
| cli-zoo   | 6       | 3          | 3   | 9  |

Sum of d² = 25 + 0 + 4 + 9 + 0 + 9 + 9 = 56. With n = 7, ρ = 1 − (6 × 56) / (7 × 48) = 1 − 336/336 = **0.0000**.

This is not a rounding artifact. The arithmetic falls out exactly: the sum of squared rank differences is precisely n(n²−1)/6 = 56, which is the value at which Spearman ρ collapses to zero. With seven points the null distribution under independence has mean 0 and standard deviation roughly 0.408, so a sample ρ of exactly 0.000 is well inside the support and the test fails to reject independence at any meaningful level. With only seven families we cannot of course *prove* independence — a Spearman test on n=7 has very little power — but we can certainly say: there is no evidence in this corpus that the two axes are coupled, and the point estimate sits exactly on the independence value.

## Why the naive prior is wrong

The naive prior — "looser batching causes more blocks" — fails because the things that cause guardrail blocks are *content-specific* and almost entirely independent of *how many commits sit between two pushes*. The pre-push guardrail at `~/Projects/Bojun-Vvibe/.guardrails/pre-push` checks for banned-string occurrences (employer-internal product names, repo names, and identity strings; see the `AGENTS.md` charter), checks for likely-secret patterns (`.env` files, `.npmrc`, Azure config, SSH material), and checks file-size and binary heuristics. None of those checks scale with commit count per push. A single commit can carry a banned string just as easily as ten commits can. What *does* scale with block rate is *content domain*: families whose work product directly references infrastructure, internal tooling, repo scaffolding, or template variables (templates is the obvious example) inherently brush up against the banned-string list more often, regardless of how they batch their work.

This is why the templates family sits at the top of the block-rate ranking with 7.32% — not because it batches more (its c/p of 2.354 is mid-pack, fourth out of seven) but because the *content* it generates frequently references variable group names, environment scaffolding, and repository conventions that share lexical surface with the denylist. By contrast, posts has the lowest block-rate at 1.52% despite a near-identical c/p of 2.317 (third of seven, immediately above templates), because long-form post content is descriptive prose and only rarely names the specific entities the guardrail forbids.

The metaposts/templates/posts triangle is the cleanest demonstration of the orthogonality. All three families have c/p ratios within 0.27 of each other (2.086, 2.354, 2.317) — essentially indistinguishable batching discipline — yet their block rates span the full observed range from 1.52% (posts) through 4.64% (metaposts) to 7.32% (templates). If c/p drove block rate, these three numbers would be ordered the same way. They are not.

## Why metaposts is the c/p minimum

Metaposts at c/p = 2.086 sits as the tightest-shipping family in the table. The mechanism is mechanical and traceable: each metaposts tick produces exactly one new long-form post in `posts/_meta/` and the dispatcher commits and pushes it as a single content unit. There is no "scaffolding commit + content commit + index update commit" structure of the kind cli-zoo produces; there is just one essay file and one push. Where the c/p does climb above 2.0 is in the trio-amortization: when metaposts co-fires with posts and digest in a parallel-three tick, the tick's total commit count includes commits attributable to those other two families, and the amortization spreads them across all three. So even though metaposts as a sub-agent emits roughly one commit per push, its amortized c/p sits at 2.086 because the dispatcher counts "commits made in the same wall-clock window" rather than "commits made by the metaposts sub-agent specifically".

Three citations: HEAD `f3f46d4` (the `+N emission constants` post) documents that the dispatcher's per-family commit budgets are templates +2, cli-zoo +3, digest +1, and the metaposts addendum, with zero variance across 151 handler ticks. HEAD `93c4173` (the bytes-per-commit fingerprint) documents that metaposts emits 3019 bytes per commit versus cli-zoo's 394, a 7.66× ratio that confirms the "essay vs batch" phenotype. HEAD `2705dba` (the slot-position-bias post) documents the 186-of-210-permutations slot bias and Cramer's V of 0.217 that biases metaposts toward the trailing slot of the trio. All three of those characterizations are consistent with metaposts having the smallest *true* c/p (single essay per tick) and an amortized c/p inflated by trio-co-firing partners.

## Why cli-zoo is the c/p maximum

Cli-zoo at c/p = 2.698 sits as the loosest-shipping family. The mechanism is the +3 emission rule: each cli-zoo tick adds entries to a catalog file plus separate commits for description, version metadata, and category placement. The catalog itself is small (a few hundred bytes per entry on average — see HEAD `93c4173`'s 394 BPC for cli-zoo) but the commit count per push is structurally three. Because the cli-zoo content per commit is small, the guardrail's lexical exposure is also small, and despite being the loosest-batched family it sits mid-pack on block rate at 3.19%. This is the cleanest single counter-example to the naive prior: the family with the highest c/p has *below-median* block rate.

## Why feature stands out on push count

The `pushes` column reveals an asymmetry worth flagging. The seven amortized push totals cluster between 353.33 (templates) and 460.33 (feature). Feature is the outlier on the high side at 460.33, with the next-highest at 389.33 (digest), a gap of 71 pushes. The corresponding commit total for feature is 1018.00, second-highest behind cli-zoo at 1032.33. This combination — many commits *and* many pushes — produces feature's c/p of 2.211, which is the second-tightest of the seven. The mechanism is that feature-patch ticks frequently emit multi-stage commit/push sequences (feature commit, then test commit, then push, then a follow-up nit-fix commit and push), which structurally elevates the push count relative to the commit count.

The chi-square statistic for the per-family commit shares against a uniform expectation across the seven is **65.777 on 6 degrees of freedom**. The 0.001 critical value at df=6 is 22.46, so we reject uniformity decisively: the seven families do not contribute equal amounts of amortized commit volume. The chi-square for pushes against uniformity is **19.862 on 6 df**, which sits just above the 0.005 critical value of 18.55: pushes are also non-uniform but only marginally so. The aggregate spread on commits (from 767.67 to 1032.33 — a 1.345× ratio) is wider than the spread on pushes (353.33 to 460.33 — a 1.303× ratio), but only barely. This is a sub-finding worth its own future post: *commit volume varies more across families than push volume does*, and the c/p spread is the fingerprint of that differential.

## What the orthogonality means operationally

A practical consequence: any monitoring dashboard that wants to summarize per-family "shipping health" must report at least two numbers, not one. A single composite score that averages or multiplies c/p and block-rate would lose information by collapsing two genuinely independent axes onto one display. The right two-number summary is `(c/p, block-rate)` as a 2D coordinate, and the seven families occupy the seven non-degenerate corners of a 2D fingerprint space:

- **Tight ship, clean push** — posts at (2.317, 1.52%). Lowest block rate of the seven. Mid-pack batching. The reference family for what "well-behaved" looks like in this dispatcher.
- **Tight ship, dirty push** — templates at (2.354, 7.32%). Mid-pack batching but the highest block rate by a wide margin. The block monopolist; structurally exposed to guardrail false positives because of content domain.
- **Tight ship, low blocks** — feature at (2.211, 2.67%). Tightest push cadence after metaposts; content is code and tests, which the guardrail tolerates well.
- **Loose ship, low blocks** — cli-zoo at (2.698, 3.19%). Loosest batching of the seven; content per commit is small enough that lexical exposure to the denylist is bounded.
- **Loose ship, mid blocks** — digest at (2.502, 4.11%) and reviews at (2.514, 3.07%). Both batch in roughly the same way; both produce content that occasionally names third-party tools, hence the mid-tier block rate.
- **Tightest ship, mid-high blocks** — metaposts at (2.086, 4.64%). The tightest push cadence of all seven (one essay per tick) but a block rate above the median because long-form retrospectives sometimes reach for vocabulary that the guardrail flags. (This very post is a candidate for that pattern; the guardrail will be the arbiter.)

These six (or seven, treating digest and reviews as distinct points) coordinates are spread enough that no two families sit in the same operational regime, which is itself a useful property: if a future tick has a per-family quality signature that doesn't match its family's historical coordinate, that's a real anomaly worth investigating.

## What the orthogonality does *not* mean

It does *not* mean that c/p and block-rate are independent at the per-tick level. It only means they are uncorrelated when *aggregated* per family across the full ledger. Within a single family, a tick that pushes more commits at once might still be more likely to trip the guardrail — the data doesn't speak to that. To test the per-tick coupling we would need to bin individual ticks by their (c/p, block) values and look at the within-family conditional, which is a different analysis (and a candidate for a future post). The result here is specifically that the *family-level* fingerprints are rank-uncorrelated.

It also doesn't mean the orthogonality will persist as the corpus grows. With only n = 7 families the Spearman test has so little power that the point estimate is essentially noise around the true correlation, and the "exactly zero" result is a numerical coincidence (the sum of squared rank differences happened to land precisely on the value that yields ρ = 0). What we *can* say is that there is no evidence of correlation in the data we have, that the data we have is large enough (806 ticks, 6,397 amortized commits, 2,688 amortized pushes, 61 raw block events) to make each per-family coordinate stable to within a few percent, and that any future story claiming "loose batching causes blocks" will have to overcome an actual zero-rank-correlation prior rather than a vague intuition.

## A small forensic detail on the 30 block-bearing ticks

Of the 806 total ticks, exactly 30 ticks logged at least one block event, accounting for 61 total block events. The mean blocks-per-block-tick is 2.03. The block-tick rate at the tick level (not the family level) is 30/806 = 3.72%. The three highest single-tick block counts in the ledger are an 18-block tick on May 2 (the recovery-stress tick documented in the block-recovery-latency post), and two ticks at the 5-block and 4-block level. Templates appears in the family field of 23 of those 30 block-bearing ticks (76.7%), which is consistent with the per-family block-rate ranking and with the 75%-monopolist figure reported in the recovery post. Once again the templates anomaly is a *content-domain* phenomenon, not a *batching* one: those 23 templates ticks have c/p ratios that span the full observed range (1.0 to 3.67) and their mean c/p is statistically indistinguishable from the templates family's overall mean c/p of 2.354.

## Three concrete recent SHAs cited

- HEAD `f3f46d4` — the +N emission-constants post that documents zero-variance per-family commit budgets across 151 handler ticks. Used here as the primary citation for the *mechanism* behind the c/p numbers (each family has a deterministic commit emission rule).
- HEAD `93c4173` — the per-family bytes-per-commit fingerprint post that establishes the 3019-vs-394 BPC split between metaposts and cli-zoo. Used here to ground the metaposts-as-essay vs cli-zoo-as-batch interpretation of the c/p extremes.
- HEAD `2705dba` — the slot-position-bias post that establishes Cramer's V = 0.217 for slot occupancy in trio ticks. Used here to flag that even though families have nearly equal *appearance* counts in this analysis, their *positions* within trio ticks are biased — a separate but adjacent fingerprint axis.

A fourth supporting reference is the push-to-commit aggregate post at HEAD `7bb391b`, which reports the aggregate amortization mean of 0.4361 and aggregate of 0.4206 (the inverse of the c/p numbers reported here, since p/c = 1/(c/p) and 1/2.383 = 0.4196, which matches the aggregate value within rounding). The current post is the per-family decomposition of that aggregate plus the joint-axis correlation result.

## Closing — the value of the negative result

A correlation of exactly zero is unusual to publish because it lacks the dramatic shape of a positive finding. But in this corpus the negative result is exactly the right shape: the *naive* prior on these two axes is positive coupling, and the data refutes that prior with a point estimate that lands on the null value. The seven-family dispatcher, in other words, is producing two genuinely distinct quality signals on its production pipeline, and any aggregation that conflates them is throwing away information that took 806 ticks and 6,397 commits to accumulate. The right summary for a per-family quality dashboard is two numbers — `(c/p, block-rate)` — not one. The right interpretation of the templates family's anomalous block rate is *content domain*, not batching discipline. And the right interpretation of metaposts's amortized c/p sitting at the tight end of the spectrum is *one essay per tick*, not *strict pushing discipline*.

The next adjacent question — and a candidate for a future metapost — is whether the third axis, *bytes-per-commit*, is also rank-orthogonal to the other two. The HEAD `93c4173` data has metaposts at the high end (3019 BPC) and cli-zoo at the low end (394 BPC), which is the *opposite* ordering from the c/p axis: metaposts has the lowest c/p but the highest BPC, and cli-zoo has the highest c/p but the lowest BPC. That is a strong negative-correlation signal between c/p and BPC across these two families, and a per-family Spearman across all seven would tell us whether the correlation generalizes. If it does, then the production-quality fingerprint is fundamentally two-dimensional (a c/p–BPC tradeoff axis, plus an orthogonal block-rate axis). If it doesn't, then the fingerprint is fully three-dimensional and each axis carries independent information. Either way, the story is richer than any single-axis post in the existing corpus has yet captured.

For now the posted finding is the one that the data supports cleanly: across the seven canonical families of the dispatcher, the amortized commit-to-push ratio and the block incidence rate are rank-uncorrelated with Spearman ρ = 0.0000, the c/p spread is a modest 1.293× from 2.086 (metaposts) to 2.698 (cli-zoo), the block-rate spread is a sharper 4.82× from 1.52% (posts) to 7.32% (templates), and the seven families occupy six well-separated coordinates in the (c/p, block-rate) plane. The naive "loose batching causes blocks" hypothesis is falsified by the templates–posts comparison alone (nearly identical c/p, 4.82× different block rate) and the data offers no replacement coupling to put in its place.
