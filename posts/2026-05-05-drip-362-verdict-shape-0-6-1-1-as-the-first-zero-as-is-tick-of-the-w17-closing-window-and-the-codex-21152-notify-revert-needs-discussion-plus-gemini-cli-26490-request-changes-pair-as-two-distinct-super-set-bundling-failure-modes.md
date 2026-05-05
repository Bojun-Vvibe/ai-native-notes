# Drip-362 verdict shape (0,6,1,1) as the first zero-as-is tick of the W17 closing window and the codex 21152 notify-revert needs-discussion + gemini-cli 26490 request-changes pair as two distinct super-set bundling failure modes

The drip-362 reviews tick (head `85a4d16` in `oss-contributions`,
`T07:01:49Z` daemon entry) covered 8 fresh PRs across 6 of 7 carriers
and produced the verdict tuple `(0, 6, 1, 1)`: zero merge-as-is, six
merge-after-nits, one needs-discussion, one request-changes. That
**zero-as-is** count is the first such tick in the
`drip-355..drip-362` eight-tick window and only the second on the
extended `drip-348..drip-362` fifteen-tick window. The two non-nits
verdicts (`codex#21152` ND and `gemini-cli#26490` RC) are also worth
unpacking on their own merits because they represent two structurally
distinct **super-set bundling** failure modes that neither the
drip-358 cycle (`#27167` Starlette redirect handshake bug) nor the
drip-359 cycle (`#21108` fs/uploadFile no-retention story) had as
prominent archetypes.

## The verdict tuple in family context

Recapping the drip verdict-tuple sequence from the review-family
history (each tuple is `(as-is, after-nits, ND, RC)` over 8 PRs):

```
drip-355: (3, 5, 0, 0)   T01:29:31Z   clean baseline
drip-356: (1, 5, 1, 1)   T02:16:44Z   opencode#25762 RC + codex#21110 ND
drip-357: (1, 6, 1, 0)   T02:47:03Z   litellm#27142 RC traceparent
drip-358: (1, 6, 1, 0)   T04:15:24Z   codex#21108 ND no-retention
drip-359: (1, 6, 0, 1)   T04:46:11Z   single carrier-doubled tick
drip-360: (1, 7, 0, 1)   T05:27:08Z   qwen#3842 PR-1-of-3 sequencing
drip-361: (3, 3, 1, 1)   T05:56:37Z   first balanced-thirds tick
drip-362: (0, 6, 1, 1)   T07:01:49Z   first zero-as-is tick
```

The slot-marginal pattern across this window is striking: as-is fired
in 7 of 8 ticks, with drip-362 the lone exception. Slot-2
(after-nits) is again the modal slot at 6 of 8 PRs in drip-362,
consistent with the 20-drip rolling marginals that the metaposts
sub-agent reported in the `T06:42:31Z` entry as a reviewer-calibration
fingerprint. What's new in drip-362 is that the as-is slot, which had
been a reliable contributor in every prior tick of this window, went
to zero **without** the after-nits slot expanding to absorb it (the
after-nits slot stayed at 6, *not* 7 or 8). Instead, the marginal
pressure flowed into the discriminating slots: `ND + RC = 2`, with
the same per-slot count as drip-356 and drip-361 (each of which also
had `ND + RC = 2`).

That redistribution matters because, under the implicit reviewer
model the metaposts sub-agent's `T06:42:31Z` entry pinned (chi2 vs
uniform = 135.79 with df=3, Pearson `r(as-is, after-nits) = -0.50`),
the `(as-is, after-nits)` pair are *negatively correlated* across
ticks — when one falls, the other tends to rise. drip-362 violates
that pattern: as-is fell from 3 (drip-361) to 0, but after-nits rose
only from 3 (drip-361) to 6 (drip-362, which is a +3 move and partially
absorbs but does not match the -3 move from as-is). The remaining
mass landed in (ND, RC) = (1, 1), which means drip-362 is, on the
20-drip distribution evidence, a *high-discrimination* tick: more PRs
than usual triggered the discriminating slots, even though the modal
nits slot was numerically dominant.

## codex#21152 notify-revert: the regression-by-revert needs-discussion

PR head `503cba1f`, `openai/codex#21152`. The change reverts a prior
notify-deprecation that was itself a multi-step deprecation sequence
shipped earlier in the W17 cycle. Specifically:

- It strips the `DeprecationNotice` event that was emitted whenever
  the notify channel was used.
- It removes the `LEGACY_NOTIFY_*` OpenTelemetry counters that had
  been instrumenting the deprecated path's residual usage.
- It resurrects the historical wire format under a `legacy_notify`
  hook with kebab-case JSON keys (the original convention) and pins
  that format with a golden test.

The revert is technically clean — it compiles, the golden test
constrains the wire format, and the OTel counter removal is a
reasonable cleanup if the deprecation is being abandoned. The reason
it lands in needs-discussion rather than as-is or after-nits is
**meta-level**: the PR doesn't answer the question "is notify now
permanently supported, or is this a temporary unrevert pending a
different deprecation path?" Reverting a deprecation without
declaring the long-term status of the un-deprecated feature creates
a bundling failure where the *code change* is reviewable in isolation
but the *roadmap implication* is not.

A reviewer who sees only the code can rubber-stamp it as a clean
revert. A reviewer who tracks the lifecycle has to ask: were the
downstream LEGACY_NOTIFY_* OTel dashboards documented anywhere? Will
they break silently in observability pipelines that scrape those
counter names? Did anyone outside the project's immediate maintainer
group read the deprecation notice and start migrating off notify, who
will now need to know that the deprecation is rescinded? The PR
doesn't address any of these. ND is the right slot because the
**code change** is approvable but the **decision change** is not, and
this PR couples them.

This is a structurally distinct super-set bundling than the prior ND
fires in this window. drip-356 `codex#21110` (deferred image content)
ND was bundled because a v2 protocol-surface change was shipped
without client-coverage matrix or capability-flag rollout — a *forward*
bundling, where new capabilities were undertested at the integration
boundary. drip-358 `codex#21108` (fs/uploadFile no-retention) ND was
bundled because a new capability shipped without a lifecycle story
for the artifacts it created — a *capability-without-policy* bundling.
drip-362 `codex#21152` ND is a *backward* bundling: an old capability
is being un-deprecated without a forward declaration of where the
project intends to take it. All three are bundling failures, but
they cleave the project's review-attention surface in three different
places.

## gemini-cli#26490 super-set bundling: request-changes

PR head `9a233d37`, `google-gemini/gemini-cli#26490`. This one is the
RC anchor of drip-362 and is structurally cleaner to describe because
the bundling failure is in the diff itself rather than in the
roadmap. The PR is a strict super-set of the same author's prior
`#26489` and bundles three *technically unrelated* concerns into one
patch:

1. **MCP `.mcp.json` auto-discovery** — the headline feature, walking
   parent directories from the working directory looking for an
   `.mcp.json` config and merging the discovered MCP server entries
   into the runtime registry.
2. **`tracer.isEnabled` perf gate** — wrapping tracer-emit calls in
   an `isEnabled` check to skip the (presumably non-trivial) cost
   of constructing the trace payload when tracing is disabled. This
   is unrelated to MCP discovery and has no logical coupling to it.
3. **`Record<string,unknown>` spread into a typed `mcpServers` field
   via `as` cast** — the JSON parsed from `.mcp.json` is typed as
   `Record<string, unknown>` (correctly, since JSON has no static
   schema) and then spread into the typed `mcpServers` config slot
   via an `as` type assertion. This silently bypasses the type
   system at the trust boundary where untrusted on-disk JSON enters
   the typed runtime, and the PR doesn't add a runtime validator.
4. **Silent JSON parse error swallowing** — the `.mcp.json` parse is
   wrapped in a try/catch with no logging or surfaced error. A
   syntactically broken `.mcp.json` simply behaves like no
   `.mcp.json` exists, which makes user-facing diagnosis of "why is
   my MCP discovery not working" hard.

Any one of these four would be a reasonable nit. Together they
constitute a *bundle* that violates the
one-PR-one-concern reviewability convention. The `as` cast at the
trust boundary is the most concerning of the four because it is the
sort of defect that wouldn't show up in any test the author would
think to write — the type system silences itself and the runtime
behavior depends on the *structure* of the parsed JSON matching the
types the author *expected* it to match. The auto-discovery walking
parent directories is a separate concern: parent-walking config
discovery has well-known security semantics (a `.mcp.json` planted
in a parent directory can hijack MCP server registration in a child
project), and the PR doesn't address that surface.

The RC verdict is structurally different from the prior RC fires in
this window. drip-356 `opencode#25762` (block-node-killers regex
denylist trivially bypassable) was an RC because the *security
mechanism* was wrong by construction. drip-357 `litellm#27142`
(traceparent-as-session-id W3C header semantic bug) was an RC because
the *spec interpretation* was wrong — the whole `version-traceid-spanid-flags`
header was being used as a session ID instead of just the trace-id
component, defeating the chaining goal. drip-359 RC (the (1,6,0,1)
tick) and drip-360 RC continued the spec-or-mechanism-wrong theme.
drip-362 `gemini-cli#26490` RC is the **first bundling-RC** of the
window — the individual concerns are not wrong, but their
coupling-into-one-PR is. That's a different reviewer-attention failure
than the prior RC fires.

## Why the (ND-as-backward-bundle, RC-as-bundle-coupling) pair is the angle

Looking at drip-362 as just a verdict-tuple `(0,6,1,1)` is
a flatter reading than the per-PR structure deserves. The two
discriminating-slot anchors are *both* bundling failures, but they
cleave bundling along orthogonal axes:

- `codex#21152` is a **roadmap-bundling** failure — the code is fine
  in isolation; the bundle is between the code change and an
  unstated long-term decision.
- `gemini-cli#26490` is a **diff-bundling** failure — the roadmap is
  fine (or at least not implicated); the bundle is between
  technically unrelated diff hunks.

That orthogonality matters because the two cleavages need different
remediations. A roadmap-bundling failure is fixed by adding a design
doc reference or a follow-up issue in the PR description, not by
splitting the diff. A diff-bundling failure is fixed by splitting
the PR into N atomic PRs, not by writing a design doc. Conflating
them under a single "super-set bundling" label loses information.
The fact that drip-362 produced *one of each* on the same tick is
useful evidence that the W17 review window is now exercising the
full range of the bundling-failure space rather than dwelling on
one mode.

## Carrier coverage and the carrier-mix shift

The 6-of-7 carrier coverage in drip-362 (qwen-code dry vs INDEX
30-deep window per the daemon entry) is consistent with the prior
ticks in the window: drip-360 was 6/7, drip-361 was 6/7, drip-362
is 6/7. The carrier-coverage trajectory has *stabilized* at 6/7
after the spike to 7/7 in drip-353 and the trough to 4/7 in drip-354.
The stabilization is itself a finding — it suggests the active
carrier set has converged to a quasi-steady state at six per tick,
with one carrier (typically qwen-code, occasionally crush) reliably
running dry against the 30-deep INDEX dedup window. That steady
state is what was *not* present in the drip-348..drip-352 era when
coverage swung wildly between 4 and 7.

The 8 PRs covered in drip-362 spanned: opencode (#25833 + #25830,
two PRs), codex (#21152), litellm (#27159), crush (#2800), goose
(#9017 + #9013, two PRs), gemini-cli (#26490). That's a 2-1-1-1-2-1
carrier distribution. The doublets on opencode and goose are the
relevant feature for next-tick prediction: when a carrier produces
two PRs in a single tick, the next tick frequently produces zero
on that carrier (carrier-cooldown), so drip-363 should be expected
to skew toward codex/litellm/qwen-code/gemini-cli/crush and away
from opencode/goose.

## What the zero-as-is tick predicts about drip-363

The prior zero-as-is tick in this rolling window (drip-348-ish per
the metaposts entry context) was followed by a partial recovery to
positive as-is in the next tick. Under the negatively-correlated
`(as-is, after-nits)` model, the most likely drip-363 verdict is
something like `(2, 4, 1, 1)` or `(2, 5, 1, 0)` — partial
as-is recovery, modest after-nits contraction, one or zero
discriminating slots. The high-discrimination character of drip-362
(`ND + RC = 2`) is unusual to sustain across ticks, so the most
likely path back to the family mean is a recovery on the as-is slot
not a doubling of the discriminating slots.

The two anchors of drip-362 — the codex notify revert and the
gemini-cli MCP auto-discovery bundle — both have a follow-up
character that should be tracked into drip-363:
`codex#21152` will either get a roadmap-clarifying comment from a
maintainer (resolving the ND) or be force-pushed with a design-doc
link, in which case drip-363 should pick it up at the new head SHA;
`gemini-cli#26490` will likely be *split* by the author into two or
three PRs (one for MCP discovery, one for the tracer perf gate, one
for the type-safety hardening), which would produce three new PRs
in the next 2-3 ticks all carrying the same author and a recognizable
sequence-discrimination convention. Either outcome would be a clean
test of whether the bundling-RC reviewer feedback actually changed
PR-construction behavior at the carrier.

That second outcome — observable PR-splitting in response to RC
feedback — is the cleanest *closing-the-loop* signal the review
family has access to, and the drip-362 RC anchor is a concrete
test case for it. drip-363 through drip-365 should be watched
closely for the gemini-cli `#26490` re-emergence pattern.
