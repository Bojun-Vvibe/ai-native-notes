# The Permissive Duopoly — Apache-2.0 and MIT in a 102-CLI Catalog

I keep a catalog of 102 AI command-line tools at
`~/Projects/Bojun-Vvibe/ai-cli-zoo/clis/`, one directory per tool, each
with a `README.md` that walks through install footprint, supported
models, MCP support, telemetry, and — section 2 of every entry — the
license. The catalog spans the full bestiary: REPL coders, chat
front-ends, agent runtimes, repo-map indexers, commit-message
generators, kubernetes operators, embeddings shells, and the long tail
of single-purpose wrappers. It was assembled snapshot-by-snapshot
through 2026-04 and is large enough to ask the basic licensing
question: *what is the actual license shape of the open-source
AI-CLI ecosystem in 2026?*

The answer is short. The ecosystem is a permissive duopoly. Two
licenses — MIT and Apache-2.0 — together cover **86 of the 102
tools**, or roughly 84%. The remaining 16% is a long tail of weak
copyleft, strong copyleft, source-available, and the occasional
unconventional construction. There is essentially no middle.

## The headline tally

Walking each `*/README.md` and pulling the first license token that
appears under the license heading, the breakdown comes out as:

- **MIT — 47** (≈46%)
- **Apache-2.0 — 39** (≈38%)
- **AGPL-3.0 — 6** (≈6%)
- **Source-available — 2** (≈2%)
- **GPL-3.0 — 2** (≈2%)
- **BSD-3-Clause — 2** (≈2%)
- **FSL-1.1-MIT — 1** (≈1%)
- **Unparseable / dual / "see file" — 3** (≈3%)

102 tools, accounted for.

The two findings worth dwelling on are (1) that MIT and Apache-2.0
together are the entire centre of mass, and (2) that despite Apache-2.0
being the more featureful license — explicit patent grant, contributor
provisions, NOTICE-file mechanism — MIT still leads it by 8 entries.
That ratio is the part of the data that most surprised me, and most of
this post is about why.

## Why permissive at all

Before getting into MIT vs Apache-2.0, the prior question is: why is
the ecosystem permissive *as a default*? The numbers say 86/102, or
84%, are MIT-or-Apache. Eight of the remaining 16 entries are some
flavour of copyleft (AGPL-3.0, GPL-3.0); two are source-available;
the rest are stragglers.

Three forces explain the permissive default.

The first is the *dependency surface*. A modern AI CLI sits on top of
a stack — `tokio` or `asyncio`, an HTTP client, a tokenizer, a
provider SDK, a TOML/YAML parser, a TUI framework. Every layer of
that stack is overwhelmingly permissive (Apache-2.0 in the Rust
world, MIT in the JavaScript and Python worlds). Adopting copyleft
at the leaf of that stack imposes a license-compatibility analysis
that is uphill in every direction. The path of least resistance is
to match the stack you sit on. The stack is permissive, so the leaf
is permissive.

The second is *embeddability*. AI CLIs do not sit alone. They get
shelled out to from editors, embedded in CI scripts, wrapped by
internal tools at companies, vendored into product code. A
copyleft license — even a weak one like MPL — creates friction at
each of those embedding points. AGPL creates extreme friction. A
project that wants to be reachable by the largest possible
audience picks a license that does not require the audience to
re-paper their use. MIT and Apache-2.0 both clear that bar. AGPL
does not.

The third is *commercial alignment*. Of the 102 tools in the
catalog, a non-trivial fraction have an associated company that
sells a hosted offering (`anomalyco/opencode`, `cline/cline`,
`charmbracelet/crush`, `Aider-AI/aider` is community-funded but
the leaderboard is hosted, `BerriAI/litellm` has a hosted
gateway, `cognition-AI/devin` style projects, and so on). For
these projects, the open-source license must allow the hosted
business to coexist with the OSS distribution. Permissive
licenses do this trivially. AGPL does not — which is exactly
why the AGPL contingent in the catalog (6 entries) is dominated
by tools whose business model is *to* be the hosted offering and
to dare a fork to compete.

## MIT vs Apache-2.0 — the tighter race

That MIT leads Apache-2.0 47 to 39 is the part of the data that
needs a closer look. Apache-2.0 is, on paper, the better license
for this category of software:

- It carries an explicit patent grant. AI CLIs interact with
  patented model architectures and patented tokenization
  schemes; an explicit patent grant insulates downstream users
  from the contributor's patent portfolio.
- It carries a contributor representation. The contributor warrants
  they have the right to contribute the code. For projects that
  accept PRs from a wide author base, that representation reduces
  legal exposure.
- It carries a NOTICE-file convention. Downstream users get a
  predictable path to satisfy attribution requirements without
  digging through commit history.

MIT carries none of these. It is six paragraphs and a copyright
notice. So why does MIT lead?

Three reasons emerge from reading the catalog entries.

**Reason 1: MIT is the JavaScript/Python default.** Of the 47 MIT
entries, the overwhelming majority are written in TypeScript,
JavaScript, or Python. `npm init` defaults to ISC, but the
JavaScript community culturally defaults to MIT, and `pyproject.toml`
templates from popular project generators default to MIT. The
license is a downstream effect of language choice, not an active
decision. Apache-2.0, by contrast, dominates the Go and Rust
entries — `cargo new` does not pick a license, but the Rust
ecosystem culturally defaults to dual MIT/Apache, and Go
projects associated with a corporate sponsor (Google, HashiCorp,
Charm) default to Apache-2.0.

**Reason 2: MIT is the path of least friction for the smallest
projects.** Many of the catalog entries are individual-author
tools — a single maintainer, a single weekend's worth of code,
a README that says "this is what I wished existed." The MIT
license is shorter and easier to drop in. Apache-2.0 with its
NOTICE-file convention and patent-termination clause feels
ceremonial for a 400-line tool that wraps `curl` around an API
key. The catalog has many such tools. They land on MIT.

**Reason 3: Apache-2.0 is a stronger signal for serious
projects.** The Apache-2.0 entries skew toward projects that
expect to be embedded by other organisations — `aider`,
`gptscript`, `llamafile`, `LocalAI`, `OpenHands`, `LiteLLM`,
several MCP servers. These are tools whose adopters are doing
license audits, and Apache-2.0 makes the audit pass with no
follow-up questions. The MIT-licensed entries that survive a
similar audit do so because the auditor has already classified
MIT as audit-clean.

So: MIT wins on count because it's the cultural default in the
languages where most CLIs are written. Apache-2.0 wins on
seriousness — the projects that pick it tend to be the ones
that pass adoption review at organisations.

## The copyleft tail

Eight of 102 tools are copyleft. That is small but not negligible,
and the tools in this bucket are interesting precisely because
they bucked the default.

The six AGPL-3.0 entries are concentrated among tools whose
maintainer has an explicit "no SaaS-without-contribution"
posture. The AGPL's network-use trigger is the lever — if you
host the tool as a service, you must release the modifications.
For tools whose authors expect the cloud providers to repackage
their code, AGPL is the most teeth-bearing OSI-approved option.
The tradeoff is that AGPL excludes the tool from many corporate
deployments. The maintainer is making that tradeoff knowingly.

The two GPL-3.0 entries are older and reflect the pre-AGPL
copyleft instinct. GPL-3.0 has a weaker network-use story, but
strong distribution-time copyleft. These tools are typically
shipped as binaries, not as services, so GPL-3.0 still has
teeth at the distribution boundary.

The two BSD-3-Clause entries are essentially MIT with an
attribution-in-promotional-materials clause. They are
functionally permissive and group with the duopoly for any
practical adoption analysis.

## The source-available outliers

Two tools sit outside the OSI-approved space entirely. Their
licenses are commonly described as "source-available" — the
source is publicly readable, but the license withholds one or
more rights an OSI license would grant. Typical patterns:

- a non-compete clause prohibiting building a competing
  hosted service
- a use-case restriction (no use in defence applications, no
  use to train other models, no use by entities above a
  revenue threshold)
- a delayed-conversion clause (FSL, BSL) that converts to an
  OSI license after N years

The single FSL-1.1-MIT entry in the catalog is the cleanest
form of the delayed-conversion pattern: the code is
non-compete-restricted today and converts to MIT after two
years. For users who can wait, FSL is functionally MIT-with-
a-delay. For users who cannot, it is a non-OSI license today.

The two pure source-available entries do not convert. They are
permanent restrictions, with the source published as a courtesy
to users who want to read it but not as a grant of the freedoms
OSI defines. Whether to include them in an "open-source" CLI
catalog is a judgment call. I include them because the source
is reachable and because their existence is part of the
landscape, but I flag the license category clearly in each
entry's section 2.

## The unparseable three

Three entries did not yield a clean license token under the
license heading because their license is dual or qualified —
"MIT or Apache-2.0 dual-licensed" (Rust convention),
"AGPL-3.0 for the open-source core, MIT for the CLI client"
(split-license), and one entry that uses a custom phrasing
that requires reading the linked LICENSE file to disambiguate.
Folding the dual-license entries into both buckets would push
MIT and Apache-2.0 even closer together at the top of the
tally. The headline ratio (84% permissive duopoly) does not
move.

## What this means for picking a license

If you are writing an AI CLI in 2026 and trying to decide
what license to use, the data has three suggestions.

First, *match your stack*. If your dependencies are MIT, MIT.
If your dependencies are Apache-2.0, Apache-2.0. The compatibility
matrix is permissive in both directions, but the audit story
is cleaner when the leaf matches the trunk.

Second, *match your project's seriousness signal*. If you
expect the tool to be adopted by other organisations and to
go through their license-audit process, Apache-2.0 sends a
stronger signal that you have thought about patent grants,
NOTICE-file conventions, and contributor representations.
MIT sends a weaker signal but is not in itself a problem;
the audit will pass either way.

Third, *only pick copyleft if you have a specific lever in
mind*. AGPL-3.0 is a fine choice if your business model
depends on cloud providers being unable to silently
repackage your work. It is a poor choice if your business
model depends on being embedded inside other companies'
internal tools — most adopters will not be allowed to embed
you. Pick AGPL with intent, not by default.

The 6% AGPL share in the catalog matches that posture. The
projects on AGPL are the ones with a specific lever to pull.
The projects on MIT and Apache-2.0 are the ones who picked
the path of least friction, which is the right choice for
the majority case.

## What this catalog cannot tell you

Three things the licensing tally is *not* doing.

It is not measuring code freedom in the deepest sense. A
tool licensed MIT but built around a closed-source hosted
backend (where the CLI is a client to a SaaS that holds the
state, the prompts, the model weights) is, in practice,
less free than a copyleft tool whose entire stack is
open. The license tells you what the *code* can do; it does
not tell you what the *system* can do. Several entries in
the catalog are MIT-licensed clients of closed-source
backends. A purely license-based read would call them
permissive; a system-level read would call them gated.

It is not measuring contributor experience. Apache-2.0
projects often run a CLA process; MIT projects often do
not. Whether the CLA is a barrier or a comfort depends on
the contributor, but it is a real difference in
day-to-day OSS experience that the license-name tally does
not surface.

It is not measuring durability. A license can change. A
project that is Apache-2.0 today can re-license under a
source-available scheme tomorrow, and several high-profile
infrastructure projects have done exactly that in recent
years. The catalog is a snapshot of 2026-04, and the
snapshot will age. The 84% permissive duopoly is a true
statement about *now*, not a prediction about *next year*.

## Closing

102 tools, two licenses covering 86 of them. The
remaining 16 distribute across copyleft, source-available,
and dual-license arrangements. The headline finding is the
duopoly. The interesting finding is that within the
duopoly, MIT (47 entries) leads Apache-2.0 (39 entries) not
because MIT is the better-engineered license — it is not —
but because MIT is the cultural default of the languages
in which most CLIs are written, and because the smallest
projects in the catalog reach for the shortest paperwork.
Apache-2.0 wins where seriousness wins: corporate-sponsored
projects, projects that expect organisational adoption,
projects whose maintainers have done a license audit at
their day job and know what makes one easy.

The license picture is, in this sense, a class portrait
of the ecosystem. The big serious projects are
Apache-2.0. The lone-author weekend projects are MIT.
The projects with a lever to pull are AGPL. The
projects that wish OSI did not exist are
source-available. And the rest is the long tail — three
unparseable entries that resisted classification because
their authors thought hard enough about licensing to do
something unusual. The tally adds to 102 because the
catalog adds to 102. The shape of the tally is the
shape of the work.
