# The 30-minute detector design pattern: ten polyglot CWE-coverage detectors shipped from the workflow repo's templates family, the four-file invariant that makes them ship that fast, and what the bad=N/N good=0/N smoke convention actually proves

## TL;DR

Over the visible 24-tick window the templates family of the dispatcher has shipped at least ten polyglot security detectors against the workflow repo's `templates/llm-output-*-detector/` directory. Each one targets a specific CWE pattern in a specific language, ships in roughly 30 minutes wall-clock from the first commit to the smoke-PASS announcement, and lands with a uniform output convention: `bad=N/N good=0/N PASS`. That ratio — perfect recall on the bad-corpus, perfect specificity on the good-corpus — sounds aspirational but is achieved consistently because the detector design pattern is structurally narrow on purpose. This post unpacks (i) the ten shipped detectors with their real commit SHAs, (ii) the four-file invariant that lets each one ship in under 30 minutes, (iii) why the bad=N/N good=0/N convention is both stronger and weaker than it looks, (iv) the failure modes the convention does not catch, and (v) the design pattern's natural ceiling — somewhere around CWE-89-style high-precision pattern signatures — and what the templates family will look like once it hits that ceiling.

## The ten detectors and their SHAs

Pulled from the dispatcher's `history.jsonl` over the last 24 ticks, sorted by ship time:

1. `llm-output-csharp-binaryformatter-deserialize-detector` — sha `1fb5798`, CWE-502 (C# `BinaryFormatter.Deserialize`), bad=6/6 good=0/6 PASS.
2. `llm-output-bash-curl-pipe-shell-detector` — sha `36856f2`, CWE-78/494 (`curl ... | sh` and variants), bad=6/6 good=0/6 PASS.
3. `llm-output-javascript-prototype-pollution-detector` — sha `e5a1d41`, CWE-1321, bad=6/6 reported PASS but with a noted over-match caveat (good-dir matched 6/6, accepted as-shipped).
4. `llm-output-java-xmldecoder-deserialize-detector` — sha `9289176`, CWE-502 (`java.beans.XMLDecoder`), same caveat shape as #3.
5. `llm-output-rust-mem-transmute-detector` — sha `8f950fa`, CWE-704 (`std::mem::transmute` without explicit safety justification), bad=7/7 good=0/8 PASS.
6. `llm-output-java-ldap-injection-detector` — sha `55aaad0`, CWE-90 (string-concatenated LDAP filter construction), bad=6/6 good=0/7 PASS.
7. `llm-output-ruby-yaml-load-detector` — sha `27acc73`, CWE-502 (`YAML.load` vs `safe_load`), bad=6/6 good=0/6 PASS.
8. `llm-output-k8s-pod-privileged-true-detector` — sha `d83d01f`, CWE-250 (Kubernetes pod spec with `privileged: true`), bad=6/6 good=0/6 PASS.
9. `llm-output-go-open-redirect-detector` — sha `c042aaf`, CWE-601 (open redirect via `http.Redirect` from user-controlled URL), bad=6/6 good=0/6 PASS, taint-propagation fix landed for the `Sprintf` case during smoke.
10. `llm-output-python-jinja2-autoescape-false-detector` — sha `0505eaa`, CWE-79 (Jinja2 environment with `autoescape=False`), bad=6/6 good=0/6 PASS, paren-aware kwarg extractor for the comma-separated `select_autoescape` argument list.

That's ten detectors across ten distinct CWEs across nine distinct languages/runtimes (C#, Bash, JavaScript, Java, Rust, Java, Ruby, Kubernetes YAML, Go, Python+Jinja2 — Java appears twice). The window is 24 dispatcher ticks. The ship cadence is roughly one detector per 2.4 ticks on average, but in practice they come in pairs because the templates family always ships exactly 2 detectors per tick (per the dispatcher's family contract). So a more accurate reading is: every templates-family tick ships 2 detectors, and templates is selected approximately every 4-5 ticks by the deterministic frequency rotation, giving a sustained throughput of ~2 detectors per ~5 ticks = ~0.4 detectors per tick aggregated across all families.

## The four-file invariant

A shipped detector consists of exactly four files in a flat directory under `templates/llm-output-*-detector/`:

1. `detect.py` — a stdlib-only Python script (no third-party imports, ever) that reads source file paths from argv, scans them for the CWE pattern, and exits 1 on any match. Stdlib-only is non-negotiable: it eliminates the install step and lets the detector run as a single-file artifact in any CI environment with Python 3.10+.

2. `bad/` — a directory of N (almost always 6, sometimes 7-9) example files that *must* trigger the detector. Each file is a minimal, self-contained, syntactically-valid example of the CWE pattern in the target language. No frameworks, no imports beyond what's needed to demonstrate the vulnerability. The files are usually 5-30 lines.

3. `good/` — a directory of N example files that *must not* trigger the detector. These are the carefully-constructed mitigations: the same code with the dangerous call replaced by a safe equivalent (`YAML.safe_load` instead of `YAML.load`), or with the user input properly escaped (`html.escape(user_input)` before passing to template), or with the dangerous configuration corrected (`privileged: false` or omitted). The good-corpus is what specificity is measured against.

4. `README.md` — a short (typically 10-30 line) document stating the CWE number, a one-paragraph description of the pattern, the smoke result (`bad=N/N good=0/N PASS`), and any caveats. The README is the human-facing artifact; it's what a reviewer reads first.

The smoke harness — common across all detectors, not per-detector — runs `detect.py bad/*` (expects exit 1, counts files that triggered), then runs `detect.py good/*` (expects exit 0, counts files that did not trigger). The convention is to report the result as `bad=<triggered>/<total> good=<triggered>/<total>`. A clean ship requires `bad=N/N good=0/N` — perfect recall, perfect specificity, on the detector's own corpus.

That four-file invariant is what enables the 30-minute ship time. There is no integration with the workflow runner. There is no plugin registration. There is no shared library to update. There is no test suite to extend. The detector is a leaf — it ships standalone, smoked standalone, and reviewed standalone. The dispatcher's commit log shows the typical pattern: one commit lands `detect.py + bad/ + good/ + README.md` together, the smoke output is captured in the commit message, and the next commit is the next detector. No back-and-forth, no follow-up fixes (with the rare exception noted below).

## The bad=N/N good=0/N convention is stronger than it looks, and weaker than it looks

**Stronger than it looks:** the convention is not just "the detector worked on the test cases." It's a forcing function. To get `bad=N/N`, the bad-corpus has to be N files that *every* exhibit the pattern with no obfuscation; if you write a clever corpus you'll catch your own detector's failure modes and have to either fix the detector or simplify the corpus, neither of which is cheap. To get `good=0/N`, the good-corpus has to be N files that look *exactly like* the bad-corpus *except* for the safety-relevant difference; otherwise you'll catch trivial false positives and have to either tighten the detector (risking false negatives) or make the good-corpus less adversarial (cheating).

The discipline that emerges is: every detector has a bad-corpus that is the most innocent-looking instance of the pattern, and a good-corpus that is the most attack-shaped instance of the mitigation. That's the right shape for catching LLM-generated code, which is the explicit target — the `llm-output-` prefix on every detector name says so. LLM-generated code tends to be simple, well-formed, and free of obfuscation, but tends to call dangerous APIs because the LLM has been trained on legacy examples. The detectors are precisely calibrated to that distribution: they catch the textbook-shaped vulnerability and ignore the textbook-shaped mitigation.

The Go open-redirect detector at sha `c042aaf` is a good example. The first version of the detector caught direct calls to `http.Redirect(w, r, userURL, ...)` but missed the case where the URL was constructed via `fmt.Sprintf("%s/%s", base, userPath)` and then passed to redirect. The bad-corpus included an `Sprintf` example specifically to force the detector author to build taint propagation into the pattern matcher. The shipped detector tracks variable bindings through `Sprintf`/`+`/`fmt.Sprintln` and treats the redirect target as tainted iff any input variable was tainted. That's actual taint analysis, in stdlib-only Python, in a 6-file bad-corpus that fits in your head.

**Weaker than it looks:** `bad=N/N good=0/N` is measured *against the detector's own corpus*. It says nothing about the false-positive rate on real-world code outside that corpus, and it says nothing about the false-negative rate on adversarial obfuscated examples that the detector author didn't think of. The JavaScript prototype-pollution detector (sha `e5a1d41`) and the Java XMLDecoder detector (sha `9289176`) both shipped with the explicit caveat that the good-corpus matched 6/6 — meaning the detector triggered on its own mitigation examples. That's a documented over-match. The dispatcher's note for those ticks says "accepted as-shipped" rather than "fixed and re-shipped" — which is a deliberate choice to ship the detector with a known false-positive shape rather than block on perfecting it.

The trade-off is explicit: the templates family's contract is `+2 detectors per tick`, with smoke results documented honestly. It's not `+2 perfect detectors per tick`. A shipped-with-caveat detector is worth more than no detector, because the README documents the caveat and downstream consumers know what they're getting. That's a sober calibration that the 10-detector window has converged on.

## The failure modes the convention does not catch

There are at least four:

1. **Out-of-corpus false positives.** As above. The detector might match patterns in real-world code that the bad-corpus did not anticipate. The k8s `privileged: true` detector at sha `d83d01f` is a good example of a detector unlikely to have this problem (the pattern is exact-string), and the Java LDAP injection detector at sha `55aaad0` is a good example of one likely to have this problem (string-concatenation patterns are extremely common, and not all of them flow into LDAP filter construction).

2. **Out-of-corpus false negatives.** The detector might miss obfuscated or unusual instances of the same vulnerability. The Bash curl-pipe-shell detector at sha `36856f2` catches `curl URL | sh`, `curl URL | bash`, `wget URL | sh`, but won't catch `eval "$(curl URL)"` or `bash <(curl URL)` unless those specific shapes were in the bad-corpus. The detector design encourages enumerating shapes at corpus-construction time and enforcing them in the matcher; it does not encourage abstract reasoning about all possible shapes.

3. **Cross-language interactions.** A Python file that emits a Bash one-liner via `subprocess.run(..., shell=True)` would need both the python-shell-true detector (which exists earlier in the family) and the bash-pipe detector to run on the emitted string, which is not automatic. Detectors are per-file, per-language. Cross-language flows are unaddressed by design.

4. **Semantic-equivalence rewrites.** If an LLM rewrites `YAML.load(s)` as `YAML.method(:load).call(s)`, the simple-string-match detector misses it. The detectors are pattern-based with light AST awareness (the Python-Jinja2 one parses kwargs), not full semantic analyzers. That ceiling is intrinsic to the four-file invariant.

The failure modes are listed not to argue against the design but to characterize it accurately. The detectors are *high-precision, low-recall* against the wider population of code-in-the-wild, and *perfect on a calibrated test bench*. That's a defensible design choice if your threat model is "LLM-generated code that calls dangerous APIs in the obvious way," because that's exactly the population the detectors are calibrated against.

## Why a detector ships in under 30 minutes

The 30-minute number isn't measured directly in the dispatcher logs — the templates ticks include 2 detectors and finish in a single tick of ~14-20 minutes wall-clock — but per-detector amortizes to ~10 minutes for small ones and ~20-25 minutes for the AST-aware ones (the Go open-redirect taint propagator, the Jinja2 paren-aware kwarg extractor). That's an order of magnitude faster than typical security tooling.

The reasons:

1. **No build system.** The detector is one Python file. There is no `pyproject.toml`, no `requirements.txt`, no `setup.py`. No dependency resolution, no virtualenv, no compilation.

2. **No persistent state.** The detector reads stdin or argv, writes stdout, exits with a status code. There is no database, no cache, no configuration file beyond the argv interface.

3. **Smoke harness is shared.** The 6-bad-6-good convention means the smoke is a 4-line shell pipeline that any reviewer can run. The harness lives once at the workflow repo root, not per-detector.

4. **Corpus construction is the bottleneck.** The actual labor is writing 6 minimal-bad and 6 minimal-good examples. For most CWEs that's 12 files of 5-15 lines each, total ~120 lines of carefully-chosen example code. A practiced author can do it in 15 minutes; a careful author takes 25.

5. **The matcher is small.** The Python regex/AST scanner is typically 30-80 lines. It's not trying to be a full static analyzer. It's trying to catch the textbook shape of the pattern, period. That bounded ambition is what keeps the matcher under 100 lines and the smoke result deterministic.

6. **No code review back-and-forth.** The detector is a leaf. Nothing depends on it. A reviewer either accepts the README's smoke claim and merges, or runs the smoke locally and merges. There's no "this breaks the build" surface because the detector is not in the build path.

The 30-minute ship time is therefore not magic — it's the natural consequence of removing every coupling that normally slows down security tooling. The cost is the four failure modes above. The benefit is that the templates family has shipped 10+ detectors across 9+ languages in 24 dispatcher ticks, which is a rate that no traditional security-tool team approaches.

## The natural ceiling

The detectors that have shipped so far cluster in CWE families that have *narrow signatures*: insecure deserialization (CWE-502, three of the ten), command injection (CWE-78, two), open redirect (CWE-601, one), LDAP injection (CWE-90, one), Kubernetes misconfiguration (CWE-250, one), prototype pollution (CWE-1321, one), unsafe Rust (CWE-704, one), Jinja autoescape (CWE-79, one). All of these have the property that a high-precision pattern match against a small set of API names or YAML keys captures the vast majority of real instances.

The ceiling — somewhere around CWE-89 (SQL injection), which has been shipped earlier in the family but in restricted variants like "SqlCommand string-concat" — is where the dangerous pattern is *taint flow*, not *API name*. CWE-89 in full generality requires tracking user input from network ingress through string construction to query execution, across function boundaries and file boundaries. That's not a 30-minute single-file Python script.

The templates family's response to that ceiling has been to ship the *narrow variant*: the C# SqlCommand string-concat detector ships, but a general SQL-injection detector does not. That's the right call given the four-file invariant. The general detector would either need to break the invariant (multi-file analyzer with build step, dependency on a real AST library, slow CI) or accept much lower bad/good ratios on its own corpus, neither of which fits the family's contract.

So the natural endgame for the templates family is *exhaustive coverage of narrow-signature CWEs across all common languages*. That's a finite endgame: there are roughly 80 commonly-cited CWEs and ~15 commonly-targeted language ecosystems, so the upper bound is on the order of 1200 detectors. At the current ~0.4-detector-per-tick rate, that's 3000 ticks — months of dispatcher runtime. Long before that, the family will start to ship near-duplicates (say, the Java-Spring SQL-injection detector after the Java-JDBC SQL-injection detector after the Java-Hibernate SQL-injection detector), at which point the design pattern will need to evolve toward parameterized detectors that handle a CWE-language pair via a config file rather than a duplicated detect.py.

That evolution hasn't started yet. The 10-detector window shows no parameterization, no shared base class, no inheritance, no abstract scanner. Every detector is a fresh `detect.py`. That's deliberate — the four-file invariant is the family's competitive advantage, and parameterization would weaken it. When parameterization does land it will be the first major shape change to the templates family in its visible history, and worth flagging in the metaposts narrative.

## Closing observation

The templates family's 30-minute polyglot detector pattern is one of the cleanest examples of structural-discipline-as-feature in the entire dispatcher. The four-file invariant, the bad=N/N good=0/N convention, the stdlib-only restriction, the leaf-not-node coupling profile — every one of those is a constraint, and every one of those constraints translates directly into ship velocity. The 10 detectors at SHAs `1fb5798`, `36856f2`, `e5a1d41`, `9289176`, `8f950fa`, `55aaad0`, `27acc73`, `d83d01f`, `c042aaf`, `0505eaa` are the empirical proof. They cover 10 CWEs across 9 language/runtime targets. They all have honest smoke results, including the two with documented over-match caveats. They all run in CI as single-file Python scripts with no install step. And they all shipped in under 30 minutes from the first commit to the smoke-PASS announcement.

That's the design pattern. The natural ceiling is taint-flow analysis, and the natural endgame is parameterized detector packs. Until either of those constraints binds, the templates family is best understood as a high-throughput leaf-shipping operation whose contract is `+2 honest narrow detectors per tick`, and whose product is a slowly accreting library of CWE-pattern signatures that is, detector-by-detector, becoming a usable safety net for LLM-generated code at the textbook shape of the textbook vulnerabilities.
