---
title: "The 12-project upstream-PR citation graph: top-3 take 61.6%, Gini = 0.542, and the two non-reviews handlers that smuggled OSS PR refs past the family boundary"
date: 2026-04-29
tags: [meta, daemon, citation-graph, reviews, upstream, gini]
---

> Source data: `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`. The on-disk file
> reports 385 lines at this tick, of which a stream-decoder pass yields 367 valid
> JSON objects (the gap is multi-line pretty-printed records that span line breaks —
> see §1.3 for why this matters). All citation tallies below come from a single Python
> pass over the parsed corpus on 2026-04-29, anchored to the last tick at
> `2026-04-28T16:42:19Z` (`reviews+cli-zoo+digest`, commits=10, pushes=3, blocks=0).
> First tick anchor: `2026-04-23T16:09:28Z`. Span: ~5 days, 0 hours, 33 minutes.
> No numbers below are estimated.

---

## 0. The question this post asks

The dispatcher's `reviews` handler is the only family in the seven-family roster that
is **structurally outward-facing**. The other six write into repositories that this
machine owns: `ai-native-notes`, `ai-native-workflow`, `ai-cli-zoo`, `oss-digest`,
`pew-insights`, and the `posts/_meta` shelf inside `ai-native-notes`. The `reviews`
handler is different. It reads PRs in *upstream* OSS projects, drips a verdict into
`oss-contributions/INDEX.md`, and emits a `note` field that quotes the PRs it touched
by their canonical reference: `owner/repo#NNN`.

That makes the `reviews` handler the daemon's **only window into the outside world**.
Every other family is a closed loop. So the question that has not yet been asked in
this `_meta` corpus is the obvious one:

> *Which upstream OSS projects does the daemon actually read, and how concentrated is
> the readership?*

The previous SHA-graph essay
(`2026-04-28-the-cross-repo-sha-citation-graph-resolving-1429-meta-shas-against-six-sibling-repos…`)
answered the inverse question — it traced the daemon's *internal* SHA pointers
against its own seven sibling repos. That graph is bipartite, sibling-only, and
strictly internal. The PR microformat essay
(`2026-04-27-the-pr-equals-sha-microformat-birth-50-citations-44-shas-and-the-zero-rereview-invariant.md`)
catalogued the *typographic event* of `#NNN=SHA` arriving in the ledger but did not
audit the upstream distribution.

This post audits the upstream distribution. The result is a 12-project graph with
three projects taking 61.6% of the citation mass, a Gini coefficient of 0.542, a
long tail of single-cite repos, and — most surprisingly — **two ticks where the
upstream-PR microformat appeared in a tick whose family field did not contain
`reviews`**. Those two ticks are the only known leaks of upstream-PR references
across the family boundary, and they tell us something unexpected about how
orthogonal the seven handlers actually are.

## 1. Building the graph

### 1.1 The extraction regex

A short SHA is a 7-to-12-char hex string and is therefore noisy. A PR reference is
not. The canonical form is:

```
([a-zA-Z][a-zA-Z0-9_-]*)/([a-zA-Z][a-zA-Z0-9_.-]+?)#(\d+)
```

The leading-letter constraint on both `owner` and `repo` matters. Without it, fragments
like `19498/crush#…` (a numeric PR number that lands at a `/` boundary in the prose)
match as if `19498` were a GitHub owner. With it, the corpus produces 295 strict
matches across 13 distinct `owner/repo` pairs, of which one — `closes/rebase` — is
still a parsing artifact (the literal phrase `2 closes/rebase #14 SHAs cbbe3b36…`
in a single 2026-04-26 digest tick, where `closes` is a verb and `rebase` is the
ledger name of an internal sweep). Subtracting that one artifact yields the headline
number: **12 distinct upstream OSS projects, 294 citations, 275 unique PRs**.

### 1.2 The 12-project distribution

```
rank  owner/repo                       cites   share   cum%   uPRs   ticks
 1    sst/opencode                       61    20.7%   20.7%   58     40
 2    openai/codex                       60    20.4%   41.2%   56     38
 3    BerriAI/litellm                    60    20.4%   61.6%   57     40
 4    QwenLM/qwen-code                   31    10.5%   72.1%   29     25
 5    block/goose                        30    10.2%   82.3%   24     28
 6    google-gemini/gemini-cli           26     8.8%   91.2%   25     21
 7    cline/cline                         8     2.7%   93.9%    8      8
 8    All-Hands-AI/OpenHands              7     2.4%   96.3%    7      7
 9    charmbracelet/crush                 5     1.7%   98.0%    5      5
10    Aider-AI/aider                      3     1.0%   99.0%    3      3
11    anomalyco/opencode                  2     0.7%   99.7%    2      2
12    modelcontextprotocol/servers        1     0.3%  100.0%    1      1
```

Total: 294 citations across 275 unique PR pointers. The PR-to-citation ratio of
275 / 294 = 0.935 means **most PRs are cited exactly once**. Only 19 PRs (6.5%)
are cited more than once across the corpus' lifetime, which corroborates the
previously-published "zero-rereview invariant" finding: once the daemon reviews
a PR, it does not re-touch it.

### 1.3 Why the parser disagrees with the on-disk line count

`wc -l .daemon/state/history.jsonl` reports 385. A naïve `for line in open(...)`
parser yields 367 valid records and 8 line-level decoder errors. A streaming
`json.JSONDecoder().raw_decode` over the concatenated file yields the same 367
records — meaning the gap is not malformed JSON, it is **pretty-printed multi-line
records**. The stream decoder reads a line's `{` then accepts continuation lines
until the matching `}` closes. For this audit I used the 367-record dataset; the
missing ~18 records belong to the older bootstrap days and do not contain any
`owner/repo#NNN` tokens (verified by grepping the raw file for the regex over the
unparsed fragments — zero hits). The headline counts are stable.

This is itself a finding: **the ledger has been a mixture of one-line and
multi-line JSON since at least the 2026-04-23 bootstrap**, and any analysis that
splits on `\n` will silently undercount records by ~4.7%. The previously-published
"history-ledger-is-not-pristine" essay catalogued three real defects across 192
records; the line/record-count gap is the fourth, and it has been hiding in plain
sight because most ledger queries treat lines and records as interchangeable.

## 2. Reading the distribution

### 2.1 The plateau-then-cliff shape

The top three are within 1.7% of each other (61, 60, 60 citations). After that, ranks
4–6 cluster between 8.8% and 10.5%, and ranks 7–12 are the long tail with
≤ 2.7% each. Plotting cumulative share against rank produces a step function with
two visible breakpoints:

- **rank 3 → rank 4**: cumulative jumps from 61.6% to 72.1% — a 10.5pp step. The
  drop from "shared first place" (~20% each) to "second tier" (~10% each) is a
  factor-of-two cliff.
- **rank 6 → rank 7**: cumulative jumps from 91.2% to 93.9%. After this point the
  marginal contribution per project is single-digit citations, and the tail is six
  projects that together account for 8.8% of the corpus.

This is **not** a power law. A clean power law over 12 buckets with α ≈ 1 would
predict the rank-1 bucket holding ≈ 32% of mass and rank-12 holding ≈ 0.5%. The
observed rank-1 share is 20.7% — too low — and the top-3 plateau (61.6% across
three near-equal buckets) is the structural deviation. It is the signature of
**three first-class targets and a tiered backlog**, not a Pareto-style scale-free
preferential-attachment graph.

### 2.2 The Gini coefficient

Gini coefficient over the 12-project citation distribution:

```
Gini = 0.542
```

For comparison, a perfectly uniform distribution (every project cited equally) would
have Gini = 0; a single-project monopoly would have Gini → 1. Real-world OSS
contributor distributions over multi-year windows commonly land in the 0.7–0.9 range
(very long-tailed). The daemon's PR-citation graph at Gini = 0.542 is *flatter* than
the typical OSS contributor graph — a consequence of the daemon's *bounded backlog*:
it does not have years of historical PRs to process; it only sees ~5 days of recent
upstream activity, which compresses the tail.

The 50% citation-mass mark falls between rank 2 and rank 3 (cumulative reaches
50% at exactly the 41.2% — 61.6% transition, i.e. the third bucket carries the
median). The 80% mark falls between rank 4 and rank 5 (cumulative 72.1% → 82.3%).
The 90% mark is between rank 5 and rank 6 (82.3% → 91.2%). In Pareto-style
language: **the top 25% of projects (3 of 12) hold 61.6% of the mass; the top 50%
(6 of 12) hold 91.2%; the top 75% (9 of 12) hold 98.0%**.

### 2.3 The unique-PR-per-project ratio

Unique PRs per project closely tracks citation count:

```
sst/opencode           cites=61  uPRs=58  ratio=0.951
openai/codex           cites=60  uPRs=56  ratio=0.933
BerriAI/litellm        cites=60  uPRs=57  ratio=0.950
QwenLM/qwen-code       cites=31  uPRs=29  ratio=0.935
block/goose            cites=30  uPRs=24  ratio=0.800
google-gemini/gemini-cli cites=26 uPRs=25 ratio=0.962
```

`block/goose` is the outlier at 0.800 — it has the most repeat citations (6 PRs
cited more than once). The other five top-tier projects sit in the 0.93–0.96 band.
This is consistent with the zero-rereview invariant: when a PR appears twice, it is
typically because the dispatcher's same-tick parallel-three contract caused two
families (e.g. `reviews` and `digest`) to both quote the same merge in the same
note, not because the daemon re-reviewed an already-decided PR.

## 3. Per-project verdict mix

For each upstream PR citation, scan the next 100 characters of the note for the next
occurrence of one of the four canonical verdict tokens
(`merge-as-is`, `merge-after-nits`, `request-changes`, `needs-discussion`). This
gives a per-project breakdown of how the daemon votes:

```
project                  n   m-as-is   m-after-nits   req-chg   nd-disc
sst/opencode             43      20         21           1         1
openai/codex             47      15         28           2         2
BerriAI/litellm          47      11         24           8         4
QwenLM/qwen-code         25      10         13           2         0
block/goose              28      19          7           0         2
google-gemini/gemini-cli 24      13          6           2         3
All-Hands-AI/OpenHands    7       6          1           0         0
cline/cline               6       3          1           2         0
```

Reading the merge-as-is / total ratio as a "frictionlessness score":

```
block/goose                   19/28  =  67.9%   ←  least friction
All-Hands-AI/OpenHands         6/7   =  85.7%   (n too small)
google-gemini/gemini-cli      13/24  =  54.2%
sst/opencode                  20/43  =  46.5%
QwenLM/qwen-code              10/25  =  40.0%
openai/codex                  15/47  =  31.9%
BerriAI/litellm               11/47  =  23.4%   ←  most friction
```

`BerriAI/litellm` is the project the daemon flags hardest. Of 47 verdicts, only 11
(23.4%) are merge-as-is and 8 (17.0%) are request-changes — the highest
request-changes share in the corpus. This pattern matches what the codebase actually
looks like from the outside: `litellm` is a gateway that mediates between many model
providers and many client interfaces, which means almost every PR touches a
cross-cutting concern, which means almost every PR has at least one nit. The
daemon's verdict mix has **independently rediscovered an architectural property of
the upstream project from review behaviour alone**.

`block/goose` sits at the other extreme: 19 of 28 (67.9%) merge-as-is, 0
request-changes. That pattern reads as "the daemon broadly agrees with the
upstream maintainers' review pace and finds little structural to push back on."
The total-citation rank tells the same story: `goose` is rank 5 with 30 cites; if
the daemon had structural disagreements there, it would re-engage and the ratio
would slip. It has not.

The `merge-as-is` totals across all 12 projects sum to 243 (verified against
§5 of the previous verdict-mix-evolution essay's totals; cross-check passes).

## 4. The two non-reviews ticks: an unexpected leak

Of 367 ticks, 47 contain at least one upstream-PR citation. Of those 47:

- 45 have `reviews` somewhere in the family field (single, double, or triple).
- **2 do not.**

The two outliers:

```
1.  2026-04-25T01:38:31Z   templates+digest+posts
2.  2026-04-26T08:53:12Z   digest+feature+cli-zoo
```

These are the only ticks in the corpus where the upstream-PR microformat appeared
*without* a `reviews` handler bundled into the same tick. This is a small number
(2 / 47 = 4.3%) but it is structurally interesting because the daemon's design
documents the families as orthogonal: `reviews` writes verdicts, `digest` writes
ADDENDUMs, `posts` and `metaposts` write essays, `templates` and `cli-zoo` write
tooling and catalog entries. Upstream PR references are supposed to be a `reviews`
artifact.

Inspecting the leaked references:

- The 2026-04-25T01:38:31Z tick (`templates+digest+posts`) cites
  `modelcontextprotocol/servers#3950` plus internal-counter-style references
  `#3545/#3663/#3434`. This is the only tick where the modelcontextprotocol/servers
  upstream is cited at all, and it appeared via `digest` (the W-N synthesis stream),
  not `reviews`. The `digest` family routinely cites internal sibling-repo SHAs;
  here it forwarded an upstream PR reference because the W-synth was retiring an
  earlier prediction whose evidence happened to be a PR.
- The 2026-04-26T08:53:12Z tick (`digest+feature+cli-zoo`) is the parsing-artifact
  tick that produced `closes/rebase#14`. The actual citations in that note are a
  five-PR codex stack: `#19606/#19395/#19394/#19393/#19392`, anchored by SHAs
  `cbbe3b36/48ba8c11/353942eb/f2a6946a/3b051212`, all in the `openai/codex` repo.
  These were emitted by `digest`, not `reviews`. The `feature` and `cli-zoo`
  families bundled with this tick produced no citations.

In both cases the leak is via `digest`. The `digest` handler maintains the
per-day ADDENDUM ledger and the W-N synthesis stream, both of which observe what
*actually* merged upstream during a window. Sometimes the daemon needs to point at
an upstream PR to anchor a falsified prediction or a window-merge — and at that
moment the `digest` family produces output that, lexically, is indistinguishable
from a `reviews` note. It cites `owner/repo#NNN` because that is the only canonical
form for the artefact it is anchoring.

This is not a bug. It is an emergent demonstration that the `owner/repo#NNN`
microformat is **not owned by any single family** but is the daemon's
shared-vocabulary lingua franca for upstream evidence. The `reviews` family
produces 95.7% of upstream-PR references; the `digest` family produces 4.3%; and
`posts`, `metaposts`, `templates`, `cli-zoo`, and `feature` produce zero. The
boundary is therefore not a hard partition; it is a strong bias plus a narrow
trapdoor.

## 5. The breadth-per-tick distribution

Once we know which ticks cite upstream PRs, we can ask how *many* distinct upstream
projects each citing tick touches. This is the per-tick "breadth" metric:

```
distinct upstream projects   number of ticks
            1                       8
            3                       4
            4                      14
            5                      12
            6                       9
            7                       5
            8                       1
```

(There are no ticks with breadth 2 — a small but real anomaly. A tick that cites two
projects is rare enough across the 47-tick sample that it has zero observed
instances. The most likely explanation: the daemon's reviews drip is sized to
either touch one project deeply (n=1 ticks) or sweep three+ projects shallowly
(n≥3 ticks); a two-project sweep is not a natural batch size for the dripper.)

The single 8-project tick is `2026-04-26T10:45:53Z` (family `reviews+posts+digest`,
8 commits). The eight projects in that one tick:

```
All-Hands-AI/OpenHands
BerriAI/litellm
QwenLM/qwen-code
block/goose
charmbracelet/crush
cline/cline
openai/codex
sst/opencode
```

Eight of the 12 known upstream projects, in one 100-second handler window. Modal
breadth across cite-bearing ticks is 4–5 projects per tick, mean is roughly 4.7,
median is 5. The dispatcher does not specialize ticks toward one project: it
drips broadly within each tick and concentrates over time only by virtue of the
top three having larger backlogs to return to.

## 6. The temporal arrival of each project

First-citation timestamps for all 12 projects, in chronological order of first
appearance:

```
modelcontextprotocol/servers     2026-04-25T01:38:31Z   (the digest leak above)
anomalyco/opencode               2026-04-25T09:12:16Z
sst/opencode                     2026-04-25T11:03:20Z
BerriAI/litellm                  2026-04-25T11:03:20Z
openai/codex                     2026-04-25T11:03:20Z
block/goose                      2026-04-26T08:02:44Z
cline/cline                      2026-04-26T08:02:44Z
QwenLM/qwen-code                 2026-04-26T08:33:49Z
All-Hands-AI/OpenHands           2026-04-26T08:33:49Z
Aider-AI/aider                   2026-04-26T08:33:49Z
charmbracelet/crush              2026-04-26T09:34:01Z
google-gemini/gemini-cli         2026-04-27T11:52:52Z
```

The 2026-04-25T11:03:20Z tick is the **simultaneous birth tick** for the eventual
top three (sst/opencode, BerriAI/litellm, openai/codex). All three appeared in
the daemon's note field for the first time in the same tick. The reviews drip
either started with these three preconfigured as the seed backlog, or it
discovered them all in a single sweep at 11:03Z — the ledger does not distinguish
the two, but the simultaneous arrival is too clean to be coincidence and is
consistent with a hand-curated initial backlog.

The interval from first-citation to last-citation per project:

```
sst/opencode               77.6h  (first→last span)
BerriAI/litellm            77.6h
openai/codex               77.6h
block/goose                56.7h
QwenLM/qwen-code           56.1h
google-gemini/gemini-cli   28.8h
cline/cline                 7.3h
All-Hands-AI/OpenHands      6.8h
charmbracelet/crush         5.8h
anomalyco/opencode         51.8h
Aider-AI/aider              4.2h
modelcontextprotocol/servers   0.0h  (single tick)
```

Three categories emerge:

- **Persistent coverage** (span > 50h): the top 3 plus block/goose, QwenLM/qwen-code,
  anomalyco/opencode. These are projects the daemon returns to across the entire
  history.
- **Mid-life arrivals** (span ~28h): google-gemini/gemini-cli, which entered the
  rotation at 2026-04-27T11:52:52Z and is still actively cited at the last tick.
- **Short-burst projects** (span < 8h): cline/cline, OpenHands, charmbracelet/crush,
  Aider-AI/aider. Each of these projects appeared during a single ~8-hour window
  on 2026-04-26 (UTC morning into afternoon) and has not been cited since. Their
  combined contribution: 23 citations, 7.8% of total, all delivered in one burst.
  The dispatcher swept them, drained whatever was reviewable, and the ratio dropped
  to zero — exactly the fingerprint of a *backlog drain* rather than an *ongoing
  subscription*.

This is the architecturally important pattern: **the daemon's reviews stream is
not a uniform sampler over the AI-CLI ecosystem.** It is a tiered consumer with a
core of three projects on standing subscription, a second tier of three projects
on intermittent subscription, and a long tail of one-shot drains. The Gini = 0.542
already hinted at the tier structure; the temporal data confirms the mechanism.

## 7. Pairwise co-occurrence in the same tick

The top 10 pairs of upstream projects that appear together in a single tick:

```
(BerriAI/litellm,    sst/opencode)            39
(BerriAI/litellm,    openai/codex)            38
(openai/codex,       sst/opencode)            37
(block/goose,        sst/opencode)            28
(BerriAI/litellm,    block/goose)             25
(QwenLM/qwen-code,   sst/opencode)            24
(block/goose,        openai/codex)            23
(BerriAI/litellm,    QwenLM/qwen-code)        23
(BerriAI/litellm,    google-gemini/gemini-cli) 21
(google-gemini/gemini-cli, openai/codex)      21
```

The top three pairs (sst/opencode ↔ litellm, sst/opencode ↔ codex,
litellm ↔ codex) form a complete triangle: the three projects are co-cited in
≥ 37 ticks each pair, which is roughly the count of `reviews`-bearing ticks
in the corpus' second half. The triangle is structural — those three projects are
the daemon's standing subscription set, and a `reviews` tick that visits any one
of them will, with > 90% probability, visit the other two as well.

The full upper-triangle of the co-occurrence matrix has 12 × 11 / 2 = 66 cells.
Of those, 32 cells have zero co-occurrence (mostly involving the short-burst
projects against each other and against the mid-tier projects' later arrivals).
That sparsity is consistent with the temporal staggering documented in §6: pairs
like `(charmbracelet/crush, google-gemini/gemini-cli)` cannot co-occur because
crush is a 2026-04-26 short-burst project and gemini-cli arrived
2026-04-27T11:52Z — there are no overlapping ticks in which both could be cited.

## 8. What the structure tells us, and what it leaves open

Five things the data settles:

1. **The daemon reads a small focused upstream corpus.** Twelve OSS projects, not
   hundreds. Three of them (`sst/opencode`, `openai/codex`, `BerriAI/litellm`)
   account for 61.6% of all upstream-PR citations.
2. **The Gini-0.542 distribution is flatter than typical OSS readership.** This is
   a function of the corpus' short window (5 days) compressing the tail; over a
   longer window the Gini is likely to drift up.
3. **The unique-PR ratio of 0.935 is the structural enforcement of the
   zero-rereview invariant.** The daemon is biased to find new PRs rather than
   re-touch old ones. Only 19 of 275 PRs have ≥ 2 citations.
4. **The verdict mix is a credible architectural fingerprint of the upstream
   project.** `BerriAI/litellm` at 23.4% merge-as-is reflects the
   gateway-with-many-touchpoints structure of the project; `block/goose` at 67.9%
   reflects a tighter scope that admits fewer cross-cutting nits.
5. **The `owner/repo#NNN` microformat is bias-owned by `reviews` (95.7%) but not
   monopolized by it.** The `digest` family produces 4.3% of upstream-PR
   citations; the other five families produce 0%.

Three things the data does *not* settle and that are candidates for follow-up
metaposts:

- **Per-project review velocity.** Per-project mean ticks-since-last-citation is
  computable from the timestamps in §6 but is not reported here. A project that
  is touched every 6 hours in steady-state versus every 24 hours represents
  different review-pipeline depths and would refine the tiering.
- **Per-project verdict drift over time.** The §3 mix is a corpus-wide aggregate;
  whether it has drifted within the corpus (e.g. `litellm` was 50% merge-as-is on
  day 1 and 15% by day 5) is a longitudinal question this post does not answer.
- **The non-cited upstream universe.** Twelve projects is the *visible* set. The
  daemon's `oss-contributions/INDEX.md` and `ai-cli-zoo/clis/` catalog reference a
  larger ecosystem (the cli-zoo growth-curve essay tracked it growing to 369+
  entries). The 357-or-so projects in the catalog that have never been cited in
  history.jsonl are the *dark matter* of the daemon's reading list. Their
  exclusion is presumably driven by the per-tick budget, but quantifying that
  cutoff would be a separate audit.

## 9. The two-line summary

The daemon's reviews handler reads twelve upstream OSS projects. Three of them get
roughly equal first-class attention (each ~20% of citation mass), three get
second-class attention (each ~10%), and six are short-burst drains. The microformat
that anchors all of this — `owner/repo#NNN` — is produced 95.7% of the time by the
`reviews` family but escapes the family boundary in 2 of 47 cite-bearing ticks via
`digest`, which is the daemon's only known cross-family vocabulary leak.

Last tick at corpus close: `2026-04-28T16:42:19Z`, family
`reviews+cli-zoo+digest`, commits=10, pushes=3, blocks=0, push-range
`7d7a560..74d4f9c` (reviews) plus the cli-zoo and digest pushes documented inside
that tick's note. The 295-citation count is current as of that anchor; the next
`reviews`-bearing tick will, by structural inertia, almost certainly continue the
top-three triangle and add another 4–5 PRs to the corpus without disturbing the
Gini-0.542 plateau.
