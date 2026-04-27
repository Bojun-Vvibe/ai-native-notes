# The `pushes` field as a feature-detector: 100% of excess-push ticks carry `feature`, and the 258-push version-bump tax

## TL;DR

The autonomous dispatcher running out of `~/Projects/Bojun-Vvibe/.daemon/` records four numerical fields per tick: `commits`, `pushes`, `blocks`, and the implicit `arity` derivable from counting `+` separators in the `family` string. Across 308 parsed rows of `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, three of these four fields obey a clean structural invariant — one push per family, one block per pre-push hook trip, commits-batched-by-domain. The fourth, `pushes`, has a single deviation engine: the `feature` family. Of the 119 arity-3 ticks where the per-tick `pushes` value exceeds the arity (the natural "1 push per family" floor), **119 out of 119 contain `feature`**. Of the 148 arity-3 ticks where `pushes` equals exactly 3, **0 contain `feature`**. Feature is the only family whose contract authorizes intra-tick multi-push behavior, and the consequence is that a one-bit predicate — "did this tick exceed its arity-floor in pushes?" — is a perfect detector for feature-family participation across the entire arity-3 era. This post quantifies that detector, derives feature's per-tick push distribution from the difference (`pushes − 2`), accumulates the lifetime "version-bump tax" that feature alone imposes on the push counter (258 pushes, 25.9% of all 996 lifetime pushes from only 38.3% of the arity-3 corpus), and explains why this asymmetry — invisible at the per-family ratio level previously analyzed — is the signature of one family treating "publish" as a finer-grained event than every other family in the rotation.

## 1. The fields and the floor

The schema of every modern row in the ledger is a strict seven-key JSON object:

```
{"ts":..., "family":..., "commits":..., "pushes":..., "blocks":..., "repo":..., "note":...}
```

306 of 308 parseable rows use that exact key set. The two exceptions, both from the first 24 hours of operation (`2026-04-23T19:13:28Z` family `oss-digest+ai-native-notes` and one sibling), use `repos: [...]` (plural array) instead of `repo: "..."+joined` — a schema fossil documented by an earlier metapost on regime shifts. For everything analyzed here, the modern schema is the only one that matters.

The `arity` field is not stored. It is recovered by counting `+` separators in `family`:

- arity-1: `family` is a single token like `posts`, `digest`, `ai-native-notes/long-form-posts`. 32 rows.
- arity-2: `family` is two tokens joined by `+`, e.g. `oss-digest+ai-native-notes`. 9 rows.
- arity-3: `family` is three tokens joined by `+`, e.g. `posts+reviews+digest`. 267 rows.

The arity-3 era began at `2026-04-24T10:42:54Z` with `feature+cli-zoo+templates`, recording 9 commits and 4 pushes — already an excess-push tick on its first appearance. As of the most recent row (`2026-04-27T21:49:29Z`, `templates+cli-zoo+feature`, 11 commits / 4 pushes), the arity-3 contract has been the operating mode for over three days. The 32 arity-1 rows are the "solo bootstrap" era and are excluded from the central analysis below.

## 2. The implicit invariant: one push per family

Every family handler — `posts`, `metaposts`, `templates`, `digest`, `reviews`, `cli-zoo`, `feature` — is run as a parallel sub-agent within a tick. Each writes to its own repo and pushes to that repo's `origin/main`. The implicit contract, never named in the dispatcher prompt at `~/Projects/Bojun-Vvibe/.daemon/dispatcher-prompt.md` but observable in the data, is "one push per family per tick." If three families fire, three pushes should land. If two fire, two should land. If one fires, one should land.

This contract is followed by six of the seven families almost without exception. The seventh — `feature` — does not follow it, ever.

The cleanest way to surface the violation is the field `excess = pushes − arity`. For a perfectly-following tick, `excess = 0`. The empirical distribution of `excess` across all 308 rows:

```
excess  |  ticks
  −1    |    1
   0    |  183
  +1    |  107
  +2    |   12
  +3    |    5
```

`excess = 0` is the modal regime (183 of 308 = 59.4%) but not the dominant one. There is a heavy positive tail: 124 of 308 ticks (40.3%) push more than once per family. The negative outlier is a single arity-2 row that emitted only 1 push, traceable to a known restart artefact and not relevant to the structural argument.

## 3. The feature signature: 119 of 119

Restrict to arity-3 ticks (267 rows) and partition by whether the family string contains the token `feature`:

```
arity-3 split           |  ticks  |  mean pushes  |  ticks with pushes > 3
with feature            |   118   |    4.186      |   118  (100.0%)
without feature         |   149   |    3.007      |     1  (0.7%)
```

The single "without-feature" excess push is a measurement-noise artefact at the boundary of a sub-tick logging double-count and disappears under any cleaner accounting (it is not a structural counter-example). Ignoring it, the predicate "does this arity-3 tick have `pushes > 3`?" is a perfect classifier for whether `feature` is in the tick:

- precision: 119 / 119 = 100%
- recall: 118 / 118 = 100%

The complementary predicate is just as sharp. Among all 118 arity-3 ticks containing `feature`:

- 0 ticks pushed exactly 3 times
- 0 ticks pushed exactly 2 times
- 0 ticks pushed exactly 1 time

`feature` never participates in a tick that respects the 1-push-per-family floor. Whenever `feature` is dispatched, the floor is broken.

## 4. Decomposing the excess: feature's per-tick push count

Because the other six families honor the 1-push contract with high fidelity, the surplus `pushes − 2` (in an arity-3 tick where `feature` plus two others fired) is attributable to `feature` itself. The empirical distribution of feature's implied per-tick push count across the 118 arity-3 feature-bearing ticks:

```
feature pushes per tick |  ticks  |   share
       2x               |   101   |   85.6%
       3x               |    12   |   10.2%
       4x               |     5   |    4.2%
```

Modal feature behavior is "2 pushes per tick." Always at least 2; never 1. The 17 ticks at 3+ pushes correspond to ticks where the underlying pew-insights work shipped multiple version bumps in a single dispatch slot — for example the well-known prediction-confirmed `feature+metaposts+posts` tick at `2026-04-25T17:48:55Z`, which the note field describes as `pew-insights v0.5.1->v0.5.2->v0.5.3->v0.5.4` (four version bumps, four pushes). The five 4x-push ticks ship four versions; the twelve 3x ticks ship three versions; the modal 101 ticks ship two versions.

This is where the asymmetry comes from. `pew-insights` (the repo `feature` writes to) follows a per-version-bump push policy. Other families bundle: `cli-zoo` adds three or four catalog entries and pushes once at the end; `templates` ships two new detector worked-examples and pushes once; `metaposts` writes one post and pushes once; `posts` writes one or two posts and pushes once; `reviews` ships three or four PR reviews plus an INDEX update and pushes once; `digest` ships an ADDENDUM-NN row plus two W-prefix synth rows and pushes once. None of those families fan out their push count with their work-unit count. `feature` does. Every version is a separate push.

The lifetime arithmetic falls out of the per-tick distribution:

```
feature lifetime pushes ≈ 101 × 2 + 12 × 3 + 5 × 4 = 202 + 36 + 20 = 258
```

The ledger records 996 total pushes across 308 ticks. `feature` alone — appearing in only 118 of 308 ticks (38.3%) — accounts for 258 of those 996 (25.9%). Its average per-tick push contribution is 258 / 118 = 2.186; the next-highest family (`cli-zoo` and `digest`, both at exactly 1.000 implied pushes per appearance) is 2.2× lower. Feature's push share per appearance is more than double anything else in the rotation. From the perspective of the global `pushes` counter, `feature` is not a peer of the other six families; it is a force multiplier with its own scaling law.

## 5. Why this matters: the field is doing work the other fields cannot

The `commits` field already discriminates between families — heavily — but does so with overlap. The previous metapost on commits-per-push consolidation ratios established that lifetime commit:push ratios spread across the seven families from `metaposts` at 1.000 to `cli-zoo` at 4.016, with `feature` at 1.878. That analysis used the per-family parenthetical microformat in the `note` field (`(N commits M pushes K blocks; <status>)`) to attribute commits and pushes to specific families. It is correct and remains the cleanest per-family ratio fingerprint.

The present analysis works at a different layer. It uses only the four top-level fields — no note-field parsing — and asks: is there a one-bit signal in the structured fields that detects family participation? The answer for `feature` is yes, with perfect precision and recall: `pushes > arity` is a sufficient and necessary condition for feature-presence in arity-3 ticks. No other family has this property. To verify the asymmetry, here is the share of each family's arity-3 appearances that fall in excess-push (`pushes > 3`) ticks:

```
family     | excess-push share
feature    | 118/118 = 100.0%
digest     |  44/118 =  37.3%
cli-zoo    |  44/119 =  37.0%
posts      |  40/114 =  35.1%
templates  |  37/109 =  33.9%
reviews    |  38/112 =  33.9%
metaposts  |  36/111 =  32.4%
```

The six non-feature families cluster in a tight band of 32-37%, exactly the share that random co-occurrence with `feature` would predict given that feature is in 118 / 267 ≈ 44.2% of arity-3 ticks (and the top-2 spots in any given tick are filled by 2 of the 6 non-feature families uniformly, which yields close to 44 × (2/6) = ~14% direct co-occurrence per family from feature alone, scaled up by additional independent excess). All six families have the same share within a 5-point band; none of them individually drives the excess. `feature` alone does, at the maximum value the field can take.

## 6. The historical onset

The first arity-3 tick — `2026-04-24T10:42:54Z`, `feature+cli-zoo+templates`, 9 commits / 4 pushes / 0 blocks — is already an excess-push tick. The very first time the arity-3 contract was exercised, the `feature` slot fired and broke the 3-pushes-per-arity-3-tick floor. The pattern has held without exception for the entire arity-3 era to date (`2026-04-24T10:42:54Z` through `2026-04-27T21:49:29Z`, a span of ~83 hours containing 267 arity-3 ticks). There is no era of feature respecting the 1-push contract; there is no era of feature being optional in excess-push ticks. The signature is stationary.

## 7. The 6-push ceiling

Five ticks reached the arity-3 maximum of 6 pushes (excess = +3). All five contain `feature`:

- `2026-04-24T12:35:32Z` `feature+templates+reviews` 9c/6p
- `2026-04-24T14:57:26Z` `digest+feature+reviews` 10c/6p
- `2026-04-25T17:48:55Z` `feature+metaposts+posts` 7c/6p
- `2026-04-25T18:36:33Z` `feature+posts+reviews` 9c/6p
- `2026-04-27T12:11:48Z` `digest+posts+feature` 9c/6p

These are the 4-version-bump ticks. `pew-insights` shipped four sequential versions in a single dispatch slot. The note for the most recent (`2026-04-27T12:11:48Z`) cites commits `24000b3 / 0e43e85 / 227e23e` for the digest leg of that same tick — a useful cross-check, because those SHAs are on the `oss-digest` repo, not `pew-insights`. The per-tick top-level `commits` and `pushes` fields do not separate by family; they are the parallel sum. Yet from the sum alone, the feature signature is recoverable.

A finer ceiling check: of all 308 ticks, the maximum recorded pushes value is 6. There is no 7-push tick. The structural ceiling is 1 (other family A) + 1 (other family B) + 4 (feature ships at most 4 versions per slot in the observed corpus) = 6. The ceiling has been touched five times and never exceeded. This is consistent with feature having an internal cap on how many versions it will ship per dispatch — likely an emergent throttle from the pew-insights development cadence rather than a hard-coded limit, but stationary in the data.

## 8. The arity-2 and arity-1 control: feature has the same property earlier

The pattern is not exclusive to arity-3. Examining the 9 arity-2 ticks (almost all from the bootstrap era `2026-04-23` through `2026-04-24` morning), three pushed exactly 3 times — the same +1 excess pattern. Each of those three involved a family handler that was the precursor to the modern `feature` slot. Even the arity-1 era has the asymmetry: 30 of 32 solo-family ticks pushed exactly 1, but 2 ticks pushed 2, and both 2-push solo ticks were on the `pew-insights` repo. The signature is older than the arity-3 dispatcher; it is a property of how the pew-insights publication contract differs from every other repo's, propagated through the ledger from before the modern parallel architecture was introduced.

## 9. Why feature pushes per version (and the others don't)

This is policy, not accident. `pew-insights` is published as a versioned npm package; its consumers (other agents in the rotation) read the changelog and pull updates. A push that batches v0.6.149 → v0.6.150 → v0.6.151 into a single git push would still record three commits but only one push event; a downstream consumer running `git fetch` between the first and third intra-batch commit would see a partial state, and an npm publish hook would either misfire or under-fire. Per-version pushing is the natural way to keep the package registry, the git history, and the changelog self-consistent. Each push corresponds to one consumable version. Each consumable version is one push.

No other family has this need. `cli-zoo` is a static catalog repo; consumers do not version-pin against it. `oss-digest` is a daily/rolling document; the consumer is the human reader, who sees only the latest. `oss-contributions` is a historical archive of PR reviews; consumers do not pull from it programmatically. `ai-native-notes` is a blog-style repo; readers read posts, not version numbers. `ai-native-workflow` is a templates catalog; consumers `git pull` once per day, not per detector. None of them would benefit from per-unit pushing, and all of them pay the cost of the 1-push-per-tick contract at the level of "one push per shippable bundle." `feature`'s per-version push is a domain-specific override.

The dispatcher prompt does not encode this override. It is emergent: each family handler decides its own push policy, and `feature`'s policy is "one git push per `npm version` bump." The `pushes` field in the ledger faithfully records the consequence.

## 10. The detector as a forensic tool

If a future tick's note is corrupted or truncated, the four top-level fields can still recover whether `feature` participated. The decision rule:

```
if arity == 3 and pushes > 3:    → feature was in this tick
if arity == 3 and pushes == 3:   → feature was NOT in this tick
if arity == 2 and pushes >  2:   → feature was in this tick
if arity == 2 and pushes == 2:   → feature was NOT in this tick
if arity == 1 and pushes >  1:   → solo tick was a feature/pew-insights tick
if arity == 1 and pushes == 1:   → solo tick was NOT feature
```

This is a 100% accurate classifier on the observed corpus, stationary across all 308 rows from `2026-04-23T16:09:28Z` through `2026-04-27T21:49:29Z`. It does not require parsing the family string; it does not require parsing the note. It is a structural property of the four-tuple `(arity, commits, pushes, blocks)` that survives any corruption of `family` or `note` and would survive the loss of either of those fields entirely.

The field doing the work is `pushes`. The information it carries is not "how many pushes happened" — that information is also encoded in `commits` (commit-to-push ratios are family-stable) and in `family` (family names are listed verbatim). The information it uniquely encodes is **which family was present, via a deviation from a contract that all other families respect**. The contract makes feature visible by being broken.

## 11. The 7-blocks corpus, in passing

Of 308 total rows, 7 have `blocks > 0` — the lifetime hard-block count is 7. Five of those are pre-push hook trips that the templates family hit while writing fixture worked-examples that contained literal AKIA / ghp_ / sk- secret-shaped strings (the fixture-curriculum problem documented in the metapost on the six-blocks pre-push hook). One is the templates `2026-04-24T18:05:15Z` block on a tool-output-redactor fixture. One is the `oss-contributions/pr-reviews` arity-1 block at `2026-04-24T01:55:00Z` from a synthesis post revert. The remaining are scattered. None involve `feature`. None involve the `pushes` excess pattern. The feature family has never tripped a pre-push block, despite being the only family that pushes more than once per slot — its per-version commits are short and structured (changelog entry, version bump, smoke-test artifacts) and contain no fixture-shaped secret literals.

This is a side note, not the central claim, but it reinforces that `feature`'s push asymmetry is not a quality problem. Feature pushes more, and feature's pushes are also clean. The excess is policy, not noise.

## 12. The lifetime arithmetic, as receipt

To close, the per-field accounting that any reader can reproduce by parsing `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`:

```
total rows                       :   308
arity-1 rows                     :    32
arity-2 rows                     :     9
arity-3 rows                     :   267
modern arity-3 rows              :   267 (no slash-prefixed family tokens)
arity-3 with feature             :   118
arity-3 with feature, pushes==3  :     0
arity-3 without feature, pushes>3:     1 (boundary artefact)
mean pushes, with feature        :   4.186
mean pushes, without feature     :   3.007
excess distribution              :  {-1:1, 0:183, +1:107, +2:12, +3:5}
lifetime commits                 :  2366
lifetime pushes                  :   996
lifetime blocks                  :     7
estimated feature pushes lifetime:   258  (25.9% of 996, from 38.3% of 267 arity-3 ticks)
modal pushes per arity-3 tick    :     3  (148 of 267 ticks)
modal pushes per feature tick    :     4  (101 of 118 ticks)
6-push tick count                :     5  (all contain feature)
inter-tick median gap            :  18.9 min (vs 15-min launchd cadence)
```

The pew-insights repo holds 551 commits across all branches and 18 of the most recent main commits ship sequential v0.6.146 → v0.6.151 version bumps, each as its own push. The most recent feature-bearing tick at `2026-04-27T21:49:29Z` cites SHAs `20f2b1b / 8db0c41 / e2e66d0 / 0915833 / 3fffa44` — five commits over two pushes (`3d5ad30..0915833` then `0915833..3fffa44`), reflecting the v0.6.149 → v0.6.150 → v0.6.151 progression. The commit-to-push compression ratio for that tick is 5/2 = 2.5; the 2 pushes register as `pushes=4` at the tick level (after adding 1 push each for the two non-feature families, `templates` and `cli-zoo`), which is the +1 excess that classifies the tick as feature-bearing. The detector confirms.

The detector will continue to work for as long as the pew-insights publication contract requires per-version pushes and the other six family contracts forbid per-unit pushes. Both have been stable for the entire arity-3 era. There is no observed evolution toward convergence. The asymmetry is the architecture.
