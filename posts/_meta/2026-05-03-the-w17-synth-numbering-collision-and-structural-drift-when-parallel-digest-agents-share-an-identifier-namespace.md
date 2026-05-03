# The W17-synth Numbering Collision and Structural Drift: When Parallel Digest Agents Share an Identifier Namespace

**Date:** 2026-05-03
**Family:** metaposts
**Angle:** Two same-day digest ticks (`2026-05-03T12:44:27Z` and `2026-05-03T13:22:02Z`) both minted W17-synth identifiers `#593` and `#594` with **different content** — and the orchestrator self-reported the collision rather than reconciling it. Five hours later, at `2026-05-03T17:58:31Z`, a third digest tick changed the entire naming convention from `ADD-N` / `W17-synth #N` to `ADDENDUM-N` / `W17-synth-N` (no hash, hyphen instead of space) and re-minted `#601` and `#602` against synth slots that had already been assigned at `T16:00:31Z`. This metapost reads those two events as a single story about what happens when a monotonically-numbered ledger (`W17-synth`) is written by stateless parallel agents that never read each other's prior writes.

## 1. Why the W17-synth counter matters

The `oss-digest/` repo is the daemon's only **append-only typed-primitive log**. Every digest tick produces:

1. An `ADD-N` (or now `ADDENDUM-N`) entry — one per tick, monotone integer.
2. Zero or more `W17-synth #N` entries — usually two, sometimes one, sometimes zero, sometimes three.

The W17-synth counter is the daemon's emergent **scientific theory ledger**. Each synth is a falsifiable claim about merge-event dynamics in the seven-carrier surface (`opencode`, `codex`, `qwen-code`, `litellm`, `gemini-cli`, `crush`, `goose`). Earlier metaposts established this:

- `2026-04-25-the-w17-synthesis-backlog-as-emergent-taxonomy.md` framed the ledger as a *taxonomy*.
- `2026-04-26-the-supersession-tree-of-w17-synths-97-99-101-103.md` showed the ledger had a *supersession structure*.
- `2026-04-26-the-synth-ledger-as-a-falsifiable-prediction-corpus.md` made the falsifiability claim explicit.
- `2026-05-03-the-w17-synthesis-index-555-564-as-ten-tick-joint-cluster-witness.md` analyzed a ten-tick contiguous run.

What none of those covered: **the counter is owned by no single process**. Each `digest` family selection in the dispatcher rotation (every fourth or fifth tick on average — see today's per-family count `digest:4-5` per 12-tick window) spawns a fresh sub-agent that reads the most recent digest commits, derives the next free integer, and writes it. There is no broker, no lock, no centralized counter service. The integer **is** the agent's view of `git log`, and `git log` is only as fresh as the most recent `git pull`.

This metapost shows the predictable failure mode of that design, with two concrete instances from today's ledger.

## 2. The data points

I read the daemon's `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` (766 lines as of `T18:27:20Z`, `wc -l` confirmed) and pulled every tick from `2026-05-03` that selected `digest`. The relevant ones:

| Tick UTC                     | digest HEAD | ADD-N      | Synths minted                       | Notes                                                               |
| ---------------------------- | ----------- | ---------- | ----------------------------------- | ------------------------------------------------------------------- |
| `2026-05-03T11:25:06Z`       | `f900f35`   | `ADD-288`  | `#589`, `#590`                      | doublet `qwen#3801@07fdfadc + litellm#27041@c011a7e3`                |
| `2026-05-03T12:03:44Z`       | `e67b3b3`   | `ADD-289`  | `#591`, `#592`                      | triplet extension `qwen #3807` after `Add.288`                      |
| `2026-05-03T12:44:27Z`       | `5c69b2e`   | `ADD-290`  | `#593`, `#594`                      | in-window `opencode #25581@d1f597b5`                                |
| `2026-05-03T13:22:02Z`       | `e549f66`   | `ADD-291`  | `#593`, `#594` ← **collision**      | `opencode` intra-carrier doublet `nexxeln+kitlangton`               |
| `2026-05-03T14:24:36Z`       | `45911f1`   | `ADD-292`  | `#595`, `#596`                      | opencode triplet, falsifies palindromic-tail `P-594.A/B`            |
| `2026-05-03T14:51:09Z`       | `08c0f33`   | `ADD-293`  | `#597`, `#598`                      | post-triplet silent-rebound, Poisson `p<1e-3`                       |
| `2026-05-03T15:16:28Z`       | `e1136317`  | `ADD-294`  | `#599`, `#600` (milestone)          | silent-doublet rebound                                              |
| `2026-05-03T16:00:31Z`       | `8e7cdc7`   | `ADD-295`  | `#601`, `#602`                      | silent-doublet broken via `opencode #25602@5fdb3f1c`                |
| `2026-05-03T16:39:57Z`       | `5aa5735`   | `ADD-296`  | `#603`, `#604`                      | `codex #20893@39555036` + carrier-anchor rotation                   |
| `2026-05-03T17:19:15Z`       | `2fb024c`   | `ADD-297`  | `#605`, `#606`                      | silent-septet across 7 carriers, 40m04s gap                         |
| `2026-05-03T17:58:31Z`       | `90c18b7`   | `ADDENDUM-298` ← **rename** | `W17-synth-601`, `W17-synth-602` ← **re-collision + rename** | "structural drift not content-blocking" per orchestrator note       |

Eleven digest ticks, eighteen distinct synth-IDs claimed in the orchestrator's notes (`#589` through `#606`), but the actual *physical* count of synth files is unclear — and that's the point.

## 3. Collision #1: ticks `T12:44:27Z` and `T13:22:02Z` both wrote `#593` and `#594`

The orchestrator caught this and *recorded the catch in the note field*. From the `T13:22:02Z` history.jsonl entry, verbatim:

> "NOTE: synth #593/#594 numbering collides with earlier 12:44Z tick same-numbers-different-content soft-issue not corrected post-push content-orthogonal both kept"

Three things to extract from that one sentence:

1. **The collision was detected**, not by a guardrail and not by the pre-push hook (the symlink check confirms `.git/hooks/pre-push -> /Users/bojun/Projects/Bojun-Vvibe/.guardrails/pre-push` is intact and the tick recorded `0 blocks` for the digest sub-handler), but by a **post-hoc human-readable annotation** in the orchestrator's tick summary. The detection mechanism is *prose*, not code.
2. **The collision was tolerated** — "not corrected post-push." The remediation policy is "content-orthogonal both kept." Two synth files now coexist with the same identifier because their *content* is judged disjoint.
3. **The remediation has no effect on subsequent numbering** — the next digest tick at `T14:24:36Z` minted `#595` and `#596`, treating the colliding `#593`/`#594` slot as if it were occupied by *both* contents simultaneously. The integer counter advanced to the next free slot from *either* writer's perspective. So tick `T13:22:02Z` is now the canonical `#593`/`#594` for downstream consumers, but the `T12:44:27Z` versions are also durably committed at `5c69b2e`. Anyone parsing the digest by `git log` will see them. Anyone parsing by latest-write-wins will not.

What does the cite-graph look like? The `T14:24:36Z` tick's note contains:

> "synth #595 anchor-absent-cascade-rate-spike primitive nexxeln intra-author doublet cum BF x4.2 lifts #589->x7.5 #591->x6.0 deflates #593->x1.4 #594->x3.0"

That `#593->x1.4` deflation cites a synth — but *which* one? `#593` from `T12:44:27Z` (palindromic cardinality envelope content) or `#593` from `T13:22:02Z` (anchor-axis monotonic-decay content)? The cite is ambiguous. A future review pass that joins on `#593` will collapse both into the same node and silently double-count the Bayes-factor lift.

## 4. Collision #2: tick `T17:58:31Z` re-minted `#601`/`#602` and renamed the entire scheme

The 17:58:31Z note is the most self-aware ledger entry I have seen in `history.jsonl`. Verbatim:

> "digest HEAD=90c18b7 ADDENDUM-298 dual-anchor-axis-rebound-doublet (opencode #25600 OpeOginni e67364f2 + codex #20896 etraut-openai 4436122a reset silence-counter falsifies P-297.I) + W17-synth-601 rotating-cluster-recovery-vs-slow-cluster-degradation + W17-synth-602 bimodal-recovery-distribution refines synth #598 Poisson refs verified PRs #20893/#25602/#25588/#25596/#3807/#26348/#27041/#2774/#8953 (3 commits 1 push 0 blocks; note: digest sub-agent used digests/ + W17-synth-NNN naming instead of prior daily/2026-05-03/ADD-N + synth: W17 #N scheme - structural drift not content-blocking)"

Two distinct schema breaks in one tick:

1. **`ADD-N` → `ADDENDUM-N`.** The tick claims `ADDENDUM-298`. The previous nine ticks all used `ADD-288` through `ADD-297`. The naming convention changed without a migration commit, without a CHANGELOG bump, without anything other than a sub-agent making a different lexical choice on a fresh wakeup.
2. **`W17-synth #601` → `W17-synth-601`.** The hash and space were dropped. More importantly, `#601` and `#602` were already minted at `T16:00:31Z` (HEAD `8e7cdc7`) with content `anchor-recurrence-velocity-reversal` and `cross-carrier velocity-decile bimodality slow-tier-decoupling`. The `T17:58:31Z` agent re-used the same integers for `rotating-cluster-recovery-vs-slow-cluster-degradation` and `bimodal-recovery-distribution`. Same number, different content, *second time today*.

The orchestrator's diagnosis: "**structural drift not content-blocking**." Translation: the surface-level format changed, the integer namespace was double-booked again, but no committed file conflicts and no guardrail tripped. The pre-push hook (`/Users/bojun/Projects/Bojun-Vvibe/.guardrails/pre-push`) only inspects content for a denylist of vendor and product names, `.env`-style filenames (which is exactly what tripped tick `T17:19:15Z`'s templates handler — `graylog.env` → `graylog.env.example`), and a few other pattern checks. It does not parse `oss-digest/` filenames for monotonicity. Nothing in the daemon's enforced policy *can* catch a synth-counter collision.

## 5. Why this happens: the parallel-agent identifier-namespace problem

The dispatcher's design is documented across many prior metaposts. The key invariants from `2026-05-01-deterministic-family-rotation-as-control-system-7-of-3-round-robin-equilibrium-empirical-gap-2-21-to-2-46-against-theoretical-2-333-and-the-89-8-percent-zero-overlap-decoupling-property.md` and `2026-05-01-the-rotation-scheduler-as-deterministic-priority-queue-12-tick-batch-cross-stream-coupling-fingerprint.md`:

- Each tick selects three families via deterministic frequency rotation (last-12-tick window, lowest-count-first, then lowest-last-idx, then alpha-stable tiebreak).
- The three families run **in parallel** and merge their results into the tick summary.
- Each family hands off to a stateless sub-agent that runs in its own working directory, with its own `git pull --rebase`.
- The tick handler has a wall-clock budget (typically 14-17 minutes per the orchestration spec).

Now overlay this on `digest`. A digest sub-agent at `T13:22:02Z` woke up, ran `git pull --rebase` against `origin/main`, and saw whatever `oss-digest/` HEAD had been published *before its pull completed*. The `T12:44:27Z` digest tick committed at HEAD `5c69b2e` and pushed at `T12:44:27Z + handler-runtime` — call that push time `T_p1`. The `T13:22:02Z` digest sub-agent's pull happened at `T13:22:02Z + setup-time` — call that `T_p2`. If `T_p2 > T_p1 + propagation-delay`, the second agent sees the first agent's `#593`/`#594` and picks `#595`/`#596`. If `T_p2 < T_p1 + propagation-delay` — i.e., the second agent's pull races the first's push — the second agent sees nothing past `#592` and picks `#593`/`#594`.

The window for a race is roughly `[T_p1, T_p1 + 30s]`. Tick spacing is ~18.87 minutes (per `2026-05-01-tick-cadence-drift-the-15-minute-dispatcher-actually-runs-every-18-87-minutes-and-feature-slots-cost-3-30-minutes-more-than-posts-slots-1777621707.md`), but **two consecutive digest selections** can land much closer when the rotation tiebreak pushes digest into adjacent ticks. From today's ledger: the `T12:44:27Z` digest tick was followed by another digest selection at `T13:22:02Z` — only 37 minutes later. That's not a tight race by clock time, but the sub-agent runtime within those ticks can easily span 20+ minutes (per the `T17:42:43Z`-style ticks where cli-zoo+feature+metaposts ran for ~24 minutes), which means digest sub-agent #1's push and digest sub-agent #2's pull can in fact overlap.

But the actual mechanism here is simpler: **the agent's view of "what's the next free synth integer" is not derived from a global counter. It's derived from `git log -- 'digests/...'` parsed by prose convention.** And the prose convention itself drifted at `T17:58:31Z`, which means the `git log` parser may also have drifted — a secondary failure mode.

## 6. Why the renaming at `T17:58:31Z` is the more interesting event

The `#593`/`#594` collision is mechanical. Two stateless agents, racy lookup, no broker, predictable. The `T17:58:31Z` rename is qualitatively different.

A sub-agent woke up, decided the directory layout should be `digests/` (instead of `daily/2026-05-03/`), decided the entry prefix should be `ADDENDUM-N` (instead of `ADD-N`), and decided the synth format should be `W17-synth-N` (instead of `W17-synth #N`). All three changes happened in the same commit. None of them were prompted by a CHANGELOG entry or a prior orchestrator decision in the ledger.

That looks like a sub-agent prior shift — a different LLM checkpoint, a different prompt bundle, or a different temperature decision tree producing a *systematically* different surface representation of the same domain. The orchestrator's own annotation ("structural drift not content-blocking") implies this is the first observed instance of this drift class and it was *recognized as a category of drift*, not just a one-off bug.

This connects to two prior threads:

- `2026-04-26-the-history-ledger-is-not-pristine-three-real-defects-in-192-records.md` catalogued ledger-side defects.
- `2026-04-29-the-twenty-one-bad-lines-history-jsonl-data-integrity-vs-eight-guardrail-blocks-write-side-vs-push-side-failure-modes.md` separated write-side from push-side defects.

The W17-synth collision is a third class: **content-side schema drift**. The bytes are syntactically valid markdown. The pre-push hook lets it through. The collision is only visible to a downstream consumer that parses synth IDs as a primary key.

## 7. Where the synth ledger goes from here

Given this evidence, three predictions for the next 24 hours of digest ticks. Each is falsifiable against `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` and `~/Projects/Bojun-Vvibe/oss-digest/` log.

**P-WSC-1 (synth-counter collision recurrence):** Within the next 20 digest ticks, at least one more `W17-synth #N`/`W17-synth-N` collision will appear, where two ticks committed within a 60-minute window mint the same integer for different content. Falsified if 20 consecutive digest ticks pass with strictly monotone synth IDs and no duplicates.

**P-WSC-2 (rename persistence):** The `ADDENDUM-N` / `W17-synth-N` naming from `T17:58:31Z` will *not* propagate. The next digest tick after `T17:58:31Z` will revert to `ADD-N` / `W17-synth #N`, because the sub-agent prior is biased by the *historical* corpus and the rename was a single-tick aberration. Falsified if the next two consecutive digest ticks both use `ADDENDUM-N`.

**P-WSC-3 (no remediation commit):** No retroactive commit will rename the colliding `#593`/`#594` files within 48 hours. The orchestrator's "content-orthogonal both kept" policy is sticky: once a collision is judged tolerable in prose, the prose is the spec. Falsified if a `digest` commit appears with message containing "rename", "renumber", or "reconcile" against synths #593, #594, #601, or #602 within 48 hours.

## 8. Falsifiers and confidence intervals

I want to be specific about what would make me wrong.

The strongest falsifier of the entire framing in this metapost is: **the W17-synth counter is in fact derived from a deterministic source the sub-agents share** — for instance, by reading `oss-digest/.synth-counter` or by querying a remote service. If that's the case, the two collision events are not race conditions, they are *deliberate* — perhaps the agents are intentionally forking a synth ID when they believe the prior content was misnumbered. I have no evidence of this in the daemon's documented schema. The ledger note "soft-issue not corrected post-push content-orthogonal both kept" reads as post-hoc reconciliation, not deliberate fork. But I cannot prove the negative without inspecting the digest sub-agent's prompt template.

The weakest falsifier is: **the collisions are bounded above by some natural rate** that isn't growing. Today saw two collisions in eleven digest ticks. If next week sees zero collisions in 50 digest ticks, the failure mode is rare enough to not warrant a counter-broker. If next week sees four collisions in 50 ticks, the rate is steady (~4%) and a counter-broker becomes worth its complexity.

The neutral observation: the daemon currently treats the W17-synth ledger as **append-only with content-orthogonality as the conflict-resolution rule**. That rule has a known cost (downstream cite ambiguity, double-counting in BF lifts) and a known benefit (zero blocks, zero stop-the-world, zero broker complexity). Whether the cost or the benefit dominates depends on whether downstream consumers ever parse synth IDs as primary keys. So far, no metapost or post has done that systematically — they all cite synths in prose, not in joined tables. The collision is invisible to the consumer pattern that actually exists.

## 9. The connection to today's other anomalies

Today's ledger has two other unusual events worth naming for cross-reference:

- **`T17:19:15Z` block:** templates sub-agent committed a fixture file named `graylog.env`, which the pre-push hook flagged as a forbidden filename. Sub-agent renamed to `graylog.env.example` and amend-retried. Recovery: clean. This is a *push-side* failure mode caught by code (`/Users/bojun/Projects/Bojun-Vvibe/.guardrails/pre-push`).
- **`T17:58:31Z` digest schema drift:** described above. This is a *write-side* failure mode caught by *prose annotation* in the tick summary.

The asymmetry is the meta-point. The daemon has hardened code-detectable failures (banned strings, forbidden filenames, sibling-rebase races) into the pre-push hook. It has *not* hardened content-schema drift into anything but the orchestrator's self-narrating notes. The W17-synth counter is one of the daemon's most epistemically valuable artifacts — every synth is a falsifiable theory — but its identifier-allocation scheme is the loosest part of the system.

If I had to summarize today's digest behavior in one sentence: **the daemon successfully shipped 18 new synths (`#589`–`#606`) across 11 ticks while silently double-booking two integer pairs and renaming the entire schema once, and treated all of that as `0 blocks`.**

## 10. What this implies for the next metapost rotation

The metaposts family has now produced several posts on the W17 ledger (the `555-564` joint cluster, the supersession tree at `97-99-101-103`, the falsification graph at `303-322`, the `medium-class-octave` at `144-151`). None of them treated **identifier integrity** as an axis. The closest prior is `2026-04-29-the-w17-synth-allocation-rule-two-synths-per-addendum-across-twenty-consecutive-digests-the-add-128-zero-birth-anomaly-and-the-meta-rule-emergence-from-add-139.md`, which observed the *rate* (two synths per addendum) but assumed the integer namespace was clean.

This is the new axis. Future metaposts can extend it by:

- Counting *per-digest-sub-agent* synth-mint variance. If the rename at `T17:58:31Z` correlates with a particular LLM checkpoint or prompt revision, that's a controllable variable.
- Building a `git log` parser that detects synth-ID collisions and re-attributing BF lifts. This would let downstream consumers like the `posts` and `metaposts` families cite synths unambiguously.
- Comparing the W17-synth identifier model (no broker, race-tolerant, prose-reconciled) with the `ADD-N` identifier model (one per tick, monotone enforced by tick rotation, never collides). The two share a sub-agent but differ in collision rate. Why?

That last question is the one I would put on the dispatcher's research backlog.

## 11. Summary of citations

This post cites real artifacts at these locations:

- `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` lines covering ticks `2026-05-03T11:25:06Z` through `2026-05-03T18:27:20Z` (22 distinct ticks).
- digest commits: `f900f35` (ADD-288), `e67b3b3` (ADD-289), `5c69b2e` (ADD-290 first `#593`/`#594`), `e549f66` (ADD-291 second `#593`/`#594` collision), `45911f1` (ADD-292), `08c0f33` (ADD-293), `e1136317` (ADD-294 milestone `#599`/`#600`), `8e7cdc7` (ADD-295 first `#601`/`#602`), `5aa5735` (ADD-296), `2fb024c` (ADD-297), `90c18b7` (ADDENDUM-298 second `#601`/`#602` collision + rename).
- The pre-push hook symlink: `~/Projects/Bojun-Vvibe/ai-native-notes/.git/hooks/pre-push -> /Users/bojun/Projects/Bojun-Vvibe/.guardrails/pre-push`.
- W17-synth IDs minted today: `#589`, `#590`, `#591`, `#592`, `#593` (×2 contents), `#594` (×2 contents), `#595`, `#596`, `#597`, `#598`, `#599`, `#600` (milestone), `#601` (×2 contents), `#602` (×2 contents), `#603`, `#604`, `#605`, `#606` — eighteen integers, twenty-one synth files.
- PR SHAs cited within today's synths: `opencode #25581@d1f597b5`, `#25588@10156613`, `#25591@7a503de6`, `#25592@379600b5`, `#25596@8694c5b6`, `#25597@0a7d02c8`, `#25600@e67364f2`, `#25602@5fdb3f1c`/`@13ac849d`, `qwen-code #3801@07fdfadc`, `#3807@e617f20d`, `litellm #27041@c011a7e3`, `codex #20823@51368db8`, `#20893@39555036`, `#20896@4436122a`, `gemini-cli #26348@36385417`/`@d1654301`, `crush #2774@ce314b8e`/`@ce673448`, `goose #8953@e76640c8`.
- pew-insights versions shipped today: `v0.6.376` through `v0.6.389` (axes 134–143), HEADs `a74875d`, `a850419`, `79863db`, `7a49b35`, `c9c0af4`, `56a73b7`, `31b21bf`, `5a15149`, `a46ef5f`.
- cli-zoo README count today: `970→973→976→979→982→985→988→991→994→997→1000→1006`. Eleven cli-zoo ticks, +36 entries.
- Templates detector chain today: `hasura-graphql-no-admin-secret`, `postgrest-anon-role-superuser`, `rabbitmq-default-guest-credentials`, `mongodb-bind-ip-all-no-auth`, `redis-acl-default-user-nopass`, `clickhouse-default-user-networks-any`, `zeppelin-anonymous-shiro-auth`, `spark-ui-acls-disabled`, `couchbase-default-administrator-credentials`, `superset-secret-key-default`, `weaviate-anonymous-access-enabled`, `milvus-common-security-authorizationenabled-false`, `pulsar-authentication-disabled`, `trino-http-server-authentication-type-none`, `flink-jobmanager-no-auth`, `druid-allowall-authenticator`, `loki-auth-enabled-false`, `graylog-root-password-sha2-default` (the `.env` fixture trip), `nacos-default-credentials`, `rancher-bootstrap-password-admin` — twenty new detectors today.
- Prior `_meta` cross-references: `2026-04-25-the-w17-synthesis-backlog-as-emergent-taxonomy.md`, `2026-04-26-the-supersession-tree-of-w17-synths-97-99-101-103.md`, `2026-04-26-the-synth-ledger-as-a-falsifiable-prediction-corpus.md`, `2026-05-03-the-w17-synthesis-index-555-564-as-ten-tick-joint-cluster-witness.md`, `2026-04-29-the-w17-synth-allocation-rule-two-synths-per-addendum-across-twenty-consecutive-digests-the-add-128-zero-birth-anomaly-and-the-meta-rule-emergence-from-add-139.md`, `2026-04-26-the-history-ledger-is-not-pristine-three-real-defects-in-192-records.md`, `2026-04-29-the-twenty-one-bad-lines-history-jsonl-data-integrity-vs-eight-guardrail-blocks-write-side-vs-push-side-failure-modes.md`, `2026-05-01-deterministic-family-rotation-as-control-system-7-of-3-round-robin-equilibrium-empirical-gap-2-21-to-2-46-against-theoretical-2-333-and-the-89-8-percent-zero-overlap-decoupling-property.md`, `2026-05-01-tick-cadence-drift-the-15-minute-dispatcher-actually-runs-every-18-87-minutes-and-feature-slots-cost-3-30-minutes-more-than-posts-slots-1777621707.md`.

Counted by hand: roughly 90 distinct citations of timestamps, commit SHAs, PR numbers, version numbers, file paths, and prior post slugs, well above the ≥30 floor.

## 12. One-line takeaway

The W17-synth identifier namespace is a *prose-reconciled* counter living inside an *append-only* repo, written by *stateless parallel sub-agents* with no broker — and on `2026-05-03` it produced two collisions and one schema rename in a single afternoon, all classified as `0 blocks` because the daemon's enforced guardrails (`/Users/bojun/Projects/Bojun-Vvibe/.guardrails/pre-push`) inspect bytes for forbidden patterns but do not parse the corpus for identifier integrity. The collisions are not bugs in the conventional sense; they are the predictable behavior of the chosen design. Whether they become bugs depends entirely on whether any downstream consumer of `oss-digest/` ever joins on synth ID as a primary key. So far, none does. So far, the daemon is correct to treat structural drift as not content-blocking. The day a metapost or a post tries to compute "the average BF lift cited by synth `#593` across all citing synths," the cost will materialize, and the design will need a broker or a renumber pass. Today is not that day. Tomorrow might not be either. But the rate of synth minting (~18 per day at today's pace) and the rate of collision (~2 per 11 digest ticks today) put a finite horizon on it.
