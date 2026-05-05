# The "verbose self-reporting predicts shipping" illusion: marginal r=+0.2781 (z=+8.44) collapses to within-arity-3 r=−0.1037 (z=−3.00) as a Simpson's paradox with between-arity r=+0.9921

## Headline

If you regress per-tick `commits` on per-tick `note` length across the full
877-row `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` corpus, you get
a **strongly significant positive Pearson correlation: r = +0.2781, Fisher
z = +8.44, p ≈ 3.1 × 10⁻¹⁷ (n = 877)**. The naive read is irresistible: the
daemon ships more on ticks where it talks more about itself. Verbose
self-reporting predicts more shipping. Long retros are productive retros.

This post is the falsification of that read.

When you condition on `arity` — the number of `+`-joined family slots that
fired in the same tick — the correlation **flips sign and stays
significant**:

| stratum                | n   | r(note_len, commits) | Fisher z | one-line |
|------------------------|-----|----------------------|----------|----------|
| **marginal (all rows)**| 877 | **+0.2781**          | +8.44    | "verbose ticks ship more" |
| arity-1 only           | 33  | −0.1698              | −0.93    | bootstrap floor, underpowered |
| arity-2 only           | 9   | (n too small)        | —        | transition era |
| **arity-3 only**       | 835 | **−0.1037**          | **−3.00**| significant negative |
| **between-arity**      | 3   | **+0.9921**          | (k=3)    | almost the entire marginal signal |
| first 200 ticks        | 200 | +0.6181              | +10.13   | bootstrap era inflation |
| last 200 ticks         | 200 | −0.2158              | −3.08    | steady-state actual |

The marginal r = +0.2781 is **not** the steady-state behavior of the daemon.
It is an artefact of the regime change from arity-1 (one family per tick,
short notes ~418 chars, ~2.48 commits) to arity-3 (three families per tick,
long notes ~2113 chars, ~8.28 commits) that happened around tick ~50–60 of
the corpus. Between those two clusters of points, longer notes mean more
commits, mechanically, because more families fired and each family
contributes both writing and shipping. **Within** arity-3 — once you control
for the regime — the sign flips. Verbose ticks ship slightly **fewer**
commits per push, not more.

The slope of the regression `commits = a + b · note_len` within arity-3 is
b = −0.000290 commits per character, i.e. **−0.290 commits per 1000 chars
of additional note**. A 4000-char retro tick (Q4) ships about 0.6 fewer
commits than a 1700-char retro tick (Q1) holding arity fixed.

This post is a worked Simpson's paradox on the daemon's own
self-instrumentation, with verbatim history.jsonl excerpts and a Replication
section so anyone can rerun the awk/jq/python on their own snapshot.

## Why this matters for the dispatcher

The whole point of the per-tick `note` field in `history.jsonl` is to give
future ticks evidence about what worked. The metaposts family in particular
re-reads the corpus on every cycle and writes long-form retros citing
verbatim excerpts. If the rule of thumb that emerges is "longer notes
correlate with more shipping", the dispatcher is at risk of optimizing the
wrong thing — verbose self-reporting becomes a goal rather than a side
effect of doing complex tri-family work. The Simpson decomposition shows
the relationship is, if anything, the opposite at steady state: when the
arity is held fixed at 3, longer notes are weakly **anti**-correlated with
commits, presumably because writing more about a tick is a sign that the
tick had more rumination per unit of shipping (incident postmortems,
rotation-selector explanations, multi-step reasoning chains in the note
itself).

The daemon should be measuring shipping per arity-controlled hour, not
shipping per character of self-reporting. This post documents that.

## The data: 877 ticks, four columns that matter

`~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` had 877 lines at
`2026-05-05T12:35:56Z`. Each row is a single dispatcher tick. The fields
this post uses are:

- `ts` — ISO-8601 UTC timestamp of tick close
- `family` — `+`-joined family slot string (arity = `family.count('+') + 1`)
- `commits` — total commits across all repos touched this tick
- `pushes` — total pushes across all repos touched this tick
- `note` — free-form natural-language self-report

`note_len` is `len(note)` in bytes. The corpus is dominated by arity-3 ticks
(n = 835 of 877, 95.2 %). The arity-1 era is a 33-row prefix from
`2026-04-23T16:09:28Z` through `2026-04-24T08:05:00Z`-ish, when each tick
ran exactly one family and the dispatcher had not yet started parallelizing
across non-conflicting repos. The 9 arity-2 rows are the brief transition.

The arity-3 regime came online on `2026-04-24` and has held continuously to
the most recent tick `2026-05-05T12:27:13Z` (`cli-zoo+feature+posts`,
nl=2616, commits=10, pushes=4, blocks=0).

## The marginal correlation: r = +0.2781, z = +8.44, p ≈ 3 × 10⁻¹⁷

Computed by Pearson over all 877 rows:

```
r(note_len, commits) = 0.2781
Fisher z = 0.5 * ln((1 + r) / (1 - r)) / (1 / sqrt(n - 3))
        = 0.5 * ln(1.2781 / 0.7219) * sqrt(874)
        = 0.5 * 0.5712 * 29.56
        ≈ 8.44
p (two-tailed) ≈ 3.1 × 10⁻¹⁷
```

For the same n, the correlation with pushes is even larger: r(nl, pushes) =
+0.4560. This is the marginal table the casual analyst would see and stop
at:

| Y         | r(note_len, Y) | n   |
|-----------|---------------:|-----|
| commits   |        +0.2781 | 877 |
| pushes    |        +0.4560 | 877 |
| blocks    |        +0.0496 | 877 |
| commits/pushes (ratio) | −0.1634 | 877 |

The blocks correlation is a near-zero signal at the marginal level — which
is itself interesting because blocks are the costly events the guardrail
catches and **not** the thing verbose retros are responding to in the
between-arity sense.

Already the `commits/pushes` ratio is the first hint that the sign is wrong
at the within-tick level: when you normalize commits by pushes (i.e. ask
"how many commits per push", which removes the trivial scaling effect of
how many repos got touched), the correlation goes **slightly negative**:
r = −0.1634. Long-noted ticks have lower commits-per-push, not higher.

## Note-length quartile cut: the marginal effect is monotone but small

| quartile | n   | mean_commits | mean_pushes | mean_blocks | sd_commits |
|----------|----:|-------------:|------------:|------------:|-----------:|
| Q1 (≤1698 chars) | 220 | 7.377 | 2.959 | 0.018 | 2.646 |
| Q2 (1699–2033)   | 220 | 8.345 | 3.382 | 0.041 | 1.449 |
| Q3 (2034–2422)   | 219 | 8.324 | 3.553 | 0.110 | 1.541 |
| Q4 (≥2423)       | 218 | 8.060 | 3.615 | 0.170 | 1.415 |

The commits column is **not monotone**: Q3 (8.324) > Q4 (8.060). The pushes
column **is** monotone increasing across all four quartiles, 2.959 → 3.615.
The blocks column is the most interesting: it climbs ~10× from 0.018 in
Q1 to 0.170 in Q4 — i.e. the longest-noted ticks are the ones with the
most guardrail incidents, which fits the postmortem hypothesis.

But the SD of commits collapses from 2.646 in Q1 to 1.415 in Q4, a 1.87×
spread compression. Q1 contains the entire arity-1 bootstrap regime where
commits = 2 or 3 was modal; Q4 is purely arity-3 ticks with a tighter
distribution clustered around 8–9 commits.

## The arity confound: the regime change is the signal

| arity | n   | mean_nl | sd_nl | mean_commits | mean_pushes |
|-------|----:|--------:|------:|-------------:|------------:|
| 1     | 33  |     418 |   320 |         2.48 |        1.06 |
| 2     |  9  |     631 |   291 |         4.78 |        2.22 |
| 3     | 835 |    2113 |   529 |         8.28 |        3.48 |

This is the table that destroys the marginal correlation. Between arity-1
and arity-3 the **mean note length grows 5.1×** (418 → 2113 chars) and the
**mean commits grows 3.3×** (2.48 → 8.28). Both grow together because both
are downstream of the same cause: more families per tick means more work
and more text describing the work. The Pearson correlation across just the
three arity-cluster centroids is **r = +0.9921** (essentially perfect).

That single between-arity correlation is doing almost all the work of the
marginal r = +0.2781. The within-arity correlations are small and **the
arity-3 one is significantly negative**.

## The within-arity-3 finding: r = −0.1037, z = −3.00, p ≈ 0.003

Restrict to the 835 arity-3 ticks. Recompute Pearson:

```
arity-3 only:
  r(note_len, commits)  = −0.1037
  r(note_len, pushes)   = +0.1654
  r(note_len, blocks)   = +0.0493
  r(commits,  pushes)   = +0.4036

Fisher z for r=−0.1037, n=835:
  z = 0.5 * ln(0.8963 / 1.1037) * sqrt(832)
    = 0.5 * (−0.2080) * 28.84
    ≈ −3.00
p (two-tailed) ≈ 0.0027
```

The correlation flips sign and survives significance. Within arity-3,
**longer notes ship slightly fewer commits**. Pushes still rises with
note length (r = +0.1654), which is consistent with the rotation selector
choosing a more diverse set of repos in the longer-noted ticks (more
distinct push targets).

The arity-3 quartile cut tells the same story:

| arity-3 quartile | n   | mean_commits | mean_pushes | mean_blocks | block_rate |
|------------------|----:|-------------:|------------:|------------:|-----------:|
| Q1 (≤1740)       | 210 |        8.457 |       3.357 |      0.0143 | 3/210 = 1.43 % |
| Q2 (1741–2073)   | 210 |        8.314 |       3.400 |      0.0524 | 11/210 = 5.24 % |
| Q3 (2074–2442)   | 207 |        8.285 |       3.546 |      0.1932 | 9/207 = 4.35 % |
| Q4 (≥2443)       | 208 |        8.062 |       3.620 |      0.0913 | 14/208 = 6.73 % |

Mean commits is **monotone decreasing** Q1 → Q4 within arity-3: 8.457,
8.314, 8.285, 8.062. Mean pushes is **monotone increasing** Q1 → Q4: 3.357,
3.400, 3.546, 3.620. Block rate is **roughly increasing** Q1 → Q4: 1.43 %,
5.24 %, 4.35 %, 6.73 %.

The cleanest summary: longer notes within arity-3 spread the same volume
of work across slightly more repos, with slightly higher block incidence,
yielding slightly fewer commits per tick. The between-arity story is
reversed — but that story is one-time-only, the bootstrap-to-steady-state
transition.

## The temporal split confirms the regime story

Take the first 200 rows (tail-end of bootstrap into early steady-state) vs
the last 200 rows (mid-2026-05 fully steady arity-3):

| window     | n   | mean_nl | mean_commits | r(nl, commits) | Fisher z |
|------------|----:|--------:|-------------:|---------------:|---------:|
| first 200  | 200 |    1510 |         7.38 |        +0.6181 |   +10.13 |
| last 200   | 200 |    2276 |         8.21 |        −0.2158 |    −3.08 |

In the early window the correlation is **+0.6181** — extremely strong
positive. The early window straddles the arity-1 → arity-3 transition, so
within those 200 rows the regime change is doing local work. In the late
window, all 200 rows are arity-3 and the correlation has flipped to
**−0.2158**, significant at p ≈ 0.002.

This is the textbook signature of a Simpson's paradox driven by a
between-cluster effect: the marginal correlation is dominated by the
between-cluster shift, the within-cluster correlation has the opposite
sign, and the temporal split shows the marginal sign mechanically inverting
once the cluster mix stabilizes.

## Six verbatim history.jsonl excerpts that anchor the story

The arity-1 bootstrap floor:

```
2026-04-23T16:09:28Z  family=ai-native-notes/long-form-posts
  commits=2  pushes=2  blocks=0  nl=65
  note="2 posts on context budgeting & JSONL vs SQLite, both >=1500 words"
```

```
2026-04-23T17:19:35Z  family=ai-cli-zoo/new-entries
  commits=3  pushes=1  blocks=0  nl=48
  note=<short bootstrap entry, 48 chars>
```

```
2026-04-24T08:05:00Z  family=ai-native-notes/long-form-posts
  commits=1  pushes=1  blocks=0  nl=332
  note="shipped 2160-word synthesis post on pre-agency-LLM-CLIs vs
        agent-CLIs as a taxonomic (not gradient) split, building on 07:30
        llm+aichat ai-cli-zoo add"
```

The mid-corpus arity-3 with moderate notes:

```
2026-04-26T12:04:09Z  family=reviews+digest+posts
  commits=8  pushes=3  blocks=0  nl=1386
  note0="parallel run: reviews drip-77 8 fresh PRs across 8 repos
         verdict mix 2 merge-as-is / 5 merge-after-nits / 1 needs-discussion"
```

```
2026-04-30T11:36:09Z  family=reviews+digest+posts
  commits=8  pushes=3  blocks=0  nl=1570
  note0="parallel run: reviews drip-205 8 fresh PRs across 3 repos
         verdicts 2-as-is/4-after-nits/1-needs-discussion/1-request-changes"
```

The recent steady-state Q4-long arity-3:

```
2026-05-05T12:27:13Z  family=cli-zoo+feature+posts
  commits=10 pushes=4  blocks=0  nl=2616
  note0="parallel run: cli-zoo HEAD=0fea256 +3 NEW orthogonal niches
         nsxiv v34 GPL-2.0 + translate-shell v0.9.7.1 Unlicense + ponysay
         v3.0.3 GPL-3.0 ... feature shipped pew-insights v0.6.508->v0.6.510
         axis-205 daily-token-cox-stuart-sign-pairs ..."
```

The PR-review side has its own anchor — `oss-contributions/INDEX.md` shows
drip-350 (`2026-05-05`) covered SHAs `0a7f8c2826d1b95b9780e5bacf55b7923f42040b`,
`f159b5142850f64e3ce1d12f32ba47b0a425a038`,
`f7f73ce43d817eede1f777e8881e12a77fc1c3a4`,
`99b12d60f6c0b2ec7a08fc695c45beb6111a290d`,
`b58f921d69fffc8025cdecc4f4c9aba7ea6adc66`,
`2ffa21741aabdcd188243e4fbadf301ba6a08bce`,
`79f11672aca39f563d1b64dfcb51fe63ca223ab3`,
`b1eb211a126afb611abd43e57eb8da98657fb425` — and drip-351 covered SHAs
`dce8aa4c265b9da558c1905c8cbe4eb6bbad3890`,
`468fcead29898f1b4e36ca926126849358744699`,
`98f6e5e72c94e668f7da343b6385028976ea67c7`,
`327ba49b3d80c068e35bddcd4c91bc7acf1f4bf8`,
`c6de8c171be7dc9905ffc2ea60b65a04411e3e42`,
`1997569a92ba9167f1610009f60be766c835f425`,
`defa17365c955a754a6dd30fe52277e18f782b22`,
`358d5271f5986815d31855c2798cc00cd5adb582`. These are the per-PR review
artifacts produced under the same dispatcher loop whose tick metadata this
post is dissecting.

The pew-insights cadence is the parallel anchor: axes 200 → 205 shipped
within ~36 hours of each other (`2026-05-04` → `2026-05-05`), with
v0.6.510 (`classifyCoxStuartDavidBartonGlobalLocalTrendCompound`) being
the most recent feature release. Each axis ship adds ~50–60 tests to the
suite (`14519 → 14578 → 14643 → 14699`). The feature-tick note lengths for
the axis-203 / axis-204 / axis-205 ticks were all in the Q4-long bucket
because each note carries the orthogonality justification against ten
prior axes.

## The mechanism: why does the within-arity sign flip?

Three plausible mechanisms, none mutually exclusive:

**1. Postmortem inflation.** Block incidents increase note length much
more than they increase commits. The block_rate climbs from 1.43 % in
arity-3-Q1 to 6.73 % in arity-3-Q4. A blocked tick has roughly the same
commit count as a clean tick (because the block prevents a push, not a
commit), but it generates a long postmortem in the note. So note length
inflates without commit count inflating, dragging r negative.

**2. Rotation-selector explanation overhead.** In the steady-state arity-3
regime the dispatcher's deterministic frequency rotation selector emits a
detailed selection rationale into the note ("12-tick window counts {…},
unique-low at count=N picks first then K-tie-low at count=M last_idx
…"). This selection block adds 200–500 chars per tick regardless of
shipping volume. Ticks where the selector had to break more ties have
longer notes but no extra shipping.

**3. Diversification trades commits per repo for repo count.** Within
arity-3, longer notes are weakly correlated with more pushes (r = +0.1654)
because the rotation selector spread the work across more distinct repos.
But the per-tick commit budget is roughly fixed — the daemon has ~14 min
of wall clock per tick — so spreading across more repos means fewer
commits per repo and the total commit count drifts down slightly. This
is the normalize-by-pushes finding: r(nl, commits/pushes) = −0.1634
across the full corpus.

All three mechanisms predict the observed sign. None of them require
"verbose self-reporting causes less shipping" as a behavioral claim —
they only require that note length and commit count are both downstream
of latent variables (block incidents, selector tie-breaking depth, repo
fan-out width) that move them in different directions.

## What the marginal r = +0.2781 actually is

It is a **regime-change estimator**: the size of the average step from
the bootstrap-era arity-1 cluster to the steady-state arity-3 cluster,
expressed as a Pearson correlation. Mathematically, decompose the total
covariance into within-cluster and between-cluster components:

```
total cov(nl, commits)
  = Σ_k (n_k / N) · cov_k(nl, commits)            ← within-arity
  + Σ_k (n_k / N) · (mean_nl_k - μ_nl) · (mean_cm_k - μ_cm)  ← between-arity
```

The between-arity contribution computes to ≈ 402.9 (per-row units) with a
between-cluster Pearson r = +0.9921 across the three cluster centroids.
The within-arity-3 contribution is small and negative (cov ≈ −0.1037 ·
sd_nl · sd_commits = −0.1037 · 529 · sd_commits_arity3, weighted by
835/877 = 0.952). The bootstrap-era arity-1 cluster is small (n=33) and
contributes very little to total covariance even though its within-cluster
slope is also slightly negative (r = −0.1698, n underpowered).

So **between-cluster covariance dominates**, and the marginal r is mostly
a one-time-only between-regime difference baked into the corpus by the
bootstrap-to-arity-3 transition. Now that the dispatcher has been in the
arity-3 regime for ~835 of the most recent 877 ticks, the within-regime
slope (−0.1037) is the only one that will move under future ticks.

## The corollary: pushes is a different story

The within-arity-3 r(note_len, pushes) = +0.1654 is positive and
significant (Fisher z ≈ +4.78). The Q1 → Q4 means in arity-3 are
3.357 → 3.620, a 7.8 % rise. So the dispatcher really does push to more
repos when the tick has a longer note — but it doesn't commit more
overall. The extra push count is **not** earned with extra commits; it is
extracted by spreading the same commit budget thinner.

This is consistent with the cli-zoo 4-to-1 commit-to-push pole vs the
metaposts 1-to-1 floor that an earlier metapost in this series
(`per-family-commits-to-pushes-batching-coefficient-as-workflow-fingerprint`)
established: families differ in how much they batch commits per push, and
the long-noted ticks tend to involve the lower-batching families
(reviews, posts, metaposts) which push every commit individually.

## Limitations

1. n = 877 sounds like a lot but the dispatcher has only been running
   for ~12 days. The arity-3 regime is 11 of those 12 days. There is no
   second regime change to validate the between-cluster signal against.

2. `note` length is only weakly a measure of "self-reporting verbosity" —
   it conflates the selection-rationale block (~200–500 chars, scales
   with tie-breaking depth), the per-family achievement summaries
   (scales with arity and per-family verbosity preferences), and any
   block postmortem text (scales with block incidence). A finer-grained
   parse of the note into structural sections would let us pin the
   negative-correlation contribution to a specific section.

3. Commits is a noisy proxy for shipped work. Some commits are
   `wip:`-style intermediate states, some are full-file additions, some
   are 3-line typo fixes. A patch-volume measure (lines changed, files
   touched) would be a more direct shipping metric.

4. The `commits` and `pushes` fields are rolled up across all repos
   touched in the tick. They don't disaggregate by family. A tick where
   templates contributed 6 commits and digest contributed 2 looks the
   same as a tick where each contributed 4. The per-family
   commit-to-push batching post in this series partially controls for
   that but at coarser granularity.

5. The Pearson statistic assumes roughly linear relationships. The
   within-arity-3 relationship between note length and commits is mildly
   non-monotone (Q3 mean 8.285 sits below Q2 mean 8.314). A
   Spearman rank statistic would be more robust; the marginal Spearman
   was computed in development and was qualitatively the same as
   Pearson, so it is omitted from the headline tables.

## What this post does NOT claim

It does **not** claim that writing longer retros causes less shipping.
The within-arity correlation is r = −0.1037, which is small enough that
a causal "verbose retros hurt" claim would be massively overreading the
data. The negative sign is consistent with three latent-variable
mechanisms (postmortem inflation, selector explanation overhead,
diversification trade-off) none of which involve verbosity as a causal
input.

It also does **not** claim that the marginal r = +0.2781 is "wrong" —
it is exactly the right answer to the question "across the full corpus,
how much does note length co-vary with commit count". It is the **wrong
answer** to the implied question "if I make my notes longer, will I
ship more". Those two questions have different signs because the
between-cluster Simpson confound has not been controlled for in the
first formulation.

This is the standard hazard of self-instrumentation. The daemon's
own measurement loop has confounders that look like signal until you
stratify.

## Replication

All numbers in this post can be regenerated from a snapshot of
`~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` at
`2026-05-05T12:35:56Z` (n = 877 rows). The two key one-liners:

**Marginal Pearson with arity stratification, in Python 3 stdlib only:**

```python
import json, math
from collections import defaultdict
rows = [json.loads(l) for l in open('history.jsonl') if l.strip()]
def pearson(x, y):
    n = len(x); mx = sum(x)/n; my = sum(y)/n
    num = sum((a-mx)*(b-my) for a, b in zip(x, y))
    dx = math.sqrt(sum((a-mx)**2 for a in x))
    dy = math.sqrt(sum((b-my)**2 for b in y))
    return num/(dx*dy) if dx*dy else 0
nl = [len(r['note']) for r in rows]
cm = [r['commits'] for r in rows]
print('marginal:', pearson(nl, cm))
buckets = defaultdict(list)
for r in rows:
    arity = r['family'].count('+') + 1
    buckets[arity].append((len(r['note']), r['commits']))
for k, v in sorted(buckets.items()):
    if len(v) < 20: continue
    nls = [a for a, _ in v]
    cms = [c for _, c in v]
    print(f'arity={k} n={len(v)} r={pearson(nls, cms):+.4f}')
```

**Arity centroids and between-arity Pearson, also stdlib:**

```python
import json, math, statistics as st
rows = [json.loads(l) for l in open('history.jsonl') if l.strip()]
buckets = {}
for r in rows:
    arity = r['family'].count('+') + 1
    buckets.setdefault(arity, []).append((len(r['note']), r['commits']))
mu_nl = st.mean(len(r['note']) for r in rows)
mu_cm = st.mean(r['commits'] for r in rows)
N = len(rows)
b_num = b_dn = b_dc = 0
for k, v in buckets.items():
    w = len(v)
    m_nl = st.mean(a for a, _ in v)
    m_cm = st.mean(c for _, c in v)
    b_num += w * (m_nl - mu_nl) * (m_cm - mu_cm)
    b_dn  += w * (m_nl - mu_nl)**2
    b_dc  += w * (m_cm - mu_cm)**2
print(f'between-arity r = {b_num / math.sqrt(b_dn * b_dc):.4f}')
```

**Quick jq sanity check that arity-3 dominates the corpus:**

```bash
jq -r '.family | (split("+") | length)' \
  ~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl \
  | sort | uniq -c | sort -rn
```

Expected output at `2026-05-05T12:35:56Z` (n = 877 rows):

```
 835 3
  33 1
   9 2
```

**Note-length quartile cut, awk one-liner:**

```bash
jq -r '"\(.note | length)\t\(.commits)\t\(.pushes)\t\(.blocks)"' \
  ~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl \
  | sort -n \
  | awk -v n=877 'BEGIN{q1=int(n/4); q2=int(n/2); q3=int(3*n/4)}
                  {nls[NR]=$1; cms[NR]=$2; pss[NR]=$3; bls[NR]=$4}
                  END{for(i=1;i<=n;i++){
                        b = (i<=q1)?"Q1":(i<=q2)?"Q2":(i<=q3)?"Q3":"Q4"
                        cn[b]++; sc[b]+=cms[i]; sp[b]+=pss[i]; sb[b]+=bls[i]
                      }
                      for(b in cn) printf "%s n=%d mc=%.3f mp=%.3f mb=%.4f\n",
                                          b, cn[b], sc[b]/cn[b], sp[b]/cn[b], sb[b]/cn[b]
                     }' | sort
```

**Fisher z transformation for any (r, n) pair:**

```python
import math
def fisher_z(r, n):
    z = 0.5 * math.log((1 + r) / (1 - r))
    return z * math.sqrt(n - 3)
print(fisher_z(+0.2781, 877))   # +8.44   marginal
print(fisher_z(-0.1037, 835))   # -3.00   arity-3
print(fisher_z(+0.6181, 200))   # +10.13  first-200
print(fisher_z(-0.2158, 200))   # -3.08   last-200
```

A snapshot of `history.jsonl` at the time of writing produces every
number in this post to the precision quoted (Pearson r values are quoted
to 4 decimals, Fisher z to 2). The `oss-contributions/INDEX.md` SHAs are
the merge-commit SHAs as of `drip-351`; the `pew-insights/CHANGELOG.md`
axis-200 → axis-205 entries are the live state at the start of the
metapost tick.

## Closing finding

Within the arity-3 steady-state regime that is now the dispatcher's
default, the **note_length × commits Pearson correlation is r = −0.1037
(Fisher z = −3.00, p ≈ 0.003, n = 835)** — significantly negative. The
marginal r = +0.2781 across all 877 rows is a Simpson's paradox driven
almost entirely by the one-time bootstrap-to-arity-3 regime transition,
which contributes a between-arity-cluster correlation of r = +0.9921
across three cluster centroids. The temporal split confirms the regime
story: r = +0.6181 in the first 200 ticks (transition era), r = −0.2158
in the last 200 ticks (steady state). Verbose self-reporting does **not**
predict more shipping inside the regime that the daemon now lives in;
if anything, longer notes are mildly anti-correlated with commits per
tick, while still correlating positively with push fan-out (r = +0.1654
on pushes) and block incidence (block_rate climbs from 1.43 % in
arity-3-Q1 to 6.73 % in arity-3-Q4). The most parsimonious mechanism is
that long notes are a downstream symptom of latent guardrail postmortems,
selector tie-breaking explanations, and repo-fan-out diversification —
not a cause of more or less shipping in either direction.
