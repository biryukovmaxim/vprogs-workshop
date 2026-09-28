# How it all chains

This chapter puts the pieces together: every piece from the last
chapter (the lane, deposits, exits, user actions, settlements) moves
at once, across multiple L1 blocks, tied together by
proofs.

Vocabulary check before it piles up: a *witness* is confirmed L1 data fed
to execution; a *batch* is the work over one block; its proof is a *batch
proof*; runs of batches compound into *aggregate* proofs; and the one a
settlement carries, the last aggregate, is the *bundle proof*. The guest
program that does the compounding is the *aggregator* (chapter 7). Five
words, one pipeline.

The word *block* needs pinning too. Execution does not
walk the blockdag's web; it walks the single line consensus has already
ordered: Kaspa's selected chain, also called the virtual chain or the
mega chain. When a block on that line absorbs other blocks (its
*mergeset*), the transactions those blocks carry are counted as part of
that step. One chain block plus its mergeset's uncounted transactions is
one block to the machine: one witness set, one batch, one step of
execution, and the dag's parallelism is flattened before execution ever
sees it.

```mermaid
sequenceDiagram
    participant U as User
    participant OP as Operator
    participant P as Provers
    participant L1 as Kaspa L1

    U->>L1: submit signed action to the lane (subnetwork)
    U->>L1: deposit (output paying the covenant's deposit address)
    Note over L1: blocks pass, actions, deposits and settlements interleave
    L1-->>OP: witnesses: confirmed lane data, deposits, chain context
    OP->>OP: execute actions + credit deposits, block by block
    OP->>P: batch transitions (state root X -> Y, lane tip, block context)
    P-->>OP: batch proofs
    OP->>P: compound into one proof up to block N
    P-->>OP: bundle proof
    OP->>L1: settlement: (new_state, new_lane_tip, prove-to block N) + proof
    Note over L1: confirmation window passes
    L1-->>U: exit entitlements live, user claims via permission spend
```

## State digests succeed each other

The program's history is a sequence of state transitions. Every executed
block produces a journal entry of this form: *previous state root, new
state root, previous lane tip, new lane tip, the L1 context it saw, and
the window's deposit and exit commitments when it carried any*. A batch
proof attests one such transition; an aggregate proof compounds a run of
them; a settlement commits the final root on L1. From the L1's point of
view, then, the program's history is a chain of 32-byte state roots with
the context of each step alongside them (lane tip, block proof point,
exit commitments; chapter 4 lists what each settlement carries), and
every step is provably reachable from the one before.

## Why multiple L1 blocks?

A match takes minutes: moves arrive, deposits confirm, other users act,
and the L1 keeps producing blocks the whole time. A proof that froze the
world at one block would be stale before it landed. Instead, each proof
covers a *window* of
L1 history (all lane entries and deposits up to a named block), and the
settlement says so explicitly: this state is proven against L1 block N,
this lane tip is included, and if you disagree, verify the proof. User
actions submitted in block 3 and block 40 end up proven together into one
settled state, with nothing in between silently dropped: the node's
lane commitments ([KIP-21](https://github.com/kaspanet/kips/blob/master/kip-0021.md)) give the lane one canonical history, and
the proof binds its tip. Every block header carries a commitment to
every active lane. When a node validates a settlement, the settlement
script requires the proof's journal to commit the same value the header
carries for the block the settlement names, so a proof about a fictional
or stale L1 world cannot satisfy a node that follows the real chain (the
appendix has the mechanics, including why no range can be skipped).

The machine only cites blocks already behind the confirmation window,
so a cited block always sits deeper than the settlement resting on it.
A reorg deep enough to remove the cited block would also remove the
settlement, and past that depth both are as permanent as any Kaspa
payment. The opcode does enforce one bound of its own: it reads
commitments only from roughly the last twelve hours of blocks. A machine
stalled longer than that must first prove a window that reaches a recent
block, then settle; the chaining shape makes that possible (the appendix
covers the recovery).

## Deposits and exits use the same proof path

Deposits are L1 outputs, so they're witnessed the same way lane actions
are, and bound twice: the proof's journal commits the deposit address it
credited, and the settlement script re-derives that address from the
covenant id and requires the proof to name exactly it. Exits flow the
other way on the same rails: when execution debits a user and emits an
exit, the entitlement lands in the permission tree, and the settlement
carries the tree's commitment. One proof cycle carries the whole ledger
of who-entered and who-may-leave.

## Waiting for finality, and surviving reorgs

Kaspa orders blocks fast (roughly one per second), but "a block" is not yet "a fact". The machine
follows the chain behind a confirmation window, a configurable number of
confirmations, widened adaptively when the network looks reorg-prone, and
treats a block as solid only inside it. If the chain reorganizes anyway,
blocks the machine was watching simply vanish from its view as rollbacks:
bundles that stood on the reorganized side are dropped and rebuilt
against the surviving chain, and a settlement removed by a reorg before
it is confirmed is simply resubmitted (the appendix has the recovery
detail, including which proving work gets reused). The settlement chain, immutable
once Kaspa has it, is never reinterpreted. The short version: the
machine never trusts a dead block, and never needs to.

Money that already moved follows Kaspa's own rule. An exit claim is an
ordinary Kaspa transaction; once it is buried as deep as the network's own
habits require, undoing it means rewriting Kaspa, not rewriting the rollup.
The machine protects *state*; Kaspa's proof-of-work depth protects
*payments*, payouts included.

## What one settlement buys you

Look at a single settlement tx on an explorer. It names
its covenant id. It commits the new state digest, the lane tip, and the block
it proves to. Its first output can only be spent by the next settlement of
the same covenant id, so the history cannot fork without splitting real
money on L1. Where the bundle emitted exits, a permission output commits
the tree: live, claimable entitlements (a bundle with no exits settles
without one). And everything inside it, every move of every game,
every deposit, every balance, is a 32-byte root away, verified by a proof
anyone can check. What L1 does *not* enforce is freshness: nothing on-chain
forces a settlement to advance the tip to today's lane head; the tip moves
when the operator settles, and an operator can stall (chapter 6 lists
the stall cases). The rest of this book covers who runs the machine, what
proves it, and what building on it is like.
