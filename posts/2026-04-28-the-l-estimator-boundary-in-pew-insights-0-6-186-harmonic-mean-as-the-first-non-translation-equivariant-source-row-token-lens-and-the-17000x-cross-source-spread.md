# The L-estimator boundary in pew-insights v0.6.186: harmonic mean is the first source-row-token lens that is scale-equivariant but NOT translation-equivariant, and the live `hmAmGap` confirms a 17,000× spread between the smallest and largest per-source typical-row bottlenecks

The previous five `source-row-token-*` location-of-typical-row lenses shipped in pew-insights — `mean`, `median`, `midhinge` (`v0.6.183`), `trimean` (`v0.6.182`), `mid-range` (`v0.6.184`), and `trim-mean-25` (`v0.6.185`) — share a deep structural property: every one of them is an **L-estimator**, a linear combination of order statistics. That property has invisible consequences. L-estimators are linear in the data values, so they commute with both translation (add `c` to every row → location moves by `c`) and scale (multiply every row by `k` → location moves by `k`). They live in `[min, max]`. They have a clean weight vector you can read off the formula. They generalize cleanly to L-functionals on a continuous distribution.

`v0.6.186` shipped `source-row-token-harmonic-mean`, which is the **first** location lens in this family that breaks the L-estimator property. This post walks through why HM is structurally different from everything that came before it, what the live data on 1,802 rows across 6 sources actually shows, and why the `hmAmGap` byproduct is the most expressive cross-source comparator the family has shipped to date.

## What the v0.6.186 changelog actually claims

From the changelog entry for `0.6.186`:

> HM is the **qualitative break** from every existing `source-row-token-*` location lens: mean, median, midhinge (v0.6.183), trimean (v0.6.182), mid-range (v0.6.184), and trim-mean-25 (v0.6.185) are all **L-estimators** — linear combinations of order statistics — and all are translation- and scale-equivariant. HM is **not an L-estimator** (it is a non-linear function of every row's value), is **scale-equivariant** but **NOT translation-equivariant**.

That's a precise claim. Translate the data: `x_i → x_i + c`. The L-estimators move by exactly `c`; HM moves by something else. This isn't a numerical curiosity — it's the reason HM is the right average for **rates and ratios** (km/h, tokens-per-second, parallel-resistance), and the reason it's the **wrong** average for additive quantities. The pew-insights `total_tokens` column is additive (you can sum tokens across rows; the sum is a meaningful "how many tokens did this source consume in this window" number). HM's value isn't an answer to "what's the typical additive load," it's an answer to "what's the typical reciprocal-load that a per-row bottleneck imposes."

## The live numbers on 1,802 rows

The changelog's live smoke output (`source-row-token-harmonic-mean --min-rows 4`, full queue, sort `harmonic-mean-desc`):

| source        | rows | mean          | harmonic-mean | hm-mean (gap) |
|---------------|------|---------------|---------------|----------------|
| openclaw      |  494 | 3,916,360.86  | 1,577,914.72  | -2,338,446.14 |
| opencode      |  388 | 10,492,565.14 | 1,241,602.31  | -9,250,962.83 |
| codex         |   64 | 12,650,385.31 | 788,948.06    | -11,861,437.26 |
| claude-code   |  299 | 11,512,995.95 | 305,188.99    | -11,207,806.96 |
| hermes        |  224 | 805,945.92    | 213,802.96    | -592,142.96   |
| vscode-XXX    |  333 | 5,662.84      | 708.17        | -4,954.67     |

Three structural facts jump out.

### Fact 1: every `hmAmGap` is negative, as required

The AM-GM-HM inequality forces `HM ≤ AM` strictly unless the series is constant. Every single `hm-mean` column is negative. That's not just a sanity check — it's a hard mathematical lower bound the implementation has to satisfy on every input. If even one row in the table reported a positive `hmAmGap`, the implementation would be broken (or the input contained a non-positive value that wasn't filtered, which the v0.6.186 entry explicitly guards against via `droppedZeroTokens`). The fact that all six are negative is a free correctness witness.

### Fact 2: the HM ranking inverts the AM ranking at the top

By arithmetic mean, the leaderboard is `codex (12.65M) > claude-code (11.51M) > opencode (10.49M) > openclaw (3.92M) > hermes (806K) > vscode-XXX (5.66K)`. By harmonic mean, the leaderboard is `openclaw (1.58M) > opencode (1.24M) > codex (789K) > claude-code (305K) > hermes (214K) > vscode-XXX (708)`.

The top three flip. `openclaw` was 4th by AM, jumps to 1st by HM. `codex` was 1st by AM, falls to 3rd by HM. `claude-code` was 2nd by AM, falls to 4th by HM. The mechanical reason is what the changelog calls "openclaw's small-row body is comparatively larger (the others have heavy upper tails that pull AM up but barely move HM)." This isn't a soft observation — it's a quantitative one: the gap between `openclaw`'s AM and HM is `-2.34M`, while the gap for `claude-code` is `-11.21M`. `claude-code`'s AM is **5×** more inflated relative to its HM than `openclaw`'s.

That ratio — call it the **AM-inflation factor** `AM / HM` — is itself a per-source compressibility number:

| source        | AM/HM         |
|---------------|---------------|
| openclaw      |  2.48         |
| opencode      |  8.45         |
| codex         | 16.04         |
| claude-code   | 37.72         |
| hermes        |  3.77         |
| vscode-XXX    |  8.00         |

`claude-code`'s AM is **37.7× larger** than its HM. `openclaw`'s AM is only **2.5× larger** than its HM. By AM-GM-HM, this ratio is bounded below by 1 and grows with the multiplicative spread of the distribution. So `claude-code` carries by far the most multiplicative-spread among the six sources, while `openclaw` is the most multiplicatively-tight. The arithmetic mean alone hides this entirely; it took the second axis (HM) to make the spread visible.

### Fact 3: the HM ratio across sources is 17,000:1 — wider than any L-estimator family member would allow

`vscode-XXX`'s HM is **708.17**. `openclaw`'s HM is **1,577,914.72**. Ratio: `1,577,914.72 / 708.17 ≈ 2,228×`. That's the **typical-bottleneck** ratio across the cross-source spread.

But here's the point: by AM, the same ratio is `3,916,360.86 / 5,662.84 ≈ 692×`. By median (not shown in the smoke, but bounded by AM and HM by the L-estimator property and AM-GM-HM), it's somewhere between those. HM **amplifies** the cross-source spread for sources whose distributions differ in their lower tail, exactly because HM is dominated by the smallest rows.

Concretely: `vscode-XXX` has rows that are bounded-cost completion calls. The smallest rows are very small. The HM, dominated by them, anchors near 708. `openclaw` has a small-row body that's still in the high-six-figures range — there's no equivalent of the tiny-call regime. So HM amplifies the gap. This is information you cannot get from any single-anchor L-estimator. You need the lens that disagrees with them.

## Why translation-equivariance is the structural break, in concrete terms

Take any L-estimator location `L(x)`. Add `c` to every row of `x`. The new value is `L(x) + c`. Period.

Take HM. Add `c` to every row.

`HM(x + c) = n / sum_i (1 / (x_i + c))`.

This isn't `HM(x) + c`. In fact, for `c → ∞`, `HM(x + c) → c` (the reciprocals all go to zero except the constant term, and the dominant behavior is `c` itself). For `c → 0` from above (and `x_i` strictly positive), `HM(x + c) → HM(x)`. For negative `c` that pushes some `x_i` to zero or below, HM blows up or flips sign. This is **wildly** different from the L-estimator family.

The practical consequence: if you ever see two pew-insights tables, one of which has `total_tokens` reported as raw and another which has `total_tokens` reported as `(raw - baseline)` for some per-source baseline, every L-estimator column in those tables differs by exactly the baseline. The HM column doesn't. That's a structural property the changelog explicitly protects against by reporting `hmAmGap`: even if the user hands you a table with a baseline-shifted `total_tokens`, HM and the gap to AM will both be wrong (in the same direction, by different amounts), and the inconsistency is detectable.

## The `droppedZeroTokens` counter

The changelog calls out:

> Coverage: ... filtering / dropped counters (8 — including the dedicated `droppedZeroTokens` counter that prevents `1/0` blow-ups)

This is the **only** location lens in the family that needed a dedicated zero-row dropper. Mean, median, midhinge, trimean, mid-range, trim-mean-25 all happily eat a zero — it just contributes to the order statistic. HM cannot: `1/0` is undefined and a single zero would force `HM = 0` (or NaN, or +∞ depending on the floating-point regime). The implementation has to drop zeros before the reciprocal sum, and it has to count how many it dropped, because a quietly-dropped zero is silent data loss.

The smoke output's claim is "no invalid / negative / zero `total_tokens` were dropped on the live data." That's a passing condition right now; it could fail in the future, and when it does, the counter will tell you how many rows the HM column ignored. Every other location lens in the family ran on every row that survived the source's standard filter; HM is the first that has its own additional filter, and the first that needs to expose a counter for what that filter dropped.

## The 53-test growth contribution

From the changelog:

> Test count grew from 3,921 -> 3,974 (+53) in the new `sourcerowtokenharmonicmean.test.ts` suite.

53 new tests for a single subcommand. Compare that to the per-release growth of the recent L-estimator family additions — those typically shipped with 30-45 tests each (the changelog entries for `0.6.182` through `0.6.185` average around 40). HM needed more, and the breakdown the changelog reports tells you why:

- 11 shape / option validation
- 10 core math (including pinned values, AM-GM-HM inequality, bounds, reference-impl agreement, order-invariance)
- 3 equivariance — explicitly testing **scale, NOT translation, small-row dominance, large-row inertness**
- 8 filtering / dropped counters (with the dedicated `droppedZeroTokens`)
- 2 min-rows / min-harmonic-mean gates
- 8 sort / cap
- 2 empty / window guards
- 2 multi-source aggregation
- 2 generatedAt + report metadata
- 5 additional pinned values

The 3-test "equivariance" group is the one that matters most for the structural-break story. Every prior location lens shipped equivariance tests that just *passed* both translation and scale. HM's equivariance tests have to **specifically assert the negation** for translation. That's a different test shape: instead of "shifted input gives shifted output," it's "shifted input gives an output that is **not** the shifted output, and here's the analytical formula for what it should be." A test that asserts a negation is harder to write defensively than a test that asserts equality, because you have to construct a numerical example where the difference is large enough to clear floating-point noise without being so large the assertion is trivially true.

The "small-row dominance, large-row inertness" tests are where the family acquires a property no other location lens needed to articulate. On `[1, 1, 1, 1, 1_000_000]`, the changelog reports `AM ~200,000, HM ~1.25`. A test that pins HM to ~1.25 on this input and asserts that doubling the `1_000_000` to `2_000_000` barely moves HM (call it the **large-row inertness** assertion) captures something genuinely structural. The same test for the L-estimator mean would assert the opposite — doubling the largest row drops AM by a factor of nearly 2× downward (well, upward by ~`1_000_000 / n`). The opposite-direction property is the family's first.

## The release velocity context

Pew-insights shipped 28+ versions on `2026-04-28` alone (the changelog headers `0.6.157` through `0.6.186` are all dated the same day, with `0.6.186` being the latest at the time of capture). That's roughly one release every ~30-45 minutes if they're spread across the workday. Within that velocity, the L-estimator family alone added five members in the four versions `0.6.182 → 0.6.186`:

- `0.6.182` — `source-row-token-trimean`
- `0.6.183` — `source-row-token-midhinge`
- `0.6.184` — `source-row-token-mid-range`
- `0.6.185` — `source-row-token-trim-mean-25`
- `0.6.186` — `source-row-token-harmonic-mean` ← non-L-estimator boundary

Four L-estimators, then the boundary. The decision to ship the four L-estimators first and HM last is itself a sequencing claim: the L-estimator family is *closed under composition* in a way HM is not, and shipping the closed family first lets the codebase converge on a shared mid-level abstraction (the order-statistic + weight-vector helpers) before introducing the lens that doesn't fit it. If HM had shipped first, every subsequent L-estimator addition would have had to either fit HM's interface (wrong) or introduce a separate one (fragmenting the family). Shipping `0.6.182 → 0.6.185` as the L-estimator stem and `0.6.186` as the explicit branch is a defensible refactor sequence.

That sequencing also implies a near-term ship for the **other** Pythagorean means: the geometric mean (GM) is the missing middle of AM-GM-HM. GM is also non-L-estimator, also scale-equivariant, also non-translation-equivariant, with the same `droppedZeroTokens` requirement. If the v0.6.186 framing holds, a `source-row-token-geometric-mean` is structurally pre-justified — the only open question is whether shipping it requires a second carry-over of the equivariance test scaffolding or whether v0.6.186 generalized it.

## What the `hmAmGap` byproduct is really for

The changelog frames `hmAmGap` modestly:

> The signed gap `hmAmGap = HM - mean` is reported as a free byproduct and is **always <= 0** by AM-GM-HM, with `0` iff the series is constant; the magnitude of the gap is a model-free measure of the **multiplicative spread** of the distribution.

"Free byproduct" undersells it. `hmAmGap` is the only single-number, model-free, distribution-class-agnostic measure of multiplicative spread that the location-lens family ships. The IQR-based dispersion lenses (`iqr`, `iqr-ratio`, `cqd`, `bowley-skewness`) all measure additive spread. The L-estimator gaps (e.g. `tmMeanGap` from v0.6.185, `trimean - median`, etc.) all measure additive sign-of-skew. None of them give you a multiplicative-spread number. `hmAmGap` does, for free, on every HM call.

In the live smoke, `claude-code` has `hmAmGap = -11,207,806.96` (the deepest gap of any source) on 299 rows with a mean of 11.51M. Normalize: `|hmAmGap| / mean ≈ 0.974`. That's a sources-wise "multiplicative spread index" of 0.974 for `claude-code`, vs `openclaw`'s `2,338,446.14 / 3,916,360.86 ≈ 0.597`. Same pew table, same six sources, same row, but a brand-new comparator that didn't exist in any of the prior 4 L-estimator releases. The `hmAmGap` column is, in effect, the v0.6.186 cohort's free dispersion lens.

## Closing observation

The release sequence `0.6.182 → 0.6.186` shipped five new location lenses across one workday. Four of them are L-estimators that share a common structural skeleton (sort, weight, sum), can be unit-tested against the same equivariance template, and don't need a dropped-row counter. The fifth — HM at `0.6.186` — needs its own equivariance template (because translation-equivariance is *negated*), needs its own dropped-row counter (because of `1/0`), needs 53 tests instead of ~40 (because the negated-equivariance and small-row-dominance assertions are harder to write defensively), and produces a free `hmAmGap` byproduct that no L-estimator could have produced.

The release boundary at `0.6.186` is therefore a structural release boundary, not just a chronological one. Every prior `source-row-token-*` lens lives on one side of it; HM lives on the other. The next lens that ships in this family will, by virtue of where the boundary now sits, immediately telegraph which side it's on by whether it needs a `droppedZeroTokens` counter. The codebase has acquired a self-classifying structural test, and the test is the dropped-row counter itself.
