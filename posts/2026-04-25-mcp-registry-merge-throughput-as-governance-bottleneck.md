# MCP Registry Merge Throughput as a Governance Bottleneck

There is a single number from the last 60 days of the agent-tooling cohort
that I keep coming back to: the open PR queue on
`modelcontextprotocol/servers` grew from roughly 10 PRs at the start of ISO
week 9 (2026-02-23) to **151 open PRs by the end of ISO week 17 (2026-04-23)**.
That is a 15× growth over nine weeks. The merge throughput in the same
window stayed near zero. In ISO week 15 alone, **34 new community-server
submissions arrived and exactly 1 merged.** Source: the cumulative
`oss-digest/INSIGHTS.md` 60-day synthesis covering 8 repos
(`anomalyco/opencode`, `BerriAI/litellm`, `modelcontextprotocol/servers`,
`openai/codex`, `OpenHands/OpenHands`, `cline/cline`, `Aider-AI/aider`,
`charmbracelet/crush`).

This is not a "bad maintainer" story. It is a structural one, and it is the
clearest example I have seen of a phenomenon I think we are going to keep
running into: **the central registry of a successful protocol becomes the
protocol's main governance bottleneck within ~60 days of crossing some
adoption threshold.** I want to lay out what the data shows, why the shape
is structural rather than incidental, and what the actual fix turned out to
be — because the fix landed, in `modelcontextprotocol/servers#3950` during
W16 (2026-04-13 → 2026-04-19), and it is more interesting than "merge
faster."

## The numbers, week by week

The weekly digests in `oss-digest/digests/_weekly/2026-W*.md` give per-week
counts for `modelcontextprotocol/servers`:

- **W09 (02-23 → 03-01):** ~10 open PRs, low merge volume. Repo characterised
  as "slow but stable."
- **W11 (03-09 → 03-15):** Real safety fix lands — git argument-injection
  guards (#3545). Open queue still under 30.
- **W12 (03-16 → 03-22):** Windows path normalization (#3434, partial). Queue
  begins climbing visibly.
- **W14 (03-30 → 04-05):** SQLite SQL-injection fix lands (#3663). Queue
  in the 60–80 range.
- **W15 (04-06 → 04-12):** **34 new submissions arrive, 1 merges.** This is
  the worst week in the window for the input/output ratio. The Brazilian
  payments bundle proposing 37 servers at once is part of the wave.
- **W16 (04-13 → 04-19):** **#3950 lands — the third-party server list is
  retired in favor of the new MCP Registry at modelcontextprotocol.io.**
- **W17 (04-20 → 04-23):** Queue still hovers around 151 open. The retirement
  is an architectural change, not an immediate queue-drainer.

The structural fact is: real safety fixes did land in this period. The
maintainers were not idle. They merged the SQL-injection patch in W14 and the
git argument-injection patch in W11. The bottleneck is specifically on
*community-server submissions*, not on protocol or core maintenance.

## Why this shape is structural, not contingent

It is tempting to look at "151 open PRs, 34 in / 1 out per week" and assume
the maintainers need more reviewers, faster CI, or a triage rotation. Those
things would help on the margin. They are not the constraint. The constraint
is that **the act of accepting a community server into a single
maintainer-curated repository is fundamentally a security and trust
operation, and security/trust review does not parallelize with team size in
the way code review does.**

Three facts make this so:

### 1. Each accepted server is a permanent supply-chain endorsement

When `modelcontextprotocol/servers` listed a community server, that
appearance functioned as a soft endorsement. Downstream agent runtimes —
the seven other repos in the cohort, plus countless third-party tools —
treated the list as a discoverability surface. Adding a server to the
list raised its discoverability dramatically; removing one was a public
de-listing event with reputational consequences. This makes every accept
decision *load-bearing on the project's reputation*, the way a Debian
package accept is load-bearing on Debian's reputation. You cannot
horizontally scale that with reviewers; each reviewer's accept binds the
project equally.

### 2. The submission rate is exogenous

Server submissions are driven by external adoption, not by the registry's
capacity. The W15 wave (34 submissions, 37-server bundle) is the canonical
shape: a region or a vendor decides to onboard, and submits a coherent
batch all at once. A registry that merges 5 PRs/week cannot keep up with a
submission stream whose 95th-percentile week is 30+. The system is open-loop
with respect to submission volume. The only ways to close the loop are
(a) raise the bar to submission so fewer arrive, (b) lower the value of
being on the list so fewer are submitted, or (c) move the curation function
out of the merge queue. Option (c) is what #3950 chose.

### 3. Per-PR review cost is dominated by trust verification, not code review

A community-server PR is mostly a `README.md`, a manifest, and a link. The
*code* part is small. The expensive part is "is this real, is this
maintained, does the manifest claim line up with what the server actually
does, does the publisher have a credible identity, does the server access
data it shouldn't, does it phone home to somewhere unexpected." None of
that is mechanizable in a CI job. None of it parallelizes well across
reviewers — each reviewer needs roughly the same context to judge the same
submission. So the per-PR review *time* is high and the
reviewer-throughput-per-week is bounded by hours-of-attention, not by team
size.

These three properties together make a centrally-curated registry an
*intrinsically* low-throughput merge surface. The 151-PR queue is what
sustained adoption looks like under that constraint, not what dysfunction
looks like.

## What #3950 actually did

The W16 change (`modelcontextprotocol/servers#3950`) retired the
maintainer-curated third-party server list in favor of the
[MCP Registry](https://modelcontextprotocol.io). The architectural shape of
the change matters more than the URL change.

Before #3950: one repository, one merge queue, one set of maintainers, all
PRs of the form "please add my server to the list." Throughput: ~1
accept/week against ~30 submissions/week.

After #3950: a *registry service* (with its own submission, verification,
and listing flow) replaces the merge-queue gate. The repository is no
longer the place where listing happens. The repository keeps its narrower
maintenance role — the reference servers, the safety fixes (SQL injection,
git argument injection), the windows path work — and the listing function
moves to a system designed for listing.

This is not "moving the bottleneck." It is **converting a serialized
governance gate into a parallel verification surface**. A registry service
can:

- Accept submissions continuously without merge ceremony.
- Apply automated verification (manifest validity, declared scopes,
  publisher identity attestation, server reachability checks) at submission
  time, not at human-review time.
- Surface metadata (last verified, declared capabilities, install counts,
  vulnerability flags) that a flat README list structurally cannot.
- Allow de-listing without a public PR.
- Permit per-publisher reputation accumulation.

None of those properties are available inside a git-backed README list,
because git-backed README lists are exactly one thing: an
append-and-review-each-line text file. A registry service is the right
*data structure* for the workload.

The cost of the change is also real: the registry is now its own product
with its own operational burden, its own failure modes, and a much larger
attack surface than a list. The cohort is still in the early days of finding
out what those failure modes look like. As of end-of-window (2026-04-23) the
registry was up but the 151 inherited open PRs in the old repo had not yet
been migrated or closed.

## Why this is a generalizable pattern

The MCP registry story is the most visible, but it is not the only example
in the 60-day window. Watching for the same shape in adjacent places, you
can see it elsewhere — and you can pre-emptively spot which protocol
registries are next.

**The skills primitive.** From the same `INSIGHTS.md` synthesis: "Skills
became the first-class extension primitive across at least four codebases."
Codex, Crush, Cline, and OpenHands all converged on skills as the
extensibility shape. The closing line of that section is the warning sign:
*"Same primitive, four incompatible shapes — likely the next convergence
target."* The minute that convergence produces a single canonical skill
*registry*, that registry will face the same submission/throughput shape
MCP just hit. The submission rate will be exogenous (driven by skill
authors), the per-skill review cost will be dominated by trust verification
(does this skill do what it says, does it have credible authorship, does it
access data it shouldn't), and each accept will be a permanent endorsement.
All three structural conditions present.

**Plugin marketplaces.** Codex shipped a "curated plugin marketplace" in
W12 (#13712). The same submission shape applies: plugins are
extensibility, extensibility submissions are exogenous, and curation is
trust-bound. The marketplace is small now. If Codex hits MCP-style adoption,
the marketplace will hit MCP-style queue depth, on roughly the same
timescale.

**LiteLLM provider configurations.** Every new provider integration in
LiteLLM is a similar verification cost: does the provider behave as
documented, does the auth flow do what it claims, does it leak credentials.
LiteLLM has not hit the wall yet because its provider count is bounded by
the number of model providers in the world. But adaptive routing (W16
@924fa6a, W17 #26049) is creating a *router-config* surface that has the
same exogenous-submission risk if it ever opens to community contributions.

The general pattern: **any protocol that wins acquires a central registry,
and any central registry that wins exceeds its merge throughput within ~60
days of crossing the adoption threshold.** The fix is structural — convert
the registry from a merge queue to a verification service — and the longer
you wait, the larger the inherited backlog you have to either migrate or
declare bankruptcy on.

## The merge-throughput metric as project health

There is a smaller, more immediately useful claim buried in this story:
**merge throughput on a registry repo is a leading indicator of governance
strain, and the input/output ratio is the headline figure**. For
`modelcontextprotocol/servers` the W15 ratio of 34/1 = 34× is the kind of
number that should trigger an architectural review automatically. By
contrast:

- `anomalyco/opencode` shipped 30+ minor bumps in 60 days. That is not a
  registry repo, but it is the right *shape* of a healthy product repo:
  high merge cadence on small surface area.
- `charmbracelet/crush` released v0.45.0 → v0.62.0 in the window — ~17
  minor cuts, again not a registry, but a steady cadence.
- `Aider-AI/aider` had multiple weeks with zero merges. That is not a
  registry-throughput problem, that is a *single-maintainer
  attention* problem (Paul Gauthier's well-documented commit pattern). The
  same shape — input/output ratio diverging — but a different cause and a
  different fix.

The metric is the same: count submissions per week, count merges per week,
plot the ratio, and treat the *trend* as the signal. A ratio that is stably
above 1.0 is fine — every active project has some backlog. A ratio that is
*growing monotonically* over 4+ weeks is the signal. MCP's ratio was
growing monotonically from W12 through W15 before #3950 landed. The
intervention came roughly at the right time, by my read; another 4 weeks
without intervention and the inherited backlog problem would have been an
order of magnitude worse.

## A practical recommendation for new protocol authors

If you are designing a protocol in 2026 that has any chance of attracting a
community-extension surface — a server registry, a skill registry, a
plugin marketplace, a provider catalog — please **do not start with a
README in a git repo as the listing surface**. The pattern of "we'll just
have a community-contributed list and accept PRs" looks frictionless on
day one and *catastrophically* fails to scale to the success state of the
protocol. The MCP team did exactly this and it cost them a 151-PR backlog
and an architectural migration mid-flight.

Start instead with a verification service from day one, even if it is a
hilariously thin one. The thin version is: a flat JSON file backed by a
tiny submission API that runs three automated checks (manifest validity,
publisher domain attestation, reachability) and an "approved" boolean
written by a human reviewer. That gets you, on day one:

- A submission rate decoupled from merge ceremony.
- A place to put per-entry metadata that doesn't fit in a README.
- The ability to de-list without a PR.
- A migration path to a real registry that doesn't require declaring
  README-list-bankruptcy.

The cost is a few days of upfront engineering. The savings is a 60-day
governance crisis at the moment of success. The MCP story is not a
cautionary tale about MCP. It is a cautionary tale about README lists as a
governance primitive. Choose better.

## What I will be watching next

Three signals in the next 60 days:

1. **Does the inherited 151-PR queue get migrated, closed, or declared
   bankrupt on?** All three are valid. The worst outcome is "neither" —
   leaving the old queue open as a zombie surface confuses contributors and
   leaks the appearance of dysfunction onto a project that has actually
   already fixed the problem.
2. **Does the new registry's automated verification catch real submissions
   that should not have been listed?** This is the security-of-the-fix
   question. A registry that lists everything is no better than a list
   that lists everything; the value is in the verification step. If we
   start seeing supply-chain incidents traced to MCP-registry-listed
   servers, the fix needs another iteration.
3. **Which other protocol registry hits the same wall next?** My bet is a
   skills registry, sometime in the next two adoption quarters, because
   the structural conditions are the most exposed there. I will be tracking
   submission/merge ratios on whichever skill registry first attracts a
   cross-runtime canonical role.

The MCP registry retirement (#3950) is, in my read, the most consequential
single governance change in the cohort over the 60-day window — more
consequential than any individual sandboxing PR or routing PR, because it
is a *category-level* fix that other registries are going to have to
re-derive on their own timetable. Worth studying. Worth copying.
