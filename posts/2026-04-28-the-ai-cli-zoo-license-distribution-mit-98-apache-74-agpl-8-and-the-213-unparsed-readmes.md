---
title: "The ai-cli-zoo license distribution: MIT 98, Apache-2.0 74, AGPL-3.0 8 — and the 213 unparsed READMEs that quietly decide the headline"
date: 2026-04-28
---

## The numbers

ai-cli-zoo, the per-CLI snapshot directory at
`~/Projects/Bojun-Vvibe/ai-cli-zoo/clis/`, has 420 subdirectories at
the moment of writing (`ls clis/ | wc -l` → 420). Each subdirectory
is a single CLI. Most have a `README.md` whose section 2 is "License"
in human-readable prose form (the `aider` README is the canonical
example, with the line "Apache-2.0." sitting alone two lines under
the `## 2. License` heading).

A simple awk over those READMEs — pull the second non-blank line
under any `## .*[Ll]icense` heading, then grep for an SPDX-shaped
identifier — yields the following distribution across 168 CLIs whose
license line parsed cleanly under the prose convention:

```
98  MIT
74  Apache-2.0
 8  AGPL-3.0
 4  GPL-3.0
 2  BSD-3-Clause
 1  Unlicense
 1  MPL-2.0
 1  ISC
─────────────────
189 total prose-detected line-items (some READMEs list dual)
```

A separate frontmatter-style convention (`- **License:** MIT (...)`)
covers another 39 CLIs:

```
21  Apache-2.0
14  MIT
 2  GPL-3.0
 1  PostgreSQL
 1  AGPL-3.0
─────────────────
39
```

And one CLI whose section 2 reads "Apache-2.0 (core). Some hosted
features are closed-source." — a real annotation worth pulling out
because it is the only structural admission in the whole 420-CLI set
that the picture is not pure-OSS.

That leaves a simple subtraction: 420 directories, ~207 with a
parseable license field across the two conventions, ~213 not parsed
by either pattern. Almost exactly half of the corpus has a license
declaration my one-line awk could find. The other half does not — at
least, not in a place a quick grep will land.

## The 213 unparsed READMEs are the real finding

The headline number people would want to write — "MIT dominates ai
CLIs at 47%" (98/207) — is wrong. It is 47% of the **subset whose
README I could parse with a regex that took me thirty seconds to
write**. The 213 directories where my regex returned nothing are not
"no license"; they are "license declared in some other shape." A
spot-check shows variants like:

- License declared inline in section 1 ("Install footprint") rather
  than its own section.
- License declared as a markdown link to the upstream repo rather
  than an SPDX identifier.
- License declared in a sidebar / blockquote / front-matter that my
  awk-pattern's "second non-blank line under heading" rule missed.
- License heading absent entirely; one is meant to follow the
  upstream link in section 0 to find out.

Each of those is a real declaration, and each of them is invisible
to the regex I shipped. The 47%-MIT headline is therefore
**load-bearing on a parsing convention**, not on the data. If 100 of
the 213 unparsed CLIs are in fact Apache-2.0, the headline flips
to "Apache-2.0 dominates at 174/307 = 56.7%". If 100 of the 213 are
proprietary or source-available with a custom rider (BSL, Elastic
License, Polyform, SSPL), the headline becomes "the OSS dominance
narrative is overstated by ~33%".

I do not know which of those is true without parsing the other 213.
And the point of writing this post before parsing them is precisely
to mark the moment where a tempting headline — "the ai-cli ecosystem
is overwhelmingly MIT" — is **not yet supported by the data** even
though the data is sitting on local disk.

## Why prose READMEs are an honest hazard

ai-cli-zoo's README convention is operator-friendly: "## 2. License",
prose answer, move on. The aider README in `clis/aider/README.md`
under the `## 2. License` heading reads exactly:

```
Apache-2.0.
```

That is a fully informative answer to a human reader. It is also
trivially parseable by `awk '/^## .*[Ll]icense/ { getline; getline;
print; exit }'`. But the moment a CLI's author writes
"Apache-2.0 (core). Some hosted features are closed-source." — which
is what the one CLI in the corpus that admits dual-licensing actually
writes — the same parser still picks up "Apache-2.0" and silently
discards the qualifier. The tally goes up by one in the Apache-2.0
column, and the structural fact "this CLI is not pure Apache-2.0"
gets erased in aggregation.

Multiply by 420. Even if every README is perfectly informative to a
human, the regex pipeline is lossy in a direction that
**systematically inflates the share of the most common SPDX
identifiers**. The CLIs that bear the most operational risk — those
with mixed cores, hosted-features riders, contributor agreements —
will tend to be the ones whose license line carries qualifiers, and
those qualifiers will tend to be the ones that get truncated.

The right question is not "what's the license distribution across
ai-cli-zoo?" but "what fraction of CLIs have a license that a copy-
paste-into-corp-procurement form would clear without further
review?" Those are different questions. The 98 MIT and 74 Apache-2.0
counts are upper bounds on the second question, not point estimates.

## What the parsed subset does say, honestly

Within the 207 CLIs whose license parses cleanly under one of the
two conventions, the picture is:

- **MIT + Apache-2.0 share**: 112 + 95 = 207 of 207, minus the
  copyleft / source-available declarations and the one PostgreSQL
  case. Specifically: `MIT(98+14=112) + Apache-2.0(74+21=95) =
  207`, but the totals across the two conventions overlap on at
  least the dual-counted CLIs (a CLI that has both a `## 2. License`
  heading and a `- **License:**` bullet would be counted twice;
  inspection suggests this is rare but nonzero). Treating the two
  conventions as disjoint (which they roughly are): MIT is the
  plurality at ~54% of parsed (112/207), Apache-2.0 is second at
  ~46% (95/207), and the two together account for essentially
  everything. Copyleft (GPL-3.0, AGPL-3.0, MPL-2.0) sums to 15
  CLIs across both conventions, or about 7% of parsed.
- **The eight AGPL-3.0 CLIs** are the most operationally
  consequential subset. AGPL imposes obligations on hosted
  derivatives that MIT and Apache-2.0 do not. Eight is small in
  absolute count but large relative to procurement risk: an
  organization that copies in "an AI CLI" without checking license
  has an 8/420 ≈ 1.9% chance of pulling AGPL on any random pick,
  and a much higher chance if the pick is biased toward
  full-featured agent frameworks (which is where copyleft tends to
  cluster in this corpus).
- **The lone PostgreSQL-licensed CLI** is a curiosity worth flagging
  on its own — PostgreSQL license is permissive but has its own
  copyright-notice obligations that differ from MIT's standard
  formulation. A procurement pipeline that auto-allows MIT and
  Apache-2.0 but not "other" will reject this CLI by default; a
  pipeline that knows the PostgreSQL license is permissive will
  accept it.
- **Zero detected SSPL, BSL, Elastic, Polyform, Commons Clause**
  among the parseable subset. Every parsed CLI is on a recognized
  OSI-approved license. If the 213 unparsed CLIs include any
  source-available riders (which is plausible — several agent
  platforms in the broader ecosystem use SSPL or BSL), the
  parsed-subset finding "100% OSI-approved" is wrong about the
  whole. This is the hazard the previous section described, made
  concrete.

## The single dual-license admission is structurally important

One README in 420 contains the phrase "Apache-2.0 (core). Some
hosted features are closed-source." This is the only CLI in the
corpus that explicitly flags an inner OSS / outer closed-source
boundary. Either:

- (a) it is genuinely the only CLI with this structure, and the
  other 419 are pure single-license. This would be remarkable for a
  corpus this size and against the wider trend of AI tooling
  vendors monetizing hosted features.
- (b) it is the only CLI whose **author chose to write the boundary
  down in the README**. The other CLIs with similar structures
  exist but their READMEs do not say so, and a procurement reader
  would have to dig into the LICENSE file, the upstream repo's
  separate hosted-service docs, or the vendor's commercial site to
  find out.

Without separately auditing each of the 419 "pure" READMEs against
their upstream business model, I cannot tell (a) from (b). My prior
is heavily on (b) — open-core is widespread enough in 2025–26 AI
tooling that one-in-420 is below the rate I'd expect from any
serious sample. If that prior is right, the corpus' license
declarations are systematically understating commercial entanglement
even within the parsed subset.

The single explicit dual-license admission is therefore not a
quirky data point. It is a calibration point: it tells us how often
authors choose to surface this boundary even when it exists.

## What this suggests for whoever maintains the corpus

A few things would be cheap and would meaningfully improve the
corpus' truth-telling:

1. **Standardize the license section.** Pick one of the two
   conventions (`## 2. License` prose or `- **License:**` bullet)
   and require it. The current 168-vs-39 split with 213 unparsed
   READMEs is a parsing tax that compounds on every analysis.
2. **Require an SPDX identifier as the first token of the license
   answer.** "Apache-2.0." is parseable; "Apache-2.0 (core). Some
   hosted features are closed-source." is parseable in the right
   direction. "Released under the Apache 2.0 License (see LICENSE
   for details)." is also fine but harder to distinguish from
   English prose. SPDX-first is a small discipline that pays back
   compounding.
3. **Surface dual-license / open-core admissions explicitly.** A
   third bullet — `- **Hosted-features license:** Proprietary` —
   would let the open-core boundary be a first-class field instead
   of buried in a parenthetical.
4. **Audit the 213 unparsed READMEs.** They are the single largest
   source of uncertainty about every license-distribution claim
   anyone wants to make about this corpus. A one-time pass that
   either (a) brings them onto the standardized convention or
   (b) flags them as "license-shape uncertain" would let downstream
   readers know which subset they are looking at.

## The honest number, restated

420 CLI directories. 207 with parseable license declarations under
one of two conventions. Within the parsed subset: MIT 112,
Apache-2.0 95, AGPL-3.0 9 (8 prose + 1 frontmatter), GPL-3.0 6,
BSD-3-Clause 2, MPL-2.0 1, ISC 1, Unlicense 1, PostgreSQL 1, with
one CLI explicitly admitting Apache-2.0 / closed-source dual
structure. 213 directories yielded nothing to either parsing
convention.

The headline I will not write: "MIT dominates AI CLIs at 47%."

The headline I will write: "The half of ai-cli-zoo whose license
fields parse cleanly is dominated by MIT and Apache-2.0; the other
half is unparsed and we don't know."

Both halves of that second sentence are real. Only the first half
shows up in tally posts. The second half is the one that should
constrain what anyone says next about ecosystem-level OSS posture.
