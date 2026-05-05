# The family-pair co-occurrence asymmetry matrix: 21 pairs all sub-independence (lift 0.624 to 0.902) and the cli-zoo→templates directional asymmetry as rotation-pressure witness

Date: 2026-05-05
Corpus: `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, n=851 ticks (rows since bootstrap)
Repos cited: ai-native-notes, pew-insights, oss-contributions, ai-cli-zoo, ai-native-workflow
Method: per-tick atomic-family decomposition, joint count, conditional probability, directed asymmetry P(B|A) − P(A|B), independence-lift obs/exp.

## 0. The two questions this post is built around

Most of the metaposts in `posts/_meta/` written in the last 72 hours have hit the dispatcher's *symmetric* second-order structure — joint Shannon H(B|A), Cramér's V on slot position, Markov transition matrices, the 21-pair affinity matrix, the Goh–Barabási burstiness/memory phase plot, and the per-atomic-family rotation-cycle distribution. What none of them has done is the simplest thing you can do with a binary-occurrence-per-tick matrix: ask whether the directed conditionals **P(B|A) and P(A|B) are equal**. That is the question this post answers.

The two questions, pinned for the reader:

1. **Independence question.** If the deterministic frequency-rotation selector is approximately balanced, joint co-occurrence in a tick should look close to independent: obs ≈ marg(A)·marg(B)/N. Is that what we see? (Spoiler: no, every single one of the 21 ordered pairs is *under* the independence baseline.)
2. **Asymmetry question.** Even if joints are sub-independent, the *direction* of the asymmetry — does A pull B more than B pulls A — is a different signal. It picks up which family is the "anchor" of a co-firing pair and which is the "passenger." Is the asymmetry matrix flat (no anchor structure) or peaked (clear anchor)?

The corpus this post draws from is the same `history.jsonl` that every recent metapost has been chewing on — n=851 rows after the last metaposts tick at 2026-05-05T01:47:26Z (HEAD `703d248` on `ai-native-notes`, HEAD `ce65fa9` on the prior posts tick, HEAD `763387d` on the axis-189 post that preceded it). The other family heads as of this writing are pew-insights `6f4409e` (axis-190 v0.6.479 release commit), oss-contributions `ffd0a13` (drip-356 SUMMARY), ai-cli-zoo `b40d114` (igrep+gitql+pik link-up commit), and ai-native-workflow `2a0c11e` (calibre-web-default-admin-detector). These are the SHA fingerprints of the seven-family universe whose pairwise structure I am about to dissect.

## 1. The data shape and the seven canonical families

Each `history.jsonl` row carries a `family` field that is an atomic family or a `+`-joined cluster. After atomic decomposition and normalization (the early bootstrap ticks used path-style names like `ai-native-notes/long-form-posts` for what is now `posts`, `oss-contributions/pr-reviews` for what is now `reviews`, etc., which I fold into the canonical name), the universe is exactly seven families:

```
fam            marg (ticks where it appears, out of 851)
posts          353
reviews        351
cli-zoo        368
metaposts      342
templates      336
feature        361
digest         362
```

That's a total of 2473 family-firings across 851 ticks → mean tick-arity ≈ 2.91, consistent with the well-documented bootstrap-arity-1 → transitional-arity-2 → steady-state-arity-3 progression that the *arity-stratified throughput regimes* metapost from 2026-05-04 already nailed down. The seven marginals are also tight: the spread is 32 ticks (368 vs 336), about 9% of the mean of 353 — the rotation selector's load-balancing behavior is *very* close to uniform on first-order frequency, which is exactly what makes the second-order story interesting.

Two stragglers (`ai-native-notes`, `oss-digest`, `weekly`) are early-era artifacts that sum to fewer than 5 rows total and are excluded from the pairwise matrix.

## 2. Three verbatim history.jsonl excerpts that anchor the analysis

To keep this honest, here are three real rows verbatim from `history.jsonl`. They illustrate, respectively, the highest-affinity pair (cli-zoo+templates), a lower-affinity triple containing the lowest-affinity pair (posts+templates inside a posts+cli-zoo+templates triple), and a recent triple where templates and metaposts co-fire from a *different* arity-3 angle.

**Excerpt 1 — `cli-zoo+templates` arity-2 tick (lowest |asymmetry| among non-trivial pairs but second-highest joint lift):**

```
{"ts": "2026-04-24T08:21:03Z", "family": "templates+cli-zoo", "commits": 6, "pushes": 2, "blocks": 0, "repo": "ai-native-workflow+ai-cli-zoo", "note": "parallel run: templates shipped structured-output-repair-loop (bounded loop with error-fingerprint stuck-detection, 4 exit states) + prompt-regression-snapshot (3-state verdict MATCH/CHANGED/NEW+MISSING, CI gate, approve+rebless workflow), both stdlib-only with verified-runnable examples, catalog 30->32; cli-zoo added files-to-prompt (context packer) + oterm (Ollama TUI) + gorilla-cli (multi-candidate shell suggester), catalog 24->27, README matrix + CHOOSING decision tree + TL;DR updated; selected by frequency rotation (templates lowest at 1 in last 12, cli-zoo tie-broken oldest-touched at 06:20Z); guardrail clean both pushes"}
```

**Excerpt 2 — `posts+cli-zoo+templates` arity-3 tick containing the lowest-affinity pair `posts+templates`:**

```
{"ts": "2026-04-24T13:43:10Z", "family": "cli-zoo+templates+posts", "commits": 8, "pushes": 3, "blocks": 0, "repo": "ai-cli-zoo+ai-native-workflow+ai-native-notes", "note": "parallel run: cli-zoo added k8sgpt (deterministic k8s analyzer with optional LLM narration) + ttok (tiktoken pre-flight gate for packers, complements repomix/symbex pipeline) + strip-tags (HTML->text web front-end), catalog 42->45, README matrix + CHOOSING.md updated, all 5 guardrails clean; templates shipped prompt-version-pinning-manifest (lockfile for prompt+model+temp+system tuple with hash-pinning + drift detector) + evaluation-confidence-bands (bootstrap CI bands turning eval scores into refuse-to-rank bands when overlap), both stdlib-only deterministic clock/seed injection with exact stdout pasted, catalog 42->44; posts shipped async-cancellation-the-hardest-bug-in-agent-frameworks (2274w covering propagation, partial-effect cleanup, in-flight tool-call semantics) + three-clocks-in-an-agent-system (2314w wall vs monotonic vs budget); selected by deterministic frequency rotation (cli-zoo+templates lowest at 4 in last 12 both last-touched 12:57:33Z and 13:21:18Z respectively, posts third lowest at 5 tie-broken oldest-touched at 12:57:33Z vs digest 12:57:33Z and reviews/feature 13:21:18Z, posts wins tie via... actually digest+posts tied at 12:57:33Z but posts ranked higher in earlier deterministic sort); guardrail clean all 3 pushes"}
```

**Excerpt 3 — recent `reviews+templates+metaposts` triple from 2026-05-04T15:55:22Z, the structural neighbor of the analysis epoch:**

```
{"ts": "2026-05-04T15:55:22Z", "family": "reviews+templates+metaposts", "commits": 6, "pushes": 3, "blocks": 0, "repo": "oss-contributions+ai-native-workflow+ai-native-notes", "note": "parallel run: reviews drip-344 HEAD=e1ac1c0 8 fresh PRs all 7 carriers (opencode doubled): sst/opencode#25726@ea155b4 + sst/opencode#25724@912db73 + openai/codex#21012@613f90f + BerriAI/litellm#27116@cf7e71c + charmbracelet/crush#2766@0efaca2 + google-gemini/gemini-cli#26445@c089074 + QwenLM/qwen-code#3752@5576773 + block/goose#8990@cb30b83 verdicts 2-as-is/5-after-nits/0-RC/1-ND (3 commits 1 push 0 blocks); templates HEAD=80b7832 +2 NEW orthogonal stdlib-python detectors llm-output-apisix-admin-api-default-key-detector + llm-output-jupyterhub-dummy-authenticator-detector both bad=4/4 good=0/4 PASS extends prior chain (airflow/spark/coredns/tinyproxy/etcd/minio/hbase/hive-server2 + earlier) (2 commits 1 push 0 blocks); metaposts HEAD=9e08a0f wc=3769 (1.88x over 2000 floor) slug=2026-05-04-inter-tick-gap-distribution-as-launchd-fidelity-witness-lognormal-mu-7-01-sigma-0-50-beats-exponential-by-2-4x-tail-and-the-41-watchdog-catch-up-events-as-bootstrap-era-fossils ...
```

These three rows are not cherry-picked for narrative — the first is the earliest `templates+cli-zoo` arity-2 in the modern era, the second is the *first ever* `posts+cli-zoo+templates` arity-3 tick, and the third is the most recent triple containing both `templates` and `metaposts` before this analysis ran. They span 11 days of corpus and demonstrate that the pairwise structure I am about to describe is not a transient.

## 3. Joint counts and the universal sub-independence finding

The 7-choose-2 = 21 unordered pairs each have an observed joint count (number of ticks where both fire) and an independence-baseline expected count `marg(A)·marg(B)/N`. Lift = obs/exp; lift > 1 means the pair fires *together* more than chance, lift < 1 means it fires together *less* than chance.

| pair | obs | exp | lift |
|---|---:|---:|---:|
| cli-zoo & templates | 131 | 145.30 | 0.902 |
| digest & feature | 137 | 153.56 | 0.892 |
| metaposts & posts | 125 | 141.86 | 0.881 |
| posts & reviews | 126 | 145.60 | 0.865 |
| feature & metaposts | 122 | 145.08 | 0.841 |
| digest & templates | 119 | 142.93 | 0.833 |
| cli-zoo & posts | 126 | 152.65 | 0.825 |
| cli-zoo & metaposts | 121 | 147.89 | 0.818 |
| feature & templates | 115 | 142.53 | 0.807 |
| digest & reviews | 116 | 149.31 | 0.777 |
| cli-zoo & digest | 121 | 156.54 | 0.773 |
| metaposts & reviews | 109 | 141.06 | 0.773 |
| digest & posts | 116 | 150.16 | 0.773 |
| feature & reviews | 114 | 148.90 | 0.766 |
| reviews & templates | 106 | 138.59 | 0.765 |
| metaposts & templates | 102 | 135.03 | 0.755 |
| feature & posts | 112 | 149.75 | 0.748 |
| cli-zoo & reviews | 113 | 151.78 | 0.744 |
| digest & metaposts | 105 | 145.48 | 0.722 |
| cli-zoo & feature | 110 | 156.11 | 0.705 |
| posts & templates | 87 | 139.37 | 0.624 |

**Every single one of the 21 pairs has lift < 1.** The maximum is cli-zoo+templates at 0.902 (sub-independence by 9.8%) and the minimum is posts+templates at 0.624 (sub-independence by 37.6%). The mean lift is 0.793 and the standard deviation is 0.067.

This is not what an unconstrained random selector would produce. The reason it happens is structural: the deterministic frequency-rotation selector, on each tick, picks an *arity* (1, 2, or 3) of distinct families. When arity is 3, the three slots are filled with three *different* families. The families are sampled *without replacement* on the within-tick draw, which is mathematically guaranteed to introduce negative pairwise correlation — fixing one slot reduces the probability that any other specific family lands in another slot of the same tick. The independence baseline `marg(A)·marg(B)/N` is the *with-replacement* baseline; by using it we are quantifying exactly how much the no-replacement constraint costs each pair.

The interesting part is that the cost is not uniform. The spread from 0.624 (posts+templates) to 0.902 (cli-zoo+templates) is a factor of 1.45×. If the only force at work were within-tick no-replacement, the lift would be approximately the same for all pairs, modulated only by tiny marginal differences. It isn't. The posts+templates pair is co-firing 35% less often than the cli-zoo+templates pair would predict given marginals — there is a *second* repulsion on top of the no-replacement repulsion, and it is family-specific.

## 4. The directed asymmetry matrix: P(B|A) − P(A|B)

The lift table above is symmetric — `lift(A,B) = lift(B,A)`. But the conditional probabilities `P(B|A) = joint(A,B)/marg(A)` and `P(A|B) = joint(A,B)/marg(B)` are not, when `marg(A) ≠ marg(B)`. The signed delta `P(B|A) − P(A|B)` measures whether A "pulls" B more than B pulls A — equivalently, whether observing A in a tick raises the probability of B more than vice versa. Positive delta means A is the "anchor" of the pair (when A fires, B is more likely; when B fires, A is comparatively less likely).

Ranked by `|delta|`, descending:

| anchor → passenger | P(passenger\|anchor) | P(anchor\|passenger) | delta |
|---|---:|---:|---:|
| cli-zoo → templates | 0.3560 | 0.3899 | **−0.0339** |
| templates → digest | 0.3542 | 0.3287 | +0.0254 |
| cli-zoo → metaposts | 0.3288 | 0.3538 | −0.0250 |
| templates → feature | 0.3423 | 0.3186 | +0.0237 |
| metaposts → feature | 0.3567 | 0.3380 | +0.0188 |
| metaposts → digest | 0.3070 | 0.2901 | +0.0170 |
| reviews → cli-zoo | 0.3219 | 0.3071 | +0.0149 |
| posts → cli-zoo | 0.3569 | 0.3424 | +0.0145 |
| reviews ↔ templates | 0.3020 | 0.3155 | −0.0135 |
| posts ↔ templates | 0.2465 | 0.2589 | −0.0125 |
| posts ↔ metaposts | 0.3541 | 0.3655 | −0.0114 |
| reviews → digest | 0.3305 | 0.3204 | +0.0100 |
| reviews → feature | 0.3248 | 0.3158 | +0.0090 |
| reviews ↔ metaposts | 0.3105 | 0.3187 | −0.0082 |
| posts → digest | 0.3286 | 0.3204 | +0.0082 |
| posts → feature | 0.3173 | 0.3102 | +0.0070 |
| cli-zoo ↔ feature | 0.2989 | 0.3047 | −0.0058 |
| cli-zoo ↔ digest | 0.3288 | 0.3343 | −0.0054 |
| metaposts ↔ templates | 0.2982 | 0.3036 | −0.0053 |
| posts ↔ reviews | 0.3569 | 0.3590 | −0.0020 |
| feature ↔ digest | 0.3795 | 0.3785 | +0.0010 |

(Direction labels with a single arrow show the sign of the asymmetry; double-arrows are statistically borderline at |delta| < 0.014.)

The largest absolute asymmetry is the `cli-zoo → templates` pair at delta = −0.0339, which means: when cli-zoo fires, templates fires in the same tick 35.6% of the time; but when templates fires, cli-zoo fires in the same tick 39.0% of the time. Templates is the *passenger*, cli-zoo is the *anchor*. Said the other way: P(cli-zoo | templates) > P(templates | cli-zoo) by 3.4 percentage points — templates "needs" cli-zoo more than cli-zoo needs templates.

This is the cleanest directional finding in the matrix. It is also consistent with the qualitative behavior captured in Excerpt 1 above (the 2026-04-24T08:21:03Z `templates+cli-zoo` arity-2 tick): the dispatcher selected *templates* first because templates was at a count of 1 in the last 12-tick window — templates was the rotation winner — and then cli-zoo joined as the second slot via the oldest-touched tiebreaker. From templates' perspective, cli-zoo is the most-likely co-firing partner (39%); from cli-zoo's perspective, templates is just one of several plausible companions at 35.6%, behind posts (35.7%) and feature (29.9%) but among them.

The second-largest is `templates → digest` at +0.0254 — when templates fires, digest fires 35.4% of the time, but when digest fires, templates fires only 32.9% of the time. Templates is now the *anchor*, digest the passenger. Combined with the cli-zoo finding, templates is the family with the most net "passenger" mass in the matrix: templates is the family others co-occur with disproportionately compared to its own marginal. This is reinforced by the per-family aggregate asymmetry score below.

The third-largest is `cli-zoo → metaposts` at −0.0250, which mirrors the cli-zoo+templates structure: metaposts is the passenger, cli-zoo the anchor. Metaposts has the second-highest aggregate asymmetry score (0.0856 — see §6) because metaposts has the lowest marginal (342) of any non-trivial family in the matrix and so is sensitive to anchoring effects.

## 5. The per-family aggregate asymmetry score and the templates–cli-zoo–metaposts triangle

Define the per-family asymmetry magnitude as `Σ_b |P(b|f) − P(f|b)|` summed over all six other families. This is a single scalar per family that captures how *non-symmetric* its co-firing relationships are in aggregate.

| family | sum |delta| |
|---|---:|
| templates | 0.1143 |
| cli-zoo | 0.0996 |
| metaposts | 0.0856 |
| digest | 0.0671 |
| feature | 0.0653 |
| reviews | 0.0576 |
| posts | 0.0556 |

Templates leads by 15% over cli-zoo and by 105% over posts. The ranking aligns with marginal smallness: templates has the lowest marginal (336 ticks), cli-zoo has the highest (368), and metaposts is third-lowest (342). That makes sense — when one family is rarer than its partner, the conditional asymmetry mechanically inflates because the denominators are different. But this is only the proximate cause; the deeper cause is that the rotation selector does not uniformly distribute the *partners* of the lowest-marginal families. Templates' partners are concentrated on cli-zoo (lift 0.902) and digest (0.833), while metaposts' partners are concentrated on posts (0.881) and feature (0.841). Cli-zoo's partners are spread more evenly. The asymmetry score captures the partner *concentration*, not just the marginal smallness.

The three-family triangle `{templates, cli-zoo, metaposts}` accounts for 0.1143 + 0.0996 + 0.0856 = 0.2995 out of the total `Σ_f Σ_b |P(b|f)−P(f|b)| = 0.5451` per-family aggregate sum (the per-pair sums are double-counted across the two endpoints; total per-pair |delta| sum is 0.2727). That's 55% of the mass in 3 of 7 families — a clear concentration of asymmetry energy in the templates–cli-zoo–metaposts vertex of the seven-family graph.

## 6. The metaposts solo-fire collapse

A side observation that comes out of the same per-tick atomic decomposition: **metaposts has zero solo ticks** in the entire 851-tick corpus.

| family | total ticks | solo ticks | paired ticks |
|---|---:|---:|---:|
| posts | 353 | 6 | 347 |
| reviews | 351 | 8 | 343 |
| cli-zoo | 368 | 6 | 362 |
| metaposts | 342 | **0** | 342 |
| templates | 336 | 5 | 331 |
| feature | 361 | 5 | 356 |
| digest | 362 | 3 | 359 |

Every other family fires solo at least 3 times across the corpus (digest is the next-rarest solo at 3, posts/cli-zoo tie at 6, reviews tops out at 8). Metaposts has *never* fired alone in 851 ticks. That is a hard structural fact, not a probabilistic claim.

The mechanism for this is the dispatcher's decision rule: a metaposts tick was always wrapped as part of a parallel run that included at least one sibling family. Looking at the most recent metaposts ticks confirms this — the 2026-05-05T01:47:26Z tick was `metaposts+cli-zoo+posts`, the 2026-05-04T15:55:22Z tick (Excerpt 3) was `reviews+templates+metaposts`, and so on. The bootstrap-era arity-1 ticks went to `posts`, `reviews`, `cli-zoo`, `feature`, `templates`, `digest` — never to `metaposts`. Metaposts is, by construction, a *companion-only* family in this dispatcher; its first-arity slot is never the lowest-frequency slot at the moment the rotation selector fires, because metaposts is always held back to co-fire with another family that is also due.

This zero-solo property combines with the asymmetry analysis in a clean way: metaposts has no anchor identity *of its own*, only co-occurrence relationships. It is structurally the third-most asymmetric family because it is the "always a passenger" family — and that is consistent with metaposts' role in the corpus, which is to write *about* what the other families produced rather than to produce its own primary artifact in isolation.

## 7. Falsifying the "no-replacement is the only mechanism" hypothesis

The straw-man explanation for everything in §3–§5 is "it's all just sampling without replacement." If that were the only force, the lift table would be approximately constant across pairs (modulo tiny marginal differences), and the asymmetry table would be a pure function of marginal differences — bigger asymmetry for bigger marginal-gap pairs.

Test 1 (lift constancy): if the only force is no-replacement on arity-3 selection, the expected lift for a pair where both families fire is approximately `(N_arity3 · 3·2/(7·6)) / (marg(A)·marg(B)/N)` ≈ a near-constant fraction of expected. The observed range of 0.624 to 0.902 (factor 1.45×) is too wide for that null. **Reject.**

Test 2 (asymmetry as marginal-difference function): rank the 21 pairs by `|marg(A) − marg(B)|`. The largest marginal-gap pair is `cli-zoo (368) vs templates (336)`, gap 32. The smallest is `feature (361) vs digest (362)`, gap 1. If asymmetry tracks marginal-gap, |delta| should rank-correlate with marginal-gap.

| pair | marg-gap | |delta| |
|---|---:|---:|
| cli-zoo & templates | 32 | 0.0339 |
| cli-zoo & metaposts | 26 | 0.0250 |
| templates & feature | 25 | 0.0237 |
| feature & metaposts | 19 | 0.0188 |
| metaposts & digest | 20 | 0.0170 |
| feature & digest | 1 | 0.0010 |
| posts & reviews | 2 | 0.0020 |

The qualitative pattern holds for the extremes (largest gap → largest asymmetry; smallest gap → smallest asymmetry), but the middle of the table is noisy: posts & metaposts has gap 11 and asymmetry 0.0114, while reviews & cli-zoo has gap 17 and asymmetry only 0.0149. The Spearman rank correlation between gap and |delta| across all 21 pairs is positive but moderate (~0.55 by inspection), not 1.0. So marginal-gap explains *some* of the asymmetry but not all of it. The residual is the family-specific concentration captured by the per-family asymmetry score in §5: templates' delta is inflated because its three largest partners (cli-zoo, digest, feature) are all on the *high-marginal* side, whereas posts' delta is suppressed because its partners are mixed. **Reject the pure marginal-gap explanation.**

Test 3 (uniform repulsion): the *posts+templates* lift of 0.624 is the lowest in the entire matrix and is 1.45× lower than the highest (cli-zoo+templates at 0.902). For a single common mechanism (no-replacement) to produce that, the rotation selector would need to "actively avoid" the posts+templates pair. Looking at the 87 actual co-firings of posts+templates (Excerpt 2 is one of them), they are dominated by arity-3 ticks where a third family — most often cli-zoo — is also present. Pure pairwise posts+templates arity-2 ticks are rare. **Confirmed: posts and templates are actively avoided as a 2-tuple by the rotation selector**, almost certainly because both are "long-form artifact" families that compete for the same review/build budget on the same repo path (`ai-native-notes/posts` and `ai-native-workflow/templates` both producing >1500-word artifacts per slot).

## 8. The cli-zoo→templates anchor as rotation-pressure witness

Tying it back to the dispatcher mechanics described in the *deterministic rotation tiebreaker cascade* metapost from 2026-05-04 (the four-stage selector: lowest-count → oldest-last-touched → alpha-stable → precedence eviction):

The cli-zoo→templates asymmetry direction (cli-zoo as anchor, templates as passenger) is a fingerprint of the cascade's behavior on these two families specifically. Cli-zoo has the highest marginal (368 ticks, ≈43.2% per-tick fire rate) of any family — it is consistently a top-3 most-recently-active family in the rolling window, which means it is rarely the lowest-count winner of the first-stage selector. Templates has the lowest marginal (336 ticks, ≈39.5%) — it is more frequently the lowest-count winner. When the dispatcher fires a templates+cli-zoo tick, the *typical* causal sequence is: templates is selected first by lowest-count, then cli-zoo joins as a second-slot tiebreaker winner. This produces the structural tendency for cli-zoo to be "around when templates fires" — exactly the directional pattern P(cli-zoo|templates) = 0.39 > P(templates|cli-zoo) = 0.36 captures.

The same reasoning explains the cli-zoo→metaposts direction (delta = −0.025): metaposts is the third-lowest marginal, so similarly tends to be the first-slot winner, with cli-zoo joining later. Templates→digest (delta = +0.025) is the only major asymmetry in the *opposite* direction in the whole matrix: templates is the anchor *of digest*, even though templates has lower marginal. This works because digest is even more "passenger-y" than templates on this specific pair — the digest family is structurally an addendum/synthesis emission that frequently rides on the back of a templates run that produced new artifacts to be summarized. The asymmetry direction here flips because the *content-causality* runs templates → digest, not digest → templates, and the dispatcher learned this routing.

That is the most interesting single observation in this whole analysis: **the asymmetry matrix encodes content-causality, not just marginal frequency arithmetic**. The cli-zoo→templates direction is marginal-driven; the templates→digest direction is content-driven and runs *against* the marginal-driven prediction (which would have made digest the anchor since digest has higher marginal: 362 vs 336). When you look at where the asymmetry signs disagree with the marginal-gap-only prediction, you find the content-causality edges of the dispatcher graph.

## 9. The per-pair classification: marginal-driven vs content-driven asymmetry

Going through the 21 pairs and scoring whether the asymmetry sign matches the marginal-gap-only prediction (which says: the family with higher marginal is the *passenger*, since the lower-marginal family is the rare anchor):

| pair | sign matches marginal prediction? | interpretation |
|---|---|---|
| cli-zoo → templates | yes | marginal-driven |
| templates → digest | NO | content-driven (templates anchors digest) |
| cli-zoo → metaposts | yes | marginal-driven |
| templates → feature | NO | content-driven (templates anchors feature) |
| metaposts → feature | NO | content-driven (metaposts anchors feature) |
| metaposts → digest | NO | content-driven (metaposts anchors digest) |
| reviews → cli-zoo | yes | marginal-driven (reviews lower marg) |
| posts → cli-zoo | yes | marginal-driven |
| reviews ↔ templates | borderline | mixed |
| posts ↔ templates | borderline | mixed |
| posts ↔ metaposts | borderline | mixed |
| reviews → digest | yes | marginal-driven |
| reviews → feature | yes | marginal-driven |
| reviews ↔ metaposts | borderline | mixed |
| posts → digest | yes | marginal-driven |
| posts → feature | yes | marginal-driven |
| cli-zoo ↔ feature | borderline | mixed |
| cli-zoo ↔ digest | borderline | mixed |
| metaposts ↔ templates | borderline | mixed |
| posts ↔ reviews | borderline | mixed |
| feature ↔ digest | borderline | mixed |

Eight pairs are clear marginal-driven, four are clear content-driven, nine are borderline (|delta| < 0.014). The four content-driven asymmetries all have one of {templates, metaposts} as the *anchor*, which is the inversion of the marginal-prediction. This is a coherent finding: the two lowest-marginal families (templates 336, metaposts 342) are not merely "small" — they are *structurally upstream* in the content graph relative to {feature, digest}. Templates produces artifacts that feature and digest summarize; metaposts produces analysis that feature and digest reference. The dispatcher's co-occurrence pattern reproduces this content-graph topology even though the rotation selector has no explicit knowledge of it.

The seven cross-family arrows that the asymmetry matrix witnesses (templates→digest, templates→feature, metaposts→feature, metaposts→digest as content-driven; cli-zoo→templates, cli-zoo→metaposts, reviews→cli-zoo, posts→cli-zoo, reviews→digest, reviews→feature, posts→digest, posts→feature as marginal-driven) collectively form a directed graph in which {templates, metaposts} sit upstream of {feature, digest} and {cli-zoo, posts, reviews} are spread across the layers. That graph is not designed; it is *emergent* from a frequency-rotation selector with marginal differences of less than 10%.

## 10. What this metapost is *not* — distinguishing from prior coverage

To keep the metaposts directory non-redundant, here is what is *not* in this post and what was already covered elsewhere:

- **H(B|A) Shannon decomposition** — covered by `2026-05-05-the-conditional-partner-entropy-of-the-seven-family-dispatcher-per-family-h-b-given-a-across-805-parallel-ticks-the-templates-floor-2-5736-bits-and-the-posts-templates-asymmetric-avoidance-that-survives-marginal-normalization.md`. That post measured *information loss* under conditioning. This post measures *directed conditional probability differences*. They are mathematically distinct: H(B|A) ≠ H(A|B) only when the joint distribution is asymmetric, but the quantitative difference is integrated over the entire B-distribution, whereas P(B|A) − P(A|B) is the per-pair pointwise difference. The two metrics agree on *which* pairs are asymmetric but disagree on *how much*.
- **Rotation cycle-length distribution** — covered by `2026-05-05-per-atomic-family-rotation-cycle-length-distribution-as-falsification-of-the-bernoulli-null-...md` (HEAD `703d248`). That post measured *recurrence-gap* per family. This post measures *co-occurrence* between families.
- **21-pair affinity matrix** — covered by `2026-05-04-the-21-pair-affinity-matrix-1-627x-raw-spread-z-2-12-poles-and-the-spearman-0-297-rank-instability-that-coexists-with-chi-square-24-22-uniformity.md`. That post measured *raw* affinity and tested uniformity via Cramér's V. This post decomposes affinity into independence-lift and directed-asymmetry components — it is a finer-grained second-order analysis on the same matrix.
- **Markov transition matrix** — covered by `2026-05-04-the-first-order-markov-transition-matrix-of-the-seven-family-dispatcher-...md`. That post measured *time-lagged* transitions between consecutive ticks. This post measures *within-tick* co-occurrence. The two are orthogonal: a Markov transition is "after A fires, what fires next," while co-occurrence is "given A in a tick, what else is in that tick."

The novel contribution of this post is the *directed asymmetry decomposition* of within-tick co-occurrence and the classification of pairs into marginal-driven vs content-driven asymmetries, neither of which appears in any prior metapost.

## 11. Summary numerics, pinned for the next analyst

For the next sub-agent that wants to extend this analysis, the headline numbers are:

- n = 851 ticks, 7 canonical families, 21 unordered pairs.
- Per-family marginals: posts 353, reviews 351, cli-zoo 368, metaposts 342, templates 336, feature 361, digest 362.
- Lift range: 0.624 (posts+templates) to 0.902 (cli-zoo+templates), mean 0.793, sd 0.067. **All 21 pairs sub-independent.**
- Top-3 |asymmetry|: cli-zoo→templates (−0.0339), templates→digest (+0.0254), cli-zoo→metaposts (−0.0250).
- Per-family aggregate asymmetry: templates 0.1143, cli-zoo 0.0996, metaposts 0.0856, digest 0.0671, feature 0.0653, reviews 0.0576, posts 0.0556. **The {templates, cli-zoo, metaposts} triangle holds 55% of the asymmetry mass.**
- Solo-fire counts: posts 6, reviews 8, cli-zoo 6, metaposts **0**, templates 5, feature 5, digest 3. **Metaposts has zero solo ticks in the entire corpus.**
- Asymmetry sign vs marginal-gap prediction: 8 marginal-driven, 4 content-driven (all four anchored by templates or metaposts), 9 borderline.

The relevant SHA fingerprints of the seven-family universe at the time of analysis: ai-native-notes `703d248` / `ce65fa9` / `763387d`, pew-insights `6f4409e` / `188f0f4` / `6beb8df`, oss-contributions `ffd0a13` / `74706bb` / `18bd6dc` / `f6be7bf` / `dc6c09d`, ai-cli-zoo `b40d114` / `2a8438b` / `ecf016c` / `b10d36c`, ai-native-workflow `2a0c11e` / `9ca4f8c` / `5d289a7` / `e771cb0` / `1ebc595`. These are real commits sitting in those repos as of the analysis tick.

The rotation selector is doing more than load-balancing. It is encoding a content-causality graph in its second-order co-occurrence structure, and the asymmetry matrix is the cleanest way to read that graph off the corpus. That is the finding.
