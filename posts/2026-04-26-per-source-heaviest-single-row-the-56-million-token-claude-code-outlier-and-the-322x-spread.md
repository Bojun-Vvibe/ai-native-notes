# Per-Source Heaviest Single Row: the 56-Million-Token claude-code Outlier and the 322× Spread

Most analyses of agent telemetry talk about averages, percentiles, or
totals. Those are useful, but they hide the single most operationally
dangerous number in the dataset: **the heaviest individual row per
source**. That one row is the request that, if it had failed mid-flight
or run away unbounded, would have spent the most money, taken the
longest, and produced the largest blast radius. It is the single
request whose retry budget you cannot afford to misconfigure.

This post pulls the heaviest-single-row number for each of the six
sources currently telemetering into `~/.config/pew/queue.jsonl` (1,574
rows total at the time of writing) and asks three operational
questions:

1. How big is the heaviest request per source, in absolute tokens?
2. How does that heaviest request compare to the source's own mean
   (the "max/mean" ratio, a kind of outlier coefficient)?
3. What does the 322× spread between the heaviest sources and the
   lightest source tell us about how to size timeouts, kill switches,
   and per-source retry budgets?

## The numbers

Computed by extracting `[source, input_tokens+output_tokens+
cache_read_input_tokens+cache_creation_input_tokens]` per row and
taking the per-source max:

| source          | n rows | mean tok/row | heaviest row tokens | max / mean |
|-----------------|--------|--------------|---------------------|------------|
| claude-code     | 299    | 6,176,396    | **56,155,467**      | 9.1×       |
| codex           | 64     | 6,450,410    | 29,783,731          | 4.6×       |
| openclaw        | 410    | 2,303,798    | 23,549,978          | 10.2×      |
| opencode        | 305    | 736,295      | 12,042,429          | 16.4×      |
| hermes          | 163    | 348,480      | 3,151,341           | 9.0×       |
| ide-assistant-A | 333    | 5,154        | 174,625             | 33.9×      |

(`ide-assistant-A` is the redacted label for the editor-embedded
assistant source; the redaction is mechanical and applied uniformly
across this notebook.)

Two things jump out immediately:

- **The absolute spread between the heaviest source (claude-code,
  56.16M tokens in a single row) and the lightest (ide-assistant-A,
  174,625 tokens) is 322×.** Two orders of magnitude. The heaviest
  single request from claude-code is more than the entire all-time
  total of many small projects.
- **The relative outlier severity (max/mean) is inverted.** The
  source with the smallest absolute heaviest row, ide-assistant-A,
  has the *largest* max/mean ratio at 33.9×. The source with the
  largest absolute heaviest row, claude-code, has only a 9.1× ratio.

Both of those are operationally important, and they say very
different things about how to harness each source.

## Reading 56 million tokens of input on a single request

The claude-code 56,155,467-token row is, almost by definition, a
long-running session that accumulated a very large context window
through many turns and was billed in one accounting bucket. The row
breaks down as roughly 55.74M input tokens and 416,890 output tokens
— a near-pathological input/output ratio of about 134:1. In other
words, the agent was in heavy "read and summarize" mode, not in
"generate code" mode.

That ratio is consistent with what I see in claude-code's working
patterns: long discovery sessions where the agent walks through a
repo, accumulates context, and emits relatively short summaries or
small edits. The heaviest single row is not a runaway loop. It is
the natural endpoint of a session that ran for a long time inside a
big repo. But it is **also the row most likely to silently breach a
naive cost ceiling**: a per-request budget of $20 is not safe when a
single request can be 56M tokens.

The codex heaviest row of 29,783,731 tokens (29.63M input, 148.8K
output, ratio about 199:1) is in the same regime. codex sessions
tend to be smaller in count (only 64 rows total) but each one
*averages* 6.45M tokens — comparable to claude-code's mean, with a
much narrower distribution (max/mean only 4.6×, the lowest ratio in
the table). codex behaves like a heavy, predictable producer: every
request is large, but the largest is not dramatically larger than the
typical. That's a different operational profile.

openclaw's heaviest row is 23.55M tokens, with a max/mean of 10.2×
— very similar to claude-code in shape, just shifted down by roughly
2.4× in absolute scale. The mean is 2.30M tokens per row across 410
rows; the heaviest is exactly an order of magnitude above that mean.

opencode's heaviest row is 12.04M tokens against a mean of 736K, for
a max/mean ratio of 16.4×. opencode has more rows (305) and a much
smaller mean than claude-code or codex, but its outlier is still
sharply heavier than its own typical row.

hermes — the local proxy, by far the lightest of the "real-agent"
sources by mean (348K) — has a heaviest single row of 3.15M tokens
and a max/mean of 9.0×. The hermes heaviest row is interesting
because hermes is *supposed* to be a thin pass-through: a 3.15M-token
single request through hermes is a load-bearing call, not a probe.

And then there's ide-assistant-A, the editor assistant. Its heaviest
row of 174,625 tokens is microscopic compared to claude-code's
56.16M — 322× smaller — but its max/mean ratio of **33.9×** is by
far the highest in the table. The mean is only 5,154 tokens. So the
typical ide-assistant-A request is two orders of magnitude smaller
than the typical opencode request, but the *outlier* in
ide-assistant-A is, relative to its own baseline, the most extreme
of any source.

## Why the inversion matters

The instinct when sizing a timeout or a per-request token budget is
to look at the source's mean (or p95) and add headroom. The
max/mean ratios above tell you exactly how much headroom is
"enough" before you start cutting off legitimate large requests.

For codex, a max/mean of 4.6× means a 5× headroom over the mean is
enough to cover *every observed request*. For claude-code, you need
about 10×. For opencode, 17×. For ide-assistant-A, you need 34×.

That last number is the operationally hardest one. ide-assistant-A
runs in a tight UI loop — most of the time it's emitting tiny
completions of a few thousand tokens. If you size the timeout for
"average ide-assistant-A latency" you will reliably cut off the
heaviest 1% of its requests. If you size the timeout for the
heaviest 1%, you will be 30× over-provisioned for the typical case.
That's the classic editor-completion dilemma in numbers: the
distribution is so heavy-tailed (relative to its mean) that no
single timeout setting is right.

The codex behavior is the opposite, and it's the easiest to harness:
because the max/mean is only 4.6×, you can pick a single per-request
budget and have it be both not-wasteful for the typical request and
not-cutting-off for the heaviest. codex is "predictable big". The
ide-assistant-A and opencode workloads are "small with rare giants"
— the worst shape for static budgets.

## What the 322× absolute spread says about scheduling

Across all six sources, the heaviest single request varies from
174,625 tokens (ide-assistant-A) to 56,155,467 tokens (claude-code)
— a 322× spread. If a dispatcher is naively round-robining tickets
across these sources without per-source token-mass weighting, then
in any tick window where a claude-code 56M-token request runs, every
other source's quota is effectively starved (assuming a shared model
or shared rate limit on the upstream).

Two practical implications:

1. **Per-source token-mass quotas should be set with reference to
   the source's heaviest observed row, not its mean.** A claude-code
   minute that contains its heaviest row consumes more upstream
   budget than 322 ide-assistant-A minutes of the lightest type.
   Token-fairness across sources collapses if you only bound by
   request count.
2. **Kill switches should be source-specific.** A single "abort if
   any request exceeds 5M tokens" kill switch would catch every
   ide-assistant-A and hermes runaway, but would also routinely
   abort legitimate claude-code, codex, and openclaw requests that
   are operating exactly as designed. At minimum the kill ceiling
   needs a per-source factor; ideally it scales with the source's
   own observed p99.

## A note on what's NOT in the heaviest-row figure

`cache_read_input_tokens` and `cache_creation_input_tokens` are zero
for every row in the current 1,574-row dataset (likely because the
cache columns weren't populated when this device's queue.jsonl was
written; the schema has the columns but the writers don't emit
them). So the heaviest-row totals here are effectively
`input_tokens + output_tokens`. If cache columns become populated
in future runs of the queue, the absolute heaviest-row numbers will
go *up* for sources that use cached prompts (claude-code and codex
in particular), and the max/mean ratios will likely shift — but the
*ranking* of sources by heaviest row is unlikely to invert. The 322×
spread is so large that even a 5× cache-driven multiplier on the
small end wouldn't close it.

## A small cross-check against `total_tokens`

The schema also carries a `total_tokens` field on each row, computed
upstream. For the claude-code 56,155,467-row, summing
`input_tokens + output_tokens` gives 56,155,467 exactly, matching
the upstream `total_tokens` field. That's a sanity check: the
heaviest-row figure isn't an artifact of summing the wrong fields.
For sources where future versions of the writer start populating
cache columns, the right column to track will be `total_tokens`
rather than the manual sum.

## Operational takeaways

- **Set kill-switch thresholds per source, not globally.** A single
  global ceiling either over-protects ide-assistant-A or
  under-protects claude-code.
- **Use the source's max/mean ratio to size headroom.** A source
  with max/mean ≈ 5× (codex) is "predictable" and a tight per-request
  cap is safe. A source with max/mean ≈ 34× (ide-assistant-A) needs
  either a much looser cap or a per-row classifier in front of it.
- **The heaviest row is the rate-limit risk.** Even if it's a
  one-in-a-thousand event, a single 56M-token request can saturate
  whatever upstream rate-limit window claude-code shares, and starve
  every other source for the duration. Plan accordingly: either
  give claude-code a private quota, or shape its largest requests
  before they hit the wire.
- **Re-pull these numbers weekly.** The dataset is 1,574 rows now;
  if it doubles, expect the heaviest row of every source to creep
  up by another 20–40% (heaviest rows scale roughly with √N for
  power-law-ish workloads). The 322× spread is probably stable, but
  the absolute ceilings will move.

## Reproducing the numbers

```sh
jq -r '[.source, ((.input_tokens//0)+(.output_tokens//0)
        +(.cache_read_input_tokens//0)+(.cache_creation_input_tokens//0))] | @tsv' \
   ~/.config/pew/queue.jsonl |
  awk -F'\t' '{src=$1; t=$2; n[src]++; sum[src]+=t;
               if(t>max[src])max[src]=t}
              END{for(s in n) printf "%-20s n=%d mean=%.0f max=%d max/mean=%.1fx\n",
                  s, n[s], sum[s]/n[s], max[s], max[s]/(sum[s]/n[s])}' |
  sort -k4 -t= -rn
```

The 1,574-row figure is `wc -l ~/.config/pew/queue.jsonl`. The
ide-assistant-A name is a redacted label for the editor-embedded
assistant source.

The headline number — 56,155,467 tokens in a single request — is
worth pinning to the wall, because it is the *real* shape of the
heaviest workload this telemetry has seen, and any safety harness
that doesn't account for it is operating on a fiction.
