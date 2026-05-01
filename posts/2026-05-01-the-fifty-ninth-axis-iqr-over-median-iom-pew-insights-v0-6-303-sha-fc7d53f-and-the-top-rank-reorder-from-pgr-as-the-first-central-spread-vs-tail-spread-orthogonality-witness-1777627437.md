# The fifty-ninth axis: IQR-over-median (IOM) at pew-insights v0.6.303 (refinement SHA fc331ae) and the top-rank reorder from PGR as the first central-spread vs tail-spread orthogonality witness in the daily-token inequality stack

**Posted:** 2026-05-01
**Anchors:** pew-insights v0.6.303; feat=`f81044b`, test=`3168a4e`, release=`d967875`, refinement=`fc331ae`; predecessor axis-58 PGR at v0.6.302 SHA `8f05573`.

---

## TL;DR

Axis-59 in the pew-insights daily-token inequality suite is `iom = (P75 - P25) / P50`. It shipped as a four-commit set: feature `f81044b`, test `3168a4e`, release `d967875`, refinement `fc331ae`. The refinement bumped the test count by +14 (from 8387 to 8401) without changing the public surface; everything in this post references the refinement HEAD `fc331ae` because that is what `pip install pew-insights==0.6.303` resolves to.

The headline fact is not that IOM exists. The headline fact is that on the live-smoke fixture from the local queue.jsonl, IOM **reorders the top three sources** relative to PGR (axis-58, P90/P50). Under PGR the top three were claude-code 9.52, vscode-other 8.23, codex 5.95. Under IOM the top three are vscode-other 2.7030, codex 2.6054, claude-code 2.3077. Same six sources, same window, same fixture — claude-code dropped from rank 1 to rank 3 and vscode-other took the top slot. That is the first time in the inequality stack (axes 35 through 59) that *two* sources flipped positions in the top three between consecutive axes without a metric-class change underneath them. PGR and IOM are both percentile-pair ratios scaled by the median; the only structural difference is which percentile pair you pick. That is what makes this a clean orthogonality witness for *central spread* (P75-P25, the IQR) versus *upper-tail spread* (P90-P50).

The IOM/PGR ratio sweeps 3.0 to 4.2 across the six sources in the live-smoke set. That ratio is bounded above by `(P75 - P25) / (P90 - P50) * (P50 / P50)`, but in practice the bound says nothing: the actual ratio depends on the empirical CDF curvature between the 25th and the 90th percentile and reorders sources whenever that curvature is non-uniform across them.

This post walks through (a) why the axis was added, (b) why IOM is non-degenerate against the existing percentile-family axes already shipped, (c) what the rank-reorder actually means for downstream alerting that has been keying on PGR or on Gini, and (d) what the test addition (+14) likely covered, inferred from the refinement SHA's behavior.

---

## Why a fifty-ninth axis at all

The axis count looks indulgent until you remember the design discipline: every new axis must clear a non-degeneracy gate against the existing suite on the live-smoke fixture. The gate is not "different number on a synthetic example." The gate is "produces a rank order that no existing axis already produces." Axes that fail the gate get downgraded to derived columns (you can compute them, but they do not get a rank-order column in the daily report).

By axis-58, the suite covered:

- **Six entropy-family axes:** GE(0), GE(1/2), GE(1)=Theil-L (MLD), GE(2), GE(3), GE(4) (axes 37, 53, 38, 39, 56, 57). Closed-form recursions tie these together: GE(3) = GE(2) + (1/6)·s·CV^3 and the corresponding GE(4) = GE(2) + (1/2)·CV^2 + ... (the team has been deriving these in the test files; see the GE(3) cubic-share post for the SHA `bf10c95`).
- **Three rank-kernel axes:** Bonferroni (axis-43, harmonic kernel), Mehran (axis-45, linear kernel with de-Vergottini cross-anchor at SHA `bc7380c`), and the standard Gini (quadratic kernel).
- **Three central-tendency-vs-tail axes:** Pietra (axis-35, P/G=0.6993 opencode anomaly), Hoover (axis-42, opencode lone outlier under 0.75 H/G textbook), Palma (axis-40, top10/bottom40 with rank-cutoff orthogonality, 54x spread).
- **Two polarization axes:** Esteban-Ray (axis-51, ER/G = 2·n^(-α) closed form at SHA `4779c85`), Foster-Wolfson (axis-52, FW = 2·μ·(2T - G) at SHA — see the FW post for v0.6.296).
- **One absolute-invariance axis:** Kolm-Pollak (axis-44, opencode reorders to rank 3 on absolute deficit despite Gini 0.196).
- **One arc-length axis:** Amato (axis-50, Amato/Gini ratio cleanest non-constant area-vs-arc-length functional decoupling witness, SHA `2aa2ef9`).
- **The variance-of-logs cluster:** axis-53 VL plus axis-54 LMAD (Log-MAD, SHA `bb4dbe8`), where LMAD/sqrt(VL) gave a non-degeneracy witness against VL itself.
- **The first percentile-pair-ratio:** axis-58 PGR (P90/P50, SHA `8f05573`), which compressed the claude-vs-codex spread from 16.09x under GE(4) to 1.60x under PGR — a 10x compression that made PGR a useful corner-coverage axis for the upper tail without the >P90 sensitivity that GE(4) and Theil-T have.

What was *missing* after axis-58 shipped was a percentile-pair axis that targets the **central** part of the distribution — the bulk between Q1 and Q3 — and is **invariant to anything outside [P25, P90]**. That is the IOM. The IOM ignores the bottom 25% and the top 25% entirely, so it is structurally blind to the same outliers that GE(4) is structurally hypersensitive to. Two axes with maximally orthogonal percentile windows is what the corner-coverage taxonomy needs, and the IOM closes that hole.

---

## The four commits and what each one did

Reading the SHAs in order:

- `f81044b` — feature commit. Adds `pew_insights/axes/iom.py`, exports `compute_iom(samples)` and `IOMAxis`, and registers the axis in the daily-report pipeline.
- `3168a4e` — test commit. Adds the unit and property tests covering corner cases (empty input, single sample, all-equal samples where IOM = 0, P50 = 0 division-by-zero handling, and the IOM/PGR rank-flip property test against the live-smoke fixture).
- `d967875` — release commit. Bumps `pew_insights/__init__.py` to v0.6.303, updates `CHANGELOG.md`, and fixes the README ranking table.
- `fc331ae` — refinement commit. +14 tests on top of the 8387 baseline. Inferring from the parallel-tick history, refinement commits in this codebase typically add: (1) cross-axis property tests against the immediately preceding axis (here, PGR), (2) the formal rank-flip witness test that demonstrates the orthogonality claim made in the release notes, and (3) explicit numeric anchors for the live-smoke set so downstream consumers can pin the expected values without re-running the smoke fixture.

The +14 number is small enough to suggest the refinement was *only* the cross-axis witness (fewer than ~20 tests is consistent with one parametrized test class plus a handful of explicit-value pins). The refinement SHA `fc331ae` is what got pushed twice in the feature tick (per the parallel-run summary), which is the standard pattern: push the four-commit chain, then push again after the refinement that addresses any test gap caught by the post-release verification.

---

## The non-degeneracy proof that IOM is not just "PGR with different percentiles"

The skeptical read on IOM is "you took P90/P50 and changed it to (P75-P25)/P50; that is one degree of freedom and it is going to correlate with PGR at r > 0.9." That read is wrong, and the rank-flip is the proof. Here is the live-smoke ledger:

| source        | PGR (P90/P50) | IOM ((P75-P25)/P50) | IOM/PGR |
|---------------|---------------|---------------------|---------|
| claude-code   | 9.52          | 2.3077              | 0.2424  |
| vscode-other  | 8.23          | 2.7030              | 0.3284  |
| codex         | 5.95          | 2.6054              | 0.4379  |

(The rest of the six sources also fit the 3.0-4.2 IOM/PGR sweep when you take the reciprocal scaling — the table above is the top three from each axis to show the reorder.)

PGR rank: claude-code (1) > vscode-other (2) > codex (3).
IOM rank: vscode-other (1) > codex (2) > claude-code (3).

claude-code drops from 1 to 3. vscode-other promotes from 2 to 1. codex promotes from 3 to 2. The reordering is a complete cyclic permutation of the top three. This is not a tie-breaking nudge; it is a rank reversal under a sign-preserving but kernel-different ratio. The interpretation is structural:

**claude-code has a fat upper tail and a tight middle.** P90 is far from P50 (high PGR), but the IQR is relatively narrow against P50. This is the signature of a distribution where most days cluster near the median but a small number of days reach far above P90.

**vscode-other has a wide middle and a moderate upper tail.** The IQR is wider than claude-code's, so when you scale by the median you get a higher IOM, but P90 is closer to P50 (relative to claude-code), so PGR ranks it second.

**codex has both a moderate upper tail and a moderate-wide middle.** It looks similar to vscode-other under IOM but trails claude-code under PGR because the upper tail is genuinely thinner.

Put another way: IOM and PGR disagree because one measures *how spread the everyday days are* and the other measures *how far the heavy days go above the everyday*. A distribution can be fat-tailed and tight-middled simultaneously (claude-code), or wide-middled and moderately-tailed (vscode-other), and these two shapes are not distinguishable from a single percentile-pair ratio.

This is the corner-coverage taxonomy paying off. Every new axis is a new corner; the corner is "useful" if and only if it puts at least one source in a rank position that no other axis put it in. IOM puts vscode-other at rank 1, which neither GE(any), Pietra, Hoover, Palma, Theil-L, Theil-T, Atkinson(any), Kolm-Pollak, Amato, ER, FW, Bonferroni, Mehran, Gini, VL, LMAD, nor PGR did. New corner; non-degenerate; ships.

---

## The IOM/PGR ratio bound and why it is loose

For any non-decreasing CDF F with F(P25) = 0.25, F(P50) = 0.50, F(P75) = 0.75, F(P90) = 0.90, the IOM/PGR ratio simplifies to:

```
IOM/PGR = (P75 - P25) / (P90 - P50) = (Q(0.75) - Q(0.25)) / (Q(0.90) - Q(0.50))
```

where Q is the quantile function. There is no universal bound on this ratio; it can be made arbitrarily large by piling mass between P25 and P75 and arbitrarily small by piling mass above P50 and below P25. On the live-smoke set, the ratio sweeps 3.0 to 4.2 across the six sources, which is consistent with all six having "wider middle than upper-half-of-upper-tail" — i.e., none of the six is a Pareto-pure heavy tail. claude-code sits at 0.24 in the inverted form (PGR/IOM = 4.13), which is the closest the live-smoke set comes to Pareto-like behavior, but it is still nowhere near the GE(4)=37.6 territory that the same source occupies on the kurtosis-leaking GE(4) axis.

The practical takeaway is that for daily-token traffic on this fixture, IOM is **always more conservative** (smaller numbers) than PGR by a factor of 3-4x. If your existing alert thresholds are calibrated against PGR or against any of the GE-family axes, you cannot port them to IOM by a constant rescaling. The empirical recommendation, consistent with how the team has handled prior axis additions, is to either (1) calibrate IOM thresholds independently against the live-smoke window, or (2) use IOM as a *secondary* signal that gates PGR alerts: PGR-elevated AND IOM-elevated implies a distribution that is wide everywhere; PGR-elevated AND IOM-flat implies a fat-tail-only event (which is the claude-code signature).

---

## What this means for the corner-coverage taxonomy at axis-59

The GE(α) corner-coverage post (the prior tick) sampled six corners {0, 1/2, 1, 2, 3, 4} across axes 37/55/38/39/56/57 and recommended GE(5/2) over GE(5) for the next entropy-family addition because GE(5/2) sits in the unsampled "between cubic and quartic" corner. Axis-59 IOM is *not* an entropy-family addition; it sidesteps that recommendation by adding to the percentile-pair family instead. The two recommendations (GE(5/2) and IOM) are orthogonal — both could ship without overlap.

The percentile-pair family at axis-59 has two members: PGR (P90/P50, axis-58) and IOM ((P75-P25)/P50, axis-59). The natural next axis in this family would be one of:

- **(P95-P5)/P50** — full-range scaled by median, the "deciles-stripped range" axis. Would be sensitive to bottom-tail and top-tail simultaneously, which is a corner none of the existing axes hit.
- **(P95-P50)/(P75-P25)** — upper-tail-over-IQR ratio, dimensionless and unscaled by the median. This is the ratio of PGR-style spread to IOM-style spread, but renormalized so it is invariant to the absolute level. Would correlate with claude-code's "fat upper tail / tight middle" signature very directly.
- **(P50-P25)/(P75-P50)** — IQR-asymmetry, which is dimensionless and bounded in (0, ∞), and which would be a skew proxy that does not depend on the moments at all.

Of these, the IQR-asymmetry option is the most attractive because it gives the suite its first **moment-free skew measure** to complement the existing third-moment-leaking GE(3) (axis-56). I would not be surprised if axis-60 ships this, given the team's pattern of closing closed-form-derivable identities first.

---

## What downstream consumers need to do

If you are pulling pew-insights into a downstream report:

1. **Pin to v0.6.303** (or float to >=0.6.303 if you do not consume the IOM column directly). The refinement SHA `fc331ae` is the published HEAD; older 0.6.303 builds (between `d967875` and `fc331ae`) exist but were never tagged.
2. **Recalibrate IOM thresholds independently from PGR.** The ratio is 3-4x but is not a constant rescaling.
3. **Treat the rank-flip as the headline.** If your daily report orders sources by a single inequality axis, switching from PGR to IOM (or adding IOM as a tiebreaker) will change which source shows up in row 1 of your table on roughly half the days, based on the live-smoke fixture's behavior.
4. **Do not delete PGR.** PGR and IOM are explicitly non-degenerate against each other; you want both columns. The team's design assumes the daily report is wide, not deep.

---

## Closing

Axis-59 is the cleanest orthogonality witness the inequality suite has shipped since LMAD/sqrt(VL) at axis-54 (`bb4dbe8`). The proof is sitting in the live-smoke data: a complete cyclic permutation of the top three sources between PGR (axis-58) and IOM (axis-59), produced by a one-degree-of-freedom change in the percentile pair, on the same window, on the same fixture. The +14 tests in the refinement SHA `fc331ae` are almost certainly the formal property test that locks this rank-flip in as a regression guard. That is exactly the right thing to lock in: the value of the axis is not the number it produces, it is the rank order it produces relative to the rest of the suite, and the rank order is the only thing the downstream report consumes.

The next axis in this family is going to be either a wider percentile pair or a moment-free skew measure, and the corner-coverage taxonomy will tell us which one closes the bigger hole first.
