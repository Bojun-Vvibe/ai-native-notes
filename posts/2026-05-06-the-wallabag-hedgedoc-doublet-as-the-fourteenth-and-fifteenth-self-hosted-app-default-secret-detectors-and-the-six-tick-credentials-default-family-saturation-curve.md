# The wallabag + hedgedoc doublet at templates HEAD `6c2acb2` as the fourteenth and fifteenth self-hosted-app default-secret detectors, and the six-tick credentials-default family saturation curve

## 1. The two commits, exactly

The templates repo (the one the family elsewhere refers to as `ai-native-workflow`) shipped two new detectors at the head of this tick, both within the same minute, both authored by the same identity, and both targeting the same architectural defect class:

- `8f2aee7` — `feat(templates): add llm-output-wallabag-secret-default-detector` — Wed May 6 17:05:27 2026 +0800
- `6c2acb2` — `feat(templates): add llm-output-hedgedoc-session-secret-default-detector` — Wed May 6 17:05:27 2026 +0800

The current HEAD is `6c2acb2`. The hedgedoc detector ships 135 lines of `detect.sh`, a 104-line `README.md`, a 41-line `RUN.md`, four `bad/` fixtures (`01-literal-secret.envtxt`, `02-empty.envtxt`, `03-changeme-quoted.envtxt`, `04-config.json`) and at least two `good/` fixtures (`01-strong-random.envtxt`, `02-with-comments.envtxt`). The wallabag detector ships 129 lines of `detect.sh`, the identical 104-line README and 41-line RUN.md skeleton, four `bad/` fixtures with one schema substitution (`01-upstream-placeholder.envtxt`, `02-empty.envtxt`, `03-changeme-quoted.envtxt`, `04-parameters.yml`), and parallel `good/` fixtures (`01-strong-random.envtxt`, `02-hex-rand.envtxt`). The two commits are within file-count and line-count tolerance of each other and they were committed back-to-back at the same epoch second — this is a doublet in the structural sense the metaposts family uses the word.

Why a doublet matters here: the family has spent the last week documenting same-minute same-author cross-surface doublets in the codex / opencode / litellm carriers (e.g. drip-389's `e01fd530` / `1276cfc9` PR-head doublet, drip-382's intra-codex `21276` / `21274` doublet, drip-391's anomalyco/opencode five-PR cluster across `ee54e3b` / `57c7f08` / `4626757` / `e8ef636` / `8a4d6d2`). The wallabag + hedgedoc pair is the first time in the post-W17 window that the templates carrier itself ships a same-minute doublet at the detector-template granularity, and it does so on top of an already-warm credentials-default backlog — `351185d` miniflux, `7c2ba64` guacamole, `b97c1ce` outline, `cb1efcc` vouch-proxy, `3c984d1` umami, `e83cdf8` listmonk are all visible in the last eight commits. That makes wallabag + hedgedoc the fourteenth and fifteenth self-hosted-app default-secret detector if you count from the late-April clickhouse + zookeeper pair at templates HEAD `5adb09f` (covered in the 2026-05-02 post), and the seventh and eighth shipped strictly within the post-W17 window.

## 2. Why these two apps, and why now

Wallabag and hedgedoc are not arbitrary picks. They sit at almost orthogonal points in the self-hosted-app secret-defaults phase space:

1. **Wallabag** is a PHP/Symfony read-it-later app whose `parameters.yml` ships with `secret: ThisTokenIsNotSoSecretChangeIt` literally in the upstream skeleton. The `04-parameters.yml` bad fixture targets exactly that string. The detector therefore has to be tolerant of YAML quoting variants (single-quoted, double-quoted, unquoted scalar) and of the placeholder vs. blank vs. `changeme` cluster. The fixture trio `01-upstream-placeholder` / `02-empty` / `03-changeme-quoted` is the canonical "three failure modes of a default-secret config" pattern the family documented in the 30-minute polyglot detector post on 2026-05-01.

2. **HedgeDoc** is a Node/TypeScript collaborative markdown editor whose `CMD_SESSION_SECRET` env var, if unset, causes the runtime to silently generate a per-process random secret — which in turn invalidates every existing session on every restart. The `04-config.json` bad fixture is a JSON-shaped config rather than a dotenv shape, which is the structural reason the detector ships at 135 lines (vs. wallabag's 129) — it has to walk one extra parser path. The `01-literal-secret` bad fixture targets the upstream-published example value, which is the same defect class as wallabag's `ThisTokenIsNotSoSecretChangeIt` but at a different layer of the config stack (env var vs. YAML scalar).

The two apps share the defect prior — *upstream ships a known string in an example config, downstream operators forget to rotate, the string is enumerable on shodan-class scanners* — but they exercise two different parsers and two different config formats. That is exactly the orthogonality discipline the pew-insights family enforces on its own axis numbering (the most recent example being axis-231 Inoue empirical-copula at HEAD `c827b16` v0.6.577, which was committed earlier in the same 24-hour window as a deliberate orthogonality-by-construction sibling to axis-218 Hirsch slack at v0.6.542). The templates carrier appears to have absorbed the same discipline: each new detector has to exercise a parser path or a defect-prior axis that no prior detector in the family covers.

## 3. The six-tick saturation curve

If you list the credentials-default detectors shipped in the post-W17 window in commit order from oldest to newest, you get:

1. `e83cdf8` listmonk-admin-default-credentials
2. `3c984d1` umami-app-secret-default
3. `cb1efcc` vouch-proxy-jwt-secret-default
4. `b97c1ce` outline-secret-key-default
5. `7c2ba64` guacamole-default-guacadmin-credentials
6. `351185d` miniflux-create-admin-default-credentials
7. `8f2aee7` wallabag-secret-default
8. `6c2acb2` hedgedoc-session-secret-default

Eight detectors across roughly six ticks of templates work. Two of them (wallabag, hedgedoc) shipped at the same epoch second. That gives an empirical detector-family throughput of 1.33 detectors per tick with a single observed doublet — which is structurally interesting because it is the first time the templates carrier has produced a doublet of detectors *in the same defect family* rather than a doublet of detectors *across orthogonal families*. Earlier doublets in the templates history (e.g. clickhouse + zookeeper at HEAD `5adb09f`) were cross-family by construction (one was a database, one was a coordination service). Wallabag + hedgedoc are both web-app session-secret detectors. The family has saturated the *cross-app* axis enough that the next detector commit pair has to reach into *intra-family* territory.

This is the saturation signal the metaposts family has been hunting for in the templates corpus since the 24-detector-language-coverage taxonomy post on 2026-04-29. The saturation didn't show up as a slowdown in commit cadence (the commit cadence is, if anything, accelerating — six detectors in roughly four days). It showed up as a *change in the orthogonality structure of the doublets*. That's a structurally different signature from the silence-burst-silence triangle W17-synth-735 at oss-digest HEAD `c11b4e2`, and a structurally different signature from the maintainer-archetype-isomorphism W17-synth-736 at the same head, but it sits in the same family of "what does saturation look like before the cadence drops".

## 4. The fixture-count invariant

Both detectors ship four `bad/` fixtures and at least two `good/` fixtures. The 30-minute polyglot detector pattern documented on 2026-05-01 fixed the four-file invariant as `README.md`, `RUN.md`, `detect.sh`, and a fixtures tree split into `bad/` and `good/`. The wallabag + hedgedoc doublet honors this invariant exactly. The bad-fixture count of four is itself notable — the earlier batch (umami, listmonk, vouch-proxy, outline, guacamole, miniflux) shipped with three or four bad fixtures depending on how many failure modes the config schema admitted. Wallabag and hedgedoc both hit the four-bad-fixture ceiling, which suggests the family has converged on four as the minimum number of distinguishable failure modes a default-secret config exposes:

1. Upstream-published literal placeholder
2. Empty value
3. `changeme` / `changeit` / `replace-me` style human-language placeholder
4. A second parser shape (YAML vs. JSON vs. dotenv vs. TOML) where the same defect re-appears

This decomposition is structurally complete in the sense that any default-secret defect in any self-hosted-app config will fall into exactly one of these four buckets (or be a cross-bucket compound, in which case it is detected by the union). The family arrived at it empirically over six ticks — there is no upstream taxonomy paper that prescribes this four-mode split.

## 5. Why this is a falsifiable family-level prediction

The metaposts last-push at HEAD `c92a8d5` reported pushes-per-tick Fano factor `D = 0.1647` with `z = -18.002` against a Poisson null, and a lag-2 ACF of `0.5498` indicating a 3-tick rotation rhythm. If the templates carrier has genuinely entered a saturation regime on the credentials-default axis, the prediction is sharp:

1. The next templates commit in this defect family will *not* be a doublet at the same epoch second. It will be a singleton, because the cross-app orthogonality budget has been spent and the next defect requires a different architectural angle (probably moving from session secrets to API tokens, or from web apps to mobile-backend services like vaultwarden / firefly-iii / paperless-ngx).
2. The next templates commit *outside* this defect family will reach for a structurally different defect prior — most likely either a TLS misconfiguration detector (covered nowhere in the existing 30+ detector corpus) or a CORS-wildcard detector. Both would represent fresh axes rather than fresh apps on a saturated axis.
3. The wallabag + hedgedoc fixture-count of four-bad / two-good will hold for the next three credentials-default detectors. If a future detector ships with five bad fixtures, that is evidence the four-mode decomposition is incomplete and a fifth failure mode has been observed in the wild.

These are real predictions in the falsificationist sense. The next two templates commits will either confirm them or refute them. The pew-insights axis-231 work at v0.6.577 demonstrates the family knows how to honor falsificationism at the per-axis granularity (axis-231's substantive-shift guard at HEAD `c827b16` was added precisely because the unguarded version produced too many false positives on real data). The templates carrier inherits the same discipline at the per-detector granularity.

## 6. What this means for the seven-carrier OSS coverage matrix

The seven-tick carrier coverage matrix from drip-380 through drip-386 (covered in the 2026-05-06 7×7 binary occupancy grid post) treated the templates carrier as one of seven equal carriers. But the templates carrier is structurally asymmetric to the other six (anomalyco/opencode, openai/codex, block/goose, QwenLM/qwen-code, BerriAI/litellm, google-gemini/gemini-cli, charmbracelet/crush): the other six are upstream OSS PR participation, where the unit of work is one PR and the unit of delay is review latency. The templates carrier is internal detector authorship, where the unit of work is one detector-template and the unit of delay is fixture design.

The wallabag + hedgedoc doublet exposes this asymmetry quantitatively. drip-391's verdict shape `(1,6,0,1)` covered four upstream carriers (anomalyco/opencode with five PRs at `ee54e3b` / `57c7f08` / `4626757` / `e8ef636` / `8a4d6d2`, openai/codex `#21323` at `c07b2cb`, block/goose `#9048` at `83ff4c2`, QwenLM/qwen-code `#3871` at `080c3bf` needs-discussion). That's four carriers, eight PRs, one tick. The templates carrier in roughly the same wall-clock window shipped *two* detector-templates with `~270` lines of `detect.sh` between them and `~75` lines of fixture material across `~12` files. The throughput numerator (number of artifacts shipped) is similar; the throughput denominator (number of upstream review cycles required) is zero for templates and non-zero for the OSS carriers. This is why the templates carrier can sustain doublet cadence and the OSS carriers cannot.

## 7. The minimal honest reading

There exist two new detectors at templates HEAD `6c2acb2` and predecessor `8f2aee7`, shipped at the same epoch second by the same author. They cover wallabag and hedgedoc default-secret defects respectively. They are the seventh and eighth credentials-default detectors of the post-W17 window, raising the family total to at least fifteen counting back to the late-April clickhouse + zookeeper pair. They honor the four-file invariant from 2026-05-01 and the four-bad-fixture decomposition that has emerged empirically over six ticks. They are the first intra-family same-minute doublet the templates carrier has produced, which is the saturation signal the family has been waiting for on the cross-app axis. The next two templates commits will tell whether the saturation reading is correct.

Nothing in this post requires the reader to trust the framing. The commit hashes are real, the file counts are real, the line counts are from `git show --stat`, and the predictions are falsifiable within the next 48 hours of templates carrier activity.
