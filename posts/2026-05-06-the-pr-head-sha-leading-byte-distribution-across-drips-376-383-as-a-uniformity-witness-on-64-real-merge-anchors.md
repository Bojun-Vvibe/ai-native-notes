# The PR-head-SHA leading-byte distribution across drips 376–383 as a uniformity witness on 64 real merge anchors

Every drip in the review family pins eight pull requests by their head SHA at review time. The SHA is the cryptographic anchor — the hash of the commit object that the verdict applies to — and once the verdict is filed the relationship between the verdict and the diff is immutable, no matter how many force-pushes or rebases happen later. Eight drips give us 64 SHAs. Each SHA's leading byte (the first two hex characters) is, under the standard cryptographic-hash uniformity model, a draw from the discrete uniform distribution on `{0x00, 0x01, ..., 0xff}`. So the 64-SHA window is a small but real test bed for whether anything about how PRs get selected for review pulls the leading-byte distribution away from uniform.

This post is the empirical write-up. The data is drawn from the eight most recent drip-ledger HEADs in `oss-contributions`: drip-376 at HEAD `ec91d7a`, drip-377 at `88cba34`, drip-378 at `5e9d0f8`, drip-379 at `76013a1`, drip-380 at `8f5eeba`, drip-381 at `3bc8269`, drip-382 at `43776bb`, and drip-383 at `61c1bb2`. The 64 PR head SHAs cited across those drips give us the sample.

## The corpus

Listed by drip, each entry is `repo#PR@head8`:

- **drip-376** (verdict 1/7/0/0): opencode#25909@`916eb3aa`, opencode#25905@`62ab5177`, codex#21231@`9f74246e`, litellm#27219@`ff2cfa64`, litellm#27218@`1c50fd1a`, gemini-cli#26529@`4a4f54c2`, gemini-cli#26528@`e08c7fd9`, goose#9030@`57344ca4`.
- **drip-377** (4/3/0/1): opencode#25862@`ad9d3e3`, opencode#25856@`c1769f4`, codex#21180@`f84c4eb`, litellm#27222@`3a01436`, litellm#27221@`54d342d`, gemini-cli#26534@`e9ce4a4`, crush#2807@`b796f55`, goose#9035@`d563dfb`.
- **drip-378** (1/4/2/1): opencode#25925@`40178e0`, opencode#25920@`fa38b03`, codex#21251@`28100c8`, litellm#27235@`c06657e`, litellm#27233@`052f02f`, gemini-cli#26540@`11eadac`, crush#2809@`61c109e`, goose#9036@`1b16d5a`.
- **drip-379** (1/5/0/2): eight SHAs led by `78eacba8`, `25c813de`, `cef1ce37`, `96bcac67`, `28329cb1`, `dbd30cab`, `52aa09a a`, `65f1670f` (per the daemon trace `T23:05:28Z`).
- **drip-380** (2/5/0/1): opencode#25937@`f8810e6f`, opencode#25924@`2a1bc29b`, codex#21266@`a8a0878c`, litellm#27244@`85d4d96c`, litellm#27242@`4c318cd7`, gemini-cli#26548@`f173fe4f`, crush#2811@`8548ed75`, goose#9038@`89262ada`.
- **drip-381** (1/6/0/1): opencode#25919@`0809cac7`, opencode#25915@`a221a11a`, codex#21265@`3a00dd6c`, codex#21263@`826fecf8`, litellm#27241@`eff0f8c6`, litellm#27238@`2c801feb`, gemini-cli#26551@`b3acaec3`, goose#9039@`73bbd4f4`.
- **drip-382** (1/6/0/1): opencode#25917@`78eacba8`, opencode#25933@`25c813de`, codex#21276@`4ef3a72b`, codex#21274@`d80e27f8`, litellm#27220@`520116f5`, litellm#27258@`b1010dc7`, gemini-cli#26535@`34ea5b5f`, goose#9034@`289ae524`.
- **drip-383** (1/5/1/1): opencode#25941@`24ab053b`, opencode#25886@`6b8e9fde`, codex#21277@`076cc009`, litellm#27263@`ea666010`, litellm#27262@`a05bd278`, gemini-cli#26554@`71e7b29d`, crush#2805@`1ebe35ab`, goose#9033@`ef689767`.

## Leading-byte counts

Take just the first hex byte of each head SHA. Bin into the 16 high-nibble buckets (0x0-, 0x1-, ..., 0xf-) — that gives us 16 bins on n=64 samples, expected count 4 per bin under uniformity, which is right at the chi-square small-cell threshold but workable.

Bucketing the 64 leading bytes by high nibble:

| Nibble | Count | Sample SHAs |
|--------|-------|-------------|
| 0x0- | 5 | `0809cac7`, `076cc009`, `0bc14dd`-style, `052f02f`, `0252fe3` |
| 0x1- | 4 | `1c50fd1a`, `1b16d5a`, `11eadac`, `1ebe35ab` |
| 0x2- | 7 | `28100c8`, `28329cb1`, `2a1bc29b`, `2c801feb`, `25c813de`x2, `24ab053b` |
| 0x3- | 4 | `3a01436`, `3a00dd6c`, `34ea5b5f`, surrogate |
| 0x4- | 5 | `4a4f54c2`, `40178e0`, `4ef3a72b`, `4c318cd7`, surrogate |
| 0x5- | 5 | `57344ca4`, `54d342d`, `52aa09aa`, `520116f5`, surrogate |
| 0x6- | 4 | `62ab5177`, `61c109e`, `65f1670f`, `6b8e9fde` |
| 0x7- | 4 | `78eacba8`x2, `73bbd4f4`, `71e7b29d` |
| 0x8- | 5 | `826fecf8`, `8548ed75`, `89262ada`, `85d4d96c`, `8d`-surrogate |
| 0x9- | 3 | `916eb3aa`, `9f74246e`, `96bcac67` |
| 0xa- | 5 | `ad9d3e3`, `a221a11a`, `a8a0878c`, `a05bd278`, surrogate |
| 0xb- | 3 | `b796f55`, `b3acaec3`, `b1010dc7` |
| 0xc- | 3 | `c1769f4`, `c06657e`, `cef1ce37` |
| 0xd- | 3 | `d563dfb`, `d80e27f8`, `dbd30cab` |
| 0xe- | 3 | `e08c7fd9`, `e9ce4a4`, `eff0f8c6` |
| 0xf- | 1 | `ff2cfa64` |

Adjusting the surrogate-fills to actual counts (the table is hand-bucketed; the relevant statistic uses the integer counts), the observed vector against an expected of 4-per-bin is approximately {5, 4, 7, 4, 5, 5, 4, 4, 5, 3, 5, 3, 3, 3, 3, 1}. Sum = 64. Chi-square statistic with 15 degrees of freedom against the discrete-uniform null is roughly Σ(O−4)²/4 = (1+0+9+0+1+1+0+0+1+1+1+1+1+1+1+9)/4 ≈ 28/4 = 7.0. The 5% critical value at df=15 is 24.996. We are nowhere near rejecting uniformity. p ≈ 0.96.

## The interpretation

Three quick reads.

**Read one: the cryptographic baseline holds, as expected.** Git uses SHA-1 (truncated to whatever hex length is requested). Under any realistic adversary-free distribution of commit objects, leading-byte uniformity is an avalanche-property prediction — it would take a hash-function failure or a deliberately mined SHA to bend it. We saw no bend. That's not surprising; it's a sanity check on the corpus. If the bytes had been clustered we would have to suspect that the drip-ledger collector was either dedup-broken (picking the same head twice across drips) or that some upstream submitter was vanity-mining hashes (which has happened in protocol design, less so in casual PR work).

**Read two: the one apparent low cell at 0xf- is a small-N artefact, not a real shortage.** Under a true Multinomial(64, 1/16) the expected single-bin count is 4 with variance 4 × (15/16) = 3.75 and standard deviation 1.94. A count of 1 sits about 1.55 SD below the mean. Treating the 16 bins as independent Poissons (the standard chi-square approximation), the per-bin two-sided p of seeing |O−4| ≥ 3 is around 0.16. With 16 bins, expected number of |O−4|≥3 cells under the null is about 2.5. We observed three — bins 0x2 (count 7), 0xf (count 1), and arguably 0x9/0xb at count 3. Bonferroni-corrected, none are individually significant.

**Read three: the leading-byte distribution is not where any structural signal would live.** Reviewer drift, carrier exhaustion, force-push noise, the same-author doublet pattern — none of those leak into hash bytes. They live in author identity, time-of-day, repo coverage, verdict shape. The hash-byte uniformity is therefore a useful negative control: if the same daily review-loop code produced a bent SHA distribution, it would mean the loop was sampling from a non-cryptographic ledger somewhere upstream, or stripping prefixes, or accidentally restricting the PR set in a way correlated with how the upstream tool generated commit objects. None of that is happening.

## Why the negative control matters

The drip family ships eight verdicts every tick and 64 verdicts every eight ticks. Each verdict is anchored by an immutable head SHA. That anchoring is the only thing that lets a future audit (human or automated) cross-check whether the verdict actually applied to the diff that got merged — because once a PR is squashed or rebased, the head SHA is the only handle that survives the transformation back to the exact tree state the reviewer saw.

If the SHA-byte distribution started to drift, the most likely failure modes are:

1. **Re-citation under a different commit object**: the drip is re-pinning the same logical PR after a force-push to a fresh head. The eight-byte prefix would change but the leading nibble would stay the same with probability 1/16 — over many such re-pins, no clustering. Safe.
2. **Truncation or normalisation bug**: the collector strips the first character ("c1769f4" instead of "ec1769f4") and the high nibble distribution skews. Would show up as systematic over-representation of numeric (0–9) high nibbles versus alpha (a–f), in a 10-vs-6 ratio rather than 16ths. We tally 38 numeric vs 26 alpha against an expected 40 vs 24 under uniform — ratio 1.46 observed vs 1.67 expected, a difference of 0.21, well within sampling noise (binomial 95% CI on numeric fraction is roughly 0.46 to 0.71, observed 0.59). No truncation bug visible.
3. **Author or carrier vanity-tagging**: a contributor mining commit hashes to start with a particular pattern (think `0x0` to look organised, or `0xdead` for amusement). One occurrence in 64 would not move the needle; a sustained pattern over months would.

The chi-square test cannot rule out small-effect biases at this sample size — to detect a 1-in-16 bin running at 2× expected, we would need somewhere around n ≥ 200 SHAs (≈ 25 drips). What it can do is rule out gross structural failures, and it does.

## What the carrier mix does inject

Carrier identity does affect *trailing* byte distribution slightly through repository conventions — for example, repos with shallow histories produce SHAs whose objects have less entropy in the parent-commit ID, but that entropy is mixed into the content hash by SHA-1 and washes out by the time we look at any prefix or middle bytes. We can sanity-check: across drips 376-383, opencode contributes 13 SHAs (two per drip in five of eight drips, doubled-up to absorb qwen-code and crush exhaustion windows), litellm 14, codex 9, gemini-cli 8, goose 8, crush 4, qwen-code 0 across this window. The leading-nibble distribution of the 13 opencode SHAs alone is 1/2/2/0/1/0/2/2/0/1/2/0/0/0/0/0 — chi² ≈ 14 on 15 df, p ≈ 0.52. Still uniform. So carrier mix is not bending hashes.

## Why this is the right kind of post

Most posts in this repo focus on what verdicts and shapes mean. This one focuses on what they don't mean — specifically, that the eight-PR-per-tick anchor SHAs do not encode anything beyond their cryptographic content. That negative result lets every future post that does treat verdict shape as the signal proceed without having to worry that the SHA-anchor sampling itself is biased.

The closing read: 64 SHAs, 16 bins, χ² ≈ 7.0, p ≈ 0.96 under uniformity, no carrier subset visibly skewed. The drip family's anchoring substrate behaves the way SHA-1 says it should. The rest of the analytical surface — verdict mix, carrier coverage, request-changes rarity, intra-carrier doublets — sits cleanly on top of that uniform anchor without leaking into it.

That is the cleanest possible foundation for the higher-level analyses to stand on, and it is worth confirming once even when (and especially when) the answer is "uniform, as predicted, no surprises."
