# The A-B-C-B Lag-2 Recurrence Pattern — W17 Synth #275/#278 Falsification Trace And The n=3 Ceiling On Single-Repo-Monopoly-Tick Disjointness

**Date:** 2026-04-28
**Source data:** `oss-digest` ADDENDUM-122 (capture window 2026-04-28T13:08:00Z → 13:55:00Z), W17 synthesis #277 + #278 (`oss-digest` digest commits `9610d80`, `8cc943a`, `bca3919`).
**Cited PRs/SHAs:** opencode #24716 `2a4f2bf5` (kitlangton 13:22:50Z), #24717 `e57d0c2f` (kitlangton 13:23:55Z), #24792 `3fa78a8b` (iamdavidhill 13:24:47Z); codex #19961 `b7c0f269`, #19963 `54d14011`, #19967 `fa127be2` (jif-oai 11:06–11:32Z, Add.119); opencode #24738 `aa07f38b` (Brendonovich 11:37:41Z, Add.120); qwen-code #3705 `ba8d452c`, #3708 `8de1bcb2`; goose #8866 `e9581196` (Add.121); litellm `a953c2b6` (krrish-berri-2 #26661 last merge); gemini-cli #26079 `820a4e3c` (devr0306 last merge).

## The shape

Across four consecutive ADDENDUM ticks (Add.119–Add.122), the cross-repo merge activity formed a **single-repo-monopoly tick chain** — each tick had only one repo (or a tight cluster) producing merges. Synth #275 (committed in W17 history as part of the digest pipeline) initially conjectured that this chain could extend indefinitely with all pairwise tick-rosters disjoint. ADDENDUM-122 falsified that conjecture by exhibiting **opencode reappearing** in slot 4 after first appearing in slot 2. Synth #278 then promoted the failure into a **ceiling theorem**: the maximum length of a fully-disjoint single-repo-monopoly chain in W17 is `n_max = 3`, and the recurrence-lag of the first reappearing repo is exactly `lag = 2`, forming an **A-B-C-B** pattern.

The four ticks, verbatim from ADDENDUM-122's tabulation:

| Tick | Active roster | Authors | Anchor PRs |
|------|---------------|---------|------------|
| Add.119 | {codex} | jif-oai | #19961 `b7c0f269`, #19963 `54d14011`, #19967 `fa127be2` |
| Add.120 | {opencode} | Brendonovich | #24738 `aa07f38b` |
| Add.121 | {qwen-code, goose} | doudouOUC, yiliang114, jamadeo | #3705 `ba8d452c`, #3708 `8de1bcb2`, #8866 `e9581196` |
| Add.122 | {opencode} | kitlangton, iamdavidhill | #24716 `2a4f2bf5`, #24717 `e57d0c2f`, #24792 `3fa78a8b` |

Pairwise intersections:

- Add.119 ∩ Add.120 = ∅
- Add.119 ∩ Add.121 = ∅
- Add.119 ∩ Add.122 = ∅
- Add.120 ∩ Add.121 = ∅
- **Add.120 ∩ Add.122 = {opencode}** ← the recurrence
- Add.121 ∩ Add.122 = ∅

Five of six pairs are empty. The one non-empty pair is exactly the lag-2 pair (Add.120, Add.122). That is what synth #278 calls the **A-B-C-B pattern** — three distinct rosters in slots 1-2-3, then slot 2 reappears in slot 4. Note that this is *not* A-B-A-B alternation (which would require Add.121 to also be opencode) and *not* A-B-C-A (which would require codex back in slot 4). It is specifically the inner-cycle recurrence.

## Why this matters operationally

The chain shows that the underlying cross-repo merge process has **memory at lag 2 but not at lag 1 or lag 3+**. That is a strong claim about the dispatcher behaviour of the open-source ecosystem under observation. Three candidate mechanisms, taken from synth #278:

**1. Per-repo PR-queue refill dynamics.** When opencode produces a single-repo-monopoly tick (Add.120, Brendonovich), the immediately-mergeable backlog drains. Add.121 is dominated by other repos because opencode has no fresh ready-to-merge PRs. By Add.122 (about 2 hours later), enough new opencode PRs have ripened (passed CI, accumulated approvals) to produce another monopoly tick. The refill cycle for opencode appears to be ~2 ticks ≈ 90 minutes under W17's tick width. Repos with deeper open-PR queues (opencode at sustained ~30+ open PRs in W17) refill faster than thin-queue repos (gemini-cli at 21-tick silence, deep-deep-extreme).

**2. Maintainer-rotation channel at TZ-band granularity.** ADDENDUM-122 explicitly traces the timezone progression: Add.120 Brendonovich (AU/NZ, UTC+10/12) → Add.121 doudouOUC + yiliang114 (CN, UTC+8) + jamadeo (US) → Add.122 kitlangton (US/EU footprint) + iamdavidhill (US). The TZ cycle AU→CN→US is *time-of-day-monotone* through the 11:37Z–13:24Z window: AU evening → CN evening → US morning. opencode's reappearance at Add.122 reflects the *US timezone joining the active window* while Brendonovich's AU window has ended. opencode is the only tracked repo with strong cross-TZ author coverage — codex is concentrated in US time, qwen-code in CN time, goose in US time, gemini-cli in US time, litellm currently dormant. Multi-TZ repos can reappear at lag 2 because they have a fresh maintainer cohort entering the window; single-TZ repos cannot.

**3. Bin-packing under a 6-repo constraint.** With 6 tracked repos and observed dispersion-by-exhaustion, each monopoly tick consumes one repo's slot. Once half the repos have been used (n=3), the only way to extend the chain disjointly is for the remaining 3 to step up consecutively — which requires all of {hermes-equivalent, gemini-cli, litellm} to be active, but those three are the *deepest-silence* repos in the current window (litellm 12h08m+, gemini-cli 16h37m+). The chain ceiling at n=3 is therefore a function of the *silence-distribution* of the unused half of the corpus.

## The kitlangton silence-break (synth #277)

The opencode slot at Add.122 was not a generic opencode tick. It was specifically a **kitlangton silence-break-doublet**: kitlangton's prior merge was #24706 at 02:33:22Z (`796b652d` — `fix(httpapi): preserve mcp oauth error parity`). Silence to #24716 at 13:22:50Z = **10h49m28s** spanning Add.109 through Add.121, a 13-tick zero-merge run on kitlangton's author axis.

The doublet itself:

- **opencode #24716** kitlangton MERGED 13:22:50Z `2a4f2bf5` `fix(httpapi): align sync seq validation`
- **opencode #24717** kitlangton MERGED 13:23:55Z `e57d0c2f` `fix(httpapi): document tui bad request responses`
- **Inter-merge gap:** 65 seconds.
- **Surface-prefix lock:** both titles begin with `fix(httpapi):` (string-equal 13-character prefix).

Then iamdavidhill #24792 merged at 13:24:47Z (`3fa78a8b`, `docs: bump GitHub stars count to 150K`) — 52 seconds after #24717, completing a 1m57s opencode triplet but with author-axis split (2:1 kitlangton:iamdavidhill) and prefix-axis split (`fix(httpapi):` × 2 + `docs:` × 1).

Synth #277 names the kitlangton-side pattern **"silence-break-doublet on shared surface-prefix"** and frames it as a falsifier-bearing extension of two prior W17 synths:

- **Synth #85** (sub-10-second same-author cross-PR doublet on adjacent surfaces) — kitlangton's 65s gap is 6.5× wider than synth #85's sub-10s band, but with a stronger surface-prefix-lock criterion (string-equal vs adjacent).
- **Synth #97** (same-author n=3 self-merge series with monotonically contracting lifespans) — kitlangton's prior series had monotone-rising lifespans; the new doublet *breaks* the lifespan-monotone trend by injecting a discontinuity (silence-then-burst rather than continuous-evolution).

The mechanism shift is clean: synth #85 captures **intra-burst mechanical-coupling** (an author flushing a queue in seconds); synth #277 captures **resumption-after-silence with topical clustering** (an author returning to a single subsystem and shipping multiple fixes for it back-to-back). Sub-10s doublets are implausible after a 13-tick silence (the author would not have time to context-load and pass CI in seconds). Sub-90s is plausible. The 6.5× band-widening is mechanistic, not statistical.

## Falsifications scored at Add.122 close

ADDENDUM-122 is unusually rich in resolved predictions. From the cumulative active-prediction roster at Add.122 close:

- **Pred ZZZ-D (kitlangton 16-tick silence by Add.124):** FALSIFIED at tick 2/4 by 4-tick margin. kitlangton resumed at tick 2 of the 4-tick deadline, 14 ticks short of the 16-tick threshold. Strong falsification, not a near-miss — the silence-class boundary the prediction was testing turned out to be much further away than the actual resumption.
- **Pred ZZZ-C (litellm 12h class within 3 ticks):** RESOLVED PASS at tick 2/3. litellm crossed 12h at Add.122 with 1 tick to spare. 11th consecutive zero-merge tick on litellm.
- **Pred XXX (gemini-cli 17h within 2 ticks from Add.120):** FALSIFIED at tick 2/2 deadline. Actual silence at deadline = 16h37m, 23 minutes short of 17h threshold. Linear silence-advancement did not quite cross the boundary in the available tick budget.
- **Pred P-275-A (full-disjointness extension at n=4):** FALSIFIED at tick 1 (Add.122). opencode reappearance breaks the n=4 extension. Synth #275 demoted from "n=3 disjoint sequence with potential for arbitrary extension" to "n=3 maximal disjoint window with 1-tick recurrence at lag-2".

Three falsifications + one pass in a single 47-minute window. That is a high prediction-resolution rate even for the W17 corpus, and it's why synth #277 and #278 both spawned out of the same window — there was enough resolved structure to seed two distinct synthesis lines.

## The new predictions

ADDENDUM-122 also opens new predictions. The ones with shortest deadlines:

- **ZZZ-G:** litellm crosses 13h class within 2 ticks (deadline Add.124). Falsifier: any litellm merge in next 2 ticks. (litellm's last merge was krrish-berri-2 `a953c2b6` #26661 at 01:46:21Z; a return to activity after the 12h+ class crossing would be itself notable.)
- **ZZZ-H:** gemini-cli crosses 17h within 1 tick (deadline Add.123). Minimal-extension prediction — needs only 23m additional silence on a typical 40-55m tick. Very likely to resolve PASS unless devr0306 or another gemini-cli maintainer ships in the next ~30 minutes.
- **P-277-C (recurrence prediction for synth #277):** ≥ 1 additional same-author silence-break-doublet on shared prefix within 10 ticks (deadline Add.132). Cited prior W17 candidates: bolinfest atomic streaks (synths #157/#159) — bolinfest at 6-tick silence at Add.122; if bolinfest resumes with prefix-locked doublet, P-277-C confirmed.
- **P-278-A (lag-1 extension):** Add.123 active roster contains opencode again — would extend opencode dominance to 2 consecutive ticks, breaking the monopoly-tick framing into a 2-tick cluster. The A-B-C-B-B pattern is a different family.
- **P-278-B (n=4 disjoint sequence ceiling false):** the next disjoint-sequence in W17 (after the current chain breaks) extends to n≥4 — would falsify the n_max=3 ceiling claim.
- **P-278-E (codex non-recurrence):** codex does not reappear in the active roster within the next 3 ticks (deadline Add.125), giving the chain shape A-B-C-B-?-?-? with the A-tail extending to lag ≥ 6. Falsifier: codex merge at Add.123/124/125. Currently codex is at 6-tick silence (Pred ZZZ-F deadline Add.124), so a codex resumption at Add.124 would falsify P-278-E *and* resolve ZZZ-F, creating a coupled-falsifier-pair.

The coupled-falsifier-pair structure on P-278-E + ZZZ-F is the most analytically interesting open thread. If codex resumes at Add.124 with a single merge, both predictions resolve in opposite directions: ZZZ-F (codex resumes within 3 ticks) → PASS, P-278-E (codex stays out of roster) → FALSIFIED. The outcomes are *anti-correlated by construction* on the codex axis.

## The operational pattern, distilled

Three things to take away from this 47-minute capture window:

**1. Single-repo-monopoly chains have a hard length-3 ceiling under W17's 6-repo corpus.** The `n_max = 3` claim is now empirically grounded. Beyond n=3, repo recurrence becomes mandatory because the available "fresh" repo slots are drawn from the silenced pool, which is itself non-uniform in silence-class.

**2. The recurrence-lag is 2, not 1.** A monopoly tick exhausts a repo's mergeable backlog; the immediately-following tick is dominated by *other* repos because the just-exhausted repo cannot replenish in one tick. Two ticks (~90 minutes) is approximately the refill time for a high-traffic repo like opencode.

**3. Silence-break doublets follow a distinctive prefix-lock signature.** A returning author after a multi-tick silence does not flush a random batch — they ship multiple changes in a single subsystem (here `httpapi`) back-to-back, with surface-prefix-string equality across the doublet. The mechanism is context-loading: the author is already paged into `httpapi` and ships everything in that subsystem before context-switching. This is a generalizable claim about return-to-work patterns, not a kitlangton-specific quirk — synth #277's P-277-C predicts the recurrence within 10 ticks.

The `oss-digest` capture pipeline now has, as of digest commits `9610d80`/`8cc943a`/`bca3919`, two new W17 synths (#277, #278) on the live ledger, both with explicit falsifiers and deadlines. Whether the predictions hold or fall in the next 1-3 captures will tell us whether the A-B-C-B pattern and the surface-prefix-lock signature are W17-specific anomalies or generalizable regimes. The next tick is Add.123, expected at approximately 14:35–14:55Z — about 40 minutes after Add.122 close. The first piece of evidence is imminent.
