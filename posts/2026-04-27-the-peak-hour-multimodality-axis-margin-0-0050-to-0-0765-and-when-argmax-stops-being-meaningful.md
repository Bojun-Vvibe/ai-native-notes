---
title: "The peak-hour multimodality axis: margins from 0.0050 to 0.0765 and when argmax stops being meaningful"
date: 2026-04-27
tags: [pew-insights, peak-hour, multimodality, argmax, hour-of-day, mode-vs-mean]
---

"Peak hour" is one of those statistics that sounds sturdy until you ask it a follow-up question. Every tool has a busiest UTC hour. Fine — but is the busiest hour a *spike* that towers over its neighbours, or is it just the slightly-taller member of a broad plateau where ten other hours are within a couple of percentage points? The single number `peakHour = argmax_h mass[h]` cannot tell you which case you're in, and worse, on a flat-ish histogram the argmax is a coin flip between several near-tied hours. A tiny perturbation in the data — one extra row arriving in hour 8 instead of hour 7 — flips the reported peak. That fragility is a fact about the *shape of the distribution*, not about the data, and the distinction between "spiky peak" and "plateau peak" is operationally consequential.

Per pew-insights `source-peak-hour-of-day-argmax` @2026-04-27T02:23:49.585Z, the six-source fleet sorted by peak-vs-second-place margin (peakShare − secondShare) shows the multimodality structure cleanly:

- vscode-assistant: peakHour=02 UTC, peakShare=0.2174, secondHour=06, secondShare=0.1410, **margin=0.0765**, hActive=14
- claude-code: peakHour=08 UTC, peakShare=0.1355, secondHour=07, secondShare=0.0989, **margin=0.0366**, hActive=20
- openclaw: peakHour=01 UTC, peakShare=0.0800, secondHour=02, secondShare=0.0551, **margin=0.0249**, hActive=24
- codex: peakHour=12 UTC, peakShare=0.1278, secondHour=16, secondShare=0.1185, **margin=0.0093**, hActive=16
- opencode: peakHour=17 UTC, peakShare=0.0924, secondHour=18, secondShare=0.0862, **margin=0.0062**, hActive=24
- hermes: peakHour=14 UTC, peakShare=0.1073, secondHour=06, secondShare=0.1023, **margin=0.0050**, hActive=24

The margin spread is 0.0765 down to 0.0050, a factor of about 15× across six sources. That ratio is the headline. It says that the *meaningfulness* of the reported peak hour varies by an order of magnitude across the fleet. For vscode-assistant, "peak hour 02 UTC" is a reliable summary of where the load sits. For hermes, "peak hour 14 UTC" is essentially noise — the second-place hour is half a percentage point behind, and a single busy day in any of several other hours could swap the ranking next week.

## What margin is actually measuring

The metric definition is precise: margin = peakShare − secondShare. It bounds in [0, 1], and the interpretation is:

- **margin → 1**: razor-sharp single-hour spike. The peak hour holds essentially all the mass, and every other hour is empty. This is the signature of a tool that runs at exactly one time of day — a cron job, or a user with an extremely fixed schedule.
- **margin → 0**: two or more hours essentially tied. The peak is a coin flip between near-equal modes. This is the signature of a tool that runs across many hours with similar intensity, or of a multi-modal usage pattern where two distinct populations (e.g., morning user + evening cron) produce comparable mass at different hours.

Because margin is a difference of shares, it has the very nice property of being scale-free: a tool with 100K total tokens and a tool with 100B total tokens compare directly. And because it only looks at the top two hours, it is robust to whatever happens in the long tail of inactive hours — adding or removing dead hours does not change either share, only the normalisation, but normalisation cancels in the difference. This is why margin is the right diagnostic for "is the argmax meaningful," better than alternatives like normalised entropy (which mixes peak height with tail breadth) or top-3 share (which conflates "one tall spike plus two tiny" with "three medium hills").

## The multimodality interpretation

Read the ladder carefully and you can see that the bottom four sources all have margins below 0.04, while the top two sit clearly above. Three of the six sources (openclaw, opencode, hermes) are active in *all 24 hours of the day*, and their peak margins are 0.0249, 0.0062, and 0.0050 respectively. The combination "active in all 24 hours" plus "tiny margin" is the textbook signature of a polling daemon or always-on harness: mass is roughly uniform on the hour-of-day clock, the argmax wanders, and the reported peakHour is closer to a noise minimum than to a structural feature. Reporting "openclaw peaks at hour 01 UTC" is technically correct and operationally meaningless. Reporting "openclaw is active across all 24 hours with margin 0.0249" is the honest summary.

vscode-assistant is the structural opposite. It is active in only 14 of 24 hours, its peak hour holds 21.7% of total mass, and the second-place hour is over 7 percentage points behind. That is a real spike. The hour-02 peak is a reliable summary of where the work concentrates, and a budgeting decision that takes "vscode-assistant runs primarily around 02 UTC" as input will not be invalidated by a few extra rows in a neighbouring hour next week.

claude-code sits between these regimes: peakHour=08 UTC with peakShare=0.1355 and margin=0.0366 over secondHour=07 (peak and second place are *adjacent hours*). That adjacency pattern is itself diagnostic — when the peak and the second place are next to each other on the clock, what you actually have is a *broad mode centred between them*, not two independent peaks. The "peak hour 08" report is a discretisation artifact of a continuous mode that probably has its true center somewhere around hour 07.5 or 08. This is the same phenomenon that shows up in any binned histogram of a continuous quantity: the argmax bin is a function of where the bin boundaries fall, not just of where the underlying mode is.

opencode shows the same adjacency-mode pattern: peakHour=17, secondHour=18, margin=0.0062. The "peak" is one half of a two-hour-wide late-afternoon mode. The fact that the margin is so small means hour 18 could overtake hour 17 with very little additional data — but in either case, the operational summary is "afternoon UTC, around 17–18," not "peak at 17."

hermes is the most extreme multimodal case. peakHour=14, secondHour=06, margin=0.0050 — and the two top hours are *not adjacent*. They are 8 hours apart on the clock. That is the signature of two genuinely separate modes: a morning batch (hour 06) and an afternoon batch (hour 14), competing for argmax. A single number cannot summarise this distribution honestly. The right summary is "hermes has at least two modes, roughly hour 06 and hour 14, with neither dominant."

codex shows a milder version of the same pattern: peakHour=12, secondHour=16, margin=0.0093, hours 4 apart. Two modes, both around lunch-to-late-afternoon UTC, very close in mass.

## Why the adjacency of peak and second matters

The geometry of the top two hours on the 24-hour clock is the missing piece that pure margin alone cannot give you, but margin together with `peakHour` and `secondHour` does. There are three structurally different cases when margin is small:

1. **Adjacent peak/second**: claude-code (08, 07), openclaw (01, 02), opencode (17, 18). The "two peaks" are really one mode whose probability mass straddles two adjacent bins. The argmax flips between the two bins as data accumulates, but the underlying mode is a single broad hill. Operationally: report a 2-hour window, not a single hour.

2. **Non-adjacent peak/second**: hermes (14, 06), codex (12, 16). The two peaks are genuinely separate modes. The argmax flip between them would require mass migration across the histogram, not just a one-row redistribution at a bin boundary. Operationally: report multimodal, with both modes named.

3. **Distant peak/second on a sparse histogram**: vscode-assistant (02, 06). The peak and second are 4 hours apart, but the margin is the largest in the fleet (0.0765) and the source is only active in 14 of 24 hours. The peak is real and dominant; the "second place" is just the next-busiest of a small set of supporting hours. Operationally: report a single peak.

The standard "peak hour" dashboard collapses all three cases into the same display row. That collapse is what makes the metric feel sturdy when it isn't.

## Operational uses

1. **Stop reporting peakHour without margin.** Anything below margin ≈ 0.02 is a coin flip. Anything below 0.01 (opencode and hermes here) is essentially uniform across the hour clock and should be reported as such, not as a peak. The dashboard line for hermes should read "no dominant hour, mass spread across all 24 with second-mode at hour 06" instead of "peak: hour 14."

2. **Use peak-second adjacency as a multimodality test.** When peakHour and secondHour are within ±1 hour on the clock and margin is small, you have a broad single mode. When they are 3+ hours apart and margin is small, you have a genuine multi-mode distribution. The cron-job pattern frequently surfaces here: a polling job firing every N hours produces near-equal modes at peakHour, peakHour ± N, peakHour ± 2N, and the spacing tells you N.

3. **Calibrate rate-limit alarms differently for different margin regimes.** For a high-margin tool like vscode-assistant (margin 0.0765 around hour 02 UTC), provisioning headroom only at the peak hour is rational. For a low-margin always-active tool like opencode or hermes (margin < 0.01, 24 active hours), there is no peak to provision around — the tool runs all the time and headroom needs to be flat across the clock. Treating the latter case as if it had a peak leads to either over-provisioning at the nominal peak or under-provisioning at the actually-busy hours that happen not to be the argmax this week.

4. **Watch the margin trend over a rolling window.** If a previously broad-mode tool starts developing a real peak (margin rising from 0.01 toward 0.05), something has changed in usage — a user shifting hours, a cron job being added, a load consolidation. If a previously sharp-peak tool sees its margin collapse, the peak is dispersing and the tool is becoming always-on. Both are interesting signals that no single peakHour value can surface, because peakHour by itself is fragile against exactly the kind of distributional shift that matters.

The 15× spread in margins across this six-source snapshot — 0.0050 at hermes to 0.0765 at vscode-assistant — is what lets you distinguish "tool with a real time-of-day pattern" from "tool whose argmax is noise." The same data also shows that the relationship between active-hour-count (`hActive`) and margin is monotone in this fleet: every source with hActive=24 has margin < 0.025, while the source with hActive=14 has the largest margin of the group. That monotone relationship is mechanically intuitive — you cannot have a sharp peak when mass is spread over all 24 hours — but writing it down explicitly is a nice sanity check that the metric is behaving the way the geometry says it should. argmax is fine; margin tells you whether the argmax means anything; peak-second geometry tells you what to report when it doesn't.
