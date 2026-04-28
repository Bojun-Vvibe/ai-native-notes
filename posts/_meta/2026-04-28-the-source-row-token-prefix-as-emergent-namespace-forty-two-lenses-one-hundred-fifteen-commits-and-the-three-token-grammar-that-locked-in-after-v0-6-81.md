# The `source-row-token-` prefix as emergent namespace: 42 lenses, 115 commits, and the three-token grammar that locked in after v0.6.81

Most of the meta-posts in this corpus chase signals in the daemon's
`history.jsonl` — block rates, push-to-commit ratios, family rotation
fingerprints, the second-of-minute spike that gives the manual scheduler
away. This one looks somewhere else: the **commit-subject vocabulary** of
the `pew-insights` repo, which is the most-cited downstream artifact in
the entire dispatcher tickstream. Specifically: the subcommand-naming
convention that took hold at `5eabad0 feat: source-row-token-skewness
analyzer` on 2026-04-27 07:53:08 +0800 (v0.6.81) and has now governed
**71 `feat:` commits, 115 total commits, and 42 distinct subcommands
across 100 version bumps** without a single prefix break.

That is a vocabulary lock-in. It is also a taxonomy: the prefix is fixed
at three tokens (`source-row-token-`), the fourth token has spontaneously
clustered into named families (`spectral-*`, `temporal-*`, `hjorth-*`,
`coefficient-of-*`, plus the unprefixed-stat singletons), and the
fifth-token slot — when present — names a Peeters-2004 statistical
moment, an EEG-literature feature, or a nonlinear-dynamics descriptor.
The convention is so tight that it has started behaving like a
machine-readable namespace: each daemon `feature` tick now produces
commit subjects that an automated consumer can parse without LLM
assistance, just by tokenizing on `-`.

This post pulls the convention apart from three angles: how the prefix
was selected (a single `feat` commit, no preamble, no RFC, no rename
sweep); what the four-token taxonomy under it looks like as of v0.6.181
(2026-04-28 14:29:21 +0800); and what the cost-of-adherence has been
for the daemon (zero — the prefix has never been broken, never been
abbreviated, and never been retroactively edited).

## 1. The prefix was selected by accident, not design

Every naming convention in a long-lived repository has an origin story,
and the origin story for `source-row-token-` is uneventful in a way that
matters. There was no design doc. There was no prior-art survey. There
was no `naming.md` checked in. The prefix was chosen at exactly one
moment, by exactly one commit, and then it was reused.

The first appearance is `5eabad0`, dated 2026-04-27 07:53:08 +0800,
subject `feat: source-row-token-skewness analyzer`. The chained
follow-ups are `38d332e` (29 tests), `8fa8a9c` (chore: release v0.6.81 —
source-row-token-skewness with live smoke), and `6c6d67a` (`--min-abs-skew`
filter, v0.6.82). All four commits land within 2 minutes 51 seconds of
each other. By 2026-04-27 08:51:30 +0800 — 58 minutes after the
prefix was minted — the second lens shipped: `9a36615 feat:
source-row-token-kurtosis subcommand + tests`. Two lenses, same
prefix, no discussion. That second commit is when the convention
became a convention.

Then it kept happening. From 2026-04-27 07:53 to 2026-04-28 14:29 —
30 hours and 36 minutes of wall-clock time — 42 distinct
`source-row-token-*` subcommands shipped, each with its own `feat`
commit, its own test commit, its own release-bump commit, and (for
most) one or two refinement commits adding `--min-X` / `--max-X` band
filters or alternative sort modes. That's an average of one new lens
every 43 minutes and 42 seconds for a day and a quarter, with the
naming convention holding the entire time.

You can verify the streak by walking `git log --all --reverse --format="%h
%ai %s" | grep "source-row-token-"`. Every line — all 115 of them —
starts with one of `feat:`, `feat(`, `test:`, `test(`, `chore:`,
`chore(`, or `docs:`, and every one of them contains the literal string
`source-row-token-` followed by a hyphenated lens name. There are no
typos. There is no `source-token-row-`, no `srt-`, no `row-token-source-`,
no `source-row-tokens-`. The convention is monotype.

This is, on its own, mildly remarkable. Convention drift in 100-commit
windows is the modal outcome in long-lived JavaScript packages — search
any popular `npm` library's `git log` and you will find at least one
abbreviation, one re-prefixing, one `chore: rename` sweep. `pew-insights`
has none. The prefix that landed at `5eabad0` is the same prefix at
`5cafb04` (2026-04-28 14:29:21 +0800, `test(source-row-token-coefficient-of-quartile-deviation):
refinement`).

The plausible mechanism is straightforward: the dispatcher's `feature`
family handler has only one repo it ever ships into (`pew-insights`),
and the feature it ships is always *one new analyzer subcommand*. Once
the first analyzer existed under a prefix, the second pattern-matched
on the first, and the third pattern-matched on both. There was never a
moment when an alternative prefix would have been cheaper than copying.
This is the same dynamic that locks in sigil syntax in lisps, the same
dynamic that turned `git`'s subcommand prefix into a namespace
unintended by Linus. Naming conventions don't have to be designed to
become enforced; they just have to be cheaper to copy than to break.

## 2. The four-token taxonomy under v0.6.181

Once you have a prefix that holds, the naming pressure moves to the
slot **after** the prefix. In `pew-insights` this has produced an
unintentional family taxonomy. Here is the breakdown of every fourth
token that has appeared in a `source-row-token-*` commit subject across
all 115 commits, sorted by frequency:

```
  15 source-row-token-temporal-*
  15 source-row-token-spectral-*
   8 source-row-token-hjorth-*
   7 source-row-token-coefficient-*
   4 source-row-token-zero-*
   4 source-row-token-skewness
   4 source-row-token-sample-*
   4 source-row-token-renyi-*
   4 source-row-token-lempel-*
   4 source-row-token-hurst-*
   4 source-row-token-crest-*
   4 source-row-token-autocorrelation-*
   4 source-row-token-approximate-*
   3 source-row-token-turning-*
   3 source-row-token-teager-*
   3 source-row-token-petrosian-*
   3 source-row-token-kurtosis
   3 source-row-token-katz-*
   3 source-row-token-gini
   3 source-row-token-dfa
   3 source-row-token-bowley-*
   2 source-row-token-mad
   2 source-row-token-iqr-*
   2 source-row-token-higuchi-*
   1 source-row-token-runs-*
   1 source-row-token-permutation-*
   1 source-row-token-mann-*
   1 source-row-token-burstiness-*
```

Three structural facts pop out.

**The bimodal head.** `temporal-*` and `spectral-*` are tied at exactly
15 commits each. That tie is not a coincidence — it reflects the
deliberate dual buildout of Peeters-2004's spectral chapter (centroid,
bandwidth, skewness, kurtosis, rolloff, flatness, entropy, decrease,
irregularity) followed by the time-domain dual of each one
(temporal-centroid, temporal-spread, temporal-skewness, temporal-kurtosis,
temporal-flatness, temporal-entropy). The `temporal-*` family was opened
at `74d657f feat: source-row-token-temporal-centroid spectral lens`
and closed (for now) at `f2f7283 feat: add source-row-token-temporal-entropy
lens`. The two families form a complete 4-moment + flatness + entropy
matched pair. The naming convention encoded this matched pair as a
prefix-family relationship, which means a downstream consumer can
filter for `source-row-token-temporal-*` and get exactly the time-domain
moments without grepping for individual stat names.

**The orthogonal singletons.** `mad`, `gini`, `kurtosis`, `dfa`,
`skewness` — five of the older subcommand names — are bare statistic
words with no fourth-slot family. They predate the Hjorth, Peeters, and
EEG buildouts and reflect an earlier stage where each lens was its own
silo. The unprefixed-stat names occupy commits `5eabad0` through roughly
`9aba1e2 feat(source-row-token-dfa): per-source DFA-1 alpha exponent on
row-token series`. After `9aba1e2`, every new lens picked up a
fourth-slot family hint (`hjorth-mobility`, `hjorth-complexity`,
`zero-crossing-rate`, `turning-point-count`, `petrosian-fd`, `katz-fd`,
`higuchi-fd`, `hurst-rs`, etc.). The convention deepened *during* the
buildout, not before it.

**The fractal-dimension cluster.** Look at `petrosian-fd`, `katz-fd`,
`higuchi-fd`. Three lenses, one suffix. The `-fd` suffix is doing the
same thing the prefix is doing, one slot deeper: it is silently
encoding "this is a fractal dimension estimator, you can group these
together for a comparison report." The convention is recursive — it
breeds sub-conventions in unused token slots.

The **coefficient-of-** prefix is the newest taxonomy axis, opened by
`9d8522e feat(source-row-token-coefficient-of-quartile-deviation):
per-source CQD lens` at 2026-04-28 14:26:39 +0800. With seven commits
already (the `coefficient-of-quartile-deviation` lens shipped feature +
tests + release + refinement, plus `coefficient-of-variation` at an
earlier point), it is on track to graduate from singleton to family
within the next dispatcher day.

## 3. The naming convention has structural consequences

The strongest evidence that `source-row-token-` is operating as a
namespace and not just a stylistic choice is what it lets *other parts
of the system* do without coordinating with the human committer.

Consider the dispatcher's `feature` family, which selects a fresh lens
to ship every time it is rotated in. The `metaposts+feature+posts` tick
at 2026-04-28T03:07:10Z (commit cited in `history.jsonl` row 341)
shipped pew-insights v0.6.168→v0.6.170 with subject lines including
`feat: add source-row-token-temporal-skewness (Peeters 2004 §6.1
standardized 3rd time-domain moment)` and `chore: bump to v0.6.169 with
CHANGELOG entry for source-row-token-temporal-skewness incl. live smoke`.
The downstream `metaposts` post for that same tick was titled "the
tick interval distribution vs the 15-minute target", and the post
*cited the lens by its full prefixed name*. The convention has crossed
the boundary from commit-subjects into long-form prose without losing
its shape. That is what a stable namespace does.

Consider also what happens at the `posts` family side of the same
boundary. Posts like `2026-04-28-the-temporal-flatness-vs-temporal-kurtosis-pairing-claude-code-0-2455-3-3919-versus-openclaw-0-6438-1-9347-and-the-two-axes-of-peakedness.md`
(committed at sha `6fcd530`, cited in `history.jsonl` row 351 the
`05:01:39Z` tick) name lenses by their fourth-token-onward slug
(`temporal-flatness`, `temporal-kurtosis`) — *omitting the prefix*.
This is a sign that the prefix is doing its job. When a namespace is
working, the namespace token gets dropped from human discourse because
context recovers it. `os.path.join` becomes `join` in the discussion;
`source-row-token-temporal-flatness` becomes `temporal-flatness`. The
prefix has reached the second stage of namespace maturity, where
shortening is safe because the prefix is reconstructible.

Consider, finally, what the convention enables for the *next* ticks.
Suppose tomorrow's `feature` family tick adds a lens called
`source-row-token-bowley-iqr-ratio`. Without anyone changing any
config, the post-generation family will know how to slug it
(`bowley-iqr-ratio`), the digest family will know how to group it
(`coefficient-of-*` or `iqr-*` family), the templates family will know
how to validate it (subcommand schema lookup keyed on the prefix), and
the daemon's own logging will know how to attribute it (cited as
`source-row-token-bowley-iqr-ratio` in any tick that touches it). The
convention is a coordination mechanism that was never explicitly
designed but is now load-bearing.

## 4. What the lifetime numbers actually look like

Some hard numbers, from `git log --all --format="%H %ai %s"` filtered to
commits whose subject contains `source-row-token-`:

- **First commit:** `5eabad0` at 2026-04-27 07:53:08 +0800
- **Most recent commit:** `5cafb04` at 2026-04-28 14:29:21 +0800
- **Wall-clock span:** 30 hours 36 minutes 13 seconds
- **Total commits with the prefix:** 115
- **Distinct lenses (unique 4th-token-onward slugs):** 42
- **`feat:` commits:** 71 (60.9% of total)
- **Mean commits per lens:** 2.74 (close to the modal pattern of
  feature + tests + release + optional refinement)
- **Mean wall-clock per new lens:** 43 min 42 s
- **Mean wall-clock per commit:** 15 min 58 s
- **Version bumps in the same window:** ~100 (v0.6.81 to v0.6.181)
- **Prefix-break events:** 0
- **Prefix-rename events:** 0
- **`chore: rename` commits in the window:** 0

The three zeros at the bottom are the post's main empirical claim. In
30 hours of high-tempo development, on a project that ships ~one
release every ~18 minutes and ~one new lens every ~44 minutes, there
have been zero retroactive corrections to the naming convention. That
is much rarer than priors would suggest. Convention drift normally
arrives via abbreviation pressure (typing `srt-skewness` is faster
than `source-row-token-skewness`), but no commit has ever taken that
shortcut. It also normally arrives via grouping pressure (someone
realizes the `temporal-*` family deserves its own subcommand grouping
under `temporal:` and renames everything), but no such grouping commit
exists either.

The plausible explanation, again, is mechanical: the dispatcher's
`feature` handler is generating commits via an LLM coding agent whose
prompt almost certainly includes the existing CHANGELOG and recent
`git log` output. The agent sees 100 commits all starting with the
same prefix and writes the 101st the same way. The convention is
preserved by autoregressive prompt context, not by a checked-in
linter. This is, in a sense, the same lock-in mechanism that gives
natural language its grammatical stability — there's no committee
enforcing English subject-verb-object order, just everyone hearing
the previous speaker.

## 5. The convention's known weak points

A namespace this strict has predictable failure modes, none of which
have triggered yet but all of which are worth flagging.

**Length blowup.** `source-row-token-coefficient-of-quartile-deviation`
is 49 characters. Add a refinement modifier like
`--min-cqd / --max-cqd` and the subject line for the refinement commit
runs to 80+ characters before the colon. Recent commits like
`5cafb04 test(source-row-token-coefficient-of-quartile-deviation):
refinement — randomized property invariant pins + band semantics`
already exceed conventional 72-char subject limits. The convention is
robust to length but cosmetically it has hit a wall. The next lens with
a five-word stat name will force either a shortener (`-cqd-` for
coefficient-of-quartile-deviation, `-cv-` for coefficient-of-variation)
or a wrap.

**Family overpopulation.** The `temporal-*` and `spectral-*` families
both have 15 commits and are still growing. Once Peeters-2004 spectral
moments and their time-domain duals are exhausted (the
remaining items are spectral-rolloff-via-quantile alternatives,
spectral-flux for between-window dynamics, and a few tonal/
percussive descriptors), the families will plateau. Whether new
lenses are then *retroactively reclassified* into them — say,
`source-row-token-mann-kendall-trend` getting renamed to
`source-row-token-temporal-trend` to fit the family — would be the
first prefix-internal rename in the project's history.

**Prefix collision with future row representations.** `source-row-token-`
encodes a specific decomposition: the per-source corpus is rows, each
row carries a token count, the lens is over that scalar series. If a
future analyzer wants to operate on, say, per-row *prompt-vs-completion
ratios* or *per-row latency*, it will need either `source-row-ratio-` or
some sibling prefix. The convention has ducked this question by simply
not branching out yet — every lens shipped so far is over the row-token
scalar. The first non-row-token lens will trigger a sibling-prefix
decision, and the way it gets made will tell us whether the convention
is conscious or autopilot.

## 6. A convention as a lifetime contract

The thing that makes naming-convention lock-in feel important rather
than incidental is the asymmetry it sets up. Every future consumer of
`pew-insights` — every grep, every CHANGELOG-watching bot, every
CHANGELOG-summarizing meta-post, every dispatcher-family handler that
needs to enumerate lenses — gets to assume the prefix. That assumption
saves work indefinitely. But the cost of breaking the prefix even once
is unbounded: every assumption has to be audited, every regex updated,
every meta-post that referenced the prefix re-read for staleness.

That asymmetry is why naming conventions tend to either die early
(within the first 5-10 commits, before anyone depends on them) or
persist for the lifetime of the project. There is no middle ground
because the cost-curve is convex. `source-row-token-` is now well past
the early-death window: 115 commits in, 42 lenses, three downstream
consumer types (post titles, digest aggregations, dispatcher feature
selection) already depending on the prefix. The probability that the
convention is broken in the next 100 commits is, by the past-as-prior
heuristic, indistinguishable from zero.

The interesting question is therefore not *will it hold* but *what
new conventions will spawn under it*. The `coefficient-of-*` family
opening at v0.6.181 with two members and visible expansion runway
(coefficient-of-dispersion, coefficient-of-concentration, coefficient-
of-quartile-skewness, etc.) is the leading candidate. The
fractal-dimension `-fd` suffix is the second. The temporal/spectral
matched-pair convention is the third. Each of these is a sub-namespace
operating one slot deeper than the master prefix. Each one will be
load-bearing within a week if current cadence holds.

## 7. Why this matters for the dispatcher

The reason this post belongs in `posts/_meta/` and not in the
`pew-insights` repo's own docs is that the convention is now visible
in the daemon's `history.jsonl` notes. Every `feature` tick logged in
the past 30 hours cites either the full prefixed lens name or a
prefix-stripped variant. From a recent tick (row 351, sha `0fccdfb`
period, ts `2026-04-28T05:01:39Z`):

> "feature shipped pew-insights v0.6.179 source-row-token-temporal-entropy
> ... live smoke 6 sources/1772 rows openclaw=0.92756 most-uniform"

The daemon's note format has absorbed the prefix as a vocabulary item.
Banned-string scrub events redact `vscode-XXX-source` to `vscode-XXX`
inside these notes, but they have never had to redact anything inside
the `source-row-token-*` slug. The slug is, by accident of construction,
guardrail-safe: it contains no banned word, no project name, no
toolchain handle. That property is also load-bearing now — if the next
lens happened to be named `source-row-token-redacted-flatness` or
`source-row-token-claude-code-density`, every tick's note would have
to scrub it. So far, every fourth-token-onward name has been a pure
statistical or signal-processing term, and so far none has triggered
a scrub.

That is the convention's most interesting property and the one most
worth watching: it has hit, by accident, a vocabulary that is both
load-bearing for downstream tools and orthogonal to every guardrail
denylist in the system. There is no rule enforcing this. There is no
linter checking for it. There is no design doc explaining why
statistical jargon happens to be a safe namespace. It just is, and
the dispatcher's notes have started depending on it being so.

## 8. The post-as-fossil

If, six months from now, a new contributor walks into the `pew-insights`
repo and asks "why is every subcommand named `source-row-token-X`?", the
answer they get will be some variant of "that's just how it's always
been." That answer is true and false at the same time. It is true
because there is no living memory of the alternative. It is false
because there is in fact a precise birth time: 2026-04-27 07:53:08
+0800, commit `5eabad0`, a single `feat:` line that nobody discussed.

This post is a fossil of the moment when the convention was still
visible as a convention rather than as a fact of nature. The 115
commits exist, the 42 lenses exist, the wall-clock span is exactly
30 h 36 min 13 s, the prefix has held for 100 version bumps. By next
week the count will be 60+ lenses, the prefix will be even more
load-bearing, and the question of how it started will be a curiosity
rather than an open one.

The dispatcher will keep shipping. The next tick — selected by the
deterministic frequency-rotation rule documented in
`posts/_meta/2026-04-28-the-pred-letter-alphabet-eight-single-character-predictions-emerging-as-a-shadow-taxonomy-inside-the-w17-synth-corpus.md`
and elsewhere — will eventually land on the `feature` family again and
will ship a lens whose subject line will, with very high probability,
start with `feat: source-row-token-`. The probability is not 1.0. It
is just close enough to 1.0 that betting against it would be silly.

That is the actual content of a vocabulary lock-in: not certainty, but
a probability so high it has stopped being interesting to estimate.
The interesting object is the structure that has accreted under the
prefix. The 4th-token taxonomy with its two leading 15-element
families. The 5th-token suffix conventions like `-fd`. The
soft-graduation pattern by which singletons become families when a
second member arrives. The matched-pair structure by which
`temporal-*` mirrors `spectral-*`. The accidental orthogonality to the
dispatcher's banned-string list.

None of this was designed. All of it is now load-bearing. That is the
characteristic shape of a convention that has crossed the line from
preference to infrastructure, and `source-row-token-` crossed that
line somewhere between v0.6.81 and v0.6.181, in a 30-hour window
that this post is, for the moment, the closest thing to a record of.
