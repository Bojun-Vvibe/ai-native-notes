# Backpressure Semantics for Streaming Agent Output

There is a class of bug that only shows up after you ship: the agent works fine in your terminal, works fine in the unit tests, works fine when one developer is poking at it, and then falls over the moment a real consumer tries to pipe its output into something slower than a TTY. The symptom is usually one of three flavors. Either the agent's output gets truncated halfway through a tool call argument, or it stalls for ten seconds and then dumps everything in one burst, or — my personal favorite — the process holds three gigabytes of resident memory because it has buffered the entire stream of tokens from a forty-thousand-token response while the downstream writer is doing something dumb like flushing on every newline through an SSH tunnel.

All of these are backpressure bugs. None of them are visible in the happy-path single-developer loop, because the TTY is fast, the operating system is generous with pipe buffers, and you never push hard enough to surface the underlying flow-control gap. But streaming agent output is, fundamentally, a producer-consumer system with three unequal participants — the model, your bridge process, and the eventual sink — and if you don't think about backpressure between every adjacent pair, you will eat one of these failures within a month of going to production.

## What backpressure actually means here

The textbook framing is: backpressure is the signal that flows backwards from a slow consumer to a fast producer, telling it to pause until the consumer catches up. In a synchronous world this happens for free, because `write()` blocks until the kernel has accepted the bytes. In an async world it has to be modeled explicitly, because the natural shape of an async producer is "fire events into a queue and forget about them," which is exactly the wrong shape for backpressure.

Streaming LLM responses are firmly in the async camp. The provider sends you a chunked HTTP response — typically server-sent events with one delta per chunk — and your client library hands you a callback or an async iterator. The model does not slow down based on whether you are processing the chunks. The model has already produced them, they are sitting in the provider's edge cache, and they are arriving at you at whatever rate the network can deliver them. There is no real backpressure on the model itself; the most you can do at that boundary is close the connection, which is the nuclear option.

So the first useful framing: there is no upstream backpressure to the model. You cannot ask it to wait. The only thing you can do is decide what to do with chunks once they arrive at your process. That decision is where most teams get it wrong.

## The three places it breaks

Consider a typical agent bridge: a Python or Node process that holds an HTTP connection open to the model provider, processes streaming deltas, and forwards them to some sink — a websocket connected to a browser, a stdout pipe consumed by another process, a row in a database that's being updated in place, a chat platform's incoming-webhook endpoint that wants one message per N tokens.

Bug shape one: unbounded buffering. The naive implementation reads every chunk from the provider and pushes it into an in-memory list, then forwards the list at some natural boundary like "end of message" or "tool call complete." For short responses this is invisible. For long responses — a model emitting eight thousand tokens of code — you have just buffered eight thousand tokens of UTF-8 in your process while the downstream consumer is doing nothing useful. If you are running ten of these in parallel, you have buffered eighty thousand tokens. If your process is in a small container with a 512 MB limit, you OOM. The fix is conceptually simple — flush incrementally — but most agent libraries do not make this easy because they want to give you a "complete response" object.

Bug shape two: head-of-line blocking on a slow sink. Suppose your downstream is a chat platform that accepts at most one message per second. You decide to forward token deltas as they arrive. Now your stream-reading coroutine is blocked on the chat platform's rate limit, and while it is blocked, the HTTP connection to the model provider is not being read. The kernel TCP buffer fills up. The provider's edge eventually notices and either slows down — which might be fine — or, in practice, times out the connection because their side has been writing for thirty seconds with no acknowledgement and they assume you are dead. You get a connection reset halfway through a forty-thousand-token response, and the agent's tool call never completes because the JSON was cut off mid-argument.

Bug shape three: byte-level versus message-level boundaries. The streaming protocol gives you bytes. Your agent logic wants messages. If you naively forward bytes to the sink in one-byte chunks, you might be fine for a websocket but you will absolutely destroy a logging pipeline that does one syscall per write. Conversely, if you batch up messages until you see a "complete" marker, you have introduced latency that the user will notice as a stutter. The right granularity is almost never "as it arrives" and almost never "when complete" — it is something in between, governed by the cost-structure of the sink, and you have to model that explicitly.

## A working mental model

The model I keep coming back to is: every adjacent pair of producer and consumer in your stream needs an explicit policy answer to two questions. First, what is the maximum amount of unprocessed data this consumer is willing to hold in memory? Second, what does the producer do when the consumer is at capacity? There are only three sensible answers to the second question — block, drop, or fail — and you must pick one per pair, and you must pick them consciously.

Block is the default for synchronous code and the rare default for async code. It is correct when the producer can afford to wait — for instance, when the producer is ultimately the model, and "waiting" means "stop reading the HTTP connection, let the kernel buffer fill, let the provider's edge slow you down." Note that in practice the provider will not actually slow down — they will time you out — so this is rarely the right answer at the model boundary. It is often the right answer between two of your own coroutines.

Drop is correct when the data is purely informational and a stale value is worse than no value. The classic case is a progress indicator: if you are emitting "tokens generated so far" updates to a UI at sixty hertz and the UI can only render at thirty, you should drop the intermediate values, not queue them. Most agent UIs use this policy without realizing it.

Fail is correct when the data must arrive in full or not at all, and you cannot afford to wait. This is the right policy for tool call arguments — you cannot drop bytes from a JSON payload, and you cannot wait forever, so if you cannot keep up you must abort the call and surface the error.

## Implementation: bounded queues, explicit watermarks

The concrete pattern that makes this manageable is: between every async producer-consumer pair, put a bounded queue with explicit high and low watermarks, and write the queue's full-behavior policy at the call site. In Python with `asyncio.Queue(maxsize=N)`, the producer's `await queue.put(item)` will block when the queue is full — which gives you "block" semantics for free. If you want "drop," you wrap the put in a try/except on `QueueFull` after a non-blocking `put_nowait`. If you want "fail," you raise from the same except.

The high-water/low-water distinction matters because oscillating around a single threshold causes thrashing. If your queue holds at most a thousand items, you do not want to start dropping at item one thousand and start accepting again at item nine hundred and ninety-nine — you will spend all your time toggling. The standard pattern is "stop accepting at the high watermark, do not resume until you are below the low watermark, where low is significantly less than high." A common ratio is high equals eighty percent of capacity and low equals thirty percent. This gives the consumer a meaningful chunk of breathing room before the producer turns back on.

For a streaming agent bridge, the concrete topology I recommend is three queues. Queue A sits between the HTTP-reading coroutine and the parser coroutine; it holds raw byte chunks, bounded at maybe sixteen items of up to sixty-four kilobytes each (so about a megabyte of raw bytes max). Queue B sits between the parser and the message-assembly coroutine; it holds parsed delta events, bounded at a few hundred. Queue C sits between message assembly and the sink; its size depends entirely on how slow and bursty the sink is.

The reason for three queues is that the cost-structure changes at each boundary. Reading bytes is cheap, parsing is moderate, sink writes might be very expensive. If you collapse them into a single queue you cannot apply different policies at different stages, and you cannot diagnose where backpressure is actually building when something goes wrong.

## What the metrics look like

If you only instrument one thing, instrument the time-weighted depth of each queue. Not the peak depth — the time-weighted average depth over a one-minute window. Peak depth tells you nothing because it is dominated by single-millisecond bursts. Time-weighted depth tells you whether the queue is genuinely backed up.

The healthy pattern for a queue with capacity N is a time-weighted depth that hovers near zero and occasionally spikes during bursts. The unhealthy pattern is a depth that climbs steadily over the lifetime of a response and does not recover. The pathological pattern is a depth that pegs at the high watermark for sustained periods, because that means you are continuously applying backpressure and the producer is constantly having to wait.

The second-most useful metric is the count of policy invocations per pair: how many times per minute did we drop an item, block on put, or fail. A nonzero count is not necessarily bad — for a "drop" policy on progress updates, a high drop count is fine — but a nonzero count on a "fail" policy is a production incident, and a nonzero count on a "block" policy is at minimum a latency problem worth knowing about.

## The sink-shape question

The hardest part of all this in practice is figuring out the right shape for the sink. There is a temptation to expose the model stream straight through to the end consumer — give the user the same character-by-character experience they get from the provider's own UI — but this is almost never the right answer for a programmatic consumer.

A websocket to a browser can comfortably handle one delta per token, and probably wants exactly that for the typewriter-effect UX. But a webhook to a chat platform absolutely cannot; you need to coalesce into chunks of at least a paragraph and probably more, with a maximum of one message per few seconds. A database update wants the complete response, not a stream of partials, because doing a row update per token is going to obliterate your write throughput.

I have started writing every sink in our agent stack as an explicit "shaper" object that takes the message stream and outputs sink-appropriate batches, with the batching policy declared per sink type. The shapers all implement the same interface — they accept incremental message events and emit batched output events — but their internal policies differ wildly. The browser shaper is roughly a passthrough. The chat platform shaper batches by paragraph or by a five-hundred-millisecond timer, whichever comes first. The database shaper is just a buffer that flushes on stream-complete. Treating these as separate, named objects with explicit policies has eliminated about ninety percent of the production incidents we used to get from "the agent suddenly looks broken in a specific channel."

## What about cancellation

A backpressure conversation that ignores cancellation is incomplete. The two most common reasons your stream stops are that the consumer hung up — user closed the tab, websocket dropped — or the consumer told you it had enough. In both cases, the right behavior is to close the upstream HTTP connection to the model provider promptly, because every additional token you read is a token you are paying for and discarding.

The trap here is that cancellation has to propagate upstream through every queue. If your sink-writing coroutine notices the websocket is dead and just stops reading from queue C, queue C fills up, the message-assembly coroutine blocks on put, queue B fills up, the parser coroutine blocks on put, queue A fills up, and finally — finally — the byte-reading coroutine blocks on put and stops reading the HTTP socket. In the best case this takes a few seconds. In the worst case, your queues are sized large enough that the model finishes generating before the cancellation propagates, and you have paid for the entire response anyway.

The fix is to make cancellation explicit and to propagate it sideways rather than relying on backpressure to do it. A cancellation token, checked at every coroutine boundary, with the byte-reading coroutine specifically checking before every `recv` call. When cancellation fires, every coroutine drops its remaining work and exits, the HTTP connection closes, and you stop paying. This is more code than just letting backpressure do its work, but it is dramatically cheaper at scale.

## Closing thought

Most agent infrastructure is written by people who started from "the model's HTTP stream gives me chunks" and built upward. The shape of the resulting code reflects that — it tends to assume the chunks should be processed and forwarded as they arrive, with the same granularity, in the same order, with no bounded buffering. That works fine until the day it doesn't, and the day it doesn't is usually a day where the only diagnostic you have is "memory went up and then the process died."

If you take exactly one practical thing from this: put a bounded queue between every async pair in your bridge process, give each queue a high and low watermark, write down the full-behavior policy, and instrument time-weighted depth. That single discipline will eliminate a startling fraction of the surprises you would otherwise discover at three in the morning.
