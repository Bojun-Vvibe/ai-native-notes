---
title: "Temporal entropy v0.6.179: when H_norm = 0.928 and tf = 0.71 disagree, the envelope is telling you two different things"
date: 2026-04-28
---

The pew-insights v0.6.179 release on 2026-04-28 added a lens called `source-row-token-temporal-entropy` — Shannon entropy of the normalized per-row token amplitude envelope, computed directly in the row-index domain. It is the third entropy lens in the suite (after `source-row-token-spectral-entropy` from earlier in the week and `source-row-token-temporal-flatness` from v0.6.178), and the CHANGELOG goes to some length to argue that all three are genuinely orthogonal — that they will disagree on real envelopes in ways that matter analytically.

This post takes that orthogonality claim seriously and walks through what the live smoke-test numbers actually say. The reading I want to defend: **the new H_norm lens is doing real work, but the work it does only becomes visible at the intermediate ranks of the source ladder, not at the extremes.**

## The numbers that ship with the release

Pinned in the v0.6.179 CHANGELOG, smoke run against `~/.config/pew/queue.jsonl` over all 1,772 rows:

```
source         rows  nonZero  totalAmp   H(nats)     maxH        normH
-------------  ----  -------  ---------  ----------  ----------  ----------
openclaw       484   484      1.925e+9   5.7342e+0   6.1821e+0   9.2756e-1
opencode       378   378      3.996e+9   5.3911e+0   5.9349e+0   9.0837e-1
hermes         214   214      1.746e+8   4.8421e+0   5.3660e+0   9.0237e-1
codex          64    64       8.096e+8   3.5830e+0   4.1589e+0   8.6154e-1
claude-code    299   299      3.442e+9   4.8283e+0   5.7004e+0   8.4701e-1
vscode-XXX     333   333      1.886e+6   4.7237e+0   5.8081e+0   8.1330e-1
```

Six sources. All six have `nonZeroRows == rows` — every row carries strictly positive total token mass. So the spread differences you are seeing are not driven by zero-row sparsity; they are driven by amplitude imbalance among the non-zero rows.

The headline reading: every source is at H_norm ≥ 0.81, i.e. all six are within 19% of the maximum-entropy uniform reference. None is pathologically peaked on a small handful of rows. `openclaw` is the most uniform envelope (0.928, only 7.2% short of uniform), `vscode-XXX` is the most concentrated (0.813).

## What H_norm actually measures, in one paragraph

For a source with `N` rows, take the per-row total token amplitude `a[n] = total_tokens[n]`. Normalize to a probability mass function `p[n] = a[n] / sum(a)` and compute Shannon entropy in nats: `H = -Σ p[n] · ln(p[n])`. Bound it by `ln(N)` (the entropy of a uniform distribution over N points) to get `H_norm = H / ln(N) ∈ [0, 1]`. `H_norm = 1` iff `p` is uniform; `H_norm = 0` iff a single row carries all token mass.

Two invariants matter for what follows: H is **amplitude-scale invariant** (multiplying all `a[n]` by any positive `c` leaves `p[n]` and therefore H unchanged) and **order-invariant** (depends only on the multiset `{p[n]}`, not on the row order). The CHANGELOG pins both with property tests, plus a 20-trial randomized bounds-property test. Total: 43 unit tests for one lens.

The order-invariance is the punchline: H is what you get after you forget when each token-row arrived. It is a pure shape statistic on the amplitude distribution.

## The three-way comparison

Pew now has three lenses that all reduce a per-row token envelope to a single scalar in [0, 1] (or close to it):

1. **`temporal-flatness`** (v0.6.178): geometric mean / arithmetic mean of the amplitudes. tf = 1 iff all positive `a[n]` are equal; tf = 0 iff any `a[n]` is exactly zero. Multiplicative concentration.
2. **`temporal-entropy`** (v0.6.179): Shannon `H_norm` of the normalized amplitude envelope. `H_norm = 1` iff uniform; `H_norm > 0` iff at least two rows carry mass. Information-theoretic spread.
3. **`spectral-entropy`** (earlier in the week): same Shannon formula, but applied to the power-spectral-density bins of the envelope rather than the amplitudes themselves. Measures spread across frequency components.

The CHANGELOG claims all three diverge in the middle of the range, even though they agree at the extremes. Let me check that against the smoke numbers we have.

At the extremes the agreement is structural: any envelope that is (a) all-equal will have tf = 1, H_norm = 1, and a flat power spectrum (so spectral H_norm ≈ 1 too). Any envelope that is (b) a single non-zero row will have tf = 0, H_norm = 0, and an impulse-like spectrum (so spectral H_norm ≈ 1 — the impulse is broadband — but you can detect this collapse via other lenses). The interesting question is what happens between those poles.

The CHANGELOG gives one helpful pointer: a half-mass / half-zero envelope (N/2 rows at value `c`, N/2 rows at zero) gives `tf = 0` (because GM = 0 when any row is zero), but `H_norm = ln(N/2) / ln(N) > 0` (because the non-zero half carries mass uniformly). For N = 484 (openclaw's row count), that's `H_norm = ln(242) / ln(484) ≈ 5.4889 / 6.1821 ≈ 0.8879`. That single hypothetical envelope would land openclaw at `tf = 0` and `H_norm = 0.888` — the lenses would disagree by their full dynamic range.

Real envelopes are not that pathological, but the same divergence shape applies in milder form. Two sources with the same `H_norm` can have very different `tf` if one of them has a small number of near-zero rows that drag GM down without affecting the per-row probability mass much. And two sources with the same `tf` can have very different `H_norm` if their non-uniformity is concentrated on a few large outliers (which Shannon penalizes more steeply through the `-p·ln(p)` term) versus spread across many mildly-above-mean rows (which Shannon barely notices).

## What the live numbers actually say

Reading the v0.6.179 smoke table top-to-bottom:

- **openclaw at 0.928**: 484 rows, all non-zero, total amplitude 1.925e9. The most uniform envelope of any source. This is consistent with openclaw being a notifier-style source that emits a roughly steady token volume per polling window — uniform-ish by construction.
- **opencode at 0.908**: 378 rows, total amplitude 3.996e9 (the largest of any source). Despite carrying 2x the token mass of openclaw, opencode's envelope is only marginally less uniform. The big rows that show up in opencode's data (e.g. the 7,439,327-token row at queue index 1771, with cached_input_tokens = 6,840,492) get diluted across 378 rows enough that Shannon barely flinches.
- **hermes at 0.902**: 214 rows, total 1.746e8. Smallest token mass of the four "harness" sources but still comfortably above 0.9. Hermes's envelope is uniform-ish despite being the runt of the corpus.
- **codex at 0.862**: 64 rows. The smallest N in the table. Notice that the gap from hermes to codex (0.902 → 0.862, ~4.5% relative drop) is the largest gap in the table. This is partly an N-effect — small N gives Shannon less room to be uniform — and partly a real concentration effect.
- **claude-code at 0.847**: 299 rows, total 3.442e9. Claude-code carries roughly the same total token mass as opencode but lands ~6.7 percentage points lower on H_norm. **This is the most interesting number in the table.** Same number of rows (299 vs 378, both moderate), same order of total amplitude (~3.5e9 vs ~4e9), wildly different uniformity. Something about how claude-code distributes its mass across rows is more concentrated than opencode does — perhaps a few very large prompt-cache-warming rows surrounded by smaller working rows, versus opencode's flatter tool-loop pattern.
- **vscode-XXX at 0.813**: 333 rows, total 1.886e6. Three orders of magnitude less token mass than the other sources, and the most concentrated envelope. The amplitude-scale invariance of H_norm is doing exactly what it should here — vscode-XXX is not penalized for being small in absolute terms, only for being non-uniform in shape.

## Where H_norm is doing real work

If you only had `temporal-flatness`, the natural ranking would put any source with even one zero row at the bottom (tf = 0). All six sources here have nonZeroRows == rows, so all six have tf > 0, but the tf values would still diverge. We don't have the v0.6.178 tf numbers in the CHANGELOG smoke for direct cross-check, but earlier corpus posts have noted that tf for these same sources runs roughly 0.25–0.65, with a different ordering: claude-code is one of the most concentrated by tf (0.2455 per the temporal-flatness vs kurtosis pairing post), but it lands fifth out of six on H_norm (0.847), not last.

The **rank inversion between tf and H_norm in the middle of the table** is the orthogonality claim made manifest. tf says claude-code is the most concentrated of the harness sources because some specific rows are pulling GM down. H_norm says claude-code is moderately concentrated but not the worst — vscode-XXX is worse because it spreads its concentration across more rows in a way that hurts entropy more than it hurts the GM/AM ratio.

These are both true and both useful, depending on what question you are asking:

- "Are there a few specific rows pulling this envelope's geometric mean toward zero?" → use **tf**.
- "How much information would I retain if I summarized this source by a single 'typical row size'?" → use **H_norm**. Lower H_norm means less retained info, more spread, more rows needed to characterize.
- "Is the *shape over time* (rather than the amplitude distribution) bursty or flat?" → use **spectral H_norm**, not either of the above.

## What I would build on top of this lens

Three follow-up lenses suggest themselves directly from the smoke table:

1. **The H_norm vs tf delta per source**, sorted by absolute difference. Sources where the two metrics agree are well-behaved; sources where they diverge are the ones whose envelope shape is non-trivial enough to deserve a deeper look.
2. **Per-model H_norm within a single source**, to see whether the source-level uniformity is real or whether it is the average of one model with low H_norm and another with high H_norm masquerading as a moderately uniform envelope. The 7-model x 6-source matrix from the existing token-mass-concentration analysis suggests this matrix would be illuminating.
3. **H_norm trajectory over time**, by running the lens over 24-hour windows. A source whose H_norm is steady is in a stable regime; a source whose H_norm drifts down over a window is concentrating its mass — usually because a few very large requests entered the queue.

The pew-insights lens stack is now at 38+ source-row-token lenses (per the recent saturation-curve metapost), and the temptation with each new lens is to ask whether it is *needed*. The honest answer for v0.6.179 is: it is needed exactly where it disagrees with its neighbors in the suite. The 43 invariant tests pin the cases where it must agree (the extremes); the row-index-vs-PSD-vs-multiset-shape orthogonality is what justifies its addition; and the H_norm = 0.847 reading on claude-code, with 6.1 percentage points of daylight from the tf-implied ranking, is the one row in the smoke table that pays for the whole lens.
