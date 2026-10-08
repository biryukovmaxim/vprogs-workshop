# Appendix: settlement and exit mechanics

[Chapter 4](transactions.md) kept the settlement and the exit to what they attest and pay,
and [How it all chains](chaining.md) kept the block model to its simplified form; this page
carries the mechanics a careful reader asks about next. Nothing here
changes the design; it is the same machine, closer up.

## What counts as one block

[Chapter 5](chaining.md) treats one selected-chain block as one block to the machine.
The full rule: a block on the selected chain absorbs the parallel
blocks consensus folds into it, its *mergeset*, and the transactions
those parallel blocks carry count as part of that step, provided they
were not counted in an earlier step. One chain block plus its
mergeset's uncounted transactions is one block to the machine: one
witness set, one batch, one step of execution. The dag's parallelism
is flattened before execution begins, which is why the rest of the
book can speak of a plain sequence of blocks.

## Snapshot claims, chained

A settlement records one verified snapshot: at block Y,
the state digest was D. The proof inside shows how the digest got there,
root by root, from block X to block Y, where X is the previous
settlement's proof point. The windows are contiguous, each proof's
journal continuing exactly where the last one ended, so no block range
goes unwitnessed. The settlement transaction itself lands at least one
block after Y, and sometimes later: the gap is proving time plus the
confirmation window the cited blocks must pass, before the settlement is
even submitted. Each settlement commits one more provable snapshot.

## Can a range be skipped?

No. Each proof's journal picks up exactly where the previous one ended,
at its lane tip, and the settlement names the tip it proves to. Skip a
stretch of the lane and the two tips no longer meet: the next proof
would claim a tip it never reached. The bind is checked on-chain:
Kaspa's block headers carry a commitment to every active lane
([KIP-21](https://github.com/kaspanet/kips/blob/master/kip-0021.md)),
and the settlement script requires the proof's journal to commit the
same value the chain carries for the block it names. A skipped range
fails that check, so a settlement cannot leave a hole.

## What if cited blocks reorganize?

The proving pipeline has tiers ([The zkVM, briefly](zkvm.md) lists the three guests), and a reorg
invalidates only what stood on the reorganized side. Bundles whose
proving base was rolled back are thrown away and rebuilt against the
surviving chain, reusing the
lower-tier proofs that still chain; a settlement removed by a reorg
before it is confirmed is simply resubmitted. Proof cancellation is
the pipeline dropping work on bundles whose base was rolled back: the
bundle that would have stood on the losing side is not generated, and
one already in flight is abandoned, keeping its still-valid
lower-tier proofs for reuse. The aggregate prover ships this. This is
rare by construction: the machine only cites blocks already behind its
confirmation window ([How it all chains](chaining.md)).

## The anchor window and the long stall

The opcode that reads lane commitments (KIP-21's, a different one
from the [KIP-16](https://github.com/kaspanet/kips/blob/master/kip-0016.md) opcode that verifies proofs) has a reach limit: it serves
commitments only from a recent window of blocks, roughly the last
twelve hours' worth, so a
settlement cannot cite an anchor older than that. A machine stalled longer
than that must first prove its way forward to a recent block, then
settle. The chaining shape is what makes that recovery possible: a proof
window can span many blocks, so the backlog drains in larger windows, at
the price of later snapshots.

## An exit claim, up close

[Chapter 4](transactions.md) kept exits to the leaf and the payout; here is the claim
transaction itself. It pulls in the commitment UTXO, up to eight whole
coins from the program's deposit pile (the claim wallet, the
builder-side tool, picks which; the cap of eight is protocol, not
wallet policy, because it sizes the script itself), and one ordinary
coin of the builder's own to carry the fee. A claim pays a whole leaf
or the part the swept coins cover; the unlock data names the deduct.
The outputs, in the order the script pins them: output 0 is the
payout, the deducted value paid to the lock the leaf names; output 1
is the fresh commitment with the paid part removed, serving the next
claimant; then the pile's untouched change, re-locked at the same
deposit lock for the remaining claimants; then the unspent part of
the fee coin back. The commitment is a self-updating UTXO: each claim
spends it and hands the next claimant a tree with less left to claim.

```mermaid
flowchart LR
    A["the commitment<br/>(this settlement's exits)"] --> C["one claim transaction"]
    B["up to 8 coins from<br/>the deposit pile"] --> C
    F["one of the claimer's own coins<br/>(carries the fee)"] --> C
    C --> P["payment to the claimer"]
    C --> N["new commitment,<br/>this leaf deducted"]
    C --> R["pile change, re-locked"]
    C --> X["fee-payer's change"]
```

What the script itself checks: the commitment's redeem
script [embeds the tree's root and the unclaimed-leaf
count](https://github.com/kaspanet/vprogs/blob/055ae28a/zk/backend/risc0/api/src/permission_script.rs#L718),
and the unlock data supplies the leaf, its branch of sibling hashes,
and the deduct. The script requires the deduct to be positive and no
more than the leaf's amount, requires output 0 to pay the leaf's
destination exactly the deduct, and requires the leaf hash, folded up
the branch, to equal the embedded root. A leaf hashes as SHA-256 over
a leaf tag, the destination's script bytes, and the amount; a branch
node as SHA-256 over a branch tag and its two children
([definitions](https://github.com/kaspanet/vprogs/blob/055ae28a/zk/backend/risc0/api/src/permission_tree.rs#L81-L88)).
The handover is also script work: the same branch
folded around the reduced leaf gives the next root, and the script
takes its own template bytes, as revealed in the claim's unlock data,
prefixed with the new root and count, hashes them into the next
commitment's script hash, and requires output 1 to be exactly that,
with the commitment's own value passing through unchanged. The count
rides along for the endgame: it drops by one only when a claim empties
a leaf whole, and reaching zero is what tells the script the tree is
done. While any leaf remains, the payout is exactly the deduct; when
the last leaf empties there is no output 1 left to carry the
commitment's value, so the script folds it into the final payout
instead (output 0 becomes the deduct plus the commitment's own
value). The deposit coins are conserved exactly and the fee burns
only from the claimer's collateral coin, so swept value cannot ride
out anywhere but the payout and the pile
([the full phase list](https://github.com/kaspanet/vprogs/blob/055ae28a/zk/backend/risc0/api/src/permission_script.rs#L178-L203)).

Two background facts close the money flow. First, the commitment's
own value: the settlement that emits exits funds the permission
output from the settler's own wallet, alongside the fee; the amount
is pinned in the covenant script, and none of it is program money.
Every claim passes it through unchanged, and the final claim folds it
into the payout, which is why the last claimant collects it. Second,
what the pile coins are locked with: each is a deposit that paid the
program's deposit address, a P2SH whose [redeem script derives from
the covenant id
alone](https://github.com/kaspanet/vprogs/blob/055ae28a/zk/abi/src/delegate_script.rs#L1-L25)
and demands that the spending transaction itself run under that same
covenant. No key ever unlocks a deposit; only the machine's own
covenant transactions can move one, and the claim script does not
take its funding on faith: it [rebuilds the expected lock bytes from
its own covenant id
on-chain](https://github.com/kaspanet/vprogs/blob/055ae28a/zk/backend/risc0/api/src/permission_script.rs#L530)
and counts only inputs that match, so coins from another program or a
plain wallet output cannot sneak in as funding.

And the tree's build site: each transaction's proof journals the
exits it emitted, and the aggregator guest, while proving the bundle,
replays them from those journals in canonical order (journal order
within a transaction, transaction order within a batch, batch order
within the
bundle, [accumulator](https://github.com/kaspanet/vprogs/blob/055ae28a/zk/abi/src/withdrawal/exit_accumulator.rs#L1-L23)),
into the one tree whose root the settlement carries.

Who builds a claim is economics, not protocol. The user can build it
and pay the fee; anyone can offer claims as a service; an operator who
benefits from working exits can subsidize them. The script accepts a
valid claim from whoever submits one.

## Exit contention and pile maintenance

Claims against the same commitment are spends of one coin, so they
conflict: two claims racing in the mempool behave like any double-spend.
One wins; the other's transaction can no longer confirm (its input is
gone), and the wallet rebuilds it against the new commitment once it
sees the winner. Claims against different settlements' commitments are
independent and pay out in parallel. The machine watches landed claims
to update its own books, marking exits claimed in program state, not
as an additional safety check, and a malformed claim fails its own
validation without
blocking later claims. What does not ship is a sweeper for the pile itself:
every claim splits what it sweeps, the network's dust rules floor how
small the pieces can get, and keeping the pile in spendable coins is
operational work.
