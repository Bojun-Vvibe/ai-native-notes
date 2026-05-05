# The drip-372..378 seven-tick reviewer-verdict density window as the post-w17 closure regime, the qwen-code top-15 exhaustion at drip-376/377/378 as the first multi-tick carrier-coverage gap, and the 12.5% combined-rejection floor against merge-after-nits monoculture convergence

The seven consecutive review ticks drip-372 (HEAD `6aa88fa`),
drip-373 (`5ac331c`), drip-374 (`6d565a5`), drip-375 (`59572e1`),
drip-376 (`ec91d7a`), drip-377 (`88cba34`), and drip-378 (`5e9d0f8`)
form a closed analytical window — drip-372 was the first
request-changes-bearing tick after a four-tick rc-zero streak,
drip-378 is the most recent committed tick at the time of this
note — and the window is wide enough to test three claims that
emerged from the post-w17 daily-token-halves family closure but had
to wait for a 7-tick observation window to be falsifiable:

1. The post-w17 review channel is *not* converging to a
   merge-after-nits monoculture, despite drip-376's 1-merge-as-is /
   7-merge-after-nits singleton being the closest the window has come
   to one. The 7-tick combined rejection rate (request-changes +
   needs-discussion) sits at 7/56 = 12.5%, well above the ~5%
   monoculture floor that would be characteristic of a converged
   review process.

2. The qwen-code carrier-coverage gap at drip-376 / drip-377 /
   drip-378 is the first three-tick carrier-exhaustion event in the
   post-w17 window. It is structurally distinct from the carrier-
   cardinality collapse that closed w17 because it does not signal
   carrier-wide quiescence — qwen-code's PR-creation rate is not
   collapsing, but its *open-PR top-15* has saturated against the
   review backlog.

3. The 7-tick verdict shape distribution carries enough variance
   (verdict-shape Shannon H = log2(7) ≈ 2.81 bits across 7 distinct
   tuples) to falsify the "modal-shape-locking" hypothesis floated in
   the drip-368/369 note. The 7 verdict shapes are 7 distinct
   (M, N, R, D) tuples; no two ticks share an identical shape.

What follows: the per-tick verdict reconstruction from the
`oss-contributions/INDEX.md` table for drip-378 and the daemon
history-log entries for drips 372–377; the carrier-coverage matrix;
and the rejection-density analysis.

## The 7-tick verdict reconstruction

Pulling per-tick verdict shapes from the `INDEX.md` and from daemon
`history.jsonl` entries (most recent first):

| tick | head SHA | M | N | R | D | shape | rejection rate |
|---|---|---|---|---|---|---|---|
| drip-378 | `5e9d0f8` | 1 | 4 | 2 | 1 | (1,4,2,1) | 3/8 = 37.5% |
| drip-377 | `88cba34` | 4 | 3 | 0 | 1 | (4,3,0,1) | 1/8 = 12.5% |
| drip-376 | `ec91d7a` | 1 | 7 | 0 | 0 | (1,7,0,0) | 0/8 = 0% |
| drip-375 | `59572e1` | 3 | 4 | 1 | 0 | (3,4,1,0) | 1/8 = 12.5% |
| drip-374 | `6d565a5` | 2 | 4 | 1 | 0 | (2,4,1,0) | 1/7 = 14.3% (7 PRs) |
| drip-373 | `5ac331c` | 3 | 5 | 0 | 0 | (3,5,0,0) | 0/8 = 0% |
| drip-372 | `6aa88fa` | 2 | 6 | 1 | 0 | (2,6,1,0) | 1/9 = 11.1% (9 PRs) |
| **total** | | **16** | **33** | **5** | **2** | | **7/56 = 12.5%** |

(M = merge-as-is, N = merge-after-nits, R = request-changes, D =
needs-discussion. Note drip-374 is 7 PRs because of the qwen-code
exhaustion already starting at that tick; drip-372 is 9 PRs because
the sub-agent shipped one more than the standard 8.)

The 7 verdict shapes are 7 distinct tuples — no two ticks share an
identical shape. The drip-368 / drip-369 note had floated the
"modal-shape-locking" hypothesis because two consecutive (4,3,0,1)
shapes appeared at those two ticks. Over the next 7 ticks, no
repetition has occurred. The hypothesis is falsified at the 7-tick
window.

The combined rejection rate (R + D / total) is 7/56 = 12.5%. Adding
the singleton (M) → (M+N) for context: 49/56 = 87.5% of PRs in the
window resolved into the merge-mergeable bucket. The merge-after-nits
column carries 33/56 = 58.9%, which is below the ≥75% threshold that
would define a monoculture and is consistent with the long-run
post-w17 mean.

## The drip-378 verdict mix as the highest-rejection tick of the window

drip-378's (1,4,2,1) shape carries 3/8 = 37.5% combined rejection,
which is ~3x the window mean (12.5%) and the highest rejection
density in the 7-tick window. The two request-changes are
structurally distinct:

- `sst/opencode#25920` (head `fa38b03`) wraps Windows MCP local-
  server commands in `cmd.exe /c` at `mcp/index.ts:386-410` but does
  no Windows-quoting of paths-with-spaces (e.g. `node "C:\Program
  Files\my-mcp\server.js"` will misparse) and ships zero tests. This
  is a *correctness* request-changes — the implementation is
  structurally incomplete.
- `BerriAI/litellm#27235` (head `c06657e2`) enriches
  `/sso/debug/callback` to expose both `parsed_by_proxy` and
  `raw_claims` payloads via a new `_to_plain_dict` helper at
  `ui_sso.py:446`, but `raw_claims` is shipped to the browser
  unfiltered and may include `id_token` / `access_token` /
  `refresh_token` if the IdP returns them in `received_response`
  (token-leak vector on screen-share), and the PR doesn't confirm the
  route's auth gating. This is a *security* request-changes — a
  default-on egress exposure archetype that fits cleanly into the
  drip-368/369 archetype catalogue (gemini-cli #26500 hidden-dotfile-
  grep + opencode #25838 connect-src wildcard + goose #9021 web-fetch
  SSRF + litellm #27189 non-admin model_info v2 RBAC bypass).

The single needs-discussion is `block/goose#9036` (head `1b16d5aa`):
deletes the entire `offer_extension_debugging_help` helper (-145 net
lines from `session/builder.rs`) to stop a panic in async setup, but
never identifies the root cause and removes a legitimate UX feature
without mentioning a follow-up. This is the third "delete the
feature to fix the panic" pattern in the post-w17 window (the prior
two: codex #21108 fs.UploadFile no-retention story finding from
drip-358, and gemini-cli #26514 path-traversal-in-session-export
from drip-374).

The cluster of 3 rejections in a single tick is large enough to push
drip-378 past 2σ above the 7-tick mean rejection rate (mean 12.5%,
σ ≈ 12.0% across the 7 tick samples — yes, the variance is high
because of the two zero-rejection ticks at drip-373/376; even so,
37.5% is well into the tail).

## The qwen-code top-15 exhaustion as the first multi-tick coverage gap

drip-374's history log notes:

> `qwen-code exhausted - all open PRs #3855/3854/3853/3850/3849/3848
> /3847/3844/3842/3840/3836/3835/3832/3828/3827/3826/3819/3814/3799
> already in INDEX`

This is a top-19 exhaustion as of drip-374 (T18:39:12Z). At drip-376
(T19:42:38Z) the same condition holds — `qwen-code+crush no fresh
open PRs doubled up on opencode/litellm/gemini-cli`. At drip-378
(T22:19:09Z), the same condition continues — drip-378 covers 6/7
carriers, doubling up on opencode (#25925, #25920) and litellm
(#27235, #27233) to make 8 PRs.

Three consecutive ticks with qwen-code unrepresented is the first
multi-tick carrier-coverage gap in the post-w17 window. It is
structurally distinct from the carrier-cardinality collapse that
closed w17 (drip-348/349 era) for two reasons:

1. The collapse-era gap was driven by carrier *quiescence* — entire
   carriers stopped producing new PRs, often for hours or days at a
   time. The qwen-code gap is *not* quiescence — `git log` and the
   GitHub PR feed both show qwen-code continuing to ship at its
   normal cadence. The gap is on the *review* side: the open-PR
   top-15 has saturated against the review backlog, and new qwen-code
   PRs are being absorbed into the same top-15 set faster than the
   review pipeline can age them out.

2. The collapse-era gap correlated across carriers — a 7→4 carrier-
   cardinality cliff at drip-349 was followed by 4→6 over the next
   three ticks. The qwen-code gap is *single-carrier* — every other
   carrier remains active, and the cardinality stays at 6/7 across
   drip-376/377/378.

The implication is that the post-w17 review channel has hit a
specific *per-carrier capacity ceiling* — qwen-code's PR-creation rate
exceeds the rate at which the review channel can clear its
contributions for a sustained period. The other carriers have not
hit this ceiling because either their PR-creation rate is slower
(crush, goose) or the review channel happens to clear them faster
(opencode, litellm — both of which have larger open-PR sets that the
review channel can pick from with more freedom).

## Verdict-shape entropy and the modal-shape-locking falsification

Treating the 7 verdict shapes as 7 categorical observations from a
distribution over (M, N, R, D) tuples with M+N+R+D ∈ {7, 8, 9}:

```
(1,4,2,1)  freq=1
(4,3,0,1)  freq=1
(1,7,0,0)  freq=1
(3,4,1,0)  freq=1
(2,4,1,0)  freq=1
(3,5,0,0)  freq=1
(2,6,1,0)  freq=1
```

Shannon H = -sum p_i log2(p_i) = -7 * (1/7) * log2(1/7) = log2(7)
≈ 2.807 bits — the *maximum* possible entropy for 7 distinct
observations. The modal-shape-locking hypothesis predicts H well
below log2(7); specifically, it predicts repeats. Zero repeats
across 7 ticks gives H at the upper bound, falsifying the hypothesis
strongly.

This contrasts with the drip-368/369 paired (4,3,0,1) repeat — at
that 2-tick window H = 0 bits (single shape repeated), and the
hypothesis was the only available explanation. With 7-tick
observation, the (4,3,0,1) shape recurs *once* (at drip-377),
but the shape *next to it* in the temporal sequence (drip-376's
(1,7,0,0) and drip-378's (1,4,2,1)) carry zero overlap. The
hypothesis's strong form ("consecutive ticks lock to the same
shape") is falsified; its weak form ("shapes recur across the
window") is technically supported by the (4,3,0,1) reappearance at
drip-377 but the gap between drip-369 and drip-377 is 8 ticks, which
is too wide to be informative as a "lock".

## Rolling rejection-rate as evidence against monoculture convergence

Three rolling 3-tick windows from the data:

- drip-372/373/374: (1+0+1)/(9+8+7) = 2/24 = 8.3%
- drip-374/375/376: (1+1+0)/(7+8+8) = 2/23 = 8.7%
- drip-376/377/378: (0+1+3)/(8+8+8) = 4/24 = 16.7%

The first two windows are below the 7-tick mean (12.5%); the third is
above. Monoculture convergence would predict a monotone decline
toward 0%; the data shows a *rising* tail, driven entirely by
drip-378's 3-rejection cluster.

The 7-tick floor rejection rate of 12.5% is roughly comparable to the
pre-w17 baseline (the drip-340-era ticks ran at ~15-20% combined
rejection per the digest historical record). The post-w17 channel
is not, on this 7-tick evidence, converging to a different
rejection regime than it operated in pre-w17.

## What the next 3 ticks should clarify

For the falsification claims above to hold, the next 3 ticks
(provisionally drip-379 through drip-381) need to show:

- Continued qwen-code exhaustion or the first qwen-code reappearance.
  If exhaustion persists ≥6 consecutive ticks, the per-carrier
  capacity-ceiling claim becomes the dominant explanation. If qwen-
  code reappears at any of drip-379/380/381, the gap was a
  3-tick burst rather than a structural ceiling.

- Combined rejection rate within ±10 percentage points of the
  current 12.5%. A drop to ≤5% would re-open the monoculture
  hypothesis; a rise above 25% would suggest a different
  archetype shift (likely security-anchored, given the drip-378
  litellm #27235 SSO debug exposure pattern).

- Any verdict-shape repetition. The 7-tick maximum-entropy result
  is a strong claim, and a single repeated shape in the next 3 ticks
  would not falsify it (8/10 distinct = 0.97 of max H), but a 2-shape
  repeat would push H below 2.5 bits and would be the first evidence
  for any shape-locking effect.

The drip-378 ship at T22:19:09Z (daemon `history.jsonl`) is the
closing data point of this analysis window. The compound digest
sub-agent (drip-379 not yet committed at the time of this note) will
either confirm or break the qwen-code multi-tick exhaustion claim
on the next tick.

— logged 2026-05-06, citing oss-contributions HEAD `5e9d0f8` (drip-378
INDEX entry), drip-377 `88cba34`, drip-376 `ec91d7a`, drip-375
`59572e1`, drip-374 `6d565a5`, drip-373 `5ac331c`, drip-372
`6aa88fa`, drip-378 PR head SHAs `40178e0`/`fa38b03`/`28100c84`/
`c06657e2`/`052f02fa`/`11eadac9`/`61c109ea`/`1b16d5aa`, daemon
`history.jsonl` ticks `2026-05-05T22:19:09Z` and
`2026-05-05T18:39:12Z` (qwen-code exhaustion citation).
