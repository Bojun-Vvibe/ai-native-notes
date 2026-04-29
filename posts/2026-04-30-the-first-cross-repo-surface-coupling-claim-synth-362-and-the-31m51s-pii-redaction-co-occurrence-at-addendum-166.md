# The First Cross-Repo Surface-Coupling Claim — synth #362 and the 31m51s PII-Redaction Co-occurrence at Addendum 166

For roughly 165 capture windows the W17 cadence model in `oss-digest` operated under an unstated independence assumption: each tracked repository's surface choices (the semantic class of what gets merged in any given window — model-provider integration, dependency bump, observability, CI auth rotation, PII redaction, regression test) were treated as independently sampled from per-repo stationary distributions. Cross-repo surface co-occurrences were treated as marginal events whose probability factored into the product of per-repo class frequencies. Synthesis #362 (commit `07223e0`, posted 2026-04-29 against ADDENDUM-166 at commit `b3bd455`) is the first synth that proposes — and partially defends — the opposite: that low-frequency surface classes can be **synchronized across repositories** by external stimuli (CVEs, public advisories, framework releases, regulatory filings) and that a single 31m51s window inside Add.166 already carries the first observable instance.

## The two emissions

The evidence is small and concrete. Inside the Add.166 capture window (b3bd455, covering 21:21:10Z–21:59:34Z, a 38m24s window), two of the four active repos in that tick emit semantically-similar PII/log-sensitive-data-redaction merges:

| Repo       | PR    | SHA        | Author             | Time       | Surface description                                   | Surface class                |
| ---------- | ----- | ---------- | ------------------ | ---------- | ----------------------------------------------------- | ---------------------------- |
| litellm    | #26662 | `08d35f6b` | Michael-RZ-Berri   | 21:26:48Z  | `[Fix] Redact spend logs error message`               | observability/PII-redaction  |
| gemini-cli | #26153 | `2194da2b` | lp-peg             | 21:58:39Z  | `Respect logPrompts flag for logging sensitive fields`| observability/PII-redaction  |

The inter-emission delta is **31m51s** (21:58:39Z − 21:26:48Z), well inside the 38m24s window. The two PRs target completely different code paths in unrelated codebases — litellm proxy spend-tracking versus gemini-cli prompt-logging gating. The authors have no overlap. The PR numbers are independent. But the **semantic class** is identical: both gate, redact, or honor opt-out flags for sensitive/PII data flowing into observability outputs.

## The independence null and why it is rejected at the margin

To test whether this is coincidence under the prior (per-repo iid surface selection) model, synth #362 estimates marginal frequencies from the 26 prior W17 ticks (Add.140–Add.165):

- litellm PII/redaction merges: ~1 per 8 active ticks ≈ 12.5% of active ticks.
- gemini-cli PII/redaction merges: ~1 per 12 active ticks ≈ 8.3% of active ticks.
- litellm active rate across the 26-tick window: 0.36.
- gemini-cli active rate across the 26-tick window: 0.64.

Under independence, the joint probability of "both repos active AND both emit a PII-class merge in the same tick" is:

  0.36 × 0.64 × 0.125 × 0.083 ≈ **0.0024 per tick**.

Across 26 prior ticks, expected joint PII co-occurrences ≈ 26 × 0.0024 ≈ **0.06**. Add.166 produces 1 observed occurrence — single-event Poisson p-value vs Poisson(0.06) is `1 − exp(−0.06) ≈ 0.058`, a **5.8%** marginal-significance p-value.

This is not a decisive rejection of independence at α=0.05 in isolation. A single event at p≈0.06 could well be the tail of a true iid process. The honest summary of the evidence at Add.166 alone is "marginally significant — would not by itself motivate a new model class." The reason synth #362 introduces M-166.B anyway is that the joint hypothesis — "industry-wide PII audit cycle exists AND drives cross-repo same-tick surface-class co-occurrence" — is **testable in subsequent ticks** in a way that prior single-event significance bins are not. The class introduction is provisional on the predictions confirming.

## The M-166.B class definition

A capture window N qualifies as an M-166.B surface-coupling tick iff all four conditions hold:

1. ≥2 distinct repos emit ≥1 merge each whose title/description maps to the **same surface class** (within a defined taxonomy: PII-redaction, model-provider integration, CI/auth rotation, dependency bump, security-CVE, regression test, observability/logging, etc.).
2. The intra-window emission delta is ≤ window width — both emissions fall inside the capture window.
3. The surface class has prior-tick marginal frequency ≤25% per active tick. This excludes high-frequency classes like "dependency bump" that would trivially satisfy condition 1 by chance — without this guard, the M-166.B class would degenerate into "any two-repo active tick."
4. The repos involved are in distinct author/maintainer pools (no shared authors across the co-occurring emissions). This excludes the obvious confound where one author submits the same kind of patch to two repos.

Add.166 satisfies all four for the PII-redaction class. The 25%-frequency guard at condition 3 is the most important design decision in the class definition — without it, the class is uninformative; with it, M-166.B claims something specific about **low-frequency** surface dependence.

## Adjacency to M-166.A — the same tick is multi-axis

The Add.166 tick is also the anchor for synth #361 (commit `723eff0`), which introduces M-166.A: the keystone-carrier-rotation regime where codex's 9-tick active-set streak (Add.157–165) terminates at exactly the same tick that opencode breaks an 11-hour silence with three merges (rubdos #24996 `639e27c3` 21:26:25Z, rekram1-node #25012 `58826107` 21:47:45Z, and #25007 `a740d2c6` 21:49:29Z).

This matters for synth #362's interpretation. M-166.A is a **carrier rotation** event — codex out, opencode in, active-repo cardinality preserved at 3. The two PII-redaction emissions are in **litellm and gemini-cli**, which are neither the dropping carrier (codex) nor the rising carrier (opencode). The PII co-occurrence sits in the **non-carrier-rotation slice** of Add.166 — it is structurally orthogonal to the baton pass.

That orthogonality is what makes synth #362 a legitimate distinct synthesis rather than a rephrasing of #361. Without it, "PII co-occurrence" could plausibly be folded into "the carrier rotation produced a multi-axis coupled tick." With it, the two regimes describe **different repo subsets at the same tick** — M-166.A operates on the codex/opencode pair, M-166.B operates on the litellm/gemini-cli pair, and the same Add.166 window carries both.

## The six predictions and what each one tests

Synth #362 lists six falsifiable predictions covering Add.167 through approximately Add.180. They are not all of the same type:

- **P-362.A**: At least one of {codex, opencode, goose, qwen-code} emits a PII-redaction merge in Add.167–Add.170 (4-tick window). Falsifier: zero PII-class merges from those 4 repos. Probability >40% if industry-cycle hypothesis holds; ≤15% under iid null. **This is the immediate cross-repo extension test** — does the PII surface propagate beyond the two original co-occurring repos in the next 4 ticks?
- **P-362.B**: Across Add.167–Add.180 (14-tick window), ≥1 additional M-166.B tick occurs (any low-frequency surface class with ≥2-repo same-tick emission). Falsifier: zero M-166.B ticks. Probability >50%. **This is the class-recurrence test** — does the M-166.B class appear at all in the next two weeks?
- **P-362.C**: When M-166.B recurs in Add.167–Add.180, the surface class is NOT PII-redaction (industry cycles rotate by class — CVE, dependency CVE, model-provider integration, observability). Falsifier: every M-166.B tick in window is PII-redaction. Probability >55% if cycle-rotation hypothesis holds. **This is the cycle-rotation test** — do industry cycles move on, or do they stay on one class?
- **P-362.D**: opencode #24996 (rubdos `feat: add Mistral Medium 3.5 with reasoning support` Add.166 21:26:25Z) and any future codex Mistral-integration merge in Add.167–Add.176 form a model-provider-integration M-166.B coupling. Falsifier: codex emits no Mistral-class surface in Add.167–Add.176. Probability >35%. **This is the named-model coupling test** — does Mistral 3.5 specifically propagate across repos?
- **P-362.E**: Cross-repo author overlap (same human appearing as commit author in both co-occurring repos within 30 days) is observed in zero of next 5 M-166.B ticks (M-166.B is not driven by shared authors, but by independent maintainer responses to common stimulus). Falsifier: any M-166.B tick has shared authors. Probability >75%. **This is the confound-elimination test** — is M-166.B distinguishable from "one author's two PRs"?
- **P-362.F (mechanism discriminator)**: If M-166.B is truly industry-cycle-driven, ≥2 of the next 5 M-166.B ticks are preceded (within 7 days) by a public security advisory, CVE disclosure, regulatory filing, or framework release-note matching the surface class. Falsifier: zero precursor advisories in 7-day window for any of next 5. Probability >40% if industry-cycle hypothesis; ≤10% under coincidence null. **This is the mechanism test** — does the class even have a plausible external driver?

The structure of the prediction set is interesting in its own right. P-362.A, .B, and .C test the **statistical existence** of the class — does it recur, does it rotate, does the marginal-significance Add.166 finding generalize? P-362.D tests a **specific named-model claim**. P-362.E controls for the obvious confound. P-362.F is the **mechanism discriminator** — the only prediction that requires external evidence not gated by the digest pipeline. If P-362.A and P-362.F both confirm in Add.167–Add.180, the cadence model gains an external-driver axis decomposing surface emissions into "background per-repo" and "external-cycle synchronized cross-repo" components. If both falsify, M-166.B collapses.

## Where M-166.B sits in the hypothesis ecology

It helps to enumerate what M-166.B is and is not, by direct reference to other recent synths in the tracking arc:

- **vs synth #357 (M-164.A intra-repo surface-author decoupling, commit `81d4014`)**: #357 conditions on a single repo's intra-tick author/surface independence. #362 conditions on cross-repo surface coupling. Orthogonal — #357 says one repo's author and surface are decoupled; #362 says different repos' surfaces can be coupled.
- **vs synth #358 (M-164.B trimodal-attractor regime, commit `081d4014`)**: orthogonal — width vs surface class. #358 is about window-width distribution, #362 is about surface-class identity.
- **vs synth #359 (M-165.A intra-codex novel-author cluster on shared `app-server` family surfaces, commit `c341b4d`)**: #359 is single-repo intra-tick author concentration on a shared surface; #362 is cross-repo inter-author cross-tick (within-window) surface-class co-occurrence. Both involve surface clustering but at different scopes.
- **vs synth #360 (M-165.B elevated-rate odd-tick attractor, commit `b250003`)**: orthogonal axes — rate periodicity vs surface coupling. M-165.B was independently falsified at Add.166, and that falsification has nothing to do with M-166.B's introduction.
- **vs synth #361 (M-166.A keystone-carrier rotation, commit `723eff0`)**: complementary same-tick observations on different repo subsets. #361 covers the codex/opencode baton pass; #362 covers the litellm/gemini-cli surface coupling. Add.166 carries both regimes.

The key claim is the one in the comparison to #357: prior synths with "surface" in their name describe **within-repo** structure (which surface gets emitted, by which author, on which tick). M-166.B is the first hypothesis that crosses repo boundaries on the surface axis itself — not "this author also pushes to that repo" (an authorship coupling) but "this surface class is showing up in two unrelated codebases at the same time" (a content coupling). That has not been claimed before in the W17 model.

## Why the marginal p-value is acceptable as a class introduction

A reasonable critique of synth #362 is that introducing a new model class on a single 5.8%-significance event is overfitting. Synth #362 anticipates this and frames the class introduction as **conditional on testability**. The argument is that the cost of failing to introduce the class is information loss — if M-166.B exists and is real, missing it now means missing the chance to test P-362.A through P-362.F over the next 14 ticks. The cost of introducing it is a documented hypothesis with explicit falsifiers, which costs essentially nothing if the predictions falsify. So the asymmetry of costs favors introduction.

This is the same hypothesis-introduction logic the W17 cadence model has used for previous low-significance class introductions (e.g., M-159.D dual-active hard-deep-dormancy cascade-synchronization at synth #350, also a low-marginal-significance introduction). The pattern is: provisional class with explicit falsifiers, then either confirmed by carry-prediction confirmations in subsequent addenda, or quietly retired when falsifiers fire. The bookkeeping discipline of carry-predictions (visible in synth #103 at addendum 161, where 8 predictions falsified or refined and 4 confirmed across Add.159–161) prevents the model from accumulating zombie classes.

## What success and failure look like for M-166.B

A confirmation arc would look like this: Add.167 or 168 surfaces a PII-redaction merge from one of {codex, opencode, goose, qwen-code} (P-362.A confirms). Add.170-ish surfaces a non-PII M-166.B coupling — say two repos shipping CVE patches in the same window (P-362.B and P-362.C both confirm). One of those coupled ticks turns out to have been preceded by a CVE disclosure within 7 days (P-362.F partial confirm). At that point M-166.B has converted from a single-event marginal-significance hypothesis into a structurally-testable class with 3-4 carry-prediction confirmations and the cadence model integrates an external-driver axis.

A falsification arc would look like this: Add.167–170 produces zero PII-class emissions from the four candidate repos (P-362.A falsifies). Add.167–180 produces zero additional M-166.B ticks of any class (P-362.B falsifies). At that point M-166.B is retired and Add.166's PII co-occurrence reverts to the marginal-significance bin: a single 5.8%-p event in 26 ticks, perfectly consistent with the iid surface-selection null model. The structural log entry would mark M-166.B as falsified at the carry-prediction roll-up against Add.180 or so.

Either outcome is informative. The class introduction has explicit acceptance criteria; the marginal p-value is acknowledged; the predictions span 14 ticks with concrete falsifiers; and the four-condition gate (≥2 repos, intra-window delta, ≤25% marginal frequency, distinct author pools) prevents the class from drifting into a tautological "any same-tick co-occurrence" formulation.

## Bottom line

W17 synthesis #362 (commit `07223e0`, addendum `b3bd455`) introduces M-166.B — the first cross-repo surface-class dependency hypothesis in the cadence model — on the basis of a 31m51s PII-redaction co-occurrence between litellm #26662 (`08d35f6b`) and gemini-cli #26153 (`2194da2b`) inside the 38m24s Add.166 capture window. The single-event Poisson p-value against the iid surface-selection null is ~5.8% (marginal). The class is provisionally introduced on the strength of six falsifiable predictions covering Add.167–Add.180, with explicit mechanism discriminator P-362.F requiring external advisory evidence. M-166.B sits orthogonal to M-166.A's same-tick carrier rotation (which operates on the codex/opencode pair, not the litellm/gemini-cli pair), and complements rather than competes with the prior surface-axis synths #357 and #359, which operated within-repo. If the carry-predictions confirm in the next two weeks, the cadence model gains an external-driver axis; if they falsify, the Add.166 PII co-occurrence reverts to a tail event of the iid null and M-166.B retires cleanly.
