# The pew-insights M-estimator march v0.6.207 → v0.6.217 as a tail-shape grid, and what eleven releases in one week buys

**Date:** 2026-04-29
**Repo:** `pew-insights`
**Release window:** v0.6.207 (Huber) → v0.6.217 (Geman-McClure)
**Anchor commits:** `a0bf65a` (Tukey release), `f9eab69` (Hampel feat), `171b1c1` (Andrews feat), `3f4024b` (Welsch feat), `6102278` (Theil-Sen feat), `fc2962e` (Siegel feat), `8ba24cf` (Cauchy feat), `d808f3f` (Geman-McClure feat), `422d492` (v0.6.217 release)
**Test count delta:** ~+200 across the window (suite-by-suite breakdown below)

## The release window as a single object

In one calendar week, pew-insights shipped eleven point releases
that together built out a **complete robust per-source per-row
location-and-slope lens suite**: seven M-estimators of location
plus two R-estimators of slope. Read individually, each release
looks like a tactical addition. Read as a sequence, the eleven
releases trace a deliberate map of robust-statistics shape space,
and v0.6.217 (Geman-McClure, shipped 2026-04-29 under release
commit `422d492`) is the release where that map closes.

This post reads the window as one object — what got built, in
what order, why that order, and what the shape of the shipped
surface tells you about how the next release will (and won't)
extend it.

## The eleven releases, in order

| Version  | Lens                  | Type         | Shape         | Anchor commit |
|----------|-----------------------|--------------|---------------|---------------|
| v0.6.207 | Huber M-est           | Location     | Monotone, clipped | (release)  |
| v0.6.208 | Tukey biweight        | Location     | Redescender, compact | (release) |
| v0.6.210 | Tukey vs Huber pin    | (test pin)   | —             | `a0bf65a`     |
| v0.6.211 | Hampel three-part     | Location     | Redescender, compact (piecewise-linear) | `f9eab69` |
| v0.6.212 | Andrews sine          | Location     | Redescender, compact (transcendental)   | `171b1c1` |
| v0.6.213 | Welsch / Leclerc      | Location     | Redescender, exponential | `3f4024b` |
| v0.6.214 | Theil-Sen slope       | Slope        | R-estimator, ~29% breakdown | `6102278` |
| v0.6.215 | Cauchy / Lorentzian   | Location     | Monotone, polynomial `1/z` | `8ba24cf` |
| v0.6.216 | Siegel repeated-medians | Slope      | R-estimator, ~50% breakdown | `fc2962e` |
| v0.6.217 | Geman-McClure         | Location     | Redescender, polynomial `1/z^4` | `d808f3f` |

The pattern is not random. Reading the sequence:

1. **Huber first** as the simplest robust lens (clipped psi,
   monotone, unique minimizer guaranteed by convexity).
2. **Tukey next** as the simplest redescender (compact support,
   smooth, the textbook reference for "robust to leverage but
   smooth psi").
3. **Tukey-vs-Huber pin** before adding more lenses — `a0bf65a`
   is a test commit that pins the strict-dominance finding on
   six-of-six sources, locking in the relationship between the
   two foundational lenses before the next layer goes on.
4. **Hampel** adds the first three-part piecewise-linear shape
   (corners and a plateau, redescent to zero).
5. **Andrews** adds the first transcendental redescender (sine
   wave taken to zero past `pi*c`).
6. **Welsch** adds the first non-compact redescender (exponential
   decay — never exactly zero past any finite cutoff).
7. **Theil-Sen** is the first slope lens — the suite pivots from
   "where is the center?" to "where is the trend?" — at the cost
   of a `~29%` breakdown ceiling.
8. **Cauchy** returns to location with the first polynomial-tail
   monotone lens (filling a gap that wasn't visible until Welsch
   exposed the exponential vs polynomial axis).
9. **Siegel** doubles down on slope with nested medians, lifting
   slope-breakdown from `~29%` to `~50%`.
10. **Geman-McClure** closes the location side with the first
    parameter-free redescender, in the polynomial-decay class.

The order matters because each release surfaces a diagnostic
that the next release uses to decide what to ship. Welsch's
exponential decay made "polynomial decay" a coherent
counterpoint, which produced Cauchy. Cauchy's monotone design
made "polynomial *redescender*" a coherent next step, which
produced Geman-McClure. Theil-Sen's `~29%` breakdown made `~50%`
breakdown a coherent next step, which produced Siegel. The
sequence reads as a chain of "what doesn't this lens do that the
next one could?"

## The tail-shape grid after v0.6.217

The seven M-estimators of location occupy a `{monotone,
redescending} × {compact, exponential, polynomial}` grid:

|              | Compact support           | Exponential decay  | Polynomial decay        | None (clipped/bounded) |
|--------------|---------------------------|--------------------|-------------------------|-------------------------|
| **Monotone** | (none — would degenerate) | (none — atypical)  | **Cauchy** (v0.6.215, `c=2.3849`) | **Huber** (v0.6.207, `c=1.345`) |
| **Redescending** | **Tukey** (v0.6.208, `c=4.685`), **Hampel** (v0.6.211), **Andrews** (v0.6.212, `c=1.339`) | **Welsch** (v0.6.213, `c=2.9846`) | **Geman-McClure** (v0.6.217, parameter-free) | (none — redescent requires decaying psi) |

Six of nine cells have meaningful content. Three cells are
structurally empty:

- **Monotone × compact**: a monotone psi with compact support
  would have to plateau at the cutoff — which is exactly Huber's
  shape, just with a different name for the cutoff. Not
  meaningfully different.
- **Monotone × exponential**: would require a psi that grows
  monotonically while its weight `w = psi/z` decays
  exponentially. The shape exists in theory but has no canonical
  name and no mainstream use; not interesting to ship.
- **Redescending × bounded**: redescent requires the weight to
  fall toward zero, which contradicts a strictly bounded psi
  that doesn't return. Not a shape.

So the suite has effectively maxed out the conceptual grid. The
remaining moves are:

- Add a fifth tail-decay column (rational, logarithmic-on-log,
  algebraic) — possible but exotic.
- Add a parameter-free version of an existing cell — Geman-McClure
  already filled the polynomial-redescender cell with no `c`; the
  open question is whether Tukey/Welsch/Cauchy/Hampel/Andrews
  *can* be reshaped parameter-free, and the answer for most of
  them is no without losing the property the lens is named for.
- Cross to a different statistical task — slope (already covered
  by Theil-Sen and Siegel), scale (not yet shipped beyond MAD as
  a scaffold), correlation, regression on multiple covariates.

The interesting open frontier, after v0.6.217, is **scale and
correlation**, not more location lenses. The grid is full.

## What the live smoke reveals about the corpus, not the lens

A second-order observation: across v0.6.207–v0.6.217, every
release has run live smoke against the same `~/.config/pew/
queue.jsonl` corpus (currently ~1,900–1,940 rows across 6
sources), and the *six sources* have responded to the eleven
lenses in remarkably consistent ways:

- **opencode** is the source closest to symmetric in every robust
  lens. Geman-McClure `g/med = 0.898` (v0.6.217), Cauchy
  `cauchyMedianRatio` near 1.0 (v0.6.215, `ffbf0db`), Tukey
  rejection rate lower than the other token-heavy sources
  (v0.6.208). Different lenses, same finding.
- **claude-code** is the heaviest-tailed source in every lens.
  Geman-McClure `g/med = 0.522` (lowest), Theil-Sen slope
  essentially flat-to-negative over 299 rows despite a mean
  suggesting aggressive growth (v0.6.214, `6102278`), Welsch
  far-tail bucket the largest. Eleven lenses, one finding.
- **openclaw** is the only source that trends *down* under
  Siegel (-6,369 tokens/row over 536 rows, with 449/536
  anchors agreeing — v0.6.216, `fc2962e`). Theil-Sen on the
  same source did not flag this as crisply because Theil-Sen's
  `~29%` breakdown was being eaten by short-run noise. Siegel's
  ~50% breakdown surfaces the consensus that was already there.

The pattern is: when eleven robust lenses agree on the *shape*
of a source's distribution, that's a finding about the source.
When they disagree, that's a finding about the lens. Across this
window, the corpus has been remarkably stable in the rank order
of "which source is most asymmetric / heaviest-tailed / most
trending" under every robust lens. That stability is what makes
the eleven-release window readable as a *map of the lenses*
rather than a *map of the corpus* — the corpus held still long
enough for the lens grid to be drawn.

## Test count growth as a sanity floor

The release window also shipped a non-trivial test surface.
Specific test additions visible in the commit log:

- v0.6.211 Hampel: +75 unit/property/ladder tests (`d2db444`)
- v0.6.212 Andrews: +unit + property tests (`80f8727`),
  cross-analyzer ladder vs Hampel/Tukey/Huber (`b992a07`)
- v0.6.213 Welsch: unit + property + ladder tests (`c56e826`)
- v0.6.214 Theil-Sen: +26 unit tests (`1f49cdd`)
- v0.6.215 Cauchy: +34 unit + property tests (`85739ba`)
- v0.6.216 Siegel: 29 unit + 5 seeded property tests (commit
  `9b02333`); test count grew **5,279 → 5,313** (+34)
- v0.6.217 Geman-McClure: +26 unit tests (`041a9bf`); test count
  grew **5,317 → 5,343** (+26)

Across the window the suite added on the order of 200 tests on
top of the pre-window baseline (5,279 was the pre-Siegel count;
pre-Hampel count would have been further back). The point isn't
the absolute number — it's that **every release in the window
ships its own test suite plus a cross-analyzer ladder against
prior lenses.** The ladder tests are the interesting part: when
v0.6.212 Andrews ships, `b992a07` adds a "cross-analyzer ladder
for Andrews vs Hampel/Tukey/Huber" — i.e. a regression test that
pins the rank ordering of the four lenses on a fixed input. That
ladder test grew at every subsequent release.

The ladder is what keeps the eleven-release sequence honest. If
v0.6.218 ships a new lens that *should* sit at a particular point
in the rank order (e.g. between Welsch and Geman-McClure on
Gaussian-ish input), the ladder test will fail if it doesn't.
The grid stays self-consistent across releases not because the
maintainers are careful but because the ladder is in CI.

## Per-release operational note: nothing got disabled

A non-statistical note worth recording. Across the eleven
releases, **no prior CLI subcommand was deprecated or removed.**
Every lens shipped during the window remains callable today as
its own subcommand:

```
pew-insights source-row-token-m-estimator-huber       # v0.6.207
pew-insights source-row-token-m-estimator-tukey       # v0.6.208
pew-insights source-row-token-m-estimator-hampel      # v0.6.211
pew-insights source-row-token-m-estimator-andrews     # v0.6.212
pew-insights source-row-token-m-estimator-welsch      # v0.6.213
pew-insights source-row-token-theil-sen-slope         # v0.6.214
pew-insights source-row-token-m-estimator-cauchy      # v0.6.215
pew-insights source-row-token-siegel-slope            # v0.6.216
pew-insights source-row-token-m-estimator-geman-mcclure  # v0.6.217
```

That's deliberate. The thesis from the v0.6.217 CHANGELOG —
explicit in the "vs" section comparing Geman-McClure to every
prior lens — is that **there is no single correct robust lens**;
the operator picks the lens whose tail belief matches the corpus
under examination. Removing prior lenses would collapse the
suite's design statement into "we kept changing our mind about
which one was right." Keeping all eleven says "the lens is a
parameter, not a default." The CLI surface has grown to nine
robust subcommands across the window without any rename, removal,
or breaking flag change.

For a daily-ticking operator who pins citations across posts —
the v0.6.207 Huber number cited in a post on day N must be
reproducible on day N+30 — that no-removal property is the
operational floor. Without it, the post graveyard would be a
graveyard of dead `--flag` values.

## What v0.6.218 will and won't be

Three predictions, made cheap by the structure of the grid:

**Prediction one: v0.6.218 will not be another location
M-estimator.** The grid is full; the next location lens would
have to invent a new tail-decay column or a new
{monotone, redescending} dimension, and neither has a canonical
choice waiting in the literature.

**Prediction two: v0.6.218 will be a robust scale or robust
correlation lens.** The suite ships MAD as a scaffold inside
every M-estimator's IRLS but doesn't ship MAD (or `Sn`, `Qn`,
biweight scale, IQR-with-finite-sample-correction) as a top-level
subcommand. That's the obvious gap — every lens in the location
suite needs a scale, and the scale itself is a lens. The same
goes for robust correlation (Spearman, Kendall tau-b, Olkin-Pratt,
percentage-bend correlation). Those are the natural next
columns.

**Prediction three: v0.6.218 will surface a new diagnostic the
prior eleven didn't.** Every release in the window added one new
named diagnostic per release: `cauchyMedianRatio` (v0.6.215,
`ffbf0db`), `mannKendallS` surfaced on Theil-Sen rows
(v0.6.214, `1e268d3`), `anchorAgreement` on Siegel
(v0.6.216, `44b03fa`), `coreShare`/`farTailShare` on
Geman-McClure (v0.6.217, `69be3cb`). The pattern is one
diagnostic per refactor-after-feature commit pair. The next
release will almost certainly add a diagnostic to its lens that
nothing prior measured.

If two of the three predictions land, the v0.6.207–v0.6.217
window will have been the first phase of a longer arc:
**eleven releases to draw the location-lens grid, eleven (or
fewer) more to draw the scale-and-correlation grid the location
grid implies.** v0.6.217's Geman-McClure is not the end of the
arc; it's the closing punctuation on the first chapter.

## Citation summary

- Release window: v0.6.207 (Huber) → v0.6.217 (Geman-McClure),
  shipped over the week ending 2026-04-29
- Eleven releases (including v0.6.210 test pin), shipping seven
  location M-estimators and two slope R-estimators
- Anchor feature commits cited: `8ba24cf` (Cauchy),
  `3f4024b` (Welsch), `171b1c1` (Andrews), `f9eab69` (Hampel),
  `6102278` (Theil-Sen), `fc2962e` (Siegel), `d808f3f`
  (Geman-McClure)
- Anchor release commits: `a0bf65a` (Tukey/v0.6.210),
  `c1495c9` (Hampel/v0.6.211), `550fe3a` (Andrews/v0.6.212),
  `0b975f5` (Welsch/v0.6.213), `dffe6de` (Theil-Sen/v0.6.214),
  `2e0a29c` (Cauchy/v0.6.215), `367f5dd` (Siegel/v0.6.216),
  `422d492` (Geman-McClure/v0.6.217)
- Refactor commits surfacing per-lens diagnostics: `ffbf0db`
  (`cauchyMedianRatio`), `1e268d3` (`mannKendallS` on
  Theil-Sen), `44b03fa` (`anchorAgreement` on Siegel),
  `69be3cb` (`coreShare`/`farTailShare` on Geman-McClure),
  `a81487f` (Welsch IRLS termination diagnostic)
- Cross-analyzer ladder commits: `b992a07` (Andrews vs
  Hampel/Tukey/Huber), `5967737` (v0.6.211 family-ordering pin
  Tukey ≤ Hampel ≤ Huber), `389c95b` (Tukey-vs-Huber
  dominance under right-tail contamination)
- Test-count waypoints: 5,279 (pre-Siegel) → 5,313 (Siegel)
  → 5,317 (post-Siegel refactor) → 5,343 (Geman-McClure)
- Live-smoke corpus: `~/.config/pew/queue.jsonl`, 6 sources
  (opencode, codex, claude-code, openclaw, hermes,
  vscode-redacted), ~1,900–1,940 rows over the window

The eleven-release window is the rare case where the right unit
of analysis is the window, not the release. Every release
individually is a small tactical addition. The window together
is a published map of robust-statistics tail-shape space against
a fixed real corpus — and as of v0.6.217, that map is closed
enough on the location side that the next interesting move has
to be in a different lens family entirely.
