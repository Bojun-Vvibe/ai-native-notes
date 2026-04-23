---
title: "Five tasks where coding agents make things worse"
date: 2026-04-23
tags: [agents, anti-patterns, when-not-to-use, opinion]
est_reading_time: 7 min
---

I run coding agents most working hours of most working days. I am as committed to the bet as anyone. The honest version of the bet is that agents are excellent at a specific class of task, mediocre at most, and actively destructive at a few. Knowing which is which is the difference between using them and being used by them.

Below: five tasks where reaching for an agent is the wrong move, and why. Each is something I have lost time to personally, more than once, before I learned to type the code myself or pick up the phone.

## 1. Pure UI tweaks where you'd be faster typing

Move a button. Change a color. Swap two columns in a table. The whole change is 30 seconds of typing and you can see whether it worked by glancing at the page.

The agent loop for this kind of task: describe the change in English (15 seconds), wait for the model to read the file (10 seconds), wait for the diff to come back (15 seconds), realize the diff missed the right CSS class because there are three places this style is defined (5 seconds), correct the prompt (15 seconds), wait again (25 seconds), apply, refresh, see it didn't work because the agent edited the wrong file. Now you're 90 seconds in, you've burned a few thousand tokens, and you still haven't seen the change.

The break-even point for using an agent is somewhere around "the change requires touching files I don't have memorized." For UI work in a codebase you know, that bar is rarely cleared. Open the file, type the change, save. The agent has nothing to add and adds latency to a task whose whole point was its triviality.

Counter-example: large mechanical UI refactors (renaming a design-token across 200 files) are perfect agent work. The distinction is whether the bottleneck is *deciding what to change* or *typing the change*. Agents reduce the typing cost, not the deciding cost.

## 2. Debugging a heisenbug that requires the scientific method

A bug that disappears when you add logging. A bug that only fires under a specific load. A bug that's reproducible in production and never in dev. The diagnostic loop here is: form a hypothesis, design an experiment that would distinguish it from alternatives, run the experiment, update beliefs.

Agents are bad at this. Specifically, they are bad at *not running an experiment that they already know the answer to.* They will read the code, identify three plausible causes, and immediately propose a fix for the most plausible one. If you push back, they will propose a fix for the second one. They are reasoning forward from "what could cause this" to "what to change," skipping the middle step of "what evidence would distinguish these causes."

The result is a sequence of plausible-looking patches, none of which fix the bug, each of which slightly muddies the code so that the next investigator (probably you, in two weeks) has more confounding variables to deal with.

What works for heisenbugs: a debugger, a careful reading of the actual logs, and a notebook. The agent can help you write the instrumentation once you've decided what to instrument. It cannot help you decide what to instrument.

## 3. Anything where the spec is genuinely unclear

This is the most expensive category and the most tempting one to use an agent for, because "the spec is unclear" feels like exactly the kind of grunt-work-of-thinking you'd want to delegate. It is not. It is the kind of work where the agent's failure mode is maximally damaging.

When the spec is clear, an agent that misreads it produces obviously wrong code that you reject. When the spec is unclear, the agent will pick *one consistent interpretation*, write coherent confident code against that interpretation, and present it as if it were the only possible reading. You read the diff, it looks reasonable, you merge it. Three weeks later in production, behavior diverges from intent because the interpretation the agent picked was not the one anyone wanted, and now there are 30 commits on top of it.

The honest move when a spec is unclear is to stop coding and go talk to whoever owns the spec. The agent will not do this for you. It will silently assume.

A useful test: before launching the agent, write down in one sentence what the success criterion is. If you can't, the spec is unclear. Don't launch.

## 4. Tasks dominated by waiting

A CI pipeline takes 18 minutes to run. A deploy takes 25 minutes. A schema migration takes an hour. The thing you're doing is mostly waiting and occasionally reacting to a failure.

The temptation is to have an agent "watch" the CI and fix any failures that come up. In practice, what happens: the CI fails for a reason that requires reading a 400-line log carefully and noticing the one line that says "test database disk full." The agent reads the log, identifies the failing test by name, proposes a "fix" to that test, pushes it, the CI fails again, this time on a different test that was using the same disk. You now have an agent that is rapidly making the situation worse and burning tokens to do it.

Wait-dominated tasks have two properties that wreck agents: low feedback frequency and high cost-per-mistake. The agent's strength — fast iteration on a tight loop — is exactly absent. The right human move during a long CI run is to look at something else for 18 minutes and check the result with full attention. The agent's tendency is to keep poking, which is the wrong tendency for the situation.

A specific anti-pattern I've watched several times: "agent restarts the deploy when it fails." A deploy failure is information. Restarting it without understanding why it failed is how you end up with three half-deployed environments and a rollback you can't run.

## 5. Security-critical code review

This one I have the strongest feelings about. Agents are confidently wrong about security in a way that is dangerous out of proportion to their general accuracy.

A general code reviewer who's not a security specialist will hedge: "I don't fully understand the threat model here, but this looks suspicious." That hedge is correct and useful. An agent will tell you with full confidence that "this code is safe because input is validated upstream," when actually it isn't, because the agent has no way to verify what happens upstream and is pattern-matching on the fact that validation usually does happen upstream.

The confidence-vs-correctness gap is the killer. For most kinds of review, the agent is wrong sometimes and you can catch it. For security review, the agent is wrong in a small fraction of cases and you cannot catch the wrongness without doing the review yourself, in which case you didn't need the agent.

Concrete cases I've seen agents miss: SQL injection through a column name (validated for value, not for identifier); auth bypass through a "convenience" admin override flag left in a debug branch; a race condition in a token-refresh path that leaks the old token under load; a permissions check that runs on the wrong identity object after a recent refactor moved the field. In each case the agent was asked to review the diff, said "looks good," and a human reviewer caught it later.

For security-critical paths: use the agent to *find* code that should be reviewed (it's good at "list every place that constructs a SQL string"). Do not use it to *do* the review.

## The pattern across all five

Each of these tasks has the same shape: low feedback frequency, high cost per wrong action, or a spec that the agent has to invent. Agents excel at the inverse — high feedback frequency, low cost per wrong action, clear spec. That is most of what mid-level coding work looks like, which is why they are so useful most of the time. It is not all of what work looks like.

The discipline I try to apply: before launching an agent on a task, ask whether each of the three failure conditions applies. If any do, type the code yourself.

## What I would do differently

I learned all five of these by ignoring them, more than once each. The most expensive one was #3 — the unclear spec. Twice in the last three months I have shipped agent-generated code that was internally consistent and externally wrong, and discovered the gap weeks later. The fix is depressingly simple: when launching an agent on a feature, write the success criterion as one sentence first. If the sentence won't form, walk away from the keyboard.

## Links

- Hillel Wayne on the inadequacy of LLMs for ambiguous specs: https://www.hillelwayne.com (search his blog for "ambiguity")
- "How to Read Logs" by Brian Kernighan-style discipline: any good debugger talk; Andreas Zeller's book *Why Programs Fail* is the canonical reference.
