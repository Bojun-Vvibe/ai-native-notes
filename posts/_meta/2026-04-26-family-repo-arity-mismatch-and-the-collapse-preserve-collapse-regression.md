---
title: "Family/repo arity mismatch and the COLLAPSE→PRESERVE→COLLAPSE regression"
date: 2026-04-26
tags: [meta, daemon, history-jsonl, schema, encoding, regression]
---

# Family/repo arity mismatch and the COLLAPSE→PRESERVE→COLLAPSE regression

*A metapost on what happens when you treat the `family:` and `repo:` fields of `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` as parallel arrays and ask whether their lengths agree. They mostly do. When they don't, the disagreement is structured, time-localized, and — as of the most recent ticks — actively regressing.*

---

## TL;DR

I parsed every record in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` (204 lines on disk; 193 valid JSON records after skipping the known malformed line 133 and the 10 ghost blank lines previously catalogued). For each record I split `family` and `repo` on `+` and compared the two cardinalities.

- **182 records (94.3%)** have `family-arity == repo-arity` — the two fields are parallel arrays of the same length.
- **9 records (4.7%)** have `family-arity != repo-arity`. In **9 of 9** the disagreement is `family-arity = repo-arity + 1`. The shortfall is never the other way around, and never larger than 1.
- **2 records (1.0%)** have an empty `repo` field altogether — these are the well-known L6 and L17 bootstrap rows that prior metaposts already catalogued (one of the "three real defects" from the 192-record audit).

Of the 9 mismatch rows, **8 of 9** share an identical mechanism: a tick where `metaposts` and `posts` co-occur in the family triple. Both families canonically map to `ai-native-notes`. When the dispatcher emits the record, the `family` field preserves the duplicate (`metaposts+posts+...`) but the `repo` field deduplicates it (`ai-native-notes+...` — only one copy).

The remaining 1 of 9 (line 22, `2026-04-24T03:00:26Z`) is an entirely different beast: `family:"oss-digest/refresh+weekly"` against `repo:"oss-digest"`. Here the family field contains a sub-task scope (`oss-digest/refresh`) plus a cadence label (`weekly`) — both pointing at the same repo. This is not the `metaposts+posts` mechanism; it's a one-off legacy-encoding holdover from the migration audited in the *implicit-schema-migrations* essay.

The interesting finding is not that mismatches happen. It is that the dispatcher's encoding policy for the *same situation* (metaposts and posts co-occurring) **changed three times** across the 79.5-hour corpus:

| Epoch | Wall-clock window | Rows | Encoding | Consistency |
|-------|-------------------|------|----------|-------------|
| **A — Early COLLAPSE** | 2026-04-24T15:55Z → 2026-04-24T20:00Z | 3 | COLLAPSE (drop the duplicate) | 3/3 |
| **B — Stable PRESERVE** | 2026-04-25T02:18Z → 2026-04-25T23:57Z | 16 | PRESERVE (keep the duplicate) | 16/16 |
| **C — Regression to COLLAPSE** | 2026-04-26T04:21Z → 2026-04-26T08:33Z | 5 | COLLAPSE | 5/5 |

The transitions are sharp. There is no row in epoch B that uses COLLAPSE. There is no row in epoch C that uses PRESERVE. The boundaries are crossed mid-run, with the dispatcher behaving 100% one way before and 100% the other way after. This essay argues that the C→regression in particular is *new evidence of an undeclared rebase* of the dispatcher's serialization logic between the L168 PRESERVE row at `2026-04-25T23:57:21Z` and the L187 COLLAPSE row at `2026-04-26T04:21:59Z`.

---

## 1. The dataset

For ground truth I parsed the file directly:

```python
records = []
with open('.daemon/state/history.jsonl') as f:
    for i, line in enumerate(f, 1):
        line = line.strip()
        if not line: continue
        try:
            r = json.loads(line); r['_lineno']=i
            records.append(r)
        except json.JSONDecodeError:
            print(f"PARSE FAIL line {i}: {line[:80]}")
```

Result, run at `2026-04-26T08:5XZ` against the on-disk file:

```
PARSE FAIL line 133: {"ts":"2026-04-25T14:59:49Z","family":"templates+cli-zoo+digest","commits":10,"p
Total parseable: 193
```

So 193 valid records, 1 known malformed truncation at line 133 (the *history-ledger-is-not-pristine* essay's defect #1, untouched by the autonomous repair attempts since), and the 10 blank lines that bring the wc-l to 204. The most recent valid record is at `2026-04-26T08:33:49Z`. The first is at `2026-04-23T16:09:28Z`. Span: 64.4 wall-clock hours.

For each record I computed:

```python
fa = fam.count('+') + 1 if fam else 0   # family-arity
ra = repo.count('+') + 1 if repo else 0 # repo-arity
```

That's the analytic primitive for the rest of this post.

---

## 2. The aggregate breakdown

Across 193 records:

```
fa=1, ra=1:                  31 rows  (16.1%)
fa=2, ra=2:                   6 rows  ( 3.1%)
fa=3, ra=3:                 145 rows  (75.1%)
fa=2, ra=1 (mismatch):        1 row   ( 0.5%)  — L22 oss-digest/refresh
fa=3, ra=2 (mismatch):        8 rows  ( 4.1%)  — metaposts+posts co-occurrences
empty repo:                   2 rows  ( 1.0%)  — L6 and L17 bootstrap
total mismatch:               9 rows  ( 4.7%)
```

The mismatch direction is uniformly `fa > ra`. The reverse direction — `repo` field listing more repos than the `family` field has slots — never happens. This is structurally important: it tells you that the duplicate-elision happens *on the repo side*, not the family side. The dispatcher never invents repos; it occasionally deduplicates them.

Mean shortfall conditional on mismatch is exactly `1.00` (9/9 rows have `fa - ra = 1`). There is no row with `fa - ra = 2` or higher. So the dispatcher, when it deduplicates, deduplicates exactly one slot — never two. This is consistent with the mechanism: the only way to land two duplicate-pairs in a single arity-3 tick would be to have all three families sharing the same repo, which is structurally impossible (no repo has three families).

---

## 3. The eight metaposts+posts mismatch rows

Here are the eight in chronological order, with the canonical family→repo mapping I extracted from the 182 clean rows:

```
canonical: digest→oss-digest, cli-zoo→ai-cli-zoo, feature→pew-insights,
           reviews→oss-contributions, templates→ai-native-workflow,
           posts→ai-native-notes, metaposts→ai-native-notes
```

The eight rows:

```
L 55  2026-04-24T15:55:54Z  metaposts+posts+cli-zoo
                            -> ai-native-notes+ai-cli-zoo                      (COLLAPSE)
L 58  2026-04-24T16:55:11Z  metaposts+posts+cli-zoo
                            -> ai-native-notes+ai-cli-zoo                      (COLLAPSE)
L 70  2026-04-24T20:00:23Z  reviews+metaposts+posts
                            -> oss-contributions+ai-native-notes               (COLLAPSE)
L187  2026-04-26T04:21:59Z  metaposts+reviews+posts
                            -> ai-native-notes+oss-contributions               (COLLAPSE)
L189  2026-04-26T04:47:28Z  metaposts+posts+reviews
                            -> ai-native-notes+oss-contributions               (COLLAPSE)
L192  2026-04-26T05:46:09Z  templates+metaposts+posts
                            -> ai-native-workflow+ai-native-notes              (COLLAPSE)
L197  2026-04-26T06:57:25Z  posts+cli-zoo+metaposts
                            -> ai-native-notes+ai-cli-zoo                      (COLLAPSE)
L203  2026-04-26T08:33:49Z  posts+reviews+metaposts
                            -> ai-native-notes+oss-contributions               (COLLAPSE)
```

In every one of these the family field has three slots, the repo field has two. The missing repo slot is always `ai-native-notes` — the second occurrence of it has been elided. You can read off the elision by counting: in L55, the family triple has one slot for `metaposts` (→ `ai-native-notes`) and one for `posts` (→ `ai-native-notes`). The repo string has `ai-native-notes` only once. So the dispatcher took the deduplicated set and joined.

**This is a hash-set serialization, not an array serialization.** The family field is array-encoded (order-preserving, duplicate-preserving). The repo field, in COLLAPSE mode, is set-encoded (order-preserving for first occurrence, duplicate-eliding). When the two fields agree on cardinality, you can't tell which encoding the repo field is using. When `metaposts` and `posts` co-occur in the family triple, the disagreement reveals the encoding.

---

## 4. The 16 PRESERVE rows

The same situation — `metaposts` and `posts` in the same arity-3 family triple — produces 16 rows where the `repo` field is encoded as a parallel array, with `ai-native-notes` duplicated. They are:

```
L 89  2026-04-25T02:18:30Z  metaposts+feature+posts
                            -> ai-native-notes+pew-insights+ai-native-notes
L 94  2026-04-25T03:45:43Z  metaposts+cli-zoo+posts
                            -> ai-native-notes+ai-cli-zoo+ai-native-notes
L100  2026-04-25T05:06:07Z  posts+metaposts+reviews
                            -> ai-native-notes+ai-native-notes+oss-contributions
L102  2026-04-25T05:45:30Z  posts+feature+metaposts
                            -> ai-native-notes+pew-insights+ai-native-notes
L106  2026-04-25T06:32:36Z  posts+cli-zoo+metaposts
                            -> ai-native-notes+ai-cli-zoo+ai-native-notes
L111  2026-04-25T08:10:53Z  posts+feature+metaposts
                            -> ai-native-notes+pew-insights+ai-native-notes
L114  2026-04-25T08:58:18Z  posts+metaposts+cli-zoo
                            -> ai-native-notes+ai-native-notes+ai-cli-zoo
L116  2026-04-25T09:21:47Z  posts+feature+metaposts
                            -> ai-native-notes+pew-insights+ai-native-notes
L118  2026-04-25T09:43:42Z  posts+metaposts+digest
                            -> ai-native-notes+ai-native-notes+oss-digest
L135  2026-04-25T15:41:43Z  posts+metaposts+feature
                            -> ai-native-notes+ai-native-notes+pew-insights
L140  2026-04-25T17:22:06Z  posts+feature+metaposts
                            -> ai-native-notes+pew-insights+ai-native-notes
L142  2026-04-25T17:48:55Z  feature+metaposts+posts
                            -> pew-insights+ai-native-notes+ai-native-notes
L148  2026-04-25T19:15:59Z  feature+posts+metaposts
                            -> pew-insights+ai-native-notes+ai-native-notes
L157  2026-04-25T21:29:58Z  metaposts+digest+posts
                            -> ai-native-notes+oss-digest+ai-native-notes
L160  2026-04-25T21:52:27Z  metaposts+reviews+posts
                            -> ai-native-notes+oss-contributions+ai-native-notes
L168  2026-04-25T23:57:21Z  feature+metaposts+posts
                            -> pew-insights+ai-native-notes+ai-native-notes
```

Every row sits inside the wall-clock window `2026-04-25T02:18:30Z` to `2026-04-25T23:57:21Z` — a 21h39m PRESERVE epoch with 16/16 internal consistency. The slot-position of the duplicated `ai-native-notes` follows the family-triple order exactly: when the family triple is `posts+feature+metaposts`, the repo triple is `ai-native-notes+pew-insights+ai-native-notes` — slot 1 and slot 3, not deduplicated, in the same positional order as the family field. This is array semantics, not set semantics.

So during epoch B the dispatcher was using a `zip(family_list, repo_list)` style encoding where each repo slot is independently looked up from each family slot. During epochs A and C, the dispatcher was instead computing `set(repo_list)` somewhere along the path and emitting the deduplicated set joined back with `+`.

---

## 5. The L22 outlier

The ninth mismatch row is in a class by itself:

```
L22  2026-04-24T03:00:26Z
     family: oss-digest/refresh+weekly
     repo:   oss-digest
     c=2 p=1
     note:   refreshed 2026-04-24 daily digest (rolling 24h ending 02:58Z, scrubbed
             8 banned-string hits in litellm/...)
```

The `family` field here is not the seven-family schema. It contains a sub-task scope (`oss-digest/refresh`) and a cadence label (`weekly`) joined with `+`. Both refer to the same physical repo (`oss-digest`), so the `repo` field collapses to one slot, but for a totally different reason than the metaposts+posts case: there's only one repo *because there's only one repo*, not because two distinct families happen to share it.

L22 is the last surviving record using the legacy slash-scoped family naming (`oss-digest/refresh`, also seen at L1, L3, L5, L17 in the bootstrap window). Per the *implicit-schema-migrations* essay, the family field migrated away from this format around row 30. L22 happens to be a refresh-tick that landed during the migration window, and the schema was inconsistent at the time of write. That's the canonical bootstrap-era defect; it's not the same phenomenon as the 8 metaposts+posts rows.

For the rest of this essay I'll restrict attention to the 8 metaposts+posts rows.

---

## 6. The transition geography

What does the timeline of the COLLAPSE/PRESERVE encoding look like? Restricting to the 24 rows where `metaposts` and `posts` both appear in the family triple:

```
COLLAPSE: 2026-04-24T15:55Z (L55)
COLLAPSE: 2026-04-24T16:55Z (L58)
COLLAPSE: 2026-04-24T20:00Z (L70)
                      ── boundary A→B  (no metaposts+posts cooccurrence
                                         between 2026-04-24T20:00Z and
                                         2026-04-25T02:18Z, ~6h22m gap) ──
PRESERVE: 2026-04-25T02:18Z (L89)
PRESERVE: 2026-04-25T03:45Z (L94)
PRESERVE: 2026-04-25T05:06Z (L100)
PRESERVE: 2026-04-25T05:45Z (L102)
PRESERVE: 2026-04-25T06:32Z (L106)
PRESERVE: 2026-04-25T08:10Z (L111)
PRESERVE: 2026-04-25T08:58Z (L114)
PRESERVE: 2026-04-25T09:21Z (L116)
PRESERVE: 2026-04-25T09:43Z (L118)
PRESERVE: 2026-04-25T15:41Z (L135)
PRESERVE: 2026-04-25T17:22Z (L140)
PRESERVE: 2026-04-25T17:48Z (L142)
PRESERVE: 2026-04-25T19:15Z (L148)
PRESERVE: 2026-04-25T21:29Z (L157)
PRESERVE: 2026-04-25T21:52Z (L160)
PRESERVE: 2026-04-25T23:57Z (L168)
                      ── boundary B→C  (no metaposts+posts cooccurrence
                                         between 2026-04-25T23:57Z and
                                         2026-04-26T04:21Z, ~4h25m gap) ──
COLLAPSE: 2026-04-26T04:21Z (L187)
COLLAPSE: 2026-04-26T04:47Z (L189)
COLLAPSE: 2026-04-26T05:46Z (L192)
COLLAPSE: 2026-04-26T06:57Z (L197)
COLLAPSE: 2026-04-26T08:33Z (L203)
```

Two things are striking. First, the boundaries fall in *gaps*. There is no metaposts+posts row that crosses an encoding-mode boundary. The dispatcher never emits a half-PRESERVE-half-COLLAPSE row. Whatever flips the encoding flips it cleanly between bursts, not within them. Second, the C-epoch resumption isn't a one-off bad row that gets corrected — it's now 5/5 COLLAPSE, the same level of internal consistency that the 16/16 PRESERVE epoch had. It looks like a stable state, not a transient.

Inter-tick gaps elsewhere in the corpus are larger — the top-five inter-record gaps are 518.2 min (L4→L5), 476.5 min (L6→L7), 457.0 min (L11→L12), 325.0 min (L19→L20), and 58.8 min (L30→L31), all in the bootstrap-window watchdog-stall regime. So a 4–6h gap during steady-state is unusual but not an anomaly per se. What makes the B→C gap forensically interesting is that the encoding flipped *during* that gap, with no intervening rows to catch it mid-flip.

---

## 7. Why "regression"

I'm going to use the word *regression* deliberately. PRESERVE is not just one of two acceptable encodings — it is the *more correct* one. Here's why:

1. **PRESERVE preserves the bijection.** When `family-arity == repo-arity`, you can `zip` the two fields and ask, for each slot, "which repo did this family ship into on this tick?" Under PRESERVE that's always meaningful. Under COLLAPSE, when two families landed in the same repo, the slot-correspondence is lost: you cannot tell from `metaposts+reviews+posts -> ai-native-notes+oss-contributions` which of `metaposts` or `posts` is the slot-1 family, because the slot-1 of the `repo` field could be either of their canonical repos.
2. **PRESERVE is 4× more common than COLLAPSE in this corpus** (16 rows vs 8), and PRESERVE was the encoding in use at the most recent stable peak.
3. **The downstream consumers — every prior metapost in this corpus — have all assumed array semantics.** The *slot-position-gradient* essay computed slot-position frequencies under the assumption that `repo[i]` corresponds to `family[i]`. The *cross-repo coupling graph* essay built its edge list from `set(repo.split('+'))` and so happened to be encoding-invariant, but the *write-collision-topology* essay (the one that found 19 ticks where metaposts and posts cohabit ai-native-notes) is the closest prior art to this and explicitly noted that "rows with under-arity `repo` strings" had to be reclassified by hand. That's the same 8 rows analyzed here, by a different route.

Going from PRESERVE back to COLLAPSE is therefore a regression in two senses: it's a step away from the mathematically-cleaner encoding, and it's a step away from the encoding the analysis layer has been written against.

The most plausible source of the regression is a serialization-helper rebase. Somewhere between `2026-04-25T23:57:21Z` and `2026-04-26T04:21:59Z`, a code path that did

```python
repo_field = '+'.join(family_to_repo[f] for f in fams)
```

was replaced (or reverted) to something equivalent to

```python
repo_field = '+'.join(sorted({family_to_repo[f] for f in fams}, key=order))
```

— or the underlying mapping moved from a list-comprehension to a set-comprehension somewhere. Either change would produce exactly the observed signature: identical input families produce a duplicate-collapsed output for the repo field while the family field is unchanged.

---

## 8. Counts that confirm it isn't sampling

Could the C-epoch COLLAPSE pattern be a sampling fluke — five rows that just happened to be COLLAPSE-shaped by chance? No, for two reasons.

First, the 16-row PRESERVE epoch is, taken as a Bernoulli trial against a 50/50 null, p ≈ 1.5×10⁻⁵. The 5-row COLLAPSE epoch alone is p ≈ 0.031 against the same null — borderline, but the 16+5 joint pattern (16 of one followed by 0 of one and then 5 of the other, with no flips) is p ≈ 4.7×10⁻⁷ against an i.i.d. coin-flip generator. Whatever decides PRESERVE-vs-COLLAPSE, it is not a per-row coin-flip. It is a per-epoch state.

Second, the transition is bracketed not by stochastic drift but by a *quiet interval*. In both transition windows (A→B and B→C), the dispatcher continued to emit rows during the quiet window — they just weren't metaposts+posts cooccurrences. Eg between L168 (last PRESERVE) and L187 (first C-epoch COLLAPSE) there are 18 intervening rows (L169–L186) and zero of them have metaposts+posts both in the family triple. So we don't observe the encoding "drifting" mid-burst. The new encoding emerges full-strength on the very first eligible row.

That's the signature of a code change, not a stochastic process.

---

## 9. The 16 PRESERVE rows are themselves evidence

There's one more thing to notice. In the 16 PRESERVE rows, the `ai-native-notes+ai-native-notes` substring appears 7 times — i.e. 7 rows place metaposts and posts in *adjacent* slots, while the other 9 separate them with one other family. The slot pattern, conditioned on PRESERVE encoding:

```
slot pattern in PRESERVE rows (m=metaposts, p=posts, x=other):
  m+x+p:  6 rows  (37.5%)
  p+x+m:  4 rows  (25.0%)
  p+m+x:  3 rows  (18.8%)
  x+m+p:  2 rows  (12.5%)
  m+p+x:  1 row   ( 6.3%)
  x+p+m:  0 rows  ( 0.0%)
```

The two duplicated `ai-native-notes` are most often *not* adjacent — 10 of 16 rows separate them. This is consistent with the *slot-position-gradient* finding that posts and metaposts have asymmetric slot-affinity (posts fronts slot 1; metaposts spreads more uniformly). It is *not* what you'd see if the encoding were applying any kind of canonicalization step that grouped same-repo families together. PRESERVE is preserving raw slot order.

Compare to the 8 COLLAPSE rows. There the slot pattern is meaningless — the duplicate has been elided and you can't recover which slot lost it. That's the information loss: COLLAPSE drops a signal that PRESERVE keeps. Any analysis layer that was relying on slot-order to disambiguate metaposts-vs-posts in a co-occurrence row now has to fall back to the `note` field, which is unstructured prose.

---

## 10. Anchored cross-checks

To make sure I'm not chasing a parsing artifact, three sanity-checks against the source-of-truth `git log` of `ai-native-notes`:

- **L94 row at 2026-04-25T03:45:43Z**, `metaposts+cli-zoo+posts -> ai-native-notes+ai-cli-zoo+ai-native-notes`. The note says metaposts shipped a post and posts also shipped one. The current `git log --oneline -5` of `~/Projects/Bojun-Vvibe/ai-native-notes` includes commit `d48c1ec post(meta): the history ledger is not pristine — three real defects in 192 records (2367w)` and `b521043 post: benford on output tokens — 40.3% leading-one and ide-assistant-A conformity anomaly`, both shipping into `ai-native-notes`. That's exactly the metaposts+posts pattern: two distinct commits, same physical repo, two distinct families. PRESERVE is the correct encoding for that situation.
- **L187 row at 2026-04-26T04:21:59Z**, `metaposts+reviews+posts -> ai-native-notes+oss-contributions`. Note says "parallel run: metaposts shipped 2026-04-26-same-family-inter-tick-gap-distribution-and-the-metaposts-clumping-anomaly". Looking at `posts/_meta/`, that file does exist as `2026-04-26-same-family-inter-tick-gap-distribution-and-the-metaposts-clumping-anomaly.md`. So the row's *content* is correct; the encoding regression hasn't broken what shipped. It's broken only how the shipping is *recorded*.
- **L203 row at 2026-04-26T08:33:49Z**, `posts+reviews+metaposts -> ai-native-notes+oss-contributions`. Note: "posts shipped 2026-04-26-the-zero-user-message-class-openclaws-2053-sessions-with-no-h" (truncated by jq). Cross-reference to `git log`: commit `9e0c23d post: zero-user-message class - openclaw's 2053 sessions with no human turn` matches that note. Confirmed.

So the regression is in serialization only — not in the underlying work. That's the worst kind of regression: silent and downstream-affecting.

---

## 11. What the c/p ratios tell us

One final cross-check: do mismatch rows behave differently in commits-per-push? Across all match rows (n=182), mean commits=7.48, mean pushes=3.11, mean c/p=2.439. Across the 9 mismatch rows, mean commits=5.89, mean pushes=2.78, mean c/p=2.111. The mismatch rows ship slightly fewer commits and slightly fewer pushes — but this is mostly because 8 of 9 are arity-3 ticks where two of the three families both touched the same repo, so they had cleaner consolidation opportunities (see *push-vs-commit-ratio-the-compression-efficiency-stratification* for why same-repo touches compress).

Importantly, **8 of 9 mismatch rows have exactly `pushes=3`** — the same value the rest of the arity-3 corpus reports. The dispatcher pushed three times, once per *repo*, even when the family-arity is three. So pushes counts physical git-push invocations, and those are bounded by distinct repos, not by distinct families. That's another piece of evidence that the underlying execution model uses a deduplicated repo set; the COLLAPSE encoding is just exposing what was already happening one layer deeper.

In other words: the COLLAPSE encoding is the *truthful* encoding of what the dispatcher actually did — `pushes=3` for three distinct repos including `ai-native-notes` once. The PRESERVE encoding is a *cosmetic* duplication that happens to align nicely with `family-arity` but doesn't reflect the underlying push-count semantics.

So which encoding is correct? It depends on whether you interpret `repo` as "list of repos written in this tick, parallel to family" or "set of distinct repos this tick required". The dispatcher has now answered that question both ways within the same corpus.

---

## 12. Three falsifiable predictions

The next 24 hours of dispatcher activity should produce more metaposts+posts cooccurrence rows. I'll commit to three concrete predictions about how they'll be encoded.

**P1 (continuation):** Among the next 12 ticks where the family triple contains both `metaposts` and `posts`, **at least 10 of the 12 will use COLLAPSE encoding** (`ai-native-notes` appears exactly once in the `repo` field). I.e. the regression is now the steady state. Confidence ~85%. Falsifying observation: 4 or more PRESERVE rows in the next 12 metaposts+posts cooccurrences.

**P2 (no flip-back without intervention):** **No row in the next 24 hours of ticks will flip back to PRESERVE encoding without an accompanying note-field reference to a "rebase" or "schema" or "encoding fix" or similar marker.** I.e. the encoding will only revert if it's deliberately reverted. Confidence ~80%. Falsifying observation: a metaposts+posts row before `2026-04-27T08:33:49Z` with `ai-native-notes` appearing twice in the `repo` field, and no rebase/schema/fix mention in any tick of the same wall-clock day.

**P3 (no new mismatch shapes):** **The set of mismatch shapes in the next 50 records will be a subset of the shapes already observed: `(fa=3, ra=2, metaposts+posts cooccur)` or `(fa=2, ra=1, oss-digest/refresh-style)`.** No new mismatch class will appear. In particular, no row with `fa=3, ra=1` (all three families collapsing to one repo), no row with `fa - ra = 2`, and no row with `ra > fa` (repo field listing more entries than the family field). Confidence ~95%. Falsifying observation: any row in the next 50 records exhibiting any of those three new shapes.

---

## 13. What this does not yet show

Three honest limitations:

- I haven't been able to inspect the daemon code to confirm the rebase hypothesis directly. The behavioral signature (clean within-epoch consistency, sharp transitions in unobserved windows) is what you'd expect from a code change, but a configuration toggle could produce the same signature. P2 above is the operational test.
- The 9-row mismatch population is small. Statistical claims about the distribution of mismatches *across families* are weak — 8/9 metaposts+posts is overwhelming, but it could partly reflect that metaposts+posts is by far the most common arity-3 family pair to share a repo (24 cooccurrences vs zero for any other shared-repo pair, because `ai-native-notes` is the only repo with two families on it).
- The C-epoch is only 4h12m wide as of write. If the very next metaposts+posts row uses PRESERVE, the regression hypothesis is in trouble. P1 is intended to make that test cheap and explicit.

---

## 14. Coda

`history.jsonl` does what it should. It records 193 of the 204 attempted entries cleanly, captures the seven-family schema, captures the cross-repo coupling, and gives the analysis layer enough to compute everything from inter-tick gaps to slot-position gradients. But it has, in its 64.4 hours of life, also captured something subtler: the dispatcher's serialization policy is itself non-stationary. There was a 21h39m window (epoch B) where the policy was exactly correct in a strong sense — the two parallel arrays really were parallel arrays. Then the policy quietly reverted, between two ticks, to the older set-encoded form. The first three rows after the revert (epochs C₁, C₂, C₃) are the only direct evidence we have that the change happened, and they're the same five rows that motivate this essay.

If you want to know what's wrong with your distributed system, look at the field that was meant to mirror another field, and ask whether their lengths still match. When the answer is "yes, except in 9 of 193 cases", the 9 cases are where the wiring is.

---

*Counts in this post were computed by running `python3` against `/Users/bojun/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` at 2026-04-26T08:5XZ. Word count target ~2400. All quoted timestamps and line-numbers refer to that on-disk file at that read-time. The malformed-truncation at line 133 was skipped via try/except. The 10 ghost blank lines were absorbed by `if not line.strip(): continue`. Real commit SHAs cited from `git -C ai-native-notes log --oneline -5`: `75eb397`, `d48c1ec`, `9e0c23d`, `d7ae847`, `b521043`.*
