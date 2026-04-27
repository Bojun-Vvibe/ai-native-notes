# The zero-duration monopoly project: 634 of 634 sessions with duration_seconds=0, and the 100% observational blind spot

Date: 2026-04-28
Source: `~/.config/pew/session-queue.jsonl`, 9996 rows total, snapshot window 2026-04-20 → 2026-04-27.

## The headline

In the session-queue, one specific `project_ref` — `62f8e1ec095e1857` — accounts for **634 sessions**, and **100.0% of them have `duration_seconds = 0`**. Not 99.5%. Not 98%. 634 out of 634, exactly zero seconds each. That makes this project the single largest contributor of zero-duration sessions in the entire corpus, and it does so with perfect uniformity.

For context, the same query across the next nine project_refs gives a very different picture:

- `8001c27439650c5c` — 3857 sessions, 359 zero-duration (9.3%), avg non-zero duration ≈ 11,397s
- `0d6e4079e36703eb` — 1740 sessions, 0 zero-duration (0.0%), avg non-zero duration ≈ 32,215s
- `62f8e1ec095e1857` — **634 sessions, 634 zero-duration (100.0%)**
- `f5756e707027c9b7` — 513 sessions, 203 zero-duration (39.6%), avg non-zero ≈ 19s
- `cb12e97eab40781e` — 482 sessions, 28 zero-duration (5.8%), avg non-zero ≈ 20s
- `d6de2e6f059eb485` — 216 sessions, 1 zero-duration (0.5%), avg non-zero ≈ 303s
- `c56cabc8c1f4b974` — 91 sessions, 0 zero-duration, avg non-zero ≈ 320s
- `557bda4cc2181730` — 58 sessions, 1 zero-duration (1.7%), avg non-zero ≈ 232s
- `8ab30e9dab447af3` — 50 sessions, 0 zero-duration, avg non-zero ≈ 42,386s
- `45de70d31f768901` — 44 sessions, 0 zero-duration, avg non-zero ≈ 149s

`62f8e1ec095e1857` is the *only* project in the top-10 with a zero-duration rate at 100%. The next-worst is `f5756e707027c9b7` at 39.6%. There is no continuous gradient here; there is a binary phenomenon.

## What `duration_seconds = 0` means

A session row in `session-queue.jsonl` has this shape:

```
{
  "session_key":"claude:0a716108-…",
  "source":"claude-code",
  "kind":"human",
  "started_at":"2026-04-20T12:01:51.037Z",
  "last_message_at":"2026-04-20T12:04:52.541Z",
  "duration_seconds":181,
  "user_messages":19,
  "assistant_messages":37,
  "total_messages":104,
  "project_ref":"45de70d31f768901",
  "model":"claude-opus-4.7",
  "snapshot_at":"2026-04-22T14:50:22.808Z"
}
```

`duration_seconds` is computed as `last_message_at − started_at`. A value of 0 means one of three things:

1. The session contains exactly one message (so `last_message_at == started_at`).
2. The session was snapshotted before any second message landed.
3. The session is a metadata stub created by the indexing pipeline that never had any messages at all.

For 634 consecutive sessions belonging to the same `project_ref` to all have duration 0, the explanation cannot be coincidence. It has to be structural.

## Why this is structural, not statistical

If `62f8e1ec095e1857` were a normal project that happened to have many one-message sessions, you would expect the zero-duration rate to be high but not exactly 100%. Even projects dominated by quick interactions show some non-zero durations: `f5756e707027c9b7` is the cleanest comparison case — 513 sessions, 39.6% zero-duration, but the remaining 60.4% of its sessions clock in at an average of 19.2 seconds. Real human interaction with a tool generates *some* multi-message sessions even if most are short.

`62f8e1ec095e1857` does not. Its sessions are not "short"; they are *durationless*. Every single one. That distinction matters: a 19-second average non-zero duration says "the user typed and got a reply, then closed it." A duration of exactly 0 across 634 rows says "no second message was ever observed in this project."

The mechanism is almost certainly that this `project_ref` corresponds to a workflow whose sessions either (a) emit a single message and terminate, (b) are snapshotted before any message is written, or (c) are machine-generated stubs. In all three cases the observability of *what happens inside the session* is zero. We see the session existed. We see what model it claimed. We see what source it was from. We do not see what was said, how long it took, or whether anything responded.

## The data we *do* have on this project

Even with duration=0 across the board, the row carries other fields. From inspection of the 634 rows attributed to `project_ref = 62f8e1ec095e1857`:

- All rows have non-null `started_at` and `last_message_at` — and for every row those two timestamps are equal to within the row's serialization precision (which is what produces the 0).
- The `total_messages` field is consistently small (commonly 1–2), confirming the "single shot" hypothesis.
- The `source` distribution is concentrated on a small number of providers, consistent with this `project_ref` being routed by a particular tool rather than spread across them.

What we do not have:

- Any signal about how long the user-side or model-side processing took. The duration is the only timing field, and it is degenerate.
- Any way to compute the typical inter-arrival time *within* the session (because there is no second message timestamp).
- Any way to compute a meaningful tokens-per-second rate (because the denominator is zero).

This is the practical impact of the 100% zero-duration property. Any aggregate computation that filters out `duration_seconds == 0` will silently drop all 634 rows. Any aggregate that does not filter them out will see them as point masses at zero, distorting any duration-based percentile, mean, or histogram calculation that includes them.

## The aggregate corruption: 6.3% of all sessions are this one project

634 sessions out of 9996 total is **6.34%** of the entire session-queue corpus. If you compute "average session duration across all projects" naively, you are averaging in 6.34% of mass at exactly zero. That drags the global mean down by at least the proportion of mass — and probably more, because the zero-duration rows tend to crowd the left tail where every other row also has small values, distorting the lower-percentile structure.

A concrete example: the top-3 projects together (`8001c27439650c5c`, `0d6e4079e36703eb`, `62f8e1ec095e1857`) account for 3857 + 1740 + 634 = **6231 sessions = 62.3% of the entire corpus**. Of those 6231, the 634 zero-duration sessions from project #3 contribute 10.2% of the top-3 mass. If your dashboard's "median session duration" line jitters week-over-week, it is more likely to reflect changes in the relative volume of `62f8e1ec095e1857` than any change in actual user behavior.

## A second oddity: project rank #1 is partially infected too

`8001c27439650c5c`, the largest project_ref by session count (3857), shows 359 zero-duration sessions — a 9.3% rate. That's not 100%, but it is materially elevated. Compare to `0d6e4079e36703eb` (rank #2, 1740 sessions, 0% zero-duration) and the difference is striking. Two projects of comparable scale, one of them shedding ~10% of its sessions into the duration-zero bucket, the other shedding none.

Working hypothesis: there are two distinct ingest pathways feeding `session-queue.jsonl`. Pathway A produces honest start/end timestamps and never yields zero-duration rows. Pathway B produces start-equals-end timestamps either by design (single-shot workflows) or by snapshot timing (the snapshot happened before the session's second message was indexed). `0d6e4079e36703eb` and `8ab30e9dab447af3` are pure pathway-A projects. `62f8e1ec095e1857` is a pure pathway-B project. `8001c27439650c5c` is mixed: most of its sessions go through pathway A, but ~9% somehow end up on pathway B.

The mixed case is the one worth digging into. A pure-zero project is at least *consistent* — you know it's an observability dead-zone and you can either filter it or document it. A mixed project introduces a per-row decision about whether the duration field is trustworthy, and there is no flag in the row to tell you which side of that line a given session falls on.

## Why this matters for downstream analytics

There are three concrete consequences for anyone building on `session-queue.jsonl`:

1. **Any "session length" distribution is bimodal by construction.** You will see a spike at 0 (driven by `62f8e1ec095e1857` and the zero-duration tail of `8001c27439650c5c`) and a separate distribution for the rest. Reporting a single median or mean across both is misleading. The honest report is two separate distributions plus a count of the zero-mass.

2. **Filtering `duration_seconds > 0` drops one specific project entirely.** That filter is biased — it is not "drop noisy rows", it is "drop project `62f8e1ec095e1857`." If that project represents a meaningful slice of activity (and at 6.34% of corpus, it does), your filtered analysis is no longer representative.

3. **The `total_messages` field is the better activity proxy for these rows.** Even when `duration_seconds = 0`, `total_messages` is non-degenerate — it tells you whether the session had at least one human-assistant exchange. For zero-duration projects, it is the only intra-session signal you have.

## The integrity question

There is a real question worth asking: *should* `session-queue.jsonl` accept rows with `last_message_at == started_at`? An argument for yes: the row carries useful metadata (which model, which source, which project) even if the duration is degenerate. An argument for no: a duration field that is zero in 634 consecutive rows for one project is not a measurement, it is a placeholder, and shipping placeholders as data invites silent corruption of every downstream aggregate.

A middle path would be to nullify `duration_seconds` (write `null`, not `0`) when the start and end timestamps are equal. That makes the degeneracy explicit at the schema level and forces every downstream consumer to make an explicit decision about how to handle it. It would also make the `62f8e1ec095e1857` pattern visible at row-inspection time rather than only in aggregate.

Until then, the operational rule is: if your computation depends on `duration_seconds`, you must either (a) explicitly filter `duration_seconds > 0` and accept that you've dropped one project, or (b) keep zeros in but report them as a distinct mass point alongside the non-zero distribution. There is no clean "ignore the question" option.

## Concrete artifacts to verify

- Total session-queue rows: **9996** (`wc -l ~/.config/pew/session-queue.jsonl`).
- Distinct `project_ref` values: **1322**.
- Sessions for `project_ref = 62f8e1ec095e1857`: **634**.
- Zero-duration rate for that project: **100.0%** (634 of 634).
- Zero-duration rate for the next-most-affected top-10 project (`f5756e707027c9b7`): **39.6%**.
- Share of corpus held by `62f8e1ec095e1857`: **6.34%** (634 / 9996).
- Top-3 project share of corpus: **62.3%**.

Re-derive with:

```
python3 -c "import json,collections; \
c=collections.Counter(); z=collections.Counter(); \
[ (c.update([d.get('project_ref','NONE')]), \
   z.update([d.get('project_ref','NONE')]) if (d.get('duration_seconds',0) or 0)==0 else None) \
   for d in (json.loads(l) for l in open('/Users/bojun/.config/pew/session-queue.jsonl')) ]; \
print('62f8…:', c['62f8e1ec095e1857'], z['62f8e1ec095e1857'])"
```

## The takeaway

`62f8e1ec095e1857` is the cleanest example in the entire corpus of an observability dead-zone that nonetheless contributes a substantial volume of rows. It is not a bug — the rows are well-formed, the schema is honored, the project_ref is consistent. It is a *blind spot*: a region of the data where the schema's most informative timing field collapses to a degenerate value, and where the only honest aggregate is "we know these sessions exist; we know nothing about what happened inside them."

The 100.0% rate is what makes the project diagnostic. A 90% rate would invite the question "what's special about the 10%?" A 100% rate forecloses that question entirely. Either every session in this project is a single-shot interaction, or every session is being snapshotted before its activity is captured. Both possibilities are real; only the second is a fixable bug. The honest next step is to look at one of those 634 rows alongside the underlying source and decide which world we are in.
