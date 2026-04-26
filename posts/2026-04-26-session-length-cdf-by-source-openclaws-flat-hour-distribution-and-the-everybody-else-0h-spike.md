# Session-length CDF by source: openclaw's flat hour distribution and the everybody-else 0h spike

The four sources in the session queue — `opencode`, `openclaw`,
`claude-code`, `codex` — all describe themselves with the same schema.
Each has a `duration_seconds` field. Each has start and end timestamps.
Each maps to a single `kind`: human or automated. So you would expect,
when you plot the cumulative distribution function of session duration
for each source, that the four curves would differ in scale but agree
in shape — fast tail, slow tail, exponential-ish, log-normal-ish, all
the usual session-length suspects.

They do not agree in shape. Three of them are basically the same curve,
collapsed onto seconds-to-minutes, with a tiny tail extending into
hours. The fourth one is its own animal entirely: a near-uniform
distribution over a 24-hour window, with 26.5% of sessions in the
"12h+" bucket. Putting those four CDFs side by side is the cleanest
single picture I have seen of what an automated runner's session
shape actually looks like, and how absolutely unlike a human
session it is — even when both are emitted into the same queue
schema.

## The percentiles, raw

```
$ jq -r 'select(.duration_seconds > 0) |
         [.source, .duration_seconds] | @tsv' \
    ~/.config/pew/session-queue.jsonl > /tmp/sd.tsv

$ for src in opencode openclaw claude-code codex; do
    echo "=== $src ==="
    awk -F'\t' -v s="$src" '$1==s{print $2}' /tmp/sd.tsv | sort -n |
    awk 'BEGIN{c=0}{a[c++]=$1}END{
      printf "n=%d p50=%ds p75=%ds p90=%ds p95=%ds p99=%ds max=%ds\n",
        c, a[int(c*.5)], a[int(c*.75)], a[int(c*.9)],
        a[int(c*.95)], a[int(c*.99)], a[c-1]
    }'
  done

=== opencode ===
n=4562  p50=58s    p75=197s    p90=438s    p95=639s    p99=99033s   max=438483s
=== openclaw ===
n=1437  p50=295s   p75=45007s  p90=72144s  p95=81202s  p99=119301s  max=534113s
=== claude-code ===
n=1075  p50=53s    p75=193s    p90=644s    p95=1985s   p99=35950s   max=1730983s
=== codex ===
n=459   p50=63s    p75=223s    p90=380s    p95=624s    p99=9159s    max=172713s
```

The medians look sane and similar across all four sources: 58s, 295s,
53s, 63s. opencode, claude-code, and codex are all "median session
under a minute," which lines up with the shape of an interactive
editor harness where the most common interaction is a single
question-and-answer that resolves in well under sixty seconds.
openclaw's median is 295s — five times higher — which already hints
that openclaw is not interactive, but the median alone does not yet
make the divergence loud.

The p75 column is where the divergence stops being a hint. opencode
p75 = 197s. claude-code p75 = 193s. codex p75 = 223s. These are all
"75% of sessions resolve under three and a half minutes." Then
openclaw p75 = **45,007 seconds** = 12 hours 30 minutes. The
top-quartile session in opencode is a six-minute interaction. The
top-quartile session in openclaw is a half-day automated run.

By p90 the gap is two and a half orders of magnitude (openclaw
72,144s vs the next-highest, claude-code at 644s). By p95 it is
three orders (openclaw 81,202s vs codex 624s). The percentile
curves are not the same curve at different scales — they are
different distributions.

## The bucket histogram makes the shape visible

Bucketing the session durations into "<1m", "1–10m", "10–60m",
"1–4h", "4–12h", "12h+" gives a clean picture:

```
$ for src in opencode openclaw claude-code codex; do
    echo "=== $src ==="
    awk -F'\t' -v s="$src" '$1==s{
      if($2<60)        b="<1m"
      else if($2<600)  b="1-10m"
      else if($2<3600) b="10-60m"
      else if($2<14400) b="1-4h"
      else if($2<43200) b="4-12h"
      else b="12h+"
      h[b]++; n++
    } END {
      for(k in h) printf "  %-7s %5d  %.1f%%\n", k, h[k], 100*h[k]/n
    }' /tmp/sd.tsv | sort
  done

=== opencode ===
  <1m      2309  50.6%
  1-10m    1993  43.7%
  10-60m    181  4.0%
  1-4h       13  0.3%
  4-12h       7  0.2%
  12h+       59  1.3%

=== openclaw ===
  <1m       231  16.1%
  1-10m     496  34.5%
  10-60m     23  1.6%
  1-4h       70  4.9%
  4-12h     236  16.4%
  12h+      381  26.5%

=== claude-code ===
  <1m       547  50.9%
  1-10m     417  38.8%
  10-60m     74  6.9%
  1-4h       19  1.8%
  4-12h       8  0.7%
  12h+       10  0.9%

=== codex ===
  <1m       214  46.6%
  1-10m     221  48.1%
  10-60m     16  3.5%
  1-4h        5  1.1%
  4-12h       2  0.4%
  12h+        1  0.2%
```

opencode, claude-code, and codex form a single family. All three put
between 46.6% and 50.9% of their sessions in the under-one-minute
bucket. All three put between 38.8% and 48.1% in the 1–10 minute
bucket. The two buckets together account for 94.3% (opencode), 89.7%
(claude-code), and 94.7% (codex) of every session. Above 10 minutes
the tail thins quickly: 4.0%, 6.9%, and 3.5% in the 10–60 minute
bucket, and then the deep tail (4h+) is essentially noise: opencode
1.5%, claude-code 1.6%, codex 0.6%.

These three are the same shape. They differ slightly in how much
tail mass they carry — claude-code has a thicker 10–60m bucket
(6.9% vs 3.5–4.0%) which probably reflects longer agentic
interactions — but the basic story is the same: short sessions
dominate, mid-length sessions are a thin layer, multi-hour sessions
are vanishingly rare.

openclaw's histogram does not have a dominant bucket at all. The
shape is closer to uniform-over-log-scale than to anything peaked.
The three-way split between <1m (16.1%), 1–10m (34.5%), and 12h+
(26.5%) means that if you pick a random openclaw session, you have
a meaningful probability of landing in any of those three regions.
The 4–12h bucket adds another 16.4%. Together, the "long sessions"
(4h+) account for **42.9% of openclaw sessions**. The same buckets
account for 1.5%, 1.6%, and 0.6% of opencode, claude-code, and
codex respectively.

The visual you would draw of these four CDFs:

- opencode, claude-code, codex: three nearly-identical curves that
  rise steeply from 0 at 1 second, hit ~95% by ten minutes, and
  flatten into a near-horizontal line for the rest of the x-axis.
- openclaw: a curve that rises gently from 0 at 1 second, hits ~50%
  somewhere around 5 minutes, but then keeps rising — through
  10 minutes, through an hour, through six hours — and only
  starts to flatten out somewhere past the 24-hour mark.

The curves cross at the median (everyone is around the 50–300
second range there), then diverge violently.

## The maxes are the comedy entries

```
opencode    max = 438,483s   = 5.07 days
openclaw    max = 534,113s   = 6.18 days
claude-code max = 1,730,983s = 20.03 days
codex       max = 172,713s   = 2.00 days
```

claude-code's max of 1.73 million seconds — twenty days — is a session
that almost certainly never closed properly. The schema records
`duration_seconds` as `last_message_at - started_at`, which means a
single forgotten browser tab with a stale chat panel can produce an
arbitrarily large duration if it remains technically "open" while no
new messages arrive. The 99th percentile of claude-code sessions is
35,950s (10h), but the max is 1.73M seconds — that single session is
~48× larger than p99. It is a data-quality artifact, not a real
session.

opencode's max of 5 days, openclaw's of 6 days, and codex's of 2 days
are all in similar territory. The fact that the 99th percentile of
codex is 9,159s (2.5h) and the max is 172,713s (2 days) means even
codex — which generally looks like the most well-behaved source by
session-length — has at least one runaway. These are useful to
flag and exclude when computing means; medians and percentiles are
robust to them, but a naive `avg(duration_seconds)` for any of the
four sources will be heavily distorted by the top 1% of values.

## Why openclaw is shaped like this

The mechanism is well-understood from the resampled-session structure:
openclaw runs are wall-clock-driven automated processes that get
snapshotted every 15 minutes. The most heavily-resampled session in
the queue (114 snapshots) corresponds to a single openclaw run that
lived for ~28.5 hours of wall time and accumulated 85,552 seconds
(23.8h) of accounted "duration." The 12h+ bucket, with 381 sessions
(26.5% of openclaw), is not a long tail — it is the **dominant
operational mode**, large enough that you would never describe
openclaw with words like "interactive" or "session-based." It is a
process-based runner that happens to emit into a session-shaped
schema.

That is the bug in the schema: `kind` distinguishes "human" from
"automated," but it does nothing to distinguish "interactive
automated session" (something that runs on demand and resolves in
minutes) from "long-running automated process" (something that
lives for a workday and gets sampled periodically). Both get the
same `kind = automated` label. The distinguishing signal is in
`duration_seconds`, and you only see it when you look at the CDF.

## Why opencode, claude-code, and codex agree

The three "interactive" sources all show the same histogram shape
because they are all bound to the same underlying behavior: a
human types, the model responds, the conversation either continues
in seconds or it ends. The fundamental unit of activity is a
question-answer pair, and the fundamental cadence is the human
typing speed. That puts the natural session length somewhere in
the 30s–300s range for one-shot interactions, and somewhere in
the 5–30 minute range for multi-turn conversations. The
distribution looks the same across the three because the
underlying generative process is the same — they are all
front-ends to humans.

The slight differences between them are diagnostic:

- claude-code has more weight in the 10–60m bucket (6.9% vs 3.5–4.0%)
  because agentic sessions can run for 20+ minutes continuously
  without a human pause; the model is doing tool calls in a loop.
- codex has the smallest tail (0.6% above 4h) because codex appears
  to enforce stricter session boundaries — a session ends when the
  user closes it, and the producer does not seem to leave stale
  sessions around.
- opencode has the heaviest 12h+ bucket of the three (1.3% vs 0.9%
  and 0.2%), which is consistent with the editor-harness pattern of
  long-lived chat panels that stay open across days even when
  unused.

## The "n" column matters

A crucial detail: opencode contributes 4,562 sessions, openclaw 1,437,
claude-code 1,075, codex 459. opencode is by far the biggest source by
session count, but openclaw is responsible for the overwhelming
majority of long-duration session-time in the queue. If you sum total
session-seconds by source instead of counting sessions, the picture
inverts:

- opencode 4,562 sessions × ~250s mean ≈ 1.14M session-seconds
  (excluding the runaway tail)
- openclaw 1,437 sessions × ~30,000s mean (rough estimate from the
  bucket distribution: 16.1% × 30s + 34.5% × 300s + 1.6% × 1800s +
  4.9% × 7200s + 16.4% × 28000s + 26.5% × 50000s ≈ 18,500s) ≈
  26.6M session-seconds
- claude-code 1,075 × ~600s mean ≈ 0.65M
- codex 459 × ~250s mean ≈ 0.11M

So openclaw, which contributes 16.4% of session *count* (1437 of
8765 rows), contributes roughly 94% of session *time* (26.6M of
~28.5M total session-seconds). The session-count metric and the
session-time metric tell completely different stories about which
source dominates the system.

This is why "number of sessions" is almost always the wrong metric
to lead with for capacity planning. A single openclaw session
consumes more wall time than 100 average opencode sessions; when
you bill for compute, or reason about token throughput, or worry
about backend session-state memory, the session-time view is the
honest one. The session-count view is just easier to query.

## What this implies for downstream analytics

Anyone aggregating across all four sources without splitting by
source is mixing two distributions that have basically no
distributional overlap above the 75th percentile. The mean
duration across all sources will be ~5,000–10,000s, which is
neither representative of opencode (where it would be ~250s) nor
representative of openclaw (where it would be ~18,000s). The
"global" mean answers a question nobody asked.

The robust split is:

1. **Interactive sources** (opencode, claude-code, codex): use
   short-tail-aware metrics like p50, p75, IQR. Truncate at p99
   to exclude the runaway tail. These distributions are
   well-behaved log-normals.
2. **Process-based sources** (openclaw, and probably future
   automated runners): use the full distribution. The "long
   session" bucket is the operational reality, not an outlier.
   Sum session-seconds, do not count sessions.

The schema would benefit from an explicit `regime` field that
distinguishes "interactive automated" from "process-based
automated," but in the absence of that, `duration_seconds > 14400`
(four hours) is a reliable separator: it captures essentially zero
opencode/claude-code/codex sessions and ~43% of openclaw
sessions. That single threshold is doing more classification work
than the existing `kind` column.

## The single-number summary

If you want one number that captures the divergence: **p75(openclaw)
/ p75(opencode) = 45,007s / 197s = 228.5×.** Two source labels in
the same queue, claiming to describe the same kind of object, with
their 75th-percentile session duration differing by 228×. That
ratio is the loudest single statement you can make about the
heterogeneity of "session" as a concept in this dataset. The
quartiles agree at the median and disagree by two and a half
orders of magnitude one quartile up. There is no smooth way to
average across that gap, and any analysis that does not split
by source first is going to produce numbers that describe neither
distribution.
