# Based rollup on Kaspa

## The one-sentence version

A based rollup moves a program *off* the L1 (its state, its rules, its
execution) but keeps its *enforcement* on the L1: every state change worth
trusting is proved and settled back to Kaspa. That off-chain half is the
**L2**: the program's own world, leaning on Kaspa for what it must not
provide itself. Ordering and data
availability come from the program's lane on Kaspa; enforcement and
payouts land on Kaspa; no external committee, DA service, or bridge token
sits in between. A DA level still exists, and Kaspa is it.

A note for readers from the Ethereum world (others can skip this
paragraph): in Ethereum discourse "based" means L1 proposers do the
sequencing. Here nobody sequences at all; ordering is not a service. Users
publish actions straight to the L1 lane, and the L1's own order *is* the
order. The word is used in its root sense: the app is based directly on
its chain, with no committee, no separate data layer, and no bridge token
between them.

The shape of the whole system fits in one diagram:

```mermaid
flowchart LR
    U["Users (signed actions)"] -- "actions → lane (subnetwork)" --> L1["Kaspa L1"]
    U -. "state reads (index)" .-> OP["Program instance (operator)"]
    OP --> PR["Provers"]
    PR -- "proofs" --> OP
    OP -- "settlements + proofs" --> L1
    L1 -- "witnesses (lane data, deposits, blocks)" --> OP
    L1 -- "payouts (exits)" --> U
```

Users sign actions and submit them to the program's lane on Kaspa
*themselves*: the lane is an L1 subnetwork that serves as the program's
public inbox and its data availability. The operator never has to be asked:
it reads the lane and the chain directly and executes actions
off-chain. Provers produce cryptographic proofs that the execution followed
the program's rules. Settlements, carrying those proofs and commitments to
the new state, land on Kaspa. Money flows back out to users through exits,
enforced by exactly those settled proofs.

One arrow in the diagram deserves a note: apps read current state from the
operator's index (the dashed line), not from Kaspa directly. The index is
a convenience; the authority behind what it serves is the settlement chain
on Kaspa, and chapter 8 covers what a skeptic can do without the index.

## The walls, and the hinges

Kaspa's script engine is real, and richer than Bitcoin's; abstraction
layers are being built on it (SilverScript, Argent among them). The
caricature of "no smart contracts because no scripting" is wrong. The
walls are structural, and there are two. First,
Kaspa runs on UTXOs: every coin is an output consumed by exactly one
transaction, and no other transaction can reference it afterward; shared
state has no native home, so a program must carry its pot, board, and
balances forward output by output (abstractions like SilverScript make the
threading tractable; it remains a fight with the grain). Second, the
script language is deliberately not Turing-complete: flexible enough to
lock and check, not to *be* an application.

Those walls are not accidents to be fixed; they are design choices: the
price of a chain that stays fast and simple to reason about. But the
walls came with hinges, added through Kaspa's own proposal process and
activated on testnet-10 by the Toccata hard fork. KIP-16 gave the script
engine a zk-verify precompile: a script opcode that checks a zk receipt
inside script execution, so a script can make the chain trust a
computation it never ran. KIP-20 added covenant ids, so scripts can bind
outputs to one program instance's identity. KIP-21 added lane
commitments: the node's consensus anchors, references, and proves the
subset of transactions belonging to one lane. Proof verification is
therefore a consensus rule, not a service: every Kaspa node that executes
a settlement runs the check, and a bad-proof settlement is invalid,
rejected like a bad signature. Which proof system and which program
version to trust are not choices made at spend time; they are baked into
the covenant's script hash itself, which chapter 3 opens up.

Where this lives today deserves its own sentences. Those extensions are
**implemented and activated on Kaspa's public testnet-10** (the network
the live demo settles on, mined by its public miners) and **not yet on
Kaspa mainnet**. Until mainnet activates them, every vprogs settlement is
a testnet fact, and the mainnet path is the standard
proposal-and-activation process, not a promise. "Kaspa holds the money"
is a statement about the design and about the testnet of today; mainnet is
still the road ahead.

A based rollup is what the three hinges make, and it dissolves both walls
at once: the program's state lives in the proved world, shared and owned
outright in any shape the program likes, and the rules can be arbitrary
code, because the chain never runs them; it verifies a compact proof and
moves money according to the result. The L1 stays simple; the
applications don't have to be.

## What are you actually trusting?

Every design for "a game with a pot" answers the same question: who could
steal, and what stops them? It's a spectrum:

| Model | Who holds the pot | What stops them stealing | What's left to trust |
|---|---|---|---|
| **Custodial referee** | The operator | Reputation, law | Everything: they can just take it |
| **Multisig escrow** | A set of signers | M-of-N honesty | The majority of signers, and their liveness |
| **Zk-proven program** | The L1 itself | Proofs the L1 verifies | Only the code being proved, and that someone keeps the machine running |

Moving down the table removes trust in *people* one layer at a time. The
zk-proven model's trick is that "did the execution follow the rules?" stops
being a question about anyone's honesty; the operator can be a complete
stranger, run on junk hardware, in a bad mood, and still cannot produce a
settlement for a state the program's rules don't allow. The proof either
checks out on Kaspa or the settlement doesn't happen.

What *remains* is a different kind of trust, and it deserves plain words:
**liveness** and **data availability** (you can see the state you need to
act). Liveness, honestly: a settlement only exists if someone executes
actions and settles proofs, and today that someone is the operator. No
operator key is needed anywhere in the machine: the on-chain scripts check
proofs, never signatures, so in principle anyone can run the stack and
settle. But no permissionless escape-hatch flow is shipped yet either. If
every operator of a covenant stops before your balance has become a
committed exit, your funds wait until someone resumes the stack. The
machine is built to make "someone" cheap to become (the lane, the chain,
and the proofs are all public), but cheap is not automatic. We'll meet
both trusts again, with their machinery, in the chapters ahead.

## The shape and the rules

One idea runs through everything that follows, and it is worth stating
plainly now:

> **The rollup fixes the shape; the program picks the rules.**

The framework fixes the *shape* of the machine: user actions are signed,
state changes are proved, settlements land on Kaspa, money exits through
enforced doors. But the *rules* (what a deposit requires, who may move
which funds, how the state is derived, even how exits work) are not laws
of nature. They are choices made by each program. vprogs ships solid,
ready-to-use implementations of all of them (deposit logic, lockers and
signers, the permission tree, the exit mechanism); treat each as a
battery: a standard part you can use as-is, and in principle replace with
a different design.

That is the whole difference between "a chain with contracts" and "a
program with its own proved world". The next chapter starts the tour where
it matters most: the handful of transaction types everything else is built
from.
