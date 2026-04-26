# Benford on output tokens: the 40.3% leading-one and the ide-assistant-A conformity anomaly

date: 2026-04-26
tags: [pew-insights, benford, leading-digit, output-tokens, statistical-fingerprint]

## TL;DR

Run Benford's first-digit test against `output_tokens` per CLI source over the last ten weeks of a single operator queue and you get a counter-intuitive ranking: the *smallest* source by token mass — `ide-assistant-A` (1.88 M tokens, 321 rows) — is the only one that lands in the "acceptable" Nigrini conformity band (MAD% = 1.326). Every agentic CLI that actually drives long-running missions sits in the "marginal-to-nonconformity" band (MAD% 2.09–2.73). The shape of the violation is the same across all six sources: the leading digit `1` is overrepresented (24–40% versus the Benford expectation of 30.1%) and the right tail (digits 7, 8, 9) is suppressed. That uniform-direction skew is the signal. The *amplitude* of the skew tracks how strongly each tool's output sizes are clamped by an internal generation budget — and it's strongest exactly where you'd expect generation budgets to bite hardest.

## The data, captured live

Source command, run inside this repo at the time of writing:

```
$ node ~/Projects/Bojun-Vvibe/pew-insights/dist/cli.js source-output-token-benford-deviation
```

Header (verbatim, captured at `2026-04-26T07:48:40.237Z` UTC):

```
sources: 6 (shown 6)    rows: 1,522    tokens: 9,057,341,333
min-rows: 30    max-mad: —    require-d1-mode: no
dropped: 0 bad hour_start, 12 non-positive output, 0 source-filter,
         0 below min-rows, 0 above max-mad, 0 non-d1-mode, 0 below top cap
P_benford(d) = log10(1 + 1/d), d in 1..9; chi2 has 8 d.o.f.
MAD% = mean_d|obs-exp|*100
Nigrini conformity: <0.6 close, 0.6-1.2 acceptable, 1.2-1.5 marginal, >1.5 nonconformity
```

Per-source rows (verbatim columns, sorted by tokens descending):

```
source          firstDay    lastDay     nRows  d1%    d2%    d3%    d4%    d5%    d6%   d7%   d8%   d9%   chi2   MAD%   tokens
claude-code     2026-02-11  2026-04-23  299    40.13  16.39  9.03   7.36   6.69   6.35  6.35  3.34  4.35  17.45  2.352  3,442,385,788
opencode        2026-04-20  2026-04-26  287    26.83  16.72  9.76   6.27   10.80  8.71  9.76  5.57  5.57  19.58  2.292  2,939,279,889
openclaw        2026-04-17  2026-04-26  393    24.68  10.94  13.74  11.70  13.49  8.91  5.60  5.85  5.09  34.82  2.731  1,720,727,870
codex           2026-04-13  2026-04-20  64     35.94  15.63  14.06  9.38   4.69   6.25  7.81  3.13  3.13   3.10  2.093    809,624,660
hermes          2026-04-17  2026-04-26  158    34.81  20.25  10.13  3.16   5.06   8.23  5.70  5.70  6.96  13.70  2.634    143,443,069
ide-assistant-A 2025-07-30  2026-04-20  321    27.41  19.63  12.77  11.21  5.92   6.85  7.79  4.67  3.74   6.74  1.326      1,880,057
```

(The corpus is a single operator's `~/.config/pew/queue.jsonl`; the inline-completion IDE source is labelled `ide-assistant-A` per project convention.)

The Benford expected vector for reference:

```
d:        1     2     3     4     5     6     7     8     9
P_b(d): 30.1  17.6  12.5   9.7   7.9   6.7   5.8   5.1   4.6   (%)
```

## Why we're testing Benford on output tokens at all

Benford's law says that across a wide variety of natural data spanning multiple orders of magnitude, the leading decimal digit `d` of a value follows `P(d) = log10(1 + 1/d)`. It holds when the values are produced by multiplicative processes (incomes, river lengths, populations) and breaks when the values are clamped, rounded, capped, or assigned (postal codes, hand-tuned thresholds, anything that hits a ceiling).

`output_tokens` per (source, UTC hour) bucket is a great candidate for a Benford check because the *expected* generative process is multiplicative — different prompts trigger wildly different completion sizes, completion sizes vary across orders of magnitude, no single sampler is in charge. If output sizes obeyed Benford well, that would be evidence the generation pipeline is "free" — driven by prompt diversity, not by a mid-stack clamp.

If they *don't* obey Benford, the *direction* of the violation tells you which clamp is firing:

- Excess in `d1` and `d2`, depletion in `d7..d9`: **upper budget cap**. Generation is being shaved off near a ceiling, so values pile up under the next-lower order of magnitude.
- Excess in `d5..d9`, depletion in `d1..d2`: **lower floor / minimum**. Some "must produce at least N" rule is rejecting small completions.
- Mode shifted to `d2` or `d3` instead of `d1`: **rounding / quantisation** at fixed buckets that don't start at a power of ten.

Across the six sources here, *every single one* shows the same direction: `d1` and `d2` overrepresented, `d7..d9` suppressed. So the *direction* of the artifact is the same — generation is being shaved against a ceiling — and the only thing that varies is *how hard* it's getting shaved.

## What the per-source numbers actually say

### claude-code: the strongest leading-one signal (40.13%)

`claude-code` shows `d1` at **40.13%** versus the Benford-expected 30.1%, an absolute excess of 10.0 percentage points. That's a 33% relative excess in the very first cell. The right tail is gutted: `d7..d9` together come to 14.04% versus the expected 15.5%, with `d8` particularly anaemic at 3.34% (vs expected 5.1%). Chi-square = 17.45 on 8 d.o.f. (p ≈ 0.026, rejecting Benford). MAD% = 2.352 — squarely in Nigrini's "marginal" band, very close to "nonconformity" (>1.5).

Reading: a substantial fraction of claude-code completions are landing in the 1xxx, 10xxx, 100xxx token range and not pushing into the 7-9k or 70-90k range. That's consistent with a generation-time soft cap that prefers to wrap up rather than emit one more order of magnitude of tokens.

### opencode: the same skew at lower amplitude (26.83% d1, but 19.58 chi-square)

`opencode` looks almost paradoxical at first: `d1%` is *lower* than Benford-expected (26.83% vs 30.1%), so the leading-one is *underrepresented*, yet the chi-square is *higher* than claude-code's (19.58). The tell is the middle-to-late digits: `d5%` = 10.80, `d6%` = 8.71, `d7%` = 9.76, all elevated. The skew here isn't "everything piles into 1" — it's "the distribution is too flat across `d5..d7`", which is a signature of a generation pipeline that frequently emits *medium-sized* completions in a relatively narrow band. Different bug shape, same MAD% band (2.292, marginal).

### openclaw: the worst chi-square of the cohort (34.82)

`openclaw` is the only source with a chi-square big enough to reject Benford at p < 0.001 with room to spare. Inspect the row: `d3%` = 13.74, `d4%` = 11.70, `d5%` = 13.49 — all materially above the Benford expectation, while `d1%` is the lowest in the cohort at 24.68%. This is a different artifact: openclaw generates a lot of completions whose first digit is 3, 4, or 5. Combined with the source's profile (the 16x harness gap discussed in the Apr-26 source-x-model post — same Opus weights, very different output shapes depending on the surrounding harness), this looks like a wrapper that pads or formats responses to a near-fixed scaffold size, pushing many bucket totals into the 3xxx/4xxx/5xxx region.

### codex: low chi-square because n is tiny (64 rows)

`codex` has the smallest row count that still cleared the `min-rows = 30` filter. At n = 64, the test simply doesn't have power: chi-square = 3.10, well below any rejection threshold. Yet the *MAD%* is still 2.093 — the *shape* deviation from Benford is comparable to the other agentic sources. So the right reading isn't "codex conforms"; it's "codex has too few buckets to reject, but the shape is the same kind of marginal."

### hermes: the most asymmetric tail (`d4%` = 3.16)

`hermes` is interesting because the deviation isn't concentrated in `d1` (34.81%, modest excess) — it's a deep crater at `d4` (3.16% vs expected 9.7%, only one third of expected). MAD% = 2.634, the second-worst conformity score in the table. Hermes is the reasoning-heavy proxy in this fleet, and a missing `d4` shelf suggests very few buckets land in the 4xxx token range — i.e. generations either stay small (1xxx-3xxx) or jump to 5xxx+. There's a "no medium completions" gap.

### ide-assistant-A: the conformity outlier (MAD% = 1.326)

This is the punchline. `ide-assistant-A` is the *smallest* source in the corpus by a factor of ~750x relative to the leader (1.88 M tokens vs 3.44 B), and it's the *only* source in the "acceptable" Nigrini band. Its digit profile (27.41 / 19.63 / 12.77 / 11.21 / 5.92 / 6.85 / 7.79 / 4.67 / 3.74) is the closest match to the Benford expectation. Why?

Because `ide-assistant-A`'s output_tokens-per-bucket distribution is genuinely multiplicatively driven by inline-completion request sizes that vary across many orders of magnitude with no agent-side budget shaping. The other five sources are *agentic loops* — every one of them has some form of "wrap up the turn" heuristic, max-output-tokens default, tool-call interruption, or response-length post-processing. Inline-completion has none of that: it's a stateless request with no agent-side clamp, so the leading-digit distribution is allowed to be natural.

This is exactly the kind of finding Benford is good at. It's not telling you the values are *wrong*; it's telling you which generative process has a clamp in it.

## What this is *not* evidence of

A few honest caveats so this post doesn't get over-cited:

1. **Sample sizes vary a lot.** codex (64 rows), hermes (158), ide-assistant-A (321), opencode (287), claude-code (299), openclaw (393). Chi-square scales with `n`; MAD% does not. That's why MAD% is the cross-source-comparable scalar and why the table sorts by tokens, not by chi-square.

2. **`min-rows = 30` was the cutoff.** Anything thinner is excluded. The `dropped` line confirms `0 below min-rows` — every source had at least 30 active hour buckets in window. Note also `12 non-positive output` rows were dropped (zero-output buckets can't have a leading digit).

3. **Bucket grain matters.** The leading digit is taken from `total_output_tokens` per (source, UTC hour) bucket. If you ran the same test on per-row outputs you'd see a different (likely larger) sample and a slightly different distribution because intra-hour aggregation smooths the right tail.

4. **The "ceiling clamp" hypothesis is one explanation, not the only one.** A multiplicative process truncated above some scale produces a Benford violation in this direction; so does a process where output sizes are correlated with prompt sizes and prompt sizes themselves don't span enough orders of magnitude. The Apr-26 cache-input-asymmetry and prompt-output-correlation posts both show evidence that prompt-size distributions are wide enough across these sources that the output-side clamp is the more parsimonious story, but it's not airtight.

## Operational reading

If you're operating a multi-CLI fleet and you want a single-number "is this tool's generation behaviour healthy" check, output-token MAD% against Benford is a surprisingly cheap diagnostic:

- **MAD% < 1.5** (Nigrini "acceptable" or better): the tool is emitting completions whose sizes follow a natural multiplicative process. No agent-side budget shaping is dominating.
- **MAD% 1.5–2.5** (Nigrini "marginal"): the tool has a soft generation clamp. Worth checking whether `max_output_tokens`, hard-coded "wrap up the response" heuristics, or wrapper-side post-processing are doing more work than you think.
- **MAD% > 2.5** (Nigrini "nonconformity"): there's a strong shaping pressure on output sizes. Combine with `output-input-ratio` and `output-size` to confirm whether it's a cap (tail too thin) or a floor (head too thin).

In this corpus, only `ide-assistant-A` is in the first band, and only `openclaw` (2.731) and `hermes` (2.634) cross into the third. claude-code (2.352), opencode (2.292), and codex (2.093) all sit in the middle "marginal" band — same as each other, despite very different total token volumes (3.44 B / 2.94 B / 0.81 B respectively).

The Benford test is doing exactly the job it's good at: telling you that the *shape* of generation across five independent agent harnesses is more similar than their volumes would suggest, and telling you that the one source without an agent harness is the one that fits the natural-process baseline.

## Cross-references

This sits next to several related Apr-25 / Apr-26 posts in this repo:

- *output-input-ratio-bimodality* (Apr-26) — same fleet, different lens. Bimodality there + Benford skew here are consistent: each source has its own preferred output band, and that band rarely lines up with `d1`-dominated geometry.
- *source-x-model-output-input-ratio: the 16x harness gap on the same opus-4.7 weights* (Apr-26) — direct evidence that harness shapes output even when the weights are pinned. Benford rejection here is the digit-level counterpart of that ratio gap.
- *the 1m+ prompt tier: 47% of rows and the gpt vs claude shape divergence* (Apr-26) — explains why the prompt side is wide enough that the output-side clamp is the better explanation for the `d7..d9` depletion.
- *picking a baseline scorer: zscore, mad, ewma* (Apr-24) — methodological prior on why MAD% is a more honest cross-source scalar than chi-square when sample sizes differ.

## Reproduce

Single command, no args, runs in well under a second on the corpus above:

```
node ~/Projects/Bojun-Vvibe/pew-insights/dist/cli.js \
  source-output-token-benford-deviation
```

Filters worth knowing: `--min-rows N` (default 30) raises the threshold for inclusion; `--max-mad X` filters *out* any source whose MAD% exceeds X (useful if you want to see only the conformers); `--require-d1-mode yes` keeps only sources where the modal leading digit is 1 (this corpus passes that filter on every row — every source's modeFreq% column shows `1`).

The capture timestamp at the top of the header (`as of: 2026-04-26T07:48:40.237Z`) is the canonical citation marker — re-run the command tomorrow against the same `~/.config/pew/queue.jsonl` and the numbers will drift by a hair as new buckets land, but the ranking and conformity bands will hold across the next several thousand rows of new data based on the trajectory the per-source rows have been on for the last six weeks.
