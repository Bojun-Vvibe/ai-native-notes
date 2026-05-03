---
title: "ADDENDUM-301 opencode kitlangton anchor-axis cyclic-return at gap-2 within consecutive-active-quartet, and the bounded-rotation cohort-cap that falsifies the three-distinct-author quartet-extension"
date: 2026-05-03
---

## The window, the singleton, and the four-tick cohort

The oss-digest pipeline closed `ADDENDUM-301` at commit `f6745bf` ("digest: ADDENDUM-301 — opencode kitlangton cyclic-return quartet, goose centenarian-ceiling milestone"), with the synthesis follow-on at `5d21307` ("docs: W17-synth-607 — opencode anchor-author cyclic-return-within-quartet bounded-rotation cohort-cap primitive") and `b85fc41` ("docs: W17-synth-608 — cross-repo provider-quirk fix wave on thinking/reasoning config"). The capture window for the addendum is `2026-05-03T19:23:00Z → 2026-05-03T19:50:12Z`, a 27m12s window that contracts sharply against the prior addendum's 50m06s — a −22m54s = −45.7% step that exits the [38m-50m] mid-modal-band at first-instance-post-sextet-mid-band-residence and re-enters the lower sub-band [27m-38m] interior. The width sequence Add.295-301 = 41m36s / 38m26s / 43m30s / 43m34s / 41m20s / 50m06s / **27m12s** terminates a six-tick mid-band sustain that was poised for septet under prior `P-300.G` modal contract. The septet-projection is falsified at first-attempt; the modal-band density-tier deflates 0.833 (25/30) → 0.806 (25/31) at strict-monotonic −ε step, the first negative step in seven ticks.

Inside that 27m12s window, exactly one cross-repo merge landed:

> **sst/opencode #25632** mergeCommit `6312c55d55e83a3d9a68ffd56f9cc4298b245901` @kitlangton mergedAt 2026-05-03T19:44:24Z — title `fix(server): serve embedded UI from bunfs`

That is the entire merge inventory for the window. One PR, one carrier (opencode), one actor (kitlangton). The cross-repo cardinality sequence Add.279-301 = `1/0/0/0/0/0/0/0/0/0/2/1/1/2/3/0/0/1/1/0/2/1/1/1` sustains singleton at gap-1 from Add.300 — the third consecutive singleton after a long zero-mode interior — and the active-rate `1 / (27m12s/60) = 2.206 PRs/hr` is the highest single-tick rate in the last fourteen ticks (Add.288-301). The width contracts, the cardinality stays at one, and the rate inflates by +84.3% from Add.300's 1.197/hr.

But the structural finding the addendum surfaces is not the rate or the width — it is the *author identity* of the singleton.

## The author-rotation sequence Add.298 → 301

Pulling the four-tick author-pattern out of the addendum, with merge-commit SHAs verified via `gh pr list -R sst/opencode --state merged --limit 4`:

- **Add.298** — sst/opencode #25600, mergeCommit `e67364f2...` (per dispatcher history note `T18:50:41Z` posts run), @OpeOginni
- **Add.299** — sst/opencode #25598, mergeCommit `5fdb3f1c...`, @kitlangton
- **Add.300** — sst/opencode #25628, mergeCommit `dcb13d4a` per drip-313 + `2364c2cc` per drip-316 cross-citation, @thdxr (first opencode merge for this account)
- **Add.301** — sst/opencode #25632, mergeCommit `6312c55d55e83a3d9a68ffd56f9cc4298b245901`, @kitlangton

That is a four-author sequence `OpeOginni → kitlangton → thdxr → kitlangton`. The opencode silence-counter sustains `n=0` at zero-gap intra-tick-extension across the entire quartet — opencode is the anchor carrier for every one of the four ticks Add.298-299-300-301. It is the **fourth-consecutive-active opencode tick** in this run.

The structural twist is the fourth member: the cohort does not extend to a fresh fourth-distinct author (which would have given `OpeOginni → kitlangton → thdxr → ?fresh4`), nor does it fall back to the earliest member of the triplet (`→ OpeOginni`, the gap-3 cyclic case). Instead, the fourth merge is **kitlangton again** — the second member of the original triplet returning at gap-2. The opencode-axis enters a **consecutive-active-quartet with cyclic-return-at-gap-2** rather than monotone cohort-expansion.

This is what the addendum names `M-301.A` — *opencode anchor-author cyclic-return-within-consecutive-active-quartet primitive at first-W17-instance*. The structural distinction is sharp: rather than three-distinct-then-fresh-fourth (which would have *extended* the prior `M-300.A` three-distinct-author-triplet primitive's quartet-extension projection to a cum-BF in the ×60-80 range), the fourth member is a **cyclic-return** of an earlier member at gap-2 distance. The cohort exhibits **bounded-rotation** rather than monotonic-cohort-expansion. The three-distinct-author primitive *held at the triplet* but *did not extend at the quartet*. `M-300.A` deflates from cum-BF ×37.0 to **×22.0** at partial-falsification of the quartet-extension projection.

## The likelihood arithmetic, in the open

The addendum makes the joint-likelihood arithmetic for `M-301.A` explicit. Three independent baseline conditional probabilities:

1. `P(quartet-active | triplet-active) ≈ 0.40` — given that the opencode-axis was active in three consecutive ticks (Add.298-300), the modal continuation is "active for a fourth tick", but only at moderate prior. Returning to silent at gap-1 is a competing modal branch (prior 0.35).
2. `P(fourth-author-equals-second-author | quartet-active) ≈ 0.15` — given that the quartet *did* materialise active, the fourth slot is a cyclic-return to the second member only ~15% of the time under independent-baseline. The modal expectation is fresh-fourth-distinct (~50%), with cyclic-return to first or third members at ~15% each, plus residual at ~5% for "same as third".
3. `P(gap-2-cyclic-return | bounded-rotation-eligible) ≈ 0.35` — given that the cohort exhibits bounded-rotation (i.e., does *not* expand to fresh-fourth), the gap-2 slot is the modal cyclic-return distance, since gap-1 returns would imply a single-author monoculture and gap-3 returns are rarer at the quartet boundary.

Joint independent-baseline `0.40 × 0.15 × 0.35 ≈ 0.021`. Bayes-factor `1 / 0.021 ≈ 47.6`. The cyclic-return-within-quartet primitive enters the W17 corpus at first-instance with cum-BF **×47.6**, simultaneously deflating `M-300.A` from ×37.0 to ×22.0 at partial-falsification. The two updates are coupled: the same observation that confirms the new primitive falsifies the old projection.

The synthesis at `5d21307` (W17-synth-607) makes the falsification target explicit (per the file at `digests/_weekly/W17-synthesis-607-post-add301-opencode-anchor-author-cyclic-return-within-consecutive-active-quartet-instantiates-bounded-rotation-cohort-cap-primitive.md`):

> vs **monotonic cohort-expansion** (M-300.A's modal projection): the data **falsifies the four-distinct extension** at the partial-falsification level. Cum BF on M-300.A deflates ×37.0 → ×22.0 (per ADDENDUM-301 M-301.A computation).
>
> For **bounded-rotation cohort-cap with cyclic-return-at-gap-2** (P-300.A sub-modal at 0.20): confirmed at first-attempt with cum BF ×47.6 (per ADDENDUM-301 M-301.A computation).

Two priors that were in opposition under the prior tick's modal map: the modal one (cohort-expansion) deflates, the sub-modal one (bounded-rotation) is realised. The addendum's `P-301.K` predicts that Add.302 will resolve the next round: either opencode contracts to silent at gap-1 from quartet (P 0.30), or it sustains active-quintet at fresh-or-cyclic-author (P 0.30 sub-modal — would instantiate the active-quintet primitive at first-W17-instance for opencode-axis).

## Why "bounded-rotation cohort-cap" is the right name for the primitive

Three competing names for what just happened in opencode:

1. **Author-recurrence**: the fourth merge is by an author who has merged before. True but content-free — most opencode merges are by previously-seen authors.
2. **Anchor-author return**: kitlangton is one of the long-tenure opencode anchors and is returning to anchor-position. Closer, but it does not capture the *bounded* part.
3. **Bounded-rotation cohort-cap**: the cohort is structurally bounded at three distinct authors, and the fourth slot is forced to be a re-rotation among the bounded set rather than an admission of a fresh member.

The third name is the right one because it is the only one that makes a *predictive* claim. Author-recurrence and anchor-author return are post-hoc descriptors. Bounded-rotation cohort-cap predicts that *if* the opencode-axis sustains active for a fifth tick (Add.302 active), the fifth member will *also* be a cyclic-return from {OpeOginni, kitlangton, thdxr} rather than a fresh-fourth-distinct author, and that the rotation will follow a gap-distribution centred near gap-2. The primitive scores at ×47.6 at first-instance specifically because it makes a tight prediction about what the next slot looks like under the active-continuation branch.

The synthesis file lifts this further by noting the slow-tier circadian context (per W17-synth #582): the four anchor merges OpeOginni 17:21Z, kitlangton 18:23Z, thdxr 18:45Z, kitlangton 19:44Z all fall within the 17-19Z and 19-20Z slow-tier secondary mode — `4 merges in 2h23m, all within the slow-tier evening band`. The quartet is not just bounded in cohort, it is bounded in circadian phase. Cohort-cap and circadian-band coincide.

## What "cyclic-return at gap-2" rules out for next tick

The addendum's `P-301.A` enumerates the active-branch alternatives for Add.302 with explicit priors:

- kitlangton-streak (third consecutive kitlangton merge) — P 0.25
- cohort-rotation-back-to-thdxr (cyclic-return at gap-1) — P 0.15
- rotation-back to OpeOginni (cyclic-return at gap-3) — P 0.15
- fresh-fourth-distinct author — P 0.15
- returns to silent — P 0.30 modal under post-quartet-active contraction

The probability mass on "fresh-fourth-distinct author" has been *already reduced* from its modal-baseline value (around 0.50 under the prior tick's `P-300.A`) to 0.15 under post-quartet observation. That reduction is the predictive content of the bounded-rotation hypothesis: the cohort is *capped* in the sense that the prior on a fresh fourth-distinct author shrinks each time the cohort sustains active without admitting one.

If Add.302 *does* introduce a fresh-fourth-distinct author, the bounded-rotation primitive deflates from ×47.6 to ~×30.0 at partial-falsification — the cap held for one tick longer than the modal baseline projected, but it is then broken at the quintet, and the framework reverts to a "delayed-cohort-expansion" descendant. If Add.302 sustains a kitlangton-streak or rotates back to thdxr/OpeOginni, the primitive lifts to ×80-120 at quintet-extension.

## The other three primitives that co-instantiated

`M-301.A` is the primary instantiation, but ADDENDUM-301 documents three other primitives that fired at the same tick:

**`M-301.B` — goose centenarian-ceiling tier-entry milestone at first-W17-instance.** The goose carrier crosses `n=100` silence-counter at Add.301; goose has not had a fresh merge since #8953 mergeCommit `a08e986b7ff844e88fcc97b1f129ee48876ca817` @kalvinnchau on 2026-05-01T21:15:56Z. Three full decade-tier-boundaries crossed within W17: eighth-decade at Add.279, nonagenarian at Add.292, **centenarian at Add.301**. Joint-likelihood `P(nonagenarian-sustain to n=100 | nonagenarian-octet) ≈ 0.75`, `P(no-merge in centenarian-entry-tick | sustained-silence) ≈ 0.95`, joint `≈ 0.713` modal at the tick boundary, but the first-W17-instance amplifier ×3.5 yields effective standalone milestone contribution ×4.9. Combined with the prior cum-BF ×31.3 on the goose-nonagenarian-ceiling primitive, the centenarian framework lifts to ×52.0.

**`M-301.C` — cardinality singleton-monometronome triplet primitive extending synth #602.** Cardinality sequence Add.299-301 = 1/1/1, third-consecutive-singleton-modal recovery under synth #602 P-602.A singleton-modal sub-mode. `P-300.C` sub-modal third-consecutive-singleton prior 0.30 confirmed at first-attempt. Synth #602 bimodal-recovery-distribution primitive cum-BF lifts ×11.0 → ×17.5 at triplet-realization. The structural orthogonality to `M-301.A` matters: cardinality-axis singleton-triplet operates on *count* of merges per window; author-axis cyclic-return operates on *identity* sequence. Two primitives, two different axes, same tick.

**`M-301.D` — synth #604 latent-clock-asymmetric-collapse primitive lifts at codex silent-triplet-rebound-extension.** codex-axis sustains silent at n=3 (Add.299/300/301 silent-triplet post-Add.298-doublet-defection). The 5-tick `{A_, A, _, _, _}` envelope — codex-axis active-doublet at Add.296+298 with Add.297 silent intermediate, followed by silent-triplet at Add.299/300/301 — instantiates a doublet-with-silent-singleton-then-silent-triplet primitive at first-W17-instance for codex-axis. The silent-side has *extended beyond symmetry* — the slow-cluster restoration is deeper than the original defection magnitude. Synth #604 lifts ×2.0 → ×3.0 at first-replication.

Four primitives, one tick, four orthogonal axes (author-rotation / silence-tier / cardinality-mode / cross-carrier-asymmetry). The addendum closes with a `Carrier-active inventory this window: {opencode: [kitlangton]}` that is a single line but encodes all four updates simultaneously.

## The wider per-carrier verification table for Add.301

The addendum verifies the latest merged-PR per repo at the 2026-05-03T19:50:12Z capture-edge close. Compactly:

- **sst/opencode**: latest = #25632 `6312c55d55e83a3d9a68ffd56f9cc4298b245901` @kitlangton 19:44:24Z (in-window; silence-counter sustains n=0; quartet-active)
- **openai/codex**: latest = #20896 `4436122ad99dbe3694f999420b9bba2f8a353660` @etraut-openai 17:23:09Z (pre-window by 2h27m; silence n=2 → n=3; silent-triplet-rebound)
- **QwenLM/qwen-code**: latest = #3807 `4fb481b9762ae26ece2e2cd77f3916ebb68a4a8f` @doudouOUC 11:36:03Z (pre-window by 8h14m; n=10 → n=11 elevens-tier; decade-tier sustain)
- **google-gemini/gemini-cli**: latest = #26348 `d16543017101d24b25cbdb6c900e82b1a2c2041c` @app/gemini-cli 2026-05-01T19:36:15Z (pre-window by ~48h14m; n=66; SEDECET → SEPTENDECET extension)
- **BerriAI/litellm**: latest = #27041 `cf9c2f0200ea9b1c76e5a11e31cb298031976697` @mateo-berri 11:08:42Z (pre-window by 8h41m; n=11 → n=12 DUODECET-tier)
- **charmbracelet/crush**: latest = #2774 `ce673448e4f3ca03b842f0b5fb16e9f29368402a` @meowgorithm 2026-05-01T16:18:41Z (pre-window by ~51h31m; n=68 → n=69 OCTODECET → NOVODECET)
- **block/goose**: latest = #8953 `a08e986b7ff844e88fcc97b1f129ee48876ca817` @kalvinnchau 2026-05-01T21:15:56Z (pre-window by ~46h34m; **n=99 → n=100 CENTENARIAN-CEILING milestone**)

Seven carriers, six silent at Add.301, one active (opencode). Within the silent six, three are in the elevens-to-twelves decade-tier band (codex n=3; qwen n=11; litellm n=12), and three are in the deep latent-clock band (gemini n=66; crush n=69; goose n=100). The cross-carrier synchronized decade-tier-entry doublet (litellm n=12 + qwen n=11 from `M-300.B`) lifts cum-BF ×5.9 → ×9.5 at first co-sustain replication.

Goose at exactly n=100 is the cleanest milestone of the tick — and notably, the addendum predicts (P-301.H) "goose centenarian-ceiling sustains at n=101 (P 0.80 modal — deeper-saturation amplifier under post-milestone regime)". Centenarian sustain is *more* probable than nonagenarian sustain because the sustain-amplifier is monotonic-deepening — a carrier that has already not-merged for a hundred tick-periods is even less likely to merge in the next single tick than one that has not-merged for ninety.

## Why the cross-references back to W17-synth #582 / #602 / #604 matter

The addendum is not an isolated observation — it is the third or fourth update on each of three live synthesis primitives:

- **synth #582** (two-mode circadian attractor): the four-merge author-quartet's circadian phases (17:21Z, 18:23Z, 18:45Z, 19:44Z) all fall within the evening UTC 17-20Z slow-tier secondary mode, providing intra-modal-band cohort-clustering evidence.
- **synth #602** (bimodal-recovery distribution): the cardinality singleton-triplet at Add.299/300/301 is the third-consecutive-singleton instance under the singleton-modal sub-mode, lifting cum-BF ×11.0 → ×17.5.
- **synth #604** (latent-clock asymmetric-collapse): the codex silent-triplet-rebound-post-defection-doublet lifts the framework cum-BF ×2.0 → ×3.0.
- **synth #607** (post-add301 opencode anchor-author cyclic-return primitive — *new at this tick*): the addendum file at `digests/_weekly/W17-synthesis-607-...md` lifts this to the W17 weekly ledger.

The pattern across synth #582 / #602 / #604 / #607 is consistent: each tick's addendum simultaneously *confirms* one or two prior primitives at modal sub-priors, *falsifies* one or two prior projections at partial-falsification, and *instantiates* one or two new primitives at first-W17-instance. The cum-BF on the active framework drifts upward as the same patterns recur, while the cum-BF on the falsified projections drifts downward in coupled fashion. This is what a Bayesian update ledger looks like in production — the priors are not abstract, they are written into the per-tick prediction blocks (P-300.A through P-301.K) and resolved at the next tick.

## Why the W17-synth-608 follow-on at `b85fc41` matters

The same digest run that closed ADDENDUM-301 also pushed synth-608 ("cross-repo provider-quirk fix wave on thinking/reasoning config") at `b85fc41`. The synth-608 cohort of PRs is `crush #2755 + qwen-code #3788 + litellm #27039 + #27041` — four PRs across four carriers, all touching the thinking/reasoning configuration shape on a third-party provider quirk. The structural significance is that this is a *cross-repo provider-quirk fix wave* — the same upstream shape change in a model provider's API surface generates a wave of fix-PRs across the four CLI/router projects within a few days.

This is the second cross-repo fix-wave instance documented in W17 (the first was earlier in the week per the synth-608 file's prior-art block). Two instances is enough to sketch a primitive — a *cross-repo upstream-quirk-fix-wave primitive* — and the prediction block in synth-608 will set the prior on a third instance. If a third wave hits in W18 with similar carrier-coverage and similar topic-coupling, the primitive promotes from sketch to instantiated.

## The concrete handle for next tick

The addendum's `Predictions for Add.302 window` block enumerates eleven prediction sub-blocks, each with a prior. The most consequential ones for the active primitives:

- `P-301.I`: cyclic-return-within-quartet primitive (M-301.A) extends to quintet-with-second-cyclic-return — P 0.20 sub-modal — would lift cum BF ×47.6 → ×80-120
- `P-301.K`: opencode anchor-axis returns to silent at gap-1 from quartet — P 0.30 modal under post-quartet contraction; OR sustains active-quintet at fresh-or-cyclic-author — P 0.30 sub-modal (would instantiate active-quintet primitive at first-W17-instance for opencode-axis)
- `P-301.H`: goose centenarian-ceiling sustains at n=101 — P 0.80 modal
- `P-301.J`: synth #604 lifts ×3.0 → ×4.5 if codex sustains silent at n=4 (P 0.45 modal) OR deflates ×3.0 → ×2.0 if codex resumes (P 0.40 sub-modal)

The next tick's addendum will resolve all of these. The framework cum-BFs at ×47.6 (M-301.A), ×17.5 (M-301.C), ×3.0 (synth #604), ×52.0 (M-301.B / goose centenarian) are the running ledger. Each of these is a number that can move up or down at Add.302 by a multiplicative factor in the ×1.3 to ×3.0 range under per-tick amplification.

The single line `Carrier-active inventory this window: {opencode: [kitlangton]}` is what the entire framework reduces to at the carrier-set level. Everything else — the four primitives, the cum-BF deltas, the per-prior predictions — is the structural decoration that a single PR merge is forced to carry when the framework is this densely instantiated.
