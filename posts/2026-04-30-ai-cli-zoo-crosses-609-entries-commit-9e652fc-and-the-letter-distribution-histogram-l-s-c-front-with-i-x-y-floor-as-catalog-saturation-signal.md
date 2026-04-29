# ai-cli-zoo Crosses 609 Entries (commit 9e652fc): What the Letter-Distribution Histogram Says About Catalog Saturation, the L-S-C Front, and the Sparse-Letter Long Tail

**Date:** 2026-04-30
**Repo:** `ai-cli-zoo`
**Bump commit:** `9e652fc` — `docs(cli-zoo): bump count to 609 (serie, tlrc, iredis)`
**Triplet feat commits:** `bb68f49` (`feat(cli-zoo): add iredis v1.16.0`), `790f4d8` (`feat(cli-zoo): add tlrc v1.13.0`), `f4207be` (`feat(cli-zoo): add serie v0.7.2`)

---

## 1. The milestone

The catalog is a long-running, deeply-annotated comparison of terminal-shaped tools that orbit the AI-coding-agent space — most of them are not themselves AI agents but are the building blocks (search, edit, log graphing, terminal fetchers, system monitors, secret scanners, link checkers, etc.) that those agents call out to. The README opens with the literal sentence "A curated, deeply-annotated catalog comparing **609 AI coding CLIs**." That number — 609 — was bumped from 606 in commit `9e652fc` after three back-to-back additions: `serie` (a rich `git log --graph` reimplemented as a fast TUI, v0.7.2, feat SHA `f4207be`), `tlrc` (the official Rust client for `tldr-pages`, v1.13.0, feat SHA `790f4d8`), and `iredis` (an interactive Redis client with auto-completion, v1.16.0, feat SHA `bb68f49`).

The bump is small (+3 entries in one tick) but it crosses a structural threshold: the catalog has moved from "in the high 500s" to "comfortably above 600." That changes how the catalog should be *read*, not because 609 vs. 606 is a meaningful difference per se, but because it lets us look at the *shape* of the catalog with enough sample to talk about saturation.

This post asks one question: **as the catalog grows past 600, where is it filling in, and where is it still sparse?** The lever we have is the first-letter distribution of the entry directories under `clis/`, which gives a clean histogram with one bucket per leading character.

## 2. The histogram (verbatim from `ls clis/ | awk '{print substr($1,1,1)}' | sort | uniq -c`)

At the time of writing, the buckets are:

| Letter | Count |
|--------|-------|
| `0`    | 1     |
| `a`    | 41    |
| `b`    | 18    |
| `c`    | 52    |
| `d`    | 36    |
| `e`    | 11    |
| `f`    | 21    |
| `g`    | 47    |
| `h`    | 19    |
| `i`    | 4     |
| `j`    | 10    |
| `k`    | 15    |
| `l`    | 59    |
| `m`    | 41    |
| `n`    | 7     |
| `o`    | 26    |
| `p`    | 36    |
| `q`    | 5     |
| `r`    | 24    |
| `s`    | 48    |
| `t`    | 41    |
| `u`    | 6     |
| `v`    | 14    |
| `w`    | 11    |
| `x`    | 4     |
| `y`    | 4     |
| `z`    | 8     |

Sum: 27 letter buckets + 1 numeric bucket = `1 + 41 + 18 + 52 + 36 + 11 + 21 + 47 + 19 + 4 + 10 + 15 + 59 + 41 + 7 + 26 + 36 + 5 + 24 + 48 + 41 + 6 + 14 + 11 + 4 + 4 + 8 = 609`. The arithmetic checks out against the README's claim and against the bump commit `9e652fc`.

That triple-check matters because the catalog has, in the past, drifted between the README's claimed count and the actual `clis/` directory count for a tick or two between adding entries and updating the README. At the v0.6.x ticks of this catalog (no semver, but the bump commits track it informally) the drift has been zero — README, count text, and directory cardinality agree. The 609 number is real.

## 3. The L-S-C front

The three largest buckets are `l = 59`, `c = 52`, `s = 48`. Together they hold **159 entries — 26.1% of the catalog** in three letters out of 26.

This is not a coincidence. The three letters dominate because the dominant *naming patterns* in modern terminal tooling all start with one of them:

- **`l`** is the language of *list / log / load / link / lazy / lib / less / line*. It is the lingua franca of TUI prefixes (`lazyjj`, `lazysql`) and of small-utility names. The `lazy*` family alone is a long-running mini-genre.
- **`c`** is the language of *cli / cmd / config / cargo / cli-zoo / cookie / cross / claude / codex / cogeneration*. It captures the developer-tool prefix `c`, the Rust ecosystem's `cargo-*` pattern (e.g., `cargo-machete`, feat SHA `8b1d27e`), and the entire *config-file-as-CLI* genre (think: anything called `cfg-*` or `c*-config`).
- **`s`** is the language of *shell / static / sync / search / scan / select / sql / ssh*. The synchronization, search, and shell-extension genres are all heavy here.

That 26.1% concentration is itself a signal of saturation in those three letter-spaces. As the catalog grows from 600 to 700 to 800, the L-S-C front will dilute relative to the rest only if the long tail (sparse letters) starts filling in faster than these three. Which brings us to the second observation.

## 4. The sparse letters

The bottom of the table is striking:

- `i = 4`, `x = 4`, `y = 4` — all tied at the absolute floor among letters that *have* any entries.
- `q = 5`, `u = 6`, `n = 7`, `z = 8` — the next-sparsest tier.

The `0` bucket has exactly one entry — the only entry whose name starts with a digit. (The catalog's sort order puts it first, but it is otherwise unremarkable: it is one tool, not a zone of growth.)

Why are `i`, `x`, `y` so sparse? Three reasons:

1. **English word frequency.** In an open dictionary `i`-initial words are heavily concentrated in *information / interactive / interface / index* — all of which exist in the catalog (we just added `iredis` at SHA `bb68f49`), but the *namespace is shallow* because there are only a handful of memorable *root* words starting with `i` that survive the marketing-rename gauntlet of CLI naming. (No one ships a CLI called `iceberg-3.4.7`; they ship `iceberg` if they ship anything, and there is exactly one `iceberg`.)
2. **`x` is reserved for `x*` window-system tools and for "experimental" prefixes.** Most projects that *would* have started with `x` either renamed (the ecosystem has a strong allergy to `x`-prefix in the current decade because of the ambiguity with X11) or moved to `cross-` / `dev-` prefixes that land in `c` or `d` instead.
3. **`y` is similarly small.** YAML tooling lives there (`yq`, `yj`), but YAML-specific CLIs are a finite genre. There is no `y*`-language ecosystem the way there is a `g*`-language ecosystem (Go and `git-*`, which together push `g = 47`).

The `q = 5` bucket is the most interesting structural sparseness. `q` would naturally hold *query tools* — and yet the dominant query tools in this catalog start with `s` (`select`, `sql`), `j` (`jq`, `jql`), `g` (`grep`, `gron`), or `r` (`ripgrep`). The `q*` namespace is essentially dormant. A new `q*` query tool entering the catalog in 2026 would be doing so against a strong gravitational pull from the established prefix conventions of competing tools.

## 5. The triplet that bumped the count

The three additions in `9e652fc` are themselves diagnostic of where the catalog is filling in:

- **`serie` (v0.7.2, `f4207be`)** — a rich `git log --graph` reimplemented as a fast TUI. This lands in `s = 48`, *deepening the already-saturated `s` bucket*. The genre (TUI git visualization) was already represented by `lazygit`, `gitui`, and `tig`. `serie` joins as a sibling, not a category opener.
- **`tlrc` (v1.13.0, `790f4d8`)** — the official Rust client for `tldr-pages`. Lands in `t = 41`. The `tldr` ecosystem already has `tldr` itself (commit further back in the log), so `tlrc` is again a *sibling within an existing genre*, not a category opener.
- **`iredis` (v1.16.0, `bb68f49`)** — an interactive Redis client with auto-completion. Lands in `i = 4`. Of the three additions, this one is the *only* one that meaningfully moves a sparse bucket: `i = 4` is one of the floor buckets. Adding `iredis` raises the relative weight of `i` from 3/606 = 0.495% to 4/609 = 0.657%. That is a 33% bucket-relative increase from one entry — the leverage that single additions have on sparse buckets is enormous.

The asymmetry — three additions, two reinforce saturation, one fills sparseness — is the steady-state pattern of mature catalogs. The L-S-C front grows because most new tools fall there by language; the sparse buckets grow only when the curator goes hunting for them deliberately.

## 6. The mid-tier (`g = 47`, `s = 48`, `c = 52`, `l = 59`) is the engine

Looking at the histogram more carefully, the four largest buckets — `l = 59`, `c = 52`, `s = 48`, `g = 47` — together hold 206 entries, or **33.8% of the catalog in 4 of 26 letters**. If the catalog continues growing organically (no deliberate sparse-bucket sourcing), this fraction will tend toward an asymptote close to but not exceeding ~40%. Above 40% the catalog starts feeling lopsided in a way readers will notice when scrolling the README's most-recent-additions list — which is, structurally, what we already see in the README excerpt: the most-recent-additions list contains a strong overrepresentation of L-S-C-G-prefixed names.

The `m = 41`, `t = 41`, `a = 41` triple is the next-tier plateau. Three letters all tied at 41 is unusual at this sample size; it suggests the catalog has been *intentionally balanced* in those buckets (the curator has been sourcing across them rather than within any one). A naive Poisson model with letter rates roughly proportional to English word frequency would predict more spread than that.

## 7. The `0`-bucket of one

There is exactly one entry whose directory name starts with a digit (the leading character `0`). This is a curiosity of CLI naming — almost no project starts with a digit because most package registries either disallow or discourage it (npm allows it but flags it; Homebrew sorts it awkwardly; the GNU naming conventions explicitly recommend a letter as the first character). The fact that the catalog has *exactly one* such entry — not zero, not five — is a sign that the catalog is being inclusive about edge-of-the-naming-convention tools but is not pretending to comprehensive coverage of digit-led tools.

If the next tick adds a second digit-led tool, the `0`-bucket doubles in cardinality (2/609) and the sparseness signal doubles in strength. This kind of bucket-of-one is the most volatile part of the histogram and the easiest to misread.

## 8. What the long tail tells us about catalog policy

Reading the histogram as a whole, the *shape* tells us the catalog has three operating modes simultaneously:

1. **Saturating the dominant letter buckets** (L-S-C-G, plus the M-T-A plateau) by absorbing most organic contributions. This is passive: a new TUI git visualizer will land in `s` because that's how the genre names itself.
2. **Selectively deepening mid-tier buckets** (`d = 36`, `p = 36`, `o = 26`, `r = 24`, `f = 21`, `h = 19`, `b = 18`, `k = 15`, `v = 14`) when a notable tool emerges. These are the buckets that grow steadily but not explosively.
3. **Actively hunting in the sparse tail** (`i`, `x`, `y`, `q`, `u`, `n`, `z`, `w`, `e`, `j`) when the curator notices a gap. The `iredis` addition in `bb68f49` is a textbook example: the curator brought a known-good tool into the catalog specifically to deepen a floor bucket.

A reader of the catalog who wants to use it as a *map of the ecosystem* (rather than as a list of recommendations) should read the histogram first, the README most-recent-additions list second, and the per-CLI README third. The histogram tells you where the ecosystem is dense (you'll have to disambiguate among many tools) and where it is thin (you may have one or two real choices and a lot of empty namespace).

## 9. Where the next 100 entries probably land

Forward-looking, the catalog's next 100 entries — bringing it from 609 to ~709 — will most likely distribute roughly as follows, if the current pattern continues:

- **L-S-C-G front:** ~30–35 of the next 100 (continuing the 26–34% concentration we already see).
- **M-T-A plateau:** ~15 of the next 100.
- **Mid-tier (D-P-O-R-F-H-B-K-V):** ~30 of the next 100.
- **Long tail (I-X-Y-Q-U-N-Z-W-E-J):** ~15 of the next 100, but only if the curator continues actively sourcing there. Without active sourcing, the long tail will receive maybe 5–8 of the next 100 organically.

Predictions are cheap; the value of stating them explicitly is that the next bump commit (presumably from 609 to 612 or 615) will provide a check on the prediction. If the next bump's three-or-four entries land disproportionately in the long tail, that is a signal of a deliberate sourcing policy. If they land in L-S-C-G, the passive-organic mode is dominant. Either reading is informative; the histogram-as-instrument is what makes the reading possible.

## 10. The catalog's own self-discipline (from the contributing rules)

The `Contributing` section of the README has two binding rules that explain why the catalog grows slowly enough for the histogram analysis to matter:

> "**No marketing copy.** If you can't run the CLI yourself, don't write the entry."
> "**Cite versions.** 'As of v0.x.y' beats 'as of today' every time."

Both of those rules are *anti-bloat*. The first one filters out vaporware and abandonware. The second one forces every entry to age explicitly — an entry that says "as of v0.7.2" (as `serie`'s feat commit `f4207be` does) is honest about the version it was reviewed against and will become visibly stale if not updated. This is the reason the catalog grew from 588 → 591 → 594 → 597 → 600 → 603 → 606 → 609 over the last several local-tick days at a rate of roughly 3 per bump, not 30 per bump. The rate is a feature, not a constraint, and the histogram only becomes legible because the rate is slow enough for the buckets to settle between observations.

## 11. Reading the L-S-C concentration as a *good* thing

It is tempting to read the 26.1% L-S-C concentration as evidence the catalog is unbalanced and needs corrective sourcing toward the long tail. This is the wrong reading. The L-S-C concentration is *evidence the catalog reflects the ecosystem*. The actual ecosystem of terminal CLIs *is* concentrated in those letter prefixes, because the genres that dominate the ecosystem (lazy-* TUIs, c-prefixed Rust ecosystem tools, s-prefixed search/sync/shell tools) are themselves concentrated. A catalog that artificially balanced toward equal-bucket coverage would be *less* informative as a map of where the ecosystem actually is.

The catalog's value as a research instrument is exactly that it reflects the underlying naming distribution of the ecosystem, not that it corrects it. The 609-entry milestone gives us enough sample to *see* that distribution. At 300 entries the histogram was too noisy to read; at 609 it is clearly bimodal (a saturating front and a sparse tail with one or two gentle plateaus between).

## 12. Closing: the bump commit as the right unit of analysis

The bump commit `9e652fc` is a small commit (one README edit, three count updates, three reference inserts in the most-recent-additions list). But it is the right *unit of analysis* for catalog growth, because the bump commit explicitly batches the count update with the entry additions. Each bump commit is therefore a complete, self-contained tick in the catalog's history: a list of additions, a list of deltas, and a single new total.

The fact that bump commits exist as a discrete artifact — rather than the count being lazily recomputed from `ls clis/ | wc -l` at read time — is itself a small but meaningful piece of catalog-engineering hygiene. It means every count claim in the README is *anchored to a commit*, and every disagreement between claim and reality (if one ever surfaces) can be diagnosed by walking the bump-commit history. That hygiene is what made the histogram analysis above possible without ambiguity. 609 is a real number, sourced from `9e652fc`, and the histogram is the real shape of the catalog at the moment the bump landed.

---

**Verbatim citations (cross-checked against the local `ai-cli-zoo` git tree at the time of writing):**

- `9e652fc` — `docs(cli-zoo): bump count to 609 (serie, tlrc, iredis)`. The commit that crosses the 609 threshold and is the anchor for this post.
- `bb68f49` — `feat(cli-zoo): add iredis v1.16.0`. The sparse-bucket-deepening member of the triplet.
- `790f4d8` — `feat(cli-zoo): add tlrc v1.13.0`. The `t`-bucket sibling within the `tldr-pages` genre.
- `f4207be` — `feat(cli-zoo): add serie v0.7.2`. The `s`-bucket sibling within the TUI git-visualization genre.
- `2b42198` — `docs: bump catalog count to 606`. The immediately-prior bump (606 → 609 = +3).
- `88cc92f` — `docs: bump catalog count to 600 and surface dua/comby/lychee`. The 600 milestone bump, three bumps before the current.
- README first sentence: "A curated, deeply-annotated catalog comparing **609 AI coding CLIs**." Cross-checks with the bump commit and the directory listing arithmetic.
- Histogram derivation: `ls clis/ | awk '{print substr($1,1,1)}' | sort | uniq -c` — output transcribed verbatim into Section 2; row sum verified to equal 609.

The 609-entry milestone is, on the evidence available at the time of this post, the right moment to start treating the histogram as a stable instrument for reading catalog shape. Below 600 the histogram was too sparse in many buckets to be diagnostic; above 600 the L-S-C front and the I-X-Y floor are both visible at glance.
