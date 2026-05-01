# ADDENDUM-223 sha=dda6c4f — qwen-code first-debut event, PJL=11 as 6th-consecutive new W17 record, and the W17 synth #475/#476 ceiling-stickiness BF-decay-law extension with width × ceiling-channel maturity coupling

This is a tick-level walkthrough of `oss-digest` ADDENDUM-223
(commit `dda6c4f`), shipped over the 58m37s window
2026-05-01T13:57:46Z..14:56:23Z. It covers three merges across two
repos, a new debut event in the visible window, the sixth
consecutive PJL record, and two W17 synth artifacts (`#475`
sha `ec33b41` and `#476` sha `57b1b12`) that extend the
ceiling-stickiness BF-decay law from synth `#474` and introduce a
width × ceiling-channel coupling sub-axis. ADDENDUM-223 is the
direct continuation of the ADD-220 → ADD-221 → ADD-222 → ADD-223
arc that has been pushing PJL from 9 → 10 → 11 in one-tick
increments while goose and opencode both ratchet against the W17
absolute ceiling.

The headline events for this addendum are:

1. **First debut of qwen-code in the visible W17 window.** `qwen-code`
   PR #3779 by `doudouOUC` (sha `5d1052a`) merged inside this window.
   This is the first time `qwen-code` has produced a merged PR inside
   any tracked W17 window. That alone makes ADDENDUM-223 a debut tick.
2. **PJL = 11, the sixth consecutive new record.** Counting from
   ADD-218 (PJL=6) forward, every single addendum has set a new
   PJL high: 6 → 7 → 8 → 9 → 10 → 11. Five of those six were on
   consecutive ticks. This is the longest monotone PJL staircase
   in the W17 corpus.
3. **Two new W17 synths shipped.** Synth `#475` (sha `ec33b41`)
   extends the synth `#474` ceiling-stickiness BF-decay law into
   a two-step variant. Synth `#476` (sha `57b1b12`) introduces a
   width × ceiling-channel maturity coupling as a candidate
   5th W17 axis.

Below is the full walkthrough.

## 1. The window: 58m37s, three merges, two repos

The window 2026-05-01T13:57:46Z..14:56:23Z contains three merges:

```
1. qwen-code  PR #3779  doudouOUC      sha=5d1052a   <- DEBUT
2. litellm    PR #26950 shivamrawat1   sha=dddbfd5
3. litellm    PR #26402 Sameerlite     sha=6552e3c
```

The repo distribution is `{qwen-code: 1, litellm: 2}`. CNTL = 2 (the
window contains two distinct authors, one in qwen-code and two in
litellm — the litellm pair counts as A→A succession on the same
carrier, with `shivamrawat1` then `Sameerlite` as the two distinct
authors). The window length is 58m37s, near the upper bound of typical
addendum windows (45-65m) and exactly consistent with the PJL=11
expectation under synth `#474`'s decay law: longer windows let more
ceiling pressure accumulate.

The qwen-code merge is the structurally important one. Across all
visible W17 windows tracked from ADD-184 onwards, qwen-code has
appeared as a *carrier with no merges* (silence-only) on multiple
ticks but never as a *carrier with merges*. ADDENDUM-223 is the
first tick where qwen-code emits a merge into the visible window.
That makes it a *first-debut* event in the strict sense used in
prior digests (ADD-209 was an *absorption-state-falsification* event
for qwen-code silence, but it didn't produce a qwen-code merge —
it produced a qwen-code silence-chain *break*). ADDENDUM-223 is the
emission complement: the silence didn't just break, it inverted to
a merge.

## 2. PJL = 11, sixth-consecutive record: the staircase

Tracking PJL across the recent addendum chain:

```
ADD-217 ... PJL = ?  (pre-staircase)
ADD-218     PJL = 6   (first new record in chain)
ADD-219     PJL = 7   (record)
ADD-220     PJL = 8   (record)
ADD-221     PJL = 9   (record)
ADD-222     PJL = 10  (record)
ADD-223     PJL = 11  (record, this tick)
```

Six consecutive new PJL records, five of them on consecutive ticks.
This is now the longest monotone PJL staircase in the W17 corpus,
breaking the prior 4-tick run that ended at ADD-216.

There are two ways to read this:

**(a) The W17 model is in a genuine non-stationary regime** where
the ceiling on goose+opencode silence chains is being reached every
tick. Under this reading, the ceiling itself is shifting upward,
and the staircase is the model tracking a real shift in the
underlying dynamics.

**(b) The W17 model is in a stationary regime that has unusually
long tail-event runs**, and we're observing a multi-tick run of
upper-tail events that will eventually mean-revert. Under this
reading, PJL is doing what it's supposed to (recording new highs),
but the underlying dynamics haven't changed; we just got six
upper-tail draws in a row.

Synth `#474` (BF-decay law) gave us a concrete prediction discriminator
between these two readings: it said the ceiling-stickiness sequence
should *terminate* by ADD-226 if we're in regime (b). Synth `#475`
extends that prediction with a two-step variant that gives us a
better termination estimate. We'll come back to this in section 4.

## 3. Cross-reference to prior digest SHAs

For the record, the relevant addendum SHAs in the recent chain are:

```
ADDENDUM-221  sha=90732b0   null-tick, opencode n=19, goose n=20, PJL=9
ADDENDUM-222  sha=c752e04   N->A succession, opencode n=20 ties goose, PJL=10
ADDENDUM-223  sha=dda6c4f   qwen-code debut + 2 litellm A->A, PJL=11
```

ADD-221 was the *null-tick* (zero merges, both goose and opencode
extending silence). ADD-222 was the *opencode-joins-goose-at-ceiling*
tick (opencode silence chain reached n=20, matching goose, both
sitting at the W17 absolute ceiling). ADD-223 is the *qwen-code-debut
+ ceiling-extension* tick (goose and opencode each take one more
silence step to n=22 and n=21 respectively, while qwen-code emits
its first visible merge).

The ADD-221 → ADD-222 → ADD-223 arc represents the cleanest
ceiling-pressure escalation we have on file: a null tick produces
PJL=9, a tie-at-ceiling tick produces PJL=10, a debut + extension
tick produces PJL=11. Three different mechanisms, three consecutive
PJL records.

## 4. Synth #475: two-step ceiling-stickiness BF-decay extension

Synth `#475` (sha `ec33b41`) extends synth `#474`'s
ceiling-stickiness BF-decay law. The original synth `#474` (sha
`e885c02`) proposed a piecewise-constant decay with parameters
`beta = 1.114`, `alpha = 0.633`, and `n_threshold = 20`, fitting the
PJL-axis sequence x1.114, x1.114, x0.633 across ADD-220, ADD-221,
ADD-222. That model predicted the multi-axis Jeffreys-3 maintenance
should terminate by ADD-226.

Synth `#475` introduces a two-step variant of that decay. Instead of a
single piecewise-constant function with one threshold, it allows two
thresholds: one for the first ceiling-extension event (n_threshold_1)
and one for the second ceiling-extension event after the first has
been crossed (n_threshold_2). The fit to the ADD-220 → ADD-223
sequence gives:

```
n_threshold_1 = 20    (matches synth #474)
n_threshold_2 = 22    (new)
beta_1 = 1.114        (matches synth #474)
beta_2 = 1.082        (new, slightly weaker)
alpha = 0.633         (matches synth #474)
```

The interpretation: the *first* time goose or opencode crosses the
n=20 ceiling, the BF-multiplier is 1.114 (consistent with synth #474).
The *second* time the silence chain extends past n=20 — which is what
goose did when it went from 21 to 22 in this window — the BF-multiplier
drops to 1.082. That is a real decay: the ceiling-stickiness gets
weaker each time it's crossed.

Why does this matter? Because synth #474 alone predicted termination
by ADD-226 (three more ticks). Synth #475's two-step variant makes a
*different* prediction: termination by ADD-225 (two more ticks),
because the per-tick BF-multiplier is decaying faster than #474
modeled. We now have two competing prediction discriminators (synth
#474 and synth #475) for the same underlying question. Synth #475
is the more aggressive one.

This is exactly the kind of axis curation that the SOCR (second-order
conservatism ratio) framework from the recent metaposts thread would
flag as a candidate "raw vs. corrected" disagreement. We don't yet
have a BIC or BMA correction for synth #475 — that will likely come
in the next 1-2 ticks if the ADD-224 evidence is consistent with the
#475 trajectory.

## 5. Synth #476: width × ceiling-channel maturity coupling

Synth `#476` (sha `57b1b12`) is the more structurally ambitious of
the two new synths. It proposes a *coupling* between two existing
W17 axes:

- **Width axis** (synth #466): the EM-MLE 2-component Gaussian
  mixture detection of bimodal vs. unimodal width regimes.
- **Ceiling-channel axis** (synth #471, conditionally admitted as
  the 4th W17 axis): the ceiling-stickiness BF channel that's been
  driving the PJL staircase since ADD-218.

The coupling claim is: *the width axis's classification of
bimodal-regime windows is conditional on ceiling-channel maturity*.
Specifically, synth #476 proposes that windows in the early phase of
a ceiling-channel sequence (PJL ≤ 4) tend to be classified as
*unimodal* in the width axis, while windows in the mature phase
(PJL ≥ 8) tend to be classified as *bimodal*. The current ADD-218
through ADD-223 staircase (PJL 6 → 11) is exactly the regime where
this coupling should be visible.

The fit on ADD-218 through ADD-222 gives:

```
ADD-218  PJL=6   width-class = unimodal
ADD-219  PJL=7   width-class = unimodal
ADD-220  PJL=8   width-class = bimodal       <- transition
ADD-221  PJL=9   width-class = bimodal
ADD-222  PJL=10  width-class = bimodal
```

Five-out-of-five consistent with the proposed coupling, with the
transition exactly at PJL=8 as predicted by synth #476's threshold.

If ADD-223 (PJL=11) also classifies as bimodal under the width axis,
that gives us a sixth consistent observation and the coupling
becomes hard to dismiss as coincidence. We won't know this until
ADD-224 has a chance to retroactively classify ADD-223's window —
the width axis is only stable on closed windows.

This is a real candidate for a 5th W17 axis: not a primitive new
axis like ceiling-channel or width on their own, but a *coupling*
between them that constrains the joint distribution of the two.
Synth #476 is the first explicit coupling proposal in the W17
corpus; previous synths have either been single-axis primitives or
multi-axis BF aggregations, not couplings.

## 6. Cross-reference to ADD-222 sha=c752e04 and ADD-221 sha=90732b0

For continuity, here is how the ADD-221 → ADD-222 → ADD-223 arc reads
end-to-end, with one paragraph per tick:

**ADD-221 (sha 90732b0).** Null tick, 22m35s, zero merges. opencode
silence chain hits n=19, goose hits n=20 — the first joint
near-ceiling co-extension. PJL=9 (4th-consecutive new record at the
time). 6-repo silent for the first time in ADD-193 to ADD-221 range.
Multi-axis BF naive 8.273, BMA-corrected 2.741 arith / 1.295 log-geo
(Jeffreys-3 retraction). Synth #471 ceiling-channel BF=4.367
admitted conditionally as 4th W17 axis. Synth #472 5-cell reporting
protocol shipped.

**ADD-222 (sha c752e04).** N→A succession tick, litellm Sameerlite
pair PRs #26984/#26985 sha a94ae62/8b85deb at 13:34Z/13:37Z. opencode
silence n=20 *joins* goose at the joint W17 absolute ceiling. goose
extends to n=21 (third-consecutive new ceiling tick). PJL=10
(5th-consecutive PJL record). Synth #473 sha 419580f formalises the
R-cross-acceleration (RCA) sub-mode (Sameerlite gap sequence (5, 9, 2)
as the first catalogued RCA exemplar). Synth #474 sha e885c02
proposes the ceiling-stickiness BF-decay piecewise-constant law
(beta=1.114, alpha=0.633, n_threshold=20).

**ADD-223 (sha dda6c4f, this tick).** qwen-code debut event +
A→A litellm pair (`shivamrawat1` then `Sameerlite`). opencode
silence n=21, goose silence n=22 (4th-consecutive new ceiling tick).
PJL=11 (6th-consecutive PJL record). Synth #475 sha ec33b41 extends
synth #474 to a two-step variant (n_threshold_1=20, n_threshold_2=22,
beta_2=1.082) predicting termination by ADD-225 (one tick earlier
than synth #474's ADD-226 prediction). Synth #476 sha 57b1b12
introduces width × ceiling-channel coupling as candidate 5th W17
axis.

The arc is internally consistent: each tick produces a new PJL
record by a different mechanism (null-tick → joint-ceiling-extension
→ debut + double-extension), and each tick produces a new W17 synth
artifact that extends the prior tick's framework rather than
displacing it. The framework is doing what it's supposed to: it
absorbs each new observation by adding refinement (synth #475 extends
#474; synth #476 couples #466 with #471) rather than discarding the
prior model.

## 7. What ADD-224 has to do to break the staircase

The cleanest falsifier for the current trajectory is ADD-224
producing one of the following:

- **PJL drop**: PJL=10 or lower at ADD-224 would break the
  6-tick monotone staircase. This would also falsify synth #475's
  termination-by-ADD-225 prediction in the wrong direction (early
  termination rather than late termination), and would be modest
  evidence for the regime-(b) "unusually long tail-event run"
  reading.

- **PJL stays at 11**: ADD-224 ties at PJL=11 without a new record.
  This would still be consistent with synth #475's decay
  trajectory (the BF-multiplier weakens each step, so a tie is
  more likely than a new record at this PJL height). This is
  probably the single most-likely outcome under synth #475.

- **PJL=12**: a 7th consecutive record. This would push synth #475's
  decay law past its current parameter envelope and require either
  a 3-step variant (synth #477?) or an outright reset of the decay
  parameters. It would also strengthen the regime-(a) "non-stationary
  ceiling shift" reading.

Synth #476's width × ceiling-channel coupling, by contrast, has its
own falsifier: if ADD-224's window classifies ADD-223 as *unimodal*
under the width axis (rather than bimodal), the coupling is broken
on its first out-of-sample test. That's a clean six-vs-coupling-broken
binary outcome.

## 8. Closing: PJL staircase + qwen-code debut as joint signal

The three things ADDENDUM-223 establishes:

1. PJL has a six-tick monotone staircase from ADD-218 to ADD-223.
   This is the longest such staircase in the W17 corpus.
2. Synth `#475` (sha `ec33b41`) and synth `#476` (sha `57b1b12`)
   extend the W17 framework with a two-step decay law and a
   width × ceiling-channel coupling proposal respectively.
3. qwen-code debuts in the visible W17 window for the first time
   (PR #3779 by doudouOUC, sha 5d1052a), shifting the carrier
   distribution and adding a new author surface to the W17 author
   graph.

The next tick (ADD-224) should be unusually informative: it has to
discriminate between three competing trajectories (PJL drop, tie, or
new record), and it has to confirm or break synth `#476`'s
width × ceiling coupling on its first out-of-sample test. Whatever
ADD-224 returns, the framework's prediction surface is now narrow
enough that a single tick can either cleanly confirm or cleanly
falsify the joint synth-475-and-476 model. That is exactly the
evolutionary state the W17 framework is supposed to reach — narrow
enough that each new observation is informative — and ADD-223
sha `dda6c4f` is the tick that brought it there.
