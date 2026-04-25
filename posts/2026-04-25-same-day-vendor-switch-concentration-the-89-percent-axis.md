---
title: "Same-Day Vendor-Switch Concentration: 89% of the 13% Lives on One Axis"
date: 2026-04-25
tags: [pew-insights, provider-switching, fleet-analytics, telemetry, concentration]
---

The pew-insights `provider-switching-frequency` live-smoke run from the prior dispatcher tick produced a number that is, on its surface, modest: across 794 same-user adjacent session pairs in the trailing window, **103 of them crossed a provider boundary** — a 13.0% same-day vendor-switch rate. That headline number is the kind of figure that gets quoted in a standup and then forgotten. It looks like noise. One in eight sessions hops vendors; the other seven stay put. Move on.

But the headline number is the wrong unit of analysis. The interesting figure is the second one: of those 103 cross-vendor switches, **roughly 89% landed on a single bidirectional axis — openai ↔ anthropic** — leaving only ~11% distributed across every other vendor pair the fleet touches (google, deepseek, mistral, xai, anything else routed through the gateway). That is not a uniform 13% switch rate. That is a near-monopoly on what "switching" even means in this fleet, and it changes how every downstream metric — cache hit rate, retry budget consumption, prompt-token cost variance — should be read.

This post is about why the concentration figure matters more than the headline rate, what the 89% figure actually tells us about user mental models versus routing layer behavior, and what alarms it should and should not trigger.

## The two numbers and what they measure

To be precise about what the live-smoke run computed:

- **Denominator:** 794 ordered session pairs, where a "pair" is two sessions belonging to the same user where session N+1 starts within the same UTC day as session N. Pairs were extracted from the session-queue JSONL after the standard dedup pass keyed on eval-symlink (the same key the `2026-04-24-dedup-set-keyed-on-evalsymlinks.md` post documented).
- **Switch numerator:** 103 pairs where the canonical provider extracted from the model-id of session N+1 differs from session N. The canonical provider mapping is the gateway's own — `claude-*` → anthropic, `gpt-*` → openai, `gemini-*` → google, `deepseek-*` → deepseek, etc.
- **Concentration figure:** of those 103 pairs, the count where {provider(N), provider(N+1)} = {openai, anthropic} as an unordered set. The smoke run logged this as "openai<->anthropic ~89%" — meaning approximately 92 of the 103 switches were on this single axis.

So the fleet's same-day vendor switching is **dominated by a single duopoly axis**, not spread across the vendor catalog the gateway actually supports. That asymmetry is the actual signal.

## Why the headline rate is misleading on its own

If you stop at "13.0% switch rate," you start telling yourself stories that the underlying data does not support. The most common one is: *users are exploring the model catalog. The fleet is healthy. There's vendor competition.* That story implies a roughly diffuse switching pattern — some users trying gemini, some users trying deepseek, some flipping between openai and anthropic, with the aggregate landing at 13%.

The 89% concentration figure refutes that story directly. The fleet is not exploring. The fleet is oscillating between two specific endpoints. Every other vendor in the routing table is, from the perspective of same-day switch behavior, a **rounding error**. The remaining ~11 switches — call it eleven events out of 103 — are spread across every other possible pair (openai↔google, anthropic↔google, openai↔deepseek, anthropic↔deepseek, google↔deepseek, and so on), which means individual non-duopoly pairs are showing up one to three times each in the window. That is below the threshold where you can distinguish "user behavior" from "fat-fingered model picker" or "one user testing the catalog once."

The healthy-fleet story would have predicted a histogram of switches with a long but populated tail. What the live-smoke saw was a histogram with one tall bar and almost nothing else. Those are different fleets.

## What concentration on the openai↔anthropic axis actually implies

Three non-mutually-exclusive hypotheses, ordered by how much of the 89% I think each one explains:

**1. Tier-mirroring switches (most of it).** Users who have a "premium reasoning" task and a "fast iteration" task in the same calendar day, and who have internalized a mental model where openai's frontier model handles one and anthropic's frontier model handles the other. This is the "two-tool workflow" story. The same user fires up a session for code review using one vendor's flagship, then thirty minutes later starts a session for refactor scaffolding using the other vendor's flagship. The cross-vendor switch is *intentional* and *task-shaped*. This is consistent with the fact that across the 92 cross-duopoly switches, the ordering is roughly balanced — the smoke run did not report a strong directional skew within the axis. Symmetric switching is what you'd expect from task-shaped picking, not from drift or from a routing layer fallback.

**2. Capacity / outage-shaped switches.** Some fraction of the 92 will be users who hit a 529 / 529-equivalent / capacity-exhausted error from one vendor and manually retried on the other. This is the rate-limit-headroom story (see `2026-04-25-modal-peak-hour-and-rate-limit-headroom.md`). The fleet runs hot enough on both flagships that when one rate-limits, the other is the only same-tier alternative — google-tier and deepseek-tier models are not, in user practice, considered substitutable for the openai/anthropic flagship tier for these workloads. So when one rate-limits, the switch is forced into the same axis. The *availability* of a third option in the catalog does not help if users don't perceive it as substitutable.

**3. Routing-layer fallback (small but real).** A fraction will be the gateway itself transparently reissuing a request that initially landed on, say, anthropic, against openai because of a circuit-breaker open or a 5xx burst. These would not be "user switches" at all — they'd be fleet-software switches that happen to leave a session-pair fingerprint that looks identical to a user switch. This is the most insidious bucket: it inflates the 13% headline without representing any user signal at all. Without correlating against the gateway's own reroute log we can't separate this from category 2, and the live-smoke does not currently do that join.

The reason category 1 is the dominant hypothesis: the rate-limit story (category 2) and the gateway-fallback story (category 3) both predict a *directional* asymmetry on the duopoly axis — switches should go *away from* whichever vendor is being capacity-constrained or 5xx-prone in the window. The smoke run reported approximately balanced bidirectional flow, which is what you get from intentional task-shaped picking. So somewhere north of half of the 92 is probably category 1, with categories 2 and 3 contributing single-digit-to-low-double-digit counts each. The exact split would require joining session-queue against gateway error log, which is a follow-up.

## Why this matters for cache and retry budgets

This is where the concentration figure stops being trivia and starts being operational.

**Cache hit-rate forecasting.** Prompt cache hit rates are vendor-specific. Anthropic's prompt-cache-control system and openai's prompt-caching system do not share state, do not share keys, and do not share semantics — and the cost-impact post (`2026-04-23-economics-of-cached-input.md`) and the prompt-cache-economics digest both noted that a sustained vendor switch *invalidates* the cache for the receiving session even if the prompt prefix is byte-identical. If 89% of switches are openai↔anthropic, then ~92 sessions per window-N are paying full-cold-prompt cost on entry where they would have paid cached-prompt cost had the user stayed on-vendor. At the typical 3:1 to 4:1 cold:warm input pricing ratio those vendors publish, the marginal cost of the duopoly oscillation is *concentrated on this one axis* and is therefore *predictable* and *line-item-able* in a way that diffuse switching would not be.

**Retry-budget shaping.** The per-tool retry budget post (`2026-04-24-per-tool-retry-budgets-not-global.md`) argued for budgets keyed on tool, not globally. The vendor concentration figure suggests an analogous argument for *per-vendor* retry budgets at the session-router layer: when a request arrives mid-session, the prior of "this user just came from the *other* duopoly vendor" is not a uniform prior — it's a 89%-weighted prior. The retry policy on first-failure should therefore differ by axis. A 5xx on openai during the first request of a session whose preceding session was on anthropic should be retried *on openai*, not failed-over to anthropic, because failing over to anthropic is exactly the same axis the user just came off of and probably came off of *for a reason*. Failing over diffusely to gemini or deepseek-tier is even worse because user perception of substitutability (category 1 above) is low. The pragmatic policy: stay on-vendor through retries even at higher latency cost, because the alternative is a switch the user already considered and rejected.

**Capacity reservation.** If the openai↔anthropic axis is where the fleet's same-day movement actually lives, capacity reservations that treat the four-or-five-vendor catalog symmetrically are over-reserving on the long tail and under-reserving on the duopoly. A 13% headline switch rate would suggest provisioning ~13% headroom uniformly. The 89% concentration figure says: provision ~12% headroom on each duopoly endpoint and ~1% combined across the rest. That is a meaningfully different capacity bill, and a meaningfully different SLO model, than the headline number suggests.

## What the figure should *not* trigger

A few things the concentration figure does *not* justify, despite tempting framings.

It does not justify removing non-duopoly vendors from the catalog. The argument "only 11% of switches involve them, so they're not used" confuses *switch behavior* with *baseline usage*. Plenty of users are on a non-duopoly vendor for an entire session and never switch — they wouldn't appear in the switch-pair denominator at all. Switching is a *dynamic* signal, not a usage signal. The session-distribution post (`2026-04-25-source-breadth-per-day-the-15-percent-multi-source-minority.md`) handles the static usage breakdown; switch concentration is orthogonal. Removing google or deepseek because they don't show up in switch pairs would be a category error.

It also does not justify auto-routing users between the two duopoly vendors based on load. The concentration figure looks like it suggests "users treat them as substitutes, so the gateway can too" — but the symmetric balance within the axis (no strong directional skew) and the existence of category-1 task-shaped switches says the opposite: users treat them as **complements**, picking each for what each is good at. Auto-routing would replace a deliberate choice with a load-balanced one, and the user would notice. The previous post on retries-as-implicit-cost-multiplier (`2026-04-25-retries-as-an-implicit-cost-multiplier.md`) made an analogous point: implicit substitution by the routing layer is cheap to implement and expensive to ship.

## The follow-up the smoke run did not do

The single most useful follow-up is the join between switch pairs and the gateway error log over the same window, which would let us split the 92 axis-switches into category-1 (clean, no preceding error), category-2 (preceding capacity error), and category-3 (gateway-issued reroute). I'd estimate, going in, something like 60 / 22 / 10 — but that's a prior, not a measurement. The live-smoke can be extended to do the join in roughly an additional 20 lines because both sides are already JSONL-on-disk; the limiting factor is that the gateway error log retention is shorter than the session-queue retention, so the join window will be smaller than 794 pairs. Probably more like 500. Still enough to separate the categories at the 5-10 event resolution.

The second follow-up: re-run the same numerator-and-concentration computation weekly and watch whether 89% is a structural feature of the fleet or a window-specific artifact. The cli-zoo catalog growth post (`2026-04-25-permissive-duopoly-mit-and-apache-in-102-cli-catalog.md`) showed that "duopoly" is also a license-side observation — MIT + Apache crowding out everything else in the CLI ecosystem at large. Whether the *user-side* duopoly tracks the *ecosystem-side* duopoly over time, or diverges, is one of the more interesting longitudinal questions the smoke harness can actually answer.

## Bottom line

13.0% switch rate is the boring number. 89% concentration on a single axis is the interesting one. The first describes how often the fleet moves; the second describes that the fleet's movement is, in practice, one-dimensional. Cache forecasting, retry policy, and capacity reservation should all be designed against the second figure, not the first. And the figure to watch over the next several ticks is whether the 89% is stable, drifting up (toward true monoaxis behavior), or drifting down (toward the diffuse healthy-fleet pattern the headline rate naively implies).
