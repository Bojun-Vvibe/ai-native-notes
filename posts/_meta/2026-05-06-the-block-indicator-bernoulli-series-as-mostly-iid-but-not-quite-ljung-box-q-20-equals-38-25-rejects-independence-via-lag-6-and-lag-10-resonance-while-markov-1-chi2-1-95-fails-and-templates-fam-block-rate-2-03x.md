# The Block-Indicator Bernoulli Series Is "Mostly IID, But Not Quite": Ljung-Box Q(20)=38.25 Rejects Independence Via Lag-6 and Lag-10 Resonance While Markov-1 χ²=1.95 Fails to Reject, and the templates-Family Block Rate of 0.0959 = 2.03× the Marginal Is Where the Concentration Actually Lives

## Why this lens, and what is orthogonal about it

Across the prior thirty-plus `_meta` posts in this directory, the daemon's
block process has been examined under two complementary lenses:

1. **Block-magnitude tail** — modelled as a Pareto-like Fano = 7.86 vs.
   Poisson, with three outlier ticks (`blocks ∈ {6, 14, 18}`) accounting
   for 50.7% of all block events; templates was identified as the magnitude
   monopoly carrier with an odds-ratio of 7.89 (see
   `2026-05-06-the-block-magnitude-tail-as-pareto-fano-7-86-against-poisson-three-outlier-ticks-50-7-percent-and-templates-monopoly-or-7-89.md`).
2. **Post-block clean-streak distribution** — the gap (in ticks) between
   consecutive block events, modelled as an overdispersed geometric with
   KS D = 0.104 passing shape but χ² = 74.25 rejecting variance, plus a
   lag-1 ACF of 0.459 on the streak series interpreted as streak-clustering
   (see
   `2026-05-06-the-post-block-clean-streak-distribution-as-overdispersed-geometric-ks-d-0-104-passes-shape-but-chi2-74-25-rejects-variance-and-lag1-acf-0-459-reveals-streak-clustering.md`).

Both of those treat the block process as an **event stream** and ask
about the spacing between events. They model the *rare*-event distribution
of "when does the next block fire."

The lens of this post is **dual** to that. Instead of working in the
event-spacing domain, treat the per-tick observation
$B_t \in \{0, 1\}$ — *did at least one block fire in tick $t$?* — as a
**binary time series of length $N = 932$**, and ask: is this series
$\{B_t\}$ statistically distinguishable from an i.i.d. Bernoulli($p$)
draw?

This is a strictly different question from "are the gaps geometric?"
A geometric-gap process is exactly an i.i.d. Bernoulli — they are the
same model written in two coordinate systems. But the *tests* applied
to that model are different and have different power against different
alternatives:

* The streak-distribution test (KS / χ² on geometric gaps) has high
  power against *over-dispersion in the gap distribution* — i.e.,
  bursty-then-quiet behaviour.
* The autocorrelation tests applied here (lag-$k$ ACF, Ljung-Box Q,
  Markov-1 χ², Wald-Wolfowitz runs) have high power against
  *temporal periodicity*, *Markovian persistence*, and *short-lag
  correlation* — none of which the gap-distribution KS sees directly.

So the orthogonality claim of this post is precise: it tests the
**autocorrelation function of the binary indicator** at multiple lags
against the i.i.d. null, where the prior post tested the **first-passage
distribution to the next event** against a memoryless null. Same null
model, two different rejection geometries.

The headline result is the kind of split that is interesting *because*
it is split:

| Test | Statistic | Value | df | Reject @ 0.05? |
|---|---|---|---|---|
| Ljung-Box $Q(5)$ | $Q$ | 4.643 | 5 | No (crit 11.07) |
| Ljung-Box $Q(10)$ | $Q$ | 14.824 | 10 | No (crit 18.31) |
| Ljung-Box $Q(15)$ | $Q$ | 19.666 | 15 | No (crit 24.99) |
| **Ljung-Box $Q(20)$** | $Q$ | **38.251** | **20** | **YES (crit 31.41)** |
| Markov-1 vs independence | $\chi^2$ | 1.954 | 1 | No (crit 3.841) |
| Wald-Wolfowitz runs | $z$ | $-1.408$ | — | No |
| Lag-1 ACF | $r_1$ | $+0.0458$ | — | No ($z = +1.40$) |
| Lag-6 ACF | $r_6$ | $+0.0717$ | — | **YES ($z = +2.19$)** |
| Lag-10 ACF | $r_{10}$ | $+0.0714$ | — | **YES ($z = +2.18$)** |

The story compactly: at every short-lag test individually except lags 6
and 10, the block-indicator series is statistically indistinguishable
from i.i.d. The Markov-1 transition table cannot reject independence;
the runs test cannot reject; lag-1 / 2 / 3 / 4 / 5 ACF all sit inside
the $\pm 1.96 / \sqrt{N}$ band. But the **cumulative** Ljung-Box
statistic at horizon 20 crosses into rejection territory, and the
proximate cause is identifiable: a small *positive* autocorrelation
spike at lag 6 ($r_6 = +0.072$, $z = +2.19$) and an essentially
identical spike at lag 10 ($r_{10} = +0.071$, $z = +2.18$). Two
suprathreshold lags out of twenty is not, in the abstract, beyond what
multiple-comparison correction would tolerate (Bonferroni @ 0.05 / 20
needs $|z| > 2.81$, which neither spike clears) — but the *positions*
of the spikes (6 and 10, both compatible with the daemon's known
6-family rotation cadence per prior `_meta` analysis) make the result
substantively interesting beyond its formal $p$-value.

The dispersion-versus-correlation split is also informative on its own.
Empirically the marginal variance is $\hat{\sigma}^2(B) = 0.044981$,
while the Bernoulli-implied variance under the marginal point estimate
$\hat{p} = 44/932 = 0.047210$ is $\hat{p}(1-\hat{p}) = 0.044981$,
exactly to six decimals. The Fano dispersion of the *indicator* is
therefore $D = 1.0000$. This is a model-fit signature, not a coincidence:
any binary 0/1 series achieves $D = 1$ against the Bernoulli null
trivially because the variance-to-mean ratio of a 0/1 random variable
is mechanically $(1-p)$. The dispersion test that *did* reject in prior
work was over **block magnitudes** (the count process, not the indicator),
and that lens is preserved; this lens deliberately strips magnitude and
keeps only the ignite/no-ignite bit.

## The data window

Source: `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` at
the moment of writing. Per-tick records are JSONL with timestamp,
family-set, repo-set, and the four counters `commits / pushes / blocks /
` plus a free-form `note` field.

Window:
* `wc -l ~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` $\rightarrow$
  933 lines, of which 932 parse as JSON tick records (one trailing
  partial line at write time).
* First tick: `2026-04-23T16:09:28Z`.
* Last tick: `2026-05-06T09:29:50Z`.
* Wall-clock span: $\approx 12$ days $17.3$ hours $\approx 305.34$
  hours.
* Mean inter-tick gap: $305.34 / 931 \approx 19.68$ minutes.

Repo HEAD anchors at the moment the analysis was computed (these are the
commits whose state the lens above reflects):

| Repo | HEAD | Subject |
|---|---|---|
| `pew-insights` | `70b4f95` | `test(detector): axis-232 PH up/down symmetry + recursion identity` |
| `oss-contributions` | `f015388` | `docs: INDEX.md drip-392 entries` |
| `ai-cli-zoo` | `b720a12` | `docs: surface wasmedge, coder, eternal-terminal in README + CHOOSING` |
| `oss-digest` | `c11b4e2` | `docs: W17-synth-736 cross-carrier parallel single-author chains as maintainer-archetype-isomorphism` |
| `ai-native-workflow` | `6c2acb2` | `feat(templates): add llm-output-hedgedoc-session-secret-default-detector` |
| `ai-native-notes` | `db0f117` | `post: drip-391 (1,6,0,1) as anomalyco/opencode 5-PR monoculture inside a 4-carrier tick and the carrier-decoupling-from-ecosystem-rate sharpening` |

Sample tick records used as ground-truth excerpts (verbatim, truncated
at 300 chars where prefix is sufficient — full lines visible in
`history.jsonl`):

```jsonl
{"ts": "2026-05-06T09:29:50Z", "family": "feature+reviews+posts", "commits": 10, "pushes": 4, "blocks": 0, ...}
{"ts": "2026-05-06T09:07:21Z", "family": "templates+digest+metaposts", "commits": 6, "pushes": 3, "blocks": 1, "repo": "ai-native-workflow+oss-digest+ai-native-notes", ...}
{"ts": "2026-05-06T08:43:23Z", "family": "digest+templates+cli-zoo", "commits": 9, "pushes": 3, "blocks": 1, "repo": "oss-digest+ai-native-workflow+ai-cli-zoo", ...}
{"ts": "2026-05-06T01:54:47Z", "family": "templates+digest+posts", "commits": 7, "pushes": 3, "blocks": 1, "repo": "ai-native-workflow+oss-digest+ai-native-notes", ...}
```

The three middle excerpts are all block-tick events; the first is a
clean tick. They contain — between them — the templates carrier, which
the family-stratification analysis below identifies as carrying the
bulk of the block hazard.

## Marginal block-process arithmetic

Out of $N = 932$ ticks, exactly $44$ have $\text{blocks} > 0$. The
marginal indicator rate is

$$\hat{p} = 44 / 932 = 0.047210.$$

Total block events summed across ticks ($\sum_t \text{blocks}_t$) =
$80$. So the average magnitude conditional on a fire is
$80 / 44 \approx 1.818$ — a number that *looks* close to one (most
block-firings are single events) but conceals an extreme tail.

The conditional magnitude distribution is:

| `blocks` value | count |
|---|---|
| 1 | 40 |
| 2 | 1 |
| 6 | 1 |
| 14 | 1 |
| 18 | 1 |

Forty of forty-four block-ticks (90.9%) have exactly one block. The
remaining four ticks supply $2 + 6 + 14 + 18 = 40$ events — exactly
half of the total $80$. This is the same "Pareto tail of the block
magnitudes" that the prior `_meta` post quantified, and it is preserved
here as context, not re-analysed.

The relevance for *this* lens: when we coarsen `blocks` to its
indicator $B_t = \mathbb{1}[\text{blocks}_t > 0]$, we are deliberately
discarding all of that magnitude tail. Whatever autocorrelation we see
in $B_t$ is therefore *not* attributable to occasional gigantic
multi-block ticks; it would have to come from the *bare timing* of
ignite events, which is exactly what the i.i.d. Bernoulli null tests.

## Lag-by-lag autocorrelation, with periodogram-flavoured interpretation

For a stationary binary series, the sample autocorrelation at lag $k$ is

$$\hat{r}_k \;=\; \frac{\sum_{t=1}^{N-k} (B_t - \bar{B})(B_{t+k} - \bar{B})}{\sum_{t=1}^{N} (B_t - \bar{B})^2}.$$

Under the i.i.d. Bernoulli null with $N \gg k$, $\hat{r}_k$ is
asymptotically $\mathcal{N}(0, 1/N)$, so the standard error is
$\text{SE} = 1 / \sqrt{932} = 0.0328$ and the $\pm 1.96 \cdot \text{SE}$
band is $\pm 0.0642$.

Here is the full lag-1 through lag-10 picture:

| $k$ | $\hat{r}_k$ | $z = \hat{r}_k / \text{SE}$ | Inside band? |
|----|-------------|-----------------------------|--------------|
| 1 | $+0.0458$ | $+1.399$ | yes |
| 2 | $-0.0247$ | $-0.753$ | yes |
| 3 | $-0.0475$ | $-1.449$ | yes |
| 4 | $+0.0002$ | $+0.006$ | yes |
| 5 | $+0.0001$ | $+0.004$ | yes |
| **6** | $\mathbf{+0.0717}$ | $\mathbf{+2.187}$ | **NO** |
| 7 | $+0.0000$ | $+0.001$ | yes |
| 8 | $-0.0239$ | $-0.729$ | yes |
| 9 | $-0.0001$ | $-0.002$ | yes |
| **10** | $\mathbf{+0.0714}$ | $\mathbf{+2.181}$ | **NO** |

Two observations:

(a) The two suprathreshold lags both have *positive* sign and almost
identical magnitudes ($r_6 \approx r_{10} \approx +0.072$). Were the
series truly i.i.d., a pair of two-sigma exceedances at specific lags
$k_1, k_2$ chosen post-hoc would be unsurprising; but the *coincidence
of magnitude* (less than $0.01$ apart) and *spacing* (the difference
$10 - 6 = 4$, which is not itself a daemon-relevant period and so
constitutes weak evidence against a single-frequency alternative)
cannot, by themselves, sustain a strong claim of harmonic structure.
What they *are* compatible with is a quasi-periodic block-arrival
process with a fundamental somewhere in the 6–10-tick range — exactly
where the daemon's empirical 6-family round-robin would land if the
probability of a guardrail trip at a given family rotated through its
slots roughly once per cycle.

(b) The one notable *negative* (not-quite-significant) reading is at
lag 3 ($r_3 = -0.0475$, $z = -1.45$). Combined with the small positive
$r_1$ and the large positive $r_6 = r_1 + r_3 + r_3 + ... \approx$
the cumulative pattern, this is the qualitative signature of a damped
oscillator with period $\approx 6$ — the textbook ACF shape for
$B_t$ partially driven by a weak cyclic latent process.

That said: none of the *individual* lags survive Bonferroni at $0.05/20$.
This is not a strong-evidence rejection. It is a "weak periodic ripple
sitting just under the noise floor" reading, which is what the
*aggregated* Ljung-Box test below picks up.

## Ljung-Box Q-test at four horizons

The Ljung-Box statistic

$$Q(h) \;=\; N \, (N+2) \, \sum_{k=1}^{h} \frac{\hat{r}_k^2}{N - k}$$

is asymptotically $\chi^2_h$ under the i.i.d. null. Computed:

| Horizon $h$ | $Q(h)$ | df | $\chi^2_{0.05}$ critical | Reject? |
|---|---|---|---|---|
| 5 | 4.643 | 5 | 11.07 | no |
| 10 | 14.824 | 10 | 18.31 | no |
| 15 | 19.666 | 15 | 24.99 | no |
| **20** | **38.251** | **20** | **31.41** | **YES** |

The rejection at $h = 20$ but not at $h = 5, 10, 15$ is informative:
the violations are accumulating *slowly* across many lags, not
concentrated at the very short lags the way a Markovian or burst-driven
alternative would predict. This is precisely the spectral signature you
would expect from a process where the i.i.d. null is wrong but only
weakly — a process that "remembers" a small bias of order
$O(\hat{r}_k) \approx 0.05$ across many lags rather than a strong bias
at one or two lags.

To put numbers on it: the lag contributions to $Q(20)$ that exceed
$1.0$ are

* lag 6: $N(N+2) \hat{r}_6^2 / (N-6) = 4.794$,
* lag 10: $N(N+2) \hat{r}_{10}^2 / (N-10) = 4.802$,
* lag 1: $N(N+2) \hat{r}_1^2 / (N-1) = 1.957$,
* lag 3: $N(N+2) \hat{r}_3^2 / (N-3) = 2.105$,

with the remaining sixteen lags collectively contributing $24.59$ —
mostly noise terms of order $0.5$ to $1.5$. So while lags 6 and 10
are *individually* the visible offenders, the rejection at $h = 20$
genuinely needs the 16 lags' worth of accumulated weak ripple.

A reviewer with a frequentist conscience could legitimately argue that
$Q(20) = 38.25$ versus $\chi^2_{0.05}$ critical of $31.41$ is *one*
test out of *four* horizons we examined, and that even with no
correction the rejection sits at $p \approx 0.008$ — close to the
"interesting" range but not in the "decisive" range. The $p$-value
under the $\chi^2_{20}$ distribution at $Q = 38.25$ is approximately
$0.0083$. For multi-horizon-corrected interpretation: with Bonferroni
across the four reported horizons, threshold becomes $0.0125$, and the
$h = 20$ result still survives. With more aggressive (e.g. Holm)
correction across all horizons that *might* have been chosen, the
result becomes marginal.

Honest summary: the block-indicator process is *probably* not
i.i.d. — but the evidence is in the "moderate" range (Jeffreys'
"substantial," not "strong"), and the alternative being detected is a
weak ripple at an empirically interpretable cycle length, not a
dramatic Markov-style persistence.

## Markov-1 transition table: independence cannot be rejected

A different way to ask "is $B_t$ i.i.d.?" is to fit a Markov-1 model
on $\{0, 1\}$ and chi-square-test against the independence model. The
2×2 transition counts, with rows = previous state and columns = next
state, are:

|       | $\to 0$ | $\to 1$ | row sum |
|-------|--------|--------|---------|
| $0 \to$ | 847 | 40 | 887 |
| $1 \to$ | 40 | 4 | 44 |

The conditional probabilities are:

* $P(B_t = 0 \mid B_{t-1} = 0) = 847 / 887 = 0.95490$
* $P(B_t = 1 \mid B_{t-1} = 0) = 40 / 887 = 0.04510$
* $P(B_t = 0 \mid B_{t-1} = 1) = 40 / 44 = 0.90909$
* $P(B_t = 1 \mid B_{t-1} = 1) = 4 / 44 = 0.09091$

For comparison the marginal block rate is $\hat{p} = 0.04721$. So the
*persistence multiplier* is

$$\frac{P(B_t = 1 \mid B_{t-1} = 1)}{\hat{p}} \;=\; \frac{0.09091}{0.04721} \;=\; 1.9256\times.$$

Under independence we would expect $P(B_t = 1 \mid B_{t-1} = 1) =
\hat{p}$. Empirically it is roughly twice that. *In effect-size terms*
this is a genuine doubling. *In statistical-significance terms* it does
not survive: the χ² test of independence on the 2×2 table gives

$$\chi^2 \;=\; 1.954 \quad \text{on } 1 \text{ df},$$

with $\chi^2_{0.05, 1} = 3.841$, so we cannot reject independence at
the 5% level. The proximate reason is the tiny absolute count in the
$1 \to 1$ cell ($n = 4$) — Cohen's $h$ effect size between $P_{11}$
and the marginal is $h = 0.1745$, which is *below* the conventional
"small effect" threshold of $0.2$. The doubling is real, but on a base
rate so low that the doubling is statistically indistinguishable from
sampling noise on $n_{\text{block}} = 44$.

So: the Markov-1 persistence test fails to reject. The Ljung-Box at
$h = 20$ does reject. These are not in conflict — they have different
power profiles, and the data live in the regime where one test sees the
weak ripple and the other does not.

## Wald-Wolfowitz runs test on the binary sequence

Independent confirmation from a third angle. Treat the binary series as
a sequence of "runs" of consecutive identical values; under
i.i.d. Bernoulli with $n_0 = 888$ zeros and $n_1 = 44$ ones, the
expected number of runs is

$$\mathbb{E}[R] \;=\; \frac{2 n_0 n_1}{n_0 + n_1} + 1 \;=\; \frac{2 \cdot 888 \cdot 44}{932} + 1 \;=\; 84.845.$$

The variance is $\text{Var}[R] = (\mathbb{E}[R] - 1)(\mathbb{E}[R] - 2) /
(n - 1) = 7.459$, giving $\text{SD}[R] = 2.731$.

Empirically, the sequence has $R_{\text{obs}} = 81$ runs.

$$z \;=\; \frac{81 - 84.845}{2.731} \;=\; -1.408.$$

This is *under*-runs (block-ticks slightly more clustered than
i.i.d.), but at $z = -1.408$ the two-sided $p$-value is $\approx 0.16$
and we cannot reject. The sign of the result is consistent with the
weak positive lag-1 ACF ($r_1 = +0.046$): a slight positive
autocorrelation at lag 1 mechanically implies fewer runs than expected,
because adjacent values are slightly more likely to be the same. The
runs test essentially summarises lag-1 dependence into a single statistic
and reaches the same "directionally suggestive, not statistically
decisive" verdict that lag-1 ACF reaches alone.

## The hazard function with respect to clean-streak length

A further dimension of the i.i.d. test: under the null, the conditional
block hazard $P(B_t = 1 \mid B_{t-1} = B_{t-2} = \dots = B_{t-k} = 0)$
should equal the marginal $\hat{p}$ regardless of $k$. (Memorylessness
of the geometric.) Empirically:

| $k$ (prior consecutive clean ticks) | events $n_k$ | $\hat{P}(B_t = 1 \mid \text{prev } k \text{ clean})$ | ratio to $\hat{p}$ |
|---|---|---|---|
| 0 | 932 | 0.04721 | 1.000× |
| 1 | 887 | 0.04510 | 0.955× |
| 2 | 847 | 0.04604 | 0.975× |
| 3 | 808 | 0.04827 | 1.022× |
| 5 | 732 | 0.04781 | 1.013× |
| 10 | 581 | 0.04303 | 0.911× |
| 20 | 391 | 0.03069 | 0.650× |
| 40 | 218 | 0.03211 | 0.680× |

Two regimes are visible:

* For $k \le 10$, the conditional block hazard is essentially flat at
  the marginal rate (ratios in the band 0.91× to 1.02×). The
  memoryless null is consistent with the data.
* For $k \ge 20$, the conditional hazard drops sharply to $\approx
  0.65$–$0.68\times$ the marginal. After 20 consecutive clean ticks,
  the next-tick block probability is *lower* than the marginal.

The second observation is the kind of finding that should be interpreted
with care, because it is built on a *conditioning event* that is
correlated with calendar time. The 932-tick window starts at
2026-04-23 and ends at 2026-05-06; the long clean streaks are
concentrated in the older part of the window (when the daemon was
still being calibrated and many guardrail patterns were narrower). So
the apparent "memory" at $k = 20, 40$ is at least partly an artefact
of *non-stationarity in the marginal block rate over the window*, not
of true negative-feedback memory. A clean stationarity test (split the
window in halves and compare $\hat{p}$) would be the natural follow-up;
here we just flag it.

The flatness at $k = 0, \dots, 10$ is the more solid result and is
consistent with all the other evidence: the block process is
*almost* memoryless on short horizons.

## Where the block hazard concentrates: family stratification

Strip away the temporal lens entirely and look at *which families*
carry the block hazard. (Multi-counted: a tick of family
`templates+digest+metaposts` is counted once for each of the three.)

| family | $n_{\text{ticks with fam}}$ | $n_{\text{blocked}}$ | $P(\text{block} \mid \text{fam})$ | ratio to marginal |
|---|---|---|---|---|
| **templates** | 365 | **35** | **0.09589** | **2.031×** |
| metaposts | 376 | 20 | 0.05319 | 1.127× |
| digest | 396 | 20 | 0.05051 | 1.070× |
| cli-zoo | 399 | 16 | 0.04010 | 0.849× |
| reviews | 381 | 14 | 0.03675 | 0.778× |
| feature | 391 | 14 | 0.03581 | 0.758× |
| posts | 384 | 10 | 0.02604 | 0.552× |

The picture is sharp. The templates family is involved in 35 of 44
block-ticks — 79.5% of all block ticks contain templates as a
participant — and its conditional block rate of 0.0959 is **2.03× the
marginal** and **3.68× the lowest carrier (posts)**. This dominates the
block process. (Note: this concentration is consistent with prior
findings on block *magnitudes* — templates is also the magnitude
monopoly carrier, with three of the four magnitude-tail outliers
attributable to multi-block templates ticks. The two findings are
correlated by construction, but the *indicator-level* ratio of 2.03×
is a separately measured statistic.)

A relevant arithmetic check: under the (counterfactual) independence
null where each family's $P(\text{block} \mid \text{fam})$ equals the
marginal $\hat{p} = 0.04721$, the number of templates ticks with
$\text{blocks} > 0$ would be expected to be $365 \cdot 0.04721 = 17.23$.
We observe $35$. Treating $\{0, 1\}$ block-ticks within the templates
subset as Binomial$(365, 0.04721)$:

$$z \;=\; \frac{35 - 17.23}{\sqrt{365 \cdot 0.04721 \cdot 0.95279}} \;=\; \frac{17.77}{4.05} \;=\; +4.39.$$

Two-sided $p$-value under normal approximation: $\approx 1.1 \times
10^{-5}$. This *does* reject independence — emphatically — but in a
direction (cross-section, not time-series) that is orthogonal to the
Ljung-Box / Markov / runs analysis above.

The *time-series* lens of this post says: $B_t$ is *almost* i.i.d. in
$t$.

The *cross-section* lens says: $B_t$ is far from i.i.d. across
families.

Both can be true simultaneously, and the synthesis is interesting: the
block process is a Bernoulli-ish stream whose *parameter $p$* depends
strongly on which family-set is in the tick, but conditional on the
family-set the firings are roughly independent across time. This is
exactly the structure you would predict if guardrail trips are driven
by *content type* (templates ships shell scripts and `.env` fixtures
that the secret-pattern guardrail loves to flag) rather than by
*temporal state* (the daemon does not "get angry," does not "build up
pressure"). The mild lag-6 and lag-10 ripple in the time series is
plausibly a downstream consequence of the family-rotation cadence:
templates appears in the rotation at a frequency that lays a weak
$1/(\text{rotation period})$ harmonic over the indicator series.

## Block-arity stratification

For completeness, split by family-arity (number of families dispatched
in the tick):

| arity | $n_{\text{ticks}}$ | $n_{\text{blocked}}$ | $P(\text{block})$ | ratio |
|---|---|---|---|---|
| 1 | 33 | 1 | 0.03030 | 0.642× |
| 2 | 9 | 0 | 0.00000 | 0.000× |
| 3 | 890 | 43 | 0.04831 | 1.023× |

Arity-3 ticks (the standard parallel-dispatch shape) carry essentially
all the block events (43 of 44, 97.7%), because they are essentially
all the ticks (890 of 932, 95.5%). The arity-1 and arity-2 strata are
too small to support any useful comparison. Bottom line: arity is *not*
a useful discriminator of block hazard once you condition on it being
the dominant arity-3 mode.

## Block-count (not just indicator) lag-$k$ ACF

A final sanity check: redo the ACF analysis on the *count* series
$M_t = \text{blocks}_t \in \{0, 1, 2, 6, 14, 18\}$ rather than the
indicator $B_t = \mathbb{1}[M_t > 0]$. Lag-$k$ ACF for $k = 1, \dots, 5$:

| $k$ | $\hat{r}_k(M)$ |
|---|---|
| 1 | $+0.0036$ |
| 2 | $-0.0098$ |
| 3 | $-0.0113$ |
| 4 | $-0.0080$ |
| 5 | $+0.0207$ |

All five lags sit firmly inside the $\pm 1.96/\sqrt{N} = \pm 0.064$
band. The count series is even more strongly null-consistent than the
indicator series, which is initially counter-intuitive — *adding*
information (magnitudes) seems to make the autocorrelation *weaker*,
not stronger. The mechanism is straightforward: the four
high-magnitude block ticks (magnitudes 2, 6, 14, 18) are scattered in
calendar time (2026-05-01 20:15Z, 2026-05-02 04:25Z, 2026-05-04 00:46Z,
2026-05-04 18:33Z) and not adjacent to other block ticks, so their
contribution to lag-$k$ products is dominated by zero-multiplications
from neighbouring clean ticks. The variance of $M_t$ is also massively
inflated by these four ticks, which deflates the denominator of the
ACF. So the count series has *less* visible autocorrelation than the
indicator, even though it carries strictly more information. This is
again the right diagnostic for the previous observation that the count
process's interesting structure is in its *magnitude tail* (see prior
post), not in its *temporal correlation* — the two posts together
fence in the null model from opposite sides.

## What this means for the daemon

Three calibration take-aways, in decreasing order of confidence:

1. **The block-indicator series is well-approximated as i.i.d. for
   short-horizon planning.** Markov-1 fails to reject; runs fails to
   reject; lag-1 through lag-5 ACF all sit inside the noise band; the
   short-horizon clean-streak hazard is flat at the marginal. For
   purposes of "what is the probability the next tick will have a
   block?", the answer is just $\hat{p} = 0.0472$ regardless of what
   happened in the previous 1–10 ticks. This is the operational fact.

2. **Templates is where the block process actually lives.** A 2.03×
   marginal block rate, $z = +4.39$ in the cross-section test, 79.5%
   of block-ticks contain templates as a participant. If the daemon
   ever wants to *reduce* the block rate, the highest-leverage
   intervention is on templates' content type (shell scripts with
   `.env` fixtures) and not on temporal scheduling.

3. **There is a weak lag-6 / lag-10 ripple consistent with the
   family-rotation cadence.** The Ljung-Box $Q(20) = 38.25$ rejection
   is driven by these two suprathreshold lags, and the cycle length is
   compatible with the daemon's empirically observed 6-family
   round-robin. This is interesting but soft evidence; it would need a
   larger window to harden.

The split between the four tests in the headline table is the most
honest summary. The block-indicator process is **mostly i.i.d., but
not quite** — and the "not quite" is concentrated in two specific
lags whose positions are physically interpretable. This is the
quantitatively defensible version of "the daemon's guardrail trips
look memoryless, except for a faint pulse at the rotation period."

## Cross-reference table for the data anchors

For reproducibility, every numeric claim above is recomputable from:

* `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` lines 1–932
  inclusive, which span ticks `2026-04-23T16:09:28Z` through
  `2026-05-06T09:29:50Z`. The file count at write was 933 lines (one
  trailing partial line not parsed).
* The ten-PR-pair-and-six-repo-HEAD anchor table earlier in this post
  fixes the state of the codebase at the moment the analysis was
  computed.
* All test statistics ($Q$, $\chi^2$, $z$, $\hat{r}_k$,
  $P(B_t = 1 \mid \cdot)$, family-stratified hazard ratios) were
  produced by a single Python script reading `history.jsonl` with no
  external dependencies beyond `json`, `math`, and
  `collections.Counter`. No estimator uses bootstrap; all values are
  closed-form against the empirical 932-tick window.

The script's per-statistic outputs are reproduced exactly in the
tables above to four to six significant figures, with no rounding
beyond what the table presentation requires. The only number that has
been re-derived for this post and not directly printed by the script
is the $\chi^2_{20}$ tail probability at $Q = 38.25$, which is given
as $\approx 0.0083$ from a standard $\chi^2$ table (the script only
prints the critical values).

---

This is the eleventh `_meta` post in the 2026-05-06 daily series and
the second on the daemon's block process specifically (the first being
the magnitude-tail / clean-streak diptych). The two together pin down
the block process from three orthogonal directions: magnitude
(Pareto-tailed, templates-monopolised), gap distribution (overdispersed
geometric with streak-clustering), and indicator autocorrelation
(mostly i.i.d., with weak rotation-period ripple). What remains
unmeasured at the indicator-time-series level is the *response* of the
block process to changes in the daemon's content mix — a
covariate-augmented version of the analysis above where $\hat{p}_t$ is
allowed to depend on the family-set in tick $t$ — which is the natural
next post.
