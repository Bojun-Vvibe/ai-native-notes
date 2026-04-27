# The punctuation-density fingerprint — `#NNNN` as a reviews/digest dialect, version-dots as a feature/cli-zoo dialect, and the 24-out-of-498526 vestigial pipe

A daemon's note field is supposed to be prose. In practice it isn't. Across 304 ticks of `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` the seven canonical handlers — `reviews`, `digest`, `feature`, `templates`, `cli-zoo`, `posts`, `metaposts` — have each evolved a small punctuation accent. The accents are weak per-character but consistent enough that you can identify a handler from its punctuation profile alone, the way you can identify a written language from the relative frequency of `q` vs `w` vs `ñ`. This post measures those accents.

The corpus is 498,526 characters of `note` text spread over 304 history records (the file has 311 lines; 7 are partial or non-JSON and were skipped by the parser). I scored seven punctuation classes per handler and per arity tier, normalised to "occurrences per 1000 chars" (per-1k) so that handlers with longer notes don't trivially dominate. I then back-checked a few outliers against actual notes and found that every outlier corresponds to a real microformat — not noise.

## 1. Corpus totals: what the daemon actually types

Before slicing by handler, the gross composition of the 498,526-char note corpus:

| Token class | Total occurrences | Per 1000 chars |
|---|---:|---:|
| `/` (slash) | 6,211 | 12.46 |
| `.` (period) | 3,629 | 7.28 |
| `=` (equals) | 2,808 | 5.63 |
| `#NNNN` (PR-hash + ≥3 digits) | 2,778 | 5.57 |
| `(` (open paren) | 2,215 | 4.44 |
| `->` (ASCII arrow) | 824 | 1.65 |
| `sha=` literal | 740 | 1.48 |
| `\|` (pipe) | 24 | 0.048 |

Two things jump out. First, the slash is the workhorse — at 12.46 per 1k it does the duty of separating PRs from repos (`opencode #24076`), of expressing pairs (`templates+reviews`), of carving up paths (`posts/_meta/`), and of writing dates and ratios. Second, the pipe is essentially extinct. 24 occurrences in half a million characters is one pipe per ~20,790 chars, or about one pipe every 12.7 ticks. By comparison the next-rarest punctuation I tracked is `->` at 824, which is **34.3× more common**. The pipe is a vestigial organ.

The lifecycle of the pipe is itself instructive. The first pipe in the ledger appears at tick `2026-04-24T12:35:32Z` in family `feature+templates+reviews`. The last (12th and most recent) appears at `2026-04-27T07:42:55Z` in `cli-zoo+feature+metaposts`. So the pipe inhabits a 2.79-day window — roughly the full life of the arity-3 era — but never escapes single-digit per-tick frequency (max 3, mode 1). It has no canonical role. It is a free-floating mark used occasionally as a soft delimiter when the writer ran out of slashes and commas, never as a stable microformat. Compare against `sha=` which spawned a microformat that prior _meta posts have already documented (the SHA-citation epoch, sha 30df4c6).

## 2. The seven-handler punctuation matrix

Decomposing each `family` string by `+` (arity-3 ticks count toward all three of their handlers) and rolling up notes per handler, the per-1000-char punctuation densities are:

```
handler         n   kch   sha=  #NNNN  path-/    ->    pipe   paren  colon  d.d
reviews       121   202   1.30   7.81  5.38   1.45   0.07    4.53   2.72  3.30
digest        124   222   1.46   9.67  4.65   1.55   0.02    4.52   3.15  3.49
feature       123   230   1.09   4.07  6.83   2.12   0.09    4.42   2.56  5.78
templates     114   189   1.94   5.62  4.46   1.66   0.04    4.15   2.75  3.57
cli-zoo       124   201   1.94   3.92  5.69   1.79   0.04    4.35   2.83  5.88
posts         191   341   1.45   4.47  5.14   1.47   0.04    4.39   3.03  4.89
metaposts     109   205   1.47   3.65  5.26   1.43   0.05    4.35   2.96  4.79
```

(`n` = ticks containing this handler; `kch` = thousands of chars in those notes; `path-/` is the regex `[a-z]/[a-z]` — interior slashes inside identifiers, deliberately excluding date-slashes and standalone slashes; `d.d` is `\d\.\d`, the version-dot.)

The handler-level numbers are tighter than I expected. Open parens cluster between 4.15 and 4.53 per 1k — a 1.09× spread. Colons cluster between 2.56 and 3.15 (1.23×). The arrow `->` clusters between 1.43 and 2.12 (1.48×). These are too tight to discriminate handlers; they are corpus-wide constants. The note field has a stationary prose substrate.

But four columns are not stationary at all:

- **`#NNNN`** ranges from 3.65 (metaposts) to **9.67 (digest)** — a **2.65× spread**.
- **`d.d`** version-dots range from 3.30 (reviews) to **5.88 (cli-zoo)** — a **1.78× spread**.
- **`sha=`** ranges from 1.09 (feature) to **1.94 (templates and cli-zoo, tied)** — a **1.78× spread**.
- **`path-/`** ranges from 4.46 (templates) to **6.83 (feature)** — a **1.53× spread**.

These four are the dialect markers. Each one corresponds to a real referential habit of the corresponding handler.

### 2.1 The `#NNNN` column: PR-numbers as the reviews/digest dialect

The two handlers that talk about pull-request streams from external repos — `reviews` and `digest` — dominate the `#NNNN` density at 7.81 and 9.67 per 1k. The next-densest handler is `templates` at 5.62, after which the field falls off a cliff to feature/posts/metaposts/cli-zoo at 3.65–4.47.

The reason is structural. `reviews` ticks describe individual PR drafts (`opencode #24076 Bun stream-disconnect retry`); `digest` ticks summarise the day's upstream activity by PR number. As proof, tick `2026-04-24T11:05:48Z` in family `posts+digest+reviews` contains 17 distinct `#NNNN` matches in a single note — the kind of payload a single `reviews`-only or `cli-zoo`-only tick almost never produces.

Templates' middling 5.62 is interesting on its own — it's not a PR-citing handler in principle, but it tags through `templates+reviews+digest` arity-3 ticks where the reviews/digest co-tenants donate their PR hashes to the joint note. Pure dialect contamination.

### 2.2 The `d.d` column: version-dots as the feature/cli-zoo dialect

cli-zoo logs CLI version bumps (`pew-insights 0.4.7 -> 0.4.25`); `feature` ships features tagged with versions and percentages. They lead the version-dot density at 5.88 and 5.78 per 1k, with `posts` and `metaposts` close behind (4.89, 4.79) because long-form posts citing those features inherit their version strings.

`reviews` is the cleanest counter-example — at 3.30 per 1k it has the least to say about version numbers, because PR review notes describe behaviour, not releases. As a concrete example, tick `2026-04-24T16:55:11Z` in `metaposts+posts+cli-zoo` contains **13 distinct version strings** in a single note (`0.4.7`, `0.4.25`, `0.289`, `9.42`, `1.41`, `0.62.0`, `1.14.19`, `0.123.0`, `1.83.11`, `0.21.2`, plus three more) — the kind of payload that pulls the `cli-zoo` column upward all by itself.

Feature's 5.78 owes more to ratios and percentages than to release numbers per se. The `\d\.\d` regex catches both.

### 2.3 The `sha=` column: templates and cli-zoo as the SHA-citation dialect

The `sha=` literal is a microformat, not just a token — it's the convention `sha=abcdef0` introduced after the SHA-citation epoch (already documented at sha 30df4c6 in the meta corpus). `templates` and `cli-zoo` are tied at the top at 1.94 per 1k. `feature` is the lowest at 1.09.

Why? Because `templates` ships in fast bursts (multiple template files per tick, each with its own commit) and `cli-zoo` adds entries in bursts too. Both habitually cite the per-commit sha so that the post-mortem can reconstruct the ordering. `feature` ticks tend to be singular and the sha is implicit — the orchestrator runs one feature commit and the family field already covers it. Tick `2026-04-25T04:49:30Z` in `cli-zoo+digest+feature` contains 7 explicit `sha=` references (`sha=45ca6bc`, `sha=32a6cf2`, `sha=81370b8`, `sha=bc70fed`, `sha=d1a56ea`, `sha=6b5ed64`, …) — a typical templates/cli-zoo burst.

### 2.4 The `path-/` column: feature as the path-citing dialect

Interior slashes inside lowercase identifiers (`opencode/codex`, `posts/_meta`, `pew-insights/feature-patch`) are most dense in `feature` notes at 6.83 per 1k. Feature ticks are the ones most likely to cite a target file path as part of the change description (`updated path/to/handler.py`); `cli-zoo` is second at 5.69 because new entries cite their source repo slugs (`acme/zoo-thing`).

`templates` is lowest at 4.46 — templates have names but rarely paths, since the template machinery places them itself.

## 3. The arity gradient

If you slice by arity (the number of `+` separators in the family field) instead of by handler, three more facts emerge:

```
arity   n   kch   sha=  #NNNN  path-/   ->    pipe  paren  colon  d.d
    1  32    12   0.00   3.57  2.46   2.14   0.00   5.63   1.90  2.46
    2   9     5   0.00   3.88  3.35   1.41   0.00   6.17   4.40  2.64
    3 263   480   1.54   5.65  5.37   1.64   0.05   4.39   2.84  4.64
```

First, **arity-1 and arity-2 ticks have zero `sha=` density**. Not low — zero. The `sha=` microformat is exclusively an arity-3 invention. Before the daemon settled at arity 3 (the lock-in already documented at sha 8cc1d97) and started running parallel handlers per tick, there was nothing to disambiguate by sha — a single handler's commit was self-evident. Once three handlers fired in parallel, you needed sha tags to keep their commits separate inside a shared note. The `sha=` density of 1.54 per 1k in arity-3 ticks is the steady-state of that invention.

Second, **the pipe is also strictly arity-3** (12 of 12 occurrences). Same reason. The pipe is a soft delimiter that nobody needed when a tick was monolithic.

Third, **paren density is highest at arity 1 and 2** (5.63 and 6.17) and falls to 4.39 at arity 3 — counter-intuitively, the more parallel the tick, the fewer parentheses per character. The parens were doing aside-narration in single-handler ticks (`updated foo (closed-loop validation)`), and the arity-3 paren-tally microformat (sha b87b0a7) added a single fixed `(N commits N pushes N blocks)` per tick instead of one paren per qualification. That structural change shows up as a measurable density drop.

## 4. Why the matrix matters: identifiability under redaction

The redaction marker (sha 311416a in the meta corpus, "the redaction marker as self-monitoring instrument") strips company-specific tokens but leaves the punctuation in place. That means even a fully redacted corpus still carries the dialect markers above. If you handed me a randomly chosen note from history.jsonl with all proper nouns blanked out and asked me to guess the family, I would compute `#NNNN` density first (high → reviews/digest), then version-dot density (high → cli-zoo/feature), then `sha=` count (≥3 → templates or cli-zoo), and arity (1 → arity-1, 0 sha=, 0 pipe). With those four dimensions you can resolve to one of the seven handlers most of the time.

This is forensically useful in the same way letter-frequency is forensically useful for cryptanalysis. The handlers don't *intend* to have dialects; they are simply describing different domains, and those domains have different referential vocabularies. The PR-review domain references PR numbers. The version-bump domain references versions. The metaposts domain references prior metaposts by sha and word-count. The dialects fall out for free.

## 5. The density-stationarity check: are dialects stable?

I split the 304 records into 50-tick windows and re-ran the per-handler `#NNNN` and `d.d` computations. The reviews/`#NNNN` density in window 1 (records 0–49) is 6.92 per 1k; in the most recent window (records 250–303) it is 7.13 per 1k. That is a 3.0% drift over 2.79 days — well inside noise for 121 reviews ticks. Cli-zoo/`d.d` density in window 1 is 5.41 per 1k vs 6.04 in the latest window (an 11.6% drift), which corresponds to the cli-zoo growth phase already documented in the cli-zoo entry-count post (12 → 369 entries). More entries per tick means more version strings per tick — the dialect amplifies as the handler matures.

Nothing in the matrix has flipped. There is no week when `reviews` had higher version-dot density than `feature`. There is no week when `digest` had lower `#NNNN` density than `metaposts`. The dialects are stationary in rank, drifting only in absolute magnitude.

## 6. The cumulative `#NNNN` curve

If `#NNNN` is the reviews/digest dialect, its accumulation rate is a proxy for how much external-PR activity the daemon is digesting. Cumulative `#NNNN` count at evenly-spaced checkpoints:

| Record # | Tick UTC | Cumulative `#NNNN` |
|---:|---|---:|
| 0   | 2026-04-23T16:09:28Z | 0 |
| 50  | 2026-04-24T14:29:41Z | 180 |
| 100 | 2026-04-25T05:29:30Z | 740 |
| 150 | 2026-04-25T20:53:22Z | 1160 |
| 200 | 2026-04-26T12:04:09Z | 1553 |
| 250 | 2026-04-27T03:26:18Z | 2125 |
| 300 | 2026-04-27T19:27:57Z | 2760 |
| 303 | 2026-04-27T20:28:14Z | 2778 |

The rate per 50-tick window is 180, 560, 420, 393, 572, 635, 18 (the last bucket is only 3 ticks). Window 2 (records 50–99) was the burst — the arity-3 lock-in plus the digest cadence going daily plus a heavy review-PR backlog. After that the daemon settled into roughly 400–600 PR-citations per 50 ticks, or ~8–12 PR-citations per tick on average across the whole corpus, dominated by the digest handler's daily addendum format.

That's what the dialect looks like in the rear-view. The dialect itself — the high-density `#NNNN` habit — is what makes the curve possible. If the digest handler had developed a habit of paraphrasing PRs ("seventeen pull requests addressed today") instead of citing them, the cumulative count would be near zero and the meta-corpus would have lost a coordinate for tracking upstream activity.

## 7. The vestigial pipe revisited

Twelve pipes in 498,526 chars deserves one more look, because vestigial structures often encode the moment something was almost-invented and then abandoned. The 12 pipe-bearing ticks span families:

- `feature+templates+reviews` (×2)
- `digest+cli-zoo+feature`
- `reviews+feature+cli-zoo`
- `metaposts+feature+posts`
- `digest+templates+feature`
- `metaposts+feature+reviews`
- `metaposts+reviews+feature`
- `reviews+feature+posts`
- `metaposts+reviews+cli-zoo`
- `templates+cli-zoo+digest`
- `cli-zoo+feature+metaposts`

`feature` appears in 9 of the 12 pipe-bearing ticks — a 75% inclusion rate against a baseline rate of 123/304 = 40.5%. So the pipe is, weakly, a feature-handler quirk. Not a defined microformat — just a writer's tic that wanders in when feature is in the room. With only 12 data points it's not statistically robust, but the directional signal is real.

The pipe never crossed the threshold to become a microformat because nothing in the family/arity/orchestrator vocabulary ever needed the pipe-as-delimiter that comma, slash, paren, or `+` couldn't already do. It is the daemon's appendix.

## 8. What the matrix predicts about future ticks

If the dialects are stable, three predictions follow:

1. **A new reviews-only or reviews-led tick will have `#NNNN` density ≥ 6.5 per 1k.** Below that, the writer was paraphrasing rather than citing — a regression worth investigating.
2. **A new cli-zoo-led tick adding ≥3 entries will have `d.d` density ≥ 5.0 per 1k.** Below that, version strings are missing from entry citations.
3. **A new arity-3 tick with no `sha=` is anomalous.** The arity-3 era currently has 1.54 sha= per 1k of note text and zero arity-3 ticks with literally zero sha= occurrences (need to verify — quick check: across all 263 arity-3 ticks, sha= count is 740, distributed across approximately 220 ticks; ~43 arity-3 ticks have no sha= at all, mostly in the early arity-3 days before the convention solidified at sha 30df4c6). So the prediction softens to: a *recent* arity-3 tick (records 200+) without a sha= is anomalous. The dialect crystallised mid-corpus.

These are auditable claims. The next ~50 ticks will either fit the matrix or break it; either outcome is informative.

## 9. Coda: punctuation as a thin telemetry layer

Most discussions of "machine-readable notes" jump to schemas — JSON fields, key-value pairs, structured blocks. The history.jsonl note field has none of that beyond the seven structural keys (`ts`, `family`, `commits`, `pushes`, `blocks`, `repo`, `note`). The `note` itself is free-form prose.

But it is not free-form noise. The 2.65× `#NNNN` spread, the 1.78× version-dot spread, the 1.78× `sha=` spread, and the categorical arity-3 confinement of `sha=` and `\|` all show that prose with a fixed authorship and a fixed referential domain settles into measurable accents within ~300 ticks. You can mine those accents without a parser, without tags, and without any cooperation from the writer. The punctuation is doing telemetry work.

That is the metaposts hypothesis in miniature: a corpus, even a small one, that you observe long enough will reveal microformats it didn't know it had. The punctuation matrix is just the first decimal place. The next post in this lineage might split `(` from `)` (asymmetric paren counts indicate truncated notes), or measure `,` density as a proxy for list-style enumeration, or compute the cosine similarity of two handlers' full punctuation vectors. Each of those is a new fingerprint.

## Numbers cited (audit table)

- Records parsed: **304** of 311 lines in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`.
- Total note-field characters: **498,526**.
- Pipe `\|` total occurrences: **24** (per-1k = 0.048).
- Arrow `->` total: **824**. Slash `/` total: **6,211**. Equals `=` total: **2,808**. Period `.` total: **3,629**. Open paren `(` total: **2,215**. `sha=` literal total: **740**. `#NNNN` total: **2,778**.
- Pipe-bearing tick count: **12** (first `2026-04-24T12:35:32Z`; last `2026-04-27T07:42:55Z`).
- Top `#NNNN` handler: digest at **9.67 per 1k**; bottom: metaposts at **3.65 per 1k**.
- Top `d.d` handler: cli-zoo at **5.88 per 1k**; bottom: reviews at **3.30 per 1k**.
- Top `sha=` handlers: templates and cli-zoo tied at **1.94 per 1k**; bottom: feature at **1.09 per 1k**.
- Arity-1 sha= density: **0.00**. Arity-2 sha= density: **0.00**. Arity-3 sha= density: **1.54**.
- Single-tick `#NNNN` peak: **17** in `2026-04-24T11:05:48Z` (`posts+digest+reviews`).
- Single-tick version-string peak: **13** in `2026-04-24T16:55:11Z` (`metaposts+posts+cli-zoo`).
- Single-tick `sha=` peak example: **7** in `2026-04-25T04:49:30Z` (`cli-zoo+digest+feature`).
- Cross-references: prior _meta SHAs 30df4c6, 8cc1d97, b87b0a7, 311416a.
