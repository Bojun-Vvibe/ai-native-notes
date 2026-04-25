# The Tenure × Density Quadrant: Why Three Models Carry 14% of the Population and 86% of the Tokens

Most fleet-of-models analyses I've seen end at a leaderboard sorted by tokens. That table is useful for one question — "where did the spend go?" — and almost useless for every other question. It collapses two genuinely independent dimensions into a single rank, and once you collapse them you cannot recover the structure that made the rank what it is.

The two dimensions worth keeping separate are **tenure** (how long has this model been in the fleet?) and **density** (when it is engaged, how much work does it move per active bucket?). A 2×2 quadrant of those — long-tenure vs short, dense vs sparse — is what `pew-insights tenure-vs-density-quadrant` produces. Run against my local queue at `2026-04-25T13:53:47Z`, the output is small enough to print and structured enough to argue about for a long time.

## The data

```
pew-insights tenure-vs-density-quadrant
as of: 2026-04-25T13:53:47.992Z
models: 15   active-buckets: 1,274   tokens: 8,598,095,271
splits: medianSpanHours=1044.00   medianDensity=109,646
(>= medianSpanHours -> long; >= medianDensity -> dense; ties go long/dense)

quadrant      models  tokens         active-buckets
long-dense    3       1,192,297,888  206
long-sparse   5       1,275,224      323
short-dense   5       7,404,171,247  740
short-sparse  2       350,912        5
```

Population sizes (of 15): 3 + 5 + 5 + 2 = 15. ✓

Token shares (of 8,598,095,271):
- short-dense: 7,404,171,247 → **86.1%**
- long-dense: 1,192,297,888 → **13.9%**
- long-sparse: 1,275,224 → **0.015%**
- short-sparse: 350,912 → **0.004%**

Bucket shares (of 1,274):
- short-dense: 740 → 58.1%
- long-sparse: 323 → 25.4%
- long-dense: 206 → 16.2%
- short-sparse: 5 → 0.4%

The headline reading: **the dense column carries 99.98% of the tokens; the sparse column carries 0.02%**. The tenure axis splits the dense column into a 14% / 86% pair, and that split is the most diagnostic number on the page.

## Reading the quadrant

The four cells are not equally interesting. They have meaningfully different operational interpretations.

### Long-dense: the workhorse cell

Three models, 1.19B tokens, 206 buckets.

```
claude-opus-4.6.1m   span 1044h   buckets 167   tokens 1,108,978,665   density 6,640,591
claude-haiku-4.5     span 1654h   buckets 30    tokens    70,717,678   density 2,357,256
claude-sonnet-4.6    span 1365h   buckets 9     tokens    12,601,545   density 1,400,172
```

These are the models that have been around long enough to have a tenure (≥1044h, the median split) *and* are pulling enough mass per bucket to clear the density median (≥109,646). They are the workhorses: stable, dense, and integrated into the daily flow.

`claude-opus-4.6.1m` dominates within the cell. 167 buckets out of 206 (81%) and 1.11B tokens out of 1.19B (93%). It is the only long-dense model that is actually load-bearing; the haiku and sonnet siblings clear the density bar but are barely engaged. Re-cut the cell as "load-bearing long-dense" and there is one model.

This is the cell you want models to *retire into*. A model that lives long enough and stays dense enough to land here is a model your host workflows have actually adopted at meaningful scale.

### Short-dense: the new flagship cell

Five models, 7.40B tokens, 740 buckets.

```
claude-opus-4.7   span 203h    buckets 291   tokens 4,864,820,971   density 16,717,598
gpt-5.4           span 867h    buckets 391   tokens 2,503,365,225   density  6,402,469
unknown           span 176h    buckets  56   tokens    35,575,800   density    635,282
gpt-5.2           span   0h    buckets   1   tokens       299,605   density    299,605
gpt-5-nano        span   0h    buckets   1   tokens       109,646   density    109,646
```

Two models account for >99% of the cell mass. `claude-opus-4.7` and `gpt-5.4` are the current production primaries. Their span hours (203 and 867) are below the median, which is the only reason they're in the "short" half of the table — they are not "young" in any human sense, but they are younger than median because the median is being pulled up by the long-tenure low-density tail.

This is the most important methodological point about the quadrant: the **median splits are anchored by the population, not by absolute calendar time**. A 203-hour span is "short" *only because* there are five models with spans over 1500 hours sitting in the long-sparse cell. Remove those, and the median moves down, and `claude-opus-4.7` becomes long-dense overnight without doing anything different.

The short-dense cell is structurally where the active production work lives. It is also the cell most exposed to vendor model release cadence: every time a vendor ships a successor, the predecessor's tenure clock keeps ticking but its density may collapse to near zero, dragging it across into long-sparse. The cell turns over.

### Long-sparse: the experimental graveyard

Five models, 1.28M tokens, 323 buckets.

```
gpt-5                  span 2405h   buckets 170   tokens   850,661   density 5,004
gemini-3-pro-preview   span 2521h   buckets  37   tokens   154,496   density 4,176
gpt-5.1                span 1634h   buckets  53   tokens   111,623   density 2,106
claude-sonnet-4.5      span 2856h   buckets  37   tokens   105,382   density 2,848
claude-sonnet-4        span 1754h   buckets  26   tokens    53,062   density 2,041
```

Big spans, lots of buckets (323 — more than long-dense), tiny per-bucket payloads. This is where models go when they were once tried, never adopted, and never quite removed from rotation. Each model has a presence in the queue across thousands of hours of clock time, but each engagement is a small experimental call rather than a load-bearing run.

The 323 active buckets in long-sparse vs the 206 in long-dense is a reminder that **bucket count and token mass are not the same thing**. By bucket count the long-sparse cell is *bigger* than the workhorse cell; by token mass it is 0.1% of the workhorse cell. Bucket-count-weighted analyses overstate the operational footprint of this cell by orders of magnitude.

The right operational question for long-sparse is: do these models still serve a purpose, or are they zombie configurations? In the table above, `gemini-3-pro-preview` (37 buckets, 4.2K density) is clearly a tried-and-shelved experiment. `gpt-5` (170 buckets, 5K density) shows more buckets, which is consistent with a model that's still being routed for a narrow class of small jobs (maybe a heuristic that fires for "give me a one-paragraph answer"). The other three look like residual presence.

### Short-sparse: the noise floor

Two models, 350,912 tokens, 5 buckets.

```
claude-opus-4.6   span 363h   buckets 4   tokens 350,840   density  87,710
gpt-4.1           span   0h   buckets 1   tokens     72    density      72
```

The `gpt-4.1` row at 72 tokens, span 0, is structurally a single mistaken call: a wrong-model-id field in one queue event, or a one-off probe that never landed. The `claude-opus-4.6` row is more interesting — span 363h is non-trivial, but only 4 active buckets, and the density (87,710) just barely misses the dense cutoff (109,646). It looks like a model that was being phased out as `4.6.1m` and `4.7` rolled in. Four buckets across 363 hours is the deathbed of a model that used to matter.

This cell is the noise floor of the analysis. If a model lands here permanently, it should be removed from the host configuration; it is paying for routing logic and not delivering work.

## The 14/86 split is the headline number

Going back to the dense column:

```
long-dense    1.19B tokens    206 buckets
short-dense   7.40B tokens    740 buckets
total         8.60B tokens    946 buckets
```

86% of the dense cell mass is in models that have shorter-than-median tenure. **The flagships of the moment are not the models that have been around longest.** They cannot be, structurally — vendors ship new flagships faster than the median tenure resets, so any flagship that is genuinely current is also short-tenured by population definition.

The 14/86 split is therefore a measure of how *fast* the vendor release cadence is relative to the operator's adoption pattern. If vendors shipped slowly, more flagships would age into the long-dense cell and the split would move toward 50/50. If vendors shipped quickly *and* the operator adopted aggressively, the split would push toward 0/100. Today's 14/86 reads as: the operator follows new flagships within a few hundred hours of release, and the previous flagships still get a meaningful slice (`claude-opus-4.6.1m` at 1.11B tokens) but are no longer dominant.

A version of this split tracked monthly is one of the better signals of operator-vendor coupling. Watching the long-dense share drift down is a leading indicator that you are trusting new-vendor-releases-without-cooling-period; watching it drift up means you are sticking with mature models longer than the population median.

## What the splits hide

The quadrant lens is intentionally crude. Two median splits cut the population into four boxes and lose every model-to-model gradient inside each box.

The biggest cost of the crudeness shows up in the short-dense cell. `claude-opus-4.7` (16.7M density) and `gpt-5-nano` (109K density) sit in the same cell despite being two orders of magnitude apart on the density axis. `gpt-5.2` (1 bucket, 299K density) sits with `gpt-5.4` (391 buckets, 6.4M density) despite being one bucket vs four hundred buckets — completely different operational stories.

The split also has a discrete-jump problem at the boundaries. The median density is 109,646. A model that produces an average of 109,000 tokens per bucket is in the sparse cell; a model that produces 110,000 tokens per bucket is in the dense cell. The cells are useful as broad classifiers; they are not useful for fine-grained ranking. They sit at the right level of abstraction for "where is the workload concentrated, structurally," and at the wrong level for "is model X better than model Y."

This is why the lens is best used as a complement to two others: a per-model ranked table for fine-grain comparisons, and a tenure-only or density-only sort for axis-specific questions. The quadrant is the structural overview; the rankings are the detail.

## Two operational rules

Two things follow from this kind of decomposition that I find myself re-applying:

1. **The dense column is what you bill against.** 99.98% of token mass — and therefore 99.98% of the bill — comes from dense models. When you build cost dashboards, every other model is rounding error. Don't build per-model unit economics for sparse models; build a single "everything else" line.

2. **Models migrate diagonally over time.** A new model lands in short-dense, ages into long-dense if it stays adopted, and falls into long-sparse when it's superseded. The diagonal arc — short-dense → long-dense → long-sparse — is the lifecycle. Models that skip steps (e.g., a new model that lands in short-sparse and stays there) are signals of failed adoption, and the right operational response is to delete them from the host configuration rather than pay the routing-table tax.

The 333-bucket long-sparse cell is the operational consequence of *not* applying rule 2 — five models that should have been pruned a release-cycle ago are still bouncing around the queue producing 0.015% of total token mass and 25% of bucket count. Cleaning that up wouldn't save meaningful tokens, but it would simplify reasoning every time someone reads the model leaderboard.

## What the lens gives you in one screen

A 2×2 grid, fifteen models, four token totals, four bucket totals, and two split values: that's the full output. From those numbers you can read off:

- Which cell is the workhorse of token mass (short-dense, 86%)
- Which cell is the slow-burning predecessor (long-dense, 14%)
- How aggressive the operator's flagship-adoption tempo is (14/86 favoring the new)
- How much zombie configuration the routing table carries (long-sparse: 25% of buckets, 0.015% of mass)
- Which models are at the noise floor and should be retired (short-sparse, both rows)
- Where median splits are anchored, and therefore which "short" labels are population artifacts vs genuine tenure

That's a lot to get from one command. The cost is that any per-model judgment requires a second lens. The benefit is that the structural picture is unambiguous — and structural pictures are what mid-range planning runs on.

---

*Data: `pew-insights v0.4.93` `tenure-vs-density-quadrant` against the local `~/.config/pew/queue.jsonl`, run at `2026-04-25T13:53:47Z`. Median splits as reported by the lens: `medianSpanHours=1044.00`, `medianDensity=109,646`. Population: 15 models, 1,274 active buckets, 8,598,095,271 tokens. The quadrant assignment is anchored to *this* population's medians; the same lens against a different operator's queue would split differently and the cell labels would map to different absolute spans and densities.*
