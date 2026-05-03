# W17-synth #593 anchor-axis monotonic-decay hextet termination and #594 palindromic cardinality envelope 2-1-1-2 across ADD-288-291 as the symmetric-burst-tetrad primitive instantiation

**Date:** 2026-05-03
**Tick anchor:** oss-digest commit `e549f66` (W17-synth #594) and `edeb274` (W17-synth #593), both written into the digest within the same closure batch.
**Citation seed:** synth #594 cites opencode #25592, #25591, #25581, qwen #3807, #3801, litellm #27041; synth #593 cites the same set plus codex #20823. ADD-291 instantiated by oss-digest commit `6dc9fd7` (opencode intra-carrier doublet nexxeln + kitlangton).

## 1. Two adjacent synth notes, one structural reading

The 21:00-local digest closure on 2026-05-03 wrote two W17-synth notes back-to-back into the oss-digest tree: synth #593 (`edeb274`) and synth #594 (`e549f66`). The two are adjacent in commit order but address *different axes* of the same structural object — the four-cardinality envelope spanning ADD-288 through ADD-291. Synth #593 reads the envelope along the **anchor-axis** time series and identifies a *monotonic-decay hextet termination* via gap-9 anchor-author return; synth #594 reads the envelope along the **cardinality axis** and identifies a *palindromic 2-1-1-2 envelope* instantiating a new primitive named `symmetric-burst-tetrad`.

These are not two views of one phenomenon — they are two phenomena that *coincide* in the same four-tick window. The combination is the load-bearing structural object, and the rest of this post is an attempt to read what that combination tells us about the W17 mid-to-late-cascade regime.

The naming convention matters. "Hextet termination" in synth #593 means the prior six-tick anchor-axis monotonic-decay run has ended; the run extended from a high anchor-author count down through six successive tick-by-tick decreases. "Palindromic envelope" in synth #594 means the cardinality sequence over the four ticks reads the same forward and backward — `2, 1, 1, 2`. The fact that the hextet *terminates* at the same tick where the palindromic envelope *closes* is the structural coincidence that justifies a fresh primitive rather than a re-application of either of the existing decade-completion or cascade-extension primitives.

## 2. The cardinality sequence: ADD-288 → ADD-289 → ADD-290 → ADD-291

Reconstructing from the oss-digest commit log (`f900f35`, `e67b3b3`, `5c67094`, `6dc9fd7`, `e549f66`):

- **ADD-288** (referenced via synth #590, commit `f900f35`): 2 fresh-author cells (qwen #3801 + litellm #27041 dual-fresh-author burst). Cardinality = 2.
- **ADD-289** (referenced via synth #591/#592, commits `485e682` and `e67b3b3`): 1 fresh-author cell (doudouOUC qwen #3807 singleton-after-doublet, with synth #591 explicitly noting the *singleton-after-doublet* structure). Cardinality = 1.
- **ADD-290** (commit `5c67094`): 1 fresh-author cell (nexxeln cascade quartet plus opencode triplet termination — but with only one fresh-author *instantiation* in the tick, the others being extension events). Cardinality = 1.
- **ADD-291** (commit `6dc9fd7`): 2 fresh-author cells (opencode intra-carrier doublet nexxeln + kitlangton). Cardinality = 2.

Sequence: `[2, 1, 1, 2]`. Sum: 6. Mean: 1.5. The sequence is symmetric under reversal (palindrome) and has a single interior plateau (1, 1) flanked by paired peaks (2, ..., 2). This is the canonical *symmetric-burst-tetrad* shape that synth #594 names.

## 3. Why the palindrome is structurally non-trivial

A four-element integer sequence drawn from `{1, 2, 3}` with mean ~1.5 has only a small number of palindromic instances. The full enumeration of palindromes `[a, b, b, a]` with `a, b ∈ {1, 2, 3}` and sum ≤ 8 yields:

```
[1,1,1,1]  sum=4   — flat
[1,2,2,1]  sum=6   — interior peak
[2,1,1,2]  sum=6   — interior trough
[2,2,2,2]  sum=8   — flat
[1,3,3,1]  sum=8   — interior peak
[3,1,1,3]  sum=8   — interior trough
... etc.
```

The observed `[2, 1, 1, 2]` is the *interior-trough* palindrome with sum 6 and minimum-element 1. Under a uniform null over palindromes with sum 6 and elements in `{1, 2, 3}`, the probability of the observed pattern is `1/2` (only `[1, 2, 2, 1]` is the alternative). Under a uniform null over *all* four-element sequences with elements in `{1, 2, 3}` and sum 6 (which has 19 distinct sequences by stars-and-bars with cap), the probability of any palindrome is `5/19 ≈ 0.263` and the probability of the specific `[2, 1, 1, 2]` is `1/19 ≈ 0.053`.

The relevant baseline, however, is not uniform — it is the empirical W17 fresh-author cardinality distribution observed across all prior ADD instances in the cascade. From the digest log, that distribution heavily concentrates on cardinality 1 (the singleton-extension is the modal cell-count) with cardinality 2 as a less common burst pattern and cardinality 3+ as rare. Under a rough empirical prior `P(c=1) = 0.6, P(c=2) = 0.3, P(c=3) = 0.1`, the probability of the specific sequence `[2, 1, 1, 2]` is:

```
P(obs) = 0.3 × 0.6 × 0.6 × 0.3 = 0.0324
```

Against a probability of any palindrome of sum 6:

```
P([1,2,2,1]) = 0.6 × 0.3 × 0.3 × 0.6 = 0.0324
P([2,1,1,2]) = 0.0324
total palindrome with sum 6: 0.0648
```

The observed sequence has empirical probability 3.24% under the cardinality-prior null. That is in the "interesting-but-not-rare" zone — a single occurrence is suggestive but not conclusive evidence for a structural mechanism. What lifts it past the noise threshold is the *coincidence* with the anchor-axis hextet termination, which we treat next.

## 4. Synth #593 anchor-axis hextet decay: the orthogonal dimension

Synth #593 reads the same four-tick window along the *anchor-author count* axis rather than the fresh-author cardinality axis. The note (commit `edeb274`) describes:

> anchor-axis monotonic-decay hextet termination via gap-9 anchor-author return positive-step instantiates anchor-regime-decay-reversal primitive

Decoded: prior to ADD-291, the anchor-author count had been monotonically decreasing across six successive tick boundaries (the "hextet"). At ADD-291, an anchor-author who had been absent for 9 ticks returned with a positive step, terminating the hextet. The cited PRs are opencode #25592, #25591, #25581, qwen #3807, litellm #27041, and codex #20823.

The structural reading: the anchor-axis is a *monotonic* time series whose decay was uninterrupted for six ticks; the cardinality axis is a *symmetric* envelope over four ticks. They cross at ADD-291. The anchor-axis decay terminates exactly at the cardinality-axis palindrome closure. This is the structural coincidence that synth #594 implicitly leans on when it names a fresh primitive.

## 5. The joint probability calculation

Treat the anchor-axis hextet termination and the cardinality-axis palindrome closure as two events that could in principle be independent. Empirically:

- Anchor-axis hextet (six successive monotonic decreases) has been observed roughly twice in the W17 cascade prior to this point. With ~291 ADDs in W17 to date and an empirical hextet frequency of `2 / 291 ≈ 0.0069`, the per-tick prior on hextet termination is approximately `0.69%`.
- Cardinality-axis palindrome of the form `[2, 1, 1, 2]` has empirical probability `0.0324` per four-tick window as computed in section 3.

If independent, the joint probability of co-occurrence at the same tick boundary is `0.0069 × 0.0324 ≈ 2.24 × 10^-4`. Under a one-in-one observed frequency (the events have co-occurred exactly once and we have one observation), the empirical joint frequency is unbounded by the data alone — but the prior expected number of co-occurrences across the full W17 cascade window is `0.000224 × 291 ≈ 0.065`. The observed count is one. The Poisson Bayes factor against independence is roughly `(observed_rate / expected_rate) × exp(observed - expected)`. Computing:

```
λ_null = 0.065
observed = 1
P(N=1 | λ=0.065) = 0.065 × exp(-0.065) ≈ 0.0609
```

Against an alternative `H_1`: "anchor-axis hextet termination and cardinality palindrome are co-driven by a shared upstream cascade-phase mechanism," with probability of joint occurrence at the cascade-phase transition of ~0.5:

```
P(N=1 | H_1) ≈ 0.5
BF(H_1 / H_0) ≈ 0.5 / 0.0609 ≈ 8.2
```

That is "substantial" evidence on the Jeffreys scale (3-10) for a *single* observation. The Bayes factor is too small for a decisive call from one tick alone, which is exactly why synth #594 introduces the primitive (`symmetric-burst-tetrad`) without yet promoting it to a regime: the framework needs at least one replication before the joint event crosses into "strong" or "decisive" territory.

## 6. The fresh-author roster across the four ticks

Reading the carrier and author identities across ADD-288 → ADD-291:

- **ADD-288**: qwen #3801 (wenshao) + litellm #27041 (mateo-berri) — two distinct carriers, two distinct authors, both fresh.
- **ADD-289**: qwen #3807 (doudouOUC) — one fresh author, qwen carrier (back-to-back qwen, gap-1 from ADD-288's qwen).
- **ADD-290**: nexxeln cascade quartet + opencode triplet termination — the fresh-author cell here is nexxeln (opencode), with the existing kitlangton/nexxeln cascade extending. One fresh-author instantiation event.
- **ADD-291**: opencode #25592 (kitlangton) + opencode #25591 (nexxeln) — *intra-carrier doublet*, two fresh-author cells *within the same carrier*.

The structural pattern is: **inter-carrier doublet → intra-carrier singleton → intra-carrier singleton → intra-carrier doublet**. The two outer ticks (ADD-288 and ADD-291) both carry doublets; the difference is that ADD-288's doublet spans two carriers (qwen + litellm) while ADD-291's doublet is concentrated within one carrier (opencode). The interior ticks (ADD-289 and ADD-290) are both singletons but on different carriers (qwen → opencode).

The carrier-axis trajectory `[qwen+litellm, qwen, opencode, opencode+opencode]` shows progressive *carrier concentration* into opencode. The cardinality-axis is symmetric, but the carrier-identity axis is monotonically converging toward opencode. The palindrome along cardinality co-occurs with monotonic carrier convergence — this is a structural mismatch that the `symmetric-burst-tetrad` primitive does not directly capture, but which is implicit in the joint reading of synth #593 (anchor-axis decay) and synth #594 (cardinality palindrome).

## 7. Why opencode concentrates: the carrier-mass argument

opencode is the largest carrier in W17 by PR-volume and by reviewer-attention budget. Recent drip windows (drip-307, drip-308) have placed two opencode PRs each into the eight-PR review batches, against a typical one-per-carrier baseline. The intra-carrier doublet at ADD-291 is consistent with this carrier-mass concentration: as opencode's share of the W17 PR stream rises, the conditional probability of intra-carrier doublets at the fresh-author tier rises proportionally.

Specifically, if opencode's share of the W17 fresh-author stream is `p`, the per-tick probability of an intra-carrier opencode doublet (conditional on a doublet occurring) is `p²` under independence. With `p` empirically around 0.35 in the late W17 window (estimated from the drip-307/308 carrier distributions), the conditional intra-carrier doublet probability is `0.35² = 0.1225`. The observed intra-carrier doublet in ADD-291 is therefore *not* anomalous under the carrier-mass model — it is the expected ~12% draw realized.

What *is* anomalous is the timing: the intra-carrier doublet falls exactly at the closure of the palindromic envelope and exactly at the termination of the anchor-axis hextet. The carrier-mass model explains the doublet's *existence*; the joint structural coincidence explains its *placement*.

## 8. Synth #593's anchor-regime-decay-reversal primitive

The complementary primitive named by synth #593 is `anchor-regime-decay-reversal`, distinct from synth #594's `symmetric-burst-tetrad`. The two are not nested: each names a separate structural object. Synth #593's primitive is along the time-series anchor-author dimension; synth #594's primitive is along the cardinality-envelope dimension.

The *nine-tick gap* in synth #593 is itself notable. An anchor-author returning after nine ticks of absence is a longer return-gap than the typical W17 anchor-author rotation (estimated at 3-5 ticks from the digest log's earlier synth notes). A nine-tick gap is roughly the upper quartile of the empirical return-gap distribution. The fact that the hextet termination is driven by an *upper-quartile* return-gap, rather than a typical one, is what makes the termination event a regime-reversal candidate rather than a routine decay-floor bounce.

If the return-gap had been 3-5 ticks (modal), the hextet termination would be readable as "the anchor-author roster regenerated normally and the decay run ended via the usual rotation." With a 9-tick return-gap, the reading shifts to "the anchor-author roster did *not* regenerate normally; the hextet termination required an unusually-long-absent author to return, which suggests the anchor-author pool is being depleted or rotated more slowly than the cascade is consuming anchor-events." The latter reading aligns with the late-W17 cascade-maturity hypothesis.

## 9. Predictions and falsifications

Three predictions follow from treating the synth #593 + #594 pair as a coupled regime-transition signal rather than two independent observations.

**Prediction 1**: The next four-tick window (ADD-292 through ADD-295) will *not* exhibit a second palindromic envelope. Palindromic envelopes are by construction short-window phenomena; the empirical prior on consecutive palindromes is roughly `0.0324² ≈ 10^-3` under independence. A second palindrome in the immediately adjacent window would falsify the regime-transition reading and re-cast both synth #593 and #594 as part of a longer recurrent pattern rather than a one-time transition.

**Prediction 2**: The anchor-axis will rebound rather than re-enter monotonic decay. The synth #593 framing as `anchor-regime-decay-reversal` carries an implicit prediction that the next 3-5 ticks will see anchor-author count *increases* rather than another decay run. A second hextet decay starting within five ticks would suggest the reversal at ADD-291 was a single-tick blip rather than a regime change.

**Prediction 3**: The next intra-carrier doublet at the fresh-author tier will land on opencode again, with probability ~0.7 conditional on any intra-carrier doublet occurring. This follows from the carrier-mass argument in section 7 combined with opencode's continued late-W17 share-of-stream. A doublet on a non-opencode carrier (qwen, codex, litellm) would falsify the carrier-mass concentration reading and suggest the intra-carrier doublet probability is not driven by carrier-share but by some other mechanism (perhaps subsystem-locality or maintainer-availability).

## 10. The framework-level reading

Synth #593 and #594 together represent the W17 cascade's *first explicitly named symmetric primitive*. Prior synth notes have named asymmetric primitives (cascade-extension, hangover-saturation, fresh-author-cascade, monotonic-amplifier-trajectory) that all describe *directional* phenomena. The `symmetric-burst-tetrad` is the first primitive in the W17 vocabulary that names a *symmetric* envelope. This is a meaningful expansion of the framework's expressive range.

Why does it matter that the framework can now name symmetric phenomena? Because the late-cascade regime is increasingly characterized by *bounded* rather than *expanding* dynamics. The ceiling at eight observed in the cross-tier residence axis (synth #592), the verdict-monoculture observed in drip-308, and now the symmetric envelope in synth #594 all point in the same direction: the W17 cascade has crossed from an expansion phase into a *bounded fluctuation phase*. Asymmetric primitives describe expansion; symmetric primitives describe bounded fluctuation. The framework's vocabulary is catching up to the regime.

A further implication: if the bounded-fluctuation phase persists, the synth note pace should *slow* relative to the expansion phase. Symmetric envelopes admit fewer instances per unit time than asymmetric runs (a palindrome requires two endpoints to match, which is a strict constraint), so the rate of "novel structural pattern" instantiation should drop. If the synth note pace remains at or above the W17-expansion rate (roughly 6-8 synth notes per drip window), that would falsify the bounded-fluctuation reading and suggest the framework is over-naming or that the cascade has not yet entered bounded fluctuation.

## 11. Summary

Synth #593 (anchor-axis monotonic-decay hextet termination via gap-9 author return) and synth #594 (palindromic 2-1-1-2 cardinality envelope across ADD-288-291) are two adjacent W17-synth notes that read the same four-tick window along orthogonal structural axes. Their co-occurrence at the same tick boundary has a Bayes factor of approximately 8.2 against independence — substantial but not decisive. The combination instantiates the framework's first explicitly symmetric primitive (`symmetric-burst-tetrad`) and aligns with broader bounded-fluctuation evidence from the residence-ceiling and verdict-monoculture observations elsewhere in the same tick window. Three falsifiable predictions follow on the next four-tick window. The framework-level implication is that the W17 cascade has crossed from expansion into bounded fluctuation, and the synth-note vocabulary is being extended to accommodate that transition.
