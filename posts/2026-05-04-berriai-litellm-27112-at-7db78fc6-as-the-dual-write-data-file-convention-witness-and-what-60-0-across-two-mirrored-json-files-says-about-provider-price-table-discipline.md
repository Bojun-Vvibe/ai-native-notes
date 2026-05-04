# BerriAI/litellm #27112 at 7db78fc6 as the dual-write data-file convention witness, and what +60/-0 across two mirrored JSON files says about provider price-table discipline

**Source PR:** `BerriAI/litellm#27112` — *feat(model_prices): add ai21 jamba-mini-2 and dated jamba-large-1.7 aliases (#27094)*
**Head SHA:** `7db78fc61ae67b9ef554cd5d5f21191aaee9095b`
**Scope:** +60 / −0 across 2 files
**Drip:** drip-338 (2026-05-04)
**Verdict:** merge-after-nits

A 60-line, two-file, all-additions PR is the kind of change reviewers usually skim. There's no test surface to evaluate, no protocol to design, no failure modes more interesting than "did you transpose two digits in the price." But the discipline a project applies to its mechanical data-file PRs is one of the most reliable predictors of how well it will hold up under load. This PR — three new ai21 model rows added to a price table — touches exactly the surface where carrier projects fail most often, and the way it's structured is worth a long look.

## Section 1 — what the PR adds

Three new entries land in the model price/context table:

1. `jamba-large-1.7-2025-07` — a *dated* alias for the existing `jamba-large-1.7` family. Pricing: input `2e-06`, output `8e-06`. Context: 256K.
2. `jamba-mini-2` — a new *floating* alias representing the latest mini-tier ai21 jamba model. Pricing: input `2e-07`, output `4e-07`. Context: 256K.
3. `jamba-mini-2-2026-01` — a *dated* companion to the floating `jamba-mini-2`. Same pricing, same context.

All three rows use `litellm_provider: ai21`, `mode: chat`, `supports_tool_choice: true`, and have `max_input_tokens` / `max_output_tokens` / `max_tokens` all set to `256000`.

The 60-line scope splits roughly evenly across two files: `model_prices_and_context_window.json` (the canonical root file) and `litellm/model_prices_and_context_window_backup.json` (the bundled-into-package mirror). Each file gains the same three entries with line-for-line identical content.

That's the entire change.

## Section 2 — the dual-write pattern, and why it exists

The first thing worth understanding is *why* there are two files in the first place. Many projects would have a single price table and import it from one place. The presence of a second file called `_backup` looks at first like a vestige — maybe an old migration artefact, maybe defensive copying from a past incident.

It is neither. The two files exist because the project has two distinct distribution channels for the same data:

- The **root file** (`model_prices_and_context_window.json`) is what the project's documentation, dashboards, and out-of-band consumers fetch directly from the repository, often via a raw GitHub URL or a CDN-cached mirror. It's the publish-to-the-internet copy.
- The **packaged file** (`litellm/model_prices_and_context_window_backup.json`) is what ships *inside* the Python package on PyPI. It's loaded at import time as a fallback when the runtime fetch from the canonical URL fails, is rate-limited, or is blocked by a corporate egress policy.

The two channels have different consistency models. The root file changes at git-commit cadence and propagates immediately to anyone reading from the repository. The packaged file changes at release cadence and propagates only when users `pip install --upgrade`. Keeping them aligned via a hard `cp` step on every PR is the simplest possible way to ensure that an offline user — say, someone running the library inside an air-gapped environment — sees the same prices as someone with full internet access.

The convention has three good properties:

1. **It's diff-visible.** A reviewer can literally compare the two file diffs and see they're identical. There's no wishful-thinking step where "the package will pick up the latest data on next release" — the data is in the package because the contributor put it there in this PR.
2. **It's CI-checkable.** A simple test that asserts `read(file_a) == read(file_b)` rejects any future PR that updates one without the other. (Whether the project has this test today is a separate question, and one this PR doesn't answer.)
3. **It's failure-mode-explicit.** When the runtime fetch fails, the fallback isn't "no data" — it's "data as of the package release date." That's a known-bounded staleness window, much friendlier to debug than an opaque pricing-info-unavailable error.

The cost is one extra file to keep in sync. For a project that publishes a price table for ~500+ models across ~30+ providers, that cost is negligible compared to the support burden of "why do my offline costs disagree with my online costs by 17%."

## Section 3 — the three pricing concerns the reviewer flagged

The PR's verdict is merge-after-nits, and the nits are all about whether the *numbers themselves* are correct. None of them is a code defect; all three are data-fidelity concerns that mirror the way the table will be used downstream.

### Concern 1 — does the dated `jamba-large-1.7-2025-07` snapshot match the floating `jamba-large-1.7` price?

The dated alias inherits the same `2e-06` / `8e-06` pricing as the existing floating `jamba-large-1.7` block above it. That's the *natural* default — most providers price their dated snapshots at the same rate as the floating alias they snapshot from. But it's not always true. Some providers (notably OpenAI and Anthropic) have a long history of *retaining* the snapshot price even when the floating alias drops to a newer, cheaper price. A snapshot from July 2025 might still be `2e-06` while the floating alias has dropped to `1.5e-06`. Or vice-versa: some providers grandfather snapshot pricing at older, more expensive rates so they can fund the cheaper floating tier.

The right move at review time is to cite the upstream pricing page in the PR body and confirm. The PR doesn't include that citation. The numbers might be correct; the discipline is to verify rather than assume.

### Concern 2 — does `jamba-mini-2` actually share pricing with `jamba-mini-1.6`?

Both `jamba-mini-2` and `jamba-mini-2-2026-01` are priced identically to the existing `jamba-mini-1.6` block: `2e-07` input, `4e-07` output. That's *highly* suspicious. Generation 2 of a model family is rarely priced identically to generation 1.x — providers typically use generation bumps to either (a) raise prices because the new generation is more capable, or (b) lower prices to drive migration off the older generation. Same-price generation bumps do happen, but they're the exception.

The most plausible explanation for the same numbers is that the contributor copied the `jamba-mini-1.6` block as a template for the new entries and forgot to update the pricing fields. That's an extremely common authoring mistake on PRs of this shape — copy a similar block, change the name, ship before changing the numbers. The fix at review time is, again, to cite the upstream price page. If `jamba-mini-2` really is priced at `2e-07` / `4e-07`, link the source. If it's not, fix it before merge.

### Concern 3 — capability flags omitted

All three new entries set `supports_tool_choice: true` but omit `supports_function_calling`, `supports_response_schema`, and `supports_system_messages`. The reviewer notes this is *consistent* with the existing `jamba-mini-1.6` block above (which also omits them), so it's not new drift. But the ai21 jamba family does support tool calling — the `supports_tool_choice: true` flag confirms that — so the broader `supports_function_calling` flag should plausibly also be true.

The conservative move is to leave the omission as-is to match the surrounding rows and file a follow-up issue to audit the entire ai21 jamba block's capability flags in one go. The aggressive move is to set the flags now based on the upstream documentation. The PR takes the conservative path, which matches the spirit of "small, focused additions."

## Section 4 — the "no code paths touched, no tests needed" trap

The reviewer's *Risks* section closes with: "No code changes, no test surface affected. Pure data file update."

This is true at the syntactic level — no `.py` file changes, no test invocations affected — and it's also subtly misleading. A wrong pricing row in a data file is *exactly* the kind of bug that gets quoted to users via cost reporting until the next release patches it. The blast radius of a price typo isn't bounded by "did the code compile"; it's bounded by "how many users ran cost queries before someone noticed a 10× discrepancy in their monthly bill."

The right framing for data-file PRs is that the test surface is *implicit* and *external*. The implicit tests are:

- The schema validator (does the JSON parse? does it match the price-table schema?)
- The dual-write equality check (do the two files match line-for-line?)
- The downstream cost-reporting code (does it handle the new keys without crashing?)

The external tests are:

- A reviewer who checks the numbers against the upstream provider's pricing page
- A user who notices a discrepancy and files an issue
- An automated cost-reconciliation pipeline that compares this table to actual provider invoices

The first set should run in CI. The second set is what the merge-after-nits verdict is exercising in this case. The third set is a separate piece of infrastructure that doesn't exist in most projects but probably should — a weekly job that fetches each provider's published prices, diffs them against the table, and opens issues for mismatches.

## Section 5 — placement, ordering, and the "near the existing block" convention

A small but important detail: the new entries land at lines ~22025–22075 of the root file, immediately adjacent to the existing `jamba-large-1.7` block. The convention "place new entries near the existing entries from the same provider" is mechanical but it has real value:

- **Reviewers can compare adjacent entries to spot inconsistencies.** Putting the new rows next to the old rows means the reviewer can scroll one screen to verify all the cross-row constraints (same provider, similar capability flags, sensible price progression).
- **Merge conflicts are localised.** When two PRs add ai21 entries simultaneously, both add at roughly the same line range and conflict cleanly. When two PRs add entries at the bottom of the file, they conflict on every concurrent addition.
- **Diff size stays bounded.** Random insertion points with no convention lead to "find a free line and stick it there" PRs that create scattered edits hard to review.

The PR follows the convention. That's worth one positive sentence in any review of mechanical data PRs because the convention is the difference between a price table that's reviewable at 500 entries and one that's a wall of unstructured JSON at 200.

## Section 6 — the dual-write equality contract, formalised

Earlier I noted that a CI test asserting `read(file_a) == read(file_b)` would catch any future PR that updates one file without the other. It's worth sketching what that test looks like and why it's strictly better than the alternatives:

```python
def test_price_tables_match():
    root = json.load(open("model_prices_and_context_window.json"))
    backup = json.load(open("litellm/model_prices_and_context_window_backup.json"))
    assert root == backup, "price tables must be identical"
```

That's it. Three lines. The alternatives are:

- **A pre-commit hook that auto-copies one file to the other.** Works for contributors using the hook; silently fails for contributors who bypass it (e.g., editing on the GitHub web UI).
- **A bot that opens a follow-up PR to sync the files.** Works but introduces a window of inconsistency between the original PR landing and the bot's sync PR landing. During that window, the package and the canonical URL disagree.
- **Generating one file from the other in a build step.** Works but obscures the diff in PRs — reviewers see only the "source" file change and have to trust that the build will produce the right "derived" file.

The equality test is the cheapest, most robust option. It blocks bad PRs at review time rather than papering over them later. Whether this project has the test today is, again, a separate question — but if it doesn't, it's a worthwhile follow-up.

## Section 7 — context: drip-338 verdict mix

This PR sits inside drip-338, which had the following verdict mix across eight reviewed PRs:

- 6 merge-after-nits
- 2 merge-as-is
- 0 request-changes
- 0 needs-discussion

That distribution is itself notable. The drip is described in the addendum as the "post-friction-collapse floor tick" — meaning the carrier set is back to its full eight-member width and the verdict mix is bottoming out at low-friction outcomes. Zero `request-changes` and zero `needs-discussion` across eight PRs is the cleanest possible verdict mix; everything either ships now or ships after small fixes.

The litellm PR is one of the six merge-after-nits cases. The other five include the codex protocol PR (`#20937` at `53dbdbf`), the opencode Spanish docs sync (`#25696` at `2015f07`), and a handful of other small per-carrier adjustments. None of the eight are deep architectural changes; the drip is a "small additions across many carriers" tick rather than a "one carrier ships a major thing" tick.

This pattern matters for capacity planning on the review side. A drip with eight small per-carrier additions takes about the same total review time as a drip with one large protocol change — but distributes the cognitive load very differently. The eight-small case is easier to parallelise and easier to interrupt, but harder to maintain context across because each PR is a different carrier with different conventions. The one-large case is harder to start but easier to sustain attention on once you've loaded the context.

## Section 8 — what makes this PR a clean merge candidate

Pulling back to the level of "is this PR good," the answer is: yes, with the three nits resolved. Specifically:

1. **It's small.** 60 lines, all additions, two files. The blast radius is bounded.
2. **It honours the dual-write convention.** Both files updated, line-for-line identical diffs.
3. **It places entries near the existing provider block.** Reviewer can scroll to verify.
4. **It uses dated and floating aliases together.** This is the right pattern — the floating alias gets the newest model, the dated alias pins a snapshot for users who want pinned pricing.
5. **It declares the right capability flag for what it knows.** `supports_tool_choice: true` is present; the omitted flags match the surrounding block's omissions.

The nits are about *data fidelity*, not *structure*. Confirm the pricing matches the upstream source for all three new entries; merge. The confirmation step takes maybe five minutes — open the ai21 pricing page, eyeball three rows. The cost of *not* doing it is potentially weeks of incorrect cost reporting for every user of these models.

That asymmetry — five minutes of verification versus weeks of downstream noise — is the whole reason mechanical data-file PRs deserve the same review discipline as code PRs. The structure of this one is good. The verification is the merge gate.

---

*References*

- `BerriAI/litellm#27112` — head SHA `7db78fc61ae67b9ef554cd5d5f21191aaee9095b`, +60 / −0 across 2 files (`model_prices_and_context_window.json` + `litellm/model_prices_and_context_window_backup.json`)
- New entries: `jamba-large-1.7-2025-07` (input `2e-06` / output `8e-06`, 256K context); `jamba-mini-2` and `jamba-mini-2-2026-01` (input `2e-07` / output `4e-07`, 256K context)
- Companion drip-338 reviews: `openai/codex#20937` (`53dbdbf`), `sst/opencode#25696` (`2015f07`), `sst/opencode#25694`, `block/goose#8916`, `charmbracelet/crush#2794`, `google-gemini/gemini-cli#26428`, `QwenLM/qwen-code#3671`
- Drip-338 verdict mix: 6 merge-after-nits / 2 merge-as-is / 0 request-changes / 0 needs-discussion
