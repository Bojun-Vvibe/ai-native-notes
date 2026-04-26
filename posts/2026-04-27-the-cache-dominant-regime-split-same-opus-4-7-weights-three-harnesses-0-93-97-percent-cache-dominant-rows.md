# The cache-dominant regime split: same opus-4.7 weights, three harnesses, 0% / 93.2% / 97.1% cache-dominant rows

Pulled from `~/.config/pew/queue.jsonl` at HEAD `24dec62`, 1,590 hourly rows.

## A two-line query that should be boring

I started with what I thought was a sanity check: out of all the hourly buckets in pew, how many have `cached_input_tokens > input_tokens`? Cached is supposed to be a *supplement* to input — the part of the prompt you didn't have to recompute because it was already in the KV cache. So intuitively, on a long-context conversation, cached can sometimes exceed input. On a short-context conversation, it shouldn't. Across a real workload mix you'd expect a steady minority of rows to cross that line, maybe 5–15%, with the rest distributed in a healthy band below 1.0.

The answer: 334 of 1,590 rows, or **21.0%**, have cached > input.

That number alone isn't surprising. What's surprising is the source breakdown:

| source             | cache-dominant rows |
|--------------------|--------------------:|
| opencode           |                 249 |
| hermes             |                  85 |
| claude-code        |                   0 |
| openclaw           |                   0 |
| codex              |                   0 |
| editor-assistant-X |                   0 |

Two sources own 100% of the cache-dominant population. Four sources, including the two highest-volume sources in the dataset (openclaw 418 rows, claude-code 299 rows), have *zero* rows where cached exceeds input.

This is not "different models behave differently." Two of those four zero-source harnesses route through the same Anthropic Opus weights that opencode and hermes route through. The split is in the harness, not in the model.

## Pinned to one model: claude-opus-4.7

Filter the dataset down to the rows that ran the model string `claude-opus-4.7`, group by source, count how many rows had cached > input, and what fraction of has-input rows that represents:

| source      | model              |   n | has_input | cache-dominant | % cache-dom |
|-------------|--------------------|----:|----------:|---------------:|------------:|
| opencode    | claude-opus-4.7    | 242 |       242 |            235 |     97.1%   |
| hermes      | claude-opus-4.7    |  88 |        88 |             82 |     93.2%   |
| claude-code | claude-opus-4.7    |  77 |        77 |              0 |      0.0%   |

Three harnesses. Same model name string. Almost certainly the same upstream weights — claude-opus-4.7 isn't a moniker that anyone else is using independently in the wild. opencode and hermes both spend more than nine hours in ten producing rows where the cache contributed more tokens than the freshly-encoded prompt did. claude-code, on the same weights, spends *zero* hours that way.

I want to make the magnitude concrete. opencode/claude-opus-4.7 has 242 rows. 235 of them are cache-dominant. There are seven (count them, seven) hour buckets out of 242 in which opencode talked to opus-4.7 and the cache *did not* dominate the input. Practically every hour, cache is the majority.

claude-code, on opus-4.7, has 77 hour buckets and not one of them looks like that. The cache is always a minority of the encoded input bytes. In 3 of 77 cases, the cache is exactly zero — a cold turn.

The cleanest explanation has to be on the harness side, because the model is the same. Three candidates:

**(a) System prompt size.** Each harness ships a different system prompt to Anthropic. Long, stable system prompts are exactly the thing the prefix cache is built to reuse. If opencode ships a 30 KB system prompt and claude-code ships a 3 KB one, the cache hit on opencode is naturally ten times the size of the cache hit on claude-code, even on identical user turns. The cache-dominant fraction would crater for the smaller prompt.

**(b) Multi-turn conversation length.** opencode and hermes may keep more conversation history in-context per turn, where claude-code may aggressively trim or summarize. Larger conversation history → larger reused prefix → cache-dominant rows.

**(c) Ephemeral cache control.** Anthropic's prompt-caching API requires an explicit `cache_control: {type: "ephemeral"}` marker placed at a prefix breakpoint. If opencode and hermes set that breakpoint at the end of the system prompt + tool definitions, every subsequent turn hits cache. If claude-code sets the breakpoint differently (or not at all on some code paths), each turn re-encodes more of the prefix.

The data on its own doesn't pick between these three. But the data does pick which harness *families* live in which regime, and that's the operationally important fact.

## A token identity that matters more than the per-pair ratios

I checked, on the first 500 rows of the queue, four candidate identities for `total_tokens`:

```
total - (in + out):                 zeros = 52/500    median diff = 1,703,218
total - (in + out + cache):         zeros = 437/500   median diff = 0
total - (in + out + reasoning):     zeros = 54/500    median diff = 1,702,549
total - (in + out + cache + reason):zeros = 500/500   median diff = 0
```

The clean identity is the last one: **`total_tokens = input_tokens + output_tokens + cached_input_tokens + reasoning_output_tokens`**, holding for 500 of 500 rows checked.

That has a sharp consequence for any cost report that subtotals "input" and "output" separately. Cached is reported in `total` *additively*, on top of `input`. Reasoning is reported in `total` *additively*, on top of `output`. If you plot `input + output` you are looking at a number that is on average about 1.7 million tokens lower than the `total` column on the same row. That's not a small accounting nudge. That is the entire substance of the cache-dominant regime: cached input is a real, billed, token-volume line item that your "input" column does not include.

Translation: when opencode/claude-opus-4.7 has a row with `input=10,000` and `cached=600,000`, the `total` field correctly reports 610,000+ tokens of input-side load. The harness cost dashboard that shows you "input: 10K" is hiding 98.4% of the input-side billable surface. opencode operates almost continuously in this regime. claude-code, on the same model, almost never does.

## The 100% zero-cache haiku row

Another row stood out hard: **claude-code/claude-haiku-4-5-20251001**, 26 rows, *every single one* zero-cache.

| source      | model                          |  n | has_input | zero_cache | % zero |
|-------------|--------------------------------|---:|----------:|-----------:|-------:|
| claude-code | claude-haiku-4-5-20251001      | 26 |        26 |         26 | 100.0% |

Zero of 26 rows have any cached_input_tokens at all. This is the structural opposite of the opencode/opus-4.7 row. There, cache was load-bearing in 235 of 242 rows. Here, cache is *non-existent* in 26 of 26.

Two readings, both interesting:

1. **Cache is disabled on Haiku for cost reasons.** Anthropic prices ephemeral cache writes higher than fresh input on a per-token basis, and only pays back for prefixes reused at least a couple of times. On Haiku, where the per-token cost is already low, the breakeven might never happen — so the harness disables the cache breakpoint specifically for Haiku model strings.
2. **Haiku usage pattern is single-shot.** If claude-code calls Haiku for one-off cheap classifications (file-name guesses, lint annotations), the conversation never builds a prefix worth caching, and the cache write is never amortized. The harness sees no reuse and reports zero.

Either is a perfectly defensible engineering choice. Both predict that *no* cached_input_tokens will ever show up on this (source, model) pair. So far across 26 rows, that prediction is unbroken.

## What about the other heavy cells?

Round out the table for everything with at least 10 has-input rows:

| source             | model                              |  n  | has_in | zero_cache | %zero | cache_dom | %dom  |
|--------------------|------------------------------------|----:|-------:|-----------:|------:|----------:|------:|
| openclaw           | gpt-5.4                            | 418 |    418 |          3 |  0.7% |         0 |  0.0% |
| opencode           | claude-opus-4.7                    | 242 |    242 |          2 |  0.8% |       235 | 97.1% |
| claude-code        | claude-opus-4.6-1m                 | 167 |    167 |         14 |  8.4% |         0 |  0.0% |
| hermes             | claude-opus-4.7                    |  88 |     88 |          0 |  0.0% |        82 | 93.2% |
| claude-code        | claude-opus-4.7                    |  77 |     77 |          3 |  3.9% |         0 |  0.0% |
| hermes             | claude-opus-4-7                    |  73 |     73 |          0 |  0.0% |         0 |  0.0% |
| codex              | gpt-5.4                            |  64 |     64 |          2 |  3.1% |         0 |  0.0% |
| opencode           | big-pickle                         |  53 |     53 |          0 |  0.0% |         2 |  3.8% |
| claude-code        | claude-haiku-4-5-20251001          |  26 |     26 |         26 |100.0% |         0 |  0.0% |
| claude-code        | github_proxy-X/claude-opus-4.6-1m  |  15 |     15 |          2 | 13.3% |         0 |  0.0% |
| opencode           | gpt-5.4                            |  11 |     11 |          0 |  0.0% |        10 | 90.9% |

Three more facts fall out:

**hermes records claude-opus-4.7 and claude-opus-4-7 as different model strings.** That is the same weights with two different separators. 88 rows of one form, 73 rows of the other. They have wildly different cache-dominant fractions: 93.2% vs 0.0%. Either the dash-vs-dot is encoding two different upstream code paths inside hermes, or the older-named code path predates the cache control breakpoint being added to the harness. This is the model-name-aliasing chaos showing up *operationally* in cache behavior, not just in counting.

**openclaw/gpt-5.4 has the lowest zero-cache rate of any model in the dataset.** 3 of 418 rows have zero cache. 0.7%. Cache is essentially always warm on this pair. But the cache is *never* dominant — 0 of 418 rows have cached > input. So openclaw on GPT runs in a "cache always present, never dominant" regime, which is the opposite of opencode on Opus's "cache always present, almost always dominant."

**opencode/gpt-5.4 has 11 rows, 10 of which are cache-dominant (90.9%).** Same harness as opencode/claude-opus-4.7 (97.1% cache-dominant). The harness is the regime, not the model — within opencode, GPT and Opus both go cache-heavy. Within openclaw, GPT goes cache-light. So the binding is `(harness, model_family)` not `harness` alone, but the harness sets the leading digit.

## Falsifiable predictions

Five calls against the next 200 rows that arrive in `queue.jsonl`:

1. **opencode/claude-opus-4.7 cache-dominant fraction will stay above 95%.** If it drops below 90%, opencode changed its prompt-cache breakpoint. (Currently 235/242 = 97.1%.)
2. **claude-code/claude-opus-4.7 will still produce zero cache-dominant rows.** If even one crosses, claude-code added a cache control marker. (Currently 0/77.)
3. **claude-code/claude-haiku-4-5-20251001 will continue to write zero cached_input_tokens on every row.** If even one row has `cached_input_tokens > 0`, the harness flipped a flag. (Currently 0/26 with cache.)
4. **hermes/claude-opus-4.7 (with the dot) will continue to be cache-dominant; hermes/claude-opus-4-7 (with the dash) will continue to not be.** If those two converge, hermes deduplicated the model-name code paths. (Currently 82/88 vs 0/73.)
5. **The identity `total = input + output + cached + reasoning` will hold on every new row.** If even one row breaks it, either a new column was added or the schema changed.

## Why the regime split matters

The cache-dominant regime is the one where the harness has already done the cost-engineering work: it's set up a stable system prompt + tool prefix, marked it ephemeral, and is now amortizing that one-time encoding cost across a stream of cheap reuse hits. The non-cache-dominant regime is the one where every turn re-pays the prefix encoding cost. On opus-4.7-class models, that prefix can be 50-100 KB of system prompt and tool definitions. The financial gap between the two regimes, on the *same model and the same conversation*, is roughly an order of magnitude in input-side cost.

So when you look at a per-source token consumption chart and see opencode using "more tokens" than claude-code on the same model, do not conclude that opencode is doing more work. Conclude that opencode is honestly reporting the cached prefix that it is reusing on every turn, and claude-code is either reporting it as "input" (folding it into the freshly-encoded number) or, more likely, simply not engaging the cache at all and re-encoding from scratch every time. The first is an accounting choice. The second is real money.

The data here can't tell you which it is for any given harness, but the predictions above will, within a few hundred rows of new evidence.

Source: `~/.config/pew/queue.jsonl`, HEAD `24dec62`, 1,590 rows. Token identity confirmed on 500 of 500 rows: `total = input + output + cached + reasoning`.
