---
title: "Reverse-engineering closed-source CLIs ethically: what to do, what to never do"
date: 2026-04-23
tags: [reverse-engineering, ethics, cli, debugging, observability]
est_reading_time: 14 min
---

## TL;DR

You can learn a lot about a closed-source CLI tool without violating its license, its terms of service, or any reasonable ethical line. The right techniques are the ones that observe the tool's behavior at boundaries the tool already exposes: its filesystem footprint, its network calls, its environment variables, its stdout and stderr, its exit codes, its config files. The wrong techniques are the ones that look inside the binary, bypass its license enforcement, or extract paid functionality for re-use. This post documents the ethical workflow I follow when I need to integrate with or debug a CLI whose source I cannot see, with concrete commands and a checklist of lines I will not cross.

The framing: treat the closed-source binary the way you would treat a third-party HTTP API. Observe its public-facing behavior, document it, build on it. Do not read its source, even if technically possible.

## Section 1: why this needs an explicit ethics statement

Reverse engineering has a long history of legitimate use: interoperability, debugging, security research, accessibility, and learning. It also has a long history of being used as a euphemism for "I want this paid software for free." The two are easy to conflate, and a careless engineer can drift from one to the other without noticing.

This post is about the first kind, with the line drawn explicitly so I do not drift. The legitimate goals I have for reverse-engineering a closed-source CLI are:

1. Integrate it into a pipeline that needs to invoke it programmatically.
2. Debug a misbehavior that the tool's documentation does not explain.
3. Understand the on-disk format the tool writes, so I can back it up or migrate off.
4. Build a wrapper that adds telemetry, retry, or logging the tool itself does not provide.

The illegitimate goals I will not pursue, even when technically feasible:

1. Bypass license enforcement (free trial limits, seat limits, feature gates).
2. Extract or replicate paid functionality in a way that competes with the original.
3. Distribute disassembled binaries or decompiled source.
4. Violate the tool's terms of service, even in ways the vendor would not detect.

The techniques in this post are aimed at the first list and explicitly do not enable the second. Several of them could be misused; that is true of any debugging technique. The mitigation is the explicit line, not the absence of the technique.

## Section 2: the order I work in

When I pick up a new closed-source CLI to integrate, I work in this order. Each step is cheaper and more ethical than the next, so I get as far as I can on the cheap steps before going further.

1. Read the public documentation, including the man page if there is one.
2. Run `--help`, `--version`, and any subcommand `help` text. Document the surface.
3. Run the tool with verbose flags. Most CLIs have a `-v` or `--debug` mode.
4. Observe the tool's filesystem and network footprint while running.
5. Read its config files and on-disk artifacts.
6. Read its log output at the highest verbosity.
7. Wrap it in a thin shim that captures inputs and outputs.

I almost never go past step 7. The remaining steps (looking at the binary itself, decompiling, hooking system calls) are where the ethical risk concentrates and where the documentation rules above start to matter most.

## Section 3: documenting the help surface

The first artifact is a flat text file capturing the entire `--help` tree. This is mechanical:

```bash
#!/bin/bash
# dump-help.sh CLI_NAME
CLI=$1
out=help-$CLI.txt
{
  echo "=== $CLI --version ==="
  $CLI --version 2>&1 || true
  echo
  echo "=== $CLI --help ==="
  $CLI --help 2>&1 || true
  echo
  for sub in $($CLI --help 2>&1 | awk '/^  [a-z]/ {print $1}' | head -50); do
    echo
    echo "=== $CLI $sub --help ==="
    $CLI $sub --help 2>&1 || true
  done
} > "$out"
echo "wrote $out"
```

This usually surfaces 80 percent of the surface area in a single file you can grep. The auto-extracted subcommand list is approximate (CLIs format help differently), but it works for tools that follow common conventions.

Why this matters: it forces you to confront the tool's own documentation before assuming it is missing. A surprising number of "undocumented" features are documented in subcommand help text that nobody reads.

## Section 4: filesystem footprint

When the tool runs, what does it touch on disk? On macOS, the safest answer comes from `fs_usage`, which reports filesystem syscalls. It requires sudo and works on stock macOS without installing anything.

```bash
sudo fs_usage -w -f filesys $(pgrep -f the-cli) 2>/dev/null | \
  grep -v "fs_usage" | \
  awk '{print $5}' | \
  sort -u
```

Or, more thoroughly, with `dtruss` (also macOS, also sudo):

```bash
sudo dtruss -t open -f the-cli some-args 2>&1 | \
  grep -oE '"[^"]+"' | sort -u
```

On Linux, `strace` is the equivalent:

```bash
strace -e openat -f -o /tmp/the-cli.strace the-cli some-args
grep -oP 'openat\([^,]+, "\K[^"]+' /tmp/the-cli.strace | sort -u
```

What I look for: config file locations, cache directories, temp file patterns, and any files the tool reads from `~/.config/`, `~/Library/`, or `~/.cache/`. Document these. They are the tool's persistent state, and understanding them is usually the key to debugging persistence-related bugs.

This step is observation only. I am watching the tool perform actions it normally performs. I am not modifying it, not patching it, not bypassing anything. The same observation any user could make from looking at their disk after running the tool.

## Section 5: network footprint

CLIs that hit the network deserve special attention, both for security (what is being sent) and for debugging (what endpoint is timing out).

The simplest move: HTTPS proxy with `mitmproxy`, with the CLI's HTTP client trusting the proxy's CA cert.

```bash
# install mitmproxy: brew install mitmproxy
mitmweb --listen-port 8080 &

# point the CLI at the proxy. Most CLIs respect HTTPS_PROXY.
HTTPS_PROXY=http://localhost:8080 \
SSL_CERT_FILE=$HOME/.mitmproxy/mitmproxy-ca-cert.pem \
the-cli some-command
```

The proxy logs every HTTP request and response. You see the full URL, headers, request body, and response body. This is invaluable for understanding which endpoint a CLI hits, what it sends, and what it gets back.

Ethical note: this works because the CLI is making requests on your behalf, on a network you control, with credentials you control. You are not intercepting someone else's traffic. You are inspecting your own. That is a meaningful distinction.

If the CLI uses certificate pinning and refuses to trust your proxy, that is the vendor's signal that they do not want you to inspect the traffic. Respect the signal. Move to a different technique. Do not patch the binary to disable pinning.

## Section 6: environment variables and config

Many CLIs read configuration from environment variables or files in known locations. The discovery technique:

```bash
# print all env vars referenced in the binary's strings
strings $(which the-cli) | grep -E '^[A-Z_][A-Z0-9_]{3,}$' | sort -u | head -50
```

This is literally just looking at strings in the binary. It does not disassemble, it does not decompile. The strings are present in any executable file and are usually a mix of error messages, format strings, and (relevant here) environment variable names.

Combine with the help text dump and the filesystem footprint, and you usually have a complete picture of the tool's configuration surface.

For config file formats: open the file in a text editor. If it is plaintext (TOML, YAML, JSON, INI), the format is self-documenting. If it is a binary format (SQLite, custom), use the appropriate inspection tool: `sqlite3 .schema` for SQLite, `file` for unknown binaries. SQLite is by far the most common; in three years of poking at CLIs, I have seen maybe two custom binary formats and both turned out to be Protocol Buffers with discoverable schemas.

## Section 7: the wrapper-shim pattern

Once you understand the tool's inputs and outputs, the highest-leverage move is to wrap it in a thin script that adds whatever the tool itself lacks: structured logging, retries, telemetry, output transformation.

```bash
#!/bin/bash
# the-cli-wrapped: wraps the-cli with timing and JSONL logging
LOG=~/.cache/cli-wrapper/the-cli.jsonl
mkdir -p $(dirname "$LOG")
START=$(date +%s.%N)
TMPOUT=$(mktemp)
TMPERR=$(mktemp)

the-cli "$@" > "$TMPOUT" 2> "$TMPERR"
EXIT=$?

END=$(date +%s.%N)
DUR=$(echo "$END - $START" | bc)

cat "$TMPOUT"
cat "$TMPERR" >&2

# JSONL log entry
python3 -c "
import json, sys
sys.stdout.write(json.dumps({
    'ts': $START,
    'cmd': 'the-cli',
    'args': $(printf '%s\n' "$@" | python3 -c 'import json,sys; print(json.dumps(sys.stdin.read().splitlines()))'),
    'exit': $EXIT,
    'dur_s': $DUR,
    'stdout_bytes': $(wc -c < "$TMPOUT"),
    'stderr_bytes': $(wc -c < "$TMPERR"),
}) + chr(10))
" >> "$LOG"

rm -f "$TMPOUT" "$TMPERR"
exit $EXIT
```

I now have a JSONL log of every invocation with timing and exit codes, queryable with the same `jq` patterns from the previous post. I can build retry logic on top of this, or alerting, or rate limiting. The original CLI is unchanged.

This is the pattern I reach for in 90 percent of cases. It adds the missing observability without ever touching the binary. The wrapper is mine, the CLI is the vendor's, the contract between them is the documented one.

## Section 8: the line I do not cross

There are several techniques I know exist and have used in academic contexts but will not use against a closed-source CLI in production:

I do not run a decompiler over the binary. Not Ghidra, not IDA, not Hopper. Even if the EULA permits it (some do, for "interoperability purposes"), the result is source code I am not licensed to redistribute or build on, and the cognitive risk of "I now know the algorithm, can I write something similar" is real. The clean line is to never have the source in front of me in the first place.

I do not patch the binary to remove license checks, even on software I have already paid for. If my license is broken, the right move is to contact the vendor. If the vendor is unresponsive, the right move is to switch tools, not to patch around the check.

I do not extract embedded models, prompts, or training data from binaries that ship them. These are usually the vendor's most valuable IP and are often encrypted; the encryption is a clear "do not look" signal.

I do not bypass certificate pinning. If the vendor pinned a cert, they have signaled they do not want third-party inspection of the network traffic. The respectful move is to find a different debugging path, not to defeat the pin.

I do not redistribute findings about a tool's internals in a way that would harm the vendor. If I discover that a tool's "premium" features are gated by a single environment variable, I do not write a blog post telling the world how to set it. I either contact the vendor (if it is a security issue) or just keep the information to myself (if it is a business model issue and not a security one).

These rules feel restrictive, but in practice they are the rules that keep my work both useful and defensible. Every integration I have built with closed-source tooling sits within them, and none of them has made my work meaningfully harder.

## Section 9: a worked example, end to end

To make this concrete: about a year ago I needed to integrate a closed-source command-line tool into a pipeline. The tool worked great interactively but had no documented programmatic API. Total time: about an afternoon. Total source code read: zero lines.

Step 1: dumped the help tree to a file. Found two undocumented flags by grepping for `--` in the strings output that did not appear in the help tree, both turned out to be debugging flags that produced JSON output. Useful, used, documented.

Step 2: ran `fs_usage` while the tool ran. Found that it wrote a SQLite database under `~/Library/Application Support/<vendor>/state.db`. Opened the database with `sqlite3`, ran `.schema` to see the tables, and confirmed that the data I needed was a straightforward join across three tables. Wrote a 30-line Python script that read the database directly for my read-only use case.

Step 3: confirmed via `mitmproxy` that the tool's "sync" command made a single POST to a documented endpoint with a documented JSON body. My pipeline already had a way to make HTTP requests, so I never needed the CLI for the sync step at all; I just made the HTTP call directly.

Step 4: wrote a bash wrapper that ran the CLI for the operations I still needed it for, captured timing and exit codes to a JSONL log, and added a single retry on transient network failures.

End result: a pipeline that uses the CLI where the CLI is the right tool, bypasses it where my own code is simpler, and observes everything via JSONL. The vendor's binary is unchanged. My code is mine. The license is intact. And when the vendor next ships an update, my integration either keeps working (because I did not depend on undocumented internals) or breaks loudly (because I depended on a documented surface that changed), in which case I update.

## Section 10: the discipline summary

The whole post is one paragraph if you compress it. Treat the binary as a black box. Inspect its public-facing behavior the way you would inspect a remote API. Build wrappers, not modifications. Keep yourself away from the binary's internals so that you cannot accidentally internalize the vendor's IP. Document what you find for your own future self, but do not publish anything that would harm the vendor's business model.

Done well, this is normal engineering hygiene. Done poorly, it is the start of a piracy story or a lawsuit. The difference is in habits, not in tools.

## References

- macOS `fs_usage(1)`: https://ss64.com/mac/fs_usage.html
- `mitmproxy` docs: https://docs.mitmproxy.org/stable/
- DMCA Section 1201 reverse engineering exemption (US): https://www.copyright.gov/1201/
- EU Software Directive Article 6 (interoperability): https://eur-lex.europa.eu/eli/dir/2009/24/oj
- `strace(1)`: https://man7.org/linux/man-pages/man1/strace.1.html
