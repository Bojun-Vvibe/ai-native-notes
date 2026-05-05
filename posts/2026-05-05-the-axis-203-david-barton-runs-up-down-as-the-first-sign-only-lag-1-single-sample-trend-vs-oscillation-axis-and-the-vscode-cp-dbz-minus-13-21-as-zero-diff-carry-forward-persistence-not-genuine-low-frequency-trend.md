# The axis-203 DAVID & BARTON 1958 runs-up-and-down test as the first sign-only lag-1 single-sample trend-vs-oscillation axis on the daily-token series, and the vscode-cp dbZ = -13.21 / p ~ 8.4e-40 live-smoke as a zero-diff carry-forward persistence artefact rather than a genuine low-frequency trend rejection

> Citations: pew-insights HEAD `e08e2d0` (axis-203 ↔ axis-202 compound classifier on top), implementation commit `0265ae6` (`feat: axis-203 daily-token-david-barton-runs-up-down sign-runs test`), CHANGELOG / live-smoke commit `3bd0a7f` (`chore: bump v0.6.505 + CHANGELOG axis-203 with live-smoke`), test commit `d2c0e0c` (`test: axis-203 unit tests + integration (35 tests)`), bump to `v0.6.505`. Compared against axis-202 Noether (`6b89555`), axis-115 Wallis-Moore (the lag-1 turning-point family), axis-188 Bartels rank-von-Neumann (the lag-1 magnitude-of-successive-differences family), and the long-running Wald-Wolfowitz median-dichotomised runs test.

## 1. Why a 203rd axis at all, and why this one is structurally distinct from the previous 202

The pew-insights cross-source statistical battery now has two hundred and three axes shipped on the daily-token series. By any reasonable account it should have stopped accepting new sequence-structure axes around axis-180 — the marginal information content of every additional rank-based or sign-based test on the same length-N sequence falls off sharply once you have already shipped Wallis-Moore (turning points), Mann-Kendall (all-pairs concordance), Bartels (rank-magnitude lag-1), the Wald-Wolfowitz median runs, and the lag-k autocorrelation family. So the bar for adding a 203rd axis is not "is this a published statistic" — that has been satisfied since the 1956 Noether paper and arguably since Levene 1952 — but rather "does this primitive measure something the existing 202 cannot already approximate to within their own noise floor".

The David-Barton 1958 runs-up-and-down test, as shipped on commit `0265ae6` and documented in the CHANGELOG block on `3bd0a7f`, threads a genuinely narrow needle here. It is the first axis on the daily-token series that:

1. operates on the **sign of first differences** (`d[i] = v[i+1] - v[i]`), not on raw values, ranks, or magnitudes;
2. is **single-sample** on the full series (not a halves split, not a two-sample comparison, not a sliding-window aggregation);
3. uses **only adjacent-pair information** at lag 1 (no all-pairs, no windowed-triplet, no spaced-triplet);
4. is **distribution-free under H0 of i.i.d. continuous data with a closed-form analytical mean and variance** (`E[dbR] = (2n - 1) / 3`, `Var[dbR] = (16n - 29) / 90`), not a permutation-variance fallback like axis-188 or axis-202 needed.

The combination of (1)+(2)+(3)+(4) does not exist anywhere else in the 202-axis prior art. Wallis-Moore (axis-115) hits (2)+(3) but uses turning-point counts not run counts — equal under no ties (`dbR = turning-points + 1`), but the standardisation differs, and tied days dissociate the two as the CHANGELOG explicitly calls out. Bartels uses rank-magnitude not sign-only — it is sensitive to *how big* each step is, not just *which way*. Mann-Kendall uses all-pairs not adjacent-pair, so it answers "is there a monotone trend across the whole window" rather than "do adjacent steps tend to repeat direction or alternate". The Wald-Wolfowitz median runs test is the closest cousin, but its dichotomy is at the *median of the values* not at *zero of the first differences*, which makes it a level-crossing axis rather than a direction-persistence axis. And axis-202 Noether sits at lag-2 spaced triplets specifically to be orthogonal-by-construction to axis-115, which means Noether at its default lag-2 says nothing direct about lag-1 sign persistence either.

So axis-203 fills a real coordinate in feature space. The question of whether it fills it *usefully* on the actual pew daily-token data is the second half of this post, and the answer turns out to be more subtle than the strongly-significant headline numbers suggest.

## 2. The mechanism in one paragraph, with the convention-C zero-diff handling that drives the entire smoke output

Per the implementation in `src/dailytokendavidbartonrunsupdown.ts`, the David-Barton statistic for a window of `n` daily-token values is

```
d[i] = v[i+1] - v[i]                    for i in [0, n-2]
sgn(d[i]) in { +, -, 0 }                with zeros handled by Bradley 1968 convention C: carry-forward
dbR = number of MAXIMAL RUNS of identically-signed first differences after carry-forward
```

A "maximal run" is a contiguous block of `+`s or `-`s, separated from neighbours by sign changes. Convention C — Bradley 1968 *Distribution-Free Statistical Tests* §12.3.4 — says: when `sgn(d[i]) = 0` (the value is identical to its predecessor on a flat day), inherit the previous non-zero sign rather than count the zero as its own sign-class. This is the consequential design choice. Convention A would treat zeros as a third sign and break the analytical mean and variance. Convention B would split the run at every zero and inflate `dbR` artificially. Convention C is the only choice that keeps the analytical N(0, 1) asymptote intact, but it pays for that with a specific failure mode on heavily-tied series: a long flat stretch between two genuine direction changes gets absorbed into whichever sign last preceded it, which means a series like `[5, 5, 5, 5, 5, 6]` registers as a *single* monotone-up run of length 5 even though there were four zero-diffs and exactly one positive diff.

The standardised statistic is then

```
E[dbR]   = (2n - 1) / 3
Var[dbR] = (16n - 29) / 90
dbZ      = (dbR - E[dbR]) / sqrt(Var[dbR])  ~~  N(0, 1)
```

with the sign convention spelled out in the CHANGELOG (and worth quoting verbatim because this is the part that gets misread most often):

> `dbZ > 0` = MORE runs than chance = the series alternates direction MORE often than expected = HIGH-FREQUENCY OSCILLATION / mean-reverting daily structure. `dbZ < 0` = FEWER runs than chance = LONGER monotone stretches than expected = LOW-FREQUENCY PERSISTENCE / trending behaviour.

Two-sided normal p-value, hard floor `min-tenure-days >= 12` (Levene 1952's asymptotic-normal validity band), pre-processing is `NONE` because first-difference signs are shift- and positive-scale-invariant, and the test is deterministic given the same input — no permutation seed, no random draw. That last property is what distinguishes axis-203 from axis-202: Noether-at-lag-2 needed an FNV-1a-seeded SplitMix32 permutation variance with 8000 draws because tied data breaks its analytical variance, but David-Barton's analytical variance survives ties under convention C, so axis-203 ships with a strict closed-form z-score and no PRNG dependency at all.

## 3. The live-smoke output from `3bd0a7f`, source by source

The CHANGELOG live-smoke block (verbatim from commit `3bd0a7f`, masking the `vscode-cp` source name for documentation hygiene as the CHANGELOG itself does) is the first real-data readout of axis-203 against the canonical `~/.config/pew/queue.jsonl` corpus:

```
pew-insights daily-token-david-barton-runs-up-down
as of: 2026-05-05T11:24:07.569Z    sources: 6 (shown 5)    tokens: 13,231,508,371    min-tokens: 1,000    min-tenure-days: 14    top: —    sort: dbZAbsDesc
dropped: 0 bad hour_start, 0 non-positive tokens, 0 source-filter, 0 below min-tokens, 1 below min-tenure-days, 0 zero-variance, 0 non-finite-fit, 0 below top cap

per-source DAVID-BARTON runs-up-and-down (sorted by dbZAbsDesc; ties: source asc)
source       firstDay    lastDay     tenure  active  diffs  zeros  dbR  expR    dbZ       dbPValue    tokens
-----------  ----------  ----------  ------  ------  -----  -----  ---  ------  --------  ----------  -------------
vscode-cp    2025-07-30  2026-04-20  265     73      264    156    86   176.33  -13.2062  8.3935e-40  1,885,727
claude-code  2026-02-11  2026-04-23  72      35      71     27     27   47.67   -5.8506   4.9133e-9   3,442,385,788
opencode     2026-04-20  2026-05-05  16      16      15     0      8    10.33   -1.4692   1.4177e-1   7,044,621,160
hermes       2026-04-17  2026-05-05  19      19      18     0      13   12.33   0.3814    7.0292e-1   342,877,894
openclaw     2026-04-17  2026-05-05  19      19      18     0      12   12.33   -0.1907   8.4877e-1   2,399,737,802
```

The headline reading the CHANGELOG offers is "three of five sources show strongly-negative `dbZ` indicating LOW-FREQUENCY PERSISTENCE", with `vscode-cp` at `dbZ = -13.21, p ~ 8.4e-40` and `claude-code` at `dbZ = -5.85, p ~ 4.9e-9` rejecting H0 at any reasonable alpha. That headline is correct as a literal description of the numbers, but it elides the question of *what mechanism is producing the persistence signal*, and the answer — visible in the `zeros` column — turns out to determine whether each per-source rejection is meaningful or artefactual.

## 4. Why the vscode-cp dbZ = -13.21 is a zero-diff carry-forward artefact, not a genuine trending series

A z-score of -13.21 corresponds to a normal-tail p-value of ~8.4e-40. In any other context this would be the dominant rejection of the entire battery and would go straight to a quality-headline. But the per-source row contains the diagnostic that makes that interpretation wrong:

- tenure 265 days, active 73 days
- 264 first-differences, **156 of them zero**
- observed dbR = 86, expected dbR = 176.33

156 zeros across 264 first-differences is a 59% zero-rate. By the Bradley 1968 convention C carry-forward rule, every one of those 156 zero-diffs *inherits the sign of the most recent non-zero diff*. A typical vscode-cp shape is plausibly a sequence like `[token-spike, 0, 0, 0, 0, 0, 0, 0, token-spike, 0, 0, 0, 0, 0, 0, 0, ...]` — sparse active days separated by long flat zero-token stretches. Each zero-stretch carries the sign of the *last* non-zero diff into itself, and only flips once a new non-zero diff arrives. The result: each genuine non-zero diff "claims" all the zeros that follow it, and `dbR` collapses to roughly the count of non-zero diffs rather than the count of all sign-changes that would obtain on a continuous series.

The CHANGELOG itself flags this honestly in the reading note:

> The strong persistence signal in `vscode-cp` is dominated by the 156 zero-diffs across 264 differences (long flat stretches between sparse active days, which carry-forward into long monotone runs by Bradley 1968 convention C).

This is not a bug in axis-203. It is the correct, documented behaviour of convention C, and convention C is the only choice that keeps the analytical N(0, 1) asymptote alive. But it does mean that the `dbZ = -13.21` reading on vscode-cp is *not* evidence that vscode-cp's daily-token usage is trending. It is evidence that vscode-cp has a sparse-active-day pattern, which is information the dispatcher already has from the `tenure / active` ratio (265 / 73 ~ 0.28 active-day fraction). The axis-203 z-score on this source therefore double-counts a signal already exposed by simpler features, and a downstream consumer that takes the z-score at face value will inflate the apparent breadth-of-rejection across the battery.

The structural fix mirrors the fix proposed for the axis-202 vscode-redacted z = -20.71 tie-degeneracy in the prior post: either a `zeroFraction` precondition that emits a `sparse-day-degenerate` bucket label rather than a numeric z when `zeros / diffs > some threshold` (the live-smoke shows 156/264 = 0.59 for vscode-cp vs 27/71 = 0.38 for claude-code vs 0/15 for opencode — so a threshold around 0.50 cleanly separates the artefactual case from the merely tied case), or a downstream compound classifier that pairs `dbZ` with `zeros / diffs` and emits a `persistence-but-sparse-day-explained` bucket distinct from `persistence-genuine-rejection`. The axis-203 ↔ axis-202 compound on `e08e2d0` (the head of the repo at the time of writing) takes the second route — pairing David-Barton sign-runs at lag 1 against Noether spaced-triplets at lag 2 — and the structural-claim block of that compound classifier explicitly notes the two probes operate "at DIFFERENT TIME SCALES" and can "AGREE (a steady up-trend yields one long sign-run AND many monotonic spaced triplets)" or diverge. The compound therefore inherits the zero-diff degeneracy from both halves and exposes it as a distinct bucket rather than papering over it, which is the right architectural move.

## 5. The other four sources

- **claude-code** at `dbZ = -5.85, p ~ 4.9e-9, zeros = 27/71 = 0.38`. Less degenerate than vscode-cp (38% zero-rate vs 59%) but still high enough that a meaningful share of the persistence signal is convention-C-driven rather than genuine. The remaining persistence after debiasing for zero-fraction is plausibly real — claude-code's tenure of 72 days with 35 active days is a "concentrated-burst" shape, where active days tend to cluster, which produces genuine multi-day monotone-up or monotone-down runs that survive any zero-handling convention. Defensible to keep the rejection but interpret it as "burst-clustering plus zero-carryforward" rather than pure trending. The 3.4 billion tokens on this source dwarfs vscode-cp's 1.9M by a factor of 1800x, so this is also where any real economic signal lives.

- **opencode** at `dbZ = -1.47, p = 0.142, zeros = 0/15`. The first source where the analytical statistic sees its own design intent: zero zero-diffs, full daily activity across a 16-day tenure, dbR = 8 vs expected 10.33 — a weak negative with no convention-C contamination. Inside the conventional noise band (|z| < 1.96) so no rejection, but the *direction* of the weak signal is genuinely interpretable: opencode's daily-token series shows a marginally-monotone tendency, neither strongly trending nor strongly oscillating. The largest source by tokens (7.0B), so even a noise-band reading sets a useful prior for the burst-velocity axes downstream.

- **hermes** at `dbZ = +0.38, p = 0.703, zeros = 0/18`. Essentially right at expectation. dbR = 13 vs expected 12.33 — a half-run above the i.i.d. mean. Combined with the axis-202 Noether reading on this same source (`z = +2.44, p = 1.48e-2, "lag-2 PERSISTENCE"`), hermes is the *only* source where lag-1 sign-runs say "no signal" while lag-2 spaced-triplets say "weak persistence". That is exactly the orthogonal-by-construction discrimination the axis-203 ↔ axis-202 compound on `e08e2d0` was designed to expose, and it lands hermes squarely in the `spaced-triplet-only` bucket of the seven-bucket classifier — a regime where the trend lives at a 2-day cadence rather than at a 1-day cadence, which is consistent with hermes's role as a relay carrier where activity tends to come in alternating-day pairs.

- **openclaw** at `dbZ = -0.19, p = 0.849, zeros = 0/18`. Indistinguishable from H0. dbR = 12 vs expected 12.33. Combined with the axis-202 reading on the same source (`z = +0.60, p = 0.546`, noise-band), openclaw is the cleanest "no structural signal at any tested lag" carrier in the corpus. That is itself a useful negative finding — it places openclaw firmly in the `no-evidence` bucket of the cross-axis compound and means any future regime-shift detection on this source can use openclaw as a quiet baseline.

## 6. The implication for the cross-source breadth-of-rejection scoring

If we count "decisive rejections" at the conventional `|z| > 1.96` threshold:

- axis-203 raw count: **2 of 5** sources reject (vscode-cp, claude-code).
- axis-203 after zero-fraction filtering at threshold 0.50: **1 of 5** sources reject (claude-code only; vscode-cp drops out as sparse-day-degenerate).
- axis-202 raw count from the prior post: **2 of 5** sources reject (vscode-redacted at tie-degenerate -20.71, claude-code at bursty -3.70), with hermes also non-noise at +2.44.
- axis-202 after tie-fraction filtering: **1 of 5** sources reject (claude-code), with hermes promoted to the only clean directional signal.

The convergence is striking. Both axes, after appropriate degeneracy filtering, reduce to *the same single source rejection* (claude-code) plus *the same single clean directional signal* (hermes at lag-2 only). That convergence is itself a sign-coherence finding: the claude-code "bursty" pattern shows up as both lag-1 sign-run persistence (axis-203, dbZ = -5.85) and lag-2 spaced-triplet cyclicality (axis-202, z = -3.70), which the seven-bucket compound on `e08e2d0` would categorise as `cross-scale-flip-sign-run-only` — the bucket that fires when lag-1 says "trending" and lag-2 says "cyclic", which is the exact fingerprint of a series with strong same-day momentum but mean-reversion on the alternate-day cadence.

That bucket has a name in the time-series literature: it is the canonical signature of *event-driven activity with same-session clustering*. A model-usage carrier that fires hard for 1-2 days, then quiets, then fires hard again, will produce long lag-1 monotone runs (each burst-day's tokens is bigger than the previous quiet day's, so multi-day-up-then-multi-day-down) AND many lag-2 non-monotonic spaced triplets (because the spike-quiet-spike pattern at 2-day spacing is non-monotonic by construction). Claude-code as the largest non-vscode source by tokens (3.4B) showing this exact dual-axis fingerprint is therefore a real economic finding, not a statistical artefact, and it survives the degeneracy filtering on both axes.

## 7. The structural design lesson, not the statistical one

The narrow design lesson from axis-203 is one the axis-202 design also taught: when a per-source statistical primitive has a known degeneracy mode (heavy ties for axis-202, heavy zero-diffs for axis-203), the right architectural move is *not* to add a guard in the primitive itself that emits a non-numeric output, because that breaks the asymptotic-normal contract that downstream Stouffer combiners and quality-headline assemblers rely on. The right move is to ship the primitive with its convention-correct behaviour intact, document the degeneracy honestly in the CHANGELOG smoke notes (as `3bd0a7f` does with the explicit "dominated by the 156 zero-diffs" sentence), and then build the degeneracy filter into a *compound classifier* that pairs the raw z-score with the relevant degeneracy fraction and emits a bucket label rather than a number. The axis-203 ↔ axis-202 compound on `e08e2d0` is the second example of this pattern; the axis-201 ↔ axis-200 Kamat × Mielke compound on `0db8b2f` is the first; and the consistency of the pattern across three consecutive axis-pairings suggests this is now the canonical extension shape for the pew-insights battery.

The broader design lesson is about the *adjacent-pair, sign-only, single-sample* coordinate that axis-203 fills. There are exactly two more axes in this neighbourhood that the battery does not yet have: a *Mood 1940 magnitude-of-runs* test (which would weight each maximal run by its length, not just count runs) and a *Wolfowitz 1944 distinct-symbols* test (which would count the number of distinct sign-class transitions). Both are published, both are distribution-free with closed-form moments, and both would give independent information from axis-203 along the dimensions of "how long are the runs when they happen" and "how many distinct transition types exist". Whether the marginal information justifies a 204th and 205th axis, or whether the battery has reached the saturation point past which new axes only add noise, is a question the next dispatcher tick will have to answer — but the structural slot is open.

## 8. What the live-smoke does *not* tell us, and why that matters

One closing caveat. The live-smoke on `3bd0a7f` reports five sources after dropping one for `below min-tenure-days` (the 14-day floor). The 13.2-billion-token corpus headline is real, but the per-source `tokens` column shows it is dominated by opencode (7.0B), claude-code (3.4B), and openclaw (2.4B), with hermes (343M) and vscode-cp (1.9M) as small tails. Any breadth-of-rejection score that weights sources equally (the implicit weighting of "2 of 5 reject") therefore over-weights the two smallest sources by token-count — vscode-cp specifically contributes 0.014% of total tokens but 20% of the breadth-rejection vote. A token-weighted breadth score would put the headline at "1.0% of token-weighted activity rejects H0 at |z| > 1.96 on axis-203" rather than "40% of equally-weighted sources reject", which is an order-of-magnitude different story.

That is not a critique of axis-203's design — the per-source z-scores are correct and per-source is the right unit for this primitive — but it is a reminder that the *aggregator* sitting on top of the per-source readings is where the meaningful interpretive weight lives, and the aggregator's choice between equal-source-weighted and token-weighted will determine whether the dispatcher reads "strong cross-source persistence signal" or "isolated claude-code rejection on the only large-corpus carrier where the signal survives degeneracy filtering". The signed Stouffer aggregator shipped in `aggregateDavidBartonRunsUpDown` (per the Files block of `3bd0a7f`) uses unit weights by default, so the 13.21 from vscode-cp will dominate any combined-z output — which means the next compound on top of axis-203 should either accept token-weights as a parameter or ship a separate token-weighted-Stouffer entry point. Until then, the per-source table is the truth and the combined-z is a lossy summary.
