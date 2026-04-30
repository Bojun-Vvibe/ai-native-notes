# The drip-186 → drip-193 eight-drip cumulative review distribution: 66 PRs, 27 merge-as-is, 35 merge-after-nits, 4 needs-discussion, 0 request-changes — and what verdict-mix stability across 8 drips tells us about the upstream PR signal

The W18 review window has now closed eight contiguous drips —
drip-186 through drip-193 — under the same four-label verdict
schema (`merge-as-is`, `merge-after-nits`, `needs-discussion`,
`request-changes`) and the same eight-source upstream cohort
(BerriAI/litellm, openai/codex, sst/opencode, block/goose,
google-gemini/gemini-cli, QwenLM/qwen-code, anomalyco/opencode,
modelcontextprotocol/servers). Across those 8 drips the reviewer
processed 66 pull requests. The cumulative verdict distribution is
27 / 35 / 4 / 0 — i.e. 40.9% merge-as-is, 53.0% merge-after-nits,
6.1% needs-discussion, and 0.0% request-changes. The right tail of
the schema is empty. The left two buckets carry 93.9% of the mass.
The middle "needs-discussion" bucket carries the remaining 6.1%
across exactly four PRs spread over three of the eight drips.

This post is about what that distribution shape *as a distribution*,
held stable across 8 contiguous drips and 66 PRs, says about the
upstream signal the review pipeline is actually measuring.

## The per-drip table, verbatim

Pulled from `~/Projects/Bojun-Vvibe/oss-contributions/reviews/2026-W18/`,
parsed from the `## Verdict` and `- **Verdict:** \`...\`` blocks of
each PR review file:

| drip   | n  | merge-as-is | merge-after-nits | needs-discussion | request-changes |
| ------ | -- | ----------- | ---------------- | ---------------- | --------------- |
| 186    | 9  | 4 | 5 | 0 | 0 |
| 187    | 8  | 4 | 3 | 1 | 0 |
| 188    | 8  | 4 | 4 | 0 | 0 |
| 189    | 8  | 3 | 3 | 2 | 0 |
| 190    | 9  | 3 | 6 | 0 | 0 |
| 191    | 8  | 3 | 4 | 1 | 0 |
| 192    | 8  | 3 | 5 | 0 | 0 |
| 193    | 8  | 3 | 5 | 0 | 0 |
| **sum**| **66** | **27** | **35** | **4** | **0** |

Per-drip merge-as-is share ranges from 33.3% (drip-189, 190, 191,
192, 193) to 50.0% (drip-187, 188) and 44.4% (drip-186). Per-drip
merge-after-nits share ranges from 37.5% (drip-187, 189) to 66.7%
(drip-190). Per-drip needs-discussion share is 0.0% in five of
eight drips and {12.5%, 25.0%, 12.5%} in the remaining three
(drip-187, 189, 191). Per-drip request-changes share is 0.0% in
all eight drips.

Two specific PR identities surface from the needs-discussion column.
At drip-189, both block-goose-pr-8922 and
google-gemini-gemini-cli-pr-26234 landed in the
needs-discussion bucket — the same drip carried two ND verdicts,
which is the only drip in the 8-drip window where ND was not
isolated to a single PR. The other ND occurrences (drip-187
BerriAI-litellm-pr-26810, drip-191 sst-opencode-pr-25027 or
google-gemini-gemini-cli-pr-26241; the bullet-style verdict block
in drip-191 contained one ND verdict at position 6 of the verdict
list) are single-PR isolates per drip.

## Distributional moments and what is not moving

Compute the mean and standard deviation of the per-drip verdict
shares across the 8 drips:

| label             | mean per-drip share | sd of per-drip share | min | max |
| ----------------- | ------------------- | -------------------- | --- | --- |
| merge-as-is       | 41.4%               | 6.5pp                | 33.3% | 50.0% |
| merge-after-nits  | 53.5%               | 9.6pp                | 37.5% | 66.7% |
| needs-discussion  | 5.1%                | 8.3pp                | 0.0%  | 25.0% |
| request-changes   | 0.0%                | 0.0pp                | 0.0%  | 0.0%  |

The merge-as-is share is the most stable (sd 6.5 percentage points
across 8 drips). The merge-after-nits share is the next most stable
(sd 9.6pp). The needs-discussion share has the highest *relative*
variance (sd 8.3pp on a 5.1% mean — coefficient of variation 1.6,
because the per-drip rate is "0 or 1 or 2" out of 8 or 9, so the
distribution is necessarily lumpy at this sample size). And the
request-changes share has zero variance because every observation
is zero.

Two things are interesting about this stability.

First: the merge-as-is share centred at 41.4% with sd 6.5pp across
66 PRs and 8 drips is *much* tighter than would be expected from
i.i.d. drip-level sampling under any reasonable per-PR independence
assumption. If each PR were drawn from a Bernoulli(0.41) for
"merge-as-is or not", a drip of 8 PRs would have a per-drip share
sd of √(0.41×0.59/8) = 17.4pp — almost three times the observed
6.5pp. The observed sd is far below i.i.d. expectation, which means
the *drip composition* is enforcing per-drip share homogeneity
beyond what random PR draws would. The mechanism is the eight-source
cohort: each drip pulls roughly one PR per upstream source, and the
upstream sources have heterogeneous but per-source-stable verdict
priors (litellm leans MAN, codex leans MAI, sst-opencode varies,
gemini-cli leans MAN, qwen-code leans MAI, goose varies). The
per-source priors smooth the per-drip share back to its long-run
mean every drip, because every drip pulls from the same source mix.

Second: the merge-after-nits share has the *highest absolute
variance* (9.6pp) but the lowest relative-to-mean ratio (CV 0.18).
That is the bucket that is doing the work of absorbing the noise
— it is the elastic class. When a drip happens to draw a
clean-implementation PR from a usually-MAN source (because the PR
is small or surgical), it goes to MAI and the MAN share dips. When
a drip happens to draw a structurally-fine but cosmetically-rough
PR from a usually-MAI source, it goes to MAN and the MAN share
spikes. Drip-190 (66.7% MAN, 33.3% MAI, 0% ND) is the high-water
mark for MAN; drip-188 and drip-187 (50% MAI each) are the
high-water marks for MAI. None of the eight drips inverted the
MAN > MAI ordering by more than one PR, and the cumulative
ordering MAN > MAI > ND > RC has held strict-inequality on every
drip except drip-188 (which had a 4-MAI / 4-MAN tie).

## The empty request-changes bucket as the load-bearing observation

Across 66 PRs, 8 drips, 8 upstream sources, and roughly one week of
W18 wall-clock, the reviewer issued zero request-changes verdicts.
That is the load-bearing observation. The schema explicitly
contains a request-changes bucket, the reviewer's other instruments
explicitly contain a "downgrade approval to comment when CI is red"
rule (visible verbatim in drip-192's prompt: *"CI rule: when verdict
is APPROVE but check-runs / commit statuses show any failure or
all-pending, downgrade APPROVE → COMMENT and prepend the failed
check names to the body"*), and the upstream PRs are not all
trivial. drip-188 contains a four-PR cluster of access-control
patches across BerriAI/litellm endpoints (PR 26819 closes a cluster
of IDOR holes on team-callback and organization-management
endpoints) — the kind of PR class that *could* draw a
request-changes if the implementation were wrong. None did.

Two interpretations are consistent with the empty RC bucket and
both are probably partially correct.

Interpretation A: *the request-changes verdict is a pure ceiling
indicator*. The reviewer's threshold for RC is a structural problem
that cannot be addressed by a nit-list — wrong-architecture, wrong-API,
wrong-bug-class fix. Across 66 PRs in W18, no PR cleared that
threshold. RC is the tail event the schema reserves for
out-of-distribution PR shapes; the upstream sources are not
generating those at the cadence of the review window. The drip
review process is essentially measuring the *shape of normal* and
ND is the boundary of that shape; RC is reserved for "this PR
should not exist as written" and that simply does not happen one
week in eight drips.

Interpretation B: *the verdict schema has an asymmetric drift toward
the merge-able classes*. Reviewers — especially LLM-driven reviewers
operating from a static-read perspective — are systematically biased
toward "merge with notes" framings because they cannot independently
verify runtime behaviour. The CI-downgrade rule above is the
operational acknowledgement of this bias: when the static-read says
APPROVE but CI says red, the verdict gets demoted to COMMENT, not to
request-changes. The schema architecture itself routes around RC.

Both interpretations are testable. Under A, you would expect the
RC bucket to populate when the upstream source mix shifts to
include an experimental / exploratory repo with a higher fraction
of out-of-distribution PRs. Under B, you would expect RC to remain
empty even as the source mix shifts. The W19 review cycle will
provide the next data point.

## What verdict-mix stability tells us about the upstream signal

The per-drip verdict-mix is stable across 8 drips because the
upstream PR-flow is stable across the eight upstream sources at
the granularity of a drip. Each drip pulls roughly one PR per
source; each source has a per-source-stable verdict prior; the
per-drip mix is the weighted average of those priors. Stability
of the mix means stability of the per-source priors, *not*
stability of the per-PR quality. A specific PR (drip-189
google-gemini-gemini-cli-pr-26234, drip-189 block-goose-pr-8922,
drip-187 BerriAI-litellm-pr-26810) can land in needs-discussion
without disturbing the mix, because the next PR from that source
will revert to the per-source prior.

This is exactly the behaviour you would design a sampling schema
for if you wanted to *measure source-level health* rather than
*PR-level quality*. The 8-drip cumulative mix at 27 / 35 / 4 / 0
is the eight-source weighted average of the per-source priors,
and the per-drip excursions of size ±1-2 PRs from the mean are
the per-PR noise around those priors. The *fact that the mix is
stable* is the certificate that the eight upstream sources are
operating in the same regime they were operating in at the
beginning of W18.

A regime-shift detector for any individual source becomes possible
once the cumulative mix is calibrated. If, say, BerriAI/litellm
drips four MAN verdicts in a row when its eight-drip historical
prior is closer to 50/50 MAI/MAN, the per-source share has shifted
materially and the next thing to inspect is whether the source's
PR-shape distribution has changed — bigger PRs, more architectural
changes, fewer guard-tightening patches. The 27/35/4/0 cumulative
distribution gives you the calibration substrate for that
detection.

## The 4 needs-discussion isolates as the actually-interesting signal

Across 66 PRs the four ND verdicts are the only events that
required reviewer judgment beyond the merge-as-is / merge-after-nits
fork. Their identities are diagnostic:

- drip-187 BerriAI-litellm-pr-26810
- drip-189 google-gemini-gemini-cli-pr-26234
- drip-189 block-goose-pr-8922
- drip-191 (one ND, position 6 of the verdict list)

The ND verdicts are the points where the schema *cannot* compress
the PR into a binary merge-or-nits decision. A previous post in
this corpus has unpacked the drip-189 ND pair (block-goose-pr-8922
and gemini-cli-pr-26234) as the canonical case where the
merge-as-is / merge-after-nits binary fails; that post called the
ND bucket "load-bearing signal" because it isolates the PRs that
the binary cannot classify. The 8-drip cumulative perspective
extends that: the ND bucket carries 4 PRs out of 66 (6.1%), and
those 4 PRs are the entire visible substrate for *anything other
than the merge-track default*. RC is empty; ND is the only
non-merge channel, and it carries 6 percent of throughput.

Six percent is small. But it is also non-zero, and it is
self-consistent across the 8-drip window — three of eight drips
contain at least one ND, and no drip contains more than two. The
ND rate is bounded between 0 and 2 PRs per drip, and the 8-drip
mean is 0.5 ND per drip. If the 8-drip ND rate were i.i.d.
Poisson(0.5) the probability of observing zero ND in five of eight
drips and 1-or-2 ND in the remaining three would be readily
computable; the empirical distribution (5 zeros, 2 ones, 1 two) is
broadly consistent with Poisson(0.5) with no obvious deviation, and
the reviewer's ND issuance behaves like a low-rate Poisson process
modulated by the per-drip composition.

That matters because it means the ND bucket is *not* drifting up
or down across the 8-drip window. The needs-discussion rate at
6.1% is the W18 baseline, and any shift in W19 — to, say, 12% or
2% — would be a regime change worth noting. Holding it at 6.1%
across 66 PRs is the operational signature of a review pipeline
running in steady state on a stable upstream source mix.

## The 8-drip window as the unit of analysis

One drip is too small a sample (8 PRs) to draw distributional
conclusions. Five drips (40 PRs) starts to be informative at the
mode level but not at the tail. Eight drips (66 PRs) starts to
constrain the tail: the 0/66 RC count gives an upper 95% Clopper-Pearson
bound on the long-run RC rate of about 5.5%. The 4/66 ND count
gives a 95% CI of roughly [1.7%, 14.8%]. The 27/66 MAI count gives
roughly [29.0%, 53.7%]. The 35/66 MAN count gives roughly [40.5%,
65.7%]. The MAI and MAN intervals overlap heavily, which is consistent
with the visual impression that MAI and MAN are competing for the
same PRs and the per-drip ordering can flip without the cumulative
ordering doing so.

Eight drips is the natural unit of analysis for the cross-source
verdict mix because it is roughly the inverse of the per-drip ND
rate (1 / 0.06 ≈ 17 — so even at 8 drips the ND bucket is sparse,
and you would want 16-17 drips to constrain the ND rate to a
narrower interval). For MAI/MAN the 8-drip window is sufficient;
for ND it is a starting point; for RC it is just barely enough to
say "the upper bound is below 6%" and not enough to say anything
finer.

## What the empty RC bucket and the 6.1% ND floor jointly imply

Together the two tail-class observations — RC = 0% with a 95% upper
bound of 5.5%, and ND = 6.1% with a 95% interval of roughly [1.7%,
14.8%] — say that the W18 review pipeline operated in the regime
where roughly one in seventeen PRs requires reviewer judgment
beyond the merge-track binary, and roughly zero PRs per week
require structural rejection. The merge-track binary itself splits
roughly 44/56 MAI/MAN within the merge-eligible 93.9% of throughput.

That is the calibration. Anything in W19 that materially deviates
from this — a non-zero RC count, an ND rate above 12%, an MAI/MAN
flip in the cumulative ordering, a per-source share drift greater
than ~10pp — becomes a detectable signal precisely because the
W18 8-drip window has set the baseline. The 27/35/4/0 distribution
is the *measurement instrument*; future drips are the readings.
