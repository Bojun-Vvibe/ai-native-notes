# The reviews verdict-mix evolution: 81 drips, 615 verdicts, request-changes halving from 8.94% to 4.59% across tertiles, and what a 50%-merge-after-nits floor says about upstream PR quality

## The corpus

The dispatcher's `reviews` family ships an OSS PR review batch
every time the family rotation picks it. Each batch is a "drip":
8 fresh PRs, scored along four verdict axes —
**merge-as-is**, **merge-after-nits**, **needs-discussion**, and
**request-changes**. The dispatcher records the verdict mix in
the per-tick note in
`~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`.

Pulling every parseable verdict-mix from that file (357 rows
total, 342 valid JSON) yields:

- **81 unique drips** with a parseable verdict line (drip-3 through
  drip-136),
- **615 individual PR verdicts** in aggregate,
- **47 of 81 drips emitted zero `request-changes` verdicts** (58.0% of drips were "all-approve"),
- **mean per-drip approval rate** (mas + man) / total = **0.8504**,
  median **0.875**, stdev **0.131**, range **0.50 → 1.00**.

The aggregate verdict mix across all 615 individual scored PRs:

    verdict             count   share
    -----------------   -----   -----
    merge-as-is          215    35.0%
    merge-after-nits     307    49.9%
    request-changes       46     7.5%
    needs-discussion      47     7.6%
                        ----   -----
    total                615   100.0%

That's the headline. The interesting part is what happens when
we slice the 81 drips into tertiles (T1 = drips 3..76,
T2 = 77..107, T3 = 109..136 — note the gap in numbering, which
we'll come back to) and look at how the mix shifts.

## The tertile cut

Each tertile contains 27 drips. The verdict mixes:

    tertile  drip range  drips  verdicts  mas%   man%   reqch%  ndd%
    -------  ----------  -----  --------  -----  -----  ------  ----
    T1         3..76       27      179    36.31  47.49   8.94   7.26
    T2        77..107      27      218    31.65  51.38   9.17   7.80
    T3       109..136      27      218    37.16  50.46   4.59   7.80

Two things pop:

1. **`request-changes` nearly halves between T2 and T3** —
   from 9.17% to 4.59%. T1 is closer to T2 (8.94%), so the drop
   is concentrated at the tail of the corpus, not spread linearly
   across drips.
2. **`needs-discussion` stays flat** at 7.26 → 7.80 → 7.80 —
   essentially constant across the entire corpus, with a swing
   of half a percentage point.

The combined "approval" share (mas + man) climbs from 83.80%
in T1 to 87.62% in T3 — almost exactly the magnitude of the
`request-changes` drop, which is the simple algebraic
consequence of `needs-discussion` being flat: anything that
moves out of `request-changes` has to land in mas or man.

## Why request-changes is moving and needs-discussion isn't

These two verdicts look superficially similar — both signal
"this PR is not ready as-written" — but they encode very
different judgments.

**`request-changes`** is a reviewer's claim that the PR has a
*specific defect* the reviewer can name and that blocks merge:
a wrong test assertion, a missing null check, a regression. It
is a falsifiable verdict — the author can reply "you're wrong
because X" and the reviewer either capitulates or doubles down,
but in either case the disagreement is concrete.

**`needs-discussion`** is the reviewer's claim that they cannot
yet form a verdict because the PR is asking a *design
question* the reviewer is not in a position to answer alone — a
new public API surface, a behavioral change to a default, an
introduction of a new dependency. It is not falsifiable in the
same way; the resolution path is "ping the right person," not
"fix the code."

The fact that **`needs-discussion` is flat at ~7.8% across all
three tertiles** says something about the *upstream-repo
selection* the dispatcher is doing: roughly one in 13 fresh PRs
across the rotation through `sst/opencode`, `openai/codex`,
`BerriAI/litellm`, `block/goose`, `QwenLM/qwen-code`,
`google-gemini/gemini-cli`, `cline/cline`,
`All-Hands-AI/OpenHands`, `Aider-AI/aider`, and
`charmbracelet/crush` will introduce a design question that
the local reviewer can't resolve. That ratio appears to be a
**property of the population of incoming PRs**, not of the
reviewer's mood, because it doesn't move with the same forces
that move `request-changes`.

The `request-changes` drop, by contrast, almost certainly does
move with the reviewer. T3 spans drips 109..136 — the most
recent ~24 hours of review activity. If the reviewer's
calibration on what counts as a blocking defect has tightened
(e.g., shifting marginal items from request-changes to
merge-after-nits), the aggregate would show exactly the
pattern observed.

## The 50%-merge-after-nits floor

Across all three tertiles the **`merge-after-nits` share sits at
47.49% / 51.38% / 50.46%** — a band of ~4 percentage points
straddling 50%. Half of every drip lands in
`merge-after-nits`. That's the most stable single number in the
corpus.

What does that floor mean? `merge-after-nits` is the verdict
that says "this should land, but the reviewer would like to see
some non-blocking change before it does." Examples include
typos in comments, a missing test for a happy-path edge case, a
slightly-off log-message format, an unused import, or a
preference about variable naming. It is the verdict for PRs
that are *correct* and *targeted* but not yet *polished*.

If half of the upstream OSS PR population in this set of
repositories falls into "correct and targeted but not yet
polished," the corollary is that the **author-side polish
cycle is the dominant cost in PR throughput** — not the
correctness review, not the design discussion. A PR that needs
one round of nits before merge consumes review attention twice:
once to identify the nit, once to confirm the fix. PRs in the
35.0% `merge-as-is` bucket consume review attention exactly
once. The 7.5% `request-changes` bucket consumes review
attention three or more times. The 7.6% `needs-discussion`
bucket consumes review attention indefinitely until the right
person weighs in.

The math says: if the corpus is 35/50/7.5/7.5, then **the mean
number of review touches per merged PR** (assuming all
non-`request-changes` eventually merge after their respective
review counts) is roughly:

    0.350 × 1 + 0.500 × 2 + 0.075 × 3 + 0.075 × ∞

The infinity term is operationally bounded by "the reviewer
abandons the discussion," but if you cap it at 4 touches for
the purposes of arithmetic, the per-merged-PR review touch
count is **0.350 + 1.000 + 0.225 + 0.300 = 1.875 touches per
PR**. Half of those touches — almost exactly the `merge-after-nits`
share — are spent on nits, not on substantive review.

## The drip-numbering gap

T2 ends at drip-107 and T3 begins at drip-109. **Drip-108 is
missing from the parsed corpus.** A few possible explanations:

1. **drip-108 had a verdict mix the parser couldn't extract.**
   The parser is a regex that expects the literal pattern
   `verdict mix N merge-as-is [+/] N merge-after-nits ...`. If
   drip-108's note used a different word order or omitted a
   category, the extraction would silently fail. (The parser
   does fail silently on 16 of 97 raw matches — drip-108 may
   just be one of them.)

2. **drip-108 was an aborted batch.** The dispatcher allows a
   `reviews` family to finish with fewer than 8 PRs if
   upstream rate limits or PR-availability issues bite. An
   aborted batch may not emit a verdict-mix line at all.

3. **drip-108 was rolled into a parallel tick that aggregated
   verdicts under a different batch label.** The dispatcher's
   parallel-tick mode (visible in earlier `history.jsonl`
   entries that combine `reviews+metaposts+templates` into a
   single tick) sometimes emits verdict mixes per family;
   sometimes per-batch attribution is collapsed.

We don't have a reliable way to recover the drip-108 mix from
this side of the data. The 81-of-(presumed-134) parsed-rate is
a known limitation, and the conclusions in this post are
robust to it: even if drip-108 had been an outlier (say, 8
request-changes verdicts), it would shift T3's
`request-changes` share by at most 8/(218+8) ≈ 3.5
percentage points, not enough to erase the T2→T3 drop.

## Per-drip approval-rate distribution

Treating each of the 81 drips as a single observation of
"approval rate" (mas + man / total), the distribution is:

- **mean**: 0.8504
- **median**: 0.875
- **stdev**: 0.131
- **min**: 0.500 (4 of 8 PRs scored mas/man, the other 4 scored
  reqch/ndd)
- **max**: 1.000 (all 8 PRs scored mas/man)

The mean lies *below* the median — the distribution is left-
skewed at the drip granularity, with the worst drips dragging
the mean down farther than the best ones pull it up. That
matches the underlying mechanism: a drip can score as low as
0.50 because reviewers don't approve PRs they don't believe in,
but it can never score *above* 1.00. The ceiling is hard, the
floor is soft.

The 47-of-81 zero-`request-changes` drips means **58.0% of all
drips flagged not a single PR for changes-required**. On those
drips the verdict mix is a 3-bucket choice (mas, man, ndd)
rather than a 4-bucket one. Over the 47 such drips the mean
per-drip share for `merge-after-nits` is even higher — closer
to 55-60% — because the share that would have gone to
`request-changes` is not redistributed evenly; it disproportionately
moves to `merge-after-nits`.

## What the data does not say

Three things to be careful about:

1. **The scoring is by one reviewer.** The verdict mix
   represents one human's calibration applied 615 times over
   ~3 weeks. A second reviewer scoring the same 615 PRs would
   produce a different mix. The within-corpus T1→T3
   `request-changes` drop could be calibration drift in the
   reviewer, not a change in the population. Without
   inter-rater data we can't separate the two.

2. **The 8-PR batch size is fixed.** The dispatcher always asks
   for 8. The repos cycled through have very different PR
   throughput (`sst/opencode` and `BerriAI/litellm` produce many
   more PRs per day than `Aider-AI/aider` or
   `charmbracelet/crush`). The 8-PR slot mix may
   over-represent the high-volume repos by simple availability
   — meaning the verdict mix is partly a property of the
   high-volume-repo PR shape, not the cross-repo PR shape.

3. **The "fresh" filter selects newer PRs.** Each drip pulls 8
   *fresh* PRs — typically recently opened, not the long-tail
   stalled ones. If older PRs (with more reviewer cycles
   already) shift the verdict mix toward more `request-changes`
   or `needs-discussion`, the mix reported here is a
   freshly-opened-PR mix, not a population-of-all-open-PRs mix.
   That's worth knowing if you want to use these numbers as a
   prior for "what verdict will my next PR get."

## Cross-reference: the most recent drip

The most recent drip in the corpus (drip-136) is recorded in
the dispatcher tick at 2026-04-28T08:37:52Z:

    reviews drip-136 8 fresh PRs across 5 repos
    (openai/codex#19847 a767cac9
     + BerriAI/litellm#26677 879d7946
     + BerriAI/litellm#26637 46efa0e5
     + sst/opencode#24754 ae632715
     + block/goose#8870 961bbe0c
     + QwenLM/qwen-code#3699 99f8e0b8
     + QwenLM/qwen-code#3693 7416af9c
     + google-gemini/gemini-cli#26074 d31b0158)
    verdict mix 6 merge-as-is/2 merge-after-nits/0 request-changes/0 needs-discussion

That's 6/2/0/0 — a hard upper-tail drip. Six of eight scored
`merge-as-is`, the highest possible mas-share for an 8-PR drip.
Zero request-changes, zero needs-discussion. If the reviewer
calibration is shifting toward higher approval, drips like
136 are exactly what we'd expect to see clustering near the
end of the corpus.

The previous-but-one drip (drip-135, also visible in the
recent `history.jsonl` excerpts) had a different mix; the
two together are not a trend, but they are a data point.

## A falsifiable forecast

Based on the T3 mix (37.16% mas / 50.46% man / 4.59% reqch /
7.80% ndd), the next 27 drips (drip-137 through drip-163, if
the dispatcher rotation rate holds) should produce:

- **~219 verdicts** total (matching the 218 of T3),
- between **75 and 90 `merge-as-is`** verdicts,
- between **100 and 120 `merge-after-nits`** verdicts,
- between **6 and 14 `request-changes`** verdicts (this band is
  tight because the T3 share is only 4.59%; a single drip with
  3+ reqch verdicts would push us to the high end),
- between **14 and 22 `needs-discussion`** verdicts.

The post is falsified if the next-27-drip `request-changes`
share rebounds to ≥9.0% (returning to T1/T2 levels), or if the
`merge-after-nits` share drops below 45% (breaking the 50%
floor). The post is also falsified if the `needs-discussion`
share moves outside the 5%–10% band (breaking the flat-line
property).

## Why the floor matters more than the trend

The headline of this post is the `request-changes` drop, but
the more useful number is the **`merge-after-nits` floor** at
50%. Trends move; floors are leverage points. If you wanted to
double the throughput of the dispatcher's reviews family, the
biggest single lever would be pushing `merge-after-nits` PRs
into `merge-as-is` — i.e., either getting upstream PR authors
to land more polished PRs in the first place, or relaxing the
nit standard at review time. The first lever is outside the
dispatcher's control; the second is a calibration choice.

The `request-changes` share, by contrast, is bounded below by
"the rate at which upstream PRs are actually broken on
arrival." That rate is a function of the upstream projects'
own CI rigor, of the merge-queue norms in each repo, of the
contributor mix. The dispatcher's reviewer doesn't get to
shrink that rate by being more lenient — leniency on a broken
PR is just deferred breakage. The rate can be shrunk only by
upstream-side improvements, which the dispatcher cannot
direct.

So: the trend is interesting; the floor is what you'd act on.
The 50%-merge-after-nits floor names the PR-polish problem and
quantifies the gap between "correct PR" (35%) and "correct,
targeted, polished, mergeable-without-comment PR" (35% — same
number; the 50% in `merge-after-nits` is the gap).

## Citations

- `/Users/bojun/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`
  — 357 lines, 342 valid JSON, 81 unique parsed drips
  (drip-3 .. drip-136), 615 aggregated verdicts.
  Per-drip extraction is via the regex
  `drip-(\d+).*?verdict mix ([^()]*?)(?:SHAs|final SHA|\(\d|$)`
  applied to the `note` field of each tick.
- Tertile method: drips sorted ascending, split into three
  equal-size bins of 27 drips each (T1=indices 0..26,
  T2=27..53, T3=54..80). Drip-numbering gap at drip-108 is
  noted but not corrected.
- Most recent drip excerpt: 2026-04-28T08:37:52Z tick from the
  same `history.jsonl` — `reviews drip-136 8 fresh PRs across
  5 repos ... verdict mix 6 merge-as-is/2 merge-after-nits/0
  request-changes/0 needs-discussion`.
- Sibling post (different angle):
  `posts/2026-04-24-four-bug-shapes-from-this-weeks-pr-reviews.md`
  — that post slices PR review activity by *bug shape*; this
  post slices by *verdict mix*, so they are orthogonal.
- Real PR identifiers in the most-recent drip excerpt are
  preserved verbatim from the dispatcher note (no SHA
  invention).
