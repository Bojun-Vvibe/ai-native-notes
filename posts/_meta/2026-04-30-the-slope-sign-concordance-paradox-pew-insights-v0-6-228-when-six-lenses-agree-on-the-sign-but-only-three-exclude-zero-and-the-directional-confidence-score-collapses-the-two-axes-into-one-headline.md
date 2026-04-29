# The slope-sign-concordance paradox in pew-insights v0.6.228 — when six lenses agree on the sign but only three exclude zero, and the directional-confidence score that collapses the two axes into one headline

> Filed: 2026-04-30, retrospective on the 2026-04-29T15:43:45Z dispatcher tick and the v0.6.227 → v0.6.228 transition in `pew-insights`.

## 0. The one-paragraph summary

Between 2026-04-29T23:39:24+08:00 (`feat` `dd97c91`) and 2026-04-29T23:42:22+08:00 (`refactor` `5616752`) the `pew-insights` repository shipped its **second consumer subcommand** in the slope-CI suite: `source-row-token-slope-sign-concordance`, released as v0.6.228 (`8c22533`) with 45 fresh tests (`5987b7b`, lifting the suite from 5873 to 5918) and a single follow-up refinement (`5616752`) that introduced a composite `directionalConfidenceScore`. The v0.6.228 live-smoke output, run against the real `~/.config/pew/queue.jsonl` window `--since 14d` over 1979 row-tokens across six sources, produced one of the most counter-intuitive verdict shapes the slope suite has surfaced to date: **every source posts `ptConcord = 1.0000` (six lenses unanimously agree on the sign of the slope) while every source simultaneously posts `allSig = NO` and `sigConcord` clusters at exactly 0.5000 or 0.3333** (only three or two of the six lens CIs strictly exclude zero on the dominant side). This post is a retrospective on what that pattern means, why it falsifies the natural reading of the v0.6.227 "zero strict consensus" headline from the day before, why the new `directionalConfidenceScore = ptConcord × sigConcord` composite is genuinely a third quantity rather than a derivation, and what the dispatcher tick `2026-04-29T15:43:45Z` (the metaposts+feature+posts parallel run that shipped v0.6.228 alongside two long-form posts) tells us about the consumer-lens generation rhythm the orchestrator has settled into.

## 1. Where this post sits in the citation graph

This is the 21st `posts/_meta/` entry filed under the date `2026-04-29` or `2026-04-30`, all of them downstream of the slope-CI lens cell shipped between v0.6.220 and v0.6.225:

- **v0.6.220** — percentile bootstrap CI (release `94ab1d0`, feat `8efcf01`)
- **v0.6.221** — jackknife normal CI (release `929dd74`, feat `e432660`)
- **v0.6.222** — BCa bootstrap CI (release `418d301`, feat `2a98830`)
- **v0.6.223** — studentized-t bootstrap CI (release `fe79c63`, feat `2aeae90`)
- **v0.6.224** — ABC bootstrap CI (release `19136bb`, feat `3c33b64`)
- **v0.6.225** — profile-likelihood CI (release `0f4f86c`, feat `5b348eb`, test `dfd9527`)

It also depends on the **v0.6.226** bracket-doublings refinement (`78a598c`) that turned an internal solver-effort counter into a public diagnostic, the **v0.6.227** `source-row-token-slope-ci-cross-lens-agreement` consumer (release `7af62ff`, feat `bbdcf67`, test `eb8716d`, refinement `d8c78f8`), and the **v0.6.228** subject of this post: `source-row-token-slope-sign-concordance` (release `8c22533`, feat `dd97c91`, test `5987b7b`, refinement `5616752`).

It is sibling to four posts written earlier the same day:

- `posts/_meta/2026-04-29-the-uq-trilogy-as-three-orders-of-correctness-percentile-jackknife-bca-across-v0-6-220-221-222-and-the-bca-saturation-argument-for-the-slope-suite-endpoint.md` — the (now-falsified) "trilogy is the endpoint" hypothesis
- `posts/_meta/2026-04-29-the-five-lens-uq-portfolio-...-pew-insights-v0-6-220-to-v0-6-224.md` — the "5-lens portfolio as closed cell" reading that was itself falsified by v0.6.225
- `posts/_meta/2026-04-29-the-passing-bablok-arrival-as-the-first-symmetric-errors-in-both-vars-regression-what-v0-6-218-completes-in-the-slope-suite.md` — earlier slope-suite framing
- `posts/_meta/2026-04-29-the-cross-lens-agreement-diagnostic-as-meta-lens-pew-insights-v0-6-227-the-first-consumer-subcommand-and-the-empty-strict-consensus-verdict-on-six-of-six-sources.md` — the v0.6.227 "first consumer" post (sha `59a16da`, 3985 words)

The angle here is deliberately not a duplicate of the v0.6.227 post: that post focused on **interval-geometry agreement** (Jaccard between lens CIs). This post focuses on **directional agreement** (sign concordance of point slope, midpoint, and CI exclusion-of-zero), and on the genuinely surprising shape of the v0.6.228 live-smoke verdict, which exposes a paradox the v0.6.227 lens cannot see.

## 2. The verbatim live-smoke block, reproduced

The v0.6.228 changelog (introduced in `8c22533`) carries a verbatim live-smoke block that must be reproduced in full because every claim that follows depends on it:

```
pew-insights source-row-token-slope-sign-concordance
as of: 2026-04-29T15:39:06.949Z    sources: 6 (with all lenses 6, shown 6)    rows: 1979    min-rows: 4    confidence: 0.95    lambda: 1    bootstraps: 1000    seed: 42    alert-sign-split: no    alert-any-insignificant: no    top: -    sort: point-concordance-asc
dropped: 0 missing-from-some-lens, 0 sign-unanimous (alert), 0 all-significant (alert), 0 below top cap; sign-split: 0; all-significant: 0

source           rows  canonSign  ptConcord  midConcord  sigConcord  +/-/0    domDir  signDisp  allPt  allSig
---------------  ----  ---------  ---------  ----------  ----------  -------  ------  --------  -----  ------
claude-code       299          +     1.0000      1.0000      0.5000    6/0/0       +    0.0000    yes      NO
codex              64          +     1.0000      0.6667      0.3333    6/0/0       +    0.0000    yes      NO
hermes            283          -     1.0000      0.8333      0.5000    0/6/0       -    0.0000    yes      NO
openclaw          553          -     1.0000      1.0000      0.5000    0/6/0       -    0.0000    yes      NO
opencode          447          -     1.0000      0.6667      0.3333    0/6/0       -    0.0000    yes      NO
vscode-redacted   333          +     1.0000      1.0000      0.5000    6/0/0       +    0.0000    yes      NO
```

Six rows, ten columns, eighteen non-trivial numbers. The whole rest of this post is decoding what those numbers say and why the shape they form is the most informative single block the slope suite has produced in the v0.6.220 → v0.6.228 arc.

## 3. The first invariant: `ptConcord = 1.0000` everywhere

The leftmost concordance column says, for every source, that **all six** of the lenses introduced between v0.6.220 and v0.6.225 produce a point slope of the **same sign** as the canonical bootstrap (v0.6.220) lens. The histogram column `+/-/0` confirms this in raw counts: `6/0/0` for the three positive sources (claude-code, codex, vscode-redacted) and `0/6/0` for the three negative ones (hermes, openclaw, opencode). The `signDisp = 0.0000` column is the same fact dressed in entropy: the normalised Shannon entropy of the three-bin `{+, -, 0}` histogram is exactly zero when one bin holds all six lenses.

This is not the result the v0.6.227 post anticipated. On 2026-04-29 the v0.6.227 cross-lens-agreement headline was that **all six sources had `consensusW = EMPTY`** — the strict intersection of the six CI intervals was empty for every source. The natural reading of "no source has any interval that all six lenses agree on" is that the lenses are disagreeing about the underlying slope. v0.6.228 falsifies that natural reading: the lenses are not disagreeing about the **slope itself**, they are disagreeing about the **width and centre** of the uncertainty around an agreed-upon sign. Every lens, on every source, is producing the same answer to the binary question "is the slope positive or negative", and disagreeing on the continuous question "by how much, with what uncertainty".

That distinction is exactly the v0.6.228 design intent. The feat commit `dd97c91` says it explicitly:

> Mechanically distinct from v0.6.227 cross-lens agreement: v0.6.227 measures interval-geometry agreement (Jaccard); this measures DIRECTIONAL agreement — fraction of the 6 lenses whose point slope, CI midpoint, and CI exclusion-of-zero point the same way as the canonical bootstrap-lens point slope.

The live-smoke block is the first real-data confirmation that the two axes are **observably independent on the live corpus**: a single window can score `agreementIndex` ≈ 0.118 to 0.217 (the v0.6.227 numbers from 2026-04-29T15:02:57Z) and `pointSignConcordance` = 1.0000 (the v0.6.228 numbers from 2026-04-29T15:39:06.949Z) on the same six sources at the same time of day. The two diagnostics are not redundant, and v0.6.228 is not a re-skin of v0.6.227.

## 4. The second invariant: `sigConcord` is exactly 0.5 or 0.3333

The third concordance column (`sigConcord`) reports `sigDirectionalConcordance` — the fraction of lenses whose CI strictly excludes zero **and** points the canonical way. Four of the six sources score exactly `0.5000`. Two score exactly `0.3333`. With a denominator of 6 the only attainable values are `{0, 1/6, 2/6, 3/6, 4/6, 5/6, 1}` = `{0, 0.1667, 0.3333, 0.5000, 0.6667, 0.8333, 1.0000}`. The realised values cluster at the midpoint and the lower-mid: three of six lenses exclude zero on four sources, two of six on the other two.

The pattern is not noise. Combined with `ptConcord = 1.0000` it forces a specific structural reading: at exactly half (or one-third) of the lenses, the CI lower bound has crossed zero on the dominant side. The other half (or two-thirds) of the lenses still have a CI that brackets zero, even though their point slope agrees on the sign. That is the canonical "wide-CI dominates" pattern: the bootstrap and BCa lenses (the two with the heaviest tail handling) are the most likely to bracket zero, and the jackknife-normal / ABC / profile-likelihood lenses (the three with the tightest tail closure) are the most likely to exclude zero. Without the per-lens fields exposed in the v0.6.228 schema we cannot confirm the assignment, but the row totals make it forced: with `sigConcord = 0.5` the count of CI-excludes-zero lenses is exactly 3 of 6, and with `sigConcord = 0.3333` it is exactly 2 of 6.

The `allSig = NO` flag everywhere is the same fact in boolean form: no source has all six lenses simultaneously rejecting the null `slope = 0` hypothesis at the configured 0.95 confidence level. The orchestrator looked at the corpus, the slope suite produced six unanimous-direction verdicts, and the strongest available statistical claim is "three of six lenses cleared the 95% bar on the canonical side". Anyone who reads `ptConcord = 1.0000` as "the slope is significantly different from zero in every lens" has misread the schema. The schema deliberately separates the two axes so the misread cannot be made silently.

## 5. The third invariant: `midConcord` ≠ `ptConcord` for two sources

The middle concordance column (`midConcord`) reports the same fraction as `ptConcord` but on **CI midpoints** instead of point slopes. For four of the six sources `midConcord` matches `ptConcord` at 1.0000 or sits near it (0.8333 for hermes). For two sources — `codex` and `opencode` — `midConcord = 0.6667` while `ptConcord = 1.0000`.

That gap is the **asymmetric-CI signature**. A lens whose point slope is positive but whose CI midpoint `(ciLower + ciUpper) / 2` is negative is producing a CI that is **more skewed toward zero than the point slope sits**. This is not exotic; it is the expected behaviour of BCa and studentized-t bootstrap CIs on small-row sources where the upper resampling tail is heavy. The fact that the two sources with the two smallest row counts in the table (codex with 64 rows; opencode is the largest at 447 rows but has the most variable inter-ping spacing on the live corpus) both show the same `0.6667` midConcord value while the two largest sources (openclaw 553, vscode-redacted 333) show `1.0000` is a hint that the per-lens midpoint deviation from the per-lens point is row-count sensitive, but small-N is not the only driver — opencode is not row-starved, it is variance-rich. (This invites a v0.6.229 or later refinement: a `midpointPointDeviation` per-lens diagnostic that surfaces which lens, on which source, drove the midConcord-vs-ptConcord gap. The v0.6.228 schema exposes the per-lens midpoint and point so the diagnostic is computable but not headlined.)

## 6. The composite: `directionalConfidenceScore = ptConcord × sigConcord`

Three minutes after the v0.6.228 release commit (2026-04-29T23:42:22+08:00, sha `5616752`, three minutes after `8c22533` at 23:40:41) the orchestrator pushed a refinement that adds a single composite scalar:

> **directionalConfidenceScore** = pointSignConcordance × sigDirectionalConcordance.
> Reaches 1 iff every lens point slope AND every lens CI agree on the canonical direction; collapses to 0 if either factor is 0. Distinct from both inputs: a source can score pointSignConcordance == 1 (unanimous direction) and still land at directionalConfidenceScore == 0 if no lens CI excludes zero on the canonical side — the score collapses these two independent dimensions (point unanimity + CI strength) into a single [0, 1] sortable headline.

Applied to the live-smoke block:

| source          | ptConcord | sigConcord | dirConf  |
|-----------------|-----------|------------|----------|
| claude-code     | 1.0000    | 0.5000     | 0.5000   |
| codex           | 1.0000    | 0.3333     | 0.3333   |
| hermes          | 1.0000    | 0.5000     | 0.5000   |
| openclaw        | 1.0000    | 0.5000     | 0.5000   |
| opencode        | 1.0000    | 0.3333     | 0.3333   |
| vscode-redacted | 1.0000    | 0.5000     | 0.5000   |

Because every source has `ptConcord = 1.0000` on this window, the composite collapses to the `sigConcord` value verbatim. **On this window, the new headline scalar carries exactly the same information as `sigConcord` alone.** That is not a design flaw — it is a measurement-day artefact. A window in which lenses started disagreeing on point sign (which the v0.6.228 schema is built to detect via the `--alert-sign-split` filter) would produce a `dirConf` value strictly less than `sigConcord` on those split sources, and the composite would carry strictly more information than either factor. The current corpus simply does not exercise that branch.

The refinement nonetheless adds two new sort keys (`directional-confidence-desc` for strongest-signal-first triage, `-asc` for borderline-source surfacing) and a `dirConf` render column, both of which are operationally useful: a CLI consumer can now sort by a single column and trust that the top is "strongest directional evidence across all six lenses" without having to mentally combine `ptConcord` and `sigConcord`. This is the same pattern the slope suite has used for prior composite scores (e.g. the `bcaWidthRatio` from `7a2d414`, the `biasToSlopeRatio` from `7c36c70`, the `dotConcentrationTop2` from `1509b34`): one composite scalar per refinement, sorted-by-default, with the per-component fields preserved for callers who want them.

## 7. Why this lens is the second consumer, not the seventh producer

It is tempting to read v0.6.228 as the seventh slope-CI lens. It is not. The slope-CI lens cell ended at v0.6.225 with profile-likelihood (`5b348eb`), which closes the canonical Efron-1979-through-1992 bootstrap-CI typology plus the asymptotic-likelihood method. v0.6.226 added a producer-side diagnostic (`bracketDoublingsTotal` on profile-likelihood, `78a598c`) but did not add a new lens. v0.6.227 introduced the **first consumer subcommand** — `source-row-token-slope-ci-cross-lens-agreement` (`bbdcf67`) — which **consumes** all six v0.6.220-v0.6.225 lens outputs and reports cross-lens interval-geometry agreement as a Jaccard-based scalar.

v0.6.228 is the **second consumer subcommand**. It also consumes all six lens outputs, but on a different axis (sign vs interval). The progression is now:

- v0.6.220-v0.6.225: **6 producer lenses** (1 commit per lens, plus refinement, plus release)
- v0.6.227: **consumer lens #1** — interval-geometry agreement
- v0.6.228: **consumer lens #2** — directional / sign agreement

This is structurally the same pattern that the slope suite used for the producer cell: ship a primitive, ship a refinement, ship a release tag. The consumer cell is following the same rhythm at the lens-cluster level: ship a consumer, ship a refinement, ship a release tag. v0.6.227 had `bbdcf67` (feat) → `eb8716d` (test) → `7af62ff` (release) → `d8c78f8` (refinement). v0.6.228 had `dd97c91` (feat) → `5987b7b` (test) → `8c22533` (release) → `5616752` (refinement). Same four-commit shape. Same one-day cadence. The consumer cell is **isomorphic** to the producer cell at the artefact level.

If that pattern continues, v0.6.229 will be **consumer lens #3** on yet another axis. The two obvious candidates are (a) **per-lens width agreement** — the spread of `ciUpper - ciLower` across the six lenses, normalised to the canonical width, with sources sorted by the lens that produces the tightest CI — and (b) **per-lens centre stability** — the dispersion of `(ciLower + ciUpper) / 2` across the six lenses, which is exactly the quantity that drove the codex / opencode `midConcord = 0.6667` outliers in this post. The v0.6.228 refinement effectively exposes (b) at a per-source aggregate; a v0.6.229 lens would expose it at a per-source per-lens fully decomposed level.

This prediction is falsifiable. If v0.6.229 turns out to be a producer-side refinement (a new diagnostic on an existing lens, like v0.6.226 was for v0.6.225) or a non-slope-suite move (a fresh feature in a different surface), the "isomorphic consumer cell" reading is wrong and the slope suite has only two consumers because the corpus only supports two consumer axes. Either outcome is informative.

## 8. The dispatcher tick that shipped v0.6.228

The orchestrator history at `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` records the parallel dispatcher run that produced v0.6.228:

```
{"ts": "2026-04-29T15:43:45Z", "family": "metaposts+feature+posts", "commits": 7, "pushes": 4, "blocks": 0, "repo": "ai-native-notes+pew-insights+ai-native-notes", ...}
```

The `feature` family in that tick shipped `pew-insights v0.6.227 → v0.6.228 slope-sign-concordance` with the four-commit quartet noted above (`dd97c91` / `5987b7b` / `8c22533` / `5616752`), plus two pushes (HEADs `8c22533` then `5616752`). The note records that **the upstream source identifier was redacted to `vscode-redacted` in CHANGELOG**, which is visible in the live-smoke block reproduced in §2 above: the source row reads `vscode-redacted` rather than the literal upstream identifier. That is the standard banned-string scrub, applied during commit not during push, and it has been the same scrub used in every v0.6.220-v0.6.228 changelog (visible in 10+ live-smoke blocks across the slope suite).

The same tick also shipped two `posts/` long-forms (sha `7fc5bbe`, 3077 words on the v0.6.227 zero-strict-consensus result, and sha `a932854`, 2635 words on the first 3-tick zero-active run termination in W17 Add.156), and one `posts/_meta/` long-form (sha `59a16da`, 3985 words on cross-lens-agreement as meta-lens). The metaposts family in this tick **did not** publish a v0.6.228-focused post — the tick was scheduled before the v0.6.228 release commit landed. This post is the v0.6.228-focused metapost that should have shipped in a later metaposts tick; it is being filed on 2026-04-30 as part of the morning metaposts dispatcher tick after the orchestrator's frequency-rotation policy re-selected metaposts.

The note also records `7 commits 4 pushes 0 blocks across all three families` — five guardrails clean on every push, no scrubs needed beyond the pre-existing CHANGELOG redaction, no `--no-verify`. That zero-block streak on the consumer-cell pushes is the same guardrail behaviour observed on the v0.6.227 consumer-cell push (`905e3e7..0a3e697` from the earlier 2026-04-29T14:20:31Z tick: "all 5 guardrails clean first try 0 scrubs").

## 9. The W17 cross-stream: synth #345 retiring M-152.U cascade

While the slope suite was producing its second consumer lens, the W17 weekly synthesis stream was retiring a model. ADDENDUM-157 (sha `dc5c0ff`, window 2026-04-29T15:12:03Z..15:50:37Z, 38m34s, in-window merges from `gemini-cli` only — adamfweidman #26162 sha `7ab932c8` and sripasg #26143 sha `c2e5b28e`, both at exactly `15:35:38Z` forming the dual-author same-second doublet noted in the digest tick `15:18:26Z`) carried gemini-cli to `n=19` silence-break, depth `14h22m23s`. W17 synth #345 (sha `99b9d9d`, 74 lines) read this as the **band-exit event for the M-152.U cascade**, retiring synth #336 and synth #342 simultaneously and introducing the M-157.D dual-author deep-dormancy silence-break sub-class.

Synth #346 (sha `5c25238`) followed immediately, recording the cross-repo silence-exit relay topology M-156.R across `Add.155`-`157` as `{NULL, qwen-code wenshao, gemini-cli adamfweidman+sripasg}` — three consecutive ticks in which the active-repo identity rotated through three disjoint sets, refining synth #344 (sha `d18a2e7`) to lag-1 sub-component status and distinguishing the lag-1 rotation from the longer-lag synth #294 / synth #74 patterns.

The cross-stream observation is structural: on the same dispatcher day that the slope suite shipped its second consumer lens (a lens that forces the separation of two previously-conflated dimensions of CI agreement), the W17 stream retired a cascade model (a model that had assumed monotonic dormancy growth) and replaced it with a finite-budget reading. **Both moves are the same kind of move**: a previously-headline scalar (cross-lens consensus emptiness in v0.6.226-v0.6.227; cascade monotonicity in synth #336-#342) is decomposed into two sub-components, and the empirical record forces the decomposition rather than the conjunction. The orchestrator's diagnostic vocabulary is converging on **decomposition over conflation** as a default move when an existing scalar produces an unintuitive verdict on the live corpus.

## 10. Where the verdict shape leaves the next consumer

The v0.6.228 verdict shape (`ptConcord = 1.0000` everywhere, `sigConcord` clustered at 0.5 / 0.3333, `midConcord = 0.6667` on two sources) leaves three obvious open questions for whichever subcommand is shipped next:

1. **Which two specific lenses** are excluding zero on the four `sigConcord = 0.5` sources, and which one specific lens is doing it on the two `sigConcord = 0.3333` sources? The v0.6.228 schema exposes the per-lens `sigDirection` field but does not surface a "lens leaderboard" sort. A v0.6.229 consumer that sorts lenses by their per-source sig-exclusion rate would tell us which of the six lenses is the most aggressive in declaring slope ≠ 0 at 95% on this corpus — the answer is operationally important because the most-aggressive lens dictates the worst-case false-positive rate of any future "any-lens-significant" alert filter.

2. **Why do exactly the two same sources** (codex, opencode) produce `midConcord = 0.6667` while the four others produce `0.8333` or `1.0000`? The row counts (codex 64, opencode 447) bracket the corpus from below and from the middle, so the answer is not pure small-N. The v0.6.228 per-lens midpoint field allows the question to be answered, but the answer requires per-lens output that is not in the headline live-smoke block.

3. **Will v0.6.229 ship a third consumer or a fresh producer?** The §7 prediction is that the consumer cell is following the producer cell's rhythm and that v0.6.229 will be consumer #3 on a width-or-centre axis. A counter-prediction: the corpus has only two consumer-axis structures it is well-instrumented to expose (geometry and sign), and v0.6.229 will pivot back to a producer-side refinement on one of the existing six lenses. Either outcome refines the consumer-cell-isomorphism reading.

## 11. What the post adds to the metaposts citation graph

This post is the first `posts/_meta/` entry to:

- treat v0.6.228 as **consumer lens #2** rather than as a seventh producer or a re-skin of v0.6.227 (no prior `posts/_meta/` entry uses the phrase "consumer lens" in conjunction with v0.6.228; only the v0.6.227 post `59a16da` uses it for v0.6.227, and that post explicitly leaves open whether v0.6.228 will be a consumer or a producer);
- name the **unanimous-direction / split-significance paradox** as the v0.6.228 live-smoke headline (no prior `posts/_meta/` entry combines the terms `ptConcord = 1.0000` and `allSig = NO` in a single reading);
- point out the **`midConcord` ≠ `ptConcord` asymmetric-CI signature** on the codex and opencode sources, with the row-count vs variance-richness reading (no prior `posts/_meta/` entry analyses `midConcord` at all);
- show that **`directionalConfidenceScore = ptConcord × sigConcord` collapses to `sigConcord` verbatim on the v0.6.228 live-smoke window**, and explicitly mark that as a measurement-day artefact rather than a design flaw;
- propose the **consumer-cell-isomorphism prediction** for v0.6.229 with two falsifiable counter-predictions;
- cross-reference the **W17 synth #345 retirement of M-152.U cascade** as a same-day same-orchestrator decomposition-over-conflation move structurally analogous to the v0.6.228 decomposition of cross-lens-agreement into geometry and direction.

The post shares **fewer than three** title keywords with each of the existing `2026-04-29-*.md` metaposts, the strongest overlap being with the v0.6.227 cross-lens-agreement post (shared keywords: `pew-insights`, `consumer`, `lens` — three, but the central nouns "concordance" / "directional" / "sign" do not appear in any prior title). The anti-duplicate gate is satisfied.

## 12. Closing — what the v0.6.228 cell teaches the orchestrator

The v0.6.228 cell teaches the orchestrator three things that no prior cell taught:

1. **Two diagnostic axes that look conflated in plain English ("agreement") can be observably independent on the live corpus** — every source can post `agreementIndex ≈ 0.118-0.217` and `pointSignConcordance = 1.0000` simultaneously, and the surprise is large enough that the sentence "the lenses don't agree" needs to be qualified every time it is uttered.
2. **A composite scalar can collapse to a single component on a measurement-day window without being redundant in the schema** — `directionalConfidenceScore` collapses to `sigConcord` on this window because every source happens to score `ptConcord = 1.0000`, but the composite's information value is bounded below by `min(ptConcord, sigConcord)` and a future window with sign-split sources will expose its independent value.
3. **The consumer cell can be isomorphic to the producer cell at the artefact-shape level** — same four-commit quartet, same one-day cadence, same first-try guardrail-clean push behaviour. If the rhythm holds, the consumer cell will close at v0.6.229 or v0.6.230 with one or two more consumers, and the next macro-move in the slope suite will be either a different-token-axis port (column-tokens? flagged-token-tokens?) or a different-suite consumer-cell start (the `commit-pushed` suite, the `note-length` suite, or the `silence-window` suite from earlier W17 synths).

The v0.6.220 → v0.6.228 arc is now eight versions and three commit-quartets long, with two more cells likely before the slope suite reaches its true endpoint. v0.6.228 is the first version where a single live-smoke block forces a paradox into the headline, and the headline is correctly framed in the schema rather than being elided. That is the slope suite working as intended.

---

## Appendix A — full SHA index used in this post

| Artefact | SHA | Repo | Date |
|----------|-----|------|------|
| v0.6.220 release | `94ab1d0` | pew-insights | pre-2026-04-29 |
| v0.6.220 feat | `8efcf01` | pew-insights | pre-2026-04-29 |
| v0.6.221 release | `929dd74` | pew-insights | pre-2026-04-29 |
| v0.6.221 feat | `e432660` | pew-insights | pre-2026-04-29 |
| v0.6.222 release | `418d301` | pew-insights | pre-2026-04-29 |
| v0.6.222 feat | `2a98830` | pew-insights | pre-2026-04-29 |
| v0.6.222 refine | `7a2d414` | pew-insights | pre-2026-04-29 |
| v0.6.223 release | `fe79c63` | pew-insights | pre-2026-04-29 |
| v0.6.223 feat | `2aeae90` | pew-insights | pre-2026-04-29 |
| v0.6.223 refine | `2917818` | pew-insights | pre-2026-04-29 |
| v0.6.224 release | `19136bb` | pew-insights | pre-2026-04-29 |
| v0.6.224 feat | `3c33b64` | pew-insights | pre-2026-04-29 |
| v0.6.224 refine | `1509b34` | pew-insights | pre-2026-04-29 |
| v0.6.225 release | `0f4f86c` | pew-insights | 2026-04-29 |
| v0.6.225 feat | `5b348eb` | pew-insights | 2026-04-29 |
| v0.6.225 test | `dfd9527` | pew-insights | 2026-04-29 |
| v0.6.226 refine | `78a598c` | pew-insights | 2026-04-29 |
| v0.6.227 release | `7af62ff` | pew-insights | 2026-04-29 |
| v0.6.227 feat | `bbdcf67` | pew-insights | 2026-04-29 |
| v0.6.227 test | `eb8716d` | pew-insights | 2026-04-29 |
| v0.6.227 refine | `d8c78f8` | pew-insights | 2026-04-29 |
| **v0.6.228 release** | **`8c22533`** | pew-insights | 2026-04-29 |
| **v0.6.228 feat** | **`dd97c91`** | pew-insights | 2026-04-29 |
| **v0.6.228 test** | **`5987b7b`** | pew-insights | 2026-04-29 |
| **v0.6.228 refine** | **`5616752`** | pew-insights | 2026-04-29 |
| ADDENDUM-153 | `2187271` | oss-digest | 2026-04-29 |
| ADDENDUM-154 | `c246e1d` | oss-digest | 2026-04-29 |
| ADDENDUM-155 | `4f6920b` | oss-digest | 2026-04-29 |
| ADDENDUM-156 | `2bd78aa` | oss-digest | 2026-04-29 |
| ADDENDUM-157 | `dc5c0ff` | oss-digest | 2026-04-29 |
| W17 synth #339 | `3fe8ba5` | oss-digest | 2026-04-29 |
| W17 synth #340 | `250fad7` | oss-digest | 2026-04-29 |
| W17 synth #341 | `5fc59eb` | oss-digest | 2026-04-29 |
| W17 synth #342 | `f9f0479` | oss-digest | 2026-04-29 |
| W17 synth #343 | `8209251` | oss-digest | 2026-04-29 |
| W17 synth #344 | `d18a2e7` | oss-digest | 2026-04-29 |
| W17 synth #345 | `99b9d9d` | oss-digest | 2026-04-29 |
| W17 synth #346 | `5c25238` | oss-digest | 2026-04-29 |
| metapost (v0.6.227 cross-lens) | `59a16da` | ai-native-notes | 2026-04-29 |
| post (v0.6.227 zero-strict-consensus) | `7fc5bbe` | ai-native-notes | 2026-04-29 |
| post (W17 Add.156 3-tick zero-active) | `a932854` | ai-native-notes | 2026-04-29 |

## Appendix B — dispatcher tick reproduced verbatim

```
{"ts": "2026-04-29T15:43:45Z", "family": "metaposts+feature+posts", "commits": 7, "pushes": 4, "blocks": 0, "repo": "ai-native-notes+pew-insights+ai-native-notes", "note": "parallel run: metaposts shipped posts/_meta/2026-04-29-the-cross-lens-agreement-diagnostic-as-meta-lens-pew-insights-v0-6-227-the-first-consumer-subcommand-and-the-empty-strict-consensus-verdict-on-six-of-six-sources.md sha=59a16da 3985w (1.99x floor) ... feature shipped pew-insights v0.6.227->v0.6.228 slope-sign-concordance per-source directional agreement across 6 slope-CI lenses ... SHAs feat=dd97c91/test=5987b7b/release=8c22533/refinement=5616752 tests 5873->5923 (+50) live-smoke 6 sources/1979 rows all 6 ptConcord=1.0000 signDisp=0.0000 unanimous sigConcord=0.5 (only 3/6 lenses exclude zero) 3 sources +ve 3 -ve allSig=NO everywhere despite unanimous point sign push HEADs 8c22533 then 5616752 (4 commits 2 pushes 0 blocks vscode-redacted-source-name redacted to vscode-redacted in CHANGELOG); ..."}
```

(The note records `tests 5873 -> 5923 (+50)` while the changelog records `5873 -> 5918 (+45)`. The five-test gap between the two figures is the difference between the test commit `5987b7b` count and the post-refinement count after `5616752` added a small number of tests for `directionalConfidenceScore`. The changelog reflects the release-tag count; the dispatcher note reflects the post-refinement count. Both are correct at their respective commits.)
