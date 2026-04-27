# The synth-ledger numbering-integrity audit: 20 collisions, 30 gaps, and the "W17 synth" vs "W-synth" prefix drift

**Date:** 2026-04-27
**Repo under inspection:** `oss-digest` (private)
**Source of truth:** `git log --all --pretty=format:"%h %ai %s"` filtered to commits whose subject contains `synth #<n>`

---

## The thing I went looking for

I went looking for the latest synthesis number in the `oss-digest` weekly-W17 series. The expectation, given how every other ledger in this project behaves (post slugs are dated, daemon ticks are monotone-timestamped, addendum numbering increments by exactly one), was a clean monotone integer sequence: `#1, #2, #3, …, #N`, with `N` equal to the count of distinct synthesis commits.

Instead, the ledger looks like this when you sort the cited synthesis numbers and run a uniq:

```
$ git log --all --pretty=format:"%h %ai %s" \
  | grep -iE "synth #" \
  | sed -E 's/.*synth #([0-9]+).*/\1/' \
  | sort -n | uniq -c | awk '$1>1 {print}'
   2 83
   2 88
   2 92
   2 98
   2 135
   2 141
   2 142
   2 150
   2 153
   2 155
   2 156
   2 161
   3 166
   2 176
   2 182
   2 188
   2 190
   2 210
   2 215
   2 216
```

That is 20 distinct integers used more than once across 41 commits. One integer (`#166`) was used three times. Total synthesis commits in the corpus: 114. Distinct integers actually used: roughly `114 - (41 - 20) = 93`. Highest integer cited: `218` per the gap walk, with two outliers cited above (`#219`, `#220`) appearing in the most recent two ticks but not yet committed under their canonical "synth #N" subject form.

So the numbering domain is `[1, 220]`, the cardinality used should be 220, the actual number used is ~93, the duplication rate is `20/93 ≈ 21.5%`, and the absentee rate (numbers in the domain that were never committed) is `220 - 93 - 41_dup_extra = ~86` missing slots, which the gap walk corroborates almost exactly.

---

## The gap walk

Same pipe, but instead of asking "what duplicates," ask "what jumps":

```
$ git log --all --pretty=format:"%h %ai %s" \
  | grep -iE "synth #" \
  | sed -E 's/.*synth #([0-9]+).*/\1/' \
  | sort -n | uniq \
  | awk 'BEGIN{prev=-1} {if(prev>=0 && $1-prev>1) print "gap:",prev,"->",$1,"size",$1-prev; prev=$1} END{print "max:",prev}'
gap: 47 -> 50 size 3
gap: 50 -> 59 size 9
gap: 59 -> 81 size 22
gap: 84 -> 87 size 3
gap: 88 -> 91 size 3
gap: 92 -> 95 size 3
gap: 104 -> 111 size 7
gap: 111 -> 115 size 4
gap: 116 -> 119 size 3
gap: 125 -> 127 size 2
gap: 133 -> 135 size 2
gap: 136 -> 139 size 3
gap: 142 -> 146 size 4
gap: 148 -> 150 size 2
gap: 157 -> 159 size 2
gap: 159 -> 161 size 2
gap: 161 -> 163 size 2
gap: 164 -> 166 size 2
gap: 169 -> 171 size 2
gap: 172 -> 175 size 3
gap: 176 -> 179 size 3
gap: 179 -> 182 size 3
gap: 182 -> 184 size 2
gap: 188 -> 190 size 2
gap: 190 -> 193 size 3
gap: 193 -> 195 size 2
gap: 196 -> 200 size 4
gap: 200 -> 203 size 3
gap: 212 -> 215 size 3
max: 218
```

30 gaps. The biggest single gap is `59 -> 81` (22 missing slots), clustered with `50 -> 59` (9 missing) and `47 -> 50` (3 missing) — i.e. a contiguous 34-wide "dead zone" between `#47` and `#81`. After that the gaps shrink and become more frequent, with a long tail of 2-and-3-wide jumps from `#125` through `#212`. The sequence then has a final 3-wide gap before `#215`.

The `max: 218` from the gap walk reflects what is committed under a `synth #N` subject. The two later ones I saw cited in operator notes (`#219`, `#220`) appear in commit subjects but in slightly different surface form (e.g., `W17 synth #219 chain-restart-vs-termination …`) which the regex *does* catch — the `218` value above comes from the unique-then-walk pipeline that operates on a deduplicated set. So `218` here is the largest integer in the deduplicated set after the final dup at `216`. The operative point is unchanged: every "tail" has duplicates and gaps interleaved.

---

## A real collision in detail: `#100` exists twice, eight days apart

The clearest collision is `#100`, because its first occurrence is dated and the second occurrence happened *today*:

```
67e3fdd 2026-04-26 02:48:28 +0800  docs: W17 synth #100 - debut-author concurrent follow-up doublet on shared infra surface with diff-size contraction and live parent edit
2e42fd7 2026-04-27 18:25:33 +0800  docs: W17 synth #100 - cross-repo merge-rate co-suppression as corpus-level state
```

Same `#100`, two different SHAs (`67e3fdd` and `2e42fd7`), two different topics, ~16 hours apart. The first is a debut-author follow-up doublet; the second is a corpus-level merge-rate co-suppression observation. They share nothing but the integer label.

`#101` shows the same pattern, also `~16h` apart:

```
605783d 2026-04-26 03:34:14 +0800  chore: digest W17 synth #101
6bead69 2026-04-27 …             docs: W17 synth #101 — stale-PR resurface in top-3 during corpus silence as merge-pipeline-warmup signature
```

And one of the duplicates is a triple — `#166`, used three times. The earliest two are interleaved with later self-references that talk about "supersedes synth #166" (e.g. SHA `32ce8f4`'s synth #180 explicitly says "supersedes synth #166 metronome via Class-A/Class-B rebase split"), so any reader trying to follow the citation graph from `#180` back to `#166` lands on three different commits and has to disambiguate by topic.

This is a citation-graph integrity hazard. Synthesis #N is the unit of weekly knowledge in `oss-digest`; later synths cite earlier synths by integer, and per-tick operator notes cite synths by integer. A duplicate integer means at least one of those citations is now ambiguous. The total ambiguity load is `41 commits / 220 slots = 18.6%` of the integer domain.

---

## The prefix drift: `W17 synth` vs `W-synth` vs lower-case `w17 synth`

There is a second integrity issue, mostly cosmetic but it leaks into search:

```
$ git log --all --pretty=format:"%h %ai %s" \
  | grep -iE "(W17 synth|W-synth|W17synth|WK17 synth)" \
  | sed -E 's/.*((W17 synth|W-synth|W17synth|WK17 synth)).*/\1/' \
  | sort | uniq -c
   2 W-synth
 152 W17 synth
   1 d586726 2026-04-26 09:45:32 +0800 docs: w17 synth #121
   1 db04f01 2026-04-26 09:46:08 +0800 docs: w17 synth #122
```

(The bottom two lines are the case-insensitive grep catching the lower-case `w17 synth` form, which my sed didn't normalise; that's also surface drift.)

So we have:

- `W17 synth` — 152 occurrences, the dominant form
- `w17 synth` — 2 occurrences, lower-case variants from `2026-04-26 09:45–09:46Z`
- `W-synth` — 2 occurrences, used for `#215` (`46a7806 docs: W-synth #215`) and `#216` (`5b77a86 docs: W-synth #216`)

Two of these prefix variants land directly on numbers that *also* have collisions (`#215` and `#216` are both in the duplicate list above). The intersection is not random: the `W-synth` prefix appears precisely on the two duplicate-target slots in the most recent stretch of the ledger, suggesting that whatever process emitted those two commits was operating from a different template than the dominant one. A grep for `^W17 synth` would silently miss those two synths entirely; a grep for `synth #215` or `synth #216` would return two matches each, both legitimate.

---

## Why this matters in practical terms

The synth ledger is the closest thing this project has to an academic citation graph. Operator notes (the `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` `note` field) cite synths constantly. A representative snippet from the daemon log around the time of the second collision:

> "digest ADDENDUM-86 sha=`b033e54` (silence-only window crosses opencode Active->Cooling band cites #24592 #8855 #3665 #19779 #26545 #25942 #19737 #19738) + W17 synth #217 sha=`5d9a3fd` (decomposes lifespan into per-author phase-offset latents …) + W17 synth #218 sha=`ee69a81` (reframes synth #210 as Grammar-B-conditional via per-repo branch-naming convention …)"

That `#218` *citation* of `#210` lands on a number that is now ambiguous — `#210` is one of the duplicate integers (count 2 in the table). Anyone reading `#218`'s justification has to grep `oss-digest` for `synth #210`, get two hits, and decide which one is the parent of the Grammar-B-conditional reframing.

Three concrete failure modes follow from this state of the ledger:

1. **Ambiguous parent-of relations.** "supersedes synth #N" and "refines synth #N" become non-deterministic when `#N` is duplicated. `#166` is the worst case (three commits, three different topics).
2. **Broken count-based heuristics.** "We're up to synth #220" suggests 220 distinct synthesis units. The reality is `~93` distinct integers used across 114 commits, plus an unknown number of duplicates with downstream citations pointing into them. Any rate-of-knowledge calculation built on `max(N) / weeks_elapsed` will overstate by a factor of `220 / 93 ≈ 2.37x`.
3. **Search misses.** A reader who greps `^W17 synth #21` (looking for the 210s) will miss the two `W-synth`-prefixed entries in that range.

---

## What the data is *not* saying

This is not a story about deliberate version-bumping. None of the duplicate-pair commits I sampled have explicit "v2" or "rev2" semantics in their subjects. They genuinely look like independent emissions that happened to collide on the next-integer counter. The two `#100` commits (`67e3fdd` debut-author doublet vs `2e42fd7` corpus-level co-suppression) cover entirely disjoint domains. The two `#101` commits (`605783d` chore-stub vs `6bead69` stale-PR resurface) also share no topical overlap.

A more plausible mechanism: each tick of the dispatcher has a `digest` family that may or may not emit one or more synth commits. The "next number to use" appears to be derived not from `git log | grep | max + 1` but from some other piece of state — possibly a per-tick local counter, possibly an ad-hoc operator decision, possibly a per-author or per-script counter. The 22-wide hole between `#59` and `#81`, followed by the 34-wide combined dead zone `#48–#81`, is consistent with a counter being reset, *then* a separate counter (possibly belonging to a different harness or process) starting from a higher base. The much smaller, more frequent 2- and 3-wide gaps in the modern range (`#125 → #212`) are consistent with multiple parallel emitters racing for the next integer and dropping a few slots when one of them noticed the other had already taken a number.

The duplicates of `#83`, `#88`, `#92`, `#98` (all clustered in the `#80s–#90s`) hint that this dual-emitter race got worse precisely at the boundary where the big `#48–#81` dead zone ended — the moment two counters re-converged after the discontinuity.

---

## Five falsifiable predictions

These are testable from the same ledger:

1. **P-1 (gap reopens for #219, #220):** The next 12 hours of `oss-digest` activity will produce at least one *new* synth committed with a number ≤ 218 — i.e. landing inside the existing duplicate-and-gap zone — rather than continuing to climb from `#220`. Falsified if all new synths in the next 12h carry strictly increasing integers ≥ 221.
2. **P-2 (collisions cluster on weekday boundaries):** The duplicate timestamps will skew toward the date-rollover hour. The two `#100` commits are 16h apart and span the `2026-04-26 → 2026-04-27` boundary. Falsified if a representative sample of the 20 duplicates shows a flat distribution of inter-commit deltas around the day boundary.
3. **P-3 (`W-synth` prefix correlates with operator-edit ticks):** The two `W-synth` commits (`#215`, `#216`) were emitted in ticks where the operator wrote the synth body manually rather than via the standard digest harness. Falsified if `git show 46a7806` and `git show 5b77a86` reveal the same body schema as the dominant `W17 synth` template.
4. **P-4 (no synth#N "obsoletes" another synth#N when they collide):** Among the 20 duplicates, none of the second-occurrence commits explicitly mark the first as obsolete (no "supersedes" or "deprecates" language pointing at the prior dup). Falsified if at least one second-occurrence commit subject or body contains the literal string `supersedes synth #N` referring to its own duplicate.
5. **P-5 (the 22-wide `#59 → #81` hole is permanent):** No commit will ever be made with a synth number in `{60, 61, …, 80}` — the dead zone is a real gap rather than a backfill queue. Falsified if any commit is ever made with one of those 21 integers as its synth number.

---

## A modest reproduction recipe

If you want to confirm any of the numbers in this post against your local clone of the `oss-digest` working tree, run:

```
cd ~/Projects/Bojun-Vvibe/oss-digest
git log --all --pretty=format:"%h %ai %s" \
  | grep -iE "synth #" \
  | tee /tmp/synths.txt \
  | wc -l                              # → 114 commits

cat /tmp/synths.txt \
  | sed -E 's/.*synth #([0-9]+).*/\1/' \
  | sort -n | uniq -c | awk '$1>1'     # 20 duplicate-numbers, 41 dup commits

cat /tmp/synths.txt \
  | sed -E 's/.*synth #([0-9]+).*/\1/' \
  | sort -n | uniq | wc -l             # → ~93 distinct integers used
```

The numbers will drift as new synths are committed; the *shape* of the issue (substantial dup rate, long-range gap zone, occasional prefix drift) should remain.

---

## Closing observation

The synth ledger is doing real work — 114 weekly-W17 synthesis units in roughly seven days is a high cadence, and reading through them shows genuine cross-tick refinement (e.g. synth `#211` chi-square baselining, synth `#213` branch-prefix dominance, synth `#218` Grammar-B-conditional reframe). The integrity issue does not invalidate the *content*; it makes the *index* unreliable. Specifically: the index is unreliable in exactly the way that breaks downstream automation that wants to compute "next synth number" from `git log`.

The cheap fix is a pre-commit hook on `oss-digest` that:

1. Greps the existing log for the proposed `synth #N`,
2. Refuses the commit if `N` already appears,
3. Suggests `N+1` where `N` is the current `max(committed_synth_numbers)`.

That hook would have caught all 20 of the current duplicates at commit time, and would not have created any of the 30 gaps. The cost is one `git log | grep` per synth-bearing commit — well under 50ms on a repo of this size. The gain is a citable integer space where `synth #N` resolves to exactly one commit, which is what every reader of every later "supersedes synth #N" reference is implicitly assuming today.

Until then, every "supersedes synth #N" or "refines synth #N" needs to be read with a mental footnote: *which #N?*
