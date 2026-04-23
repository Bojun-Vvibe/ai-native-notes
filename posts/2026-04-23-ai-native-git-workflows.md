---
title: "AI-native git workflows: branches, worktrees, and the agent that opens the PR"
date: 2026-04-23
tags: [git, agents, workflow, worktrees, automation]
est_reading_time: 14 min
---

## TL;DR

Git was designed for humans who think before committing. Agents do not. They commit eagerly, branch impulsively, and produce diffs that are correct in pieces and wrong as a whole. The naive answer — "let the agent commit to a branch, then a human reviews" — works once, breaks the third time you try to run two agents in the same repo, and falls over completely the moment you wire any of it into CI. After several iterations I have settled on a workflow built on three primitives: **one worktree per agent task**, **machine-readable commit trailers**, and **a guardrailed `pre-push` that treats the agent like an untrusted contributor**. None of this is exotic git; what is new is using these primitives deliberately for non-human authors. This post is the playbook, with the scripts I actually use.

## The problem

The first time I let an agent commit on its own, it worked great: small task, clean diff, push, merge, done. The second time, the agent rebased my in-flight branch onto main, "to keep things tidy," and lost about an hour of unpushed work. The third time, two agents in two terminal tabs raced each other on the same branch and produced a merge state nobody could explain. The fourth time, an agent pushed a commit that contained an API key it had cheerfully extracted from a `.env` file to "make the example runnable." Every one of these was preventable, but only after I stopped treating "the agent uses git" as a single behavior and started treating it as a workflow with isolation, attribution, and gates.

## The setup

- Modern git (2.40+) for `worktree` ergonomics and `--update-refs`.
- A long-lived "main" repo clone with no agent activity in it directly.
- Agents launched into per-task worktrees under a dedicated parent directory.
- A `pre-push` hook installed at the repo level, enforcing project-wide rules, and a per-task `commit-msg` hook validating trailers.
- A small `task` shell wrapper that bootstraps everything.

Nothing here requires a hosted git provider — it works the same on plain `git`.

## Primitive 1: one worktree per agent task

The single biggest change is to stop running agents in the same working tree as the human. Worktrees give you cheap, isolated checkouts that share the underlying object database, which means switching is fast, disk usage is reasonable, and merges remain easy because everyone is talking to the same repo.

The directory layout I use:

```
~/code/proj                 # main human checkout, branch: main or a feature branch
~/code/proj.tasks/          # parent for agent worktrees
  fix-pagination-2026-04-22/  # one task = one worktree = one branch
  refactor-auth-2026-04-22/
  ...
```

A `task new` command sets it up:

```bash
# bin/task — installed on PATH
set -euo pipefail
cmd=$1; shift

case "$cmd" in
new)
  slug=$1
  date=$(date +%Y-%m-%d)
  branch="agent/${date}-${slug}"
  dir="$HOME/code/proj.tasks/${date}-${slug}"
  cd "$HOME/code/proj"
  # Always branch off the current main, never off an in-flight feature.
  git fetch origin main --quiet
  git worktree add -b "$branch" "$dir" origin/main
  # Install per-task hooks (symlinks to repo-tracked scripts).
  ln -sf "$PWD/.githooks/commit-msg" "$dir/.git/hooks/commit-msg"
  ln -sf "$PWD/.githooks/pre-push"   "$dir/.git/hooks/pre-push"
  echo "$dir"
  ;;

done)
  dir=$1
  cd "$dir"
  branch=$(git symbolic-ref --short HEAD)
  cd "$HOME/code/proj"
  git worktree remove "$dir"
  git branch -D "$branch" 2>/dev/null || true
  ;;
esac
```

Three things to call out:

1. **The branch name encodes provenance.** `agent/2026-04-22-fix-pagination` tells me at a glance that an agent produced it, when, and roughly what it tried to do. I never have to ask "did a human make this branch or did the bot?"
2. **Always branch from `origin/main`.** Branching off the current human checkout means an agent picks up half-finished work, which is the single most common cause of "the agent's PR has 50 unrelated changes."
3. **Hooks are installed per worktree.** `core.hooksPath` would be cleaner but breaks if a teammate hasn't configured it. Per-worktree symlinks are visible in `.git/hooks/` and obvious when missing.

## Primitive 2: commit trailers as machine-readable attribution

Once agents are committing on your behalf, you want to be able to answer: which commits were authored by which agent, with which model, in which session, against which task spec. The clean way to do this is RFC-822-style trailers, which git natively supports via `git interpret-trailers` and `git log --format`.

A commit message from an agent should look like:

```
fix(pagination): clamp page size to [1, 100]

Out-of-range page_size values were causing 500s in /search. This commit
adds explicit clamping, returns a 400 when the input is non-integer, and
adds a unit test for both edges.

Agent-Profile: implementer
Agent-Model: anthropic/claude-opus
Agent-Session: 2026-04-22T18:11:04-sess-7c2
Task-Id: fix-pagination
Reviewed-By: human-pending
Co-Authored-By: Bojun <bojun@example.com>
```

The `commit-msg` hook validates the trailers exist and look right:

```bash
#!/usr/bin/env bash
# .githooks/commit-msg
msg_file=$1
required=(Agent-Profile Agent-Model Agent-Session Task-Id)

for trailer in "${required[@]}"; do
  if ! git interpret-trailers --parse < "$msg_file" \
       | grep -qi "^${trailer}:"; then
    echo "commit-msg: missing required trailer ${trailer}" >&2
    exit 1
  fi
done

# Subject line: conventional-commit prefix + <=72 chars
subject=$(head -1 "$msg_file")
if ! [[ "$subject" =~ ^(feat|fix|chore|refactor|test|docs|perf|build|ci)(\(.+\))?:\ .+ ]]; then
  echo "commit-msg: subject must be conventional-commit form" >&2
  exit 1
fi
if [ ${#subject} -gt 72 ]; then
  echo "commit-msg: subject longer than 72 chars" >&2
  exit 1
fi
```

The hook only runs in agent worktrees (because that is where it is symlinked). Human commits in the main checkout are unaffected.

The payoff comes when you can run queries like:

```bash
# All commits by the implementer profile in the last week, with their sessions.
git log --since=1.week \
  --format='%h %s%n  %(trailers:key=Agent-Profile,valueonly,separator=)%n  %(trailers:key=Agent-Session,valueonly,separator=)' \
  | awk '/implementer/{print prev1; print prev2; print $0} {prev2=prev1; prev1=$0}'
```

Or, more usefully: feed the `Agent-Session` trailer back into your token telemetry and answer "what did this PR cost in tokens?" That single linkage — commit ↔ session ↔ tokens — is the foundation of every honest cost-per-feature number you will ever produce for AI-assisted work.

## Primitive 3: pre-push as the last line of defense

Hooks before the push are advisory; the agent can be told to skip them, and sometimes will. The `pre-push` hook is the one that actually has to be right, because by the time it runs, the agent thinks it is done. Treat it like a CI step: deterministic, fast, loud when it fails.

A skeleton that has held up in real use:

```bash
#!/usr/bin/env bash
# .githooks/pre-push
set -uo pipefail
remote=$1; url=$2

# Only enforce on remotes you control. Forks/personal remotes opt out.
case "$url" in
  *github.com:myorg/*|*github.com/myorg/*) ;;
  *) echo "pre-push: skipping non-myorg remote ($url)"; exit 0 ;;
esac

while read -r local_ref local_sha remote_ref remote_sha; do
  [ "$local_sha" = "0000000000000000000000000000000000000000" ] && continue
  if [ "$remote_sha" = "0000000000000000000000000000000000000000" ]; then
    range="$local_sha --not --remotes --max-count=200"
  else
    range="$remote_sha..$local_sha"
  fi

  commits=$(git rev-list $range)
  for sha in $commits; do
    diff=$(git show --no-color --pretty=format: "$sha")

    # 1. Block obvious secrets.
    if echo "$diff" | grep -qE \
      '(sk-[A-Za-z0-9]{20,}|AKIA[0-9A-Z]{16}|-----BEGIN [A-Z ]*PRIVATE KEY-----)'; then
      echo "pre-push: secret-looking content in $sha" >&2
      exit 1
    fi

    # 2. Block files that should never be committed.
    files=$(git show --name-only --pretty=format: "$sha")
    if echo "$files" | grep -qE '(^|/)\.env(\.|$)|\.p12$|\.pem$|id_rsa$'; then
      echo "pre-push: forbidden file in $sha" >&2
      exit 1
    fi

    # 3. If commit was authored by an agent, require matching trailers.
    msg=$(git show -s --format=%B "$sha")
    if echo "$msg" | grep -qi '^Agent-Profile:'; then
      for t in Agent-Model Agent-Session Task-Id; do
        if ! echo "$msg" | grep -qi "^${t}:"; then
          echo "pre-push: agent commit $sha missing $t trailer" >&2
          exit 1
        fi
      done
    fi

    # 4. Reject pushes of >2MB single blobs unless allow-listed.
    big=$(git diff-tree --no-commit-id -r --root "$sha" \
          | awk '$5 != "D" {print $4, $6}' \
          | while read blob path; do
              size=$(git cat-file -s "$blob" 2>/dev/null || echo 0)
              [ "$size" -gt 2000000 ] && echo "$path ($size)"
            done)
    if [ -n "$big" ]; then
      echo "pre-push: oversized blobs in $sha:" >&2
      echo "$big" >&2
      exit 1
    fi
  done
done

echo "pre-push: ok"
```

Notes from running this in anger:

- **Per-remote enforcement.** A blanket hook that blocks every push will eventually block a personal fork at 11pm and you will disable it. Scope it to remotes you control.
- **Don't try to lint code in pre-push.** That belongs in CI. The hook is for things a human reviewer cannot catch quickly: secrets, forbidden files, missing attribution, oversized blobs.
- **Make failures easy to fix.** Print the offending sha and a one-line reason. Agents can read these and recover; cryptic failures send them into retry loops.

## Reviewing an agent PR

Once the workflow above is in place, reviewing an agent's PR has a different shape than reviewing a human's PR. The mechanical things — formatting, trivial bugs, basic test coverage — should be caught by CI before you look. What is left for you is two questions: **does this change match the task** and **is the design coherent across the diff**.

Two tools I use during review:

```bash
# Summarize the agent's reasoning trail across the branch.
git log --reverse --format='%h %s%n  %(trailers:key=Agent-Session,valueonly)' \
    origin/main..HEAD

# Show only files touched by agent commits, ignoring any human fixups.
git log --reverse --format='%H %ae' origin/main..HEAD \
  | awk '$2 ~ /agent\./{print $1}' \
  | xargs -n1 git show --stat --format= \
  | sort -u
```

The first gives me the chronology of what the agent decided to do, in order. The second answers "if I trust the human commits and only need to scrutinize the agent commits, what files do I look at?" Both queries are only possible because of the trailers from primitive 2.

## Merging policy

The default `git merge` for agent branches should be `--no-ff` with a merge commit, even for tiny changes. Squash-merging an agent branch into main destroys the per-commit attribution that the rest of this workflow depends on. Yes, the history is noisier; that is the point — you want the noise when you are auditing what an autonomous process did.

For long-running feature branches that contain both agent and human commits, `git merge --update-refs` (2.38+) keeps stacked branches sane after a rebase, and `git rerere` keeps you from re-resolving the same conflicts. Neither is agent-specific, but both pay off more when you have multiple agent worktrees in flight.

## What does not work

A few things I tried and abandoned:

- **A single shared "agent" branch.** The intuition was "all agent work in one place." The reality was constant conflicts, no per-task attribution, and a branch that lived forever. One worktree per task is strictly better.
- **Letting the agent push directly to `main` behind a CI gate.** The CI gate caught the obvious things, but an agent landing changes faster than humans can read them is a recipe for a system nobody understands. PRs with a human approver step are cheap insurance.
- **Storing prompts in the commit message.** Tempting (full traceability!) but commit messages become unreadable, and the prompts are usually re-derivable from the session ID. Keep the trailer; keep the prompt out.
- **Rebasing agent branches before merge.** Loses the chronology, makes attribution noisy, and tempts the agent into more rebasing on its own. Merge commits are fine.

## What I would do differently

Install the `pre-push` hook on day one, even before you have any agents. It costs nothing while you are the only committer, and it means the first time an agent tries to do something dumb, the safety net is already in place. I waited until after the API-key-in-a-commit incident, which was about three months too late.

## Links

- git worktree: https://git-scm.com/docs/git-worktree
- git interpret-trailers: https://git-scm.com/docs/git-interpret-trailers
- git rerere: https://git-scm.com/docs/git-rerere
- git --update-refs: https://git-scm.com/docs/git-rebase#Documentation/git-rebase.txt---update-refs
- Conventional Commits: https://www.conventionalcommits.org
