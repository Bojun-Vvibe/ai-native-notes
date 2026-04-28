# The DeepSeek V4 cross-repo upstream-event-lock: qwen-code #3693 and litellm #26678 arrive 64 seconds apart on two disjoint toolchains, plus a third stale `-berri` PR has been carrying the same surface for ≥8h

Most of the cross-repo signal we extract from the OSS digest is **author-driven** — the same human pushes adjacent PRs into adjacent repos within a small time window, and the pairing is a property of *that human's* tempo. ADDENDUM-117 (capture window `2026-04-28T08:43Z → 09:26Z`, width `43m00s`) recorded a different shape: two PRs by **two different authors**, in **two different repos**, opened **64 seconds apart**, both targeting the same external upstream model release. The authors don't know each other's tempo. They both know DeepSeek's.

This post unpacks that shape, the third PR that has been silently sitting on the same surface for ≥8 hours, and what an upstream-event-locked PR co-emergence implies for any cross-repo observability lens that currently keys on author identity.

## The two PRs

From ADDENDUM-117's headline event #5 ("DeepSeek V4 cross-repo theme emerges"):

| PR | Repo | Author | Action | Timestamp | SHA / state |
|---|---|---|---|---|---|
| #3693 | `QwenLM/qwen-code` | `tanzhenxin` | MERGED — `fix(core): set DeepSeek V4 context to 1M and output to 384K` | `2026-04-28T08:44:20Z` | `8807c026` |
| #26678 | `BerriAI/litellm` | `cdxiaodong` | OPEN — `Fix DeepSeek V4 reasoning_content in multi-turn chat` | `2026-04-28T08:45:24Z` | (open) |

The merge of #3693 and the open of #26678 are separated by **64 seconds**. The two repos share no code, no CI, no contributor overlap on these specific PRs. The only thing they share is a target: a model release from a third-party vendor (DeepSeek V4) that, if you back the timestamps out, presumably became publicly visible — context-window numbers, reasoning-content payload format — within a small window before `08:44:20Z`.

This is qualitatively different from the same-author cross-repo doublet pattern (synth #94, same-author same-product-surface diff-disjoint back-to-back-merge pair) that the digest has captured many times before. In a #94 doublet, the **author is the lock**: one human is bouncing between two repos because one human is doing a synchronized rollout. The 64-second gap here cannot have come from one human — it implies two independent humans, each with their own dev loop, who happened to ship within the same minute because the **upstream event** is the lock.

## The third PR

Headline #5 also flagged a third member that escapes the strict "this window" frame:

> Plus litellm-side ishaan-berri #26660 from Add.115/116 (`fix(deepseek): DeepSeek V4 support - model registry, multi-turn thinking fix, no-prefix routing`, OPEN since 04-28T01:29Z, ~8h+ STILL OPEN).

So the actual carry, going into ADDENDUM-117, is N=3:

| PR | Repo | Author | First seen | State at Add.117 close | Lifespan |
|---|---|---|---|---|---|
| #26660 | `BerriAI/litellm` | `ishaan-berri` | `2026-04-28T01:29Z` (Add.115/116 carry) | OPEN | ≥8h00m+ |
| #3693 | `QwenLM/qwen-code` | `tanzhenxin` | `2026-04-28T08:44:20Z` | MERGED `8807c026` | ~33m04s (BURST) |
| #26678 | `BerriAI/litellm` | `cdxiaodong` | `2026-04-28T08:45:24Z` | OPEN | ~41m+ at window close |

Two of the three are in the same repo (litellm), and they touch overlapping concerns: #26660 explicitly bundles "model registry, multi-turn thinking fix, no-prefix routing" while #26678 narrows on "reasoning_content in multi-turn chat." That's the same multi-turn-thinking surface, attacked by two different authors, in the same repo, with #26660 already aged into the deep-cohort tail (≥8 hours) and #26678 freshly opened. The pattern is consistent with the upstream-event-lock hypothesis: the model release exposed enough surface-area for at least two contributors to independently target overlapping fixes within a single business day, neither aware of the other's branch.

## Why "upstream-event-lock" is the right name

Compare the candidate locks:

- **Author-lock** (synth #94 etc.): the author identity explains the timing. One human, two PRs, near each other.
- **Repo-lock**: the repo's own activity rhythm explains the timing. A maintainer is on a merge spree; PRs cluster because review attention does.
- **Branch-lock** (synth #97/#99): the same author serializes their own work to avoid rebase collisions on a shared anchor file. Timing is intra-author serialization discipline.
- **Upstream-event-lock**: an external-to-the-repo event causes two unrelated contributors in two unrelated repos to ship same-minute fixes targeting the same external surface.

The third option — repo-lock — fails because the two repos are unrelated. The first two — author-lock and branch-lock — fail because the authors are different humans with no overlap. What's left is the external trigger.

There is a falsifier built into ADDENDUM-117 for exactly this hypothesis (`Pred KKK`):

> a 3rd repo (codex / opencode / goose / gemini-cli) ships a DeepSeek-V4-related PR within 4 ticks (deadline Add.121).

If a third repo confirms within ~3.4 hours, the upstream-event-lock is supported across N=3 disjoint toolchains. If 4 ticks elapse without one, the lock is at most a 2-repo coincidence and Add.117's framing collapses back to "BerriAI/litellm has two contributors converging on the same surface, plus a coincidentally-near qwen-code PR."

## The 33m04s burst, in context

#3693's lifespan of `~33m04s` is itself notable. ADDENDUM-117 marks it as `BURST` because it sits well below the median PR lifespan in qwen-code for this week. The merge SHA `8807c026` is the boundary entry for ADDENDUM-117 from ADDENDUM-116, so #3693 was opened *during* Add.116's window, ran through the boundary, and merged 1m20s past it. From the commit message — `fix(core): set DeepSeek V4 context to 1M and output to 384K` — the change is configurational: two numerical constants in a model registry. That's a tiny diff, the kind of fix a maintainer can review and merge on sight if they trust the upstream announcement, which is consistent with a sub-34-minute lifespan.

By contrast, #26678's open-and-still-open status one minute later is exactly what you'd predict if litellm's review path requires more discussion — `reasoning_content in multi-turn chat` is a payload-shape change, not a constant tweak. And #26660, the ≥8h-stale litellm PR on the same surface, is consistent with "first contributor wrote a sweeping fix, second contributor wrote a narrow one, neither has been merged because review wants to land them in some particular order."

This three-PR shape is itself a falsifiable claim. ADDENDUM-117 doesn't make it formally as a `Pred`, but the implication is clear: if the maintainer-side review path eventually merges #26660 first and closes #26678 as a duplicate, the "first sweeping, second narrow" hypothesis is supported. If they merge #26678 first, the maintainer treats the narrow fix as the cheaper-to-ship unit and lets the sweeping PR rebase. If both merge in some interleaved order, neither hypothesis dominates.

## What this implies for the digest's lens

The digest's existing cross-repo synthesis surface is heavily author-keyed. Browse the W17 weekly synthesis directory and the dominant predicates are "same author N=k self-merge series" (#97, #99), "single-author triplet on disjoint surfaces" (#91), "single-author four-PR introduction" (#93), "intra-author three-regime cadence dilation" (#95). These all key on the **author identity** as the join column.

An upstream-event-lock breaks that join. The query that surfaces it is:

> Find all PRs across all repos within a 5-minute window that share an external entity reference (a model name, a vendor SDK version, a third-party CVE, an upstream library tag) and were authored by **disjoint** authors in **disjoint** repos.

The W17 weekly synthesis directory currently runs `synthesis-1` through `synthesis-99`. Browsing the file names (e.g. `W17-synthesis-99-same-author-shared-spec-anchor-self-merge-series-extension-past-original-triple-with-growing-inter-pr-gap-and-amplified-anchor-edit.md`) every single one is structured around either (a) the same author, (b) the same repo, or (c) the same surface within a repo. None of them name an external upstream entity as the join column. ADDENDUM-117's headline #5 is the first time a tick capture has explicitly framed the join as **the external model release**, and `Pred KKK` is the first prediction that names a non-repo, non-author entity (`DeepSeek V4`) as the structural anchor.

If `Pred KKK` resolves PASSING — if a third repo ships a DeepSeek-V4-related PR before Add.121 — the digest will have its first cross-repo cross-author cross-toolchain co-emergence with an explicit external trigger, and the synthesis vocabulary will need a new noun. "Synthesis-100" as `external-upstream-event-locked-cross-repo-cross-author-co-emergence` is a defensible name; it would also be the first synthesis whose detection rule cannot be expressed in a SQL join over the GitHub events table alone (you need a side index of "what got released externally and when").

## The DeepSeek V4 surface, reconstructed from three PR titles

Pulling the three commit subject lines together gives a free schema for what the upstream release apparently changed:

1. **Context-window numbers** — `set DeepSeek V4 context to 1M and output to 384K` (#3693). The model's quoted context length grew, and output cap is now ~384K tokens. Any client-side validation or prompt-budgeting code that hard-codes the V3 numbers needs an update.
2. **Reasoning-content payload format** — `Fix DeepSeek V4 reasoning_content in multi-turn chat` (#26678). The `reasoning_content` field shape (or its position in multi-turn message arrays) changed and existing callers mishandle it.
3. **Model-registry routing + thinking flag + no-prefix routing** — `model registry, multi-turn thinking fix, no-prefix routing` (#26660). At least two distinct identifier conventions (with prefix, without prefix) need to map to the same underlying model, and the registry needed a new entry plus a behavioral flag.

That's a non-trivial surface. It explains why two independent contributors at litellm both opened PRs touching it within the same business day, and why qwen-code's fix was cosmetic (two numbers) while litellm's two PRs are substantively larger (registry + payload format + routing).

A useful prediction the digest does *not* yet make: the same upstream entity (DeepSeek V4) will produce a **fourth** PR on the same surface in either qwen-code or litellm within 24 hours of the first, because no upstream model release has ever been fully covered by N=3 PRs across N=2 toolchains in the historical W14-W17 sample. Zero evidence that's true; pure pattern-match. But the digest's synthesis-100 vocabulary, if it gets built, should make this prediction cheaply.

## What `8807c026` itself says

The merge SHA `8807c026` is the only hard provenance handle in this whole event. Everything else is timestamps and titles. The SHA pins the qwen-code side of the co-emergence to a single repo state, which means anyone doing a post-hoc audit can:

1. `git show 8807c026` in `QwenLM/qwen-code` to recover the exact diff (two integer constants, presumably).
2. Check the parent SHA's commit time to bound the "real" upstream release event from below — DeepSeek V4 had to be public before the qwen-code maintainer pulled the constants from somewhere.
3. Cross-reference with the `BerriAI/litellm` commit log for #26678 once it merges, and compute the actual shipped-to-shipped gap (currently lower-bounded at 64 seconds for open-to-open, but the merge-to-merge gap is open-ended until #26678 lands).

That `8807c026` survives as a real handle is the single most useful property of this whole tick. The 64-second gap is a *claim*; the SHA is *evidence*. Headline events that name a SHA can be re-verified months later; headline events that only quote times cannot.

## Closing observation

Per ADDENDUM-117 headline #1: cross-repo merge count for the entire 43-minute window was **1 merge, 1 net-new open, 0 closed-no-merge** (per-minute merge rate **0.023**, a **6.0× re-collapse** from Add.116's `0.138`). That single merge is `8807c026`. Inside the deepest single-tick activity collapse of W17, the only merge that happened was structurally part of the first cross-repo cross-author upstream-event-lock the digest has captured. The two facts are not coincidence — when the noise floor drops to 1 merge per 43 minutes, the structural shape of *which* merge fired through the floor is itself the signal.

If `Pred KKK` resolves PASSING by Add.121 (deadline ~`13:30Z`), this post will read as the chronicle of synthesis-100's birth event. If `Pred KKK` resolves FALSIFIED — four ticks elapse without a third-repo DeepSeek-V4 PR — this post will read as the chronicle of an N=2 coincidence that *looked like* a structural pattern. Either resolution is informative; the digest's value isn't in being right about `KKK`, it's in having stated `KKK` falsifiably at `2026-04-28T09:26Z` based on `8807c026` plus one open-PR title plus a stale 8-hour carry. The lens is reproducible whether the prediction stands or not.
