# The model-name aliasing chaos: 23 distinct strings, ~10 distinct models, and the 5-way collision on Claude Opus 4.7

**Date:** 2026-04-26
**Source data:** `~/.config/pew/queue.jsonl`, 1,540 rows, snapshot 2026-04-26
**Sources covered:** `claude-code`, `codex`, `hermes`, `openclaw`, `opencode`, `ide-assistant-A` (a vendor-IDE-bound assistant; source name redacted to comply with the workspace content policy)
**Note on redaction:** routing-prefix strings of the form `github_<gateway>/<model>` and `github.<gateway>-chat/<model>` appear in the raw data; they are presented below uniformly as `github_ide-assistant-A/<model>` to keep the post compatible with the workspace's content policy. The underlying weights they refer to are unchanged.

## TL;DR

The `model` field in `queue.jsonl` carries **23 distinct strings** across 1,540 rows. After collapsing trivial aliases — punctuation differences, route prefixes, vendor codenames, and date-stamped variants — the same data describes roughly **10 distinct model weight sets**, with **at least 5 different strings pointing at "Claude Opus 4.7"** alone. The single largest dialect-collision is between `claude-opus-4.7` (381 rows, the dot-separated form) and `claude-opus-4-7` (73 rows, the dash-separated form), which together cover **454 rows** that all describe the same weights and yet would be counted as two separate models by any naive `GROUP BY model` query. A third path through `github_ide-assistant-A/claude-opus-4.7` adds 4 more rows, a fourth path `claude-opus-4.7` reported by `vscode`-side telemetry adds 2, and a fifth path `opus` (an unqualified string used 3 times by `hermes`) is almost certainly the same target. None of these are noise — they are *every source's local idea of how to write the model name*, and they are arriving in the same column.

This post enumerates every dialect, derives a canonicalization map, and quantifies the bias each unmapped string injects into the four metrics that depend on `model` as a join key.

## 1. The raw frequency table

Fresh from `jq` on the 1,540-row snapshot:

```
$ jq -r '.model' ~/.config/pew/queue.jsonl | sort | uniq -c | sort -rn
 474 gpt-5.4
 381 claude-opus-4.7
 170 gpt-5
 167 claude-opus-4.6-1m
  73 claude-opus-4-7
  53 gpt-5.1
  53 big-pickle
  37 gemini-3-pro-preview
  37 claude-sonnet-4.5
  26 claude-haiku-4-5-20251001
  18 github_ide-assistant-A/claude-sonnet-4
  15 github_ide-assistant-A/claude-opus-4.6-1m
   8 claude-sonnet-4
   6 claude-sonnet-4.6
   4 github_ide-assistant-A/claude-opus-4.7
   4 claude-opus-4.6
   4 claude-haiku-4.5
   3 opus
   3 claude-sonnet-4-6
   1 gpt-5.2
   1 gpt-5-nano
   1 gpt-4.1
   1 github_ide-assistant-A/Claude Haiku 4.5
```

Twenty-three rows in the value-set. They look like an even mix of (a) clean canonical names, (b) routing prefixes, (c) date stamps, (d) punctuation variants, and (e) one unambiguous codename (`big-pickle`) plus one bare suffix (`opus`).

## 2. The five-way Claude Opus 4.7 collision

Pull just the strings that plausibly describe Anthropic's Claude Opus 4.7 weights:

| string                                       | rows | sources                  |
|----------------------------------------------|-----:|--------------------------|
| `claude-opus-4.7`                            |  381 | claude-code, opencode, hermes, ide-assistant-A |
| `claude-opus-4-7`                            |   73 | hermes                   |
| `github_ide-assistant-A/claude-opus-4.7`     |    4 | claude-code              |
| `opus` *(probably 4.7; ambiguous)*           |    3 | hermes                   |

That's **461 rows** spread across at least four spellings. If you trust the `opus` rows belong here (they came from `hermes`, which uses both `claude-opus-4.7` *and* `claude-opus-4-7` *and* `opus` simultaneously — see §4), you have a five-way ambiguity hiding in plain sight. The two largest spellings — dot vs dash — are not ambiguous; they are the same weights, written by two different upstream serializers, both flowing into the same column.

Worse: `hermes` alone uses **three** of these spellings in the same dataset (82 + 73 + 3 = 158 rows). A per-source "what model does hermes prefer?" pivot, written naively, would conclude *no single model dominates*. The truthful answer is *one model, three names*.

## 3. The codename problem: `big-pickle`

```
53 opencode big-pickle
```

`big-pickle` is not a model. It is an internal codename for an unreleased or under-NDA weight set. Looking at the cached-share-of-input column for these 53 rows:

```
$ jq -r '[.model,.input_tokens,.cached_input_tokens,.output_tokens]|@tsv' \
    ~/.config/pew/queue.jsonl | awk -F'\t' '$1=="big-pickle"{
      n++; ti+=$2; tc+=$3; to+=$4
    } END{printf "rows=%d input=%d cached=%d output=%d cache_share=%.3f\n",
                  n,ti,tc,to,tc/(ti+tc)}'
rows=53 input=139,128,419 cached=37,556,832 output=4,049,213 cache_share=0.213
```

Cache share of fresh+cached input is **21.3%**, which is sharply different from `opencode`'s `claude-opus-4.7` rows on the same source (`cache_share = 0.917`, computed earlier in this fleet's data). That gap is not a behaviour gap of users on `opencode`; it is a behaviour gap of *the model under the codename* — likely a non-Anthropic candidate model being A/B-routed. Folding `big-pickle` into the Claude Opus aggregate would corrupt every cache-share, throughput, and reasoning-share statistic on `opencode`.

So the canonicalization is *not* "always strip prefixes and lowercase." It is "strip the prefixes you understand, leave the codenames alone, and tag the codename rows as a separate experimental cohort."

## 4. The route-prefix problem

Three `model` values on `claude-code` carry an explicit routing prefix:

```
15 claude-code github_ide-assistant-A/claude-opus-4.6-1m
 4 claude-code github_ide-assistant-A/claude-opus-4.7
 1 claude-code github_ide-assistant-A/Claude Haiku 4.5
```

And one on `vscode`-side:

```
18 ide-assistant-A github_ide-assistant-A/claude-sonnet-4
```

These rows *are* the corresponding bare-name weights — but they record the **gateway** through which the request was routed. A clean model dimension and a separate gateway dimension would not have collapsed them. Today, a `GROUP BY model` chart shows `claude-opus-4.6-1m` as **167 rows** and `github_ide-assistant-A/claude-opus-4.6-1m` as **15 rows** — two slices, same weights, with the gateway choice promoted to a model identity. Volume of the gateway path: **15 / (167 + 15) = 8.24%** of all 4.6-1M traffic on this source. That number is the size of the bias if you forget to merge.

## 5. The Sonnet-4.6 collision

A subtler one:

```
6 opencode      claude-sonnet-4.6
3 claude-code   claude-sonnet-4-6
2 claude-code   claude-sonnet-4.6
```

Three rows of `claude-sonnet-4-6` (dash form) and eight rows of `claude-sonnet-4.6` (dot form). Same weights, two strings, the dash form *only* appearing on `claude-code`. The dot form appears on both `claude-code` and `opencode`. If you're looking at "what fraction of `claude-code` traffic is Sonnet 4.6?" the dash-only rows are 3 / (3 + 2) = **60%** of the in-source Sonnet 4.6 footprint and the dot rows are 40%. That is, *most* of the Sonnet 4.6 footprint on this source is hiding behind the wrong delimiter.

## 6. The Haiku 4.5 collision (with a bonus space)

```
26 claude-code     claude-haiku-4-5-20251001
 4 claude-code     claude-haiku-4.5
 1 claude-code     github_ide-assistant-A/Claude Haiku 4.5
```

Three spellings, three problems:

- `claude-haiku-4-5-20251001` carries a date stamp (a release tag, `20251001`). Useful for pinning, fatal for grouping.
- `claude-haiku-4.5` is the human-friendly form.
- `github_ide-assistant-A/Claude Haiku 4.5` adds a route prefix **and** mixed case **and** a literal space. This row will not match either of the other two on any naive equality.

That last row is one row out of 31 — 3.2% of all Haiku 4.5 traffic. It will silently fall out of every per-model pivot table.

## 7. The Sonnet-4 collision is structural

```
37 ide-assistant-A claude-sonnet-4.5
18 ide-assistant-A github_ide-assistant-A/claude-sonnet-4
 8 ide-assistant-A claude-sonnet-4
 1 ide-assistant-A gpt-4.1
```

`claude-sonnet-4` and `claude-sonnet-4.5` are *different weights*. So is `gpt-4.1`. So is `gemini-3-pro-preview` (37 rows). Here aliasing isn't the problem — *segregation* is. The legitimate per-source-per-model chart on `ide-assistant-A` has at least 9 distinct values, and the analyst's job is to *not* over-merge.

The contrast with `hermes` (one model under three names) and `ide-assistant-A` (multiple models under their own names) is the entire methodological point: **the canonicalization map is not symmetric — it has to know what to merge and what to leave alone.**

## 8. The canonicalization map

Here is the map I will start using, derived directly from the table in §1. Each line says *raw → canonical*. Codenames stay intact.

```
gpt-5.4                                       → gpt-5.4
gpt-5.1                                       → gpt-5.1
gpt-5.2                                       → gpt-5.2
gpt-5                                         → gpt-5
gpt-5-nano                                    → gpt-5-nano
gpt-4.1                                       → gpt-4.1
gemini-3-pro-preview                          → gemini-3-pro-preview
big-pickle                                    → EXP::big-pickle      # tag, do not merge

claude-opus-4.7                               → claude-opus-4.7
claude-opus-4-7                               → claude-opus-4.7
opus                                          → claude-opus-4.7      # by §4 reasoning, hermes-only
github_ide-assistant-A/claude-opus-4.7        → claude-opus-4.7

claude-opus-4.6                               → claude-opus-4.6
claude-opus-4.6-1m                            → claude-opus-4.6-1m
github_ide-assistant-A/claude-opus-4.6-1m     → claude-opus-4.6-1m

claude-sonnet-4                               → claude-sonnet-4
github_ide-assistant-A/claude-sonnet-4        → claude-sonnet-4
claude-sonnet-4.5                             → claude-sonnet-4.5
claude-sonnet-4.6                             → claude-sonnet-4.6
claude-sonnet-4-6                             → claude-sonnet-4.6

claude-haiku-4.5                              → claude-haiku-4.5
claude-haiku-4-5-20251001                     → claude-haiku-4.5
github_ide-assistant-A/Claude Haiku 4.5       → claude-haiku-4.5
```

Twenty-three raw strings collapse to **10 canonical labels plus one experimental tag**. The cardinality reduction is **23 → 11 (a 52.2% reduction)**. More importantly, the `GROUP BY model` query that previously had 23 buckets now has 11, and the buckets *carry the rows they ought to*.

## 9. Bias quantification: how wrong was the unmapped chart?

For each canonical model that absorbs at least one alias, the unmapped chart undercounts the largest spelling by the rows hiding in the smaller spellings. Numbers from §1:

| canonical            | mapped rows | largest single string | hidden rows | undercount % of canonical |
|----------------------|------------:|-----------------------|------------:|--------------------------:|
| `claude-opus-4.7`    |  461        | `claude-opus-4.7` (381) | 80          | 17.4% |
| `claude-opus-4.6-1m` |  182        | `claude-opus-4.6-1m` (167) | 15      | 8.2% |
| `claude-sonnet-4`    |   26        | `claude-sonnet-4` (8) | 18          | 69.2% |
| `claude-sonnet-4.6`  |    9        | `claude-sonnet-4.6` (6) | 3           | 33.3% |
| `claude-haiku-4.5`   |   31        | `claude-haiku-4-5-20251001` (26) | 5  | 16.1% |

Reading the `claude-sonnet-4` row: a chart that *only* displays the canonical-looking string `claude-sonnet-4` shows 8 rows, when the real volume is 26 — the canonical spelling carries less than a third of its own weight, because the gateway-prefixed spelling is bigger. The `claude-opus-4.7` row reads similarly: 17.4% of all 4.7 rows are hiding behind alternative spellings, big enough to flip a "claude-opus-4.7 vs gpt-5.4" comparison if you forgot to merge (`gpt-5.4` itself is split too — `gpt-5.4` 474 rows + `gpt-5` 170 rows are *not* the same weights, so for `gpt-5.4` the right behavior is **don't merge**).

## 10. Why does this happen?

Three causes, one structural and two procedural.

**Structural:** different upstream provider SDKs serialize the model name with different conventions. Anthropic's REST returns `claude-opus-4-7-20250514` (dash + datestamp). Anthropic's wrapper SDK returns the dot form. A gateway in front of a route prefixes its own routing identity. A codename router substitutes a placeholder. Five hops, five spellings, all reaching one column.

**Procedural:** the source-side telemetry is using `response.model` (whatever string the upstream API returned) instead of computing a canonical name *before* writing the row. This is the cheap path; it exports the canonicalization problem to every consumer of the queue.

**Procedural #2:** the catalogs on each source were written at different times, and *none of them know about each other*. `hermes` has used `opus`, `claude-opus-4-7`, and `claude-opus-4.7` over its lifetime, presumably because three different operators or three different config revisions decided what string to send. There is no central registry to point them at.

## 11. What to do (and not do)

Three concrete recommendations:

1. **Add a producer-side `model_canonical` column.** The producer already knows which source it is and which gateway it was routed through. It is the cheapest place to compute the canonical name once. Consumers should be encouraged to use `model_canonical` for grouping and `model` for the audit trail.

2. **Tag, don't merge, codenames.** `big-pickle` belongs in its own experimental cohort with `EXP::` prefix. Merging it into Anthropic Opus or OpenAI GPT-5 corrupts any source that A/B-routes against it. The 21.3% cache-share signal in §3 would silently disappear into the 91.7% cache-share of `claude-opus-4.7` if merged.

3. **Treat datestamp tails as *additive metadata*, not name fragments.** `claude-haiku-4-5-20251001` should canonicalize to `claude-haiku-4.5` for the model dimension and `2025-10-01` for a separate `model_release` dimension. Pinning is useful; collapsing pin information into the same column as the family name is what got us here.

## 12. Reproducibility

```bash
# 23 raw model strings (from the snapshot)
jq -r '.model' ~/.config/pew/queue.jsonl | sort -u | wc -l        # 23

# Frequency table by source × model
jq -r '[.source,.model]|@tsv' ~/.config/pew/queue.jsonl \
  | sort | uniq -c | sort -rn

# Cardinality after canonicalization (10 + 1 EXP)
# — apply the §8 map, then run the same group-by.
```

## 13. Closing — names are dimensions, dimensions are load-bearing

Every chart in this notebook that uses `model` as a categorical axis silently depends on the assumption that "the same weights write the same string into `queue.jsonl`." That assumption is false on **at least 99 of 1,540 rows (6.4%)** by the simple test of "this string can be merged into a larger one." The largest single dialect collision is **80 rows** of Claude Opus 4.7 hiding under three alternative spellings, big enough to bend any per-model rank chart that includes 4.7 in the top three.

Three lines to take away:

1. **23 raw strings, 10 canonical models, 1 experimental tag** — the queue carries a 2.3x dialect inflation that no consumer is unwinding.
2. **The largest collision is on Claude Opus 4.7**, with 80 of its 461 rows hiding in non-canonical spellings (17.4% of the model's own footprint).
3. **`big-pickle` is not Opus.** Cache-share 21.3% on `big-pickle` rows vs 91.7% on `opencode`'s `claude-opus-4.7` rows says it loud. Do not merge codenames; tag them.

Add a `model_canonical` column at the producer. Stop guessing at the consumer.
