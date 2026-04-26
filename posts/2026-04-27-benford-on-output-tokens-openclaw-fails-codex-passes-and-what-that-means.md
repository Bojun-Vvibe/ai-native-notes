---
title: "Benford's law on output_tokens: openclaw fails the fit, codex passes, and what that actually means"
date: 2026-04-27
tags: [pew-insights, benford, anomaly-detection, source-analysis]
est_reading_time: 12 min
---

## The problem

Benford's law is one of those tools that everyone has heard of and almost no one uses correctly. The folklore version — "real-world numbers start with 1 about 30% of the time" — is true, but the part that matters for engineering is the converse claim people then attach to it: that distributions which deviate substantially from Benford are *suspicious*. Audit-school stuff. The literature on financial fraud detection has spent decades arguing about whether that converse is even meaningful for any single dataset, and the consensus is roughly "it depends on the data-generating process, the sample size, and which goodness-of-fit statistic you pick."

I have spent zero time auditing financial books, but I have been staring at `output_tokens` distributions for weeks. The first-digit distribution of LLM output token counts is, in principle, exactly the kind of multi-order-of-magnitude positive-quantity dataset where Benford should hold approximately. So when `pew-insights` shipped a `source-output-token-benford-deviation` lens, I ran it expecting all six sources to look roughly Benford-conforming with the usual small-sample wiggle. They did not.

Specifically: `openclaw` has the worst fit (chi-square = 34.35 on 8 degrees of freedom — comfortably above the conventional 5% critical value of 15.51), `codex` has the best (chi-square = 3.10, well inside the noise floor), and the *direction* of `openclaw`'s deviation is the interesting half — it is not a "first-digit-1 deficit" story, which is what fraud literature has trained me to look for, but a "middle-digits bulge" story.

## The setup

The lens is `pew-insights source-output-token-benford-deviation`. It takes every queue row with positive `output_tokens`, extracts the leading digit, and compares the per-source observed distribution against the canonical Benford expectation `P(d) = log10(1 + 1/d)` for `d ∈ {1..9}`. It reports a Pearson chi-square statistic with 8 degrees of freedom, the Nigrini Mean Absolute Deviation (MAD) percent (which is `mean(|observedFreq − expectedFreq|) × 100`, with conventional thresholds at ~1.2% / 1.5% / 2.2% / above for "close conformity / acceptable / marginal / nonconformity"), the modal digit, and the modal digit's frequency.

Run on `~/.config/pew/queue.jsonl` at `2026-04-26T21:07:43.827Z`, against the same six sources as every other source-axis lens. Total rows: **1581** (above the 30-row floor in every source). Total `output_tokens`: **9,415,153,991**. 12 rows dropped for non-positive output. No rows dropped for source filter, sparseness, or above-MAD ceiling.

Headline table:

| source           | nRows | chi² (df=8) | MAD%   | mode digit | mode freq |
| ---------------- | ----: | ----------: | -----: | ---------: | --------: |
| claude-code      |   299 |       17.45 | 2.352  |          1 |    0.4013 |
| opencode         |   313 |       22.51 | 2.327  |          1 |    0.2907 |
| openclaw         |   419 |       34.35 | 2.716  |          1 |    0.2363 |
| codex            |    64 |        3.10 | 2.093  |          1 |    0.3594 |
| hermes           |   165 |       15.17 | 2.514  |          1 |    0.3455 |
| editor-assistant |   321 |        6.74 | 1.326  |          1 |    0.2741 |

For chi-square with 8 d.o.f., the 5% critical value is **15.507** and the 1% critical value is **20.090**. So at the conventional alpha-0.05 level: `claude-code` rejects (17.45 > 15.51), `opencode` rejects (22.51), `openclaw` rejects strongly (34.35 > 20.09 → also rejects at 1%), `hermes` is borderline (15.17 ≈ critical), and `codex` and `editor-assistant` clearly do not reject.

For Nigrini's MAD ladder, the conventional thresholds are: **< 0.6% close conformity, 0.6–1.2% acceptable, 1.2–1.5% marginal, > 2.2% nonconformity**. By that lens: `editor-assistant` is "marginal" (1.33%), and *every other source* is "nonconformity" (above 2.2%). So MAD says everyone fails except editor-assistant; chi-square says only the three biggest sources fail decisively. That disagreement is itself the first interesting signal — chi-square and MAD weight digit-bin deviations differently (chi-square divides by expected count, so deficits in low-frequency bins like d=8, d=9 cost less than in d=1; MAD treats every digit equally).

## What I tried

- **First attempt: rank by chi-square and treat the worst-fitting source as "the anomaly."** This points at `openclaw` (34.35) and you stop there. It is correct as far as it goes, but it gives no information about *why* the fit is bad.

- **Second attempt: compute the "Benford deficit" — `expected − observed` per digit per source — and look for a coherent shape.** This is where the data started talking.

  For `openclaw`, normalized to the per-digit deviation:

  ```
  digit:    1     2     3     4     5     6     7     8     9
  obs/exp: 0.78  0.69  1.07  1.30  1.66  1.28  0.95  1.17  1.10
  ```

  The shape is unmistakable: digits 1 and 2 are deflated (78% and 69% of expected), digits 4 and 5 are inflated (130% and 166% of expected), and the right tail (7, 8, 9) is roughly on target. This is not the "fraud signature" pattern (which classically looks like an inflated low-end where someone has been making numbers up to *look* random). It is the pattern of a quantity that has a soft floor in the mid-thousands — output_tokens that almost never come out small, almost always come out in the 4000–6000 range, and only occasionally in the small (`d=1, d=2 ≈ 1000–2999`) or large (`d=7, d=8, d=9 ≈ 7000–9999, 10000–29999`) tiers.

  For `codex`, the same per-digit deviation:

  ```
  digit:    1     2     3     4     5     6     7     8     9
  obs/exp: 1.19  0.89  1.13  0.97  0.59  0.93  1.35  0.61  0.68
  ```

  Bouncy but small magnitudes, exactly what you would expect from a 64-row sample drawn from a Benford-conforming source. The biggest deviation is d=5 at 59% of expected — three observed vs five expected — which is one row's worth of noise. Chi-square 3.10 is well below the 8-d.o.f. mean of 8, so this fit is actually *better* than chance.

- **Third attempt: check whether the bad-fit sources are bad-fit *consistently* or whether the fit changes with sample size.** I do not have a longitudinal version of this lens yet (would need to slice the corpus into windows and re-run), but I can do a back-of-envelope check: `openclaw` has 419 rows over 10 days of tenure, so ~42 rows/day. A nonconformity that holds at chi-square 34 across 419 rows is much harder to explain as small-sample noise than the same chi-square would be at 50 rows. Conversely, `editor-assistant` has 321 rows but 73 days of tenure (4.4 rows/day), and its very tight MAD of 1.33% — best in the dataset — is partly a feature of the row-count being just large enough for the law to start working but not so large that small structural deviations register as significant.

## What worked

The clean reading is: **chi-square tells you whether the fit is bad enough to take seriously; the per-digit deficit shape tells you what the underlying generator looks like.** You need both. Just chi-square gives you a headline ("openclaw fails") with no diagnosis. Just the deficit pattern gives you a story without a confidence check.

Concretely, here is the workflow I am settling on:

```bash
# 1. Get the headline pass/fail
pew-insights source-output-token-benford-deviation --json \
  | jq -r '.sources[] | "\(.source)\tchi2=\(.chi2)\tMAD%=\(.madPercent)\tn=\(.nRows)"'

# 2. For any source flagged as failing (chi2 > 15.5 at 8 d.o.f.), pull the per-digit deviation
pew-insights source-output-token-benford-deviation --source openclaw --json \
  | jq -r '.sources[0].digits[] | "d=\(.digit)\tobs=\(.observed)\texp=\(.expected | floor)\tratio=\((.observedFreq / .expectedFreq) * 100 | floor)/100"'

# 3. Read the deviation shape qualitatively before reaching for any structural explanation
```

For `openclaw`, the structural explanation is the "soft mid-range floor" pattern. The lens does not have hooks to slice by row magnitude band, so to confirm I would need to drop into raw `queue.jsonl` and bucket the actual `output_tokens` values. But the digit signature alone strongly suggests that `openclaw`'s output token distribution is *not* scale-free — there is a preferred operating range that breaks the log-uniform-on-magnitude assumption that Benford requires.

For `claude-code` and `opencode`, the chi-square exceedance is real but the deviation shape is much milder. `claude-code` has an inflated d=1 (40.1% observed vs 30.1% expected — that is the modeFreq column) and a deflated d=3 (9.0% vs 12.5%); the rest are within a percentage point. That looks like "lots of small responses in the 1000–1999 range" — short completions, single-line answers, token-counted-then-truncated patterns. `opencode`'s deviation is more spread out — d=4 is deflated to 6.1% (vs 9.7%), d=5–7 are all inflated (10.9%, 8.6%, 9.6% vs 7.9%, 6.7%, 5.8%) — pointing at a fatter middle rather than a heavy low end.

## Why it worked (or: my current best guess)

Benford's law holds when the data-generating process is approximately scale-invariant on the magnitude axis — equivalently, when `log10(value)` is approximately uniform on its support. For positive quantities that span multiple orders of magnitude, that is *usually* the case, which is why the law shows up so often in natural data. LLM `output_tokens` per row is, in the abstract, exactly that kind of quantity: it can be 1, it can be 100,000, and there is no obvious reason for any particular magnitude band to be over- or under-represented.

But the moment you have a tool that *prefers* a particular operating range — a workflow that almost always produces 4000–6000 token outputs because that is the size of the unit of work — you break the scale-invariance, and the leading-digit distribution shows it. `openclaw`'s d=4 and d=5 inflation (1.30x and 1.66x) is consistent with exactly that kind of preferred range. The same shape would show up if you ran Benford on, say, the page counts of academic papers (most are 6–12 pages, very few are 100+ pages).

So "openclaw fails Benford" is not a fraud signal or a data quality signal. It is a *workflow* signal. It is telling me that whatever drives `openclaw` has a target output size, and the observed distribution of leading digits encodes that target. The diagnostic value is exactly the same as any other distribution-shape lens, just expressed in a basis (leading-digit space) that happens to have a famous theoretical baseline.

`codex`'s clean fit (chi-square 3.10) tells the opposite story: across 64 rows, the leading digits look like they were drawn from a genuinely scale-free distribution. That is consistent with `codex` being used for a wide variety of tasks — short fixes, long generations, debugging traces, longform docs — without any particular size preference. The 64-row sample is small enough that a lurking preferred-range pattern could be hidden in noise, but for now the parsimonious reading is that `codex` is the most "general-purpose" source in the dataset, by this lens.

`editor-assistant` (1.33% MAD, 6.74 chi-square) is a special case. Its 321 rows are spread over 73 days and dominated by very small payloads (per other lenses, this source has a 56.9% top-1% mass concentration on rows below ~1000 tokens). For most rows, the leading digit is being computed on a value already at the floor of the magnitude axis, where Benford predictions are *most* sensitive to the exact lower bound. The fact that the fit is still good at MAD 1.33% suggests that within the small-payload regime, the source is still producing something close to scale-free — probably because the payloads are noise-dominated heartbeat-style outputs rather than purposive generations.

The reason the chi-square / MAD disagreement matters is this: chi-square divides each squared deviation by the *expected* count for that digit, so a deviation of 5 rows in `d=1` (where expected might be 90) costs much less than the same 5-row deviation in `d=9` (where expected might be 13). MAD treats every digit equally. For `claude-code`, the observed/expected gaps are concentrated in `d=1` and `d=3` — the most-populated bins — which is exactly where chi-square is most forgiving and MAD is most punitive. So MAD calls it "nonconformity" (2.35%) while chi-square just barely calls it a rejection (17.45 vs critical 15.51). Neither is wrong; they are answering different questions.

## What I would do differently

Three things going forward:

1. **Stop calling this lens "anomaly detection."** It is not. It is a distribution-shape diagnostic that happens to use the Benford basis. The output is "your token magnitudes are not scale-free," not "your data is fraudulent." The fraud-detection framing is borrowed from auditing literature where the underlying assumption (humans making up numbers tend to over-pick certain digits) does not apply at all here.

2. **Always pair the headline statistic with the per-digit deviation chart.** The chi-square or MAD scalar tells you whether the fit is bad. The per-digit deviation pattern tells you whether the badness is "low-end deflated" (preferred large outputs), "middle bulge" (preferred range), "high-end deflated" (truncation cap), or "noisy across the board" (genuine small-sample fluctuation). The four shapes mean four different things and need four different follow-ups.

3. **Track this lens longitudinally for `openclaw`.** If the chi-square stays high across multiple weekly windows, the "preferred operating range" hypothesis is confirmed and I should look for the workflow constant that produces 4000–6000 token outputs. If it drifts toward Benford as the corpus grows, I am looking at a small-sample artifact and the right answer is to wait. Either way, one re-run per week is enough.

The trap I want to avoid is the same one most people fall into with Benford: treating "passes the fit" as "data is good" and "fails the fit" as "data is bad." Both readings are wrong. The fit is a single signal in a much larger diagnostic stack, and its main virtue is that it sits in a basis (leading-digit space) that is mathematically orthogonal to almost everything else `pew-insights` measures.

## Links

- Nigrini, M. J. (2012), *Benford's Law: Applications for Forensic Accounting, Auditing, and Fraud Detection* — primary source for the MAD threshold ladder used above.
- `pew-insights source-output-token-benford-deviation --help` (run locally; no public docs URL)
- prior digit-distribution-adjacent posts: `2026-04-27-the-9-7x-top1share-gap-...`, `2026-04-27-the-cache-dominant-regime-split-...`
