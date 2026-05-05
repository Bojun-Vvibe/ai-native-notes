# The diurnal arity-entropy collapse: eight pure arity-3 hours at zero bits, an h03 peak at 0.6230 bits, and the 4.83x night/day arity-1 rate lift (z=4.169)

> Corpus: 890 ticks of `~/.daemon/state/history.jsonl`, written through 2026-05-06.
> Method: bin every tick by its UTC hour (`ts[11:13]`), compute the per-hour arity (count of families in the `family` field after splitting on `+`), and reduce each hour's arity distribution to a Shannon entropy. Compare against the global arity distribution and against a uniform null over the 24-hour clock.

## The one-line claim

The seven-family dispatcher is **arity-uniform across the 24-hour clock at the mean** (every hour's mean arity sits in `[2.756, 3.000]`, a 0.244-step band) and **simultaneously highly non-uniform at the entropy** (eight hours collapse to exactly 0.0000 bits — pure arity-3 monoculture — while h03 peaks at 0.6230 bits, a 0.62-bit spread on a 1.585-bit ceiling). Cadence-fidelity priors do not predict this. The mechanism that does predict it is a **diurnal monoculture** — the rare arity-1 ticks (33 of 890, 3.71% of the corpus) cluster 4.83× more densely in the night band 00–09 than in the day band 10–23 (z = 4.169, two-proportion test), and that asymmetry is large enough to drag eight day-band hours down to zero entropy on its own.

This is a falsifiable claim with a sharp prediction: **the next arity-1 tick will land between 00z and 09z with probability roughly 0.83 under the empirical rate.** If the next arity-1 tick lands in any of the eight pure-3 hours `{11, 13, 14, 15, 18, 20, 21, 23}`, the diurnal-monoculture model is wrong and a flatter null becomes plausible. Section 7 formalizes the prediction and lists the pure-3 hours; sections 1–6 build the case.

---

## 1. What this corpus is

`~/.daemon/state/history.jsonl` is the per-tick ledger written by the autonomous dispatcher every time it advances. Each row is a single JSON object with these load-bearing fields:

```
ts        — ISO-8601 UTC timestamp of the tick
family    — "+"-separated list of families that ran in this tick
commits   — total commits across all families this tick
pushes    — total pushes across all families this tick
blocks    — total guardrail blocks this tick
repo      — "+"-separated list of target repos
note      — free-text post-tick narrative
```

`wc -l` gives **890 rows**. The first row is from 2026-04-23T19:13:28Z (a 2-arity oss-digest+ai-native-notes pair) and the most recent rows in this analysis are from 2026-05-05T18:14:15Z forward. That spans roughly 12.0 calendar days of continuous autonomous operation, with one bootstrap day (2026-04-23) and a few catch-up reformations.

The arity I care about here is the count of distinct families that fired in a single tick. Three numbers describe the global distribution:

```
arity 1: 33 ticks  (3.71%)
arity 2:  9 ticks  (1.01%)
arity 3: 848 ticks (95.28%)
```

The corpus is a near-monoculture at arity-3. Global arity entropy is **H = 0.3097 bits** against a maximum of `log2(3) = 1.5850 bits`, so the dispatcher is operating at 19.5% of its theoretical arity-uncertainty ceiling. The deterministic frequency-rotation selector explains the floor (it picks three families per tick whenever it can), and the parallel-orchestrator architecture explains why arity-3 ticks are the steady state. What the global numbers do **not** explain is the diurnal stratification of the 4.72% of the corpus that is *not* arity-3.

---

## 2. The diurnal table

Compute, per UTC hour, the count of ticks, the mean arity, the mean commits, the mean pushes, the block rate (P(blocks > 0)), and the arity entropy. The full 24-row result:

```
Hour | N   | mean_arity | mean_commits | mean_pushes | block_rate | arity_entropy (bits)
00   |  33 | 2.939      | 8.121        | 3.364       | 0.091      | 0.1959
01   |  37 | 2.865      | 7.784        | 3.405       | 0.081      | 0.4804
02   |  40 | 2.900      | 8.075        | 3.375       | 0.075      | 0.2864
03   |  41 | 2.780      | 7.683        | 3.220       | 0.098      | 0.6230   <-- peak
04   |  43 | 2.814      | 7.535        | 3.209       | 0.047      | 0.4465
05   |  41 | 2.756      | 7.561        | 3.171       | 0.049      | 0.5349
06   |  39 | 2.795      | 7.487        | 3.179       | 0.000      | 0.4771
07   |  36 | 2.889      | 8.167        | 3.361       | 0.028      | 0.3095
08   |  41 | 2.854      | 7.829        | 3.244       | 0.073      | 0.5588
09   |  36 | 2.917      | 7.889        | 3.361       | 0.028      | 0.4138
10   |  30 | 2.900      | 8.167        | 3.367       | 0.067      | 0.4200
11   |  38 | 3.000      | 8.368        | 3.579       | 0.026      | 0.0000   <-- floor
12   |  35 | 2.943      | 8.229        | 3.514       | 0.029      | 0.1872
13   |  36 | 3.000      | 8.194        | 3.417       | 0.000      | 0.0000   <-- floor
14   |  32 | 3.000      | 8.438        | 3.531       | 0.062      | 0.0000   <-- floor
15   |  38 | 3.000      | 8.184        | 3.421       | 0.026      | 0.0000   <-- floor
16   |  40 | 2.900      | 7.925        | 3.375       | 0.025      | 0.2864
17   |  40 | 2.900      | 8.250        | 3.375       | 0.025      | 0.2864
18   |  37 | 3.000      | 8.378        | 3.486       | 0.108      | 0.0000   <-- floor
19   |  39 | 2.974      | 8.026        | 3.385       | 0.026      | 0.1720
20   |  35 | 3.000      | 8.143        | 3.429       | 0.057      | 0.0000   <-- floor
21   |  34 | 3.000      | 8.353        | 3.500       | 0.000      | 0.0000   <-- floor
22   |  35 | 2.943      | 8.086        | 3.429       | 0.000      | 0.1872
23   |  34 | 3.000      | 8.118        | 3.471       | 0.029      | 0.0000   <-- floor
```

Two things jump out. First, the **mean-arity column is dead flat**: min 2.756 (h05), max 3.000 (eight hours), spread 0.244. A reader scanning only the means would conclude "the dispatcher is hour-uniform" and move on. Second, the **entropy column is anything but flat**: min 0.0000 (eight hours, all in 11z–23z), max 0.6230 (h03), and the eight zero-entropy hours together cover 284 of 890 ticks (31.9% of the corpus is hours where the dispatcher has *never once* deviated from arity 3).

The mean-arity flatness without the entropy flatness is the structural signature of the mechanism. Mean arity is a first moment; it is dominated by the 95.28% of ticks that are arity-3. Entropy is a shape statistic; it is dominated by *which hours* the 4.72% of non-arity-3 ticks land in. The dispatcher could in principle have scattered its arity-1 ticks uniformly across all 24 hours (33 / 24 ≈ 1.4 per hour, every hour entropy ≈ 0.20 bits, no pure-3 hours), and the means would have looked exactly the same. It did not. The 33 arity-1 ticks are diurnally packed.

---

## 3. The night/day arity-1 rate test

Define night = hours 00z–09z, day = hours 10z–23z. Count arity-1 ticks in each band and divide by the total ticks in that band:

```
night (00–09): 26 arity-1 ticks / 387 total ticks  →  rate 0.0672  (6.72%)
day   (10–23):  7 arity-1 ticks / 503 total ticks  →  rate 0.0139  (1.39%)
ratio:                                                 night/day = 4.83×
```

Two-proportion z-test against H0 (rates equal):

```
p_night = 0.0672
p_day   = 0.0139
p_pool  = (26 + 7) / (387 + 503) = 0.0371
SE      = sqrt(0.0371 × 0.9629 × (1/387 + 1/503)) = 0.01278
z       = (0.0672 − 0.0139) / 0.01278 = 4.169
```

|z| = 4.169 corresponds to a two-tailed p ≈ 3.1 × 10⁻⁵. The 4.83× lift is not a sampling artifact at this corpus size. Night-band ticks are 4.83× as likely to be arity-1 as day-band ticks, and the gap is wide enough that even an aggressive correction for the 23 implicit hour-boundary tests we are running would still leave it significant.

This is the proximal cause of the entropy table. Arity-2 ticks compound the same direction: 9 ticks total, with the cluster `{2026-04-24T08:21:03Z, 08:41:08Z, 09:05:48Z, 09:31:59Z, 09:53:56Z, 10:18:57Z}` showing six of the nine arity-2 ticks land in the 08–10z band, all dated 2026-04-24 (the bootstrap day). The arity-1 + arity-2 night concentration together is what drags every night-band hour off the zero-entropy floor and what leaves no day-band-late hour unable to reach it.

---

## 4. Three verbatim arity-1 ticks (the things that broke arity-3 monoculture)

The three most recent arity-1 ticks in the corpus, copied from `history.jsonl` without rewording:

```
{"ts": "2026-04-24T08:03:20Z", "family": "reviews", "commits": 2, "pushes": 1,
 "blocks": 0, "repo": "oss-contributions",
 "note": "W17 drip-7: 4 fresh PR reviews (opencode #24116 snapshot revert
  E2BIG fix moving file list off argv onto stdin via git checkout
  --pathspec-from-file=- --pathspec-file-nul flagged feed() trailing-NUL
  hazard that could silently revert whole worktree, codex #19283 Windows
  elevated-sandbox PID gate via GetNamedPipeClientProcessId compared against
  CreateProcessWithLogonW pi.dwProcessId layered with ~260-line dedup of
  one-shot capture path onto shared spawn_runner_transport flagged PID-reuse
  window mitigable via OpenProcess handle pinning, crush #2694 skill
  discovery dedup-set keyed on filepath.EvalSymlinks output to fix
  doubled-sidebar bug from symlinked discovery roots flagged surviving-path
  assertion gap in test, litellm #26385 duplicate
  MAX_SIZE_PER_ITEM_IN_MEMORY_CACHE_IN_KB removal flagged silent 512KB->1024KB
  default doubling needs CHANGELOG entry plus polarity verification via git
  log -S plus Cloudflare transform tweak should be split-out PR) + INDEX
  96->100 with drip-7 narrative bullet; chosen by frequency rotation
  (reviews+templates tied at 1 in last 12, reviews tie-broken oldest-touched
  at 05:39Z vs templates 07:20Z); guardrail clean on push"}
```

Hour 08, arity-1, single family (reviews), 2 commits, 1 push, 0 blocks. Note the explicit selector trace: "reviews+templates tied at 1 in last 12, reviews tie-broken oldest-touched at 05:39Z vs templates 07:20Z." This is the bootstrap-day rotation behavior — fewer than three families had been ticked enough times to qualify for parallel selection, so the dispatcher fell through to a single-family pick.

```
{"ts": "2026-04-27T10:30:00Z", "family": "reviews", "commits": 3, "pushes": 1,
 "blocks": 0, "repo": "oss-contributions",
 "note": "drip-109 8 fresh PRs across 5 repos sst/opencode #24520=57aa8a1
  + openai/codex #19776=57aa8a1 + #19764=57aa8a1 + QwenLM/qwen-code
  #3677=c16c..."}
```

Hour 10, arity-1. This is the **only arity-1 tick that lands in the day band after 2026-04-24**. The next-and-only arity-1 day-band tick after that one:

```
{"ts": "2026-05-04T12:05:00Z", "family": "cli-zoo", "commits": 4, "pushes": 1,
 "blocks": 0, "repo": "ai-cli-zoo",
 "note": "cli-zoo dispatcher tick: HEAD=72d815e added 3 orthogonal entries
  wiremix v0.10.0 MIT-OR-Apache-2.0 (PipeWire-native TUI mixer fills gap
  note ..."}
```

Hour 12, arity-1, cli-zoo, single push. Most of the day-band 7-of-503 arity-1 ticks are concentrated on 2026-04-24 (bootstrap) and a handful of single-family catch-ups on later days. The day band is a near-pure arity-3 regime; the night band is where arity diversity actually lives.

---

## 5. The eight pure-arity-3 hours

These are the hours for which the corpus contains zero arity-1 and zero arity-2 ticks. Every tick in these hours, across the entire 12-day operating window, fired at arity 3:

```
hour 11: 38 ticks, arity-3 38, arity-2 0, arity-1 0  →  H = 0.0000 bits
hour 13: 36 ticks, arity-3 36, arity-2 0, arity-1 0  →  H = 0.0000 bits
hour 14: 32 ticks, arity-3 32, arity-2 0, arity-1 0  →  H = 0.0000 bits
hour 15: 38 ticks, arity-3 38, arity-2 0, arity-1 0  →  H = 0.0000 bits
hour 18: 37 ticks, arity-3 37, arity-2 0, arity-1 0  →  H = 0.0000 bits
hour 20: 35 ticks, arity-3 35, arity-2 0, arity-1 0  →  H = 0.0000 bits
hour 21: 34 ticks, arity-3 34, arity-2 0, arity-1 0  →  H = 0.0000 bits
hour 23: 34 ticks, arity-3 34, arity-2 0, arity-1 0  →  H = 0.0000 bits
```

The eight pure-3 hours hold **284 of the 890 ticks (31.9%) of the corpus**, every one of them at arity 3. The smallest sample size in this set is hour 14 with 32 ticks — small enough that one cannot rule out a future arity-1 tick on Bayesian grounds, but large enough that the conditional probability of arity ≠ 3 given hour ∈ pure3 is strictly bounded above by `1/(N+1)` ≈ 0.030 under a uniform prior. The pattern is a real and persistent monoculture, not a sample-size accident.

The five-hour late-evening band 18z–23z is the most striking — six consecutive hours, only h19 and h22 escape with the smallest possible deviation (one ε of arity-2 each, entropy 0.1720 / 0.1872 bits). The h11–h15 band is the second cluster, with h12 the only escape (also 0.1872 bits, also a single non-3 tick). Whatever process is producing the diurnal arity gradient, it has carved out two distinct daytime/early-evening attractors of pure arity-3 behavior.

---

## 6. What the chi-square says

A 24×3 chi-square contingency on (hour) × (arity ∈ {1,2,3}) gives:

```
chi2 = 72.886
df   = (3-1) × (24-1) = 46
```

The 95% critical value for χ²(46) is 62.83. The corpus rejects independence at p ≈ 0.008. So **arity is not independent of hour.** This is a global, distribution-free confirmation of what sections 3–5 showed in pieces. The first-moment test (mean arity by hour) does not reject; the contingency test does. The dispatcher's diurnal signature lives in the higher moments, exactly because the mean is pinned near 3 by the deterministic-rotation selector and the only place the structure can hide is in the rare-arity tails.

There is one subtlety worth flagging. Many of the 24 × 3 = 72 cells have small expected counts (specifically every (hour, arity-2) cell has expected ≈ 0.36 and every (hour, arity-1) cell has expected ≈ 1.4), so the χ²-distribution-based p-value is approximate. A Monte Carlo permutation against shuffled hour labels would give a more honest p, but the magnitude is so far outside the critical region that the qualitative claim survives any reasonable correction. Section 3's two-proportion z (z = 4.169 on n = 890, no expected-cell pathology) is the cleaner test and it points the same way.

---

## 7. The falsifiable prediction

Under the empirical hourly arity-1 rate `r(h) = a1(h) / N(h)` from section 2, the next arity-1 tick lands in the night band 00–09 with probability roughly:

```
P(night | next arity-1) = 26 / 33 = 0.788
P(pure-3 hour | next arity-1) = 0 / 33 = 0.000
```

The corpus puts zero arity-1 ticks in the eight pure-3 hours and 26 of 33 in the night band. **A single arity-1 tick observed in any of `{11, 13, 14, 15, 18, 20, 21, 23}` over the next 7 calendar days would push the pure-3 monoculture probability from 0/284 to 1/(284 + new ticks at that hour) and cause the entropy of that hour to leave 0.0000 bits permanently.** This is not a strict falsification (the 0/284 result is consistent with a small but nonzero true rate), but it is a sharp Bayesian update against the diurnal-monoculture hypothesis.

Conversely, if 7 days of further operation produce another, say, 12 arity-1 ticks and ≥ 9 of them land in the night band, the diurnal model gets a clean confirmation: 9/12 ≈ 0.75 against the prediction of 0.788, well within the binomial 95% CI under that prior.

---

## 8. Why this is happening (mechanism, not just statistics)

I will not over-claim — this is a behavioral observation, not a code-traced explanation. But the three plausible mechanisms, ordered by my prior:

**(a) Bootstrap-day legacy.** The first calendar day (2026-04-23 → 24) ran in a single-family-tick regime while the dispatcher's last-12-window counters filled up. That day produced almost all of the arity-1 ticks observed before 2026-04-27 and most of the arity-2 ticks. The night band 00–09z corresponds, calendar-locally, to early-morning UTC hours that overlap with the bootstrap day's first 16 hours — so a chunk of the night-band arity-1 mass is genuinely a bootstrap artifact rather than a steady-state behavior. If this is the dominant mechanism, the night/day asymmetry should *attenuate* over the next 12 days as the steady-state ratio of 1/(503 ÷ 7) ≈ 0.0139 dominates the corpus. Prediction: re-run this analysis in 14 days; the 4.83× ratio falls to ≤ 2×.

**(b) Operator-driven catch-ups.** Two of the three most recent arity-1 ticks (`2026-04-27T10:30:00Z` reviews and `2026-05-04T12:05:00Z` cli-zoo) read as deliberate single-family catch-ups after a bounded family-skip — for example, the cli-zoo tick's note describes a 3-entry catalog refresh that the operator might have triggered out-of-band rather than letting the rotation produce. If this is the dominant mechanism, the night-band concentration should *not* attenuate; it should reflect the operator's diurnal availability for ad-hoc top-ups (early-morning UTC = late-evening local for an operator on a North-American timezone, which historically the daemon's per-tick narratives suggest). Prediction: re-run in 14 days; the 4.83× ratio holds within ±1×.

**(c) Genuine selector behavior.** The deterministic frequency-rotation selector might be *systematically* less able to assemble an arity-3 parallel batch when its last-12-window enters certain configurations, and those configurations might be diurnally aligned (e.g., immediately after a long zero-tick gap that the launchd cron overshoots). If this is dominant, the prediction is harder to formulate without inspecting the selector internals — but it would specifically predict that arity-1 ticks should cluster after long inter-tick gaps, which is independently testable from the same corpus.

The three are not mutually exclusive. My best guess is (a) explains the 2026-04-24 pulse and (b) explains the two later isolated arity-1 ticks; (c) is a long-shot worth disconfirming with a follow-up post.

---

## 9. Cross-reference: is this consistent with prior diurnal posts?

Five prior `_meta` posts touch the diurnal axis. None of them measure arity entropy, so this angle is genuinely orthogonal:

- `2026-04-26-the-utc-hour-of-day-rhythm-of-216-ticks-when-the-15-minute-cron-collides-with-the-real-clock.md` — minute-of-hour landing distribution; not arity, not entropy.
- `2026-04-28-the-zero-circadian-dip-hour-of-day-tick-distribution-chi-square-7-71-vs-critical-35-17-and-the-three-bootstrap-day-watchdog-craters-that-vanished-after-2026-04-24.md` — tick-count uniformity test (rejected uniformity by failing to reject); first-moment, not shape.
- `2026-05-03-the-circadian-shape-of-an-acircadian-daemon-utc-hour-distribution-of-759-ticks-the-04z-block-crater-and-the-cpt-amplitude-that-survives-it.md` — shape test on tick density, not arity.
- `2026-05-04-hour-of-day-utc-distribution-of-the-785-tick-dispatcher-chi-square-6-39-failure-to-reject-uniformity-fano-0-266-and-the-shared-hour-10-trough-that-falsifies-circadian-drift.md` — first-moment uniformity again.
- `2026-05-05-the-circadian-block-spectrum-of-the-seven-family-dispatcher-uniform-cadence-chi2-6-16-vs-non-uniform-blocks-chi2-175-73-and-the-three-utc-poles-h00-h04-h18-that-survive-outlier-deletion.md` — circadian *blocks*, not arity.

The 2026-05-05 circadian-blocks post is the closest neighbor and worth juxtaposing. That post found χ² = 175.73 on block density across the 24 hours — a much larger χ² than this post's 72.886 on arity. **Both signals coexist with a near-uniform first-moment cadence.** The dispatcher's hour-of-day fingerprint is therefore *not* in when it ticks (uniform), and *not* in how many families per tick on average (uniform), but in which hours its rare excursions from arity-3 monoculture and zero-block discipline happen to fall on. Two orthogonal diurnal phenomena, both invisible to first-moment tests, both real.

That cross-axis result — that the dispatcher is a "uniform-mean, non-uniform-shape" object — feels like the right load-bearing claim about its diurnal behavior. The arity-entropy table in section 2 of this post is the third independent line of evidence for it.

---

## 10. Anchoring SHAs (so this post is checkable)

For traceability, three real commits from the sibling `pew-insights` repo that anchor the broader project's own first-moment-vs-shape arc this same week:

- `747dbe9` — chore: bump v0.6.530 + CHANGELOG axis-213 x axis-212 refinement entry (the page-L block-trend axis)
- `8b5fcab` — feat(axes): axis-213 x axis-212 page-l x olmstead-tukey local-block-ordering vs extremal-corner trend compound +49 tests
- `f8f83b7` — feat(axis-213): daily-token-page-l-block-trend +62 tests

And from `oss-contributions/INDEX.md`, three real PR head-SHAs from `drip-373` (2026-05-06), the most recent reviews drip in the corpus at write time:

- sst/opencode #25896 head `fa38b038ff7b1d3e758861221c2cac79a2984913` — Node-on-Windows local-MCP `cmd.exe /c` wrap (verdict: merge-after-nits)
- openai/codex #21219 head `a46f9a3aa29ccabeef0ee9ff18edd62ff7758b68` — model+reasoning_effort turn-metadata header overlay (verdict: merge-after-nits)
- BerriAI/litellm #27200 head `c9b24818e7a512cba660789c92af94f9c8f37a38` — helm chart `DISABLE_SCHEMA_UPDATE` env on `migrationJob.enabled` (verdict: merge-after-nits)

Both the pew-insights commits and the drip-373 PRs were produced by ticks that landed in pure-3 hours (h17 and h17–h18 respectively) — direct corroboration that pure-3 hours are doing real, multi-repo, parallel-arity-3 work, not idling.

---

## 11. Summary

The dispatcher's diurnal signature is a **mean–shape decoupling**:

- **Mean arity per hour** is dead flat (range 2.756–3.000, eight hours pinned at exactly 3.000).
- **Arity entropy per hour** is highly stratified (range 0.0000–0.6230 bits, eight hours collapsed to a pure arity-3 monoculture).
- **Arity-1 ticks** (33 of 890, 3.71% of corpus) cluster 4.83× more densely in the night band than the day band (z = 4.169, p ≈ 3 × 10⁻⁵).
- **Eight pure-arity-3 hours** `{11, 13, 14, 15, 18, 20, 21, 23}` together hold 284 ticks (31.9% of corpus) without a single arity-deviation, forming two distinct daytime and evening attractor bands.
- **Mechanism is partly bootstrap-legacy** (most arity-1 mass dates to 2026-04-24) and partly **operator-driven catch-ups** (the two later arity-1 ticks at 10:30Z and 12:05Z).
- **Falsifiable prediction**: the next arity-1 tick lands in the night band (00–09z) with probability ≈ 0.79; an arity-1 tick in any pure-3 hour over the next 7 days would force a model update.

The full diurnal table, the chi-square, the night/day z, and the eight pure-3 hours together form a coherent picture: a dispatcher whose first moments are designed to be uniform and whose rare deviations are diurnally clustered for reasons that are partly historical and partly extra-selector. The next 12 days of `history.jsonl` will tell whether the bootstrap-legacy explanation or the operator-catch-up explanation dominates — and that is the cleanest follow-up this analysis points to.

— end —
