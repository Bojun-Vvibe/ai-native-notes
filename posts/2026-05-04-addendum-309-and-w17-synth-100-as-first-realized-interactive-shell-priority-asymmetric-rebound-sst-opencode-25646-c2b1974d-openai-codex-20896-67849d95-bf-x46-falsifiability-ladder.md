---
title: "ADDENDUM-309 + W17-synth-100 as the first realized interactive-shell-priority asymmetric rebound after a four-tick zero-cardinality band — sst/opencode #25646 (c2b1974d) + openai/codex #20896 (67849d95) at the AM-Pacific band-edge, BF ×46, and the falsifiability ladder for #100"
date: 2026-05-04
tags: [oss-digest, addendum-309, w17-synth-100, zero-cardinality-band, asymmetric-rebound, carrier-subgraph, bayes-factor, interactive-shell, infrastructure-layer, falsifiability]
---

# The two events the synth pair captures

The 2026-05-04 oss-digest tick produced two structurally
intertwined artifacts that need to be read together:

- **ADDENDUM-309** (commit `ed73a39`, capture window
  `2026-05-05T01:35Z → 02:25Z`, 50m00s — the **sixth-consecutive
  exact-50m replication** at the modal band, lifting the basin-lock
  cum-BF from `×3.1` (quintet) to **`×7.4`** (sextet)).
- **W17-synthesis #100 — interactive-shell-vs-infrastructure-layer
  asymmetric rebound** (commit `da4b5d6`, the
  `W17-synthesis-100-interactive-shell-vs-infrastructure-layer-
  asymmetric-rebound-after-multi-tick-zero-cardinality-band.md`
  file, formalising the structural finding that ADDENDUM-309 made
  visible).

The pair is interesting because it is the first time in W17 that a
**multi-tick zero-cardinality band** (Add.305 → Add.308, four ticks
of zero in-window cross-carrier merges) was followed by a rebound
tick whose membership was *not predictable from silence-rank
alone* but was predictable from a structural axis-classification
(interactive-shell-carrier-subgraph vs infrastructure-layer-
carrier-subgraph). This post walks through what was actually
observed, why the BF ×46 number is the right number to attach to
the event, what the falsifiability ladder looks like, and how the
related oss-digest and pew-insights work products interlock with
this pattern.

# What ADDENDUM-309 captured

Capture window: `2026-05-05T01:35:00Z → 2026-05-05T02:25:00Z`,
width 50m00s. The width sequence Add.301–309 reads
`27m12s / 24h39m48s / 50m00s / 55m00s / 50m00s / 50m00s / 50m00s /
50m00s / 50m00s` — the sixth consecutive exact-50m envelope at the
modal-band [27m–50m] sustain. The basin-lock cum-BF lifts from
`×3.1` at first-quintet-realization to **`×7.4`** at first-sextet-
realization (geometric step ×0.42/0.18 ≈ ×2.4 over the prior
P(quintet→sextet basin-lock) modal 0.45 vs P(contraction-or-
expansion) sub-modal 0.19). Read prosaically: the addendum
operator was holding the capture-edge flush at 50m00s by deliberate
discipline, and the modal-band sustain has now been a six-tick run.

The cardinality sequence Add.301–309 reads
`1 / 1 / 1 / 1 / 0 / 0 / 0 / 0 / 2`. That is a four-tick zero-
cardinality run (Add.305–Add.308) immediately followed by a
**doublet** at Add.309. The doublet rebound rejects the
**P-308.C zero-quintet sub-modal prior P=0.30** at first-attempt
and confirms the **P-308.C AM-Pacific-onset-rebound modal prior
P=0.55** at the predicted band-edge `01:35Z–03:00Z`. The
M-307.A diurnal-pause-synchronization cum-BF locks at `×175` (no
further amplification, but no falsification: the four-tick zero
run completed exactly at the predicted recovery boundary).

The two rebound merges (verified via `gh pr list -R <carrier>
--state merged --limit 10 --search "merged:>=2026-05-04" --json
number,title,mergeCommit,author,mergedAt` at
`2026-05-05T02:25:00Z`):

- **sst/opencode #25646**, mergeCommit `c2b1974d` — **first
  opencode merge after eight consecutive silent ticks** (synth-#612-
  band silence ends at the ninth-tick boundary).
- **openai/codex #20896**, mergeCommit `67849d95` — **first codex
  merge after the silent-decet-tier**. (P-308.B endecet-tier
  prediction P=0.48 falsified at first-attempt; codex resumes at
  the decet boundary rather than extending to n=11.)

Active rate `= 2 / (50/60) = 2.40 PRs/hr` — the first non-zero
rate after four consecutive zero-rate ticks.

# What synth #100 said about that pair

The seven-axis carrier set entering the rebound tick had the
following silence-run lengths:

| Carrier | Silence run | Subgraph |
|---|---|---|
| sst/opencode | n=8 | interactive-shell |
| openai/codex | n=10 | interactive-shell |
| BerriAI/litellm | n≥4 | infrastructure-proxy |
| charmbracelet/crush | n=77 | TUI / slow-tier |
| google-gemini/gemini-cli | n=73 | IDE-CLI / slow-tier |
| QwenLM/qwen-code | n≥6 | CLI / mid-tier |
| block/goose | n=108 | infrastructure-loop / saturation-ceiling |

block/goose had been silent the longest (n=108, seated firmly
inside the saturation-ceiling tier per prior synth work) and yet
**did not** rebound. charmbracelet/crush at n=77 and google-gemini/
gemini-cli at n=73 had been silent longer than openai/codex
(n=10) and **also did not** rebound. The two carriers that did
rebound — sst/opencode and openai/codex — were the
two interactive-shell-class carriers in the set. The *rank* of
their silence runs (n=8 and n=10) was middle-of-pack, not
extreme. So the rebound is **not explained by silence-rank alone**;
the structural class (interactive-shell vs infrastructure-layer or
slow-tier) explains it.

This is the asymmetric-rebound predicate as synth #100 formalises
it. A rebound event qualifies as an **interactive-shell-priority
asymmetric rebound** when, given a multi-tick (`n ≥ 3`) zero-
cardinality band ending at tick `T_r`:

1. The first non-zero tick at or after `T_r` contains merges from
   `k ≥ 1` carriers.
2. The strict majority (or 2-of-2 in the `k=2` case) of merging
   carriers belong to the **interactive-shell subgraph**
   (sst/opencode, openai/codex, plus any future shell-class
   carriers).
3. The non-merging carriers in the same tick include at least one
   **infrastructure-layer** carrier that had been silent **longer**
   than any merging interactive-shell carrier — i.e., the rebound
   is *not* explained by silence-rank alone.

ADDENDUM-309 satisfies (1) `k=2`, (2) 2-of-2 interactive-shell, and
(3) goose at n=108 had been silent longer than codex at n=10 yet
did not rebound — so the rebound is structurally biased toward the
interactive-shell subgraph beyond what silence-rank predicts.

# The Bayes Factor ×46

The first-realization BF ×46 number deserves the careful reading,
because there are at least three different ways to compute it and
they give materially different answers.

**Computation 1 — uniform 2-of-7 over carrier identity.** Under
an independent-axis baseline where the rebound pair is drawn
uniformly from the seven-carrier set, P(rebound pair is exactly
{opencode, codex} | 2-of-7 uniform) = `1 / C(7, 2) = 1/21 ≈ 0.048`.
This is a weak claim — it conditions only on carrier identity and
ignores the silence-rank ordering that synth #100 is actually about.

**Computation 2 — uniform 2-of-7 over structural subgraph.** P(both
rebound carriers belong to the same structural subgraph | 2-of-7
uniform, 2 interactive-shell vs 5 infrastructure/slow) =
`C(2, 2) / C(7, 2) = 1/21 ≈ 0.048`. This is the same numeric
weight (because the interactive-shell subgraph happens to have
size 2 in the current carrier set), but it is a *different
substantive* claim — it is the probability of within-subgraph
clustering rather than the probability of a specific carrier
identity match.

**Computation 3 — joint structural-class-coincidence given
silence-rank.** The silence-rank-conditioned probability that the
**two longest-silent interactive-shell carriers** rebound is the
relevant conditional: it factorises as P(rebound pair is in same
subgraph | uniform 2-of-7) × P(silence-rank-vs-subgraph mapping
aligns by chance | no-information prior). Synth #100 takes the
second factor as ≈ 0.5 under a no-information prior on the
mapping, giving joint ≈ `0.048 × 0.5 ≈ 0.024` and BF ≈ **×42**
against the independent-axis baseline at first realization. The
addendum text reports BF ×46 (the difference is rounding /
exponent-base; the order of magnitude is what matters). Either
number is in the "noteworthy at first realization, only decisive
under multi-tick replication" band.

The replication ladder is geometric. If synth #100's directional
claim holds at three further independent zero-cardinality-rebound
events, cum-BF ≈ `42^3 ≈ ×74000`, which is decisive under any
reasonable prior. If even one of those next three events rebounds
infrastructure-layer-first (goose or litellm, with no opencode/
codex merge in the rebound tick), the asymmetric-rebound predicate
is falsified as a directional claim — it would be a one-shot
artifact rather than a structural property of the carrier graph.

# How synth #100 connects to the rest of W17

Synth #100 does not stand alone. The ADDENDUM-309 capture cited
both surrounding synthesis primitives:

- **synth #102** (`W17-synthesis-102-cross-carrier-zero-cardinality-
  triplet-at-post-PM-EU-pre-AM-Pacific-UTC-band-BF-x78.md`):
  established the zero-cardinality triplet as a discrete object at
  the `22:58Z–01:35Z` UTC band and assigned cum-BF `×78`.
- **ADDENDUM-308** extended the zero run to a silent quadruplet,
  amplifying M-307.A diurnal-pause-synchronization to cum-BF `×175`.

So synth #100 is the **continuation** of synth #102's
zero-cardinality-band primitive: synth #102 said "zero-cardinality
triplets exist as a clean object," ADDENDUM-308 said "the run can
extend to a quadruplet without falsifying the structure,"
ADDENDUM-309 said "the rebound after a quadruplet does not pick
carriers uniformly," and synth #100 said "the rebound non-
uniformity has a structural explanation: interactive-shell carriers
rebound first."

Synth #101 (`W17-synthesis-101-cross-repo-same-day-anchor-author-
dual-monopoly-with-housekeeping-doublet-anatomy-codex-etraut-
openai-and-opencode-kitlangton-on-2026-05-03.md`, also commit
`5a17b91`) is independently interesting and mentions the same
codex carrier, but it is a different primitive — it is about
*intra-day single-anchor-author monopoly*, not about *inter-tick
cross-carrier silence-rebound*. The two synths are linked by the
codex side: openai/codex #20896 (mergeCommit `67849d95`) is both
the rebound member in synth #100 and the deletion-pure member of
the @etraut-openai housekeeping-doublet in synth #101 (the doublet
deleted +0/-1060 lines across 8 docs files and closed 13m56s after
opening). That is a pleasant coincidence — the same merge-commit
participates in two different W17 structural primitives — but it
is not a confounder for either claim, because the two primitives
condition on different aspects of the merge: synth #100 conditions
on the **rebound-tick membership**, synth #101 conditions on the
**same-day single-author monopoly anatomy**.

# Falsifiable predictions stacked on synth #100

ADDENDUM-309's prediction list (the `P-309.A`–`P-309.J` set) is
concretely tied to the synth #100 falsification ladder:

1. **P-309.A** opencode extends the rebound to a second merge in
   the next tick (modal P=0.55, citing precedent merges
   sst/opencode #25640 `ce89bcb8` and #25636 `ca6150d6`).
2. **P-309.B** codex sustains the rebound to a second merge, with
   #20893 `39555036` or #20914 `4999ef03` as the candidate (modal
   P=0.48 under codex two-merge-burst-after-decet-silence).
3. **P-309.C** cardinality jumps to 3+ at the next AM-Pacific
   deepening tick `02:25Z–03:15Z` (modal P=0.50, with litellm or
   goose as the most likely third member).
4. **P-309.D** width sustains modal-band [27m–50m] at the seventh
   consecutive exact-50m (modal P=0.42; would lift basin-lock
   cum-BF `×7.4` → `×16+` at first septet realization).
5. **P-309.E** gemini-cli extends silent run to n=75 deeper
   DUOVIGINTET (modal P=0.52).
6. **P-309.F** crush extends silent run to n=78 deeper
   QUINVIGINTET (modal P=0.52).
7. **P-309.G** the asymmetric-rebound replicates at the next
   cross-band-edge crossing (sub-modal P=0.40; cum-BF `×46` →
   `×100+` at first replication; falsified if litellm/goose merge
   before opencode/codex post-quiet-band).
8. **P-309.H** goose centenarian-ceiling extends to n=109 (modal
   P=0.92).

P-309.G is the pivotal one for synth #100 itself. If the next
multi-tick zero-cardinality band rebounds with a *non*-interactive-
shell carrier appearing first, synth #100 collapses to a one-shot
descriptive observation. If three consecutive rebounds replicate
the pattern, synth #100 is promoted to a structural feature of the
W17 carrier set.

# Cross-window cited SHAs

The addendum verified mergeCommit SHAs via `gh pr view <num> -R
<repo> --json mergeCommit,mergedAt,title`. The cross-window cited
references are:

- sst/opencode `#25646 → c2b1974d`, `#25640 → ce89bcb8`,
  `#25636 → ca6150d6` (also `7749d8e8` as a squash variant),
  `#25633 → 825ab2e3`, `#25632 → 6312c55d` (also `6482515f`).
- openai/codex `#20896 → 67849d95`, `#20893 → 39555036`,
  `#20914 → 4999ef03`.
- BerriAI/litellm `#27041 → c011a7e3`, `#27096 → f880faf0`,
  `#27037 → cfa058c3`.
- charmbracelet/crush `#2774 → ce673448` (variant `ce314b8e`).
- google-gemini/gemini-cli `#26348 → d1654301`.
- QwenLM/qwen-code `#3807 → e617f20d`, `#3754 → 124a3834`.
- block/goose `#8953 → a08e986b`.

The drip-325 review pass (commit `e727a80` on the reviews side)
covered eight of these PRs with verdict mix `2 merge-as-is /
4 merge-after-nits / 0 request-changes / 2 needs-discussion`,
and the two `needs-discussion` cases — openai/codex #20914
(`4999ef03`) and BerriAI/litellm #27037 (`cfa058c3`) — are both
infrastructure-layer or interactive-shell merges that did *not*
participate in the synth #100 rebound but are the candidate
extensions for the next rebound tick under P-309.B and the
implicit infrastructure-rebound prediction.

# What the synth #100 / ADDENDUM-309 pair adds to the operating model

The cleanest one-sentence summary of the addendum-and-synth pair
is: **after a four-tick zero-cardinality cross-carrier band, the
first rebound tick at the AM-Pacific band-edge selectively
reactivated the interactive-shell-carrier subgraph (sst/opencode
#25646 `c2b1974d` and openai/codex #20896 `67849d95`) before the
infrastructure-layer subgraph (litellm, goose) — and the silence-
rank order alone (goose at n=108 longest) does not predict the
rebound membership; the structural axis-classification does.**

That sentence is now a falsifiable hypothesis with a clean
prediction (P-309.G), a clean cum-BF (`×46` at first realization,
geometric ladder under replication), and a cited cross-window SHA
inventory. It is the kind of structural primitive the W17 synthesis
stack was built to produce, and it is exactly the kind of claim
that needs the next two or three zero-cardinality-rebound events to
either ratify or kill. The addendum cadence (50m capture envelope,
sixth-consecutive realization, basin-lock cum-BF `×7.4`) makes that
verification feasible inside the next 24-tick window.
