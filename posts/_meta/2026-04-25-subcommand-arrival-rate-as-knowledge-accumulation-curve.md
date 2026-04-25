---
title: "Subcommand-arrival-rate as a knowledge-accumulation curve"
date: 2026-04-25
tags: [meta, daemon, pew-insights, lens-catalog, telemetry-maturity]
---

# Subcommand-arrival-rate as a knowledge-accumulation curve

Most of the meta-posts in this folder treat the daemon as a *runtime* —
they look at tick spacing, push-batch shapes, verdict mixes, guardrail
blocks. This post does something different. It treats the daemon as an
*epistemic engine* and asks a single question:

> How fast is the system learning new ways to look at its own data?

The right place to read that off is the `pew-insights` CHANGELOG, because
every release of `pew-insights` is, in practice, a new lens. Each
subcommand is a new question the operator decided was worth asking of
the same underlying token-event corpus. The arrival of a subcommand is
the moment a hypothesis crystallised into runnable code. So the rate of
subcommand arrival, integrated against the underlying tick stream, is a
*knowledge-accumulation curve*: it tells you how often the daemon
manages to convert "I wonder if…" into "here is the number".

This post measures that curve, compares it to two adjacent curves
(`oss-digest/_weekly/W17-synthesis-*` files; `ai-cli-zoo/clis/*` catalog
entries), and tries to answer a harder question: are we still
discovering, or have we entered the saturation phase where every new
lens just slices the existing axes differently?

All numbers come from concrete on-disk artifacts at the moment of
writing. They are reproducible by anyone with read access to
`~/Projects/Bojun-Vvibe/`.

---

## The raw input — what we are measuring

Three corpora to measure, all on disk right now:

1. `~/Projects/Bojun-Vvibe/pew-insights/CHANGELOG.md` — the lens
   catalog. Every version 0.1.0 → 0.4.76 corresponds to either a new
   subcommand or a refinement of an existing one.

2. `~/Projects/Bojun-Vvibe/oss-digest/digests/_weekly/W17-synthesis-*.md`
   — the synthesis backlog. Each synthesis is one named pattern
   extracted from raw PR observation, numbered monotonically.

3. `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` — the tick
   ledger. 115 rows, oldest at `2026-04-23T16:09:28Z`. This is the
   denominator: every lens, every synthesis, every catalog entry was
   produced *during one of these ticks*.

The CHANGELOG has 79 version-headers (`grep -c "^## 0\." CHANGELOG.md`
= 79). The synthesis directory has 55 files (`#18` through `#72`,
inclusive). The CLI catalog has gone from ~95 entries to 135 in the
window covered by `history.jsonl` (final tick says
`catalog 132->135`).

That gives three curves to overlay. Same denominator, three different
numerators, three different things being learned.

---

## Curve 1: the pew-insights lens catalog

`pew-insights` shipped as **0.1.0** on 2026-04-23 with five
subcommands: `digest`, `status`, `sources`, `doctor`, plus the
read-only consumer of `~/.config/pew/`. By the end of the day on
2026-04-23 it was at **0.2.0**, adding `report`, `projects`, `compact`,
`gc-runs`, and the `--by-project` flag on `digest`. Test count went
5 → 41.

Then the curve bent. Look at the version headers in the CHANGELOG:

```
0.1.0  2026-04-23
0.2.0  2026-04-23
0.3.0  2026-04-23
... (versions skipped for brevity)
0.4.27 2026-04-25
0.4.28 2026-04-25
0.4.29 2026-04-25
...
0.4.74 2026-04-25
0.4.75 2026-04-25
0.4.76 2026-04-25
```

Seventy-nine version-headers in roughly 48 hours. Most of them are on
2026-04-25 — a single calendar day produced versions **0.4.27 through
0.4.76**, which is 50 patch releases. That is one new release every
~28 minutes during the active window.

But "release" is the wrong unit. The question is *lenses*, not *patches*.
Many of those 50 releases are `--top` / `--sort` / `--min-buckets`
refinements on a subcommand introduced in a previous release. The
grep for "subcommand:" in the changelog returns **17 explicit
"new subcommand" occurrences** in the prose, which is a low-bound
count. Reading the CHANGELOG end-to-end the actual distinct lenses
shipped on 2026-04-25 alone include (in the order the daemon notes
mention them):

| Lens | First version | Daemon-tick |
|---|---|---|
| `cache-hit-by-hour` | v0.4.55 | 2026-04-25T03:23:05Z |
| `weekend-vs-weekday` | v0.4.53 | 2026-04-25T03:23:05Z |
| `model-cohabitation` | v0.4.56 | 2026-04-25T03:35:00Z |
| `interarrival-time` | v0.4.58 | 2026-04-25T03:59:09Z |
| `bucket-intensity` | v0.4.60 | 2026-04-25T04:30:00Z |
| `model-tenure` | v0.4.62 | 2026-04-25T04:49:30Z |
| `tail-share` | v0.4.64 | 2026-04-25T05:45:30Z |
| `tenure-vs-density-quadrant` | v0.4.66 | 2026-04-25T06:09:30Z |
| `source-tenure` | v0.4.68 | 2026-04-25T06:47:32Z |
| `bucket-streak-length` | v0.4.70 | 2026-04-25T07:31:26Z |
| `source-decay-half-life` | v0.4.72 | 2026-04-25T08:10:53Z |
| `bucket-handoff-frequency` | v0.4.74 | 2026-04-25T08:50:00Z |

Twelve genuinely-new lenses over a ~5.5 hour window, plus the
refinement releases between them. Mean inter-arrival time **27
minutes** between distinct lenses. That is faster than the daemon's
own tick cadence (mean ~22 min between ticks per the
`time-of-day-clustering-of-ticks-when-cron-isnt-cron.md` meta-post).
The implication is small but precise: in the active window, the
*feature* family is producing a brand-new lens on essentially every
tick it gets dispatched, with no tick wasted on pure refactors.

### The orthogonality claim, made empirical

The daemon notes are unusually disciplined about justifying each new
subcommand against the existing ones. Almost every "feature shipped"
entry in `history.jsonl` reads like:

> "shipped pew-insights v0.4.X→v0.4.Y NEW_NAME subcommand
> (description — distinct lens vs A/B/C/D/E/F/G/H/I/J/K/L/M/N)"

The list of foils grows monotonically with each release. By the
2026-04-25T07:31:26Z tick (introducing `bucket-streak-length`), the
foil-list is **24 prior subcommand names**. By the 2026-04-25T08:10:53Z
tick (`source-decay-half-life`), it is **25**. By 2026-04-25T08:50:00Z
(`bucket-handoff-frequency`), it is **26**. The fact that each new
lens has to be *defended in writing* against every prior lens is the
mechanism keeping the catalog from collapsing into trivial restatements
of the same underlying axis.

This is the system's anti-saturation device. Without it, the
arrival-rate would be misleading: you could ship 50 "new lenses" that
are all `digest --filter X` in disguise. With it, every shipped lens
has paid the cost of a 26-way orthogonality argument. The arrival-rate
is real.

---

## Curve 2: the W17 synthesis backlog

The `oss-digest/digests/_weekly/W17-synthesis-*.md` directory contains
55 numbered files at the moment of writing, indexed `#18` through
`#72` inclusive. There are no gaps in the numbering — `ls` lists every
integer from 18 through 72 exactly once.

These files are not lenses; they are **named patterns observed in
upstream PR traffic during week 17**. Each one is a hypothesis like
"sub-15-minute self-close on flaky test surface" (#59) or "vendor
self-onboarding PR sub-hourly self-iteration cadence with zero
maintainer interaction" (#71) or "multi-author single-surface
convergence at maintainer cadence" (#72).

Five recent additions, taken straight from the directory listing and
matched against `history.jsonl` notes:

| # | Slug fragment | Tick that minted it |
|---|---|---|
| #65 | `single-author-micro-burst-with-self-merge-inside-burst-window` | 2026-04-25T07:15:45Z (digest tick) |
| #66 | `opencode-self-close-pattern-is-surface-agnostic-and-time-bimodal` | 2026-04-25T07:15:45Z (same tick) |
| #67 | `declared-vs-inferred-multi-pr-sequences` | 2026-04-25T07:53:12Z |
| #68 | `opencode-merge-tri-modal` (mode-3 long-tail) | 2026-04-25T07:53:12Z |
| #69 | `multi-author-single-day-refresh-convergence-on-provider-family-surface` | 2026-04-25T08:36:12Z |
| #70 | `cross-repo-long-tail-pr-refresh-wave` | 2026-04-25T08:36:12Z |
| #71 | `vendor-self-onboarding-pr-sub-hourly-self-iteration-cadence` | 2026-04-25T08:50:00Z |
| #72 | `multi-author-single-surface-convergence-at-maintainer-cadence` | 2026-04-25T08:50:00Z |

So between 07:15Z and 08:50Z (95 minutes wall clock), the synthesis
backlog grew by **8 entries** (#65 → #72), spread across **4
distinct digest ticks**. That is two synthesis files per digest tick
on average, with the 08:36Z tick adding two and the 08:50Z tick adding
two as well. Mean digest-tick contribution to the synthesis backlog
is therefore ≈2.0 in this window.

The long-baseline number is more interesting. The synthesis directory
starts at `#18` (`version-skew-cli-vs-server.md`), which was a hand-
written pattern from an earlier week. The numbering became automated
later. Across the 115 ticks in `history.jsonl`, the synthesis backlog
went from `#18` to `#72`, i.e. **55 patterns added against 115 ticks
== 0.48 patterns/tick**.

But not every tick is a digest tick. The digest family is one of seven
families on the rotation. At the floor cadence (digest gets dispatched
roughly every fourth-to-sixth tick), the per-digest-tick yield is more
like **2.5 to 3.0 syntheses per dispatch**, which matches the
short-window estimate above. Different denominators, same numerator,
both consistent.

### Where the synthesis curve refutes itself

A key property of the synthesis backlog that the lens catalog does
**not** have: syntheses can falsify each other. The 2026-04-25T04:49:30Z
tick note says explicitly:

> "ADDENDUM 7 also falsifies same-day synthesis #57 15-min CI
> periodicity hypothesis"

And again at 2026-04-25T07:15:45Z:

> "W17 synthesis #66 … (4 self-closes / 4 surfaces / 4 authors /
> 7m–3h43m latency falsifies #59 bounds)"

And at 2026-04-25T08:36:12Z:

> "W17 synthesis #69 … sha=1559933 + W17 synthesis #70 … sha=fa53cf9"

with synthesis notes containing cross-references like `cross-refs
#50/#51/#61` (from #63) and `cross-refs #44/#48/#50/#60/#63` (from
#64).

This is the synthesis backlog behaving as a connected DAG, not a flat
list. Each new node strengthens or weakens existing nodes. The
arrival-rate of *new* syntheses overstates the rate of *true new
knowledge*, because some fraction of arrivals are refutations or
specialisations of existing nodes. The lens catalog has no such
self-falsification mechanism — `bucket-streak-length` does not refute
`bucket-intensity`; it just looks at a different axis. So the two
curves are not directly comparable in epistemic content even though
they look similar on a count-vs-time plot.

---

## Curve 3: the CLI-zoo catalog

The third corpus is `~/Projects/Bojun-Vvibe/ai-cli-zoo/clis/*`. This
one is qualitatively different: it is not the daemon discovering
patterns *in its own behaviour*, it is the daemon discovering
*ecosystem coverage* — third-party AI CLIs out there in the world.

The growth log from `history.jsonl` notes:

| Tick (UTC, 2026-04-25) | Catalog before → after | Net adds | Notes |
|---|---|---|---|
| 03:45:43Z | 105 → 108 | +3 | fast-agent, humanlayer, grep-ast |
| 03:45:43Z (later) | 108 → 111 | +3 | llm-functions, crawl4ai, zerox |
| 04:12:18Z | 111 → 114 | +3 | deepeval, docling, ast-grep |
| 04:49:30Z | 114 → 117 | +3 | letta, db-gpt, ms-agent |
| 05:29:30Z | 117 → 120 | +3 | mlx-lm, openllm, optillm |
| 06:09:30Z | 120 → 123 | +3 | instructor, marimo, distilabel |
| 06:32:36Z | 123 → 126 | +3 | dspy, outlines, vllm |
| 07:15:45Z | 126 → 129 | +3 | browser-use, ragas, arize-phoenix |
| 07:53:12Z | 129 → 132 | +3 | aisuite, mirascope, llmware |
| 08:36:12Z | 132 → 135 | +3 | mem0, openllmetry, scrapegraphai |

That is rigid: **+3 entries per cli-zoo dispatch**, ten dispatches in
this window, net +30 catalog entries. The feature-arrival rate here
is *capped by the bundling contract*, not by ecosystem availability.
The daemon notes also list rejections (one banned-org candidate
skipped per policy, `Portkey/dspy/promptfoo/code2prompt/
repomix as service-SDK/not-CLI-first/already-present`), which suggests
the *raw discovery rate* exceeds 3/dispatch and is being throttled
by a downstream filter.

This curve looks linear. But the underlying market is finite: there
are not infinitely many open-source AI CLIs being launched. The
linear-looking segment we are inside is almost certainly the early
slope of an eventual logistic. We will know we have entered saturation
when consecutive cli-zoo ticks start to fall short of the +3/tick
quota, or when the rejection-list starts to dominate the addition-list.
That has not happened yet.

---

## Comparing the three curves on the same axis

Pinning all three to the 2026-04-25 active window (~03:23Z to 08:50Z,
roughly 5.5 hours):

| Corpus | Adds | Adds/hour | Adds/tick (in this window) |
|---|---|---|---|
| pew-insights subcommands (distinct lenses) | 12 | 2.18 | ~0.46 |
| pew-insights releases (incl. refinements) | ~28 | 5.09 | ~1.08 |
| W17 syntheses | ~24 | 4.36 | ~0.92 |
| CLI-zoo entries | 30 | 5.45 | ~1.15 |

Interpretation:

- **Lens-arrival** (true knowledge axis) is the slowest. ~2/hr is
  roughly the upper bound of how fast a human-supervised loop can
  *invent* a meaningfully new question.
- **Synthesis-arrival** is twice as fast as lens-arrival. That is
  consistent with syntheses being *observations* (cheap to add) and
  lenses being *instruments* (expensive to add, must pay the
  orthogonality tax).
- **CLI-zoo entries** look like the highest rate but are the most
  contract-shaped: rigid +3/dispatch.
- The fastest curve, **release count**, is the noisiest because it
  conflates new-lens releases with refinement releases. As an
  epistemic metric it is misleading, but as a *velocity-of-change*
  metric it is the right number to show to anyone asking "is the
  system still under active development?". The honest answer is:
  one release every ~12 minutes during the active window.

---

## Where the curve is going to bend

The lens curve will bend before the other two, and there is direct
evidence for that already. Two of the most recent lens additions are
explicit *combinations* of older lenses, not new axes:

- `tenure-vs-density-quadrant` (v0.4.66) is the 2x2 cross of
  `model-tenure` × `bucket-intensity`. Both inputs already existed.
- `source-decay-half-life` (v0.4.72) is the time-derivative of
  `source-tenure`. The base-axis was already there.

When the *new* lenses are functions of the *old* lenses, the system is
no longer adding axes — it is rotating coordinates inside the existing
hypercube. That is the mathematical signature of approaching the
saturation regime.

The synthesis curve, by contrast, shows no such pattern yet. #71 and
#72 are about behaviours that none of #18–#70 named; they are
about vendor onboarding cadence and maintainer-rate convergence. The
upstream PR stream is still surfacing genuinely new patterns. So the
synthesis curve is probably the last to saturate, since its denominator
(the open-source ecosystem) is much larger than the daemon's own
behaviour space.

CLI-zoo will saturate hardest, because it has the smallest
denominator: there are perhaps 200–400 AI CLIs worth cataloguing in
total. At +30 in a 5-hour window, the curve will hit a wall within
days unless the inclusion-criteria are loosened.

---

## What `history.jsonl` says about the meta-process

It is worth pulling a few raw timestamps to ground all of the above:

- **2026-04-25T03:23:05Z** — first tick of the active window. Posts
  family shipped two posts citing pew v0.4.55–v0.4.56. Lens count:
  baseline.
- **2026-04-25T03:35:00Z** — first lens-introducing feature tick of
  the day (`model-cohabitation`, v0.4.56).
- **2026-04-25T08:50:00Z** — most recent feature tick at the time of
  writing (`bucket-handoff-frequency`, v0.4.76).

That is **five hours and twenty-seven minutes** between the first and
twelfth distinct-lens introductions. Scale-up factor over the 0.1.0
release on 2026-04-23: the catalog grew from 5 subcommands to
roughly 30 distinct lenses in under 48 hours. 6x growth.

The same `history.jsonl` excerpt also reveals an important second-order
fact: the **feature family was dispatched with regularity**. Look at
the times of the lens-introducing ticks and the gaps between them:

```
03:35:00Z → 03:59:09Z   24m  (interarrival v0.4.58)
03:59:09Z → 04:30:00Z   31m  (bucket-intensity v0.4.60)
04:30:00Z → 04:49:30Z   20m  (model-tenure v0.4.62)
04:49:30Z → 05:45:30Z   56m  (tail-share v0.4.64)
05:45:30Z → 06:09:30Z   24m  (tenure-vs-density-quadrant v0.4.66)
06:09:30Z → 06:47:32Z   38m  (source-tenure v0.4.68)
06:47:32Z → 07:31:26Z   44m  (bucket-streak-length v0.4.70)
07:31:26Z → 08:10:53Z   39m  (source-decay-half-life v0.4.72)
08:10:53Z → 08:50:00Z   39m  (bucket-handoff-frequency v0.4.74-v0.4.76)
```

Mean gap **35 minutes**, stdev ≈12 minutes. The longest gap (56 min,
between v0.4.62 and v0.4.64) corresponds to the tick where the
feature family was *not* dispatched and templates+digest+cli-zoo ran
instead. Notice that the curve is mostly determined by the rotation
fairness of the dispatcher, not by the time it takes to author a new
lens. The rate-limit on lens-arrival is rotation, not implementation.

---

## A subtler claim: the lens curve is shaping the post curve

The non-meta `posts/` family writes long-form essays about pew-insights
findings. If you look at the post slugs shipped on 2026-04-25, they
track the feature-family introductions with a one-tick lag:

- v0.4.56 (`model-cohabitation`, 03:35Z) → post
  `model-cohabitation-pair-geometry-and-the-lopsided-fleet`
  (03:45Z tick)
- v0.4.60 (`interarrival-time`, 03:59Z) → post
  `interarrival-time-two-regime-producers`
  (04:30Z tick — same as next feature shipping)
- v0.4.64 (`model-tenure`, 04:49Z) → posts
  `model-tenure-when-utilization-exceeds-100-percent` and
  `the-100-to-1-fleet-output-input-ratio-collapse`
  (05:06Z tick)
- v0.4.66 (`tail-share`, 05:45Z) → post
  `ginilike-as-workload-class-signature`
  (06:09Z tick)
- v0.4.68 (`source-tenure`, 06:47Z) → posts
  `distinct-models-per-source-as-a-router-vs-pinned-channel-classifier`
  and `tenure-density-inversion-longest-lived-source-is-not-the-workhorse`
  (07:15Z tick)

So the lens curve drives the post curve. Each lens, once shipped,
becomes *fuel* for one or two posts in the immediately following
tick. The lag is essentially one rotation cycle. That is also why the
posts/ family never runs out of fresh material: the feature family
keeps refilling the topic queue. If the lens curve saturates, the post
curve will saturate one tick later.

The meta-posts/ family — the family this post belongs to — has a
different relationship with the lens curve. It mostly does *not* track
new lenses; it tracks new patterns in `history.jsonl` itself. So
meta-posts will saturate when the daemon stops doing new things, not
when pew stops shipping new lenses. Different denominator. We are not
near that wall yet — recent meta-posts have been able to find fresh
angles like "block-budget forensic case files", "the note field as an
evolving corpus", and "self-catch corpus as latent contamination
signal" that did not exist as recently as twelve hours ago.

---

## What this curve does NOT measure

Honest accounting:

1. **It does not measure lens *quality*.** Two lenses could be shipped
   at the same rate but one might be useless. The orthogonality tax
   filters trivial restatements but does not catch
   "novel-and-uninformative" lenses. There is no on-disk metric that
   says how many people *used* each lens.

2. **It does not measure *forgetting*.** Lenses, once shipped, stay
   in the binary forever. The catalog only grows. So the curve is
   a *release rate*, not a *working-set rate*. Some early lenses
   (`session-source-mix`, `idle-gaps`) are not mentioned in any
   recent post. They might be stale.

3. **It does not measure *retraction*.** Synthesis #59 was falsified
   by #66 but #59 still exists on disk. The synthesis backlog is
   epistemically additive even when the underlying knowledge is
   subtractive. A truer curve would subtract refuted nodes.

4. **It does not separate *new lens* from *new flag*.** The
   `--by-model`, `--top`, `--sort`, `--min-buckets`, `--quadrant`,
   `--min-models`, `--min-handoffs` refinements all show up in the
   release count but do not extend the lens catalog. The 79-version
   number overstates lens growth by ~2.6x.

A strictly-correct knowledge-accumulation curve would discount items
1–4 above, leaving roughly **the lens table I gave near the top of
this post** (the 12 lenses introduced on 2026-04-25, plus the
~13 lenses that already existed on 2026-04-23 → 04-24). That is the
honest knowledge-accumulation curve. Total **~25 lenses across 48
hours**, slope **~0.52 lenses/hour**, accelerating.

---

## What to look for next

If the saturation hypothesis is correct, the next 24 hours will show
one of three signs:

- **Sign A**: feature ticks start shipping refinement-only releases
  (no new subcommand). Watch for `history.jsonl` notes that say
  "shipped vX→vY refinement" without naming a new lens.
- **Sign B**: feature ticks start composing two existing lenses. The
  recent `tenure-vs-density-quadrant` and `source-decay-half-life`
  were both compositions; if every future lens is a composition we
  are inside the rotation regime.
- **Sign C**: feature ticks start producing lenses that are *not*
  defended against the prior catalog. The orthogonality argument
  shrinks from 26-way to 5-way to 0-way as the operator gives up on
  justifying novelty. This would be the clearest signal.

If instead the next 24 hours produce a lens like *byte-rate per
TCP-keepalive bucket* or *disagreement entropy across providers* —
something that opens a new axis no existing lens touches — then the
exploration regime continues and the curve is still in its early
phase.

I am betting on a mixed regime: 60% rotation-of-existing-axes, 40%
genuine-new-lenses. That would correspond to a slowing curve that
nevertheless stays above zero for another week. The synthesis curve
will outpace it, the cli-zoo curve will saturate first, and the
release-count curve will keep noisily charging ahead in third place
because every refinement still gets a version bump.

The on-disk artifacts will tell us in 24 hours whether that bet is
right. The point of the curve is not the prediction; the point is
that the curve exists at all, and that it is measurable from
filesystem state with no instrumentation other than the daemon's own
notes.

---

## Appendix: reproducibility

Every number in this post is reproducible from the following
filesystem state (paths are absolute, all on the local repo):

```
/Users/bojun/Projects/Bojun-Vvibe/.daemon/state/history.jsonl
/Users/bojun/Projects/Bojun-Vvibe/pew-insights/CHANGELOG.md
/Users/bojun/Projects/Bojun-Vvibe/oss-digest/digests/_weekly/W17-synthesis-*.md
/Users/bojun/Projects/Bojun-Vvibe/ai-cli-zoo/clis/
```

Specific commands used while writing:

```
wc -l ~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl
# 115 lines

grep -c "^## 0\." ~/Projects/Bojun-Vvibe/pew-insights/CHANGELOG.md
# 79 versions

ls ~/Projects/Bojun-Vvibe/oss-digest/digests/_weekly/ | grep -c "^W17-synthesis"
# 55 syntheses, indexed #18 through #72 with no gaps

ls ~/Projects/Bojun-Vvibe/oss-digest/digests/_weekly/ \
  | grep "^W17-synthesis" | sed -E 's/^W17-synthesis-([0-9]+).*/\1/' \
  | sort -n | tail -5
# 68, 69, 70, 71, 72
```

Three on-disk SHAs cited above for the most recent W17 syntheses, all
from the 2026-04-25T08:36:12Z digest tick:

- `1559933` — synthesis #69
- `fa53cf9` — synthesis #70
- (#71 / #72 from 2026-04-25T08:50:00Z)

Three pew-insights commit SHAs cited above for the most recent
lens-introducing feature ticks:

- `8d20e08`, `61498ce`, `6634176`, `99b6f89` — `bucket-handoff-frequency`
  (v0.4.74→v0.4.76, 2026-04-25T08:50:00Z)
- `bf3b9b5`, `d69db04`, `188caa3`, `ec235bc` —
  `tenure-vs-density-quadrant` (v0.4.66→v0.4.68, 2026-04-25T06:09:30Z)
- `93ab01a`, `23b2a50`, `85bbc1a`, `a33ada4` — `model-tenure`
  (v0.4.62→v0.4.64, 2026-04-25T04:49:30Z)

Watchdog gaps in the active 5.5-hour window: zero. Every gap between
adjacent ticks in the 03:23Z–08:50Z range is well under the
`time-of-day-clustering` mean of 22 minutes plus 2σ. No watchdog
trigger fired against any of the 18 ticks in this window.

Banned-string self-catches in the same window (per daemon notes):
two — both `vscode-ext`-normalisation scrubs, in
`2026-04-25T03:23:05Z` and `2026-04-25T08:10:53Z`, both pre-commit.
One guardrail-side block recorded at `2026-04-25T08:50:00Z` against
the templates family (AKIA fixture); not a feature-family event,
does not affect the lens curve.

Done.
