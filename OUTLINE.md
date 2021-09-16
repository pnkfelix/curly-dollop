Note: This has migrated to:

  https://quip-amazon.com/V6zUAhK3N9kt/Outline-for-workshop-Rust-Hurry-Up-and-await-reInvent-2021

# Rust: Hurry Up and `.await` (re:Invent 2021)

Note: Two hour workshop.

Abstract: Async Rust enables building efficient and reliable networking
services. However, its learning curve can be steep, and gotchas can trick even
experienced Rust developers. This workshop, aimed at intermediate and expert
Rust developers (either with or without async experience), will dive into Async
programming in Rust. We will explain the operational model for async, discuss
some common pitfalls that developers encounter, and demonstrate tools for
resolving correctness and performance problems.

## [30 min] Part 1: Opening

### [[5 min]] Cloud Account Setup Steps

(We need to initiate this at outset, *before* presentation/brainstorming chats,
because event-engine folks say may take a while to complete for a large
audience; however, I don't think we can dedicate much time to *support* of
issues that arise.)

### [[10 min]] Why Rust (Brain-storming)

   "Why Rust" review: Workshop partcipants engage, building lists of benefits of
   Rust, and also its associated tradeoffs.

   (This section is largely an engagement exercise, to get the participants
   warmed up and talking to each other and/or to me.)

### [[10 min]] Async Programming (Brain-storming)

 * What is it

 * When/why to use it

 * How to write async program

 * Where Async Rust differs from other async paradigms

### [[5 min]] Our Example Service: Interactive Game Play (Presentation)

We will be making a simple turn-based board game engine, driven by AWS Lambda.

 * Player AI: graph/tree search over future moves
   * Search can be compute-intensive; motivates efficient code.

 * Async communication with external world
   * Store info about past game outcomes persistently
   * Use that stored info to inform future games

 * Want to terminate search when either:
   1. Move timer expires (i.e. move is forced) or
   2. Network communication returns with information provides evidence that
      overrides search (e.g. that move under consideration has 100% loss record,
      or 100% win record).

 * So: Start search and initiate cloud database query, and then if
   historical-move database bears fruit, then use that (and cancel search).

   (Desire to cancel ongoing search should motivate simple demo of `select!`;
   and *maybe* the async model in general?)

## [20 min] Part 2: Basics of Deploying Rust to the Cloud

We'll be using AWS Lambda and the Serverless Application Manager (SAM).

(Not sole option for Rust in Cloud; just way to launch something here and now.)

Exercise 1: Get a TicTacToe game launched on participants' accounts, and
demonstrate it running.

Exercise 2: make simple modification and re-doploy: Go from provided trivial
move selection to *anything* more intelligent.

Exercise 3 (optional): Switch from TicTacToe to Connect Four, using code
provided that replaces game core.

## [20 min] Part 3: Writing simple async/await code

Exercise 4: Implement minimax move selection (for either TicTacToe or Connect
Four, depending on whether you got to Exercise 3 above) as an async fn.


(From here; things get potentially branchy: e.g. we might encourage people to go
to one table or another in order to colocate the people interested in skipping
ahead and spending all remaining time on Part 5.)


## [15 min] Part 4: Investigation

Presentation: How can we debug this stuff?

 * Application-Oriented: Logging (see also `tracing` crate)

 * OS-Oriented: dtrace events; perf

 * Async-Runtime-Oriented: Tokio-Console

Exercise 5: Add tokio-console integration to the service. Use the console to
observe the tasks in your app.


## [25 min] Part 5: Async Communication with Cloud Services

(I am not expecting people to complete this section. I'm expecting to present
the basics, and then turn people loose on trying to get as far as they can.)

Presentation: Basics of IAM and DynamoDB storage.

Exercise 6: Store win/loss records in DynamoDB.

Exercise 7: Update move AI to query DynamoDB in parallel with move search. When
historical records are present and relevant, use them to inform search results.
