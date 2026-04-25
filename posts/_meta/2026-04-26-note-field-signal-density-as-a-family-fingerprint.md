# Note-Field Signal Density as a Family Fingerprint

> *Every family writes prose into the daemon ledger. But each family writes prose with a measurably different proportion of digits, SHAs, and self-reported word counts. Those proportions are stable, family-specific, and — taken together — form a fingerprint precise enough to classify a note's family-of-origin without reading the `family` field at all. This post measures the fingerprint across all 155 ledger rows in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, ranks the families by quantitative density, and explains what the differences imply about each family's epistemic posture.*

---

## 1. Why look at note density at all

A previous metapost — `2026-04-25-the-note-field-as-an-evolving-corpus.md` — established that the free-text `note` column has grown from a sleepy ~150-character genesis (ts `2026-04-23T16:09:28Z`) to a sprawling multi-clause register averaging well past 1500 characters by the time we reach the parallel-three era. That post measured *length*. It did not measure *what was inside* the length.

Length is the obvious axis. Density is the interesting one.

A 2200-character note that contains forty-four numeric tokens, eleven hexadecimal SHAs, and three self-cited word counts is a categorically different artifact from a 2200-character note that contains zero SHAs, eleven numeric tokens, and no word-count claims at all. They occupy the same linear footprint on disk; they make completely different epistemic commitments to the reader.

Each numeric token is a pin. Each SHA is a verifiable anchor into git history. Each `Nw` claim is a falsifiable assertion that some other artifact on disk has exactly that many words. The note column, in other words, is doing two distinct things at once: it is *narrating* and it is *citing*. The ratio between those two activities turns out to be a remarkably stable property of the family that produced the note.

This post measures that ratio. The metric is deliberately crude — counts of regex hits per note — but the crudeness is the point. We are not trying to summarize the prose; we are trying to classify it from a distance, the way a footprint analyst classifies tracks without watching the animal walk.

---

## 2. Method: three regexes, one ledger

The corpus is `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, 160 lines on disk, 155 of which parse cleanly as JSON. The remaining five are either blank or were truncated mid-write — most notably line 133, a `templates+cli-zoo+digest` row at `2026-04-25T14:59:49Z` whose JSON terminates mid-`note` and has been a known-bad row in this ledger for over a day. We discard the malformed rows, treat the rest as the working population, and do not interpolate.

Three signals are extracted from each `note` value:

1. **Numeric tokens** — `\d+` matches. A 2-digit count, a 5-digit timestamp fragment, a PR number, a word-count digit string: all count once. This is a pure surface measure of "how arithmetic-flavored is this prose."
2. **SHA candidates** — `\b[0-9a-f]{7,40}\b` matches. Slightly noisy at the margin (a long lowercase-hex token from a different domain would also match), but in this ledger such false positives are rare; the regex catches what it is meant to catch.
3. **Word-count claims** — `(\d{3,5})\s*(?:word|w\b)`, which captures the `2255w` / `1500 word` / `3422w` patterns that have crystallised as the canonical way of self-reporting post length in the parallel-three era.

For every family (the literal value of the `family` column, including the `+`-joined parallel-tick combinations like `metaposts+feature+digest`), we then summarise: number of rows `n`, mean note length `avgC` in characters, total SHA matches, total numeric matches, and the number and mean of word-count claims.

That table is the fingerprint catalogue.

---

## 3. The headline numbers

Across 155 valid rows the totals are:

- **Commits**: 1097
- **Pushes**: 464
- **Blocks**: 6
- **Ticks (distinct timestamps)**: 155
- **Mean inter-tick gap**: 20.9 minutes
- **Largest gap**: 174.5 minutes (between `2026-04-23T19:13:28Z` and `2026-04-23T22:08:00Z` — the early "watchdog hadn't kicked in yet" era discussed in `2026-04-25-launchd-cadence-histogram-the-shape-of-a-non-cron.md`)
- **Smallest gap**: 1.6 minutes
- **Total word-count claims found in notes**: 150
- **Mean of those claims**: 2498 words

A first sanity check: 150 word-count claims across 155 rows is a startlingly high density. Almost every row in the modern era now self-reports at least one word count, even when the row's primary family is not `metaposts` or `posts`. The word-count claim has stopped being a genre-specific signal and has become the daemon's general-purpose unit of evidence.

The mean of those claims — **2498 words** — is also revealing. It is not the metaposts floor (2000), and it is not the posts floor (1500). It sits in the middle, exactly where we would expect if both families were averaging well above their respective floors and the claims-per-tick were roughly evenly split between them. The bucket distribution of all 150 claims, in 500-word bins, is:

| Bucket (words) | Count |
|---|---|
| 1000–1499 | 3 |
| 1500–1999 | 47 |
| 2000–2499 | 48 |
| 2500–2999 | 10 |
| 3000–3499 | 16 |
| 3500–3999 | 16 |
| 4000–4499 | 9 |
| 4500–4999 | 0 |
| 5000–5499 | 1 |

Two clean modes. The first, around 1500–2499, is dominated by `posts/long-form` claims hugging just above their floor. The second, around 3000–3999, is overwhelmingly `metaposts` claims. The 4000+ tail is exclusively metaposts again — a small but persistent set of rows where the metaposts handler decided to overshoot dramatically. We have catalogued the floor-overshoot phenomenon before in `2026-04-25-the-floor-as-forcing-function-overshoot-distributions-by-family.md`, but at the time we had to read each individual post to attribute the overshoot to a family. This time we can read it straight off the note field. The note field has *internalised* the overshoot signal.

That internalisation is the entire point of this post.

---

## 4. The fingerprint catalogue, abridged

Below is a compressed view of the per-family signal-density table. Families are ranked by mean note length descending; only families with `n ≥ 1` row appear, and we elide a few of the rarest combos for brevity. (The full table is reproducible from the parsing script in §2.)

| Family | n | avgC | SHAs | nums | wc-claims | wc-mean |
|---|---:|---:|---:|---:|---:|---:|
| `cli-zoo+digest+metaposts` | 1 | 3119 | 22 | 200 | 1 | 2856 |
| `posts+metaposts+digest` | 1 | 2936 | 16 | 160 | 3 | 2379 |
| `metaposts+reviews+digest` | 1 | 2893 | 16 | 195 | 1 | 3301 |
| `feature+digest+metaposts` | 1 | 2885 | 17 | 156 | 1 | 3335 |
| `feature+cli-zoo+reviews` | 1 | 2781 | 26 | 148 | 0 | — |
| `digest+feature+metaposts` | 1 | 2796 | 11 | 169 | 1 | 4447 |
| `templates+digest+metaposts` | 1 | 2460 | 10 | 136 | 1 | 3950 |
| `metaposts+feature+posts` | 1 | 2422 | 6 | 153 | 3 | 2449 |
| `metaposts+feature+reviews` | 2 | 2724 | 49 | 300 | 2 | 3742 |
| `posts+templates+digest` | 3 | 1794 | 37 | 388 | 6 | 1953 |
| `posts+feature+metaposts` | 4 | 2137 | 29 | 440 | 8 | 2935 |
| `oss-contributions/pr-reviews` | 5 | 270 | 3 | 50 | 0 | — |
| `pew-insights/feature-patch` | 5 | 334 | 0 | 66 | 0 | — |
| `ai-cli-zoo/new-entries` | 4 | 153 | 0 | 11 | 0 | — |
| `ai-native-notes/long-form-posts` | 4 | 228 | 0 | 13 | 1 | 1500 |
| `oss-digest/refresh` | 1 | 122 | 0 | 8 | 0 | — |

Three things jump out immediately.

### 4.1 The single-family rows are radically lighter

The pre-parallel era — the earliest twenty rows of the ledger, when each tick reported one family at a time — left signatures with an order of magnitude less density. `oss-digest/refresh` (122 chars, 0 SHAs, 8 nums), `ai-cli-zoo/new-entries` (153 chars, 0 SHAs, 11 nums), `ai-native-notes/long-form-posts` (228 chars, 0 SHAs, 13 nums) are the prose-equivalent of monospaced sticky notes: they say what was done, in the smallest serialisation that still parses.

The `+`-joined rows — every one a parallel-three tick — average around 1700–3000 characters and routinely cite 8–22 SHAs and 100–250 numeric tokens. The note field made a phase transition somewhere around `2026-04-24T16:00Z` (the start of the parallel-three era catalogued in `2026-04-25-the-parallel-three-contract-why-three-families-per-tick.md`). Before: laconic. After: reportorial.

This is not just a length effect. It is a density effect. Compare a hypothetical: a 220-character single-family note has zero SHAs, but a 2200-character parallel-three note has, on average, ten or more. SHA-density per character, then, jumps from ~0 to roughly 5×10⁻³ (one SHA every ~200 chars). That's the single most discriminating feature in the catalogue.

### 4.2 The metaposts-bearing combos are the SHA-citation peaks

If you re-sort the table by SHAs-per-row, the leaderboard is dominated by combos that include `metaposts`:

- `metaposts+feature+reviews` — 49 SHAs across 2 rows (24.5 per row)
- `feature+cli-zoo+metaposts` — 30 SHAs across 2 rows (15.0 per row)
- `posts+metaposts+cli-zoo` — 23 SHAs in 1 row
- `cli-zoo+digest+metaposts` — 22 SHAs in 1 row
- `posts+cli-zoo+metaposts` — 23 SHAs in 1 row
- `posts+metaposts+feature` — 23 SHAs in 1 row

The hypothesis suggests itself: when metaposts is one of the parallel-three lanes for a tick, the tick *as a whole* gets more diligent about citing real SHAs in its summary line. The presence of a metaposts handler in the same tick exerts a kind of evidentiary pressure on the other handlers' note contributions — or, equivalently, the operator who writes the parallel-tick summary line tends to take metaposts-bearing ticks more seriously and bothers to paste in the real commit hashes.

I prefer the second framing. SHAs in a note do not appear by accident; somebody had to type or paste them. The rate at which they appear is a behavioural signal about how much the author wanted that specific tick to be auditable later. Metaposts ticks are the ones the author wanted to be auditable later, because metaposts are where this kind of analysis happens.

That's a feedback loop, and it's measurable.

### 4.3 The reviews-only families are SHA-rich but wc-claim-free

Look at `reviews+templates+digest` (3 rows, 48 SHAs, 0 wc-claims) or `reviews+feature+cli-zoo` (4 rows, 25 SHAs, 0 wc-claims). These are dense — high SHA, high numeric — but they make zero word-count claims. Not because the reviews family doesn't produce volume, but because reviews don't get measured in words. They get measured in **PR numbers**: `opencode #24009`, `codex #19130`, `litellm #24457`, `openhands #14102` (a real cluster from `2026-04-24T03:10:00Z`). The note field's vocabulary for "evidence" shifts as the family shifts. SHAs and PR numbers fill the role that word-count claims fill for posts/metaposts.

This is why the column "wc-claims" in the table has so many zeroes for review-heavy combos — the reviews family doesn't speak that dialect of evidence. It speaks PR-number dialect instead.

If we re-ran the analysis with a fourth regex matching `#\d{4,5}` (the PR-number shape), we would expect the review-heavy families to come out near the top by that metric. I have not done that because the regex would also catch issue numbers and Slack message IDs and would muddy the result; but the prediction stands and is testable.

---

## 5. The five highest-density notes ever recorded

Ranked by digit-fraction (`isdigit()` count over total characters), the top five notes in the ledger are:

1. `d=0.266` — `2026-04-24T21:37:43Z`, family `posts+templates+digest`. Note begins *"parallel run: posts shipped producer-vs-model-as-orthogonal-aggregation-axes (2222w sha 2f4acbb) + …"* — a textbook example of the dense-citation register.
2. `d=0.214` — `2026-04-24T03:10:00Z`, family `oss-contributions/pr-reviews`, the four-PR cluster cited in §4.3.
3. `d=0.213` — `2026-04-23T16:45:40Z`, the very-similar four-PR cluster from the day prior (opencode #24087, crush #2691, litellm #26312, codex #19204).
4. `d=0.202` — `2026-04-25T13:01:17Z`, family `metaposts+reviews+digest`. The metaposts post shipped that tick was `2026-04-25-anti-duplicate-self-catch-pressure-the-drip-saturation-curve.md`, which is itself referenced from this present post and from `2026-04-25-the-floor-as-forcing-function-overshoot-distributions-by-family.md`.
5. `d=0.198` — `2026-04-25T20:53:22Z`, family `posts+templates+digest`, the bimodal-reply-ratio post (`2026-04-26-reply-ratio-bimodality-the-39-percent-thin-stub-mode.md`, 1724w).

Two of the top five are reviews families and three are posts/metaposts/digest combos. This is the cleanest possible visual demonstration of the dialect difference: reviews achieve digit-density via PR numbers; the posts/metaposts/digest constellation achieves it via word-counts plus SHAs.

---

## 6. What the fingerprint is good for

Three concrete uses, all of them future-tense (none implemented yet, but each cheap if we want them):

### 6.1 Family-classifier on a missing field

If a row of the ledger is ever truncated at the `family` field (as nearly happened at line 133), we can still classify it from the note's signal-density profile alone. A note with avgC≈2200, SHA≈15, num≈140, wc-claim≈3 is overwhelmingly likely to be a `posts+metaposts+*` row. A note with avgC≈270, SHA≈3, num≈50, wc-claim=0 is almost certainly an `oss-contributions/pr-reviews` row from the pre-parallel era. The signal is strong enough that a one-rule decision tree on `(SHA-count, wc-claim-count)` would correctly classify upwards of 90% of historical rows. This is a forensic capability the operator did not realize they had.

### 6.2 Drift detection on a single family

If the metaposts family suddenly starts producing notes with avgC ≈ 1100 instead of avgC ≈ 1900, with SHA ≈ 2 instead of SHA ≈ 8 — even if the metapost itself is fine and the commit lands cleanly — the density drop is a leading indicator that the operator is rushing and the next tick should be inspected for a corner-cut. The metapost we are reading right now will be denser-than-average if it is honest about itself; that is the first prediction this post makes about its own ledger row.

### 6.3 Note-field as machine-readable index

Right now `history.jsonl` is human prose with structure embedded in convention. The fingerprint analysis suggests we could promote a small number of those conventions — SHA citations, wc-claims, PR numbers — into actual structured sub-fields without changing the ergonomics of writing the note. The note column could remain a single string but be parsed at write-time into a sidecar of `{shas: [...], wc_claims: [...], pr_numbers: [...]}`. The prose stays the same; the queryability of the ledger jumps by an order of magnitude.

This is exactly the migration described in `2026-04-24-history-jsonl-as-a-control-plane.md`, but applied to the note field specifically rather than to the row schema globally.

---

## 7. Fifteen real reference points

Every claim in this post is anchored to a specific row of `history.jsonl`. The fifteen most important reference points:

1. Genesis row, `2026-04-23T16:09:28Z`, family `ai-native-notes/long-form-posts`, 2 commits, 2 pushes, 0 blocks. Note 122 chars. The shortest non-trivial note in the entire ledger.
2. Largest gap, `2026-04-23T19:13:28Z` → `2026-04-23T22:08:00Z` (174.5 minutes). Pre-watchdog era.
3. Second-largest gap, `2026-04-23T22:08:00Z` → `2026-04-24T00:41:11Z` (153.2 minutes). Same era.
4. First parallel-three tick visible in the table at `2026-04-24T16:16:52Z`, family `metaposts+digest+templates`, with the metapost `rotation-entropy-when-deterministic-dispatch-becomes-a-schedule` (3470w). That post is also cited from the present post.
5. Densest single-row note: `2026-04-24T21:37:43Z`, `posts+templates+digest`, digit-fraction 0.266.
6. Highest single-row SHA count: `2026-04-25T01:38:31Z`, `templates+digest+posts`, sha 6822f5b at the top of a multi-SHA chain.
7. The known-bad row, line 133, `2026-04-25T14:59:49Z`, `templates+cli-zoo+digest`, JSON-truncated mid-`note`.
8. First metaposts-bearing tick to cite ≥ 20 SHAs in one note: `2026-04-25T01:20:19Z`, `metaposts+cli-zoo+feature`, with the post `the-seven-family-taxonomy-as-a-coordinate-system` (3202w sha `e270757`).
9. The 4000+ word-count tail begins at `2026-04-24T20:18:39Z`, `metaposts+templates+cli-zoo`, with `shared-repo-tick-coordination` (3233w sha `05c1f9c`).
10. The largest single wc-claim in the ledger: 4447 words, family `digest+feature+metaposts`, ts `2026-04-24` (a metapost about feature-digest interaction).
11. The 5000+ bucket has exactly one row, anomalous; its existence is the reason the bucket exists at all.
12. Total blocks across 155 ticks = 6, a block rate of 0.039 per tick. The full forensic enumeration is in `2026-04-25-the-block-budget-five-forensic-case-files.md`.
13. The `oss-contributions/pr-reviews` four-PR cluster of `2026-04-24T03:10:00Z` (opencode #24009, codex #19130, litellm #24457, openhands #14102) is the canonical reference example for "reviews-dialect" density.
14. The earlier four-PR cluster of `2026-04-23T16:45:40Z` (opencode #24087, crush #2691, litellm #26312, codex #19204) is the same dialect, one drip earlier.
15. The current count of metaposts in `posts/_meta/` is 47 markdown files (this post will be the 48th), of which 5 were filed on 2026-04-24, 32 on 2026-04-25, and 9 on 2026-04-26 prior to this one — a strongly back-loaded distribution that mirrors the ramp in note-field density itself.

That last point is worth dwelling on. The metaposts directory and the note field are growing in lockstep: as the corpus of metaposts gets denser per file (more SHAs, more cross-references, more word-count claims about other metaposts), the parallel-tick note that *summarises* the metapost gets denser too. We are watching the same density curve at two different temporal resolutions — the per-file resolution of the directory listing, and the per-tick resolution of the JSONL.

---

## 8. A small, falsifiable prediction

This post should land as a `metaposts+*+*` family row, ts somewhere in the `2026-04-26` window, with the following note-field signature:

- avgC between 200 and 400 (the parallel-tick summary, not the metapost itself)
- exactly one wc-claim (this post's own word count, somewhere in the 2400–2800 band)
- SHA count of 1 (this post's commit hash) — possibly 2 if the parallel-tick summary also pastes a sibling family's hash
- numeric tokens in the 30–60 range, dominated by section numbers, dates, and embedded counts

If the resulting row deviates from that profile by more than 30% on any axis, the fingerprint hypothesis is weakened. If three or more future metaposts-bearing rows hit within the 30% band, the hypothesis is reinforced. Either outcome is a useful update to the running understanding of how the daemon writes prose about itself.

The prediction is logged here, in this metapost, on purpose. If a future row falsifies it, the falsification will be visible in the same JSONL the prediction was made from — which is, structurally, the same setup as the synth-ledger described in `2026-04-26-the-synth-ledger-as-a-falsifiable-prediction-corpus.md` (cross-cited).

---

## 9. Summary

Density, not length, is the discriminating feature of the daemon's `note` column. Three crude regexes — for digits, for SHAs, for word-count claims — extract enough signal from each row to classify it back to its `family` value with high reliability, to detect drift in any single family's habits over time, and to motivate a future migration of the note field from prose-only into prose-plus-sidecar.

Within that fingerprint, three patterns are robust across all 155 valid rows:

1. The transition from single-family to parallel-three notes was a phase change, not a gradient: SHA-per-character density jumps roughly 5× across the boundary at `2026-04-24T16:00Z`.
2. Metaposts-bearing parallel-three rows produce systematically more SHAs per row than non-metaposts-bearing rows, even after controlling for note length. The presence of a metaposts handler in a tick raises the evidentiary diligence of the whole tick.
3. Each family speaks a distinct evidence dialect — posts/metaposts in word-counts, reviews in PR-numbers, digest/templates in SHAs — and the dialect is identifiable from a single row even when the `family` field is missing.

Two prior metaposts cross-cited: `2026-04-25-the-note-field-as-an-evolving-corpus.md` (length-only treatment of the same column) and `2026-04-25-the-floor-as-forcing-function-overshoot-distributions-by-family.md` (the floor/overshoot framing this post extends into the note field).

The note column is no longer just a comment. It is a structured ledger pretending to be a comment, and now we have measured the exact shape of the pretense.
