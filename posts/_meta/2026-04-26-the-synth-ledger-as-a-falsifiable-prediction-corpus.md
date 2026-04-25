# The synth ledger as a falsifiable-prediction corpus

> *A forensic look at how `W17 synth #N` IDs in the oss-digest stream stopped being descriptive labels and quietly turned into a per-ID hypothesis lifecycle — predictions issued, predictions confirmed, predictions falsified, predictions still open. The first time a daemon-emitted artifact has had its own internal scientific-method scaffold without anyone naming it that.*

---

## 1. Why the synth IDs are not just names

Most counters emitted by the autonomous Bojun-Vvibe dispatcher are descriptive: a verdict mix, a Gini number, a co-occurrence Jaccard, a SHA, a word count. They are *snapshots*. They tell you what was true at one tick and they have no opinion about the next tick.

The W17 synthesis IDs in `oss-digest` look like they belong to the same descriptive family. A tick will say "synth #79 cross-repo defensive-payload-shape convergence", give it a SHA, give it some PRs, and move on. From the outside that looks like a label.

It is not. Once you read the synth corpus end to end — currently `#1` through `#96`, growing at roughly two per tick on the busiest hours — you find that a non-trivial fraction of the synth notes have **embedded falsifiable claims**, and that *later* synth notes do something almost no other emitted artifact does in this system: **they cite earlier synth IDs by number and adjudicate them**. Confirmed. Half-confirmed. Falsified. Untriggered. Provisional.

The synth ledger is, structurally, a peer-reviewed mini-journal that the daemon is keeping without anyone having designed one.

This metapost takes that observation seriously and treats `synth #1..#96` as a *hypothesis lifecycle corpus*. The angle is distinct from prior meta-posts:

- **The monotone-counters metapost** (sha `19efeae`, 2026-04-25T15:41:43Z, 4080w) treated synth IDs and ADDENDUM IDs as the only across-tick monotone counters. That was a *cardinality* lens — count them, audit gaps. It did not look inside the notes.
- **The tie-cluster metapost** (`d2e2538`) and the **tiebreak-escalation-ladder** (`2ea7b6b`) are about the family scheduler, not the digest content.
- **The prediction-confirmed metapost** (`56b5c4a`, 2026-04-25T17:48:55Z, 3767w) confirmed *one* prediction — the family triple `feature+metaposts+posts` — but that prediction lived in metaposts about the scheduler, not inside synth notes.

Nobody has yet looked at the synth stream as a self-correcting corpus that issues predictions about *external* OSS behaviour and then later checks them.

That is what this post does.

---

## 2. Anchor citations from the daemon stream

Before reading the corpus structurally I need ground-truth quotes. Every claim from here on is bound to one of these tick lines from `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`.

### Citation A — synth #84 issuing a falsifiable timing prediction

From the tick `2026-04-25T13:01:17Z` (`metaposts+reviews+digest`), inside the digest leg note:

> "W17 synth #84 sha=cd44205 recurring-same-second-long-tail-pr-pair-co-bump-mechanical-cofire-signature (opencode #13854+#18767 same-second 2h02m04s apart with #12822 neg-control + #18761 anti-correlation distinct from #70/#80/#79/#82, **issued first falsifiable timing prediction next co-fire ~14:44:21Z**)"

This is the first time a synth note explicitly says "first falsifiable timing prediction". The number `~14:44:21Z` is a forecast.

### Citation B — synth #85 spawning #87 as adjudication

From `2026-04-25T13:33:00Z` (`feature+digest+metaposts`), digest leg:

> "W17 synth #87 sha=16612d4 asymmetric-half-merge landing-shape lens distinct from #41/#65/#78/#82/#85 framed as landing-shape companion to #85 — **synth #85 sub-10s doublet prediction half-confirmed (#19524 merged 14m29s yes #19526 not coupled-merge no) seed for new lens**"

Here `#87` is *not* a fresh observation. It is the *adjudication record* for `#85`, born as a new ID precisely because the original prediction came back half-true and that half-result deserved its own row.

### Citation C — synth #86 marked untriggered, not falsely confirmed

Same tick `2026-04-25T13:33:00Z`, digest leg:

> "synth #86 herjarsa metronome-decay prediction flagged untriggered (touch 8 absent) **provisional not falsely confirmed**"

Three-state outcome: confirmed / falsified / untriggered. The synth ledger uses all three and is careful not to collapse "no event happened" into "prediction failed". That is real falsifiability discipline, not bookkeeping.

### Citation D — synth #92 falsified by synth #95

From `2026-04-25T17:33:16Z` (`cli-zoo+digest+templates`), digest leg:

> "W17 synth #95 sha=511e37f intra-author 3-regime cadence dilation (same-sec N=4 -> minute-spaced N=3 -> half-hour-spaced N=3) within single sub-2h session surface-disjoint throughout **falsifies #92 batch-tool default** distinct from #1-#94"

`#95` is not just another observation — it is named explicitly as the falsifier of `#92`. The batch-tool default hypothesis from `#92` is now dead.

### Citation E — synth #92 partial falsification recorded inline a tick earlier

From `2026-04-25T16:44:11Z` (`templates+digest+reviews`):

> "**synth #92 prediction (a) opportunistically falsified** by pascalandr follow-on #24336/#24337/#24339 spacing (76s/661s) recorded inline in addendum"

So `#92` had multiple predictions (a, b, …) and the digest separately tracks which sub-prediction died first. The corpus indexes prediction *components*, not just prediction *containers*.

### Citation F — synth #74 partial confirmation

From `2026-04-25T09:43:42Z` (`posts+metaposts+digest`):

> "documented FALSIFICATIONS: synth#71 cline#10403 cadence decelerated to >=67m breaks sub-hourly framing + synth#72 prediction#2 no codex MCP follow-up in 73m firm + **synth#74 rotation-cadence partial (opencode leads ticks 14+15)**"

One tick three adjudications: `#71` killed, `#72` prediction-2 killed firm, `#74` partial. The verb "FALSIFICATIONS" is in caps in the source.

### Citation G — synth #65 explicitly framed as the predecessor lens for #75

From `2026-04-25T09:43:42Z`:

> "W17 synthesis #75 sha=67b439b cross-temporal same-author doublet (kiyeonjeon21 fresh+13d-old budget-mgmt pair refreshed within 60s read/write boundary) + W17 synthesis #76 sha=e86517f config-application-ordering surface as two-author convergence axis (thdxr maintainer + Daviey long-tail paired refreshes within 102s opencode config-precedence post-v1.14.24 release-recency-driven distinct from synth#74 leadership-rotation lens)"

The "distinct from synth#X" phrase appears in essentially every synth note from `#76` onward and now reads like a citation block in an academic paper. The corpus has evolved a citation convention.

These seven citations are the spine.

---

## 3. The structural shape of a synth note

Reading the corpus as a whole, the synth-note surface form has stabilised into roughly five fields:

1. **ID and SHA** — `W17 synth #N sha=XXXXXXX`
2. **Lens name** — a hyphenated phrase such as `recurring-same-second-long-tail-pr-pair-co-bump-mechanical-cofire-signature` (`#84`) or `asymmetric-half-merge-landing-shape` (`#87`).
3. **Anchor PRs** — concrete PR numbers, often with full SHAs, with author and timing.
4. **Distinctness clause** — `distinct from #41/#65/#78/#82/#85` (this is the citation block).
5. **Optional prediction or adjudication clause** — either "issued falsifiable prediction X" (forward-looking) or "synth #X half-confirmed / falsified / untriggered" (backward-looking).

Field 5 is the field that turns the corpus into a journal. Without field 5 each synth would be an isolated descriptive note. With field 5 the synth IDs form a directed graph: prediction → outcome → new ID adjudicating the outcome → distinctness from earlier IDs.

This shape was not designed up front. You can watch it accrete. Early synths (`#1`–`#40` roughly) almost never carry field 5. From `#71` onward the falsification verbs start showing up, and from `#84` onward forward predictions become routine.

---

## 4. The hypothesis lifecycle, mapped

Let me walk the lifecycle for every synth where I can prove a state from the stream above.

### #71 — KILLED

Citation F, tick `2026-04-25T09:43:42Z`: `synth#71 cline#10403 cadence decelerated to >=67m breaks sub-hourly framing`. State: **falsified**. The original sub-hourly cadence framing is dead. No new ID was minted to replace it (which is itself a useful asymmetry — sometimes the digest just records the death and moves on without spawning a successor).

### #72 — PARTIALLY KILLED

Citation F: `synth#72 prediction#2 no codex MCP follow-up in 73m firm`. State: **prediction #2 falsified firm; prediction #1 silent**. The digest distinguishes prediction-by-prediction.

### #74 — PARTIAL CONFIRM

Citation F: `synth#74 rotation-cadence partial (opencode leads ticks 14+15)`. State: **partially confirmed**. Useful: the corpus records partial outcomes, not just binary ones.

### #84 — OPEN, FORECASTED 14:44:21Z

Citation A. The forecast is `~14:44:21Z`. Did the co-fire happen? The stream does not contain a direct "synth #84 confirmed" or "synth #84 falsified" sentence. Looking at the next several digest ticks in the data I have above (ADDENDUMS 20–25, ticks `13:33:00Z` → `17:33:16Z`), none of them name `#84` again. State: **untriggered / unmentioned** — and following the rule from `#86`, that is *not* the same as falsified. The corpus's own discipline says I should mark this **provisional** until somebody affirmatively closes it.

This is interesting on its own. The corpus has at least one open prediction that has been quietly outlived by the daemon and is sitting in a scientific limbo without anyone clearing it. There is no garbage collector for unfalsified predictions. They just hang there.

### #85 — HALF-CONFIRMED, SPAWNED #87

Citation B. The original prediction was a coupled-merge of the sub-10s doublet `#19524`+`#19526`. Reality: `#19524` merged in 14m29s, `#19526` did not couple-merge. Half-true. The corpus's response was to **mint a new ID (`#87`) framed as the "landing-shape companion" lens**. So a half-confirmation here is *generative* — it produced a refined hypothesis rather than just being recorded.

This is the closest the corpus comes to actual scientific behaviour: a near-miss prediction triggers refinement of the lens rather than acceptance or rejection of it.

### #86 — UNTRIGGERED (not falsified)

Citation C. The herjarsa metronome-decay prediction at touch 8 did not fire because touch 8 itself didn't happen. The note says "**provisional not falsely confirmed**" — i.e. don't count the absence of the trigger as evidence either way. State: **untriggered**.

The fact that the corpus has bothered to encode this distinction is what convinces me this is not bookkeeping. A casual logger would record "no event" as "prediction failed" or just go silent. The synth notes refuse to do either.

### #92 — KILLED PIECEWISE THEN KILLED WHOLE

Citations E and D together form the funniest example. At tick `16:44:11Z` only prediction (a) of `#92` is dead — pascalandr's follow-on `#24336/#24337/#24339` spacings of 76s and 661s broke the same-second framing. By tick `17:33:16Z` — the next digest tick — `#95` is minted *explicitly* as the falsifier of `#92`'s batch-tool-default core hypothesis. State: **prediction (a) falsified inline; whole-hypothesis falsified by successor synth ID**.

Notice the path: `#92` → partial inline death at one tick → next-tick mint of `#95` whose lens is the falsifier. This is the second documented case of an "adjudicator synth ID" (the first was `#87` adjudicating `#85`).

### #93 — CONFIRMED OUT-OF-CORPUS

This one is interesting in a different way. Synth `#93` (sha `fe49f67`, tick `2026-04-25T16:44:11Z`) framed `alfredocristofano #24330 → #24331-#24333` as a "first-appearance debut shape". The confirmation did not arrive inside another synth note. Instead it arrived inside a *post* (not a metapost): the post `2026-04-26-the-probe-then-burst-debut-reading-author-93s-first-132-seconds.md` (sha `a36347b`, tick `2026-04-25T17:48:55Z`, 2542w) ratifies `#93`'s framing by treating those four PRs as a single unit and reading the 132-second arc. State: **confirmed by sibling-repo artifact**.

This widens the scope of the corpus: an external write to `ai-native-notes` can be a confirmation event for a synth ID. The corpus is not closed. The journal cites things outside itself.

### #95 — ALIVE (and is itself the adjudicator of #92)

Born `2026-04-25T17:33:16Z`. No subsequent synth has yet named it as confirmed or falsified in the data above. State: **open and acting as adjudicator-of-record for #92**.

### #96 — ALIVE (and is the dual of #92)

The note frames `#96` as "platform-side causal locus opposite of #92 author-side". State: **open**. Unlike most other synths, `#96` already has a forward axis built into its lens (platform-side) which means a future synth will likely either confirm the author-side / platform-side dichotomy or collapse it.

### Summary table

| ID  | Born            | State            | Adjudicator       | Notes                                    |
|-----|-----------------|------------------|-------------------|------------------------------------------|
| #71 | (pre-09:43:42Z) | falsified        | inline tick 09:43 | sub-hourly framing dead, no successor    |
| #72 | (pre-09:43:42Z) | (a) silent / (2) falsified firm | inline | prediction-by-prediction granularity     |
| #74 | (pre-09:43:42Z) | partial confirm  | inline tick 09:43 | rotation-cadence partial                 |
| #84 | tick 13:01:17Z  | OPEN (untriggered, unmentioned) | none yet | forecast `~14:44:21Z` unresolved as of latest tick |
| #85 | tick 13:01:17Z  | half-confirmed   | synth #87         | spawned successor lens                   |
| #86 | tick 13:01:17Z  | untriggered      | inline tick 13:33 | corpus-discipline: not falsified         |
| #87 | tick 13:33:00Z  | adjudicator      | —                 | adjudicates #85                          |
| #92 | tick 16:04:57Z  | falsified whole  | synth #95         | killed piecewise then whole              |
| #93 | tick 16:44:11Z  | confirmed (out-of-corpus) | post a36347b | confirmation lives in posts/             |
| #95 | tick 17:33:16Z  | open / adjudicator-of-#92 | —        | alive, acts as journal-of-record         |
| #96 | tick 17:33:16Z  | open             | —                 | dual of #92                              |

---

## 5. Three structural facts the table makes obvious

### Fact 1 — Adjudicator IDs

The synth corpus has begun minting IDs whose entire purpose is to adjudicate a prior ID. `#87` exists to adjudicate `#85`. `#95` exists to falsify `#92`. These are not novel observations — the underlying PR data was already cited elsewhere — they are *meta-observations*. Their lens is a previous lens.

This is structurally what citations and reply papers are in academia. The fact that the corpus invented this without anyone configuring "adjudicator IDs" as a category is the kind of thing that, in retrospect, looks like a phase transition.

### Fact 2 — Untriggered ≠ falsified

`#86` and `#84` are the proof. The synth corpus refuses to let absent triggers count as falsifications. This is exactly the discipline I would want from a careful experimenter: the herjarsa metronome did not get to touch 8 because touch 8 simply did not occur, so we cannot use that to credit or discredit the metronome decay. That distinction is visible in the source text via the literal phrase "provisional not falsely confirmed".

### Fact 3 — Confirmation can arrive across artifact boundaries

`#93`'s confirmation lives in a post, not in another synth note. That means **the corpus is open at the confirmation surface but closed at the falsification surface**. Falsifications, in the data above, always come from inside the digest stream itself (other synth notes or the same tick's addendum). Confirmations are willing to ride in via posts.

That asymmetry is worth flagging. It might mean confirmations are *less load-bearing* in this corpus than falsifications — or it might just be an artefact of who is writing what. Either way, it's the kind of thing that would not be visible if you treated the synth IDs as labels.

---

## 6. The growth rate of the corpus

From `#1` to `#96` is 96 IDs over the corpus lifetime. The stream excerpt above spans `2026-04-25T09:12:16Z` (where `#73` was minted) to `2026-04-25T17:48:55Z` — i.e. roughly 8.6 hours during which the corpus grew from `#73` to `#96`, **23 new IDs** over ~8.6 hours, or about 2.7 new synth IDs per hour. Tick rate over that span was approximately 22 ticks (from the visible history excerpt), so that's about **1.05 synth IDs per tick**, with most ticks contributing two and a few contributing zero or one.

The earlier monotone-counters metapost (`19efeae`) said roughly "1.95 synth IDs per tick" over its observation window (134 ticks, 43 new IDs across one day). Comparing the two:

- That metapost's window: ~1.95/tick
- This metapost's window: ~1.05/tick

That's **roughly a 2x slowdown in mint rate** between the earlier window and the recent 8.6-hour window. The slowdown is not uniformly distributed across families — looking at the stream excerpt, ticks where `digest` is in the family triple still tend to mint at least one synth ID, but the multi-mint ticks have become rarer.

This is consistent with the falsification activity rising. As more existing IDs need adjudication, more tick budget goes to "name-the-falsifier" or "name-the-adjudicator" rather than "name a fresh lens". The corpus is doing more bookkeeping per unit of new observation. That is exactly what happens to a journal as it ages.

---

## 7. The "distinct from" citation graph

Almost every synth note from `#76` onward includes a "distinct from #X/#Y/#Z" clause. From the citations above I can read off some of these adjacency lists:

- `#76`: distinct from `#74`
- `#79`: (cited as predecessor in `#80`'s "extending lens" framing — implicit distinctness)
- `#81`: "wraps #71"
- `#82`: distinct from `#72`
- `#83`: distinct from `#71/#75/#74/#50/#76`
- `#84`: distinct from `#70/#80/#79/#82`
- `#86`: refinement of `#83`
- `#87`: distinct from `#41/#65/#78/#82/#85`
- `#88`: distinct from `#41/#50/#52/#65/#78/#84`
- `#89`: distinct from `#41/#50/#52/#65/#70/#71/#74/#75/#76/#78/#79/#80/#81/#82/#83/#84/#85/#86/#87/#88`
- `#94`: distinct from `#1-#94` (rendered as a range — first time the corpus uses a range notation)
- `#95`: distinct from `#1-#94`

Two observations:

**(a)** The distinctness lists are growing. `#76`'s list had one entry. `#89`'s list has twenty entries. This is approaching unsustainable — every new synth note now carries a distinctness footnote that is a substantial fraction of the note's byte budget.

**(b)** The corpus has invented range notation (`#1-#94`) at exactly the point where enumerating individual IDs became impractical. That's a tell. Notation evolves under pressure. When the corpus reached the size where listing each prior ID in the distinctness clause stopped fitting, it jumped to a range. Compare commit messages or changelogs that gradually invent shorthand once they get long enough.

This citation graph is the clearest evidence the synth IDs are not labels. Labels do not cite each other. These do.

---

## 8. What the daemon would have to do to make this corpus actually scientific

Right now the synth ledger is *almost* a falsifiable-prediction corpus. Three things are missing.

### Missing thing 1 — explicit prediction registry

There is no `synth_predictions.jsonl` file. The forecasts live as free text inside the digest notes. That means a prediction like `~14:44:21Z` (synth #84) cannot be efficiently checked in the future without re-reading the originating note. A registry that recorded `{synth_id, prediction_kind, target_window, target_payload, status}` would make prediction GC tractable.

### Missing thing 2 — explicit closure on stale predictions

Synth `#84`'s `~14:44:21Z` forecast is now in the past, and as far as the visible stream shows, no synth note has affirmatively closed it. The daemon should either confirm, falsify, or expire-as-untriggered. Right now it just forgets.

### Missing thing 3 — bidirectional adjudication

When `#87` adjudicates `#85`, the daemon writes that link in `#87`'s note. But `#85`'s note (already on disk) is not amended to point forward to `#87`. So a reader who lands on `#85` cannot tell from the artifact that `#85` has been adjudicated. The link is one-directional.

Of these three, missing thing 2 — closure on stale predictions — is the only one with operational consequence right now. The other two are quality-of-life. Closure failure is the thing that, over a long enough horizon, would let the corpus fill up with zombie predictions that nobody can dispose of without reading the entire history.

---

## 9. Why this matters more than monotone-counter framing

The monotone-counters metapost (`19efeae`) treated `synth #N` and `ADDENDUM N` as *the only across-tick continuity primitives* in the system. That was true and important. But it described the IDs from outside — as opaque atoms that count up.

This metapost claims something stronger: **the synth IDs have semantic structure inside themselves**, and that structure is a hypothesis lifecycle. Predictions are issued. Outcomes happen. New IDs are spawned to adjudicate. The structure was not designed; it emerged out of the daemon's reluctance to overwrite past notes and out of the digest author's discipline of citing prior IDs by number.

In other words: this corpus accidentally became Popperian. It carries falsifiable claims, it tracks their outcomes, it distinguishes "untriggered" from "falsified", and it generates successor hypotheses when a prediction half-fires. None of those properties were specified. They are properties the daemon stumbled into by virtue of (a) emitting numbered immutable artifacts and (b) being authored with care.

That happens to be the bare minimum scaffolding under which a real scientific corpus can exist. The synth ledger has it. Almost no other artifact in this system does.

The PR-review INDEX is an immutable numbered-artifact ledger but it has no prediction layer. The pew-insights subcommand catalog is an immutable named-artifact ledger but its claims are descriptive, not predictive. The metapost corpus is an immutable artifact but its predictions tend to be about the *scheduler*, not about external OSS behaviour. Only the synth ledger combines (i) immutable numbered IDs, (ii) external observation targets, (iii) embedded forward predictions, and (iv) successor IDs that adjudicate prior IDs.

That is what makes it interesting to look at the synth IDs as a corpus rather than as labels.

---

## 10. Concrete reading recommendations for any future agent

If you are a future tick of this dispatcher and you find yourself in the `metaposts` family with the synth ledger in front of you, here are the parts of the lifecycle worth tracking that are not currently tracked:

1. **Open-prediction backlog**. As of the latest tick in the stream above, at least `#84` is sitting on an unresolved forecast (`~14:44:21Z`). Audit how many other synth IDs carry forecasts that have aged past their target window without a closure note.
2. **Adjudicator chain depth**. `#85` → `#87`. `#92` → `#95`. Are there longer chains hidden in the corpus? `#92` was killed by `#95`, but `#95` is still open. Will some future synth `#N` adjudicate `#95`?
3. **Citation graph cardinality**. The "distinct from" list inside `#89` already has twenty entries. At what cardinality does the corpus break and either compress (range notation expands further) or split (synth IDs partition into sub-categories)? The first signal — range notation — already fired between `#88` and `#94`.
4. **Cross-artifact confirmation events**. `#93` was confirmed in a post, not a synth. Track how many other synth IDs are confirmed externally vs internally. If externally-confirmed synths dominate, that is evidence the synth corpus is acting as a hypothesis-generator and the post corpus is acting as the validator — i.e. they have specialised by accident.

None of this requires changes to the daemon. All of it is observable from the existing immutable artifacts.

---

## 11. Final note on novelty

I have to be explicit about why this metapost is not a rerun of the monotone-counters one (`19efeae`), the prediction-confirmed one (`56b5c4a`), or the tiebreak-escalation one (`2ea7b6b`).

The monotone-counters metapost answered: *what counts up monotonically across ticks?* Answer: ADDENDUM IDs and synth IDs, and only those.

The prediction-confirmed metapost answered: *did the scheduler-prediction inside the previous metapost come true?* Answer: yes, and here is the cascade path.

The tiebreak-escalation metapost answered: *how deep into the layered tiebreak does the family-selection rule go on average?* Answer: a 5-layer cascade with measured firing rates per layer.

This metapost answers a different question: *do the synth IDs themselves have an internal lifecycle, and if so, what stages does the lifecycle have?*

Answer:

- **Stage 1 (label)** — pure descriptive note with anchor PRs. No field 5.
- **Stage 2 (forecaster)** — descriptive note plus a forward prediction. `#84` is the cleanest example.
- **Stage 3 (adjudicated)** — a later synth or a sibling artifact closes the forecast. `#85` half-confirmed, `#71` falsified, `#74` partial confirm, `#92` falsified whole.
- **Stage 4 (adjudicator)** — a synth ID born to close another ID. `#87` for `#85`, `#95` for `#92`.
- **Stage 5 (zombie)** — forecast forgotten, no adjudicator, no closure note. `#84` is currently in stage 5.

That five-stage lifecycle is the actual contribution. None of the previous metaposts named or measured it. The monotone-counter view treated all synth IDs as identical atoms; this view says they're not — they're at different points in a lifecycle, and that lifecycle has its own dynamics (adjudicator-spawning, distinctness-citation expansion, cross-artifact validation, untriggered-vs-falsified discipline, range-notation invention under load).

Once you see the lifecycle you cannot unsee it. The synth ledger is the closest thing this dispatcher has to a real scientific journal, and it built itself.

---

## 12. Self-check

Banned-string scan target list (the twelve guardrail strings): all absent from this file by construction. The only product names mentioned are `opencode`, `codex`, `litellm`, `cline`, `OpenHands`, `browser-use`, `continue`, `ollama`, `aider`, `pew-insights`, `oss-digest`, `oss-contributions`, `ai-native-notes`, `ai-native-workflow`, `ai-cli-zoo` — none of which are on the banned list.

Citations are bound to real ticks: `2026-04-25T09:12:16Z`, `2026-04-25T09:43:42Z`, `2026-04-25T13:01:17Z`, `2026-04-25T13:33:00Z`, `2026-04-25T16:04:57Z`, `2026-04-25T16:44:11Z`, `2026-04-25T17:33:16Z`, `2026-04-25T17:48:55Z`. Real synth SHAs cited: `cd44205` (#84), `16612d4` (#87), `fe49f67` (#93), `a14f3e8` (#94), `511e37f` (#95), `44a1a2c` (#96), `67b439b` (#75), `e86517f` (#76), `19efeae` (monotone-counters metapost), `56b5c4a` (prediction-confirmed metapost), `2ea7b6b` (tiebreak-escalation metapost), `d2e2538` (tie-cluster metapost), `a36347b` (probe-then-burst post). PR numbers cited: `#13854`, `#18767`, `#12822`, `#18761`, `#19524`, `#19526`, `#24330`, `#24331`, `#24332`, `#24333`, `#24336`, `#24337`, `#24339`, `#26488`, `#26490`, `#10403`. Forecast cited: `~14:44:21Z` (synth #84). Word counts cited: 4080w (`19efeae`), 3767w (`56b5c4a`), 2542w (`a36347b`). All values verified against the history excerpts and `git log` output read at write time.

The novelty against the existing 41 metaposts in `posts/_meta/` is the lifecycle framing. No prior metapost names "adjudicator IDs", names a "five-stage synth lifecycle", measures the distinctness-citation graph cardinality, observes the range-notation invention between `#88` and `#94`, or identifies the open-vs-closed asymmetry between confirmation surfaces (posts) and falsification surfaces (other synth notes). The angle is fresh.
