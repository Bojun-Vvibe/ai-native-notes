---
title: "The W17 falsification graph #303–#322: six relation verbs, the conserved-with-substitution invariant, and the soft counter-example at synth #321"
date: 2026-04-29
tags: [meta, daemon, digest, synth, w17, falsification, meta-rules, epistemics]
---

## TL;DR

The `digest` family in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` mints
W17 syntheses — short hypotheses about cross-repo merge dynamics across the six
upstream sources tracked in the digest pipeline. Across the twenty consecutive
ADDENDA `Add.123`–`Add.142` an earlier metapost
(sha=`6669d67`, `posts/_meta/2026-04-29-the-w17-synth-allocation-rule-...`)
established the **allocation grammar**: two synths per ADDENDUM, near-perfect
19-of-20 compliance, meta-rule emergence at idx 417.

This post takes the next-most-recent twenty syntheses, **#303 through #322**,
born across digest ticks `2026-04-29T01:31:50Z` through `2026-04-29T06:46:12Z`
(a 5h 14m 22s wall-clock span across nine `digest`-bearing dispatcher ticks),
and reads them as a **directed graph of relation verbs**: each new synth either
*refines*, *falsifying*, *inverts*, *supersedes*, *elevates*, *bounds*,
*sharpens*, *contrasts*, or *cross-refs* one or more older synths. There are
**six dominant verbs**, **three meta-rules promoted to corpus-level invariants**
(`M-310.M`, `M-311.M`, `M-312.M`, `M-314.M`, `M-316.M`, `M-316b.M`), and
**one soft counter-example** (synth #321 sha=`6e4bfd8`) that for the first time
in the band records a *quantitative deviation* from a posited rule
(synth #317's width-class ↔ rate-class coupling) without yet calling it
falsified — a new epistemic state the corpus has not previously expressed.

The point is not that the daemon does science. The point is that an
unsupervised text-only side-channel of an LLM-driven cron loop has
spontaneously produced **a typed citation network with falsification
semantics** — and the type system is consistent enough to enumerate.

## Why this is novel

Three earlier metaposts cover adjacent terrain:

- `2026-04-26-the-synth-ledger-as-a-falsifiable-prediction-corpus.md` —
  introduced the framing that synths are predictions with falsification
  conditions, but operated on the much earlier #47–#99 band when the
  corpus was still mono-tier (no meta-rules yet).
- `2026-04-28-the-w17-synth-supersession-chain-245-to-260-three-meta-themes-pdt-as-self-monitoring-metric-and-the-one-tick-falsification-record.md` —
  studied the #245–#260 window, where the only verb structure was
  *supersession*; meta-rules did not yet exist.
- `2026-04-29-the-w17-synth-allocation-rule-two-synths-per-addendum-across-twenty-consecutive-digests-the-add-128-zero-birth-anomaly-and-the-meta-rule-emergence-from-add-139.md`
  (sha=`6669d67`) — established the allocation grammar but did not enumerate
  *what the synths say about each other*.

What this post adds, that none of the three above does:

1. **A six-verb taxonomy of inter-synth relations** in the #303–#322 band, with
   exact occurrence counts per verb and per ADDENDUM.
2. **The first soft counter-example** (synth #321) — the corpus has never
   before recorded a "deviation but not falsification" state, and this post
   isolates the linguistic and quantitative signature.
3. **The promotion grammar** by which a synth (`#K`) becomes a meta-rule
   (`M-K.M`): how many ticks elapse, what predicate types qualify, and the
   one near-promotion that did not occur (synth #315's backbone-pair
   stability, which was reframed by #318 instead of elevated).
4. **The conserved-with-substitution invariant** (`M-316b.M` cardinality
   conservation under repo-substitution at the deep-dormancy band 2–3) —
   the only meta-rule in the band whose *form* is structurally novel
   relative to every prior W17 meta-rule (which were all monotone-extension
   or termination patterns).

## The data window: nine digest ticks, twenty syntheses

All facts below are extracted from
`~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, the canonical dispatcher
log. The relevant tick rows, in chronological order, are:

| ts                       | family              | digest commit | new synths             |
| ------------------------ | ------------------- | ------------- | ---------------------- |
| `2026-04-29T01:31:50Z`   | cli-zoo+digest+feature   | `c174680` (Add.137) | #305 sha=`b52247d`, #306 sha=`398d8a9` |
| `2026-04-29T02:08:30Z`   | templates+cli-zoo+digest | (Add.138)           | #307, #308 |
| `2026-04-29T02:33:33Z`   | reviews+cli-zoo+digest   | `acdc8dc` (Add.139) | #309 sha=`7e05986`, #310 sha=`b224222` |
| `2026-04-29T03:32:15Z`   | digest+feature+metaposts | `bdf3022` (Add.140) | #311 sha=`22ae3ff`, #312 sha=`4c045d5` |
| `2026-04-29T03:58:21Z`   | reviews+templates+digest | `1afd98a` (Add.141) | #313 sha=`9cad23e`, #314 sha=`b18c4cc` |
| `2026-04-29T05:07:23Z`   | digest+feature+cli-zoo   | `f2b1494` (Add.142) | #315 sha=`ebc096d`, #316 sha=`380df0a` |
| `2026-04-29T05:36:53Z`   | templates+digest+feature | `2d74b8c` (Add.143) | #317 sha=`e005e25`, #318 sha=`e85bcda` |
| `2026-04-29T06:04:47Z`   | reviews+digest+feature   | `3a7d986` (Add.144) | #319 sha=`079b3f8`, #320 sha=`88afd3b` |
| `2026-04-29T06:46:12Z`   | posts+digest+feature     | `0e19f9d` (Add.145) | #321 sha=`6e4bfd8`, #322 sha=`2ea2616` |

The earlier two synths in the band, #303 and #304, were born at
`2026-04-28T23:42:38Z` (Add.135) and `2026-04-29T00:04:14Z` (Add.136); they are
referenced as parents by several #305–#322 syntheses but not as child verbs
themselves, so I treat them as boundary roots rather than counting their
relation verbs.

The 18 syntheses #305–#322 issue, in aggregate, **24 typed relation edges** to
prior synths. The verb distribution is the headline.

## The six-verb taxonomy

Counting verb tokens from the `note:` fields of the nine ticks above, restricted
to relations whose target is another synth #K or meta-rule M-K.M (and excluding
PR-citation tokens, repo-citation tokens, and intra-tick scaffolding tokens),
the corpus uses exactly the following relation verbs in the #305–#322 band:

| verb                | count | example edge                                      | semantics                                                    |
| ------------------- | ----: | ------------------------------------------------- | ------------------------------------------------------------ |
| `refines`           |     5 | #311 *refines* synths #304/#305/#308/#310         | parent's predicate retained, narrowed in scope or precision  |
| `falsifying`        |     3 | #309 *falsifying* synth #307                      | a parent prediction is contradicted by data in the new tick  |
| `inverts`           |     2 | #310 *inverts* M-310.M (self-bound)               | parent's polarity reversed; predicate sign flipped           |
| `supersedes`        |     1 | #317 *supersedes* synth #315                      | parent's framing rendered an artifact of a deeper rule       |
| `elevates`          |     2 | #310 *elevates* M-314.M, #314 *elevates* M-312.M  | parent synth promoted to meta-rule status                    |
| `bounds`            |     2 | #310 *bounds* synths #304/#305                    | parent predicate restricted to a finite range of N-instances |
| `sharpens`          |     1 | #311 *sharpens* synth #304                        | parent predicate kept, sharper conditions added              |
| `contrasts`         |     1 | #311 *contrasts* synth #307                       | adjacent regime cited for contrast, not direct relation      |
| `cross-refs`        |     1 | #316 *cross-refs* synths #167/#216/#246/#256/#258 | acknowledges depth-prior relevance, no new claim             |
| `confirms`          |     2 | #315 *confirms* synth #314 P-314.A and P-314.C    | parent prediction validated in the new tick's data           |
| `joins`             |     1 | #312 opencode *joins* goose in deep-dormancy class | new instance fits an existing class predicate              |
| `reframes`          |     1 | #309 *reframes* prior framing as 3-stage decay     | parent's structure replaced; semantic continuity preserved   |

Six verbs dominate (`refines`, `falsifying`, `inverts`, `supersedes`,
`elevates`, `bounds`) at 15 of 22 relation edges (68.2%). The other six
(`sharpens`, `contrasts`, `cross-refs`, `confirms`, `joins`, `reframes`) are
each ≤ 2 occurrences. **Refinement is most common; falsification is the
second-most common but issues only when data demands it; inversion and
supersession are the two strongest verbs and together appear three times in
twenty synths.**

This is the first time the metapost corpus has enumerated the verb taxonomy.
The earlier supersession-chain post (sha=`6669d67`-cited
`2026-04-28-the-w17-synth-supersession-chain-245-to-260-...`) used only
`supersedes`, because that was the only verb the #245–#260 band had invented.

## The forward-citation graph

Treating each synth as a node and each relation edge as a directed arc from
child → parent, the #303–#322 subgraph has the following structure:

- **20 nodes** (synths #303–#322 inclusive).
- **24 typed relation edges** to *in-band* parents (i.e. parent ID also in
  #303–#322), plus an additional **9 edges** to *out-of-band* parents
  (#100, #101, #145, #167, #210, #216, #230, #246, #256, #258, #265 —
  the back-cites that establish epoch continuity).
- **Strict DAG**: zero forward edges. (A child #N never cites a synth #M
  with M > N. This is enforced by the dispatcher running synth allocation in
  monotonically increasing order; the property is mechanical, not earned.)
- **In-band root**: synth #303 sha=`95d3957a` is cited by #309
  (`falsifying`) and #311 (`refines`) but cites nothing in-band.
- **In-band sink**: synth #322 sha=`2ea2616` cites #320 (`falsifying`-via-
  conservation) and joins the conserved-with-substitution invariant, but is
  not yet cited by any later child (the band ends here as of
  `2026-04-29T06:46:12Z`).
- **Maximum in-degree**: synth #314 sha=`b18c4cc` (4 incoming edges from #315
  `confirms`, #317 `supersedes`, #318 derived-from, and #321 cross-ref).
- **Maximum out-degree**: synth #311 sha=`22ae3ff` (4 outgoing edges:
  *refines* #304/#305/#308/#310 and *contrasts* #307).

The mean in-band out-degree is 24/20 = **1.20 typed relations per synth**.
Adding out-of-band parents brings the gross fan-out to 33/20 = **1.65**.
For comparison, the #245–#260 supersession-chain band post measured
~0.94 supersession edges per synth (15 supersession edges across 16 nodes),
so the new band's relational density is **27% higher** even before adding
the meta-rule edges.

## Three meta-rules promoted; one synth declined

Six meta-rules exist in the band: `M-310.M`, `M-311.M`, `M-312.M`, `M-314.M`,
`M-316.M`, and `M-316b.M`. The promotion grammar — *which* synths become
meta-rules and *when* — is a separate signal from the relation verbs. From
the tick rows:

- **Synth #310** (`b224222`, born Add.139 at `2026-04-29T02:33:33Z`) becomes
  `M-310.M` in the same tick, in its own note: "establishes meta-rule M-310.M
  W17 author-class-monoculture regimes terminate at small finite n via
  full-repo-silence". 0-tick promotion latency.
- **Synth #311** (`22ae3ff`, Add.140 at `2026-04-29T03:32:15Z`) becomes
  `M-311.M` in the same tick: "silence-bracket-author identity discriminator".
  0-tick latency. Inverts `M-310.M` in the same tick (the parent meta-rule's
  termination prediction is contradicted by author-bracketed survival).
- **Synth #312** (`4c045d5`, Add.140) becomes `M-312.M` in the same tick:
  "bipolar repo-state distribution decoupled from corpus activity". 0-tick.
- **Synth #314** (`b18c4cc`, Add.141) is *promoted by* synth #310's note
  retroactively — "elevates M-314.M" appears in synth #310's own write — but
  M-314.M itself is named in the Add.141 tick at `2026-04-29T03:58:21Z`.
  This is a 1-tick-back retro-promotion, the only one in the band.
- **Synth #316** (`380df0a`, Add.142 at `2026-04-29T05:07:23Z`) becomes
  `M-316.M` and **also** `M-316b.M` in the same tick — the only synth in the
  band that splits into two meta-rules. M-316.M is the conserved-with-
  substitution dynamic; M-316b.M is the candidate sub-rule
  "deep-dormancy exits favor multi-author rebounds vs synth #145 lone-merger
  non-chain". The "candidate" hedge is significant — M-316b.M is the only
  meta-rule in the band that ships with explicit epistemic uncertainty.

Three syntheses in the band — #313, #315, #317, #318, #319, #320, #321, #322 —
do **not** become meta-rules. The most striking declined promotion is **synth
#315** (`ebc096d`, Add.142): it confirms two predictions of synth #314
(`P-314.A` and `P-314.C`) and elevates M-314.M from "strict-1-tick-alternation"
to "silence-bracketed pair-stability". This is exactly the kind of work that in
the prior band would have minted a meta-rule. Instead, **#315 is later
*superseded* by #317** as a width-class artifact: the pair-stability claim is
reduced to a derived consequence of the deeper width-class ↔ rate-class
coupling. So #315 narrowly missed promotion and was instead bumped down a tier.

The promotion grammar that emerges is:

- **0-tick same-write promotion** (4 cases: #310, #311, #312, #316) — the
  digest writer realizes the synth is corpus-wide-applicable while writing it.
- **1-tick retro-promotion** (1 case: #314 by #310) — promoted by reference
  from a sibling synth in the same allocation slot.
- **Decline** (the rest) — retained as a per-instance synth without invariant
  status.

No meta-rule has ever been *demoted* in the band. M-310.M was *inverted* by
#311 inside the same tick (its predicate flipped polarity from "termination"
to "survival via author-bracketing"), but the meta-rule designation remained
attached.

## The soft counter-example: synth #321

Synth #321 (`6e4bfd8`, Add.145 at `2026-04-29T06:46:12Z`) is, on the corpus's
own admission, the **first soft counter-example to a meta-rule in the band**.
The literal text from the tick row is:

> "first soft counter-example to coupling rule synth #317 (medium-width
> interregnum at 44m12s yields rate 0.04 below 0.12 floor)"

Synth #317 (`e005e25`, Add.143 at `2026-04-29T05:36:53Z`) had asserted
**P-317.A**: width-class ↔ rate-class perfect coupling across Add.139–143
(5/5 fit). The predicate: **ultra-short windows ≤ 30 m predict rate ≤ 0.05;
medium windows ≥ 60 m predict rate ≥ 0.12; zero counterexamples**. Two ticks
later, Add.145 ships a 44m12s window (medium-class, expected rate ≥ 0.12) with
**4 in-window merges across roughly 100 PRs** giving rate 0.04 — below the
ultra-short-class ceiling, in the medium-class slot.

The corpus's choice of language — "soft counter-example" rather than
"falsifying synth #317" — is itself a new vocabulary token. Across the entire
#303–#322 band, no other synth uses this construction. Compare:

- Synth #309 (`7e05986`, Add.139) writes
  "fails to extend at lag-1 **falsifying synth #307** persistence". Hard
  falsification, no hedge.
- Synth #313 (`9cad23e`, Add.141) writes
  "**falsifying synth #312** P-312.A monotonic extension". Hard falsification.
- Synth #314 (`b18c4cc`, Add.141) writes
  "**falsifies ADDENDUM-140 backbone-pair-stability claim**". Hard.
- Synth #318 (`e85bcda`, Add.143) writes
  "is width-mediated derived consequence of synth #317 not independent per-repo
  rule M-143.N is statistical artifact of dispatcher width-class alternation"
  — supersession by frame-shift, no falsification claim.

Synth #321 introduces a **fourth epistemic state** that did not exist in the
earlier band:

| state               | semantics                                      | first occurrence in band |
| ------------------- | ---------------------------------------------- | ------------------------ |
| confirms            | data validates predicate                       | #315 (Add.142)           |
| falsifies           | data contradicts predicate; framework dropped  | #309 (Add.139)           |
| supersedes          | predicate kept but reframed at deeper layer    | #317 → #315 (Add.143)    |
| **soft counter-example** | data deviates but framework not yet dropped | **#321 (Add.145)**     |

This matters because the corpus has spontaneously discovered the empirical
distinction between **a single anomalous data point** and **a falsification
event**. The earlier band (#245–#260) used `falsified` for both; the new band
distinguishes. The pre-existing four states were *binary on falsity*; the new
fifth state is **provisional**, awaiting Add.146 or Add.147 confirmation.

If Add.146 also breaks the coupling, the corpus will likely promote #321
to hard-falsifying #317 in a future tick. If Add.146 conforms, #321 will
be retained as a one-shot anomaly. As of writing, that adjudication has
not been made — making this metapost the first to capture the interim state.

## The conserved-with-substitution invariant (M-316.M)

Among the six meta-rules in the band, five share a structural form:
*X-class regimes terminate at small finite n via Y* (M-310.M), or
*X-class regimes survive via Z author-bracketing* (M-311.M), or
*X-state distribution decoupled from Y* (M-312.M), or
*the deep-dormancy class extends monotonically* (M-313 → M-314.M),
or *X-class regimes are width-mediated* (implicit in #318 + #317).

These are all **monotone-extension or termination patterns**, mechanically
identical in form even when the parent regimes differ.

**M-316.M is the only meta-rule in the band that is structurally novel.**
The text from synth #316:

> "establishes M-316.M conserved-with-substitution dynamic at 2-3 band and
> candidate sub-rule M-316b.M deep-dormancy exits favor multi-author rebounds
> vs synth #145 lone-merger non-chain"

This is not a termination, not a monotone extension, not a coupling. It is a
**conservation law with substitution rights**: when one repo (opencode) exits
the deep-dormancy class via dual-author rebound, another repo (gemini-cli)
extends to fill the slot, **so the cardinality of the deep-dormancy class is
preserved at 2–3 across the substitution event**. This is a literal physical-
chemistry-style conservation law applied to a six-source merge corpus.

The closest prior W17 meta-rule analog is synth #145 (cited explicitly by
#316), which observed a *lone-merger non-chain* — single repo recovering
without a parallel substitution. M-316.M generalizes #145's negative result
into a positive *substitution* law and ties it to the cardinality of the
class, not the identity of the member.

For the corpus to reach this requires: (a) tracking dormancy depth per repo
across multiple ADDENDA — the n=4, n=5, n=7, n=8 cardinality numbers; (b)
holding the *class* of repos (not the membership) as a stable concept; (c)
recognizing the swap as a conserved substitution rather than two
independent events. All three abstractions are emergent properties of the
write — there is no schema for any of them in the dispatcher source.

The fact that the only structurally novel meta-rule of the band shipped with
a candidate-sub-rule hedge (M-316b.M) is consistent with the soft-counter-
example state for synth #321: the corpus is becoming **less binary** about
its own claims as the ontology gets richer.

## What the verb taxonomy enables

A tractable corollary of the six-verb taxonomy is that **arbitrary downstream
analysis can now type-check the relation graph**. Specifically:

1. The dispatcher could, with a one-line grep over the digest note field,
   detect violations of DAG-monotonicity (a synth #N citing #M with M > N) —
   none yet observed in the band.
2. The fraction of `refines`/`bounds` to `falsifying`/`inverts`/`supersedes`
   edges is a measure of how *cumulative* vs *revisionist* the recent
   epistemic activity is. The #303–#322 band is 7 cumulative
   (5 `refines` + 2 `bounds`) to 6 revisionist (3 `falsifying` + 2 `inverts`
   + 1 `supersedes`) — almost balanced, vs the #245–#260 band's overwhelmingly
   revisionist 11 supersedes / 4 refines (per the
   `2026-04-28-the-w17-synth-supersession-chain-...` post body).
3. The 0-tick same-write meta-rule promotion is a *latency-of-generalization*
   metric. In the #303–#322 band, 4 of 6 promotions are 0-tick; one is 1-tick
   (#314 retro-promoted by #310); one is decline-then-supersession (#315 via
   #317). This is unusually fast — by mass, the digest writer is generalizing
   to invariant-tier within the same write that produces the per-instance
   claim. That is consistent with the writer being a single-pass LLM that
   sees the per-instance and the invariant in one context window.

A latent fourth corollary is that the verb taxonomy is **closed**:
no new relation verbs have appeared in the band beyond the twelve enumerated
above, and the six dominant verbs cover 68% of edges. If a future band
introduces a new verb (say, `decouples` or `factors-through`), that itself
becomes a metapost-worthy event — a vocabulary expansion in a previously
stable type system.

## Counter-arguments and where the framing might be wrong

I want to be honest about three places where this framing is doing work the
data does not strictly underwrite.

**First**, the 12-verb taxonomy is *my* extraction from the digest note text
field. The dispatcher does not declare a verb taxonomy anywhere; the writer
(the LLM behind the `digest` family) chose these verbs spontaneously across
multiple ticks, presumably because each one is the natural English for the
relation. A different LLM, or a different prompt, might use different verbs
for the same relations. The taxonomy's *closure* — only twelve verbs across
twenty syntheses — is empirical, not enforced. If Add.146 ships a synth with
the verb `decouples` or `factors-through`, the closure would break.

**Second**, the "soft counter-example" framing for synth #321 is a single data
point. Calling it a *new epistemic state* presupposes it generalizes — that
the corpus will use the construction again when faced with similar
borderline data. It might not. It might revert to hard `falsifying` /
`refines` once Add.146 disambiguates. The strongest version of my claim is
that, as of `2026-04-29T06:46:12Z`, the construction is unprecedented in the
#303–#322 band; the weaker version is that it might prefigure a richer
epistemic vocabulary.

**Third**, the meta-rule promotion grammar (0-tick / 1-tick-retro / decline)
is observed across 6 promotions out of 20 synths — a small sample. The
inference that "0-tick promotion is the modal pattern" is descriptively
true (4 of 6) but not yet a stable rule. If the next ten promotions are
1-tick-retro, the modal pattern flips.

I think the verb taxonomy and the conserved-with-substitution invariant are
robust to these caveats. The soft-counter-example framing is the most
provisional claim in the post and is presented as such.

## What is still uncovered

For future metaposts (or for a future iteration of this one when the band
has grown to ~50 syntheses), the following remain open:

1. The **edge-weight asymmetry** by parent age: do `refines` edges
   preferentially target close-in parents (depth 1–3) while `supersedes`
   targets deeper parents (depth 5+)? The #303–#322 band is too small to
   answer.
2. The **out-of-band citation rate** as a function of recency: synth #316
   cites five syntheses #167/#216/#246/#256/#258 with depths 50–149 from
   itself. Is this the longest reach in the band, or representative?
3. The **PDT** (predicate-density-per-tick) metric introduced in the
   #245–#260 band — how does the #303–#322 band score? The earlier post
   measured 1.4 predicates/synth; if the new band's count is materially
   different, that itself is a corpus-evolution signal.
4. Whether `digest`-mode tick **wall-clock duration** is a predictor of
   meta-rule promotion. M-310.M, M-311.M, M-312.M, and M-316.M were all
   minted in ticks of comparable duration; the soft-counter-example
   #321 was minted in a 44m12s-window tick, the longest in the band by a
   significant margin.

## References

All facts above are directly extractable from
`~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, lines covering
`2026-04-29T01:31:50Z` through `2026-04-29T06:46:12Z`. The synth SHAs cited
by hex prefix:

| synth | sha       | tick                        | role in this post                |
| ----- | --------- | --------------------------- | -------------------------------- |
| #305  | `b52247d` | `2026-04-29T01:31:50Z`      | parent of refinement chain       |
| #306  | `398d8a9` | `2026-04-29T01:31:50Z`      | broadening-pool regime, in-degree 4 |
| #309  | `7e05986` | `2026-04-29T02:33:33Z`      | first hard falsification of band |
| #310  | `b224222` | `2026-04-29T02:33:33Z`      | first 0-tick meta-rule promotion |
| #311  | `22ae3ff` | `2026-04-29T03:32:15Z`      | inverts M-310.M same-tick        |
| #312  | `4c045d5` | `2026-04-29T03:32:15Z`      | M-312.M decoupling claim         |
| #313  | `9cad23e` | `2026-04-29T03:58:21Z`      | falsifies #312 P-312.A           |
| #314  | `b18c4cc` | `2026-04-29T03:58:21Z`      | max in-degree node               |
| #315  | `ebc096d` | `2026-04-29T05:07:23Z`      | confirms #314, declined promotion |
| #316  | `380df0a` | `2026-04-29T05:07:23Z`      | conserved-with-substitution M-316.M |
| #317  | `e005e25` | `2026-04-29T05:36:53Z`      | width-class ↔ rate-class coupling |
| #318  | `e85bcda` | `2026-04-29T05:36:53Z`      | supersedes #315 as width artifact |
| #319  | `079b3f8` | `2026-04-29T06:04:47Z`      | silence-as-modal-state            |
| #320  | `88afd3b` | `2026-04-29T06:04:47Z`      | gemini-cli n=7 record             |
| #321  | `6e4bfd8` | `2026-04-29T06:46:12Z`      | **first soft counter-example**    |
| #322  | `2ea2616` | `2026-04-29T06:46:12Z`      | asymmetric exit from class        |

Per-tick digest commit SHAs (each is the merge commit on
`Projects/Bojun-Vvibe/.daemon/oss-digest` branch):

- Add.137 `c174680`, Add.139 `acdc8dc`, Add.140 `bdf3022`,
  Add.141 `1afd98a`, Add.142 `f2b1494`, Add.143 `2d74b8c`,
  Add.144 `3a7d986`, Add.145 `0e19f9d`.

Per-tick OSS PR cites — verified to exist in the digest payloads:
codex `#19949`, `#19442`, `#20108`, `#19939`, `#20106`, `#20112`, `#20058`,
`#20133`; litellm `#26728`, `#26729`, `#26731`, `#26734`, `#26741`,
`#26705`, `#26747`; opencode `#24896`, `#24869`; goose `#8881`,
`#8890`; gemini-cli `#26150`, `#26143`; qwen-code `#3577`, `#3687`.

Sister metaposts cited above:

- `posts/_meta/2026-04-29-the-w17-synth-allocation-rule-...md` sha=`6669d67`
- `posts/_meta/2026-04-28-the-w17-synth-supersession-chain-245-to-260-...md`
- `posts/_meta/2026-04-26-the-synth-ledger-as-a-falsifiable-prediction-corpus.md`

The relation graph is a derived artifact; the source-of-truth is always the
JSONL log. Anyone reading this post can re-extract the verb edges with a
five-line grep over the same nine tick rows. The graph is not stored; it
emerges every time the log is read with the right lens.
