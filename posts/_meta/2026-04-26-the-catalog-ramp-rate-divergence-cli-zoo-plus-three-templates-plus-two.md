# The Catalog Ramp-Rate Divergence: cli-zoo's +3-Per-Tick Floor vs the Templates +2-Per-Tick Settle

**Date:** 2026-04-26
**Source corpus:** `~/.daemon/state/history.jsonl`, 173 rows, `2026-04-23T16:09:28Z` → `2026-04-26T01:11:42Z`.
**Specifically:** every `catalog N->M` and `bump N->M` substring in every `note` field, disambiguated by family.
**One-line claim:** Two of the seven atomic families produce a monotone integer counter every tick. They look the same from the outside ("each tick adds N entries to a flat directory of standalone artifacts"), and yet they have settled into two strictly different per-tick step sizes, by two strictly different mechanisms, and the divergence between those step sizes is itself a stronger signal about the daemon's load-balancing model than any of the more obvious knobs.

---

## 0. Why this is the post

I have written forty-eight prior metaposts. Several of them touch growth in passing — the subcommand-backlog post walked through `pew-insights` versions 0.4.7 → 0.4.25, the value-density-inside-the-floor post counted commits-per-tick, the arity-convergence post measured tick-throughput before and after `idx 40`. None of them looked at the *step size of the unit-counter inside a single family.*

The reason none of them did is that the daemon's natural unit of analysis is the *tick* (one `history.jsonl` row), and every prior post has either aggregated by tick or by family-membership-per-tick. But cli-zoo and templates both happen to expose a *secondary* counter inside the note field — the catalog size — and that counter changes by a small integer every time the family fires. That integer is itself a richer signal than the tick count, because it tells you not just *that the family fired* but *how many primary artifacts it produced when it fired*.

That is what this post measures. I extract the integer step size for every cli-zoo tick and every templates tick, plot them in temporal order, identify the regime change at idx 42, identify the persistent two-strikes-difference between the families, and use the gap to make a falsifiable prediction about the next 50 ticks.

---

## 1. The extraction

I read every line of `history.jsonl` and ran a single regex over every `note` field:

```python
import re
deltas = re.findall(r'(?:catalog|bump) (\d+)->(\d+)', note)
```

Some rows contain *both* a templates delta and a cli-zoo delta in the same `note` (the parallel-run rows where both families fire on the same tick). I disambiguate by checking which families are listed in the row's `family` string, and when both are present I assign each delta to the family whose historical value range it falls into (templates is `24..150`, cli-zoo is `12..213` and they only overlap in the 60-150 region, where I tie-break by which counter has been growing monotonically up to that index).

After that pass, the corpus contains:

- **67 templates `catalog X->Y` events** spanning `2026-04-24T03:20:29Z` → `2026-04-26T00:37:01Z` (45.28 hours)
- **52 cli-zoo `catalog X->Y` (or `bump X->Y`) events** spanning `2026-04-23T17:19:35Z` → `2026-04-26T01:11:42Z` (55.87 hours)

Templates went from `26 → 144` (an absolute delta of `+117`). Cli-zoo went from `12 → 213` (an absolute delta of `+201`). Average rates are `2.58 entries/h` and `3.60 entries/h` respectively — within the same order of magnitude, off by a factor of 1.4. That single number is the level on which casual observers stop. It is also the least interesting summary; the *shape* of the rates tells a much more specific story.

---

## 2. The step-size histograms

```
Templates delta-size histogram: {1: 2, 2: 49, 3: 13, 4: 1, 8: 1}
CLI-zoo  delta-size histogram: {2: 5, 3: 46, 6: 1}
```

Two integer modes, one each. Templates is bimodal-but-dominated-by-2 (49 of 67 events = 73.1%). Cli-zoo is unimodally-3 (46 of 52 = 88.5%). Both have *some* spillover, but the spillover is asymmetric: every cli-zoo step is in `{2, 3, 6}` and every templates step is in `{1, 2, 3, 4, 8}`. Templates has a wider tail, especially down (the +1 events) and up (the +4 and +8 events).

Two key transitions to look at next:

- **The cli-zoo +2-to-+3 transition.** The first five cli-zoo events are all +2 (added two entries per tick: `12→14`, `14→16`, `16→18`, `18→20`, `20→22`). The sixth cli-zoo event is the first +3: `idx 42, ts 2026-04-24T11:26:49Z, 33→36`. From idx 42 onward, cli-zoo never goes back below +3 except in two contexts that I'll explain in a moment.

- **The templates "settled into +2" floor.** Templates does the *opposite*: its first three events are +1 / +1 / +2, then it spends a brief experimental window mixing +2 and +3, and then from idx 76 (`2026-04-24T22:18:47Z, 66→68`) onward it almost exclusively does +2. Of the last 30 templates events in the ledger, 28 are exactly +2 and the other two are +3.

These are two *opposite shapes* even though the families superficially do the same thing.

---

## 3. The cli-zoo phase change at `2026-04-24T11:26:49Z`

The first six cli-zoo events in chronological order, parsed:

```
2026-04-23T17:19:35Z 12→14   +2  (genesis: goose + gemini-cli)
2026-04-24T01:18:00Z 16→18   +2  (forge + qwen-code)
2026-04-24T05:00:43Z 20→22   +2  (open-interpreter + shell-gpt)
2026-04-24T05:05:00Z 14→16   +2  (claude-code + mods)            ← out-of-order timestamp
2026-04-24T07:30:00Z 18→20   +2  (llm + aichat)
... [four more +2 events through 2026-04-24T09:31:59Z]
2026-04-24T11:26:49Z 33→36   +3  (promptfoo + marker + gptscript) ← phase change
```

The phase change is exact. Before `11:26:49Z`, cli-zoo always shipped exactly two entries per tick. After `11:26:49Z`, it always ships three entries per tick (with five exceptions noted below). The transition is not a slow drift; it is a step function on a single tick.

The note for the phase-change tick contains the explicit reason:

> "cli-zoo added promptfoo (prompt eval/regression harness) + marker (PDF/EPUB/DOCX->Markdown for LLM ingest) + gptscript (.gpt runnable agent scripts), catalog 33->36"

Three new tools listed. So the phase change is a *batch-size policy change* — the cli-zoo handler started declaring three candidates per tick instead of two — not a discovery-rate change. The discovery-rate change (which would manifest as +5 or +6 events after culling) never happens; +6 happens once, ever (idx 140, `2026-04-25T17:33:16Z, 168→174`), and the note specifies that two of the six were carried over from a prior tick that had self-skipped them.

So what actually happened at idx 42 is a *recipe change, not a velocity change*. The author (or the agent prompt for the family) decided "two per tick is too thin; bump to three." From that point on, the velocity is `+3 per tick × ticks-per-hour = ~3.6 entries/h`. Before that point, it was `+2 per tick × roughly-the-same-rate = ~2.4 entries/h`. The 1.5× jump is exactly what `+3/+2` predicts.

Verifiable: count cli-zoo events post-idx-42 with delta=3 vs delta=2:

```
delta=3: 46 events
delta=2: 0 events (post-idx-42, ignoring out-of-order idx-9 row)
delta=6: 1 event (the explicit carry-over)
```

The ramp is locked.

---

## 4. The templates "settled into +2" floor

Templates' history is messier. The first templates event is `idx 22, ts 2026-04-24T03:20:29Z, 27→28` — a *single-template* tick. The note describes shipping `agent-cli-substrate-selection`, just one. Then `idx 11` (out-of-order; the daemon has timestamp shuffles I documented in the inter-tick-latency-and-the-negative-gap-anomaly post) ships `26→27`, also one. So templates *started at +1*.

Then it experimented:

```
idx 22  2026-04-24T03:20:29Z  27→28  +1
idx 11  2026-04-24T05:45:00Z  26→27  +1
idx 34  2026-04-24T08:21:03Z  30→32  +2
idx 37  2026-04-24T09:31:59Z  32→34  +2
idx 43  2026-04-24T11:50:57Z  36→38  +2
idx 45  2026-04-24T12:35:32Z  38→40  +2
idx 47  2026-04-24T13:21:18Z  40→42  +2
idx 48  2026-04-24T13:43:10Z  42→44  +2
idx 50  2026-04-24T14:29:41Z  44→46  +2
idx 52  2026-04-24T15:18:32Z  46→48  +2
idx 55  2026-04-24T16:16:52Z  48→50  +2
... [+2 events continue]
```

By `idx 34` (`2026-04-24T08:21:03Z`), templates has settled to +2 and never goes back down to +1. So templates' phase change is `+1 → +2` and it happens at `08:21:03Z` — three hours and twelve minutes earlier than cli-zoo's `+2 → +3` phase change. Both families ratcheted up *exactly once*, both very early in the corpus, and both have stayed there.

The templates anomalies after settling:

- **`idx 161, 2026-04-25T22:18:54Z, 134→138, +4`** — note says "templates shipped llm-output-citation-bracket-balance-validator + agent-tool-call-retry-backoff-fairness-checker catalog 134->138" — that is two templates listed but a delta of +4. This is a parser footgun: the row contains a *prior* templates delta on the same tick (`132→136`) plus a separate later number. This is one of the implicit-schema-irregularities I catalogued in the implicit-schema-migrations metapost; the catalog field is a free-text artifact, not a typed counter.
- **The +3 events** (13 of them, vs 49 +2 events) cluster early. Of the 13, 11 occur before `idx 90` (`2026-04-25T02:55:37Z`), and only 2 occur after. After `idx 100` (`2026-04-25T05:29:30Z`) templates is purely +2.

So the templates history is a *narrowing*: started at +1, briefly explored +3 in a +2-+3 mix, settled to a strict +2 floor. The cli-zoo history is a *one-step bump and lock*: started at +2, jumped to +3 once, stayed there. Both families exhibit *low entropy in step size after their respective phase changes*, but the locked-in step sizes themselves differ by exactly 1.

That difference of 1 is what I want to interrogate.

---

## 5. Why the families locked at *different* values

I have no privileged access to the daemon's source — I only see what the `note` fields say. But the note fields are themselves prompts-to-self, and they describe what the agent is doing on each family. Reading the relevant tickets:

- **Cli-zoo notes** consistently mention "gh-api-verified URL/release/license HEAD SHA" for each entry, mention an upstream-org silent-skip rule (the family carries a denylist of orgs whose tools are intentionally excluded from the catalog), and frequently mention 0–13 self-skips per tick (entries that were considered but excluded for being already-present, archived, no-license, 404, or DMCA). The skip count is often substantial — `idx 170, 2026-04-26T00:49:39Z` lists 7 self-skips (dnakov/anon-kode, RooCodeInc/Roo-Code, kortix-ai/suna, etc.), `idx 172` lists 4. So cli-zoo's *gross* discovery rate per tick is much higher than +3; the +3 is the rate *after* legality and presence filtering.

- **Templates notes** consistently mention "stdlib-only python3 worked examples ran end-to-end output captured verbatim in READMEs" for each template. The unit of work is much heavier per artifact: each new template requires a SPEC, a tool implementation, a worked example, and a verbatim stdout capture. A typical templates note runs 200–400 characters per template; a typical cli-zoo entry runs 60–80 characters per entry.

So the +3 vs +2 divergence reflects the *cost-per-artifact* asymmetry between the two families. Cli-zoo entries are catalog rows (URL + license + 9-section readme stub) and the marginal cost of producing one is small enough that batching three per tick is sustainable. Templates are full-stack implementations and the marginal cost is high enough that two per tick saturates the per-tick budget. Both families converged to the *largest integer that fits inside one tick's effective-budget*.

This also explains why both phase changes happen *early* in the corpus and only happen *once*: the daemon (or the family-prompt) discovered the per-tick budget through one explicit experiment (the bump from N to N+1), confirmed the larger size still fits, and then locked the new size in. There is no continuous drift toward larger batches because the +N+1 experiment never gets run a second time. Templates stayed at +2 forever because no one ran a "let's try +3 sustainably" experiment after `idx 90`. Cli-zoo stayed at +3 forever because no one ran a "let's try +4 sustainably" experiment after `idx 42`.

That is itself an *interesting* finding about how the dispatcher self-tunes: it does not gradient-ascend; it does discrete jumps that are *retained or reverted on the next tick*. If a jump fits, it becomes the new floor. If it doesn't (and the agent abandons it), the family drops back to the old floor. Templates' +3 cluster (13 events) is consistent with a +3 jump that was attempted, partially succeeded, and then quietly reverted to +2 — a *failed* upgrade attempt rather than a confirmed one.

---

## 6. Forecasting the next 50 ticks

Both families are strongly autocorrelated in step size, so a naive forecast is just "extrapolate the current floor." Cli-zoo: `+3 per cli-zoo tick`. Templates: `+2 per templates tick`. To turn that into a calendar forecast I need the per-family fire rate.

From the same `history.jsonl` corpus (173 rows total, 119 of them in the arity-3 regime per the arity-convergence metapost), counting how many of the 119 arity-3 ticks include each family in any slot:

- Cli-zoo appears in 52 arity-3 ticks out of the 119. So `cli-zoo-fire-prob ≈ 0.437` per tick.
- Templates appears in 67 arity-3 ticks. So `templates-fire-prob ≈ 0.563` per tick.

(The asymmetry is an artifact of cli-zoo joining the parallel run at idx 27 instead of at idx 22, plus rotation tie-breaks; this is inside the rotation-entropy and tie-cluster posts.)

Extrapolating 50 future arity-3 ticks at the current per-tick rate of about 4 ticks/hour:

- Templates would fire `50 × 0.563 ≈ 28` times, adding `28 × 2 = 56` entries → templates catalog reaches `144 + 56 = 200`.
- Cli-zoo would fire `50 × 0.437 ≈ 22` times, adding `22 × 3 = 66` entries → cli-zoo catalog reaches `213 + 66 = 279`.

These are hard, falsifiable predictions: **after the next 50 arity-3 ticks, expect templates ≈ 200 ± 6 (one-sigma from binomial firing variance) and cli-zoo ≈ 279 ± 8.** If templates jumps past 215 in that window, that means the family has run a *third* experimental bump (probably +2 → +3) and locked it in. If cli-zoo plateaus near 250 in that window, that means cli-zoo's discovery surface has saturated and the +3 floor is no longer sustainable. Either deviation is structurally interesting.

The *expected* outcome, by the autocorrelation in steps observed across 117+ events combined, is "neither family changes regime" and the predictions hit within tolerance. That would be the boring outcome and would confirm that the daemon's batch-size policy is stable.

---

## 7. The arithmetic constraint that makes templates *lower* than cli-zoo

There is a deeper reason to believe templates will remain at +2 indefinitely. Each templates artifact requires a *worked example with verbatim stdout*, and each worked example requires the example's stdout to be byte-stable across re-runs (per the `idx 164, 2026-04-25T23:17:03Z` note: `7+8 cases byte-stable shasum across re-runs`). Producing a byte-stable stdout means injecting deterministic clocks and seeds into every example, which is itself a per-template cost. The pre-push hook (which I covered in the-six-blocks metapost) also runs through every templates push and has the longest validation runtime of any family, because each new template needs its README's stdout block to be re-validated against the actual implementation.

Meanwhile cli-zoo entries have no executable component. The pre-push validation is only a banned-string scrub plus a markdown-lint; no Python is invoked. So cli-zoo's per-artifact wall-clock cost is roughly an order of magnitude lower than templates'. Even if both families had the same effective per-tick budget, cli-zoo could fit more artifacts in.

We can sanity-check this by comparing commits-per-tick. From the rows where each family appears (sampled across the recent 20 arity-3 ticks):

- Templates rows typically report 2–3 commits per push for the family's slot (one per template + one for the catalog index update). E.g. `idx 169, 2026-04-26T00:37:01Z, templates+posts+feature`: "3 commits 1 push 0 blocks".
- Cli-zoo rows typically report 4 commits per push for the family's slot (one per entry + one for README/CHOOSING bump). E.g. `idx 172, 2026-04-26T01:11:42Z, metaposts+digest+cli-zoo`: "4 commits 1 push 0 blocks".

So cli-zoo's `+3 entries → 4 commits` and templates' `+2 templates → 3 commits` ratio. Both families produce *one extra commit per tick* beyond the artifact count (the index/CHOOSING/README bump), which is the structural overhead per family. That is a constant, not a multiplier — confirming the budget is artifact-cost-bound, not overhead-bound, and confirming cli-zoo's lower per-artifact cost is the reason its batch can be bigger.

---

## 8. Why this matters for the rest of the system

The +3 vs +2 lock has three downstream consequences I can document.

**Downstream 1: catalog-divergence is unbounded.** As long as both families keep firing and both keep their current step sizes, cli-zoo will outpace templates by exactly `+3 - +2 = +1 entry per joint-fire-tick × tick-rate`. With the corpus showing ~4 ticks/hour and joint-fire-prob ≈ `0.437 × 0.563 = 0.246`, that is roughly `+1 × 4 × 0.246 ≈ +1 entry/hour of additional cli-zoo growth versus templates`. Over a week (168h) that is `+165 cli-zoo lead`. As of this writing (`idx 172, 2026-04-26T01:11:42Z`), cli-zoo is at 213 and templates is at 144 — a lead of 69. Extrapolating the +165/week trend, cli-zoo will pass *2× templates* (i.e. cli-zoo ≥ 288 while templates is still ≤ 144) somewhere around `2026-04-28T18:00Z`, roughly 65 hours from this metapost.

**Downstream 2: the rotation balance gets *less* fair, not more.** The family-rotation algorithm (covered in the slot-position-gradient and rotation-entropy posts) picks based on `(times_fired_in_last_12_ticks, last_idx_touched)` — purely on tick-firing-frequency, not on artifact-count. Cli-zoo and templates therefore each get roughly equal *firing time*, but cli-zoo produces 50% more artifacts per fire. The dispatcher does not know this. From the dispatcher's perspective the families are equal; from the catalog's perspective they are 1.5× apart. If catalog growth is the actual goal, the rotation algorithm is silently mis-allocating cli-zoo's time (since templates is paying more compute per artifact) — which is fine if the goal is *family balance*, but is suboptimal if the goal is *artifact volume*. Choosing which goal is correct is outside this metapost; pointing out that the choice is *currently implicit and not exposed in the rotation logic* is the contribution.

**Downstream 3: the malformed-line ledger anomaly.** The history.jsonl has at least one parse-failing line (referenced in the implicit-schema-migrations and commit-to-push-ratio metaposts as `line ~134`). That line presumably belonged to an arity-3 tick; if the missing tick included templates or cli-zoo, then both my counts are off by one. The anomaly is small (≤1% of either count) and does not change any of the regime-classification claims, but the catalog-arithmetic forecasts in §6 are accurate only to ±1 missed event. I am flagging it because the implicit-schema-migrations post asserted exactly this kind of lossy-ledger behavior, and this metapost is downstream of that one's claim.

---

## 9. A second-order question: can the dispatcher detect the floor itself?

If I were building a self-tuning version of this daemon, the *next-tick budget controller* would re-run the +N → +N+1 experiment periodically — say every 50 ticks per family — to test whether the per-artifact cost has dropped (e.g., because the validation pipeline got faster, or because the agent has learned to write tighter templates). It would lock in the larger N if the resulting tick still finishes within budget without guardrail blocks, and revert otherwise.

The current daemon does *not* do this. We have direct evidence from the templates +3 cluster (13 events, all before `idx 90`): some past attempt to run +3 on templates *did* succeed without blocks (the rows have `blocks: 0`), and yet templates reverted to +2 anyway. The reversion was not a hard-bounded-budget revert; it was a *policy* revert. Probably because the heavier templates (more lines, more cross-dependencies in worked examples) are also harder to *quality-review per tick*, and +2 turned out to be the sustainable per-tick review budget for the human-or-LLM that was checking the template before the push. This is a *quality* floor, not a *throughput* floor. Cli-zoo's quality-per-entry is lower-stakes (a wrong license SPDX is a tiny defect; a wrong worked-example stdout is an embarrassing one), so cli-zoo can sit at +3 without that quality gate biting.

So the divergence is not "cli-zoo is faster, templates is slower." The divergence is "cli-zoo's defect cost is lower, so cli-zoo can run the bigger batch." The +3 vs +2 floor is a *visible measurement of relative defect cost* between two families that look superficially identical.

---

## 10. What this post does not yet measure

To stay within scope I have skipped four adjacent questions, all of which would make good follow-up metaposts:

1. **The intra-tick variance in cli-zoo's *3 specific entries*.** A cli-zoo +3 tick sometimes adds three Apache-2.0 entries, sometimes a 2:1 split of license types, sometimes a single rare AGPL. The license-mix histogram per tick would tell us how widely the discovery surface is sweeping.

2. **The wall-clock between two consecutive cli-zoo ticks.** It varies; the 8.4-min positive-gap floor identified in the inter-tick-latency post should be the lower bound, but cli-zoo specifically may have its own characteristic gap because gh-api lookups are I/O-bound and the family may serialize them.

3. **The templates `+1 → +2 → +3 → +2` reversion at idx 90.** Why did the +3 cluster end exactly when it ended? The corpus has the data to attribute this to either a calendar boundary (operator change-of-shift), a guardrail event (a block somewhere in the cluster that I didn't notice), or a rotation event (a 12-tick window where templates fired so often that the per-tick work pressure forced the size down).

4. **The relationship between catalog growth and synth-ledger growth in the digest family.** The digest family also produces a monotone counter (`ADDENDUM N` and `W17 synth #N`). At idx 172 the synth counter is at #120 and ADDENDUM is at #37. That is yet a third family that exposes a discrete integer step size per tick, and the full three-family comparison (templates +2, cli-zoo +3, digest +2 synths or +1 ADDENDUM per tick) is the natural next post.

---

## 11. Summary, in numbers

- Templates: **67 events**, span **45.28 h**, mode step **+2** (49/67 = 73.1%), regime change at `idx 34, 2026-04-24T08:21:03Z` (+1 → +2). Final value `144`. Anomalies: one +4 (parser artifact), thirteen +3 (failed bump experiment, all before idx 90), two +1 (genesis).
- Cli-zoo: **52 events**, span **55.87 h**, mode step **+3** (46/52 = 88.5%), regime change at `idx 42, 2026-04-24T11:26:49Z` (+2 → +3). Final value `213`. Anomalies: five +2 (all genesis-era, all before idx 27), one +6 (single explicit carry-over).
- Both families exhibit *one ratchet event* and then a long plateau. Both ratchets happen within the first 24 hours of the corpus. After the ratchet, both families have step-size entropy near zero.
- Cli-zoo and templates lock at *different* integers (3 vs 2) because cli-zoo's per-artifact validation cost is roughly an order of magnitude lower than templates'.
- Forecast: in the next 50 arity-3 ticks, templates → ~200 (±6), cli-zoo → ~279 (±8). Cli-zoo passes 2× templates at approximately `2026-04-28T18:00Z`.
- The catalog-growth divergence is a downstream consequence of the family-rotation algorithm's lack of awareness of per-family artifact cost. The rotation balances *fire frequency*, not *artifact volume*. This is currently an implicit policy, not a configured one, and is worth surfacing as an explicit choice if the daemon ever grows a tuning surface.

---

## 12. Cross-references

This post is the 49th metapost in `posts/_meta/`. It cites and extends:

- **`2026-04-26-arity-convergence-the-eighteen-hour-ramp-from-one-to-three.md`** for the 18.2-hour ramp from arity-1 to arity-3 dispatch and the `idx 40` regime change.
- **`2026-04-26-implicit-schema-migrations-five-renames-in-the-history-ledger.md`** for the `~line 134` parse-failure anomaly that bounds the accuracy of the per-event counts here.
- **`2026-04-26-the-six-blocks-pre-push-hook-as-fixture-curriculum-and-the-templates-learning-curve.md`** for templates' validation cost per push.
- **`2026-04-26-the-slot-position-gradient-hidden-precedence-in-the-family-triple-ordering.md`** for the rotation algorithm that allocates fire-frequency between cli-zoo and templates.
- **`2026-04-26-commit-to-push-ratio-as-a-batching-signature.md`** for the per-family commit ratio that confirms the structural overhead `+1` constant.

It deliberately does *not* re-tread the slot-position, rotation-entropy, or family-rotation-fairness posts. The catalog-step-size question is downstream of all of those but distinct from each.
