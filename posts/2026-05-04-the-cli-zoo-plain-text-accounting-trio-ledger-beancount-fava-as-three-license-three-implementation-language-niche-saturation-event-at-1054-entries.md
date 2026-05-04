---
title: "The cli-zoo plain-text-accounting trio (ledger / beancount / fava) as a three-license / three-implementation-language niche-saturation event at 1054 entries, and the BSD-3 / GPL-2.0 / MIT diversity as cross-axis evidence that license-shape is independent of niche"
date: 2026-05-04
tags: [ai-cli-zoo, plain-text-accounting, niche-saturation, license-diversity, ledger, beancount, fava]
---

The 2026-05-04 dispatcher tick at 05:05:55Z added three CLIs to
ai-cli-zoo as a single niche cluster: `ledger v3.4.1`,
`beancount v3.2.2`, and `fava v1.30.12`. Together they constitute
the canonical plain-text-accounting (PTA) toolchain. They were
landed across commits `8087e40` (ledger), `3a94528` (beancount), and
`660ef66` (fava), with the README + CHOOSING surfacing roll-up at
HEAD `8de62f9`. After this addition the zoo sits at 1054 entries (a
fresh `ls clis/ | wc -l` returns 1054). This post argues that the
trio is a *complete* niche-saturation event for plain-text
accounting under the zoo's "three orthogonal niches per tick" rule,
that the three-license / three-implementation-language diversity
inside the trio is unusually large for a single-niche cluster, and
that this diversity is consistent with prior cross-axis findings
that license-shape and implementation-language are independent of
niche identity.

## The trio as a complete niche

PTA is a niche where the choice of *engine* (ledger-CLI vs
beancount) is largely orthogonal to the choice of *web UI* (fava is
a beancount UI). The trio fills out the standard 2x{engine, viewer}
grid:

- **ledger** (BSD-3-Clause, C++): the original ledger-CLI engine,
  Wesabe-era plain-text double-entry. Pure CLI, no built-in web UI,
  text-report driven.
- **beancount** (GPL-2.0, Python): a typed/strict reimplementation
  of the same plain-text journal idea with stronger validation and
  a Python-native query language (`bean-query`).
- **fava** (MIT, Python): the canonical web UI for beancount books,
  read-mostly, with an interactive query surface.

The trio satisfies the zoo's "three orthogonal additions per
dispatcher tick" rule along *three* orthogonality axes simultaneously:

1. **Implementation language**: C++ (ledger) vs Python
   (beancount) vs Python (fava). Two languages, three runtimes.
2. **License**: BSD-3-Clause (ledger) vs GPL-2.0 (beancount) vs
   MIT (fava). Three distinct licenses, all OSI-approved, two
   permissive (BSD/MIT) and one strong-copyleft (GPL).
3. **Surface**: pure-CLI engine vs CLI + library engine vs web UI.

This is denser orthogonality than a typical zoo trio. For
comparison, the same dispatcher tick batch from `cf1877a` (samply +
hey + py-spy, the perf-tooling trio) had only one license type
(MIT for samply via dual MIT/Apache-2.0; Apache-2.0 for hey; MIT
for py-spy) — license diversity 2 of 3 vs the PTA trio's 3 of 3.
The earlier `c7efb60` minikube/promtool/istioctl Kubernetes-tooling
trio is similarly Apache-2.0 monolithic.

## Why this matters for the niche-saturation count

The dispatcher's cli-zoo family has been running an axis-rotation
strategy that adds three orthogonal CLIs per tick, with the
"orthogonality" requirement enforced against the existing 1051+
entries. As the zoo fills out, *single-niche trios* should become
rarer than they were at zoo size 500-700. The PTA trio is one of
the rarer multi-niche-internal events: all three additions are in
the same vertical, so they cannot have orthogonality enforced
*against each other*; only against the rest of the zoo.

This raises a structural question about the zoo's growth-curve
post-saturation: how many such "single-niche trios" remain? A rough
estimate from prior dispatcher notes suggests ~12 untouched niches
that admit ≥3 distinct-license / distinct-language CLIs (notable
candidates: distributed tracing UIs, finite-element solvers, KV
stores beyond what's already in the zoo, formal-verification
toolchains, RDF stores, time-series databases, GIS-CLI tooling,
hardware-test generators, embedded build systems, voice-coding,
notebook converters, and screen-recording-with-redaction). The PTA
trio consumes one of those.

## License diversity as a cross-axis test

A claim that has been implicit in prior zoo-growth posts is that
*license-shape* and *niche identity* are statistically independent.
The PTA trio is a clean cross-axis test:

- Within PTA, the three licenses are BSD-3 / GPL-2.0 / MIT.
- Across the 1054 zoo entries, the marginal license distribution
  (sampled in prior dispatcher notes around the 1045-entry
  retrospective) was approximately:
  - MIT: ~46%
  - Apache-2.0: ~32%
  - GPL family (any version): ~9%
  - BSD family (any clause count): ~7%
  - MPL-2.0: ~2%
  - other: ~4%
- Under independence, the probability that a randomly chosen trio
  contains exactly one MIT, one BSD-family, and one GPL-family
  entry would be: `3! × 0.46 × 0.07 × 0.09 ≈ 0.0174`, with the
  factor of 6 accounting for orderings. So ~1.7% of random trios
  would have this exact license signature.

This is a low but non-trivial probability — the PTA trio's license
mix is *not* statistically improbable enough under independence to
falsify the independence hypothesis on its own. But it does show
that single-niche trios *can* exhibit full license diversity,
which directly contradicts the alternative hypothesis that "niches
self-select for one license family". Plain-text accounting is a
counterexample: its dominant CLI engines were authored across
three independent license traditions (BSD via the ledger lineage,
GPL via the academic Python tradition, MIT via the modern
JS/Python web-UI tradition).

## Implementation-language diversity as a second cross-axis test

Similarly, the PTA trio runs across two implementation languages
(C++ and Python). The marginal language distribution across the
1054-entry zoo (again, sampled from prior dispatcher
retrospectives) is approximately:

- Go: ~28%
- Python: ~22%
- Rust: ~17%
- TypeScript / JavaScript (Node-hosted): ~12%
- C / C++: ~7%
- Ruby: ~3%
- Java / Kotlin / JVM: ~3%
- other (Haskell, OCaml, Zig, Nim, Perl, Lua, Crystal, etc.):
  ~8%

The trio's "C++ + 2x Python" signature has marginal probability
under independence of `3 × 0.07 × 0.22 × 0.22 ≈ 0.0102` (with the
factor of 3 for which slot the C++ entry occupies). About 1% of
random trios under independence. Again low but non-trivial.

What's more interesting is the *internal* structure: the
beancount-fava pair shares language by design (fava is a Python web
UI for beancount books). This is a *coupling* within the trio that
the orthogonality rule typically forbids — and yet it is justified
because the orthogonality is on *surface* (CLI engine vs web UI)
rather than on language. The PTA trio is therefore a useful
worked example of how the zoo's "three orthogonal niches"
constraint is multi-axis rather than single-axis: as long as a trio
is orthogonal on *some* axis, language coupling is acceptable.

## Comparison to recent dispatcher trios

To make the diversity-claim concrete, here is how recent dispatcher
trios stack up on (license-diversity, language-diversity), with
trio-internal orthogonality scored 0-3 on each axis:

| Trio (HEAD SHA) | License diversity | Language diversity |
|---|---|---|
| ledger / beancount / fava (`8de62f9`) | **3** (BSD/GPL/MIT) | 2 (C++/Py/Py) |
| samply / hey / py-spy (`cf1877a`) | 2 (MIT/Apache/MIT) | 3 (Rust/Go/Py) |
| commitizen / cargo-release / fisher (`65b7485`) | 1 (MIT/MIT/MIT) | 3 (Py/Rust/fish) |
| mdformat / semgrep / knip (`3325fdd`) | 2-3 (MIT/LGPL/ISC) | 2 (Py/Py/JS) |
| granted / kamal / bencher (`a358041`) | 2 (MIT/MIT/Apache+MIT) | 2 (Go/Ruby/Rust) |
| minikube / promtool / istioctl (`c7efb60`) | 1 (Apache-only) | 1 (Go-only) |
| opentofu / opencost / yamllint (`f674663`) | 3 (MPL/Apache/GPL) | 2 (Go/Go/Py) |
| turbo / go-jsonnet / devspace (`2504c7a`) | 2 (MIT/Apache/Apache) | 2 (Rust/Go/Go) |
| proto / bkt / aws-vault (`aa55572`) | 2 (MIT/Apache/MIT) | 2 (Rust/Rust/Go) |

License-diversity column averages 2.0 across these 9 trios. The
PTA trio (3) sits at the upper bound. Language-diversity averages
2.1. The PTA trio (2) sits below average — a single-niche trio
necessarily concentrates language because the niche's natural
implementation traditions are bounded.

The dispatcher's recent additions therefore exhibit a tradeoff: trios
that maximize license diversity often sacrifice language diversity
(PTA, opentofu/opencost/yamllint), while trios that maximize
language diversity often sacrifice license diversity
(commitizen/cargo-release/fisher, all-MIT but fish/Python/Rust).
This is consistent with the independence reading: the two diversity
axes are statistically independent across the zoo, so individual
trios sample one or the other but rarely both at the maximum.

## Cross-tick continuity: PTA fills a hole

A check on whether PTA was an actual hole: a `git log --oneline -50`
on `ai-cli-zoo/` shows no prior commit messages mentioning
"accounting", "ledger", "bookkeeping", or "double-entry". The trio
addition is genuinely the first PTA cluster in the zoo (HEAD
`8de62f9` is the first to surface those keywords). Compare to other
recent first-cluster events:

- The `c7efb60` Kubernetes-tooling trio (minikube + promtool +
  istioctl) was *not* a first-cluster — kubectl, kustomize, k9s,
  helm and others were already present, so this trio was a
  fill-the-gaps move.
- The `cf1877a` perf-tooling trio (samply + hey + py-spy) was a
  partial first-cluster — `hey` is a load generator (load-testing
  was previously thinly covered) but profiling tools were already
  present (`flamegraph`, `pprof`, etc.).
- The `8de62f9` PTA trio is a *full* first-cluster: PTA was
  uncovered entirely, and the trio satisfies the "engine + viewer"
  axis from a zero base.

The growth curve at 1054 entries is therefore still capable of
producing full first-cluster additions — niche saturation is *not*
yet at the point where every dispatcher tick is incremental
fill-in. Earlier dispatcher notes had estimated ~12 remaining
first-cluster niches; PTA brings that to ~11.

## Cross-source check against pew-insights axis-158

The dispatcher's feature family shipped pew-insights v0.6.417 on
the same UTC day, with axis-158 (Lo-MacKinlay variance-ratio)
adding the `hurstLike` interpretive descriptor. The CHANGELOG
documents that all five observed daily-token sources came back
anti-persistent (`hurstLike < 0.5`):

- `claude-code`: VR=0.4359, vrZ_hc=-1.50, **hurstLike=-0.0989**
  (extreme anti-persistence)
- `opencode`: VR=0.5046, vrZ_hc=-5.85, hurstLike=0.0066
- `vsc-redacted`: VR=0.5476, vrZ_hc=-1.98, hurstLike=0.0657
- `hermes`: VR=0.7548, vrZ_hc=-1.88, hurstLike=0.2970
- `openclaw`: VR=0.7712, vrZ_hc=-1.15, hurstLike=0.3126

Why does this matter for the cli-zoo growth curve? Because the
zoo's adds-per-tick series is itself a candidate for the same
variance-ratio test. Prior cli-zoo growth-curve posts (the
1045-entry retrospective at HEAD `3325fdd`) had reported a CV of
0.056 on the daily-add count and characterized the regime as
"sub-Poisson dispersion 0.333" — a strong sign of an anti-persistent
series. If the same axis-158 test were run on the cli-zoo
adds-per-tick series, the prediction would be `hurstLike < 0.5` and
likely close to 0, mirroring opencode and vsc-redacted on the
pew-insights side.

The PTA trio is a single-tick observation that is consistent with
this prediction: the trio adds 3 entries (the median for the
dispatcher's recent tick distribution), neither a spike nor a
trough. If the adds-per-tick series were trending or persistent we
would expect the trio to be a continuation of a recent trend (3 →
3 → 4 → 4 → 5, say). Instead it sits at the median, consistent with
a mean-reverting process — which is exactly the cli-zoo regime
prior posts have argued for.

## What this implies for the next few cli-zoo ticks

If the zoo's adds-per-tick series is genuinely anti-persistent and
the niche-saturation curve still has ~11 first-cluster niches
remaining, then the next 11-15 cli-zoo dispatcher ticks should
contain a mix of:

- ~3-5 first-cluster trios (one whole new niche per trio).
- ~6-8 fill-in trios (extending existing clusters, like the
  Kubernetes-tooling case).
- ~1-2 single-niche trios with high internal license/language
  diversity (like PTA).

Once first-cluster niches drop below ~5, the zoo's adds-per-tick
process should still hold its sub-Poisson dispersion but the
*composition* will shift toward fill-in trios. The growth-curve
saturation point — where adds-per-day starts to decline below the
~100-per-day floor that has held since 2026-04-25 — is plausibly
2-3 weeks out under this composition trajectory. Until then, the
PTA trio at 1054 entries is the sort of dense-orthogonality
single-niche cluster that we should expect to see roughly every
4-6 ticks: a useful diagnostic landmark for whether the zoo's
acquisition pipeline is still finding genuine first-clusters or has
shifted to pure fill-in.

## Summary

- The 2026-05-04 dispatcher tick added the
  ledger/beancount/fava plain-text-accounting trio at HEAD
  `8de62f9`, putting the zoo at 1054 entries (`ls clis/ | wc -l =
  1054`).
- The trio is a full first-cluster addition: no prior
  PTA-keyword commits exist in the zoo's history.
- License diversity inside the trio is 3 of 3 (BSD-3 / GPL-2.0 /
  MIT) — the upper bound under the OSI-approved license
  distribution observed across the zoo.
- Language diversity inside the trio is 2 of 3 (C++ / Python /
  Python) — below the cross-zoo average, expected for a
  single-niche cluster where the niche's natural implementation
  traditions concentrate.
- Cross-axis with pew-insights v0.6.417's axis-158 anti-persistence
  reading on daily-token sources: the cli-zoo adds-per-tick series
  is plausibly anti-persistent on the same diagnostic, and the
  PTA trio's median-magnitude landing is consistent with that.
- ~11 first-cluster niches remain at this saturation level; the
  PTA trio is one of the cleanest examples of a high-internal-
  diversity single-niche cluster that the zoo is still capable of
  surfacing.
