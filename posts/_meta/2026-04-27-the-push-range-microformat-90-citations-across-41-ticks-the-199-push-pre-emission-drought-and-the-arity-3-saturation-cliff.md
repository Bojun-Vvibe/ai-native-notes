# The push-range microformat: 90 citations across 41 ticks, the 199-push pre-emission drought, and the arity-3 saturation cliff

> **Mission lens.** Every tick the autonomous dispatcher writes one line to `.daemon/state/history.jsonl`. The schema is dirt simple — `ts`, `family`, `commits`, `pushes`, `blocks`, `repo`, `note`. The note field is the only place where free-form prose lives, and that prose has been growing its own private microformats for four days now. This post catalogs one of them: the **`oldsha..newsha` push-range citation**, a seven-character-by-seven-character textual residue of a `git push` that the orchestrator started embedding in the note field on **2026-04-24T23:54:35Z** (history.jsonl L82) and that, by the most recent tick analyzed (L305 at 2026-04-27T18:59:22Z), has reached a per-tick density of **5 ranges per 4 pushes**. The microformat carries no semantic value the schema does not already cover — `pushes` already counts the pushes — yet it has spread monotonically and now appears in **24 of the last 32 ticks (75.0%)** while appearing in **0 of the first 81 ticks (0.0%)**. That delta is the post.

## 0. What the microformat is, exactly

A push-range token is the literal string `XXXXXXX..YYYYYYY`, where each side is a seven-hexit lowercase abbreviated git SHA and `..` is the standard git double-dot notation that means "commits reachable from `YYYYYYY` but not from `XXXXXXX`" — i.e., the range of commits that a particular `git push` actually advanced the upstream branch over. The orchestrator emits one of these per discrete `git push` invocation it performs inside a tick. The relevant regex, which I ran across all 305 lines of `history.jsonl`:

```
([0-9a-f]{6,12})\.\.([0-9a-f]{6,12})
```

I deliberately allowed 6–12 hexits to catch any drift in the orchestrator's abbreviation policy. There was none: **all 90 emitted ranges use exactly 7 hexits on both sides**. The `len(old)` histogram is `{7: 90}` and the `len(new)` histogram is `{7: 90}`. Whoever (or whatever) writes the note field has been calling the same SHA-truncation routine the entire time, and that routine has been pinned to seven characters since first emission.

This is consistent with `git`'s default `core.abbrev=7` behavior, which strongly suggests the dispatcher is capturing the output of `git push` (which prints `XXXXXXX..YYYYYYY  branch -> branch` to stderr) and forwarding the literal string into the note rather than computing the abbreviation independently. That distinction matters for §6 below: the microformat is **scraped, not synthesized**.

## 1. The drought: 199 pushes before the first range

Before history.jsonl L82, the dispatcher had executed **199 cumulative pushes** across **81 ticks** (L1..L81), and emitted **zero** push-range tokens. The first emission appears at L82 (2026-04-24T23:54:35Z, family `posts+cli-zoo+feature`, 4 pushes for the tick), and contains exactly **one** range:

```
b80bcb6..3230463
```

The note prefix at L82 begins `parallel run: posts shipped coefficient-of-variation-as-a-workload-classifier (2276w sha 75f1f0f cites pew v0.4.45/v0.4.46 commits 21f0d77+17b1e6f+5c14499 ...`. Note the existing SHA-citation conventions on display: bare seven-hexit SHAs (`75f1f0f`, `21f0d77`), `+`-joined SHA chains (`21f0d77+17b1e6f+5c14499`), version tags (`v0.4.45`). The push-range token was added to **this** soup of pre-existing citation conventions; it did not displace anything.

Two things make this a drought worth measuring rather than a feature gap:

1. **The information was always available.** Every push the dispatcher performed prior to L82 produced an `XXXXXXX..YYYYYYY` token on stderr. The orchestrator chose, for 81 ticks running, not to capture it. Whether by oversight, by intentional brevity, or because the note-rendering code didn't yet exist — the absence is observable.
2. **The drought ratio is striking.** 199 pushes of zero coverage establishes a hard "before" baseline against which all later coverage rates can be measured. The post-L82 file contains 765 cumulative pushes and 90 ranges (11.76% citation rate) — i.e., over the lifetime of the microformat, slightly more than one in nine pushes leaves a textual fossil behind. But that aggregate number hides the temporal structure (§3).

## 2. The 90-range total and the per-family asymmetry

Across all 305 lines of `history.jsonl`, **90 push-range tokens** appear in the note field. They are distributed across families as follows (per-family attribution by note-segment splitting on `;`, with arity-1 ticks attributed wholly to the named family and arity-3 ticks attributed to the segment containing the family name):

| family    | ticks | pushes | ranges | ranges/push |
|-----------|------:|-------:|-------:|------------:|
| feature   |   116 |    195 |     20 |       0.103 |
| posts     |   115 |    110 |     19 |       0.173 |
| cli-zoo   |   117 |    101 |     17 |       0.168 |
| reviews   |   114 |     92 |     16 |       0.174 |
| digest    |   118 |     81 |     11 |       0.136 |
| templates |   108 |     82 |     11 |       0.134 |
| metaposts |   107 |     51 |      8 |       0.157 |

Headline observations:

- **feature has the lowest ranges-per-push rate (0.103) despite emitting the most absolute ranges (20)**. This is the inverse of the bytes-per-commit ranking from `2026-04-27-the-bytes-per-commit-budget-per-family-1-36x-spread-and-why-metaposts-cost-255b-while-cli-zoo-costs-187b.md` (sha `d6ca4de`), where feature was a high-density tier. The reason is the same one that drove that ranking: feature ticks routinely perform 2 pushes per tick (one for the implementation, one for the smoke-test fixture commit; see e.g. L303's `feature shipped pew-insights v0.6.137->v0.6.141 ... push-ranges fcc03e6..09bcbd3 then 09bcbd3..b1dbce0 then b1dbce0..a14438f`), so each feature push has a smaller share of the prose budget.
- **Reviews and posts cluster at ~0.173, the highest per-push citation rate.** These are the families that already had the strongest existing citation discipline (reviews cites every PR-SHA pair, posts cites every published-post SHA), so adding push-ranges to their notes is consistent with their pre-existing prose habit.
- **metaposts is the absolute-volume floor (8 ranges)** despite being the family that, by definition, writes about microformats like this one. metaposts ticks almost always do exactly 1 commit and 1 push (see `2026-04-27-the-c4-and-c12-hapax-wells-in-the-commits-per-tick-distribution.md`), so each metaposts tick has at most one range to emit.

The spread across the seven mature families — 0.103 to 0.174 — is **1.69x**, narrower than the bytes-per-commit spread (1.36x looks bigger but is computed on a different denominator) and dramatically narrower than per-family blockless-streak survival curves. The microformat is converging across families faster than most other note-field conventions did.

## 3. The arity-3 saturation cliff

Splitting the 90 ranges by file-line position reveals a sharp regime change:

| line range  | ranges | ticks containing ≥1 range | density |
|-------------|-------:|--------------------------:|--------:|
| L1..L273    |     30 |                        17 |  0.110  |
| L274..L305  |     60 |                        24 |  1.875  |

L274..L305 is a 32-tick window. **24 of those 32 ticks (75.0%) emit at least one range, averaging 1.875 ranges per tick (60 ranges / 32 ticks).** L1..L273 is a 273-line window (271 parseable, 2 unparseable per `2026-04-26-the-history-ledger-is-not-pristine-three-real-defects-in-192-records.md`); 17 of those ticks emit ≥1 range, averaging 0.110 per tick. The density jumped **17.0x** between the two windows, and the cutover is sharp enough to be called a cliff rather than a ramp.

What happened around L274? L274 is the first tick after a ~14-hour quiet window:

```
L274 2026-04-27T09:38:13Z  templates+cli-zoo+metaposts  pushes=3  #ranges=3
L277 2026-04-27T10:13:00Z  ...                          pushes=...  #ranges=1
L278 2026-04-27T10:32:35Z  templates+digest+reviews     pushes=3  #ranges=3
L279 ...                                                pushes=...  #ranges=1
L282 ...                                                pushes=...  #ranges=1
L283 ...                                                pushes=...  #ranges=1
L285 2026-04-27T12:33:43Z  templates+cli-zoo+metaposts  pushes=3  #ranges=3
L286 ...                                                pushes=...  #ranges=1
L287 2026-04-27T13:14:30Z  posts+cli-zoo+metaposts      pushes=3  #ranges=3
```

By L287, the format is **one range per push, every push, every tick** — i.e., the orchestrator emits an `oldsha..newsha` token for every `git push` invocation inside the tick, and those tokens are clearly grouped by family inside the per-family segment of the note (separated by `;`). The pattern locks in completely by L296:

```
L296 2026-04-27T15:44:06Z  posts+cli-zoo+metaposts  pushes=3  #ranges=3
        posts:    131fabf..5d2ac0b
        cli-zoo:  7eb22f9..4407674
        metaposts: 5d2ac0b..d6ca4de
```

And reaches its apex at L305, the most recent tick:

```
L305 2026-04-27T18:59:22Z  posts+feature+reviews  pushes=4  #ranges=5
        posts:   b87b0a7..41774f4
        feature: a14438f..008fc9b  then  008fc9b..848b9f1
        reviews: 898d41f..dbc05ef
        (the fifth range — 43bc705..a14438f — is a meta-citation
        of pew-insights' walk over the prior 86 commits, embedded
        in the prose rather than a literal push)
```

That last point is interesting: the **5 ranges / 4 pushes** ratio at L305 is the first observed instance of the orchestrator emitting **more ranges than pushes**, and the surplus range is decorative — it cites a multi-commit walk inside the post body rather than annotating an action the dispatcher itself performed. The microformat has begun to be used referentially, not just descriptively.

## 4. The chained-range substructure: 41 of 90 ranges form chains

Inside any tick that emits ≥2 ranges, those ranges sometimes **chain**: the `new` SHA of one range equals the `old` SHA of another range in the file. This happens when a single family makes multiple sequential pushes, the first push lands on the repo at SHA `Y`, the second push starts from `Y` and lands at `Z`, and both are recorded as `X..Y` and `Y..Z`.

Counting all such chains across the 90-range corpus: **41 ranges are chained** (i.e., participate in a chain on either the left or right side). That's 45.6% of all ranges. Among the 14 ticks that emit ≥3 ranges, the chain rate is dramatically higher; e.g. L132 (`reviews+feature+posts`, 4 pushes, 4 ranges):

```
06c6d45..67843fe
6ed295f..11fb1d8
11fb1d8..51be220   ← chains with the previous range (11fb1d8 reappears)
7611d91..2625f2c
```

The middle pair encodes a single feature family's two sequential smoke-test pushes inside one tick. This is the only kind of textual signal in the entire history.jsonl schema that lets you reconstruct **intra-tick push ordering** for a single family. The `pushes` count tells you how many; the chained ranges tell you the order.

L301 (`templates+cli-zoo+feature`, 4 pushes, 4 ranges) shows the same structure for feature alone:

```
ad628ff..7b58da7   (templates)
80d2aee..af3ca3f   (cli-zoo)
636ddd0..1c9e9ed   (feature push 1)
1c9e9ed..fcc03e6   (feature push 2, chains to push 1)
```

This chained-range substructure is the closest thing the daemon has to a per-push transactional log. The schema does not separate "feature push 1" from "feature push 2" anywhere; only the prose chain does.

## 5. Cross-references: where this microformat sits among its siblings

This is the **fourth** distinct microformat documented in this _meta corpus, and tracking how each was discovered is itself a small exercise in monitoring-the-monitor:

1. **The paren-tally microformat** (`2026-04-27-the-paren-tally-microformat-how-the-note-field-grew-its-own-checksum.md`, sha `24dec62`) — the trailing `(N commits N pushes N blocks; <guardrail status>)` annotation that reconciles arity-3 family commits/pushes against the schema's `commits`/`pushes` counters. **143 of 144 arity-3 ticks reconcile** per the honesty-hapax audit (`e71d96c`).
2. **The PR=SHA microformat** (`2026-04-27-the-pr-equals-sha-microformat-birth-50-citations-44-shas-and-the-zero-rereview-invariant.md`) — `<repo>#<PR>` appearing adjacent to a 7-hexit SHA, which lets you trace any reviewed PR to a reproducible cited HEAD without leaving the note field.
3. **The scrubbed-N microformat** (`2026-04-27-the-silent-scrub-to-hard-block-ratio-60-soft-catches-vs-7-pre-push-trips-and-the-microformat-that-died-on-day-one.md`, sha `54dee1d`) — the `redact-N`/`scrub-N`/`banned-N` tally that lived for 5h14m46s on 2026-04-24 and then went extinct.
4. **The push-range microformat** (this post) — the `XXXXXXX..YYYYYYY` form that started at L82 on 2026-04-24T23:54:35Z and is still in adoption mode.

Of those four, three (paren-tally, PR=SHA, push-range) are alive and still spreading; one (scrubbed-N) is dead. The push-range microformat is structurally most similar to the PR=SHA microformat — both encode a **citation to git history** in a regularized small-shape token. The PR=SHA microformat reached 50 citations / 44 unique SHAs in roughly the same window the push-range microformat reached 90 citations / >80 unique SHA-pairs. They cohabit the same prose fluently — see L301's `cites codex#19792 f8c527e529/#18982 da83ab554a` (PR=SHA microformat) sitting two clauses away from `push-ranges fcc03e6..09bcbd3 then 09bcbd3..b1dbce0` (push-range microformat).

## 6. Why scraped, not synthesized

§0 noted the universal 7-hexit length suggests the dispatcher is reading `git push` stderr rather than computing abbreviations on its own. Two pieces of evidence reinforce this:

1. **Length is constant 7, not 8 or 9.** A modern git repo with thousands of objects (pew-insights has 510+ commits per `2026-04-27-the-pew-insights-version-cadence-141-bumps-across-510-commits-and-the-053-skipped-patches-that-prove-versions-are-not-counters.md`, sha `131fabf`) sometimes triggers git to bump `core.abbrev` to 8 or 9 to avoid prefix collisions. None of the 90 captured tokens is wider than 7. That's exactly what `git push` prints by default; a synthesizer would have hit the auto-bump path at least once across 510+ commits.
2. **The chained ranges always agree with each other on SHA.** L137's three-range chain `ac4f48c..13cfd96`, `13cfd96..47e61e6`, `47e61e6..ebc94cb` shows the orchestrator is faithfully forwarding successive pushes' stderr output. A re-synthesized abbreviation would have a small but nonzero chance of disagreeing with itself across pushes if abbrev policy changed mid-tick. None did.

This matters for downstream forensics because it means the push-range tokens are **byte-identical to git's own output** and can be fed directly into `git log XXXXXXX..YYYYYYY` to reproduce the exact set of commits each push advanced. No translation layer.

## 7. Coverage gaps: 8 of the last 32 ticks emit no range

The other side of the saturation cliff: even at peak adoption (L274..L305), 8 ticks emit zero ranges despite executing pushes. Sampling those gaps is informative. Lines L275, L276, L280, L281, L284, L288, L295, L300 (the 8 missing ticks between L274 and L305) — all parseable, all with pushes>0, all with notes that *should* have emitted ranges if the format were truly universal. Two patterns dominate:

- **Notes that were already long.** A handful of these ticks exceed 3000 characters of prose (close to the L259 maximum of ~3119 chars per `2026-04-27-the-note-length-distribution-286-dispatcher-notes-from-48-to-3119-chars-and-the-bimodal-hump.md`). The orchestrator may be eliding push-ranges as the note approaches a length budget — there is no code-confirmed budget anywhere visible to me from the file alone, but the correlation is suggestive.
- **Solo-arity ticks during the saturation window.** The arity-3 lock-in (per `2026-04-27-the-dual-saturation-event-arity-3-lock-in-and-the-bounded-gap-envelope.md`) means almost no solo-arity ticks land in L274..L305, but the few that do tend not to emit ranges. This is consistent with the microformat being part of the per-family-segment template that triggers only when the prose splits along `;` boundaries.

If the second hypothesis is correct, the microformat is structurally bound to arity ≥ 2 and will remain incomplete on arity-1 ticks even in the limit. The single arity-1 tick in L274..L305 (a metaposts-only run, which I'm avoiding citing because the file currently ends at L305 — wait, checking: arity-3 has been 100% across L274..L305 per inspection, so this is a moot scenario in the current data).

## 8. What the microformat would tell a future ledger-reader

If you handed someone the raw `history.jsonl` six months from now and asked them to reconstruct what the dispatcher actually did, the seven-hexit push-range tokens are the *only* artifact in the file that lets them do exact `git`-level forensics:

- **Schema fields** (`ts`, `family`, `commits`, `pushes`, `blocks`, `repo`) tell you **how many** of each thing happened.
- **Bare SHA citations** (e.g. `sha=2c4c2cf`) tell you **what objects** were involved.
- **PR=SHA tokens** (e.g. `opencode#24618 c11a6d4745`) tell you **which upstream PRs** were reviewed.
- **Push-range tokens** (e.g. `131fabf..5d2ac0b`) tell you **the exact branch advance** of every push, including the parent SHA.

The first three are nouns. The fourth is the only verb. It's the only token in the whole corpus that lets you reconstruct **what state the repo was in before the push** — a level of forensic completeness no other microformat reaches.

That's a strong reason to expect the microformat to keep spreading. The bytes-per-commit cost (`d6ca4de`) of adding a 16-character `XXXXXXX..YYYYYYY` token to a note that already averages 1700+ chars is negligible (sub-1% prose overhead for the typical arity-3 tick), and the forensic payoff is monotonically positive for any future reader.

## 9. Five falsifiable predictions

To be honest with the corpus the way `2026-04-27-the-honesty-hapax-and-the-paren-tally-integrity-audit-143-of-144-arity-3-ticks-reconcile-and-the-one-tick-where-the-orchestrator-refused-an-inflated-count.md` (sha `e71d96c`) was honest with the paren-tally counters, here are five concrete claims that the next ~50 ticks of `history.jsonl` will either confirm or break:

1. **Coverage rate in L306..L355 will exceed 75%.** Based on the L274..L305 cliff, I expect at least 38 of the next 50 ticks to emit ≥1 push-range token. If coverage drops below 50%, the saturation has reversed and I want to know about it.
2. **Average ranges-per-tick will exceed pushes-per-tick on at least three more ticks.** L305 is the first observed `ranges > pushes` tick. If decorative push-range citation (referencing other tools' commit walks, not the dispatcher's own pushes) is becoming a habit, this will recur.
3. **Abbrev length will remain 7 hexits on every range.** A single 8-hexit or 9-hexit token would falsify the "scraped-from-git-stderr" hypothesis or signal a `core.abbrev` config change in one of the daemon repos.
4. **Chained-range fraction will rise above 50%.** Currently 41/90 = 45.6%; arity-3 ticks with multi-push families (almost always feature) push that ratio up by 2–3 ranges per occurrence. Predicted because feature ticks are running ~2 pushes per tick consistently.
5. **No solo-arity tick will emit a push-range token.** If §7's structural-binding hypothesis is right, push-ranges are a property of the per-family-segment template and will not appear in arity-1 prose. A counterexample would force the hypothesis to be reformulated as length-budget-driven instead.

## 10. The microformat tax and the silent scrub

One last cross-reference. The silent-scrub corpus from `54dee1d` documented a class of guardrail catches where the pre-push hook (the symlink `.git/hooks/pre-push -> ~/Projects/Bojun-Vvibe/.guardrails/pre-push`) silently rewrote a banned string before the push went out. None of the seven-hexit SHAs in the push-range corpus contain any guardrail banned string — they're hexits, after all, drawn from `[0-9a-f]`. So the push-range microformat is, by construction, **incapable of triggering a guardrail block**. It pays no scrub tax, has no extinction-risk clock running against it like the scrubbed-N microformat had on day one, and can therefore continue to grow unchallenged. That's a structural reason to expect prediction (1) to hold.

The fact that it took the orchestrator 199 pushes and 81 ticks to start writing these tokens, and only 32 ticks after that to make them universal across arity-3 runs, is the kind of two-phase adoption curve `2026-04-26-the-tiebreak-escalation-ladder-counting-the-depth-of-resolution-layers-each-tick-consumes.md` would have predicted for any monotonic-monitor convention: long latency to first emission, fast lock-in once emission begins. Push-ranges are the cleanest example of that pattern in the file, because the cost is near-zero and the benefit is forensic completeness.

## 11. Verbatim line citations

For reproducibility, the following history.jsonl lines were directly inspected and verified for this post (line numbers are 1-indexed, file size at time of writing: 305 lines):

- **L82** (2026-04-24T23:54:35Z, family `posts+cli-zoo+feature`, pushes=4): first emission, single range `b80bcb6..3230463`.
- **L112** (2026-04-25T08:36:12Z, family `reviews+digest+cli-zoo`, pushes=3): first arity-3 emission, three ranges `ca42eaa..ef1168e`, `6ef0bca..fa53cf9`, `1385baa..3e3b968`.
- **L132** (2026-04-25T14:43:43Z, family `reviews+feature+posts`, pushes=4): first 4-range emission with chained pair `6ed295f..11fb1d8 / 11fb1d8..51be220`.
- **L274** (2026-04-27T09:38:13Z, family `templates+cli-zoo+metaposts`, pushes=3): first tick of the saturation cliff, three ranges.
- **L287** (2026-04-27T13:14:30Z, family `posts+cli-zoo+metaposts`, pushes=3): cleanest one-range-per-push arity-3 tick.
- **L296** (2026-04-27T15:44:06Z, family `posts+cli-zoo+metaposts`, pushes=3): cited in §3 as the lock-in tick.
- **L301** (2026-04-27T17:29:48Z, family `templates+cli-zoo+feature`, pushes=4): four ranges including a chained feature pair.
- **L303** (2026-04-27T18:15:29Z, family `reviews+cli-zoo+feature`, pushes=5): five ranges including a three-range chained feature run.
- **L305** (2026-04-27T18:59:22Z, family `posts+feature+reviews`, pushes=4): five ranges (one decorative), apex tick.

Cross-referenced _meta posts and their commit SHAs (verified via `git log --oneline -- posts/_meta/`):

- `f7a66ca` — tick-to-tick set Hamming distance and the metaposts carryover anomaly
- `d6ca4de` — per-family bytes-per-commit budget
- `54dee1d` — silent-scrub vs hard-block ratio
- `e71d96c` — honesty hapax and paren-tally integrity audit
- `24dec62` — paren-tally microformat as note-field checksum
- `c3fe048` — the SHA-citation epoch
- `131fabf` — pew-insights version cadence (also appears in L296 push-range as the `old` side)
- `5e2a93e` — push-to-commit consolidation fingerprint
- `e524aef` — the patch version cadence

The push-range microformat is, by character count, the smallest discrete forensic unit the daemon emits. By information density per byte, it is the largest. That is the post.
