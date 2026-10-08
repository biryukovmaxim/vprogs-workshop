# The machinery

[Chapter 5](chaining.md) showed how proofs tie the pieces together across L1 blocks.
This chapter describes who runs what. Five roles cover the whole machine.

```mermaid
flowchart TB
    BR["Bridge: watches confirmed L1, feeds witnesses from lane and deposits"]
    EX["Executor: runs the program's rules over actions and deposits"]
    PR["Provers: transaction -> batch -> aggregate proofs"]
    ST["Settler: builds and submits settlement txs"]
    DA["DA / index: serves program state to apps"]
    BR --> EX --> PR --> ST
    BR --> DA
```

[**The bridge**](https://github.com/kaspanet/vprogs/tree/055ae28a/l1/bridge) is the machine's view of L1. It follows the Kaspa chain
behind the confirmation window and turns confirmed lane entries and
deposits into the inputs execution consumes. Users submit to the lane
themselves; the bridge only reads. Every fact the machine holds about
L1 arrives through it.

**The executor** applies the program's rules, the guest code, to the
witnessed actions. It is deterministic: same inputs, same state, same
result. What it computes is defined by the program, not by the executor.

**The provers** produce proofs of the execution, in three stages, one
guest program each: the transaction guest proves one transaction, the
batch guest compounds a block of those proofs, and the aggregator
compounds batches into the single proof a settlement carries ([The zkVM, briefly](zkvm.md)
details the pipeline). Batch proofs chain within a bundle, and bundles
chain across settlements, so the pipeline is a chain by construction.

**The settler** [builds the settlement transaction](https://github.com/kaspanet/vprogs/blob/055ae28a/l1/wallet/src/build/settlement.rs) (state digest, lane
tip, proof, continuation and permission outputs) and submits it to
Kaspa.

**The DA/index layer** is the read side: an operator can serve the
program's current state over an API so apps can query it without
replaying proofs. [Chapter 9](building-an-app.md) builds on it.

In tt's deployment these roles are in-process components of the single
`ttd` daemon (the framework's [reusable runner engine](https://github.com/kaspanet/vprogs/tree/055ae28a/runner)), plus the web app
reading the DA APIs. The roles are independent of the packaging: a
bigger deployment could scale each separately.

## Who are you trusting, again?

With the roles named, the trust question from [Based rollup on Kaspa](based-rollup.md) gets concrete.
The bridge can't invent L1 facts; the proofs check everything against
the real chain. The executor can't cheat; its output is proven. The
settler can't settle a fabricated state; Kaspa verifies the proof before accepting
the tx, and the settlement chain can't fork without splitting real
money on L1. What the operator *can* do is stop: stop executing, stop
proving, stop settling, or keep settling against an old lane tip so
your action is never included. That is the liveness trust: **safety
needs no operator; liveness does, until someone else takes over.**

Two properties make "someone else" possible. First, nothing in the
machine is operator-keyed: the settlement script checks proofs and ends
without any signature (a valid proof from anyone extends the chain),
the bridge only reads public chain data, and the lane is public. So
anyone can, in principle, stand up this same open stack against the
same lane and continue where the last operator stopped. The limits of
that claim: resuming means running a proving stack, real work at real
cost ([Building an app on it](building-an-app.md) says who would pay), so it is a capability, not a
service anyone promises. Second, exits already committed to the
permission tree are claimable by their holders alone; no operator sits
in that loop.

Permissionless settlement is also open to griefers. A griefer with a proving
stack can settle empty extensions: bundles that execute nothing new and
leave the lane tip behind its true head, so pending actions (yours,
perhaps) stay unsettled for as long as the griefer keeps winning. Both
sides spend the same continuation output, so each link is a mempool
race: whoever confirms first wins, the loser's settlement dies with its
input, and its proving work is wasted. The honest side can be drawn
into losing races the same way. Nothing on-chain punishes any of this;
cost is the only limit; both sides pay it. What the griefer cannot
touch is safety: state stays unforgeable, committed exits stay
claimable. What stalls is liveness.

What is *not* shipped today is an escape hatch: a flow that lets a user
force a settlement through without first running the machine. Until one
exists, a balance that never became a committed exit waits for an
operator. The practical advice: exit early if you plan to leave.
