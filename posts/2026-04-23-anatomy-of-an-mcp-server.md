---
title: "The anatomy of an MCP server: protocol, transport, tools, and the parts you can skip"
date: 2026-04-23
tags: [mcp, model-context-protocol, tools, agents, architecture]
est_reading_time: 15 min
---

## TL;DR

A Model Context Protocol server is, at its core, a long-running process that speaks JSON-RPC over stdio (or sometimes WebSocket) and exposes a small set of named capabilities: tools (functions the agent can call), resources (data the agent can read), and prompts (parameterized templates the agent can use). The protocol is small, the transport is boring, and most of the complexity in real MCP servers comes from the application logic, not from the protocol layer. This post is a tour of the protocol from the ground up: how the message format works, how a minimal server fits in 100 lines, what each of the three primitives is for, and which parts of the spec you can ignore for 80 percent of useful servers. Built around a runnable Python implementation that you can extend.

The framing: MCP is a thin wrapper around "expose a function to a model client" with a standard handshake and discovery mechanism. Treat it that way and it stops being mysterious.

## Section 1: why MCP exists

Before MCP, every coding agent and every LLM client invented its own way of describing tools. The OpenAI function-calling format and the Anthropic tool-use format were similar but not identical; the surrounding plumbing (where the tool list lives, how to register a new tool, how to invoke a tool from the agent) was completely different across clients. If you wrote a useful tool, you had to write a separate adapter for every client you wanted to support.

MCP standardizes this. A server written once works with any MCP-aware client. The client handles the model communication; the server handles the tool implementation; the protocol handles the boundary.

This is not a new idea. It is the same idea as the Language Server Protocol (LSP) for IDEs, which standardized "how does an editor talk to a language analyzer" so that one Python language server could be used by every editor that supports LSP. MCP is to AI agents what LSP is to editors. The wire format is even similar (JSON-RPC for both).

The practical implication: if you have a tool you want to expose to multiple AI clients, MCP is the right home for it. If you only have one client, MCP is overhead and you can just use that client's native tool format.

## Section 2: the wire format

MCP uses JSON-RPC 2.0 as its wire format. Each message is a JSON object with a few standard fields:

```json
{"jsonrpc": "2.0", "id": 1, "method": "tools/list", "params": {}}
```

The server replies:

```json
{"jsonrpc": "2.0", "id": 1, "result": {"tools": [...]}}
```

Or, for an error:

```json
{"jsonrpc": "2.0", "id": 1, "error": {"code": -32601, "message": "Method not found"}}
```

Notifications (no `id`, no response expected) are also valid:

```json
{"jsonrpc": "2.0", "method": "notifications/initialized"}
```

That is the entire transport-level protocol. Everything else is method names and parameter shapes.

The transport for stdio servers is line-delimited: each JSON object on its own line, no length prefix. This is the simplest case and the one most servers use. For WebSocket transports, the protocol is the same but each frame holds one JSON-RPC message.

## Section 3: the handshake

Before any real work, the client and server perform an `initialize` handshake. The client sends:

```json
{
  "jsonrpc": "2.0", "id": 1, "method": "initialize",
  "params": {
    "protocolVersion": "2024-11-05",
    "capabilities": {"tools": {}, "resources": {}},
    "clientInfo": {"name": "example-client", "version": "1.0"}
  }
}
```

The server replies with its own capabilities:

```json
{
  "jsonrpc": "2.0", "id": 1,
  "result": {
    "protocolVersion": "2024-11-05",
    "capabilities": {"tools": {"listChanged": true}},
    "serverInfo": {"name": "my-server", "version": "1.0"}
  }
}
```

Then the client sends a notification confirming initialization:

```json
{"jsonrpc": "2.0", "method": "notifications/initialized"}
```

After this, normal method calls can proceed. The handshake exists so that client and server can negotiate which capabilities are mutually supported. In practice, most servers support `tools` and skip `resources` and `prompts`; most clients accept whatever the server advertises and ignore the rest.

## Section 4: a minimal server in 100 lines

Here is a working MCP server that exposes one tool. It uses no MCP-specific library, just stdin/stdout and `json`. This is intentional, to show that the protocol is not magic.

```python
#!/usr/bin/env python3
"""Minimal MCP server. Exposes a single 'echo' tool."""
import json, sys

PROTOCOL_VERSION = "2024-11-05"
SERVER_INFO = {"name": "minimal-mcp", "version": "0.1.0"}

TOOLS = [
    {
        "name": "echo",
        "description": "Echo the input string back to the caller.",
        "inputSchema": {
            "type": "object",
            "properties": {"text": {"type": "string"}},
            "required": ["text"],
        },
    },
]

def handle(req: dict) -> dict | None:
    method = req.get("method")
    rid = req.get("id")
    params = req.get("params") or {}

    if method == "initialize":
        return {
            "jsonrpc": "2.0", "id": rid,
            "result": {
                "protocolVersion": PROTOCOL_VERSION,
                "capabilities": {"tools": {}},
                "serverInfo": SERVER_INFO,
            },
        }

    if method == "notifications/initialized":
        return None  # notifications get no reply

    if method == "tools/list":
        return {"jsonrpc": "2.0", "id": rid, "result": {"tools": TOOLS}}

    if method == "tools/call":
        name = params.get("name")
        args = params.get("arguments") or {}
        if name == "echo":
            text = args.get("text", "")
            return {
                "jsonrpc": "2.0", "id": rid,
                "result": {
                    "content": [{"type": "text", "text": text}],
                    "isError": False,
                },
            }
        return {
            "jsonrpc": "2.0", "id": rid,
            "error": {"code": -32602, "message": f"Unknown tool: {name}"},
        }

    return {
        "jsonrpc": "2.0", "id": rid,
        "error": {"code": -32601, "message": f"Method not found: {method}"},
    }

def main():
    for line in sys.stdin:
        line = line.strip()
        if not line: continue
        try:
            req = json.loads(line)
        except json.JSONDecodeError:
            continue
        resp = handle(req)
        if resp is not None:
            sys.stdout.write(json.dumps(resp) + "\n")
            sys.stdout.flush()

if __name__ == "__main__":
    main()
```

Save as `mcp-minimal.py`, make it executable. To test it without an MCP client, you can pipe JSON-RPC messages to it directly:

```bash
{
  echo '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"test","version":"1"}}}'
  echo '{"jsonrpc":"2.0","method":"notifications/initialized"}'
  echo '{"jsonrpc":"2.0","id":2,"method":"tools/list","params":{}}'
  echo '{"jsonrpc":"2.0","id":3,"method":"tools/call","params":{"name":"echo","arguments":{"text":"hello"}}}'
} | python3 mcp-minimal.py
```

You will see the four responses, one per line. That is a complete MCP server in 60 lines of executable Python.

## Section 5: the three primitives

MCP defines three kinds of capability a server can expose. Most servers use only the first.

**Tools.** Functions the agent can invoke. Each tool has a name, a description, an input schema (JSON Schema), and an implementation. The agent decides when to call which tool based on the description. The tool returns one or more content blocks (text, image, embedded resource).

**Resources.** Data the agent can read by URI. Each resource has a URI, a name, a description, and a content type. The server lists resources via `resources/list` and serves their content via `resources/read`. This is for "the agent can browse the filesystem" or "the agent can read this database table" patterns where the data is large and you do not want to inline it as a tool result.

**Prompts.** Parameterized templates the agent can use. Each prompt has a name, a description, an argument list, and a template body. The client lists prompts via `prompts/list` and instantiates them via `prompts/get`. This is for "give the user a quick way to invoke a common task" patterns: the IDE shows a menu of prompts, the user picks one, the prompt fills in.

In practice: 80 percent of servers I have seen expose only tools. Resources are used by some servers that wrap databases or filesystems. Prompts are rarely used in my experience; their value is in client integration (showing them as menu items) and most clients do not surface them well.

For the rest of this post I will focus on tools.

## Section 6: tool schema design, the part that matters

The tool's `inputSchema` is the contract between the agent and your code. The model reads the schema (and the description) to decide whether to call the tool and what to pass it. A bad schema produces a tool the model cannot use correctly.

Good schema design rules I have settled on:

The first rule: every property has a `description`. The model uses descriptions, not just type names. A property named `path` of type `string` with no description is ambiguous. A property named `path` with description `"Absolute filesystem path to read. Must start with /."` is unambiguous.

The second rule: use `enum` when the input is one of a small set. Free-form strings are harder for the model to constrain. `{"type": "string", "enum": ["read", "write", "delete"]}` produces fewer wrong calls than `{"type": "string", "description": "Operation to perform"}`.

The third rule: keep schemas shallow. Deeply nested objects are harder for the model to construct correctly. If your tool needs many parameters, consider splitting it into multiple tools instead.

The fourth rule: write the description as if explaining to a new hire. The model reads it the same way. "Reads a file from disk" is a category. "Reads the contents of a UTF-8 text file from the local filesystem and returns the contents as a string. Fails if the file does not exist or is binary." is a specification.

Here is a tool schema that follows these rules:

```python
{
    "name": "read_file",
    "description": (
        "Read the full contents of a UTF-8 text file from the local filesystem. "
        "Returns the contents as a string. Fails if the file does not exist, is "
        "binary, or is larger than 1 MB. For larger files, use 'read_file_lines' "
        "with explicit start and end line numbers."
    ),
    "inputSchema": {
        "type": "object",
        "properties": {
            "path": {
                "type": "string",
                "description": "Absolute filesystem path. Must start with '/'. Example: '/tmp/notes.txt'.",
            },
        },
        "required": ["path"],
    },
}
```

This level of description detail feels excessive when you are writing the tool. It pays for itself the first time you use the tool with a model that has not seen your codebase.

## Section 7: error responses, the often-overlooked layer

Tool calls can fail. The protocol distinguishes between two kinds of failure:

The first: a tool was called correctly but the operation itself failed (file not found, network error, invalid input that passed schema but failed business logic). For these, return a normal `tools/call` result with `isError: true` and a description of what went wrong:

```python
return {
    "jsonrpc": "2.0", "id": rid,
    "result": {
        "content": [{"type": "text", "text": f"File not found: {path}"}],
        "isError": True,
    },
}
```

The agent sees the error in the tool result and can decide what to do. This is the case for most failures.

The second: the server itself encountered a protocol-level problem (unknown method, malformed parameters, internal exception). For these, return a JSON-RPC error:

```python
return {
    "jsonrpc": "2.0", "id": rid,
    "error": {"code": -32603, "message": "Internal server error"},
}
```

The client treats these as protocol violations and may surface them differently to the user. They should be rare; tool-level failures should use the first form.

A subtle point: returning a JSON-RPC error for what is really a tool-level failure (file not found) breaks the agent's ability to recover. The agent expects tool calls to succeed at the protocol level even when the underlying operation fails. Reserve JSON-RPC errors for genuine protocol issues.

## Section 8: testing without a real client

Before integrating with a real MCP client (Claude Desktop, an IDE plugin, an agent harness), test the server in isolation. Two patterns that work well:

The first: a script that pipes pre-recorded JSON-RPC sessions and checks the responses. This is essentially a integration-test runner specialized for JSON-RPC.

```python
import json, subprocess

SCRIPT = [
    {"req": {"jsonrpc": "2.0", "id": 1, "method": "initialize",
             "params": {"protocolVersion": "2024-11-05", "capabilities": {},
                        "clientInfo": {"name": "test", "version": "1"}}},
     "expect_id": 1},
    {"req": {"jsonrpc": "2.0", "method": "notifications/initialized"},
     "expect_id": None},
    {"req": {"jsonrpc": "2.0", "id": 2, "method": "tools/list", "params": {}},
     "expect_id": 2},
]

def test():
    proc = subprocess.Popen(
        ["python3", "mcp-minimal.py"],
        stdin=subprocess.PIPE, stdout=subprocess.PIPE, text=True,
    )
    for step in SCRIPT:
        proc.stdin.write(json.dumps(step["req"]) + "\n")
        proc.stdin.flush()
        if step["expect_id"] is not None:
            line = proc.stdout.readline()
            resp = json.loads(line)
            assert resp["id"] == step["expect_id"], resp
            assert "result" in resp, resp
    proc.stdin.close()
    proc.wait()
    print("ok")

test()
```

The second: an interactive REPL that lets you fire individual requests and see responses. The official MCP project ships an "inspector" tool for this; for ad-hoc work, a four-line Python REPL is enough.

```python
import json, subprocess
proc = subprocess.Popen(["python3", "mcp-minimal.py"], stdin=subprocess.PIPE, stdout=subprocess.PIPE, text=True)
def call(req): proc.stdin.write(json.dumps(req) + "\n"); proc.stdin.flush(); return json.loads(proc.stdout.readline())
print(call({"jsonrpc":"2.0","id":1,"method":"tools/list","params":{}}))
```

Both let you iterate on the server without restarting an entire client.

## Section 9: the parts of the spec you can skip

The MCP spec is reasonably large (several pages of method definitions) but most of it is optional. A working production server can ignore:

- **Resources**, unless you specifically need URI-addressable data. For most tool-only use cases, skip.
- **Prompts**, for the same reason. Add them if your client surfaces them well.
- **Sampling**, where the server can ask the client to make model calls on its behalf. Powerful, rarely needed for typical tool servers.
- **Logging**, the structured log shipping protocol. Useful for hosted servers; for local stdio servers, just write to stderr.
- **Progress notifications**, for long-running tool calls. Only matters if you have tool calls that take more than a few seconds and you want to show progress to the user.
- **Roots**, where the client tells the server which filesystem paths the user has authorized. Important for security-sensitive servers; skippable for trust-the-user setups.

Implementing only `initialize`, `tools/list`, and `tools/call` gets you a server that works with every MCP-aware client I have tried. Add more only when you have a specific reason.

## Section 10: what I have built with this

Three small servers I run locally, all tool-only, all under 300 lines:

The first: a `git-mcp` server that exposes `git status`, `git log`, `git diff`, and `git show` as tools. The agent can ask "what changed in the last commit" and the server returns the diff. Total implementation: about 150 lines, mostly subprocess management.

The second: a `notes-mcp` server that wraps a directory of Markdown files. Tools are `list_notes`, `read_note`, `search_notes`, `create_note`. The agent uses it as a long-term memory across sessions.

The third: a `journal-mcp` server that wraps the JSONL telemetry pipelines from the earlier post. Tools are `query_events`, `summarize_day`, `cost_breakdown`. The agent uses it to answer "how much did I spend on input tokens yesterday" without me writing a one-off `jq` pipeline.

None of these are clever. All of them are useful daily. The pattern is: identify a small set of read or write operations against some local resource, wrap each operation as a tool, expose them via MCP. Total time per server: an afternoon.

The MCP protocol is the smallest thing it could be, which is exactly why it works. Anything more elaborate would have higher friction and lower adoption. The boring shape, again.

## References

- MCP specification: https://spec.modelcontextprotocol.io/
- MCP Python SDK: https://github.com/modelcontextprotocol/python-sdk
- JSON-RPC 2.0 spec: https://www.jsonrpc.org/specification
- Language Server Protocol (the inspiration): https://microsoft.github.io/language-server-protocol/
- MCP example servers: https://github.com/modelcontextprotocol/servers
