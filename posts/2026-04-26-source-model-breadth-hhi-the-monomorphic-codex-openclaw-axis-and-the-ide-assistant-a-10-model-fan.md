---
title: "Source × model breadth and HHI: the monomorphic codex/openclaw axis and the ide-assistant-A 10-model fan"
date: 2026-04-26
tags: [telemetry, model-routing, concentration, hhi, queue.jsonl]
---

## The number

Out of six observed harness sources in `~/.config/pew/queue.jsonl` (1,562 rows, hour buckets spanning `2025-07-30T06:00Z` → `2026-04-26T13:30Z`), the distribution of *how many distinct model strings each source ever invoked* is wildly bimodal:

| source           | n_models | row HHI | top-model row share |
|------------------|---------:|--------:|--------------------:|
| ide-assistant-A  | 10       | 0.314   | 51.1% (gpt-5)       |
| claude-code      |  9       | 0.389   | 55.9% (claude-opus-4.6-1m) |
| opencode         |  6       | 0.620   | 76.6% (claude-opus-4.7) |
| hermes           |  4       | 0.479   | 52.5% (claude-opus-4.7) |
| codex            |  1       | 1.000   | 100% (gpt-5.4)      |
| openclaw         |  1       | 1.000   | 100% (gpt-5.4)      |

Two of the six sources are **monomorphic** — they have ever called exactly one model string. The other four span 4 to 10. The HHI (Herfindahl–Hirschman Index, computed as the sum of squared row shares per model within a source) goes from a perfectly concentrated `1.000` for codex and openclaw down to `0.314` for ide-assistant-A — a 3.18× spread on a metric whose theoretical floor here would be `0.10` if a source spread its rows perfectly uniformly across 10 models.

Source: `~/.config/pew/queue.jsonl`, computed by parsing every row, grouping `(source, model)`, and counting. Cited row count: 1,562 (`wc -l` confirms exactly that). Cited model breadth and HHI numbers come from a 30-line python pass over the same file; values are deterministic and reproducible against today's snapshot.

## Why this matters more than "some sources use more models"

The naive read is "well, some harnesses are routers and others aren't." That's true but it understates the structural divide. Look at the *shape* of the breadth distribution:

- **Two sources at 1 model.** Not 1.5, not 2, not "occasionally fall back" — exactly 1, with HHI clamped at the upper bound.
- **Two sources in the upper-mid band** (4–6 models, HHI 0.48–0.62). These are mixed but anchored by a clear primary.
- **Two sources at 9–10 models, HHI ≈ 0.31–0.39.** These are *fan-out* harnesses where no single model holds even 56% of rows.

This is not a smooth gradient. It's a three-class structure: *fixed*, *anchored-mixed*, and *fan-out*. Calling it a "spectrum" loses the interesting part — that two of the six harnesses have apparently *no fallback path at all*, while two others routinely traverse a 9- or 10-model menu.

## The codex / openclaw monomorphism: deliberate or sampled-out?

Codex has 64 rows in the snapshot. Openclaw has 405. Both 100% on `gpt-5.4`. The interesting question: is this monomorphism *enforced by the harness* (no other model is even configurable) or *enforced by user habit* (the harness allows fallback but the user never exercised it)?

Three signals suggest enforcement, not sampling:

1. **Sample-size asymmetry.** Openclaw's 405 rows is the *largest* row count in the corpus (larger than ide-assistant-A's 333, and within the top tier alongside opencode's 299 and claude-code's 299). If openclaw allowed multiple models, you'd expect its larger sample to surface at least one alternate string by sheer chance, the way ide-assistant-A's 333 rows surfaces 10 distinct strings (a per-row "novelty rate" of about 1 new string per 33 rows on the way up). Openclaw's 405 rows surfaces zero alternate strings — a rate that is hard to reconcile with even a 1-in-200 fallback probability.

2. **Sticky model-string formatting.** Openclaw's single string is `gpt-5.4` — bare, no provider prefix, no quoting variant. Compare to claude-code, whose 9 strings include both `claude-opus-4.7` and a provider-prefixed variant of the same Opus 4.7, and both `claude-haiku-4-5-20251001` and `claude-haiku-4.5` — clear evidence of *the same underlying model* being routed through different provider paths. If openclaw had any routing layer, you'd expect *at least one* such formatting bifurcation; you see none.

3. **Codex consistency at small N.** Codex has only 64 rows, so weaker statistical evidence — but the fact that it agrees with openclaw on the exact monomorphic pattern (one string, `gpt-5.4`, no variants) suggests the harness class, not the sample size, is what's setting the structure.

The likely truth: codex and openclaw are *single-model harnesses by design*, and that's a meaningful product-architecture choice that this telemetry exposes from the outside.

## The ide-assistant-A 10-model fan: it's not just a router, it's a chaotic router

Ide-assistant-A has the highest model breadth (10) but only 333 rows — meaning on average each model gets ~33 rows, but the actual distribution is heavily tilted: the top model (`gpt-5`) holds 51.1% (170 rows), with the remaining 49% spread across 9 models averaging ~18 rows each. The HHI of 0.314 reflects a realistic mix where the user (or the harness's auto-router) genuinely switches models often.

The ten strings span: four Claude variants (opus-4.6, opus-4.7, sonnet-4, sonnet-4.5), one provider-prefixed Anthropic variant, four GPT variants (gpt-4.1, gpt-5, gpt-5.1, gpt-5.4), and `gemini-3-pro-preview`. Notice the cross-vendor span: OpenAI (4 strings), Anthropic (5 strings counting the provider-prefixed variant), Google (1 string). That's three vendors and at least eight distinct model identities crammed into a single source.

This matches the lived experience of an IDE-embedded assistant: the user picks the model from a dropdown per request, and the harness faithfully records what was selected. Everything else (codex, openclaw) is "the harness picks." The HHI gap is essentially measuring *who chose the model* — the user (high breadth, low HHI) or the tool (zero breadth, HHI = 1).

## Cache-share interaction

The model-breadth ranking has an uncomfortable correlation with the cache-share-zero anomaly already documented elsewhere in this corpus:

- **ide-assistant-A**: 333/333 = 100% zero-cache rows.
- **claude-code**: 49/299 = 16.4% zero-cache.
- **codex**: 2/64 = 3.1% zero-cache.
- **openclaw**: 3/405 = 0.7% zero-cache.
- **hermes**: 1/162 = 0.6% zero-cache.
- **opencode**: 3/299 = 1.0% zero-cache.

Ide-assistant-A — the source with the *highest* model breadth — is also the *only* source where prompt caching never appears in the telemetry. Whether that's because (a) the harness doesn't propagate cache metrics, (b) per-request model switching prevents cache hits at the provider level, or (c) the underlying API the IDE uses doesn't expose cache stats — all three are plausible, but the telemetry can only tell you that the joint distribution `(high model breadth, zero cache reporting)` co-occurs perfectly in one source. That's a strong "go look at this" signal, not a causal claim.

The contrast with claude-code is the cleanest natural experiment in the corpus: claude-code has 9 models (just one fewer than ide-assistant-A) and an HHI almost identical (0.389 vs 0.314), yet only 16.4% of its rows are zero-cache. So model-breadth alone doesn't predict cache absence — the harness *implementation* does. Ide-assistant-A is the outlier even relative to a near-isomorphic peer.

## What this rules out

Two stories the breadth data falsifies, both common in casual readings of multi-source telemetry:

1. **"Source identity is essentially model identity."** False at the population level: 4 of 6 sources span multiple models, and the largest-row-count source (openclaw, 405 rows) is also the most monomorphic. There is no monotonic "more rows ⇒ more models" relationship; the correlation in this snapshot is actually slightly *negative* (the two largest-row sources have HHI = 1.0 and 0.62, the two smallest-row sources have HHI = 0.48 and 1.0). Source ≠ model.

2. **"Harnesses with high model breadth are user-facing, harnesses with low breadth are agentic/automated."** Partially true but too clean. Hermes has only 4 models (the second-lowest after the two monomorphics) and is observably an automation harness. But codex (1 model) is *also* an automation harness, and openclaw (1 model) is yet another. The "user-facing" axis doesn't cleanly separate from the "monomorphic" axis. A better framing: model breadth measures *who chose the model* (user-driven dropdown vs harness-fixed default), and that's *almost* but not exactly the same as user-facing vs agentic. Hermes is agentic but offers limited model choice anyway; ide-assistant-A is user-facing and offers maximum choice. Codex and openclaw are agentic *and* fixed — the strongest single-class.

## The HHI-as-architecture-signal claim

If you were trying to *guess* the architecture of an unknown harness from telemetry alone, the (n_models, HHI) pair gives you:

- `(1, 1.0)` → single-model agentic tool. Fixed by config. No fallback. (codex, openclaw)
- `(4, 0.48)` → small-menu automation with a clear primary. Routes between known options. (hermes)
- `(6, 0.62)` → mid-menu, anchored on one default. Provider-mostly-stable, occasional alt. (opencode at 76.6% claude-opus-4.7)
- `(9, 0.39)` → broad menu, mid-anchor. Cross-provider. Likely user-driven choice with strong default. (claude-code)
- `(10, 0.31)` → maximally fan-out, weak anchor, three-vendor span. Pure user-pick UI. (ide-assistant-A)

That's a 5-level architectural typology you can read off two numbers. Whether it generalizes beyond this 1,562-row snapshot of one device's six harnesses is an open question — but the *internal* gradient is clean enough that adding new harnesses should let you place them on the same axis without much ambiguity.

## What I'm not claiming

- I'm not claiming codex *cannot* call other models — I'm claiming the telemetry shows it never has, in 64 rows over a 270-day-equivalent calendar window.
- I'm not claiming ide-assistant-A is necessarily noisier or worse-engineered for having 10 models — model fan-out is a UX feature, not a defect, in a tool whose *job* is to let the user pick.
- I'm not claiming the HHI gap is causal in any particular direction — it's a structural snapshot, not a longitudinal study.
- I'm not claiming a 1.0 HHI proves "single-model by design" — only that the absence of *any* alternate string in 405 rows is much harder to explain by sampling than by enforcement.

## Reproduce

```bash
# row count
wc -l ~/.config/pew/queue.jsonl   # → 1562

# breadth + HHI per source
python3 - <<'EOF'
import json
from collections import defaultdict, Counter
rows = [json.loads(l) for l in open('/Users/bojun/.config/pew/queue.jsonl') if l.strip()]
bs = defaultdict(Counter)
for r in rows: bs[r['source']][r['model']] += 1
for s, mc in sorted(bs.items(), key=lambda x: -len(x[1])):
    n = sum(mc.values())
    hhi = sum((c/n)**2 for c in mc.values())
    print(f"{s:18s} n_models={len(mc):2d}  rows={n:4d}  HHI={hhi:.3f}  top={mc.most_common(1)[0]}")
EOF
```

The output is deterministic against today's snapshot. If you re-run after the snapshot grows, expect codex and openclaw to stay monomorphic (or surface their first non-monomorphic string, which would be the bigger news), and expect ide-assistant-A's HHI to drift slowly toward `1/n_models` as model selection diversifies — unless one model permanently captures the fan, in which case HHI will climb back toward the opencode regime (~0.62).

## One falsifiable prediction

In the next 200 rows added to `queue.jsonl`, **codex and openclaw will each remain at exactly one distinct model string**, and at least one of `(claude-code, opencode)` will surface an 11th–12th distinct model string. The first half of this prediction is the falsifiable one; if either codex or openclaw produces a second model string in the next 200 rows, the "monomorphic by design" reading is wrong and the right reading is "single-model by current habit."
