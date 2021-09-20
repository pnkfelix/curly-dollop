---
title : "Writing ``trivial'' async code"
weight : 30
---

Exercise 4\: Minimax

Implement minimax move selection (for either TicTacToe or Connect Four,
depending on whether you got to Exercise 3 above) as an `async fn`.

For this initial implementation, you can feel free to hard-code the search depth
as a series of nested `for` loops, and make assumptions about who the "next
player" is after each move.

For example, this here is some code that might have the right flavor of what we
could use\:

```rust
moves
    .iter()
    .enumerate()
    .max_by_key(|(j, m0)| {
        let m0_value = m0.next_state.value_for(p);
        m0.next_state
            .moves()
            .iter()
            .map(|m1| m1.next_state.value_for(p))
            .min()
            .unwrap_or(m0_value)
    })
    .map(|(j, _)| j)
```
