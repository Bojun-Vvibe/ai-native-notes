# Source breadth per day: the 15% multi-source minority

Most of the discourse around AI-native developer tooling assumes that
the average user runs more than one AI CLI in parallel. The pitch decks
draw a Venn diagram: a coding assistant in the IDE, a chat CLI in the
terminal, an agentic dispatcher in the background, maybe a code-review
bot in CI. The implied user has all of them, switching between them by
context. It's a clean story, and it's also wrong for the typical day in
the typical dataset.

`pew-insights source-breadth-per-day` (v0.4.90) measures, for each UTC
calendar day, the count of distinct AI usage sources that produced any
non-zero token volume. A "source" here is the upstream tool ID:
`claude-code`, `codex`, `hermes`, `openclaw`, `opencode`,
`vscode-copilot`, etc. The lens emits one row per day with the source
count, the source list, the bucket count, and the day's token total.

The headline ratio in this dataset is the one most people would not
guess.

## The headline number

```
sourceCount: min=1  p25=1  median=1  mean=1.33  p75=1  max=6
single-source: 89    multi-source: 16    multi-share: 15.2%
```

Across 105 days, **89 days are single-source and only 16 are
multi-source**. Multi-source share is **15.2%**. The median, the p25,
*and* the p75 of source count are all 1. You have to go all the way to
the maximum to find the day with 6 distinct sources.

Said differently: in the data behind this lens, the modal day involves
exactly one AI CLI. Not two, not three. One. The "use everything in
parallel" workflow is real but rare.

## Why the modal day is single-source

Look at the source-list column for the single-source days and a clear
pattern emerges. From 2026-03-23 through 2026-04-08 — a 17-day stretch
— every active day reports exactly one source: `claude-code`. Earlier,
from 2026-01-22 through 2026-02-12, the single source is
`vscode-copilot`. The user is not doing a controlled experiment; they
are doing what almost every individual developer does, which is *pick a
primary tool and stay in it*.

The cost of multi-source workflows is not technical. It's cognitive.
Each AI CLI has its own:

- Prompt-shape conventions (slash commands, system prompts, file refs).
- Tool-call etiquette (when to ask, when to act, what to confirm).
- Failure modes (rate limits, context windows, cache behavior, output
  truncation rules).
- Pricing model (per-token, per-request, subscription, with or without
  cache discounts).

Switching between two of them mid-task means context-switching all of
the above. The marginal value of the second tool has to exceed that
switching cost, and for most tasks on most days, it doesn't. So the
typical day collapses to one source.

This has a direct implication for product comparisons. If you're trying
to win a user from `claude-code` to your tool, you are not competing for
a *slice* of their AI usage day. You are competing for the whole day.
The single-source mode is the default, and you either own it or you
don't get used.

## When does multi-source happen?

The 16 multi-source days cluster, and the cluster is recent.

Of the 16, the most concentrated pocket is **2026-04-13 onward**, where
9 of the next 13 days are multi-source. Before 2026-04-13, multi-source
days are scattered (2026-03-05, 2026-03-20, 2026-02-11). After
2026-04-13, they become routine.

The composition is also telling. The richest days are clustered in the
2026-04-19 → 2026-04-25 range and report the source list:

```
2026-04-25  3  hermes,openclaw,opencode
2026-04-24  3  hermes,openclaw,opencode
2026-04-23  4  claude-code,hermes,openclaw,opencode
2026-04-22  3  hermes,openclaw,opencode
2026-04-21  4  claude-code,hermes,openclaw,opencode
2026-04-20  6  claude-code,codex,hermes,openclaw,opencode,vscode-copilot
2026-04-19  4  claude-code,codex,hermes,openclaw
```

There is a stable backbone of `hermes`, `openclaw`, `opencode` — three
sources that fire together every day in this window. Then there's a
floating set of `claude-code`, `codex`, `vscode-copilot` that show up
on some days and not others. The backbone looks like a daemon stack
(it runs whether the human is at the keyboard or not); the floating
set looks like interactive sessions the human opened.

So the multi-source days aren't multi-source because the human is
juggling tools. They're multi-source because the human has stood up
*background infrastructure* that emits its own tokens, on top of
whatever interactive tool they happened to use that day. Those are
qualitatively different things and it would be a mistake to aggregate
them.

## Single-source vs. multi-source is a workflow generation marker

Walking the lens chronologically:

- **2026-01-22 → 2026-02-12** (early phase): single-source, almost
  always `vscode-copilot` or `claude-code`. Sometimes 2 sources, but
  mostly 1. This is the "interactive only" generation of the
  workflow. The human opens an AI tool, asks it things, closes it.
- **2026-02-25 → 2026-04-12** (consolidation phase): single-source,
  consistently `claude-code`. The user picked a primary and stayed
  with it. Daily token volumes are climbing but breadth is not.
- **2026-04-13 onward** (infrastructure phase): multi-source becomes
  the norm. New sources appear: `hermes`, `openclaw`, `opencode`.
  These names are not interactive CLIs; they are the user's own
  agentic plumbing. Daily token volumes climb another order of
  magnitude.

The 15.2% multi-share rate is therefore a *misleading aggregate*.
Across the dataset's full 105 days it's true. Across the most recent
13 days, multi-source share is approximately **9 / 13 = 69%**. The
distributional shift from "single-source by default" to "multi-source
by default" is not gradual — it happened in a roughly two-week
window in mid-April.

This is the kind of regime change that totals can't see. If you only
look at `tokens_per_day`, you see a smooth-ish growth curve. If you
look at `source_count_per_day`, you see a step function: 1, 1, 1, 1,
1, 1, ..., 4, 3, 4, 6, 4, 3, 3, 3.

## The 6-source maximum

The single day with 6 distinct sources is **2026-04-20**:

```
claude-code, codex, hermes, openclaw, opencode, vscode-copilot
```

This is also the day with the highest token volume in the entire
dataset (1.77B tokens). The two facts are not coincidental. A
6-source day requires:

- The interactive coding CLI (claude-code) ran.
- The reasoning-focused CLI (codex) also ran.
- The dispatcher daemon (hermes) ran.
- Two custom orchestrators (openclaw, opencode) ran.
- An IDE-coupled source (vscode-copilot) emitted tokens.

That is, almost every category of AI tool the user has installed
fired in the same UTC day. It's the saturation point of *breadth*,
the way 24-hour spans with 100% duty cycle are the saturation point
of *time*.

The multi-source share lens is asking: how often does the user
saturate breadth? Answer for this dataset: rarely. Even in the
infrastructure phase, the typical multi-source day is 3 sources, not
6. The 6-source day is an outlier, and the lens correctly flags it
as one.

## What single-source days actually mean

A single-source day with `claude-code` is not the same kind of day as
a single-source day with `vscode-copilot`. The lens reports them
identically — `sources=1, source-list=X` — but the underlying
behaviors differ:

- A `vscode-copilot` single-source day is almost always
  IDE-tab-completion driven. Token volumes are low (tens of KB to
  low MB), bucket counts are low, span hours are low. These are
  exploratory days where the user is barely using AI.
- A `claude-code` single-source day can be enormous. 2026-04-08 is
  single-source with 107.9M tokens across 11 buckets. 2026-04-03 is
  single-source with 160.6M tokens. The user is fully engaged with
  one tool, doing real work inside it for the whole day.

The lens output bottom-row bucket count and token total are critical
for distinguishing these. A naive aggregation that just counted
"single-source days = 89" would lose the fact that those 89 days span
three orders of magnitude in token volume and two orders of magnitude
in bucket count.

If you wanted to refine the lens, you'd want to add a per-source
share within the day — e.g. "claude-code: 95%, codex: 5%". That
would tell you whether the multi-source days are truly multi-source
in workload terms, or whether one source dominates and the others
are background noise. The current lens lumps the 99/1% day with the
50/50% day; both report `sources=2`.

## The asymmetry is the story

The most important number in this lens output is not the maximum or
the median — it's the gap between them.

```
median: 1
mean:   1.33
max:    6
```

A median of 1 with a max of 6 and a mean of only 1.33 means the
distribution is heavily right-skewed but the right tail is thin. Most
days are clustered at the floor; a small handful of recent days have
pulled the mean barely above 1. The single-source/multi-source split
is **89/16**, almost 6:1 in favor of single-source.

This asymmetry has product-design consequences:

**1. Tools that try to *be* the whole stack** (one CLI doing IDE,
chat, agentic loop, review, refactor, deploy) are betting on the
89/16 ratio. They're betting that users will keep collapsing breadth.
If the future of AI-native development is single-source-by-default
plus background daemons, the integrated CLI wins.

**2. Tools that try to *integrate* with the stack** (orchestrators,
routers, MCP servers) are betting on the opposite — that breadth will
grow. They're betting on the recent 9/13 trend extrapolating, where
multiple tools coexist by design and the orchestrator is the glue.

**3. Telemetry that aggregates across sources without preserving
breadth** loses the most interesting signal. The right primary key
for usage analysis on this dataset is `(day, source)`, not `day`.
Roll-ups to the day level should always carry the source-count
column with them.

## What the lens does not capture

A few caveats:

- It counts *any* non-zero token activity. A 1KB heartbeat from
  `vscode-copilot` counts the same as 1GB of `claude-code` traffic.
  The bucket and token columns let you see this, but the
  `sourceCount` column doesn't weight by it.
- It uses UTC days. A user whose local timezone straddles UTC
  midnight will have their workday split across two rows, possibly
  with different source counts on each side.
- It does not look at *order* within the day. A day with sources
  `[claude-code, hermes]` reports the same as a day with
  `[hermes, claude-code]` even if the morning was claude-code-only
  and the afternoon was hermes-only.

These are not bugs; they're the lens being narrow. Source breadth is
asking one question — *how many distinct AI tools fired on this day?*
— and answering it. Anything else is a different lens.

## The takeaway

In a dataset that is heavily skewed toward heavy AI usage (105 days,
8.58B tokens, recent days at 24-hour duty cycle saturation), the
**modal day is still single-source**. 89 of 105 days. p75 of 1.
Median of 1.

The narrative of the multi-tool AI-native developer is, for the
typical day, false. It becomes true in narrow windows and for users
who have built their own infrastructure. For everyone else, it's one
CLI per day, picked early in the adoption cycle and rarely
revisited.

The 15.2% multi-source minority is where the future is being built.
The 84.8% single-source majority is where the present lives.
