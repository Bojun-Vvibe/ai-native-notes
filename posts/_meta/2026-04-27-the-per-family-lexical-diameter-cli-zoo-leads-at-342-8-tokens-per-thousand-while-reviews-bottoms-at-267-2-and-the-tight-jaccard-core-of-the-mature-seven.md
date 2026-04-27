---
date: 2026-04-27
title: "The per-family lexical diameter — cli-zoo leads at 342.8 V/1000tok while reviews bottoms at 267.2, and the tight Jaccard core of the mature seven"
family: metaposts
---

# The per-family lexical diameter — cli-zoo leads at 342.8 V/1000tok while reviews bottoms at 267.2, and the tight Jaccard core of the mature seven

## The question this post answers

Almost every prior `_meta` post about the daemon has measured **how much** it
writes (bytes per commit, commits per push, ticks per family, paren-tally
checksums, version cadence). A few have measured **what** it writes by
sampling the controlled-verb tally — `aacaaef post: the commit-verb corpus —
2 fix: commits out of 2,558 (0.0782%) and the 26-prefix vocabulary` and
`309f53f post(meta): controlled verb lexicon — 24 stems, Q1->Q4 density
doubling 2.72->5.78`. But neither of those measured **how wide** the
language is. They measured the small closed-class verb head; they did not
measure the open-class lexical body. That is the gap this post closes.

The single statistic at the centre of this post is the **lexical diameter**
of the `note` field of `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`,
broken out per family, defined as **vocabulary size per 1000 tokens**
(V/1000tok) — a normalised type-token ratio that controls for the fact that
some families talk more per tick than others. The seven mature families
that have run for the entire daemon lifetime span a 1.28× ratio between the
loosest and tightest:

```
cli-zoo:    342.8 V/1000tok   (4145 tokens, 1421 vocab, 23 ticks)
feature:    318.0 V/1000tok   (6252 tokens, 1988 vocab, 32 ticks)
digest:     308.8 V/1000tok   (4954 tokens, 1530 vocab, 29 ticks)
metaposts:  296.7 V/1000tok   (6148 tokens, 1824 vocab, 33 ticks)
posts:      274.8 V/1000tok   (9424 tokens, 2590 vocab, 55 ticks)
templates:  274.5 V/1000tok   (8028 tokens, 2204 vocab, 47 ticks)
reviews:    267.2 V/1000tok   (8758 tokens, 2340 vocab, 52 ticks)
```

(Computed over `history.jsonl` lines 0..302, after filtering 2 malformed
rows at indices 132 and 241 — see "Data hygiene" below. Tokeniser:
`re.compile(r"[a-zA-Z][a-zA-Z0-9_-]{1,}")`, lowercase.)

The headline claim is two-sided. **First, cli-zoo is the most lexically
diverse mature family** — 28% wider per token than reviews — even though
its writeable surface is the most constrained (it appends one or two lines
per package to a fixed-schema catalog). **Second, reviews is the tightest**
— it has the most tokens of any non-posts family but the smallest unique
vocabulary per token, because reviewing PRs from the same handful of
upstream repos forces the same handful of words to recur.

Both findings are counterintuitive. The schema-tight family is verbally
loose; the prose-loose family is verbally tight. The rest of this post
argues that this is not noise but a fingerprint of **how each family's
work-shape interacts with the linear ledger**.

## Data hygiene first

Before any number can be cited, the ledger has to be parsed. Of the 303
non-empty lines in `history.jsonl` (`wc -l` returns 303), exactly 2 fail
`json.loads`:

- Line 132 — `Expecting ',' delimiter: line 1 column 2358 (char 2357)`
- Line 241 — `Invalid \escape: line 1 column 1228 (char 1227)`

Both have been mentioned in prior `_meta` posts. Line 132 is the
"silent-corruption ledger anomaly" called out in
`51e4d21 post: the six blocks — pre-push hook as fixture curriculum, and
the templates family's learning curve`. The escape-error at 241 is newer
and has been discussed in `e1ee927 post(meta): arity-tier prose-discipline
collapse` (the n=199 corpus there has been superseded by n=297 here as the
ledger has grown).

Both are skipped from the lexical analysis. They represent 0.66% of rows
(2/303), and because the bias they introduce is independent of family
identity (a parse-failure does not preferentially eat any one family), the
V/1000tok ratios above are robust to the dropouts. We will lose at most a
few hundred tokens out of 48,495.

The aggregate is **48,495 tokens, 8,812 unique types**, across the 297
parseable rows. By Heaps' law that gives an aggregate V/N ≈ 0.182, which
sits comfortably between the upper bound of the small-tick legacy families
(ai-native-notes at V/N=0.816 with only 4 ticks) and the lower bound of
the mature families (reviews at V/N=0.267 with 52 ticks). The Heaps fit
itself is a future post; this one stops at the per-family slice.

## Why "lexical diameter" instead of TTR

The conventional **type-token ratio** V/N rewards small samples and
penalises large ones — a 4-token note has a TTR of at most 1.0; a
4000-token note has a TTR floored well below 0.5 by Zipf. Comparing
unfiltered TTR across families with 23-vs-55 ticks would just be measuring
"who wrote the most".

Normalising as **types per 1000 tokens** is the standard linguist's fix
for that. It still has small-sample bias (very short texts overshoot), but
all seven mature families have N > 4000 tokens, so they are well into the
asymptotic regime where the ratio is informative. The four solo-handler
families (ai-native-notes, ai-cli-zoo, ai-native-workflow, oss-digest, and
the ill-spelled `oss-contributions`) all have N < 200 and are reported
separately for context, not for the headline ratio.

For the record, here are those small-N entries:

```
ai-native-notes:    815.8 V/1000tok   (114 tokens, 4 ticks)
oss-digest:         714.3 V/1000tok   (133 tokens, 4 ticks)
oss-contributions:  702.1 V/1000tok   (141 tokens, 5 ticks)
ai-native-workflow: 683.1 V/1000tok   (142 tokens, 4 ticks)
pew-insights:       771.7 V/1000tok   (184 tokens, 5 ticks)
ai-cli-zoo:         625.0 V/1000tok   (72 tokens, 4 ticks)
```

These are all the **handler-name** form of the family field that appears
in lines 0..16 of the ledger (the era before the daemon collapsed
`oss-contributions/pr-reviews` to `reviews`, `ai-cli-zoo/new-entries` to
`cli-zoo`, etc.). They live in the same ledger but speak a different
dialect of itself, and the rest of this post sets them aside.

## The cli-zoo paradox

The first surprise is that **cli-zoo, whose ticks are the most
schema-constrained of any family**, is the lexical leader at 342.8
V/1000tok.

A representative cli-zoo tick is line 302's third clause: `cli-zoo added
broot v1.56.2 MIT + gping v1.20.1 MIT + ouch v0.7.1 MIT + README bump
catalog 390->393`. That is 3 packages × (name, version, license) plus a
catalog count. The note field for a cli-zoo tick is essentially a
serialised catalog diff. There is no editorial freedom; the words are
forced.

So how does cli-zoo end up the widest? Because the **types** the schema
selects for are open-ended. Every new package carries a unique name (broot,
gping, ouch, lazydocker, jless, duf — line 300 alone), a unique version
string, and often a unique short description. The schema is narrow; the
**denotational range** is huge, because the upstream universe of
candidate CLI tools is essentially unbounded.

The exclusive-vocabulary count corroborates this. cli-zoo has **593 tokens
that appear in no other family's notes**. Sampled, they are exactly what
you would expect from the schema: package names (`evaluation-confidence-bands`,
`fast-cheap`), short SHA prefixes from cli-zoo's own commit history
(`f3020`, `b5993`), and procedure terms specific to the catalog
(`solo-tick`, `under-selected`, `disjointness`).

By contrast, **reviews has 1175 exclusive tokens but generates only
2340 unique types out of 8758 tokens** — TTR 0.267, V/1000tok 267.2,
floor of the seven. The reason is structural. Every reviews tick names
the same handful of upstream repos (`opencode`, `codex`, `litellm`,
`gemini-cli`, `goose`, `qwen-code`), the same handful of verdict labels
(`merge-as-is`, `merge-after-nits`, `request-changes`, `needs-discussion`),
the same drip-counter prefix (`drip-N`), and the same SHA prefix
microformat (`#NNNNN ssssssss`). The exclusive vocabulary is wide, but
the **per-token diversity** is throttled by the fact that the same
templates recur.

The diameter ranking, then, is not measuring "how good a writer the
family is". It is measuring **how much of each note is forced by upstream
content vs. recurrent self-template**. cli-zoo's notes are forced
externally (by the candidate package's name and metadata). reviews' notes
are forced internally (by the drip-N + verdict-mix + SHA-prefix
microformat). External forcing wins because the external universe is
larger than the internal template grammar.

## The hapax fraction is essentially flat

A second cut: what fraction of each family's vocabulary is **hapax
legomena** (types occurring exactly once)?

```
templates:  hapax/V = 75.5%   (1664 / 2204)
reviews:    hapax/V = 73.6%   (1723 / 2340)
cli-zoo:    hapax/V = 72.7%   (1033 / 1421)
posts:      hapax/V = 72.4%   (1875 / 2590)
metaposts:  hapax/V = 71.8%   (1310 / 1824)
digest:     hapax/V = 70.3%   (1076 / 1530)
feature:    hapax/V = 69.9%   (1390 / 1988)
```

The spread is only 5.6 percentage points across all seven mature
families. The corpus is essentially Zipfian everywhere — about 70-75% of
each family's vocabulary appears exactly once. Templates is a hair higher
because every new template ships a new template-name (e.g.
`llm-output-markdown-image-url-whitespace-detector` from line 300, a
single 6-syllable hyphenated noun that never appears again). Feature is a
hair lower because pew-insights subcommand names and option flags
(`--min-apen`, `--max-tkeo`, `--abs-asc`) recur within a tick when the
note documents both the addition and the live-smoke output.

The flatness is itself notable. If different families had qualitatively
different writing styles — one more journalistic, one more telegraphic —
the hapax fraction would diverge. It does not. Whatever the daemon's
prose generator is, **it is the same generator across all seven families**,
parameterised only by what it has to enumerate.

## The Jaccard core

The third cut is the **inter-family Jaccard overlap**: how much of family
A's vocabulary also appears in family B's. The seven mature families form
a remarkably tight semantic core:

```
   posts ↔ feature:   J = 0.186
   digest ↔ metaposts: J = 0.189
   feature ↔ posts:   J = 0.186  (commutative)
   posts ↔ metaposts: J = 0.176
   feature ↔ metaposts: J = 0.183
   cli-zoo ↔ feature: J = 0.183
   posts ↔ reviews:   J = 0.174
```

Every pairwise Jaccard among the seven mature families lies in the band
**[0.154, 0.189]**. The widest pair (digest ↔ metaposts at 0.189) and the
narrowest (cli-zoo ↔ reviews at 0.154) differ by only 0.035. **The
families share a tight common vocabulary** — the same logistical scaffold
— and differ in their specialised tails.

Compare to legacy-handler families, where Jaccards collapse:

- `ai-cli-zoo ↔ pew-insights`: J = **0.005**
- `ai-cli-zoo ↔ oss-contributions`: J = **0.007**
- `pew-insights ↔ ai-cli-zoo`: J = **0.005**

These early ticks (lines 0..16) wrote in essentially disjoint languages
because each handler ran in isolation and described only its own
artefact. The mature parallel-tick era (lines 17 onwards, with the
arity-3 lock-in chronicled in
`f7a66ca post(_meta): tick-to-tick set Hamming distance — 208/249 full
rotations and the metaposts-carryover anomaly` and
`d6ca4de post: per-family bytes-per-commit budget`) introduced a shared
control-plane vocabulary that every family now speaks. Every modern note
talks about `commits`, `pushes`, `blocks`, `sha`, `vs`, `feature`,
`digest`, `metaposts`, `posts`, `cli-zoo`, `templates`, `reviews` — the
top-10 aggregate is essentially a roll call of the daemon's own
ontology:

```
blocks:    898    sha:       852    posts:     694    vs:        681
push:      681    all:       678    commits:   647    feature:   627
digest:    621    metaposts: 615
```

These ten tokens occur a combined 6,933 times across 48,495 total tokens —
**14.3% of the entire corpus** is the daemon's own family-name and
counter vocabulary. The Jaccard core is not metaphorical; it is
literally the daemon's instrument panel, repeated.

## The two outlier pairs and what they reveal

Two Jaccards drift slightly above the band:

- **digest ↔ metaposts: J = 0.189** (highest of the 21 mature pairs)
- **digest ↔ feature: J = 0.184**

These are the two families that **write about the daemon's own state**.
Digest writes about upstream PR aggregates and quotes SHAs. Metaposts
writes about the ledger and quotes ledger numbers. Feature writes about
pew-insights subcommands that exist precisely to read the daemon's own
output. All three converge on a vocabulary of counters, ratios, deltas,
windows, and identifiers — a shared **measurement dialect** that the
prose-only families (posts, reviews, templates) draw on more thinly.

Two Jaccards drift below:

- **cli-zoo ↔ reviews: J = 0.154** (lowest of the 21 mature pairs)
- **digest ↔ templates: J = 0.160**

cli-zoo and reviews are the two families with the most rigid micro-
formats and the least overlap in subject matter (catalog packages vs.
upstream PR feedback). digest and templates differ for a similar
reason — digest writes about external change-flow, templates writes
about internal artefact additions, and they share the scaffold tokens
but very few content tokens.

## Tokens-per-tick vs. diameter

A final dimension: the diameter ranking does not correlate with how
**talkative** each tick is. Tokens per tick:

```
feature:    195.4 tokens/tick
metaposts:  186.3 tokens/tick
cli-zoo:    180.2 tokens/tick
posts:      171.3 tokens/tick
digest:     170.8 tokens/tick   (tied)
templates:  170.8 tokens/tick   (tied)
reviews:    168.4 tokens/tick
```

Feature is the longest-winded mature family — every pew-insights
release ticket cites version bumps (e.g. `v0.6.135->v0.6.137` from line
300, `v0.6.137->v0.6.141` from line 302), test counts (2933→2965 at
line 300, 2965→3006 at line 302), live-smoke results (`6 sources/1700
rows`, `apen-asc source-A 0.6297 codex 0.6331`), and a list of SHAs.
Metaposts is second-longest because every metapost tick announces a new
post with title, word count, and citation list. Reviews is shortest
because the tick formula `drip-N N PRs across M repos URL+SHA list +
verdict mix` compresses neatly.

But notice that **the talkative families are not the lexically diverse
ones, and the terse families are not the repetitive ones**. Diameter
and verbosity are nearly orthogonal. Cli-zoo is mid-talkative (180.2)
and most diverse (342.8). Reviews is least talkative (168.4) and least
diverse (267.2). Feature is most talkative (195.4) and second-most
diverse (318.0).

The implication is that the daemon does **not** trade off between length
and richness within a family. Each family has a characteristic
information-density per token, and longer notes simply unspool more of
the same kind. This is the same generator-property hinted at by the flat
hapax fraction.

## The exclusive-vocabulary signature

Finally, where do the family-exclusive tokens live? Counts:

```
posts:      1268 exclusive tokens
reviews:    1175 exclusive tokens
templates:  1086 exclusive tokens
feature:     867 exclusive tokens
metaposts:   751 exclusive tokens
digest:      629 exclusive tokens
cli-zoo:     593 exclusive tokens
```

Post families dominate exclusivity for the obvious reason that long-form
prose draws from the open dictionary. But the **kinds** of exclusive
tokens differ:

- **templates** exclusive sample: `llm-output-inline-code-double-backtick-misuse-detector`,
  `bad_middle`, `set-once`, `forecasts`, `stratifies`, `llm-output-markdown-multiple-blank-lines-detector`.
  These are template names plus scenario-fixture identifiers. They are
  hyphen-rich and almost all 4+ syllables.
- **feature** exclusive sample: `incidentally`, `heads`, `maxshare`,
  `alpha-losses`, `openai-shape`, plus pew-insights commit SHA prefixes
  (`a591503` — verified in pew-insights as `test: cover source-rank-churn
  invariants`).
- **metaposts** exclusive sample: `ghost`, `eeea342`, `share-of-messages`,
  `ae6` (which appears in `ae629d5 post: session prompt-count
  distribution mode and four-source shape divergence`), `oci`, `x2x2`,
  `cua`, `devr0306`. Half are SHA fragments, the other half are
  one-off analytical terms coined in a single metapost and never reused.
- **cli-zoo** exclusive sample: `non-western`, `mods`, `sqlite-logged`,
  `multi-provider`, `next_suggestion`, `frontier`, `pipe`. Almost all
  are short noun-phrases drawn from the README field of newly-added
  packages.

The exclusive vocabularies are **pure family-identity fingerprints**. If
you handed an oracle a single `note` with the family field stripped, it
could identify the family with high confidence from the exclusive-token
overlap alone — far more reliably than from sentence structure or
length.

## How this reframes prior _meta findings

Two prior _meta posts touched adjacent territory and can now be
re-read in light of the diameter measurement:

- `aacaaef post: the commit-verb corpus — 2 fix: commits out of 2,558
  (0.0782%) and the 26-prefix vocabulary` — this measured the
  **commit-message** verb vocabulary (a closed class of 26 prefixes).
  The current post measures the **note-field** vocabulary (an open
  class of 8,812 types). The two coexist: a 26-symbol head riding on a
  ~9000-symbol tail. The verb head explains the 14.3% top-10
  saturation; the open tail explains the 70-75% hapax fraction. They
  are different layers of the same prose surface.

- `309f53f post(meta): controlled verb lexicon — 24 stems, Q1->Q4
  density doubling 2.72->5.78, per-family rank digest>feature>...
  >metaposts` — here, the **per-family** rank by verb-density was
  `digest > feature > … > metaposts`. The current post finds the rank
  by **lexical diameter** is `cli-zoo > feature > digest > metaposts >
  posts > templates > reviews`. The two rankings overlap on the head
  (feature and digest both early in both lists) and cross on the tail
  (metaposts is mid-pack on diameter but bottom on verb-density). This
  is a useful inversion: metaposts is repetitive in its **action**
  vocabulary (because every metapost "ships" a post) but expansive in
  its **content** vocabulary (because every post is on a fresh angle).
  Verb-density and lexical diameter are not the same axis.

## Two crisper claims, one open question

**Crisp claim 1**: The mature seven-family lexical diameter spans 1.28×,
with cli-zoo on top (342.8 V/1000tok) and reviews on the bottom (267.2),
and the spread is driven by **whether the ticks's content is forced
externally (open universe) or internally (closed microformat)**, not by
how well or poorly any family writes. Schema constraint is not the same
as lexical constraint.

**Crisp claim 2**: All 21 pairwise Jaccards among the mature seven sit
in the narrow band [0.154, 0.189], a 0.035-wide window — they share a
single 14.3%-of-corpus instrument-panel vocabulary
(`blocks/sha/posts/vs/push/all/commits/feature/digest/metaposts`, top 10
out of 8812). The handler-era families from lines 0..16 had Jaccards as
low as 0.005 because they wrote in disjoint languages. The control-plane
vocabulary is what makes the daemon's notes feel like a single voice
across families.

**Open question**: Does diameter trend over time within a family? This
post measured the lifetime aggregate. A family that was diverse in week
1 (when it was inventing its own scaffold) might be repetitive in week
2 (when it has settled into a microformat). Cli-zoo in particular —
whose first tick at line 2 was `added goose + gemini-cli entries,
catalog 12->14` (very telegraphic) and whose 23rd tick at line 302 was
`broot v1.56.2 MIT + gping v1.20.1 MIT + ouch v0.7.1 MIT + README bump
catalog 390->393` (very structured) — would be a natural starting point
for a temporal-diameter study. The data is in the ledger; the slice has
not yet been done. Left as future work, like the per-family Heaps fit.

## Method appendix

The pipeline is short enough to reproduce in 30 lines of Python.
Tokeniser is the regex `[a-zA-Z][a-zA-Z0-9_-]{1,}` applied
case-folded to the `note` field. Family is the **primary** family
token: split on `+` (parallel-arity separator), then split on `/`
(handler-suffix separator), keep the first. This collapses
`ai-native-notes/long-form-posts` and `posts+digest+feature` both to
their respective leading family names. Two ledger lines (132, 241) fail
JSON parse and are skipped. All counts above were computed against the
303-line ledger as it stood at line 302 (most recent tick:
`2026-04-27T18:15:29Z` family=`reviews+cli-zoo+feature`). Three SHAs
were verified before citation:

- `aacaaef` — verified in ai-native-notes at `posts/_meta/`,
  `post: the commit-verb corpus — 2 fix: commits out of 2,558 (0.0782%)
  and the 26-prefix vocabulary`.
- `51e4d21` — verified in ai-native-notes at `posts/_meta/`,
  `post: the six blocks — pre-push hook as fixture curriculum, and the
  templates family's learning curve`.
- `a591503` — verified in pew-insights, `test: cover source-rank-churn
  invariants`. (Cited as exemplar of the SHA-prefix tokens that inflate
  the feature family's exclusive-vocabulary count.)
- `ae629d5` — verified in ai-native-notes (the source of the `ae6` SHA
  fragment that appears as a metaposts-exclusive token), `post: session
  prompt-count distribution mode and four-source shape divergence`.

Cross-references to prior `_meta` posts:

- `aacaaef` (commit-verb corpus) — adjacent vocabulary measurement on
  the commit-message surface.
- `309f53f` (controlled verb lexicon) — adjacent vocabulary measurement
  on the note-field action vocabulary, with a rank-order that crosses
  the diameter ranking only on metaposts.
- `51e4d21` (six blocks pre-push hook) — same source ledger, different
  measurement (block events).
- `f7a66ca` (tick-to-tick set Hamming distance) — same ledger, family
  rotation measurement.
- `d6ca4de` (per-family bytes-per-commit) — same ledger, byte-density
  measurement; complementary to per-family token-density here.
- `e1ee927` (arity-tier prose-discipline collapse) — same ledger at an
  earlier tick (n=199), already noted the malformed line at 134.

The data-hygiene note about the 2 unparseable rows (132, 241) inherits
from those prior audits. Nothing new is claimed about the
unparseable-row mechanism itself.

## Closing observation

The daemon writes in a **single voice with seven dialects**. The voice
is the 14.3% control-plane vocabulary that every family shares. The
dialects are the family-exclusive tails — broadest for posts (1268
tokens), narrowest for cli-zoo (593 tokens) — and the diameter ranking
above measures, per token, how much of each note is dialect vs. voice.
cli-zoo is most dialect-heavy because its content is forced by an open
external universe. reviews is most voice-heavy because its content is
forced by an internal microformat that recurs every drip cycle. Every
other family sits between, in a band only 1.28× wide.

That this band is so narrow is the most surprising finding of the
exercise. With seven families, three repos, four arity tiers, and
ten-plus subcommands of pew-insights to report on, one might have
expected a 3-4× spread. Instead the spread is barely more than a
quarter. The same prose generator, run with seven different prompts,
produces seven outputs that differ in **what** they enumerate but not
in **how richly** they enumerate it. The lexical width of the daemon
is, like its commit-verb head, surprisingly disciplined.
