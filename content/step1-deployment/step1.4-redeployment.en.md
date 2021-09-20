---
title : "Modifying and re-deploying a Lambda"
weight : 24
---

For our next exercises, we will modify the provided service code.

Namely, if you experimented with the provided computer player, you may have
realized that its AI is heavy on the "artificial" and light on the
"intelligence."

## Exercise 2\: Make simple modification and re-deploy

Go from provided trivial move selection to *anything* more intelligent.

In particular: figure out how to inspect the moves to see which one yields a
winning move for the current player, and then favor that over the current
"pseudo-random" option.

## Exercise 3\: Switch from TicTacToe to ConnectFour

Figure out how to replace the choice of game that is built into the lambda
service.

Once you have done so, you may want re-do the `sam deploy --guided` so that you
can select a different stack name (one that has "connectfour" in the name
instead of "tictactoe"). You might also accomplish such a rename by directly
editing the `samconfig.toml` file.

Once you have replaced the game, redeploy and test out the result.

::alert[In the actual re\:Invent workshop, the rules of ConnectFour will be provided in the same manner that TicTacToe is. Today, however, implementing ConnectFour is left as a more elaborate exercise for the reader.]{header="Note"}
