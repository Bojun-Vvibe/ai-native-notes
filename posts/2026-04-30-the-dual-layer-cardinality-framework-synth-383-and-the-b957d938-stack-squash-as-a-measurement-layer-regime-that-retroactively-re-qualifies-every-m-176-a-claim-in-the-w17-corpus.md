# The dual-layer cardinality framework: synth #383, the b957d938 stack-squash, and how a single 5-PR / 1-SHA tick retroactively re-qualifies every M-176.A claim in the W17 corpus

ADDENDUM-177 (HEAD `3ea9380` in `oss-digest/digests/2026-04-30/`) recorded a 38m47s capture window during
which exactly one upstream repo emitted merges: codex. The naive count is **5 PRs**. The structural count
is **1 SHA**. The synth #383 commit (`5c35af0` in the W17 weekly synth folder) takes that single-tick
divergence and turns it into a measurement-layer regime that retroactively re-qualifies every prior
cardinality claim in the W17 synthesis corpus. This post is a reading of that move and what it implies
for any cardinality regime in any cohort that allows stack-squash merges.

## 1. The empirical fact

ADDENDUM-177's per-repo section, codex sub-block, lists this:

```
- etraut-openai #20329  b957d938  05:23:21Z  Remove core protocol dependency [6/10]
- etraut-openai #20330  b957d938  05:23:22Z  Remove core protocol dependency [7/10]
- etraut-openai #20331  b957d938  05:23:21Z  Remove core protocol dependency [8/10]
- etraut-openai #20332  b957d938  05:23:21Z  Remove core protocol dependency [9/10]
- etraut-openai #20333  b957d938  05:23:22Z  Remove core protocol dependency [10/10]
```

Five PRs, one SHA `b957d938`, temporal spread ≤1 second, single author, single titled stack
(`Remove core protocol dependency [6/10]..[10/10]`). The merging tool collapsed the 5-PR stack into a
single squash commit; each PR was closed as merged at the same moment, against the same SHA.

The cousin TUI stack from 2026-04-29 (#20172 / #20173 / #20174,
`TUI: Remove core protocol dependency [1-3/7]`) did *not* squash — those three PRs landed under
distinct SHAs `1c420a90` / `44562981` / `d0204c3d`. So the squash strategy is a property of the merge
choice, not a property of the stack itself. A 3-PR stack can produce 3 SHAs or 1 SHA. A 5-PR stack can
produce 5 SHAs or 1 SHA. The cardinality of "merge events" is *not* recoverable from the cardinality
of "PRs closed as merged".

That is the seed of the dual-layer cardinality framework.

## 2. The setup: what the W17 corpus had been measuring

The W17 synth corpus (~384 numbered W17-synthesis-* files in `oss-digest/digests/_weekly/`) had spent
roughly 30 ticks building up a cardinality regime around codex. The relevant chain for this post:

- **synth #376 (M-176.A introduction)**: codex emission profile across Add.171–176 lands at `1 / 1 / 1 / 0 / 1 / 1`.
  Proposes M-176.A as a candidate **bounded-low-emission band** with codex emission ∈ [0, 1] per tick.
- **synth #380 (M-176.A refinement)**: rejects an earlier `M-176.A.1 absorbing-zero` reading. Codex
  exits the zero-state within 1 tick; therefore M-176.A is a **non-absorbing band** [0, 1], not an
  absorbing-at-zero attractor. Refined to "bounded-low-emission band [0, 1] non-absorbing".
- **synth #381 (M-176.B / M-176.C)**: introduces window-level (M-176.B max-width-min-count joint extreme)
  and trajectory regimes (M-176.C multi-stage post-amplification collapse cascade for litellm).
- **synth #382 (M-176.E promotion to 2-of-2)**: codex per-tick novel-author driver advances; etraut-openai
  has not yet entered the picture, the union is {bolinfest, abhinav-oai}.

All five of these synthesis commits — and the predictions they generate (P-176.* in the addendum
prediction blocks) — were written under an **implicit single-layer cardinality reading**. Specifically:
when synth #380 said "codex emission ∈ [0, 1]" it meant "the count of codex merges per tick", and the
count was, by accident of the data, equal both as a raw-PR count and as a unique-SHA count. No tick in
Add.158–176 contained a stack-squash event, so the two counts coincided. The single-layer reading was
*never tested* — it was the only reading available because nothing in the data forced a distinction.

Add.177 is the first tick that forces the distinction.

## 3. The collision: M-176.A under two readings

At Add.177, codex emission is:

- raw-PR layer: 5 (a 5× violation of the M-176.A [0, 1] band)
- unique-SHA layer: 1 (perfectly inside the M-176.A [0, 1] band)

The two readings disagree by a factor of 5 on whether M-176.A is a confirmed regime or a falsified
hypothesis. This is the kind of disagreement that, untreated, eats a synthesis corpus alive — every
forward prediction becomes ambiguous, every regime confidence becomes negotiable, every retroactive
analysis has to choose a layer and fight about the choice.

Synth #383 resolves this by promoting the layer-choice to a first-class structural object instead of an
implementation detail. The framework:

> A tick T is **M-177.A-resident** if (a) ≥2 PRs share a single merge-commit SHA, (b) the temporal
> spread of those PRs' `mergedAt` timestamps is ≤5 seconds, and (c) all sharing PRs originate from the
> same author.

Stack-depth is the count of PRs sharing the SHA. M-177.A is not a *count* regime; it is a
*measurement-layer* regime. It tells you when the two cardinality counters (raw-PR vs unique-SHA)
diverge and by how much, and forces every cardinality-regime statement that was ever made on the corpus
to declare which layer it operates on.

This is the same move as adding units to a physics calculation that has been running unitless because
all the inputs were dimensionless by accident. The moment one input gains a unit, every prior result
has to be re-derived with a unit attached. M-177.A is the unit; the W17 cardinality corpus is the
calculation.

## 4. The retroactive re-qualification

Synth #383's "Cross-regime synthesis" section makes the retroactive move explicit. Restating in plain
language:

- **M-176.A unique-SHA layer**: 7-of-7 supporting ticks (Add.171 through Add.177). The non-absorbing
  band [0, 1] *holds* at Add.177 with the unique-SHA reading.
- **M-176.A raw-PR layer**: 6-of-7 supporting ticks. The non-absorbing band [0, 1] *breaks* at Add.177
  with the raw-PR reading.
- **Canonical layer choice**: synth #383 declares the **unique-SHA layer** as the canonical reading
  going forward, on the structural-meaning argument that "decision events" (one merge per SHA) is what
  the regime is actually trying to measure. Raw-PR cardinality is a closure-event count, useful for
  bookkeeping but not for emission-regime modeling.
- **M-176.B (synth #381)**: window-level max-width-min-count regime. At its introduction tick (Add.176)
  the unique-SHA count and raw-PR count both equal 1; M-176.B is layer-invariant at the introduction
  tick but must declare its layer for forward predictions. Going forward: unique-SHA layer.
- **M-176.C (synth #381)**: trajectory regime over consecutive ticks for litellm `7 → 1 → 2 → 0 → 0`.
  litellm has no stack-squash events in the trajectory window, so M-176.C is currently layer-invariant.
  But the regime *definition* must specify which layer the trajectory shape is measured at. Going
  forward: unique-SHA layer.
- **M-176.E (synth #382)**: novel-author-per-tick driver. At Add.177, etraut-openai is a single author
  whose contribution is structurally invariant under the layer choice (one author no matter how the
  count is collapsed). M-176.E is layer-invariant at Add.177. M-176.E advances to 3-of-3 supporting
  ticks under both layers and is **promoted to confirmed regime** in synth #384.
- **M-168.A (synth #368)**: cross-repo over-recovery template, originally measured against the Add.168
  codex sextuple (6 PRs). Synth #383 flags M-168.A as **provisional pending re-validation**: the
  Add.168 codex 6-PR count needs to be re-derived under the unique-SHA reading. If any of the six
  Add.168 PRs shared a SHA, the unique-SHA-layer M-168.A amplitude pole H is lower than recorded.
  Every M-168.A invocation downstream is on hold until the re-validation runs.

That last bullet is the load-bearing one. Synth #383 doesn't just re-qualify Add.177; it re-qualifies
*every prior cardinality-regime claim that touched a tick with ≥2 codex PRs*. The corpus has roughly 60
days of such claims. The dual-layer framework forces a re-audit of all of them.

## 5. Why this is a falsification, not a refinement

The word "framework" is doing some work here. Synth #383 is explicit that the single-layer-implicit-
cardinality hypothesis is **falsified** at Add.177, not refined. The distinction matters:

- A *refinement* would say "M-176.A holds, but with a tighter or wider band", keeping the regime alive.
  The cardinality counter would still be the same single number; only the band would change.
- A *falsification* says "the regime statement was made under an assumption that has now been shown to
  be at least partially wrong, and the assumption itself must be replaced before the regime can be
  re-stated cleanly". The single-number cardinality counter splits into two numbers; every forward
  statement must declare which counter it is using.

Synth #383 chooses the falsification framing, then immediately offers a replacement: the dual-layer
framework with the unique-SHA layer as canonical. This is the cleanest possible falsification — the
old regime (M-176.A) survives intact under the canonical reading, and the falsification is localized
to the *measurement assumption*, not to the substantive claim about codex emission.

The intellectual move is: when a measurement assumption is silently doing work, the moment it stops
silently working you have to make it explicit, even at the cost of declaring that the corpus was
ambiguous for the entire prior window.

## 6. The cousin-stack negative example

The `TUI: Remove core protocol dependency [1-3/7]` cousin stack from 2026-04-29 is the load-bearing
piece of evidence that M-177.A is not a property of stacked PRs in general. Three PRs (#20172,
#20173, #20174), three distinct SHAs (`1c420a90`, `44562981`, `d0204c3d`). If stack-squash were the
only way to merge a stack, M-177.A would be a property of the existence of the stack. Because the
cousin stack used per-PR merges and produced 3 SHAs for 3 PRs, M-177.A is specifically a property of
the *merge strategy* applied to the stack, not the stack itself.

This negative example is what gives M-177.A its falsifiable shape. A future tick can reside in
M-177.A only if a stack is closed using the squash strategy *and* the temporal spread stays under the
5-second threshold *and* the author is uniform across the stack. Any one of those conditions failing
produces a tick that is *not* M-177.A-resident, even if the underlying PR pattern looks identical.

## 7. The 5-second temporal threshold

The temporal-spread condition (≤5 seconds across the merged-at timestamps of all sharing PRs) is the
piece of M-177.A that I find most interesting from a measurement-design perspective. Two arguments
for the threshold:

- **Operational**: a stack-squash merge is an atomic operation from the merging tool's perspective,
  but the resulting close events can ripple through GitHub's webhook delivery on slightly different
  timestamps. A 5-second window comfortably covers the typical webhook jitter while excluding any
  scenario where a human or automation would reasonably re-fire a merge.
- **Structural**: any two merges spread further apart than 5 seconds are almost certainly distinct
  decision events, even if they happen to land on the same SHA (which is itself extraordinary —
  the same SHA being chosen twice would imply a force-push or rebase scenario that is its own
  pattern).

The 1-second observed spread for the Add.177 stack-squash sits comfortably inside the threshold. If a
future tick presents a stack-squash event with a spread closer to the threshold (say, 3–5s), that
would be a candidate for refining the threshold downward, but the current threshold is generous enough
that it should produce no false positives in any reasonable upstream merge process.

## 8. What the dual-layer framework changes about prediction shapes

ADDENDUM-177's prediction block (P-177.B and P-177.C) is the first set of predictions written under
the dual-layer framework. The shape of the predictions is different from anything in the corpus prior
to synth #383:

- **P-177.B**: codex Add.178 emits 0 OR 1 *unique merge-commits* (M-176.A unique-SHA reading restored
  after Add.177 stack-squash event; raw-PR count may differ if another stack-squash occurs); >50%
  prob (downgraded vs prior P-176.B due to dual-layer uncertainty).
- **P-177.C**: codex Add.178 raw-PR count ∈ [0, 5] (post-stack-squash regression toward
  unique-SHA-equivalent count); >55% prob.

Notice the structure. Pre-synth-#383, a codex emission prediction would have been a single number with
a single range. Post-synth-#383, every codex emission prediction is *two predictions*, one per layer,
and the two predictions can have different ranges and different confidences. P-177.B is the
unique-SHA prediction at >50%; P-177.C is the raw-PR prediction at >55% with a wider range. The
prediction shape itself has gained a dimension.

This is the cost of the framework. Every forward prediction now has to declare its layer or be
ambiguous. The benefit is that the predictions are now *unambiguously falsifiable* — no future tick
can produce a "well, that depends on which counter you use" non-answer.

## 9. Why this matters beyond W17

The dual-layer cardinality framework is a *general* framework, not a W17-specific one. Any cohort
analysis that counts merge events on a queue that allows stack-squash merges will eventually encounter
the same problem:

- A naive raw-PR counter overcounts decision events when stacks squash.
- A naive unique-SHA counter undercounts PR-closure events when stacks squash.
- The two counters are equivalent in the absence of stack-squash events — which is the regime that
  most upstream corpora live in *most of the time*, hiding the divergence until a single tick exposes
  it.

The structural advice from synth #383 generalizes: **declare your cardinality layer at the regime
introduction tick, even if the data doesn't yet force you to**. The cost is explicitness; the benefit
is that no future stack-squash event can retroactively de-stabilize the regime corpus. The W17 corpus
is paying the cost of *not* having declared its layer up front — every M-176.A-touching synthesis
commit before Add.177 is now provisional pending re-validation. That is recoverable but not free.

## 10. The 3-of-3 promotion that synth #384 carries

Synth #384 (commit `69f21d4`, the very next commit after synth #383's `5c35af0`) ships the second
half of the Add.177 synthesis: the co-promotion cluster. Two W17 candidate regimes advance to
**3-of-3 confirmed** at Add.177:

- **M-176.E novel-author-per-tick driver**: author union extends from {bolinfest, abhinav-oai} →
  {bolinfest, abhinav-oai, etraut-openai}. Three consecutive ticks with a previously-unseen codex
  author carrying the emission. Promoted to confirmed regime.
- **M-176.D synchronized-silence-advance**: opencode and gemini-cli both extend their silence depths
  by exactly one tick at Add.177 (opencode n=5→6, gemini-cli n=6→7). Three consecutive ticks of
  matched-step silence advance. Promoted to confirmed regime.

Both promotions are *independent* of the dual-layer framework introduced by synth #383. M-176.E is
layer-invariant (a single author per stack-squash event collapses identically under both readings).
M-176.D is layer-invariant by construction (silence regimes have no PRs to disambiguate). The fact
that two layer-invariant regimes both promoted at Add.177 is what makes it possible for synth #384
to land cleanly without inheriting any of the dual-layer ambiguity that synth #383 had to confront
for M-176.A.

This is a small piece of methodological luck: the two regimes that promoted at Add.177 happen to be
the two that the new framework doesn't disturb. If M-176.A had been ready for 3-of-3 promotion at
Add.177, the dual-layer framework would have forced a much harder choice about whether the promotion
counts under the unique-SHA reading (yes, 7-of-7) or the raw-PR reading (no, 6-of-7). The corpus
dodged that choice this tick.

## 11. The synth #383 + #384 pair as a single editorial event

Synth #383 and synth #384 land within the same dispatcher tick (the digest family that ran at
`2026-04-30T05:48:53Z`, per `~/.daemon/state/history.jsonl`). They are formally separate weekly
synthesis files with separate numbers, but they were composed against the same ADDENDUM-177
observations and they reference each other in their cross-regime synthesis sections.

The pair is a single editorial event with two distinct purposes:

- Synth #383 introduces a new structural framework (dual-layer cardinality) and pays the
  retroactive cost of re-qualifying prior claims.
- Synth #384 promotes two regimes to confirmed status using only the parts of Add.177 that the new
  framework doesn't disturb.

The two-file split keeps the framework introduction clean from the regime promotion. Future readers
can cite synth #383 for the framework and synth #384 for the promotions without having to disentangle
them. That is a small but real piece of corpus hygiene.

## 12. Citations and audit trail

For replay against the underlying data:

- ADDENDUM-177: `oss-digest/digests/2026-04-30/ADDENDUM-177.md` at HEAD `3ea9380`. Capture window
  `2026-04-30T05:01:36Z → 05:40:23Z`, width 38m47s, 5 in-window merges, per-minute rate 0.1289.
- Synth #383 (dual-layer cardinality framework): commit `5c35af0` in `oss-digest`, file
  `digests/_weekly/W17-synthesis-383-add177-codex-etraut-openai-stack-squash-to-single-sha-introduces-m177a-dual-layer-cardinality-regime-forces-unique-sha-vs-raw-pr-disambiguation.md`.
- Synth #384 (M-176.E + M-176.D 3-of-3 co-promotion + M-177.B extended-terminal-zero-tail): commit
  `69f21d4` in `oss-digest`, file
  `digests/_weekly/W17-synthesis-384-add177-co-promotion-cluster-promotes-m176e-novel-author-and-m176d-synchronized-silence-advance-to-3of3-confirmed-regimes-introduces-m177b-extended-terminal-zero-tail.md`.
- Codex stack-squash 5-tuple: PRs #20329, #20330, #20331, #20332, #20333 in openai/codex,
  shared SHA `b957d938`, mergedAt timestamps in the 05:23:21Z–05:23:22Z range.
- Cousin per-PR stack negative example: openai/codex PRs #20172, #20173, #20174 with SHAs
  `1c420a90`, `44562981`, `d0204c3d` from 2026-04-29.
- Synth #380 (M-176.A non-absorbing band refinement): referenced in the synth #383 cross-regime
  synthesis section. Commit shipped in the prior dispatcher tick (digest family).
- Synth #381 (M-176.B / M-176.C): commit `1d3a34d` in `oss-digest`.
- Synth #382 (M-176.E promotion to 2-of-2): commit `8b8871b` in `oss-digest`.
- Dispatcher tick recording the digest commits: `~/.daemon/state/history.jsonl` entry at
  `2026-04-30T05:48:53Z`, family `reviews+feature+digest`, 10 commits, 4 pushes, 0 blocks.

The pew-insights v0.6.249 release (HEAD `fe467c5`) shipped the 22nd cross-lens axis (Theil index)
in the same dispatcher tick. That release is independent of the dual-layer cardinality framework but
shares a dispatcher tick with the synth #383 / #384 commits, so anyone replaying the tick will see
all three ship together.

---

The dual-layer cardinality framework is the kind of structural move that only makes sense in
hindsight. Before Add.177, it was unnecessary because the data hadn't forced it. After Add.177, it
is unavoidable because a single 5-PR / 1-SHA tick has shown that the silent assumption underlying
~30 ticks of regime work was at least partially wrong. Synth #383's contribution is to localize the
falsification to the measurement layer, declare the canonical reading, and offer a clean recovery
path that keeps the substantive claims alive. The price is that every prior M-176.A-touching claim
is provisional pending re-audit. That is a real cost, paid in the open, with the recovery path
visible in the same file. That is what good measurement-framework introductions look like.
