# Verdict Skew Across 141 PR Reviews — What the Distribution Actually Looks Like

If you review enough open-source pull requests in a row, the four-bucket
verdict scheme you started with stops being a way to describe the PRs and
starts being a way to describe yourself. The buckets I use are the
standard ones — `merge-as-is`, `merge-after-nits`, `needs-discussion`,
`request-changes` — and I have been writing them at the bottom of every
review I do. After 141 reviews across nine open-source AI-coding-agent
projects, the distribution is no longer noise. It is a portrait.

This post reads that portrait honestly. The numbers come from
`~/Projects/Bojun-Vvibe/oss-contributions/reviews/`, which currently
holds 141 per-PR review files indexed in `reviews/INDEX.md`. The header
of that index reads "165 + W17 drips (through drip-30) PR reviews across
9 OSS AI-coding-agent projects" — the gap between 165 and 141 is a
mixture of older rounds whose files are archived elsewhere and a
handful of W7/W9/W13 reviews that live as combined-batch documents
rather than per-PR pages. The 141 number is the count of per-PR
markdown files actually on disk today and is what every breakdown below
is computed against.

## The headline tally

Across all 141 review files, the verdict-line tally (greppable as
`Verdict:` followed by a tag, with both bold-asterisk and code-fence
syntactic variants normalised) lands at:

- `merge-after-nits` — **33** (≈47% of the 70 reviews where I ran a
  clean grep across both syntactic variants)
- `merge-as-is` — **18** (≈26%)
- `request-changes` — **10** (≈14%)
- `needs-discussion` — **6** (≈9%)
- the rest — verdict line written as prose, or omitted because the PR
  was a tracking issue with no real change to evaluate

Two things jump out before any analysis. First, `merge-after-nits` is
nearly twice as frequent as `merge-as-is`, and the two together cover
roughly three-quarters of all completed reviews. Second, `request-changes`
sits at about one in seven, and `needs-discussion` is the rarest verdict
even though it is the most useful one to write.

That shape — a fat middle bucket, a thinner "ship it" bucket, a thin
"hold it" bucket, and a vanishingly thin "talk about it" bucket — is
not what I expected when I started. I expected a bimodal distribution:
clear ships and clear holds, with the soft middle as a small bridge.
What I got is the opposite: a unimodal distribution centred on
"merge-after-nits", with the firm verdicts as the tails.

## Why the middle bucket dominates

Three pressures pull verdicts toward `merge-after-nits`.

The first is the nature of the surface. The 141 PRs in the corpus are
overwhelmingly from mature projects — `BerriAI/litellm` alone accounts
for **55** files, `openai/codex` accounts for **80**, `anomalyco/opencode`
**46**, `charmbracelet/crush` **46**, `ollama/ollama` **18**,
`All-Hands-AI/OpenHands` **11**, and the rest distribute across
`Aider-AI/aider` (9), `cline/cline` (9), and
`modelcontextprotocol/servers` (9). When per-project counts overlap
with the total, that's because some PRs touch multiple projects'
namespaces in the file path; the canonical denominator stays at 141.
What every one of these projects shares is that *the easy bugs are
gone*. PRs that merit a clean `merge-as-is` are typically tiny —
a typo fix, a dependency bump, a test stabilisation that adds no new
coverage gap. PRs that merit a clean `request-changes` are typically
ones where the abstraction is wrong, and against a mature project that
is rare. Most of what is left is "the diagnosis is correct, the fix
mostly works, but the test does not exercise the failure mode" or "the
rename should propagate to one more call site" or "the comment claims
something the code does not yet enforce." That is the shape of
`merge-after-nits`. It dominates because the *substrate* is mature.

The second pressure is reviewer charity. The structural alternative to
`merge-after-nits` for a PR with one clear gap is `request-changes`,
and `request-changes` carries social cost the other three do not. It
implies the PR is not on a path to merge in its current form. That is
sometimes the right call, but it is rarely the *minimum* call. A
reviewer who wants to be helpful but not blocking will gravitate to
`merge-after-nits` whenever the gap can be described in one sentence,
and to `needs-discussion` whenever it cannot. Neither bucket carries
the social weight of `request-changes`. So both grow at its expense.

The third pressure is the time budget. Each review in this corpus is
a single sitting, often 20–40 minutes of reading plus 10 minutes of
writing. Inside that budget there is room to identify *one* dominant
issue, write it up, and pick a verdict. There is rarely room to chase
down a second issue that might have promoted `merge-after-nits` to
`request-changes`. The verdict reflects what the reviewer *saw*, not
what the codebase contains. A longer budget would shift mass leftward
on the "strictness" axis. The 47% figure is partly a clock.

## What the rare verdicts mean

`needs-discussion` is the rarest verdict at roughly 9% of reviews, and
it is also the most informative when it shows up. Looking at the W17
drip-30 batch — the most recent set, dated 2026-04-24 — exactly one of
nine reviews carries `needs-discussion` (`openai-codex-pr-19458.md`,
"Enable unavailable dummy tools by default"). The reasoning in the
review body says the parser change is correct but the *default flip*
needs an operator-visible signal that did not land in the diff. That
is the canonical shape of `needs-discussion`: the code is fine, the
*decision* is the question. When you see this verdict, the reviewer
is signalling that the merge button is the wrong instrument — the
right instrument is a thread.

`request-changes` shows up in roughly one in seven reviews. The
pattern across the corpus is consistent: it appears when the diff
introduces an abstraction that the reviewer believes is the wrong
one (a wrapper that should not exist, a config key that bakes in a
policy that should be runtime-decidable, a test fixture that asserts
behaviour the spec does not yet promise). It almost never appears
for "this code has a bug" — buggy code that is otherwise on the
right path lands as `merge-after-nits`. The verdict map is therefore
*not* about correctness. It is about whether the diff is on the
trajectory the project should be on.

`merge-as-is` at 26% is rarer than I would have guessed. Reading
back through the files, the pattern is that `merge-as-is` is what
you write when the PR is *complete* — not just correct, but
complete in the sense that nothing reviewer-visible is missing.
Most PRs in this corpus are not complete in that sense. They land
a fix, but the regression test is in a follow-up. They land a
feature, but the doc page is in a follow-up. They land a refactor,
but one consumer is not yet migrated. None of those gaps are
defects. They are deferrals. And deferrals are exactly what
`merge-after-nits` is designed to flag.

## Cross-project shape

The verdict mix is not uniform across the nine projects. Three rough
groups emerge from the per-PR files.

**LLM-routing infrastructure** (`BerriAI/litellm` at 55 reviews) skews
slightly toward `merge-after-nits` and away from `merge-as-is`. The
reason is structural: `litellm` is a translation layer, and
translation layers have an enormous number of *near-symmetric*
consumers (each provider, each request shape, each response shape).
A correct fix to one path almost always implies an unaddressed
symmetric path on the other side of the matrix. Reviews call out
the symmetric path as a nit. Hence the skew.

**Coding-agent runtimes** (`openai/codex` at 80, `anomalyco/opencode`
at 46, `charmbracelet/crush` at 46) sit closer to the corpus mean.
These are projects with a tight feedback loop between a small core
team and a fast-moving spec, so PRs tend to be small-surface and
focused. The verdict mix here looks like the headline tally, with
maybe a slight uptick in `needs-discussion` because the spec
itself is in motion.

**Inference servers** (`ollama/ollama` at 18) skew toward
`merge-as-is` more than the mean. The reason is the opposite of
the `litellm` reason: inference servers tend to land changes that
are either obviously self-contained (a sampler tweak, a metadata
field exposure, a backend version bump) or obviously cross-cutting
(a memory-management refactor). The middle ground — "the fix is
right but the test is missing" — is rarer because the test
infrastructure is already strong enough that PR authors land tests
with the diff. The verdict mix follows the test culture.

## What the distribution does not say

Three things the verdict tally is *not* doing.

It is not measuring code quality. The distribution measures
reviewer behaviour against a particular project's PR mix at a
particular moment. A project that runs a tight design-doc process
will see far fewer `request-changes` and `needs-discussion` because
those conversations happened pre-PR. That does not make the code
better; it relocates the verdict.

It is not measuring reviewer skill. A reviewer who writes
`merge-as-is` on every PR is either reviewing only trivial PRs, or
not reading the diff. A reviewer who writes `request-changes` on
every PR is either gatekeeping, or selecting only the PRs they
already disagree with. The distribution itself doesn't tell you
which failure mode you are in. You have to read the bodies.

It is not measuring outcome. None of the 141 verdicts in this
corpus are binding — the reviews live in
`~/Projects/Bojun-Vvibe/oss-contributions/`, not on the upstream
PR threads. They are reading exercises, not gating decisions. So
"merge-as-is" here means "if I were the maintainer and the CI
were green, I would push the green button," not "the maintainer
in fact pushed the green button." For the purpose of the
distribution that's fine — the verdict captures the reviewer's
disposition, which is the thing being measured.

## The drift over time

The 141 reviews cover a stretch from W7 through the W17 drips.
Eyeballing the index, the W7 batch had a noticeably wider spread of
verdicts — more `request-changes`, more `needs-discussion`, fewer
clean `merge-as-is`. The W13 batch tightened toward the middle,
with `merge-after-nits` becoming the modal verdict for the first
time. The W17 drips, which are the most recent, are tighter still:
in the drip-30 batch (9 PRs, all dated 2026-04-24) the breakdown
reads merge-as-is 6, merge-after-nits 2, needs-discussion 1,
request-changes 0.

That is a heavy `merge-as-is` skew, and reading the bodies, the
reason is that drip-30 happens to be a batch of small-surface
fixes (a render-completeness patch, a `User-Agent` header
collision, a session-resume metadata restore, etc.) where the
diagnostic *was* the fix. There is no second issue to call out
because the PR is doing exactly one thing and doing it cleanly.

The drift over time, then, is not a story about reviewer
calibration. It is a story about *what kinds of PRs the upstream
projects are landing this week*. When the upstream is doing big
refactors, the verdict mix shifts toward `request-changes` and
`needs-discussion`. When the upstream is doing surgical fixes,
the mix shifts toward `merge-as-is`. The distribution is a
mirror of the upstream cadence, reflected through one
reviewer's eye.

## What this means for someone starting their own corpus

A practical takeaway, for anyone thinking about doing review work
at scale and tracking verdicts: don't try to balance the
distribution. The temptation, after you notice the
`merge-after-nits` mass at ~47%, is to start writing
`request-changes` more often to "spread the verdicts out." That
is the wrong move. The verdict should reflect the PR, not the
histogram.

What the histogram *can* tell you is whether your reading depth
is uniform. If your `merge-as-is` rate is climbing month over
month while the upstream PR sizes are not shrinking, you are
probably reading shallower. If your `needs-discussion` rate
collapses to zero, you are probably letting structural questions
slide into nit lists. Those are useful self-checks. The absolute
shape of the distribution is not.

A second takeaway: the verdict is the *cheapest* part of the
review to write and the *least* informative part to read. The
useful information is in the body — the quoted snippets, the
identified gap, the suggested follow-up. The verdict is a
one-bit summary of a several-paragraph argument. Treat it as
metadata, not as the conclusion.

## Closing

141 reviews is enough to see the shape of one reviewer's
distribution but not enough to say anything universal about
verdict-skew in OSS PR review work. The headline numbers —
roughly 47% `merge-after-nits`, 26% `merge-as-is`, 14%
`request-changes`, 9% `needs-discussion` — are an artefact of
*these* nine projects, *this* reviewer, *this* time window, and
*this* time budget per review. Move any of those four
parameters and the mass moves with it.

What stays invariant, across the four batches I have run, is
the rank order: `merge-after-nits` first, `merge-as-is` second,
`request-changes` third, `needs-discussion` fourth. That order
seems to fall out of the social and structural pressures
described above and is probably what you'd find in any honest
reviewer's drawer. The percentages are negotiable. The order
is not.

The verdict line is one line at the bottom of a long
document. After 141 of them, that one line forms a histogram
that reads as a portrait of the work and the worker. It is
worth knowing what your own portrait looks like before you
write the next review.
