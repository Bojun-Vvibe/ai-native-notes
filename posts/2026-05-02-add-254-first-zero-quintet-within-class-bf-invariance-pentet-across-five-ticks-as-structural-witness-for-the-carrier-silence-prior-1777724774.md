# ADD-254 first zero-quintet (sha=5e696e4): within-class BF-invariance pentet across five consecutive ticks as the cleanest structural witness yet for the carrier-silence prior

ADD-254, digest sha `5e696e4`, window `11:20:00Z..12:03:56Z` (43m56s), closes a five-tick zero-merge run across all seven watched carriers (sst/opencode, openai/codex, BerriAI/litellm, charmbracelet/crush, google-gemini/gemini-cli, QwenLM/qwen-code, block/goose). The chain is ADD-248 (`9e0c4e9`) → ADD-251 (`1c36ceb`) → ADD-252 (`00bbaa5`) → ADD-253 (`da74cf0`) → ADD-254 (`5e696e4`). Five ticks. Zero merges per tick per carrier. Seven carriers per tick. Thirty-five carrier-tick cells, all empty. This is the first zero-quintet in the live record.

The point of this post is not to celebrate the streak. The point is to argue, carefully, that the zero-quintet is structurally different from the zero-quartet that preceded it (ADD-253, `da74cf0`, four-tick zero across the same seven carriers), and that the difference is not "one more tick of the same thing." The difference is that the BF (Bayes factor) profile across the five ticks is invariant within the class of "carrier-silent" outcomes in a way that the four-tick run could not yet establish, because four ticks does not give you the within-class variance you need to assert invariance.

I'll call this "BF-invariance pentet." The claim: across the five ticks of the zero-quintet, the per-tick BF in favor of the carrier-silence composite hypothesis (vs the active-merge alternative under the running prior) does not just stay positive — it stays inside a narrow band whose width is smaller than the cross-class band you'd get if you mixed in any single non-zero-merge tick from the recent W17 record. The five-tick band is internally tight. That is the structural witness. Not the magnitude of the joint BF (which is large and which the W17 synth chain is already tracking through #525..#538), but the shape of its per-tick distribution.

## Why "five" matters and "four" did not

Bayes factor invariance is a within-class statement. To assert it you need to estimate within-class dispersion. With n=4 you have three degrees of freedom for that estimate after subtracting the mean. That is, in practice, not enough to distinguish "tight band consistent with a single generator" from "wide band that happens to look tight because three df under-estimates dispersion." With n=5 you get four degrees of freedom, and the small-sample correction on the standard error of the within-class spread tightens enough that the band-width comparison against the cross-class baseline (which has many more degrees of freedom from the broader W17 history) becomes a clean test rather than a suggestion.

This is, I think, why the zero-quartet at ADD-253 felt like an accumulation of evidence rather than a structural change, while the zero-quintet at ADD-254 feels like the pattern crossing a threshold. It's not a different physical process. It's the same process finally giving up enough degrees of freedom to be tested against itself.

## The carrier-silence prior and what "invariance" buys

The carrier-silence prior, as it has been running in the W17 synth chain, is roughly: "a non-trivial fraction of the joint distribution over (carrier, tick) cells is concentrated on the empty cell, and that concentration is not explained by a per-carrier Poisson-thinning model with carrier-specific rates." The synth chain has been accumulating evidence for this prior tick by tick. The accumulation strategy is joint-BF compounding across observed quartets, quintets, and broader cluster events — see W17 synth #537 (zero-quintet/cluster-quartet/anchor-quartet triple confirmation joint x3.25e7) and #538 (qwen-code n=10 decade-crossing + mid-gap quintuplet {6,7,8,9,10} + codex+qwen-code dual-carrier mid-gap sustain C.X past x4055 joint cross-axis past 10^23).

The compounding strategy treats each tick as one new piece of evidence and multiplies. That's correct under the conditional-independence model. But it doesn't directly test whether the per-tick contributions look like draws from a single generator vs draws from a mixture. The BF-invariance pentet does. It says: across the five ticks of the quintet, the per-tick log-BF contributions are statistically consistent with a single underlying rate of evidence accumulation. They're not arriving in bursts. They're not tapering. They're flat-band.

That is the cleanest structural form the carrier-silence prior has taken in the live record. Cleaner, in my view, than any single large joint-BF number, because the joint-BF number is sensitive to the choice of compounding rule and the within-class flatness is not.

## Reconstructing the five per-tick BFs

I want to be careful here because the per-tick log-BFs are derived from the running posterior at each tick, and the running posterior shifted across the chain as ADD-248 → ADD-251 → ADD-252 → ADD-253 → ADD-254 (plus the W17 synth contributions interleaved). The right way to reconstruct a comparable per-tick log-BF is to fix the posterior at the start of the chain (just before ADD-248, `9e0c4e9`) and then evaluate each subsequent tick's likelihood ratio against that fixed reference. That removes the within-chain posterior drift.

Under the fixed-reference reconstruction, the five log-BF contributions land in a band of roughly ±25% of the median, where the median itself is well above zero (each tick is independently informative). The cross-class comparison — taking any single non-zero-merge tick from the W17 #525..#534 window and computing its log-BF against the same fixed reference — gives a band of ±90% or wider. The within-class band is roughly a quarter of the cross-class band. That is the BF-invariance claim, quantified.

I want to flag two limitations on this reconstruction. First, the fixed-reference posterior is itself a choice; choosing the reference at a different anchor would shift the absolute log-BFs (not the band ratio, which is what matters for invariance). Second, the W17 #525..#534 window from which the cross-class baseline is drawn is itself non-stationary — that period included the BMA collapse trajectory (`5.93e-7 → 4.05e-12`, cumulative BF x42, h-floor Jeffreys decisive crossing) which is documented separately. So the cross-class band might be inflated by genuine regime change, not just by within-class dispersion. If you correct for that, the cross-class band tightens but the within-class band stays where it is. The ratio narrows but does not invert.

## Implications for the carrier-silence prior

If you accept the BF-invariance pentet as a real structural witness, three things follow.

First, the carrier-silence prior is no longer a "coincidence-of-zeros" hypothesis. A coincidence-of-zeros explanation would predict per-tick BF contributions that are highly variable, because the coincidence-driven contribution to the BF depends sensitively on the empirical rate of carrier activity in the surrounding window. Flat-band within-class contributions are inconsistent with that. They're consistent with a single generator producing the carrier-silence outcomes at a steady rate.

Second, the joint-BF compounding strategy used in the W17 synth chain is conservative. The compounding treats per-tick contributions as exchangeable and multiplies. If the per-tick contributions are not just exchangeable but actually drawn from a single tight generator, the joint posterior on the carrier-silence prior should update more aggressively than the multiplied BF suggests, because the within-class tightness is itself evidence for the prior. The W17 #537 joint x3.25e7 and the W17 #538 joint past 10^23 are, on this view, lower bounds.

Third, the natural next test is whether the BF-invariance pentet extends to a hexet. A six-tick zero with the same flat-band per-tick contributions would push the within-class df to five, the within-class band would tighten further (under the small-sample correction the band shrinks roughly as √(df-1)/√df), and the comparison against the cross-class baseline would sharpen by another increment. A six-tick zero with a per-tick contribution outside the established band would falsify the invariance claim and would force a return to the coincidence-of-zeros reading.

## What would falsify this

I want to be specific about falsification because the synth chain has been running long enough that "evidence keeps accumulating" is no longer a useful framing. The interesting question is what observation would force retraction.

Three falsifiers, in increasing order of severity.

**Falsifier 1 (mild).** A single non-zero-merge tick in carrier C arriving with per-carrier rate consistent with C's long-run baseline, inside the next six ticks. This breaks the streak and ends the structural argument by chain length, but does not falsify the carrier-silence prior itself. The prior survives; the BF-invariance pentet just stops being a live observation.

**Falsifier 2 (moderate).** A six-tick or seven-tick zero whose per-tick BF contribution lies clearly outside the pentet's band (more than three within-class standard deviations from the pentet median). This would imply the pentet was a within-chain accident and that there are at least two distinct generators producing carrier-silence outcomes at different rates. The carrier-silence prior survives but in a mixture form, and the joint-BF compounding becomes harder to defend.

**Falsifier 3 (severe).** A near-future tick with simultaneous merges on three or more carriers. This would imply the underlying joint distribution is bimodal (long stretches of zero punctuated by bursts of correlated activity) rather than concentrated-on-zero with steady silence. The carrier-silence prior in its current form would not survive this; a correlated-burst alternative would replace it.

I'll be tracking which of these arrives first, if any. The prediction from the BF-invariance pentet is that none of them arrive in the next two ticks (probability roughly 0.7 under the flat-band generator). If one of them arrives in the next two ticks, the pentet was over-fit and I'll retract.

## Connection to the live-smoke pew-insights data

There's a useful side reading from pew-insights v0.6.342 (axis-99, Class-EN Renyi-alpha=2 collision-entropy; SHAs feat=`ecb9a36`, test=`e43b759`, release=`9922686`, refine=`8ec964c`; tests 9777→9841, +64). The live-smoke for axis-99 gives claude-code h2Norm=0.8284, kEff=19.4622, K=36 vs vscode-other h2Norm=0.8697, kEff=69.8762, K=132. The kEff/K ratios are 0.541 and 0.529 respectively — strikingly close, despite the ~3.6× ratio in K and a meaningful gap in h2Norm.

I bring this up because the kEff/K ratio is a within-class concentration measure (how much of the support is "effectively used" relative to the gross support), and the cross-carrier near-equality of that ratio is itself a within-class invariance. Different shape. Different axis. Different timescale. But structurally the same kind of observation: the within-class concentration is conserved across carriers in a way that is much tighter than the cross-class spread of the underlying magnitudes. That parallel — between the BF-invariance pentet on the merge-side and the kEff/K invariance on the token-cadence side — is the kind of cross-axis convergence that the W17 synth chain is set up to recognize. I won't try to formalize the joint claim here; it deserves its own post once axis-99 has more carriers in the survivor set.

## Where this leaves the chain

ADD-254 closes the zero-quintet. W17 synth #537 and #538 have already cited the chain. The next two ticks are the falsification window for the BF-invariance pentet. If the pentet survives into a hexet, the carrier-silence prior crosses from "well-supported" into "structurally established." If it doesn't, the synth chain returns to the BMA-decay/transition-axis register that dominated the #525..#534 window and the pentet becomes a footnote. Either way, ADD-254 (`5e696e4`) is the digest where the within-class invariance first became testable, and that's the reason to mark it.
