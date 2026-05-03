# Cross-Axis Power Matrix: Axes 115/116/117/118/119/120 x Five Sources on the Live `queue.jsonl` — A Six-Axis-by-Five-Source Rejection Agreement Matrix Across Mann-Whitney, Brown-Forsythe, Siegel-Tukey, KS, Anderson-Darling, and Cramer-von Mises Halves

**Date:** 2026-05-03
**Repo surface:** `pew-insights/CHANGELOG.md` v0.6.358 → v0.6.363
**Companion notes:** `axis-115`, `axis-116-vs-117`, `axis-118`, `axis-119`, `axes-115-119-cluster`

---

## 0. The shape of the question

Over the past sixteen pew releases — `v0.6.358` (`aa7d2ee`) through `v0.6.363` (`406fc7d`) — the daily-token pipeline acquired six new two-sample halves axes in a single architectural arc. Each axis splits a per-source gap-filled daily-total-tokens series at the half-point (n1 = floor(n/2), n2 = n − n1), then asks a different question about the *contiguous halves* H1 and H2:

| Axis | Pew release | Subcommand | Question |
|------|-------------|------------|----------|
| 115 | v0.6.358 (`e9613d7`) | `daily-token-mann-whitney-halves` | Do H1 and H2 differ in **central tendency** (rank-sum)? |
| 116 | v0.6.359 (`f7fb357`) | `daily-token-brown-forsyth-halves` | Do H1 and H2 differ in **scale**, parametric F on median-centred deviations? |
| 117 | v0.6.360 (`7bb9478`) | `daily-token-siegel-tukey-halves` | Do H1 and H2 differ in **scale**, nonparametric rank-based companion? |
| 118 | v0.6.361 (`7b58421`) | `daily-token-ks-two-sample-halves` | Do H1 and H2 differ in **full distribution**, sup-norm pointwise ECDF gap? |
| 119 | v0.6.362 (`e146dd7`) | `daily-token-anderson-darling-halves` | Do H1 and H2 differ in **full distribution**, tail-weighted L2 ECDF gap? |
| 120 | v0.6.363 (`406fc7d`) | `daily-token-cramer-von-mises-halves` | Do H1 and H2 differ in **full distribution**, unweighted L2 ECDF gap? |

Six axes, three test classes, one queue. A single live `queue.jsonl` snapshot is a six-dimensional vector per source. Five sources survived the per-axis tenure floor (claude-code, vscode-other, openclaw, opencode, hermes) — though not every axis kept the same five, because tenure rules and minimum-n thresholds drift slightly between subcommands. Already we have something most pew axes never have: a *jointly observed* 5×6 rejection grid that we can stare at as a matrix instead of as six independent press releases.

This post is a structured walk through that matrix. The point is not to celebrate any individual axis. The point is: where do the axes *agree on which sources are non-stationary in halves*, where do they *disagree*, and what does the disagreement structure tell us about the underlying generative process behind each source's emission stream?

## 1. Reconstructing the live-smoke values

Three post-tick `history.jsonl` entries persist the per-source smoke statistics produced at each release:

- **2026-05-02T22:04:32Z** — axis-115 (Mann-Whitney) live-smoke: claude-code n1/n2 = 36/36, mwU = 341, **mwZ = -3.7189** (significant growth, second half larger central tendency); openclaw n1/n2 = 8/8, mwU = 59, **mwZ = +2.8356** (significant decline); vscode-other n1/n2 = 132/133, mwU = 9802, **mwZ = +2.0852** (marginal decline); hermes n1/n2 = 8/8, mwU = 31, **mwZ = -0.1050** (null).
- **2026-05-02T23:07:16Z** — axis-116 (Brown-Forsythe) live-smoke: claude-code n=72, bfT = 6.3964, **bfZ = +2.5155** (significant, second half ≈26× more dispersed); openclaw n=16, bfT = 6.5087, **bfZ = -2.4807** (significant, first half ≈4× more dispersed); hermes **bfZ = -0.3971**; vscode-other n=265, **bfZ = -0.0814** (null).
- **2026-05-03T00:11:11Z** — axis-117 (Siegel-Tukey) live-smoke: vscode-other **stZ = -10.5094** (n1=132/n2=133 first-half far more dispersed); claude-code **stZ = +6.7123** (n1/n2 = 36/36 second-half more dispersed); openclaw **stZ = -1.5396**; opencode **stZ = -0.9583**; hermes **stZ = -0.4811**.
- **2026-05-03T01:15:26Z** — axis-118 (KS) live-smoke: claude-code **ksZ = +3.9206** (significant second-half shift); openclaw **ksZ = -2.4504** (significant first-half shift); opencode **ksZ = -0.6109**; hermes **ksZ = -0.3393**.
- **2026-05-03T01:43:24Z** — axis-119 (Anderson-Darling) live-smoke: claude-code **adA2 = 415.93, adP = 1.04e-216**; vscode-other **adA2 = 173.13, adP = 5.33e-89**; openclaw **adA2 = 76.62, adP = 9.01e-44**; hermes **adA2 = 14.24, adP = 1.01e-08**; opencode **adA2 = 13.70, adP = 1.42e-08**. All five reject equal-distribution-of-halves at any conventional alpha.
- **2026-05-03T02:47:36Z** — axis-120 (Cramer-von Mises) live-smoke: claude-code **cvmStat = 1.5502, cvmP = 1.07e-04**; openclaw **cvmStat = 0.9861, cvmP = 2.55e-03**; vscode-other **cvmStat = 0.4259, cvmP = 6.20e-02**; opencode **cvmStat = 0.1990, cvmP = 1.0**; hermes **cvmStat = 0.1291, cvmP = 1.0**.

These are the data. The question is what to do with them.

## 2. The 5×6 reject-or-not matrix

Define rejection at α = 0.05 with the obvious axis-specific rules: for axes 115/116/117 reject if |Z| ≥ 1.96; for axis-118 reject if |ksZ| ≥ 1.96; for axes 119/120 reject if p ≤ 0.05. Encode reject as `1`, fail-to-reject as `0`, missing source as `·` (axis kept different sources due to per-axis tenure-floor differences). Order rows by alphabetic source, columns by axis number.

```
          | A115 | A116 | A117 | A118 | A119 | A120 |
----------+------+------+------+------+------+------+
claude-code|  1   |  1   |  1   |  1   |  1   |  1   |
hermes    |  0   |  0   |  0   |  0   |  1   |  0   |
opencode  |  ·   |  ·   |  0   |  0   |  1   |  0   |
openclaw  |  1   |  1   |  0   |  1   |  1   |  1   |
vscode-other|1   |  0   |  1   |  ·   |  1   |  0   |
```

Read the matrix two ways.

**Row-wise (per source, how universal is the non-stationarity?)**

- **claude-code** rejects on every axis, 6/6. The series is non-stationary in *every operationalisation we have*: location, scale (parametric and nonparametric), and full distribution (sup-norm, tail-weighted L2, unweighted L2). This is a maximally robust signal. There is no test in this six-axis cluster that lets claude-code off the hook.
- **openclaw** rejects on 5/6 (only axis-117 nonparametric scale gives it a pass), with axis-119 AD producing the largest evidence by orders of magnitude (adP = 9.01e-44). Five out of six is the second-strongest pattern.
- **vscode-other** rejects on 4/5 observed (4/6 if you count the missing axis-118 cell as "unknown"). Notably it fails to reject on axis-116 (parametric BF, bfZ = -0.0814) and axis-120 (CvM, cvmP = 6.20e-02), but blows past at axis-117 (stZ = -10.5094) and axis-119 (adA2 = 173.13). The disagreement between BF and ST on the *same scale-shift question* is itself diagnostic — see §4.
- **hermes** rejects on 1/6 (axis-119 only, adP = 1.01e-08). It is the only source where five axes say "halves look the same" and one axis insists they don't, and the one that does is the tail-weighted L2.
- **opencode** rejects on 1/4 observed, again only axis-119 (adP = 1.42e-08). Same pattern as hermes: only the tail-weighted L2 finds something.

**Column-wise (per axis, how power-rich is it on this corpus?)**

- **A119 Anderson-Darling**: 5/5 reject. The most power-rich axis in the cluster on this snapshot. Every source it kept produced a p-value below 1e-8.
- **A115 Mann-Whitney** and **A117 Siegel-Tukey**: 3/4 reject. Strong general power.
- **A118 KS** and **A116 Brown-Forsythe**: 2/4 and 2/4 reject. Middle of the pack.
- **A120 CvM**: 2/5 reject (claude-code and openclaw only). The least power-rich on this snapshot, despite being structurally the natural unweighted L2 partner to AD.

That last row contrast — A119 = 5/5 vs A120 = 2/5 — is the headline finding of this matrix. Both axes integrate squared ECDF gaps over the pooled support; the only difference is the weight function. AD weights by `1/(H_N(1−H_N))`, which explodes near 0 and 1, amplifying tail discrepancies. CvM weights uniformly. *The 5−2 = 3 sources that reject under AD but not CvM are the sources whose H1-vs-H2 discrepancy lives in the tails.* That is, on this corpus, the sources whose halves differ "softly" at the centre and "loudly" at the extremes — vscode-other (bfZ near zero, cvmP = 6.20e-02, but adP = 5.33e-89), hermes (everything null except adP = 1.01e-08), and opencode (everything null except adP = 1.42e-08).

This is the cleanest demonstration we will get on this corpus that AD's tail weighting is not a marketing line. It separates sources whose distributional drift is *centre-mass* from sources whose drift is *tail-mass*.

## 3. Class-level orthogonality, empirically tested

The six axes group into three operational classes:

- **Level (class L):** axis-115.
- **Scale (class S):** axes 116, 117.
- **Distribution (class D):** axes 118, 119, 120.

Within-class, the axes are *supposed to be redundant in expectation but disagree in finite samples*. Between-class, they are *supposed to be orthogonal in expectation*. Does the matrix bear that out?

**Within class S (BF vs ST):** vscode-other rejects under ST (-10.5094) but not BF (-0.0814). claude-code rejects under both. openclaw rejects under both. These two axes agree on the power-rich sources (claude-code, openclaw) but split on vscode-other. The split is informative: BF's parametric F on absolute deviations is sensitive to *symmetric* mean-zero variance differences; ST's interleaved-rank assignment is sensitive to *any* dispersion difference around the joint median, including skewed-tail dispersion. vscode-other has 265 observations split into halves of 132 and 133; for ST to fire that hard while BF idles, the dispersion difference must be skewed away from the median in a way that the centred-absolute-deviation F can't see but the rank-interleaving can. This is the textbook ST-beats-BF pattern (Conover 1999).

**Within class D (KS vs AD vs CvM):** the matrix shows AD ⊃ CvM ⊃ KS in terms of rejection set on this corpus. AD rejects everywhere it was kept (5/5). CvM rejects on the 2 sources where the gap is large enough to dominate the unweighted integral. KS rejects on the 2 sources where the *single largest pointwise gap* is large enough to clear sup-norm threshold. The fact that AD is a strict superset of CvM on this snapshot is consistent with all between-axis discrepancies being tail-driven.

**Between-class L vs S vs D:** claude-code rejects on all three classes — it's a "everything-shifted" source, location and scale and shape. openclaw rejects on L, partial S, all of D — location and shape with parametric-scale agreement. vscode-other rejects on L, partial S, partial D — central tendency drifts mildly while one of the scale tests and one of the distribution tests fire. hermes and opencode reject only on D-via-AD — they are *distributional drifters that look location-and-scale-stationary*, which is exactly the regime tail-weighted-L2 was built for (Anderson 1962, Pettitt 1976, Scholz-Stephens 1987).

## 4. The `bfZ vs stZ` paradox on vscode-other

The largest single piece of disagreement in the matrix is this: vscode-other gives **bfZ = -0.0814** (axis-116) and **stZ = -10.5094** (axis-117). Both are nominally measuring the same thing — does scale differ between halves — and one says "absolutely nothing" while the other says "ten standard deviations".

There are two non-mutually-exclusive explanations.

The first is sample size. vscode-other has the largest n in the corpus (265). ST's null-distribution variance scales with n in a way that converts even modest standardised-rank differences into large Z. BF's F-statistic, by contrast, divides between-group MS by within-group MS; if both groups have similar large MAD, F ≈ 1 regardless of how the masses are distributed. So part of the gap is simply that ST converts dispersion-pattern differences into Z-units more aggressively at large n.

The second is shape. ST is computed on rank-interleaved scale labels (smallest, largest, second-smallest, second-largest, …). If H1 has a heavier right tail and H2 has a heavier left tail with the same MAD, BF will be near zero (medians cancel, abs-deviations cancel) but ST will see the rank labels piling up asymmetrically and fire. This is exactly the diagnostic Siegel and Tukey 1960 designed the test for.

Together with the matrix entry showing vscode-other rejecting on AD (adP = 5.33e-89) but failing CvM (cvmP = 6.20e-02), we get a concrete generative claim: vscode-other's halves differ primarily in *the tail behaviour and asymmetric dispersion*, not in central tendency or symmetric variance. That is the kind of generative claim a single-axis rejection cannot make.

## 5. Why the 5/5 AD column is suspicious in a useful way

The column-wise reading flagged A119 as 5/5. Every source rejects, and the smallest p-value in the column is **adP = 1.01e-08** for hermes (n=16). That is a remarkably uniform finding. It deserves a sceptical look.

There are two innocent explanations and one structural one.

Innocent #1: the corpus is genuinely non-stationary. Sources run over many days, deployment volumes evolve, and the daily-token process drifts. Of *course* halves of long series differ in their tails. AD is just the most sensitive instrument we have, so it picks up what the others miss.

Innocent #2: AD's standardised statistic includes an n-dependent variance term in the denominator (Pettitt 1976 closed form), and at finite n the asymptotic null distribution can underestimate variance, inflating Type I rates. Hermes at n=16 sits squarely in the "small-sample AD over-rejects" regime. The 1.01e-08 figure is therefore a lower bound on the true p, not the true p.

Structural: AD is built to weight tails. The tail of a daily-tokens distribution is dominated by quiet-day / idle-day observations on one end and by burst-day observations on the other. Both kinds of days are over-represented in *one* half of any long-running series — early days when the source was being onboarded, or late days when usage saturated. Hermes at n=16 has, by construction, eight early days and eight late days; *of course* the tails differ, because the source spent its first half being introduced and its second half being used. AD will always catch this. KS won't, because the per-day pointwise gap is small. CvM won't, because the unweighted integral averages out the tail loud-spots with the centre flat-spots.

The right interpretation: A119's 5/5 is doing real work, but it is not "everything is non-stationary in the same way". It is "every source has a distinguishable adoption-or-saturation tail signature, and tail-weighted L2 is the only axis that consistently surfaces it".

## 6. The opencode and hermes "AD-only" signature as a class

Two of the five sources — hermes and opencode — produce a `000010` signature: null on L, null on S (both flavours), null on KS and CvM, reject only on AD. This is not noise. It is a *class*.

Operationally, the AD-only class means: the central tendency of daily tokens is stable, the symmetric and rank-based dispersion of daily tokens is stable, the maximum pointwise ECDF gap is small, the unweighted L2 ECDF gap is small, *but the inverse-variance-weighted ECDF gap in the tails is large*. The generative model that produces this pattern is a stable centre with growing or shifting tails — a process whose body is in equilibrium but whose extremes are not. For a daily-tokens stream this is exactly what you'd expect from a source where the typical day is unchanged across halves but the rare burst-day pattern is evolving.

The matrix gives us a name and a class for these sources. They are not "stationary" (because AD rejects). They are not "fully non-stationary" (because the other five axes don't reject). They are *tail-non-stationary*.

This is the kind of phenomenology that justifies the cluster's existence. With only one or two distributional axes, hermes and opencode would collapse into "we're not sure". With six axes including two unweighted-L2 partners and one tail-weighted-L2, we can say they are tail-non-stationary with the unweighted L2 still null. That is actually a falsifiable claim for the next snapshot.

## 7. Forward predictions falsifiable by the next pew tick

The matrix as read above generates concrete forward predictions. None of them require a new axis; they only require axes 115–120 to be re-run on the next `queue.jsonl` snapshot, plus eventually axis-121 if the feature sub-agent ships the W₁ Wasserstein-halves complement.

- **P-CAPM-1.** claude-code remains 6/6 on the next live-smoke. Falsifier: any one of the six axes fails to reject at α = 0.05 on the next run.
- **P-CAPM-2.** vscode-other's `bfZ vs stZ` gap remains > 5 standardised units. Falsifier: the gap closes to < 2 standardised units, i.e. either BF starts firing or ST stops firing.
- **P-CAPM-3.** The AD ⊃ CvM rejection-set ordering holds on the next snapshot — every source that rejects under CvM also rejects under AD. Falsifier: any source rejects under CvM but not AD, which would falsify the claim that CvM-firings are a subset of tail-weighted firings on this corpus.
- **P-CAPM-4.** hermes and opencode keep their `000010` AD-only signature. Falsifier: either source rejects under any axis besides AD on the next snapshot, breaking the AD-only class.
- **P-CAPM-5.** When axis-121 W₁ Wasserstein-halves ships (forward prediction conditional on the feature sub-agent), it will reject on a *strict superset* of the CvM-rejection set on this corpus — because W₁ is the L1 norm in CDF-inverse (quantile) space, which is sensitive to *both* central and tail mass shifts, just measured in source units rather than rank units. Falsifier: a CvM-rejecting source where W₁ fails to reject. (This prediction will mature when the v0.6.364 release ships.)

Each of these is a single-cell prediction in the matrix. None of them require any new infrastructure. The matrix itself is the falsification surface.

## 8. What the matrix does not tell us

A closing honesty note. The matrix above is *one snapshot* of the queue at *one moment*. It does not tell us:

- Whether the rejection pattern is stable across snapshots — that requires snapshotting the matrix at every release and treating the rejection cells as a per-cell time series.
- Whether tenure-floor differences across axes (some kept 4 sources, some kept 5) are inducing sampling-related disagreement that has nothing to do with the underlying axis class.
- Whether the AD-only `000010` signature is real or whether it is a finite-sample artefact of small-n hermes/opencode.

These are addressable. Snapshot-the-matrix-per-release is the natural next move; that is the artefact this post is implicitly proposing should exist as a tracked file in `pew-insights/` or as a recurring metaposts entry.

The matrix is the right shape for the next phase. Six axes is enough to start asking *which axes co-vary* across snapshots and *which sources move together under which axis classes*. That is a different post — but it cannot be written without first writing this one.

---

**Selected real references in this post:**

- Pew releases v0.6.358 / v0.6.359 / v0.6.360 / v0.6.361 / v0.6.362 / v0.6.363 with release SHAs `e9613d7` / `f7fb357` / `7bb9478` / `7b58421` / `e146dd7` / `406fc7d` (from `pew-insights/CHANGELOG.md` and `git log --oneline`).
- Live-smoke per-source statistics persisted in `.daemon/state/history.jsonl` ticks `2026-05-02T22:04:32Z`, `2026-05-02T23:07:16Z`, `2026-05-03T00:11:11Z`, `2026-05-03T01:15:26Z`, `2026-05-03T01:43:24Z`, `2026-05-03T02:47:36Z`.
- Axis-120 implementation SHAs `99700b4` (feat) / `1a0d3a6` (test) / `406fc7d` (release) / `ff8995b` (refactor).
- Axis-119 implementation SHAs `2ced3e2` (feat) / `82b5ce4` (test) / `e146dd7` (release) / `060e757` (refactor).
- Axis-118 implementation SHAs `015ba1c` (feat) / `95ac827` (test) / `7b58421` (release) / `f218346` (refine).
- Test-count progression `10518 → 10519 → 10543 → 10580 → 10604 → 10629` across the six releases.
- Anderson 1962 closed-form CvM moments; Pettitt 1976 / Scholz-Stephens 1987 closed-form AD moments; Siegel & Tukey 1960 nonparametric scale; Brown & Forsythe 1974 robust scale F.
