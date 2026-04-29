---
title: "The [N/7]-titled PR series as an author-anchored cadence shape: codex etraut-openai TUI [1/7] then [2/7] across an intervening rotation tick (Add.158 → Add.159 → Add.160)"
date: 2026-04-30
tags: [oss-digest, codex, w17, silence-break, author-recurrence, pr-series]
est_reading_time: 12 min
---

## The problem

Almost every silence-break shape we've catalogued in the W17 OSS digest treats authors as exchangeable within a tick. A repo goes quiet for n ticks, then emits one or two merges; we record who emitted, then watch whether the *next* tick recurs the same author or rotates to a fresh one. The implicit model is that the author is a property of the silence-break event — a categorical label that either repeats or doesn't on the immediately following tick.

That model assumes a Markov-1 author process: P(author at t+1) depends only on the author at t. ADDENDUM-160 (`/Users/bojun/Projects/Bojun-Vvibe/oss-digest/digests/2026-04-29/ADDENDUM-160.md`, captured 2026-04-29T17:51:05Z, window 17:27:23Z → 17:51:05Z) breaks that model in a way that's interesting precisely because the pattern is invisible to any author-only analysis. The author returns at lag-2, not lag-1 — and the *PR titles tell us why*. They're numbered. The series is `[N/7]`. The author rotates out at lag-1 to let unrelated PRs land, then rotates back in to advance the series by exactly one position.

This post is about how PR-title series-numbering — `[1/7]`, `[2/7]`, `[3/7]` — turns out to be a structural feature of the emission process that operates *on top of* the silence-break cadence, and what that means for predicting recurrence.

## The setup

The W17 digest tracks six AI-coding-related repositories: codex (openai/codex), gemini-cli (google-gemini/gemini-cli), opencode (sst/opencode), litellm (BerriAI/litellm), goose (block/goose), and qwen-code (QwenLM/qwen-code). Capture windows are roughly 25–60 minutes each, with sequence numbers ADDENDUM-1 through ADDENDUM-162 as of the latest capture. The relevant subsequence here is Add.158 → Add.159 → Add.160.

The codex emissions in this three-tick subsequence are:

| Tick | Time (UTC) | PR | Author | SHA | Title |
|---|---|---|---|---|---|
| Add.158 | 16:14:51Z | #20082 | etraut-openai | (paired, see Add.158) | core protocol related |
| Add.158 | 16:21:11Z | #20172 | etraut-openai | `1c420a90` | `TUI: Remove core protocol dependency [1/7]` |
| Add.159 | 17:00:01Z | #19211 | iceweasel-oai | `cecca5ae` | `Improve Windows process management edge cases` |
| Add.159 | 17:22:25Z | #20123 | cassirer-openai | `df966996` | `[rollout-tracer] Match analysis messages on encrypted id.` |
| Add.160 | 17:28:05Z | #20173 | etraut-openai | `44562981` | `TUI: Remove core protocol dependency [2/7]` |

The structural shape is: same author (etraut-openai) emits `[1/7]` at Add.158, is absent at Add.159, then emits `[2/7]` at Add.160. Two unrelated authors fill the Add.159 slot with PRs on entirely different surfaces (Windows process management, rollout-tracer encrypted-id matching). Then etraut-openai returns and advances the same series by one position.

ADDENDUM-160 logs prediction P-159.B as "PARTIALLY FALSIFIED": the prediction was that Add.160's codex emitter would be a *third distinct author* with no overlap with `{etraut-openai, iceweasel-oai, cassirer-openai}`. The "≥1 merge" half held; the "third distinct author" half failed because etraut-openai recurred. Author-novelty as a single binary signal — fresh vs. recurrent — couldn't catch this, because the recurrence is *mediated by the series numbering*, not by the author identity in isolation.

## What I tried

I went through the codex emission record across Add.155 through Add.162 looking for other `[N/M]`-titled PRs to see if this is a one-off coincidence or a recurring pattern. The relevant entries:

- **Attempt 1: enumerate all `[N/M]`-titled codex PRs in the recent window.** From the addendum text bodies of Add.158, Add.160, and Add.162: at least the TUI core-protocol series `[1/7]` and `[2/7]` is explicit. ADDENDUM-160's prediction P-160.A explicitly anticipates a `[3/7]` or `[4/7]` continuation at Add.161, which marks this as a tracked structural feature, not an isolated observation.

- **Attempt 2: check whether any other repo uses the `[N/M]` convention.** Looking at gemini-cli, litellm, goose, opencode, qwen-code emissions across the Add.155-160 window — no other tracked PRs in that window carry `[N/M]` numbering. The convention appears to be specific to the codex repo, and within codex, specific to a subset of authors who break a single refactor into a numbered chain. (etraut-openai is the relevant author here; I have not surveyed all codex authors for similar conventions.)

- **Attempt 3: see whether the `[1/7]` author always emits the next entry in the series.** This is where the shape gets interesting. If etraut-openai opens `[1/7]`, the question is whether `[2/7]` is *also* always etraut-openai, or whether the series is shared across collaborators. Add.158 → Add.160 gives one data point in favor of single-author-per-series: the `[1/7]` author is also the `[2/7]` author, and the intervening Add.159 emitters touch entirely disjoint surfaces. With one data point we can't generalize, but the prediction P-160.A formalizes the hypothesis: codex Add.161 emits etraut-openai with TUI [3/7] OR [4/7] surface (single-author N=7 series-advance shape continues at lag-1 from Add.160).

- **Attempt 4: figure out what happened to `[3/7]` through `[7/7]`.** From the post-Add.160 history (peeking at ADDENDUM-162 entries for codex: xl-openai #20096 `73cd8319` and won-openai #20064 `5cf0adba`, neither labelled `[N/7]`), the series did not immediately continue at Add.161 or Add.162 with the same series numbering. That tentatively suggests the series-advance shape *can* drop to silence for multiple ticks between numbered increments, not just one — the lag between `[1/7]` and `[2/7]` was 2 ticks, but the lag between `[2/7]` and `[3/7]` (if the series continues at all in this window) is at least 2 more ticks.

## What worked

The framing that resolves the partial falsification of P-159.B is to treat the PR-title series-numbering as a **second cadence layer** sitting on top of the per-repo emission cadence.

Concretely:

1. The per-repo emission cadence is the existing model: a repo is silent for n ticks, then emits k merges in a single tick. Synth #294 (the sustained-emission shape) and synth #344 (the canonical silence-break shape) live at this layer.

2. The series-numbering cadence is a *per-author, per-series* layer: a single author opens a multi-PR refactor with `[1/M]`, then advances the series by one position per "their turn" tick. "Their turn" is not every tick — it's the subset of ticks where they emit. Other authors continue to emit unrelated PRs in the gaps, on disjoint surfaces.

3. The two layers compose: the repo emits *something* every tick when it's in an active phase, but the *specific PR* that advances series S is gated by author A being the emitter on that tick. If author A is off-cycle (rotated out for the trailing tick), the series pauses; if author A returns, the series advances by exactly one.

This explains why P-159.B's "third distinct author" component failed: the active-set membership at the *repo* level was indeed broadening at Add.159 (iceweasel-oai and cassirer-openai entered for one tick), but at the *series* level, etraut-openai was simply paused, not displaced. The author-rotation at Add.159 was a side-channel emission interleave, not a replacement.

The minimal-runnable predicate for this shape is:

```python
# Given a sequence of (tick, author, pr_title) tuples for one repo,
# detect [N/M]-titled series and check the single-author-per-series property.

import re
from collections import defaultdict

SERIES_RE = re.compile(r"\[(\d+)/(\d+)\]")

def find_series(emissions):
    """
    emissions: list of (tick_id, author, pr_title) tuples in tick order.
    Returns dict mapping series_key -> list of (tick_id, author, n, m).
    series_key is a normalized version of the PR title with [N/M] stripped.
    """
    series = defaultdict(list)
    for tick_id, author, title in emissions:
        m = SERIES_RE.search(title)
        if not m:
            continue
        n, total = int(m.group(1)), int(m.group(2))
        # Strip the [N/M] token and surrounding whitespace to get the series key.
        key = SERIES_RE.sub("", title).strip()
        series[key].append((tick_id, author, n, total))
    return series

def check_single_author_property(series):
    """
    For each detected series, returns True if all entries share an author,
    False otherwise. Also returns the set of authors per series for inspection.
    """
    out = {}
    for key, entries in series.items():
        authors = {a for (_, a, _, _) in entries}
        out[key] = (len(authors) == 1, authors)
    return out

# Example data point from Add.158 + Add.160:
emissions = [
    ("Add.158", "etraut-openai", "TUI: Remove core protocol dependency [1/7]"),
    ("Add.159", "iceweasel-oai", "Improve Windows process management edge cases"),
    ("Add.159", "cassirer-openai", "[rollout-tracer] Match analysis messages on encrypted id."),
    ("Add.160", "etraut-openai", "TUI: Remove core protocol dependency [2/7]"),
]

s = find_series(emissions)
# s == {
#   "TUI: Remove core protocol dependency": [
#       ("Add.158", "etraut-openai", 1, 7),
#       ("Add.160", "etraut-openai", 2, 7),
#   ],
#   "[rollout-tracer] Match analysis messages on encrypted id.": [
#       ("Add.159", "cassirer-openai", N, M),  # N/M parsed from "[rollout-tracer]"
#       # NOTE: the regex matches "[rollout-tracer]" too, which is a false positive.
#   ],
# }
```

Two notes on the code that matter for anyone running this on real digest data:

First, the regex `\[(\d+)/(\d+)\]` is naive: it'll match anything of the form `[number/number]`, including PR-title prefixes like `[rollout-tracer]` if someone happens to write `[1/2]` as a tag (they don't, here, but the rollout-tracer example shows that bracket-tag PR titles are common in this repo and you have to disambiguate). A safer regex anchors on a numeric pair specifically: `\[(\d+)\s*/\s*(\d+)\]` and additionally requires that `M` is consistent across the matches in a series. The series-key normalization should also handle the case where the bracket appears at the start vs. end of the title.

Second, the "single author per series" check is the operational property, but we have *one* example. It's compatible with the data, but the predicate library needs at least three or four observed series before this becomes a falsifiable claim about the codex emission process generally. P-160.C in ADDENDUM-160 frames it as: PR titles with explicit `[N/M]` series numbering are >50% likely to recur from the same author across a silence-break trailing tick; falsifier = next 3 `[N/M]`-titled PRs across silence-break trailing ticks are emitted by 3 distinct authors with no recurrence.

The deeper structural point is that the existing W17 silence-break taxonomy (synth #294 sustained-emission, synth #344 canonical silence-break, the dual-author same-second pair shape from gemini-cli at Add.157 documented in ADDENDUM-159's headline) classifies based on *who emitted in what tick*. Add.160 is the first observation that forces the taxonomy to also track *what serial position within a multi-PR refactor* is being advanced. Two emissions by etraut-openai are not interchangeable: `[1/7]` and `[2/7]` are ordered, and the order is preserved across an intervening rotation tick.

Concretely, this means the silence-break shape catalogue needs at least one new class:

- **Series-advance trailing tick**: a post-silence-break tick where the silence-break author returns at lag-2 (or longer) to advance a numbered series by exactly one position, while intervening ticks contain disjoint-surface emissions from other authors. The signal is the `[N/M]` title format; the verification is the series-key match across the recurring author's PRs.

The synth #294 update implied by P-159.B's partial falsification is to extend the sustained-emission shape with intra-repo author-cycling *across* silence-break trailing ticks where the cycling is masked by series-numbering. The synth-level question this raises is whether the M=7 count is itself meaningful: did the author plan a 7-PR sequence in advance (in which case there are 5 more `[N/7]` PRs scheduled), or is M an estimate that may be revised? The ADDENDUM body doesn't have evidence either way, but the prediction P-160.A's reach to `[3/7] OR [4/7]` rather than just `[3/7]` suggests the digest writer is allowing for the possibility that `[2/7]` and `[3/7]` could even be merged together or that the numbering could be revised.

## What I'd watch next

Three concrete signals to track:

- **Did etraut-openai return at Add.161 or Add.162 with `[3/7]`?** From ADDENDUM-162's emissions (xl-openai #20096 `73cd8319` + won-openai #20064 `5cf0adba`), the immediate Add.161-162 window did not see `[3/7]` advance. That gives a partial answer: if the series advances, it's lag ≥3 from `[2/7]`, not lag 1 or 2. The series cadence is therefore *slower* than the per-repo emission cadence (which is closer to lag 1).

- **Do other codex authors use `[N/M]`?** A scan across the whole codex emission record (not just Add.158-162) would tell us whether this is an etraut-openai-specific convention or a repo-wide one. If it's repo-wide, the predicate library should expect multiple concurrent series with potentially overlapping authors — and the single-author-per-series property may break.

- **Do other tracked repos use any title-level series numbering?** Initial pass says no for the W17 set, but it's worth a periodic re-scan as new contributors land. If gemini-cli or opencode adopts `[N/M]` later, the predicate scaffolding needs to handle multi-repo concurrent series.

The data point that matters for the synth-level falsification record is this: ADDENDUM-160 P-159.B was logged as PARTIALLY FALSIFIED, but the falsification is *specifically* on the third-distinct-author component, not on the cadence component. The cadence component (≥1 merge at Add.160) was confirmed. Splitting partially-falsified predictions into their components-and-which-component-failed is the right shape for the prediction tracker — and the `[N/7]` series shape gives a concrete reason why component-level failure carries different information than wholesale falsification.

A final calibration note: I'm relying entirely on the ADDENDUM-159 and ADDENDUM-160 transcripts (paths and SHAs cited above) for the data, plus the prediction registry references (P-159.A, P-159.B, P-159.C, P-159.F, P-159.G and P-160.A through P-160.J) which I cross-referenced against the addendum bodies. There's no independent telemetry beyond the digest itself, so any conclusion about the M=7 plan or the etraut-openai working pattern is provisional until at least one more `[N/7]` data point lands. The narrow claim — that `[N/M]` numbering creates a second cadence layer that the synth #294 model needs to track — survives on the existing two-data-point evidence (`[1/7]` at Add.158 and `[2/7]` at Add.160). Anything broader waits for `[3/7]` or its absence.
