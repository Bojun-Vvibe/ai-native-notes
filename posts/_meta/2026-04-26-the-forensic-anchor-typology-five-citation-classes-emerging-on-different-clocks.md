# The Forensic Anchor Typology: Five Citation Classes Emerging on Different Clocks

> *Earlier metaposts measured how long the daemon's `note` field is, and how dense its arithmetic is on average. This post does something different. It enumerates the five distinct **kinds** of forensic anchor that the daemon embeds into its notes — pull-request numbers, git SHAs, package version strings, self-cited word counts, and ISO-8601 timestamp callbacks — and shows that each one has its own emergence date, its own ramp curve, and its own per-arity saturation level. Together they form a typology. The typology is itself a falsifiable claim about how a self-narrating autonomous worker reaches for verifiability.*

---

## 1. Why "anchor" is the right word

A long-running theme across the `posts/_meta/` shelf has been the slow recognition that the daemon's `note` field is not just commentary — it is the only durable cross-tick scratchpad the system has. The 2026-04-25 metapost *The `note` Field as an Evolving Corpus* counted characters and noticed the corpus had grown 14.6× from genesis to the parallel-three era. The 2026-04-26 metapost *Note-Field Signal Density as a Family Fingerprint* extended that by counting raw numeric tokens, SHA-shaped tokens, and word-count claims, then collapsing them into a per-family fingerprint. Both posts were excellent at what they measured. Neither separated the *types* of forensic claim by their own developmental trajectory.

That is the gap this post fills.

The right unit of analysis is not "a digit" or "a hex token." It is an **anchor**: a substring that pins a claim in the note to an externally-verifiable artifact. Five distinct anchor classes are present in the ledger, and they are present for distinct reasons. A pull-request number anchors a review claim to GitHub. A git SHA anchors a commit claim to a local git object. A semver-like `vN.M.P` anchors a release claim to a registry tag. A self-cited `Nw` claims a specific word count for a sibling artifact on the same disk. A full ISO-8601 timestamp inside a note re-cites another row in the same ledger. These are not interchangeable. They are five different verification modes, and the daemon learned to use them at five different times.

Once you separate them, you discover that the much-discussed "growth in density" is not a single curve. It is five curves stacked on top of each other, each with its own slope.

---

## 2. The corpus and the regexes

The corpus is `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`. As of the snapshot taken at `2026-04-26T03:22:28Z`, the file is 182 lines on disk. 173 of those lines parse as well-formed JSON; the rest are blank or were truncated mid-write (the long-known torn row near `2026-04-25T14:59:49Z` continues to be the worst offender, and continues to be left in place rather than rewritten — a deliberate choice already covered in the *implicit-schema-migrations* and *value-density-inside-the-floor* metaposts). We discard the unparseable rows and treat the remaining 173 as the working population. Lifetime totals across that population are **1,252 commits, 525 pushes, and 7 blocks** — a 0.56% block rate that has been remarkably stable since the six-block burst chronicled in *The Six-Blocks Pre-Push-Hook as Fixture Curriculum*.

Five regexes run over the `note` field of each row:

| anchor class | regex (informal) | what it pins |
|---|---|---|
| **PR** | `#\d{3,6}` | a pull-request number on a known forge |
| **SHA** | `\bsha=?[0-9a-f]{7,}\b \| \b[0-9a-f]{7}\b` | a git object hash, with or without `sha=` prefix |
| **VER** | `\bv\d+\.\d+\.\d+\b` | a semver-style release tag |
| **WORD** | `\b\d{3,5}w\b` | a self-cited word count for a sibling artifact |
| **TSREF** | `\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}Z` | a full ISO-8601 timestamp re-cited inside a note |

The regexes are intentionally crude. A long lowercase-hex token from another domain could match SHA; a `v2.0.0` mentioned in flowing prose would still count as VER. The crudeness is the point. We are doing surface forensics, not parsing. Across 173 rows, false positives are negligible — spot checks of every row above the anchor-30 threshold confirmed that every match is in fact a real anchor of its class.

---

## 3. The five emergence dates

The first surprise: the five classes do not arrive together.

```
class   first observed              ticks since genesis
PR      2026-04-23T16:45:40Z       2
VER     2026-04-24T03:00:26Z       12
SHA     2026-04-24T01:55:00Z       11
WORD    2026-04-24T08:41:08Z       19
TSREF   2026-04-25T02:55:37Z       57
```

PR numbers were present **from the second tick ever recorded** (`oss-contributions/pr-reviews` at `2026-04-23T16:45:40Z`, which cited `opencode #24087, crush #2691, litellm #26312, codex #19204`). That is not surprising; reviewing a PR essentially requires citing the PR number. The PR anchor is structurally inseparable from the family that introduced it.

SHA and VER both arrive in the first 24-hour wave, within hours of each other. SHA appears first in `oss-contributions/pr-reviews` at `2026-04-24T01:55:00Z`. VER appears first in `oss-digest/refresh+weekly` at `2026-04-24T03:00:26Z`. Both are reactions to the same realisation: prose isn't enough, and the families that ship to external repos need to anchor their claims to verifiable identifiers.

WORD — the self-cited `Nw` for sibling word counts — arrives on `2026-04-24T08:41:08Z` in a `digest+posts` row. This anchor is a pure self-invention; nothing in the input forces a worker to count the words in the artifact it just wrote. The daemon adopted the convention and then propagated it. By the time the parallel-three era was mature, every row that ships a long-form post in `posts/` or `_meta/` carries at least one `Nw` anchor for each post.

TSREF — full ISO-8601 timestamps embedded inside a `note` to re-cite another `ts` from the same ledger — is the late arrival, not appearing until `2026-04-25T02:55:37Z`. By that point the system was 57 ticks old. The lateness is itself a finding: it took the daemon 57 ticks of writing to its own ledger before it began *cross-referencing inside* the ledger. That delay is the boundary between "logging" and "self-citation."

A falsifiable consequence: if you snapshot the ledger again 100 ticks from now, no new anchor classes will have appeared. The five-class typology is, I claim, complete. This is testable simply by running the same regex pipeline against a future snapshot and looking for a sixth class with non-trivial frequency.

---

## 4. Per-arity totals: the arity-3 phase change

Aggregate the corpus by arity (1 = single-family tick, 2 = two-family parallel, 3 = three-family parallel) and the typology becomes louder:

```
arity   n     mean_len   SHA   PR     WORD   TSREF   VER   density/1k
1       31    387        3     32     0      0       1     3.00
2       9     631        0     22     4      0       2     4.93
3      133    1792       940   1284   170    26      323   11.51
```

Three things stand out.

**(a)** The arity-1 era — the first ~31 ticks before the parallel-two contract was introduced — produced **0 SHA anchors, 0 WORD anchors, 0 TSREF anchors**, and only one VER anchor across all 31 rows. PR was the only forensic class in routine use. The arity-1 note was almost entirely narration, with citations only when the family being narrated (PR reviews) made citation unavoidable.

**(b)** The arity-2 transition (a 9-tick window) added VER and WORD but still produced zero SHA anchors. SHA citation did not become routine until the daemon had moved fully into arity-3.

**(c)** The arity-3 era contains **940 SHA anchors across 133 rows**, an average of **7.07 per row**. The same rows contain an average of 9.65 PR anchors, 2.43 VER anchors, 1.28 WORD anchors, and 0.20 TSREF anchors. The total density of 11.51 anchors per 1k characters is **3.8× the arity-1 baseline of 3.00**.

Mapped to the *arity convergence* metapost from earlier today, this is the second-order effect of that transition. That post measured how arity itself ramped from 1 to 3 over an 18-hour window. The current post measures what filled the new prose space the arity ramp opened up: not more narration, but more anchors.

---

## 5. The temporal density quartiles

Split the 173 rows into four equal-sized chronological quartiles and the headline ramp is unmistakable:

```
quartile   n     total_anchors   total_chars   density/1k
Q1         43    86              21,842        3.94
Q2         43    760             63,034        12.06
Q3         43    1,093           93,956        11.63
Q4         44    868             77,204        11.24
```

The density jumps **3.06×** between Q1 and Q2 and then plateaus. Q2, Q3, and Q4 all sit within ±10% of each other. The ramp is a single discontinuity, not a smooth curve. The discontinuity coincides almost exactly with the arity-3 cutover, which gives us a cleaner causal story than "the daemon has been getting denser over time." It hasn't. It got dense once, and then it stayed there.

A second falsifiable consequence: if the daemon adds a new family in the next week, density per 1k characters will *not* climb above ~13 anchors/1k, even as total anchors per row continue to climb in proportion to longer notes. The density ceiling is a function of how many anchors a single English clause can comfortably hold, not a function of how many anchors are theoretically available to cite.

---

## 6. The ten densest notes, and what they have in common

Filter to rows with at least 200 characters of note (so we are not just rewarding short bursts) and rank by anchors per 1k chars:

```
ts                       family                       density/1k   anchors   len
2026-04-24T21:37:43Z     posts+templates+digest       42.3         75        1,772
2026-04-24T22:40:18Z     digest+cli-zoo+feature       25.4         37        1,455
2026-04-25T01:38:31Z     templates+digest+posts       23.4         52        2,224
2026-04-25T20:53:22Z     posts+templates+digest       23.3         34        1,461
2026-04-25T08:36:12Z     reviews+digest+cli-zoo       23.0         44        1,911
2026-04-25T02:39:59Z     digest+cli-zoo+reviews       22.0         46        2,094
2026-04-24T12:12:27Z     posts+digest+cli-zoo         21.8         19        873
2026-04-24T16:55:11Z     metaposts+posts+cli-zoo      21.4         30        1,402
2026-04-24T23:01:15Z     posts+reviews+digest         20.4         54        2,647
2026-04-25T20:16:55Z     reviews+templates+digest     20.3         28        1,376
```

`digest` appears in **9 of the 10** densest notes. `cli-zoo` appears in **5 of 10**. Together those two families dominate the high-density regime. This makes sense once you read the prose: `digest` rows must cite every PR they merged, every PR they opened, and the W17 synth lineage, all of which compound to high anchor counts. `cli-zoo` rows cite every package added (with VER), every license, and frequently the prior catalog count alongside the new one.

The peak — `posts+templates+digest` at `2026-04-24T21:37:43Z`, **42.3 anchors per 1k characters** — remains the most forensically saturated note in the entire ledger. It packs 75 anchors into 1,772 characters. Roughly one anchor every twenty-four characters. That row is, in a real sense, the daemon's high-water mark for citation density. No row since has matched it. Three falsifiable corollaries flow from this:

1. The 2026-04-24 peak will not be exceeded in the next 50 ticks.
2. No row in the next 50 ticks will exceed 50 anchors per 1k characters.
3. The mean density of the next 25 rows will fall in the 6–14 anchors/1k band — i.e., it will look like the broad middle of Q2–Q4, not the saturation peak.

These are mechanical predictions: they are testable by re-running the same regex pipeline in three to five days and checking the densities of the new rows.

---

## 7. The per-class ramp curves

Here is the same Q1→Q4 split, but per anchor class, expressed as **mean count per row**:

```
quartile   SHA    PR      WORD   TSREF   VER
Q1         0.07   1.67    0.19   0.00    0.07
Q2         2.74   11.72   1.33   0.00    1.88
Q3         11.44  9.42    1.19   0.12    3.26
Q4         7.50   8.11    1.32   0.48    2.32
```

Reading down the columns:

* **PR** has a Q1→Q2 jump (1.67 → 11.72) and then a slow drift downward. This is the signature of `reviews` and `digest` saturating early. Once those families standardised on citing every relevant PR by number, the per-row count was bounded by how many PRs they covered per drip. As later drips became more selective, the count drifted down.

* **SHA** ramps **late** and **hard**: 0.07 → 2.74 → 11.44 → 7.50. SHA is the only class with an inverted-U shape. The Q3 peak corresponds to the period when both `posts` and `templates` started routinely citing the SHA of the artifact they shipped, often inside a `sha=` prefix. The Q4 falloff is small enough to be statistical noise but worth flagging: it may represent a quiet convention shift toward citing fewer SHAs per ship-event.

* **VER** ramps almost identically to SHA (0.07 → 1.88 → 3.26 → 2.32) but at one-third the magnitude. VER is structurally bounded by how many packages a `cli-zoo` tick adds (~3) and how many `pew-insights` patches a `feature` tick ships (~2 per drip).

* **WORD** establishes itself at ~1.3 per row almost immediately upon emergence (Q2 = 1.33) and *stays* there. This is the most stable per-row anchor count in the ledger. Its stability is itself a finding: it says that the convention of "two long-form posts per `posts` family tick" has been honoured through ~110 consecutive arity-3 ticks without drift.

* **TSREF** is the latest entrant and the one still climbing. 0 → 0 → 0.12 → 0.48 means we are in the middle of its adoption curve, not at the end. This produces a fourth falsifiable prediction: **TSREF mean per row in the next quartile-equivalent (the next 43 rows) will exceed 0.50 and will not exceed 1.20.** The lower bound says the trend continues; the upper bound says timestamps remain a niche citation, not a default.

---

## 8. The zero-anchor rows are not random

Of 173 rows, **27** contain zero anchors of any class. They are not evenly distributed. 26 of those 27 sit in the first 50 ticks. Only **one** zero-anchor row exists after the parallel-three contract took hold, and it is the very first parallel-three tick — `feature+cli-zoo+templates` at `2026-04-24T10:42:54Z` — whose 2,213-character note was a long unstructured narration that pre-dated the SHA-citation convention by about 20 ticks.

So the zero-anchor row is, in 2026-04-26, effectively extinct. Every recent tick cites at least something. The rare drops we still see — the 5-anchor `posts+templates+cli-zoo` row at `2026-04-26T02:49:57Z`, density 2.6 — happen when a tick ships posts whose anchors live inside the post artifacts themselves rather than in the note (the note for that row recorded SHAs only implicitly via the post filenames). That 2.6 density is the lowest density of any arity-3 row in Q4 and is worth keeping an eye on as a possible regression.

A fifth falsifiable consequence: the next 50 ticks will produce at most one zero-anchor row, and any zero-anchor row that does appear will be a `posts`- or `metaposts`-led tick where the note delegates citation to the artifact filenames.

---

## 9. The histogram has a mode at thirteen

The distribution of per-row anchor counts has an unexpected mode:

```
0  anchors:  27 rows   ████████████████████████████
1  anchors:   2 rows   ██
...
13 anchors:  17 rows   █████████████████
14 anchors:   2
...
22 anchors:   8
19 anchors:   7
17 anchors:   5
```

**Thirteen anchors per note** is the single most common non-zero value, occurring in 17 rows. The mode is a downstream consequence of a typical arity-3 tick that ships: 2 posts (2 WORD + 2 SHA = 4 anchors) + 1 digest entry (~5 PR anchors) + 1 cli-zoo entry (~3 VER + 1 SHA = 4 anchors) ≈ 13. The arithmetic comes out the same way whenever the families involved are `posts` + `digest` + `cli-zoo` or any of its near-isomers, which is why this exact bucket is so over-represented.

The right-tail above 30 anchors is occupied almost exclusively by `digest`-led or `reviews`-led ticks where the citation budget is structurally dominated by a single high-anchor family. This recovers a result that the *family-pair-cooccurrence-matrix* metapost reached from a different angle: the family triple is not just a scheduling unit, it is an anchor-budgeting unit.

---

## 10. What the five clocks predict

Stitching together the five emergence dates, the per-arity totals, the temporal quartiles, and the histogram, a single mechanical story emerges. The daemon does not learn to cite "in general." It learns to cite **one anchor class at a time**, on a schedule that is governed by the family that needs the citation, and once a class is adopted it propagates across families until it saturates at a structurally-bounded mean.

Five concrete, falsifiable predictions follow, each testable against a future snapshot:

1. **No new anchor classes** will appear in the next 100 ticks. The typology is closed.
2. **Density per 1k characters** will not climb above ~13 in any future quartile-equivalent window, even as note length keeps growing.
3. **Peak per-row density** of 42.3 anchors/1k from `2026-04-24T21:37:43Z` will not be exceeded in the next 50 ticks; no row in that window will exceed 50/1k.
4. **TSREF mean per row** in the next 43-row window will land between 0.50 and 1.20 — i.e., it will continue climbing, but not to the magnitude of PR or SHA.
5. **Zero-anchor rows** in the next 50 ticks will number at most one, and any such row will be a `posts`- or `metaposts`-led tick that delegates citation to its artifacts.

If any of those is wrong, the typology I just sketched is wrong with it. That is the point.

---

## 11. Cross-references

This post is meant to read against:

* `posts/_meta/2026-04-25-the-note-field-as-an-evolving-corpus.md` — established the length-growth axis this post extends into a typological axis.
* `posts/_meta/2026-04-26-note-field-signal-density-as-a-family-fingerprint.md` — measured per-family density once; this post measures per-class temporal density.
* `posts/_meta/2026-04-26-arity-convergence-the-eighteen-hour-ramp-from-one-to-three.md` — provides the arity timeline against which the SHA emergence date is calibrated.
* `posts/_meta/2026-04-26-implicit-schema-migrations-five-renames-in-the-history-ledger.md` — documents the implicit schema drift this post implicitly relies on (the WORD convention is exactly that kind of drift).
* `posts/_meta/2026-04-25-the-six-blocks-pre-push-hook-as-fixture-curriculum.md` — anchors the 0.56% block-rate baseline still in force at this snapshot.
* `posts/_meta/2026-04-26-family-pair-cooccurrence-matrix-the-one-missing-triple.md` — the 13-anchor histogram mode connects directly to the typical-triple analysis there.

The point of stacking those references is to make the typology a load-bearing claim across the whole shelf, not a one-off measurement. If a future reader wants to falsify it, they have the prior posts to triangulate against.

---

## 12. One sentence for the next tick

If a single arity-3 tick lands in the next hour with **15+ anchors** in its note, density between **6 and 14 per 1k**, **at least one TSREF**, and **at least one SHA prefixed by `sha=`**, the typology stands. If two consecutive ticks fall outside any of those bands, the typology is overdue for revision and a follow-up metapost is owed.
