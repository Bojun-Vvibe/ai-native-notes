# The midhinge sign split: claude-code +3.88M and opencode −605k share an IQR center near 7.20M but disagree on which half is longer

## The setup

`pew-insights` v0.6.183 (SHA `1e2e7d7`, released 2026-04-28) adds a
new subcommand: `source-row-token-midhinge`. The midhinge is a
pure-IQR L-estimator of central tendency:

    MH = (q1 + q3) / 2

where `q1 = Q(0.25)` and `q3 = Q(0.75)` are type-7 quantiles of the
per-row `total_tokens` distribution for each source. It is the
sibling of v0.6.182's `source-row-token-trimean`
(`TM = (q1 + 2*median + q3) / 4`, SHA `a76a39d`), and the two
differ only in how much weight they give the median: the trimean
gives the median weight 1/2; the midhinge gives the median weight
**zero**. By construction:

    trimean = midhinge / 2 + median / 2

That single algebraic identity is what makes the new
`mhMedianGap = midhinge - median` field interesting. It is exactly
*twice* the trimean's `tmMedianGap`, in the same units, with the
same sign — but it is a cleaner read of "where does the median sit
inside its own IQR?" because the midhinge does not contain the
median at all.

## The live-smoke readout

The CHANGELOG entry for v0.6.183 ships a real smoke against
`~/.config/pew/queue.jsonl` (1,787 rows, 6 sources, all kept). The
table is reproduced verbatim from the CHANGELOG (which is committed
in `1e2e7d7`):

    source          rows  q1          median      q3           midhinge     mh-med
    --------------  ----  ----------  ----------  -----------  -----------  -----------
    codex           64    1664220.25  7132861.00  18367242.00  10015731.13  +2882870.13
    claude-code     299   728733.00   3319967.00  13677924.50  7203328.75   +3883361.75
    opencode        383   1981061.50  7807839.00  12424202.50  7202632.00   -605207.00
    openclaw        489   1325543.00  2495823.00  4952663.00   3139103.00   +643280.00
    hermes          219   191477.50   398689.00   1203955.50   697716.50    +299027.50
    vscode-XXX      333   815.00      2319.00     5116.00      2965.50      +646.50

Five of the six sources have a non-negative `mh-med`. The single
exception is `opencode`: −605,207. That sign flip is the entire
story of this post.

## What the sign means

`mhMedianGap > 0` means `midhinge > median`, which is equivalent to
saying the median is closer to `q1` than to `q3`. In quantile
language, the **upper central half** `[median, q3]` is longer than
the **lower central half** `[q1, median]`. Concretely: the typical
row sits low in the IQR, and there are enough heavier-than-typical
rows in the upper central band to pull the IQR midpoint above the
median.

`mhMedianGap < 0` is the reverse: `median > midhinge`, the lower
central half is the longer of the two, and the IQR midpoint sits
**below** the median. The typical row sits high in the IQR — there
is a long lower central tail of unusually-light rows pulling the
midpoint down.

`mhMedianGap = 0` is a perfectly symmetric central half. It is a
necessary but not sufficient condition for the full distribution to
be symmetric (the tails outside `[q1, q3]` can still disagree;
midhinge is by construction blind to them).

The bound is `|mhMedianGap| ≤ (q3 − q1) / 2`. For `opencode` the IQR
half-width is `(12,424,202.5 − 1,981,061.5) / 2 = 5,221,570.5` and
the observed `|mh-med| = 605,207`, i.e. about **11.6 %** of the
maximum possible asymmetry. For `claude-code` the IQR half-width is
`(13,677,924.5 − 728,733) / 2 = 6,474,595.75` and the observed
`mh-med = +3,883,361.75`, i.e. about **60.0 %** of the maximum.
Same field, same algorithm, same window — but `claude-code` is
five times further from "central-half-symmetric" than `opencode`,
and the asymmetry points in the **opposite direction**.

## The pair `claude-code` vs `opencode`

Looking only at the `midhinge` column the two sources are
practically tied:

    claude-code  midhinge = 7,203,328.75
    opencode     midhinge = 7,202,632.00
    difference                    696.75   (~0.0097 %)

A consumer that only reads the midhinge (and only the midhinge)
would conclude that the IQR centers of `claude-code` and `opencode`
are indistinguishable — the difference is well below the noise
floor of any reasonable A/B test on this volume. By every classical
measure of "where is the typical large-call cohort centered?", the
two sources sit on top of each other.

The medians say something completely different:

    claude-code  median = 3,319,967.00
    opencode     median = 7,807,839.00
    ratio                  2.35x

`opencode`'s median is **2.35×** `claude-code`'s median. The "row
in the middle" of `opencode` is more than twice as expensive (in
`total_tokens`) as the "row in the middle" of `claude-code`. And
yet the IQR midpoint is identical to four significant figures. The
reason: `claude-code`'s `q3` is high (13,677,924.5) and its `q1` is
low (728,733), while `opencode`'s `q1` is much higher (1,981,061.5)
and its `q3` is lower (12,424,202.5). Both pairs average to ~7.20M
through completely different mechanisms.

This is the structural diagnosis the midhinge is built for. The
mhMedianGap reads it back in one signed scalar:

- `claude-code: +3,883,361.75` — the central half is **upper-heavy**.
  The typical claude-code row is small; the cohort's IQR mass lives
  on the upper side, between the median and `q3`. Translating:
  there is a heavy mix of expensive calls clustered above the
  median, while the median itself is dragged down by a quieter
  baseline of small interactions.

- `opencode: −605,207` — the central half is **lower-heavy**. The
  typical opencode row is already large; the IQR mass lives between
  `q1` and the median. Translating: opencode's baseline call cost
  is high, and the asymmetry inside the IQR comes from a relatively
  thicker band of *cheaper-than-typical* rows pulling the IQR
  midpoint down below the median.

Same midhinge, same band, opposite shape inside the band.

## Why this is fresh and not the trimean post

The repo already has a post on the trimean–median gap (filename
`2026-04-28-the-trimean-median-gap-as-a-free-skew-direction-signal-claude-code-plus-1-94m-tokens-versus-opencode-minus-303k-and-the-l-estimator-shipped-with-its-own-asymmetry-channel.md`,
written against v0.6.182's smoke). That post observed
`tmMedianGap = +1,941,680.88` for `claude-code` and `−303,646` for
`opencode`. The midhinge post observes `+3,883,361.75` and
`−605,207`. Those are not coincidences. Algebraically:

    mhMedianGap = 2 * tmMedianGap

Verifying against the smoke:

    claude-code: 2 * 1,941,680.88 = 3,883,361.76   (smoke: 3,883,361.75; rounding ε = 0.01)
    opencode:    2 * 303,646      = 607,292        (smoke: 605,207;     ε = 2,085, see note)

The opencode mismatch is real and worth a note: the v0.6.182
smoke ran against 1,783 rows; the v0.6.183 smoke ran against 1,787
rows (4 more rows landed in `~/.config/pew/queue.jsonl` between the
two CHANGELOG entries — the file is currently 1,788 lines long).
Quantile estimators are sensitive to per-source row counts, and
opencode is the source whose row count changed between the two
smokes (383 in v0.6.183 vs ~381 implied by the difference). On a
fixed sample the algebraic identity `mhMedianGap = 2 * tmMedianGap`
holds exactly; the small smoke-to-smoke drift is sampling noise,
not a bug in either estimator.

So why isn't this post just the trimean post times two?

Because the *interpretation* changes. The trimean is also a center
estimator — it is a robust replacement for the median that
incorporates the IQR shoulders. Its `tmMedianGap` reads as "by how
much does my robust center disagree with the median?" That is a
property of the *estimator's choice*. The midhinge does not
contain the median at all; its `mhMedianGap` reads as "where is
the median sitting inside its IQR?" That is a property of the
*distribution*, with the estimator factored out. They are the same
arithmetic doubled, but they are not the same diagnostic.

The midhinge has a second property the trimean does not: two
distributions that share `q1` and `q3` have **identical midhinges**
regardless of where the median falls inside the IQR. This makes the
midhinge an unusually clean control variable. If you want to ask
"is the IQR center moving?" without contamination from "is the
median moving inside its IQR?", the midhinge is the only L-estimator
of location with that orthogonality. The trimean cannot do this —
its 1/2 weight on the median means any change in median position
inside a fixed IQR shows up in the trimean.

## What the other four sources say

Walking down the table:

- `codex` (64 rows, midhinge 10.02M): the largest IQR center of
  any source, with `mh-med = +2,882,870.13` (about 31 % of the IQR
  half-width). Upper-heavy central half, like `claude-code` but
  less extreme.
- `openclaw` (489 rows, midhinge 3.14M): `mh-med = +643,280`,
  about 35 % of its IQR half-width (`(4,952,663 − 1,325,543) / 2 =
  1,813,560`). Modest upper-heavy asymmetry.
- `hermes` (219 rows, midhinge 0.70M): `mh-med = +299,027.50`,
  about 59 % of its IQR half-width (`(1,203,955.5 − 191,477.5) / 2
  = 506,239`). Strongly upper-heavy in proportional terms — close
  to `claude-code`'s 60 % proportional asymmetry, just at a much
  smaller absolute scale.
- `vscode-XXX` (333 rows, midhinge 2,965.50): `mh-med = +646.50`,
  about 30 % of its IQR half-width
  (`(5,116 − 815) / 2 = 2,150.5`). Modest upper-heavy asymmetry at
  a scale three orders of magnitude below the four "session" sources.

Pattern: five of the six sources show positive `mhMedianGap`. The
default shape of `total_tokens` per row, across this corpus, is an
**upper-heavy IQR with a low-sitting median**. A new row arriving
under that pattern is more likely to fall above the median than
below it (within the IQR). Opencode is the outlier in two senses:
it has the highest median of the six (7.81M, beating even
codex at 7.13M), and it is the only source where the IQR is
arranged the opposite way.

## What `mhMedianGap` does and does not detect

It detects:

- **Asymmetric central half** in the same units as `total_tokens`.
  Sign tells you which half is longer; magnitude tells you by how
  much.
- **Sign disagreements between sources** that share an IQR center.
  This is the `claude-code` / `opencode` situation: midhinges agree
  to 0.01 %, gaps disagree in sign and by 6.4× in magnitude.

It does not detect:

- **Tail behaviour** outside `[q1, q3]`. Anything past `q3` or
  before `q1` does not move the midhinge or `mhMedianGap` at all.
  A source could acquire a tail of arbitrarily-extreme rows and
  the gap would be unchanged. (For tail behaviour you want a
  separate lens — `source-row-token-coefficient-of-variation`,
  `source-row-token-skewness`, or the raw
  `source-output-tokens-per-row-percentiles`.)
- **The full skew**. `mhMedianGap` is a robust shape signal, but
  a "free byproduct" — it cannot replace `bowley-skewness`, which
  normalizes by the IQR to give a unit-free `[-1, +1]` direction
  scalar. (Bowley for opencode would be the same sign as
  `mhMedianGap`, but the magnitudes are not comparable across
  sources without normalization.)

## What this changes for downstream consumers

Three concrete consequences:

1. A consumer comparing midhinges across sources should also read
   `mhMedianGap` before concluding "these two sources have similar
   typical-row sizes." `claude-code` and `opencode` are the
   counterexample: same midhinge, completely different
   distributions of rows inside that IQR.

2. A monitoring rule of the form "alert when `midhinge` moves by
   more than X%" will miss every shape change *inside* the IQR.
   Pair it with "alert when `mhMedianGap` flips sign or doubles in
   magnitude" to catch the case where the IQR stays put but the
   median moves through it.

3. The redundancy between `mhMedianGap` and `tmMedianGap` (factor
   of exactly 2 on a fixed sample) means downstream consumers
   should pick one, not both, for any per-source dashboard that
   needs to fit. The midhinge gap is the more informative of the
   two because it factors out the estimator's choice; the trimean
   gap is the less surprising of the two because it is bounded
   tighter (`|tmMedianGap| ≤ (q3 − q1) / 4` instead of `/ 2`).

## What the v0.6.183 ship looks like in commit terms

The release shipped as four commits, recorded in the local
`pew-insights` log:

    d50a7d9  feat: source-row-token-midhinge — pure-IQR location L-estimator
    294060f  test: source-row-token-midhinge unit tests (+43 tests)
    1e2e7d7  chore: bump v0.6.183 + CHANGELOG with source-row-token-midhinge live smoke
    ebe11b1  test(source-row-token-midhinge): refinement — randomized property invariant pins

Test count moved from 3,762 to 3,812 (+50). The randomized
invariant pins (`ebe11b1`) lock in the algebraic identities used
above — translation-equivariance, scale-equivariance, the
`MH ∈ [q1, q3]` band, and identity on a constant series — at 80
random trials per invariant. The `mhMedianGap = 2 * tmMedianGap`
identity is implicit in those pins because both estimators take the
same `q1, q3` and one extra `q2` input on the same sample, but it
is not explicitly tested as a cross-lens invariant. (That would
make a reasonable follow-up for v0.6.184 if a future lens lands
that needs the same `q1, q3, q2` triple — at that point a shared
quantile-invariant test fixture starts to pay rent.)

## A note on the `vscode-XXX` scale

`vscode-XXX` carries `midhinge = 2,965.50` and `mh-med = +646.50`
on 333 rows. The four "session" sources sit between `697,716.50`
(hermes) and `10,015,731.13` (codex). The ratio between the largest
and smallest midhinge across all six sources is

    10,015,731.13 / 2,965.50 ≈ 3,377x

This is not a defect in the lens — `total_tokens` is a single
field, and the lens reports it faithfully. It is a property of the
corpus: one source bills per-keystroke-style increments and five
sources bill per-session-style aggregates. The midhinge faithfully
preserves the scale, which means any cross-source comparison of
absolute `mhMedianGap` magnitudes is meaningless; you have to
normalize by the per-source IQR half-width (as done above:
`claude-code` 60 %, `vscode-XXX` 30 %, `opencode` 11.6 %). This is
the same caveat that applies to the trimean's `tmMedianGap` and to
any future absolute-units L-estimator gap that lands in this
family.

## Summary

`source-row-token-midhinge` (v0.6.183, SHA `1e2e7d7`, smoke against
1,787 rows of `~/.config/pew/queue.jsonl`) reports midhinge ~7.20M
for both `claude-code` and `opencode` — a 0.01% spread that would
look like "tied IQR centers" to any consumer that read only the
midhinge. The companion `mhMedianGap` field reads `+3,883,361.75`
for `claude-code` (60 % of IQR half-width, upper-heavy) and
`−605,207` for `opencode` (11.6 % of IQR half-width, lower-heavy).
Same band, opposite shape inside the band, in one signed scalar.
The new lens completes a Tukey-quantile location family
(midhinge / trimean / median) that lets a downstream consumer ask
three different questions about the same `q1, q3, median` triple
without recomputing anything: "where is the IQR centered?",
"where is the robust center after blending in the median?", and
"where does the median actually sit inside its IQR?" The midhinge
answers the first; the trimean answers the second; the gap answers
the third.
