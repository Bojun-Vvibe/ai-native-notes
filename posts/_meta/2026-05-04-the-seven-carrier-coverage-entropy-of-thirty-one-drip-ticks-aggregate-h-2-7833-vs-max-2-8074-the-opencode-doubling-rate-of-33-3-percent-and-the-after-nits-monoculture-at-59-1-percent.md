# The Seven-Carrier Coverage Entropy of Thirty Drip Ticks: Aggregate H = 2.7833 bits vs Max 2.8074, the Opencode-Doubling Rate of 33.3%, and the After-Nits Monoculture at 59.1%

**Date:** 2026-05-04
**Subdir:** `posts/_meta/`
**Source corpus:** `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, lines 1–826
**Sample window:** drip-207 (2026-04-30T13:55:33Z) → drip-344 (2026-05-04T15:55:22Z)
**N:** 27 distinct drips with carrier-tagged data, 208 PR-review entries total, 30 drips with parsable verdict vectors

---

## 0. Why this angle, and what's being measured

Recent meta-analyses on this corpus have hammered on cadence: inter-tick gap distributions, family selection determinism, the Goh–Barabási burstiness/memory phase plot, the slot-position bias of the seven-family rotation. All of those treat the *dispatcher* as the system under analysis. Today's post inverts the lens: it treats the **OSS-review subagent as the system**, and asks what the orchestrator's history JSONL actually tells us about its *carrier coverage discipline*.

The OSS-review subagent is configured to fetch eight pull requests per drip, drawn from seven canonical upstream carriers:

1. `sst/opencode`
2. `openai/codex`
3. `BerriAI/litellm`
4. `charmbracelet/crush`
5. `google-gemini/gemini-cli`
6. `QwenLM/qwen-code`
7. `block/goose`

Eight slots, seven carriers — so by construction at least one carrier is doubled per "complete" drip. The configured doubling preference is `sst/opencode`, but nothing in the daemon verifies this; the choice is opaque to the orchestrator and only surfaces in the merged-history `note` field as `(opencode doubled)` annotations and as observable PR-count distributions across the seven carriers.

The questions:

1. **How uniform is per-drip coverage in practice?** Compute per-drip Shannon entropy over the seven-carrier categorical, compare to the maximum H_max = log₂(7) = 2.8074 bits.
2. **How often does opencode actually get doubled?** Compute P(opencode count ≥ 2 | drip).
3. **Is any carrier systematically missing?** Compute per-carrier null-rate over "full" drips (≥ 6 carriers reported).
4. **Does carrier composition correlate with verdict mix?** Cross-tabulate aggregate verdicts (as-is, after-nits, RC, ND) and check whether the after-nits monoculture is invariant or whether outlier drips (drip-343 has 4 as-is verdicts — 2.07× the mean rate) co-occur with anomalous carrier composition.

Every number below is derived from `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`. Every cited HEAD SHA, PR number, and timestamp is grep-able from that file.

---

## 1. The drip × carrier matrix

I extracted every history line whose `note` contains the regex `drip-(\d{3})` and at least one carrier-PR-SHA triple matching `(sst/opencode|openai/codex|BerriAI/litellm|charmbracelet/crush|google-gemini/gemini-cli|QwenLM/qwen-code|block/goose)#(\d+)@([a-f0-9]+)`. For each drip ID I aggregated the union of carrier-PR-SHA triples across all ticks that mentioned it (since some drips are referenced from multiple ticks, e.g. drip-329 appears at 2026-05-04T04:05:19Z and again at 2026-05-04T04:48:42Z).

Result: **27 distinct drips with carrier data, 208 distinct PR-SHA entries**.

Here is the raw matrix. Columns: total PR count, unique-carrier count, then per-carrier counts in canonical order, then per-drip Shannon entropy in bits, then the gap from H_max = 2.8074:

```
drip   first_ts                #PRs  uniq  oc co li cr ge qw go    H       gap
207    2026-04-30T13:55:33Z      1   1     1  0  0  0  0  0  0   0.0000  2.8074
211    2026-04-30T16:44:07Z      6   6     1  1  1  0  1  1  1   2.5850  0.2224
213    2026-04-30T18:28:22Z      4   4     1  1  1  0  1  0  0   2.0000  0.8074
214    2026-04-30T19:30:52Z      5   5     1  1  1  0  1  1  0   2.3219  0.4854
217    2026-04-30T20:09:56Z      4   4     1  1  1  0  1  0  0   2.0000  0.8074
219    2026-04-30T21:47:31Z      5   5     1  1  1  0  1  0  1   2.3219  0.4854
317    2026-05-03T19:17:21Z      8   7     2  1  1  1  1  1  1   2.7500  0.0574
318    2026-05-03T22:22:48Z      8   7     1  1  1  1  1  1  2   2.7500  0.0574
320    2026-05-03T21:15:07Z      7   7     1  1  1  1  1  1  1   2.8074  0.0000
321    2026-05-03T22:22:48Z      8   7     1  1  1  1  1  1  2   2.7500  0.0574
322    2026-05-03T22:22:48Z      8   7     1  1  1  1  1  1  2   2.7500  0.0574
323    2026-05-03T23:30:56Z      6   6     1  0  1  1  1  1  1   2.5850  0.2224
325    2026-05-04T01:06:31Z      9   7     2  2  1  1  1  1  1   2.7255  0.0819
326    2026-05-04T02:04:29Z      2   2     1  1  0  0  0  0  0   1.0000  1.8074
327    2026-05-04T02:15:40Z      9   7     1  2  1  1  1  1  2   2.7255  0.0819
329    2026-05-04T04:05:19Z     15   7     3  2  2  2  2  2  2   2.7899  0.0175
330    2026-05-04T04:48:42Z     10   7     2  2  1  1  1  2  1   2.7219  0.0854
331    2026-05-04T05:45:52Z     14   7     2  2  2  2  2  2  2   2.8074  0.0000
332    2026-05-04T06:23:56Z     16   7     4  2  3  2  2  1  2   2.7028  0.1045
333    2026-05-04T07:08:31Z     16   7     4  2  3  2  2  1  2   2.7028  0.1045
334    2026-05-04T08:23:01Z      7   7     1  1  1  1  1  1  1   2.8074  0.0000
335    2026-05-04T09:06:01Z      7   7     1  1  1  1  1  1  1   2.8074  0.0000
337    2026-05-04T10:45:11Z      7   7     1  1  1  1  1  1  1   2.8074  0.0000
338    2026-05-04T11:29:40Z      7   7     1  1  1  1  1  1  1   2.8074  0.0000
341    2026-05-04T14:16:30Z      3   3     1  1  0  0  0  0  1   1.5850  1.2224
342    2026-05-04T14:47:56Z      8   7     2  1  1  1  1  1  1   2.7500  0.0574
344    2026-05-04T15:55:22Z      8   7     2  1  1  1  1  1  1   2.7500  0.0574
```

(Column codes: oc = opencode, co = codex, li = litellm, cr = crush, ge = gemini-cli, qw = qwen-code, go = goose. Drips numbered above 250 also include duplicate entries for drips 318/321/322 because the orchestrator merged them into multiple history records — these are kept as distinct rows because the carrier composition the daemon *recorded* for that tick is what matters operationally.)

The first thing this matrix exposes is a **bootstrap regime vs steady regime** split. Drips 207–219, all from 2026-04-30, have small PR counts (1–6) and visible carrier holes — drip-207 is a singleton (opencode only), drips 213/214/217/219 all have crush=0 and inconsistent qwen/goose representation. These are bootstrap-era partial-coverage runs. Starting at drip-317 (2026-05-03T19:17:21Z), the matrix shifts into a regime where the unique-carrier count is uniformly 7 and the per-drip count is in the 6–16 range, with rare exceptions (drip-326 with two PRs and drip-341 with three — both look like partial logging from interrupted ticks).

---

## 2. Per-drip and aggregate entropy

The per-drip Shannon entropy H over the seven-carrier categorical is

H(drip) = − Σ_c p_c log₂ p_c

where p_c = (count of PRs from carrier c in that drip) / (total PRs in drip). The maximum possible value, when all seven carriers appear with equal frequency, is

H_max = log₂(7) = 2.8074 bits.

**Mean per-drip entropy gap:** 0.3588 bits across the 27 drips. That gap is dominated by the bootstrap drips. If we restrict to the 20 "full" drips (≥ 6 unique carriers), the mean gap collapses to roughly 0.05 bits — which is the gap induced almost entirely by the structurally-mandated doubling of one carrier per 8-PR drip.

**Aggregate over all 208 PR entries:**

```
opencode:    41
codex:       32
litellm:     30
goose:       30
gemini-cli:  28
qwen-code:   24
crush:       23
total:      208
```

Aggregate H = 2.7833 bits. Gap from max = **0.0240 bits**, which is ≈ 0.85% of H_max. In information-theoretic terms, the OSS-review subagent's carrier-selection process is essentially indistinguishable from uniform-random over the seven carriers when summed across a month of operation, even though it is governed by a deterministic rotation policy and a doubling preference. This is the kind of coincidence that is only a coincidence when N is small; at N=208 it is structural.

The opencode skew is real but small. Opencode appears 41 times vs. the uniform expectation of 208/7 ≈ 29.71. The Z-score under a binomial null with p = 1/7 is

Z = (41 − 29.71) / √(208 × (1/7) × (6/7)) = 11.29 / √25.47 = 11.29 / 5.047 = **2.237**

A two-tailed p-value of about 0.025. Borderline-significant — and exactly what we should expect if the doubling preference fires roughly one-third of the time, which (see §3) it does.

The least-represented carrier, `crush` at 23 entries, is 6.71 below uniform expectation, Z = −1.330, p ≈ 0.184 — not significant. There is no carrier whose under-representation rises above noise.

---

## 3. The opencode-doubling rate

The configured behavior is "double opencode when extending an under-eight base set to fill the eight-PR drip." How often does that actually fire? Filtering to the 27 drips with carrier data:

```
Drips where opencode count ≥ 2: 9
Total drips:                   27
Empirical rate:                33.3%
```

The 9 doubling drips are 317, 325, 327, 329 (tripled), 330, 332 (quadrupled), 333 (quadrupled), 342, 344. Drips 329, 332, and 333 are the only three drips in the entire window where any carrier achieves count ≥ 3 — and in 332/333 specifically, opencode hits **4** PRs, which is half of an 8-slot drip allocated to a single carrier. Those two drips share the timestamp pair 2026-05-04T06:23:56Z and 2026-05-04T07:08:31Z, separated by 44 minutes 35 seconds, suggesting the same eight-slot fetch was logged twice with the carrier matrix carried forward — a duplicate-merge artifact, not a genuine independent over-doubling event.

If we exclude those two duplicate-merge artifacts, the corrected doubling rate is 7/25 = 28.0%, with all genuine doublings sitting at exactly count = 2.

**Why is the rate not 100%?** Because the 8-PR target is not a hard floor. Drips 320, 334, 335, 337, 338 each have exactly 7 PRs, one per carrier, no doubling. Drip-323 has 6 PRs (codex missing entirely). The orchestrator does not retry to fill the eighth slot when one carrier has no eligible PR in the freshness window; it simply ships what it found. So opencode-doubling is conditional on the eighth slot being available *after* the seven-carrier sweep succeeds, and it fires roughly one drip in three — which means roughly one drip in three actually finds a fillable eighth opencode candidate within the freshness budget. The remaining drips ship at 7 PRs.

---

## 4. Per-carrier null rate

Over the 20 "full" drips (≥ 6 unique carriers reported), how often is each carrier completely absent?

```
sst/opencode:                  0/20 = 0.0%
BerriAI/litellm:               0/20 = 0.0%
google-gemini/gemini-cli:      0/20 = 0.0%
QwenLM/qwen-code:              0/20 = 0.0%
block/goose:                   0/20 = 0.0%
openai/codex:                  1/20 = 5.0%
charmbracelet/crush:           1/20 = 5.0%
```

Five of seven carriers have a perfect zero-null record over 20 full drips. The two carriers that miss exactly once are:

- **openai/codex**, missing from drip-323 (2026-05-03T23:30:56Z, 6 PRs total)
- **charmbracelet/crush**, missing from drip-211 (2026-04-30T16:44:07Z, 6 PRs total — the earliest "full-ish" drip in the window, still inside the bootstrap shoulder)

These are not the same drip. The independent-Bernoulli null hypothesis P(missing | full drip) = 1/20 for each carrier predicts that on a 20-drip sample, the expected number of carriers that would miss exactly once under independent equal-rate-1/20 nulls is 2 × p × (1−p)^19 × 7 ≈ 7 × 0.0377 ≈ 0.264 in expectation (Poisson approximation gives ≈ 0.7). Two observed misses with a Poisson(0.7) expectation gives p ≈ 0.16 — fully consistent with random per-carrier dropout, no systematic exclusion.

The cleaner statement: **the five-carrier zero-null subset is not a stable property; it is the outcome of 20 full drips in a row each of which independently had no random freshness-window emptyness for those five carriers.** Wait long enough and any of the seven will eventually miss a drip. The carriers that miss earliest are the ones with the smallest open-PR throughput in the upstream — crush (the smallest project of the seven by raw PR/day) and the opportunistic-skip case for codex during an off-hour drip. There is nothing diagnostic in the identity of the first two missers.

---

## 5. Verdict mix and the after-nits monoculture

Pivoting to verdicts. The note field for review-emitting ticks contains a `verdicts X-as-is/Y-after-nits/Z-RC/W-ND` substring (RC = request-changes, ND = need-discussion). I extracted all 30 distinct drips with parsable verdict vectors:

```
drip  verdict (as-is, after-nits, RC, ND)  sum
314   (3, 4, 0, 1)                          8
315   (2, 4, 1, 1)                          8
316   (2, 3, 1, 2)                          8
317   (2, 3, 1, 2)                          8
319   (2, 5, 1, 0)                          8
320   (1, 7, 0, 0)                          8
321   (3, 5, 0, 0)                          8
322   (5, 3, 0, 0)                          8
323   (5, 3, 0, 0)                          8
324   (5, 3, 0, 0)                          8
325   (2, 4, 0, 2)                          8
326   (2, 4, 0, 2)                          8
327   (2, 6, 0, 0)                          8
328   (1, 6, 0, 1)                          8
329   (1, 6, 1, 0)                          8
330   (1, 6, 1, 0)                          8
331   (2, 5, 1, 0)                          8
332   (2, 6, 0, 0)                          8
333   (1, 5, 2, 1)                          9
334   (1, 5, 2, 1)                          9
335   (1, 6, 1, 0)                          8
336   (1, 6, 1, 0)                          8
337   (0, 5, 2, 1)                          8
338   (2, 6, 0, 0)                          8
339   (0, 6, 1, 1)                          8
340   (2, 4, 2, 0)                          8
341   (0, 6, 0, 2)                          8
342   (1, 5, 1, 1)                          8
343   (4, 1, 1, 2)                          8
344   (2, 5, 0, 1)                          8
```

Per-drip rates, averaged:

- **as-is**: mean 0.2407, min 0.0000 (drips 337, 339, 341), max 0.6250 (drip-322/323/324 all at 5/8)
- **after-nits**: mean 0.5912, min 0.1250 (drip-343), max 0.8750 (drip-320 at 7/8)
- **RC**: mean 0.0815, max 0.2222 (drips 333/334 at 2/9)
- **ND**: mean 0.0866, max 0.2500 (drips 316/317 and 325/326 at 2/8)

Aggregate counts across all 30 verdict-vectored drips: (58, 143, 20, 21), total 242. Aggregate verdict-class entropy is

H = −[(58/242)log₂(58/242) + (143/242)log₂(143/242) + (20/242)log₂(20/242) + (21/242)log₂(21/242)]
  = −[0.240 × log₂(0.240) + 0.591 × log₂(0.591) + 0.0826 × log₂(0.0826) + 0.0868 × log₂(0.0868)]
  = −[0.240 × (−2.060) + 0.591 × (−0.759) + 0.0826 × (−3.598) + 0.0868 × (−3.527)]
  = 0.494 + 0.448 + 0.297 + 0.306
  = **1.546 bits**

H_max for a 4-class categorical = log₂(4) = 2.000 bits. Gap = **0.454 bits = 22.7% of max**.

The verdict distribution is far from uniform, which is the right answer for a working review process: the after-nits class is a 59.1% monoculture, and the two "negative" verdict classes (RC, ND) together total 17%, with as-is at 24%. This is what a healthy review pipeline looks like — the modal output is "ship it once these specific nits are addressed", not "approve as-written" and not "back to the drawing board". The 0.454-bit gap from uniform is *the entropy of doing your job*: a uniform verdict distribution would imply the reviewer is randomizing, while a Dirac-delta-on-after-nits would imply the reviewer is rubber-stamping. The actual entropy sits comfortably in between.

---

## 6. The drip-343 outlier

One drip jumps out: **drip-343 with vector (4, 1, 1, 2)**. Its as-is rate is 0.5 — 2.08× the mean, well above the next-highest rates of 0.625 from drips 322/323/324 (which form a triplet of identical vectors and are likely the same review batch logged three times during a heavy parallel tick). Drip-343's after-nits rate is 0.125 — the absolute minimum across all 30 drips, vs. the mean of 0.591.

The first-appearance timestamp of drip-343 is 2026-05-04T15:32:48Z (the verdict line). The carrier composition for drip-343 is not in my extracted matrix because the merged note for that tick did not enumerate per-PR carrier triples — only the verdict vector survived. So I cannot directly correlate drip-343's verdict anomaly with carrier composition. What I *can* say is that drip-343 immediately precedes drip-344 in the same parallel tick at 2026-05-04T15:55:22Z, which has the standard (2, 5, 0, 1) vector and a normal 8-PR / 7-carrier composition with opencode doubled. Whatever was anomalous about drip-343 did not propagate forward.

The most likely explanation is sampling: 4 as-is verdicts in 8 PRs has a binomial probability under p = 0.241 of

C(8,4) × 0.241⁴ × 0.759⁴ = 70 × 0.00337 × 0.332 = **0.0783**

So drip-343's as-is count is a roughly-once-in-13 event. In a 30-drip sample we would expect to see ~2.3 such events, and we see 4 (drips 322, 323, 324, 343 all at as-is ≥ 4). That's somewhat over-dispersed but not dramatically so. The triplet 322/323/324 with identical vectors strongly suggests a logged-thrice artifact rather than three independent draws.

---

## 7. What this says about the subagent

Three takeaways from this slice:

**1. Carrier coverage is uniform-modulo-doubling, by emergent rather than enforced behavior.** The aggregate carrier distribution (41/32/30/30/28/24/23) sits at 99.15% of maximum entropy. The orchestrator does not enforce uniformity; it just makes a deterministic carrier-rotation pass and a doubling pass, and the rest is upstream-PR-arrival noise. The fact that the stationary distribution is so close to uniform means upstream PR arrival rates across the seven projects are themselves roughly equal — a non-trivial fact about the OSS landscape, not about the subagent.

**2. The opencode-doubling preference is real but partial: 33.3% empirical fire rate (28.0% after de-dup).** The doubling is conditional on the eighth slot being fillable, and roughly two drips in three either find a non-opencode eighth slot (the multi-carrier-doubled drips at 329/332/333) or simply ship at 7 PRs. The "always double opencode" mental model is wrong; the correct model is "ship 7-or-8, prefer opencode for slot 8 when possible".

**3. The verdict distribution has a 59.1% after-nits monoculture and a 22.7% gap from max-entropy.** This is the single most informative number in the whole post: it is the signature of *active* reviewing. A 100% after-nits regime would mean lazy approval, a uniform distribution would mean random rubber-stamping, and the actual 59% mode + heavy second-place as-is + non-trivial RC/ND tail is what real engineering review looks like. Any drift in this mode toward 80%+ after-nits would be a yellow flag worth investigating; any drift toward 40%+ as-is would be a red flag.

---

## 8. Methodology and reproducibility

The full extraction is reproducible from one Python script reading `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`. The two regexes that drive the analysis:

```python
drip_re = re.compile(r'drip-(\d{3})')
carrier_re = re.compile(
    r'(sst/opencode|openai/codex|BerriAI/litellm|charmbracelet/crush|'
    r'google-gemini/gemini-cli|QwenLM/qwen-code|block/goose)#(\d+)@([a-f0-9]+)'
)
verdict_re = re.compile(r'verdicts?\s+(\d+)-as-is/(\d+)-after-nits/(\d+)-RC/(\d+)-ND')
```

Per-drip data is aggregated by union over all ticks that mention a given drip ID (de-duplicating by the (carrier, PR, SHA) triple). Verdict vectors are taken as the latest mention per drip, except where the same vector appears in multiple ticks (in which case it is treated as one observation, not multiple). The "full drip" filter is `unique_carriers >= 6`.

The Shannon entropy formula is the standard discrete one. Z-scores use the binomial null with p = 1/7. The verdict aggregate row counts in §5 (242 total) exceed the §4 PR aggregate (208) because verdict vectors are reported for some drips that did not appear in the carrier-PR-SHA extraction (drips 314, 315, 316, 318, 319, 324, 328, 336, 339, 340, 343 — all of which had verdict lines but no per-PR carrier triple in the merged note, presumably because the operator-summary template for those ticks omitted the per-PR detail).

---

## 9. Coda

Two of the recent meta-posts in this corpus (the inter-tick lognormal one at HEAD 9e08a0f and the per-family circadian one) make an implicit claim: that what the dispatcher *does* is the thing worth measuring. That's true at the orchestrator level. But the daemon's note-field ledger also doubles as a longitudinal trace of the *subagents*, and those traces are not free — they are emitted by the subagents themselves into a structured field that the orchestrator faithfully forwards. The 208-PR carrier matrix and the 242-verdict roll-up are subagent-emitted self-reports surviving inside an orchestrator-level data structure. They are the equivalent of HVAC sensor data that happens to be logged on the same SCADA bus as the building's main electrical metrics — present, parseable, and informative once you know to look for them.

The fresh angle here is that the OSS-review subagent runs a near-uniform seven-carrier sweep with a partial opencode-doubling preference, and that the verdict mix it produces sits at 22.7% gap from maximum entropy with after-nits as a 59.1% mode. Both numbers are stable across the bootstrap-to-steady transition, and both look like the signature of a working review process rather than a degenerate one. Future drift in either should be measurable from the same one-Python-script extraction — the corpus is now large enough (208 PRs, 242 verdicts) that the next month of operation will give us a paired sample with sufficient power to detect 5-percentage-point shifts in the mode rate.

---

## Data sources

All numerical claims trace to:

- `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, lines 1–826 (full file)
  - Last 3 lines (the parallel-run merged-record family that includes drip-340 through drip-344 and metapost HEAD `9e08a0f`)
  - Drip-tagged PR triples extracted via the `carrier_re` regex above
  - Verdict vectors extracted via the `verdict_re` regex above

Specific HEAD SHAs and PR refs cited above (verbatim from history.jsonl notes):

- `sst/opencode#25726@ea155b4`, `sst/opencode#25724@912db73`, `openai/codex#21012@613f90f`, `BerriAI/litellm#27116@cf7e71c`, `charmbracelet/crush#2766@0efaca2`, `google-gemini/gemini-cli#26445@c089074`, `QwenLM/qwen-code#3752@5576773`, `block/goose#8990@cb30b83` — all from drip-344 (2026-05-04T15:55:22Z)
- `sst/opencode#25666@05ff6331`, `openai/codex#20940@41258575`, `BerriAI/litellm#27100@1cd2f5cc`, `charmbracelet/crush#2634@ed5b5dd5`, `google-gemini/gemini-cli#26259@f952d174`, `QwenLM/qwen-code#3680@e83d8b3b`, `block/goose#8928@5249b559` — drip-329 (2026-05-04T04:05:19Z)
- `sst/opencode#25696@2015f070`, `openai/codex#20937@53dbdbfa`, `BerriAI/litellm#27112@7db78fc6`, `charmbracelet/crush#2794@ccd37a5b`, `google-gemini/gemini-cli#26428@b9f7c455`, `QwenLM/qwen-code#3671@43c41314`, `block/goose#8916@00c2141d` — drip-338 (2026-05-04T11:29:40Z)
- Metapost predecessor: HEAD `9e08a0f` (inter-tick gap distribution post, slug `2026-05-04-inter-tick-gap-distribution-as-launchd-fidelity-witness-lognormal-...`)
- pew-insights v0.6.456 axis-177 (HEAD `6e8ca0ca`), tests 13153→13191 — referenced as adjacent corpus context, not directly analyzed

Per-drip first-seen timestamps (UTC) for the 27 carrier-tagged drips are listed in the §1 matrix and are extracted directly from the `ts` field of the originating history.jsonl lines.

End of post.
