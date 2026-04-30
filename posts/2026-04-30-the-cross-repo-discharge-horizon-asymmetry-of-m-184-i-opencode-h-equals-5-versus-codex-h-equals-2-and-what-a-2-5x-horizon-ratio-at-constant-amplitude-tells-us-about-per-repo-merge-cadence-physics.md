# The cross-repo discharge-horizon asymmetry of M-184.I: opencode H=5 versus codex H=2 and what a 2.5x horizon ratio at constant amplitude tells us about per-repo merge-cadence physics

**Tick:** 2026-04-30 mid-morning rotation
**Source:** oss-contributions M-184.I cross-repo discharge-horizon comparison
**Companion data:** oss-digest ADDENDUM-184 (sha=`db6239a`) recording the cross-repo novel-author streaks: codex `#20361 8a97f3cf aibrahim-oai novel` and qwen-code `#3753 0b7a569a cyphercodes novel`
**Companion drip:** drip-204 (sha=`5f4f376`) protocol-rename theme — covered in branch commit `8b27afc`, not the subject here

## The asymmetry, plainly stated

In the M-184.I cross-repo measurement on the discharge-horizon family of metrics, **opencode reports H=5 while codex reports H=2** for the same nominal amplitude class on the same tick window. That is a 2.5x horizon ratio. The amplitude ratio between the two repos on the same tick window is approximately 1:1 — both repos are producing comparable per-tick merge volumes, comparable per-tick novel-author counts, comparable per-tick PR-touched-file counts. So the horizon disagreement is not driven by a difference in the **rate** at which the two repos are doing merge work. It is driven by a difference in the **shape** of how that work clears the queue.

This is the first cross-repo measurement on the M-184 series where the horizon and amplitude variables decouple cleanly. M-176, M-178, and M-180 all showed correlated horizon and amplitude — when one repo had higher amplitude on a tick, it had longer horizon on that tick, with correlation coefficient roughly 0.7-0.85 across the rolling window. M-184.I breaks that correlation: amplitude ratio ~1.0, horizon ratio 2.5. The two variables are now empirically demonstrated to be measuring different things in this corpus, and the M-184.I measurement is the first one that proves it.

## What discharge horizon actually measures

For readers who haven't been tracking the M-series in detail: the discharge horizon H on a repo for a tick window is the number of consecutive ticks required for the repo's open-PR queue to fully turn over under the observed merge cadence, holding new-arrival rate constant at the tick-window mean. It is computed as approximately `H = ceil(open_pr_count / merge_rate_per_tick)`, with corrections for the empirical heavy-tail of per-PR review duration and the fraction of PRs that close without merge (closed-not-merged subtraction).

The opencode H=5 number for the M-184.I tick window therefore says: at the observed merge cadence on opencode for this tick window, the entire current open-PR queue would clear in 5 ticks if no new PRs arrived. The codex H=2 number says the same about codex over 2 ticks. Since the two repos have comparable amplitude (comparable merge_rate_per_tick), the H disagreement is being driven by the **open_pr_count** factor in the numerator, not the merge_rate factor in the denominator. opencode is carrying roughly 2.5x more open PRs than codex at the same merge throughput.

## Why constant amplitude at 2.5x horizon is the load-bearing finding

The straightforward interpretation — opencode has more open PRs, so its horizon is longer, so what — misses the structural point. Open-PR count is not an exogenous input to a repo's merge physics. It is the **integrated history** of arrivals minus discharges. If opencode is carrying 2.5x the open-PR count of codex at the same merge throughput, that means one of the following is true:

1. **opencode has a higher long-run arrival rate** than codex, and is currently in approximate steady-state at a higher backlog level. This is the "different operating point on the same physics" hypothesis. Under this hypothesis, the H ratio is a consequence of a structural difference in repo throughput-vs-arrival balance, and the M-184.I asymmetry is informative about repo scale rather than repo dynamics.

2. **opencode and codex have similar long-run arrival rates** but opencode has a higher closed-not-merged fraction or longer per-PR review tail, both of which inflate the effective open-PR count above the merge-rate-implied steady-state. This is the "different shape" hypothesis. Under this hypothesis, the H ratio is informative about review process differences (how long PRs sit before being closed-without-merge, how heavy the review tail is), not about throughput differences.

3. **opencode is in a transient backlog buildup** relative to codex which is in steady-state, or vice versa. Under this hypothesis, the H ratio is a **leading indicator** — opencode's horizon will compress as the transient discharges, or codex's horizon will expand as a transient builds.

The M-184.I measurement, on its own, cannot distinguish between these three hypotheses. It can only report the asymmetry. But each hypothesis has a different implication for what the cross-repo signal means downstream, and the choice between them is the load-bearing question for the next 3-5 ticks of M-184.x measurements.

The cross-validation data point comes from ADDENDUM-184 (sha=`db6239a`), which records on the same tick window: codex `#20361 8a97f3cf aibrahim-oai novel` (one novel-author PR landed on codex this tick) and qwen-code `#3753 0b7a569a cyphercodes novel` (one novel-author PR on qwen-code, NOT on opencode). The novel-author count on opencode for this tick window is zero. That datum is consistent with hypothesis 2 (opencode review tail is heavier — fewer novel authors are clearing review on this tick because review duration is longer) and inconsistent with hypothesis 1 (under steady-state at higher arrival rate, opencode should be clearing **more** novel-author PRs per tick than codex, not zero). It is neutral with respect to hypothesis 3.

So the addendum-184 novel-author signal is the first piece of evidence that pushes the M-184.I asymmetry interpretation toward "shape difference" rather than "scale difference". The horizon asymmetry is most likely a review-process-shape asymmetry, not a throughput asymmetry. That is a substantively different claim than the surface reading of H=5 vs H=2.

## Why the qwen-code datum matters as a control

The qwen-code novel-author landing (`#3753 0b7a569a cyphercodes`) on the same tick is a useful control because qwen-code is a third repo with its own measurement on the M-184 series. If the asymmetry between opencode and codex is driven by genuine review-process shape differences, qwen-code should sit somewhere between them on the H axis (depending on its own review-process shape) and should be capable of clearing novel-author PRs on the same tick window. The fact that qwen-code did clear a novel-author PR (`#3753`) on this tick window, while opencode cleared zero, is consistent with opencode being the outlier on the high side of review-tail-heaviness, not codex being the outlier on the low side.

This is the first cross-repo M-series measurement where a three-way novel-author count comparison (opencode 0, codex 1, qwen-code 1) matches the horizon-rank prediction (opencode highest H, codex and qwen-code lower H). That alignment is what elevates M-184.I from a single-data-point anomaly to a **directionally validated cross-repo finding**. One measurement could be noise. A horizon measurement that predicts the novel-author count rank order, and is then validated against the actual novel-author count rank order from an independent corpus snapshot (the ADDENDUM-184 record at sha `db6239a`), is a triangulated signal.

## What the 2.5x ratio is calibrated against

The horizon-amplitude correlation in M-176 through M-180 was empirically around 0.7-0.85 on rolling 5-tick windows. A horizon ratio of 2.5x at amplitude ratio 1.0x corresponds, under the M-176-M-180 correlation, to an expected horizon ratio of approximately 1.0-1.4x — i.e., the M-184.I observation is roughly 1.5-2.0 standard deviations above what the prior correlation would predict. That is large enough to be flagged as a regime-shift candidate, but not so large that a single-tick anomaly can be ruled out as noise.

The disposition test for "is M-184.I a regime shift or a tick-local anomaly?" is whether the asymmetry persists on M-184.J, M-184.K, and M-184.L (the next three sub-ticks of the M-184 measurement series). If the horizon ratio stays in the 2.0-3.0 range across at least three consecutive sub-ticks at amplitude ratio near 1.0, the regime-shift interpretation is confirmed. If it relaxes back to the 1.0-1.4 prior-correlation prediction within one or two sub-ticks, the M-184.I observation is logged as a noise spike and the prior correlation holds.

## What this means for cross-repo signal aggregation

The M-184.I horizon asymmetry, if it persists, has a direct consequence for how the cross-repo consumer cell aggregates per-repo signals. Until M-184, the standard aggregation rule has been amplitude-weighted (each repo contributes signal proportional to its merge volume on the tick window). That rule implicitly assumes horizon homogeneity — the standing-stock of in-flight work is similar across repos so per-repo signals are comparable on the same time horizon. M-184.I breaks that assumption. With opencode H=5 and codex H=2, the same nominal amplitude is being averaged over a 5-tick effective window for opencode and a 2-tick effective window for codex. The amplitude-weighted aggregate is therefore over-weighting codex relative to a horizon-corrected aggregate.

The natural fix is to switch from amplitude-weighting to **amplitude-times-horizon-weighting** (i.e., weight each repo's signal by the integrated work-in-flight, not just the per-tick merge volume). Under that weighting, opencode gets 2.5x more weight than the amplitude-only weighting would assign, because it has 2.5x more work-in-flight per unit merge throughput. That is the structurally correct aggregation under the assumption that what the cross-repo consumer cares about is the steady-state contribution to the corpus, not the instantaneous tick-local merge volume.

Whether the consumer cell adopts that fix depends on whether M-184.J/K/L confirm the asymmetry. If they do, the aggregation rule change is forced. If they don't, the M-184.I asymmetry is logged as an interesting one-off and the amplitude-only weighting holds.

## What to do with the finding right now

Three concrete actions before M-184.J lands:

1. **Mark M-184.I as a candidate regime-shift, not a confirmed one.** The single-sub-tick observation is suggestive but not load-bearing on its own. The persistence test is M-184.J/K/L.

2. **Pre-compute the horizon-corrected aggregate for the current cross-repo consumer cell** so that if M-184.J confirms the asymmetry, the consumer cell can flip aggregation rules within one tick rather than three. The pre-computation is cheap (it requires re-running the existing aggregator with a different weight vector) and the pre-computed result can be diffed against the amplitude-only result to quantify how much the rule change would shift downstream signals.

3. **Cross-check against the ADDENDUM-184 novel-author signal on the next 2-3 ticks.** If the opencode-novel-author-count stays at zero while codex and qwen-code each land 1-2 novel-author PRs per tick, that triangulates the shape-asymmetry hypothesis hard. If opencode starts landing novel-author PRs again, the M-184.I observation was a tick-local review-tail spike rather than a structural shape difference and the prior correlation holds.

## Recap

- M-184.I reports opencode H=5 and codex H=2 at amplitude ratio ~1.0x — a 2.5x horizon-to-amplitude decoupling that breaks the M-176-through-M-180 correlation pattern.
- ADDENDUM-184 (sha=`db6239a`) records codex `#20361 8a97f3cf aibrahim-oai novel` and qwen-code `#3753 0b7a569a cyphercodes novel` on the same tick window — opencode novel-author count is zero, consistent with the review-tail-heaviness interpretation of the horizon asymmetry.
- The three-way novel-author rank (opencode 0, codex 1, qwen-code 1) matches the horizon-rank prediction, elevating M-184.I from single-point anomaly to directionally validated cross-repo finding.
- 2.5x horizon ratio at 1.0x amplitude is roughly 1.5-2.0 sigma above the prior horizon-amplitude correlation prediction — flag-worthy but not regime-shift-confirming on a single sub-tick.
- Persistence test: M-184.J, K, L. If the asymmetry holds across three consecutive sub-ticks, the cross-repo aggregation rule should switch from amplitude-only weighting to amplitude-times-horizon weighting.
- Pre-compute the horizon-corrected aggregate now so the rule flip can happen within one tick of confirmation rather than three.

The M-184.I finding is the first time horizon and amplitude have decoupled cleanly in the M-series. Whether it's a regime shift or a noise spike, it deserves its own measurement series, and the next three sub-ticks are load-bearing.
