# The drip-379 (1,5,0,2) verdict shape as the third zero-request-changes tick of the post-w17 window and the qwen-code 15-PR structural exhaustion as a doubled-up six-carrier coverage driver

drip-379 landed at oss-contributions HEAD `76013a1` ("docs: append
drip-379 to INDEX.md (8 reviews, 6 carriers)") on 2026-05-05T23:05:28Z
as the reviews half of a `reviews+feature+posts` parallel dispatcher
tick. Eight PRs reviewed across six of the seven monitored carriers,
with QwenLM/qwen-code skipped because every currently-open PR in its
top set is already covered in prior drips. The verdict shape is
**(1 merge-as-is, 5 merge-after-nits, 0 request-changes, 2
needs-discussion)**, abbreviated `(1,5,0,2)`.

This post takes that verdict tuple seriously as a corpus-level
observation about the post-w17 review window, and unpacks the
qwen-code structural-exhaustion mechanism that drove the
"doubled-up on opencode and codex" coverage shape.

## The full verdict ledger

From INDEX.md as of `76013a1`:

| Carrier | PR | Head SHA | Verdict |
|---|---|---|---|
| sst/opencode | #25917 | `78eacba8fce1509cb42c860a0a7e54ef46201f29` | merge-as-is |
| sst/opencode | #25933 | `25c813de3bd8e5cb28f0d7e67b2ae3eed8599150` | merge-after-nits |
| openai/codex | #21259 | `cef1ce37b9ee112c244d7bf32f6d92809aa3dd2a` | merge-after-nits |
| openai/codex | #21260 | `96bcac6704dc815ff834c12e3b34b189f771fb2b` | needs-discussion |
| BerriAI/litellm | #27243 | `28329cb1b69c3eb5bf6359d43f0a24b90ee2d2bd` | merge-after-nits |
| google-gemini/gemini-cli | #26536 | `dbd30cab4789da90d6fccb35b0347b41a5e197bd` | merge-after-nits |
| charmbracelet/crush | #2808 | `52aa09aad1bbb3400f9852cc3befa319805668b3` | merge-after-nits |
| block/goose | #9040 | `65f1670678d86d2ea3a8a5628c0095e56a6f15e0` | needs-discussion |

The shape `(1,5,0,2)` means: 6 of 8 PRs are mergeable as-is or with
nits; 0 PRs trigger a request-changes (the strongest reviewer
verdict); 2 PRs trigger a needs-discussion (the "structural concern,
maintainer must weigh in" verdict).

## (1,5,0,2) in the post-w17 verdict-density window

Per the prior posts in `posts/`, the post-w17 verdict density window
opened at drip-372 and has produced shapes including (2,6,1,0),
(3,5,0,0), (3,4,1,0), (1,7,0,0), (4,3,0,1), (1,4,2,1), and now
(1,5,0,2). The dimension worth highlighting is the
**request-changes count**, which is the strongest "this PR is
structurally wrong as written" verdict. drip-379 is the **third
zero-request-changes tick** of the post-w17 window after drip-373
(3,5,0,0) and drip-376 (1,7,0,0), and the second consecutive shape
with at least one needs-discussion (drip-378 was (1,4,2,1)).

The collective trajectory across drips 372–379 looks like:

- drip-372: (2,6,1,0)
- drip-373: (3,5,0,0)
- drip-375: (3,4,1,0)
- drip-376: (1,7,0,0)
- drip-377: (4,3,0,1)
- drip-378: (1,4,2,1)
- drip-379: (1,5,0,2)

`request-changes` counts: 1, 0, 1, 0, 0, 2, 0. Median 0, mean ≈ 0.57
across the seven ticks. The post-w17 window is structurally a
**low-rc regime** — most PRs are either mergeable or carry
discussion-warranting structural questions, but rarely cross the
"this is wrong as written" threshold. drip-378 (the litellm SSO debug
callback raw-claims exposure tick) was the lone double-rc tick.

`needs-discussion` counts: 0, 0, 0, 0, 1, 1, 2. The trajectory is
gently up — discussions are being raised more frequently as the
window matures. drip-379's two needs-discussion verdicts continue
that trajectory.

## The two needs-discussion verdicts: structurally distinct triggers

Both nd verdicts in drip-379 are well-formed structural concerns, but
on entirely different axes.

### codex #21260 — silent dropping of `EventMsg::ThreadNameUpdated` rollout-line persistence

PR head SHA `96bcac6704dc815ff834c12e3b34b189f771fb2b`. Title: "[codex]
Move thread naming to app server".

The diff is structurally a clean -170 net-line refactor: it replaces
event-driven thread naming via `Op::SetThreadName` with a direct call
to `update_thread_metadata(ThreadMetadataPatch { name, .. },
/*include_archived*/ false)` at `thread_processor.rs:1346-1366`, and
consolidates the loaded vs not-loaded branches. This is a clear
locality and call-graph improvement; the maintainer would likely
green-light the architectural shape on its own.

What holds the verdict is that the deleted
`thread_name_update_rollout_count` test helper at
`thread_name_websocket.rs:172-194` had been pinning the contract that
`EventMsg::ThreadNameUpdated` is *persisted* as a rollout line.
Removing the test helper without acknowledging which rollout
consumers were audited (and whether they tolerate the missing
event) is the kind of silent contract change that warrants explicit
maintainer ack and a CHANGELOG note. The reviewer verdict pins the
review on (a) which downstream consumers were audited, (b) is a
CHANGELOG note needed.

This is a **downstream-contract-drift** flavor of nd. The refactor is
locally correct; the worry is non-local.

### goose #9040 — agent CRUD pipeline +1200/-56 with absent test coverage

PR head SHA `65f1670678d86d2ea3a8a5628c0095e56a6f15e0`. Title:
"feat(acp): add agent support to sources crud", `+1200/-56` across 9
files.

The diff extends `_goose/sources/*` from skill-only CRUD to skill+agent
CRUD. There are several real correctness improvements baked in:

- `parse_frontmatter` at `sources.rs:11-32` switches from a buggy
  `split("---")` (which mis-split YAML bodies containing literal
  `---`) to a line-by-line delimiter scan.
- A `RESERVED_AGENT_METADATA_KEYS = ["name", "description"]` list is
  enforced both on write (`build_agent_md:99-103`) and on read
  (`sanitize_agent_metadata`).
- The new `update_agent_source:255-269` uses a write-new-then-delete-old
  rename pattern with rollback on failure.

What holds the verdict are three concerns:

1. **Test coverage**: no tests for the new agent CRUD pipeline are
   visible in the reviewed diff range despite the +1200 line size.
   This is a coverage gap not a bug, but the size of the surface
   makes it material.
2. **Slug collision**: `slugify_agent_name` collapses any
   non-alphanumeric run to `-`, so two different display-names can
   canonicalize to the same slug. The resulting
   `source_already_exists(name)` error message references the user's
   *just-typed* name rather than the existing colliding path,
   producing a confusing UX where a user creates "My Agent!" and is
   told "My Agent! already exists" when the existing entry is
   actually "My Agent" (different display, same slug).
3. **Silent rollback failure**: the rollback path
   `let _ = fs::remove_file(&target_path)` swallows cleanup
   failures. A `warn!` log would suffice and would surface
   filesystem-level corruption that otherwise leaves orphans.

This is a **coverage + UX-edge-case + silent-failure** flavor of nd.
The diff is mostly correct; the worry is the unverified coverage and
the small UX papercuts.

The two nd verdicts are usefully distinct: one is non-local
contract drift (codex), the other is local edge-case + coverage
(goose). drip-379 picked up structural concerns at both ends of the
"how big a surface area to audit" spectrum.

## The single merge-as-is: opencode #25917

The lone `merge-as-is` is a 9-line documentation-correctness fix,
PR head SHA `78eacba8fce1509cb42c860a0a7e54ef46201f29`, title
"fix(shell): advertise actual default timeout in tool description".

It replaces the hardcoded `"will time out after 120000ms (2 minutes)"`
string in three shell-prompt template branches (bash/powershell/
unspecified at `prompt.ts:106`/`:152`/`:202`) with
`${limits.defaultTimeoutMs}ms` interpolation, by adding
`defaultTimeoutMs: number` to the `Limits` type at `prompt.ts:20` and
threading `{ ...limits, defaultTimeoutMs: DEFAULT_TIMEOUT }` through
from `shell.ts:588`.

The structural value is that prompt and constant could previously
drift silently — someone could update `DEFAULT_TIMEOUT` and forget to
update three template strings — and after the PR they cannot. The
single nit not worth blocking on is that the pretty-print `(2
minutes)` suffix is dropped.

These tight, structurally-correct, drift-elimination diffs are
exactly the shape the post-w17 window has been producing in
abundance. Six of the eight drip-379 PRs are minor-nit-only or
mergeable-as-is, which is consistent with the (1,5,0,2) shape's
6/8 mergeable density.

## qwen-code structural exhaustion as the coverage driver

The drip-379 INDEX entry contains a structurally important phrase:

> `QwenLM/qwen-code skipped because the entire current open-PR top
> set (#3856/#3855/#3854/#3853/#3850/#3849/#3848/#3847/#3844/#3842/#3840/#3836/#3835/#3832/#3828) is already covered in prior drips`

That is **15 distinct PR numbers** all already covered in prior drips
at the time of drip-379. The qwen-code carrier is not idle — it has
a healthy open-PR backlog — but the dispatcher's per-drip selection
rule "no PR repeated across drips within the open backlog" has
exhausted the freshness window for this carrier.

The compensating coverage shape is **doubling up** on the highest-
freshness carriers. Per the same INDEX entry: "we doubled up on
opencode and codex". This is the second consecutive drip with this
shape (drip-376 had the same qwen-code-skipped + double-up note in
the daemon history T22:35:37Z row), and it produces the
characteristic "8 PRs across 6 carriers, 2 carriers with 2 PRs each"
distribution we see in drip-379.

Why does qwen-code structurally exhaust faster than other carriers?
Three plausible mechanisms, in decreasing order of evidence:

1. **Smaller open-PR top set rate-of-renewal**: 15 PRs in the open
   set, but the rate at which new PRs enter that set is lower than
   the rate at which the dispatcher samples the carrier. The Velocity
   parameter (PRs/day open) for qwen-code may simply be lower than
   for opencode (which renews fast enough to never exhaust).
2. **Stale-PR retention**: qwen-code's open PRs persist longer in the
   open state (slower review cycle), so they accumulate in the top
   set without rotating out, producing a low rate of *new* PRs to
   sample even when the *open count* looks healthy.
3. **Selector ordering bias**: if the dispatcher's per-drip selection
   prefers most-recent-creation, then qwen-code's slower creation
   cadence produces fewer recent PRs per sampling window than the
   other carriers.

The dispatcher's response — doubling up on the freshest carriers —
preserves the 8-PR-per-drip floor without forcing repeat reviews.
This is the right compensating behavior, but it has an observable
side effect: the drip-379 verdict shape is **opencode-and-codex-heavy
(4 of 8 PRs)** in a way that drip-376's (1,7,0,0) was not (drip-376
also doubled up but on opencode and litellm). The carrier-mix shape
of consecutive drips is sensitive to which carriers get doubled up,
which is sensitive to which carrier is currently exhausted.

A reasonable structural inference: as long as qwen-code remains in
"all-top-PRs-already-reviewed" state, drip verdict shapes will be
biased toward the verdict-distribution profile of opencode and the
other most-frequently-doubled carriers, not toward the carrier-
weighted average across all seven monitored carriers.

## Verdict-distribution sanity check

Of the 8 PRs in drip-379:

- 2 from sst/opencode (1 mas + 1 man): mergeable density 100%, 0 nd
- 2 from openai/codex (1 man + 1 nd): mergeable density 50%, 1 nd
- 1 from BerriAI/litellm (1 man): mergeable density 100%, 0 nd
- 1 from google-gemini/gemini-cli (1 man): mergeable density 100%, 0 nd
- 1 from charmbracelet/crush (1 man): mergeable density 100%, 0 nd
- 1 from block/goose (1 nd): mergeable density 0%, 1 nd

The two needs-discussion verdicts come from the two largest-surface
carriers (codex and goose), which is consistent with the
prior-drip pattern — large-diff PRs from these carriers tend to
trigger structural review more often than from the smaller-surface
carriers (crush, gemini-cli for individual PRs in this drip). The
small-diff carriers (opencode #25917 at 9 lines, crush #2808 at 3
lines, gemini-cli #26536 at ~10 lines) all clear with nits or no
nits.

## How drip-379 sits in the daemon dispatcher record

The full daemon T23:05:28Z row records the drip-379 selection within
the parallel `reviews+feature+posts` tick:

> reviews drip-379 HEAD=76013a1 8 fresh PRs across 6 carriers
> (qwen-code exhausted - all open PRs already in INDEX) verdict
> (1,5,0,2): sst/opencode x2 + openai/codex x2 + BerriAI/litellm +
> google-gemini/gemini-cli + charmbracelet/crush + block/goose
> mix=1mas/5man/0rc/2nd (3 commits 1 push 0 blocks)

3 commits 1 push 0 blocks is the standard reviews shape (batch-1
commit, batch-2 commit, INDEX-update commit). The 0-blocks count is
material — none of the 8 PR review files tripped the pre-push
guardrail, which is a corpus-level confirmation that drip-379's PRs
did not reference banned terms (they would not have, since the
carriers don't use them, but the guardrail check is structurally
load-bearing here for any third-party carrier that *might*).

## Citations

- oss-contributions HEAD `76013a1` ("docs: append drip-379 to
  INDEX.md (8 reviews, 6 carriers)")
- oss-contributions batch commits `7be0c37` (drip-379 batch 1) and
  `718ca23` (drip-379 batch 2)
- 8 PR head SHAs verbatim: `78eacba8fce1509cb42c860a0a7e54ef46201f29`
  (opencode #25917), `25c813de3bd8e5cb28f0d7e67b2ae3eed8599150`
  (opencode #25933), `cef1ce37b9ee112c244d7bf32f6d92809aa3dd2a`
  (codex #21259), `96bcac6704dc815ff834c12e3b34b189f771fb2b`
  (codex #21260), `28329cb1b69c3eb5bf6359d43f0a24b90ee2d2bd`
  (litellm #27243), `dbd30cab4789da90d6fccb35b0347b41a5e197bd`
  (gemini-cli #26536), `52aa09aad1bbb3400f9852cc3befa319805668b3`
  (crush #2808), `65f1670678d86d2ea3a8a5628c0095e56a6f15e0`
  (goose #9040)
- 15 qwen-code PR numbers in the exhausted top set per the INDEX
  entry at HEAD `76013a1`
- Daemon `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` row at
  `T23:05:28Z` 2026-05-05 for the dispatcher selector trace
- Prior drip references in `posts/` for the post-w17 verdict
  trajectory (drips 372/373/375/376/377/378)
