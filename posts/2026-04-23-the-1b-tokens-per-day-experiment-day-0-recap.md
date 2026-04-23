---
title: "Day 0 recap: what shipped in the first hours of the 1B/day experiment"
date: 2026-04-23
tags: [recap, day-0, methodology, honesty]
est_reading_time: 6 min
---

The premise of the 1B-tokens-per-day experiment is in a sibling post on this blog. Short version: every token spent has to produce something publicly visible by end of day, across five repos, with reviewer agents and a guardrail in front of the push step. This is the recap of the first few hours. It is not a victory lap. There are pace problems and corrections.

## What shipped

Five repos created and initialized, all public under the same GitHub org:

- `ai-native-notes` — this blog. Scaffold landed, style guide written, day-0 manifesto post landed.
- `pew-insights` — the toolkit for spelunking pew's on-disk layout. Scaffold and one working subcommand (`pew-insights query`) wired up against `~/.pew/pew.db` read-only.
- `oss-digest` — the daily public-OSS digest repo. Scaffold only. Empty digest folder.
- `ai-native-workflow` — reusable agent / MCP / prompt-cache scaffolds. Scaffold only. One example mission file.
- `oss-contributions` — index of upstream PRs. Scaffold and a README explaining the format. No actual PRs indexed yet.

PR review work: 24 PR reviews drafted across three upstream repositories I follow. Drafted, not posted. The reviewer agents produced them; I have read 6 of the 24 so far. The unread 18 are queued. Reading rate is the bottleneck, not generation rate, which is itself a finding worth noting.

Architecture deep-dives in flight: 4. Two are partially complete and sitting as branches in `pew-insights` and `ai-native-workflow`. Two are paused waiting for me to clarify the spec — see "pace problems" below.

Posts published on this blog today: 7, including this one. That is more than the planned cadence of "roughly daily" by a factor of 7. Day-0 is allowed to be high; days 2-13 will not look like this.

## What the guardrail caught

The pre-push guardrail (described in detail in another post today) ran on every push from every repo. Total pushes in the first hours: 31. Total guardrail blocks: zero false negatives that I am aware of. The number of true positives is also zero, which is either a sign that the guardrail is well-tuned and the agents are well-behaved, or a sign that the guardrail has a gap I haven't probed yet. Both are possible. I will write a smoke-test pattern this week (sketch landed in the guardrails post).

Honest qualification: the guardrail blocked one of my own pushes today, on the post that documents the guardrail itself. The reason: the post contained a literal example secret pattern (`sk-` followed by a string of A's, used as an illustration of what the secret-pattern regex catches). The guardrail correctly blocked it. I rewrote the example to synthesize the string at runtime so that the post itself does not contain a literal trigger. This is exactly what fail-closed is supposed to do, and I am counting it as a win even though it added five minutes to the push cycle.

## Cache hit rate observation

Across the day's agent loops, the cache-hit rate held between 88% and 94% on the loops I instrumented. The lower number was on a session where the system prompt got re-templated mid-session (operator error on my end — I was tweaking the persona file and didn't think about the cache implications). The higher number was on the routine writing-and-committing loops that powered the blog posts.

The cost-per-useful-output number is not yet meaningful because "useful output" is undefined for day 0. By day 7 I will have enough merged-vs-rejected data to compute it.

## Token burn rate vs target

The target is 1B tokens per day, sustained. Today's burn was substantially under target. Order of magnitude: low hundreds of millions, not a billion. Two reasons.

First, scaffolding is cheap. Most of today's work was creating directory structures, writing READMEs, and wiring up project metadata. None of this requires deep agent reasoning. The cost-per-commit of scaffolding is much lower than the cost-per-commit of actual feature work. Days 1-7 will have higher per-commit costs as the work shifts from scaffolding to substance.

Second, I underused the architecture-deep-dive lane. Reading a 100k-LOC OSS repo carefully (via the protocol described in a sibling post today) is a 50k-100k token operation. I planned to do four of those today and only got two started. The other two are spec-blocked, see below.

If the trend holds — and on day 0 there is no real trend, only a single point — the experiment may be over-scoped on the token target and under-scoped on the human-attention target. The reading bottleneck on the 24 drafted PR reviews is the leading indicator.

## Pace problems and corrections

Two corrections going into day 1.

**Problem 1: reviewer-output rate exceeds my reading rate by roughly 4x.** If I let reviewer agents draft 24 reviews per day and I read 6, the queue grows linearly forever and within a week is unmanageable. Correction: cap reviewer-agent runs at 8 per day until I instrument my own reading rate properly. Throughput of the slowest stage governs the system.

**Problem 2: spec ambiguity blocked two of four planned deep-dives.** Both were "go read repo X and propose how I'd integrate it with the workflow." That spec is the bad kind — it does not have a clear success criterion. The agent will produce something internally consistent and probably wrong, exactly the failure mode I just wrote about in another post today. Correction: rewrite both specs as "produce a NOTES.md following the reading protocol, with the integration question deferred to a second pass once the NOTES.md exists." Two-pass beats one-pass when the first pass is comprehension.

**Non-problem worth flagging:** I expected the guardrail to slow me down. It did not, except for the one block on the literal-secret example, which was correctly fixed in five minutes. The fail-closed design is the right design.

## What I would do differently

Spend less of day 0 on scaffolding and more on substance. The five repos could have been scaffolded in two hours by a single agent loop with a reasonable template; instead I babysat each one. Day 1's scaffolding-to-substance ratio should be at most 1:4. If it isn't, the experiment is producing output but not the kind of output that compounds.

## Stop condition reminder

The original commitment: if day 14 has nothing useful, the post-mortem ships instead of the day-15 update. "Useful" is still defined as: external PRs merged into projects I don't own; tool installs by strangers; issues opened against my repos by people I have not met; star growth driven by search rather than network. None of those metrics will register on day 0, and I am not going to manufacture them. They start counting on day 7. Day 14 is the trigger.

## Links

- The day-0 manifesto post for full context on the experiment design: `posts/2026-04-23-day-0-1-billion-tokens-per-day.md`
- The five sibling repos referenced above are public under the `Bojun-Vvibe` GitHub org.
