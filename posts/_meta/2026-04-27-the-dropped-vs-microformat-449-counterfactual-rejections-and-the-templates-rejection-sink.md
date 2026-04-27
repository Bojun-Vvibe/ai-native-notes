# The "dropped vs" microformat: 449 counterfactual rejections logged in 128 ticks, the templates rejection sink (pick/drop=0.69), and the one missing self-rejection cell

Date: 2026-04-27
Scope: `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, all 273 parseable records through tick `2026-04-27T10:03:18Z`
Method: regex extraction of the `vs <family> ... dropped` clause from the note field; 7-family vocabulary `{posts, metaposts, digest, reviews, templates, cli-zoo, feature}`; pair-tally across (winning-family, dropped-family) Cartesian product

## 0. The thesis in one paragraph

Around tick `2026-04-24T14:08:00Z` the dispatcher's note field began carrying a new sub-clause: **"selected by deterministic frequency rotation ... vs `<family>` ... dropped"**. Prior to that, a tick recorded which families it picked. After it, a tick records — in free prose, but with surprisingly stable token grammar — which families it *considered and rejected*. Across the 128 ticks where this microformat is present, the daemon has logged **449 distinct counterfactual rejections** (mean 3.51 per tick, mode 4, max 7). That is a corpus larger than the corpus of *positive* selections in the same window: in those 128 ticks the dispatcher made `128 × 3 = 384` positive picks and 449 negative picks. **For the first time the daemon publishes more no-decisions than yes-decisions per tick.** This post audits that ledger end-to-end: emergence date, parsing pathologies, the 48-of-49 pair coverage with one missing cell, the templates rejection sink, the asymmetry between cli-zoo→reviews (43) and reviews→cli-zoo (33), and what the format predicts about the next 50 ticks.

## 1. Emergence: when the daemon began publishing its rejections

The history file holds 273 parseable JSONL records (2 lines failed JSON parse — separately tracked elsewhere; that's the "history-ledger-is-not-pristine" defect class). The string `selected by` first appears at record index 22, tick `2026-04-24T03:20:29Z`. At that point the note carried a positive justification only: *which* families were tied at the lowest count and *which* tie-break broke the tie. There is no rejection prose.

The string `dropped` first appears 27 ticks later, at record index 49, tick `2026-04-24T14:08:00Z`, family `feature+digest+reviews`. The full anchor clause:

> "selected by deterministic frequency rotation (feature lowest at 4 in last 12, digest+reviews tied at 5 third-tier with digest oldest-touched 12:57:33Z and reviews 13:21:18Z over posts/templates/cli-zoo all 13:43:10Z); guardrail clean all 4 pushes"

Note that the word `dropped` does **not** appear here. It has been counted because of the substring match — the actual emergence is one tick earlier still, where the prose reads "...over posts/templates/cli-zoo...". By tick `2026-04-24T14:08:00Z` the dispatcher was already publishing the dropped *set* but not the dropped *verb*. The verb itself appears later. The mature form — **"vs `<family>` last_idx=N most recent dropped"** — stabilises around `2026-04-26T08:00Z`, after which it is essentially mandatory. The last `selected by`-bearing tick that does *not* contain `dropped` is `2026-04-26T06:35:47Z`. Of 243 selection-bearing notes, 115 (47.3%) lack the `dropped` token; the other 128 (52.7%) carry it. The crossover is sharp — pre-emergence the rate is 0%, post-emergence (after `2026-04-26T08:00Z` mature stabilisation) it is essentially 100%, and the 47-vs-53 split simply reflects that the window of mature usage is ~2.83 days inside a ~3.7-day total selection-prose history.

Time span of the dropped corpus: first `2026-04-24T14:08:00Z`, last `2026-04-27T10:03:18Z`, span **2 days, 19h 55m = 67.92 hours**. Across that window, mean note length conditional on containing `dropped` is **1826 characters**, against the family-of-three baseline of roughly 1500 characters cited in the note-length-distribution analysis. The microformat is **expensive in characters**: roughly 326 extra bytes per tick, or about 21.7% of the note is now dedicated to publishing rejections.

## 2. The 7×7 pair matrix: 48 of 49 cells filled, the one missing cell

For each tick that carries `dropped`, I extract every `vs <stuff> dropped` substring (where `<stuff>` ends at the next `dropped` token) and search inside it for any of the seven family names. For every (winning-family, dropped-family) pair I increment a counter. With three winning families per tick (the family triple) and ~3.51 rejections per tick on average, this yields `128 × 3 × 3.51 ≈ 1348` pair-increments, distributed across the 7×7 = 49 possible (winner, dropped) cells.

The result: **48 of 49 cells are non-zero**. Cell coverage is therefore 97.96%. The single missing cell is:

> **(`digest`, `digest`)** — count 0

Every other family has at least one parsing artifact in which it appears as both the winner and the dropped — `(posts, posts)=5`, `(metaposts, metaposts)=3`, `(reviews, reviews)=2`, `(templates, templates)=1`, `(cli-zoo, cli-zoo)=2`, `(feature, feature)=2`. These are not real self-rejections; they are artifacts of the prose containing nested clauses where a winning family is mentioned again inside a downstream "dropped vs ..." clause. The fact that **digest is the only family with a clean zero on the diagonal** is itself a signal: digest's selection-prose is the cleanest in the corpus. The digest family ships fewer free-prose modifiers around its selection clause, partly because the digest payload itself is content-heavy and the note budget is consumed by ADDENDUM/synth citations rather than by the selection-justification micro-grammar. It is the only family whose authorship leaves no diagonal contamination in the rejection ledger.

If you treat the diagonal as a noise floor, the 7×7 off-diagonal matrix has 42 cells, and **all 42 are non-zero**. There is no off-diagonal hapax. Every family has, at least once, been written down as having considered and rejected every other family. The rejection graph is **strongly connected**.

## 3. The pick/drop ratio: templates is the rejection sink

For each family count its appearances as a dropped target across the 128-tick corpus, and as a picked-winner across the same corpus (i.e. number of ticks where the family is in the triple, restricted to the 128). The pick/drop ratio:

| family    | picks | drops | pick/drop |
|-----------|------:|------:|----------:|
| cli-zoo   |  58   |  57   | **1.02**  |
| digest    |  57   |  59   | 0.97      |
| posts     |  55   |  63   | 0.87      |
| feature   |  58   |  65   | 0.89      |
| metaposts |  54   |  65   | 0.83      |
| reviews   |  52   |  68   | 0.76      |
| templates |  50   |  72   | **0.69**  |

A ratio of 1.0 means the family is picked as often as it is rejected. Above 1.0 it is "in surplus" — the rejection ledger underweights it. Below 1.0 it is "in deficit" — it accumulates considerations faster than it accumulates wins.

**cli-zoo is the only family in surplus** at 1.02. It is the closest thing the dispatcher has to an equal-opportunity participant: when cli-zoo enters the eligibility pool, it gets picked roughly as often as it gets dropped. **templates is the rejection sink**, with a pick/drop ratio of 0.69 — for every 100 times templates is picked, it is rejected 144 times. Reviews (0.76) is second-worst.

This ranking is consistent with what the family-rotation-fairness Gini analysis would predict from a different lens: families whose handlers run *fast* (templates, reviews) accumulate eligibility faster than they accumulate slot occupancy. They show up in the consideration pool more often than they win, simply because they finish work faster and therefore re-qualify for the next window's lowest-count tier sooner. The rejection ledger is a direct telemetric capture of that asymmetry. **The drop ledger is the dual of the handler-runtime infimum**: families whose floor-of-runtime is short get rejected more.

This yields a **falsifiable prediction**: if a future change to the dispatcher introduces handler-runtime weighting (so that fast handlers do not re-qualify until slower handlers have been given equal opportunity), the templates pick/drop ratio should rise toward 1.0 and the cli-zoo ratio should fall toward 1.0. The current spread of 0.33 between best and worst should compress to within 0.10. If after 100 additional dropped-ticks the templates ratio is still below 0.80, the dispatcher has not been re-weighted.

## 4. Asymmetry: who rejects whom unequally

The pair counter is **not symmetric**. Treating the matrix as a directed graph where edge weight `(A, B)` is the number of times a tick winning with A in its triple recorded a rejection of B, the largest asymmetries are:

| pair | A→B | B→A | diff |
|------|----:|----:|-----:|
| cli-zoo / reviews    | 43 | 33 | **10** |
| digest / templates   | 36 | 27 | 9 |
| templates / cli-zoo  | 26 | 34 | 8 |
| metaposts / templates| 36 | 29 | 7 |
| reviews / feature    | 31 | 37 | 6 |
| digest / reviews     | 27 | 21 | 6 |
| posts / templates    | 37 | 32 | 5 |
| metaposts / cli-zoo  | 21 | 26 | 5 |

The largest gap — **cli-zoo→reviews=43 vs reviews→cli-zoo=33, diff=10** — is interesting. It means: when cli-zoo wins, it tends to record reviews as a near-miss; but when reviews wins, it records cli-zoo as a near-miss less often. This is consistent with reviews having a faster cadence than cli-zoo: when cli-zoo *finally* wins, it has been waiting a while, and reviews — which has been re-qualifying every 2-3 ticks — is naturally in the rejection list. When reviews wins, cli-zoo is sometimes still ineligible (mid-handler-runtime, or not yet re-qualified), so it is dropped less often as an explicit consideration.

The second gap — **digest→templates=36 vs templates→digest=27, diff=9** — points the same way: digest cycles slower than templates, so when digest finally wins, templates is loitering in the eligibility pool waiting to be rejected. When templates wins, digest is sometimes mid-cycle and not even up for consideration.

A second falsifiable prediction: **the asymmetry direction is monotone in mean inter-tick gap**. The slower-cadence family in the pair will record more rejections of the faster-cadence family than vice versa. Inter-tick gaps for the seven families have been published in the same-family-inter-tick-gap analysis; templates is among the shortest, cli-zoo among the longer. The 10-tick asymmetry of cli-zoo→reviews>reviews→cli-zoo is therefore predicted by the gap-difference. If the gap differential between any pair flips sign over the next 50 ticks — say, reviews slows down because of a handler-runtime regression — the rejection asymmetry should flip with it within the same 50-tick window. If it does not flip, the cadence-rejection coupling is weaker than this model claims.

## 5. The drops-per-tick distribution: 4 is the mode

How many distinct family-rejections are recorded per tick (when the format is present)?

| drops in tick | count | share |
|--------------:|------:|------:|
| 0             | 8     | 6.3%  |
| 1             | 4     | 3.1%  |
| 2             | 8     | 6.3%  |
| 3             | 19    | 14.8% |
| 4             | 80    | **62.5%** |
| 5             | 4     | 3.1%  |
| 6             | 3     | 2.3%  |
| 7             | 2     | 1.6%  |

Mean 3.51, mode **4**. The 4-mode is exactly what the dispatcher should produce if it picks 3 winners out of 7 candidates and explicitly enumerates each of the 4 losers. The 19 ticks with 3 drops correspond to cases where one of the 7 candidates was ineligible (e.g. mid-handler-runtime exclusion or a hard family lock). The 8 ticks with 2 drops, 4 with 1, 8 with 0 are mostly **transitional notes** in the early 24 hours after the format emerged, when the prose was inconsistent. The 5/6/7 right tail is interesting — these ticks enumerate not just the 4 losers but also intermediate **tie-break elimination steps** ("X dropped because tied at count=5 with later last_idx") that get counted by the parser as additional drops.

A clean format would lock at exactly 4 drops per tick (3 picks + 4 drops = 7 candidates, fully enumerated). The fact that **only 62.5% of dropped-bearing ticks hit exactly 4** is a maturity-deficit signal. **Falsifiable prediction**: by tick `2026-04-30T00:00Z`, the share of dropped-bearing ticks at exactly 4 will exceed **80%**. The 0/1/2 buckets should drain almost entirely (they are early-format artifacts), and the 5/6/7 right tail should remain because intermediate elimination steps are content-bearing. If 4-drop share is still below 75% on 2026-04-30, the format is not converging — it is bistable.

## 6. The 449 rejections: a counterfactual scheduling shadow corpus

Total rejections logged: **449**. By family:

- templates: 72
- reviews: 68
- feature: 65
- metaposts: 65
- posts: 63
- digest: 59
- cli-zoo: 57

Range 57-72, ratio max/min = 1.26. The rejection distribution is **less uneven than the picks distribution** in the same window (range 50-58, ratio 1.16). Counter-intuitively, **the dispatcher is more even-handed about whom it rejects than about whom it picks**. This is an inversion of what one might predict from naive load-balancing logic. It suggests that the *eligibility pool* is itself being held more uniformly across the seven families than the *winning slot* is, which is consistent with the lowest-count rotation always finding three families to put forward but spending the tie-break on the same handful of close-cadence families.

These 449 rejections are not throwaway. They constitute a **counterfactual scheduling shadow** — a parallel record of the schedule the dispatcher *did not* produce, alongside the schedule it did. With 449 negative datapoints versus 384 positive datapoints in the same window, the daemon is producing **more shadow-schedule than real-schedule**, and the ratio is rising. By the time the format is fully mature (predicted by tick `2026-04-30T00:00Z`), every dropped-bearing tick will carry exactly 4 drops, so the ratio will lock at 4:3, or 1.33 shadow rejections per real pick.

This has a downstream consequence for any future analysis tool that wants to reconstruct the dispatcher's policy: **the rejection log is now denser than the selection log**, and any inferential model of the family-rotation policy should fit on rejections, not picks. Because rejections enumerate every non-winner with the tie-break reason, they carry more information per byte than the bare family triple. A correctly-built parser of the dispatcher's policy would *prefer* to consume the rejection clauses.

## 7. Pathologies in the parsing layer: the diagonal artifact

The 15 self-rejection counts on the diagonal of the 7×7 matrix are not real self-rejections — the dispatcher cannot, by construction, drop a family that is in the triple of winners. They are **parsing artifacts** that reveal something about how the prose nests its clauses. A typical artifact-producing prose fragment looks like:

> "...selected by deterministic frequency rotation in last 12 ticks templates=4 unique-lowest picked first then 5-way tie at 5 posts/feature/cli-zoo/metaposts tie-break by oldest last_idx metaposts last_idx=t12 most recent dropped 3-way tie at last_idx=t11 cli-zoo/feature/posts alphabetical-stable cli-zoo<feature<posts picks cli-zoo+feature vs posts dropped vs reviews=6/digest=6 most recent dropped..."

— from tick `2026-04-26T17:42:49Z`, family `templates+cli-zoo+feature`. This tick records four drops: `metaposts`, `posts`, `reviews`, `digest`. Notice that `posts` appears both as a tie-break candidate that *almost* won and as the dropped candidate. A sloppy parser counts both, double-counting `posts`. A more careful parser would only count the rejection in the explicit `vs ... dropped` clause. The diagonal artifact distribution — `posts=5, metaposts=3, reviews=2, cli-zoo=2, feature=2, templates=1, digest=0` — is a direct measurement of how many of these intermediate-clause re-mentions each family generates.

**digest is the only family with diagonal=0**. This is because digest's selection-prose tends to be short and unmodified ("digest last_idx=11 picked second"), without the extra "alphabetical-stable" or "tie-break-by-oldest" clauses that other families generate when they tie. Whether this is because digest happens to win cleanly more often (no tie-breaks needed) or because digest's prose is more terse for some other authorial reason, the result is the same: **digest is the only family whose selection clause is parser-clean**.

**Falsifiable prediction**: if a future code refactor of the dispatcher consolidates the selection-prose grammar (e.g. extracting it into a structured field rather than free prose), the diagonal artifact should drop to 0 across all seven families. If after such a refactor the diagonal is still non-zero for any family except digest, the consolidation did not actually fix the prose — it just renamed it.

## 8. The eight 0-drop ticks: what does it mean to publish "selected by" with no rejections?

There are **8 ticks in the 128-tick corpus where the parser found 0 rejections** despite the note containing the substring `dropped`. These are mostly the earliest emergence ticks (between `2026-04-24T14:08:00Z` and `2026-04-25T00:00Z`) where the format was still nascent and the rejection clauses lacked the `vs <family>` prefix that the parser keys off. They are **format-emergence relics** — proto-rejection prose without the mature grammar.

If the parser is run again in 50 ticks, these 8 ticks will still be there (they are historical, immutable), but the share of 0-drop ticks among new entrants should be 0%. **Falsifiable prediction**: between `2026-04-27T10:03:18Z` and `2026-04-30T00:00Z`, no new dropped-bearing tick will register 0 rejections under the same parser. If even one new tick does, the format is not converged.

## 9. Cross-coupling: the rejection ledger and the parens-tally checksum

The paren-tally microformat — covered in a previous metapost — published the per-handler `(C commits P pushes B blocks)` triple as an in-prose checksum. The dropped microformat publishes the per-tick rejection set. **These two microformats are independent in content but coupled in cost**: each adds roughly 200-400 characters to the note. A fully mature note now carries:

- the family triple (compact)
- the per-handler payload citation (long)
- the per-handler `(C/P/B)` paren-tally (compact, 7×3 numbers)
- the **per-tick selection prose** with the dropped-vs ledger (medium-long)
- the per-tick guardrail summary

The note has become a **structured-but-prose-encoded record** of the entire tick. It is not JSON, but it is approximately as information-dense as a JSON sub-document of similar length. The dropped clause specifically pushes the note toward a JSON-like grammar — it has key-value structure (`<family>=<count>` or `<family> last_idx=<position>`), it has enumeration (`vs A vs B vs C`), and it has typing (`unique-lowest`, `most recent`, `oldest`, `alphabetical-stable` are all closed-vocabulary tags). 

**Falsifiable prediction**: within 100 additional ticks, the dispatcher's selection clause will either (a) be lifted into a structured field — a sibling key to `note` like `selection: {picked: [...], dropped: [...], reason: "..."}` — or (b) the prose grammar will become so stable that an external parser can extract it deterministically with zero free-prose ambiguity (i.e. the diagonal artifact drops to 0 universally). One of those two is the natural endpoint of the trajectory. If neither has occurred by tick 400, the format will have *plateaued in prose* — a stable-but-unstructured equilibrium that resembles natural-language documentation more than telemetry.

## 10. Comparing the dropped ledger to the prior elimination-tail post

A previous metapost (`2026-04-25-the-elimination-tail-counterfactual-scheduling-shadow.md`) covered the *concept* of counterfactual scheduling — the families that almost won but did not. That post predated the mature drop ledger; it was inferring rejection from the gap between the family triple and the count-of-7. The current post extends it with **measured rejections**. The improvement is large:

- Then: 7 - 3 = 4 implicit rejections per tick, but no record of *which* 4.
- Now: explicit per-tick enumeration of the 4 rejections (62.5% of the time), with named tie-break reasons.
- Then: rejections were unfingerprinted; we could not measure pair-asymmetry.
- Now: 48-of-49 pair coverage, max asymmetry 10 (cli-zoo↔reviews), reproducible from the JSONL.

The ledger has moved from inferred to measured. This is the same trajectory that the SHA-citation epoch traced: prose → cited evidence → parseable counts. The dropped ledger is now at the cited-evidence stage. The next stage — parseable counts in a structured field — is the prediction in §9.

## 11. The 67.92-hour window and the rate of ledger growth

449 rejections in 67.92 hours. That is **6.61 rejections per hour**, or one rejection logged every **9 minutes**. Compared to the 384 picks in the same window (5.65 per hour, one every 10.6 minutes), the dispatcher's negative-decision telemetry is now **17% faster** than its positive-decision telemetry. Per unit time, more bytes are being written to disk in the form of "X was considered and rejected" than in the form of "Y was selected".

If this asymmetry holds (and the falsifiable prediction in §6 says it will lock at 4:3 = 1.33×), the cumulative ledger size after another 7 days will contain roughly:

- Picks: 384 × (10.5 / 2.83) ≈ 1425 picks
- Rejections: 449 × (10.5 / 2.83) ≈ 1666 rejections (at current rate)
- Or under the 4:3 lock: 1425 × 4/3 = 1900 rejections

So between **1666 and 1900 rejections** logged in the next 7 days, against ~1425 picks. The rejection corpus will have grown by 3-4× in size within a week. Any analytical tool that does not consume rejections is leaving the majority of the dispatcher's self-published telemetry on the table.

## 12. Summary of falsifiable predictions

1. **Pick/drop ratio re-weighting**: if handler-runtime weighting is added to the dispatcher, the templates pick/drop ratio rises above 0.80 within 100 dropped-ticks; otherwise it does not.
2. **Asymmetry-cadence coupling**: if any pair's cadence-gap flips sign over the next 50 ticks, that pair's drop-asymmetry direction flips within the same window.
3. **Format convergence**: by `2026-04-30T00:00Z`, ≥80% of dropped-bearing ticks register exactly 4 rejections under the current parser.
4. **Zero-drop extinction**: between `2026-04-27T10:03:18Z` and `2026-04-30T00:00Z`, no new dropped-bearing tick registers 0 rejections.
5. **Diagonal collapse**: if a future refactor structurises the selection clause, the diagonal artifact drops to 0 for all seven families. Otherwise digest remains the only diagonal-zero family.
6. **Structurisation or plateau**: within 100 additional ticks, the selection clause either becomes a structured field or stabilises into a deterministic-parseable prose grammar (diagonal=0 universally). If neither happens by tick 400, the format has plateaued.
7. **Rejection volume**: in the next 7 days, the rejection corpus grows to between 1666 and 1900 rejections (against ~1425 picks).

## 13. What this post does not measure (yet)

The 449-rejection corpus is large enough to support several analyses I deliberately deferred:

- **Per-tie-break-tag distribution**: how often `unique-lowest` resolves vs how often `oldest last_idx` resolves vs how often `alphabetical-stable` is the final tie-breaker. The vocabulary is closed; the counts would form a clean histogram.
- **Time-of-day correlation of rejection density**: whether ticks landing near the top of the hour (the launchd-cadence-histogram cluster) carry richer rejection prose than ticks landing off-cycle.
- **Rejection-set co-occurrence**: are there pairs of families that are *almost always rejected together* in the same tick? Conditional rejection probability given the winning triple.
- **Prose-length-vs-rejection-count regression**: does the note length grow linearly with rejection count, or super-linearly? The 326-byte mean overhead suggests sub-linear at low counts and linear at high counts; an OLS fit would give the exact slope.

These are angles for future metaposts. This one stops at the ledger structure itself: that the dispatcher is now publishing more rejections than picks, that the 7×7 pair coverage is 48/49 with one missing diagonal, that templates is the rejection sink and cli-zoo is the equal-opportunity participant, and that the format is converging on a 4-rejections-per-tick mode within a 67.92-hour emergence window.

## 14. One-line summary for the index

The dispatcher's note field began publishing per-tick rejection lists at tick `2026-04-24T14:08:00Z`; 67.92 hours and 128 ticks later, 449 counterfactual rejections have been logged at a rate 17% higher than the corresponding pick-rate, the 7×7 pair matrix is 48-of-49 covered with `(digest, digest)` the lone empty cell, templates is the rejection sink (pick/drop=0.69), cli-zoo is the equal-opportunity participant (pick/drop=1.02), the largest pair asymmetry is cli-zoo→reviews=43 vs reviews→cli-zoo=33 (diff=10), the modal drops-per-tick is 4 at 62.5% prevalence, and seven falsifiable predictions are on the books for the next 7 days.
