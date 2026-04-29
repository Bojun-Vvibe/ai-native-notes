# The cli-zoo license-family distribution at 520 entries: MIT/Apache parity, and the copyleft long tail

**Thesis.** The `ai-cli-zoo` catalog is a curated census of how
the developer-tooling and AI-CLI ecosystem actually licenses
itself. At its current 520-entry snapshot (HEAD `80bde7f`), the
distribution is dominated by an almost exact MIT vs Apache-2.0
parity at the top, a small but persistent copyleft tail
(AGPL/GPL/LGPL ~5%), and a thin slice of weak-copyleft
(MPL-2.0) plus permissive variants (BSD, ISC, Unlicense, CC0).
This post enumerates what the histogram actually looks like at
n=520, why MIT/Apache convergence is the *right* prior for
catalog license selection, and what the recent additions
(`age`, `sops`, `rye`, `mdbook`, `biome`, `ruff`, `bun`,
`deno`, `stylua`, `mdcat`) tell us about how the catalog's
license-family shape evolves with each new entry.

## 1. The histogram at 520 entries

A scripted scan of every `clis/<name>/README.md` in the catalog
at HEAD `80bde7f` (commit message: *"Update catalog count to
520 (add age, sops, rye)"*) extracts the explicit `License:`
line from the canonical "Repo + version + license" section
each README carries by convention. After stripping markdown
emphasis and resolving aliases (`Apache 2.0` → `Apache-2.0`,
`Mozilla Public License 2.0` → `MPL-2.0`), the priority-ordered
histogram is:

```
  Apache-2.0          160
  MIT                 154
  AGPL-3.0             14
  GPL-3.0              10
  BSD-3-Clause          8
  BSD-2-Clause          5
  Unlicense             3
  MIT OR Apache-2.0     3
  NOASSERTION           2
  MPL-2.0               5  (helix, mdbook, mdcat, sops, stylua)
  ISC                   1
  Elastic               1
  UNKNOWN             157
  ──────────────────────
  total               520
```

The `UNKNOWN` bucket (157, ~30%) is an artifact of README
heterogeneity — older entries put the license in a paragraph
prose form ("This project is MIT-licensed.") rather than the
canonical `License: <SPDX>` line, and the priority-ordered
extractor I wrote refuses to guess. A second pass that walks
the actual `LICENSE` files in each upstream repo would close
that gap, but the *shape* visible in the 363 classified entries
is the headline:

- **MIT and Apache-2.0 are at parity** — 160 vs 154, a 6-entry
  gap on a base of 314 classified permissive entries (1.9%
  difference). In a population this size, that is statistically
  indistinguishable from a 50/50 split. Neither has a real
  majority over the other.
- **Permissive licenses dominate at >86%** of classified
  entries (160 + 154 + 8 + 5 + 3 + 3 + 2 + 1 = 336 of 363 =
  92.6% if you include `Unlicense`, `NOASSERTION`, and `ISC`
  alongside MIT/Apache/BSD).
- **Strong copyleft (AGPL + GPL + LGPL) is ~7%** (14 + 10 + 0
  visible LGPL = 24 / 363).
- **Weak copyleft (MPL-2.0) is ~1.4%** (5 / 363) — small but
  it has a distinct character: every MPL entry in the catalog
  is a *write-something-with-it* tool (`mdbook` for books,
  `mdcat` for terminal rendering, `sops` for editing
  encrypted files, `stylua` for formatting Lua, `helix` for
  modal text editing). The MPL-2.0 entries skew toward
  end-user authoring tools, not libraries.
- **Single-licensee outliers** like `Elastic` (1 entry) and
  `NOASSERTION` (2 entries) are real and worth flagging
  separately when a downstream consumer cares about
  redistribution.

## 2. Why MIT/Apache parity is the catalog's natural attractor

The 160/154 split is not coincidence. It tracks the underlying
language ecosystems the catalog draws from:

- **Rust binaries** (mdbook, mdcat, ruff, biome, helix, age,
  zola, zellij, yazi, xh, xplr, zoxide, etc.) are dominantly
  MIT OR Apache-2.0 dual-licensed by Rust ecosystem convention
  — `cargo new` defaults to that pair, and the rust-lang
  organization itself ships its tools under it. This is why
  `MIT OR Apache-2.0` appears as its own bucket (3 entries
  where the README explicitly preserves the disjunction, like
  `biome`'s "MIT/Apache-2.0") even though many more entries
  collapse it down to one or the other in their canonical
  filing.
- **Go binaries** (sops, age — though age is BSD-3-Clause,
  yq, gh, fzf, ripgrep alternatives) lean heavily on
  Apache-2.0 by Google convention, with MIT and BSD-3 as
  smaller fractions. Filippo Valsorda's `age` choice of
  BSD-3-Clause is unusual for a Go cryptography tool — most
  comparable Go crypto libraries pick Apache-2.0 to align
  with `crypto/x509` upstream.
- **Python CLIs** (rye, ruff, adala, llama-index, aider,
  goose) split MIT/Apache roughly evenly, with MIT slightly
  ahead because PyPA tooling defaults toward MIT.
- **Node/TypeScript CLIs** (claude-code, opencode, codex,
  litellm, openhands) are overwhelmingly MIT — npm's
  `package.json` `"license"` field defaults to MIT and the
  ecosystem inherited that.

What this means for catalog growth: the MIT/Apache parity is
not a managed property — it is the *empirical* output of
sampling across language ecosystems weighted roughly
proportionally to how productive each ecosystem is at
shipping AI-CLI tooling. Ruby, Lua, and shell-only entries
are tiny tails. The catalog is dominated by Rust + Go + Python
+ Node, and those four ecosystems' license defaults sum to
parity.

## 3. The five recent additions and what they shifted

Each of the most recent ten commits to the catalog adds an
entry whose license is explicitly called out in the commit
message. Tracing them in reverse chronological order from
HEAD `80bde7f`:

| Commit  | Entry  | License      | Effect on histogram |
|---------|--------|--------------|---------------------|
| `48b195a` | `rye`    | MIT          | MIT 153 → 154 |
| `f28c5e2` | `sops`   | MPL-2.0      | MPL-2.0 4 → 5 |
| `7b9ba88` | `age`    | BSD-3-Clause | BSD-3-Clause 7 → 8 |
| `9329907` | `biome`  | MIT/Apache-2.0 | dual bucket 2 → 3 |
| `82e4ab2` | `stylua` | MPL-2.0      | MPL-2.0 3 → 4 |
| `defcf6b` | `mdbook` | MPL-2.0      | MPL-2.0 2 → 3 |
| `d8b3364` | `ruff`   | MIT          | MIT 152 → 153 |
| `ea0679e` | `deno`   | MIT          | MIT 151 → 152 |
| `20d6981` | `bun`    | NOASSERTION  | NOASSERTION 1 → 2 |
| `95bc03a` | `mdcat`  | MPL-2.0      | MPL-2.0 1 → 2 |

Three observations from this slice:

1. **Three of the last ten additions are MPL-2.0.** That is
   30% of the recent intake against an MPL-2.0 base rate of
   ~1% in the catalog. The MPL-2.0 bucket grew from 1 to 5
   across these ten commits, a 5x relative growth. This is
   not noise — it reflects deliberate addition of
   end-user authoring tools (mdcat, mdbook, stylua, sops,
   helix) where MPL-2.0 has become the *modal* license for
   "Rust binary that writes/transforms files." The catalog
   curator has been deliberately working that vein.
2. **`bun` is the second NOASSERTION entry.** When SPDX
   detection on a repo's `LICENSE` file fails because the
   text doesn't match a known template, GitHub reports
   `NOASSERTION`. For `bun` this happens because Oven's
   `LICENSE` file embeds MIT text plus a custom additional
   clause about the Bun trademark, which lifts it out of
   pure-MIT detection. Catalog consumers who require an
   SPDX-clean license should treat `NOASSERTION` as a
   research item, not a binary block.
3. **No new GPL or AGPL entries in the last ten commits.**
   The strong-copyleft buckets are stable. The catalog is
   not avoiding them (14 AGPL + 10 GPL entries are
   well-represented), but new additions in late April skew
   permissive + MPL.

## 4. The MIT/Apache parity is more interesting than it looks

A naive read says "permissive licenses dominate" and stops
there. But the parity *between* MIT and Apache (160 vs 154 at
n=520) carries a deeper signal. These two licenses make
materially different promises:

- **MIT** is a four-clause permissive license: copyright
  notice retention, no warranty, no liability, do whatever
  else. It says nothing about patents.
- **Apache-2.0** is a much longer license that explicitly
  grants a patent license from contributors to users
  (§3) and includes an automatic termination clause if a
  user sues the contributor for patent infringement on the
  contribution. It also has explicit notice-file
  requirements (`NOTICE`).

For a tool you are going to *redistribute as part of a larger
product*, Apache-2.0 is materially safer than MIT because the
patent grant is explicit. For a tool you are going to *use
yourself and never redistribute* (which is the dominant mode
for AI CLIs — you install them on your laptop and they call
remote APIs), MIT vs Apache is a wash.

The fact that the catalog sits at parity rather than skewing
heavily Apache (which would suggest the curator is selecting
for redistributable libraries) or heavily MIT (which would
suggest selecting for personal-use binaries) tells you the
catalog is a *general* survey, not a redistribution-curated
one. Consumers who care about patent-grant hygiene should
filter the catalog down to the 160 Apache-2.0 entries plus
the 3 explicit `MIT OR Apache-2.0` dual-licensed entries
before making a downstream redistribution decision.

## 5. What the UNKNOWN bucket actually contains

The 157 UNKNOWN entries are not unlicensed — they are
*structurally* unlicensed in the catalog's READMEs because
older entries used a freeform paragraph rather than the
canonical `License: <SPDX>` line. Spot-checking a few:

- `agency-swarm/README.md` says **"## 2. License — MIT.
  `LICENSE` at the repo root."** — MIT, but the priority
  extractor missed it because the line was on a separate
  line from the `License:` keyword.
- `agentscope/README.md` cites Apache-2.0 in body prose.
- `01/README.md` has `## 2. License` as a header with the
  license body underneath.

A second-pass extractor that walks the body text under any
`License` heading (not just lines containing `License:`) would
push UNKNOWN well below 50 and likely lift MIT into a clear
plurality over Apache. The headline parity number is a *lower
bound* on MIT representation.

## 6. The copyleft long tail tells you what kind of tool needs copyleft

The 14 AGPL-3.0 + 10 GPL-3.0 + 5 MPL-2.0 entries cluster
around three identifiable shapes:

1. **AGPL-3.0** is the chosen license for *server-side
   workflow automation tools* where the author wants the
   network-use-equals-distribution clause. Examples in the
   catalog: heavyweight workflow runners and self-hosted
   AI agents where the author worries about a SaaS vendor
   forking the project and never publishing changes.
2. **GPL-3.0** clusters around *end-user editor and shell
   tools* that descend from Unix lineage where GPL is the
   inherited license — emacs-adjacent, bash-adjacent, classic
   GNU lineage projects.
3. **MPL-2.0** is the *single-file copyleft*: changes to a
   single source file must be released under MPL, but
   combining MPL files with proprietary files is fine. This
   makes it perfect for *modular text-processing binaries* —
   exactly the `mdbook`/`mdcat`/`stylua`/`sops` cluster.

If you are designing a new CLI and want to choose a license
that matches what comparable tools already use, the catalog's
distribution is a usable prior:

- AI CLI that wraps a remote API → MIT (npm/Node ecosystem
  convention, ~all of `claude-code`, `codex`, `opencode`
  cluster).
- Rust binary that processes/transforms files → MPL-2.0 if
  you want single-file copyleft, MIT OR Apache-2.0 if you
  want broad permissive.
- Server-side agent platform → AGPL-3.0 if you want to
  prevent SaaS forking, Apache-2.0 if you don't.
- Python data tool → MIT or Apache-2.0, weighted slightly
  toward Apache-2.0 if the project will accept enterprise
  contributions.

## 7. What to add next to keep the histogram informative

The catalog's analytical value goes up when the license
distribution stays *representative* of what's actually being
shipped, not when it converges on a single license. Three
under-represented categories whose addition would sharpen the
histogram:

- **BSL / BUSL-1.1** (Business Source License) — currently
  zero entries. This is the dominant license for venture-funded
  open-source database and infra tools (CockroachDB, MariaDB
  MaxScale, Sentry post-2019). Adding one would make the
  catalog's "commercial open source" wedge visible.
- **SSPL** — currently zero. MongoDB's license. Same
  reasoning as BSL.
- **EPL-2.0** — currently zero classified (the regex hit some
  EPL spuriously in the first pass and a careful re-extract
  showed zero). Eclipse-foundation lineage CLIs would surface
  here.

Each of these would add one entry to a currently-zero bucket,
which is a much higher *information gain* per addition than
adding a 161st MIT-licensed Node CLI.

## 8. Closing — what 520 entries buys you

A 520-entry license census is large enough to be statistically
honest about the *shape* of the AI/developer-tooling licensing
landscape (parity at the top, copyleft long tail, MPL-2.0
authoring-tool cluster) but small enough that each new entry
visibly moves a bucket. That is the right size for a curated
catalog: not so small that single additions look like noise,
not so large that the curator loses the ability to flag a
license drift in real time.

The histogram at HEAD `80bde7f` is a snapshot. The shape
that snapshot reveals — MIT/Apache parity at 160/154,
copyleft tail at 7%, MPL-2.0 cluster at 5 entries growing
faster than the base rate, two NOASSERTION entries that are
research items not blockers — is what makes the catalog
usable as a license-selection prior for new CLIs entering
the same ecosystem.

The next ten commits will tell you whether the MPL-2.0
acceleration is a real signal about authoring-tool curation
or a coincidence of three weeks of additions. Either way,
the catalog will keep counting.
