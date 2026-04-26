# Max-over-Mean as an Outlier Coefficient: the 33.9× ide-assistant-A Shock and the 4.6× codex Floor

When you're sizing a timeout, a token budget, or a kill switch for a
class of requests, the most useful single number is rarely the mean
and rarely the p99. It's the ratio between the heaviest single
observation and the mean — what I'll call here the **outlier
coefficient**, defined as `max / mean` of per-row total tokens.

That ratio tells you, in a way that p95 and p99 do not, how badly a
naive "size for the average" budget will blow up under the worst
real observed case. A source with an outlier coefficient near 1
behaves predictably; a source with an outlier coefficient of 30
behaves predictably 99% of the time and explodes the other 1%.

This post computes the outlier coefficient for each of the six
sources telemetering into `~/.config/pew/queue.jsonl` (1,574 rows
total) and ranks them. Then it argues that the outlier coefficient
is the right metric for sizing budgets, more so than the standard
deviation, the coefficient of variation (CV), or any single
percentile.

## The numbers

Per-row total tokens are computed as `input_tokens + output_tokens
+ cache_read_input_tokens + cache_creation_input_tokens`. The cache
columns are zero across every row in this device's current dataset,
so the totals reduce to `input + output`.

| source          | n rows | mean tok/row | sd tok/row | CV    | max tok/row   | max / mean |
|-----------------|--------|--------------|------------|-------|---------------|------------|
| ide-assistant-A | 333    | 5,154        | 14,924     | 2.896 | 174,625       | **33.9×**  |
| opencode        | 305    | 736,295      | 1,087,714  | 1.477 | 12,042,429    | 16.4×      |
| openclaw        | 410    | 2,303,798    | 2,669,602  | 1.159 | 23,549,978    | 10.2×      |
| claude-code     | 299    | 6,176,396    | 9,062,442  | 1.467 | 56,155,467    | 9.1×       |
| hermes          | 163    | 348,480      | 499,585    | 1.434 | 3,151,341     | 9.0×       |
| codex           | 64     | 6,450,410    | 7,207,265  | 1.117 | 29,783,731    | **4.6×**   |

(`ide-assistant-A` is the redacted label for the editor-embedded
assistant source. The redaction is mechanical and applied uniformly
across this notebook.)

The spread is dramatic: from 4.6× (codex, the floor) to 33.9×
(ide-assistant-A, the ceiling) — a **7.4× spread in outlier
coefficient itself**, which is to say, the very *severity* of
outliers varies by an order of magnitude across sources.

## Why max/mean and not CV

The coefficient of variation (CV = sd/mean) is the textbook measure
of distribution spread. The CV column is in the table for
comparison. Notice the rankings differ:

- By CV, the most volatile source is ide-assistant-A at 2.896, then
  claude-code at 1.467 ≈ opencode at 1.477, then hermes at 1.434,
  then openclaw at 1.159, then codex at 1.117.
- By max/mean (outlier coefficient), the most volatile is
  ide-assistant-A at 33.9×, then opencode at 16.4×, then openclaw at
  10.2× ≈ claude-code at 9.1× ≈ hermes at 9.0×, then codex at 4.6×.

The two metrics disagree most sharply on **claude-code** (CV says
2nd-most-volatile; max/mean says only 4th) and on **openclaw** (CV
says 5th; max/mean says 3rd).

The reason is that CV is sensitive to the entire shape of the
distribution, while max/mean is sensitive only to the heaviest
single point relative to the average. For operational sizing — what
budget do I need to *not* cut off any observed request — the
heaviest single point is what actually matters. The rest of the
distribution shape is interesting for capacity planning but doesn't
tell you whether a particular timeout will fire.

## Reading each source's outlier coefficient

**codex (4.6×).** This is the lowest outlier coefficient in the
dataset, and it is consistent with codex's other behavior: low row
count (64 rows), high mean (6.45M tokens per row), narrow
distribution. codex is "predictable big". A single per-request
budget of `1.05 × 4.6 × mean ≈ 31M tokens` covers every observed
codex request with about 5% headroom. There is no need to special-case
the tail.

**hermes (9.0×).** The local proxy, lightest mean of the real-agent
sources at 348K tokens, max of 3.15M. The 9× ratio is moderate. A
budget at 10× mean — 3.5M tokens per request — covers all observed
hermes traffic. Worth noting: the heaviest hermes request is itself
a load-bearing call, not a stray, since hermes by design carries the
larger producers' tokens through to upstream. Think of hermes's
heaviest row as "the worst-case relay", not "the worst-case origin".

**claude-code (9.1×).** Despite having by far the heaviest absolute
heaviest-row in the dataset (56.16M tokens), claude-code's outlier
coefficient is moderate at 9.1×. That number is operationally
hopeful: it means that if you size budgets at ~10× the claude-code
mean, you cover everything observed so far. The bad news is the mean
is so high (6.18M tokens) that 10× of it is 62M tokens — and any
upstream rate-limit window will need to be sized for that.

**openclaw (10.2×).** Almost the same shape as claude-code, scaled
down by ~2.4× in absolute. The 10.2× outlier coefficient implies a
similar sizing rule (~10× mean for budget headroom). openclaw and
claude-code are operationally similar shapes: heavy-mean producers
with one-decade outlier coefficients.

**opencode (16.4×).** Now we're in different territory. Mean of 736K
tokens, heaviest of 12.04M tokens. To cover every observed opencode
request with budget headroom, you'd need to set the cap at
~17× mean, or about 12.5M tokens per request. Either you accept
that the cap will be far above the typical request (and so the
budget is mostly idle), or you classify opencode requests by
expected size before scheduling them.

**ide-assistant-A (33.9×).** This is the operational headache. Mean
of 5,154 tokens, max of 174,625. The cap-vs-typical gap is so
extreme that a static per-request budget cannot simultaneously be
"tight enough to catch a runaway loop" and "loose enough to permit
the heaviest 1% of legitimate completions". The numbers force you
into one of two designs:

1. A two-tier budget: typical-budget for the bottom 99%, escalation
   path for the top 1%. The escalation path needs explicit signal
   from the source, since you can't predict which request will be
   the giant.
2. A streaming budget: cancel based on observed token-rate over a
   sliding window, not on a per-request total. This is harder to
   implement but degrades more gracefully.

The ide-assistant-A figure is also the strongest argument that
"sizing for p95" is wrong for editor-embedded assistants. The p95
of ide-assistant-A is far below 174,625; if you size for p95, you
cut off the top 5% of requests, which are exactly the hardest and
most user-visible ones. Editor-embedded sources have heavy tails by
design, and the design assumption needs to be heavy-tail-aware.

## Sanity-checking against the standard deviation

The CV (sd/mean) column gives a different but complementary view.
ide-assistant-A's CV of 2.896 is the largest by far, and is
consistent with its 33.9× outlier coefficient: a CV well above 1
already implies that the distribution is dominated by rare large
events, and a max/mean of >30 is the predictable extreme of that
shape.

codex's CV of 1.117 — the smallest CV in the dataset — is also
consistent with its 4.6× outlier coefficient. codex requests cluster
tightly around a high mean, and the heaviest one is not far above
the bulk.

The interesting case is claude-code: CV 1.467 (high, second only to
ide-assistant-A and very close to opencode) but max/mean only 9.1×.
That combination tells you the *bulk* of claude-code's distribution
is wide (lots of variation in the 1M-to-20M-token range), but the
heaviest single request isn't an extreme departure from that wide
bulk. claude-code is "broadly distributed but not exotic-tailed".
That's a much friendlier shape than ide-assistant-A's "narrow bulk
plus rare giants".

## A practical sizing rule

A reasonable, conservative per-request token budget for each source,
based purely on `1.10 × max` (i.e. covering the heaviest observed
row with 10% headroom):

| source          | suggested per-request budget (tokens) |
|-----------------|---------------------------------------|
| codex           | ~32.8M                                |
| claude-code     | ~61.8M                                |
| openclaw        | ~25.9M                                |
| opencode        | ~13.2M                                |
| hermes          | ~3.5M                                 |
| ide-assistant-A | ~192K                                 |

A few obvious cautions:

- These are budgets for the *current* observed maxes. The dataset is
  1,574 rows. As more data flows in, the max will creep up. Re-fit
  weekly or set the budget to `2 × current_max` and tolerate the
  waste.
- Per-request budgets should be enforced *upstream* of any retry
  loop. If a request is going to consume 56M tokens, you do not want
  to retry it three times.
- A per-request budget is only one of three layers. The other two
  are per-source per-window budgets (caps total tokens-per-hour) and
  a global kill switch (caps tokens-per-tick across all sources).
  The outlier coefficient informs the per-request layer most
  directly.

## What this metric does not tell you

The outlier coefficient is silent on:

- **Frequency.** A source can have a 30× outlier coefficient and
  emit one giant request per quarter (operationally trivial) or one
  per hour (operationally expensive). max/mean treats both
  identically. Pair it with the row count and the mean rate to
  judge frequency.
- **Direction of growth.** A source whose max is creeping up week
  over week is more dangerous than a source with a stable max, even
  at the same outlier coefficient. The metric is a snapshot, not a
  trend.
- **Cause.** A 33.9× outlier might be a legitimate hard task or a
  runaway loop. The number can't tell you which. It only says: "if
  this happens again, here is how big a hole it will leave."

## Reproducing the numbers

```sh
jq -r '[.source, ((.input_tokens//0)+(.output_tokens//0)
        +(.cache_read_input_tokens//0)+(.cache_creation_input_tokens//0))] | @tsv' \
   ~/.config/pew/queue.jsonl |
  awk -F'\t' '{src=$1; t=$2; n[src]++; sum[src]+=t; sumsq[src]+=t*t;
               if(t>max[src])max[src]=t}
              END{for(s in n){m=sum[s]/n[s];
                   v=sumsq[s]/n[s]-m*m; sd=sqrt(v);
                   printf "%-20s n=%d mean=%.0f sd=%.0f cv=%.3f max=%d max/mean=%.1fx\n",
                          s, n[s], m, sd, m>0?sd/m:0, max[s], max[s]/m}}' |
  sort -k7 -rn
```

The 1,574-row figure is `wc -l ~/.config/pew/queue.jsonl`.
ide-assistant-A is the redacted label for the editor-embedded
assistant source.

## Bottom line

The 33.9× outlier coefficient on ide-assistant-A and the 4.6× floor
on codex bracket the operational reality of this telemetry: at one
end you have a source whose typical request is two orders of
magnitude smaller than its worst, and at the other you have a
source whose worst request is barely 5× its typical. **No single
budget rule covers both shapes.** The dispatcher needs per-source
sizing — not because of fairness, but because the underlying
distributions are that different.

The outlier coefficient is a one-number summary of "how badly will
sizing-for-the-average fail you". For sources with values near 1,
sizing-for-the-average is fine. For sources at 10×, you need 10×
headroom. For sources at 33×, you need a different design, not just
a bigger number.
