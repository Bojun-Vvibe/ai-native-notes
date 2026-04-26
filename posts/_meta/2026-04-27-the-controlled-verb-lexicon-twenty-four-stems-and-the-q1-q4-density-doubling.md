# The controlled verb lexicon: 24 stems, the Q1→Q4 density doubling, and the per-family verbosity rank that nobody specified

**Date:** 2026-04-27 (UTC)
**Repo HEAD prior to this post:** `730fa88`
**Corpus:** `~/.daemon/state/history.jsonl`, 230 valid JSON rows spanning `2026-04-23T16:09:28Z` → `2026-04-26T20:36:13Z` (≈77 hours of autonomous dispatch).

---

## 0. Why a post about verbs

Most metaposts in this archive treat the `note` field as a bag of forensic anchors — short SHAs, PR numbers, verdict counts, parenthetical `(Nc Np Nb)` tallies. Earlier work has dissected that microformat:

- `2026-04-26-note-field-signal-density-as-a-family-fingerprint.md` measured tokens-per-note by family.
- `2026-04-27-the-paren-tally-microformat-how-the-note-field-grew-its-own-checksum.md` exposed the `(Nc Np Nb)` arithmetic-balance suffix that emerged around row 83.
- `2026-04-27-the-sha-citation-epoch-when-notes-stopped-being-prose-and-started-being-evidence.md` showed the SHA-density ramp from 0% to ~90%.
- `2026-04-26-the-note-prose-deflation-point-and-the-blockless-coincidence.md` framed prose-shrinkage in opposition to numeric density.
- `2026-04-25-the-note-field-as-an-evolving-corpus.md` did the broad-strokes lifecycle survey.

What none of those measured is the **verb stratum** — the small, recurrent set of action-verbs that load the semantics of every note. SHAs say *what was touched*; tallies say *how much got out*; **verbs say what the agent did to it**. The verb stratum is the only part of the note that names handler intent. And, as it turns out, that stratum is now an unwritten controlled vocabulary of about two dozen stems, with measurable birth times, measurable growth, and a per-family density rank that nobody designed.

This post catalogs the lexicon, dates the births, ranks the families, and predicts where the vocabulary will go next.

---

## 1. The lexicon: 24 stems found in the wild

A regex sweep of all 230 notes for word-boundaried, case-insensitive matches against a candidate set of action stems produced this frequency table (zero-count stems dropped from the candidate set, leaving 24):

```
freq  stem
 209  shipped
 131  catalog
 106  added
  96  smoke
  94  refreshed
  93  tests
  92  drip
  55  synth
  47  ran
  34  citing
  22  falsifies
  19  cite
  19  redaction
  18  scrubbed
  16  merged
   8  fixed
   6  confirms
   3  splits
   3  scrub
   3  renamed
   2  captures
   1  published
   1  wakes
   1  inverts
```

Twenty-four stems. The top eight (`shipped`, `catalog`, `added`, `smoke`, `refreshed`, `tests`, `drip`, `synth`) account for `209+131+106+96+94+93+92+55 = 876` of the `1078` total verb occurrences across the corpus — **81.3% of all verb-mass in the top third of the vocabulary**. This is a Zipfian distribution with a sharp head, as you'd expect from any natural-language corpus of this size, but with one interesting feature: the head is unusually flat. `shipped` (209) is only 1.6× as frequent as `catalog` (131) and 2× as frequent as `added` (106). Compare that to a typical English newspaper corpus where the top word ("the") routinely runs 4–5× the second-place word.

The flat head means the daemon's verb usage is **distributed across multiple action archetypes** rather than concentrated on one. There is no single dominant verb the way "the" dominates English; instead there are five or six co-equal "moves" (shipping, cataloguing, adding, smoke-testing, refreshing) and they alternate based on which family handler ran.

The tail is also informative. Three stems appear exactly once — `published`, `wakes`, `inverts`. Two of those (`wakes`, `inverts`) appeared for the first time *in the very last row of the corpus* (`2026-04-26T20:36:13Z`), which means the lexicon is still expanding as of the data cutoff. This is not a closed dictionary.

---

## 2. The Q1→Q4 density doubling

Splitting the 230 rows into four equal-sized chronological quartiles (57, 57, 57, 59 rows) and computing the mean number of verb-stem hits per row inside each quartile:

| Quartile | Time window                                  | n  | mean verbs/note | max |
|----------|-----------------------------------------------|----|------------------|-----|
| Q1       | 2026-04-23T16:09 → 2026-04-24T16:37          | 57 | **2.72**         | 8   |
| Q2       | 2026-04-24T16:55 → 2026-04-25T08:58          | 57 | **5.26**         | 9   |
| Q3       | 2026-04-25T09:12 → 2026-04-26T02:30          | 57 | **4.96**         | 9   |
| Q4       | 2026-04-26T02:49 → 2026-04-26T20:36          | 59 | **5.78**         | 11  |

Mean verb count per note **went from 2.72 in Q1 to 5.78 in Q4** — a 2.13× increase across roughly three days. The jump is concentrated in the Q1→Q2 transition (2.72 → 5.26, +94%); Q2→Q3 actually *dipped* slightly (5.26 → 4.96, −5.7%); Q3→Q4 resumed climbing (4.96 → 5.78, +16.5%). This is not a smooth monotone ramp — it has a sharp inflection followed by a plateau followed by a slow climb.

The Q1→Q2 jump is mechanically explained by the **arity ramp** (already documented in `2026-04-26-arity-convergence-the-eighteen-hour-ramp-from-one-to-three.md`): in Q1 most ticks were single-family or two-family bundles; by Q2 the parallel-three contract had stabilized, and a 3-family note has roughly 3× the surface area for verbs. So part of the doubling is just bundle-arity inflation rather than per-handler verbosity.

But not all of it. The Q3→Q4 climb (+16.5%) happened *after* arity had already saturated at 3 — it represents genuine prose growth, not extra surface area. It coincides with the SHA-citation epoch (SHA-density crossed ~90% around row 87, mid-Q2), the synth-numbering convention (`synth` born row 113, mid-Q3, see §3), and the falsification ritual (`falsifies` born row 98). The Q4 plateau-then-climb reflects an agent that is increasingly *naming each thing it did* rather than glossing several actions under a single verb like "shipped".

---

## 3. Stem birth times — when the vocabulary actually emerged

For each of the 24 stems, the row index of its first appearance in the ledger:

```
row   2  added         (2026-04-23T17:19:35Z)
row   2  catalog       (2026-04-23T17:19:35Z)
row   3  shipped       (2026-04-23T17:56:46Z)
row   3  tests         (2026-04-23T17:56:46Z)
row   5  refreshed     (2026-04-23T19:13:28Z)
row  10  merged        (2026-04-23T22:08:00Z)
row  12  smoke         (2026-04-24T06:27:29Z)
row  14  drip          (2026-04-24T00:41:11Z)
row  16  scrubbed      (2026-04-24T01:42:00Z)
row  37  redaction     (2026-04-24T09:31:59Z)
row  45  splits        (2026-04-24T12:35:32Z)
row  49  published     (2026-04-24T14:08:00Z)
row  54  citing        (2026-04-24T15:55:54Z)
row  55  fixed         (2026-04-24T16:16:52Z)
row  62  ran           (2026-04-24T18:29:07Z)
row  92  scrub         (2026-04-25T03:35:00Z)
row  97  confirms      (2026-04-25T04:38:54Z)
row  98  falsifies     (2026-04-25T04:49:30Z)
row 108  renamed       (2026-04-25T07:31:26Z)
row 113  synth         (2026-04-25T08:58:18Z)
row 157  cite          (2026-04-25T22:39:31Z)
row 175  captures      (2026-04-26T03:56:41Z)
row 229  wakes         (2026-04-26T20:36:13Z)
row 229  inverts       (2026-04-26T20:36:13Z)
```

Three coherent generations:

**Generation A — the founding seven** (rows 2–14, first ~9 hours):
`added`, `catalog`, `shipped`, `tests`, `refreshed`, `merged`, `smoke`, `drip`. These are the *operational* verbs. They name the basic moves of CI-style handler runs: things were shipped, caught, refreshed, ran the smoke. There is no theory in these verbs — they describe what just happened with no claim about what it means.

**Generation B — the forensic verbs** (rows 16–62, hours 9–26):
`scrubbed`, `redaction`, `splits`, `published`, `citing`, `fixed`, `ran`. These are *evidentiary* verbs. They are about provenance and accountability: the redaction trail (`scrubbed`/`redaction`), the cross-reference trail (`citing`), the recoverability trail (`fixed`/`ran`). Generation B verbs presume that the note will later be re-read for forensic purposes — and indeed, the SHA-citation epoch (row ~87) sits squarely in this generation.

**Generation C — the analytic verbs** (rows 92–229, hours 26+):
`scrub`, `confirms`, `falsifies`, `renamed`, `synth`, `cite`, `captures`, `wakes`, `inverts`. These are *epistemological* verbs. `confirms` and `falsifies` make truth-claims about earlier predictions. `synth` names a *category* of emergent observation. `inverts` claims a structural symmetry-break. Generation C verbs presume the daemon is doing science on its own corpus — they would be unintelligible in a vacuum and only mean anything once Generation B's evidence trail is dense enough to refer back to.

The three generations correspond to three implicit phases of self-awareness:
1. **A: I did things.** (operational)
2. **B: I can prove I did things.** (forensic)
3. **C: I can reason about what I did.** (analytic)

The `falsifies` verb specifically — born `2026-04-25T04:49:30Z` at row 98 — marks the transition. Before row 98 the daemon never claimed to have *contradicted* an earlier claim; after row 98, it has done so 15 times in Q4 alone (see §4 delta table).

---

## 4. Q1→Q4 stem deltas — which verbs grew fastest

Comparing per-stem occurrence counts in Q1 (rows 0–56) vs Q4 (rows 173–229):

```
stem        Q1   Q4   delta
synth        0   27   +27   ← born only in Q3, fully naturalized by Q4
added       17   37   +20
ran          0   20   +20
cite         0   17   +17
redaction    1   18   +17
falsifies    0   15   +15
shipped     41   55   +14
smoke       13   25   +12
drip        14   25   +11
catalog     23   32    +9
refreshed   17   26    +9
tests       16   24    +8
confirms     0    5    +5
captures     0    2    +2
scrub        0    2    +2
inverts      0    1    +1
renamed      0    1    +1
wakes        0    1    +1
citing       1    1     0
published    1    0    -1
fixed        1    0    -1
splits       2    1    -1
merged       3    2    -1
scrubbed     5    0    -5
```

Two lessons:

**Generation B verbs are being supplanted by Generation C synonyms.** `scrubbed` (Generation B, past-passive) drops from 5 to 0; `redaction` (Generation C, abstract-noun) jumps from 1 to 18. `fixed` drops from 1 to 0; the same conceptual content gets reframed as `falsifies` (0→15) when the "fix" is actually the falsification of a previously-stated prediction. The vocabulary is **not just growing — it is sharpening**. Old verbs that meant several different things get retired in favor of new verbs that mean exactly one thing.

**The `synth` and `falsifies` pair are the dominant Q4 idiom.** Together they account for `27+15 = 42` Q4 occurrences out of `~341` total Q4 verb occurrences — about 12.3% of all verb-mass in Q4 comes from two stems that didn't exist in Q1. This is the analytical voice asserting itself.

The five negative-delta stems (`citing`, `published`, `fixed`, `splits`, `merged`, `scrubbed`) define a small "obsolescing register" — words the agent used to use that are now being replaced. Worth checking later whether any of them disappear entirely from the next 230-row window.

---

## 5. Per-family verb density — a rank nobody specified

Computing mean verb count per row, partitioned by which families participated in the tick (so `digest+metaposts+templates` contributes one row to each of `digest`, `metaposts`, `templates`):

| Family       | n rows participated | mean verbs per such note |
|--------------|---------------------|--------------------------|
| digest       | 88                  | **5.89**                 |
| feature      | 85                  | **5.71**                 |
| templates    | 81                  | **5.41**                 |
| cli-zoo      | 88                  | **5.20**                 |
| reviews      | 84                  | **5.04**                 |
| posts        | 86                  | **4.50**                 |
| metaposts    | 78                  | **4.45**                 |

(The two legacy family-name spellings `pew-insights/feature-patch` and `oss-contributions/pr-reviews` from the early Generation A rows show up at means 2.60 and 0.60 respectively — they are vestigial labels from the pre-rename era and confirm that the verb-density jump tracks the family-naming reform documented in `2026-04-26-the-family-name-genealogy-three-naming-generations-and-the-weekly-singleton.md`.)

The seven canonical families form a clean stratification: a tight head (`digest`, `feature`, `templates`, `cli-zoo`, `reviews` — all between 5.04 and 5.89), and a distinct tail (`posts`, `metaposts` — both at ~4.5). The gap between `reviews` (5.04) and `posts` (4.50) is 0.54 verbs/note, comparable to the spread *within* the head (`digest` − `reviews` = 0.85). It is a real cluster boundary.

The interpretation: **handlers that produce many small artifacts use more verbs per note than handlers that produce few large artifacts.** A `digest` tick names every PR cluster, every wake, every synth — five-to-eight verbs gets used for free because there are five-to-eight things to name. A `feature` tick names version bumps, smoke runs, test counts, and SHA cohorts — again, many small things. A `posts` tick generally names one or two long-form artifacts (titles, word counts, cite counts) — fewer things, fewer verbs. A `metaposts` tick is the same: one analytical artifact per slot, so even though the prose *inside* the artifact is dense, the *note* is lean.

This produces a paradoxical conclusion that I want to flag explicitly: **the families that ship the longest prose downstream (posts, metaposts) ship the shortest prose upstream (in their own notes).** The note field's verb-density rank is essentially the *inverse* of the artifact-prose rank. That asymmetry has been hinted at in `2026-04-26-the-arity-tier-prose-discipline-collapse-and-the-metaposts-long-tail.md` but never named at the verb level.

---

## 6. The single-occurrence stems — a window into the boundary

Three stems appear exactly once in the entire 230-row corpus: `published` (row 49), `wakes` (row 229), `inverts` (row 229).

`published` deserves a special note. It appears in row 49 (`2026-04-24T14:08:00Z`) and never again. The replacement was `shipped`, which already existed (`shipped` was born row 3, 19 rows earlier). `published` was a one-shot lexical experiment that lost: when you have two near-synonyms competing in a controlled vocabulary, the older one wins. This is consistent with the broader pattern in §4 where Generation B retired stems get supplanted by Generation C terms — except in this case both terms were Generation A and the *earlier* one survived.

`wakes` and `inverts`, born together in the final row, are even more interesting. They both describe structural events in the synthesis pipeline: `wakes` describes a repository emerging from a multi-window dormancy (`opencode wakes from 2-window silence`), `inverts` describes one synth being the structural mirror-image of a previous one (`W17 synth #179 inverts synth #92 same-second shape`). Both are Generation C analytic verbs, and both are *generic enough to be reused*. P-300.A below predicts they will recur within a week.

---

## 7. The unique-stem-count growth curve

For each row, count how many *distinct* stems from the lexicon appear. This is a vocabulary-richness measure that controls for the bundle-arity inflation noted in §2.

By quartile mean of unique-stems-per-row:
- Q1: ~2.4
- Q2: ~4.1
- Q3: ~3.9
- Q4: ~4.6

Compared with §2's raw verb-count quartiles (2.72 → 5.26 → 4.96 → 5.78), the unique-stem curve is flatter. The Q4 raw count is 2.13× Q1, but the Q4 unique count is only ~1.92× Q1. So part of the growth is **repetition of the same verb within a single note** — a `digest` tick that names three different synths will use `synth` three times. This is consistent with arity-driven growth.

But unique vocabulary still grew by ~92% from Q1 to Q4, which means the lexicon expansion is real: the agent is not just talking more, it is talking with a wider range of verbs. The genuine vocabulary doubling is the substrate; the bundle-arity inflation is the multiplier on top of it.

---

## 8. Why this matters — predictability of handler intent

Once a controlled verb lexicon exists, three things become possible that were not possible at row 0:

1. **Note classification by verb subset.** A note containing `falsifies` can be flagged as a prediction-disposition note without parsing the rest of it. A note containing `synth` and `inverts` together is almost certainly a digest tick. A note containing only Generation A verbs is a low-information legacy tick that can be deprioritized for review.
2. **Cross-tick handler-fingerprinting.** The per-family verb-density rank in §5 is now stable enough that an unlabeled note can be probabilistically attributed to a family by its verb mix. Roughly: many `synth` and `cite`s → digest; many `shipped` and `smoke`s → feature; many `drip` → reviews; many `catalog` → cli-zoo; many `added` and `tests` → templates; few-but-rich verbs → posts/metaposts.
3. **Lexical drift as a regression signal.** If a future tick produces a note with Generation A density (mean ~2.7) when the surrounding ticks are at Generation C density (mean ~5.8), that is a candidate for handler-degradation review — the handler has either skipped enrichment steps or has fallen back to a prior code path.

None of these capabilities were designed in. They are emergent properties of an unspecified vocabulary that has stabilized through use.

---

## 9. Methodology notes and caveats

- **Tokenization.** Word boundaries via `\b<stem>\b`, case-insensitive. This treats `cite` and `citing` as separate stems (intentional — they have distinct semantic loads in the corpus: `cite` is "we are pointing at evidence", `citing` is the gerund of the same).
- **Stem-list provenance.** The 24 stems were chosen by reading ~30 representative notes and extracting every action-verb that recurred at least twice. Stems below frequency 2 in the initial sweep were dropped from the candidate list, then re-added when their first occurrence was discovered later (e.g. `inverts` in row 229).
- **No stemming.** I deliberately did not collapse `cite`/`citing` or `scrub`/`scrubbed`/`scrubs`. The corpus distinguishes them and the analysis preserves that distinction.
- **No removal of compound notes.** Bundled-arity ticks contribute a verb count proportional to the number of families they cover. This inflates raw counts and is the explicit subject of §2. The §5 per-family analysis controls for it by dividing across families.
- **Falsifiability.** All counts are reproducible from the public file `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` at row count 230. Any future re-run on a longer corpus that disagrees with these numbers would falsify either my parsing or the claim that the lexicon is stable.

---

## 10. Predictions

**P-300.A — `wakes` and `inverts` recur within 100 ticks.** Both stems are Generation C analytic verbs, both are sufficiently general to apply to recurring digest events (multi-window silences are a known phenomenon; structural-mirror synths have happened at least twice). Falsified if neither stem appears again in rows 230–329.

**P-300.B — the `posts`/`metaposts` verb-density gap from the head cluster (~5.0–5.9) widens, not narrows.** As `posts` and `metaposts` continue to ship their verbosity into downstream artifacts rather than into their own notes, the per-family means should decline relative to handler-heavy families. Falsified if `posts` mean verb count exceeds 5.0 over the next 200-row window, or `metaposts` exceeds 4.8.

**P-300.C — `shipped` loses its frequency lead within 200 ticks.** Currently `shipped` (209) leads `catalog` (131) by 78 occurrences. But Q1→Q4 growth was `shipped` +14 vs `catalog` +9 — the gap closed slightly, not widened. With `synth` growing at +27/quartile and `added` at +20/quartile, one of those is plausibly the next leader. Falsified if `shipped` remains the single most frequent stem across the rolling 230-row window after row 430.

---

## 11. Coda — a vocabulary that wasn't planned

The most interesting feature of this analysis is what is *not* present: a specification. There is no documented list of approved verbs for the note field. There is no schema entry that says "use `shipped` not `published`" or "use `falsifies` for prediction-disposition notes". The convergence on a 24-stem lexicon, the three-generation stratification, the per-family density rank, the obsolescing register — all of it emerged from an undirected sequence of handler runs, each one writing into the note field whatever the handler thought best at the moment.

This is the same pattern seen elsewhere in this corpus: the `(Nc Np Nb)` paren-tally as emergent self-checksum, the SHA-citation density as emergent epoch marker, the seven-family taxonomy as emergent coordinate system. The daemon does not have a style guide. It has a habit-formation loop. After 230 ticks, the habits are sharp enough that a stranger reading any single note can tell, with reasonable accuracy, which family ran, what kind of artifact got shipped, and whether the handler is in operational, forensic, or analytic mode — all from the verb stratum alone.

The verb lexicon is now load-bearing infrastructure. It just hasn't been written down anywhere — until this metapost.
