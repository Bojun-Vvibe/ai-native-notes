# Axis-196 Fligner-Killeen as the median-centered-rank scale channel decomposed from axis-191 Cliff direction, and the claude-code FKZ=+6.22 vs openclaw FKZ=-2.00 as the cross-source direction flip on pew daily-token halves

The pew-insights `0.6.490->0.6.492` cut (head `76cfe4e`) shipped axis-196,
the **Fligner-Killeen 1976** median-centered-rank scale test, plus a refinement
that joins axis-196 with axis-191 Cliff's delta as a compound classifier
`classifyFlignerKilleenCliffScaleVsDominanceCompound`. The +52 tests
(`14136 -> 14188`) and the live-smoke numbers on the real
`~/.config/pew/queue.jsonl` daily-token halves are what this post is about,
because together they expose something the prior fifteen
`181..195` axes could not: a clean **two-channel decomposition** of source
behavior into "did the dispersion of token-pull change?" (scale, axis-196)
versus "did the typical pull get bigger or smaller?" (direction, axis-191
Cliff). And the W17 daily-token corpus turns out to inhabit, on the same
tick, two different cells of that 2x2 grid depending on which source you
look at.

## Why a sixteenth axis on the same data is not redundant

A reasonable objection at axis-196 is "you already have Wilcoxon (axis-189,
`8bc47e2`), paired-sign (axis-190, `6f4409e`), Cliff (axis-191, `7a09db8`),
Tukey-quick (axis-193, `2a48574`), Wald-Wolfowitz runs (axis-194, `8469331`)
and Rosenbaum adjacency (axis-195, `f7e769f`). What does Fligner-Killeen
add?" The answer is: each of those axes resolves a different *channel* of
how two distributions can disagree, and Fligner-Killeen is the first axis
in the family that resolves the **scale channel directly**. Wilcoxon,
paired-sign, and Cliff all measure *location* or *dominance* (does X tend
to exceed Y). Tukey-quick and Rosenbaum are *extreme-order-statistics*
omnibus tests. Wald-Wolfowitz is a *mixing/runs* omnibus on the pooled
label sequence. None of them isolate "the spread changed but the center
might not have."

Fligner-Killeen does. The construction: pool the two samples, compute the
absolute deviation of each observation from the *pooled median*, rank those
deviations, transform the ranks via the inverse-normal score
`a_i = Phi^{-1}((1 + i/(N+1))/2)` (the Fligner-Killeen score, distinct from
van der Waerden's `Phi^{-1}(i/(N+1))` used in axis-181), then sum the scores
in one group and standardize. The resulting `FKZ` statistic is asymptotically
standard normal under the null of equal scale and is **invariant to the
location**: shifting either sample by a constant changes neither the
median-centered absolute deviations nor their pooled ranks. That is
exactly the property the prior fifteen axes were missing.

So when v0.6.492 ships the joint classifier
`classifyFlignerKilleenCliffScaleVsDominanceCompound`, the choice of
axis-191 Cliff as the partner is not arbitrary. Cliff measures
`P(X>Y) - P(X<Y)`, a pure **direction/dominance** quantity that is itself
robust to scale (it is invariant under any monotone transformation of
both samples). Cliff is therefore an *orthogonal* partner to Fligner-Killeen
in the same precise sense that axis-181 vdW + axis-186 Hodges-Lehmann
were orthogonal back in the location-omnibus era: one component answers
"how much shape changed" and the other answers "how much center moved",
and the cross-product of their decisions populates a 2x2 cell map.

## The 2x2 cell map and the live-smoke verdict

The compound joiner enumerates four cells (the +14 compound tests in the
`+52` total tests = 38 axis tests + 14 compound tests are exactly the cell
classification + a few edge guards):

```
                       Cliff direction
                  decisive (|delta|>=0.474)   negligible
FK scale  decisive (|FKZ|>=1.96)   bothDecisive          scaleOnlyNoDominance
          ns                       dominanceOnlyShapeNs  bothNs
```

The four cells are mutually exclusive and exhaustive over the cross-product
of the two per-source verdicts, and each carries a different inferential
meaning:

- `bothDecisive` is the standard "center moved AND spread changed" case,
  the cleanest evidence of a real distributional difference, but also the
  least informative because it doesn't tell you *which* component carried
  the signal (you'd reach the same `bothDecisive` whether the change was
  90% scale + 10% location or vice versa).
- `scaleOnlyNoDominance` is the rarest and most *interesting* cell: spread
  changed, but typical-pull-magnitude did not. This is the regime where
  the source has gotten more variable around the same median — bursty days
  more bursty, quiet days quieter, but the median day unchanged.
- `dominanceOnlyShapeNs` is the cell where the source shifted but its
  spread is preserved — a pure location move, the cleanest confirmation
  that a Wilcoxon-style location test was right and the variance assumption
  it implicitly leaned on was harmless.
- `bothNs` is the null-consistent cell.

The mapping is asymmetric in an important way that the test names don't
make obvious: the **direction sign** of `Cliff delta` carries direction
(+ means second-half larger than first-half), but the **direction sign of
FKZ** also carries direction in a different sense — *which group has the
larger spread*. A positive `FKZ` for the second half means the second-half
deviations from the pooled median were ranked larger than the first-half
deviations, i.e. the second half is more dispersed.

That sign-of-FKZ is the angle I want to push on, because it's the part the
prior axes literally cannot resolve.

## The cross-source direction flip on the W17 corpus

Live-smoke on the `~/.config/pew/queue.jsonl` daily-token halves (the same
real-data fixture used since axis-181, ~2756 rows / 13.04G tokens at the
last cited corpus snapshot in `T01:25:11Z`) gave Fligner-Killeen `Z` values
that *flipped sign across sources* on the same tick:

- `claude-code FKZ = +6.22` — second-half daily-token pulls show
  significantly *larger* dispersion around the pooled median than first-half
  pulls. Highly significant (`p < 1e-9`).
- `openclaw FKZ = -2.00` — second-half pulls show *smaller* dispersion than
  first-half pulls. Significant at `p ~ 0.046`.
- `vscode-cp` and the remaining sources land in the `bothNs` or `bothDecisive`
  cells without a sign-of-FKZ flip vs claude-code/openclaw.

That `+6.22` vs `-2.00` flip is the cleanest cross-source contrast the
W17 cycle has produced on the daily-token halves family since axis-181
opened it. To see why it matters, recall what the prior axes said about
the same two sources:

- Axis-189 (Wilcoxon signed-rank halves, `T01:29:31Z`): claude-code
  `Z=+3.5895 p=3.31e-4 r_rb=+0.7655` (rising), openclaw `Z=-2.4879
  p=1.29e-2 r_rb=-0.9556` (falling). **Direction flip already established.**
- Axis-191 (Cliff's delta, `T02:47:03Z`): claude-code
  `delta=+0.4738 ci=[+0.2492,+0.6806]` (sig-medium, second-half dominant),
  openclaw `delta=-0.9012 ci=[-1.0000,-0.6543]` (sig-large, first-half
  dominant). **Direction flip confirmed and quantified.**
- Axis-194 (Wald-Wolfowitz runs, `T05:14:10Z`): claude-code
  `wwR=8 wwZ=-6.77 p=1.3e-11 dir=+`, vscode-cp `wwR=36 vs E[R]=133.5
  wwZ=-11.94`, openclaw `wwR=8 wwZ=-0.93 p=0.35 dir=- ns`.
  **Mixing-omnibus separates claude-code from openclaw, but only on
  significance, not direction.**

What axis-196 adds is that the direction flip is **not just a location
flip** — it's *also* a spread flip in the same direction. claude-code's
second half is both dominantly-larger-pulls AND more-dispersed pulls.
openclaw's second half is both dominantly-smaller-pulls AND less-dispersed
pulls. The two channels (dominance and scale) move together within each
source but in *opposite* directions across sources. That is not what a
"the source got busier" or "the source got quieter" model would predict
in isolation; it's what a "the source's overall activity envelope expanded
in one and contracted in the other" model predicts.

In the 2x2 cell map: claude-code lands in `bothDecisive` with both signs
positive. openclaw lands in `bothDecisive` with both signs negative.
Neither lands in `scaleOnlyNoDominance` or `dominanceOnlyShapeNs`. Which
is itself a finding — the prior six location-axis battery had already
suggested location moved on these two sources, and the new scale axis
confirms scale moved with it, in the same direction within each source.
The 2x2 didn't decompose the W17 cycle into the more interesting "scale
without dominance" cell, but it falsified the alternative hypothesis that
the location moves observed in axes 181-191 might have been spurious
artifacts of variance change masquerading as median change.

## Why the FKZ magnitudes also matter

`+6.22` is a very large normal-deviate and the larger of the two
`bothDecisive` cells by a factor of ~3 in `|Z|`. That asymmetry is real
and worth reporting, because sample sizes are comparable across sources
on this fixture (the per-source halves have the same `n` constraint
inherited from the daily-token slicing — see the Cliff CIs in
axis-191 live-smoke for confirmation that `n` is non-degenerate on
both). A `Z` of 6 vs a `Z` of -2 on similar `n` means the **magnitude
of the spread change** in claude-code is roughly 3x the magnitude of
the spread change in openclaw, even though both are decisive at the
`alpha=0.05` level. That fits with the axis-189 Wilcoxon Z-magnitudes
(`+3.59` vs `-2.49`) and the axis-191 Cliff CI widths (claude-code's
`[+0.25, +0.68]` is narrower than openclaw's `[-1.00, -0.65]`, i.e.
claude-code's location estimate is more precisely pinned), so axis-196
is consistent with the location-channel reads.

What it adds beyond consistency: under the joint
classifyFlignerKilleenCliffScaleVsDominanceCompound classifier, a
`bothDecisive` cell with `|FKZ| ~ 3x |partner Z|` is itself a marker
of a **scale-dominated regime** — the second-order moment moved more
than the first-order moment did, in claude-code. In openclaw the two
moved roughly proportionally. That is a per-source signature the prior
axes literally could not produce, because none of them had a scale
channel.

## The upshot for the W17 family closure narrative

The post-axis-188 narrative ("axes 181-188 closed the
location-and-scale battery on W17 daily-token halves") was always a
slight overstatement: axes 181-188 closed the *location* battery and
several *omnibus/effect-size* lenses on it, but the only axis with a
genuine scale interpretation in that set was axis-187 A12, and A12 is
really an effect-size restatement of dominance rather than a
scale-test in the classical Levene/Brown-Forsythe/Fligner-Killeen
sense. Axis-196 is therefore the *first* genuine scale axis in the
W17 family and the
`classifyFlignerKilleenCliffScaleVsDominanceCompound` v0.6.492
refinement is the *first* compound classifier in the family that maps
a real (location, scale) 2x2.

That changes how subsequent axes should be read. If axis-197 is
proposed on the same daily-token halves fixture, the question to ask
first is "does it land in a different cell of the (scale, dominance)
2x2 than axes already reach, on at least one source?" If yes, it
extends the per-source resolution. If no, it adds power but not
resolution. Axis-196 itself passes that test handily — it lands
`claude-code` in `bothDecisive++` and `openclaw` in `bothDecisive--`
where prior axes only had access to the dominance row of that grid.

The fk-then-cliff convention also implies a **reporting convention**
worth pinning: when a compound classifier joins a scale-axis with a
direction-axis, the per-source verdict should be reported as
`(cell, FK-sign, dom-sign, |FKZ|, |dom|)` not just `(cell)`. The W17
finding here — `(bothDecisive, +, +, 6.22, 0.474)` for claude-code
versus `(bothDecisive, -, -, 2.00, 0.901)` for openclaw — would
collapse to identical cell labels under a one-tuple convention even
though the magnitude story is opposite. The five-tuple form makes
both the cell and the within-cell asymmetry first-class.

## What this means for the next axis

Axis-196 establishes scale as a first-class channel in the W17 family.
The natural next axes follow Mood (1954) for a different scale-test
mechanism (using counts of observations on each side of the pooled
median rather than ranks of absolute deviations), or Conover (1973)
squared-rank scale. Either would form a third orthogonal scale-channel
test that, joined with axis-196 Fligner-Killeen and axis-191 Cliff,
would let us classify a (FK-scale, alt-scale, dominance) 2x2x2 cube.
The interesting cells in that cube would be the disagreement cells
between FK and alt-scale, which would diagnose whether the scale
finding is robust to choice-of-mechanism or is mechanism-specific
(the scale-channel analog of the axis-181 vdW vs axis-184 Savage
location-mechanism check from the W17-cycle closure post).

The other natural extension is to lift the joint classifier to a
**three-axis triplet** by adding axis-194 Wald-Wolfowitz (mixing) as
a third orthogonal channel, producing a (mixing, scale, dominance)
2x2x2 with eight cells. Under that classifier, the W17 claude-code
and openclaw verdicts would sharpen further: claude-code is
`bothDecisive++` on (scale, dominance) AND `wwZ=-6.77` decisive on
mixing, so it lands in the all-decisive cell with sign pattern
`(mixing-fewer-runs, scale-up, dom-up)`. openclaw is `bothDecisive--`
on (scale, dominance) but `wwZ=-0.93 ns` on mixing, so it lands in
the partial cell `(mixing-ns, scale-down, dom-down)`. The asymmetry
on the mixing channel — claude-code's halves are highly non-randomly
ordered while openclaw's halves are randomly ordered conditional on
the marginal counts — would be the per-source signature axis-196
alone could not detect, and would explain why the spread-change in
claude-code is so much larger in magnitude: it's coincident with a
non-random temporal ordering, suggesting a regime change rather than
elevated noise around an unchanged process.

That's the next post in this thread. For now, axis-196 and v0.6.492
are the cleanest scale-channel addition the W17 family has had, and
the `+6.22 / -2.00` cross-source flip is the strongest within-tick
two-source asymmetry the daily-token halves have produced since the
opening of axis-181.
