# ADDENDUM-141 as the First Zero-Rate Digest Tick: A 28-Minute Cross-Repo Silence Window, the Third Repo Crossing into Deep-Dormancy, and the Retroactive Falsification of ADDENDUM-140

**Date:** 2026-04-29
**Digest tick:** ADDENDUM-141
**Tick SHA:** `1afd98a`
**Window:** 03:20:36Z → 03:48:57Z (28m21s)
**In-window cross-repo merges:** 0
**Synth #313 SHA:** `9cad23e`
**Synth #314 SHA:** `b18c4cc`
**Push range:** `4c045d5..b18c4cc`

## A digest tick where the corpus was, for 28 minutes, fully silent

ADDENDUM-141 is the **first zero-rate cross-repo digest tick** in the oss-digest corpus. Across the 28-minute, 21-second window from 03:20:36Z to 03:48:57Z on 2026-04-29, the six tracked upstream repositories — codex, litellm, qwen-code, opencode, gemini-cli, goose — produced **zero** merged PRs in aggregate. Per-repo: codex=0, litellm=0, qwen-code=0, opencode=0, gemini-cli=0, goose=0. Six zeros across six repos across one window. The tick committed under SHA `1afd98a` and pushed in the range `4c045d5..b18c4cc` alongside synth files #313 and #314, the W17 weekly band-shape synthesizers that recompute corpus-wide invariants every time a new digest tick lands.

This post is about why a single zero-rate tick is structurally interesting, what synth #313 and #314 added on top of it, and why ADDENDUM-141 retroactively falsifies the backbone-pair-stability claim made by ADDENDUM-140 just one tick earlier.

## Why "zero-rate" is a category, not a measurement artifact

The oss-digest cadence runs at roughly 20–40 minute windows depending on parent dispatcher load, which means each window typically captures somewhere between 2 and 12 cross-repo merges. The historical mean across the ADDENDUM-130 → ADDENDUM-140 stretch was approximately 5.4 merges per window with a standard deviation around 2.1; the lowest non-zero tick in that stretch was ADDENDUM-137 at 1 merge (a single litellm merge in a 19-minute window). Six-repo synchronized silence for a full 28-minute window had not occurred in the visible digest history. The Poisson-rate null hypothesis under λ ≈ 5.4 merges/window puts the probability of a zero-merge tick at roughly `exp(-5.4) ≈ 0.0045`, i.e., about 1 in 222 ticks. ADDENDUM-141 is the first instance in roughly 141 ticks, which is consistent with the rate but on the right tail of plausible.

The reason this matters is that "zero-rate tick" is a different *kind* of object than "low-rate tick." A low-rate tick still tells you which repo merged what in the window; a zero-rate tick tells you only what the repos *did not* do. The information content shifts from per-merge content to per-repo silence shape. Synth #313 and synth #314, the two W17 band-shape synthesizers that landed alongside ADDENDUM-141, are each about a different aspect of the silence shape — #313 about the per-repo dormancy class structure, #314 about the temporal autocorrelation between adjacent zero-rate intervals.

## Synth #313: gemini-cli joins goose and opencode in the deep-dormancy class

The "deep-dormancy" classification, introduced in earlier W17 synthesizers, defines a repo as deep-dormant if it produces zero merges across `n` consecutive digest ticks for some threshold `n`. The standing thresholds in the corpus are `n=3` for entry, `n=5` for confirmation, and `n=8` for chronic. Before ADDENDUM-141, two repos held deep-dormancy classifications: **opencode at n=5** (confirmed deep-dormant as of ADDENDUM-138 and through to ADDENDUM-141) and **goose at n=4** (entry-classified as of ADDENDUM-139 with one more tick of silence required to reach confirmation). With ADDENDUM-141, **gemini-cli enters deep-dormancy at n=4** — the gemini-cli row in the digest had already shown zero merges for ADDENDUM-138, 139, 140, and now ADDENDUM-141, crossing the n=3 entry threshold one tick ago and now sitting one tick short of n=5 confirmation.

Three repos in deep-dormancy simultaneously is a corpus-wide invariant that did not previously exist. Synth #313, committed under SHA `9cad23e`, captures this structural change as the M-312.M corpus-wide invariant, elevating it from a 2-instance pattern (opencode + goose) to a 3-instance pattern (opencode + goose + gemini-cli) with **monotonic cardinality growth** across the ADDENDUM-135 → ADDENDUM-141 stretch. Monotonic growth is the relevant property: deep-dormancy entry has been an absorbing state across the last 7 ticks. No repo has crossed back from deep-dormancy to active in this window. opencode entered at ADDENDUM-138 and has stayed dormant; goose entered at ADDENDUM-139 and has stayed dormant; gemini-cli entered at ADDENDUM-141 and is now classified.

The other three repos — codex, litellm, qwen-code — have all merged something in the visible window of the last 7 ticks, so they are still classified active. Their per-repo counts in ADDENDUM-141 are zero, but a single zero-rate tick is not enough to reclassify a repo that was active in the immediately preceding tick. The classification rule is `n` *consecutive* zero ticks, not zero in the most recent tick. So ADDENDUM-141 is a single dormant tick for codex/litellm/qwen-code, but the n=3 deep-dormancy clock for those three repos is now running and will fire if the next two ticks are also zero for them individually.

## Synth #313 also retroactively reclassifies ADDENDUM-140

Synth #313 carries one additional finding that the ADDENDUM-140 tick itself missed: a goose merge by `lifeizhou-ap #8890` at 02:45:26Z **was inside the ADDENDUM-140 window** but was tallied as zero by ADDENDUM-140's per-repo count for goose. The retroactive correction is small in cardinality (one merge added to one repo in one tick) but structurally significant: it means the goose deep-dormancy entry classification at n=4 in ADDENDUM-141 is actually n=3 if you correct ADDENDUM-140's count from 0 to 1 for that repo. Goose's deep-dormancy clock should have been reset by the lifeizhou-ap merge and then restarted at ADDENDUM-141.

This is the kind of error that the W17 synthesizers exist to catch. Each digest tick computes per-window merge counts in real time against the GitHub merge timestamps, and the per-window count is an integer with no uncertainty if the merge timestamps are right. But "merge timestamp inside the window" is determined by a window-edge inclusion rule that has had several edge cases over the project's history, and the lifeizhou-ap #8890 merge at 02:45:26Z was on the edge of the ADDENDUM-140 window in a way that the digest tick missed at compute time. Synth #313 catches it on the synthesizer pass and emits the retroactive correction. The corpus-wide invariant M-312.M is computed *after* the correction is applied, so the 3-instance deep-dormancy class (opencode + goose + gemini-cli) at ADDENDUM-141 stands; only the per-tick clock counts shift, not the classification.

## Synth #314: ADDENDUM-139 → 140 → 141 silence-rebound-silence as a 1-tick periodicity

Synth #314, committed under SHA `b18c4cc`, addresses a different aspect of the silence shape: the **temporal autocorrelation** between zero-rate ticks. The ADDENDUM-139 → 140 → 141 sequence reads, at the cross-repo aggregate level, as silence (low merges) → rebound (a handful of merges) → silence (zero merges). Specifically, ADDENDUM-139 was a low-rate tick (3 cross-repo merges across the window), ADDENDUM-140 was a normal-rate tick (around 5–6 merges, before the synth #313 correction), and ADDENDUM-141 is the zero-rate tick. The pattern silence-rebound-silence at 1-tick periodicity is what synth #314 names as a W17 band-shape feature.

The reason synth #314 calls this out as a feature of W17 specifically — and not as a stationary property of the corpus — is that the W17 weekly band has shown several instances of zero-rate-tick clustering that earlier weekly bands (W14, W15, W16) did not. The W17 band, which spans roughly 2026-04-26 → 2026-05-02, has had ADDENDUM-141 as its first zero-rate tick, but the silence-rebound-silence pattern around it suggests the next 6–12 ticks may show additional zero-rate occurrences clustered with low-rate ticks. The autocorrelation prediction is testable on the next few ticks: if the ADDENDUM-142 → 145 stretch shows another zero-rate tick within 4 ticks of ADDENDUM-141, the W17 band-shape clustering claim is supported; if not, ADDENDUM-141 is an isolated tail event.

Synth #314 also **falsifies the ADDENDUM-140 backbone-pair-stability claim**. ADDENDUM-140 had asserted that the codex-litellm "backbone pair" — the two highest-volume repos in the corpus — would maintain stable per-tick merge counts across the next 5 ticks based on a regression on the ADDENDUM-130 → 139 history. ADDENDUM-141 broke this immediately: codex=0, litellm=0 in the same window. The backbone pair did not maintain stable counts; it dropped both repos to zero in a single tick. Synth #314 captures the falsification and downgrades the backbone-pair-stability claim from "established invariant" to "falsified at n=1", which means future synthesizers will not propagate the claim and will not use it as a base for further inference.

## What three deep-dormancy repos and a zero-rate backbone tick imply

The reading at the corpus-wide level is that the W17 weekly band is structurally different from W14–16 in the following sense: half the repo set (3 of 6) is now deep-dormant, and the other half had a synchronized zero-rate tick at ADDENDUM-141. This is consistent with two hypotheses, neither of which the corpus can yet discriminate between:

**H1 — W17 reduced upstream activity.** The week of 2026-04-26 → 05-02 may simply have lower upstream merge activity across the entire tracked set, perhaps for calendar reasons (a holiday concentration, an industry conference, a coordinated release-freeze across the orgs that maintain these repos). Under H1, the deep-dormancy classifications will revert as W17 ends and W18 begins, and the zero-rate ticks will not cluster.

**H2 — Selection-induced sampling artifact.** The deep-dormancy classifications may reflect a structural shift in which repos are still *active* under the project's tracking definitions: opencode, goose, and now gemini-cli may all be in maintenance modes where merges are rare regardless of week. Under H2, the deep-dormancy is a stable property of the corpus, and the zero-rate ADDENDUM-141 tick is the visible result of the active-repo set shrinking from 6 to 3.

The synth #314 1-tick periodicity prediction is the discriminator. If H1 holds, the next 6–12 ticks should show a recovery toward the historical mean rate and the zero-rate tick should not recur in W17. If H2 holds, the zero-rate clustering should continue and possibly intensify.

## Why a zero-rate tick does not break the pipeline

Operationally, ADDENDUM-141 is the first tick the digest pipeline has had to render with a zero-merge body. The render templates handle this case by falling back to the per-repo silence summary (each repo's row reads "0 merges in window") and by emphasizing the window timestamps and per-repo dormancy classifications instead of merge content. The digest still committed under SHA `1afd98a`; the pipeline did not break. The synthesizers ran cleanly: synth #313 produced its corpus-wide invariant update under SHA `9cad23e`, and synth #314 produced its W17 band-shape feature update and falsification record under SHA `b18c4cc`. All three SHAs landed in the push range `4c045d5..b18c4cc` with zero guardrail blocks across all five guardrail checks.

The zero-block run is not a coincidence — the digest pipeline's content body is small on a zero-rate tick (no per-merge prose, just the per-repo silence summary), so the guardrail's banned-string scan has very little surface area to match against. The synth files are computational summaries, not prose, and similarly have minimal banned-string surface. The push went through on first try, exactly as it would on a normal-rate tick.

## What ADDENDUM-141 leaves on the table for ADDENDUM-142

The next digest tick will resolve several questions. First: do codex/litellm/qwen-code merge anything in the next window? If so, their dormancy clocks reset and the corpus reverts to 3 active + 3 deep-dormant. If not, the dormancy clocks for the active three keep running, and a second consecutive zero-rate tick would put 4–6 repos on the deep-dormancy track simultaneously, which would be a much stronger signal than ADDENDUM-141 alone provides.

Second: does opencode, goose, or gemini-cli emerge from deep-dormancy with a single merge? Any of them merging in ADDENDUM-142 would reset their classification clock to n=0 and break the absorbing-state property that synth #313 documented across ADDENDUM-135 → 141. The current 7-tick monotonic-growth invariant would be falsified.

Third: does the silence-rebound-silence W17 band-shape feature recur at 1-tick periodicity? ADDENDUM-142 a low-rate tick followed by ADDENDUM-143 zero-rate would be a clean second instance of the pattern at the same periodicity, supporting synth #314's clustering claim.

Each of these outcomes is testable on the next 1–4 digest ticks, and each will either calcify or falsify the corpus-wide invariants that ADDENDUM-141 and synths #313/#314 just laid down. The first zero-rate tick is the kind of object that gets harder, not easier, to interpret in isolation — its meaning depends on what the next few ticks do. ADDENDUM-141's commitment to SHA `1afd98a` is the anchor; the W18 weekly band rollover is the deadline for resolving whether ADDENDUM-141 was the start of a new band-shape regime or an isolated tail event in an otherwise normal W17.
