# Source-pair co-occurrence: the openclaw+opencode doublet and the Jaccard floor

> Window: `2026-04-12T00:00:00Z → 2026-04-26T00:00:00Z` (14 UTC days). Subcommand: `pew-insights source-pair-cooccurrence`. Data: `~/.config/pew/queue.jsonl`, snapshot timestamp `2026-04-25T17:39:14.969Z`.

Most of the lenses I've been writing about for the past three days operate on **single-source** axes — `source-tenure`, `source-decay-half-life`, `source-breadth-per-day`, `provider-tenure`. They each ask the same shape of question: how does *one* CLI tool spend its hours? Co-occurrence is the first lens in this family that asks the *complementary* question, and the one that turns out to expose the most structurally interesting fact about my fleet so far: **two-thirds of my active hour-buckets contain more than one CLI tool**.

That's the headline number. Out of 442 active hour-buckets in the 14-day window, 301 of them — **68.1%** — show at least two distinct sources writing tokens into the same hour bucket. The single-tool hour is the minority case. The fleet, viewed at hourly resolution, is almost always doing two things at once.

This post takes that headline and does three things with it: (1) dissect the co-occurrence matrix to see *which* pairs do the work, (2) explain why the Jaccard column matters more than the raw count column once you have a dominant pair, and (3) surface the anti-pattern that the `--min-jaccard` flag was added to suppress.

## The raw output

Here's the table verbatim from the snapshot:

```
window: 2026-04-12T00:00:00Z -> 2026-04-26T00:00:00Z
active-buckets: 442    multi-source: 301 (68.1%)    total-pairs: 640
distinct-pairs: 14     dominant pair: openclaw + opencode (195 buckets)

source-a     source-b        count  share  jaccard
-----------  --------------  -----  -----  -------
openclaw     opencode        195    30.5%  0.534
hermes       openclaw        139    21.7%  0.370
hermes       opencode         72    11.3%  0.264
claude-code  openclaw         67    10.5%  0.162
claude-code  hermes           46     7.2%  0.209
claude-code  codex            39     6.1%  0.277
codex        openclaw         31     4.8%  0.078
codex        hermes           25     3.9%  0.132
claude-code  opencode         13     2.0%  0.044
codex        opencode          5     0.8%  0.020
claude-code  vscode-copilot    3     0.5%  0.026
codex        vscode-copilot    2     0.3%  0.030
openclaw     vscode-copilot    2     0.3%  0.005
hermes       vscode-copilot    1     0.2%  0.007
```

Six sources, fourteen distinct pairs (out of a possible C(6,2)=15; the missing pair is `claude-code + codex … wait, that's row 6). Actually all 15 possible pairs except `vscode-copilot + opencode` — the most isolated tool in the fleet didn't share a single bucket with the most heavily used one, which is itself a finding worth holding onto.

## Reading the count column: the dominant pair

The first row dominates. `openclaw + opencode` shares 195 of 442 active buckets — **44.1% of all active buckets, and 30.5% of all co-occurrence pair-events**. To put it in absolute terms: if I sample a random hour bucket from the 14-day window in which *anything* is happening, there's slightly better than a 4-in-9 chance that openclaw and opencode are both writing tokens into it.

This is consistent with what other lenses have been telling me about my workhorse tools. `provider-tenure` shows openclaw and opencode as the two longest-tenured producers. `bucket-density-percentile` puts both in the upper deciles. `source-breadth-per-day` puts both at >75% daily presence. So when the source-pair lens says "these two run together", it's not a surprise — what's interesting is *how often*.

The next two pairs by count both involve `hermes` — `hermes + openclaw` (139 buckets) and `hermes + opencode` (72 buckets). That makes hermes the third leg of the dominant triad. The fact that `hermes + openclaw` (139) is roughly twice `hermes + opencode` (72) confirms what `inter-source-handoff-latency` showed last week: hermes and openclaw share an underlying control loop, while hermes and opencode share a much weaker affinity.

## Reading the Jaccard column: why count alone misleads

The count column tells you which pair *appears together most often*. The Jaccard column tells you which pair *most reliably appears together when either appears at all*. They are different questions and they answer to different fleet pathologies.

Jaccard for a pair `{a, b}` is defined as

```
jaccard(a, b) = |buckets(a) ∩ buckets(b)| / |buckets(a) ∪ buckets(b)|
```

— the fraction of "either-active" buckets in which *both* are active. A pair with Jaccard 1.0 is perfectly coupled (whenever one runs, the other runs). A pair with Jaccard near 0.0 is essentially independent (one is active in many buckets the other isn't).

Look at the table again with that in mind. The dominant pair `openclaw + opencode` has Jaccard **0.534** — meaning slightly more than half of the buckets in which *either* tool runs contain *both*. That's the signature of two tools that are heavily used and also heavily coupled.

But notice the second-place pair by Jaccard isn't the second-place pair by count. `claude-code + codex` is sixth by count (39 buckets, 6.1% share), but its Jaccard is **0.277** — higher than `hermes + opencode`'s 0.264 (which was third by count). Why? Because `claude-code` and `codex` are both lower-volume tools, and most of their few active buckets *do* overlap. They're a small but tightly-coupled pair. The count column hid that, the Jaccard column reveals it.

Now look at the bottom of the table. `codex + opencode` has Jaccard **0.020** — co-occurring in only 5 buckets out of a union of ~250. Those two tools are functionally *independent* in my fleet: when one is running, the other almost never is. That's a useful finding too — it tells me that the workloads I drive through `codex` are temporally disjoint from the workloads I drive through `opencode`. Either I context-switch between them at multi-hour granularity, or they're scheduled by different cron jobs that don't overlap.

## The Jaccard floor and the noise pair

This is what the `--min-jaccard <n>` flag exists to suppress. Picture a fleet with one extremely heavy "background" tool (say `vscode-copilot` if I used it more) that runs in nearly every active bucket. Every other tool would then show a high *count* of co-occurrence with it — not because they're coupled, but because the background tool is everywhere by construction. The count column would be saturated with `vscode-copilot + X` rows for every other X, and the actually-coupled pairs would get pushed off the top of the table.

The Jaccard floor is the antidote. Set `--min-jaccard 0.1`, and any pair where the overlap is small relative to the union gets dropped. The remaining pairs are the ones where co-occurrence is a real signal, not an artifact of one tool being on all the time.

In my current 14-day snapshot I don't have that pathology — `vscode-copilot` is the *least* active tool, with only 6 total co-occurrences (3 with claude-code, 2 with codex, 2 with openclaw, 1 with hermes, all at Jaccard ≤ 0.030). It would be the first pair filtered out by any min-jaccard floor above ~0.04. But the flag exists because once you start slicing windows narrower than 14 days, or once one tool's volume changes by an order of magnitude, the noise pair pattern *will* show up, and at that point the count-only ranking lies.

The lesson generalizes: any concentration metric that pivots on raw counts has to be normalized by something — Jaccard, share-of-row, share-of-pair, or per-source-share — before it can claim to be a coupling metric rather than a volume metric. The `source-pair-cooccurrence` subcommand surfaces both axes side by side specifically so you can't be fooled by either alone.

## What 68.1% multi-source means operationally

The headline number — 68.1% of active buckets are multi-source — is the inverse of a number I haven't seen reported by any of my other lenses: **31.9% of active buckets are single-source**. That's the fraction of my active fleet-time when only one CLI tool is doing work. Less than a third.

This has implications for several adjacent metrics that I've been computing as if the fleet were single-source-at-a-time:

- **`bucket-handoff-frequency`** counts how often the *primary* model changes between consecutive active hour-buckets. But "primary" is only well-defined when one source dominates a bucket. In 68.1% of buckets, multiple sources are competing for primary — so the handoff count is being computed against a definition that's slightly fictional. Last week I noted the handoff frequency seemed higher than I expected; the multi-source rate may explain part of that.

- **`provider-switching-frequency`** has the same issue, one level up. If hermes (anthropic) and openclaw (mixed) are both active in the same bucket, "primary provider" is a tiebreak, not an observation.

- **`source-decay-half-life`** treats each source's tenure as an independent process. The fact that openclaw and opencode share 195 of 442 buckets means their tenures are *not* independent — they're driven by overlapping workload triggers. The half-life numbers are still descriptive, but they're not generating processes.

- **`peak-hour-share`** as a per-source metric understates contention. If the 17:00 UTC peak hour for openclaw is *also* the peak hour for opencode, the rate-limit headroom calculation needs to sum across sources, not max.

The 68.1% number is, in effect, a correction factor I've been missing. Every per-source distribution I've been publishing has an implicit "and X% of the time, this source is competing for the same bucket as another source". For the dominant pair, that X is 44%. For the median pair it's much lower. But for the fleet as a whole it's two-thirds.

## What 14 distinct pairs out of 15 possible says

Six sources can form C(6,2) = 15 unordered pairs. My snapshot shows 14 of them have at least one shared bucket. The missing pair is `vscode-copilot + opencode` — and in fact `vscode-copilot` only appears in 6 of the 640 pair-events total (less than 1%). The single most isolated pair in my fleet is between the lowest-volume tool and the highest-volume tool. They've literally never been simultaneously active in the 14-day window.

Compare that to a hypothetical pure-pinned-channel fleet, where each tool serves one workload and the workloads are scheduled disjointly. You'd expect *high* Jaccard within pinned pairs and *zero* count between unrelated pairs. My fleet does not look like that. Instead it looks like a small number of high-Jaccard cluster pairs (`openclaw+opencode`, `hermes+openclaw`, `claude-code+codex`) embedded in a generally promiscuous mixing pattern. Almost every tool eventually shares a bucket with almost every other tool — except for the one tool that almost never runs at all.

That promiscuity is itself a fleet shape. It says my workload triggers are *not* tightly partitioned by tool. The same hour that fires up openclaw is statistically likely to fire up opencode. The same hour that fires up hermes is moderately likely to fire up either of them. Only `vscode-copilot` lives off in its own scheduling regime — and the fact that it does makes it look, from this lens alone, like a tool that hasn't been integrated into the same trigger system as the rest.

## The 640 pair-events vs 442 active buckets

One last quirk worth flagging. The output reports `total-pairs: 640` and `active-buckets: 442`. Those numbers can't both be the same units — and they're not. `active-buckets` counts the number of hour-buckets in which *anything* happened. `total-pairs` counts the number of *unordered pair-events*: for each bucket containing N distinct sources, that bucket contributes C(N,2) pairs.

Working backwards: 442 buckets contain 640 pair-events, of which 0 come from the 141 single-source buckets (which contribute zero pairs). So 301 multi-source buckets contribute 640 pairs, for an average of 640/301 ≈ **2.13 pairs per multi-source bucket**. That implies the average multi-source bucket contains ≈ (1 + √(1 + 8·2.13))/2 ≈ **2.6 distinct sources** — most multi-source buckets have either 2 or 3 tools active, with 3 being common enough to pull the mean above 2.

That's consistent with my fleet reality: when openclaw and opencode are both running, hermes is often there too as a third leg. The triad is the modal multi-source bucket shape, not the doublet.

## Falsifiable predictions for next snapshot

Three things to check when this lens runs again next week:

1. **The dominant pair share will stay above 25%.** Openclaw+opencode at 30.5% is structural, not transient. If it drops below 25% in a 14-day window, it means I've materially changed my workload triggers — likely by introducing a new dominant tool or moving one of the two off the workhorse path. Falsification: if openclaw+opencode share drops below 25% without me changing anything operationally, the pairing was less stable than I thought.

2. **The `vscode-copilot + opencode` pair will remain at zero, or at most one or two.** If those two suddenly start co-occurring in 10+ buckets, something has integrated `vscode-copilot` into the trigger system that drives opencode. Falsification: if the pair shows up at >5 buckets with no operational change, the prior isolation was a sampling artifact of the 14-day window, not a fleet property.

3. **The single-source-bucket share (currently 31.9%) will drift downward over the rest of W17 and W18.** As I add more cron-driven background tools, the fraction of buckets where only one tool runs should decrease. Falsification: if the single-source share *grows* despite no tools being deactivated, the trigger system is partitioning more aggressively than the multi-tool default I expected.

The cheap version of all three checks: re-run `source-pair-cooccurrence --since 2026-04-19T00:00:00Z --until 2026-05-03T00:00:00Z` next Sunday and compare the headline numbers. If any of the three predictions miss, there's a fleet-shape change worth a follow-up post.

## Why this lens is now in my dashboard

For the past two weeks I've been treating each tool's per-bucket statistics as if the fleet were a collection of independent producers. The 68.1% multi-source rate is the empirical disproof of that assumption. The right model is a small set of *correlated* producers, with `openclaw + opencode` as the dominant correlated pair, `hermes + openclaw` as the secondary control-loop pair, and `claude-code + codex` as the small-but-tight third pair. Everything below those three pairs is either incidental overlap or noise.

The next pew-insights subcommand on my list to wire into the dashboard is `bucket-handoff-frequency`, conditioned on which pair was active in the prior bucket — because if openclaw+opencode dominated bucket N, the handoff in bucket N+1 should look very different than if a single tool dominated bucket N. Multi-source state is the hidden variable I've been integrating over without knowing it.
