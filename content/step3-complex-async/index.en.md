---
title : "Writing more complex async code"
weight : 40
---

If you have played with the `tictactui` program, you may have noticed something
a little odd about it.

## Move Preview Oddity

Namely, when you select a move as a human player, it shows a preview of the move
below the prompt, like so\:

```text
      |     |
 -----|-----|-----
      |     |
 -----|-----|-----
      |     |
 X moves: 1 2 3 4 5 6 7 8 9
 ? 2
      |  X  |
 -----|-----|-----
      |     |
 -----|-----|-----
      |     |
```

Hitting enter at this point commits to the move choice and hitting backspace deletes the preview.

What happens if you hit more numbers\:

```text
      |     |
 -----|-----|-----
      |     |
 -----|-----|-----
      |     |
 X moves: 1 2 3 4 5 6 7 8 9
 ? 21





```

If you're lucky, you get the above (namely, a deleted preview, since there is no "move 21"
for this tictactoe board.

But if you type it "too fast", you'll get this\:

```text
      |     |
 -----|-----|-----
      |     |
 -----|-----|-----
      |     |
 X moves: 1 2 3 4 5 6 7 8 9
 ? 2
      |  X  |
 -----|-----|-----
      |     |
 -----|-----|-----
      |     |
 1
```

Whoa!

## What is happening here?

What we are observing is an *interference* between the distinct actions
occurring under the hood of `tictactui`.

Namely, when you type digits that match a current move, `tictactui` initiates an
asynchronous request for the rendered board resulting from that move.

But concurrently with that, `tictactui` is still accepting user input. And if
the new digit ("1" in this case) arrives before the network communication has
completed, it will print out the digit at the bottom, and then print out the
result of the preview of "move 2" once it completes.

Note that the input buffer is actually holding "21" at this point; if you type
another digit, like "3", you will directly observe that fact\:

```text
      |     |
 -----|-----|-----
      |     |
 -----|-----|-----
      |     |
 X moves: 1 2 3 4 5 6 7 8 9
 ? 213






```

## What can we do about this?

One option would be to not accept user input until after the preview step has
completed.

However, that would yield a subpar user experience; people trying to type "10"
will get frustrated if the system seemingly locks up after just typing the digit
"1".

This example provides a motivation for using a `select!` expression\: we have
two operations that are running concurrently (waiting for more digits, and
downloading a preview of the board). If we get more digits, then we don't want
the preview anymore, and can cancel that operation entirely.

## The task

Exercise 5\: `select!`

You can read more about `select!` here:

https://tokio.rs/tokio/tutorial/select

Revise the code for `tictactui` so that the preview download and input read are
both in a `select!` expression.

You can start by commenting out the existing call to `preview_board`, and instead
setting things up so that it is called at the same place where events are read
(the line that include `reader.next().await`).

If the input read wins the race, then you can just proceed with that input
character and leave the preview cancelled. (Explanation\: if the user is typing
"10", then they probably do not care about seeing the preview for "move 1".)

If the preview download wins the race, then you *still* want to get user input.
So, within that `select!` arm, update any global state as necessary and then
re-await the user input.
