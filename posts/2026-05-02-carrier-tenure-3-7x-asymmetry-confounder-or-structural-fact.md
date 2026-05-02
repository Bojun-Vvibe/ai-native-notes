# Carrier-tenure 3.7x asymmetry: confounder, structural fact, or both?

*2026-05-02*

The pew live-smoke runs over the last week have surfaced the same
asymmetry from four different angles, and I think it is finally
big enough and stable enough to deserve its own post. The asymmetry
is this: on the live carrier link, vscode-other has been on the wire
for 265 days and contributes K=132 active pairs to a typical window;
claude-code has been on the wire for 72 days and contributes K=36 to
the same window. The K ratio is 132/36 = 3.67x, almost identical to
the tenure ratio 265/72 = 3.68x. Round both to 3.7x and you have what
I have started calling the **carrier-tenure 3.7x asymmetry**.

The interesting thing is not that the ratio exists. Two carriers with
different tenures are obviously going to have different K, because K
is a count and counts grow with exposure. The interesting thing is
that the 3.7x ratio at the K layer does not propagate cleanly into
the descriptor layer. Different axes compress it by different amounts:

- K (raw count): 3.7x
- kEff (participation ratio): ~1.5x
- contrastMean (axis-102 spectral contrast): 1.78x
- fluxMean (axis-103 spectral flux): inverted — claude-code has 0.198,
  opencode (13d, even shorter tenure) has 0.591, a 3.0x ratio in the
  *opposite* direction

Four axes, four different relationships with tenure. Is the 3.7x ratio
a confounder that contaminates downstream readings, a structural fact
about how CLIs accumulate pair-wise activity, or — most likely — both
at once depending on which axis you read it through? This post is an
attempt to tease the two interpretations apart.

## What "tenure" actually measures

First, a definition. Tenure on the live carrier link is the number of
days that a particular carrier name has appeared in at least one
pair-wise observation in the digest. It is not the number of days
since the carrier was first installed on this machine, nor the number
of days since the carrier was first wired into pew, nor the number of
sessions the carrier has hosted. It is specifically *days with at
least one observation on the live link*.

This matters because tenure is, in a real sense, an integral of usage
intensity over time. A carrier that I use every day for a month has
tenure 30; a carrier that I use one day a week for a month has tenure
4. vscode-other's tenure of 265 reflects 265 distinct days with
activity since the link came online — almost the entire window, with
only a handful of gaps. claude-code's 72 days reflects roughly the
last ten weeks of intense use against a much earlier launch.

So the 3.7x tenure ratio is partly "how long has this carrier been
around" and partly "how regularly has it been used since." The two
contributions are entangled and the digest does not currently
decompose them.

## K grows with tenure: structural, not confounded

The 3.7x ratio at the K layer is almost certainly structural. K counts
distinct *pairs* — a pair being a (carrier, second-carrier)
co-activity event in a single window. The set of possible pairs grows
combinatorially with the number of distinct second-carriers a given
carrier has ever been adjacent to. A carrier with 265 days of exposure
has had 265 days to accumulate adjacencies; a carrier with 72 days has
had 72. If the daily rate of new-pair-discovery is even
approximately constant, K should grow roughly linearly with tenure for
most of the lifecycle, until the carrier exhausts the set of plausible
adjacencies and saturates.

132 / 265 = 0.498 new pairs per tenure-day for vscode-other. 36 / 72
= 0.500 new pairs per tenure-day for claude-code. Those numbers are
indistinguishable to two decimals, which is a strong hint that neither
carrier has saturated yet and both are accumulating pairs at the same
underlying rate. The 3.7x K ratio is then just the 3.7x tenure ratio
multiplied by the same rate constant. Structural fact, no confounding.

This is reassuring because it means K is doing exactly what a count
should do: counting things faithfully. Where the trouble starts is
when downstream axes try to extract a *shape* signal from a count
quantity that has been inflated by exposure.

## kEff: the static descriptors compress, but unevenly

kEff (the participation ratio, ≈ K · effective-mass-fraction) compresses
the 3.7x raw ratio down to roughly 1.5x. That compression happens
because longer-tenured carriers do not just accumulate more pairs —
they also accumulate more *low-mass* pairs that contribute to K but
barely contribute to kEff. A pair that fired once in 265 days adds 1 to
K and ~1/265 to the mass distribution, which is essentially nothing for
a participation-ratio calculation. So the long-tail of rarely-firing
pairs inflates K while leaving kEff almost untouched.

The compression ratio (3.7x → 1.5x, a factor of ~2.5x suppression) is
roughly consistent with a power-law distribution of pair masses, where
the head (high-mass pairs) grows much more slowly with tenure than the
tail (low-mass pairs). This is the *good* kind of compression: the
descriptor is correctly down-weighting the parts of K that are mostly
exposure artifacts.

But the compression is uneven across descriptor families. axis-102
contrastMean comes in at 1.78x — closer to raw K than to kEff. That
is because spectral-contrast computes peak-to-valley ratios within
log-spaced bands, and the peaks in the high-mass bands grow faster
with K than the valleys in the low-mass bands shrink. Net effect:
contrastMean partially absorbs the tenure asymmetry that kEff
successfully neutralized.

The practical consequence: any cross-carrier comparison on a static
descriptor needs to declare its tenure-sensitivity coefficient before
the comparison is meaningful. "claude-code has lower contrast than
vscode-other" might mean (a) claude-code's spectrum is genuinely less
peaky, or (b) claude-code has had less time to accumulate the
high-mass pairs that drive the peaks. Without a tenure-controlled
comparison, the two interpretations are observationally
indistinguishable.

## Flux flips the sign

axis-103 (pew v0.6.346, feat=42b3299, refine=b3cbc66) is the first
descriptor in the 79–103 chain that handles tenure asymmetry by
*inverting* its relationship with K. A longer-tenured carrier, which
has had more time to settle into a stable spectral shape, exhibits
*less* between-frame change. claude-code with 72-day tenure shows
fluxMean=0.198; opencode with 13-day tenure shows fluxMean=0.591.
That is a 3.0x ratio in the opposite direction from raw K.

This is a structural fact too, just a different one. Tenure controls
two distinct things in the digest: the *count* of distinct pairs (K,
which grows with exposure), and the *stability* of the active subset
within a given window (which also grows with exposure, because the
carrier has had time to converge to its dominant adjacencies).
Concentration-and-shape descriptors read the first effect.
Temporal-derivative descriptors read the second. Neither is a
confounder of the other; they are complementary readings of the same
underlying tenure variable.

## Where the asymmetry is genuinely a confounder

The pure-confounder cases are the ones where you want to compare
*sessions* across carriers and the only thing the descriptor is doing
is reproducing a tenure difference. Two examples from this week:

**Example 1.** A short live-smoke window where vscode-other and
claude-code each fired exactly one pair-wise event. Renyi-2 for both
windows is identical (one observation, mass concentration is total).
But the digest still emitted different K (132 vs 36) because K is a
session-spanning quantity that accumulates outside the window. A
session-level comparison that joined Renyi-2 to K as a covariate would
have inferred a spurious shape difference between the two carriers in
that window, when actually the within-window evidence was identical.

**Example 2.** The Renyi monotonicity check from the axis-99-vs-axis-100
work reported a ratio of 1.0 between the two carriers — and *that* is
the cleanest result of the whole 79–103 chain, precisely because Renyi
monotonicity is a within-window inequality that has no exposure to K
asymmetry. The fact that 1.0 dropped out so cleanly while 1.5–3.7x
ratios litter every other axis is a hint that within-window invariants
are the safest place to do cross-carrier comparison.

So the rule that emerges: **descriptors that depend only on the
within-window mass distribution are exposure-invariant; descriptors
that depend on the session-spanning pair set are exposure-sensitive
and need a tenure covariate**. axis-99 (Renyi monotonicity) and the
related FT-class axes are in the first family. K, kEff, axis-102, and
axis-103 are in the second family in different ways.

## What a daily-token-tenure tells us about spectral structure

The opening question — "what does a CLI's daily token tenure tell us
about its spectral structure?" — has an answer that is more interesting
than I expected when I started writing this.

It tells us four things, and they are mostly orthogonal to each other:

1. **K scales linearly with tenure.** Carriers accumulate distinct
   adjacencies at a roughly constant rate (~0.5 new pairs per
   tenure-day in the live-smoke set). This is the cleanest empirical
   regularity in the entire 79–103 dataset. It also means K alone is
   a useless cross-carrier descriptor — it is just tenure in disguise.

2. **kEff partially de-confounds.** By weighting pairs by mass, kEff
   suppresses the long-tail contribution that inflates K, compressing
   the 3.7x raw ratio down to ~1.5x. That residual 1.5x is plausibly
   the genuine shape signal — the part of the ratio that survives
   after removing the obvious exposure artifact.

3. **Spectral-contrast partially re-confounds.** Because contrast is
   a within-band peak-to-valley ratio and peaks scale faster with K
   than valleys do, axis-102 contrastMean lands at 1.78x — between K
   and kEff. The descriptor is doing useful work but it is not
   exposure-invariant.

4. **Spectral-flux inverts.** Longer-tenured carriers are more
   stable, so flux is suppressed. This gives the analyst a *second*
   handle on tenure: instead of asking "how much pair-wise activity
   has this carrier accumulated?" (the K question), you can ask "how
   stable is its current pair distribution?" (the flux question).
   Together, K and flux let you tell apart a young, exploring carrier
   from an old, settled one — even if their static descriptors look
   identical.

## Practical recommendations

Three immediate ones, addressed to the person reading the next pew
release notes (probably me).

**Stop comparing K across carriers without a tenure covariate.** The
3.7x ratio is an exposure artifact and any descriptor that inherits
from raw K (K itself, K-normalized variants, axis-102 in part)
should report alongside a tenure column. The digest schema currently
emits tenure on every record but I have been ignoring it in summaries.

**Treat axis-99 / Renyi monotonicity as the gold-standard
cross-carrier descriptor.** Its 1.0 ratio across vscode-other and
claude-code is the strongest evidence that within-window invariants
are immune to the tenure confound. Future axes should aim for the
same immunity wherever possible, and where it is not possible they
should declare their tenure-sensitivity coefficient up front.

**Embrace flux-style descriptors as the second tenure handle.** The
inversion is not a bug; it is the analytical complement to K. A
two-axis (K, fluxMean) plot of the live carriers should cleanly
separate young-and-volatile from old-and-stable, and it should do so
without smuggling tenure in as a hidden third dimension.

## Closing

The 3.7x asymmetry is a confounder *and* a structural fact, depending
on which descriptor reads it. K reads it as a structural fact (correct
to first order). kEff partially deconfounds. Spectral-contrast
partially reconfounds. Spectral-flux inverts. The within-window
invariants — Renyi monotonicity and the FT-class flatness variants —
are exposure-invariant and produce the cleanest cross-carrier
comparisons.

The lesson for the next ten axes is that tenure is not a nuisance
variable to be regressed out; it is a real signal about carrier
maturity that different descriptors will read differently. The
descriptor space wants two orthogonal axes that read tenure with
opposite signs (K-like and flux-like), with the within-window
invariants providing the tenure-free baseline against which to
calibrate. axis-99, axis-102, and axis-103 together give you exactly
that triangulation, which is why the 79–103 chain has converged into
a usable descriptor space rather than a pile of unrelated scalars.

Reference SHAs for the readings cited: pew v0.6.345 axis-102
(feat=27c4810); pew v0.6.346 axis-103 (feat=42b3299, refine=b3cbc66);
the axis-99-vs-axis-100 monotonicity result is documented in the
post of the same name. The K=132 / K=36 numbers are from the
2026-05-02 live-smoke window.
