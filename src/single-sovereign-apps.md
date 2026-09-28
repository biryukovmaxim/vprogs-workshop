# Single sovereign apps

Chapter 8 named the cost of the design; this chapter sizes it. What
*is* a vprogs program today, and what isn't it yet?

## One covenant id, one world

Each program instance is one covenant id: one lane, one state tree, one
settlement chain, one set of pinned guest images, one operator stack.
Every proof in the system names its covenant id and settles into it; the
identity is welded into the batch journal itself. The consequence:

**A vprogs program is a sovereign app.** It rules its own world
completely (its rules, its state, its exits) and shares nothing with
any other program. There is, at the current stage, no mechanism
by which program A reads or calls program B's state. Composability, the
thing that makes a smart-contract chain feel like one financial system,
does not exist here yet. Each app is an island: a well-built one, with its
own money in and money out, but an island.

## What that means in practice

- **You can build tt.** A staked game with accounts, transfers,
  withdrawals, exits, a web frontend, live on a testnet with real proofs.
  Any application whose state fits inside one program (games, escrows,
  marketplaces where the market *is* the program) fits the shape.
- **You cannot build "the DeFi stack" as one weave.** Lending here,
  DEX there, sharing balances atomically: that requires either one
  monolithic program (which works, but then composability is just...
  internal function calls) or cross-program mechanisms the shape doesn't
  ship yet.
- **Trust is per-app.** Each program has its own operator liveness and
  its own rules; using three programs means three of each. There is no
  global validator set to lean on, by design, since the L1 never runs
  the programs.

## Why start here?

Because sovereignty composes badly but verifies beautifully. A program
that owns everything it touches can be proved end-to-end with one proof
chain, exited through one permission tree, audited as one artifact. The
hard problems vprogs solves first (proof chaining across blocks, reorg
survival, trustless exits) are exactly the problems an isolated app
needs solved. Shared-world mechanisms built on top of that foundation
have something solid to stand on; built before it, they'd be bridges on
fog.

Where it goes from here, multi-program worlds and cross-covenant
calls, is future work with design behind it: a yellow paper sketches
composability, cross-program invocation between instances, and none of
it is implemented yet. The current shape is a deliberate first step,
not the destination.

For now, the mental model to take away: **vprogs today lets you stand up a
small, self-contained chain for a single application, based on Kaspa.**
One app, one covenant id, one proof chain, no landlord, and no new coin: the
money is Kaspa's KAS end to end, only the rules are the app's.
