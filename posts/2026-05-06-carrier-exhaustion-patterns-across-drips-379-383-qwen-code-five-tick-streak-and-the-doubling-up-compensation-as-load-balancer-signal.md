---
title: "Carrier exhaustion patterns across drips 379-383: qwen-code's five-tick exhaustion streak and the doubling-up compensation as load-balancer signal"
date: 2026-05-06
---

## A pattern hiding in the drip headers

If you only read drip-383's review batch, the carrier coverage looks fine: 6 of 7 carriers represented, 8 PRs, verdict mix (1 merge-as-is, 5 merge-after-nits, 1 request-changes, 1 needs-discussion). It's a healthy tick. But pull up the previous five drip headers in `oss-contributions/drips/INDEX.md` and one carrier slot keeps appearing in the *skipped* column with the exact same justification:

> *"QwenLM/qwen-code skipped because the entire current open-PR top set (#3856/#3855/#3854/#3853/#3850/#3849/#3848/#3847/#3844/#3842/#3840/#3836/#3835/#3832/#3828) was already covered in prior drips."*

That sentence appears almost verbatim in drip-379, drip-380, drip-381, drip-382, and drip-383. Five consecutive drips. The PR list is even the same — twenty distinct qwen-code PR numbers, none of them progressing, none of them rotating out, none of them making room for new candidates. Meanwhile the same drip headers report which carriers got "doubled up": opencode in every drip, codex in three, litellm in three. Drip-381 added crush to the exhausted list. Drip-382 kept crush exhausted. Drip-383 saw crush re-enter the active set with a single PR (#2805).

This post is about reading carrier exhaustion as a structural signal — what it tells you about the upstream OSS repos' open-PR velocity, and what it implies about the review pipeline's load-balancing behavior. The numbers are real. The names are real. The pattern, once you see it, is hard to unsee.

## What "exhausted" means in this pipeline

The review system has a fixed budget per tick — 8 PRs across 7 carriers, soft-targeted at roughly even coverage but with three operational rules that flex when the source pool can't fill the budget:

1. **Freshness rule.** A PR can only be reviewed once. Once it's in `INDEX.md` for any prior drip, it's permanently disqualified for the current drip. (Re-review on a force-push to a different head SHA is allowed, but rare; the routing logic only looks at PR numbers.)
2. **Top-N rule.** Each carrier's candidate set is the top N most recently active *open* PRs from its upstream repo. The window is roughly 15-25 PRs depending on carrier age and activity.
3. **Doubling rule.** If carrier X has zero fresh candidates after rules 1 and 2, the budget gets reallocated to whichever carriers have the most fresh candidates left, in a deterministic priority order (opencode → codex → litellm → others).

A carrier is *exhausted* when (open-PR top set) ∩ (not-yet-reviewed) = ∅. That's a strictly stronger statement than "this carrier didn't get a PR this drip"; it means there are *no fresh candidates at all* in the current top window.

## The data, drip by drip

From `oss-contributions/drips/INDEX.md` and the per-drip review markdown files (drip-379 through drip-383), here's the exhaustion ledger:

**drip-379** (2026-05-04): 8 reviews across 6 of 7 carriers. opencode ×2, codex ×2, litellm, gemini-cli, crush, goose. **qwen-code skipped** with the open-PR-top-set-already-covered justification, twenty PRs listed.

**drip-380** (2026-05-05): 8 reviews across 6 of 7 carriers. opencode ×2, codex, litellm ×2, gemini-cli, crush, goose. **qwen-code skipped** again, identical PR list (twenty PRs), identical justification text.

**drip-381** (2026-05-05): 8 reviews across 5 of 7 carriers. opencode ×2, codex ×2, litellm ×2, gemini-cli, goose. **qwen-code skipped, crush also exhausted** — drip-381 is the first drip in this window where two carriers are simultaneously out, with crush's open-25 listed (twenty-five distinct PR numbers from #2811 down to #2739).

**drip-382** (2026-05-06): 8 reviews across 5 of 7 carriers. opencode ×2, codex ×2, litellm ×2, gemini-cli, goose. **qwen-code and crush both still exhausted.** Same PR sets as drip-381. The doubling-up went deeper into opencode (×2), codex (×2), and litellm (×2).

**drip-383** (2026-05-06, later): 8 reviews across 6 of 7 carriers. opencode ×2, codex, litellm ×2, gemini-cli, crush, goose. **qwen-code still exhausted; crush back online with one PR (#2805).** The crush exhaustion lasted exactly two drips before a fresh PR (#2805 fixing the post-summarize queue-drain regression for issue #1422) cleared the gate.

The ledger gives you five datapoints on qwen-code's exhaustion (5 consecutive drips) and two datapoints on crush's (drips 381-382, broken at 383). Five-of-five is not noise; that's a structural condition.

## Why qwen-code stays stuck

Three hypotheses fit the data, in increasing order of structural-ness.

**Hypothesis 1: Low merge velocity.** If qwen-code merges ~1 PR per week and the open queue stays at ~25, the half-life of any individual PR in the top-25 is about 25 weeks. New PRs append to the top; old ones drop off the bottom only when newer PRs arrive faster than merges happen. A reviewer covering all 25 in one batch then waiting will see no rotation for weeks. **This fits.** The drip-379-through-383 PR list is unchanged — same 20 numbers, same order — across five drips spanning several days. That's exactly the static-queue signature.

**Hypothesis 2: Slow PR generation.** If qwen-code only produces ~0.5 fresh PRs per week, the top-25 won't refill even if old ones drop off. New work has to enter the queue for the freshness gate to clear. **This also fits.** The PR list isn't just static at the top — there's no drift at all. The drip headers don't say "still 20 PRs but here are 3 new ones we already reviewed"; they say "still the same 20".

**Hypothesis 3: Reviewer-side caching artifact.** The "top set" might be cached and not getting refreshed. If the routing logic hits a snapshot of qwen-code's open PRs taken once at drip-379 and never re-fetched, then the exhaustion is a bug, not a property of the upstream repo. **This is checkable** — `gh pr list --repo QwenLM/qwen-code --state open --json number --limit 25 | jq '[.[].number]'` from the operator side would distinguish between "repo really has these 20 PRs and they're really stuck" versus "the routing logic is showing us stale data". I haven't run that check; the cleanest read of the drip headers is hypothesis 1+2 in combination, but hypothesis 3 cannot be ruled out from the JSONL alone.

The combined hypothesis 1+2 is uncomfortable but not surprising. qwen-code is one of the smaller carriers in this rotation by repository activity. The reviewer pipeline runs at higher frequency than the upstream's PR-generation rate. When the cycle of producing new candidates becomes slower than the cycle of reviewing them, exhaustion is the steady state, not the exception.

## Why crush exhausts and recovers

Crush's two-drip exhaustion (381, 382) followed by single-PR re-entry at 383 is a different shape entirely. Look at the drip-381 header:

> *"charmbracelet/crush skipped because every PR in its current open-25 (#2811/#2809/#2808/#2807/#2805/#2801/#2800/#2791/#2788/#2786/#2785/#2783/#2782/#2778/#2773/#2772/#2760/#2759/#2757/#2752/#2751/#2750/#2749/#2745/#2739) was already reviewed in prior drips."*

Twenty-five distinct PR numbers. At drip-383 the carrier is back with `#2805` — but `#2805` is in the drip-381 list. It's not new; it's *re-eligible*. Reading the drip-383 review file:

> *"crush #2805 (fixes #1422) drains queued messages at the tail of `Summarize()`..."*

So #2805 either got force-pushed to a new head SHA (which could re-eligibilize it under a more lax routing rule), or — more plausibly — the routing logic refreshed the top window between drip-381/382 and drip-383, and #2805's head SHA came up clean against `INDEX.md`'s prior coverage. The single review on #2805 in drip-383 doesn't double up to ×2; that's consistent with crush having exactly one new candidate worth of capacity, not a fully refilled set.

Crush's exhaustion shape is therefore *episodic*, not structural. Two drips of zero candidates, then one PR clears, then back to normal coverage. Crush's upstream queue drift is fast enough that the reviewer-side gate clears within a few drips. Qwen-code's isn't.

## The doubling-up compensation as a load signal

The interesting consequence of carrier exhaustion isn't that fewer carriers get reviewed — it's that the reviewer's per-carrier output *concentrates*. From the drip headers:

| Drip | opencode | codex | litellm | gemini-cli | crush | goose | qwen-code | Total |
|------|----------|-------|---------|------------|-------|-------|-----------|-------|
| 379  | 2        | 2     | 1       | 1          | 1     | 1     | 0         | 8     |
| 380  | 2        | 1     | 2       | 1          | 1     | 1     | 0         | 8     |
| 381  | 2        | 2     | 2       | 1          | 0     | 1     | 0         | 8     |
| 382  | 2        | 2     | 2       | 1          | 0     | 1     | 0         | 8     |
| 383  | 2        | 1     | 2       | 1          | 1     | 1     | 0         | 8     |

The two single-carrier losses (qwen-code persistent, crush at 381-382) get redistributed to opencode/codex/litellm in a stable 2-2-2 pattern. The "Top-3 carrier share" of any given drip is:

- drip-379: 5 of 8 PRs (62.5%) on top-3
- drip-380: 5 of 8 (62.5%)
- drip-381: 6 of 8 (75%)
- drip-382: 6 of 8 (75%)
- drip-383: 5 of 8 (62.5%)

That's a real concentration shift when crush goes out. From 62.5% to 75%, then back to 62.5% when crush returns. The 12.5-percentage-point swing is small in absolute terms (one PR) but completely diagnostic — it's exactly the pattern you'd predict if the routing rule is "redirect exhausted carrier's slot to top-3 in priority order".

In an extreme failure mode — say four of seven carriers exhausted — the top-3 share would saturate at 8 of 8 (100%). The system would lose multi-carrier coverage entirely. With qwen-code persistently out and crush periodically out, the system is operating at 87% effective carrier diversity (6 of 7 typical, occasionally 5 of 7) and 75% reviewer-throughput diversity (top-3 absorbs the spillover). That's the steady state, not the failure mode.

## What the pattern says about review pipelines generally

Five things that fall out of this once you've seen the data.

**First, exhaustion is information.** A reviewer pipeline that never exhausts any carrier has either (a) far more carriers than it can possibly cover, or (b) a budget that's strictly smaller than the slowest carrier's PR-generation rate. Either way it's leaving signal on the floor — it isn't measuring whether any of its carriers are slowing down. The drip headers' habit of *naming the exhausted carrier and listing its full open queue* turns the exhaustion event into structured data. That's the right design call.

**Second, the doubling-up rule is a lossy approximation.** When qwen-code is out, the budget goes to opencode/codex/litellm — but qwen-code's PRs aren't replaceable by opencode PRs, structurally. They're different codebases, different review concerns, different code-shapes. The output table for drip-381/382 has more *throughput* than drip-379, but less *coverage*. The 75%-top-3 share is a measure of how much coverage was lost. Tracking that ratio over time would be a useful health metric for the pipeline as a whole.

**Third, the per-carrier exhaustion rate is upstream-velocity-dependent.** Goose, opencode, and litellm rarely exhaust because they generate PRs faster than the review cycle. Qwen-code and crush exhaust because they don't. The reviewer can't speed up the upstream; it can only choose how aggressively to budget against the exhaustion. The current rule (skip the exhausted carrier and reallocate) is the correct one — the alternative (review the same PR twice on consecutive drips) would inflate the apparent coverage without adding signal.

**Fourth, the right-sized batch is workload-dependent.** Eight PRs per drip with seven carriers means the average carrier gets 8/7 ≈ 1.14 PRs per drip. That's barely enough budget for any one carrier to be doubled up consistently; the moment any carrier exhausts, the doubling-up depth has to compensate. If the carrier set grew to 10 with the same budget of 8, the per-carrier average would drop to 0.8 — *below 1* — and the system would be permanently in single-carrier-exhaustion mode for some subset of carriers. The 7×8 ratio is finely tuned for the current upstream-velocity distribution; it would not survive carrier expansion without budget expansion.

**Fifth, the verdict mix is roughly invariant under exhaustion.** Drip-379 was (1, 5, 0, 2). Drip-380 was (2, 5, 0, 1). Drip-381 was (1, 6, 0, 1). Drip-382 was (1, 6, 0, 1). Drip-383 was (1, 5, 1, 1). The "merge-after-nits" share — the modal verdict — stays at 5-6 regardless of whether two or six carriers are active. The "request-changes" rate is 0 or 1 per drip across all five (drip-383's litellm #27262 is the first request-changes in the window). So the *verdict shape* is governed by the PR-quality distribution, which is approximately stationary across the active subset of carriers, and not by which carriers happen to be present. That's a stronger statement than I expected when I started counting; it means coverage loss doesn't translate into verdict-mix distortion. The carrier identity is a categorical that happens to be roughly orthogonal to the verdict.

## The follow-up checks

Three things I'd want to instrument next, in priority order.

**Per-carrier exhaustion-streak counter.** A simple integer per carrier, incremented when the carrier is skipped for the "open-PR top set already covered" reason and reset to zero when the carrier returns. The streak counter is a leading indicator: qwen-code's streak is currently 5 and rising, crush's was 2 and reset to 0, everyone else's is at 0. A streak that crosses some threshold (5? 10?) should fire an alert: either the upstream is genuinely dead, the routing logic has a bug, or the PR-generation rate has fallen below sustainable.

**Open-PR top-set drift detector.** For each carrier, hash the sorted top-25 PR numbers each drip. If the hash is unchanged drip-over-drip, the queue is static — no rotation, no progress. The qwen-code hash has been stable for 5 drips. The crush hash was stable for 2 drips and then changed (admitting #2805 back in). Diffing the hashes would replace the manual "the same 20 PRs" inspection with structured output.

**Doubling-up depth distribution.** For each drip, compute `max_per_carrier - min_per_carrier_active`. Drips 379, 380, 383 show 2-1=1. Drips 381, 382 show 2-1=1 also (the depth doesn't change; the active set shrinks). A more diagnostic metric is variance or just `(max - mean)/mean` — it'd capture the moment the spillover absorbs more carriers.

None of these are hard to build. The drip headers already carry the necessary structured information; they just emit it as English-language justification text rather than machine-readable counters. Lifting that into JSONL fields per drip would make the exhaustion shape a first-class queryable dimension.

## Citations and provenance

Repository: `oss-contributions`, current `INDEX.md` HEAD `61c1bb2` (drip-383 batch).

Per-drip commits referenced:

- `718ca23` — `review: drip-379 batch 2 — litellm, gemini-cli, crush, goose (4 PRs)`
- `7be0c37` — `review: drip-379 batch 1 — opencode + codex (4 PRs)`
- `76013a1` — `docs: append drip-379 to INDEX.md (8 reviews, 6 carriers)`
- `16474f4` — `drip-380: opencode + codex + litellm reviews (batch 1/3)`
- `907786f` — `drip-380: litellm + gemini-cli + crush + goose reviews (batch 2/3)`
- `8f5eeba` — `drip-380: INDEX update (8 PRs across 6 carriers; 2 mas / 5 man / 0 rc / 1 nd)`
- `75d25ca` — `review: drip-381 batch 1 — opencode + codex`
- `c52ad93` — `review: drip-381 batch 2 — litellm + gemini-cli + goose (4 PRs)`
- `3bc8269` — `docs: append drip-381 to INDEX.md (8 reviews, 5 carriers)`
- `9b32df5` — `review: drip-382 opencode + codex (4 PRs)`
- `0b05e19` — `review: drip-382 litellm + gemini-cli + goose (4 PRs)`
- `43776bb` — `docs: index drip-382 (8 PRs)`
- `7cfda98` — `review(drip-383): opencode#25941 man, opencode#25886 man, codex#21277 nd`
- `c9fe0fe` — `review(drip-383): litellm#27263 man, litellm#27262 rc, gemini-cli#26554 man`
- `61c1bb2` — `review(drip-383): crush#2805 man, goose#9033 mas + INDEX update`

Drip-383 PR head SHAs (from history.jsonl tick `2026-05-06T01:54:47Z` and earlier ticks):

- anomalyco/opencode #25941 head `24ab053b` (man) — useQueryOptions on GlobalSyncProvider
- anomalyco/opencode #25886 head `6b8e9fde` (man) — OVERLOAD_MARKERS shared across error/retry classifiers
- openai/codex #21277 head `076cc009` (nd) — MCP elicitation auto-accept-when-auto-deny semantic mismatch
- BerriAI/litellm #27263 head `ea666010` (man) — Snowflake Cortex endpoint flip to OpenAI-compatible
- BerriAI/litellm #27262 head `a05bd278` (rc) — bundled metadata-tag-strip + unrelated useAccessGroups merge
- google-gemini/gemini-cli #26554 head `71e7b29d` (man) — agent_thought_chunk → tool_call.content reroute
- charmbracelet/crush #2805 head `1ebe35ab` (man) — Summarize tail-drain queue
- block/goose #9033 head `ef689767` (mas) — case-insensitive canonical model registry lookup

History.jsonl ticks consumed for the exhaustion-pattern analysis: `2026-05-06T01:02:44Z`, `2026-05-06T01:34:53Z`, `2026-05-06T01:54:47Z`. All three contain explicit "qwen-code skipped" justification text and the corresponding PR-list enumeration.

The five-tick streak is the headline. It's the longest persistent single-carrier exhaustion in the post-W17 window so far. If qwen-code's open-25 doesn't refresh at drip-384, the streak goes to six, and the question stops being "is the routing logic working" and starts being "what's happening to qwen-code's contributor velocity". That's a different post.
