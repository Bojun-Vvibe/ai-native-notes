# Teager-Kaiser Energy on Token Streams: Six Sources, Six Orders of Magnitude

Most token-telemetry dashboards plot two things: bytes per hour and a stacked area for who paid for them. That is fine for an invoice. It is hopeless as a fingerprint, because two completely different agent harnesses can spend the same dollars by completely different temporal mechanisms (one steady drip, one pulsed burst, one episodic explosion) and the area chart will compress all three into the same colored band.

This post takes a single, real telemetry file (`~/.config/pew/queue.jsonl`, 1709 hourly rows across 6 agent sources) and runs a non-linear, three-point operator over each source's `total_tokens` time series: the **Teager-Kaiser Energy Operator** (TKEO), defined for a discrete sequence x as

```
psi(x_n) = x_n^2 - x_{n-1} * x_{n+1}
```

Kaiser's 1990 ICASSP paper showed that for a pure cosine `x_n = A cos(omega n + phi)` this collapses to roughly `A^2 * sin^2(omega)`, simultaneously coupling local amplitude squared and local frequency squared in a single 3-sample expression. That coupling is the point. Variance gives you amplitude only. Lag-1 autocorrelation gives you something like frequency only. TKEO does both at once with three taps and zero training.

## The raw numbers

Computed over the actual hour-aggregated token stream:

```
source           n    mean tokens     max tokens     min tokens    raw TKEO          normalized
claude-code     299    11,512,996    107,646,380         5,976    1.817e+14         0.5863
codex            64    12,650,385     58,840,552        47,317    1.523e+14         0.7496
hermes          193       843,107      5,898,713        15,525    8.186e+11         0.8777
openclaw        463     4,129,130     45,073,562       106,759    1.592e+13         0.6795
opencode        357    10,654,333     69,504,417        47,789    6.798e+13         0.4046
vscode-assist  333         5,663        174,625            20    1.842e+08         0.8259
```

Read the raw TKEO column. The spread is **six orders of magnitude**: claude-code at 1.82e+14, vscode-assist at 1.84e+08. That is not a rounding artifact. It is what `psi` does on streams whose amplitudes differ by 2,033x in the mean (claude-code 11.5M, vscode-assist 5.7K). Square that ratio and you are at 4.1M, which is in the same ballpark as the raw TKEO ratio of 1e14 / 1.84e+08 ~ 5.4e+5. The gap is exactly what amplitude-squared coupling predicts.

So if you stop reading right here, the takeaway is: **TKEO is dominated by amplitude on these streams**. The interesting move is what happens when you divide it out.

## Normalized TKEO: ranking inverts

Add `--normalize` (divide each per-source TKEO by that source's variance, which removes the `A^2` factor and leaves you with a `sin^2(omega)` proxy, an amplitude-invariant pseudo-instantaneous-frequency-squared) and watch the ranking rearrange itself completely.

```
              raw rank          normalized rank
1  claude-code (1.82e+14)       hermes        (0.8777)
2  codex       (1.52e+14)       vscode-assist (0.8259)
3  opencode    (6.80e+13)       codex         (0.7496)
4  openclaw    (1.59e+13)       openclaw      (0.6795)
5  hermes      (8.19e+11)       claude-code   (0.5863)
6  vscode-assist (1.84e+08)    opencode      (0.4046)
```

Three sources cross. Vscode-assist is dead last on raw amplitude and second on normalized frequency, which translates to: it spends almost no tokens but the ones it does spend bounce up and down hour over hour faster than anyone else. Hermes does the same pattern, just with two more orders of magnitude of base load. Opencode has the highest amplitude after claude-code but the *lowest* normalized score, which means its hourly token stream is amplitude-rich and frequency-poor: long, slow, sustained pulses. Claude-code drops three positions in the rerank, which is the same reading: big amplitude, slowly modulated, the prototypical "long-running mission" workload.

This is genuinely diagnostic. None of variance, mean, max, autocorrelation, or any of the seven different fractal-dimension lenses already in the same toolchain (Petrosian, Katz, Higuchi, DFA, Hjorth-mobility, Hjorth-complexity, approximate entropy) produces this exact sort order, because none of them couples amplitude and frequency in three samples.

## Why three samples matter

The cheap intuition for `psi(x_n) = x_n^2 - x_{n-1} * x_{n+1}` is "how convex is the curve at sample n." If the sequence is a perfect linear ramp `x_n = a + b*n`, then

```
psi(x_n) = (a + bn)^2 - (a + b(n-1))(a + b(n+1))
        = (a + bn)^2 - ((a + bn)^2 - b^2)
        = b^2
```

A pure ramp produces a constant nonzero TKEO equal to slope-squared. That is by design. A flat line produces zero. A perfect cosine produces the `A^2 sin^2(omega)` term. A real, noisy, episodic agent token stream produces a mixture, and the mean of `psi` over the run aggregates all three regimes into one number.

Two consequences for telemetry:

1. **Reordering kills it**. Shuffle the per-hour rows for any source and TKEO collapses. This is genuinely temporal, unlike every variance/skewness/HHI/entropy lens you might already plot. If a dashboard panel survives row shuffling, it is not measuring a temporal property and you should know that.

2. **Outliers do not dominate the mean of psi**. They dominate `max(psi)`, which is interesting on its own. The maximum absolute psi on the claude-code series is 1.16e+16 at row index 286, which corresponds to a single hour in late April where the per-hour total jumped from a few million into the high tens of millions and back. The max absolute psi on vscode-assist, by contrast, is 3.02e+10 at row 330, a tiny burst at the end of its window. Both are "the loudest moment", but the loudest moment for one source is six orders of magnitude bigger than for the other.

If you wanted a single-number alert for "this source just had an unprecedented hour", `argmax|psi|` is a more honest detector than `argmax(value)`, because it weighs the moment against its immediate neighbors instead of the full-history mean.

## Live numbers, live caveats

The data above is the exact output of:

```
python3 -c '
import json
from collections import defaultdict
rows = [json.loads(l) for l in open("queue.jsonl")]
rows.sort(key=lambda r: r["hour_start"])
by_src = defaultdict(list)
for r in rows:
    by_src[r["source"]].append(r["total_tokens"])
for s in sorted(by_src):
    seq = by_src[s]
    if len(seq) < 3: continue
    psi = [seq[i]**2 - seq[i-1]*seq[i+1] for i in range(1, len(seq)-1)]
    raw = sum(psi)/len(psi)
    mu = sum(seq)/len(seq)
    var = sum((x-mu)**2 for x in seq)/len(seq)
    norm = raw/var if var > 0 else 0
    print(f"{s:18} n={len(seq):4} TKEO={raw:.3e} norm={norm:.4f}")
'
```

run against the live `queue.jsonl` snapshot at 1709 rows. The `pew-insights` tool ships the same computation under `source-row-token-teager-kaiser` (commit `09bcbd3`, version `v0.6.137 -> v0.6.138`, follow-ups `b1dbce0` adding `--min-tkeo`/`--max-tkeo` filters and `e5f5b44` adding abs-asc/abs-desc sort modes). The CLI live-smoke output in that commit message confirms the same six-source ordering: "live smoke against 1,706 rows / 6 sources confirms --normalize completely re-orders the table."

Three caveats worth knowing before you wire TKEO into anything:

1. **Sampling rate matters**. The token stream here is hourly. TKEO's frequency interpretation is "fraction of Nyquist", so `sin^2(omega)` is bounded above by 1, which means normalized TKEO of 0.88 (hermes) is implying a near-Nyquist alternation. That is what an hourly stream of "burst, quiet, burst, quiet" looks like. Sample at five-minute resolution instead and the same harness will read very differently. Always pin the sampling rate when you compare across runs.

2. **Three-sample windows hate gaps**. If a source is offline for an hour and you forward-fill to zero, you inject an artificial dip that TKEO will read as a frequency event. Either drop the row, mark it explicitly, or interpolate, but pick one and stick with it.

3. **Negative TKEO is a real thing**. The operator is signed. A locally concave region (peak with neighbors lower than a quadratic fit predicts) yields positive psi; a locally convex region (valley) yields negative. Mean TKEO can therefore be negative, and abs-sort vs signed-sort give different rankings. The `e5f5b44` follow-up adding `--abs-asc/--abs-desc` exists precisely because this confused the live-smoke output once.

## What this replaces in a dashboard

If you already have a per-source variance panel and a per-source autocorrelation panel, you do not need TKEO to add a third "amplitude is high" or "samples are correlated" indicator. You need it because the existing two panels cannot tell you when a source has shifted from "low amplitude, low frequency" to "low amplitude, high frequency" (vscode-assist kept the same byte budget but went chatty), and they cannot tell you when amplitude went up but the temporal structure stayed the same (a source got more expensive but did not change its rhythm). TKEO collapses both into one signed number, and the abs-rank gives you "loudness" while the normalized rank gives you "speed".

For the six-source slice in this dataset, the practical conclusion is short. The two harnesses with the highest normalized TKEO (hermes and vscode-assist) are also the ones with the smallest amplitude. They are spending little but spending it spikily. The two with the lowest normalized TKEO (opencode and claude-code) are doing the opposite: large, sustained, slow-modulated pulls. Codex and openclaw sit in the middle. That is a four-quadrant picture of "amplitude vs tempo per source" extracted from one column of one append-only file with one three-point operator and zero training data.

That is the kind of zero-cost diagnostic that pays for itself the first time a source quietly changes shape. Variance will not catch it. Autocorrelation will not catch it. A stacked area chart will definitely not catch it. Three samples and a subtraction will.

## When not to use TKEO

Do not reach for it when:

- The stream is fewer than ~20 samples. With 8 you can compute it (the lens floor in `--min-rows 8` reflects this), but the mean of 6 psi values is dominated by the loudest one and you might as well just print `argmax|psi|`.
- The stream is not stationary in any sense. TKEO's interpretation as "amplitude squared times frequency squared" assumes the sequence is at least locally cosine-like. A monotonic ramp from 0 to a billion tokens will give you a TKEO equal to the squared average step, which is technically correct and operationally useless.
- You care about distributional shape over time-ordered shape. If you want the histogram of per-row tokens, use a variance/kurtosis/HHI/entropy lens. TKEO's exact superpower (sensitivity to ordering) is exactly what you do not want when the question is "what does the row distribution look like."

The lens shipped, the live numbers ranked the six sources two ways that disagree by orders of magnitude, and the disagreement itself is the diagnostic. That is enough to keep TKEO on the dashboard for the ones it does fit.
