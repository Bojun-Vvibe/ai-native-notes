# The 23 model strings tracking five real models, and the March-20 / April-17 twin cliff of frontier rollover

A model-version churn timeline drawn from `~/.config/pew/queue.jsonl` (1,718 hourly aggregation rows spanning 2025-07-30 to 2026-04-27). I have written before about model-name fragmentation as a static count; this post is about the *time axis* — when each label was born, when it died, and what the shape of the per-day model-set tells us about how a single user actually adopts and abandons frontier checkpoints.

## The headline

Across nine months of token telemetry I see **23 distinct model-string values** in the `model` column. After collapsing aliases (`opus` → `claude-opus-4.7`, `claude-opus-4-7` → `claude-opus-4.7`, `gh-assist/claude-opus-4.7` → `claude-opus-4.7`, `gh-assist/claude-haiku-4.5` → `claude-haiku-4.5`, etc.) those 23 strings reduce to roughly **five real frontier checkpoints I actually exercised** at meaningful volume:

- `gpt-5.x` family (5, 5.1, 5.2, 5.4, 5-nano) → 770 rows
- `claude-opus-4.x` family (4.6, 4.6-1m, 4-7, 4.7, plus the `opus` alias and the two `gh-assist/...` aliases) → 681 rows
- `claude-sonnet-4.x` family (4, 4.5, 4.6, 4-6, plus the `gh-assist-chat/claude-sonnet-4` alias) → 72 rows
- `claude-haiku-4.5` family (`claude-haiku-4-5-20251001`, `claude-haiku-4.5`, `gh-assist/claude-haiku-4.5`) → 31 rows
- `gemini-3-pro-preview` → 37 rows

Plus one *experimental* checkpoint that lived for **exactly three days** and then vanished: `big-pickle`, 53 rows total, first seen `2026-04-22`, last seen `2026-04-24`.

That is the dataset. The interesting thing is not the count of strings but the shape of the timeline.

## Two structural cliffs

If you plot first-seen and last-seen per model on a calendar, two cliff days dominate everything else:

**Cliff #1 — 2026-03-20.** First appearance of `gpt-5.4` (eventually the most-used label in the entire dataset at 545 rows). The previous `gpt-5.1` label was last seen `2026-01-27` — meaning there is a **52-day gap** in the GPT-5 timeline where I emitted basically zero GPT-family tokens. The harness side that drives `gpt-5.x` (the `openclaw` source for `gpt-5.4`, the `vscode-assist` source for `gpt-5` / `gpt-5.1`) had a real downtime, not just a label rename. `gpt-5.4` did not slot into where `gpt-5.1` left off; a different harness picked it up.

**Cliff #2 — 2026-04-17.** Same-day handoff between `claude-opus-4.6-1m` (167 rows total, last seen `2026-04-17`) and `claude-opus-4.7` (488 rows total, first seen `2026-04-17`). On that single day five distinct model strings co-exist in the dataset: `claude-opus-4-7`, `claude-opus-4.6-1m`, `claude-opus-4.7`, `gpt-5.4`, and the bare alias `opus`. That five-way coexistence is the highest model-diversity-per-day count anywhere in the timeline except for `2026-04-23` (which I will return to).

These two cliffs are not symmetric. Cliff #1 is a *gap-and-restart*: GPT-5.1 ends, fifty-two days of nothing, then GPT-5.4 starts. Cliff #2 is a *crossfade*: 4.6-1m is still emitting tokens on the same calendar day that 4.7 first appears. The reason is upstream-policy: 4.6-1m and 4.7 are both Opus checkpoints I had legitimate reasons to use simultaneously (one for long-context workloads, one for frontier reasoning), so my routing kept both alive for one day before settling on 4.7. GPT-5.1 → 5.4 was not that — there is no overlap, no gradient, just a hard restart.

## The per-day model-set is a five-element ceiling

I compute, for each calendar day in the data, the size of the set of distinct model strings that emitted at least one row. Last fourteen days:

```
2026-04-14: 2 models  (claude-opus-4.6-1m, gpt-5.4)
2026-04-15: 5 models  (claude-haiku-4.5, claude-opus-4.6-1m,
                       gh-assist/claude-haiku-4.5,
                       gh-assist/claude-opus-4.6-1m, gpt-5.4)
2026-04-16: 4 models
2026-04-17: 5 models  (the cliff)
2026-04-18: 3 models
2026-04-19: 3 models
2026-04-20: 5 models
2026-04-21: 5 models
2026-04-22: 3 models  (big-pickle debut)
2026-04-23: 6 models  (big-pickle, claude-opus-4.7,
                       claude-sonnet-4.6, gpt-5-nano,
                       gpt-5.2, gpt-5.4)
2026-04-24: 3 models
2026-04-25: 2 models
2026-04-26: 2 models
2026-04-27: 2 models  (claude-opus-4.7, gpt-5.4)
```

The structure is clear: on a normal day I run **2 models** (one Opus, one GPT). On the days where I am *exercising* something — testing a new harness alias, trying a preview, or being routed by an upstream provider that decides to canary me onto a different label — the count spikes to 5 or 6. The dispersion is roughly bimodal: 2 (steady-state) versus 5–6 (testing days).

The 2026-04-23 spike at 6 models is the most diverse day in the dataset, and inspecting it row-by-row, four of those six models are *single-day-only* labels that never reappear: `gpt-5-nano` (1 row), `gpt-5.2` (1 row), `claude-sonnet-4.6` is a longer-lived label but its appearance on that day is the last time it ever shows up, and `big-pickle` is in the middle of its three-day window. The day looks like a deliberate canary: I was routed onto five different upstream variants in a short span and almost none of them ever returned. That implies my own usage was steady — the diversity came from upstream rotating me through canaries, not from me actively model-shopping.

## Single-day-only labels: the canary corpus

Six model strings appear on **exactly one day** in the entire dataset:

```
gh-assist/claude-haiku-4.5   2026-04-15
claude-sonnet-4-6                 2026-02-25
gpt-4.1                           2025-08-22
opus                              2026-04-17
gpt-5.2                           2026-04-23
gpt-5-nano                        2026-04-23
```

Two patterns jump out. First, the `claude-sonnet-4-6` (with a hyphen) versus `claude-sonnet-4.6` (with a dot) split: one row total uses the hyphen form, six rows use the dot form. That is a **server-side string-format slip** — an upstream node, possibly on a different deploy, briefly emitted the hyphenated variant before the canonical dot form took over. There is no ambiguity about which model it is; it is literally a typographical artifact of which gateway version stamped the row.

Second, `gpt-5.2` and `gpt-5-nano` both appear on 2026-04-23 and never again. `gpt-5.2` is plausible as an intermediate frontier label that got re-routed back to 5.4. `gpt-5-nano` looks like a deliberate canary on a small-tier model — perhaps to test the hosted price tier — that was never retried. Either way, they are operationally dead labels in my log.

The bare `opus` alias on 2026-04-17 is the most diagnostic of the six. That is the same day as Cliff #2. Three rows emerged with the model field literally `"opus"` and no version. That is the kind of stamp a *fallback* code path emits when the upstream dispatcher cannot decide which Opus checkpoint to charge against and just writes the family name. The fact that `opus` appears only on the cliff day is consistent with: the upstream gateway was mid-rollover for a few hours and the row-stamping logic took a degenerate path while the routing settled.

## Per-source model diversity

The source-wise diversity is bimodal in a different way. Using the per-source model histogram:

```
openclaw:        1 distinct model (gpt-5.4 only)        466 rows
codex:           1 distinct model (gpt-5.4 only)         64 rows
hermes:          4 distinct models (top: opus 4.7)      ~196 rows
opencode:        6 distinct models (top: opus 4.7)      ~360 rows
claude-code:     9 distinct models (top: opus 4.6-1m)   ~299 rows
vscode-assist: 10 distinct models (top: gpt-5)         ~333 rows
```

Two of the six sources are **monogamous** — `openclaw` and `codex` only ever emit one model string in the dataset (`gpt-5.4`), even though they account for 530 of the 1,718 total rows. The other four sources are **polygamous** but with a long tail: `vscode-assist` shows ten distinct strings, but seven of them have fewer than 20 rows each. The diversity is not "ten different models in active use," it is "one or two real models plus eight aliases / canaries / experiments."

That asymmetry has an operational explanation. `openclaw` is a fixed harness pinned to one upstream model id; it does not fan out across model variants. `vscode-assist` is a UI surface that exposes a model picker, so I have personally flipped through more options inside it than I have inside the harness-pinned sources. The diversity in the data is *me using the picker*, not the harness deciding to fan out.

## What "model" actually means here

The string in the `model` column is whatever the **emitting client decided to stamp** at the time it wrote the row. That is the source of all the alias noise. There is no canonical model registry that all six sources agree on. Concretely:

- Some clients stamp the version with a dot (`claude-opus-4.7`).
- Some stamp it with a hyphen (`claude-opus-4-7`).
- Some prefix the gateway namespace (`gh-assist/claude-opus-4.7`).
- Some include the precise minor revision (`claude-haiku-4-5-20251001`).
- Some collapse to family (`opus`).
- Some embed a UI string with spaces and capitalization (`gh-assist/claude-haiku-4.5`).

If I ever want to do real cohort analysis on "what did Opus 4.7 cost me last week," the alias-collapsing has to happen *before* aggregation. The current 23-string split makes the raw column unreliable as a join key. The fix is a normalization layer at ingest, but I have not built one yet — the cost of doing so was not justified until I started doing time-cohort comparisons across the 4.6-1m → 4.7 cliff. Now it might be.

## The big-pickle window

`big-pickle` is the most curious string in the dataset. It exists for **three calendar days** (2026-04-22 → 2026-04-24), accounts for **53 rows**, and originates exclusively from the `opencode` source. The string `big-pickle` is not a model name in any model registry I know of — it is a code-name. The fact that it landed in the `model` column means an upstream gateway was reporting itself with the code-name during a brief window, presumably while the real model identity was still under wraps.

The 53-row window is enough to compute averages but not enough to do tail analysis. What I *can* say is that during the three-day big-pickle window the `opencode` source spread itself across three models simultaneously (`big-pickle`, `claude-opus-4.7`, `gpt-5.4`) — meaning my routing was actively splitting traffic, not migrated. This is the only window where `opencode` showed three-way model diversity; before and after, it is dominated by `claude-opus-4.7` alone.

Three days, 53 rows, then gone. That is the operational footprint of an A/B canary surfacing in my telemetry. Whatever `big-pickle` was, it was either renamed to one of the existing labels (most likely a `claude-opus-4.7` minor revision) or rolled back. Either way, the only evidence I have that it ever existed is in this dataset.

## What this changes for me

Three downstream actions fall out of this timeline analysis:

1. **Alias normalization is now justified.** With 23 strings collapsing to roughly five real models, any cross-day cohort comparison is silently miscounting unless I collapse at ingest. The fragmentation tax post quantified the problem statically; this post quantifies it temporally. The 4.6-1m → 4.7 same-day handoff is the worst case: a naive join key splits one continuous Opus usage history into two cohorts.

2. **The cliff days are diagnostic anchors.** 2026-03-20 and 2026-04-17 are now permanent reference points. Any per-day metric I compute should be sliced relative to those cliffs; mixing pre-cliff and post-cliff data in a single mean is a mistake. The 52-day GPT gap also means any "monthly average" computed for GPT-5 family is meaningless without explicitly handling the gap.

3. **Single-day-only labels deserve their own bucket.** Six strings appear on one day each. Treating them as first-class model cohorts inflates the model-count denominator by 26%. The right move is to gate them out of the default per-source model count and report them separately as "one-shot canaries observed."

The structural fingerprint of upstream model rollovers is *visible* in my own telemetry, not because I instrumented for it, but because the model column inherits whatever the upstream gateway happens to be stamping that day. The 23 strings are not 23 models. They are five models plus eighteen artifacts of how the rollover got reported. The artifacts are themselves the data.

## Methodology footnote

All counts above come from `~/.config/pew/queue.jsonl` (1,718 rows, last row hour `2026-04-27T...`). The per-day model set is the set of distinct `model` field values whose `hour_start[:10]` equals that calendar date. The per-source diversity is the set of distinct `model` strings grouped by `source`. No alias collapsing was applied for these counts; the 23 figure is the raw distinct-count.

The pew-insights repo (commit `4574e08` is the latest at the time of writing) does not yet have a model-churn lens. The TKEO, ApEn, spectral-flatness, and spectral-rolloff lenses there all operate on token magnitudes, not on the categorical `model` column. A first-class `model-churn` lens that emits per-day model-set entropy would be a natural next addition; the data structure is already perfectly suited.
