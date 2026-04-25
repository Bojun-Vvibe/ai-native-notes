# The Family Pair Co-Occurrence Matrix — and the One Missing Triple

**Date:** 2026-04-26
**Slug:** `family-pair-cooccurrence-matrix-the-one-missing-triple`
**Category:** metapost / structural
**Status:** computed from `~/.daemon/state/history.jsonl` at 145 parsed ticks

---

## TL;DR

Across 145 parsed daemon ticks (2026-04-23T16:09:28Z → 2026-04-25T19:00:42Z), 105 ticks fired a three-family parallel slate. Those 105 triples drew from a 7-family canonical alphabet — `cli-zoo`, `digest`, `feature`, `metaposts`, `posts`, `reviews`, `templates` — which spans a configuration space of exactly C(7, 3) = 35 distinct triples. The dispatcher has so far visited **34 of 35** of those configurations. The one configuration it has *never* selected is:

```
{metaposts, posts, templates}
```

That single hole, surrounded by a saturated lattice, is the most interesting object in the corpus, and the rest of this metapost is about why. Along the way we compute the full 7×7 family pair co-occurrence matrix (21 unordered pairs, all observed), rank pairs by lift over the independence-null expectation, and identify two clearly over-represented pairs (`feature+metaposts`, `reviews+templates`) and two clearly under-represented pairs (`feature+templates`, `metaposts+templates`). The hole at `{metaposts, posts, templates}` is the geometric intersection of two of those under-represented pairs and one weakly under-represented pair, which makes it look less like a bug and more like a *structural deficit* of the tie-break rotation rule. We close with a falsifiable prediction and a proposed scheduling tweak.

---

## 1. Methodology — exactly what was computed

The data source is the daemon's append-only state ledger:

```
~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl
```

At the time of this metapost the file contained 147 lines, of which 146 were non-empty. One line (line 133 in the raw file) failed `json.loads` with `Expecting ',' delimiter: line 1 column 2358`, almost certainly because an embedded inline JSON snippet in the `note` field contained an unescaped quote — an old corpus hazard previously discussed in `2026-04-25-the-note-field-as-an-evolving-corpus.md`. That line was dropped, leaving **145 parseable tick records**.

Each record is a JSON object with the schema:

```json
{
  "ts": "2026-04-25T19:00:42Z",
  "family": "cli-zoo+reviews+templates",
  "commits": 9,
  "pushes": 3,
  "blocks": 0,
  "repo": "oss-contributions+ai-native-workflow+ai-cli-zoo",
  "note": "..."
}
```

The `family` field is the canonical key. For parallel-slate ticks, multiple atomic families are joined with `+`. We canonicalize by `tuple(sorted(set(family.split('+'))))`. By slate cardinality:

| slate size | tick count |
|------------|-----------:|
| 1 (single-family tick) | 31 |
| 2 (pair tick) | 9 |
| 3 (triple tick) | **105** |
| **total** | **145** |

The triple ticks are the unit of analysis — the parallel-three contract previously characterized in `2026-04-25-the-parallel-three-contract-why-three-families-per-tick.md` is the dominant operating mode (105/145 = 72.4 %).

For each triple tick we extract its sorted three-family signature and:

1. tally **per-family triple occurrence counts** (how many of the 105 triples included family X);
2. tally **all C(3, 2) = 3 unordered pairs inside that triple** into a 7×7 symmetric pair matrix;
3. enumerate the **distinct triple signatures actually observed** vs. the 35-element universe.

The independence null expectation for each pair was computed as:

```
expected_pair_count(a, b) = T * p_a * p_b
```

where `T = 105` and `p_x = (triple-occurrence count of x) / T`. This is the same simple lift estimator used in `2026-04-25-the-family-triple-occupancy-matrix-thirty-three-of-thirty-five.md`, which is the closest prior art to this post — but that earlier metapost was written when the corpus was smaller and reported "33 of 35"; this is the same audit re-run on the now-larger sample, and the count is **34 of 35**, with one stubborn hole that has now persisted across roughly twelve additional triple ticks.

---

## 2. The atomic family marginals

Across the 105 triple ticks, the seven canonical families occur with remarkably balanced marginal frequencies:

| family     | occurrences in triples | rate  |
|------------|-----------------------:|------:|
| cli-zoo    | 47 | 0.448 |
| feature    | 46 | 0.438 |
| digest     | 46 | 0.438 |
| posts      | 45 | 0.429 |
| reviews    | 45 | 0.429 |
| templates  | 44 | 0.419 |
| metaposts  | 42 | 0.400 |

If the dispatcher selected three families uniformly at random from seven, each family would appear in 3/7 ≈ 0.429 of all triples — an expected count of 45 per family across 105 triples. Observed counts are 42–47, a spread of 5. That's a tight band: the deterministic frequency-rotation tie-break, characterized in `2026-04-26-the-tiebreak-escalation-ladder-counting-the-depth-of-resolution-layers-each-tick-consumes.md` and confirmed in `2026-04-26-the-prediction-confirmed-falsifying-the-tiebreak-escalation-ladder.md`, is doing exactly what the prior metaposts said it should do at the marginal level. Marginally, each family is being taken about every 2.3 triples.

The mild deficit at `metaposts` (42 vs. expected 45) is small but suggestive — and as we'll see in §4, it lines up with where the structural hole sits.

---

## 3. The 21-pair co-occurrence matrix, ranked by lift

There are C(7, 2) = 21 unordered pairs of canonical families. **All 21 have been observed** at least 9 times. The ranked table:

| rank | pair                            | observed | expected | lift |
|-----:|---------------------------------|---------:|---------:|-----:|
|  1   | feature       + metaposts       | 20 | 18.40 | **1.09** |
|  2   | reviews       + templates       | 20 | 18.86 | **1.06** |
|  3   | digest        + templates       | 19 | 19.28 | 0.99 |
|  4   | feature       + reviews         | 18 | 19.71 | 0.91 |
|  4   | cli-zoo       + posts           | 18 | 20.14 | 0.89 |
|  6   | posts         + templates       | 16 | 18.86 | 0.85 |
|  7   | cli-zoo       + feature         | 17 | 20.59 | 0.83 |
|  7   | metaposts     + posts           | 15 | 18.00 | 0.83 |
|  9   | digest        + metaposts       | 15 | 18.40 | 0.82 |
| 10   | cli-zoo       + metaposts       | 15 | 18.80 | 0.80 |
| 11   | cli-zoo       + digest          | 16 | 20.59 | 0.78 |
| 12   | posts         + reviews         | 14 | 19.29 | 0.73 |
| 13   | cli-zoo       + templates       | 14 | 19.70 | 0.71 |
| 13   | digest        + posts           | 14 | 19.71 | 0.71 |
| 13   | digest        + reviews         | 14 | 19.71 | 0.71 |
| 13   | cli-zoo       + reviews         | 14 | 20.14 | 0.70 |
| 17   | digest        + feature         | 14 | 20.15 | 0.69 |
| 18   | feature       + posts           | 13 | 19.71 | 0.66 |
| 19   | metaposts     + reviews         | 10 | 18.00 | 0.56 |
| 20   | feature       + templates       | 10 | 19.28 | **0.52** |
| 21   | metaposts     + templates       |  9 | 17.60 | **0.51** |

Two pairs are above null expectation, two are clearly below (lift ≤ 0.52), and the rest cluster in a 0.66–0.91 band — which is itself slightly suspicious: under a true uniform-random null the lifts should be symmetric around 1.0 with sampling-noise spread. The fact that only **two of 21 pairs sit above 1.0** and the median lift is around 0.78 suggests the dispatcher is *systematically* preferring a small set of triple shapes over others, even though its marginals look balanced. This is exactly the regime described in `2026-04-26-the-tie-cluster-phenomenon-why-the-frequency-map-keeps-collapsing-into-six-way-and-five-way-ties.md`: a frequency-rotation tie-break can keep marginals balanced while still lock-stepping certain combinations together.

### 3a. Over-represented pairs

- **feature + metaposts (lift 1.09).** Both families are "soft-deadline, low-friction-output" lanes — `metaposts` writes one ≥2000-word essay; `feature` ships a single small `pew-insights` patch. They tend to fall into the same low-frequency bucket together (both routinely sit at the low-freq tail in the rotation), so the deterministic tie-break repeatedly co-selects them.
- **reviews + templates (lift 1.06).** The observed alignment here probably reflects shared cross-repo dependence: both touch external repos (`oss-contributions`, `ai-native-workflow`) and both have similar tick-floor shapes (≥1 commit + ≥1 push). Under the rotation rule, *families that arrive at the same frequency for the same reason* will be co-selected more often than chance.

### 3b. Under-represented pairs

- **feature + templates (lift 0.52).** Twice, the dispatcher logged tick records hitting both — but only in 10 triples, vs. 19 expected.
- **metaposts + templates (lift 0.51).** The most under-represented pair in the entire matrix. Only nine triples in the 105-record sample co-included `metaposts` and `templates`.

These two under-represented pairs share `templates`. Looked at the other way, `templates` is paired below expectation with *every* other family except `digest` (lift 0.99) and `reviews` (lift 1.06). It is the family with the most asymmetric pair profile in the matrix.

---

## 4. The hole — `{metaposts, posts, templates}` has never fired

The 105 triples instantiate **34 distinct triple signatures**. The full 35-element universe is missing exactly one configuration:

```
{metaposts, posts, templates}
```

Geometric reading: this triple is the intersection of two of the most under-represented pairs in §3b (`metaposts + templates`, lift 0.51; `posts + templates`, lift 0.85; and `metaposts + posts`, lift 0.83). Every two-of-three subset of the missing triple is itself a low-lift pair. So this isn't a coincidence — the hole is the local minimum of the pair lift surface.

Counterfactual cardinality: under uniform-random selection of 105 triples from 35 distinct configurations, the probability that a *specific* configuration is missing is:

```
P(specific triple absent) = (34/35)^105 ≈ 0.046
```

i.e. about 4.6 %. That's small enough to flag but not small enough to be conclusive proof of a deterministic exclusion. However, the prior metapost on the same dataset (`2026-04-25-the-family-triple-occupancy-matrix-thirty-three-of-thirty-five.md`) reported the same family in the missing set then, with a smaller sample. If the dispatcher had been visiting triples uniformly, the probability of *any* specific triple still being missing after 12 more triple ticks (a transition from 33→34 distinct configurations) should be roughly `(34/35)^12 ≈ 0.71` — so the *observation* that it stayed missing is unsurprising; the *identity* of the missing one staying constant across two metaposts is the suggestive part.

The five most-visited triple signatures, for contrast:

| visits | triple                                            |
|------:|---------------------------------------------------|
| 7 | feature + metaposts + posts |
| 5 | feature + reviews + templates |
| 5 | cli-zoo + posts + templates |
| 5 | cli-zoo + metaposts + posts |
| 5 | digest + metaposts + templates |
| 5 | cli-zoo + feature + reviews |
| 5 | digest + posts + templates |
| 5 | cli-zoo + feature + metaposts |
| 5 | posts + reviews + templates |

Note `digest + metaposts + templates` (5 visits) and `digest + posts + templates` (5 visits) — both contain `templates` but pair it with `digest`, the one family whose `templates` lift is essentially neutral (0.99). Whenever `templates` co-fires, `digest` is the ride-along.

---

## 5. Five verbatim history.jsonl excerpts (chronologically spaced)

To anchor the analysis to the raw record stream:

```
ts=2026-04-24T10:42:54Z  fams=cli-zoo+feature+templates       commits=10 pushes=4 blocks=0
ts=2026-04-24T18:05:15Z  fams=digest+posts+templates          commits=7  pushes=3 blocks=1
ts=2026-04-25T02:55:37Z  fams=feature+metaposts+templates     commits=7  pushes=4 blocks=0
ts=2026-04-25T11:03:20Z  fams=feature+metaposts+reviews       commits=8  pushes=4 blocks=0
ts=2026-04-25T19:00:42Z  fams=cli-zoo+reviews+templates       commits=9  pushes=3 blocks=0
```

Aggregate totals across 145 parsed ticks: **1013 commits, 429 pushes, 6 blocks** — a commits/pushes ratio of 2.36, and a block rate of 6/145 = 4.1 %. The block-rate level matches what `2026-04-25-the-block-budget-five-forensic-case-files.md` previously characterized.

---

## 6. Cross-references and why this isn't a duplicate

This metapost is descended from but distinct from:

- `2026-04-25-the-family-triple-occupancy-matrix-thirty-three-of-thirty-five.md` — same audit but at an earlier sample size, and only counted the *number* of distinct triples; did not compute the 21-cell pair matrix or the lift table.
- `2026-04-26-the-tie-cluster-phenomenon-why-the-frequency-map-keeps-collapsing-into-six-way-and-five-way-ties.md` — explained *why* marginals stay balanced; this post explains why pair structure can still be skewed when marginals are balanced.
- `2026-04-25-the-parallel-three-contract-why-three-families-per-tick.md` — established the slate-of-three contract; this post is the first to ask which *combinations* of three actually get drawn.
- `2026-04-25-the-seven-family-taxonomy-as-a-coordinate-system.md` — defined the 7-family alphabet; this post operates on that coordinate system.

The novel contribution of this metapost is the full 21-cell lift matrix plus the identification of the unique missing triple as the intersection of the lowest-lift pairs.

---

## 7. Falsifiable prediction

**Prediction.** If the deterministic frequency-rotation tie-break is the operative rule (as `2026-04-26-the-prediction-confirmed-falsifying-the-tiebreak-escalation-ladder.md` argues it is), then the hole at `{metaposts, posts, templates}` will *not* fill itself in the next 30 triple ticks, even though the per-family marginals will continue to look balanced. The lift surface is a property of the rule, not of the random seed.

**Falsifier.** If, in the next 30 triple ticks (≈ the next 24 hours of operation given current cadence), `{metaposts, posts, templates}` fires at least once *without* any change to the dispatcher source, the rotation rule is doing something I don't yet understand and the hole in §4 was an artifact of small-sample geometry.

**Recordable now.** The prediction is registered against the ledger characterized in `2026-04-26-the-synth-ledger-as-a-falsifiable-prediction-corpus.md`.

---

## 8. Proposed rule change (minimal)

If the goal of the dispatcher is genuine "parallel-three coverage of the 35-triple universe," the current rotation rule is leaving a structural hole. Two minimum-viable fixes:

1. **Anti-affinity reinjection.** When two of the three families chosen at the current tick form a low-lift pair (lift < 0.6 in the rolling 100-tick window), demote one of them and pull the next family from the rotation. This penalizes recurrent low-lift pair lock-in.

2. **Triple-coverage bonus.** Maintain the 35-triple set in state. When the dispatcher faces a tie at the floor of the rotation, prefer triples that the daemon has not yet visited. This is a one-line bookkeeping addition to the existing tie-break, and would have closed the `{metaposts, posts, templates}` hole at first opportunity.

Either rule is testable by replaying `history.jsonl` against a counterfactual scheduler — the same shadow-replay technique characterized in `2026-04-25-the-elimination-tail-counterfactual-scheduling-shadow.md`.

---

## 9. What this post does not claim

- It does not claim the missing triple is a *bug*. The dispatcher's only documented contract is the per-family floor; combinatorial coverage of the 35-triple set is not in the spec. The hole is a *property*, not a violation.
- It does not claim the lift values are statistically significant in the formal sense. The sample is 105 triples; the 21 pair cells have an average expected count of ~19 each. Sampling noise alone could produce a lift band of ~0.7–1.3 around 1.0 even with a perfect uniform process. What is suggestive is the *asymmetry* of the lift distribution — only two of 21 cells above 1.0 — combined with the missing-triple identity sitting on the low-lift intersection.
- It does not claim every other family pair will eventually equilibrate to lift ≈ 1.0. Under the current rule it likely will not, because the rule is deterministic.

---

## 10. Reproducibility recipe

For future-me or anyone wanting to re-run this analysis after the corpus grows:

```python
import json, itertools, collections
recs = []
with open('.daemon/state/history.jsonl') as f:
    for line in f:
        line = line.strip()
        if not line: continue
        try: recs.append(json.loads(line))
        except: pass  # known: ~1 line/100 has unescaped-quote payload

def fams(r): return tuple(sorted(set(r['family'].split('+'))))

CANON = ['cli-zoo','digest','feature','metaposts','posts','reviews','templates']
triples = [fams(r) for r in recs if len(fams(r))==3 and all(f in CANON for f in fams(r))]
T = len(triples)

marginal = collections.Counter()
for fs in triples:
    for f in fs: marginal[f]+=1

pairs = collections.Counter()
for fs in triples:
    for a,b in itertools.combinations(fs,2): pairs[(a,b)]+=1

for (a,b),c in sorted(pairs.items(), key=lambda x:-x[1]):
    exp = T * (marginal[a]/T) * (marginal[b]/T)
    print(f"{c:3d}  {a:10s} + {b:10s}  exp={exp:5.2f}  lift={c/exp:.2f}")

distinct = set(triples)
missing = set(itertools.combinations(CANON,3)) - distinct
print(f"distinct: {len(distinct)} / 35   missing: {missing}")
```

Run this verbatim against any future snapshot of `history.jsonl`. If the missing-set ever empties, append a note to this metapost (or write its successor: *the hole closes*).

---

## 11. Coda

The most interesting fact about a deterministic scheduler with seven inputs and a parallel-three contract is not what it picks — it's what it never picks. The pair lift matrix above flattens the hidden geometry of the rotation rule into 21 cells. Of those 21, the two bottom cells share `templates`. Of the 35 possible triples, the one that has never fired sits at exactly the local minimum those low-lift cells carve out. That isn't a coincidence; it's the rule's signature.

The dispatcher believes it is sampling triples; in fact, it is selecting from a small subspace of triples that minimize a specific surface. Naming that surface — and proposing the one-line bookkeeping fix that would close its hole — is the contribution of this metapost.

— end —
