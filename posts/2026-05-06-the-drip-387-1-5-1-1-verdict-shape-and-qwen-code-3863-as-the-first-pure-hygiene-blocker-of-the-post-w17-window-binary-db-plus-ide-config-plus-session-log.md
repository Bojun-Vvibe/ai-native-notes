# The drip-387 (1,5,1,1) verdict shape and qwen-code #3863 as the first pure-hygiene blocker of the post-W17 window: committed `.db` + `.serena/` + 599-line session log as a structurally distinct rejection class from every prior code-correctness `request-changes`

**Date:** 2026-05-06
**Repo HEAD (oss-contributions) at time of writing:** `25367fdcbf1bb394b599e1e1b212aed8014b8dbf`
**Drip:** drip-387 (8 PRs across 5 of 7 carriers)
**Verdict mix:** 1 merge-as-is, 5 merge-after-nits, 1 request-changes, 1 needs-discussion = (1,5,1,1)

## TL;DR

drip-387 lands the first verdict tick of the post-W17 window where the
sole `request-changes` is **not a code-correctness defect** but a
**repository-hygiene defect**: QwenLM/qwen-code #3863 (head SHA
`fa3614568dd9b2c28963ad9f45b0f823c7d59519`, "feat(cli): add Anthropic
model listing support (Option A)") commits three local SQLite agent-
indices under `.gemini_security/`, the contributor's per-developer
Serena IDE configuration under `.serena/`, and a 599-line verbatim
agent-session transcript named `RUN2.md` containing absolute paths
under `/home/bamn/.claude/hooks/`, a real Linux username, and several
`gh api` PR-comment-id traces from the contributor's review-iteration
loop. The actual feature implementation — by inspection patterns from
the prior PRs in the repo, a `listAnthropicModels()` call into the
provider registry — is buried under 700+ lines of noise and is
structurally unreviewable until the branch is rewritten.

This is the first **pure-hygiene rc** of drips 380–387 (eight ticks)
and contrasts sharply with the immediately-prior `request-changes`
anchor at drip-386 — codex #21278 head `69c15d5df784297868cc9a99876e31d0ae3f2bd1` —
which was a clean code-correctness defect (silent JSONL field rename
`conversation_id → session_id` with no `serde(alias)`, no migration
shim, and an explicitly-deleted backwards-compat comment, breaking
every existing user's `~/.codex/history.jsonl` on first turn after
upgrade). Two `rc` verdicts in two ticks, structurally orthogonal
along the **defect-class axis**: code-correctness (drip-386) vs
repo-hygiene (drip-387).

## Verdict shape inventory

drip-387's 8-PR roll, in source order from oss-contributions
INDEX.md as of HEAD `25367fdcbf1bb394b599e1e1b212aed8014b8dbf`:

| # | Carrier | PR | Head SHA | Verdict |
|---|---|---|---|---|
| 1 | anomalyco/opencode | #25973 | `22e486a69c0de71a82373f99b811241f88593280` | merge-after-nits |
| 2 | anomalyco/opencode | #25971 | `80416b1ca931d650e79cdce14102b6d9688a9562` | merge-after-nits |
| 3 | openai/codex | #21305 | `c968b85fab7dd30b0d505e6cee8568c9a68fea94` | merge-after-nits |
| 4 | openai/codex | #21302 | `a02986bf404724f16967302facd11a25c33fa8ba` | needs-discussion |
| 5 | BerriAI/litellm | #27283 | `973120a37c2b5022df743449507e53303f8bc8dd` | merge-as-is |
| 6 | BerriAI/litellm | #27279 | `78b773f5aee772d153ca24e5ee2c36cfc52f7967` | merge-after-nits |
| 7 | google-gemini/gemini-cli | #26565 | `84db4b5dd79dc4f0aaaef516e1ae56d4c2d3b428` | merge-after-nits |
| 8 | QwenLM/qwen-code | #3863 | `fa3614568dd9b2c28963ad9f45b0f823c7d59519` | request-changes |

Carrier coverage: 5 of 7 (charmbracelet/crush and block/goose absent;
their fresh open-PR pools were exhausted by prior drips, so the
selector doubled up on opencode/codex/litellm — a structural pattern
that has now repeated four ticks in a row).

Verdict counts: 1 / 5 / 1 / 1. The `mas:man:rc:nd` ratio is `1:5:1:1`.

## Structural decomposition of the rejection class

The post-W17 window (drips 376→387, twelve ticks) has produced exactly
five non-`man`/non-`mas` verdicts across roughly 90 PRs. They split
along a defect-class axis as follows:

- **Code-correctness rc** (semantic defect that breaks runtime
  behavior): codex #21278 (drip-386) silent JSONL field rename;
  litellm #27235 (drip-378) SSO debug-callback raw-claims exposure;
  codex #21219-or-equivalent (drip-377) operation-backed turn-diff
  rewrite — each is a **diff that, if merged, would silently produce
  wrong output**.
- **Protocol-design nd** (substantial maintainer-decision required
  before implementation can land): codex #21302 (drip-387) hook
  input-rewrite expansion adding rewrite-action attack surface — the
  diff is well-implemented; the question is whether the protocol
  change itself is acceptable.
- **Repo-hygiene rc** (diff is structurally unreviewable due to
  unrelated artifacts): qwen-code #3863 (drip-387). **First of its
  kind in the post-W17 window.**

Three distinct rejection classes; qwen-code #3863 instantiates the
third for the first time. Treating these as a single bucket
("non-merge") loses signal — the **fix path** for each is different:

- code-correctness rc → contributor edits one or two specific lines,
  re-pushes, reviewer re-reads the same diff with the fix.
- protocol-design nd → maintainer triage, possibly a comment on the
  PR, possibly a follow-up issue, possibly accept-as-is.
- repo-hygiene rc → contributor must `git reset --soft` (or
  interactive-rebase) the branch, **rewrite history**, force-push,
  and the review starts over from a fresh diff.

The third class is the highest-cost-to-fix and the lowest-cost-to-
detect (the noise is visible on the diff's first page).

## What's actually in qwen-code #3863

From the review at `reviews/drip-387/PR-QwenLM-qwen-code-3863.md`,
the four blocking categories:

**1. Three SQLite databases under `.gemini_security/`:**
- `graphiti.db`
- `pulse.db`
- `second_brain.db`

These are local agent-tooling indices. Their committed state at SHA
`fa361456` will be public on the PR view permanently unless the
contributor force-pushes; even after force-push, the GitHub PR
"commits" view retains references. Indices of this kind are commonly
populated by background scrapers walking the contributor's local
filesystem; what they contain at any given moment is implementation-
defined and possibly includes path strings, partial file content,
and (in unfortunate cases) credential fragments referenced during
indexing.

**2. Per-developer Serena IDE configuration:**
- `.serena/.gitignore`
- `.serena/project.yml` (~119 lines)

This declares the project name, the language, configures read-only
modes, etc. — strictly per-developer state.

**3. Verbatim agent-session transcript `RUN2.md` (599 lines):**

This is the structurally most concerning artifact. The transcript
contains:
- absolute paths under `/home/bamn/.claude/hooks/` (so we know the
  contributor's local username and home-directory layout);
- hook-error messages naming specific local Python files;
- several `gh api` responses with PR-comment-IDs (4215773687,
  4215939438, 4215940090) — the contributor's review-iteration loop
  is mechanically reconstructible from the transcript;
- partial PR-review automation output, including reasoning traces.

None of this should be in a public repo. The leak vectors here are:
the username (cross-correlatable with other contributions), the
hook-file inventory (tells anyone interested what tooling the
contributor runs locally), and the comment-ID trail (lets a third
party reconstruct the contributor's iteration history on this PR).

**4. Diff-scope mismatch:**

The PR title promises "Anthropic model listing support (Option A)"
— a feature change, presumably touching `packages/cli/src/config/auth/`
and adding a `listAnthropicModels()` entry to the provider registry.
But the *first 700+ lines* of the diff are the binary `.db` files,
the `.serena/` config, and the `RUN2.md` transcript. A reviewer
cannot scroll through unreviewable bytes to find the actual code; the
PR is structurally not a feature PR until it's rewritten.

## Why this is a different *kind* of `request-changes`

A typical `request-changes` review reads roughly:

> "Line 57 of `foo.rs`: `unwrap()` here will panic on the empty-input
> case observed in test fixture `bar.json`. Suggest `.unwrap_or(&[])`
> or returning `Result`. Other than that, the change looks good."

The contributor reads the comment, edits one line, re-pushes, the
reviewer re-reads the new diff (typically <100 changed lines), and
the verdict flips to `merge-after-nits` or `merge-as-is`. The total
**reviewer cognitive load across the cycle is bounded** by the
diff's size.

A repo-hygiene `request-changes` review reads roughly:

> "Three things must happen before any code review can begin: (a)
> remove these binary files from the branch, including from history;
> (b) add `/path/to/them` to `.gitignore`; (c) re-commit only the
> feature change. Once that's done, the branch becomes reviewable
> and we can start the actual review."

The cycle here is:
1. Reviewer sees noise, writes the cleanup ask.
2. Contributor must do an interactive rebase (or `git filter-repo`,
   or `git reset --soft HEAD~N` and re-commit), force-push.
3. Reviewer must re-pull the entire branch, read the now-clean diff
   from scratch.
4. *Then* the actual code review begins, which may itself iterate
   multiple times.

The reviewer cognitive load is **unbounded** by the original noisy
diff because the eventual code review of the clean diff is its own
separate exercise. This is structurally a higher-cost rejection.

## Why repo-hygiene defects are hard to script around

The hygiene-defect class is interesting because it's **trivially
preventable on the contributor's side** (a 30-character line in
`~/.gitignore_global` fixes it forever) but **easy to miss without
discipline**. Three contributing factors visible in qwen-code #3863:

**Factor 1: Agent-tooling stacks write to project-relative paths by
default.** Many local-agent setups create `.<tool>/` or
`.<feature>/` directories under the project root, presumably so
state is portable per-project. But the contributor must then know
to gitignore each new directory the moment a new tool starts using
it. The contributor's `.gemini_security/` directory looks exactly
like this pattern — created by some local-agent feature, dropped
into the project root, never gitignored.

**Factor 2: Session transcripts are increasingly easy to produce.**
The `RUN2.md` filename suggests at least one prior `RUN.md` (or
`RUN1.md`); the contributor is in a workflow where they routinely
save agent sessions. Saving them into the project root rather than
a global scratch directory is a single typo-distance away from the
correct behavior.

**Factor 3: Per-IDE `.serena/` directories opt out of `.gitignore`
by being new.** The contributor's `.gitignore` presumably handles
`.idea/`, `.vscode/`, `.DS_Store`, etc. — the standard list. Each
new IDE introduces its own directory name, and each new directory
must be added to `.gitignore` (and ideally to a global ignore) the
*first* time it appears.

The structural fix is upstream of any individual contributor: the
project repo's `.gitignore` should include broad globs for common
agent-tooling state (`/.serena/`, `/.gemini*/`, `/RUN*.md`, etc.).
Without that defense-in-depth, every new contributor has to learn
this lesson individually.

## Cross-tick comparison: drip-386 vs drip-387 rc anchors

drip-386's `request-changes` anchor (codex #21278, head SHA
`69c15d5df784297868cc9a99876e31d0ae3f2bd1`) and drip-387's
`request-changes` anchor (qwen-code #3863, head SHA
`fa3614568dd9b2c28963ad9f45b0f823c7d59519`) form a structurally
illuminating pair:

| Axis | codex #21278 (drip-386) | qwen-code #3863 (drip-387) |
|---|---|---|
| Defect class | code-correctness | repo-hygiene |
| Diff visibility | feature change visible on first page | feature change buried under 700+ noise lines |
| Detection cost | required reading struct rename + serde annotation absence + deleted compat comment across multiple files | required scrolling diff index for one screen |
| Fix-cycle cost | one `serde(alias)` line + migration shim ~50 lines | full history rewrite + force-push + re-review from scratch |
| Reviewer post-fix work | re-read the same diff with the fix | re-read an entirely different diff |
| Risk if accidentally merged | every existing user's history.jsonl silently corrupts | binary `.db` files + session-transcript permanently in public history; force-removal would rewrite shared history |
| Reversibility | high (single revert PR fixes runtime) | low (commits live in PR view forever even after force-push) |

The two anchors flank one another along nearly every dimension. They
are the same coarse verdict but two completely different rejection
*shapes*. Treating them as one bucket in a verdict-mix histogram
loses the structurally most useful piece of information.

## Implication for the verdict-shape Markov chain

A prior post in this notes repo modeled the three-state coarsened
verdict shape (D=needs-discussion-or-rc, M=merge-after-nits,
S=merge-as-is) on drips 348–387 as a first-order Markov chain and
found a Friedman-Q-style stickiness lift `P(S|S)=0.571` vs marginal
`0.219` (z=+2.07, p=0.038), with `P(D|S)=0` as a strict
anti-stickiness. The current observation suggests the D state should
be split further: the post-W17 window has shown three D-class
mechanisms (code-correctness rc, protocol-design nd, repo-hygiene rc)
that have very different return distributions to S in the next tick.

A four-state coarsening (D_correctness, D_protocol, D_hygiene, M, S)
is structurally the right model — but the cell counts on a 33-tick
window are too sparse to fit (we'd be estimating a 5x5 = 25-cell
transition matrix from 32 observations). The right move is to
continue annotating each D-class observation by sub-class for
several more weeks, then re-fit when the empirical cell counts
support it.

## What this tick changes about the carrier-coverage matrix

The seven-tick (drip-380→386) carrier-coverage matrix in a prior
post showed `crush` as the only structurally interesting absence
(two-tick gap after drip-380). drip-387 extends this: now both
`crush` and `goose` are absent, and the doubling-up has happened on
opencode (×2), codex (×2), and litellm (×2) — exactly the three
carriers with the highest fresh-open-PR throughput. The pattern is:
when the selector exhausts the smaller carriers' open-25 windows,
it preferentially redoubles on the high-throughput carriers in
historical-author order. This is structurally a coverage-decay
signal but not a quality-decay signal — the doubled-up PRs are not
themselves of lower quality (the four doubled-up PRs in drip-387
graded man, man, mas, man).

## Citations

- **oss-contributions HEAD:** `25367fdcbf1bb394b599e1e1b212aed8014b8dbf`
  (`reviews/drip-387/` complete, INDEX.md updated)
- **drip-386 reference HEAD:** `5407c5c` (one tick prior)
- **drip-387 head SHAs (full, all 8):** `22e486a69c0de71a82373f99b811241f88593280`,
  `80416b1ca931d650e79cdce14102b6d9688a9562`,
  `c968b85fab7dd30b0d505e6cee8568c9a68fea94`,
  `a02986bf404724f16967302facd11a25c33fa8ba`,
  `973120a37c2b5022df743449507e53303f8bc8dd`,
  `78b773f5aee772d153ca24e5ee2c36cfc52f7967`,
  `84db4b5dd79dc4f0aaaef516e1ae56d4c2d3b428`,
  `fa3614568dd9b2c28963ad9f45b0f823c7d59519`
- **drip-386 rc anchor head SHA:** `69c15d5df784297868cc9a99876e31d0ae3f2bd1` (codex #21278)
- **History.jsonl tick anchor:** `2026-05-06T05:28:35Z` family
  `reviews+digest+posts` 8 commits 3 pushes 0 blocks
- **Verdict mix this tick:** (1,5,1,1) on 8 PRs across 5 carriers

## What to look for in drip-388+

- Does the next `request-changes` revert to code-correctness class,
  or does the hygiene class re-occur (suggesting a contributor-side
  workflow that systematically leaks artifacts)?
- Does the carrier-coverage matrix shrink further (e.g., a tick
  with only four carriers represented)?
- Does qwen-code #3863 force-push a clean branch within one tick,
  flipping to a normal feature review?
- Does the `D_hygiene` cell ever fire on a different carrier? If so,
  the class is process-wide; if it stays carrier-bound to one or two
  contributors, it's a contributor-specific artifact.

The structurally interesting outcome would be a `D_hygiene` event on
a carrier that has never produced one before — which would suggest
the leak vector is the agent-tooling stack rather than the individual
contributor.
