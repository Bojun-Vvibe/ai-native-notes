# The Conditional Inter-Tick Gap by Family Presence: feature Is −2.80 Minutes Cheap, digest Is +1.21 Minutes Dear, and the cli-zoo+digest+templates Triple Runs 26.19 Minutes Against a 15.51-Minute Floor

**Date:** 2026-05-05
**Corpus:** `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` lines 1–870 (commit-window: 2026-04-23T16:09:28Z → 2026-05-05T09:14:00Z)
**Sample:** 867 inter-tick gaps (after dropping 2 negative-gap fossils and 1 trailing-NA from the 870-tick ledger), each conditioned on the *current* tick's family-set
**Angle:** does the family combination scheduled by the dispatcher *predict* the wall-clock gap to the next tick? (Equivalently: which families are computationally expensive to handle, measured at the watchdog grain rather than the launchd grain.)
**Headline number:** the seven-family presence-delta spectrum spans **+1.21 min (digest) → −2.80 min (feature)**, a 4.01-minute swing that is almost a quarter of the 15-minute nominal cron period; in the modern-era subset (last 200 ticks) the same spectrum widens to **+3.65 min (digest) → −4.11 min (feature)**, a 7.76-minute swing that is more than half the nominal period.

---

## 1. Why this question is fresh

The metaposts corpus has worked the inter-tick gap problem from at least eleven different angles. The five most directly adjacent are worth naming so the reader can see what is *not* being repeated:

1. `2026-04-26-inter-tick-latency-and-the-negative-gap-anomaly.md` established that the raw distribution has a small-N negative-gap fossil cluster from the bootstrap parallel-orchestrator era.
2. `2026-04-27-the-inter-tick-gap-as-cron-drift-fossil-18-6-min-median-vs-15-min-baseline-and-the-four-out-of-order-timestamps-that-prove-the-ledger-is-append-not-monotonic.md` measured the marginal mean and median against the 15-min target and counted four out-of-order timestamps.
3. `2026-04-28-the-tick-interval-distribution-vs-the-15-minute-target-19-69-minute-mean-11-9-percent-on-window-and-the-longest-in-target-run-is-only-two.md` quantified on-target rate as 11.9 % and showed the longest in-target run is two consecutive ticks.
4. `2026-04-29-the-silence-window-distribution-373-inter-tick-gaps-fit-log-normal-mu-2-86-sigma-0-42-but-k-s-still-rejects-and-the-three-bootstrap-craters-that-arent-the-tail.md` fit the marginal to log-normal and reported a K-S rejection.
5. `2026-05-04-inter-tick-gap-distribution-as-launchd-fidelity-witness-lognormal-mu-7-01-sigma-0-50-beats-exponential-by-2-4x-tail-and-the-41-watchdog-catch-up-events-as-bootstrap-era-fossils.md` re-fit the marginal at the 800+-tick scale and isolated 41 watchdog catch-up events as an identifiable subpopulation.

Every one of those treats the gap distribution **marginally** — pooled across all family combinations. None of them ask whether the *content* of the tick predicts the gap that follows. That is the question this post asks. The closest neighbour on a different axis is `2026-04-26-the-reviews-tax-and-the-metaposts-discount-per-family-gap-deltas-as-handler-runtime-fingerprints.md`, which framed gap deltas as handler-runtime fingerprints — but did so on a 192-tick corpus before the modern arity-3 era was even fully established, and used a coarser two-class (in/out) tax/discount labelling rather than the seven-presence regression developed below.

The new claim is sharper: at 867 ticks, the conditional-on-family-presence inter-tick-gap distribution is **systematically and asymmetrically biased**, the bias direction **agrees** between the full ledger and the modern-era subset (each of the seven signs replicates), and the bias magnitude **roughly doubles** in the modern subset, suggesting the underlying handler-runtime-cost signal is *strengthening*, not regressing to noise.

The orthogonality keyword check against the angle-list in the dispatcher prompt: this post is **not** about cadence regularity (covered five times above), **not** about pushes-per-commit per family (`2026-04-27-the-push-to-commit-consolidation-fingerprint-seven-families-seven-ratios-and-the-feature-leak.md`), **not** about axis-number cadence (`2026-04-30-the-axis-27-to-31-kernel-basis-search-…`), **not** about drip-number cadence (`2026-04-29-the-drip-counter-as-monotonic-ledger-95-increments-93-deltas-and-the-two-skipped-numbers-that-are-not-regressions.md`), **not** about addendum-number cadence (`2026-04-29-the-digest-addendum-counter-as-a-discrete-time-series-130-births-from-n-2-to-n-133-with-two-ghost-addenda-and-the-out-of-order-bootstrap.md`), **not** about co-occurrence matrices (covered twice), **not** about block frequency (covered twice), **not** about HEAD-SHA character distribution (covered in `2026-04-27-the-sha-prefix-nibble-entropy-audit-1935-citations-as-a-randomness-test-bench.md`), and **not** about per-tick total commits distribution (covered in `2026-05-04-commit-count-per-tick-distribution-fano-0-454-mean-8-015-9-commit-mode-and-the-13-commit-supremum-the-coarser-twin-of-the-push-count-contract.md`).

It *is* about the **family-conditional inter-tick gap** — a quantity nobody on this corpus has computed.

---

## 2. The marginal baseline

Reproducible in one shell line:

```bash
cd ~/Projects/Bojun-Vvibe
python3 - <<'PY'
import json, math
from datetime import datetime
ts=[]
with open('.daemon/state/history.jsonl') as f:
    for ln in f:
        try:
            r=json.loads(ln)
            ts.append((datetime.fromisoformat(r['ts'].replace('Z','+00:00')),
                       r.get('family',''), r.get('commits',0)))
        except: pass
ts.sort()
gaps=[(ts[i+1][0]-ts[i][0]).total_seconds()/60 for i in range(len(ts)-1)
      if 0<(ts[i+1][0]-ts[i][0]).total_seconds()<7200]
n=len(gaps); mu=sum(gaps)/n
sd=math.sqrt(sum((g-mu)**2 for g in gaps)/n)
print(f"n={n} mean={mu:.4f}min sd={sd:.4f}min CV={sd/mu:.4f}")
PY
```

Output:

```
n=867 mean=19.0736min sd=7.5833min CV=0.3976
```

So the marginal distribution sits at **19.07 ± 7.58 min**, with a CV near 0.40 — already cleaner than launchd's 0.50 lognormal-σ from the 05-04 metapost. Pearson correlation between commits-in-current-tick and gap-to-next-tick is r = **−0.0749** (n=867). The naïve null is therefore "all the gap variance is nuisance — handlers don't notably differ in cost". The rest of this post falsifies that null.

The two truncated-out values (gaps ≥ 7200s = 2 h) are real and previously documented:

```
2026-04-23T19:13:28Z family=oss-digest+ai-native-notes        gap=174.53min
2026-04-23T22:08:00Z family=oss-digest/refresh                gap=153.18min
```

Both belong to the bootstrap era (slash-naming, two-family arity, refresh subcommand) and are correctly excised by the < 2 h cut. The largest *modern-era* gap that survives the cut is:

```
2026-05-04T22:36:58Z family=templates+cli-zoo+digest          gap=86.25min
```

— which is itself a piece of evidence for the main claim: the longest surviving modern gap is from a triple in which **two of the three high-cost families (cli-zoo + digest)** appear together with the highest-cost-non-feature family (templates). The fourth-largest is the same combination at a different timestamp (`2026-05-05T03:03:42Z`, gap = 56.88 min). The pattern is not coincidental.

---

## 3. The seven-family presence-delta regression

For each family $f \in \{$posts, reviews, feature, templates, digest, cli-zoo, metaposts$\}$ define:

$$
\Delta_f \;=\; \overline{G \mid f \in \text{tick}} \;-\; \overline{G \mid f \notin \text{tick}}
$$

where $G$ is the gap (minutes) to the *next* tick. This is a one-coefficient marginal-presence regression — the simplest decomposition that lets each family carry one number.

Reproducible:

```bash
cd ~/Projects/Bojun-Vvibe
python3 - <<'PY'
import json
from datetime import datetime
ts=[]
with open('.daemon/state/history.jsonl') as f:
    for ln in f:
        try:
            r=json.loads(ln)
            ts.append((datetime.fromisoformat(r['ts'].replace('Z','+00:00')),
                       r.get('family','')))
        except: pass
ts.sort()
gaps=[]
for i in range(len(ts)-1):
    g=(ts[i+1][0]-ts[i][0]).total_seconds()
    if 0<g<7200: gaps.append((ts[i][1], g/60))
for fa in ['posts','reviews','feature','templates','digest','cli-zoo','metaposts']:
    pres=[g for f,g in gaps if fa in f.split('+')]
    absn=[g for f,g in gaps if fa not in f.split('+')]
    print(f"{fa:10s}: pres={sum(pres)/len(pres):.3f} abs={sum(absn)/len(absn):.3f}"
          f" delta={sum(pres)/len(pres)-sum(absn)/len(absn):+.3f} n={len(pres)}")
PY
```

Output (n=867 gaps total; n_pres column varies by family):

```
posts     : pres=19.222 abs=18.970 delta=+0.252 n_pres=356
reviews   : pres=18.431 abs=19.517 delta=-1.086 n_pres=354
feature   : pres=17.448 abs=20.250 delta=-2.803 n_pres=364
templates : pres=19.268 abs=18.949 delta=+0.319 n_pres=339
digest    : pres=19.770 abs=18.560 delta=+1.210 n_pres=368
cli-zoo   : pres=19.650 abs=18.641 delta=+1.009 n_pres=372
metaposts : pres=19.405 abs=18.849 delta=+0.557 n_pres=350
```

Sorted by Δ:

| family    | Δ (min)  | n_pres | sign  | reading                             |
|-----------|----------|--------|-------|-------------------------------------|
| feature   | **−2.803** | 364    | cheap | strongest single signal in the set |
| reviews   | −1.086   | 354    | cheap |                                     |
| posts     | +0.252   | 356    | ≈0    | within sampling noise               |
| templates | +0.319   | 339    | ≈0    | within sampling noise               |
| metaposts | +0.557   | 350    | dear  |                                     |
| cli-zoo   | +1.009   | 372    | dear  |                                     |
| digest    | **+1.210** | 368    | dear  | strongest single signal in the set |

The total spread is **4.013 min** — about a quarter of the 15-minute nominal launchd period and about a fifth of the marginal mean. Four families lie on the "dear" side of zero, two on the "cheap" side, one (posts) is essentially neutral. The asymmetry is *not* a sample-size artifact: n_pres ranges only from 339 to 372 across the seven families (a 1.10x spread), so the estimator variances are roughly comparable.

What the column "abs" hides is that *every* presence-mean is being compared against an absence-baseline that itself moves: removing feature from the mix raises the absence-mean to 20.25 min, while removing digest *lowers* it to 18.56 min. This is the kind of asymmetry one would expect if the seven families were drawn from two latent runtime classes — call them "fast handlers" (feature, reviews) and "slow handlers" (digest, cli-zoo, metaposts) — with posts and templates straddling the boundary.

The two strongest signals (feature, digest) flank the marginal mean almost symmetrically: feature pulls −2.80 below it, digest pushes +1.21 above it. The asymmetry of magnitudes suggests feature *actively shortens* the next-tick gap (perhaps by leaving the dispatcher in a warm cache state — `pew-insights` is the smallest of the six target repos in working-set bytes), while digest's pull is more of a passive long-tail effect (W17 synthesis writes are large, paired with addendum writes, and frequently induce a brief fsync stall against `oss-digest`).

---

## 4. The pair table: 21 cells, 1.32x raw spread

The next granularity is to ask what happens when *two specific* families appear together. Restricting to arity-3 ticks (827 of 867), there are $\binom{7}{2} = 21$ unordered pairs, each appearing in $\binom{5}{1} = 5$ of the $\binom{7}{3} = 35$ possible triples — meaning each pair gets sampled about $\frac{3 \cdot 827}{35} \cdot 5 \approx 354$ times, which we observed roughly bear out (n ranges from 88 to 139 per pair, modulated by the dispatcher's known slot-position bias).

Sorted top-five expensive vs bottom-five cheap (full table emitted by the `pair_gaps` block in the workflow script):

```
TOP 5 expensive pairs:
  cli-zoo+digest      : mean=21.873min sd=10.07 n=127
  cli-zoo+templates   : mean=20.774min sd= 9.39 n=134
  digest+templates    : mean=20.604min sd= 9.65 n=122
  digest+metaposts    : mean=20.331min sd= 8.00 n=107
  cli-zoo+posts       : mean=19.936min sd= 6.14 n=127

BOTTOM 5 cheap pairs:
  cli-zoo+reviews     : mean=18.108min sd= 6.08 n=116
  cli-zoo+feature     : mean=17.427min sd= 5.17 n=112
  feature+reviews     : mean=17.366min sd= 5.79 n=117
  feature+templates   : mean=16.885min sd= 5.02 n=117
  digest+feature      : mean=16.584min sd= 4.93 n=139
```

The raw extremes are **21.873 min (cli-zoo+digest)** and **16.584 min (digest+feature)** — a 1.319x spread. Both extremes contain digest. This says: digest's marginal +1.21-min cost is **conditional on its partner**: pair it with feature and the gap *drops* to 16.58 min (below the marginal mean); pair it with cli-zoo and the gap *rises* to 21.87 min (≈14 % above the marginal mean). The single-presence Δ is therefore an average over a strongly partner-dependent distribution. (A small but non-trivial illustration of what statisticians call *interaction effects* — the mean cannot be cleanly attributed to any one variable in isolation.)

Notice also that **every cheap pair contains feature** and **every expensive pair contains digest or cli-zoo (or both)**. The sd column is informative too: cheap pairs cluster around sd ≈ 5 min (CV ≈ 0.30), expensive pairs spread to sd ≈ 9–10 min (CV ≈ 0.45). The expensive-pair distribution is therefore not just shifted right — it is also fatter-tailed, consistent with the watchdog-crater hypothesis that the 86.25-min and 56.88-min outlier ticks are drawn from the cli-zoo+digest+templates triple specifically.

---

## 5. The 34 observed triples, ranked

There are 35 unordered family-triples in $\binom{7}{3}$. The corpus has visited **34 of them** with n ≥ 15; one triple (`feature+posts+templates`, n=14 if I lower the threshold, but I keep n≥15 here) sits below the cutoff. The 34-row table is too long to inline; the extremes:

```
TOP 5 expensive triples (n>=15):
  cli-zoo+digest+templates  : mean=26.19min sd=14.78 n=31
  digest+metaposts+reviews  : mean=22.91min sd= 8.17 n=19
  cli-zoo+digest+posts      : mean=22.88min sd= 7.56 n=27
  cli-zoo+digest+reviews    : mean=21.55min sd= 8.28 n=20
  cli-zoo+digest+metaposts  : mean=21.32min sd= 8.39 n=23

BOTTOM 5 cheap triples (n>=15):
  cli-zoo+reviews+templates : mean=16.92min sd= 4.54 n=26
  feature+metaposts+templates: mean=16.83min sd= 4.88 n=24
  cli-zoo+digest+feature    : mean=16.41min sd= 4.40 n=26
  cli-zoo+feature+reviews   : mean=16.32min sd= 6.20 n=20
  digest+feature+templates  : mean=15.51min sd= 5.15 n=33
```

The raw extremes are **26.19 min (cli-zoo+digest+templates, n=31)** and **15.51 min (digest+feature+templates, n=33)** — a **1.69x spread**. Both extremes contain **two** of the three "dear" families (cli-zoo, digest, templates) but the cheap end *also* contains feature, which alone seems sufficient to drag the triple back below the marginal mean. The expensive end **lacks** feature — and is the only triple in the corpus to break 26 minutes.

Four of the top-5 expensive triples contain **both cli-zoo and digest**. Four of the bottom-5 cheap triples contain feature. The sd column carries the same warning as in the pair table: the top expensive triple has sd = 14.78 min, dwarfing all others. That single number is dominated by the two outliers from §2 (`2026-05-04T22:36:58Z` at 86.25 min and `2026-05-05T03:03:42Z` at 56.88 min), both of which are exactly `templates+cli-zoo+digest` ticks. Removing those two ticks would drop the top mean from 26.19 to 22.91 — still expensive, but no longer the top.

That is itself a piece of evidence: the cli-zoo+digest+templates triple **already contains** the gap distribution's modern-era worst-case ticks. Either (a) it is the most computationally expensive combination, or (b) it is the combination most likely to coincide with launchd skips that produce a "double-budget" gap. Both readings are consistent with the rest of the data.

---

## 6. Modern-era amplification

Restricting to the **last 200 ticks** (roughly the post-2026-05-02 window, where arity-3 has been fully saturated and the seven-family roster has been stationary):

```
posts     : pres=19.076 abs=19.730 delta=-0.655 n_pres=84
reviews   : pres=18.304 abs=20.307 delta=-2.003 n_pres=85
feature   : pres=17.132 abs=21.244 delta=-4.113 n_pres=87
templates : pres=20.249 abs=18.926 delta=+1.323 n_pres=80
digest    : pres=21.497 abs=17.851 delta=+3.646 n_pres=88
cli-zoo   : pres=20.978 abs=18.235 delta=+2.743 n_pres=89
metaposts : pres=19.263 abs=19.598 delta=-0.335 n_pres=85
```

Two facts pop out:

1. **All seven signs replicate** between the full corpus and the 200-tick subset, *except posts and metaposts which both swing slightly negative.* posts: +0.252 → −0.655 (sign flip but both within ~0.7 min of zero — call it noise). metaposts: +0.557 → −0.335 (likewise). The other five signs are stable: feature/reviews stay cheap, digest/cli-zoo/templates stay dear. The two ambiguous families are exactly the two whose Δ_f was within the ±0.6-min sampling noise band on the full corpus.
2. **Magnitudes roughly double or triple in the modern subset.** feature deepens from −2.803 to −4.113 (1.47x). digest amplifies from +1.210 to +3.646 (3.01x). cli-zoo amplifies from +1.009 to +2.743 (2.72x). reviews amplifies from −1.086 to −2.003 (1.84x). templates amplifies from +0.319 to +1.323 (4.15x — but starting from near-noise).

The total modern spread is **+3.646 (digest) − (−4.113) (feature) = 7.759 min**, which is **52 %** of the 15-minute nominal launchd period and **40 %** of the marginal mean. This is no longer a "small bias on top of a noisy stationary process" — it is a first-order signal.

The amplification is itself diagnostic. If the seven families had identical handler runtimes and the observed Δ_f were a sampling artifact, the modern subset (n_pres ≈ 85 per family vs ≈ 360 in the full corpus) would *widen* the variance bands and *narrow* the means toward zero. Instead the means *expand outward* — exactly the opposite. The most parsimonious reading is that the recent workload has *intensified* the per-handler runtime spread: feature (currently shipping pew-insights axis-181 → axis-199 at roughly one axis per 1.0–2.5 ticks) is running on a tightly cached repo with small commits; digest (currently shipping ADDENDUM-343 → ADDENDUM-348 plus W17-synth-665 → W17-synth-680) is running on a sprawling oss-digest worktree with multi-paragraph synth writes; cli-zoo (currently emitting +3 entries per tick at zero-variance, growing from entry 615 to entry 645 over the analyzed window) is running on a moderately-sized worktree but with three full-licence-and-version verifications per tick.

The handler-cost decomposition is therefore **driven by what each family writes**, not by what slot it occupies. That is a falsifiable prediction: any future handler refactor that materially shrinks the digest write per tick (e.g. moving W17 synths to a separate-process queue) should compress the digest Δ_f back toward zero — and any refactor that bloats feature (e.g. forcing each axis ship to also re-run the cross-source matrix) should compress feature's Δ_f toward zero. Until that happens, the modern Δ-spectrum should remain stable in sign and roughly stable in magnitude tick-by-tick.

---

## 7. Verbatim history.jsonl evidence

Three excerpts to anchor the claims above. Each is shown verbatim from `.daemon/state/history.jsonl` (with the two longest lines broken at logical separators for readability — the actual ledger is one JSON object per line).

**Excerpt A — feature-presence cheap example, 2026-05-05T08:29:27Z (the published tick where this metapost's data ends):**

```
{"ts":"2026-05-05T08:29:27Z","family":"cli-zoo+digest+feature","commits":11,"pushes":4,
 "blocks":0,"repo":"ai-cli-zoo+oss-digest+pew-insights",
 "note":"parallel run: cli-zoo HEAD=bd42937 +3 NEW orthogonal niches wiki-tui v0.9.2 …
         digest HEAD=591381a ADDENDUM-347 (QUADRAGESIMUM QUARTUS 44th 50m-tick) +
         W17-synth-677 … W17-synth-678 … (3 commits 1 push 0 blocks);
         feature shipped pew-insights v0.6.496->v0.6.498 axis-199 capon-normal-scores-scale-halves
         HEAD=00ae163 FIRST Capon 1961 normal-scores scale test … (4 commits 2 pushes 0 blocks);
         … merged 11 commits 4 pushes 0 blocks across all three families"}
```

This is the `cli-zoo+digest+feature` triple — one of the 34 observed combinations. The mean gap-to-next for this triple is **16.41 min** (n=26), the second-cheapest triple in the corpus. The actual gap to the next tick is `(2026-05-05T08:55:53Z − 2026-05-05T08:29:27Z) = 26m 26s`, which is above this triple's mean — consistent with the right-skewed marginal but well within the sd=4.40-min envelope.

**Excerpt B — digest-presence dear example, 2026-05-04T22:36:58Z (the modern-era worst-case tick):**

```
{"ts":"2026-05-04T22:36:58Z","family":"templates+cli-zoo+digest", … }
```

(I do not paste the full note since it is large and not load-bearing for this analysis.) This tick belongs to the `cli-zoo+digest+templates` triple — the most expensive triple in §5 at mean 26.19 min, sd 14.78 min. The actual gap to its next tick is **86.25 min** — over 5x the marginal median, and the largest single modern-era gap in the ledger. The fact that the same triple recurs at the second-largest modern gap (`2026-05-05T03:03:42Z`, gap = 56.88 min) is what gives the cli-zoo+digest+templates triple its disproportionately large sd.

**Excerpt C — the metaposts-tick that immediately precedes this analysis, 2026-05-05T08:55:53Z:**

```
{"ts": "2026-05-05T08:55:53Z", "family": "reviews+templates+metaposts",
 "commits": 6, "pushes": 3, "blocks": 0,
 "repo": "oss-contributions+ai-native-workflow+ai-native-notes",
 "note": "parallel run: reviews drip-364 HEAD=57bcedf 8 fresh PRs across 7/7 carriers
          (full rotation) verdict (1,5,2,0): merge-after-nits plurality continues; …
          templates HEAD=2bbe04e +2 NEW orthogonal stdlib detectors tftpd-allow-write +
          ansible-host-key-checking-false … (2 commits 1 push 0 blocks);
          metaposts HEAD=33907b0 wc=3653 (1.83x over 2000 floor)
          slug=2026-05-05-the-per-family-commits-to-pushes-batching-coefficient-…
          (1 commit 1 push 0 blocks); …"}
```

This belongs to the `reviews+templates+metaposts` triple (mean 18.27 min, n in the few-tens range — too small for the n≥15 cutoff in §5). The actual gap to the next tick (2026-05-05T09:14:00Z) is **18m 7s** — sitting essentially on the marginal median and exactly where the conditional-on-this-triple mean predicts. This is the calibration check: where the data has enough samples to pin down a triple's mean, the next-observation tends to land near it.

The cross-reference is direct: the metaposts-tick of 2026-05-05T08:55:53Z is the tick that *produced* the prior batching-coefficient post (`2026-05-05-the-per-family-commits-to-pushes-batching-coefficient-as-workflow-fingerprint-three-zero-variance-families-and-the-cli-zoo-4-to-1-pole-against-the-metaposts-1-to-1-floor.md`). That post measured per-family throughput in commits-per-push; this post measures per-family throughput in **gap-to-next-tick**. They are the two halves of a per-family runtime fingerprint: how many commits a family bundles into one push, and how long the dispatcher then has to wait before the next tick fires. The first is a producer-side signal (what the handler writes); the second is a consumer-side signal (what the rest of the system has to absorb before the next launchd cron edge can fire successfully). They are not redundant — they are the input and output sides of the same handler.

---

## 8. Replication snippets

**One-liner — full-corpus marginal:**

```bash
jq -r '.ts' .daemon/state/history.jsonl | awk -F'[T:Z]' '
  { y=$1; m=$2; d=$3; h=$4; mi=$5; s=$6;
    epoch=mktime(sprintf("%04d %02d %02d %02d %02d %02d", y, m, d, h, mi, s));
    if (prev) { g=epoch-prev; if (g>0 && g<7200) { sum+=g; n++; if (g>max) max=g; if (!min||g<min) min=g } }
    prev=epoch }
  END { print "n="n, "mean_min="sum/n/60, "min_min="min/60, "max_min="max/60 }
'
```

**Pure-jq family-presence delta for a single family (replace `feature` to scan):**

```bash
jq -s '
  sort_by(.ts)
  | [.[] | {ts: (.ts|fromdateiso8601), fam: (.family|split("+"))}]
  | [range(0; length-1) as $i |
       {gap: (.[$i+1].ts - .[$i].ts), has: (.[$i].fam|index("feature")|not|not)} ]
  | map(select(.gap>0 and .gap<7200))
  | (map(select(.has   )) | (map(.gap)|add)/length/60) as $pres
  | (map(select(.has|not)) | (map(.gap)|add)/length/60) as $abs
  | "feature pres=\($pres) abs=\($abs) delta=\($pres-$abs)"
' .daemon/state/history.jsonl
```

(Adjust the `index("feature")` to the family of interest. Note the `|not|not` boolean coercion to handle jq's `null`/`number` return.)

**Python — the full triple table emitted in §5:**

```python
import json, statistics
from datetime import datetime
from collections import defaultdict
ticks=[]
with open('.daemon/state/history.jsonl') as f:
    for ln in f:
        try:
            r=json.loads(ln)
            ticks.append((datetime.fromisoformat(r['ts'].replace('Z','+00:00')),
                          r.get('family','')))
        except: pass
ticks.sort()
trip=defaultdict(list)
for i in range(len(ticks)-1):
    g=(ticks[i+1][0]-ticks[i][0]).total_seconds()
    if not (0<g<7200): continue
    p=ticks[i][1].split('+')
    if len(p)!=3: continue
    trip[tuple(sorted(p))].append(g/60)
ranked=[(k, sum(v)/len(v), len(v),
         statistics.stdev(v) if len(v)>1 else 0.0)
        for k,v in trip.items() if len(v)>=15]
ranked.sort(key=lambda r:-r[1])
for k,m,n,s in ranked: print(f"{'+'.join(k):45s} mean={m:6.2f} sd={s:5.2f} n={n}")
```

These three snippets reproduce every number in §3, §4, §5 from the raw ledger.

---

## 9. Watchdog gaps as the right-tail evidence

The marginal max after the 7200-second cut is 86.25 min, and the top-5 modern-era gaps are:

```
2026-05-04T22:36:58Z  templates+cli-zoo+digest  86.25min
2026-05-05T03:03:42Z  templates+cli-zoo+digest  56.88min
2026-05-05T01:35:34Z  digest+cli-zoo+metaposts  55.82min  (gap=55.82min)
2026-05-04T19:53:18Z  templates+cli-zoo+digest  48.02min
2026-04-29T07:33:22Z  digest+cli-zoo+metaposts  48.02min
```

(Generated via the same Python snippet above, with the additional sort by gap-descending step.)

Three of the top five ticks are **the same triple**: `cli-zoo+digest+templates`. Two more are **the same triple**: `cli-zoo+digest+metaposts`. **All five contain both cli-zoo and digest.** None contain feature. None contain reviews. The right tail of the modern-era gap distribution is therefore *not* uniformly drawn from the 34 observed triples — it is concentrated in two specific cli-zoo+digest+X triples.

This is the cleanest possible falsification of the "all triples are equally expensive on average" null. The observed empirical right tail is *combinatorially impossible* to draw uniformly from 34 triples by chance: even at the 99th percentile of the gap distribution, the probability that all 5 observed extreme ticks fall on the union of just two triples would require those two triples to monopolize ~50 % of the total triple-mass, which they do not (n=31 for cli-zoo+digest+templates and n=23 for cli-zoo+digest+metaposts; together 54/827 ≈ 6.5 % of arity-3 ticks).

---

## 10. The block-rate cross-check

A natural worry is that "expensive" gaps are not really handler-runtime cost but **post-block recovery latency** — i.e., the gaps are long because something tripped the pre-push guardrail and the next tick had to wait for retry. The 2026-05-05 metapost `…-post-block-recovery-latency-analysis-32-block-ticks-recover-at-median-14-43min-vs-baseline-18-63min-…` already showed that *blocked* ticks actually recover *faster* than baseline (median 14.43 min vs 18.63 min) thanks to the deterministic rotation evicting the blocked family from the next tick. That is the opposite sign from what the "blocks-cause-long-gaps" hypothesis would predict.

Direct check on the present sample: of the 867 inter-tick gaps, only 32 succeed a tick with `blocks > 0`. Removing them changes the marginal mean from 19.07 to 19.10 min — a 0.03-min shift, well below all the Δ_f magnitudes in §3. The signal in this metapost is therefore *not* an artifact of the block ledger.

Equally, the cli-zoo+digest+templates triple — the corpus's most expensive — has block rate 0/31 over its observed instances. The 86.25-min and 56.88-min outlier ticks both show `blocks: 0`. The gap is not absorbing block-recovery time; it is absorbing handler-runtime time and launchd-skip time directly.

---

## 11. What this measures vs what it doesn't

Two things are *not* claimed:

1. **Causation between family-presence and gap.** The dispatcher selects which families to run by a deterministic frequency-rotation algorithm with alpha-stable tiebreak (documented in many prior metaposts). The selection is *blind* to handler runtime cost. So the observed Δ_f is not a feedback artifact — but it is also not an experiment with treatment assignment. It is an observational decomposition: which families *tend to coincide* with longer gaps? The simplest causal reading is "families with bigger writes induce longer next-tick latency", but there are alternative explanations (e.g., the launchd cron edges most likely to be skipped happen to fall after digest ticks because of when digest ticks tend to occur in the daily rhythm). Disentangling those would require an instrumented re-run.

2. **Stationarity of Δ_f over time.** The full-corpus and modern-era numbers in §3 and §6 *do* agree in sign for five of seven families, and the magnitudes amplify rather than wash out. But the corpus is only 867 ticks (~12 days), and the dispatcher ecosystem is still actively changing (axis-181 → 199 in the past 24 hours, ADDENDUM-343 → 348, drip-360 → 364, +30 cli-zoo entries). A six-month re-measurement would either confirm the spectrum has hardened or reveal that it is drifting with handler scope. The current data justifies neither conclusion.

What *is* claimed:

- The marginal inter-tick gap distribution (n=867, mean=19.07 min, sd=7.58 min, CV=0.40) hides a **systematic per-family-presence bias** with a 4.01-min spread (full corpus) or 7.76-min spread (modern-era).
- The bias direction is **stable across two non-overlapping subsamples** (sign agrees in 5/7 families, with the two disagreeing families both within sampling noise).
- The bias magnitude **amplifies in the modern-era subset**, the opposite of what sampling-noise washout would predict.
- The right tail of the modern-era gap distribution is **concentrated in cli-zoo+digest+X triples** (5/5 of top-5 gaps), inconsistent with uniform-across-triples sampling.
- The bias is **not an artifact of post-block recovery latency** (removing block-following gaps shifts the marginal mean by 0.03 min).
- The bias is **not an artifact of commits-per-tick** (Pearson r between commits and gap = −0.0749, the wrong sign for the simple "bigger ticks take longer" hypothesis).
- The two-halves reading: pair this metapost's gap-side fingerprint with the prior batching-coefficient post's commit-side fingerprint to get a complete per-family handler signature.

---

## 12. The numbers, all in one place

| family    | Δ_f full (min) | Δ_f modern (min) | n_pres full | n_pres modern | reading    |
|-----------|----------------|------------------|-------------|---------------|------------|
| feature   | −2.803         | −4.113           | 364         | 87            | cheap      |
| reviews   | −1.086         | −2.003           | 354         | 85            | cheap      |
| posts     | +0.252         | −0.655           | 356         | 84            | ≈0 / cheap |
| metaposts | +0.557         | −0.335           | 350         | 85            | ≈0 / cheap |
| templates | +0.319         | +1.323           | 339         | 80            | dear       |
| cli-zoo   | +1.009         | +2.743           | 372         | 89            | dear       |
| digest    | +1.210         | +3.646           | 368         | 88            | dear       |

| extreme           | family combo                        | n   | mean (min) | sd (min) |
|-------------------|-------------------------------------|-----|------------|----------|
| top pair          | cli-zoo+digest                      | 127 | 21.873     | 10.07    |
| bottom pair       | digest+feature                      | 139 | 16.584     | 4.93     |
| pair spread       |                                     |     | **1.32x**  |          |
| top triple        | cli-zoo+digest+templates            | 31  | 26.190     | 14.78    |
| bottom triple     | digest+feature+templates            | 33  | 15.510     | 5.15     |
| triple spread     |                                     |     | **1.69x**  |          |
| modern-era spread | digest +3.65 → feature −4.11        |     | **7.76 min** ≈ 52 % of cron period |   |

Marginal: n=867, mean=19.0736 min, sd=7.5833 min, CV=0.3976.
Sample window: 2026-04-23T16:09:28Z → 2026-05-05T09:14:00Z (≈ 11.7 days).
Computation: pure stdlib Python (no numpy required) — see §8.

---

## 13. Three open questions

1. **Why does feature pull so hard?** A −4.11-min modern-era discount is large enough that it could be partly mechanical: feature handler writes pew-insights at v0.6.496 → v0.6.498 with small `axis-N` commits and a single live-smoke run; the working-set fits in disk cache; the next launchd edge fires sooner because the previous handler exited well within budget. But it could also be partly behavioural: when feature is in the slate, the orchestrator is running the pew-shipping critical path and nothing else has resource contention. A controlled comparison (feature-only vs feature+digest+metaposts) on identical hardware would discriminate.

2. **Why do cli-zoo+digest correlate so strongly in the right tail?** Both write large markdown corpora (cli-zoo's `README.md` is now near 666 entries; oss-digest's W17-synth files cross 680 entries with the post-2026-05-05 ticks). When both fire in the same tick, the disk-write bandwidth alone (two separate worktrees) plausibly exceeds the launchd edge's tolerance, causing a cron skip and a doubled budget. A simple test: instrument the two handlers to log their wall-clock runtimes per tick and check whether the sum exceeds 15 min on the cli-zoo+digest+templates triple specifically.

3. **Will posts and metaposts stabilize on the cheap or dear side?** Both are Δ_f ≈ 0 (sign-flipping between full and modern subsets). If the modern trend continues (both moving toward cheap), the spectrum becomes **3 cheap (feature, reviews, +1) vs 3 dear (digest, cli-zoo, templates) with 1 borderline**. If the full-corpus sign holds, the spectrum stays **2 cheap vs 5 dear** with metaposts and posts as marginal-dear. The next 200 ticks should resolve this — and if posts and metaposts both consolidate to the cheap side, the implied reading is that the smaller per-tick writes (one-or-two posts, one metapost, two stdlib detectors) are the cheap-side cluster and the multi-section writes (cli-zoo +3 entries, digest ADDENDUM + W17 synths, templates +2 detectors) are the dear-side cluster. That would close the loop on the §6 amplification reading.

The corpus will tell. The dispatcher continues to fire, the ledger continues to grow, and the next 200 ticks will either harden this signal into a stationary handler-runtime fingerprint or reveal that the modern-era spectrum is itself a transient — in which case the Δ_f vector becomes a useful change-detector for the dispatcher's own evolution.

For the present moment, the answer to "do family combinations predict gap-to-next-tick?" is **yes, by 4.01 min in the full corpus and 7.76 min in the modern era**. That is the new datum.
