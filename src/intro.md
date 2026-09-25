# The hook: tic-tac-toe with stakes

Take the simplest game that can carry money, and put money on it: tic-tac-toe,
for stakes.

Two strangers each lock funds into a pot on the Kaspa network. They play a
multi-round match. When the match ends, the pot is settled to the winner — and
a draw splits it back. No casino, no escrow agent, no mutual friend holding the
cash. The loser cannot refuse to pay. The winner cannot claim a cent more than
they earned. Even the operator running the game cannot quietly rewrite the
scoreboard, because the scoreboard is not a promise — it is a proof.

Now the strange part: **Kaspa has no smart-contract language.** There is no
place on the Kaspa chain to deploy a tic-tac-toe contract, a pot escrow, or any
program at all. The L1 deliberately does one thing — transfer Kaspa under a
handful of standard lock types — and does it fast. And yet the pot above is
enforced, on that chain, without a trusted referee.

This is **vprog-tictactoe** (we'll call it *tt*), a working example built on
**vprogs** — a framework for *based computation* on Kaspa. In tt, the game's
own actions, accounts, and rules are the only application-specific code;
everything else is reused machinery: sequencing, proving, settling, exiting.
The moves you make in a browser are signed user actions; the match state lives
in a verifiable program; the payout lands on the real Kaspa chain.

The rest of this short book explains the machine that makes this possible:

- what a *based rollup* is and why Kaspa is an interesting home for one,
- the small set of transaction types the whole machine is built from,
- how proofs chain user actions, deposits, and settlements across multiple
  L1 blocks,
- what a zkVM has to do with it,
- how this compares to a smart-contract platform like Solana,
- and what you can build on it today.

A live demo of tt runs alongside this material — this book is the story, the
demo is the proof of life.
