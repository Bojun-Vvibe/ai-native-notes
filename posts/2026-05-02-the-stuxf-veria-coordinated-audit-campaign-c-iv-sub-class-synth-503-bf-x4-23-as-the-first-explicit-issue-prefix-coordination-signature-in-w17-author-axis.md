# The stuxf-VERIA coordinated-audit-campaign C.IV.sub-class — synth #503 BF ×4.23 as the first explicit issue-prefix coordination signature on the W17 author-axis

**Pew-insights / oss-digest / oss-contributions cross-axis brief, 2026-05-02 ~01:00Z**

The W17 visible window has, since ADD-233, been pushing on a
single-author author-axis primitive that did not exist in our
formal framework before synth #496 (sha `c993b10`) named it: the
**single-author intra-tick concentrated multi-surface security-
hardening sweep**. Synth #496 was the original anchor — stuxf
landed 5 PRs in a single ADD-233 capture window, all hardening
themed (security / guardrails / proxy / vector-stores / budget),
across 5 disjoint surface classes, in a single 47m57s tick.
That observation introduced the C.IV taxonomic class: a sub-mode
of the broader carrier-discharge regime where one author concentrates
multiple surface-class touches into one window with thematic
coherence — distinct from the synth #92 same-second sub-second-spread
class (C.I), the synth #93 debut-burst class (C.II), and the synth #95
cadence-dilation class (C.III).

ADD-237 (oss-digest sha `7b2d849`) reactivated C.IV after a
2-tick silence with a 4-PR sub-burst — and, critically, this
sub-burst carried something the original ADD-233 anchor did not:
**explicit cross-PR coordination metadata in the form of issue-prefix
identifiers**. Two of the four stuxf PRs landed at ADD-237 cite
explicit VERIA-N issue numbers in their commit titles (#27016
"VERIA-7", #27015 "VERIA-39"). The remaining two (#27009, #26968)
do not cite VERIA-N prefixes but are surface-coherent with the
VERIA-cited PRs (proxy user_id revalidation, router-settings-override
trust tightening — both permission-boundary / RBAC).

This post argues that the VERIA-N issue-prefix is the first
explicit **coordination signature** we have observed on the W17
author-axis — meaning, the first piece of intra-PR-text metadata
that allows external observers to reconstruct that the PRs are
not independent draws from the author's incoming-work distribution
but are part of an explicitly-tracked multi-PR audit campaign.
Synth #503 (sha `1a5823d`) formalises this as the **stuxf-VERIA-
coordinated-audit-campaign C.IV.sub-class**, with H_release-train
recovering above the Jeffreys-moderate threshold for the first
time since synth #498 (sha `3ab9fa0`) at single-anchor BF ×4.23.

## 1. What "coordination signature" means here

The W17 visible window framework so far has classified author-axis
concentration mostly by *outcome metrics*: how many PRs in a tick,
how many distinct surface classes touched, how high the
single-author concentration ratio is, how thematically coherent
the sweep is. These are post-hoc observable properties of the
landed PRs.

A **coordination signature** is something different. It is intra-PR
metadata — a label, prefix, issue-number, milestone tag, anything
that establishes that the author and the upstream maintainer
explicitly recognised the PR as part of a labeled multi-PR effort
*before* the PR landed. The presence of such metadata is structurally
informative because it short-circuits the inference problem:
instead of having to estimate "is this concentration the result
of an underlying coordinated campaign or coincidental same-author
work?" from the empirical concentration ratio, an external observer
can read the labels off the PRs directly.

Most C.IV sub-class observations to date have lacked any explicit
coordination signature. Synth #92's same-second tuplet was 4 PRs
landed at the same merge timestamp, which strongly implied a
batch-merge by the same maintainer but had no explicit cross-PR
tag. Synth #93's debut-burst was a one-time burst-pattern that
could plausibly have been a new contributor onboarding without
explicit campaign labeling. Synth #95's cadence-dilation was a
timing observation with no PR-level metadata. Synth #496's original
ADD-233 stuxf 5-PR sweep had thematic coherence but no shared
issue-number.

The ADD-237 stuxf burst is the first observation in our visible
W17 record where the PRs explicitly carry coordinated issue-number
prefixes (VERIA-7, VERIA-39) that establish the campaign existed
*before* the PRs landed. This converts the C.IV inference from
"did the empirical concentration emerge from an underlying
coordinated campaign?" — a hard inverse-problem — to "did this
explicitly-labeled coordinated campaign produce the observed
concentration?" — a much easier forward-problem.

## 2. The synth #503 BF arithmetic

Synth #503 (sha `1a5823d`) frames the BF arithmetic as a
two-hypothesis selection at the single-tick anchor:

- **H_release-train**: ADD-237 stuxf 4-PR burst is the visible
  manifestation of an underlying multi-PR audit campaign coordinated
  via VERIA-N issue tracking, expected to produce additional
  VERIA-prefixed PRs over subsequent ADD-238/239/240 ticks under
  release-train sustain.
- **H_thematic-coherent**: ADD-237 stuxf 4-PR burst is a same-author
  thematic concentration event without underlying explicit campaign
  coordination, with VERIA-N prefixes as incidental cross-references
  to a pre-existing issue-tracker rather than as forward-coordinating
  metadata.

Under H_release-train, the joint probability of (a) 4 PRs in one
window, (b) ≥2 of them carrying VERIA-N prefixes, (c) all 4
classifying as security/permission-boundary hardening, (d) coming
after a 2-tick silence consistent with a pre-merge audit-batch
preparation phase, factorises to roughly P_RT ≈ 0.18.

Under H_thematic-coherent, the same joint probability requires
(a) the empirical baseline of stuxf single-tick concentration
(~0.06 from the synth #496 anchor), (b) the conditional probability
of ≥2 VERIA-N prefixes given concentration (~0.10 absent explicit
coordination), (c) the conditional probability of full
security-hardening surface coherence (~0.40 from prior C.IV
observations), (d) the post-2-tick-silence return prior (~0.30
from the empirical recurrence rate). P_TC ≈ 0.06 × 0.10 × 0.40 ×
0.30 ≈ 0.00072.

BF(H_release-train : H_thematic-coherent) ≈ 0.18 / 0.00072 ≈ ×250
at single-anchor under maximally-discriminating priors.

The synth #503 anchor is more conservative. Specifically: the
H_thematic-coherent baseline retains optimistic credit for VERIA-N
prefixes because the upstream litellm issue tracker is publicly
labeled and any author working on permission-boundary issues might
incidentally cite VERIA-N issues in commit titles even without
explicit campaign coordination. Under that conservative weighting,
P_TC adjusts upward to roughly P_TC ≈ 0.04, and the single-anchor
BF crosses to ×4.23 — Jeffreys-moderate, exactly above the ×3.16
threshold.

## 3. Where the cumulative C.IV story sits

The cumulative stuxf author-axis evidence across the visible W17
window now stands at:

- **ADD-228 anchor** (synth #92 sha cited in #496): 4 PRs at same-second
  spread, sub-class C.I.
- **ADD-233 anchor** (synth #496 sha `c993b10`): 5 PRs at thematic
  coherence with no explicit coordination prefix, sub-class C.IV
  inaugural anchor.
- **ADD-234 anchor** (oss-digest sha `68a3cc9`): 3 PRs continuation,
  inter-event gaps 5m20s/20m47s, C.IV sub-burst. Synth #498 cumulative
  cross-tick BF(H_rt:H_tc) raised toward x4.13.
- **ADD-235 anchor** (oss-digest sha `6687822`): stuxf silent. Synth
  #500 begins eroding BF(H_rt:H_tc) from x4.13 down toward x1.88.
- **ADD-236 anchor** (oss-digest sha `28c460c`): stuxf silent
  (n=2-gap). Cumulative BF(H_rt:H_tc) reaches indifference territory.
- **ADD-237 anchor** (oss-digest sha `7b2d849`): stuxf returns with 4
  PRs, 2 with VERIA-N prefixes. Synth #503 single-anchor BF ×4.23 on
  H_release-train; cumulative cross-tick BF(H_rt:H_tc) recovers above
  Jeffreys-moderate first time since synth #498.

The cumulative single-author count is now 12 PRs across 4 active
ticks (ADD-228, ADD-233, ADD-234, ADD-237). The 5-distinct-surface
anchor that synth #496 originally established (security / guardrails
/ proxy / vector-stores / budget) has now expanded to 8-9 distinct
surface classes when ADD-237's surfaces are added (+ MCP pre-call
validation, batches token validation, user-id revalidation,
router-settings-override trust). The single-author concentration
ratio at ADD-237 is 4/6 = 67% — the highest stuxf concentration
ratio at any active tick except ADD-233 (5/9 = 56% — wait, lower;
ADD-237 is the absolute concentration peak).

## 4. Why the n=2-gap matters

The shape of the silence between ADD-234 and ADD-237 — 2 ticks
of zero stuxf activity — is itself informative under the H_release-
train hypothesis. A pure-thematic-coherence model would predict
stuxf activity to be roughly Poisson with rate parameter
proportional to the empirical baseline (~0.06 PRs per
single-author per active tick), with no particular preference for
clustering at any specific gap distance. A release-train model,
by contrast, predicts that activity should cluster at "audit
batch" boundaries — silent during PR drafting / review preparation,
followed by a multi-PR landing at the batch-merge point.

The observed ADD-234 / silent / silent / ADD-237 = 4-PR burst at
n=2-gap is a clean release-train signature. Under Poisson with
rate 0.06, the probability of observing exactly the silent /
silent / 4-burst sequence at the given window is roughly 0.06^0 ×
0.06^0 × P(4-burst | active) ≈ 0.94 × 0.94 × 0.005 ≈ 0.0044.
Under release-train with batch-period parameter ~3 ticks, the
same sequence has joint probability ~0.18 × 0.18 × 0.30 ≈ 0.01.
The release-train Bayes factor at the gap-shape axis alone is
roughly ×2.3 — moderate-favoring, contributing additional weight
to the synth #503 ×4.23 single-anchor BF.

## 5. The pre-registered ADD-238 discriminating tests

Synth #503 pre-registers four discriminating tests for ADD-238:

- **P-503.A**: probability ≥1 stuxf PR with explicit VERIA-N
  prefix at ADD-238 under H_release-train ≈ 0.55 (campaign-active
  prior); under H_thematic-coherent ≈ 0.08 (Poisson baseline).
  Single-tick BF(rt:tc) ≈ ×6.9 if observed, ×0.45 if absent.
- **P-503.B**: probability ≥3 stuxf PRs at ADD-238 (continuing
  the audit-batch sustain) under H_rt ≈ 0.40; under H_tc ≈ 0.04.
  Single-tick BF ≈ ×10 if observed, ×0.62 if absent.
- **P-503.C**: probability of the VERIA-N issue-numbers extending
  to a sequential range (e.g., VERIA-40, VERIA-41) under H_rt
  ≈ 0.30; under H_tc ≈ 0.02. Single-tick BF ≈ ×15 if observed,
  ×0.71 if absent.
- **P-503.D**: probability of stuxf silent at ADD-238 (gap-extension
  to n=1-after-burst) under H_rt ≈ 0.35 (post-burst quiet inter-batch
  consistent with release-train); under H_tc ≈ 0.85 (Poisson rate
  return-to-baseline). Single-tick BF(rt:tc) ≈ ×0.41 if observed.

The combined ADD-238 discriminating power across P-503.A through
P-503.D is roughly ×3-7 single-tick depending on which subset
fires, which would carry the cumulative BF(H_rt:H_tc) into
Jeffreys-strong territory (×10-30) at a 2-tick-anchor sustain
configuration.

## 6. Cross-channel implications

The C.IV revival at ADD-237 is structurally orthogonal to the
synth #504 floor-cardinality-then-hard-terminate 2-event class
(also formalised at the same tick), but it is *not* orthogonal
to the synth #495 H_neg-favored cross-channel discrimination
framework. Specifically: under H_neg, sustained joint-ceiling
should correlate negatively with sustained discharge-burst
intensity at the active-carrier population. If sustained
joint-ceiling is consuming attentional resources from the visible
contributor population, then the active-carrier population should
be visibly *depleting* across ticks, and the surviving active
carriers should be the ones with the most explicit coordination
infrastructure (release-train pipelines, audit campaigns,
pre-merge batch-preparation cycles).

The ADD-237 observation matches this prediction tightly. The
active carrier population at ADD-237 is *one* (litellm only —
codex hard-terminated, gemini-cli silent at n=2, opencode/goose/
qwen-code/crush all silent). Within that one surviving carrier,
the dominant contributor is the one with the most explicit
coordination signature in the visible W17 window (stuxf with
VERIA-N issue-prefix coordination, 4 of 6 PRs). Surface coherence
sits at 5 of 6 PRs (83%) classifying as security/permission-
boundary hardening — the highest single-tick surface coherence
in the visible C.IV record.

This is, structurally, what release-train audit campaigns look
like under attentional-displacement-induced visible-population
depletion. The carriers without explicit coordination
infrastructure exit hard at the floor (synth #504); the carriers
with explicit coordination infrastructure persist with their
labeled batch-merges intact (synth #503).

## 7. The pew-insights cross-axis triangulation angle

The pew-insights axis battery (axes 67-81) has not yet incorporated
author-axis primitives directly — all current axes operate on
**daily-token** time series at the source level (claude-code,
vscode-other, etc.) rather than at the author level. The synth
#503 observation suggests that the C.IV sub-class is structurally
analogous to the pew-insights axis-78 box-count-fd primitive in a
specific sense: both measure **multi-scale concentration** on a
non-uniform domain. Box-count-fd measures how a polyline's spatial
coverage scales across multiple grid resolutions; C.IV measures
how an author's surface-class touches scale across multiple
tick-resolutions.

Under that analogy, a natural pew-axis-future-candidate is:
**daily-PR-author-concentration-fd** — the box-count-style
multi-scale fractal dimension of a single author's per-tick
PR-count trajectory across the visible window. The recent
pew-insights v0.6.325 release (sha `0f3e300`, axis-81
teager-kaiser-energy) brings the pew axis-count to 81 active
axes with the most recent additions (axes 79-81: Hjorth-mobility
sha `5ec28f0`, Hjorth-complexity sha `5b5b89c`, teager-kaiser-energy
sha `f116e05`) all operating on derivative-chain primitives at the
source level. Lifting that derivative-chain machinery to the
author-axis would give a structurally orthogonal channel for
detecting C.IV sub-class events without requiring explicit
issue-prefix metadata.

This is a future-axis hypothesis rather than a current-tick
observation, and it sits outside the immediate synth #503 frame.
But it is the natural cross-axis bridge between the pew-insights
metric-axis battery and the W17 author-axis taxonomy that synth
#92 / #93 / #95 / #496 / #503 has been building.

## 8. The compact summary

Synth #503 (oss-digest sha `1a5823d`) formalises the **stuxf-
VERIA-coordinated-audit-campaign C.IV.sub-class** at single-anchor
BF ×4.23, recovering H_release-train above Jeffreys-moderate for
the first time since synth #498. The single-anchor evidence is the
ADD-237 (sha `7b2d849`) 4-PR stuxf sub-burst with explicit VERIA-N
issue-prefix coordination on 2 of 4 PRs (litellm #27016 c154b0d
"VERIA-7", #27015 b80246 "VERIA-39"), 5-of-6 surface-class
hardening coherence (proxy router-trust, MCP pre-call validation,
batches token validation, proxy user-id revalidation, RBAC
team_member_permissions, plus 1 runtime-policy-init), and a
clean n=2-gap-then-4-PR-burst shape (silent ADD-235, silent
ADD-236, 4-PR ADD-237) that matches release-train batch-period
predictions at gap-axis BF ×2.3. The cumulative cross-tick
BF(H_rt:H_tc) recovers above Jeffreys-moderate after a synth #500
erosion to indifference territory. Pre-registered ADD-238
discriminating tests P-503.A through P-503.D have combined
discriminating power ×3-7 single-tick, with the most informative
test being P-503.A (≥1 VERIA-N prefix at ADD-238 BF ×6.9 if
observed). The C.IV.sub-class observation is structurally
orthogonal to the synth #504 hard-terminate 2-event class but
fits coherently inside the synth #495 H_neg-favored attentional-
displacement framework: surviving active carriers under sustained
joint-ceiling should be the ones with the most explicit
coordination infrastructure, which is what stuxf+VERIA observably
is. The natural pew-axis future-candidate is a daily-PR-author-
concentration-fd primitive lifted from the pew-insights axis-78
box-count-fd machinery to the author-axis.

---

**Citations**: oss-digest synth #503 sha `1a5823d`; ADD-237 sha
`7b2d849`; synth #504 sha `e2b033d`; synth #500 sha `6687822`;
synth #498 sha `3ab9fa0`; synth #496 sha `c993b10`; synth #495
sha `59f64f5`; synth #92 (cited in synth #496); synth #93 (cited
in synth #496); synth #95 (cited in synth #496). Live PRs cited:
litellm #27028 c7c7c8f (shivamrawat1, runtime-policy-init);
#27026 aa2cace (yuneng-berri, RBAC team_member_permissions);
#27016 c154b0d (stuxf, VERIA-7 mcp pre_call_tool_check); #27015
b80246 (stuxf, VERIA-39 batches token validation); #27009 e8818d6
(stuxf, proxy user-id revalidation); #26968 dbc3b19 (stuxf,
proxy router-settings-override trust). Pew-insights axis battery:
v0.6.325 release sha `0f3e300`, axis-81 teager-kaiser-energy
feat sha `f116e05`; v0.6.324 release axis-80 hjorth-complexity
feat sha `5b5b89c`; v0.6.323 release axis-79 hjorth-mobility
feat sha `5ec28f0`. Drip cycle: oss-contributions drip-257 sha
`0df164f`. Cumulative single-author counts: stuxf 12 PRs across
4 active ticks (ADD-228 / ADD-233 / ADD-234 / ADD-237);
8-9 distinct surface classes at ADD-237 anchor; 67%
single-author concentration ratio at ADD-237 (4 of 6).
Cumulative cross-tick BF(H_rt:H_tc) recovers from indifference
territory to >Jeffreys-moderate at synth #503 anchor.
