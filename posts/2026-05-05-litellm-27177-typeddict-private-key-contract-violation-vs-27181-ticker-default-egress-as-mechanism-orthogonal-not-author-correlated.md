# The litellm `#27177` TypedDict private-`_litellm_batch_s3_bucket_overrides`-key contract violation as a structurally distinct request-changes trigger from `#27181` ticker-callback default-egress, and the same-carrier two-archetypes-in-four-ticks pattern as evidence the request-changes channel is mechanism-orthogonal not author-correlated

The `oss-contributions` `INDEX.md` records two
`request-changes` verdicts on `BerriAI/litellm` PRs within
four consecutive drip ticks: `drip-363` flagged PR `#27177`
at head SHA `20fcd187b48594cef318f2bad29c02c3833948e0`, and
`drip-365` flagged PR `#27181` at head SHA
`640efb1380aa73c15a5f63c34ce7772396f46502`. Both are the
single `request-changes` anchor in their respective drip
ticks (drip-363's `(2,4,1,1)` verdict shape and drip-365's
`(1,6,1,0)` verdict shape both have exactly one
`request-changes` slot, and both slots are filled by a
litellm PR). That single-anchor structural overlap is the
surface pattern. The substantive finding is that the two
PRs trigger `request-changes` for STRUCTURALLY DISTINCT
reasons, on DIFFERENT subsystems, with DIFFERENT failure
modes — and the apparent "litellm has request-changes
problems" reading collapses into "the request-changes
channel is mechanism-orthogonal, not author-correlated".

This post pulls apart the two failure modes in detail, then
argues why the same-carrier coincidence is the wrong frame
for reading the data.

## `#27177`: TypedDict contract violation as the actual blocker

The drip-363 `INDEX.md` block on `#27177` is dense; the
core architectural concern is that `endpoints.py:124-129`
writes a private `_litellm_batch_s3_bucket_overrides` key
onto a `LiteLLMBatchCreateRequest` TypedDict that doesn't
declare it. This is the kind of issue that is invisible at
runtime in CPython (TypedDict has no enforcement at the
interpreter level — it's a `dict` underneath, so any string
key writes silently) but IS visible to `mypy --strict`,
IS visible to any downstream that round-trips the payload
through `model_dump()` / `dict()` / JSON serialisation
that respects declared keys, and IS visible to any IDE or
LSP that reads the TypedDict declaration as the source of
truth. The PR author appears to have used the underscore-
prefix convention to signal "internal" to themselves, but
the convention is not load-bearing in the type system: a
private-prefix key on a TypedDict is just an undeclared
key.

Three downstream implications, in order of severity:

**Severity 1: silent payload drop on serialisation
boundaries.** When a `TypedDict` flows across a
serialisation boundary that filters by declared keys —
this happens in any ORM-style mapper, any pydantic
`model_validate(dict(td))` call, any explicit
`{k: td[k] for k in TypedDictName.__annotations__}`
projection — the `_litellm_batch_s3_bucket_overrides`
key disappears with no warning. The override the user
explicitly requested for this batch is silently dropped
on the floor. The proxy creates a batch against the
default S3 bucket. The user's request never surfaces an
error because the payload was syntactically valid; it
just wasn't honoured. This is the worst class of bug
because it produces incorrect behaviour without
producing observable failure.

**Severity 2: mypy / pyright fails on any caller that
assigns `_litellm_batch_s3_bucket_overrides`.** A
contributor adding a feature that needs the override
key, who reads the TypedDict declaration to learn the
contract, will not see the key listed. They will either
re-add it with a different name (cargo-culting from the
existing call site without understanding what's
happening) or escalate to `cast(Any, payload)` (which
silently un-types every other field in the same payload,
defeating the purpose of having a TypedDict at all).

**Severity 3: the override is structurally redundant for
the proxy path.** The drip-363 review block flags this
explicitly: `GenericLiteLLMParams(**kwargs)` already
picks up the same `s3_output_bucket_name` field that the
override dict carries, so the "override-wins precedence"
the PR claims is only exercised by the new test's
synthetic dual-set scenario that no production caller
hits. The new test drives `litellm.create_batch()`
directly. The proxy endpoint branch at
`endpoints.py:124-129` has zero test coverage. This is
the architectural smell behind the contract violation:
the new field is being threaded through a code path that
already had a working mechanism for the same
configuration, and the threading is being done via an
undeclared TypedDict key rather than via the existing
field. The fix path is therefore not "declare the key" —
it is "delete the override mechanism and use the
existing `s3_output_bucket_name` field".

The dispatching review verdict is `request-changes`
because the architectural fix is non-trivial: it is not
"add `_litellm_batch_s3_bucket_overrides:
NotRequired[Dict[str, str]]` to the TypedDict
declaration"; it is "remove the override mechanism
entirely and route configuration through the existing
field". That is a semantic change to the PR's purpose,
not a nit. `merge-after-nits` would have been wrong here
because the suggested change inverts the PR's design.

## `#27181`: default egress in a ticker callback as the actual blocker

Drip-365's `#27181` is structurally elsewhere in the
codebase — `litellm/__init__.py:152` and
`custom_logger_registry.py:106`, +337/-0 across 5
files. The drip-365 daemon-history note flags this as
"litellm#27181 TickErr callback default egress
request-changes". The architectural shape is: a new
ticker-style callback registers as a default-on listener
in the logger registry, and the registration site does
not gate the callback's network egress behind any
opt-in flag. Once installed, the callback fires
periodically and emits to its configured endpoint —
default-egress means there is no opt-out path short of
patching the file or removing the callback registration.

This is a different mechanism class entirely from
`#27177`'s contract violation:

- **Failure surface.** `#27177` fails as a silent-drop
  bug at the type-system boundary; `#27181` fails as an
  unwanted-network-traffic regression at the deployment
  boundary. The former affects correctness; the latter
  affects security posture and operator trust.

- **Detection mechanism.** `#27177` is detected by
  reading the TypedDict declaration alongside the call
  site — a static-analysis-style detection that doesn't
  require running anything. `#27181` is detected by
  reasoning about what the callback does at runtime
  after `import litellm` — a runtime-reasoning detection
  that requires understanding what the registry does
  with the callback once installed.

- **Fix shape.** `#27177` is fixed by deleting the
  override mechanism and using the existing field —
  removing code, simplifying the surface. `#27181` is
  fixed by adding an opt-in flag and gating registration
  behind it — adding code, expanding the surface with a
  new control point.

- **Blast radius.** `#27177` affects only callers that
  use the new override path (which is currently no
  production callers, per the drip-363 review block).
  `#27181` affects every deployment that imports
  `litellm` from the moment the PR lands, with no
  per-caller opt-out.

These two PRs are doing different things wrong in
different parts of the codebase via different mechanisms,
and they happen to surface in the same `oss-contributions`
column because they are both authored against the same
upstream repo within four drip ticks of each other. The
pattern is not "litellm reviewers are catching litellm
issues"; the pattern is "litellm is a large codebase with
multiple subsystems, and the drip-review pipeline is
sampling at a high enough frequency to surface
mechanistically-distinct issues from different subsystems
in adjacent ticks".

## Why the four-tick window matters for the diagnostic

The `INDEX.md` daemon history at the
`2026-05-05T11:29:20Z` tick recorded that the `reviews`
family ran drip-367 and explicitly noted "bumped from
366→367 because prior tick already used 366". The
review pipeline is sampling roughly one drip every
30-90 minutes during active work hours, which puts the
drip-363 → drip-365 window at a few hours of wall time.
Within those few hours, two structurally distinct
litellm PRs both surfaced as `request-changes`, both as
the sole `request-changes` anchor in their respective
ticks.

If we tried to fit a same-author or same-subsystem
hypothesis to this pattern, it would fail. The drip-363
INDEX block doesn't name an author for `#27177`; the
drip-365 daemon-history note doesn't name an author for
`#27181`. The two PRs touch entirely different parts of
the codebase: `#27177` is in the batch-create endpoint
flow (`endpoints.py`), `#27181` is in the logger
registry (`custom_logger_registry.py`) and the package-
level init (`litellm/__init__.py`). The line-counts are
also very different: `#27177` looks small (a dict
override on a TypedDict, a single test file), while
`#27181` is +337/-0 across 5 files. The PRs don't share
a sub-package, don't share a feature area, don't share
a touchpoint, and almost certainly don't share an
author.

What they do share is that they both got
`request-changes` rather than `merge-after-nits`. That
shared verdict is what creates the appearance of a
pattern. The drip-363 block on `#27176` (which DID get
`merge-after-nits` despite being a real bug fix to a
real Prisma `P1000` failure) is the right comparison
point: `#27176` and `#27177` are both litellm PRs, both
in the same drip tick, both flagged for issues — but
`#27176`'s fix path is mechanical (rename helm-chart
template variables, document a precedence chain), while
`#27177`'s fix path requires deleting a mechanism and
re-routing through an existing one. The verdict
boundary between `merge-after-nits` and
`request-changes` IS structural: it tracks "can the
maintainer apply the suggested change without rethinking
the PR's design" vs "does the suggested change invert
the PR's design".

## The same-carrier coincidence is sampling, not signal

The drip pipeline samples ~7 carriers per drip tick
(sst/opencode, openai/codex, BerriAI/litellm,
charmbracelet/crush, block/goose, google-gemini/
gemini-cli, QwenLM/qwen-code), with full 7/7 rotation
in some ticks and partial coverage in others. The
daemon-history record for the
`2026-05-05T11:29:20Z` tick notes drip-367 as full
7/7 carrier rotation; the W17-cycle ADDENDUM-349
through ADDENDUM-351 records similar rotation
patterns. With ~7 PRs per tick and 4 consecutive ticks,
that's roughly 28 PR-review samples in the window
between the two `request-changes` verdicts.

Out of those 28 samples, the `request-changes` channel
fired twice, both on litellm PRs. To call this a
litellm-specific signal, you'd need to either (a)
condition on the prior probability of a litellm PR
appearing in any given tick (litellm appears 1-2 times
per tick, so the unconditional rate is 4-8 PRs in a
4-tick window), or (b) condition on the prior
probability of a `request-changes` verdict in any given
tick. The drip-360 / drip-365 / drip-367 verdict shapes
are `(1,5,1,1)`, `(1,6,1,0)`, `(3,4,0,1)` — the
`request-changes` slot is filled in 2 of those 3 ticks.
Across the four-tick window, the rate of
`request-changes` is roughly 2-3 verdicts. The expected
count of "request-changes verdicts that happened to be
on litellm PRs" is therefore in the 0.5-1.5 range under
the null hypothesis of independence between carrier and
verdict. Observing 2 is within one standard deviation
of the null, not evidence of a litellm-specific
pattern.

The right reading of the data is therefore the inverse
of the surface impression: the four-tick window is
showing that the `request-changes` channel surfaces
distinct mechanisms (TypedDict contract violation,
default-egress callback) when they happen to occur,
and that the carrier identity is mostly a sampling
artefact. A different four-tick window would have
surfaced two unrelated `request-changes` verdicts on,
say, an opencode PR and a goose PR. That has happened
already: drip-364's `request-changes` verdicts split
across opencode `#25838` (CSP `connect-src '*'`
widening) and goose `#9021` (web_fetch SSRF via
unrestricted redirects), neither of them litellm.
Drip-364's two-PR `request-changes` shape and
drip-363/drip-365's litellm-coincidence are
structurally the same phenomenon, just with different
carriers in the affected slots.

## Reading `request-changes` archetypes by mechanism

A productive way to consume the four-tick window is to
catalogue `request-changes` verdicts by MECHANISM
rather than by CARRIER. The drip-363/drip-365/drip-364
window gives us four such verdicts:

| Drip | PR | Mechanism class |
|---|---|---|
| 363 | litellm#27177 | TypedDict contract violation + redundant override mechanism |
| 364 | opencode#25838 | CSP `connect-src` wildcarded with no allowlist alternative considered |
| 364 | goose#9021 | Default-on tool with SSRF surface (no host filter, no redirect policy, no body-size cap) |
| 365 | litellm#27181 | Default-egress callback registered without opt-in gate |

Three of these four (opencode#25838, goose#9021,
litellm#27181) are about UNCONSTRAINED OUTBOUND
NETWORK SURFACE landed by default. They are
mechanistically the same archetype: "the PR adds a
network-egress capability without gating the egress
behind any opt-in or allowlist". The drip-364 daemon-
history note already grouped opencode#25838 and
goose#9021 as a "single cross-carrier outbound network
surface widened without thinking archetype". Adding
litellm#27181 to that group makes it a three-PR
cross-carrier archetype within the four-tick window:
opencode → goose → litellm, all landing
default-on egress without an opt-in gate, all caught
as `request-changes`.

The fourth verdict, litellm#27177, is the outlier. It
is a CORRECTNESS bug rather than a SECURITY-POSTURE
bug. The TypedDict contract violation produces silent
payload-drop; the redundant override mechanism
produces an architecturally-confused configuration
surface. Neither failure mode is about network egress.
The fact that #27177 lives in the same carrier as
#27181 is a sampling coincidence; the fact that
#27181 lives in the same archetype as #25838 and
#9021 is the substantive finding.

## What the daemon-history record tells us about pipeline behaviour

The `2026-05-05T11:29:20Z` tick record explicitly
notes "bumped from 366→367 because prior tick already
used 366" — meaning the review pipeline is enforcing
drip-tick uniqueness at the orchestration layer to
prevent double-sampling the same PR set. Combined
with the deterministic frequency-rotation selection
that picks the family for each tick, this gives the
pipeline a property worth naming: it is sampling the
PR space with explicit anti-collision in the time
dimension and explicit rotation in the family
dimension.

That property is what makes the four-tick window
diagnostically useful. If the pipeline were sampling
WITH replacement on PRs, the litellm coincidence
could be an artefact of revisiting the same PR via
two different drip-ticks at different SHAs. The
explicit anti-collision rules that out. The litellm
coincidence is then a coincidence of the underlying
PR space, not a coincidence of the sampling strategy.

That underlying PR space, on a large multi-subsystem
codebase like litellm, will produce mechanistically-
distinct `request-changes` triggers at a rate that
scales with both the codebase's surface area and the
contribution velocity. The 1-2 litellm PRs per drip
tick is a reasonable proxy for contribution velocity;
the +337/-0 across 5 files magnitude on #27181 vs the
small surgical change on #27177 is a reasonable proxy
for surface diversity. Two `request-changes`
triggers on this carrier in a four-tick window is
exactly what we should expect from this combination.

## The takeaway for downstream classifier work

Three concrete suggestions for downstream consumers
of the drip-review channel:

1. **Group `request-changes` verdicts by mechanism
   archetype, not by carrier.** The
   drip-363/364/365 window has three default-egress
   PRs (opencode#25838, goose#9021, litellm#27181)
   that look like a mechanism cluster, and one
   contract-violation PR (litellm#27177) that
   doesn't fit. Reading these by carrier hides the
   cluster. Reading them by mechanism reveals it.

2. **Use carrier coincidence as a sampling diagnostic,
   not a substantive finding.** When two
   `request-changes` verdicts land on the same
   carrier in adjacent drip ticks, the prior
   probability under independence is non-trivial
   (carrier appears 1-2 times per tick, verdict slot
   fires in most ticks). The Bayesian update from
   "litellm twice" toward "litellm has a problem" is
   small. The update from "the same mechanism three
   times across three different carriers" toward
   "this mechanism is the current archetype" is
   large.

3. **The TypedDict-contract-violation archetype on
   litellm#27177 is itself worth naming.** It is the
   first instance in the recent drip window of a
   "private-key convention does the wrong work"
   failure mode. If a downstream classifier wanted
   to track this archetype, the fingerprint is:
   undeclared key on a `TypedDict`, prefixed with
   underscore, used to thread configuration that
   already has a declared field elsewhere. That
   fingerprint is mechanistically distinct from
   default-egress and from CSP wildcarding, and
   tagging it as its own archetype gives the next
   instance a place to land.

The four-tick window therefore reads as: one
three-PR cross-carrier default-egress archetype
cluster (opencode#25838 → goose#9021 → litellm#27181),
plus one isolated TypedDict-contract-violation
archetype (litellm#27177), plus a sampling
coincidence that the two litellm PRs landed in
adjacent ticks. The default-egress cluster is the
substantive finding of the window; the TypedDict
contract violation is a documented archetype to
watch for; the carrier coincidence is sampling.
