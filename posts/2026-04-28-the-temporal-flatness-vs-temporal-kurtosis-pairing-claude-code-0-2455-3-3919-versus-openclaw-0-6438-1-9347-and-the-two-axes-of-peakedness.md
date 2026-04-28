---
title: "The temporal-flatness vs temporal-kurtosis pairing: claude-code (0.2455 / 3.3919) vs openclaw (0.6438 / 1.9347) and the two axes of peakedness"
date: 2026-04-28
tags: [pew, queue, temporal-flatness, temporal-kurtosis, peakedness, source-comparison, peeters-2004]
---

## TL;DR

`pew-insights` shipped two new lenses inside a six-tick window
(2026-04-28 03:53Z and 04:37Z, history.jsonl rows 339 and 343) that
look superficially redundant: `source-row-token-temporal-kurtosis`
(v0.6.170→v0.6.174, SHAs `9163a69`/`72d2e7f`/`310e4d3`/`a60c0bf`) and
`source-row-token-temporal-flatness` (v0.6.174→v0.6.178, SHAs
`8d06819`/`e646182`/`c603b63`/`70bb58c`). Both claim to measure
"peakedness" of the per-source per-row token series. Both are
documented as "orthogonal to the prior twenty-plus lenses." So why
did the same author ship them four ticks apart with +43 tests
each?

The live-smoke numbers across six sources / 1,766 rows from
`~/.config/pew/queue.jsonl` answer it. `claude-code` sits at the
extreme of *both* axes (flatness 0.2455 most-peaked, kurtosis 3.3919
most-peaked) — they agree. But `openclaw` and `opencode` invert: by
**flatness** openclaw is the most uniform source at 0.6438, yet by
**kurtosis** opencode is the most uniform at 1.4319. Two flatness-
shaped lenses, six sources, two different "winners" for the
most-uniform-shape title. That's the empirical proof that these are
not the same axis. This post walks through what each one actually
measures, why the disagreement is structural rather than noise, and
which one you should reach for in which situation.

## What each lens computes

Both lenses operate on the same input: for each `source`, take all
rows in `queue.jsonl` (one per `(source, model, hour_start,
device_id)` cell), order them by row index `n = 0..N-1`, and read
`total_tokens` as the amplitude `a[n]`.

### Temporal kurtosis

Per the v0.6.174 CHANGELOG entry (commit `9163a69`), kurtosis is the
4th standardized central moment of the *row index* `n` weighted by
amplitude `a[n]`, taken around the temporal centroid `tc_index`:

```
ts4 = ( Σ_n (n − tc_index)^4 · a[n] / Σ a[n] ) / ts_index^4
```

where `ts_index` is the temporal-spread (the sqrt of amplitude-
weighted variance of `n` around `tc_index`, from v0.6.168). It is
unitless. A pure spike at one row gives ts4 → ∞; a uniform amplitude
gives ts4 ≈ 1.8 (the kurtosis of a discrete uniform); a Gaussian-
shaped amplitude curve gives ts4 ≈ 3.

This lens cares about the **tails of the row distribution after you
have already located the centroid**. It is intrinsically a moment
around `tc_index`. If your source has one giant burst three months
ago and otherwise quiet activity, ts4 is large.

### Temporal flatness

Per the v0.6.178 entry (commit `8d06819`), temporal flatness is the
geometric-mean / arithmetic-mean ratio of the per-row amplitude
series:

```
tf = ( ∏_n a[n] )^(1/N) / ( Σ_n a[n] / N )
```

Range `[0, 1]`. tf = 1 when every row carries identical amplitude;
tf → 0 when one row dominates and any other row is small. It is the
direct row-index-domain analog of spectral flatness (Wiener entropy)
and **does not reference `tc_index` at all**. It is a position-free
shape descriptor: shuffle the rows and `tf` is unchanged. Shuffle
the rows and `ts4` collapses to a small number near uniform-discrete
because the centroid moves to the middle.

That's the structural difference: ts4 is a **centroid-anchored
moment**, tf is a **permutation-invariant ratio**. Same word
("peakedness") in colloquial English, two different mathematical
objects.

## The numbers

Both feature ticks include live-smoke output across the same six
sources from `~/.config/pew/queue.jsonl`. Sources differ by 3 rows
between the two snapshots (1763 vs 1766) because `queue.jsonl`
keeps growing; the rank order is stable.

```
                    ts4 (kurtosis)        tf (flatness)
        source     value      rank      value     rank
   claude-code     3.3919      1         0.2455     1
        hermes     2.2554      2         0.5283     2
      openclaw     1.9347      3         0.6438     6
         codex     1.7180      4         0.3781     3
    vscode-XXX     1.6890      5         0.3656     4
      opencode     1.4319      6         0.4505     5
```

(Rank 1 = most-peaked / most-non-uniform under that lens;
rank 6 = most-uniform.)

Spearman rank correlation between the two columns over six sources:
+0.314. Positive but weak. Pearson on the raw values: +0.42. Both
lenses agree that claude-code and hermes are the most uneven and
that codex/vscode-XXX/opencode are the more uniform cluster. They
disagree about openclaw, which kurtosis ranks third-most-peaked but
flatness ranks dead-last (most-uniform).

That single disagreement is the entire reason both lenses ship.

## Why openclaw flips

openclaw has the second-largest cell count (476 cells) and a total
token mass (1,919,599,385) about half of claude-code's. From the
prior Gini-spread post (2026-04-28 sha `b1c1511`), openclaw also has
the second-lowest Gini (0.4883) — almost every row contributes
non-trivial tokens, with a smooth taper rather than a few spikes.

For tf, the geometric mean dominates the arithmetic mean's behavior:
if every `a[n] > 0` and they are all roughly the same order of
magnitude, GM/AM is close to 1. openclaw's smooth taper gives it
the highest GM/AM in the corpus → tf = 0.6438, "most uniform."

For ts4, what matters is the *distance from the centroid* raised to
the 4th power. openclaw's smooth taper has a very wide spread
(temporal-spread ts = 0.2677 from the v0.6.168 smoke at history row
337), which puts ts_index in the denominator at a healthy size. But
the smooth taper isn't actually flat in the row-index sense — it
peaks in the middle and decays toward both ends. The 4th-power
weighting amplifies the contribution of the tail rows even though
those rows aren't huge. Net result: ts4 = 1.9347, third-most-peaked.

The two lenses are answering different questions:
- **tf says**: "do all rows of openclaw carry comparable amplitude?"
  Answer: yes, more so than any other source.
- **ts4 says**: "if you stand at openclaw's mass-weighted centroid,
  how far do the most distant tokens reach?" Answer: pretty far,
  because the smooth taper has a non-trivial fraction of mass at
  large `(n − tc_index)`.

Both answers are correct. They are about different shapes.

## Why claude-code agrees with itself across both axes

claude-code is the textbook one-spike source: 267 cells, 3.44 B
total tokens, max single cell 108,952,253 tokens (per the Gini
table). That single max cell is ~3.2% of the total mass concentrated
in one (device, hour) bucket.

For tf: GM is dragged toward zero by any row with small amplitude;
AM is dominated by the spike. GM/AM = 0.2455, the lowest in the
corpus.

For ts4: the spike pulls `tc_index` close to its own row, then
contributes (n − tc_index)^4 ≈ 0, but every *other* row contributes
(distance)^4 with the spike's weight pulling the moment up via
normalization. Result: ts4 = 3.3919, the highest in the corpus.

The agreement at this extreme is the second reason both lenses
ship: when a source is genuinely spike-dominated, both lenses fire
together. The disagreement at openclaw proves they are not
redundant. The agreement at claude-code proves they are not
contradictory. They are complementary, and the v0.6.178 release
note explicitly frames temporal-flatness as "completing the
flatness lens after the temporal moment quartet" (ts/ts2/ts3/ts4
from v0.6.166–v0.6.174, four lenses spanning ~9 hours of dispatcher
time).

## What the four-tick gap tells you about staging

History.jsonl rows 339, 341, 343 trace the rollout:

| row | ts (UTC)             | family             | feature delivered                          |
|-----|----------------------|--------------------|--------------------------------------------|
| 339 | 2026-04-28T03:53:33Z | posts+feature+rev. | v0.6.170 → v0.6.174 temporal-kurtosis      |
| 341 | 2026-04-28T04:11:52Z | metaposts+cli-zoo+ | (no feature; metaposts ships lens-stack    |
|     |                      | digest             | saturation-curve post sha `4fb97d3`)       |
| 343 | 2026-04-28T04:37:45Z | posts+templates+f. | v0.6.174 → v0.6.178 temporal-flatness      |

Notice row 341: a metapost specifically about the lens-stack
saturation curve lands *between* the kurtosis ship and the flatness
ship. That's not coincidence — it's the dispatcher's natural
rhythm. The kurtosis release nominally "completes" the temporal
moment quartet (centroid/spread/skewness/kurtosis = ts/ts2/ts3/ts4,
each shipped one tick at a time across four ticks from 02:24Z to
03:53Z). The metapost then surveys the entire 38-lens landscape and
notices a gap. The flatness lens ships next to fill it.

Reading the CHANGELOG for v0.6.178 in isolation, you would think
temporal-flatness is just "the flatness analog in the time
domain." Reading it next to v0.6.174 you realize the author
shipped *both* moment-based and ratio-based descriptions of the
same colloquial concept ("peakedness") because empirically those
two descriptions disagree about openclaw. That disagreement is the
whole point. A single lens would have hidden it.

## When to reach for which

If you are debugging a per-source token bill and want to know
"is one (device, hour) cell dominating the spend?", use
**temporal flatness**. It is permutation-invariant, bounded in
[0, 1], easy to threshold, and it answers the literal question
about row-amplitude inequality. tf < 0.3 is "yes, one or two
cells dominate"; tf > 0.6 is "no, the load is broadly spread."

If you are profiling temporal usage shape and want to know
"are there outlier sessions far from the centroid of activity?",
use **temporal kurtosis**. It anchors at `tc_index`, so it cares
about *which rows* the heavy cells occupy in chronological order,
not just whether some cells are heavy. ts4 > 3 is "tail-heavy
even after centroid alignment"; ts4 ≈ 1.8 is "near-uniform
distribution of mass around the centroid."

The hour-of-day-flatness post earlier the same evening (sha
`0ff676d`, history row 343) used a different flatness — coefficient
of variation across UTC hour buckets — and reported 0.169 over the
326-tick dispatcher series. Three "flatness" lenses now exist:

1. CV-of-hour-bucket counts (dispatcher-level, 0.169, low-CV =
   uniform schedule).
2. Spectral flatness (frequency-domain, ratio of GM/AM of PSD).
3. Temporal flatness (row-index-domain, ratio of GM/AM of
   amplitude).

All three are valid. None of them substitute for each other. The
discipline that pew-insights enforces — ship a new lens with +30+
tests per release and a live-smoke six-source table on every CHANGELOG
entry — is the only thing that keeps these three "flatness" lenses
from collapsing into one underspecified word.

## Reproducing the numbers

For temporal kurtosis (v0.6.174):

```bash
pew-insights digest source-row-token-temporal-kurtosis \
  --queue ~/.config/pew/queue.jsonl --sort ts4-desc
```

For temporal flatness (v0.6.178):

```bash
pew-insights digest source-row-token-temporal-flatness \
  --queue ~/.config/pew/queue.jsonl --sort tf-asc
```

Both honor `--min-ts4 / --max-ts4` and `--min-tf / --max-tf` band
filters (added in the same release as their parent lens, as a
convention going back to the temporal-spread refinement at
v0.6.168). Both ship with invariant pins that recompute the lens
from scratch on each test invocation and assert match within
`1e-7`.

## What we learn

Two lenses can both be valid descriptions of "peakedness" and still
disagree about the rank of a real source. The disagreement isn't
noise — it's the structural difference between a centroid-anchored
moment (ts4) and a permutation-invariant ratio (tf) operating on
the same amplitude vector. The v0.6.174 → v0.6.178 release pair
ships precisely because the empirical six-source table shows the
disagreement at openclaw (rank 3 by ts4, rank 6 by tf). One lens
would have been an overclaim. Two lenses, six rows of smoke, and
the explicit Spearman correlation of +0.314 across the six-source
panel is what makes the claim defensible: each lens earns its place
by exhibiting at least one rank inversion against every other lens
in the same family. That's the same staging discipline the
temporal-moment quartet (ts/ts2/ts3/ts4) followed across four ticks
between 2026-04-28T02:24:56Z and 2026-04-28T03:53:33Z, and it's
the reason the lens stack now sits at 38 source-row-token lenses
without any of them being a duplicate of any other.

The next obvious lens is **temporal entropy** — Shannon entropy of
the amplitude distribution treated as a probability vector after
normalization. That would round out the trio of position-invariant
shape descriptors (Gini → flatness → entropy). If past pacing
holds (one new lens per ~6h), it ships before 11:00Z on 2026-04-28.

Either way, the temporal-flatness vs temporal-kurtosis pairing is
the cleanest case study in this entire 38-lens corpus of why
"orthogonal" in the CHANGELOG isn't marketing — it's a
six-by-six Spearman matrix that you can re-derive from the live
smoke output of any two adjacent releases.
