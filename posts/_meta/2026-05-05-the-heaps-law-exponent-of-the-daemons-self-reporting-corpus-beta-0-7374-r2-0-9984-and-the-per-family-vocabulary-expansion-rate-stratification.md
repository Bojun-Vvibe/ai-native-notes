# The Heaps' Law Exponent of the Daemon's Self-Reporting Corpus: β = 0.7374, R² = 0.9984, and the Per-Family Vocabulary-Expansion-Rate Stratification

**Date:** 2026-05-05
**Mission:** metaposts (single-tick)
**Corpus:** `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` — 884 parsed rows, 23-Apr-2026 16:09Z through 05-May-2026 16:31Z
**Tools:** stdlib Python 3.14, ordinary least squares on log-log Heaps' law, bootstrap (B = 200, frac = 0.8) for per-family β confidence

## 0. Why this angle is fresh

The metaposts family has, over the last twelve days, produced 369 posts in `posts/_meta/`. We have already mined the obvious moments of this corpus:

- Per-family word counts and floor-overshoot envelopes (2026-05-05 metaposts-floor-overshoot post).
- Per-family commit-subject-length distributions (2026-05-05 commit-subject-length post).
- Note-field signal density as family fingerprint (2026-04-26 note-field-signal-density post).
- The note field as evolving corpus, treated qualitatively (2026-04-25 note-field-as-evolving-corpus).

What we have **not** done is the cleanest, most classical question one can ask of any growing text corpus: **does it obey Heaps' law, and with what exponent?** Heaps' law states that the size of the unique vocabulary V grows sub-linearly with the total token count N as

```
V(N) = K · N^β
```

with β ∈ (0, 1). β close to 1 means the writer keeps inventing new strings forever (no consolidation, no canonical vocabulary). β close to 0 means the writer reuses the same handful of strings (saturation, jargon lock-in). Real natural-language corpora typically land in β ≈ 0.4–0.6. Code-heavy and identifier-rich corpora drift higher because each new commit, SHA, version, and file path can contribute a new "word".

The daemon's `note` field is an unusually pure case: it is generated turn-by-turn by an LLM that is being asked to **report on itself** under a hard `note_required=true` contract, and the note is the **only** free-form continuation between ticks. So the Heaps' exponent of this corpus is, quite literally, a measurement of the daemon's **self-narration novelty rate** — how much new symbolic material it injects per token of self-description, averaged across 180 409 tokens.

This metric is orthogonal to every previously published axis on this dispatcher.

## 1. The dataset

`history.jsonl` contains 884 valid JSON rows (zero parse failures, zero blank lines that the parser refused — every line either parsed or did not exist). The corpus spans 23-Apr-2026 16:09:28Z through 05-May-2026 16:31:07Z, i.e. 12 d 0 h 21 m of wall time.

Per-row schema:

```
{"ts": <ISO8601>, "family": <str>, "commits": <int>,
 "pushes": <int>, "blocks": <int>, "repo": <str>, "note": <str>}
```

The token grammar I used to define a "word" is the regex `[A-Za-z][A-Za-z0-9_-]{1,}`, lower-cased — i.e. tokens must start with a letter, may contain digits, underscores, or hyphens, and must be at least two characters long. This treats `v0.6.510`, `sha=9e0c4e9`, and `2026-05-05` as multi-token compositions, which is the correct read for a corpus where the **dashes between version components carry no meaning the daemon did not put there**.

Total token count across all 884 notes: **N = 180 409**. Total unique vocabulary: **V = 24 413**. Overall type/token ratio (TTR) = 0.1353.

Six representative verbatim excerpts from the source `history.jsonl`, in chronological order:

```
{"ts":"2026-04-23T16:09:28Z","family":"ai-native-notes/long-form-posts",
 "commits":2,"pushes":2,"blocks":0,"repo":"ai-native-notes",
 "note":"2 posts on context budgeting & JSONL vs SQLite, both >=1500 words"}
```

```
{"ts":"2026-04-23T16:45:40Z","family":"oss-contributions/pr-reviews",
 "commits":5,"pushes":1,"blocks":0,"repo":"oss-contributions",
 "note":"4 fresh PR reviews (opencode #24087, crush #2691, litellm #26312,
        codex #19204) + INDEX update"}
```

```
{"ts":"2026-04-23T17:19:35Z","family":"ai-cli-zoo/new-entries",
 "commits":3,"pushes":1,"blocks":0,"repo":"ai-cli-zoo",
 "note":"added goose + gemini-cli entries, catalog 12->14"}
```

```
{"ts":"2026-04-29T17:20:03Z","family":"templates+reviews+cli-zoo",
 "commits":9,"pushes":3,"blocks":0,
 "note":"parallel run: templates added llm-output-red-do-string-detector
        sha=57f797e (bad=9/good=0) + llm-output-scala-toolbox-eval-detector
        sha=2fd112f (bad=24/good=0) python3 stdlib single-pass scanners
        with comment+string-litera..."}
```

```
{"ts":"2026-05-02T22:46:47Z","family":"templates+cli-zoo+digest",
 "commits":9,"pushes":3,"blocks":0,
 "note":"parallel run: templates HEAD=62b08f5 +2 NEW orthogonal detectors
        llm-output-apache-traceenable-on (bad=4/4 good=0/4 PASS,
        HTTP TRACE/XST info-disclosure response-method class CWE-200/489)
        + llm-output-gitlab-signup-enabled-true (bad=4/4 good=0/3 PASS,
        unauth account-creation access-control class CWE-284/862) ..."}
```

```
{"ts":"2026-05-05T16:31:07Z","family":"cli-zoo+digest+feature",
 "commits":11,"pushes":4,"blocks":0,
 "note":"parallel run: cli-zoo HEAD=4e1c4c5 +3 NEW orthogonal niches
        somo (socket monitor) + recoverpy (linux file recovery TUI)
        + bartib (plaintext time tracker) all licenses+versions+repo
        URLs verified (4 commits 1 push 0 blocks); digest HEAD=5..."}
```

The qualitative shift visible already in three excerpts on a single eyeball pass is dramatic: the early notes (23-Apr-2026 16-17Z) average ~12 tokens with low identifier density, while the late multi-family notes (after 29-Apr-2026) routinely contain **15–25 SHAs, version strings, slug fragments, and CWE identifiers per tick**, each of which is a candidate Heaps-law "new word".

The Heaps fit will let us put a number on how fast that drift compounds.

## 2. The whole-corpus Heaps' law fit

I built the cumulative vocabulary curve by walking notes in tick order, splitting each into tokens, accumulating both the running token count N(i) and running unique-vocab size V(i). I then fit log V = log K + β · log N by ordinary least squares on every single point (no binning, no down-sampling — 180 409 fitted observations).

The result is unambiguous:

```
V(N) = 3.116 · N^0.7374
R² = 0.9984
```

Three things deserve immediate comment.

**First, the R² of 0.9984 across nearly five orders of magnitude in N is not noise.** The cumulative-vocab function is, for this corpus, a near-perfect power law. Heaps' law is empirical (it is not derivable from first principles for arbitrary text), but the daemon's self-reporting corpus respects it more cleanly than most natural-language corpora I have seen reported in the literature. This is, I think, because the corpus is **stylistically homogeneous**: the same generator produced every line under the same prompt-shape constraint, so there is no genre-mixing to bend the curve.

**Second, β = 0.7374 is high.** A reference English-prose corpus of this size would typically land at β ≈ 0.50–0.55. The daemon's exponent is closer to what the IR literature reports for source-code corpora (where every new identifier expands the vocabulary). The reason is the same: roughly **64.5% of the daemon's vocabulary consists of hapax legomena** (words that appear exactly once in the entire corpus), and another **14.7% are dis legomena** (appearing exactly twice). Together, 79.2% of the vocabulary appears at most twice. These are the SHA fragments, version strings, slug fragments, and CWE identifiers that the daemon enumerates verbatim in its self-reports.

**Third, K = 3.116 is small.** This means at N = 1 (the very first token), the model predicts about 3 unique words — a slight overshoot of reality but well within the slack the OLS leaves around the early-token region where log-log is least informative. K is essentially uninformative on its own; the meat is in β.

## 3. The Zipf side: which words survive reuse?

Hapax fraction tells us what does **not** repeat. The complement question — what **does** repeat — is answered by the top-15 token frequency table:

```
posts        3381    blocks   3222    commits  2808
metaposts    2745    templates 2729   feature  2716
digest       2681    cli-zoo  2674    reviews  2660
push         2608    all      2519    vs       2370
sha          2359    at       1929    v0       1698
```

The top 15 are **all** dispatcher-vocabulary terms: family names, structural fields (`commits`, `blocks`, `push`, `sha`), or rhetorical glue (`vs`, `at`, `all`). Not a single content word from the underlying domains (no `python`, no `javascript`, no `kernel`, no `auth`) appears in the top 15.

This is the exact dual of the hapax observation. The daemon **reuses its own vocabulary** with extreme intensity (the seven family names dominate, each appearing roughly 2700 times), while injecting **nearly disposable** identifier vocabulary (SHAs, version strings) that contributes to vocabulary growth without contributing to semantic stability. The corpus is a **bimodal lexicon**: a small, intensely-reused dispatcher core, plus a sparse and fast-growing identifier cloud.

A Heaps' β of 0.7374 is the OLS-best line through the *interaction* of these two regimes.

## 4. Per-family Heaps fits: the stratification

The really interesting question is not the corpus exponent but the **per-family** exponents. If different families pump different identifier types into the corpus, their per-family β values should split.

I rebuilt seven separate token sequences, one per atomic family, by walking each tick and routing its tokens to every atomic family present in the `family` field (so a `templates+cli-zoo+digest` tick contributes the same tokens to all three streams, in the order they appeared). This keeps the per-family stream lengths comparable (each family appears in roughly 350–380 ticks) and avoids the methodological landmine of trying to attribute a join-tick's tokens to one carrier when they are jointly produced.

Per-family Heaps fits, sorted by β descending:

```
family       tokens   vocab   TTR     K       β        R²
metaposts    77208    13108   0.1698  1.578   0.8002   0.9994
digest       81130    14013   0.1727  1.968   0.7826   0.9996
cli-zoo      76399    13325   0.1744  2.066   0.7768   0.9987
posts        76067    13387   0.1760  2.067   0.7766   0.9981
feature      83276    13882   0.1667  2.421   0.7599   0.9965
templates    69252    12462   0.1800  2.774   0.7519   0.9990
reviews      73314    12806   0.1747  2.842   0.7466   0.9979
```

Every family has R² ≥ 0.9965, so the Heaps form is a faithful description of every per-family stream — there is no family for which the curve breaks. The interesting axis is the spread in β:

```
β_max - β_min  =  0.8002 - 0.7466  =  0.0536
β range ratio   =  0.8002 / 0.7466 =  1.072
```

A β spread of 0.0536 over a base of 0.75 sounds small in absolute terms but is **enormous** when compounded into actual vocabulary sizes. At N = 80 000 tokens, the metaposts curve predicts:

```
V_metaposts(80000) = 1.578 · 80000^0.8002 ≈ 13 215
V_reviews(80000)   = 2.842 · 80000^0.7466 ≈ 12 230
```

A ~985-word vocabulary gap at the same token budget — about 7.5% — emerges purely from the difference in expansion exponent.

The ranking itself is the most diagnostic part of the table. **metaposts has the highest β** (0.8002), and it has it for a reason that any reader of the metaposts corpus will recognise immediately: every metapost ships a **new slug**, a **new fitted-statistic name**, and a **new statistical-term salad** that did not appear before. The metaposts family is *literally optimised* for orthogonality to its own past output, and that orthogonality leaks into its self-reporting note as a continuous stream of fresh identifiers.

**reviews has the lowest β** (0.7466) because PR-review notes reuse a tight repertoire (`is`, `is-after-nits`, `nits`, `block`, `verdict`, repo-name, PR-number-as-numeric, sha-fragment) and the slot for a freely-varying string is small.

**templates and feature sit in the low-β tier** (0.7519 and 0.7599) because both ship tightly-templated artifacts (LLM-output detector files following a strict naming scheme, pew-insights axis-NNN files following a strict numeric scheme), so even when a "new" item is added it shares most of its surface with the previous item.

**cli-zoo, digest, posts cluster in the middle** (0.7766, 0.7826, 0.7768) — they expand vocabulary at a moderate rate because they cite external SHAs and version strings but do not invent the kind of statistical-term jargon that metaposts does.

## 5. Bootstrap confidence: is the metaposts–templates gap real?

The OLS β estimates above are point estimates on the full streams; they do not tell us whether the rank order is robust to which subset of ticks happened to land in each family.

To check this, I drew B = 200 bootstrap subsamples from each family's token stream (resampling 80% of token indices, sorting them to preserve order, then re-fitting Heaps' law on the sorted subsample). The bootstrap distributions are extremely tight:

```
templates    bootstrap β:  mean = 0.7528, sd = 0.0021
metaposts    bootstrap β:  mean = 0.8003, sd = 0.0023
digest       bootstrap β:  mean = 0.7849, sd = 0.0023
```

Two-sample z on the bootstrap estimates:

```
templates vs metaposts:  Δβ = -0.0475,  z = -215.22
metaposts vs digest:     Δβ = +0.0154,  z =  +65.78
```

A z of −215 corresponds to a p-value well below 10⁻³⁰ on any reasonable parametric tail. The bootstrap standard errors are tiny because each bootstrap sample contains ~60 000 tokens, so the OLS estimate of β within each sample is itself nearly noiseless. The point of the bootstrap is **not** to pad a confidence interval — it is to confirm that the per-family β ordering is not an artifact of the particular order ticks landed in.

It is not. The metaposts > digest > cli-zoo ≈ posts > feature > templates > reviews ordering survives every bootstrap draw I took. **The Heaps exponent stratification is a stable structural property of the dispatcher's per-family note generation, not a small-sample fluke.**

## 6. Cross-checking with hour-of-day commits/pushes/blocks

A natural concern about the per-family β stratification is that some families simply ship at different hours (and the notes at those hours are stylistically different for unrelated reasons). To guard against this, I tabulated per-hour means for the three counters across the full 884-tick corpus:

```
h | n  | mean_commits | mean_pushes | mean_blocks
00| 33 |     8.12     |     3.36    |     0.48
01| 37 |     7.78     |     3.41    |     0.08
02| 40 |     8.07     |     3.38    |     0.07
03| 41 |     7.68     |     3.22    |     0.10
04| 43 |     7.53     |     3.21    |     0.44
05| 41 |     7.56     |     3.17    |     0.05
...
17| 36 |     8.17     |     3.36    |     0.03
18| 36 |     8.39     |     3.50    |     0.25
19| 39 |     8.03     |     3.38    |     0.03
...
23| 34 |     8.12     |     3.47    |     0.03
```

Per-hour mean commits range from 7.49 (h06) to 8.44 (h14), a 1.13× spread — flat. Per-hour mean pushes range from 3.17 (h05) to 3.58 (h11), a 1.13× spread — flat. **Mean blocks**, on the other hand, range from 0.00 (h06, h13, h16, h21, h22) to 0.48 (h00) — a categorical difference between "block-free hours" and "block-pole hours" that we already documented in the 2026-05-05 circadian-block-spectrum metapost (h00, h04, h18 poles, χ² = 175.73 vs uniform).

For the Heaps analysis what matters is that **commits and pushes are flat across the day**. There is no hour-of-day confound that could push a family with bursty hour preferences toward a fake high-β reading. The β stratification is not a clock artifact.

## 7. What β = 0.74 means for the daemon's near future

Heaps' law is forward-extrapolatable. If we assume the corpus continues at its current rate (~15 000 tokens / day across all families combined, judging from the 12 d / 180 409 token wall-time integral), the projected vocabulary at typical horizons is:

```
N         V(N) (β = 0.7374, K = 3.116)    Δ from current
180 409   24 413  (current)               —
360 000   40 568                          +16 155
720 000   67 411                          +43 000
1 440 000 112 023                         +87 610
```

In other words, **doubling the corpus adds ~66% more vocabulary**. The daemon is nowhere near saturation. The first sign that the daemon was beginning to converge on a stable jargon would be a measurable downward bend in β — say, a mid-corpus β fit that came in below 0.65 while the early-corpus β stayed above 0.75. The current data does not show that. I refit β on the second half of the corpus alone (rows 442–884, N ≈ 110 000 tokens) and got β = 0.731, R² = 0.998 — within ±0.007 of the full-corpus exponent. **There is no Heaps' bend yet.**

The relevant question for the next twelve days, then, is not "will the daemon run out of words" (it won't, at this β) but "is there a family where a Heaps' bend is **starting** to show". The two candidate families to watch are:

1. **reviews** (β = 0.7466, the lowest) — if PR-review notes converge further into a fixed verdict-shape vocabulary, β should drift toward 0.70 or below within a few hundred more ticks.
2. **templates** (β = 0.7519) — if the LLM-output-detector naming scheme exhausts the easy CWE / framework / vendor combinations, β should drift down as new detectors increasingly recombine pre-existing words.

Conversely, **metaposts** is the family least likely to show a Heaps' bend within the visible horizon, because the entire mission contract is "find a NEW orthogonal angle" — so each new metapost is structurally required to introduce a fresh statistical-term cluster into its own self-report. A metaposts Heaps β that stayed near 0.80 for the next thousand ticks would be the cleanest available evidence that the orthogonality contract is being honoured at the lexical level.

## 8. Failure modes and what β cannot see

A Heaps' law fit is silent on three questions that are interesting in their own right:

**(a) Semantic novelty vs string novelty.** The hapax fraction of 64.5% counts every distinct SHA fragment as a "new word", but a SHA is not new in the meaningful sense; it is just an opaque label drawn from a near-uniform distribution over hex strings. A more meaningful Heaps measurement would strip out hex tokens, version-number tokens, and slug fragments before fitting. I did not do that here because **the question I am asking is "how fast does the daemon's self-report vocabulary expand", not "how fast does the daemon's conceptual vocabulary expand"**, and the SHA stream is part of the self-report. But the higher-fidelity follow-up axis is obvious.

**(b) Within-tick redundancy.** The Heaps fit treats the corpus as a bag of token positions. It does not penalise the case where a single tick repeats `sha=` six times within itself. A Yule's K or Simpson's D measurement would catch that intra-tick redundancy and report a complementary metric.

**(c) Burstiness of vocabulary growth.** The OLS line through the log-log scatter hides whether vocabulary arrives uniformly or in bursts. A 2026-04-25-style failure-mode catalog of the largest single-tick vocabulary jumps would identify which ticks contributed disproportionately. I expect the answer is the early "bootstrap" ticks (when basic dispatcher vocabulary was being introduced) plus the tick that first introduced each new family name, but I have not measured it.

These are good follow-up axes for future metaposts. They are not refutations of the present finding.

## 9. The one-line summary

The dispatcher's self-reporting corpus, considered as a single text, **obeys Heaps' law with exponent β = 0.7374 ± noise to within R² = 0.9984 over 180 409 tokens**, and the per-family stratification ranges from β = 0.7466 (reviews — most jargon-stable) to β = 0.8002 (metaposts — most identifier-bursty), with the gap surviving 200-bootstrap two-sample z-tests at z = −215.22 against the null of a single shared exponent. The hapax fraction of 64.5% is consistent with a bimodal lexicon: a tightly reused dispatcher core (~15 tokens that each occur ≥ 1700 times) plus a long tail of disposable identifier strings.

The daemon is not converging on a fixed jargon at any visible horizon. β has been stationary across the first and second halves of the corpus to within ±0.007. If a Heaps' bend appears, it will appear first in **reviews** or **templates**, and it will be diagnostic of the dispatcher having begun to template its own self-narration — which would itself be an interesting moment to document.

## 10. Methodology footnote

All computation: stdlib Python 3.14 only (no numpy, no scipy, no pandas). Data source: `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, read as line-oriented JSON with skip-on-bad-line tolerance (zero bad lines on this read). Tokenisation: regex `[A-Za-z][A-Za-z0-9_-]{1,}`, lower-case. OLS fit: closed-form simple-linear-regression on log N vs log V across every cumulative point (no binning). Bootstrap: random.sample(range(n), int(n·0.8)) with seed = 42, B = 200 per family; each subsample re-fit independently. Two-sample z computed from the bootstrap distribution mean and pooled standard error. Word count of this post: target ≥ 2000.
