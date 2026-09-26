# Solana: the real difference

If you know one smart-contract platform, this chapter is for you. If you
don't, the one-line version: there, the platform's rules are fixed by the
network and your program lives inside them; here, your program's rules are
the only rules, and the network just checks the proof. vprogs
rollups and Solana look strikingly similar on the surface, and the one
place they differ explains everything else about the design.

## The similarities are real

Both systems are, at heart, the same picture:

| | Solana | a vprogs rollup |
|---|---|---|
| State shape | accounts, addressed by keys, holding data and lamports | resources, addressed by derived ids, holding data and balances |
| User intent | signed transactions naming programs and accounts | signed user actions naming targets in program state |
| Execution | a runtime validates and applies each transaction | a runtime validates and applies each action |
| Concurrency discipline | non-conflicting txs run in parallel; a losing tx fails and is resubmitted | the same split: work touching the same state never runs in parallel; bundle proving is sequential by construction |
| Money | native token, rent (a minimum-balance floor) on accounts | native KAS, fees and storage mass (Kaspa's extra charge for outputs that park data on nodes) |

A Solana developer reading tt's guest code will feel at home: there are
user accounts with balances, locks authorizing spends, a minimum-balance
floor on accounts, an index of "who owns what". The shape rhymes
deliberately; it's a good shape.

## The difference: whose runtime is it?

On Solana, the runtime is **the chain**. Sealevel's scheduling model, the
account model, rent and fee rules, what makes a transaction well-formed:
all of that is protocol, identical for every program, enforced by every
validator, changeable only by a network upgrade. Programs are guests in a
fixed house: they orchestrate accounts, but the rules of the house are not
theirs to define.

In a vprogs rollup, the runtime is **the program**. tt's guest literally
ships a `runtime.rs`, and inside the proved world it is the *only*
runtime there is. Resource derivation lives in the app too: tt decides
its resource kinds, its hash domains, its id derivations (the framework
hands you the pattern; the app picks the keyspace). Transaction validity,
lock semantics, what a "turn timer" means: all rollup program logic,
executed and proved like any other line of guest code. There is no fixed
house; every program builds, or rather assembles from batteries, its
own.

This is not a small philosophical point; it is the direct consequence of
the zkVM: since the guest is an ordinary program, whatever it computes
*about its own rules* is covered by the same proof that covers the rules
themselves. On a chain, the runtime must be fixed because every validator
must agree on it before running your code. In a proved world, agreement is
bought by the receipt, so the runtime can be as application-specific as
the application.

## What it buys

- **Resource derivation as a design surface.** tt gives games their own
  keyspace derived from the creator's lock and a counter; a Solana
  program would express that as PDA conventions. Here it is just code,
  with domain separation to keep keyspaces from colliding.
- **Rules that can be arbitrary.** A turn timer that forfeits a round, a
  pot that splits on a draw: no need to fit these into a shared
  execution model, because there is no shared execution model.
- **New rules, new identity.** "The protocol" is the guest ELF; changing
  the rules is changing the program (covenants pin image ids precisely so
  this is explicit: a new rules version is a new identity, not a
  surprise, and an upgrade is an emigration). The money moves the honest way: a new image id is a new
  covenant, so an upgrade is an emigration. Users exit through the old
  instance's permission tree and deposit into the new one. In-place
  migration does not ship today. And the window is honest about its own
  limits: draining the old covenant needs its stack to keep settling,
  the same liveness trust as ever, pointed at the instance with the least
  reason to stay alive. Announce, drain, then walk away is operational
  discipline, not a protocol guarantee; a straggler who sleeps through
  the announcement waits with the old covenant, not the new.

## What it costs

- **No free composability.** Solana programs share one state machine, so
  one program can call another atomically. Each vprogs program is its own
  proved world; cross-program calls aren't a feature of the shape
  (chapter 10 lives entirely inside this limitation).
- **The runtime is your responsibility.** Nobody else guarantees your
  rules make sense. The framework's batteries (locks, unlockers, signers)
  are the strong default, and `runtime.rs`'s job is largely
  *assembling* them rather than inventing from zero. But the choice, and
  the consequences, belong to the program.

## The Ethereum reader's checklist

The Solana comparison above rhymes on accounts and runtimes; the reader
from EVM rollups has a different checklist, and the honest answers are
short:

| The question | Here |
|---|---|
| Validity or fraud proofs? | Validity: every settlement carries a zk proof every Kaspa node checks |
| Forced inclusion? | Not shipped: no escape hatch; the tip moves only when someone settles (chapter 6) |
| Sequencer failure? | There is no sequencer; whoever settles picks how far the tip moves, and anyone with a valid proof can settle |
| Exit latency? | Confirmation window plus proving plus L1 inclusion (chapter 5); no measured numbers published yet |
| Data availability? | The lane on Kaspa itself: public, consensus-ordered, provable to any block (chapter 4) |
| Preconfirmations? | None: a state is known when its settlement is buried, not before |

Same shape, different landlord. In a Solana program you rent the runtime;
in a vprogs rollup you own it, assembled from good parts, and proved.
