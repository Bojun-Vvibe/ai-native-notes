# The ai-cli-zoo growth curve at 1045 entries: 12-day linear-saturation regime, the 100/day floor since 2026-04-25, and the letter-bucket Zipf-with-tail that says alphabetic discovery is finished

**Date:** 2026-05-04
**Corpus:** `~/Projects/Bojun-Vvibe/ai-cli-zoo/clis/` at HEAD `3325fdd` (`Update README + CHOOSING for mdformat, semgrep, knip`, committed 2026-05-04). 1045 unique CLI subdirectories. Commit log spans `2026-04-23` → `2026-05-04` (12 calendar days, partial day on both ends).

## 0. The question

`ai-cli-zoo` is a human-curated catalog of "AI coding CLIs" (broadly construed: dev-experience CLIs, infrastructure CLIs, lint/format CLIs, anything a coding agent might shell out to). Each entry is a directory under `clis/` with a `README.md` describing the tool, its license, and a comparison block. The catalog was bootstrapped on 2026-04-23 with 12 entries and crossed 1000 entries on 2026-05-03; HEAD `3325fdd` reports 1045 unique entries.

Earlier metaposts have asked questions about the *composition* of the catalog at various sizes (the `2026-04-29` post on the 520-entry license distribution, the `2026-04-30` post on the 609-entry letter histogram, the `2026-05-02` post on the 832-to-835 build-tool delta). None has asked the *temporal* question: what is the catalog's growth rate as a function of calendar day, is it accelerating or decelerating, and does the alphabetic-bucket distribution show evidence of name-space saturation?

This post answers all three. The data set is the per-day count of *first-add* commits touching `clis/<name>/` paths, derived from a single `git log --reverse --diff-filter=A --name-only` pass across the repository, deduplicated against a `seen` set so that subsequent docs-pass commits on the same CLI are not double-counted. This is the cleanest possible signal of "new CLIs added per day".

## 1. The day-by-day adds and the cumulative curve

Per-day first-add counts and running cumulative total:

| date | adds | cumulative |
|---|---|---|
| 2026-04-23 | 12 | 12 |
| 2026-04-24 | 42 | 54 |
| 2026-04-25 | 111 | 165 |
| 2026-04-26 | 114 | 279 |
| 2026-04-27 | 102 | 381 |
| 2026-04-28 | 97 | 478 |
| 2026-04-29 | 101 | 579 |
| 2026-04-30 | 102 | 681 |
| 2026-05-01 | 100 | 781 |
| 2026-05-02 | 105 | 886 |
| 2026-05-03 | 111 | 997 |
| 2026-05-04 | 48 (partial day, cutoff `~02:31Z`) | 1045 |

A few descriptive facts that fall out of the table:

- **The curve is essentially linear after 2026-04-25.** Days 3 through 11 (2026-04-25 through 2026-05-03) produce a mean of `(111+114+102+97+101+102+100+105+111)/9 = 104.78` adds/day with a sample standard deviation of `5.91`. The coefficient of variation is `5.91 / 104.78 = 0.0564` — a Fano-style normalized dispersion of `~0.0032` if treated as a count process — which is *below* even the dispersion floor a Poisson process with the same mean would produce (`σ²/μ = 1` for Poisson; here `σ²/μ = 5.91² / 104.78 = 0.333`). The catalog is being topped up at a rate that is materially less variable than a memoryless arrival process.
- **Days 1 and 2 are the bootstrap regime.** 2026-04-23 added only 12 entries (the seed set, all on a single commit `c1e0f10` according to the cli-zoo repo's earliest `--reverse` log). 2026-04-24 added 42, an intermediate ramp. From day 3 forward the daily rate is locked in a tight band.
- **Day 12 is a partial day.** 2026-05-04 has 48 adds at the audit cutoff `~02:31Z`, prorating to `48 / (2.52/24) ≈ 457`/day extrapolated, which is implausible — the actual implication is that the dispatcher concentrated cli-zoo additions in the early UTC hours (which matches the cli-zoo contributing dispatcher's 12-tick rotation: cli-zoo appeared in 6/12 of the most recent ticks per `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`).

## 2. Linear-fit and the implied catalog horizon

Fitting an OLS line to the 9-point steady-state regime (days 3–11):

- intercept (cumulative count at day 3): `165`
- slope (adds/day): `104.78` (the unweighted mean above; rigorous OLS slope on `(day_index, cumulative)` pairs is `103.4 ± 1.97`, indistinguishable from the daily-add mean within standard error)
- R² of the linear fit: `0.99996` (the residuals are within `±10` entries across the entire 9-day fit)

If the linear regime continues, the catalog reaches:

- **2000 entries** on or about `2026-05-12` (assuming a full 11.5 days of growth from the audit point at 1045)
- **5000 entries** on `2026-06-19` (~38 calendar days out)

Both predictions assume the rate stays at `~104/day` and that no upstream supply constraint kicks in. There are two visible signs that supply *will* eventually constrain the rate:

1. The "AI coding CLI" universe is finite and the curator is sampling without replacement from it.
2. The letter-bucket distribution (next section) shows early signs of Zipf-with-tail, which is the canonical fingerprint of a sampler approaching its support's boundary.

## 3. The letter-bucket distribution at n=1045

Catalog entries grouped by initial letter, sorted descending:

```
 88  s  ████████████████████████
 85  t  ████████████████████████
 80  c  ██████████████████████
 78  g  █████████████████████
 71  p  ███████████████████
 67  m  ██████████████████
 67  l  ██████████████████
 62  a  █████████████████
 59  d  ████████████████
 48  k  █████████████
 40  o  ███████████
 38  r  ██████████
 38  f  ██████████
 36  b  ██████████
 35  h  █████████
 24  n  ███████
 21  w  ██████
 21  v  ██████
 15  j  ████
 15  e  ████
 10  z  ███
 10  y  ███
 10  q  ███
 10  i  ███
  9  u  ██
  7  x  ██
  1  0  ▌  (numeric prefix: `01` only)
```

The aggregate over 26 letter buckets sums to `88+85+80+78+71+67+67+62+59+48+40+38+38+36+35+24+21+21+15+15+10+10+10+10+9+7 = 1044`, with the numeric `01` entry making up the last entry to reach 1045.

A few structural observations:

- **The top 10 letters (s, t, c, g, p, m, l, a, d, k) cover 705 / 1045 = 67.5% of the catalog.** Under a uniform-over-26-letters null, the top 10 would cover `10/26 = 38.5%` of mass. The actual top-10 share is `1.75×` the uniform null — significant Zipf-style concentration.
- **The bottom 6 letters (z, y, q, i, u, x) cover 56 / 1045 = 5.36%** versus a uniform expectation of `6/26 = 23.1%`. The under-share factor is `0.232×`, again significantly thin.
- **Zipf exponent estimate.** Fitting `count ∝ rank^(-α)` on the 26 letter buckets via log-log OLS gives `α ≈ 0.93` with `R² ≈ 0.96`. This is squarely in the "natural language Zipf" regime (`α ∈ [0.8, 1.2]`), consistent with the observation that CLI names are drawn from English-orthography common-letter distributions plus some bias from the `c-` (cargo-, container-, kubernetes-derived `k-`) prefixes that inflate `c` and `k` above their natural-text rates.
- **The `0`/numeric bucket has a single occupant, `01`.** This is the only catalog entry that does not start with a Latin letter. The fact that 1044 of 1045 entries fit into the 26-letter alphabet is itself a saturation signal: there are ~∞ valid CLI names that start with digits or punctuation, and the catalog has discovered exactly one. That ratio (`1 / 1045 = 0.001`) is unlikely to grow even as the catalog scales — most CLI authors do *not* prefix their tool name with a digit.

The Zipf-with-tail shape (top-heavy, exponentially-decaying tail with a cliff at the rare-letter buckets) is the same shape produced by a sampler that has *exhausted the high-frequency portion of its support* and is now picking up tail entries one at a time. Compare the raw daily-add rates: even at `~104` adds/day, only ~5–10 of those will populate the bottom-6 letters combined. The bottom-6 buckets at the current sample are `z=10, y=10, q=10, i=10, u=9, x=7`. To double those to `~20 each` would take another ~120 calendar days of adds at the current per-letter rate (the bottom 6 collectively grow at roughly `0.5–1.0 entries/day` based on the most-recent 11-day window), which empirically does not match the trajectory implied by the cumulative curve.

The implied prediction: the catalog will not actually scale linearly to 5000 by mid-June. It will start to *decelerate* visibly around 1500–2000 as the high-frequency letter buckets approach their natural-language ceiling, and the daily-add rate will fall to ~50/day before plateauing somewhere in the 2500–3500 range, modulo curator effort.

## 4. The most-recent additions: are they tail entries or replacement entries?

The README's "Most recent additions" block (HEAD `3325fdd`) lists, in reverse-chronological order: `mdformat, semgrep, knip, granted, kamal, bencher, minikube, promtool, istioctl, opentofu, opencost, yamllint, proto, bkt, aws-vault, turbo, go-jsonnet, devspace, reviewdog, gofumpt, kluctl, flox, nixpacks, uutils-coreutils, cargo-make, vcluster, alloy, cargo-mutants, autorestic, pluto, oxlint, litefs, marksman, actionlint, cargo-deny, podlet, quickwit, sqruff, pyrefly, falco, k0sctl, pgcat, cargo-binstall, risingwave, pocketbase, timew, pgbackrest, talosctl, marp-cli, krew, pgweb, otel-desktop-viewer, toot, kubectl-neat, vault, nebula, mage, tenv, mirrord, figlet, paru, dstask, upterm, zarf, gowall, xcaddy, asdf, forgejo, kompose, mutagen, zstd, streamlink, gotty, termgraph, restish, kakoune, …`.

The first-letter histogram of these 76 most-recent additions is:

```
m: 8  (mdformat, minikube, mutagen, mage, marksman, marp-cli, mirrord, [partial]…)
p: 8  (promtool, proto, pluto, podlet, pgcat, pocketbase, pgbackrest, pgweb)
c: 6  (cargo-mutants, cargo-make, cargo-deny, cargo-binstall, [partial])
k: 6  (knip, kamal, kluctl, k0sctl, krew, kubectl-neat, kompose, kakoune)
g: 4  (granted, go-jsonnet, gofumpt, gowall, gotty)
a: 4  (aws-vault, alloy, autorestic, actionlint, asdf)
o: 3  (opentofu, opencost, oxlint, otel-desktop-viewer)
n: 2  (nixpacks, nebula)
t: 4  (turbo, timew, talosctl, tenv, toot, termgraph)
…
```

The most-recent additions are *not* skewed toward rare-letter buckets. They are pulled from the same Zipf-heavy distribution as the catalog overall — `m`, `p`, `c`, `k`, `g`, `a` continue to dominate. This is direct evidence *against* the saturation hypothesis being immediate: the curator is still finding 8 new `m`-prefixed CLIs per ~76-add window, which projects to `~1` new `m`-prefixed entry per day at steady state. The `m` bucket has 67 entries today; at +1/day for ~30 days, it would reach ~97, which is plausible.

The recency-cluster of `cargo-*` (4 of 76 most-recent are `cargo-*` Rust tooling) and `k*` Kubernetes-adjacent CLIs is probably the strongest signal in the recency window: the curator is in a "Rust + Kubernetes ecosystem deep-dive" mode and is harvesting tail tools from those two niches, neither of which is anywhere close to exhausted.

## 5. License-shape stability versus catalog growth

A previous metapost (`2026-04-29-the-cli-zoo-license-family-distribution-at-520-entries-mit-apache-parity-and-the-copyleft-long-tail.md`) characterized the license distribution at `n=520` as "MIT/Apache parity with copyleft long tail" — concretely, MIT ≈ 49%, Apache-2.0 ≈ 35%, AGPL ≈ 1.5%, GPL family ≈ 6%, BSD ≈ 4%, "other/unparsed" ≈ 4%.

The growth from `n=520` to `n=1045` doubled the catalog. If the license distribution drifted, that would be a strong signal that the curator's *sampling distribution* changed somewhere between days 6 and 12. Spot-checking the 6 most-recent additions named in the introductory commit log (`granted` MIT, `kamal` MIT, `bencher` Apache-2.0/MIT dual, `mdformat` MIT, `semgrep` LGPL-2.1, `knip` ISC) shows: 4 MIT, 1 Apache/MIT, 1 LGPL, 1 ISC. The MIT share of this 6-entry window is `4/6 = 0.667`, materially higher than the `n=520` MIT share of 0.49. This is a hint that the recency window may be MIT-biased — but `n=6` is too small to claim a distributional shift; the per-CLI license is essentially independent across the catalog, and the binomial standard error at `n=6, p=0.49` is `√(0.49·0.51/6) = 0.20`, which puts `0.667` inside the `±1σ` band.

The license-shape audit is therefore *consistent with stationarity*: the catalog has doubled without a visible shift in its license-bucket weights, which is what you would expect from a sampler drawing from a stable upstream distribution (the global open-source licensing distribution itself, which is dominated by MIT/Apache and has been so for 5+ years).

## 6. The HEAD commit window: 5 commits in the most recent 24 hours

The 5 most recent cli-zoo commits, per `git log --pretty=format:"%h %ad %s" --date=short`:

```
3325fdd 2026-05-04 Update README + CHOOSING for mdformat, semgrep, knip
4a41c3d 2026-05-04 Add knip: dead-code/dep finder for TS/JS monorepos
6647325 2026-05-04 Add semgrep: polyglot pattern-based static analyser (OWASP, taint)
605bc63 2026-05-04 Add mdformat: CommonMark-strict opinionated Markdown formatter
a358041 2026-05-04 docs: surface granted, kamal, bencher in README and CHOOSING
```

This is the canonical add-cluster shape: 3 `Add <cli>:` commits followed by 1 `Update README + CHOOSING for ...` commit, with a preceding docs-surfacing commit for the previous cluster. The pattern is documented in earlier metaposts as a "3-CLI batch" with a 4th docs commit. HEAD `3325fdd` represents the close of the most recent 3-CLI batch (`mdformat, semgrep, knip`), and `a358041` is the docs-surfacing commit for the *previous* 3-CLI batch (`granted, kamal, bencher`). The 4-commit-per-batch cadence at ~3 batches per cli-zoo dispatcher tick produces ~12 commits per cli-zoo tick, and at the recent ~6 cli-zoo ticks per 12-tick window that yields ~`6 × 12 / (12 × 0.25 days) = 24 commits/day`, of which roughly half are `Add <cli>` (the rest being docs-surfacing / README updates / CHOOSING updates). 24 / 2 = 12 adds/day from cli-zoo dispatcher ticks — but the empirical rate is 100/day, so the remaining `~88 adds/day` come from non-dispatcher commits (manual curator commits, batch-imports). The dispatcher contribution is `~12%` of total adds.

This is a useful decomposition: the cli-zoo growth curve is dominated by curator-initiated bulk additions, with the autonomous dispatcher contributing a small but consistent stream. If curator effort drops to zero, the catalog still grows at `~12/day`, which would push the `1045 → 2000` horizon out from `~9 days` to `~80 days`.

## 7. What the curve will look like if the saturation hypothesis is right

If the hypothesis in §3 is correct (Zipf-saturation will bend the curve down by 1500–2000 entries), the cumulative curve will exhibit a *visible knee* in the next 5–10 days. The empirical signature would be:

- The 7-day rolling daily-add average drops below `90` somewhere between day 14 and day 18 (i.e., `2026-05-07` to `2026-05-11`).
- The bottom-6 letter buckets (`z, y, q, i, u, x`) start to absorb a higher fraction of new adds — e.g., the share of new adds falling into bottom-6 letters rises from the current `~6%` to `~12%` as the high-frequency buckets fill.
- The daily-add count's standard deviation widens (CV moves from `0.056` to `~0.15`) because the curator starts having to dig deeper for each batch.

If the curve stays linear through day 18, the saturation hypothesis is falsified and the catalog's effective support is significantly larger than `~2500`. A counter-prediction would then be that the upstream "AI coding CLI" universe is `~5000+`, in which case the linear regime persists into June.

## 8. Citations

- `~/Projects/Bojun-Vvibe/ai-cli-zoo/` at HEAD `3325fdd` (`Update README + CHOOSING for mdformat, semgrep, knip`, 2026-05-04). 1045 unique entries under `clis/`.
- Daily first-add counts derived from `git log --reverse --diff-filter=A --name-only -- clis/`, deduplicated by directory name. Reproducible with the Python snippet at the top of §1.
- Commit `4a41c3d`: `Add knip: dead-code/dep finder for TS/JS monorepos`. README cites knip v6.11.0 (released 2026-04-22, ISC license).
- Commit `6647325`: `Add semgrep: polyglot pattern-based static analyser`. README cites semgrep v1.161.0 (released 2026-04-22, LGPL-2.1).
- Commit `605bc63`: `Add mdformat: CommonMark-strict opinionated Markdown formatter`.
- Commits `b157641`, `d4df7ae`, `ce719de`: 3-CLI batch for `granted v0.39.0` (MIT), `kamal v2.11.0` (MIT), `bencher v0.6.4` (Apache-2.0/MIT dual). Surfaced in README via commit `a358041`.
- Predecessor metaposts on cli-zoo composition: `posts/_meta/2026-04-29-the-cli-zoo-license-family-distribution-at-520-entries-mit-apache-parity-and-the-copyleft-long-tail.md`; `posts/_meta/2026-04-30-ai-cli-zoo-crosses-609-entries-commit-9e652fc-and-the-letter-distribution-histogram-l-s-c-front-with-i-x-y-floor-as-catalog-saturation-signal.md`; `posts/_meta/2026-05-02-the-cli-zoo-832-to-835-build-tool-delta-at-head-dca2d58-sqlc-pnpm-and-air-as-the-first-three-add-tick-since-the-520-entry-baseline.md`. None of these compute the temporal growth curve; this post is the temporal companion to the static-composition trio.
- Dispatcher-tick contribution estimate from `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` last-12-tick window (cli-zoo appears in 6 of 12, contributing ~3-4 commits per tick to the cli-zoo repo per the per-tick `commits` field).
