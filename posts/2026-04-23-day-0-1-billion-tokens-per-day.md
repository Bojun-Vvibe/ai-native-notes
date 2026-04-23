---
title: "Day 0: Why I'm putting one billion tokens per day through this account"
date: 2026-04-23
tags: [meta, methodology, economics, multi-agent]
est_reading_time: 5 min
---

Most AI usage stats are vanity. "I burned 800M tokens this week" is the new "I wrote 10k lines of code" — sounds impressive, tells you nothing about whether anything useful happened. Tokens can be wasted on the same refactor loop seventeen times because the agent forgot what it did on attempt sixteen.

Token count without context is meaningless. But token count *with a forcing function* is something else. The forcing function on this account: every token spent has to produce something publicly visible by end of day. No private branches, no "I'll clean it up later." That constraint changes the shape of the problem.

## The setup

I'm running five public repos in parallel. Each one takes a slice of the daily token budget and is responsible for a different output type:

- a tool repo (working CLI, with tests)
- an OSS-PR repo (tracking patches I send to other people's projects)
- a notes repo (this one — technical write-ups)
- a digest repo (daily summary of public OSS activity I find worth flagging)
- a workflow-template repo (reusable scaffolds — agent configs, MCP server setups, prompt-cache layouts)

Each repo has its own definition of a valid commit. The notes repo wants prose with at least one concrete example. The tool repo wants either a feature with a test or a bug fix with a regression test. The OSS-PR repo wants a link to a PR I actually opened upstream, not a fork I never pushed. If a repo gets nothing valid by end of day, it gets nothing — no padding commits.

## The mechanism

The work is done by a small fleet of agents, not by me typing. The split:

- **Coding agents** (terminal-native — opencode, claude code, codex, depending on which one is least flaky that day) do the implementation. They get scoped tasks: "write the function, write the test, make the test pass."
- **Reviewer agents** read every diff before I see it. Their job is to catch the slop — hallucinated APIs, tests that assert nothing, comments that contradict the code, "TODO: implement" left behind, or the classic "added try/except: pass to make the test green." Roughly a third of diffs get bounced back at this stage.
- **A guardrail script** runs on every staged change before it can be committed. It greps for an explicit denylist (employer names, internal hostnames, codenames, secret patterns, paths under work directories) and refuses to let the commit proceed if it finds a hit. This is the boring, unglamorous piece, and it's the most important one. Multi-agent setups leak context if you don't clamp the output side.

Orchestration on top of all this is spec-kitty. MCP servers handle the side-channel reads (issue trackers, calendars, doc stores) so the agents don't paste private content into prompts by accident.

## The economics

"One billion tokens per day" sounds expensive. It is much less expensive than people assume, for one reason: prompt-cache hit rates above 90% are achievable once your workflow is stable. The agent re-reads the same repo layout, the same style guide, the same toolchain definitions, hundreds of times per day. Cached input tokens cost a fraction of novel ones — order of magnitude, in some pricing tiers.

The actual cost driver is *novel input* — long unfamiliar files pulled into context for the first time, large diffs from upstream, long search results. Output tokens matter too, but a well-scoped agent emits little per task: a function, a test, a commit message. The headline number is dominated by cache hits on boilerplate context, replayed across hundreds of small tasks.

I'll publish the actual cost-per-useful-commit number once I have two weeks of data.

## The honest part

This is an experiment. If it produces noise, I kill it. The KPIs that matter are not "tokens spent" or "commits pushed." They are:

1. External PRs merged into projects I don't own.
2. Tool downloads (or installs, or stars from search-not-from-network — anything that indicates a stranger found the work and used it).
3. Issues opened against my repos by people I've never interacted with.
4. Search-driven star growth — i.e., not "my friends starred it" but "someone Googled a problem and landed here."

Token count is the input. None of the four metrics above are downstream of token count in a linear way. You can spend zero tokens and get a merged PR. You can spend a billion and get nothing. The interesting question is whether the multi-agent pipeline produces enough leverage to make the input/output ratio favorable in a way that wasn't possible eighteen months ago.

I will publish the daily report. If on day 14 nothing useful has shipped, I'll write the post-mortem instead of the day-15 update.

## Links

The four sibling repos this one references (placeholders, will be live as they ship):

- `Bojun-Vvibe/pew-insights` — the tool repo
- `Bojun-Vvibe/oss-digest` — the daily public-OSS digest
- `Bojun-Vvibe/ai-native-workflow` — reusable agent / MCP / prompt-cache scaffolds
- `Bojun-Vvibe/oss-contributions` — index of upstream PRs
