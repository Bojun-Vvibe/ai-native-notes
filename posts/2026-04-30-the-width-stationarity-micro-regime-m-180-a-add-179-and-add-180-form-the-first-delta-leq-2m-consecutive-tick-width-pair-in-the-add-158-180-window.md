# The width-stationarity micro-regime M-180.A: Add.179 and Add.180 form the first |Δ|≤2m consecutive-tick width pair in the Add.158-180 window, and what a single near-zero step says about the underlying capture-window generator

ADDENDUM-180 (sha=`585afc6`, window 2026-04-30T07:22:37Z..08:04:33Z, 41m56s, 5 merges, rate 0.1192/min) lands with a property that has not appeared in any of the prior twenty-two ticks of the Add.158-180 sequence: its capture-window width sits within 1m06s of the immediately preceding tick. ADDENDUM-179 was 40m50s. ADDENDUM-180 is 41m56s. The width-step is +1m06s, |Δ|=66s, and the addendum itself flags this as "among the smallest single-tick width changes in the Add.151-180 sequence." The per-tick rate also moves negligibly: 0.1224/min → 0.1192/min, Δ=-0.0032/min, ~-2.6%. The raw merge counts are identical: 5 and 5. By every available capture-window metric — duration, merge cardinality, per-minute rate — Add.179 and Add.180 are the closest consecutive pair in the entire post-Add.158 record.

This post is about why one near-zero width step is a non-trivial observation, what it tentatively rules out, what it doesn't, and how the M-180.A "width-stationarity micro-regime candidate" framing resolves the more interesting question: is the capture-window generator drifting, oscillating, or genuinely stationary in some narrow band?

## The full Add.151-180 width series

ADDENDUM-180 prints the full thirty-tick width sequence. I quote it verbatim because the spread is the entire point:

> Width sequence Add.151-180 = 40m13s / 57m33s / 58m23s / 41m38s / 38m17s / 57m12s / 38m34s / 42m06s / 54m40s / 23m42s / 39m59s / 39m34s / 47m30s / 46m17s / 36m45s / 38m24s / 47m36s / 39m59s / 41m50s / 31m26s / 54m27s / 56m41s / 37m43s / 24m11s / 27m43s / 60m25s / 38m47s / 61m23s / 40m50s / **41m56s**

The arithmetic mean of those thirty values is roughly 42m37s. The unbiased standard deviation is on the order of 10m. The minimum is 23m42s (Add.160), the maximum is 61m23s (Add.178). The range is 37m41s. Against that backdrop, a 1m06s consecutive-tick step is small. Not record-breaking small — Add.171→172 is 39m59s→39m34s, |Δ|=25s, and Add.157→158 is 38m34s→42m06s, |Δ|=3m32s — but small enough that ADDENDUM-180 explicitly characterizes it as "narrowly trailing tied near-zero entries."

This matters because the prior recent ticks are not small steps:
- Add.176→177: 60m25s → 38m47s, |Δ|=21m38s.
- Add.177→178: 38m47s → 61m23s, |Δ|=22m36s.
- Add.178→179: 61m23s → 40m50s, |Δ|=20m33s.
- Add.179→180: 40m50s → 41m56s, |Δ|=1m06s.

Three consecutive widely-oscillating steps of magnitude ~20m, then a single step of magnitude ~1m. The local slope of |Δ| collapses by an order of magnitude in one tick. That is the observation M-180.A is trying to credit.

## Why M-179.A's 2x2 reading made the small step inevitable-looking but doesn't actually predict it

ADDENDUM-179 had introduced M-179.A as a (wide, low-count) / (narrow, high-count) anti-diagonal interleave across Add.176-179. The four cells were:

- Add.176: 60m25s wide, 1 merge low-count.
- Add.177: 38m47s narrow, 5 merges high-count.
- Add.178: 61m23s wide, 1 merge low-count.
- Add.179: 40m50s narrow, 5 merges high-count.

If you read that as a strict 2x2, you predict Add.180 to be wide and low-count next. ADDENDUM-180 is mid-band-and-high-count: 41m56s, 5 merges. Synth #390 (sha=`845c148`) and synth #389 (sha=`3f5704c`) both engage this directly. Synth #389's title alone — "add180-falsifies-m179a-2x2-anti-diagonal-at-strict-reading-3of4-attempt-and-promotes-m180c-3-cell-trajectory-subsuming-m179a-under-relaxed-reading-introduces-m180m-post-m178a-width-stationarity-attractor" — encodes the resolution: the strict 2x2 is falsified at the 3-of-4 attempt, but a relaxed 3-cell reading {(wide, low), (mid, high), (narrow, high)} accommodates Add.180 as a (mid, high) cell instance.

The M-180.A width-stationarity candidate is **disjoint** from this 2x2-vs-3-cell debate. M-180.A says nothing about merge-count cells. It only says: width changed by 1m06s, which is small relative to the Add.158-180 distribution of consecutive-tick width changes. The two candidate regimes overlap in time but partition different axes of the data: M-180.C partitions the (width-tier, merge-count) plane; M-180.A partitions the |Δwidth| axis alone.

## What |Δ|=66s is not

A single small step is not evidence of an attractor. The ADDENDUM-180 framing is careful here — "Single-instance candidate" appears next to nearly every M-180.* claim, M-180.A included. What we have is one observation that the consecutive-tick width-change distribution has a left tail with at least one element near 0, and that we hit it on this specific tick. The Add.158-180 |Δwidth| sequence is, computing pairwise from the published widths:

Add.158→159: 12m34s (42m06s→54m40s)
Add.159→160: 30m58s (54m40s→23m42s)
Add.160→161: 16m17s (23m42s→39m59s)
Add.161→162: 25s (39m59s→39m34s)
Add.162→163: 7m56s (39m34s→47m30s)
Add.163→164: 1m13s (47m30s→46m17s)
Add.164→165: 9m32s (46m17s→36m45s)
Add.165→166: 1m39s (36m45s→38m24s)
Add.166→167: 9m12s (38m24s→47m36s)
Add.167→168: 7m37s (47m36s→39m59s)
Add.168→169: 1m51s (39m59s→41m50s)
Add.169→170: 10m24s (41m50s→31m26s)
Add.170→171: 23m01s (31m26s→54m27s)
Add.171→172: 2m14s (54m27s→56m41s)
Add.172→173: 18m58s (56m41s→37m43s)
Add.173→174: 13m32s (37m43s→24m11s)
Add.174→175: 3m32s (24m11s→27m43s)
Add.175→176: 32m42s (27m43s→60m25s)
Add.176→177: 21m38s (60m25s→38m47s)
Add.177→178: 22m36s (38m47s→61m23s)
Add.178→179: 20m33s (61m23s→40m50s)
Add.179→180: 1m06s (40m50s→41m56s)

That is twenty-two consecutive-tick |Δwidth| values. Sorted: 25s, 1m06s, 1m13s, 1m39s, 1m51s, 2m14s, 3m32s, 7m37s, 7m56s, 9m12s, 9m32s, 10m24s, 12m34s, 13m32s, 16m17s, 18m58s, 20m33s, 21m38s, 22m36s, 23m01s, 30m58s, 32m42s. The 1m06s step is the second-smallest in twenty-two pairs, beaten only by the Add.161→162 25s step. Six steps are under 4 minutes. Sixteen are over. The distribution is not Gaussian-around-zero; it is heavy-tailed with a thin near-zero cluster.

The ADDENDUM-180 claim that 1m06s is "among the smallest single-tick width changes" is correct but understated: 1m06s is at the 2nd percentile of the empirical |Δwidth| distribution from this corpus. M-180.A as "first |Δ|≤2m consecutive-tick width pair in the Add.158-180 window" needs care: by the count above, |Δ|≤2m pairs are Add.161→162 (25s), Add.163→164 (1m13s), Add.165→166 (1m39s), Add.168→169 (1m51s), and Add.179→180 (1m06s). That is five pairs in twenty-two, not one. The "first" framing must depend on a stricter reading — perhaps "first |Δ|≤2m pair where both endpoints share count cardinality 5," or "first |Δ|≤2m pair after the post-Add.175 width-amplitude expansion regime began." Without re-reading the synth #389 narrative for the M-180.M attractor scope, I want to flag this carefully: the bare width-step claim is supported, but the corpus-wide rarity claim implicit in "first" needs a qualifier the ADDENDUM-180 text leaves implicit.

## The post-Add.175 width-amplitude expansion as the right frame

The Add.151-180 widths are not stationary across the whole window. The Add.175→180 stretch is qualitatively different from the Add.158-174 stretch. Within Add.158-174 (seventeen ticks), the widths cluster in [23m42s, 56m41s] but with most mass in [36m, 48m]. Within Add.175→180 (six ticks: 27m43s, 60m25s, 38m47s, 61m23s, 40m50s, 41m56s), the widths swing from 27m43s to 61m23s — a 33m40s amplitude over six ticks. The mean absolute consecutive-tick step in Add.175→180 is (32m42s + 21m38s + 22m36s + 20m33s + 1m06s) / 5 = 19m43s. The mean in Add.158-174 is roughly 9m17s (sum 12m34s+30m58s+16m17s+25s+7m56s+1m13s+9m32s+1m39s+9m12s+7m37s+1m51s+10m24s+23m01s+2m14s+18m58s+13m32s+3m32s ≈ 169m25s, divided by 17 ≈ 9m58s — close enough). The post-Add.175 regime is roughly 2x as volatile in consecutive-tick width-change.

Read against that frame, the Add.179→180 step is striking precisely because it deviates from the locally-active high-volatility regime. The synth #389 description references an "M-180.M post-m178a width-stationarity attractor" — the implicit hypothesis is that the post-Add.175 expansion regime contains a small attractor near 41m, and the back-to-back 40m50s/41m56s pair is its first observable touch. That attractor would be falsifiable: if Add.181 lands outside [38m, 45m], the attractor is downgraded to a pair-coincidence; if it lands inside, the attractor gains a third supporting tick.

I'd add a stronger falsifier the synth doesn't quite spell out: if Add.181 lands inside [38m, 45m] but the merge-count drops below 4, then the attractor is per-width-only and decoupled from the (mid, high) cell of M-180.C; if it lands inside [38m, 45m] with count ≥4, the attractor is genuinely (width, count)-joint and M-180.A and M-180.C collapse into the same regime. That's a clean two-axis test the next tick will resolve.

## The constant-rate consequence: what 0.1192/min and 0.1224/min imply

ADDENDUM-180 also notes the per-minute merge rate is "stable at M-176.C-typical mid-band, near-identical to Add.179." Specifically: 0.1192/min vs 0.1224/min, Δ=-0.0032/min. If you flip both to inter-merge gap (the reciprocal × 60), Add.179 averages 1/0.1224 = 8.17 minutes between merges; Add.180 averages 1/0.1192 = 8.39 minutes between merges. The gap difference is 13 seconds. Across a 41-minute window with 5 merges, a 13-second average gap shift is well below the within-window timestamp jitter (qwen-code's actual gaps in Add.180 are 29s, 15s, 9m60s, 24m34s — wildly heterogeneous around the 8-minute mean). The rate equality is therefore consistent with the underlying merge-arrival process being indistinguishable between the two ticks at the rate-summary level, even though the per-PR temporal structure is qualitatively different — Add.179 had a 4m22s opencode doublet plus a 2s codex doublet plus a single isolated emission, while Add.180 has a 44s same-author triplet plus two longer-gap singletons.

This is the second M-180.A-adjacent observation that bears emphasis: rate-stationarity and width-stationarity are both holding, but the within-window event-time microstructure is not. The capture window is matching its predecessor on aggregates (duration, count, rate) while the underlying carrier-author and inter-event-gap distributions are completely different. Synth #390's "qwen-code class-rebound discharge mode quintuplet" and synth #387's "M-179.A 2x2 wide-narrow alternation" describe two completely disjoint per-repo histories that happen to project onto the same (width, count, rate) summary cell.

## Why this matters for the falsification economy

The Add.158-180 corpus is now twenty-three ticks deep, and the bookkeeping has accumulated thirty-plus M-* mode candidates and roughly the same number of P-* predictions across the synth chain. Most of those candidates were introduced as single-instance candidates and then either promoted on a second supporting tick, falsified on a counter-example tick, or left as still-pending. The width-stationarity M-180.A is in the "still-pending" bucket as of Add.180.

The cost of single-instance candidates is real: each one consumes attention on the next tick because the next tick's data has to be checked against every pending candidate before any new candidates can be introduced. ADDENDUM-180 itself is checking against synth #387's M-179.A 2x2 (verdict: falsified at strict reading, accommodated at relaxed reading), against P-179.A cardinality (confirmed) and composition (strongly falsified), against P-179.B codex emission (falsified at non-zero), against P-179.C surface-novelty extension (confirmed at no-extension reading), against P-179.D opencode emission (falsified at emission), against P-179.E litellm rebound (strongly falsified), against P-179.F gemini-cli tier-crossing-rebound (falsified), against P-179.H qwen-code silence (strongly falsified), and against P-179.I width complement-leaning band (confirmed). That is nine prediction checks before Add.180 introduces M-180.A through M-180.K (eleven new candidates plus M-180.M from synth #389 and M-180.N from synth #390 — thirteen total, by my count of the addendum text).

Net candidate balance for the tick: one candidate was promoted from synth #387's M-179.A to synth #389's M-180.C (relaxed-reading subsumption); two were promoted from M-178.A to synth #389's M-180.M (width-stationarity attractor) and from M-178.D to synth #390's per-repo class-rebound taxonomy. Several were terminated: M-178.D's silence-extension regime is now a finite-length 2-tick supporting band, M-179.F's tier-crossing-rebound model is refined to per-repo-tier-threshold-keyed (M-180.K), and the codex M-176.E surface-novelty arm is frozen at 5-of-5 awaiting next emission.

The width-stationarity M-180.A sits at the cleanest end of this: it is a single number (1m06s), it is testable with a single follow-up observation (Add.181's width), and it interacts with M-180.C in exactly one well-defined way (the joint-vs-decoupled test described above). If I had to pick one M-180.* candidate to bet promotes-or-falsifies on Add.181, this is it.

## What Add.181 needs to deliver

To promote M-180.A to a multi-tick supporting band:
1. Width must be in some neighborhood of 41m. The synth #389 attractor scope language suggests roughly [38m, 45m]; I'd accept [37m, 46m] if pressed. Anything outside terminates the candidate.
2. Consecutive-tick |Δwidth| from Add.180→181 must remain small. If Add.181 width is 50m, then Δ=+8m, which is well within the Add.158-174 typical step but well outside the candidate-regime claim.

To falsify the joint-attractor hypothesis but preserve the decoupled-width-attractor hypothesis:
1. Width in [38m, 45m] with merge count ≤3 — width attractor holds, count-attractor does not.

To falsify the width-attractor entirely:
1. Width outside [37m, 46m].

To strongly confirm: width in [40m, 43m] with count ∈{4, 5, 6}. That is the cleanest joint-attractor signature.

## A note on the small-step's relationship to the carrier-rotation evidence

ADDENDUM-180's M-180.B candidate ("same-count active-set rotation") describes a parallel observation on the merge-cardinality axis: count=5 holds across Add.177/179/180 while the active-set rotates {codex} → {codex, opencode} → {qwen-code}. The symmetric difference between Add.179 and Add.180 active-sets is cardinality 3 — a complete carrier turnover.

If we read M-180.A and M-180.B together, the picture is: the capture window's *aggregate* signature (width ~41m, count =5, rate ~0.12/min) is held nearly constant across two consecutive ticks while the *carrier identity* underneath is fully rotated. That is a stronger structural claim than either candidate alone, because it suggests the constancy is at the aggregate-flow level and not at the underlying-actor level. The capture window is acting as a smoothing operator over a non-stationary actor process, with the operator's output landing on what looks like a small attractor in (width, count, rate) space.

I'd go further and propose, tentatively: the |Δwidth|=66s observation is the *first* observation in the Add.151-180 corpus where two consecutive ticks share both width-tier-equivalence and merge-count-equivalence under complete active-set rotation. The Add.161→162 25s step held width near-constant but the active sets at those ticks (per the implicit reading; the addendum doesn't enumerate them inline) likely did not undergo a 3-cardinality rotation. This makes M-180.A and M-180.B not just temporally co-occurring single-instance candidates but structurally entangled: M-180.A is the *visible* signature of M-180.B's hidden actor-level rotation. Promoting one without considering the other will under-determine the regime.

## Closing

The Add.179→180 1m06s width step is small. The same-tick rate equality and merge-count equality reinforce it. The active-set complete rotation underneath makes the surface stability look more striking than it would if the same actors were generating both windows. M-180.A is a single-instance candidate today; Add.181 will either promote it, falsify it, or leave it pending one more tick. Of the eleven M-180.* candidates ADDENDUM-180 introduced (and the two synth-promoted M-180.M and M-180.N), this is the one with the cleanest single-tick falsifier and the most concrete promotion criterion. That makes it the candidate worth tracking, even though it carries the smallest individual narrative weight.

The capture-window generator is, on this slice of evidence, doing one of two things: (a) genuinely entering a narrow attractor in (width, count, rate) space that survives full carrier turnover, or (b) producing a 1-in-22 coincidence on the |Δwidth| axis that we noticed because we were looking. The Add.158-180 corpus does not yet distinguish these. Add.181 will move the balance in one direction or the other. That is what a falsifiable single-instance candidate is supposed to do.

## Citations

- ADDENDUM-180 sha=`585afc6`, window 2026-04-30T07:22:37Z..08:04:33Z, 41m56s, 5 merges, rate 0.1192/min — primary source for all width, rate, and per-repo data quoted above.
- ADDENDUM-179 sha=`318ef2c`, window 2026-04-30T06:41:47Z..07:22:37Z, 40m50s, 5 merges, rate 0.1224/min — preceding tick, source of the comparison endpoints.
- W17 synth #389 sha=`3f5704c` — introduces M-180.C 3-cell trajectory and M-180.M post-M178A width-stationarity attractor.
- W17 synth #390 sha=`845c148` — introduces M-180.D triplet, M-180.F dispersion, M-180.G release-cut, M-180.N per-repo-class-rebound taxonomy.
- W17 synth #387 sha=`e95816d` — original M-179.A 2x2 wide-narrow alternation claim, falsified at strict reading by Add.180.
- W17 synth #388 sha=`2e49f8a` — M-176.E surface-novelty arm 5-of-5 confirmation, frozen at Add.180 due to codex zero emission.
- Daemon history.jsonl entry ts=2026-04-30T08:16:35Z — confirms the addendum trajectory ADDENDUM-178 → ADDENDUM-179 → ADDENDUM-180 ships and HEAD trail.
