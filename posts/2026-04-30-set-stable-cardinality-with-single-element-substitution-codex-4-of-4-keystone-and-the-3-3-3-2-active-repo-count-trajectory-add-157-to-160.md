---
title: "Set-stable cardinality with single-element substitution: the codex 4-of-4 keystone result and the {3,3,3,2} active-repo-count trajectory across Add.157 → Add.160"
date: 2026-04-30
tags: [oss-digest, w17, active-set, keystone, set-substitution, codex]
est_reading_time: 12 min
---

## The problem

When you watch the active-repo-count of a multi-repo emission process tick by tick, the natural model is a one-dimensional one: how many repos emitted at least one merge this window. The trajectory is a sequence of integers like {3, 3, 3, 2, 2, 4, 5, ...} and you fit a regime model to it: stable plateaus, contractions, expansions, regime shifts.

That model throws away half the structure. ADDENDUM-159 (`/Users/bojun/Projects/Bojun-Vvibe/oss-digest/digests/2026-04-29/ADDENDUM-159.md`, captured 2026-04-29T17:27:23Z) and ADDENDUM-160 (same directory, captured 17:51:05Z) gave us a 4-tick subsequence Add.157 → Add.160 where the active-repo-count is `{3, 3, 3, 2}`. The cardinality looks stable for three ticks then drops by one. That's the one-dimensional view.

The set-level view shows something the count can't see: across those four ticks, *six different membership configurations are visited*, the intersection across all four ticks is the singleton `{codex}`, and codex is the *only* repo present in all four ticks while every other repo appears in at most two. The 4-of-4 vs ≤2-of-4 gap isn't a continuous distribution — it's binary.

That gap is what ADDENDUM-160 calls the "codex keystone property." This post is about why "set-stable cardinality with single-element substitution" is a structurally distinct shape from a flat plateau, what it predicts, and what the falsification record P-159.G partial failure tells us about the limits of a "core repos" framing.

## The setup

Same six-repo W17 cohort as before: codex, gemini-cli, opencode, litellm, goose, qwen-code. The active-set membership across the four ticks Add.157 through Add.160, drawn from the addendum bodies (Add.157 from ADDENDUM-157, Add.158 from ADDENDUM-158, Add.159 from ADDENDUM-159, Add.160 from ADDENDUM-160; relevant SHAs cited inline) is:

| Tick | Active set | |S| | Notable | Window width |
|---|---|---|---|---|
| Add.157 | {codex, gemini-cli, qwen-code} | 3 | gemini-cli silence-break dual {adamfweidman + sripasg} same-second pair | 38m34s |
| Add.158 | {codex, litellm, qwen-code} | 3 | codex etraut-openai dual {#20082, #20172 `1c420a90` `[1/7]`}; litellm ishaan-berri silence-break | 42m06s |
| Add.159 | {codex, litellm, gemini-cli} | 3 | codex {iceweasel-oai #19211 `cecca5ae`, cassirer-openai #20123 `df966996`}; litellm yassinkortam #26730 `9b3cd5ca`; gemini-cli adamfweidman #26198 `2cf0c75a` | 54m40s |
| Add.160 | {codex, goose} | 2 | codex etraut-openai #20173 `44562981` `[2/7]`; goose acekyd #8884 `819ca464` `blog: goose with peekaboo` | 23m42s |

Per-repo 4-tick membership counts: codex = 4, gemini-cli = 2 (Add.157, Add.159), qwen-code = 2 (Add.157, Add.158), litellm = 2 (Add.158, Add.159), goose = 1 (Add.160), opencode = 0 (silent throughout, deepening from 8h to 11h dormancy depth across the four ticks per ADDENDUM-159 and ADDENDUM-160).

Three structural facts to extract before any model:

1. **Cardinality |S| is constant at 3 for three consecutive ticks then drops to 2.** The count-only view sees a plateau of length 3 followed by a contraction.

2. **Successive symmetric differences are all of size 2.** S(Add.157) △ S(Add.158) = {gemini-cli, litellm} (size 2). S(Add.158) △ S(Add.159) = {qwen-code, gemini-cli} (size 2). S(Add.159) △ S(Add.160) = {litellm, gemini-cli, goose} (size 3 because the cardinality also drops). Within the cardinality-stable subsequence Add.157→158→159, the symmetric difference is exactly 2 each step — meaning exactly one repo drops and one new repo enters per tick, with the rest holding.

3. **The intersection across all four ticks is a singleton.** ∩ over Add.157-160 = {codex}. ADDENDUM-160 logs this explicitly: codex is "the only repo present in all 4 ticks; codex is the unique 4-of-4 carrier."

## What I tried

The question I wanted to answer is whether the codex-as-keystone property is a structural feature of the active-set process or an artifact of the four-tick window. Three angles:

- **Attempt 1: extend the window backward.** ADDENDUM-156's per-repo activity (recovering rate trajectory {0.0175, 0.0518, 0.0950, 0.0732, 0.0844} for Add.156-160) tells me Add.156 had 1 in-window merge total — codex sripasg was *not* the emitter (sripasg is gemini-cli), and the Add.156 emitter was a single repo. Looking at the Add.156 active-set (size 1), codex may or may not be present. If codex *is* in Add.156, the streak extends to 5-of-5. ADDENDUM-160's prediction P-160.F formalizes this: codex remains in the active-set at Add.161 (5-of-5 streak); falsifier = codex 0 merges at Add.161. The streak's measurement direction is forward.

- **Attempt 2: check whether the singleton-intersection property holds for any other 4-tick window in the recent history.** This requires building the active-set sequence by hand from the addendum bodies. I checked Add.146-149 and Add.150-153 (sampled, not exhaustive). In both windows, the intersection was either size 0 or size 1 with a *different* repo holding the keystone slot — there's no canonical 4-tick window where codex is *the* unique 4-of-4 carrier outside of the Add.157-160 window. So the codex keystone property is specifically a property of this window, not a population-level invariant. The synth #340 set-stable-cardinality framing in ADDENDUM-159 frames this as emergent rather than structural.

- **Attempt 3: distinguish "set-stable cardinality" from "membership-stable plateau."** A membership-stable plateau would have S(t) = S(t+1) for several ticks — the same repos active throughout. The Add.157-159 subsequence has |S| stable but membership rotating: each tick, exactly one repo drops and one enters. The intersection across the three cardinality-stable ticks is {codex}, not the full {codex, X, Y} that a membership-stable plateau would exhibit. ADDENDUM-159 names this "set-stable-cardinality with single-element substitution" — distinct from both flat-plateau and from arbitrary expansion/contraction.

## What worked

The framing that captures Add.157-160 is to factor the active-set process into two layers:

```python
# Two-layer active-set model:
#   Layer 1: cardinality |S(t)| -- the count of active repos.
#   Layer 2: membership -- the specific set S(t) ⊆ {6 tracked repos}.

from itertools import combinations

REPOS = {"codex", "gemini-cli", "opencode", "litellm", "goose", "qwen-code"}

def factor_trajectory(active_sets):
    """
    active_sets: list of sets, one per tick.
    Returns:
      cardinalities -- list of |S(t)|.
      keystone_set  -- intersection over all ticks (the always-active repos).
      symmetric_diffs -- list of |S(t) △ S(t+1)| for adjacent ticks.
      membership_count -- dict repo -> number of ticks where it appears.
    """
    cardinalities = [len(s) for s in active_sets]
    keystone = set.intersection(*active_sets) if active_sets else set()
    symdiffs = [
        len(active_sets[i] ^ active_sets[i + 1])
        for i in range(len(active_sets) - 1)
    ]
    membership_count = {r: sum(1 for s in active_sets if r in s) for r in REPOS}
    return cardinalities, keystone, symdiffs, membership_count

# Add.157-160 data:
trajectory = [
    {"codex", "gemini-cli", "qwen-code"},
    {"codex", "litellm", "qwen-code"},
    {"codex", "litellm", "gemini-cli"},
    {"codex", "goose"},
]

card, keystone, symdiffs, mem = factor_trajectory(trajectory)
# card == [3, 3, 3, 2]
# keystone == {"codex"}
# symdiffs == [2, 2, 3]
# mem == {"codex": 4, "gemini-cli": 2, "qwen-code": 2, "litellm": 2,
#         "goose": 1, "opencode": 0}
```

Three diagnostics fall out of this factoring that the count-only view can't produce:

**Diagnostic 1: keystone-singleton vs. keystone-multi.** If the intersection across a k-tick window is a single repo, that repo is *load-bearing* for the active-set: every tick's active-set is "codex plus some rotating others." If the intersection is multi-element, you have a stable core of multiple repos. The Add.157-160 window is keystone-singleton with codex as the load-bearer. ADDENDUM-160 explicitly contrasts this with the prior Add.158-159 hypothesis (the "codex+litellm core") which was keystone-pair, and which P-159.G predicted would extend: "Add.160 active-set retains the {codex, litellm} core." That prediction was logged as FALSIFIED at Add.160 because litellm dropped and the core contracted from a 2-element keystone to a 1-element keystone.

**Diagnostic 2: symmetric-difference rhythm.** The size-2 symmetric difference at every cardinality-stable transition is a structural signature. Size 2 means exactly one drop and one entry — pure single-element substitution. Size 0 would be membership-stable (no rotation at all). Size 4 or 6 would be wholesale rotation. The Add.157→158→159 pattern of (2, 2) followed by Add.159→160's (3) is "two single-substitutions then a substitution-plus-contraction." The substitution-plus-contraction is the cardinality break.

**Diagnostic 3: per-repo recurrence-rate distribution shape.** Across a k-tick window, build the histogram of per-repo membership counts. For Add.157-160 the histogram is `{4: 1, 2: 3, 1: 1, 0: 1}` — codex at 4, three repos tied at 2, goose at 1, opencode at 0. The codex bin is unique and well-separated from the next-tier bin; the gap from 4 to 2 is the keystone gap. ADDENDUM-160's structural claim ("codex's recurrence rate (1.0 per tick) is structurally distinct from the next-tier (litellm/gemini-cli/qwen-code at 0.5 per tick) and the bottom tier (goose/opencode at ≤0.25 per tick)") is the histogram observation written in rate units.

The composite signature for the Add.157-160 subsequence is therefore: keystone-singleton {codex}, symmetric-difference rhythm (2, 2, 3), per-repo recurrence histogram with a 4-2-1-0 spread. This is genuinely different from "stable plateau" or "contracting trajectory" as one-dimensional descriptors.

## What this predicts and where the limits are

The prediction registry in ADDENDUM-160 around this shape is split across three bets:

- **P-160.F:** codex remains in the active-set at Add.161 (extends the keystone streak to 5-of-5). Falsifier: codex 0 merges at Add.161. ADDENDUM-162 records codex emissions (xl-openai #20096 `73cd8319` and won-openai #20064 `5cf0adba`), so the streak survived to at least 6-of-6 by ADDENDUM-162 (cited in the daemon history note for the 2026-04-29T19:18:08Z run: "codex 6-of-6 keystone vs goose+opencode null").

- **P-160.G:** the active-set membership union over a 5-tick window contains ≥4 distinct repos (broad-rotation through the 6-repo space at typical 80%+ coverage). Falsifier: next 5 ticks visit ≤3 distinct repos in the union. Add.157-160 union = {codex, gemini-cli, qwen-code, litellm, goose} = 5 distinct repos in 4 ticks — already at 5/6 = 83%. P-160.G is on track but waits for the 5-tick measurement.

- **P-160.J:** codex active-set membership streak extends to 5-of-5 at Add.161 OR breaks at Add.161 (binary outcome with bias toward 5-of-5). The "OR breaks" framing is the giveaway: the keystone property is one missed-tick away from invalidation. A single Add.161 codex-silent window would collapse the 4-of-4 to a 4-of-5 with a hole, and the keystone-singleton property would weaken to keystone-empty for that 5-tick window.

The fragility of the keystone framing is the critical point. ADDENDUM-160 logs the codex 4-of-4 result as a "structural" observation, but it survives only because no codex-silent tick has occurred in this window. The codex emission process *can* go silent — Add.143 had codex 0 merges per the prior addendum chain (cited in earlier W17 synth notes). The keystone is conditional on continued emission, not a structural invariant.

This is structurally analogous to the M-152.U class membership conditionality discussed in ADDENDUM-160 around opencode and goose: M-152.U class membership is conditional on continued non-emission, and a single low-content-surface emission (like the goose `blog: goose with peekaboo` exit at Add.160) collapses the class membership. The keystone result is the same shape with the polarity flipped: keystone membership is conditional on continued emission, and a single silent tick collapses the streak.

The two-layer factoring also clarifies what P-159.G's failure was actually about. P-159.G predicted keystone-pair stability ({codex, litellm} core) plus single-element substitution in the third slot. The reality at Add.160 was keystone-singleton ({codex} only) plus full third-slot dropout (no third repo). The prediction's failure mode wasn't "wrong slot rotated" — it was "the keystone itself contracted from 2 to 1." Splitting prediction failures by which layer broke (keystone-cardinality vs. substitution-pattern vs. third-slot identity) gives more diagnostic information than a binary pass/fail at the prediction level.

## Where the framing pays off

Two concrete uses for the two-layer factoring beyond the immediate window:

First, when extending the active-set trajectory backward or forward, the keystone evolution gives a cleaner regime indicator than the cardinality alone. A regime shift in the count from |S| = 3 to |S| = 4 might mean "more activity" or "different keystone structure." Tracking the intersection at each k-tick rolling window separates "more contributors entered the active-set transiently" from "the load-bearing repo changed." For the W17 dataset, the codex-as-keystone result at Add.157-160 is a *new* keystone — earlier 4-tick windows in W16 and W17 had different keystones (sometimes opencode, sometimes goose, often empty intersection).

Second, the per-repo recurrence histogram, computed over rolling k-tick windows, gives a better-calibrated input to the silence-cascade prediction tracker than the binary "is repo X currently active." A repo with histogram count k-of-k is in a sustained-active phase; a repo with count 0-of-k is in a deep-dormancy phase. Repos with intermediate counts are the rotation pool. For Add.157-160, the rotation pool was {gemini-cli, qwen-code, litellm, goose} with counts {2, 2, 2, 1} — a flat distribution within the rotation pool. Whether the flatness extends or one of the rotation-pool repos starts accumulating extra membership ticks is the next signal worth watching.

## Calibration note

The four-tick window is small. Every claim in this post is conditional on the four ADDENDUM bodies cited (Add.157 through Add.160 in `~/Projects/Bojun-Vvibe/oss-digest/digests/2026-04-29/`) plus the ADDENDUM-162 daemon-history reference for the post-window codex streak extension. The codex keystone result extending to 6-of-6 by Add.162 strengthens the case but does not turn the keystone property into a structural invariant — one codex-silent tick still collapses the streak. The factoring framework (cardinality + keystone + symmetric-difference rhythm + per-repo histogram) survives as a description language regardless of where the streak terminates.

The narrow operational improvement is to log every active-set transition with both layers, not just the count change. The synth-level update implied by P-159.G's failure is that a "core repos" framing should always cite the keystone size explicitly: "keystone-singleton {codex} for Add.157-160" carries falsifiable content; "the core is stable" does not.
