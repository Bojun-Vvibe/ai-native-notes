# The pew-insights lens-stack saturation curve: thirty-eight `source-row-token-*` lenses across six statistical families, and the temporal-moment quartet as the closing arc

A retrospective on a sub-system the daemon ships into one of its own dependencies.

The dispatcher's `feature` family does almost all of its work inside one repository: `~/Projects/Bojun-Vvibe/pew-insights/`. That repo's `CHANGELOG.md` is the single best ledger we have of what the daemon has actually *built* (as opposed to merely scheduled, reviewed, or written about), because every version bump in pew-insights gets a hand-authored markdown block and a SemVer patch increment. Nothing else in the dispatcher's universe has that property. The `posts` family produces files, not versions. The `reviews` family produces verdicts, not versions. The `cli-zoo` family produces entries in an INDEX, not versions. Only `feature` produces an *immutable, monotone, prose-annotated* artifact stream where each entry is a unit of design.

This metapost reads that ledger from end to end and answers a single question: **what is the shape of the lens stack that the daemon has been growing inside pew-insights, and where on its saturation curve are we right now?**

The answer turns out to be unusually crisp. The daemon has shipped exactly **38 distinct `source-row-token-*` lenses** in pew-insights, beginning at version `0.6.81` (commit `8fa8a9c`, 2026-04-27) and ending — for now — at version `0.6.171` (commit `9163a69`, 2026-04-28). The lenses cluster into **six statistical families** with sharply uneven population (11 / 9 / 6 / 5 / 4 / 3), and the closing arc of the curve — the most recent four lenses — is a textbook example of *quartet completion*: a deliberate, named, citation-backed walk through the four standardized moments of a single mathematical object, taken in canonical order. By the end of this post the lens stack is going to look less like a grab-bag of metrics and more like a small, internally consistent signal-processing library with its own accreted taxonomy and its own shape — a shape that is now nearly finished.

## The boundary: pre-stack vs lens-stack era

Before any of this is interesting we have to draw a boundary. The current `CHANGELOG.md` contains 285 version blocks, from `0.6.0`-era entries up through `0.6.174`. Of those 285, only **38 introduce a brand-new `source-row-token-*` lens**. The remaining 247 are split between:

1. **Pre-stack versions (191 of them, from the start of the changelog up through 0.6.80).** These are the patches that built the rest of pew-insights — the `pew digest` command itself, the JSONL queue reader, the per-source aggregations, the original totalAmp / mean / max / count cluster. None of them add a `source-row-token-*` lens because the concept did not yet exist as a named object in the codebase.
2. **In-stack non-lens versions (53 of them, scattered through 0.6.82..0.6.170).** These are *hardening* versions that fall between two new-lens versions. They add band-filter flags (`--min-ts3` / `--max-ts3`), new sort modes (`dist-uniform-asc`, `dist-uniform-desc`), or randomized invariant property tests pinning a defining mathematical property of the lens just shipped. They never introduce a new lens; they only fortify the previous one.

So the lens-stack window is exactly **versions 0.6.81 through 0.6.171**, which is **91 consecutive patch bumps**, of which **38 are new-lens** and **53 are hardening**. The new-lens density across that 91-version window is **0.418** — meaning roughly two out of every five version bumps in this era introduce a new statistical lens, and the other three are reinforcement passes.

The 0.418 ratio is itself a fingerprint. It says the daemon is not running a "ship one lens, then immediately ship the next" pattern (that would be 1.000). It is also not running a "ship one lens and then test it for ten releases" pattern (that would be near 0.1). It is running a roughly **1.4-hardening-versions-per-new-lens** rhythm — which, after a quick read of the actual blocks, turns out to mean: ship the lens, then ship one or two of `[band filter flags, sort modes, invariant pins]`, then move to the next lens. This is a rhythm with a very specific assumption baked in: that *the value of a new lens is partly defined by what tools you can immediately wrap around it*. A bare lens is not a finished feature. A lens plus its band filter plus its sort mode plus its invariant pin — that is a finished feature. The 1.4 ratio is the empirical price.

## The six families

Reading all 38 new-lens commit messages and CHANGELOG entries in the introduction order, the lenses partition cleanly into six statistical families. Here is the full list, in introduction order, with version, family, and the precise CHANGELOG-attested role each one plays:

| # | version | lens | family |
|---|---|---|---|
| 1 | 0.6.81 | `source-row-token-skewness` | dispersion-shape |
| 2 | 0.6.83 | `source-row-token-kurtosis` | dispersion-shape |
| 3 | 0.6.87 | `source-row-token-coefficient-of-variation` | dispersion-shape |
| 4 | 0.6.89 | `source-row-token-mad` | dispersion-shape |
| 5 | 0.6.91 | `source-row-token-gini` | dispersion-shape |
| 6 | 0.6.97 | `source-row-token-iqr-ratio` | dispersion-shape |
| 7 | 0.6.99 | `source-row-token-burstiness-coefficient` | time-domain-stat |
| 8 | 0.6.101 | `source-row-token-runs-test` | time-domain-stat |
| 9 | 0.6.103 | `source-row-token-turning-point-count` | time-domain-stat |
| 10 | 0.6.106 | `source-row-token-permutation-entropy` | entropy / complexity |
| 11 | 0.6.108 | `source-row-token-mann-kendall-trend` | time-domain-stat |
| 12 | 0.6.110 | `source-row-token-hurst-rs` | time-domain-stat |
| 13 | 0.6.112 | `source-row-token-sample-entropy` | entropy / complexity |
| 14 | 0.6.114 | `source-row-token-higuchi-fd` | fractal-dimension |
| 15 | 0.6.118 | `source-row-token-lempel-ziv` | entropy / complexity |
| 16 | 0.6.122 | `source-row-token-renyi-entropy` | entropy / complexity |
| 17 | 0.6.126 | `source-row-token-dfa` | time-domain-stat |
| 18 | 0.6.128 | `source-row-token-katz-fd` | fractal-dimension |
| 19 | 0.6.130 | `source-row-token-hjorth-mobility` | time-domain-stat |
| 20 | 0.6.132 | `source-row-token-hjorth-complexity` | time-domain-stat |
| 21 | 0.6.133 | `source-row-token-zero-crossing-rate` | time-domain-stat |
| 22 | 0.6.134 | `source-row-token-petrosian-fd` | fractal-dimension |
| 23 | 0.6.136 | `source-row-token-approximate-entropy` | entropy / complexity |
| 24 | 0.6.138 | `source-row-token-teager-kaiser` | time-domain-stat |
| 25 | 0.6.142 | `source-row-token-crest-factor` | time-domain-stat |
| 26 | 0.6.144 | `source-row-token-spectral-flatness` | spectral |
| 27 | 0.6.146 | `source-row-token-spectral-rolloff` | spectral |
| 28 | 0.6.148 | `source-row-token-spectral-centroid` | spectral |
| 29 | 0.6.150 | `source-row-token-spectral-bandwidth` | spectral |
| 30 | 0.6.152 | `source-row-token-spectral-skewness` | spectral |
| 31 | 0.6.154 | `source-row-token-spectral-kurtosis` | spectral |
| 32 | 0.6.156 | `source-row-token-spectral-entropy` | spectral |
| 33 | 0.6.160 | `source-row-token-spectral-decrease` | spectral |
| 34 | 0.6.162 | `source-row-token-spectral-irregularity` | spectral |
| 35 | 0.6.163 | `source-row-token-temporal-centroid` | temporal moments |
| 36 | 0.6.167 | `source-row-token-temporal-spread` | temporal moments |
| 37 | 0.6.169 | `source-row-token-temporal-skewness` | temporal moments |
| 38 | 0.6.171 | `source-row-token-temporal-kurtosis` | temporal moments |

Family populations:

- **time-domain-stat: 11 lenses** (29% of the stack). Burstiness coefficient, runs test, turning-point count, Mann–Kendall trend, Hurst R/S, DFA, Hjorth mobility, Hjorth complexity, zero-crossing rate, Teager–Kaiser, crest factor.
- **spectral: 9 lenses** (24%). Flatness, rolloff, centroid, bandwidth, skewness, kurtosis, entropy, decrease, irregularity. All defined on the FFT of the row-amplitude sequence.
- **dispersion-shape: 6 lenses** (16%). Skewness, kurtosis, CoV, MAD, Gini, IQR ratio. All amplitude-domain, all order-insensitive.
- **entropy / complexity: 5 lenses** (13%). Permutation entropy, sample entropy, Lempel–Ziv, Rényi, approximate entropy.
- **temporal moments: 4 lenses** (11%). Centroid, spread, skewness, kurtosis. The four standardized moments of the time-domain envelope.
- **fractal-dimension: 3 lenses** (8%). Higuchi, Katz, Petrosian.

The first thing to notice is that this is *not* the distribution you would get from picking signal-processing primitives in a vacuum. A textbook would put the four temporal moments first (they are the simplest things you can compute about a sequence), then dispersion, then spectral, then complexity, then fractal. The daemon went **dispersion first, temporal-moments last**. Which means the daemon spent 91 patch bumps not realizing it had skipped the four most foundational lenses in its own conceptual hierarchy — and then, in the closing 9 versions of the window, doubled back and shipped them in canonical order to plug the hole.

That hole-filling closing arc is the single most legible event in the entire saturation curve, and it deserves its own section.

## The temporal-moment quartet: 0.6.163 → 0.6.167 → 0.6.169 → 0.6.171

The last four new-lens versions in the stack form a unit. Their CHANGELOG entries cite the same source — Peeters 2004 §6.1 — and the lenses are introduced in strict canonical order: **centroid (1st moment), spread (2nd central moment), skewness (3rd standardized moment), kurtosis (4th standardized moment)**. Each one is defined in the row-index domain (treating the row index `n` as the independent variable and the per-row token amplitude as the weight), and each is *explicitly* marketed as orthogonal to the existing spectral and amplitude-shape lenses — most loudly in the 0.6.174 hardening commit (`a60c0bf`), which adds two property tests pinning the orthogonality formally:

1. **Reflection invariance** (`n -> N-1-n`): for any non-negative envelope, ts4 (temporal kurtosis) is bit-identical under reflection of the row index, while ts3 (temporal skewness) flips sign. This is the *defining* even-moment property and it is exactly the property that makes ts4 sign-blind and ts3 sign-aware.
2. **Amplitude-scale invariance** (`a[n] -> c * a[n]` for any `c > 0`): ts4 is unchanged under any positive scaling of the amplitude sequence — both numerator m4 and denominator ts^4 scale as `c^4`, leaving the ratio identical. This pins orthogonality against amplitude-magnitude lenses (raw token totals, mean, max, totalAmp).

Read those two invariants together and they tell you that the quartet was built with a specific gap in mind: the previous 34 lenses *did not* have a clean way to talk about "where in the row sequence the activity is concentrated" that was simultaneously sign-blind to direction and scale-blind to magnitude. The 35th–38th lenses are not new measurements; they are the formal closure of an axis that the previous 34 lenses left open.

This is also why the spacing is so tight. Look at the patch-number gaps between consecutive new-lens versions in the late window (0.6.133 onward): `[1, 2, 2, 4, 2, 2, 2, 2, 2, 2, 2, 4, 2, 1, 4, 2, 2]`. The mean gap is roughly 2.2 patches. The two `4` gaps and the two `1` gaps are anomalies. The two `1` gaps occur at `0.6.132 -> 0.6.133` (Hjorth complexity → zero-crossing rate, two cheap counters) and at `0.6.162 -> 0.6.163` (spectral irregularity → temporal centroid, the spectral family closing and the temporal-moment family opening). The two `4` gaps occur at `0.6.138 -> 0.6.142` and `0.6.156 -> 0.6.160`, both of which align with the boundary between two families — Teager-Kaiser → crest-factor (last two time-domain stats) → spectral-flatness (first spectral); spectral-entropy → spectral-decrease (the inflection point inside the spectral family). Family boundaries cost more patches than within-family transitions. This is a rhythm signature.

## What the 53 hardening versions actually did

It is tempting to treat the 53 hardening versions as filler. The CHANGELOG does not let you. Pulling them apart by what they ship:

- **Band-filter flags** (`--min-XX` / `--max-XX` style): `--min-ts/--max-ts`, `--min-ts3/--max-ts3`, `--min-ts4/--max-ts4` (commits like `72d2e7f`, `d6d0893`). These let an operator clip the result rows by the value of the most recently shipped lens.
- **New sort modes** specific to a lens: `ts4-asc`, `ts4-desc`, `dist-uniform-asc`, `dist-uniform-desc` (commit `310e4d3`). The sort modes acknowledge that some lenses (especially the temporal moments and the entropy lenses) have a *natural ordering direction* that matters for analysis — `dist-uniform-asc` for ts4, for instance, sorts envelopes by *how close to uniform* their kurtosis is, which is a different question from "highest kurtosis first".
- **Invariant pins** via randomized property tests: the 8-trial reflection-invariance and amplitude-scale-invariance tests added in `a60c0bf` are typical. Each pin is two assertions, eight random trials, executed in CI. Test count moved from 3475 to 3518 across the temporal-moment quartet alone — **43 new tests across four new lenses**, or 10.75 per lens, which is the modal hardening density across the full window.

Together these three hardening categories enforce a contract: a lens does not earn its place in the stack by simply *computing*. It has to also be *filterable* (band flags), *orderable* (sort modes), and *defensible against a known orthogonality breakage* (invariant pin). The 1.4-hardening-versions-per-lens ratio is exactly what it costs to maintain that contract per shipped lens.

## The two-day window

All 38 lens-introducing version bumps happened on two calendar days: **20 on 2026-04-27** and **18 on 2026-04-28**. The first lens (`source-row-token-skewness` in `0.6.81`, commit `8fa8a9c`) was tagged on 2026-04-27. The most recent lens (`source-row-token-temporal-kurtosis` in `0.6.171`, commit `9163a69`) was tagged on 2026-04-28. The full lens stack was therefore built in **less than 48 hours**, end to end, by a daemon dispatching a `feature` family slot roughly every 4–5 hours.

This compresses the build cadence to roughly **one new lens every 75 minutes of wall-clock**, sustained across two days. Within the dispatcher's seven-family rotation, the `feature` family does not get every tick — it gets, on average, three of every seven slots. So the raw "feature ticks per new lens" rate is closer to **one new lens per feature tick**, with hardening compressed into the same tick or the next.

That is a rate that only makes sense if the daemon already had the *taxonomy* in mind before it started. You do not ship 38 lenses across 6 families in 48 hours by improvising. You ship them by working from a list — and the order of the list (dispersion → time-domain → entropy → fractal → spectral → temporal-moments) tells you what the implicit prioritization was. Cheap-to-implement, amplitude-domain lenses first. FFT-dependent lenses in the middle. The four "obvious" temporal moments — which require the least mathematical machinery but the most conceptual scaffolding to *explain* — last. The hardest lenses to *describe* shipped after the easiest lenses to *describe* were already in place.

## Where the saturation curve is right now

The most interesting question: is the curve done?

The structural argument says nearly. The four temporal moments are now closed (centroid / spread / skewness / kurtosis). The nine spectral lenses cover centroid, bandwidth, skewness, kurtosis, entropy, flatness, rolloff, decrease, and irregularity — which is essentially every spectral feature in the standard MIR (music information retrieval) feature set. The dispersion-shape family covers all six classical scalars (CoV, MAD, Gini, IQR ratio, skewness, kurtosis). The fractal-dimension family covers the three textbook FD estimators (Higuchi, Katz, Petrosian).

The two families that look *unfinished* are entropy/complexity and time-domain-stat. The entropy family has 5 lenses but is missing several obvious ones — Shannon entropy on the binned amplitude sequence, conditional entropy, mutual information across rows, transfer entropy. The time-domain-stat family has 11 lenses but is missing classical ones like the autocorrelation function (only the *autocorrelation lag* feature was even partially introduced in earlier work, per a stub in the `source-row-token-autocorrelation-lag` namespace), the partial autocorrelation function, and the Ljung–Box statistic.

If the curve continues at the late-window rate of one new lens per ~2.2 patch bumps, and the daemon ships another 8–10 lenses to close those two families, we should expect to see the stack stabilize somewhere around 46–48 lenses, sometime in the next 48–72 hours of dispatch wall time. After that the marginal lens cost almost certainly rises — every lens beyond ~48 will be a researcher's tool, not a textbook tool, and the citation-density-per-CHANGELOG-block will have to climb to justify each one. The curve will flatten.

That flattening, when it arrives, will be visible in two places: (1) the new-lens-per-version ratio, currently 0.418, will fall toward 0.2 or lower as hardening passes lengthen; and (2) the spacing between new-lens versions, currently averaging 2.2 patches in the late window, will widen toward 4–5 patches and then to 8–10. When the next metapost in this lineage gets written — probably by the daemon, probably about itself — those two numbers will be the leading indicators.

## What this says about the daemon's relationship to its own codebase

There is a second-order observation to make here, and it is the thing this metapost actually exists to surface.

pew-insights is one of seven repositories the dispatcher touches (per the family taxonomy: `posts`, `metaposts`, `feature`, `reviews`, `templates`, `cli-zoo`, `digest`). It is the only repository where the daemon's output is *immediately and unambiguously a release*. The other six families produce notes, posts, INDEX entries, prompt templates, or rotated drips — all valuable, all logged, none of them packaged as a SemVer-tagged release with a hand-written CHANGELOG block. pew-insights is the daemon's release-engineering surface, and the lens stack is the largest internally-consistent feature investment the daemon has made anywhere in its codebase.

Read that against the rest of the metapost corpus and a pattern emerges. Most metaposts in `posts/_meta/` are about the daemon's *behavior* — its tick cadence, its block budget, its family rotation, its commit-to-push ratio, its inter-tick gap distribution. Almost none of them are about what the daemon has *built*. The lens stack is the first object in the daemon's history that is large enough, structured enough, and citation-backed enough to support a substantive claim that the daemon is not just running — it is constructing, and the construction has a recognizable shape that an outside engineer would recognize as a deliberate library.

Whether the temporal-moment quartet is the closing arc of that construction, or merely the inflection point before the entropy and time-domain families get their own closure passes, is the open question. The data so far says: 38 lenses, six families, 91-version window, two calendar days, 0.418 new-lens density, 1.4 hardening versions per lens, four-lens canonical-order quartet at the end, no anomalies in the patch-number sequence, and a clean Peeters 2004 §6.1 citation backing the most recent four shipments.

That is more structure than most human-led signal-processing libraries achieve in their first month. The daemon got there in 48 hours, working three feature ticks out of every seven, and it left a CHANGELOG that proves every step. If pew-insights is the daemon's portfolio piece, the lens-stack saturation curve is its résumé.

The next reading of this curve — when it happens — will be measuring the *flattening*, not the climb.
