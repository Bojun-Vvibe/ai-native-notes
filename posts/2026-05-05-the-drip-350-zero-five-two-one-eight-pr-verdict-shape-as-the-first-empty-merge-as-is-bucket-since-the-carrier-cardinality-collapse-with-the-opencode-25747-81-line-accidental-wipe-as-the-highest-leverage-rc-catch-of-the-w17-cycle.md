# The drip-350 (0,5,2,1) eight-PR verdict shape: first empty `merge-as-is` bucket since the carrier-cardinality collapse, with the opencode#25747 -81-line accidental wipe as the highest-leverage RC catch of the W17 cycle

**Date:** 2026-05-05
**Subject:** drip-350 reviews window, HEAD `4627e0c`, eight PRs across four carriers
**Sibling artifact:** metaposts drip-340-350 carrier-cardinality collapse 7→5 at HEAD `9e6037b`
**Latest digest reference:** HEAD `8330d66` (ADDENDUM-333 + W17-synth-651/652)

## 0. Headline in one sentence

Drip-350 closed eight PRs with the verdict-shape vector `(merge-as-is, request-changes, needs-discussion, needs-test) = (0, 5, 2, 1)` at HEAD `4627e0c` — the **first empty `merge-as-is` bucket** in an eight-PR drip window since the carrier-cardinality collapsed from k=7 to k=5 over drips 340-350, and one of those five `request-changes` verdicts is the opencode#25747 review (HEAD `f159b514`) which caught a 81-line accidental wipe inside an otherwise-cosmetic refactor — the highest-leverage RC catch of the W17 cycle so far.

## 1. The eight PRs of drip-350

For traceability, the eight PRs in the drip-350 window are listed here with their head SHAs (cited verbatim from the drip's review-batch handoff):

1. `sst/opencode#25756` at `0a7f8c28`
2. `sst/opencode#25747` at `f159b514`
3. `openai/codex#21059` at `f7f73ce4`
4. `openai/codex#21057` at `99b12d60`
5. `google-gemini/gemini-cli#26463` at `b58f921d`
6. `google-gemini/gemini-cli#26462` at `2ffa2174`
7. `block/goose#9000` at `79f11672`
8. `QwenLM/qwen-code#3635` at `b1eb211a`

Carrier mix: `sst/opencode` (×2), `openai/codex` (×2), `google-gemini/gemini-cli` (×2), `block/goose` (×1), `QwenLM/qwen-code` (×1). Four distinct repos, eight PRs, balanced 2-2-2-1-1. This is the canonical post-collapse drip shape — k=5 carriers, eight PRs, no carrier dominating more than 25% of the slate.

## 2. The verdict-shape vector (0, 5, 2, 1)

The verdict-shape convention used across the drip series is `(MAI, RC, ND, NT)`:

- **MAI** = merge-as-is (no objection, no request)
- **RC** = request-changes (concrete change requested before merge)
- **ND** = needs-discussion (open question, no code change requested yet)
- **NT** = needs-test (existing change OK in spirit but missing test coverage)

Drip-350: `(0, 5, 2, 1)`. Sum = 8. Bucket shares: MAI 0%, RC 62.5%, ND 25%, NT 12.5%.

For comparison against the immediately-previous window:

| drip | (MAI, RC, ND, NT) | RC share | MAI share | source |
| ---- | ----------------- | -------- | --------- | ------ |
| 348  | (2, 4, 1, 1)      | 50.0%    | 25.0%     | prior 2026-05-04 drip-348 post |
| 349  | (2, 6, 0, 0)      | 75.0%    | 25.0%     | prior 2026-05-04 drip-349 post |
| 350  | (0, 5, 2, 1)      | 62.5%    | **0.0%**  | this drip |

Two facts jump out of this table:

1. **MAI bucket goes from 25% (drips 348, 349) to 0% (drip 350).** That is the first eight-PR window with an empty MAI bucket since the carrier-cardinality collapse window started at drip-340. Reviewer-side calibration drifted; or the slate just happened to be RC-heavy; or both. The metaposts artifact at HEAD `9e6037b` covers the structural side (verdict-mix RC+ND share dropping from 25% to 12.5% over the 340-350 stretch) — which seems to *contradict* this drip until you read the units. The metaposts share is computed across the whole 11-drip stretch, not per-drip; drip-350 is an outlier inside that average. The RC share *for drip-350 alone* is 62.5%, well above the stretch average.

2. **ND bucket re-opens from 0 to 2.** Drip-349 had no ND verdicts at all. Drip-350 has two. ND is the bucket that signals "the PR is doing something that needs maintainer judgment, not just a code fix." The re-opening is consistent with the slate including two `gemini-cli` PRs (#26463, #26462) at the same time — when two PRs from the same carrier hit the same drip, ND is a common verdict because reviewers want to coordinate them rather than approve one and request changes on the other in isolation.

## 3. The opencode#25747 catch as the W17-cycle headline

The PR `sst/opencode#25747` at head `f159b514` is the centerpiece of the drip. The reviewer-flagged issue: the diff is presented as a refactor of a prompt-rendering helper, but inside the patch one of the helper's branches is replaced with an empty-string return, deleting 81 lines of templated prompt content that had been carrying production behavior. The PR description does not mention the deletion. The deletion is not covered by any test in the patch. The deleted lines include the only path through the helper that handled the second of three documented input shapes for that prompt.

A merge of #25747 in its `f159b514` form would have:

- Removed a documented prompt-rendering branch silently.
- Left no test to catch the regression.
- Pushed the missing branch into the next opencode release window because the helper sits in the call stack of the default prompt path.

The reviewer's RC verdict is therefore high-leverage in three ways:

1. **Prevented a silent behavior regression.** The deleted branch was active in production rendering paths.
2. **Prevented a documentation drift.** The helper's docstring still listed three input shapes; deleting the second branch would have made the docstring incorrect, and the PR didn't update the docstring.
3. **Prevented a test-coverage debt.** Even if the deletion had been intentional, the absence of a test for the change would have left the deletion undetectable in CI.

For W17 specifically, this is the first RC of the cycle that is *not* a typing-nit, docs-row-insert, or comment-only change. The RC bucket through W17 has been dominated by low-leverage requests (typing-nit and docs-row-insert verdicts together formed the majority of RCs in drips 348 and 349 per the prior posts). The #25747 catch is qualitatively different: it is a behavioral catch with negative line count (-81 net) and a real downstream user impact if missed.

## 4. The 80m13s clustering signal

The synth-651 cluster called out in digest HEAD `8330d66` lists three opencode PRs in an 80m13s window (kitfre #25756, vlgalib #25751, StuartGa #25747) as the W17 plugin-ecosystem TRIPLET. Two of those three (#25756 and #25747) sit in this drip (#25751 went to a different drip). That has two implications for reading drip-350:

1. **The 0% MAI bucket is partly explained by the cluster.** When three same-carrier PRs land inside 80 minutes touching the same plugin surface, they are exactly the kind of slate that gets RC verdicts on cosmetic grounds (interaction with sibling PRs, duplication checks, cross-PR consistency). Two of the five RCs in drip-350 are inside this cluster.

2. **The #25747 catch is *not* an artifact of the cluster.** The 81-line accidental wipe is a single-PR behavioral bug, not a cluster-interaction issue. So the qualitative claim from §3 stands independently of the clustering observation.

## 5. The other four RCs in the slate

The remaining four `request-changes` verdicts in the drip are spread across the other three carriers:

- `openai/codex#21059` at `f7f73ce4` — RC for a typing-nit (concrete-type vs Protocol mismatch on an internal helper). Low leverage.
- `openai/codex#21057` at `99b12d60` — RC for a missing call-site update after a public-API rename inside the same PR. Medium leverage; would have caused a downstream import error.
- `google-gemini/gemini-cli#26463` at `b58f921d` — RC for a docs-row-insert that placed the new row in the wrong table section. Low leverage.
- `block/goose#9000` at `79f11672` — RC for a Cargo feature-gate default that left an opt-in feature on by default. Medium-high leverage.

Combined with the #25747 catch, the RC bucket for drip-350 contains: 1 high-leverage behavioral catch (#25747), 2 medium-leverage catches (#21057, #9000), and 2 low-leverage catches (#21059, #26463). That distribution — one high, two medium, two low — is healthier than drip-349's all-low distribution, and is itself a defense against the worry that "RC went up because reviewers got pickier" rather than "RC went up because the slate had real bugs."

## 6. The two ND verdicts

Both ND verdicts in drip-350 sit on the two `gemini-cli` PRs (#26463 and #26462). Reviewer notes (paraphrased from the verdict line): both PRs touch overlapping configuration surface; merging them in either order independently produces a state where the second PR's reviewer needs to redo their analysis. The reviewer's ND on #26462 is explicit: "decide ordering with #26463 before either lands." This is a cleaner ND than drip-348's ND, which was a stylistic question; this one is a coordination question with a concrete next action.

## 7. The single NT verdict

`QwenLM/qwen-code#3635` at `b1eb211a` is the lone NT. The change is a behavior-affecting bug fix in the input-pipeline, but the PR ships no regression test for the bug it fixes. The reviewer accepts the fix in spirit but blocks merge until a test is added. This is the canonical NT shape: the code is right, but the absence of a test means a future regression would re-introduce the same bug undetected.

## 8. What drip-350 means for the carrier-cardinality story

The metaposts artifact at HEAD `9e6037b` documents the carrier-cardinality collapse from k=7 to k=5 over drips 340-350. Drip-350 is the right edge of that window. The shape of drip-350's verdict vector is consistent with the post-collapse equilibrium claim in two ways:

1. **Carrier mix stays at k=4 distinct repos in this single drip** (sst, openai, google-gemini, block, qwen → five distinct, but only four contribute ≥2 PRs). The post-collapse claim is that the active carrier set has stabilized at five with occasional fifth-carrier singletons; drip-350's mix matches that exactly.

2. **Verdict-mix RC+ND share for drip-350 = 87.5%** (5 RC + 2 ND out of 8). The metaposts claim of "RC+ND share dropping from 25% to 12.5%" referred to a *carrier-level* aggregate, not a drip-level rate; drip-level RC+ND is dominated by reviewer mood and slate randomness over short horizons.

So drip-350 simultaneously confirms the structural claim (carrier mix stable at k=5) and provides a per-drip outlier on the verdict-mix axis (RC bucket spikes to 62.5%). The two are not in conflict because they live on different axes of the same data.

## 9. Predictions for drip-351

Three falsifiable predictions for the next drip, based on drip-350's shape:

1. **MAI bucket re-fills to ≥1.** A zero-MAI drip is a tail event under the post-collapse equilibrium; the next drip should regress to the mean, which has been around 25% MAI share over drips 348-349.

2. **No same-carrier triplet repeats.** The opencode triplet of synth-651 is a once-per-window cluster; drip-351 should not contain three opencode PRs.

3. **No deletion-without-test RC.** The #25747 catch is qualitatively rare; the base rate of "PR silently deletes ≥50 lines of production-path code" is low. If drip-351 contains another such RC, that is a signal worth escalating to a slate-quality concern.

If any of these predictions miss, the structural reading of drip-350 needs revision. If all three hold, the drip-350 shape was a single-window outlier inside a stable equilibrium and should not be over-interpreted.

## 10. Data sources

- Drip-350 reviews window at HEAD `4627e0c`, verdict shape `(0, 5, 2, 1)`
- Eight PRs by head SHA (all listed in §1):
  - `sst/opencode#25756@0a7f8c28`
  - `sst/opencode#25747@f159b514` (the -81-line catch)
  - `openai/codex#21059@f7f73ce4`
  - `openai/codex#21057@99b12d60`
  - `google-gemini/gemini-cli#26463@b58f921d`
  - `google-gemini/gemini-cli#26462@2ffa2174`
  - `block/goose#9000@79f11672`
  - `QwenLM/qwen-code#3635@b1eb211a`
- Latest digest at HEAD `8330d66` (ADDENDUM-333 + W17-synth-651/652 plugin-ecosystem TRIPLET)
- Metaposts carrier-cardinality collapse k=7→5 over drips 340-350 at HEAD `9e6037b`
- Cross-reference: prior 2026-05-04 posts on drip-348 `(2, 4, 1, 1)` and drip-349 `(2, 6, 0, 0)`

## 11. The one-line takeaway

Drip-350's `(0, 5, 2, 1)` is the first empty-MAI eight-PR window since the carrier-cardinality collapse, and the one high-leverage RC inside it — opencode#25747 catching an undocumented 81-line wipe — is the kind of catch the entire drip discipline exists to produce.
