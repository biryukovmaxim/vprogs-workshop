# Where things stand

This chapter summarizes what exists, what does not, and what it costs.

## What is real today

- **The framework.** vprogs (the lane/bridge machinery, the three-stage
  proving pipeline, the settlement construction, the permission-tree
  exits, the daemon engine that runs it all) exists as working code,
  exercised by automated end-to-end tests.
- **A complete application.** tt is not a toy snippet: a RISC0 guest with
  its own runtime and rules, a node with DA APIs, a scripted scenario
  driver, a browser wallet signing with the same wire library the zkVM
  verifies. The full loop (deposit, play, settle, exit) runs locally
  against an in-process simnet L1 in minutes.
- **Testnet with real proofs.** The same stack has run end-to-end on
  Kaspa testnet-10: a fresh covenant id, a full match, real GPU-produced
  proofs settling on the public testnet, exits claimed. The
  [KIP-16](https://github.com/kaspanet/kips/blob/master/kip-0016.md),
  [KIP-20](https://github.com/kaspanet/kips/blob/master/kip-0020.md),
  and [KIP-21](https://github.com/kaspanet/kips/blob/master/kip-0021.md)
  extensions it depends on are live on Kaspa mainnet; the deployment sits
  on the public testnet as a demonstration choice (chapter 3 gives the
  details). Dev-mode stub receipts are strictly for the local
  demo; the testnet deployments prove for real.
- **Operational hardening.** The machinery survives restarts, reorgs, and
  pruned nodes (nodes that have discarded old block data); snapshot
  bootstrap, resume, and catch-up modes exist because operations demanded
  them.

## What it is not yet

- **Production.** The project's own status: early development /
  prototype phase; APIs and architecture may change significantly.
- **Multi-program.** As chapter 10 said: single sovereign apps, at the
  current stage. Composability is future work with a yellow paper
  behind it; nothing is implemented yet.
- **Multi-zkVM.** RISC0 today; the backend seam exists (chapter 7), the
  migrations don't yet.

## What it costs, and how long it takes

No published numbers yet, and this book won't invent them. What is
structural: every user action is an ordinary Kaspa transaction
(user-paid, included at L1 speed); settlement latency is the
confirmation window plus proving plus L1 inclusion; exit claims add
their own confirmations. The window is a deployment choice, widened
adaptively when the network looks reorg-prone. Measured end-to-end
figures belong in the runbooks and will be added there when they exist.

## Where to follow and join

The two repositories are the source of truth; code, runbooks, and
issues live there: **vprogs** (the framework) and **vprog-tictactoe**
(the example application and its deployment runbooks, including the
multi-machine testnet walkthrough). There is no token and no foundation;
at this stage the repositories are the project.

## Closing

Return to the opening setup: two strangers, a pot of Kaspa, a game with
no casino and no escrow agent. The machine behind it is now fully
described: signed
actions into a public lane; execution by rules the program defines and
a zkVM proves; state as a digest chain anchored settlement by
settlement into Kaspa; money out through exits that, once committed, no
operator can withhold (until committed, chapter 6's stalls apply:
robbery and delay are different failures, and only the first is
solved). The rollup fixes the shape; the program picks the rules; the
L1 holds the money. If the live demo is running next door, lose a game
knowing why the pot cannot be stolen, and what has to keep
running so you can leave.
