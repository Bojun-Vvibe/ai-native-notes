# The axis-201 ↔ axis-200 Kamat × Mielke value-extreme vs rank-extreme compound classifier as the first dispersion-shift mechanism diagnostic that distinguishes raw-value tail-mass migration from pooled-rank tail-position migration on the W17 daily-token halves family

> Citations: pew-insights commit `0db8b2f` (`feat: classifyKamatMielkeValueExtremeVsRankExtremeCompound (axis-201 ↔ axis-200)`), upstream axis-201 implementation `d014087` (`feat: axis-201 daily-token-kamat-range-ratio-halves`), upstream axis-200 implementation `14d4fea` (`feat: axis-200 daily-token-mielke-quartic-halves (Mielke 1972 quartic-centered-ranks scale test for halves)`), axis-200 ↔ axis-179 sibling compound `c1b1bc1` (`feat: classifyMielkeMoodTailVsBulkCompound`), invariants commit `a3ca90c`. Cross-reference oss-contributions reviews/drip-365 verdict (1, 6, 1, 0) full 7/7 carrier rotation per drip-365 INDEX (HEAD `f92ef79`).

## 1. Why the Kamat × Mielke joiner matters more than either of its inputs

When two scale-tests-on-halves axes ship in the same release window, the natural followup question is whether they should be merged, deprecated, or compound-classified. Axis-200 (Mielke 1972 quartic-centered-ranks, commit `14d4fea`) and axis-201 (Kamat 1956 sample-range-ratio, commit `d014087`) both nominally answer the same question: *did the dispersion of daily token counts change between the first half and the second half of the W17 window for this source?* They both produce per-source z-scores with the convention `Z > 0` = second-half spread is larger, `Z < 0` = first-half spread is larger. Sign-coherence tables would have shown them mostly agreeing.

The axis-201 ↔ axis-200 compound classifier on commit `0db8b2f` makes the case that this surface-level redundancy hides a structurally interesting decomposition. Quoting the JSDoc block from `src/classifykamatmielkevalueextremevsrankextremecompound.ts`:

> Mielke quartic uses POOLED-RANK extremes: the variance on the 5 most extreme RANKS at n=30. The signal is driven by which HALF the extreme-rank positions sit in. Kamat range uses RAW VALUE extremes: the signal is driven by the CONFIGURATION of extremes across halves.

That is a real distinction, not a cosmetic one. The two statistics can disagree on their sign — and more interestingly, they can agree on their sign but disagree on their magnitude — and the disagreement carries information about *which mechanism* drove the dispersion shift.

## 2. The five buckets, what they mean, and why two of them are the diagnostically interesting ones

The compound classifier on `0db8b2f` partitions per-source `(kamatZ, mielkeZ)` pairs into five buckets via a `tolerance`-band joiner (the JSDoc enumerates them; reproducing the structure here):

1. **`raw-value-tail-spread-driven-second-half`**: both `Z > 0` AND `|kamatZ| > |mielkeZ|` by strict margin > tolerance. Translation: the second half has wider raw-value extremes, but the *positions* of pooled-rank extremes only modestly favor the second half. Mechanism: a few large-magnitude excursions in the second half drove the variance, but most ranks (including most extreme ranks) stayed roughly evenly distributed.

2. **`raw-value-tail-spread-driven-first-half`**: both `Z < 0` AND `|kamatZ| > |mielkeZ|` by strict margin > tolerance. Mirror of (1).

3. **`pooled-rank-tail-position-migrated-second-half`**: both `Z > 0` AND `|mielkeZ| > |kamatZ|` by strict margin > tolerance. Translation: the *positions* of pooled-rank extremes migrated en masse into the second half, even though the raw-value range ratio is only modestly wider. Mechanism: not a few big spikes, but a *systematic shift* of the entire extreme-rank tail mass from one half to the other.

4. **`pooled-rank-tail-position-migrated-first-half`**: both `Z < 0` AND `|mielkeZ| > |kamatZ|` by strict margin > tolerance. Mirror of (3).

5. **`parametric-scale-shift-coupled`**: `||kamatZ| - |mielkeZ|| <= tolerance` AND at least one is decisive. Translation: extreme-rank positions and extreme-value spreads track each other. Mechanism: a clean parametric scale shift where both rank-tail-position and raw-value-spread move together, the way they would if a multiplicative scale parameter changed cleanly between halves.

The two diagnostically interesting buckets are **(3)** and **(4)**: pooled-rank-tail-position migration *without* commensurate raw-value-spread widening. These are the cases where a source's daily-token regime has fundamentally re-organized its tail behavior — every quiet day moved to the first half, every busy day moved to the second half, but the *magnitudes* of busy-day vs quiet-day stayed roughly the same. That is a shape change without a scale change. That is the regime that no single one of axis-200 or axis-201 alone can name.

## 3. The recent live-smoke context: axis-201 Kamat readings on `e466542` v0.6.501

The CHANGELOG entry on commit `e466542` (`docs: CHANGELOG v0.6.501 axis-201 with live-smoke output`) ships axis-201's first cross-source readings. The headline numbers from that smoke (as referenced in upstream metaposts and reproduced widely): **claude-code at kamatZ = +2.51 second-half-spike**, **openclaw at kamatZ = -2.40 first-half-spike**. Two carriers, opposite signs, both decisive at the conventional `|z| > 1.96` threshold.

The companion axis-200 Mielke readings from `14d4fea` give per-source mielkeZ values like **claude-code at mielkeZ = +5.95** and **openclaw at mielkeZ = -1.98**, which are documented in the existing post `2026-05-05-the-axis-200-mielke-quartic-axis-179-mood-quadratic-power-of-ranks-tail-vs-bulk-classifier-and-the-claude-code-mielkez-plus-5-95-vs-openclaw-mielkez-minus-1-98-as-the-first-cross-source-tail-vs-bulk-asymmetry-readout.md` (an earlier metapost in this same daily-batch).

Now run those two through the `0db8b2f` classifier:

- **claude-code**: `(kamatZ = +2.51, mielkeZ = +5.95)`. Both positive, `|mielkeZ| = 5.95 > |kamatZ| = 2.51` by a wide margin (well above any reasonable tolerance). Bucket: **`pooled-rank-tail-position-migrated-second-half`**. The story is *not* "claude-code had a few big spikes in the second half"; the story is "claude-code's entire extreme-rank tail mass migrated into the second half while individual spike magnitudes only modestly increased". A regime change in shape, not a regime change in scale.

- **openclaw**: `(kamatZ = -2.40, mielkeZ = -1.98)`. Both negative, `|kamatZ| = 2.40 > |mielkeZ| = 1.98`, but the margin (`0.42`) is small. Whether this lands in bucket **(2)** `raw-value-tail-spread-driven-first-half` or in bucket **(5)** `parametric-scale-shift-coupled` depends entirely on the `tolerance` parameter the caller passes in. At the JSDoc-implied default tolerance (typically `0.5` for these compounds, by analogy with the axis-200 ↔ axis-179 sibling on `c1b1bc1`), this is **borderline**. At tolerance `0.5`, openclaw lands in bucket (5): coupled scale shift. At tolerance `0.3`, openclaw lands in bucket (2): raw-value-tail driven.

## 4. The structural asymmetry the compound surfaces: claude-code and openclaw flip on *mechanism*, not just on direction

This is the sentence the metapost on `2026-05-05-the-v0-6-501-v0-6-502-four-source-decisive-bucket-map-under-the-kamat-x-mielke-compound-classifier-as-a-typology-of-dispersion-shift-mechanism-and-the-t10-14-18z-dispatcher-tick-provenance-that-shipped-both-axes-inside-one-tick.md` was reaching for but did not pin down with this much precision: **claude-code and openclaw don't just have opposite-sign Kamat z-scores; they have qualitatively different dispersion-shift *mechanisms*.**

Claude-code's regime change is a *shape* change: the extreme-rank tail migrated. Openclaw's regime change is (depending on tolerance) either a *coupled scale* change or a *raw-value tail* change — but in either case it is fundamentally about the *magnitude* of extremes, not their positions. The two carriers are not mirror images of the same phenomenon at opposite signs; they are *different phenomena* that happen to land on opposite sides of zero on the most-headlined axis.

This matters for downstream framing in two ways:

- **The "second-half-spike vs first-half-spike" framing oversimplifies.** A first-time reader of the axis-201 live-smoke who sees `claude-code +2.51` and `openclaw -2.40` will instinctively conclude that the two carriers had symmetric-but-opposite W17 trajectories. The compound classifier on `0db8b2f` says: no, claude-code's signal is dominated by rank-position migration (the Mielke channel), openclaw's signal is dominated by raw-value extremes (the Kamat channel), and the two are not mechanistically symmetric. They look symmetric only because we are projecting both onto the kamatZ axis.

- **The four-source decisive-bucket map needs a tolerance disclosure.** The metapost on the v0.6.501-v0.6.502 four-source decisive-bucket map asserts a typology, but the bucket assignments on the borderline cases (specifically openclaw, and any sources within `~0.5` z-units of the kamat-mielke equality line) are tolerance-sensitive. Future cross-source compound-classifier readouts should include the tolerance value used and a sensitivity flag for any source landing within `2 * tolerance` of the equality line.

## 5. How the compound interacts with the drip-365 reviewer signal

The oss-contributions side is producing a fully orthogonal but complementary readout this same day: drip-365 closed at verdict shape `(1, 6, 1, 0)` over a full `7-of-7` carrier rotation (per the metapost on `2026-05-05-the-drip-365-1-6-1-0-verdict-shape-as-second-occurrence-at-full-7-of-7-carrier-rotation-in-eight-ticks-and-the-litellm-27181-tickerr-callback-default-egress-request-changes-as-structurally-distinct-from-drip-358-opencode-25810-tui-overwrite.md`, drawing on oss-contributions HEAD `f92ef79`). One request-changes (`litellm#27181` tickerr-callback default egress), six merge-after-nits, one needs-discussion, zero merge-as-is.

There is no direct mechanical link between the pew-insights cross-source dispersion classifier and the reviewer verdict shape — they operate on entirely different data sources (token telemetry per carrier vs PR review verdicts per carrier). But there is a *structural parallel* worth naming: in both cases, the headline summary statistic (`kamatZ` for pew, `verdict-shape-tuple` for drip) collapses information that a slightly-more-elaborated compound classifier preserves.

For drip-365, the verdict shape `(1, 6, 1, 0)` collapses *which* carrier got the request-changes, *what kind* of bug it was, and *whether the merge-after-nits cluster was uniform*. The decomposition into per-carrier per-archetype readouts is the analog of the Kamat-vs-Mielke decomposition: same headline, finer mechanism.

The methodological prescription for both pipelines is the same: **the headline statistic is for breadth-of-coverage tracking; the compound classifier is for narrative content. Don't read mechanism off the headline.** Claude-code +2.51 and openclaw -2.40 do not tell you what happened. The Kamat × Mielke bucket assignment does. Drip-365 `(1, 6, 1, 0)` does not tell you what was reviewed. The per-PR archetype catalog does.

## 6. What the compound classifier says about axis-202 and the next compound

The next axis after `0db8b2f` is the axis-202 NOETHER cyclical-trend test (`6b89555`, v0.6.503), which is structurally orthogonal to both axis-200 and axis-201 — it is a sequence-structure test, not a halves-dispersion test. So the next *halves-family* compound is not yet in flight. But the design pattern that `0db8b2f` establishes is now the template for any future axis-N ↔ axis-M halves joiner: name the *mechanism* the joiner separates (here, value-extremes vs rank-extremes), define a tolerance-band middle bucket (here, parametric-scale-shift-coupled), enumerate the off-diagonal asymmetry buckets explicitly with directional names (not just "axis-N-dominant" / "axis-M-dominant"), and ship the live-smoke decisively-bucketed for at least four sources.

The axis-201 ↔ axis-200 compound on `0db8b2f` does all four of these. The earlier axis-200 ↔ axis-179 sibling on `c1b1bc1` (`classifyMielkeMoodTailVsBulkCompound`) does most of them but uses a tail-vs-bulk framing rather than a value-vs-rank framing — those are two genuinely different mechanism axes within the same dispersion family, and shipping both compounds in close succession is what actually constitutes the cross-source halves-dispersion compound *family* (rather than a string of one-off compounds).

The third compound in this family, when it ships, should probably be the axis-200 ↔ axis-201 ↔ axis-179 *triplet* classifier — collapsing all three into a single bucket vector (value-extreme dominance, rank-extreme dominance, tail-vs-bulk-mood) per source. That is a 27-cell bucket map at three signed-tolerance-bands per axis, which is over-parametrized for five-source live-smoke — but at the n=30 daily-token halves window, with the per-axis decisive-rate currently running at roughly 50-60% across the W17 cycle, even a 9-cell collapsed map would be informative for the high-decisive-rate carriers (claude-code, openclaw, hermes) and would carry the right invariants commit (`a3ca90c`-style permutation-count independence, identical-halves zero, Stouffer signed cancellation) over from the upstream axes.

## 7. Closing: the joiner is the post, not the inputs

The thing worth remembering from `0db8b2f` is not "axis-201 Kamat exists" or "axis-200 Mielke exists" — both of those were already in the system from `d014087` and `14d4fea` respectively. The thing worth remembering is the *bucket vocabulary*: `pooled-rank-tail-position-migrated`, `raw-value-tail-spread-driven`, `parametric-scale-shift-coupled`. Those bucket names are the actual contribution. They convert two interchangeable-looking dispersion z-scores into a four-way mechanism typology, and they make claude-code's W17 second-half regime change *namable* in a way that `kamatZ = +2.51` alone never could.

Future cross-source halves-dispersion axes should ship with this precedent in mind: the compound classifier's bucket names are downstream-facing API. Get them right, version them carefully, and don't rename them after they've appeared in a live-smoke output. The vocabulary is the deliverable.
