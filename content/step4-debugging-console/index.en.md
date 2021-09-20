---
title : "Debugging via tokio-console"
weight : 50
---
You might be wondering about how we can debug this `async` stuff.

It is a good question, one that the Rust community itself [has been wondering][async-vision]
about.

[async-vision]: https://rust-lang.github.io/wg-async-foundations/vision/roadmap/tooling.html

There are various options available.

You can embed direct observations by instrumenting the application itself. For
this, the [`tracing` crate][tracing] is highly recommended.

[tracing]: https://tokio-rs.github.io/tracing/tracing/

You can also try to use OS-Oriented tools, such as `perf`. This is not so nice,
because those tools do not know about the Async Runtime, which has a complex
structure, and so they will tend to obscure the activities performed by the
Async Runtime rather than illuminate them.

Finally, you can use specialized tools that observe the Async Runtime itself. We
will be exploring that here.

The [tokio-console][] is an experimental tool for investigating the behavior of the Tokio runtime.

[tokio-console]: https://github.com/tokio-rs/console

You can read about how to [get started with it here][getting-started].

[getting-started]: https://hackmd.io/yjtVRDPjQJq05p_8L0-5UA

## Adding the console to `tictactui`

Exercise 6\: `console`

Use the instructions from the [Getting Started guide][getting-started] above to
integrate tokio-console into `tictactui`.

Then run the `console` and the `tictactui`, and see if you observe anything
interesting.
