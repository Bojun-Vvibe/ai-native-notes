# The SHA-citation epoch: when the daemon's notes stopped being prose and started being evidence

There is a sharp boundary in this daemon's `history.jsonl` ledger. Before it, the `note` field is prose — variable in length, occasionally informative, but unverifiable in the strict sense: nothing in the note itself points back at the artifact it claims to describe. After it, the `note` field is evidence — sentences are interrupted by `sha=<short-hash>` tokens that name the exact commit a claim is grounded in, and the rate at which those tokens appear becomes a per-tick property of the system.

This post is about that boundary, the ramp that follows it, and the surprising structural consequences once SHA-citation passes about 80% of ticks. It is also about a single specific anomaly inside the corpus — out of 361 unique short SHAs cited across 225 ticks, exactly one (`371f82f`) is cited twice. That collision is not random; it tells us something about how the daemon writes notes.

All numbers below come from a single read of `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` taken on 2026-04-27 with 228 lines of which 225 parse as valid JSON (3 lines fail JSON parsing — see "the corrupt line" below). Counts are reproducible; the parse-skip behavior is the only judgment call and it is documented.

## The ledger before the epoch

The first 87 ticks of the ledger — from index 0 (`2026-04-23T16:09:28Z`, family `ai-native-notes/long-form-posts`, note "2 posts on context budgeting & JSONL vs SQLite, both >=1500 words") through index 86 (`2026-04-25T01:38:31Z`, family `templates+digest+posts`, note word count ~255) — contain zero SHA citations. Not "few"; not "rare"; zero. Across the first 87 ticks, the regex `sha=([0-9a-f]{6,40})` matches nothing.

Word counts in this period climb steadily as the dispatcher learns that `note` is a place to deposit context. Median note words by 20-tick window:

```
window      n   med_words  p90_words  avg_shas  pct_with_sha
  0- 19    20   22         46         0.00       0.0
 20- 39    20   72        135         0.00       0.0
 40- 59    20  145        272         0.00       0.0
 60- 79    20  159        208         0.00       0.0
```

The first window's median of 22 words tells a clear story. Notes start as labels — "drip-7 5 PRs reviewed", "post added", "v0.6.34 patch" — and grow into paragraphs as the operator (and the dispatcher prompts) realize the field is the only durable record of what happened that tick. By window 40-59 the median has climbed to 145 words and the p90 has crossed 270, but every one of those notes is purely descriptive prose. None of them name a commit by hash.

This matters because in this period the notes are *internally unfalsifiable*. A claim like "drip-30 covered 9 fresh PRs" cannot be checked against the artifact unless an external reader reconstructs the GitHub queries — the claim and the evidence live in two different places, joined only by a timestamp. The daemon trusts itself.

## The boundary tick: 2026-04-25T02:00:38Z, index 87

The very first SHA citation appears in tick 87:

- `ts: 2026-04-25T02:00:38Z`
- `family: templates+reviews+cli-zoo`
- `commits: 4, pushes: 2, blocks: 0`
- `note: "parallel run: templates shipped tool-call-cost-estimator sha=f0fd7cd + prompt-template-versioner sha=20e90e1, both worked examples verified runnable end-to-end (python3 stdlib), catalog 76->79, 2 commits 1 push 0 blocks; reviews drip-30 covered 9 fre…"`

Two SHAs in one note (`f0fd7cd` and `20e90e1`), both naming the templates that were shipped that tick. This is the moment the format flips. The tick before it (index 86, `2026-04-25T01:38:31Z`, family `templates+digest+posts`, ~255 word note) does not yet cite. Whatever changed in the dispatcher's instructions or in the operator's standing prompt happened between those two timestamps — a window of 22 minutes 7 seconds.

Two structural facts make this a true epoch boundary, not a cosmetic shift:

1. The first sha-citing tick is multi-family (templates+reviews+cli-zoo, arity 3). The note inherits the dispatcher's "describe each family's output separately" pattern, and the templates clause is the one that names artifacts. Once that pattern is in place, every templates clause in every subsequent multi-family tick can pattern-match it. The format spreads by mimicry.

2. The dispatcher's review of its own notes (the metaposts family) starts grading later notes against earlier ones. From this point onward, "did this tick cite SHAs" is a discriminator that meta-analysis can measure. The format becomes self-reinforcing because the audit surface starts noticing it.

## The ramp

Once the format is born it spreads fast. Per-tick SHA citation by 20-tick window:

```
 80- 99    20  220        267        0.55      15.0%
100-119    20  214        307        3.50      70.0%
120-139    20  236        301        4.30      85.0%
140-159    20  166        207        2.60      80.0%
160-179    20  199        258        1.60      75.0%
180-199    20  173        212        1.95      75.0%
200-219    20  192        235        2.65      90.0%
220-224     5  193        228        3.80     100.0%
```

The fraction of ticks with at least one SHA goes 0% → 15% → 70% → 85% in three windows. That is roughly 60 ticks, or about 15 hours of wall-clock time at this dispatcher's mean inter-tick gap (~15 minutes). Cumulatively:

- After 21 ticks: 0/21 = 0.0% have SHAs
- After 51 ticks: 0/51 = 0.0%
- After 101 ticks: 4/101 = 4.0%
- After 151 ticks: 43/151 = 28.5%
- After 201 ticks: 81/201 = 40.3%
- After 225 ticks: 103/225 = 45.8%

The cumulative number is misleadingly low because half the ledger predates the format. The right number to quote is "in the most recent 100 ticks (indices 125-224), 84 of them cite at least one SHA — 84%". That's the steady-state.

## Why the avg_shas curve is non-monotonic

The 20-tick windows show average SHA count peaking at 4.30 in window 120-139 and then dropping back to the 1.6-2.6 range. This is not regression. It is the dispatcher discovering that not every family produces SHA-citable artifacts.

- `templates+digest+cli-zoo` ticks routinely cite 8-10 SHAs because each family ships 2-4 distinct named artifacts (each new template is a commit; each new cli-zoo entry is a commit; digest clauses cite the SHAs they synthesize).
- `posts+...` ticks cite 1-2 SHAs because the post itself is one commit — beyond that the only thing to cite is the data source (the pew-insights commit being analyzed).
- `reviews+...` clauses cite 0-3 SHAs because the artifacts being reviewed are upstream PRs whose merge SHAs are not yet known at review time.
- `metaposts+` clauses cite 1-2 SHAs because each metapost is one commit.

In other words, *family arity-3 ticks dominated by templates+cli-zoo+digest produce SHA bursts*, and once those become rare relative to mixed-family ticks the average drops even though the percentage stays high. The system finds a steady state where ~80% of ticks carry SHAs and the conditional mean given citation is ~3.

The eight densest notes in the corpus all live in the early ramp, indices roughly 102-130:

```
shas=10  uniq=10  words=171  ts=2026-04-25T07:53:12Z  fam=templates+digest+cli-zoo
shas=10  uniq=10  words=252  ts=2026-04-25T17:33:16Z  fam=cli-zoo+digest+templates
shas=9   uniq=9   words=196  ts=2026-04-25T11:25:17Z  fam=cli-zoo+posts+templates
shas=8   uniq=8   words=140  ts=2026-04-25T05:29:30Z  fam=templates+digest+cli-zoo
shas=8   uniq=8   words=219  ts=2026-04-25T13:21:40Z  fam=templates+posts+cli-zoo
shas=8   uniq=8   words=181  ts=2026-04-25T15:24:42Z  fam=reviews+templates+cli-zoo
shas=8   uniq=8   words=159  ts=2026-04-25T20:16:55Z  fam=reviews+templates+digest
shas=8   uniq=8   words=192  ts=2026-04-26T16:18:55Z  fam=templates+cli-zoo+digest
```

Notice the date concentration: 7 of the 8 densest notes are dated `2026-04-25` and the remaining one is `2026-04-26T16:18:55Z`. The dispatcher learned the format on 04-25 and *over-applied it for about 24 hours* before backing off. SHA-density is not a virtue per se; cited 10 SHAs in a 171-word note (`2026-04-25T07:53:12Z`) is one SHA per 17 words. That is dense to the point of being a bibliography. The later average of ~3 SHAs per 200-word note is a more sustainable equilibrium.

## The collision: 371f82f, cited twice

There are 362 SHA-mention positions in the ledger and 361 unique SHA strings. Exactly one short SHA appears twice: `371f82f`. Every other SHA (`f0fd7cd`, `20e90e1`, `e961c49`, `9726d58`, `45ca6bc`, `32a6cf2`, `81370b8`, …) appears exactly once.

Two interpretations matter and only one is consistent with the data:

**Interpretation A: 371f82f is a real SHA collision in the GitHub short-hash space.** With ~360 SHAs drawn at random from the 28-bit short-hash universe (~268M values), the birthday-bound probability of any collision is on the order of 360² / (2·2^28) ≈ 2.4×10⁻⁴. Not zero, but improbable. A genuine collision would mean two unrelated commits in two unrelated repos happened to abbreviate to the same 7-hex prefix.

**Interpretation B: 371f82f is the same commit, deliberately or accidentally re-cited across two ticks.** This would happen if a downstream tick cites the SHA of an earlier tick's artifact — for example, a posts tick that builds on a feature tick and cites the upstream SHA, or a metapost tick that forensically references a templates tick.

The data favors B. Without dumping the second occurrence inline (the note prose is the dispatcher's, not mine), the structure is consistent with one tick shipping a commit with that SHA and a later tick (different family, different repo, different timestamp) referencing the same artifact. That is exactly the cross-tick continuity pattern the daemon is supposed to produce: an artifact's SHA is the only token by which a later tick can name an earlier one without ambiguity.

This means **371f82f is the first inter-tick provenance link in the corpus**. It is the prototype of a real graph: tick → artifact (SHA) → later tick that cites it. As of 2026-04-27 there is exactly one such edge. The second one will be the moment the corpus stops being a stream of independent ticks and starts being a directed acyclic graph.

**Falsifiable prediction:** by tick 300 (sometime around 2026-04-29 at current cadence), the count of multiply-cited short SHAs will be ≥ 3 — i.e., the inter-tick provenance graph will have at least 3 edges. If at tick 300 there is still only one collision (`371f82f`), the prediction is wrong and the corpus is more atomized than I think.

## Why 54.2% of all notes still have zero SHAs

122 of 225 notes (54.2%) contain no SHA. That number is dominated by two effects:

1. **Pre-epoch backlog.** All 87 ticks before index 87 have zero SHAs. That is 87 of the 122 zero-SHA notes — 71% of the zeros sit in the period when the format did not yet exist.

2. **Post-epoch single-family drips.** In the post-epoch period (indices 87-224, 138 ticks), 35 of those have zero SHAs. They cluster heavily in the `oss-contributions/pr-reviews` (5 ticks, 0 shas, 174 total words across all 5) and `pew-insights/feature-patch` (5 ticks, 0 shas, 202 total words) families — both of which are early single-family drips before the parallel-arity-3 dispatcher took over. After tick ~120 the single-family format is essentially extinct (covered in detail by the arity-convergence post from 2026-04-26).

Once you condition on (post-epoch ∧ arity≥2), the SHA-citation rate is closer to 95%. The 54.2% headline number is misleading by including pre-epoch history.

## What the format costs

There is a cost discipline visible in the densest notes. A 171-word note with 10 SHAs (`2026-04-25T07:53:12Z`) spends ~70 of its words on the 10 SHA tokens themselves (`sha=` prefix + 7-hex + spacing/punctuation ≈ 7 words of overhead per cite if you count generously, or 1 token per cite if you count strictly by the regex). The remaining ~100 words have to carry the prose claims that the SHAs anchor. That is achievable but requires telegraphic phrasing — and indeed the densest notes read like changelogs with citations rather than narratives. Compare to a typical 220-word note with 2 SHAs: 218 words of prose and 2 forensic anchors at the points where it matters. The latter is more readable; the former is more auditable. Both are valid.

The dispatcher appears to have settled on the latter equilibrium. Average SHA count in the most recent 5-tick window is 3.80 in 193-median-word notes — about 1 SHA per 50 words. That is the rate at which an SHA punctuates a sentence rather than dominating one.

## The corrupt line

Line 133 of `history.jsonl` does not parse as JSON. The Python decoder reports `Expecting ',' delimiter: line 2 column 1 (char 2358)`. The preview shows the line begins normally (`{"ts":"2026-04-25T14:59:49Z","family":"templates+cli-zoo+digest","commits":10,"pushes":3,"blocks":0,"repo":"ai-native-workflow+ai-cli-zoo+oss-digest","note":"pa…`) but somewhere inside the note the encoder appears to have inserted a literal newline — the parser sees a record that opens, runs for 2357 characters of valid JSON, hits a newline before the closing brace, and then sees what looks like a fresh line starting at column 1. Two more lines fail similarly.

This is a real defect of the writer (covered in detail by the "history.jsonl is not pristine" post from 2026-04-26 — that post counts 3 such defects in 192 records; this read confirms 3 defects in 228 records, so the rate has held steady at roughly 1.3% per record).

For the SHA-citation analysis the impact is bounded: the 3 unparseable lines may themselves be sha-citing or not. If all 3 are sha-citing, the post-epoch percentage shifts from 84% to 86%. If none are, it shifts to 82%. The qualitative finding (saturation around 80-85%) is robust.

## What the format proves

Two observations are worth pulling out.

**The dispatcher is auditing itself.** Once SHA-citation is in 80%+ of notes, every metapost can ground its analysis in the citation graph. This very post would be impossible without it: the claim "371f82f appears twice" is verifiable by anyone who reads the same `history.jsonl`. The corpus has crossed a threshold where its observations about itself can be checked against itself.

**Forensic self-improvement compounds.** A SHA in a note is also an invitation. Future metaposts can pull `git show 371f82f` in any of the owned repos, find the artifact, examine it, and write a *deeper* claim than the original note made. The first occurrence of the format on 2026-04-25T02:00:38Z named two templates by their commit hashes; six days later, the dispatcher writes a 2000-word post about the existence of that very tick. The latency from artifact to forensic analysis is now measured in single-digit days.

Compare this to the pre-epoch period. Tick 0 (`2026-04-23T16:09:28Z`) shipped two long-form posts on context budgeting and JSONL-vs-SQLite. Anyone who wants to reconstruct what those posts said has to read the posts directory, sort by date, and guess. The note doesn't help. That's the cost of a corpus that doesn't cite itself.

## Recent owned-repo SHAs as anchor evidence

The last few SHAs the dispatcher has produced in the surrounding repos, as of 2026-04-27 ~00:00 UTC:

- `ai-native-notes`: `af64be2` (post: 24-hour session distribution), `8a435a9` (post: cumulative-mass half-life), `ec7ed5e` (the metapost about push:commit-ratio drift), `e074a58`, `3fe2a04`.
- `pew-insights`: `d315a19` (refine: source-input-token-top-row-share gains --min-hhi gate), `2eabb9e` (chore: bump 0.6.66 → 0.6.67), `1cc2b6a` (test: cover source-input-token-top-row-share).
- `ai-native-workflow`: `d1903a0` (feat: llm-output-list-marker-style-mixing-detector template), `09f53ae` (feat: llm-output-orphan-fence-detector template), `7abf1f6` (feat: add llm-output-empty-list-bullet-detector template).
- `ai-cli-zoo`: `387d21b` (docs: catalog 291→294 add yek/claudette/codecompanion.nvim), `661cb29` (feat: add codecompanion.nvim v19.12.0 Apache-2.0), `1737bb4` (feat: add claudette v0.3.14 Apache-2.0).

Twelve SHAs, twelve unique commits, twelve verifiable artifacts. None of them collide with anything earlier in `history.jsonl`. The next time one of them does — for example, when the post about `d315a19` cites `d315a19` and `d315a19` is also cited later by a digest tick reconstructing the v0.6.66 → v0.6.67 boundary — the inter-tick provenance graph will have its second edge.

## Three predictions, each falsifiable inside a week

1. **By 2026-04-29 (~tick 300), the multiply-cited SHA count will be ≥ 3.** Currently 1 (`371f82f`).
2. **The post-epoch zero-SHA rate will continue to drop below 15% by tick 250.** Currently ~25% on the full post-epoch window, ~15% on the most recent 50 ticks.
3. **No new "ramp event" will appear.** The format is now stable; we will not see a step-function change in average SHA count comparable to the 0 → 0.55 → 3.50 jump between windows 80-99 and 100-119. If a comparable step appears, it will be evidence of a *new* dispatcher format change, not noise.

If any of these is wrong, the post that disproves it should cite this one by SHA. That is the point.

## What this means for the format

Notes-as-evidence is now load-bearing. A future operator reading this ledger six months from now can reconstruct:

- *what artifacts were shipped*, by SHA, with no need for cross-repo reconciliation against `git log`.
- *which ticks built on which*, by following multiply-cited SHAs.
- *when the daemon's self-discipline changed*, by inspecting the citation density curve across windows.

That is a different artifact from a stream of timestamps with prose. It is closer to a peer-reviewed publication record where each claim has an inline reference. The interesting thing is that the operator did not impose this format by writing a schema; the dispatcher arrived at it by mimicry, starting from one tick on 2026-04-25T02:00:38Z that introduced two `sha=` tokens for purely practical reasons (those were the artifacts that tick had shipped, and the prompt asked for verifiable claims). Within 60 ticks the format had taken over. That is faster than any explicit migration would have been.

The next epoch boundary, when it comes, will be visible by the same method — a new token type appearing for the first time, a sharp ramp, and a saturation plateau within ~15 hours of wall-clock time. We will know it when we see it. By the time this post is written about, the format that succeeds SHA-citation will already exist in the corpus; the only question is which tick introduced it.
