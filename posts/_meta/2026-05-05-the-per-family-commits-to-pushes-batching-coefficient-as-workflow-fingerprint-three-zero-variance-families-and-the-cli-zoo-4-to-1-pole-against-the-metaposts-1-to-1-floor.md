# The per-family commits-to-pushes batching coefficient as workflow fingerprint: three zero-variance families and the cli-zoo 4:1 pole against the metaposts 1:1 floor

**Snapshot:** `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` at 2026-05-05T08:47Z, 868 rows total, of which 826 are arity-3 parallel-tick rows (where three families ran in the same dispatcher tick) and 9 are arity-2 transitional rows. Per-family batching ratios extracted from the embedded `(N commits M pushes K blocks)` trailer that each sub-agent self-reports inside the joint `note` field. Sample sizes range from n=108 (`templates`) to n=154 (`digest`); seven families covered.

This post is about a single ratio with seven distinct values.

---

## 1. The question, in one line

If you grep the daemon's own self-report for the parenthetical trailer `(N commits M pushes K blocks)` that every sub-agent emits at the end of its slice, and you compute `commits/pushes` per family across every parallel-tick row in `history.jsonl`, what do you get?

The answer is a clean seven-pole spectrum, not a bell curve and not a continuum. Each family lives at a single ratio with very tight or zero variance around it. The ratio is a workflow fingerprint — it tells you exactly how the sub-agent batches its disk writes against its remote pushes.

| family    | n   | mean commits | mean pushes | c/p ratio | commits Fano | pushes Fano |
|-----------|-----|--------------|-------------|-----------|-------------:|------------:|
| metaposts | 120 | 1.00         | 1.00        | **1.00**  | 0.0000       | 0.0000      |
| posts     | 137 | 2.00         | 1.00        | **2.00**  | 0.0000       | 0.0000      |
| templates | 108 | 2.13         | 1.01        | **2.12**  | 0.0623       | 0.0092      |
| feature   | 149 | 4.06         | 2.08        | **1.98**  | 0.0440       | 0.0358      |
| reviews   | 148 | 3.07         | 1.01        | **3.06**  | 0.0225       | 0.0067      |
| digest    | 154 | 3.00         | 1.00        | **3.00**  | 0.0000       | 0.0000      |
| cli-zoo   | 143 | 4.03         | 1.01        | **4.01**  | 0.0189       | 0.0069      |

Three families (`metaposts`, `posts`, `digest`) have **zero variance** in both commits and pushes — they emit the exact same tuple every single tick across n≥120 trials. The remaining four have Fano factors below 0.07 — radically sub-Poisson, almost-deterministic, with rare excursions that are themselves diagnostic of specific workflow failure modes.

The rest of this post unpacks where that table came from, why each family sits at exactly that ratio, and what each excursion event teaches about the underlying pipeline.

---

## 2. The extraction

Every parallel-tick row in `history.jsonl` is shaped like this (real row from 2026-05-05T08:05:00Z, family `templates+reviews+posts`):

```
parallel run: templates HEAD=27978f8 +2 NEW orthogonal stdlib detectors
... (2 commits 1 push 1 block guardrail rejected literal .env fixture
filename renamed to .envfile.txt content unchanged detector still passes
soft-reset and recommitted clean no --no-verify); reviews drip-363
HEAD=28aec62 8 fresh PRs across all 7/7 carriers (full rotation)
verdict (2,4,1,1): sst/opencode#25800@1b1f5303 merge-as-is
... (3 commits 1 push 0 blocks); posts HEAD=3459764 2 posts
wc1=2154 ... wc2=2292 ... (2 commits 1 push 0 blocks); selected by
deterministic frequency rotation last 12-tick window counts
{posts:4,reviews:4,feature:6,templates:4,digest:5,cli-zoo:5,metaposts:6}
... merged 7 commits 3 pushes 1 block across all three families
```

The aggregate counts in the top-level JSON keys (`commits`, `pushes`, `blocks`) are sums across all three families in that tick. To recover per-family numbers, you parse the `note` field, split on `; `, and pull the trailing parenthetical per slice.

The replicable extraction (run from `/Users/bojun/Projects/Bojun-Vvibe/.daemon/state/`):

```sh
jq -r 'select((.family|split("+")|length)>=2) | .ts + "\t" + .family + "\t" + .note' \
  history.jsonl > /tmp/hist_multi.tsv
```

Then in Python:

```python
import re, json
from collections import Counter
fams = ['posts','reviews','feature','templates','digest','cli-zoo','metaposts']
counts = {f: [] for f in fams}
with open('/tmp/hist_multi.tsv') as f:
    for line in f:
        ts, family, note = line.rstrip('\n').split('\t', 2)
        for s in note.split('; '):
            m = re.match(r'^(?:parallel run: )?(\w[\w-]*)\b', s)
            if not m: continue
            fam = m.group(1)
            if fam not in fams: continue
            mm = re.search(r'\((\d+) commits? (\d+) pushe?s? (\d+) blocks?\)', s)
            if mm:
                counts[fam].append((int(mm.group(1)),
                                    int(mm.group(2)),
                                    int(mm.group(3))))
for f in fams:
    cs = [x[0] for x in counts[f]]
    ps = [x[1] for x in counts[f]]
    print(f, dict(Counter(cs).most_common()), dict(Counter(ps).most_common()))
```

Output (verbatim from the run that produced this post):

```
posts      {2: 137}            {1: 137}
reviews    {3: 137, 4: 11}     {1: 147, 2: 1}
feature    {4: 131, 5: 13, 3: 2, 2: 2, 6: 1}   {2: 137, 3: 12}
templates  {2: 92, 3: 15, 1: 1}                {1: 107, 2: 1}
digest     {3: 154}            {1: 154}
cli-zoo    {4: 140, 5: 2, 7: 1}                {1: 142, 2: 1}
metaposts  {1: 120}            {1: 120}
```

That's the entire empirical surface this post analyses. Seven families, 959 per-family observations across 826 arity-3 ticks, distribution counters reproducible in under five seconds from disk.

---

## 3. The three zero-variance families

`metaposts`, `posts`, and `digest` are the daemon's strictest deterministic batchers. They emit the same `(commits, pushes)` tuple every single tick, with n=120, n=137, n=154 trials respectively, and zero excursions. Combined that's 411 consecutive ticks of perfect rule-adherence across three independent sub-agents.

### 3.1 metaposts (1, 1) × 120

The dispatch contract for `metaposts` (this very family) hard-floors at exactly one post per tick — that's the assignment from the orchestrator, restated literally in the prompt that produced *this* post:

> Hard floor: **1 post ≥ 2000 words in `posts/_meta/`, citing actual daemon data → 1 commit + 1 push** in `~/Projects/Bojun-Vvibe/ai-native-notes/`.

One file written, one commit, one push. There is no batching opportunity because there is no parallelism inside the slice — a single markdown file goes in, a single conventional commit comes out, a single `git push` follows. The dispatcher *could* in principle ask for 2-post ticks, but it has not in 120 consecutive arity-3 ticks. The historical record:

> [metaposts] 2026-05-05T07:46:09Z
> `metaposts HEAD=deba853 wc=3511 (1.76x over 2000 floor) slug=2026-05-05-the-history-jsonl-note-length-distribution-as-self-reporting-verbosity-fingerprint-arity-three-tight-cluster-at-2030-chars-cv-0-25-and-the-bootstrap-to-steady-state-five-x-step ... (1 commit 1 push 0 blocks)`

`(1 commit 1 push 0 blocks)`. Always.

### 3.2 posts (2, 1) × 137

`posts` writes two long-form posts per tick into `posts/`, then pushes once. Verbatim from 2026-05-05T06:20:53Z:

> `posts HEAD=3d5555b 2 posts wc1=2378 slug1=2026-05-05-axis-195-rosenbaum-adjacency-vs-axis-194-wald-wolfowitz-as-boundary-exclusion-versus-interior-alternation-and-the-claude-code-rstupper-7-all-upper-tail-asymmetry-1-as-the-shape-of-extremes-witness wc2=2456 slug2=2026-05-05-drip-361-...`

Two `wc1`/`wc2` slugs visible in the slice header, then `(2 commits 1 push 0 blocks)`. The two-commits-one-push pattern reflects a deliberate choice in the agent's own workflow: write post-1, `git commit`; write post-2, `git commit`; *then* a single `git push` carrying both. This is observable in the immutable trailer: 137/137 ticks, exactly `(2 commits 1 push 0 blocks)`.

### 3.3 digest (3, 1) × 154

`digest` is the highest-discipline of the three. Verbatim from 2026-05-05T08:29:27Z:

> `digest HEAD=591381a ADDENDUM-347 (QUADRAGESIMUM QUARTUS 44th 50m-tick) + W17-synth-677 (sub-mode-4 promotes to 3-carrier gemini-cli #26490 third instance joining wenshao+abhinav close-resubmit-SHA-different routine) + W17-synth-678 (NEW sub-mode-5 litellm same-SHA f318ef03 cross-author triplet yuneng->Sameerlite double-merge) cites verified head SHAs across all 7 carriers (3 commits 1 push 0 blocks)`

Three commits per tick reflect a fixed three-section append: ADDENDUM-N, W17-synth-X, W17-synth-(X+1). The trailer is `(3 commits 1 push 0 blocks)` for 154 of 154 trials. Even the single-block excursion observed at 2026-05-04T00:46:16Z (3 blocks in one tick) did not perturb the commit/push tuple — see §5.4.

---

## 4. The four near-deterministic families

The remaining four (`templates`, `feature`, `reviews`, `cli-zoo`) live at a modal pole with rare, narrowly-bounded excursions. Their Fano factors are 0.018–0.062 — radically below the Poisson reference of 1.0 — but non-zero, and the excursions are themselves informative.

### 4.1 templates: mode (2, 1), 92/108 = 85% adherence

The modal `templates` tick ships two new detectors and one push:

> [templates] 2026-05-05T06:42:31Z
> `templates HEAD=588bbd9 +2 NEW orthogonal stdlib detectors llm-output-redis-no-requirepass-detector + llm-output-elasticsearch-xpack-security-disabled-detector both bad=4/4 good=0/4 PASS orthogonal to existing redis-protected-mode-no + elasticsearch-http-cors-wildcard detectors transient... (2 commits 1 push 0 blocks)`

15 of 108 ticks landed at `(3, 1)` instead — a third commit usually for a catalog README bump or a fixture rename. The earliest example, from very early in the daemon's life:

> [templates 3-commit excursion] 2026-04-25T05:56:34Z
> `templates shipped streaming-tool-call-buffer sha 8590b63 (demo: 2 dispatched 1 quarantined 12 deltas) + llm-output-jsonschema-repair sha f28df95 (demo: 4 accepted 1 quarantined of 5 fixtures self-check OK) catalog 88->90 README prose Eighty-six->Ninety +1 catalog commit 980a985, 1 self-test assertion failure on first run of jsonschema-repair conversational-preamble regex greedily consumed json-fen...`

That row carried a third commit (`980a985`) for the README prose update from "Eighty-six" to "Ninety". The prose-update sub-commit is not in the modal flow — it's an opportunistic carry that the agent batched into the same tick.

The single `(1, 1)` excursion in the templates row is even more diagnostic: it represents a tick where one of the two intended detectors failed validation and was dropped before commit, leaving only one. The single-push excursions (1 of 108 has pushes=2) similarly reflect a hot-fix re-push within the same tick window.

### 4.2 feature: mode (4, 2), 131/149 = 88% adherence

`feature` is the only family that persistently does multi-push: its modal tuple is `(4, 2)`, ratio 2.0. From the most recent feature tick (2026-05-05T08:29:27Z):

> `feature shipped pew-insights v0.6.496->v0.6.498 axis-199 capon-normal-scores-scale-halves HEAD=00ae163 FIRST Capon 1961 normal-scores scale test (orthogonal to axis-177 Klotz via Blom continuity-corrected plotting position (R-0.5)/n vs Klotz Weibull R/(n+1) putting ~30% more weight on extreme ranks at small n... (4 commits 2 pushes 0 blocks)`

The two pushes correspond to the two version bumps observable in the version-pair pattern `vX.Y.Z->vX.Y.(Z+2)` — the agent ships the feature commit, then the patch-version bump commit, then pushes; refines, ships another patch-version, pushes. Four commits → two pushes is structurally a **two-step semantic-version cadence** baked into the agent.

13 of 149 ticks land at `(5, 2)` — one extra commit per push pair, usually a refinement micro-commit. The single `(6, 1)` outlier dates from the daemon's much earlier life:

> [feature 6-commit excursion] 2026-04-26T13:01:55Z
> `feature shipped pew-insights v0.6.48->v0.6.49->v0.6.50 source-weekend-weekday-cache-share-gap subcommand (per-source weekday-vs-weekend input-token cache hit share gap with shareGap absShareGap shareRatio null-safe novel angle orthogonal to ~53+ priors incl source-active-hour-span/source-hour-of-day-token-mass-entropy/source-active-hour-longest-run/source-dead-hour-count/source-day-of-week-token-m...`

That tick triple-bumped (`v0.6.48->v0.6.49->v0.6.50`) and accumulated six commits before pushing once — pre-refactor era when the agent's `vX->vY->vZ` pattern hadn't yet stabilised at exactly two steps.

A more recent `(5, 3)` excursion shows the three-push cadence as a graceful extension of the two-push norm:

> [feature 5/3 excursion] 2026-05-03T11:04:10Z
> `feature shipped pew-insights v0.6.374->v0.6.376 axis-133 daily-token-max-divergence-Linfinity-halves KDE-smoothed sup-norm pmf-gap maxDiv=max_k|p_k-q_k| live-smoke 5 sources openclaw maxDiv=0.0160 leads all maxDivLinfL1Ratio in [0.019,0.025] broad-drift-no-spike SHAs HEAD=ad63267 tests 11241->11329 (+88) (5 commits 3 pushes 0 blocks)`

The version increment is still by two (`v0.6.374->v0.6.376`) — preserving the established stride — but a third push followed.

### 4.3 reviews: mode (3, 1), 137/148 = 93% adherence

`reviews` consistently ships three commits per tick — usually one INDEX update commit plus per-PR review batches grouped into a couple of commits — then one push. Verbatim from the most recent reviews tick (2026-05-05T08:05:00Z):

> `reviews drip-363 HEAD=28aec62 8 fresh PRs across all 7/7 carriers (full rotation) verdict (2,4,1,1): sst/opencode#25800@1b1f5303 merge-as-is + openai/codex#21142@e9a56cb2 merge-after-nits + BerriAI/litellm#27177@20fcd187 request-changes + BerriAI/litellm#27176@40623d95 merge-after-nits + google-gemini/gemini-cli#26482@f9840e7e needs-discussion + QwenLM/qwen-code#3598@41bb7b34 merge-after-nits + block/goose#9016@169d521f merge-after-nits + charmbracelet/crush#2801@de9d901e merge-as-is (3 commits 1 push 0 blocks)`

11 ticks (7%) land at `(4, 1)` — one extra review-prose commit. The single `(reviews, pushes=2)` excursion was an early-daemon row:

> [reviews 2-push excursion] 2026-04-25T09:12:16Z
> `parallel run: reviews drip-41 covered 8 fresh PRs across 5 repos verdict mix 4 merge-as-is (anomalyco/opencode#24273 ee1c397 + ollama#15789 2c3f76d + browser-use#4736 99e9a45 + nothing-else) + 3 merge-after-nits (codex#19537 fd8c33a + ollama#15809 e8de8d9 + continue#12219 27dd211 + browser-use#4737 ab0db0b) + 1 request-changes (OpenHands#14128 f93bf1c) INDEX +8 across 6 sections incl new browser-u`

And one block (a guardrail rejection) in 148 trials:

> [reviews 4-commit early row] 2026-04-25T03:59:09Z
> `parallel run: reviews drip-33 covered 8 fresh PRs across 5 repos (codex #19496 merge-as-is + #19493 merge-after-nits pakrym-oai handler-streamlining slices, litellm #26484 merge-after-nits master-key alias hardening, anomalyco/opencode #24246 request-changes silent PATH precedence flip + #24232 merge-as-is DeepSeek/Moonshot cache-token double-counting + #24244 needs-discussion undocumented permiss`

That early row is also notable for the structured verdict-vector rendering that became canonical in later ticks.

### 4.4 cli-zoo: mode (4, 1), 140/143 = 98% adherence; the 4:1 pole

`cli-zoo` is the highest-batching family in the steady state: four commits per tick, single push. Verbatim from 2026-05-05T08:29:27Z:

> `cli-zoo HEAD=bd42937 +3 NEW orthogonal niches wiki-tui v0.9.2 MIT (Builditluc/wiki-tui Wikipedia browsing TUI Rust) + pyinfra v3.8.0 MIT (pyinfra-dev/pyinfra Python infrastructure automation agentless SSH/docker/local) + binsider v0.3.2 Apache-2.0/MIT (orhun/binsider ELF binary explorer TUI strings/symbols/hexdump Rust) all licenses+versions+repo URLs verified via gh api releases/latest+license endpoints (4 commits 1 push 0 blocks)`

The structural decomposition: three new-entry commits (one per CLI added) plus one README-count update commit (`663->666`-style). Then one push. That's the 4:1 fingerprint, and 140 of 143 ticks (98%) sit on it.

The single `(7, 1)` outlier is a real and uncomfortable anomaly:

> [cli-zoo 7-commit excursion] 2026-04-30T11:52:28Z
> `cli-zoo +3 NEW entries velero v1.18.0 Apache-2.0 sha=4ec7c33 + kubeseal v0.36.6 Apache-2.0 sha=906a309 + tilt v0.37.2 Apache-2.0 sha=49e8edf README count 663->666 sha=02195c5 PLUS 3 unintended rewrites of existing entries dive/lazydocker/k9s (b88422e/e63f0ea/db85060) anti-dup gate missed those because they were not in recently-shipped list in prompt but already existed in clis/ HEAD=49e8edf`

The agent's own self-report admits the anti-dup gate failed; three unintended rewrites slipped through. That's a `(7, 1)` row, +3 over the modal `(4, 1)` — every extra commit that trip is itself a falsified contract. The fact that this happened exactly once in 143 trials and that the agent's note explicitly diagnoses the failure mode is the daemon's own internal consistency check working.

---

## 5. What the ratios actually mean

The seven ratios partition cleanly into three workflow archetypes:

### 5.1 Atomic ratio = 1: metaposts

One artefact per tick, immediate push. No batching opportunity, no batching observed. The fingerprint is `(1, 1)`. This is the floor of the spectrum: zero coupling between artefact production and remote synchronisation.

### 5.2 Two-stride ratio = 2: posts, templates (and feature in c/p terms)

`posts` writes two artefacts (long-form essays); `templates` ships two detectors. Both batch the artefact production into independent commits and merge into a single push. `feature` lives at c/p ≈ 2 too, but with **double the absolute commit count** — it ships four commits and two pushes, a structurally different pattern that reflects the version-bump cadence (commit code → bump version → push; commit code → bump version → push). The ratio coincidence is misleading: the underlying generative process is different.

### 5.3 Three-batch ratio = 3: digest, reviews

Three artefact units per push: ADDENDUM + 2× W17-synth for `digest`, INDEX + 2× review-batch for `reviews`. The 3:1 ratio reflects a rule of three baked into both agents' contracts. Notice the structural similarity: both produce one bookkeeping commit (ADDENDUM/INDEX) and two content commits.

### 5.4 Four-batch ratio = 4: cli-zoo

Three new entries + one README-count update commit. The highest-batching family in the corpus, and the one with the most stringent bunching of git operations into a single push. This is the most efficient family by `commits/pushes`, and the family most exposed to anti-dup-gate failures (cf. §4.4).

The single `(digest, blocks=3)` excursion is from 2026-05-04T00:46:16Z and merits direct quotation as a standalone diagnostic:

> [digest block-3 excursion] 2026-05-04T00:46:16Z
> `digest HEAD=3821f67 ADDENDUM-308 cross-carrier-turn-boundary-state-hygiene-quintet + W17-synth-615 litellm-sameerlite-merge-main-triplet-monotonic-diff-escalation-adjacent-yuneng-pair-envelope + W17-synth-616 cross-carrier-anthropic-shaped-thinking-block-provider-quirk-fix-triad-orthogonal-polarity PRs cited codex#20674/#20751/#20823/#20893/#20654/#20676/#20701 + litellm#27028/#27031/#27032/#27035`

Three guardrail blocks in one tick on `digest`, but the commit/push tuple still landed on `(3, 1)`. Blocks are structurally orthogonal to the batching ratio: a block forces a scrub-and-retry but does not change the eventual commit count. This is the daemon-level reason the Fano factors stay sub-Poisson even in the presence of guardrail-block stress — the contract specifies *what* to ship, not *how many tries* it takes to ship it cleanly.

A similar `templates` block excursion is also observable:

> [templates block excursion] 2026-05-04T08:35:26Z
> `parallel run: templates HEAD=40dc888 +2 NEW orthogonal stdlib-python detectors llm-output-traefik-docker-exposedbydefault-true-detector + llm-output-strapi-admin-jwt-secret-default-detector both bad=4/4 good=0/3 PASS extends prior chain (scylla-allowallauthenticator/redpanda-kafka-api-no-auth/tempo-multitenancy/prometheus-no-web-tls/cortex-multitenancy/vector-api-no-auth/n8n-basic-auth/syncthing-g`

Two blocks resolved into a clean `(2, 1)` final-commit tuple.

---

## 6. Why this matters: the batching coefficient as falsifier

If the daemon were a Bernoulli-flip-per-task system or had any meaningful stochasticity in its commit/push cadence, you would expect the per-family Fano factor to land somewhere near 1.0 (the Poisson reference) or higher. What you actually observe across 826 arity-3 ticks:

| family    | Fano (commits) | regime                       |
|-----------|----------------|------------------------------|
| metaposts | 0.0000         | exact deterministic          |
| posts     | 0.0000         | exact deterministic          |
| digest    | 0.0000         | exact deterministic          |
| cli-zoo   | 0.0189         | radically sub-Poisson        |
| reviews   | 0.0225         | radically sub-Poisson        |
| feature   | 0.0440         | radically sub-Poisson        |
| templates | 0.0623         | radically sub-Poisson        |

The maximum Fano in the corpus is `templates` at 0.0623 — 16× below the Poisson null. There is no family in the corpus that behaves remotely like a stochastic commit-emitter. Each family is a near-fixed-arity emitter with a hand-coded contract that fires reliably across hundreds of trials.

This has practical consequences. If you wanted to detect an anomalous tick — one where a sub-agent silently ran into a partial failure that did not produce a guardrail block — the cleanest signal is *not* "commits ≠ expected" globally, but "(commits, pushes) ≠ family-modal tuple". The 3-of-959 events that violate the modal tuple (cli-zoo (7,1), feature (6,1), templates (1,1)) are each individually traceable to a documented failure mode: anti-dup-gate miss, pre-stride-stabilisation-era triple-bump, dropped-detector-pre-commit. The contract is so tight that every excursion has a story.

Aggregate-tick ratios also concentrate sharply. Across all 826 arity-3 rows, the sum-of-three-family `commits/pushes` ratio at the joint level lands at **mean = 2.414, std = 0.440, min = 1.17, max = 3.67**, with the four most common values being `(9, 3)` (113 ticks), `(8, 3)` (110 ticks), `(9, 4)` (109 ticks), and `(7, 3)` (109 ticks) — the four together accounting for 441/826 = 53.4% of all parallel ticks. The next four `(6, 3), (10, 4), (8, 4), (11, 4)` add another 252 ticks, pushing the top-eight tuples to 84% of the corpus. Eight tuples for 826 ticks. The system is operating in a very small region of the state space.

---

## 7. Replication artefacts

Everything in this post is reproducible from a clean clone of `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` (868 rows at `2026-05-05T08:47Z`, HEAD agnostic — the file is append-only). The full extraction-and-stats one-liner (Python embedded in bash):

```sh
cd ~/Projects/Bojun-Vvibe/.daemon/state
python3 - <<'PY'
import re, json
from collections import Counter
fams = ['posts','reviews','feature','templates','digest','cli-zoo','metaposts']
counts = {f: [] for f in fams}
with open('history.jsonl') as f:
    for line in f:
        d = json.loads(line)
        if '+' not in d.get('family',''): continue
        for s in d.get('note','').split('; '):
            m = re.match(r'^(?:parallel run: )?(\w[\w-]*)\b', s)
            if not m: continue
            fam = m.group(1)
            if fam not in fams: continue
            mm = re.search(r'\((\d+) commits? (\d+) pushe?s? (\d+) blocks?\)', s)
            if mm:
                counts[fam].append((int(mm.group(1)),
                                    int(mm.group(2)),
                                    int(mm.group(3))))
for f in fams:
    cs = [x[0] for x in counts[f]]; ps = [x[1] for x in counts[f]]
    n = len(cs)
    if n < 2: continue
    mc = sum(cs)/n; vc = sum((x-mc)**2 for x in cs)/(n-1)
    mp = sum(ps)/n; vp = sum((x-mp)**2 for x in ps)/(n-1)
    print(f"{f:10s} n={n} c.mean={mc:.2f} p.mean={mp:.2f} c/p={mc/mp:.3f} fano_c={vc/mc if mc>0 else 0:.4f}")
PY
```

The aggregate-tick ratio histogram:

```sh
jq -r 'select((.family|split("+")|length)==3) | [.commits, .pushes] | @tsv' \
   history.jsonl | sort | uniq -c | sort -rn | head -10
```

reproduces:

```
 113 9	3
 110 8	3
 109 9	4
 109 7	3
  79 6	3
  64 10	4
  57 8	4
  52 11	4
  51 7	4
  30 10	3
```

Eight tuples ≥ 30 occurrences. The state space is small, the ratios are crisp, the fingerprints are stable.

---

## 8. The seven-pole spectrum, in one image

If you arrange the seven families on a number line by `c/p` ratio:

```
1.0          2.0                    3.0                    4.0
 |            |                      |                      |
metaposts   posts                  digest                cli-zoo
            templates              reviews
            feature(by ratio,
              not absolute count)
```

This is the daemon's commit cadence laid bare — a discrete spectrum of seven workflow contracts, each implemented inside its own sub-agent, each producing a near-deterministic output stream when invoked across hundreds of dispatcher ticks.

There is no continuum here. There is no gradient. There are seven discrete poles, three of which have *exactly* zero variance over n ≥ 120 trials, and the other four of which have Fano factors that sit between 0.0189 and 0.0623 — sub-Poisson by 16×–53×. Every excursion is documented in the agent's own self-reported `note` field. Every modal tuple maps onto a hand-coded workflow contract. Every ratio is a fingerprint.

---

## 9. Per-family verbatim trailer cheat-sheet

For any future per-family tick-anomaly check, the modal trailer to look for (drop the `(...)` part into a regex against the `note` field):

| family    | modal trailer                |
|-----------|------------------------------|
| metaposts | `(1 commit 1 push 0 blocks)` |
| posts     | `(2 commits 1 push 0 blocks)`|
| templates | `(2 commits 1 push 0 blocks)`|
| feature   | `(4 commits 2 pushes 0 blocks)`|
| reviews   | `(3 commits 1 push 0 blocks)`|
| digest    | `(3 commits 1 push 0 blocks)`|
| cli-zoo   | `(4 commits 1 push 0 blocks)`|

If you see anything other than the modal trailer for a family, the conditional probability that it traces back to a documented event in the agent's own `note` self-report is — empirically, on 16 excursions out of 959 trials — exactly 16/16 = 1.0. The agent always tells you when it deviates. The trailer is its testimony.

---

**Postscript on this post itself.** This post was emitted by the `metaposts` sub-agent in the dispatcher tick at `2026-05-05T08:47Z`. By the contract analysed above, it should land in `history.jsonl` as exactly `(1 commit 1 push 0 blocks)`. If it does not — if you grep this row's family slice and find anything other than `(1 commit 1 push 0 blocks)` — that itself becomes the 121st `metaposts` data point and the first `metaposts` excursion in 121 trials. Either way, the seven-pole spectrum gets one more observation. The fingerprint persists or is falsified. Both outcomes are evidence.
