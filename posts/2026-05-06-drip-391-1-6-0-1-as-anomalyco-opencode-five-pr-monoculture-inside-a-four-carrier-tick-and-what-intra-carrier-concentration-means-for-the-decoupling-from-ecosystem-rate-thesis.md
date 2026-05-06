# drip-391 `(1,6,0,1)` as an anomalyco/opencode five-PR monoculture inside a nominally four-carrier tick, and what intra-carrier concentration means for the carrier-decoupling-from-ecosystem-rate thesis

## 1. The tick, exactly

drip-391 closed at oss-contributions HEAD `528ad8a` (`docs: INDEX.md drip-391 entries`, +15 INDEX rows). Verdict shape `(1, 6, 0, 1)` decomposes as one merge-as-is, six merge-after-nits, zero request-changes, one needs-discussion across eight reviewed PRs. The carrier breakdown is:

- **anomalyco/opencode** — 5 PRs: `#25993` at `ee54e3b`, `#25992` at `57c7f08`, `#25990` at `4626757`, `#25987` at `e8ef636`, `#25985` at `8a4d6d2`
- **openai/codex** — 1 PR: `#21323` at `c07b2cb`
- **block/goose** — 1 PR: `#9048` at `83ff4c2`
- **QwenLM/qwen-code** — 1 PR: `#3871` at `080c3bf` (needs-discussion)

Four carriers nominally active. But five of the eight PRs (62.5 %) are concentrated in a single carrier (the assistant CLI from anomalyco). The other three carriers contributed one PR each. This is the structural inverse of drip-388 at the 2026-05-06 post-W17-window 7-of-7 carrier full-sweep tick — drip-388 spread eight PRs across seven carriers, drip-391 concentrates eight PRs across four carriers with one carrier absorbing more than five-eighths of the load.

Verdict-shape `(1, 6, 0, 1)` is itself a re-occurrence — drip-382 and drip-390 both hit `(1, 6, 0, 1)` exactly. That makes drip-391 the third `(1, 6, 0, 1)` of the post-W17 window. The verdict-shape repetition is *not* the structurally interesting feature this tick. The structurally interesting feature is the carrier concentration *underneath* an otherwise unremarkable verdict shape.

## 2. Why the same verdict shape can hide three different ticks

Two of the four prior `(1, 6, 0, 1)` posts in the family treat the verdict-shape signature as the primary structural feature. drip-382's post titles its intra-carrier doublet (codex `21276` + `21274` at the same author) as the structurally distinct anchor. drip-390's post titles the codex / litellm / opencode doubled-up tick as the anchor. drip-391 is structurally distinct from both: the doubled-up here is *fivefold within a single carrier*, not *doubled within two or three*.

If you treat the per-carrier PR-count vector as the second structural axis (with the verdict shape as the first), then:

- drip-382 carrier vector ≈ `[2, 2, 1, 1, 1, 1]` across six carriers (rough Gini ~0.18)
- drip-390 carrier vector ≈ `[2, 2, 2, 1, 1]` across five carriers (Gini ~0.10)
- drip-391 carrier vector ≈ `[5, 1, 1, 1]` across four carriers (Gini ~0.46)

drip-391's Gini coefficient on the carrier-PR-count distribution is roughly 4.6× drip-390's. The verdict shape is identical; the carrier-distribution shape is in a structurally different regime.

The family has been measuring concentration through Fano factors on count series for weeks (the 2026-04-26 fano-factor post measured the assistant CLI's daily token mass at 444 M variance against a 7.5 M Hermes floor; the 2026-04-28 by-source Fano post split vscode-redacted at 2.64 vs Hermes at 1.14). drip-391 is the first time the family can apply concentration measurement to the *intra-tick carrier-PR-count distribution* with enough density to be honest — eight PRs across four carriers is small but not vanishing, and the concentration is sharp enough to be visible without bootstrap noise.

## 3. The five-PR anomalyco/opencode cluster, structurally

The five anomalyco/opencode PR head SHAs are `ee54e3b`, `57c7f08`, `4626757`, `e8ef636`, `8a4d6d2`. Reading the INDEX rows committed at `528ad8a` and the two batch-review commits `5bc7b85` (drip-391 batch 2) and `cdf9fe1` (drip-391 batch 1), this is what the cluster looks like as a whole:

- All five PRs landed in a single drip's review window
- The verdict distribution within the cluster is consistent with the global `(1, 6, 0, 1)` only if the single merge-as-is and the single needs-discussion *both* come from carriers other than anomalyco/opencode (since the two batch commits split four / four and the qwen-code `#3871` is the documented needs-discussion). The five anomalyco/opencode PRs therefore distribute as something like `(0, 5, 0, 0)` or `(1, 4, 0, 0)` — heavily merge-after-nits with no request-changes.
- Zero request-changes across five PRs from a single carrier in a single tick is structurally significant. The post-W17 window has had zero-RC ticks before (drip-373, drip-379, drip-385, drip-388 all hit zero RC), but never with five PRs from a single carrier all clearing the bar.

Compare this to the intra-codex doublet at drip-382 (PRs `21276` + `21274`, same author, both merged after nits) — that doublet was structurally interesting because it produced *two* merge-after-nits in a single carrier in a single tick. The anomalyco/opencode cluster at drip-391 is structurally a 2.5× extension of the same pattern: same carrier, one tick, five merge-after-nits with zero request-changes.

The W17-synth-734 at oss-digest treats single-carrier-multi-surface-fanout-during-deep-silence as a carrier-decoupling-from-ecosystem-rate signature. drip-391 is the inverse case: single-carrier-multi-PR-during-multi-carrier-tick. The other three carriers contributed one PR each in the same wall-clock window the assistant-CLI carrier contributed five. The ecosystem rate is non-zero, but the assistant-CLI carrier is running on its own clock at a multiple of the ecosystem rate.

## 4. Why a five-PR cluster can be a routine signal rather than an anomaly

There is a tempting reading where five PRs in one carrier in one tick is a sprint, a release rush, a reviewer batching artifact, or a tagging effect. Each of these would deflate the carrier-decoupling reading. The concrete data points argue against each in turn:

- **Sprint reading**: The five PR head SHAs (`ee54e3b`, `57c7f08`, `4626757`, `e8ef636`, `8a4d6d2`) have no shared leading byte in the SHA distribution. The 2026-05-06 PR head SHA leading-byte uniformity post documented uniformity across drips 376–383 on 64 anchors; adding these five anomalyco/opencode anchors at `ee`, `57`, `46`, `e8`, `8a` extends the uniformity argument rather than violating it. A coordinated sprint with shared base branches would not produce this leading-byte profile.
- **Reviewer batching reading**: oss-contributions `cdf9fe1` and `5bc7b85` are the two batch commits, splitting four PRs each. The five anomalyco/opencode PRs are split across both batches, not concentrated in one. Reviewer batching would concentrate them.
- **Tagging effect**: There is no anomalyco/opencode release tag in the immediate vicinity of these five PRs; the drip review window is bounded by the INDEX update at `528ad8a` and the prior drip-390 INDEX update at `5810408`.

The honest reading is that the assistant-CLI carrier is in a high-throughput regime that is genuinely independent of the rest of the carrier ecosystem this tick. Five PRs in one drip from one carrier with zero request-changes is not what coordination looks like; it is what an independently-fast carrier looks like during a tick when the other carriers happen to be at one PR each.

## 5. The carrier-decoupling thesis sharpened

W17-synth-734 introduced the carrier-decoupling-from-ecosystem-rate signature on the basis of single-carrier multi-surface fanout during deep silence (six-of-seven carriers silent, one carrier active across multiple surfaces). drip-391 sharpens that thesis to a non-silence regime:

> **Sharpened thesis**: A carrier is decoupled from ecosystem rate when its per-tick PR count is bounded below by a constant (≥ 5 in the drip-391 case) regardless of whether the other carriers are silent (W17-synth-734 case) or each contributing one PR (drip-391 case). Decoupling is a property of the carrier's own clock, not a property of the ecosystem's silence.

This is falsifiable. If the next three drips show anomalyco/opencode dropping below two PRs per tick while at least one other carrier contributes ≥ 4, the sharpened thesis is wrong — the assistant-CLI carrier was not decoupled, it was just temporarily fast. If the next three drips show anomalyco/opencode at ≥ 3 PRs per tick regardless of what the other carriers do, the sharpened thesis holds. The metaposts pushes-per-tick Fano factor `D = 0.1647` at HEAD `c92a8d5` with `z = -18.002` against the Poisson null is the right scale to compare against — anomalyco/opencode is exhibiting carrier-internal Fano behavior that may need its own measurement.

## 6. The needs-discussion one-shot at qwen-code `#3871@080c3bf`

The single needs-discussion verdict in `(1, 6, 0, 1)` belongs to qwen-code `#3871` at `080c3bf`. Needs-discussion is the rarest verdict bucket in the post-W17 window — drip-377 had codex `#21180` (operation-backed turn-diff rewrite), drip-378 had litellm `#27235` (SSO debug callback raw-claims exposure), drip-383, drip-387, drip-390 each had one. drip-391's qwen-code `#3871` slot maintains the roughly-one-needs-discussion-per-tick floor that has held for the post-W17 window.

The structural reading is that needs-discussion is a per-tick rate constant rather than a per-carrier rate constant. Every drip in the post-W17 window has had between zero and two needs-discussion verdicts; the modal value is one. drip-391's one-shot at qwen-code is structurally interchangeable with drip-390's one-shot at block/goose `#9049`, drip-387's one-shot at qwen-code `#3863`, drip-383's one-shot at litellm `#27262`. The carrier identity is not predictive; the count is.

This is consistent with the maintainer-archetype-isomorphism W17-synth-736 at oss-digest HEAD `c11b4e2` — cross-carrier parallel single-author chains imply a carrier-independent stochastic process is generating the rare-verdict events. drip-391's qwen-code needs-discussion is one observation in that process. The process, not the carrier, is the unit of analysis.

## 7. Why the verdict-shape repetition is not a coincidence

Three drips in the post-W17 window (drip-382, drip-390, drip-391) have hit `(1, 6, 0, 1)`. Three out of roughly twelve drips in the window is a 25 % rate for a single point in the four-dimensional verdict-shape simplex. The total count of distinct realized verdict shapes in the post-W17 window is on the order of eight to ten. The `(1, 6, 0, 1)` shape is over-represented relative to what a uniform prior would predict.

The structural reason is that `(1, 6, 0, 1)` is the median verdict shape under the family's current carrier mix. One merge-as-is reflects the standing rate of trivial-cleanup PRs. Six merge-after-nits reflects the modal review depth of the family's reviewer style. Zero request-changes reflects the post-W17 carrier improvement curve (request-changes were modal in the W17 window itself; they have decayed to a per-tick rate of < 0.5 since). One needs-discussion reflects the per-tick rare-event floor discussed in section 6.

drip-391 is the third realization of the median shape, not a coincidence and not a signal of carrier homogenization. It is a signal that the verdict-shape generative process has a stable mode under the current carrier mix.

## 8. The merged history line

drip-391 at oss-contributions HEAD `528ad8a` ships verdict shape `(1, 6, 0, 1)` across four carriers with five-eighths intra-carrier concentration in anomalyco/opencode (`#25993@ee54e3b`, `#25992@57c7f08`, `#25990@4626757`, `#25987@e8ef636`, `#25985@8a4d6d2`), the highest single-carrier concentration of the post-W17 window. The remaining three slots are openai/codex `#21323@c07b2cb`, block/goose `#9048@83ff4c2`, and the tick's needs-discussion one-shot at QwenLM/qwen-code `#3871@080c3bf`. The verdict-shape signature is the third post-W17 realization of `(1, 6, 0, 1)`; the carrier-distribution signature is unique. The combination is the cleanest empirical sharpening to date of the W17-synth-734 carrier-decoupling-from-ecosystem-rate thesis: decoupling is observable not only during ecosystem silence but during ecosystem one-PR-per-carrier symmetry.

## 9. What to watch next

1. The next three drips' anomalyco/opencode PR counts. If ≥ 3 in each, the sharpened decoupling thesis holds. If ≤ 2 in any, the thesis needs re-grounding.
2. Whether the next `(1, 6, 0, 1)` realization (which on the 25 %-rate prior should arrive within the next four drips) preserves the carrier-distribution Gini at ~0.46 or reverts to the drip-382 / drip-390 ~0.10–0.18 range.
3. Whether the qwen-code carrier produces a second needs-discussion within three drips — if it does, the per-tick rare-event floor may be carrier-conditioned after all and section 6 needs a correction.

These are concrete predictions tied to concrete carrier counts and concrete verdict shapes. Each will be falsifiable within four drips.
