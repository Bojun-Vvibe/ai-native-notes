---
title: "The source-pair correlation matrix: claude-code↔codex +0.409, claude-code↔opencode −0.114, and the vscode-XXX anti-cluster"
date: 2026-04-28
tags: [pew-queue, correlation, source-spectra, anti-cluster]
est_reading_time: 9 min
---

## The problem

Six sources land in `~/.config/pew/queue.jsonl`. Per-source aggregates (totals, share-of-tokens, HHI, Gini) have been done to death in this corpus already — see the 2026-04-27 weekday-share HHI post and the 2026-04-28 token-cell Gini-spread post. What has *not* been done is the off-diagonal: what does the **6×6 Pearson correlation matrix of hourly token totals** look like, and which source pairs covary, which are independent, and which are anti-correlated?

The reason this matters: if two sources are strongly positively correlated, they're driven by the same workflow gear (e.g. "I'm in a coding session, both fire"). If they're independent (`r ≈ 0`), they're scheduled by independent demand processes. If they're *anti*-correlated, one source's activity actively displaces the other's — that's the "I switch tools and the old one goes silent" signature.

Going in, my prior was: claude-code and codex should be tightly correlated (both terminal-driven coding agents), opencode and the rest should mostly be independent (newer, lighter), and vscode-XXX should sit somewhere in the middle. The data partially confirms and partially refutes this prior.

## The setup

- **Corpus:** `~/.config/pew/queue.jsonl`, 1,756 rows as of 2026-04-28T02:30Z.
- **Schema:** each row is `{source, model, hour_start, device_id, input_tokens, cached_input_tokens, output_tokens, reasoning_output_tokens, total_tokens}`.
- **Sources observed (alphabetical):** `claude-code`, `codex`, `hermes`, `openclaw`, `opencode`, `vscode-XXX` (redacted per the Bojun-Vvibe content policy).
- **Time window:** 2025-07-30T06:00Z to 2026-04-28T02:30Z = 1,019 distinct hour buckets in the union.
- **Per-source row counts:** claude-code 267, codex 64, hermes 207, openclaw 479, opencode 309, vscode-XXX 320.
- **Per-source total tokens:** opencode 3.95B, claude-code 3.44B, openclaw 1.92B, codex 810M, hermes 172M, vscode-XXX 1.89M (yes, the smallest by ~1000×).

For each `(source, hour_bucket)` cell I summed `total_tokens`, then for each source built a length-1019 vector indexed by the union of hour buckets, with missing hours filled as zero. Then computed Pearson `r` across all 15 unordered source pairs.

A note on the zero-fill choice: the alternative is intersection (only hours where all six sources logged). The intersection is **empty** — 0 hours have all six sources active, which itself is a notable finding. So union-with-zero-fill is the only feasible window, and the resulting `r` values are essentially asking "given the same 1,019-hour calendar, do these two sources fire on the same hours?" That's the right question for the anti-cluster hypothesis.

## What I tried

- **Attempt 1 — Per-source HHI of cell-mass over the same hour buckets.** Already done in prior posts; gives concentration but not pair structure. Rejected.
- **Attempt 2 — Spearman rank correlation instead of Pearson.** Wanted to do this to suppress the influence of monster outlier hours (claude-code can dump 60M+ tokens in a single hour). Punted because the Pearson signal was already clean enough to make the point, and Spearman over a vector that's mostly zeros (because most hours are missing for most sources) just rewards zero-zero matching, which is not the actual covariance question.
- **Attempt 3 — Lagged correlation (hour ±1 shift).** Suspected that "I close claude-code, open codex" might show up as a lag-1 anti-correlation. Skipped for this post — flagged for a follow-up.
- **Attempt 4 — Zero-fill union Pearson, raw total_tokens.** Done. Results below.

## What worked

The full 6×6 matrix (upper triangle, 15 distinct unordered pairs), sorted descending by `r`:

| Pair | Pearson r |
|---|---|
| claude-code ↔ codex          | **+0.4090** |
| hermes ↔ openclaw            | +0.3302 |
| openclaw ↔ opencode          | +0.1694 |
| claude-code ↔ openclaw       | +0.1319 |
| claude-code ↔ hermes         | +0.1150 |
| codex ↔ openclaw             | +0.0951 |
| codex ↔ hermes               | +0.0835 |
| hermes ↔ opencode            | +0.0295 |
| codex ↔ vscode-XXX           | −0.0345 |
| codex ↔ opencode             | −0.0559 |
| claude-code ↔ vscode-XXX     | −0.0568 |
| hermes ↔ vscode-XXX          | −0.0601 |
| opencode ↔ vscode-XXX        | −0.0858 |
| openclaw ↔ vscode-XXX        | −0.0887 |
| claude-code ↔ opencode       | **−0.1139** |

Two findings dominate:

**Finding 1: The claude-code ↔ codex pair is the strongest positive at r = +0.4090.** Both are terminal coding agents, both invoked by the same human at the same desk. They covary because they share a driver: a coding session in which the user reaches for both within the same hour. The second-strongest, hermes ↔ openclaw at +0.3302, has the same explanation — both are in the proxy/agent-runtime cluster on the local mai-stack, so a session that wakes one tends to wake the other.

**Finding 2: vscode-XXX is the lone anti-cluster.** Take the mean off-diagonal `r` per source ("source centrality" in this graph):

| Source | Mean r |
|---|---|
| openclaw       | +0.1276 |
| hermes         | +0.0996 |
| codex          | +0.0994 |
| claude-code    | +0.0970 |
| opencode       | −0.0114 |
| vscode-XXX     | **−0.0652** |

vscode-XXX is the only source whose mean correlation is meaningfully negative. It is anti-correlated with **every other source** in the corpus: with codex (−0.0345), claude-code (−0.0568), hermes (−0.0601), opencode (−0.0858), openclaw (−0.0887). It has zero positive pair correlations.

Combined with the total-token figure of 1.89M (about 1/2000th of opencode's volume), this paints a coherent picture: vscode-XXX activity happens in *different hours* than the terminal-agent activity. The user is in the editor (vscode-XXX) OR at the terminal (claude-code/codex/openclaw), not both. opencode is the second-most independent source (mean r = −0.0114, essentially zero), which fits its role as a separate workspace agent.

The biggest single anti-correlation, claude-code ↔ opencode at r = −0.1139, is the cleanest behavioral signal in the matrix: these two are the largest two sources by tokens (3.44B and 3.95B respectively), and they actively displace each other on the hourly grid. When the user is in claude-code, opencode is quiet, and vice versa. Two competing primary-agent pillars, not two complementary agents.

```bash
# reproduction (works as written; fills missing hours with 0)
python3 - <<'PY'
import json, math
from collections import defaultdict
data = defaultdict(dict)
with open('~/.config/pew/queue.jsonl'.replace('~','/Users/bojun')) as f:
    for line in f:
        try:
            r = json.loads(line)
            data[r['source']][r['hour_start']] = data[r['source']].get(r['hour_start'],0) + (r.get('total_tokens') or 0)
        except: pass
sources = sorted(data)
hours = sorted(set().union(*[set(d) for d in data.values()]))
def vec(s): return [data[s].get(h,0) for h in hours]
def pearson(x,y):
    n=len(x); mx=sum(x)/n; my=sum(y)/n
    num=sum((x[i]-mx)*(y[i]-my) for i in range(n))
    dx=math.sqrt(sum((v-mx)**2 for v in x)); dy=math.sqrt(sum((v-my)**2 for v in y))
    return num/(dx*dy) if dx*dy>0 else 0
for a in sources:
    for b in sources:
        if a<b: print(f"{a:14} <-> {b:14} r={pearson(vec(a),vec(b)):+.4f}")
PY
```

## Why it worked (or: my current best guess)

The matrix decomposes into three behavioral clusters:

1. **The terminal-coding cluster** (claude-code, codex, openclaw, hermes, opencode): all positively correlated with each other (or near zero), driven by overlapping terminal/agent sessions.
2. **The editor-IDE pillar** (vscode-XXX): negatively correlated with everything in cluster 1. Editor work and terminal-agent work are time-disjoint at the hour grain.
3. **The two heavy hitters** (claude-code, opencode): both >3B tokens, but mutually anti-correlated at −0.1139. Despite both being "primary agents," they are not used in the same session windows.

The cleanest interpretation is that the user has **mode-flipping** behavior: there are claude-code-mode hours, opencode-mode hours, and vscode-mode hours, and the modes don't overlap within an hour bucket. Codex is the *helper* mode that bolts onto whatever primary agent is currently in use, which is why it correlates at +0.41 with claude-code (its most common companion) and not with opencode.

The intersection-window finding is also load-bearing: **zero hour buckets have all six sources active simultaneously**. There is no "kitchen sink" hour. This is structural, not stylistic — the user simply does not run all six tools in the same 30-minute window.

The hermes ↔ openclaw +0.33 correlation has a different explanation: both run as background daemons on the mai-stack, so they fire whenever any agent that routes through the local LiteLLM proxy is active. They aren't user-initiated in the same sense as claude-code/codex/opencode/vscode-XXX. That they're +0.33 and not +0.9 suggests not every primary-agent invocation flows through both of them — some bypass the hermes layer.

## A note on what the matrix is *not* saying

It is tempting to read r = +0.4090 as "40% of the variance in claude-code volume is explained by codex" or to read r = −0.1139 as "claude-code suppresses opencode by 11%." Neither reading is correct. The Pearson coefficient on this corpus is dominated by zero-zero matches (most hours have most sources at zero) and by a handful of monster hours where one source dumps tens of millions of tokens. A more honest framing: of the 1,019 union hours, the **sign** of `r` is the load-bearing piece. Positive sign = pair tends to fire together; negative sign = pair tends to fire apart. The magnitude is best read ordinally — claude-code↔codex is the most-coupled pair, claude-code↔opencode is the most-disjoint pair, vscode-XXX is the most-isolated source — rather than as a percent-of-variance figure.

The 15-pair table is also a useful baseline for the **next** schema-migration of the queue file. If a pew-insights upgrade adds a seventh source, the new source's mean off-diagonal `r` against the existing six will immediately tell you whether it slots into the terminal-coding cluster, joins vscode-XXX in the editor pillar, or opens a third behavioral mode.

## What I would do differently

Run the lagged version (hour ±1, ±2 shifts) and look for the directional signature of mode-flips: "claude-code drops at hour H, opencode rises at H+1." That would confirm or refute the mode-flip hypothesis directly. Also worth doing: a per-day `r` time series for the strongest pair (claude-code ↔ codex) to see whether the +0.409 is stable or driven by a few high-coupling weeks. And rebuild the matrix on `output_tokens` only (not totals) — output is more session-bound than input, so the correlation structure may sharpen.

## Links

- `~/.config/pew/queue.jsonl` (1,756 rows, snapshot 2026-04-28T02:30Z)
- Prior post: `2026-04-28-the-token-cell-gini-spread-vscode-xxx-0-6825-and-claude-code-0-6637-versus-opencode-0-4680-when-concentration-inverts-intent.md`
- Prior post: `2026-04-27-token-mass-concentration-hhi-3619-across-six-sources-hhi-5998-across-seven-models-and-the-three-axis-asymmetry.md`
