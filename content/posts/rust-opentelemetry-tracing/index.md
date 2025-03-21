---
title: "I Just Wanted Pretty Spans: A Rust OpenTelemetry Story"
date: 2025-03-21T14:22:05-05:00
draft: false
---

Seriously give me the spans.

<!--more-->

## A waht?

I've been working on a rust project to mess with LLMs. Both using LLM APIs and exploring using various AI coders (Cursor, Trae, Gemini). Since the AI writes a lot of the code, I needed an easy way to debug it. I also want to get structured input I can give back to the AI and berate it for mistakes.

Tracing seems like a nice way to get exactly what's happening while filtering out some noise.

## Tracing

I was already using the `tracing` crate for rust logging. It makes nice structured logs that can capture the call stack and context. These were going to stdout and already include logs for other crates.

Integrating the tracing crate is rather straightforward. It mostly has the same interface as the log crate so you can largely just change the import.

It's usually something like `debug!("my cool message")` or `info!("just for you")`. You can also add fields like `debug!("my cool message", my_field = my_value)`. That's cool and all, but it can still be hard to _trace_ what's going on.

## OpenTelemetry

OpenTelemetry is a set of common specs for (now) tracing, metrics, and logging. It supporting more than just tracing still seems WIP, but maybe some day it will just work™.

I've used opentelemetry in go before and it works pretty well. Golang standardizes pretty well on `Context` for keeping info around for a request so the span info can be shoved in their fairly transparently. I found a lot of the ecosystem was also using the telemetry so a lot of things show up nicely.

In rust, it seems like most things use the `tracing` crate, but rely mostly on the logging aspect. So I could magically see all the logs, but more contextual span breakdowns are missing.

This is all just the available information. You still have to figure out configuring the exporter and the backend.

## Backends
I originally tried to use `hyperdx` since it looked nice and supposedly supported metrics, logs, and traces. I got it working eventually, but was really missing my beautiful span view (the whole point of this exercise).

I ended up just using Jaeger as I have in the past. I still had trouble getting my spans to actually be sent to Jaeger. This part was annoying in go from what I remember as well. I had a hard time finding an example of how to set it up fully. Most things I found were either just for `tracing`, `opentelemetry` or were outdated.

I ended up finding this example in the [tracing-opentelemetry](https://github.com/tokio-rs/tracing-opentelemetry/blob/v0.29.0/examples/opentelemetry-otlp.rs) repo. In hindsight, this is the obvious place to look for it, but here we are. One of the things, I had been getting wrong was not keeping a reference to the provider objects. They were probably being dropped before my spans could be sent. The example creates an `OtelGuard` struct to hold the reference and implement the `Drop` trait. Then when it's dropped (at the end of my main) it calls the `shutdown` method on the providers and all is right with the world.

Then setting it all up and keeping it around is just `let _guard = crate::telemetry::init_tracing_subscriber()` in `main`.

You can see my example in the [repo](https://github.com/kasuboski/hal/blob/af9d988e0c2ada7aced082e75e17ce65940081d6/src/telemetry.rs) for my LLM playground. You'll notice the log provider is commented out. It seems like Jaeger doesn't support this part. The logs still show up in the span events though.

## Moving Forward

I finally got it to a place I like. It's really nice to debug LLM calls with the full span that can include the prompt if needed. This was especially helpful when I was creating an index for a RAG system. I was using an LLM to create context and then calling an embedding model to get the vectors. Tracking down which part of that chain would fail was much nicer with the spans.

It's still annoying having the tracing logs attached to the spans. It'd be nicer if there were separate so I wasn't worried about giant requests muddying the interface.

It also seems like I need to add `#[instrument]` to literally everything. This is maybe just a consequence of being used to go, but it is pretty annoying to have it everywhere. Especially when you're missing a span you were expecting and then realize you forgot to instrument.

Oh well.
