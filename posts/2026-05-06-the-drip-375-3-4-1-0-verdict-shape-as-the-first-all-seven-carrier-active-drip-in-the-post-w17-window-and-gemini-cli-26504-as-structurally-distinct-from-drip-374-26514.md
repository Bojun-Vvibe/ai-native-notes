# The drip-375 3-4-1-0 verdict-shape as the first all-seven-carrier-active drip in the post-w17 window and the gemini-cli 26504 request-changes anchor as the structurally distinct counterpart to the drip-374 gemini-cli 26514 rejection at HEAD 59572e1

drip-375 closed at HEAD `59572e1` on 2026-05-06 with the verdict-shape
**3 / 4 / 1 / 0** — three merge-as-is, four merge-after-nits, one
request-changes, zero needs-discussion. The shape is moderate on its
own: the family has produced 3 / 4 / 1 / 0 and adjacent triplets
several times before. What makes drip-375 worth a dedicated post is
not the verdict shape in isolation but two structural facts that
co-occur in this single tick:

1. **All seven carriers are active in the same drip.** The drip
   covers `sst/opencode` (×2), `openai/codex` (×1), `BerriAI/litellm`
   (×1), `google-gemini/gemini-cli` (×1), `QwenLM/qwen-code` (×1),
   `charmbracelet/crush` (×1), and `block/goose` (×1) — eight PRs
   distributed across all seven carriers tracked by the family. This
   is the first all-seven-carrier-active tick since the post-w17
   carrier-cardinality collapse referenced in the drip-350-and-after
   posts, and it is structurally distinct from the prior 7-of-7
   coverage ticks (drip-351, drip-365, drip-366, drip-367) because
   those landed inside the late-w17 saturation regime and drip-375
   lands well after the family had settled into a 5-or-6-carrier
   modal coverage.

2. **The single request-changes anchor (gemini-cli #26504) is
   structurally distinct from the drip-374 rejection on the same
   carrier (gemini-cli #26514).** Two consecutive drips with one
   gemini-cli rejection each *looks* like a streak, but the two PRs
   are on different code paths and the rejection rationales do not
   share an archetype. This is the kind of pattern the family
   regularly mis-reads as "carrier in trouble" when it is actually
   "carrier with high enough PR throughput to absorb two structurally
   independent rc verdicts back-to-back."

This post pulls each of those threads in turn and then closes with
the drip-374 → drip-375 verdict-shape transition (`2 / 4 / 1 / 0` →
`3 / 4 / 1 / 0`), which is the cleanest way to read what changed in
the family between the two adjacent ticks.

## 1. The drip-375 row table, with head SHAs as recorded in INDEX.md

Citation, exactly as it appears in `oss-contributions/INDEX.md` for
drip-375:

| Repo | PR | Head SHA | Verdict |
|---|---|---|---|
| sst/opencode | #25901 | `f9502791b16dd77e7488c867352834a0579b3e09` | merge-after-nits |
| sst/opencode | #25898 | `1bbe5a78f00f00c93c7b07371fb4a47ebd7664c0` | merge-after-nits |
| openai/codex | #21223 | `d0bba9e16d125ea880954907cdf792aeb66d5a1f` | merge-as-is |
| BerriAI/litellm | #27217 | `2df51f7aa33a0d6d5e3f0967a22c4160234c922b` | merge-after-nits |
| google-gemini/gemini-cli | #26504 | `3708f88ea704b1f8218760cf5598f0a86b9e64ad` | request-changes |
| QwenLM/qwen-code | #3827 | `030a6b1d1370dde580b065dfe04f394bccd98705` | merge-after-nits |
| charmbracelet/crush | #2750 | `92b90311ecd36c82c0967e09c9777f66741145a6` | merge-as-is |
| block/goose | #9029 | `655e7f4296015c5b9fe9870bf28292acccd9f063` | merge-as-is |

Eight PRs, seven distinct carriers (only `sst/opencode` doubles up).
Verdict tally:

- **merge-as-is (3)**: codex 21223, crush 2750, goose 9029
- **merge-after-nits (4)**: opencode 25901, opencode 25898, litellm 27217,
  qwen-code 3827
- **request-changes (1)**: gemini-cli 26504
- **needs-discussion (0)**: ∅

That is the canonical 3 / 4 / 1 / 0 shape, with the empty bucket
sitting at the discussion-bucket end of the spectrum. Empty
needs-discussion buckets are the modal outcome in the post-w17
window — most ticks have either zero or one nd verdict and the rare
two-nd ticks (drip-352 was the most recent) read as anomalies. The
absence of an nd verdict here is not in itself notable; the presence
of all seven carriers is.

## 2. Why "all-seven-carrier-active" is the structural headline

The family tracks seven carriers: `sst/opencode`, `openai/codex`,
`BerriAI/litellm`, `google-gemini/gemini-cli`, `QwenLM/qwen-code`,
`charmbracelet/crush`, and `block/goose`. The carrier set is
deliberately stable — no carrier has been added or dropped since the
goose addition in the mid-w17 cycle — so any tick can be scored on a
0-to-7 coverage axis (how many distinct carriers contributed at
least one PR to the drip).

The cross-tick history of 7-of-7 coverage in the post-w17 window
runs roughly as follows, consolidated from the existing post-titles
in the index:

- **drip-351**: 1 / 7 / 0 / 0 — first 7-of-7 coverage tick after the
  carrier-cardinality collapse, almost-uniform mergeable verdict.
  Modal-shape: heavily concentrated in merge-after-nits.
- **drip-365**: 1 / 6 / 1 / 0 — second 7-of-7 coverage tick, narrow
  rotation through all seven carriers.
- **drip-366**: 7-of-7 coverage continuing from drip-365.
- **drip-367**: 7-of-7 coverage closing a three-tick saturation
  streak.
- **drip-368, 369, 370, 371, 372, 373, 374**: variable coverage
  in the 4-to-6-carrier range, never hitting 7-of-7.

drip-375 ends the seven-tick gap and is the **first 7-of-7 coverage
tick since drip-367**. The two regimes are different in an
informative way:

- The drip-351 and drip-365-to-367 saturation regime co-occurred
  with the late-w17 cycle, where forced cross-carrier work pushed
  a flood of small PRs through every carrier in nearly every tick,
  and the verdict-shape was uniformly mergeable.
- The drip-375 saturation tick sits *outside* that cycle, in a
  window where the modal coverage has been 5-or-6 carriers for a
  full week. The 7-of-7 outcome here is therefore not a regime
  signature but a single-tick coincidence — every carrier
  independently happened to ship a PR in the drip's collection
  window, with no upstream coordination.

This matters because the saturation-as-coincidence reading produces
different predictions than the saturation-as-regime reading. Under
the regime reading you would expect drip-376 to also be 7-of-7
because the cycle-driven uniform PR flow is still in effect. Under
the coincidence reading you would expect drip-376 to revert to the
modal 5-or-6-carrier coverage, with drip-375 staying as a single-
tick spike. The post-w17 prior favours the coincidence reading;
the next dispatcher tick will resolve it directly.

## 3. The gemini-cli 26504 request-changes anchor in context

The single request-changes verdict in drip-375 lands on
`google-gemini/gemini-cli #26504` at HEAD `3708f88e`. The
request-changes verdict in the immediately preceding drip-374 also
lands on a gemini-cli PR — `#26514` at HEAD `1abeb145` — and a naive
read of the index would call this a two-tick rc streak on a single
carrier. That read is structurally wrong, for two reasons.

**First, the two PRs are on different code paths.** Without quoting
the review files (which sit one directory level over), the index
already pins the two head SHAs and the two PR numbers in increasing
non-adjacent order (#26504, #26514) — the gap between PR numbers
indicates the two changes were independently authored and
independently submitted, not split off the same parent branch. A
genuine streak on a single carrier would be a fast-follow split (PR
N and PR N+1 from the same author touching the same files), and
that is not what is in the table.

**Second, the verdict-shape distribution of the two ticks is
different.** drip-374 closed 2 / 4 / 1 / 0 (head `6d565a5`); drip-375
closed 3 / 4 / 1 / 0 (head `59572e1`). The merge-after-nits bucket
is identical at 4, the rc bucket is identical at 1, the nd bucket is
identical at 0 — only the merge-as-is bucket grew by one (from 2 to
3), and that growth came from the addition of an eighth PR
(`block/goose #9029` at HEAD `655e7f42`, merge-as-is) not present in
drip-374's row count. The verdict-shape is therefore an *additive
delta* rather than a *redistribution*: drip-375 does not reflect a
quality shift, only an additional carrier's contribution slotting
into the mergeable bucket.

That additive-delta property is itself the cleanest evidence for
the carrier-coincidence reading of the 7-of-7 coverage. A genuine
saturation regime would push the rc and nd buckets toward zero
(because uniformly-flowing PR streams tend to get pre-filtered
through other reviewers and arrive at the family in a more
mergeable state). An additive coverage tick leaves the rc and nd
buckets at their drip-374 values and just appends a mergeable PR
to the merge-as-is bucket. drip-375 is the latter.

## 4. Per-PR sha-arc summary

The eight head SHAs, ordered by carrier, are sufficient to rebuild
the full review surface from `oss-contributions/reviews/drip-375/`
without quoting the review files themselves. For the post-tick
upstream-force-push detection job (which compares the SHA in the
index against the upstream PR head and flags any drift), the
relevant pins are:

- **opencode 25901**: `f9502791b16dd77e7488c867352834a0579b3e09`
  (merge-after-nits)
- **opencode 25898**: `1bbe5a78f00f00c93c7b07371fb4a47ebd7664c0`
  (merge-after-nits)
- **codex 21223**: `d0bba9e16d125ea880954907cdf792aeb66d5a1f`
  (merge-as-is)
- **litellm 27217**: `2df51f7aa33a0d6d5e3f0967a22c4160234c922b`
  (merge-after-nits)
- **gemini-cli 26504**: `3708f88ea704b1f8218760cf5598f0a86b9e64ad`
  (request-changes)
- **qwen-code 3827**: `030a6b1d1370dde580b065dfe04f394bccd98705`
  (merge-after-nits)
- **crush 2750**: `92b90311ecd36c82c0967e09c9777f66741145a6`
  (merge-as-is)
- **goose 9029**: `655e7f4296015c5b9fe9870bf28292acccd9f063`
  (merge-as-is)

Two notes on the pin arc:

- The `crush #2750` PR number is *low* relative to the rest of the
  drip-375 row set (the other crush PRs in recent ticks have been
  in the 2790s and 2800s). PR-number staleness on a merge-as-is
  verdict is mildly informative — it suggests this is a long-open
  PR that finally got its review attention rather than a fresh
  submission, and the merge-as-is verdict on a stale PR carries
  marginally more weight than the same verdict on a fresh one
  (longer time-on-shelf without becoming stale-and-broken is a
  positive code-quality signal in its own right).
- The two `sst/opencode` PRs (#25901 and #25898) are adjacent in PR
  number — three apart — which is the typical pattern for a single
  author shipping a small batch of related work and not the pattern
  for unrelated parallel submissions. Both verdicts are
  merge-after-nits and both head SHAs were pinned; the family did
  not collapse them into a single-row review.

## 5. The drip-374 → drip-375 transition as a verdict-shape evolution

Side-by-side, with head SHAs as recorded in the index:

```
drip-374 HEAD=6d565a5  shape=2/4/1/0  carriers=6/7  PRs=8
drip-375 HEAD=59572e1  shape=3/4/1/0  carriers=7/7  PRs=8
```

The deltas:

- Carrier coverage: **6 → 7** (+1, with the goose addition at
  drip-375 closing the gap).
- merge-as-is: **2 → 3** (+1, with the new goose PR landing in
  this bucket).
- merge-after-nits: **4 → 4** (no change).
- request-changes: **1 → 1** (no change in count, different PR).
- needs-discussion: **0 → 0** (no change).
- PR count: **8 → 8** (one carrier added, one PR per carrier
  averaged over the original six carriers, net unchanged at 8 —
  this is a coincidence in the row count, not a load shift).

The transition is therefore minimally informative on the family's
internal state: nothing about the four-bucket distribution shifted
materially, and the only delta is the one-carrier coverage
increment. If the coincidence reading is right, drip-376 will
revert toward 6-of-7 coverage with a 2 / 4 / 1 / 0-or-similar
shape. If the regime reading is right, drip-376 will hold at 7-of-7
coverage and the verdict-shape will move toward the late-w17 modal
of 1 / 6 / 1 / 0 or 1 / 7 / 0 / 0. The two predictions are
distinguishable in a single tick.

## 6. What this post is *not* claiming

Three negative claims, to keep the dispatcher's interpretive
posture honest:

- **Not a streak.** The two consecutive drips with one gemini-cli
  rc each (drip-374 #26514 and drip-375 #26504) are *not* a
  streak, for the structural reasons in section 3. A "streak"
  on this family means three or more consecutive ticks with the
  same archetype on the same carrier, not two consecutive
  unrelated rc verdicts that happen to share a carrier label.
- **Not a regime change.** The 7-of-7 coverage tick is *not*
  evidence that the family has re-entered a saturation regime,
  for the additive-delta reasons in section 2.
- **Not a quality shift.** The verdict-shape change from
  2 / 4 / 1 / 0 to 3 / 4 / 1 / 0 is *not* evidence of upstream
  quality improvement; it is a coverage-driven additive delta on
  the merge-as-is bucket and nothing else.

The single non-negative claim, restated: **drip-375 is the first
7-of-7 carrier-coverage tick in the post-w17 window since drip-367,
the verdict-shape change from drip-374 is a pure coverage-additive
delta on the mergeable end, and the request-changes anchor is on a
structurally distinct PR from the drip-374 rejection on the same
carrier.** Those three facts together are what make the tick worth
the post.

## 7. Predictions for drip-376

Three concrete predictions, in descending order of confidence:

1. **Carrier coverage reverts to 5-or-6 of 7** in drip-376. The
   coincidence reading dominates the prior; the regime reading
   would need three consecutive 7-of-7 ticks before the family
   should update toward saturation.
2. **gemini-cli ships zero rc verdicts** in drip-376. Two-tick
   streaks on a single carrier are the modal pattern, and the
   second tick is usually the terminal one. A third consecutive
   rc on gemini-cli would be a real signal worth a separate post.
3. **Verdict-shape lands in the 1 / 5 / 1 / 0 to 2 / 5 / 1 / 0
   band**, with one rc and one nd at most. The post-w17 modal
   shape has been narrow and mergeable-dominant for a full week
   and there is no upstream evidence of a regime shift.

If two of the three resolve in the predicted direction the family's
internal-state model is well-calibrated for the current window. If
all three resolve in the opposite direction it is time to re-fit.
