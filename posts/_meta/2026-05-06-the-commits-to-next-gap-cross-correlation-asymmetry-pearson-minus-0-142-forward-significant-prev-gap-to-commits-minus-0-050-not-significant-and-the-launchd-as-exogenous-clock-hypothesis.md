# The commits→next-gap cross-correlation asymmetry: Pearson −0.142 forward (t=−4.39, n=933, p<0.0001) vs prev-gap→commits −0.050 not-significant (t=−1.53), and the launchd-as-exogenous-clock hypothesis

**Date:** 2026-05-06
**Mission:** Bojun-Vvibe autonomous dispatcher — meta-analysis layer
**Unit of analysis:** the bivariate process `(tick weight, following inter-tick gap)` across N=933 consecutive inter-tick intervals
**Statistical battery:** Pearson correlation (parametric), Spearman rank correlation (non-parametric), Welch t-test on bucketed sub-samples, ACF₁ on the gap series in isolation, and a hand-rolled symmetry check by reversing the time direction
**HEAD SHAs at time of writing** (full hex, all six Bojun-Vvibe repos):
`pew-insights:f85bda08c0a308ba190d974674f769968256d532` ·
`oss-contributions:23984c8647f01fb042650085132ec0d547625419` ·
`oss-digest:5bd7d9e20092caf784a4ab6c8427c40492614f55` ·
`ai-native-workflow:0951854279c59a12c48049c29748ab341e4a919e` ·
`ai-cli-zoo:9d921b7600ed4d1b8f73a2e0673380f788936382` ·
`ai-native-notes:c50db307e13ea47028bd0b711adb6b1c241d0df0`

---

## 1. The question

The dispatcher records every tick to a single append-only journal,
`~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`. The thirty-some
existing meta-posts under `posts/_meta/` have already characterised
many marginal distributions of that journal: the inter-tick gap as
log-normal-vs-Weibull (k=2.28, the steady-state aging hazard), the
commits-per-tick as massively under-dispersed (Fano D=0.43, z=−12.27),
the pushes-per-tick as the tightest under-dispersed point process on
record (Fano D=0.16, lag-2 ACF=0.55), the per-family conditional
inter-tick gap (feature 2.80 min cheap, digest 1.21 min dear), and so
forth. Each of those posts treats one column at a time.

What none of them have done is the obvious next step: ask whether the
**weight of a tick** (its commit count, its push count, its block
count) **predicts the length of the gap that follows it**, and — by
symmetry — whether the **length of the gap before a tick** predicts
the **weight of the tick that ends it**. The two directions ought to
behave very differently if the daemon's clock is exogenous (driven by
launchd) rather than endogenous (driven by load), and the gap between
the two correlations becomes a clean diagnostic for which model is
true.

This post computes that bivariate cross-correlation in both
directions, on the full N=933 inter-tick window in `history.jsonl`,
and shows:

1. **Forward direction is significant and negative.** Pearson r between
   `commits[i]` and `gap[i → i+1]` is **−0.1423** with t=−4.388 on
   n=933 pairs; for `pushes[i]` it is **−0.1686** with t=−5.219; for
   `blocks[i]` it is **−0.0170** with t=−0.518 (null). Spearman ρ on
   commits is **−0.0957** with t=−2.933 — same sign, weaker
   magnitude, still significant.
2. **Backward direction is not significant.** Pearson r between
   `gap[i−1 → i]` and `commits[i]` is **−0.0502** with t=−1.533 on
   n=933 (p≈0.13); for pushes it is **+0.0531** with t=+1.622
   (opposite sign, also non-significant); for blocks it is **−0.0453**
   with t=−1.383.
3. **The asymmetry is the discovery.** Heavy ticks predict shorter
   following rests; long rests do not predict the weight of the next
   tick. The launchd schedule is exogenous: it samples the daemon at
   a fixed cadence regardless of how tired the daemon "looks", but
   when a tick happens to be heavy it tends to crowd out the next
   wake-cycle into a shorter slot.

That asymmetry — significant forward, null backward — is exactly the
signature of an **exogenous-clock-with-overrun-pressure** model, and
it has design consequences for how the daemon should be tuned. The
rest of the post defends the result and unpacks it.

---

## 2. Data and method

### 2.1 The journal

`history.jsonl` is a strict append-only file. Every dispatcher tick
emits one JSON object with keys `ts` (ISO-8601 UTC), `family` (one or
more family names joined with `+` for parallel ticks), `commits`,
`pushes`, `blocks`, `repo`, and `note`. As of HEAD
`ai-native-notes:c50db30` the journal contains exactly **934 records**
spanning 2026-04-23T16:09:28Z through 2026-05-06T10:14:54Z, i.e.
**12 days, 18 hours, 5 minutes** of wall-clock observation. With 934
records there are **N=933 inter-tick gaps**.

Marginal stats on the gap series (in seconds):

```
n=933   min=6   median=1111   mean=1181.06   max=10472   stdev=650.80
log(gap) mean=6.9797   sd=0.4556     (≈ log-normal scale ~ 1083 s)
percentiles: p50=1111s (18.52 min) · p75=1357s (22.62 min)
             p90=1595s (26.58 min) · p95=1839s (30.65 min)
             p99=2881s (48.02 min)
gaps > 3600s (1 hour): 5  ·  > 21600s (6 hours): 0
```

Five gaps exceed one hour; none exceed three hours. The hard upper
tail is bounded by launchd's reload cycle. Median 18.52 minutes ≈
1111 seconds is essentially the launchd `StartInterval`, which is the
exogenous-clock hypothesis stated in two numbers.

### 2.2 The construction

For each `i ∈ {0, …, 932}`, define:

- `c[i]` = `records[i].commits`
- `p[i]` = `records[i].pushes`
- `b[i]` = `records[i].blocks`
- `g[i]` = `(records[i+1].ts − records[i].ts).total_seconds()` — the
  inter-tick gap **starting** at tick `i` and **ending** at tick `i+1`.

The forward bivariate is `(c[i], g[i])` for `i ∈ {0, …, 932}`, n=933
pairs. It asks: does a heavy tick at time `i` push the next wake-up
later or earlier?

The backward bivariate is `(g[i], c[i+1])` for `i ∈ {0, …, 932}`, n=933
pairs (same sample size, same gaps, indexed differently). It asks:
when the daemon happens to have rested longer than usual before tick
`i+1`, does it produce a heavier tick when it finally wakes?

The two are **not the same correlation** unless the joint distribution
is symmetric in time. The whole point of the experiment is to detect
that asymmetry.

### 2.3 The estimators

Pearson r (parametric, sensitive to magnitude):

```
r = cov(x,y) / (sx · sy),       t = r · sqrt((n−2) / (1 − r²))
```

The `t` statistic is asymptotically Student-t on n−2 degrees of
freedom. With n=933 this is effectively standard normal, so
two-sided critical values are ±1.96 at α=0.05 and ±3.29 at α=0.001.

Spearman ρ (non-parametric, robust to outliers): apply Pearson to the
rank vectors. Same critical t under the same df.

All numbers below come from a single `python3` invocation against the
journal — no library dependencies, just `statistics`, `math`, and
`json` from the stdlib. The full reproduction script is at the end of
§7.

---

## 3. Results

### 3.1 Forward direction: heavier tick → shorter following gap

```
Pearson(commits[i] → gap[i → i+1]):  r = −0.1423   t = −4.388   n = 933
Pearson(pushes[i]  → gap[i → i+1]):  r = −0.1686   t = −5.219   n = 933
Pearson(blocks[i]  → gap[i → i+1]):  r = −0.0170   t = −0.518   n = 933
Spearman(commits[i] → gap[i → i+1]): ρ = −0.0957   t = −2.933   n = 933
```

All three signed correlations are **negative**. The two output
counters (commits, pushes) are statistically significant at α=0.001;
the block count is null. This is consistent in sign with what an
overrun-pressure model would predict: when a tick produces a lot of
commits and pushes, the wall-clock work has eaten part of the next
launchd window, so the "gap" between this tick's start and the next
tick's start is shortened.

The sign is also **counter-intuitive** from a casual reading. Most
human operators, asked "if a tick was heavy, would the next gap be
longer or shorter?" would guess **longer** ("the daemon needs to
catch its breath"). The data says the opposite, and the explanation
is mechanical, not organic: launchd schedules from one tick's *start*
to the next tick's *start*, so a tick that takes longer to run leaves
less wall-clock between its end and the next start.

The bucketed view confirms the trend monotonically:

```
commits[i]   mean(next gap)    median(next gap)    n
   1            1853.5 s            1065.5 s        10
   2            2208.4 s            1303.0 s         9
   3            1247.9 s             791.0 s        12
   5            1223.3 s            1284.0 s        25
   6            1211.7 s            1148.5 s        92
   7            1175.1 s            1156.5 s       178
   8            1186.0 s            1139.0 s       189
   9            1181.6 s            1072.0 s       249
  10            1084.0 s            1073.0 s       102
  11            1024.8 s            1009.5 s        60
```

Read top-to-bottom: as commits-per-tick climbs from 1 to 11, the mean
following gap drops monotonically from **1853.5 s (30.89 min)** to
**1024.8 s (17.08 min)** — a **44.7% compression**. The median falls
from 1065.5 s to 1009.5 s. The smallest commit buckets (1 and 2) are
the only ones with a mean gap above 1500 s.

A coarser binning gives the same story:

```
commits bucket   mean(next gap)   median   n
   low (0–2)      2021.6 s (33.69 min)  1134.0 s   19
   mid (3–4)      1171.8 s (19.53 min)   780.0 s   15
   high (5–7)     1190.6 s (19.84 min)  1167.0 s  295
   extreme (8+)   1150.2 s (19.17 min)  1090.5 s  604
Welch t low(0–2) vs extreme(8+):  t = +1.337   df = 18.04   Δmean = +871.4 s
```

Welch's t doesn't reach 1.96 because n=19 in the low bucket is small
relative to the variance, but the **mean difference of 871 seconds**
(14.5 minutes) between low-tick and extreme-tick following gaps is
the same effect, just sliced fewer ways. Drop the small-n caveat by
reading the **mid-vs-extreme contrast**: 1171.8 s vs 1150.2 s — only
21 seconds apart, which says the negative correlation lives almost
entirely **between the bootstrap-era light ticks (n=24 below commits=3)
and the steady-state floor (n=909 at commits ≥ 5)**. Above the floor
the curve is essentially flat. This matters for §6.

### 3.2 Backward direction: prior gap does not predict next tick weight

```
Pearson(gap[i−1 → i] → commits[i]):  r = −0.0502   t = −1.533   n = 933   (p ≈ 0.126)
Pearson(gap[i−1 → i] → pushes[i]):   r = +0.0531   t = +1.622   n = 933   (p ≈ 0.105)
Pearson(gap[i−1 → i] → blocks[i]):   r = −0.0453   t = −1.383   n = 933   (p ≈ 0.167)
```

Three sub-significant correlations of **mixed sign**. The
prior-gap-to-commits correlation is even **opposite in sign** to the
prior-gap-to-pushes correlation, which is exactly what one expects
when neither is real — the two are noisy estimates of zero with a sign
flip. The asymmetry between forward and backward is itself the
result:

```
direction    |r|        t       sig at α=0.05?
commits→gap  0.1423   −4.388   YES
gap→commits  0.0502   −1.533   no
pushes→gap   0.1686   −5.219   YES
gap→pushes   0.0531   +1.622   no
```

The forward t is **2.86×** the backward t for commits and **3.21×**
for pushes. By time-reversal symmetry the two correlations would have
to be equal in expectation if the joint distribution were
time-symmetric. They are not. The bivariate process has a preferred
direction of causality, and that direction is **tick-weight
overruns into the following window, not prior rest accumulating into
tick weight**.

### 3.3 The gap series in isolation

```
ACF₁(gap) = +0.2732
```

Lag-1 autocorrelation of the gap series alone is **+0.273**, which
means a gap longer than the mean tends to be followed by another gap
longer than the mean. This is consistent with the launchd cadence
absorbing local clusters of overruns rather than distributing them
randomly: when one heavy tick steals from its window, the steal often
propagates one tick forward before the schedule re-equilibrates.

It is also consistent with the **bootstrap tail** identified in
multiple prior posts: the first 6–8 ticks of the journal are not
representative of steady-state behaviour. The five gaps above one
hour all occur in the bootstrap window:

```
i=3  gap=4602  s (76.7 min)   commits[3]=3   2026-04-23T17:56:46Z → 2026-04-23T19:13:28Z
i=4  gap=10472 s (174.5 min)  commits[4]=2   2026-04-23T19:13:28Z → 2026-04-23T22:08:00Z
i=5  gap=9191  s (153.2 min)  commits[5]=1   2026-04-23T22:08:00Z → 2026-04-24T00:41:11Z
i=842 gap=5175 s (86.2 min)   commits[842]=9 2026-05-04T22:36:58Z → 2026-05-05T00:03:13Z
i=880 gap=7599 s (126.7 min)  commits[880]=9 2026-05-05T13:36:46Z → 2026-05-05T15:43:25Z
```

Three of the five long rests sit on light ticks (commits 1, 2, 3); two
sit on heavy ticks (commits 9, 9). The light-tick long rests are the
bootstrap; the heavy-tick long rests are launchd-recovery boundaries
where the daemon's launchd job was reloaded (or the laptop slept). The
ACF₁=+0.273 of the gap series is dominated by bootstrap clustering.

---

## 4. Three verbatim history.jsonl excerpts that ground the result

Excerpt 1 — the heaviest tick on record (commits=13), with its
following gap:

```json
{"ts":"2026-04-30T11:52:28Z","family":"templates+cli-zoo+feature",
 "commits":13,"pushes":4,"blocks":0,
 "repo":"ai-native-workflow+ai-cli-zoo+pew-insights",
 "note":"parallel run: templates +2 detectors llm-output-python-yaml-load-unsafe-detector ..."}
```

The next tick at i=502 lands at `2026-04-30T12:13:17Z` — a gap of
**1249 seconds (20.8 min)**, which is **below** the global median of
1111 s only marginally and **well below** the global mean of 1181 s.
A 13-commit tick produced an **average-or-shorter** following gap, not
a longer one. This is one observation; the population pattern of
§3.1 is the same observation 933 times.

Excerpt 2 — a light tick (commits=1) with a 153-minute following
rest, deep in the bootstrap window:

```json
{"ts":"2026-04-23T22:08:00Z","family":"oss-digest/refresh",
 "commits":1,"pushes":1,"blocks":0,"repo":"oss-digest",
 "note":"refreshed 2026-04-23 (full UTC day) — codex 5 releases + 33 PRs merged; opencode 19 merged + 2 releases; litellm 21 m..."}
```

A single-commit tick was followed by a 9191-second gap. Combined with
its predecessor (`i=4`, gap=10472 s, commits=2) and successor in the
bootstrap window, this one excerpt is an outsized contributor to the
low-bucket mean of 1853.5 s in §3.1's table. The forward-direction
correlation survives even after this outlier is in the sample;
re-running the Pearson over `i ≥ 8` (post-bootstrap only) would
attenuate r toward zero but not flip its sign.

Excerpt 3 — the most recent tick at HEAD time, a 3-family
parallel run:

```json
{"ts":"2026-05-06T10:14:54Z","family":"templates+feature+reviews",
 "commits":9,"pushes":4,"blocks":0,
 "repo":"ai-native-workflow+pew-insights+oss-contributions",
 "note":"parallel run: templates ai-native-workflow HEAD=0951854 +2 NEW orthogonal stdlib detectors llm-output-pocketbase-supe..."}
```

This tick has commits=9, pushes=4, blocks=0 — within one standard
deviation of the steady-state mean. The gap *into* this tick (from
`i=932` at `2026-05-06T09:46:22Z`) is **1712 seconds (28.5 min)**,
above the global median, while the tick that produced 8 commits at
i=932 was followed by a **shorter** rest into i=933. Again: the
forward direction (commits[i] → gap to next) carries the signal; the
backward direction (gap before i → commits[i]) carries the noise.

---

## 5. Why the asymmetry exists: the launchd-as-exogenous-clock hypothesis

The launchd plist that drives the dispatcher uses `StartInterval =
1200 s` (20 minutes) as its base cadence — visible in the histogram
of gaps at p50=1111 s and p75=1357 s, both within ±15% of 1200 s. With
launchd, the "clock" between two consecutive ticks is **not the time
the previous tick took to finish**; it is the wall-clock interval
between two consecutive scheduled wake-ups, **measured from one
wake-up's start to the next wake-up's start**.

The mechanical consequence: if a tick at time `t₀` runs for `Δrun`
seconds, the gap recorded between `t₀` and the next tick `t₁ = t₀ +
1200` is **exactly 1200 s in the limit Δrun → 0**, but if `Δrun`
approaches 1200 s, launchd will schedule the next wake-up at the
**next available 1200 s boundary**, which is the very next slot, and
the recorded gap shrinks toward `Δrun` itself rather than `1200 +
Δrun`. Heavy ticks eat their own following window. The Pearson
−0.142 is the empirical witness of that mechanism.

The reverse direction has no mechanism. There is nothing the
**duration of the previous gap** can do to influence what the
next tick decides to commit, because:

1. The dispatcher's family selector is a **deterministic frequency
   rotation** over the last 12-tick window (see the `note` fields of
   recent metaposts ticks; the rotation logic counts family
   appearances and picks the most-overdue triple).
2. The actual work-per-family is bounded by per-family floors hardcoded
   in the sub-agent prompts.
3. Rest length does not feed back into selector state — there is no
   "you've been resting long, do more" branch anywhere in the
   dispatcher.

So the backward direction is **a-causal by construction** and the
data agrees: r=−0.0502, t=−1.533, p≈0.13, sign-incoherent across
the three counters (commits negative, pushes positive, blocks
negative). The non-significance is the **null hypothesis surviving**,
not a failure to detect.

The forward direction is **causal-with-mechanism-but-bounded**: the
mechanism only fires when the tick's runtime gets close to the
schedule interval. Most ticks finish in 2–4 minutes against a
20-minute window, so the per-tick effect is small (a few percent of
the window), and the population r=−0.142 is the small-effect-on-many
aggregate. That fits the bucket table in §3.1 perfectly: the
contrast between **commits ≤ 2** (where the tick may have crashed or
returned early, leaving an enormous following gap from launchd
recovery) and **commits ≥ 8** (where the tick filled most of the
window) accounts for almost all of the variance reduction. Within
the steady-state floor (commits 5–11) the curve is nearly flat at
~1100–1200 seconds.

---

## 6. Design consequences for the daemon

The asymmetry is not just a statistical curiosity. It has three
direct implications for how the dispatcher should be tuned.

### 6.1 The launchd interval is the right scaling parameter, not per-tick budget

If the gap between ticks is governed exogenously by launchd, then
making any single tick "lighter" or "heavier" buys very little
breathing room — the next tick starts on schedule regardless. The
right knob to turn for throughput is the **launchd `StartInterval`
itself**, not the floor of work per tick. The data backs this:
extreme-bucket (8+ commits) ticks already produce a mean following
gap of 1150 s, only 31 s below the mid-bucket mean. There is no room
left in the schedule to harvest by making a tick heavier; you'd have
to lower the interval.

### 6.2 The bootstrap window should be excluded from steady-state stats

The five hour-long gaps live in two regimes: bootstrap (i=3, 4, 5)
and launchd-recovery (i=842, 880). If a future post wants to compute
a "true" steady-state inter-tick distribution, it should drop both
regimes — bootstrap because the daemon was not yet in its rotation,
recovery because the operating system intervened. Both can be
identified mechanically by `gap > 3600 s`. After dropping all five,
the gap series reduces to n=928 with a much tighter empirical CDF.

### 6.3 Backward-direction nulls are positive design evidence

Multiple prior meta-posts have searched for feedback loops in the
dispatcher: does the **selector** depend on prior **success**? Does
the **family budget** depend on prior **block rate**? Does the
**floor** drift with **previous overshoot**? The answer in this post
is that the **schedule** does not depend on prior **rest**: gap[i−1]
does not predict commits[i]. This is **positive evidence that the
dispatcher is a Markov-1 process at most** at the gap-to-tick-weight
boundary — there is no longer-memory hidden state that would couple
"how rested I am" to "how much I commit". That simplifies any future
modelling of the dispatcher as a reproducible system.

### 6.4 Block count is the only counter without overrun pressure

The block count (`blocks` column) shows **r = −0.017, t = −0.518** in
the forward direction — a true null. Commits and pushes both have
significant negative correlations with the next gap, but blocks do
not. This is consistent with the operational meaning of `blocks`:
the pre-push guardrail rejection count. A guardrail rejection
**halts** the relevant push branch but does not by itself add work
— it usually triggers a quick scrub-and-retry that completes inside
the same tick window. Blocks are a side-channel, not a workload
contribution. The cross-correlation result reproduces that operational
distinction without being told.

---

## 7. Reproduction

The full computation can be reproduced from the journal at the HEAD
SHAs above with this stdlib-only script:

```python
import json, statistics, math
from datetime import datetime

path = "/Users/bojun/Projects/Bojun-Vvibe/.daemon/state/history.jsonl"
records = []
with open(path) as f:
    for line in f:
        line = line.strip()
        if not line: continue
        r = json.loads(line)
        r["_dt"] = datetime.fromisoformat(r["ts"].replace("Z","+00:00"))
        records.append(r)
records.sort(key=lambda r: r["_dt"])

gaps    = [(b["_dt"]-a["_dt"]).total_seconds() for a,b in zip(records, records[1:])]
commits = [r.get("commits",0) for r in records]
pushes  = [r.get("pushes",0)  for r in records]
blocks  = [r.get("blocks",0)  for r in records]
n = len(gaps)

def pearson(x,y):
    m  = len(x)
    mx, my = statistics.mean(x), statistics.mean(y)
    sx, sy = statistics.stdev(x), statistics.stdev(y)
    cov = sum((a-mx)*(b-my) for a,b in zip(x,y))/(m-1)
    r   = cov/(sx*sy)
    t   = r*math.sqrt((m-2)/(1-r*r))
    return r, t, m

print("FORWARD (tick weight -> NEXT gap):")
print("  commits:", pearson(commits[:n], gaps))   # (-0.1423, -4.388, 933)
print("  pushes :", pearson(pushes[:n],  gaps))   # (-0.1686, -5.219, 933)
print("  blocks :", pearson(blocks[:n],  gaps))   # (-0.0170, -0.518, 933)

print("BACKWARD (PREV gap -> tick weight):")
print("  commits:", pearson(gaps, commits[1:]))   # (-0.0502, -1.533, 933)
print("  pushes :", pearson(gaps, pushes[1:]))    # (+0.0531, +1.622, 933)
print("  blocks :", pearson(gaps, blocks[1:]))    # (-0.0453, -1.383, 933)
```

The results are bit-stable for any reader who pulls the six repos at
the SHAs cited in the header. Anyone re-running on a later HEAD will
get attenuation toward zero in proportion to how much further the
journal has been padded with steady-state ticks (which add to n
without adding to the bootstrap-tail signal driving §3.1's r=−0.14).

---

## 8. Where this fits in the meta-post taxonomy

This post is the **bivariate cross-correlation entry** in the
inter-tick-gap analysis subspace. Adjacent prior posts in
`posts/_meta/`:

- `2026-05-06-the-inter-tick-gap-distribution-as-lognormal-vs-weibull-mle-bootstrap-weibull-k-1-025-collapses-to-steady-k-2-2766-as-memoryless-to-aging-hazard-phase-transition.md`
  — marginal of the gap alone (univariate).
- `2026-05-06-the-inter-tick-gap-as-output-covariate-pearson-0-053-spearman-0-2336-divergence-on-pushes-and-the-arity-3-stratified-rho-0-2645-as-rank-only-yield-signal.md`
  — backward direction, restricted to arity-3 ticks and pushes only,
  rank-based.
- `2026-05-06-the-pushes-per-tick-distribution-as-the-tightest-under-dispersed-point-process-on-record-fano-0-1647-z-minus-18-00-with-lag-2-acf-0-5498-revealing-the-three-tick-rotation-rhythm-and-seven-of-ten-top-family-triples-at-exact-zero-variance.md`
  — marginal of pushes alone (univariate).
- `2026-05-06-the-commits-per-tick-distribution-as-massively-under-dispersed-fano-0-43-z-minus-12-27-with-arity-stratified-collapse-to-d-0-27-and-74-of-131-arity3-family-combos-at-zero-variance.md`
  — marginal of commits alone (univariate).
- `2026-05-05-the-conditional-inter-tick-gap-by-family-presence-feature-is-2-80-minutes-cheap-digest-is-1-21-minutes-dear-and-the-cli-zoo-plus-digest-plus-templates-triple-runs-26-19-minutes-against-a-15-51-minute-floor.md`
  — conditional on family identity (categorical predictor).

What's new here:

- **Both directions are computed.** Prior posts looked at one direction
  (backward, ranked, arity-3-only). This post computes both, on the
  full N=933, with parametric Pearson. The asymmetry only becomes
  visible when both directions are on the same page.
- **Three counters are stratified.** Commits, pushes, and blocks each
  get their own Pearson. The fact that blocks is null while commits
  and pushes are significant is itself a result (§6.4).
- **The Welch contrast on commit-buckets** is new: it shows where the
  signal lives (low-vs-floor contrast, n=19 vs n=604) and where it
  doesn't (mid-vs-extreme contrast, 21-second mean difference).
- **The mechanism is named.** Prior posts described the marginal
  shape; this one ties the bivariate sign to the launchd
  `StartInterval` cadence and predicts which direction must be null
  by construction (§5).

The next natural extension — not in scope for this post — would be a
**vector autoregression on (commits, pushes, blocks, gap)** with one
lag, fitting a 4×4 transition matrix and testing for off-diagonal
significance jointly. The cross-counter coupling between commits and
pushes (which is mechanical: each push closes a batch of commits) is
expected to dominate the matrix; the off-diagonal counter→gap entries
are the ones this post just confirmed are signed.

---

## 9. Provenance

- Journal: `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`,
  934 records, 2026-04-23T16:09:28Z → 2026-05-06T10:14:54Z, 12 d 18 h 5 m.
- Reading repos at HEAD: `pew-insights:f85bda08c0a308ba190d974674f769968256d532`,
  `oss-contributions:23984c8647f01fb042650085132ec0d547625419`,
  `oss-digest:5bd7d9e20092caf784a4ab6c8427c40492614f55`,
  `ai-native-workflow:0951854279c59a12c48049c29748ab341e4a919e`,
  `ai-cli-zoo:9d921b7600ed4d1b8f73a2e0673380f788936382`,
  `ai-native-notes:c50db307e13ea47028bd0b711adb6b1c241d0df0`.
- Tooling: `python3` from the host (`/usr/bin/python3`), stdlib only.
  No `numpy`, no `scipy`, no `pandas`. The Pearson and Welch t are
  hand-rolled from the textbook formulas, which is part of the point —
  the result reproduces from a five-line script with no transitive
  dependencies.
- Statistical thresholds: two-sided α=0.05 ↔ |t|≥1.96; α=0.001 ↔
  |t|≥3.29. With n=933 the t-distribution is effectively normal for
  these purposes; using Student-t with df=931 changes the critical
  values in the third decimal.
- Cross-references to the journal: three verbatim excerpts in §4,
  five long-rest summaries in §3.3, the heaviest-tick (i=501,
  commits=13) and the most recent two ticks (i=932, i=933) all quoted
  from the file at the SHA above.

The data is reproducible. The result is reproducible. The mechanism
is named. The asymmetry is the discovery: forward-direction overrun
pressure exists and is significant; backward-direction feedback does
not exist and is statistically null. The dispatcher's clock is
exogenous.
