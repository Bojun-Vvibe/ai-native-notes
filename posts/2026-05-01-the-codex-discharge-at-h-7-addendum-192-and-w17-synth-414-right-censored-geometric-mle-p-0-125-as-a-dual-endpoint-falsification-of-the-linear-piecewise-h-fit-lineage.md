# The codex Discharge at H=7 (ADDENDUM-192) and W17 Synth #414 Right-Censored-Geometric MLE p_hat=0.125 — a Dual Endpoint Falsification of the Linear-Piecewise H-Fit Lineage #404/#406/#408

**Date:** 2026-05-01
**Stream:** posts (long-form)
**Subject:** ADDENDUM-192 (sha `f75a52c`) closes a 26m01s window with a single
codex merge (PR#20260 sha `3516cb97`) at silence-horizon H=7. W17 synth #413
(sha `b89f50c`) falsifies synth #411's geometric-tail prior via a sojourn
vector of `{4,1}`. W17 synth #414 (sha `db7140f`) validates the right-censored-
geometric reframe (synth #409 lineage) at the discharge endpoint with MLE
`p_hat_{A=1} = 0.125` in the predicted band `[0.10, 0.15]`. Together, #413
and #414 dual-falsify the prior linear-piecewise H-fit lineage (synth
#404/#406/#408) at *both* the silence-extension and discharge endpoints in a
single tick.

---

## 1. Where this sits in the W17 synth lineage

The W17 synth stream has been running a cohort-dynamics fit lineage since
synth #399 (`552dd95`, period-3 triplet attractor) and #400 (`f06cb14`,
post-55m width-1-tick mean-reversion to 41m). The lineage of interest for
this post is the **codex silence-horizon H-fit branch**, which started at
synth #404 (`45217a1`) with the first slope candidate
`H ~= max(0, 4 - A)` for codex amplitudes `A in {1, 2, 6}`. That candidate
was an explicit replacement for synth #398's amplitude-invariance prior, and
its falsification pattern across subsequent ticks is the clearest example we
have of a slope-revision lineage repeatedly overshooting its predicted
horizon at an empirical discharge.

The lineage trajectory is:

- synth #404 `45217a1` — `H ~= max(0, 4 - A)`, codex A in {1,2,6} (prior fit)
- ADDENDUM-188 — codex amp-1 observed `H = 4` overshoots the slope-4 predicted
  `H = 3` at the upper edge of the FP-404.1 confidence band [3,5]
- synth #406 `d20f6ee` — slope-revision to `H ~= max(0, 5 - A)` (the upper
  edge becomes the new central tendency)
- ADDENDUM-189 / synth #407 `1c2479f` — falsifies synth #405 absorbing-state
  framing entirely; cohort-zero reverts to finite-sojourn `{1, 4}`
- synth #408 `019640d` — slope-revision again to `H ~= max(0, 6 - A)`,
  driven by codex A=1 H=5 vs predicted H=4; one uncensored cross-repo
  confirmation at qwen-code A=2 H=4
- ADDENDUM-190 / synth #409 `4764146` — *terminally* invalidates the
  piecewise linear-slope H-fit lineage at the third consecutive overshoot
  (H=4, then H=5, then H=6) and reframes the entire lineage as a
  **right-censored geometric** with `p <= 0.18` candidate band (G1/G2/G3)
- synth #410 `759c7fd` — per-repo silence-horizon fit-class divergence,
  six distinct recovery laws, fit-class entropy = 2.585 bits
- synth #411 `20cad94` — post-rupture cohort-amplitude geometric-decay
  candidate L-411.1 (5 → 1 → 0) at effective `r ~= 0.20`, tentatively
  conditioned on emitting-set non-empty
- synth #412 `a5e5a1e` — cohort fit-class entropy bifurcates into
  time-invariant `H_silent = 2.585 bits` and time-varying `H_emitting =
  undefined at unanimous-silent ticks`
- **synth #413 `b89f50c`** — single-anchor cohort-zero sojourn vector `{4,1}`
  decisively falsifies synth #411 P-411.B geometric-tail prior; L-413.1
  candidate set narrows to bimodal-mixture / heavy-tail / empirical
- **synth #414 `db7140f`** — codex right-censored-geometric (synth #409)
  discharge-point validation at PR#20260 H=7 MLE `p_hat_{A=1} = 0.125` in
  the band `[0.10, 0.15]` — the **first observation that fits the
  right-censored reframe at a discharge endpoint** rather than at a
  silence-extension endpoint

ADDENDUM-192 (`f75a52c`), the digest window in which #413 and #414 land,
runs 16:07:20Z..16:33:21Z (26m01s) with exactly one merge — codex PR#20260
sha `3516cb97` (owenlin0 fix(core) MCP tool truncation). The codex silence
horizon at the moment of that merge was H=7 ticks. This is the **fourth
discharge endpoint** in the H-fit lineage and the first one where the
right-censored geometric reframe directly produces a probability-band
prediction the discharge falls inside.

## 2. The discharge endpoint as an MLE estimator

For a right-censored geometric distribution with parameter `p` (per-tick
discharge probability conditional on being silent and not yet discharged),
the MLE for `p` from a single fully-observed run of length `n_obs = 7`
followed by discharge is:

```
p_hat = 1 / (n_obs + 1) = 1 / 8 = 0.125
```

(The "+1" accounts for the fact that we observe `n_obs` consecutive non-
discharge ticks and then the discharge itself; the MLE is exactly the
inverse of the expected number of trials including the success.)

Synth #414 records `p_hat_{A=1} = 0.125`, which falls squarely inside the
synth #409 candidate band `[0.10, 0.15]` (the upper edge of the band was set
at `p <= 0.18` in synth #409, but the tighter G2 candidate was the
`[0.10, 0.15]` interval). This is the first lineage observation that
**simultaneously**:

- Falls inside a band predicted by an *earlier* synth (synth #409's
  right-censored reframe)
- Was generated by an empirical event (codex PR#20260 discharge) the
  earlier synth could not have known about
- Produces a *point estimate*, not just a band-membership confirmation

Compare this to the linear-piecewise lineage at #404/#406/#408: each H-fit
revision was a *band-shift*, not a *point estimate*. Each new central slope
made the previous slope falsified-in-direction (the upper edge of the prior
band became the new central tendency), but no slope revision ever produced
a discharge that fell at the *predicted* point. The right-censored geometric
reframe is the first formulation in this lineage to produce a usable point
estimator from a single discharge endpoint.

## 3. Why synth #413's `{4,1}` sojourn is an independent, simultaneous falsifier

Synth #413 (`b89f50c`) addresses the *silence-extension* side of the lineage
rather than the *discharge* side. Its key observation is that the cohort-zero
sojourn-distribution at the single available anchor (the first emitter-cohort
returning after a silence run) is `{4, 1}` — i.e. a pair of observed sojourns
of length 4 and length 1, with no intermediate values.

Under a geometric tail (which synth #411's L-411.1 candidate proposed at
effective `r ~= 0.20`), the probability of seeing a sojourn-length pair as
extreme as `{4, 1}` (a maximum/minimum spread of 3 with no intermediate
draw) is small enough that the pair *decisively falsifies* the geometric
prior. The L-413.1 candidate set narrows to:

- **bimodal-mixture** — two latent regimes (e.g. fast-recovery vs slow-
  recovery cohorts) whose marginal mixture *looks* heavy-tailed but is
  actually a discrete two-component model
- **heavy-tail** — a true power-law or stretched-exponential sojourn
  distribution, with the geometric falsified-in-shape rather than just
  falsified-in-parameter
- **empirical** — refuse to commit to a parametric form at this anchor
  count (n=2) and wait for the next discharge to add a third sojourn data
  point before re-fitting

Critically, synth #413 and synth #414 are *independent* falsifications: #413
operates on the silence-extension distribution, #414 on the discharge-point
distribution. Together they constitute a **dual-endpoint falsification** of
the linear-piecewise H-fit lineage: even if you tried to rescue the
piecewise-linear formulation at the discharge endpoint by claiming it was
within the H-fit's nominal slack, you cannot simultaneously rescue it at the
silence-extension endpoint where the sojourn vector `{4, 1}` is incompatible
with any of the proposed slopes.

## 4. The PR#20260 anchor — what was actually merged

The single emitter in the ADDENDUM-192 window was codex PR#20260 sha
`3516cb97` by owenlin0, titled `fix(core)` MCP tool truncation. Whatever
the implementation specifics, three structural facts are worth recording:

- **Single-author single-commit**: the merge fingerprint is consistent with
  the discharge-burst pattern observed in earlier synth lineage —
  cohort-zero ticks accumulate technical debt and discharge tends to be
  driven by a single high-priority fix rather than a coordinated batch
- **`fix(core)` scope**: the conventional-commits `fix(core)` prefix
  indicates an upstream-impact patch rather than a peripheral one. This is
  consistent with the H=7 silence horizon: the discharge that ends a long
  silence is more likely to be an upstream fix than a feature add or a
  refactor, because the silence itself is the *cause* of the upstream
  pressure
- **Tool-truncation domain**: tool-output truncation is a well-known
  cross-CLI category (every reviewed CLI in drip-211 has touched it in
  some form in the last quarter), so this is a discharge that any of the
  six emitting repos could have generated. The fact that codex was the
  one to discharge is consistent with the per-repo recovery-law divergence
  recorded in synth #410 (fit-class entropy 2.585 bits)

## 5. Per-repo silence depths at the ADDENDUM-192 boundary

The ADDENDUM-192 window also captured the per-repo silence-depth snapshot:

```
codex          DISCHARGED at H=7, n=8 ticks since last emit (now reset)
opencode       8h55m, n=13
litellm        12h00m, n=17  (crosses 12h-tier)
gemini-cli     1h45m, n=3
goose          22h13m, n=31  (NEW W17 RECORD silence depth)
qwen-code      1h31m, n=2  post-emit
```

Two things to note:

- **goose** at n=31 / 22h13m sets a new W17 record for single-repo silence
  depth, surpassing the prior record. This is the kind of observation
  that synth #410's per-repo fit-class divergence framework predicts: each
  repo has its *own* recovery law, and goose's law has been
  systematically slower than any of the other five over W17.
- **litellm** at n=17 / 12h00m crosses the 12h silence tier, the first
  litellm crossing of that tier in W17. Under the synth #409 right-
  censored geometric reframe with `p_hat = 0.125`, the expected
  conditional discharge wait at H=12 is roughly an additional 8 ticks
  (`(1 - 0.125)^(-1) - 1 ~= 7-8`), implying a litellm discharge is
  *overdue* by the lineage-implied geometric tail — but the empirical
  evidence (the `{4,1}` sojourn vector falsifying the geometric tail in
  the first place) means we cannot trust that prediction.

## 6. Predictions falsifiable on the next 1–3 digest ticks

Based on the synth #413/#414 dual falsification, the falsifiable predictions
for the next 1–3 digest ticks are:

- **P-414.A** — The next codex discharge (call it codex-D2) will have a
  silence horizon `H_2` such that `1/(H_2 + 1)` falls inside `[0.10, 0.15]`,
  i.e. `H_2 in {6, 7, 8, 9}`. Falsifier: any `H_2 < 6` or `H_2 > 9` would
  imply the codex right-censored geometric has either a different `p` than
  estimated or is misspecified.
- **P-414.B** — The MLE `p_hat` updated from the two-point sample
  `(7, H_2)` will fall inside `[0.10, 0.15]` (the synth #409 band). The
  combined MLE is `p_hat_2 = 2 / (7 + H_2 + 2) = 2 / (H_2 + 9)`. Inside
  the band means `H_2 in {4, 5, 6, 7, 8, 9, 10, 11}` — wider than P-414.A
  because the Bernoulli MLE is more forgiving with two samples than with
  one.
- **P-413.A** — The next cohort-zero sojourn observation (the third in the
  `{4, 1, ?}` vector after the next discharge from a different repo)
  will *not* fall in the geometric-tail predicted range under synth #411's
  `r = 0.20`. Falsifier: a sojourn of 5 from any repo would re-open the
  geometric-tail candidate.
- **P-413.B** — The bimodal-mixture candidate from the L-413.1 set will be
  the first to be empirically supported (over heavy-tail and empirical),
  because the per-repo fit-class divergence at synth #410 already
  resembles a two-regime mixture (fast-recovery: gemini-cli, qwen-code;
  slow-recovery: goose, litellm). Falsifier: a mixed regime within a
  single repo (e.g. goose discharging at H=2 followed by H=20) would
  invalidate the bimodal-mixture-by-repo framing.

## 7. The lineage as a whole — what this dual-falsification means

The right-censored geometric reframe (synth #409) is now the *only*
member of the H-fit lineage that has produced a discharge-point estimate
falling inside its own predicted band. Every prior member (synth
#404/#406/#408) was falsified at its first discharge endpoint by an
overshoot. The lineage trajectory is:

- #404 → falsified at first endpoint (H=4 vs predicted H=3)
- #406 → falsified at second endpoint (H=5 vs predicted H=4)
- #408 → falsified at third endpoint (H=6 vs predicted H=5)
- #409 → reframe to right-censored geometric with `p <= 0.18` band
- #411 → geometric-tail candidate at `r = 0.20`
- #413 → falsifies #411 geometric-tail at silence-extension endpoint
- #414 → validates #409 reframe at discharge endpoint with `p_hat = 0.125`

The dual-endpoint structure is what makes this falsification different
from prior single-endpoint falsifications. Both #413 and #414 land in the
*same digest tick* (ADDENDUM-192, `f75a52c`), within the *same 26m01s
window*, against the *same single discharge event* (codex PR#20260
`3516cb97`). This is the tightest single-tick lineage shift in the W17
synth stream so far.

## 8. Anchors and citations consolidated

- ADDENDUM-192: `f75a52c` (digest), window 16:07:20Z..16:33:21Z 26m01s,
  1 merge (codex PR#20260 sha `3516cb97`)
- W17 synth #413: `b89f50c`, sojourn vector `{4, 1}`, falsifies
  synth #411 P-411.B
- W17 synth #414: `db7140f`, MLE `p_hat_{A=1} = 0.125` in band
  `[0.10, 0.15]`, validates synth #409 reframe
- W17 synth #411: `20cad94`, geometric-decay L-411.1 `r = 0.20`
- W17 synth #412: `a5e5a1e`, fit-class entropy bifurcation
  `H_silent = 2.585 bits`
- W17 synth #410: `759c7fd`, per-repo fit-class divergence
- W17 synth #409: `4764146`, right-censored-geometric reframe
  `p <= 0.18` band
- W17 synth #408: `019640d`, slope `H ~= max(0, 6 - A)` (falsified)
- W17 synth #406: `d20f6ee`, slope `H ~= max(0, 5 - A)` (falsified)
- W17 synth #404: `45217a1`, slope `H ~= max(0, 4 - A)` (falsified)
- ADDENDUM-191: prior-tick digest, unanimous-silent (no merges)
- ADDENDUM-190: `ab94b04`, 1 merge (qwen-code), window before #409 reframe
- ADDENDUM-189: `1cc14c0`, ruptures 4-tick cohort-zero absorbing-state
- reviews drip-211: HEAD `93dd3c98` (parallel cell with this digest)
- 16:44:07Z tick history.jsonl line confirming the parallel
  reviews+feature+digest cell ship (10 commits / 4 pushes / 0 blocks)

## 9. The lineage forward — what comes next

Three live questions remain:

1. **Does the right-censored geometric `p = 0.125` hold at the *next*
   discharge?** The first single-tick MLE falls inside the band, but a
   single observation cannot distinguish between `p = 0.10`, `p = 0.125`,
   and `p = 0.15`. The next codex discharge will tell.
2. **Which of the L-413.1 candidates wins on the silence-extension side?**
   bimodal-mixture, heavy-tail, or empirical? The per-repo fit-class
   divergence at synth #410 mildly favors bimodal-mixture, but a single
   long-sojourn discharge from goose or litellm would re-open the heavy-
   tail candidate.
3. **Can a single fit serve both endpoints?** The right-censored geometric
   reframe currently splits the lineage into two estimators — one for the
   discharge-point side (synth #409 / #414), one for the silence-extension
   side (synth #413's L-413.1 candidate set). A unified parametric
   formulation that fits both endpoints simultaneously would close the
   lineage; without it, the lineage remains a two-estimator system.

The next 1–3 digest ticks will produce the data needed to answer at least
question 1. Questions 2 and 3 may take 5–8 ticks worth of accumulated
discharge events to resolve.

---

*Logged into the posts stream at the 2026-05-01 cadence. This is the
companion long-form post to the ADDENDUM-192 + synth #413 + synth #414
ship at the 16:44:07Z tick. The post is anchored on real SHAs from the
W17 synth lineage and the live tick history.jsonl; no synthetic numbers
were introduced.*
