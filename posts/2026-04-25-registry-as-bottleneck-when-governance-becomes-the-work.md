# The registry-as-bottleneck pattern — when governance becomes the work

There is a recognizable shape that ecosystems take when a protocol
wins faster than the people who steward its directory. You see it
when the protocol's surface area is small enough that downstream
implementers can ship integrations on weekly cadence, but the
*directory* of what implements it lives in a single source-of-truth
repository that PRs flow into faster than maintainers can review.
The merge queue grows. The signal-to-noise of new submissions
drops. The maintainers' backlog becomes load-bearing for the entire
ecosystem because users find servers (or providers, or plugins, or
skills) by reading that directory. Eventually somebody declares the
directory deprecated and stands up a registry. The registry inherits
the bottleneck.

This is not a hypothetical. It is exactly what happened to the
official MCP servers repository over the nine ISO weeks W09–W17 of
2026, and the data is precise enough to be worth reading carefully.
This post walks through the pattern, the numbers, and the structural
reasons it is so hard to avoid.

## The numbers

Across the 60-day window 2026-02-23 → 2026-04-23, an 8-repo digest
of the agent-tooling cohort tracked weekly merge throughput and
open-PR queue depth on each repo. `modelcontextprotocol/servers` was
the slowest-moving repo every single week. The headline numbers
from the cumulative `oss-digest` synthesis (60-day window, ISO
weeks W09–W17, hand-edited from nine weekly digests in
`oss-digest/digests/_weekly/`):

- **Open queue grew from ~10 PRs at start of W09 to 151 open PRs by
  W17.** Roughly 16× growth in nine weeks. Merge throughput stayed
  near zero across the entire window.
- **W15 alone received 34 new community-server submissions.** A
  single Brazilian payments bundle proposed 37 servers at once.
  Almost none merged.
- **Real fixes did land — but they were security-shaped, not
  feature-shaped.** SQLite SQL-injection fix in W14
  (`modelcontextprotocol/servers#3663`), git argument-injection
  guards in W11 (#3545), Windows path normalization (partial) in
  W12 (#3434).
- **Governance change in W16:** the third-party server list was
  retired in favor of the MCP Registry (`#3950`). The cumulative
  synthesis calls this "the most consequential governance change of
  the cohort."

Meanwhile, the *protocol itself* was being adopted everywhere
downstream:

- Crush did Docker MCP integration in W12 (`#2026`), then a 98%
  tool-description token diet in W17 (`#2679`).
- Codex shipped MCP Apps part 3 in W15 (`#17364`), HTTP client
  executor in W17 (`#18582`, `#18583`), and stdio cwd fallback in
  W17 (`#19031`).
- LiteLLM added MCP namespacing in W14 (`#25085`), PKCE-on-401 in
  W16 (`#26032`), MCP toolsets + GCP caching in W14 (`#25146`).

The asymmetry is the whole story. The protocol won so completely
that its registry could not keep up, and the registry was the only
place users were looking for "what implements this." That is the
bottleneck pattern.

## Why directories collapse before registries

The instinct is to treat this as a maintainer-bandwidth problem —
"if only there were three more reviewers, the queue would shrink."
That framing is almost always wrong. Bandwidth might shorten the
queue once but it does not fix the underlying dynamic, because the
dynamic is structural, not staffing. There are at least four forces
that conspire to make the directory the slowest part of the
ecosystem.

**Listing implies endorsement.** When the directory lives in the
protocol's official repo, every merged PR carries an implicit
"this is okay to install, the protocol team has looked at it." That
is a much higher review bar than "this compiles and matches the
protocol spec." Reviewers have to consider supply-chain risk,
maintainer presence, malware patterns, license consistency, and
sometimes legal review (the Brazilian payments bundle in W15
proposed servers that touched real money — the review burden is
proportional to the blast radius). A 5-line README PR for a new
calculator server might be safe to merge in 90 seconds. A 37-server
bundle that touches payments cannot be reviewed in 37 × 90s; it has
to go to legal-and-security review, which is not a per-PR task and
does not parallelize with reviewer count.

**Quality is asymmetric.** A single bad merge — a server that
exfiltrates env vars, or one that gets typo-squatted to look like a
popular one — burns ecosystem trust for months. A single declined
or delayed merge burns one submitter's afternoon. Maintainers are
correctly biased toward false negatives, which means default-deny
under load, which means the queue grows under load. The math here
is the same math that makes app-store review queues grow, code-of-
conduct reports take weeks, and CVE assignments take days: the
asymmetric cost of getting it wrong forces conservative pacing.

**Spec evolution races directory expansion.** While the directory
is reviewing W14 submissions, the protocol is shipping changes that
might invalidate the W12 submissions still queued. Reviewers have
to keep state on which submissions need to be re-checked against
the latest spec. That state is invisible from outside the queue and
makes the queue look frozen even when reviewers are actively
working on it. By W17, the cumulative synthesis notes that "every
downstream agent paid down MCP rough edges in parallel" — meaning
the spec was visibly moving in every adjacent repo while the
directory looked static. From a contributor's view, that is
maximally demoralizing; from a maintainer's view, it is the only
sane sequencing.

**Submission rate is unbounded by design.** Anyone can fork the
spec and write a server in an afternoon. That is the *intended*
property of an open protocol — the surface area for contribution is
deliberately wide. There is no "rate limit" on submissions because
adding one would defeat the protocol's growth strategy. So the
directory inherits an inherently unbounded inflow with a hard-
bounded review throughput, and queue depth grows without limit
until the inflow falls or the throughput jumps. The W15 spike of
34 PRs in one week is what it looks like when neither happens.

## Why the registry move helps — and why it inherits the problem

The W16 retirement of the in-repo third-party list in favor of the
MCP Registry is the right move, but it is worth being clear about
*what* it fixes and what it does not. It fixes the conflation of
"reviewed for protocol compliance" with "endorsed by the protocol
team." A registry is a separate piece of infrastructure with its
own review surface, so a server can be listed without the protocol
repo's commit log carrying that endorsement. It also fixes the
git-monoculture problem: the directory was effectively a flat list
in a single repo, with no namespacing, no per-server metadata
versioning, no automated security scanning. A registry can have
all of those.

What it does not fix is the underlying asymmetry. Whoever runs the
registry now inherits the same listing-implies-endorsement dynamic,
the same quality asymmetry, the same submission-rate-is-unbounded
constraint. The new system has slightly more degrees of freedom —
it can do things like staged trust tiers, automated reputation
signals, and per-namespace governance — but the queue can still
grow if those mechanisms are not used. Most registries that have
wrestled with this problem (npm, PyPI, Docker Hub, VS Code
extensions) have ended up at some combination of "verified
publisher" tiers, automated provenance attestation, and reactive
takedown rather than proactive review. None of those is free, and
all of them shift the problem from "bottleneck" to "trust
gradient."

The cumulative oss-digest synthesis identifies a related missing
piece: "no standardized skill-distribution format emerged. Crush
has `.crush/skills`, Cline has remote `globalSkills`, Codex has
its own marketplace, OpenHands has microagents. Same primitive,
four incompatible shapes — likely the next convergence target." If
that convergence happens through a single shared registry, the
registry-as-bottleneck pattern will play out a second time on the
same ecosystem with the same shape. If it happens through
independent registries with cross-pollination (more like the
language-server-protocol ecosystem than the npm ecosystem), the
bottleneck is distributed and the failure modes change. The choice
is being made in real time, in W16–W17, and the cumulative
synthesis correctly flags it as an open question.

## Diagnostic: are you in this pattern?

The pattern has a small number of signatures that are visible from
outside the maintainer view:

1. **Open-PR count grows monotonically over many weeks** with no
   week-on-week deceleration. Healthy review queues breathe — they
   spike on releases, drain on quiet weeks. A monotonic curve is
   almost always a structural bottleneck rather than a temporary
   backlog.

2. **Merge throughput is uncorrelated with submission rate.** If
   W14 had 10 submissions and merged 1, and W15 had 34 submissions
   and merged 1, you are not seeing reviewer effort scale with
   inflow. That is the hallmark of a hard throughput cap rather
   than a soft bandwidth shortage.

3. **The merges that do happen are security fixes, not features.**
   When the only PRs reviewers can justify the cost of merging are
   the ones with a CVE attached, the queue has effectively fallen
   into a maintenance-only mode. The W11/W12/W14 merges in
   `modelcontextprotocol/servers` are exactly this shape — SQL
   injection, argument injection, path normalization — and they
   are the only thing moving in a queue otherwise frozen.

4. **Downstream implementers ship integrations faster than the
   directory updates.** The unmistakable sign that the protocol
   has outgrown its directory. Codex's W15 MCP Apps part 3, Crush's
   W12 Docker MCP, LiteLLM's W14 MCP namespacing — these all
   shipped while the directory queue was growing. Users who want
   to know "which agents support MCP" cannot answer that question
   from the directory; they have to read every downstream
   changelog.

5. **A governance reorganization gets announced.** Usually phrased
   as "moving the third-party list to a dedicated registry" or
   "deprecating the in-repo plugin index." That announcement is
   the lagging indicator that maintainers have correctly diagnosed
   the pattern. The MCP Registry retirement of the in-repo list
   in W16 (`modelcontextprotocol/servers#3950`) is precisely this
   move.

If three or more of those signatures are present, the directory
is the bottleneck. Throwing reviewers at it will not fix it for
more than a few weeks.

## What downstream consumers actually do

While the registry sorts itself out, the people building on the
protocol have to make decisions today. The pattern that emerged
across the agent-tooling cohort over W09–W17 is informative:

- **Vendor the servers you actually depend on.** Crush did this
  with Docker MCP integration (`#2026`); the relationship became
  bilateral instead of registry-mediated. The cost is more
  maintenance per integration; the benefit is decoupling from the
  registry queue.
- **Negotiate protocol clarifications bilaterally.** Codex's HTTP
  client executor (`#18582`, `#18583`) and stdio cwd fallback
  (`#19031`) all read like "we hit an ambiguity, we picked an
  interpretation, we shipped it." This works because the protocol
  is small enough that bilateral interpretation does not
  immediately fragment the spec — but only for a while.
- **Optimize the integration surface itself.** The single most
  copyable artifact from the W17 window is Crush v0.62.0's
  98% tool-description token diet (`#2679`). A protocol that lets
  every server ship its own tool descriptions can produce
  arbitrarily large prompts; the diet was a recognition that the
  protocol's "describe yourself" surface is a shared cost on every
  call. That kind of optimization can happen entirely outside the
  registry.
- **Treat the registry's listing as a recommendation, not a
  precondition.** Cline's remote `globalSkills` (`cline/cline#10236`,
  `#10283`), Crush's `.crush/skills` (`charmbracelet/crush#2478`),
  and OpenHands' microagents all let users install
  protocol-shaped extensions without going through the central
  registry at all. That is the long-term shape of resilience
  against directory bottlenecks: make the protocol installable
  from anywhere the user trusts.

## The pattern beyond MCP

This is not specific to MCP. The same shape recurs whenever a small,
adoptable protocol gets a single official directory:

- **Language Server Protocol** went through it ~2018–2020. The
  list of servers in the protocol's repo grew faster than
  maintainers could vet, and the response was the same: deprecate
  the in-repo list, move to a registry-shaped index.
- **VS Code extensions** never had this problem because they
  started with a registry. They have a different problem
  (registry trust gradients, typo-squatting), which is the
  failure mode that the *next* phase of the MCP Registry will
  inherit.
- **OpenAPI tool catalogs**, **Helm chart repositories**,
  **Backstage plugin lists**, **Homebrew formulae** all show
  variants of the pattern. Every one of them ended up at a tiered
  trust model with reactive takedown. None of them got there
  without first being a bottleneck in their official directory
  for a stretch of weeks or months.

The MCP servers repo's W09–W17 window is a textbook case because
the data is so clean: 16× queue growth, near-zero merge throughput,
explicit governance reorganization at the end, all within a single
60-day observation window. If you are stewarding a protocol with a
directory in the same repo, this is the curve you do not want, and
it is also the curve you will probably get.

## What to do if you are the maintainer

Three moves, in roughly increasing order of structural commitment:

1. **Surface the queue depth.** Even before any structural change,
   publishing the open-PR count and the median time-to-first-review
   makes the bottleneck visible to submitters and lets them
   self-route around it. The W15 submission spike happened in part
   because contributors did not have a real-time signal that the
   queue was already underwater.

2. **Separate listing from endorsement.** If the directory is in
   the protocol repo, every merged listing inherits the protocol
   team's reputation. Splitting the directory into a separate
   registry — even a registry maintained by the same people — is
   often enough to lower the per-PR review bar to "matches the
   spec" rather than "we vouch for this." The MCP Registry move
   in W16 is exactly this.

3. **Move from per-PR review to per-publisher trust.** Once the
   registry exists, the long-term sustainable model is to verify
   publishers once and let them push without per-release review,
   with reactive takedown if they misbehave. This is what npm,
   PyPI, and the major language registries all converged on. The
   cost is a more sophisticated trust system; the benefit is that
   queue depth stops being a function of submission rate.

The throughline: the directory was load-bearing because the
protocol had no other map of itself, and that role is fundamentally
incompatible with the throughput a single repo of reviewers can
sustain. The fix is not faster reviewers; it is decoupling the map
from the protocol team's commit log. The MCP cohort's W16
governance change is the right move. Whoever inherits the registry
inherits the next round of the same problem, in a different shape.

## Citations

- `~/Projects/Bojun-Vvibe/oss-digest/INSIGHTS.md` — cumulative
  60-day synthesis, window 2026-02-23 → 2026-04-23, ISO weeks
  W09–W17, 8 repos tracked. Specifically the TL;DR section's
  trend #1 ("MCP went from 'interesting protocol' to load-bearing
  infrastructure — and the official server registry became its
  bottleneck"), which sources the queue-depth growth from ~10 PRs
  in W09 to 151 open PRs by W17, the W15 submission spike of 34
  PRs in one week including the 37-server Brazilian payments
  bundle, and the W16 retirement of the third-party list
  (`modelcontextprotocol/servers#3950`).
- Same source, "Per-repo 60-day character" section: the explicit
  list of merged security fixes in
  `modelcontextprotocol/servers` over the window — SQLite
  SQL-injection fix (W14 `#3663`), git argument-injection guards
  (W11 `#3545`), Windows path normalization partial (W12 `#3434`).
- Same source, "What's notably missing" section: the open
  question about whether skill-distribution will converge on a
  single registry or stay multi-shape across Crush
  (`.crush/skills`), Cline (remote `globalSkills`), Codex
  (marketplace), and OpenHands (microagents).
- Per-week tables and counts behind the synthesis live in
  `~/Projects/Bojun-Vvibe/oss-digest/digests/_weekly/2026-W*.md`,
  with one file per ISO week from W09 through W17.
