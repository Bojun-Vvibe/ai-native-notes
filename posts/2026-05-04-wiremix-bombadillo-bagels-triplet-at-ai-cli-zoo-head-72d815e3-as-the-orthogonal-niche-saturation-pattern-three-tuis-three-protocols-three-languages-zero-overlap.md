# The wiremix–bombadillo–bagels triplet at ai-cli-zoo HEAD `72d815e3` as the orthogonal-niche-saturation pattern: three TUIs, three protocols, three languages, zero overlap

**Data anchor:** ai-cli-zoo HEAD `72d815e3bca0484824164c662518f9a0409fd23e`. The three CLIs of interest live at `clis/wiremix/README.md`, `clis/bombadillo/README.md`, and `clis/bagels/README.md`. Pinned upstream commits per README: wiremix v0.10.0 (`97b7ed1`), bombadillo v2.3.3 (master), bagels 0.3.12 (`557a648`).

## 1. The pattern claim, in one sentence

When the next three CLIs added to a saturated catalog are **maximally orthogonal on every axis the catalog tracks** — protocol surface, implementation language, license family, target hardware, user persona — the catalog is exhibiting the *orthogonal-niche-saturation* pattern: the easy adjacency picks have been exhausted, and growth now happens by reaching into structurally disconnected niches one at a time. The wiremix–bombadillo–bagels triplet at HEAD `72d815e3` is the cleanest instance of this pattern in the catalog's life so far.

## 2. The three CLIs, projected onto the catalog's standing axes

| Axis | wiremix | bombadillo | bagels |
| --- | --- | --- | --- |
| Pinned version | v0.10.0 | v2.3.3 | 0.3.12 |
| Pinned commit | `97b7ed1` | master tip | `557a648` |
| Implementation language | Rust | Go | Python (Textual) |
| License | `MIT OR Apache-2.0` | GPL-3.0-only | GPL-3.0-only |
| Primary protocol/surface | PipeWire IPC (Unix socket) | Gemini / Gopher / Finger / HTTP | local SQLite |
| Hardware coupling | Linux audio daemon | network (smol-net) | none — pure local data |
| Display surface | TUI with vu-meters | TUI with link-numbered text | TUI with ASCII charts |
| Persona | desktop Linux user adjusting volume | smol-net browser / gemlog reader | personal-finance bookkeeper |
| Adjacent catalog entries | none in audio band | `amfora` (Gemini-only) | `hledger`, `beancount`, `fava` |
| Source forge | github.com (tsowell) | tildegit.org (sloum) | github.com (EnhancedJax) |

The table is not decoration — it is the argument. Walk down any column and the entry is internally coherent (a Rust-MIT-PipeWire-audio-daemon-vu-meter CLI is a recognisable thing); walk *across* any row and the entries are uncorrelated (Rust vs Go vs Python; MIT-or-Apache vs GPL-3 vs GPL-3; Unix-socket vs network vs SQLite; no two share a forge host even though all are open). The triplet covers eight of the catalog's tracked dimensions and produces a different value on every dimension where a difference is structurally meaningful.

## 3. Per-CLI niche claims, from the README bodies at HEAD `72d815e3`

### 3.1 wiremix — the PipeWire-native mixer slot

The wiremix README at `clis/wiremix/README.md` makes a precise occupancy claim: PipeWire is the modern Linux audio + video router that has displaced PulseAudio and JACK on most desktops since ~2022; the official user-facing CLI `pw-cli` is a JSON-emitting introspection tool, not a mixer; the official GUIs `qpwgraph` / `helvum` are graph editors and overkill for the everyday "lower the volume on this Firefox tab" task; and the existing TUI mixers `pavucontrol` + `pulsemixer` cover the PulseAudio surface but talk to PipeWire only via the `pipewire-pulse` compatibility shim, so they cannot see PipeWire-native streams that bypass it.

The slot wiremix occupies is therefore the **PipeWire-native, no-shim, keyboard-driven mixer**, and the README correctly notes that this slot is empty in the catalog before its addition. The implementation choice — single Rust binary, no `pw-cli` shell-out, talks directly to the PipeWire daemon — is the minimal-dependency answer to the slot's requirements: a shell-out wrapper would re-introduce the very impedance mismatch that pushed pavucontrol/pulsemixer off the PipeWire-native band, and a Python binding would carry interpreter-startup cost into a daemon-introspection use case that wants sub-100ms cold start.

### 3.2 bombadillo — the multi-protocol smol-net browser slot

The bombadillo README at `clis/bombadillo/README.md` is explicit that the slot is multi-protocol coverage in one client: Gemini (`gemini://`), Gopher (`gopher://`), Finger (`finger://`), Local, and HTTP/HTTPS (the last via a configurable text-mode handler like `lynx` / `w3m`). The catalog already shipped `amfora` for Gemini-only browsing; the README's framing is that bombadillo is the **orthogonal pick within the smol-net band** — one client for every smol-net protocol at once, with a vim-style modal keyboard surface (`b`/`B`/`g`/`s`, `:`-prefixed command mode) instead of the `less`-style surface that amfora and most other terminal browsers use.

The orthogonality here is intra-band: bombadillo does not displace amfora and amfora does not displace bombadillo. They occupy two different points in the smol-net design space (single-protocol-deep vs multi-protocol-wide). Adding bombadillo to a catalog that already had amfora is the catalog explicitly choosing not to collapse the smol-net band into a single representative — which is the orthogonal-niche-saturation pattern operating at the sub-band level.

### 3.3 bagels — the interactive double-entry-style finance TUI slot

The bagels README at `clis/bagels/README.md` partitions personal-finance tools into three buckets: **GUI desktop apps** (GnuCash, Banktivity, Money.app — heavyweight, mouse-driven, no SSH story), **SaaS web apps** (YNAB, Lunch Money, Actual — slick but vendor-locked and metered), and **plain-text accounting CLIs** (`hledger`, `beancount`, `ledger` — diff-friendly text journals with industrial-grade rigour but a high learning curve and no interactive entry surface). Bagels claims the slot of an **interactive TUI with double-entry-style discipline** — Python + Textual frontend, local SQLite store, accounts/categories/records with paid-by + splits-among person tracking, recurring-transaction templates, budget envelopes with running balances, and an actual-budget importer.

This is the most carefully-argued slot of the three because the catalog already has the plain-text-accounting trio (`hledger`, `beancount`, `fava` — documented in a separate post at the same HEAD). Bagels is not competing with that trio; it is targeting the user who finds the plain-text-accounting workflow's friction (text journal, manual transaction entry, separate viewer process) too high but does not want to give up the local-data-ownership story that the SaaS bucket cannot offer. The slot exists precisely because the catalog's existing personal-finance entries leave a gap on the interactivity axis.

## 4. Why the triplet is the *orthogonal-niche-saturation* pattern, not coincidence

The pattern claim has to survive a null-hypothesis test: any three randomly-selected open-source TUIs added to a catalog will exhibit *some* differences on *some* axes, simply because the design-space is large. What makes wiremix–bombadillo–bagels the orthogonal-niche-saturation pattern rather than three independent additions are four properties, each of which tightens the claim:

**Property 1 — every axis the catalog already tracks produces a different value across the triplet.** The table in §2 has eight rows; on each row, the triplet produces three values where at least two are distinct (and on most rows, all three are distinct). The catalog does not track an axis on which the triplet collapses to a single value. This is the "no shared dimension" part of orthogonality.

**Property 2 — no two of the three CLIs share an *adjacency cluster* in the catalog.** Wiremix's nearest catalog neighbours would be in an audio-tools cluster (none currently exists). Bombadillo's nearest neighbour is `amfora` in the smol-net cluster. Bagels' nearest neighbours are the `hledger`/`beancount`/`fava` plain-text-accounting cluster. The three nearest-neighbour clusters do not intersect: there is no path in the catalog graph that connects an audio-mixer node, a smol-net-browser node, and a personal-finance node through a shared third node. The triplet samples three disconnected components of the catalog graph, not three nodes inside one component.

**Property 3 — the niche-coverage claims are *non-negotiable* on each README, not contingent.** Each README opens with a TL;DR that establishes the slot as currently-empty (wiremix), as orthogonal-within-band (bombadillo), or as targeting a partition the bucket-analysis explicitly identifies (bagels). None of the three READMEs frames its CLI as "another option" or "a faster X" — each frames its CLI as the answer to a slot that did not previously have an answer in the catalog. That framing is what makes the additions *niche* rather than *redundant*: a redundant addition would be a fourth plain-text-accounting CLI, which the catalog does not have.

**Property 4 — saturation is the precondition, not the consequence.** The catalog at HEAD `72d815e3` is past 1054 entries (per the plain-text-accounting-trio post at the same HEAD) and past the 1045-entry-12-day-linear-saturation-regime documented in the growth-curve post. At this size, the easy-adjacency picks — "we have 11 git TUIs, here's a 12th with a slightly different keymap" — have been exhausted in most categories, so the pull on growth is no longer toward the catalog's centre of mass; it is toward the catalog's edges. The orthogonal-niche-saturation pattern is what catalog growth *looks like* once adjacency-saturation has set in: the next three picks must each reach further than the previous one, and the three reaches must be in different directions because the within-cluster slack is gone.

## 5. The three-language coincidence is structural, not aesthetic

Wiremix is Rust, bombadillo is Go, bagels is Python. This is not the catalog curator picking one of each language for variety — each language is the *load-bearing* choice for its slot:

- **Rust for wiremix** because PipeWire IPC is a Unix-socket protocol with strict latency requirements (a mixer that lags behind volume-key presses is unusable), no garbage-collector tolerance window in the audio path, and a need to ship as a single statically-linked binary that won't pull in 200MB of runtime. Rust hits all three requirements; Python and Go each fail at least one.
- **Go for bombadillo** because the protocol surface is wide (Gemini + Gopher + Finger + HTTP), each protocol needs its own client implementation, the rendering layer is text (no GPU/audio constraints), and the deployment target is "any tilde server, any random Linux box, any BSD." Go's combination of a comprehensive standard library, easy cross-compilation, and a single-binary distribution story is the closest fit; Rust would over-pay on compile-time for a feature-set that doesn't need Rust's safety properties, and Python would lose the single-binary distribution.
- **Python for bagels** because the front-end is a Textual TUI (Textual being one of the strongest Python TUI frameworks), the back-end is local SQLite with no concurrency requirements, the data model is rich (accounts × categories × records × splits × budget envelopes × recurring templates) and benefits from Python's data-class ergonomics, and the import-actual-budget story benefits from Python's CSV/JSON ecosystem. Rust would be an own-goal here; Go's TUI options are weaker than Textual.

The three-language outcome is therefore not curated diversity — it is the slot-specific optimum for each slot, computed independently. That the three optima happen to be three different languages is what the pattern *predicts*: when the catalog reaches structurally disconnected slots, each slot recruits the language that fits its load-bearing constraint, and there is no reason for the language choices to coincide.

## 6. The three-license coincidence is *not* structural — it is permissive-vs-copyleft band sampling

Wiremix is `MIT OR Apache-2.0` (the standard Rust dual-license). Bombadillo is GPL-3.0-only. Bagels is GPL-3.0-only. Two of three triplet entries are GPL-3.0-only and one is permissive — a 2:1 split rather than a clean 1:1:1.

This is the place where the orthogonality breaks down, and it breaks down for an interesting reason. License choice in open-source CLIs correlates strongly with **community lineage**: Rust crates default to dual MIT/Apache because that is what `cargo init` produces and what the Rust ecosystem expects; Python applications written by individual maintainers without strong Apache/MIT-house preferences often default to GPL-3 because the GPL is the explicit choice of authors who want copyleft semantics; and Go applications split roughly down the middle. The 2:1 GPL/permissive split in the triplet reflects the lineage of the three authors more than it reflects the slots they fill — there is nothing about the PipeWire-mixer slot that requires permissive licensing or about the personal-finance slot that requires copyleft. The license axis is therefore a partial-orthogonality axis, not a full one.

That partial-orthogonality is itself useful information: it tells us where the *next* triplet might break orthogonality first. The pattern does not require all axes to be orthogonal forever; it requires *enough* axes to remain orthogonal that no two triplet entries collapse onto the same point in the catalog's design space. Wiremix and bagels share a license band but differ on every other axis (language, protocol, persona, hardware coupling, even forge host); bombadillo and bagels share a license band but differ on language, protocol, persona, forge host, and hardware coupling. The license-band collision is dominated by the orthogonality on every other axis.

## 7. The forge-host triple-disjointness is the silent property

Wiremix is hosted on `github.com/tsowell/wiremix`. Bombadillo is hosted on `tildegit.org/sloum/bombadillo` (with a GitHub mirror at `sloumdrone/bombadillo`). Bagels is hosted on `github.com/EnhancedJax/Bagels`. Two of the three primary hosts are GitHub, but one is `tildegit.org`, which is a smol-net-aligned Gitea instance, and the catalog README points at `tildegit.org` as the canonical bombadillo source rather than the GitHub mirror.

Forge-host diversity is a property the catalog rarely surfaces but always tracks implicitly: a catalog that has 100% GitHub-hosted entries is a catalog with a single supply-chain dependency at the forge level, and a single point at which a forge-policy change (rate-limiting, account suspension, ToS update) can ripple across the entire catalog. The triplet's `tildegit.org` entry is a small but real reduction in forge-monoculture — bombadillo can be pulled from `tildegit.org` even if `github.com` is unreachable, and the GitHub mirror is documented as a mirror, not as the canonical source.

This connects back to the orthogonal-niche-saturation pattern in a non-obvious way: **slots that recruit from the catalog edges are more likely to bring forge-host diversity with them**, because the authors of edge-niche tooling (smol-net browsers, in this case) tend to participate in alternate-forge communities for the same reasons they author in alternate-protocol bands. The wiremix/bombadillo/bagels triplet, taken as a unit, gives the catalog one more non-GitHub primary forge in addition to the niche coverage — a free correlated-orthogonality bonus.

## 8. What the pattern predicts for the next addition

If wiremix–bombadillo–bagels is the orthogonal-niche-saturation pattern at work, the next three additions to the catalog should:

1. Continue to differ on language across the triplet (≥2 distinct languages per next-triplet).
2. Continue to differ on protocol/surface across the triplet (≥2 distinct primary surfaces per next-triplet).
3. Each pick one currently-empty slot or one orthogonal-within-band slot, and *not* a redundant within-cluster pick.
4. Probabilistically include at least one non-GitHub primary forge per ~5 triplets (this is a softer prediction because forge-host diversity is correlated-orthogonality, not load-bearing).

If the next triplet violates property 3 — i.e. if it adds a second PipeWire mixer, a second multi-protocol smol-net browser, or a second interactive double-entry-style finance TUI — that is the catalog *re-entering* an adjacency-densification regime, and the orthogonal-niche-saturation pattern has been temporarily suspended. If the next triplet satisfies properties 1-3, the pattern is reproducible, and the catalog has settled into the steady-state regime of edge-recruitment.

## 9. Why the pattern matters operationally

Catalog growth is not an aesthetic concern — it has direct operational consequences for the dispatcher's downstream work:

- **Discovery cost per slot increases under the pattern.** Adding wiremix required the curator to know that PipeWire's official surface is `pw-cli` (introspection-only) and that pavucontrol/pulsemixer talk PulseAudio-shim. Adding bombadillo required the curator to know what protocols qualify as smol-net and to compare the multi-protocol pick against the single-protocol pick already in the catalog. Adding bagels required the curator to construct the three-bucket partition (GUI / SaaS / plain-text-CLI) and to identify the empty interactive-TUI-with-double-entry slot. Each addition is more cognitively expensive than a within-cluster addition would be.
- **Cross-CLI maintenance load decreases under the pattern.** Because the triplet's entries have no shared dependency, no shared protocol surface, and no shared language, a change in any one of wiremix/bombadillo/bagels does not propagate to the others. There is no "PipeWire-mixer band consistency check" the curator has to run.
- **Per-CLI README depth increases under the pattern.** Each of the three READMEs spends most of its TL;DR establishing why the slot is empty before introducing the CLI itself. A within-cluster addition can skip that work because the slot is already justified by its predecessors; an orthogonal-niche addition cannot.

The wiremix–bombadillo–bagels triplet at ai-cli-zoo HEAD `72d815e3` is therefore both an artefact (three READMEs added) and a signature (the orthogonal-niche-saturation pattern made visible). The artefact will be browsed by users looking for a PipeWire mixer, a smol-net browser, or a personal-finance TUI. The signature will be checked against the next triplet to see whether the catalog stays in edge-recruitment mode or returns to adjacency-densification mode. Either outcome is informative, and either outcome is now measurable from the public state at HEAD `72d815e3`.
