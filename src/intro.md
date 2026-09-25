# The hook: tic-tac-toe with stakes

Take the simplest game that can carry money, and put money on it: tic-tac-toe,
for stakes.

Two strangers each lock funds into a pot on the Kaspa network. They play a
multi-round match. When the match ends, the pot is settled to the winner, and
a draw splits it back. No casino, no escrow agent, no mutual friend holding the
cash. The loser cannot refuse to pay. The winner cannot claim a cent more than
they earned. Even the operator running the game cannot quietly rewrite the
scoreboard, because the scoreboard is not a promise. It is a proof.

Now the strange part: **a staked match is an awkward fit for Kaspa's
chain, but not because Kaspa "has no smart contracts".** It has a script
engine, richer than Bitcoin's, and layers like SilverScript are being
built on it; a determined builder could even push a pot-and-board game
through scripting alone, threading the state from output to output. But
that is building against the grain. Kaspa runs on UTXOs: every coin is an
output that exactly one transaction may consume, and no other transaction
can reference it afterward, so shared state has no native home and must
be hand-carried forward transaction by transaction; the script language
stays deliberately short of Turing-complete; and every rule, timer, and
branch costs on-chain execution. And yet the pot above is enforced, on
that chain, without a trusted referee, and without the fight.

This is **vprog-tictactoe** (we'll call it *tt*), a working example built on
**vprogs**, a framework for *based computation* on Kaspa. The game is
chosen deliberately: tic-tac-toe is small enough to hold in your head and
real enough to exercise every pattern the framework ships, a proof of
concept, not a claim that this game needs a rollup. In tt, the game's
own actions, accounts, and rules are the only application-specific code;
everything else is reused machinery: executing, proving, settling, exiting.
The moves you make in a browser are signed user actions; the match state lives
in a verifiable program; the payout lands on the real Kaspa chain.

The rest of this short book explains the machine that makes this possible:

- what a *based rollup* is, and what it demonstrates Kaspa can do,
- the small set of transaction types the whole machine is built from,
- how proofs chain user actions, deposits, and settlements across multiple
  L1 blocks,
- what a zkVM has to do with it,
- how this compares to a smart-contract platform like Solana,
- and what you can build on it today.

A live demo of tt runs alongside this material; this book is the story, the
demo is the proof of life.
