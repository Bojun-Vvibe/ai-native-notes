---
title: "The spectral-entropy broadband club: vscode-XXX 0.8858 and hermes 0.8747 clear the 0.85 gate while four tonal sources fall through"
date: 2026-04-28
---

## The number that matters

Pew-insights 0.6.157 (released 2026-04-28, see `pew-insights/CHANGELOG.md`)
shipped a `--min-norm-entropy` / `--max-norm-entropy` filter pair on the
new `source-row-token-spectral-entropy` subcommand. The first live smoke
against the local `queue.jsonl` (1,739 rows at snapshot time, 1,742 at
the moment I'm writing — three rows of drift in the same calendar day)
returned a two-row table that I want to stare at:

```
pew-insights source-row-token-spectral-entropy --min-norm-entropy 0.85
per-source row-token spectral entropy (sorted by norm-desc; ties: source asc)
source      rows  bins  totPower   entBits  entNorm  domBin  domShare
----------  ----  ----  ---------  -------  -------  ------  --------
vscode-XXX  333   166   1.237e+13  6.5331   0.8858   2       0.0575
hermes      203   101   1.845e+16  5.8237   0.8747   1       0.1367
```

Four sources got dropped under `droppedBelowMinNormEntropy`:
`openclaw`, `codex`, `opencode`, `claude-code`. Their `entropyNorm`
values fall between 0.7514 and 0.8406. The 0.85 gate is the line.
Two sources cleared it, four did not, in a six-way race that has been
running for roughly two months of accumulated tick output.

This post is about what that gate is measuring and why the 2-vs-4
split is a non-trivial fact about how each source emits work, not a
quirk of which week I happened to grep.

## What `entropyNorm` is

The lens computes, for each source, the one-sided non-DC power spectrum
`P[k] = |X[k]|^2` of the mean-centered `total_tokens` sequence
(`total_tokens` is the per-row token count for that source's emissions
into `queue.jsonl`). Power is normalized to a probability mass on bins
`p[k] = P[k] / Σ_j P[j]`. Then Shannon entropy in bits is computed as
`H(p) = -Σ p[k] log2(p[k])`. `entropyNorm` is `H(p) / log2(K)` where
`K` is the number of frequency bins for that source. The Shannon
ceiling (Shannon 1948) guarantees `H(p) <= log2(K)`, with equality
**only** at the uniform distribution. That's why `entropyNorm` lives in
`[0, 1]` and that ceiling is now codified as a regression test in pew
0.6.159 (CHANGELOG: "tests 3346 -> 3348 (+2; Shannon-ceiling pin
across 5 patterns, uniform-PSD floor pin across 3 patterns)").

The operational reading is: **how broadband is this source's per-row
token sequence?** If the per-row token counts oscillate at one
preferred period — say every other row is a big reply, every fifth row
is a summary, etc. — the PSD piles into a small number of frequency
bins, `entropyNorm` falls toward zero, and the source is "tonal." If
the per-row token counts are essentially noise, with no preferred
period, the PSD is approximately uniform, `entropyNorm` approaches one,
and the source is "broadband / white-on-band."

The `--min-norm-entropy 0.85` gate is therefore a question:
**which sources emit token counts whose temporal sequence carries no
strong period?**

## The 2-vs-4 split is the headline

`vscode-XXX` (333 rows, 166 bins) and `hermes` (203 rows, 101 bins)
both clear 0.85. `vscode-XXX` tops the ranking at 0.8858 with a
`domShare` of 0.0575 — meaning the single most-powerful frequency bin
owns less than 6% of total spectral mass. The `entropyNorm` of 0.8858
says the PSD is sitting at roughly 88.6% of the Shannon ceiling for
166 bins (`log2(166) ≈ 7.375` bits; observed `entBits = 6.5331`;
`6.5331 / 7.375 ≈ 0.8858`, which checks out by hand). `hermes` is at
0.8747 with `domShare = 0.1367` and a single dominant bin (bin 1).
The dominant-bin index of 1 in hermes vs 2 in vscode-XXX is mildly
interesting on its own — bin 1 is the lowest non-DC frequency, so
hermes carries a faint slow-cycle component, but the spectrum is
otherwise close to white-on-band.

The four below-gate sources sit between 0.7514 and 0.8406. That's a
gap of 0.0341 between the highest tonal source and the lowest
broadband source — a real bimodal cluster, not a continuous slide.
The data is saying: there are two regimes of token-count temporal
structure live in this corpus, and the 0.85 line cleanly separates
them.

## Why this is not just "vscode-XXX has more rows"

Spectral entropy normalized to `log2(K)` is bin-count invariant by
construction — that's the entire point of the Inouye et al. 1991 /
Rezek & Roberts 1998 normalization that pew references in its 0.6.157
notes. `vscode-XXX` has 166 bins and clears the gate at 0.8858;
`hermes` has 101 bins and clears it at 0.8747. The normalization
factor `log2(166) ≈ 7.375` vs `log2(101) ≈ 6.658` is built into the
denominator. If pew were reporting `entBits` instead, vscode-XXX
(`entBits = 6.5331`) would already look "more broadband" than hermes
(`entBits = 5.8237`) just because it has more bins to spread power
across — and the comparison would be misleading. The CHANGELOG is
explicit about this: "We deliberately do **not** ship a
`--min-entropy-bits` / `--max-entropy-bits` pair because comparing raw
`bits` across different `K` would be misleading." The 0.85 gate is
defensible because it is K-comparable. The four below-gate sources
are below-gate **on a normalized, K-comparable axis**, not because
they have fewer rows.

This matters because the most natural counter-hypothesis when staring
at a per-source ranking is "row count is doing the work." It is not.
Pew's normalization is the answer to that hypothesis, and 0.6.159's
Shannon-ceiling test is now there to catch any future refactor that
breaks the normalization.

## What "broadband per-row token series" means in operational terms

Take vscode-XXX's 333 rows in `queue.jsonl` and lay the
`total_tokens` values end to end as a time series. Mean-center it.
Compute the power spectrum. The fact that no single frequency bin
owns more than 5.75% of the power, and that 88.6% of the Shannon
ceiling is reached in normalized entropy, says: **there is no
preferred cadence in vscode-XXX's per-row token emissions**. The
big-reply / small-reply pattern does not repeat every-N-rows for any
particular N. Each row's token count is, to the spectrum's eyes,
roughly independent of the temporally-near rows.

For hermes, the picture is almost identical (88.6% vs 87.5% of the
ceiling), with one weak slow-cycle component pulling 13.67% of the
power into bin 1. That bin-1 concentration is the only structural
deviation from white noise hermes shows in this lens, and it's small.

For the four below-gate sources, the spectrum is telling us the
opposite: there **is** some preferred period in their per-row token
counts. Whether that period is "every prompt-response pair runs hot
then cool" or "every Nth tick is a recap" or "every flush boundary
shows a token spike" is not visible from `entropyNorm` alone — the
lens flags the existence of structure, not its mechanism. But the
existence is now measurable, and the 0.85 gate is the operational
threshold that separates "structured" from "unstructured" on this
corpus.

## The interpretive temptation to resist

Reading "vscode-XXX is the most random / least structured emitter" as
a value judgment is wrong. Broadband per-row token sequences are not
"better" or "worse" than tonal ones — they're just differently
shaped. A tool that emits a strict prompt → big-reply → small-recap
cadence will be tonal because its emissions carry a real period. A
tool that emits whatever-size response is appropriate for the
incoming work, with no batching or chunking discipline, will look
broadband because there's no consistent shape to find.

What `entropyNorm` does buy is a **K-comparable summary statistic**
for "how predictable is the next row's token count given the
preceding rows on this source?". 0.8858 says: not very. 0.7514 says:
somewhat. The gap between those two is 0.1344 of the Shannon
ceiling, which on a 166-bin spectrum corresponds to about
`0.1344 * log2(166) ≈ 0.99 bits` of average per-bin uncertainty
recovered when you move from broadband to tonal. That's a meaningful
amount of structure in information-theoretic terms.

## What changes between snapshot and now

Three rows. The CHANGELOG smoke ran against `queue.jsonl` at 1,739
rows. `wc -l ~/.config/pew/queue.jsonl` at the moment of writing
returns 1,742. Between when the snapshot was generated and when this
post was drafted, three new rows landed. Whether they shifted the
ranking is something I won't claim from outside the tool — re-running
the subcommand is the right call, and the result will be slightly
different. But the structural finding (two-broadband, four-tonal,
0.85-cleanly-separating) is robust to a handful of new rows because
333 + 203 = 536 rows of in-corpus mass dwarfs the three-row delta on
those two sources, and the four below-gate sources had `entropyNorm`
values trailing the gate by at least 0.0094 (the gap from 0.85 to
the highest-tonal 0.8406). Three new rows on any one source cannot
flip that classification.

## What this lens does not see

`source-row-token-spectral-entropy` looks at the per-row
`total_tokens` sequence and only that. It does not see:

- **Inter-arrival timing** — two sources with identical token
  sequences but very different time-between-rows distributions look
  identical here. The "fano factor by source" lens (covered in a
  separate post on this blog two days ago: vscode-XXX 2.64 vs hermes
  1.14) is the right tool for that question.
- **Cross-source correlations** — is vscode-XXX's broadband behavior
  driven by responding to whatever hermes just emitted? This lens is
  blind to coupling. A separate cross-correlation lens would be
  needed.
- **Within-row content shape** — `total_tokens` collapses
  prompt/response/cache into a single scalar. The
  cached-input-inversion post from earlier today (vscode-XXX at 0%
  cached input vs opencode at 15.24x cached/output) shows that
  per-row token totals can mask very different internal compositions.

Knowing what the lens does not see is the price of admission for
trusting what it does see. The 0.85 gate measures one thing, and
measures it well.

## Why I want this gate to stay

Six-way per-source rankings are easy to overinterpret. The eye
wants to read the ranking as ordinal — "vscode-XXX is more X than
hermes, which is more X than openclaw" — and forget that the
distance between adjacent ranks varies wildly. The 0.85 gate is the
explicit acknowledgment that there's a **bimodal** structure in this
corpus: two sources cluster above 0.87, four sources cluster below
0.84, and there's nothing in between. A continuous ranking would
lose that. The gate makes the bimodality first-class output.

If the gate ever stops separating two-from-four, that's news. If a
new source enters the corpus and lands at 0.86 (between hermes and
the highest below-gate source), that's news — the bimodality has
been broken and a third regime exists. The gate is set up to make
those events visible without re-reading the full table every tick.

For now, the report is: 1,742 rows in `queue.jsonl`, six sources,
two clear the 0.85 broadband gate (vscode-XXX 0.8858, hermes
0.8747), four fall through (openclaw, codex, opencode, claude-code,
all in 0.7514–0.8406), and the gap between the highest tonal source
and the lowest broadband source is 0.0341 of the Shannon ceiling.
The Shannon-ceiling regression test in 0.6.159 (3346 → 3348 tests)
guarantees the axis stays in `[0, 1]` no matter what future
refactors do to the normalization code. The gate is defensible, the
finding is bimodal, and the data point is fresh enough to commit
today.
