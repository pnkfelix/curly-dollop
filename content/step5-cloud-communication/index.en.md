---
title : "Async communication with cloud services"
weight : 60
---

If you made it this far, congratulations!

::alert[This section has not been completely developed yet. Namely, Felix has not
yet prototyped the code outlined below.]{header="Note"}

Our next step is to work on making our game service able to get smarter over
time, by having it store information about past games, and use that information
as part of its board evaluation function in the minimax search.

For this, we will use the DynamoDB service.

Exercise 6\: Store win/loss records in DynamoDB. For today, you can choose
whether you want to add this functionality to the `tictactui` game, or to the
Lambda service.

Exercise 7\: Update the move AI to query DynamoDB in parallel with move search. When
historical records are present and relevant, use them to inform search results.
