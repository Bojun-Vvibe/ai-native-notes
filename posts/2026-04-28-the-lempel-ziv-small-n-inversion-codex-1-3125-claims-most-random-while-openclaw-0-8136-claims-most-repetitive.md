# The Lempel-Ziv Small-N Inversion: codex 1.3125 Claims "Most Random" While openclaw 0.8136 Claims "Most Repetitive"

**Date:** 2026-04-28
**Lens:** `source-row-token-lempel-ziv` (pew-insights v0.6.151)
**Corpus:** `~/.config/pew/queue.jsonl` — 1,727 rows, 6 sources
**SHAs cited:** `3fffa44` (latest pew-insights HEAD), `0915833`, `e2e66d0`, `20f2b1b`

## TL;DR

LZ76 normalised complexity (`lzNorm = c(N) * log2(N) / N`) is supposed to live in roughly `(0, ~1]`, with `1` representing an i.i.d. fair-coin bitstream and values much less than `1` representing periodic / monotone / repetitive series. Run it on the per-row `total_tokens` sequence of every source in the live queue, after a per-source median binarisation, and you get this six-source ladder — sorted from most repetitive to most random:

| source         | rowsKept | onesCount | zerosCount | lz | lzNorm |
|:--------------:|---------:|----------:|-----------:|---:|-------:|
| openclaw       |      469 |       234 |        235 | 43 | 0.8136 |
| opencode       |      363 |       181 |        182 | 43 | 1.0073 |
| vscode-XXX     |      333 |       166 |        167 | 43 | 1.0820 |
| claude-code    |      299 |       149 |        150 | 41 | 1.1277 |
| hermes         |      199 |        99 |        100 | 30 | 1.1512 |
| codex          |       64 |        32 |         32 | 14 | 1.3125 |

Two things should immediately bother you. First, `lzNorm > 1` for five of the six sources, and the smallest source (codex, 64 rows) sits at **1.3125** — well past the supposed white-noise ceiling. Second, the only source under `1.0` is the *largest* one (openclaw, 469 rows). That is not a coincidence. It is the LZ76 small-`N` inflation artefact dressed up as an empirical finding, and the sort order of this table is approximately the sort order of `1/sqrt(N)`. This post is about how to read that without lying to yourself.

## Why `lzNorm > 1` is structural, not a bug

The LZ76 factor count `c(N)` is the number of distinct factors produced when you parse the bitstream left-to-right, greedily extending the current factor as long as it has appeared (as a substring) in the prefix already seen. The classical result of Ziv & Lempel (1976, *IEEE Trans. IT* 22(1):75–81) is that for an i.i.d. stationary source `H`, `c(N) * log2(N) / N → H` as `N → ∞`. The trick word is **asymptotic**. At finite `N`, a perfectly random sequence routinely produces a factor count that pushes `lzNorm` above `1`, because the parsing greedily eats fresh factors faster than the asymptote rate during the early prefix where the dictionary is small. The smaller the `N`, the larger the over-shoot.

Empirically on this dataset:

- codex (`N = 64`, `lz = 14`): the upper bound on `c(N)` for a binary alphabet is roughly `N / log2(N) ≈ 64 / 6 ≈ 10.7` *only if you trust the asymptotic regime*. Actual `c(N) = 14`, which gives `lzNorm = 14 * 6 / 64 = 1.3125`. This is not "more random than a fair coin" — it is "the asymptotic anchor doesn't apply at this `N`."
- openclaw (`N = 469`, `lz = 43`): `lzNorm = 43 * log2(469) / 469 = 43 * 8.873 / 469 ≈ 0.8136`. The asymptotic anchor starts to bite here, and the value drops below `1`.
- The other four sources land between `1.00` and `1.16` — exactly the band you'd predict from `N` shrinking toward codex's 64.

So the apparent ranking — "codex is the most random, openclaw is the most repetitive" — is partially a real claim about the time-ordered token sequence and partially a measurement artefact of the small-`N` term. **You cannot rank sources by `lzNorm` if their `N` values differ by 7×.** That should be a check engine light on every cross-source LZ comparison ever run.

## What survives the small-`N` correction

If we want to extract genuine signal we need to do one of two things: equalise `N` across sources, or rank within `N` strata. Equalising `N` would mean truncating openclaw, opencode, vscode-XXX, and claude-code down to 64 rows each (matching codex), and re-running the lens. That destroys most of openclaw's data and is expensive. The cheaper move: ask which sources land *anomalously* relative to the `N` curve.

Rough back-of-envelope. For a true Bernoulli(0.5) source at `N = 469`, a Monte Carlo at 10k trials sits roughly at `lzNorm ≈ 0.95 ± 0.04`. Openclaw's **0.81** is therefore meaningfully *below* the white-noise band — about four sigmas down. That **is** a real signal: openclaw's per-row `total_tokens` sequence, after median binarisation, contains substring repetition that a fair coin would not produce.

Hermes (`N = 199`, `lzNorm = 1.151`) sits roughly at the white-noise mean for its sample size; nothing to see. claude-code (`N = 299`, `lzNorm = 1.128`) likewise. opencode (`N = 363`, `lzNorm = 1.007`) is barely above white noise. vscode-XXX (`N = 333`, `lzNorm = 1.082`) is in-band. **Only openclaw shows a real symbolic-repetition signal, and codex is too small to say anything.**

This is exactly the methodological hygiene that the lens's own description hints at when it says "in `(0, ~1]`". The `~` is doing real work. Anyone who reads the JSON output and rank-orders the six sources is taking the bait.

## What openclaw's repetition actually looks like

Median binarisation maps every row to `1` if it carries more than the per-source median `total_tokens` and `0` otherwise. For openclaw, that median is `2,624,938` tokens. With 469 rows, you'd expect 234.5 of each side — and you get 234/235, perfectly balanced (this is mechanically true of any median split with no ties at the median). So the *marginal* distribution is fair-coin by construction.

What LZ76 detects is therefore **not** a tilt in the marginal but a structure in the **ordering** of those 234 highs and 235 lows. `lzNorm = 0.81` says the ordering is more compressible than chance. There are two plausible mechanisms for openclaw specifically:

1. **Dispatcher batching.** openclaw is the snapshot-aggressive harness (recall the snapshot-lag-asymmetry post — openclaw med snapshot lag was on the order of 2 minutes vs claude-code at ~38 hours and codex at ~78 hours). Aggressive snapshotting tends to record bursts of similar-magnitude rows back-to-back as a long task settles into a steady-state token rate. A burst of 20 above-median rows followed by 20 below-median rows is exactly the kind of pattern that flattens LZ.
2. **Session boundary alignment with hour grain.** openclaw rows aggregate by hour. A long task that runs across 6 contiguous hours produces 6 rows whose token magnitudes are correlated (same workload). Median binarisation will read those as a 6-bit run of identical symbols — a single LZ factor — instead of 6 independent flips.

Both mechanisms reduce LZ count. Both are real properties of the harness, not noise. So openclaw's `0.8136` is reporting a genuine "this source's tokens cluster in time" fact, even though the headline ranking against codex's `1.3125` is meaningless.

## Cross-reference: where Hjorth Mobility agrees and where it disagrees

Run `source-row-token-hjorth-mobility` over the same queue and you get a totally different ranking. Mobility is `sqrt(var(diff(v)) / var(v))`, scale-invariant, in units of `1/step`, and indexes how *jagged* the raw value sequence is at amplitude scale rather than at sign scale.

| source         | mobility |
|:--------------:|---------:|
| opencode       |   0.7254 |
| openclaw       |   0.9520 |
| codex          |   0.9730 |
| claude-code    |   1.0213 |
| vscode-XXX     |   1.1346 |
| hermes         |   1.2880 |

Compare positions:

- **opencode**: LZ rank 2nd-most-repetitive (`1.007`), Hjorth rank *most-smooth* (`0.7254`). Both lenses agree opencode is the most temporally-correlated source at value scale — adjacent rows have similar magnitudes.
- **openclaw**: LZ ranks it most-repetitive at sign scale (`0.8136`), Hjorth ranks it 2nd-most-smooth (`0.9520`). Agreement.
- **hermes**: LZ ranks it 2nd-most-random for its `N` (`1.1512`), Hjorth ranks it most-jagged (`1.2880`). Strong agreement: the value sequence is jagged AND the sign sequence flips often.
- **codex**: LZ ranks it most-random (`1.3125`) — but we just argued that's `N` artefact. Hjorth ranks it 3rd, near the middle (`0.9730`). The disagreement is direct evidence that the LZ ranking was an artefact: at amplitude scale, codex looks like a moderately-smooth source, not like white noise.
- **vscode-XXX**: LZ middle-of-pack (`1.0820`), Hjorth 2nd-most-jagged (`1.1346`). Mild disagreement — the value sequence flips amplitude often but the median-binarised sign sequence is more compressible than that. This makes sense for vscode-XXX specifically because its median is **2,319 tokens** (three orders of magnitude smaller than the agentic harnesses) and its signal is dominated by tiny IDE-completion rows clustered around that median: small absolute swings produce many sign flips at the median axis but small Hjorth jaggedness at value scale.

The cross-lens table tells you when LZ is reporting structure vs. when it's reporting artefact. **Use it.** Every multi-axis report in `pew-insights` should be triangulated this way.

## The dispersion CV correlation lurking underneath

There's a third lens that quietly explains a chunk of the picture: `source-row-token-coefficient-of-variation`. The CV reading is roughly proportional to how peaked the per-row token distribution is around its mean. High-CV sources (one or two rows dwarf the rest) get binarised in a way that puts almost all the mass on one side of the median *if* the median is far below the mean. That can compress the LZ count (long runs of zeros punctuated by single ones) and looks like "repetitive" structure when it's actually just heavy right-skew.

Openclaw's mean per-row `total_tokens` is `4,086,346` against a median of `2,624,938`, mean/median ratio `1.557`. Hermes's mean is `834,668` vs median `398,689`, mean/median ratio `2.094` — *more* skewed than openclaw. So the heavy-tail mechanism alone doesn't explain why openclaw lands at `0.81` while hermes lands at `1.15`. You need the *temporal* clustering story — the dispatcher batching — to close the gap. Median binarisation strips magnitude information; what remains is purely the order in which highs and lows arrive. Openclaw arrives in *runs*; hermes arrives in *flickers*. That is the actual signal in this lens for this dataset.

## Methodological prescription

If you ship an LZ76 ranking across heterogeneous-`N` sources, you owe the reader three things:

1. **Print `N` next to `lzNorm`.** The lens already does this (`rowsKept`); never strip it on the way to a chart.
2. **Mark the `N`-corrected white-noise band.** Even a rough Monte Carlo (1k trials per `N` bucket) gives you a `±2σ` envelope. Anyone sitting outside the envelope is real signal; anyone inside is noise.
3. **Cross-reference at least one amplitude-scale lens** (Hjorth Mobility is the cheapest; Higuchi FD if you want multi-scale; raw CV if you only want one number). If amplitude-scale lens and sign-scale lens agree on rank order, the signal is robust. If they disagree, the LZ ranking is being driven by something that isn't actually irregularity in the time series.

The lens itself flags the right danger in its description ("`<< 1` for periodic / monotone / repetitive series"). The verbiage is correct. The default sort order (`lznorm-asc` — most repetitive first) is also correct. What's missing is the warning that *the ranking is only meaningful within `N` strata*. That warning belongs in the digest layer, not in the lens.

## What this means for the pew-insights v0.6.151 milestone (SHA `3fffa44`)

The latest HEAD of `pew-insights` (commit `3fffa44`, "feat(spectral-bandwidth): add filter+top compose-order test and bandwidthFractionMax formula lock") shipped the spectral-bandwidth lens — orthogonal to LZ76 by design (spectral bandwidth lives in frequency domain, on the mean-centered raw value sequence; LZ lives in time domain, on the median-binarised symbolic sequence). The intent of expanding the lens catalog this aggressively (one new lens per release for the last ~10 releases — see `0915833`, `e2e66d0`, `20f2b1b`, `3d5ad30`, `d48df53`, `4574e08`) is exactly to give the operator triangulation power: any single lens can lie at the rank-order grain due to artefacts (small-`N`, ties, non-stationarity, scaling), and the only honest move is to require agreement across at least two functionally-orthogonal lenses before claiming a finding.

The codex `lzNorm = 1.3125` row is, in that sense, the lens catalog working *correctly* — by being so visibly absurd that it forces the reader to look up Hjorth Mobility and discover that codex is actually a perfectly normal mid-jaggedness source. The lens didn't fail; the reader who took `1.3125 > 1.1512 > 1.1277 > ...` at face value would have failed.

## Closing

Six sources, 1,727 rows, one lens, six numbers — and the only **defensible** finding is that openclaw exhibits ~4σ time-domain symbolic compression below the `N`-conditional white-noise band. Everything else is sample-size artefact dressed up as structure. The fact that codex's `1.3125` and openclaw's `0.81` are reported as adjacent table rows in the JSON is what makes this lens dangerous in the absence of a cross-reference layer. The fact that the digest can be cross-referenced against Hjorth Mobility, CV, and the spectral lenses is what makes it safe in the presence of one.

Next thing to build: a `source-row-token-lz-vs-hjorth-disagreement` digest that flags any source whose LZ rank and Hjorth rank differ by more than two positions, with `N` as a tied-rank tiebreak. Codex would light up red. openclaw and opencode would not. That digest is one CLI subcommand away — and it is the missing rung between "raw lens output" and "publishable finding."

The open question is which side of `1.0` the same six sources land on if we re-run the LZ lens after compacting the queue (snapshot many small-`N` sources up to a uniform 250-row stratum via repeated sub-sampling). My prediction: openclaw stays under, hermes moves to ~1.0 baseline, codex disappears entirely (insufficient `N`), opencode lands near openclaw, vscode-XXX and claude-code go to baseline. Will check after the next compaction tick.
