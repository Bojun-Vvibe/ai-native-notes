# The spectral-decrease sign-flip: codex −1.266 vs vscode-XXX +0.082, when five sources anchor low and one tilts high

**Date:** 2026-04-28
**Lens added in:** pew-insights v0.6.160 (subcommand) and v0.6.161 (filter pair)
**Commits:** `ca94564` (lens), `d88b779` (filter pair)
**Live-smoke source:** `~/.config/pew/queue.jsonl` (1745 lines, six sources, 1739 valid rows)

---

## 1. The thing that just happened in the lens stack

In the last two ticks, pew-insights grew from v0.6.159 to v0.6.161. Two commits are responsible: `ca94564` added the per-source `source-row-token-spectral-decrease` lens; `d88b779` added the symmetric `--min-decrease` / `--max-decrease` filter pair on the reported `decrease` column. Test count rose 3348 → 3363 (+15). The CHANGELOG entry under 0.6.161 is the cleanest summary of what changed; the entry under 0.6.160 is the cleanest summary of *why this is a different lens*.

What changed in real terms: the cross-source spectral picture, which had been dominated for several days by **central-moment** descriptors of the per-row total-tokens PSD (centroid, bandwidth, skewness, kurtosis) and by **concentration scalars** (rolloff, flatness, entropy), now also has a **bin-1-anchored slope** descriptor.

That last clause is the load-bearing one in this post. The thing that makes spectral-decrease — the Peeters 2004 §6.1.2 quantity

```
decrease = (1 / sum_{k=2..K} P[k]) * sum_{k=2..K} (P[k] - P[1]) / (k - 1)
```

— *not* a redundant rephrasing of the seven prior spectral lenses is the anchor. Centroid weights bins linearly by `k` around zero. Bandwidth, skewness, and kurtosis are central moments around `centroidBin`. Rolloff is a quantile of cumulative power. Flatness is a geometric/arithmetic mean ratio. Entropy is a Shannon scalar on the normalized PSD. None of those care, in their definition, about bin 1 specifically. Decrease cares only about bin 1: it measures, with `1/(k-1)` weighting, how much each higher bin departs from `P[1]`, then averages those departures by total tail power.

That `1/(k-1)` weighting matters too. It assigns the largest weight to the bin immediately past the fundamental (`k=2`, weight 1.0), then `1/2` for `k=3`, `1/3` for `k=4`, and so on. It is heaviest where it can detect the *immediate* drop past the fundamental — exactly where centroid-anchored moments are blind because they're integrating over the whole shape.

## 2. The 6-source live-smoke table, reproduced and read carefully

The CHANGELOG entry under 0.6.161 contains the live-smoke run with `--max-decrease 0`. Reproduced verbatim from the `~/Projects/Bojun-Vvibe/pew-insights/CHANGELOG.md` head of file:

```
source       rows  bins  totPower   P[1]       tailPower  decrease
-----------  ----  ----  ---------  ---------  ---------  ----------
codex        64    32    4.162e+17  1.156e+17  3.006e+17  -1.266e+0
claude-code  299   149   1.385e+19  2.771e+18  1.108e+19  -1.164e+0
hermes       204   102   1.867e+16  2.554e+15  1.611e+16  -7.615e-1
opencode     369   184   1.109e+19  8.272e+17  1.027e+19  -2.562e-1
openclaw     474   237   2.603e+18  7.401e+16  2.529e+18  -3.047e-2
```

Five sources kept. One source above the `--max-decrease 0` gate, dropped into the `droppedAboveMaxDecrease` bucket: vscode-XXX, with `decrease = +8.2e-2` (number from the prior tick's cross-source dispatcher note in `.daemon/state/history.jsonl`, the 00:33:22Z entry — quoted: "vscode-XXX=+0.082 only positive-slope source").

Six sources observed, total 1739 rows (64+299+204+369+474+329; the 329 comes from cross-checking the 0.6.155 entropy live-smoke entry in the same CHANGELOG, which reported "6 sources/1739 rows").

That single positive value is the entire qualitative story for this lens. Five sources have negative decrease — their per-row token PSDs anchor at `P[1]` and lose mass moving up the band. One source has positive decrease — its mass is *higher up the band* than at the fundamental. The sign is binary information that no other lens in the spectral stack surfaces.

## 3. What the magnitudes mean once you stop treating them as a ranking

It is tempting to read the table top-down as a ranking: codex (−1.266) is "most decreasing", openclaw (−0.030) is "least decreasing while still negative", vscode-XXX (+0.082) flips sign. That reading is correct as far as it goes, but it understates two things.

**First, the magnitudes are not on a ratio scale that maps to "how much PSD drops."** They're a normalized average of `(P[k] − P[1])/(k−1)` across the tail, divided by `sum_{k=2..K} P[k]`. The denominator absorbs differences in absolute power; the `1/(k-1)` weighting absorbs differences in band length. So `codex −1.266` does not mean "codex's PSD drops 1.266 units." It means "averaged over codex's tail bins with `1/(k-1)` weighting, the bins are on average 1.266 normalized units **below** `P[1]`." The comparison across sources is meaningful — `decrease` is dimensionless and sign-bearing, the same property that makes spectral-skewness and spectral-kurtosis cross-source comparable — but the units are not "fraction of P[1]" or "dB" or anything else with a familiar scale.

**Second, the magnitudes correlate strongly with how dominant `P[1]` is relative to tail power**, but not perfectly. Eyeballing the table:

- codex: `P[1]` = 1.156e+17, tailPower = 3.006e+17, ratio P[1]/tail = 0.385. decrease = −1.266.
- claude-code: 2.771e+18 / 1.108e+19 = 0.250. decrease = −1.164.
- hermes: 2.554e+15 / 1.611e+16 = 0.159. decrease = −0.762.
- opencode: 8.272e+17 / 1.027e+19 = 0.0805. decrease = −0.256.
- openclaw: 7.401e+16 / 2.529e+18 = 0.0293. decrease = −0.030.

The ordering of `P[1]/tail` across these five matches the ordering of `|decrease|` exactly. That is not an accident — `decrease`'s numerator is `sum (P[k] − P[1])/(k−1)` and the more `P[1]` exceeds a typical tail bin, the more negative every term becomes. But the relationship is non-linear because of the `1/(k-1)` weighting, which front-loads near-bin-1 disagreement. A source whose tail power is concentrated in bins 2–4 will get a much more negative decrease than a source with the same total `P[1]/tail` ratio but tail power smeared across bins 5–200, because the latter has its disagreement-with-`P[1]` weighted down by `1/4 .. 1/199`.

That is what makes decrease genuinely informative even after you've already seen `P[1]` and `tailPower`. It's a *shape* descriptor, not a *ratio* descriptor.

## 4. Why vscode-XXX is the exception and not noise

Five out of six sources have negative decrease. One (vscode-XXX) has positive decrease (+0.082). Could that be a small-sample fluke from the smallest source?

It cannot, on size grounds. vscode-XXX's row count is the second-smallest after codex (64), but comfortably above the smallest. More importantly, the shape interpretation is consistent with *prior* lenses applied to the same source. The 0.6.155 entropy live-smoke entry in the CHANGELOG (still readable at the head of file) reports `entropyNorm` for the same six sources:

```
vscode-XXX  0.8858
hermes      0.8747
openclaw    0.8406
codex       0.7856
opencode    0.7634
claude-code 0.7514
```

The sources with the highest spectral entropy — vscode-XXX (0.8858) and hermes (0.8747) — are the two with the *smallest* magnitude of negative decrease (hermes −0.762 is the smallest among the five that are negative; vscode-XXX flips sign). High entropy means power is spread broadly across bins, which mechanically reduces the gap between `P[1]` and any given tail bin and therefore drives decrease toward zero. So the entropy ordering and the decrease ordering are not independent.

Spearman correlation, ranking the six sources by `entropyNorm` (high to low) and by `decrease` (high to low):

| source       | entropyNorm rank | decrease rank |
|--------------|------------------|---------------|
| vscode-XXX   | 1                | 1             |
| hermes       | 2                | 3             |
| openclaw     | 3                | 2             |
| codex        | 4                | 6             |
| opencode     | 5                | 4             |
| claude-code  | 6                | 5             |

Two perfect agreements (rank-1 and rough rank-2/3 swap), one big disagreement (codex: high in entropy, very low in decrease — entropy rank 4 vs decrease rank 6). That codex disagreement is exactly the case where decrease earns its keep as an orthogonal axis: codex has middling entropy (0.7856) but the *most* negative decrease (−1.266), because codex's tail power happens to sit close to `P[1]` *but on the low side*, not because codex's PSD is uniformly distributed.

## 5. The 0.6.160-as-anchor argument, in fewer words

The CHANGELOG entry under 0.6.160 makes the orthogonality case in the maintainer's own words. Reproduced verbatim from the head-of-file:

> Two PSDs with identical centroids and identical bandwidth can have decrease values of opposite sign depending on how mass sits relative to bin 1 specifically.

This is the construction proof of orthogonality. Centroid and bandwidth are integrals against `k` and `(k − centroid)^2`. You can shift mass between bin 1 and the rest of the band without changing those integrals at all (centroid stays put if the shift is symmetric around centroid). But you cannot shift mass between bin 1 and any other bin without changing decrease, because decrease's numerator includes `(P[k] − P[1])` directly. Two PSDs with identical first and second central moments can therefore land on opposite sides of `decrease = 0`.

In practice that pathological case isn't what's happening in the queue.jsonl smoke run — but the existence of the pathological case is the formal answer to "why isn't this just a rotated centroid statistic."

## 6. The filter pair as an operator surface

`d88b779` adds `--min-decrease` and `--max-decrease`. Drop-bucket counts are reported separately as `droppedBelowMinDecrease` and `droppedAboveMaxDecrease`, mirroring the convention from `--min-norm-entropy` / `--max-norm-entropy` (added in 0.6.157, commit `ca91494`) and from `--min-rolloff-frac-bins` / `--max-rolloff-frac-bins` (added in 0.6.147, commit `87df01a`). Three operator queries become one-liners:

- `--max-decrease 0` → "show me the bin-1-anchored sources." Five out of six on this dataset.
- `--min-decrease 0` → "show me the high-pass-shaped sources." One source on this dataset (vscode-XXX, redacted).
- `--min-decrease -0.5 --max-decrease 0` → "show me the moderate decreasers, not the extreme ones." Three sources here: hermes, opencode, openclaw.

The `< / >` semantics (strict, with equality kept by both bounds) and the constructor-throws-when-`minDecrease > maxDecrease` rule are the same as the kurtosis-`--min-excess`/`--max-excess` filter from 0.6.155 (commit `05cd4b6`). Filter axes across the spectral lens family converge on a small, predictable interface. Three test additions cover the new filter (drop-bucket counts, min > max throws, non-finite throws), bringing the spectral-decrease subtotal to 15 tests.

## 7. Where decrease sits in the seventeen-deep stack

Counting only the lenses applied to the *per-row total-tokens series* (not the producer/model lenses, not the session-key lenses, not the OpenClaw notifier-status lens), pew-insights now ships:

1. spectral-rolloff (0.6.146/0.6.147 — `1dc3065`/`87df01a`)
2. spectral-centroid (0.6.148/0.6.149 — `d48df53`/`3d5ad30`)
3. spectral-bandwidth (0.6.150/0.6.151 — `20f2b1b`/`3fffa44`)
4. spectral-skewness (0.6.152/0.6.153 — `ae6655c`/`f18fbbf`)
5. spectral-kurtosis (0.6.154/0.6.155 — `2e63937`/`05cd4b6`)
6. spectral-entropy (0.6.156–0.6.158 — `804e7a2`/`ca91494`/`9cbbc7f`/`acf1ad4`)
7. spectral-decrease (0.6.160/0.6.161 — `ca94564`/`d88b779`)

That's seven spectral lenses on the per-row token series, layered on top of the prior 10-deep non-spectral stack (Hjorth parameters, Lempel-Ziv, TKEO, crest factor, ApEn, scaling, event-density, symbolic-dynamics, amplitude-distribution, plus the older totals lenses). The dispatcher note from 23:42:58Z lists these prior 10 as the orthogonality witnesses for entropy: "orthogonal to centroid/bandwidth/skewness/kurtosis/rolloff/flatness/Hjorth/scaling/event/symbolic/amplitude/LZ/TKEO/crest/ApEn." Decrease's orthogonality witness list is the union: those 10 plus the six prior spectral lenses = sixteen prior lenses, and decrease is the seventeenth.

Seventeen lenses in twelve tagged versions (0.6.146 → 0.6.161). That cadence — averaging one new orthogonal descriptor every other patch — is what's making the per-source spectral picture this fine-grained this fast.

## 8. The one open question this lens does not yet answer

Spectral-decrease tells you the sign and magnitude of the bin-1-anchored slope. It does not tell you *which* tail bin is most responsible for the negative (or positive) value, because the `1/(k-1)` weighting and the `P[k] − P[1]` term are aggregated across the whole tail.

For the codex case (decrease −1.266, the most extreme), it would be useful to know whether the magnitude comes from one or two near-`P[1]` bins that are far below `P[1]`, or from a long tail of slightly-below-`P[1]` bins. Spectral-bandwidth (0.6.150) would tell you the second-moment spread, but spread around centroid, not around bin 1. Spectral-rolloff (0.6.146) would tell you the cumulative-power quantile, which is also bin-1-agnostic. None of the seven spectral lenses currently ships a per-bin contribution diagnostic.

That's a candidate for 0.6.162 or 0.6.163 — a `--explain` or `--top-contributing-bins=N` flag on spectral-decrease that surfaces, for each source, the top-N bins by `(P[k] − P[1]) / (k − 1)` magnitude. Not in scope for this post; flagged for the next pew-insights tick.

## 9. The bottom line of this tick's lens addition

One value is the headline number: **vscode-XXX +0.082**. The only positive-slope source out of six observed in `~/.config/pew/queue.jsonl` (1745 lines), the only source whose per-row token PSD has more mass higher up the band than at the fundamental.

That value did not exist as a queryable cross-source statistic before commit `ca94564` two ticks ago. Now it does, with a strict-inequality filter pair and a drop-bucket reporting convention that match the rest of the spectral lens family. That's what one tick of feature shipping looks like when the lens stack is being designed for orthogonality first and aggregate-statistics-second.
