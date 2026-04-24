# AI-Native Notes

Technical notes from running AI-native dev workflows daily — terminal coding agents, LLM proxies, MCP servers, prompt engineering at scale, multi-agent orchestration, and prompt-cache economics. Written from real usage, not from reading docs.

## Index

### 2026-04-24

- [State machine vs free-form ReAct: a design choice every agent loop is making whether it admits it or not](posts/2026-04-24-state-machine-vs-react-as-an-agent-loop-design-choice.md)
- [State machine vs free-form ReAct: the architecture choice every agent CLI quietly makes](posts/2026-04-24-state-machine-vs-react-loop-as-an-architecture-choice.md)
- [Context window budgeting for long-running agents](posts/2026-04-24-context-window-budgeting-for-long-running-agents.md)
- [Why your AI agent should write to JSONL, not SQLite](posts/2026-04-24-why-your-ai-agent-should-write-to-jsonl-not-sqlite.md)
- [EWMA in logit space, not raw space, for bounded ratio metrics](posts/2026-04-24-ewma-in-logit-space-for-bounded-ratios.md)
- [Four bug shapes that keep showing up in agent infrastructure](posts/2026-04-24-four-bug-shapes-from-this-weeks-pr-reviews.md)
- [Picking a baseline scorer for agent metrics: z-score vs MAD vs EWMA](posts/2026-04-24-picking-a-baseline-scorer-zscore-mad-ewma.md)

### 2026-04-23

- [Day 0: Why I'm putting one billion tokens per day through this account](posts/2026-04-23-day-0-1-billion-tokens-per-day.md)
- [The economics of prompt caching: why your token costs are not what you think](posts/2026-04-23-prompt-cache-economics.md)
- [What I learned reverse-engineering the pew CLI on-disk layout](posts/2026-04-23-pew-internals-tour.md)
- [Three patterns for orchestrating multiple coding agents](posts/2026-04-23-multi-agent-orchestration-patterns.md)
- [A pre-push guardrail for AI-generated commits](posts/2026-04-23-guardrails-for-ai-pushed-commits.md)
- [A reading protocol for understanding a 100k-LOC OSS repo with a coding agent in 30 minutes](posts/2026-04-23-reading-an-oss-repo-with-an-agent.md)
- [Five tasks where coding agents make things worse](posts/2026-04-23-when-to-not-use-an-agent.md)
- [Architecture deep-dive: openai/codex](posts/2026-04-23-architecture-openai-codex.md)
- [Architecture deep-dive: Aider-AI/aider](posts/2026-04-23-architecture-aider-ai-aider.md)
- [Architecture deep-dive: OpenHands/OpenHands](posts/2026-04-23-architecture-openhands-openhands.md)
- [Architecture deep-dive: BerriAI/litellm](posts/2026-04-23-architecture-berriai-litellm.md)
- [Day 0 recap: what shipped in the first hours of the 1B/day experiment](posts/2026-04-23-the-1b-tokens-per-day-experiment-day-0-recap.md)

## Cadence

Roughly daily. Some days will be terse. If a post would be filler, it doesn't ship.

## License

- Posts (prose, under `posts/`): [CC-BY-4.0](LICENSE)
- Code samples (any fenced code block, any file under a future `code/` directory): [MIT](LICENSE)

See `LICENSE` for the full text of both.

## About the author

Independent developer working on AI-native tooling. Daily driver: terminal coding agents (opencode, claude code, codex), an LLM proxy (litellm), MCP servers, and orchestration on top of spec-kitty. These notes are the by-product.

## Contributing

See [meta/CONTRIBUTING.md](meta/CONTRIBUTING.md). Style notes for the author live in [meta/STYLE.md](meta/STYLE.md).
