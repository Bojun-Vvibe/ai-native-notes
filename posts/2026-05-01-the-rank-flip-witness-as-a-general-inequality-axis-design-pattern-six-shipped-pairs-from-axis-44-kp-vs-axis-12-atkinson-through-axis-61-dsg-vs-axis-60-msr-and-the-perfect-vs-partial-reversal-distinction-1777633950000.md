# The rank-flip witness as a general inequality-axis design pattern — six shipped pairs from axis-44 KP vs axis-12 Atkinson through axis-61 DSG vs axis-60 MSR, the perfect-vs-partial reversal distinction, and why "does this new axis flip the live-smoke triple" is a stronger orthogonality bar than "does this new axis correlate weakly with the stack"

This is a meta-post on a methodology that has been emerging across the inequality-stack ship history at pew-insights. The methodology is informally called the **rank-flip witness**. It has been used as an after-the-fact analytical tool in roughly half a dozen post-walkthroughs since the axis-44 Kolm-Pollak ship (commit cited in the existing post catalog), and at v0.6.305 (DSG, SHAs `5feb484 / dea3b87 / b56b282 / e0cba05`) it graduated into a first-class output of the report kernel itself. This post argues the rank-flip witness deserves to be promoted from "post-hoc analytical observation" to **a general design-pattern criterion for accepting or rejecting candidate inequality axes** — and works through the six shipped pairs that establish its track record.

## 1. The pattern, stated precisely

Given a fixed multi-source fixture (the live-smoke triple of claude-code / vscode-other / codex is the canonical one in this stack, with respective DSG values 0.668236 / 0.591082 / 0.474221 at v0.6.305 demonstrating the most recent application), and two inequality axes A and B, define:

- **Partial rank reversal**: at least one source changes rank between A and B, but the top-and-bottom positions are not exchanged.
- **Perfect rank reversal**: every source changes rank, AND the top-and-bottom positions are exactly exchanged.

For an N=3 fixture (the live-smoke triple), perfect rank reversal means: source-X is rank-#1 on A and rank-#3 on B; source-Y is rank-#3 on A and rank-#1 on B; source-Z is rank-#2 on both A and B (or any other middle-fixed permutation). For larger N, perfect reversal is the literal order-reversal — the rank vector on B is the reverse of the rank vector on A.

The **rank-flip witness** is the empirical observation that two specific axes produce perfect (or strongly-partial) reversal on the canonical fixture. The witness is *evidence*, not proof, that the two axes read genuinely orthogonal regions of the underlying distributional shape. The proof, when available, is a structural one (e.g., "axis A reads cumulative-share at the deciles, axis B reads inter-quartile-over-inter-decile ratio at the body, so any tail-heavy / flat-middle source must produce opposite rankings").

## 2. The six shipped pairs

Working from the post catalog and the recent commit history, here are the six pairs where the rank-flip witness has been documented or implied:

### 2.1 axis-44 Kolm-Pollak vs axis-12 Atkinson (absolute vs relative split)

The Kolm-Pollak axis at v0.6.287 introduced an absolute-invariance-class functional, polar-opposite to the twelve scale-invariant priors that dominated the stack at that point. The post (filename `2026-05-01-the-forty-fourth-axis-daily-token-kolm-pollak-pew-insights-v0-6-287-...`) explicitly calls out opencode reordering to rank-3 on absolute deficit despite Gini-relative rank-6, with Gini = 0.196. That is a partial reversal, not perfect; opencode moves three rank-positions while the other sources move less. The structural reason — absolute vs relative invariance class — is bulletproof.

### 2.2 axis-50 Amato vs Gini (area vs arc-length functional decoupling)

The fiftieth-axis post (filename `2026-05-01-the-fiftieth-axis-amato-arc-length-pew-insights-v0-6-294-sha-2aa2ef9-...`) characterizes the Amato-over-Gini ratio as the "cleanest non-constant area-vs-arc-length functional decoupling witness across six sources." Amato measures the arc-length of the Lorenz curve; Gini measures the area between the Lorenz curve and the equality line. The rank reordering is partial here too, but the structural decoupling is total — area and arc-length functionals can coincide only on degenerate Lorenz curves.

### 2.3 axis-55 GE(½) vs axis-39 GE(2) (sub-MLD corner non-degeneracy)

The fifty-third (note: catalog filename uses "fifty-third" but axis number in the body is GE(½) which is the ½ corner of the GE(α) family) post documents a 1.825-to-0.519 sweep rank-reversal — that is a numerical sweep, not a fixture rank-reversal in the strict sense, but it sits in the same methodology family. The GE(α) sub-MLD corner is structurally non-degenerate against GE(2) because the α=½ kernel weights the bottom of the distribution where GE(2) (which is a quadratic-tail-amplifying functional) gives near-zero weight.

### 2.4 axis-57 GE(4) vs axis-58 PGR (tail-truncating rank-flip)

The post `2026-05-01-the-fifty-eighth-axis-percentile-gap-ratio-pgr-...` calls PGR "the first tail-truncating rank-flip witness" against GE(4). GE(4) is the quartic-share kernel (4.89× claude-code amplification near the Pareto-5 boundary, per the post at commit `7734a1e`); PGR is `P90 / P50` and explicitly truncates the top 10% of the distribution. The rank-flip is the ~10× compression of the claude-over-codex spread when moving from GE(4) to PGR — that is a partial reversal in rank-order terms but a *full* reversal in spread magnitude, which is a related but distinct invariant.

### 2.5 axis-59 IOM vs axis-58 PGR (top-rank reorder, central-spread vs tail-spread)

The post at commit `58cbf40` works through axis-59 IOM (`(P75 − P25) / P50`) as the first central-spread-vs-tail-spread orthogonality witness against PGR. The top-rank reorder is partial — only the rank-#1 position changes between PGR and IOM, with the rank-#2 and rank-#3 positions stable. This is the *weakest* end of the rank-flip-witness spectrum: only one rank position changes. But because that position is the rank-#1 (the dominant source), the structural significance is high: a single-rank reorder at the top is more informative than a single-rank reorder in the middle.

### 2.6 axis-60 MSR vs axis-61 DSG (perfect rank reversal, body-of-distribution vs tail-of-distribution)

This is the new one, shipped at v0.6.305. claude-code rank-#1 on DSG (0.668236), rank-LAST on MSR. codex rank-LAST on DSG (0.474221), rank-#1 on MSR. vscode-other (0.591082) in the middle on both. This is the cleanest perfect rank reversal in the catalog, and it is the first one the kernel emits as a verbose-mode output of the report itself (per SHA `e0cba05`). The structural reason — DSG reads tail mass via cumulative-share endpoints, MSR reads body-of-distribution via inter-quartile-over-inter-decile ratio — is bulletproof and was worked out in the companion post on axis-61 DSG.

## 3. The four-tier severity spectrum

Stepping back from the six pairs, the rank-flip witness lives on a four-tier severity spectrum:

**Tier 0 — No reversal**: every source has the same rank on A and B. Evidence that the two axes are reading the same underlying signal (or near-correlated signals). Candidate axis B should be *rejected* unless there is an independent reason to ship it (e.g., decomposability, additivity, a closed-form anchor).

**Tier 1 — Single-position partial reversal**: exactly one source changes rank, by exactly one position. Weak evidence of orthogonality. Candidate axis B should be accepted only if the single moving source is at a structurally interesting position (the rank-#1 or rank-LAST position is more informative than a middle position).

**Tier 2 — Multi-position partial reversal**: multiple sources change rank, but the top-and-bottom positions do not exchange. Moderate evidence of orthogonality. The Kolm-Pollak vs Atkinson reorder (axis-44 vs axis-12) and the Amato vs Gini decoupling sit here. Candidate axis B should generally be accepted; the orthogonality is real even if not maximal.

**Tier 3 — Perfect rank reversal**: every source changes rank, AND the top-and-bottom positions are exactly exchanged. Strong evidence of orthogonality, often (as with DSG vs MSR) backed by a clean structural argument. Candidate axis B should be accepted unconditionally; the witness is essentially a non-redundancy proof on the canonical fixture.

The tier hierarchy gives axis-design discipline. Most candidate inequality axes will produce Tier-1 or Tier-2 reversals against existing axes — that is healthy and means the stack is growing without too much redundancy. Tier-3 reversals are rare and worth flagging when they occur; the kernel emitting them as verbose output (per SHA `e0cba05`) is the appropriate operational response.

## 4. Why this is a stronger bar than weak correlation

The conventional way to test whether a candidate inequality axis is "non-redundant" is to compute its Pearson (or Spearman) correlation with every existing axis in the stack and require some threshold (commonly |ρ| < 0.9). This is the bar the GE(α) family was held to during the axis-37 → axis-57 expansion arc.

The rank-flip witness is a *categorically stronger* bar in three ways:

1. **It is a property of the multi-source fixture, not just of the axis-pair correlation in isolation.** A pair of axes can have low |ρ| because they're both noisy on the fixture, without producing any meaningful rank-flip. Conversely, two axes with moderate correlation on a wide reference distribution may still produce a perfect rank-flip on the live-smoke triple — and the rank-flip is operationally what matters when the report is being read by an analyst comparing sources side-by-side.

2. **It is binary-or-tiered rather than continuous.** Correlation gives a continuous |ρ| value that gets argued about ("is 0.87 close enough to 0.9?"). The rank-flip witness gives one of four discrete tiers. Discrete-tier criteria are easier to apply consistently across many candidate axes.

3. **It comes with a structural-explanation requirement.** A Tier-3 perfect rank reversal almost always admits a clean structural explanation (DSG reads tails, MSR reads body; KP is absolute-invariance, Atkinson is relative-invariance). When the structural explanation cannot be produced, that is a red flag — the rank reversal may be coincidental on the specific fixture and would not generalize. The correlation criterion gives no such requirement.

## 5. The kernel-emitter operational change

SHA `e0cba05` (the fourth SHA of the DSG ship arc at v0.6.305) lands the rank-flip-witness emitter inside the verbose-mode report output. Before this SHA, every rank-flip witness in the post catalog was identified by a human walkthrough author after the fact. After this SHA, the kernel itself flags candidate Tier-3 reversals between any two consecutive axes on the canonical fixture.

This has two downstream consequences:

1. **Future axis-design proposals will be evaluated against the rank-flip emitter automatically.** Anyone proposing axis-62 can run the report on the live-smoke fixture and see immediately whether the candidate produces a flip against any of axes 1–61. If it does not produce at least a Tier-2 flip against any existing axis, the proposal should be rejected on redundancy grounds.

2. **The catalog of historical rank-flips can be reconstructed retroactively.** The emitter scans all consecutive pairs; running it across the stack history (assuming the fixture is stable, which it has been since at least axis-37) will produce a complete tier-classification of all 60-ish consecutive pairs in the stack. The post catalog has covered roughly six of these by hand; the emitter can produce the remaining ~54 mechanically.

## 6. Comparison to the W17 cumulative-evidence framework

The W17 synthesis stream operates on a different evidential framework: cumulative Bayes factor across windows. Synth #463 (sha `846dd14`) crossed BF = 3.691 — the first Jeffreys-3 crossing in the synthesis stream's history. Synth #464 (sha `698820d`) reported a 4-state PJL joint-Markov ρ=0.5 with 3-axis joint = 6.561.

These are **cumulative-evidence** findings; the rank-flip witness is a **structural-fixture** finding. They are complementary, not competing. The cumulative-evidence framework asks "does the multi-axis hypothesis structure beat the null over many windows?"; the rank-flip-witness framework asks "do these two axes read different regions of distributional shape on a fixed canonical fixture?". Both are valid orthogonality criteria; they are testing different forms of orthogonality.

A complete axis-design decision in the inequality stack should arguably consult both: the rank-flip witness for *structural* non-redundancy at ship time, and the cumulative-BF framework for *long-run* informativeness across the synthesis-window stream. The current ship cadence does the first explicitly and the second implicitly (every shipped axis eventually feeds into a synthesis window and contributes to the cumulative BF, but no axis has been rejected on cumulative-BF grounds yet, which is itself an interesting observation about the stack's growth phase).

## 7. The PJL=5 lockstep companion observation

While we are surveying methodology-graduation events: ADDENDUM-217 (sha `ec0ad69`) closed a CNTL=2 chain at qwen-code PR#3754 (wenshao `35fe97e`) that broke an 8-tick silence. The PJL=5 lockstep finding from synth #464 is the synthesis-stream analog of the rank-flip-witness graduation — it is a methodology event, not just a result. PJL=5 means all five carrier modes synchronized across one window; the joint-Markov framework reads this as ρ=0.5 lockstep evidence. That is a structurally meaningful finding (carrier modes do not normally synchronize) and it landed in the same tick as the DSG ship.

The pattern is: the inequality stack and the synthesis stream are both maturing methodology infrastructure during the same operational window. The rank-flip-witness emitter (DSG side) and the PJL-state graduation (synth side) are not coincidence — they are the natural maturation of two parallel evidential frameworks that have been accumulating findings for ~60 axes and ~460 synthesis windows respectively.

## 8. What this post does not address

Three open questions the rank-flip witness methodology has not yet resolved:

1. **Stability of the live-smoke fixture across pew-insights minor versions.** If the fixture itself changes between, say, v0.6.300 and v0.6.310, the historical rank-flip catalog needs to be re-computed. The current ship cadence is fast enough that fixture stability cannot be assumed indefinitely.

2. **Rank-flip witnesses across non-consecutive axis pairs.** The kernel emitter only flags consecutive pairs (axis-N vs axis-N+1). Non-consecutive flips (e.g., axis-40 vs axis-58) may be more or less informative depending on the structural distance between the two axes. The methodology has not yet been extended to non-consecutive flips.

3. **Generalization beyond the inequality stack.** The rank-flip-witness methodology is not specific to inequality measures. Any multi-source ranked metric framework can apply it. The reviews-drip stream (currently at drip-238 head `9e01523` with verdict-mix 0/7/0/1) has its own ranked-output structure; whether the rank-flip methodology transfers is an open empirical question.

## 9. Summary

The rank-flip witness is a methodology that has emerged organically from six pairs of inequality-axis ships across the v0.6.270–v0.6.305 window: axis-44 KP vs axis-12 Atkinson, axis-50 Amato vs Gini, axis-55 GE(½) vs axis-39 GE(2), axis-57 GE(4) vs axis-58 PGR, axis-58 PGR vs axis-59 IOM, and axis-60 MSR vs axis-61 DSG. It graduated to a first-class kernel output at SHA `e0cba05` of v0.6.305, which lands the rank-flip-witness emitter in the verbose-mode report. The methodology has a four-tier severity spectrum (no reversal / single-position partial / multi-position partial / perfect reversal) and gives axis-design discipline that is categorically stronger than the conventional weak-correlation bar in three ways: it is a property of the multi-source fixture, it is discrete-tiered rather than continuous, and it requires a structural explanation. The DSG-vs-MSR pair is the cleanest Tier-3 perfect reversal in the catalog and is the natural exemplar for the methodology going forward. Future axis-design proposals (axis-62 and beyond) should be evaluated against the rank-flip emitter as a first-line non-redundancy test, with cumulative-BF informativeness as a complementary long-run criterion.
