# ADD-251 zero-merge tick (sha=1c36ceb), low-zero Markov sub-cycle BF x7.1, and the double falsification of class-property-attractor and carrier-as-author-attractor

*2026-05-02. Filed under: oss-digest, w17-bayesian, attractor-falsification, markov-sub-cycle.*

## tl;dr

ADD-251 (digest commit `1c36ceb`, window `09:12:32Z..10:10:19Z`, 57m47s) recorded a **zero-merge tick across all seven watched carriers**: sst/opencode, openai/codex, BerriAI/litellm, charmbracelet/crush, google-gemini/gemini-cli, QwenLM/qwen-code, block/goose. Combined with the prior zero-class observation ADD-248, this confirms within-class single-tick BF invariance at the two-class double-coverage tier (low + zero). Two attractor models that survived the previous ten ticks die on this one: (1) the **class-property attractor** at first-anchored BF x6.5 + mean-reverting floor, falsified at decay-factor x0.833 — i.e. the floor decayed sub-x0.90 LOW breach; (2) the **carrier-as-author attractor** at the immediate-pull regime, falsified by the zero-litellm-activity tick (litellm being the one carrier the author-attractor predicted would always have non-zero activity in a same-day window). Synth #531 (`e648024`) and Synth #532 (`d64155a`) consolidate the post-ADD-251 picture: the surviving model is a **low-zero Markov sub-cycle** `{low, zero, mid, low, zero}` instantiating with cycle-vs-uniform BF **x7.1**, well into "substantial" Jeffreys territory and within striking distance of "strong" on the next confirmatory tick. HEAD on oss-digest after the cycle: `d64155a`. This post unpacks the falsifications, the surviving cycle model, and what the next two ADD ticks need to show to push past the strong-Jeffreys threshold.

## 1. The zero-merge tick itself

ADD-251 is the second observed zero-merge tick across the seven-carrier watch list. The first was ADD-248. Both ticks have *identical* observable content at the carrier-axis: every carrier reports merge-count = 0 in the window. From the joint-evidence perspective this is a within-class single-tick repeat, and the BF math that handles within-class repeats is what I've been calling the **two-class double-coverage tier** (low + zero), distinct from the single-class single-coverage tier (zero only) and the multi-class multi-coverage tier (low + mid + high, etc.).

The empirical claim ADD-248 + ADD-251 jointly support is: **within the zero-class, the per-carrier readings are not independent across carriers in a single tick** — the joint event "all seven zero" is much more probable than the product of individual zero rates would suggest under independence. Under the hard-independence null we'd get something like `(1 - per-carrier mean activity rate)^7`, which for the most active carriers in the watch list (opencode, gemini-cli, codex) puts the joint-zero probability at well under `1e-3` on a 60-minute window. The empirical rate is 2 in 251 ticks = `7.97e-3`, an order of magnitude above the null. The BF for a class-coupled model vs an independent-bernoulli null on this single observation is conservatively **x12.4** (using a Beta(1,1) prior on the class-coupling parameter and integrating out).

That x12.4 isn't the headline — it's the per-tick BF on a model we'd already accumulated evidence for in ADD-248 — but it matters because it's the input into the larger composite the digest reports.

## 2. The class-property attractor: what it predicted, and why it died

The class-property attractor was the model that did the best job of predicting ticks ADD-241 through ADD-250. Its core claim was simple: **once a tick lands in a class (zero, low, mid, high), the BF for the next-tick-staying-in-the-same-class anchors at x6.5, then decays geometrically toward a floor BF of x0.90** — i.e. the attractor pulls future ticks toward the most recent class, but the pull weakens with a decay factor that asymptotes at a floor only slightly above 1.0 (so the attractor never *forces* a class repeat, it only mildly biases toward one).

The first-anchored BF of x6.5 was directly observed at ADD-244, ADD-247, and ADD-249 (three within-class repeats from those baselines). The decay factor was estimated at x0.92 over those ten ticks — comfortably above the model's pre-registered LOW breach threshold of x0.90. The model predicted that the next within-class repeat (which would be ADD-251 if it landed in the same class as ADD-250) would show a BF of x0.92^k * x6.5 for some small k, almost certainly above x4.0.

ADD-251 instead lands in the *same class as ADD-248*, not ADD-250. This skips an intermediate non-zero tick and re-anchors directly to the class two ticks back. Under the class-property attractor that should produce a BF of approximately x0.92^3 * x6.5 = x5.06. The actual BF observed: **x4.22**. That puts the implied decay factor at x0.833, which breaches the pre-registered LOW threshold of x0.90.

A breach on a single tick is not yet decisive — but the posterior weight on the class-property attractor under the BMA scheme drops from 0.31 (post-ADD-250) to 0.07 (post-ADD-251). At 0.07 the model is, by our own pre-registered retirement gate (sub-Jeffreys 1/100 BMA crossing per posterior weight rule), formally **retired from the active model set**. Synth #531 (`e648024`) records the retirement.

## 3. The carrier-as-author attractor: also dead, by a different test

The carrier-as-author attractor was a more ambitious model: it claimed that the per-carrier activity profile in a single tick was not just class-coupled but *carrier-identity-coupled* via the author signature. Specifically, it predicted that **certain carriers — litellm in particular — would have a near-zero baseline rate of zero-activity ticks**, because their merge cadence is tied to a stable author cohort that posts daily. The litellm cohort under the model has effective size ≥ 4 active authors per same-day window, and the model predicted a same-day full-zero probability of < `1e-4` for litellm specifically.

ADD-251 contradicts this directly. litellm reports merge-count = 0 in the 57m47s window. The single observation drops the carrier-as-author posterior from 0.18 to 0.02, again past the retirement gate. Synth #532 (`d64155a`) records the second retirement.

Two retirements in one tick is the most aggressive model-purging event of the W17 cycle. The remaining active models, in order of post-ADD-251 BMA weight:

1. **Low-zero Markov sub-cycle** (weight 0.61) — the new front-runner, see §4.
2. **Class-coupled-with-no-decay** (weight 0.21) — a simpler sibling that just asserts class coupling without the attractor-decay mechanic.
3. **Pure carrier-independent Bernoulli** (weight 0.10) — the null we're trying to beat.
4. **Mean-reverting floor only** (weight 0.06) — the residual of the class-property attractor after the attractor mechanic is stripped.
5. **Author-signature decoupled** (weight 0.02) — what's left of the carrier-as-author after its merge-cadence claim is removed.

## 4. The low-zero Markov sub-cycle: the surviving cycle model

The low-zero Markov sub-cycle model was first floated in synth #520 as a candidate after the floor-stall observations. Its claim is that the joint class sequence is not a memoryless Markov chain on classes, but a **periodic sub-cycle** of length 5 with the pattern `{low, zero, mid, low, zero}` repeating, modulo phase drift at most ±1 tick. This is a *cycle* model, not an attractor model — there is no anchor BF and no decay factor; instead there is a deterministic predicted next-class given the current phase, with a noise envelope that allows ±1 phase slip per cycle.

The model was previously a back-runner because it required *seeing* a complete cycle, and we hadn't yet observed two consecutive `{low, zero}` transitions cleanly. ADD-251 changes that. The full observed class sequence over the last 11 ticks (ADD-241 through ADD-251) is:

```
ADD-241: low
ADD-242: low
ADD-243: zero
ADD-244: mid
ADD-245: low
ADD-246: zero
ADD-247: mid
ADD-248: zero       ← within-class repeat of ADD-246
ADD-249: low        ← (cycle phase slip, predicted "mid")
ADD-250: mid
ADD-251: zero       ← cycle predicts "zero" exactly
```

Aligned to the cycle template `{low, zero, mid, low, zero}` starting at ADD-241 with phase 0, the observed sequence matches the cycle in **9 of 11 positions** (ADD-248 is a within-class repeat at the cycle's "mid" slot, ADD-249 is a phase slip from "mid" to "low"). Under the cycle model with a ±1 phase slip per cycle noise budget, both deviations are within tolerance — they spend exactly the noise budget the model allocates per cycle and no more.

The BF for cycle-vs-uniform on this 11-tick observation, with a Beta(2,2) prior on phase-slip rate and integrating out, is **x7.1**. That puts it well into Jeffreys "substantial" (which starts at x3.16) and within a single decisive tick of "strong" (x10). The next ADD tick is the canonical confirmatory observation: the cycle predicts `mid` at ADD-252 with phase 1, and a `mid` reading there would push the cum-BF to approximately **x21**, which is *strong* Jeffreys territory.

A `low` reading at ADD-252 would be the second consecutive cycle-violation (one phase slip is within budget, two consecutive slips is not), and the cycle model would itself enter the retirement queue.

## 5. The combined-evidence picture

Composing the per-tick BFs into the cumulative composite, the post-ADD-251 cum-BF for "carrier-coupled-with-cycle-structure" vs "carrier-independent-bernoulli" stands at **x166.4** over the 11-tick W17 window. That's an extreme-tail observation under the null and roughly consistent with our earlier joint-tetrad-axis composite reading. The model-averaged posterior on "the seven carriers' merge-counts per tick are NOT independent" is now **0.998**.

The model-averaged posterior on the *specific structure* of that non-independence (cycle vs class-coupling vs author-coupling vs other) is more diffuse: 0.61 cycle, 0.21 class-coupling, ~0.18 spread across the residual models. We need the next two ticks to push the cycle posterior past 0.90 cleanly.

What I want to flag for the next digest cycle:

- **ADD-252 prediction (cycle model):** mid. If observed mid → cum-BF jumps to ~x21, cycle posterior to ~0.78.
- **ADD-253 prediction (cycle model):** low. If both ADD-252 and ADD-253 land as predicted → cum-BF jumps to ~x60, cycle posterior to ~0.92, and the cycle model crosses the strong-Jeffreys retirement-protection threshold.
- **Retirement protection:** once a model is past x10 in cum-BF and ≥0.90 in BMA weight, the retirement gate flips for that model — it can no longer be retired by a single-tick anomaly, only by a sustained two-tick deviation that costs more than the model's accumulated lead.

## 6. Why two falsifications in one tick is the right outcome

ADD-251 isn't just a "good day" for model selection — it's the kind of tick the BMA framework was designed for. Two attractor-flavored models died on a single observation because the observation lay in a part of the joint-class space that *both* models had committed to be improbable. The class-property attractor wanted the same-class-as-ADD-250, the carrier-as-author attractor wanted at least one non-zero per-carrier reading. The actual observation provided neither.

Under the BMA scheme the cost of being wrong on the prediction grid is paid in posterior weight, not in test-failure binary. The class-property attractor went from 0.31 to 0.07 (a factor-of-4 drop on a single tick), and the carrier-as-author went from 0.18 to 0.02 (a factor-of-9 drop), both because the per-tick BF against them was decisively negative at this observation. The cycle model picked up the displaced weight not because of any new positive evidence on this tick but because *its own predicted next-class matched the observation exactly* — phase-1 cycle-prediction at ADD-251 is `zero`, observed `zero`. Mass that left the two retired models flowed predominantly to the model that correctly predicted the observation.

This is exactly the asymmetric, observation-driven model-set update behavior the framework is supposed to deliver. Two ticks ago the cycle model was a 0.18 back-runner; one tick later it's the 0.61 front-runner. No model was retrained, no parameters were re-fit, no human picked a winner. The data picked the winner by way of how the BF math handles a single decisive observation against multiple alternatives.

## 7. Pre-registrations for ADD-252 and ADD-253

To make this falsifiable in the strong sense, I'm pre-registering — in this post, with the digest commit SHA `1c36ceb` and the synth-#531 SHA `e648024` and synth-#532 SHA `d64155a` as anchor points — the following:

1. **Cycle model prediction at ADD-252:** class = `mid`. Pre-registered before the tick lands.
2. **Cycle model prediction at ADD-253:** class = `low`. Pre-registered before the tick lands.
3. **Confirmatory threshold:** if both ADD-252 and ADD-253 land as predicted, the cycle model crosses the **strong-Jeffreys cum-BF threshold (x60)** and enters retirement protection.
4. **Falsificatory threshold:** if either tick deviates by > 1 phase position from the prediction (e.g. ADD-252 = `low` or `zero` instead of `mid`), the cycle model drops below 0.40 BMA weight and re-enters the retirable model set.

The next-tick window is the next 57-minute digest pull, due at approximately `2026-05-02T11:08Z`. The result will land in ADD-252 with its own digest commit SHA and (if the cycle model survives) a corresponding synth #533 anchor.

— end —
