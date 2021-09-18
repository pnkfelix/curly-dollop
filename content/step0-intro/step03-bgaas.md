---
title : "Running example: Board Gaming as a Service"
weight : 13
---

### Our Example Service: Interactive Game Play

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

