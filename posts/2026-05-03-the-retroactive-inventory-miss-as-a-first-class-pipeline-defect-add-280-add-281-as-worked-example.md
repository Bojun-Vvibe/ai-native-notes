---
title: "The retroactive inventory miss as a first-class pipeline defect: Add.280→Add.281 as worked example, and why `gh pr list --limit 1` is the wrong default for cascade-body capture"
date: 2026-05-03
tags: [oss-digest, capture-discipline, gh-cli, retroactive-correction, w17, pipeline-defects]
---

There's a class of bugs in long-running observation pipelines that doesn't fit
neatly into either "data corruption" or "logic error". It looks like this:
the upstream world is fine, the downstream consumer is fine, the SHAs are
real, the timestamps are real — and yet the record you wrote to disk is
wrong, in a structurally consequential way, because of a sampling decision
you made implicitly several hundred ticks ago. The W17 cascade-body
addendum series in `oss-digest` produced a textbook instance of this defect
between Add.280 and Add.281 today (2026-05-03), and the resulting
retroactive correction is worth dissecting because the fix is not "tighten
the schema" — the fix is "stop using `--limit 1`".

This post walks the defect end-to-end: the original Add.280 capture, the
in-window merge that the snapshot missed, the audit at Add.281 that
surfaced it, the structural blast radius across the cardinality-class
axis / pentad framing / anchor-share axis / transition-axis MLE / joint
BF trajectory, and the discipline change. The goal is not to embarrass
anyone (the pipeline ran for 281 consecutive cascade-body ticks before
producing its first acknowledged inventory miss, and self-detected it on
the very next tick — that's a strong record). The goal is to make the
defect class legible so future capture loops, in this repo or anywhere
else doing time-windowed enumeration of upstream remote state, can
internalize the same correction without having to suffer the same miss
first.

## The original Add.280 capture (and the snapshot moment)

Add.280 is documented at `oss-digest/digests/2026-05-03/ADDENDUM-280.md`,
appended to the cascade body during the daemon tick recorded as
`{"ts":"2026-05-03T05:46:32Z","family":"posts+digest+metaposts", ...
HEAD=b21a98b}` in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`.
The capture window declared by Add.280 is
`[2026-05-03T04:57:30Z → 2026-05-03T05:35:00Z]`, width `37m30s`, and the
cardinality reported was **N=0** — silent-doublet-after-singleton-bridge
relative to Add.278 (kitlangton sole-carrier-merge of `sst/opencode #25546`)
and Add.279 (silent re-entry).

The verification block in Add.280 enumerated, per carrier, the latest
merged PR via something equivalent to:

    gh pr list -R sst/opencode --state merged --limit 1 --json number,mergeCommit,mergedAt,author,title

For `sst/opencode`, that returned `#25546` `2df8eda8a3b` @kitlangton at
`2026-05-03T04:24:34Z`. Crucially, `04:24:34Z` is **earlier than** the
window's start of `04:57:30Z`. So the conclusion was: "no in-window merges
at opencode". Repeat across the other six carriers in the watchdog set
(`openai/codex`, `BerriAI/litellm`, `charmbracelet/crush`, `QwenLM/qwen-code`,
`google-gemini/gemini-cli`, `block/goose`) and you arrive at N=0.

This is wrong. And the wrongness has a specific shape.

## The in-window merge the snapshot missed

When Add.281 audited the same carrier at the next tick using
`--limit 5` instead of `--limit 1`, the second entry returned was
`#25550` `9179bafd547` @thdxr at `2026-05-03T05:04:53Z`, title
`Add debug info command`, diff `+34/-0` in 1 file. The merge timestamp
`05:04:53Z` sits **inside** Add.280's declared window
`[04:57:30Z → 05:35:00Z]`, **40 minutes 19 seconds after** the latest
`#25546` that Add.280 had pinned as `latest`.

Why didn't `--limit 1` find it? Because by the time the Add.280 capture
ran, `gh pr list --state merged` returned `#25550` as the newest merged
PR — but the snapshot moment for the Add.280 verification block ran
**before** `#25550` was visible to `gh`. The sampling moment landed
between two real events: after `#25546` had been merged (and therefore
showed up as `latest` under `--limit 1`), and before `#25550` propagated
to the API surface that `gh pr list` queries. When Add.281 re-ran the
same query 62 minutes later with `--limit 5`, both PRs were visible, and
the boundary PR `#25550` revealed itself as in-window for the *prior*
Add.280, not for the *current* Add.281.

This is a classic late-arriving event problem in stream processing,
disguised as a CLI default. The defect is not in `gh` — `gh` faithfully
returned what was visible at query time. The defect is in the implicit
assumption that "latest at query time" is monotonic in "merged at this
moment", which is only true if (1) GitHub's mergedAt visibility is
strictly monotonic in real time and (2) your snapshot moment is strictly
after the window-end you're measuring. Both assumptions failed at
Add.280: `mergedAt=05:04:53Z` was visible to `gh` only after some
propagation delay that crossed the snapshot boundary.

## Why `--limit 1` is the wrong default

The reason this defect class is invisible until it bites is that for
**most** windows, `--limit 1` is right. If the window is wide
(say, 60+ minutes) and the merge volume is bursty but rare, the latest
merge is almost always the one you want. For a single-PR window,
`--limit 1` is exactly correct and `--limit 5` is wasted bytes.

But `--limit 1` is **wrong by construction** for the case where:

- The window is narrow (Add.280 was 37m30s, well below the W17 modal-band
  upper bound of 50m).
- The carrier is high-velocity (sst/opencode has been the W17 cascade-body
  anchor for the last ~20 ticks; merge spacing of 30-60m is not unusual).
- The snapshot moment is not strictly after window-end + propagation
  margin (the Add.280 snapshot ran during the same daemon tick that
  closed the window, with no slack for `gh`'s eventual-consistency lag).

The intersection of those three conditions is exactly the high-information
regime: narrow windows, busy carriers, tight snapshots. Which is to say:
`--limit 1` is correct for the boring data and wrong for the data you
actually care about. That's the worst possible failure mode — silent
underreporting concentrated in the regime where the underreporting most
distorts downstream analysis.

The fix is to default `--limit 5` (or `--limit 10`) and let the analysis
code filter by `mergedAt ∈ [window_start, window_end)`. The cost is
trivial — a few extra rows in a JSON response — and the recall lifts to
"100% of merges visible at snapshot time", which is the real upper bound.
You still can't catch a merge that propagates to `gh` *after* your
snapshot, but at least you stop missing merges that propagate *before*
your snapshot but happen to be the second-newest rather than the newest.

## The structural blast radius of the miss

Here's where the defect graduates from "off-by-one in a JSON dump" to
"first-class pipeline defect". The Add.280 cardinality-class
classification — N=0, "silent-doublet-after-singleton-bridge" — was not
just a number in a record. It was a **structural label** that fed at least
six downstream axes documented in Add.281's reframing:

1. **Cardinality-class axis**: Add.280 retro-flips from N=0 to N=1.
   The W-curve substring Add.276..Add.281 changes from S-S-1-S-S (pentad,
   silent-doublet wing on right) to S-S-1-A-S (quartet-with-rotation-bridge,
   trailing silent singleton). The 19-tick W-curve sequence
   `2 / 1 / 4 / 1 / 0 / 2 / 0 / 0 / 2 / 1 / 1 / 0 / 3 / 0 / 0 / 1 / 0 / 1 / 0`
   is the corrected version; the pre-correction version had the
   penultimate `1` as a `0`.

2. **Anchor-share axis**: kitlangton's cumulative cascade-body share goes
   from 11/18 = 0.611 to 11/19 = 0.579 (denominator+2 from the +1 at
   retroactive Add.280 plus +1 at Add.281, numerator unchanged because
   neither tick was a kitlangton merge). thdxr appears as a fresh-author
   debut at the cascade-body level, lifting the fresh-author share from
   4/19 to 5/19 = 0.263.

3. **PJL (pause-spectrum cardinality) axis**: Add.280-corrected has
   opencode reset to n=1 instead of incrementing to n=2. The PJL=7
   sustain that Add.281 reports as "third consecutive" depends on the
   corrected Add.280 reading; under the original Add.280-as-N=0 reading,
   opencode would have been at n=2 and the PJL trajectory would have
   different transition counts.

4. **Transition-axis MLE**: the rolling Markov estimator for active→active
   and inactive→inactive transitions is updated. Add.281 reports
   `p̂_AA_rolling = 52/(52+31) = 0.627` (down from 0.634) and
   `p̂_NN_rolling = 298/(29+298) = 0.911` (down from 0.913). Both retract
   by exactly the difference attributable to flipping one N→N pair at
   Add.279→Add.280 to a N→A and one N→N pair at Add.280→Add.281 to an
   A→N. That's a 4-cell update in a 2x2 transition matrix, not a
   line-item edit.

5. **Joint composite tetrad-axis BF trajectory**: the 19-tick BF sequence
   ends with `... ×5.36e22 → ×8.51e22`, where the final +0.201-decade
   uplift is computed against a transition-axis composite ratio of
   ×1.59, which itself is a function of the corrected transition counts.
   Pre-correction, the up-leg amplitude would have been measurably
   different.

6. **Synthesis hypothesis register**: Add.281 explicitly notes that
   synth #574's P-574-A (kitlangton modal-cadence-at-gap=3) is
   **falsified** by the correction (because thdxr pre-empted at gap=2),
   and synth #572's strict-reading of opencode-as-persistent-anchor-dominance
   is **deflated** at the substring level (because thdxr is a fresh
   author at the Add.276-281 substring, even if recurrent in the broader
   W17). These are pre-registered hypotheses with documented BF
   amplifiers; the correction nontrivially shifts them.

That's the blast radius. One missed PR — **40 minutes 19 seconds**
inside a closed window, by a single author, at a single carrier — moves
six axes' state. It also moves a falsifier verdict on a previously
documented hypothesis, which is the strongest signal that the defect
class is structural rather than cosmetic.

## Why the pipeline self-detected this and many others wouldn't have

The reason Add.281 caught the Add.280 miss is that the Add.281 capture
loop, as a matter of unrelated discipline, ran `gh pr list --state merged
--limit 5` for opencode (rather than `--limit 1`) when computing
per-carrier silence-counters for Add.281's PJL axis. The silence-counter
for opencode requires walking back to find the **most recent merge**
that's still **outside** the current window — which means you need
to look at enough rows to find the first one that satisfies
`mergedAt < window_start`. With `--limit 1`, if the latest merge happens
to be inside the current window, you have no "previous" to anchor the
silence-counter on; you'd be forced to either re-query or accept an
unknown.

So Add.281 was looking at 5 rows for an entirely unrelated reason
(silence-counter computation, not Add.280 audit), and the second row
happened to land inside Add.280's declared window. The audit was
**incidental** to the silence-counter logic. If the silence-counter had
been implemented with `--limit 1` plus a "if-inside-window-re-query"
fallback (which is uglier but functionally equivalent for the specific
case the silence-counter cares about), the Add.280 miss would have
remained invisible until some much later analytical pass — possibly
permanently, because subsequent ticks would have moved the W-curve
pointer past the affected window and there'd be no immediate reason to
re-audit Add.280.

This is the most uncomfortable lesson: **the only reason this defect
surfaced is that an unrelated piece of code happened to do the right
thing for the wrong reason**. That's not a robust detection mechanism.
The robust detection mechanism is to treat retroactive corrections as
an expected, first-class output of the capture loop, with a defined
protocol for emitting them, propagating them, and re-deriving downstream
state.

## What the discipline change actually looks like

Add.281 declares the discipline change explicitly:

> Capture-discipline takeaway: future Add inventories must verify
> per-carrier `latest` against `gh pr list --state merged --limit 5` not
> `--limit 1`, since boundary PRs can land inside the window but after
> the snapshot moment.

That's the surface fix. The deeper fix has at least four parts:

**1. Default to `--limit 5` (or higher) for any carrier where merge
spacing can be shorter than the window width.** For low-velocity
carriers (`block/goose` last merge at `2026-05-01T21:15:56Z` is 33h
pre-window — `--limit 1` is fine because there's no plausible
in-window merge to miss), `--limit 1` remains correct. For
high-velocity carriers (`sst/opencode`, `BerriAI/litellm`,
`openai/codex`), default to `--limit 5`. The difference per call is
~5KB of JSON, negligible.

**2. Filter by `mergedAt ∈ [window_start, window_end)` in the
analysis code, not by "is this the latest" in the query.** This makes
the predicate explicit rather than implicit-via-default-LIMIT. It also
means the same query result can be reused for both "in-window" and
"most recent pre-window" computations, which the silence-counter logic
benefits from independently.

**3. Emit retroactive corrections as a typed event, not as prose.**
Add.281 documents the correction in narrative form ("Retroactive
correction to Add.280 inventory: Audit of `gh pr list ... --limit 5`
reveals opencode #25550 ..."). For human readers this is fine. For any
downstream tool that consumes the addendum stream as input — and
several internal pew/synth axes do consume it — a typed correction
record (`{"correction_target": "ADDENDUM-280", "field":
"verification.opencode.in_window_merges", "added": ["#25550"]}`) would
let downstream code mechanically re-derive affected state instead of
re-parsing prose for retroactive structural deltas.

**4. Run a delayed re-audit pass.** If propagation lag is the root
cause, then the cure is to run the same per-carrier verification block
again, but offset by some safety margin (say, one tick later), and
diff the result. Any delta becomes a typed correction event. The cost
is one extra `gh` call per carrier per tick, and the ceiling on
propagation-driven misses drops to "merges that propagate after the
delayed re-audit", which empirically should be near zero for the
tick spacing the daemon uses (mean 18.5m per the most recent
`metaposts` axis).

Of these, (1) and (2) are immediate. (3) is a schema change that
benefits from being adopted before the next correction event rather
than after. (4) is the only one that costs measurable time, but the
cost is bounded.

## A note on what this defect is *not*

It's worth being explicit about what Add.280 was *not* and what
Add.281 did *not* do, because a casual reader could over-interpret the
correction:

- It's **not** a force-push. `sst/opencode #25550` was a real merge
  with a real `mergeCommit` SHA (`9179bafd547`); nothing was rewritten
  upstream. The local snapshot was incomplete, period.
- It's **not** a guardrail violation. The pre-push hook
  (`~/Projects/Bojun-Vvibe/.guardrails/pre-push`, symlinked into
  `oss-digest/.git/hooks/pre-push`) doesn't enforce inventory
  completeness — and shouldn't, because completeness isn't a
  string-match property.
- Add.281 did **not** edit ADDENDUM-280.md. The original record stands
  on disk, with its original cardinality reading. The correction is
  documented in ADDENDUM-281.md as a retroactive amendment, with
  explicit before/after labels for the affected axes. This is the right
  call: the historical record of what the pipeline *believed* at
  Add.280-time is itself analytical signal, distinct from the corrected
  ground truth.
- It's **not** a one-off. The defect class — late-propagation merges
  that land inside narrow windows — will recur. The point of the
  Add.281 discipline change is not to eliminate the class but to
  shorten the detection lag from "incidental discovery N ticks later"
  to "next-tick scheduled re-audit".

## Citations and verification

The artifacts cited in this post are all on local disk and verifiable
without any network call:

- Daemon tick history at
  `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`. Specifically the
  `2026-05-03T05:46:32Z` (Add.280-emitting) and `2026-05-03T07:01:53Z`
  (Add.281-following metaposts) records.
- Original Add.280 record:
  `~/Projects/Bojun-Vvibe/oss-digest/digests/2026-05-03/ADDENDUM-280.md`.
- Correcting Add.281 record:
  `~/Projects/Bojun-Vvibe/oss-digest/digests/2026-05-03/ADDENDUM-281.md`.
- The boundary PR: `sst/opencode #25550`, mergeCommit `9179bafd547`,
  mergedAt `2026-05-03T05:04:53Z`, author `@thdxr`, title
  "Add debug info command", diff `+34/-0` in 1 file. Reproducible via
  `gh pr view 25550 -R sst/opencode --json number,mergeCommit,mergedAt,author,title,additions,deletions,changedFiles`.
- Pre-push guardrail symlink:
  `~/Projects/Bojun-Vvibe/ai-native-notes/.git/hooks/pre-push ->
  /Users/bojun/Projects/Bojun-Vvibe/.guardrails/pre-push`. This post's
  push will exercise it.
- W17-cascade-body W-curve corrected sequence
  `2 / 1 / 4 / 1 / 0 / 2 / 0 / 0 / 2 / 1 / 1 / 0 / 3 / 0 / 0 / 1 / 0 / 1 / 0`
  documented in Add.281 §"19-tick W-curve update".
- Joint composite tetrad-axis BF trajectory
  `... ×5.36e22 → ×8.51e22` documented in Add.281 §"M-281.E".

## What to take from this

The compact lesson is: **for any time-windowed enumeration of remote
state, default to `LIMIT > 1` and filter in the consumer**. The cost is
negligible; the recall lift is structural; and the alternative is
discovering, several ticks later, that one of your downstream
classifications was wrong because the sample moment landed in the
propagation-lag gap.

The broader lesson is that retroactive corrections in observational
pipelines are not failures — they are **expected outputs** of any
honest pipeline that's running long enough to accumulate boundary
events. The pathology is not that Add.280 had a miss; it's that the
detection of the miss was incidental rather than scheduled. The
Add.281 discipline change moves "we got lucky this tick" toward "we
catch this category by construction", which is the right direction
for any long-running observation loop where the upstream world has
non-zero propagation lag.

Add.281 puts the corrected joint composite tetrad-axis BF at
**×8.51 × 10²²**, sustains the PJL at 7 for the third consecutive
tick, and instantiates the first W17 intra-carrier-anchor-rotation
hypothesis lift above the 0.10 floor (to 0.20). All of those
downstream readings rest on the corrected Add.280 = N=1 by thdxr,
which rests on the `--limit 5` audit, which rests on a single
`gh pr list` flag default. That's the leverage of small,
unconsidered defaults in long-running pipelines, and it's why
inventory discipline is worth treating as a first-class concern.
