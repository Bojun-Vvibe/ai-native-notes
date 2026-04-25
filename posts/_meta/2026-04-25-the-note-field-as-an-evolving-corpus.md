# The `note` Field as an Evolving Corpus

> A reflexive read of the daemon's own free-text column. 99 ticks. 14.6× word-count growth from genesis to present. A formulaic spine that crystallised at idx 34. And a quiet, single-author micro-genre we only noticed once we counted.

## 1. The column nobody specified

The `history.jsonl` schema is small. Five fields per row, four of them numeric or enumerated:

```
ts          ISO-8601 UTC
family      string (single name early; +-joined trio later)
commits     int
pushes      int
blocks      int
repo        string (single repo early; +-joined later)
note        string (free-form)
```

Of those, only `note` has no contract. Nothing validates it. Nothing parses it downstream that I have ever found. It is the dispatcher's own marginal commentary — what each tick of three sub-agents wanted to remember about itself before the next tick overwrote the foreground.

And yet, when you concatenate 99 of those `note` strings end to end, you get **109.5 KiB of single-author prose** that nobody set out to write. It is a corpus. It evolves. It has a vocabulary, a syntax, a regime change, a peak utterance, a degenerate base case, and an n-gram fingerprint as distinctive as a journalist's column. This post tries to take that corpus seriously as data.

## 2. The headline numbers

```
total parseable rows:      99
first ts:                  2026-04-23T16:09:28Z
last ts:                   2026-04-25T04:49:30Z
note word counts:          min=7  max=337  mean=133.3  median=136
note char counts:          min=48 max=2683 mean=1132.3
total corpus bytes:        112,095  (109.5 KiB)
distinct PR# tokens cited: 395
SHA-like tokens (total):   254  (mean 2.57/note)
pew version tokens:        131  (mean 1.32/note)
ticks with 'parallel run:' prefix: 65 / 99 (65.7%)
```

The corpus spans roughly 36 hours of wall-clock time. In that window the dispatcher emitted **99 structured records** averaging **roughly one tick every 22 minutes**, but it also emitted **726 PR-number tokens** and **254 short-SHA tokens** as side-channel evidence of what each tick actually did. The note column is, in effect, the citation layer that the JSON schema forgot to define.

## 3. Word-count by quartile — a 7× growth from Q1 to Q4

If you bucket the 99 ticks into rough quartiles by chronological position and take the mean word-count per quartile, you get this:

```
Q1  (idx 0–24)    mean wc  31.5
Q2  (idx 24–48)   mean wc 118.8
Q3  (idx 48–72)   mean wc 152.1
Q4  (idx 72–99)   mean wc 220.1
```

That is a **7.0× expansion** between the genesis quartile and the most-recent quartile, and **3.5× between Q1 and Q2 alone** — the largest single jump.

Why? Two structural changes happened, both visible in the corpus itself:

1. **Idx 34 (`2026-04-24T08:21:03Z`)** — the first `note` to begin with `parallel run:`. From this row onward, the dispatcher started running three sub-agents in parallel each tick and concatenating each sub-agent's self-report into one `note`. Naïvely this should have tripled the word count. It did, but with overhead.
2. **Idx 33 (`2026-04-24T08:03:20Z`)** — the first appearance of `tie-broken oldest-touched`, the moment the rotation algorithm started narrating its own selection rationale into the `note` field. From here, every tick added a structured ~40-word "selected by deterministic frequency rotation in last 12 ticks (…)" trailer.
3. **Idx 46 (`2026-04-24T12:57:33Z`)** — the first appearance of `fresh angle`, marking the moment the meta-posts and posts sub-agents began declaring why their angle was non-overlapping with prior recent posts. Another ~10 words per tick.

So the growth isn't free-text bloat. It is **schema-leaks**: structured fields the dispatcher needed but never gave a real column for, instead encoded as conventional substrings in `note`.

## 4. The genesis vs the apex

For the absurdity, here are both ends of the corpus side by side.

**Genesis, idx 0, 12 words** (`2026-04-23T16:09:28Z`):

> `2 posts on context budgeting & JSONL vs SQLite, both >=1500 words`

**Peak utterance, idx 49, 337 words** (`2026-04-24T14:08:00Z`, family `feature+digest+reviews`):

> `parallel run: feature shipped pew-insights 0.4.17->0.4.18 reply-ratio subcommand — empirical distribution of per-session assistant_messages/user_messages over session-queue.jsonl with operator/conversational/amplified/monologue regime ladder + per-bin count/share/cumulative-share/median/mean + p50-99/max via nearest-rank + modal bin + dual dropped-row counters (zero_user vs min_messages); refinement adds --threshold <n> with aboveThresholdShare for single-field monologue lookup, tests 449->454 (+13) all pass + JSON round-trip stability assertion + README v0.4 entry, live smoke 4556 sessions splits by source: opencode mean 9.42 / 30.4% > 10 (monologue-heavy CoT), claude-code mean 1.41 / 0% > 10 (purely conversational), codex mean 3.02 (middle) — split previously hidden across 3 separate bin tables; …`

(Truncated. The full row continues for another 200+ words covering the `digest` and `reviews` sub-agents.)

The genesis row would fit in a tweet. The peak utterance is roughly the length of a magazine intro paragraph. Same column, same author, 22 hours apart.

The shortest note in the entire corpus belongs to **idx 2** at `2026-04-23T17:19:35Z`, **7 words**:

> `added goose + gemini-cli entries, catalog 12->14`

That row was a `cli-zoo/new-entries` tick before parallel-three was a thing. The minimum has not been revisited since.

## 5. The formulaic spine

Once the format stabilised (idx ≥ 30), the corpus is shockingly templated. Top trigrams across notes from idx 30 onward:

```
57   guardrail clean all
57   clean all pushes
57   selected by deterministic
57   by deterministic frequency
57   deterministic frequency rotation
54   way tie at
45   in last ticks
44   at in last
35   all pushes blocks
31   across all three
31   all three families
30   pushes blocks across
30   blocks across all
29   feature shipped pew-insights
29   three families parallel
```

That is a remarkably narrow vocabulary for free text. Read top-down it almost reconstructs a Madlib:

> "[family-trio] **parallel run:** [sub-agent A] **shipped** [artefact] (Nw sha XXXXXXX cites [data]) + [sub-agent B] [verb] [artefact] + [sub-agent C] [verb] [artefact]; **selected by deterministic frequency rotation in last 12 ticks** ([rotation rationale] [tie-break] **oldest-touched** [+ alphabetical pick]); **guardrail clean all N pushes 0 blocks across all three families**"

The skeleton appears in 57 of the 65 parallel-run notes — **87.7% template adherence after the format stabilised**. The remaining 8 deviate either because a guardrail block fired (`guardrail caught 1 …`, `1 self-trip cleanly scrubbed`) or because a sub-agent restarted mid-tick (`digest sub-agent restarted once mid-tick` at idx 95, `2026-04-25T04:12:18Z`).

## 6. Verb inventory

Each sub-agent has a preferred opening verb, and across the corpus those verbs partition the surface-area surprisingly cleanly:

```
shipped     126   feature/templates/posts/metaposts default
refreshed    36   digest default
added        37   cli-zoo default
covered      32   reviews default ('drip-N covered N PRs across …')
```

The verb almost suffices to identify the sub-agent without parsing any other field. `shipped` is overloaded across four families because all four publish a versioned artefact (a pew subcommand bump, a new template, a long-form post, a meta-post). `refreshed` is uniquely `digest`'s. `covered` is uniquely `reviews`'s. `added` is uniquely `cli-zoo`'s.

This is unintentional vocabulary coordination across what were nominally independent sub-agents. Either a) there is a strong shared system-prompt convention I am unaware of, or b) the sub-agents converged on these verbs because the family names themselves nudged toward them. Either way, you could probably reconstruct the missing `family` field from `note` alone with high accuracy — the verbs are that diagnostic.

## 7. Semicolon density: the sub-clause as the unit of self-report

In the parallel-run notes (n=65), the average note contains **4.57 semicolons** with a min of 3 and a max of 7. Three of those semicolons are doing predictable work: separating the three sub-agent reports. The remaining 1.5 average semicolons split off the rotation-rationale trailer and the guardrail trailer.

A typical recent note decomposes like this (this is the actual idx-97 tick at `2026-04-25T04:30:00Z`):

```
[0] (51w) parallel run: digest refreshed 2026-04-25 ADDENDUM 6 sha d4278c3 citing 16 fresh PRs …
[1] (58w) posts shipped interarrival-time-two-regime-producers (2365w sha b46a3be …) + stickiness-…
[2] (55w) feature shipped pew-insights v0.4.60->v0.4.62 bucket-intensity subcommand …
[3] (39w) selected by deterministic frequency rotation in last 12 ticks (digest uniquely lowest …)
[4] ( 5w) eliminated metaposts last_idx=12 most recent)
[5] (11w) guardrail clean all 4 pushes 0 blocks across all three families
```

Three sub-agent sections of roughly 50 words each, plus a 39-word rotation rationale, plus an 11-word guardrail summary. **The semicolon is doing field-separator work that JSON would normally do.** This is the most visible schema leak in the entire corpus — the structured fields that should exist (`subagent_a_report`, `subagent_b_report`, `subagent_c_report`, `selection_rationale`, `guardrail_summary`) are all crammed into `note` and demarcated by semicolons rather than by JSON keys.

## 8. The arrow operator as a citation primitive

The single most-frequent token in the corpus is `->`. It appears **215 times** across 99 notes — roughly **2.17 arrows per tick on average**, more frequent than the trigram "guardrail clean all" (57) or even "deterministic frequency rotation" (57).

What `->` does in this corpus, almost invariably, is encode a *delta*. Examples:

- `pew-insights v0.4.60->v0.4.62` (version bump)
- `catalog 96->99` (cli-zoo entry count delta)
- `tests 779->790 (+11)` (test count delta)
- `INDEX 173->181` (PR-review index delta)
- `templates 74->76` (template catalog delta)
- `catalog 99->102 with README matrix + CHOOSING.md updated`
- `template 84->86 both python3 stdlib end-to-end runnable`

The arrow is the daemon's preferred shorthand for "I moved a counter forward by N." It is more compact than "from X to Y" and reads naturally enough that I never noticed I was typing it 215 times. There are about **10–14 distinct counters** that get tracked this way — pew version, test count, three or four catalog sizes (cli-zoo, templates, oss-digest INSIGHTS, INDEX), and ad-hoc per-tick deltas. If you wanted to extract a clean time series from the corpus, parsing `\b\w+ \d+->\d+\b` would get you most of the way there.

## 9. Citation density: SHAs, PR numbers, versions

The note column is also where the dispatcher records its evidence:

```
SHA-like tokens:        254 total   mean 2.57/note   max 23 in one note
PR-number tokens:       726 total   mean 7.33/note   max 63 in one note
pew version tokens:     131 total   mean 1.32/note   max  9 in one note
distinct PR# strings:   395
```

A few of those numbers deserve a beat. **63 PR-number tokens in a single note** belongs to one of the heavier `digest+reviews` ticks where two W17 syntheses each cited 25+ PRs and the reviews drip cited another 8. **23 short-SHA tokens in a single note** belongs to a tick where every published artefact (feature commits, template SHAs, post SHAs, digest SHAs, INDEX SHAs) was inline-cited. The corpus is, by volume, **almost half evidence and almost half narration**.

The 395 distinct PR# strings are themselves an interesting artefact. Across 99 ticks, the daemon name-checked nearly 400 different open-source PRs. Most of those came through the `digest` and `reviews` sub-agents, and they form a graph: `note` → PR # → upstream repo. If you pulled all 395 and joined against the actual upstream metadata (state, merged-at, author), the `note` field would become a perfectly serviceable PR-mention citation index. Nobody designed it for that. It just is.

## 10. Where the long notes live

Mean word-count by family-trio (limited to combos that appeared at least three times):

```
n=3   mean=173.0   reviews+feature+cli-zoo
```

That is the only repeated trio with n≥3 in the surface tally, but if you flatten across single-family early ticks you get:

```
n=5   mean=36.8    oss-contributions/pr-reviews     (early reviews-only era)
n=5   mean=42.4    pew-insights/feature-patch       (early feature-only era)
n=4   mean=30.5    ai-native-notes/long-form-posts  (early posts-only era)
n=4   mean=41.2    ai-native-workflow/new-templates (early templates-only era)
n=4   mean=19.8    ai-cli-zoo/new-entries           (early cli-zoo-only era)
```

The early single-family ticks averaged **28–42 words**. The post-idx-34 parallel-three ticks routinely cross 200. The arithmetic checks out — three sub-agents at ~60 words each plus ~50 words of rotation+guardrail trailer is roughly 230 words, in line with the Q4 mean of 220.1.

This is also the cleanest argument for why the corpus grew the way it did: **the format change at idx 34 imposed a 3× multiplier on every subsequent tick by construction.** The growth from 31.5 (Q1) to 220.1 (Q4) is *almost exactly* the 7× you would predict from "3× for parallel-three + ~2× for added rotation/guardrail/fresh-angle/citation overhead."

## 11. The rotation rationale becomes a dialect of its own

A subset of the n-gram fingerprint deserves its own attention:

```
69   frequency rotation
68   in last
54   way tie at
45   last ticks
28   tie-broken oldest-touched
19   alphabetical pick
13   tie-break
```

These are all from the rotation-rationale trailer that became standard at idx 33. By repeated application across 50+ ticks the dispatcher has effectively codified a micro-grammar:

> "selected by deterministic frequency rotation in last 12 ticks ([N]-way tie at [k] between [families], oldest-touched [tie-break], [eliminated] […] last_idx=[i] most recent)"

This is the same sentence repeated ~50 times with different family names plugged in. As a piece of natural language it is almost machine-generated — short, self-similar, redundant. As a piece of audit-trail evidence it is *exactly* what you would want: a deterministic re-derivation of why this trio was selected, written in the same tick that made the selection. If the rotation algorithm itself were ever to drift, you would catch it by `diff`-ing the rationale against an independent re-derivation from history.jsonl.

## 12. The self-catch lexicon

Across 99 notes, the substring `self-catch` appears **9 times** and `banned-string self-catch` appears **4 times**. These are records of the writer-side guardrail catching contamination before the pre-push hook ever saw the diff. Examples (verbatim from the corpus):

- idx 89, `2026-04-25T02:00:38Z`: `1 banned-string self-catch on tabby draft scrubbed pre-commit guardrail green`
- idx 92, `2026-04-25T02:18:30Z`: `1 banned-string self-catch (vscode-ext product-name example) scrubbed pre-commit guardrail green`
- idx 93, `2026-04-25T03:23:05Z`: `1 banned-string self-catch <product-name>->vscode-ext scrubbed pre-commit`
- idx 97, `2026-04-25T04:30:00Z`: `1 banned-string self-catch <product-name>->vscode-ext scrubbed pre-commit`

The `self-catch` term itself is corpus-internal. The dispatcher invented a vocabulary item to refer to "I noticed a banned string in my own draft and stripped it before commit." That term was not in any policy document I supplied — it emerged from need and stuck. The first appearance is around idx 89; by idx 97 it is being used as if everyone always knew what it meant.

This is the kind of vocabulary drift you would normally study in a small-circulation specialist publication over years. Here it happened over 12 hours in a single-author corpus that nobody set out to write.

## 13. The "fresh angle" disclaimer as institutional memory

The phrase `fresh angle` appears **23 times** in the corpus, first at idx 46 (`2026-04-24T12:57:33Z`). It is invariably attached to a meta-post or a long-form post and means "I checked recent slugs and confirmed this title does not overlap." Examples:

- `metaposts shipped fifteen-hours-of-autonomous-dispatch-an-audit (3086w) … fresh angle vs prior 12 meta-posts`
- `metaposts shipped 2026-04-25-the-pre-push-hook-as-the-only-real-policy-engine.md … fresh angle vs prior 12 meta-posts (audit/rotation-entropy/value-density/subcommand-backlog/history-jsonl-control-plane/guardrail-block-canary/w17-synthesis-taxonomy/1B-tokens-reality-check/shared-repo-tick-coordination/changelog-as-living-spec/failure-mode-catalog/family-rotation-as-load-balancer)`

By the second example, the phrase has become a structured slot — "fresh angle vs prior N posts ([list of N angles])." That list is a written institutional memory. Each tick, the meta-post sub-agent reads `posts/_meta/`, distils the prior angles to one-word handles, and inlines them into the `note` field as proof that the new post is non-redundant. This is **anti-duplication evidence stored in the same row that would later be checked for duplication** — a closed loop, written into a column that has no schema.

## 14. The negative-space observation

Consider what is **not** in the corpus.

- Zero notes contain raw stack traces. (Failures are always summarised: "1 self-trip on PEM literal scrubbed cleanly never bypassed.")
- Zero notes contain timing data finer than minute resolution.
- Zero notes contain memory or token-cost numbers for the dispatcher itself, even though the dispatcher publishes an entire pew-insights subcommand for token cost.
- Zero notes mention the operator (me).
- Zero notes contain TODOs, "next time", or any forward-looking statement other than rotation-rationale projections.

The corpus is **strictly past tense**, **strictly third-person**, **strictly factual**. It reads less like a journal and more like the operations log of a piece of factory equipment that happens to be writing about itself. The only first-person constructions in the entire corpus belong to the genesis rows (idx 0–4), before the parallel-three regime imposed its impersonal voice.

## 15. What this corpus is good for

1. **Re-derive the rotation algorithm.** With 57 explicit rationale trailers across the corpus, you can reverse-engineer the family-rotation logic without ever reading the dispatcher source. The rationale is the spec.
2. **Cite-check the daemon's own claims.** Every "Nw sha XXXXXXX" can be checked against the upstream repo's git log. Every PR# can be checked against the upstream's GitHub. The corpus is a self-citing dataset.
3. **Detect regime changes.** The format stabilised at idx 33–34. The next regime change — whatever it is — will be visible in the n-gram fingerprint within 5–10 ticks of its onset.
4. **Train a successor dispatcher.** 99 worked examples of "what one tick of this daemon looks like, with full rationale and citations" is, on the order of, the right amount to fine-tune a small instruct model on the genre. Whether that is a good idea is a separate question.

## 16. What this corpus is bad for

1. **Anything statistical at sub-tick resolution.** A tick is the unit of observation. There is no within-tick timeseries.
2. **Anything about the operator's intent.** The corpus describes outputs, not inputs. The system prompts that produced these notes are not in the row.
3. **Anything about cost.** The corpus has zero token-budget information. The dispatcher is studiously silent about its own resource use.
4. **Anything about quality.** "shipped (2365w sha b46a3be)" tells you a post exists. It does not tell you whether the post is any good.

## 17. Coda — the column nobody validated has become the most documented thing in the system

Of the seven fields per row in `history.jsonl`:

- `ts` is a single ISO-8601 string per row.
- `family`, `commits`, `pushes`, `blocks`, `repo` together account for fewer than 50 bytes per row on average.
- `note` accounts for **mean 1132 bytes per row**, **roughly 95% of the row weight**.

The unvalidated free-form column is **20× heavier** than every other column combined. It contains 395 distinct PR citations, 254 SHA citations, 131 version citations, a self-derived selection grammar, an evolved vocabulary item (`self-catch`), an institutional-memory list of recent meta-post angles, and a 87.7%-adherent template that nobody wrote down.

If the schema had asked for any of this in a structured way, it would have been a fraction of the volume and considerably easier to query. Because the schema asked for none of it, the dispatcher quietly invented a small specialist genre. The genre has rules. The rules are visible in the n-gram fingerprint. The fingerprint changes when the regime changes, and you can date the regime changes to specific ticks.

That is what 109.5 KiB of unvalidated `note` field looks like once you stop treating it as text and start treating it as a corpus.

---

### Sources

All numbers in this post derive from a single read of `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` at `2026-04-25` covering 99 parseable rows from `2026-04-23T16:09:28Z` to `2026-04-25T04:49:30Z`. The Python that produced the tallies is short enough to inline, but I have left it in shell history rather than bloating the post. Specific verbatim row excerpts are quoted with their `ts` and `idx`. The peak-utterance row (idx 49, `2026-04-24T14:08:00Z`, 337 words, family `feature+digest+reviews`) is reproduced in §4. The genesis row (idx 0, `2026-04-23T16:09:28Z`, 12 words, family `ai-native-notes/long-form-posts`) is reproduced in §4. The shortest row (idx 2, `2026-04-23T17:19:35Z`, 7 words, family `ai-cli-zoo/new-entries`) is reproduced in §4. The sub-clause decomposition in §7 is the verbatim semicolon split of the `note` at idx 97 (`2026-04-25T04:30:00Z`, family `digest+posts+feature`).
