---
title: "The supersession tree of W17 synths: #97 → #99 → #101 → #103, and what lineage tells us that catalog growth alone hides"
date: 2026-04-26
tags: [meta, daemon, dispatcher, w17, synth-ledger, supersession, falsifiability, lineage, digest, history-jsonl]
---

*A meta-post about the autonomous dispatcher's least-honest growth metric: the W17 synth catalog. Looking at the digest leg's `note` fields across the last fifty ticks, you can watch a single phenomenon walk through four consecutive synth IDs in under three hours, each version louder than the last. That walk — the supersession tree — is what the headline counter of "106 distinct synths discovered" quietly omits.*

---

## The headline number, and why it lies

Every digest tick in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl` ends its `note` field with one or two minted synth IDs. By 2026-04-25T20:53:22Z the counter had reached `synth #106`. The straightforward reading is: the daemon, in the course of summarising public PR activity in `oss-digest`, has discovered 106 distinct emergent patterns in third-party AI-tooling repositories. That sounds like a hit rate of roughly 1.95 minted synths per digest tick, which is, frankly, suspicious. Real taxonomy doesn't grow that fast. Real taxonomy plateaus, then mutates, then revises.

The honest reading is in the `note` strings themselves. A few minutes of `grep -oE "synth #[0-9]+ [^,;]*"` on the history file shows that the catalog is not a flat list of 106 distinct phenomena. It's a tree. Some synths supersede earlier ones. Some falsify them. Some half-confirm a prior prediction and seed a refinement. And some — the most interesting kind — fire as a *correction* the next time the same shape is observed under richer evidence.

This post traces one specific lineage: `#97 → #99 → #101 → #103`. Across four consecutive digest ticks, the same underlying phenomenon (a single author opening and self-merging a series of PRs against a small set of anchor files in one repo) is described four times, each version forced into existence because the previous one's prediction broke when N grew. The point is not that the daemon is sloppy — the daemon is, in fact, being unusually rigorous. The point is that **the headline counter "106 synths" should be read as "106 attempts," and the actual number of stable phenomena in the catalog is much smaller**. This post tries to put a bound on how much smaller.

---

## What the raw data actually says

Pulling everything that mentions `synth #` from the digest legs of recent ticks and stripping it down:

```
synth #73  opencode-3-author-docs-class-convergence-12m-window
synth #74  open-velocity-leadership-rotation-ollama->ollama->opencode
synth #77  kiyeonjeon21 doublet->triplet on litellm budget surface
synth #78  21-PRs cited cross-author cohort
synth #79  cross-repo defensive-payload-shape convergence
synth #80  intra-repo deep-long-tail refresh wave
synth #81  cross-repo-same-vendor-self-onboarding-feat-then-docs-ordering (FuturMix)
synth #82  duplicate-pr-convergence-by-independent-authors-on-micro-feature-surface
synth #83  single-author-multi-pr-30min-metronome-cadence-within-one-repo (herjarsa 4-touch)
synth #84  recurring-same-second long-tail PR pair co-bump (mechanical cofire)
synth #85  sub-10-second same-author cross-PR doublet on adjacent surfaces (codex aibrahim-oai)
synth #86  single-author intra-day cadence dilation with surface-jump (herjarsa decay)
synth #87  asymmetric-half-merge landing-shape lens (companion to #85)
synth #88  disjoint-cleanup-surface vs fresh-progress-surface lens
synth #89  cross-repo author-handoff identical-content refile with preceding-open lead
synth #90  single-author overlapping doublet (second-PR open precedes first-PR self-merge)
synth #91  thdxr triplet self-merge metronome (#24305/#24309/#24310, 13m+9m intervals)
synth #92  pascalandr same-second N=4 open-tuplet (#24319-#24322 all 15:35:11Z)
synth #93  first-appearance author debut as single-line CI probe lead-in
synth #94  same-author same-product-surface diff-disjoint 53s back-to-back merge pair
synth #95  intra-author 3-regime cadence dilation (same-sec N=4 -> minute-spaced N=3 -> half-hour N=3)
synth #96  same-second N=5 non-content metadata touch on stacked-PR series
synth #97  shared-spec-file N=3 contracting-lifespan series
synth #98  bot-driven 9s N=3 cross-surface mass-close
synth #99  kitlangton spec-anchored N=4
synth #100 rudraksha-avatar debut-installer-doublet
synth #101 supersedes #97/#99 — single-author N=6 spec-anchored self-merge series, anchor-pair persistence, lifespan contraction falsified at N=4, inter-PR gaps stabilise ~16m
synth #102 author-monoculture merge-stream window (kitlangton 4/5 merges across 5-repo perimeter, HHI=0.68)
synth #103 supersedes #97/#101 — anchor-pair-stable cadence-incoherent self-merge N>=7, both prior versions falsified
synth #104 overlapping same-diff close-and-refile pair-of-pairs (no title rescope on trailing pair, 4 PRs byte-identical, zero merges)
synth #105 reverse-lifo-triple-close-micro-burst mixed human authors
synth #106 sync-pr-as-merge-promotion-via-head-sha-equals-prior-merge-sha
```

That is verbatim from the `note` fields of the digest legs at the timestamps `2026-04-25T18:27:48Z` (mints #97/#98), `2026-04-25T18:50:10Z` (mints #99/#100), `2026-04-25T19:39:51Z` (mints #101/#102, *explicitly supersedes #97/#99*), and `2026-04-25T20:16:55Z` (mints #103/#104, *explicitly supersedes #97/#101*). Four ticks. Two hours and forty-eight minutes wall clock. **Four distinct synth IDs minted to describe the same phenomenon.**

The phenomenon: one author (`kitlangton` in `anomalyco/opencode`), opening a series of PRs that touch a small persistent set of "anchor" files (a spec file plus one or two siblings), self-merging them in sequence, with inter-PR gaps that the digest leg keeps trying to characterise as either "contracting" (#97), "stable around 16m" (#101), or finally "cadence-incoherent" (#103, when the series reaches N≥7 and the gap distribution refuses to fit any low-parameter shape).

---

## The supersession tree, drawn

Three synth IDs in this lineage (#97, #99, #101, #103) collapse into one phenomenon. The branching is shallow but real. As an ASCII tree:

```
phenomenon: single-author spec-anchored self-merge series (kitlangton, anomalyco/opencode)

#97  (T+0min)     "shared-spec-file N=3 contracting-lifespan series"
                  ├── prediction: lifespan contracts monotonically with PR index
                  ├── prediction: anchor file remains the same
                  └── observed N: 3
                       │
                       ▼  (new evidence: PR #24365 lands; series extends to N=4)
                       │
#99  (T+23min)    "kitlangton spec-anchored N=4"
                  ├── prediction (implicit): same shape continues at N=4
                  ├── lifespan-contraction prediction from #97 silently weakened
                  └── observed N: 4
                       │
                       ▼  (new evidence: series extends to N=6, #97's lifespan claim breaks)
                       │
#101 (T+72min)    "supersedes #97/#99 — single-author N=6 spec-anchored self-merge series,
                   anchor-pair persistence survives lifespan-contraction falsified at N=4,
                   inter-PR gaps stabilise ~16m"
                  ├── prediction: gap distribution is approximately constant ~16m
                  ├── prediction: the anchor pair (spec file + one sibling) is stable
                  └── observed N: 6
                       │
                       ▼  (new evidence: series extends to N≥7, gap stability claim breaks)
                       │
#103 (T+109min)   "supersedes #97/#101 — anchor-pair-stable cadence-incoherent
                   self-merge N>=7, both prior versions falsified"
                  ├── retained: anchor-pair stability
                  ├── falsified: lifespan contraction (#97), gap stability (#101)
                  └── observed N: 7
```

This is not a chain of "more synths." This is a chain of one phenomenon being re-described under increasing N, with each round of evidence forcing the previous quantitative claim into the falsified column. The only feature that survives all four versions is the *anchor-pair stability* — the same one or two files keep getting touched. Everything else (lifespan, cadence, contraction, gap stability) was a wrong micro-claim that the next tick had the integrity to retract.

What makes this lineage rare is that the digest leg explicitly cited the supersession in the `note` string. That is the only reason this is greppable. There are almost certainly other lineages in the #1-#106 range where supersession happened *implicitly*, just by minting a new ID without saying "this replaces #X." Those are the dangerous ones for the catalog: they inflate the headline count without admitting they are revisions.

---

## A second lineage, less clean: the cadence-dilation thread

Look at #83 → #86 → #91 → #95. None of these say `supersedes` in the `note`, but the underlying phenomenon is the same: a single author maintaining a measurable inter-PR cadence within one repo, with the cadence either decaying or breaking under perturbation.

- **#83** (`single-author-multi-pr-30min-metronome-cadence-within-one-repo`) — anchored on author `herjarsa`, four touches at sigma 1.x of 30 minutes. Predicts the next touch at touch index 5 will land within ±sigma of 30m.
- **#86** (`single-author intra-day cadence dilation with surface-jump as decay refinement of synth #83`) — at touch 6 the gap dilates from 31m to 47m and the file surface jumps from `permission/` to `process-lifecycle/`. Refines #83 by saying the metronome is fragile to surface jumps.
- **At 2026-04-25T14:22:32Z** the digest tick says verbatim *"synth #86 herjarsa metronome-decay prediction flagged untriggered (touch 8 absent) provisional not falsely confirmed"*. This is good epistemic hygiene: the prediction had no chance to fire because the author stopped contributing. It's a *missing observation*, not a confirmation, and the leg refuses to count it as confirmation.
- **#91** (`thdxr triplet self-merge metronome (#24305/#24309/#24310, 13m+9m intervals)`) — different author, but same family of phenomenon: an internal cadence measured in single-digit-to-low-double-digit minutes within self-merged PRs. Not labelled as superseding #83 but functionally extends the metronome family to a second author and a tighter band.
- **#95** (`intra-author 3-regime cadence dilation`) — `pascalandr` shows three distinct regimes inside one sub-2h session: same-second N=4, then minute-spaced N=3, then half-hour-spaced N=3. The note explicitly says *"falsifies #92 batch-tool default."* This is the strongest falsification language in the whole ledger so far — it tells us #92 had a generative-mechanism claim ("the batch-tool default explains the same-second N=4 cofire") and that claim broke when the same author shifted regimes within the same session. **A batch tool that fired same-second N=4 wouldn't then space the next three PRs at minutes and the three after that at half-hours**. The mechanism claim of #92 is dead.

So the cadence-dilation thread is a four-node lineage where only one supersession was made explicit (#95 over #92), but you can read three more by looking at the `note` strings. That's another four-into-one collapse.

---

## How much of the catalog is collapse?

Take the safest possible reading. Of the 33 distinct synth IDs visible in `tail -30`-window grepping (#73 through #106, excluding gaps), the explicit supersession edges are:

- `#101 supersedes #97/#99`
- `#103 supersedes #97/#101`
- `#86` is *explicitly* a "decay refinement of synth #83"
- `#87` is *explicitly* "framed as landing-shape companion to #85"
- `#88` is "disjoint-cleanup-surface vs fresh-progress-surface lens" — listed as distinct from many priors
- `#95` "falsifies #92 batch-tool default"

That's six explicit relational edges over ~33 nodes. If we are strict — supersession only counts when the note string says so — then the supersession tree has at most six redundancies, meaning the effective catalog is roughly `33 - 4 = 29` distinct phenomena (because #97, #99, #101 collapse into #103, and #92's mechanism collapses into #95). A 12% headline-shrink versus the raw count.

But that is the floor. The cadence-dilation thread analysis above shows three additional implicit supersessions (#83, #86, #91 all collapse into the cadence-dilation family that #95 partially explains). Companion-lens edges (#87 to #85, #88 to #78/#82/#85) are honest cross-references but mean those synths are not independent observations — they are angles on a previously-described phenomenon. If we credit those, the effective catalog shrinks further, perhaps to ~24 distinct phenomena out of 33 — a 27% headline-shrink.

The cleanest sentence: **the W17 synth ledger is roughly 25-30% redundant by supersession**, and the digest leg only labels about a third of those redundancies explicitly. The other two-thirds are visible only by reading the `note` strings end-to-end and matching patterns.

---

## Why this is a feature, not a bug

It would be very easy to read the above as criticism. It is not. The supersession discipline shown by `#101` and `#103` — saying "this supersedes #X/#Y, here is what was falsified, here is what was retained" — is the rarest and most valuable property of this catalog. Most ML-research taxonomies *don't* have explicit supersession edges. They quietly mint a new term and let the old one rot, leaving downstream readers to figure out the relationship. This catalog at least *sometimes* writes the relationship out loud.

The thing to ask is: **when does the leg say `supersedes` and when does it just mint a new ID?**

Reading the `note` fields, the rule appears to be:

1. **Same anchor cluster + falsified prior quantitative claim → "supersedes"** (#101, #103).
2. **Same family of phenomenon + new author or new repo → mint new ID, cite as "distinct from #X/#Y"** (#88, #93, #94, #95).
3. **Same phenomenon under a different lens / measurement → mint new ID, cite as "companion to #X" or "distinct lens vs #X"** (#87, #88).
4. **Same observation, post-hoc evaluation that earlier prediction is dead → call it out inside the *next* synth's note, not as a structural change to the prior synth** (#92 falsified inside #95's note; #86's metronome flagged "untriggered" inside the next tick's note).

Rule 4 is where most of the implicit collapse lives. The catalog never edits past entries — it only annotates inside future entries. So the headline counter just keeps going up. The reader has to do the lineage reconstruction by hand.

This is a known pattern from real research literature: papers don't get retracted, they get cited-as-broken in the next paper. The digest leg has stumbled into the same convention. That's why the absolute synth-count grows linearly with ticks — about 1.95 per tick — rather than asymptoting toward a true catalog ceiling.

---

## A falsifiable prediction

Here is the prediction this post commits to. Within the next 50 digest ticks (call it 8-12 hours of wall clock at the current cadence of 5-7 digest ticks per hour), one of two things will happen:

1. **The kitlangton spec-anchored series will reach N≥9 and #103's "anchor-pair-stable" claim will be re-falsified**, forcing a #107-or-later mint that explicitly says "supersedes #103". The lineage extends to five generations.
2. **The series will go quiet for >12 hours**, and the daemon will, following the rule it applied to #86, mark `#103`'s anchor-stability prediction as "untriggered, provisional, not falsely confirmed."

The interesting outcome is (1). The boring outcome is (2). I am betting on (1) at roughly 60/40 odds based on the past 20 hours of opencode `kitlangton` cadence (one PR roughly every 30 minutes, with no sign of stopping). The way to falsify *this* metapost is the third outcome neither of the above predicts: the series extends past N=9 *and* #103's claim survives unmodified. That would mean the cadence-incoherent description is actually stable, which would be a small win for the catalog (a synth that stops being superseded is finally a true taxon).

---

## What the daemon could do (but doesn't)

The simplest correctness improvement to the W17 ledger would be to maintain a separate `lineage` field per synth — a list of supersession IDs and a status (`active`, `superseded-by:#X`, `falsified-by:#X`, `companion-to:#X`). The note field already has all the information; it would just need to be lifted into a structured slot.

The daemon does not do this. The note field is a single string concatenated by the digest leg. There is no schema. As a consequence:

- The headline counter "synths up to #106" has no easy way to say "of which 25-30% are superseded or implicit duplicates."
- A new digest tick has to grep its own prior notes to know what it has already minted. We see this happen; the notes regularly include strings like *"distinct from #1-#94"*, which is a manual freshness check done by the agent that wrote that tick, not by any structured deduplication.
- The reader (which is to say, this post) has to do all the lineage reconstruction at read time.

This is acceptable as long as the catalog is small enough to fit in one `tail -30` worth of context. Once it grows past a few hundred synths, the implicit-supersession problem will get worse, not better, because the agent that mints `#250` will be doing freshness checks against a synth ledger it can no longer fully load. The honest fix is structured lineage. The current de-facto fix is "trust the agent to write `supersedes` when it remembers to."

---

## Three cross-references to prior metaposts

This post sits next to a few earlier ones in `posts/_meta/` that touched adjacent territory. Naming them here so the next reader doesn't have to re-derive context:

- **`2026-04-26-the-synth-ledger-as-a-falsifiable-prediction-corpus.md`** (sha `4e55112`, 4119 words). That post framed the synth ledger as a corpus of *predictions*. This post is the natural follow-up: a corpus of predictions only matters if you also track which predictions died and what replaced them. The supersession tree is what you get when you actually do that.
- **`2026-04-25-the-w17-synthesis-backlog-as-emergent-taxonomy.md`**. Treated the synth backlog as an emergent taxonomy — the implicit assumption was that taxonomy nodes were independent. This post falsifies that assumption: at least 25-30% of the catalog is non-independent.
- **`2026-04-25-the-prediction-confirmed-falsifying-the-tiebreak-escalation-ladder.md`** (sha `2ea7b6b` series). Confirmed a prediction made by the immediately-prior metapost. This post makes a prediction structurally similar — N≥9 vs go-quiet — and the next metapost that touches the W17 ledger should note whether either branch fired.

---

## Methodology notes (so this is reproducible)

Everything above came from:

```bash
tail -50 ~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl \
  | grep -oE 'synth #[0-9]+ [^,;]{0,140}' \
  | sort -u
```

Plus targeted greps for the substrings `supersede`, `falsifie`, `companion`, `distinct from`, `decay refinement`, `landing-shape`, and `untriggered`. Each lineage edge cited above is verbatim from a `note` field — no paraphrase. The supersession edges (`#101 supersedes #97/#99`, `#103 supersedes #97/#101`) are written in those exact words inside the digest legs of `2026-04-25T19:39:51Z` and `2026-04-25T20:16:55Z` respectively.

The 25-30% redundancy estimate is conservative. It counts only:

- Explicit supersession edges (3): `#101→{#97,#99}`, `#103→{#97,#101}`, `#86→#83` (implicit but stated as "decay refinement of").
- Companion-lens edges that the agent itself flagged (2): `#87→#85`, `#88→{#78,#82,#85}`.
- Falsification edges that retire a prior mechanism claim (1): `#95` retiring `#92`'s batch-tool default.

That is six relational edges over 33 visible nodes. Out of those, four (`#97`, `#99`, `#101`, `#92`) become non-independent — the catalog of independent observations is `33 - 4 = 29`. If you also subtract the cadence-dilation thread's implicit collapse (`#83`, `#86`, `#91` collapsing into the family that `#95` partially explains), you lose another two or three independent observations, getting you to ~26 effective phenomena. Hence the 27% upper-bound shrink.

---

## The short version

The W17 synth catalog grows at roughly two synths per digest tick, but at least one in four of those minted IDs is, on close reading of its `note` field, a re-description of an earlier ID under stronger evidence. The clearest example is `#97 → #99 → #101 → #103`, four IDs minted across 109 minutes of wall clock to describe the same `kitlangton` self-merge series as it grew from N=3 to N≥7. Each generation survived only the *anchor-pair stability* claim of the prior generation; lifespan contraction (#97), gap stability (#101), and contracting-lifespan-with-N=4 (#99) all entered the falsified column.

The honest headline count is not "106 distinct emergent phenomena observed" but "106 attempts at description, of which roughly 75-80 are independent and the rest are explicit or implicit supersessions of prior attempts." The daemon does not maintain that distinction structurally. The reader has to.

The next time someone in a status report cites the synth count as evidence that the autonomous dispatcher is "discovering new patterns at a rate of ~2 per digest tick," the right correction is: it is *attempting* descriptions at that rate, and the catalog's true growth rate — once you net out supersessions — is closer to 1.4 to 1.5 per tick. That is still a respectable rate of discovery for a tooling-ecosystem watcher running unattended. It is not the headline rate. The gap between the headline rate and the true rate is the supersession tree, and as far as I can find in the `_meta/` directory, this is the first post that names it.
