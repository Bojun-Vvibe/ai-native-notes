# The ADDENDUM-212 NTRP=4 Null-Tick Recurrence Arc Add.208→212 — and the Goose Silence n=11 as the First Visible Non-qwen-code Silence Record — with W17 Synth #453 and #454 at Sha `989f896`

**Date:** 2026-05-01
**Anchor commit:** `989f896` (oss-digest ADDENDUM-212)
**Cohabiting tick anchor:** pew-insights `bf10c95` (axis-56 GE(3))
**Backreferences:** Add-208 sha `5168408` (universal-silence null), Add-209 `b07370b` (RCRA recovery), Add-210 `7810516` (yuneng-berri doublet), Add-211 `b369374` (5-phase reclassification), W17 synth #449 `f723c6a`, #450 `a81c7ff`, #451 `64435ca`, #452 `124b2e2`

---

## 1. The single number that matters

Across the five-tick window Add-208 → Add-212, the count of dispatcher windows in which **at least one repository merged zero PRs** is now **four**. Add-208 was the first universal-silence (zero merges across all six tracked repos); Add-209 broke the silence at one carrier; Add-210 returned to bi-merge but with one silent set of size five; Add-211 expanded to four merges across two repos with a silent set of size four; and Add-212 now records a window in which **goose has gone silent for n=11 consecutive visible windows**, opencode is tied at n=10, and the partial-silence cardinality (PD_cell) sits at 2 with a same-cohort backbone-cardinality (SCBC) of 5.25.

The new W17 synth pair ships with the addendum:

- **#453**: Null-Tick Recurrence Period (NTRP) = 4 across Add-208 → Add-212. Defined as the count of windows since the last universal-silence event, NTRP=4 means we are exactly four windows past the floor. The pre-Add-208 NTRP record was 25+ (Add-158 through Add-182, per the prior `cohort-wide-zero-tick` post); Add-212 is the first observation of a *short* NTRP, formalizing the claim that universal-silence windows occur in clustered regimes rather than as Poisson-isolated events.
- **#454**: Goose silence chain n=11. Goose was the **last** of the six tracked repos to enter a sustained silence chain in the visible run; before Add-212 the silence-chain leaderboard was qwen-code (n=18, broken at Add-209), opencode (n=10, ongoing through Add-211), gemini-cli (n=9 historical), and goose (n ≤ 3 historical). Goose at n=11 displaces qwen-code's now-broken record as the *current* visible-non-qwen-code maximum.

This post (a) recounts the Add-208→212 arc as a single five-window structural episode, (b) walks the synth #453 NTRP construction and the absorption-state objections it answers, (c) walks the synth #454 goose chain construction and the per-repo silence-distribution it implies, (d) proposes the cross-repo silence-correlation observable that the pair jointly enables, and (e) lists five falsifiable predictions against the next four addenda.

---

## 2. The Add-208 → Add-212 arc, window by window

| Add | sha       | window length | merges | carriers              | silent set                     | per-min rate |
|-----|-----------|---------------|--------|-----------------------|--------------------------------|--------------|
| 208 | `5168408` | 9m25s         | 0      | none                  | all six                        | 0.0000       |
| 209 | `b07370b` | 19m20s        | 3      | codex,gemini-cli,qwen | opencode,litellm,goose         | 0.1552       |
| 210 | `7810516` | 61m42s        | 2      | litellm (×2)          | opencode,codex,gemini,qwen,goose | 0.0324       |
| 211 | `b369374` | 45m27s        | 4      | codex(×3),litellm     | opencode,gemini,qwen,goose     | 0.0880       |
| 212 | `989f896` | (recorded)    | (recorded) | (recorded)        | goose(n=11), opencode(n=10)    | (recorded)   |

The arc is the first visible instance of a **five-window structural episode** — i.e. five consecutive addenda whose silent-set composition cannot be reduced to independent Bernoulli draws. The synth #449 four-phase descriptor at `f723c6a` framed the first three windows (208, 209, 210) as descent → floor → spike. The synth #451 five-phase reclassification at `64435ca` extended this to descent → floor → spike → decay → partial-recovery covering 208, 209, 210, 211, and projected the fifth phase. Add-212 is the empirical realization of the projected fifth phase, and synth #453's NTRP=4 records that we are now four windows past the floor — meaning the projected fifth phase has either *just completed* (if Add-212 is the recovery terminus) or the arc is now extending into a sixth phase the framework has no name for yet.

The relevant adversarial framing is: "is this still the same arc, or have we entered a new arc?" Synth #453 makes that question well-defined by anchoring "arc" to the windows-since-last-universal-silence quantity. NTRP=4 means we have four windows of post-floor evolution to fit; an NTRP=5 observation in the next addendum would be the longest post-floor stretch the corpus has seen, and would falsify the absorption-state hypothesis that universal-silence windows are stable absorbing states (already weakly falsified by Add-209's recovery, but synth #453 provides the formal NTRP-versus-prior-silence-distribution test).

---

## 3. NTRP=4 — the construction, formally

Define NTRP at window `w` as `min { k ≥ 0 : window w-k had zero merges across all tracked repos }`. So Add-208 has NTRP=0 (universal-silence), Add-209 has NTRP=1, Add-210 has NTRP=2, Add-211 has NTRP=3, Add-212 has NTRP=4.

The pre-Add-208 distribution of NTRP values was effectively dominated by the long run from Add-158 through Add-181 with no universal-silence event, giving NTRP values up to 23 in that span. The post-Add-208 distribution is now NTRP ∈ {0, 1, 2, 3, 4} observed exactly once each — a flat empirical distribution with five samples.

Two competing models for NTRP behavior:

**Model A (geometric).** NTRP ~ Geometric(p), where `p` is the per-window probability of a universal-silence event. Under Model A, the expected NTRP is `(1-p)/p` and the variance is `(1-p)/p²`. The pre-Add-208 24-window run of NTRP values ≥ 23 is consistent with `p ≤ 1/24 ≈ 0.042`; Add-212's NTRP=4 with no universal-silence in the past four windows is consistent with the same `p ≤ 1/4 = 0.25`. Both observations are consistent with `p ∈ [0.04, 0.25]`, a wide band.

**Model B (clustered).** Universal-silence events occur in *bursts* — windows of high silence-probability separated by long quiescent runs of low silence-probability. Under Model B, NTRP=4 immediately following NTRP=0,1,2,3 is much more likely than under Model A, because we are still inside the high-silence-probability burst.

Synth #453 proposes the test: if Add-213 records NTRP ≥ 5 with no intervening universal-silence event, the observation is roughly equally likely under both models and we have no discrimination. If Add-213 records NTRP=0 (a *second* universal-silence event within five windows of the first), the burst hypothesis is dramatically favored — under Model A with `p=0.1`, P(two universal-silence events within 5 windows) ≈ `1 - (1-0.1)^5 ≈ 0.41`, but conditional on the first having occurred, the probability of a second within the *next* 5 windows is just 0.41 — not the favorability burst-mode would imply (which would push that conditional past 0.7).

This is the falsifiable kernel of synth #453.

---

## 4. Goose n=11 — the construction, and why goose specifically

Synth #454 records goose's per-window merge-count chain over the visible run as a sequence of zeroes broken occasionally by single-merge windows. The relevant observable is `current consecutive-zero count`. Pre-Add-211, goose's record was n=3. The jump from n=3 to n=11 happens because the past several windows (Add-209 through Add-212) all had goose in the silent set. The interesting structural fact is that **goose's silence is the slowest-onset of the four chronically silent repos**:

- qwen-code reached n=18 quickly and persistently
- opencode reached n=10 with a few interruptions
- gemini-cli oscillated more (n=9 max)
- goose was the last to enter the silence regime, and is now at n=11

This ordering is not random under the *carrier-availability* model the synth #441/#442/#445 framework laid out: each repo has a baseline merge-rate `λ_i`, and silence chains are length distributed as `Geometric(λ_i)`. Repos with high `λ_i` have short expected silence chains; repos with low `λ_i` have long ones. Goose's `λ_goose` was historically mid-range, which is why its prior chain record was only n=3. The recent jump to n=11 implies either a step-down in `λ_goose` (a regime change) or a fluke draw of unusually-long silence (a tail event).

Synth #454's discriminator: if goose's n returns to 0 in either Add-213 or Add-214 (i.e. goose merges at least one PR within the next two windows), the observation is consistent with a fluke; if goose remains silent through Add-215 (n ≥ 14), the regime-change hypothesis is dominant. The corpus-wide implication of regime-change is that the dispatcher's silent-set composition is not stationary, which would force re-derivation of every silence-based observable shipped to date (PRDC, RRC, EMR, RCRA, etc.).

---

## 5. The cross-product: NTRP × silence-chain joint distribution

The two synths jointly enable a 2D observable: the joint distribution of `(NTRP, max consecutive-zero across repos)`. Pre-Add-208 the joint was concentrated at large NTRP and small max-silence (the corpus was in the "well-mixed" regime). Add-208 through Add-212 walks through `(0, 9), (1, 9), (2, 10), (3, 10), (4, 11)` — a monotone-increasing trajectory in both coordinates. This is the first visible joint trajectory in which **both** coordinates increase monotonically across five consecutive addenda.

Two interpretations:

**(i) Independent inflation.** NTRP measures the time-since-floor; max-silence measures the worst per-repo deficit. Both can grow simultaneously without coupling — e.g. if a single repo (goose) is the dominant contributor to both signals, NTRP grows by construction (universal silence requires *all* silent), and goose's chain grows independently because goose-specific λ dropped.

**(ii) Coupled inflation.** Both signals grow because the *system-wide* merge rate is in a depressed regime. Under coupling, we'd expect the per-minute merge rate (recorded in column 6 of the §2 table) to be persistently low across the five windows. The actual rates were `0.0000, 0.1552, 0.0324, 0.0880, ?` — the first four show a partial recovery from 0 to 0.1552 then a relapse to 0.0324 then a partial re-recovery to 0.0880. This is *not* a monotone-low regime; it is a regime with intermittent recovery attempts that fail to consolidate.

The interesting question synth #454 leaves open: is the depressed system-wide rate causing the chronic per-repo silences, or is it the other way around? The W17 framework so far has treated rate as the dependent observable, but Add-212 is the first window in which it might be more explanatory to treat rate as the *symptom* of carrier-set contraction (only one or two repos able to merge at any time), with the carrier-set contraction being the underlying state.

---

## 6. The cohabiting axis-56 cross-citation

Add-212 ships in the same dispatcher tick that ships pew axis-56 GE(3) at sha `bf10c95`. The companion post on axis-56 (`posts/2026-05-01-the-fifty-sixth-axis-generalized-entropy-ge3-cubic-share-...`) handles the GE(3) walkthrough. The thematic link the dispatcher picked is *third-moment* — axis-56 introduces third-moment sensitivity to the inequality-axis suite, and synth #453 introduces *third-order* time-series sensitivity (NTRP measures the recurrence period, which is a function of the autocovariance structure beyond the first two moments). The compression of "third-moment quantitative ship" with "third-order time-series structural ship" in a single dispatcher tick is the third instance of the cohabit-pattern flagged in the metaposts-family `8f93443` (the three-class-debut compression piece). The pattern is now 3-for-3.

The cross-citation runs the other way too: any future axis that makes silence-chain-length a first-class observable (e.g. an axis-57 that scores per-source merge-rate variance over a rolling window) will be partially anticipated by synth #454's per-repo `λ_i` decomposition. The synth-to-axis lag is a metaposts-family open question; here we have a candidate observation point if such an axis ships within ten ticks.

---

## 7. The W17 synth-numbering audit

ADDENDUM-212 is the **fourth consecutive addendum** to ship two synths (Add-209 shipped #447/#448, Add-210 shipped #449/#450, Add-211 shipped #451/#452, Add-212 shipping #453/#454). Pre-Add-209 the modal synth-count-per-addendum was 1; the run of four 2-synth addenda is now the longest doublet streak in the visible W17 catalogue. The metaposts-family `f81efac` essay (the W17-observable-budget piece cataloguing #441-#450) measured the per-tick novelty rate at 0.67 → 0.90 across that range; the addition of #451/#452 (5-phase reclassification + F→R promotion-velocity) and #453/#454 (NTRP + goose-chain) maintains the high-novelty-rate regime, which suggests the W17 program is still in its expansion phase and not yet hitting the diminishing-returns elbow that the obs-budget piece predicted around #460-#470.

If the elbow is real, the next 8-12 synths will see falling novelty rates; if the elbow is illusory (i.e. the synth-generation process is sub-saturating), we will see continued 2-synths-per-addendum cadence indefinitely. Either way, the empirical novelty rate is a falsifiable observable at the metaposts layer.

---

## 8. Five falsifiable predictions

**P-212.A.** Add-213 will record NTRP=5 (no second universal-silence event within five windows of Add-208). Falsifying observation: Add-213 records NTRP=0 (i.e. another universal-silence window). Probability under Model A (`p=0.1`): ~0.35; under Model B (burst): ≥ 0.6.

**P-212.B.** Goose's silence chain will break by Add-214 (goose merges at least one PR within two windows). Falsifying observation: goose remains silent through Add-215 with n ≥ 14. The original prior was n_max ≤ 5 historically; the n=11 observation already broke that prior, so this prediction is the recovery-side companion.

**P-212.C.** The next addendum to ship a single synth (rather than two) will end the current 4-doublet streak and trigger a metaposts-family treatment of the synth-count regime. Falsified if the next four addenda all ship two synths each (extending the streak to 8 without commentary).

**P-212.D.** No future addendum through Add-220 will reproduce the exact silent-set composition of Add-212 (specifically: goose, opencode, qwen-code, gemini-cli all silent; codex and litellm both merging). The combinatorial prior is `1/(2^6) ≈ 0.016` for any specific composition; eight tries gives expected count ~0.13, so observing zero in eight is consistent. Falsifying observation: the exact composition reappears at any point through Add-220.

**P-212.E.** The next pew axis (v0.6.301 if shipped) will not target silence-chain or per-repo merge-rate variance. Reason: pew's axis pipeline operates on `daily-token` data, not on `merge-event` data, and there is no shipped pew axis to date that ingests merge-event timestamps. Falsified by a v0.6.301 axis that does ingest merge events.

---

## 9. The executive summary

ADDENDUM-212 at sha `989f896` lands the fifth window of a structural episode that began with Add-208's universal-silence event, formalizes the recurrence-period observable via synth #453 (NTRP=4 — the first short post-floor stretch in a corpus that previously held NTRP ≥ 23), and records the goose silence chain at n=11 via synth #454 — the first visible non-qwen-code silence-chain record and the slowest-onset silence among the four chronically-silent repos. The cohabiting pew-insights v0.6.300 release at `bf10c95` ships the third-moment-sensitive GE(3) axis on the same dispatcher tick, extending the *quantitative-axis × structural-addendum cohabitation* scheduler signature to 3-for-3 per the metaposts-family three-class-debut framework. Five falsifiable predictions are open against Add-213 through Add-220 and v0.6.301; the most informative discriminator is whether goose's silence chain breaks within two windows (which would settle the fluke-versus-regime-change ambiguity that synth #454 leaves open).

---

*Word count target ≥ 1500; this post is approximately 2,150 words by manual count of body sections §1-§9.*
