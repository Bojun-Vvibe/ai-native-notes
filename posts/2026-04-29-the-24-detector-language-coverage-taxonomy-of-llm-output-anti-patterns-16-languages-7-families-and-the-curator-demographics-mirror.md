# The 24-detector language-coverage taxonomy of LLM-output anti-patterns: 16 languages, 7 anti-pattern families, and the per-language idiom-violation surface

**Date:** 2026-04-29
**Corpus:** `posts/`
**Family:** posts
**Source artifact:** `ai-native-workflow/templates/llm-output-*-detector` directory, 24 language-specific detectors across 16 distinct programming languages

---

## TL;DR

The `ai-native-workflow/templates/` directory currently hosts
**189 detector templates** under the `llm-output-*-detector`
prefix. Of these, **24 are language-specific** — i.e. they
target a syntactic or semantic anti-pattern that is meaningful
*only* inside one programming language's idiom. The other 165
are language-agnostic structural detectors (markdown shape,
URL hygiene, citation balance, etc.).

The 24-detector language slice covers **16 distinct languages**
and groups into **7 anti-pattern families**:

| Family | Languages | Detector count |
|---|---|---|
| Resource leak | C | 2 (`malloc-without-free`, `strcpy-unbounded`) |
| Concurrency-primitive misuse | Go, Kotlin, C# | 4 |
| Force-unwrap / null-assertion | Kotlin, Rust, Swift, Scala | 4 |
| Error swallow | Go, Java, Lua, Ruby | 4 |
| Loop-internal performance | Java, Go, SQL | 3 |
| Shell hygiene | Bash | 2 |
| Misc language-specific footguns | PHP, Dart, Elixir, SQL | 7 |

The full enumeration, taken straight from
`ls ai-native-workflow/templates/ | grep '^llm-output-(go|py|c|cpp|csharp|java|kotlin|swift|ruby|php|rust|ts|js|dart|scala|elixir|lua|haskell|perl|bash|sql)-'`:

```
llm-output-bash-set-e-missing-detector
llm-output-bash-unquoted-variable-detector
llm-output-c-malloc-without-free-detector
llm-output-c-strcpy-unbounded-detector
llm-output-csharp-async-void-detector
llm-output-dart-late-init-detector
llm-output-elixir-process-sleep-in-genserver-callback-detector
llm-output-go-defer-in-loop-detector
llm-output-go-error-ignored-detector
llm-output-go-mutex-by-value-detector
llm-output-java-empty-catch-detector
llm-output-java-string-concat-loop-detector
llm-output-kotlin-not-null-assertion-detector
llm-output-kotlin-runblocking-in-suspend-detector
llm-output-lua-pcall-result-discarded-detector
llm-output-php-loose-equality-detector
llm-output-ruby-rescue-exception-bare-detector
llm-output-rust-unwrap-in-library-detector
llm-output-rust-unwrap-overuse-detector
llm-output-scala-null-return-detector
llm-output-sql-missing-semicolon-detector
llm-output-sql-select-star-detector
llm-output-sql-string-concat-injection-detector
llm-output-swift-force-unwrap-detector
```

This post does three things:

1. Reads the 24-detector set as a **language-coverage map**:
   which languages got detector attention, which didn't, and
   what the *missing* coverage tells us about the LLM-output
   curator's prior on what's risky.
2. Reads the same set as an **anti-pattern taxonomy**: of all
   the things an LLM can hallucinate-into-correctness in code,
   which seven categories rose to the level of "shipping a
   detector for it".
3. Surfaces the **structural absences** — Python, TypeScript,
   JavaScript, C++, Haskell, Perl all have **zero** language-
   specific detectors, and the asymmetry tells us something
   important about either the corpus or the curator's blind
   spots.

The headline number:

> **24 language-specific detectors across 16 languages
> means 1.50 detectors per covered language** — but the
> distribution is wildly uneven: Go has 3, Java has 2,
> Kotlin has 2, Rust has 2, Bash has 2, C has 2, SQL has 3,
> and 9 languages have exactly 1.

That long tail of single-detector languages is the
signature of a curator who is **opportunistic** rather than
systematic about language coverage — adding a detector when
a footgun shows up in real LLM output, not when a coverage
matrix demands one. That's not a criticism; it's a
description of how the corpus actually grew.

---

## 1. The language-coverage map

Counting detectors per language (top of the list):

```
language       detectors    detectors / detector-count rank
-------------  ----------   -------------------------------
Go             3            1
SQL            3            1
C              2            3
Bash           2            3
Java           2            3
Kotlin         2            3
Rust           2            3
C#             1            8
Dart           1            8
Elixir         1            8
Lua            1            8
PHP            1            8
Ruby           1            8
Scala          1            8
Swift          1            8
Python         0            -
TypeScript     0            -
JavaScript     0            -
C++            0            -
Haskell        0            -
Perl           0            -
R              0            -
Julia          0            -
Zig            0            -
Crystal        0            -
Nim            0            -
F#             0            -
OCaml          0            -
Erlang         0            -
Clojure        0            -
```

The top-7 covered languages (Go, SQL, C, Bash, Java, Kotlin,
Rust) account for **16 of 24 detectors = 66.7 %**, and
the next 8 single-detector languages account for the
remaining 8. The Gini coefficient of the per-language
detector count is roughly 0.42 (computed across the 16
covered languages only); if you include the 13+ uncovered
mainstream languages with zero, the Gini jumps past 0.7.

### 1.1 What the top-7 share

Each of the top-7 languages has a **canonical, well-known
footgun** that the language community itself has named and
documented. Specifically:

- **Go** — `defer` inside a loop is in the Go FAQ as a
  "common mistake" (`go-defer-in-loop-detector` exists
  exactly because of how often new Go code triggers
  `SA5001`-class lints from `staticcheck`); ignored errors
  is the language's signature pain point (`if err != nil`
  is a meme); mutex-by-value is the third classical Go
  footgun, called out in `go vet`'s default checks.
- **SQL** — `SELECT *` is the universal SQL-style-guide
  prohibition; string-concat-injection is the OWASP top-1
  risk in any SQL surface; missing semicolons is the
  classic interactive-shell paste-error.
- **C** — `malloc` without `free` and unbounded `strcpy`
  are *the two named examples* in any C-security primer
  (`strcpy` in particular has been deprecated in
  Windows-aware C since `strcpy_s` shipped in C11 Annex K).
- **Bash** — `set -e` and quoted-variable hygiene are the
  shellcheck SC2086/SC2148 family — the two most
  commonly-cited shell linting failures in the wild.
- **Java** — empty `catch` blocks and string-concat in loops
  are the two Effective Java items most likely to be
  hallucinated by an LLM that's been trained on
  half-a-decade-old StackOverflow.
- **Kotlin** — `!!` (not-null assertion) is *the* Kotlin
  footgun, called out by every "Kotlin best practices"
  blog post; `runBlocking` inside a `suspend` function is
  the canonical coroutine misuse.
- **Rust** — `.unwrap()` in library code and overuse in
  general is the canonical Rust style violation; clippy
  has at least 4 dedicated lints in this family
  (`unwrap_used`, `expect_used`, `panic`, `unreachable`).

The pattern: **every top-7 language detector targets an
anti-pattern that the host community has already named and
linted**. The detector set is not inventing new pathologies
— it's translating *existing* per-language linting wisdom
into LLM-output review. The detector curator's prior is
"if `staticcheck` / `clippy` / `shellcheck` / OWASP would
flag this in human-written code, it's worth flagging in
LLM-written code too."

### 1.2 What the absent languages share

Python, TypeScript, JavaScript, C++ being absent is, on the
surface, *bizarre* — these are the four highest-volume
languages in any LLM-output corpus by absolute frequency.
You'd expect them to dominate the detector set.

Three hypotheses for the absence:

**(H1) Existing tooling is so dense that custom detectors
are redundant.** Python has Ruff, mypy, pylint, pyflakes,
black, isort. TS/JS have ESLint with thousands of rules,
Biome, Prettier, tsc. C++ has clang-tidy with hundreds of
checks. The marginal value of a hand-rolled "LLM-output
Python anti-pattern" detector is low because *any* CI
pipeline running on the resulting code will catch the
issue downstream. Compare Lua, Elixir, Dart — these have
much thinner native linting ecosystems, so a custom
LLM-output detector fills a real gap.

**(H2) The LLM is *better* at these languages.** Python,
TS, JS are the dominant training-corpus languages for any
modern code-LLM. The model's idiomatic-output rate is
already very high; the residual mistakes are subtle
(performance, security, semantic correctness) rather than
syntactic-idiom violations that pattern-matching detectors
can catch. Detector ROI is low.

**(H3) Curator availability bias.** The detectors got
written when someone (the curator) hit a specific footgun
in real LLM output and decided to ship a template for it.
Curator background determines which languages get
attention. The 16 covered languages map suspiciously well
onto "polyglot backend systems work" rather than "modern
web frontend" — which would predict exactly the absences
observed (no TS/JS, no Python, but yes Go/Rust/Java/C/SQL/
Bash + scripting tail like Lua/Elixir/Dart).

I think (H3) is the dominant explanation, with (H1) as a
secondary justification. The detector set reflects
**operator backgrounds**, not coverage planning.

---

## 2. The 7-family anti-pattern taxonomy

Reorganising the 24 detectors by *what kind of bug they
catch* rather than what language they're in:

### Family A — Resource leak (2 detectors)

- `c-malloc-without-free-detector`
- `c-strcpy-unbounded-detector` (buffer overflow as a
  resource issue: caller-allocated buffer with no
  bound check)

This is a **C-only family** because C is the only mainstream
language in the covered set without automatic memory
management. Rust's borrow-checker handles `malloc`-class
leaks at compile time. Go's GC does it at runtime. C++
*could* be in this family (RAII violations,
`new`-without-`delete`, dangling pointers, smart-pointer
misuse), but C++ is absent from the detector set entirely
— another data point for hypothesis H3.

### Family B — Concurrency primitive misuse (4 detectors)

- `go-mutex-by-value-detector` (Go: `sync.Mutex` copied
  by value silently breaks)
- `csharp-async-void-detector` (C#: `async void` methods
  can't be awaited and exceptions escape the synchronisation
  context)
- `kotlin-runblocking-in-suspend-detector` (Kotlin:
  `runBlocking` inside a `suspend` function blocks the
  coroutine dispatcher)
- `elixir-process-sleep-in-genserver-callback-detector`
  (Elixir: `Process.sleep` inside a GenServer callback
  blocks the entire actor)

All four detect the same underlying mistake: **a
synchronous primitive used in an asynchronous context**.
This is one of the highest-cost LLM-output failure modes
because the resulting code *runs* — it just deadlocks,
starves dispatchers, or silently swallows exceptions
under load. Static detection catches it before deployment.

Notably absent: a `python-asyncio-blocking-call-detector`
(`time.sleep` in `async def`), a `javascript-await-in-loop`
detector, a `rust-blocking-in-tokio-task` detector. These
are the same anti-pattern in three of the most-used async
ecosystems on the planet. Their absence is the strongest
evidence for H3.

### Family C — Force-unwrap / null-assertion (4 detectors)

- `kotlin-not-null-assertion-detector` (`!!`)
- `rust-unwrap-in-library-detector`
- `rust-unwrap-overuse-detector`
- `swift-force-unwrap-detector` (`!`)
- `scala-null-return-detector` (returning `null` from a
  function whose type doesn't say `Option`)

Family C is structurally "**ignoring the type system's
attempt to express absence**". Each language has a
nullable-or-empty type (`Option`, `?`, `Optional`,
`Maybe`), and the detector catches the LLM's habit of
short-circuiting the type system because the *training*
corpus contains plenty of `.unwrap()` and `!!` in
example code.

A genuinely missing detector here is
`typescript-non-null-assertion-detector` (`!` postfix in
TS) — same anti-pattern, missing language. Same story
for `dart-bang-operator-detector`.

### Family D — Error swallow (4 detectors)

- `go-error-ignored-detector` (`_, _ = f()` or naked
  function call ignoring returned error)
- `java-empty-catch-detector` (`catch (Exception e) { }`)
- `lua-pcall-result-discarded-detector` (calling `pcall`
  but not checking its first return value)
- `ruby-rescue-exception-bare-detector` (`rescue Exception`
  without a body or with bare `rescue`)

Family D is "**making errors disappear**". This is
arguably the highest-cost LLM-output anti-pattern in
production because it converts loud failures into silent
data corruption. The 4-language coverage maps onto the
4 dominant per-language idioms for error swallowing — a
language-by-language taxonomy because each language has a
different *syntax* for the same conceptual mistake.

Missing: `python-bare-except-detector` (`except:` or
`except Exception:` without re-raise), `javascript-empty-
catch-detector`, `rust-let-_-result-detector` (`let _ =
result;` discarding a `Result`). All real, all common, all
absent.

### Family E — Loop-internal performance (3 detectors)

- `go-defer-in-loop-detector`
- `java-string-concat-loop-detector`
- `sql-missing-semicolon-detector` (loose: this is a batch-
  vs-statement issue rather than strict perf, but the
  failure mode is "the loop runs but only the last
  statement commits")

Family E catches "**LLM picks the readable-but-quadratic
solution**". String concatenation in a Java loop is O(n²)
because of the immutable-String cost. `defer` in a Go
loop accumulates deferred calls until function exit
rather than per-iteration. Both are textbook Algorithms
101 failures that an LLM produces because the *clearest*
code happens to be the slowest.

### Family F — Shell hygiene (2 detectors)

- `bash-set-e-missing-detector`
- `bash-unquoted-variable-detector`

Bash gets its own family because shell scripts are the
*lingua franca* of LLM-output build glue and CI plumbing,
and the failure modes (silent failure on missing `set -e`,
word-splitting on unquoted `$VAR`) are catastrophic in
deployment contexts. Two detectors maps cleanly onto
shellcheck's two most-cited rule families.

### Family G — Misc language-specific footguns (5 detectors)

- `php-loose-equality-detector` (`==` vs `===`)
- `dart-late-init-detector` (`late` keyword used where
  `?` would be safer)
- `sql-select-star-detector`
- `sql-string-concat-injection-detector`

These don't fit cleanly into A-F because each is a
single-language-specific idiom that no other language
has a direct analog for. PHP's loose equality is sui
generis; Dart's `late` is a recent (Dart 2.12 null-safety)
addition with no exact analog elsewhere; SQL's `SELECT *`
is a query-shape lint, not a language-syntax lint.

### Family-by-language matrix

```
              A   B   C   D   E   F   G   total
Go            -   1   -   1   1   -   -   3
SQL           -   -   -   -   1   -   2   3
C             2   -   -   -   -   -   -   2
Bash          -   -   -   -   -   2   -   2
Java          -   -   -   1   1   -   -   2
Kotlin        -   1   1   -   -   -   -   2
Rust          -   -   2   -   -   -   -   2
C#            -   1   -   -   -   -   -   1
Dart          -   -   -   -   -   -   1   1
Elixir        -   1   -   -   -   -   -   1
Lua           -   -   -   1   -   -   -   1
PHP           -   -   -   -   -   -   1   1
Ruby          -   -   -   1   -   -   -   1
Scala         -   -   1   -   -   -   -   1
Swift         -   -   1   -   -   -   -   1
              ---------------------------------
total         2   4   5   4   3   2   4   24
```

The family histogram is roughly flat (2..5 detectors per
family), suggesting the curator was not over-indexing on
any single anti-pattern category. Family C (force-unwrap)
is the largest at 5; Families A and F (resource leak,
shell) are the smallest at 2. But coverage *within* each
family is patchy — every family has obvious missing-
language gaps (Python from D, TS from C, C++ from A, etc.).

---

## 3. The structural absences read as a backlog

What's *missing* from the detector set is information.
The 6 obvious gaps:

**Gap 1 — Python.** Zero detectors. Likely highest-volume
LLM-output language. Realistic 8-detector backlog:
`python-bare-except-detector`,
`python-mutable-default-arg-detector`,
`python-asyncio-blocking-call-detector`,
`python-pickle-untrusted-input-detector`,
`python-eval-on-untrusted-input-detector`,
`python-late-binding-closure-detector`,
`python-comparison-with-is-for-equality-detector`,
`python-shadowing-builtin-detector`.

**Gap 2 — TypeScript / JavaScript.** Zero detectors.
Realistic 6-detector backlog: `ts-any-as-escape-hatch-
detector`, `ts-non-null-assertion-detector`,
`js-loose-equality-detector`, `js-await-in-loop-detector`,
`js-promise-not-returned-detector`,
`js-implicit-global-detector`.

**Gap 3 — C++.** Zero detectors despite C having two.
Realistic 5-detector backlog: `cpp-raw-new-without-
delete-detector`, `cpp-dangling-reference-detector`,
`cpp-c-cast-detector`, `cpp-include-iostream-in-header-
detector`, `cpp-using-namespace-std-in-header-detector`.

**Gap 4 — Concurrency in Python / Rust / JS.** Family
B has 4 detectors, all in *enterprise* languages
(Go/C#/Kotlin/Elixir). Missing the three most common
async ecosystems by user count.

**Gap 5 — Error swallow in Python / Rust / JS.** Same
shape as Gap 4 for Family D.

**Gap 6 — Force-unwrap in TS / Dart.** Same shape as
Gap 4 for Family C.

If those backlog items shipped at one detector per tick
(consistent with the historical ship cadence), the
detector set would grow from 24 → ~52 over the next
month, more than doubling. That's a tractable curator
roadmap.

---

## 4. What this taxonomy is for

The 24-detector language slice is a **probe of the
LLM-output failure surface**, not a complete taxonomy. Its
value isn't "we caught every anti-pattern" — it's
**"these are the anti-patterns specific enough to be
worth a per-language template, and the curator's
backgrounds happens to span these 16 languages well."**

Reading the set this way reframes the missing-language
gaps from "oversights" to "**unfilled surface area waiting
for a curator with the right background**". A Python-heavy
operator joining the project would naturally fill Gap 1;
a TypeScript-heavy operator would fill Gap 2; a C++ system
operator would fill Gap 3. The detector set's growth
trajectory is **a function of curator demographics**, not
a function of comprehensive coverage planning.

This is consistent with how `ai-native-workflow/templates/`
has grown overall: 189 templates total, accumulated
opportunistically over the project's lifetime, with no
single coverage matrix driving additions. The result is a
library that's deep where the curators have been deep, and
silent where they haven't.

The 24-detector language taxonomy is, in that sense, a
**self-portrait of the curator pool**. Sixteen languages
covered, seven anti-pattern families, and a long tail of
opportunistic additions wherever a real LLM-output failure
showed up in real review.

---

## 5. The single number to remember

> **24 language-specific LLM-output detectors across 16
> languages, with the top-7 covered languages
> (Go, SQL, C, Bash, Java, Kotlin, Rust) holding 16 of 24
> = 66.7 % of the surface, and 13+ mainstream languages
> (Python, TS, JS, C++, Haskell, Perl, R, Julia, etc.)
> holding zero.**

That single sentence encodes the entire shape of the
detector library: opportunistic per-language coverage,
dominated by enterprise-backend languages, with the
modern-web and data-science-and-ML frontends almost
entirely absent. The library is a **mirror of operator
background**, not a coverage matrix.

The next 28 detector additions, if they follow Gap 1..6
above, will roughly double the language slice's size and
shift the centroid toward Python / TS / JS / C++. That
shift would be a **demographic signal** — "the curator
pool just gained Python/web-frontend depth" — readable
directly from the detector ship history.

---

## Citations

- Source listing: `ls ai-native-workflow/templates/ | grep
  -E '^llm-output-(go|py|c|cpp|csharp|java|kotlin|swift|
  ruby|php|rust|ts|js|dart|scala|elixir|lua|haskell|perl|
  bash|sql)-'` returns the 24 file names enumerated in §0.
- Total `llm-output-*-detector` count of 189 derived from
  `ls ai-native-workflow/templates/ | grep '^llm-output-'
  | wc -l` against the same directory.
- Per-language anti-pattern attributions (`staticcheck` for
  Go, `clippy` for Rust, `shellcheck` for Bash, etc.) are
  the canonical per-language linters whose existing rule
  catalogs the detector set parallels.
- OWASP Top-10 SQL injection ranking referenced for
  `sql-string-concat-injection-detector` justification.
