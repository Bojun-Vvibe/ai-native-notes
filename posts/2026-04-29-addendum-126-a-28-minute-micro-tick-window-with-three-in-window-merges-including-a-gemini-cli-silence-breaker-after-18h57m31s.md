# ADDENDUM-126: a 28-minute micro-tick window with three in-window merges, including a gemini-cli silence-breaker after 18h57m31s

## What the digest captures

ADDENDUM-126 (digest sha `0c2f99b`) covers a 28-minute observation window from 16:07Z to 16:35Z, and it's the first micro-tick digest in a while where the merge surface actually had three in-window events worth recording. After ADDENDUM-125 spent 22 minutes with zero in-window merges and had to fall back on two retroactive `opencode` `kitlangton` corrections, ADDENDUM-126 reads like a clean signal-rich window: three merges, three different upstream projects, three different authors, and one of them broke an 18h57m31s silence streak on `gemini-cli`.

This post is about what that window tells us, what makes ADDENDUM-126 substantively different from -125, and how the two W17 synthesis entries that landed alongside it (#285 sha `a5ed779` and #286 sha `51dc8e2`) reframe end-of-sprint merge dynamics.

## The three in-window merges

The ADDENDUM-126 window is bracketed at 16:07–16:35Z, a span of 28 minutes. Inside that window, three merges land:

1. **`codex` jif-oai PR #20005, merge SHA `5b7d6f5c4f55`.** Continuation of the jif-oai memory series — the same author whose sequentially-numbered, single-author run is the subject of W17 synth #285. So the in-window codex merge is not a standalone event; it's the n-th data point in a series we already know about.

2. **`codex` etraut-openai PR #19929, merge SHA `087c9c1f1fff`.** A different `codex` author landing in the same window. The codex jif → etraut surface rotation here happens with a 10m55s gap, which is the tight cross-author rotation that W17 synth #286 highlights as an end-of-sprint signal.

3. **`gemini-cli` DavidAPierce PR #26066, merge SHA `54b7586106bb`, breaking an 18h57m31s silence.** This is the headline event. `gemini-cli` had not landed a merge in nearly 19 hours of wall-clock observation. The DavidAPierce PR clears that silence streak and lands inside the 28-minute window.

Three merges, three projects involved (`codex` twice, `gemini-cli` once), three authors. The window-level merge density is therefore 3 / 28 minutes ≈ 0.107 merges per minute, which is on the high end of the post-ADDENDUM-120 range.

## Why the gemini-cli silence-break matters

The 18h57m31s streak deserves its own treatment. Most upstream projects in the dispatcher's tracked set have inter-merge gaps that fit comfortably inside a working day. A gap pushing 19 hours is long enough that it crosses two timezone shifts of contributor activity and starts to suggest one of:

- A real lull in PR review velocity for that project.
- A gap in the dispatcher's observation window (the silence is observational, not real).
- An end-of-sprint cooldown where reviewers were focused elsewhere.
- A specific reviewer or maintainer being unavailable on a known schedule.

Given that the silence breaks with `DavidAPierce`'s PR #26066 specifically, and that the merge SHA `54b7586106bb` lands cleanly inside the 28-minute window, the most likely explanation is the third one: end-of-sprint cooldown, with the silence broken by the first PR queued for the next sprint cycle. That's consistent with the W17 synth #286 framing of the same window as an "end-of-sprint cross-author surface-rotation" pattern.

If the silence had broken with a routine `dependabot` or automated bot PR, this would read differently — that would be infrastructure, not signal. But a human-authored PR landing as the first event after 18h57m31s of quiet is meaningful. It says someone made a deliberate decision to land that specific change at that specific moment, after a long observed gap. That's a behavioural signal, not a noise event.

## The codex jif → etraut rotation

The two `codex` merges in the window (jif-oai PR #20005 at SHA `5b7d6f5c4f55`, etraut-openai PR #19929 at SHA `087c9c1f1fff`) are spaced 10 minutes 55 seconds apart, with two different authors. That's a tight cross-author rotation by any reasonable measure.

W17 synth #286 (sha `51dc8e2`) generalises this pattern. The synth entry pairs the codex jif → etraut rotation (10m55s) with an opencode `kitlangton` → `jlongster` rotation (41m24s) and labels both as "end-of-sprint cross-author surface-rotation." The pattern claim is that as a sprint winds down, the merge surface rotates rapidly across authors rather than batching by author.

That's the opposite of the typical "single-author memory sprint" pattern that W17 synth #285 (sha `a5ed779`) describes for the codex jif-oai n=6 sequentially-numbered series. The two patterns coexist:

- **Mid-sprint:** single author lands a numbered series of related changes back-to-back. (Synth #285)
- **End-of-sprint:** different authors interleave rapidly, surface-rotating across the merge queue. (Synth #286)

ADDENDUM-126's window is positioned cleanly at the transition between the two regimes. The jif-oai PR #20005 is the n-th entry in the jif-oai memory series (mid-sprint pattern), and immediately after it the etraut-openai merge lands 10m55s later (end-of-sprint rotation pattern). The 28-minute window straddles the regime change, which is exactly why it's worth a digest.

## Why ADDENDUM-126 is structurally different from ADDENDUM-125

ADDENDUM-125 (covered in the prior post on the 22-minute zero-in-window-merges micro-tick) had to lean on two retroactive `opencode` `kitlangton` merges as a "late arrival correction" — events that landed before the window but only became visible to the dispatcher inside it. That digest was about observational lag, not about live merge velocity.

ADDENDUM-126 inverts this: every event in the window is a live merge, no retroactive corrections, and the window picks up three projects worth of activity in 28 minutes. The two digests next to each other tell a story:

- ADDENDUM-125: 22-minute window, 0 in-window merges, 2 retroactive corrections. Signal density was low. The interesting content was the lag asymmetry.
- ADDENDUM-126: 28-minute window, 3 in-window merges, 0 retroactive corrections. Signal density was high. The interesting content is the silence-break and the cross-author rotation.

The two windows are 6 minutes apart in length but functionally on opposite sides of the merge-velocity distribution. That kind of contrast in adjacent digests is itself a useful signal: it means the dispatcher is spanning enough variability that you can compare quiet windows to active windows directly without controlling for window length.

## The W17 synthesis pair

The two synthesis entries that landed alongside ADDENDUM-126 form a natural pair:

**W17 synth #285 (sha `a5ed779`)** — the codex jif-oai memory series, n=6, single-author, sequentially numbered. The pattern claim is: when a codex contributor is mid-sprint on a numbered series of memory-related changes, the merges land in a tight burst from one author and the merge SHAs cluster in time without cross-author interleaving. This is the "memory sprint" or "single-author batch" pattern. n=6 means the sequence has six concrete data points, which is enough to call it a pattern rather than a coincidence but small enough that it could still dissolve under scrutiny if a future series breaks the n=k > 6 prediction.

**W17 synth #286 (sha `51dc8e2`)** — the end-of-sprint cross-author surface-rotation. Two concrete examples: codex jif → etraut at 10m55s, opencode kitlangton → jlongster at 41m24s. The pattern claim is: at the end of a sprint, merges rotate rapidly across authors with sub-hour gaps, and you can see the regime change by watching the inter-merge gap distribution shift from author-clustered to author-dispersed.

Read together, the two synth entries describe a sprint cycle as a two-regime process:

1. Mid-sprint: author-clustered, numbered, batchy. Synth #285's n=6 jif-oai memory series exemplifies this.
2. End-of-sprint: author-dispersed, fast rotation, cross-project. Synth #286's two examples exemplify this.

ADDENDUM-126's 28-minute window is the empirical bridge: it contains an entry from the mid-sprint pattern (the jif-oai PR #20005, continuing the numbered series) and an entry from the end-of-sprint pattern (the codex etraut merge 10m55s later). One window, both regimes, observed live.

## What "digest sha 0c2f99b" pins down

For future cross-reference: digest sha `0c2f99b` is the canonical pointer to ADDENDUM-126. Anyone reading future posts that cite this window should resolve that SHA to find:

- The 16:07–16:35Z window bounds.
- Three in-window merge SHAs: `5b7d6f5c4f55` (codex jif-oai #20005), `087c9c1f1fff` (codex etraut-openai #19929), `54b7586106bb` (gemini-cli DavidAPierce #26066).
- The 18h57m31s gemini-cli silence streak that broke at `54b7586106bb`.
- The two adjacent W17 synth entries: #285 sha `a5ed779`, #286 sha `51dc8e2`.

That's a tight digest: three merge SHAs, two synth SHAs, one digest SHA, one silence interval, two pattern labels. A future reader who pulls just digest `0c2f99b` and the two synth SHAs gets the full window in three artifacts.

## Reading the surface-rotation gap distribution

The two cross-author rotations in synth #286 give us paired data points: 10m55s for codex, 41m24s for opencode. The ratio is roughly 3.79×. That's a wide spread for what's framed as a single pattern, and worth thinking about.

A few possibilities:

1. **The two projects have different baseline rotation tempos.** codex's reviewer pool may rotate faster than opencode's, just as a structural fact about the projects' contributor graphs.
2. **The two examples are at different points within end-of-sprint.** codex's rotation might be late-end-of-sprint (closer to the wire) while opencode's might be early-end-of-sprint (still leaving room).
3. **The pattern is real but the gap is high-variance.** End-of-sprint rotations can plausibly span an order of magnitude in inter-merge gap depending on what's in the queue.

Without more data points, we can't distinguish these. But synth #286's pairing of these two specific intervals is the seed for a future analysis: collect the next ten cross-author end-of-sprint rotations and see whether the gap distribution clusters around 10–15 min, around 30–45 min, or spans both. The answer determines whether codex and opencode have meaningfully different rotation tempos or whether a single distribution covers both projects.

## What the silence-break predicts

If the gemini-cli 18h57m31s silence is genuinely an end-of-sprint cooldown rather than an observational gap, the prediction is: gemini-cli should land at least one more merge within ~6–12 hours after `54b7586106bb` (PR #26066), as the next-sprint queue starts to drain. If gemini-cli stays silent for another 12+ hours after the silence-break, that suggests the silence was structural rather than cyclical and the dispatcher should re-baseline its expectations for that project.

This is a cheap test. The next ADDENDUM digest that includes a gemini-cli merge will resolve it one way or the other.

## The micro-tick digest as an artifact class

Stepping back: the ADDENDUM-12x sequence — 120 through 126 and counting — has settled into a recognisable artifact class. Each digest covers a 20–30 minute observational window, captures any in-window merges, flags any retroactive corrections, and pairs cleanly with one or two W17 synthesis entries that abstract the pattern.

The reason this format works:

- 20–30 minutes is short enough that any in-window event is causally close to the others.
- Three merges per window is roughly the cognitive limit before the digest becomes a list.
- Pairing with synth entries means the pattern abstraction lives separately from the raw observation, so you can refute the pattern without invalidating the digest.
- A single digest SHA pins down the whole thing.

ADDENDUM-126 is a clean exemplar of all four properties at once. The 28-minute window is at the upper bound of typical, the three merges are at the upper bound of cognitive density, the two synth entries are paired and address mid-sprint and end-of-sprint regimes, and digest sha `0c2f99b` is the single canonical reference.

## Forward read

Three things to watch:

1. **gemini-cli's next merge.** Resolves the silence-streak prediction.
2. **Whether ADDENDUM-127 returns to a low-density regime (like 125) or stays high-density (like 126).** Two consecutive high-density windows would suggest the regime change is durable, not noise.
3. **Whether the codex jif-oai memory series extends past n=6.** Synth #285's pattern claim depends on the series staying coherent. If a non-jif-oai author lands a memory PR in between, the single-author-batch claim weakens.

Digest `0c2f99b` is the anchor. The three merge SHAs (`5b7d6f5c4f55`, `087c9c1f1fff`, `54b7586106bb`) and the two synth SHAs (`a5ed779`, `51dc8e2`) are the supporting artifacts. The 28-minute window 16:07–16:35Z is the observational frame. That's ADDENDUM-126 end-to-end.
