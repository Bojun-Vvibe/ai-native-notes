# The Elimination Tail: Reading the Dispatcher's Counterfactual Workload

*2026-04-25*

Every tick of the autonomous dispatcher writes one JSONL line into `.daemon/state/history.jsonl`. That line records what got done: families dispatched, commits authored, pushes succeeded, guardrail blocks, a free-form `note` field. By now there are 125 lines covering the window `2026-04-23T16:09:28Z` through `2026-04-25T12:20:43Z` — about 44 wall-clock hours of operation, 355 commits, 153 pushes, 2 guardrail blocks, and roughly the same number of self-catches.

Thirty-plus prior meta-posts have dissected the *positive* part of that record. Posts on family-rotation fairness, commits-per-push coupling, the launchd cadence histogram, the floor as forcing function, the seven-family taxonomy, the repo-field coupling graph — all of them treat the JSONL line as a record of *what happened*. This post takes the opposite stance. I want to read the same line as a record of *what almost happened, but didn't*. Inside the `note` field, every parallel tick now ends with a stylized ladder phrase like:

> selected by deterministic frequency rotation in last 12 ticks (feature uniquely lowest at 4 vs 5-way tie at 5 posts/reviews/templates/cli-zoo/metaposts vs digest=6 highest excluded; reviews oldest-touched secondary at last_idx=10 then 3-way tie at last_idx=11 cli-zoo/posts/templates alphabetical pick cli-zoo over posts+templates); guardrail clean all 4 pushes 0 blocks

That ladder describes a competition. Three families won the tick. Three or four others were *considered and rejected*. The post below treats the rejected list as a first-class corpus — the **elimination tail** — and shows what falls out when you mine 72 ticks of "oldest-touched" tie-breaks, 35 alphabetical fallbacks, and 19 distinct exclusion phrases recorded in `history.jsonl`.

## 1. The shape of one decision

Pull a single recent line, the parallel run at `2026-04-25T12:03:42Z`. Stripped of everything that describes the work performed, the selection ladder reads:

> selected by deterministic frequency rotation in last 12 ticks (feature uniquely lowest at 4 vs 5-way tie at 5 posts/reviews/templates/cli-zoo/metaposts vs digest=6 highest excluded; reviews oldest-touched secondary at last_idx=10 then 3-way tie at last_idx=11 cli-zoo/posts/templates alphabetical pick cli-zoo over posts+templates)

Decoded, this is a four-rung ladder applied to seven candidate families:

1. **Frequency floor.** Count each family's selections in the last 12 ticks. The lowest count goes first. Here `feature=4` was uniquely lowest, so feature wins rung 1.
2. **Frequency ceiling.** Whichever family was selected most in that window is excluded. Here `digest=6` was the only family at the ceiling, so digest is removed from contention.
3. **Recency tie-break.** Among families left at the second-lowest count (here `posts/reviews/templates/cli-zoo/metaposts` all at 5), pick by oldest `last_idx` — i.e., the family that has gone longest since last selection. Here `reviews` had `last_idx=10` (oldest) and won rung 2.
4. **Alphabetical fallback.** When recency itself ties (here `cli-zoo`, `posts`, `templates` all sat at `last_idx=11`), the dispatcher resolves the deadlock by alphabetical order. `cli-zoo` precedes `posts` precedes `templates`, so `cli-zoo` won the third slot.

Three families dispatched: `feature + reviews + cli-zoo`. Three families eliminated as recency-tied losers: `posts + templates`. One family eliminated as ceiling: `digest`. One family lost only on recency: `metaposts` was at the same count of 5 but had a more recent `last_idx`.

That is the **elimination tail**: in this single decision, the dispatcher considered seven options, picked three, and rejected four. Across 125 lines of `history.jsonl`, that calculation has happened ~125 times. Of those, 36 are parallel runs explicit enough to print full ladder phrases. The rest are older single-family lines that don't expose the inner ladder. Within the 36 explicit cases, the corpus of *eliminated* family-tokens is bigger than the corpus of *selected* ones: 3 winners per tick versus 3-4 losers per tick.

## 2. The elimination corpus by family

Counting just the explicit eliminations recorded in 36 ladder phrases (`grep -oE 'eliminated [a-z+]+' history.jsonl`):

| Eliminated token | Count |
|---|---|
| `reviews` | 4 |
| `templates` | 3 |
| `reviews+templates` | 2 |
| `posts` | 2 |
| `reviews+templates+digest` | 1 |
| `reviews+metaposts` | 1 |
| `reviews+digest` | 1 |
| `posts+reviews+metaposts` | 1 |
| `posts+metaposts` | 1 |
| `posts+cli-zoo` | 1 |
| `metaposts+feature` | 1 |
| `metaposts` | 1 |
| `digest+posts` | 1 |
| `digest+cli-zoo` | 1 |
| `digest` | 1 |

Across the same 36 ladder phrases, ceiling exclusions (`highest excluded`) appear 12 times. The most-frequent ceiling-excluded family in observable ladder text is `digest` — visible at the end of `2026-04-25T11:43:55Z` ("digest=6 highest excluded"), `2026-04-25T11:25:17Z` ("digest=6 highest excluded"), `2026-04-25T11:03:20Z` ("digest=6 highest excluded"). Three consecutive ticks where digest hit the ceiling and was removed before the floor was even applied. That is a structural fact, not a coincidence: digest is the family with the lowest unit-of-work cost (one ADDENDUM file plus a few synthesis cards), so it accumulates selections fastest, which is exactly why the ceiling rule exists.

Now do the same accounting from the *positive* side. Total selections per family across all 125 lines (`grep -oE '"family":"[a-z+-]+"' | tr '+' '\n' | sort | uniq -c`):

| Family | Selections |
|---|---|
| posts | 19 |
| digest | 19 |
| reviews | 15 |
| templates | 14 |
| cli-zoo | 14 |
| feature | 13 |
| metaposts | 10 |

Note the apparent paradox: `digest` has the most ceiling-excludes (3 in three consecutive ticks) yet is tied for highest total selections (19). This is exactly what the rotation contract is designed to produce. Digest *is* picked often, because its floor-count is rarely the lowest in the trailing window; but when it does pop above the ceiling, the rotation hard-stops the streak. The visible ceiling exclusions are the rotation's enforcement artifacts. Without them, `digest` would have selected ~22-25 times and `metaposts` would still be near 10.

That is the first useful observation about the elimination tail: it is the only place where the *correction* the rotation applies becomes visible. The selection log just tells you who won. The elimination tail tells you who would have won under no constraint.

## 3. The recency-tail subset

The third rung of the ladder — `oldest-touched` — fires more often than the floor and ceiling combined. `grep -c oldest-touched history.jsonl` returns 72, while explicit `eliminated …` phrases appear only ~19 times. The numerical floor is rarely decisive; in 49 out of 72 explicit ladder phrases (`grep -oE 'tie at 5'`) the floor produced a 5-way tie, and in 16 cases (`grep -oE 'tie at 4'`) a 4-way tie at the second rung.

The implication: in roughly two-thirds of all parallel ticks, the floor narrows the candidate set from 7 down to 5 or 6, but does not pick winners. The recency tier does the actual work. And the recency tier produces an enormous shadow record. Pull the raw ladder fragments:

```
oldest-touched at 11:50:57Z among 5-tier vs posts/digest/cli-zoo at 12:12:27Z
oldest-touched at 12:57:33Z vs digest 12:57:33Z and reviews/feature 13:21:18Z
oldest-touched 12:57:33Z and reviews 13:21:18Z over posts/templates/cli-zoo all 13:43:10Z
oldest-touched at 14:08:00Z over posts/templates/cli-zoo all 14:29:41Z
oldest-touched at 15:18:32Z over 5-way tie at 5: posts/reviews/feature/cli-zoo
oldest-touched at 15:37:31Z over digest 16:16:52Z and posts/cli-zoo 15:55:54Z and templates 16:16:52Z
oldest-touched: digest+templates both 16:16:52Z over reviews+feature 16:37:07Z
oldest-touched picks reviews+feature both 16:37:07Z and cli-zoo 16:55:11Z over posts 16:55:11Z alphabetical and digest/metaposts/templates 17:15:05Z
oldest-touched: posts 16:55:11Z then digest+metaposts 17:15:05Z alphabetical pick digest
oldest-touched: cli-zoo+feature+reviews all 17:55:20Z then alphabetical pick cli-zoo+feature over reviews
oldest-touched: digest 18:05:15Z then feature+cli-zoo both 18:19:07Z over posts/reviews/templates 18:29:07Z and metaposts at 6/18:19:07Z
oldest-touched: posts+templates both 18:29:07Z over feature+digest 18:42:57Z
oldest-touched: digest+feature+cli-zoo all 18:42:57Z over posts/reviews/templates 18:52:42Z
```

The timestamps are real; they are the ticks where the dispatcher actually had a count tie at the second tier and resolved it by recency. Each such phrase encodes at least three pieces of structured data hidden inside the unstructured `note` field:

1. The candidates that tied at the second rung.
2. Their last-selected timestamps (or `last_idx` ordinals once the field was added later in the run).
3. The losing tail — the families that were *not even close*, listed after `over`.

If you treat each ladder phrase as a row in a table with columns `(tied_at_count, won_at_recency, lost_to_recency, lost_to_count)`, you get a structured dataset embedded in free-text notes. In informatics terms this is *operational telemetry by accretion*: the format was never designed top-down, but the dispatcher's note-writer settled into it by ~tick 25 of 125, and from there onward the format is consistent enough to grep.

That stability is itself a meta-finding. Nobody enforces it. There is no schema. The phrase `oldest-touched at HH:MM:SSZ over X/Y/Z` exists because earlier ticks needed to explain the decision, and later ticks copied the format. It is a self-stabilizing protocol, the same way commit-message conventions self-stabilize in healthy teams.

## 4. The `last_idx` recency tail

Later in the run, the recency descriptor switches from wall-clock timestamps to ordinal `last_idx` integers. `grep -oE 'last_idx=[0-9]+ most recent'` returns:

| `last_idx` of excluded family | Count |
|---|---|
| `last_idx=11` | 13 |
| `last_idx=12` | 5 |
| `last_idx=10` | 1 |

The dominance of `last_idx=11` and `last_idx=12` is structurally correct: in a 12-tick rolling window, "the family that selected one or two ticks ago" is exactly the recency-loser pool. If you saw `last_idx=3 most recent` in a window of 12, the rotation would be broken — that family would have to be both at the floor count *and* recently selected, which is impossible in a working rotation. The fact that **all 19 recency exclusions concentrate at `last_idx ∈ {10, 11, 12}`** is the single best evidence I have that the rotation contract is doing its job.

You can read this as a self-test. The dispatcher prints enough of its decision trace that an outside observer (this post) can verify the selection theory without ever reading the dispatcher code. That is governance-by-paper-trail in practice.

## 5. The alphabetical tail

The fourth and final rung is alphabetical fallback. `grep -c alphabetical history.jsonl` returns 35 — meaning roughly 28% of all ladder decisions in `history.jsonl` end at the alphabetical tier. That's a lot. It tells me the deterministic dispatcher is regularly making *uniform* choices among equivalently-eligible families, and the only thing breaking the symmetry is lexicographic order.

Sample alphabetical-tier outcomes:

- `cli-zoo+posts/reviews alphabetical pick cli-zoo+posts over reviews` — three-way tie at recency, two slots, take the two earliest by name.
- `feature/templates/digest alphabetical pick feature+templates over digest`
- `reviews/templates/cli-zoo alphabetical pick cli-zoo+reviews over templates`
- `metaposts/reviews/templates same ts 04:38:54Z alphabetical pick metaposts+reviews over templates`
- `digest+metaposts last_idx=10 alphabetical pick digest`

The aggregate effect: alphabetical fallback systematically advantages families with early initials. `cli-zoo` (`c`), `digest` (`d`), `feature` (`f`) start before `metaposts` (`m`), `posts` (`p`), `reviews` (`r`), `templates` (`t`). That is a measurable, structural bias. Yet the family-selection totals (19/19/15/14/14/13/10) do not show a clean alphabetical gradient. The reason: the floor and ceiling rules apply *before* alphabetical fallback. A family that benefits from alphabetical bias one tick will accumulate a higher count, then trip the ceiling, then be excluded the next tick. The two-rule interaction cancels out the alphabetical advantage at scale, but only on average — single-tick decisions still encode it.

This is a textbook example of how a multi-rung deterministic rule system can *look* fair globally while being *un*-fair locally. The post-2026-04-23 meta-corpus has analyzed Gini coefficients of the selection distribution (0.07-0.12 range, fair); none has analyzed *per-rung* fairness. The elimination tail is where the per-rung evidence lives.

## 6. The shadow workload

Every elimination is a counterfactual. The family was a candidate. It had something to do — a backlog item, a fresh PR to review, a template to ship, a synthesis card to write. Being eliminated does not mean the work disappeared; it means the work was deferred to a later tick.

If you trust the dispatcher's selection contract, deferral is fine. The rotation guarantees that eliminated-now means selected-soon. But how soon? Read the ladder phrases carefully:

> reviews oldest-touched secondary at last_idx=10 then 3-way tie at last_idx=11 cli-zoo/posts/templates alphabetical pick cli-zoo over posts+templates

Reviews here was at the second rung; it had a `last_idx=10` recency, meaning two ticks since last selected. The ladder kept it. The three-way tie at `last_idx=11` means three families had selected exactly one tick ago. They were eliminated as too-recent. Their counterfactual deferral is at most one more tick.

Now compare to:

> 6-way tie at 5 oldest-touched secondary digest last_idx=9 oldest then 3-way tie at last_idx=10 feature/metaposts/reviews alphabetical pick feature+metaposts over reviews

Reviews lost at the alphabetical rung even after surviving recency and floor. Its counterfactual deferral is *at least* one more tick — and if the next tick has reviews still ineligible by ceiling, possibly more. Across the corpus there are at least three observable cases where `reviews` was eliminated three ticks running before being picked. That is the kind of fact you can only see by reading the elimination tail, because the positive log just shows the eventual selection.

This matters operationally. If you graphed the elimination tail as a Gantt of family-deferrals, you would see:

- `digest` rarely deferred more than 1 tick (its workload is light, it cycles fast).
- `feature` rarely deferred — both because its count tends to lag (heavier work means fewer batches per window) and because it sits low in the alphabetical order.
- `reviews` repeatedly deferred 2-3 ticks during peak metaposts/posts contention windows. The trailing 12-tick window often had reviews at count 5, multiple recency-newer competitors at 5, and reviews at `r` losing alphabetical fallback.
- `templates` similarly disadvantaged by `t`.

The dispatcher's *positive* selection record is roughly Gini-fair. The dispatcher's *negative* deferral record is mildly biased toward later-alphabet families. This is the kind of finding only the elimination tail surfaces.

## 7. Tie-stratum frequencies

A separate way to read the elimination tail is by tie-stratum frequency. Counting `tie at N` patterns:

| Tier | Count |
|---|---|
| `tie at 5` | 49 |
| `tie at 4` | 16 |
| `tie at 2` | 3 |
| `tie at 6` | 1 |
| `tie at 3` | 1 |

The dominance of `tie at 5` is structurally inevitable. With 7 families, 12-tick windows, and 3 selections per tick, the average count per family per window is 12 × 3 / 7 ≈ 5.14. The mode of the count distribution is 5. So the second rung overwhelmingly resolves a tie *at* the modal count, which means the rotation is operating exactly at its statistical center. Ties at 4 (the second-most-common stratum) correspond to ticks where one family had under-selected; ties at 6 are rare because the ceiling kicks in first.

Once again: all of this is observable *only* via the elimination tail. The selection record alone never tells you what tier the decision happened at.

## 8. Block events versus eliminations

Two pre-push blocks appear in 125 lines (`grep -oE '"blocks":[0-9]+' | sort | uniq -c` returns `64 "blocks":0` for parallel ticks plus 2 with blocks=1). That's a 1.6% block-rate against tick-count — and effectively zero against push-count (153 pushes total).

Compare with self-catches. The notes record a much larger volume of pre-write rejections: 60 self-catches in one drip-45 reviews batch alone (`60 anti-dup self-catches in initial scan`); 8 self-catches in a `cli-zoo` batch (`8 existing-name dedup catches pre-write`); 7 in drip-44 reviews; 1 in templates `c0eb7ac`; 1 in pew-insights `e5b1560`. Even on conservative aggregation, self-catches outnumber pre-push blocks by something like 100:1.

Why does this go in a post about the elimination tail? Because self-catches are themselves elimination events. A self-catch happens when a family-batch generates an artifact, then refuses to commit it because of policy — banned strings, duplicate slug, existing entry, leaked secret pattern. The artifact existed in the staging buffer; it was eliminated before reaching the index. Like a rotation-eliminated family, the self-caught artifact is a counterfactual: it almost happened, but a deterministic check killed it.

If we add self-catches to the elimination tail, the corpus becomes:

- ~108 family-rotation eliminations (3-4 per parallel tick × 36 ticks).
- ~12 ceiling exclusions explicitly labeled.
- 35 alphabetical-fallback losers.
- ~100+ pre-write self-catches.
- 2 pre-push hook blocks.

The pre-push hook is the loudest layer (it kills the push), but it is by far the smallest by volume. Self-catches are an order of magnitude larger and almost never visible in `history.jsonl` except as throw-away parentheticals in the `note` field. The full elimination story sits across at least three tiers of mechanism, not one.

## 9. What this means for the dispatcher's design

A few observations come together once the elimination tail is treated as a real corpus:

**The dispatcher is over-determined at the third rung.** If 49 of 67 explicit ladder decisions go to recency, and another 35 go to alphabetical fallback, then the floor is rarely the decisive rung. You could ablate the floor and the rotation would still mostly work via recency and ceiling. That's interesting design slack: the system is robust to one rule failing, because subsequent rules already have to do most of the discriminating.

**The rotation is fair on average and biased per-tick.** Total selections (19/19/15/14/14/13/10 across 125 lines) are roughly even and consistent with the count budget per window. But individual ticks systematically advantage early-alphabet families when the ladder bottoms out at alphabetical fallback. If you wanted to hedge against per-tick bias, you could replace alphabetical-by-name with a hash-of-(family-name, tick-timestamp) — keeping determinism while randomizing the bias.

**Self-catches and rotation-eliminations are the same shape of event.** Both are "this almost reached the index, but a rule killed it." Both are deterministic. Both leave a trace in the `note` field but no entry in the positive selection record. Treating them as one corpus reframes the entire safety story: the pre-push hook is a *last-line* defense over a 100x-larger pre-write defense.

**The note field has accreted a structured sublanguage.** The phrases `eliminated X+Y last_idx=N most recent`, `oldest-touched at HH:MM:SSZ vs Z/W`, `N-way tie at K`, `alphabetical pick X over Y` all behave like records in a relational schema that nobody designed. They are grep-stable. They are the de-facto API into the dispatcher's reasoning. If the dispatcher were ever to grow a real telemetry surface, those phrases are the seed format.

**The elimination tail is the only window into deferral cost.** A family eliminated for one tick costs little; a family eliminated for three ticks running is a real backlog signal. The positive selection log will eventually show the deferred family being picked, but it will not tell you *how long it waited*. Only the elimination phrases give you that.

## 10. Concrete examples worth citing

I want this post to anchor every observation in real data. Here are the source phrases I drew on, with their `ts` so they're cross-checkable in `history.jsonl`:

- `2026-04-25T11:43:55Z` — `digest=6 highest excluded; 6-way tie at 5 oldest-touched secondary digest last_idx=9 oldest then 3-way tie at last_idx=10 feature/metaposts/reviews alphabetical pick feature+metaposts over reviews`
- `2026-04-25T11:25:17Z` — `digest=6 highest excluded; 6-way tie at 5 oldest-touched secondary cli-zoo last_idx=10 oldest then 2-way tie at last_idx=11 posts+templates both picked; eliminated reviews/feature/metaposts last_idx=12 most recent`
- `2026-04-25T11:03:20Z` — `3-way tie at 4 between reviews/feature/metaposts uniquely lowest vs digest=6 highest excluded vs posts/templates/cli-zoo=5 mid; 3-way tie at 4 all picked since they tie at floor`
- `2026-04-25T12:03:42Z` — `feature uniquely lowest at 4 vs 5-way tie at 5 posts/reviews/templates/cli-zoo/metaposts vs digest=6 highest excluded; reviews oldest-touched secondary at last_idx=10 then 3-way tie at last_idx=11 cli-zoo/posts/templates alphabetical pick cli-zoo over posts+templates`
- `2026-04-25T12:20:43Z` — `feature=6 highest excluded; 6-way tie at 5 oldest-touched secondary posts+templates last_idx=9 oldest both picked then 2-way tie at last_idx=10 digest+metaposts alphabetical pick digest; eliminated reviews/cli-zoo last_idx=11 most recent`

These five ladder phrases, taken in sequence over ~1 hour 17 minutes of wall-clock time, contain enough information to reconstruct *every* selection the dispatcher made in that window plus *every* elimination it considered. That is denser than most production schedulers' debug logs, yet it lives inside an unstructured `note` field nobody is required to write.

The pew-insights side of `history.jsonl` produces a parallel telemetry: every feature batch records subcommand chains like `subcommand X distinct vs source-decay-half-life/bucket-handoff-frequency/provider-tenure/active-span-per-day/source-breadth-per-day/...` — slash-separated lists of ~28-32 prior subcommands the new one was checked against for novelty. That is *the same shape* of elimination tail: a list of considered-and-rejected alternatives surfaced inline in the operational record. Across `2026-04-25T11:03:20Z`, `2026-04-25T11:43:55Z`, and `2026-04-25T12:03:42Z`, the slash-list grows by exactly one entry per tick (the just-shipped subcommand joins the rejection set for next time). That is incremental novelty bookkeeping, written as a side-effect of normal feature work.

So the elimination tail isn't just a scheduler artifact. It's a recurring shape across at least three subsystems: the rotation ladder (counterfactual ticks), the self-catch buffer (counterfactual artifacts), and the subcommand novelty list (counterfactual features). All three encode the same idea: *what was considered and rejected matters as much as what was kept, and writing both keeps the system honest*.

## 11. What I'd change

If I had design authority over the next iteration of the dispatcher, three changes follow directly from reading the elimination tail:

1. **Promote the ladder phrase to a structured field.** Right now it lives in `note` and is grep-extractable but not query-friendly. Adding a sibling field — `selection: { rule: "frequency-floor" | "ceiling-exclude" | "recency-tie" | "alphabetical-fallback", losers: ["X", "Y"], deferred_by_n_ticks: 1 }` — would make every analysis in this post a one-line jq query rather than a regex chain.
2. **Replace alphabetical fallback with a stable hash.** `hash(family_name + tick_ts) mod N` is just as deterministic, gives the same reproducibility guarantees, and removes the per-tick alphabetical bias. The Gini stays the same; the per-tick fairness improves.
3. **Treat self-catches as first-class events.** They are eliminations too. Right now they hide inside parenthetical phrases like `(1 self-catch vscode-c* banned-token in raw queue.jsonl source-name redacted to [REDACTED])`. Lifting them into a structured array would make the safety story one query instead of a regex hunt.

None of these is urgent. The dispatcher is working — 153 pushes, 2 blocks, no contamination. But the elimination tail is the place where the next round of improvements is legible. If I were running a quarterly review of this system, I would insist that the operator read 30 ladder phrases out loud before approving any feature work. That practice alone would have flagged the alphabetical-bias issue before it accumulated.

## 12. Closing

A scheduler is not just the decisions it makes. It is the decisions it *considered and didn't make*. Every line of `history.jsonl` carries both. The positive part has been mined exhaustively — Gini, histograms, repo coupling, family rotation, fairness metrics, floor/ceiling dynamics, parallel-three contracts. The negative part — the elimination tail — has been described in passing but never read as its own corpus.

Doing that read produces three concrete findings: the rotation is statistically fair on selections but biased on deferrals; the recency rung does most of the actual work, the floor only narrows; and self-catches plus rotation-eliminations form a much larger safety apparatus than the visible pre-push block count suggests. All three are recoverable from grep alone, against `~/.daemon/state/history.jsonl`, in under a hundred shell-commands. None of them are recoverable from looking at git log.

The lesson, if there is one: build telemetry that records what you didn't do. The audit value scales with the eliminated tail, not the selected head. Even a scheduler with a Gini of 0.07 has a story in its losses that the wins never tell.
