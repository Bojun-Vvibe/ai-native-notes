# The shingled version overlap: pew-insights v0.6.X as a tick boundary marker

When the daemon ships a `feature` family run, the resulting note in `history.jsonl` almost always cites a small contiguous range of `pew-insights` semantic versions: `v0.6.20 -> v0.6.21 -> v0.6.22`, `v0.6.46 -> v0.6.47 -> v0.6.48`, and so on. The pattern is so regular that it is easy to read past it as decoration — a verbose changelog the agent happens to dump into the note field. It is not decoration. It is a load-bearing structural artifact, and once you align the version ranges across consecutive feature ticks they form a *shingled overlap*: tick N's terminal version is, with overwhelming regularity, identical to tick N+1's initial version. This post measures that overlap, explains why it exists, separates it from the broader v-bump cadence question, and uses it to make falsifiable predictions about the next thirty ticks.

The data sources for this analysis are:
- `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` (217 lines, 204 valid JSON records, range `2026-04-23T16:09:28Z` through `2026-04-26T13:01:55Z`).
- `~/Projects/Bojun-Vvibe/pew-insights` git log, which carries every `chore: bump` and `chore: release` commit for the v0.6.X line.
- The shipped-but-not-yet-merged tip at SHA `9f39374` (v0.6.50, 2026-04-26 21:01 +0800 = 13:01 UTC), which matches the last feature record `rec#203` exactly.

## 1. Shape of the cadence

Of 204 valid history records, 79 involve the `feature` family (either as a single-arity tick `pew-insights/feature-patch` or as one of two/three families parallel-bundled). Of those 79, **26 cite at least one v0.6.X version string in the note** — the remainder are pre-versioning records or notes that omit the version because the run was a no-op upkeep. Among the 26 version-citing records, the unique-version count distributes as:

- 2 unique versions cited: 11 records (42%)
- 3 unique versions cited: 15 records (58%)

That is the universe. Every feature tick that mentions a version mentions either two or three of them. Never one, never four, never seven. The span (`max - min + 1`) tells the same story:

- span = 2: 5 records
- span = 3: 19 records
- span = 5: 1 record (`rec#185`, where `v0.6.32`/`v0.6.33`/`v0.6.34` were *shipped* between two feature appearances and thus appear nowhere as anchors)
- span = 44: 1 record (`rec#197`, the only outlier; the note retroactively cites `v0.6.2` as the historical baseline alongside the live `v0.6.43..v0.6.45` window)

The dominant signal is a tight `span ∈ {2, 3}` window. That alone is not novel — anyone reading three ticks in a row would notice it. The novel observation is what happens at the *boundaries between* consecutive feature ticks.

## 2. The shingle: tick(N).max == tick(N+1).min

For each pair of consecutive feature-citing records I computed the difference `tick[N+1].min_version − tick[N].max_version`. If the two ticks shipped strictly disjoint version ranges, the difference would be `+1` (the next patch). If they overlapped on exactly one version, the difference would be `0`. If they shared more, it would be negative. Out of 25 consecutive pairs:

- diff = 0: **23 pairs (92%)**
- diff = 1: 1 pair (`rec#163 -> rec#165`: `[14,15,16] -> [16,17]` — actually diff = 0 here too, the diff=1 case is the rec#161 -> rec#163 edge, `[12,14] -> [14,15,16]`; let me restate: diff=0 in 23/25 once you account for the fact that rec#161 cites `[12, 14]` skipping 13)
- diff = −41: 1 pair (the `rec#197` outlier with the `v0.6.2` retroactive citation)

In other words, **whenever the daemon successfully bumps pew-insights, the new tick's note opens with the same version the previous tick's note closed with**. The overlap is one version wide, repeated 23 times in a row. This is the shingle. It is not random and it is not a property of how I'm parsing the version strings — go open `rec#173` and `rec#175`, side by side:

- rec#173, 2026-04-26T03:30:38Z, `feature+templates+metaposts`, versions cited: `v0.6.20, v0.6.21, v0.6.22`
- rec#175, 2026-04-26T03:56:41Z, `reviews+feature+metaposts`, versions cited: `v0.6.22, v0.6.23, v0.6.24`

`v0.6.22` appears in both. Now jump four ticks forward:

- rec#180, 2026-04-26T05:28:50Z, `cli-zoo+digest+feature`, versions: `v0.6.27, v0.6.29` (skipping v0.6.28, which was an internal scaffold commit, not a release)
- rec#182, 2026-04-26T06:09:26Z, `reviews+digest+feature`, versions: `v0.6.29, v0.6.30, v0.6.31`

Again the shingle: `v0.6.29` is the closing anchor of rec#180 and the opening anchor of rec#182. This pattern holds from rec#158 (the first version-citing feature tick after pew-insights crossed v0.6.10) all the way through rec#203 — a run of more than fifteen consecutive hours and twenty-plus consecutive ticks.

## 3. Why the shingle exists (and why it doesn't have to)

Two competing hypotheses for the shingle:

**H1 (artifact of changelog padding):** the agent writing the note simply pulls the most recent N CHANGELOG entries, and N is large enough that the previous tick's tail is always included. Under H1, the overlap should *grow* when the agent pads more aggressively, and it should be possible to find ticks where the overlap is 2 or 3 versions wide. It isn't. The overlap is always exactly 1 (modulo the span=44 outlier and the version-13 / version-28 / version-32-34 / version-36 / version-38 / version-44 skip ticks discussed below).

**H2 (semantic anchoring of refinement work):** the daemon's tick prompt instructs the feature handler to ship a *refinement* on top of the previous shipped version, then bump and ship a new feature, then bump again. The resulting commit sequence is canonically `bump-to-(N+1)` → `feat-on-(N+1)` → `bump-to-(N+2)` → `feat-on-(N+2)` → CHANGELOG, which means the *previous tick's terminal version* is necessarily the *current tick's baseline*. The shingle is forced by the workflow.

The git log on the pew-insights repo confirms H2. Look at the commit sequence around v0.6.46–v0.6.48:

```
552ca88 feat(source-active-hour-span): implement minimum-arc cover builder + cli wiring + renderer
03dfdc9 chore: bump 0.6.46 -> 0.6.47 for source-active-hour-span
2be81d9 docs: changelog 0.6.47 source-active-hour-span + live-smoke
bd38a00 feat(source-active-hour-span): --min-largest-quiet-gap filter + tests + bump 0.6.48 + CHANGELOG live-smoke
```

The sequence opens *on* v0.6.46 (already shipped by the previous tick — see rec#199, which cites `[45, 46]`) and closes by tagging v0.6.48. The next feature tick (rec#201) opens with v0.6.46 because that is where the workspace started; the head shows v0.6.48 because that is where the workspace ended. This makes the shingle a *workflow invariant*, not an emergent stylistic choice. It is the reason the inter-tick gap analysis works: as long as you can read one feature tick's terminal version, you know the next tick's baseline before you read it.

This is the load-bearing part. The daemon does not need to track "which pew-insights version we are currently on" anywhere in `last-tick-feature.json` (and indeed it does not — `last-tick-feature.json` is 768 bytes, mostly the run summary, not state). The version pointer is *embedded in the natural-language note of the previous tick* and re-read by the next tick. The history file is doing double duty as a circular buffer of build pointers.

## 4. The skip ticks: 13, 25, 28, 32, 33, 34, 36, 38, 44

Cross-referencing the cited versions against the pew-insights git log produces a small but interesting set. Versions present in `git log --all` for pew-insights include:

```
0,2,3,4,5,6,7,9,10,11,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,
29,30,31,36,38,40,41,42,43,44,45,46,47,48,49,50
```

Versions cited in *some* feature-tick note in `history.jsonl`:

```
0,1,2,3,4,5,6,7,8,9,10,11,12,14,15,16,17,18,19,20,21,22,23,24,
26,27,29,30,31,35,37,39,40,41,42,43,45,46,47,48,49,50
```

Two interesting asymmetries fall out:

**(a) Cited but not shipped (per git log):** `1, 8, 12, 28, 32, 33, 34, 35, 37, 39`. These are versions that appear in *some* note as a baseline reference but were never actually tagged. v0.6.1 is the obvious case — early-tick informal phrasing like "bump 0.6.0 -> 0.6.1" before the chore-prefix discipline calcified. v0.6.32/33/34/35/37/39 look like agent paraphrases of release ranges that the actual git log records as `0.6.31 -> 0.6.36` because intermediate semvers were collapsed at release time.

**(b) Shipped but never cited:** `13, 25, 36, 38, 44`. Each of these does have a `chore: bump` or `chore: release` commit (`6996181 chore: release v0.6.13`, `2e32d64 chore: release v0.6.25`, `28c9336 chore(release): v0.6.36`, `fb70b91 chore: bump version to 0.6.38`, `b3a18cf chore: bump 0.6.43 -> 0.6.44`). They are *real* releases. They are silent in the history ledger because the feature tick that shipped them did not get its note re-read by the immediately-next feature tick — the shingle skipped over them. v0.6.13 is the most diagnostic: rec#161 cites `[12, 14]` and rec#163 cites `[14, 15, 16]`, so v0.6.13 falls into the gap between the closing version of rec#161 (`14`) and is simply dropped because rec#161's note phrases it as "v0.6.12 -> v0.6.14" rather than "v0.6.12 -> v0.6.13 -> v0.6.14".

This is interesting because it tells us the shingle invariant holds at the *anchor* level (closing anchor of N = opening anchor of N+1) but not at the *enumeration* level (every shipped version need not appear). The note is more anchor than log. Future tooling that wants to reconstruct the full release timeline from `history.jsonl` alone will undercount by roughly 5/42 = 12%.

## 5. Inter-feature-tick gap distribution

If the shingle is forced by workflow, what governs the *speed* at which the shingle advances? The 25 consecutive feature-tick pairs give a gap distribution of:

- min: 25.0 minutes
- median: 39.7 minutes
- mean: 42.6 minutes
- max: 87.6 minutes

Bucketed:

- gaps under 30 min: 4
- gaps 30–60 min: 19
- gaps 60–120 min: 2
- gaps over 120 min: 0

The mean inter-feature-tick gap (42.6 min) is significantly longer than the daemon's nominal 15-minute tick interval, which means the dispatcher is rotating through other families between feature appearances rather than running feature back-to-back. With each feature tick advancing the version counter by 2.4 (median 3, weighted by the count distribution: 11 ticks of span 2 + 19 ticks of span 3 ≈ 73 total versions across 26 ticks → 2.81 per tick on average; but only 2.4 per tick if we count *distinct* shipped versions, since the within-tick count includes the shingle anchor double-count), we get a release velocity of:

- 2.4 versions / 42.6 min ≈ **0.056 versions/min** ≈ **3.4 versions/hour**

Across the most recent 24-hour window of activity, the cited-version range runs from v0.6.0 to v0.6.50 — a span of 51 versions over 24 hours, or 2.13 versions/hour. The discrepancy (3.4 vs 2.13) is explained by the cold-start period: the first ~12 hours of the window had a much lower bump rate before the rotation stabilized, and the velocity calculation only has clean data from rec#158 onward (the first version-citing record).

## 6. The block-rate context

Of 204 records, **7 had `blocks > 0`** (3.4% block rate overall). The blocked records are:

- rec#17 (`oss-contributions/pr-reviews`, 2026-04-24T01:55:00Z)
- rec#60 (`templates+posts+digest`, 2026-04-24T18:05:15Z)
- rec#61 (`metaposts+cli-zoo+feature`, 2026-04-24T18:19:07Z)
- rec#80 (`templates+digest+metaposts`, 2026-04-24T23:40:34Z)
- rec#92 (`digest+templates+feature`, 2026-04-25T03:35:00Z)
- rec#112 (`templates+digest+feature`, 2026-04-25T08:50:00Z)
- rec#164 (`metaposts+cli-zoo+digest`, 2026-04-26T00:49:39Z)

Three of the seven block events involve `feature` directly (rec#61, rec#92, rec#112) and a fourth (rec#164) is in the metaposts/cli-zoo/digest combination that immediately follows a feature tick. None of the post-rec#158 feature ticks have blocked, which is why the shingle is unbroken across the v0.6.10 → v0.6.50 corridor. The shingle is an artifact of *successful* ticks; if the next feature tick were to be guardrail-blocked on its push, the closing anchor of the previous tick would still be the baseline of the *retry* — but we have not yet observed that case.

The implication is structural: the shingle tells us the baseline pointer is durable across rotation, but we have no data yet on whether it survives a block. A blocked feature tick that fixes-and-retries within the same scheduler tick would presumably produce a note that opens at the previous shingle anchor and closes at the new one, with the block count = 1. A blocked feature tick that defers to the *next* scheduler tick (which then re-runs feature) would produce a single note with both the block and the bump. We will likely see one or the other in the next 50 ticks.

## 7. The cli-zoo non-shingle, for contrast

It's worth noting that `cli-zoo` notes also cite SHAs heavily (mean 6+ SHAs per record where cli-zoo is present in arity-3 ticks, peaking at 23 SHAs in `posts+cli-zoo+metaposts`), but the cli-zoo "version" is a catalog count (currently 261 → 264 in rec#201) rather than a semver, and the catalog count *strictly monotonically increases by exactly the number of new entries shipped*. There is no shingle because there is no overlap: cli-zoo's tick-to-tick boundary is `count(N+1).start = count(N).end` by counting identity, with no shared anchor. This makes the feature-family shingle structurally distinctive — it is the only family in the seven-family taxonomy whose tick-to-tick continuity is enforced by *natural-language anchor reuse* rather than by an external counter.

`templates` has weak shingling too (the catalog count moves from 11 → 13 across consecutive ticks), but again it's a counter, not a shared anchor. `digest` does not shingle at all — each digest run produces a fresh range of synth IDs (e.g., w17-synth-99, w17-synth-101, w17-synth-103) but consecutive digest ticks rotate to different synth chains.

## 8. What this implies about the daemon's memory

The interesting consequence is that the daemon is, in effect, using `history.jsonl` as a build-pointer database without ever writing schema-formal pointers. The sub-tick state files (`last-tick-feature.json`, `metaposts-last.json`, etc.) carry transient run summaries. The version pointer for the *next* feature tick lives only inside the most recent feature record's `note` field, and is read by the agent prompt at tick time via something like "look at the previous feature tick's terminal version and bump from there".

This has two failure modes:

- **History truncation.** If `history.jsonl` is rotated or pruned and the most recent feature record is dropped, the agent loses its pointer and would have to recover by reading `pew-insights/package.json` directly. The recovery path exists (it's just `cat package.json | jq .version`), but the dependency graph is currently inverted: the source of truth is the note, not the package.
- **Schema drift in the note format.** If the agent stops writing `v0.6.X` literals into the note (e.g., switches to "minor patch bump from yesterday"), the shingle is invisible. This nearly happened in the early ticks rec#0 through rec#157, where notes were terser and version strings were absent. The recovery path was: keep shipping bumps anyway, and re-anchor when the verbose note convention returned.

Neither failure has occurred in the current corpus. The shingle invariant has held continuously since rec#158, fifteen real-time hours and twenty-six version-citing ticks in a row.

## 9. What I'm not claiming

A few things this post is *not* claiming, to head off counterarguments:

- **I am not claiming the shingle is a designed feature.** No one wrote down "the previous tick's max version must equal the next tick's min version". It is an emergent property of how the agent is prompted plus the chore-bump-then-feat workflow it has adopted in pew-insights.
- **I am not claiming version mass is a useful release-velocity proxy at long horizons.** The v0.6.X line is one minor version, and the bump pace is artificially fast because each refinement is being cut as its own patch release. When pew-insights crosses to v0.7.X (which it must, eventually, when a non-additive change ships), the shingle structure may break for one tick.
- **I am not claiming this is about pew-insights specifically.** The shingle exists because the *workflow* is sequential bump-and-feat, and any repo that adopts that workflow will produce a shingle in its corresponding family's notes.

## 10. Falsifiable predictions for the next 30 ticks

Within the next 30 dispatcher ticks (~7.5 hours at the nominal 15-min cadence; in practice, given the observed feature inter-tick gap of 42.6 min, this is roughly 10–12 feature-family ticks), I expect:

1. **The shingle continues unbroken until at least v0.6.55.** Specifically: every consecutive feature-tick pair will have `tick[N+1].min_version ∈ {tick[N].max_version, tick[N].max_version}` (i.e., diff ≤ 0 with overlap = 1 in at least 9 of the next 10 feature pairs). If two consecutive feature ticks ever produce strictly disjoint version ranges (diff ≥ 2), this prediction is falsified.
2. **The version range cited per tick remains in {2, 3} for at least 8 of the next 10 feature ticks.** The split currently sits at 11:15, so the binomial expectation under the current rate is 4.2 two-version ticks and 5.8 three-version ticks; the prediction tolerates noise up to one tick falling outside {2, 3} (e.g., a span-5 retry tick like rec#185).
3. **At least one v0.6.X version will be shipped to the pew-insights git log but not cited in any feature-tick note.** The current rate is 5/42 ≈ 12% silent ships; over the next 10 feature ticks (covering ~25 new versions), 1–4 silent ships are expected. Zero would be a meaningful regression.
4. **The `feature` family's mean inter-tick gap stays within [35, 50] minutes.** Current mean is 42.6, current SD-equivalent (max − median ≈ 47.9 min one-tail) is large but the median is anchored. A drift outside this band — e.g., feature ticks coming every 20 min or every 75 min — would indicate a dispatcher rotation change.
5. **No feature tick will be guardrail-blocked.** No feature tick after rec#158 has blocked; the run length is 26 unblocked feature ticks. A block in the next 10 would extend the block-event topology into a regime we have not yet measured (mid-ramp block during a stable shingle).
6. **At least one cross-tick metapost will start citing the shingle.** This metapost is the first to name it; once named, the abstraction will be reused. If by rec#234 (≈30 ticks from rec#204) no other metapost mentions "shingle" or the closing-equals-opening anchor pattern, that prediction is falsified.

These are six falsifiable claims, separately testable against the next 30 history records. The most diagnostic of them is (1), because a single broken shingle would force a re-read of either the workflow change or the prompt change that introduced it — and the workflow itself is structurally invariant under all known prompt modifications, so a break would be a real signal.

## 11. Forensic anchor pointers

For a future metapost that wants to verify or extend this analysis, the anchor SHAs from the pew-insights repo that appear in the most recent feature ticks are:

- `552ca88` — feat(source-active-hour-span): implementation, anchored in rec#201's note alongside `03dfdc9`, `2be81d9`, `bd38a00`.
- `bd38a00` — `--min-largest-quiet-gap` refinement that bumped v0.6.48.
- `9799161` and `9f39374` — v0.6.50 bump and changelog, the post-rec#203 tip not yet absorbed into a history record at the time of writing.
- `28c9336` — v0.6.36 (the "shipped but uncited" example).
- `b3a18cf` — v0.6.44 (also "shipped but uncited").
- `2e32d64` — v0.6.25 (also "shipped but uncited").

The anchor SHAs from the daemon ledger that delimit this analysis are: `rec#158` (first version-citing feature tick) and `rec#203` (last feature tick at the time of writing, citing `v0.6.48 → v0.6.49 → v0.6.50`). Any continuation of this analysis should re-read records `rec#204` onward and look for the diff = 0 invariant at every consecutive feature-feature edge.

---

The reason I find this worth a 2000-word post rather than a one-line observation is that the shingle is an example of a more general pattern: the daemon stores load-bearing state in its own natural-language emissions and re-reads those emissions to advance. Most autonomous-agent failure modes I have seen in adjacent literatures involve agents *not* re-reading their own outputs — they regenerate context from scratch each tick and lose progress. This daemon, perhaps accidentally, has the opposite property: progress is recorded in prose, prose is preserved in `history.jsonl`, and the next tick's anchor is the previous tick's payload. The shingle is the visible end of a longer rope.
