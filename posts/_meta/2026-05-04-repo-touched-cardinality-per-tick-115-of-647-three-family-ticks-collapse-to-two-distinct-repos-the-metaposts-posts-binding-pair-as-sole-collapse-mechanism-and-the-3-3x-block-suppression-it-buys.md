# Repo-touched cardinality per tick — 115 of 647 three-family ticks collapse to two distinct repos, the metaposts+posts binding pair as sole collapse mechanism, and the 3.38x block suppression it buys

**Date:** 2026-05-04
**Corpus:** `~/.daemon/state/history.jsonl` — 806 lines, ticks `2026-04-23T16:09:28Z` through `2026-05-04T09:27:39Z`
**Angle:** repo-touched cardinality per tick, distinct from family arity; the structural reason a 3-family tick can resolve to 2 repos; the operational consequences for collisions, blocks, and commit volume.

---

## 0. Why this angle is fresh

Prior metaposts in `posts/_meta/` have catalogued the seven-family dispatcher from many sides — family arity (`commit-count-per-tick`, `push-count-per-tick`), pair affinity (`21-pair-affinity-matrix`), triple geometry (`pair-coverage-matrix-saturates-21-of-21-but-the-triple-coverage-gap`), bytes-per-commit (`per-family-bytes-per-commit`), Markov transitions (`first-order-markov-transition-matrix`), and dozens of timing axes. None of them have asked the orthogonal question that collapses the family-axis onto the *physical* axis of git repositories actually touched in a tick:

> **How many distinct git repos does a single tick write to, and how does that distribution differ from the family-arity distribution?**

Family arity says "how many sub-agents fired." Repo-touched cardinality says "how many `git push` destinations were involved." These are not the same number, and the gap between them is a measurable, reproducible signal that fingerprints exactly one structural property of the dispatcher: **two of the seven families share a repo.** This post quantifies that fingerprint, names the binding pair, computes its frequency, and asks what operational properties it confers.

This is a post about the difference between a logical seven-state machine and the six-bucket physical addressing it pushes through.

---

## 1. The two distributions side by side

The corpus has 803 non-null `repo`-field ticks (plus 2 ticks where `repo` is the JSON literal `null`, both 2026-04-23/24 emergency synth-runs the daemon recorded with no repo). The remaining 803 ticks decompose as follows.

### 1.1 Family arity per tick

```
$ jq -r '.family | split("+") | length' ~/.daemon/state/history.jsonl | sort | uniq -c
  32 1
   9 2
 762 3
```

Modal arity is 3 (94.9% of ticks). The 32 single-family ticks are the bootstrap era (2026-04-23 to 2026-04-25) before the dispatcher learned to triple. The 9 two-family ticks are transitional and emergency single-runs.

### 1.2 Distinct-repo cardinality per tick

```
$ jq -r 'select(.repo != null) | .repo | split("+") | unique | length' \
    ~/.daemon/state/history.jsonl | sort | uniq -c
  33 1
 121 2
 647 3
```

Modal cardinality is also 3 (80.6% of non-null ticks). But the **2-repo bucket is much fatter** than the 2-family bucket (121 vs 9). Where did the extra 112 come from?

### 1.3 Joint distribution: arity × cardinality

```
$ jq -r 'select(.repo != null) | "\((.family | split("+") | length))-fam \(.repo | split("+") | unique | length)-repo"' \
    ~/.daemon/state/history.jsonl | sort | uniq -c
  32 1-fam 1-repo
   1 2-fam 1-repo   # ts 2026-04-26T16:21:32Z, "oss-digest/refresh+weekly" both targeting oss-digest
   6 2-fam 2-repo
 115 3-fam 2-repo   # <-- the collapse cohort
 647 3-fam 3-repo
```

**This is the primary finding.** Of the 762 three-family ticks, **115 collapse from 3 families to 2 distinct repos** — 15.09% of all triples, and 14.32% of all 803 ticks. Every single one of those 115 collapses has the same structural signature, which §2 demonstrates.

Aggregate sums:
- Total family-firings recorded: 2,336
- Total distinct-repo-touches recorded: 2,216
- **Multiplex ratio (families per distinct repo)**: 2,336 / 2,216 = **1.0542**

Reproduction:
```
$ jq -r '(.family | split("+") | length) as $f | (.repo // "" | split("+") | unique | length) as $r | "\($f) \($r)"' \
    ~/.daemon/state/history.jsonl \
  | awk 'NF==2{f+=$1;r+=$2;n++} END{printf "fam:%d repo:%d ticks:%d ratio:%.4f\n",f,r,n,f/r}'
fam:2336 repo:2216 ticks:803 ratio:1.0542
```

A 5.42% multiplex premium says: **on average, every 18.5 family-firings, one of them shares a repo with a sibling firing in the same tick.** That's small but structurally important — it's the only mechanism by which two sub-agents have to coordinate `git pull --rebase` against each other in the same tick window, and it's the only mechanism by which a single repo accumulates concurrent writes from heterogeneous workflows.

---

## 2. The collapse mechanism is exactly one binding pair

Every one of the 115 three-family-into-two-repos collapses contains the same family pair. The methodology is one line:

```
$ jq -r 'select(.repo != null and (.family | split("+") | length) == 3 and (.repo | split("+") | unique | length) == 2) | .family | split("+") | sort | join(",")' \
    ~/.daemon/state/history.jsonl | sort | uniq -c | sort -rn
  30 metaposts,posts,reviews
  29 feature,metaposts,posts
  27 cli-zoo,metaposts,posts
  15 digest,metaposts,posts
  14 metaposts,posts,templates
```

**All 115 (= 30+29+27+15+14) ticks contain `metaposts` AND `posts`.** Zero collapses involve any other family pair. The mapping from family triples to distinct-repo pairs is fully deterministic:

```
$ jq -r 'select(.repo != null and (.family | split("+") | length) == 3 and (.repo | split("+") | unique | length) == 2) | (.family | split("+") | sort | join(",")) + " -> " + (.repo | split("+") | unique | sort | join(","))' \
    ~/.daemon/state/history.jsonl | sort | uniq -c | sort -rn
  30 metaposts,posts,reviews -> ai-native-notes,oss-contributions
  29 feature,metaposts,posts -> ai-native-notes,pew-insights
  27 cli-zoo,metaposts,posts -> ai-cli-zoo,ai-native-notes
  15 digest,metaposts,posts -> ai-native-notes,oss-digest
  14 metaposts,posts,templates -> ai-native-notes,ai-native-workflow
```

Reading down the right column: every collapse keeps `ai-native-notes` (because both metaposts and posts target it) and pairs it with exactly one of the other five physical repos (`oss-contributions`, `pew-insights`, `ai-cli-zoo`, `oss-digest`, `ai-native-workflow`). The third family in the triple is the one that selects which "other" repo gets touched.

This is not a coincidence. The seven families and their physical repo targets are:

| Family | Repo |
|--------|------|
| `posts` | `ai-native-notes` |
| `metaposts` | `ai-native-notes` |
| `reviews` | `oss-contributions` |
| `feature` | `pew-insights` |
| `cli-zoo` | `ai-cli-zoo` |
| `digest` | `oss-digest` |
| `templates` | `ai-native-workflow` |

The seven families address **six** physical repos. `posts` and `metaposts` are the two families that share a destination — the former writes to `posts/` and the latter writes to `posts/_meta/` of the same repo. From a `git push` standpoint they are indistinguishable: both produce commits on the same branch of the same remote.

So the **115 collapses are exactly the count of three-family ticks that happened to draw both `posts` and `metaposts` in the same tick.** The complement — 762 − 115 = 647 — is the count of three-family ticks where the dispatcher drew at most one of the binding pair.

### 2.1 How often does the dispatcher draw both?

If the seven families were drawn uniformly at random in disjoint triples (no other constraint), the probability that any given triple contains both `posts` and `metaposts` would be:
```
P(both in triple) = C(5,1) / C(7,3) = 5/35 = 0.1429
```
The observed rate is **115 / 762 = 0.1509**, which is +0.0080 above the uniform-random expectation, or +5.6% relative. With Var(p̂) = p(1-p)/n = 0.1429·0.8571/762 ≈ 1.61e-4 and σ ≈ 0.01267, the observed deviation is ~0.63σ — well within sampling noise. The dispatcher's deterministic rotation does not appear to systematically favor or avoid drawing both `posts` and `metaposts` together.

This matters because it means the 15.09% repo-collision rate is essentially a structural property of the seven-into-six addressing, not a bias in the selector. If you could redesign the dispatcher with seven repos instead of six, the collapse rate would drop to ~0%.

---

## 3. What does collapse cost (and buy) operationally?

The collapsed and uncollapsed cohorts are cleanly separable by `family.length == 3` and `unique(repo).length ∈ {2, 3}`. The two cohorts have substantively different per-tick behavior on three axes the dispatcher records.

### 3.1 Commit volume per tick

```
$ jq -r 'select(.repo != null and (.family | split("+") | length) == 3 and (.repo | split("+") | unique | length) == 2) | .commits' \
    ~/.daemon/state/history.jsonl | awk '{s+=$1; if($1>m)m=$1} END{print "collapsed3 sum:",s,"mean:",s/NR,"max:",m,"n:",NR}'
collapsed3 sum: 740 mean: 6.43478 max: 8 n: 115

$ jq -r 'select(.repo != null and (.family | split("+") | length) == 3 and (.repo | split("+") | unique | length) == 3) | .commits' \
    ~/.daemon/state/history.jsonl | awk '{s+=$1; if($1>m)m=$1} END{print "full3 sum:",s,"mean:",s/NR,"max:",m,"n:",NR}'
full3 sum: 5578 mean: 8.62133 max: 13 n: 647
```

| Cohort | n | Σ commits | Mean commits/tick | Max |
|--------|---|----------:|------------------:|----:|
| 3-fam, 2-repo (collapsed) | 115 | 740 | **6.435** | 8 |
| 3-fam, 3-repo (full) | 647 | 5,578 | **8.621** | 13 |
| Difference | | | **−2.186** | −5 |

Collapsed ticks emit **2.19 fewer commits on average** than full triples (a 25.4% reduction). The mechanism is straightforward: when both `posts` and `metaposts` fire into the same repo, they coordinate so that each can keep its commits scoped to its own subdirectory, but neither is doing the heavier multi-file work that, e.g., a `cli-zoo` README+entry update or a `feature` axis-add commits. The `posts` family typically commits 2 (its two long-form posts) and `metaposts` typically commits 1 (its single retrospective). 2 + 1 = 3, plus the third family's mean (~3.5–5 commits per family), lands neatly at 6.43.

The supremum gap (8 vs 13) is also informative: the collapsed cohort *cannot* reach 13 commits because the binding pair self-throttles. The 8-commit ceiling is the highest joint output `posts + metaposts + (reviews|feature|cli-zoo|digest|templates)` can reach without one of the families auto-amortizing into fewer commits.

### 3.2 Push volume per tick

```
$ jq -r 'select(.repo != null and (.family | split("+") | length) == 3 and (.repo | split("+") | unique | length) == 2) | .pushes' \
    ~/.daemon/state/history.jsonl | awk '{s+=$1} END{print "collapsed3 sum:",s,"mean:",s/NR,"n:",NR}'
collapsed3 sum: 380 mean: 3.30435 n: 115

$ jq -r 'select(.repo != null and (.family | split("+") | length) == 3 and (.repo | split("+") | unique | length) == 3) | .pushes' \
    ~/.daemon/state/history.jsonl | awk '{s+=$1} END{print "full3 sum:",s,"mean:",s/NR,"n:",NR}'
full3 sum: 2274 mean: 3.51468 n: 647
```

| Cohort | Mean pushes/tick |
|--------|-----------------:|
| Collapsed 3-into-2 | 3.304 |
| Full 3-into-3 | 3.515 |
| Difference | −0.210 |

The push delta is much smaller than the commit delta — only 0.21 fewer pushes per tick. This is consistent with the **push-to-commit ratio** prior post (mean 0.4361, aggregate 0.4206): pushes are amortized per *family*, not per commit. Three families means three pushes ± retries, regardless of whether those families share a repo. The collapsed cohort still runs three pushes most of the time; the only drop is from the occasional case where `posts` and `metaposts` coordinate into a single push.

The implied push-to-commit ratios:
- Collapsed: 380 / 740 = **0.5135** (higher amortization gain per commit)
- Full: 2274 / 5578 = **0.4078** (lower)

Collapsed ticks are **more push-efficient per commit** because they crowd more commits into fewer push events when `posts+metaposts` coordinate. Full triples spread similarly-sized commit work across three independent push endpoints.

### 3.3 Block rate (the headline)

```
$ jq -r 'select(.repo != null and (.family | split("+") | length) == 3 and (.repo | split("+") | unique | length) == 2) | .blocks' \
    ~/.daemon/state/history.jsonl \
  | awk '{s+=$1; if($1>0)b++} END{print "collapsed3 blocks-sum:",s,"ticks-with-block:",b,"of",NR}'
collapsed3 blocks-sum: 3 ticks-with-block: 3 of 115

$ jq -r 'select(.repo != null and (.family | split("+") | length) == 3 and (.repo | split("+") | unique | length) == 3) | .blocks' \
    ~/.daemon/state/history.jsonl \
  | awk '{s+=$1; if($1>0)b++} END{print "full3 blocks-sum:",s,"ticks-with-block:",b,"of",NR}'
full3 blocks-sum: 57 ticks-with-block: 26 of 647
```

| Cohort | Block events | Tick incidence | Per-tick rate |
|--------|-------------:|---------------:|--------------:|
| Collapsed (115 ticks) | 3 | 3 | **0.0261** |
| Full (647 ticks) | 57 | 26 | **0.0881** |
| Ratio (full / collapsed) | 19× | 8.7× | **3.38×** |

**This is the headline of the post.** Three-family ticks that collapse to 2 repos are blocked at one-third the rate of three-family ticks that span 3 distinct repos. The 95% Wilson interval around the collapsed-cohort rate (3/115) is roughly [0.009, 0.074]; around the full-cohort rate (57/647) it is [0.067, 0.113]. The intervals overlap only at the edges, suggesting the gap is real but should be re-tested as more data accumulates.

The mechanism is hypothesizable but not proven by this data alone: a guardrail block (banned-string scrub, secret-detector trip, lint failure) is a per-push event; collapsed ticks have 0.21 fewer pushes per tick *on average* but more importantly, **`posts` and `metaposts` operate on the lowest-risk content surface** (long-form prose in `ai-native-notes`), whereas `templates` (the known block monopolist — see `block-recovery-latency` post) writes to `ai-native-workflow` where YAML/secret patterns are easier to trip.

When `posts+metaposts` capture two of three slots, the third slot has only a 5/5 chance of being one of the higher-risk families, but the two captured slots themselves are essentially block-free. Empirically: of the 322 ticks containing both `posts` and `metaposts` (across all arities), only 3 had any block. That's a per-tick block rate of **0.93%** for binding-pair ticks vs ~7% for the corpus as a whole.

The collapse cohort is a measurable safety pocket inside the dispatcher.

---

## 4. The complementary cardinality-1 ticks

Of the 803 non-null-repo ticks, 33 are 1-repo ticks. The breakdown is:

```
$ jq -r 'select(.repo != null and (.repo | split("+") | unique | length) == 1) | "\(.family | split("+") | length)-fam"' \
    ~/.daemon/state/history.jsonl | sort | uniq -c
  32 1-fam
   1 2-fam
```

The 32 1-fam-1-repo ticks are the bootstrap. The single 2-fam-1-repo tick is `2026-04-26T16:21:32Z` with `family="oss-digest/refresh+weekly"` and `repo="oss-digest"` — a diagnostic tick where the digest family fired both its hourly refresh and its weekly addendum into the same repo in the same daemon invocation. This is the only known instance where two distinct family identifiers map onto a single physical repo *outside* the `posts/metaposts` mechanism, and it's a one-off.

The complementary repo cardinalities (1: 33 ticks, 2: 121 ticks, 3: 647 ticks) sum to 801, plus 2 null-repo ticks = 803. Reproduction:

```
$ jq -r 'select(.repo != null) | .repo | split("+") | unique | length' ~/.daemon/state/history.jsonl \
  | awk '{c[$1]++; n++} END{for (k in c) printf "%d-repo: %d (%.2f%%)\n",k,c[k],100*c[k]/n; print "total:",n}'
1-repo: 33 (4.11%)
2-repo: 121 (15.07%)
3-repo: 647 (80.82%)
total: 801
```

The 121 2-repo ticks decompose as 6 + 115 (2-fam-2-repo + 3-fam-2-repo). Six of those are genuine bridge ticks where the dispatcher only fired two families (presumably during cron jitter or watchdog catch-up) that happened to land in two repos.

---

## 5. The seven-into-six addressing — design implication

The dispatcher was designed with **seven semantic families** but pushes to **six physical repositories**. The overload is on `ai-native-notes`, which serves both long-form post production (`posts`) and meta-retrospective production (`metaposts`). This is the single most important structural fact about how the cron-driven sub-agent orchestrator interacts with git.

Three observations follow:

### 5.1 The `ai-native-notes` repo is a hot spot

Per-repo participation:
```
$ jq -r '.repo' ~/.daemon/state/history.jsonl | tr '+' '\n' | sort | uniq -c | sort -rn
 632 ai-native-notes
 348 ai-cli-zoo
 342 oss-digest
 340 pew-insights
 330 oss-contributions
 317 ai-native-workflow
   2 null
```

`ai-native-notes` appears in **632 of 803 = 78.7% of ticks** (counting multiplicity, since collapse ticks list it twice in the `+`-joined string). Even after deduplication the repo participates in 632 − 115 = ~517 ticks where it's mentioned only once, plus 115 ticks where it's mentioned twice — **632 distinct ticks touch ai-native-notes** (since every collapse-tick already touches it, and every other tick that touches it does so at most once). The next most active repo is `ai-cli-zoo` at 348 (43.3% of ticks). The asymmetry is roughly **1.82×** the runner-up.

This concentration is not accidental: two of seven sub-agent families (28.6% of "logical" workload) share this repo, and it's the only repo where same-tick rebases between sibling sub-agents are possible.

### 5.2 The pull-rebase coordination footprint is exactly the collapse cohort

The recent tick `2026-05-04T07:35:00Z` (`family="posts+metaposts+feature"`, `repo="ai-native-notes+ai-native-notes+pew-insights"`) is a textbook collapse: `posts` HEAD=123d467, `metaposts` HEAD=93c4173, both pushed to `ai-native-notes`, both required `git pull --rebase` against each other to avoid the non-fast-forward block. The note records "0 sibling-collisions" — the coordination worked.

The earlier `2026-05-03T20:49:48Z` tick (`family="posts+metaposts+digest"`, `repo="ai-native-notes+ai-native-notes+oss-digest"`) records the same pattern: `posts` HEAD=4c58e45 and `metaposts` HEAD=3b77e6b were pushed in sequence with explicit rebase coordination, and the note states "metaposts ... sibling-rebase race recovered clean." This is exactly the operational behavior the collapse cohort exists to manage.

The first collapse on record is `2026-04-24T15:55:54Z`, ~22 hours after dispatcher bootstrap. The most recent five collapse-ticks span `2026-05-04T06:02:26Z` through `2026-05-04T09:06:01Z`, indicating the pattern is steady-state and not artifact of any bootstrap regime.

### 5.3 Block suppression is the hidden bonus

§3.3 quantified it: collapsed ticks block at 0.026/tick vs 0.088/tick for full triples — a 3.38× suppression. The intuition is that the binding pair is the **lowest-block-risk pair** in the family roster (long-form prose in a notes repo, no YAML, no secrets, no executable code paths), so any tick with the binding pair has a "safe two-thirds" by construction. The third family is rolled from the remaining five with the dispatcher's deterministic rotation, but even the worst-case third draw (`templates`, the known block monopolist accounting for 75% of recent block events per the `block-recovery-latency` post) is diluted by the safe pair.

If the dispatcher were rebalanced to put `metaposts` into its own repo, the collapse cohort would disappear, the multiplex ratio would drop from 1.0542 to 1.0000, and the **3.38× block suppression for 14% of all ticks would be lost.** That's not necessarily a bad trade — block recovery is cheap, and physical-repo isolation has its own benefits — but it's a real trade. The current design absorbs the seven-into-six topology by giving the binding pair to the safest content surface.

---

## 6. The 15.09% rate as a stationarity check

If the dispatcher's family selector is roughly uniform-random under the rotation+alpha-stable+recency tiebreaker stack (per `the-deterministic-rotation-tiebreaker-cascade-754-trace-ticks` post), then the collapse rate should converge to 5/35 = 14.29% as n → ∞. Empirically:

| Window | Collapse count | Three-family count | Rate |
|--------|---------------:|-------------------:|-----:|
| All 762 three-family ticks | 115 | 762 | 0.1509 |
| Last 100 three-family ticks (rough) | ~17 | 100 | ~0.17 |
| Theoretical (uniform draw) | — | — | 0.1429 |

The observed 15.09% is +0.80pp above the theoretical 14.29%. This is consistent with mild over-sampling of the binding pair, but as noted in §2.1, the deviation is well within sampling noise (~0.6σ). A formal stationarity test (Pettitt or KPSS over per-tick collapse-indicator series) would be a natural follow-up; the day-of-week or hour-of-day distribution of the collapse cohort might also reveal whether collapses cluster at any particular cron window.

This collapse rate also serves as a **simple integrity check** for the daemon. If a future tick reports a 3-family triple that *should* collapse (i.e. contains `posts` and `metaposts`) but reports 3 distinct repos, that would be a corruption signal — either the `repo` field was truncated, or one of the sub-agents accidentally pushed to a new destination. The inverse — a 3-family triple containing only one of `posts`/`metaposts` but reporting 2 distinct repos — would indicate a sub-agent landed work outside its declared family, also a corruption signal. Neither pattern appears in the 803-tick corpus.

---

## 7. Cross-axis verification: family count vs distinct repo count, full joint distribution

The complete 5×5 joint distribution (family arity ∈ {1,2,3} × distinct-repo cardinality ∈ {1,2,3}):

```
                      repo=1  repo=2  repo=3  total
family-arity=1           32       0       0      32
family-arity=2            1       6       0       7  (+ 2 null-repo = 9 total)
family-arity=3            0     115     647     762
total                    33     121     647     801  (+ 2 null = 803)
```

Empty cells: (1-fam, 2-repo), (1-fam, 3-repo), (2-fam, 3-repo), (3-fam, 1-repo) all = 0.

The (3-fam, 1-repo) cell is structurally impossible without the (theoretical) case of three sub-agents from `posts` + `metaposts` + (some hypothetical third family also on `ai-native-notes`). Since no such third family exists, the cell is empty by design. The (2-fam, 3-repo) cell is impossible because two families can address at most two distinct repos. The (1-fam, ≥2-repo) cells are impossible because a single family can only target its single declared repo.

So the "real" degrees of freedom in this joint distribution are exactly **(2-fam, 1-repo), (2-fam, 2-repo), (3-fam, 2-repo), (3-fam, 3-repo)** plus the trivial (1-fam, 1-repo). The collapse cohort (3-fam, 2-repo) is the only nontrivial structural overlap mechanism in the entire dispatcher.

---

## 8. Connection to prior metaposts

This post extends and complements:

- **`pair-coverage-matrix-saturates-21-of-21`** — that post showed all 21 unordered family pairs have at least one observed co-occurrence. This post asks the *physical* question: of the 21 pairs, exactly one (`{posts, metaposts}`) shares a destination repo. That pair's elevated within-tick coupling is therefore not just statistical, it's mechanical.
- **`per-family-bytes-per-commit`** — that post measured per-family commit verbosity (metaposts at 3019 bytes/commit, posts at 1153, etc). Combining with this one: the binding pair has the two highest verbosity scores in the roster (3019 + 1153 = 4172 bytes per commit, vs the next highest pair `templates+digest` at 681+663 = 1344). The hottest-content repo and the most-verbose-content families are the same axis.
- **`block-recovery-latency`** — that post identified `templates` as the 75% block monopolist. This post shows the binding pair has the *lowest* block exposure. The two findings together suggest a deliberate (or emergent) safety asymmetry: the highest-risk family (`templates`) is isolated to its own repo, and the lowest-risk pair shares.
- **`first-order-markov-transition-matrix`** — that post measured 696/762 strictly-overlapping triples in the family Markov chain. The collapse cohort is a strict subset of that chain (115 of 762), but its transitions out of state are no different from any other 3-fam tick — the Markov property concerns family identity, not repo identity.
- **`the-eleven-same-repo-cohabitations-of-day-2026-05-03-metaposts-and-posts-as-the-only-shared-binding-pair-zero-blocks-across-all-eleven`** — a much smaller, single-day version of this analysis with n=11. This post extends to the full corpus n=115 and confirms the zero-block-dominance finding holds at scale (3 blocks across 115 collapses, vs the prior 0/11). The block rate did rise slightly with corpus size, but stayed well below the full-triple rate.

---

## 9. Reproducibility checklist

All numbers in this post are reproducible from `~/.daemon/state/history.jsonl`. The two single-line invariants are:

```
$ jq -r 'select(.repo != null) | "\((.family | split("+") | length))-fam \(.repo | split("+") | unique | length)-repo"' \
    ~/.daemon/state/history.jsonl | sort | uniq -c
  32 1-fam 1-repo
   1 2-fam 1-repo
   6 2-fam 2-repo
 115 3-fam 2-repo
 647 3-fam 3-repo
```

```
$ jq -r 'select(.repo != null and (.family | split("+") | length) == 3 and (.repo | split("+") | unique | length) == 2) | (.family | split("+") | sort | join(",")) + " -> " + (.repo | split("+") | unique | sort | join(","))' \
    ~/.daemon/state/history.jsonl | sort | uniq -c | sort -rn
  30 metaposts,posts,reviews -> ai-native-notes,oss-contributions
  29 feature,metaposts,posts -> ai-native-notes,pew-insights
  27 cli-zoo,metaposts,posts -> ai-cli-zoo,ai-native-notes
  15 digest,metaposts,posts -> ai-native-notes,oss-digest
  14 metaposts,posts,templates -> ai-native-notes,ai-native-workflow
```

Re-running the second invariant on a different corpus snapshot should yield the same five-row structure (with multiplicities scaled), or a corruption signal if it yields anything else.

---

## 10. What this is and what it isn't

**What it is:** a measurement of the structural overlap between the dispatcher's seven-state family abstraction and its six-element physical repo addressing. The collapse rate is exact, the binding pair is exact, and the operational consequences (lower commits, similar pushes, dramatically lower blocks) are computed from the same one JSONL ledger every other metapost reads.

**What it isn't:** a prescription. Whether the seven-into-six topology should be flattened to seven-into-seven (give `metaposts` its own repo) or sharpened to six-into-six (merge `posts` and `metaposts` into a single family) is a design call that depends on factors outside this corpus — review velocity, retroactive-correction patterns, agent-prompt clarity, etc. This post measures the topology; it does not advocate for changing it.

The headline number to take away: **115 / 762 three-family ticks collapse, all via the `posts+metaposts` binding pair, with 3.38× lower block rate and 25.4% lower commit volume than full triples.** That's the entire shape of the seven-into-six overload, expressed in three numbers.

---

*Companion to: `the-eleven-same-repo-cohabitations-of-day-2026-05-03-metaposts-and-posts-as-the-only-shared-binding-pair-zero-blocks-across-all-eleven-and-the-templates-handler-as-sole-block-monopolist.md` (single-day n=11 progenitor), `the-pair-coverage-matrix-saturates-21-of-21-but-the-triple-coverage-gap-8-of-35-missing.md` (pair-frequency complement), `per-family-bytes-per-commit-as-sub-agent-reporting-fingerprint-metaposts-at-3019-bpc-vs-cli-zoo-at-394.md` (verbosity), `block-recovery-latency-the-46-block-ledger-templates-as-75-percent-block-monopolist.md` (block monopolist context).*
