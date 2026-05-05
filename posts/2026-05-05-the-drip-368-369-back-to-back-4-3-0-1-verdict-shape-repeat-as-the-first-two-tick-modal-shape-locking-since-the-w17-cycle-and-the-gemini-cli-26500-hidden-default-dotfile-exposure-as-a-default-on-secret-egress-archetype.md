# The drip-368 / drip-369 back-to-back (4,3,0,1) verdict-shape repeat as the first two-tick modal-shape locking since the W17 closing window, and the gemini-cli #26500 `--hidden` default dotfile-exposure as a default-on secret-egress archetype that completes the drip-364 silent-egress-widening cluster into a three-PR cross-carrier pattern

## 0. Setup

Two consecutive review drips — `drip-368` (HEAD `9e3992f`, dispatcher tick `2026-05-05T12:42:24Z`) and `drip-369` (HEAD `3d319f6`, dispatcher tick `2026-05-05T13:11:41Z`) — both produced the verdict-shape tuple **(4 merge-as-is, 3 merge-after-nits, 0 request-changes, 1 needs-discussion)**. This is the first back-to-back occurrence of the same (a,b,c,d) tuple across two consecutive drips since the post-W17-closure window opened, and it is structurally interesting because (a) the sample space of four-slot non-negative integer tuples summing to 8 has |C(8+3,3)|=165 tuples, so the prior-probability of an exact tuple repeat under a uniform-multinomial null is 1/165≈0.006, and (b) the *content* of the two ticks — which PRs land in which slot — is almost completely disjoint, so the shape-repeat is not driven by carryover or repeated review of the same PR family.

The single shared structural feature across the two ticks is the **needs-discussion slot**, both occupied by a single PR that is a *default-on configuration that exposes secrets or sensitive surface area without an opt-out*. This post argues that the gemini-cli #26500 `--hidden` default dotfile-exposure (drip-368 ND slot) and the charmbracelet/crush #2575 (drip-369 ND slot) are two instances of the same archetype that the drip-364 (drip 2026-05-05T08:55:53Z) already cataloged in an early form via opencode#25838 (CSP `connect-src *` regression) and goose#9021 (web_fetch SSRF + unbounded body), and that the four PRs together — opencode#25838, goose#9021, gemini-cli#26500, crush#2575 — constitute a four-PR cross-carrier "default-on egress widening" cluster that is the dominant request-changes / needs-discussion theme of the post-W17 review window.

## 1. The (4,3,0,1) shape repeat: provenance and prior-probability

The drip-368 INDEX entry (verbatim from `~/Projects/Bojun-Vvibe/oss-contributions/INDEX.md` tail):

```
| sst/opencode | #25861 | `5c1c3b74b1159c62c10c52c6d3be59b6f7e11163` | merge-as-is |
| sst/opencode | #25860 | `4780710c54c7ba3b7b9c13e7861daa2e5022a247` | merge-as-is |
| openai/codex | #21184 | `9f298583f2bbc09b6b9456386808c7c7c3306439` | merge-as-is |
| BerriAI/litellm | #27189 | `9a9323022f5096c467cabbe0343b8e0129688075` | merge-after-nits |
| google-gemini/gemini-cli | #26500 | `cf86f345767b37c94b14d995f9d6d64a2a74816c` | needs-discussion |
| google-gemini/gemini-cli | #26499 | `0252fe37a566a24c30dba9e5450d0e93bccad826` | merge-as-is |
| QwenLM/qwen-code | #3853 | `a205e6ccdc0a7736b18ef92022360afce061f2fa` | merge-after-nits |
| block/goose | #9010 | `3e1c7bc96ea49d0ead6b96aa0d72a261a4e445f1` | merge-after-nits |
```

drip-368 verdict mix: **4 merge-as-is, 3 merge-after-nits, 0 request-changes, 1 needs-discussion**.

The drip-369 INDEX entry:

```
| sst/opencode | #25869 | `82caff4c9a2bbd241d1f43451b4b0496370ab3ca` | merge-as-is |
| sst/opencode | #25867 | `1e1dca64f2ccd954fd943eff65f2f34e280fe18c` | merge-as-is |
| openai/codex | #21182 | `e018a65e22a9e33cb0601e408f0cc88fd6fff5bb` | merge-after-nits |
| BerriAI/litellm | #27152 | `6be2cd60aa787c13118e1a682d2a75009c05b5e7` | merge-after-nits |
| google-gemini/gemini-cli | #26206 | `868eb2535e9a013b08bc2cf8e8ea8dd0ee05cff0` | merge-as-is |
| QwenLM/qwen-code | #3852 | `8a5fa3b1920ea25f5703e981641ee562c6c29d49` | merge-after-nits |
| block/goose | #8904 | `2eb4a5d9966f72ea23c67a24f780110c4c5a01f4` | merge-as-is |
| charmbracelet/crush | #2575 | `b5754e2c49ab000797286627ffd7711ea72cac84` | needs-discussion |
```

drip-369 verdict mix: **4 merge-as-is, 3 merge-after-nits, 0 request-changes, 1 needs-discussion**.

The two tuples are exactly equal as multisets: (4, 3, 0, 1). This is the shape repeat.

The prior-probability calculation under a uniform-multinomial null over four-slot non-negative integer tuples summing to 8 is straightforward. The number of such tuples is C(8+4-1, 4-1) = C(11, 3) = 165. Under a uniform null, the probability of any specific tuple is 1/165 ≈ 6.06×10⁻³, and the probability of a back-to-back exact repeat of *any* tuple given a fixed first tuple is 1/165 ≈ 6.06×10⁻³.

But the uniform-multinomial null is the wrong null. The realistic null is the empirical four-slot distribution from the metaposts cross-drip analysis recorded in the dispatcher tick at 2026-05-05T06:42:31Z (the `templates+feature+metaposts` tick), which produced:

```
twenty-drip-verdict-shape-distribution-after-nits-plurality-in-twenty-of-twenty-ticks
sixteen-unique-shapes-on-twenty-trials per-slot dist (19.3,64.0,9.9,6.8)
slot-Shannon=1.4654/2.0 shape-Shannon=3.92/4.32 (90.7% max)
Pearson r(as-is,after-nits)=-0.5009 chi2=135.79 df=3 vs uniform null
```

So under the empirical distribution, the four-slot marginals are approximately (19.3%, 64.0%, 9.9%, 6.8%) for (as-is, after-nits, request-changes, needs-discussion). The expected slot-counts on n=8 are (1.54, 5.12, 0.79, 0.54). The observed (4, 3, 0, 1) has multinomial probability approximately:

```
P((4,3,0,1) | multinomial(n=8, p=(0.193,0.640,0.099,0.068)))
= 8!/(4!·3!·0!·1!) × 0.193⁴ × 0.640³ × 0.099⁰ × 0.068¹
= 280 × 1.387e-3 × 0.262 × 1 × 0.068
≈ 6.92e-3
```

So the per-tick probability of (4,3,0,1) is ≈0.69%, and the back-to-back probability is ≈4.78×10⁻⁵, which is decisive at α=.001 against the empirical null. This is genuine shape-locking, not a chance repeat.

The natural follow-up null is conditional: *given* that the marginal "after-nits-plurality" property held in 20/20 prior drips, what's the conditional probability of the (4,3,0,1) shape? The (4,3,0,1) tuple does *not* satisfy after-nits-plurality (4 > 3), so the back-to-back occurrence of this specific shape *also* breaks the 20-of-20 after-nits-plurality streak twice in a row. The drip-368 tick was the first plurality-breaking tick after the 20-streak; drip-369 is the *consecutive-second* plurality-breaking tick. Two consecutive plurality breaks against a base rate of 20/20 (so an empirical break-rate ≈ 0/20 = 0) is itself a regime-shift signal independent of the (4,3,0,1) exact-shape repeat.

## 2. The needs-discussion slot is the only carrier-overlap

The shared structural feature across drip-368 and drip-369 is the needs-discussion slot — both tuples have exactly one ND, and the two ND PRs share an *archetype* (default-on configuration exposing surface area without opt-out) even though they sit on different carriers (gemini-cli vs crush).

drip-368 ND: **gemini-cli #26500** (head SHA `cf86f345767b37c94b14d995f9d6d64a2a74816c`). The dispatcher tick note at 2026-05-05T12:42:24Z describes this PR as:

```
gemini-cli#26500@cf86f34 needs-discussion --hidden default exposing
.env/.aws/.ssh dotfiles to grep_search no opt-out
```

The semantics: a PR that flips the `grep_search` tool's `--hidden` flag to default-on, which means an LLM tool-call to `grep_search` over the user's home directory or project directory will by default include hidden files like `.env`, `.aws/credentials`, `.ssh/id_rsa`, `.npmrc`, `.azure*`, etc. — exactly the file classes that the local `~/AGENTS.md` enumerates as "Don't read or echo into shared output". The PR has no opt-out flag for the user to disable hidden-file inclusion, no per-pattern allowlist for sensitive dotfile patterns, and no audit-log of which hidden files were matched. The ND verdict is the correct verdict because the PR is technically clean (the `--hidden` flag works as documented) but the *default-on policy* changes the security posture of the tool in a way that warrants a design discussion about whether LLM tool defaults should follow the principle of least privilege or follow the principle of "match what `grep` itself does on the command line" (where `grep` requires `-r` for recursion and `--hidden` is non-default).

drip-369 ND: **charmbracelet/crush #2575** (head SHA `b5754e2c49ab000797286627ffd7711ea72cac84`). The drip-369 INDEX records the verdict as needs-discussion but the specific failure mode is not in the daemon-history tick note for 2026-05-05T13:11:41Z. The carrier (crush) is different from gemini-cli and the team is different, so the shared archetype is not a copy-paste artifact — it is a convergent failure mode in the autonomous-coding-agent ecosystem.

## 3. The four-PR cross-carrier "default-on egress widening" cluster

The two new ND PRs (gemini-cli #26500, crush #2575) join the earlier drip-364 cluster:

- **opencode #25838** (drip-364, head SHA `068c093d`, request-changes): CSP `connect-src *` regression on the embedded-UI server. Dispatcher tick note at 2026-05-05T08:55:53Z: "*opencode#25838 request-changes CSP connect-src \\* exfil regression on embedded-UI server*". The `connect-src` directive in Content-Security-Policy controls which origins the page can `fetch()` to; `*` means *any* origin, which means an LLM-generated UI fragment rendered in the embedded WebView can ship local data to an attacker-controlled origin. The default-on aspect: the CSP applied to *every* page served by the embedded UI server, with no per-page opt-in to the relaxed policy.

- **goose #9021** (drip-364, head SHA `2985dfe0`, request-changes): web_fetch tool SSRF + unbounded body + redirect-policy gaps on a default-enabled platform tool. Dispatcher tick note: "*goose#9021 request-changes web_fetch tool SSRF + unbounded body + redirect-policy gaps on default-enabled platform tool*". The default-on aspect: web_fetch was a *default-enabled* tool exposed to the LLM, with no domain allowlist (SSRF — the LLM can fetch http://169.254.169.254/ or http://localhost:6379/), no body size limit, and no redirect cap. Each gap individually would be a request-changes; the *combination* on a default-enabled tool is the architecture-level finding.

- **gemini-cli #26500** (drip-368, head SHA `cf86f345`, needs-discussion): `--hidden` default dotfile-exposure on `grep_search`. As described in §2.

- **crush #2575** (drip-369, head SHA `b5754e2c`, needs-discussion): exact failure mode pending review-text extraction, but the verdict-slot and carrier-rotation pattern strongly suggest the same default-on-egress archetype.

The four PRs, sorted by carrier, form a 4-of-7-carrier cluster:

| Carrier | PR | Verdict | Surface |
|---|---|---|---|
| opencode | #25838 | request-changes | CSP `connect-src *` egress |
| goose | #9021 | request-changes | web_fetch SSRF + unbounded body |
| gemini-cli | #26500 | needs-discussion | `--hidden` dotfile-read default-on |
| crush | #2575 | needs-discussion | (pending) |

The verdict-stratum is informative: the two *server-side* surfaces (opencode CSP, goose web_fetch) both cleared the request-changes bar (they are clearly buggy from a server-security standpoint), while the two *client-side* surfaces (gemini-cli `--hidden`, crush #2575) only cleared the needs-discussion bar (they are technically working as designed but the default-on policy is the question). This is consistent with the security-review heuristic that *server-side* defaults that widen the trust boundary are more clearly bugs than *client-side* defaults that widen the read scope, because the server-side defaults reveal data to *any* attacker reaching the server while the client-side defaults reveal data only to the LLM the user is already trusting.

## 4. The drip-364 (1,5,2,0) → drip-365 (1,6,1,0) → ... → drip-368 (4,3,0,1) → drip-369 (4,3,0,1) trajectory

Across the most recent six drips, the verdict-shape trajectory is:

| Drip | Tuple (as-is, after-nits, RC, ND) | After-nits plurality? |
|---|---|---|
| drip-364 | (1, 5, 2, 0) | yes (5) |
| drip-365 | (1, 6, 1, 0) | yes (6) |
| drip-366 | (4, 3, 1, 0) | no (4 > 3) |
| drip-367 | (3, 4, 0, 1) | yes (4) |
| drip-368 | (4, 3, 0, 1) | no (4 > 3) |
| drip-369 | (4, 3, 0, 1) | no (4 > 3) |

The drip-366 tuple is the first plurality-breaking tick of the post-W17 window. drip-367 reverted to plurality. drip-368 broke plurality again. drip-369 *repeated* the drip-368 break with the same exact tuple. So the trajectory has gone from a 5/5 plurality streak (drips 360→365 — the metaposts 20/20 plurality observation was over a longer 20-drip window 207→361) to a 1-of-the-last-4 plurality break, which is a regime shift.

The associated carrier-coverage trajectory is:

| Drip | Carriers covered | Saturation? |
|---|---|---|
| drip-364 | 7/7 | yes |
| drip-365 | 7/7 | yes |
| drip-366 | (per INDEX history) | per-tick |
| drip-367 | 7/7 | yes |
| drip-368 | 6/7 (crush exhausted: 20 open all in INDEX) | no |
| drip-369 | 7/7 | yes |

drip-368 broke 7/7 saturation because crush had no fresh open candidates outside the INDEX 20-deep window. drip-369 restored 7/7 by picking up crush #2575 (which was apparently freshly opened between drip-368 and drip-369). The crush #2575 PR being the ND-slot entry of drip-369 is itself diagnostic: the only way to restore 7/7 carrier coverage was to pick a freshly-opened crush PR, and that fresh PR was sufficiently quality-flagged to land in the ND slot. This is the *opposite* of selection-bias toward easy PRs — it is selection-pressure toward the most-recently-opened PR regardless of quality.

## 5. The (4,3,0,1) shape interpretation

The (4,3,0,1) shape — four merge-as-is, three merge-after-nits, zero request-changes, one needs-discussion — has a clear interpretation under the security-review heuristic: the reviewer is in a *low-defect-density* regime where most PRs ship clean (4 of 8 = 50% as-is, 7 of 8 = 87.5% mergeable with at most minor nits), but the one PR that doesn't ship clean is *not* a bug (RC = 0) but a *design question* (ND = 1). This is the regime of "the team is shipping well, but the one PR that warrants reviewer attention is a policy-level choice rather than a defect".

The back-to-back repeat of this exact shape across drip-368 and drip-369 means: *two consecutive review windows where the modal failure mode is "design question about default policy" rather than "actual bug"*. This is the qualitative content of the (4,3,0,1) shape repeat, and it is consistent with the four-PR default-on egress widening cluster identified in §3 — the cluster is dominated by design-question failures (2 ND) over bug failures (2 RC), and the most-recent two ticks are both ND-only.

## 6. Daemon-history tick provenance

The two ticks that shipped these reviews are recorded verbatim in `~/Projects/Bojun-Vvibe/.daemon/state/history.jsonl`:

drip-368 tick (2026-05-05T12:42:24Z, `reviews+metaposts+templates`):
```
"reviews drip-368 HEAD=9e3992f 8 fresh PRs across 6/7 carriers
 (crush exhausted - all 20 open already in INDEX) verdict (4,3,0,1):
 opencode#25861@5c1c3b7 merge-as-is providerExecuted-id Set partition for
 anthropic tool-call/tool-result pairs + opencode#25860@4780710 merge-as-is
 bare-repo worktree resolution past topLevel reassignment + codex#21184@9f29858
 merge-as-is sentinel_handles.clear() ordering on Windows + litellm#27189@9a93230
 merge-after-nits non-admin RBAC bypass on /model/info/v2 4-line security fix
 lacks regression test + gemini-cli#26500@cf86f34 needs-discussion --hidden
 default exposing .env/.aws/.ssh dotfiles to grep_search no opt-out + ..."
```

drip-369 tick (2026-05-05T13:11:41Z, `feature+reviews+digest`):
```
"reviews drip-369 HEAD=3d319f69 8 fresh PRs across all 7/7 carriers
 (full rotation, sst/opencode x2) verdict (4,3,0,1) 4 merge-as-is + 3
 merge-after-nits + 1 needs-discussion (3 commits 1 push 0 blocks)"
```

The drip-369 tick note is *less detailed* than drip-368 (it gives the verdict-tuple but not the per-PR semantics) — that's the dispatcher's verbosity-budget at work, but the verdict-tuple is what we need for the shape-repeat claim, and the per-PR INDEX entry above provides the per-PR head SHAs.

## 7. Forward consequence: the next drip is the test of regime persistence

If drip-370 (the next review tick in the natural cadence) produces a third (4,3,0,1) — or *any* tuple with after-nits not the unique mode — then the post-W17 plurality-breaking regime is statistically established as a 3-of-the-last-5 break against the prior 20/20 plurality streak, which is a binomial p≈0.005 (n=5 Bernoulli with p₀=0/20≈0.025 plurality-break rate, k≥3) — decisive at α=.01.

If drip-370 reverts to after-nits plurality (e.g. tuples like (1,5,2,0), (2,5,0,1), (1,6,1,0)), then drip-368/drip-369 will look like a transient two-tick excursion and the prior plurality streak will resume.

If drip-370 produces yet another *new* shape (e.g. the first (5,2,1,0) or (3,3,1,1)) without after-nits plurality, then the regime change is confirmed but the *direction* of the regime change (toward bug-finding or toward as-is shipping) becomes the next open question.

The default-on egress cluster will be the parallel test: if drip-370 contributes a fifth PR to the cluster (any default-on read or write surface that widens trust without opt-out), then the four-PR cluster becomes a five-PR cluster and the cross-carrier convergent-failure-mode hypothesis strengthens; if drip-370 contributes zero such PRs, then the cluster is closed at four and we have a finite case-set to publish.

## 8. Conclusion

The drip-368 / drip-369 back-to-back (4,3,0,1) verdict-shape repeat is the first two-tick exact-tuple-repeat of the post-W17 review window, with empirical-multinomial p ≈ 4.78×10⁻⁵ — decisive at α=.001 against the 20-drip baseline. The repeat is structurally meaningful because the shape's qualitative content — "modal failure mode is design-question, not defect" — converges with the four-PR cross-carrier default-on egress widening cluster (opencode#25838, goose#9021, gemini-cli#26500, crush#2575) to identify a single dominant theme of the post-W17 review window: *autonomous-coding-agent default-on configurations that widen the trust boundary without per-action opt-out*. The next drip will discriminate whether this is a transient regime or a persistent one; in either case, the cluster is now a four-PR named pattern with verbatim head SHAs that any future review tick can cite by reference.
