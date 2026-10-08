# Solana: the real difference

If you know one smart-contract platform, this chapter is for you. If you
don't, the one-line version: there, the platform's rules are fixed by the
network and your program lives inside them; here, your program's rules are
the only rules, and the network just checks the proof. vprogs
rollups and Solana look strikingly similar on the surface, and the one
place they differ explains everything else about the design.

## The similarities are real

Both systems share the same core shape:

| | Solana | a vprogs rollup |
|---|---|---|
| State shape | accounts, addressed by keys, holding data and lamports (Solana's smallest unit) | resources, addressed by derived ids, holding data and balances |
| User intent | signed transactions naming programs and accounts | signed user actions naming targets in program state |
| Execution | a runtime validates and applies each transaction | a runtime validates and applies each action |
| Concurrency discipline | non-conflicting txs run in parallel across programs; a losing tx fails and is resubmitted | transactions with disjoint resources execute and prove in parallel, across blocks too; only the stitching is sequential, the batch and bundle proofs that chain them by construction ([How it all chains](chaining.md) defines the tiers) |
| Money | native token, rent (a minimum-balance floor) on accounts | native KAS, fees and storage mass (an extra charge for creating small outputs, since every full node stores the unspent set; the [Kaspa appendix](appendix-kaspa-depth.md#storage-mass-paying-for-the-unspent-set) has the formula) |

A Solana developer reading tt's guest code will feel at home: there are
user accounts with balances, locks authorizing spends, a minimum-balance
floor on accounts, an index of "who owns what". The similarity is
deliberate.

## The difference: whose runtime is it?

On Solana, the runtime is **the chain**. Sealevel (Solana's parallel
transaction scheduler), the account model, rent and fee rules, what makes
a transaction well-formed: all of that is protocol, identical for every
program, enforced by every validator, changeable only by a network
upgrade. Programs run inside a runtime they cannot change: they
orchestrate accounts, but the rules are set by the network.

In a vprogs rollup, the runtime is **the program**. tt's guest literally
ships a [`runtime.rs`](https://github.com/biryukovmaxim/vprog-tictactoe/blob/fe6b0e8/guest/src/runtime.rs), and inside the proof it is the *only*
runtime there is. Resource derivation lives in the app too: tt decides
[its resource kinds, its hash domains, its id derivations](https://github.com/biryukovmaxim/vprog-tictactoe/blob/fe6b0e8/guest/src/program/resources.rs#L1-L11) (the framework
hands you the pattern; the app picks the keyspace). Transaction validity,
lock semantics, what a "turn timer" means: all rollup program logic,
executed and proved like any other line of guest code. There is no shared runtime; every program assembles its own from the
framework's parts.

The claim needs a boundary, because it is smaller than it sounds. A
chain's hardest problems are agreement: what order transactions
happened in, where the data lives, what counts as settled, what is
final. The program re-solves none of them: it buys ordering, data
availability, settlement, and finality from Kaspa unchanged (chapters [3](based-rollup.md)
to [5](chaining.md)). What moves into the program is the layer Solana fixes in
protocol, the system program's duties:

- **Account creation and addressing.** On Solana the system program
  creates accounts, and PDAs (program-derived addresses) hand programs
  control of derived accounts. Here derivation is app code: tt gives
  games their own keyspace, derived from the creator's lock and a
  counter, with domain separation keeping keyspaces from colliding.
- **The cost of holding state.** Solana's rent is protocol: holding
  state costs lamports, enforced by every validator. Nothing meters
  rollup state per block; state costs what proving and storage cost the
  operator, so the honest limit is how much you can afford to keep, not
  how much the protocol lets you. A program can still place floors as
  rules: tt's minimum-balance floor and its withdrawal minimum are
  config the app chose, batteries rather than metering.
- **Transaction well-formedness and balance.** On Solana these are
  protocol rules, enforced by the runtime and the system program.
  Here they are program logic, proved with everything else.

This is a direct consequence of the zkVM: since the guest is an
ordinary program, whatever it computes
*about its own rules* is covered by the same proof that covers the rules
themselves. On a chain, the runtime must be fixed because every validator
must agree on it before running your code. In a rollup, agreement
comes from the receipt, so the runtime can be as application-specific as
the application.

## What it buys

- **Resource derivation as a design surface.** The keyspace pattern
  above is plain code: the app picks any derivation it can compute, and
  the proof pins the result.
- **Rules that can be arbitrary.** A turn timer that forfeits a round, a
  pot that splits on a draw: no need to fit these into a shared
  execution model, because there is no shared execution model.
- **New rules, new identity.** "The protocol" is the guest ELF; changing
  the rules is changing the program (covenant ids pin image ids precisely
  so this is explicit: a new rules version is a new identity, not a
  surprise). The money path is explicit: a new image id is a new
  covenant id, so an upgrade means moving to a new instance: users exit
  through the old instance's permission tree and deposit into the new
  one. In-place
  migration does not ship, and draining the old instance still needs
  someone to keep its stack settling; an instance everyone is leaving
  is the least likely to keep one, which is [The machinery](machinery.md)'s liveness trust
  at its sharpest.

## What it costs

- **No free composability.** Solana programs share one state machine, so
  one program can call another atomically. Each vprogs program is its own
  proved system; cross-program calls are future work, sketched but not
  shipped ([Single sovereign apps](single-sovereign-apps.md) lives inside this limitation).
- **The runtime is your responsibility.** Nobody else guarantees your
  rules make sense. The framework's batteries (locks, unlockers, signers)
  are the strong default, and `runtime.rs`'s job is largely
  *assembling* them rather than inventing from zero. But the choice, and
  the consequences, belong to the program.

## The Ethereum reader's checklist

The Solana comparison above covers accounts and runtimes; the reader
from EVM rollups has a different checklist, and the answers are
short:

| The question | Here |
|---|---|
| Validity or fraud proofs? | Validity: every settlement carries a zk proof every Kaspa node checks |
| Forced inclusion? | Data inclusion is permissionless (the lane has no gatekeeper); forced processing is not shipped: nothing forces the tip to advance, it moves when someone settles ([The machinery](machinery.md)) |
| Sequencer failure? | There is no sequencer; whoever settles picks how far the tip moves, and anyone with a valid proof can settle |
| Exit latency? | To entitlement: confirmation window plus proving plus L1 inclusion; to funds: plus the claim transaction's own confirmations ([How it all chains](chaining.md)); no measured numbers published yet |
| Data availability? | The lane on Kaspa itself: public, consensus-ordered, its commitments verifiable from any node's retained headers; entry data older than the pruning window needs an archival node ([the Kaspa appendix](appendix-kaspa-depth.md#reconstructing-the-program-from-l1)) |
| Preconfirmations? | None as a protocol promise; anyone can run a node in execution mode, replay the lane, and read the pending state, anchored to the latest settlement |

In Solana the runtime belongs to the network; in a vprogs rollup it
belongs to the program, assembled from framework parts and covered by
the proof, on top of an L1 that still owns ordering, data availability,
and settlement.
