# pew axis-147 calendar-mask RLE-entropy as the third path-dependent cross-source daily-token axis, and the shape-versus-magnitude orthogonality witness against axes 145 and 146

Date: 2026-05-04
Source axis: `pew-insights daily-token-calendar-mask-rle-entropy`
Shipping commits: `5dd9576` (initial implementation + tests), `365c98c` (refinement: `entropyDeficitBits` + `dominantSegmentShare` / `Kind`)
Version bump: `0.6.393 -> 0.6.394` (commit `cf96a37`)

## 1. Why a third path-dependent axis at all

The cross-source daily-token measurement program at pew-insights has, for most of its run, been dominated by **permutation-invariant** functionals. Gini, HHI, Pielou, CR4, Atkinson, Theil, Hoover, Pietra, Bonferroni, Mehran, Wolfson, Foster-Wolfson, Palma, Kolm-Pollak, Chakravarty, Amato, Esteban-Ray, FGT, the GE family, Var-of-Logs, Log-MAD, Zenga, S-Gini, Hill-tail, decile-share-gap, quintile-share-ratio, percentile-gap-ratio, top-4-CR — every one of those is a function of the **multiset** of active-day token counts. Reorder the days, drop the calendar entirely, and the value does not move.

That class is mathematically rich, but it is also fundamentally blind. It cannot see whether a source emitted its tokens in a single contiguous burst, in a regular weekly pulse, or in two clusters separated by a 28-day silence. The cross-table evidence accumulated over the prior 144 axes has been, in this sense, a single very high-dimensional view of the **active-day distribution shape**, repeated under many functional substitutions, with zero leverage on calendar geometry.

Axis-145 (max-drawdown-rate, MDR) and axis-146 (longest-zero-run, LZR) finally cracked that wall. MDR sees magnitude collapse — the steepest peak-to-trough fractional decline in the per-day cumulative — while LZR sees the worst single silent stretch in the calendar mask. Together they were the first two members of a path-dependent family.

Axis-147 is the third member, and the first one that is genuinely a **shape** functional rather than a single-extremum functional. It computes the Shannon entropy of the run-length encoding of the 0/1 calendar mask `[firstActiveDay..lastActiveDay]`, in bits.

## 2. The construction in one paragraph

For a source with daily token series `total_tokens[d]` over the inclusive window `[firstActiveDay..lastActiveDay]`, define the binary mask `m[d] = 1 iff total_tokens[d] >= min-tokens`. Run-length-encode the mask into segment lengths `r_1, r_2, ..., r_K` (with `sum_i r_i = spanDays`). Each `r_i` belongs to either an active or a silent segment, alternating. Then compute

    H = - sum_{i=1..K} (r_i / spanDays) * log2(r_i / spanDays)

with units of bits. The range is `[0, log2(K)]`: zero when there is a single segment (either all-active or, in the degenerate edge case, all-silent), maximised at `log2(K)` when all `K` segments share equal length `spanDays/K`. The published axis adds a normalised companion `rleEntropyNormalised = H / log2(K) in [0,1]` so that fragmentation regime can be read directly without reasoning about K.

## 3. Why RLE-entropy is not a relabelling of axis-146

The temptation, on first reading, is to dismiss axis-147 as "a fancier longest-zero-run". The CHANGELOG entry for 0.6.394 anticipates that objection and ships a witness in the test suite that falsifies it on a single counterexample.

Construction (verbatim from the CHANGELOG):

- mask A = `[1,0,0,1,0,0,1]`: LZR = 2, segments `[1,2,1,2,1]`, K = 5
- mask B = `[1,1,0,0,0,0,1]`: LZR = 4, segments `[2,4,1]`, K = 3

LZR differs (2 vs 4). RLE-entropy differs in the opposite-sense direction: mask A has a uniform-ish segment distribution dominated by length-1 and length-2 runs and yields a higher normalised entropy, whereas mask B has one dominant length-4 silent segment and yields a lower normalised entropy despite the worse LZR. The crucial property is not that the two axes give different numbers — that is trivially true — but that they can give numbers in **opposing rank order**: a source with a worse LZR can have a more orderly mask shape, and a source with a tame LZR can be heavily fragmented. Any monotone function of LZR is therefore ruled out as an explanation for axis-147.

This is the first time in the daily-token axis program that we have two members of the path-dependent family that are simultaneously (a) both calendar-aware and (b) provably non-monotone in each other. That gives us, for the first time, a genuine 2-D path-dependent diagnostic plane (LZR on one axis, RLE-H on the other) rather than a single line of "more or less calendar-rough".

## 4. Why RLE-entropy is not a relabelling of axis-145

The MDR-vs-RLE-H separation is shipped in the same test suite under a different witness:

- Source U = `[100, 1, 100]` on three adjacent days. MDR = 0.99 (deep one-day collapse). Calendar mask = `[1,1,1]`. RLE-H = 0 (single active segment).
- Source V = `[100, 100]` on day-1 and day-30 only. MDR = 0 (monotone non-decreasing cumulative). Calendar mask = `[1,0,0,...,0,1]` of length 30, segments `[1,28,1]`. RLE-H ≈ 0.55 bits.

So the same source can score worst on one path-dependent axis and best on the other. MDR is a magnitude functional on the **active-day value vector**; it ignores the calendar entirely once the active days are listed in order. RLE-H is a shape functional on the **calendar 0/1 mask**; it ignores token magnitudes entirely once the mask is computed. The two functionals are operating on disjoint sub-objects of the same source.

This is, structurally, the first cross-source axis pair where the input objects to the two functionals are disjoint. Every prior pair shared at least the active-day value vector as common input. Axis-145 and axis-147 share only the underlying timestamped event log, and the two functionals lift it into different reduced objects.

## 5. The 0.6.394 refinement: `entropyDeficitBits` and `dominantSegmentShare/Kind`

The initial implementation (commit `5dd9576`) shipped the scalar `rleEntropyBits` and the normalised companion. The refinement at commit `365c98c` added two diagnostics that turn out to matter for live-queue interpretation:

- `entropyDeficitBits = log2(K) - rleEntropyBits`. This is the absolute distance to the per-source maximum-entropy ceiling. It is more useful than the normalised value when comparing two sources with different K, because two sources both at `rleEntropyNormalised = 0.6` are at very different absolute distances from their own ceilings if one has K = 3 and the other K = 30.
- `dominantSegmentShare = max_i r_i / spanDays`, with companion `dominantSegmentKind in {active, silent}`. The dominant segment carries the largest single mass in the entropy sum and is, almost by construction, the largest single contributor to the entropy deficit. Its **kind** matters: a source whose dominant segment is silent is in a long-dormancy regime; a source whose dominant segment is active is in a long-burst regime. Two sources with identical `rleEntropyBits` and identical K can therefore be in opposite operational regimes. The `Kind` field exposes that distinction without a join back to the underlying mask.

These are the kinds of refinements that only emerge from looking at the live-smoke output. The CHANGELOG entry includes the full live-smoke per-source row dump under `--min-tokens 1000 --since 2026-04-04 --until 2026-05-04`, and reading that table is what surfaced the K-disparity confound that motivated `entropyDeficitBits`.

## 6. The fragmentation-regime classifier

The per-row diagnostic `fragmentationRegime` is a six-class label derived from `rleEntropyNormalised`:

- `continuous`: no silent segments at all — the mask is all 1s over `[firstActiveDay..lastActiveDay]`. RLE-H is bounded above by `log2(K)` where K is small; this is the "always-on" source class.
- `low-fragment`: one or two short silent segments, normalised entropy ≤ 0.3.
- `fragmented`: normalised entropy in (0.3, 0.6].
- `shattered`: normalised entropy in (0.6, 0.85].
- `pulverised`: normalised entropy in (0.85, 1.0).
- `degenerate`: K ≤ 1, entropy 0 by definition (single active or single silent segment over the whole span).

The classifier is intentionally coarse — six bins for a continuous quantity is well below what the underlying axis can resolve — but coarseness is the point. The classifier exists so that downstream consumers (the digest renderer, the per-drip narrative) can **name** a source's calendar texture without having to quote the raw bits. "litellm is in `pulverised` regime this week, qwen-code is in `low-fragment`" is more useful in a paragraph than "litellm RLE-H = 2.81 bits over K = 14, qwen-code RLE-H = 0.42 bits over K = 5".

## 7. Where this fits in the path-dependent axis ladder

The ladder, as of version 0.6.394, has three rungs:

1. Axis-145 (max-drawdown-rate): magnitude functional on active-day value vector. First cross-source path-dependent axis. Permutation-sensitive in the value sequence.
2. Axis-146 (longest-zero-run): single-extremum functional on the silent-segment lengths only of the calendar mask. Permutation-sensitive in the mask, but only via the maximum.
3. Axis-147 (calendar-mask RLE-entropy): shape functional on the full segment-length distribution of the calendar mask. Permutation-sensitive in the mask, via the entire shape.

Each rung adds a strictly larger amount of calendar information into the cross-source comparison. Axis-145 sees only the active-day cumulative (a path on R). Axis-146 sees only the worst silent gap (a single scalar from the mask). Axis-147 sees the full segment-length profile (a vector). The natural fourth rung — and a candidate for axis-148 — is something like an **active-segment-length entropy** that conditions only on active runs, separating "long active bursts with rare gaps" from "short active flickers with rare gaps". The CHANGELOG does not pre-commit to that, but the structural argument for it falls out of the same analysis that made axis-147 worth shipping.

## 8. Falsification surface

A new axis is only worth shipping if there is a short list of empirical claims that could falsify it. For axis-147 the falsifiers are:

- **Collinearity claim**: if `rleEntropyBits` is, on the live cross-source matrix, a near-perfect monotone function of either axis-145 or axis-146 across all observed sources, then the axis carries no marginal information and should be retired. The witness pair shipped in tests proves this is not analytically true; the live-smoke output for 0.6.394 will need to be re-checked at every drip to confirm it is not empirically true within rank-correlation tolerance.
- **Regime-classifier saturation**: if every source falls into a single fragmentation regime over a long window, the classifier is uninformative and the bin boundaries need recalibration. The 0.6.394 live-smoke shows at least three regimes populated simultaneously in the daily-token table — `continuous`, `low-fragment`, and `fragmented` — so this falsifier is not currently triggered.
- **Dominant-kind degeneracy**: if `dominantSegmentKind` is always `active` (or always `silent`) across the live source set, the field carries no separator value. The 0.6.394 live-smoke shows both kinds populated, so this is also not triggered.

If none of these falsifiers fires across the next ten drips, axis-147 graduates from "newly shipped" to "stable diagnostic" and earns the right to be cited in cross-source narrative posts without re-justification.

## 9. Operational consequence for the live queue

The immediate operational consequence is that the per-drip review summaries can now talk about a source's calendar texture in addition to its activity volume and concentration. A source with high HHI and high `rleEntropyNormalised` is concentrated in volume **and** fragmented in time — the worst combination for a downstream classifier that wants stationary inputs. A source with low HHI and `continuous` regime is the easiest case. The cross-product of axis-143 (HHI) and axis-147 (RLE-H) defines a 2x3 grid of operational regimes that the digest renderer will start exposing in the weekly roll-up.

This is, in a sense, the entire reason for shipping axis-147 ahead of any further inequality refinement: the marginal information it contributes is in a dimension orthogonal to everything the inequality / diversity stack has been measuring for 144 axes. It buys a new column for the cross-source comparison table, not a refinement of an existing column. That is the highest-value kind of axis to add when the existing column count is already in the hundreds.

## 10. Summary

Axis-147 ships at version 0.6.394 (`cf96a37`) with implementation `5dd9576` and refinement `365c98c`. It is the third path-dependent cross-source daily-token axis after MDR (axis-145) and LZR (axis-146). It is provably non-monotone in both prior path-dependent axes, with witnesses in the test suite. It introduces a six-class `fragmentationRegime` classifier and two diagnostics (`entropyDeficitBits`, `dominantSegmentKind`) that emerged from live-smoke inspection. It adds an orthogonal column to the cross-source comparison matrix rather than refining an existing one, which is the highest-leverage shape an axis can take when the existing axis count is already large. The next natural rung — an active-segment-only entropy at axis-148 — is structurally implied but not yet committed.
