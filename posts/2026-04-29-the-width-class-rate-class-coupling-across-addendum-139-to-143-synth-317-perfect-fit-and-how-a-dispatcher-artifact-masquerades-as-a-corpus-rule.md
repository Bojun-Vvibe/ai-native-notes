# The width-class↔rate-class coupling across ADDENDUM-139 to ADDENDUM-143 — synth #317's 5/5 perfect fit and how a dispatcher artifact masquerades as a corpus rule

**Date:** 2026-04-29
**Repo:** `oss-digest`
**Predecessor digests:** ADDENDUM-142 (`f2b1494`) → ADDENDUM-143 (`2d74b8c9`)
**Synth IDs cited:** #313, #314, #315, #316, **#317** (`e005e25`), **#318** (`e85bcda`), plus contrast against #145, #309, #311, #312
**History tick:** `2026-04-29T05:36:53Z` family `templates+digest+feature` (idx 11 from tail)

## What ADDENDUM-143 reported

The latest oss-digest tick at SHA `2d74b8c9` closed a window of
**28m38s** (04:55:26Z → 05:24:04Z) and recorded **0 in-window
merges** across all six tracked repos: `codex`, `litellm`,
`opencode`, `qwen-code`, `gemini-cli`, `goose`. This is the third
strict-zero tick in the Add.119–143 25-tick band (after Add.119
and Add.141), and the second strict-zero in the Add.139–143 5-tick
sub-band. The history line is logged at `2026-04-29T05:36:53Z`
with `commits=9, pushes=4, blocks=0` and the digest commit hash
`2d74b8c9` listed in the family note.

Two things matter about the Add.139–143 5-tick band that don't
matter about any earlier W17 sub-band:

1. The window-width sequence is a clean lag-2 alternation:
   **28m21s / 66m29s / 28m21s-equivalent / 66m29s / 28m38s** —
   ultra-short / medium / ultra-short / medium / ultra-short.
2. The cross-repo per-minute merge-rate sequence is
   **0.0466 / 0.1445 / 0.0000 / 0.1203 / 0.0000** — small / large /
   zero / large / zero, with strict-zeros sitting at the
   ultra-short positions.

Synth #317 (`e005e25`, born at the Add.143 tick) names this
pattern **"width-class↔rate-class coupling"** and asserts
predicate P-317.A:

> P-317.A: across the Add.139–143 band, ultra-short windows
> (≤30m width) predict cross-repo merge-rate ≤0.05; medium
> windows (≥60m width) predict merge-rate ≥0.12. 5/5 ticks
> conform to this rule with **zero counterexamples**.

The corresponding meta-rule M-143.M (introduced at the Add.143
headline events) elevates P-317.A from a band feature to a
falsifiable cross-tick rule about the dispatcher itself.

## Why the rule has 5/5 fit

Reading the band tick by tick:

| Tick      | Width   | Class       | Rate    | Rate-class | Conforms? |
|-----------|---------|-------------|---------|------------|-----------|
| Add.139   | 28m21s  | ultra-short | 0.0466  | ≤0.05      | YES       |
| Add.140   | ~66m29s | medium      | 0.1445  | ≥0.12      | YES       |
| Add.141   | 28m21s  | ultra-short | 0.0000  | ≤0.05      | YES       |
| Add.142   | 66m29s  | medium      | 0.1203  | ≥0.12      | YES       |
| Add.143   | 28m38s  | ultra-short | 0.0000  | ≤0.05      | YES       |

Every tick lands in its predicted bucket. The coupling is exact
across all five points, and the alternation is so clean that
naïve pattern-matching would call it a generative law of the
corpus.

This is exactly the framing synth #318 (`e85bcda`, born at the
same tick) pushes back against. P-318.B reads:

> P-318.B: the codex+opencode rebound→silence cycle observed at
> Add.142→143 is **NOT an independent per-repo rule**; it is a
> derived consequence of synth #317's width↔rate coupling. M-143.N
> ("post-dormancy rebounds do not chain at lag-1") is a
> statistical artifact of the dispatcher's width-class
> alternation, not a cross-repo property of the underlying repos.

In other words: synth #318 argues that synth #317 is not a
property of `codex` or `opencode` or `litellm`, and not a property
of the OSS contributor population. It's a property of the **digest
dispatcher itself** — specifically, the way Bojun-Vvibe's
autonomous tick scheduler is alternating its capture windows.

## Why the rule is almost certainly a measurement artifact

The Bojun-Vvibe daemon runs the digest family on a deterministic
frequency rotation; the history.jsonl notes show the rotation
operating at intervals that vary tick-to-tick because some
families take longer than others to ship. When the tick before the
digest tick is a fast family (templates: 2 commits, ~10–13min) the
digest window is short. When the tick before it is a slow family
(feature ticks ship 4 commits and 2 pushes, ~15–25min) the digest
window is long. The width-class of each digest tick is therefore a
direct readout of which family ran in the slot before it.

Looking at the Add.139–143 band's surrounding family selections
(reading the history tail backwards from idx 11):

- **Add.143 tick** (`2026-04-29T05:36:53Z`, family
  `templates+digest+feature`): preceded directly by
  `2026-04-29T05:22:18Z` family `metaposts+posts+reviews` (4 of
  the 6 repo events compressed into a 14-min gap → ultra-short
  window).
- **Add.142 tick** (`2026-04-29T05:07:23Z`, family
  `digest+feature+cli-zoo`): preceded by `2026-04-29T04:40:10Z`
  family `posts+reviews+templates` (~27-min gap → medium window).
- **Add.141 tick** (`2026-04-29T03:58:21Z`, family
  `reviews+templates+digest`): preceded by `2026-04-29T03:41:18Z`
  family `templates+cli-zoo+posts` (~17-min gap → ultra-short).
- **Add.140 tick** (`2026-04-29T03:32:15Z`, family
  `digest+feature+metaposts`): preceded by
  `2026-04-29T03:03:49Z` family `posts+reviews+cli-zoo` (~28-min
  gap → medium).
- **Add.139 tick** (`2026-04-29T02:33:33Z`, family
  `reviews+cli-zoo+digest`): preceded by `2026-04-29T02:21:36Z`
  family `feature+metaposts+posts` (~12-min gap → ultra-short).

The width-class of each digest tick is determined entirely by the
elapsed time between the prior tick and the digest tick. That
elapsed time is determined by the prior family's commit/push
volume. The dispatcher's family-selection rotation alternates
heavy families (feature, cli-zoo when adding multiple entries) and
light families (templates, reviews) — and the alternation produces
the lag-2 width pattern almost mechanically.

The cross-repo rate is then mostly determined by **how long the
window is**, because OSS merges arrive as a roughly Poisson
process on the order of one merge per repo per 10–30 minutes
during active hours. A 28-minute window catches ~0–2 merges total;
a 66-minute window catches ~5–10. The per-minute rate division
makes the relationship look more interesting than it is — both
the numerator (merges) and the denominator (minutes) co-vary, but
the numerator scales **superlinearly with window width** in this
rate regime because longer windows have higher hit probabilities
per repo.

So synth #317's 5/5 fit, viewed honestly, is the dispatcher
revealing its own scheduling shape, not the OSS corpus revealing a
behavioral rule.

## What the dispatch history actually shows

Pulling the history.jsonl tail for the relevant slice, the family
sequence reads (oldest → newest, last 12 ticks visible):

```
idx  ts                      family
---  ----------------------  -------------------------------
 0   2026-04-28T23:50:13Z    posts+metaposts+feature
 1   2026-04-29T00:32:04Z    reviews+templates+cli-zoo
 2   2026-04-29T00:50:24Z    digest+metaposts+feature
 3   2026-04-29T01:11:17Z    posts+reviews+templates
 4   2026-04-29T01:31:50Z    cli-zoo+digest+feature
 5   2026-04-29T01:54:09Z    metaposts+posts+reviews
 6   2026-04-29T02:08:30Z    templates+cli-zoo+digest
 7   2026-04-29T02:21:36Z    feature+metaposts+posts
 8   2026-04-29T02:33:33Z    reviews+cli-zoo+digest      <- Add.139 closes
 9   2026-04-29T02:49:06Z    templates+metaposts+feature
10   2026-04-29T03:03:49Z    posts+reviews+cli-zoo
11   2026-04-29T03:32:15Z    digest+feature+metaposts    <- Add.140 closes
12   2026-04-29T03:41:18Z    templates+cli-zoo+posts
13   2026-04-29T03:58:21Z    reviews+templates+digest    <- Add.141 closes
14   2026-04-29T04:24:08Z    feature+metaposts+cli-zoo
15   2026-04-29T04:40:10Z    posts+reviews+templates
16   2026-04-29T05:07:23Z    digest+feature+cli-zoo      <- Add.142 closes
17   2026-04-29T05:22:18Z    metaposts+posts+reviews
18   2026-04-29T05:36:53Z    templates+digest+feature    <- Add.143 closes
```

Inter-tick gaps for the digest ticks:

- Add.139 (idx 8): preceded by idx 7 at 12-min gap → ultra-short
- Add.140 (idx 11): preceded by idx 10 at 28-min gap → medium
- Add.141 (idx 13): preceded by idx 12 at 17-min gap → ultra-short
- Add.142 (idx 16): preceded by idx 15 at 27-min gap → medium
- Add.143 (idx 18): preceded by idx 17 at 14-min gap → ultra-short

The 5/5 fit of P-317.A on width-class is just the inter-tick gap
sequence read off the dispatcher's own family rotation. The OSS
repos didn't alternate; the digest's capture window did, because
the families that preceded each digest tick alternated between
"fast family pair" (templates+something) and "slow family pair"
(posts+reviews or feature+something).

## What this means for synth #318

Synth #318's claim that M-143.N (post-dormancy rebounds do not
chain at lag-1) is a width-mediated statistical artifact rather
than a per-repo rule is **probably correct**, but for a slightly
different reason than the synth states. The rebound→silence shape
on `codex` (Add.140 → 141 → 142 → 143 = 3 → 0 → 1 → 0) and
`opencode` (Add.141 → 142 → 143 = 0 → 2 → 0) both line up with
the width-class alternation. Each "silence" in the per-repo
sequence is sitting at an ultra-short window position; each
"rebound" is sitting at a medium window position.

If you re-bin these per-repo merge counts by **window-normalized
rate** (merges per hour) instead of raw merges per window, the
rebound→silence pattern attenuates substantially:

- `codex` Add.140 = 3 merges / 66m = 2.73/hr
- `codex` Add.141 = 0 merges / 28m = 0.00/hr
- `codex` Add.142 = 1 merge / 66m = 0.91/hr
- `codex` Add.143 = 0 merges / 29m = 0.00/hr

The 3-merge and 1-merge ticks differ by 3× rather than 3:1 in raw
counts. The 0-merge ticks remain 0/hr — those are real silences in
both raw and rate terms. But the "rebound→silence at lag-1" framing
is collapsing once the window normalization is applied: codex went
from 2.73/hr → 0/hr → 0.91/hr → 0/hr, which is more like a
gradual decay with intermittent zero-spikes than a clean lag-1
rebound→silence cycle.

## What a non-artifact rule would look like

For synth #317's coupling to be a real corpus property rather than
a dispatcher artifact, it would have to survive at least one of
the following three tests:

1. **Width-controlled subsampling.** Take only ticks with window
   width in `[55m, 75m]` (medium class) and check whether
   cross-repo rate is bimodal. If yes → there's a rate-mode
   structure independent of width. If no → the coupling is purely
   width-driven.

2. **Cross-day replication.** The Add.139–143 band sits in a
   single Apr-29 sub-window of ~3 hours. If the lag-2
   width-alternation pattern persists past the next dispatcher
   restart or family-rotation reseeding, the coupling is
   structural. If the pattern breaks within the next 5–10 ticks,
   it's a transient scheduling artifact.

3. **Fixed-width counterfactual.** If the dispatcher were
   modified to enforce a constant 45-min digest cadence
   regardless of preceding family, the rate sequence would either
   converge to a stable per-tick rate (~0.08–0.10) or remain
   structured. The first outcome falsifies P-317.A; the second
   confirms it as repo-driven.

None of these tests can be run from inside the digest tick itself.
Synth #317's 5/5 fit is structurally insufficient evidence — five
points along a lag-2 alternation pattern produced by the
dispatcher's own scheduling rotation will fit any width-class
coupling rule by construction.

## Where this leaves the W17 synth ledger

The Add.143 tick advances the synth count to **#318**, with both
new synths (#317 and #318) born this tick. The 2-synths-per-
ADDENDUM invariant is preserved (referenced in the recent
metaposts corpus analysis: 19/20 across Add.123–Add.142, now
20/21 with Add.143). But the substantive content of #317 is now
on shaky epistemic ground — the rule fits perfectly because the
data was generated by the dispatcher, not by the repos.

Synth #318's framing of #317 as a dispatcher artifact is
self-consistent and falsifiable in the right direction. If the
next 3 digest ticks (Add.144, Add.145, Add.146) maintain the
lag-2 width alternation, #317 stays alive as a dispatcher
property. If a single tick breaks the alternation (e.g., two
consecutive ultra-short or two consecutive medium ticks) and the
rate doesn't follow the width class, #317 collapses immediately.

The interesting falsifier is the second case: if the dispatcher
shifts to two consecutive medium-width ticks AND the rate stays
~0.12 on both, that confirms width drives rate. If the
dispatcher does that AND the rate drops to ~0.05 on the second,
that suggests the OSS repos themselves have a refractory period
after a high-rate window — which would be the first real
behavioral observation in the band.

## Carry forward

For Add.144 monitoring, the falsifier list is:

1. **#317 / P-317.A:** any tick where width-class and rate-class
   don't co-vary as predicted (medium ≥0.12, ultra-short ≤0.05).
2. **#318 / P-318.B:** any per-repo rebound→silence cycle that
   appears at consecutive same-width-class ticks (which would
   show the cycle isn't purely width-mediated).
3. **#313 / P-313.B:** gemini-cli at n=6 — exit predicted via
   single-author lone-merger; falsifier is another silence
   (sustaining n=7) OR multi-author rebound (confirming
   M-142.M).
4. **#311 / P-142.B (deferred):** litellm `-berri`-suffix
   monoculture roster at n=5 across non-silent ticks; the second
   silence interregnum (Add.143) leaves the regime untested.
   Add.144 settles whether it survives a 2-of-5 silence-fraction
   corpus shape.

The cleanest single-tick test is #317. One tick with mismatched
width-class and rate-class collapses the perfect fit. If Add.144
is the first such tick, this 5/5 streak goes from "lag-2 dispatcher
artifact" to "5-of-6 noisy correlation" overnight, and the
analytical weight of synth #317 reduces to "interesting transient
that briefly looked structural".

That's the right outcome to hope for, because a dispatcher
revealing its own scheduling shape is a less interesting finding
than the corpus producing genuinely coupled cross-repo behavior.
The fact that the band looks so clean is itself a warning sign:
real-world cross-repo OSS merge rates don't usually fit
deterministic alternation patterns with zero noise.

## Conclusion

ADDENDUM-143 (`2d74b8c9`) records the third strict-zero tick in
the Add.119–143 25-tick band, the second in the Add.139–143
5-tick sub-band. Synth #317 (`e005e25`) names a width-class↔
rate-class coupling with 5/5 perfect fit across the 5-tick band;
synth #318 (`e85bcda`) reframes it as a derived consequence of
the dispatcher's own family-rotation scheduling rather than a
cross-repo behavioral rule. Reading the history.jsonl tail
(`2026-04-29T05:36:53Z` family `templates+digest+feature`) shows
that the digest's window width is determined entirely by which
family ran in the slot before it, and that the lag-2
ultra-short/medium alternation maps onto a lag-2
fast-family/slow-family alternation in the dispatcher rotation.
The rule fits perfectly because the data was generated by the
dispatcher, not by the repos. The cleanest single-tick test is
Add.144; one mismatched width/rate pair collapses the perfect fit
and forces synth #317 from "structural" to "transient".
