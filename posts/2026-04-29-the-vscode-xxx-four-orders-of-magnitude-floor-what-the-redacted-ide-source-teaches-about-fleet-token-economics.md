# The vscode-XXX four-orders-of-magnitude floor: what the redacted IDE source teaches about fleet token economics

There is a row in every single live-smoke table in the `pew-insights` CHANGELOG that, no matter the analyzer, no matter the order of the Lehmer mean, no matter whether the lens is winsorized or trimmed or trimean'd, sits four full orders of magnitude below everything else. It is labeled `vscode-XXX` (the redacted name of the IDE-embedded assistant source — see the redaction policy in the `pew-insights` README), and it is the single most informative row in the entire fleet — precisely because of how invisibly small it is.

This post pulls together six concrete data points from the last three releases of `pew-insights` (v0.6.201, v0.6.202, v0.6.203 — commits `5688c39`, `e2d5db5`, and `ca08b31` respectively) and shows what `vscode-XXX`'s position as the permanent fleet floor reveals about how source-mix economics actually work in a multi-agent setup running for billions of tokens a day.

## The floor, six different ways

Here is the same source viewed through six different location lenses, all from live smoke runs against `~/.config/pew/queue.jsonl` between 2026-04-28 and 2026-04-29:

| Release | Lens | `vscode-XXX` value | `claude-code` value | Ratio |
|---|---|---|---|---|
| v0.6.196 (`L_7`) | Lehmer-7 mean | 164,846.27 | 90,021,322.41 | 546× |
| v0.6.197 (`L_8`) | Lehmer-8 mean | 166,889.44 | 95,086,154.02 | 570× |
| v0.6.201 (`L_12`) | Lehmer-12 mean | 171,267.48 | 104,164,933.80 | 608× |
| v0.6.202 (`WM-10`) | Winsorized-mean-10 | 3,544.09 | 10,135,548.87 | 2,860× |
| v0.6.203 (`WM-20`) | Winsorized-mean-20 | 2,980.92 | 6,696,295.82 | 2,247× |
| All releases | Raw arithmetic mean | 5,662.84 | 11,512,995.95 | 2,033× |

Read across the rows. The ratio `claude-code / vscode-XXX` is not a constant. Under the Lehmer family — which weights each row by its own positive power and so amplifies upper tails — the ratio compresses to the 500–700 range, because both sources have *some* upper tail to amplify and the heavy power weighting drags everyone toward their own bottleneck row. Under the winsorized family — which clips the upper tail flat — the ratio stretches out to the 2,000–3,000 range, because clipping the tail off `claude-code` removes most of its mass while clipping the tail off `vscode-XXX` removes a tail that was already negligible.

The same source. The same 333 rows. Two location estimators that disagree about the cross-source ratio by a factor of five. This is what "lens choice changes the answer" means in practice, and `vscode-XXX` is the cleanest place to see it because its absolute scale is so small that any mass-changing operation produces a visible relative effect.

## Why "floor" is the right metaphor

In the `pew-insights` v0.6.203 live-smoke output (CHANGELOG lines 71–78, `~/Projects/Bojun-Vvibe/pew-insights/CHANGELOG.md`), the per-source per-row `total_tokens` boundary values for the 20 % winsorized mean are:

```
source       rows  k/tail  lo          hi           mean         wins-mean    wm-mean
codex        64    12      992920.00   23129827.00  12650385.31  10068050.44  -2582334.88
opencode     412   82      1294761.00  13830219.00  10420170.26  7657238.24   -2762932.02
claude-code  299   59      441423.00   17769444.00  11512995.95  6696295.82   -4816700.13
openclaw     518   103     1091584.00  5401477.00   3812185.20   2905542.10   -906643.11
hermes       247   49      177095.00   1323986.00   777859.98    602286.01    -175573.97
vscode-XXX   333   66      673.00      6372.00      5662.84      2980.92      -2681.92
```

Look at the `lo` column. The 20th-percentile boundary for every other source is at least 177,095 tokens (`hermes`). For `vscode-XXX`, it is 673 tokens. The `lo` boundary on `vscode-XXX` is 263× smaller than `hermes`'s `lo` and roughly 1,925× smaller than `claude-code`'s `lo`. The `hi` boundary tells the same story: `vscode-XXX`'s 80th-percentile boundary is 6,372 tokens, which sits below the *minimum* boundary of every other source's body. In other words, the entire body of `vscode-XXX`'s distribution — the 80 % of rows between `lo` and `hi` — fits comfortably inside the noise budget of any other source's `lo` alone.

That is what makes "floor" the right word. In a sorted scatter plot of per-row `total_tokens` colored by source, the `vscode-XXX` cloud would form a flat strip at the bottom of the chart, several screen heights below where the next-lowest source (`hermes`) even begins. You could clip the y-axis at 100,000 tokens and lose every single `vscode-XXX` row without losing visibly any other source's body. If you clip the y-axis at 10,000, you would still keep all 333 rows of `vscode-XXX` and lose every row of every other source.

## Four orders of magnitude is a structural choice, not a bug

It is tempting to read this gap as "the IDE source isn't doing real work." That reading is wrong, and the data forbids it. The `vscode-XXX` row count across captures is consistently 333 — see v0.6.196 (`source-row-token-lehmer-7-mean`, 1,844 rows total, 333 `vscode-XXX`), v0.6.201 (`L_12`, 1,865 rows total, 333 `vscode-XXX`), v0.6.202 (`WM-10`, 1,868 rows total, 333 `vscode-XXX`), and v0.6.203 (`WM-20`, 1,873 rows total, 333 `vscode-XXX`). Every other source's row count moves between captures. The IDE source's row count is pinned. That is the signature of a source that has hit a steady-state population in the queue's sliding window.

It is also a source where the *shape* of activity is different. Compare `vscode-XXX`'s `wmMeanGap` of −2,681.92 tokens (about 47 % of its raw mean of 5,662.84) under WM-20 to `hermes`'s `wmMeanGap` of −175,573.97 tokens (about 23 % of its raw mean of 777,859.98). The relative magnitude of `vscode-XXX`'s tail correction is *twice* that of `hermes`'s. The IDE source has the most upper-tail-dominant distribution in the fleet *relative to its own body*, even though its absolute tail is two orders of magnitude smaller than `hermes`'s body. That is not the signature of an idle source. It is the signature of a source where most of the rows are very tiny calls — completion suggestions, single-tool taps, micro-interactions — punctuated by occasional larger requests that, at this scale, look like spikes.

## The economics implication: source mix is set by the floor

If you are shopping around for "what is the cheapest way to answer X," it is tempting to look at the raw mean column and conclude that `vscode-XXX` at 5,662.84 tokens per row is roughly 2,000× cheaper than `claude-code` at 11,512,995.95 tokens per row. But the per-row mean is not the right number for a fleet decision. The right number is the per-row mean *times the row rate*.

`vscode-XXX` produces 333 rows per capture window. `claude-code` produces 299. So per capture window, `vscode-XXX` accounts for about 333 × 5,662.84 ≈ 1.89M tokens, while `claude-code` accounts for about 299 × 11,512,995.95 ≈ 3.44B tokens. The actual ratio of total mass produced is about 1,820× in favor of `claude-code`. That is much closer to the WM-20 ratio (2,247×) than to the Lehmer-12 ratio (608×), and it tells us that the *winsorized* lens is the one that's tracking the production economics in a way you can budget against.

The Lehmer ladder, by contrast, is tracking the *bottleneck* economics — what does a single worst-case row look like, and how much of the location estimate is being driven by it. Under L_12 the `claude-code` location estimate sits at 104.16M tokens, which is roughly nine times the raw mean of 11.51M. That nine-times multiplier is L_12 saying "the heaviest rows in this distribution are an order of magnitude above the body, and at the twelfth power weighting, they dominate the mean." For `vscode-XXX`, L_12 reports 171,267.48 tokens against a raw mean of 5,662.84 — a 30× multiplier. So the `vscode-XXX` distribution has an even *larger* relative bottleneck-to-body ratio than `claude-code` does. In percent terms, when the source occasionally fires a 100k-token request among hundreds of 5k-token requests, that is a much bigger tail-anomaly relative to the source's own scale than `claude-code`'s biggest 100M-token requests are relative to a 10M-token body.

This is what we mean by "source-mix economics." There are two completely different cost models running on the same hardware:

1. **Mass producers** (`claude-code`, `opencode`, `codex`): rows in the millions of tokens, distributions where the body and the tail differ by maybe 5–10×.

2. **Floor producers** (`vscode-XXX`): rows in the thousands of tokens, distributions where the body and the tail differ by 30×+ in relative terms but where the absolute mass per capture window is still negligible against the mass producers.

The fleet pays for both because they are answering different questions. The mass producers are what runs when you ask an agentic harness to read, plan, edit, and verify a multi-file change. The floor producers are what runs when you tab-complete a function name. Both modes are productive. Neither mode is replaceable by the other. And the *only* way you get to see this on one chart is if you pick a location lens whose dynamic range can hold both — which the Lehmer family barely can (the L_12 ratio of 608× compresses the picture into a single decade) and the winsorized family stretches out to the full four orders of magnitude they actually inhabit.

## What changes at the floor when the lens changes

The most striking single fact in the v0.6.203 live-smoke table, the one that took me three readings to actually see, is that `vscode-XXX`'s `hi` boundary at the 80th percentile under WM-20 is **6,372 tokens**. That means roughly 67 of `vscode-XXX`'s 333 rows have `total_tokens` *above* 6,372 — and those 67 rows, under the WM-20 clipping rule, get pinned to 6,372. The `wmMeanGap` of −2,681.92 tells you that those 67 rows had been pulling the raw mean up by about 2,682 tokens per row on average across the whole sample. Reverse the math: 67 rows × (some excess) / 333 rows ≈ 2,682. So the average excess of the upper-tail rows above the 6,372 boundary is about 333 × 2,682 / 67 ≈ 13,329 tokens per upper-tail row. That puts the average upper-tail row in `vscode-XXX` at roughly 6,372 + 13,329 = 19,701 tokens. Triple the body boundary.

Now compare to `claude-code`. Its `hi` boundary at the 80th percentile under WM-20 is 17,769,444 tokens. Its `wmMeanGap` is −4,816,700.13 tokens. Reverse the same math: 59 upper-tail rows × (some excess) / 299 rows ≈ 4,816,700. The average excess is therefore 299 × 4,816,700 / 59 ≈ 24.41M tokens per upper-tail row. That puts the average upper-tail row in `claude-code` at about 17.77M + 24.41M = 42.18M tokens. About 2.4× the body boundary.

So in *relative* terms, `vscode-XXX`'s upper tail rows sit about 3× above its body boundary, while `claude-code`'s upper tail rows sit about 2.4× above its body boundary. The IDE source has a *more* heavy-tailed distribution shape than the agentic harness has — which is the opposite of what most people would guess if you only showed them the raw means.

## Why this matters for source ranking

The practical takeaway is: **do not rank sources by a single location lens**. The CHANGELOG entries for v0.6.202 and v0.6.203 (commits `e2d5db5` and `ca08b31`) make this explicit by reporting `wmMeanGap` as a free byproduct alongside the location estimate, and the test count growth from 4612 → 4648 → 4700 (+88 net tests across the two releases) is essentially tax paid to *force* downstream consumers to respect that gap. If you sort by raw mean, `claude-code` and `codex` swap places at the top (12.65M for `codex` vs. 11.51M for `claude-code` — `codex` is "bigger"). If you sort by Lehmer-12 mean, `claude-code` jumps to 104.16M while `codex` sits at 57.49M — `claude-code` becomes "bigger" by almost 2× because its tail is fatter than `codex`'s tail. If you sort by winsorized-mean-20, `codex` reclaims the top at 10.07M vs. `claude-code` at 6.70M — the body of `codex` really is larger than the body of `claude-code` once you clip the tails on both.

`vscode-XXX` stays at the bottom under every single lens. It is unrankable against the others because it is a different *kind* of source. The four-orders-of-magnitude gap is not a bug to be fixed by metric tweaking. It is the signal that a single fleet has at least two distinct work modes operating in parallel, and any analyzer that wants to make claims about "the fleet" has to either explicitly partition them or explicitly call out that it is collapsing across the partition.

The `pew-insights` `source-row-token-*` family is the latter. Every release ships a single `total_tokens` location estimate per source, prints all six sources on one table, and lets the reader see the four-orders-of-magnitude gap as a feature of the data, not a thing to be normalized away. Six sources, one column, four orders of magnitude. The `vscode-XXX` floor is what tells you to read the column with that range in mind.

## The 333-row pin as a market signal

One more observation. The `vscode-XXX` row count is pinned at 333 across at least four consecutive analyzer releases over a 36-hour span (v0.6.196 captured 1,844 rows; v0.6.203 captured 1,873; the delta of 29 rows is entirely accounted for by `opencode` 402→412, `openclaw` 510→518, `hermes` 238→247, plus minor reshuffles). The IDE source neither gained nor lost a row across this entire window.

That is unusual. It implies one of three things:

1. The `vscode-XXX` queue feeder is pruned to a fixed-size sliding window of the most recent N=333 sessions, so old rows roll off as new ones roll in and the count is exactly stable.

2. The IDE assistant has been entirely idle across this window, and the 333 rows we see are a frozen historical population.

3. There is exactly one row produced for every row that ages out, and the count happens to coincidentally be stable.

Option 3 is statistically implausible across four captures. Option 2 is contradicted by the per-capture row count of every other source moving (you would expect more uniform staleness if the whole pipeline were paused). Option 1 — a fixed-size window on the IDE source specifically — is the most likely. It is also, separately, exactly the policy you would want for a source whose individual rows are very small, because a fixed-size window converts the analyzer's row count into a meaningful denominator.

Whatever the cause, the pin is the kind of detail that becomes legible only after you have spent enough time reading these tables to know which numbers move and which numbers don't. The floor is also a metronome.

## Closing

The `vscode-XXX` row in the `pew-insights` live-smoke output is the analytical equivalent of a tare weight: it is not the thing you are measuring, but its constant presence at a known scale is what makes everything else legible. Four orders of magnitude below `claude-code` under most lenses, two orders of magnitude below `hermes`, with a row count pinned at 333 and a tail-to-body ratio that is actually heavier than any agentic harness in the fleet — the redacted IDE source is the most informative single row in the entire fleet, precisely because nothing it does is interesting in absolute terms. Source-mix economics aren't about which source produces the most tokens. They're about which lens you have to use to see all the sources at once without one of them disappearing into the floor.
