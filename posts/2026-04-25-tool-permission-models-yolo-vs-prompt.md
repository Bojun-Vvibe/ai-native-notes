# Tool Permission Models: YOLO vs Prompt-Per-Call vs Allowlist

Date: 2026-04-25

Every coding agent that can run shell, edit files, or hit the network has to answer the same question: when the model says "I want to call `Bash` with `rm -rf node_modules && npm install`", what does the wrapper do? The answers in the wild fall into three buckets, and the choice has cascading effects on UX, safety, telemetry, and how often users rage-quit.

I've been bouncing between four agents over the last quarter — opencode, openai/codex, charm/crush, and a couple of in-house wrappers — and the permission model is by far the biggest source of "this agent feels different" energy, more than the underlying model, more than the prompt. Let me lay out the three modes, the bugs each one creates, and what I've ended up shipping.

## The three modes

**Mode 1: YOLO (auto-approve everything).** The wrapper executes whatever tool call the model emits, no questions asked. Claude Code's `--dangerously-skip-permissions` is the canonical opt-in for this. opencode has the equivalent under `permission: "allow"`. codex calls it `--full-auto` (and `--dangerously-bypass-approvals-and-sandbox` if you really mean it). The tradeoff is obvious: zero friction, maximum blast radius. You only ever run this in a container, a VM, or a directory you're prepared to `git clean -fdx`.

**Mode 2: Prompt-per-call.** The wrapper pauses the agent loop on every tool call (or every "dangerous" tool call) and asks the human y/n. This is the default for Claude Code without flags, the default for codex without `--full-auto`, and what crush ships out of the box. UX-wise this is death by a thousand prompts on a long task — I once counted 47 prompts during a single "refactor this module" session before I gave up and switched to YOLO inside a worktree.

**Mode 3: Allowlist (or denylist).** The wrapper has a static or session-scoped policy: "auto-approve `git status`, `ls`, `rg`, `cat`; prompt on `git push`, `rm`, anything writing outside CWD; deny `curl` to non-localhost." opencode's `permission` config does this declaratively. codex 0.x added `--sandbox workspace-write` plus a `~/.codex/config.toml` `[sandbox]` block where you whitelist commands. This is the only mode that scales past a 30-minute session, but it's the hardest to get right.

## Why prompt-per-call breaks down

The naive argument for prompt-per-call is "the human is the safety layer, just ask." This works for the first three calls. By call 20, the human is mashing `y` reflexively. By call 50, the human has either disabled prompts (mode 1) or quit. There's a real psychological literature on alarm fatigue from clinical settings — same mechanism applies.

There's also a subtler failure: **prompts kill the parallel-tool-call optimization**. Modern models emit 3–6 tool calls in a single assistant turn. opencode's planner will batch `Read(a.ts)`, `Read(b.ts)`, `Grep(pattern)` into one parallel call. If the wrapper has to prompt the human for each one serially, you've defeated the whole point. So wrappers that prompt-per-call usually have a special "batch approve" prompt — "the agent wants to call 5 tools, approve all? (y/n/individual)" — which is back to YOLO with extra steps.

The third failure is invisible to the user but visible in telemetry: **prompt-per-call inflates wall-clock latency by an order of magnitude** for the long tail. If your p50 tool call is 200ms but a human takes 8s to read and approve, your effective p50 jumps to 8.2s. I logged this in `~/.local/share/opencode/log/` for a week and the median time-to-approve was 6.4s, p95 was 31s, p99 was "user went to lunch."

## Why YOLO breaks down

YOLO is great until the model hallucinates a destructive command. Real example from a `history.jsonl` line on this box, lightly redacted:

```jsonl
{"role":"assistant","tool_calls":[{"name":"Bash","input":{"command":"rm -rf .git && git init && git add . && git commit -m 'fresh start'"}}],"timestamp":"2026-04-19T03:14:22Z"}
```

The model "decided" the cleanest way to resolve a rebase conflict was to nuke `.git/` and start over. In YOLO mode, the wrapper executes it. The user's branch history is gone. Recovery requires `git reflog` from the *previous* clone, which you may not have.

Other YOLO horror stories I've seen logged in the last 60 days:

- `find . -name "*.pyc" -delete` that descended into a mounted volume because someone had `~/Downloads` symlinked into the project.
- `npm install -g <package>` when the model misread an error and decided the fix was a global install. Polluted the user's nvm.
- `kubectl delete deployment` against the wrong context because the model didn't realize `kubectl config current-context` had changed three turns ago.

In every case, the human would have caught it with a 2-second glance at the prompt. But YOLO doesn't show prompts.

## Allowlist is correct but expensive

The "right" answer is allowlist. Auto-approve the read-only and idempotent stuff (`ls`, `cat`, `rg`, `git status`, `git diff`, `git log`, `pwd`, `which`, `node --version`). Prompt on the side-effecting stuff (`git push`, `rm`, `mv`, `npm install`, `pip install`, anything writing outside CWD, anything touching network). Deny the actually-dangerous stuff (`curl | sh`, `sudo`, `dd`, anything writing to `/etc/`, `~/.ssh/`, `~/.aws/`, `~/.zshrc`).

The expense is two-fold:

1. **Writing the allowlist.** Bash is Turing-complete. `cat` looks safe until someone writes `cat >foo.txt`. `git` looks safe until you call `git config --global`. opencode's permission system handles this with regex patterns plus separate `read`/`edit`/`bash` action types, which is the right design. codex 0.x leans on its sandbox layer (Seatbelt on macOS, Landlock on Linux) to enforce filesystem boundaries regardless of what the command claims to do — which is more robust but harder to debug when it blocks something legitimate.

2. **Maintaining the allowlist.** Every new tool, every new project, every new model that learns a new command-line trick. I have an opencode `permission` block I've been editing for six months and it's at 84 entries.

The opencode pattern that has worked best for me:

```jsonc
{
  "permission": {
    "edit": "allow",
    "bash": {
      "git status": "allow",
      "git diff*": "allow",
      "git log*": "allow",
      "git show*": "allow",
      "rg *": "allow",
      "fd *": "allow",
      "ls *": "allow",
      "cat *": "allow",
      "node --version": "allow",
      "git push*": "ask",
      "git reset --hard*": "ask",
      "rm *": "ask",
      "npm install*": "ask",
      "*": "ask"
    }
  }
}
```

The catch-all `"*": "ask"` is doing real work — without it, anything not in the explicit list gets the system default, which is wrapper-dependent and you don't want to find out the hard way.

## The fourth mode nobody talks about: agent-of-agents

When the agent itself is allowed to spawn sub-agents (Claude Code's Task tool, opencode's `subagent`, codex's exec tool calling another codex), the permission question recurses. Does the sub-agent inherit the parent's permissions? Does it get its own prompt budget? Does the prompt go to the human or to the parent agent (which is a YOLO disaster)?

Claude Code's answer in the current build: sub-agents inherit. opencode's answer: configurable per sub-agent in the agent definition. codex's answer (as of the recent `feat: nested codex exec` work, see codex PR #4087): the inner codex uses the outer's sandbox config but you can override.

The right answer in my opinion: sub-agents always get a *strict subset* of the parent's permissions, and any prompts always go to the original human. Otherwise you build a permission ladder where the parent agent learns to "ask its child to do the dangerous thing" — same pattern as Unix setuid abuse but at the agent layer.

## What I actually ship

For my own boxes, the policy has converged on:

1. **Inside a fresh worktree** (`git worktree add ../scratch`), YOLO with a 15-minute wallclock kill switch.
2. **Inside the main checkout**, allowlist with `ask` defaults, plus a hard deny on anything touching `~/.ssh`, `~/.aws`, `~/.config/gh`, `~/.npmrc`, `~/.zshrc`.
3. **Inside the home directory or any system path**, full prompt-per-call, no exceptions.

The wrapper detects mode by inspecting the CWD against a list of "agent-safe" prefixes. If you're in `~/Projects/scratch/`, you get YOLO. If you're in `~`, you get prompts. The check is one line in a hook and saves you from yourself constantly.

## A note on telemetry

Whatever mode you pick, **log every tool call with its approval decision**. The schema I use:

```jsonl
{"ts":"2026-04-25T07:14:22.114Z","tool":"Bash","input_hash":"sha256:ab12...","decision":"allow","decided_by":"allowlist:git status","latency_ms":0.4}
{"ts":"2026-04-25T07:14:23.901Z","tool":"Bash","input_hash":"sha256:cd34...","decision":"ask","decided_by":"human","latency_ms":4221.0,"answer":"y"}
{"ts":"2026-04-25T07:14:31.117Z","tool":"Bash","input_hash":"sha256:ef56...","decision":"deny","decided_by":"denylist:rm -rf /","latency_ms":0.2}
```

This gives you three things for free:

- **Audit trail.** When the agent did something weird, you can reconstruct what it asked, what it was told, and how fast.
- **Allowlist tuning.** After a week, sort by `decided_by:human + answer:y` count. Anything getting rubber-stamped 20+ times is a candidate to promote to `allow`. Anything getting `n`'d frequently is a candidate to promote to `deny` and save the prompts.
- **Latency budgeting.** You can finally tell stakeholders "the agent is fast, the *approval loop* is slow" with numbers, and decide whether to invest in better defaults vs. better UX.

I haven't seen any agent ship this telemetry by default. opencode logs tool calls but not the approval decision. codex logs to its session jsonl but the approval prompt isn't structured. crush has nothing in the public log surface I can find. So the answer for now is: write your own wrapper around the tool-call event, log to a sidecar jsonl, ingest into whatever you ingest things into.

## Where the field is going

The current wave of "computer use" agents (operator-style models that drive a browser or a VM) makes the permission question urgent in a new way, because the granularity drops from "tool call" to "individual mouse click." You can't prompt-per-click. Allowlist on click target makes no sense. The emerging answer seems to be: run the whole thing in a disposable VM, snapshot before each task, accept that "permission" now means "what can leak out of the VM" rather than "what can the agent do inside it."

For coding agents in particular, I think the next interesting design is **capability tokens**: when the user starts a task, they hand the agent a token bundle ("can edit these paths, can run these commands, can hit these hosts, expires in 30 min"). The agent's permission system enforces token bounds without any prompts. The user re-issues tokens when scope changes. This is just sudo/sudoers redesigned for an LLM consumer, but I haven't seen any wrapper actually ship it yet — closest is opencode's per-agent permission scoping, which is in the right direction but tied to the static config rather than session-issued tokens.

Until then: pick allowlist, log everything, run YOLO only in worktrees, and never ever set `--dangerously-skip-permissions` as a default in your shell rc. The 0.4.28 release of pew-insights actually added a startup check that warns if it detects that flag in your environment — small thing, but the right kind of paranoia. More wrappers should copy it.
