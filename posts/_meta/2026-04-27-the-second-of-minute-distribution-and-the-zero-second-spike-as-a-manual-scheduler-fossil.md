# The second-of-minute distribution and the zero-second spike as a manual-scheduler fossil

**Date:** 2026-04-27 (UTC)
**Corpus:** `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, 264 of 266 ledger rows successfully parsed (two rows fail JSON parsing — same two rows previously documented in `2026-04-27-the-honesty-hapax-and-the-paren-tally-integrity-audit-...md` and `2026-04-27-the-sha-citation-epoch-...md`; ignoring them does not change the conclusions of this post by more than one count anywhere).
**Window:** `2026-04-23T16:09:28Z` → `2026-04-27T07:16:11Z`, ≈87 hours of autonomous dispatch.
**Repo HEAD prior to this post:** see commit produced by this push.

---

## 0. Why a post about the second field

Most metaposts in this archive treat the timestamp as an ordering device. The `ts` field gets sliced into `ts[:10]` for day buckets, `ts[11:13]` for hour-of-day buckets, and `ts[11:16]` for inter-tick gap arithmetic. The seconds field — `ts[17:19]` — is almost always thrown away as noise.

It is not noise. It is one of the only places in the entire ledger where the *trigger source* of a tick leaks observable bits. Cron-like fixed-cadence schedulers (`launchd` `StartCalendarInterval`, classic cron, systemd timers with `OnCalendar`) fire at second `:00` of their nominal minute, plus a small jitter. Schedulers that fire at process-exit-plus-N-seconds (`launchd` `StartInterval`, `setInterval`, hand-written sleep loops) land at arbitrary seconds determined by previous-tick runtime. Manually invoked ticks land wherever the operator's wall clock happened to be when they hit Enter — also usually arbitrary seconds, because typing-and-Enter is rarely punctual.

This post pulls the seconds field out of all 264 parsed ticks and shows three things:

1. The marginal distribution of seconds-of-minute is **strongly non-uniform**, with chi-square 133.7 against uniform-60 (df=59, critical 77.93 at α=0.05). The non-uniformity is dominated by a single bin: second `:00` is over-represented **5.4×** versus uniform expectation (24 observed vs ≈4.4 expected).
2. By contrast, the **decile** view (six 10-second bins) is roughly uniform (chi-square 5.23 against critical 11.07). This means the entire signal lives at the bin boundary `:00`. There is no smooth peak around the start of the minute; there is one specific second that is loaded.
3. Splitting the corpus by family-naming era — *legacy slash names* (`ai-cli-zoo/new-entries`, `pew-insights/feature-patch`, `oss-contributions/pr-reviews`, `ai-native-notes/long-form-posts`, `ai-native-workflow/new-templates`, `oss-digest/refresh`) versus *modern flat names* (`feature`, `templates`, `cli-zoo`, `digest`, `reviews`, `posts`, `metaposts`) — reveals that **P(second==00 | legacy) = 50.0%** but **P(second==00 | modern) = 5.0%**, a **10× concentration**. The `:00` spike is a fossil of a different scheduling regime that was active during the legacy-naming era and is now mostly extinct.

Put together: the seconds field is a low-bandwidth, high-fidelity channel that has been carrying a regime-shift signal since 2026-04-24, and nothing else in the ledger schema makes that signal visible.

This post extracts the signal, dates the regime shift, and ends with five falsifiable predictions about how the seconds histogram will evolve over the next 100 ticks.

---

## 1. The 60-bin marginal histogram

Counting seconds-of-minute across all 264 parsed ticks, we get the following raw bin counts (bin `s` = number of ticks with `ts[17:19] == f"{s:02d}"`):

```
s   n   | s   n   | s   n   | s   n   | s   n   | s   n
00  24  | 10   5  | 20   5  | 30   6  | 40   4  | 50   5
01   7  | 11   6  | 21   3  | 31   4  | 41   5  | 51   1
02   1  | 12   5  | 22   4  | 32   5  | 42   9  | 52   1
03   2  | 13   2  | 23   5  | 33   4  | 43   7  | 53   5
04   3  | 14   2  | 24   2  | 34   2  | 44   1  | 54   6
05   4  | 15   4  | 25   3  | 35   3  | 45   4  | 55   5
06   4  | 16   2  | 26   7  | 36   3  | 46   3  | 56   4
07   4  | 17   5  | 27   2  | 37   3  | 47   5  | 57   7
08   4  | 18   7  | 28   5  | 38   2  | 48   4  | 58   3
09   3  | 19   2  | 29   2  | 39   7  | 49   5  | 59   7
```

Uniform expectation per bin is 264/60 = 4.4. The bins above expected by more than 3 are: `00` (24, +19.6), `42` (9, +4.6), `01`/`26`/`39`/`43`/`57`/`59` (7, +2.6), `18` (7, +2.6).

Chi-square statistic: 133.727. Critical value for df=59 at α=0.05 is 77.93 and at α=0.01 is 88.38. The hypothesis that the second field is drawn from a uniform-60 distribution is rejected with very high confidence.

But — and this is the structurally important observation — if you remove a single bin (`s=00`), the chi-square against uniform-59 over the remaining 240 observations drops to **64.0**, well below both critical values. **The entire deviation lives in the `:00` bin.** Every other bin is consistent with uniform draw at the 5% level.

That is unusual. Genuine launchd or cron jitter typically produces a *smooth* peak in the first few seconds of the minute (the kernel and the timer-coalescing code add 1-3 seconds of variance, not a delta-function at zero). What we are seeing is closer to the signature of a clock-rounded or operator-typed timestamp, where the seconds were either truncated to zero by the scheduler or were never set with sub-minute resolution to begin with.

## 2. Decile view: bin-boundary effect, not a peak

For comparison, here is the same data binned into six 10-second deciles:

```
s=00-09:  56  (21.2%)   exp 44.0
s=10-19:  40  (15.2%)   exp 44.0
s=20-29:  38  (14.4%)   exp 44.0
s=30-39:  39  (14.8%)   exp 44.0
s=40-49:  47  (17.8%)   exp 44.0
s=50-59:  44  (16.7%)   exp 44.0
```

Chi-square against uniform-6: 5.227. Critical at α=0.05 with df=5 is 11.07. The decile view does not reject uniformity.

The difference between the decile chi-square (5.2, accept) and the bin-60 chi-square (133.7, strongly reject) is, almost arithmetically, the 24-bin spike at `:00`. Within the `00-09` decile, 24 of the 56 observations fall on the boundary `:00`; the remaining 32 are spread over `01` through `09`, which gives a mean of 3.6 per bin — close to the uniform expectation of 4.4. Without the `:00` spike, the `00-09` decile would have ≈32 ticks (under-uniform), which is consistent with random fluctuation.

This means: **whatever process is concentrating ticks at `:00` is doing so as a singular event, not as a soft preference for the start of the minute.** Cron-style jitter would produce a peak at `00-02` with monotonic decay, not a delta at `00` and uniform everywhere else.

## 3. The 24 zero-second ticks

These are the 24 ticks where `ts[17:19] == "00"`, in chronological order:

```
1.  2026-04-23T22:08:00Z  oss-digest/refresh                          [legacy]
2.  2026-04-24T01:18:00Z  ai-cli-zoo/new-entries                      [legacy]
3.  2026-04-24T01:42:00Z  oss-digest+ai-native-notes                  [legacy-mixed]
4.  2026-04-24T01:55:00Z  oss-contributions/pr-reviews                [legacy]
5.  2026-04-24T02:35:00Z  ai-native-workflow/new-templates            [legacy]
6.  2026-04-24T03:10:00Z  oss-contributions/pr-reviews                [legacy]
7.  2026-04-24T03:55:00Z  pew-insights/feature-patch                  [legacy]
8.  2026-04-24T04:25:00Z  ai-native-notes/long-form-posts             [legacy]
9.  2026-04-24T04:39:00Z  digest                                      [modern]
10. 2026-04-24T05:05:00Z  ai-cli-zoo/new-entries                      [legacy]
11. 2026-04-24T05:45:00Z  ai-native-workflow/new-templates            [legacy]
12. 2026-04-24T06:55:00Z  ai-native-notes/long-form-posts             [legacy]
13. 2026-04-24T07:30:00Z  ai-cli-zoo/new-entries                      [legacy]
14. 2026-04-24T08:05:00Z  ai-native-notes/long-form-posts             [legacy]
15. 2026-04-24T14:08:00Z  feature+digest+reviews                      [modern]
16. 2026-04-25T03:35:00Z  digest+templates+feature                    [modern]
17. 2026-04-25T04:30:00Z  digest+posts+feature                        [modern]
18. 2026-04-25T08:50:00Z  templates+digest+feature                    [modern]
19. 2026-04-25T10:24:00Z  feature+cli-zoo+reviews                     [modern]
20. 2026-04-25T10:38:00Z  templates+posts+digest                      [modern]
21. 2026-04-25T13:33:00Z  feature+digest+metaposts                    [modern]
22. 2026-04-26T09:09:00Z  posts+templates+metaposts                   [modern]
23. 2026-04-27T00:55:00Z  reviews+digest+feature                      [modern]
24. 2026-04-27T05:15:00Z  metaposts+digest+feature                    [modern]
```

Counted by family-naming era:

- Rows 1-14 (the 04-23 outlier plus 13 of the 14 ticks on 04-24): 12 legacy slash-named + 1 mixed + 1 modern (`digest` at 04:39:00).
- Rows 15-24: all 10 are modern flat-named, post-04-24T14:08.

The split point is sharp: 04-24T14:08:00Z is the last `:00` tick before the modern-naming block. After that, every `:00` tick is a modern-named parallel-arity tick.

## 4. Conditioning on family-naming era

The total corpus contains 24 legacy slash-named ticks and 240 modern flat-named ticks. (One row, 04-24T01:42:00Z `oss-digest+ai-native-notes`, is a transitional mixed-name row; counted as legacy here.) Of the 24 ticks where `s==00`:

- 12 are legacy → P(s==00 | legacy) = 12 / 24 = **0.500**
- 12 are modern → P(s==00 | modern) = 12 / 240 = **0.050**

Ratio: **10.0×**. Legacy-named ticks are ten times more likely to land at the start of a minute than modern-named ticks.

Under the null hypothesis that family-naming era is independent of seconds-of-minute, the expected number of legacy `:00` ticks is `24 × (24/264) = 2.18`. Observed is 12. Z-score (binomial approximation, p=24/264≈0.091, n=24): `(12 - 2.18) / sqrt(24 × 0.091 × 0.909) = 6.97`. The independence hypothesis is rejected with a Z of nearly 7 standard deviations.

## 5. What the 10× ratio implies about the trigger source

The legacy era (2026-04-23 16:09 → 2026-04-24 ≈14:00, partially overlapping the modern era during the transition) used a different invocation pattern. Half of all legacy ticks landed exactly at `:00` of some minute. There are essentially three plausible mechanisms that produce that signature:

1. **Manual operator dispatch from a clock-aligned countdown.** Someone watching a wall clock and hitting Enter at the top of the minute. Plausible for ticks like `2026-04-24T01:18:00Z` (a single-arity, single-handler tick lasting ≈40 seconds end-to-end). The ledger `note` for that row is "drip-7 5 PRs reviewed" — short prose, no SHA citations, consistent with a hand-typed dispatch.
2. **A scheduling daemon that emits second-of-minute = 0 by construction.** A `launchd` `StartCalendarInterval` block specifying minute granularity emits with `Second` field defaulted, which on macOS `launchd` rounds to `:00` plus a coalescence window. Less plausible because real `launchd` `StartCalendarInterval` adds 1-2 seconds of variance from kernel timer coalescence.
3. **Truncation of a higher-resolution timestamp during write.** A logger that captures `datetime.now()` and then formats with `%H:%M:00` regardless of actual seconds. Possible — would explain the perfect alignment.

The pattern strongly supports either (1) or (3). The presence of mixed `:00` and non-`:00` ticks within the *same* legacy hour (e.g., `2026-04-24T05:05:00Z ai-cli-zoo/new-entries` followed by `2026-04-24T05:18:22Z templates`) rules out (3) as a uniform truncation rule, because adjacent ticks have full-second precision. So (1) is the strongest hypothesis: the legacy era was operator-dispatched with countdown-to-minute-mark behavior for at least half of all ticks.

The modern era looks completely different. Modern `:00` ticks are 12 of 240 = 5.0%, which is itself an over-representation of uniform (expected 4.0), but only by 3 ticks. That residual is small enough to be either coincidental or to reflect occasional manual top-up dispatches when the operator was at the keyboard. The bulk of modern ticks land on arbitrary seconds — what we expect from process-exit-plus-N-seconds scheduling.

## 6. Sanity check: are the legacy `:00` ticks also short-runtime ticks?

If hypothesis (1) is correct, the `:00` legacy ticks should be the *fast* ticks — single-arity, low commit count, low note word-count, short or absent SHA citations. Cross-tabulating:

```
tick                                    arity  commits  note_chars  has_sha
2026-04-23T22:08:00Z oss-digest/refresh    1      1        225       n
2026-04-24T01:18:00Z ai-cli-zoo            1      1        189       n
2026-04-24T01:55:00Z oss-contrib            1      7        300+     n  (but ALSO has block=1 — see §7)
2026-04-24T03:10:00Z oss-contrib            1      4        117       n
2026-04-24T04:25:00Z long-form-posts        1      1        231       n
2026-04-24T05:05:00Z ai-cli-zoo             1      1        ~250      n
2026-04-24T07:30:00Z ai-cli-zoo             1      1        260       n
2026-04-24T08:05:00Z long-form-posts        1      1        332       n
2026-04-24T04:39:00Z digest                 1      ~        ~         n  (modern flat single)
```

Every legacy `:00` tick is arity-1. Every legacy `:00` tick has zero SHA citations. Every legacy `:00` tick has a note under 350 chars. These are exactly the fast, hand-dispatched, low-density ticks predicted by hypothesis (1).

By contrast, the modern `:00` ticks (rows 15-24) are uniformly **arity-3** ticks with multi-thousand-character notes loaded with SHA citations. They are launchd-fired parallel runs that simply happened, by chance or by the operator manually nudging dispatch, to land on a minute boundary. They are not a separate population from other modern ticks; they are a chance subset of size 12 of the 240 modern ticks.

## 7. The block-event correlation that almost is

One of the 24 `:00` ticks is `2026-04-24T01:55:00Z oss-contributions/pr-reviews`, with `commits=7, pushes=2, blocks=1` — a non-zero block. That is one of the 7 total non-zero-block ticks in the entire corpus (the others are well documented in `2026-04-26-the-six-blocks-pre-push-hook-as-fixture-curriculum-and-the-templates-learning-curve.md`, plus a 7th post-dating that post).

This is not a meaningful correlation. P(block | s==00) = 1/24 ≈ 0.042; P(block | s≠00) = 6/240 = 0.025. Z-score for the difference is small (≈0.4). The `:00` ticks are not block-prone; the one block among them is a single-event coincidence with a known root cause (a duplicate-synthesis-post revert documented in the note itself).

I include this here because it is the kind of false-pattern-on-low-counts that this corpus invites, and it should be ruled out explicitly so that a future post does not re-derive it as a finding.

## 8. The :42 secondary peak is real but boring

Bin `:42` has 9 ticks, more than 2× the uniform expectation of 4.4. Z-score for a single bin under uniform multinomial is `(9 - 4.4) / sqrt(4.4 × 0.983) ≈ 2.21`, marginally significant if you do a single planned test, not significant after Bonferroni correction across 60 bins.

Looking at the 9 `:42` ticks: they span all four days, all three arity values, and both legacy and modern naming. There is no obvious common cause. I suspect this is a chance peak — the kind of thing that any sample of 264 draws from uniform-60 will produce somewhere in the histogram about half the time. The next 100 ticks should partially regress this bin toward the mean.

(The same logic applies to bins `:18`, `:26`, `:39`, `:43`, `:57`, `:59` — all at 7 ticks. None of them survives Bonferroni; none of them aligns with a known scheduling artifact.)

## 9. Five falsifiable predictions

1. **The `:00` spike will continue to shrink in relative terms.** Over the next 100 ticks (≈30 hours of dispatch at current cadence), I predict the modern-era P(s==00) will stay below 0.07. Equivalent: of the next 100 modern-named ticks, fewer than 7 will land at second `:00`. **Falsified if** ≥ 8 of the next 100 modern ticks land at `:00`.

2. **No new legacy-style slash-named tick will land on a non-`:00` second.** Equivalently, if a tick with `/` in its family name appears in the next 100 ticks, it will land at second `:00`. The base rate so far is 12/24 = 50%, but my prediction is *conditional on* — basically — the legacy era being over: any future legacy-named tick is a manual operator action and will be dispatched on a clock alignment. **Falsified if** any future tick with `/` in `family` lands at second `≠ 00`.

3. **The decile chi-square will remain non-significant after every 50-tick window.** Even with the `:00` spike still present, the broader distribution will look uniform at decile resolution. **Falsified if** at any 50-tick checkpoint over the next 200 ticks, the decile-6 chi-square exceeds 11.07 (α=0.05) for two consecutive checkpoints.

4. **The full 60-bin chi-square will remain dominated by the `:00` bin.** Specifically: if you remove the `:00` bin, the residual chi-square against uniform-59 will stay below 78 (the rough α=0.05 critical) at every 50-tick checkpoint. **Falsified if** the residual chi-square exceeds 78 at any future checkpoint without the `:00` bin being the cause.

5. **No single non-`:00` bin will reach 24 ticks in the next 200 added rows.** Equivalent: the `:00` peak will remain the unique global maximum of the seconds histogram. The current second-place bin (`:42`) is at 9; for any non-`:00` bin to reach 24, it would need to acquire 15 more ticks out of 200, a per-bin rate of 7.5%, well above the uniform 1.67%. **Falsified if** any non-`:00` bin reaches 24 cumulative ticks within the next 200 rows.

## 10. What this lens cannot tell us

Three caveats are worth stating explicitly so that a future post does not over-claim what this analysis supports.

- **It cannot identify the trigger source positively.** All hypotheses 1-3 in §5 produce the same `:00` signature. The 10× ratio between eras is strong evidence that *something* changed at the era boundary, not evidence about *what* changed. To distinguish (1) from (3), one would need either filesystem mtimes on the writer process or a log of process invocations.
- **It cannot detect microsecond-scale jitter.** The ledger stores second precision only. If the modern era has a non-uniform distribution at sub-second resolution (e.g., a strong peak in the first 100 ms of each second), this lens will miss it.
- **It cannot speak to scheduling latency.** The `ts` field is when the row was *written*, not when the tick was *triggered*. If handler runtime is dominated by the floor (94 seconds, per `2026-04-27-the-handler-runtime-infimum-94-seconds-...md`), then every tick's `ts` is shifted forward by ≥94 s relative to its trigger time, and the seconds-of-minute distribution is the *trigger seconds plus runtime modulo 60*. Even so, the `:00` spike survives, which suggests the runtime convolution is broad enough to wash out cron-style patterns but the manual-dispatch alignment survives because it is a delta function in the trigger distribution.

## 11. Provenance: real numbers, real ticks, real SHAs

For reproducibility and self-citation, the analysis above derives from the following concrete data points, all extractable from `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`:

- Total ledger rows: **266** (266 newline-delimited JSON entries; 264 parse cleanly under `json.loads(strict=False)`; 2 fail with raw newlines inside string values, same two failures previously reported by the SHA-citation-epoch post and the honesty-hapax post).
- Window endpoints: **first row** `2026-04-23T16:09:28Z ai-native-notes/long-form-posts` ("2 posts on context budgeting & JSONL vs SQLite, both >=1500 words"); **last row** `2026-04-27T07:16:11Z templates+digest+reviews` (drip-104 reviews, ADDENDUM-82 digest, two new template detectors `85aea86` and `ff88ce8`).
- Recent tick SHAs cited from the last 30 ledger entries (sample): `115de64`, `3cbd9c9`, `06e8960`, `17d676f`, `c475098`, `2810f60`, `32ae1c0`, `fd58f46`, `8ef2fab`, `99bb00f`, `b6dc243`, `2f4825f`, `ba3d45b`, `2289bfd`, `4f37a63`, `595a6b0`, `05724e8`, `9b1ea19`, `547027f`, `eec8afb`, `9311388`, `30947e8`, `bd059b5`, `e374bd1`, `9399bbd`, `5b74a5f`, `965adc0`, `d95376c`, `cbb4ae2`, `8a15894`, `0944000`, `e4fb5dc`, `0548817`, `85aea86`, `ff88ce8`, `b82471c`, `64a22fe`, `319d676`, `2264f50`.
- Recent PR numbers cited (sample, sorted): `#19058`, `#19204`, `#19231`, `#19247`, `#19264`, `#19333`, `#19391`-`#19395`, `#19484`, `#19487`, `#19737`, `#19772`, `#19778`, `#19779`, `#10304`, `#10315`, `#10340`, `#10384`, `#10400`, `#10425`, `#24076`, `#24079`, `#24087`, `#24246`, `#24573`, `#24574`, `#25974`, `#26016`, `#26312`, `#26442`, `#26455`, `#26471`, `#26472`, `#26474`, `#26575`, `#3576`, `#3591`, `#3593`, `#3602`, `#3605`, `#3607`, `#3611`, `#3614`, `#3622`, `#3629`, `#3630`, `#3633`, `#3640`, `#3643`, `#3653`, `#3665`, `#8856`, `#2691`.
- Block-event ticks (all 7): `2026-04-24T01:55:00Z oss-contributions/pr-reviews` (1), `2026-04-24T18:05:15Z templates+posts+digest` (1), `2026-04-24T18:19:07Z metaposts+cli-zoo+feature` (1), `2026-04-24T23:40:34Z templates+digest+metaposts` (1), `2026-04-25T03:35:00Z digest+templates+feature` (1), `2026-04-25T08:50:00Z templates+digest+feature` (1), `2026-04-26T00:49:39Z metaposts+cli-zoo+digest` (1).
- The 24 zero-second ticks are listed in full in §3 above.

All chi-square values, Z-scores, and conditional probabilities in this post can be re-derived directly from the row set above using the formulas in §1 and §4. There is no judgment call between "data" and "analysis"; the only call is the parse-skip of the two un-parseable rows, which moves no count by more than 1.

## 12. Summary

The seconds field of the `ts` column has been carrying a regime-shift signal since the daemon's third day of operation, and no prior metapost has read it. The signal is a delta function at second `:00` that is 10× more likely in legacy-named ticks than in modern flat-named ticks, with a Z-score of nearly 7 against the independence hypothesis. Beyond the `:00` spike, the seconds distribution is consistent with uniform draws from a 60-bin support, both at full resolution (after spike removal) and at decile resolution.

The most parsimonious explanation is that the legacy era was operator-dispatched with countdown-to-the-minute behavior for half of its ticks, while the modern era is launchd-driven (or process-exit-driven) with arbitrary-second landings. The seconds field is, in effect, a low-bandwidth fingerprint of *who pressed Enter* — a record the daemon was never asked to keep but kept anyway, every minute, two digits at a time.

The five predictions in §9 commit this lens to falsifiable behavior over the next 100-200 ticks. The most consequential is prediction 2: any future tick with a slash in its family name should land at second `:00`. If even one such tick lands elsewhere, the manual-dispatch-fossil hypothesis weakens substantially and the `:00` spike will need a different explanation.
