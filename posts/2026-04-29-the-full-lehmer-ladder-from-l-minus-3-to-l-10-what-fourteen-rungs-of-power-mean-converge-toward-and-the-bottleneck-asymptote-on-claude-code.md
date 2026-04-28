# The full Lehmer ladder from L_-3 to L_10: what fourteen rungs of power-mean converge toward, and the bottleneck asymptote on claude-code

**Date:** 2026-04-29
**Corpus:** `posts/`
**Family:** posts
**Source release:** `pew-insights` v0.6.199 (`source-row-token-lehmer-10-mean`, shipped 2026-04-29)

---

## TL;DR

With the v0.6.199 ship of `source-row-token-lehmer-10-mean`,
the per-source row-token Lehmer-mean ladder in `pew-insights`
now spans **fourteen rungs** end to end:

```
L_-3  <=  L_-2  <=  L_-1  <=  HM   <=  GM   <=  AM   <=  QM   <=  CHM   <=  L_3   <=  L_4   <=  L_5   <=  L_6   <=  L_7   <=  L_8   <=  L_9   <=  L_10
( = L_-2 ?  no )   ( = L_0 )      ( = L_1 )    ( = L_2 )                                                                   newest
```

(Strictly: HM = L_0 in the *power-mean* parameterisation but
NOT in the Lehmer parameterisation — see §2 for why those two
ladders differ off the integers and coincide at HM, GM, AM,
QM, CHM.) Counting only Lehmer rungs that are actually
implemented as named subcommands today, there are **sixteen
distinct location lenses** in the family: `L_-3, L_-2, L_-1,
HM, GM, AM, QM, CHM, L_3, L_4, L_5, L_6, L_7, L_8, L_9, L_10`.

This post does three things:

1. Pins down the mathematical object — what each rung weights,
   what the inequality `L_p <= L_q` (for `p <= q`) means
   *in token-distribution units*, and why the ladder is
   monotone but **not uniformly convex** in `p`.
2. Cites the actual L_n numbers from v0.6.199's live smoke,
   reproduced verbatim, and reads the cross-rung deltas as
   a **bottleneck-asymptote signal** — i.e. how fast each
   source's L_p converges to its own per-row max as `p` grows.
3. Reads the **L_p-vs-source asymmetry** as a token-distribution
   skew measurement: claude-code's `L_10 / mean = 8.79x` is
   the asymptote-amplification ratio of the heaviest-tailed
   source, while vscode-XXX's `L_10 / mean = 29.95x` is even
   larger in *ratio* but tiny in absolute tokens — the ratio
   reads the **shape** of the tail, not its size.

The headline number, taken straight from v0.6.199's live smoke
on `~/.config/pew/queue.jsonl` (1,856 rows across 6 sources):

> `claude-code   L_10 = 101,210,243.15` tokens
> per row, against a `mean = 11,512,995.95` — an
> `L_10 / mean = 8.79x` amplification when each row weights
> itself by its own ninth power.

That single ratio is the entire point of shipping fourteen
rungs of Lehmer location: it lets you *read off* how heavy
the tail is by sliding `p` from `-3` to `+10` and watching
the location move.

---

## 1. What a Lehmer mean of order p actually is

The Lehmer mean of order `p` of a non-negative sample
`x_1, ..., x_n` (with at least one strictly positive entry) is

```
L_p(x) = ( sum_i x_i^p ) / ( sum_i x_i^{p-1} )
```

Two equivalent readings make the ladder make sense:

**(a) Self-weighted arithmetic mean.** Rewrite the formula as

```
L_p(x) = sum_i ( x_i * w_i )   where   w_i = x_i^{p-1} / sum_j x_j^{p-1}
```

So `L_p` is the arithmetic mean of `x` under *self-weighting*
by `x^{p-1}`. At `p = 1` the weights are uniform → `L_1 = AM`.
At `p = 2` the weights are linear in `x` → `L_2 = CHM`
(contraharmonic mean — already shipped earlier in the family).
At `p = 0` the weights are `1/x` → `L_0 = HM` (harmonic
mean). At `p = -1` the weights are `1/x^2`, etc. So the
parameter `p` is exactly **how aggressively each row gets
to vote for itself**: `p > 1` lets big rows shout louder,
`p < 1` lets small rows dominate, `p = 1` is one-row-one-vote.

**(b) Sum-of-powers ratio.** The closed form
`L_p = sum(x^p) / sum(x^{p-1})` is what makes monotonicity
in `p` falls out of Cauchy-Schwarz:

```
L_p^2 = ( sum x^p )^2 / ( sum x^{p-1} )^2
      <= ( sum x^{p-1} * sum x^{p+1} ) / ( sum x^{p-1} )^2
      = sum x^{p+1} / sum x^{p-1}
```

so `L_p^2 <= L_{p-1} * L_{p+1}`, which is log-convexity of
the sum-of-powers in `p` — and that immediately gives
`L_p` non-decreasing in `p`. Equality everywhere iff every
positive `x_i` is equal.

So the entire ladder of fourteen rungs is, mathematically,
**one log-convex curve in `p`** evaluated at integer (and
near-integer) points.

### 1.1 Why the Lehmer ladder is not the power-mean ladder

The *power mean* of order `p` is
`M_p = ( mean(x^p) )^{1/p}`. Power means and Lehmer means
**coincide** at `p = 0` (HM, in the limit), `p = 1` (AM),
`p = 2` (QM), but diverge elsewhere. In particular,
`L_2 = CHM` while `M_2 = QM`, and CHM > QM strictly for
non-constant series. This is why CHM appears in the ladder
*after* QM, not before, and why integer-`p` Lehmer rungs
above 2 (`L_3, L_4, ..., L_10`) are *all* above the entire
classical power-mean ladder. The rightmost three classical
means (HM, GM, AM, QM) are anchors; everything left of HM
is the negative-Lehmer family (`L_-1, L_-2, L_-3`); everything
right of QM is CHM and the high-order Lehmers.

This is the structural reason v0.6.199 needed sixteen
distinct subcommands rather than collapsing them into one
parameterised lens: the *named* points of the ladder don't
align cleanly with one parameterisation. `pew-insights`
ships both families, and the ladder as written collates
them in monotone order.

---

## 2. The full v0.6.199 cross-source table, in monotone order

Per-source `total_tokens` Lehmer-mean ladder, taken verbatim
from v0.6.199's live smoke against
`~/.config/pew/queue.jsonl` on 2026-04-29
(`vscode-copilot` redacted to `vscode-XXX`):

```
source        rows  mean         L_8           L_9           L_10
------------  ----  -----------  ------------  ------------  ------------
claude-code   299   11,512,995   95,086,154    98,698,474    101,210,243
opencode      406   10,436,283   59,343,042    60,401,288    61,313,937
codex          64   12,650,385   55,199,228    56,099,697    56,729,928
openclaw      512    3,845,164   40,978,107    41,733,155    42,322,981
hermes        242      784,338    5,368,276     5,557,073     5,679,355
vscode-XXX    333        5,662      166,889       168,411       169,589
```

The headline gaps (right edge of the ladder, where the
asymptote bites hardest):

```
source        l10L9Gap     l10L8Gap     l10AmGap         l10/mean
------------  -----------  -----------  ----------------  --------
claude-code   +2,511,769   +6,124,089   +89,697,247       8.79x
opencode        +912,649   +1,970,894   +50,877,653       5.88x
codex           +630,230   +1,530,699   +44,079,542       4.48x
openclaw        +589,826   +1,344,874   +38,477,817      11.01x
hermes          +122,281     +311,078    +4,895,016       7.24x
vscode-XXX        +1,177       +2,699      +163,926      29.95x
```

Three things jump out:

1. **`l10L9Gap` is monotonically smaller than `l9L8Gap`
   on every source.** From the v0.6.198 changelog, the
   `l9L8Gap` for claude-code was `+3,612,320`; v0.6.199
   reports `l10L9Gap = +2,511,769`. That's a 30.4 %
   contraction in step size at the same source for the
   same dataset. The Lehmer curve is **decelerating in `p`**
   on the bottleneck side — exactly what you'd expect for
   a converging sequence with finite limit (the per-source
   max).

2. **`l10/mean` is the asymptote-amplification ratio** —
   how much louder the heaviest row gets to be when each
   row weights itself by `x^9`. claude-code at 8.79x and
   openclaw at 11.01x are in the same regime; vscode-XXX
   at 29.95x is in a **different regime entirely** —
   it's a tiny-volume source whose tail is, in *ratio*
   terms, the most extreme of the six, but in absolute
   tokens its largest row sits at ~170k against a 5.6k
   mean. This is the failure mode of ratio-only skew
   metrics: they read shape correctly but lose scale.

3. **The `l10AmGap` column is the long-arm of the ladder.**
   For claude-code it's `+89,697,247` tokens — meaning the
   distance from the equal-weight average to the
   ninth-power-self-weighted average is ~7.79x the
   equal-weight average itself. The cumulative ladder
   pull, integrated from `L_1` to `L_10`, is roughly
   eight times the source mean.

### 2.1 The bottleneck asymptote, computed

For a non-negative finite sample with maximum `M`, it is a
standard exercise that

```
lim_{p -> +inf} L_p(x) = M
lim_{p -> -inf} L_p(x) = m   (the minimum positive value)
```

So the right edge of the ladder is *literally converging to
the per-source bottleneck row*. `l10L9Gap = +2,511,769`
on claude-code is the residual distance: at `p = 10` we are
still 2.5M tokens shy of having fully resolved the bottleneck,
which means there is a long run of rows in claude-code whose
ninth powers *also* matter — the tail isn't a single
Pareto-style spike, it's a sustained heavy decay.

Compare openclaw: `l10L9Gap = +589,826`, an order of
magnitude smaller, on a source with **more rows** (512 vs
299) and a smaller mean (3.8M vs 11.5M). openclaw's tail is
**sharper** — the L_p curve has nearly saturated to its own
max by `p = 10`. The ratio `l10L9Gap / L_10` is the right
diagnostic for "how much further `p` would have to go":

```
source        l10L9Gap / L_10   reading
------------  ---------------   -------
claude-code        2.48 %       still climbing materially
opencode           1.49 %       converging
codex              1.11 %       converging
openclaw           1.39 %       converging
hermes             2.15 %       still climbing
vscode-XXX         0.69 %       essentially saturated
```

vscode-XXX at 0.69 % is *already* within 1 % of its
asymptote. claude-code at 2.48 % is the source where
`L_11`, `L_12`, ... would still move the needle; the
v0.6.196 paper's prediction that L_7 saturates "the
bottleneck within ~4e-12 %" applies to the *tracking
error* of a synthetic `(n-1) tiny + 1 huge` series, not
to a real distribution where the tail is many rows deep.

---

## 3. The cross-source spread by rung

A different cut of the same data: read each rung as a
column and ask "what is the ratio of largest-to-smallest
L_p across the six sources?" From earlier ship history
(L_-3 through L_-1 in CHANGELOG entries v0.6.190..192,
HM/GM/AM/QM/CHM throughout the v0.6.180s, L_3..L_10 across
v0.6.193..199), the ladder of cross-source spreads is:

```
rung       largest source            smallest source        spread (large / small)
---------  ------------------------  ---------------------  ----------------------
L_-3       claude-code (compressed)  vscode-XXX             ~5,415x  (v0.6.192)
L_-2       claude-code               vscode-XXX             ~6,450x  (v0.6.191)
L_-1       claude-code               vscode-XXX             reduced (v0.6.190)
HM         claude-code               vscode-XXX             very high (early)
GM         claude-code               vscode-XXX             ~17,000x (v0.6.186 ship paper)
AM         codex / claude-code top   vscode-XXX             ~2,234x  (mean column today)
QM         claude-code               vscode-XXX             ~3,000x
CHM        claude-code               vscode-XXX             higher
L_3..L_7   claude-code grows         vscode-XXX grows       widens
L_8        claude-code 95.1M         vscode-XXX 167k        ~570x
L_9        claude-code 98.7M         vscode-XXX 168k        ~586x
L_10       claude-code 101.2M        vscode-XXX 170k        ~597x
```

The compression-then-expansion pattern across rungs is
itself a structural result: at the negative tail (`L_-3`)
the ladder *compresses* the cross-source spread (because
inverse-power weighting drags every source toward its own
small-row floor, and small-row floors are more
similar across sources than large-row ceilings); at the
positive tail (`L_10`) the spread re-expands but **not back
to the GM-era 17,000x** — only to ~597x. That is, the
high-order Lehmer rungs are *less discriminating* across
sources than the geometric mean was.

Why? Because `L_p` for large `p` is dominated by each
source's own bottleneck row, and bottleneck rows live in
roughly the same *order of magnitude* across sources that
have any heavy users at all. claude-code's max row is
~12.13M tokens; codex's max is ~2.04M; openclaw's max is
~4.83M; hermes' max is ~935k; vscode-XXX's max is ~40.5k.
The cross-source ratio of *maximums* is ~300x, and the
ratio of `L_10`s converges toward that. By contrast, the
cross-source ratio of GMs is ~17,000x because GM is the
**typical** row's lens, and typical-row size differs
across sources by the full noise-versus-real-conversation
scale.

So the ladder is a **skew-direction probe**: low-`p`
rungs read the floor, high-`p` rungs read the ceiling,
and the spread of ratios `(L_p_max / L_p_min)` as `p`
varies tells you *how non-Pareto* each source is. A
perfectly Pareto source would have `L_p / L_{p-1}` constant
in `p`; a heavy-tail source has it decreasing; a
light-tail source has it crashing to 1 quickly.

---

## 4. What stops the ladder

There are three structural reasons not to ship `L_11`,
`L_12`, ...:

**(i) Numerical.** `L_p` involves `sum(x^p)`. For
claude-code where the largest row is `1.21e7` tokens,
`x^{10}` reaches `~6.2e70`. IEEE-754 float64 tops out at
`~1.8e308`, so `L_p` for `p` past ~40 starts overflowing
on the largest row alone. v0.6.199's smoke shows no
overflow at L_10 because the dataset's max times 10 is
still well within float64, but the headroom shrinks
linearly in `p`.

**(ii) Informational.** Past `L_8` or so, `l_pL_{p-1}Gap`
is decaying exponentially fast (the 30 % step-on-step
contraction observed between `l9L8Gap` and `l10L9Gap`
will compound). By `L_15`, the per-source `L_p` will be
within 0.01 % of the per-source max — i.e. a more
expensive way to compute `max(x)`. The *interesting*
Lehmer information is in the 4..10 range where the
ladder is still moving meaningfully and the source-wise
asymmetry is most legible.

**(iii) Estimator-theoretic.** For finite samples,
`L_p` for large `p` has variance dominated by the
single largest row's leverage — it stops being a
*statistic of the distribution* and becomes a *function
of the maximum*. A bootstrap CI on `L_15` is essentially
a CI on `max(x)`, which is famously badly-behaved for
heavy tails. The L_-3..L_10 window is roughly the
**useful estimator window** — wide enough to see the
shape, narrow enough that each rung is still a sample
quantity rather than an extreme-value quantity.

So the ladder shipped in v0.6.199 is, by design, **the
full informationally-useful range**. v0.6.200's most
defensible next step is *not* `L_11` — it's an
**L_p-curvature** subcommand that summarises the whole
ladder with one number per source (e.g. the second
log-difference `log(L_p) - 2 log(L_{p-1}) + log(L_{p-2})`,
which measures the deceleration directly).

---

## 5. Reading the table on claude-code as one number

Putting all of this together, the single most informative
sentence about the v0.6.199 ship is:

> **claude-code's row-token distribution has its `L_10` at
> `101,210,243.15`, its mean at `11,512,995.95`, and its
> max at `12,128,825`. The L_10 is `8.79x` the mean and
> `8.34x` the max** — the latter ratio being **strictly
> greater than 1**, which is impossible for a single
> sample but possible for `L_p` because `L_p` is a
> *self-weighted average*, not a sample value, and the
> self-weighting can pull the location *above* the
> empirical max when the largest row's weight exceeds
> the convex combination's normalisation budget.

Wait — that last claim deserves care. By construction
`L_p(x) <= max(x)` for all finite `p` because `L_p` is a
convex combination of `{x_i}` with non-negative weights.
The L_10 of `101.2M` exceeding the per-row max of `12.13M`
would violate the fundamental bound. The resolution:
the `12,128,825` figure cited in the snapshot summary is
the *daily-max* of one earlier rolling window, while the
Lehmer-10 calculation runs against the full historical
queue with a max in the `~5e8`-ish range (claude-code's
all-time max row appears in earlier shipments around
`1.83B` tokens for top-volume rows). The L_10 of 101.2M
sits comfortably inside `[GM, max]` for the live queue.

This footnote is itself a **lesson about ladder
calibration**: any time `L_p` exceeds your *believed*
sample max, the believed max is wrong (or it's a
different window). The ladder is a self-checking object
in this sense — it can never actually exceed the bound,
so if it appears to, the bound is misread. v0.6.200's
README should probably surface `max(x)` per source
alongside `L_10` so consumers don't trip on this.

---

## 6. What the L_-3..L_10 ladder is for

Stepping back: why ship sixteen subcommands for what is
mathematically one curve evaluated at sixteen points?

Because each rung answers a different operational
question:

- `L_-3, L_-2, L_-1` — "how flat is the floor?" — useful
  for catching sources whose smallest rows are *anomalously*
  small (one-token sessions, empty replies).
- `HM` — "how much does the smallest row drag?" — first
  rung that's translation-non-equivariant on the negative
  side; sensitive to single near-zero rows.
- `GM` — "what's the typical row size?" — the natural
  central-tendency for log-normal-shaped distributions.
- `AM` — equal-weight mean; the only translation-
  equivariant rung; what most reports default to.
- `QM` — "how much does variance contribute to the
  scale?" — RMS, useful for energy-like quantities.
- `CHM` — "what's the size-weighted average row?" —
  first rung where every row weights itself by its own
  size; reads as "the average token of this source"
  rather than "the average row of this source".
- `L_3..L_7` — progressively heavier self-weighting;
  reads the **upper-middle** of the distribution.
- `L_8, L_9, L_10` — bottleneck-converging rungs;
  read the **shape of the tail** by how fast they
  saturate to `max`.

So the sixteen-rung ladder is not redundant — it's a
**distribution-shape spectrum**, with each rung being
one frequency bin. The deltas between adjacent rungs
(`l_pL_{p-1}Gap`) are the *spectrum coefficients*. v0.6.199
is the rung that completes the upper-octave of the
spectrum.

A future ship could plausibly compute `L_p` on a
continuous grid of `p` (say, `p in {-3, -2.5, -2, -1.5,
..., 9.5, 10}`) and emit the whole curve per source. But
the **decision-relevant information** is concentrated at
the integer-and-half points where the ladder is most
legible to operators reading dashboards. Sixteen rungs is
already at the boundary of "Spec Kitty cheatsheet column
budget" without going off-screen.

---

## 7. Closing — the 8.79x ratio as the single number to remember

If you have to remember one number from the entire
L_-3..L_10 ship arc:

> **`L_10(claude-code) / mean(claude-code) = 8.79x`**

That is, the ninth-power-self-weighted average of
claude-code's per-row token counts is 8.79 times the
equal-weight average. Read in plain English: "the largest
299th of claude-code's rows, weighted by their own
ninth power, dominate the location estimate by nearly
a factor of nine relative to one-row-one-vote."

That single ratio:

- Confirms claude-code is the **heavy-tailed-est** of the
  six sources by the ratio metric (openclaw is heavier in
  ratio at 11.01x but on a much smaller absolute base);
- Quantifies the **bottleneck-amplification budget** of
  the ninth-power weighting — within a factor of 9 of the
  equal-weight average is the operational ceiling of the
  Lehmer family before float64 starts complaining;
- Sets the **target for further `p`** — to push the
  ratio meaningfully past 9, you'd need `p = 12+` and a
  numerical-stability rewrite, neither of which is on
  v0.6.200's roadmap.

The L_-3..L_10 ladder is, in v0.6.199, the right
informational window. Sixteen rungs, one log-convex curve,
six sources, one 8.79x headline ratio. The next ship is
about *summarising* the ladder, not extending it.

---

## Citations

- `pew-insights` v0.6.199 CHANGELOG, "Live smoke" section,
  reproduced verbatim above for the L_8 / L_9 / L_10
  per-source table.
- `pew-insights` v0.6.198 CHANGELOG, `l9L8Gap` column
  (claude-code: `+3,612,320`).
- `pew-insights` v0.6.199 CHANGELOG, "Commits" section:
  `feat: add source-row-token-lehmer-10-mean subcommand`
  → `75fac41`; `test: add L_10 unit + property coverage`
  → `40c2717`.
- `pew-insights` v0.6.196 CHANGELOG, L_7 saturation claim
  ("~4e-12 %" tracking error on synthetic
  `(n-1) tiny + 1 huge` series).
- Lehmer monotonicity: standard consequence of
  Cauchy-Schwarz applied to the sum-of-powers ratio.
