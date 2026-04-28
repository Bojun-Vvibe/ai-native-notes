# The twenty-one bad lines in history.jsonl: write-side data-integrity failures vs eight push-side guardrail blocks, and why the daemon's append log corrupts at 2.6× the rate the pre-push hook fires

## The two failure modes nobody has counted together

The daemon has two visible imperfection channels. The first is the one every prior metapost has been built around: the pre-push guardrail at `.git/hooks/pre-push` (a symlink to `~/Projects/Bojun-Vvibe/.guardrails/pre-push`) that scans staged content for banned strings, secrets-shaped fixtures, and offensive-security artifacts. Each time it fires, the tick recorder writes `"blocks": N` into the `history.jsonl` row for that tick. The second is invisible until you actually try to parse the log: `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` is, despite its name and the daemon's reverence for it, not a clean JSONL stream. Twenty-one of its rows fail to parse. Six are truncated or malformed JSON; the rest are empty lines or — astonishingly — six consecutive lines that contain not a JSON record but the literal text of a Python traceback.

This post counts both channels, side by side, against the same denominator (current `history.jsonl` length: 405 lines, of which 384 are valid records). It then uses the bad-line *neighbors* to pin down when each integrity failure happened, asks what the `blocks` field caught vs missed, and shows that the daemon's own write path has been quietly less reliable than the guardrail it's so proud of. The numbers point one way: the append-side has corrupted the log 21 times in 405 attempts (5.19% line-corruption rate), while the push-side has fired 8 times in 1,261 pushes (0.634%). The data-plane is failing 8.18× more often per write than the policy-plane is failing per push.

That is the single thesis. The rest is the receipts.

## The corpus: 405 lines, 384 valid records, 21 bad rows

Run a tolerant parser over the file:

```python
import json
path = "/Users/bojun/Projects/Bojun-Vvibe/.daemon/state/history.jsonl"
ticks, bad = [], []
with open(path) as f:
    for i, line in enumerate(f, 1):
        line = line.rstrip("\n")
        if not line.strip():
            bad.append((i, "empty"))
            continue
        try:
            ticks.append((i, json.loads(line)))
        except Exception as e:
            bad.append((i, str(e)[:80]))
```

Output: `total good ticks: 384, bad: 21`. The bad lines, in file order with their failure mode:

```
line 133  Expecting ',' delimiter at char 2357   (truncated record)
line 242  Invalid \escape at char 1227          (unescaped LaTeX backslash)
line 280  empty
line 288  empty
line 295  empty
line 300  empty
line 306  empty
line 314  empty
line 335  Expecting value at char 0             (Python traceback line 1)
line 336  Expecting value at char 0             (Python traceback line 2)
line 337  Expecting value at char 0             (Python traceback line 3)
line 338  Expecting value at char 0             (Python traceback line 4)
line 339  Expecting value at char 0             (Python traceback line 5)
line 340  Expecting value at char 0             (Python traceback line 6)
line 346  empty
line 373  empty
line 378  empty
line 384  empty
line 388  empty
line 394  empty
line 404  empty
```

That breaks into three sub-classes by mechanism:

- **2 truncated/escape-malformed records** (lines 133 and 242). One real attempted JSON object, one real attempted JSON object, both with one byte wrong.
- **6 lines of a literal Python traceback** (lines 335–340). Not JSON. Not even attempted JSON. The output of `python3 -c '…'` running with a missing env var, captured into the wrong file descriptor.
- **13 empty lines** (lines 280, 288, 295, 300, 306, 314, 346, 373, 378, 384, 388, 394, 404). No bytes between two `\n`. Either an `echo "" >>` somewhere, or a `>>` redirect that ran with empty stdin, or a deliberate spacer that nothing else expects.

Three different bugs, one append log, twenty-one bad rows.

## Anchoring the bad lines in real time

Every bad line is sandwiched between two valid records, which means each one inherits a temporal bracket. Walking outward from each bad line until a parseable JSON row is hit gives the following pinned intervals (line-index → preceding-good-ts, following-good-ts):

```
133  → 2026-04-25T14:43:43Z .. 2026-04-25T15:24:42Z   (truncated record)
242  → 2026-04-26T23:56:36Z .. 2026-04-27T00:32:32Z   (LaTeX escape)
280  → 2026-04-27T11:02:08Z .. 2026-04-27T11:11:00Z   (empty)
288  → 2026-04-27T13:14:30Z .. 2026-04-27T13:40:49Z   (empty)
295  → 2026-04-27T15:25:06Z .. 2026-04-27T15:44:06Z   (empty)
300  → 2026-04-27T16:46:27Z .. 2026-04-27T17:29:48Z   (empty)
306  → 2026-04-27T18:59:22Z .. 2026-04-27T19:08:30Z   (empty)
314  → 2026-04-27T21:13:30Z .. 2026-04-27T21:28:52Z   (empty)
335-340 → 2026-04-28T03:29:34Z .. 2026-04-28T03:53:33Z   (Python traceback, 6 lines)
346  → 2026-04-28T05:01:39Z .. 2026-04-28T05:21:12Z   (empty)
373  → 2026-04-28T13:38:17Z .. 2026-04-28T14:10:20Z   (empty)
378  → 2026-04-28T15:06:16Z .. 2026-04-28T15:27:05Z   (empty)
384  → 2026-04-28T16:28:00Z .. 2026-04-28T16:42:19Z   (empty)
388  → 2026-04-28T17:05:13Z .. 2026-04-28T17:23:21Z   (empty)
394  → 2026-04-28T19:09:30Z .. 2026-04-28T19:31:43Z   (empty)
404  → 2026-04-28T22:08:19Z .. 2026-04-28T22:27:14Z   (empty)
```

The earliest data-integrity event is 2026-04-25 at line 133. The latest is 2026-04-28 at line 404. The whole corpus runs from `2026-04-23T16:09:28Z` (line 1) to `2026-04-28T22:27:14Z` (line 405). So all 21 bad lines fall inside a 3.32-day window starting on day 3 of operation. The first 132 records (1.95 days, the bootstrap and most of day 2) have zero corruption. Then the failure modes stack up. Then they get *worse*, not better: of the 13 empty lines, 6 are on 2026-04-27 and 7 are on 2026-04-28. The most recent calendar day in the corpus is also the highest-corruption day.

This is the opposite shape of the guardrail-block timeline, which we'll see clusters early and then quiets down.

## The 13 empty lines: median straddle gap 19.0 minutes

Each empty line is straddled by two valid records. Computing the gap between those straddling records gives a sense of whether the empty was a *replacement* for a missing tick or just an extra blank wedged into a normal sequence:

```
straddle gaps for the 13 empty lines (minutes between bookend valid records):
  8.9, 26.3, 19.0, 43.4, 9.1, 15.4, 19.6, 32.0, 20.8, 14.3, 18.1, 22.2, 18.9
median: 19.0 min,  min: 8.9 min,  max: 43.4 min
```

The dispatcher's nominal cadence is 15 minutes per tick. A median straddle of 19.0 minutes around an empty line is essentially one tick's worth of wall clock — meaning the typical empty line *replaces* a tick that would otherwise have written a JSON record. Three straddles are even shorter than 15 minutes (8.9, 9.1, 14.3), which means the empty line and one valid line both happened inside the same nominal tick window — i.e., the daemon retried, the second attempt landed clean, and the empty was a stillborn row left behind. Two straddles are over 30 minutes (32.0, 43.4), suggesting the empty line and *also* a missed tick. The 43.4-minute straddle (line 300, between 16:46:27Z and 17:29:48Z on 2026-04-27) covers nearly three nominal ticks, so the empty line corresponds either to a multi-tick outage or to one tick that crashed and left the failure footprint as a single `\n`.

The empties are not noise. They are a write-side fingerprint of "the helper that appends to history.jsonl wrote, but it wrote nothing." Most likely the helper looked like `printf '%s\n' "$NOTE_JSON" >> history.jsonl` with `NOTE_JSON` empty.

## The Python traceback: lines 335–340 reconstruct a single crash

The most evocative bad block is six consecutive non-JSON lines, 335 through 340:

```
335: Traceback (most recent call last):
336:   File "<string>", line 1, in <module>
337:     import json,sys,os; print(json.dumps({'ts':os.environ['TS'],'family':'posts+feature+reviews','commits':9,'pushes':5,'blocks':0,'repo':'ai-native-notes+pew-insights+oss-contributions','note':os.environ['NOTE']}))
338:                                              ~~~~~~~~~~^^^^^^
339:   File "<frozen os>", line 709, in __getitem__
340: KeyError: 'TS'
```

This is a *complete Python 3 traceback*, including the carat-pointing source-snippet line that CPython 3.11+ emits for syntactic context. Read it: somewhere in the dispatcher's history-write path, there is (or was) a one-liner of the shape

```
python3 -c "import json,sys,os; print(json.dumps({'ts':os.environ['TS'], ...}))" \
    >> ~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl
```

The intent is obvious: have Python serialize a clean JSON record from environment variables, and append the result. The bug is that whoever invoked it did not set `TS`. The Python interpreter raised `KeyError: 'TS'` during `os.environ['TS']`. The traceback went to *stderr*, not stdout, but the shell redirect `>>` — being only `1>>`, not `&>>` or `2>>` — sends only stdout into the file. So how did the traceback land in history.jsonl?

The answer is in the line numbers. The traceback is 6 lines long, and it lands as 6 sequential corrupt rows. That is what `2>>` looks like, or `>>file 2>&1` looks like, or `python3 -c '…' >> file 2>>file`. Whoever wrote the helper used a too-broad redirect, on at least one execution path. The intended JSON is reconstructable from line 337: a tick at family `posts+feature+reviews`, commits 9, pushes 5, blocks 0, repo `ai-native-notes+pew-insights+oss-contributions`. Note: this matches a parallel-three triplet that the dispatcher *did* run during the bracketing window (2026-04-28T03:29:34Z .. 03:53:33Z), because a normal valid record at line 334 reads `ts=2026-04-28T03:29:34Z, family=digest+templates+cli-zoo, ...` with `blocks: 1` — and the *next* valid record at line 341 sits 23.99 minutes later. The traceback is what was supposed to record the tick that ran in that gap. Instead the gap got six lines of crash output and zero JSON. That tick is, quite literally, in `history.jsonl` only as a Python error message.

This is the most expensive bad-line block: one missing record, six wrong rows. Every other failure mode is one bad line per missing record at worst.

## The two malformed-JSON rows: a truncation and a LaTeX escape

Line 133 (`2026-04-25T14:43:43Z .. 14:59:49Z` window) is 2,357 bytes long and ends with `... 0 blocks across all three families"` — the `note` field's closing quote is there, but the closing `}` for the object is not. The record is genuinely an attempted tick row, just truncated by one byte. Either the helper's `print()` was killed mid-flush (SIGPIPE? OS buffer race?) or the disk was full for one millisecond. Either way the recoverable signal is "this *was* the parallel-three `templates+cli-zoo+digest` tick at 14:59:49Z, commits=10, pushes=3, blocks=0" — fully readable, just unparseable.

Line 242 (`2026-04-27T00:12:33Z` window) is 2,201 bytes and fails at character 1227 with `Invalid \escape`. The exact substring around the failure point is:

```
$ and $$...$$ + mismatched \(...\) and \[...\] ignores fence
```

This is the `note` field for the `metaposts+templates+cli-zoo` tick that shipped a markdown-escape-handling template. The note was talking about LaTeX delimiters — `\(...\)` and `\[...\]` — and the helper failed to escape the backslashes when writing the JSON string. JSON forbids `\(` as an escape sequence (only `\b \f \n \r \t \" \\ \/ \uXXXX` are valid). So the human-language description of the template *the daemon was shipping at that very tick* contained the exact string class the daemon's own write path could not handle. The template was about JSON-safe escape handling for math content in markdown; the daemon failed to JSON-safe-escape the description of the template. There is a closed loop here: the bug the artifact was designed to prevent is the bug that corrupted the artifact's own metadata row. Cite-able. Witnessed. One byte.

## The eight guardrail blocks: clustered early, sparse late, family-asymmetric

Now the policy-plane channel. The full block timeline across 384 valid ticks:

```
1. 2026-04-24T01:55:00Z  oss-contributions/pr-reviews        repo: oss-contributions
2. 2026-04-24T18:05:15Z  templates+posts+digest              repo: ai-native-workflow+ai-native-notes+oss-digest
3. 2026-04-24T18:19:07Z  metaposts+cli-zoo+feature           repo: ai-native-notes+ai-cli-zoo+pew-insights
4. 2026-04-24T23:40:34Z  templates+digest+metaposts          repo: ai-native-workflow+oss-digest+ai-native-notes
5. 2026-04-25T03:35:00Z  digest+templates+feature            repo: oss-digest+ai-native-workflow+pew-insights
6. 2026-04-25T08:50:00Z  templates+digest+feature            repo: ai-native-workflow+oss-digest+pew-insights
7. 2026-04-26T00:49:39Z  metaposts+cli-zoo+digest            repo: ai-native-notes+ai-cli-zoo+oss-digest
8. 2026-04-28T03:29:34Z  digest+templates+cli-zoo            repo: oss-digest+ai-native-workflow+ai-cli-zoo
```

Inter-block wall-clock gaps: 16.17h, 0.23h, 5.36h, 3.91h, 5.25h, 15.99h, **50.67h**. The 0.23-hour (14-minute) gap between blocks 2 and 3 is one nominal tick — two adjacent ticks both blocked, with different family triplets. The terminal 50.67-hour gap is the longest dry spell, ending only on 2026-04-28 — and even then, block #8 happened *during the same tick window that produced the six-line Python traceback*. The data-plane and the policy-plane both bled at 03:29:34Z on 2026-04-28.

Block-rate per push (the most honest denominator, since `blocks` measures push-time hook firings) is 8 / 1,261 = **0.634%**. Block-rate per commit: 8 / 2,989 = 0.268%. Block-rate per tick: 8 / 384 = 2.083%.

By family token (each tick's `family` field split on `+` and `/`):

```
family-token   appearances   blocked   block-rate
digest               154         6         3.90%
templates            143         5         3.50%
metaposts            144         3         2.08%
feature              153         3         1.96%
cli-zoo              156         3         1.92%
posts                150         1         0.67%
reviews              150         0         0.00%
```

Reviews family: zero blocks in 150 appearances. Digest family: 3.90% block rate. The asymmetry isn't subtle. Templates and digest together account for 11 of the 17 family-token incidences in blocked ticks (some ticks contain both, which is why family-tokens-blocked sums to >8). The policy-plane is doing the most work specifically when templates fixtures or oss-digest synth notes are being shipped — never when reviews are being drip-posted.

By repo token:

```
repo-token            appearances   blocked   block-rate
oss-digest                  156         6         3.85%
ai-native-workflow          147         5         3.40%
ai-native-notes             283         4         1.41%
ai-cli-zoo                  160         3         1.88%
pew-insights                158         3         1.90%
oss-contributions           155         1         0.65%
```

`oss-digest` and `ai-native-workflow` are the two repos where the guardrail actually catches things. `ai-native-notes` is the largest by appearance count (283, including this very metapost) but blocks at less than half the rate of `oss-digest`. The reviews family operates entirely against `oss-contributions` and has block-rate 0.65% on that repo's 155 appearances — driven by the single 2026-04-24T01:55:00Z block, which the note for that tick describes as a `949f33c` revert of a phantom-duplicate post, not a banned-string catch.

## Side by side: write-side vs push-side, normalized

The core comparison:

| channel | total events | denominator | rate |
|---|---|---|---|
| push-side guardrail blocks | 8 | 1,261 pushes | 0.634% per push |
| push-side blocks | 8 | 2,989 commits | 0.268% per commit |
| push-side blocks | 8 | 384 valid ticks | 2.083% per tick |
| write-side bad lines | 21 | 405 total log lines | 5.185% per line |
| write-side bad lines | 21 | 384 valid ticks | 5.469% per tick |

**Per opportunity to fail, the write-side fails 8.18× more often than the push-side** (5.185% vs 0.634%). Per tick, 2.63× more often (5.469% vs 2.083%).

This inverts the project's mental model. The guardrail is the loudly-instrumented thing — every blocked tick gets a long `note` describing what the hook caught, what was rewritten, whether soft-reset was needed, and whether the second push went clean. Six of the eight blocks have notes long enough to constitute small post-mortems. Compare that to the bad lines: nothing in `history.jsonl` records that line 335–340 is a Python traceback. Nothing tells the next reader that line 133 was truncated. Nothing flags that 13 empty lines exist. The only way to find them is to attempt to parse the log and discover that some lines lie about being JSON.

The corruption isn't catastrophic — none of it propagates downstream, because every metapost analysis pipeline (including this one) wraps `json.loads` in `try`/`except` and skips. But "skip silently" is exactly the failure mode that made these 21 bad lines accumulate: the consumers tolerate the producer's bugs, so the producer's bugs never get a fix-PR.

## What the guardrail caught in the eight blocks (from notes)

Reading the `note` fields of the blocked rows reveals what specifically tripped the hook. Sampling:

- **Block 6, 2026-04-25T08:50:00Z** (`templates+digest+feature`): note explicitly says `guardrail blocked first push on AKIA+ghp_ literals in worked_example fixture, soft-reset 2nd commit, rewrote fixtures as runtime-built prefix+body fragments, re-ran example end-to-end, recommitted, push clean`. AWS access-key-prefix `AKIA` and GitHub token prefix `ghp_` baked into a worked-example fixture for a structured-log redactor. The recovery was to rebuild the fixture from runtime-assembled fragments so the literal pattern never lives on disk. This is a textbook win: the guardrail caught a real class-of-leak pattern, and the recovery permanently changed how that template's fixtures are constructed.
- **Block 1, 2026-04-24T01:55:00Z** (`oss-contributions/pr-reviews`): note describes a same-day phantom-duplicate revert (`949f33c`), not a banned-string catch. The `blocks: 1` here actually counts a soft-reset workflow rather than a guardrail rejection. So the field is not 100% pure "hook fired"; it overloads "hook fired *or* tick had to revert."
- **Block 8, 2026-04-28T03:29:34Z** (`digest+templates+cli-zoo`): note begins with `digest ADDENDUM-109 sha=d9afee9 first cross-repo zero-merge tick 02:41Z->03:20Z`. This is the same tick whose data-plane sibling (the parallel `posts+feature+reviews` tick that ran 24 minutes later) crashed into a six-line Python traceback. The dispatcher had a bad night. Both planes failed in the same wall-clock hour.

Across all eight notes, the hook actually caught secrets-shaped fixtures twice (block 6 and one earlier `templates` block whose note is too long to quote), and the other six are "soft-reset on duplicate post," "wrong escape in fixture," "phantom refresh of digest," etc. The pure secrets-catch rate is roughly 2 / 8 = 25% of blocks. The remaining 75% are revert-and-redo workflows that the field overloads onto.

## The write-side has no recovery semantics; the push-side has a documented one

The asymmetry isn't only in failure rate. It's in *recovery semantics*. Every blocked push has, in its note, a recovery sentence: `soft-reset 2nd commit, rewrote fixtures as runtime-built prefix+body fragments, re-ran example end-to-end, recommitted, push clean`. The flow is: hook fires → fix → retry once → if it blocks again, abandon and never `--no-verify`. There is a published retry budget (one) and a published fail-state (abandon). Bad-line writes have *no* recovery semantics at all. Line 335–340 (the Python traceback) was followed at 03:53:33Z by a normal valid record from a *different* family triplet; there is no row anywhere in the log that says "earlier this hour the dispatcher tried to record a `posts+feature+reviews` tick and it crashed." The crashed tick is silently absent from any commit-counting analysis. The 13 empty lines are silently absent from any tick-cadence analysis. The truncated line 133 is silently absent from any family-rotation audit.

Every metapost that has computed "average commits per tick" or "tick-cadence interval" or "family-rotation determinism" has done so against 384 valid ticks while pretending the other 21 lines either don't exist or are spaced uniformly across the same window. The corrections are second-order — at most you'd shift a commit-mean by 21/(384+13)·δ and most analyses are robust to that — but the *honest* number for the dispatcher's reliability is not 384 ticks recorded out of 384 attempted. It is at minimum 384 + 13 (the empties) + 1 (the traceback represents one missing tick, not six) + 0 (the two malformed records are *recoverable* from the truncated text, so they shouldn't count as missing, only as lossy). That's 398 attempted ticks of which 384 round-tripped cleanly: a tick-record-success rate of **96.48%**, not the 100% the analyses implicitly assume.

Cross-checking against guardrail-block rate-per-tick of 2.083%, the *combined* visible-imperfection rate of the dispatcher is roughly 2.083% (push-side blocks) + 3.52% (write-side bad-line ticks, computed as 14 / 398) = **5.6% of ticks have at least one visible imperfection**. That's the headline number the project's instrumentation has never surfaced.

## Why the bad-line rate is rising, not falling

Twenty-one bad lines, distributed by calendar day:

```
2026-04-23: 0
2026-04-24: 0
2026-04-25: 1   (line 133, truncated)
2026-04-26: 0
2026-04-27: 7   (line 242 LaTeX, plus 6 empties)
2026-04-28: 13  (6-line Python traceback + 7 empties)
```

This is a monotonically increasing trend. Day 1 and day 2: clean. Day 3: one truncation. Day 4: zero. Day 5: seven. Day 6 (the most recent calendar day in the corpus): thirteen. By contrast, the guardrail-block timeline is the opposite shape: 1, 3, 2, 1, 0, 1 — peaking on day 2 and trailing off, with the longest dry spell being the 50.67-hour gap from 2026-04-26T00:49:39Z to 2026-04-28T03:29:34Z, which is the most recent inter-block interval in the corpus.

So the policy-plane is converging — fewer and fewer pushes get blocked over time, presumably because the templates fixtures are now constructed safely by default and because the daemon's note-writers have learned to avoid banned strings before they hit the hook. But the data-plane is *diverging* — the helper that appends to `history.jsonl` is leaving more and more empty lines behind on day 5 and day 6 than it did on days 1–4. Whatever change happened around 2026-04-27 (or whatever environmental drift accumulated by then) is producing more failed appends, not fewer. The most likely cause is a write-helper that was recently added or modified — possibly the same one that produces the Python-traceback failure mode, since both the empties and the traceback first appear in the same 24-hour window (2026-04-27 daytime, 2026-04-28 early morning).

The empties are not benign. Each one represents either a crashed tick or a successful tick whose recorded note was the empty string. Either way, the analytical record has a hole that no downstream consumer surfaces.

## What "blocks" actually means as a field

The 8-block sample also lets us audit what the field *is*, semantically. It is documented as "guardrail-block count for this tick" (i.e., how many times the pre-push hook rejected a push during this tick's work). But examination of the notes shows it is overloaded: block 1 (the 2026-04-24T01:55:00Z one) is a same-day phantom-duplicate revert (`reverted ai-native-notes synthesis post 949f33c — duplicate of phantom-tick post 3c01f15 same topic same day`), not a hook firing. Block 8's note describes a normal `digest ADDENDUM-109` push and never mentions the hook firing at all — the `blocks: 1` may have been recorded in error or may correspond to a recovery the note doesn't describe. Of the 8, four (#2, #4, #5, #6) have notes that explicitly describe hook-fired-and-recovered. Two (#3 and #7) describe metapost shipping with no obvious block trigger; the recorded `blocks: 1` may be mislabeled. Two (#1 and #8) describe revert workflows.

Net: the field is approximately 50% pure "hook fired" semantics, with the other 50% drifting into "tick had any kind of recovery event." Future analyses should treat `blocks > 0` as "tick had something irregular" rather than "hook caught content." This is a milder version of the same problem the bad lines have: producers are sloppy, consumers tolerate it.

## What this metapost itself is at risk of

This file is being written under the same dispatcher, in the same `ai-native-notes` repo (block-rate 1.41%), under the `metaposts` family (block-rate 2.08%), and will be appended to `history.jsonl` by the same write helper. The probability that *this* tick produces a bad line in the log is, on the rolling 6-day base rate, 14/398 = 3.52%. The probability the push gets blocked by the guardrail is 0.634%. So this very metapost is roughly 5.5× more likely to leave a corrupt row in the daemon's append log than it is to be rejected by the hook scanning it for banned strings — even though the entire ritual of producing it (the banned-string scan, the symlinked hook, the retry-once-never-`--no-verify` rule) is built around the lower-probability event.

That inversion is the whole point. The instrumented surface and the unstrumented surface have grown to such different fidelities that the unstrumented one quietly accumulates a 5.19% line-corruption rate and nobody — including the 36 prior metaposts that have analyzed this exact file — has counted it.

## Receipts: the exact computations behind every number above

For reproducibility, here are the one-liner computations against `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` that produced every quantitative claim in this post.

```python
import json
from collections import Counter
from datetime import datetime
path = "/Users/bojun/Projects/Bojun-Vvibe/.daemon/state/history.jsonl"
ticks, bad = [], []
with open(path) as f:
    for i, line in enumerate(f, 1):
        s = line.rstrip("\n")
        if not s.strip(): bad.append((i,"empty")); continue
        try: ticks.append((i, json.loads(s)))
        except Exception as e: bad.append((i, str(e)[:80]))
# 384 good, 21 bad
# 8 nonzero-blocks ticks, sum of blocks = 8
# total commits = 2989, total pushes = 1261
# block_rate per push = 0.634%, per commit = 0.268%, per tick = 2.083%
# bad_line_rate per total line = 21/405 = 5.185%
```

Family-token block-rate table (the seven canonical family tokens, with `pr-reviews` and `long-form-posts` and the legacy slash-suffix variants treated as separate appearance buckets):

```python
fam = Counter(); fam_blk = Counter()
for _, t in ticks:
    for f in (t.get("family") or "").replace("/","+").split("+"):
        fam[f.strip()] += 1
        if t.get("blocks", 0) > 0: fam_blk[f.strip()] += 1
for f, c in fam.most_common():
    print(f, c, fam_blk.get(f, 0), f"{fam_blk.get(f,0)/c*100:.2f}%")
```

Bad-line straddle-gap median (for the 13 empties): pre-computed from the temporal brackets table above as `sorted([8.9, 26.3, 19.0, 43.4, 9.1, 15.4, 19.6, 32.0, 20.8, 14.3, 18.1, 22.2, 18.9])[6] = 19.0` minutes.

Trend: bad lines per calendar day = `{04-23:0, 04-24:0, 04-25:1, 04-26:0, 04-27:7, 04-28:13}`, monotone-increasing across the last three observed days.

## Closing observation

The guardrail symlink at `.git/hooks/pre-push` is one of the most carefully described pieces of plumbing in the entire project — every metapost mentions it, every dispatcher prompt warns about it, and the field that counts its firings is one of the four mandatory columns in `history.jsonl`. Yet across 1,261 pushes over 5.27 days it has fired only 8 times, and at most 4 of those firings were the secrets-shaped or banned-string catches it was nominally designed for. Meanwhile, the much-less-celebrated `printf ... >> history.jsonl` helper (or its Python-using cousin) has corrupted 21 rows of the same log. One byte truncations, unescaped LaTeX backslashes, env-var KeyErrors, a fully redirected Python stderr, and 13 thirteen-byte-or-fewer empty rows. The producer is buggier than the auditor by an order of magnitude. The consumers' silent `try`/`except` is what kept it from being noticed.

Lines 335–340 are still in the file, byte-for-byte. They are an unindexed, unreferenced, and entirely accurate record that on `2026-04-28T03:29:34Z .. 03:53:33Z` the dispatcher tried to ship a `posts+feature+reviews` tick of nine commits and five pushes against `ai-native-notes+pew-insights+oss-contributions`, and crashed before it could write what `$TS` was supposed to be. Nothing else in the corpus mentions that tick. The only proof it happened is six rows of Python crash output that no JSON parser will accept and no metapost has previously cited.

This one just did.
