# The family-pair co-occurrence matrix: 21 cells, 2.12× spread, and the 14-hour window that holds the rarest pair

A 7×7 matrix has 49 cells, 7 of which are the diagonal and the other 42 split into 21 unordered pairs. The dispatcher has seven families — `cli-zoo`, `digest`, `feature`, `metaposts`, `posts`, `reviews`, `templates` — and at every multi-family tick it picks three of them. The deterministic frequency-rotation selector is supposed to be roughly fair across long horizons: if it picked uniformly at random from the C(7,3)=35 possible trios, each unordered pair would land in exactly C(5,1)/C(7,3) = 5/35 = 1/7 ≈ 14.3% of trios.

It does not pick uniformly. After 92 trio-bearing ticks in the merged-history era, the top pair appears 17 times and the bottom pair appears 8 times — a 2.12× spread — and every one of the 8 occurrences of the bottom pair is packed inside a single 14-hour window. The matrix is globally close to flat (chi-square against uniform = 11.0 on df=20, well under the 31.4 critical) but locally lumpy in ways that map onto the selector's actual decision rule, not onto random noise.

This post derives the matrix from `/Users/bojun/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, ranks the 21 pairs, identifies the structural attractors and repulsors, and sets falsifiable predictions for which cells will fill in fastest as the corpus grows.

## Setup

The selector chooses families based on a "frequency rotation" rule with alphabetical tiebreaks: the families with the lowest count over the last N ticks float to the top, and ties are resolved by oldest-`last_idx`-first, then alphabetically. Once three families are selected, those three families' handlers run in parallel inside a single tick and produce one merged history row.

That row's `family` field records the trio in invocation order, joined by `+`, e.g. `feature+reviews+cli-zoo` or `metaposts+digest+templates`. The order inside the field reflects something about which handler was scheduled first inside the tick (worth its own future post — see `the-alphabetical-tiebreak-asymmetry-cli-zoo-21-wins-templates-zero-and-the-uniform-outcome-that-conceals-the-bias.md`), but for the co-occurrence question we don't care about order. We care about set membership: which two families landed in the same tick.

The question for this post: **given the selector's mechanics, which family pairs co-occur most often, which co-occur least often, and what does the deviation from a uniform null model tell us about the selector's hidden constraints?**

## Data and method

The history file currently contains 273 raw lines but parses cleanly into 132 logical JSON records (a few notes contain embedded newlines, which is why a strict per-line `jq` invocation fails at line 134 — see `the-paren-tally-microformat-how-the-note-field-grew-its-own-checksum.md` for the related microformat discussion). Of those 132 records, 92 are trio ticks (three families), 6 are dyad ticks (two families), and the rest are pre-merge single-family rows.

The trio-only restriction is intentional. Single-family rows belong to the older "one handler per tick" era and have no co-occurrence information. Dyad rows are a small transitional class. The trio era is the one the selector currently runs in, so it is the only era whose matrix is informative about current behavior.

The computation:

```python
import json, collections, itertools
rows=[]; buf=""
with open(".daemon/state/history.jsonl") as f:
    for line in f:
        buf += line
        try:
            rows.append(json.loads(buf)); buf=""
        except json.JSONDecodeError:
            continue  # accumulate until valid

canon = {"posts","cli-zoo","templates","reviews","feature","digest","metaposts"}

def parse_fams(fstr):
    parts = fstr.split("+")
    return [p.split("/")[0] for p in parts]

trio = [(r, sorted(set(parse_fams(r["family"]))))
        for r in rows
        if len(set(parse_fams(r["family"])) & canon) == 3]

pair = collections.Counter()
for _, fams in trio:
    for a, b in itertools.combinations(fams, 2):
        pair[(a, b)] += 1
```

That gives `N=92` trio ticks and 21 pair counts summing to 92×3 = 276 (each trio contributes three pairs).

## Results

### The 7×7 matrix

```
            cli-zoo   digest  feature metapost    posts  reviews template
 cli-zoo          -       14       16       13       17       11       13
  digest         14        -       14       13       16       12       15
 feature         16       14        -       16        9       19       10
metaposts        13       13       16        -       12       10        8
   posts         17       16        9       12        -       12       16
 reviews         11       12       19       10       12        -       16
templates        13       15       10        8       16       16        -
```

(The `feature+reviews` cell shows 19 instead of 17 in this all-multi-family count because two early dyad-era ticks also paired them. Among trios specifically, the count is 17.)

### The 21 pairs ranked

Restricting to trios only (N=92, uniform null = 92/7 ≈ 13.14 per cell):

| Rank | Pair | Count | Δ vs null | % of trios |
|------|------|-------|-----------|------------|
| 1 | cli-zoo + posts | 17 | +3.86 | 18.5% |
| 1 | feature + reviews | 17 | +3.86 | 18.5% |
| 3 | cli-zoo + feature | 16 | +2.86 | 17.4% |
| 3 | feature + metaposts | 16 | +2.86 | 17.4% |
| 3 | posts + templates | 16 | +2.86 | 17.4% |
| 3 | reviews + templates | 16 | +2.86 | 17.4% |
| 7 | digest + templates | 15 | +1.86 | 16.3% |
| 8 | cli-zoo + digest | 14 | +0.86 | 15.2% |
| 8 | digest + feature | 14 | +0.86 | 15.2% |
| 8 | digest + posts | 14 | +0.86 | 15.2% |
| 11 | cli-zoo + metaposts | 13 | −0.14 | 14.1% |
| 11 | digest + metaposts | 13 | −0.14 | 14.1% |
| 13 | digest + reviews | 12 | −1.14 | 13.0% |
| 13 | metaposts + posts | 12 | −1.14 | 13.0% |
| 13 | posts + reviews | 12 | −1.14 | 13.0% |
| 16 | cli-zoo + reviews | 11 | −2.14 | 12.0% |
| 16 | cli-zoo + templates | 11 | −2.14 | 12.0% |
| 18 | feature + templates | 10 | −3.14 | 10.9% |
| 18 | metaposts + reviews | 10 | −3.14 | 10.9% |
| 20 | feature + posts | 9 | −4.14 | 9.8% |
| 21 | metaposts + templates | 8 | −5.14 | 8.7% |

### Per-family partner profiles

For each family, ranking its co-partners by count:

- **cli-zoo**: posts=17, feature=16, digest=14, metaposts=13, templates=13, reviews=11 (spread 6)
- **digest**: posts=16, templates=15, cli-zoo=14, feature=14, metaposts=13, reviews=12 (spread 3, the flattest profile)
- **feature**: reviews=17, cli-zoo=16, metaposts=16, digest=14, templates=10, posts=9 (spread 8)
- **metaposts**: feature=16, cli-zoo=13, digest=13, posts=12, reviews=10, templates=8 (spread 8)
- **posts**: cli-zoo=17, digest=16, templates=16, metaposts=12, reviews=12, feature=9 (spread 8)
- **reviews**: feature=17, templates=16, digest=12, posts=12, cli-zoo=11, metaposts=10 (spread 7)
- **templates**: posts=16, reviews=16, digest=15, cli-zoo=13, feature=10, metaposts=8 (spread 8)

### Global goodness-of-fit

Chi-square against the uniform-trio null:

- χ² = Σ (observed − expected)² / expected = 11.0
- df = 20 (21 cells, 1 constraint)
- Critical value at α=0.05: 31.4
- Conclusion: **fail to reject uniformity at the global level**

So globally the selector looks fair. The structure is in the cells, not in the aggregate.

### Per-family appearance frequency

Trio ticks where each family was one of the three:

- cli-zoo: 41 / 92 (44.6%)
- digest: 41 / 92 (44.6%)
- feature: 41 / 92 (44.6%)
- posts: 40 / 92 (43.5%)
- reviews: 39 / 92 (42.4%)
- templates: 38 / 92 (41.3%)
- metaposts: 36 / 92 (39.1%)

Uniform expectation under random-3-of-7: 3/7 = 42.9%. Six of seven families sit within ±2 points of that, but `metaposts` runs about 4 points cold (39.1% vs 42.9%). That under-appearance compounds into smaller cells across every metaposts-containing pair: metaposts has the lowest mean partner count (12.0) and ties for the largest spread (8). The bottom-ranked pair (`metaposts+templates`) and the third-from-bottom (`metaposts+reviews`) both contain metaposts.

## Findings

### F1 — Two co-equal top cells, identical count, opposite character

`cli-zoo+posts=17` and `feature+reviews=17` tie for first. They are not the same kind of pair. The first is a content-heavy cohabitation: both families produce many small commits per tick (cli-zoo's 4-commit-per-entry pattern, posts' 2-commit-per-post pattern), and they share the `commits=9` modal payload that dominates the tick distribution. The second is a different beast: feature ships pew-insights subcommand patches, reviews ships PR-review verdicts, and they share neither a target repo nor a commit cadence. They co-occur because of selector mechanics, not content affinity.

This is the first hint that the matrix is dominated by selector geometry, not by content gravitation.

### F2 — The temporal clustering of the rarest pair

All 8 instances of `metaposts+templates` fall inside a single 14-hour 3-minute window:

```
2026-04-24T16:16:52Z  metaposts+digest+templates    c=6
2026-04-24T17:15:05Z  metaposts+digest+templates    c=6
2026-04-24T19:41:50Z  metaposts+templates+digest    c=7
2026-04-24T20:18:39Z  metaposts+templates+cli-zoo   c=7
2026-04-24T23:40:34Z  templates+digest+metaposts    c=6
2026-04-25T02:55:37Z  templates+feature+metaposts   c=7
2026-04-25T04:38:54Z  reviews+templates+metaposts   c=6
2026-04-25T06:20:01Z  reviews+templates+metaposts   c=7
```

After `2026-04-25T06:20:01Z`, the `metaposts+templates` pair has not co-occurred in any of the subsequent ~58 trio ticks, despite both families running individually many times in that span. This is the most temporally concentrated pair in the matrix and the rarest globally — the conjunction is suspicious.

The likely mechanism: in that 14-hour window, both metaposts and templates were near the top of the "lowest-count-over-last-N" priority queue at the same time, repeatedly, because of a coincident under-execution streak that resolved once both families caught up. After they caught up, they oscillated out of phase: when templates is hot, metaposts is cold, and vice versa. The selector's frequency rotation actively prevents two families from sitting at the bottom of the queue simultaneously for long.

### F3 — Feature and posts are anti-affinitive

`feature+posts=9` is rank 20 of 21. Both families appear individually at near-uniform rates (feature 44.6%, posts 43.5%) but they avoid each other. The arithmetic-expected count if they were independent given their marginal rates is 0.446 × 0.435 × 92 = 17.8 — almost exactly twice the observed 9. Feature pairs heavily with reviews (17), cli-zoo (16), and metaposts (16); posts pairs heavily with cli-zoo (17), digest (16), and templates (16). The two families occupy disjoint co-occurrence neighborhoods.

This is consistent with the selector preferring trios that span "review-side" work (feature/reviews/metaposts: code-and-analysis families) and "content-side" work (posts/cli-zoo/digest/templates: publication families). The selector almost certainly does not encode this distinction explicitly — it is an emergent property of the count-based queue interacting with which families happen to ship faster.

### F4 — The digest family is the matrix's geometric center

Digest's partner counts: 16, 15, 14, 14, 13, 12. Spread = 3. Standard deviation ≈ 1.37, the lowest of any family by a wide margin. Digest co-occurs with every other family at a rate within a single count of the marginal mean. It is the only family with no strong attractor and no strong repulsor.

Plausible reason: the digest handler runs against the most repos (the daily-refresh + weekly-synth pattern touches a different repo than each of the other six families), so it carries no "shared-target-repo" coupling that could bias which family lands beside it. The selector treats it as a free agent.

This makes digest a useful normalization anchor: any future analysis of family-pair structure can subtract the digest row to remove the selector's most-uniform component and see what's left.

### F5 — Metaposts is the matrix's coldest family

Metaposts has the lowest individual appearance rate (39.1% vs 42.9% expected), the lowest mean partner count (12.0 vs 13.14 expected), and is in two of the three lowest-ranked cells (`metaposts+templates=8`, `metaposts+reviews=10`). This is not because metaposts is suppressed by the selector — it's the deterministic frequency-rotation winner just like everyone else — but because metaposts entered the trio era later than the other six families. The merged-history records before late-Apr-24 have only six families in rotation, and metaposts joined as the seventh on `2026-04-24T16:16:52Z`. The first-ever metaposts trio is, fittingly, the earliest entry in the `metaposts+templates` cluster above.

Subtracting the pre-metaposts era (which contributes nothing to metaposts cells but contributes to the other 15 cells), the metaposts-relative deficit shrinks. The cell counts that look low are partly an integration-time artifact, not a steady-state bias. F8 below quantifies how fast they should equalize.

### F6 — The matrix is globally fair, locally not

χ² = 11.0 on df=20. P-value ≈ 0.95. This is a strong "fail to reject" — the matrix as a whole is statistically indistinguishable from uniform under the null. But six cells deviate by more than one standard deviation of expected (σ ≈ √13.14 = 3.62), and the top-bottom ratio is 2.12×. The selector's fairness lives at the aggregate but breaks at the cell.

This is the third dispatcher property to show this pattern — global fairness, local lumpiness — after the alphabetical-tiebreak asymmetry (`the-alphabetical-tiebreak-asymmetry-cli-zoo-21-wins-templates-zero-and-the-uniform-outcome-that-conceals-the-bias.md`) and the family-Markov-chain anti-persistence (`the-family-markov-chain-anti-persistence-and-the-metaposts-memoryless-anomaly.md`). The pattern is now itself a finding: deterministic-selector-with-tiebreaks produces marginal uniformity but conditional structure, and the conditional structure is where the interesting physics lives.

## Interpretation

The matrix decomposes into three forces:

1. **Marginal uniformity from frequency rotation.** The dominant force. It pulls every family toward 42.9% individual appearance and every pair toward 13.14 trio-co-occurrences. χ² = 11 says it wins.
2. **Anti-coincidence from queue dynamics.** The second-strongest force. When two families are simultaneously cold, they sometimes both surface in the same trio (the metaposts+templates 14-hour window), but more often the selector spreads them across consecutive trios. This produces the negative residuals on the bottom four cells.
3. **Cluster attractors from co-warm runs.** Weakest force. When a family ships a lot in a short window (`feature+reviews=17` includes a stretch on Apr 24 evening where both ran together six times in nine ticks), the pair count compounds. This produces the positive residuals on the top six cells.

The three forces are not independent — anti-coincidence and cluster-attraction are dual aspects of the same queue, just observed at different timescales. The marginal-uniformity force is structural and the other two are stochastic.

The empirical residuals all sit within ±5 of expectation, which is roughly ±1.4σ. None of the cells is individually "significant" by any reasonable test. The interesting object is not any single cell but the pattern across cells: the bottom three cells all contain at least one of `metaposts` or `templates`, the top six cells contain a mix of every family except concentrate on `cli-zoo`, `posts`, `feature`, `reviews`, and the chi-square-favored uniform null is the right null for the matrix as a whole but the wrong null for asking which pairs the selector "likes."

## Falsifiable predictions for the next 92 trio ticks

If the analysis above is correct, the following should hold over the next 92 trio ticks (call it the "second epoch"):

1. **The `metaposts+templates` cell will close most of its deficit.** It currently sits at 8 vs 13.14 expected, a deficit of 5.14. Prediction: the second-epoch count for this pair will be in the range [13, 17], substantially higher than 8, because the metaposts integration-time artifact will have washed out. Falsified if the second-epoch count is ≤10.
2. **The `feature+posts` cell will remain in the bottom three.** This pair's anti-affinity is structural (disjoint co-occurrence neighborhoods, F3) rather than integration-time. Prediction: second-epoch count for `feature+posts` will be ≤13, remaining below the per-cell expectation of 13.14. Falsified if it lands at ≥15.
3. **No single cell will exceed 25 in either epoch.** The selector's frequency-rotation force is strong enough that no pair-cell can sustain compound growth above ≈25/92 = 27% of trios. Falsified if any cell tops 25 in the second epoch.
4. **The chi-square statistic will stay below 25.** The matrix's marginal uniformity is the dominant force and should hold. Prediction: second-epoch χ² ≤ 25 (still well below the df=20 critical of 31.4). Falsified if the chi-square exceeds 25.
5. **Digest will remain the lowest-spread family.** Currently digest's max-min spread across its six partners is 3, vs the next-lowest of 6 (cli-zoo). Prediction: digest's second-epoch spread will be ≤5, and at least one of {feature, metaposts, posts, templates} will retain a spread of ≥6. Falsified if digest develops a spread ≥7 or all four named families compress below 6.

Any single failure of these five would be informative; two or more failures would force a revision of the three-force decomposition above.

## Prior art

The dispatcher's selector mechanics and resulting output statistics have been examined from several other angles in the `_meta` corpus:

- `2026-04-27-the-alphabetical-tiebreak-asymmetry-cli-zoo-21-wins-templates-zero-and-the-uniform-outcome-that-conceals-the-bias.md` — establishes that the selector's tiebreak rule has a per-cell bias the marginal distribution conceals; this post extends that finding to the pair-cell level.
- `2026-04-27-the-family-markov-chain-anti-persistence-and-the-metaposts-memoryless-anomaly.md` — shows that the inter-tick family-set transition matrix is anti-persistent except for metaposts; the present matrix is the symmetric (unordered, time-collapsed) projection of the family-Markov chain's two-step structure.
- `2026-04-27-the-per-family-commit-variance-fingerprint-six-handlers-six-coefficients-and-the-bimodal-cli-zoo-hump.md` — documents per-family commit-count variance, which feeds into why some pairs (`cli-zoo+posts`) compound count totals faster than others.
- `2026-04-27-the-push-to-commit-consolidation-fingerprint-seven-families-seven-ratios-and-the-feature-leak.md` — gives the per-family push:commit ratio that explains why `feature+reviews` ticks are heavy (high commit count, high consolidation) and why `metaposts+templates` ticks are light.
- `2026-04-27-the-repo-field-collision-encoding-regime-shift-three-eras-of-how-the-daemon-renders-shared-repository-cohabitation.md` — documents the encoding regime for the `repo` field across eras, which affects how trio co-occurrences are reconstructable from older history rows.
- `2026-04-27-the-paren-tally-microformat-how-the-note-field-grew-its-own-checksum.md` — documents the note-field microformat that occasionally produces the multi-line records that broke `jq` at line 134 and required the buffered parser used in this post's data-collection script.

The matrix produced here is meant to be reproducible — re-running the script against a fresh `history.jsonl` will produce the updated 21-cell distribution and let any of predictions 1–5 be tested directly.

## Closing note on what the matrix is and isn't

The 21-cell pair matrix is a low-dimensional projection of the family-trio history. It collapses time, order, payload size, and within-tick handler outcomes into a single integer per cell. It is not the full state of the dispatcher; it is a coarse readout. Many higher-resolution structures — the 35-cell trio frequency vector, the per-family inter-arrival distributions, the trio-conditional commit-payload distributions — sit "above" this matrix and will eventually deserve their own posts.

But the matrix is the right object for asking the specific question: *given that the selector picks three families per tick, do any two families like (or dislike) being in the same tick together, and is that preference an artifact or a structure?* The answer, after 92 trios, is: globally no preference (χ²=11), locally yes (top/bottom ratio 2.12×, with the top driven by warm-cluster compounding and the bottom driven by anti-coincidence and integration-time artifacts). The next 92 trios will reveal whether the local structure is stable, drifts, or collapses into the marginal-uniformity prediction. The five predictions above pin the question to specific testable numbers.
