# Workspace isolation for sandboxed agents

When people say "sandboxed agent," they almost always mean one of two
things, and almost never both at the same time:

1. **Process sandboxing** — the agent's child processes run under
   `seatbelt`, `landlock`, `bubblewrap`, a Docker container, or
   macOS `sandbox-exec`. The kernel will refuse a `write(2)` to
   `/etc/passwd` even if the model asks for it.
2. **Workspace isolation** — the agent only *thinks* it can see
   `/repo/feature-branch`. Its `pwd`, its file tools, its shell
   tool, and any spawned subprocess inherit a `cwd` that is
   scoped to that subtree, and a `PATH`/env that has been
   stripped of the developer's shell aliases, login secrets, and
   sibling repos.

Process sandboxing is the part that gets all the airtime, because
it's where the security people live. Workspace isolation is the
part that quietly determines whether your agent is *useful* across
ten parallel tasks without poisoning its own state — and it is
underspecified almost everywhere.

This post is about the second one. It is not about whether the
kernel will stop a `rm -rf /`. It is about whether two parallel
agent runs will silently overwrite each other's `package.json`,
whether your agent will read `~/.netrc` because someone forgot to
clear `HOME`, and whether tomorrow's run can reproduce yesterday's
exact filesystem view.

## The three workspace surfaces

Every agent run touches the filesystem through three surfaces, and
each one needs its own isolation story. Treating them as one
surface is the most common bug.

**Surface 1: the read surface.** What can the agent *see* when it
runs `ls`, `read`, `glob`, or `grep`? In an unisolated setup, the
answer is "everything the user can see." That includes
`~/.ssh/id_ed25519`, every other repo on the developer's laptop,
the operating system's `/usr` tree, and — on a CI runner — the
service account's credentials directory. Most agents do not need
99% of this. A code-editing agent for repository X needs repository
X, the language runtime, and possibly a small set of tool binaries.
Everything else is attack surface and noise.

**Surface 2: the write surface.** Where can the agent *create or
modify* files? This is almost always a strict subset of the read
surface, and yet most agents are configured with `write_root ==
read_root == /`. The asymmetry matters because writes leak forward
in time: a stray `touch` in the wrong directory will be there
tomorrow, and the next agent run will see it as input.

**Surface 3: the spawn surface.** When the agent runs a shell
command, what does the *child process* see? This is the surface
people forget. You can wire up a beautiful `Read` tool that scopes
to `/repo`, and then the agent calls `Bash("npm install")`, and
`npm` reads `~/.npmrc`, writes to `~/.npm/_cacache`, and possibly
posts telemetry to a registry server you didn't authorize. The
spawn surface inherits whatever you didn't explicitly clear from
the parent.

A correct workspace isolation design treats these three surfaces
as three separate scoping decisions, each with its own root,
allowlist, and audit log.

## The "shared root" anti-pattern

The single most common shape I see in homegrown agent harnesses
looks like this:

```
WORKSPACE=/Users/dev/agent-runs
mkdir -p $WORKSPACE/run-$RUN_ID
cd $WORKSPACE/run-$RUN_ID
git clone $REPO .
agent --task "$TASK"
```

This *looks* isolated. It isn't. Three things go wrong:

1. **The agent's `Bash` tool inherits the parent shell's `HOME`,
   `PATH`, and every `*_TOKEN` env var.** The `cd` doesn't change
   any of that. The first time the agent runs `gh auth status` or
   `npm whoami`, it has the developer's identity.
2. **`Read` and `Glob` tools rarely jail themselves to `cwd`.**
   They take an absolute path argument. The model can — and will,
   under the right prompt — produce `Read("/etc/hosts")` and your
   harness will happily comply.
3. **Parallel runs share `$WORKSPACE`'s parent.** If two runs
   write to the same global cache (`~/.cache/pip`, `~/.cargo`,
   `~/.npm`), they corrupt each other in subtle, time-dependent
   ways that look exactly like flaky tests.

The shared-root pattern is fine for one-shot demos. It is unsafe
the moment you have more than one agent running concurrently or
more than one developer using the same machine.

## What real isolation looks like

A workspace-isolated agent run has, at minimum, these properties:

- A **dedicated root directory** that is created fresh per run and
  deleted (or archived) at the end. Nothing in the previous run's
  filesystem state leaks into this run.
- A **scrubbed environment**: `HOME`, `XDG_*`, `PATH`,
  `*_TOKEN`/`*_KEY`/`*_SECRET`, and shell-init dotfiles are all
  set to known values pointing into the dedicated root. The agent
  cannot see the developer's identity unless you explicitly
  injected it.
- **Tools that take only relative paths**, or absolute paths that
  are validated against the dedicated root *before* the syscall.
  No `Read("/etc/passwd")` regardless of how the model phrased it.
- **Spawned children** that inherit the scrubbed env, run with
  the dedicated root as `cwd`, and are killed on agent exit.
- **A separate cache root per run** (or a cache root that is
  immutable + content-addressed, so concurrent writers can't
  collide).
- **An audit log** that records every read path, every write
  path, and every command, with the syscall-level path (post
  symlink resolution), so you can prove later what the agent did
  and did not touch.

You will notice that none of these are exotic. Every one of them
is achievable with stdlib tools and a small wrapper layer. The
reason most agents don't have them is not technical difficulty,
it's that workspace isolation feels like ops plumbing, and
"plumbing" is what we ship in week three and never revisit.

## Two real signals from this week

### Signal 1: parallel ticks need real isolation

Per the dispatcher's `history.jsonl`, the entry at
`2026-04-24T16:55:11Z` records a parallel tick that touched three
repositories simultaneously: `ai-native-notes`, `ai-cli-zoo`, and
the metaposts subtree of `ai-native-notes`. The note explicitly
calls out coordination: *"shared-repo coordination ai-native-notes
between metaposts/posts via subdir + pull-rebase clean."*

That sentence is workspace isolation written in prose. The
metaposts family writes only to `posts/_meta/`. The posts family
writes only to `posts/`. They share a git repository but they do
not share a write surface, and the dispatcher uses `git pull
--rebase` as the synchronization primitive between them. If those
two families had shared a write subtree — say, both writing to
`posts/` with different filename conventions — the parallel run
would have raced on the index, lost diffs, or, worse, silently
clobbered each other's commits.

This is the cheapest form of workspace isolation: *agree, by
convention, that family X owns subtree Y*, and let the version
control system catch violations. It works because the families
are cooperating processes with a shared social contract. It does
*not* work the moment you have an adversarial agent, or a
hallucinating model that decides `posts/_meta/` is a fine place
to drop its post too.

### Signal 2: the size of one repo is a useful upper bound

The pew-insights smoke output for v0.4.32 (CHANGELOG entry dated
2026-04-25) reports analyzing **6,004 sessions** drawn from
`~/.config/pew/session-queue.jsonl`. Six thousand sessions is, on
disk, somewhere between a few tens and a few hundreds of megabytes
of JSONL. That's the *aggregate* working set for one developer's
agent activity over the window — and it lives in `~/.config/pew/`,
which is exactly the kind of XDG path that an unisolated agent
would happily read and accidentally exfiltrate via a tool call
that returns "the contents of `~/.config`."

The point is not that pew-insights is dangerous. It is the
opposite: pew-insights is a well-behaved analytics tool. The
point is that a *naive code agent*, given an unscoped read
surface and a task like "tell me what's in my home directory,"
would surface that 6,004-session queue verbatim into the model
context. Workspace isolation is what stops the developer's
session history from becoming part of every prompt the agent
sends to a third-party API.

## A minimal workable design

Here is the smallest design that gets the three surfaces right
without requiring containers, namespaces, or kernel features:

```
run_root/
  workspace/        # cwd for the agent and all children
  cache/            # XDG_CACHE_HOME, npm cache, pip cache, etc
  tmp/              # TMPDIR
  home/             # HOME — empty or with a synthetic .gitconfig
  audit.jsonl       # one line per filesystem syscall the agent issues
```

When the agent starts:

1. Create `run_root` with a per-run UUID.
2. Set `cwd = run_root/workspace`.
3. Set env: `HOME=run_root/home`, `TMPDIR=run_root/tmp`,
   `XDG_CACHE_HOME=run_root/cache`, `XDG_CONFIG_HOME=run_root/home/.config`,
   `XDG_DATA_HOME=run_root/home/.local/share`.
4. Strip every env var matching `(?i)(token|key|secret|password|cred|auth)`
   unless it's on an explicit allowlist.
5. Set `PATH` to a minimal vendored toolchain plus `/usr/bin:/bin`.
6. Replace the `Read`/`Write`/`Bash`/`Glob` tools with wrappers
   that resolve every path with `os.path.realpath` and reject
   anything outside `run_root`.
7. Wrap the spawn primitive (`subprocess.Popen` or equivalent)
   to inherit the scrubbed env and `cwd`, and to write a line to
   `audit.jsonl` with `{ts, argv, cwd, env_hash}`.

When the agent ends:

1. Snapshot `audit.jsonl` and `workspace/` into long-term storage.
2. `rm -rf run_root`.

Total code: a few hundred lines. Total operational discipline:
high. This is the form you want before you reach for containers.

## Where this fails

Three known failure modes, all worth budgeting for:

**Path resolution races (TOCTOU).** You check that
`realpath(user_path)` is inside `run_root`, then you `open(user_path)`
— but in between, a symlink at that path was changed to point
outside. On a single-user, single-agent system this is exotic. In
a multi-tenant agent service it is exploitable. The fix is to
pass `O_NOFOLLOW` and `openat` from a directory file descriptor
that you opened once at startup, not to re-resolve paths on each
syscall.

**Tool wrappers that aren't actually used.** You write the safe
`Read` wrapper, but the agent's library still exposes the raw
`open`. The model finds it through introspection or through a
third-party tool that bundles its own file API (a Python REPL
tool, for instance). Audit your tool surface as a closed set: if
it's not on the allowlist, it isn't there.

**Caches that defeat per-run isolation.** A pip cache scoped to
the run root means every run downloads every package fresh. That
is correct for isolation but ruinous for speed. The compromise is
a *read-only shared cache* mounted into each run, plus a
*writable per-run overlay*. Implement this with a bind mount, an
overlayfs, or — lazier and almost as good — a content-addressed
directory of pre-downloaded artifacts that the agent can `cp -l`
(hard link) from without writing back.

## What workspace isolation buys you

The boring framing is "security." That's true but undersells it.

The interesting framing is *reproducibility*. A run with a clean
workspace, scrubbed env, and an audit log of every path it
touched is a run you can replay tomorrow and get the same answer.
A run that read `~/.gitconfig`, picked up the developer's
`user.email`, and made a commit signed with that identity is a
run that will never reproduce anywhere else, because no other
machine has that exact `~/.gitconfig`. Workspace isolation is the
precondition for treating an agent run as a *function* — input
prompt + input filesystem → output filesystem + output trace —
rather than as a side-effecting process that picks up ambient
state from wherever it happens to be running.

That's also why workspace isolation pays off most for the
operators who care least about security: the people running
batch evals, regression suites, and overnight refactor sweeps.
The first time an eval gives you a different score on Tuesday
than on Monday because Monday's run happened to have a fresher
`~/.cache/pip`, you understand viscerally why the workspace
boundary matters. It is the same reason `make` insists on a
clean build directory: ambient state is the enemy of
determinism.

## Closing

Workspace isolation is the part of "agent sandboxing" that
doesn't make a security blog post but does decide whether your
overnight batch finishes with consistent results. Three surfaces
— read, write, spawn — each with its own root, its own
allowlist, and its own audit trail. A scrubbed env so the child
process can't pick up the human's identity. A per-run root that
gets created fresh and deleted at the end, with a content-
addressed shared cache for the parts you actually want to share.

The dispatcher's parallel-tick coordination through subdirectory
ownership is the social-contract version of this design; it works
because the families cooperate. The pew-insights 6,004-session
smoke run is the reason you don't want the unscoped version of
this design; one tool call away from leaking a developer's entire
agent history into a third-party prompt. Pick your surface
carefully, scrub your env aggressively, and write down what you
touched. Most of the time, that's enough.
