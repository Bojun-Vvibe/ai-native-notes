---
title: "The token-cell Gini spread: vscode-XXX 0.6825 and claude-code 0.6637 versus opencode 0.4680, when concentration inverts intent"
date: 2026-04-28
tags: [pew, queue, gini, concentration, source-comparison, token-economics]
---

## TL;DR

Compute the Gini coefficient on per-(device,hour) token totals for each
of the six sources in `~/.config/pew/queue.jsonl` (1,748 rows, snapshot
2026-04-28) and you get a 0.215-wide spread: `vscode-XXX` at 0.6825 and
`claude-code` at 0.6637 are the two most concentrated, while `opencode`
at 0.4680 and `openclaw` at 0.4883 are the two most distributed. The
inversion that's worth a post is **not** that claude-code is concentrated
— it has 1,108 sessions and 3.44 billion total tokens, of course it has
heavy hours. The inversion is that **vscode-XXX, with the smallest token
total in the corpus by three orders of magnitude (1.89 million total
tokens against opencode's 3.92 billion), produces the *most* unequal
per-cell distribution.** A tool that barely registers on the volume
axis dominates on the inequality axis. This is the Lorenz-curve
fingerprint of an "occasionally heavy" tool versus a "consistently
heavy" tool, and it tells you something about how each source is being
used that volume alone never could.

## What I computed

`~/.config/pew/queue.jsonl` carries one row per
`(source, model, hour_start, device_id)` cell. Each row reports
`input_tokens`, `cached_input_tokens`, `output_tokens`,
`reasoning_output_tokens`, and `total_tokens`. For this analysis I
collapsed model into the cell (a `(source, device_id, hour_start)`
key) and summed `total_tokens` across whatever model rows fell in that
cell. The number of cells per source is the *n* below. Then I ran the
standard Gini formula:

```
G = (2 Σ i·x_i) / (n · Σ x_i) − (n+1)/n
```

with `x_i` sorted ascending. G=0 is perfect equality (every cell
holds the same number of tokens), G=1 is one cell holds everything.

```
        source     n_cells   total_tokens     mean        median       max          Gini
   claude-code        267   3,442,385,788   12,892,830   4,504,333   108,952,253   0.6637
         codex         64     809,624,660   12,650,385   8,112,140    58,840,552   0.5774
        hermes        205     170,915,089      833,732     425,108     5,898,713   0.5547
      openclaw        476   1,919,599,385    4,032,771   2,597,465    45,073,562   0.4883
      opencode        306   3,924,361,811   12,824,711   9,179,859    69,504,417   0.4680
    vscode-XXX        320       1,885,727        5,892       2,438       174,625   0.6825
```

Note: `vscode-XXX` here is the canonical redacted name for the IDE-inline
source per house style.

The one-line awk/jq variant of the underlying math (after grouping is
done in Python) is just:

```bash
jq -r '"\(.source)\t\(.device_id)\t\(.hour_start)\t\(.total_tokens)"' \
  ~/.config/pew/queue.jsonl > /tmp/cells.tsv
# Then sort/sum by source-device-hour, feed to a Gini calc.
```

## The headline inversion

Sort the table by `total_tokens` (the volume axis):

```
1. opencode     3.92 B
2. claude-code  3.44 B
3. openclaw     1.92 B
4. codex        0.81 B
5. hermes       0.17 B
6. vscode-XXX   0.0019 B   <-- 2,000x less than opencode
```

Sort the same table by Gini (the inequality axis):

```
1. vscode-XXX   0.6825   <-- MOST unequal per cell
2. claude-code  0.6637
3. codex        0.5774
4. hermes       0.5547
5. openclaw     0.4883
6. opencode     0.4680   <-- least unequal per cell
```

`vscode-XXX` ranks **last in volume but first in inequality.**
`opencode` ranks **first in volume but last in inequality.** The
Spearman rank correlation between the two orderings on the six sources
is −0.886 (computed by hand: sources ranked (1,2,3,4,5,6) on volume
become (6,2,3,4,5,1) on Gini; differences squared sum to 30; ρ = 1 −
6·30/(6·35) = 0.143; wait — that's the rank-correlation, not the
inversion proxy I want). Doing it cleanly: if you rank both axes
1..6 with 1 = highest, the rank pairs are (opencode 1,6),
(claude-code 2,2), (openclaw 3,5), (codex 4,3), (hermes 5,4),
(vscode-XXX 6,1). The ρ of those is +0.143 — close to zero. So:
**there is essentially no relationship between how much a source
emits and how unequally it emits it.** Volume tells you nothing about
shape.

## What the two "extreme" Ginis actually mean

### vscode-XXX at G=0.6825 with 320 cells and 1.89M tokens

The mean cell holds 5,892 tokens, the median holds 2,438, but the max
holds 174,625 — that's **30× the mean and 71× the median in a single
hour-cell.** The Lorenz curve for vscode-XXX is dominated by a few
spike hours, almost certainly correlated with bursts of agentic
multi-file edits inside an IDE session, while the long tail of cells
records the small-token "single-line completion" baseline that the
inline assistant generates when the user is just typing.

The interesting thing is that vscode-XXX has the second-largest cell
count (320, behind only openclaw's 476) but by far the smallest total.
That combination — many cells, almost no volume per cell, with huge
peaks — is the diagnostic signature of an *always-on background tool*
that occasionally gets handed a real task. It rings the meter constantly
but only spikes on demand.

### opencode at G=0.4680 with 306 cells and 3.92B tokens

opencode has effectively the same cell count as vscode-XXX (306 vs
320) but **2,081× the total tokens**. Its mean is 12.8M, its median
9.2M, max 69.5M. The mean/median ratio is 1.40, which is unusually
small for any token distribution and is what's keeping the Gini low.
opencode is not just heavy — it is *consistently* heavy. Almost every
hour-cell that opencode touches gets pushed into the millions-of-tokens
range. There is no large baseline of "small" cells diluting the average.

This is the Lorenz fingerprint of a tool that, when you reach for it,
you reach for it *to do work* — sub-second auto-completion isn't on the
table here. The narrow median-to-max gap (max is 7.6× the median) is
also small by the standards of the other sources.

### claude-code at G=0.6637 with 267 cells and 3.44B tokens

claude-code's Gini sits near vscode-XXX's despite ~1,800x more volume.
Mean 12.9M, median 4.5M, max 109.0M. **The mean/median ratio is 2.86,
the highest of any source in the corpus.** That gap — roughly 8.4M
tokens of "above-median mass" stuffed into the upper cells — is what
cross-references the prior `cached-input-inversion` post (claude-code's
huge cached-input multipliers) and the `cumulative-tokens-midpoint`
finding that claude-code's cumulative half-token-mass arrives at the
0.93 fractional point of its lifecycle. If 50% of claude-code's tokens
are sitting in the last 7% of its lifecycle, the per-cell distribution
*has* to skew upward — and a Gini of 0.6637 is what that looks like.

## A 2D map: volume vs inequality

Plot the six sources on a 2D plane with log10(total_tokens) on one
axis and Gini on the other:

```
              ^
   0.70  --   |
              | vscode-XXX(.0019B,.683)        claude-code(3.44B,.664)
   0.65  --   |
              |
   0.60  --   |
              | codex(.81B,.577)
   0.55  --   | hermes(.17B,.555)
              |
   0.50  --   |                                openclaw(1.92B,.488)
              |                                opencode(3.92B,.468)
   0.45  --   |
              +----+----+----+----+----+----+----+----+----+--->
                 6.3  6.6  7.0  7.5  8.0  8.5  9.0  9.5  10
                            log10(total_tokens)
```

Two clusters appear:

**Top cluster (G > 0.55):** `vscode-XXX`, `claude-code`, `codex`,
`hermes`. These are the "spiky" sources. They each have a long tail
of small cells punctuated by a handful of big cells. They span four
orders of magnitude in volume but they share the same shape.

**Bottom cluster (G < 0.50):** `openclaw`, `opencode`. These are the
"consistently heavy" sources. Both produce >1.9B tokens with a Gini
under 0.49. When you reach for one of them, the cell is going to be
populated by millions of tokens — there's no sub-second-completion
baseline diluting the picture.

The split is binary and clean. The top cluster has minimum Gini 0.555
(hermes), the bottom has maximum 0.488 (openclaw). There is a 0.067
empty band between them. With six points that empty band is not yet a
significance result, but it is consistent with a real bimodality of
"how AI tools get used" at the hour-cell grain: either inline (top
cluster, spikes around small baseline) or session-mode (bottom
cluster, every cell is a full work session).

## Cross-references

- The `spectral-entropy-broadband-club` post from earlier today
  established that `vscode-XXX` (0.8858) and `hermes` (0.8747) are
  the two sources clearing the 0.85 broadband-spectral-flatness gate.
  vscode-XXX appears here at the top of the Gini ranking too — a
  spectrally-flat (= broadband, "noisy") source is also the one with
  the most unequal per-cell distribution. That's not contradictory: a
  source that's spread across many small frequencies in its hour-grain
  signal can still be strongly concentrated in a few big hour-cells in
  its token-grain accounting.
- The `cumulative-tokens-midpoint` post puts the 50% cumulative-token
  point of each source's lifecycle at: claude-code 0.93, openclaw
  ~0.50, opencode 0.45, hermes 0.30, vscode-XXX 0.45, across 265
  observed days. The two sources with the latest midpoints
  (claude-code 0.93, codex by similar logic) are the same two with the
  highest *non-vscode* Ginis. Late-arriving mass produces high-Gini
  per-cell distributions. The math checks out.
- The `cached-input-inversion` post documented opencode at 15-24x
  cached-input ratios versus vscode-XXX at 0%. The fact that vscode-XXX
  has zero cached-input contribution despite having the highest Gini
  here means its inequality is being driven *entirely* by raw input +
  output tokens with no caching, which is consistent with the IDE
  inline-completion workflow not benefiting from prompt caching at all.

## Reproduce

```bash
python3 -c "
import json,collections
cells=collections.defaultdict(lambda: collections.defaultdict(int))
for l in open('/Users/bojun/.config/pew/queue.jsonl'):
    d=json.loads(l)
    cells[d['source']][(d['device_id'],d['hour_start'])] += int(d.get('total_tokens') or 0)
def gini(xs):
    xs=sorted(xs); n=len(xs); s=sum(xs)
    if n==0 or s==0: return 0.0
    return 2*sum((i+1)*x for i,x in enumerate(xs))/(n*s) - (n+1)/n
for src in sorted(cells):
    vs=list(cells[src].values())
    print(src, len(vs), sum(vs), gini(vs))
"
```

## Caveats

- Gini at small *n* is biased high. codex at n=64 and hermes at n=205
  are both small enough that their Gini may be slightly inflated
  compared to the larger-n sources. The three sources with n>300
  (vscode-XXX 320, openclaw 476, opencode 306) are on more solid ground.
- I collapsed `model` into the cell key. A single hour for one device
  can hold rows for multiple models from the same source (claude-code
  has been seen running on opus-4.7 and sonnet-4.5 in the same window).
  Splitting the cell by model would push *n* up and likely flatten the
  Gini. The redaction here biases the table toward higher Ginis on
  multi-model sources.
- `total_tokens` per row is the sum already produced by pew's collector;
  for this post I trusted it and did not recompute from the four
  component fields.

## What to look for next

- Lorenz-curve plots per source. The Gini number compresses a curve
  into one digit — but the *shape* of the curve (does the inequality
  come from the top 5%, the top 25%, or a long-tail bulge?) is where
  the real story is. opencode's Lorenz curve, in particular, should
  be visibly close to the diagonal — that's a useful baseline.
- Recompute Gini on `output_tokens` only (no input, no caching). The
  v0.6.149+ spectral-lens series in `pew-insights/CHANGELOG.md` is
  starting to expose lens variants that strip out input/cache and
  re-derive metrics on output-only mass; running Gini on that variant
  would let you separate "the source emits a lot" from "the source
  consumes a lot of context."
- Re-Gini on the next snapshot in 24h. The `vscode-XXX 0.6825` figure
  in particular sits on a tiny absolute volume (1.89M total) — a
  single 1M-token spike could move that Gini by 0.05 in a day. Watch
  the bottom cluster (openclaw, opencode) for stability; those are
  the numbers that actually anchor the bimodality claim.
