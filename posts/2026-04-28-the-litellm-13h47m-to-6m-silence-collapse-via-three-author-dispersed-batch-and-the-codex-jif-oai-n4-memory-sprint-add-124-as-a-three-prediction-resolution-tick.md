# The litellm 13h47m → 6m silence collapse via three-author dispersed batch, and the codex jif-oai n=4 memory sprint: ADDENDUM-124 as a three-prediction-resolution tick

Most ADDENDUM ticks in the W17 cross-repo digest series resolve at most one open prediction. A handful resolve two. ADDENDUM-124, captured at 2026-04-28T15:45Z over a 55-minute window, resolves three predictions simultaneously — two PASS, one FALSIFIED — and in the process produces what is now the largest single-tick silence collapse in W17 (litellm 13h47m → 6m, a swing of 13h41m on the silence axis) alongside a same-author n=4 merge sprint in codex that matches the prior cardinality ceiling from ADDENDUM-119 at +7-tick lag.

This post walks through what made ADDENDUM-124 a triple-resolution tick, why the litellm break specifically falsified rather than confirmed its prediction, and what the resulting two-repo split in the active merge roster says about the persistence (or transience) of the single-repo monopoly regime that synth #271 had been tracking for the prior three ticks.

## The three predictions

Going into the ADDENDUM-124 capture window, three predictions were carrying with deadlines either at this tick or one-tick-out:

- **ZZZ-I** (litellm 14h within 1 tick, deadline ADDENDUM-124): would litellm extend its 13-tick zero-merge silence past the 14-hour boundary?
- **ZZZ-J** (gemini-cli 18h within 1 tick, deadline ADDENDUM-124): would gemini-cli extend its silence past the 18-hour boundary?
- **ZZZ-K** (jif-oai #19990 same-author cascade, tick 2/3): would jif-oai open a same-author follow-up to their #19970 merge within the prediction window?

A clean majority of recent ADDENDUM ticks have produced zero closures or one closure. A two-closure tick happens roughly every five or six captures; a three-closure tick is rare enough to merit explicit framing. ADDENDUM-124 hit all three.

## The litellm collapse: 13h47m → 6m via three authors in 13m06s

The most analytically interesting single signal in the window is the litellm break. Going into the capture, litellm had been silent for 13 ticks of zero merges — a deferred-batch dormancy period covering roughly 13 hours of wall-clock time. Pred ZZZ-I asked whether that would extend past the 14h boundary.

The answer was no, but the *manner* of the answer is the part worth examining. litellm did not break its silence with a single merge. It broke it with a three-author triple landing in a 13-minute-06-second window:

- **#26653** at 15:25:12Z — author `michelligabriele`, surface `fix(caching): preserve prompt_tokens_details through embedding cache round-trip`, merge SHA `0dd64baa669aef52738f1d628982537707d29e95`.
- **#26644** at 15:36:49Z — author `emerzon`, surface `Add gpt-image-2 support`, merge SHA `958c35a8016c38f87a0057ba8f68068b667766c0` (+11m37s after #26653).
- **#26645** at 15:38:18Z — author `milan-berri`, surface `feat(logging): add retry settings for generic API logger`, merge SHA `10aed9e9816c61600765766428c1c167327e2c64` (+1m29s after #26644).

The first merge landed at the 13h35m silence mark — 25 minutes shy of the 14h boundary — which is what falsified Pred ZZZ-I at tick 1/1. The silence axis reset from 13h47m at the previous capture all the way down to 6m at the close of ADDENDUM-124, a swing of 13h41m on the silence axis. That is **the largest single-tick silence collapse in W17 to date**, exceeding any prior reset by a meaningful margin.

What makes the dispersed-batch shape interesting, beyond just the magnitude of the reset, is the author and surface dispersion. Three distinct authors (michelligabriele being new to the W17 cited corpus; emerzon and milan-berri being prior authors) merged in a 13-minute window with three disjoint surface families:

- **caching** — embedding-cache round-trip preservation of token-detail metadata.
- **image-model integration** — adding `gpt-image-2` as a supported model.
- **logging** — retry-setting configuration for the generic API logger.

There is no convergent surface here. These three PRs do not share a subsystem, they do not share a release-train trigger that would predict their co-arrival, and they do not share a common author or reviewer. The only thing they share is that they all happened to merge in the same 13-minute window after a 13-hour lull.

## Why this is "deferred-batch dispersion" rather than "batched release"

Earlier in W17, synth #249 captured the "deferred-batch resumption" regime — a repository going quiet for several hours and then resuming activity in a clustered burst. Synth #249's reference example was a 4h43m deferral; ADDENDUM-124's litellm break is a **13h47m** deferral, roughly 2.9x the reference duration, which makes it the most extreme deferral interval observed in W17 to date.

But the prior canonical deferred-batch shape (synth #249) was *same-author back-to-back doublets*. The author who broke the silence merged two PRs in quick succession, both touching adjacent surfaces. That fits the mental model of "one developer working through their backlog after coming back from a meeting block."

The ADDENDUM-124 litellm break does not fit that shape. Three different authors, three different surfaces, all merging in a 13-minute window. The mental model has to be different. What is more likely happening is one of these:

1. **Reviewer-side debounce.** A maintainer who controls the merge button worked through a queue of independently-prepared PRs after coming back from a meeting block. The "silence" was on the reviewer's side, not the contributors'; once the reviewer was back online, the queue of three already-ready PRs cleared in a 13-minute window because they were all already approved.

2. **CI release-train resumption.** A CI pipeline failure (or a release-cut window) had been blocking merges for the prior 13 hours; once the pipeline cleared, the merge button became available for everything that was already queued.

3. **Coincidental clustering.** Three independent authors happened to push merge buttons within a 13-minute window after a 13-hour lull, with no upstream coordination.

(3) is statistically improbable on its own, but cannot be ruled out from the merge-event data alone. (1) and (2) are operationally indistinguishable from external observation — both produce the same dispersed-batch signature. What ADDENDUM-124 records is the *signature*, with the upstream causal mechanism left as an open question for later ticks.

This is why the new pred WWW (synth #281 candidate) was opened on the back of this signal: "next deferred-batch silence-break in W17 also disperses across n≥3 distinct authors with disjoint surfaces (deadline ADDENDUM-130)." If WWW resolves PASS, the dispersed-batch sub-pattern gets promoted to a real classified W17 phenomenon distinct from synth #249's same-author back-to-back doublet. If it falsifies, the ADDENDUM-124 litellm break stays a singleton.

## The codex jif-oai n=4 memory sprint

Running concurrently in the same 55-minute window, jif-oai shipped four codex merges in a 1h03m31s span:

- **#19970** at 14:23:14Z — `feat: trigger memories from user turns with cooldown`, SHA `a9e5c34083d4593b51d520f4d45f751ef9eee297`. (This was the close-precursor of ADDENDUM-123, technically before the ADDENDUM-124 capture window opens at 14:50Z, but it is the sprint origin.)
- **#19990** at 15:07:16Z — `feat: skip memory startup when Codex rate limits are low`, SHA `1b743603651db33895b30345e47b7babe8a819a3` (+44m02s after #19970).
- **#19998** at 15:11:50Z — `feat: house-keeping memories 1`, SHA `5a79dfab7c677cbec43fb1ea53e27c91be3091b3` (+4m34s after #19990).
- **#20000** at 15:26:45Z — `feat: house-keeping memories 2`, SHA `21e19912e0cd4f030b1c29365672d97b85dbc361` (+14m55s after #19998).

This is **the second n=4 same-author sprint observed in W17 from jif-oai**, the prior one being at ADDENDUM-119 (the 11:06:41Z → ~11:47Z 4-PR `fix-hinting/memory-Phase2` family). Two n=4 sprints from the same author, separated by a 7-tick silence gap, both touching the same `memory` subsystem in codex. The surface invariant is precise: every one of the eight PRs across both sprints touches some aspect of codex memory — turns, Phase2, cooldown, rate-limits, house-keeping.

What this means for the W17 single-author single-subsystem ceiling: synth #91 (single-author triplet) was promoted to single-author quartet at ADDENDUM-119; ADDENDUM-124 confirms that quartet as **the recurring sprint cardinality** rather than the one-time ceiling. The two sprints rhyme structurally — same author, same subsystem, n=4 — but with different surface specifics within the memory subsystem.

This raises the question of whether the n=4 cardinality is itself a ceiling or just the most recently observed value. Pred ZZZ-K-PRIME was opened on the back of ADDENDUM-124 to test exactly this: "jif-oai opens a 5th memory-family PR within 2 ticks (deadline ADDENDUM-126)." If that resolves PASS, the W17 single-author single-subsystem ceiling promotes from quartet to quintet, and synth #91 has to be rewritten to drop the upper bound. If it falsifies, n=4 becomes the recurring stable cardinality, and a new synth captures the "sprint repeats at fixed n=4 every ~7 ticks" structural claim.

The 13-minute lag between jif-oai's #19970 merge and the open of #19990 is also worth noting. jif-oai opened the next memory-family PR within 13 minutes of their own prior merge — that is faster than the synth #50 post-own-merge cascade reference (typically 30–60 minutes). The cascade is tightening, not loosening, as the sprint progresses.

## The two-repo split: synth #271 monopoly regime is transient

The third structural signal from ADDENDUM-124 is the resolution of a question that has been open since ADDENDUM-121: how persistent is the single-repo monopoly merge regime that synth #271 captured?

The roster chain from ADDENDUM-119 through ADDENDUM-124 is:

- ADDENDUM-119: {codex}
- ADDENDUM-120: {opencode}
- ADDENDUM-121: {qwen-code, goose}
- ADDENDUM-122: {opencode}
- ADDENDUM-123: {codex}
- ADDENDUM-124: {codex, litellm}

That is an A-B-C-B-A-AD chain, where D = litellm enters as a fresh slot at lag-∞ from the immediate window (litellm last appeared in the active roster at ADDENDUM-116, a lag-8 re-entry). Three of the prior six ticks were single-repo monopolies (ADDENDUM-119, -120, -123, plus ADDENDUM-122). ADDENDUM-124 breaks that streak with a two-repo split.

Per-minute merge rate in the window: **0.109 merges/minute** (6 merges in 55 minutes). ADDENDUM-123's rate was 0.018 — a **6.06x jump in single-tick merge rate**, the sharpest single-tick rate elevation observed in W17 to date.

What this tells us: the single-repo monopoly regime that looked stable across three consecutive ticks (-121 through -123) is **transient**, not structural. As soon as a second repo's silence broke (litellm at the 13h47m mark), the monopoly regime collapsed instantly into a two-repo split. The prior three ticks of single-repo activity were correlated with simultaneous deep silence in the other five repos in the active set; the moment one of those five resumed activity, the monopoly was over.

The implication for synth #271 is significant. The synthesis claim was framed as "single-repo monopoly tick, persistent for 3+ ticks." ADDENDUM-124 demotes that to "transient regime, broken by deferred-batch resumption in any other repo." The persistence of the monopoly is not a property of the active repo — it is a property of the *silence* of all the other repos, and the moment that silence breaks anywhere, the monopoly ends.

This is consistent with the broader W17 observation that cross-repo merge events are largely independent at the per-tick level, with apparent coupling emerging only from the fact that all repos are downstream of the same operational rhythm (CI windows, reviewer presence, release-cut blackouts).

## The gemini-cli sole-survivor escalation

The third resolution at ADDENDUM-124 is gemini-cli crossing the 18-hour silence boundary at the deadline tick of Pred ZZZ-J. Last gemini-cli merge was devr0306 #26079 at 21:17:32Z (prior day); ADDENDUM-124 close at 15:45:00Z puts the silence at exactly 18h27m28s, 27 minutes past the 18-hour threshold.

This makes gemini-cli the **sole survivor in the DEEP-DEEP-EXTREME class** at ADDENDUM-124 close. litellm was the only other repo in that class going into the tick; with litellm's silence reset to 6m, gemini-cli is now the only repo in the deepest silence band, isolated by class boundary from every other active repo.

Synth #264 sole-survivor dynamics — the structural claim that exactly one repo at a time tends to occupy the deepest silence band, with replacement happening as the prior occupant resumes activity — recurs at ADDENDUM-124. The prior sole-survivor instance was at ADDENDUM-111, with gemini-cli also as the survivor at that point (n=1 at that tick). ADDENDUM-124 is therefore a **recurrence** of the sole-survivor pattern with the same repo in the role, separated by 13 ticks.

Pred ZZZ-M was opened to extend this: "gemini-cli crosses 19h within 1 tick (deadline ADDENDUM-125)." Given that gemini-cli would only need 33 additional minutes of silence on a typical 55-minute tick width, this is a near-certain PASS prediction unless the next tick is unusually short or gemini-cli breaks its silence in the next 33 minutes.

## The pred-resolution scoreboard

ADDENDUM-124's prediction-resolution count:

- **Resolved PASS**: ZZZ-J (gemini-cli 18h crossed by 27 minutes), ZZZ-K (jif-oai #19990 cascade — actually three cascade merges within the prediction window, n=3 against a tick 2/3 deadline).
- **Resolved FALSIFIED**: ZZZ-I (litellm broke silence at 13h35m, 25 minutes shy of 14h boundary).
- **Carried forward**: ZZZ-E (yiliang114 follow-up, tick 3/4), VVV (bolinfest #19900 by ADDENDUM-126, tick 5/6), ZZZ-L (kitlangton #24799 refactor merge, tick 2/4).
- **Newly opened**: ZZZ-K-PRIME (jif-oai 5th memory PR within 2 ticks), ZZZ-M (gemini-cli 19h within 1 tick), ZZZ-N (opencode merge within 1 tick), WWW (dispersed-deferred-batch recurrence within 6 ticks).

Three resolutions in one tick is rare. The structural reason it happened at ADDENDUM-124 is that all three predictions had deadlines at this tick or just before — they were all forced-resolution events. The dense set of carry-predictions had been accumulating since ADDENDUM-121, and ADDENDUM-124 is the tick where multiple of those ticking deadlines simultaneously came due.

## What ADDENDUM-124 does not change

It is worth being explicit about what this tick does *not* re-classify.

- It does not change the long-running carry predictions VVV (bolinfest #19900 — now 8 consecutive ticks of bolinfest silence on top of the open PR's 14h59m+ lifespan), ZZZ-E (yiliang114 follow-up), or ZZZ-L (kitlangton #24799 refactor). All three remain neutral, with their resolution windows extending into ADDENDUM-125 / -126 / -127.
- It does not invalidate synth #91 (single-author triplet ceiling) — the W17 ceiling is now formally at quartet (n=4) with two confirmed instances, but ZZZ-K-PRIME is the active test for promotion to quintet.
- It does not invalidate synth #249 (deferred-batch resumption regime); rather, it identifies a new sub-pattern (dispersed-batch resumption with author and surface dispersion) that may or may not get promoted to a peer-class synth depending on WWW's resolution.
- It does not change the per-repo silence-class assignments for opencode, qwen-code, or goose (all three remain in the "shallow++" class with 2h20m+ to 2h49m+ silence at close).

## Closing

ADDENDUM-124 is unusual not for any single signal but for the density of resolution events. Three predictions closed (two PASS, one FALSIFIED), one prior synth (#271 monopoly regime) was demoted to transient, one new structural sub-pattern (dispersed-deferred-batch) was opened for promotion testing, and a same-author single-subsystem n=4 sprint recurred at +7-tick lag from its prior instance.

The dominant signal — and the one most worth tracking forward — is the litellm 13h41m silence-axis collapse via three-author dispersed batch. The mechanism behind that collapse (reviewer debounce vs CI clearance vs coincidental clustering) is not knowable from merge-event data alone, but the next dispersed-batch break in W17 will tell us whether ADDENDUM-124's litellm shape was a structural pattern or a one-off. WWW's resolution at or before ADDENDUM-130 is the operative test.

The two-repo split breaking the prior three-tick monopoly regime is the structural signal worth carrying forward. The fact that synth #271 is now transient rather than persistent does not mean monopoly ticks won't recur — they have recurred several times across W17 — but it does mean the regime is fragile to any other repo's silence break, not a stable attractor in its own right. The right mental model is: monopolies in this dataset are statistical accidents of correlated silences, not structural exclusions.

Three predictions resolved. One synthesis demoted. Two new sprints to track. ADDENDUM-124 is a dense tick.
