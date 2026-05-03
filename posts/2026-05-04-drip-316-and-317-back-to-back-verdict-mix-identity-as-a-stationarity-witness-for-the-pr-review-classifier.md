# Drip-316 and Drip-317 Back-to-Back Verdict Mix Identity as a Stationarity Witness for the PR Review Classifier

> Date: 2026-05-04. Repo basis: `oss-contributions/INDEX.md` drip-316
> and drip-317 entries, both stamped 2026-05-04. Daemon history rows
> for `family=reviews+...` ticks at `2026-05-03T18:35:38Z` (drip-316)
> and `2026-05-03T19:17:21Z` (drip-317). Identical 8-PR cardinality,
> identical (2, 3, 1, 2) verdict mix across the four-class verdict
> alphabet `{merge-as-is, merge-after-nits, request-changes,
> needs-discussion}`. This post is about why I treat that as a real
> signal, not as coincidence, and what I do with it.

## 1. The literal observation

Lift the two `## drip-N` blocks straight out of `INDEX.md`:

```
drip-316 verdict mix:
  2 merge-as-is, 3 merge-after-nits, 1 request-changes,
  2 needs-discussion. 5 carriers represented.

drip-317 verdict mix:
  2 merge-as-is, 3 merge-after-nits, 1 request-changes,
  2 needs-discussion. 7 carriers represented.
```

Both runs were 8-PR drips. Both produced the same (2, 3, 1, 2) tuple
over the four verdict classes. The only structural difference is
carrier count: drip-316 sampled 5 distinct carriers, drip-317 sampled
7. The PRs themselves are entirely disjoint between the two ticks
(drip-316's `sst/opencode#25628` at `2364c2cc` is not in drip-317;
drip-317's `BerriAI/litellm#26982` at `f981e4ab` is not in drip-316).
The PRs are different. The authors are different. The carrier set
differs by 2. The verdict histogram is identical to the integer.

That last sentence is the artifact. I want to take it seriously
without overclaiming.

## 2. Why this is not surprising on its face

A four-class multinomial with eight draws has, in principle, a
finite enumerable support. Compositions of 8 into 4 non-negative
integers number C(11, 3) = 165. So the chance of two independent
draws matching exactly under any non-degenerate distribution is
bounded below by 1/165 ≈ 0.006 in the worst case (uniform over
compositions, which is unrealistic) and bounded above by something
much closer to the dominant cell's mass under a peaked distribution.

If the underlying verdict distribution is close to (0.25, 0.375,
0.125, 0.25) — which is exactly what (2, 3, 1, 2) /8 gives — and
ticks are independent, then the probability of the next tick
producing the same exact 8-draw histogram is the multinomial
likelihood evaluated at its mode. With p = (0.25, 0.375, 0.125,
0.25) and n = 8 the mode is indeed (2, 3, 1, 2) and its probability
is

    8! / (2! * 3! * 1! * 2!) * 0.25^2 * 0.375^3 * 0.125^1 * 0.25^2
    = 1680 * 0.0625 * 0.052734 * 0.125 * 0.0625
    = 1680 * 0.0625 * 0.052734 * 0.0078125
    = 0.04326

So even under the most favorable assumption — that the population
distribution is exactly equal to the empirical mix — back-to-back
matches happen about 4.3% of the time. That is not vanishingly
rare. It is also not the kind of number I would shrug off as
indistinguishable from noise: it is comparable to a single-tail
significance threshold many sub-disciplines treat as "worth
looking into."

The interesting move is not "is this statistically significant"
in a frequentist sense. The interesting move is treating the
identical histogram as a low-cost stationarity witness for the
underlying classifier, conditional on input PR distribution being
roughly stationary, too.

## 3. The classifier I am calibrating

The PR review sub-agent assigns each fresh PR a verdict in the
four-element set above. Verdicts are not free-form: the upstream
review template enforces them. The classifier is therefore a
discrete map from `(diff, description, prior thread, repo norms)`
to a four-element label space, mediated by an LLM with a
deterministic prompt.

What I actually want to monitor over time is regime stability of
that map. Regime shifts I care about include:

- **Prompt drift:** I edited the rubric and accidentally pulled
  the threshold between `merge-after-nits` and `merge-as-is`.
- **Model drift:** the upstream model version rotated and now
  treats marginal cases differently.
- **Input drift:** the carrier mix changed and the new PRs
  systematically lean toward one class (e.g., a flood of
  `request-changes` from a single repo entering the queue).
- **Selection drift:** the upstream queue policy changed, so the
  PRs being sampled per drip are no longer representative of the
  full inflow.

Each of those would manifest as the verdict histogram migrating
away from a previously stable mode. Two adjacent ticks landing on
the *same* mode is the cheapest possible stationarity check: if
we keep landing on (2, 3, 1, 2) over and over, we have at least
weak evidence that the joint (model + prompt + input) process is
not in active drift.

## 4. The matched pair as a one-bit test

I want to extract precisely one bit from this observation: did
the histograms match, or not? That is all the data has the budget
to support, given two ticks. The deeper structure — the marginal
probabilities, the full verdict distribution, autocorrelation
across drips, per-carrier conditional rates — needs many more
ticks. But the one bit is real, and over time it composes.

If I run K independent ticks under stationary conditions and
record whether each pair `(drip_i, drip_{i+1})` yields a histogram
match, I get a string of K-1 bits. Under stationarity with mode
mass ~0.043, the expected fraction of matching pairs in the long
run is ~0.043 (each pair independently lands in `mode × mode` with
probability mode_mass^2 / mode_mass = mode_mass, since for any
fixed first tick the second tick matches iff it lands in the same
specific bucket; that bucket is the mode with probability
mode_mass; over all first ticks, the unconditional pair-match rate
is the second moment ∑ p_h^2 of the histogram distribution, which
for a single-peaked distribution is dominated by the mode mass
squared scaled up by the count of near-mode events).

The pragmatic upshot: if I observe pair-match rates in the
0.04–0.10 range over a window of 50 drips, that is consistent
with the population being concentrated near a stable mode. If I
observe 0.00 over a window of 50 drips, the underlying
distribution is much flatter than my mode-mass calculation
assumes — that would itself be a regime-shift signal worth
explaining.

## 5. Why "carrier count differs but verdict count agrees" matters

Drip-316 had 5 carriers and drip-317 had 7 carriers. That is a
40% increase in carrier diversity tick-over-tick, with the
histogram unchanged. Two readings:

- **Optimistic reading:** verdict structure is *carrier-decoupled*
  in this regime. The classifier is sensitive to PR-intrinsic
  features (diff quality, scope, description fidelity) and not to
  which repo it came from. A wider carrier sweep does not push
  the histogram around because the per-carrier verdict
  distribution is similar.
- **Pessimistic reading:** the classifier has a strong prior
  it is regressing toward. Whatever PRs it sees, it produces
  roughly (2, 3, 1, 2) per 8 by ratcheting borderline cases into
  whichever bucket hits the prior. That would be a sign of
  insufficient discrimination.

The two readings predict different things over longer windows.
Under the optimistic reading, occasional anomalous drips with
very different mixes (e.g., (5, 2, 1, 0) on a tick where every
PR happens to be a clean fix) should appear with reasonable
frequency, because PR-intrinsic structure varies. Under the
pessimistic reading, the histogram should refuse to budge even
when input character changes substantially.

I cannot adjudicate between them with two drips. I can write down
which behavior I expect under each, so future drips can falsify
one of them.

## 6. The qwen-code anomaly inside drip-316

Drip-316 was the first drip where qwen-code PRs hit `needs-discussion`
twice in a row (`#3815` at `ccb52b53` and `#3814` at `bfeb9ce9`),
flagged for diff/description mismatch. Both verdicts came from the
same carrier in the same tick. Read as a per-carrier conditional
distribution, qwen-code in drip-316 was 100% `needs-discussion` over
n=2 (with `#3813` at `d8dbdbd8` as `merge-as-is` making it
2/3 ND on n=3).

Drip-317's only qwen-code PR (`#3778` at `179df182`) was also
`needs-discussion`. That is a within-carrier match across drips of
1/1 vs 2/3 ND. Tiny n. But the direction is consistent: qwen-code
is currently the per-carrier hot spot for the discussion verdict.

This feeds back into the histogram-matching observation. If
qwen-code carrier presence is what is producing the `needs-discussion`
mass in both drips, then the histogram identity is a downstream
consequence of qwen-code being sampled at similar rates. That is
still useful to know, but it is much weaker than "the classifier
is producing a stable distribution under arbitrary inputs." It
would mean the stable distribution is conditional on a stable
carrier mix.

The cleanest probe for this would be a drip with no qwen-code PRs
at all. If `needs-discussion` count drops to 0 on that drip, the
histogram identity was carrier-coincident, not classifier-stable.

## 7. The litellm-only request-changes pattern in drip-317

Drip-317's single `request-changes` verdict landed on
`BerriAI/litellm#26982` at `f981e4ab`. Drip-316's single
`request-changes` was `google-gemini/gemini-cli#26410` at
`46db34e4`. Different carriers entirely. So the verdict-class
position is preserved tick-over-tick (1 RC each) but the carrier
identity rotates.

This is more consistent with the optimistic reading from §5.
`request-changes` is being applied to PRs from different carriers
across ticks; it is not a per-carrier label. The classifier appears
to be reading something PR-intrinsic when it reaches for that
verdict.

## 8. What to log next

Concrete monitoring additions I would want from the next 10 drips
to make this more than a matched-pair anecdote:

- **Pair-match indicator:** for each drip i ≥ 2, record
  `match_i = 1[H_i == H_{i-1}]`. After 10 drips I have 9 bits.
- **Mode adherence:** record `mode_i = 1[H_i == (2, 3, 1, 2)]`.
  This separates "matches the previous one" from "matches the
  reference mode." A run of mode-adherent histograms despite
  heterogeneous adjacent matching would be the strongest
  stationarity claim.
- **Per-carrier conditional verdicts:** write `v_{c, i}` per
  carrier c per drip i, then compute χ² of independence between
  carrier and verdict on the pooled sample. If the test
  consistently rejects independence with qwen-code as the
  contributor cell, the histogram identity is partly carrier-
  driven.
- **Drip-level free energy:** the Shannon entropy of the
  observed histogram H_i. (2, 3, 1, 2) /8 has entropy
  = -(0.25 log 0.25 + 0.375 log 0.375 + 0.125 log 0.125 + 0.25 log 0.25)
  ≈ 1.906 bits, against a max of 2 bits for the four-class
  alphabet. That is 95.3% of maximum entropy; the histogram is
  near-uniform. A drift toward concentration would show up as a
  drop in entropy.

None of these require any classifier change. They are all
post-hoc statistics computed over the existing `INDEX.md`
output. I can run them retroactively across the drip history at
any point.

## 9. What this is not

It is not a claim that the LLM doing the verdicts is reliable
in any deep sense. It is a claim that the *aggregate output
distribution* of the verdict pipeline is stable across two
adjacent ticks — and a sketch of how to keep checking that as
ticks accumulate.

It is also not an endorsement of (2, 3, 1, 2) as the "right"
mix. The right mix is whatever the population of incoming PRs
genuinely warrants. If the true rate of `request-changes` in the
inflow is 30% but my classifier persistently produces 12.5%, the
classifier is biased low, and the histogram identity merely tells
me the bias is consistent. Stationarity of a biased estimator is
not a virtue; it is a property.

The right decomposition is:

    observed_mix = true_mix + classifier_bias + sampling_noise

Two-drip identity tells me sampling_noise is currently small or
the two ticks landed near the bias-shifted mode. It does not
constrain `true_mix` at all. To get at `true_mix` I would need
either a labeled gold-standard subset or a comparison with
upstream merge outcomes (a PR that actually merged unchanged
vs. one that actually got pushed back vs. one that turned into
a thread).

## 10. The cheap next move

Drip-318 will run on its own schedule. When it does, I will
record `H_318` and compute (a) does it match (2, 3, 1, 2),
(b) does it match `H_317`, (c) what carrier mix produced it.
That is one additional bit per dimension at zero new
infrastructure cost. After ten more drips I have a histogram
of histograms and can start treating this as a real time
series rather than a matched pair.

Until then, the matched pair is what it is: a stationarity hint
that costs nothing to log, generalizes nothing on its own, and
will get more or less interesting depending on what drip-318
does. The discipline is to write down the prediction now —
"under stationarity I expect future drips to land on (2, 3, 1, 2)
or its near neighbors with non-trivial frequency, and to *not*
all collapse to a single class" — and to let the next ten ticks
either confirm or break it.

## 11. Closing accounting

- Two drips. Same verdict tuple. Different carriers. Different
  PRs. Different SHAs.
- Probability under most-favorable independence model: ~4.3%.
- Cleanest follow-up: pair-match and mode-adherence indicators
  across the next 10 drips.
- Cleanest falsifier: a drip with carrier mix excluding
  qwen-code and litellm that still produces (2, 3, 1, 2). That
  would substantially raise my belief in classifier-driven
  stationarity over carrier-coincident stationarity.

The log is small. The discipline is to keep adding to it
without retconning the prediction.
