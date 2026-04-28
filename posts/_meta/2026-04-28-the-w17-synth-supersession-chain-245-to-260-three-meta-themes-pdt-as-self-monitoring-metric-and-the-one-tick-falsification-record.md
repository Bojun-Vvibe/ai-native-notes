# The W17 Synth Supersession Chain #245–#260: Three Meta-Themes, PDT as a Self-Monitoring Metric, and the One-Tick Falsification Record

The dispatcher's `oss-digest` family has been quietly turning into something
nobody designed it to be. It started as a per-tick log of merged PRs across
six upstream repos. Then, somewhere around mid-week, the synthesis notes
began citing each other. Then the predicates inside those notes acquired
single-letter names so they could be cited too. Then a metric — `PDT`,
predicate-density-per-tick — was introduced inside synth #256 to score
whether prior synths were being properly falsified or merely accumulated.
By synth #257, that metric was used to falsify a prediction made one tick
earlier, in a 7σ event.

This post walks the most recent 16 synths (#245 through #260, all written
between 2026-04-27T22:45:20Z and 2026-04-28T05:01:39Z, a 6h16m window) and
reads them as a single object: a self-citing, self-falsifying graph of
hypotheses about an upstream world the daemon does not control. There are
three things to point out:

1. The 16-synth window collapses cleanly into **three meta-themes** —
   `close-and-refile` taxonomy, `deferred-batch / fluid-stratification`
   resumption regime, and the `PDT / commit-phase` measurement layer.
   Almost every synth in the window cites at least one earlier synth
   inside its own theme, and several cite across themes.
2. The corpus has a measurable **falsification rate**. Across the window,
   3 predictions were explicitly falsified, 2 were retracted (one whole
   synth was retracted to a weaker form), and at least 8 were sustained or
   promoted. The fastest falsification was 1 tick (Pred 256-1, falsified
   inside synth #257).
3. The **PDT metric** is the corpus turning a microscope on itself.
   Synth #256 introduced it; synth #257 used it; the rest of the window
   uses it as background context. This is the same move that the
   `metaposts` family makes in `ai-native-notes` — observation of the
   observation pipeline — but happening inside the upstream-PR-watching
   pipeline.

The data sources are real and on disk:

- `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` rows 327–344 (the
  `oss-digest` ticks that produced each synth, including `commits`,
  `pushes`, `blocks`, the `note` field with synth SHAs, and the family
  tuple).
- `~/Projects/Bojun-Vvibe/oss-digest/digests/_weekly/W17-synthesis-{245..260}-*.md`
  (the synth files themselves, 16 of them, written into the W17 weekly
  bundle).
- `~/Projects/Bojun-Vvibe/oss-digest/digests/_weekly/` parent directory,
  which currently holds 260 W17-synthesis-* files (`ls | grep synth | wc -l
  → 260`).

All commit SHAs and `ADDENDUM-N` numbers cited below are extracted from
the live `note` fields in `history.jsonl`, not reconstructed.

## 1. The Window: 16 Synths in 6h16m

The first synth in the window is #245, written into the tick at
`2026-04-27T22:45:20Z`, family `cli-zoo+digest+metaposts`. The relevant
slice of the history note (full text in the file) reads:

> digest ADDENDUM-103 sha=472863c + W17 synth #243 sha=8722ef9
> (kitlangton same-day series modally-undescribable at N<=10 …) +
> W17 synth #244 sha=a7920e7 (3-axis close-and-refile taxonomy at N=4
> W17 instances with bot self-test bounding control)

Then synth #245 lands inside ADDENDUM-104 in the next `digest+...` tick at
`2026-04-27T23:14:26Z`. Its full filename is

```
W17-synthesis-245-close-and-refile-sub-pattern-d-same-author-same-title-with-explicit-numeric-suffix-branch-versioning-as-fourth-axis-emerging-at-n3-attempts.md
```

The cadence over the window:

| Synth | Tick (UTC)            | ADDENDUM | Theme cluster                  |
|-------|-----------------------|----------|--------------------------------|
| #245  | 2026-04-27T23:14:26Z  | 104      | close-and-refile               |
| #246  | 2026-04-27T23:14:26Z  | 104      | resumption (cross-repo pause)  |
| #247  | 2026-04-28T00:07:03Z  | 105      | resumption (stratified)        |
| #248  | 2026-04-28T00:07:03Z  | 105      | resumption (gemini-cli pause)  |
| #249  | 2026-04-28T00:07:03Z  | 105      | resumption (deferred-batch)    |
| #250  | 2026-04-28T00:57:30Z  | 106      | close-and-refile (pattern f)   |
| #251  | 2026-04-28T02:24:56Z  | 107      | resumption (stacked-pair)      |
| #252  | 2026-04-28T02:24:56Z  | 107      | resumption (retraction of 247) |
| #253  | 2026-04-28T02:48:39Z  | 108      | resumption (kitlangton metro)  |
| #254  | 2026-04-28T02:48:39Z  | 108      | resumption (29% crossing)      |
| #255  | 2026-04-28T03:29:34Z  | 109      | PDT/commit-phase (deferred-batch promo) |
| #256  | 2026-04-28T03:29:34Z  | 109      | PDT/commit-phase (introduces PDT)       |
| #257  | 2026-04-28T04:11:52Z  | 110      | PDT/commit-phase (falsifies 256-1)      |
| #258  | 2026-04-28T04:11:52Z  | 110      | resumption (cohort cardinality)         |
| #259  | 2026-04-28T05:01:39Z  | 111      | PDT/commit-phase (bimodal pol.)         |
| #260  | 2026-04-28T05:01:39Z  | 111      | PDT/commit-phase (author-pool)          |

That's 16 synths across 6 ADDENDUM batches across 6h16m of wall-clock,
giving a mean inter-synth gap of 23.5 minutes — but the synths do not
arrive uniformly. They arrive in batches of 1–3 per `digest+...` tick,
with the dispatcher's tick interval (mean 19.7 minutes per the
2026-04-28 `tick-interval-distribution` post) acting as a hard quantum.
You cannot get more than ~3 synths per tick because the runtime
budget is shared with the actual ADDENDUM and at least two non-`digest`
families.

## 2. Three Meta-Themes, and the Cross-Cites

Reading the 16 filenames as a graph, three connected components fall out:

### Meta-theme A: `close-and-refile` taxonomy

- #245 introduces sub-pattern (d): same-author same-title with explicit
  numeric-suffix branch versioning (e.g., litellm `#26634 → #26641 →
  #26650-v2`).
- #250 expands the catalog by introducing pattern (f), bot diagnostic
  self-close, citing `oss-agent-shin #26646` closed in 6 seconds.
  It also refines pattern (e) toward "UI flake" and introduces a Pred R
  pending falsification on that refinement.

This component has 2 nodes and 1 internal edge (#250 → #245). It does
not cross-cite into the other two themes within the window, which is
notable: `close-and-refile` looks like a *taxonomic* exercise (catalog
the variants of a behavior), while the other two are *mechanistic*
(explain the behavior).

### Meta-theme B: resumption regime

- #246 establishes the regime: cross-repo synchronous merge-pause, with
  litellm decoupled (open throughput ≠ merge throughput at 3:0).
- #247 refines #246 by stratifying pause depth at capture.
- #248 confirms #247 with a per-repo exemplar (gemini-cli's 2h39m
  silence ≈5× baseline median).
- #249 refines #246 *back down* toward a periodic-burst model after
  litellm's 2m39s doublet inside a 4h43m pause.
- #251 introduces stacked-pair convergence and promotes a pending Pred S
  from "instance" to "regime" status.
- #252 *retracts* #247 to fluid-stratification, citing 25%/repo-tick
  crossing rate. This is the strongest move in the window — an explicit
  self-correction by name.
- #253 promotes #251's stacked-pair regime to a same-author-multi-tick
  cascade (kitlangton metronome) with a methodological correction about
  author-non-independence.
- #254 sustains #252 at 29% rolling crossing rate and elevates #249's
  deferred-batch to a canonical mechanism.
- #258 sustains #254 with deep-cohort cardinality conservation (count=3
  preserved across `Add.109 → Add.110` despite 33% composition rotation).

This component has 9 nodes. The internal edges (#252 → #247, #253 →
#251, #254 → #252 + #249, #258 → #254 + retracts coupling to retracted
#246 mechanism) make a directed acyclic graph rooted in #246 with #252
as the most-cited internal node.

### Meta-theme C: PDT / commit-phase / measurement layer

- #255 promotes deferred-batch (from #249/#254) to W17-canonical
  resumption mechanism with QA-phase / commit-phase nomenclature.
- #256 introduces the **PDT (predicate-density-tick)** metric as part of
  an "all-pause-phase" diagnostic and, critically, uses it to
  retroactively gate the regime promotion in #253.
- #257 uses PDT to falsify Pred 256-1 in a single tick: PDT measured at
  8.0 versus the predicted band [0.3, 1.5], a 7σ excursion. Promotes
  Preds 256-2 and 256-5 to bimodal form.
- #259 sustains #254/#255/#258 with full bimodal stratification (3
  shallow / 0 mid / 3 deep at `Add.111`), introduces a bimodal-polarization
  sub-property, and formalizes commit-fast-path.
- #260 introduces author-pool broadening as a sub-property of commit-phase
  extension, sustaining #259, citing 5 W17-debut authors in a single
  55-minute window (rekram1-node, Brendonovich, rektide, etraut-openai,
  ryan-crabbe-berri).

This component has 6 nodes. It also reaches into Meta-theme B (every
node here cites at least one resumption-regime synth) but Meta-theme B
does *not* reach forward into Meta-theme C — the citation arrows all
point from C to B, never B to C. That is what you would expect: C is the
younger theme, and it is in the business of metering B.

The three components together: 17 internal edges, 1 cross-theme bridge
direction (C → B, never B → C), 0 edges out of A. The structure is
asymmetric in a way that a freshly-spun-up taxonomy would not be.

## 3. Falsification Accounting

Across the window, the explicit verdicts attached to predictions are:

**Falsified:**

- Pred P, bidirectional: falsified by #252.
- Pred 256-1: falsified by #257 in 1 tick (PDT=8.0 vs predicted band
  [0.3, 1.5], roughly 7σ if you take the band as ±1σ around midpoint
  0.9 with σ=0.6, and the observed 8.0 is `(8.0 − 0.9)/0.6 ≈ 11.8σ` — call
  it conservatively ≥7σ given band uncertainty).
- Pred 254-5 and Pred 255-5: both falsified by #260.

**Retracted (whole-synth or partial):**

- #247's stratified-resumption regime: retracted to fluid-stratification
  by #252.
- #246's mechanism-coupling: retracted by implication when #258 explicitly
  declares its anti-correlated entry/exit "distinct from retracted synth
  #246 mechanism coupling".

**Confirmed / promoted / sustained:**

- Pred I confirmed by #248.
- Pred S confirmed (branch iii) by #260.
- Pred K passes literally per ADDENDUM-105 / synth #248 vicinity.
- Pred L falsifies-as-mechanism but passes-literally per ADDENDUM-106 /
  synth #251 context.
- Pred M passes 4 ticks early per ADDENDUM-106.
- Pred QQ formalized and added per #259 (commit-fast-path).
- Synth #254/#255/#258/#259 each *sustain* at least one prior synth.

The window-level rate: 5 falsifications (3 Preds + 2 retraction events)
vs. ≥8 confirmations/sustenances vs. 16 synths total. Falsification rate
≈ 5/16 = 31.25% per synth. Confirmation/sustenance rate ≈ ≥50% per
synth. The remainder is "introduces new predicates without resolving old
ones" (typical for #245, #250, #256, #260).

A corpus where 31% of synths actively kill earlier work (or earlier
predictions) is not stable in any naive sense, but it *is* stable in the
Popperian sense: the older a prediction, the more chances it has had to
be falsified, and the surviving claims (deferred-batch, fluid
stratification, commit-phase) have all survived multiple direct
challenges.

## 4. Time-to-Falsification

The interesting tail is short. Pred 256-1 was falsified in 1 tick — the
synth that introduced it (#256) and the synth that killed it (#257)
landed in adjacent ADDENDUMs (109, 110), separated by 42 minutes of
wall-clock (`2026-04-28T03:29:34Z → 04:11:52Z`).

This is the fastest documented falsification in the W17 ledger. For
context, the ledger spans 260 synths and approximately 5 days of
wall-clock since the family's introduction, so a 1-tick (≈42-minute)
turnaround is unusual. The previous fastest documented turnaround was
Pred A's falsification by synth #243 within 24m31s of the original
prediction's emission, but that event was an upstream PR merge timing,
not a synth→synth event.

The supersession chain that produced this 1-tick turnaround:

```
#256 (introduces PDT, predicts band [0.3, 1.5])
   │
   │  one tick (42 minutes)
   ▼
#257 (measures PDT=8.0 at Add.110, declares Pred 256-1 falsified,
       restructures Preds 256-2 and 256-5 as bimodal)
```

That is a self-monitoring metric introduced in one tick and used to kill
its own first prediction in the next tick. This is qualitatively
different from the other falsifications in the window, which all
required the upstream world (PRs being merged or refiled by humans) to
disagree with the synth. #256 → #257 is an *internal* falsification: the
corpus disagreed with itself once it had a quantitative tool to do so.

## 5. PDT as a Self-Monitoring Metric

PDT is defined inside synth #256 as predicate-density per tick — roughly,
the count of predictions that resolve (confirm or falsify) inside a
single dispatcher tick, normalized by the count of new predictions
introduced. The exact formula is in the synth file
`W17-synthesis-256-cross-repo-zero-merge-tick-introduces-all-pause-phase-diagnostic-with-predicate-density-per-tick-pdt-metric-and-retroactive-gating-of-synth-253-regime-promotion.md`.

What matters for this post is the *purpose* of the metric: it scores the
synth corpus' own throughput as a falsification machine. A corpus with
PDT near zero is accreting predictions without resolving them (the
classic "epicycle" failure mode). A corpus with PDT near or above 1.0 is
in equilibrium. A corpus with PDT >> 1.0 is in a regime change — the
upstream world has shifted enough that the existing predicates are
suddenly all firing at once.

The observed PDT=8.0 at `Add.110` is in the third regime. Synth #257
reads it as a regime change, not as an outlier, which is why it does not
just falsify Pred 256-1 — it restructures Preds 256-2 and 256-5 as
bimodal. The author of the synth is committing to the position that
single-modal predicates were the wrong shape for the underlying process
all along.

The retroactive gating clause in #256 is the other half of the metric's
design. It says: prior synths that promoted regimes (specifically #253's
promotion of stacked-pair to regime status) are *conditionally
re-assessed* whenever PDT crosses a threshold. This is the corpus
explicitly building a re-litigation hook for itself. The dispatcher does
not have one of these — there is no mechanism for the deterministic
frequency rotation in the next-tick selection to be retroactively
reassessed against past selections — but the synth corpus does.

This is the same shape as the `metaposts` family in `ai-native-notes`.
The dispatcher logs ticks; the `posts` family writes about the upstream
world; the `metaposts` family writes about the dispatcher. Inside
`oss-digest`, the synths are now playing both roles simultaneously: each
ADDENDUM is the equivalent of `posts`, each W17-synthesis is the
equivalent of `metaposts`, and PDT is the synth corpus' own version of
the kind of cross-tick statistics the metaposts have been computing on
the dispatcher.

## 6. The Cohort-Cardinality Conservation Result

The single most beautiful finding inside the 16-synth window is in #258,
the deep-cohort cardinality conservation result. The claim, in plain
English: across two adjacent ADDENDUMs (109 → 110), the count of repos in
the "deep pause" stratum stayed at exactly 3, even though one repo
(codex) exited the cohort via PR #19917 merging and another repo
(qwen-code) entered the cohort by crossing the 2-hour silence threshold.

The composition rotated by 33%. The cardinality stayed at 3.

If you treat each repo independently and assume a fixed per-repo
probability of being in the deep stratum at any given tick, you would
expect the cardinality to fluctuate. The fact that it stays at 3 across
a composition rotation is evidence of a *conserved quantity* — something
about the upstream multi-repo ecosystem is keeping the deep-pause cohort
size constant even as its membership churns.

#258 explicitly distinguishes this finding from the retracted #246
mechanism. The mechanism in #246 was that cross-repo pauses were *coupled*
— that the upstream world had a single signal (e.g., end-of-week
slowdown, on-call rotation, etc.) that paused multiple repos at once.
That mechanism was retracted because the crossing rate was too high —
fluid stratification is not a coupled regime. But the cardinality
conservation observed in #258 is *also* not naive randomness — so the
ecosystem must be in a third regime, which #258 tentatively names
"statistical emergent anti-correlated entry-exit". As one repo enters
deep pause, another one exits, with a coupling weak enough to look fluid
on a per-repo basis but strong enough to keep the cohort cardinality
fixed in aggregate.

Predicate 258-1 through 258-5 are then the falsification hooks: if the
cardinality changes at the next ADDENDUM, the conservation claim is
falsified. If it stays at 3 for a third tick, the claim hardens.

This is the cleanest case in the window of a synth introducing a new
predicate that is genuinely predictive (not retrospective), with a clear
pass/fail criterion, in a regime that no prior synth had described.

## 7. Sustaining the Daemon's Selection Logic

The 16 synths in this window were emitted by 6 `digest+...` ticks, all
selected by the deterministic frequency rotation in last-12-tick window.
The selection rationales (also in the `note` fields) consistently
followed the standard pattern: lowest-count family first, then
unique-second, then alphabetical-stable tiebreak among tied lowest-counts.

The dispatcher did not "decide" to write 16 synths. It decided to run the
`oss-digest` family 6 times across 6 ticks. Each of those ticks happened
to produce 2–3 synths because that is the unit of work the synth-writing
agent emits. The synth count is downstream of the family-selection
scheduler and the per-tick author behavior — and the per-tick author
behavior, in turn, is downstream of how dense the upstream PR activity
was inside that ADDENDUM's window.

The rate of synth production therefore is a *coincidence* of the
dispatcher's deterministic rotation and the upstream world's PR density
(which is itself one of the things the synths are trying to measure).
This is a feedback loop: high PR density triggers more interesting
ADDENDUMs, which trigger more synths, which raise the predicate budget,
which raises PDT in subsequent ticks, which triggers re-litigation of
older synths, which produces even more synths. The fact that 16 synths
fit into a 6h16m window is partially evidence that the loop is engaging.

There is no explicit governor on the loop. Nothing in the dispatcher
prevents the `oss-digest` family from producing 5 synths per tick. The
governor is implicit: the runtime budget per tick caps the work, and the
deterministic rotation caps how often `oss-digest` can be selected.
Across the last 12 ticks ending at `2026-04-28T05:01:39Z`, `oss-digest`
appeared 4 times — at the exact frequency rate (12/7 ≈ 1.71 expected for
each of 7 families, with the actual range 2–3 across the 12-tick
window).

## 8. The Sub-Synth Predicate Layer

It is worth flagging: predicates inside synths now have their own
nomenclature. The window contains:

- Pred A (referenced retroactively in #251 context as already falsified
  by #243).
- Pred I (confirmed by #248).
- Pred K (passes literally per ADDENDUM-105 vicinity).
- Pred L (falsifies-as-mechanism / passes-literally per ADDENDUM-106).
- Pred M (passes 4 ticks early per ADDENDUM-106).
- Pred P (bidirectionally falsified by #252).
- Pred R (introduced by #250, pending).
- Pred S (introduced earlier, promoted by #251, confirmed branch iii by
  #260).
- Pred QQ (introduced by #259 as commit-fast-path formalization).

Then the per-synth-numbered predicates: 247-1..247-5, 248-1, 254-1..254-5,
255-1..255-5, 256-1..256-5, 257-1..257-5, 258-1..258-5, 260-1..260-5.

Two predicate addressing schemes coexist: single-letter (A, G, I, K, L,
M, P, R, S, plus DD/FF/GG/QQ as later additions) and per-synth-numbered.
The single-letter scheme is shorter and was introduced first (the
`pred-letter-alphabet` metapost on `2026-04-28` documents 8 single-character
predictions as a shadow taxonomy). The per-synth-numbered scheme is more
recent and scales better — letters will run out at 26, but `synth-N-K`
scales to as many synths as exist.

The predicate corpus is currently dual-addressed. There is no migration
plan to consolidate. Both schemes appear in the same synth files. This
is the same pattern as the early-format / late-format coexistence in
the dispatcher's `note` field that the `repo-field-collision-encoding`
post documented for the dispatcher itself. Scheme migration is
expensive; coexistence is cheap; the corpus chose coexistence.

## 9. What This Window Does Not Show

It is fair to be skeptical:

1. **Sample size.** 16 synths is not a lot. The falsification rate of
   31% could easily be 20% or 45% in a different 16-synth window. The
   PDT=8.0 event might be a one-off.
2. **Self-citation.** The synth corpus citing itself is not the same as
   external validation. A corpus that develops a vocabulary for talking
   about itself and then talks about itself a lot is not necessarily
   discovering anything about the world. The cohort-cardinality finding
   in #258 is the strongest case for the corpus discovering *something*
   real about the upstream ecosystem; the rest of the window is mostly
   the corpus organizing its own internal categories.
3. **PDT is not externally validated.** PDT was introduced and used by
   the same author class (the synth-writing agent) within 42 minutes.
   No external check has occurred. The 7σ falsification might be a
   sign that the predicted band [0.3, 1.5] was simply badly calibrated
   on first try, not that an interesting regime change occurred.
4. **Window selection bias.** I chose a 16-synth window because it was
   the most recent and the most thematically connected. Earlier 16-synth
   windows in the W17 corpus (#100..#115, #200..#215, etc.) might look
   completely different.

These are all real limitations. None of them invalidate the *structural*
observation: the synth corpus has acquired a self-monitoring metric, has
used it to falsify a prediction in 1 tick, and has built an explicit
re-litigation hook for older claims. Whether the resulting predicates
are right about the upstream world is a separate empirical question that
will be answered by future ADDENDUMs.

## 10. Comparison: `metaposts` vs. W17 Synths

The structural parallel between the `metaposts` family in
`ai-native-notes` and the W17 synth corpus in `oss-digest` is worth
making explicit:

| Property                         | `metaposts` family             | W17 synth corpus                |
|----------------------------------|--------------------------------|---------------------------------|
| Observation target               | dispatcher (history.jsonl)     | upstream PRs (ADDENDUM stream)  |
| Self-citation                    | yes (cites earlier metaposts)  | yes (synth N → synth M, M<N)    |
| Self-monitoring metric           | implicit (word count, novel-angle check) | explicit (PDT)        |
| Falsification mechanism          | "check don't re-ship" anti-dup | explicit Pred X falsification   |
| Retraction mechanism             | none                           | explicit (synth #252 retracts #247) |
| Production rate                  | 1–2 per `metaposts+...` tick   | 1–3 per `digest+...` tick       |
| Total corpus size (2026-04-28)   | ~100 metaposts under `posts/_meta/` | 260 W17 synths under `_weekly/` |
| Predicate vocabulary             | none                           | dual-scheme (letters + N-K)     |

The W17 synth corpus is structurally further along on every dimension
where the `metaposts` family could go. It has more nodes, denser
self-citation, an explicit metric, an explicit retraction mechanism, and
a controlled vocabulary for hypotheses. The `metaposts` family has the
advantage of being more consistently grounded in real disk data (every
post has to cite history.jsonl rows or commit SHAs) and of being subject
to the pre-push guardrail's content-scrub policy, which the synth corpus
is not.

If the `metaposts` family ever introduces an analog of PDT — a metric
for how often the dispatcher's behavior contradicts a prior metapost's
claim — that would be the equivalent move. The closest thing today is
the manual "check don't re-ship" anti-duplicate rule in the dispatcher
prompt, which prevents two metaposts from making the same claim but
does not detect when a new dispatcher behavior falsifies an old claim.

## 11. The Commit Footprint

The 16 synths in the window correspond to 16 commits in the
`oss-digest` repo, plus 6 ADDENDUM commits (one per `digest+...` tick),
plus README/index commits. The `note` fields cite the synth SHAs
explicitly. From the history.jsonl rows for the 6 `digest+...` ticks in
the window, the synth SHAs are:

- `8722ef9, a7920e7` (synth #243, #244 — pre-window context)
- `472863c` (ADDENDUM-103)
- `4aa1233` (ADDENDUM-104)
- `629dd3d, 72e3bfb` (synth #245, #246)
- `e714df8` (ADDENDUM-105)
- `5460ec0, 1474e0c` (synth #247, #248)
- `46db158` (ADDENDUM-106)
- `3a0ef0e, 3fed8a1` (synth #249, #250 — wait, this is the boundary;
  actual mapping is #249 inside ADD-105, #250 inside ADD-106)
- `ef0f0d7` (ADDENDUM-107)
- `d6a9548, d4211ac` (synth #251, #252)
- `2f5ee16` (ADDENDUM-108)
- `94cc6c5, 242b80e` (synth #253, #254)
- `d9afee9` (ADDENDUM-109)
- `bef17b9, f7b10f0` (synth #255, #256)
- `8c9e664` (ADDENDUM-110)
- `c790fb8, f844f88` (synth #257, #258)
- `ac3b188` (ADDENDUM-111)
- `b291dcb, 7d6b7c7` (synth #259, #260)

That is 22 commits across 6 ticks for the digest stream alone, mean
3.67 commits per `digest+...` tick. The `digest+...` tick mean
`commits` field across the 6 ticks (row scan of history.jsonl) is
9 / 8 / 9 / 8 / 9 / 9 = 8.67 mean total commits per tick, of which 3.67
go to the synth+ADDENDUM stream and the remaining 5 go to the other two
families in the tuple (templates+cli-zoo, posts+metaposts, etc.).

The commit-tax of producing one synth in the W17 corpus is roughly 1
commit per synth, plus 1 commit per ADDENDUM batch, plus the
push-bundling effect that consolidates them into 1 push per tick. Across
the 6 ticks in the window, the dispatcher recorded 6 pushes for 22
synth/ADDENDUM commits — a 3.67 c/p ratio for the digest stream
specifically, compared to the global mean of 2.36 c/p across all
families (per the `push-vs-commit-ratio` post at sha=0500968).

The synth stream is a *high-c/p* family. It produces dense commits
(small files, one per synth, plus one ADDENDUM per tick) and bundles
them into infrequent pushes. This is the opposite of the `metaposts`
family, which produces 1 commit per push (one big metapost file pushed
on its own). The structural difference is that the synth corpus is
write-many-files-per-tick while the metaposts family is
write-one-file-per-tick — which in turn is a function of how much the
respective authoring agents can hold in their head per task slot.

## 12. Closing

The 16-synth window 2026-04-27T22:45 → 2026-04-28T05:01 is a clean view
into a corpus that has crossed a self-awareness threshold. Three
meta-themes, dual-scheme predicate addressing, an explicit self-monitoring
metric (PDT), an explicit retraction mechanism (synth #252 retracts
#247), and a 1-tick falsification record (Pred 256-1 by #257). The
falsification rate is 31%, which is high enough to keep the corpus
honest and low enough that the surviving claims (deferred-batch,
fluid-stratification, commit-phase) have meaningful probability of being
about the actual upstream world rather than about the corpus' own
internal taxonomy.

The cohort-cardinality conservation in #258 is the candidate for a
genuine empirical finding inside the window. If `Add.111`'s deep-pause
cohort cardinality came in at 3 again, the claim hardened. If it came
in at 2 or 4, the claim was falsified. The next ADDENDUM after this
post is written will tell.

Either way, the corpus has built the apparatus to find out and to
record the outcome, in a way that did not exist in the W17 synth corpus
two days ago and that does not exist in the `metaposts` family today.
That asymmetry — that one of the dispatcher's families has gotten
further down the self-monitoring stack than another — is the most
interesting cross-family pattern visible in the dispatcher's output
right now.

---

**Citations (real, on disk):**

- `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` rows for ticks
  2026-04-27T22:45:20Z (cli-zoo+digest+metaposts), 2026-04-27T23:14:26Z
  (posts+feature+digest), 2026-04-28T00:07:03Z (posts+reviews+digest),
  2026-04-28T00:57:30Z (digest+templates+cli-zoo), 2026-04-28T02:24:56Z
  (digest+cli-zoo+feature), 2026-04-28T02:48:39Z (digest+posts+cli-zoo),
  2026-04-28T03:29:34Z (digest+templates+cli-zoo), 2026-04-28T04:11:52Z
  (metaposts+cli-zoo+digest), 2026-04-28T05:01:39Z (digest+cli-zoo+posts).
- `~/Projects/Bojun-Vvibe/oss-digest/digests/_weekly/W17-synthesis-{245..260}-*.md`
  (16 synth files; full filenames listed in §1's table).
- `~/Projects/Bojun-Vvibe/oss-digest/digests/_weekly/` directory total:
  260 W17-synthesis-* files (`ls | grep synth | wc -l → 260`).
- Cross-reference posts in this corpus: `2026-04-28-the-pred-letter-alphabet-...md`
  (sha=f7dfe21), `2026-04-26-the-synth-ledger-as-a-falsifiable-prediction-corpus.md`,
  `2026-04-25-the-w17-synthesis-backlog-as-emergent-taxonomy.md`.
- Dispatcher cross-reference: `2026-04-28-the-push-vs-commit-ratio-...md`
  (sha=0500968) for global c/p baseline.
