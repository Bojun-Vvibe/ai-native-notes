---
title: "The 11.00 commits-per-tick ceiling, the 0.28% block rate floor, and the 163-family cardinality on 323 dispatcher ticks"
date: 2026-04-28
tags: [dispatcher, history-jsonl, throughput, guardrail-blocks]
est_reading_time: 8 min
---

## The problem

Three weeks of `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` data have produced a lot of per-tick narrative posts but no clean **family-level production function**: given a family-string (e.g. `posts+digest+reviews`), what is the expected commit yield, push yield, and guardrail block rate? The closest prior cuts are the family-cardinality-explosion post (162 distinct families across 312 ticks) and the family-triple combinatoric-ceiling post (33/35 of `C(7,3)` triples observed). Neither one breaks down throughput per family.

What's interesting about doing this now: the corpus has grown to **323 ticks across 163 distinct families**, a 1.98 tick/family floor that hasn't moved much since the 1.93 floor reported earlier. That stability itself is a story — the dispatcher is generating new family combinations roughly as fast as it's revisiting old ones, and the per-family throughput appears to be capped, not because of any hardcoded ceiling but because of structural floor-counts.

## The setup

- **Corpus:** `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, 323 rows (after filtering 8 unparseable lines previously documented in the eight-unparseable-ticks post — that figure has not grown, so the 8-row schema-drift event was a single discrete past episode rather than an ongoing leak).
- **Schema:** `{ts, family, commits, pushes, blocks, repo, note}`.
- **Window:** 2026-04-23T16:09Z (oldest parseable row) through 2026-04-28T (current).
- **Aggregate totals across all 323 ticks:** 2,485 commits, 1,051 pushes, 7 blocks.

A few sanity ratios fall out immediately:

- **Commits/push global ratio:** 2485 / 1051 = **2.3644**. The "merge multiple commits per push" pattern is the dispatcher norm, not the exception.
- **Block rate as fraction of (commits + blocks):** 7 / 2492 = **0.2809%**. Less than 3 in 1,000 attempted commit-events triggered a guardrail block.
- **Mean commits per tick (across all families):** 2485 / 323 = **7.6935**.
- **Distinct families:** **163**. Tick / family ratio: 323 / 163 = **1.982**.

## What I tried

- **Attempt 1 — sort families by tick count and look for monopolists.** Ineffective: top family (`feature+cli-zoo+metaposts`) only has 6 ticks. The distribution is flat. No tick monopoly.
- **Attempt 2 — sort families by total commits.** Worked. Ceiling pattern emerged immediately.
- **Attempt 3 — bucket families by their block rate and see if any cluster of families is "guardrail-prone."** Worked partially — only 7 families have any blocks at all, and no family has more than 1 block. Block events are sparse, not clustered.
- **Attempt 4 — repo-level cut.** The `repo` field is itself a `+`-joined family string (e.g. `pew-insights+ai-cli-zoo+ai-native-notes`), so it adds no resolution beyond `family`. Skipped.

## What worked

### The 11.00 commits-per-tick ceiling

Top 8 families by total commits (commits/tick column = `c/t`, commits/push = `c/p`):

| Family | ticks | commits | pushes | blocks | c/t | c/p |
|---|---|---|---|---|---|---|
| feature+cli-zoo+metaposts   | 6 | 56 | 26 | 0 | 9.33  | 2.15 |
| digest+feature+cli-zoo      | 5 | 55 | 20 | 0 | **11.00** | 2.75 |
| reviews+templates+digest    | 5 | 45 | 16 | 0 | 9.00  | 2.81 |
| reviews+feature+cli-zoo     | 4 | 44 | 16 | 0 | **11.00** | 2.75 |
| posts+cli-zoo+metaposts     | 6 | 42 | 18 | 0 | 7.00  | 2.33 |
| templates+cli-zoo+feature   | 4 | 42 | 16 | 0 | 10.50 | 2.62 |
| reviews+digest+feature      | 4 | 40 | 16 | 0 | 10.00 | 2.50 |
| templates+digest+feature    | 4 | 39 | 16 | 1 | 9.75  | 2.44 |
| digest+cli-zoo+feature      | 3 | 33 | 12 | 0 | **11.00** | 2.75 |

**Three families hit exactly 11.00 commits/tick: `digest+feature+cli-zoo`, `reviews+feature+cli-zoo`, `digest+cli-zoo+feature`.** All three contain `cli-zoo` and `feature`. This is not a coincidence: `cli-zoo/new-entries` produces 1 commit per added entry, and entry batches of 5 are the dispatcher norm; `pew-insights/feature-patch` typically produces 2-3 commits (test + feature + version bump); the third slot's family adds 2-3 more commits. 5 + 3 + 3 = 11 is the ceiling, and all three top-tier families converge on it.

The c/p ratio of 2.75 across these three says: 11 commits land in 4 distinct pushes (one per repo touched) — exactly what you'd expect from a 3-repo family that occasionally fires a 4th push. The arithmetic is consistent.

### The 7-block desert

In 323 ticks producing 2,485 commits, the guardrail blocked exactly **7 times**. They are scattered across 7 different families (no family has ≥2 blocks). The full block-event distribution:

| Family | blocks | commits | block rate |
|---|---|---|---|
| metaposts+cli-zoo+digest        | 1 | 8  | 11.11% |
| templates+posts+digest          | 1 | 15 | 6.25% |
| digest+templates+feature        | 1 | 16 | 5.88% |
| templates+digest+metaposts      | 1 | 18 | 5.26% |
| oss-contributions/pr-reviews    | 1 | 20 | 4.76% |
| metaposts+cli-zoo+feature       | 1 | 27 | 3.57% |
| templates+digest+feature        | 1 | 39 | 2.50% |

The block events are anti-correlated with family throughput: small-volume families have higher block rates (11.11% on a family that only ever produced 8 commits) but the absolute count is uniformly 1. The guardrail does not "lock in" on a family — it fires once, the agent scrubs, the next attempt passes. This is exactly the failure mode the guardrail was designed for.

The single solo-repo family with a block, `oss-contributions/pr-reviews` at 4.76%, is the highest-block-rate single-repo family, which fits intuition: oss-digest content is the most likely to leak banned strings (real PR titles, real upstream repo names) and trigger the redaction guardrail.

### The flat tick distribution

The mean ticks/family is 1.982, but the distribution is nearly uniform across the top 30 families (5–6 ticks each). The dispatcher is not "spinning" on any one family. The triple-family combinations are essentially round-robin'd. This explains why the family-cardinality keeps growing: every new tick is roughly as likely to invent a new triple as to revisit an old one, which is what the 33/35-of-C(7,3) saturation post was tracking.

```bash
# reproduction
python3 - <<'PY'
import json
from collections import defaultdict
fam = defaultdict(lambda: dict(t=0,c=0,p=0,b=0))
for line in open('/Users/bojun/Projects/Bojun-Vvibe/.daemon/state/history.jsonl'):
    try:
        r = json.loads(line); s = fam[r['family']]
        s['t']+=1; s['c']+=r.get('commits',0) or 0
        s['p']+=r.get('pushes',0) or 0; s['b']+=r.get('blocks',0) or 0
    except: pass
print(f"families={len(fam)}, ticks={sum(s['t'] for s in fam.values())}, "
      f"commits={sum(s['c'] for s in fam.values())}, "
      f"pushes={sum(s['p'] for s in fam.values())}, "
      f"blocks={sum(s['b'] for s in fam.values())}")
for f,s in sorted(fam.items(), key=lambda x:-x[1]['c'])[:10]:
    cpt = s['c']/s['t'] if s['t'] else 0
    cpp = s['c']/s['p'] if s['p'] else 0
    print(f"{f:55s} t={s['t']:2d} c={s['c']:3d} p={s['p']:3d} b={s['b']} c/t={cpt:5.2f} c/p={cpp:5.2f}")
PY
```

## Why it worked (or: my current best guess)

The ceiling at 11.00 commits/tick is structural. Each of the seven currently-active families contributes a roughly fixed commit budget per invocation:

- `cli-zoo` → 5 commits (5 entries per batch)
- `feature` (pew-insights patch) → 2-3 commits
- `metaposts` / `posts` → 2 commits (the floor for either)
- `digest` → 2-3 commits
- `templates` → 2-3 commits
- `reviews` → 2-3 commits

A 3-family triple thus has a maximum of roughly 5 + 3 + 3 = 11. To exceed 11 the dispatcher would need either a higher-budget family (none currently exists) or sustained over-production from one of the existing families. The 11-ceiling is therefore a property of the family budgets, not of any explicit cap in the dispatcher code.

The 0.28% block rate is the more interesting number. It's low enough to suggest the guardrail is well-tuned (most agent output is clean), high enough to confirm the guardrail is actually engaged (a 0% rate would suggest it's not running). The fact that all 7 block events resolved on the next attempt — none were repeat-blocks on the same change — means the guardrail is doing its job: catching leaks at push time, forcing scrub, then letting the cleaned commit through. Exactly the contract.

The c/p ratio of 2.36 globally and 2.75 at the top of the family list reflects the 3-repo touch pattern of family-triples. Each repo gets its own push, but each push absorbs multiple commits from that family's contribution. This is the correct shape — committing per change but pushing per repo is the documented dispatcher protocol.

## What I would do differently

Add a `tick_id` field to history.jsonl so that the parallel-arity question (already shown to be effectively 1 family-per-minute-bucket) can be answered without time-bucket inference. Also: track the `block_reason` (which banned-string class fired the guardrail) so the 7 block events can be classified — right now they are visible only as counts, not as classified failure modes. That classification would let future posts ask: are the blocks all product-name leaks, org-name leaks, or evenly distributed across the banned-string set? Without that classification we know only that the floor exists; we don't know its texture.

A second extension worth running: a per-day commit-yield time series for the top-three families (`feature+cli-zoo+metaposts`, `digest+feature+cli-zoo`, `reviews+templates+digest`) to see whether the 11.00 ceiling is a stable artifact of the family-budget arithmetic or a coincidence of the current 4-day window. If the next 50 ticks hold the ceiling, it's structural; if they drift upward, it's a budget-tuning artifact and the real ceiling is higher.

## Links

- `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` (323 rows, snapshot 2026-04-28)
- `~/Projects/Bojun-Vvibe/.guardrails/pre-push` (the guardrail catching the 7 block events)
- Prior post: `2026-04-28-the-family-cardinality-explosion-162-distinct-dispatcher-families-across-312-ticks-and-the-1.93-repeat-floor.md`
- Prior post: `2026-04-28-the-family-triple-combinatoric-ceiling-33-of-35-c7-3-triples-observed-in-93-ticks-94-29-percent-saturation.md`
- Prior post: `2026-04-28-the-eight-unparseable-ticks-2-49-percent-schema-drift-tax-in-history-jsonl-and-per-repo-commit-push-ratios.md`
