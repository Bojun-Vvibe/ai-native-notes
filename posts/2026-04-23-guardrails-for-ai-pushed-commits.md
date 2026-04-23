---
title: "A pre-push guardrail for AI-generated commits"
date: 2026-04-23
tags: [guardrails, security, git-hooks, ai-generated-code, leakage]
est_reading_time: 7 min
---

The first time a coding agent quietly pasted an internal hostname into a public commit, I learned that "I'll just be careful" is not a control. Agents do not know which strings are sensitive. They will cheerfully echo whatever sat in their context window, including the contents of the file three turns ago that you forgot was open. The only durable defense is a script that runs every time, refuses to be skipped, and fails closed.

This post describes the actual shape of that script — the categories of check, the regex strategy, the smoke-test pattern that catches the script breaking, and the architectural decision to keep guardrails in their own repo.

## Fail closed, not open

The single most important property of a guardrail is what it does when it doesn't know. If the script is unsure — pattern is borderline, file is binary, parse failed — it must block, not allow. "Fail closed" means: ambiguity becomes a human decision, not a silent pass-through. Agents will exploit any silent pass-through, not maliciously, just by sheer volume.

This is the inverse of how most lint tools work. A linter that crashes on a weird file usually skips the file and moves on. A guardrail that crashes on a weird file must abort the push. The asymmetry is intentional: a false positive costs you 30 seconds of investigation; a false negative leaks something irreversible.

## The four categories of check

Every guardrail I have written or maintained ends up with the same four buckets. Order matters; cheap checks first.

1. **Internal-string blacklist.** A regex matching identifiers that should never appear in a public commit: employer names, internal codenames, internal hostnames, project slugs from private workspaces. This is the most likely thing to fire and the cheapest to check.
2. **Secret patterns.** API keys, private keys, OAuth tokens. Use the same regex set that GitHub's secret scanner uses, plus any provider-specific patterns you've seen leak before.
3. **Forbidden file types.** `.env`, `.pem`, `.p12`, `.mobileprovision`, `.npmrc`, `id_rsa`, `id_ed25519`. Block by filename and extension. These almost never have a legitimate reason to be in a public repo.
4. **Oversized blobs.** Anything above ~5 MB in a commit. Either it's a binary that doesn't belong (debug build, screenshot, dataset) or it's something you should be using Git LFS for. Either way, ask before letting it through.

A fifth optional bucket I've added recently: **offensive-security artifact fingerprints.** Strings that match the names of well-known sample files distributed by red-team tooling repositories. If an agent has somehow pulled one of these into a commit, that is a several-alarms event and the push must be blocked. This bucket has a low false-positive rate because legitimate application code does not contain those exact filenames or repository names.

## The actual script shape

The hook lives in a separate repo and is symlinked into each protected repo's `.git/hooks/pre-push`. The skeleton:

```bash
#!/usr/bin/env bash
set -u

fail() { echo "[guardrail BLOCK] $*" >&2; exit 1; }
ok()   { echo "[guardrail OK]    $*" >&2; }

remote="${1:-}"
url="${2:-}"

# Scope: only enforce on the repos under the protected GitHub org.
case "$url" in
  *MyPublicOrg/*) ;;
  *) ok "non-protected remote ($url) — skipping"; exit 0 ;;
esac

# Read the git pre-push protocol from stdin.
refs_payload=$(cat)
[ -z "$refs_payload" ] && { ok "no refs"; exit 0; }

# Build the commit range.
range_list=()
while read -r local_ref local_sha remote_ref remote_sha; do
  [ "$local_sha" = "0000000000000000000000000000000000000000" ] && continue
  if [ "$remote_sha" = "0000000000000000000000000000000000000000" ]; then
    range_list+=("$local_sha" "--not" "--remotes" "--max-count=200")
  else
    range_list+=("$remote_sha..$local_sha")
  fi
done <<< "$refs_payload"

commits=$(git rev-list "${range_list[@]}" 2>/dev/null | head -500)

# Block 1: internal string blacklist.
internal_patterns='(acmecorp\.internal|secret-codename|build-cluster-[a-z0-9]+|CompanyConfidential)'
for sha in $commits; do
  hits=$(git show --no-color --pretty=format: "$sha" | grep -inE "$internal_patterns" | head -5)
  [ -n "$hits" ] && fail "commit $sha contains internal token(s):
$hits"
done
ok "no internal strings"

# Block 2: secret patterns.
secret_patterns='(sk-[A-Za-z0-9]{20,}|gh[pso]_[A-Za-z0-9]{20,}|AKIA[0-9A-Z]{16}|-----BEGIN (RSA |OPENSSH |EC )?PRIVATE KEY-----|xox[bp]-[0-9]+-[0-9]+)'
for sha in $commits; do
  hits=$(git show --no-color --pretty=format: "$sha" | grep -inE "$secret_patterns" | head -5)
  [ -n "$hits" ] && fail "commit $sha contains likely secret(s):
$hits"
done
ok "no obvious secrets"

# Block 3: forbidden filenames.
forbidden='\.(mobileprovision|p12|pem|pfx|keystore|env)$|(^|/)\.(npmrc|netrc)$|(^|/)id_(rsa|ed25519)$'
for sha in $commits; do
  files=$(git show --no-color --name-only --pretty=format: "$sha" | grep -E "$forbidden" | head -5)
  [ -n "$files" ] && fail "commit $sha touches forbidden file(s):
$files"
done
ok "no forbidden filenames"

# Block 4: oversized blobs.
for sha in $commits; do
  big=$(git diff-tree --no-commit-id -r --root "$sha" 2>/dev/null \
        | awk '{print $4,$6}' \
        | while read -r blob path; do
            size=$(git cat-file -s "$blob" 2>/dev/null || echo 0)
            [ "$size" -gt 5242880 ] && echo "$path ($size bytes)"
          done)
  [ -n "$big" ] && fail "commit $sha contains oversized blob(s):
$big"
done
ok "no oversized blobs"

# Block 5: offensive-security artifact fingerprints (your own list goes here).
attack_names='(KnownOffensiveRepoName1|KnownOffensiveRepoName2|known_offensive_repo_marker)'
for sha in $commits; do
  hits=$(git show --no-color --pretty=format: "$sha" | grep -inE "$attack_names" | head -3)
  [ -n "$hits" ] && fail "commit $sha references offensive-security artifact(s):
$hits"
done
ok "no offensive-security artifact references"

ok "all checks passed for $url"
exit 0
```

A few things in here are deliberate.

The `set -u` at the top makes unset variables an error. If the hook ever stops being called with the standard pre-push arguments, it crashes loudly instead of silently allowing the push.

The pattern `*MyPublicOrg/*) ;;` scopes enforcement to the public org. Pushing to private remotes (a personal scratch fork on a different account) is unaffected. This is the only safe default — you do not want a guardrail that blocks pushes to legitimate private remotes because it can't tell them apart.

The use of `head -5` on every grep result keeps error messages bounded. A 1000-hit regex match dumping into the terminal is its own kind of failure.

## The smoke test

A guardrail that has never been tested is a guardrail that does not work. The most common failure mode is silent: the regex has a typo, no commit ever matches, every push is allowed. You will discover this exactly once, at the worst possible time.

The fix is a smoke test that lives next to the script and runs in CI:

```bash
#!/usr/bin/env bash
# smoke-test.sh — synth a commit that should be blocked, verify it is.
set -e
TMP=$(mktemp -d)
git -C "$TMP" init -q
git -C "$TMP" remote add origin https://github.com/MyPublicOrg/test.git

# Build a commit that hits each rule. One file per rule.
echo "an acmecorp.internal hostname" > "$TMP/leak.txt"
printf 'sk-%s\n' "$(printf 'A%.0s' {1..36})" > "$TMP/secret.txt"   # synthesized at runtime so this script itself does not contain a literal secret pattern
touch "$TMP/cert.pem"
dd if=/dev/zero of="$TMP/big.bin" bs=1M count=6 status=none
touch "$TMP/known_offensive_repo_marker.txt"   # synthesize a path that hits the offensive-artifact regex

git -C "$TMP" add -A
git -C "$TMP" -c user.email=t@t -c user.name=t commit -q -m "should-be-blocked"

# Run the hook against this synthetic push.
sha=$(git -C "$TMP" rev-parse HEAD)
echo "refs/heads/main $sha refs/heads/main 0000000000000000000000000000000000000000" \
  | ./pre-push origin https://github.com/MyPublicOrg/test.git \
  && { echo "FAIL: hook allowed a commit with 5 violations"; exit 1; } \
  || echo "PASS: hook blocked as expected"
```

Run this on every change to the hook. If it ever stops blocking, you have a regression and you know about it before the next push to a real repo.

## Why guardrails belong in their own repo

I keep the hook in a separate `.guardrails` repo, symlinked into each protected repo:

```
ln -s /Users/bojun/Projects/MyOrg/.guardrails/pre-push \
      /Users/bojun/Projects/MyOrg/some-repo/.git/hooks/pre-push
```

Three reasons.

First, one place to update. When a new internal codename appears, I add it to the regex once and every protected repo benefits on the next push. With per-repo hooks I would need to remember to update every one, and I would forget.

Second, version control on the controls themselves. The `.guardrails` repo has its own git history, its own smoke tests, its own README documenting why each pattern is in the list. If a pattern gets removed, there's a commit explaining why.

Third, easy to audit. A reviewer asking "what does your guardrail block" gets a single repo URL, not "let me grep across 12 repos for differences."

The downside: a fresh clone of a protected repo does not have the symlink, so the hook is not active until you install it. I solve this with a one-line install script in the guardrails repo and a CI check on the protected repo that fails if the symlink is missing on the maintainer's machine. Not perfect; sufficient.

## What I would do differently

I added the offensive-security-artifact bucket months after the others, after a single incident in which a research-style repo was cloned by an automated workflow and dropped sample exploit files into a working directory under my home folder. They never got committed. They did get scanned by the endpoint security tool, which opened a high-severity ticket. The guardrail-as-installed at the time would not have caught the commit because the pattern wasn't there. It is now. Add new patterns the day you learn the failure mode; do not wait for a second incident.

## Links

- git pre-push hook protocol: https://git-scm.com/docs/githooks#_pre_push
- GitHub secret scanning patterns reference: https://docs.github.com/en/code-security/secret-scanning/secret-scanning-patterns
- gitleaks (off-the-shelf alternative if you don't want to maintain regexes yourself): https://github.com/gitleaks/gitleaks
