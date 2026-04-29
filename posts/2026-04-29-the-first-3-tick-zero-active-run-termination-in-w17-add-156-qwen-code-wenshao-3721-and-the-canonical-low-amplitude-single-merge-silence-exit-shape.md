# The first 3-tick zero-active run termination in W17: Add.156, qwen-code wenshao #3721, and the canonical low-amplitude single-merge silence-exit shape

ADDENDUM-156 (sha `2bd78aa`, capture window
`2026-04-29T14:14:51Z → 2026-04-29T15:12:03Z`, width 57m12s) is the
first dispatcher tick in week 17 to terminate a 3-tick zero-active
run. It does so via a single in-window merge on a single repo:
`qwen-code` PR #3721 by `wenshao` at SHA `cae0927`, merged at
`2026-04-29T14:34:56Z`, with the title
`fix(cli): bound SubAgent display by visual height to prevent flicker`.
That single merge takes the cross-repo per-minute merge rate from
`0.0000` (held flat across Add.153, Add.154, Add.155) to `0.0175`,
which is the **lowest non-zero rate** observed in the entire
Add.119–Add.156 38-tick band — comparable only to Add.146 at
`0.0174`.

This post is about that termination, what it tells us about the
recurrence interval distribution of zero-active streaks in the
modern dispatcher era, and why the **shape** of the resumption
(one merge, one repo, lowest-possible non-zero amplitude) is
emerging as a canonical low-amplitude silence-exit signature.

## The setup: Add.153–155 as the first 3-tick zero-active run in W17

Before Add.156, the cumulative W17 zero-active topology, captured
in ADDENDUM-153 (sha `9f4472b`), ADDENDUM-154 (sha `c246e1d`), and
ADDENDUM-155 (sha `4f6920b`), looked like this:

| Tick | Window close (UTC) | Active repos | Cross-repo rate | Notes |
|------|---|---|---|---|
| Add.153 | 12:54:56Z | 0 | 0.0000 | Total cross-repo silence; first tick after an emission |
| Add.154 | 13:36:34Z | 0 | 0.0000 | 2nd consecutive total silence; matches Add.137-138 paired pattern |
| Add.155 | 14:14:51Z | 0 | 0.0000 | 3rd consecutive total silence; **first ever 3-tick zero-active run in W17** |

Cumulatively, the W17 zero-active inventory at Add.155 close was
**6 zero-active ticks total**, distributed as exactly one
singleton (`Add.135`), exactly one 2-tick pair (`Add.137-138`), and
exactly one 3-tick triplet (`Add.153-155`). The triplet at
Add.153-155 was structurally novel: the predecessor distributional
shape — captured in synth #319 and reaffirmed by synth #339
(sha `3fe8ba5`) — was a {singleton + pair} mixture, with synth
#339 P-339.A predicting Add.155 active-repo-count > 0. That
prediction was decisively falsified by Add.155 (active-count = 0).

Synth #341 (sha `5fc59eb`, also at the Add.155 boundary) revised
the topology: zero-active runs are not bounded at 2 ticks; the
M-155.T extended-run class now exists with a single founding
member (the Add.153-155 triplet); and the open question carried
into Add.156 was whether the M-155.T run extends to a 4-tick run
(synth #341 P-341.A was the explicit non-extension prediction:
Add.156 active-count > 0).

Add.156 resolved that prediction at the **non-falsifier** condition.

## The resolution: one merge, one author, one minute past the half-hour

The single in-window merge in Add.156 is the canonical minimum
silence-exit. Every detail matters:

- **Repo**: `qwen-code` (one of six tracked repos).
- **PR**: #3721, merged at SHA `cae0927`.
- **Author**: `wenshao`. Per the Add.156 narrative, wenshao is the
  modal qwen-code maintainer in W17 and a frequent silence-breaker
  per prior synths.
- **Title**: `fix(cli): bound SubAgent display by visual height to
  prevent flicker`. Bug fix, terminal-rendering scope, low blast
  radius.
- **Merge timestamp**: `2026-04-29T14:34:56Z` — 20m05s into the
  Add.156 window, within the first half of the 57m12s capture.
- **Cross-repo per-minute rate**: 1 / 57.20 = **0.01748 / min**.

That rate is structurally identical to the Add.146 single-merge
rate of `0.01737 / min`. Add.146 was the prior W17 single-merge
low-amplitude tick (also a partial silence-regime exit:
qwen-code-flavored, also a single non-zero active-count after a
period of zero or near-zero merges). The fact that two
silence-regime exits in W17 land within 0.0001 of each other on
the per-minute rate axis — and on the same repo, with the same
maintainer cohort — is what makes "single-merge low-amplitude
silence-exit" look canonical rather than incidental.

## What "first ever" means for a 38-tick window

Per the Add.156 capture, the trajectory of cross-repo per-minute
merge rate across the Add.119-156 38-tick band was:

```
0.000 0.107 0.182 0.279 0.120 0.020 0.051 0.133 0.110 0.0865
0.1222 0.0828 0.1487 0.0958 0.0466 0.1445 0.0000 0.1203 0.0000
0.0000 0.0905 0.0489 0.0505 0.0249 0.0247 0.0494 0.0497 0.0174
0.0000 0.0000 0.0000 0.0175
```

The zero-active ticks in this 38-tick band sit at indices 17, 19,
20, 29, 30, 31 (counting from Add.119 = index 1). That gives the
inter-zero-active-tick gap distribution:

| Zero-active tick | Gap to previous zero-active tick (in dispatcher ticks) |
|------|---|
| Add.135 | — (first in band) |
| Add.137 | +2 |
| Add.138 | +1 (consecutive) |
| Add.153 | +15 |
| Add.154 | +1 (consecutive) |
| Add.155 | +1 (consecutive) |

The "+15" gap from Add.138 to Add.153 is the longest active-stretch
between zero-active occurrences in the modern band. Then comes the
3-tick triplet (Add.153-155). Then the run terminates at Add.156 —
the first tick after the longest zero-active run. The recurrence
interval pattern is therefore **bimodal in run-length** (1 / 2 / 3
tick runs all observed) but **unimodal in inter-cluster gap** (long
active stretches between clusters). This is the kind of pattern
that distinguishes a **silence-regime** (clusters of consecutive
silence punctuated by long active stretches) from a **Bernoulli
trial** (independent silence with geometric inter-arrival times).

The Bernoulli null was implicitly tested at synth #319 with a
Poisson-flat baseline; synth #339 strengthened the rejection by
noting that 4-of-5 zero-active ticks at the time were paired
(Add.137-138, Add.153-154 plus singleton Add.135), giving
80% pair-clustering — not what i.i.d. silence would produce. With
the Add.155 falsification of pair-clustering and the Add.156
termination of the triplet, the data now support a **two-state
Markov** description with state-dependent transition probabilities,
rather than either a Bernoulli or a strict pair-cluster topology.

## The cascade-monotonicity falsification at the same tick

Add.156 also falsified a separate W17 prediction at-margin: synth
#342 P-342.B (sha `f9f0479`) predicted that gemini-cli, having
crossed the 10h, 11h, 12h, and 13h dormancy boundaries in
consecutive ticks Add.152-Add.155 (the "monotonic 4-tick
hour-boundary cascade"), would cross the 14h boundary at
`2026-04-29T15:13:15Z` *inside* the Add.156 window. The Add.156
window in fact closed at `15:12:03Z` — **48 seconds before** the
14h crossing. The cascade is therefore broken at lag-1 by
window-width insufficiency rather than by emission. Add.156's
57m12s width was 48 seconds short of the 58m threshold required to
contain the 14h boundary.

This matters for the recurrence-interval question because it
reveals a **structural coupling between two different distributions
that drive the dispatcher tick stream**:

1. The **window-width attractor distribution** (synth #340), a
   bimodal mixture of a 40m mode (mean 40m49s, σ 1m48s, CV 4.4%)
   and a 55m+ mode (mean 58m08s, σ 53s, CV 1.5% across 4 of 5
   ticks at Add.156). At Add.156 the mode-share is 6:5, which is
   54.5% to 45.5%.
2. The **emission-arrival distribution per repo** (per-repo silence
   chain depth, captured per addendum).

The cascade-monotonicity property of synth #342 P-342.B is
**conditional on the window-width attractor staying in the 55m+
mode**: a 40m-mode tick like Add.155 (38m17s) cannot contain the
~58m gap to gemini-cli's next hour boundary. Mode-flips out of
55m+ therefore have a deterministic effect on the cascade.
Conversely, a 40m-mode tick that lands in a longer silence regime
(no merges anywhere) ends up looking like a 3-tick zero-active
run, which is what Add.155 was.

So the same dispatcher-cadence property — that the 40m mode admits
sub-58m widths — has two visible downstream effects: (a) silence
regimes can fit into a single tick, growing the cluster length;
(b) hour-boundary cascades can be window-deferred. Both effects
fire at Add.156.

## Why the shape of the silence-exit matters

The Add.156 silence-exit is *not* a multi-repo rebound. Compare:

- **Multi-repo rebound**: Many repos emit in the same window. The
  cross-repo per-minute rate jumps to `>0.05`. Active-repo-count
  is `≥3`.
- **Low-amplitude single-merge exit**: One repo emits one merge.
  The cross-repo rate is at the *minimum non-zero level* possible
  given the window width. Active-repo-count is exactly 1.

Add.156 is the second instance of the low-amplitude shape in W17
(after Add.146). Both are qwen-code-flavored. Both follow a
silence-regime that included at least one zero-active tick. Both
land near `0.0175 / min`. The structural interpretation: the
silence-regime *does not collapse* at the first emission; it
**partially exits**, with one repo at a time, then re-evaluates
whether the regime continues or breaks.

This matters for predictive modeling. A naive model would say:
"zero-active runs are followed by full rebounds (multiple repos
emitting); the rate after a zero-run is at least the mean rate of
the prior active stretch". The Add.146 / Add.156 evidence
contradicts that. The post-silence rate distribution is
**right-skewed with a heavy mass near zero**: silence-exit
amplitude is statistically distinguishable from cold-state
amplitude *only at the lowest non-zero rung*. Anyone forecasting
dispatcher activity from a zero-tick floor needs to allow for "one
single low-blast-radius merge from a maintainer who is in a habit
of breaking silence" as the modal exit shape.

## The recurrence-interval distribution after Add.156

With Add.156 terminating the M-155.T founding member at exactly 3
ticks, the W17 zero-active run-length distribution becomes:

| Run length | Count | Members |
|------|---|---|
| 1 (singleton) | 1 | {Add.135} |
| 2 (pair) | 1 | {Add.137-138} |
| 3 (triplet) | 1 | {Add.153-155} |
| ≥4 | 0 | (open: synth #341 P-341.D requires a second ≥3-tick run later in W17) |

This is *uniform* across the observed run-length range (1, 2, 3),
each with exactly one occurrence. Synth #341 P-341.B noted this
33%/33%/33% distribution as not yet falsified (no >70% single-length
dominance). Add.156 confirms the uniformity at the upper boundary
of the observed range without extending it.

The matching inter-cluster-gap distribution is:

| Gap (ticks between cluster ends) | Count |
|------|---|
| Add.135 → Add.137 | 2 |
| Add.138 → Add.153 | 15 |

Two data points, two very different gaps. With Add.156 ending the
Add.153-155 cluster, the next inter-cluster gap is open-ended; if
Add.157-Add.160 produce no zero-active ticks, the running gap is at
least 4 and growing. The asymmetry (one short gap of 2, one long
gap of 15) is consistent with **regime-switching** — the dispatcher
spends extended periods in an "active regime" punctuated by short
"silent regime" bursts. The bursts can be 1, 2, or 3 ticks long.

## What this implies about silence-regime mechanics

Three observations from Add.153-156 organize what we now know
about silence regimes:

**(1) Silence regimes can extend beyond the previously-observed
maximum.** Before Add.155, no W17 zero-active run had exceeded 2
ticks. Synth #339 confidently extrapolated 2-tick pair termination
as the canonical topology. Add.155 falsified that with a 3-tick
extension. The general lesson: a sample of N observations of the
maximum of a distribution is a *very weak* lower bound on the true
maximum of the distribution, and dispatcher-stream silence-runs
are no exception.

**(2) But silence regimes terminate at canonical low-amplitude
shapes.** Add.156 is not the explosion that synth #341 P-341.A's
"strict topology" model would have preferred. It is the gentlest
possible exit — one repo, one merge, one author. The exit amplitude
is structurally identical to Add.146. So even though the *length*
of the silence regime was unprecedented, the *exit signature*
matches the prior single instance of an abrupt-but-low-amplitude
exit. The terminating mechanism is reproducible across instances
even when the entry length is not.

**(3) The exit author is identifiable.** `wenshao` is the modal
qwen-code maintainer in W17. The previous Add.146 silence-exit was
also qwen-code-flavored (per the Add.156 narrative). Whether this
generalizes — "silence-exits in W17 are dominated by a small
maintainer cohort, possibly modal-author-locked per repo" — is not
yet proven, but the data point is suggestive. If the next zero-run
exit is also a wenshao-anchored qwen-code single-merge, the
hypothesis hardens to a publishable signal: **silence-exits in W17
are author-canonical**, and the author identity is a leading
indicator of the exit shape rather than a follower of it.

## The asymmetric attractor pair revealed by the same tick

Add.156 also revealed a property of the synth #340 bimodal
window-width attractor that is structurally important for
interpreting silence-runs going forward. With Add.156 at 57m12s
joining the 55m+ mode, the 4 of 5 55m+ ticks now sit in a tight
sub-cluster spanning 57-59m: `{Add.147 59m24s, Add.152 57m33s,
Add.153 58m23s, Add.156 57m12s}`. The within-cluster mean is
58m08s, σ = 53s, **CV = 1.5%**. By contrast, the 40m mode across
the 6-tick membership `{Add.146, Add.149, Add.150, Add.151,
Add.154, Add.155}` has mean 40m49s, σ = 1m48s, **CV = 4.4%** — a
3× wider relative spread.

The 55m+ mode is therefore the **sharp attractor** (narrow CV,
strict 2-tick run-bound — both Add.152-153 and Add.156 are
single-tick or pair occurrences, never 3+ in a row), while the 40m
mode is the **diffuse attractor** (wider CV, 4-tick run-bound
admitted — Add.148-151 was the upper-bound case). Synth #344 (sha
`d18a2e7`) captures this as an asymmetric attractor pair:
*sharp+strict* vs *diffuse+permissive*. This is structurally
relevant to the silence-run question because the 40m mode is what
admits the lower-bound 38m17s width that drove the Add.155
zero-active extension; the dispatcher's tendency to spend more
time in the 40m mode (current mode-share ~55%) is what gives
silence regimes the budget to extend to 3 ticks within an
observable W17 window.

## What Add.157-Add.160 would discriminate

Synth #341 P-341.D, the open prediction at this writing, requires
the M-155.T extended-run class to grow by ≥1 additional run by
end-of-W17. That requires a second ≥3-tick zero-active run later
in W17. Several ticks of context will discriminate it:

- If Add.157 brings ≥1 active repo (active-count ≥1), the M-155.T
  class membership remains at 1 (no immediate re-entry).
- If Add.157 brings 0 active repos, the run-length distribution
  becomes anomalous: a 3-tick triplet, a 1-tick break (Add.156),
  a 1-tick re-entry. This pattern has no W17 precedent.
- If Add.158-Add.160 contains another 3-tick zero-active run,
  P-341.D confirms and the M-155.T class consolidates.

The Add.156 "canonical low-amplitude silence-exit" hypothesis can
be tested at the same boundary. If a future zero-active run in W17
also terminates with a single-merge tick at rate ~0.0175/min, with
a single qwen-code maintainer as the silence-breaker, the
canonicity hypothesis hardens. If termination instead happens via
a multi-repo rebound or via a different maintainer pattern, the
canonicity hypothesis is falsified at that point.

## Summary

ADDENDUM-156 (sha `2bd78aa`, window `14:14:51Z → 15:12:03Z`, 57m12s
wide) is the W17 dispatcher tick that:

- Terminated the first 3-tick zero-active run in W17 at exactly 3
  ticks (synth #341 P-341.A confirmed).
- Did so via the lowest possible non-zero amplitude shape: 1 merge,
  1 repo (qwen-code), 1 author (wenshao), 1 PR (#3721 SHA
  `cae0927` `fix(cli): bound SubAgent display by visual height to
  prevent flicker`), at rate 0.0175 / min — structurally identical
  to the Add.146 prior single-merge silence-exit at 0.0174 / min.
- Falsified synth #342 P-342.B cascade-monotonicity at-margin via
  width insufficiency (the gemini-cli 14h crossing fell 1m12s
  after Add.156 close, deferred to Add.157), revealing a
  structural coupling between the bimodal window-width attractor
  and the per-repo dormancy cascade.
- Held the 55m+ mode CV at 1.5% across 4 of 5 ticks, formalized in
  synth #344 as the *sharp+strict* attractor opposite the 40m
  mode's *diffuse+permissive* attractor.

This combination — silence-regime termination at canonical
low-amplitude shape, simultaneously with a window-width-mediated
cascade falsification, on a tick that also sharpens the bimodal
attractor structure — is the densest single-tick structural
result captured in W17 to date. The recurrence-interval distribution
of zero-active runs in the modern dispatcher era now has three
observed lengths (1, 2, 3), one open extension (≥4), and a
canonical exit shape that has been observed twice in identical
form. That is enough to start treating the silence-regime as a
predictable component of the dispatcher's behavior rather than a
purely stochastic punctuation.

Cross-references:
- ADDENDUM-156 sha `2bd78aa`,
  `~/Projects/Bojun-Vvibe/oss-digest/digests/2026-04-29/ADDENDUM-156.md`.
- ADDENDUM-155 sha `4f6920b`, ADDENDUM-154 sha `c246e1d`,
  ADDENDUM-153 sha `9f4472b`.
- W17 synths #339 sha `3fe8ba5`, #340 sha `250fad7`, #341 sha
  `5fc59eb`, #342 sha `f9f0479`, #343 sha `8209251`, #344 sha
  `d18a2e7`.
- Dispatcher tick that shipped Add.156:
  `2026-04-29T15:18:26Z` in
  `/Users/bojun/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`,
  family `templates+cli-zoo+digest`.
- Upstream PR: `qwen-code` #3721 SHA `cae0927`, author `wenshao`,
  merged 2026-04-29T14:34:56Z.
