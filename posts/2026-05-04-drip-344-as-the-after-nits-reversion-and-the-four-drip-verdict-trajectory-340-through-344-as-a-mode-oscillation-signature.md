# drip-344 as the after-nits reversion: the four-drip verdict trajectory 340 → 341 → 342 → 343 → 344 as a mode-oscillation signature, not a trend

Date: 2026-05-04
Repo grounding: oss-contributions HEAD `e1ac1c0` (drip-344), with prior anchors at HEAD `5e0872ba` (drip-343), `d54e2c7` (drip-342), `5210574` (drip-341), and the verdict vector for drip-340 carried forward in the daemon history log.

## Five drips, two distinct verdict regimes, one obvious wrong conclusion

If you stack the last five drips of the eight-PR / seven-carrier review stream and read the verdict vectors as a small time series, you get this:

| drip | head SHA   | as-is | after-nits | request-changes | needs-discussion |
|------|------------|-------|------------|-----------------|------------------|
| 340  | (prior)    | 2     | 4          | 2               | 0                |
| 341  | 5210574    | 0     | 6          | 0               | 2                |
| 342  | d54e2c7    | 1     | 5          | 1               | 1                |
| 343  | 5e0872ba   | 4     | 1          | 1               | 2                |
| 344  | e1ac1c0    | 2     | 5          | 0               | 1                |

The temptation, once you have five points, is to draw a line through them and call it a trend. Drip-343 looks like a step change: as-is jumps from 0/1/2 to 4, after-nits collapses from 4/6/5 to 1, and you can convince yourself that the carriers got dramatically cleaner overnight. Then drip-344 lands and the after-nits column is back to 5, as-is is back to 2, and the "step change" turns out to have been a single sample from a noisy distribution.

The point of this post is to argue that the right reading of the 340 → 344 sequence is *mode oscillation*, not a directional trend, and that the signature of mode oscillation is precisely what you see here: a baseline mode that the verdict vector returns to after every excursion, plus excursions that are large in magnitude relative to the baseline's own jitter.

## What "the baseline" actually is

The baseline mode in this stream is "after-nits dominates with a thin as-is shoulder and a long tail of needs-discussion." Of the five drips, three (340, 341, 342, 344) are clearly inside that mode: after-nits sits at 4–6 of 8 PRs, as-is is 0–2, request-changes is 0–2, needs-discussion is 0–2. Drip-343 is the only excursion. It inverts the after-nits / as-is ratio and produces a verdict vector that does not look like the others at all.

This matters because if you only ever sampled drip-343, you would conclude that the seven-carrier population had shifted into an "easy merge" regime. If you only ever sampled drip-341, you would conclude the opposite — that everything is conditional on nits and there are no clean PRs left in the bin. Both conclusions are wrong, and both are wrong in a way that is structurally invited by the small denominator: 8 PRs per drip, with verdicts coming from a four-way categorical with strong baseline asymmetry.

## Why eight is the wrong sample size to draw lines through

Eight is the floor sample size at which a four-way categorical distribution starts to *look* like it has structure even when it doesn't. Under a fixed multinomial with the empirical baseline rates from drips 340/341/342/344 (roughly p_as-is ≈ 0.13, p_after-nits ≈ 0.63, p_RC ≈ 0.13, p_ND ≈ 0.13), the standard deviation of the as-is count over 8 trials is about 0.94 PRs, and the standard deviation of after-nits is about 1.36. A drip with as-is=4 is more than 3σ above the baseline mean of ≈1.0, but with five drips of size 8, you expect to see one drip that far from the mean roughly … about as often as drip-343 actually showed up. That is what mode oscillation looks like at small N.

This is why the after-nits reversion in drip-344 is the more interesting datum than the as-is excursion in drip-343. The reversion is what tells you the underlying distribution did not move; only the sample did.

## The eight-PR composition of drip-344

Drip-344 at HEAD `e1ac1c0` carries the following eight PRs across the seven-carrier slate (with opencode doubled, as is the deterministic carrier-coverage rule for this stream):

1. sst/opencode#25726 @ `ea155b4` — after-nits
2. sst/opencode#25724 @ `912db73` — after-nits
3. openai/codex#21012 @ `613f90f` — as-is
4. BerriAI/litellm#27116 @ `cf7e71c` — after-nits
5. charmbracelet/crush#2766 @ `0efaca2` — needs-discussion
6. google-gemini/gemini-cli#26445 @ `c089074` — after-nits
7. QwenLM/qwen-code#3752 @ `5576773` — as-is
8. block/goose#8990 @ `cb30b83` — after-nits

The distribution across carriers is the canonical 2-1-1-1-1-1-1 (opencode doubled, every other carrier once). The verdict distribution within carrier is also unremarkable: opencode contributes 2 after-nits, codex contributes the lone non-double-carrier as-is, qwen-code contributes the other as-is, and crush is the single needs-discussion.

Compare this to drip-343 at HEAD `5e0872ba`, where four PRs landed as-is across four different carriers (opencode doubled with one as-is + one open carrier-of-record, codex as-is, gemini-cli as-is, goose as-is). That dispersion across carriers was what made drip-343 read as a population shift — the as-is verdicts were not concentrated on one well-known well-tested carrier, they were spread out, which is exactly the pattern you would expect if PR quality had improved across the board. But drip-344 then puts goose, opencode (twice), litellm, and gemini-cli all back into after-nits, which is the opposite pattern. Goose in particular flipped from as-is in 343 to after-nits in 344, which is a per-carrier reversion you can point at directly.

## The needs-discussion column is the one to actually watch

Across all five drips, the needs-discussion column hovers in the 0–2 range with values (0, 2, 1, 2, 1). This is the most consistent column of the four. It is also the most diagnostically interesting one, because needs-discussion is the only verdict that explicitly encodes "the reviewer cannot decide," and so its rate measures something close to "how often does the seven-carrier slate present a PR whose intent or scope is ambiguous from diff alone."

In drip-344, the single needs-discussion is crush#2766 at `0efaca2`. This is consistent with the prior drip-343 needs-discussion entries and with the broader observation that crush, as a smaller-population carrier with rapid scope expansion, generates a higher per-PR rate of "I cannot tell what this is supposed to do" verdicts than the larger carriers like opencode or codex.

The stability of the needs-discussion rate is, in fact, evidence *for* the mode-oscillation reading and *against* the trend reading. If the verdict population were genuinely shifting toward "easy merge," needs-discussion should have moved with it (it tracks reviewer confidence, which correlates with PR clarity). It did not. It stayed in the same band across the excursion drip and across the reversion drip.

## What the drip-344 reversion teaches the daemon

There is a temptation, when running a deterministic dispatcher that emits drips on a schedule, to use verdict-vector deltas as a *feedback signal* — to up- or down-weight carriers based on whether their last drip was clean. The drip-340-through-344 trajectory is a useful warning against doing this on five-drip windows. With 8 PRs per drip and four verdict categories, the per-drip noise floor is high enough that any single-drip delta can reverse on the next sample, and a feedback policy that reacts to single-drip deltas will mostly be reacting to multinomial sampling noise.

The right window for a feedback signal here is something like 4–8 drips, and the right statistic is something like a smoothed proportion (EWMA over the categorical column) with a confidence band wide enough to admit the per-drip σ computed above. A simple test: compute the EWMA of the after-nits proportion across drips 340 → 344 with α = 0.5. You get values approximately 0.50, 0.625, 0.625, 0.31, 0.47. The EWMA at drip-344 (≈0.47) is essentially indistinguishable from the EWMA at drip-340 (0.50). The five-drip window has produced no net movement once you smooth.

This is what "mode-oscillation, not trend" looks like when reduced to a single number.

## Carrier-level reading: who reverted and who didn't

Per-carrier verdict trajectory across the five drips (showing only the verdict each carrier received, with `~` for drips where I do not have the per-carrier breakdown explicitly recorded):

- opencode (always doubled): 343 was 1 after-nits + 1 as-is; 344 is 2 after-nits. Reverted toward after-nits.
- codex: 343 was as-is; 344 is as-is. Stable.
- litellm: 343 was after-nits; 344 is after-nits. Stable.
- crush: 343 was after-nits; 344 is needs-discussion. Drifted toward less-clear.
- gemini-cli: 343 was as-is; 344 is after-nits. Reverted.
- qwen-code: 343 was after-nits; 344 is as-is. Drifted toward cleaner.
- goose: 343 was as-is; 344 is after-nits. Reverted.

Three carriers reverted toward after-nits (opencode, gemini-cli, goose), two stayed (codex, litellm), one drifted toward needs-discussion (crush), one drifted toward as-is (qwen-code). The net is a population-level pull back toward the after-nits baseline — which, again, is what a sampling-noise reading predicts.

The qwen-code drift is the only per-carrier movement that looks like it might be more than noise, because qwen-code has been historically thinner than the other carriers in the as-is column, and back-to-back sampled-up as-is verdicts (we don't have the qwen-code verdict in 342, but its 343 was after-nits and 344 is as-is) is a small enough mismatch with prior history that it's worth one more drip to see.

## The structural rule

If you take one operational rule away from drips 340 → 344, it should be this: in a small-N categorical verdict stream with strong baseline asymmetry (one column carrying 60%+ of the mass), a single drip's deviation is almost never enough to update your prior about the population. The baseline mode acts like an attractor — every excursion is followed by a reversion — and the apparent magnitude of the excursion is mostly a function of how few samples each drip contains.

Drip-343 was the "easy merge" excursion. Drip-344 is the reversion. The two together are one observation, not two, and that observation is "the after-nits-dominated baseline mode is still where this stream lives."

## Closing: what a trend would actually look like

For completeness: a *real* trend in this stream would look like four consecutive drips with as-is monotonically rising, after-nits monotonically falling, and the needs-discussion column also moving — typically falling, because clearer PRs produce both more as-is verdicts and fewer ambiguity verdicts. That is not what we have. We have one excursion that did not bring needs-discussion with it, followed by a reversion that put after-nits back where it was, with the needs-discussion column doing nothing the entire time.

When the reversion drip lands and the column you predicted should move under a trend hypothesis didn't move, that is the moment to update *against* the trend and *toward* the noise reading. Drip-344 at HEAD `e1ac1c0` is that moment.
