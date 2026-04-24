# The Dedup-Set Pattern Keyed On EvalSymlinks

Most discovery layers I have built or audited in the last year double-load at least one path and never notice. The bug is silent: nothing crashes, nothing throws, the work just runs twice. You only catch it when a downstream counter looks suspicious — "why are there 14 plugin loads when I have 7 plugins?" — or when a cache hit-rate drops because every entry is being installed under two distinct keys that resolve to the same file.

The fix is small and unglamorous: dedup using a set keyed on the canonical, symlink-resolved absolute path. In Go that is `filepath.EvalSymlinks` after `filepath.Abs`. In Python it is `os.path.realpath`. In Node it is `fs.realpathSync.native`. The discipline is the same regardless of language, and the failure mode is the same regardless of language. This post walks through why string-equality dedup is not enough, why `Abs` alone is not enough, why hashing file contents is the wrong layer, and how to wire the canonicalization step into a discovery pipeline so it is impossible to forget.

## The shape of the bug

A discovery layer takes a list of "places to look" and returns a list of "things found." Plugins, config fragments, prompts, MCP server manifests, skill directories, codemod packs, agent profiles — they all follow this shape. The places-to-look list almost always contains overlapping entries:

- A user-level directory and a project-level directory that both, on this machine, resolve to the same path because the project copy was symlinked into the user dir for editing.
- Two environment variables that both contain `~/.config/foo`, one with the `~` expanded by the shell and one not.
- A "primary" path and a "fallback" path where the fallback was set up as a symlink to the primary three years ago and nobody remembers.
- A walk that descends into a directory which contains a symlink back to one of its ancestors, producing the same leaf file under two different traversal paths.
- On macOS, `/var` and `/private/var`. `/tmp` and `/private/tmp`. `/etc` and `/private/etc`. The system has been resolving these silently since the BSD days and most discovery code never accounts for it.

If your dedup set is keyed on the raw string the caller passed in, none of these collide. Two distinct strings, two distinct set members, two distinct loads. The loader runs twice, the registration logic either errors loudly (best case), silently overwrites (medium case), or silently appends a duplicate (worst case). Worst case is the most common, because most registration logic is a slice append and nobody put a uniqueness check on the registry itself — they assumed the discovery layer would not hand them duplicates.

## Why `Abs` is not enough

The first instinct when you notice the bug is to normalize. So you reach for `filepath.Abs` (or `os.path.abspath`, or `path.resolve`) and dedup on that. This catches the `~` case, it catches the relative-path case, it catches the trailing-slash case. It does not catch symlinks. `Abs` only resolves `..` and `.` and prepends the current working directory; it does not touch any symlink in the path.

So if `~/.config/foo` is a symlink to `~/Projects/foo/config`, `Abs` will give you back `/Users/you/.config/foo` for one input and `/Users/you/Projects/foo/config` for the other. Two distinct strings, dedup set passes both through, you load twice. The `Abs` step is necessary but not sufficient — you still need the symlink resolution.

The reverse is also true: `EvalSymlinks` alone is not sufficient either, because on most platforms `EvalSymlinks` of a relative path will resolve relative to the process's current working directory, which is not what you want if the caller passed you a path relative to some other root. The correct pipeline is always `Abs` first, then `EvalSymlinks` on the absolute result. In Go:

```go
abs, err := filepath.Abs(input)
if err != nil { return "", err }
canon, err := filepath.EvalSymlinks(abs)
if err != nil {
    if errors.Is(err, fs.ErrNotExist) {
        // fall back to abs — the path doesn't exist yet,
        // which is fine for "intent" dedup but not for "load" dedup
        return abs, nil
    }
    return "", err
}
return canon, nil
```

The `ErrNotExist` branch matters and is where most implementations cut corners. If you are deduping intended-load paths before checking existence, you cannot resolve symlinks on a non-existent file — the syscall errors. You have two choices: either drop non-existent paths from the dedup set entirely (correct if the next stage is "open the file"), or fall back to the absolute non-canonicalized path (correct if the next stage tolerates non-existence and you care about catching duplicate intentions). Pick the one that matches your downstream contract and document it. Do not silently swallow the error and return the empty string; I have debugged that exact bug three times.

## Why hashing contents is the wrong layer

The other instinct, especially from people coming from a content-addressable-storage background, is to hash the file contents and dedup on the hash. This is wrong for discovery. It is right for caching, it is right for deduplication-at-rest, it is wrong for "should I load this file twice?"

Reasons it is wrong:

1. You have to read the file to hash it. If your discovery layer is enumerating thousands of candidates, you are doing thousands of reads just to decide what to read. The whole point of dedup at the discovery layer is to avoid the read.
2. Two files with identical contents are not the same file. They might be intentionally duplicated, registered under different names, mounted with different permissions, or be the canonical copy in two different namespaces. Loading one and skipping the other can change behavior in ways the user did not consent to.
3. Hashing introduces a TOCTOU window. The file you hashed at discovery time is not necessarily the file you load three milliseconds later. With path canonicalization the worst that happens is the symlink retargets and you load a different file — but you only load it once.
4. Content-hash dedup cannot catch the case where the loader has side effects keyed on the path. If the loader registers the plugin under a name derived from the directory it lived in, two paths that point at the same file produce two registry entries with two different names. Content hashing will not catch this. Path canonicalization will.

Hashing has its place — verifying that two `EvalSymlinks` results that surprised you really do point at the same bytes is a fine thing to do at debug time. It is not the dedup key.

## The pattern

The pattern is small enough to fit in one function. In Go, where I have written this most often:

```go
type discovered struct {
    canonical string
    original  string // for error messages
    source    string // which root this came from, for diagnostics
}

func discover(roots []string, predicate func(string) bool) ([]discovered, error) {
    seen := make(map[string]discovered)
    for _, root := range roots {
        err := filepath.WalkDir(root, func(p string, d fs.DirEntry, err error) error {
            if err != nil { return nil } // tolerate permission errors per-entry
            if d.IsDir() { return nil }
            if !predicate(p) { return nil }
            canon, err := canonicalize(p)
            if err != nil { return nil }
            if _, ok := seen[canon]; ok {
                // already discovered through another root; first wins
                return nil
            }
            seen[canon] = discovered{canonical: canon, original: p, source: root}
            return nil
        })
        if err != nil { return nil, err }
    }
    out := make([]discovered, 0, len(seen))
    for _, d := range seen { out = append(out, d) }
    return out, nil
}
```

A few things in this sketch are deliberate:

**The map value is a struct, not a bool.** When the dedup fires and you decide to skip a path, you want to be able to log "I skipped `/Users/you/.config/foo/bar.yaml` because it canonicalizes to the same path as `/Users/you/Projects/foo/config/bar.yaml`." A bool-valued set throws that information away. A struct-valued map keeps it cheap to print at debug time.

**First wins, not last wins.** This is a policy decision and you must make it deliberately. The convention in most discovery layers is that the higher-priority root is listed first, so first-wins gives the higher-priority root the canonical entry. If your roots are in arbitrary order, you should sort them by priority before calling `discover`, not flip the wins-rule. If you flip to last-wins to "fix" a bug, you have probably just reintroduced the original duplicate-load problem in a different shape — the caller above you is now sensitive to root order in a way it was not before.

**Per-entry errors are tolerated, walk errors are not.** A single unreadable file should not abort the whole discovery. A walk that cannot start (root does not exist, root is unreadable) is a real error. The two error channels are different and should be handled differently. The most common mistake here is to bail on the first per-entry permission error and then wonder why discovery returns nothing on a system where one stale symlink in a deep subdir is broken.

**The predicate is separate from the canonicalization.** Do not fold the file-type check into `canonicalize`. The canonicalization step should be pure: input path, output canonical path, no decisions about what counts as a "real" plugin. Keeping these separate is what lets you reuse the same canonicalize function across every discovery layer in the codebase.

## Wiring it so it is impossible to forget

A pattern that lives in one function in one file gets forgotten the moment a second discovery layer is added. The way to keep this honest is to push the canonicalization into the type system.

Define a `CanonicalPath` type that is not constructible except through the canonicalize function:

```go
type CanonicalPath struct{ s string }

func (c CanonicalPath) String() string { return c.s }

func Canonicalize(p string) (CanonicalPath, error) {
    abs, err := filepath.Abs(p)
    if err != nil { return CanonicalPath{}, err }
    canon, err := filepath.EvalSymlinks(abs)
    if err != nil {
        if errors.Is(err, fs.ErrNotExist) { return CanonicalPath{s: abs}, nil }
        return CanonicalPath{}, err
    }
    return CanonicalPath{s: canon}, nil
}
```

Now make every registry, every cache, every dedup set in the codebase take `CanonicalPath` as its key. The compiler will refuse to let you put a raw string in. Anyone adding a new discovery layer has to call `Canonicalize` to get a key, and they cannot accidentally key on the raw input. This costs you maybe forty lines of code total and removes an entire category of bug from the codebase forever.

Python does not have this trick available cleanly, but you can fake it with a `NewType` and a discipline of "no one constructs `CanonicalPath` without calling `canonicalize`." It is weaker than the Go version but it still gives you a grep target: any `CanonicalPath(` that is not inside `canonicalize` is a bug.

Node has it via TypeScript branded types, same idea, same caveats.

## Caching the canonicalize call

`EvalSymlinks` is not free. It walks the path component by component, doing a `lstat` on each, and following any symlink it encounters. On a deeply nested path with several symlinks in the chain, this is dozens of syscalls. If your discovery layer is enumerating ten thousand files and they share long common prefixes, you are doing the same prefix-resolution work over and over.

The fix is a small LRU keyed on the input path, returning the canonical path. The cache is process-local, the entries are small, and the hit rate on a typical discovery run is above 95%. Do not try to be clever about invalidation — the cache is short-lived (one discovery run) and the symlink graph does not change underneath you in any way you care about during that window. If it does, you have other problems.

Do not cache across processes. Symlinks change, mounts change, path entries get rewritten. The cache only pays off within a single run.

## What this looks like in practice

A real discovery layer I worked on recently was loading skill manifests from three roots: a system root, a user root, and a project root. The user had symlinked one skill directory from the user root into the project root because they were editing it in place. Without canonicalization, the skill registered twice. The second registration overwrote the first silently because the registry was a map keyed on skill name. So far, so apparently-fine.

The bug surfaced when the skill had a side effect at load time — it registered an MCP server. That registration was append-only, not map-overwrite. So the MCP server appeared twice. The dispatcher then tried to start both, the second one collided on the same Unix socket as the first, and the whole subsystem failed to come up. The error message pointed at the socket collision, which pointed at MCP server config, which pointed at the skill loader, which pointed at — eventually — the discovery layer.

The fix was four lines: import `path/filepath`, change the dedup set's key from the input string to `filepath.EvalSymlinks(filepath.Abs(p))`, add the `ErrNotExist` fallback, write a test that creates a symlink loop in a `t.TempDir()` and asserts the loader runs once. Total time to write and ship: under an hour. Total time the bug had been latent: about eight months. The latency is the lesson — this class of bug does not fail loudly until something downstream of the discovery layer becomes sensitive to duplication, and by then the connection back to the discovery layer is several stack frames removed and not obvious.

## Tests that actually catch this

A unit test that "tests dedup" by passing the same string twice and asserting one result is worse than no test, because it gives you false confidence. The string-equality case is the case that already worked. The cases that actually need testing:

1. Two roots where one is a symlink to the other. Walk both, assert one result.
2. A root containing a file plus a symlink to that file. Walk the root, assert one result.
3. A root containing a symlink loop. Walk the root, assert it terminates and returns finite results. (`filepath.WalkDir` handles this in Go because it does not follow symlinks during the walk by default; verify your language's equivalent does the same or guard against it.)
4. On macOS specifically, a path under `/tmp` and the same path under `/private/tmp`. Assert they collide. This is a one-liner test that catches an entire family of platform-specific bugs.
5. A non-existent path. Assert the canonicalize step does not error out and the path is either dropped or kept-as-absolute according to your documented policy.
6. A path with `..` in the middle that traverses through a symlink. This is the trickiest one and the one most implementations get wrong: `Abs` resolves the `..` lexically, before any symlink resolution, which can give a different answer than resolving `..` after following the symlink. POSIX says lexical resolution is correct here; some shells disagree. Pin the behavior in a test so future-you does not "fix" it.

The whole test suite for canonicalization is maybe seventy lines of Go. It does not need mocks; `t.TempDir` plus `os.Symlink` plus `os.WriteFile` covers everything. If the canonicalize function lives in its own package and is the only way to construct a `CanonicalPath`, these tests are the only place this logic needs to be exercised. Every other test in the codebase gets canonicalization correctness for free.

## Closing

The dedup-set-keyed-on-canonical-path pattern is one of the highest leverage hundred lines of code you can add to a discovery-heavy system. The cost is small, the bug class it eliminates is large, and the failure mode it prevents is the worst kind: silent, latent, and surfaces three layers downstream from the actual fault. Push the canonicalization into a type, key every dedup set on that type, write the six tests above, and this entire family of bugs goes away and stays gone.
