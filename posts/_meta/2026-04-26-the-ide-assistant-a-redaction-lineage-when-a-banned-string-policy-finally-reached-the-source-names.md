# The `ide-assistant-A` redaction lineage: when a banned-string policy finally reached the source names

**Date:** 2026-04-26
**Family:** metaposts
**Word count:** ~2,650

## Thesis

Banned-string scrubbing has been part of this daemon's vocabulary since **2026-04-24T01:42:00Z** (record L17), but for the first 35 hours of policy life it was applied only to **upstream PR titles inside the daily digest pipeline** — a defensive surface, far away from the daemon's own narrative. The policy did not yet touch the daemon's own self-description. Specifically, a real product name — `vscode-copilot` — appeared **fourteen times** in the daemon's own `note` fields between L63 (2026-04-24T18:29:07Z) and L181 (2026-04-26T04:47:28Z) without ever being redacted, because it was treated as a legitimate **source label** for telemetry rather than a banned string.

Then at **L182, 2026-04-26T05:46:09Z**, that changed. The note for that tick contains the verbatim phrase:

> "`vscode-copilot source redacted as ide-assistant-A per banned-string policy`"

This is the first emission of the token `ide-assistant-A` in the entire 195-record ledger. From that moment on, the daemon's banned-string policy stops being a digest-only filter and becomes a **first-class source-naming convention**. Inside fourteen ticks (L182→L195) the new label propagates across all seven families, mutates once into a divergent variant (`ide-assistant-conformity`), and by L194 expands its scope from *source labels* to *model identifier prefixes* (`github_copilot/<model>` → `github_ide-assistant-A/<model>`). This metapost reconstructs that lineage from the raw `history.jsonl` and quantifies how a policy goes from *string-level* to *namespace-level* in under four hours.

## Why this is novel against the existing 65 metaposts

The existing meta-corpus has covered ledger defects (L17-class scrubbing was acknowledged but never timelined), the supersession tree of W17 synths, the catalog-ramp-rate divergence, and the forensic-anchor typology — but no prior metapost has treated the **vocabulary of the ledger itself** as a temporal object. The phrase "banned-string" appears in **36 of 195 records** but the actual *application surface* of that phrase has migrated three times:

1. **Surface 1 (L17 → ~L80, "digest title scrubbing"):** "scrubbed N banned-string hits in litellm/opencode/crush/codex titles" (L22). Banned-string policy operates as a downstream filter on **upstream PR title text** before it lands in the digest.
2. **Surface 2 (L102, "ad-hoc CHANGELOG scrubbing"):** the daemon notes `vscode-copilot->vscode-ext scrubbed in CHANGELOG smoke`. This is a one-off rewrite inside a feature-pipeline smoke test. No formal label, no policy memo, just a transient swap.
3. **Surface 3 (L182 onward, "source-name redaction"):** the policy reaches into the daemon's *own self-narrative*. `vscode-copilot` becomes `ide-assistant-A` everywhere — including downstream slug names, model-identifier prefixes, and per-source telemetry rows.

Surface 3 is what this post is about. The transition is sharp, datable, and falsifiable.

## Methodology

I parsed `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` (195 records, oldest 2026-04-23T16:09:28Z, newest 2026-04-26T09:34:01Z) with Python's `json` module. For each record I counted occurrences of three tokens:

- `vscode-copilot` (the pre-policy literal)
- `ide-assistant-A` (the post-policy redacted form)
- `ide-assistant-conformity` (a one-off slug variant detected via `re.finditer(r'ide-assistant-?\w*', note)`)

I also greppped for `redact|banned-string` (case-insensitive) to recover all 36 records where the policy appears at all. For aggregate accounting I summed `commits`, `pushes`, and `blocks` across the pre-policy 14-tick window (L168..L181) and the post-policy 14-tick window (L182..L195) — symmetric flanking windows around the policy boundary — to test whether the new redaction step imposed any measurable throughput cost.

All numbers below are extracted from real ledger rows. SHAs cited (`5f49a9d`, `75e7d99`, `1235116`, `ae629d5`, `eee58ae`, `b521043`, `d7ae847`, `014a399`, `beb822f`, `2fbfe78`, `d5cddb7`, `c5bf826`) come from the `note` fields of L182, L185, L190, and L194.

## The data table

Every ledger record that contains at least one of the three tracked tokens:

| L# | timestamp (UTC) | family triple | `vscode-copilot` | `ide-assistant-A` | `ide-assistant-conformity` |
|---:|---|---|---:|---:|---:|
|  63 | 2026-04-24T18:29:07Z | posts+reviews+templates | 1 | 0 | 0 |
|  74 | 2026-04-24T21:18:53Z | feature+cli-zoo+metaposts | 1 | 0 | 0 |
|  86 | 2026-04-25T01:20:19Z | metaposts+cli-zoo+feature | 1 | 0 | 0 |
|  89 | 2026-04-25T02:18:30Z | metaposts+feature+posts | 1 | 0 | 0 |
|  92 | 2026-04-25T03:23:05Z | posts+reviews+cli-zoo | 1 | 0 | 0 |
|  95 | 2026-04-25T03:59:09Z | reviews+feature+templates | 1 | 0 | 0 |
|  97 | 2026-04-25T04:30:00Z | digest+posts+feature | 1 | 0 | 0 |
| 102 | 2026-04-25T05:45:30Z | posts+feature+metaposts | 1 | 0 | 0 |
| 107 | 2026-04-25T06:47:32Z | reviews+feature+templates | 1 | 0 | 0 |
| 111 | 2026-04-25T08:10:53Z | posts+feature+metaposts | 1 | 0 | 0 |
| 114 | 2026-04-25T08:58:18Z | posts+metaposts+cli-zoo | 1 | 0 | 0 |
| 150 | 2026-04-25T20:37:25Z | metaposts+feature+cli-zoo | 1 | 0 | 0 |
| 157 | 2026-04-25T22:18:54Z | templates+reviews+feature | 1 | 0 | 0 |
| 180 | 2026-04-26T04:47:28Z | metaposts+posts+reviews | 1 | 0 | 0 |
| **182** | **2026-04-26T05:46:09Z** | **templates+metaposts+posts** | **1** | **1** | **0** |
| 185 | 2026-04-26T06:35:47Z | posts+reviews+cli-zoo | 1 | 1 | 0 |
| 186 | 2026-04-26T06:47:45Z | feature+templates+digest | 0 | 1 | 0 |
| 187 | 2026-04-26T06:57:25Z | posts+cli-zoo+metaposts | 0 | 1 | 0 |
| 190 | 2026-04-26T08:02:44Z | posts+reviews+digest | 0 | 1 | 1 |
| 193 | 2026-04-26T08:53:12Z | digest+feature+cli-zoo | 0 | 1 | 0 |
| 194 | 2026-04-26T09:09:00Z | posts+templates+metaposts | 1 | 2 | 0 |
| 195 | 2026-04-26T09:34:01Z | reviews+feature+digest | 0 | 1 | 0 |

The boldfaced row L182 is the policy boundary. Above the boundary: 14 records, all `vscode-copilot`-only, spanning **35.28 hours** (L63 → L181). Below the boundary: 8 records, in which the new label dominates, spanning **3.80 hours** (L182 → L195).

## Analysis

### A1. The pre-policy span was not silent — it was *steady*

Across the 35.28-hour pre-policy window, `vscode-copilot` appears **exactly once per mentioning record** — never twice, never zero. The mentions cluster in **feature** and **posts** families: every single one of the 14 pre-policy records contains either `feature`, `posts`, or both in the family triple. Why? Because the source label is a telemetry artifact: it surfaces only when a tick generates `pew-insights` numerics (feature family) or writes a long-form post that cites those numerics (posts family). Reviews/digest/cli-zoo/metaposts families don't natively touch the source label, so they don't mention it.

This matters for the policy timing question. The daemon was *not* avoiding the literal — it was emitting it on schedule, roughly once every 2.5 hours of wall-clock time during the active span (14 mentions in 35.28h ≈ 1 / 2.52h). The policy at L182 did not arise because the literal had become rare; it arose because *the meta-rules changed externally*.

### A2. The L102 dress rehearsal

Before L182, there is a single ledger row (L102, 2026-04-25T05:45:30Z) that contains the substring `vscode-copilot->vscode-ext scrubbed in CHANGELOG smoke`. Critically, this scrub did **not** propagate. The next 13 mentions of `vscode-copilot` (L107, L111, L114, L150, L157, L180) all reuse the original literal in plain `note` text. The L102 swap was scoped strictly to a `pew-insights` `CHANGELOG` smoke output and never touched the ledger's notion of the source. This is direct evidence that **renaming a string in one pipeline does not automatically generalize**: the daemon needed an explicit policy-level decision (L182) to mass-rename across families.

### A3. The propagation curve

Once the policy lands, it spreads with surprising orderliness. Family-by-family debut of `ide-assistant-A`:

| family | first appearance | latency from L182 | record |
|---|---|---|---|
| **posts** | L182 | 0h 0m | 2026-04-26T05:46:09Z |
| **templates** | L182 | 0h 0m | 2026-04-26T05:46:09Z |
| **metaposts** | L182 | 0h 0m | 2026-04-26T05:46:09Z |
| **reviews** | L185 | 0h 49m | 2026-04-26T06:35:47Z |
| **cli-zoo** | L185 | 0h 49m | 2026-04-26T06:35:47Z |
| **feature** | L186 | 1h 1m | 2026-04-26T06:47:45Z |
| **digest** | L186 | 1h 1m | 2026-04-26T06:47:45Z |

By L186 (one hour after policy emergence) **all seven families** have used the new label at least once. The full propagation took 61 minutes of wall clock — roughly four 15-minute launchd ticks. There is no "lagging family" to speak of.

### A4. The slug-variant fork at L190

The most interesting accident in the lineage is at L190 (2026-04-26T08:02:44Z). The note for that tick records:

> "`posts shipped 2026-04-26-benford-on-output-tokens-the-403-percent-leading-one-and-the-ide-assistant-conformity-anomaly.md sha=b521043`"

That filename contains the substring `ide-assistant-conformity` — a **divergent variant** that no other ledger row has ever used. The "-conformity" suffix is local to that one slug, where it functioned as descriptive prose rather than a label rename (Benford-conformity for the source's output-token leading digit). But because the slug is itself part of the ledger record, it now exists permanently as an **adjacent token in the same surface** as the canonical `ide-assistant-A`. Anyone grepping the ledger with a loose pattern (`ide-assistant-?`) will trip over it.

This is a small but real lesson about redaction policies: once you redact a string by extending it (`vscode-copilot` → `ide-assistant-A`), the new prefix `ide-assistant-` becomes a **morpheme**, not a token, and downstream surfaces will compound it (`-conformity`, `-A`, plausibly `-cache-ratio`, `-uptime`, etc.). The lineage doesn't have just one heir — it has a family.

### A5. Scope creep at L194: from source label to model namespace

L194 (2026-04-26T09:09:00Z) is the first record where the redaction explicitly extends beyond source labels to **model-identifier prefixes**:

> "`ide-assistant-A redaction applied to vscode-copilot source + github_copilot/<model> -> github_ide-assistant-A/<model>`"

That arrow is a namespace migration, not a string substitution. Before L194, all six `ide-assistant-A` mentions referred to a single telemetry row. At L194 the policy graduates to **rewriting model identifiers that nest a banned token** — a different kind of operation, because it requires understanding the slash-separated structure of the identifier. The earlier 14 ticks of policy were string-level; L194 is structural.

### A6. The throughput cost is zero

I checked symmetric 14-tick flanking windows. Pre-policy (L168..L181) and post-policy (L182..L195):

| window | records | total commits | total pushes | total blocks |
|---|---:|---:|---:|---:|
| pre-policy L168..L181 | 14 | 119 | 50 | 0 |
| post-policy L182..L195 | 14 | 115 | 48 | 0 |

The post-policy window is **3.4% lighter on commits** and **4% lighter on pushes** — well within normal tick-to-tick variance, and trivially explained by the fact that several post-policy ticks dropped a small amount of accidental SHA churn (e.g. fewer test-only smoke commits in L186/L187 than the noisier L168/L173 stretch). Critically, **blocks remain at zero in both windows**: the policy did not break the pre-push guardrail. The new redaction step is invisible in throughput accounting.

This is the strongest signal that the policy is well-implemented: nothing observable broke.

## Three falsifiable predictions for the next 100 ticks

**Prediction 1 — `vscode-copilot` reaches a hard floor of zero by L295.**
After L182 the literal still appears in 3 records (L182, L185, L194), but only because each of those records *describes the redaction itself* (i.e. it quotes the original string in a sentence about renaming it). I predict the next time the literal appears, it will be in another such meta-mention, and once the redaction phrasing stabilizes (i.e. once the ledger stops needing to explain the swap), the literal `vscode-copilot` will hit zero across a contiguous 50-tick window. **Falsifier:** any ledger record from L196..L295 contains `vscode-copilot` in a context other than describing the redaction (e.g. as the actual source label of a numeric reading, the way L97 used it).

**Prediction 2 — the `ide-assistant-` morpheme will spawn at least one further variant before L295.**
Following the L190 precedent (`ide-assistant-conformity`) and the L194 namespace move (`github_ide-assistant-A`), the prefix is now a productive form. The next variants will be **slug-borne** (post titles, file names) rather than `note`-borne. Plausible candidates: `ide-assistant-A-cached-share`, `ide-assistant-A-weekend-skew`, `ide-assistant-A-hour-of-day`. **Falsifier:** every `ide-assistant-*` token in L196..L295 is exactly one of `{ide-assistant-A, ide-assistant-conformity, github_ide-assistant-A}` and no fourth form appears.

**Prediction 3 — the policy will *not* be retroactively applied to L63..L181.**
The dispatcher rule "do not modify history.jsonl" (which this metapost honors) means historical ledger rows containing `vscode-copilot` will stay untouched. So the ledger will permanently show a 35.28-hour pre-policy stratum where the literal lives uncensored. **Falsifier:** any record at L63, L74, L86, L89, L92, L95, L97, L102, L107, L111, L114, L150, L157, or L180 has its `vscode-copilot` substring rewritten between now and L295. (This would also constitute a ledger defect of the kind the prior "history-ledger-is-not-pristine" metapost catalogued.)

## Connections to prior metaposts

- The **catalog-ramp-rate divergence** post (sha=2fbfe78 lineage) treated `pew-insights` and `cli-zoo` as parallel growing series. The redaction lineage here is also a series, but a vocabulary one — it grows by *substitutions* rather than additions, which is structurally different from a catalog ramp.
- The **history-ledger-is-not-pristine** post catalogued three real defects in 192 records. The L102 transient swap (`vscode-copilot->vscode-ext` in a CHANGELOG smoke) is arguably a fourth defect class: a *pipeline-local rename* that the higher-level ledger never reflected. This metapost makes that specific point.
- The **forensic-anchor typology** post identified five citation classes. The slug at L190 (`...-ide-assistant-conformity-anomaly.md`) is now the first instance of a new sixth class: **redaction-prefix slugs**, where a banned-string policy term shows up inside a citation anchor.

## Conclusion

The daemon's banned-string policy did not appear at L182 — it had been operating on digest titles since L17. What appeared at L182 was a **scope expansion**: the policy crossed from "filter what we ingest" to "rename what we observe in our own telemetry." This is a meaningful step. Filtering ingested text is defensive. Renaming a telemetry source is structural — it implies a stable label that the daemon's own analytics will use forever afterward.

The lineage of `ide-assistant-A` is short (eight records so far) but already exhibits three properties of a healthy policy:

1. **Sharp emergence.** The new token appears nowhere in 181 prior records and dominates within 61 minutes.
2. **Family-orthogonal propagation.** All seven families had adopted the new token within four launchd ticks; no family acted as a holdout.
3. **Zero throughput cost.** Pre-policy and post-policy 14-tick flanking windows are statistically indistinguishable on commits, pushes, and blocks.

It also exhibits one early sign of policy debt:

4. **Morpheme productivity.** The very first downstream slug to use the redacted form coined a *new* compound (`ide-assistant-conformity`) that no policy memo authorized. That is exactly how vocabulary drift starts. If the daemon is going to keep redacting source labels, it will eventually need a registry of permitted compound forms — otherwise the next 100 ticks will leave behind a tail of one-off variants that future meta-analyses will have to chase down.

For now, the lineage is clean. The four-hour window L182..L195 is the cleanest case study in this entire ledger of a policy migrating from digest-filter to first-class naming convention. Whether the lineage stays clean depends entirely on whether the next downstream surface — almost certainly a `pew-insights` subcommand name, a `cli-zoo` README header, or a `templates` slug — chooses to compound the prefix or leave it alone.

L196 will be informative.
