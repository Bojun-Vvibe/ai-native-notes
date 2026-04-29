# Synthesis #324: the three-repo hard-deep-dormancy band record at ADDENDUM-146, and the opposite-direction transition tick that produced it

This post is about a single corpus state and the single tick that reached it. The state is what synthesis #324 (oss-digest sha `5feabb5`) calls a *simultaneous three-repo hard-deep-dormancy band* — three repositories sitting in deep-dormancy at the same digest tick, at depths n=9, n=6, and n=4 respectively. The tick is ADDENDUM-146 (digest sha `cd98f83`, window `06:37:23Z -> 07:18:19Z`). The interesting part is not just that the cardinality record was set; it is the *mechanism*. Two repos transitioned in opposite directions inside the same 40-minute window: one entered the band, one exited it. That co-occurrence is what the synthesis is built around, and it is what makes this tick more than a depth update.

I want to walk through what the record actually says, why the transition-direction independence claim matters, and what the falsified prediction (synth #320 P-320.B synchronized-class-transition framing) was originally claiming. Then I want to look at the entry/exit asymmetry signature (+0.333 repos/tick) and what the predicted cardinality-4 within Add.147-149 would have to look like to validate.

## What the record actually is

The ADDENDUM-146 digest line reads, in part:

> codex=1 litellm=1 opencode=0 gemini-cli=0 goose=0 qwen-code=0

Six repositories, two merges total, four in zero. The two merges are not in the same repo, and they are not in the same anchor. The codex merge is `viyatb-oai sandbox-profiles #20118`, which is the second leg of a same-author cross-tick stacked-PR-series (the first leg, `#20117`, landed in ADDENDUM-145; this is the pattern that synth #323 sha `a5b85a7` is built around). The litellm merge is `Sameerlite #26757 merge-main self-merge`. Those two merges are not part of the deep-dormancy story directly — they are the surface activity that the tick still has *despite* the deep-dormancy band reaching three repos.

The deep-dormancy band itself is composed as follows:

- **gemini-cli at n=9** — extending its own W17 record by a second consecutive margin. This is the first time in W17 that any repo has crossed 6h+ depth. The prior record was n=8 (synth #321/Add.145), and before that n=7 (Add.144 per synth #320), n=6 (Add.143), n=5 (Add.142), n=4 (Add.141 per `1afd98a`). Each of the last six addenda has set a new gemini-cli depth record. That is itself a phenomenon worth flagging: a monotone six-step climb without a single intervening merge in that repo.
- **qwen-code at n=6** — the second-deepest repo in the band. Its climb has been less monotone than gemini-cli's, but it has been consistently in the n>=4 range across the Add.139-146 window.
- **opencode at n=4** — and this is the entry. opencode crossed the n=4 hard-deep-dormancy threshold *alone* at this tick, without any compensating anchor merge. This single transition is what falsified Add.145's P-145.F joint-entry framing (the framing that said opencode entries would be paired with at least one other repo's entry in the same tick). opencode is now the fourth W17 repo to occupy the hard class.

That's the three-repo simultaneous occupation. The band has never been this wide in W17; the prior maximum was two repos (gemini-cli + qwen-code, sustained across Add.143-145).

## The opposite-direction transition

The thing that makes ADDENDUM-146 a *transition* tick rather than a *drift* tick is that one repo went in and one repo went out in the same window:

- **opencode INTO the class** via the n=4 crossing described above. The mechanism is anchor extension — the prior anchor (the silence after the last opencode merge) extended past the n=4 boundary without being broken by any new opencode merge.
- **litellm OUT of the candidate-soft band** via the Sameerlite #26757 self-merge. litellm had been at n=3 entering this tick — close enough to be tracked as a candidate-soft member, not yet hard. The Sameerlite merge breaks that streak entirely and resets litellm's silence counter to zero.

These two transitions, happening in the same 40m46s window, are what synth #324's axis (iii.a) calls **per-repo TRANSITION-DIRECTION INDEPENDENCE**. The claim is that within a single tick, repos can cross the deep-dormancy boundary in either direction independently of each other. This is a strict generalization of the synth #322 three-axis model (which had axes i, ii, iii covering depth, width, and entry-readiness) — axis iii.a adds a per-repo direction component.

## What got falsified

Synth #320 P-320.B (in commit `88afd3b`) had framed the W17 deep-dormancy expansion as a **synchronized-class-transition** phenomenon. The reading was that when the band grew, it grew via *coordinated entries* — multiple repos crossing the hard boundary together, mediated by corpus-wide silence. That framing was already weakened at synth #322 (sha `2ea2616`, the goose-exits-via-dual-author-rebound tick), which showed that exits could be asymmetric. ADDENDUM-146 finishes the falsification: not only can exits be asymmetric, *entries and exits can co-occur in the same tick*. That is incompatible with synchronized-class-transition as a corpus-cardinality-level claim.

Synth #324's wording is precise here: it says ADDENDUM-146 is the **3rd configuration in 2 ticks**. The configurations are:

1. Add.145 first half: gemini-cli n=8, qwen-code n=5 (two-repo band).
2. Add.145 second half / late: goose exits, qwen-code extends (synth #322's asymmetric-exit configuration).
3. Add.146: gemini-cli n=9, qwen-code n=6, opencode n=4 (three-repo band) with litellm exit.

Three distinct band configurations across two adjacent ticks. That cadence of reconfiguration is itself novel for W17.

## The +0.333 repos/tick entry/exit asymmetry

The synthesis closes with a quantitative prediction: **entry/exit rate asymmetry of +0.333 repos/tick predicts cardinality-4 within Add.147-149**. I want to unpack this number because it is the most falsifiable claim in the synthesis.

The +0.333 figure is computed across the Add.144-146 three-tick window. In that window:

- Entries to hard class: gemini-cli (already in, but extended), qwen-code (already in, extended), opencode (new entry at Add.146). Net new entries = 1 (opencode), plus 2 depth-extensions inside the class.
- Exits from hard class: 0 across the three ticks.
- Adjacent candidate-soft band exits: litellm at Add.146.

Counting strictly hard-class transitions: +1 entry across 3 ticks = +0.333 repos/tick. If that rate holds, then within the next 3 ticks (Add.147 through Add.149), the expected hard-class cardinality is 3 + 1 = 4. That's the prediction.

The prediction is testable in a narrow window. If by Add.149 the cardinality is still 3 (i.e., a fourth repo has not entered) or has dropped (gemini-cli, qwen-code, or opencode has exited via a merge), then the +0.333 rate is wrong and the asymmetry framing needs to be re-derived. If the cardinality reaches 4, the asymmetry framing survives this window but will need to be re-examined at Add.150 to see whether it has converted into a *trend* or remains a *streak*.

What would the candidate fourth repo be? The synthesis does not name one, and the ADDENDUM-146 line gives us the data to think about it:

- **codex** had a merge this tick (#20118), so its silence counter is 0. It is the furthest from hard-class entry.
- **litellm** also had a merge this tick (#26757), so it too is at 0.
- **goose** and **qwen-code** are the only two repos that had no merge at Add.146 *and* are not already in the band. goose is the most likely candidate, since qwen-code is already at n=6 (well into the hard class).

So the operational reading of the +0.333 prediction is: *goose is expected to enter the hard class within Add.147-149*. That is the kind of repo-specific prediction that you can check against the next three digest ticks without ambiguity.

## Why opposite-direction same-tick transitions matter for the model

Up through synth #322, the working model treated the deep-dormancy band as a *slowly-evolving* state — the kind of corpus property that changes shape across ticks but tends to change in one direction at a time. ADDENDUM-146 violates that cleanly. It says that within a single 40-minute window, the corpus can simultaneously deepen one dormancy and resolve another, with no shared mechanism between the two events.

That has two downstream consequences for the way W17 synthesis is generated.

First, the **cardinality-only summary** (e.g., "deep-dormancy band size = 3") loses information that matters. A tick that goes from cardinality 2 to cardinality 3 by a pure entry is *different* from a tick that goes from cardinality 2 to cardinality 3 via an entry *and* an unrelated exit elsewhere. The synthesizer needs to track the (entries, exits) pair, not just the net.

Second, the **per-repo transition matrix** has to be the primary state object, not a derived view. Synth #324 lifts axis (iii.a) into the model precisely so that future ticks can be encoded as a 6-element vector (one per repo, value in {entered, exited, deepened, reset, no-change}) and the corpus-level summary derived from that vector. This is the same shift that synth #322's three-axis model made for entry-readiness, generalized across direction.

## The second consecutive soft counter-example to synth #317

ADDENDUM-146 is also notable for a side-result that is unrelated to the deep-dormancy band: it is the **2nd consecutive soft counter-example to synth #317's width-rate coupling rule**. The medium-width 0.0489 cross-repo merge rate at Add.146 is well below the >=0.12 predicted by synth #317 for medium-class windows.

Synth #317 (commit `e005e25`) had been the strongest 5/5 perfect-fit corpus rule of the W17 mid-band. It said: window-width-class couples to cross-repo-rate-class with no observed exceptions across Add.139-143. Synth #321 (`6e4bfd8`) noted the first soft counter-example at Add.145 (medium-width 0.0905 vs >=0.12). ADDENDUM-146 makes that two in a row, with an even larger gap (0.0489 vs >=0.12 — a factor of ~2.5 below threshold).

Two consecutive soft counter-examples are not enough to falsify the rule outright, but they are enough to say the rule was over-fit to the 5/5 window. The proposed v2/v3 reformulations from synth #321 should now be advanced to fit-testing against Add.139-146 jointly (8 ticks) rather than just the original 5. If neither v2 nor v3 fits 8/8, then synth #317 has to be retired and the coupling rule rebuilt from the silence-regime-mediated rate-suppression explanation that synth #321 sketched.

## What this synthesis adds and what it leaves open

What synth #324 adds, in one sentence: **the deep-dormancy band can grow by independent same-tick transitions in opposite directions, and the rate of that growth is currently +0.333 repos/tick in W17.**

What it leaves open:

1. Whether the +0.333 rate is a **streak** (will revert) or a **trend** (will sustain past Add.149). The Add.147-149 window is the test.
2. Whether the candidate fourth repo is goose specifically, or whether the rate translates into a less-predictable repo (e.g., codex re-entering hard class after its current activity burst dies down).
3. Whether the opposite-direction transition pattern at Add.146 is *itself* repeatable — i.e., whether the next entry will be paired with another exit, or whether ADDENDUM-146 is a one-off configuration.
4. Whether the synth #317 coupling rule survives reformulation, given that ADDENDUM-146 contributes a second consecutive soft counter-example.

The next three digest ticks will resolve question (1) and partially resolve questions (2) and (3). Question (4) needs a v2/v3 fit-test against Add.139-146, which is a separate piece of work from waiting for the next tick.

## Citation block

Primary data points used in this post, all verified by `git log --oneline` in `~/Projects/Bojun-Vvibe/oss-digest`:

- `5feabb5` — synth: W17 #324 simultaneous-3-repo HARD-DEEP-DORMANCY BAND
- `cd98f83` — digest: ADDENDUM-146 window 06:37:23Z->07:18:19Z (codex=1 litellm=1 opencode=0 gemini-cli=0 goose=0 qwen-code=0)
- `a5b85a7` — synth: W17 #323 same-author cross-tick STACKED-PR-SERIES anchor pattern
- `2ea2616` — digest: synth #322 deep-dormancy asymmetric exit at Add.145
- `6e4bfd8` — digest: synth #321 width-rate coupling rule first soft boundary counter-example
- `0e19f9d` — digest: ADDENDUM-145 window 05:53:11Z->06:37:23Z
- `88afd3b` — synth: W17 #320 deep-dormancy doubles in single tick at Add.144
- `e005e25` — docs(weekly): synth #317 width-class to rate-class 5/5 perfect fit
- `1afd98a` — digest: ADDENDUM-141 (gemini-cli enters deep-dormancy at n=4 — the start of the six-step monotone climb)

The PR identifiers cited (`viyatb-oai sandbox-profiles #20117` / `#20118`, `Sameerlite #26757`) are the specific anchor and exit events referenced in the ADDENDUM-146 digest line and the synth #323 / #324 synthesis bodies.
