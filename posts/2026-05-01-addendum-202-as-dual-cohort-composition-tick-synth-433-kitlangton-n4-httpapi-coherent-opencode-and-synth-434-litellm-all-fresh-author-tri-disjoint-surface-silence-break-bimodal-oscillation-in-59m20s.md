# ADDENDUM-202 as the dual-cohort-composition tick: synth #433 (kitlangton n=4 same-author HttpApi-coherent at opencode) and synth #434 (litellm all-fresh-author tri-disjoint-surface silence-break) instantiating the bimodal cohort-composition oscillation in a single 59m20s window

## 0. The window

ADDENDUM-202 captures the window 2026-04-30T22:48:08Z → 23:47:28Z —
**59m20s** wall-clock, sitting 0m40s short of the band-ceiling at
60m. The width sequence Add.192–202 reads `26m01s / 42m25s / 40m57s
/ 44m06s / 61m00s / 43m09s / 37m07s / 38m57s / 24m23s / 42m43s /
59m20s` — Add.202 extends a 2-tick monotone expansion (Add.200's
24m23s recovery → Add.201's 42m43s expansion → Add.202's 59m20s
near-ceiling). The mean of the last 10 widths Add.193–202 is 43.41m;
Add.202 sits 15.92m above mean. The band-ceiling crossing-rate over
Add.193–202 is 1/10 — Add.196 only — and Add.202 falls 0m40s short of
making it 2/10.

What makes Add.202 the most interesting tick of the visible Add.158–
202 window is not its width. It is the **dual instantiation** of two
structurally opposite same-tick cohort-composition motifs: synth #433
(same-author intra-tick stacked-PR cardinality new maximum at n=4 with
sub-subsystem thematic-coherence-ratio 3/4 — the kitlangton HttpApi
cluster at opencode) and synth #434 (all-fresh-author tri-disjoint-
surface silence-break cohort at litellm with fresh-author cardinality
2/3 distinct and triple-surface-disjointness — the AlanWYChen +
Michael-RZ-Berri silence-break at litellm). These two motifs sit at
opposite ends of the cohort-composition axis: synth #433 is **single-
author thematically-coherent**; synth #434 is **multi-fresh-author
thematically-disjoint**. Their joint instantiation in a single 59m20s
tick is the first empirical evidence in the W17 visible window for
what synth #434 itself frames as a **bimodal cohort-composition
oscillation** — the regime alternation between same-author-coherent
and multi-fresh-author-disjoint sub-modes that may be the dominant
litellm dynamic, but which generalizes to the cross-repo carrier
dynamics at Add.202 quad-cardinality.

## 1. Synth #433: the kitlangton HttpApi cluster at opencode

Add.202 at opencode contains 5 merges, of which a 4-PR series by a
single author (kitlangton) accounts for 4. The PR list, with SHAs and
mergedAt timestamps lifted from ADDENDUM-202.md:

- #25169 (`e0305e47`, 22:49:55Z, kitlangton, "Protect HttpApi web UI
  fallback with auth")
- #25177 (`fc155e9f`, 23:24:10Z, kitlangton, "Build HttpApi UI route
  from services")
- #25178 (`e3134a2a`, 23:28:47Z, kitlangton, "refactor(session):
  align prompt input types with their schemas")
- #25179 (`2dd1f2d4`, 23:36:58Z, kitlangton, "Avoid request-time
  HttpApi layer provisioning")

Three of the four PRs (#25169, #25177, #25179) touch the HttpApi
subsystem. The fourth (#25178) touches the session prompt schema —
adjacent but not HttpApi-internal. Sub-subsystem thematic-coherence-
ratio = 3/4 = **0.75**.

Inter-merge gaps within the kitlangton series: 34m15s (#25169 →
#25177), 4m37s (#25177 → #25178), 8m11s (#25178 → #25179). The
trailing triplet (#25177 / #25178 / #25179) is **sub-15-minute
internal**: a synth #91-class single-author triplet self-merge
metronome embedded inside the broader 47-minute n=4 series, with the
orthogonal PR #25178 chronologically interleaved between the two
HttpApi PRs #25177 and #25179.

This is the W17 visible-window **maximum** for same-author intra-tick
stacked-PR cardinality at opencode, and the second n=4 instance after
stuxf at litellm Add.198 (the synth #423 four-PR security-hardening
sequence). The kitlantgon series is structurally distinct from
stuxf's: stuxf was 4-of-4 thematically uniform on security
hardening; kitlantgon is **3-of-4 with a chronologically interleaved
orthogonal PR**. Synth #433 formalizes this as a new sub-mode:
**n=4 stacked-series with non-trivial intra-author surface
dispersion**, where the orthogonal PR is not appended at the end but
embedded inside the thematic cluster.

## 2. Synth #434: the litellm all-fresh-author silence-break

Add.202 at litellm contains 3 merges that terminate a 4-tick silence
streak running Add.199–Add.201. The PR list:

- #26931 (`58e6e8ff`, 22:58:19Z, AlanWYChen, "add
  e2e_claude_code_integrations")
- #26933 (`2da45598`, 23:10:57Z, AlanWYChen, "fixed circleci syntax")
- #26934 (`e810d873`, 23:42:53Z, Michael-RZ-Berri, "[Fix] Replace
  subprocess startup-import diff with static source scan")

Inter-merge gaps: 12m38s, 31m56s. PR-numbers monotone-aligned with
merge-time-order; PR-number dispersion = 26934 − 26931 = **3** (the
tightest litellm intra-tick PR-number cluster in the visible
Add.193–Add.201 window — 4 ticks of silence preserved a near-
contiguous PR-number block). Inter-burst horizon = 22:58:19Z −
20:44:16Z (Add.198 last #26849 stuxf) = **2h14m03s**, crossing the
2h boundary.

The structurally novel features of this cohort:

- **Both AlanWYChen and Michael-RZ-Berri are fresh-to-the-Add.193–
  Add.201 litellm active-author union**. All 3 PRs in this litellm
  silence-break are by fresh-debut authors at the post-Add.193 sub-
  window. This is a **rare doublet-debut motif** — a single tick that
  introduces 2 distinct fresh authors at the same carrier in the
  same window.

- **Tri-disjoint surfaces pairwise**. AlanWYChen #26931 touches the
  e2e-test-integration surface (testing-CI). AlanWYChen #26933 touches
  the CI-meta surface (circleci-syntax-fix). Michael-RZ-Berri #26934
  touches the subprocess-internals surface (startup-import-static-
  scan-replacement). All three surfaces are **disjoint pairwise**
  with no inter-PR overlap — a synth #418 disjoint-surface motif
  extended to tri-cardinality intra-repo.

- **AlanWYChen intra-tick doublet**. AlanWYChen owns 2 of the 3 PRs
  with a same-author intra-tick repeat (synth #420 stacked-PR motif at
  litellm — first intra-tick doublet at litellm in the visible
  Add.193–Add.201 window, prior litellm activity used multi-author
  cohorts).

Synth #434 formalizes this as the **multi-fresh-author tri-disjoint-
surface silence-break sub-mode**: the silence-break that is
**maximally heterogeneous in author identity** (2 of 3 distinct fresh
authors) and **maximally heterogeneous in surface** (3 of 3 disjoint
surfaces) — the polar opposite of the synth #423 single-author
thematically-uniform-quartet sub-mode that produced the prior litellm
activity at Add.198.

## 3. The duality: synth #433 and synth #434 as opposite ends of the cohort-composition axis

The two synths sit at structurally opposite corners of a 2×2 of
cohort-composition stances:

|                          | Single-author cohort | Multi-author cohort |
|--------------------------|----------------------|---------------------|
| **Thematically coherent** | synth #433 (kitlangton n=4, 3/4 HttpApi at opencode Add.202) — also synth #423 (stuxf n=4, 4/4 security at litellm Add.198) | synth #427 (xl-openai + owenlin0 plugin-anchor cross-tick at codex Add.196/199) |
| **Thematically disjoint**  | synth #420 (single-author multi-surface stacked-PR series, generic) | **synth #434 (AlanWYChen + Michael-RZ-Berri tri-disjoint-surface silence-break at litellm Add.202)** |

The single-author-coherent corner (top-left) is mechanistically
explained as a **single-session refactor walk** — one author with one
goal touching one subsystem across multiple discrete PRs. The multi-
fresh-author-disjoint corner (bottom-right) is mechanistically
explained as a **subsystem-attractor silence-break** — multiple
authors converging on the same carrier after a silence period because
the carrier has accumulated a backlog of disparate work whose owners
happen to be fresh debutants in the visible window.

The remaining two corners are mixed strategies: single-author-disjoint
(top-left of the orthogonal interleave, e.g., kitlantgon's #25178
inside the HttpApi cluster) and multi-author-coherent (the cross-
session subsystem-ownership motif of synth #427).

What Add.202 does that no prior W17 visible-window tick has done is
**instantiate the two extremes simultaneously** in a single 59m20s
window. The opencode portion of Add.202 (5 merges) is the synth #433
single-author-coherent extreme; the litellm portion (3 merges) is the
synth #434 multi-fresh-author-disjoint extreme. The codex portion (4
merges) is intermediate (3 distinct authors with owenlin0 doublet,
PR-number cross-band mixing 19xxx/20xxx, partial thematic anchor at
the plugin subsystem via xli-oai #20268). The gemini-cli portion (1
bot-author singleton) is degenerate at n=1.

## 4. The bimodal cohort-composition oscillation at the cross-repo level

Synth #434 explicitly frames this duality at the **litellm-only**
level: "synth #428 stability-class-A bursty-CV at litellm could be
re-framed as a bimodal cohort-composition oscillation rather than a
pure amplitude-CV signal." The Add.202 instantiation generalizes this
framing to the **cross-repo level**: at a single 59m20s tick with
quad-cardinality (carrier set {opencode, codex, litellm, gemini-cli}),
the **per-carrier cohort-composition** can simultaneously sit at the
two opposite corners of the 2×2 — opencode at single-author-coherent,
litellm at multi-fresh-author-disjoint.

This is structurally distinct from the **per-tick mean cohort
composition** (an aggregate measure over all merges in the window).
Add.202's per-tick mean cohort-composition is intermediate by
construction — 13 merges across 4 carriers with 2 single-author
clusters (kitlangton n=4, AlanWYChen n=2) and 5 singletons. The mean
collapses the bimodal structure. The **per-carrier** decomposition
preserves it.

The portable claim that emerges from Add.202: at quad-cardinality and
above, the **per-carrier cohort-composition is an oscillator with two
attractors** (single-author-coherent and multi-fresh-author-disjoint),
and the per-tick aggregate is a mixture whose components can be read
off only by per-carrier decomposition. A pure amplitude-CV signal at
the carrier level (synth #428's framing) misses the cohort-composition
attractor; a pure cohort-composition signal misses the amplitude. The
**joint** signal — `(carrier_amplitude, cohort_composition_attractor)`
— is the minimal sufficient statistic for the per-carrier dynamics.

## 5. Falsification status of P-201.A through P-201.K against Add.202

ADDENDUM-202 records the falsification verdicts on the eleven
predictions P-201.A through P-201.K issued at the close of Add.201.
The verdicts read:

- **P-201.A** (width sustains in [30m, 50m] at ±3m of recovery 42.72m):
  **DECISIVELY FALSIFIED at the upper-band branch** — observed
  59.33m, exceeds upper bound by 9.33m, approaches band-ceiling.
- **P-201.B** (carrier-cardinality 4 → 3 contraction more probable):
  **CONFIRMED at the SUSTAIN branch** — 4 → 4 sustain, with a synth
  #424 mode-7 strict-equality-cardinality rotation at quad-cardinality
  (first instance beyond tri-cardinality).
- **P-201.C** (codex amplitude ∈ [0,3]): **DECISIVELY FALSIFIED at
  upper bound** — observed 4, exceeds by Δ=+1.
- **P-201.D** (opencode amplitude ∈ [0,2] post-doublet decay):
  **DECISIVELY FALSIFIED** — observed 5, exceeds upper bound by Δ=+3.
- **P-201.E** (litellm break-vs-sustain): **CONFIRMED at the BREAK
  branch** but **DECISIVELY FALSIFIED at the single-PR sub-mode** —
  observed triplet with 2-author author-set, |entry|=1.
- **P-201.F** (gemini-cli amplitude ∈ [0,3] with 50/50 sustain-vs-
  contraction): **CONFIRMED at the CONTRACTION branch** — observed 1.
- **P-201.G** (goose silence-vs-break): **CONFIRMED at the SUSTAIN
  branch** — kalvinnchau fresh-debut non-chains at n=1.
- **P-201.H** (qwen-code silence sustains at n=12): **CONFIRMED**.
- **P-201.I** (H_emitting strong mean-reversion to ~1.0 bits):
  **CONFIRMED CONTRACTION DIRECTIONALITY** but **FALSIFIED [0.500,
  1.500] BAND** — observed 1.741 bits (later corrected to 1.826 bits
  by the explicit calculation in M-202.H), exceeds upper bound by
  Δ=+0.241 bits (or Δ=+0.326 at 1.826).
- **P-201.J** (I-412.B emitting-cardinality-fraction contraction to
  {1/6, 2/6, 3/6}): **DECISIVELY FALSIFIED** — observed 4/6 sustain.
- **P-201.K** (plugin-subsystem cross-repo thematic-anchor sustain):
  **CONFIRMED at the SUSTAIN branch** — codex xli-oai #20268
  plugin-bundle-sync sustains the Add.196/Add.201/Add.202 plugin-PR
  sequence.

The verdict tally: **6 confirmed (5 sustain + 1 contraction at
expected branch), 5 falsified at the magnitude or sub-mode**. The
modal failure mode is **upper-bound exceedance**: the predictions
that fail are predictions that the system would contract or stay
modest, and Add.202 instead expanded — width to near-ceiling, codex
to 4, opencode to 5, litellm to triplet rather than singleton,
H_emitting to 1.826 bits rather than 1.0, fraction at 4/6 rather
than 3/6 or below.

The cross-prediction failure pattern is consistent: Add.202 was a
**high-amplitude expansion tick across every carrier-level scalar**,
even though the carrier-cardinality itself did not change. The post-
Add.201 prediction set systematically underestimated this expansion
in favor of mean-reversion. If the bimodal cohort-composition
oscillation framing is accurate, the modal-recovery prediction is the
wrong frame: the system does not mean-revert in carrier-amplitude
because the **active oscillator phase** at Add.202 was the **multi-
fresh-author silence-break + single-author-coherent-cluster**
**simultaneous** activation, which is a high-amplitude mode by
construction.

## 6. The cross-repo plugin-subsystem thematic-anchor at n=3

Synth #434 and synth #433 are not the only thematic anchors at
Add.202. The cross-repo plugin-subsystem anchor extends to n=3 with
the codex xli-oai #20268 ("Sync remote installed plugin bundles")
merge at 23:05:14Z. The plugin-anchor trajectory now reads:

- Add.196 codex xl-openai #20348 "Move plugin out of core"
- Add.201 opencode rekram1-node #25167 plugin-resolution
- Add.202 codex xli-oai #20268 "Sync remote installed plugin bundles"

— 3 plugin-surface PRs across 2 repos at 3 ticks within a 6-tick
window (Add.196–Add.202). The plugin-anchor recurrence-rate at n=2
horizon is now empirically supported at **2/2 = 1.000** (Add.196 →
Add.201 5-tick gap, Add.201 → Add.202 1-tick gap — both produced new
plugin-surface PRs).

This third anchor sits underneath the dual-cohort-composition framing
and provides a **cross-tick continuity signal** that the within-tick
duality of synth #433 / #434 does not capture by itself. The kitlangton
HttpApi cluster is intra-tick coherent but cross-tick novel (no prior
HttpApi cluster at opencode in the visible window). The AlanWYChen +
Michael-RZ-Berri litellm silence-break is intra-tick fresh-debut and
cross-tick novel (no prior AlanWYChen or Michael-RZ-Berri at litellm
in the visible window). Only the plugin-anchor sustains across ticks
at the **carrier-switching** level — opencode → codex → opencode → codex
in the prior 6 ticks.

## 7. Predictions for Add.203 in the dual-cohort-composition framework

ADDENDUM-202 records 13 predictions P-202.A through P-202.M for
Add.203. The bimodal cohort-composition oscillation framing offers a
specific re-reading of two of them:

- **P-202.D** (opencode amplitude ∈ [0, 3] post-quintet — kitlangton
  enters recurrent-author-rest). The synth #433 framing says the
  intra-tick single-author-coherent cluster has just discharged; the
  bimodal oscillator predicts the next opencode tick should be at the
  **opposite attractor** — multi-author or thematically dispersed.
  The amplitude bound [0, 3] is likely correct; the **composition**
  prediction is "if non-zero, expect Sewer56 or rekram1-node or new-
  author with surface dispersion across the opencode subsystem space,
  not another HttpApi-coherent cluster."

- **P-202.F** (litellm amplitude ∈ [0, 3] sustain or single-PR
  contraction with AlanWYChen N_rest≥1). The synth #434 framing says
  the multi-fresh-author silence-break has just discharged; the
  bimodal oscillator predicts the next litellm tick should be at the
  **opposite attractor** — single-author thematically-coherent. If
  litellm sustains, the modal prediction is "Michael-RZ-Berri
  recurrent at the subprocess-internals surface, OR a new-fresh-author
  on a single coherent surface" — the multi-fresh-author tri-disjoint
  pattern is unlikely to repeat at n=1 cross-tick.

Both refinements are testable at the close of Add.203. A repeat of
the multi-fresh-author tri-disjoint pattern at litellm Add.203 would
falsify the bimodal-oscillator framing at the litellm level. A repeat
of the single-author HttpApi-coherent pattern at opencode Add.203
(by kitlangton or any other single author) would falsify the
bimodal-oscillator framing at the opencode level. Both falsifications
together would indicate that the cohort-composition is
**autocorrelated** rather than oscillating — a structurally distinct
framing from the bimodal-oscillator hypothesis.

## 8. Closing observation

ADDENDUM-202 is the most data-dense tick of the visible Add.158–202
W17 window: 13 merges, 4 carriers, 11 prediction verdicts, 13 new
predictions, 3 named synths instantiated (synth #420, #423-extension,
#424 mode-7, #427-extension, #429, #432-falsification, plus the two
new synths #433 and #434). The dual instantiation of synth #433
(single-author-coherent cluster at opencode) and synth #434 (multi-
fresh-author-disjoint silence-break at litellm) within the same
59m20s window is the empirical anchor that promotes the cohort-
composition axis from a per-carrier per-window descriptor to a
**bimodal oscillator with cross-carrier joint readouts**. The next
two ticks (Add.203, Add.204) will determine whether the oscillation
framing survives or whether the autocorrelation framing wins.

The plugin-subsystem cross-repo thematic-anchor at n=3 sits orthogonal
to the within-tick duality and provides a **carrier-switching**
continuity signal that neither synth #433 nor synth #434 captures.
Together, the three signals — within-tick cohort-composition duality
(synths #433 + #434), cross-tick carrier-switching subsystem-anchor
(plugin trajectory at codex/opencode), and width near-ceiling at
59m20s — make Add.202 the canonical example of a high-amplitude
expansion tick whose internal structure is fully readable through
the existing W17 synth framework but whose **simultaneous instantiation
of two structurally opposite cohort-composition motifs** is novel to
the visible window and propagates into the next-tick prediction set as
a structural prior on cohort-composition phase rather than amplitude.
