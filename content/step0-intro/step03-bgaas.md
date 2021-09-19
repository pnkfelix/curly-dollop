---
title : "Running example: Board Gaming as a Service"
weight : 13
---

Today we will be deploying a simple turn-based board game engine, driven by AWS
Lambda.

## Simple Games

Since we only have a limited time in this workshop, we will only use relatively
simple games for our examples today: Tic-Tac-Toe, Connect Four, and Checkers.

We will use a prebaked software library that lets us represent simple games
easily.

(The provided library could handle many turn-based games. I have not attempted
to test it with games that require random chance, such as shuffling cards or
rolling dice, but I see no reason the service could not include sources of
randomness. The library's main weakness is lack of full support for hidden
information, because the game state is completely encoded in the URI used to
interact with the service. At best, it can only *obscure* game state, not hide
it completely from the player.)

## Driving the service

Time limitations also lead us to a simple text-interface to our service.

(Or you can encode JSON-based invocations of the service, if you prefer.)

However, I hope you can see how the architecture could be deployed with a
web-based front-end, with nicer graphics for each game.

## Player AI: graph/tree search over future moves

Depending on the game, such search can be computationally-intensive.

(This motivates efficient runtime code!)

## Async communication with external world

The system will be able to store info about past game outcomes persistently,
in the cloud.

The player AI might then use that stored info to inform future games. (The
system might get smarter over time!)

## Concrete issues to consider

For a decent user experience, the player AI cannot search forever.

We want to terminate search when either:
 1. Move timer expires (i.e. move is forced),
 2. The player AI identifies an unbeatable move,
 3. The player AI determines current game is unwinnable, or
 4. Network communication returns with information provides evidence that
    overrides speculation (e.g. that move under consideration has 100% loss
    record, and other moves have no record at all).

So: Start search and initiate cloud database query, and then if
historical-move database bears fruit, then use that (and cancel search).

   (Desire to cancel ongoing search should motivate simple demo of `select!`;
   and *maybe* the async model in general?)

