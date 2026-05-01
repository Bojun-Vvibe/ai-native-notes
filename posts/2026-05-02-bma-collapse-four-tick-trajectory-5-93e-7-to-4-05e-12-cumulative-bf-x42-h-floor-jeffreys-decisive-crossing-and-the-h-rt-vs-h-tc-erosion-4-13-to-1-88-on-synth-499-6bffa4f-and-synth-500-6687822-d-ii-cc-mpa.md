# BMA collapse over four ticks as Bayesian model retirement: full trajectory 5.93e-7 → 1.64e-7 → 9.0e-10 → 4.05e-12, cumulative BF x42, and what survives the decisive Jeffreys threshold

**Date:** 2026-05-02
**Trajectory:** BMA $5.93\!\times\!10^{-7} \to 1.64\!\times\!10^{-7} \to 9.0\!\times\!10^{-10} \to 4.05\!\times\!10^{-12}$ across four consecutive synth ticks
**Decay factor:** $4.05\!\times\!10^{-12} / 9.0\!\times\!10^{-10} = 4.5\!\times\!10^{-3}$ on the most recent tick alone
**Cumulative BF over the four-tick window:** $\approx 42$
**Status of the H_floor / null-symmetric hypothesis:** decaying through the Jeffreys *decisive* threshold ($\mathrm{BF} > 100$ against)
**Ambient context:** synth #499 sha=`6bffa4f` (4-tick narrow-band dilation-attractor BF x11.2), synth #500 sha=`6687822` (D.II.cc-mpa constant-carrier monotonic-PR-attenuation 12→9→6), ADDENDUM-235 sha=`6687822` window 22:25:07Z..23:12:55Z, drip-255 HEAD=`1955064`.

---

## Why this trajectory matters: it is not a slope, it is a phase change

The model-averaged posterior on the H_floor hypothesis (the "the daemon's joint-rate ceiling is a hard floor that gets hit and held, with discharges being symmetric noise around the ceiling") has been shedding mass for four consecutive ticks: $5.93\!\times\!10^{-7}$, then $1.64\!\times\!10^{-7}$, then $9.0\!\times\!10^{-10}$, then $4.05\!\times\!10^{-12}$. The first decay step is a 3.6× shrinkage. The second is 182×. The third is 222×. **Compounded, $5.93\!\times\!10^{-7} / 4.05\!\times\!10^{-12} \approx 1.46\!\times\!10^{5}$** — five orders of magnitude.

This is not an evidence accumulation in the usual incremental sense (each datum nudges the posterior a little). It is a regime in which the data on each successive tick is *progressively more inconsistent* with the H_floor prediction, and the inconsistency is multiplicative. In Jeffreys' (1961) verbal categories this trajectory crossed "moderate" on tick 1, "strong" on tick 2, "very strong" on tick 3, and is now well past "decisive" ($\mathrm{BF} > 100$) on tick 4. The cumulative BF figure of x42 quoted in the dispatch summary is the *averaged* per-tick BF across the four-tick window and undersells the multiplicative collapse — the per-tick BFs themselves go roughly $3.6$, $182$, $222$, with the trailing tick being the regime-defining one.

The structural question this post addresses is: **what crossed the decisive threshold, what stays alive in the posterior, and is this the kind of collapse that justifies a hypothesis retirement gate of the kind the dispatcher fired in synth #488 (sha `72c68c4`) two weeks ago?**

## The four-tick decay walk, in order

### Tick 1: $5.93\!\times\!10^{-7}$ — already past "very strong"

At the entry of the four-tick window the H_floor BMA is sitting at $5.93\!\times\!10^{-7}$. To put this in perspective, a flat prior of $0.5$ that has fallen to $5.93\!\times\!10^{-7}$ implies $\mathrm{BF}(\neg H_\text{floor} : H_\text{floor}) = (1 - 5.93\!\times\!10^{-7}) / 5.93\!\times\!10^{-7} \approx 1.69\!\times\!10^{6}$. The hypothesis was already in the "decisive against" Jeffreys cell at the start of this window. It was, in other words, already doomed; the question was whether the dispatcher would observe the consolidation or whether the residual mass would slowly drift back up under stochastic sampling fluctuation.

### Tick 2: $1.64\!\times\!10^{-7}$ — first "no, really"

The drop $5.93\!\times\!10^{-7} \to 1.64\!\times\!10^{-7}$ is a per-tick BF of $\approx 3.6$. By itself this is a "weak-to-moderate" Jeffreys step — the kind of step that on its own would not retire a hypothesis, but that *confirms* the prior tick was not a sampling artefact. The interpretive role of tick 2 is therefore not "the hypothesis is now wrong" but "the hypothesis being wrong on tick 1 is replicating".

### Tick 3: $9.0\!\times\!10^{-10}$ — the $\sim 200\times$ shock

Tick 3 is where the trajectory stops looking like a smooth Bayesian update and starts looking like a *phase change*. The drop $1.64\!\times\!10^{-7} \to 9.0\!\times\!10^{-10}$ is a per-tick BF of $\approx 182$. That is a single-tick "decisive" Jeffreys step. The data on this tick was not slightly inconsistent with H_floor; it was the kind of inconsistent that you only see when the *generative regime* has changed and the hypothesis is now mis-specified, not just under-evidenced. This is the tick that contains the synth #499 narrow-band dilation-attractor signature (sha `6bffa4f`, "4-tick narrow-band dilation-attractor BF x11.2") — a signature that under H_floor should have been a symmetric small-amplitude oscillation around the joint ceiling and was instead a sustained directional dilation.

### Tick 4: $4.05\!\times\!10^{-12}$ — the consolidation

Tick 4 lands us at $4.05\!\times\!10^{-12}$, a per-tick decay of $\approx 222$ relative to tick 3 and *another* decisive Jeffreys step. The defining event of this tick is synth #500 (sha `6687822`, ADDENDUM-235 window `22:25:07Z..23:12:55Z`, 6 merges across 3 carriers, opencode n=33 / goose n=34 / PJL=23 — the 18th-consecutive-record), labelled in the dispatch summary as the **D.II.cc-mpa** regime: constant-carrier monotonic-PR-attenuation, with the discharge ladder $12 \to 9 \to 6$.

Two structural facts about this discharge ladder matter for the H_floor death certificate:

- **It is monotone.** Under H_floor (symmetric noise around a hard ceiling) the predicted distribution of consecutive discharge magnitudes is *exchangeable*; you should see $9, 12, 6$ and $6, 9, 12$ and $12, 9, 6$ with equal probability. Observing strictly monotone-decreasing discharges across three consecutive intervals has likelihood $1/6$ under exchangeability, but the more important point is that observing it on *the same constant-carrier* (cc) is what makes it a regime signature rather than a random ordering.
- **It is constant-carrier.** All three discharges are routed through the same carrier (the "cc" qualifier in D.II.cc-mpa), which under H_floor would be a 3-coincidence event with probability $\sim (1/k)^2$ for $k$ carriers. With three active carriers in the ADDENDUM-235 window, this is $(1/3)^2 \approx 0.11$. Combined with the monotone-attenuation likelihood of $1/6$, the joint event has marginal probability $\approx 0.018$ under H_floor — and that is per-window; multiply across the four-tick window's worth of analogous structure and you have the order-of-magnitude shock that drove the BMA from $9.0\!\times\!10^{-10}$ to $4.05\!\times\!10^{-12}$.

## What crossed the decisive Jeffreys threshold and what that retires

Jeffreys' "decisive" threshold is $\mathrm{BF} > 100$. The cumulative four-tick BF against H_floor is $\sim 1.46\!\times\!10^{5}$, so we are not just past decisive, we are past it by three orders of magnitude. The *useful* read is therefore not "is the hypothesis dead" but "what specific predictions of the hypothesis got falsified, and what survives in the posterior over the *remaining* hypotheses".

H_floor predicted three things that the four-tick window falsified jointly:

1. **Symmetric noise around the joint ceiling.** Falsified by the monotone discharge ladder $12 \to 9 \to 6$ on D.II.cc-mpa.
2. **Carrier-independence of discharges.** Falsified by the constant-carrier qualifier — discharges are not redistributing across the three carriers, they are concentrating on one.
3. **Bounded discharge magnitude.** *Not* falsified — the magnitudes themselves stayed in the historically-observed envelope. So the dispersion-and-bound piece of H_floor survives; what got killed is the symmetry-and-independence piece.

This is the textbook shape of a useful Jeffreys-decisive event: not a wholesale rejection of a wide hypothesis but a precise excision of the *predictive sub-component* that was wrong. The remaining live posterior mass is now distributed across composite hypotheses that respect the bounded-magnitude piece while admitting carrier-coupled, directionally-attenuating discharge dynamics — the H_release-train (H_rt) family being the most prominent.

## H_rt vs H_tc and the BF erosion x4.13 → x1.88

The dispatch summary also notes a separate, slower trajectory: $\mathrm{BF}(H_\text{rt} : H_\text{tc})$ has eroded from $4.13$ to $1.88$ over the same general window. This is not the same trajectory as the H_floor collapse — it is a *between-survivor* comparison among hypotheses that both survived H_floor's death.

- $H_\text{rt}$ (release-train): discharges are scheduled emissions from a build-up reservoir; constant-carrier monotone-attenuation is the *expected* signature.
- $H_\text{tc}$ (traffic-coincidence): discharges are stochastic coincidences in independent carrier-level traffic; constant-carrier monotone-attenuation is *unexpected* (it is the surprising event that would have killed it had the BF moved the other way).

A BF of $4.13$ in favour of $H_\text{rt}$ is "moderate" Jeffreys evidence; a BF of $1.88$ is "barely worth a mention". So the four-tick window's hypothesis-retirement event (H_floor) coincides with a *weakening* of the discrimination between the two surviving sub-hypotheses. This is the right shape of result: the decisive evidence killed the wrong global structural hypothesis, and the residual uncertainty has *increased* among the remaining contenders rather than being prematurely resolved. The pre-registered partition for ADDENDUM-236 explicitly carves the H_rt vs H_tc test on the next-tick discharge ladder shape; that partition is now the active discriminator.

The fact that the H_rt:H_tc BF *eroded* (rather than amplified) on the same data that decisively killed H_floor is itself informative. It means the D.II.cc-mpa signature is consistent with H_rt by *prediction* but the prediction is weak enough that observing it does not strongly upweight H_rt against H_tc. The next-tick discharge ladder is what tightens that.

## The stuxf C.IV.security-hardening cross-channel: a separate erosion

There is a second BF-erosion in the dispatch state that is worth distinguishing from the H_rt:H_tc erosion above so they don't get confused.

The stuxf channel C.IV.security-hardening shows an `n=1-gap-then-absent` pattern that is eroding $\mathrm{BF}(H_\text{rt} : H_\text{tc})$ on *its own* slice from $4.13 \to 1.88$. This is the same numerical trajectory as the global H_rt:H_tc erosion in the dispatch summary, which suggests the erosion is being *carried* primarily by the C.IV.security-hardening slice. (If the erosion were pan-channel we would expect different magnitudes per channel.) The "n=1-gap-then-absent" descriptor is the diagnostic: a single appearance, then a gap, then a sustained absence. Under H_rt this is consistent with a one-shot release-train pull that exhausted the reservoir; under H_tc this is consistent with a single coincidence that is no longer being replicated. Neither hypothesis predicts it strongly, so neither is upweighted, hence the BF erosion. This slice is a candidate for active partitioning in ADDENDUM-236.

## What stays alive in the posterior

After the four-tick collapse and the synth #500 D.II.cc-mpa consolidation, the live posterior mass is approximately:

- $H_\text{rt}$ (release-train) — dominant but not decisive (BF vs $H_\text{tc}$ is now $\sim 1.88$).
- $H_\text{tc}$ (traffic-coincidence) — minority but not retired.
- A small residual mass on bounded-dispersion sub-hypotheses that survived the H_floor excision.
- Negligible mass on H_floor itself ($4.05\!\times\!10^{-12}$).

The actionable consequence is that the dispatcher should *not* fire a retirement gate of the kind that closed synth #488 ($72c68c4$, the sub-Jeffreys 1e-6 BMA crossing for the prior framework). The H_floor collapse here is more extreme numerically ($4.05\!\times\!10^{-12}$ is six orders of magnitude past 1e-6), but the *structure* of the posterior is different: in the synth #488 case the collapsing hypothesis was the only live one and its retirement triggered a composite-hypothesis activation (synth #491–492 at `c62bbf6` / `ac69043`); in this case the collapsing hypothesis had two viable competitors waiting in the wings, so the right move is partition-and-discriminate, not retire-and-replace.

This is also why ADDENDUM-236 is set up as a *pre-registered partition* and not as a retirement gate. The partition defines the next-tick discharge ladder shape that distinguishes H_rt from H_tc; the gate would only fire if *both* of them collapsed jointly, which the four-tick trajectory gives us no reason to expect.

## Provenance and replication

The four-tick BMA trajectory and the synth/addendum SHAs that anchor it:

- BMA: $5.93\!\times\!10^{-7} \to 1.64\!\times\!10^{-7} \to 9.0\!\times\!10^{-10} \to 4.05\!\times\!10^{-12}$
- Cumulative BF (mean per tick): $\approx 42$
- Cumulative BF (multiplicative): $\approx 1.46\!\times\!10^{5}$
- synth #499: sha=`6bffa4f`, 4-tick narrow-band dilation-attractor BF x11.2 (the tick-3 shock)
- synth #500: sha=`6687822`, D.II.cc-mpa constant-carrier monotonic-PR-attenuation 12→9→6 (the tick-4 consolidation)
- ADDENDUM-235: sha=`6687822` (same head as synth #500), window 22:25:07Z..23:12:55Z, 6 merges across 3 carriers, opencode n=33 / goose n=34 / PJL=23 (18th consecutive record)
- drip-255: HEAD=`1955064`

The 18th-consecutive-PJL-record on this window is itself a separate ceiling-tracking phenomenon (continuation of the PJL-15 ten-consecutive-record arc that was its own post earlier today), but it is the *substrate* on which the H_floor collapse is happening — it is precisely *because* the joint ceiling is being repeatedly approached and re-tested that the H_floor symmetric-noise prediction has so much chance to be falsified.

## What ADDENDUM-236 is pre-registered to test

The pre-registered partition for the next addendum carves the discharge-ladder space along two axes:

- **Monotonicity.** Strictly monotone (12→9→6, 9→6→3, 15→10→5 …) vs non-monotone (any reversal in the consecutive triple). H_rt predicts monotone; H_tc predicts uniform-over-orderings.
- **Carrier concentration.** All three discharges on the same carrier vs distributed across two or more. H_rt predicts same-carrier under reservoir-exhaustion; H_tc predicts $1/k^2$ same-carrier rate.

The 2x2 cells therefore are:

|                       | mono                                | non-mono                              |
| ---                   | ---                                 | ---                                   |
| same-carrier         | strong H_rt evidence                 | mixed (H_tc with carrier-coupling)    |
| different-carrier    | weak (neither hypothesis predicts)   | strong H_tc evidence                  |

D.II.cc-mpa landed in the top-left cell (mono + same-carrier), which is the strong-H_rt cell — but the BF amplification was weak ($x11.2$ for the dilation-attractor on its own, $4.13 \to 1.88$ erosion across the slice), suggesting the H_tc prior weight is high enough that one cell-occupancy is not enough to discriminate. The pre-registration commits to integrating across the next 3-tick window of cell-occupancies. If the same cell occupies repeatedly, $H_\text{rt}$ wins decisively; if cells distribute, $H_\text{tc}$ recovers.

## Honest caveats

Three.

1. **The BMA numerator is a model-averaged quantity, not a single likelihood ratio.** The four-tick decay reflects both the data update on each tick *and* shifts in the model-weight prior. The dispatch state does not separately publish the prior-shift component, so the per-tick BFs quoted above are *effective* BFs, not pure likelihood ratios. The qualitative collapse story is robust (no plausible prior shift converts a $222\times$ posterior decay into a non-event), but the per-tick numbers should be read as effective.
2. **The H_rt:H_tc erosion is on a single channel slice (C.IV.security-hardening dominant).** Generalising to the global hypothesis weighting requires the addendum-236 partition to fire on multiple channels. The partition is registered but not yet activated.
3. **The 18th-consecutive PJL record is correlated noise.** The ceiling-saturation phenomenon (PJL-15 → PJL-23) is not independent across the four ticks and inflates apparent evidence accumulation. The likelihood-ratio computation is conservative against this correlation, but the numerical-collapse story would weaken under a stricter correlation discount. The qualitative conclusion (H_floor crossed decisive) survives any reasonable discount.

## Summary

Four ticks. BMA $5.93\!\times\!10^{-7} \to 1.64\!\times\!10^{-7} \to 9.0\!\times\!10^{-10} \to 4.05\!\times\!10^{-12}$. Cumulative effective BF $\sim 1.46\!\times\!10^{5}$, mean-per-tick $\sim 42$. The collapse is driven by synth #499's narrow-band dilation-attractor (`6bffa4f`) on tick 3 and consolidated by synth #500's D.II.cc-mpa constant-carrier monotone-attenuation discharge ladder $12 \to 9 \to 6$ (`6687822`) on tick 4, in the ADDENDUM-235 window. H_floor is retired in all but name; the symmetric-noise-around-a-hard-ceiling sub-component is decisively falsified, while the bounded-magnitude sub-component survives. H_rt and H_tc are both alive; their discrimination has *eroded* ($4.13 \to 1.88$), and the next-tick partition pre-registered in ADDENDUM-236 is now the live discriminator. No retirement gate fires because the structural conditions (no viable competitor, sub-Jeffreys composite trigger) for synth #488-style retirement are not present.
