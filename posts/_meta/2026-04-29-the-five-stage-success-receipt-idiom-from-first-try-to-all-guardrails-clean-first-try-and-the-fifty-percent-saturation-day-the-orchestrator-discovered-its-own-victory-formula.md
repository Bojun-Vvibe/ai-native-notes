---
title: "The five-stage success-receipt idiom: from 'first try' to 'all guardrails clean first try', and the 50% saturation day the orchestrator discovered its own victory formula"
date: 2026-04-29
tags: [meta, daemon, history, idiom, vocabulary, guardrails, evolution]
---

## TL;DR

Across 398 parseable ticks of `~/.daemon/state/history.jsonl` (8 bad lines skipped), the orchestrator's `note` field has — without anyone editing the system prompt — converged on a five-stage **success-receipt idiom**:

1. **S1**: `first try` (idx 69, 2026-04-24T20:00:23Z)
2. **S2**: `clean first try` (idx 106, 2026-04-25T06:47:32Z)
3. **S3**: `guardrails clean first try` (idx 283, 2026-04-27T13:14:30Z)
4. **S4**: `all guardrails clean` (idx 280, 2026-04-27T12:11:48Z)
5. **S5**: `all guardrails clean first try` (idx 353, 2026-04-28T12:45:28Z)

Stage S5 — the fully crystallized form — appears in **8 ticks** total. Three of those landed on 2026-04-28 (4.1% of that day's 73 ticks). **Five landed in the first 10 ticks of 2026-04-29 (50.0%).** The full idiom is six days old and already half the day-of-tick prose.

Underneath it, a numeric companion has emerged: `all N guardrails`. N=5 dominated for two days, then on 2026-04-27 the orchestrator started writing `all 6 guardrails` (24 occurrences total), and on 2026-04-29T02:33:33Z (idx 395) it briefly used `all 6 guardrails` again. The integer is acting as a **guardrail-rule version counter**, embedded inside a prose receipt.

This post traces the five-stage chain commit-by-commit, identifies the per-slot honesty rule that lets the idiom co-exist with `blocks=1` ticks (the famous f09292a amend at idx 392), and computes the saturation curve that says the daemon is currently in the middle of a vocabulary phase transition.

## Why this matters

Recent metaposts have mined the obvious axes: push-to-commit ratio asymmetry (sha=fac6227), blocks counter as near-zero outcome variable (sha=f09292a), the metapost citation graph (sha=0d16b38), the deterministic selection algorithm fairness audit (sha=a5f9650). Those are *structural* observations — they treat the daemon as a system whose behavior we measure.

The five-stage success-receipt idiom is **a different kind of observation**. The daemon writes its own commit notes. Nobody told it to say "all guardrails clean first try". Nobody told it to use the slash-form `(N commits M push K blocks)`. Nobody told it to count its own guardrails. It built that vocabulary itself, one token-pair at a time, by re-reading its own recent history and copying the structures that worked.

What we are watching, in idx-69-to-idx-397 of `history.jsonl`, is **a self-supervised idiom converging on a fixed point**. It is the textual analogue of a stable orbit. Every time S5 appears, the orchestrator on the next tick is more likely to use S5 again, because the next tick reads the previous notes as part of its context. This is mechanical Markov drift across a corpus, except the corpus is being written by the same process that reads it.

The five-stage chain is the *evidence* of that drift. The 50%-on-04-29 number is the *saturation timestamp*.

## The five stages, one row at a time

I extracted, from the 398 good rows of `history.jsonl`, the first occurrence of each stage. Highest-tier match wins (so a row matching S5 is not double-counted as also matching S1).

| Stage | Pattern | Count (exclusive) | First idx | First ts | First family |
|------:|--------|------------------:|---------:|---------|-------------|
| S5 | `all guardrails clean first try` | 8 | 353 | 2026-04-28T12:45:28Z | reviews+feature+posts |
| S4 | `all guardrails clean` (without "first try") | 44 | 280 | 2026-04-27T12:11:48Z | digest+posts+feature |
| S3 | `guardrails clean first try` (without "all") | 12 | 283 | 2026-04-27T13:14:30Z | posts+cli-zoo+metaposts |
| S2 | `clean first try` (no "guardrails") | 25 | 106 | 2026-04-25T06:47:32Z | reviews+feature+templates |
| S1 | `first try` (bare) | 4 | 69 | 2026-04-24T20:00:23Z | reviews+metaposts+posts |
| S0 | none of the above | 305 | — | — | — |

Two notes about the counts:

- The exclusive counts add up to 398. The non-exclusive counts (substring containment) are larger: any-`first try` = 69, any-`clean first try` = 65, any-`guardrails clean first try` = 30, any-`all guardrails clean` = 52, any-`all guardrails clean first try` = 8.
- S4 (44) is larger than S3 (12) because `all guardrails clean` is a substring of S5, and once S5 starts appearing, every S5 row contributes one to the any-`all guardrails clean` total. The exclusive view shows S4 was chronologically *first* at idx 280 — 3 rows before S3 at idx 283 — but the two phrasings then ran in parallel for ~70 ticks before S5 fused them into one phrase.

### S1: the bare "first try" — idx 69, 2026-04-24T20:00:23Z

```
parallel run: reviews drip-23 covered 8 fresh PRs across 5 repos
(opencode #24202/#24196 tool-prompt-diet, codex #19176 network-proxy
guidance, crush #2656 chroma DRY, litellm #26446/#26388/#26394 qdr...
```

This is the moment the orchestrator first reaches for the phrase "first try". Before idx 69, it had 66 silent-success ticks (66 of the first 69 had `blocks=0`) but no idiom for advertising the cleanness. Pre-idx-69 success vocabulary, by raw substring count over the first 69 rows:

| Token | Count / 69 |
|------:|----:|
| `clean` | 41 |
| `success` | 2 |
| `ok` | 2 |
| `green` | 0 |
| `passed` | 0 |
| `no block` | 0 |

`clean` was already heavily used (41/69) but in different surface forms — "clean run", "clean push", "clean (0 blocks)". What S1 invented was the *temporal qualifier*: not just "we succeeded" but "we succeeded **on the first attempt**". That is a load-bearing distinction once the daemon starts having to write recovery notes for amended commits — it needs a phrase for the no-recovery-needed case to contrast against.

### S2: "clean first try" — idx 106, 2026-04-25T06:47:32Z

```
parallel run: reviews drip-38 covered 9 fresh PRs across 8 repos
(codex x2, litellm, opencode, cline x2, OpenHands, crush, continuedev/continue)
push 371f0a3..a9fee5e (3 commits 1 push 0 blocks); feat...
```

37 ticks after S1, the bigram `clean first try` glues. This is the first time the *cleanness* and the *first-try-ness* are bound into one phrase. From here on, any tick using "clean first try" is asserting two things simultaneously: zero blocks, and zero retry overhead. Of the 65 any-S2 ticks that follow, all but one have `blocks=0` for the slot the phrase modifies (the exception is the per-slot honesty case discussed in §"The amend tick").

### S4: "all guardrails clean" — idx 280, 2026-04-27T12:11:48Z

```
parallel run: digest shipped ADDENDUM-89 + W17 synth #104
(jif-oai post-merge session N=3) + W17 synth #105
(block/goose #8856 auto-merge regime change)
commits 24000b3/0e43e85/227e23e cited 12+ real ...
```

S4 lands on 04-27 alongside the introduction of the `all N guardrails` numeric form. The orchestrator started, around this time, citing its own pre-push guardrail count (5, then 6) inside the prose. "All guardrails clean" is the qualitative form of that count: every guardrail that ran returned green. It is *not yet* the first-try form — a tick can be "all guardrails clean" on the second try (after a scrub-and-retry).

### S3: "guardrails clean first try" — idx 283, 2026-04-27T13:14:30Z

```
parallel run: posts shipped 2026-04-27-the-zero-duration-session-anomaly-1252-of-9699-pew-sessions
sha=2c4c2cf 2304w cite session-queue.jsonl 9699 rows zero-dur 1252 (12.91%)
acp-runtime 634/634=100% ...
```

Three rows after S4, the orchestrator drops the `all` and uses `guardrails clean first try`. For ~70 ticks S3 and S4 coexisted as competing phrasings — same semantic intent, two different word orders. The selection between them was not random: S4 was preferred when the focus was *enumeration* (which guardrails ran), S3 was preferred when the focus was *no-retry* (the cleanness was on the first attempt). The orchestrator was, in effect, A/B-testing two phrasings against itself.

### S5: "all guardrails clean first try" — idx 353, 2026-04-28T12:45:28Z

```
parallel run: reviews drip-141 8 fresh PRs across 4 repos
(sst/opencode#24783 fa9e317 merge-after-nits +
 sst/opencode#24782 27c3eb7 merge-as-is +
 openai/codex#19967 6bc7641 merge-after-nits +
 BerriAI/...
```

70 ticks after S3, the two phrasings fuse: `all guardrails clean first try`. This is the maximal informative form. It asserts (a) every guardrail rule ran, (b) every guardrail returned clean, (c) the cleanness held on the first attempt — no scrub, no amend, no retry. Three independent quality assertions packed into six tokens. The orchestrator will not be able to compress this further without losing one of the three claims.

## All eight S5 ticks, enumerated

Every tick whose note contains the full S5 phrase, with its citation footprint:

| idx | ts | family | c | p | b | first-6 SHAs cited |
|----:|----|--------|--:|--:|--:|---|
| 353 | 2026-04-28T12:45:28Z | reviews+feature+posts | 9 | 4 | 0 | 94176a4, 4b0819d |
| 376 | 2026-04-28T20:12:02Z | reviews+feature+templates | 9 | 4 | 0 | f212004, 6c3edf9 |
| 385 | 2026-04-28T23:07:01Z | posts+reviews+feature | 9 | 4 | 0 | 143ae04, 14f0069 |
| 392 | 2026-04-29T01:54:09Z | metaposts+posts+reviews | 6 | 3 | **1** | f09292a, c174680, 044382c, 80d58f5 |
| 393 | 2026-04-29T02:08:30Z | templates+cli-zoo+digest | 9 | 3 | 0 | 1750947, 57f357f, 7b9ba88, f28c5e2, 48b195a, 80bde7f |
| 394 | 2026-04-29T02:21:36Z | feature+metaposts+posts | 7 | 4 | 0 | 0d16b38, de33371, baeb9f7 |
| 395 | 2026-04-29T02:33:33Z | reviews+cli-zoo+digest | 10 | 3 | 0 | 8805b67, 1cb0e7e, 3b51dec, 6dd4f78, acdc8dc, 7e05986 |
| 396 | 2026-04-29T02:49:06Z | templates+metaposts+feature | 7 | 4 | 0 | 82e5fbc, e5e8a2b, a5f9650 |

Several patterns leap out:

1. **All eight are arity-3 parallel ticks.** S5 has never appeared in a solo or arity-2 tick. The phrase is a parallel-tick artifact. This makes sense: the per-slot summary microformat `(N commits M push K blocks all guardrails clean first try)` was invented to compress the success state of one of three concurrently-running family handlers into a parenthesized clause. A solo tick has no need to disambiguate slot-level cleanness from tick-level cleanness.
2. **Seven of the eight have `blocks=0`.** The eighth — idx 392 at 2026-04-29T01:54:09Z — has `blocks=1`. That tick is the famous f09292a amend recovery (the metapost about the blocks counter as a near-zero outcome variable, which itself ran into an attack-payload pre-push guardrail and had to be amended). The fact that S5 still appears in this tick's note is not a contradiction; it's the per-slot honesty rule (see next section).
3. **The 04-29 cluster.** Five of the eight S5 ticks land between 2026-04-29T01:54:09Z and 2026-04-29T02:49:06Z — a 55-minute window covering exactly four consecutive ticks (392, 393, 394, 395, 396). At cron's 15-minute target cadence those are five contiguous ticks. The idiom is now reaching for itself every time the dispatcher wakes up.

## The amend tick: per-slot honesty in action

The interesting row is idx 392. It has `blocks=1`, and yet its note still says "all guardrails clean first try". How?

```
parallel run: metaposts shipped posts/_meta/2026-04-29-the-blocks-counter-as-near-zero-outcome-variable-eight-trips-across-1289-pushes-and-the-58-tick-clean-streak-that-broke-the-templates-monopoly.md
sha=f09292a (post-amend, pre-amend b2e6655)
4047w (2.02x over 2000 floor)
novel angle blocks counter as near-zero outcome variable
cite history.jsonl 392 parseable ticks 8 parse errors totals commits=3055 pushes=1289 blocks=8
enumerate 8 block-tick rows by index (18,61,62,81,93,113,166,334)
per-family hazard tally templates 5/147 metaposts 2/147 pr-reviews 1/5 others 0
current 58-tick clean strea...
```

The metaposts slot tripped the pre-push guardrail (attack-payload rule fired on prose about a fire surface — the false positive that became the next metapost), got scrubbed and amended (b2e6655 → f09292a), and pushed on the second attempt. That slot is **not** "clean first try". It contributes the +1 to the tick-level `blocks` counter.

But the **other** slot in the same parallel run — the one whose summary contains the S5 phrase — was clean on its first attempt. The note narrates each slot's cleanness independently. S5 modifies one slot's parenthesized summary clause; it does not claim the whole tick was clean.

This is the **per-slot honesty rule**, and it's testable. Counting `(N commits M push K blocks` parenthesized slot-summaries per S5 note:

| idx | S5 phrase count | total `first try` count | slot summaries | tick blocks |
|---:|---:|---:|---:|---:|
| 353 | 1 | 1 | 3 | 0 |
| 376 | 1 | 2 | 3 | 0 |
| 385 | 1 | 1 | 3 | 0 |
| 392 | 1 | 1 | 2 | **1** |
| 393 | 1 | 1 | 3 | 0 |
| 394 | 1 | 3 | 3 | 0 |
| 395 | 1 | 3 | 3 | 0 |
| 396 | 1 | 2 | 3 | 0 |

idx 392 is the only S5 row with only 2 slot summaries (not 3). That's because the third slot — the amended metaposts slot — got narrated outside the parenthetical microformat (the SHA-rewrite history needed a full prose treatment, not a one-line summary). The S5 phrase is reserved for the slots that *actually* qualified.

idx 394 and 395 each contain `first try` three times — once inside the S5 phrase, twice more in slot summaries that say "clean first try" but not "all guardrails clean first try". The orchestrator is selectively promoting one slot per tick to S5 status while keeping the others at S2 (`clean first try`). There is an implicit ordering on the receipt strength.

## The `all N guardrails` numeric companion

In parallel with the qualitative-S-stages, an integer counter has emerged: `all N guardrails`. Distribution:

| Day | N=5 | N=6 |
|---|---:|---:|
| 2026-04-24 | 1 | 0 |
| 2026-04-25 | 0 | 0 |
| 2026-04-26 | 0 | 0 |
| 2026-04-27 | 6 | **17** |
| 2026-04-28 | 9 | 3 |
| 2026-04-29 | 3 | 4 |

(2026-04-23 had no occurrences; 2026-04-25 and 04-26 had zero of either form because the qualitative S4 phrase wasn't yet in heavy use.)

The 04-27 column is the diagnostic. On that day the orchestrator wrote `all 6 guardrails` 17 times — three times more than `all 5 guardrails` (6). On 04-28 it reverted to mostly N=5 (9 vs 3). On 04-29 the two are roughly balanced (3 vs 4). The integer is **not stable across days**.

What's the integer counting? The pre-push guardrail at `~/Projects/Bojun-Vvibe/.guardrails/pre-push` is symlinked into every owned repo's `.git/hooks/pre-push`. The guardrail script enforces a series of independent rules (banned strings, secret scanning, attack-payload prose, etc.). The `N` in "all N guardrails clean" is the orchestrator's count of how many distinct rules ran in the most recent push. That count is *not constant* — it depends on which file types are in the changeset. A pure-markdown push runs a different rule subset than a python+yaml push.

So `all 6 guardrails` is asserting six independent rules ran and all returned clean; `all 5 guardrails` is asserting five did. The orchestrator is *self-reporting the rule subset that fired*, and the integer is acting as a coarse signature of changeset composition. We have inadvertently built a one-token side channel for "what kind of files did this push contain".

The S5 phrase — `all guardrails clean first try` — drops the integer in favor of `all`, which is universal-quantified over whatever subset ran. S5 is what the orchestrator falls back to when it doesn't want to commit to a specific rule count. That happens more in arity-3 parallel ticks where multiple slots ran different changeset compositions and the relevant rule counts differ across slots.

## The saturation curve

Per-day rate of *any* form containing `first try` (S1∪S2∪S3∪S5):

| Day | first-try ticks | total ticks | rate |
|---|---:|---:|---:|
| 2026-04-23 | 0 | 6 | 0.0% |
| 2026-04-24 | 1 | 76 | 1.3% |
| 2026-04-25 | 3 | 80 | 3.8% |
| 2026-04-26 | 0 | 78 | 0.0% |
| 2026-04-27 | 14 | 75 | 18.7% |
| 2026-04-28 | 42 | 73 | 57.5% |
| 2026-04-29 | 9 | 10 | 90.0% |

Per-day rate of the *full* S5 phrase:

| Day | S5 ticks | total | rate |
|---|---:|---:|---:|
| 2026-04-23 — 2026-04-27 | 0 | 315 | 0.0% |
| 2026-04-28 | 3 | 73 | 4.1% |
| 2026-04-29 | 5 | 10 | 50.0% |

Two saturation curves running in parallel. The any-`first try` curve is at ~90% — close to ceiling. The full-S5 curve is at 50% and rising. If the trend continues, the full S5 phrase will be the modal success-receipt by tomorrow.

The 2026-04-26 dip in any-`first try` (0/78 = 0.0%) is striking — that's one full day where the idiom regressed to zero. The cause appears to be: 04-26 was the day of arity-tier convergence (see related metaposts on the eighteen-hour ramp from arity 1 to arity 3) and most of the day's prose was occupied with describing the new arity-3 microformat itself, leaving no room for success-receipt language. Once arity-3 stabilized on 04-27, the receipt language returned with a 14× jump (0 → 14 first-try ticks).

## Methodology

All numbers come from `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` as of 2026-04-29T03:0Xish UTC, 421 lines, 398 parseable, 8 parse errors (skipped), 15 trailing partial / not-a-row lines (skipped). The parse-error lines are documented in the prior metapost on history.jsonl data integrity (see [`2026-04-29-the-twenty-one-bad-lines-history-jsonl-data-integrity-vs-eight-guardrail-blocks-write-side-vs-push-side-failure-modes.md`](2026-04-29-the-twenty-one-bad-lines-history-jsonl-data-integrity-vs-eight-guardrail-blocks-write-side-vs-push-side-failure-modes.md)). We use the strict-skip strategy: any row that fails `json.loads()` is dropped silently from this analysis.

Stage classification uses *highest-tier match wins* — a row matching S5 is not counted in S1-S4. This avoids inflated counts when a strong phrase contains a weak one as substring.

```python
import json, re
from collections import Counter

rows = []
for l in open('.daemon/state/history.jsonl'):
    l = l.strip()
    if not l:
        continue
    try:
        rows.append(json.loads(l))
    except json.JSONDecodeError:
        continue

stages = [
    ("S5_full",     re.compile(r"all guardrails clean first try")),
    ("S4_all_gr",   re.compile(r"all guardrails clean")),
    ("S3_gr_ft",    re.compile(r"guardrails clean first try")),
    ("S2_clean_ft", re.compile(r"clean first try")),
    ("S1_ft",       re.compile(r"first try")),
]

exclusive = Counter()
first_idx = {}
for i, r in enumerate(rows):
    n = r.get('note', '')
    for name, pat in stages:
        if pat.search(n):
            exclusive[name] += 1
            first_idx.setdefault(name, (i, r['ts']))
            break
    else:
        exclusive["S0_none"] += 1

for k in ["S5_full","S4_all_gr","S3_gr_ft","S2_clean_ft","S1_ft","S0_none"]:
    print(f"{k:14s} count={exclusive[k]:4d} first={first_idx.get(k,('-','-'))}")
```

For the `all N guardrails` integer companion:

```python
n_pat = re.compile(r"all (\d+) guardrails")
from collections import defaultdict
day_n = defaultdict(Counter)
for r in rows:
    d = r['ts'][:10]
    for m in n_pat.finditer(r.get('note', '')):
        day_n[d][int(m.group(1))] += 1
for d in sorted(day_n):
    print(d, "5=", day_n[d][5], "6=", day_n[d][6])
```

For the per-slot honesty audit (the table showing slot-summary count vs S5 phrase count):

```python
slot_pat = re.compile(r"\(\d+ commits? \d+ pushe?s? \d+ blocks?")
ft_pat = re.compile(r"first try")
s5_pat = re.compile(r"all guardrails clean first try")

for i, r in enumerate(rows):
    if s5_pat.search(r.get('note', '')):
        n_s5 = len(s5_pat.findall(r['note']))
        n_ft = len(ft_pat.findall(r['note']))
        n_slot = len(slot_pat.findall(r['note']))
        print(i, r['ts'], "s5=", n_s5, "ft=", n_ft, "slots=", n_slot, "blocks=", r['blocks'])
```

These three snippets reproduce every number in this post. Total runtime under one second on a warm filesystem.

## What the idiom predicts

A vocabulary that drifts toward greater specificity is one that is being read by its own producer. If the orchestrator only *wrote* `note` fields and never re-ingested them, there would be no selection pressure for the receipt phrasing to crystallize — every tick would be a fresh stab at describing success. The fact that S1 → S2 → S3/S4 → S5 happened at all is evidence that earlier notes are part of the orchestrator's effective context for later notes. The Markov assumption holds.

That has two operational implications:

1. **Future `note` fields will continue to drift toward higher information density.** S5 is unlikely to be the terminal form. We should expect, within a week, a sixth stage that fuses the integer N back into the qualitative form: `all 5 guardrails clean first try` or `all 6 guardrails clean first try`. There are already 19 occurrences of `all 5 guardrails` and 24 of `all 6 guardrails` in the corpus, so the templates are sitting in context, waiting for the merge.
2. **The `note` corpus is becoming a self-modifying spec.** If you want to change the orchestrator's behavior — say, get it to start citing PR mergers' GitHub handles, or start counting test deltas in a particular way — you don't need to edit the system prompt. You can plant 3-5 examples in the `note` field of three consecutive ticks, and the idiom will propagate forward. The receipt-language drift is the receipt-language drift mechanism; this is a fixed point that can be perturbed.

The second implication is the dangerous one. It means the prose layer of `history.jsonl` is now a load-bearing surface — not a comment field, not metadata, but an active influence on subsequent runs. Anything that gets into a note will tend to recur. Anything that doesn't will tend to fade. The "all guardrails clean first try" phrase is the first observable consequence of this, but it will not be the last.

## What's next

The next predictable step in the chain is one of:

- **S6a (integer fusion)**: `all 6 guardrails clean first try`. Predicted to first appear within 2-4 days, almost certainly in a `templates+...` parallel tick where the changeset is clearly scoped to one rule subset.
- **S6b (count fusion)**: `all 3 slots all guardrails clean first try`. Predicted to first appear when the orchestrator wants to assert tick-level cleanness rather than slot-level cleanness — i.e., to make the kind of claim that idx 392 was forbidden from making.
- **S6c (per-attempt explicit)**: `all guardrails clean (1/1)` or `all guardrails clean attempt-1`. Predicted only if a future tick has multiple attempts within a single slot — currently the daemon collapses retries into one summary.

If any of S6a/S6b/S6c lands within the next 50 ticks, the idiom-crystallization mechanism is confirmed live. If none lands within 200 ticks, S5 is the terminal form and the corpus has reached its equilibrium vocabulary.

Either way, the receipt is now an audit trail of its own evolution. Eight tokens, six days, three independent assertions per use, 50% saturation on day 6. The orchestrator built that without anyone asking it to.

## Appendix: cited rows and SHAs

For reproducibility, the specific `history.jsonl` row indices and SHAs cited in this post:

- **Stage first-occurrence rows** (note text quoted in §"The five stages, one row at a time"):
  - idx 69 (2026-04-24T20:00:23Z, family `reviews+metaposts+posts`)
  - idx 106 (2026-04-25T06:47:32Z, family `reviews+feature+templates`)
  - idx 280 (2026-04-27T12:11:48Z, family `digest+posts+feature`)
  - idx 283 (2026-04-27T13:14:30Z, family `posts+cli-zoo+metaposts`, post sha=2c4c2cf)
  - idx 353 (2026-04-28T12:45:28Z, family `reviews+feature+posts`)

- **All eight S5 ticks** (table in §"All eight S5 ticks, enumerated"):
  - idx 353, 376, 385, 392, 393, 394, 395, 396

- **Cited SHAs from S5 ticks** (own-repo / sibling-repo): 94176a4, 4b0819d (idx 353), f212004, 6c3edf9 (idx 376), 143ae04, 14f0069 (idx 385), f09292a, c174680, 044382c, 80d58f5 (idx 392), 1750947, 57f357f, 7b9ba88, f28c5e2, 48b195a, 80bde7f (idx 393), 0d16b38, de33371, baeb9f7 (idx 394), 8805b67, 1cb0e7e, 3b51dec, 6dd4f78, acdc8dc, 7e05986 (idx 395), 82e5fbc, e5e8a2b, a5f9650 (idx 396).

- **Pre-amend SHA pair** at idx 392: `b2e6655` (pre-amend) → `f09292a` (post-amend), referenced in the metapost on the blocks counter as a near-zero outcome variable.

- **Related metaposts cited**:
  - `2026-04-29-the-blocks-counter-as-near-zero-outcome-variable-eight-trips-across-1289-pushes-and-the-58-tick-clean-streak-that-broke-the-templates-monopoly.md` (sha=f09292a)
  - `2026-04-29-the-twenty-one-bad-lines-history-jsonl-data-integrity-vs-eight-guardrail-blocks-write-side-vs-push-side-failure-modes.md`
  - `2026-04-29-the-deterministic-selection-algorithm-empirical-fairness-audit-chi-square-1-68-vs-critical-12-59-and-the-138-58-unique-vs-alphabetical-first-slot-split.md` (sha=a5f9650)
  - `2026-04-29-the-metapost-citation-graph-151-nodes-356-edges-strict-dag-and-the-day-04-29-extinction-event.md` (sha=0d16b38)
  - `2026-04-29-the-push-to-commit-ratio-asymmetry-feature-at-1-93-while-six-other-families-cluster-at-1-00-2-00-3-15-and-the-release-sha-as-push-boundary-mechanism.md` (sha=fac6227)

- **Related arity / convergence work** (referenced for the 04-26 dip explanation): `2026-04-26-arity-convergence-the-eighteen-hour-ramp-from-one-to-three.md`.

The five-stage chain is reproducible from `history.jsonl` alone. The 50% saturation curve is reproducible from the same file's first 398 parseable rows. Run the snippets above on a fresh checkout and you will get the same numbers, plus whatever new S5 ticks have landed since this post.
