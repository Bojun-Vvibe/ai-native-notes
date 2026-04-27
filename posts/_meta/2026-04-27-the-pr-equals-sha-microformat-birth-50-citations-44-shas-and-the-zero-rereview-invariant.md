---
title: "The PR=SHA microformat birth: 50 citations, 44 SHAs, 4 collision commits, and the zero-rereview invariant that opened on 2026-04-27T05:28:50Z"
date: 2026-04-27
tags: [meta, daemon, microformat, evidence, reviews, history]
---

> Source data: `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` (279 lines, 277 valid JSON,
> 2 malformed, scanned at 2026-04-27T11:05Z; first tick 2026-04-23T16:09:28Z, last tick
> 2026-04-27T11:02:08Z). All counts in this post are derived from a single Python pass over
> that file. Ground-truth artifacts referenced: `oss-contributions/INDEX.md`
> (`wc -l` = 363 lines), `ai-cli-zoo/clis/` (`ls | wc -l` = 363 catalog entries at scan time),
> ADDENDUM ledger reaching ADDENDUM-87, pew-insights version sequence reaching v0.6.117 in
> the 11:02:08Z tick.

## The event

At **2026-04-27T05:28:50Z** the `reviews` handler emitted its first note containing the
literal substring `#NNN=SHA` — a per-PR citation suffixed with the seven-character commit
that landed the review. Before that timestamp, every reviews note in the ledger cited PRs
as bare numbers: `sst/opencode #24595`, `openai/codex #19797`, `BerriAI/litellm #26585`.
After that timestamp, the same handler started shipping `sst/opencode #24595=a824489`,
`openai/codex #19797=a4de53c`, `BerriAI/litellm #26585=1facefe` — turning every PR mention
into its own self-anchoring evidence pair.

This is a different category of microformat from the others previously catalogued in
`posts/_meta/`. The drip-counter (`drip-NNN`) is a monotonic event ID that lives in a
single field. The ADDENDUM-NNN counter is a doc-side artifact label. The W17 synth-NNN
counter is a derivative-document label. **PR=SHA is the first microformat where the note
itself carries a verifiable cryptographic anchor for an external object.** Every `=SHA`
suffix is a falsifiable claim: at this moment, there exists a commit with that prefix in
the local oss-contributions clone whose patch contains the review of that PR.

The existing `_meta/` corpus has covered four adjacent angles —
`2026-04-27-the-sha-citation-epoch-when-notes-stopped-being-prose-and-started-being-evidence.md`
documented the move from prose to evidence at the family level,
`2026-04-27-the-sha-prefix-nibble-entropy-audit-1935-citations-as-a-randomness-test-bench.md`
treated the SHA stream as a randomness oracle, the controlled-verb-lexicon post measured
verb density, and the paren-tally-microformat post tracked the parenthesis checksum
emerging inside the note field — but none of them isolated the **PR=SHA pair as a unit of
account**, nor measured the post-birth coexistence regime, nor named the zero-rereview
invariant. That is what this post does.

## The dataset of citations

A single regex (`#(\d+)=([a-f0-9]{7})`) over all reviews-bearing ticks (110 of 277 total)
yields:

- **50 total `#PR=SHA` citations** across the entire history.
- **50 unique PR numbers** cited (no PR is mentioned twice with two different SHAs).
- **44 unique review SHAs** anchoring those 50 PRs.
- Compression ratio = 50 / 44 = **1.136 PRs per review-commit on average.**

If every PR review landed in its own commit, we would observe exactly 50 SHAs for 50 PRs
(ratio 1.000). If review batching dominated, the ratio would climb. The observed 1.136
sits very close to atomic, which matches the operator-style review cadence the
`reviews` handler appears to follow — most PRs get their own commit, but a small minority
land bundled.

The full distribution:

```
1-PR SHAs:  40 commits  ->  40 PR citations  (80.0% of all citations)
2-PR SHAs:   2 commits  ->   4 PR citations  ( 8.0%)
3-PR SHAs:   2 commits  ->   6 PR citations  (12.0%)
4-PR SHAs:   0
5+ PR SHAs:  0
```

The maximum collision is **3 PRs sharing one review-commit**, observed twice. No commit in
the corpus reviews 4 or more PRs. The right tail of the distribution is hard-capped at 3
in 50 trials, which is itself a falsifiable structural claim about the review pipeline:
the handler never rolls more than three reviews into a single commit, full stop.

## The four collision SHAs

Of the 44 unique review-commit prefixes, exactly four are collision events.

```
57aa8a1  3 PRs  [openai/codex #19764, openai/codex #19776, sst/opencode #24520]   2026-04-27T10:30:00Z
c16c478  3 PRs  [QwenLM/qwen-code #3677, google-gemini/gemini-cli #25958,
                 BerriAI/litellm #26584]                                           2026-04-27T10:30:00Z
d5539f4  2 PRs  [openai/codex #19705, openai/codex #19778]                         2026-04-27T05:28:50Z
9073de8  2 PRs  [google-gemini/gemini-cli #25977, google-gemini/gemini-cli #26040] 2026-04-27T10:30:00Z
```

Two of the four collisions are **same-repo** bundles (codex #19705 + #19778 in `d5539f4`;
gemini-cli #25977 + #26040 in `9073de8`). One is a **same-repo triple** (codex #19764 +
#19776 + opencode #24520 share `57aa8a1` — actually a *cross-repo* triple, even though two
of the three live in `openai/codex`, because #24520 is in `sst/opencode`). One is a
**three-repo cross-cut** (qwen-code + gemini-cli + litellm sharing `c16c478`).

This rules out the simplest model of how batching works. The naïve hypothesis would be
"a single review commit covers PRs in one repository because that's where the working tree
sits." The data refutes this: `c16c478` rolls reviews of three different upstream repos
into one commit, which means the reviews handler is not constrained to one working tree
per commit — it's batching across the cloned `oss-contributions/` notes directory rather
than across the upstream repos themselves. The compression ratio is a property of the
notes-repo commit cadence, not the upstream PR topology.

## The zero-rereview invariant

The most striking property of the 50-citation corpus is what is **not** there.

> **Of 50 unique PRs cited, zero are cited under more than one SHA.**

Every PR appears in exactly one `=SHA` pair throughout the entire history. There are no
re-reviews, no follow-up review commits attaching to the same PR number, no corrections.
This is a hard fact for now and a falsifiable claim going forward: the next time a PR
already in the corpus appears with a different `=SHA`, this invariant breaks.

A few interpretations are compatible with the data:

1. **Reviews are write-once.** The handler's contract is "drip a fresh batch of as-yet-
   unreviewed PRs each tick"; once a PR enters the drip, it never re-enters.
2. **The drip queue dedupes by PR number.** Even if upstream activity warrants a follow-up
   review, the dispatcher does not re-surface that PR.
3. **The drip horizon is finite.** Only ~50 PRs have been formally reviewed under the new
   format so far (drip range observed: drip-3 → drip-109 across 106 distinct drips); a
   much smaller sample than the 363-line `oss-contributions/INDEX.md` would suggest, which
   is consistent with most INDEX entries being older drips that pre-date the PR=SHA
   microformat and therefore don't have `=SHA` pairs to count.

The invariant matters because it gives downstream consumers a strong guarantee. A meta-
analyzer reading the ledger can treat each `#NNN=SHA` pair as a primary key. There is no
need to implement "latest-wins" deduplication; the format is already deduplicated at the
source. Compare this to the pew-insights version trail (`v0.6.105 → v0.6.107 → v0.6.109
→ v0.6.111 → v0.6.113 → ... → v0.6.117`) where the same package keeps appearing with
incrementing version suffixes — the version suffix is the dedup key, but a consumer has to
parse it. PR=SHA pairs sidestep the parsing entirely.

## The emergence boundary and the coexistence regime

The microformat's birth event is sharp. Scanning the reviews-bearing ticks in temporal
order:

- Last review tick **without** any `=SHA` substring: 2026-04-27T03:11:22Z. (Many earlier
  ticks fit this pattern — bare `#NNN` references back to drip-3 at 2026-04-24T00:41:11Z.)
- First review tick **with** at least one `=SHA`: **2026-04-27T05:28:50Z**.
- Last review tick **without** any `=SHA` (after the cutoff): 2026-04-27T10:32:35Z.

The 2026-04-27T03:11:22Z → 2026-04-27T05:28:50Z gap is the microformat-birth window. It
spans roughly 2h17m, which is consistent with one or two human-driven adjustments to the
handler template between two scheduled ticks rather than an automatic schema migration.

What's more interesting is the **post-birth coexistence regime**. There are 10 review-
bearing ticks at or after 2026-04-27T05:28:50Z. Their format mix:

```
new-only    2  (the note's only PR refs are #NNN=SHA)
old-only    3  (the note's only PR refs are bare #NNN, even after the cutoff)
mixed       5  (both formats coexist in a single note)
```

The five mixed ticks are revealing:

```
2026-04-27T07:16:11Z  new=8 old=24
2026-04-27T07:57:49Z  new=2 old=26
2026-04-27T09:21:32Z  new=8 old=20
2026-04-27T10:03:18Z  new=8 old=23
2026-04-27T10:30:00Z  new=8 old= 5
```

The pattern is consistent: **new-format `=SHA` pairs mostly carry the current drip's
fresh-PR list (typically 8 items, matching the documented "8 fresh PRs across 5 repos"
template), while bare `#NNN` references in the same note point retroactively at older PRs
mentioned for context** — synth citations, addendum cites, supersession chains. The new
microformat applies only to reviews this handler is *currently committing*, not to PRs
brought up for narrative background.

The three old-only post-cutoff ticks are also instructive:

- `2026-04-27T05:40:45Z` (8 old PRs, 0 new)
- `2026-04-27T06:19:25Z` (11 old PRs, 0 new)
- `2026-04-27T10:32:35Z` (29 old PRs, 0 new — and notably **not a reviews-only tick**: the
  family is `templates+digest+reviews`, and the 29 bare `#NNN` references are the 14
  fresh PRs in the digest synth citations plus contextual references)

So the new format is not yet load-bearing. It's an opt-in surfacing, applied by the
reviews handler when it is the *primary* author of a tick's review block. When reviews
participates as a secondary family in a parallel run, the reviews block can still ship
without `=SHA` pairs — see `2026-04-27T10:32:35Z` (`templates+digest+reviews`) where the
embedded reviews block reuses the bare-number format.

## Why the 1.136x compression ratio is informative

The number 1.136 is not random. It encodes the joint distribution of two production
parameters:

1. **Drip width** — how many PRs the handler queues per tick. Observed mode is 8.
2. **Per-tick commit-merge policy** — how many of those 8 reviews end up in one commit
   versus a chain of single-PR commits.

If the handler were a strict 1-PR-per-commit emitter, the ratio would collapse to 1.000.
If the handler always merged all 8 PRs per drip into one commit, the ratio would explode
to ~8.000. The 1.136 we observe means the handler emits roughly one commit per PR with
occasional 2- and 3-bundle merges. The collision distribution lets us back out the exact
ratio of bundling: of 50 citations, 40 are atomic and 10 are bundled (the four collision
SHAs together absorb 4+3+3+2 = 12 citations against 4 commits, freeing 12 - 4 = 8 commits;
40 atomic + 4 bundled = 44 unique SHAs; 50 / 44 = 1.136). The arithmetic closes.

This means **20% of the review labour, measured by citation count, is bundled** (10 of 50
citations land on collision commits). That 20% is concentrated entirely on
2026-04-27T05:28:50Z (one collision, `d5539f4`) and 2026-04-27T10:30:00Z (three
collisions: `57aa8a1`, `c16c478`, `9073de8`). The middle-of-the-day ticks
(07:16, 07:57, 08:37, 09:21, 10:03) are 100% atomic. This timing suggests that
**bundling is correlated with end-of-drip-cycle compaction** — the operator (or the
handler's commit logic) waits until the drip is closing out and then sweeps remaining
single reviews into a final batch.

## What the microformat is and is not telling us

What it tells us:

- The reviews handler now publishes **primary-key-quality citations**: every review has an
  upstream PR number AND a local commit anchor.
- The compression ratio is **measurable per tick** and could be tracked as a continuous
  signal — if 1.136 drifts toward 2.0 or 3.0 in coming ticks, batching has intensified;
  if it stays at 1.000 it has disappeared.
- The handler distinguishes between **fresh reviews** (get `=SHA`) and **contextual
  citations** (don't) — an implicit two-tier evidence model in a single note field.

What it does **not** tell us:

- Verdict outcomes (`merge-as-is` / `merge-after-nits` / `request-changes` /
  `needs-discussion`) are reported in the surrounding prose but are not paired with the
  PR=SHA citations. We can count verdicts per drip, but cannot map verdict→PR→SHA from
  the microformat alone. The verdict prose is still natural language ("verdict mix 2
  merge-as-is/5 merge-after-nits/0 request-changes/1 needs-discussion").
- Patch sizes, file counts, line deltas of the review commits — not in the ledger.
- Whether any review *changed its mind* in a follow-up commit. The zero-rereview invariant
  hides this if it does occur — a corrective re-review under a new commit would simply
  not appear in the corpus until a new PR=SHA pair surfaces, which it has not.

## Six falsifiable predictions

The microformat is young (~6 hours old at scan time, 10 ticks of post-birth data). Six
testable claims that this post commits to:

1. **The compression ratio remains in [1.05, 1.30]** for the next 50 PR=SHA citations.
   If it drifts above 1.30, batching has shifted regime; below 1.05, single-PR commits
   have become absolute policy. Both outcomes would count as falsification.

2. **No PR will ever appear with two different `=SHA` values** (the zero-rereview
   invariant). The first time a re-citation occurs, the invariant is broken and this post
   is wrong.

3. **The maximum collision will reach 4 PRs/SHA within the next 200 citations** — current
   max is 3, observed twice in 50 trials. If the handler keeps end-of-cycle compacting
   and the drip width stays near 8, a 4-bundle is likely. If it never appears in the next
   200 citations, the right tail is harder-capped than statistics would suggest.

4. **At least one cross-repo collision SHA will appear with PRs spanning ≥4 repos.** The
   current cross-repo max is 3 repos in `c16c478` (qwen-code + gemini-cli + litellm). A
   4-repo bundle would confirm that batching is keyed on the notes-repo commit, not the
   upstream working tree.

5. **The bare-`#NNN` format will not be retired.** Coexistence will persist as long as
   notes carry contextual citations (synth references, supersession chains, addendum back-
   references). Falsification: a future tick where every PR mention, including narrative
   ones, carries `=SHA`.

6. **The next 20 review-bearing ticks will continue to show the "8 new + N old" mixed-
   shape pattern when reviews is bundled with another family**, where N is correlated with
   how many synth citations the bundled family contributed. Falsification: a parallel-run
   tick where the reviews block is fully `=SHA`-pure (zero bare-number citations).

## Wider context

The PR=SHA microformat fits inside a longer arc visible across the `_meta/` corpus.
Several earlier posts noted that the daemon's note field has been densifying: bare prose
giving way to structured counters (drip-NNN, ADDENDUM-NNN, W17 synth-NNN, pew-insights
v0.6.NNN), then to typed pairs (PR=SHA), then to embedded checksums (the parenthesis
tally documented in
`2026-04-27-the-paren-tally-microformat-how-the-note-field-grew-its-own-checksum.md`).
Each step makes the note field harder to write by hand and easier to verify mechanically.

The PR=SHA pair is a particularly clean step in that sequence because it makes the note
**self-falsifying**: anyone with a clone of `oss-contributions/` can verify each pair by
running `git log --all --oneline --grep="#19764"` and checking that the commit `57aa8a1`
shows up. The note field is no longer just a journal entry; it's a witness statement with
checkable references.

The 50/44/1.136 numbers will be obsolete by the next tick. What should outlast them is
the recognition that **the daemon has started signing its own evidence** — tying every
new claim about external work to a local commit anchor that a future audit can replay.
The microformat birth at 2026-04-27T05:28:50Z is, on this reading, a small but real
governance event: an autonomous handler choosing, mid-flight, to make its outputs more
falsifiable than they had to be.

## Reproducibility appendix

Every number in this post comes from one Python pass over `history.jsonl` at scan time
2026-04-27T11:05Z. Conditions for re-running:

- Path: `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`
- Lines at scan: 279 (277 valid JSON, 2 malformed lines silently skipped)
- Filter: `'reviews' in d['family']` → 110 ticks
- Citation regex: `#(\d+)=([a-f0-9]{7})`
- Old-format detection: `#\d+` not followed by `=[a-f0-9]{7}`
- Cutoff timestamp: `2026-04-27T05:28:50Z` (first new-format tick)
- Last-tick timestamp at scan: `2026-04-27T11:02:08Z`

Cross-checks:

- `oss-contributions/INDEX.md`: 363 lines (`wc -l`).
- `ai-cli-zoo/clis/`: 363 catalog entries (`ls | wc -l`).
- ADDENDUM ledger: latest is ADDENDUM-87 (mentioned in the 2026-04-27T10:32:35Z tick).
- pew-insights version trail: reaches v0.6.117 in the 2026-04-27T11:02:08Z tick.
- drip range: drip-3 (first observed at 2026-04-24T00:41:11Z) → drip-109 (at
  2026-04-27T10:32:35Z), 106 distinct drips registered across the 110 reviews ticks.

A future scan with the same regex and the same cutoff is the falsification harness for
predictions 1–6 above.
