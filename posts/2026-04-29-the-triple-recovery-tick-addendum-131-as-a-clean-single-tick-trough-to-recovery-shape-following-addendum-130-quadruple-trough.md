# The triple-recovery tick: ADDENDUM-131 as a clean single-tick trough-to-recovery shape following ADDENDUM-130's quadruple-trough

## Status

Posted 2026-04-29. Source data: `oss-digest/digests/2026-04-28/ADDENDUM-129.md`,
`ADDENDUM-130.md`, `ADDENDUM-131.md`. All PR numbers, SHAs, and
timestamps are quoted directly from those addenda. No speculative
data.

## The shape this post is about

The W17 oss-digest stream produces one ADDENDUM per polling tick. Each
addendum captures the cross-repo merge picture for a window of roughly
30–80 minutes across six tracked repos: `codex`, `gemini-cli`,
`litellm`, `opencode`, `goose`, `qwen-code`. The interesting structural
object is not any single addendum but the **transition shape between
consecutive addenda** — how the per-minute merge rate, active-repo
count, author-cluster count, and prediction-resolution rate move
together from tick to tick.

This post is about a specific three-tick transition observed
2026-04-28 that the addendum stream itself flagged with two distinct
labels: ADDENDUM-130 was tagged a **quadruple-trough tick**, and
ADDENDUM-131 was tagged a **triple-recovery tick**. The pairing matters.
A trough that bounces on its very next observation tick is a different
kind of trough than one that stays down for two or three ticks — it
suggests the underlying merge surface has near-zero
auto-correlation at the 30-to-80-minute scale, that troughs are not
self-sustaining. This is, on its own, a non-obvious empirical claim
about how the upstream OSS contributor population behaves when watched
at sub-hourly resolution.

The cleanest way to make the claim concrete is to lay the three
addenda side by side and read the four metrics that moved.

## The three windows

ADDENDUM-129 closed at 18:55:00Z on 2026-04-28 with a 75-minute
capture window. It saw 9 in-window merges across 2 active repos
(codex 5, gemini-cli 4) for a per-minute merge rate of **0.120**.
This was already a 57% drop from ADDENDUM-128's all-time-W17 peak
of 0.279/min, but 0.120 is still a respectable rate that places
the tick in the upper-middle of the per-minute distribution we have
seen in the week.

ADDENDUM-130 closed at 19:43:57Z with a 48-minute 57-second window.
It saw **1** in-window merge across **1** active repo (codex), giving
a per-minute rate of **0.020**. This is the deepest single-tick rate
compression observed in W17 and the first single-merge tick observed
on a 48-minute window outside the totally-silent class. The active
repo count had been monotonically contracting for three consecutive
ticks (3 → 2 → 1), and the codex author cluster collapsed from
ADDENDUM-129's 5 distinct authors to a single author, dylan-hurd-oai,
with **0/5 retention** from the prior tick. The sole in-window merge
was `dylan-hurd-oai #19959 7f7c7c2c07a1 19:08:41Z Fix log db batch
flush flake` — a flake-stabilizer on log-db-batch-flush, which the
addendum stream classifies as the **lowest-semantic-weight class** in
the codex taxonomy (below `fix()`, `refactor()`, `feat()`).

ADDENDUM-131 closed at 20:22:54Z with a 38-minute 57-second window.
It saw **2** in-window merges across **2** active repos (codex 1,
opencode 1), giving a per-minute rate of **0.0514**. The bounce
direction was unambiguous on three independent axes simultaneously,
which is what earned the tick its triple-recovery label.

## The four trough metrics

To call ADDENDUM-130 a quadruple-trough requires four metrics to
hit local minima in the same tick. The addendum names them
explicitly:

1. **Per-minute merge rate trough.** The trajectory ADDENDUM-123 →
   ADDENDUM-127 → ADDENDUM-128 → ADDENDUM-129 → ADDENDUM-130 reads
   0.225 → 0.182 → 0.279 → 0.120 → **0.020**. The 0.020 figure is
   the lowest non-zero rate observed across the entire W17 window.
2. **Author-cluster trough.** codex's tick-by-tick distinct-author
   count read 1 (jif-oai dominance) → 3 → 5 → **1** across the same
   window. The 5 → 1 transition was the deepest single-tick
   author-count contraction observed in W17 codex.
3. **Active-repo-count trough.** The trajectory 3 → 2 → 3 → 2 → **1**
   was the lowest active-repo count of the entire week and represented
   the third consecutive monotonic contraction.
4. **Prediction-positive-resolution trough.** ADDENDUM-128 closed
   1 prediction positively (CCC); ADDENDUM-129 closed 1 (DDD —
   bolinfest #19900); ADDENDUM-130 closed **0** positively while
   resolving two preds NEGATIVE (AAA-285 jif-oai, EEE-287 kitlangton).
   First zero-positive-resolution tick in the most recent four-tick
   ledger.

All four hit minima in the same tick, on a window that was already
contracted in width (48m vs the 75m predecessor). The addendum
stream's framing — that ADDENDUM-130 represents "rate-trough +
author-trough + active-repo-trough + pred-positive-resolution-trough
all coincide in a single tick" — is justified by the numbers.

## The three recovery axes

ADDENDUM-131 reverses three of the four troughs in a single tick:

1. **Rate recovery: 0.020 → 0.0514 = 2.57× bounce.** Still well
   below ADDENDUM-128's 0.279 peak, but a clean partial recovery and
   the first non-monotonic point in the post-peak descent.
2. **Active-repo-count recovery: 1 → 2 (+100%).** Reverses the
   3-tick monotonic active-repo contraction. codex stays active and
   opencode rejoins as the second active repo via a single merge
   from a fresh non-sprint-author.
3. **Prediction-resolution recovery: 0 positive → 1 positive +
   1 elevation-to-confirmed.** Pred III (viyatb-oai network-proxy
   sub-cluster) gets a NEW CONFIRMED resolution; pred HHH (litellm
   yuneng-berri dormancy) reaches its n≥1 additional-tick elevation
   criterion and moves from candidate to confirmed.

The fourth metric — author-cluster count on codex — does **not**
recover in ADDENDUM-131. codex stays at a single author
(viyatb-oai). But the addendum makes a sharp distinction here that
is worth pulling out: viyatb-oai is operating in **sustained-
sub-cluster mode** rather than singleton mode, with a 3-member
network-proxy sub-cluster spanning ADDENDUM-130 and ADDENDUM-131.
The single-author-this-tick observation hides a structural recovery
that does show up in the cluster ledger: codex active-author-count
contracts but **active-cluster-count expands**.

## The two SHAs that anchor the recovery

The recovery tick has exactly two in-window merges, and both
deserve to be quoted in full because they each carry a distinct
structural payload.

The first is `viyatb-oai #19999 e1ba87ccb28e 19:51:44Z fix(network-
proxy): recheck network proxy connect targets`. This is codex's sole
in-window merge and the third member of viyatb-oai's emerging
network-proxy sub-cluster, joining `#20002 17:51:44Z tighten network
proxy bypass defaults` and `#20001 18:52:50Z harden linux proxy
bridge helpers`. The cluster spans 1 hour 59 minutes total with
member spacing of 1h00m + 59m06s — a near-uniform 60-minute
inter-member cadence. The PR numbers descend from #20002 to #20001 to
#19999 even as the merge times ascend, and the title-prefix
`fix(network-proxy):` is consistent across all three members. The
addendum stream contrasts this with the earlier jif-oai house-keeping
memories series, which had ascending PR numbers (#19990 → #19998 →
#20000 → #20005), an irregular 4m + 15m + 47m spacing, and a `feat:`
title prefix. The two clusters are structurally distinct in three
independent ways: PR-number direction, spacing regularity, and
semantic weight of the title prefix. The addendum-131 stream proposes
this as **synth #293 candidate** — descending-PR-number,
uniform-cadence, intra-author scoped-fix sub-cluster as a structurally
distinct sprint shape from jif-oai-class ascending-feat sprints.

The second is `rekram1-node #24734 0acac216aeee 19:58:51Z [title-
redacted: agent-variant sync from upstream API]`. This is opencode's
sole in-window merge and the structurally important one for the
recovery story, because it ends a 1-hour-58-minute opencode silence
that began at the end of kitlangton's n=6 httpapi+tui sprint
(last member `#24827 18:24:10Z fix(httpapi): finish sdk openapi
parity`). What makes the merge structurally interesting is that it
breaks the silence via a **different author** than the sprint author
and **different subsystem** than the sprint subsystem.
kitlangton's sprint was httpapi+tui; rekram1-node is integration-
variant cache / API sync. The addendum stream proposes this as
**synth #294 candidate**: post-sprint-completion silence-break
authorship inversion in which the silence-breaker is structurally
non-equal-to the sprint-author. This pairs cleanly with the earlier
gemini-cli observation in ADDENDUM-127 where the silence-break for
`#26066` was followed not by the original silence-breaker resuming
but by a `#26124` cherry-pick automation entry from a different
author. Two repos, two silence-break-authorship-inversion events,
both in the same week — it begins to look like a regularity rather
than a coincidence.

## Why the silence-break authorship inversion matters structurally

The naive prediction for a sustained single-author sprint is that
the sprint author owns the next merge after the sprint terminates,
because they have just demonstrated capacity, context, and review
attention on that surface. The empirical observation in W17 is the
opposite. Two of two observed post-sprint silence-break events have
been authored by someone other than the sprint author. The addendum
stream's hypothesis is that sprint completion locks the sprint
author out of further immediate merges via review-queue ordering
or via author-self-throttling. The mechanism is undetermined; the
phenomenon is at n=2 of n=2 confirmed observations.

A more interesting hypothesis: post-sprint quiescence on the
sprint author may be a **review-bandwidth artifact** rather than a
contributor-pacing artifact. A sustained 6-PR sprint by a single
author consumes finite reviewer attention on that surface; review
attention then has to rotate to other in-flight PRs from other
authors, and those PRs have a structural advantage in landing first
because they have been sitting in queue while the sprint cleared.
The silence-break therefore comes from whichever non-sprint-author
PR was next in queue when the sprint cleared — not from the sprint
author resuming.

If that mechanism is correct, then we expect the silence-break to
**also** be on a different subsystem from the sprint subsystem
roughly as often as the contributor distribution dictates, which
matches both observed cases (httpapi+tui sprint → integration-
variant break; gemini-cli post-#26066 → cherry-pick automation
break). It also predicts that the silence-break PR will have an
**older opened-at timestamp** than the sprint-final PR, because
it has been waiting longer in queue. That last prediction is
falsifiable from the GitHub API. Future addendum captures should
carry an opened-at timestamp on the silence-break PR and check it
against the sprint-final PR's opened-at.

## The dormancy elevation and the gemini-cli divergence

The non-recovery metric in ADDENDUM-131 — the litellm dormancy
elevation — is in some ways the most interesting observation on
the tick. ADDENDUM-130 had hypothesized litellm's `#26675
yuneng-berri 17:31:35Z` silence-break would be FALSIFIED by an
n≥3 follow-through burst (parallel to gemini-cli's
ADDENDUM-127→ADDENDUM-128 trajectory, which converted a
silence-break into a 5-merge burst). That falsification did not
arrive. ADDENDUM-130 logged 0 follow-through; ADDENDUM-131 logged
0 follow-through; total dormancy depth post-#26675 reached 2h51m
across two consecutive observation ticks of zero activity. The
hypothesis HHH thereby met its elevation criterion (n≥1
additional tick of n=0 follow-through) and moved from candidate
to confirmed.

The structural takeaway is the **per-repo divergence** in
silence-break-follow-through behavior. gemini-cli's `#26066`
silence-break produced an n=5 burst within one tick (and an
n=9 cumulative across two ticks). litellm's `#26675` silence-
break produced an n=0 follow-through across two ticks. The
differential — n=9 versus n=0 — is the largest per-repo
silence-break-follow-through divergence observed in W17 and
suggests that the silence-break-then-burst shape (gemini-cli)
and the silence-break-then-dormancy shape (litellm) are
qualitatively different release-engineering signatures driven
by different cadence regimes (gemini-cli's daily preview-channel
cadence versus litellm's weekly release cadence over a much
larger PR-numbered surface). This reinforces an earlier
synthesis: per-repo silence-break-follow-through divergence is a
release-engineering-pacing fingerprint and should be tracked as
a stable per-repo property rather than as a tick-level
prediction error.

## What ADDENDUM-131 didn't recover

Three things did not recover in the recovery tick, and they're
worth naming so the recovery framing doesn't overclaim.

First, the goose+qwen-code paired silence continued to deepen
without crossing a new threshold. goose silence stretched to 7h27m
and qwen-code to 7h18m, preserving the 9-minute paired-differential
that has now held at exact minute-resolution granularity across
ADDENDUM-129, ADDENDUM-130, and ADDENDUM-131 — three consecutive
ticks of lock-step silence-deepening. The paired-silence-deepening
process is not part of the recovery and shows no sign of being
part of any recovery; it is an independent dynamic on its own
surface.

Second, single-author-doublet count stayed at zero for the fifth
consecutive tick. ADDENDUM-129 had 3 single-author doublets;
ADDENDUM-130, ADDENDUM-131, and the three preceding ticks all
have 0. The doublet shape was a strong feature of the
ADDENDUM-128/129 surge phase and has fully evaporated in the
post-peak phase. The recovery on rate and active-repo-count is
not paired with a recovery on doublet density.

Third, the codex author-cluster count stayed at 1 (viyatb-oai)
even though, as noted above, viyatb-oai is operating in
sustained-sub-cluster mode. The author-diversity recovery — if
it is going to come — has not yet arrived on the codex surface
within the observation window.

## The auto-correlation question

If we step back from the per-tick mechanics and ask the underlying
empirical question — "do troughs at 30-to-80-minute resolution
self-sustain?" — the W17 evidence with this trough is **no**.
ADDENDUM-130 hit four simultaneous local minima; ADDENDUM-131,
the very next observation tick, reversed three of the four. The
single tick of compression was followed by a single tick of
partial recovery rather than by sustained compression, and the
recovery showed up across rate, active-repo-count, and prediction-
resolution simultaneously rather than on one axis at a time.

This is a single observation and does not prove low
auto-correlation on its own. But the addendum stream now has at
least two trough-to-recovery shapes in W17 — ADDENDUM-119 to
ADDENDUM-120 was the earlier instance — and both ran the same
single-tick-recovery pattern. If the pattern holds across more
observations, it implies the merge-stream rate at sub-hourly
resolution is essentially memoryless past a one-tick lag, which
in turn suggests that any predictor that uses last-tick rate to
forecast next-tick rate will be systematically wrong on the
direction of the move on trough-following ticks. That is a
testable claim against the back-fill of W17 addenda.

## What to track next

Three concrete follow-up questions emerge for the next four to
six addendum captures.

First, the silence-break-authorship-inversion synthesis (synth
#294 candidate) needs a third confirming observation to move
from "two-of-two coincidence" to "structural regularity." Watch
opencode and gemini-cli specifically; both have established the
shape on this week.

Second, the viyatb-oai descending-PR-number uniform-cadence
sprint (synth #293 candidate) needs a fourth member to confirm
the cadence — if `#19998` lands at roughly 20:51:44Z (one
60-minute step further), the 60-minute metronome is real; if
viyatb-oai goes silent for two ticks instead, the
uniform-cadence claim collapses.

Third, the goose+qwen-code 9-minute paired-differential needs
to break or hold across one more tick to determine whether it
is truly a stable invariant. If it survives through
ADDENDUM-132 and ADDENDUM-133, the paired-silence-deepening
process is operating at a level of synchrony that probably
deserves a dedicated synth lineage rather than a footnote in
the cross-tick metric ledger.

The clean trough-to-recovery shape on a single tick is the
headline. The three open synth candidates are the workload
this analysis hands forward to the next captures.

## Citation

Source data: `oss-digest/digests/2026-04-28/ADDENDUM-129.md`,
`ADDENDUM-130.md`, `ADDENDUM-131.md`. Anchor merges quoted from
the addenda: `viyatb-oai #19999 e1ba87ccb28e 19:51:44Z fix(network-
proxy): recheck network proxy connect targets`; `rekram1-node
#24734 0acac216aeee 19:58:51Z` (title redacted per source);
`dylan-hurd-oai #19959 7f7c7c2c07a1 19:08:41Z Fix log db batch
flush flake`; `bolinfest #19900 9e26613657bf 18:21:40Z permissions:
add built-in default profiles` (ADDENDUM-129 silence-breaker
context). Per-minute merge-rate sequence ADDENDUM-123 → 131
quoted directly from cross-tick metric ledgers.
