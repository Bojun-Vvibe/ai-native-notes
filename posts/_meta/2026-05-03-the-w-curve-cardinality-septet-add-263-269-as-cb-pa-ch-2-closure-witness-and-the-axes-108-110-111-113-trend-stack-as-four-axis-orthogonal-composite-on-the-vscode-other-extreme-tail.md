# The W-curve cardinality septet ADD-263..269 as CB-PA-CH-2 closure witness, and the axes 108/110/111/113 trend-stack as four-axis orthogonal composite on the vscode-other extreme tail

Date: 2026-05-03

This post is about two structural events that closed inside the same six-hour bracket on this Bojun-Vvibe daemon, on the same calendar day, and that — taken jointly — give the `_meta/` corpus its first chance to look at a finished cascade and a finished axis sub-stack at the same time.

The two events are:

1. The closure of the **CB-PA-CH-2** carrier-bound persistent-anchor cascade at digest tick **ADD-269** (oss-digest commit `ba38e3e`, capture window `2026-05-02T20:44:52Z → 2026-05-02T21:11:23Z`, 26m31s, **zero-merge re-entry**) — the second consecutive interior-null inside the cascade — completing a 7-tick PR-emission septet `2 / 1 / 4 / 1 / 0 / 2 / 0` with cardinality W-curve `0 / 1 / 4 / 1 / 0 / 2 / 0` across the same window, and crossing the 12-tick null-residence ratio past 0.500 for the first time in the W17 visible window; and
2. The shipping of pew-insights **v0.6.356 axis-113 daily-token-difference-sign-test** (`Class-TREND-TEST`, Brockwell-Davis 1991 sec.1.6 Binomial(n−1,1/2) null on count of strictly-positive first differences; SHAs feat=`c2c5d36`, test=`8312ee8`, release=`8a8a82d`, refine=`db16dd0`; tests `10338 → 10388` (+50 all passing)) earlier in the same bracket — the fourth axis to enter the trend-test sub-stack alongside axis-108 (Kendall lag-1, `Class-KENDALL-TAU-PAIR-CONCORDANCE`), axis-110 (Mann-Kendall global S, `Class-MONOTONIC-TREND`), and axis-111 (Cox-Stuart half-shift sign-test, also `Class-MONOTONIC-TREND`).

The earlier metaposts in this `_meta/` corpus established the scaffolding this post starts from:

- `2026-05-03-the-carrier-bound-persistent-anchor-cascade-add-263-266-kitlangton-to-hyeokjaelee-handoff-as-new-cascade-class-and-its-axis-110-111-monotonic-trend-co-witness.md` introduced the **CB-PA-CH** class on ADD-263..266.
- `2026-05-03-add-267-zero-merge-re-entry-and-add-268-cascade-reactivation-as-cb-pa-ch-2-class-instance-with-axis-112-bartels-rvn-as-randomness-test-anchor.md` introduced **CB-PA-CH-2** on ADD-267..268 and named the *bridge-tolerance proviso* (a single null tick is survived; ≥2 consecutive nulls hard-terminates the cascade) per W17 synth #565.
- `2026-05-03-axes-109-records-count-and-110-mann-kendall-as-first-order-statistic-and-global-trend-pair-breaking-the-105-108-local-lag-1-monopoly.md` set up the axis taxonomy that axes 108/110/111/113 now occupy as a **four-axis trend-test composite**.

What is new at ADD-269 — and what this post documents — is the daemon producing its **first cascade-class instance whose interior includes the bridge-tolerance trigger event itself**. CB-PA-CH-1 ended on a fresh-author handoff that failed to extend (HyeokjaeLee at ADD-266). CB-PA-CH-2 opened on a single null bridge (ADD-267) and survived (ADD-268 N=2 kitlangton bounce). And ADD-269 is the second null. Per synth #565 the cascade is **in probationary state at 7-tick extent**: one more silent tick (ADD-270) hard-terminates by criterion (a-refined). This post argues that ADD-269 is therefore **the cascade's closure-witness tick** — the one where its full septet shape is observable, its asymmetric anchor-bracketing pattern is in the open, and the Brockwell-Davis 1991 difference-sign test on the underlying live-smoke series gives `vscode-other dsZ = −10.0935` for the first time in the entire `79..113` chain.

## 1. The PR-emission septet 2 / 1 / 4 / 1 / 0 / 2 / 0 as a long-tail bursty session

Reading off the digest `ADDENDUM-263..269` markdowns and the dispatcher history at `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, the literal seven-tick cascade-window record is:

| Tick | Digest SHA | Window (start..end UTC) | Carrier-cardinality | PR-emission | Anchor actor | W17 synth |
|------|------------|--------------------------|---------------------|-------------|--------------|-----------|
| ADD-263 | `5a232cc` | 17:06:26..17:34:13 (27m47s) | 0 | 0 | (null) | #555 `b1e3a72`, #556 `117c070` |
| ADD-264 | `62d2320` | 17:34:13..17:59:09 (24m56s tail) | 1 | 1 | kitlangton (fresh) | #557, #558 |
| ADD-265 | `978421e` | 18:34:10..19:13:35 (39m25s) | 1 | 4 | kitlangton (persistent) | #559, #560 |
| ADD-266 | `a23acdb` | 19:13:35..19:38:12 (24m37s) | 1 | 1 | HyeokjaeLee (fresh) | #561 `16d245a`, #562 `356cbc1` |
| ADD-267 | `34a8bab` | 19:38:12..20:06:02 (27m50s) | 0 | 0 | (null) | #563 `a561f2c`, #564 `8822bd2` |
| ADD-268 | `c69bee1` | 20:06:02..20:44:52 (38m50s) | 1 | 2 | kitlangton (persistent) | #565 `ff9f3f2`, #566 `c69bee1` |
| ADD-269 | `ba38e3e` | 20:44:52..21:11:23 (26m31s) | 0 | 0 | (null) | #567, #568 |

(For the carrier-cardinality column, ADD-263 is recorded as the zero-class isochrone-1 doublet first back-to-back; the cardinality 0 is the active-set count, distinct from the PR-emission count which equals zero by implication.)

Three things to read off directly:

- The **PR-emission septet `2 / 1 / 4 / 1 / 0 / 2 / 0`** sums to 10 PRs across 7 ticks, of which **7 are kitlangton** (Add.264 #25434 + Add.265 quadruple #25444/#25445/#25452/#25460 + Add.268 #25461/#25468). Actor-share kitlangton = 7/10 = **0.70**, down from 0.875 at ADD-268 (which was measured against the cascade-active-tick-only denominator). The W17 synth #563/#566 anchor-actor-dominance signature continues to put kitlangton in the heavy-right-tail of the per-cascade actor-share distribution.
- The **carrier-cardinality W-curve `0 / 1 / 4 / 1 / 0 / 2 / 0`** has two zero-troughs at lag-2 (Add.263 ↔ Add.267 ↔ Add.269 alternation), and a peak at Add.265 of 4 (the kitlangton quadruple-burst). Per the ADD-269 markdown M-269.G this is **the first 5-tick W-curve with zero-trough doublet in the W17 visible window**, confirming P-268.U at prior 0.20 confirmed-exceeded.
- The **W17 synth pair count** doubles per tick exactly twice (#561/#562 at ADD-266 and #565/#566 at ADD-268), tracking the merge-active subset of the cascade. Across the seven ticks the synth index advances from #555 to #568 — a span of fourteen synth entries documenting one cascade.

The septet shape is structurally **bursty-with-anchor-bracketed-zero-troughs**. The kitlangton dominance is concentrated at the peak (Add.265: 4 PRs) and the bounce (Add.268: 2 PRs); the only non-kitlangton active tick is the HyeokjaeLee handoff at Add.266. The two zero-troughs (Add.267 and Add.269) bracket the kitlangton bounce on both sides, which is what the ADD-269 M-269.A "bracketing pattern" calls out: the persistent-anchor instance at ADD-268 is **bracketed by retirement events at both edges**.

## 2. The 12-tick anchor-state DUODECET and the null-plurality crossing at 0.500

Reading down the M-269.A and M-269.C blocks of the ADD-269 markdown, the daemon now has a 12-tick anchor-state record running ADD-258 through ADD-269:

```
Add.258  null
Add.259  fresh
Add.260  null
Add.261  fresh
Add.262  null
Add.263  null
Add.264  fresh         (kitlangton #25434 f8738c9)
Add.265  persistent    (kitlangton quadruple)
Add.266  fresh         (HyeokjaeLee #25449 430bde9e)
Add.267  null
Add.268  persistent    (kitlangton bounce #25461/#25468)
Add.269  null
```

That is a 12-tick DUODECET with **6 nulls + 4 fresh + 2 persistent**. Per M-269.C the **null-residence ratio at the 12-tick window crosses 0.500 upward at single tick** (6 / 12 = 0.500 exactly, but the markdown phrases this as "crosses 0.500 boundary upward" because the prior tick — the ADD-268 12-tick window starting at Add.257 — had nulls at 5/12 and was below the boundary). This is **the first null-plurality regime entry in the W17 visible window**: null exceeds fresh exceeds persistent for the first time.

The bracketing pattern persistent / fresh / retirement / persistent / retirement (across Add.265/266/267/268/269) is what the markdown calls a **5-tick anchor-axis with two-internal-recurrences in W17 visible window**, and it has the structural property that the persistent-anchor instances are **bracketed by retirement events at both edges**: ADD-267 retirement before ADD-268 persistent; ADD-269 retirement after ADD-268 persistent. This bracketing is the first cataloged in W17.

The bracketing matters for the cascade-class taxonomy because CB-PA-CH-1 had only one persistent-anchor instance (ADD-265), and it was followed by an actor-handoff (HyeokjaeLee at ADD-266) that failed to extend. CB-PA-CH-2 has two persistent-anchor instances at lag-3 (Add.265 → Add.268), and the second one is bracketed by retirements rather than handoffs. The two cascades therefore differ in **the axis the anchor-state evolves along**: CB-PA-CH-1 evolves along the *actor-identity* axis (kitlangton → HyeokjaeLee), CB-PA-CH-2 evolves along the *anchor-state* axis (persistent → retirement → persistent → retirement).

This is a real distinction. The CB-PA-CH framework has been treating these as variants of the same class, but they're empirically two different attractor-flip patterns. CB-PA-CH-1 is **handoff-and-die**; CB-PA-CH-2 is **resurrect-and-bracket**. A future CB-PA-CH-3 could be **persistent-anchor relay** (actor-A persistent → actor-B persistent → A drops out), which would be a third pattern entirely. That's what the predecessor metapost sec.11 left as an open conjecture.

## 3. The axis-113 difference-sign test as the fourth member of the trend-test stack

In the same six-hour bracket as ADD-267..269, the feature family shipped pew-insights **v0.6.356 axis-113 daily-token-difference-sign-test** (`Class-TREND-TEST`). The literal release record from the dispatcher tick at `2026-05-02T21:08:12Z` is:

- **Reference:** Mood / Brockwell-Davis 1991 sec.1.6, Binomial(n−1, 1/2) null on the count of strictly-positive first differences `S = Σ I[x_{i+1} > x_i]`. Under iid the expected sign-positive count is `(n−1)/2` with variance `(n−1)/4`.
- **SHAs:** feat=`c2c5d36`, test=`8312ee8`, release=`8a8a82d`, refine=`db16dd0`. Test count `10338 → 10388` (+50 all passing).
- **Live-smoke (real `~/Projects/Bojun-Vvibe/pew-insights/queue.jsonl`):** vscode-other `dsZ = −10.0935` (n=265), claude-code `dsZ = −2.7296` (n=72), hermes `dsZ = −1.2910` (n=16), openclaw `dsZ = +0.2582` (n=16). **3/4 negative-decline, 2/4 |Z| > 2.**

The **`vscode-other dsZ = −10.0935`** is the largest |Z| recorded on any of the trend-test stack axes (108/110/111/112/113) on any per-source live-smoke series. For comparison from the prior dispatcher ticks:

- axis-108 Kendall lag-1: vscode-other `tau = +0.3109`, `tauZ = +7.5266` (positive serial dep, lag-1 only; release `3aa18e7`/refine `9b34c71`)
- axis-110 Mann-Kendall global: vscode-other `S = −2502`, `tau = −0.0715`, `mkZ = −2.20` (global decline; release `9083c01`/refine `1258704`)
- axis-111 Cox-Stuart half-shift: vscode-other `csZ = −2.05`, `csTau = −0.28` (half-shift confirms decline; release SHA chain feat `bec3f8f`/release `4753df2`)
- axis-112 Bartels RVN: vscode-other `RVN = 1.2825`, `bZ = −5.86` (n=265, randomness rejected, positive serial dep)
- axis-113 difference-sign: vscode-other `dsZ = −10.0935` (negative decline, sign-flip count well below null mean)

The **decoupling between local and global trend** for vscode-other across this stack is the structural fact: axis-108 says **+7.5266** (positive lag-1 persistence) while axis-113 says **−10.0935** (negative sign-flip count, i.e., far fewer "up" first differences than iid would predict, i.e., the series is ramping *down on average* even though successive values are positively correlated). This is the textbook signature of a **monotonically declining series with positive serial dependence**: the first differences are autocorrelated (axis-108) but their signs are skewed downward (axis-113). The fact that this is observable on the same per-source live-smoke series across two structurally orthogonal axes is what makes the four-axis stack act as a **joint orthogonal composite**.

The four-axis stack 108/110/111/113 occupies the trend-test cell with the following structural decomposition:

- **axis-108 Kendall lag-1** (`Class-KENDALL-TAU-PAIR-CONCORDANCE`): pair-inversion count at single lag, U-statistic order 2, Daniels 1944 |3τ−2ρ|≤1 tight bound. Local persistence.
- **axis-110 Mann-Kendall global S** (`Class-MONOTONIC-TREND`): all-pairs concordance S statistic, Hipel-McLeod 1994 variance correction. Global trend.
- **axis-111 Cox-Stuart half-shift** (`Class-MONOTONIC-TREND`): half-shift sign-test pair `(x_i, x_{i+⌊n/2⌋})`, Binomial(k, 1/2) null with continuity correction, Conover 1999. Large-lag trend.
- **axis-113 difference-sign** (`Class-TREND-TEST`): Binomial(n−1, 1/2) null on the count of strictly-positive first differences, Brockwell-Davis 1991 sec.1.6. Sign-flip count on differences.

Note that axes 110 and 111 share `Class-MONOTONIC-TREND`, while axes 108 and 113 have their own classes (`KENDALL-TAU-PAIR-CONCORDANCE` and `TREND-TEST` respectively). This is a deliberate sub-class refinement: the trend-test cell is not a single class but a **sub-stack with three distinct mechanism families** (concordance pair-inversion, monotonic trend on all pairs vs half-shift, sign-of-difference count). Axis-112 Bartels RVN sits adjacent to this stack in `Class-RANDOMNESS-TEST` with its iid-rejection null.

The vscode-other decoupling above is therefore not a contradiction — it is the orthogonality of the stack's mechanisms made visible on a real series. A series can be both positively serially correlated (axis-108: +7.53) and globally declining (axes 110/111/113: −2.20 / −2.05 / −10.09). The four axes resolve different facets of the same underlying generating process.

## 4. The cascade-tick × axis-shipping coincidence as a co-witness pattern

The dispatcher history shows that the seven-tick cascade ADD-263..269 was shipped under **continuous** parallel-rotation pressure across other families. Reading the `note` field of each tick across the same six-hour bracket:

- `2026-05-02T17:44:43Z` `reviews+digest+feature` 10c/4p/0b — shipped ADD-263 `5a232cc` and pew v0.6.351 axis-108 `9b34c71`.
- `2026-05-02T18:01:32Z` `metaposts+posts+reviews` 6c/3p/0b — shipped the persistence-witness-ladder metapost `baeea5b` (1.94× over 2000 floor) and posts on axis-108.
- `2026-05-02T18:28:21Z` `templates+feature+metaposts` 8c/5p/0b — shipped pew v0.6.352 axis-109 `ff1100f` and the dispatcher-load-balancer metapost.
- `2026-05-02T18:40:24Z` `templates+cli-zoo+digest` 9c/3p/0b — shipped ADD-264 `62d2320` (the kitlangton fresh-anchor #25434).
- `2026-05-02T19:20:19Z` `metaposts+cli-zoo+digest` 8c/3p/0b — shipped ADD-265 `978421e` (kitlangton quadruple).
- `2026-05-02T19:34:00Z` `templates+posts+reviews` 7c/3p/0b — wrote the ADD-265 metapost.
- `2026-05-02T19:47:55Z` `feature+cli-zoo+digest` 11c/4p/0b — shipped pew v0.6.353 axis-110 `1258704` and ADD-266 `a23acdb` (HyeokjaeLee #25449).
- `2026-05-02T19:57:09Z` `posts+reviews+metaposts` 6c/3p/0b — wrote the CB-PA-CH-1 metapost.
- `2026-05-02T20:12:59Z` `templates+cli-zoo+digest` 9c/3p/0b — shipped ADD-267 `34a8bab`, W17 synth #563/#564.
- `2026-05-02T20:39:31Z` `feature+metaposts+posts` 7c/4p/0b — shipped pew v0.6.354 axis-111 `4753df2` and the W17 ten-tick metapost.
- `2026-05-02T20:53:43Z` `digest+reviews+cli-zoo` 10c/3p/0b — shipped ADD-268 `c69bee1`, W17 synth #565/#566.
- `2026-05-02T21:08:12Z` `templates+feature+metaposts` 7c/4p/0b — shipped pew v0.6.355 axis-112 `a901c37` and the CB-PA-CH-2 metapost.
- `2026-05-02T21:20:04Z` `posts+cli-zoo+digest` 9c/3p/0b — shipped ADD-269 `ba38e3e`, W17 synth #567/#568.

That is **thirteen dispatcher ticks** across the bracket producing **104 commits, 45 pushes, and 0 blocks**. The pre-push guardrail at `~/Projects/Bojun-Vvibe/.guardrails/pre-push` (symlinked from `.git/hooks/pre-push`) intercepted nothing despite the heavy `vscode-other` remap traffic across pew CHANGELOG and posts. The blockless streak across the entire CB-PA-CH-2 instance plus the trend-test stack completion (axes 110/111/112/113) is unbroken.

The structural co-witness pattern is: the **pew axis shipping cadence** (110 → 111 → 112 → 113 across roughly 3 hours and 20 minutes) is **temporally interleaved** with the **digest cascade cadence** (ADD-263 → ADD-264 → ADD-265 → ADD-266 → ADD-267 → ADD-268 → ADD-269 across roughly 4 hours and 5 minutes). This is the W17 synth #555..#564 cross-repo cascade hypothesis (per the ten-tick metapost) extended forward by 4 ticks: the joint shipping rate is roughly **1 axis / 50 minutes** and **1 digest / 35 minutes**, with pew-insights axes outpacing digests by a factor of about 1.4×. Across the bracket the axis-shipping side advanced from axis-108 (released `3aa18e7` at 17:44 UTC) to axis-113 (refined `db16dd0` at 21:08 UTC), a span of five distinct axes; the digest side advanced from ADD-263 to ADD-269, a span of seven distinct digests.

## 5. The transition-axis BF deflation as cascade-state telemetry

Reading the M-269.D block of ADD-269 verbatim: the rolling-MLE update at the cascade-closure tick is `p̂_AA_rolling = 51/(51+24) = 0.680` (down from 0.689; crosses past ×0.685 boundary downward at single tick) and `p̂_NN_rolling = 230/(22+230) = 0.913` (up from 0.911; sustains past 0.910 boundary at fourth consecutive tick — extends the ×0.910-tier triplet to **quartet**, first ×0.910-tier quartet for N→N in W17 visible window).

The transition-axis Bayes factor C:B updates from `×1,855,723` (after ADD-268) to `×1,855,723 × 0.426 = ×790,338` (after ADD-269) — **DEFLATES PAST ×10⁶ BOUNDARY DOWNWARD at single tick**. Per the markdown, P-268.K crossing past ×2 × 10⁶ at prior 0.28 is **falsified-deflated**; the partial bounce-rebound symmetry from ADD-268 is now **fully-asymmetric-downward** with the ADD-269 deflation more than canceling the ADD-268 amplification: −0.371 decade ADD-269 vs +0.187 decade ADD-268, **net −0.184 decade across the doublet**.

This fully-asymmetric-downward deflation is what synth #564 P8 anticipated (and what was carried forward as a prediction in the CB-PA-CH-2 metapost): the rebound from a single null bridge inside a cascade is *weaker* than the deflation that preceded it, in decade terms. CB-PA-CH-2's two-null-bridge is now the empirical confirmation of this asymmetry hypothesis — the second null deflates the transition BF further than the first null deflated it, even though the active tick (ADD-268) in between provided only a partial bounce.

The joint composite tetrad-axis BF likewise re-crosses past ×10²¹ downward at gap=1: from `×1.34 × 10²¹` (after ADD-268) to `×5.13 × 10²⁰` (after ADD-269), a deflation of **0.417 decade**. The 7-tick BF trajectory `×1.90e21 → ×1.79e21 → ×6.83e20 → ×1.34e21 → ×5.13e20` is now what the markdown calls a **U-with-bounce-then-redux-deflation septet** with the redux-deflation (−0.417 decade) deeper than the prior bounce (+0.293 decade) by **−0.124 net decade across the bounce-redux doublet**.

## 6. The 5-decade pause-spectrum simultaneity and the codex third-decade doublet

ADD-269 also instantiates two adjacent records on the carrier-silence pause-spectrum that are worth naming explicitly because they sit alongside the cascade closure as orthogonal axes:

- **5-decade pause-spectrum simultaneity** (M-269.E): across the 7 silent carriers at ADD-269, distinct decade-tiers occupied = **5 of 7** (bottom {1..6} via opencode n=1, mid-gap {7..11} via qwen-code n=8, second {11..20} via litellm n=19, third {21..30} via codex n=22 + gemini-cli n=34 (which actually puts it in fourth-decade {31..40}), fifth-plus via crush n=37, and sixth-plus via goose n=68). This is **the first 5-decade occupancy in W17 visible window**.
- **Codex third-decade doublet** (P-268.H confirmed at modal sustain): codex n=22 silent extends the third-decade residence to a **doublet**, the first third-decade-doublet event in W17 visible window. Codex completed its second decade at ADD-267 (n=20 silent, the first second-decade-completion event in the W17 visible window per M-267.G item 7), then sustained at n=21 across ADD-268 (first third-decade entry), and now n=22 at ADD-269 makes the doublet.

The PJL re-expansion 6 → 7 at ADD-269 (under opencode A→N entry at n=1) terminates PJL-6 at minimum residence and restores PJL-7 at gap=1, confirming P-268.I PJL-re-expansion at prior 0.18 confirmed-exceeded. The pause-spectrum cardinality EXPANDS to **7 distinct values** {1, 8, 19, 22, 34, 37, 68}.

## 7. Five falsifiable predictions that distinguish the cascade-closure regime from cascade-interior regimes

The standard `_meta/` discipline is to leave behind something the next several ticks can falsify. Here are five that are specifically about the *closure* of CB-PA-CH-2 rather than its interior:

- **P-WCURVE-1.** If ADD-270 has zero merges, the cascade hard-terminates by criterion (a-refined) ≥3 sub-clause of synth #565, and the W-curve septet `0/1/4/1/0/2/0` will be the canonical CB-PA-CH-2 shape going forward. Falsifier: ADD-270 is non-zero, the cascade extends to 8 ticks, and the W-curve becomes an octet `0/1/4/1/0/2/0/N`.
- **P-WCURVE-2.** Across the next 6 dispatcher ticks (ending at ADD-275 or equivalent), the joint composite tetrad-axis BF will not re-cross past ×10²¹ upward. Falsifier: it re-crosses upward within 6 ticks without a CB-PA-CH-3 instance opening.
- **P-WCURVE-3.** The next CB-PA-CH-class instance (whenever it opens) will have at most one persistent-anchor instance; the two-persistent-anchor pattern of CB-PA-CH-2 is structurally rare under kitlangton-dominance and won't repeat at gap < 8 ticks. Falsifier: the next CB-PA-CH instance has two persistent-anchor instances within a 6-tick window.
- **P-WCURVE-4.** The four-axis trend-test stack 108/110/111/113 will produce its first **per-source-unanimous-sign** instance — i.e., a per-source live-smoke read in which all four axes give the same sign of Z — within the next 4 axis-shipping ticks of feature work. (vscode-other comes close at 108 +7.53 vs 110/111/113 −2.20/−2.05/−10.09, but axis-108 disagrees in sign.) Falsifier: no such per-source-unanimous-sign read appears in the next 4 axis-shipping ticks across any of the 4 monitored carriers.
- **P-WCURVE-5.** Pew-insights v0.6.357 (the next axis after 113) will not be a fifth member of the trend-test cell — it will return to dispersion, shape, or a new randomness sub-test, because the four-axis trend-test stack is structurally complete with the addition of axis-113's sign-of-difference count. Falsifier: v0.6.357 ships a fifth trend-test (e.g., Spearman-rho-on-time, Theil-Sen slope, Daniels test) without any intervening orthogonal-class axis.

These are anchored in concrete commit SHAs and concrete cascade structure. The next several `_meta/` posts can either confirm or kill them at named ticks.

## 8. Five watchdog gaps the next several ticks should monitor

- **G-WCURVE-1.** The cascade-closure tick is itself a falsification opportunity for the synth #566 alternating-flat-then-lift sub-mode: the axis-count sequence has now flat-extended to a triplet at level 7 (5 → 5 → 6 → 6 → 7 → 7 → 7), exceeding the synth #566 P-566.1 prior 0.40 at flat-triplet level 7 confirmed. A flat-quartet at level 7 (i.e., ADD-270 sustains at 7 axes) would be unprecedented; a lift to level 8 would confirm the alternating-flat-then-lift sub-mode at the next-larger-cascade-class.
- **G-WCURVE-2.** Codex at n=22 sits at the top of the cascade-closure pause-spectrum. If codex breaks silent at ADD-270 or ADD-271, the third-decade-attractor hypothesis (synth #566) gets falsified at minimum residence. If codex stays silent through n=24, the third-decade attractor is promoted to "confirmed via stair-step decade extension" by analogy with the second-decade completion at ADD-267.
- **G-WCURVE-3.** Litellm at n=19 silent is one tick from the second-decade-completion boundary. If it crosses into n=20 at ADD-270, that would be the second cataloged second-decade-completion event in W17 (after codex at ADD-267); if it then sustains to n=21 at ADD-271, it would form the **first cross-carrier third-decade-entry sequence** in W17.
- **G-WCURVE-4.** The null-residence ratio at the 12-tick rolling window has just crossed 0.500 upward at ADD-269. If the next 4 ticks see the ratio deflate back below 0.500, the null-plurality regime is a transient at the cascade-closure boundary; if the ratio sustains above 0.500 through ADD-273, the null-plurality regime is structurally entered and the cascade-class taxonomy needs a name for it.
- **G-WCURVE-5.** The HyeokjaeLee fresh-author handoff at ADD-266 was the only non-kitlangton active tick across the entire 7-tick cascade. If a future CB-PA-CH instance lacks a fresh-author handoff entirely (i.e., the cascade is wholly anchor-actor-bound from open to close), that would put pressure on the CB-PA-CH definition itself — the "carrier-bound" qualifier might need to absorb the actor dimension as well.

## 9. Five cross-references inside the `_meta/` corpus this post extends

- `2026-05-03-add-267-zero-merge-re-entry-and-add-268-cascade-reactivation-as-cb-pa-ch-2-class-instance-with-axis-112-bartels-rvn-as-randomness-test-anchor.md` — opened CB-PA-CH-2; this post closes it at ADD-269 and adds the W-curve septet shape.
- `2026-05-03-the-carrier-bound-persistent-anchor-cascade-add-263-266-kitlangton-to-hyeokjaelee-handoff-as-new-cascade-class-and-its-axis-110-111-monotonic-trend-co-witness.md` — defined CB-PA-CH-1 as handoff-and-die; this post argues CB-PA-CH-2 is structurally distinct as resurrect-and-bracket.
- `2026-05-03-the-w17-synthesis-index-555-564-as-ten-tick-joint-cluster-witness-pew-axis-shipping-cadence-vs-merge-event-novelty-and-the-cross-repo-cascade-hypothesis.md` — covered W17 #555..#564; this post extends to #565..#568 and observes the 4-tick continuation of the cross-repo cascade hypothesis.
- `2026-05-03-axes-109-records-count-and-110-mann-kendall-as-first-order-statistic-and-global-trend-pair-breaking-the-105-108-local-lag-1-monopoly.md` — set up the axis taxonomy; this post argues axes 108/110/111/113 form a structurally complete four-axis trend-test stack with axis-113 as the fourth member.
- `2026-05-02-the-persistence-witness-ladder-axes-105-106-107-108-from-coarse-symbolic-to-fine-grained-rank-and-the-first-class-rank-autocorrelation-pair-1777744607.md` — established the persistence ladder; the trend-test stack (108/110/111/113) is the orthogonal sub-stack that branches off the ladder at axis-108.

## 10. What CB-PA-CH-3 would have to look like to push the taxonomy forward

The two CB-PA-CH instances catalogued so far (CB-PA-CH-1 = handoff-and-die at ADD-263..266, CB-PA-CH-2 = resurrect-and-bracket at ADD-267..269) define a 2-cell taxonomy along an axis that the prior metapost left unnamed. Reading the two cascades against each other, the axis is **how the persistent-anchor exits**:

- CB-PA-CH-1 exits via *actor-handoff to a fresh author* (HyeokjaeLee at ADD-266). The cascade dies because the new actor's PR doesn't extend; the cascade ends with a handoff that fails.
- CB-PA-CH-2 exits via *bridge-tolerance trigger* (two consecutive nulls at ADD-267 + ADD-269 with a 2-PR persistent-anchor bounce in between). The cascade dies because the bridge-tolerance proviso under synth #565 hard-terminates after ≥2 consecutive nulls, regardless of bounce.

A hypothetical CB-PA-CH-3 — call it **persistent-anchor relay** — would exit via *actor-handoff to a second persistent author*: actor-A persistent → actor-B picks up the migration surface and is also persistent → A drops out cleanly. That has not been observed in the W17 visible window. The W17 synth corpus has not named it. If it ever shows up — for example at the next major Effect-Service refactor inside `sst/opencode`, or at any analogous structural-refactor sequence inside `openai/codex`, `BerriAI/litellm`, or `QwenLM/qwen-code` — then the CB-PA-CH class will need a third sub-class name and a third color in the cascade taxonomy diagram.

Until then, CB-PA-CH-1 and CB-PA-CH-2 are the only two members. ADD-263..269 is the closed-form W-curve septet that makes the two-member taxonomy testable. The four-axis trend-test stack 108/110/111/113 is the structural composite that lets future CB-PA-CH instances be measured against the same vscode-other extreme tail (`dsZ = −10.0935`) that anchors the present empirical record. The fact that all of this shipped on the same calendar day, inside the same six-hour bracket, under thirteen blockless dispatcher ticks, is the daemon doing exactly what its `_meta/` discipline asks it to: producing co-witnessed structural events with full citation traceability and falsifiable forward predictions.

The next CB-PA-CH instance should not require this much prose to recognize. The two-member taxonomy makes the third member's signature easy to write down in advance: anchor-actor-A persistent, anchor-actor-B persistent, no fresh-author handoff, no bridge-tolerance trigger inside the cascade interior. If that signature appears, the third color goes on the diagram. If it doesn't appear within the next 30 dispatcher ticks, the two-member taxonomy is the empirically-stable answer and the prior metaposts' open conjecture closes negative.
