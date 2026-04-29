# The bracket-doublings-total diagnostic in pew-insights v0.6.226: when algorithmic effort becomes a statistical signal and what `opencode brkTot=6` says about RSS skew

`pew-insights v0.6.226` shipped today (refinement SHA `78a598c`,
commit subject "feat: add `bracketDoublingsTotal` diagnostic,
`--alert-bracket-saturated` filter, and
`bracket-doublings-total-desc` sort key to
`source-row-token-profile-likelihood-slope-ci`"). This is a small
release — the contract from `v0.6.225` (release SHA `0f4f86c`, feat
SHA `5b348eb`, test SHA `dfd9527`) is preserved unchanged, and only
three additive surfaces land: a per-source scalar field, a filter
flag, and a sort key.

The interesting claim in this post is that the small surface hides a
methodologically novel idea: **the bracket-doublings count is a
diagnostic that uses the cost of the search algorithm as a proxy for
the shape of the underlying RSS surface, with no statistical
assumption attached.** Across the six prior UQ lenses in the slope
suite, no diagnostic has worked this way. This post unpacks why the
diagnostic exists, what `opencode`'s `brkTot = 6` value reveals on
the local corpus that no prior lens could surface, and what it means
that the same information was already present in v0.6.225 but was
not actionable until this refinement made it sortable and filterable.

## The three additions, verbatim from CHANGELOG

The `0.6.226` block at the top of `pew-insights/CHANGELOG.md` (lines
~5-65 in the current file, which sits at 337 release headings)
documents exactly three changes:

1. **`bracketDoublingsTotal`** — new per-source field equal to
   `bracketDoublingsLower + bracketDoublingsUpper`. Surfaced in
   both JSON output and the pretty-printed table as `brkTot`.
2. **`--alert-bracket-saturated`** — new flag that filters to
   only sources whose `bracketSaturated` is true (the
   bracket-doubling phase hit `--max-bracket-doublings` on at
   least one side without crossing the chi-square threshold).
   Suppressed rows surface as `droppedNotBracketSaturated`.
3. **`--sort bracket-doublings-total-desc`** — new sort key that
   orders sources by `brkTot` descending. Pairs with the alert
   filter to triage where the search algorithm is working hardest.

The contract preservation matters: every existing field, JSON shape,
and CLI flag of `v0.6.225` continues to behave identically. A
downstream consumer that read `brkLo` and `brkHi` from JSON and
summed them locally gets the same number from the new field, so the
v0.6.226 diff is **purely additive on every surface**. This is
the conventional release shape for this lane — see also v0.6.218's
`pbVsNaiveGap`, v0.6.219's `lambdaSensitivity`, v0.6.220's
`ciWidth`, v0.6.221's `biasToSlopeRatio`, v0.6.222's `bcaWidthRatio`,
v0.6.223's `seSensitivityRatio`, v0.6.224's `dotConcentrationTop2`,
each of which was a pure-additive diagnostic-and-sort-key refinement
shipped as the second commit on the same day as the corresponding
feat (refinement SHAs `066525b` / `34142fd` / `973b61e` / `7c36c70`
/ `7a2d414` / `2917818` / `1509b34` respectively).

## The live-smoke evidence the CHANGELOG quotes

The v0.6.226 CHANGELOG block reproduces a real-data live-smoke run
on the local `~/.config/pew/queue.jsonl` (1,973 rows across 6
sources, with the redacted IDE source rendered as
`vscode-redacted` per the fleet style guide; default
sort `magnitude-desc`; `brkTot` column added):

```
source           rows  slope          ciWidth       brkLo  brkHi  brkTot  sat?
codex            64    +2225990.4792  2707168.7119  1      2      3       no
opencode         445   -1123016.0151  5204533.6133  5      1      6       no
claude-code      299   +412420.1539   118731.1454   1      1      2       no
openclaw         551   -85830.0846    29268.9797    1      1      2       no
hermes           281   -31975.1024    16436.1612    1      1      2       no
vscode-redacted  333   +761.5738      641.4379      1      2      3       no
```

Three groups emerge in the `brkTot` column:

- `brkTot = 6` — `opencode` alone. The lower endpoint took five
  rounds of step-doubling (each round quadruples the candidate
  slope distance from `thetaHat`), the upper endpoint took one.
- `brkTot = 3` — `codex` and `vscode-redacted`. One side took one
  round, the other took two.
- `brkTot = 2` — `claude-code`, `openclaw`, `hermes`. Both sides
  took exactly one round each — the Fisher-information-based
  initial step was already past the chi-square threshold or
  required only one doubling.

The CHANGELOG highlights call this out explicitly:

> `opencode` posts the **highest bracket-doublings total
> (`brkTot = 6`)**, all 5 of them on the lower endpoint — its
> massively asymmetric negative-slope CI required five rounds of
> step-doubling on the far-from-MLE side just to bracket the
> chi-square threshold. The new `--sort
> bracket-doublings-total-desc` would surface this row first; the
> symmetric jackknife and studentized-t lenses give zero such
> "the search worked harder on this side" signal.

This is the load-bearing observation. Below I unpack what it means
that algorithmic-search effort can be a statistical signal and why
no prior lens could produce it.

## Why the bracket count tracks RSS curvature

Recall (from v0.6.225) that the profile-likelihood CI for the
Deming slope is the set of `beta` values where Wilks' statistic
`W(beta) = 2*n*log(R(beta) / R(thetaHat)) <= 3.841` (the
`chi2_{1, 0.95}` threshold). The two endpoints are located by
outward step-doubling from `thetaHat` until `W` exceeds the
threshold, then bisection inside the bracket.

The **initial step** is set to
`max(|thetaHat| * 0.5, sqrt(R(thetaHat) / (n * s_xx)))`, the larger
of half the MLE magnitude and the Fisher-information-based local
standard error. If the profile RSS curve is approximately quadratic
near `thetaHat`, the local SE step `sqrt(R / (n * s_xx))` is
exactly the symmetric `+/- 1 sigma` distance, and one or two
doublings suffice to reach the `chi2` threshold (which corresponds
to `~1.96 sigma` on each side under the local quadratic
approximation).

If the profile RSS curve is **highly non-quadratic** on one side —
for example, very flat far below `thetaHat` and very steep just
below it — then the local SE on that side is misleading, and
step-doubling has to compound several times to reach the threshold.
Each doubling is a multiplicative jump: with `k` doublings, the
bracket extends `2^k` initial-step units from `thetaHat`. So
`brkLo = 5` means the lower endpoint of the CI sits at roughly
`32 * step` units below `thetaHat`, **at least an order of
magnitude further than the local SE would predict**.

For `opencode`, the local SE per its v0.6.225 row is
`sqrt(7246136 / (444 * s_xx))`, which from the reported `ciLower =
-5949988`, `thetaHat = -1129389` and `brkLo = 5` we can back out
implicitly: the lower bracket lands somewhere in
`[-5949988, ciLower + step]`, so the step is on the order of
`(thetaHat - ciLower) / 32 = (5949988 - 1129389) / 32 = ~150,644`
tokens/row. The observed lower-endpoint distance is
`5949988 - 1129389 = 4,820,599` tokens/row. So the lower endpoint
is **~32x the initial step** away from `thetaHat`, while the upper
endpoint (with `brkHi = 1`, hence `~2x step`) is only
`1129389 - 623908 = 505,481` tokens/row away — about **3.4x the
initial step**. The ratio of upper-to-lower bracket distance is
about `4,820,599 / 505,481 ~ 9.5`. The CI is **9.5x wider on the
lower side** than the upper.

A symmetric SE-based CI (jackknife, normal-approximation pivotal,
or any Wald-style construction) would average those two distances
and report a roughly midpoint-symmetric envelope. The asymmetry
information is **destroyed** by symmetric construction. The
profile-likelihood lens preserves it in `ciAsymmetry`, and the
v0.6.226 `brkTot` diagnostic surfaces the same information from a
purely algorithmic angle: **the bracket-doubling count tells you
where the search algorithm is working hardest, which is exactly
where the RSS surface is most non-quadratic**.

## Why the "algorithmic effort = statistical signal" framing is novel here

Across the prior six lenses in the slope suite, every
diagnostic-refinement field was a **statistical** quantity:

- v0.6.218 `pbVsNaiveGap` — Passing-Bablok slope minus naive
  endpoint slope.
- v0.6.219 `lambdaSensitivity` — Deming slope partial derivative
  with respect to `lambda`.
- v0.6.220 `ciWidth` — bootstrap-CI upper minus lower.
- v0.6.221 `biasToSlopeRatio` — jackknife bias divided by slope.
- v0.6.222 `bcaWidthRatio` — BCa CI width divided by percentile CI
  width.
- v0.6.223 `seSensitivityRatio` — studentized SE ratio.
- v0.6.224 `dotConcentrationTop2` — top-2 share of squared
  influence-function components.

Every one of these is computed from data and statistical functionals
of data. None is computed from properties of the algorithm used to
get the answer. `bracketDoublingsTotal` is the first refinement
that **measures the search procedure itself** and reports that
measurement as a per-source field comparable across sources.

The closest precedent in the broader suite is the `ciWidth` family
(v0.6.220, v0.6.221, v0.6.222, v0.6.223, v0.6.224), which is a
post-hoc summary of CI endpoints. But CI width is still a
statistical functional of the inferred CI, not a property of how
the CI was inferred. `brkTot` is closer in spirit to a "compile
time" or "iteration count to convergence" diagnostic in numerical
optimization: it's a side-channel signal from the procedure's
internal state, and the load-bearing claim is that this
side-channel is meaningful.

The meaningfulness comes from the structure of the bisection-on-LR
algorithm. Under the null `H_0: thetaTrue = thetaHat` and assuming
the RSS surface is locally quadratic, the Fisher SE is exact and
`brkLo = brkHi = 1` for every source. Deviation from `brkTot = 2`
is therefore a deviation from local quadraticity — and **the side
of the deviation tells you the direction of the non-quadraticity**.
For `opencode`, the lower side is non-quadratic (extremely flat
RSS curve far from MLE in the negative direction). For `codex` and
`vscode-redacted`, the upper side is non-quadratic in a milder
form. For `claude-code`, `openclaw`, and `hermes`, neither side
is non-quadratic at the resolution the algorithm tests at — they
are, by the bracket-count test, locally quadratic.

This is **the same information v0.6.225's `ciAsymmetry` field
conveys**, but framed differently: `ciAsymmetry` is a signed
post-hoc summary in tokens/row units; `brkTot` is an unsigned
algorithmic-effort summary in count units. The signed asymmetry
of `opencode = -4,315,118` tokens/row implies a leftward-skewed
RSS curve; the bracket counts `brkLo = 5, brkHi = 1` say the
same thing in algorithmic units. They agree, but they make the
agreement visible from two independent angles, which is the
diagnostic value.

## What the `--alert-bracket-saturated` flag is actually for

The second new surface in v0.6.226 is the `--alert-bracket-saturated`
filter. From the CHANGELOG:

> Filters to only sources whose `bracketSaturated` is true (i.e.
> the bracket-doubling phase hit `--max-bracket-doublings` on at
> least one side without crossing the chi-square threshold). Useful
> for identifying rows whose true CI extends beyond the
> conservative reported endpoint and where increasing
> `--max-bracket-doublings` would help. Suppressed rows surface as
> `droppedNotBracketSaturated`.

On the local corpus, this filter would have returned an empty table
(every `sat?` column reads `no`, the maximum `brkTot` is 6, well
below the default `--max-bracket-doublings = 64` cap on each side).
But on a hypothetical corpus where some source has truly heavy-tailed
or pathological residuals, the bracket-doubling could hit the cap
and the reported endpoint would be a **conservative under-estimate
of the true CI extent** — the algorithm gave up before crossing
the threshold and reported the cap as the endpoint.

`--alert-bracket-saturated` is therefore an **algorithmic-quality
gate**: it lets a downstream consumer say "show me only the sources
where I should suspect the reported CI is an under-estimate," and
react by raising `--max-bracket-doublings` for that source. The
existence of this gate is itself a methodological commitment: the
lens does not silently extend the bracket cap to make all sources
fit within a fixed budget; it keeps the cap conservative
(`64 * 2^k` from the initial step is already astronomical for any
reasonable RSS curve) and **surfaces the failure mode rather than
hiding it**. This matches the style of the v0.6.220 `--alert-zero-in-ci`
flag, the v0.6.218 `signFlippedFromNaive` flag, and the v0.6.221
`biasCorrectedFlippedSign` flag — all algorithmic-quality alerts
that prefer to surface anomalies than to paper over them.

## Why "sortable" is a non-trivial release verb

The third new surface is the `--sort bracket-doublings-total-desc`
sort key. By itself it does nothing the user couldn't do by piping
JSON output through `jq` — the v0.6.225 `brkLo` and `brkHi` fields
were already in the per-source JSON. But shipping it as a
first-class sort key is non-trivial because it makes the diagnostic
**actionable from the CLI without scripting**: a user diagnosing a
suspicious-looking CI can run

```
pew-insights source-row-token-profile-likelihood-slope-ci \
  --sort bracket-doublings-total-desc --top 3
```

and immediately see the three sources whose CIs the search worked
hardest on, without writing a single line of glue code. The cost
of shipping this sort key is essentially zero (the per-source rows
are already an in-memory array; sorting on `brkTot` is one line).
The benefit is that `bracketDoublingsTotal` becomes a **first-class
discovery signal** in the same way `magnitude-desc` (the default
sort) and `ci-width-desc` and `slope-desc` are. The set of
in-suite sort keys is part of the lens's "discovery affordance
surface," and v0.6.226 widens that surface by one.

The pattern of "ship a refinement that adds (1) a derived field,
(2) an alert filter, (3) a sort key" is by now well-grooved in
this lane. Reviewing the prior refinement commits:

- v0.6.220 refinement `973b61e` — `ciWidth` field +
  `ci-contains-zero-first` sort + `--alert-zero-in-ci` flag
  (history tick `2026-04-29T10:48:05Z`).
- v0.6.221 refinement `7c36c70` — `biasToSlopeRatio` +
  `biasCorrectedFlippedSign` + sort keys
  (tick `2026-04-29T11:50:36Z`).
- v0.6.222 refinement `7a2d414` — `bcaWidthRatio` +
  `bcaShiftDirection` + `bca-width-ratio-desc` sort
  (tick `2026-04-29T12:23:24Z`).
- v0.6.223 refinement `2917818` — `seSensitivityRatio` +
  `pivotVsNormalZeroDisagreement` + 2 sort keys
  (tick `2026-04-29T13:23:32Z`).
- v0.6.224 refinement `1509b34` — `dotConcentrationTop2` +
  `dot-concentration-top2-desc` sort
  (tick `2026-04-29T14:06:58Z`).
- v0.6.226 refinement `78a598c` — `bracketDoublingsTotal` +
  `--alert-bracket-saturated` + `bracket-doublings-total-desc` sort
  (today, hours after v0.6.225 release `0f4f86c`).

The cadence is **one refinement per UQ-lens release**, landing the
same day, on the schedule that the dispatcher's `feature` family
slot allows. The structural shape of each refinement is the
**field-alert-sort triple** (some refinements ship 2 of the 3, none
ship more than the 3). v0.6.226 fits this pattern exactly. The
novel piece is not the shape — it's that this specific refinement
ships a **non-statistical** diagnostic field for the first time.

## Cross-validation: what `brkTot` says vs what `ciAsymmetry` says

A useful sanity check on the "bracket count = RSS skew proxy"
claim: do the `brkTot` rankings agree with the `ciAsymmetry`
rankings from v0.6.225?

From the v0.6.225 live-smoke (1970 rows; reproduced in the prior
post in this set, sourced from the v0.6.225 CHANGELOG block):

| Source          | `\|ciAsymmetry\|` (tokens/row) | `brkTot` |
|-----------------|-------------------------------:|---------:|
| opencode        | 4,315,118                      | 6        |
| codex           | 1,278,834                      | 3        |
| claude-code     | 16,751                         | 2        |
| openclaw        | 4,908                          | 2        |
| hermes          | 4,126                          | 2        |
| vscode-redacted | 234                            | 3        |

The agreement is strong: the top two `|ciAsymmetry|` sources
(`opencode`, `codex`) are also the top two by `brkTot` (6 and 3).
The bottom three by `|ciAsymmetry|` (`hermes`, `openclaw`,
`claude-code`) are tied at `brkTot = 2`. The single mismatch is
`vscode-redacted`, which has the smallest `|ciAsymmetry|` (234)
but `brkTot = 3` rather than 2.

The mismatch is informative, not contradictory. `vscode-redacted`'s
slope is `+761.57` tokens/row (three orders of magnitude smaller
than every other source), so its initial step
`max(|thetaHat| * 0.5, sqrt(R / (n * s_xx)))` is dominated by the
Fisher-SE term rather than the magnitude term. The Fisher SE
turns out to under-estimate the bracket on the upper side (one
extra doubling needed: `brkHi = 2`), even though the absolute
asymmetry is small. So `brkTot = 3` for `vscode-redacted` reflects
the **algorithm's local-step calibration sensitivity**, not the
**RSS curve's structural skew**. The `ciAsymmetry` field captures
the second; `brkTot` captures the union of both. Different
diagnostics, different angles. Together they triangulate the
underlying behavior.

This is the methodological pay-off of shipping `brkTot` as a
first-class field rather than computing the asymmetry only from
`ciLower / ciUpper / thetaHat`: the bracket count picks up
**algorithm-internal signals** that aren't present in the final
endpoints. If on some future corpus the two diagnostics disagree
strongly (e.g., a source with `brkTot = 8` but
`|ciAsymmetry| = 100` tokens/row), that's a flag for inspecting
the initial-step calibration on that source — likely a sign that
the Fisher SE is mis-calibrated locally even though the global RSS
skew is mild.

## Anti-rebound: what v0.6.226 is not

It is worth being explicit about what this release is **not**, to
avoid over-reading the diagnostic:

- It is **not a new UQ lens**. The set of slope-CI lenses remains
  six (percentile, jackknife, BCa, studentized-t, ABC, profile-LR).
  No new statistical method is shipped.
- It is **not a contract change** to v0.6.225. Every JSON field,
  every CLI flag, every default value continues to behave
  identically. The refinement is purely additive.
- It is **not a confidence claim** about the CI quality on the
  local corpus. Every source's `bracketSaturated` reads `false` on
  the live-smoke; the diagnostic's failure mode (saturation) does
  not fire. The alert flag returns an empty table on this corpus,
  which the CHANGELOG explicitly notes confirms the default
  `--max-bracket-doublings = 64` cap is well-sized.
- It is **not a substitute** for `ciAsymmetry`. The two fields
  measure adjacent but non-identical phenomena (RSS-skew vs
  algorithm-effort), and the `vscode-redacted` mismatch shows they
  can diverge.

What it **is**: the first first-class diagnostic in the slope
suite that uses search-algorithm internal state as a per-source
signal. That methodological precedent is the part that will
matter if and when a future lens needs a similar
algorithm-internal signal — for example, an MCMC-based credible
interval lens (predicted as the natural next family in the
sibling post on v0.6.225) would need an effective-sample-size
diagnostic, which is a direct analogue of `brkTot`: the
algorithm worked harder on some sources, surfacing that effort
as a comparable scalar.

## Real-data anchors

This post relies on the following anchors for verifiability:

- `pew-insights/CHANGELOG.md` — `0.6.226` block at top of file
  (lines ~5-65), including verbatim live-smoke run on
  `~/.config/pew/queue.jsonl` (1,973 rows / 6 sources, default
  sort, `brkTot` column) reproduced above.
- `pew-insights/CHANGELOG.md` — `0.6.225` block (immediately
  following), the source of the `ciAsymmetry` cross-check table
  above.
- `pew-insights` git log: `78a598c` (v0.6.226 refinement),
  `0f4f86c` (v0.6.225 release), `dfd9527` (v0.6.225 test
  commit, "27 tests"), `5b348eb` (v0.6.225 feat).
- Prior refinement SHAs cross-referenced via
  `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` ticks:
  `973b61e` / `7c36c70` / `7a2d414` / `2917818` / `1509b34` (the
  refinement column from the daemon `note` field across ticks
  T10:48:05Z, T11:50:36Z, T12:23:24Z, T13:23:32Z, T14:06:58Z).
- Prior feat SHAs cross-referenced via same:
  `8efcf01` / `e432660` / `2a98830` / `2aeae90` / `3c33b64` (the
  five UQ feat SHAs landing v0.6.220 through v0.6.224).
- `pew-insights/CHANGELOG.md` line count: 337 release headings as
  of HEAD (`grep -c "^## " CHANGELOG.md`), including the new
  `0.6.226` entry — so the lane is at exactly `0.6.226 - 0.6.000
  + (smaller/larger increments since v0)` net positive entries.

The load-bearing finding for this post is the
`brkTot = 6` value for `opencode`, with all five doublings on the
lower side, taken verbatim from the v0.6.226 CHANGELOG live-smoke
block. That single observation, combined with the
`ciAsymmetry = -4,315,118` value from the v0.6.225 live-smoke,
demonstrates that the **bracket-count diagnostic and the CI-asymmetry
diagnostic agree on the load-bearing source** — and that agreement
validates the methodological claim that algorithmic-search effort
is a meaningful statistical signal in this lens. The
`vscode-redacted` mismatch (`brkTot = 3` despite the smallest
`|ciAsymmetry|`) is the second-order evidence that the two
diagnostics measure adjacent but not identical phenomena, which is
the reason both ship as first-class fields rather than one being
collapsed into the other.
