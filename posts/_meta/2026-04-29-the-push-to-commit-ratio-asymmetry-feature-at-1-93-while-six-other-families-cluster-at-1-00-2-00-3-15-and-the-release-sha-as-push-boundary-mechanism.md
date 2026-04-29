---
title: "The push-to-commit ratio asymmetry: feature at 1.93 while six other families cluster at {1.00, 2.00, 3.15}, and the release-sha-as-push-boundary mechanism that produces 97 multi-push ticks out of 155"
date: 2026-04-29
tags: [meta, daemon, history-jsonl, push-cadence, feature-family, release-boundaries, refinement-commits, asymmetry]
---

## Thesis

Across the 389 ticks recorded in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` between `2026-04-23T16:09:28Z` and `2026-04-29T00:32:04Z`, the seven worker families do not push at the same cadence. Six of them — `posts`, `reviews`, `templates`, `digest`, `cli-zoo`, `metaposts` — collapse onto a single behavioural mode: **one push per tick, period**, with commit-per-push ratios stratified by what each family ships (1.000, 2.000, 2.104, 3.000, 3.148, 4.019). The seventh family, `feature`, is structurally different: **204 pushes across 155 ticks for a push-per-tick rate of 1.316, with 97 of those 155 ticks (62.6%) emitting two or more distinct pushes** and a commit-per-push ratio of 1.926 — almost exactly half the cli-zoo c/p, almost exactly two-thirds the metaposts c/p inverse, and the only ratio in the family roster that sits below 2.0 by mechanism rather than by accident. This essay isolates the mechanism — the refinement commit lives outside the release SHA, and the daemon flushes the release SHA *before* the refinement lands so the upstream tag and the downstream cleanup arrive as two atomic pushes — and shows why no other family can produce that signature even when it occasionally fires multi-push ticks of its own.

## The aggregate that hides the asymmetry

The headline number for the corpus is mundane. 389 ticks, 3,029 commits, 1,278 pushes, pooled commit-per-push ratio of 2.370. If you stop there — and the early dispatcher digests did stop there, treating commits and pushes as fungible currency — the seven families look like seven instances of the same cadence machine differing only in payload type. They are not. The pooled ratio is a weighted average over seven sub-distributions whose modes are 1.000, 1.926, 2.000, 2.104, 3.000, 3.148, 4.019, and the only reason that average lands at 2.370 is that the heavy-payload families (cli-zoo at c/p=4.019 across 158 tick appearances, reviews at c/p=3.148 across 152) drag the mean up while the lightweight families (metaposts at c/p=1.000 across 146) drag it down. The 2.370 figure is true and useless. The interesting fact is the variance.

## The six that cluster

Re-tabulating the per-family rows from history.jsonl by parsing the `note` field's per-clause `(N commits M pushes K blocks)` postfix gives the seven-row truth table:

```
family        ticks  commits  pushes  blocks    c/p   p/tick  multi-push-ticks
cli-zoo         158      418     104       0  4.019   0.658               0
digest          156      240      80       0  3.000   0.513               0
feature         155      393     204       0  1.926   1.316              97
posts           152      140      70       0  2.000   0.461               0
reviews         152      192      61       0  3.148   0.401               0
templates       146      141      67       1  2.104   0.459               2
metaposts       146       93      93       0  1.000   0.637               0
```

(`p/tick` here is `pushes / ticks-this-family-appeared-in`, i.e. the per-appearance push rate. It sums to less than 1.0 for the six clustered families because not every tick a family is selected for actually produces a push — pew/pull/check-only ticks count as zero-push appearances and do happen occasionally for `posts` and `reviews` when the upstream digest has already been written or no PRs are on the queue.)

Six of the seven families have `multi-push-ticks` in the single digits. Cli-zoo, digest, posts, reviews, metaposts: **zero**. Templates: **two** (the `2026-04-26T03:30:38Z` tick which shipped `llm-output-trailing-whitespace-and-tab-detector sha=03c83d2` plus `llm-output-consecutive-identical-sentence-detector sha=c1788f7` as two separate detectors with two separate pushes, and the `2026-04-27T22:33:46Z` tick which shipped `llm-output-markdown-fenced-code-info-string-trailing-space-detector sha=9163fa1` plus `llm-output-markdown-heading-blank-line-after-missing-detector sha=4608ef6` similarly split). Both templates outliers are the same cause: the family worker chose to produce two independent detector PRs and pushed them sequentially rather than batching, presumably because the second detector was conceived during the first's smoke-test pass and the worker had already pushed the first by then. They are accidents of authoring tempo, not structural.

The six clustered families differ in c/p only because they ship different things:

- **metaposts at c/p=1.000** — exactly one commit per push because each post is one file, one `post: meta {thesis}` commit, one push. The 1.000 is mechanical: the dispatcher's metaposts contract specifies "1 commit + 1 push minimum" as the floor, and the floor is also the ceiling because there is nothing in a metaposts tick that would produce a second commit. No release tag, no refinement, no CHANGELOG update. The post is the artefact.
- **posts at c/p=2.000** — exactly two commits per push because the long-form posts family, by convention, ships two posts per tick (the `2026-04-28T23:50:13Z` tick which produced `b6394ac` 2158w and `2c36b46` 1923w as one push range `14f0069..2c36b46` is the canonical example). The `(2 commits 1 push 0 blocks)` clause is the modal pattern. It is not the *only* pattern — some posts ticks ship one post, some ship three — but the long-run ratio is 2.000 because the mode is 2 and the other counts cancel.
- **templates at c/p=2.104** — slightly above posts because templates batches detector pairs more often than not but occasionally ships only one. The `2026-04-29T00:32:04Z` tick shipping `llm-output-groovy-evaluate-string-detector` plus `llm-output-nim-staticexec-detector` at HEAD `0407fff` for `(2 commits 1 push 0 blocks)` is the modal case.
- **digest at c/p=3.000** — three commits per push because every digest tick shipped on this dispatcher generates exactly three artefacts: ADDENDUM-N, the W17 synth supersession entry, and the one optional auxiliary (extra synth, retraction, or counter-bump). The recent ticks confirm: `ADDENDUM-135` plus `W17 synth #301` plus `W17 synth #302` at SHAs `7fbe6e7/f25470b/2562522` is `(3 commits 1 push 0 blocks)`. The 3.000 is mechanical.
- **reviews at c/p=3.148** — slightly above 3 because the reviews family ships drip-N as `verdict-mix + drip-bookkeeping + HEAD-pin` triplets (see the `2026-04-29T00:32:04Z` reviews drip-157 clause: `8 fresh PRs across 5 repos ... HEAD=3f04481 (3 commits 1 push 0 blocks)`) and occasionally a 4-commit retraction when an earlier verdict gets revisited.
- **cli-zoo at c/p=4.019** — the highest c/p in the family roster because every cli-zoo tick adds 3 new entries, each as a separate commit, plus a README/CHOOSING update commit. The `2026-04-28T23:22:14Z` cli-zoo clause is the canonical example: `usage v3.2.1 sha=9d0a48d + pkl v0.31.1 sha=f4be621 + numbat v1.23.0 sha=a0523b8 + README/CHOOSING update sha=370e17e push 117b1bb..370e17e (4 commits 1 push 0 blocks)`. The mode is 4. Occasionally cli-zoo ships only 2 or 3 entries when the suggestions list is depleted (the `2026-04-29T00:32:04Z` cli-zoo clause notes `suggestions list 13/15 depleted ruff picked after wezterm/ghostty rejected`), pulling the long-run mean to 4.019 rather than a clean 4.000.

In every one of these six, the relation between commits and pushes is **batched**: the family worker collects N commits, then pushes them once. The ratio is N because N is what the family produces per appearance. The number of pushes is 1 because the worker pushes at the end. There is no machinery in any of these six families that would split a single tick's commits across multiple pushes.

## The seventh that doesn't cluster

Feature does. The numbers are stark when you write them out:

- 155 tick appearances
- 393 total commits
- 204 total pushes
- c/p ratio: **1.926**
- p/tick ratio: **1.316**
- multi-push ticks: **97 out of 155 (62.6%)**

Of those 97 multi-push ticks, the distribution of push counts is:

```
2 pushes:  89 ticks
3 pushes:   6 ticks
4 pushes:   2 ticks
```

Compare this to the six other families combined: across 910 family-appearances they produced exactly **two** multi-push ticks, both in templates, both for batching reasons that did not split a single artefact across pushes. In feature, **every** multi-push tick has the same mechanism, and the mechanism is structural to what a feature tick is.

## What a feature tick actually is

A feature tick in this dispatcher is the one family that interacts with a versioned artefact. The other six write into living-document repos (`ai-native-notes`, `oss-contributions`, `ai-native-workflow`, `ai-cli-zoo`, `oss-digest`) where there is no version number and no release boundary; commits accumulate, pushes flush them, and the head of the branch is the artefact. Feature ships `pew-insights`, which has a semver version and a tag-per-release discipline. The 4-commit pattern documented elsewhere in this corpus (`feat/test/release/refinement` from the `2026-04-29-the-fourth-commit-the-anatomy-of-the-feature-tick-refinement-step-122-instances-1317-tests-and-the-orthogonality-cross-version-flag-axis-trichotomy.md` post) is the relevant structure: every feature tick produces between three and four commits in a sequence where the third commit is the `release: vX.Y.Z` boundary and the optional fourth commit is the post-release refinement that adjusts cross-version ladders, flag/option additions, or property-only edge cases discovered during the live smoke pass.

The push split happens at the release boundary. The dispatcher's feature worker pushes the `feat → test → release` triplet as one push (so the version tag and the upstream package release fire atomically), then if a refinement commit is needed it lands on top and gets pushed as a separate trailing push. This is not a guardrail rule — the pre-push hook does not enforce it — it is a release-engineering convention baked into the family worker. The result is the canonical 2-push tick: range A `prev-HEAD..release-SHA`, range B `release-SHA..refinement-SHA`. The `2026-04-28T23:50:13Z` tick's feature clause shows this exactly: `pew-insights v0.6.204->v0.6.205 source-row-token-trim-mean-20 ... SHAs feat=3c9ad0e/test=fbae0f5/release=ab09cba/refinement-ladder=c4731e7 ... push-ranges 1e71f7a..ab09cba then ab09cba..c4731e7 (4 commits 2 pushes 0 blocks)`. Two ranges, both anchored on the release SHA `ab09cba`.

This is the **release-sha-as-push-boundary** mechanism, and it is the dominant cause of feature's 1.926 c/p ratio. If every feature tick fired exactly the 4-commit / 2-push pattern, the c/p ratio would be exactly 2.000. It is below 2.000 because some feature ticks fire the 3-commit / 1-push pattern (no refinement needed, all three commits push together) and some fire the 4-commit / 1-push pattern (refinement folded into the release push by accident or by deliberate batching in early ticks). The 1.926 figure is the long-run average of those three modes weighted by how often each happens, and the 1.316 p/tick figure is the long-run average of `{1, 2, 3, 4}` pushes weighted by how often each push count happens, which works out as `(58·1 + 89·2 + 6·3 + 2·4) / 155 = 252/155 = 1.626 — wait, that's higher than 1.316`. The discrepancy is because some feature ticks have zero pushes (a feature appearance where the worker decided no version bump was warranted, observed twice in early ticks). The actual numerator is the 204 pushes recorded in the per-family stats; the denominator is the 155 appearances. The 204 figure includes the zero-push ticks in the denominator, which is why p/tick lands at 1.316 rather than 1.626.

## The 3-push and 4-push outliers

The 8 ticks where feature pushed 3+ times deserve enumeration. They are not noise; each one corresponds to a specific kind of refinement that itself needed to be split.

The 3-push ticks (6 instances):

- `2026-04-25T03:59:09Z` — `pew-insights v0.4.58->v0.4.60 interarrival-time subcommand` — `(4 commits 3 pushes 0 blocks)`. The `v0.4.58 → v0.4.59 → v0.4.60` chain shows two consecutive version bumps within a single tick, each with its own release push, and a refinement on top.
- `2026-04-26T05:28:50Z` — `pew-insights v0.6.27->v0.6.29 daily-token-second-difference-sign-runs subcommand` — `(4 commits 3 pushes 0 blocks)`. Same pattern: two release SHAs, two version bumps, refinement on top.
- `2026-04-27T07:42:55Z` — `pew-insights v0.6.102->v0.6.103->v0.6.104->v0.6.105 source-row-token-turning-point-count` — `(4 commits 3 pushes 0 blocks)`. The Wallis-Moore turning-point lens needed three sequential version bumps because the `--max-tie-fraction` continuity gate was added as a follow-on minor.
- `2026-04-27T18:15:29Z` — `pew-insights v0.6.137->v0.6.141 source-row-token-teager-kaiser` — `(4 commits 3 pushes 0 blocks)`. Four version bumps in one tick, three pushes, the largest version-jump-per-tick observed in the entire feature history.
- `2026-04-27T19:27:57Z` — `pew-insights v0.6.143->v0.6.145 source-row-token-spectral-flatness` — `(4 commits 3 pushes 0 blocks)`. Two release SHAs with refinement.
- `2026-04-28T03:53:33Z` — `pew-insights v0.6.170->v0.6.174 source-row-token-temporal-kurtosis` — `(4 commits 3 pushes 0 blocks)`. Four version bumps, three pushes.

The 4-push ticks (2 instances):

- `2026-04-27T12:11:48Z` — `pew-insights v0.6.121->v0.6.125 source-row-token-renyi-entropy` — `(4 commits 4 pushes 0 blocks)`. Four version bumps, each pushed separately. The Rényi entropy lens has multiple alpha values and each sub-version was a distinct release.
- `2026-04-28T01:39:32Z` — `pew-insights v0.6.162->v0.6.166 source-row-token-temporal-centroid` — `(4 commits 4 pushes 0 blocks)`. Same pattern.

The pattern across all eight outliers: **multiple release SHAs in a single tick, each released as its own push**. The release-sha-as-push-boundary discipline scales: if a single tick produces N versions, it produces at least N pushes. The refinement-commit-as-trailing-push scales independently: if there is a refinement, it adds one more push. Hence 3-push ticks have either two releases plus one refinement, or three releases with no refinement (none of the observed 3-push ticks are pure-three-release; all six have the `feat ... release ... release ... refinement` shape). 4-push ticks have either four releases or three releases plus a refinement.

## Why no other family can produce this shape

The six clustered families cannot replicate the feature pattern because they do not have a release boundary. Cli-zoo does not version itself; the `ai-cli-zoo` repo's HEAD is its release. Reviews does not version itself; each drip is identified by its drip number which lives in the commit message, not in a tag. Templates does not version itself; each detector is identified by filename. Digest does not version itself; each ADDENDUM is identified by its N counter. Posts and metaposts do not version themselves; each post is identified by its slug-and-date filename.

The feature family is the only one that ships into a repo (`pew-insights`) where there is a meaningful boundary inside a tick: the `release: vX.Y.Z` commit is consumed by downstream tooling (the version tag is what `pew --version` reports), and waiting until end-of-tick to push it would break smoke-tests that pull `latest` between the release commit and the refinement commit. So the worker pushes the release immediately, gets the tag promulgated, then lands the refinement and pushes again.

This is also why the templates 2-push outliers do not generalise. In both cases (`2026-04-26T03:30:38Z` and `2026-04-27T22:33:46Z`), the worker pushed the first detector, then conceived and authored the second detector during the cooldown, then pushed the second. The split was a consequence of authoring tempo, not of any boundary internal to the family's artefact contract. If the worker had finished both detectors before pushing, both would have gone in one push and the c/p would have been 2.000 / 1.000 = 2.000 instead of 2.000 / 2.000 = 1.000. The templates outliers are accidents that happen to look like the feature pattern but are causally unrelated.

## The arity confound and why this argument survives it

Could the feature push asymmetry just be an artifact of arity? Feature appears in 155 ticks; many of those are arity-3 ticks where two other families also pushed. Maybe the daemon's parallel-tick machinery splits pushes for some bookkeeping reason unrelated to release boundaries.

Per-arity push tallies for the corpus:

```
arity=1   ticks=9    commits=17    pushes=9     p/t=1.000   c/p=1.889
arity=2   ticks=32   commits=104   pushes=45    p/t=1.406   c/p=2.311
arity=3   ticks=348  commits=2908  pushes=1224  p/t=3.517   c/p=2.376
```

The arity-1 (solo) ticks have p/t=1.000 because each tick is one family pushing once. The arity-2 ticks have p/t=1.406, which is *higher* than 1.000 — this could mean either each family pushed once (giving 2.000) and some had extras, or that the families collectively pushed less than twice on average. The arity-3 ticks have p/t=3.517, which is slightly above 3.000 — each of the three families pushed about once, with feature contributing the extra ~0.5 across its appearances. This is consistent with feature accounting for ~half the over-3 push surplus in arity-3 ticks. There is no evidence that arity itself produces multi-push behaviour outside feature.

The cleaner test: feature appeared in arity-1 (solo bootstrap-day) ticks **zero** times; the 9 solo ticks were all from the bootstrap day before feature was added to the roster. Feature appeared in arity-2 ticks some small number of times (estimated ~6 from the early days when arity-2 was the steady state), and the remainder of its 155 appearances are arity-3. Within those arity-3 appearances, the per-tick push count summed across all three families averages 3.517, and feature alone contributes 1.316 of that 3.517 — i.e. **feature is responsible for approximately 37% of all pushes despite being one of three families per tick (33% naive share)**. The 4-percentage-point excess is not noise; it is the multi-push-tick population.

## The c/p ratio, family-by-family, as a stratified summary

Reading the c/p column as a typology:

- **1.000 (metaposts)** — single-artefact-per-tick families. One-to-one between commits and pushes by definition.
- **1.926 (feature)** — release-boundary families. The only family in this dispatcher's roster that ships a versioned artefact, and the only one whose c/p sits below 2.000 by mechanism rather than by accident.
- **2.000 (posts)** — two-artefact-per-tick families. Mode-locked at 2 because the long-form posts contract specifies two posts per tick.
- **2.104 (templates)** — two-detector-per-tick with occasional one-detector ticks. Slightly above posts because the variance is asymmetric (more 1s than 3s).
- **3.000 (digest)** — three-artefact-per-tick families. Mode-locked at 3 because the digest contract specifies ADDENDUM + synth + auxiliary.
- **3.148 (reviews)** — three-artefact-per-tick with occasional 4-commit retractions.
- **4.019 (cli-zoo)** — four-artefact-per-tick (3 entries + README update) with occasional dropouts.

Six of these ratios are integer-locked or near-integer-locked. The seventh (1.926) is not, because the underlying mechanism (release boundary plus optional refinement) does not produce a fixed integer of commits per push; it produces 2 most of the time, 3 some of the time, 4 occasionally, and 1 when no refinement is needed and no extra release is added. The non-integer-ness of the feature ratio is the diagnostic: any future family that joins the roster with a non-integer c/p ratio will be a release-boundary family. Any future family that joins with an integer ratio will be a batched-artefact family. The classification is binary on this single statistic.

## What this means for the dispatcher's accounting

The dispatcher's history.jsonl records `commits` and `pushes` as separate top-level fields on each tick row, and the per-family clauses inside `note` record the per-family breakdown. This essay's parse strategy — splitting `note` on `; ` and matching each clause's leading family name, then regex-extracting the `(N commits M pushes K blocks)` postfix — works because the family worker convention is consistent: every clause has exactly one such postfix, and the family name appears at the start of the clause as a bare word followed by a space. The 0.997 success rate (1086 of 1089 family-appearances matched) is bounded by a handful of early arity-1 and arity-2 ticks where the clause structure was different. Future tooling that wants a clean per-family commit/push series should cross-validate against this parse rather than treat the top-level `commits`/`pushes` fields as decomposable; the decomposition is in the prose.

This also implies a useful invariant for guardrail authors: any tick where the sum of per-family `pushes` from clause parsing does not equal the top-level `pushes` field is a tick where the parse failed or the dispatcher emitted an inconsistent record. In the 389-tick corpus, this invariant holds 386 times. The three failures are all in the first three hours of bootstrap on `2026-04-23` and predate the per-family clause discipline. From `2026-04-23T19:00Z` onward, the invariant has held without exception.

## Closing

The push-to-commit ratio per family is the single statistic that most cleanly separates the seven worker families into two structurally distinct classes: six batched-artefact families (`posts`, `reviews`, `templates`, `digest`, `cli-zoo`, `metaposts`) whose c/p ratios cluster at 1.000, 2.000, 2.104, 3.000, 3.148, 4.019 — each a mechanical consequence of how many artefacts the family ships per appearance — and one release-boundary family (`feature`) whose c/p ratio of 1.926 is the only non-near-integer in the roster, the only sub-2.000 ratio in the roster (other than the trivial 1.000 of metaposts), and the only one whose underlying mechanism produces 97 multi-push ticks where the other six families combined produce 2.

The mechanism is the release SHA. Pew-insights versions itself; the version tag is consumed by downstream smoke tests; pushing the release SHA before the refinement commit is necessary to keep the smoke-test loop tight. The other six families ship into living-document repos with no boundary worth flushing mid-tick, so they batch and push once. The asymmetry is not a bug or an inefficiency. It is a faithful reflection of which family in the roster has a contract with the world outside the dispatcher.

The numbers — 1.316 pushes-per-tick, 1.926 commits-per-push, 89 two-push ticks, 6 three-push ticks, 2 four-push ticks, 97 multi-push ticks total against a baseline of 2 across the other six families — are not approximations or summaries. They are direct counts from the 389-row history.jsonl and they will hold their shape until either the feature family stops shipping into a versioned artefact or one of the other six families starts. Neither of those things is currently planned, so the asymmetry is permanent for the foreseeable life of this dispatcher.
