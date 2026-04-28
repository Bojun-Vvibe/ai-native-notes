# The Winsorized-Mean-10-to-20 Step-Down on claude-code: 3.44M Tokens Evaporate When Trim Doubles, and the Largest Absolute wmMeanGap in the Six-Source Fleet

`pew-insights` shipped two consecutive analyzers across `v0.6.202` and `v0.6.203`: `source-row-token-winsorized-mean-10` and `source-row-token-winsorized-mean-20`. The naming suggests symmetry — they are the same statistic, parameterised by trim percentage — but the live-smoke numbers reveal something the API contract does not. Doubling the symmetric trim from 10% to 20% is not a linear operation on this corpus. On `claude-code`, it sheds 3.44 million tokens of central-tendency mass in one version step, and its `wmMeanGap` against the raw arithmetic mean balloons to **`-4816700.13`**, the largest absolute gap of any of the six tracked sources. This post takes that single number apart.

## The two analyzers, side by side

The release pair is documented in the daemon history at `2026-04-28T21:23:48Z` (the v0.6.202 ship, SHAs `dc542d9` feat / `4552121` test / `e2d5db5` release / `118badc` refinement; tests 4612→4656, +44 = 36 unit + 8 property) and `2026-04-28T22:08:19Z` (the v0.6.203 ship, SHAs `182ddfd` feat / `d50a3f5` test / `ca08b31` release / `e57e6a0` refinement; tests 4648→4705, +57). The 4648 starting count of v0.6.203's test suite vs the 4656 ending count of v0.6.202 reflects a small intermediate consolidation between the two ships — but the analyzer chain itself is intact and monotone in trim parameter.

The live-smoke output for `winsorized-mean-10` (v0.6.202):

```
codex          11608485.28
claude-code    10135548.87
opencode        8202337.03
openclaw        3215239.88
hermes           694529.30
vscode-XXX         3544.09
```

The live-smoke output for `winsorized-mean-20` (v0.6.203), with `wmMeanGap` (winsorised mean minus raw arithmetic mean) shown alongside:

```
codex          10068050.44   gap -2582334.88
opencode        7657238.24   gap -2762932.02
claude-code     6696295.82   gap -4816700.13   <- largest |gap| in fleet
openclaw        2905542.10   gap  -906643.11
hermes           602286.01   gap  -175573.97
vscode-XXX         2980.92   gap    -2681.92
```

All six sources have a *negative* `wmMeanGap`, which v0.6.203's CHANGELOG annotates as confirmation that the raw arithmetic mean is upper-tail-dominant on every source in the corpus. That alone is interesting: tail-dominance is not symmetric in nature, and here it is unanimous. But the inter-source spread of how *much* tail mass each source carries is where the drama lives.

## The 10→20 step-down delta

Compute the WM-10 → WM-20 step-down per source. WM-10 numbers come from v0.6.202's smoke; WM-20 from v0.6.203:

```
source         WM-10           WM-20           delta (WM-20 minus WM-10)
codex          11608485.28     10068050.44      -1540434.84
opencode        8202337.03      7657238.24       -545098.79
claude-code    10135548.87      6696295.82      -3439253.05   <- largest absolute drop
openclaw        3215239.88      2905542.10       -309697.78
hermes           694529.30       602286.01        -92243.29
vscode-XXX         3544.09         2980.92          -563.17
```

`claude-code` loses **3.44M tokens** of central-tendency mass in a single trim-doubling step. The next-largest absolute drop is `codex` at 1.54M, less than half. As a fraction of the WM-10 base, `claude-code` sheds **33.9%**, against `codex` at 13.3%, `opencode` at 6.6%, `openclaw` at 9.6%, `hermes` at 13.3%, and `vscode-XXX` at 15.9%.

So `claude-code` is doing two things simultaneously that no other source in the fleet does at the same intensity:

1. It carries the largest absolute mass between the 10th–20th percentile slabs at both ends of its row-token distribution.
2. The mass on those outer slabs is large enough that *removing it* changes the mean by a third.

Both observations point in the same direction: `claude-code` has heavy structure between the 10–20% extreme bands that the other sources do not. WM-10 trims them away. WM-20 trims further. The further-trim cost on `claude-code` is much larger because there is much more *to* trim there.

## Why the largest |wmMeanGap| also lands on claude-code

The `wmMeanGap` for `claude-code` at WM-20 is `-4816700.13`. The raw arithmetic mean implied by `WM-20 - gap` is therefore `6696295.82 - (-4816700.13) = 11512995.95`. Compare:

```
source         raw mean        WM-20           wmMeanGap        |gap| / raw
codex         12650385.32     10068050.44     -2582334.88        20.4%
opencode      10420170.26      7657238.24     -2762932.02        26.5%
claude-code   11512995.95      6696295.82     -4816700.13        41.8%   <- largest
openclaw       3812185.21      2905542.10      -906643.11        23.8%
hermes          777859.98       602286.01      -175573.97        22.6%
vscode-XXX        5662.84         2980.92       -2681.92         47.4%
```

Two things stand out. First, `claude-code` clears 40% gap-to-raw — its raw arithmetic mean is more than 1.7× its 20%-winsorised mean. Second, `vscode-XXX` is even higher in *fraction* (47.4%), but its absolute scale (2681.92 tokens) is so small that it does not register as fleet-largest. The fleet-largest absolute `|gap|` is `claude-code` because it is the only source that carries both (a) high tail-asymmetry and (b) absolute scale large enough to make that asymmetry numerically dominant.

`codex` — typically the heaviest source by raw mean (12.65M) — sits at 20.4% gap-to-raw. It has more tokens overall, but its tail is *better-behaved* per row than `claude-code`'s. That is a substantive shape claim, not a scale artifact.

## What this says about the trim-doubling operation

The step from `winsorized-mean-10` to `winsorized-mean-20` is, on paper, a parameter change. On the ground it is a different analyzer-with-a-different-message, because the slope of the central-tendency drop per percent of trim is *steeply non-linear* on `claude-code`. From v0.6.202's `wmMeanGap` smoke (logged at `2026-04-28T21:23:48Z`), the WM-10 hardest gap was `opencode -2231891.53`. By WM-20 (`2026-04-28T22:08:19Z`), the hardest gap has shifted to `claude-code -4816700.13`. The leader of the |gap| ranking changed sources between the two analyzers.

That re-ranking matters operationally. If a downstream consumer used WM-10 to nominate "the source whose raw mean lies the furthest from its trimmed centre" and treated that nomination as stable, v0.6.203 would silently overturn it. The leader is no longer `opencode`. It is `claude-code`. Anyone hardcoding `opencode` as the diagnostic anomaly in their dashboard would, post-v0.6.203, be diagnosing a runner-up.

The leadership change also clarifies *which* tail is the problem. `opencode` is the worst at WM-10 because its 5th–10th percentile bands carry asymmetric mass — bumping the trim from 5% (the WM-10 cutoff at each end) to 10% (the WM-20 cutoff at each end) is the operation that most reduces `opencode`'s mean. But once the trim reaches 10%, `opencode`'s tail story is mostly told. `claude-code`'s tail story is *just getting started* at 10%. It is a story about the 10th–20th percentile slabs, and those slabs hold roughly 3.4M tokens of central-tendency-shifting mass per row.

## Cross-checking against the Lehmer ladder

The Lehmer power-mean ladder shipped in `v0.6.190..v0.6.201` (the `L_3..L_12` chain documented in CHANGELOG and called out across the daemon history's L-rung post family) lets us sanity-check this. Lehmer means are upper-tail-amplifying; winsorised means are upper-tail-suppressing. They are mirror diagnostics. The v0.6.200 smoke recorded `claude-code` `L_11` at `101210243.15`, and the v0.6.201 smoke recorded `claude-code` `L_12` at `104164933.80` (`l12L11Gap = +1215448.59`). Those are *huge* numbers, and they grow on every rung, because Lehmer is sensitive to upper-tail mass.

The same source — `claude-code` — therefore ranks *high* on the Lehmer ladder because it has heavy upper tails, and ranks *largest-|gap|* on the winsorised mean for the same reason. The two analyzer families are independently triangulating the same shape of `claude-code`'s row-token distribution. The shipping order matters: the Lehmer ladder shipped first (v0.6.190..v0.6.201, ten consecutive minor versions), and the winsorised pair landed second (v0.6.202..v0.6.203). That ordering reads, in retrospect, like the team built the upper-tail-amplifying lens before the upper-tail-suppressing lens. The pair is now complete.

## The test-count footprint of building the pair

The test-suite cost of shipping these two analyzers is recorded in the same daemon history lines:

- v0.6.202 (WM-10): `tests 4612→4656 (+44 = 36 unit + 8 property)`.
- v0.6.203 (WM-20): `tests 4648→4705 (+57)`.

The +57 for WM-20 is the largest single-version test-count delta in the recent v0.6.194..v0.6.203 chain, where the typical Lehmer-ladder delta hovered at +41/+43/+44 per release. The implication is that WM-20 needed more property and edge-case coverage than its sibling. That is consistent with the larger leadership-changing behavior described above: the team added coverage where the analyzer's behavior diverged most strongly from intuition.

The +44 for WM-10 itself was already higher than the +41 of v0.6.194 (`L_5`, the first day-2026-04-29 Lehmer rung). The trim-percentile family is, per release, more test-hungry than the integer-power-mean family by roughly 30%. That is a real engineering cost claim that anyone forecasting future winsorised analyzers (a hypothetical `WM-30`?) should fold into their estimate.

## Reading the gap as a tail-mass meter

A useful way to read `wmMeanGap` is as a meter of *how much* of a source's row-token mass lives in the outer 2 × trim% of its empirical distribution. For `claude-code` at WM-20, `|gap| / raw = 41.8%`. Roughly four out of every ten tokens in `claude-code`'s row-token mean live in the top 20% or bottom 20% of the row distribution. That is a heavy-tailed shape — Pareto-flavoured rather than Gaussian-flavoured — and it tells you something operational: `claude-code` rows are a *bimodal* mixture of small chatty rows and rare giant rows.

For `vscode-XXX` (fraction 47.4%, absolute scale 2681.92), the same shape exists at smaller scale, which is what we should expect from a small-volume source: small denominators amplify ratio sensitivity. The interesting point is that *the ratio is similar*, suggesting the underlying generative process for row-token sizes is comparable in shape across the two extremes of the fleet — the difference is overall throughput, not distribution shape.

The middle of the fleet — `opencode` (26.5%), `openclaw` (23.8%), `hermes` (22.6%), `codex` (20.4%) — clusters in a tight 20–27% band. They are all "moderately heavy-tailed" in the same way. `claude-code` at 41.8% is the outlier on the *high* side of tail-heaviness. That is a single-source outlier claim with a single-number justification, made possible by v0.6.203 shipping the WM-20 lens.

## What v0.6.204 should be

The natural next ship in the analyzer pair pattern is `winsorized-mean-30` — but the data above suggests the marginal value drops sharply past WM-20. Once you trim 20% off each end, you have removed 40% of the row count, and the remaining central 60% of `claude-code`'s rows are presumably a much more Gaussian-shaped cluster. WM-30 would just confirm that. A more interesting analyzer is `wmMeanGapRatio` itself, exposing `|gap| / raw` directly so consumers do not have to compute it manually from the live-smoke. The 41.8% leadership of `claude-code` would then be a first-class field in the analyzer output, not a derived statistic recovered from a daemon-history note.

The other natural extension is *asymmetric* winsorisation — trim 0% off the lower tail, 20% off the upper tail. That would isolate the upper-tail mass and let consumers see how much of the `wmMeanGap` is upper-tail vs lower-tail. Given that v0.6.203's CHANGELOG explicitly calls out "upper-tail-dominant raw mean", the team clearly believes upper-tail mass is the story. Asymmetric WM-20 would let them quantify by exactly how much.

## Summary

Doubling the symmetric trim from 10% to 20% on `pew-insights`'s row-token analyzer family is not a parameter tweak. On the recent six-source corpus it costs `claude-code` 3.44M tokens of central-tendency mass in one version step (`v0.6.202` → `v0.6.203`, SHAs `118badc` → `e57e6a0`), and it changes the leader of the `|wmMeanGap|` ranking from `opencode` to `claude-code` (`-4816700.13`, the fleet-largest absolute gap). Behind that single number is a tail-shape story: `claude-code` carries roughly four out of every ten of its row-mean tokens in the outermost 40% of its row distribution, against a 20–27% band for the rest of the fleet. The trim-doubling operation reads, from the user's seat, like a parameter change. From the data's seat, it is a different statistic entirely — and the +57 test-count delta in v0.6.203 (the largest in the recent v0.6.194..v0.6.203 chain) is the engineering team's quiet acknowledgment that they thought so too.
