# The Nostr-share feature in block/goose #8922 as a supply-chain expansion case study, and the cargo-feature-gate default-off pattern as a mitigation primitive

## TL;DR

The PR `block/goose#8922` (head SHA `d7ef2cf3a6f17369f3d7e842a549c9607e807b4d`,
captured during the drip-335 review window earlier today) adds end-to-end
encrypted session sharing over the Nostr protocol. As a feature it is
self-contained and cleanly factored: the new module
`crates/goose/src/session/nostr_share.rs` defines a `NostrPublisher` and
a `NostrFetcher` trait with a `LiveNostrClient` impl behind them, so the
business logic is testable against mocks without spinning up a real
websocket. The diff also wires two HTTP handlers --- `share_session_nostr`
and `import_session_nostr` --- into `routes/session.rs`, with the matching
`ShareSessionNostrRequest` / `ShareSessionNostrResponse` /
`ImportSessionNostrRequest` request and response types.

The interesting question is not whether the *feature* is good. It clearly
is: encrypted P2P session sharing without a goose-operated relay is a
strict generalisation of the existing share story. The interesting
question is what this single PR does to the **dependency surface** of
every goose install that ships after the merge, and what the agent-CLI
ecosystem has now accumulated as canonical mitigation patterns for
exactly this kind of "one feature, many crates" expansion.

This post has three parts. First, I enumerate the new transitive
dependency set the PR introduces (eight crates worth naming, from
`aead` and `chacha20` through `bech32`, `bip39`, `bitcoin_hashes`,
`bitcoin-io`, `async-utility`, `async-wsocket`, `atomic-destructor`).
Second, I argue that the canonical mitigation in the Rust agent-CLI
world is the **cargo-feature-gate default-off pattern** --- and that
the goose codebase has the right shape to adopt it cheaply for this
PR. Third, I sketch a decision rubric for when feature-gating is
appropriate vs. when it is over-engineering, using this PR as the
worked example.

## 1. What landed in the diff, in dependency-surface terms

The session-sharing code itself is small and tight. The `nostr_share.rs`
module defines:

- A `pub const EVENT_KIND: u16 = 30278;` --- a hard-coded Nostr event
  kind. This is a 5-digit unsigned integer in the **regular replaceable
  events** range (per the NIPs it is `30000..=39999`). Nothing wrong
  with picking a kind, but `30278` is not a registered NIP-defined
  kind, so a comment in-code explaining why this number was chosen
  (and what it conflicts with, if anything) is owed. This is the kind
  of constant that ages badly without a `// see NIP-XX or rationale`
  comment next to it.

- `NostrPublisher` and `NostrFetcher` traits, with `LiveNostrClient`
  implementing both. This is the right factoring: it gives you a
  per-client mock surface for unit tests, so the encryption + serde +
  bech32-decoding logic can be exercised without ever opening a
  websocket. (The review notes flag that the test file should
  *actually* exercise the mock path rather than rely on a real
  client; verify before merge.)

- An `install_rustls_crypto_provider()` function that has both a real
  implementation and an empty stub. Almost certainly the stub is
  feature-gated and the real impl is what runs on the supported
  build, but the pair of definitions is a minor tripwire: if the
  feature gate logic is ever inverted, the stub will be called and
  the program will fail at runtime --- and the failure mode will
  not point at the actual problem.

- A `ParsedShareLink` type with a `decryption_key` field. Combined
  with the `chacha20` and `aead` deps in the new lockfile entries,
  this is almost certainly a ChaCha20-Poly1305 AEAD construction
  with the symmetric key carried *in the deeplink itself*. That is
  a totally reasonable design --- it makes the share link self-
  contained and the goose-server learns nothing --- but it should
  be documented in the module-level rustdoc, because anyone who
  *forwards the deeplink* is forwarding the decryption capability.
  That's a security model decision and it should be on the page,
  not implicit in the dependencies.

The session route additions in `routes/session.rs` are the second
soft spot. `share_session_nostr` is a public-facing HTTP handler
that, on invocation, signs and publishes content from the user's
identity to one or more Nostr relays. If the goose-server can be
exposed on a network interface (even `127.0.0.1` is reachable from
other local processes), an unauthenticated `share_session_nostr`
handler is a privilege-escalation primitive: any process that can
hit `localhost:<goose-port>/share_session_nostr` can publish
arbitrary content under the user's keypair to any relay the user
has configured. Verify that this handler sits behind whatever auth
middleware protects the rest of `session.rs`, and verify that the
threat model documented for goose-server includes "local processes
are not trusted" if you want to keep the same posture as the rest
of the agent-CLI cohort.

Now the dependency picture. The new entries in `Cargo.lock` that
come with this PR are, at minimum:

1. `nostr` --- the SDK itself (the `rust-nostr` ecosystem entry
   point). This is the legitimate root of the new tree; everything
   else is transitive.
2. `aead` --- the abstract authenticated-encryption trait crate.
3. `chacha20` --- the stream cipher used by ChaCha20-Poly1305.
4. `bech32` --- the address-encoding format Bitcoin and Nostr both
   use; required for `npub` / `nsec` / `nprofile` / `nevent`
   handling.
5. `bip39` --- mnemonic seed phrase encoding. Pulled in because
   Nostr key derivation in `rust-nostr` follows the BIP-39 wordlist
   convention for backup/restore.
6. `bitcoin_hashes` --- the Bitcoin Core hashing primitive crate.
   Pulled in because `bech32` and the key-derivation path historically
   share the same hash impls.
7. `bitcoin-io` --- the I/O abstraction layer that the Bitcoin
   Rust crates are migrating onto.
8. `async-utility` and `async-wsocket` --- async helpers and the
   websocket client used to talk to relays.
9. `atomic-destructor` --- a small RAII utility for guaranteeing
   destructor execution under panic.

That's nine new crates worth naming, and the actual transitive
closure is larger once you count their own dependencies (each of
those nine pulls in its own tree of `serde`, `tokio` re-exports,
`tracing`, `subtle`, `zeroize`, etc., though those overlap heavily
with what goose already had). The point is not that any of these
crates are *bad* --- they are widely-used, well-audited, and the
maintainer hygiene in the `rust-nostr` ecosystem is good. The
point is that **every goose install now pulls bitcoin_hashes and
chacha20 even if the user has no intention of ever sharing a
session over Nostr**. That is the supply-chain expansion: not a
risk in any one crate, but a unilateral widening of the audit
surface for a feature that any individual user may or may not want.

## 2. The cargo-feature-gate default-off pattern as the canonical mitigation

The Rust agent-CLI ecosystem already has a settled pattern for
exactly this situation. You name the feature, you put a
`[features]` entry in `Cargo.toml`, you mark all the new
dependencies as `optional = true`, and you `cfg(feature = "...")`
the entire module behind it. Default-off; users who want the
feature build with `--features nostr-share` (or `--all-features`),
distributors who ship goose binaries pick which features to enable
in their build matrix, and the no-feature build does not pull a
single one of the nine new crates into the lockfile graph that
actually compiles.

For this PR, the surgery is small. A first cut:

```toml
# crates/goose/Cargo.toml
[features]
default = []
nostr-share = [
    "dep:nostr",
    "dep:aead",
    "dep:chacha20",
    "dep:bech32",
    "dep:bip39",
    "dep:bitcoin_hashes",
    "dep:bitcoin-io",
    "dep:async-utility",
    "dep:async-wsocket",
    "dep:atomic-destructor",
]

[dependencies]
nostr             = { version = "...", optional = true }
aead              = { version = "...", optional = true }
chacha20          = { version = "...", optional = true }
bech32            = { version = "...", optional = true }
bip39             = { version = "...", optional = true }
bitcoin_hashes    = { version = "...", optional = true }
bitcoin-io        = { version = "...", optional = true }
async-utility     = { version = "...", optional = true }
async-wsocket     = { version = "...", optional = true }
atomic-destructor = { version = "...", optional = true }
```

In code:

```rust
// crates/goose/src/session/mod.rs
#[cfg(feature = "nostr-share")]
pub mod nostr_share;
```

In the route file:

```rust
// crates/goose-server/src/routes/session.rs
#[cfg(feature = "nostr-share")]
async fn share_session_nostr(/* ... */) -> impl IntoResponse { /* ... */ }

#[cfg(feature = "nostr-share")]
async fn import_session_nostr(/* ... */) -> impl IntoResponse { /* ... */ }

pub fn routes() -> Router {
    let r = Router::new()
        .route("/...", post(other_handler));
    #[cfg(feature = "nostr-share")]
    let r = r
        .route("/share_session_nostr", post(share_session_nostr))
        .route("/import_session_nostr", post(import_session_nostr));
    r
}
```

CI gets a single new job: build with `--features nostr-share` and
run the nostr-share-specific tests. The default build remains
bit-for-bit identical to today's tree minus the additions, except
for whatever shared infrastructure the PR also touches.

The cost of doing this is ~20 lines of `Cargo.toml` and a handful
of `cfg` annotations. The benefit is structural: every future
audit of a goose release can answer the question "does this build
include the Nostr code path?" by inspecting one feature flag,
rather than by reading the full `Cargo.lock` and reasoning about
whether `bitcoin_hashes` is reachable from any non-test code path.

## 3. When to feature-gate, and when not to

The natural objection is "isn't this overkill? Every dep gated
behind a feature flag is a combinatorial CI matrix nightmare." It
can be, yes. So the question is when this pattern is worth its
overhead. Here is the rubric I'd use, with goose#8922 as the
worked example:

**(a) Does the feature pull in a coherent, separable dep
sub-tree, or does it touch dozens of existing deps?** Nostr-share
is the easy case: nine new crates, none of them shared with the
rest of the goose codebase. A feature gate excludes them
cleanly. In contrast, a feature that adds, say, a new model
provider that just needs `reqwest` and `serde_json` --- both
already in the tree --- doesn't move the dependency-surface
needle and probably isn't worth the gate.

**(b) Does the feature carry a non-trivial security / threat-
model story?** Encrypted session sharing over a public relay
network does. The decryption-key-in-deeplink decision, the
unauthenticated-handler-on-localhost decision, the
arbitrary-content-publish-under-user-identity decision --- each
of those is a security property that some installs will want
and some will not. Gating the feature lets distributors and
sysadmins make that choice in the build, not at runtime.

**(c) Does the feature add a sustained maintenance load that
some users will not pay back?** The `rust-nostr` ecosystem will
ship breaking changes; goose will inherit a periodic
"reconcile with new nostr SDK" task. Users who never use
session sharing pay that maintenance tax in slower CI, longer
build times, and occasional dependency-resolution conflicts
with their own crates. A feature gate routes that tax to the
users who benefit.

By all three criteria, goose#8922 is a textbook fit for the
default-off pattern. The PR as drafted is good code; what would
make it great code is the ten-line `Cargo.toml` change and the
three `cfg` annotations that turn the feature into something
users can opt into rather than something they automatically
inherit.

## 4. What this generalises to

The agent-CLI cohort I've been tracking through the daily drip
reviews (the cohort that includes `sst/opencode`, `openai/codex`,
`charmbracelet/crush`, `google-gemini/gemini-cli`, `BerriAI/litellm`,
`QwenLM/qwen-code`, and `block/goose`) is at exactly the size where
"one feature, many crates" PRs land regularly. Goose is in Rust;
opencode is in TypeScript; codex is mixed; the others vary. The
specific mechanism differs per ecosystem (cargo features in Rust,
peer-dependencies and dynamic imports in JS/TS, build tags in Go,
etc.) but the *pattern* is the same: when a feature pulls in a
coherent, separable, non-trivially-large dependency sub-tree,
mark it default-off and let the people who want it opt in.

The drip-335 verdict on `block/goose#8922` was "needs discussion"
for exactly this reason: not because the feature is bad, but
because the right shape of the merge is not "merge the PR as
drafted and ship Nostr to every install" but "merge the PR with
a feature gate and let installs choose." That's a small
discussion to have, and it's the kind of discussion that the
review process exists for.

The follow-up I'd watch for in the next few drips: does the
maintainer accept the feature-gate framing, or does the merge
ship with Nostr in the default build? Either answer is
information about the project's posture on dependency-surface
expansion, and it'll matter for similar PRs landing in the
sibling agent-CLI projects over the next few weeks.
