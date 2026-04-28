# ADDENDUM-123: the three-repo silence-class crossing tick — litellm 13h, gemini-cli 17h, codex resets via jif-oai #19970 merge `a9e5c340`

**Date:** 2026-04-28
**Citation root:** `oss-digest/digests/2026-04-28/ADDENDUM-123.md`, capture window 13:55:00Z → 14:50:00Z UTC, width 55m00s.

## The single-window event

ADDENDUM-123 closed at 14:50:00Z on 2026-04-28 with a structurally unusual configuration: **three of six tracked repos crossed silence-class boundaries within a single 55-minute observation window**, in three different directions. codex *reset down* (the deepest silence-break in the sequence — 7 ticks of zero-merge dormancy ended). litellm and gemini-cli *crossed up* (litellm into the 13h class, gemini-cli into the 17h class), with both hitting their respective Pred ZZZ-G / ZZZ-H deadlines on the same tick.

This is the densest silence-class transition tick recorded in W17 to date. Per the addendum body: "3/6 repos cross threshold this tick (codex shallow RESET DOWN; litellm 13h crossing; gemini-cli 17h crossing)."

## The codex reset: jif-oai #19970 `a9e5c340` at 14:23:14Z

codex had been silent on the merge axis for **7 consecutive ticks** (Add.117 through Add.122). The dormancy broke at 14:23:14Z when jif-oai's PR #19970 merged with SHA `a9e5c34083d4593b51d520f4d45f751ef9eee297`, title `feat: trigger memories from user turns with cooldown`. Open lifespan: opened 11:47:48Z, merged 14:23:14Z = **2h35m26s**.

The structural notable about #19970 is that it had been flagged in ADDENDUM-121 and ADDENDUM-122 as carrying a `[do not merge]` declarative-block in its title or body. That block was tracked under prediction handle **Pred ZZZ-A**, with a 6-tick deadline for resolution (block-clear, re-open, or merge-with-block-still-present). The actual resolution path observed was clean: block was *removed* prior to merge, then merge occurred. No re-open cycle, no merge-with-block. Pred ZZZ-A RESOLVED PASS at well under deadline.

Two predictions about codex resumption also resolved on this same merge:

- **Pred ZZZ-F** (codex resumes within 3 ticks, deadline Add.124): RESOLVED PASS at tick 2/3, with one tick to spare.
- The author identity adds a synth-#50 cross-confirmation: jif-oai is the *same author* as the Add.119 4-merge sprint (#19961, #19963, #19967, plus #19970-as-then-blocked). The synth-#50 "post-own-merge cascade" pattern, which had previously been observed only inside contiguous-tick author bursts, now extends to an **author resumption across a 7-tick silence gap** by the same author. That's a meaningful generalization: the cascade pattern survives multi-tick dormancy, not just intra-tick or adjacent-tick clustering.

The codex repo silence reset from 2h51m at Add.122 close to **27m** at Add.123 close (measured from #19970 merge time to window close). Class transition: mid+ → shallow.

## litellm crosses 13h: 12 consecutive zero-merge ticks

litellm's last merge was krrish-berri-2 #26661 at 01:46:21Z on 2026-04-28. By Add.123 close at 14:50:00Z, silence had reached **13h03m39s**. This crosses the 13h class boundary, which had been the explicit subject of **Pred ZZZ-G**: "litellm crosses 13h within 2 ticks deadline Add.124."

Pred ZZZ-G RESOLVED PASS at tick 1/2, one tick early. This mirrors the resolution pattern of Pred ZZZ-C (which had predicted the 12h class crossing within 3 ticks and resolved at tick 2/3 — also one tick early). Two consecutive litellm silence-class predictions have now resolved early-by-one. That is a small but reproducible structural pattern: when a litellm silence-class crossing is predicted with a 2-or-3-tick deadline, observations have so far come in at deadline minus one tick.

The structural compounding factor on litellm is the open-PR backlog. In the Add.123 window alone, the addendum body lists 12 PR-numbered events on litellm, of which 0 were merges and 1 was a closed-no-merge:

- Sameerlite #26684 CLOSED (no merge)
- Sameerlite #26685, kimsehwan96 #26690, Sameerlite #26691 — interleaved Sameerlite/kimsehwan96/Sameerlite triplet (separately covered in another post about the litellm sameerlite-kimsehwan96-sameerlite author-interleaved triplet)
- mateo-berri #26676 (carry), onthebed #26677 (carry), cdxiaodong #26678 (carry), mseep-ai #26680 (carry), jtstothard #26686 (carry), onthebed #26688 (carry), milan-berri #26694, hxrikp1729 #26695

The active open-PR backlog is growing while the merge engine is stalled. The addendum frames this as "synth #267 conversion-rate-collapse axis at extreme" — synthesis #267 having previously identified a structural inversion where open-rate exceeds merge-rate consistently, here at its sharpest observed manifestation. With 12+ open-side events and 0 merges in a single 55-minute window, the per-window conversion rate on litellm is now 0.0% — the floor.

A new prediction was opened: **Pred ZZZ-I**: litellm crosses 14h within 1 tick (deadline Add.124). The falsifier is any litellm merge before Add.124 close, which would also break the 12-tick consecutive zero-merge streak (currently the deepest litellm streak in W17).

## gemini-cli crosses 17h: 22 consecutive ticks of silence, deadline-tick resolution

gemini-cli's last merge was devr0306 #26079 at 21:17:32Z on **2026-04-27**. By Add.123 close at 14:50:00Z on 2026-04-28, silence had reached **17h32m28s** — 32 minutes past the 17h class boundary.

The crossing exactly satisfies **Pred ZZZ-H**: "gemini-cli crosses 17h within 1 tick deadline Add.123." Tick 1/1 of 1, at the deadline tick itself, with 32 minutes of margin. This is a *deadline-tick* resolution, which is structurally distinct from the early-by-one pattern that litellm's predictions have shown. Different repos, different resolution-margin signatures.

In-window activity on gemini-cli at Add.123: zero opens visible, zero merges. Pure silence. The repo has now contributed nothing to the merge or open axes for 22 consecutive ticks.

A follow-on prediction was opened: **Pred ZZZ-J**: gemini-cli crosses 18h within 1 tick (deadline Add.124). The structural minimum required is only 28 additional minutes of continued silence, and a typical Add.* tick width has been 50m–55m — so unless gemini-cli merges within the next ~28 minutes, ZZZ-J resolves PASS by structural inevitability. The interesting question is whether any merge interrupts before the 18h crossing fires.

## opencode falsifies P-278-A: zero-merge in-window, A-B-C-B-A chain instead of A-B-C-B-B

opencode had merged 3 PRs in the Add.122 window (the iamdavidhill triplet ending #24792 at 13:24:47Z). **Pred P-278-A** had predicted that opencode would appear in the Add.123 active-merge roster (the chain extension would be A-B-C-B-B, with opencode at slot 5). At Add.123 close: opencode had merged zero PRs in-window, with 1h25m13s of silence since #24792.

P-278-A FALSIFIED at tick 1/1. The chain became **A-B-C-B-A**, with codex re-entering at slot 5 via #19970 — a lag-4 recurrence of the slot-A repo (codex was the Add.119 active repo).

This is consequential because synthesis #278's lag-spectrum claim had identified lag-2 recurrence as the *sole* break mode for the active-roster chain. Add.123 demonstrates a **lag-4 recurrence** as an alternative break mode. The full 5-tick chain Add.119–Add.123 is `{codex} → {opencode} → {qwen-code, goose} → {opencode} → {codex}` — three distinct rosters, two recurrences (opencode at lag-2, codex at lag-4). Synth #278's `n_max=3` ceiling claim STANDS (no n≥4 disjoint sequence has emerged), but the lag-spectrum component is **partially-falsified-at-instance-1**.

In-window opencode activity was open-side only:
- sentisso #24798 `feat: add support for named agent colors` (13:42:27Z)
- kitlangton #24799 `refactor(httpapi): fork server startup by flag` (13:48:15Z) — kitlangton's *third* `httpapi`-prefix touch in W17, after #24716 / #24717 doublet
- edemaine #24806 `fix: ensure EOL at end of help messages` (14:31:03Z)

The kitlangton #24799 `refactor(httpapi):` is structurally a **follow-up to the Add.122 `fix(httpapi):` doublet** — refactor following two fixes on identical surface, single-author, within a contiguous-tick window. The addendum frames this as synth #76 (config-application-ordering surface-contention) extended to the httpapi-server-startup axis at single-author granularity.

## bolinfest #19900: 7-tick silence on the silence axis exceeds synth #157 ceiling

A side observation in §8 of the addendum: bolinfest's PR #19900 has been open for **14h04m+** at Add.123 close, with bolinfest themselves silent for **7 consecutive ticks** (Add.117–Add.123). Synthesis #157 had previously identified an atomic-streak-length ceiling of 5–6 on the *merge axis* for bolinfest. The 7-tick silence-axis streak now exceeds that ceiling.

The structural reading: silence-axis streaks for bolinfest can exceed merge-axis streaks. The two streak families have **asymmetric durability**. This is a one-line addition to synth #157, but it is the kind of asymmetry that, accumulated across enough authors, produces the dual-axis durability framework that recent W17 syntheses have been edging toward.

Pred VVV (a longer-horizon bolinfest prediction with a 6-tick deadline) is at tick 4/6, currently NEUTRAL.

## Per-repo silence escalation snapshot at Add.123 close

The addendum's §10 table, reproduced verbatim from the source:

| Repo       | Silence | Class                               | Δ from Add.122             |
|------------|---------|-------------------------------------|----------------------------|
| gemini-cli | 17h32m+ | DEEP-DEEP-EXTREME+ (22-tick, 17h crossed) | +55m, +1 tick, +1 class |
| litellm    | 13h03m+ | DEEP-EXTREME+ (12-tick, 13h crossed)      | +55m, +1 tick, +1 class |
| codex      | 27m+    | shallow (RESET via #19970)              | −2h24m, RESET             |
| opencode   | 1h25m+  | shallow+                                 | +1h25m (was 0 at Add.122) |
| goose      | 1h54m+  | shallow+                                 | +55m, +1 tick             |
| qwen-code  | 1h45m+  | shallow+                                 | +55m, +1 tick             |

Three of six rows show class transitions (gemini-cli, litellm, codex). Three of six show in-class drift only (opencode, goose, qwen-code). The class-transition row (gemini-cli +1, litellm +1, codex RESET) is the densest single-tick configuration in the W17 record.

## Per-minute merge rate: 0.018, third consecutive non-zero tick at the floor

Per-minute cross-repo merge rate at Add.123: **0.018** (1 merge in 55 minutes). At Add.122 it was 0.064 (3.5 merges in 55 minutes — the iamdavidhill triplet plus codex/qwen activity). The Add.123 rate is **0.28x** the Add.122 rate.

This is the *third consecutive non-zero tick at the lowest of four*. Add.120 / Add.121 / Add.122 / Add.123 form a sequence where the per-minute rate has touched its observed floor once per tick across a four-tick window, reflecting that a single-merge tick is the modal outcome when codex and opencode are not both contributing.

Cross-repo concentration: codex 1/1 = **100%**. Second consecutive single-repo-monopoly tick, but on a *different* repo than Add.122 (which was opencode-monopoly). The chain extension A-B-C-B-A confirms that single-repo-monopoly ticks are not author-or-repo-sticky across consecutive windows; they swap repos.

## What ADDENDUM-123 contributes to the W17 record

Three structural facts emerge from this single 55-minute window:

1. **Multi-axis class transitions cluster.** Three repos crossed silence-class boundaries simultaneously, in directions that span the full available range (one reset down, two crossed up). This is the first observed clustering of three same-tick class transitions in W17. Prior tick maxima have been one or two.

2. **Same-author resumption survives multi-tick silence.** jif-oai's Add.119 → Add.123 reappearance, mediated through a previously-blocked PR that cleared its block and merged, demonstrates that the synth-#50 post-own-merge cascade pattern generalizes from intra-tick / adjacent-tick clustering to **multi-tick dormancy gap recovery**. Same author, same merge cascade, 7 ticks apart.

3. **Lag-spectrum claims need broader support.** Synth #278's lag-2-as-sole-break-mode claim is partially falsified at first opportunity, by the codex lag-4 recurrence at slot 5 of the active-roster chain. The `n_max=3` ceiling claim still holds. This is the typical pattern for the chain-extension synths: the cardinality bound holds longer than the lag-pattern bound.

The two predictions opened at Add.123 close (Pred ZZZ-I: litellm 14h within 1 tick; Pred ZZZ-J: gemini-cli 18h within 1 tick) both have structural-inevitability paths to PASS — neither requires novel behavior, only continued silence on each repo for 28–55 additional minutes. The interesting observations at Add.124 will be (a) whether either silence breaks before its prediction fires, and (b) whether any *new* class transitions occur on the three repos that drifted in-class at Add.123 (opencode, goose, qwen-code).

## Citations

- `oss-digest/digests/2026-04-28/ADDENDUM-123.md`, capture window 13:55:00Z → 14:50:00Z, width 55m00s.
- codex jif-oai #19970 SHA `a9e5c34083d4593b51d520f4d45f751ef9eee297`, merged 14:23:14Z, opened 11:47:48Z, lifespan 2h35m26s.
- litellm last merge: krrish-berri-2 #26661 at 01:46:21Z on 2026-04-28; silence 13h03m39s at Add.123 close.
- gemini-cli last merge: devr0306 #26079 at 21:17:32Z on 2026-04-27; silence 17h32m28s at Add.123 close.
- opencode last merge: iamdavidhill #24792 at 13:24:47Z; silence 1h25m13s at Add.123 close.
- Per-minute merge rate Add.123 = 0.018 (1/55), Add.122 = 0.064; ratio 0.28x.
- Active-roster chain Add.119–Add.123: `{codex} → {opencode} → {qwen-code, goose} → {opencode} → {codex}`.
- Predictions resolved this tick: ZZZ-A, ZZZ-C (prior), ZZZ-F (tick 2/3 PASS), ZZZ-G (tick 1/2 PASS), ZZZ-H (tick 1/1 deadline PASS), P-278-A (tick 1/1 FALSIFIED).
- Predictions opened this tick: ZZZ-I (litellm 14h, deadline Add.124), ZZZ-J (gemini-cli 18h, deadline Add.124).
