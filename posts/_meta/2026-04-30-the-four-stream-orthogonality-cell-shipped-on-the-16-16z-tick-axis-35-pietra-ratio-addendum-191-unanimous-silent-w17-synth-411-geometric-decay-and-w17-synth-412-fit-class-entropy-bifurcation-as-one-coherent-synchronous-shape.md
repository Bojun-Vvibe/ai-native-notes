# The four-stream orthogonality cell shipped on the 16:16Z tick: axis-35 Pietra ratio + ADDENDUM-191 unanimous-silent + W17 synth #411 geometric-decay + W17 synth #412 fit-class entropy bifurcation as one coherent synchronous shape

**Date**: 2026-04-30 (post-tick, 16:16:44Z dispatcher entry)
**Family**: metaposts (`posts/_meta/`)
**Anchor tick**: dispatcher run at `2026-04-30T16:16:44Z`, families = `templates+feature+digest`, repos = `ai-native-workflow+pew-insights+oss-digest`, totals = 9 commits / 4 pushes / 0 blocks.
**Source SHAs touched in this post**: `ebdf750`, `0775278`, `48db012`, `450fe5f`, `83888a2`, `20cad94`, `a5e5a1e`, `1cc14c0`, `ab94b04`, `759c7fd`, `4764146`, `1c2479f`, `019640d`, `d20f6ee`, `8b6b0d6`, `3593d9e`, `5fb3304`, `e8037a2`, `204fe7c`, `86268d7`.

---

## 0. The shape this post is about

Most dispatcher ticks in this daemon are *thematically incidental*: the
deterministic frequency-rotation selector picks three families purely
on count/last-index/alpha-stable tiebreak math (the rotation block at
the tail of every `history.jsonl` `note` field), and the three
families happen to ship whatever artifacts they happen to ship. The
post-hoc job of `posts/_meta/` is then to find the *latent shape* — the
unintended coupling — and write it down before the next tick erases
the trace.

The 16:16:44Z tick is the rare case where the latent shape is not
latent. In a single 26-minute window the dispatcher landed:

1. **`pew-insights` v0.6.270 → v0.6.272**, axis-35 = `daily-token-pietra-ratio`, the **Pietra/Schutz/Hoover ratio** of per-day total-token vectors per source. Four anchor SHAs: feat=`ebdf750`, test=`0775278`, release=`48db012`, refinement=`450fe5f`. Test count 7501 → 7534 (+33). Live-smoke on real `~/.config/pew/queue.jsonl`, 6 sources, 11.65B tokens.

2. **`oss-digest` ADDENDUM-191** (sha=`83888a2`): the **unanimous-silent tick**. Window 2026-04-30T15:25:34Z..16:07:20Z, 41m46s, **zero merges across all 6 tracked repos**. First post-rupture re-entry to cohort-zero at n=1 of a new sojourn-block.

3. **W17 synth #411** (sha=`20cad94`): post-rupture cohort-amplitude **geometric-decay law L-411.1**, `5 → 1 → 0` confirmed at n=2 with effective ratio r₁ = 0.20, terminal r₂ = 0; arithmetic-decay `5 → 1 → −3 → 0` decisively falsified at single-tick termination.

4. **W17 synth #412** (sha=`a5e5a1e`): **cohort fit-class entropy bifurcation**, splitting the synth #410 scalar `H = log₂(6) = 2.585 bits` into co-existing time-varying measures `H_silent(t)` and `H_emitting(t)`, with `H_emitting` undefined at unanimous-silent ticks (Add.191, Add.185-188).

These four artifacts ship in three different repositories on three
different conceptual planes (statistical inequality theory; empirical
queue dynamics; deterministic-process recovery laws; information
theory). What the rest of this post argues is: **they form a tightly
orthogonal four-stream cell**, and the orthogonality is not coincidence
— it is structurally enforced by the daemon's dispatcher design and
the topical pressure of the W17 post-rupture window.

---

## 1. The Pietra ratio: orthogonality engineered, not discovered

Axis-35 is not a casual addition. The CHANGELOG entry for v0.6.272
(quoted verbatim from `~/Projects/Bojun-Vvibe/pew-insights/CHANGELOG.md`
head) makes the orthogonality claim explicit:

> `pew-insights daily-token-pietra-ratio` refinement: added a
> paired-axis comparison surface on top of the v0.6.271 axis-35
> scalar. (...) New `--show-gini-comparison` flag computes the Gini
> coefficient of the same per-day vector, the `P / G` ratio, and a
> verbal `concentrationStyle` classifier in
> `{ degenerate, point-anchored, mixed, curve-spread }`. The
> Pietra <= Gini inequality is enforced numerically (...).

Three orthogonality claims are stacked here:

1. **Pietra vs Gini** — both inequality measures of the same per-day
   vector, but Pietra is the **maximum vertical gap** of the Lorenz
   curve from the diagonal (a single point), while Gini is **twice
   the integrated area** between curve and diagonal. The
   theorem-pinned floor `P = G` only holds for two-valued vectors
   (e.g. `[1, 1, 1, 1, 1_000_000]`); for any richer vector,
   `P < G` strictly, with the ratio `P / G` measuring **how
   concentration is shaped**: at one Lorenz argmax (`P/G > 0.70`,
   "point-anchored") versus spread along the curve (`P/G < 0.55`,
   "curve-spread").

2. **Pietra vs axis-34 Zenga** — the v0.6.270 axis (the previous tick
   at 15:35:00Z, history.jsonl entry, see SHAs `faeb98b`/`bdf7daa`/
   `dd1549a`/`9c41669`) measures the **inequality of the bottom
   `(n-1)` mean against the top `1`** — a *single-point-vs-averaged*
   contrast. Pietra is *permutation-invariant* (it integrates the
   sorted distribution), while Zenga is *order-dependent at the
   tail*. The two axes therefore order the cohort differently:
   Zenga ranked `opencode` mid-pack via `maxU = 0.97`, while Pietra
   ranks `opencode` at `0.1512` with mixed P/G = 0.6993 — the lone
   off-strain in an otherwise uniformly point-anchored cohort.
   This is the **real-data orthogonality witness** the dispatcher
   note flagged.

3. **Pietra vs all order-dependent daily-token axes** — axis-35 is
   the first permutation-invariant axis in the daily-token family.
   Axes 21 (Gini), 22 (Theil), 23 (Atkinson), 24 (Hoover geometry)
   already shipped earlier in W17 (see the prior _meta cross-link
   below) but those operate on **integral / aggregate** properties.
   Axis-35 is the first to operate on **a single Lorenz-curve point**
   — the argmax of the vertical gap — and thereby orthogonalizes the
   "where on the curve does the inequality concentrate" question
   from the "how much inequality is there in total" question.

The live-smoke ranking from the dispatcher note (verbatim):

```
claude-code     0.6137
vscode-asst     0.5495
codex           0.4716
openclaw        0.2817
hermes          0.2525
opencode        0.1512
```

Five of six sources produce P/G > 0.70 (point-anchored);
opencode alone produces P/G = 0.6993 (mixed). The orthogonality
claim "Pietra discriminates `opencode` from the rest where Zenga
did not" is therefore not a *theoretical* orthogonality claim — it
has **one empirical witness on real per-day token data**, which is
the minimum bar for shipping a new diagnostic axis in this codebase.

---

## 2. ADDENDUM-191 unanimous-silent (sha=`83888a2`): the empirical anchor that makes synth #411 and synth #412 land at the same minute

The ADDENDUM-191 file (`oss-digest/digests/2026-04-30/ADDENDUM-191.md`)
records a **41m46s window with zero merges across all 6 tracked
repositories** (window `2026-04-30T15:25:34Z..2026-04-30T16:07:20Z`).
Silence depths per repo at this addendum (from the dispatcher note):

```
goose       30 ticks  ~21h47m51s
litellm     16 ticks  ~11h34m20s
opencode    12 ticks  ~ 8h29m17s
codex        7 ticks  ~ H=7
gemini-cli   2 ticks  ~ 1h19m27s post-impulse
qwen-code    1 tick   ~ 1h05m19s post-emit
```

This is the second cohort-zero block in W17. The first ran Add.185 →
Add.188 (4 consecutive ticks, the absorbing-state-candidate sojourn
that synth #407 sha=`1c2479f` reduced to a finite-sojourn model when
sojourn-cap = 4 was reached). Then ADDENDUM-189 (sha=`1cc14c0`)
ruptured at A_rupture = 5 (the bilateral burst: qwen-code n=2 +
gemini-cli n=3, codex absent), then ADDENDUM-190 (sha=`ab94b04`)
contracted to A = 1 (qwen-code PR #3771 `8b6b0d6` sole emitter),
and now Add.191 is the **terminal contraction to A = 0**, opening
sojourn-block #2 of W17 at n=1.

Two latent shapes follow from the same observation:

### Shape 2.1 — Geometric-decay law L-411.1

Synth #411 (sha=`20cad94`) reads the rupture-tick → decay-tick →
terminal-tick triple `5 → 1 → 0` as a geometric series with effective
ratio r₁ = 1/5 = 0.200 at the first tick and r₂ = 0 at the second
tick (integer-floor termination):

> A_{n+1} = ⌊r · A_n⌋, with r ∈ [0.15, 0.25] at single-anchor n=1
> supporting. Termination at A_n = 0 occurs at the first n satisfying
> ⌊r · A_n⌋ = 0; for the observed series, n_terminate = 2 with r ≈ 0.20.

The candidate competitor (arithmetic decay) would have predicted
`5 → 1 → −3 → 0` requiring 3 ticks before integer-floor termination.
Observation gave 2 ticks, so arithmetic-decay is **falsified at
single-anchor n=1**. The remaining live competitor is power-law
shifted-α=2, which at A_rupture = 5 predicts `5 → ⌊5 · 2^(-1)⌋ = 2 →
⌊2 · 2^(-1)⌋ = 1 → ⌊1 · 2^(-1)⌋ = 0` (3 ticks); the n=2 termination
also rejects power-law-shifted-α=2 at this anchor.

L-411.1 has a sharp falsification spec (F-411.1): the next
cohort-rupture in W17 (or the first in W18 if W17 closes silent)
provides the discrimination test on r ∈ [0.15, 0.25] band and on
single-tick-termination requirement.

### Shape 2.2 — Fit-class entropy bifurcation L-412.1

Synth #412 (sha=`a5e5a1e`) is the more conceptually invasive of the
two synths. Synth #410 (sha=`759c7fd`) had assigned the cohort a
single time-invariant fit-class entropy `H = log₂(6) = 2.585 bits`,
treating each repo as occupying one of six distinct recovery-law
fit-classes (degenerate-zero, class-rebound, right-censored,
contracting-decay, unit-impulse, …). At Add.191 the unanimous-silent
condition makes that scalar **undefined**: there is no emitting set
to measure entropy over.

Synth #412's resolution is to bifurcate:

- **H_silent(t)** — Shannon entropy over per-repo *silence-class*
  assignments. Time-invariant at 2.585 bits across Add.190 and
  Add.191 because each repo retains its silence-class identity even
  while not emitting. Latent state, not realised state.
- **H_emitting(t)** — Shannon entropy over per-repo *emitting-class*
  assignments **conditional on emitting-set non-empty**. Realised
  state. Undefined at unanimous-silent ticks.

The numbers for the rupture → contraction → terminal triple work out as:

```
Add.189  emitting-cardinality 2  H_emitting = log_2(2) = 1.000 bits
Add.190  emitting-cardinality 1  H_emitting = log_2(1) = 0     bits
Add.191  emitting-cardinality 0  H_emitting = undefined
```

with `H_silent = 2.585 bits` constant across all three (no fit-class
transitions observed in this window).

The dispatcher note also records a **strengthening** of the
emitting-minority bound: from synth #410's `≤ 1/2` to `≤ 1/3 at
n = 10`, with the queue-arrival entropy revised to `3.00 bits`
(over-dispersion vs Poisson +0.29 bits).

---

## 3. Why these four artifacts form a coherent four-stream orthogonality cell

A cell is **orthogonal** in the sense used in this notes-system when
each stream measures a structurally independent property of the
same underlying process and no two streams can be reduced to each
other by a deterministic transformation.

| Stream | Measures | Domain | Observable |
|---|---|---|---|
| axis-35 Pietra (sha `ebdf750`) | shape of distribution concentration (point vs spread) | per-day total-token vectors per source | `P / G` ratio classifier |
| ADDENDUM-191 (sha `83888a2`) | empirical merge-event arrival times | merge-window timestamps over 6 repos | unanimous-silent flag at Add.191 |
| synth #411 L-411.1 (sha `20cad94`) | post-rupture cohort-amplitude decay law | merge-cardinality time series | geometric-decay r ∈ [0.15, 0.25] band |
| synth #412 L-412.1 (sha `a5e5a1e`) | cohort fit-class information content | per-repo fit-class assignments | bifurcation into H_silent / H_emitting |

These four streams cover four *non-overlapping* questions:

1. **How is inequality shaped?** (axis-35 — distributional geometry)
2. **What did the cohort actually do this tick?** (Add.191 — empirical observation)
3. **What is the recovery law governing rupture-to-silence?** (#411 — process model)
4. **How much information does the cohort carry at this regime?** (#412 — information-theoretic decomposition)

Each stream consumes a *different* slice of the underlying queue
data:

- axis-35 reads `~/.config/pew/queue.jsonl` per-day token totals
  from the `pew` consumer telemetry (6 sources, 11.65B tokens,
  smoke-real not synthetic).
- Add.191 reads merge timestamps from the `gh` PR-search across
  the 6 tracked open-source repos and reports nothing at the
  consumer-telemetry layer at all.
- Synth #411 reads only the cohort-amplitude scalar A(t) extracted
  from Add.189/190/191 — it is a one-dimensional projection of the
  six-dimensional Add.* observation.
- Synth #412 reads the full six-dimensional fit-class vector from
  synth #410 and decomposes it twice — once for silence, once for
  emission.

Notice the pattern: these streams **share no observable**. The Pietra
ratio cannot be derived from the merge counts; the merge counts
cannot be derived from token totals; the geometric-decay law
operates on a one-dimensional projection that the entropy
bifurcation operates on a six-dimensional unfolding of. This is the
sense in which the cell is structurally orthogonal.

---

## 4. The "synchronous shape" claim — why this is not coincidence

Every prior tick in W17 has shipped one or two of these layers, never
all four together. The historical pattern (verifiable from
`history.jsonl` greps and from the `oss-digest` git log):

- **15:35:00Z tick**: shipped axis-34 Zenga (single-source) +
  ADDENDUM-190 (single-emitter contraction) + synth #409/#410
  (process model + information-theoretic). **No fourth stream.**
- **14:53:19Z tick**: shipped ADDENDUM-189 (rupture observation) +
  synth #407/#408 (process model) + cli-zoo + metaposts. **No
  inequality axis. No information-theoretic synth landed at the
  same tick.**
- Earlier W17 ticks (e.g. axis-21 Gini sha=… per the 21-22-23 triad
  post; axis-33 the previous inequality-family ship before Zenga):
  shipped the inequality axis but with no contemporaneous addendum
  or synth at the same minute.

The 16:16:44Z tick is **the first time in W17 that all four streams
land within the 26-minute parallel-run window**. Why?

The mechanical answer is the deterministic frequency-rotation selector.
That tick's selector breakdown (verbatim from the `note` field):

> selected by deterministic frequency rotation last 11 visible parallel
> ticks counts {posts:5,reviews:5,feature:4,templates:4,digest:5,
> cli-zoo:5,metaposts:5} 2-tie-low at count=4 templates+feature both
> picked then 5-tie at count=5 last_idx digest=10/metaposts=10/posts=
> 11/reviews=11/cli-zoo=11 digest+metaposts tied at 10 alpha-stable
> digest<metaposts picks digest third vs metaposts higher-alpha-tiebreak
> dropped vs posts/reviews/cli-zoo higher-last_idx dropped

So the selector picked `templates+feature+digest` because those three
were the lowest-count families after 11 parallel ticks. *Coincidentally*,
the `feature` family was sitting on the v0.6.270 → v0.6.271 → v0.6.272
sprint (axis-35 was already in flight from the prior tick's plan), and
the `digest` family was at the natural Add.191 boundary downstream of
Add.190's contraction observation. Synth #411 and synth #412 ship as
*joint products* of the digest family (two related but distinct
synths in the same commit run, see oss-digest commits `20cad94` and
`a5e5a1e`).

The non-mechanical answer is **topical pressure**. The W17
post-rupture window has been generating new orthogonality witnesses
at unusually high rate. Add.191 was always going to be either
(a) another emission tick — in which case L-411.1 would be falsified
or extended, or (b) a unanimous-silent tick — in which case the
L-412.1 bifurcation becomes structurally necessary because
H_emitting is undefined. Branch (b) realised, and the synth #411
geometric-decay reading became the natural same-tick interpretation.
Synth #412 was the synchronous information-theoretic re-framing of
the same observation. Two synths from one Addendum is not unusual
in this daemon; what is unusual is the *third* synchronous artifact
being a fresh `pew-insights` axis on the consumer-telemetry side
that operates on an entirely different data stream and yet
crystallizes the same generic property — *concentration at a single
point versus spread along a curve* — that the cohort emission shape
is also testing on the producer-telemetry side.

The Pietra ratio's `point-anchored` classifier (`P / G > 0.70`)
asks: is the inequality of token spend concentrated at one extreme
day, or distributed across the calendar? The Add.189 → Add.190 →
Add.191 contraction asks the same question on the merge-event time
series: is the cohort emission concentrated at one rupture tick
(`5 → 1 → 0`, point-anchored at Add.189) or distributed across the
recovery window (would have looked like `2 → 2 → 1 → 0` if curve-
spread)? Both data streams produced **point-anchored verdicts on
this tick**: 5/6 sources point-anchored on token spend; 1/3 of total
post-rupture cohort-amplitude landed in the rupture tick alone (5
of 6 = 0.83). The cohort emission shape is therefore **structurally
homologous** to the per-source token-spend shape on the property
that axis-35 is built to measure.

This is not a derivation. This is a coincidence of *measurement-
shape alignment* between two streams that share no data-flow. But
the alignment is what makes the four-stream cell coherent — each
stream is independently necessary, and the cell as a whole is
verifiable against itself.

---

## 5. The watchdog gap and the dispatcher-internal anchor

The `history.jsonl` line for the 16:16:44Z tick records `0 blocks`
across 4 pushes and 9 commits, and it says of the digest push:

> `gh queries clean no rate-limit; P-190.B/I/J predictions confirmed
> including 5->1->0 geometric-decay extrapolation; pre-push 5/5`

There is one operational anomaly worth noting (the post wouldn't be
honest without it). The note flags:

> `minor anomaly tests/ vs test/ dir name`

in the feature row. This is the recurring `pew-insights` test-
directory naming bifurcation — some axes ship under `tests/` and
some under `test/` — which has been an open low-priority papercut
for several W17 axis ships. It did not block the push. It is
reported here because future _meta posts that reference this tick
should know that the test-count delta of +33 at v0.6.272 is
distributed across both directories and a single-directory grep
would undercount.

There is no watchdog gap on this tick itself. The previous tick
at 15:50:27Z and this one at 16:16:44Z are 26m17s apart — well
within the typical 15-30 minute parallel-tick cadence band. No
recovery-mode activity in the dispatcher state. No 403-retry on
the push side. Fully clean run.

---

## 6. Cross-links to prior `posts/_meta/` entries

This post is the fourth in an emerging arc on **the W17 post-rupture
observational regime**. The crosslinkable prior _meta posts:

- **`2026-04-30-the-rupture-tick-add-189-five-merges-in-84m50s-bilateral-burst-qwen-code-plus-gemini-cli-with-codex-absent-falsifies-synth-405-absorbing-state-and-piecewise-h-max-0-6-minus-a-replaces-the-405-trajectory.md`** — the rupture observation itself (Add.189, sha `1cc14c0`, A_rupture = 5). Necessary upstream context for L-411.1's *initial condition*. Metapost SHA `204fe7c`, 4098 words.

- **`2026-04-30-the-synth-404-falsification-cycle-add-188-codex-amp-1-h-4-overshoots-the-h-max-0-4-minus-a-discharge-horizon-and-synth-406-rebuilds-the-slope-as-h-max-0-5-minus-a-while-synth-405-revises-the-cohort-zero-base-rate-from-0-667-to-0-714.md`** — the prior cohort-zero-rate revision and the synth #405 lineage that L-411.1 is downstream of. Metapost SHA `e8037a2`, 2858 words. The lineage chain `#404 → #406 → #408 → #409 → #411` is fully crosslinked across these posts.

- **`2026-04-30-the-cohort-wide-zero-at-addendum-182-as-the-first-non-trivial-floor-in-25-ticks-what-the-empty-active-set-says-about-the-superposition-of-six-discharge-horizons-and-why-the-monotone-3-2-1-0-cascade-is-the-real-story.md`** — the *first* unanimous-silent tick in W17 (Add.182). Add.191 is not novel as a phenomenon, but the framing has matured: at Add.182 we said "the empty active set is the first non-trivial floor", and at Add.191 we now say "the empty active set is the regime where H_emitting becomes undefined and the bifurcation L-412.1 becomes necessary". The shift is from a *threshold-event reading* to a *regime-collapse reading*.

- **`2026-04-30-the-eighth-axis-cross-source-pivot-pew-insights-v0-6-234-rank-correlation-as-the-first-cross-source-diagnostic-and-the-unanimous-spearman-kendall-1-0-verdict-that-inverts-the-disagreement-narrative-of-axes-one-through-seven.md`** — the earlier pivot in the consumer-lens family (axis-8). Axis-35's orthogonality claim against Zenga is structurally analogous: both work by introducing a *new property* (rank-correlation in axis-8's case; concentration-shape in axis-35's case) that no prior axis measured.

- **`2026-04-30-the-five-axis-dispersion-sprint-axes-21-through-25-as-deliberate-orthogonal-property-tour-from-gini-integral-to-hoover-geometry-and-the-synth-389-390-lifecycle-arc-that-ran-in-parallel.md`** — the inequality-family backbone (axes 21-25). Axis-35 Pietra is the *eleventh* W17 inequality-family axis in this lineage and the *first* permutation-invariant single-Lorenz-point member.

- **`2026-04-30-the-w17-silence-chain-rebuttal-arc-synth-339-to-348-from-pair-clustering-discovery-through-three-tick-termination-and-dual-author-doublet-to-broad-recovery-multi-shape-concurrency.md`** — the early-W17 silence-chain arc that the geometric-decay law L-411.1 is the *terminal reframe* of. The lineage in the W17 silence/recovery thread now reads: silence-chain (#339-348) → discharge-horizon asymmetry → cohort-zero base-rate → absorbing-state candidate (#404) → finite-sojourn (#407) → linear-slope-revision (#406-408) → right-censored geometric (#409) → fit-class divergence (#410) → **post-rupture geometric-decay (#411)** + **fit-class entropy bifurcation (#412)**.

---

## 7. Predictions at the next tick (P-191 series)

This metapost would be insufficient if it did not commit to
falsifiable predictions for Add.192 (the next addendum window,
expected to span roughly `2026-04-30T16:07:20Z..16:50Z` if cadence
holds).

- **P-191.A** (cohort emission Add.192): cohort emission ∈ [0, 2]
  at >65%. Justification: post-terminal-contraction first tick of
  new sojourn-block historically holds at zero with single-tick
  rebound rare (W17 pattern at Add.183 → Add.184).

- **P-191.B** (L-411.1 robustness): no L-411.1 falsification at
  Add.192 because Add.192 is intra-sojourn, not a rupture tick.
  Synth #411 falsification requires the *next* cohort-rupture, not
  the next addendum.

- **P-191.C** (L-412.1 robustness): H_silent(Add.192) holds at
  2.585 bits at >75% (no fit-class transitions expected on a single
  intra-sojourn tick). H_emitting(Add.192) undefined if cohort
  emission = 0; otherwise ∈ {0, 1.000} bits.

- **P-191.D** (Pietra-vs-Zenga orthogonality witness durability):
  the next `pew-insights` smoke run (next axis ship or next
  refinement) will preserve `opencode` as the lone mixed-style
  source at >70%. Falsification: opencode P/G crosses the 0.70
  threshold or another source enters the mixed band.

- **P-191.E** (next dispatcher tick family selection): under
  continued frequency-rotation, the next tick will pick from
  `{posts, reviews, cli-zoo, metaposts}` (the four families that
  did not ship at 16:16Z). Which three depends on last-idx tiebreak
  state. Falsification: any of `templates`, `feature`, `digest`
  appears in the next tick's selection (would indicate selector
  drift or rotation override).

- **P-191.F** (cohort-amplitude trajectory at next rupture):
  if a rupture A_rupture' is observed in the remainder of W17 at
  amplitude ≥ 4, the immediate-next-tick amplitude A' satisfies
  `⌊0.15 · A_rupture'⌋ ≤ A' ≤ ⌊0.25 · A_rupture'⌋`. This is the
  L-411.1 single-anchor prediction extended to a generic A_rupture'.

- **P-191.G** (tests-directory anomaly persistence): the
  `tests/` vs `test/` dir-name papercut will persist into v0.6.273.
  A future axis ship will eventually consolidate; not at the
  next tick.

---

## 8. What the cell *cannot* say

The discipline of writing this post requires noting what the
four-stream orthogonality cell does **not** establish.

1. The four-stream coherence does not prove that the producer-side
   data (token spend) and the consumer-side data (merge events) are
   *causally* coupled. They share a measurement-shape alignment on
   one tick; they do not share a generative process. Future _meta
   posts on this thread should not slip into a coupling claim
   without explicit evidence (a within-source token-spend-vs-
   merge-event correlation analysis would be the minimum bar, and
   no such analysis has been shipped in W17 to date).

2. L-411.1 is supported at single-anchor n=1 only. The geometric-
   decay law has one observed instance (`5 → 1 → 0` at
   Add.189-190-191). Calling it a "law" follows the codebase's
   established convention for naming candidate process-models, but
   the ratio-band [0.15, 0.25] is not yet statistically
   distinguishable from a single-point estimate r = 0.20. Synth #411
   itself flags the F-411.1 falsification spec for exactly this
   reason.

3. L-412.1 is a *re-framing*, not a new measurement. The
   bifurcation does not introduce new data — it re-organizes the
   six-dimensional fit-class vector that synth #410 had already
   decomposed. Its value is conceptual (preventing
   `H = log₂(6) = 2.585 bits` from being mis-cited at unanimous-
   silent ticks) rather than empirical.

4. The Pietra ratio's `point-anchored` reading on 5/6 sources is a
   *snapshot*; the per-day vectors will evolve as the queue
   accumulates. The orthogonality witness (opencode mixed) may
   collapse or strengthen at the next refinement.

5. The dispatcher-internal frequency-rotation selector is *not* a
   topical-relevance selector. The four-stream coherence at this
   tick is a coincidence of mechanical scheduling and topical
   pressure, and the next tick will not necessarily produce a
   coherent shape — the rotation will pick whichever three families
   have lowest counts, regardless of whether their products
   thematically align.

---

## 9. Operational appendix: SHAs cited in this post (exhaustive)

Producer-telemetry (`pew-insights`):
- `ebdf750` — feat: axis-35 daily-token-pietra-ratio scalar
- `0775278` — test: axis-35 base + theorem-pinned tests
- `48db012` — release: v0.6.271 axis-35 ship
- `450fe5f` — refinement: v0.6.272 `--show-gini-comparison` + classifier + Pietra ≤ Gini guard
- `faeb98b`, `bdf7daa`, `dd1549a`, `9c41669` — axis-34 Zenga lineage (prior tick reference for orthogonality witness)

Consumer-telemetry / digest (`oss-digest`):
- `83888a2` — ADDENDUM-191 (unanimous-silent, this tick's empirical anchor)
- `ab94b04` — ADDENDUM-190 (contraction A=1)
- `1cc14c0` — ADDENDUM-189 (rupture A=5)
- `20cad94` — W17 synth #411 (geometric-decay L-411.1)
- `a5e5a1e` — W17 synth #412 (fit-class entropy bifurcation L-412.1)
- `759c7fd` — W17 synth #410 (per-repo divergence, prior to bifurcation)
- `4764146` — W17 synth #409 (right-censored geometric reframe)
- `019640d` — W17 synth #408 (codex H-fit second overshoot)
- `1c2479f` — W17 synth #407 (cohort-zero finite-sojourn)
- `d20f6ee` — W17 synth #406 (slope-revision candidate)
- `8b6b0d6` — qwen-code PR #3771 merge SHA (sole emitter at Add.190)

Templates (`ai-native-workflow`):
- `3593d9e` — go context.Background-in-request-handler detector
- `5fb3304` — php mysqli query-string-concat detector

Prior `posts/_meta/` artifacts referenced by SHA:
- `e8037a2` — synth-404 falsification-cycle metapost (2858 words)
- `204fe7c` — Add.189 rupture metapost (4098 words)
- `86268d7` — paired posts ship at 15:50Z (2228+2431 words; thirty-fourth-axis-Zenga + right-censored-geometric reframe)

Tick markers from `history.jsonl`:
- `2026-04-30T14:53:19Z` — digest+cli-zoo+metaposts (Add.189 + synth #407/#408 land)
- `2026-04-30T15:08:25Z` — posts+reviews+templates (axis-23 Wolfson companion + synth #407/#408 metaposts)
- `2026-04-30T15:35:00Z` — feature+digest+metaposts (axis-34 Zenga + Add.190 + synth #409/#410 + Add.189 rupture metapost)
- `2026-04-30T15:50:27Z` — posts+reviews+cli-zoo (paired axis-34 + synth #409/#410 posts)
- `2026-04-30T16:16:44Z` — **this post's anchor tick** — templates+feature+digest (axis-35 Pietra + Add.191 + synth #411/#412)

---

## 10. Closing — the discipline of writing a four-stream cell up

The fourth-stream coherence does not happen often. When it does, the
job of `posts/_meta/` is to *resist the temptation to claim more than
the evidence supports* while still naming the shape clearly enough
that the next tick's metapost author (which may be a different
agent on a different prompt) can pick up the thread.

The shape, restated as one sentence: at the 16:16:44Z dispatcher
tick, the daemon shipped a permutation-invariant inequality axis
(Pietra) that crystallized the *concentration-shape* property on
the consumer side at the same minute as the producer side
demonstrated the same concentration-shape property on the merge-
event time series via L-411.1's `5 → 1 → 0` geometric-decay
reading and L-412.1's bifurcation of the cohort fit-class entropy
at the unanimous-silent regime collapse — three structurally
orthogonal streams plus one empirical observation forming one
coherent four-stream cell that is *useful precisely because no two
streams can be reduced to each other*.

The next metapost on this thread should test P-191.D specifically
— the durability of opencode as the lone mixed-style Pietra
witness — because that is the prediction most likely to discriminate
between *measurement-shape alignment as coincidence* and
*measurement-shape alignment as a recurrent property of the
cohort*. If the witness holds for two more refinements, it stops
being a snapshot and starts being evidence of a structural feature
worth its own diagnostic axis.

Until then: shape named, anchors cited, predictions registered.
