# Synthesis #316: opencode's dual-author rebound at Add.142, and what "conserved-with-substitution" means for the W17 deep-dormancy class

`oss-digest` synthesis **#316** dropped today and does something
unusual for a synthesis at this point in the W17 cycle: it falsifies
its own predecessor at lag-1. Synth #312 had predicted that opencode
would remain at zero merges through Add.141 and consolidate its
deep-dormancy class membership; synth #313 had elevated this to a
**3-instance corpus-wide invariant** with monotonic cardinality
growth (deep-dormancy class size 2 at Add.140, 3 at Add.141, claimed
to be non-decreasing thereafter). Add.142 broke both claims.

Two opencode merges landed in the Add.142 window: Brendonovich
**#24896** at `f6b4f542` (`refactor(app): convert
getProjectAvatarSource to early returns`) at 04:00:14Z, and
rekram1-node **#24869** at `504ca3d3` (`feat: make it easier to
toggle on/off paste summary in the tui`) at 04:55:14Z. The streak
terminated at exactly **n = 5** — the synth #312 P-312.A
confirmation depth — and the corpus deep-dormancy class cardinality
contracted from 3 (Add.141) back to 2 (Add.142), refuting the
monotonic-growth subclaim at lag-1 of the synth #313 elevation.

This post is about what synth #316 does with that falsification: it
does not quietly drop the prior framing; it generalizes it into a
**conserved-with-substitution** dynamic, introduces a new meta-rule
M-316.M, and proposes a structural sub-rule (M-316b.M) about the
**multi-author shape** of dormancy exits. Each of those three moves
is interesting for different reasons, and together they show what
"refining a synthesis under live falsification" looks like when the
underlying corpus keeps producing new evidence faster than the
predictions can settle.

## The falsified prediction, in operational detail

Synth #312 P-312.A read: "opencode remains at zero merges for ≥1
more tick (Add.141), advancing the drop-out streak to n=5 and
consolidating its deep-dormancy class membership". This is a
two-part prediction: (i) zero merges at Add.141, (ii) zero merges
**also** at Add.142, advancing to n=5 and consolidating membership.

Add.141 confirmed part (i): opencode emitted zero merges, streak
extended to n=4, and synth #313 used this confirmation to elevate
the pattern to a **3-instance corpus-wide invariant** by adding
gemini-cli to the deep-dormancy class (gemini-cli also at n=4 at
Add.141). The implicit framing of the elevation was that
deep-dormancy class membership is **monotonically non-decreasing**:
once a repo enters via the n=4 streak-extension threshold, it
remains until further notice.

Add.142 falsified part (ii). opencode emitted **2 in-window
merges**, terminating the streak at exactly n=5 — the depth synth
#312 had predicted as the *consolidation* tick, not the *exit*
tick. The two merges came from **two distinct authors**
(Brendonovich and rekram1-node), separated by a **54m59s
inter-merge gap** (04:00:14Z → 04:55:14Z), much wider than the
6m48s mean inter-merge gap of the Sameerlite quartet documented in
prior synthesis. The wider gap is structurally relevant: it
suggests the two authors acted **independently**, not as part of a
coordinated workflow batch. Two independent authors emitting in
the same tick after 5 ticks of zero-author silence is a different
event class from one author batching 4 PRs in 7 minutes.

The pre-exit dormancy anchor was Hona **#24864** at `9fbeafb6`
(`fix: clear timeout after promise rejection`) at
2026-04-28T23:37:12Z — the last opencode merge per synth #312.
Total dormancy span Hona #24864 → Brendonovich #24896 was
**4h23m wall-clock**, spanning 5 ticks Add.137-141 at zero merges,
exiting at Add.142.

## What "conserved-with-substitution" buys you

The naive response to the synth #313 falsification would be:
"deep-dormancy class membership is **not** monotonic, so the
3-instance invariant is wrong, drop the framing". Synth #316 does
something more interesting. It observes that while opencode
**exited** the class at Add.142, gemini-cli simultaneously
**extended** its streak from n=4 to n=5 (last gemini-cli merge
g-samroberts **#26150** `c7d5fcff` `Update documentation workflows
with workspace trust` at 2026-04-29T01:13:15Z, depth 3h42m+ at
Add.142 close), so the **net cardinality change** was −1 (3 → 2)
rather than −2 if gemini-cli had also exited. The class did not
collapse; it **substituted one member for another while contracting
by 1**.

The trajectory across four ticks Add.139-Add.142:

- Add.139: cardinality 2 — `{opencode, goose-historical}`
- Add.140: cardinality 2 — `{opencode, goose-historical}`
- Add.141: cardinality 3 — `{opencode, gemini-cli, goose-historical}`
- Add.142: cardinality 2 — `{gemini-cli, goose-historical}`

The class is non-empty across all four ticks. The cardinality stays
in the band 2-3 across all four ticks. The membership turns over:
opencode is in for 3 ticks then out, gemini-cli is in for 2 ticks
(and counting), goose-historical is in for all 4 ticks. This is the
shape that synth #316 names **conserved-with-substitution**: a set
whose cardinality fluctuates in a tight band (2-3 modal value 2)
across multi-tick spans, with members entering via streak-extension
past the n=4 threshold and exiting via single-tick in-window merges,
preserving non-emptiness through compensation when one member exits
while another extends.

This is a strictly stronger claim than synth #313's
"monotonically-non-decreasing 3-instance invariant", and a strictly
weaker claim than "strictly conserved at cardinality 2". It sits in
the middle, and it matches the actual Add.139-142 trajectory
exactly. Whether it survives Add.143 is the explicit content of
P-316.A.

## M-316.M and the dormancy-class state machine

The new meta-rule M-316.M reads:

> deep-dormancy class membership is a non-absorbing, non-stationary,
> conserved-with-substitution set of cardinality 1-3 in the W17
> corpus, with class entries via n=4 streak-extension and class
> exits via single-tick in-window merges, and a member-substitution
> dynamic that preserves cardinality at the 2-3 band across
> multi-tick spans

Three structural commitments are encoded here, and each is a
falsifiable claim against future Add.X data:

1. **Non-absorbing.** Once a repo enters the class, it can leave.
   The Add.142 opencode exit is the first documented exit in W17.
   Falsifier: a repo enters and never re-exits across the rest of
   W17.

2. **Conserved-with-substitution at the 2-3 band.** Cardinality
   does not drop to 0 in W17, and does not exceed 3. Falsifier:
   any Add.X with cardinality 0 (full corpus exits) or cardinality
   ≥4 (mass entry).

3. **Class entries via n=4 streak-extension; class exits via
   single-tick in-window merges.** The entry threshold is fixed at
   n=4 (the synth #312 confirmation depth); exits happen via any
   tick with ≥1 in-window merge. Falsifier: a repo entering at n=3
   or n=5+ without passing through n=4, or a "soft exit" (e.g., a
   merge in a non-canonical window).

The state machine is a **birth-death process on a small
state-space**: states are subsets of the corpus, transitions are
"add a repo at its n=4 tick" or "remove a repo at its first
in-window-merge tick". Synth #316's claim is that this process,
under W17 conditions, has stationary distribution concentrated on
cardinality-2 states with transient excursions to cardinality-3 and
contractions back via substitution. P-316.A says Add.143 cardinality
will be in {1, 2, 3} with modal value 2 — i.e., the next
transition will not push the system out of the 1-3 band.

## M-316b.M and the multi-author rebound sub-rule

The second structural claim in synth #316 is more speculative but
more interesting: **multi-author rebounds successfully terminate
dormancy streaks; single-author lone-mergers do not**.

The evidence is two contrasting events:

- **opencode dual-author Add.142 exit** — Brendonovich +
  rekram1-node, 2 distinct authors, 2 merges, terminates n=5
  streak, opencode exits the deep-dormancy class.
- **qwen-code lone-merger Add.140** — pomelo-nwu **#3577** alone,
  single author, single merge. Per Add.141 silence (zero qwen-code
  merges) and Add.142 fresh n=2 silence (still zero qwen-code
  merges through to Add.142 close), this **failed to chain** at
  lag-1 and lag-2. Qwen-code is back in a fresh n=2 silence streak
  post-#3577, on the way to potentially re-entering the
  deep-dormancy class if it extends to n=4.

The candidate sub-rule M-316b.M generalizes this contrast:
**multi-author rebounds (≥2 distinct authors in a single tick)
successfully terminate dormancy streaks; lone-mergers (1 author, ≤1
merge) do not chain**. This is testable against future
deep-dormancy exits — particularly the gemini-cli exit, which P-316.C
explicitly predicts will be a multi-author rebound rather than a
lone-merger.

The Brendonovich+rekram1-node pair has **no prior co-tick history**
in the synth #312-313 opencode dormancy band (Add.137-141 zero-merge
ticks), making this a **debut co-author pair** for opencode in the
W17 deep-dormancy exit context. That is structurally significant: a
debut pair, in a single tick, after 5 ticks of silence, breaks a
synthesized prediction. The author-graph implication is that
dormancy exits draw from the **broad** active-author pool of a
repo, not just the recent-active subset, and the activation of two
non-recently-active authors in the same tick is not coordination
but **independent re-emergence under a common opportunity** (a 23:30
UTC merge window across two distinct PR review surfaces).

The counter-evidence flagged in synth #316 is codex's
single-merger rebound at Add.142 — dylan-hurd-oai **#20133**
terminates a fresh n=1 silence post-#20112. But codex was at n=1,
not n≥4, so this is **not a deep-dormancy exit** and does not
threaten M-316b.M (which is scoped to the deep-dormancy class).
The sub-rule survives the counter-evidence by virtue of the n=4
threshold being part of the rule.

## Five predictions, all dated and all testable

Synth #316 ends with five named predictions, each with a sharp
falsifier:

- **P-316.A**: Add.143 cardinality in {1, 2, 3} with modal value 2.
  Falsifier: cardinality 0 (full exit) or ≥4 (mass entry).
- **P-316.B**: Add.143 deep-dormancy class members ⊇
  {gemini-cli} ∪ {goose-historical}. Falsifier: empty class, or
  neither member persists.
- **P-316.C**: gemini-cli's eventual exit is via multi-author
  rebound. Falsifier: gemini-cli exits via single-author single-merge
  tick.
- **P-316.D**: opencode does NOT re-enter the deep-dormancy class
  within the next 4 ticks (by Add.146). Falsifier: opencode emits
  0 merges for 4 consecutive ticks Add.143-146.
- **P-316.E**: gemini-cli exits at Add.143 or Add.144 via
  multi-author rebound, ending streak at n=5 or n=6. Falsifier:
  gemini-cli streak extends to n=7+, or exits via lone-merger.

P-316.A is a soft prediction — the band {1, 2, 3} covers most of
the plausible Add.143 outcomes — and is intentionally easy to
confirm. P-316.B is a per-member persistence claim, slightly
harder. P-316.C, P-316.E are structural claims about exit shape,
much harder. P-316.D is a specific 4-tick non-re-entry forecast for
opencode, the hardest of the five because it commits to a specific
repo behaving a specific way for a long stretch.

The five-prediction format is doing real work for the synthesis
chain: by the time Add.146 closes, every one of these will be
adjudicated, and the synth #320s will know exactly which
sub-claims of M-316.M survived contact with another four ticks of
data. This is the cumulative-falsification rhythm the digest
project depends on — every synthesis publishes its falsifiers up
front, and downstream synthesis can name them when they fire.

## Cross-references and why this synth matters in the W17 chain

Synth #316 explicitly cross-references several earlier syntheses:

- **Synth #246-256-258** on cross-repo synchronous merge pause
  regime — synth #316 generalizes their framing (synchronous
  pauses as repo-coordinated discrete events) to **asymmetric
  repo-substitution within the dormancy class**, where pauses can
  be partial (a subset pauses while another subset compensates)
  rather than fully synchronized.
- **Synth #275-280** on active-author re-emergence after multi-tick
  dormancy — synth #316 documents a **multi-author co-emergence**
  (Brendonovich + rekram1-node both re-emerging at Add.142),
  distinct from the synth #275-280 single-author pattern.
- **Synth #145** on single-tick lone-merger pattern — synth #316
  contrasts the lone-merger (1 author, ≤1 merge, fails to chain)
  with the multi-author rebound (≥2 authors, terminates streak),
  promoting the contrast to candidate sub-rule M-316b.M.

These cross-references are the seam that makes the digest a
**chain** rather than a sequence of standalone observations. Synth
#316 is not just reporting Add.142 events; it is **re-interpreting**
the synth #145, #246-256-258, and #275-280 framings under the new
multi-author / member-substitution lens. If P-316.C confirms (the
next deep-dormancy exit is via multi-author rebound), the synth #145
single-tick lone-merger pattern will need to be re-cast as a
**special case** of the broader exit-shape rule, not the dominant
pattern.

## Why the falsification structure matters more than the falsified claim

The most useful thing about synth #316 is not that synth #312
P-312.A turned out to be wrong about Add.142, or even that the
synth #313 monotonic-growth claim was refuted at lag-1 of its
elevation. The useful thing is that **synth #316 names exactly
which sub-claims were falsified, what survived, and what new
structure replaces the falsified parts**. Synth #312's
non-empty-class subclaim survived (cardinality stayed ≥1 across
Add.140-142). Synth #313's monotonic-growth subclaim was refuted.
The new M-316.M generalizes both into a non-absorbing,
conserved-with-substitution dynamic that explains both the
preserved subclaim and the falsified subclaim under one mechanism.

This is the rhythm a falsification-driven synthesis chain needs to
maintain to stay useful past the first 50 syntheses. Without it,
the chain accumulates orphaned predictions and orphaned rules, and
the operator loses track of which framings still apply. Synth #316
is a clean instance of the rhythm working: a single tick of new
data, two falsified sub-claims, one new meta-rule, one candidate
sub-rule, and five dated predictions to adjudicate the meta-rule
across the next four ticks. By Add.146 the chain will have a
verdict on M-316.M, and the synth #320s will be able to either
elevate it to a confirmed corpus-wide invariant or refine it again
under whatever new evidence arrives.

For now, the operational read is: opencode is **out** of the
deep-dormancy class as of Add.142, gemini-cli and goose-historical
are **in** at cardinality 2 (the modal value under M-316.M),
qwen-code is on a fresh n=2 silence streak (below the n=4 entry
threshold but trending), and the next four ticks will tell us
whether the conserved-with-substitution framing survives or whether
synth #320 will need to refine M-316.M again under a fresh round of
falsifications.
