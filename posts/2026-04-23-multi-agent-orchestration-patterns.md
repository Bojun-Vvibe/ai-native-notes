---
title: "Three patterns for orchestrating multiple coding agents"
date: 2026-04-23
tags: [orchestration, multi-agent, spec-kitty, patterns]
est_reading_time: 9 min
---

A single coding agent working alone produces code at roughly the rate a junior engineer does: pretty fast, with a steady stream of small mistakes that need a second pair of eyes. Two coding agents working alone produce two streams of small mistakes. The interesting question is what *structure* you put between them so that one stream catches the other's mistakes instead of compounding them.

I have spent the last few months running variants of three patterns. Each works for a specific class of task. Each has an anti-pattern it slides into when you misuse it. Below: when to reach for each, when not to, and concrete pseudo-YAML for what a mission file looks like.

## Pattern 1: Implement-review loop

One agent writes a diff. A second agent reviews the diff, in isolation, with no access to the first agent's chain of thought. The reviewer either approves or returns structured rejection feedback. On rejection, the implementer revises. Loop until approval, with a hard cap on iterations.

This is the workhorse pattern. It catches the boring-but-fatal mistakes: tests that pass for the wrong reason, hallucinated function names, comments that contradict code, stub implementations marked as complete. The reviewer agent does not need to be smarter than the implementer; it needs to be *uncorrelated*. Different model families help. A Claude implementer with a GPT reviewer (or vice versa) catches more than two Claudes in series.

A mission file in roughly the shape that spec-kitty understands:

```yaml
mission: implement-review-loop
work_package: WP-014
spec: specs/WP-014-add-pagination.md

steps:
  - id: implement
    agent: claude
    role: implementer
    prompt_template: prompts/implement.md
    inputs:
      - spec
      - repo_context
    outputs:
      - diff
      - rationale
    on_success: review

  - id: review
    agent: codex
    role: reviewer
    prompt_template: prompts/review.md
    inputs:
      - spec
      - diff               # NOT rationale — reviewer judges the artifact, not the story
    outputs:
      - decision           # one of: approve | revise
      - feedback           # structured: list of {file, line, issue, severity}
    on_decision_approve: done
    on_decision_revise: implement

caps:
  max_iterations: 4        # if not approved by 4, escalate to arbiter
  on_cap_exceeded: arbiter
```

When to use it: any task where "did this actually work" is verifiable from the diff alone. Bug fixes with a regression test. Feature additions with acceptance criteria. Refactors with type-checking and tests as the spec.

When not to use it: tasks where the spec itself is fuzzy. The reviewer cannot judge correctness against an unclear spec; it will either rubber-stamp everything or reject everything. Symptom: review iterations oscillate between approve and revise on alternating turns. Fix the spec, not the loop.

Anti-pattern: giving the reviewer the implementer's chain-of-thought. The reviewer starts confirming the implementer's framing instead of judging the artifact. Decision quality drops to roughly the implementer's quality. Strip the rationale; show only the diff.

## Pattern 2: Scout-then-act

A cheap, fast agent (the scout) explores the repo, the issue tracker, and any relevant docs, and produces a short structured brief — files to touch, functions to call, edge cases to handle, prior PRs to reference. A more expensive agent (the actor) then implements against the brief, never repeating the exploration.

This is the pattern that wins on token cost when the task involves large codebases. Exploration is cheap if you use a fast model with low per-token cost; implementation needs the better model. Doing both with the better model is wasteful by a factor of 5-10x for the exploration portion.

```yaml
mission: scout-then-act
work_package: WP-027
issue: https://github.com/example/repo/issues/4419

steps:
  - id: scout
    agent: aider               # use the cheap fast model here
    role: scout
    model_hint: small
    prompt_template: prompts/scout.md
    inputs:
      - issue
      - repo_root
    tool_budget:
      file_reads: 30
      grep_calls: 20
      web_fetches: 0           # scout does not browse; it reads the repo
    outputs:
      - brief                  # ~500 tokens, structured

  - id: act
    agent: claude
    role: implementer
    model_hint: large
    prompt_template: prompts/act.md
    inputs:
      - issue
      - brief                  # NOT repo_root — the brief is the repo summary
    outputs:
      - diff

  - id: verify
    agent: claude
    role: tester
    inputs:
      - diff
    runs:
      - cmd: pytest -x -q
      - cmd: ruff check .
    on_failure: act            # bounce back with test output appended to brief
```

When to use it: any task in a repo larger than the actor's effective context window can productively hold. Anything where you'd otherwise let the actor "look around" using grep tools — that exploration is the expensive part.

When not to use it: tiny repos where the entire codebase fits in 20k tokens and the actor can read it all directly. The scout step is overhead with no payoff.

Anti-pattern: letting the scout produce free-form prose. The brief has to be structured (file list, function list, constraints, prior art). Free-form briefs become "I think this is interesting and you should look at it" — useless. Force a JSON or strict markdown schema on the scout output.

Second anti-pattern: re-running the scout on every iteration of an inner implement-review loop. The brief is stable; cache it for the duration of the work package.

## Pattern 3: Arbiter escalation

When the implement-review loop fails to converge — typically four iterations with no approval — a third agent (the arbiter) reads the spec, the latest diff, and the full review history, and makes a binding decision: approve, reject with a definitive fix, or kick the work package back to specification.

The arbiter is not a third reviewer. It is a tie-breaker with authority to change the rules. Its decisions do not loop further. If the arbiter approves, the diff lands. If the arbiter rejects with a fix, the implementer applies that fix verbatim and the diff lands. If the arbiter kicks back to spec, the work package is paused and a human is paged.

```yaml
mission: arbiter-escalation
triggered_by: implement-review-loop.cap_exceeded

inputs:
  - spec
  - latest_diff
  - review_history       # all N rounds of feedback + responses

steps:
  - id: arbitrate
    agent: opencode      # use the model with the highest reasoning bar available
    role: arbiter
    prompt_template: prompts/arbiter.md
    outputs:
      - decision         # one of: approve | reject_with_fix | kick_to_spec
      - fix_diff         # only if decision == reject_with_fix
      - rationale

  - id: apply
    when: arbitrate.decision == 'reject_with_fix'
    agent: claude
    role: implementer
    prompt_template: prompts/apply_arbiter_fix.md
    inputs:
      - latest_diff
      - arbitrate.fix_diff
    outputs:
      - diff
    on_success: done     # no further review; arbiter has decided

  - id: page_human
    when: arbitrate.decision == 'kick_to_spec'
    notify: bojun
    block_until: human_response
```

When to use it: when you genuinely have an implement-review loop in production and you're seeing it cap out more than once a day. Below that frequency, just let the loop fail and look at it manually.

When not to use it: as a default for every task. Three-agent missions cost three times as much per task and only pay off when the disagreement is real. If your loop almost always converges in 1-2 iterations, the arbiter sits idle and adds latency to your pipeline metrics.

Anti-pattern: making the arbiter a different *prompt* for the same model that did the review. You haven't added a tie-breaker; you've added a re-roll. The arbiter must be uncorrelated with both the implementer and the reviewer — different model family if at all possible, definitely different prompt and definitely different context (it sees the history; the others didn't).

Second anti-pattern: letting the arbiter's `rationale` flow back into the implementer's context for the next work package. The arbiter is a one-shot oracle, not a teacher. Its rationale is for the human reviewing pipeline outputs, not for the agents.

## What ties them together

All three patterns are state machines with explicit transitions. None of them are "agents talking to each other in free chat." Free-chat multi-agent setups are the thing that looks impressive in demos and produces nothing useful in practice. Structure beats chat: typed inputs, typed outputs, named transitions, hard caps.

The mission file is the contract. If you can't write the mission file in pseudo-YAML before launching the run, the orchestration is not yet a pattern — it's a vibe. The vibe might work once. It will not work twice.

## What I would do differently

I started with the implement-review loop and stayed there too long. Adding scout-then-act for large-repo work cut my token spend on those tasks by roughly 60% with no quality regression I could measure. Adding the arbiter took longer to justify and is still rare in my workflow — maybe 2% of work packages escalate. The lesson is to add patterns by demand, not by ambition. Start with the loop; add scout when context windows hurt; add the arbiter when the loop is genuinely stuck.

## Links

- spec-kitty repository: https://github.com/spec-kitty/spec-kitty (mission file format reference)
- Anthropic on multi-agent research: https://www.anthropic.com/engineering/built-multi-agent-research-system
- aider docs on architect/editor mode (related pattern): https://aider.chat/docs/usage/modes.html
