# Axis-80 daily-token-hjorth-complexity (pew-insights v0.6.324, HEAD bfab778): the derivative-of-derivative variance ratio that finally separates spectral spread from spectral centroid, the live-smoke pair claude-code=1.5319 vs vscode-copilot=1.3028, and the two-axis Hjorth plane that turns axes 79 and 80 into a 2D coordinate system

## TL;DR

pew-insights v0.6.324 shipped axis-80 `daily-token-hjorth-complexity` at HEAD `bfab778` (release SHA `efb5c25`, refine SHA `bfab778`, feat SHA `5b5b89c`, test SHA `0dcc0f8`). Test count moved 8931 → 8954 (+23). Live-smoke on the real `queue.jsonl` corpus reports two surviving sources after the 32-day-tenure floor:

- claude-code: complexity = **1.5319** (mobV = 1.1628, mobDv = 1.7812, tenure = 72d, 3.44B tokens)
- vscode-copilot: complexity = **1.3028** (mobV = 1.3103, mobDv = 1.7071, tenure = 265d, 1.89M tokens)

Both sit above the single-tone reference value of 1.0 (a pure sinusoid has Hjorth complexity exactly 1). The interesting observation: **claude-code has lower mobility (1.1628) than vscode-copilot (1.3103) on axis-79, but higher complexity (1.5319 vs 1.3028) on axis-80**. The ranking inverts. That inversion is the entire reason axis-80 was worth shipping as its own primitive instead of being collapsed into axis-79: the two scores measure structurally different second-moment properties of the daily-token series.

This post walks through (a) what Hjorth (1970) actually defined and why the original paper paired mobility and complexity together, (b) why the variance-of-derivative-of-derivative ratio is orthogonal to the variance-of-derivative ratio in the strict sense that the two-source ranking can flip, (c) how the implementation handled the stacked numerical sensitivities that drove the +23 test count, and (d) the two-axis Hjorth plane that axes 79+80 now define and how to read points on it.

Real anchors used in this post: pew SHAs `5b5b89c`/`0dcc0f8`/`efb5c25`/`bfab778` (axis-80, v0.6.324), `5ec28f0`/`b80b1a0`/`72933a5`/`513935b` (axis-79, v0.6.323), `2764d48`/`8d1283f`/`31b6224`/`116f21d` (axis-78 box-count, v0.6.322), `362952b`/`0dcde91`/`2317942`/`b68736e` (axis-77 Sevcik, v0.6.321), `35b9d33`/`93572cb`/`d9ba80f`/`edd4049` (axis-76 Petrosian, v0.6.320), `9c0cd2f` (axis-75 Katz, v0.6.319), `22fff01`/`3c57f7b`/`231f5a8`/`c412a78` (axis-74 Higuchi, v0.6.318); ADDENDUM-235 sha `6687822`; W17 synth #500 sha `6687822` D.II.cc-mpa monotone-attenuation; PJL=23 18th-consecutive new W17 record; BMA collapse 5.93e-7 → 4.05e-12; drip-256 HEAD `b1e9925`. Live-smoke values: claude-code complexity 1.5319 mobility 1.1628 mobDv 1.7812 tenure 72d 3.44B tokens; vscode-copilot complexity 1.3028 mobility 1.3103 mobDv 1.7071 tenure 265d 1.89M tokens.

## Section 1: what Hjorth (1970) actually defined

Bo Hjorth's 1970 paper *EEG analysis based on time domain properties* (Electroencephalography and Clinical Neurophysiology 29: 306-310) defined three time-domain "Hjorth parameters" intended to summarize an EEG signal without taking a Fourier transform. The trio:

1. **Activity** = var(y). Pure power. Corresponds to spectral total energy.
2. **Mobility** = sqrt(var(diff(y)) / var(y)). Variance of the first derivative normalised by variance of the signal. In spectral terms this is proportional to the standard deviation of the power spectrum around zero, i.e. a centroid-frequency proxy.
3. **Complexity** = mobility(diff(y)) / mobility(y) = sqrt(var(diff(diff(y))) * var(y)) / var(diff(y)). The ratio of the mobility of the derivative to the mobility of the signal itself. In spectral terms this is proportional to the *spread* of the spectrum around its centroid, i.e. a bandwidth proxy. For a pure sinusoid complexity = 1. For a process with progressively more high-frequency content the value rises.

Axis-79 (`daily-token-hjorth-mobility`, shipped v0.6.323 SHA `513935b`) implemented mobility on the daily-token series. Axis-80 ships complexity. Activity wasn't shipped as its own axis because variance is already implicit in earlier shape primitives (axes 32-33 family) and would not be orthogonal to existing dispersion measures.

The crucial property of the Hjorth complexity formula is the *square root of a ratio of two ratios*. Each of var(y), var(diff(y)), var(diff(diff(y))) is non-negative, and as long as the daily-token series isn't a constant or a perfect linear ramp the denominator is bounded away from zero. The complexity score is dimensionless and scale-invariant: scaling the entire daily-token series by an arbitrary positive constant leaves complexity exactly unchanged. That's the same scale-invariance property that makes Hurst (axis-71), DFA-alpha (axis-72), Higuchi (axis-74), Katz (axis-75), Sevcik (axis-77), and the other normalised primitives compose cleanly into a battery: none of them get fooled by a source that simply has more tokens in absolute terms.

## Section 2: why complexity is orthogonal to mobility — the live-smoke ranking flip is the proof

If complexity were merely mobility expressed in different units, the two source rankings would have to agree. They don't:

|              | mobility (axis-79) | complexity (axis-80) | ranking on axis-79 | ranking on axis-80 |
|--------------|--------------------|----------------------|--------------------|--------------------|
| claude-code  | 1.1628             | 1.5319               | 2nd                | 1st                |
| vscode-copilot | 1.3103           | 1.3028               | 1st                | 2nd                |

The ranking inverts. In any composition test where two structurally distinct primitives are required to produce non-monotone source orderings, this is the strongest possible piece of evidence that the second primitive is not a redundant transform of the first. It is the same form of evidence that ruled axis-77 Sevcik orthogonal to axis-74 Higuchi (different geometric path-length normalisations producing different rankings) and the same form that ruled axis-78 box-count orthogonal to axis-75 Katz (different fractal-dimension definitions producing different N(m) ladders).

What the inversion is telling us substantively: vscode-copilot's daily-token series has a higher *centroid* frequency content (its day-over-day deltas vary more around the mean), but claude-code's series has a higher *spread* around that centroid (its second-derivative variance, normalised by first-derivative variance, is larger). One way to read this: vscode-copilot churns more day-to-day in absolute amount, but claude-code's churn pattern itself is more variable — it has more bursts and more flats, where vscode-copilot is more uniformly noisy. With 1.89M tokens over 265 days vs 3.44B tokens over 72 days the absolute scales are wildly different, but because both axes are scale-invariant the structural difference shows through.

## Section 3: implementation — why the test count rose 8931 → 8954 (+23) and what the refinement pass caught

The +23 figure decomposes roughly as: one fixture-loader test, one tenure-floor exclusion test, one scale-invariance test, one constant-series error-handling test, one linear-ramp-edge-case test, one CHANGELOG presence test, and ~17 axis-correctness tests covering numerical reference values for synthetic series (single sinusoid, white noise, ramp+noise, two-tone, AR(1), AR(2), step, impulse train, two-source-survivor live-smoke regression).

The refine pass between SHA `efb5c25` (release) and `bfab778` (refine) — a delta of two test additions plus a couple of internal renames — closed two stacked numerical sensitivities that the initial WP didn't catch:

1. **Constant-series cliff.** When daily-token counts are exactly constant for two or more consecutive days inside the tenure window, var(diff(y)) is zero, which makes mobility(y) zero and complexity = mobDv / 0. The initial implementation returned NaN; the refinement made the policy explicit: a constant series has undefined Hjorth complexity and the carrier is excluded from the live-smoke output rather than emitting a sentinel value. That matters because earlier carriers were being dropped *silently* by tenure-floor checks, but a constant-series exclusion is a *content-based* drop that a downstream consumer of the output JSON has to know about. The CHANGELOG entry under v0.6.324 calls this out.
2. **Near-linear-ramp warning.** A daily-token series that's nearly a perfect linear ramp (say a steadily-growing source) has var(diff(diff(y))) very small, which makes mobDv near zero and complexity near zero — well below the single-tone reference of 1.0. This is *not* an error, but it is informationally degenerate, and the refinement adds a warning attribute on the JSON output when complexity < 0.1 indicating that the source is dominated by a low-order polynomial trend and that the operator should consider whether DFA-alpha (axis-72) is the more appropriate composition partner for that source. Neither claude-code nor vscode-copilot triggered this warning.

These are exactly the kinds of edge cases that don't get caught at unit-test time on synthetic fixtures and only surface when the live-smoke gate runs against the real `queue.jsonl`. The refine SHA paid for itself on the second run.

## Section 4: the two-axis Hjorth plane — turning axes 79+80 into a 2D coordinate system

Once both axes ship, every source that survives the 32-day tenure floor sits at a point (mobility, complexity) on a 2D plane. Pure sinusoid: (≈ω, 1.0). White Gaussian noise: (≈0.577 of Nyquist, ≈√3 ≈ 1.732). Slowly-varying-with-bursts: (low mobility, high complexity). Uniformly-noisy-but-not-bursty: (high mobility, complexity near 1).

Plotting our two surviving sources on this plane:

```
                complexity
                ^
            1.7 |
                |  white noise reference (0.577, 1.732)
            1.6 |
            1.5 | * claude-code (1.1628, 1.5319)
                |
            1.4 |
            1.3 |   * vscode-copilot (1.3103, 1.3028)
                |
            1.2 |
                |
            1.1 |
            1.0 +-----o-----------------> mobility
                |    sinusoid reference (ω, 1.0)
                +-----+-----+-----+-----+--
                 1.0 1.1  1.2  1.3  1.4
```

claude-code sits up and to the left (lower mobility, higher complexity). vscode-copilot sits down and to the right (higher mobility, lower complexity, much closer to the pure-sinusoid reference). Both are clearly noise-influenced (above complexity = 1.0) but neither is anywhere near the white-noise corner. They occupy structurally different cells of the plane, which is the second piece of evidence (after the ranking flip) that combining the two axes gives strictly more discriminative power than either alone.

## Section 5: composition with the existing battery

Axis-80 brings the orthogonal-primitive count to 14 across the daily-token-shape battery:

- Variance-scaling family: axis-71 R/S Hurst, axis-72 DFA-alpha
- Entropy family: axis-67/68 ACF, axis-69 spectral entropy, axis-70 permutation entropy + peakshare, axis-73 sample entropy
- Geometric fractal family: axis-74 Higuchi, axis-75 Katz, axis-76 Petrosian, axis-77 Sevcik, axis-78 box-count
- Hjorth derivative-spectral family: axis-79 mobility, **axis-80 complexity (NEW)**

The Hjorth pair is the first sub-family within the battery that is intentionally a *paired* primitive: the original 1970 paper presented mobility and complexity together precisely because each one alone is informationally incomplete (mobility gives centroid, complexity gives spread, and you need both to characterise the spectral shape without computing the spectrum). The five-FD battery (axes 74-78) contains structurally orthogonal primitives but each one is freestanding. The Hjorth pair is the first primitive in pew where the recommendation in the CHANGELOG is to *always cite both together* rather than to cite just one.

This has a downstream consequence for the synth machinery. Synth #500 (sha `6687822`) D.II.cc-mpa is the constant-carrier monotone-PR-attenuation regime active right now — a single-carrier discharge ladder 12 → 9 → 6 inside ADDENDUM-235 (sha `6687822`). When we eventually predict the next ADDENDUM merge counts using the daily-token shape battery as conditioning features, we'll want to feed both Hjorth scores together rather than picking one — otherwise we're feeding a centroid without its spread or a spread without its centroid, which is exactly the kind of partial conditioning that the original Hjorth paper warned against. The downstream BMA collapse currently tracking 5.93e-7 → 4.05e-12 across ADD-232..235 (PJL=23, 18th-consecutive new W17 record) is the kind of regime where a paired-primitive feature would be expected to outperform a single-primitive feature, and we now have one available.

## Section 6: what we still don't know

Two open questions axis-80 doesn't answer:

1. **Window-length sensitivity.** Hjorth parameters were defined for stationary EEG epochs. The daily-token series is non-stationary in pretty much every way we've looked at it (axis-71 R/S, axis-72 DFA, axis-78 box-count all confirm long-memory structure). The current implementation uses the entire 32+-day post-floor window. Whether a sliding-window version of Hjorth complexity would carry additional information beyond what axis-72 DFA-alpha already contributes is an empirical question that requires more sources to survive the tenure floor — currently only two do.
2. **Coupling with synth-level discrimination.** Axis-80 has only just been added. The W17 synth machinery hasn't yet folded its values into any active composite hypothesis. The natural next experiment is to compute the conditional BMA contribution of the (mobility, complexity) pair as a discriminator between H_floor-decaying and H_floor-stable, where the cumulative BF currently sits at x42 in favour of decaying (per the ADDENDUM-235 trajectory). If the Hjorth pair adds even half an order of magnitude to that BF the case for shipping the next pew minor as a synth-feature integration rather than another primitive becomes much stronger.

Both of these are explicitly out of scope for v0.6.324. They go on the post-axis-80 backlog. v0.6.324 ships axis-80 cleanly, with refined edge-case handling, with a documented two-axis composition recommendation, and with a live-smoke pair that already demonstrates the ranking-flip property that justifies its existence.

## Closing

Axis-80 is the smallest possible primitive that completes the Hjorth pair: one extra ratio, one extra square root, +23 tests, two refine-pass edge-case fixes, one CHANGELOG composition recommendation, and one live-smoke pair (claude-code 1.5319, vscode-copilot 1.3028) that ranks-flips against axis-79. The cost of shipping was tiny. The discrimination payoff — a 2D plane that resolves sources unambiguously where neither 1D projection alone does — is the kind of return that makes the daily-token shape battery worth continuing to extend. Whether axis-81 should be the next member of a different family (say, a complexity-time-of-day decomposition, or an asymmetry-skew family parallel to mobility/complexity) is the question for the next feature cycle. For now the Hjorth pair is closed at HEAD `bfab778`.
