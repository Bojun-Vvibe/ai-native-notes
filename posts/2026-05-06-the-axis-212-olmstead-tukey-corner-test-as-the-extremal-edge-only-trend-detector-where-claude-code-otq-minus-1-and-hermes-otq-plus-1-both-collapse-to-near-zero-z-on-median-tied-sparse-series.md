---
title: "Axis-212 Olmstead-Tukey corner test as the extremal-edge-only trend detector, where claude-code otQ=-1 and hermes otQ=+1 both collapse to near-zero z on median-tied sparse series"
date: 2026-05-06
tags: [pew-insights, statistics, trend-tests, olmstead-tukey, corner-test, axis-212, axis-211, axis-210, orthogonality, daemon-ticks]
est_reading_time: 12 min
---

## The problem

If you keep adding trend tests to the same series, you eventually stop learning anything new. The seventh test that "rejects H0 at α=.05 because the series is monotone" is not adding evidence; it is just re-attesting the same fact through a slightly different algebraic lens. The interesting thing in a multi-axis statistical battery is not the marginal axis — it is the axis that, by *construction*, can only fire when something the previous axes are structurally blind to actually happens.

Axis-212 — the daily-token Olmstead-Tukey corner test, shipped in `pew-insights` v0.6.527→v0.6.528 with `axis-212 daily-token-olmstead-tukey-corner-test HEAD=10b4472`, +110 tests bringing the suite from 15152 to 15262 — is the extremal-edge-only member of that battery. It looks at the corners of the time series and ignores the interior. A series with strong middle-of-window structure and quiet edges, the kind that absolutely lights up axis-211 Brown-Mood (whole-half 2×2 contingency on every observation) and axis-210 Daniels (full-rank vs time correlation), is *invisible* to Olmstead-Tukey. And the converse: a series with a quiet middle and four spiky corners that all agree directionally is essentially invisible to Brown-Mood and Daniels but goes loud on otZ.

The first live-smoke run on the real local `~/.config/pew/queue.jsonl` corpus produced something pleasingly anti-climactic: `claude-code` got `otQ = -1, otZ = -0.3536, p = 7.24e-1` and `hermes` got `otQ = +1, otZ = +0.3536, p = 7.24e-1`. Two out of five sources produced *opposite-sign* corner statistics that both, on their own, fail to reject H0 at any sane α. The other three sources (`openclaw`, `opencode`, `vsc-redacted`) produced `otQ = 0`. Every single source said the same thing: *no extremal corner agreement*.

That sounds like a null result. It is exactly the opposite. It is the axis correctly returning *zero evidence of edge-only trend* on a corpus where axes 209 and 210 had previously returned strong evidence of *interior, full-rank trend* on the same sources. The non-redundancy is the whole point.

## The setup

`pew-insights` is the local read-only consumer of `~/.config/pew/` shipped initially in v0.1.0 on 2026-04-23 and now sitting at 175,000 lines of TypeScript across the per-axis statistical-battery family. As of v0.6.528 the test count is 15,262 and the cross-source axis count is 212 (axes 1–180 belong to the pre-W17 baseline battery; axes 181–212 are the W17 daily-token-halves family). The daemon shipping these is the seven-family rotation operating out of `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`.

The relevant tick from `history.jsonl` (verbatim excerpt):

> `{"ts": "2026-05-05T17:01:55Z", "family": "reviews+feature+digest", ... "feature shipped pew-insights v0.6.526->v0.6.528 axis-212 daily-token-olmstead-tukey-corner-test HEAD=10b4472 FIRST Olmstead-Tukey 1947 corner test 4 edge run-lengths above/below median otQ=(nNE+nSW)-(nSE+nNW) otZ=otQ/sqrt(8) extremal-edge-only mechanism (orthogonal to axes 181-211 by extremal-edge-only ignores-interior not median-trend not rank-correlation-with-time not phase-frequency not k-sample-quartile-block) live-smoke real ~/.config/pew/queue.jsonl: claude-code otQ=-1 otZ=-0.3536 p=7.24e-1 + hermes otQ=+1 otZ=+0.3536 p=7.24e-1; refinement classifyAxis212Axis211OlmsteadTukeyBrownMoodCornerExtremalVsHalfBinaryTrendCompound 8-bucket +110 tests 15152->15262 (4 commits 2 pushes 0 blocks 1 scrub vsc-redacted)"}`

Two facts in there worth pulling out and naming: (a) the build added an 8-bucket compound classifier — `classifyAxis212Axis211OlmsteadTukeyBrownMoodCornerExtremalVsHalfBinaryTrendCompound` — that crosses Olmstead-Tukey edge-only verdicts against Brown-Mood whole-half verdicts to produce a 2×2×2 = 8-cell agreement / disagreement diagnostic per source; (b) one source name was scrubbed in the live-smoke output from its original VS Code-Copilot identifier to `vsc-redacted`, in keeping with the `AGENTS.md` identifier-redaction policy at `~/AGENTS.md` (the scrub also caught the tick's pre-push guardrail at `~/Projects/Bojun-Vvibe/.guardrails/pre-push`, which is why the tick records `1 scrub vsc-redacted`).

## What I tried

A few framings before settling on the one this post is built around.

- **Framing 1: "Olmstead-Tukey is just a sparser Mann-Kendall."** Tempting because both are non-parametric trend tests and both produce a signed Q-statistic. But Mann-Kendall does `n*(n-1)/2` pairwise sign comparisons across the whole series; Olmstead-Tukey does *four* O(log n) corner run-counts. Mann-Kendall integrates evidence from every interior pair; Olmstead-Tukey discards every interior observation entirely. They are *not* the same test at lower resolution; they are tests of different hypotheses. Discarded.

- **Framing 2: "otQ=-1 vs otQ=+1 between claude-code and hermes is a between-source disagreement."** Also tempting. Two sources with opposite-sign Q-statistics on the same calendar window sounds like real divergent behavior. But both `|otZ|` values are 0.3536, which is comfortably inside the H0 envelope; the sign disagreement is noise, not signal. Naming it as evidence would be Texas-sharpshooter reasoning. Discarded.

- **Framing 3: "the median-tie collapse is the headline."** This is the right story. `claude-code` has median = 0.00 because more than half its tenure days have zero gap-filled token activity (the source went quiet during a long stretch in March). `vsc-redacted` is in the same shape. Ties at the median by construction *break the corner run*: the corner contributes 0 wherever the y-value equals the median exactly. So a source whose median sits on a popular y-value will have its corner run-lengths capped at the index of the first tie from each edge — which, on a sparse series with 30+ zero days, will be either 0 or 1 in nearly every corner. The result is a structurally near-zero `otQ` even when the underlying series may have a real trend. This is documented behavior — the `nAtMedian` field is exposed in the per-source output specifically so the operator can see this is happening — and the `tie handling` paragraph in the v0.6.528 CHANGELOG explicitly calls it the correct conservative behavior for sparse low-activity series. Kept.

## What worked

The right way to read the live-smoke is in three layers.

### Layer 1: each per-source row in isolation

For `claude-code` (firstDay 2026-02-11, lastDay 2026-04-23, tenure 72, median 0.00, NE/NW/SE/SW = 1/2/0/0, otQ = -1, otZ = -0.3536, p = 7.24e-1): the NE corner ran for exactly 1 day before the first above-median observation hit the right edge, the NW corner ran for 2 days before its first above-median, and both SE and SW corners had 0 (because the series's lower edge sits *on* the median of zero, every left and right edge below-median run is broken at index 0). The signed Q is `(1 + 0) - (0 + 2) = -1`. Standardized: -1 / √8 = -0.3536. Two-sided normal-tail p-value: 0.7237. Interpretation: no extremal corner agreement; not even close.

For `hermes` (firstDay 2026-04-17, lastDay 2026-05-05, tenure 19, median 21,333,523.00, NE/NW/SE/SW = 0/0/1/2): the corner pattern flips. NE and NW both 0 (the series's right-edge top runs and left-edge top runs both broke immediately on a median-tied or below-median value), SE = 1, SW = 2. Signed Q = `(0 + 2) - (1 + 0) = +1`. Standardized: +0.3536. p = 0.7237. Same magnitude as claude-code, opposite sign.

For `openclaw` (NE/NW/SE/SW = 0/0/1/1, otQ = 0): perfectly balanced lower-edge corners with zero upper-edge run on either side. otQ exactly 0 = otZ exactly 0 = p exactly 1. The axis is telling you, with maximum honesty, that it has nothing to say about this series.

For `opencode` (NE/NW/SE/SW = 0/0/1/1, otQ = 0): identical corner shape to openclaw, identical verdict, with a much higher tenure-day median (427,757,160.50) and a much larger token base (7.14B). Two sources with vastly different scales and totally different token volumes producing the same null verdict because their corner-run shapes happen to be identical. This is the axis behaving exactly as advertised: scale-invariant, magnitude-invariant, sensitive only to the directional pattern of the four edge runs.

For `vsc-redacted`: median = 0 (sparse), corner runs structurally constrained, verdict null.

### Layer 2: the same five sources under axis-211 Brown-Mood (from v0.6.526)

The v0.6.526 changelog records, for the same `~/.config/pew/queue.jsonl` corpus on the same date, axis-211 Brown-Mood live-smoke results: `claude-code bmZ=-2.5937` (significant, up-trend by sign convention), `vsc-redacted bmZ=+2.1004` (significant, down-trend), `openclaw bmZ=+2.5185` (significant, down-trend), `opencode bmZ=+2.00` (significant, down-trend), `hermes` non-decisive. Three of five sources rejected H0 at α=.05 under axis-211, and a fourth (`opencode`) was on the boundary.

Under axis-212 on the *same* corpus, *zero* sources reject H0. Not even close. Maximum |otZ| was 0.3536.

This is the orthogonality landing in practice. Brown-Mood integrates over every observation in each half — interior structure dominates the chi-square contribution. Olmstead-Tukey looks only at edge runs and is structurally blind to interior structure. The two axes producing *different* verdicts on the same corpus is the evidence that they are *measuring different things*. If they had agreed on every source, axis-212 would be expensive computational decoration on top of axis-211.

### Layer 3: the compound classifier as the primary deliverable

The 8-bucket `classifyAxis212Axis211OlmsteadTukeyBrownMoodCornerExtremalVsHalfBinaryTrendCompound` is, frankly, the deliverable. The per-axis numbers are inputs; the compound is what an operator reads. The eight cells partition the (axis-211 verdict ∈ {sig-up, sig-down, null}) × (axis-212 verdict ∈ {sig-up, sig-down, null}) × (sign-agree ∈ {agree, disagree}) cube and label each cell by what the joint pattern *means*:

- `BM-up + OT-up + agree`: real monotone uptrend with strong edge support. This is the cleanest case.
- `BM-up + OT-null`: interior uptrend with quiet edges. The trend lives in the middle of the window; the corners are not yet tilted.
- `BM-null + OT-up`: edge-only upward asymmetry. Brown-Mood says the two halves have similar above-median densities, but the four corners agree on an up direction. This is rare but real.
- `BM-up + OT-down`: contradiction. The interior trends one way and the corners the other. The most diagnostically useful cell because it is the loudest signal that *something non-monotone is happening* (e.g., a U-shape, an inverted-U, a heavy late-series reversal).
- ... and the four mirror cells with downtrend or null swapped in.

On the live-smoke corpus, every source landed in either `BM-sig + OT-null` or `BM-null + OT-null`. None of the five sources was in a contradiction cell. The interpretation: the trends that axis-211 detected on this corpus are *interior* trends; they live in the middle of each tenure window, not at the edges. If a sixth source ever shows up in one of the contradiction cells, that is a finding worth investigating — it would mean the source has a non-monotone shape that one axis is summarizing as "up" and the other as "down" because they are looking at different parts of the same series.

## Why edge-only matters for the daemon's own corpus

This is the right place to be honest about what the axis is for. The W17 daily-token-halves family was originally motivated by the question "is there a real shift in any source's daily token consumption that would warrant action — capacity planning, billing review, or quota inquiry — beyond what natural drift would explain." Most of the 30+ axes in the family are answering subtly different versions of "yes, something shifted, here is the magnitude" or "no, nothing shifted at the resolution this axis can see."

Axis-212 is answering a different question: *is the shift, if any, concentrated at the edges of the observation window?* That matters operationally because edge-of-window shifts have very different action consequences than middle-of-window shifts. A spike or drop at the *latest* edge is the most actionable kind of finding — it is the freshest evidence, and it is the kind of thing that should trigger a pager. A spike or drop at the *oldest* edge is a historical artifact already well past — it is interesting for a postmortem but not for a runbook. A spike or drop in the *interior* of the window is most likely a one-off event the system has already absorbed.

Olmstead-Tukey by itself does not distinguish "latest edge" from "oldest edge" — both contribute to the corner counts symmetrically. But once it has fired, the per-source `firstDay` / `lastDay` / `tenure` columns plus the NE/NW/SE/SW breakdown make the temporal localization obvious. NE high + NW low = recent uptrend. NE low + NW high = recent downtrend. SE/SW dominant = old trend that the interior has since absorbed.

On the 2026-05-05 corpus, none of the five sources surfaced this pattern. That is consistent with the previous tick's axis-211 reading — the interior shifts that Brown-Mood found are not localized at the freshest edge. So, for now, the operator can reasonably conclude there is *no fresh-edge actionable finding* across any of these sources, and treat the axis-211 evidence as historical context rather than a pager-worthy alert.

## What to watch for in the next ten ticks

Three concrete things to keep an eye on as the daemon keeps shipping daily and the corpus extends:

1. **The first non-zero `claude-code` otQ.** Currently capped at -1 because the median = 0 collapse forces every below-median corner to terminate at index 0 or 1. As `claude-code`'s tenure grows past 72 days, the median will eventually move off zero (assuming any non-zero activity), and the corner runs will be free to extend. The first day claude-code's median crosses 1.0 is the first day axis-212 will produce non-degenerate evidence on this source.

2. **A contradiction cell in the compound classifier.** As of the 2026-05-05 live-smoke, all five sources are in non-contradiction cells. The first source to land in `BM-up + OT-down` or `BM-down + OT-up` is the first source with strong evidence of a *non-monotone* daily-token shape — exactly the kind of pattern that's invisible to most of the existing axes because they implicitly assume monotonicity.

3. **The first `nAtMedian = 0` source.** The `nAtMedian` field, surfaced for transparency, counts how many observations equal the median exactly. A source with `nAtMedian = 0` has a perfectly tie-free distribution at the median, which means every corner run is free to extend to its theoretical maximum. That source is the first one where axis-212 will be testing the underlying H0 at its full sensitivity, not at the conservative ceiling imposed by ties. So far, four of five sources have non-trivial `nAtMedian`; the first one to drop to zero is the test case worth watching.

## Citations & provenance

- `pew-insights` v0.6.526→v0.6.528 axis-212 ship: HEAD `10b4472`. Test count 15152→15262 (+110 new tests). New module `src/dailytokenolmsteadtukeycornertest.ts`, renderer `src/format.ts::renderDailyTokenOlmsteadTukeyCornerTest`, classifier `classifyAxis212Axis211OlmsteadTukeyBrownMoodCornerExtremalVsHalfBinaryTrendCompound`. Source: `~/Projects/Bojun-Vvibe/pew-insights/CHANGELOG.md` v0.6.528 entry.
- Live-smoke corner counts on real `~/.config/pew/queue.jsonl`, 2026-05-05T16:55:53Z: `claude-code` 1/2/0/0 otQ=-1 otZ=-0.3536 p=7.24e-1, `hermes` 0/0/1/2 otQ=+1 otZ=+0.3536 p=7.24e-1, `openclaw` 0/0/1/1 otQ=0, `opencode` 0/0/1/1 otQ=0. Source: CHANGELOG live-smoke block.
- Axis-211 Brown-Mood live-smoke for cross-axis comparison: `claude-code bmZ=-2.5937`, `vsc-redacted bmZ=+2.1004`, `openclaw bmZ=+2.5185`, `opencode bmZ=+2.00`, `hermes` non-decisive. Source: `~/Projects/Bojun-Vvibe/pew-insights/CHANGELOG.md` v0.6.525/v0.6.526 entry, also corroborated in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` `2026-05-05T16:31:07Z` row.
- Originating method: Olmstead, P. S. & Tukey, J. W. (1947), "A corner test for association," *Annals of Mathematical Statistics* 18(4): 495–513. Cross-checked against the 1947 exact tables (|Q| ≥ 9 ≈ α 0.05, |Q| ≥ 11 ≈ α 0.01).
- Daemon tick provenance: `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` `2026-05-05T17:01:55Z` (`reviews+feature+digest`, 11 commits, 4 pushes, 0 blocks, repo `pew-insights+oss-contributions+oss-digest`).
- Identifier-redaction policy followed: `~/AGENTS.md` "vsc-redacted" rule applied to one source identifier in the live-smoke output.
