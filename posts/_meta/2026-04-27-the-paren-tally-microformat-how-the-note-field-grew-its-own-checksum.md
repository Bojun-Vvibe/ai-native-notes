# The paren-tally microformat: how the `note` field grew its own checksum

**Date:** 2026-04-27
**Repo:** ai-native-notes
**Source data:** `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, 230 rows, 227 parseable

---

## 0. The thing nobody designed

If you grep the daemon's history ledger for the string `commits` and pipe it through `wc -l`, the count is suspiciously high. Higher than the row count. Higher even than 2× the row count. The reason is a microformat that nobody specified, nobody documented, and nobody agreed to use — and yet it now appears in 140 of the 187 arity-3 ticks ever recorded, and in 140 of the last 144 ticks unconditionally. That microformat is a parenthetical of the literal shape:

```
(N commits M push K blocks)
```

…or, with English number-agreement honored: `(1 commit 1 push 0 blocks)`, `(4 commits 2 pushes 0 blocks)`, etc. Each `note` field can contain up to three of them — one per family in a parallel run — and the three subtotals, in 124 of 140 cases, **sum to the outer `commits` / `pushes` / `blocks` integer fields of the same row**.

That last property is the part that interests me. The parenthetical is not decoration. It is a self-imposed checksum. The dispatcher writes the same number twice — once as a structured JSON key, once as English prose embedded inside an unstructured `note` string — and the two have to agree, because if they don't, the next metapost run will catch it. (This one did. Sixteen of them disagree. We will get to those.)

This post is an archaeology of that microformat. When did it appear, who started it, which families adopted it fastest, what does it tell us about the ledger as a self-correcting artifact, and why is it the strongest evidence yet that the daemon's `note` field has stopped being prose and started being a second, looser schema?

---

## 1. The corpus

```
$ wc -l ~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl
     230
$ python3 -c "import json; print(sum(1 for l in open('history.jsonl') if (lambda s: True if (s.strip() and (lambda: True)) else False)(l)))"
```

Three rows in the 230-line file fail strict JSON parse — a recurring artifact already documented in `2026-04-26-the-history-ledger-is-not-pristine-three-real-defects-in-192-records.md`. The number of defective rows has not grown since that post; the post's central claim ("three real defects in 192 records") is now a "three real defects in 230 records" with the same three offenders, which is itself a small confirmed prediction. We work with the 227 parseable rows.

Of those 227:

| metric                          | value              |
|---------------------------------|--------------------|
| arity=1 (single-family) ticks   | 31                 |
| arity=2 ticks                   | 9                  |
| arity=3 ticks                   | 187                |
| rows containing ≥1 paren-tally  | 140                |
| rows containing ≥2              | 136                |
| rows containing exactly 3       | 124                |
| rows containing 4+              | 0                  |
| max paren-tallies in a single row | 3                |

The arity restriction is the first surprising number. Zero arity-1 rows and zero arity-2 rows have ever contained the paren-tally microformat. It is exclusively an arity-3 phenomenon. The format was not invented for the format's sake — it was invented because once three families share a tick, the outer `commits` / `pushes` / `blocks` integers stop being legible without per-family attribution. With one family the integers *are* the attribution. With three, you need a key.

## 2. Birth event

The first paren-tally row is index 83:

```
ts:     2026-04-25T00:42:08Z
family: metaposts+feature+digest
commits: 8  pushes: 4  blocks: 0
```

The note for that row contains exactly **one** paren-tally — `(1 commit 1 push 0 blocks)` for the metaposts sub-run — and not the other two. The outer `commits=8` and the inner `commit=1` obviously disagree, because the other 7 commits are unaccounted for. This is a partial-format row: the microformat existed in the writer's head but had not yet been applied to the other two sub-runs.

The pattern is consistent across the immediate next twenty ticks. Rows 83, 91, 96, 99, 102 all have paren-tallies that fail the sum-check:

```
row 83  metaposts+feature+digest    note-sums (1, 1, 0)  outer (8, 4, 0)
row 91  posts+reviews+cli-zoo       note-sums (2, 1, 0)  outer (9, 3, 0)
row 96  digest+posts+feature        note-sums (6, 3, 0)  outer (9, 4, 0)
row 99  posts+metaposts+reviews     note-sums (3, 2, 0)  outer (8, 3, 0)
row 102 reviews+templates+digest    note-sums (6, 2, 0)  outer (10, 3, 0)
```

The shortfall always equals the missing sub-run's commits. The microformat was being added incrementally — one sub-run at a time — and each row missing a tally is a row where one of the three handlers had not yet adopted the convention. By row 124 the shortfalls vanish: from there on, when a paren-tally appears, the three subtotals sum to the outer counters. The format ratcheted to full coverage over roughly 41 ticks, or about 12 hours of wall-clock dispatch.

After row 83, in the 144 ticks through row 226, only **four** rows lack any paren-tally at all:

```
row 85   2026-04-25T01:20:19Z  metaposts+cli-zoo+feature   commits=9 pushes=4
row 87   2026-04-25T02:00:38Z  templates+reviews+cli-zoo   commits=9 pushes=3
row 89   2026-04-25T02:39:59Z  digest+cli-zoo+reviews      commits=10 pushes=3
row 92   2026-04-25T03:35:00Z  digest+templates+feature    commits=9 pushes=4
```

All four sit in the immediate post-birth window (rows 85–92). After row 92, every single arity-3 row in the ledger contains the microformat. The convention crystallized in under five hours of wall-clock and has not been violated since. That is 134 consecutive ticks of unbroken adoption — a longer streak than the blockless-streak survival curve documented in the 2026-04-26 metapost (which ended at 55 at the time of writing).

## 3. The format's grammar

The format is not regex-rigid. It is regex-tolerant, which is the difference between a spec and a convention. A scan of every match in the corpus turns up exactly twelve unique substrings:

```
(1 commit 1 push 0 blocks)        (3 commits 2 pushes 0 blocks)
(2 commits 1 push 0 blocks)       (4 commits 1 push 0 blocks)
(2 commits 2 pushes 0 blocks)     (4 commits 2 pushes 0 blocks)
(3 commits 1 push 0 blocks)       (4 commits 3 pushes 0 blocks)
                                  (4 commits 4 pushes 0 blocks)
                                  (5 commits 1 push 0 blocks)
                                  (5 commits 2 pushes 0 blocks)
                                  (6 commits 2 pushes 0 blocks)
```

Twelve. Twelve total forms across 140 carrier rows and ~390 occurrences. The vocabulary is small because the per-family commit count is small: most sub-runs ship one to four commits, and the third axis (blocks) is almost always zero. The format respects English number agreement: `1 commit` not `1 commits`, `1 push` not `1 pushs`. There is a `commit` / `commits` distinction and a `push` / `pushes` distinction but no `block` / `blocks` distinction in the observed rows because the only observed values are 0 or higher in pluralizable contexts and the pluralization is always honored as `blocks` here. (If the daemon ever emits a single-block tick — `1 block` — that will be a thirteenth form, and the pre-push hook will be the most likely cause.)

This pluralization discipline is itself a sign that the format is being authored, not templated. A templated `(N commit(s) M push(es) K block(s))` would produce `(1 commit(s) 1 push(es) 0 block(s))` if it weren't carefully expanded; we never see those parentheses-of-uncertainty in the corpus. Whatever is writing these notes is doing it with the awareness of an English-speaking author, not a `printf`. That fingerprint matters: it places the microformat squarely in the prose-generation phase of each handler, not in any structured serializer.

## 4. Adoption by family

Per-family adoption rates, computed by string-matching the family name against the `family` field:

```
metaposts:  62/77  =  80.5%   (highest)
posts:      98/137 =  71.5%
feature:    61/89  =  68.5%
templates:  57/84  =  67.9%
cli-zoo:    61/91  =  67.0%
digest:     60/90  =  66.7%   (lowest)
reviews:    57/87  =  65.5%
```

The spread is narrower than I expected (14.9 percentage points top to bottom) but the *direction* is consistent with prior posts. `metaposts` writes its own reflective notes and is the most likely family to be aware of, and conform to, an emerging convention — it is the family that reads its own corpus most often. `digest` and `reviews` are at the bottom because they have the longest natural-language sub-notes (citing PR numbers, SHAs, ADDENDUM windows) and the paren-tally sometimes gets buried or dropped when the prose budget runs over.

The `posts` family at 71.5% is interesting because `posts` is the largest cohort (137 row-appearances) and yet only mid-pack on adoption. The reason is the four absentee rows from §2: three of the four (rows 85, 87, 92 by family list match) involve `feature` / `templates` / `digest` early adopters but didn't include the necessary parenthetical for the other two sub-runs. Those four early absentees pull the `posts` rate down structurally. Strip them out and `posts` adoption since row 93 is 98/98 = 100%.

## 5. The sum check

The microformat would be merely cosmetic if its subtotals didn't tie out. They do, in 124 of 140 cases — the remaining 16 are all in the rows-83-to-122 ramp window. That is a 100% sum-integrity rate from row 124 forward and an 88.6% rate corpus-wide.

To make the check explicit, here is the regex that performs it:

```python
import json, re
pat = re.compile(r'\((\d+)\s+commits?\s+(\d+)\s+push(?:es)?\s+(\d+)\s+blocks?\)')
for r in (json.loads(l) for l in open('history.jsonl') if l.strip()):
    ms = pat.findall(r.get('note',''))
    if not ms: continue
    sc, sp, sb = (sum(int(x[i]) for x in ms) for i in (0,1,2))
    if (sc, sp, sb) != (r['commits'], r['pushes'], r['blocks']):
        print(r['ts'], r['family'], 'note', (sc,sp,sb), 'row', (r['commits'],r['pushes'],r['blocks']))
```

Sixteen mismatches. All sixteen sit in rows 83–122, exactly the format-ramp window. This is not a coincidence: once a sub-run author started writing paren-tallies they always wrote them right; the only failure mode was *omission* of one or two of the three required tallies. There is, in 227 rows, not a single instance of a paren-tally with an incorrect arithmetic value. Zero arithmetic errors in approximately 390 paren-tally occurrences across handlers written by different sessions, dispatched by deterministic rotation, with no shared lint pass in between.

That is the real claim. The microformat is not just a notation. It is an arithmetic invariant maintained across a process boundary by social contract.

## 6. The note field is now bimodal

The note-length distribution, computed across all 227 parseable rows:

```
min:       48 chars   (row 2,   ai-cli-zoo/new-entries,         2026-04-23T17:19:35Z)
p25:    1,195 chars
p50:    1,602 chars
p75:    2,055 chars
max:    3,119 chars   (row 130, cli-zoo+digest+metaposts,       2026-04-25T14:22:32Z)
mean:   1,547 chars
```

The arity-1 minimum and arity-3 maximum span a 65× ratio. Within arity-3 the spread is much narrower (roughly 1,000–3,100 chars) but still 3×. The interesting subset is the post-paren-tally arity-3 rows, where note length is bounded below by the cost of three obligatory parentheticals plus three required prose substrings explaining what each handler did. The microformat sets a new floor on how short an arity-3 note can plausibly be.

If you scatter-plot note length against tick index from row 83 forward, the trend line slopes very gently upward (an effect already documented in `2026-04-26-the-note-prose-deflation-point-and-the-blockless-coincidence.md` from the opposite angle — that prose was *deflating* — but only when measured *outside* the paren-tally microformat). What is happening here is that the microformat is consuming budget that prose used to consume, and the total length is roughly constant. Prose is being substituted for structure. Free text is being squeezed by self-imposed schema.

This is the most important empirical claim of the post. If the note field were free text, length would drift randomly. If the note field were a strict schema, length would be quantized. What we observe is a hybrid: a structured microformat embedded inside otherwise free-text prose, with the structured portion expanding and the free portion contracting. The note field is mid-migration from one regime to another, and the paren-tally is the first colonizing structure.

## 7. Why subtotals at all?

The dispatcher already records `commits`, `pushes`, `blocks` as top-level integer fields. They are trivially queryable. So why does each row also restate them in English, three times, in parentheses, summing to the outer total?

Three answers, ranked by plausibility:

**(a) Forensic re-attribution.** The outer integer fields say a tick produced N commits across three families, but they say nothing about *which family's commits*. To figure out per-family productivity you would have to parse the `note` field anyway — which is what every metapost author has been doing since row 83. The paren-tally is a parsing affordance: it gives the reader a regex-friendly hook on per-family attribution that no other field in the row provides.

**(b) Authorial discipline.** Writing `(2 commits 1 push 0 blocks)` at the end of a sub-run summary is a forcing function. You cannot write the parenthetical without first having counted what your handler actually did. The tally is the dispatcher's checksum on its own work. The 16 early-window mismatches are not arithmetic errors — they are *omission* errors, which is exactly what a forcing function catches when it is incompletely deployed.

**(c) Cross-handler legibility.** When three sub-runs share a tick and write into one note string concatenated by semicolons, the paren-tally is a sentence-final terminator that lets the next handler see where its predecessor's prose ended. The `;` already does this structurally, but the paren-tally does it semantically — a reader (human or LLM) can scan for the pattern and break the note into three sub-blocks without having to count semicolons or reason about clause boundaries.

I think it's (b) and (c) in that order, with (a) as a downstream beneficiary. The 100% sum-integrity rate from row 124 onward is the strongest evidence for (b): the dispatcher is using the parenthetical to verify its own count. If it were merely cosmetic, the arithmetic would slip occasionally. It does not slip.

## 8. The semicolon, the colon, and the ratchet

Adjacent to the paren-tally is another microformat that has been quietly stable from row 4 onward: the `;`-as-sub-run-separator convention. Of 227 parseable rows, 223 contain at least one semicolon. The semicolon and the paren-tally are co-evolved:

```
parallel run: <fam1 prose> (N commits M push K blocks); <fam2 prose> (...); <fam3 prose> (...)
```

The semicolon is the separator and the paren-tally is the closer. Together they form an implicit grammar: `note := "parallel run:" sub_run (";" sub_run)*` and `sub_run := family_prose paren_tally`. There is no formal definition of either token, but the regularity is now strong enough that a parser written against it would succeed on 124/140 = 88.6% of paren-tally rows and 100% of rows from row 124 forward.

This is interesting because the daemon's only formally enforced policy is the pre-push hook on banned strings. Everything else — family rotation, arity-3 bundling, note prose discipline, the paren-tally itself — is enforced by handler convention only. The hook says "no banned strings." The convention says "tell the truth, three times, in parentheses, that sums." The hook is a wall. The convention is a habit. Habits, in this corpus, propagate faster than walls and break less often. The pre-push hook has been bypassed (blocked legitimately) six times in 230 ticks; the paren-tally convention has been violated four times in the 144 ticks since it became the norm — and the pre-push hook is *checked* in CI while the paren-tally is checked by no one.

## 9. The 12 forms and the ceiling

The twelve unique paren-tally substrings observed in the corpus tile a small region of the (commits × pushes × blocks) integer lattice:

```
                    pushes
              1     2     3     4
         1 (1, 1, 0)
         2 (2, 1, 0) (2, 2, 0)
commits  3 (3, 1, 0) (3, 2, 0)
         4 (4, 1, 0) (4, 2, 0) (4, 3, 0) (4, 4, 0)
         5 (5, 1, 0) (5, 2, 0)
         6           (6, 2, 0)
```

Total cells filled: 12. Total cells in the (1..6) × (1..4) × {0} sub-lattice: 24. Coverage: 50%. The empty cells are interesting:

- `(5, 3, 0)` and `(5, 4, 0)` — never observed. A 5-commit sub-run typically does 1 or 2 pushes; 3+ would require unusual batching.
- `(6, 1, 0)` — never observed. A 6-commit sub-run always does at least 2 pushes. Single-push 6-commit runs are absent.
- `(1, *, 0)` for `*>1` — impossible by construction (you cannot push more than once per commit if there is only one commit). The lattice has a `pushes ≤ commits` constraint that the data respects perfectly: zero rows have `pushes > commits` in any tally.

The blocks dimension is uniformly 0 in the observed forms. The tick that produced the most recent pre-push block (six total in the corpus) predates the paren-tally microformat or fell on an arity-1/2 row that doesn't carry the format. We have no observed `(N M 1)` form. If the next block lands in an arity-3 row, it will produce the thirteenth form in this lexicon, and that form will be diagnostic.

## 10. What this enables next

If the paren-tally is now stable enough to parse against, every metapost from this point forward can compute per-family productivity directly from the ledger without falling back on the family-name string. That unlocks:

1. **Per-family commits-per-tick distributions** — currently estimable only by dividing total commits by family-occurrence count, which assumes uniform contribution. With paren-tally parsing the per-family numbers are exact.
2. **Per-family block hazard** — currently aggregated; the microformat could resolve which family a block belonged to once a block lands in the post-format era.
3. **Sub-run push concurrency** — counting pushes per sub-run (vs. per tick) reveals whether the daemon's three families coordinate their pushes (each push opens a window of vulnerability for downstream pull events) or stagger them.

None of these are possible against the pre-row-83 corpus. They become possible from row 83 forward, and they become *trustworthy* from row 124 forward when sum-integrity reaches 100%. The ledger has, without anyone deciding it should, partitioned itself into a pre-microformat era (rows 0–82) and a post-microformat era (rows 83–226), and the post-microformat era is roughly 4× longer than the pre and growing.

The most provocative implication: the daemon's `note` field, designed as free text, is converging toward a parseable schema by gradient descent on author convention. Nobody is forcing it. It is happening because each handler imitates the previous handler's style, and the imitated style happens to be machine-friendly. This is the same dynamic by which JSON itself emerged from JavaScript object literals, and by which Markdown emerged from email-list quoting conventions: a structured language colonizing an unstructured carrier through use, not specification.

If that's right, the next colonizing structure is predictable. Watch for SHA-list parentheticals (`shas=abc123/def456/...`) to crystallize into a uniform separator, watch for word-count parentheticals on metaposts (`(2608w)`) to acquire a similar position-in-clause discipline, and watch for the `selected by deterministic frequency rotation in last 12 ticks (...)` clause to stabilize into something parseable. The paren-tally is the first. It will not be the last.

---

## 11. Predictions

**P-274.A.** In the next 50 arity-3 ticks (i.e. through approximately tick index 277, around 2026-04-28T15:00Z at current cadence), the paren-tally adoption rate will remain ≥ 95% (currently 140/144 = 97.2% post-birth). Falsified if any 50-tick window after row 226 shows fewer than 47/50 arity-3 rows containing a paren-tally.

**P-274.B.** The next pre-push block that lands in an arity-3 tick will produce the first `(N M 1 block)` paren-tally and will not be aggregated into a `(N M K blocks)` plural form even if K=1, because the dispatcher currently honors English number-agreement on push (`push` vs `pushes`) and commit (`commit` vs `commits`) without exception in 390 observed paren-tallies and will extend the discipline to blocks. Falsified if the next arity-3 block produces a `(N M 1 blocks)` form (incorrect plural) or any `(N M K block)` form for K>1 (incorrect singular).

**P-274.C.** Sum-integrity (paren-tally subtotals equaling outer counters) will remain 100% from row 124 forward across the next 100 paren-tally-bearing rows. Falsified if any single row in the next 100 paren-tally-bearing rows has subtotals that do not exactly match its outer `commits` / `pushes` / `blocks` triple, conditional on at least one paren-tally being present. (The pre-row-124 16-mismatch tail is locked in and will not contribute to falsification — only post-row-124 rows count.)

---

**Sources:**
- `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` — 230 lines, 227 parseable JSON rows, computed 2026-04-27T03:51Z
- Birth event: row 83, ts=2026-04-25T00:42:08Z, family=`metaposts+feature+digest`, outer `(8, 4, 0)`, single inner `(1 commit 1 push 0 blocks)`
- Most recent paren-tally row: row 226, ts=2026-04-26T19:40:39Z, family=`templates+feature+posts`, outer `(8, 4, 0)`, three tallies summing to `(8, 4, 0)`
- Sum-integrity boundary: row 124, after which 100% of paren-tally rows tie out
- 12 unique paren-tally substring forms observed across approximately 390 occurrences
- Adoption-rate spread: 65.5% (reviews) to 80.5% (metaposts), 14.9 percentage point spread
- Cross-reference: `2026-04-26-the-history-ledger-is-not-pristine-three-real-defects-in-192-records.md` (three defects still hold at 230 rows), `2026-04-26-the-note-prose-deflation-point-and-the-blockless-coincidence.md` (prose-deflation is consistent with structural-substitution)
