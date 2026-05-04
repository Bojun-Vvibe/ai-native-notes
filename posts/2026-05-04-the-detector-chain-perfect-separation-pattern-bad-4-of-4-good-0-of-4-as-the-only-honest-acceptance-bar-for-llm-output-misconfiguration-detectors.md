---
title: "The detector-chain perfect-separation pattern: bad=4/4 good=0/4 as the only honest acceptance bar for LLM-output misconfiguration detectors"
date: 2026-05-04
tags: [ai-native-workflow, detectors, evaluation, security, methodology]
est_reading_time: 8 min
---

## The pattern, stated up front

Every detector that has shipped into the `ai-native-workflow` `templates/` tree over the last several weeks carries the same little tag in its commit message or its README acceptance line: `bad=4/4 good=0/4 PASS`. It's the kind of phrase that looks like a throwaway — four-out-of-four bad samples flagged, zero-out-of-four good samples flagged — and the first time you see it you might assume it's just a sanity check. It is not. It is the entire acceptance bar. It is also, after watching the chain grow from a handful of detectors to (as of HEAD `1af024b`) the latest two additions — `llm-output-coredns-acl-allow-all-detector` (commit `99ddc84`) and `llm-output-tinyproxy-allow-all-detector` (commit `1af024b`) — the only acceptance bar I've found that consistently survives contact with real LLM output.

The thesis of this post is small and stubborn: **for misconfiguration-class detectors that grade text emitted by a model, perfect separation on a tiny curated corpus is a stronger signal than near-perfect separation on a large noisy one**, and the chain is what makes that bar credible rather than cherry-picked.

## What "bad=4/4 good=0/4" actually means

Each detector in the tree is a directory. The shape is invariant — you can verify it on any of them, e.g.:

```
templates/llm-output-tinyproxy-allow-all-detector/
  README.md
  detector.py
  examples/
  verify.sh
```

`examples/` contains exactly eight fixture files, partitioned into four "bad" (the misconfiguration is present) and four "good" (the misconfiguration is absent or actively defended against). `verify.sh` runs `detector.py` against all eight and prints two counts: how many bad were flagged, how many good were flagged. The pass condition, hard-coded into the chain's review discipline, is `bad == 4 AND good == 0`. Three-of-four on the bad side is a fail. One-of-four on the good side is a fail. There is no F1, no precision-recall curve, no ROC, no threshold to tune. The answer is binary.

That sounds primitive. It is primitive — that is the point. The two recent additions, `coredns-acl-allow-all` (CoreDNS `acl` plugin with an action that grants all sources) and `tinyproxy-allow-all` (a `tinyproxy.conf` whose `Allow` directive opens the proxy to the world), both shipped under that bar. So did everything before them: the SSH `PermitEmptyPasswords yes` detector (`42b18e0`), the Squid `http_access allow all` detector (`f993959`), the Postfix relay-permit-anyone detector (`7015539`), the Apache `Options Indexes` detector (`16bfec3`), the Kibana `elastic` default-username detector (`5d96577`), the Solr JMX-no-auth detector (`9f4e23e`), the WordPress `WP_DEBUG_DISPLAY` detector (`368b764`), the ProFTPD anonymous-allowed detector (`6d0cf6a`), and the rest of the tree back through the HBase, Hive, Artemis, WildFly, etcd, MinIO chain to the CoreDNS commit just one before HEAD. Every single one ships with the same four-by-four PASS line.

## Why the small-N matters more than it looks

The instinct of anyone trained on modern ML evaluation is to scoff at N=8. That instinct is wrong here, and it is wrong for a specific reason: **the population of LLM-emitted misconfigurations is not stationary**. The detector is not classifying inputs from a fixed distribution; it is reading text that an arbitrary model produced under arbitrary prompting, often as a fragment of a larger code suggestion, often in a fenced block with surrounding prose. There is no test set you can hold out that looks like the test set you'll see next week.

What you can do is curate four maximally-different *bad* shapes and four maximally-different *good* shapes for the specific misconfiguration in question. For `tinyproxy-allow-all`, "bad" includes the obvious `Allow 0.0.0.0/0`, the less-obvious `Allow ::0/0` IPv6 form, the comment-out-the-deny anti-pattern, and a config where Allow is implicit because no Deny is present. "Good" includes a tightly-scoped `Allow 192.168.0.0/16`, a configuration that uses upstream firewall rules with a comment saying so, a `BasicAuth`-protected setup, and a deliberately-empty allow list. If your detector flags all four bad and none of the four good, you have demonstrated something narrow but real: it discriminates on the *rule itself*, not on incidental tokens.

This is why `bad=3/4 good=0/4` is a fail, not a near-pass. The one bad case you missed is, by construction, the one that exposes a regex that anchored on incidental tokens rather than on the rule. You don't get to tune the threshold; you have to fix the detector. And `bad=4/4 good=1/4` is the same kind of fail: the one good case you flagged is, by construction, the one that proves your detector overfires on a benign-but-superficially-similar pattern. Every PASS in the chain is a claim that the author found and resolved both of those failure modes before merge.

## The chain effect: methodology certified by repetition, not by argument

A single `bad=4/4 good=0/4` line is a claim. Forty of them, all under the same bar, by the same author, on misconfigurations spanning HTTP servers, mail relays, DNS resolvers, key-value stores, message brokers, search engines, JVM management consoles, FTP daemons, and CMS debug switches, is something different. It is a methodology that has been instantiated enough times that the bar itself is now the artifact, and each new detector is just an application of it.

The chain at HEAD `1af024b` reads, going backwards from the tip:

- `1af024b` — tinyproxy allow-all
- `99ddc84` — coredns ACL allow-all
- `6756e17` — minio anonymous bucket policy
- `7137fd7` — etcd client-cert-auth disabled
- `76f2284` — wildfly management interface no security realm
- `ce67c64` — artemis broker security disabled
- `4a9166c` — hive-server2 authentication none
- `f4ed383` — hbase security authentication simple
- `6d0cf6a` — proftpd anonymous-allowed
- `368b764` — wordpress WP_DEBUG_DISPLAY true

That is ten detectors, each independent in the sense that they touch unrelated config files in unrelated ecosystems, but identical in the sense that they all clear the same four-by-four bar. The cross-ecosystem coverage is what makes the methodology claim non-trivial: if the bar were too easy, you would expect at least one config-format quirk (CoreDNS's Caddyfile-derived syntax, etcd's TOML, MinIO's JSON policy, Artemis's `broker.xml`) to break it. None of them do, because the bar is not about syntax-matching; it is about **rule-shape discrimination**.

## What this replaces

The default acceptance bar for "AI-output safety detector" work in 2025 was, broadly: build a labeled corpus of a few thousand snippets, train a classifier, report F1 in the high 90s, ship. That bar has two pathologies. First, the labeled corpus is almost always sampled from one or two upstream pipelines and the F1 number is a measurement of how well you fit *those specific pipelines' typical output*, which decays the moment a different model or a different prompting style enters the picture. Second, the F1 number itself averages over an unknown mix of failure modes; a detector that scores 0.96 might be quietly failing on the 4% that matters most — a category-shaped failure invisible to the aggregate metric.

The four-by-four bar replaces both pathologies with one assertion: *the detector either discriminates on the rule or it does not*. If it does, it will pass the curated eight. If it does not, no amount of corpus-tuning will save it. The detector is a function, not a model; the eight examples are a specification of what that function should compute, not a training set.

## Where the bar can fail honestly

I want to be clear about the limits.

The four-by-four bar does not guarantee a low false-positive rate in the wild. It guarantees that the detector did not false-positive on the four maximally-confusing benign cases the author could think of. Real LLM output will produce shapes the author did not think of, and some of them will fire. The bar is a *necessary* condition for shipping, not a sufficient one for "production-ready at scale." A detector that fires on real LLM output two percent of the time is still cheap to triage and easy to refine; a detector that *misses* the actual bad rule one percent of the time is a security failure.

It also does not generalize beyond the misconfiguration class. The detectors in this chain all check for the presence of a specific structural anti-pattern — an open allow rule, an authentication-disabled flag, a default credential, a debug-display switch. Detectors for, say, "the model produced an SQL query that is vulnerable to injection in the surrounding application context" cannot be reduced to four-by-four, because the relevant property is not a property of the snippet but of the snippet *plus* the runtime. The chain is honest about its scope: every detector in `templates/` is named `llm-output-<technology>-<flag-or-rule>-detector`. The naming itself constrains the contract.

And it does not survive author drift. The bar works because the same author is curating the eight examples, applying the same notion of "maximally-different bad" and "maximally-different good." Hand the directory to a new author and the four-by-four PASS line means less, until that author has internalized the same curation discipline. The chain is, in this sense, a craft artifact rather than a turnkey framework.

## What I would do differently

If I were starting a similar chain from scratch tomorrow, I would lift exactly two things from this one and change exactly one.

Lift first: **the binary pass condition with no threshold to tune**. Every time a detector framework has tried to ship a "score" instead of a flag, the score has become a knob, and the knob has become a way to make the dashboard green. The four-by-four bar refuses to give you a knob. You either fix the detector or you do not ship.

Lift second: **the requirement to articulate four maximally-different bad cases and four maximally-different good cases before the detector exists**. This is essentially test-driven development for security detectors, and it forces the author to think about *why* the rule is a rule before they think about *how* to match it. The detectors in this chain are short — `detector.py` is typically under 50 lines — because the eight examples did the design work.

Change one: **add a third partition**. Four bad, four good, four "near-miss-but-actually-fine" — configurations that look superficially like the bad pattern but are explicitly not the rule (the proxy has `Allow 0.0.0.0/0` *behind* a separate IP-restricted listener, the DNS server has `acl 0.0.0.0/0` but only inside a `view` clause that is itself restricted). The current bar implicitly asks for these in the "good" partition, which works, but making them their own named slice would make the contract more legible to a reader trying to extend the chain.

For now, the chain at HEAD `1af024b` is forty-some detectors deep, every one of them carrying the same four-character verdict, every one of them shipped under the same bar. That is, in this corner of the AI-native-workflow problem space, the most honest acceptance methodology I have seen.

## Links

- `ai-native-workflow` HEAD `1af024b` — `feat(templates): add llm-output-tinyproxy-allow-all-detector`
- `ai-native-workflow` `99ddc84` — `feat(templates): add llm-output-coredns-acl-allow-all-detector`
- `ai-native-workflow` `6756e17` — `feat(templates): add llm-output-minio-anonymous-bucket-policy-detector`
- `ai-native-workflow` `7137fd7` — `feat(templates): add llm-output-etcd-client-cert-auth-disabled-detector`
- `ai-native-workflow` `6d0cf6a` — `feat: add llm-output-proftpd-anonymous-allowed-detector template (bad=4/4 good=0/4 PASS)`
- `ai-native-workflow` `368b764` — `feat: add llm-output-wordpress-wp-debug-display-true-detector template (bad=4/4 good=0/4 PASS)`
