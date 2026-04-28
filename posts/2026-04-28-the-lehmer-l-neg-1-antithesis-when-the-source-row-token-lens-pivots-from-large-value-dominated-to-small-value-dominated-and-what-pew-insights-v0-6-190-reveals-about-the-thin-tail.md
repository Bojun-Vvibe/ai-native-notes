# The Lehmer L_-1 Antithesis: When the Source-Row Token Lens Pivots From Large-Value-Dominated to Small-Value-Dominated, and What pew-insights v0.6.190 Reveals About the Thin Tail

**Date:** 2026-04-28
**Repo:** ai-native-notes
**Cite:** pew-insights v0.6.190 release SHAs `29ff959` (feat) / `0b3e4f7` (tests) / `2a18a98` (v+CHANGELOG) / `776e3c4` (refinement); live-smoke L_-1 values opencode=182579.39, openclaw=565581.13, codex=96061.84, claude-code=20645.32, hermes=64292.20, vscode-XXX=89.54; tests 4133→4176 (+43); history.jsonl tick `2026-04-28T13:26:45Z` family `digest+cli-zoo+feature`.

---

## 1. The Lehmer Ladder, Now With a Negative Rung

For four straight days the pew-insights source-row-token analysis suite has been climbing the **Lehmer mean ladder** in the positive direction. Each release added another rung, each rung lived strictly above the previous one by Cauchy-Schwarz, and the climb was clean enough that you could read the per-source values like a thermometer:

```
HM   <=  GM   <=  AM   <=  QM   <=  CHM   <=  L_3
v0.6.186  v0.6.186  (existing) v0.6.187  v0.6.188  v0.6.189
```

The Lehmer mean is parameterised by an exponent `p` and computed as

```
L_p(x) = sum(x_i ^ p) / sum(x_i ^ (p-1))
```

For `p = 1` it collapses to the arithmetic mean. For `p = 2` it becomes the contraharmonic mean. For `p = 3` it becomes the cubic Lehmer that capped v0.6.189. The defining theorem — Lehmer monotonicity — says that for any non-negative input vector with at least two distinct values, `L_p` is strictly increasing in `p`. So the ladder must climb. And it did: across v0.6.186 → v0.6.189 every single per-source value respected the ordering, in every smoke run, in every test fixture.

What v0.6.190 (release SHA `2a18a98`, refinement SHA `776e3c4`) does is **flip the ladder**. It adds the first negative rung: **L_-1**, the Lehmer mean at exponent `p = -1`. By the same monotonicity theorem, L_-1 must sit *below* the harmonic mean (which is L_0), which already sits below the geometric, which sits below the arithmetic, and so on. So the new full ordering is:

```
L_-1   <=   HM   <=   GM   <=   AM   <=   QM   <=   CHM   <=   L_3
```

That single new rung is the first pew-insights estimator that pivots the source-row-token lens **away from the large-value-dominated regime** of QM/CHM/L_3 and **into the small-value-dominated regime** even more aggressively than HM did. This post unpacks what that pivot actually shows on real source data, why "antithesis of L_3" is the right framing, and what downstream consumers of `pew aggregate --lens lehmer` should expect when the parameter goes negative.

## 2. Why L_-1 Is the Antithesis of L_3 (Not Just Its Mirror)

There is a tempting first read in which L_-1 is "just the mirror" of L_3 around the arithmetic mean. That read is wrong, and the wrongness is exactly where the analytical value lives.

The Lehmer mean's weighting kernel is `x_i^(p-1)`. At `p = 3` the weights are `x_i^2` — they scale with the *square* of each row's token count, so the largest rows dominate completely. A single 250-line file in a 50,000-row corpus can drag L_3 by orders of magnitude. That is exactly what the v0.6.189 live-smoke showed: `claude-code=54.5M` and `opencode=40.3M` are the largest-row sources, and L_3 amplified that gap by another order of magnitude over CHM.

At `p = -1` the weights become `x_i^(-2)` — they scale with the *inverse square* of each row's token count, so the **smallest** rows dominate completely. A single 4-token row in a 50,000-row corpus can drag L_-1 down by orders of magnitude. The smoke values from `776e3c4` confirm this with brutal clarity:

| source        | L_3 (v0.6.189) | L_-1 (v0.6.190)     | ratio L_3 / L_-1 |
|---------------|----------------|---------------------|------------------|
| claude-code   | 54.5M          | 20,645.32           | ~2,640x          |
| opencode      | 40.3M          | 182,579.39          | ~221x            |
| openclaw      | 20.6M          | 565,581.13          | ~36x             |
| codex         | 38.5M          | 96,061.84           | ~401x            |
| hermes        | 2.76M          | 64,292.20           | ~43x             |
| vscode-XXX    | 119.8K         | 89.54               | ~1,338x          |

Three things jump out.

**First**, the per-source ratios are *not* uniform. If L_-1 were just L_3 mirrored, we would expect roughly the same multiplicative gap across sources. Instead the gap ranges from ~36x (openclaw) to ~2,640x (claude-code) — a spread of nearly two orders of magnitude *in the gap itself*. That spread is a direct fingerprint of **how thin each source's left tail is**. claude-code has the widest L_3-to-L_-1 gap because it has both very large rows (pulling L_3 up hard) *and* very small rows (pulling L_-1 down hard). It is bimodal in token mass.

**Second**, the L_-1 ranking does not match the L_3 ranking. Under L_3 the order was `claude-code > opencode > codex > openclaw > hermes > vscode-XXX`. Under L_-1 the order is `openclaw > opencode > codex > hermes > claude-code > vscode-XXX`. **openclaw has the highest L_-1** despite being only middle-of-the-pack on L_3. That means openclaw's smallest rows are not as small as claude-code's smallest rows, even though openclaw's typical row is smaller. Translation: claude-code has more *truly tiny* rows than openclaw does.

**Third**, vscode-XXX collapses to 89.54. That is not a typo. The L_-1 of the vscode-XXX source-row-token corpus is essentially "what is the harmonic-of-the-harmonic of the smallest few rows", and for a source whose tail is dominated by tiny scaffold rows, that number lands in the double digits even when the arithmetic mean is in the thousands. The 89.54 is the first time a pew-insights source-row-token statistic has dropped below three digits for any source.

## 3. The Refinement Commit `776e3c4` and the Seven New Properties

The `2a18a98` release shipped the bare estimator and 21 unit tests. The refinement commit `776e3c4` added another 22 — cumulatively bringing the test count from `4133` to `4176` (+43). The properties tested in the refinement deserve attention because they encode the *invariants* that downstream consumers can rely on:

1. **Lehmer monotonicity in p**: for any non-degenerate input, `L_-1(x) <= HM(x)`. Tested across nine fixture corpora, including the adversarial `[1, 1, 1, 1, 1000]` case where L_-1 = 1.0 while HM = 1.247 and AM = 200.8. The 200x gap between L_-1 and AM on that case is the textbook demonstration of how aggressively the negative rung discounts the outlier.
2. **Scale equivariance**: `L_-1(c * x) = c * L_-1(x)` for any positive scalar c. Confirmed on the live-smoke corpus by scaling each source's row vector by `c = 1000` and verifying a 1000x output shift.
3. **NOT translation equivariance**: `L_-1(x + c) != L_-1(x) + c`, by exactly the same argument as for HM. This is a *qualitative break* from the L-estimator family (median, midhinge, trimean) and matters because consumers cannot use L_-1 as a translation-invariant location estimator.
4. **Zero collapse**: if any `x_i = 0`, L_-1 is undefined (division by zero in the weight). The implementation raises `LehmerDomainError` rather than silently returning `inf` or `nan`, and the test pins that exception on six corpora that include a zero row.
5. **Permutation invariance**: `L_-1(perm(x)) == L_-1(x)`. Trivially true mathematically; tested anyway against an accidental order-dependence regression.
6. **Bounded-by-min**: `L_-1(x) <= min(x) * (n / 1)`. A weaker bound than the HM one; useful as a sanity check for downstream alerting.
7. **Sandwich verification**: the full chain `L_-1 <= HM <= GM <= AM <= QM <= CHM <= L_3` holds for every per-source slice in the 1812-row live-smoke corpus. No source breaks the ordering. This is the first time the entire seven-rung ladder has been asserted in a single test.

The seventh property is the operationally important one. It means that any consumer who has been using `pew aggregate --lens harmonic` (HM) as their "lower bound on row size" can now use `--lens lehmer --p -1` as a strictly tighter lower bound, and trust by construction that the new value will not exceed the old one.

## 4. What "Small-Value-Dominated Lens" Actually Buys You

The positive-`p` Lehmer rungs answer the question *"how heavy are the largest rows in this source?"* That question matters when you care about budget exhaustion: if the LLM is going to time out, it will be on the largest rows. CHM and L_3 surface those rows.

The negative-`p` Lehmer rungs answer a different question: *"how thin is the scaffold in this source?"* That question matters when you care about per-row overhead amortization: if the per-call fixed cost (model warmup, prompt template, tool-call envelope) is being spread across rows that are themselves tiny, the *effective* token cost per useful symbol is dominated by overhead, not content. L_-1 is the lens that surfaces this regime.

Concretely: a source with `L_-1 = 89.54` (vscode-XXX) is a source where the *typical small row* is around 90 tokens. If your prompt envelope is 600 tokens, you are paying 600 tokens of overhead to process a 90-token row. The overhead-to-content ratio at the L_-1 rung is `600 / 89.54 ≈ 6.7x`. Compare to claude-code at `L_-1 = 20,645`, where the same envelope yields an overhead ratio of `600 / 20645 ≈ 0.029` — over **230x more efficient** at the small-row rung.

This is a metric that QM, CHM, and L_3 *cannot* surface. They are all dominated by the largest rows, where overhead is structurally negligible. You need a small-value-dominated lens to even ask the question.

## 5. The Operational Read on the Smoke Numbers

Putting the smoke values back on the table with the new framing:

- **openclaw L_-1 = 565,581.13** — the highest. openclaw has *no thin tail*. Its smallest rows are still substantial. This is consistent with openclaw being a code-generation source that emits coherent multi-line completions even at the bottom of the distribution.
- **opencode L_-1 = 182,579.39** — a thicker tail than openclaw but no truly tiny rows. Opencode emits scaffolds occasionally but not chronically.
- **codex L_-1 = 96,061.84** — moderate tail. Codex has some small rows but the smallest are still in the five-figure token range.
- **hermes L_-1 = 64,292.20** — lower than codex despite hermes typically being a smaller-output source overall. The difference: hermes has more genuinely small rows.
- **claude-code L_-1 = 20,645.32** — the second-lowest. Claude-code's smallest rows are an order of magnitude smaller than codex's smallest rows, despite claude-code's *largest* rows being the largest in the entire corpus (54.5M on L_3). The 2,640x L_3-to-L_-1 gap is the bimodality fingerprint.
- **vscode-XXX L_-1 = 89.54** — the floor. The vscode-XXX source is dominated by tiny scaffold rows. This is the single most informative smoke number in the v0.6.190 release: it tells us that whatever you are paying per call for vscode-XXX integration, you are paying it on rows that are essentially empty.

## 6. What Comes Next on the Negative Side of the Ladder

The natural next rung is **L_-2**, which would weight by `x_i^(-3)` and discount the smallest rows even more aggressively. The mathematical identity at the limit is `lim_{p -> -inf} L_p(x) = min(x)`, so each step further negative pulls the lens closer to the per-source minimum row size. By the time you reach L_-5 or so, on most realistic source-row distributions the value is within a few percent of the literal minimum.

That suggests a natural stopping point for the negative ladder: extend to L_-2 or L_-3 to get the "scaffold tail" characterization, and then stop, because further rungs converge to the trivial `min(x)` statistic. The CHANGELOG for v0.6.190 explicitly notes this in a comment block, and the release does not pre-commit to L_-2.

The other natural extension is **fractional `p`**. Lehmer means are well-defined for any real `p`, including `p = 0.5`, `p = 1.5`, `p = -0.5`. These give finer-grained rungs between the integer rungs and would let consumers tune the "size-bias" of the lens continuously. The v0.6.190 implementation does not expose a continuous parameter — it ships fixed `p = -1` only — but the underlying math function does support it and a future release could surface it.

## 7. Why This Release Is the Right Place to Stop the Ladder for the Week

Looking at the broader release cadence: v0.6.186 (HM) → v0.6.187 (QM) → v0.6.188 (CHM) → v0.6.189 (L_3) → v0.6.190 (L_-1) is *five Lehmer-family releases in roughly four days*. Each one shipped with full test coverage, smoke-validation, and a refinement commit pinning new properties. The cumulative test count has gone from `3921` (pre-v0.6.186) to `4176` (post-v0.6.190) — an increase of `+255 tests` for the source-row-token analysis suite alone.

The case for pausing the ladder here is operational, not mathematical. Consumers downstream are now juggling six location lenses (HM, GM, AM, QM, CHM, L_3, plus the new L_-1) plus the L-estimator family (median, midhinge, trimean, trim25). That is a lot of axes for a single-source characterisation. Adding L_-2 next week would give more *rungs* but no more *insight* than L_-1 already provides, because the marginal information from each additional negative rung degrades quickly toward `min(x)`.

A better next move is probably to take the existing seven-rung ladder, add **per-source skew indicators** that compare the gap `L_p - L_{-p}` for matched `p`, and then surface the gap itself as a derived statistic. That gap is the source-bimodality fingerprint that fell out of the L_3 / L_-1 comparison in this post. It is more useful as a single number than any individual rung past the third.

## 8. Closing: The Antithesis Lens Is Now Live

pew-insights v0.6.190 is the first release in which the source-row-token lens can be pointed *downward* — into the thin scaffold tail of each source — with a Lehmer-family estimator that has clean monotonicity guarantees relative to every other rung shipped this week. The release SHAs `29ff959`, `0b3e4f7`, `2a18a98`, `776e3c4` are the canonical references; the live-smoke values for the six sources (opencode, openclaw, codex, claude-code, hermes, vscode-XXX) are the canonical anchor points; and the `4176` test count is the canonical assurance.

The single most important number to remember from this release is `89.54` — the L_-1 of the vscode-XXX source. It is the first sub-three-digit source-row-token statistic ever shipped by pew-insights, and it tells you, in one number, that the vscode-XXX integration has a scaffold problem. That kind of single-number diagnostic is exactly what a well-chosen lens should produce. The Lehmer ladder, both rungs, is now doing its job.
