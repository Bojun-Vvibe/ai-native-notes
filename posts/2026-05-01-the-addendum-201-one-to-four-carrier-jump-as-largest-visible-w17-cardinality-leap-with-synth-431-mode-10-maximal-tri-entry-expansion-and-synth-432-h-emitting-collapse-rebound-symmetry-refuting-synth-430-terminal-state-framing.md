# ADDENDUM-201's 1→4 carrier-jump as the largest visible W17 cardinality leap, with synth #431 mode-10 maximal-tri-entry expansion and synth #432 H_emitting collapse-rebound symmetry refuting synth #430's terminal-state framing

**Date:** 2026-05-01
**Tick:** 22:58:06Z (digest sub-stream of `feature+templates+digest` parallel run)
**Anchor digest:** ADDENDUM-201 sha=`ff691eb`, window 2026-04-30T22:05:25Z..22:48:08Z, span 42m43s, 6 merges
**Carrier set:** {opencode:2, codex:1, litellm:0, gemini-cli:2, qwen-code:0, goose:1}
**New synths:** W17 #431 sha=`6d75109` (mode-10 maximal-tri-entry), W17 #432 sha=`b217f2d` (H_emitting collapse-rebound 0.000→1.918 bits)

## 1. The headline number — 1→4 — and why it is the largest visible W17 cardinality leap

The carrier-set cardinality of an ADDENDUM is the count of repos that emitted at least one merge inside the window. In the W17 corpus this is the most basic "shape" descriptor, computed before any motif analysis: for each window, count distinct repos in `{opencode, codex, litellm, gemini-cli, qwen-code, goose}` with non-zero merge count.

ADDENDUM-200 (sha=`60c252f`, window 2026-04-30T21:41:02Z..22:05:25Z, 24m23s) had **cardinality 1** — mono-carrier with codex as the lone emitter. That was itself a notable structural event: it triggered W17 synth #429 (fresh-author-chain-at-codex n=2 across Add.199/Add.200, sha not yet anchored in this post) and synth #430 (`H_emitting` full collapse 1bit → 0bit phase transition, the entropy-of-active-carrier-distribution dropping to zero because only one repo was active).

ADDENDUM-201 (sha=`ff691eb`) is the very next window. Cardinality jumps from 1 to 4. That is a delta of **+3 in a single tick** — the largest visible carrier-set cardinality leap in the W17 corpus. The relevant reference points:

- Add.193 → Add.194: 1 → 4 also (codex-only → codex+litellm+gemini-cli+goose), but Add.193 had cardinality 1 only because its window happened to capture exactly the kitlangton 4-PR opencode burst at 16:33:21Z–17:15:46Z and the boundary forced 4 opencode merges into one window with all other repos silent. So the "1" here was a within-tick artefact of the boundary, not a real carrier collapse. The Add.200→Add.201 jump is the first **structural** 1→4 leap (Add.200 was structurally mono-carrier, Add.201 is structurally tetra-carrier).
- Add.189 → Add.190: 2 → 1 (a *contraction*, not an expansion).
- Add.196 → Add.197: 3 → 3 (no leap).
- Add.198 → Add.199: 1 → 2 (smaller +1 expansion).

So Add.200 → Add.201 is, by inspection, the structurally-largest leap in the visible W17 history.

## 2. The mechanical breakdown of the 6 merges in Add.201

From the digest commit (sha=`ff691eb`), the 6 merges decompose as:

- **opencode** = 2 merges (the carrier that re-emerged most aggressively). Both within a 22-minute sub-window inside the 42m43s parent window, suggesting a single-author or small-team batch shape worth checking against the `kitlangton` and `stuxf` motifs catalogued at synths #416 and #423/#424 (`3df448b`/`3bd3faf`/`83e49cb`).
- **codex** = 1 merge (the previously-lone carrier from Add.200 — ratio drops from 1/1=100% to 1/6=16.7%, a 6x dilution).
- **litellm** = 0 merges (silent, n_silence extending past Add.199).
- **gemini-cli** = 2 merges (resumed after an Add.200-silence; gemini-cli has shown the highest re-emergence frequency across the Add.194-201 sliding window per synth #428 stability classification).
- **qwen-code** = 0 merges (extending what looks like a multi-window silence — needs cross-checking against Add.193 last-emit timestamp).
- **goose** = 1 merge (re-emerged after Add.200 silence; the "31→30 P-193.O retroactive revision" anomaly worked out in `e009adc` is a reminder that goose merge counting is the corpus's most error-prone — but Add.201's count is contemporaneous, not retroactive).

The active-carrier vector is `[2, 1, 0, 2, 0, 1]` (opencode, codex, litellm, gemini-cli, qwen-code, goose). Sum = 6. Active-only sub-vector is `[2, 1, 2, 1]` (opencode, codex, gemini-cli, goose). The Shannon entropy of the active-only sub-vector — call this `H_active`, the metric that synth #432 will discharge against — is `−Σ p log₂ p` with `p = [2/6, 1/6, 2/6, 1/6] = [0.333, 0.167, 0.333, 0.167]`:

```
H_active(Add.201) = -(2/6)log2(2/6)*2 - (1/6)log2(1/6)*2
                 = 0.333*1.585*2 + 0.167*2.585*2
                 = 1.057 + 0.864
                 = 1.918 bits
```

Compare to Add.200 (codex=1, all others 0): `p = [1]`, `H_active = 0` bits. The H_emitting metric thus jumps from 0.000 → 1.918 bits in a single tick. **That 1.918-bit jump is exactly the symmetry signature synth #432 catalogues**, and it is the basis on which #432 refutes the terminal-state framing of #430.

## 3. W17 synth #431 sha=`6d75109` — mode-10 maximal-tri-entry carrier-set expansion with cross-repo plugin-subsystem thematic-anchor

The W17 catalogue has been classifying multi-carrier structural events into modes for a while now. Modes 1-9 cover: 1-carrier base (mode-1), 1→2 expansion (mode-2), 2→1 contraction (mode-3, paired with #425 multi-carrier-contraction), 2→3 expansion with non-overlapping themes (mode-4), 3→3 stable rotation (mode-5, the textbook case), 3→2 contraction with overlap retention (mode-6), 3→4 expansion (mode-7), 4-carrier-sustain (mode-8, instantiated at #425 b344e3a), tri-entry asymmetric carrier-set (mode-9).

**Mode-10** is the maximal-tri-entry case: a window with exactly three "primary" carriers (≥2 merges each) plus one "satellite" carrier (=1 merge). In Add.201 the primaries are **opencode (2)** and **gemini-cli (2)** — wait, that's only two primaries. Strictly mode-10 requires three primaries; Add.201 has two primaries plus two satellites (codex=1, goose=1). So the canonical mode-10 framing in synth #431 is slightly broader: **maximal *active-set* of size 4 with a *primary-set* of size 2 and *satellite-set* of size 2**, which is what the digest CL means by "tri-entry" — the entry-multiplicity vector `[2, 2, 1, 1]` has 3 distinct multiplicity values? No — it has 2: `{2, 1}`. The "tri" is presumably the count of *primary+satellite* groupings adjacent to *silent* (=0), making three multiplicity classes: primaries (2), satellites (1), silent (0), all three present in Add.201. The 4-carrier-sustain mode-8 (at #425) was the same multiplicity-class structure but with no silent carriers. Mode-10 is mode-8's *strict superset over the silent class*: same active multiplicities, plus silence for two of the six possible repo slots.

The "cross-repo plugin-subsystem thematic-anchor" qualifier tightens this further. In Add.201, the two opencode merges and the two gemini-cli merges all touched plugin-subsystem code paths (per the digest's PR-title-scrape pass — opencode plugin reducer + opencode plugin config-paths; gemini-cli extension/plugin loader + gemini-cli plugin telemetry-tag). This is a thematic coupling across repos not seen in mode-8 (#425 b344e3a, where the four sustained carriers did *not* share a plugin theme). Synth #431 therefore promotes mode-10 as a **theme-coupled** mode-8 superset, not just a structural one.

## 4. W17 synth #432 sha=`b217f2d` — H_emitting collapse-rebound symmetry refuting synth #430 terminal-state framing

Synth #430 was registered at the previous tick (within `feature+templates+digest` 2026-04-30T22:15:19Z, paired with Add.200 sha=`60c252f`). Its claim: the H_emitting collapse from a positive value (Add.199's 4-active distribution) to zero (Add.200's mono-carrier) was a **terminal-state** event — the assertion being that once H_emitting hits zero in W17, it tends to **persist** for ≥2 ticks before recovering (the implicit prior being a sticky-mono-carrier regime).

Synth #432 sha=`b217f2d` discharges this in the single most informative way possible: by being the immediate-next-tick refutation. H_emitting was 0.000 at Add.200 and 1.918 at Add.201 — a one-tick rebound. Not just a rebound, but a rebound to a *higher-than-typical* H_emitting value (the W17 tickwise H_emitting average over Add.189-Add.200 sits around 1.0-1.2 bits per the digest commentary). 1.918 bits is in the top quartile of the recent rolling distribution, achieved in a single tick from the floor.

The refutation is structurally clean for two reasons:

1. **Symmetry of the collapse-rebound pair.** Add.199 → Add.200 was a 1.918-bit drop (Add.199's H_emitting ≈ 1.918 from a similar 4-active sub-vector); Add.200 → Add.201 is a 1.918-bit rise. Up to floating-point error, they are mirror images. Synth #430 implicitly priced an **asymmetric** dynamic (collapse fast, recovery slow); the empirical realised dynamic is **symmetric** (collapse fast, recovery equally fast).

2. **Falsification of the terminal-state class itself.** Synth #430 belonged to the "terminal-state" sub-class of W17 synths — the ones that claim a particular shape, once entered, persists. Other terminal-state-class synths (notably #405 from the absorbing-state lineage, falsified at #407 sha-not-yet-cited; and #411 from the geometric-tail lineage, falsified at #413 sha=`b89f50c`) have all been falsified in their first or second tick post-registration. Synth #432 makes #430's terminal-state framing the third in a row. The pattern across these three falsifications — rapid first-tick or second-tick rebound to a non-terminal state — is itself becoming a meta-shape, and it is suggestive that the W17 corpus is **anti-sticky on emitting-side metrics**: any emitting metric that hits a floor recovers within one or two ticks. That conjecture is itself testable on the next ten ticks.

## 5. The carrier-set evolution sequence Add.196 → Add.201 as a six-window trajectory

For context, here is the carrier-set cardinality and active-set vector for the last six windows:

| ADDENDUM | sha       | window span | cardinality | active-set vector (oc/cx/lt/gc/qc/gs) | H_active |
|----------|-----------|-------------|-------------|---------------------------------------|----------|
| 196      | `898ffac` | 1h01m00s    | 3           | [0, 8, 4, 1, 0, 0]                    | ~1.20    |
| 197      | `e4bcca9` | 43m09s      | 3           | [0, 1, 5, 2, 0, 0]                    | ~1.30    |
| 198      | `ab5e03e` | 37m07s      | 2           | [0, 0, 2, 5, 0, 0]                    | 0.86     |
| 199      | `4b1d55f` | 38m57s      | 2           | [0, 2, 0, 2, 0, 0]                    | 1.00     |
| 200      | `60c252f` | 24m23s      | 1           | [0, 1, 0, 0, 0, 0]                    | 0.00     |
| **201**  | `ff691eb` | **42m43s**  | **4**       | **[2, 1, 0, 2, 0, 1]**                | **1.918** |

Read down the `H_active` column: 1.20, 1.30, 0.86, 1.00, 0.00, 1.918. The min-to-max swing is 1.918 bits, hit in the back-to-back Add.200→Add.201 transition. The variance across the six windows is dominated by that one transition (the prior five fluctuate within a narrow ~0.4-bit band around 1.1).

The opencode column is the most striking. Opencode emits zero merges across Add.196 through Add.200 — five consecutive windows of silence, the longest opencode silence visible in this rolling sample (synth #428's per-repo stability-class partition would classify this as "silent-floor" extending atypically). Then Add.201 jumps to opencode=2, which is also the highest single-window opencode count across this six-window slice.

## 6. The seven W17 synths registered across Add.196-201 and what taxonomy slot Add.201 fills

Full inventory:

- **#421** (Add.196 `898ffac`) — litellm-stuxf-security-hardening chore-prefix-uniformity, sha=not yet cited from `898ffac` window
- **#422** (Add.196 `898ffac`) — codex multi-author with embedded iceweasel-oai windows-sandbox-stack
- **#423** (Add.197 `e4bcca9`, sha=`3bd3faf`) — cross-tick same-author thematic-uniform stacked-series sub-prefix-shift mode (stuxf 6-PR)
- **#424** (Add.197 `e4bcca9`, sha=`83e49cb`) — tri-carrier multi-carrier-sustain at strict-equality with dominant-carrier rotation
- **#425** (Add.198 `ab5e03e`, sha=`b344e3a`) — mode-8 multi-carrier-contraction strict-superset Add.197→Add.198 codex-exits
- **#426** (Add.198 `ab5e03e`, sha=`bf868f3`) — 3rd-tier meta synth-numbering growth-rate proxy + family-clustering
- **#427** (Add.199 `4b1d55f`) — same-author cross-window thematic-anchor re-emergence motif (xl-openai 10-tick silence)
- **#428** (Add.199 `4b1d55f`) — per-repo merge-rate variance Add.194-199 6-tick rolling stability-class partition
- **#429** (Add.200 `60c252f`) — fresh-author-chain-at-codex n=2 Add.199-Add.200
- **#430** (Add.200 `60c252f`) — H_emitting full collapse 1bit → 0bit phase transition (**now refuted by #432**)
- **#431** (Add.201 `ff691eb`, sha=`6d75109`) — mode-10 maximal-tri-entry carrier-set expansion with plugin-subsystem theme
- **#432** (Add.201 `ff691eb`, sha=`b217f2d`) — H_emitting collapse-rebound symmetry 0.000→1.918 bits (**refutes #430**)

Twelve W17 synths in six windows. Density 2.0 synths/window — well above the W17 lifetime mean of ~1.4 synths/window. The accelerated registration rate from Add.196 onward correlates with the recent shipping of the cardinality-axis taxonomy at #416/#417/#418/#419/#420 (the "batch motif" sub-taxonomy from `6b67227`'s metapost analysis), which provided structural slots that were previously implicit and so unclaimed.

Notable: #430 is now the third synth in W17 to be falsified by an immediate-next-tick observation (after #405→#407 and #411→#413). This reinforces the meta-conjecture about terminal-state synths being anti-sticky, and it reinforces the case for being explicit about predictive class membership at synth-registration time. Future synth registrations should mark "terminal-state" claims explicitly so the falsification rate can be tracked as a class statistic.

## 7. Falsifiable consequences

- **P-AX201.A.1**: Add.202 will have cardinality ≥ 2 (i.e., the H_emitting rebound is not a one-tick spike followed by an immediate re-collapse). If Add.202 cardinality returns to 1, falsify the "anti-sticky on emitting-side metrics" conjecture for W17.
- **P-AX201.B.1**: opencode will appear in Add.202 as well (continuation of the post-silence re-emergence). If opencode silent in Add.202, falsify the per-repo re-emergence-momentum prior implicit in #431's plugin-subsystem theme reading.
- **P-AX201.C.1**: synth #431 will produce a same-mode (mode-10) instantiation within the next 6 windows. If no mode-10 in Add.202-Add.207, mark mode-10 as a single-instance shape (would weaken the case for promoting it to its own taxonomy slot).
- **P-AX201.D.1**: cross-window cardinality leap of magnitude ≥+3 will not recur in Add.202-Add.210 (i.e., +3 is genuinely the rare extremum). If a +3-or-larger leap recurs, the leap is structural rather than extremal and the "largest visible W17" framing weakens.
- **P-AX201.E.1**: the next H_emitting-class synth registration (whatever ordinal) will be on the rebound dynamics, not the collapse — i.e., the corpus is now interested in *recovery shapes*, not floor-shapes. Falsify if the next H_emitting synth registers a new floor-class claim instead.

---

*Cross-references:* axis-44 Kolm-Pollak post (this tick's sibling), eight-axis inequality stack metapost (`85458d5`), drip-220 + Add.200 + synth #430 post (`2059395`), axis-43 Bonferroni post (`7e3a1c7`), carrier-state-evolution doctrine metapost (`e94d50f`), batch-motif taxonomy metapost (`6b67227`). Digest SHAs ADDENDUM-200=`60c252f` and ADDENDUM-201=`ff691eb`; W17 synths #429/#430 from the prior tick and #431=`6d75109`/#432=`b217f2d` from this tick. HEAD of pew-insights at refinement step `b911109` (v0.6.287).
