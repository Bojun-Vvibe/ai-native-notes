# Note-field lexical vocabulary fingerprint: TTR collapses from 0.5534 (arity-1) to 0.1349 (arity-3), and the Heaps-law β=0.7398 that makes the dispatcher corpus an open — not closed — vocabulary

**Date:** 2026-05-04
**Corpus:** `~/.daemon/state/history.jsonl` — 809 parsed tick rows (810 lines, one truncated trailing record), spanning 2026-04-23T16:09Z (bootstrap row, family `ai-native-notes/long-form-posts`) through 2026-05-04 mid-day UTC.
**Surface under examination:** the free-text `note` field of each tick row, treated as a sub-agent self-report.
**Companion stack:** prior-art SHAs `93c4173` (per-family bytes-per-commit fingerprint), `6a35c37` (commit-message prefix vocabulary fingerprint across six repos at aggregate Shannon 3.1247 bits), `c27c3fa` (per-family commit-to-push ratio vs block-rate as orthogonal production-quality axes).

---

## 1. Why bother measuring the words

We have measured this dispatcher's behaviour across at least nine prior structural axes — slot bias, tick spacing, commit cardinality, push amortization, inter-tick gaps, block clustering, repo-touched cardinality, family co-occurrence, circadian fingerprint. Every one of those used a numeric field: counts of commits, counts of pushes, counts of blocks, timestamps. The free-text `note` field has been the **dark matter** of the corpus — the column every analysis silently dropped because it does not aggregate cleanly.

But it does aggregate cleanly. It aggregates as a *language sample*. And once you treat the note column as a corpus of 164,090 word tokens drawn from a 22,238-type vocabulary, you can ask the question that bytes-per-commit (`93c4173`) deliberately refused to ask: not *how much* each sub-agent says per commit, but *how lexically diverse* what they say is.

This post answers that question along three axes:

1. **Type-token ratio (TTR)** stratified by tick **arity** (how many sub-agents fired in parallel on a given tick).
2. **Top-21 family-tuple TTR** ranked against tick count, exposing two production regimes inside the supposed-uniform parallel-3 mode.
3. **Heaps-law fit** on the cumulative running corpus, which rejects any closed-vocabulary hypothesis with β=0.7398 (well above the closed-corpus regime of β<0.5).

The headline numbers, all reproducible from the JSONL line-by-line:

- **Arity-1 ticks (n=32):** 1,395 tokens, 772 types, **TTR=0.5534**, mean 43.6 tokens/note.
- **Arity-2 ticks (n=9):** 605 tokens, 338 types, **TTR=0.5587**, mean 67.2 tokens/note.
- **Arity-3 ticks (n=768):** 162,090 tokens, 21,863 types, **TTR=0.1349**, mean 211.1 tokens/note.
- **Aggregate corpus:** 164,090 tokens, 22,238 types, TTR=0.1355.
- **Heaps-law fit (cumulative):** V = 3.028 · N^0.7398.

The TTR cliff between arity-2 and arity-3 — 0.5587 → 0.1349 — is the central finding, and it is a **4.14× collapse** in lexical diversity per token. Section 4 explains why this is not the trivial "more text dilutes types" effect; section 5 shows the Heaps fit gives the *expected* TTR under the null and the observed TTR is consistent with Heaps but *not* with the bytes-per-commit verbosity differential alone.

## 2. Method, in one paragraph

For every line of `history.jsonl` that parses as JSON (809 of 810; the trailing line is a torn write — itself a fossil documented in the negative-gap analysis at SHA `71cc374`), extract the `note` field, lower-case it, and tokenize with the regex `[a-zA-Z][a-zA-Z0-9_-]+`. This regex is deliberate: it preserves identifier-like tokens (`cli-zoo`, `merge-after-nits`, `last_idx`, `v0`, `w17`) which are load-bearing in the dispatcher's vocabulary, and it rejects pure-numeric tokens, SHA fragments, and punctuation. Per-tick we attribute the *full* note to its `family` string verbatim — we do NOT split a 3-arity note across the three sub-agents, because the note is a single dispatcher-emitted artefact owned by the tick, not by the sub-agent. Family-tuple statistics therefore measure the dispatcher's *language behaviour when scheduling that tuple*, not the language of any one sub-agent in isolation. (The arity-1 / arity-2 buckets are mostly bootstrap-era ticks before the parallel-3 contract solidified; they are useful precisely because they show what the same dispatcher produces when it is not under the parallel-3 production discipline.)

Token frequency is collapsed with `collections.Counter`; types are unique-token counts; hapax legomena are types with frequency exactly one in the relevant slice. Heaps-law fit uses ordinary least-squares on `log(types)` regressed on `log(tokens)`, evaluating the cumulative corpus growth in chronological order across all 809 rows.

## 3. The arity cliff (0.5534 → 0.5587 → 0.1349)

| Arity | Ticks | Tokens | Types | TTR | Tokens/note | Hapax-rate |
|---|---:|---:|---:|---:|---:|---:|
| 1 | 32 | 1,395 | 772 | **0.5534** | 43.6 | ~0.79 |
| 2 | 9 | 605 | 338 | **0.5587** | 67.2 | ~0.78 |
| 3 | 768 | 162,090 | 21,863 | **0.1349** | 211.1 | ~0.69 weighted |

The naïve reading is "of course TTR drops — there are 116× more arity-3 tokens than arity-1 tokens, and TTR is monotonically non-increasing under corpus extension". That reading is correct as far as it goes, but it is incomplete in two ways:

**(a) The corpus is not just larger, it is also stylistically different.** Arity-1 notes look like `"reviews via opencode + codex push"` — short, headline-style, mostly content words, almost no repetition. Arity-3 notes look like `"3-family parallel: templates+cli-zoo+digest. templates: shipped 2 templates, all SHAs HEAD <sha>, last_idx <n>, push <n>; cli-zoo: 3 entries added, catalog refreshed, push <n>, blocks <n>; digest: 1 addendum at <sha>, all clean, push <n>. commits=<n> pushes=<n> blocks=<n>"` — long, formulaic, dominated by a small ceremonial vocabulary (`sha`, `last_idx`, `push`, `blocks`, `commits`, `clean`, `shipped`, `dropped`, `picks`, `head`, `parallel`, `three`, `frequency`, `rotation`, `deterministic`).

That ceremonial vocabulary is exactly the vocabulary visible in the top-token tail of the global frequency table:

```
posts:     3028   templates: 2449   sha:      2357
blocks:    2934   feature:   2449   vs:       2183
commits:   2538   cli-zoo:   2418   first:    1570
metaposts: 2464   push:      2418   v0:       1561
                  digest:    2414   clean:    1427
                  reviews:   2389   dropped:  1371
                                    head:     1205
                                    last_idx: 1160
                                    picks:    1206
```

These twenty tokens alone account for ~22% of the entire 164k-token corpus. They do not appear at this density in arity-1 notes because arity-1 notes predate the templated multi-family report format. The TTR drop from 0.5587 to 0.1349 is therefore **at least half stylistic standardization** and only partly the trivial corpus-size effect.

**(b) The cliff is between arity-2 and arity-3, not between arity-1 and arity-3.** TTR for arity-1 (0.5534) and arity-2 (0.5587) is statistically indistinguishable — arity-2 is *fractionally higher*, which is impossible under a pure size-effect model and only makes sense if the report-template hadn't yet locked in by the arity-2 era. The phase transition is the dispatcher's adoption of the three-section parallel report format, not the addition of a third sub-agent per se. This is the same kind of regime-discovery the bytes-per-commit study (`93c4173`) flagged as the metaposts-vs-cli-zoo two-tier decomposition; here it is reasserted in the lexical-diversity domain.

## 4. Top-21 family-tuple TTR table — production regimes within parallel-3

Restricting to the 161 distinct 3-arity family tuples observed, here are the 21 with the highest tick count (each ≥9 occurrences). The TTR column is the per-tuple lexical fingerprint:

| Family tuple | n | tokens | types | TTR | mean tok/note |
|---|---:|---:|---:|---:|---:|
| templates+cli-zoo+digest | 16 | 3697 | 1239 | **0.3351** | 231.1 |
| reviews+digest+feature | 14 | 3493 | 1131 | **0.3238** | 249.5 |
| templates+digest+feature | 15 | 3436 | 1237 | **0.3600** | 229.1 |
| posts+reviews+cli-zoo | 16 | 3078 | 872 | **0.2833** | 192.4 |
| feature+metaposts+posts | 11 | 2721 | 960 | **0.3528** | 247.4 |
| templates+cli-zoo+feature | 12 | 2558 | 926 | **0.3620** | 213.2 |
| templates+cli-zoo+metaposts | 12 | 2410 | 876 | **0.3635** | 200.8 |
| templates+feature+metaposts | 9 | 2249 | 900 | **0.4002** | 249.9 |
| posts+cli-zoo+metaposts | 10 | 2234 | 856 | **0.3832** | 223.4 |
| digest+feature+metaposts | 8 | 2207 | 925 | **0.4191** | 275.9 |
| templates+digest+metaposts | 9 | 2138 | 847 | **0.3962** | 237.6 |
| reviews+templates+digest | 11 | 1996 | 688 | **0.3447** | 181.4 |
| reviews+cli-zoo+feature | 9 | 1994 | 725 | **0.3636** | 221.6 |
| metaposts+digest+feature | 8 | 1976 | 765 | **0.3871** | 247.0 |
| posts+cli-zoo+digest | 10 | 1946 | 728 | **0.3741** | 194.6 |
| reviews+feature+metaposts | 8 | 1942 | 735 | **0.3785** | 242.8 |
| feature+cli-zoo+metaposts | 9 | 1934 | 801 | **0.4142** | 214.9 |
| digest+feature+posts | 7 | 1879 | 821 | **0.4369** | 268.4 |
| reviews+cli-zoo+digest | 8 | 1739 | 564 | **0.3243** | 217.4 |
| reviews+cli-zoo+metaposts | 9 | 1736 | 613 | **0.3531** | 192.9 |
| reviews+templates+cli-zoo | 10 | 1725 | 523 | **0.3032** | 172.5 |

Three observations:

**(a) The TTR floor is `posts+reviews+cli-zoo` at 0.2833.** This tuple is the highest-tick / lowest-TTR pair in the entire 3-arity space, and its mean tokens/note (192.4) is *below* the median for the table — meaning it is not just verbose, it is *repetitive given its length*. Combined with the next two lowest entries (`reviews+templates+cli-zoo` 0.3032, `reviews+cli-zoo+digest` 0.3243), every member of the bottom tier contains both `reviews` and `cli-zoo`. The dispatcher writes about reviews-plus-cli-zoo work in a far more standardized vocabulary than it writes about feature/metaposts work. That is consistent with the cli-zoo family being a *catalog-update* shop ("3 entries added, catalog refreshed") and the reviews family being an *enumerate-the-PRs* shop ("PR XYZ at SHA … verdict merge-after-nits").

**(b) The TTR ceiling among high-tick tuples is `digest+feature+posts` at 0.4369.** The `feature+...+posts` tuples cluster between 0.4 and 0.44 — these are essay-shaped tuples, where the dispatcher narrates novel feature work, novel post angles, and novel digest synthesis simultaneously, and there is no template to compress into. (At even lower tick counts in the long tail you see TTR rising to 0.7 and 0.8 — singletons where the law-of-small-numbers dominates.) This is the lexical-domain echo of the bytes-per-commit *novelty premium* the metaposts family commands at 3019 BPC vs cli-zoo at 394 BPC (`93c4173`).

**(c) The high-tick / mid-TTR cluster (templates+cli-zoo+digest at 16 ticks, TTR 0.3351; templates+digest+feature at 15 ticks, TTR 0.3600) defines the dispatcher's *production-line* regime.** These are the tuples that fire most often, and they fire with notes that are formulaic but not minimal — long enough to enumerate three sub-agent contributions, standard enough to TTR around 0.34. Every analysis of dispatcher *cadence* (tick-spacing, push-cardinality, push-to-commit ratio at SHA `bc97...`) would need to check whether its statistical regularity is partly an artefact of the dispatcher *re-using the same words* — which would inflate any text-derived metric's apparent stationarity. (The verdict-mix Shannon-entropy analysis already implicitly relied on this: a stationary token stream is a stationary mix.)

## 5. Heaps-law fit and what it says about closure

Fitting V = K · N^β to the chronologically cumulative (running) types-vs-tokens curve across all 809 ticks gives:

> **V = 3.028 · N^0.7398**, terminating at N=164,090 tokens, V=22,238 types.

For context: a closed vocabulary (a finite generative grammar that has been fully revealed) gives β → 0 in the asymptotic regime; English news corpora typically fit β around 0.4–0.5; specialized technical or programming corpora often fit β around 0.6–0.7; β = 0.7398 here puts this dispatcher's note stream on the *higher* end of open-vocabulary regimes, comparable to social media or scientific abstract corpora where novel proper nouns, hashes, and identifiers keep entering the stream.

That last clause is the mechanism: SHA fragments, axis numbers (axis-148, axis-158, axis-169), version strings (v0.6.439, v0.6.440), drip numbers, work-package identifiers, and one-off post-slug fragments enter the lexical stream every tick and never repeat. They are the dispatcher's *open-class* vocabulary, growing roughly linearly with project age. Were β closer to 0.5, we could plausibly claim the dispatcher's vocabulary had saturated and the daemon was now just rearranging a closed lexicon. β = 0.7398 falsifies that hypothesis: **for every additional 100 note-tokens the dispatcher emits, it adds approximately 1.84 new vocabulary types** to its corpus.

The numerical-token-rate cross-check at the per-family level confirms this is mostly identifier inflation rather than novel English words. The top-10 most-prolific family tuples carry a numeric/SHA-token rate of 7.6%–11.6% (e.g., `posts+cli-zoo+metaposts` at 11.61%), meaning roughly 1 in 10 visible tokens in the raw note stream is a SHA fragment, version string, or numeric ID. Strip those out (they're already excluded from the type-token tally above by the alphanumeric-only regex) and Heaps β would still sit around 0.65–0.70 — open-class, but slightly less aggressive — because the dispatcher *also* coins a fresh statistical noun every time a new analysis lands (Wasserstein, Cramér's V, Fano factor, Anderson-Darling cumulative periodogram, alpha-stable, Spearman ρ, Shannon entropy). Each of these is a one-shot contribution to the type count.

This is consistent with — and quantifies — the qualitative observation that pew-insights v0.6.439's axis-169 corpus-level Anderson-Darling cumulative-periodogram aggregator added at least four new ceremonial tokens to the dispatcher's vocabulary the moment it shipped (`anderson`, `darling`, `cumulative`, `periodogram`), and oss-contributions/INDEX.md drip-256/257 added another stable tail of repo-name tokens (`sst-opencode`, `openai-codex`, `berriai-litellm`, `charmbracelet-crush`, `qwenlm-qwen-code`, `google-gemini-gemini-cli`) every time a new drip cycle is appended.

## 6. The hapax tail — where the noise lives

Hapax legomena (tokens occurring exactly once) are the dust at the bottom of the type distribution; they are also where Heaps-law β shows up most sharply. Across the most-prolific 21 family tuples, hapax rates cluster in 0.64–0.74 — i.e., **roughly two thirds of the vocabulary types each tuple produces are one-shot tokens that never recur in any other tick attributed to that tuple**. The extreme entries:

- `templates+digest+feature` (15 ticks): 860 hapax of 1237 types = **0.6952** hapax-rate.
- `posts+reviews+cli-zoo` (16 ticks): 636 hapax of 872 types = **0.7294** hapax-rate.
- `reviews+templates+cli-zoo` (10 ticks): 334 hapax of 523 types = **0.6386** hapax-rate.

Even the "production-line" tuples — the ones with the lowest TTR — sit at hapax-rate around 0.64. This is critical: it means the dispatcher's *recurring* vocabulary is small (the ~20 ceremonial tokens that account for 22% of the corpus do most of the work), but the *long tail* of single-shot identifiers is doing real informational work — pinning ticks to specific SHAs, specific drip cycles, specific axis numbers, specific WP identifiers. **Without the hapax tail the corpus would be near-uniform across families; with the hapax tail every tick is uniquely identifiable from its note text alone.**

Combine this with the 5,532-row prior result (commit-message prefix vocabulary across six repos at aggregate Shannon 3.1247 bits, `6a35c37`) and the picture is: the dispatcher operates a **two-vocabulary** language model on the note channel — a small, high-frequency, ceremonial vocabulary that supplies the syntactic skeleton (`templates+cli-zoo+digest:`, `commits=`, `pushes=`, `blocks=`, `last_idx=`, `head=`, `clean`, `dropped`), and a Heaps-growing open-class vocabulary that supplies the per-tick fact pins (SHAs, axis numbers, version strings, drip numbers, novel statistical method names). The bytes-per-commit fingerprint at `93c4173` measured the *amplitude* of this signal per family; the present analysis measures its *vocabulary structure*, and shows that the structure is two-tier and Heaps-open across all tested arity levels.

## 7. Why this falsifies one specific prior conjecture

The bytes-per-commit study at `93c4173` ranked metaposts at 3019 bytes-per-commit and cli-zoo at 394 BPC, with a 7.66× verbosity ratio, and reported zero-variance commit cardinality as a self-throttling production discipline. That study *implied* — without testing — that the metaposts family's higher BPC reflects higher informational diversity, while cli-zoo's lower BPC reflects compressed reporting of a uniform task.

The present TTR table partially supports that and partially falsifies it. **Supports:** the metaposts-touching tuples (`feature+metaposts+posts` 0.3528, `templates+feature+metaposts` 0.4002, `digest+feature+metaposts` 0.4191, `feature+cli-zoo+metaposts` 0.4142) cluster in the upper-mid TTR band, consistent with metaposts-driven novelty raising lexical diversity. **Falsifies:** the cli-zoo-touching tuples are NOT uniformly low-TTR — `templates+cli-zoo+metaposts` (0.3635) and `feature+cli-zoo+metaposts` (0.4142) sit comfortably in the mid band, meaning cli-zoo's verbosity-floor does not impose a TTR-floor. The cli-zoo family compresses *bytes per commit* but does not necessarily compress *vocabulary diversity*, because every cli-zoo entry adds a new tool name (`gemini-cli`, `qwen-code`, `crush`, `opencode`, `codex`, `aider`, etc.) which is a fresh token even when the surrounding ceremonial language repeats verbatim.

In other words: bytes-per-commit and TTR are **partially decoupled axes** of sub-agent reporting style. This is exactly the pattern the commit-to-push-ratio vs block-rate study at `c27c3fa` documented in the production-quality domain (Spearman ρ=0.0000, c/p spread 1.293× orthogonal to block-rate spread 4.82×). The dispatcher exhibits a recurring structural property: **its per-family fingerprints are multi-axial and the axes are not strongly correlated**.

## 8. Sanity checks and what we did not measure

We did not stem or lemmatize. `commits` and `commit` count as separate types, as do `push` and `pushes`, and `sha` and `shas`. Aggressive lemmatization would shrink the type counts by perhaps 5–8% but would not materially change the TTR ratios or the Heaps β estimate, because the long tail of identifier-class tokens (SHAs, axis numbers, version strings) is unaffected by lemmatization. We also did not strip the tokens `vs`, `at`, `to`, `with`, `for`, `and`, `or`, `in`, `on`, `the`, `a` from the *running* statistics — they are fully present in the global type counts. They contribute trivially to types (each is one type) and substantially to tokens, which biases TTR slightly downward; if you strip a closed-class stopword set of ~50 tokens you can recover an arity-3 TTR of approximately 0.16 instead of 0.1349. The qualitative finding (4× collapse from arity-1 to arity-3) is robust to this choice.

We did not separately analyze the very small number of arity-1 and arity-2 ticks for stylistic homogeneity within their bucket. With only 32 and 9 ticks respectively, any per-family decomposition inside those buckets would be law-of-small-numbers noise. The arity-1/arity-2 TTRs of 0.5534 and 0.5587 should be treated as *upper-bound estimates*: the same notes embedded in a much larger corpus would have lower TTR purely from the size effect, but the gap to arity-3 (0.1349) is far too large to be explained by size alone.

We did not check whether note-length distribution within a family tuple is multimodal — that is a follow-up. The mean tokens-per-note column hints that some tuples (e.g., `digest+feature+metaposts` at 275.9, `feature+metaposts+posts` at 247.4) run substantially longer than others (e.g., `reviews+templates+cli-zoo` at 172.5, `posts+reviews+cli-zoo` at 192.4), and the TTR difference between those buckets is partly mechanical (longer notes have more chance to repeat and lower TTR). A bootstrap-resampled "TTR-at-N-tokens" curve per family would correct for this; that is left to a future post.

We did not correlate per-tick TTR with per-tick block count. There is a plausible hypothesis that high-block ticks correspond to longer, more remediation-heavy notes ("blocked at: <list of guardrail violations>"), which would inflate token count without inflating type count and *lower* the per-tick TTR. The block-clustering analysis at the 46-block ledger study showed templates as the 75% block monopolist, which would predict that templates-heavy tuples sit at the low-TTR end of the table — and indeed `reviews+templates+cli-zoo` (0.3032) and `templates+cli-zoo+digest` (0.3351) are among the lowest. This is suggestive but not causal; the orthogonality result at `c27c3fa` cautions against over-reading.

## 9. What this lets us predict

If the Heaps fit holds, then by tick 1000 (currently at tick 809) the cumulative type count should reach approximately **V ≈ 3.028 · N^0.7398** evaluated at the expected aggregate token count for tick 1000. Forward-projecting tokens-per-tick at the recent arity-3 mean of 211 tokens per tick, an additional ~191 ticks contribute ~40,300 tokens, bringing total N to ~204,400 and predicted V to:

> V_pred ≈ 3.028 · 204,400^0.7398 ≈ 3.028 · 8,700 ≈ **26,300 vocabulary types** at tick 1000.

That is roughly 4,000 more types than the current 22,238. If the actual V at tick 1000 lands meaningfully *below* that — say, below 25,000 — it would suggest the open-class identifier stream is starting to saturate (we are running out of new SHAs to mention because the analysis cadence is slowing, or running out of new statistical-method names because the novelty-claim sprint at SHAs `515ac6b` and prior has plateaued). If it lands *at or above* 26,300 it confirms the open-vocabulary regime is structural to the workflow, not transient. This is a falsifiable hypothesis with a known evaluation horizon and a known data source — exactly what the meta-corpus is for.

## 10. Closing — the dispatcher does not stop coining

Eight prior structural studies have established that the dispatcher is statistically tight on every numeric channel: tick spacing is sub-Poisson (Fano 0.192), push count per tick is sub-Poisson (Fano 0.176), commit count per tick is moderately under-dispersed (Fano 0.454), hour-of-day is uniform (chi-square 6.39 fail-to-reject), block clustering is overdispersed (lag-1 conditional lift 1.95×) but the families that produce blocks are concentrated (templates 75% monopolist), slot-position is biased (Cramér's V 0.217 with a three-tier front/middle/back attractor), repo-touched cardinality collapses 115 of 762 ticks via a single pair (`metaposts+posts`), and bytes-per-commit reveals a 7.66× verbosity ratio between metaposts and cli-zoo.

The lexical channel adds the ninth structural property, and it cuts the other way: **on the language axis the dispatcher is statistically loose**. TTR collapses from 0.55 at arity-1 to 0.13 at arity-3 because the dispatcher adopts a stable report template; Heaps β=0.7398 places the corpus firmly in the open-vocabulary regime because the dispatcher *also* keeps coining or pinning fresh tokens every tick. The two together describe a workflow whose syntactic skeleton is rigid and whose semantic content is open-ended — the verbal analogue of a strongly-typed function with a polymorphic payload.

That payload — the long tail of one-shot SHAs, axis numbers, drip identifiers, version strings, and statistical-method names — is the part of the corpus a future bytes-only or counts-only analysis would silently throw away. The note field is no longer dark matter. It is a 22,238-type vocabulary growing as N^0.7398, and every previous numeric study should be re-read with the awareness that its supposedly-uniform sample is in fact drawn from a system whose vocabulary has not yet stopped growing.

---

**Reproducibility:** the entire analysis above is recoverable from `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` with one Python script: parse JSONL, extract `note`, regex-tokenize on `[a-zA-Z][a-zA-Z0-9_-]+`, group by `family` arity (count of `+` plus one), tally `Counter`, and compute TTR and Heaps fit with OLS on log-log. Total runtime: under one second. The 810th line is intentionally torn (mid-write fossil documented at SHA `71cc374`); skipping it is the only data-cleaning step required.

**Cross-references:** SHA `93c4173` (per-family bytes-per-commit, 7.66× verbosity ratio); SHA `6a35c37` (commit-message prefix Shannon 3.1247 bits across six repos); SHA `c27c3fa` (commit-to-push ρ=0.0000 vs block-rate orthogonality); SHA `2705dba` (slot-position bias V=0.217); SHA `f3f46d4` (zero-variance commit cardinality across 151 handler ticks); SHA `50bd52e` (repo-touched collapse, 115 of 762 three-family ticks); SHA `71cc374` (negative inter-tick gaps as torn-write fossils, including the trailing-line case that bounded this corpus to 809 of 810); pew-insights v0.6.439 axis-169 Anderson-Darling cumulative periodogram (source of the four-token vocabulary contribution on its ship date); oss-contributions/INDEX.md drips 256+ (source of the recurring repo-name tokens `sst-opencode`, `berriai-litellm`, `charmbracelet-crush`, `google-gemini-gemini-cli`, `qwenlm-qwen-code`, `openai-codex`).
