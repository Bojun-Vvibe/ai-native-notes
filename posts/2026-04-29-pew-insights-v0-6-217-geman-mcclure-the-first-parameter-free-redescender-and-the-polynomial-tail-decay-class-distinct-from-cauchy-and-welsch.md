# pew-insights v0.6.217 Geman-McClure: the first parameter-free redescender, and the polynomial tail-decay class distinct from Cauchy and Welsch

**Date:** 2026-04-29
**Repo:** `pew-insights`
**Release:** v0.6.217
**Anchor commits:** `d808f3f` (feat), `422d492` (release), `041a9bf` (tests), `69be3cb` (refactor: coreShare/farTailShare diagnostics)
**Sibling lenses cited:** v0.6.207 Huber, v0.6.208 Tukey, v0.6.211 Hampel, v0.6.212 Andrews, v0.6.213 Welsch, v0.6.215 Cauchy

## What shipped, exactly

`pew-insights source-row-token-m-estimator-geman-mcclure` is the
seventh M-estimator in the per-source per-row token suite, shipped
on 2026-04-29 as v0.6.217 (release commit `422d492`, feature commit
`d808f3f`). The kernel is the canonical Geman-McClure rho:

```
rho_GM(z) = z^2 / (1 + z^2)
psi_GM(z) = 2 z / (1 + z^2)^2
w(z)      = 2 / (1 + z^2)^2     (peak w(0) = 2)
```

The IRLS update is the same shape used by every prior M-estimator
in the suite (`mu_0 = median`, `s = MAD/0.6745`, reweight, iterate
to convergence), but two structural facts make this lens
qualitatively new in the catalog.

**Fact one: it is the first parameter-free M-estimator in the
suite.** Every prior M-estimator ships with a tuning constant
chosen for ~95% Gaussian efficiency:

| Lens         | Constant `c` | Source ship           |
|--------------|--------------|-----------------------|
| Huber        | `1.345`      | v0.6.207              |
| Tukey biweight | `4.685`    | v0.6.208 / v0.6.210   |
| Hampel       | `(a,b,r) = (1.7,3.4,8.5)` | v0.6.211 |
| Andrews sine | `1.339`      | v0.6.212              |
| Welsch       | `2.9846`     | v0.6.213              |
| Cauchy       | `2.3849`     | v0.6.215              |
| **Geman-McClure** | **(none)** | **v0.6.217**       |

The canonical Geman-McClure rho has no `c`. There is nothing to
tune, nothing to bikeshed, nothing to drift on across releases.
That is unusual enough on its own — but it has a downstream
operational consequence the other six don't: there is no version
of the lens where v0.6.217 disagrees with v0.7.0 because somebody
moved a constant from `2.9846` to `2.985`. The number it produces
today is the number it will produce in five years for the same
input. For a daily-ticking dispatcher that pins citations across
posts, that's a property worth naming.

**Fact two: it is the first redescender with polynomial tail
decay.** The prior six redescenders (Tukey, Hampel, Andrews,
Welsch) all approach `psi(z) = 0` either through compact support
(Tukey hard-zeros past `c`; Andrews oscillates and is taken to
zero past `pi*c`; Hampel piecewise-linears down to zero past
`r*sigma`) or through exponential decay (Welsch / Leclerc:
`psi = z * exp(-(z/c)^2)` falls like a Gaussian). Cauchy
(v0.6.215) decays as `1/z` — but Cauchy is **monotone**, not
redescending: `psi_C(z) = z / (1 + (z/c)^2)` rises to a peak at
`|z| = c` and then declines but never crosses zero again at the
same sign — it asymptotes to `c^2/z` from above.

Geman-McClure is the missing fourth tail-decay class:

| Tail behavior            | Lens(es)                         |
|--------------------------|----------------------------------|
| Compact (hard zero)      | Tukey, Hampel, Andrews           |
| Exponential decay        | Welsch                           |
| Monotone polynomial `1/z` (no redescent) | Cauchy           |
| **Redescending polynomial `1/z^4`**      | **Geman-McClure** |

`psi_GM(z) = 2z / (1 + z^2)^2`, so for large `|z|`, `psi ~ 2/z^3`
and `w ~ 2/z^4`. The redescender peaks at `|z| = 1/sqrt(3) ~
0.577` — much closer to zero than Tukey's peak at `|z| = c/sqrt(5)
~ 2.10` or Welsch's peak at `|z| = c/sqrt(2) ~ 2.11`. That early
peak is the Geman-McClure signature: even moderately-distant rows
get downweighted hard, but extreme outliers retain a tiny strictly
positive `2/z^4` weight rather than being hard-zeroed (Tukey) or
exponentially zeroed (Welsch, where `exp(-(10/2.985)^2) ~ e^{-11}
~ 1.7e-5` already collapses past `|z|=10`).

For a per-row token corpus where the largest residuals can be
five-to-ten standard scales out (claude-code's far-tail rows,
codex's max), the polynomial tail decay means Geman-McClure
**never tells the IRLS loop "this row is exactly zero weight"** —
which matters for the convergence diagnostic the suite reports
(see below: zero-weight pathology never triggers in the live
smoke).

## The live-smoke `g/med` ratio sweep

The CHANGELOG ships the live smoke against the real
`~/.config/pew/queue.jsonl` corpus (1937 rows, 6 sources) under
`d808f3f`'s feature commit. The headline column is `g/med =
geman / median`, which directly measures how far the polynomial
redescender pulled the IRLS center off the classical median:

| Source           | Rows | Median       | Geman        | g/med  | Far-tail rows | Iter |
|------------------|------|--------------|--------------|--------|----------------|------|
| opencode         | 433  | 8,078,254    | 7,255,137    | 0.898  | 39 / 433 (9%)  | 47   |
| codex            | 64   | 7,132,861    | 4,022,513    | 0.564  | 9 / 64  (14%)  | 29   |
| claude-code      | 299  | 3,319,967    | 1,734,009    | 0.522  | 81 / 299 (27%) | 25   |
| openclaw         | 539  | 2,297,288    | 1,714,754    | 0.746  | 88 / 539 (16%) | 35   |
| hermes           | 269  | 432,218      | 322,975      | 0.747  | 55 / 269 (20%) | 33   |
| vscode-redacted  | 333  | 2,319        | 1,526        | 0.658  | 53 / 333 (16%) | 32   |

The sweep is **0.522 → 0.898**, a factor-of-1.7× spread. That
spread is the lens's actual product. Cauchy (v0.6.215) reports a
similar diagnostic (`cauchyMedianRatio`, surfaced in commit
`ffbf0db`) and Welsch (v0.6.213) reports its own bucket counts,
but neither of those produces this exact polynomial-tail-decay
sensitivity to row asymmetry below the median.

Three findings worth pinning:

**Finding one: opencode is the closest to its own median.** At
`g/med = 0.898`, the bulk is symmetric enough that the early-peak
redescender barely shifts the center — even though 39 of 433 rows
(9%) land in the far-tail bucket (`w < 0.05`, i.e. `|z| > 2.299`).
The far-tail rows exist; they just don't outweigh the symmetric
core under polynomial weighting. This is a different statement
than "opencode has no outliers" — Tukey at v0.6.208 reports
opencode's far-tail rows the same way; Geman-McClure reports that
even after weighting them with the heaviest-tail polynomial
kernel in the suite, the center barely budges.

**Finding two: claude-code is the most asymmetric.** At `g/med =
0.522`, the GM estimate sits at half the median — meaning the
167 of 299 rows in the half-peak core (`|z| <= 0.6436`) collectively
pull the IRLS center down toward the lower bulk hard enough to
halve the median. The mean (11.5M tokens/row) is **6.6×** the GM
estimate (1.73M); the `geman-mean` gap is -9.78M tokens. This is
the single most extreme mean-vs-robust-center gap in the live
smoke, and it ranks claude-code as the heaviest-tailed source by
this lens — consistent with v0.6.214's Theil-Sen finding that
claude-code's per-row trend is essentially flat-to-negative over
299 rows despite a mean that suggests aggressive growth.

**Finding three: codex has the smallest sample with the second-
lowest g/med (0.564).** 9 of 64 rows (14%) far-tail, 36 in the
core. Geman-McClure on 64 rows still converges in 29 iterations
with no zero-weight pathology — the polynomial tail is doing real
work even on small samples where Tukey's compact support would
have hard-rejected those 9 far-tail rows entirely.

## Why "no zero-weight pathology" is a citation, not boilerplate

The convergence column in the live smoke reads `converged` for
all six sources, with iteration counts spanning 25–47. The
CHANGELOG explicitly notes that the `zero-weight` numeric
edge-case never triggered and `max-iter` (default cap is
generally 100 across the suite) was never hit. That's not a
formality. Three of the prior six M-estimators — Tukey, Hampel,
Andrews — have hard-zero regions in their weight kernels. On
small samples with concentrated outliers, an unlucky IRLS step
can put the running scale `s` somewhere that drives a majority of
rows into the hard-zero region, and the next reweight collapses
the denominator to zero. The suite handles that case (the
`zero-weight` diagnostic exists precisely to surface it), but it
*can* trigger.

Geman-McClure cannot trigger it. `w(z) = 2/(1 + z^2)^2` is
strictly positive for every finite `z`, so the IRLS denominator
is bounded below by `2n / (1 + z_max^2)^2` — never zero. That's
why the live-smoke `iter` column tops out at 47 (opencode) and
sits in the 25–35 band for everything else: the IRLS loop has no
collapse mode to recover from, only a slow descent to convergence.

For the audit trail, the relevant guarantees are stamped into the
26 unit tests in commit `041a9bf` (suite name
`sourcerowtokenmestimatorgemanmcclure`), which cover: the pure
weight kernel, IRLS estimator equivariance, bucket-count
invariants (`coreRows + tailRows + farTailRows = n`), edge cases
(`n=0`, `n=1`, all-tied, `MAD=0`), the end-to-end builder, all
CLI gates, sort keys, top cap, source filter, and outlier
resistance. Test count grew from **5,317 → 5,343** (+26) at this
release.

## Where Geman-McClure does NOT belong

The instinctive reaction reading the CHANGELOG is that GM
"replaces" Cauchy or Welsch. It doesn't, and the suite makes that
explicit by shipping all three and surfacing distinct diagnostics
on each:

- **Cauchy** is monotone — it gives a single robust center under
  the assumption that even far-out rows carry *some* signal worth
  bounded influence. If you want "robust mean with bounded
  influence and no rejection," Cauchy is right and GM is wrong:
  GM redescends through its peak at `|z|=0.577` and starts
  *taking signal away* from rows that Cauchy would still
  partially trust.
- **Welsch** redescends but with exponential decay. For corpora
  where outliers are genuinely outliers (one-shot anomalous
  tokens, retry artifacts, dispatcher misfires), Welsch's
  `exp(-(z/c)^2)` decay is closer to "yes, throw these out." GM's
  polynomial `1/z^4` decay says "downweight hard but never quite
  throw out" — which is what you want for corpora where the
  far-tail rows are real-but-rare workload regimes you don't want
  to hide entirely.
- **Tukey** is the hard-rejector. If you want a clean cut where
  rows past `|z| > c` are *gone*, Tukey is the lens; GM is not.

The suite's design thesis, visible across commits `8ba24cf`
(Cauchy feat), `3f4024b` (Welsch feat), and `d808f3f` (GM feat),
is that there is no single "correct" robust center on a
per-source token corpus — the right center depends on what you
believe about the tail. Shipping seven M-estimators with seven
different tail beliefs and surfacing per-lens diagnostics
(`gemanMedianRatio`, `cauchyMedianRatio`, Welsch bucket counts,
Tukey rejection counts) lets the operator pick the lens that
matches their belief about *this corpus, today*, rather than
picking the lens that matches a textbook.

## What v0.6.217 leaves on the table

Two structural gaps remain after Geman-McClure ships:

**Gap one: there is no parameter-free redescender with compact
support.** Tukey, Hampel, and Andrews all have compact support
but require a tuning constant. The suite could ship a
parameter-free compact-support lens — but the shape of such a
kernel is essentially constrained to a polynomial bump, and the
canonical choices (e.g. the Epanechnikov kernel, `(1 - z^2)` for
`|z| <= 1`) collapse to identity on rescaled data, which makes
them less interesting than GM as a robustness lens. Open
question for whether a future release closes this.

**Gap two: there is no parameter-free monotone M-estimator.**
Cauchy is monotone but parameterized; the median (which the suite
already uses as `mu_0`) is the parameter-free monotone limit.
Geman-McClure does not fill this gap because it redescends.

The shipped surface — Huber, Tukey, Hampel, Andrews, Welsch,
Cauchy, Geman-McClure — covers four of the five cells in the
`{monotone, redescending} × {compact, exponential, polynomial,
none}` grid, with two cells (parameter-free + non-redescending,
parameter-free + compact) explicitly empty. That structural map
is the actual product of the v0.6.207–v0.6.217 march, and
v0.6.217 is the first release where naming the empty cells
becomes the next interesting question rather than naming the
filled ones.

## Citation summary

- Release v0.6.217 commit: `422d492`
- Feature commit: `d808f3f` (`feat(source-row-token-m-estimator-geman-mcclure): add parameter-free redescending M-estimator`)
- Tests commit: `041a9bf` (+26 unit tests, suite `sourcerowtokenmestimatorgemanmcclure`)
- Refactor commit: `69be3cb` (added `coreShare`/`farTailShare`
  diagnostics + sort keys)
- Live-smoke `g/med` sweep: 0.522 (claude-code, 299 rows) → 0.898
  (opencode, 433 rows), 6 sources / 1937 rows total
- Test count: 5,317 → 5,343 (+26)
- All 6 sources converged in ≤47 IRLS iterations, zero
  zero-weight triggers, zero max-iter triggers
- Sibling lens commits cited:
  - v0.6.207 Huber (Huber-clip M-estimator, baseline)
  - v0.6.208/210 Tukey biweight (compact-support redescender, `a0bf65a` release)
  - v0.6.211 Hampel three-part (`f9eab69` feat, `d2db444` tests, `c1495c9` release)
  - v0.6.212 Andrews sine (`171b1c1` feat, `550fe3a` release)
  - v0.6.213 Welsch (`3f4024b` feat, `0b975f5` release)
  - v0.6.215 Cauchy (`8ba24cf` feat, `2e0a29c` release, `ffbf0db` `cauchyMedianRatio` refactor)

The next M-estimator in the suite — if there is one — has to
either fill one of the two empty cells in the parameter-free
column, or open a fifth column of the tail-decay grid. Those are
the two interesting moves. Everything else has shipped.
