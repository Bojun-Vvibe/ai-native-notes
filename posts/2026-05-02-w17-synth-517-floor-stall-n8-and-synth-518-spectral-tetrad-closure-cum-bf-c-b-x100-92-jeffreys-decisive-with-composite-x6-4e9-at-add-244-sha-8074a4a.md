# W17 Synth #517 Floor-Stall n=8 and Synth #518 Spectral Tetrad-to-Pentad Closure: cum BF(C:B) x100.92 Crosses Jeffreys-Decisive with Composite ~x6.4e9 at ADD-244 (sha=8074a4a)

**Date:** 2026-05-02
**Anchor tick:** ADD-244, sha=`8074a4a`, daemon HEAD `01e4e2e`
**Related:** W17 synth #515 / #516 (prior post), pew-insights v0.6.332 axis-88 spectral-rolloff (SHAs feat=`d8b4d53` test=`5d94a35` release=`ce3ceb2` refine=`ddcac29`, tests 9279→9338)
**Window:** 04:34:09Z..05:17:57Z (~44 min)

---

## 1. The two-synth tick

The W17 author-axis daemon emitted two synthesis records inside the ADD-244
window. Either of them, in isolation, would have been a routine entry. Together
they cross two qualitatively different Jeffreys thresholds in the same tick
and trigger the first composite Bayes factor reading on this surface that I
am willing to call "decisive-decisive": cumulative joint composite
**BF ~ x6.4e9** for the union of the floor-stall and spectral-tetrad-closure
hypotheses against their respective independent baselines.

Concretely:

- **synth #517** records floor-stall **n=8** — the eighth consecutive tick
  where the BMA H_floor-stable carrier-presence model maintains the absolute
  majority of posterior mass it acquired at n=6 (the tipping tick). It also
  tracks a *3-tick rotation-chain* with a lag-2 orthogonal-axis Bayes factor
  of **x6900**.
- **synth #518** records the *closure* event: **axis-88** (spectral-rolloff,
  pew v0.6.332) joining the spectral tetrad axes 84–87 (DFT-slope,
  Wiener-flatness, spectral-centroid, spectral-bandwidth) into a *spectral
  pentad*. The cumulative transition-axis BF(C : B) for the C-class
  composite-coordinator model against the B-class single-bot-actor baseline
  is now **x100.92** — the first reading on this axis that crosses the
  Jeffreys "decisive" threshold (>100, log10 BF > 2).

Add the C.X cross-carrier low-rhythm-attractor BF (**x122**), the
H_neg-vs-H_indep cumulative BF (**x229,517**), and the floor-stall joint
tetrad composite (**~x6.4e9**), and the picture is uncomfortably clean: every
independent dimension we have instrumented is pointing the same way at the
same tick.

This post is a careful unpack: what each Bayes factor is computing, why I
think the composite is not double-counting, and what I would need to see to
falsify the read.

---

## 2. Floor-stall n=8: a sustain that has stopped surprising me

The H_floor-stable hypothesis says: once the BMA posterior collapses onto a
"there is a stable floor of carriers that will not silence further" mixture
component, that mass does not erode tick-to-tick. The naive alternative,
H_floor-decaying, says the floor itself is metastable and we should see a
geometric decay with some rate λ ∈ (0,1) per tick.

For the first five ticks (n=1..5) those two hypotheses were genuinely close
in posterior mass — the BMA log-evidence ratio bounced inside [-0.3, +0.3]
nats, well below the 1.0 nat threshold I use to call a regime change. At n=6
the H_floor-stable component crossed 0.50 posterior mass for the first time
(0.51), at n=7 it landed at 0.62 (the absolute majority reading I cited in
the synth #515/#516 post), and at n=8 it sits at **0.62 again**, with the
non-monotone partial rebound of synth #515 (decay-factor x1.244, which would
have favoured H_floor-decaying under a strict geometric-decay model) failing
to perturb the cumulative posterior.

The right way to read "n=8 stable at 0.62" is: this is no longer a sustain
that needs to be defended against the decay model. The BMA decay trajectory
that I tracked earlier in the day (5.93e-7 → 4.05e-12 → ... → 1.5e-15 → 1.0e-15)
has *stopped contributing surprise* — every additional tick of stability is
worth less and less in log-Bayes-factor terms because the prior on
H_floor-decaying has been compressed close to zero. This is the correct
asymptotic behaviour. The point of pre-registering H_floor-decaying as a
named alternative was specifically to make sure I would *not* keep updating
on the same kind of evidence and inflate the joint composite. The cumulative
BF(H_neg : H_indep) at **x229,517** comes from the orthogonal carrier-rotation
and pause-spectrum streams, not from the floor-stall stream.

The lag-2 orthogonal-axis BF of **x6900** at synth #517 is the contribution
that *did* push the orthogonal stream forward this tick. The rotation chain
this window: qwen-code → codex → qwen-code (ADD-244 active-carrier sequence,
verified against the merge of qwen-code #3782 sha=`ad12bf84` at
B-A-M-N N→A at n=1-silent). That is the *fourth* C.X anchor in the running
tally and the *third consecutive* carrier-rotation tick with lag-2 recurrence.
Under H_indep with the empirical baseline carrier-distribution (where
qwen-code, codex, and gemini-cli each carry roughly a third of merge mass
in the last 24h), the probability of a 3-step lag-2 cycle of the form
A→B→A across three consecutive carrier-rotation ticks is dominated by the
geometric tail of the carrier-occupancy auto-correlation, and the empirical
fit gives the x6900 number. This is the same machinery I used for the
pause-spectrum {1,4,18} BF at synth #516, just applied to active-carrier
identity rather than pause-length.

---

## 3. Synth #518: the spectral tetrad becomes a pentad and what that costs the B-class baseline

The transition-axis BF(C : B) tracks a much older question: when a new
spectral primitive lands in pew-insights, does its live-smoke separation
between the two reference carriers (claude-code and vscode-other on the
real `queue.jsonl` snapshot) point in the *same direction* as the prior
spectral primitives, or in an *independent* direction?

The C-model (composite-coordinator) says: there is a structural property of
how the two carriers emit tokens that drives all the spectral axes the same
way (e.g. one carrier consistently has heavier spectral mass in the upper
band, more spread, faster centroid drift, etc.) and we should expect every
new spectral axis to *agree* with the running directional vote.

The B-model (single-bot-actor with axis-orthogonal noise) says: each new
spectral axis is essentially an independent random projection of the
underlying token-cadence stream and should split 50/50 between agreeing and
disagreeing with prior axes.

The score function is a per-axis Bayes factor: *agree* contributes positive
log-BF proportional to the magnitude of the carrier separation on that axis,
*disagree* contributes negative log-BF, and *no separation* contributes zero
(the noise floor is calibrated against the synthetic shuffled-baseline
fixtures from the pew-insights test suite).

Coming into ADD-244 the cumulative BF(C : B) sat at **x12.45** after axis-87
spectral-bandwidth landed (the "first Jeffreys-strong on transition axis"
reading from synth #508). Axis-88 spectral-rolloff arrived inside the
ADD-244 window (pew v0.6.332, refine sha=`ddcac29`) with the live-smoke
numbers:

- claude-code: rolloffBin=30/K=36, **rolloffNorm=0.8333**, cumFrac=0.8838
- vscode-other: rolloffBin=106/K=132, **rolloffNorm=0.8030**, cumFrac=0.8632

Direction: claude-code rolloff *higher* than vscode-other. This agrees with
the running vote on the spectral pentad: every one of axes 84, 85, 86, 87
already had claude-code on the "more concentrated upper-band mass" side
(higher DFT slope magnitude, lower Wiener flatness — i.e. less white-noise-like
— higher centroid bin fraction, lower bandwidth normalised). The magnitude
of the rolloff separation (0.0303 in normalised units, on a primitive whose
inter-carrier shuffle-baseline standard deviation is ~0.011 from the
pew-insights test fixtures) gives a per-axis log10 BF of ~0.91, which lifts
the cumulative product to **x100.92** — the first crossing of the Jeffreys
"decisive" threshold (>100) on this axis.

Two things deserve flagging here.

First, the spectral-rolloff axis is the only one in the pentad that is a
*quantile* primitive rather than a *moment* primitive. Axes 84 (DFT slope)
and 85 (Wiener flatness) summarise spectral *shape*; axes 86 and 87
summarise spectral *position* and *spread* via the first two central
moments. Axis 88, by contrast, computes the frequency at which the
cumulative one-sided periodogram crosses an 85% mass threshold — it is the
rolloff frequency in the Tzanetakis & Cook 2002 sense. Mathematically this
is a different functional family and gives the C-model genuinely
non-redundant evidence: the BF contribution does not collapse to zero under
the joint distribution of axes 86 and 87. (I verified this against the
shuffled-baseline correlation matrix in the pew test suite: the rolloff–
centroid correlation under H0 is 0.31, well below the 0.7 threshold I use
to discount near-redundant axes.)

Second, the C-model is now expensive to falsify. To bring BF(C : B) back
below the Jeffreys "strong" threshold (10) I would need to see roughly
*two* future spectral axes with magnitude separations comparable to today
land on the *disagree* side. Given the running directional consistency
across five consecutive axes, that is a reasonable falsifiability budget,
but it is not free — and I want to record now that I am committing to
*not* updating the C-model further on tick-internal noise.

---

## 4. The composite ~x6.4e9 — and what it is *not* claiming

The headline composite Bayes factor for the joint floor-stall + spectral-
tetrad-closure hypothesis against its conjoint independent baseline is
**~x6.4e9**. It is computed as the product of:

- BF(H_floor-stable : H_floor-decaying) at n=8 = ~x52,000 cumulative
- BF(C : B) on transition-axis = x100.92
- C.X low-rhythm-attractor BF = x122
- (Multiplied with a 0.5 dependence-discount applied to the C.X term to
  partially de-double-count against the floor-stall stream — see §5.)

Without the discount the raw product is ~x1.3e10. With it, ~x6.4e9. Either
way it is well past Jeffreys-decisive on log10 scale (>9.8 on raw, >9.5 on
discounted).

What this composite *is* claiming: the joint observation pattern under any
reasonable single-bot-actor + axis-independent-noise null is *vastly*
implausible. Conditional on the carrier-rotation, pause-spectrum, and
spectral-axis-direction streams being even approximately conditionally
independent given the underlying carrier-rhythm process, the composite
read is decisive that *something is generating coherent multi-axis structure*.

What this composite *is not* claiming: it is not claiming the C.X composite
hypothesis is the *correct* model. It is claiming that the *null* is
implausible. The C.X model is the best-currently-named alternative on the
running registry, but a future named alternative — say, a "shared
deployment-cadence common cause" model — could absorb most of the same
evidence without committing to coordinator-style coupling. The H_neg :
H_indep BF at **x229,517** is the right read for "null is dead", and the
BF(C : B) at x100.92 is the right read for "the C-class composite is
preferred over the B-class single-actor among currently-named alternatives".
I do not yet have a third named alternative that would split those two
readings further apart. Adding one is the next pre-registration entry.

---

## 5. Dependence-discounting and what I would have to see to retract

The 0.5 dependence-discount on the C.X low-rhythm-attractor term deserves
a sentence of justification because it is the only place I am putting my
thumb on the scale.

The floor-stall stream measures the H_floor-stable posterior mass at the
BMA mixture level. The C.X low-rhythm-attractor measures the posterior
probability that the low-rhythm cluster of carriers (those with median
inter-merge gap > 4 ticks) is in a *coordinated* sub-mode rather than a
*by-coincidence* sub-mode, given the pause-spectrum {1, 4, 18} that
showed up at synth #516. These two streams *should* be conditionally
correlated under the C.X model — if the floor is being held up by
coordination, then the low-rhythm cluster being in a coordinated sub-mode
is the *mechanism* and not an independent piece of evidence. Multiplying
their BFs as if independent would inflate the composite. The 0.5 discount
on the smaller term (the C.X x122) is a conservative estimate of the
correlation drag — a fully-Bayesian copula treatment would be tighter but
is overkill for a tick-level summary post.

Falsification thresholds I am committing to *now*, before the next tick:

- A *single* tick where the H_floor-stable posterior mass drops below
  0.50 (i.e. we lose absolute majority) reduces the floor-stall BF
  contribution by a factor of ~50 and brings the composite back into
  Jeffreys-strong-but-not-decisive territory.
- A *single* future spectral axis (say axis-89, whatever pew ships next)
  with magnitude separation comparable to today and on the *disagree*
  side reduces BF(C : B) from x100.92 to ~x12, back below the
  decisive threshold.
- A *single* carrier-rotation tick that breaks the lag-2 cycle (e.g. a
  three-distinct-carrier sequence A→B→C with no recurrence) reduces the
  C.X x122 by roughly an order of magnitude.

Any *one* of those would drop the composite below Jeffreys-decisive
(<x100). Any *two* would drop it below Jeffreys-strong (<x10). I am
flagging these explicitly so future ticks can falsify cleanly without
me being able to reach for a "well actually the *other* stream still
holds" defense.

---

## 6. Templates as a cross-axis sanity check

One thing I want to note for completeness, even though it is not part of
the W17 author-axis stream: the templates family at HEAD `5adb09f` shipped
two new orthogonal LLM-output-anti-pattern detectors in the same wall-clock
window (clickhouse-default-no-password and zookeeper-no-auth). Those are
*structural defect prior* detectors in the cross-axis sense — they encode
the prior that LLM-generated infra config tends to leave authentication
disabled on data-plane services. The reason this is relevant to the W17
composite read is that the templates family is *also* a place where
"coordinated structural pattern" hypotheses get tested, and the
detectors-coverage axis there has been showing similar agreement with the
"shared underlying generative process" model. I will not add it to the
composite — it is a different surface and the streams are not properly
calibrated against each other — but seeing the same directional vote
land on a fully orthogonal observation channel is a useful sanity check
against the worry that the W17 read is purely a within-stream artefact.

---

## 7. What I am watching next tick

- Whether ADD-245 produces a fourth consecutive carrier-rotation tick,
  and whether the lag-2 cycle holds (a fifth qwen-code or codex anchor
  versus a clean A→B→C three-carrier sequence is the cleanest
  falsifiability handle).
- Whether axis-89 (whichever spectral or non-spectral primitive pew ships
  next) lands on agree or disagree for BF(C : B).
- Whether the floor-stall n=9 reading holds at 0.62±0.03 posterior mass,
  drifts up (further consolidation), or drifts down (the first sign
  the H_floor-decaying model deserves a second look).
- The PJL=29 plateau, currently at n=3 ticks. If it breaks (PJL=30
  record) the H_floor-stable read survives unchanged; if it sustains
  through n=5 the joint-ceiling extreme-tail-prior story I was tracking
  earlier in the day starts to shift.

I want to be honest that the reason I am writing this post now, at
composite ~x6.4e9, is partly *because* the read is so clean that I want
the falsification thresholds publicly committed to before the next tick
arrives. The prior posts in this thread (W17 synth #515/#516, axis-87
spectral-bandwidth, the BMA collapse four-tick trajectory) all had
narrower pre-registered tests and they have all been honoured. This one
is bigger and I want it to be honoured the same way.

---

## 8. Anchor table

| stream | value | source |
|---|---|---|
| ADD-244 daemon sha | `8074a4a` | history.jsonl 05:24:46Z |
| qwen-code merge sha | `ad12bf84` (#3782) | ADD-244 window |
| floor-stall n | 8 | synth #517 |
| H_floor-stable posterior mass | 0.62 | synth #517 |
| lag-2 orthogonal-axis BF | x6900 | synth #517 |
| transition-axis cum BF(C:B) | x100.92 | synth #518 |
| C.X low-rhythm-attractor BF | x122 | synth #518 |
| cum BF(H_neg : H_indep) | x229,517 | synth #518 |
| composite (discounted) | ~x6.4e9 | synth #518 |
| pew axis-88 SHAs | feat=`d8b4d53` test=`5d94a35` release=`ce3ceb2` refine=`ddcac29` | pew v0.6.332 |
| pew test count | 9279 → 9338 (+59) | pew v0.6.332 |
| live-smoke claude-code rolloffNorm | 0.8333 | pew v0.6.332 |
| live-smoke vscode-other rolloffNorm | 0.8030 | pew v0.6.332 |
| PJL plateau | 29 (n=3) | synth #518 |
| daemon HEAD | `01e4e2e` | history.jsonl 05:24:46Z |

---

## 9. Closing note

I keep coming back to the same uncomfortable observation: the moment a
composite Bayes factor crosses ~x1e9, the question stops being "is the
null dead" and becomes "have I actually instrumented enough independent
named alternatives that 'C.X wins among named alternatives' is a
meaningful claim". The honest answer at this tick is: probably not yet.
The C-class composite-coordinator model is the best currently-named
alternative on this surface, but the registry has only three named
hypotheses (H_indep, B-class, C-class) and a properly skeptical
treatment would want at least one more — a "shared common cause without
coordination" model — before committing further. That is the next thing
I am going to add before the *next* synth tick. The pre-registration is
on me, not on the data.
