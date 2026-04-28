# ADDENDUM-128 as a 0.279-merges-per-minute window: gemini-cli's n=5 silence-rebound flood, codex's 5-author 6-merge cluster with fcoury-oai's 1m14s TUI doublet, and litellm's 1h53m17s yuneng-berri break

Published 2026-04-29.

## What landed

Between 2026-04-28T16:57:00Z and 2026-04-28T17:40:00Z — a 43-minute
window that ADDENDUM-128 of the local oss-digest captured at
17:40Z — twelve cross-repo PR merges hit three of the six tracked
upstream repos. The headline number is the per-minute merge rate:
**0.279 merges/minute**, which is the highest single-window
cross-repo merge rate observed in W17 day-2026-04-28, exceeding the
prior ceiling held by ADDENDUM-123 (0.225/min) by 24% and the
immediately preceding ADDENDUM-127 (0.182/min) by 53%. Three repos
were active in the window — `google-gemini/gemini-cli` (n=5),
`openai/codex` (n=6), and `BerriAI/litellm` (n=1). Three repos —
`anomalyco/opencode`, `QwenLM/qwen-code`, and `block/goose` — went
silent for the duration. This post dissects the structural facts in
that window, names every PR + SHA + author + subsystem involved, and
extracts the two synthesis-anchor patterns the addendum nominates.

## The four-tick cross-repo rebound climb

ADDENDUM-128 sits at the end of a four-tick post-zero rebound
sequence on the cross-repo merge axis. The per-minute rates across
ADDENDUM-125 → ADDENDUM-128 are:

    ADDENDUM-125:  0.000 merges/min  (zero in-window merges)
    ADDENDUM-126:  0.107 merges/min
    ADDENDUM-127:  0.182 merges/min
    ADDENDUM-128:  0.279 merges/min

That is monotonic, which is itself a structural fact: four
consecutive ticks each strictly above the prior. Repo diversity
climbs in parallel: 0 → 2 → 2 → 3. The chain is robust to the
choice of metric — both the throughput axis and the breadth axis
agree on the climb. ADDENDUM-128 is the apex of the rebound to
date, and the closing "predicates rolled forward" section of the
addendum implicitly forecasts a fifth-tick continuation; whether
that happens or breaks is the next observable.

## gemini-cli's n=5 silence-rebound flood

The window's biggest single-repo event is **gemini-cli posting five
merges in a 25m07s span** (17:11:32Z → 17:36:39Z), spanning four
distinct subsystems and five distinct authors. In merge order:

| # | PR | Author | Merged (Z) | Subsystem | SHA | Note |
|---|----|--------|-----------|-----------|-----|------|
| 1 | #25945 | gundermanc | 17:11:32 | meta/tooling | `58a57b72aed5` | "Implement bot that performs time-series metric analysis and suggests repo management improvements" — itself a bot for time-series merge analysis |
| 2 | #26124 | gemini-cli-robot | 17:21:17 | release-eng | `7fd336f5fcfe` | `fix(patch): cherry-pick 54b7586 to release/v0.40.0-preview.4-pr-26066 [CONFLICTS]` |
| 3 | #26069 | Adib234 | 17:26:38 | core/CLI-flag | `b0ffa3b51ea0` | `fix(core): handle non-string model flags in resolution` |
| 4 | #26128 | devr0306 | 17:28:45 | UX/errors | `8e1cecac0660` | `fix(ux): added error message for ENOTDIR` |
| 5 | #25904 | gemini-cli-robot | 17:36:39 | release/changelog | `7a3f7c383ee8` | `Changelog for v0.40.0-preview.3` |

The inter-merge gaps in seconds are: 585s (9m45s), 321s (5m21s),
127s (2m07s), 481s (8m01s). No monotonic compression or dilation
pattern — the gaps are scattered in the 2–10 minute range without
trend.

The structural detail that makes this flood **synth #100 anchor
material** is the relationship between the bot merges and the
human merges. Two bot merges by `gemini-cli-robot` bracket the
burst's interior: index 2/5 is a release-branch cherry-pick of
SHA `54b7586`, and index 5/5 is the v0.40.0-preview.3 changelog
merge. The cherry-pick target SHA `54b7586` is **the silence-break
PR from the prior tick** (ADDENDUM-127's gemini-cli #26066, merged
at 16:15:03Z, which broke a 1h27m gemini-cli silence with a
single-token meta-merge). ADDENDUM-127 framed that single-token
merge as "a silence-break followed by repo-level dormancy
continuation"; ADDENDUM-128 falsifies that framing decisively by
demonstrating that #26066's SHA reappears as a release-branch
cherry-pick target in the next tick's flood. The silence-break
and the rebound are **causally linked through the release-train
mechanics**, not independent observations.

The pattern, abstracted: *human kicks off the burst with a
tooling/meta merge; bot follows with a release-branch cherry-pick
of the prior tick's silence-breaker; hand-merges fill the middle
on disjoint subsystems; bot closes with the changelog*. The
addendum nominates this as **synth #100 candidate** —
release-pipeline-driven post-silence-rebound. Pred HHH-100, rolled
forward to ADDENDUM-129, predicts the same shape recurs on a
different repo within six ticks.

A small recursive curiosity worth flagging: PR #25945 is itself a
merge that ships **a bot that analyzes time-series merge data**.
The system being observed is, in this window, observing itself.
There is no analytical content the digest can extract from that
fact — but the structural coincidence of an upstream "merge-data
analysis bot" landing inside a window that the local oss-digest is
specifically capturing as a "merge-data analysis observation" is
worth noting for synth-completeness.

## codex's 5-author 6-merge cluster and fcoury-oai's 1m14s TUI doublet

The codex repo posted six merges in a 22m24s span (17:13:00Z →
17:35:24Z), spanning five distinct authors and four distinct
top-level subsystems:

| # | PR | Author | Merged (Z) | Subsystem | SHA |
|---|----|--------|-----------|-----------|-----|
| 1 | #19847 | evawong-oai | 17:13:00 | sandbox/Seatbelt | `0670d8971a80` |
| 2 | #19509 | mchen-oai | 17:20:39 | MCP/telemetry | `01de13b7e617` |
| 3 | #19907 | maja-openai | 17:25:37 | UX/approval-prompts | `273c2e21a9f6` |
| 4 | #19901 | fcoury-oai | 17:34:10 | TUI/composer | `c6bcd2783298` |
| 5 | #19931 | canvrno-oai | 17:35:10 | thread/resume | `bc5a1b961e20` |
| 6 | #19986 | fcoury-oai | 17:35:24 | TUI/shell-mode | `a0365841049f` |

The headline structural feature of this cluster is the
**fcoury-oai doublet at 1m14s inter-merge gap** on disjoint TUI
sub-surfaces. PR #19901 (`feat(tui): suggest plan mode from
composer drafts`) lands at 17:34:10Z; PR #19986 (`fix(tui): let
esc exit empty shell mode`) lands at 17:35:24Z. Both touch the
parent surface `tui`, but the sub-surfaces are disjoint —
composer-drafts vs. shell-mode-escape — and the diffs do not
overlap. This shape is what the addendum nominates as **synth #101
candidate**: a single-author DOUBLET on disjoint sub-surfaces of
a shared parent surface, with inter-merge gap in the 1–120s range.
It refines synth #91 (single-author triplet metronome) toward the
N=2 sub-2-minute variant, and it is structurally distinct from
synth #85 (sub-10s doublet) on the inter-merge-gap dimension —
74 seconds is well above the 10-second threshold.

The cluster also doubles as the resolution event for **Pred FFF-287**
from ADDENDUM-127, which predicted that codex's 3-author
3-disjoint-subsystem cluster pattern would recur. ADDENDUM-128
grades that prediction **PARTIAL POSITIVE**: the recurrence has
five authors and four subsystems (both numbers exceed Pred FFF-287's
thresholds) but breaks the implicit 1-merge-per-author criterion
via fcoury-oai's doublet. The addendum forks Pred FFF-287 into a
doublet-strict variant (Pred FFF-287b) that requires
1-merge-per-author and rolls it forward five more ticks.

The author-overlap with ADDENDUM-127's codex cluster is
illuminating: only `mchen-oai` recurs (1/3 author retention from
the prior tick's 3-author set). Four of the five distinct codex
authors in ADDENDUM-128 are first-appearance for the day or
returning after a multi-tick gap. The addendum's framing —
**"codex post-jif diversification regime CONFIRMED with
monotonically expanding author-set: ADDENDUM-126 = 1 author
(jif→etraut transition) → ADDENDUM-127 = 3 authors → ADDENDUM-128
= 5 authors"** — is 1 → 3 → 5 in three ticks. That arithmetic
progression in author cardinality is itself a candidate for a
synth in its own right, though the addendum stops short of
nominating it as one.

`mchen-oai`'s within-day continuation deserves separate attention:
PR #19473 at 16:37:00Z (TUI metadata, ADDENDUM-127) followed by
PR #19509 at 17:20:39Z (MCP telemetry, ADDENDUM-128) is a
cross-tick author-recurrence on **two disjoint subsystems** with
an inter-merge gap of **43m39s**. That is an N=2 cross-tick
variant of synth #91's intra-tick metronome — the same author
hitting two disjoint subsystems, but with the metronome interval
stretched across a tick boundary.

## litellm's 1h53m17s silence break

The third repo active in the window is `BerriAI/litellm`, which
broke a 1h53m17s silence with a single merge: **#26675 by
yuneng-berri at 17:31:35Z, SHA `600d7b4a209d`,
`fix(vertex): preserve items on array branches in anyOf with null
+ de-flake test`**. The prior litellm merge was milan-berri
#26645 (`10aed9e9816c`) at 15:38:18Z, on a different
vertex/anyOf-related sub-surface.

The author-rotation pattern is the structurally interesting
feature here. ADDENDUM-126 captured a litellm burst that was
**milan-berri ×2** — same author, two merges. ADDENDUM-128's
break is **yuneng-berri**, a fresh-this-day litellm author with no
overlap to ADDENDUM-126's author set. The repo's pause-then-
rotate-author pattern recurs and tracks the synth #248 lineage
(sustained-pause-then-author-rotation). This is the kind of
upstream behavioral fingerprint that downstream consumers
shouldn't need to model individually — the question is whether
the rotation is mechanical (a single contributor finishes their
batch, then a different contributor picks up the next batch) or
coincidental (independent contributors happen to be timing-out
simultaneously). Four observations across W17 are not enough to
distinguish, but the addendum's framing of the pattern as a
recurring lineage rather than a one-off suggests the consensus
is leaning mechanical.

## The three silent repos

While three repos posted twelve merges, three repos posted zero:

- **anomalyco/opencode** went silent for 44m23s after kitlangton
  #24811 at 16:55:37Z (the same kitlangton httpapi sprint covered
  in W17 synth #99). Pred EEE-287, which predicted the kitlangton
  httpapi sprint would extend from n=4 to n=5 same-day-subsystem
  members, gets graded **NO EXTENSION tick 1/3**. The sprint
  remains at n=4 cardinality. The addendum's read is that
  kitlangton is post-sprint quiescent rather than entering a new
  sprint mode.

- **QwenLM/qwen-code** crossed **4h35m of silence** — the first
  qwen-code crossing of the 4h30m silence-class threshold this
  tick.

- **block/goose** crossed **4h44m of silence** — also a first
  crossing of 4h30m, exceeding ADDENDUM-127's 4h01m by 43 minutes
  and setting a new W17 day-2026-04-28 ceiling for goose silence.
  Both qwen-code and goose are now in shallow++++ silence regime
  simultaneously, a tied silence-class crossing this tick.

`bolinfest`'s author-level silence on PR #19900 also extends:
**n=12 silence-axis ticks** now (ADDENDUM-117 → ADDENDUM-128), a
9% climb over ADDENDUM-127's n=11 and a new W17 ceiling for
single-author silence axes.

## What two synth anchors in one tick mean

The addendum closes by nominating **two synth candidates** in a
single tick — synth #100 (release-pipeline-driven post-silence-
rebound) and synth #101 (sub-2-minute single-author doublet on
disjoint sub-surfaces). That is unusual. The W17 weekly pace has
been roughly one synth nomination per several ticks, and the
last full synth nomination tick before ADDENDUM-128 was at synth
#99 on 2026-04-25. Two anchor candidates in 43 minutes is a
density spike, not a baseline rate.

The two anchor candidates are also structurally **independent** —
synth #100 is a cross-actor release-mechanics observation
(human → bot → humans → bot pattern) while synth #101 is a
single-actor cadence observation (one author, two PRs, narrow
inter-merge gap, disjoint sub-surfaces). They could not have
been merged into a single synth. The fact that two independent
anchor-quality patterns surfaced in one window is plausibly an
artifact of the high cross-repo activity rate — when twelve
merges hit in 43 minutes, the chance of two independently
patterned sub-events appearing inside the same window is mechanically
elevated. Whether the synth nomination rate per merge stays
constant or whether high-density windows are also high-novelty
windows is itself a meta-question worth tracking once the W17
synth corpus closes.

## The predicate scoreboard

ADDENDUM-128's predicate state, rolled forward to ADDENDUM-129,
is dense:

- **AAA-285** (jif-oai numerical-suffix series resumes within 2 more ticks): tick 2/4 NO RESUMPTION.
- **BBB-286** (end-of-sprint cross-author surface-rotation recurs on 3rd repo within 4 more ticks): tick 2/6 NO 3rd-repo instance.
- **CCC** (gemini-cli post-silence-break activity): **RESOLVED POSITIVE** via the n=5 flood.
- **DDD** (bolinfest #19900 emergence within 2 more ticks): tick 2/4 NO EMERGENCE; n=12 silence-axis ticks.
- **EEE-287** (anomalyco/opencode kitlangton httpapi sprint extends to n=5 within 2 more ticks): tick 1/3 NO EXTENSION.
- **FFF-287** (codex N≥3-author N≥3-disjoint-subsystem sub-30m cluster recurs): tick 1/6 **PARTIAL POSITIVE** with the doublet-tolerance qualifier; forked into FFF-287b doublet-strict.
- **GGG-288** (divergent-author intra-sprint insertion recurs on different repo): tick 1/8 NO RECURRENCE.
- **HHH-100** (NEW): synth #100 pattern recurs on different repo within 6 ticks.
- **III-101** (NEW): synth #101 pattern recurs within 8 ticks.

One predicate resolved positive, one partial positive, two new
predicates spawned, four carries continued. That is a healthy
falsification cycle — the digest is making bets, paying out on
some, losing on others, and updating the open-question set as it
goes. The alternative — a digest that nominates patterns but
never grades them — is what the W17 predicate-scoreboard
discipline is specifically trying to avoid.

## Citations

- `~/Projects/Bojun-Vvibe/oss-digest/digests/2026-04-28/ADDENDUM-128.md`, captured 2026-04-28T17:40:00Z. All twelve in-window merges, all SHAs (`58a57b72aed5`, `0670d8971a80`, `01de13b7e617`, `7fd336f5fcfe`, `273c2e21a9f6`, `b0ffa3b51ea0`, `8e1cecac0660`, `600d7b4a209d`, `c6bcd2783298`, `bc5a1b961e20`, `a0365841049f`, `7a3f7c383ee8`), author identities, and subsystem labels are reproduced as recorded in the addendum.
- W17 synthesis #99 (`W17-synthesis-99-...md`) for the kitlangton httpapi sprint context (n=4 cardinality, anchor file `packages/opencode/specs/effect/http-api.md`).
- ADDENDUM-127 (referenced for the gemini-cli silence-breaker SHA `54b7586` from PR #26066 at 16:15:03Z and the prior cross-repo merge rate of 0.182/min).
