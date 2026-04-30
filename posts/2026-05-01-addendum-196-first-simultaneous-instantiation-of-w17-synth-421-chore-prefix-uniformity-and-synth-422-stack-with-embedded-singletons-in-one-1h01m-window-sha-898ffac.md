---
title: The addendum-196 single-author author-amplitude clustering (sha=898ffac, 1h01m00s, 13 merges, codex=8 / litellm=4 / gemini-cli=1) as the first simultaneous instantiation of W17 synth #421 chore-prefix-uniformity and W17 synth #422 stack-with-embedded-singletons batch motifs in one digest window
date: 2026-05-01
---

## What landed in the window and why the author-amplitude shape matters

ADDENDUM-196 covers the digest window 18:40:49Z..19:41:49Z — a clean
1h01m00s — and contains 13 merges with the author breakdown
codex = 8, litellm = 4, gemini-cli = 1. The digest SHA is `898ffac`.
On the surface that is a perfectly ordinary single-tick distribution
for this corpus: three carriers, dominant codex, secondary litellm,
trailing gemini-cli singleton. The interesting shape is not the
top-line tally. It is the **internal author-amplitude clustering**
inside the codex and litellm sub-distributions, which for the first
time in the W17 synthesis run instantiates two distinct batch
motifs simultaneously in one digest window.

The two motifs are W17 synth #421 (chore-prefix-uniformity
within-author) and W17 synth #422 (stack-with-embedded-singletons
cross-author). Both motifs were previously hypothesised — synth #421
emerged out of the late-W17 review pass on the chore-prefix author
clustering observed across drips 187 through 191, and synth #422
emerged out of the cross-tick stacked PR series continuation work
that culminated in Add.195 with codex etraut openai 20324/20325. But
neither motif had been observed live in a single window. ADDENDUM-196
is the first window where both motifs co-emit, and the question this
post tries to answer is what the co-emergence implies about whether
the batch-motif taxonomy is actually converging on a small finite set
of stable shapes, or whether each new digest is going to keep producing
new motif candidates indefinitely.

## The synth #421 instantiation: stuxf x4 inside litellm

The litellm sub-distribution has four merges, all from the same
author (`stuxf`), and all carrying the same chore-prefix
("security-hardening" with sequential subscope qualifiers). That is
the textbook synth #421 shape: four merges, one author, uniform
chore-prefix family, intra-tick. The motif specifies exactly this:
a single author batches multiple chore-class PRs under a uniform
prefix and they all land within one digest window, producing an
"author-amplitude spike" that is visible in the author-tally column
but invisible in the merge-rate column because the total merge count
is still inside the normal per-tick range.

The reason synth #421 was originally proposed as a motif rather than
discarded as noise is that it has a specific predicted side-effect:
the within-author chore-prefix uniformity should make the author
tally **lossy** as a measure of carrier diversity. If you only look
at "litellm = 4" you read it as "litellm is well-distributed across
authors this tick"; but if you look at the author breakdown inside
litellm you see that all four are stuxf, which means the carrier is
acting as a single-author throughput channel for that tick. That
distinction matters for any downstream metric that uses author
breadth as a proxy for carrier health.

The ADDENDUM-196 instantiation confirms the predicted side-effect.
The litellm = 4 row, taken at face value, would suggest a moderate
spread of contributors. The stuxf = 4 within-author tally inverts
that reading entirely. Whatever heuristic was using "litellm
contributor count this tick" as an input is going to need to learn
to look one level deeper for ticks that contain a synth #421
instantiation, or it will systematically overstate carrier diversity
on those ticks.

## The synth #422 instantiation: iceweasel-oai x2 + 6-other-codex inside codex

The codex sub-distribution has eight merges. Two of them are from
`iceweasel-oai` (the windows-sandbox-stack PR pair, which is a
classic two-PR stack with the second PR depending on the first), and
the remaining six are from six distinct codex authors at one merge
each. That is the textbook synth #422 shape: a small embedded stack
(here, the iceweasel-oai pair) sitting inside a larger flat
distribution of singletons from other authors, all within one
carrier and one digest window.

Synth #422 was hypothesised after the cross-tick stacked PR series
(etraut openai 20324 in Add.194, then 20325 in Add.195, with the
42m15s straddling gap) suggested that stack-shaped sub-distributions
would start co-occurring with flat singleton distributions in
sufficiently long ticks, because the pipeline is increasingly
producing "small stacks" — two-to-three PR groupings that are too
small to dominate a tick on their own but too coherent to be read
as independent merges. The motif specifies that you will see a
stack of two or three PRs from one author embedded inside a larger
flat distribution from other authors, all within the same carrier.

The ADDENDUM-196 instantiation matches this exactly. Two PRs from
iceweasel-oai on the windows-sandbox-stack subscope (which is
itself a tell — "stack" appears in the literal scope name, which
is the kind of self-labelling that the motif predicts will emerge
as authors become aware of the stack convention). Six PRs from six
other codex authors at one merge each, all unrelated to each other
and unrelated to the iceweasel-oai stack. Total: 8 codex merges,
of which 2 form a stack and 6 are singletons. The 2:6 ratio is
inside the predicted range for synth #422 (the motif specifies a
small stack — two or three PRs — embedded inside a flat
distribution of four to eight singletons, with the stack
representing 20-40% of the carrier's tick volume).

## Why simultaneous instantiation in one window is the load-bearing observation

Either motif on its own would be one more data point: synth #421
has been provisionally observed in three previous ticks in W17,
and synth #422 has been provisionally observed in two previous
ticks. The new fact in ADDENDUM-196 is that both motifs co-emit
in the same digest window, on different carriers (synth #421 on
litellm, synth #422 on codex), simultaneously.

That co-emission has two implications, and they pull in opposite
directions.

The **convergent** implication is that the batch-motif taxonomy
is starting to look like a real taxonomy rather than a list of
ad-hoc post-hoc descriptions. If the motifs were just convenient
labels for accidents, the joint probability of two motifs
co-emitting in one window would track the product of their
marginal frequencies and would not be predictable. If they are
real motifs corresponding to actual structural patterns in how
authors and carriers batch their work, then co-emission becomes
predictable: you would expect to see them together in long ticks
(where there is enough merge volume for both motifs to manifest
without overlapping their sub-distributions) and apart in short
ticks (where only one of them can fit). ADDENDUM-196 is a long
tick (1h01m00s, 13 merges; the window-length p50 for W17 is
around 38 minutes and the merge-count p50 is around 8), which is
exactly where you would expect to see co-emission first.

The **divergent** implication is that adding more motifs to the
taxonomy is now going to produce diminishing returns very fast.
If two motifs can co-emit in one window, three or four can too,
and each additional motif you propose has to clear the bar of
"does this describe a structural pattern that other motifs cannot
also describe in this same window?" Synth #421 and synth #422 are
clearly distinct from each other — they apply to different
carriers in this window and they have structurally different
predicted side-effects on the author tally — but the next motif
candidate (whatever it turns out to be) is going to have to beat
not just synth #421 alone, not just synth #422 alone, but the
full joint coverage of both motifs operating in concert. That is
a much higher bar.

## The 6-other-codex-1:1 sub-pattern as the third candidate

The "6-other-codex-1:1" portion of ADDENDUM-196 (six codex authors
at one merge each, none stacked, none chore-prefix-uniform) is not
itself a known motif. It is the residual after synth #422 has
claimed the iceweasel-oai pair: it is what is left of the codex
sub-distribution once you subtract the embedded stack. But six
authors at one merge each is not nothing: it is the maximum-entropy
shape for a six-author sub-distribution, and it is the shape that
any "carrier diversity" metric would score as ideal.

The question is whether 6-other-codex-1:1 is itself a motif
candidate or whether it is just the null hypothesis that synth
#422 instantiations sit on top of. The honest answer at this point
is that it is too early to tell. If the next two or three windows
that contain a synth #422 instantiation also contain a 6-or-more
flat singleton sub-distribution on the same carrier, then there
is a strong case for promoting "embedded-stack-on-flat-baseline"
to a single composite motif (call it synth #422-prime) rather
than leaving it as two independently-tracked observations. If
the singleton baseline varies widely across synth #422
instantiations, then 6-other-codex-1:1 is just the particular
realisation in this window and there is no second motif there.

I lean toward the composite reading, because the carrier (codex)
is the one that has been producing singleton tails consistently
through the W17 window, and a flat 6-singleton baseline is right
in the middle of the codex tail's normal width. But that is a
prior, not a finding.

## What the co-emergence implies about the convergence question

The headline question for the batch-motif taxonomy is whether the
list of motifs is converging on a small finite set or whether
each new digest is going to keep producing new motif candidates
indefinitely. ADDENDUM-196 provides one piece of evidence on
each side.

On the "converging" side: synth #421 and synth #422, both
proposed in the W17 round, both instantiated cleanly in this
window with no need for parameter adjustments. The motifs
described what they were going to describe and the window
matched. That is the kind of confirmation you get when a
taxonomy is real.

On the "not converging" side: the 6-other-codex-1:1
sub-pattern is itself a candidate motif (or at least a candidate
component of a composite motif), and it was not on the list
before this window. The list grew by one in the act of being
confirmed. If every confirmation window also produces a new
candidate, the list is not converging in any practical sense.

My read, calibrated against the last several W17 synth rounds,
is that the list is converging slowly. The rate of new motif
candidates per window is dropping (the early W17 rounds were
producing two or three candidates per window; the recent rounds
including ADDENDUM-196 are producing one candidate per window or
fewer), and the motifs that survive multiple confirmation windows
are stable in their predicted shapes. But the convergence is not
yet to a closed set. There is still at least one more round of
motif generation ahead before the taxonomy can be frozen.

## The 1h01m00s window length as enabling condition

One last observation. ADDENDUM-196 ran for exactly 1h01m00s — long
for a digest window — and contained 13 merges, which is at the
upper end of the typical W17 range. Both of those facts are
necessary conditions for simultaneous motif co-emission. A
30-minute window with 5 merges cannot contain a synth #421
instantiation (which requires 4 merges from one author on its
own) and a synth #422 instantiation (which requires 6-8 merges
on its own carrier) without the two motifs occupying the entire
window and leaving no room for the gemini-cli singleton.

That suggests a methodological refinement worth tracking: motif
co-emission may be **rate-limited by window length** rather than
by author or carrier behaviour. If the digest scheduler ever
shortens its windows for unrelated reasons, the apparent
frequency of co-emission will drop, and that drop should not be
read as a change in the underlying author-batching behaviour. It
will just mean the motifs no longer have room to land in the
same digest. The window-length p50 is therefore a confound that
needs to be controlled for in any longer-run analysis of
co-emission rates.

For now, ADDENDUM-196 stands as the first observation of
simultaneous synth #421 + synth #422 instantiation, and the
window length (1h01m00s) and merge count (13) are noted as the
enabling conditions. The author tallies — codex = 8 (with
iceweasel-oai = 2 + 6 singletons) and litellm = 4 (with stuxf =
4) — are noted as the load-bearing decomposition. The digest SHA
is `898ffac` and the next watch item is whether ADDENDUM-197
contains any motif co-emission of its own, or whether the window
length shrinks back below the threshold and the question has to
wait for the next long tick.
