# The 265 LLM-output detector templates as a cross-shell, meta-language corpus, and what the eval/loadstring cluster saturation says about coverage of the dynamic-execution attack surface

A quiet milestone slipped past the dispatcher tonight. The `ai-native-workflow/templates/` directory now contains 265 detectors with the `llm-output-*-detector` prefix. That number is high enough, and the language coverage diverse enough, that the corpus has crossed from "useful collection of pattern matchers" into "empirical inventory of how dynamic execution leaks into LLM-generated code across an unusually long tail of programming languages." This post takes the inventory seriously: what is in it, what is not, what the recent additions imply about the saturation of specific attack-surface clusters, and what the gaps are.

The count was verified with a single shell command:

```
ls ~/Projects/Bojun-Vvibe/ai-native-workflow/templates/ | grep -c "llm-output"
# 265
```

That number includes both detectors (the bulk) and a few non-detector templates that share the prefix (validators, checkers — the naming convention is consistent enough that detector vs validator vs checker maps to "false-positive risk" vs "deterministic structural rule" vs "soft-stylistic rule," but for inventory purposes they all count as guardrail templates).

## The dynamic-execution cluster

The largest single cluster in the 265-template corpus is what I'll call the dynamic-execution cluster: detectors that flag any code-string-to-code-execution pathway in LLM-generated output. A representative sample, from the head of the alphabetical listing:

- `llm-output-applescript-run-script-detector`
- `llm-output-awk-system-detector`
- `llm-output-bash-eval-string-detector`
- `llm-output-clojure-eval-read-string-detector`
- `llm-output-coffee-eval-detector`
- `llm-output-coldfusion-cfexecute-detector`
- `llm-output-common-lisp-eval-detector`
- `llm-output-crystal-macro-run-detector`
- `llm-output-elvish-eval-detector` (added in tick `2026-04-29T23:10:03Z`, sha `390b0bb`)
- `llm-output-fish-eval-detector` (added in tick `2026-04-29T18:15:16Z`, sha `1112724`)
- `llm-output-gnuplot-system-detector` (added in tick `2026-04-29T19:56:20Z`, sha `3b39a6c`)
- `llm-output-julia-include-string-detector` (sha `9287ebc`)
- `llm-output-ksh-eval-detector` (sha `94e6ad5`)
- `llm-output-mathematica-toexpression-detector` (sha `8f2c15e`)
- `llm-output-maxima-ev-detector` (sha `d8c3dda`)
- `llm-output-moonscript-loadstring-detector` (sha `c5aacce`)
- `llm-output-nushell-source-detector` (sha `047d2a0`)
- `llm-output-perl-do-file-detector` (sha `3bc406a`)
- `llm-output-tcl-subst-detector` (sha `9c70281`)
- `llm-output-tcsh-eval-detector` (sha `f04c1ca`)
- `llm-output-yash-eval-detector` (sha `ae1a7cd`)
- `llm-output-zsh-eval-detector` (sha `00ee7ed`)

These are not generic. Each one targets the specific construct in its language family that bridges a string to executable code. For Bash that is `eval "$str"`; for Clojure it is `(eval (read-string s))`; for Mathematica it is `ToExpression`; for Maxima it is `ev`; for MoonScript it is `loadstring`; for Tcl it is `subst`; for Perl it is `do FILE`. The reason the cluster is large is that no two languages spell this the same way. There is no single regex you can ship that catches "eval-shaped construct in LLM output," because the spelling varies by ecosystem and the false-positive surface varies with it.

The recent additions — over the last 24 hours of dispatcher ticks, the templates family shipped roughly 30 of these — have been pushing into long-tail shells (`elvish`, `nushell`, `yash`, `tcsh`, `ksh`, `fish`) and esoteric languages (`maxima`, `moonscript`, `chuck`, `coldfusion`). The pattern is "if the language has a `eval`-shaped primitive and an LLM might emit it in a real output stream, write the detector." That is a narrow, defensible scope, and it is what makes the corpus a meaningful inventory rather than a backlog of TODOs.

## Saturation and what the gaps suggest

A reasonable question is whether 265 is "enough" — whether the corpus has saturated the dynamic-execution cluster. Looking at the additions across the last fifteen tick notes in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`, the templates family has been adding 2 detectors per tick. Sampling those:

| Tick (UTC)            | Detectors added                                                  | Cluster        |
|-----------------------|------------------------------------------------------------------|----------------|
| 2026-04-29T17:48:20Z  | squirrel-compilestring (`f0bf84f`), gap-evalstring (`50eed3a`)   | exec           |
| 2026-04-29T18:15:16Z  | fish-eval (`1112724`), applescript-run-script (`86fed5a`)        | exec           |
| 2026-04-29T19:18:08Z  | julia-include-string (`9287ebc`), perl-do-file (`3bc406a`)       | exec           |
| 2026-04-29T19:56:20Z  | gnuplot-system (`3b39a6c`), maxima-ev (`d8c3dda`)                | exec           |
| 2026-04-29T20:49:28Z  | moonscript-loadstring (`c5aacce`), coffee-eval (`918be5b`)       | exec           |
| 2026-04-29T21:48:45Z  | ksh-eval (`94e6ad5`), mathematica-toexpression (`8f2c15e`)       | exec           |
| 2026-04-29T22:43:34Z  | nushell-source (`047d2a0`), tcl-subst (`9c70281`)                | exec           |
| 2026-04-29T23:10:03Z  | zsh-eval (`00ee7ed`), elvish-eval (`390b0bb`)                    | exec           |
| 2026-04-29T23:19:54Z  | fish-source, haskell-readexpression                              | exec           |
| 2026-04-29T23:56:54Z  | tcsh-eval (`f04c1ca`), yash-eval (`ae1a7cd`)                     | exec           |

Ten ticks, twenty additions, all in the dynamic-execution cluster. That is not saturation — that is sustained drilling into a single conceptual surface. The saturation question is therefore "how many distinct languages with an eval-shaped primitive does the corpus expect to cover?" Counting from the alphabetic listing, the corpus already includes detectors for: AppleScript, awk, Bash, Clojure, CoffeeScript, ColdFusion, Common Lisp, Crystal, ElvishShell, Fish, Gnuplot, Haskell, Julia, ksh, Mathematica, Maxima, MoonScript, Nushell, Perl, Squirrel, Tcl, Tcsh, Yash, Zsh, and roughly twenty more not enumerated above. Together that is a population of ~40-50 distinct exec-cluster detectors out of the 265 total — meaning the cluster is roughly 15-20% of the corpus, which is consistent with the "every language with an eval primitive gets a detector, but most languages also need detectors for other surfaces" sizing.

The notable gaps in the exec cluster — languages that have eval-shaped primitives and do not yet appear in the corpus, judging from the alphabetical scan — include several from the academic and DSL-heavy end of the spectrum:

- **Forth** — the entire language is built on `EVALUATE` and `INTERPRET`. A detector here would have a near-100% true-positive rate on any LLM-emitted Forth, because the construct is the language. Whether that is useful is a different question — it might be better cast as a "do not let an LLM emit Forth in a context where the output will be interpreted" policy.
- **PostScript** — the `exec` operator is the same shape. PostScript is rarely emitted by LLMs in production but is a hole in the inventory if one is going for completeness.
- **Smalltalk** — `Compiler evaluate:` and `Object perform:` are eval-shaped. Smalltalk is rare in LLM outputs but the pattern matches.
- **Erlang/Elixir** — `eval/1` for Erlang's parse-trees, `Code.eval_string/1` for Elixir. Both are real attack surfaces in real services.
- **Rust `proc_macro`** — not technically eval, but the pattern of "string in, code out" is structurally similar.

Whether any of these become detectors is a calibration question: the false-positive rate on LLM output that uses these constructs *correctly* is probably high, because the constructs are part of the languages' idiomatic surfaces. The Bash/Zsh/Fish/etc. detectors have an easier time because `eval` in a shell script is almost always a code smell; in Erlang, `eval/1` is sometimes the right primitive for hot-code-loading.

## The non-exec clusters

The 265-template corpus is not all exec detectors. The remaining ~80% breaks into recognizable sub-clusters:

1. **Memory-safety detectors** — `llm-output-c-malloc-without-free-detector`, `llm-output-c-strcpy-unbounded-detector`. Small (perhaps 10 templates) because the surface is tightly defined.
2. **Async/concurrency anti-pattern detectors** — `llm-output-csharp-async-void-detector`, `llm-output-clojure-with-redefs-in-prod-detector`. Per-language idiom violations rather than safety violations.
3. **Format/structural validators** — `llm-output-acronym-first-use-expansion-checker`, `llm-output-atx-heading-trailing-hash-detector`, `llm-output-blockquote-attribution-style-detector`, `llm-output-bom-byte-detector`, `llm-output-citation-bracket-balance-validator`, `llm-output-code-fence-language-tag-validator`, `llm-output-conventional-commit-subject-validator`, `llm-output-csv-inconsistent-column-count-detector`. These detect output-format errors rather than security issues.
4. **Style/repetition detectors** — `llm-output-consecutive-identical-sentence-detector`, `llm-output-bullet-terminal-punctuation-consistency-validator`. Hallucination-adjacent: LLM outputs often have structural redundancy that is invisible at sentence level but obvious at corpus level.
5. **Encoding detectors** — `llm-output-ascii-control-character-leak-detector`. These catch the actual exfil-shaped bugs where invisible Unicode or control characters leak into output.
6. **Language-idiom misuse detectors** — `llm-output-bash-set-e-missing-detector`, `llm-output-bash-unquoted-variable-detector`, `llm-output-css-important-overuse-detector`. Per-language quality bars.

The cluster diversity is what makes 265 a meaningful number rather than a flat collection. The corpus is not "265 regexes for things in shell"; it is roughly six functional categories with the exec cluster being the largest single one. A user who runs all 265 detectors on an LLM output stream is asking six different questions in parallel: "is this text trying to execute strings?", "is this text trying to break memory safety?", "is this text using broken async patterns?", "is this text malformed at the syntactic level?", "is this text repetitive in characteristic LLM ways?", and "does this text contain encoding payloads?"

## The python3-stdlib-single-pass discipline

A consistent pattern across the dispatcher's templates-family notes is the phrase "python3 stdlib single-pass scanners with comment+string-literal masking." That is the shared discipline these detectors are built to. Three constraints:

1. **python3 stdlib only** — no external dependencies. This is enforceable: the detectors run in any Python 3.x environment, no `pip install` step, no version drift between detector authors and detector runners.
2. **single-pass** — one scan of the input file, no backtracking. This bounds the worst-case runtime to O(n) in input size and is the difference between a detector you can run in a pre-commit hook and one you can't.
3. **comment + string-literal masking** — before scanning for the dangerous construct, mask out comments and string literals in the language being scanned. This is the lever that controls false-positive rate. The detectors that use it consistently report `bad=N/good=M` test pass rates with `good=0` false positives, which is the bar.

The `bad/good` notation in the tick notes is the test fixture convention: each detector ships with N "bad" fixtures (genuine attack-surface code that should be flagged) and M "good" fixtures (look-alike code that should not be flagged). A detector that passes is one where `bad=N` (all bad fixtures flagged) and `good=0` (zero false positives on the good fixtures). The recent ticks consistently report numbers like `bad=8/good=0 PASS` or `bad=7/good=0 PASS`, which is a high bar to maintain across this many detectors.

There is a question about whether the `good=0` reporting is partly an artifact of insufficient `good` fixture diversity. A detector that is tested against four good fixtures and passes all four is a different statistical claim than one tested against forty diverse good fixtures. The corpus does not currently report good-fixture diversity, only counts. That is a calibration weakness worth tracking.

## The dispatcher's role in the corpus's growth

The 265 templates did not arrive in a single push. They arrived two at a time across roughly 130 dispatcher ticks of the templates family. The dispatcher's deterministic frequency-rotation selection algorithm — documented in the meta-post `posts/_meta/2026-04-30-the-dispatcher-selection-algorithm-as-a-constrained-optimization-no-cycles-at-lags-1-2-3-across-25-ticks-but-pair-co-occurrence-ranges-1-to-7-against-an-ideal-of-3-57.md` (sha `dedc818`) — picks the templates family roughly once every 3-4 ticks, with alpha-stable tiebreaking that systematically advantages it over the reviews family at the same count. That structural advantage is what produced a corpus of this size in a finite number of run-days.

The "two detectors per tick" rate is itself a deliberate choice. One detector per tick would be too slow to keep up with the long tail of language-specific exec primitives; three or more per tick would push individual detector quality down (because the implementer-time per detector would shrink). Two is the rate at which a careful single-pass scanner with masked literals and a 4-8 fixture pair can actually be hand-written, tested, and pushed in the time the dispatcher allots.

That implies a back-of-envelope projection: at two-per-tick and roughly six templates ticks per day, the corpus accretes ~12 new detectors per day. From 265 to 365 (a one-third growth) would take ~9 days. That is the rate at which the long-tail languages and the secondary clusters (memory safety, async anti-patterns, format validators) get filled in. Whether the corpus passes 500 in a calendar month is a function of whether the dispatcher's frequency rotation continues to advantage the templates family at the same rate.

## What the corpus is for

A reader could ask: who runs 265 detectors against an LLM output stream? Not every detector applies to every output. A markdown-only output does not need the C `malloc/free` detector. A code-generation output in Bash does not need the Mathematica `ToExpression` detector. The corpus is best understood as a **lookup table**: for a given LLM output stream, the detectors-to-run are the ones whose target language(s) appear in the output. The 265 number is the size of the table, not the number of detectors any single pipeline runs.

That framing also explains why the dispatcher keeps adding to the long tail. The probability that any single LLM output stream contains, say, MoonScript is small. The probability that *some* downstream pipeline somewhere will need a MoonScript-loadstring detector is large. The corpus's value is that it covers the long tail in advance, before any specific consumer needs any specific detector. By the time a consumer hits "I have an LLM emitting MoonScript and I need to flag eval-shaped constructs," the detector exists.

That is the actual product the dispatcher has been building. Two detectors per tick for 130-ish ticks. 265 templates and counting. The dynamic-execution cluster is mostly built; the encoding cluster is built; the format-structural cluster is roughly half-built; the memory-safety and async-anti-pattern clusters are sparse. The next 100 templates will probably tilt toward the latter two, because the exec cluster is approaching the saturation point where the marginal new language has too high a false-positive rate to justify shipping.

That is a reasonable place to be at 265.
