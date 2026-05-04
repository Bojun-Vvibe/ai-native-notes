# The ConPTY close-leak pattern: when cross-platform PTY abstractions forget the Windows branch

A recurring failure mode shows up whenever a Go (or Rust, or Node) project decides
to ship its own cross-platform PTY layer instead of pulling in a vetted dependency.
The author writes the Unix path first, gets it working against `creack/pty` or the
equivalent, then bolts on a Windows ConPTY backend in a separate file behind a build
tag. Every method on the `PTY` struct then becomes a fork: if-Unix-do-X-else-Windows-do-Y.
And almost without exception, one method gets forgotten, because the Unix path is so
ergonomic that the developer mentally elides the platform check on the easier branch.

The method that gets forgotten is almost always `Close`.

## The concrete instance: charmbracelet/crush#2606

I want to anchor this in a specific, recent example so the abstract pattern has
something to attach to. PR `charmbracelet/crush#2606` (head SHA
`972b135b66f46f9ac99a824d8d64de3cc5ee5cf7`, +3085/-0 across 15 files) lands three
new packages under `internal/`: `split/` for binary-split pane trees,
`tabmgr/` for tab persistence, and `pty/` for the actual cross-platform PTY layer.
The PTY package is the load-bearing one, and it is also where the bug lives.

The relevant struct is in `internal/pty/pty.go:14-32`:

```go
type PTY struct {
    cmd        *exec.Cmd       // Unix path
    procHandle uintptr         // Windows path (ConPTY HPCON-adjacent)
    handle     interface{}     // *os.File on Unix, ConPTY HPCON on Windows
    mu         sync.Mutex
}
```

So far so good. Two state fields, one per platform, one shared opaque handle. The
Unix path constructs `cmd` via `exec.Cmd` and lets `creack/pty` wire it up. The
Windows path in `pty_windows.go` does not use `cmd` at all — it goes directly to
`kernel32.dll`'s `CreateProcessW` because ConPTY's session model does not
compose cleanly with the Go `exec` package's process-group assumptions. So on
Windows, `p.cmd` is permanently `nil` and `p.procHandle` holds the real process
handle.

Now look at `Close()` at `pty.go:124-145`. The shape is:

```go
func (p *PTY) Close() error {
    p.mu.Lock()
    defer p.mu.Unlock()
    // ... handle close logic ...
    if p.cmd != nil {
        _ = p.cmd.Process.Kill()
        _, _ = p.cmd.Process.Wait()
    }
    return nil
}
```

That `if p.cmd != nil` is the trap. The whole purpose of the cross-platform
abstraction is that consumers of `PTY` do not know or care which OS they are
on; they just call `Close()` and expect the child process to die. On Unix, the
`cmd != nil` branch fires and the child is killed. On Windows, `cmd` is always
nil, the branch is skipped, and the function returns `nil` having terminated
nothing. The ConPTY pseudoconsole gets closed (assuming the handle-close logic
above does that correctly), but the *child process* — typically a `cmd.exe` or
`powershell.exe` running whatever shell the tab was hosting — is not killed.
It becomes an orphan.

The orphan does not always die when the pseudoconsole closes. Windows' process
model is permissive about this: a child process whose console handle becomes
invalid will see I/O errors on its next read or write, but plenty of shells just
spin in their REPL waiting for input that never comes. The shell sits there
holding memory until either the OS kills it on shutdown, or the user notices
their machine is full of detached `cmd.exe` instances under the parent app's
PID space.

The fix is one branch: an `else if p.procHandle != 0` arm that calls
`TerminateProcess(p.procHandle, 1)` via the same `kernel32.dll` proc lookup that
`pty_windows.go` already uses for `CreateProcessW`. Five lines of code. The
reason it was missed is structural, not lazy.

## Why this pattern keeps happening

The structural reason is that **the Unix path is the default path in the
developer's head**. When a Go developer writes a PTY abstraction, they are
running on a Mac or a Linux box. Their entire dev loop — `go test`, `go run`,
manual smoke testing — exercises the Unix path. The Windows path exists in a
separate file behind `//go:build windows`, and it gets compiled by CI on a
runner the author does not log into. So when the author writes `Close()`,
they write the Unix-shaped close logic, see it work locally, and think they
are done. The Windows file gets the construction logic (because without
`pty_windows.go` building, the Unix-side build also fails to type-check the
shared struct), but the cleanup logic — which is symmetric in concept but
asymmetric in code — is much easier to forget because it is only invoked
on the platform you are not running on.

You can see the same shape in the corresponding `Wait()` at `pty.go:147-180`,
which *does* fork correctly:

```go
func (p *PTY) Wait() (int, error) {
    if p.cmd != nil {
        // Unix path
        err := p.cmd.Wait()
        // ...
    } else {
        // Windows path: WaitForSingleObject on procHandle
        return p.waitPlatform()
    }
}
```

`Wait()` got the symmetric treatment because it has to return a value, and the
"forgot to handle the case" branch is impossible to compile away — the type
system forces you to return *something*. `Close()` returns just `error`, and
returning `nil` is a perfectly valid no-op, so the branch can be silently
omitted without a compiler complaint and without any test failing on Linux CI.

This is the same shape as several other classes of cross-platform bug:
`os.File.Sync()` semantics differ between platforms but compile to a single
method; `os.Setenv` interacts with `cgo` differently on Windows; the
`signal.Notify` set of usable signals is platform-dependent. In each case,
the bug is silent on the dev platform and only manifests in production on the
other one.

## Detection patterns: how to catch this in review

When I'm reviewing a PR that introduces a cross-platform abstraction with
per-platform backends, I now look for three things specifically:

**1. Pair every constructor with a destructor scan.** For each field on the
struct that is initialized in `pty_windows.go` but not in `pty_unix.go` (and
vice versa), grep the cleanup methods (`Close`, `Stop`, `Cancel`, `Shutdown`,
`Dispose`, `Drop` if it's a Rust port) for a reference to that field. If the
constructor sets `p.procHandle` but no cleanup method reads `p.procHandle`,
that's a leak signal even before you understand what `procHandle` is for.

**2. Look for `if x != nil` branches in shared code where `x` is set
asymmetrically.** The Unix-Windows asymmetry tends to be encoded as "this
pointer is nil on the other platform." Any `if p.cmd != nil` in cross-platform
code is a place where the unstated `else` branch needs to do whatever the
Windows-equivalent action is. If there is no `else`, one of two things is true:
either the Windows path genuinely does not need the action (rare for cleanup
operations), or the Windows path is leaking. Default assumption should be
the latter until proven otherwise.

**3. Check that the test matrix actually exercises the Windows path.** Most
Go projects' CI is Linux-only by default. If the PR adds a `pty_windows.go`
file but the CI config in the same PR still has `runs-on: ubuntu-latest` and
no `windows-latest` job, the Windows path has *never been compiled, much less
executed*. That alone is reason to request changes — not because the code is
necessarily wrong, but because there is zero evidence either way.

For PR `crush#2606` specifically, all three signals fire. The `procHandle`
field is initialized in `pty_windows.go` but only read by `Wait()`, not
`Close()`. The `Close()` method has an `if p.cmd != nil` with no else. And
the PR doesn't modify CI to add a Windows runner, so the entire
`pty_windows.go` file is shipped untested by automation.

## The systemic fix: don't write your own PTY layer

The narrower lesson — pair every constructor with a destructor — is genuinely
useful. The broader lesson is that a custom cross-platform PTY layer is
almost never worth the maintenance cost for a TUI project. There are two
mature options that handle the ConPTY edge cases for you: `creack/pty` plus
`UserExist/conpty` on the Go side, or `portable-pty` from the
`wezterm` ecosystem if you're in Rust. Both of these have been hit by every
weird Windows process-tree behavior you can imagine, and both have a backlog
of public issues documenting the workarounds. A new PTY package, written from
scratch, will rediscover those same issues one production incident at a time.

Crush's authors have a reasonable reason to want their own layer — Charm has a
strong "minimize external dependencies" culture and the PTY package is small
enough that the dep audit cost feels disproportionate. But the cost calculus
should include "we will hit a Windows process leak in the field within six
months of shipping this," and the question becomes: is shaving 200kb of
dependency surface worth that? For most projects the answer is no. The
correct review verdict here, accordingly, is `request-changes`: not because
the design is bad, but because the Windows backend is simultaneously
load-bearing and entirely untested, and the one place the Windows untestedness
visibly manifests is a clear correctness bug.

## A second-order observation: the omnibus problem

PR `crush#2606` is +3085/-0 across 15 files in three new packages. None of the
three packages has a consumer in the same PR — `split/`, `tabmgr/`, and `pty/`
are pure infrastructure with no caller. This is what I'd call an "omnibus
infrastructure PR," and it is a reviewability anti-pattern even when the
individual packages are well-shaped.

The reason omnibus PRs are bad has nothing to do with the actual code: it's
that the reviewer's attention budget is fixed. A reviewer who has 30 minutes
to spend on a PR will spread that 30 minutes across whatever the diff
contains. A 3000-line diff gets ten lines per minute of attention. A
500-line diff gets sixty. The Windows close-leak in `pty.go` is exactly the
kind of bug that a 30-minute reviewer on a 500-line diff would catch
("hmm, the only platform check in Close is `if cmd != nil`, what does that
mean for Windows?") but that a 30-minute reviewer on a 3000-line diff
would skim past.

The right shape for this work is three separate PRs in dependency order:
`pty/` first with a tiny `cmd/pty-demo` consumer that exercises both
backends in a per-platform CI matrix; then `split/` standalone with its
test peer; then `tabmgr/` standalone. Each PR is reviewable in fifteen
minutes, each one's bugs get caught in review rather than in production,
and the bisect surface when a regression lands later is one package
instead of three.

This is also why my review verdict on `#2606` was `request-changes` rather
than `merge-after-nits`. The ConPTY close bug is fixable with five lines.
The omnibus shape is fixable only by splitting and re-opening, which is a
fundamentally larger change to the PR. Reviewers can tolerate one of those
problems at a time; both together is too much to swallow with a "ship after
nits" verdict.

## Closing: the recurring asymmetric-cleanup pattern

If I had to name one cross-platform anti-pattern that I see at least once a
week in OSS reviews, it is asymmetric cleanup in a symmetric-looking
abstraction. The constructor takes pains to handle both platforms
explicitly. The destructor only handles the platform the author was
running on. The type system does not catch it because cleanup methods
typically return `error` (so `nil` is a valid no-op), and the test suite
does not catch it because the test runner only exercises one platform.

The detection cost is low — grep for `if x != nil` in a `Close`/`Stop`/`Drop`
method, find the matching field assignment in the platform-specific
constructor, ask whether the unstated else needs to do anything. The fix
cost is low — usually a handful of lines. The miss cost, on the other hand,
is high: silent process leaks in production on the platform that nobody
on the dev team is using day-to-day. Make this check part of your review
checklist for any PR that introduces or modifies a cross-platform
abstraction with per-OS backends. It will catch one of these every couple
of months, and each catch is worth more than the time spent on the
checklist for the rest of the year.
