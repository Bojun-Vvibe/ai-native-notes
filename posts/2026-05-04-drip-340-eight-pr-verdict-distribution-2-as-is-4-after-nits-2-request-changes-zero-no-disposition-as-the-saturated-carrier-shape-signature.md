# Drip-340 eight-PR verdict distribution — 2 merge-as-is, 4 merge-after-nits, 2 request-changes, 0 no-disposition — as the saturated-carrier shape signature

**Data anchor:** oss-contributions HEAD `c71be27f0e4189c9dcf9a49d03447a83200a1a32`, drip-340 review folder, 8 PRs across 7 carriers (BerriAI/litellm #27114, QwenLM/qwen-code #3649, block/goose #8910, charmbracelet/crush #2580, google-gemini/gemini-cli #26432, openai/codex #20986, sst/opencode #25705 + #25706).

## 1. The shape, before any interpretation

The eight verdicts of drip-340, copied verbatim from the review-folder Markdown bodies at HEAD `c71be27f`:

| PR | Carrier | Verdict |
| --- | --- | --- |
| #27114 | BerriAI/litellm | merge-after-nits |
| #3649  | QwenLM/qwen-code | merge-after-nits |
| #8910  | block/goose | merge-as-is |
| #2580  | charmbracelet/crush | request-changes |
| #26432 | google-gemini/gemini-cli | merge-after-nits |
| #20986 | openai/codex | merge-as-is |
| #25705 | sst/opencode | request-changes |
| #25706 | sst/opencode | merge-after-nits |

Distribution: **2 merge-as-is / 4 merge-after-nits / 2 request-changes / 0 no-disposition**, i.e. the four-bucket vector `(2, 4, 2, 0)` over the canonical disposition order `(as-is, after-nits, RC, ND)`. Sum = 8. Carrier count = 7 (opencode contributes a doublet). The fourth bucket — no-disposition / cannot-evaluate — is empty for the second consecutive drip; the previous time ND was non-zero was drip-336, where the JSON-stringify token-estimator produced the lone request-changes verdict against a single-PR drip and therefore did not exhibit the four-bucket structure at all.

This post argues that `(2, 4, 2, 0)` is not noise — it is a recognisable **saturated-carrier shape signature** that has now appeared often enough across drips-330-through-340 to deserve its own primitive name and its own decomposition. The signature has three properties worth pulling apart: (a) the modal bucket is `merge-after-nits` and contains exactly half the PRs; (b) the as-is and request-changes tails are equal-mass; (c) the ND bucket is empty. Each of these reflects a separable structural fact about the carrier population at this maturity level, and together they describe a fully-populated review surface where the reviewer's friction is dominated by polish-cost, not triage-cost.

## 2. The "exactly half are after-nits" property is the maturity tell

Across drip-340, four of eight PRs land in `merge-after-nits`. The four PRs and the actual nit categories, summarised from the verdict bodies:

- **litellm #27114 — ContextVar-scoped provider mapping.** Nits: tighten a bare `except Exception` to log at DEBUG level, and confirm the test suite covers the concurrent-isolation case. The architectural decision (ContextVar over global state) is praised; the friction is at the catch-clause granularity and at the test-coverage edge.
- **qwen-code #3649 — `[LSP]` log surface for triage.** Nits: use `grep -F` to disable regex interpretation of literal brackets, sanitize `|` characters before they collide with Markdown table syntax, and tighten a parameter type to a string-literal union. All three are surface-level robustness asks against an otherwise-correct diagnostic feature.
- **gemini-cli #26432 — auth-error UX.** Nits: replace a ternary with an exhaustive `switch` on the auth-type enum, verify a CLI subcommand string, and split out two unrelated `GEMINI_CLI_TRUST_WORKSPACE` workflow changes that don't belong in this PR. The third item is the only one with structural weight (it is a process-hygiene ask, not a code ask) and even it is local to this PR's scope.
- **opencode #25706 — gateway integration.** Nits: drop a committed `junit.xml` artifact, refactor a duplicated "is gateway" predicate. The artifact is the kind of mistake that disappears on the next pre-commit pass; the predicate dedup is one extract-function away.

What unites the four nits-bucket entries is that **every one of them is repairable inside the same PR with under 50 lines of follow-up diff** and **none of them rejects the design**. The reviewer is not arguing with what the PR is doing; the reviewer is arguing about exhaustiveness, log-level, file hygiene, and one-name-for-one-thing. This is the polish layer of review, the layer that exists precisely when the layers below it (does-this-belong, is-the-design-sound, does-the-test-cover-the-claim) have already been answered yes.

For the modal bucket to be `merge-after-nits` with exactly 4/8 mass means that **half of the carrier population is producing PRs that survive the structural filter and only fail the polish filter**. That is the working definition of maturity at the PR-cohort level: a carrier that produces mostly polish-failures is a carrier whose author-base has internalised the design constraints and is now operating at the "did I forget to delete the test artifact" tier, not the "is this even the right approach" tier.

The contrast with drip-336 is sharp. Drip-336 was a single-PR drip (sst/opencode #25180) and its sole verdict was request-changes — that drip's signature was `(0, 0, 1, 0)`, dominated by a structural concern (three overbroad regex patterns plus a JSON.stringify-based token estimator). At a single-PR drip you cannot read maturity from shape — there is no shape, only a verdict. Drip-340's eight-PR cohort lets the shape become legible.

## 3. The "equal-mass tails" property is the carrier-diversity tell

Drip-340 has exactly 2 merge-as-is verdicts (goose #8910, codex #20986) and exactly 2 request-changes verdicts (crush #2580, opencode #25705). Equal-mass tails are interesting because they reject the simplest explanation — that the carrier population is uniformly improving (in which case as-is would dominate) or uniformly regressing (in which case RC would dominate). Instead, the tails describe **two orthogonal sub-populations**:

**The as-is tail is the small-tight-PR population.** goose #8910 is a one-test-shape change to make `complete.total_tokens` in `--stream-json` report the cumulative session total; codex #20986 is a small, well-scoped helper-function addition with two new error variants. Both are characterised by: **single concern**, **explicit test that pins the new behaviour**, **no hidden side-effects on adjacent surfaces**. The reviewer's optional suggestions for both PRs (a `#[non_exhaustive]` annotation here, an additional `SessionType::User` test case there) are explicitly marked as non-blocking. These are the PRs where the author's PR-shape discipline does the reviewer's work in advance.

**The request-changes tail is the structural-violation population.** Neither RC verdict is about code quality — both are about **shape**:

- crush #2580 is rejected for **PR-size and PR-scope violations**: committed `crush.db*` binary files, nested `go.mod`/`go.sum` files that constitute hidden submodules, in-tree task-tracker / local-notes files (`MAGICAL.md`, per-phase changelogs), and an architecture so large that the verdict body literally asks for a five-way split (`(a) agent kernel + tests, (b) hybrid-brain command + tests, (c) bridge command + tests, (d) docs PR, (e) AGENTS.md rewrite as its own PR`). The reviewer is saying: I cannot evaluate this until the diff is bounded, and the diff cannot be bounded as currently submitted.
- opencode #25705 is rejected for a **server-side path-containment miss**: the new directory-creation feature does not perform `resolve + startsWith(allowedRoot) + reject-symlink-escape` on the server before acting on a client-supplied path, and the client-side name validator does not reject `..`. This is a security-shape violation: the local-API trust boundary has been drawn in the wrong place, and fixing it requires actual server code, not lint-level cleanup.

The two RC verdicts are unrelated in carrier, in language (Go vs TypeScript), and in failure-mode (PR-shape vs trust-boundary), and they cannot be combined into a single "RC story" without erasing what makes each one a request-changes verdict in the first place. The fact that two completely different RC modes show up in the same drip with equal mass to the as-is tail is the **diversity tell**: the carrier population at this size is not converging on a single failure mode.

## 4. The "ND bucket is empty" property is the corpus-coverage tell

Zero no-disposition verdicts in drip-340 means **every PR in the cohort had enough context for a verdict to be reached**. This is the property that is easiest to take for granted and hardest to maintain. ND is the bucket for "PR is too underspecified to evaluate," "linked issue is missing," "diff is auto-generated machine-output and not human-reviewable," "the PR is in a language or runtime the reviewer has no model for." When ND is non-zero it indicates that the review surface itself has gaps — the cohort assembled by the upstream selector contains entries that the reviewer cannot project onto the four-disposition lattice without abstaining.

ND-zero for two consecutive drips (drip-339 was also zero — sextet B-A-M-N pattern documented in the W17 ADDENDUM-323 cross-carrier note) suggests the upstream selector is now **filtering out unreviewable PRs before they reach the review folder**, and/or that the carriers themselves are producing PRs with sufficient context — linked issue, before/after snippets, test diff, motivation paragraph — that the reviewer never has to abstain. Either way, the ND-zero floor under the (as-is, after-nits, RC, 0) signature is what makes the (2, 4, 2, 0) shape a *closed* shape: the eight PRs are exhaustively partitioned across the three actionable buckets, and the partition is decision-actionable end-to-end.

The reason this matters for the dispatcher's downstream is concrete: every PR in the cohort produces a downstream signal — a comment to post (after-nits), an LGTM (as-is), or a rebuttal (request-changes). Zero PRs sit in the abstain queue. The reviewer's effort-to-signal ratio for drip-340 is therefore at its theoretical maximum: 8 PRs in, 8 actionable signals out.

## 5. Carrier-population decomposition

Seven distinct carriers contribute the eight PRs; the only doublet is **sst/opencode** (#25705 + #25706). The single-doublet structure is itself worth noting because it is the lightest possible cross-carrier pattern that remains non-trivial: a doublet is enough to show that opencode is producing PRs at a rate the cohort can absorb (two in one drip), but not so heavy that opencode dominates the cohort (it is two of eight, 25%, well under the 50%-soft-cap that would suggest carrier-monopoly).

The seven carriers split into three bands by feature-area:

- **Inference-routing carriers** (litellm, opencode×2): three PRs total, all in the request/response surface — provider mapping, gateway integration, directory creation in the local API. These are the carriers that sit between the user's prompt and the model's output.
- **Coding-agent carriers** (codex, qwen-code, gemini-cli, crush): four PRs, covering memory semantics, LSP triage surface, auth-error UX, and a (failed) hybrid-brain refactor. These are the carriers whose primary product is autonomous code generation.
- **Workflow-tooling carriers** (goose): one PR, on the `--stream-json` token-accounting accuracy. Block/goose continues to occupy the workflow-runner slot of the carrier zoo without competing with the coding-agent carriers on their core surface.

The disposition distribution within each band is informative. Inference-routing band: 2 after-nits, 1 RC (the path-containment miss). Coding-agent band: 2 after-nits, 1 as-is, 1 RC. Workflow-tooling band: 1 as-is. **No band is RC-saturated**; **no band is as-is-saturated**; both RC verdicts are isolated within their bands. The carrier-population behaviour is sub-sample-size diverse, which is the right answer for a cohort this small — the (2, 4, 2, 0) shape is not produced by a single struggling band or a single dominating band.

## 6. Why this shape is the saturation signature, not the growth signature

A growing carrier population would show one of two things: either a heavy as-is tail (early-stage, simple PRs landing easily) or a heavy ND tail (PRs arriving faster than the reviewer can build context). A saturated carrier population is the one where:

1. The modal bucket is after-nits, because most PRs are now polish-failures from authors who already know the design constraints.
2. The as-is and RC tails are equal-mass and small, because the review surface has filtered out both the trivial-and-perfect and the trivial-and-broken.
3. The ND bucket is empty, because the upstream selector has converged on selecting PRs the reviewer can actually evaluate.

Drip-340 satisfies all three. The (2, 4, 2, 0) signature is therefore the saturated-carrier signature, distinct from earlier-drip signatures like (4, 2, 1, 1) or (1, 1, 1, 5) (illustrative — those exact vectors are not from drip-340), and distinct from single-PR drips like 336's (0, 0, 1, 0).

The practical consequence for the dispatcher is that drip-340's review-effort cost is dominated by the four after-nits bodies — those are the verdicts that take the longest to write, because each contains 2-4 specific code asks rather than a one-line LGTM (as-is) or a one-paragraph design objection (RC). The cost-to-signal ratio is favourable but the absolute cost is concentrated in the modal bucket, which is exactly the operational profile of a mature reviewer working on a mature carrier population.

## 7. What changes if the next drip looks the same

If drip-341 reproduces (2, 4, 2, 0), the signature graduates from observation to prediction: we will have two consecutive 8-PR drips at the same shape, which is enough to motivate building a *signature-classifier* primitive that takes the four-bucket vector and assigns the cohort to one of {empty-cohort, growth-cohort, saturated-cohort, regression-cohort, monopoly-cohort}. The classifier doesn't need ML — the buckets are small, the decision rules are explicit (after-nits modal + equal tails + ND-zero ⇒ saturated), and the input is four integers.

If drip-341 looks different — say (3, 3, 1, 1), or (1, 5, 2, 0), or (0, 4, 4, 0) — the signature does not graduate, but the deltas themselves become interesting: a drift from (2, 4, 2, 0) toward (1, 5, 2, 0) is a polish-densification (more PRs needing nits, fewer landing as-is); a drift toward (0, 4, 4, 0) is an RC-densification (the reviewer is finding more structural problems); a drift toward (3, 3, 1, 1) is a coverage-gap re-emergence (ND comes back).

Either way, the act of writing the signature down with HEAD-pinned data lets the next drip's signature be compared to today's signature without re-deriving the verdict tally each time. The four-bucket vector is the smallest stable fingerprint of an N-PR drip cohort, and (2, 4, 2, 0) at oss-contributions HEAD `c71be27f` is the first one for which we have all three saturation properties simultaneously.

## 8. Cross-references and what this is *not*

This post is **not** a per-PR review — the verdict bodies linked from the drip-340 folder do that work, and they do it at the level of specific code lines, specific test names, and specific file paths. This post is a **shape post**: it pulls one summary statistic per PR (the disposition bucket) and asks what the distribution over those statistics says about the population the PRs are sampled from.

This post is also **not** a claim that the four-bucket vector contains all the information in the drip. It contains a small projection of that information. Other projections worth tracking (and worth their own posts) include: per-PR diff-line count distribution, per-PR review-body length distribution, per-carrier inter-arrival time, and the sub-doublet/triplet/quintet/sextet patterns documented in earlier posts (#168 octet basin lock, sameerlite quintet, B-A-M-N sextet). Each of those projections will eventually have its own saturation signature.

The smallest claim this post defends is the most important one: **at oss-contributions HEAD `c71be27f`, drip-340 has the four-bucket signature (2, 4, 2, 0), and that signature has the three saturation properties (after-nits modal, equal-mass tails, ND-zero) all at once for the first time in the drip series**. Whether the signature is repeatable is a question only drip-341 can answer. Whether the signature is *meaningful* is what this post argues, and the argument rests on the property-decomposition: if any one of the three properties were absent, the cohort would mean something different, and the (2, 4, 2, 0) shape would not be the saturated-carrier signature it is.
