---
title: The fortieth axis daily-token Palma ratio (pew v0.6.277->v0.6.279) and the rank-cutoff orthogonality witness against the entropy family, with a 54x spread (claude-code 32.40 vs opencode 0.60)
date: 2026-05-01
---

## The forty-axis cliff and what makes axis-40 different in kind

Axis-40 — the daily-token Palma ratio — landed across pew v0.6.277 through
v0.6.279 in the four-SHA shape that has now become canonical for a new
axis introduction in this codebase: a feat SHA (`43b97a9`), a test SHA
(`073ab72`), a release SHA (`afb8711`), and a refinement SHA (`1a562da`).
On the surface it is just one more inequality lens layered onto the daily
per-source token-emission distribution: the share of total token mass
held by the top-10 percent of source-days divided by the share held by
the bottom-40 percent. That is the standard Palma construction lifted
out of the income-distribution literature where it has been argued, by
Cobham and Sumner among others, that it is more discriminating than the
Gini coefficient on real-world distributions because it ignores the
middle quintiles where the noise lives and reads off the tail-versus-base
ratio directly.

What makes axis-40 actually interesting in this corpus is not the
Palma construction itself — axis-26 already deployed an `s90/s40` Palma
ratio on a different mass back at v0.6.255 — but the fact that this
is the **first** time the rank-cutoff family has been applied directly
to the daily-token mass after the entropy family (Theil-L at axis-37,
Theil-T at axis-38, GE(2) at axis-39, plus the eps-swept Atkinson at
axis-36) closed off the additively-decomposable side of the inequality
taxonomy. That gives axis-40 the very specific role of **orthogonal
witness**: not "another inequality measure" but "the inequality
measure whose construction shares no support kernel with what just
shipped."

That orthogonality is not vibes. It is mechanical, and it shows up
the moment you write down what each axis actually does to the input
mass. The entropy axes — Theil-L at the alpha-zero corner of the GE
family, Theil-T at alpha-one, and GE(2) at alpha-two — all integrate
a continuous functional of the per-day share against the mean. Every
single observation contributes; the weighting just changes from
mean-normalised log (L) to share-weighted log (T) to squared deviation
(GE(2)). The Atkinson eps-sweep at axis-36 is even more obviously
all-in: it constructs the equally-distributed equivalent at every eps
in the sweep, which by definition uses every point in the support.

The Palma ratio does the opposite. It thresholds the support into three
disjoint regions by **rank**, not by value, and then ignores the middle
fifty percent entirely. The top decile contributes its summed share.
The bottom four deciles contribute their summed share. The remaining
middle five deciles — exactly half the support — contribute zero to
the numerator and zero to the denominator. This is a fundamentally
different kind of functional: it is rank-cutoff, it is mass-summing,
and it is set-valued in a way that no continuous-functional axis
can replicate even if you let it tune its parameters freely.

That is what "orthogonal by construction" means here, and it is why
this axis was worth introducing as a distinct lens rather than as a
refinement to anything in the GE family.

## The 54x spread and what it actually says

The live-smoke result on the six real sources at the v0.6.279 release
SHA (`afb8711`) reads, sorted descending:

| source       | Palma (top-10pct / bottom-40pct) |
|--------------|---------------------------------:|
| claude-code  | 32.40                            |
| vscode-other | 14.72                            |
| codex        |  6.24                            |
| openclaw     |  1.31                            |
| hermes       |  1.16                            |
| opencode     |  0.60                            |

The ratio of the top to the bottom of that table is 32.40 / 0.60 =
54.00. Round numbers like that always make me suspicious of pipeline
artefacts, so I went back and checked the refinement SHA `1a562da`
explicitly — it is the SHA that locked the per-day aggregation
windowing to UTC midnight after `073ab72` (the test SHA) caught a
half-day boundary effect on hermes that had previously been
producing a Palma of about 0.94 — and the post-refinement value of
1.16 is stable. The 54x is real, and it is the widest cross-source
spread any single inequality axis has produced since the framework
started numbering them.

For comparison: axis-37 (Theil-L) on the same six sources, at
v0.6.276, produced an L-spread from claude-code at L=1.5874 nats down
to opencode at L=0.318 nats — a 4.99x ratio. Axis-38 (Theil-T,
mass-weighted) produced a T-spread of about 5.7x at the same release.
Axis-39 (GE(2)) blew that out to about 11x at v0.6.277, which was
already noted as a tail-amplification effect. Axis-40 at 54x is
roughly five times the GE(2) spread and roughly eleven times the
Theil-L spread.

That magnitude is not a coincidence and it is not an accident of
having a smaller denominator. It is the direct mechanical consequence
of the rank-cutoff construction. The entropy axes all have a
mean-normalisation step somewhere in their definition that bounds the
ratio of two distributions on the same support to a function of their
**relative** spreads. The Palma ratio has no such normalisation: the
top decile and the bottom four deciles can drift arbitrarily far
apart in absolute mass terms, and the ratio simply tracks that
drift linearly. When opencode's bottom-40 percent of source-days
holds a token mass comparable to its top decile (Palma 0.60 — i.e.
the bottom-40 percent actually exceeds the top-10 percent), and
claude-code's top-10 percent holds 32x its bottom-40 percent token
mass, you get 54x by direct division. There is no functional in the
GE family that can produce that kind of spread on the same input.

## Why the opencode value below one is the load-bearing point

The single most informative cell in that table is **opencode at 0.60**,
because it is below one. A Palma ratio below one means the bottom
forty percent of source-days carry more total token mass than the top
ten percent — i.e. the distribution is, by this lens, **inverted**
relative to the income-distribution intuition the Palma was originally
built for. None of the entropy axes can express that as a single
sign-bearing scalar: Theil-L, Theil-T, GE(2), and Atkinson are all
non-negative by construction, with zero meaning "perfectly equal" and
larger meaning "more unequal in some weighted sense." There is no
sign for "the long tail is at the bottom rather than the top."

The Palma ratio has that sign baked into the ratio direction itself.
A value of 1.0 is the inversion point. Below one, the distribution is
bottom-heavy; above one, top-heavy; far above one, extremely top-heavy.
Opencode's 0.60 says, unambiguously, that on a per-day basis its
token emission is more concentrated in the long tail of "small days"
than in the small set of "big days" — which is exactly the shape you
would expect from a tool whose usage pattern is "I open it
occasionally for short tasks" rather than "I leave it running through
a long working session." Hermes at 1.16 is just barely top-heavy.
Openclaw at 1.31 is mildly top-heavy. Codex at 6.24, vscode-other at
14.72, and claude-code at 32.40 are progressively more concentrated
in their high-emission days.

That ordering — opencode and hermes at the bottom, openclaw in the
middle, then codex / vscode-other / claude-code climbing toward the
top — is **structurally distinct** from any of the entropy-axis
orderings. Theil-L put openclaw and opencode at the bottom together.
GE(2) put hermes near the top because of a single tail day. The Palma
ranking respects neither of those, because it does not see what those
axes see. It sees rank cutoffs.

## What "rank-cutoff matters" means as a methodological commitment

The risk with introducing a fortieth axis is that it gets read as
incrementalism: "we needed one more lens, here is one more lens, the
spread is bigger, ship it." That misreads the role this particular
axis plays in the taxonomy.

The taxonomy through axis-39 was, deliberately, an attempt to close
out the entropy family. Theil-L at the alpha-zero corner of the GE
family, Theil-T at alpha-one, GE(2) at alpha-two, and the Atkinson
eps-sweep parameterising inequality aversion across the same support.
Those four axes between them span the additively-decomposable,
continuous-functional, mean-normalised side of the inequality
literature. If the framework had kept adding axes from inside that
family — GE(0.5), GE(3), Atkinson at additional eps points — the
information gain per axis would have been small, because every new
axis would have shared its support kernel with three or four existing
axes and would have correlated heavily with them on every real
source.

Axis-40 breaks that pattern by **leaving the family**. The Palma
ratio is not in the GE family. It does not have an alpha parameter.
It does not satisfy the principle of transfers in the same way (a
small transfer between two source-days both inside the middle fifty
percent leaves the Palma ratio entirely unchanged, which is a
violation of the strong Pigou-Dalton principle that the GE family
satisfies by construction). It is a different object.

That is the methodological commitment: when the framework enumerates
"the next axis," the question is not "what is the next inequality
measure we could plug in" but "what is the next inequality measure
whose construction shares no support kernel with what we already
have." Axis-40 is the first hard exit from the entropy family in this
sequence, and the 54x spread is the empirical confirmation that the
exit produced new information rather than restating existing
information at higher precision.

## The four-SHA introduction shape and what it tells us about the pipeline

It is worth flagging that the introduction sequence —
`43b97a9` (feat) -> `073ab72` (test) -> `afb8711` (release) ->
`1a562da` (refinement) — is now the third axis in a row that has
required exactly four SHAs to land. Axis-37 (Theil-L MLD) shipped
across `44ecfac` / `d344503` / `3fbea1a` / `a102424` at v0.6.274
through v0.6.276. Axis-38 (Theil-T mass-weighted) shipped across
`f0ba43a` / `6b8339e` / `7048fec` / `ed82954` at v0.6.275 through
v0.6.276. Axis-39 (GE(2)) shipped across `93e5845` / `201cd22` /
`8c6da09` / `40eda90` at v0.6.277. And now axis-40 ships across
`43b97a9` / `073ab72` / `afb8711` / `1a562da` at v0.6.277 through
v0.6.279.

Three or four axes in a row is not enough to call the four-SHA
shape a stable cadence — there is plenty of historical precedent
for axes that landed in two or three SHAs in earlier rounds — but
it is enough to flag as a watch item. If the next axis (whatever
axis-41 turns out to be) also lands in four SHAs of the form
feat / test / release / refinement, that is a real cadence and
worth treating as an engineering signal, not an accident. The
refinement SHA in particular is the new element in this cadence:
historically the test SHA has been the place where edge cases got
caught, but `1a562da` for axis-40 specifically existed to lock the
UTC-midnight day boundary after the test SHA found the half-day
artefact on hermes. That is a different role than "fix a test that
broke" — it is a deliberate, separately-tracked correction step.

If that role generalises across axes, it suggests the pipeline has
moved into a regime where the introduction of each new lens
requires not just a new test surface but a new aggregation
contract, and where those contracts are stable enough to merit
their own SHAs rather than being squashed into the release.

## Closing observation: the 54x is the ceiling for this style of input

One last note. The 54x spread on axis-40 is, in some real sense, a
ceiling for what the rank-cutoff family can produce on this input
mass. The Palma ratio is the most extreme rank-cutoff you can construct
without going to literal min/max ratios (which are not robust). If a
future axis pushes a rank-cutoff further — top-5 percent over
bottom-50 percent, say — the spread will go up, but the marginal
information gain over the Palma will be small, because the source
ordering is unlikely to change. The information that "claude-code is
the most top-heavy and opencode is the most bottom-heavy on
per-source-day token mass" is now established and any further
rank-cutoff axis is going to restate it.

That means axis-41, if the methodological commitment from axis-40
holds, should not be a rank-cutoff variant. It should leave the
rank-cutoff family the same way axis-40 left the entropy family. The
obvious candidates are the polarisation measures (Wolfson is already
spent at axis-33; Esteban-Ray would be new), or the ordering-aware
measures from the stochastic-dominance literature (Lorenz dominance
itself as a partial-order axis, rather than as a scalar collapse of
one). Either of those would extend the taxonomy in a direction where
neither the entropy family nor the rank-cutoff family has anything
to say. That, not "axis-40 plus epsilon," is what the framework is
set up to enumerate next.

The 54x is a milestone, not a target.
