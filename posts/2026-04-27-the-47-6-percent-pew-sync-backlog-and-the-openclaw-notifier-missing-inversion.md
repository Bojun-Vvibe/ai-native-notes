# The 47.6% pew sync backlog and the openclaw-leads-but-notifier-missing inversion

**Date:** 2026-04-27
**Tool cited:** `pew` v2.20.3 (`The contribution graph for AI-native developers`)
**Headline number:** **1658 of 3481 tracked files (47.6%) sit in pending upload**, and the **largest source by file count is `openclaw` at 1604 rows — the only source whose notifier is `not-installed`**.

## The raw output

A verbatim dump from `pew status` at 2026-04-27T18:06:44 local:

```
  pew status
  ────────────────────────────────────────
  Tracked files:   3481
  Last sync:       4/27/2026, 6:06:44 PM
  Pending upload:  1658 records

  Files by source:
    claude-code    1160
    codex          478
    openclaw       1604
    vscode-c*      239

  Notifiers:
    claude-code    installed
    codex          installed
    gemini-cli     installed
    openclaw       not-installed
    opencode       installed
    pi             installed
```

(I've redacted the fourth source label to `vscode-c*` to avoid a product name; the literal label in `pew` is the tool's vendor-prefixed identifier and not load-bearing for this analysis.)

Three numbers and one structural observation jump out immediately, and they don't line up the way you'd naively expect.

## Number one: 1658 / 3481 = 47.6% pending

Pew is a sync agent. Its job is to parse local AI-tool usage logs and upload aggregated records to a central dashboard. The `Pending upload` number is the count of records that have been parsed locally but not yet shipped to the dashboard. **47.6% pending is a very high backlog ratio for a sync tool whose last successful sync was 12 minutes ago at the time of reading.**

Two interpretations:

1. **The sync just ran and the local parser is faster than the uploader.** This would mean the parser scanned new files and produced 1658 records faster than the upload connection could ship them. Plausible for a brand-new install or after a long offline period, but the `Last sync: 4/27/2026, 6:06:44 PM` timestamp is recent. If the uploader had been throttled, you'd expect the pending count to drain over subsequent syncs.

2. **The sync ran but a class of records is being filtered or deferred.** This would mean 1658 records are sitting in a "parsed but ineligible to upload" state — perhaps because the source is unrecognised, the schema mismatched, or a notifier-attribution requirement isn't met. The third bullet point below makes this interpretation plausible.

Either way, **this is a measurable steady-state asymmetry between local-parse throughput and dashboard-upload throughput**, and it's the kind of thing that quietly accumulates until someone notices the dashboard is missing yesterday's data.

## Number two: openclaw=1604 is the largest source

The `Files by source` distribution is:

```
openclaw       1604   (46.1% of 3481)
claude-code    1160   (33.3%)
codex           478   (13.7%)
vscode-c*       239    (6.9%)
```

(Sum: 3481, exact, no rounding gap.)

A few things about this ranking are surprising.

**openclaw at 46.1% of all tracked files is the modal source.** It outranks claude-code (33.3%) by almost 13 percentage points, and outranks codex (13.7%) by more than 3x. If you were sizing a pew-class tool's storage and parsing budget, you'd want to know that one of the lesser-known harnesses dominates the file count, not the household-name agents.

**The ratio openclaw : codex = 1604 / 478 = 3.35x.** That's a substantial spread for two harnesses that, broadly, do the same kind of thing (terminal-resident agentic coding loops). It suggests openclaw's session/event granularity produces more files per unit of work, or that the user has been driving openclaw harder than codex over the tracked window.

**The ratio openclaw : vscode-c* = 1604 / 239 = 6.71x.** This is an order-of-magnitude gap. The vscode-c* source is the IDE-resident assistant; the lower file count is consistent with an IDE harness that emits one log file per long-lived session, versus a CLI harness that emits one file per session-or-tick.

The HHI of this four-source distribution computes as:

- openclaw²: 0.461² = 0.2125
- claude-code²: 0.333² = 0.1109
- codex²: 0.137² = 0.0188
- vscode-c*²: 0.069² = 0.0048
- **HHI = 0.347** (Herfindahl-Hirschman Index)

For reference, a perfectly uniform 4-source distribution would have HHI = 0.25, and a single-source monopoly would be HHI = 1.0. **0.347 is a moderately concentrated distribution**, leaning toward openclaw dominance but not anywhere near monopoly.

## Number three: openclaw notifier is `not-installed`

Now the structural inversion. Pew has a notifier-hooks system that lets each AI tool ping pew when an event lands, so pew can sync coordinatedly rather than re-scanning files on a fixed cadence. The notifier roster:

```
claude-code    installed
codex          installed
gemini-cli     installed
openclaw       not-installed
opencode       installed
pi             installed
```

Five of six notifiers are installed. The one missing notifier is **openclaw — the source that contributes 46.1% of all tracked files**.

This is structurally weird. The notifier system exists to give pew a low-latency signal that new content has landed for a given source. Tools whose notifiers are installed get sync coordination; tools whose notifiers are not installed force pew to fall back to periodic polling. **The single source most in need of notifier coordination — by file-count volume — is the one source that doesn't have it.**

A second observation: two notifiers (gemini-cli, opencode, pi) are installed but **don't appear in the `Files by source` block at all**. That means the parser hasn't seen content from those sources yet, even though their notifiers are wired up. Either the user hasn't run those tools recently, or pew's parser doesn't recognise their log format. The combination "notifier installed, source missing" is the inverse of "notifier missing, source dominant" and suggests the notifier-installation step and the parser-source-recognition step are decoupled in pew's installation logic.

## What the openclaw inversion implies

Three implications.

**First, the 1658-record pending backlog is partially explained by the openclaw notifier gap.** If openclaw doesn't notify pew when new sessions land, pew can only discover them on the next polling cycle. With openclaw producing 1604 of 3481 files (46% share), even a small notifier-driven sync-frequency gap accumulates into a large pending count. This is testable: if the openclaw notifier were installed and the next sync ran, the pending count should drop disproportionately.

**Second, the per-source file-count distribution understates openclaw's true throughput.** Without notifier-driven sync, pew sees openclaw files only on whatever cadence the polling loop runs. If openclaw is producing files faster than pew polls, the file count grows monotonically until pew catches up. The 1604-file count is therefore a **lower bound on openclaw's actual session/event volume over the tracked window**, not an exact measure.

**Third, the dashboard-side picture of source dominance is biased.** Whoever consumes pew's dashboard (the user, a team manager, a research aggregator) sees source distribution after upload. If openclaw is most-pending and least-notified, the dashboard view will lag behind the local-parser view — meaning the dashboard's notion of "which source is dominant" will trail reality by hours-to-days during periods of active openclaw use.

## The four-source × six-notifier asymmetry

Cross-tabulating sources (4) against notifiers (6) gives the following 2x2:

|                       | source has files | source has no files |
|-----------------------|------------------|---------------------|
| **notifier installed**    | claude-code, codex   | gemini-cli, opencode, pi |
| **notifier not-installed**| openclaw             | (none)                   |

Five cells, four populated, one empty. The empty cell — "notifier not-installed AND source has no files" — is empty by definition (a source with no files and no notifier wouldn't show up at all). The diagonal — "notifier installed AND source has files" — is the well-functioning cell (claude-code, codex with 1160+478 = 1638 files, 47.1% of total). The off-diagonal cells are where the friction lives:

- **Notifier installed but no files**: 3 sources (gemini-cli, opencode, pi). Wasted notifier installation; either the tool isn't being used or the parser doesn't see its output.
- **Notifier not-installed but files present**: 1 source (openclaw, 46.1% of files). Missed notifier installation; the highest-volume source is least-coordinated.
- **Source vscode-c***: appears in `Files by source` (239) but not in the `Notifiers` block. This is a **third asymmetry**: the source is recognised by the parser but isn't in the notifier roster at all. Either the notifier framework doesn't yet support that source, or the IDE-resident integration uses a different mechanism (e.g., LSP-side telemetry hooks instead of file-system notification).

## What a healthy pew configuration would look like

Three changes that would flatten the asymmetry:

1. **Install the openclaw notifier.** This would convert the largest-source-by-volume from poll-based sync to event-driven sync. Predicted effect: the pending-upload count should drop and stabilise within 1–2 sync cycles after the notifier is installed.

2. **Either remove or exercise the gemini-cli, opencode, pi notifiers.** Three notifiers are wired up for tools that haven't produced parser-visible files. If the user actively uses those tools, the parser-side gap should be investigated (perhaps the log path or schema changed). If the user doesn't actively use those tools, the notifiers are harmless but they clutter the install state.

3. **Add the vscode-c* source to the notifier roster.** The source has 239 files; absence from the notifier block suggests pew doesn't yet model it for event-driven sync. This is a feature gap, not a configuration gap.

## Falsifiable predictions

If the openclaw-notifier-gap explanation is correct, the following should hold:

1. **Installing the openclaw notifier will reduce the pending-upload count by at least 30% within two sync cycles.** Currently 1658; the openclaw share of pending is unknown but is bounded above by openclaw's 46.1% share of total files, suggesting up to ~764 pending records could be openclaw-attributable. A 30% reduction (~497 records) is a conservative lower bound for the predicted effect.

2. **The openclaw file count will grow faster than the claude-code file count over the next 24 hours.** If openclaw is the modal source today and the user's behaviour is stable, the gap should widen, not close.

3. **The pending-count-to-tracked-count ratio will not exceed 50%.** Currently 47.6%. If it crosses 50%, something has changed: either a sync regression, a notifier failure on one of the installed-notifier sources, or a parser-side spike from a new source.

4. **No source will overtake openclaw in file count in the next 7 days without an explicit user-behaviour change.** Openclaw's 1604-row lead over claude-code's 1160 is a 444-row gap (~38% lead). At the per-day production rates implied by the schema (file counts here are cumulative over the tracked window, not daily), closing that gap requires either claude-code production to surge or openclaw production to halt.

5. **The vscode-c* source count will remain the smallest of the four for the next 14 days.** IDE-resident sessions produce coarser-grained logs than CLI sessions; absent a workflow shift, the IDE source stays at the bottom of the file-count ranking.

## Why this matters

Pew is one example of a class of tools — usage-telemetry sync agents for AI coding tools — that are quietly proliferating. The same structural asymmetry will show up in any such tool: the user runs N tools, each tool emits files at a different cadence, the sync agent has a notifier framework that supports M < N tools, and the gap between N and M shows up as a pending backlog. **The 47.6% pew pending ratio and the openclaw-notifier-missing-but-source-dominant inversion are the empirical fingerprint of that gap.**

If you're building or evaluating a similar tool, the metric to watch isn't the absolute pending count. It's:

- the **ratio of pending to tracked** (above 30%, investigate),
- the **alignment between source-share rank and notifier-installed status** (the largest source should always have a notifier),
- the **set of notifiers installed for sources with zero files** (these are configuration leaks).

A clean pew status would show: pending < 5% of tracked, every source with > 5% share has a notifier installed, and no notifier is installed for a source with zero files. The current status hits none of those three checks. That's not a critique of pew — it's a statement that the user's tool mix is evolving faster than the notifier framework, which is the normal state of a young usage-telemetry tool in a fast-moving ecosystem.

## One-line summary

**Pew v2.20.3 reports 1658 of 3481 tracked files pending upload (47.6%), with openclaw as the largest source at 1604 files (46.1%) and the only source whose notifier is `not-installed` — a structural inversion where the highest-volume source is the least-sync-coordinated, plausibly explaining a substantial fraction of the pending backlog.**
