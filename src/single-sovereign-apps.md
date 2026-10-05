# Single sovereign apps

[Chapter 8](solana.md) named the costs; this chapter measures them. What *is* a
vprogs program today, and what isn't it yet?

## One covenant id, one instance

Each program instance is one covenant id: one lane, one state tree, one
settlement chain, one set of pinned guest images, one operator stack.
Every proof in the system names its covenant id and settles into it;
the identity is part of the batch journal itself. The consequence:

**A vprogs program is a sovereign app.** It sets its own rules,
state, and exits, and shares nothing with
any other program. At the current stage there is no mechanism by which
program A reads or calls program B's state. Composability, the property
that makes a smart-contract chain feel like one financial system, does
not exist here yet. Each app is isolated, with its own deposits and
exits.

## What that means in practice

- **You can build tt.** A staked game with accounts, transfers,
  withdrawals, exits, a web frontend, live on a testnet with real proofs.
  Any application whose state fits inside one program (games, escrows,
  marketplaces where the market *is* the program) fits the shape.
- **You cannot build "the DeFi stack" as one system.** Lending here,
  DEX there, sharing balances atomically: that requires either one
  monolithic program (which works, but then composability is just
  internal function calls) or cross-program mechanisms the shape doesn't
  ship yet.
- **Trust is per-app.** Each program has its own operator liveness and
  its own rules; using three programs means three of each. There is no
  global validator set behind them, by design, since the L1 never runs
  the programs.

## Why start here?

Because a self-contained program is cheap to prove and audit. A program
that owns everything it touches can be proved end-to-end with one proof
chain, exited through one permission tree, audited as one artifact. The
hard problems vprogs solves first (proof chaining across blocks, reorg
survival, trustless exits) are exactly the problems an isolated app
needs solved. Shared-state mechanisms can build on that foundation;
built before it, they have nothing to build on.

Where it goes from here, multi-program systems and cross-covenant
calls, is future work with design behind it: a yellow paper sketches
composability, cross-program invocation between instances, and none of
it is implemented yet. The current shape is a deliberate first step.

For now, the mental model to take away: **vprogs today lets you stand
up a small, self-contained chain for a single application, based on
Kaspa.** One app, one covenant id, one proof chain, no shared runtime,
and no new coin: the money is Kaspa's KAS end to end, only the rules
are the app's.
