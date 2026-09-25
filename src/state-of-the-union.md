# State of the union

Where does all this stand, honestly?

## What's real today

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
  Kaspa testnet-10: a fresh covenant, a full match, real GPU-produced
  proofs settling on the public testnet, exits claimed. That testnet is
  where the KIP-16/20/21 script and lane extensions are live; mainnet
  activation is pending (chapter 2 says plainly what that means). Dev-mode
  stub receipts are strictly for the local demo; the testnet deployments
  prove for real.
- **Operational hardening.** The machinery survives restarts, reorgs, and
  pruned nodes; snapshot bootstrap, resume, and catch-up modes exist
  because they've had to.

## What it isn't yet

- **Production.** The project states it plainly: early development /
  prototype phase; APIs and architecture may change significantly. Treat
  everything accordingly.
- **Multi-program.** As chapter 9 said: single sovereign apps. The
  composability question is open future work.
- **Multi-zkVM.** RISC0 today; the backend seam exists (chapter 6), the
  migrations don't yet.

## What it costs, and how long it takes

No published numbers yet, and this book won't invent them. What is
structural: every user action is an ordinary Kaspa transaction (user-paid,
included at L1 speed); settlement latency is the confirmation window plus
proving plus L1 inclusion; exit claims add their own confirmations. The
window is a deployment choice, widened adaptively when the network looks
reorg-prone. Measured end-to-end figures belong in the runbooks, and will
be added there when they exist.

## Where to follow and join

The two repositories are the source of truth; code, runbooks, and
issues live there: **vprogs** (the framework) and **vprog-tictactoe**
(the example application and its deployment runbooks, including the
multi-machine testnet walkthrough). No token, no sale, no foundation to
join: at this stage the repositories are the project.

## The closing loop

Return, one last time, to the opening scene: two strangers, a pot of
Kaspa, a game with no referee. You now know the entire machine that
makes it boring, and boring is the compliment: signed actions into a
public lane; execution by rules the program itself defines and a zkVM
proves; state as a digest chain anchored settlement by settlement into
Kaspa itself; money out through exits that, once committed, no operator
can withhold. The rollup fixed the shape; the program picked the rules;
the L1 holds the money. If the live demo is running next door, go lose a
game of tic-tac-toe knowing exactly why you can't be robbed on the way
out.
