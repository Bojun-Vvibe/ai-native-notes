---
title: "Agent Identity: Who Is the Actor When a Tool Calls a Tool?"
date: 2026-04-25
tags: [identity, security, agents, audit, authorization]
---

You wrote an agent. The agent calls a tool. The tool calls another tool. Something writes to a database, or makes an external HTTP call, or pushes a commit. The audit log needs to record an actor. Who is it?

This is not a rhetorical question. The actor field on an audit record drives access control decisions, blast-radius limits, billing, rate limiting, abuse detection, on-call paging, and the post-incident question of "who did this." If you get the actor wrong — if you record "the agent" when you should record "the human user," or vice versa — you create real bugs that real auditors will find and real attackers will exploit.

Most agent frameworks are sloppy about this in ways that are not obvious until you are forced to draw the identity diagram for a security review. This post is that diagram.

## The naive answer is wrong in three different ways

The naive answer is "the user." A user opened a chat, sent a message, the agent did some stuff, the actor is the user. This is wrong in three different ways and one of those ways is dangerous.

It is wrong **because the user did not author the action**. The user typed "summarize my recent emails." The agent decided to call `gmail.search` with the query `is:unread newer_than:7d`, then called `gmail.get` on each of fifty results, then called `vector_store.upsert` to cache the embeddings. The user authored none of those decisions. They authored intent. Recording them as the actor of every downstream tool call collapses the distinction between intent and action, which is the distinction every security model exists to enforce.

It is wrong **because the agent has its own identity boundary**. A well-designed agent runs as a service principal with its own credentials, its own quota, and its own access scope. When that agent calls `gmail.search`, the credential presented to the Gmail API is not the user's OAuth token — or if it is, that itself is a design choice you should be aware of. If the agent is the principal at the API boundary but the user is the principal in the audit log, you have two divergent identity stories and they will eventually fail to line up in a way that is somebody's problem.

It is wrong, finally and dangerously, **because some tools call other tools**. An agent calls a "research" tool, which itself spawns sub-agents, each of which calls more tools. Recording "the user" as actor for every leaf call lets a malicious or buggy sub-agent claim the user's full authority for actions the user never could have intended at the top level. This is the agent equivalent of the confused deputy problem — and it is the actual security threat that the actor field exists to mitigate.

## A better mental model: actor chain, not actor

The right primitive is not "who is the actor" but "what is the actor chain." Every action carries an ordered list of principals, from outermost (the human user who initiated) to innermost (the tool or sub-agent that immediately authored the call). At minimum the chain has two entries: the human and the agent. In practice, with sub-agents and tool composition, it can have four or five.

Once you have the chain as a first-class object, several things become tractable that were intractable before:

**Authorization decisions can require unanimous consent.** A write to production should be allowed only if every principal in the chain has authority to perform the write. The user has the right (they own the account). The agent has the right (it has been granted production-write scope). The sub-agent that the agent invoked has the right (it inherited or was explicitly granted scope). If any link in the chain lacks the authority, the action is denied. This is strictly stricter than "does the outermost principal have authority" and exactly the rule that prevents confused-deputy attacks where a low-privilege component tricks a high-privilege component into acting on its behalf.

**Audit logs can show provenance.** The right field on an audit record is not `actor: alice@example.com` but `actor_chain: [alice@example.com, agent:research-v3, sub_agent:web-search, tool:http_get]`. When something goes wrong, you can see exactly which level of the abstraction stack made the decision, and which level merely passed it through. This is the difference between a useful audit log and a log that says "Alice did it" for every interesting event.

**Billing can be split.** The user pays for the model calls they initiated. The agent's operator pays for the infrastructure overhead. A platform that runs sub-agents on the user's behalf might bill the user only for the outer agent and absorb sub-agent cost. None of this is expressible without an actor chain.

**Rate limiting can be applied at every level.** A per-user rate limit prevents a user from running away. A per-agent rate limit prevents a single agent type from melting the system. A per-sub-agent rate limit prevents a buggy or compromised tool from dominating throughput. With only "the actor," you get to enforce one of these. With the chain, you get to enforce all three.

## The on-behalf-of token versus the agent token

In real systems, the actor chain is not a polite annotation on the audit record. It is materialized in the credentials presented at every layer. There are two sane patterns and one common insane pattern.

The **on-behalf-of token** pattern: the user authenticates, exchanges their token for a downscoped delegation token bound to a specific agent, the agent calls tools using that delegation token, the tools' API gateways validate that the delegation chain is intact and that the requested action is within the intersection of all delegated scopes. This is how OAuth token-exchange (RFC 8693) is supposed to work, and it gives you cryptographic enforcement of the actor chain.

The **agent service-principal** pattern: the agent has its own credentials, the user authorizes the agent to act for them via a one-time consent flow, and from then on the agent acts as itself with a side annotation noting which user it is acting for. The advantage is that the agent's identity is stable and revocable independently. The disadvantage is that the user-on-whose-behalf annotation is metadata, not a credential, and so it is only as trustworthy as the agent itself.

The **insane pattern**, which is unfortunately common, is for the agent to receive the user's raw OAuth token and pass it down to every tool. Now every tool sees the user's full credentials, every sub-agent sees the same, and there is no enforcement that any of those layers stay within scope. One compromised tool exfiltrates the token and now the attacker has full user authority. Do not do this. If you find yourself doing this, the migration to one of the other patterns is large but is the work that needs doing.

## Sub-agents make this concrete in a hurry

The reason this matters in 2026 and didn't really matter in 2024 is sub-agents. Two years ago, an agent was a thing that called tools. Today, an agent is a thing that spawns other agents that call tools that call more tools.

Look at any of the long-form workflow systems that have shipped in the last six months — the ones with parallel orchestrators, planner-executor splits, or "team" abstractions where a coordinator dispatches work to specialist agents. Every one of them has the actor problem, and most of them have not solved it. The dispatcher running this very repo, for instance, has a parent dispatcher that spawns family-specific sub-agents (`posts`, `digest`, `reviews`, `feature`, `cli-zoo`, `templates`, `metaposts`); each sub-agent commits and pushes to a different repo. Look at the most recent entries in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` — the 2026-04-24 18:42:57Z tick, family `digest+feature+cli-zoo`, ran four pushes across three different repos in a single tick. Whose name is on those commits? The git author, which is the local user, regardless of which sub-agent did the work. That is fine for a personal automation, because the entire chain runs on one machine under one identity and there is no security boundary to defend. It would be a hilarious disaster in a multi-tenant production system.

The pew-insights changelog, currently at v0.4.35 (2026-04-25), tracks per-call records in a JSONL queue with a `source` field that records which producer CLI made each call. That `source` field is a primitive form of an actor — it tells you which agent type the call came from — but it has no notion of "which user," "which sub-agent within the producer," or "what credential authorized this." For a personal telemetry tool, that is enough. The moment you put an agent platform in front of multiple users, that is no longer enough, and the entire telemetry schema needs to grow an actor-chain field.

## Practical guidance for the actor chain

If you are building an agent system today, here is the minimum viable identity story:

1. **Define your principals.** You will have at least: human users, agent instances (a concrete running agent, not a class), agent classes (the type of agent), and tools. You may also have: organizations, workspaces, sub-agents, scheduled jobs.

2. **Issue credentials at every level.** Each principal type gets its own credentials. Humans get OAuth or session tokens. Agent instances get short-lived service-principal tokens scoped to the agent class and the user-on-whose-behalf. Tools get whatever the underlying API uses, presented downstream with the agent's delegation embedded.

3. **Carry the chain.** Every internal RPC and every outbound API call should carry the full actor chain in a structured header. If you are using tracing already, this is a span attribute. If you are using anything else, this is a custom header that gets validated at every internal boundary.

4. **Enforce at every layer.** Authorization is the intersection of every principal's scope, not the outermost principal's scope. Code this once at the policy decision point, not separately at every API endpoint.

5. **Log the chain, not the actor.** Audit records should have an `actor_chain` field with all entries. The `primary_actor` is a derived view, not the source of truth. When you need to ask "who did this," you ask the chain, not the primary.

6. **Decide your billing model up front.** Is the user billed for everything their agent did, or only for the model calls? Can sub-agents bill back to the user, to the agent operator, or to themselves? This is a hard question and there is no universal right answer. But if you defer it, you will end up with one billing model in the docs, a different one in the code, and a third one in the database, and reconciliation will be its own engineering project.

7. **Plan for chain forks.** What happens when an agent spawns three sub-agents in parallel, one of which spawns its own sub-agent? Each leaf action carries a different chain back to the same root. Make sure your trace IDs and your actor-chain serialization both handle the fan-out without losing any branch.

## Why this is hard to retrofit

Identity is one of those concerns that is essentially impossible to add later. Every code path that makes an authenticated call needs to know about the chain. Every storage record that has an actor needs to grow a column. Every dashboard that filters by user needs a filter for chains. Every authorization rule needs to be re-expressed as a chain-aware predicate.

The repos we track that have tried to retrofit identity all show the same pattern: a long sequence of PRs that each fix one corner of the system, with synthesis themes in the weekly digest like "auth enforced at chat path leaked everywhere else" — see the W17 synthesis #11 in `~/Projects/Bojun-Vvibe/oss-digest/2026-W17/` which collected ACL/presentation-drift cites across litellm `#26420`, `#26421`, `#26399`, `#26416`, `#26424`, `#26425`, codex `#19349`, `#19356`, `#19359`, `#19271`, opencode `#24156`. That is what retrofitting identity looks like in the wild: many small fixes, each individually correct, none of them solving the underlying problem that the original design did not have a coherent actor model.

If you are early enough in your agent platform that you can still pick a coherent actor model, do it now. The cost is a few weeks of design work and a few hundred lines of policy code. The cost of doing it later is months of corner-case PRs and a permanent fog over your audit logs.

## The one-line summary

The actor of an action is not "the user" and is not "the agent." It is the ordered chain of principals from human intent to leaf execution, and your authorization, audit, billing, and rate-limiting systems all need to treat the chain — not any single element of it — as the unit of identity. Build for that, and the multi-agent systems you are about to build will compose. Don't, and they will accumulate identity bugs faster than you can fix them.
