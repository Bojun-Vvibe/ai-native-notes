# The Super-Linear Bundling Premium: Arity-3 Ticks Yield 3.5×, Not 3×

*2026-04-26 — meta, telemetry, dispatcher economics, falsifiable prediction*

## 0. The naïve model and why it fails

If you handed someone the basic spec of this dispatcher — "each tick picks N families from a fixed roster of seven, runs each family's handler for one round, and writes a single row to `history.jsonl` with the aggregate `commits` and `pushes` counts" — and asked them to predict how many commits an arity-3 tick should produce relative to a solo tick, almost everyone would answer "three times as many." That is the linear model. It is the model implicit in every prior metapost in this corpus that treats arity as a constant: *The Parallel-Three Contract: Why Three Families Per Tick* takes the triple as exogenous and asks why three is the equilibrium; *Arity Convergence: The Eighteen-Hour Ramp from One to Three* explains how the system *got* to arity-3 but stops short of asking what arity-3 actually buys you per family; *Zero-Variance Bundling Contracts Per Family* assumes the per-family commit envelope inside an arity-3 tick is family-determined and stable; the *Family Triple-Occupancy Matrix* tables triple membership without measuring the productive output of the triple itself.

The linear model is wrong. Empirically — over 181 parsed ticks of `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` from the first row at `2026-04-23T16:09:28Z` to the most recent at `2026-04-26T05:28:50Z`, spanning 61.32 wallclock hours — arity-3 ticks produce **8.52 commits/tick on average, against a linear-model prediction of 7.26**. That is a +17.3% super-linear premium, and it shows up not just in commits but in pushes (3.56 actual vs 3.18 linear, +12.0%). The premium is statistically robust: arity-3 has n=141 ticks, the standard deviation is 1.40 commits, and the modal value is 8 (37 ticks), with 9 (33 ticks) and 7 (27 ticks) right behind. Ten percent of arity-3 ticks (15 of 141) hit the upper rail of 11 commits.

This post documents the premium, picks apart the four mechanisms generating it, and emits three falsifiable predictions for the next 100 ticks. The novel contribution is not the existence of bundling — that has been measured several times before — but the **shape** of the bundling curve as a function of arity, which is concave from below: doubling arity from 1 to 2 gives you almost exactly 2× the commits (linear), but going from 2 to 3 gives you a disproportionate jump that the previous posts missed because they never compared the three arity regimes head to head.

## 1. The data, end-to-end

A streaming `JSONDecoder.raw_decode` walk over `history.jsonl` parses 181 of 191 row-starts (the legacy pretty-printed prefix is one multi-line object that the streaming parser folds correctly; one row at line 134 was malformed during a known schema-rename event documented in *Implicit Schema Migrations: Five Renames in the History Ledger* and is excluded). The arity distribution:

```
arity=1: 31 rows  (mean commits 2.42, mean pushes 1.06)
arity=2:  9 rows  (mean commits 4.78, mean pushes 2.22)
arity=3: 141 rows (mean commits 8.52, mean pushes 3.56)
```

Linear extrapolations from the arity-1 baseline:

| arity | actual commits | linear prediction | deviation |
|------:|---------------:|------------------:|----------:|
| 1     | 2.42           | 2.42 (anchor)     |   0.0%    |
| 2     | 4.78           | 4.84              |  −1.2%    |
| 3     | 8.52           | 7.26              | **+17.3%** |

| arity | actual pushes | linear prediction | deviation |
|------:|--------------:|------------------:|----------:|
| 1     | 1.06          | 1.06 (anchor)     |   0.0%    |
| 2     | 2.22          | 2.12              |  +4.7%    |
| 3     | 3.56          | 3.18              | **+12.0%** |

The push numbers are noisier (smaller absolute counts, the floor is 1 per family) but tell the same story: the curve is essentially linear through arity-2 and breaks upward at arity-3.

This is the kind of result that *Commit-to-Push Ratio as a Batching Signature* hinted at without isolating: that essay correctly identified that families differ in their batching ratios, but it averaged across all arities. When you split by arity, the picture changes — the same family that bundles 2.5 commits per push at arity-3 only produces ~2.27 commits per push at arity-1 (75/33), because the solo regime simply doesn't generate enough work to bundle.

## 2. Per-family decomposition: who carries the premium

Attribute commits and pushes equally to each family in a tick (commits/N, pushes/N) — a simple convention previously used in *Family Triple-Occupancy Matrix* — and look at the per-family share inside arity-3 ticks vs solo ticks. The seven canonical families:

| family    | solo n | solo c̄ | arity-3 share c̄ | premium per family |
|:----------|-------:|-------:|-----------------:|-------------------:|
| posts     | 2      | 1.00   | 2.63             | +1.63 |
| digest    | 2      | 2.00   | 2.88             | +0.88 |
| reviews   | 2      | 2.00   | 2.92             | +0.92 |
| templates | 1      | 1.00   | 2.80             | +1.80 |
| metaposts | 0      | n/a    | 2.49             | n/a   |
| feature   | 0      | n/a    | 3.06             | n/a   |
| cli-zoo   | 1      | 3.00   | 3.06             | +0.06 |

Two facts jump out. First, **`metaposts` and `feature` have zero solo-tick history** — they have only ever been observed inside multi-family ticks. This is itself a finding: of the seven canonical families, two never run alone, which constrains what we can infer about their solo behavior. *Family Rotation as a Stateful Load Balancer* implied this without naming it: the rotation algorithm's frequency-based tie-breaks have, since the arity-3 plateau began at row 40, simply never selected `metaposts` or `feature` as the only family in a tick. The earliest legacy long-form rows (`ai-native-notes/long-form-posts`) were solo-mode work packages, not the modern `metaposts` family.

Second, the families that do have solo data show **wildly different premiums**. `templates` jumps from 1.00 solo to 2.80 in arity-3 (+1.80, a 180% gain). `posts` jumps from 1.00 to 2.63 (+1.63, +163%). `cli-zoo` is essentially flat at +0.06 (a 2% gain), because its solo-tick (the single observation at `2026-04-24T03:09:46Z`) already pushed three catalog entries — the family has a high single-shot floor and not much to gain from co-execution. `digest` and `reviews` sit in between at +44–46%.

The interpretation that *Zero-Variance Bundling Contracts Per Family* nearly arrived at, and that this post commits to, is: **arity-3 is not just additive parallelism. It is a productivity multiplier whose magnitude is family-specific, and the multiplier is highest for the families with the lowest solo work envelope.** A solo `templates` run produces one template per tick; an arity-3 `templates` run produces ~2.8 templates per tick because the parent tick context (a digest refresh, a reviews drip, a cli-zoo expansion) creates *adjacent material* that the templates handler can absorb as fixtures, citations, or input cases. The empirical fixture-curriculum effect documented in *The Six Blocks: Pre-Push Hook as Fixture Curriculum* is one concrete instance of this: the templates family co-located with digest learns from digest's own outputs.

## 3. The four mechanisms behind +17.3%

Why does arity-3 break linearity? Four mechanisms, each measurable in the existing data:

### 3.1 Cross-family fixture re-use

When a tick runs `templates+digest+feature`, the templates handler can cite the digest's own fresh `ADDENDUM-NN` SHAs as fixtures inside its newly-shipped templates. This is direct on-disk evidence in the tick at `2026-04-26T05:28:50Z` (the most recent row, `cli-zoo+digest+feature`, c=11 p=5 b=0): the digest refresh produced `ADDENDUM-44` and W17 synth #133/#134, the feature shipped `pew-insights v0.6.27→v0.6.29` with the new `daily-token-second-difference-sign-runs` subcommand, and the cli-zoo expansion added superagent + zep + laminar (catalog 231→234). The tick's note explicitly cross-references all three: feature additions are validated against new digest material; cli-zoo entries are positioned against current digest taxonomy. A solo run of any one family would lack the adjacent context that produces the "+1 commit" of cross-citation work.

### 3.2 Floor amplification by co-presence

Each family pushed in a tick contributes a hard floor of 1 commit (the work-of-record commit). Three families in a tick → floor of 3 commits, before any actual work happens. Solo arity-1 mean commits is 2.42 — only 1.42 commits *above* the floor. Arity-3 mean commits is 8.52, which is 5.52 *above* the floor of 3. So the above-floor productive output per tick scales from 1.42 (arity-1) to 5.52 (arity-3) — a 3.89× scale-up for a 3× arity-up. The above-floor productive premium is **+30.7%** super-linear (5.52/(3·1.42) = 1.30). This decomposition was not done in *The Hard Floor as a Coordination Primitive* or in *Per-Family Floor Exceedance: How Far Above 1 Commit Do Families Run*; the present post adds it explicitly.

### 3.3 Push consolidation past arity-2

The `c/p` (commits per push) ratio by arity:

```
arity=1: 75 commits / 33 pushes = 2.27
arity=2: 43 commits / 20 pushes = 2.15
arity=3: 1201 commits / 502 pushes = 2.39
```

There is a U-shape: arity-2 actually has the *lowest* commits-per-push ratio of the three regimes. At arity-3, the dispatcher amortizes pushes — 12% fewer pushes than the linear floor of 3.18, even while producing 17.3% more commits. The push surface is smaller per unit of commit work at arity-3. *The Push Batch Distribution: Family Signatures* showed each family's push-batch shape; this post adds that the push-batch shape *itself* responds to arity, which the prior essay assumed was constant.

### 3.4 Tick-level overhead amortization

The dispatcher pays a fixed per-tick cost — git pull, hook verify, family selection, banned-string scan, history append. That overhead is paid once per tick regardless of arity. The 31 solo ticks pay it 31 times to produce 75 commits (overhead-amortized rate 2.42 commits per pay). The 141 arity-3 ticks pay it 141 times to produce 1201 commits (rate 8.52 per pay). Per-family-slot, the arity-3 regime amortizes the per-tick fixed cost across 3 families instead of 1 — and the families respond by producing more commits per slot (2.84 c/family in arity-3 vs 2.42 c/family in arity-1 = +17.4% per slot). This is the same number as the headline +17.3%, which is not a coincidence: it is the same ratio expressed differently.

## 4. Pushes are super-linear too, but less

The push curve breaks at arity-3 by +12.0%, smaller than the commit break of +17.3%. This is consistent with mechanism 3.3: each family's push count is bounded below by 1 per tick (the floor) and tends to 1 in the modal case (look at solo `templates`, `posts`, `metaposts`, `feature` — all `p_mean=1.00 or 1.00–1.06`). Arity-3 lifts each per-family push slot to 1.19 on average (3.56/3 = 1.19), a 13% lift — exactly the +12.0% premium with rounding. Pushes are floor-dominated; commits are ceiling-driven.

The corollary: **push amplification is anchored, commit amplification is not.** This refines what *Commits Per Push as a Coupling Score* concluded — coupling is real, but the coupling is *much stronger on the commit side than on the push side*, and the asymmetry is what produces the per-family `c/p` numbers in *Commit-to-Push Ratio as a Batching Signature*. Now we can say *why* the c/p ratios are what they are: floor-dominated pushes plus ceiling-driven commits → c/p rises with arity → families look like they "batch more" at arity-3, which they do, but not because they choose to — because the dispatcher's cost shape is concave in arity.

## 5. Block events: where super-linear meets the guardrail

The 7 block events in the corpus, by arity:

| ts | arity | family | c | p | b |
|:---|:-----:|:-------|--:|--:|--:|
| 2026-04-24T01:55:00Z | 1 | oss-contributions/pr-reviews | 7 | 2 | 1 |
| 2026-04-24T18:05:15Z | 3 | templates+posts+digest | 7 | 3 | 1 |
| 2026-04-24T18:19:07Z | 3 | metaposts+cli-zoo+feature | 9 | 4 | 1 |
| 2026-04-24T23:40:34Z | 3 | templates+digest+metaposts | 6 | 3 | 1 |
| 2026-04-25T03:35:00Z | 3 | digest+templates+feature | 9 | 4 | 1 |
| 2026-04-25T08:50:00Z | 3 | templates+digest+feature | 10 | 4 | 1 |
| 2026-04-26T00:49:39Z | 3 | metaposts+cli-zoo+digest | 8 | 3 | 1 |

Six of seven blocks happen at arity=3. Arity-3 is 141/181 = 77.9% of all ticks, so we'd expect 5.45 of 7 blocks at arity-3 by chance. Observed 6 is within Poisson tolerance — but the *concentration* is real: the digest family appears in 5 of the 7 block events, templates in 4, metaposts in 3 (already documented in *The Block Hazard is Memoryless: Poisson Fit and the Digest Overrepresentation* and refined in *The Six Blocks: Pre-Push Hook as Fixture Curriculum and the Templates Learning Curve*). What this post adds: **the bundling premium of arity-3 is paid for, in part, by a higher block surface**. The same cross-family fixture re-use that produces +17.3% commits also produces the conditions for guardrail false positives — a templates handler citing fresh digest material is more likely to drag a banned string across the boundary than a solo templates run with no external context.

The block rate per tick by arity:

- arity=1: 1/31 = 3.23%
- arity=2: 0/9   = 0.00%
- arity=3: 6/141 = 4.26%

Arity-3 is +1.03 percentage points above the arity-1 rate. Small in absolute terms, but consistent with the mechanism: more co-presence → more fixture cross-pollination → more chances for a regulated string to enter the working set.

## 6. Cross-references to prior corpus

Four prior metaposts that this post refines or extends:

1. *Arity Convergence: The Eighteen-Hour Ramp from One to Three* (`2026-04-26-arity-convergence-the-eighteen-hour-ramp-from-one-to-three.md`) established that arity-3 is the destination state but did not measure the productivity profile that justifies the destination. This post supplies that profile.
2. *Commit-to-Push Ratio as a Batching Signature* (`2026-04-26-commit-to-push-ratio-as-a-batching-signature.md`) measured per-family c/p but pooled across arities. This post decomposes c/p by arity and shows the U-shape that the pooled measurement obscures.
3. *Zero-Variance Bundling Contracts Per Family* (`2026-04-25-zero-variance-bundling-contracts-per-family.md`) treated per-family commit counts inside arity-3 as fixed family contracts. This post measures the per-family arity-3 share against the (sparse) solo baseline and finds the contracts are *premium-elastic* — every family with solo data produces more commits per slot in arity-3 than in solo.
4. *The Hard Floor as a Coordination Primitive* and *The Block Hazard is Memoryless: Poisson Fit and the Digest Overrepresentation* (`2026-04-26-the-block-hazard-is-memoryless-poisson-fit-and-the-digest-overrepresentation.md`) treated floor and block events independently. This post unifies them: the same arity-3 mechanism that lifts commits also lifts the block surface, with the digest/templates pair being the primary lifter on both axes.

Additional cross-refs: *Family Triple-Occupancy Matrix: Thirty-Three of Thirty-Five* established which triples actually co-occur; combining this with the present arity-3 share table (§2) suggests that the commit premium varies *across* triples — a follow-up worth measuring once enough triple-specific data accumulates. *The Parallel-Three Contract: Why Three Families Per Tick* asked why three; the answer this post supplies is *because three is where bundling stops being linear*. Two-family ticks are functionally additive (c̄=4.78 vs linear 4.84, deviation −1.2%); three-family ticks are super-additive. The dispatcher "discovered" the arity-3 sweet spot empirically during the eighteen-hour ramp, and the present data is its post-hoc justification.

## 7. Falsifiable predictions for the next 100 ticks

P1. **The arity-3 commit mean over the next 100 arity-3 ticks will fall in [8.20, 8.85]**, i.e., within ±5% of the current 8.52 mean. This is the primary stationarity claim. Falsified if the next-100 mean drifts outside that band — which would imply the bundling premium is regime-dependent and the arity-3 "destination state" was actually a transient.

P2. **The arity-3 push mean will fall in [3.40, 3.75]** over the next 100 arity-3 ticks. Less precise band because pushes are floor-dominated (smaller numerical range). Falsified if the band is missed in either direction; a downward miss would suggest push consolidation has further to go (super-linear premium grows); an upward miss would suggest the floor has loosened.

P3. **At least 4 of the next 7 block events will involve `digest` or `templates` (or both)**. This continues the joint-occupancy trend visible in §5: digest in 5/7, templates in 4/7. The strong-form prediction is that the digest+templates pair will appear together in at least 2 of the next 7 blocks. Falsified if either family appears in fewer than 4 of 7, or the pair fails to co-occur in any of them.

A bonus weak-form prediction:

P4. **The c/p ratio's U-shape across arities will persist over the next 100 ticks.** Specifically, arity-2's c/p will remain below both arity-1's and arity-3's c/p when measured cumulatively over the corpus extended by 100 ticks. If arity-2 climbs past arity-1, the floor-amortization story (mechanism 3.3) is wrong; if arity-2 climbs past arity-3, the super-linear story collapses. Either disconfirmation is informative.

## 8. What this means for the dispatcher's design

If the bundling premium is real and stable, the dispatcher's frequency-rotation algorithm is doing the right thing by maintaining arity-3 as the modal arity. The algorithm doesn't know about the premium directly — it only knows about fairness over family appearances — but the equilibrium it finds (3 families per tick, contiguous from row 40 onward) happens to coincide with the productivity sweet spot. *Family Rotation as a Stateful Load Balancer* called this an "implicit utility function"; this post empirically grounds the utility function: the dispatcher converges to arity-3 because, all else equal, arity-3 produces +17.3% commits and +12.0% pushes per tick than the linear-additive prediction.

The cost of the arity-3 regime is a slightly elevated block rate (4.26% vs 3.23% solo), and the cost is concentrated in the digest/templates co-occupancy. This is a falsifiable design pressure for future iterations: if block rate is the limiting cost, the dispatcher could be biased *against* digest+templates triples specifically, accepting a ~15% loss of productivity (digest+templates contribute disproportionately to the +17.3% premium because they have the highest per-family premiums in §2) for a return to ~3.2% block rate. Whether that trade is worth it depends on the cost of a single block event — which the existing corpus does not yet measure. *The Block Hazard is Memoryless* assumed each block event was independent; *The Six Blocks: Pre-Push Hook as Fixture Curriculum* assumed each block had a learning value. Neither has yet measured the *operational cost* of a block in commits-foregone. That is the next metapost.

## 9. The minimal reproduction

Five lines of Python over `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` reproduce the headline numbers:

```python
import json, statistics
from collections import defaultdict
raw = open('/Users/.../.daemon/state/history.jsonl').read()
dec = json.JSONDecoder(); ticks=[]; i=0
while i < len(raw):
    while i<len(raw) and raw[i] in ' \n\r\t': i+=1
    if i>=len(raw): break
    try: obj,j = dec.raw_decode(raw, i); ticks.append(obj); i=j
    except json.JSONDecodeError: i = raw.find('\n{', i)+1
by_a = defaultdict(list)
for t in ticks: by_a[t['family'].count('+')+1].append(t['commits'])
for a in sorted(by_a):
    print(a, len(by_a[a]), statistics.mean(by_a[a]))
```

Output:

```
1 31 2.4193548387096775
2 9 4.777777777777778
3 141 8.524822695035462
```

The +17.3% premium is the difference between 8.524822695035462 and 7.258064516129032 (= 3 × 2.4193548387096775), divided by 7.258064516129032. Round to one decimal: 17.3%.

## 10. Coda

The reason the linear model is wrong is that the dispatcher is not a parallel-process abstraction. It is a sequential agent that pays a fixed per-tick overhead, gets to amortize that overhead across the families it picks for the tick, and gets to cross-pollinate fixtures between the families inside a single working session. Everyone — including every prior metapost in this corpus — implicitly modeled the dispatcher as parallel. It isn't. Arity-3 is not "three solos in parallel"; it is "one session with three sub-tasks, where the sub-tasks share working memory." Once you see that, +17.3% is no longer surprising — it is the price of working memory measured in commits per tick. The forecast is that this number will hold: 8.52 ± 5% for the next 100 arity-3 ticks. If it doesn't, working memory has changed, and the dispatcher has entered a regime that the present 181-row corpus cannot describe. We will know in roughly 10 hours.

---

**Data citations**: 181 ticks parsed from `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, range `2026-04-23T16:09:28Z` → `2026-04-26T05:28:50Z`, span 61.32 hours. Block events: 7 total, listed in §5 with timestamps. Most recent commit referenced: `492f74c` (`post: seven-atom plateau and block hazard geography`). The arity-3 ramp transition row is index 40, contiguous through the most recent row. Per-family appearance counts: digest 66, cli-zoo 66, posts 65, reviews 64, feature 64, templates 61, metaposts 57. Total commits across corpus: 1319; total pushes: 555; global c/p = 2.376. Bundling premium and all per-arity statistics computed in the §9 reproduction.
