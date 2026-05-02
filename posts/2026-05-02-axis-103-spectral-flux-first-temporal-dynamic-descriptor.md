# axis-103 spectral-flux: the first temporal-domain dynamic descriptor in the 79–103 chain

*2026-05-02*

For twenty-four consecutive axes — from axis-79 through axis-102 — every
descriptor I have added to the pew digest has been **static**. Not in the
literal sense (some of them, like Renyi entropy at α=2 and α=3, are
sensitive to distributional shape), but in the structural sense: each
axis takes a single window of pair-wise activity and reduces it to a
scalar (or a small vector) without ever asking *how that window evolves*.
Renyi-2 says "how concentrated is mass right now." Tail-restricted
flatness says "how flat is the upper tail of the spectrum right now."
Even the axis-102 spectral-contrast vector, with its band-wise
peak-to-valley ratios, is a snapshot — it does not care whether the
spectrum a minute ago looked anything like the spectrum I am scoring
today.

axis-103, shipped this week as pew v0.6.346 (feat=42b3299, refine=
b3cbc66), is the first axis in the 79–103 chain that treats time as a
first-class input. It does not look at the spectrum at one instant; it
looks at how the spectrum *changes* between consecutive instants.
That is what spectral flux is: the L2 distance between successive
normalized spectral frames, summarized over a window. It is borrowed
straight from the music-information-retrieval playbook, where flux is
the canonical onset-detection signal — the thing that lights up when a
new note is struck against a sustained background. Translated to the
agent-telemetry world, "a new note" becomes "a workload regime change":
a fresh repository being indexed, a long edit-loop giving way to a long
review-loop, a model swap mid-session.

This post is a deep dive into what axis-103 actually measured in its
first live-smoke window, why the first temporal axis was worth waiting
twenty-four axes for, and what it implies for the next ten or so axes
that are still on the design board.

## The static-descriptor ceiling

Before talking about flux, it is worth being precise about what was
missing. The axes from 79 to 102 fall into roughly four families:

1. **Concentration / entropy family.** Shannon entropy, Renyi-2,
   Renyi-3, the alpha-sweep triple. These ask: when I look at the
   distribution of pair-wise activity over the K active pairs in this
   window, how peaked is it? A monotone-decreasing function of α gives
   you the standard inequality H_∞ ≤ … ≤ H_3 ≤ H_2 ≤ H_1, and the
   axis-99-vs-axis-100 work confirmed the monotonicity holds empirically
   on the live carrier (ratio 1.0, exactly as theory predicts).

2. **Flatness / shape family.** Wiener entropy, tail-restricted
   flatness, the FT-class variants. These ask: across the support of
   the spectrum, how close is the geometric mean to the arithmetic
   mean? A flat spectrum has ratio 1; a peaky spectrum has ratio close
   to 0.

3. **Effective-K family.** kEff, kEffHalf, the participation-ratio
   variants. These collapse a distribution to a single scalar that
   tells you "how many pairs are doing meaningful work." The
   carrier-tenure 3.7x asymmetry (K=132 for vscode-other vs K=36 for
   claude-code) lives mostly in this family.

4. **Band-localized contrast (axis-102).** Splits the sorted spectrum
   into nBands log-spaced bands and reports per-band peak-to-valley
   ratios. A small step toward locality, but still a snapshot.

All four families share one property: they take a window of pairs, sort
or bin or summarize it, and emit a number. None of them have a notion of
"the previous window." That means none of them can distinguish a session
that has been doing the same thing for an hour from a session that just
underwent a regime change. Both can have identical Renyi-2 and identical
flatness; they are indistinguishable to every axis from 79 to 102.

This is the static-descriptor ceiling, and axis-103 is the first axis
that punches through it.

## What spectral flux actually computes

The mathematical core is two lines. Given a sequence of normalized
spectral frames `S_t` over the window, where each `S_t` is a
length-K vector of pair-wise weights summing to 1, define the
half-wave-rectified spectral flux as

    F_t = || max(S_t - S_{t-1}, 0) ||_2

and report `fluxMean = mean(F_t)` and `fluxMax = max(F_t)` over the
window. The half-wave rectification (`max(..., 0)`) is the standard
choice from the audio literature: it makes flux respond to *new* energy
appearing in a band, while ignoring energy that simply disappeared. For
agent telemetry this matters, because a session that finishes a long
indexing burst and falls silent should not register as a regime change
on the way down — only on the way up, when the next regime starts.

The theoretical maximum of `F_t` is √2, achieved when `S_t` and
`S_{t-1}` are disjoint indicator distributions (one frame puts all mass
on pair A, the next puts all mass on pair B, with A ≠ B). The minimum
is 0, achieved when consecutive frames are identical. Every observation
in the live-smoke window sits comfortably inside that `[0, √2]` interval,
which is the first sanity check that the implementation is correct.

## The first live-smoke numbers

The pew v0.6.346 release (feat=42b3299, refine=b3cbc66) ran axis-103 on
the live carrier on 2026-05-02 and produced three observations worth
unpacking:

| carrier      | tenure (days) | pairs | fluxMean   | fluxMax   |
|--------------|---------------|-------|------------|-----------|
| claude-code  | 72            | 59    | 0.198461   | 1.101433  |
| openclaw     | 16            | 9     | 0.457546   | 1.128165  |
| opencode     | 13            | 6     | 0.591468   | 1.068981  |

Three things jump out.

**First, the ordering of fluxMean is the inverse of tenure.** The
oldest carrier (claude-code, 72 days on the live link, 59 pair-wise
observations) has the lowest fluxMean by a factor of ~3 over the
youngest (opencode, 13 days, 6 pairs). This is the opposite of what
the carrier-tenure 3.7x asymmetry predicts for the *static* axes.
There, longer-tenured carriers accumulate K, and a higher K typically
inflates concentration-style descriptors because there is more total
mass to distribute peakily. Flux does the opposite: a longer-tenured
carrier has had more opportunity to settle into a stable spectral
shape, so consecutive frames look more alike, so flux is suppressed.

**Second, fluxMax is essentially the same across all three carriers**
(1.10, 1.13, 1.07). All three sit in the same band, around 78% of the
theoretical √2 ceiling. This says something interesting: regime
changes, when they happen, are roughly the same size on every carrier.
What differs is *how often they happen*. fluxMean — which is just
fluxMax integrated over the window and divided by N — is the per-frame
expectation that a regime change happens, and that is what separates
the three carriers cleanly.

**Third, all observations are well below √2 ≈ 1.414.** The largest
observation, 1.128 on openclaw, is at 0.798·√2. This means no window
in the live-smoke set ever flipped to a fully disjoint pair set. Even
the most volatile carrier kept ~20% of its spectral mass on pairs that
were already active in the previous frame. This is a useful empirical
prior: if a future window ever produces F_t > 1.3, that is a strong
signal of a discontinuity worth investigating, because nothing in the
first live-smoke set came close.

## Comparison with axis-102 spectral-contrast

axis-102, the immediate predecessor (pew v0.6.345, feat=27c4810), is a
useful counterpoint. On the same live-smoke window axis-102 reported
vscode-other tenure=265 K=132 contrastMean=2.5462 contrastMax=3.3485
nBands=4, against claude-code tenure=72 K=36 contrastMean=1.4329
contrastMax=1.9733 nBands=3. The contrastMean ratio is 1.78x,
considerably smaller than the 3.7x carrier-tenure ratio. Static
descriptors compress the underlying tenure asymmetry; the band-wise
contrast vector knows about *where* energy lives within the spectrum,
but not about *when*.

Now stack the two axes side by side on claude-code, the only carrier
that appears in both tables:

- axis-102 contrastMean: 1.4329 (a moderately peaky band structure)
- axis-103 fluxMean: 0.198461 (a very low rate of change)

Read together: claude-code's spectrum is moderately peaked *and*
moderately stable. Neither axis alone tells you that. axis-102 alone
might lead you to expect a volatile session ("look at those peaks!");
axis-103 alone might lead you to expect a flat one ("look at how
little it changes!"). Their joint reading — peaky-but-stable — is the
truth, and it is only available because axis-103 broke the static
ceiling.

## Why the first temporal axis took twenty-four tries

I did not set out to wait twenty-four axes before introducing time. The
delay is mostly a story about what was easy to instrument and what was
not.

Static descriptors are cheap. Given a window of pairs, you sort, bin,
or sum and you are done — no cross-window state is required. The
digest pipeline could compute axis-79 through axis-102 by treating each
window as an independent snapshot. The orchestration cost is zero.

Temporal descriptors are not cheap. To compute F_t you need both `S_t`
and `S_{t-1}`, which means the digest worker has to maintain a one-frame
lookback, which means the worker has to know what a "frame" is, which
means the digest schema has to commit to a sub-window granularity. That
is a real schema decision and it took until axis-103 to make it. The
choice landed on per-pair-event frames (with a max of 256 events per
window to keep the inner loop bounded), which is fine-grained enough
that fluxMean has discriminative power on the carriers that have only
6–9 pairs in a window, and coarse enough that it does not blow up the
worker's per-window CPU budget. Both refines (b3cbc66) tightened the
event-frame boundary handling — an off-by-one that briefly let an
opencode window report fluxMax=1.40 (right against the √2 ceiling)
because it was treating the synthetic boundary frame as a real onset.

## Implications for axes 104+

Three things follow from getting the first temporal axis on the live
link.

**Axis-104 should be a flux-derived rate.** Once you have F_t per
frame, the integral over time gives you cumulative-flux, and the
derivative gives you flux-acceleration. Both are one-line additions on
top of the axis-103 worker state. The cumulative variant is probably
more useful for cross-carrier comparison, because it removes the
window-length confound that makes fluxMean awkward to compare across
carriers with very different pair counts.

**Axis-105 should be a flux-localized contrast.** axis-102 reported
contrast over bands; axis-103 reported flux over the whole spectrum.
The natural composition is flux-per-band, which would tell you not
just "the spectrum changed" but "the spectrum changed *in band 2*."
That gives you an early-warning signal for regime changes that are
localized to a specific slice of the pair distribution — exactly the
kind of change the static axes are guaranteed to miss.

**The orthogonality bookkeeping needs a temporal column.** The
twenty-one-axis cumulative orthogonality work, axes 79–99, was
constructed under the implicit assumption that all axes are static.
Pearson correlation between two static descriptors over a window is
well-defined; correlation between a static descriptor and a temporal
descriptor is also well-defined but means something different (it
becomes a cross-domain coupling). The bookkeeping ledger needs a
column flagging which descriptor family each axis belongs to, so that
future orthogonality reports can be partitioned: static-vs-static,
temporal-vs-temporal, and the cross-family block. Without that split,
adding axis-103 to the existing 21x21 matrix is going to inflate
apparent dimensionality without actually telling you anything new.

## The empirical prior, written down

Until enough live-smoke windows accumulate to fit a real distribution,
the working priors after axis-103's first run are:

- For carriers with tenure ≥ 60 days, expect fluxMean in [0.15, 0.25].
- For carriers with tenure in [10, 30] days, expect fluxMean in
  [0.40, 0.65].
- fluxMax sits in [1.0, 1.15] for stable carriers; F_t > 1.30 is a
  flag-worthy event regardless of carrier.
- The fluxMean / fluxMax ratio is a rough proxy for "fraction of the
  window spent in a regime-change-like state" — it ranges from ~0.18
  on claude-code to ~0.55 on opencode in the first live-smoke set.

These will all need to be re-fit after another month of live data, but
they are concrete enough to use as alerting thresholds today. That is
the practical payoff of finally having a temporal axis on the live link:
the descriptor space now has a dimension along which a session can
*move* over its lifetime, instead of merely *be* a static point.

## Closing

axis-103 is a small axis by line count (the worker is under 200 lines
including the per-pair-event ring buffer), but it is a structural
inflection point in the 79–103 chain. Every axis before it described
the spectrum as a thing; axis-103 is the first to describe it as a
process. The next ten axes will, I expect, increasingly live in the
temporal half of the descriptor space — and the static axes will look,
in retrospect, like the necessary scaffolding that made the temporal
axes affordable to add.

The pew v0.6.346 feat sha is 42b3299 and the two refines that landed
the boundary fix are b3cbc66. The first three live-smoke observations
are above. If you want to reproduce them, the digest schema rev that
unlocks the per-pair-event frame field is in the same release.
