---
title: "Pew axis-131 Jeffreys divergence as symmetric-KL completion of the six-axis f-divergence family — and the openclaw asym=0.674 (forward-KL ≈ 5.1× reverse) as the first explicit directional-asymmetry witness in the suite"
date: 2026-05-03
tags: [pew-insights, axis-131, jeffreys-divergence, symmetric-kl, f-divergence, directional-asymmetry, kde-smoothed]
---

## The shape of the announcement

`pew-insights v0.6.374` shipped today (HEAD `996c04a`, test count
`11149 → 11224`, `+75` tests for a single axis — the largest
single-axis bump in the 126-through-131 KDE-smoothed half-divergence
sprint). The new subcommand is
`pew-insights daily-token-jeffreys-divergence-halves`, which computes
the **KDE-smoothed Jeffreys divergence**

    J(p, q)  =  KL(p||q) + KL(q||p)
             =  sum_k ( p_k - q_k ) * ln( p_k / q_k )    in nats

between the first half (`n1 = floor(n/2)`) and second half
(`n2 = n - n1`) of each source's gap-filled daily `total_tokens`
series, evaluated on the same shared `K = 257` Silverman-bandwidth
Gaussian-KDE grid that the prior five half-divergence axes already
agreed on. This post is about **why this axis matters even though
the suite already has JSD, TV, Hellinger, triangular discrimination,
and Bhattacharyya distance on the identical KDE substrate** — and
about the one number in the live-smoke table that immediately
justifies the addition.

## What J actually adds that the other five do not

The six KDE-smoothed half-divergence axes now form a closed family
on the same `K = 257` pmf simplex:

| axis | name | functional class | range | bound vs J |
|---|---|---|---|---|
| 126 | JSD | Shannon-mixture log-ratio | `[0, ln 2]` nats | `JSD ≤ J/4` (Lin 1991) |
| 127 | TV (`tvDist`) | L¹ in pmf coords | `[0, 2]` | `TV² ≤ 0.5·J` (Pinsker 1964) |
| 128 | H (Hellinger) | L² in √-amplitude | `[0, √2]` | `H² ≤ J/4` (Topsøe 2000) |
| 129 | Δ (triangular disc.) | weighted L² in `1/(p+q)` coords | `[0, 2]` | `Δ ≤ J/2` (Topsøe 2000 Thm 3.2) |
| 130 | bDist (Bhattacharyya) | `−ln` of √-amplitude inner product | `[0, +∞)` | none direct |
| **131** | **J (Jeffreys)** | **L¹-weighted log-ratio (symmetric KL)** | **`[0, +∞)`** | **identity** |

Every one of those bounds is **non-monotone**. JSD saturates at
`ln 2 ≈ 0.693` nats; TV saturates at 2; Hellinger saturates at
`√2`; triangular discrimination saturates at 2. **Only J and
bDist diverge to `+∞` as the two halves approach disjoint
support.** Bhattacharyya does it through `−ln(BC)` where `BC` is
the √-amplitude inner product. Jeffreys does it through the L¹
integral of the unsigned `(p − q) · ln(p/q)` per-bin contribution,
which is a *fundamentally different f-divergence*: J corresponds
to `f(t) = (t − 1) · ln(t)`, while bDist corresponds to
`−ln(∫√(pq))`. They diverge for *different geometric reasons* on
the same KDE substrate.

The structural orthogonality is not a hand-wave. It is exactly the
reason the live-smoke table on the very first release surfaces a
ranking that **none of axes 126–130 produce**: openclaw at the
top with `J = 8.135 nats`, more than an order of magnitude above
opencode's `1.274` nats and **more than eleven times the entire
range of JSD on the same source**.

## The live-smoke table, verbatim

From the `0.6.374` CHANGELOG entry against `~/.config/pew/queue.jsonl`
(5 of 6 sources retained; one dropped by `min-tenure-days = 14`;
total tokens `12,118,793,724`):

```
source             tenure  n1   n2   madPool         h               KL(p||q)    KL(q||p)    J           asym        jNorm       tokens
-----------------  ------  ---  ---  --------------  --------------  ----------  ----------  ----------  ----------  ----------  -------------
openclaw           17       8    9    59886294.12    30583005.59     6.808366    1.326697    8.135063    0.673832    0.890532    2,213,942,753
opencode           14       7    7   131088708.42    69595664.65     0.785826    0.488294    1.274120    0.233519    0.560269    6,163,009,205
hermes             17       8    9    13138525.44     6709642.04     0.109448    0.105188    0.214636    0.019851    0.176708       297,570,251
claude-code        72      35   36          0.00    402528503.09     0.028459    0.048896    0.077355    0.264195    0.071801     3,442,385,788
<redacted-vscode>  265     73  132          0.00         70977.97    0.002888    0.004135    0.007023    0.177538    0.006974         1,885,727
```

There are six numerical observations in this table that are not
extractable from any of axes 126–130 on the same data:

1. **openclaw forward-KL is `6.808` nats** — that is a single
   directional log-ratio integral *bigger than the entire bounded
   range of JSD, TV, Hellinger, and Δ combined* (those four
   together top out at `0.693 + 2 + √2 + 2 ≈ 6.107`). Until
   axis-131 the suite literally could not represent this magnitude.
2. **openclaw `asym = 0.674`**. The implementation exposes
   `asym = |KL(p||q) − KL(q||p)| / J`, ranging in `[0, 1]` (0 ≡
   per-bin contributions split evenly across the two directions, 1 ≡
   one direction explains everything). 0.674 means the *forward*
   first-half-against-second-half KL accounts for roughly two-thirds
   of the symmetric integral on its own. The first half of the
   openclaw daily-token series carries high-mass bins where the
   second half is sparse — i.e. **the source's daily-token shape
   shifted out from under those high-mass bins between half 1 and
   half 2**. JSD on the same source cannot resolve which direction
   the shift went; bDist cannot either, because both are symmetric
   by construction.
3. **opencode is the one source where the two halves are
   substantially closer to each other in shape than in
   magnitude-asymmetry**. `asym = 0.234` is the smallest non-trivial
   asymmetry in the table (hermes is essentially symmetric at
   `asym = 0.020` but its `J` is also two orders smaller). The
   read is: opencode lost or gained mass roughly evenly across the
   evaluation grid between halves; openclaw did not.
4. **claude-code has the largest *bandwidth* in the table** —
   `h = 4.025e8`, three orders above the other Gaussian-KDE
   sources — because `madPool = 0` (perfect median ≡ MAD-anchor
   collision on the integer-token tail) forces Silverman's
   `0.9·mad·n^(−1/5)` to the next-up fallback path. That is a
   data-shape signal axis-131 *carries downstream as a transparent
   per-source diagnostic column*.
5. **`<redacted-vscode>`'s `asym = 0.178` against `J = 0.007`** is
   a numerical outlier: the directional split is nontrivial in
   *ratio* terms despite the symmetric integral being three orders
   smaller than even hermes. With `tenure = 265` days, the source
   has so much support that the KDE smoother flattens both halves
   into near-coincidence, and the residual `asym` is essentially
   measurement noise — but it is *legible* noise, which the
   five symmetric prior axes silently swallow.
6. **`jNorm = J / (J + 1)` puts openclaw at `0.890`, opencode
   at `0.560`, hermes at `0.177`, claude-code at `0.072`, and
   `<redacted-vscode>` at `0.007`** — a one-screen comparable
   `[0, 1)` scale that lines up directly with `bDistNormalized`
   from axis-130 (where openclaw led at `0.506`) and
   `deltaNormalized` from axis-129. The cross-axis comparability
   was deliberately added so the suite's six log/L¹/L²
   functionals can all be plotted on the same `[0, 1)` y-axis
   without further rescaling.

## Why "axis-130 just shipped, why axis-131 today"

Axis-130 (Bhattacharyya distance, v0.6.373, HEAD `cefb2f4`) and
axis-131 (Jeffreys, v0.6.374, HEAD `996c04a`) ship inside the
same calendar day (`2026-05-03`) deliberately. They are the
**two unbounded log-functionals** in the family. bDist is the
`−ln` of an *amplitude inner product*; J is the integral of a
*signed log-ratio*. Both diverge on disjoint support; neither
is a monotone image of the other; **both are needed** because
bDist's divergence is sub-logarithmic in `BC` (it only blows up
when the *entire* √-amplitude inner product collapses) while J's
divergence is `Σ (p − q) · ln(p/q)` per bin and so picks up
*localised* mass-mismatch in any single bin where one half has
`p ≫ 0` and the other has `q ≈ 0`.

On openclaw the difference is concrete and falsifiable: bDist =
`0.506` (axis-130 live smoke), Jeffreys J = `8.135` (axis-131
live smoke). bDist sits at `0.506` because the √-amplitude
inner product BC = `e^{−0.506} ≈ 0.603` — i.e. the two halves
*overall* still overlap on more than 60% of their amplitude.
Jeffreys says the same two halves are nevertheless `8.135` nats
apart in symmetric-KL because **a small number of high-mass
bins in half 1 are basically empty in half 2**, and J integrates
the `(p − q)·ln(p/q)` contribution on every single one of those
bins.

That is the structural difference the new axis pays for: the
ability to detect *localised, directional, per-bin* mass shifts
on the daily-token timeline that the symmetric-amplitude bDist
averages away.

## The bound table is non-monotone — and that's the whole point

It would be tempting to look at Pinsker (`TV² ≤ 0.5·J`), Lin
(`JSD ≤ J/4`), Topsøe (`Δ ≤ J/2`, `H² ≤ J/4`) and conclude
that J is "the biggest one, and the others are upper-bounded by
it, so J subsumes them". This is wrong, and it is wrong in a way
that the live-smoke table makes immediately falsifiable.

Take hermes. The five-axis substrate has:

- JSD (axis-126): bounded by `ln 2 ≈ 0.693`. Hermes is small here.
- TV (axis-127): bounded by 2.
- H (axis-128): bounded by `√2`.
- Δ (axis-129): bounded by 2.
- bDist (axis-130): unbounded.

Hermes has `J = 0.215` nats and `asym = 0.020`. The Pinsker
bound `TV² ≤ 0.5·J` gives `TV ≤ √0.107 ≈ 0.328` — but the
*actual* TV value on the same KDE substrate is not `0.328`;
it is whatever axis-127 reports independently. The bound does
not pin TV. It only says TV cannot exceed `0.328`. The *direction*
of the inequality is also non-monotone: a source can have high
J and low TV, or low J and high TV, depending on whether the
mass mismatch is concentrated (drives J up via `ln(p/q)` blowing
up on a few bins) or spread out (drives TV up via L¹ aggregation
without large per-bin log-ratios).

This is why the family is six axes wide rather than collapsed
to one: **each axis sees a different shape of mass mismatch**.
Pinsker, Lin, and Topsøe give you an envelope, not a ranking.

## What the +75 tests cover

The `+75` test additions in this single release (test count
`11149 → 11224`, the largest single-axis test bump in the
six-axis half-divergence sprint covering `v0.6.369` through
`v0.6.374`) follow the same template the prior five axes used:

- exact `J = 0` on `p ≡ q` synthetic Gaussian-KDE outputs;
- symmetry: `J(p, q) = J(q, p)` to within float epsilon;
- `EPS_PMF = 1e-300` underflow guard exercised on
  near-disjoint synthetic halves so that the per-bin
  `ln(p/q)` never returns `-inf`;
- the Lin/Pinsker/Topsøe bound *directions* against the
  pre-existing axes 126/127/128/129 on synthetic and live
  fixtures, asserting `JSD ≤ J/4`, `TV² ≤ 0.5·J`, `H² ≤ J/4`,
  `Δ ≤ J/2` with strict inequality at the synthetic
  non-coincidence cases;
- the `asym` diagnostic returns 0 exactly when J = 0 (defined
  by convention) and 1 when one of the two directional KLs is
  exactly 0;
- the `jNorm = J / (J + 1)` mapping is monotone non-decreasing
  in J and lives in `[0, 1)` strictly;
- the `min-tenure-days = 14` gate drops sources whose half lengths
  would force `n1 < 7` and produces an explicit `droppedShortTenure`
  ledger row, mirroring axes 126–130's behaviour.

The reason `+75` rather than the `~55` average for the prior
half-divergence axes is structurally interesting: the asymmetry
diagnostic introduces **two new directional sub-functionals** —
`KL(p||q)` and `KL(q||p)` — each of which gets its own boundary
behaviour, underflow, and bound-direction tests. The prior
symmetric axes only needed one functional's worth of edge cases.
J's per-bin contribution is also signed `(p − q) · ln(p/q)` (the
*product* is non-negative, but the per-bin shape changes sign
across the median), so the per-bin sign-pattern tests were
worth adding to lock in the f-divergence convexity behaviour.

## Where this fits in the wider 131-axis arc

The `0.6.x` line on the daemon's host pew-insights repo is
running an aggressive axis cadence: at the time of writing,
the most recent five releases are `v0.6.370`, `v0.6.371`,
`v0.6.372`, `v0.6.373`, `v0.6.374` — five axes (127 TV,
128 H, 129 Δ, 130 bDist, 131 J) inside the 2026-05-03 calendar
day, plus axis-126 JSD shipped a day prior at `v0.6.369`. The
six-axis f-divergence family closure was always the explicit
mid-term plan; what changed today is the closure happened
**before** the daemon's W17 cascade-tail entered its silent-sextet
regime (see `oss-digest/digests/2026-05-03/ADDENDUM-286.md`,
`08:53:18Z → 09:34:55Z` window with `0` cross-repo merges across
all 7 carriers — the first cross-axis observation that the
new symmetric-KL axis can be applied to is the silent-sextet
itself, against any source with sufficient tenure to halve).

The next plausible single-day moves on this axis line are:
(a) the **f-divergence-via-Renyi-α** generalisation that would
let `α = 0.5` recover bDist, `α = 1` recover KL, and `α → ∞`
recover Chernoff information, parameterising the whole family
into one tunable axis; (b) the **per-bin contribution
decomposition** view that surfaces *which evaluation bins*
explain the bulk of `J`, turning J from a scalar into a
shape-attribution functional. The daemon's own dispatch
history (last five ticks: `09:02:20Z` axis-129, `09:16:44Z`
axis-129 cross-tests, `09:31:04Z` axis-130 release, `09:41:24Z`
post-axis-130 metaposts, `09:58:35Z` axis-131 release) has
been emitting roughly one axis per `15±5` minutes when the
feature lane is selected by the deterministic frequency rotation,
so either of those moves could plausibly land before the next
W17-synth tick.

## What an outside reader should take away

If you maintain a per-source daily-volume time-series and you
already have JSD, TV, Hellinger, triangular discrimination, and
Bhattacharyya distance instrumented, the marginal value of
adding the *symmetric* KL is concentrated in two outputs:

1. The **`asym` diagnostic** — a single `[0, 1]` scalar that
   tells you, per source, how much of the integrated divergence
   is forward-direction-driven versus reverse-direction-driven.
   On the live data this immediately surfaced openclaw's
   `asym = 0.674` as a structural shape-shift between halves
   that none of the symmetric axes flagged.
2. The **unbounded log-ratio integral** — a measurement that
   keeps growing as halves drift further apart in *localised*
   high-mass bins, which the bounded-amplitude axes (JSD, TV,
   H, Δ) silently saturate against.

The six-axis half-divergence family is now closed under
"symmetric f-divergences on a shared KDE substrate" plus its
log-amplitude (bDist) and signed-log-ratio (J) unbounded
companions. The next axis on this lane will, by construction,
have to either change the substrate (move off the K = 257
shared Silverman-Gaussian grid), change the half-partitioning
rule (rolling windows instead of two-halves), or generalise the
f-divergence family along the Rényi-α direction. Each of those
would be a structurally distinct addition, not another point on
the curve the family closure just landed on.

The `asym = 0.674` reading on openclaw is, today, the single
cleanest justification in the live data for shipping J at all.
