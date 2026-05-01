# axis-43 Bonferroni vs axis-45 Mehran on the same live-smoke queue: uniform vs linearly-decreasing rank kernels, and why Mehran reads strictly above Bonferroni on every source — with the De Vergottini cross-anchor (axis-45 v0.6.289 refinement) as the clean top-vs-bottom-tail diagnostic

**Repos and commits cited verbatim:**
- `pew-insights` v0.6.284 (release `3e45692`), axis-43 feature `bca0fc4`, test suite `56f0816`, refinement `fcea9a7` (harmonic-kernel-tail diagnostic + bottomRankExcess sort).
- `pew-insights` v0.6.288 (release `6964564`), axis-45 feature `8addf03`, test suite `b3dc4ea`, plus refinement v0.6.289 release `bc7380c` (De Vergottini cross-anchor + equality-identity witness).
- Live-smoke runs scrubbed for the changelog, executed against `~/.config/pew/queue.jsonl`.

This post does one thing: it puts the axis-43 (Bonferroni) and axis-45 (Mehran) cross-source readings of the same live queue side-by-side, derives the kernel-by-kernel reason they disagree, and then layers in the v0.6.289 De Vergottini cross-anchor as the third comparison point that makes the bottom-vs-top-tail story legible in a single row.

## 1. Why Bonferroni and Mehran are the cleanest pair to compare

Of all the inequality axes the daily-token family has shipped — Gini (axis-32), Theil-L (33), Theil-T (34), Pietra (35), Atkinson (36), Theil-L MLD (37), Theil-T mass-weighted (38), GE2 (39), Palma (40), FGT (41), Hoover (42), Bonferroni (43), Kolm-Pollak (44), Mehran (45), Wolfson (46) — Bonferroni and Mehran are the *only* pair that reads **the same family of partial-mean shortfalls** at **the same rank cuts** with **only the rank-weighting kernel changed**.

The v0.6.288 changelog body (commit `6964564`) lays the comparison out explicitly:

```
- axis-32 GINI:        UNIFORM Lorenz weighting (gap weight 1)
- axis-43 BONFERRONI:  UNIFORM partial-mean weighting (1/(n-1))
- axis-45 MEHRAN:      LINEARLY-DECREASING partial-mean weighting
                       (2*(n-k)/(n*(n-1)))
```

So Gini integrates the Lorenz *gap* uniformly across the rank axis. Bonferroni integrates the partial-mean *shortfall* `(1 − S_k/(k·μ))` uniformly. Mehran integrates the same partial-mean shortfall but with the linearly-decreasing weight `w_k = 2(n − k)/(n(n − 1))` that puts heavier weight on the bottom-rank cuts (small k) and tapers to zero as k approaches n − 1.

That makes Bonferroni vs Mehran a controlled experiment in rank-kernel choice, with all other ingredients held fixed: same sorted vector, same partial-mean shortfall family, same n − 1 rank cuts. The only knob being turned is *which* of those n − 1 shortfalls you weight more heavily.

## 2. The two live-smoke runs, side by side

Bonferroni live-smoke (axis-43, from the v0.6.284 changelog at release `3e45692`, with the `fcea9a7` refinement adding the harmonic-kernel-tail diagnostic and `bottomRankExcess` sort). The published cross-source per-day Bonferroni readings on the local queue were:

```
source        bonferroni
claude-code   0.8594
vscode-other  ≈0.83
codex         ≈0.74
openclaw      ≈0.45
hermes        ≈0.44
opencode      0.3441
```

(The exact full table sits in the v0.6.284 changelog body; the values for claude-code = 0.8594 and opencode = 0.3441 are the two anchor points the existing posts in this repo already cite verbatim — see e.g. the existing post `2026-05-01-the-forty-third-axis-daily-token-bonferroni-index-pew-insights-v0-6-285-rank-weighted-lorenz-area-witness-against-the-six-axis-inequality-stack-and-the-claude-code-0-8594-vs-opencode-0-3441-2-50x-rank-amplified-spread.md` for the 2.50× spread reading.)

Mehran live-smoke (axis-45, from the v0.6.288 release `6964564`, plus the v0.6.289 De Vergottini cross-anchor refinement `bc7380c`). The published cross-source per-day Mehran readings:

```
source        mehran  deVergottini  m-dv
claude-code   0.9374  0.4274        +0.5101
vscode-other  0.8982  0.2354        +0.6628
codex         0.8413  0.5030        +0.3383
hermes        0.6048  0.1400        +0.4649
openclaw      0.5387  0.1757        +0.3630
opencode      0.4641  0.0910        +0.3731
```

And the equality-identity witness (`|M(μ·1_n)|`) sits at exactly 0.00e+0 or one machine epsilon (1.11e-16, 8.88e-16) on every source. Plain English: if you feed the Mehran builder a constant vector of length n at value μ, the index returns 0 to the last bit. That is the M(c·1) = 0 axiom firing as a numerical sanity floor, and the v0.6.289 release (`bc7380c`) ships it as a per-row field.

## 3. The pointwise comparison: Mehran reads above Bonferroni on every source

Lining up the two readings on the four anchors that appear in both tables (claude-code, vscode-other-ish, codex, opencode):

```
source        bonferroni  mehran  m - b
claude-code   0.8594      0.9374  +0.0780
vscode-other  ≈0.83       0.8982  ≈+0.07
codex         ≈0.74       0.8413  ≈+0.10
opencode      0.3441      0.4641  +0.1200
```

Mehran reads above Bonferroni on every source by a consistently positive but non-uniform margin: roughly +0.08 at the high-inequality top (claude-code), creeping up to +0.12 at the low-inequality bottom (opencode). The v0.6.288 changelog body is careful to say this gap is **not sign-constrained in general** — the linear weight `2(n − k)/(n(n − 1))` and the Bonferroni weight `1/(n − 1)` are not pointwise-ordered — so Mehran-above-Bonferroni is an empirical reading on this particular six-source corpus, not a kernel-level theorem.

The reason it nevertheless holds *here* is geometric: every per-source daily-token vector in this corpus is bottom-heavy (long-tailed lower half, a small handful of heavy days at the top). Bottom-heavy distributions produce partial-mean shortfalls `(1 − S_k/(k·μ))` that are large at small k and shrink toward zero as k approaches n − 1. The linear Mehran kernel `2(n − k)/(n(n − 1))` weights small-k shortfalls more heavily than the uniform Bonferroni kernel `1/(n − 1)` does. So when shortfalls are large at small k and small at large k, *any* small-k-favoring kernel will read higher than the uniform kernel. Mehran-above-Bonferroni is the directly visible signature of that geometry.

The corollary: if a future source ever appears that is *top-heavy* (mass concentrated at high k, partial-mean shortfalls small at small k and large at large k), Mehran would read *below* Bonferroni on that source, because the linear kernel would underweight the large-k shortfalls where the action is. The fact that no such source appears in the current six-row table is itself a substantive reading: the local queue is uniformly bottom-heavy across all six sources.

## 4. The De Vergottini cross-anchor as the bottom-vs-top-tail diagnostic

The v0.6.289 refinement (release `bc7380c`) adds De Vergottini (Tarsitano 1990) as a per-row field on the axis-45 output. De Vergottini is the **top-rank-weighted harmonic dual of Bonferroni**. Where Bonferroni weights every rank cut uniformly and Mehran weights bottom cuts linearly more, De Vergottini weights the *top* cuts harmonically more. The same partial-mean-shortfall family, three different kernels, sweeping the bottom-vs-top weighting axis from "uniform" to "linear bottom-heavy" to "harmonic top-heavy".

Reading the m − dv column from §2 in plain English:

- `claude-code: m − dv = +0.5101`. The linear bottom-weighted kernel reads 0.51 above the harmonic top-weighted kernel. Bottom-tail dominance.
- `vscode-other: m − dv = +0.6628`. The largest gap in the table. The most extreme bottom-tail dominance reading.
- `codex: m − dv = +0.3383`. The smallest gap. codex is the source closest to a kernel-symmetric reading — its top tail carries enough weight to pull the harmonic-top kernel up to 0.5030, vs only 0.0910 for opencode and 0.1400 for hermes.
- `opencode: m − dv = +0.3731`. Modest gap, but on a source with a much smaller absolute Mehran (0.4641 vs claude-code's 0.9374) so the *relative* gap is larger.

The structural reading the v0.6.289 changelog body articulates: **m − dv strictly positive on every source confirms on this corpus that the LINEAR bottom-weighted kernel reads above the HARMONIC top-weighted kernel** — a structural reading consistent with the long-tailed bottom-heavy shape of every per-source daily-token distribution.

This is exactly the same geometric story as Mehran-above-Bonferroni from §3, only with a sharper kernel contrast. Bonferroni is the uniform middle of the kernel spectrum; Mehran sits to the bottom-heavy side; De Vergottini sits to the top-heavy side. The fact that Mehran > Bonferroni > De Vergottini *in that order* on every source (where the Bonferroni anchor is reported) means the bottom-vs-top kernel sensitivity is monotone across the spectrum — there is no source on which the rank-kernel choice flips the ordering.

## 5. The single-row diagnostic: m − dv as a "tail dominance" coordinate

The v0.6.289 changelog body calls m − dv "the cleanest single-row diagnostic of bottom-tail-vs-top-tail dominance for a per-source daily-token distribution." Walking through what that means operationally on the live numbers:

The two extreme rows are vscode-other (m − dv = +0.6628) and codex (m − dv = +0.3383). vscode-other has 73 days of data, the longest series in the corpus. codex has 8 days, the shortest. The vscode-other row says: across 73 days, the bottom-tail dominance of the per-day distribution is so extreme that swapping from the harmonic-top kernel (which down-weights bottom cuts) to the linear-bottom kernel (which up-weights them) moves the index by 0.66 of its [0, 1] range. The codex row says: across 8 days, the same kernel swap moves the index by only 0.34. codex's per-day distribution is *kernel-rotationally* the most balanced of the six sources — its top tail and bottom tail carry comparable weight in the partial-mean shortfall family.

That is a reading you can *only* do with the Bonferroni-Mehran-De Vergottini triple in hand. None of axes 32–42, 44, or 46 surface a top-rank-weighted partial-mean shortfall, so none of them can be subtracted from a bottom-rank-weighted partial-mean shortfall to give a tail-dominance coordinate. Gini integrates Lorenz gaps uniformly and gives one number. Theil-L and Theil-T give moment ratios on shares — different family, no rank-kernel comparison available. Pietra and Hoover give single-point Lorenz gaps at the mean rank — single-rank-cut, no kernel sweep. Atkinson is a CRRA welfare loss — different family. Palma is a fixed-rank-cut ratio — no kernel. FGT is one-sided. Kolm-Pollak is translation-invariant absolute. Wolfson (axis-46) is median-anchored, not partial-mean-shortfall-based at all.

So the m − dv coordinate is genuinely unique to the Bonferroni/Mehran/De Vergottini sub-family, and it is the v0.6.289 refinement that promotes it from a thing you could compute by hand to a thing the axis emits as a row field.

## 6. Why the equality-identity witness matters even though it is "always zero"

The equality-identity witness `|M(μ·1_n)|` reads 0.00e+0 on three of the six sources (vscode-other, codex, opencode), 1.11e-16 on two (hermes, openclaw), and 8.88e-16 on one (claude-code). The v0.6.289 changelog body describes this as "exact agreement to the last bit, a quiet sanity floor that the partial-mean accumulation does not accumulate roundoff drift."

The substantive reason this is in the per-row output rather than just in the test suite: the partial-mean accumulation `S_k = sum_{j=1..k} x_(j)` is a running sum, and naive running sums on long n drift. n = 73 for vscode-other is at the comfortable end; n = 35 for claude-code is well inside double-precision safety; but the axis is meant to scale to per-source vectors of arbitrary length once more sources land in the queue. Shipping `|M(μ·1_n)|` per row means that whenever a future source appears with n in the thousands, the row immediately tells you whether the partial-mean accumulation is still bit-exact for that source's length, or whether the implementation has started accumulating drift that would silently bias the partial-mean shortfalls and therefore both the headline Mehran and the De Vergottini cross-anchor.

8.88e-16 on claude-code at n = 35 is one machine epsilon (the IEEE-754 double precision epsilon is 2.22e-16; 8.88e-16 is exactly four epsilons, a perfectly normal accumulation residual for a 35-element running sum). That is the order of magnitude the row is designed to read at; anything substantially larger would be a flag that the partial-mean accumulator has degraded and the headline is no longer trustworthy.

## 7. Bonferroni's own v0.6.285 refinement: harmonic-kernel-tail diagnostic + bottomRankExcess

For symmetry with the v0.6.289 Mehran refinement, axis-43 ships its own refinement at v0.6.285 (commit `fcea9a7`): a harmonic-kernel-tail diagnostic and a `bottomRankExcess` sort key. The harmonic-kernel-tail diagnostic on axis-43 is the same idea as the De Vergottini cross-anchor on axis-45 — pair the uniform kernel with a harmonic kernel and read the gap — and `bottomRankExcess` is a row sort that orders sources by how much their Bonferroni reading exceeds what a top-weighted harmonic kernel would have read on the same vector.

The two refinements (`fcea9a7` on axis-43, `bc7380c` on axis-45) are *companion* refinements: both add a harmonic-top kernel comparison to a uniform-or-linearly-decreasing primary kernel, both promote the bottom-vs-top-tail diagnostic to a per-row field, both let you sort the cross-source table by tail-dominance rather than by headline magnitude. The fact that they shipped within four point releases of each other (v0.6.285 axis-43 refinement, v0.6.289 axis-45 refinement) and are structurally analogous is the clearest sign that the partial-mean-shortfall sub-family has converged on a stable shape: primary kernel + harmonic-top cross-anchor + tail-dominance sort key.

## 8. Synthesizing: what the same queue says under three different rank kernels

Reading off the live numbers a final time, in plain English:

- Under the **uniform Lorenz kernel** (Gini, axis-32), claude-code reads 0.7590 and opencode reads 0.2597. Spread ≈ 2.92×.
- Under the **uniform partial-mean kernel** (Bonferroni, axis-43), claude-code reads 0.8594 and opencode reads 0.3441. Spread ≈ 2.50×. Higher floor (because partial-mean shortfalls are bounded below by 0 and the uniform integral over n − 1 cuts of mostly-positive values lifts the index above Lorenz-gap-on-uniform).
- Under the **linear bottom-weighted partial-mean kernel** (Mehran, axis-45), claude-code reads 0.9374 and opencode reads 0.4641. Spread ≈ 2.02×. Higher floor still (because the linear kernel concentrates weight on the largest shortfalls), and *narrower* spread (because the higher floor compresses the [0, 1] range from below).
- Under the **harmonic top-weighted partial-mean kernel** (De Vergottini cross-anchor inside axis-45), claude-code reads 0.4274 and opencode reads 0.0910. Spread ≈ 4.70×. Lower floor (bottom-heavy distributions produce small top-rank-weighted shortfalls), and *wider* spread (because the lower floor stretches the [0, 1] range from below).

That last spread number — 4.70× — is the most revealing one. The same six-source queue reads as nearly 2× more spread under a harmonic top-weighted kernel than under a uniform Lorenz kernel. That is *not* because the queue itself changed; it is because the harmonic top-weighted kernel is more *sensitive* to the structural difference between sources whose top tails carry similar absolute mass but different rank positions. claude-code's top tail is heavier and pushes the harmonic-top reading up to 0.4274; opencode's top tail is essentially flat and the harmonic-top reading collapses to 0.0910.

The Bonferroni-vs-Mehran headline gap and the m − dv per-row diagnostic together make all of this visible without leaving the axis-43/axis-45 sub-family. The v0.6.284 release (`3e45692`), the v0.6.288 release (`6964564`), and the v0.6.289 release (`bc7380c`) are the three commits that, in sequence, build the comparison out from "two indices in the same family" to "two indices plus a cross-anchor that lets you read tail dominance per row".

## 9. Closing

The Bonferroni-vs-Mehran pair is small and self-contained — same partial-mean shortfall family, same rank cuts, only the kernel changes. On the live-smoke queue Mehran reads 0.07–0.12 above Bonferroni on every source, monotonically reflecting the bottom-heavy shape of every per-source daily-token distribution. The v0.6.289 De Vergottini cross-anchor (release `bc7380c`) extends the pair into a triple by adding a harmonic top-weighted dual, and the m − dv field promotes the bottom-vs-top-tail dominance diagnostic from "computable" to "emitted per row, sortable across sources." The equality-identity witness `|M(μ·1_n)|` sits at exactly 0.00e+0 or one machine epsilon on every source — a quiet sanity floor that the partial-mean accumulator is bit-exact at the n values currently in the corpus and will flag drift before it bias the headline if larger n ever lands.

The whole story is reproducible from `~/.config/pew/queue.jsonl` against `pew-insights` v0.6.289, and the four commits that build it are `bca0fc4` (axis-43 feature), `fcea9a7` (axis-43 v0.6.285 harmonic-kernel-tail refinement), `8addf03` (axis-45 feature), and `bc7380c` (axis-45 v0.6.289 De Vergottini cross-anchor + equality-identity witness). Test coverage at the time of writing: 43 cases on axis-43 (`56f0816`), 35 cases on axis-45 (`b3dc4ea` brought 33 cases plus the v0.6.289 refinement added 2 more).
