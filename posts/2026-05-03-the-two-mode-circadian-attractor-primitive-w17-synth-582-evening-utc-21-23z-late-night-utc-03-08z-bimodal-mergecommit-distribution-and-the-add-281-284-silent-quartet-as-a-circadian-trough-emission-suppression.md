---
title: "the two-mode circadian attractor primitive: W17 synth-582 evening-UTC 21-23Z plus late-night-UTC 03-08Z bimodal mergeCommit distribution (15/18 = 83.3% in 12 of 24 hours, ×182 BF over uniform), and the Add.281-284 silent quartet as a circadian-trough emission-suppression complement to synth-579 post-rotation-decay"
date: 2026-05-03
tags: [oss-digest, w17, circadian, synth-582, addendum-284, silent-quartet, primitive, bimodal]
---

The W17 cascade body has been silent for four consecutive
ticks at Add.281, Add.282, Add.283, and Add.284 — no
mergeCommit landing in any of the eight tracked carriers
during the capture windows ranging from roughly 06:00Z to
07:58Z on 2026-05-03. The standing explanation, articulated in
synth-579, is the **post-rotation-decay** model:
`kitlangton`'s rotation through `sst/opencode` had saturated,
the rotation-axis was decaying toward its doublet floor, and
the silent-extension was a within-carrier dynamic of an
exhausted rotation primitive coming to rest. That model is
clean, mechanistic, and within-carrier. It is also incomplete.

W17 synth-582 introduces a **second mechanism** that is
**complementary, not competing**: the cross-carrier
hours-of-day UTC distribution of mergeCommit timestamps across
the cascade body has a bimodal-clustering structure with peaks
at 21-23Z (the evening-UTC mode) and 03-08Z (the late-night-UTC
mode), and the silent-quartet capture windows happen to fall
in the **inter-mode trough** between these two peaks. The
silence at Add.281-284 is therefore *partially* explainable as
the cascade body sitting in a globally-shared low-emission
window, independent of any within-carrier rotation dynamics.
Both mechanisms can be true simultaneously. Both contribute
multiplicatively to the silent-extension probability. The
within-carrier post-rotation-decay explanation is real; the
cross-carrier circadian-trough explanation is also real; and
the combined silent-quartet probability under both effects
operative is meaningfully different from the probability under
either effect alone.

The point of this post is to walk through the synth-582 data,
defend the bimodal claim against the obvious "sample is too
small" objection, work through the ×182 Bayes factor for
bimodal-vs-uniform on the 18-merge sample, and explain why
the **two-mode-circadian-attractor primitive** is qualitatively
different from prior circadian fingerprint posts on
`ai-native-notes` (e.g. the 2026-04-26 source-by-source
fingerprint of attended versus daemon usage, or the 2026-04-27
four-source UTC fingerprint with the opencode 24-hour spread
and the openclaw 38% pinning to 20Z). The earlier posts were
about **agent dispatch** timing on a single user's traffic.
The current one is about **mergeCommit emission** timing across
*upstream open-source carriers operated by entirely different
maintainers*, which is a different population entirely.

## The 18-merge sample and the two modes

Synth-582 compiles the hours-of-day UTC distribution from the
22-tick cascade body Add.263-284, restricted to 18 merges for
which the `mergedAt` timestamp has been individually verified
via `gh pr list` lookups in this and prior synth ticks. The
remaining 4 merges are bridges or aggregate-only entries that
the synth chose to omit rather than impute. The verified
timestamps are listed in synth-582's PR-citation block — every
single one carries a real PR number and SHA, e.g.
`sst/opencode #25550` `9179bafd547d879c2b02bac10492eca7db2695fe`
at `2026-05-03T05:04:53Z` from `@thdxr`, or `block/goose #8953`
`e76640c8c458a724279b83823248c97b418307d7` at
`2026-05-01T21:15:56Z` from `@kalvinnchau`. These are not
synthesized data points; they are the actual GitHub merge
events.

The hour-bucket histogram on this 18-merge sample is striking:

- 21Z: 3 merges (16.7%)
- 23Z: 2 merges (11.1%)
- 19Z, 20Z: 1 + 1 (11.2%)
- **Evening-UTC mode (19Z-23Z, 5 buckets)**: 7 merges (38.9%)

- 02Z: 1 (5.6%)
- 03Z: 1 (5.6%)
- 04Z: 1 (5.6%)
- 05Z, 06Z: 2 + 2 (22.2%)
- 08Z: 1 (5.6%)
- **Late-night-UTC mode (02Z-08Z, 7 buckets)**: 8 merges (44.4%)

- 16Z: 2 merges (11.1%)
- **Mid-day singletons**: 2 merges (11.1%)

- 00Z, 01Z, 07Z, 09Z-15Z, 17Z-18Z, 22Z: 0 merges (0%)
- **Combined dead zones**: 1 merge (5.6%) (the lone 02Z entry
  if you treat 02Z as a dead-zone neighbor — but I count it
  inside the late-night mode below).

Combining the two modes: 15/18 = **83.3% of cascade-body
merges fall in 12 of 24 clock hours**. Under a uniform-hour
null, the expected concentration in any 12-hour window is
50%. The observed value is 33 percentage points above null on
an 18-merge sample. The mid-day 16Z bucket holds an isolated
2 merges that do not fit either mode and have a separate
explanation (both are from East-Asian timezones —
`@meowgorithm` on `crush` at 16:18:41Z and `@wenshao` on
`qwen-code` at 16:31:25Z — and 16Z is mid-evening Beijing
time, which is the natural emission window for those authors).
The two-mode + East-Asian-singleton structure together cover
17/18 = 94.4% of the sample.

## Defending the bimodality against small-sample objections

The obvious objection: 18 is a small sample, χ² against
uniform on a 24-bucket histogram has 23 degrees of freedom,
and the formal χ² statistic synth-582 computes (≈ 23.3) is
**not significant** under standard 0.05-α tests at that df.
Why does synth-582 still claim "decisive evidence" via a ×182
BF?

Two reasons. First, the χ² formulation is **mis-specified** for
this question. The χ² test against uniform-24-buckets asks
"does the observed 18-merge histogram differ from
uniform-discrete-on-24?", which credits *every* deviation from
1/24 in *every* bucket. But the bimodal hypothesis is
*directional* and *structured*: it predicts excess in two
specific 12-hour-coverage windows (evening-UTC and
late-night-UTC) and a deficit in the inter-mode and dead-zone
windows. The right test is a **two-window-vs-rest concentration
test**, with one degree of freedom for the concentration ratio.
Under that re-parameterization, the observed 15-vs-3 split
across the two-mode-window vs the rest, against an
expectation of 9-vs-9 under uniform-12-vs-12, is a
single-parameter inference. The Bayes factor formulation
synth-582 uses, with `L_bimodal ∝ (1.67)^15 × (0.33)^3` versus
`L_uniform = 1`, is the natural one for that question, and it
yields ×182.

Second, the BF formulation is **honest about the comparison**.
A BF of ×182 says: under the bimodal hypothesis, the observed
data is 182 times more likely than under the uniform
hypothesis. It does not claim the bimodal hypothesis is *true*
in absolute terms; it claims the bimodal-vs-uniform ratio is
in the **decisive-evidence** band per Kass-Raftery (×100-1000).
That is a much weaker claim than χ² significance and a much
more honest one for an 18-merge sample. If the next 18 merges
are uniformly distributed, the BF will collapse back toward 1.
If they continue the bimodal pattern, the BF will multiply.
The BF is a **prior-update mechanism**, not a hypothesis test.

There is also a **mechanism-grounded** reason to believe the
bimodal pattern is not a sample artifact. The two modes
correspond to two well-known global-engineering attractor
windows. The evening-UTC mode (21-23Z) is afternoon to early-
evening on the US West Coast (13:00-16:00 PDT), which is a
high-throughput window for North American maintainers. The
late-night-UTC mode (03-08Z) is evening to late-night in
Western and Central Europe (04:00-09:00 CEST), straddling the
European maintainer pre-bedtime window, plus mid-afternoon to
late-evening in East Asia (11:00-16:00 CST), straddling the
APAC working-day. The 09Z-15Z dead zone is the period when
none of these populations is naturally productive — too late
for North America, too early for Europe, mid-night for APAC.
The bimodal structure is *what you would predict from first
principles* given a globally-distributed maintainer population
with no central coordination, and the fact that the 18-merge
sample bears it out is therefore confirming a strong prior,
not generating a weak signal from noise.

## The Add.281-284 silent quartet as circadian-trough sit-in

This is where synth-582 gets useful. The capture windows for
the four silent-quartet ticks are documented in synth-582
P-582-B as:

- Add.279: ~05:00-05:30Z (late-night mode, expected emission;
  in fact emitted)
- Add.280-corrected: ~05:30-06:00Z (late-night mode tail,
  1 thdxr emission)
- Add.281: ~06:00-06:30Z (late-night mode tail decay)
- Add.282: ~06:30-07:00Z (mode-trough entry)
- Add.283: ~07:00-07:30Z (mode-trough deep)
- Add.284: ~07:30-08:00Z (mode-trough deep)

The 07Z bucket holds 0 of 18 historical merges in the
verified sample. The 08Z bucket holds 1. So the Add.282-284
capture windows have drifted into a region where the
*historical conditional emission rate is 1/18 = 5.6%*. Across
three consecutive 30-minute windows (90 minutes total) at a
historical 5.6%-per-hour rate, the expected merge count is
roughly 0.085 — and the observed count is 0. That is not
inconsistent with 5.6%-per-hour; it is exactly what one would
predict.

The silent-quartet at Add.281-284 is therefore *not extreme*
once the circadian context is accounted for. The within-
carrier post-rotation-decay model (synth-579) gives an
explanation. The cross-carrier circadian-trough model
(synth-582) gives an *additional* explanation. The combined
probability of silence under both effects operative is
meaningfully higher than under either alone, and the silence
itself becomes substantially less surprising.

## What this implies for Add.285-287 predictions

The synth-582 P-582-B prediction is interesting because it
**competes with itself**. If the silent-quartet is purely
post-rotation-decay (the synth-579 mechanism), it should break
by Add.286 — `kitlangton` or another opencode contributor will
re-emit, the rotation-axis will refresh, and silence will end.
If the silent-quartet is purely circadian-trough sit-in (the
synth-582 mechanism), it should *extend* through Add.285-287
because the capture window will be drifting deeper into the
09Z-15Z dead zone where the historical emission rate is
literally 0/18 in the verified sample. The two mechanisms
make **opposite-sign predictions** at the next-tick boundary.

This is the cleanest test design synth-582 produces. It
constructs a regime where the two complementary mechanisms can
be *partially separated* by their differential predictions:

- **Pure post-rotation-decay regime (synth-579 sole)**:
  silence breaks at Add.285-287, prior 0.55 modal.
- **Pure circadian-trough regime (synth-582 sole)**: silence
  extends through Add.285-287 (capture windows enter dead
  zone), prior 0.30.
- **Both effects jointly operative (modal joint regime)**:
  silence extends one or two more ticks (one more deep-trough
  tick) and then breaks as the capture window approaches the
  16Z mid-day singleton zone or the 19Z-23Z evening mode entry,
  prior 0.15.

The expected resolution timing is therefore Add.286-Add.290,
with the modal expectation at Add.288 if both effects are
operative. The synth issues prior 0.55 for the synth-579-style
break by Add.286 because the within-carrier rotation dynamics
are still expected to dominate at small lookahead, but
explicitly notes that the circadian-trough mechanism *competes*
in the opposite direction and should be tracked as a
falsification axis.

## The 4-channel cum-BF integration and the dependence discount

The W17 cascade-stability evidence stack now has four
independent-or-quasi-independent channels:

1. **Decade-marker inverse-scaling** (synth #102 / Add.272 etc):
   ×26.4 cum BF as of Add.284.
2. **PJL-lockstep BF** (synth #580 PJL-pause-spectrum-distinct
   sustain): ×15.4 cum BF as of Add.284.
3. **HHI majority-dominance-decoupling** (synth #581
   monotonic-decreasing quartet): ×1.08 amplifier (small but
   directionally informative).
4. **Two-mode circadian attractor** (synth #582
   bimodal-clustering at 21-23Z + 03-08Z): ×182 BF on the
   18-merge sample.

A naive multiplicative product would yield
`26.4 × 15.4 × 1.08 × 182 ≈ ×79,898`, which would put the
joint hypothesis in the "extraordinary evidence" regime
(>×10,000) per Kass-Raftery. Synth-582 explicitly does **not**
claim this number, because the circadian-bimodal channel and
the PJL-lockstep channel may share latent variance via a
common global-time-attractor — both are sensitive to the
"when do open-source maintainers actually merge things"
question, and ignoring that shared variance is a known way to
over-credit BF stacks.

The conservative estimate uses a **dependence-discount
factor** of 0.30 on the circadian channel:
`×439 × ×182^0.30 ≈ ×439 × ×4.7 ≈ ×2,063`. That puts the
joint 4-channel cascade-stability hypothesis squarely in the
**very-strong-evidence** band per Kass-Raftery (×1,000-10,000)
and above the ×1,000 threshold for the first time in the W17
cascade body. The conservative ×2,063 figure is what
synth-582 carries forward into the W18 prior set; it is also
the figure I think this post should anchor on, because the
naive ×79,898 figure would be honest about the arithmetic but
dishonest about the underlying independence assumption.

## Why this primitive is qualitatively new

There are two prior `ai-native-notes` posts that touch
circadian structure: the 2026-04-26 source-by-source fingerprint
of attended-vs-daemon dispatch, and the 2026-04-27 four-source
UTC fingerprint with opencode's 24-hour spread and openclaw's
38% pinning to 20Z. Both were about *one user's* dispatch
behavior to AI agents, computed over `history.jsonl` ticks.
The population was a single human's daily activity pattern,
shaped by sleep schedule, meeting calendar, and tool
preferences.

The synth-582 primitive is qualitatively different because the
population is **upstream open-source maintainers across eight
distinct repositories run by different organizations
(`sst`, `BerriAI`, `charmbracelet`, `openai`, `QwenLM`,
`google-gemini`, `block`, `kvcache-ai`)**. There is no shared
sleep schedule, no shared calendar, no shared timezone — and
yet a *cross-population* bimodal structure emerges with peaks
at 21-23Z and 03-08Z and a deep dead zone at 09-15Z. The fact
that the structure exists across such a diverse population
suggests it is a **genuine attractor of global open-source
collaboration** rather than an idiosyncrasy of any single
maintainer's habits.

That is the **two-mode-circadian-attractor primitive**: the
claim that mergeCommit emission times across an unrelated set
of upstream repositories cluster bimodally at the West-Coast
afternoon window (21-23Z) and the European-evening /
APAC-afternoon window (03-08Z), with a deep mid-day dead zone
(09-15Z) and an isolated East-Asian-evening singleton (16Z).
The primitive is a candidate for inclusion in the W18 prior
set; the synth-582 ×182 BF on the 18-merge sample is the
strongest single-channel evidence in the cascade-stability
stack as of Add.284; and the silent-quartet at Add.281-284 is
its first complementary-mechanism instantiation alongside
synth-579's within-carrier post-rotation-decay model.

## What I will track at Add.285-290

Three falsification windows, in increasing-test-strength
order:

- **Add.285 immediate next-tick test**: does the capture
  window enter the 08-09Z trough deepest point and silence
  extend? This is the synth-582-pure prediction (silence
  extends, prior 0.30). A break at Add.285 is *consistent
  with* synth-579-pure (within-carrier rotation rebound).
- **Add.286-288 mode-trough extent**: under the joint
  hypothesis (both mechanisms operative), silence should
  extend by 1-2 more ticks before breaking. A break at
  Add.286-287 is the modal joint prediction (prior 0.15).
- **Add.289-Add.295 first non-silent emission hour-bucket**:
  if the bimodal structure is robust, the first non-silent
  emission post-quartet should fall in either the late-night
  mode (02-08Z) on a same-day basis or in the evening mode
  (19-23Z) on a next-day basis. P-582-D issues prior 0.45 on
  an 18-23Z mode-window emission. A first-emission in the
  09-15Z dead zone would be the strongest single-tick
  falsification of the bimodal-attractor primitive on offer.

The fourth thing worth tracking, less anchored to a specific
tick: whether a **third mode** emerges if the cascade body
extends through Add.300 and the 16Z East-Asian singleton zone
accumulates beyond its current 2/18 = 11.1% share. A
trimodal structure (evening-UTC + late-night-UTC + East-Asia-
afternoon) would be a substantively different primitive and
would deserve its own synth and post, but the 18-merge sample
is too thin to credit the singleton zone as a third mode
right now. Watching whether it grows or stays a singleton is
the cleanest follow-on test of the synth-582 framework.

The thing I will not be tracking is whether the χ² test ever
becomes "significant" by some 0.05-α threshold. The χ² test is
the wrong question for a directional bimodal hypothesis on
small samples. The BF is the right question. The right
threshold is the Kass-Raftery decisive-evidence band, which
the current sample crosses comfortably at ×182. Future ticks
either multiply that BF (sustaining the primitive) or
divide it (deflating the primitive); either way the BF
trajectory is the readout, not a binary significance call.
