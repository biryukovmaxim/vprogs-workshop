# Based rollup on Kaspa

## The one-sentence version

A based rollup moves a program *off* the L1 (its state, its rules, its
execution) but keeps its *enforcement* on the L1: every state change
worth trusting is proved and settled back to Kaspa. That off-chain half
is the **L2**: the program's own world, using Kaspa for what it must not
provide itself.

What each side carries follows from that. Ordering, verification, and
enforcement are the L1's: users publish straight to the program's lane
and the chain's own order is the order, consensus checks every
settlement's proof before it counts, and payouts move only through
scripts Kaspa itself enforces. The state itself is the one thing the L1
does not carry: it holds a single 32-byte fingerprint of it (chapter 4),
so the full data behind that fingerprint, every account, balance, and
game, is stored and served by an L2 provider, the operator's node. The
actions and deposits do sit fully on the chain, in the lane. No external
committee, no data-availability service, and no bridge token (no new
coin standing in for the locked KAS) sits in between.

A note for readers from the Ethereum world (others can skip this
paragraph): in Ethereum discourse "based" means L1 proposers do the
sequencing. Here nobody sequences at all; ordering is not a service:
users publish actions straight to the lane, and the L1's own order *is*
the order. One Ethereum connotation does not transfer: there, based also
buys forced inclusion, an L1 path that makes the rollup process your
transaction even if the sequencer refuses. This machine ships no such
forced path; the liveness limits are chapter 6's. Readers who prefer the
established name for this shape will find it in chapter 10: a sovereign
app.

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
public inbox and its data availability. The operator is not in the loop
for submissions: it reads the lane and the chain directly and executes
actions off-chain. Provers produce cryptographic proofs that the
execution followed the program's rules. Settlements, carrying those
proofs and commitments to the new state, land on Kaspa. Money flows back
out to users through exits, enforced by those settled proofs.

One arrow in the diagram needs a note: apps read current state from the
operator's index (the dashed line), not from Kaspa directly. The index
is a convenience, not the authority; chapter 9 covers what a skeptic can
do without it.

## The two limits, and the extensions that change them

Kaspa's script engine is real and richer than Bitcoin's: it concatenates,
hashes, does arithmetic, inspects its own transaction's inputs and
outputs, and, since [KIP-16](https://github.com/kaspanet/kips/blob/master/kip-0016.md), verifies zk proofs. Ecosystem projects
(SilverScript, Argent among them) are building higher-level abstractions
over it. The claim that Kaspa has "no smart contracts because no
scripting" is wrong. The limits are structural, and there are two.

First, Kaspa runs on UTXOs: every coin is an output consumed by exactly
one transaction, and no other transaction can reference it afterward.
Shared state has no native home, so a program must carry its pot, board,
and balances forward output by output (abstractions like SilverScript
make the threading easier; it is still manual work).

Second, the script language is deliberately not Turing-complete (it
cannot run arbitrary programs, only check fixed shapes): flexible enough
to lock and check, not to *be* an application.

Both limits are deliberate design choices: they keep the chain fast and
simple to reason about. Both also received protocol extensions through
Kaspa's own proposal process (KIPs, the network's improvement
proposals), activated on Kaspa mainnet by the Toccata hard fork, a
coordinated upgrade. [KIP-20](https://github.com/kaspanet/kips/blob/master/kip-0020.md) added covenant ids, so scripts can bind
outputs to one program instance's identity. [KIP-21](https://github.com/kaspanet/kips/blob/master/kip-0021.md) added lane
commitments: the node's consensus anchors, references, and proves the
subset of transactions belonging to one lane. Proof verification is
therefore a consensus rule, not a service: every Kaspa node that
executes a settlement runs the check, and a bad-proof settlement is
invalid, rejected like a bad signature. The check is verification, not
re-execution: small for every node, and priced into the settlement's own
transaction like any script work. Which proof system and which program
version to trust are not choices made at spend time; they are baked into
the covenant's script hash itself, which chapter 4 opens up.

Where this lives today: those extensions are **implemented and
activated on Kaspa mainnet**. The live demo settles on **public
testnet-10** (mined by its public miners), and that is a deployment
choice, not a protocol gap: a demonstration belongs on a demonstration
network, with deliberately looser security assumptions. Every vprogs
settlement you can watch today is a testnet fact; what separates the
demo from a mainnet deployment is operational work, not protocol
activation.

With these three extensions, a based rollup removes both limits: the
program's state lives in the proved world, in whatever shape the program
likes, and the rules can be arbitrary code, because the chain never runs
them; it verifies a compact proof and moves money according to the
result. The L1 stays simple; the applications do not have to be.

## What are you actually trusting?

Every design for "a game with a pot" answers the same question: who
could steal, and what stops them?

| Model | Who holds the pot | What stops them stealing | What's left to trust |
|---|---|---|---|
| **Custodial referee** | The operator | Reputation, law | Everything: they can just take it |
| **Multisig escrow** | A set of signers | M-of-N honesty | The majority of signers, and their liveness |
| **Zk-proven program** | The L1 itself | Proofs the L1 verifies | The code being proved, the zkVM and verifier opcode underneath it (chapter 7), and that someone keeps the machine running |

Moving down the table removes trust in *people* one layer at a time.
The zk-proven model's key property is that "did the execution follow the
rules?" stops being a question about anyone's honesty: the operator can
be anyone and still cannot produce a settlement for a state the
program's rules don't allow. The proof either checks out on Kaspa or
the settlement doesn't happen.

One more point the table compresses: the deposit pile is pooled custody
at a script, and its safety is exactly the safety of the pinned code,
bugs included. A rule-set bug that pays the wrong hands drains the pile
through perfectly valid proofs. That is what "trusting the code" means,
and it is why the pinning in chapter 4 matters.

What remains is a different kind of trust: **liveness** and **data
availability** (you can see the state you need to act). Liveness: a
settlement only exists if someone executes actions and settles proofs,
and today that someone is the operator. No operator key is needed
anywhere in the machine: the on-chain scripts check proofs, never
signatures, so in principle anyone can run the stack and settle. But no
permissionless escape-hatch flow is shipped yet either. If every
operator of an instance stops before your balance has become a committed
exit, your funds wait until someone resumes the stack. The lane, the
chain, and the proofs are all public, so becoming that "someone" is
cheap in principle, but not automatic. Chapters 6 and 9 return to both
points.

## The shape and the rules

One idea runs through the rest of the book:

> **The rollup fixes the shape; the program picks the rules.**

The framework fixes the *shape* of the machine: user actions are signed,
state changes are proved, settlements land on Kaspa, money exits through
enforced doors. But the *rules* (what a deposit requires, who may move
which funds, how the state is derived, even how exits work) are choices
made by each program. vprogs ships working implementations of all of
them (deposit logic, lockers and signers, the permission tree, the exit
mechanism); treat each as a battery: a standard part you can use as-is
and, in principle, replace with a different design.

The next chapter covers the handful of transaction types everything else
is built from.
