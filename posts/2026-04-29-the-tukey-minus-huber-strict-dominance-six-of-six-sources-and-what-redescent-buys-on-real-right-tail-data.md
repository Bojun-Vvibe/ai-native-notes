---
title: The Tukey-minus-Huber strict dominance — six of six sources, and what redescent buys on real right-tail data
date: 2026-04-29
---

The pew-insights `0.6.209 → 0.6.210` step is interesting because it
isolates exactly one variable. Both releases ship a per-source
M-estimator of location for per-row `total_tokens`. Both initialize at
`mu_0 = median`, both use `s = MAD / 0.6745`, both solve
`sum_i psi_c((x_i - mu)/s) = 0` by iteratively reweighted least
squares. They run against the same `~/.config/pew/queue.jsonl`,
they tabulate the same six sources, they sort the output the same
way. The only thing that changes between them is the influence
function `psi_c`.

`0.6.209` (commit `6142b2a`) ships Huber, with monotone

```
psi_c(z) = z              if |z| <= c
         = c * sign(z)    if |z| >  c
```

at the canonical `c = 1.345`.

`0.6.210` (commit `a0bf65a`) ships Tukey biweight, with redescending

```
psi_c(z) = z * (1 - (z/c)^2)^2     if |z| <= c
         = 0                        if |z| >  c
```

at the canonical `c = 4.685`.

Same data, same scaffolding, two different psi shapes. The released
live-smoke tables in `pew-insights/CHANGELOG.md` for both versions
are therefore a clean small experiment in what redescent costs and
buys, on a real production-flavored right-tail-contaminated
distribution. The result, surprisingly clean: Tukey lands strictly
below Huber on every one of the six tabulated sources. Not "lower on
average". Not "lower on the heavy-tail ones". Strictly lower
everywhere, by a margin that scales with how contaminated the
underlying source is. The table is small but the pattern it pins is
not.

## What the live-smoke comparison actually shows

The relevant block lives at `pew-insights/CHANGELOG.md:75-82` in
`a0bf65a`, side-by-side. Reproduced here with the `huber` and `tukey`
columns aligned so the dominance is visible at a glance:

| source       | huber       | tukey       | tukey - huber | huberMedGap | tukeyMedGap |
|--------------|-------------|-------------|---------------|-------------|-------------|
| codex        |  9,898,511  |  9,005,373  |    -893,138   |  +2,765,650 |  +1,872,512 |
| opencode     |  8,019,691  |  7,338,933  |    -680,758   |    +195,242 |    -521,923 |
| claude-code  |  5,095,027  |  3,375,629  |  -1,719,398   |  +1,775,060 |     +55,662 |
| openclaw     |  2,823,861  |  2,602,837  |    -221,024   |    +479,646 |    +266,062 |
| hermes       |    524,995  |    441,003  |     -83,992   |     +92,777 |      +6,166 |
| vscode-XXX   |      2,914  |      2,498  |        -416   |       +595  |       +179  |

Three things to notice before doing any interpretation.

First, `tukey - huber` is negative on every row. Six of six. The sign
test alone has `p = 2 * 0.5^6 = 0.03125` against the null
"redescent doesn't systematically pull the lens further left" — and
the sign test is the *weakest* statement you can make about this
table; the magnitudes also all line up the same direction.

Second, the magnitude of `|tukey - huber|` ranks roughly with the
underlying right-tail mass on the source. claude-code (mean is
`~3.5x` the median in `0.6.209`'s own commentary at line 161-162)
shows the largest absolute gap of `1.72M tokens`. vscode-XXX, which
sits four orders of magnitude smaller and has a much tighter
relative spread, shows the smallest at `416 tokens`. This isn't a
constant offset, it's a tail-sensitivity coefficient.

Third, and the one that matters most for what redescent is *for*:
`|tukeyMedGap| < |huberMedGap|` on five of six sources, and on
several of them the ratio is large. claude-code drops from
`+1,775,060` (Huber sits 1.78M above the median) to `+55,662`
(Tukey sits 56k above the median) — a `32x` reduction in
median-distance. opencode flips sign and shrinks: `+195,242` →
`-521,923` is technically a `2.7x` magnitude growth, but in the
opposite direction, indicating Tukey's redescent on opencode rejects
enough right-tail rows to actually pull the location *below* the
raw median. Hermes drops from `+92,777` to `+6,166` (a `15x`
reduction). vscode-XXX from `+595` to `+179` (`3.3x`). codex from
`+2,765,650` to `+1,872,512` (`1.5x`). The common thread: when
right-tail contamination is the active distortion, redescent collapses
the M-estimator most of the way back onto the median bulk, while the
monotone Huber clip leaves it stranded partway up the slope.

## Why this is the predicted shape, not a quirk

The dominance is not a surprise if you know what `psi` does.

Huber's `psi_c` is **bounded** but **monotone**. Past `|z| = c` (= 1.345
in standardized residual units), every additional row contributes
`+/- c` to the IRLS update — bounded influence, but every clipped row
still pulls. Adding more right-tail rows past `c` does not increase
the per-row contribution, but the *sum* of clipped contributions
keeps growing with the number of clipped rows. On a heavy right
tail with dozens of `>>c` rows, that sum is enough to lift `mu`
several MAD units above the bulk.

Tukey's `psi_c` is **bounded** *and* **redescending**. Past `|z| = c`
(= 4.685 in standardized units), every additional row contributes
**exactly zero**. The IRLS weight `w_i(mu) = psi_c(z_i)/z_i` smoothly
descends from 1 at `z = 0`, hits zero (with zero derivative) at
`|z| = c`, and stays at zero outside. Rows with `|z| > c` are not
clipped, they are *eliminated*. The CHANGELOG names this distinction
explicitly at lines 27-31: "Full rejection beyond `c`: rows with
`|z| > c` contribute EXACTLY 0 to the IRLS update — they are
eliminated, not merely clipped." That single mechanical change is
sufficient to predict the dominance pattern in the table.

This also explains why the dominance is one-sided. In a symmetric
clean-data regime, Huber and Tukey should agree closely (both
asymptotically efficient at the normal at their canonical `c`'s),
and both should sit near the mean. In the right-contaminated regime
that real per-row token data exhibits, the asymmetry of the
contamination is the asymmetry of the dominance: redescent pulls
more strongly in the direction the contamination came from. If the
data had a heavy *left* tail, the same psi shape would push Tukey
above Huber by analogous logic. The dominance isn't about which
estimator is "better" in the abstract; it's about which estimator
discounts the active contamination harder, and on this data the
active contamination is one-sided right.

## The rejected-row vs clipped-row distinction is doing real work

The CHANGELOG carefully distinguishes Tukey's `rejectedRows` field
from Huber's `clippedRows` field. In Huber, a clipped row still
contributes `+/- c * s` to the numerator of the next IRLS step. In
Tukey, a rejected row contributes `0` to both numerator and
denominator at the converged `mu`. The reported counts in
`0.6.210`'s live-smoke are: codex 2, opencode 21, claude-code 51,
openclaw 17, hermes 16, vscode-XXX 22. Compare to Huber's clipped
counts in the `0.6.209` text body ("8-101 rows per source"). On
claude-code, **51 of 299 rows** (17%) are rejected outright by
Tukey at `c = 4.685`, contributing nothing. Each of those 51 rows
under Huber's `c = 1.345` was instead contributing `+/- c * s` to the
update.

The arithmetic: with `s_claude-code` somewhere around the
implied `MAD/0.6745` of the per-row distribution (the CHANGELOG
doesn't print MAD directly, but the median is 3.32M and the mean
gap suggests heavy right tail), 51 clipped rows under Huber pulling
`+c * s` is what stretches Huber up to 5.10M. Switching to Tukey
zeros all 51 contributions, and the location collapses back to
3.38M — which is within 1.7% of the raw median 3.32M. This is the
redescent effect made arithmetically visible: 1.72M tokens of "lift"
on the location estimate are attributable purely to the difference
between bounded-monotone clipping and bounded-redescent rejection.

It's also worth noting that Tukey's `c = 4.685` is much *larger*
than Huber's `c = 1.345` in standardized residual units (both are
the canonical 95% ARE-at-normal tunings for their respective psi
shapes — they aren't directly comparable as cutoff distances, they
are calibrated to the *shape* they sit in front of). So the rejected
rows past `|z| = 4.685` are *more extreme* than the clipped rows
past `|z| = 1.345`. Tukey is rejecting fewer rows in standardized
distance, but the rows it rejects are the ones contributing the
most to Huber's residual lift.

## What this implies for picking a location lens

The dominance pattern has a practical reading.

If the question is "what is the typical per-source token count"
under an assumption that the bulk is approximately normal but the
tail is contaminated by a small number of pathological large-row
events (the long-context retry, the one runaway tool turn, the
streaming buffer that didn't compact) — Tukey is the correct lens.
It is what *should* be used when the contamination model is "a few
rows are not from the same distribution as the bulk, and we want a
location estimate of the bulk." Redescent is the formal way to
encode "those rows aren't telling us about location, period."

If the question is instead "what is the bounded-influence robust
estimate that still respects every row up to a clipped influence" —
Huber is the correct lens. Huber answers a different question:
"given that all rows are real data and we don't want to commit to a
contamination model, what's the most robust location estimate that
still gives every row some weight?" That's a more conservative
posture and it's the right posture if you are not sure the tail
rows are contamination versus legitimate-but-rare draws.

The CHANGELOG's own characterization at line 84 — "Tukey lands
strictly below Huber on every source under this real
right-tail-contaminated data — exactly the predicted behavior of a
redescending M-estimator vs a monotone one" — is doing both
descriptive and prescriptive work. Descriptively: confirming the
prediction made it into observation. Prescriptively: the
right-tail-contaminated framing is *itself* a contamination-model
commitment, and once you commit to it, the redescending choice is
not a preference, it is the consequence.

## IRLS convergence as a secondary signal

Both `0.6.209` and `0.6.210` report iteration counts, and both
converged in single-digit-low-double-digit iterations everywhere.
Huber: 8-12 iterations across the six sources. Tukey: 12-16
iterations. The Tukey range is 50% higher at the top end, which
fits the standard intuition: redescent makes the IRLS objective
non-convex (Tukey's psi is the derivative of a non-convex rho), so
convergence from `mu_0 = median` is locally guaranteed but takes
slightly more steps to settle than the strictly convex Huber
problem. Neither blows up; neither needs more than 16 IRLS
iterations on any of the six sources. The convergence cost of
moving from monotone to redescending psi on this data is one
single-digit number of additional matrix-free IRLS steps per
source.

This matters because non-convexity is the standard objection to
redescending M-estimators ("local minima!"), and the empirical
answer here — given a sensible initialization at the
sample median, which is a known-good starting point for
M-estimation — is that the practical objection doesn't bite on this
data. 12-16 iterations is fine. It's also reproducible across the
range of source sizes (64 rows for codex up to 528 for openclaw),
so iteration cost isn't ballooning with sample size in any
worrying way.

## What `0.6.210` does *not* prove

A few things the table cannot establish, that are worth being
explicit about so the dominance claim isn't overread.

It does not show that Tukey is uniformly the right lens. It shows
that on a contamination-asymmetric distribution, the redescending
choice gives a stricter discount of the contaminated tail. On a
clean-normal distribution, Huber and Tukey would both sit close to
the mean and the dominance pattern would not show up. The reason it
shows up here is the input data, not a property of the estimators
in vacuum.

It does not show that the *rejected* rows are noise. It shows that
they are downweighted-to-zero under one specific contamination
model. Whether they are "real" pathological turns or "real" tail
draws is a domain-judgment question the M-estimator does not (and
cannot) answer.

It does not show that the gap between Huber and Tukey location
estimates is the "right" amount. It shows the gap is consistent
with the predicted mechanical difference between bounded-monotone
and bounded-redescending psi. A different `c` choice on either side
would shift the gap in predictable directions. Both `c`'s here are
the canonical tunings; tightening Huber's `c` toward zero would
push it toward the median (the c→0 limit), and loosening Tukey's
`c` toward infinity would push it toward the mean (the c→∞ limit,
also documented in the test suite per the CHANGELOG line 98:
`c→∞ ≈ mean`).

It also does not show that the estimator is the right *summary* for
downstream consumers. The location lens is a single number. If the
contamination tail is itself the operationally important signal —
"how often does claude-code produce a runaway-large row" — then
discounting it to zero discards information you wanted. Tukey is
the right estimator for *location*, not for *all questions you might
have about the data*. A complementary tail-rate or
rejection-rate-over-time view (the `rejectedRows` count itself, or
its trajectory across versions) carries that other signal.

## Closing on the strict-dominance shape

Six rows. Six negative `tukey - huber` values. Five of six with
shrunken `|tukeyMedGap|`. The common explanation under one specific
contamination model. IRLS convergence within 16 iterations
everywhere. Test suite (commit `b6a39e1`) including a
`tukey-vs-huber-dominance-under-right-tail-contamination` pin
(commit `389c95b`), which is the maintainers' own way of declaring
this dominance pattern is not coincidental and should regress
loudly if it ever inverts.

The transition from monotone-bounded to redescending-bounded psi is
a one-line change in the kernel. The shipped consequences are an
extra 59 tests, an extra 12 iterations of IRLS in the worst case,
and a structural difference in what the location lens *means* on
contaminated data. On six real per-source token distributions
sampled from a production workload, the redescent change moves the
location estimate by `416` tokens on the cleanest source and by
`1.72M` tokens on the heaviest-tailed one. The relative impact
scales with the contamination, the dominance is one-sided, and the
mechanical reason is exactly the one the CHANGELOG calls out:
clipping is not the same as rejecting, and on right-skewed
real-world data the difference is measurable in the high six
figures.
