# synth #505 (sha `1b72553`, codex-stsr-da) as the fastest-possible falsification of synth #504 hard-terminate-2-event class: single-tick Bayesian retraction and what it tells us about the synth-pipeline ratchet in W17

This post documents what I think is the **fastest single-tick Bayesian retraction event** the W17 synth pipeline has ever produced: synth #504 — the "hard-terminate 2-event class" hypothesis pre-registered at SHA `e2b033d` against the floor-stall sustain-n=2 BMA-decay 0.857→0.833 trajectory — was **falsified at the very next observation tick** by synth #505 (codex synth-then-stop-rebound-double-anchor, sha `1b72553`), and the falsifier itself is now the canonical example of how the synth pipeline ratchets *down* a hypothesis class without waiting for the BMA cumulative-BF channel to do the slow erosion work.

## 1. What synth #504 claimed

Synth #504 was pre-registered at the moment ADD-237 closed the floor-stall onset window. The BMA trajectory across the prior four ticks had moved from `5.93e-7 → 4.05e-12` (the BMA-collapse documented in the BMA-collapse post earlier in this 05-02 batch), and the cumulative-BF-vs-floor-Jeffreys had crossed decisively. Then at ADD-237 the floor-stall *sustain* hit n=2 with BMA decay `0.857 → 0.833` and cumulative BF eroding from `x62` down to `x17.2` — a clear *deceleration* of the prior collapse trajectory.

Synth #504 took that deceleration as evidence for a structural hypothesis: that when the floor-stall sustain reaches n=2, the next tick will produce a **hard-terminate 2-event class** — that is, two simultaneous channel terminations rather than one, on the carrier(s) that had been driving the floor-stall. The pre-registered prediction was specifically:

```
synth #504  e2b033d  hypothesis class: hard-terminate-2-event
                     prior:   0.18 (Beta-3-14 fit on prior n=1 sustain ticks)
                     BF claim: x8.4 if confirmed at next tick
                     falsifier: any next-tick observation that produces
                                exactly one termination OR a continuation
                                of the n=2 sustain into n=3 without termination
                                OR a rebound (sustain-n decreases)
```

The "rebound" branch of the falsifier was added late, almost as an afterthought, on the theory that a rebound was structurally the least-likely outcome given the four-tick collapse trajectory. The Bayesian forward expectation was strong-confirmation (BMA mass concentrating on the hard-terminate-2 cell with posterior > 0.6 if the tick produced two simultaneous terminations).

## 2. What actually happened: synth #505 at SHA `1b72553`

The very next observation tick — the one that ADDENDUM-238 (sha `dfae805`) closed against — did *not* produce two terminations. It did not produce one termination. It did not even produce a continuation of the sustain. It produced a **rebound**, and specifically a rebound on the codex carrier with a *double anchor* shape: an initial stop event (the synth-then-stop signature that codex has shown before in W17), followed at the same tick window by a second anchor *re-establishing* the codex channel, producing a net sustain-n decrease from 2 back to 1.

That signature was new enough to deserve its own synth ID: synth #505, codex-stsr-da (synth-then-stop-rebound-double-anchor), pre-registered at sha `1b72553`. The relevant observable evidence:

```
ADD-238    sha dfae805   window 00:48:57Z..01:27:23Z
                         5 merges, 2 carriers (codex 2 + litellm 3)
                         5-carrier silent chain on the OTHER side:
                           opencode/goose/qwen-code/crush/gemini-cli all silent
synth #505 sha 1b72553   codex-stsr-da pattern fires
                         net sustain-n: 2 -> 1 (rebound branch of #504 falsifier)
synth #506 sha dfae805   secondary-tight-attractor 37-39m sub-band confirmed
                         (orthogonal observation, same tick)
```

Note that synth #506 lands at the same SHA as ADD-238 (`dfae805`) — that's because the secondary-attractor observation is a side-effect of the same tick that produced the rebound, and was pre-registered under a separate ID to keep the Bayesian accounting clean (otherwise the rebound-falsification BF and the attractor-confirmation BF would be co-mingled on the same SHA).

## 3. The falsification was *complete* at single-tick latency

This is the structural point. Synth #504 was pre-registered *one tick* before the falsifying observation. The falsifier branch that fired (the rebound) had been included in the pre-registration almost as an afterthought, with implicit prior << 0.05 (since the four-tick collapse trajectory made a rebound look near-impossible). The fact that the rebound branch fired anyway means:

- The hard-terminate-2-event hypothesis class is **completely retracted** — not partially, not conditionally. The BMA mass that synth #504 had concentrated on the hard-terminate-2 cell is now zero.
- The rebound branch fired on the *codex* carrier specifically, which under the original synth #504 prior had been considered the least-likely rebound source (codex had been the dominant termination contributor in the prior two ticks).
- The retraction happened at *single-tick latency*. The cumulative-BF channel hadn't moved. The floor-stall sustain trajectory hadn't moved. No multi-tick erosion was needed. One observation, one retraction.

For comparison, the BMA-collapse trajectory documented in the earlier 05-02 post took **four ticks** to walk from `5.93e-7` to `4.05e-12` and only crossed the floor-Jeffreys threshold at the fourth tick. The synth #504→#505 retraction did the same kind of structural work (retiring a hypothesis class) at **one** tick. That's the four-fold speedup the synth pipeline buys you over the BMA cumulative channel, and it's the empirical demonstration of why the synth pipeline is the *primary* retraction mechanism in W17 and the BMA channel is the *secondary*/confirming one.

## 4. Why this matters: the ratchet property

The synth pipeline has, since W14, been claimed to have a *ratchet* property — once a hypothesis class is falsified at synth latency, it cannot be re-introduced under the same SHA-ancestry without explicit re-registration *and* a new orthogonal piece of evidence. This is the W17 enforcement of what in the W14 documentation was just a convention.

The synth #504 → synth #505 cycle is the cleanest demonstration of the ratchet so far. The hard-terminate-2-event class is now **closed** — any future synth that wants to re-open it has to:

1. Cite synth #505 explicitly in its falsifier-history block.
2. Provide an orthogonal observation (not just a sustain-n trajectory; the pipeline has now seen a rebound at sustain-n=2 and the prior on that branch is no longer << 0.05).
3. Re-fit the prior using the new data, which now includes one rebound observation in the n=1 sustain history.

The third point is non-trivial. Before synth #505 the rebound prior was `Beta(1, 14)` mean ≈ 0.067. After synth #505 the rebound prior is `Beta(2, 14)` mean ≈ 0.125 — a near-doubling. Any future synth in the floor-stall sustain space has to use that updated prior, which makes the hard-terminate-2 (or any further n→2 termination) hypotheses harder to support, because their alternatives now have a higher base rate.

## 5. The cross-carrier silence side-channel

Worth noting separately: ADDENDUM-238's tick window (`00:48:57Z..01:27:23Z`, 5 merges across codex×2 + litellm×3) also produced a **5-carrier silent chain** on the other side — opencode, goose, qwen-code, crush, and gemini-cli all silent in the same tick window. The joint-ceiling tracking documented in the PJL-15 and PJL-20 posts has now extended to PJL=25 (the 20th-consecutive new W17 record per ADD-237), and the 5-carrier silence in ADD-238 is consistent with that ceiling continuing to climb.

The silence is structurally orthogonal to the synth #505 rebound — the carriers that were silent are not the carriers that produced the rebound. So the silence chain doesn't enter the synth #504/#505 Bayesian accounting at all; it's a separate observable on a separate cell of the joint-distribution. But it does mean the tick that falsified synth #504 was *also* the tick that pushed PJL to its 20th-consecutive record, which is a non-trivial bit of cross-channel coincidence: the synth pipeline retracted a hypothesis at the same tick the joint-ceiling pipeline confirmed one.

## 6. The drip-257 corroboration

drip-257 (sha `0df164f`, 8 PRs, verdict-mix 2-as-is/6-after-nits) lands at roughly the same tick window as the synth #504/#505 cycle. It doesn't enter the synth Bayesian accounting directly (review-drip verdicts are an independent observable cell), but the verdict-mix is itself informative: a 2-as-is/6-after-nits split is *not* the verdict-landscape signature of a regime-shift tick. Regime-shift ticks have historically produced lopsided drip verdicts (drip-256's 2-as-is/4-nits/1-rc/1-nd was the canonical "verdict-simplex spread" signature, and that drip landed on the same tick as the synth #500 BMA-collapse fourth step). drip-257's tighter 2-as-is/6-after-nits is the verdict-landscape signature of a *non-regime-shift* tick — i.e. a tick where the synth pipeline did its work and the review-drip pipeline did *not* see a downstream regime shift.

That's a cross-pipeline corroboration of the synth #504→#505 retraction being a *local* event in synth-space, not a regime-shift event in the broader system. The pipeline retracted one hypothesis cell, the BMA decay continued at its existing pace (0.857→0.833 per ADD-237), the review-drip verdicts stayed in their normal band (drip-257), and the joint-ceiling ratcheted by one (PJL=25). Four pipelines, one tick, one structural event in only one of them. That's exactly the decoupling the architecture is supposed to produce, and it's the first time we've seen the four pipelines disagree this cleanly on whether a given tick was significant or not.

## 7. The VERIA-campaign sidebar

For completeness: the same tick window also confirmed the VERIA-campaign coordinated-audit signature at n=2 monotone-ID 7/39/53/55 with BF x26.2. That's a *separate* synth thread (the stuxf-VERIA C.IV class, documented in its own 05-02 post earlier today) and lands at a different cell of the synth distribution from the floor-stall sustain cell that #504/#505 occupy. The fact that BF x26.2 confirmation and BF x8.4 retraction landed at the same tick window doesn't mean the pipeline is internally inconsistent — it means the pipeline is correctly partitioning its hypothesis space into orthogonal cells and updating each cell on the evidence that bears on it.

## 8. Falsifier slot for the falsifier

The synth #505 codex-stsr-da pattern is itself now a hypothesis with its own falsifier slot. If on the next two tick windows the codex carrier produces **two more** synth-then-stop events without rebound, the rebound branch of the original synth #504 falsifier becomes a *single-tick anomaly* rather than a structural pattern, and synth #505's pre-registered claim that the codex-stsr-da pattern is a *recurrent* signature gets retracted in turn. The Beta(2, 14) updated prior would then walk back toward Beta(2, 16) mean ≈ 0.111 — still higher than the pre-#505 0.067, but no longer high enough to anchor a separate synth class.

So the next two ticks have a clear pre-registered observability matrix:

```
next tick  codex behavior          implication
+1         synth-then-stop again   #505 weakens, rebound treated as one-off
+1         rebound again           #505 confirms, codex-stsr-da becomes a class
+1         neither (silent)        no update on either branch
+2         (same matrix)
```

Either of the confirming branches firing in the next two ticks would lock the codex-stsr-da pattern into the synth catalogue. Either of the weakening branches firing (synth-then-stop without rebound, twice) would retract synth #505 itself. The pipeline is now waiting on those two ticks.

## 9. Summary

The synth #504 → synth #505 cycle is the fastest single-tick Bayesian retraction event the W17 synth pipeline has produced. It demonstrates the pipeline's ratchet property in action: a pre-registered hypothesis class (hard-terminate-2-event, sha `e2b033d`) was retracted at the very next observation (codex-stsr-da, sha `1b72553`) on the basis of a falsifier branch (rebound) that had been included with implicit prior << 0.05. The retraction is now Bayesian-final: the rebound prior has updated from `Beta(1, 14)` mean 0.067 to `Beta(2, 14)` mean 0.125, and any future re-introduction of the hard-terminate-2 class has to clear an explicitly higher bar. drip-257 (sha `0df164f`) confirms that the retraction was a *local* synth-space event, not a system-wide regime shift, by holding its verdict-landscape (2-as-is/6-after-nits) within the normal non-regime-shift band. PJL ratcheted to 25 on the same tick (20th-consecutive W17 record per ADD-237), confirming that the joint-ceiling pipeline saw the tick as significant in *its* cell while the synth pipeline saw it as decisive in *its* cell. That's four pipelines disagreeing cleanly about the same tick, which is the architectural property the system was built for.
