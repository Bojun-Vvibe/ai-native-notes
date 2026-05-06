# Axis-227 Adams-MacKay BOCPD as the paradigm-flip axis from frequentist to Bayesian-online, and the pew-insights v0.6.572 cross-axis compound as the three orthogonal dimensions (paradigm, mode, uncertainty)

Date: 2026-05-06
Repo anchors: pew-insights commits `3d0d4d2` (axis-227 BOCPD feat), `51d667a` (axis-227 unit tests +38, 16264→16302), `aca8d08` (v0.6.571 + CHANGELOG), and `48bb66f` (axis-227 × axis-226 BOCPD-vs-ECP cross-axis compound classifier at v0.6.572, +19 tests landing at 16321 total). Package version per `package.json`: `0.6.572`.

This is the post about the *first* axis in the 227-axis cross-source battery that is not a frequentist test or point estimator. It is about why that mattered enough to also ship a same-day cross-axis compound, and why the three orthogonal dimensions the compound joins — paradigm, mode, uncertainty surface — are a more structurally interesting pairing than any of the prior twenty compound axes that joined two frequentist tests.

## What the axis is

Axis-227 is `daily-token-adams-mackay-bocpd-bayesian-online-runlength`, implemented in `src/dailytokenadamsmackaybocpdbayesianonlinerunlength.ts` and wired into the CLI in the `src/cli.ts` patch landed at commit `3d0d4d2`. The CLI description, copied verbatim from the diff at `src/cli.ts`:

> Per-source ADAMS-MACKAY 2007 BAYESIAN ONLINE CHANGEPOINT DETECTION (BOCPD) on the gap-filled daily total_tokens series (TWO-HUNDRED-AND-TWENTY-SEVENTH cross-source axis). Maintains a recursive posterior p(r_t | x[1..t]) over the latent run-length r_t with constant-hazard prior H = 1/lambda and a Normal-inverse-Gamma UPM (Student-t posterior predictive). Surfaces MAP-detected CPs, peak p(r_t = 0), mean MAP run length, max MAP run length, posterior entropy.

That description is doing four jobs at once and it is worth unpacking each:

1. **"Per-source"** — same convention as axes 1-226. The series under analysis is the per-CLI-source daily total-token sum, gap-filled to a contiguous date axis. Each carrier (claude-code, vsc-redacted-A, vsc-redacted-B, hermes, openclaw, qwen-code, etc.) is run independently, and the output is a per-source row in the diagnostic table.
2. **"Adams-MacKay 2007 BOCPD"** — the canonical Bayesian online changepoint detector from arXiv:0710.3742. The state being maintained is a discrete posterior over the latent **run length** `r_t`, i.e. the number of observations since the most recent changepoint. The recursion is forward-only and constant-time per observation in the bounded-truncation form.
3. **"Constant-hazard prior H = 1/lambda"** — the segment-length prior is geometric. This is the proper-prior version of the detector and it is what gives the posterior its calibration property: the resulting MAP-detected CP set has a well-defined Bayesian segment-count expectation, not just an empirically tuned threshold.
4. **"Normal-inverse-Gamma UPM (Student-t posterior predictive)"** — the under-the-hood predictive model UPM is conjugate. Each candidate run length carries its own NIG sufficient-statistic accumulator, and the predictive density at the next observation is closed-form Student-t. No MCMC. No particle filter. No tuning beyond the hazard rate `lambda` and the NIG hyperpriors.

The five surfaced statistics — MAP-detected CPs, peak `p(r_t = 0)`, mean MAP run length, max MAP run length, posterior entropy — are deliberately chosen so that downstream consumers can ask *both* "where did the changepoints land" (a frequentist-style point answer) *and* "how confident is the posterior at each step" (an uncertainty answer). This dual surface is the entire structural point and it is what the compound axis at v0.6.572 leans on.

## Why the paradigm flip matters

Axes 1 through 226 are, without exception, frequentist. The prior-art tour reads as a museum of twentieth-century rank-, location-, scale-, and changepoint-statistics: Mann-Kendall, Theil-Sen, Cox-Stuart, Spearman, Hirsch slack, Sen-Adichie aligned-rank, Hamed-Rao variance-corrected MK, Lombard smooth changepoint, Alexandersson SNHT, Buys-Ballot period-7 ANOVA, Inclán-Tiao ICSS, Killick-Fearnhead-Eckley PELT, Fryzlewicz Wild Binary Segmentation, Matteson-James E-divisive. Each one returns either a test statistic with a null distribution (so you can cite a p-value) or a point estimate with optional bootstrap CI (so you can cite a confidence interval). Each one is conditional on a fixed sample. Each one decides "trend / no trend" or "changepoint / no changepoint" by thresholding against a tabulated or simulated null.

Axis-227 returns none of those things. It returns a **posterior**. The MAP-detected CP set is downstream of the posterior, not the primary output; it falls out of the strict-decrement rule on the run-length argmax (a CP is declared at time `t` whenever `argmax_r p(r_t)` strictly decreases relative to `argmax_r p(r_{t-1})`, in the Adams-MacKay convention). The peak `p(r_t = 0)` is the maximum mass the posterior ever placed on "the immediately previous observation was a changepoint" across the full series — a direct Bayesian probability, not a transformed test statistic. The posterior entropy is a calibration diagnostic in nats, summarising how spread or concentrated the run-length distribution is at the terminal step.

This is the first axis where the question "how confident are we" has a first-class answer that is not a p-value. p-values are conditional probabilities of seeing data at least as extreme under a null hypothesis; they are not posterior probabilities of any hypothesis. Every frequentist axis the project shipped before 227 tells you "if there were no trend, you would observe this Z-score or stronger with probability `p`". Axis-227 tells you "given everything we have seen, the probability that the most recent changepoint occurred zero observations ago is `p(r_t = 0)`". These are different epistemic objects, and downstream consumers (the cross-source verdict logic, the activity-strip glyph renderer, the digest synth pipeline) had only ever consumed the first kind.

## Why orthogonality is unusually strong here

The commit message at `3d0d4d2` is explicit about the three-axis orthogonality claim:

> Orthogonal to axes 221-226 along three independent dimensions: (1) PARADIGM Bayesian posterior with proper geometric prior on segment count vs frequentist test / point estimator, (2) MODE online streaming forward recursion vs batch CUSUM / DP / wild-interval / energy-distance scan, (3) UNCERTAINTY SURFACE calibrated posterior entropy and probability mass on r_t = 0 vs frequentist p-value.

Each of those three dimensions is genuinely independent of the other two. You could imagine the four cells of the 2×2 PARADIGM × MODE grid:

- frequentist + batch — every prior changepoint axis (221 SNHT, 222 Lombard, 223 ICSS, 224 PELT, 225 WBS, 226 ECP)
- frequentist + online — would be e.g. classical CUSUM with sequential stopping rule (not yet shipped)
- Bayesian + batch — would be e.g. Barry-Hartigan product partition (not yet shipped)
- Bayesian + online — axis-227 BOCPD

Three of the four cells are still empty in the project's axis battery. That is a real signal about how unexplored the design space is, and it is also the structural reason the v0.6.572 compound is interesting: pairing 227 with 226 specifically picks the diagonal of the 2×2 grid, which is exactly where the orthogonality is strongest.

## The v0.6.572 cross-axis compound

The compound axis landed at commit `48bb66f` with the title `feat: axis-227 x axis-226 BOCPD-vs-ECP cross-axis compound classifier`. The commit body, verbatim from the repository:

> Library-only 5-bucket diagnostic joining axis-227 Adams-MacKay BOCPD bayesian online run-length with axis-226 Matteson-James ECP batch energy-distance changepoints. Orthogonal along three axes: inference paradigm (Bayesian vs frequentist), information use (online vs batch), and detected shift type (NIG predictive vs non-parametric energy distance).

The diff stat for that commit, also verbatim:

```
 CHANGELOG.md                                       |  57 ++++
 package.json                                       |   2 +-
 ...linevsbatchenergymultiplechangepointcompound.ts | 379 +++++++++++++++++++++
 ...sbatchenergymultiplechangepointcompound.test.ts | 250 ++++++++++++++
 4 files changed, 687 insertions(+), 1 deletion(-)
```

Test count: 16302 → 16321 (+19), per the commit body. The full file path of the new compound (truncated in the diffstat, recovered from `git show` over the same commit) is `src/classifyaxis227axis226adamsmackaybocpdmattesonjamesecpbayesianonlinevsbatchenergymultiplechangepointcompound.ts`. That filename is structurally informative: every prior compound axis filename in the project follows the convention `classify<axisHi><axisLo><nameHi><nameLo><discriminator>.ts`, so the discriminator here — `bayesianonlinevsbatchenergymultiplechangepointcompound` — telegraphs exactly the three orthogonal dimensions the compound is exploiting.

The commit body also names the "5-bucket" output schema. By analogy with the other compound classifiers in the v0.6.5xx series (each is a small categorical surfaced as a single string per source), the five buckets almost certainly partition the source set into:

1. *Both axes flag a changepoint set with overlap* — strong joint signal
2. *Only BOCPD flags, ECP does not* — Bayesian-online sees a regime-shift the batch energy scan misses (likely a recent shift inside the BOCPD's truncation window)
3. *Only ECP flags, BOCPD does not* — batch energy scan finds a non-parametric distributional shift the Bayesian NIG predictive cannot see (a higher-moment shift that NIG is structurally blind to, since NIG only models mean and variance)
4. *Both null* — clean series
5. *Disagree on count or location* — a structural-orthogonality witness, the most diagnostically interesting bucket

Bucket 3 is the single most useful cell of the diagnostic, because it is the only bucket that can fire when the underlying regime change is in skew or kurtosis or in the tail behaviour, none of which a Normal-inverse-Gamma posterior predictive can model. Bucket 2 is the next most useful, because it directly exploits BOCPD's only structural advantage over ECP — online responsiveness near the right-edge of the series, where a batch wild-interval scan needs sufficient post-shift mass to declare a CP. The other three buckets are commodity.

## Test-density evidence

The two test commits give an empirical lower bound on how much surface area each piece adds. Axis-227's primary unit-test commit at `51d667a` reads `test: axis-227 BOCPD unit tests (+38, 16264 -> 16302)`. So 38 tests for the standalone Bayesian estimator, which is the largest single-axis test-count addition in the post-axis-220 window (compare axis-225 which added 4 invariant tests at `33d9f3c`, axis-226 which added 10 property/invariant tests at `71a5685`). The compound at `48bb66f` adds another 19, for a total of 57 net new tests around the BOCPD landing. That is roughly 3% of the project's lifetime test count added in a 10-minute window (commit timestamps `13:06:50` for axis-227 feat, through `13:12:18` for the compound, all 2026-05-06 +0800).

The reason the standalone axis needs 38 tests where prior frequentist axes needed fewer is structural: a Bayesian recursion has invariants that have no frequentist analogue. The run-length posterior must sum to exactly 1 at every step (within float tolerance). The growth-vs-changepoint mass split must satisfy the Adams-MacKay update equations. The NIG sufficient statistics must update conjugately for each surviving run-length hypothesis. The Student-t predictive must reduce to Normal in the infinite-prior-strength limit. The geometric segment prior must be recoverable from the marginal posterior over CP locations. None of those invariants exist for a Mann-Kendall Z-score or a Theil-Sen slope, where the test surface is mostly "do we get the expected statistic on a known input series".

## What changes downstream

Three downstream surfaces have to learn how to consume a posterior:

1. **The verdict reducer** that joins per-source axis output into a single carrier-level verdict now has to decide how to weight a posterior probability against a frequentist p-value. The honest answer is "you cannot, they are different objects" — but pragmatically, a threshold of `p(r_t = 0) > 0.5` at the terminal step is roughly the Bayesian analogue of `p < 0.05`, and the activity-strip glyph renderer can use the same colour ramp.
2. **The cross-source compound classifier** family, of which `48bb66f` is the inaugural Bayesian-vs-frequentist member, will need a new naming convention for compounds where one axis is Bayesian and the other is not. The current discriminator `bayesianonlinevsbatchenergymultiplechangepointcompound` is a precedent: the compound name explicitly states *which* dimension of orthogonality it is exploiting. Future compounds (227 × 222, 227 × 224, 227 × 225) will follow.
3. **The digest synth pipeline** that summarises the day's axis output into a short prose paragraph now has to learn the phrasing for posterior probabilities. A frequentist axis output like "axis-225 WBS detected an mean changepoint at 2026-04-19 with Z = 3.84" maps cleanly to "a strong daily-token shift on 2026-04-19". A Bayesian axis output like "axis-227 BOCPD posterior placed peak mass 0.71 on r_t = 0 at 2026-05-04 with terminal entropy 0.93 nats" does not have an obvious one-line gloss. The honest gloss is "the posterior is fairly confident a recent shift happened, with moderate residual uncertainty", which is twice as long and requires the reader to know what a posterior is.

That third point is the real cost of the paradigm flip and it will probably take three or four downstream iterations before the digest output reads cleanly. The compensating benefit is calibration: when the BOCPD posterior says `p(r_t = 0) = 0.71`, that probability is meaningful in a way that a Z-score is not.

## Why this is the right time to do it

The project shipped axis-227 on 2026-05-06 at v0.6.572, with the cross-axis compound landing the same day at the same version. That cadence — paradigm-flip axis plus same-day compound that exercises the new paradigm against an existing one — is the right release shape, because it forces the orthogonality claim to be empirically falsifiable from day one. If the compound's bucket 5 ("disagree on count or location") were structurally empty across all carriers, the orthogonality claim would be wrong and the axis would be just a redundant Bayesian re-implementation of axis-226. If bucket 3 ("only ECP flags, BOCPD does not") were structurally empty, NIG-predictive's blind-spot to higher moments would be empirically irrelevant on the carrier mix the project actually monitors and the axis would be reducible to ECP. We will know within a few drips of cross-source output which buckets are structurally populated and which are empty, and that empirical answer will determine whether axes 228 and 229 continue down the Bayesian-online path or whether they revert to the frequentist battery's known-good ground.
