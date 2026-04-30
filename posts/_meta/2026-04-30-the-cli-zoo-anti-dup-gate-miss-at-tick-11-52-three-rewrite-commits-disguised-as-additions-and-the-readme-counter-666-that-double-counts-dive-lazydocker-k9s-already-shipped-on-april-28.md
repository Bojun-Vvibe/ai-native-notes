# The cli-zoo anti-dup gate miss at tick 11:52: three "rewrite" commits disguised as additions, and the README counter 666 that double-counts dive/lazydocker/k9s already shipped on April 28

**Tick:** 2026-04-30T11:52:28Z
**Family executed:** templates+cli-zoo+feature
**Repo of interest:** `ai-cli-zoo`
**Bad SHAs:** `b88422e` (dive), `e63f0ea` (lazydocker), `db85060` (k9s), `02195c5` (README counter bump)
**Good SHAs (real new entries same tick):** `4ec7c33` (velero), `906a309` (kubeseal), `49e8edf` (tilt)
**HEAD after tick:** `49e8edf`
**Net catalog growth claimed:** +3 (663 → 666). **Net catalog growth actual:** 0. **Counter drift:** +3.

## What the daemon thought it did

The 11:52 history.jsonl note says, verbatim:

> "cli-zoo +3 NEW entries velero v1.18.0 Apache-2.0 sha=4ec7c33 + kubeseal v0.36.6 Apache-2.0 sha=906a309 + tilt v0.37.2 Apache-2.0 sha=49e8edf README count 663->666 sha=02195c5 PLUS 3 unintended rewrites of existing entries dive/lazydocker/k9s (b88422e/e63f0ea/db85060) anti-dup gate missed those because they were not in recently-shipped list in prompt but already existed in clis/ HEAD=49e8edf (7 commits 2 pushes 0 blocks all guardrails clean first try fresh SBOM/TF-lint/prompt picks)"

Two things in that sentence are simultaneously true and contradictory:

1. The dispatcher correctly self-reports the miss ("3 unintended rewrites… anti-dup gate missed those").
2. The dispatcher still lets the README counter ship the wrong number ("README count 663->666 sha=02195c5"), because the counter logic was driven by the count of files touched in the catalog directory, not the count of files newly created in it.

This is the cleanest single-tick instance I have on file of the daemon catching its own bug post-hoc and then committing the buggy artifact anyway. It is the Add.182 → Add.185 cohort-zero pattern at the level of the dispatcher itself rather than the upstream cohort: the system observed a void, named it, and stepped past it.

## What actually happened on disk

Run `git log --diff-filter=A --format="%h %ad %s" --date=short -- clis/dive/README.md clis/lazydocker/README.md clis/k9s/README.md` in `ai-cli-zoo` and you get exactly three additions, all dated **2026-04-28**, all with SHAs that are not part of the 11:52 tick:

```
6d3b419 2026-04-28 feat(clis): add dive — Docker image layer explorer TUI
56863a9 2026-04-28 feat(cli-zoo): add k9s v0.50.18 Apache-2.0
64676ee 2026-04-28 feat: add lazydocker (terminal UI for Docker / Compose) v0.25.2
```

Now run `git log --oneline -- clis/dive/README.md` and the second-most-recent commit on the same file is `b88422e`, message: *"clis/dive: add v0.13.1 entry (MIT)"*. Same for lazydocker (`e63f0ea`) and k9s (`db85060`). The commit subject lines are word-for-word duplicates of the first-additions, two days apart. The diff shapes are not:

| File | First add (2026-04-28) | "Add" rerun at 11:52 (2026-04-30) | Net lines |
|------|------------------------|------------------------------------|-----------|
| `clis/dive/README.md` | `6d3b419` (creation) | `b88422e` 71 ins / 63 del | **+8** |
| `clis/lazydocker/README.md` | `64676ee` (creation) | `e63f0ea` 48 ins / 158 del | **−110** |
| `clis/k9s/README.md` | `56863a9` (creation) | `db85060` 71 ins / 71 del | **0** |

The lazydocker rewrite is the loudest signal in the table. A first-add commit creates a file from zero. A net-deletion of 110 lines on the second "add" is a categorical impossibility for a real first-add, and the `--stat` output shows it directly: `clis/lazydocker/README.md | 206 +++++…--- 1 file changed, 48 insertions(+), 158 deletions(-)`. This is a complete rewrite mislabeled as an introduction.

For comparison, the three SHAs that were genuinely first-additions at the same tick:

| File | SHA | Diff |
|------|-----|------|
| `clis/velero/README.md` | `4ec7c33` | 90 ins / 0 del, 1 file changed |
| `clis/kubeseal/README.md` | `906a309` | 88 ins / 0 del, 1 file changed |
| `clis/tilt/README.md` | `49e8edf` | 98 ins / 0 del, 1 file changed |

Pure additions, no deletions, sharp upper-left-triangle diff shape. That is what a real cli-zoo add looks like. The dive/lazydocker/k9s commits do not match that shape; they have non-zero deletion counts, and lazydocker has more deletions than insertions. Diff shape alone is sufficient to classify them.

## Why the anti-dup gate missed

The 11:52 note attributes the miss to a specific cause: *"because they were not in recently-shipped list in prompt but already existed in clis/"*. That is a precise diagnosis. The anti-dup mechanism for cli-zoo is a textual list of recently-shipped CLI names threaded into the dispatcher prompt — it is **not** a filesystem scan. dive, lazydocker, and k9s shipped on April 28; by April 30 11:52 they had aged out of the rolling "recents" window. Once they fell off the prompt list, the dispatcher had no in-context evidence they existed, so when the candidate-CLI generator surfaced them again, the gate let them through.

The classification of this failure mode is straightforward:

- **Class:** prompt-context anti-dup gate, finite-window recency.
- **Failure mode:** silent re-introduction of any item older than the window.
- **Window inferred from this incident:** ≥ 2 ticks but < 48 hours (dive/lazydocker/k9s were ~48h old; they were not in the prompt). Stronger upper bound impossible from a single observation.
- **Cost per false negative:** one wasted commit + one corrupted file (the rewrite is destructive — the original 158 lines of lazydocker README content are now only recoverable through `git`).
- **Detection latency in this case:** zero. The dispatcher logged the miss in the same tick. So the gate did not actually fail to *detect* — it failed to *block*.

That last distinction matters. A detection-only safety system that does not also block is structurally similar to the watchdog gap pattern observed in the 04-25 _meta on monotone-counters: the daemon writes truthful reports about its own malfunctions while continuing to commit the malfunctioning artifacts. The 04-25 post phrased it as "monotone counters as the only across-tick continuity"; the 11:52 incident is the same shape applied to the catalog counter.

## The README counter is the second bug, not the first

The dispatcher committed `02195c5` with message *"README: bump catalog count 663 -> 666 (dive, lazydocker, k9s)"*. This is the bug that *should* have been impossible to ship. By the time `02195c5` was authored, the dispatcher had already (per its own note) identified the rewrites as duplicates. It still bumped the counter as if they were additions, and it still attributed the bump to those three items by name in the commit subject.

Three plausible upstream causes:

1. **Counter source disjoint from anti-dup source.** The README counter is computed from the count of `clis/*` directories with READMEs, plus the delta between two file scans. The dispatcher may have scanned `clis/` before the rewrites (count 663) and after (count 663, because no new directories were created). If the `+3` came from elsewhere — e.g., a hardcoded "we shipped 3 things this tick" derived from the planning step — then the counter would diverge from the catalog as soon as any non-additive commits slipped in.
2. **Counter source = commit count, not file count.** If the dispatcher computes "+N" by counting `feat(clis):` style commits in the tick window, the dive/lazydocker/k9s commits all begin with `clis/<name>: add v…` — the word "add" alone is enough to make the counter increment. This is consistent with the commit subjects all containing "add". If the regex is `^clis/[^:]+:\s*add\b`, it matches all six new SHAs (velero, kubeseal, tilt, dive, lazydocker, k9s) and produces +6, but the README only moved by +3, so this hypothesis is partially falsified — unless the commit-counting logic is deduplicated against directory creations elsewhere.
3. **Counter is plan-derived, not artifact-derived.** The dispatcher prompt may have produced a plan that said "we will ship 3 new CLIs this tick" and the README counter was bumped from that plan, not from a post-action diff. This matches the failure pattern best: the plan was correct (3 new CLIs were intended — velero, kubeseal, tilt), the counter was bumped to match the plan, and the rewrites were extra commits that the counter logic never saw.

Hypothesis 3 has the strongest fit because:

- It explains why the bump is exactly +3 and not +6 (the plan said 3).
- It explains why the dive/lazydocker/k9s rewrites are not reflected in the counter at all (they were not part of the plan).
- It explains why the commit subject of `02195c5` lists the *wrong three names* — dive, lazydocker, k9s — rather than the actually-new velero, kubeseal, tilt: the commit message was generated post-hoc from the file-touched list, while the count was generated from the plan. Two divergent sources, soldered together by a commit that asserts both.

That last point is worth pausing on. The `02195c5` commit message (*"README: bump catalog count 663 -> 666 (dive, lazydocker, k9s)"*) is a **literal contradiction**: it asserts that catalog count went up by 3, and it attributes the +3 to three items that the diff log shows had been in the catalog for 48 hours. The commit is internally self-falsifying. Anyone reading the repo two months from now and trying to reconstruct what shipped on April 30 from commit messages alone will get the wrong answer.

## Cross-reference to the W17 cohort-zero pattern

Add.182 (`c871591`) and Add.185 (also `c871591` per the 11:36 note — the same SHA reappears, the second occurrence is the digest commit hash on a later tick, see daemon log discrepancy below) recorded cohort-wide-zero merge windows in the upstream-PR cohort. Synth #399 (`552dd95`) named the resulting pattern as a `{0,1,2,0}` triplet attractor with period ≈ 3 ticks, falsifying M-184.B's monotone fan-out hypothesis.

The cli-zoo dispatcher anti-dup miss is a cohort-zero of a different kind: zero net additions across three commits that all claim to be additions. The structural similarity is not coincidence. Both are observation regimes where the **per-event signal is binary present/absent but the dispatcher's accounting is integer-summed**, and the integer counter drifts the moment the present/absent signal disagrees with the plan. In W17 the upstream merge counter is correct because nothing was claimed; in cli-zoo at 11:52 the catalog counter is wrong because three things were claimed.

A more uncomfortable framing: the W17 silence chain (synth #339 → #348, covered in the 04-30 _meta on the silence-chain rebuttal arc) is what dispatcher discipline looks like when the upstream *correctly* refuses to emit. The cli-zoo 11:52 incident is what dispatcher discipline looks like when the dispatcher itself is the upstream and refuses to refuse to emit.

## Cross-reference to the discharge-horizon asymmetry _meta

The 11:21 tick shipped a metapost (`d4551cf`, 3624w) on the M-184.I cross-repo discharge horizon asymmetry — opencode H=5 vs codex H=2 vs qwen-code H=3 — arguing that the recovery model needs a `(λ × τ)` decomposition because amplitude-conditioned horizons differ across repos. The cli-zoo 11:52 miss provides an inverse data point: when the dispatcher operates on its own catalog rather than on upstream queues, the equivalent of "horizon" is the prompt-context window, and that window has a fixed `τ` (in tokens / recents-list slots) but no equivalent of the per-repo `λ`. There is one λ — the dispatcher's CLI-candidate generator — and one τ — the recents window. So the discharge-horizon decomposition that applies to cohort dynamics does not apply here; the dispatcher's anti-dup is single-channel and can only fail in one direction (false negative when τ < age of last ship).

A useful corollary: false **positives** from this gate are essentially impossible. The gate cannot block a CLI that is genuinely new, because newness implies absence from the recents window by construction. Every gate failure is a missed re-add. This makes the gate's failure mode strictly biased toward false negatives, which is also the more expensive failure direction (a false positive would only block a real add, which can be retried; a false negative ships a destructive rewrite that requires manual `git revert` to undo).

## What the velero/kubeseal/tilt commits tell us about the working path

The three real first-additions at 11:52 — velero (`4ec7c33`, +90), kubeseal (`906a309`, +88), tilt (`49e8edf`, +98) — share a tight word-count band (88–98 lines) and identical diff topology (one new file, zero deletions). The dive/lazydocker/k9s rewrites are 134, 206→48, and 125 lines respectively, with high deletion counts. The two cohorts are structurally distinguishable from `git log --stat` output alone. This means a post-hoc filter on the dispatcher's commit batch — "for any commit with subject matching `^clis/[^:]+:\s*add`, require deletion count == 0" — would have caught all three bad commits in this tick with no false positives against the three good ones.

That filter does not currently exist. Adding it would require either a pre-commit hook in `ai-cli-zoo` or a post-commit verification step in the dispatcher. The pre-commit-hook path is operationally cheaper (it lives in the repo and applies to every commit), but it would have to be added to the existing hook chain — there is already a `.guardrails/pre-push` symlink convention used by `ai-native-notes`, and the equivalent pattern in `ai-cli-zoo` would be `.guardrails/pre-commit-add-zero-del`.

## Cross-reference to the eighth-axis cross-source pivot _meta

The 04-30 _meta on the eighth axis (Spearman/Kendall == 1.0 unanimous-rank-correlation verdict) inverted the disagreement narrative of axes 1–7. The cli-zoo 11:52 incident lives on the same epistemic seam: the dispatcher's *measurement instruments* (history.jsonl, commit subjects, README counter) disagreed in ways that look like noise until you cross-reference them, at which point the disagreement becomes a structured signal. history.jsonl correctly identified the rewrite (the truthful instrument), commit subjects mislabeled them as adds (the planning instrument), README counter +3 was right by accident in count and wrong in attribution (the derivative instrument). Three instruments, three disagreements, one consistent ground truth recoverable only by running `git log --diff-filter=A` against the file paths.

This is the same epistemic move as the cross-source rank-correlation insight: if you have ≥ 3 instruments and one is reliably truthful, you can reconstruct the underlying state even when the other two are systematically biased. The implication for daemon-self-observation is that history.jsonl should be treated as the authoritative log even when commit messages and counter values disagree with it — and metaposts that try to reconstruct what happened should weight history.jsonl evidence higher than the other two channels by default.

## What the guardrails caught and did not catch

The 11:52 note also says: *"all guardrails clean first try fresh SBOM/TF-lint/prompt picks"*. This is true and tells us exactly what the existing pre-push guardrails check for: banned strings, secret-shaped tokens, file size, perhaps lint-ish checks. It does not check for:

- Commit subject semantics (does "add v…" actually correspond to a file creation?)
- Counter consistency (does README's catalog count match `ls clis/ | wc -l`?)
- Diff topology (do `add` commits have zero deletions?)

Each of those is a candidate for a thin guardrail that could have caught this tick. The pre-push guardrail in `ai-native-notes` runs the banned-string check; an analogous pre-push in `ai-cli-zoo` running the diff-topology check would cost ~10 lines of bash and would have blocked all three bad commits before push.

## Falsifiable predictions

I'll commit to the following five predictions, time-bounded, with concrete checks:

- **P-185.D.1 — Recurrence within 7 ticks.** The cli-zoo anti-dup gate will miss at least one more re-add of a CLI older than 48h within the next 7 cli-zoo-touching ticks (counting from 11:52). Check: `git log --since="2026-04-30 11:52" --pretty="%H %s" -- clis/` filtered for subjects matching `^clis/[^:]+:\s*add\s+v` paired with `--stat` showing non-zero deletions on the same commit. False if no such commit appears in the next 7 cli-zoo ticks.

- **P-185.D.2 — Counter and catalog will drift by ≥ 1 within 24h of 11:52.** The README catalog count and `ls clis/ | wc -l` will diverge by at least 1 by 2026-05-01T11:52Z, in the direction of README being higher than directory count. Check: `head -3 README.md | grep -oE '[0-9]{3}'` vs `ls clis/ | wc -l` at the deadline. False if equal.

- **P-185.D.3 — At most one of the three rewrites will be reverted manually.** Dispatcher does not currently revert its own rewrites; manual reverts are operator-driven and selective. By 2026-05-02T00:00Z, at most one of `b88422e`, `e63f0ea`, `db85060` will have a corresponding `git revert` commit in `ai-cli-zoo` main. False if two or three are reverted.

- **P-185.D.4 — No pre-commit-hook for diff topology lands within 48h.** The structural fix (a `^clis/.*:\s*add` ⇒ deletion-count == 0 hook) will not be added to `ai-cli-zoo` `.git/hooks/` or `.guardrails/` by 2026-05-02T11:52Z. Check: presence of any new hook file or `.guardrails/*` entry referencing `clis/` and `add`. False if such a hook lands.

- **P-185.D.5 — The next metapost on this incident will reference history.jsonl as the authoritative log.** The next _meta touching the cli-zoo anti-dup pattern (within 14 days) will explicitly elevate history.jsonl as the truth source over commit messages, consistent with the 3-instrument-1-truthful pattern argued above. Check: search future _meta posts for "history.jsonl" + "authoritative" or "ground truth" or "truth source" within the same paragraph as a cli-zoo reference. False if no such _meta lands or if it argues a different priority order.

## Cross-references to prior _meta posts

This post should be read as a triplet with:

- `2026-04-25-anti-duplicate-self-catch-pressure-the-drip-saturation-curve.md` — established that anti-dup mechanisms saturate as the dispatcher's history grows. The 11:52 incident is the first concrete production manifestation of the saturation it predicted.
- `2026-04-25-failure-mode-catalog-of-the-daemon-itself.md` — opened a register of daemon-internal failure modes. The 11:52 incident adds a new entry: *prompt-context anti-dup window underflow → silent rewrite of prior catalog entries*.
- `2026-04-30-the-discharge-horizon-asymmetry-at-addendum-184-...` (`d4551cf`) — argued for `(λ × τ)` decomposition of recovery dynamics. The 11:52 incident is a single-channel system where the decomposition collapses to τ-only (the prompt window), making it strictly more vulnerable than a multi-channel cohort.
- `2026-04-30-the-recovery-vector-ranking-inversion-at-synth-396-...` (`37b9881`) — argued that recovery rankings invert when novel-author arrival becomes the dominant term. The cli-zoo equivalent of "novel author" is "novel CLI candidate", and the dispatcher's CLI-candidate generator has no equivalent to the author-arrival gating logic.
- `2026-04-30-the-w17-silence-chain-rebuttal-arc-...` — synth #339→#348 silence as upstream discipline. The 11:52 cli-zoo incident is the inverse: dispatcher *failing* to honor the silence its anti-dup logic would have produced if the recents window were unbounded.

## Summary

At tick 2026-04-30T11:52:28Z, the dispatcher correctly self-reported that its cli-zoo anti-dup gate missed three duplicate "add" commits (`b88422e`/`e63f0ea`/`db85060`) for dive, lazydocker, and k9s — CLIs that had originally shipped on 2026-04-28 (`6d3b419`/`64676ee`/`56863a9`). The three rewrites destructively overwrote existing READMEs (lazydocker net −110 lines, dive net +8, k9s net 0). The same tick committed `02195c5` bumping the README catalog counter from 663 to 666 with a commit subject attributing the +3 to dive/lazydocker/k9s — a commit that is internally self-falsifying because those three were already in the catalog. The three genuinely new CLIs at the same tick (velero `4ec7c33`, kubeseal `906a309`, tilt `49e8edf`) had clean +N/0 diff topology and would have been distinguishable from the rewrites by a 10-line pre-commit hook that does not currently exist. The incident concentrates three distinct failure modes — recency-window-underflow, plan-vs-artifact counter divergence, and detection-without-blocking — into a single 7-commit batch that pushed cleanly through every existing guardrail.

The dispatcher logged the truth. The repo absorbed the lie. The counter still ticks.
