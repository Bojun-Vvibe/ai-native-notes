---
title: "Streaming token UIs: making waiting feel like working"
date: 2026-04-23
tags: [ui, streaming, ux, llm, terminal, sse]
est_reading_time: 13 min
---

## TL;DR

Streaming a model's response to the user is a UX decision dressed up as a performance optimization. The actual time-to-completion is identical whether you stream or not; what changes is the user's perception, and perception is most of the experience. Done well, streaming makes a 30-second response feel like 2 seconds. Done poorly, it produces flickering text, broken markdown rendering, mid-word truncation, and a worse experience than just showing the final result. This post is the implementation guide for terminal and HTTP streaming UIs that I have shipped, with the four UX principles that separate good streaming from bad and the technical patterns that make each work.

The framing: streaming is not "show tokens as they arrive." Streaming is "give the user a sense of progress that maps to the work being done." The token boundary is one rendering choice among several; sometimes it is the wrong one.

## Section 1: why streaming matters more than it should

The math says streaming is a wash. A model that takes 8 seconds to produce a 200-token response delivers the same total content whether you wait for the end or watch tokens arrive. The first token arrives at roughly 500ms either way; the last token arrives at 8 seconds either way.

But the perception is not a wash. A user staring at a spinner for 8 seconds gives up at 4 seconds and assumes the system is broken. A user watching tokens arrive over those same 8 seconds reads as the response unfolds and finishes reading roughly when the response finishes generating. The two experiences end at the same wall-clock time but the second one feels half as long because the user was doing something (reading) for most of it instead of nothing (waiting).

Jakob Nielsen's classic latency thresholds: 0.1 seconds feels instant, 1 second is the threshold for keeping a user's flow uninterrupted, 10 seconds is the threshold past which users give up. Streaming pulls a 10-second response down past the 1-second threshold for the user's perception, while changing the actual numbers not at all. That is a 10x perceived speedup with a 0x actual cost. Hence the "matters more than it should" framing.

## Section 2: the four UX principles

Before any code, the principles. Each one came from shipping a streaming UI, watching real users (or myself) struggle with it, and figuring out the rule.

**Principle 1: monotonic forward motion.** The text on screen only grows. It never deletes a previously-shown character, never re-renders a previously-shown section. Backtracking is jarring and feels like the system "made a mistake," even when the mistake is just a render-time correction.

**Principle 2: token boundaries are not display boundaries.** Models emit tokens that may end mid-word, mid-Unicode-codepoint, or mid-markdown-syntax. Showing partial tokens directly produces flickering and broken layout. Buffer until you have a renderable unit, then flush.

**Principle 3: structure should resolve correctly.** If the model is producing markdown, the rendered output should look right at every intermediate state, not just at the end. A half-complete fenced code block should not render as broken markdown; it should render as a code block in progress.

**Principle 4: signal absence of motion.** When the model pauses (between turns, during tool execution, during retry), the user needs to see that the system is alive. A spinner or a "thinking..." indicator that appears whenever the token stream stalls keeps trust in the system.

The rest of the post is about implementing these.

## Section 3: the basic terminal streamer

Start with the SSE event loop from Anthropic's API. The client library yields events as they arrive.

```python
from anthropic import Anthropic
import sys

client = Anthropic()

def stream_basic(prompt: str):
    with client.messages.stream(
        model="claude-3-5-sonnet-20241022",
        max_tokens=512,
        messages=[{"role": "user", "content": prompt}],
    ) as stream:
        for text in stream.text_stream:
            sys.stdout.write(text)
            sys.stdout.flush()
    print()

stream_basic("Explain prompt caching in two paragraphs.")
```

This satisfies principle 1 (text only grows) and partially satisfies principle 2 (the SDK already aggregates partial tokens into safe Unicode boundaries). It does not satisfy principles 3 or 4. For a plaintext response in a basic terminal, this is enough. For anything richer, keep reading.

## Section 4: handling token-boundary edge cases

The `text_stream` iterator from the SDK handles UTF-8 codepoint boundaries correctly. If you are working with the lower-level event stream, you have to handle this yourself. The pitfall: a multi-byte codepoint may be split across two streamed chunks. Naively decoding each chunk as UTF-8 raises `UnicodeDecodeError` on the first half.

```python
class StreamBuffer:
    def __init__(self):
        self._buf = bytearray()

    def add(self, chunk: bytes) -> str:
        self._buf.extend(chunk)
        # try to decode as much as possible
        for i in range(len(self._buf), max(-1, len(self._buf) - 4), -1):
            try:
                text = self._buf[:i].decode("utf-8")
                del self._buf[:i]
                return text
            except UnicodeDecodeError:
                continue
        return ""
```

The loop tries decoding the buffer at successively shorter lengths, peeling off up to 3 bytes (the maximum continuation length for UTF-8) until a valid decode succeeds. The unconsumed bytes stay in the buffer for the next chunk.

Higher-level SDKs do this for you; if you are streaming over raw HTTP, you need it.

## Section 5: word-boundary buffering for prettier output

Streaming raw token-by-token can produce mid-word breaks in the rendered output, especially with monospace fonts that re-flow as the line fills. A small word-boundary buffer eliminates this.

```python
import re

class WordBuffer:
    def __init__(self, flush_on=re.compile(r"[\s,.;:!?]")):
        self._buf = ""
        self._pat = flush_on

    def add(self, text: str) -> str:
        self._buf += text
        # find the last whitespace/punctuation
        m = None
        for x in self._pat.finditer(self._buf):
            m = x
        if m is None:
            return ""
        end = m.end()
        out = self._buf[:end]
        self._buf = self._buf[end:]
        return out

    def flush(self) -> str:
        out = self._buf
        self._buf = ""
        return out

# usage
wb = WordBuffer()
for text in stream.text_stream:
    out = wb.add(text)
    if out:
        sys.stdout.write(out)
        sys.stdout.flush()
sys.stdout.write(wb.flush())
```

The buffer holds onto in-progress words until the model finishes them. Latency cost: typically 50 to 200ms (one or two more chunks). Visual cost: zero mid-word breaks. Worth it on a wide screen where mid-word wrapping is ugly.

## Section 6: rendering markdown progressively

Markdown is the hard case. A streaming response that renders correctly at every intermediate state requires the renderer to know the difference between "this is a complete markdown construct" and "this is the start of a construct that has not finished yet."

The pragmatic approach: split the stream into "stable" and "in-progress" sections. The stable section is everything before the last unclosed construct (open code fence, open emphasis, open link). Re-render the stable section once and write it to the screen. The in-progress section is rendered as plaintext and re-rendered every chunk.

```python
def find_stable_split(text: str) -> int:
    """Return index where stable text ends and in-progress begins."""
    # find last unclosed code fence
    fences = [m.start() for m in re.finditer(r"^```", text, re.MULTILINE)]
    if len(fences) % 2 == 1:
        return fences[-1]  # everything before the unclosed fence is stable
    # find last unclosed inline backtick
    last_tick = text.rfind("`")
    if last_tick != -1 and text.count("`", last_tick) % 2 == 1:
        return last_tick
    # default: stable is up to last newline
    last_nl = text.rfind("\n")
    return last_nl + 1 if last_nl != -1 else 0
```

This is a sketch; a production version would handle nested constructs, escaped backticks, and link/image syntax. Libraries like `rich` (Python) and `chalk` (Node) do this for you with `rich.console.Console.print(..., markdown=True)` and friends; lean on them rather than reinventing.

## Section 7: the spinner and the heartbeat

When the stream pauses (model is thinking, tool is running, retry in progress), the user needs to see life. A simple spinner running on a separate thread, gated by an event flag the main loop sets when streaming is active or paused.

```python
import threading, time, itertools, sys

class Spinner:
    def __init__(self, frames=("⠋", "⠙", "⠹", "⠸", "⠼", "⠴", "⠦", "⠧", "⠇", "⠏")):
        self.frames = frames
        self._stop = threading.Event()
        self._active = threading.Event()
        self._thread = None

    def start(self):
        self._stop.clear()
        self._thread = threading.Thread(target=self._run, daemon=True)
        self._thread.start()

    def show(self):
        self._active.set()

    def hide(self):
        self._active.clear()
        sys.stderr.write("\r \r")
        sys.stderr.flush()

    def stop(self):
        self._stop.set()
        if self._thread:
            self._thread.join()

    def _run(self):
        for frame in itertools.cycle(self.frames):
            if self._stop.is_set(): break
            if self._active.is_set():
                sys.stderr.write(f"\r{frame} ")
                sys.stderr.flush()
            time.sleep(0.08)
```

The pattern: hide the spinner when text is flowing, show it when the stream is paused. A 200ms gap in the stream is the trigger; below that, the spinner would flicker. Above 200ms, the user notices the absence of motion and the spinner reassures.

```python
sp = Spinner()
sp.start()
last_text_time = time.time()
sp.show()
for text in stream.text_stream:
    if text:
        if time.time() - last_text_time > 0.2:
            sp.hide()
        sys.stdout.write(text)
        sys.stdout.flush()
        last_text_time = time.time()
sp.stop()
```

The spinner is on stderr so it does not corrupt piped stdout. The carriage return tricks (`\r \r`) erase the spinner cleanly when text resumes.

## Section 8: HTTP/SSE streaming for browser clients

When the client is a browser, the server streams Server-Sent Events. The browser's `EventSource` API consumes them natively.

Server side (FastAPI):

```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
from anthropic import Anthropic
import json

app = FastAPI()
client = Anthropic()

@app.post("/chat")
def chat(req: dict):
    def gen():
        with client.messages.stream(
            model="claude-3-5-sonnet-20241022",
            max_tokens=1024,
            messages=req["messages"],
        ) as stream:
            for text in stream.text_stream:
                yield f"data: {json.dumps({'text': text})}\n\n"
            yield f"data: {json.dumps({'done': True})}\n\n"
    return StreamingResponse(gen(), media_type="text/event-stream")
```

Browser side:

```javascript
const out = document.getElementById('output');
const resp = await fetch('/chat', {
  method: 'POST',
  body: JSON.stringify({messages: [{role: 'user', content: 'Hello'}]}),
  headers: {'Content-Type': 'application/json'},
});
const reader = resp.body.getReader();
const decoder = new TextDecoder();
let buf = '';
while (true) {
  const {done, value} = await reader.read();
  if (done) break;
  buf += decoder.decode(value, {stream: true});
  let idx;
  while ((idx = buf.indexOf('\n\n')) !== -1) {
    const event = buf.slice(0, idx);
    buf = buf.slice(idx + 2);
    if (event.startsWith('data: ')) {
      const data = JSON.parse(event.slice(6));
      if (data.text) out.textContent += data.text;
    }
  }
}
```

Two things to watch: HTTP intermediaries (CDNs, reverse proxies) often buffer responses and break SSE. Set `Cache-Control: no-cache` and `X-Accel-Buffering: no` (nginx) to defeat the buffering. And: use `TextDecoder` with `stream: true` to handle multi-byte UTF-8 across reads.

## Section 9: streaming tool calls

Tool calls are not a stream of text; they are a structured event. The model emits something like `{"type": "tool_use", "name": "search", "input": {"query": "..."}}` and the agent runs the tool. The user-facing experience needs to reflect this.

The pattern I use: when a tool call starts streaming, switch the UI to "tool mode," show the tool name and a progress indicator, hide the text stream until the tool completes. This makes the structure of the agent's work visible: text, then tool, then more text.

```
> What is the weather in Paris?

Looking that up.
[tool: get_weather] in Paris…
     ↳ 18°C, partly cloudy, wind 12 km/h

The current weather in Paris is 18 degrees Celsius...
```

The bracketed `[tool: get_weather]` line is rendered when the tool call starts. The indented result line is rendered when it completes. The next text is the model's continuation. Each section is visually distinct; the user sees the agent's structure.

This is more design than implementation. The trick is having the streaming layer expose enough state (which event type, which tool, completion status) for the renderer to use. Don't pretend everything is text.

## Section 10: what I have measured

Anecdotes do not scale; metrics do. Three numbers I track for streaming UIs:

The first: time to first byte (TTFB), which is how long until the first character appears. Above 1 second, users notice and lose patience. Streaming responses on Anthropic Sonnet typically have TTFB of 400 to 700ms; on Haiku, 200 to 400ms. With prompt caching, Sonnet drops to 200 to 400ms. Each shave of 100ms is perceptible.

The second: stall ratio, the fraction of session time where the stream is paused. Healthy is under 5 percent. Above 15 percent, the user feels the system is laggy even if total time is fine. Most stalls are tool calls; if they dominate, the right fix is faster tools, not better spinners.

The third: bytes-per-second smoothness, measured as the ratio of standard deviation to mean over a one-second sliding window. A smooth stream has a low ratio (under 0.5). A bursty stream has a high ratio. Bursty streams feel jerky even when the average rate is fine. Buffering to smooth out bursts (release tokens at a steady rate up to a small cap) helps.

These three numbers are on the same dashboard as everything else. Streaming is observable like anything else. When it feels worse, the numbers usually show why.

## References

- Nielsen on response times: https://www.nngroup.com/articles/response-times-3-important-limits/
- Server-Sent Events spec: https://html.spec.whatwg.org/multipage/server-sent-events.html
- Anthropic streaming docs: https://docs.anthropic.com/en/api/messages-streaming
- `rich` library docs: https://rich.readthedocs.io/
- HTTP streaming and proxy buffering: https://www.nginx.com/blog/avoiding-top-10-nginx-configuration-mistakes/
