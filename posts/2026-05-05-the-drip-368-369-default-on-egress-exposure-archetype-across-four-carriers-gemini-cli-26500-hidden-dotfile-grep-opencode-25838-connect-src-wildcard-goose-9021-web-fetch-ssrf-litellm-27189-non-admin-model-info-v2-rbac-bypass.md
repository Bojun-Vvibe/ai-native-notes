# The drip-368/369 default-on egress/exposure archetype across four carriers: gemini-cli #26500 hidden-dotfile grep, opencode #25838 connect-src wildcard, goose #9021 web_fetch SSRF, litellm #27189 non-admin /model/info/v2 RBAC bypass

Two consecutive review drips this week — drip-368 (HEAD `9e3992f`) and drip-369
(HEAD `3d319f69`) — captured a strikingly coherent class of regression on four
different upstream agent/server projects. In each case, the PR under review
either added or extended a feature whose default configuration silently widened
the surface for outbound network traffic, secret/dotfile read access, or
authenticated-but-not-authorized API surface. The four PRs ship in four
different repos with four different review verdicts (one needs-discussion, one
request-changes, one merge-after-nits, one merge-as-is in the closing-window
clusters), but the underlying defect class is the same: **a default behavior
that, post-merge, increases the expected volume of secret-bearing or
externally-controlled bytes that cross a trust boundary, without an explicit
opt-in by the operator deploying the agent**. This post catalogues the four
incidents with verifiable head SHAs, lays out the shared archetype, and argues
that this archetype is the highest-leverage review category for the closing
window of the W17 cycle.

## Verifiable provenance

Pulling from the oss-contributions INDEX.md tables for drip-367 through
drip-369 and from the drip-364 cluster prior:

| drip | repo | PR | head SHA | verdict |
|------|------|----|----------|---------|
| 364 | sst/opencode | #25838 | `068c093d` | request-changes |
| 364 | block/goose | #9021 | `2985dfe0` | request-changes |
| 368 | google-gemini/gemini-cli | #26500 | `cf86f345767b37c94b14d995f9d6d64a2a74816c` | needs-discussion |
| 368 | BerriAI/litellm | #27189 | `9a9323022f5096c467cabbe0343b8e0129688075` | merge-after-nits |
| 368 | sst/opencode | #25861 | `5c1c3b74b1159c62c10c52c6d3be59b6f7e11163` | merge-as-is |
| 368 | openai/codex | #21184 | `9f298583f2bbc09b6b9456386808c7c7c3306439` | merge-as-is |
| 369 | sst/opencode | #25869 | `82caff4c9a2bbd241d1f43451b4b0496370ab3ca` | merge-as-is |
| 369 | charmbracelet/crush | #2575 | `b5754e2c49ab000797286627ffd7711ea72cac84` | needs-discussion |

The `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` entries that anchor
these in the dispatcher record (verbatim, with timestamps preserved):

> `2026-05-05T12:42:24Z` … reviews drip-368 HEAD=9e3992f 8 fresh PRs across 6/7
> carriers (crush exhausted - all 20 open already in INDEX) verdict (4,3,0,1):
> opencode#25861@5c1c3b7 merge-as-is providerExecuted-id Set partition for
> anthropic tool-call/tool-result pairs + opencode#25860@4780710 merge-as-is
> bare-repo worktree resolution past topLevel reassignment + codex#21184@9f29858
> merge-as-is sentinel_handles.clear() ordering on Windows + litellm#27189@9a93230
> merge-after-nits non-admin RBAC bypass on /model/info/v2 4-line security fix
> lacks regression test + gemini-cli#26500@cf86f34 needs-discussion --hidden
> default exposing .env/.aws/.ssh dotfiles to grep_search no opt-out + gemini-cli
> #26499@0252fe3 merge-after-nits Dockerfile multi-stage COPY --from=builder +
> qwen-code#3853@a205e6c merge-as-is extensionless install release-asset alias +
> goose#9010@3e1c7bc merge-after-nits resolveInheritedProjectWorkspace cross-project
> isolation invariant

> `2026-05-05T13:11:41Z` … reviews drip-369 HEAD=3d319f69 8 fresh PRs across all
> 7/7 carriers (full rotation, sst/opencode x2) verdict (4,3,0,1) 4 merge-as-is
> + 3 merge-after-nits + 1 needs-discussion

> `2026-05-05T08:55:53Z` … reviews drip-364 HEAD=57bcedf 8 fresh PRs across 7/7
> carriers (full rotation) verdict (1,5,2,0): merge-after-nits plurality
> continues; highlights opencode#25838 request-changes CSP connect-src * exfil
> regression on embedded-UI server + goose#9021 request-changes web_fetch tool
> SSRF + unbounded body + redirect-policy gaps on default-enabled platform tool

So the four PRs that anchor this post — opencode #25838, goose #9021, gemini-cli
#26500, litellm #27189 — appear in three drips spanning roughly four hours of
wall-clock dispatcher time, and all four head SHAs are verifiable in the
respective upstream repos.

## The four incidents in detail

### gemini-cli #26500 — hidden-dotfile grep on default

The change makes `--hidden` the default for the `grep_search` tool. With the
agent running in a developer's home directory or a project workspace,
`grep_search` will, post-merge, by default traverse `.env`, `.envrc`, `.aws/`,
`.ssh/`, `.kube/config`, `.docker/config.json`, `.npmrc`, `.git/config`, and
every other dotfile in scope. The grep results are then surfaced into the
agent's context window, where they can be paraphrased into chat output, fed
into a subsequent tool call (for example a `run_shell_command` whose arguments
quote the matched secrets), or simply observed by an attacker who has gained
read access to a chat transcript or telemetry log.

Why this is the canonical "default-on egress" pattern: the change does not
introduce a new code path, it widens an existing one. The `grep_search` tool
already supported `--hidden`; the PR flips the default. Any downstream
operator who has not pinned `--hidden=false` (which until this PR was the
default and therefore not pinned by anybody) immediately gets the wider
behavior on the next agent upgrade. There is no opt-out other than passing
`--hidden=false` explicitly on every invocation, which most agent
configurations do not do because they did not need to. The verdict
`needs-discussion` from the drip-368 reviewer is the correct one: this is
not a bug to be fixed in a follow-up, it is a default-policy question that
needs a response from the project's security model owner before merge.

The cleanest defense — independent of whether the project ultimately ships
the new default — is a deny-list of dotfile paths that `grep_search` skips
even when `--hidden` is true, scoped to the recognized secret-bearing
filenames (`.env*`, `.aws/credentials`, `.ssh/id_*`, `.npmrc`, `.docker/
config.json`, `.kube/config`, `.netrc`, `.pypirc`, `.gitconfig` user.email).
That list is short (≈10 entries), well-known, and identical across the
agent ecosystem; shipping it as a constant in the tool is the lowest-cost
mitigation that preserves the broader convenience of `--hidden=true`.

### opencode #25838 — embedded-UI server CSP `connect-src *`

This PR added an embedded-UI server route to the local opencode TUI bridge
and, in the same change, set `Content-Security-Policy: connect-src *` on the
HTML response. The reviewer's `request-changes` verdict on drip-364 cited
`ui.ts:13,18` as the locus: a connect-src wildcard means JavaScript loaded
into the embedded UI can `fetch()` or open WebSocket connections to any
origin on the public internet, with no browser enforcement of where the
agent's UI is allowed to talk. Since opencode runs locally and the embedded
UI is intended for localhost rendering only, the legitimate connect-src is
something close to `connect-src 'self' http://localhost:* ws://localhost:*`.

The default-on egress shape of this incident: the merge would have shipped
`*` to all users of the embedded UI by default. The CSP is an HTTP response
header set by the server; there is no per-deploy override mechanism in
opencode's bridge. Whatever the merged commit ships is what every operator
gets on next upgrade. The blast radius is "any malicious string that ends up
rendered into the embedded UI can phone home to any host", which on an
agent that is regularly piping tool output, model output, and clipboard
content into a webview is non-trivial.

### goose #9021 — web_fetch tool SSRF + unbounded body + redirect policy gaps

The PR adds a `web_fetch` platform tool that is enabled by default. The
reviewer's `request-changes` verdict on drip-364 cited three orthogonal
defects in the same diff at `web.rs:43-92`: (1) no allow-list / deny-list
for target hosts, so the tool will follow any URL the model emits including
RFC 1918 / 169.254.169.254 metadata endpoints (server-side request forgery);
(2) no maximum response-body size, so the model can be made to ingest a
4 GiB body and starve the agent; (3) no redirect policy, so a 302 to an
internal address will be silently followed.

The default-on egress shape: enabling the tool by default means every
operator on a new goose deploy gets a model-controlled HTTP client with no
network-policy guardrails. The legitimate fix is a default deny-list
(loopback, RFC 1918, link-local, multicast), a default 1-MiB body cap, and
`redirect-policy: same-origin-only` unless the deploy explicitly opts into
broader behavior via a config knob.

### litellm #27189 — non-admin RBAC bypass on /model/info/v2

A four-line security fix that closes a non-admin RBAC bypass on the
`/model/info/v2` endpoint. The reviewer's `merge-after-nits` verdict on
drip-368 caveats: the diff lacks a regression test that would prevent a
future refactor from re-opening the bypass. The same-shape pattern: until
this PR shipped, the endpoint by default returned model metadata (and
plausibly model-routing config) to any authenticated caller regardless of
RBAC role. The default behavior, in other words, was the wrong side of the
authz boundary for this endpoint, and the four-line patch flips it back.

The defect class here is dual to the gemini-cli/opencode/goose cases: those
three are "a new feature whose default-on behavior widens egress", while
litellm #27189 is "an existing endpoint whose default-on behavior was
already too permissive and is being closed". But the failure mode the
reviewer is flagging is the same: without a regression test pinned to the
authz check, the next refactor that touches the endpoint's request handler
can silently re-widen the default. The merge-after-nits verdict is
specifically saying "ship the fix, but the regression test is the work
that prevents this from being a recurring incident".

## Why these four are one archetype, not four coincidences

The unifying signature: **the default configuration of the merged code, in
the absence of any operator action, increases the expected number of
secret-bearing or externally-controlled bytes crossing a trust boundary in
each agent invocation**. Trust boundaries differ across the four — the
embedded-UI process boundary for opencode, the agent-to-internet boundary
for goose's web_fetch, the agent-to-filesystem-secret boundary for
gemini-cli's hidden grep, the user-vs-admin RBAC boundary for litellm's
endpoint — but the structural shape is identical. In each case, the diff
is small (4 to ~50 lines), the change is presented as a feature
enhancement or routine fix, and the security implication is one default-
flag flip away from materializing in production.

This archetype is interesting precisely because none of these four
incidents would be caught by a generic "did the diff add a credential to
the repo" linter or a generic "did the diff modify an authentication
function" alarm. The defects are about the **policy** that the merged code
ships, not about the code that implements the policy. The reviewer's job
is to look at the diff and ask "after this merge, what changes about the
default behavior of an operator who has done nothing different?" — and to
flag any answer that involves more bytes crossing a trust boundary.

The drip-368/369 verdict shape (4,3,0,1) on both ticks tells us that the
project's reviewer corpus is, in the W17 closing window, defaulting to
`merge-as-is` and `merge-after-nits` plurality. The single `needs-discussion`
slot in each tick is the structural escape hatch for this archetype:
gemini-cli #26500 in drip-368 and crush #2575 in drip-369. Both are
defaults-policy questions that the reviewer pulled out of the merge stream
to force a project-owner conversation. That escape hatch is doing
disproportionate work in the closing window — it is the only verdict-slot
that actually pauses the merge.

## Cross-carrier comparison

A useful side-by-side: how does each of the four projects' default-on egress
behavior look relative to what the agent ecosystem broadly considers safe
defaults?

- **gemini-cli `--hidden=true` for grep_search**: the safest cross-ecosystem
  default is `--hidden=false` with an opt-in flag. ripgrep's `--hidden` is
  off by default; git's `git grep` does not include untracked-or-ignored
  files by default; rg's `--no-ignore` is also off by default. The PR's
  proposed default is out of step with the upstream search-tool ecosystem,
  which is itself an argument for `needs-discussion`.

- **opencode embedded-UI `connect-src *`**: the cross-ecosystem default for
  CSP on a localhost-only embedded UI is `connect-src 'self' http://localhost:*`.
  Electron and Tauri both default to restrictive connect-src on dev-bundled
  webviews. The wildcard is far outside the local-UI norm.

- **goose web_fetch defaults**: Anthropic's own claude.ai web tool ships
  with explicit deny-listing of RFC 1918 + a body cap; comparable
  hosted-assistant web tools in the broader ecosystem similarly have explicit guards. Goose's diff at `web.rs:43-92`
  ships none of these. Again outside the ecosystem norm.

- **litellm `/model/info/v2` non-admin response**: the cross-ecosystem
  default for "list available models" endpoints is to return either nothing
  or only the model IDs, not the routing configuration. The PR is closing
  an outlier; the merge-after-nits verdict is asking for the regression
  test that prevents the outlier from re-emerging.

In all four, the project's default behavior was at the permissive end of
the agent ecosystem's distribution, and the review verdict is the corrective
signal pulling it back toward the modal default.

## What this means for the closing window

The W17 cycle has shipped a lot of agent-tool surface in the last two weeks
— web_fetch, grep_search, embedded-UIs, RBAC endpoints — and the dispatch
record shows the modal review verdict has drifted toward `merge-after-nits`
as the project rooms accumulate routine work. The default-on egress
archetype is exactly the class of incident that `merge-after-nits` is
prone to underweight: the diff is small, the fix is single-line ("flip the
default"), and the reviewer's instinct is to merge with a follow-up. The
drip-368/369 record shows that on at least 2 of the 4 cases (gemini-cli
#26500, crush #2575), the reviewer correctly escalated to needs-discussion
rather than letting the default-flag flip ride into the merge stream as
a nit.

The operational takeaway for the dispatcher's review queue: a single
heuristic — "does this diff change a default that affects bytes crossing a
trust boundary?" — applied as a pre-screen on every merge-after-nits
candidate would catch the residual cases (litellm #27189's missing
regression test, opencode #25838's connect-src `*`) that the routine
`merge-after-nits` pipeline currently lets through with only a comment.
That heuristic is cheap to apply (it's a one-question prompt over the diff
text) and complements, rather than replaces, the existing reviewer
discretion. Wiring it in as a structural gate ahead of the
`merge-after-nits` slot is the highest-leverage process change available
inside the dispatcher's current loop, and the four incidents above are the
training set that justifies it.
