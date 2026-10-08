# Tic-tac-toe with stakes

Take the simplest game that can carry money, and put money on it:
tic-tac-toe, for stakes.

Two strangers lock funds into a pot on the Kaspa network and play a
multi-round match. When the match ends, the winner takes the pot; a draw
splits it. There is no casino and no escrow agent. The loser cannot
refuse to pay, and the winner cannot claim more than they earned. The
operator cannot rewrite the scoreboard either, because the scoreboard is
a proof, not a statement by anyone.

This is [**vprog-tictactoe**](https://github.com/biryukovmaxim/vprog-tictactoe/tree/fe6b0e8) (we'll call it *tt*), a working example built
on [**vprogs**](https://github.com/kaspanet/vprogs/tree/055ae28a), a framework for *based computation* on Kaspa ([Based rollup on Kaspa](based-rollup.md)
explains the word). The game is chosen for size: tic-tac-toe is small
enough to hold in your head and still exercises every part of the
framework, a proof of concept, not a claim that this game needs a
rollup. In tt, the game's own actions, accounts, and rules are the only
application-specific code; everything else is reused machinery:
executing, proving, settling, exiting. The moves you make in a browser
are signed user actions; the match state lives in a verifiable program;
the payout lands on the real Kaspa chain.

The rest of this short book explains the machine:

- [what a *based rollup* is](based-rollup.md), and what it demonstrates Kaspa can do,
- [the small set of transaction types](transactions.md) the whole machine is built from,
- [how proofs chain user actions, deposits, and settlements across many
  blocks of the Kaspa chain](chaining.md) (its L1),
- [what a zkVM has to do with it](zkvm.md),
- [how this compares to a smart-contract platform like Solana](solana.md),
- and [what you can build on it today](building-an-app.md).

A live demo of tt runs alongside this material.

Two things to state up front. First, everything the machine needs is
live on Kaspa mainnet; the demo itself runs on the public testnet, a
demonstration network with looser security assumptions ([Based rollup on Kaspa](based-rollup.md) gives
the details). Second, safety and liveness are different guarantees. The
chain guarantees that no one can falsify state or steal funds, but
moving money requires the machine to keep running. Who runs it, what can
stall it, and what happens when nobody does are covered in [The machinery](machinery.md)
and [Building an app on it](building-an-app.md).
