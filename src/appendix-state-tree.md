# Appendix: the state tree

[Chapter 9](building-an-app.md) hands out the chain of anchors in one line: resource id,
leaf, root, settled digest. This page is the tree itself: its shape,
where the bytes live, how proving uses it, and the walk that checks a
single account against L1.

## Shape

The whole program state is [one sparse Merkle tree](https://github.com/kaspanet/vprogs/tree/055ae28a/core/smt) of depth 256. The
key is the resource id, 32 bytes, one bit of it per level, so every
possible account has one fixed position; empty branches collapse into
a default hash, which keeps the tree as small as the live state while
every account keeps its spot. The leaf commits the id together with
the hash of the resource's bytes (not the bytes themselves), which
binds each value to its position, and a hash of empty bytes marks
deletion. The tree is versioned: a state change is a set of
(key, new value hash) pairs applied at a version, and the state digest
the settlements carry is the root that results. Versions keep past
roots reachable, so a prover can prove at the exact state a batch saw,
not only at the latest tip. Sparse plus
depth-256 is also what makes a proof short: proving one leaf is a path
of at most 256 branch hashes, and the proving host batches the
accounts one execution touches into a single multiproof over all of
them.

## Where the bytes live

The bytes behind the leaves are not on L1; L1 holds only each root.
The executing node keeps them, and the tree, in its own database (a
RocksDB store with a column family per kind: resource data, tree
nodes, version pointers). The store is a cache with a
proof: every input execution consumed, the actions in the lane, the
deposits, the block context, is public on Kaspa, so replaying them
reproduces the same tree, root for root. A rebuild seeds at the
covenant's deployment, not at a recent settlement: the whole history
is the witness, and starting near the tip would miss lane entries the
state already absorbed.

## Proving with the tree

The guests meet the tree at two levels. The transaction guest works
on whole resources: it receives the full bytes of every account its
transaction touches, checks the program's rules over them, and commits
the new bytes' hashes. Merkle paths enter one level up, at the batch
guest. The proving host reads from its store a multiproof covering
exactly the resources the batch touches and passes it in as private
input (input the receipt never publishes; what the receipt commits is
the result, the two roots); the batch guest checks the proof's keys are strictly ordered,
checks that every resource the transactions touched is accounted for
in it, and recomputes both roots, before and after, in one walk over
the witness. Those two roots leave the proof as the journal's
prev_state and new_state; the aggregator asserts each batch's
prev_state equals the previous batch's new_state, and the settlement
script pins the same chaining on L1. After proving, the host checks
its own store's root equals the root the guest committed, so a broken
store cannot quietly produce valid proofs.

## Checking one account against L1

The verification [Building an app on it](building-an-app.md) sketches, as a walk:

1. Read the settled digest from L1. The settlement's own data names
   the digest it moved to, and its continuation output locks that
   digest into the next link.
2. Get the account's resource id, its bytes, and the Merkle path from
   its leaf to the root. tt's node serves the bytes today; it does not
   serve the path.
3. Hash the bytes, hash them into the leaf with the id, walk the path,
   and compare the root you get with the digest from step 1.

The proving host runs this same walk for every touched account on
every batch, so every piece exists; the missing part is the endpoint.
Nothing in the check is operator-specific: the path either hashes to
the settled root or it does not. What the walk cannot cover is state
newer than the last settlement; that gap is [Building an app on it](building-an-app.md)'s, and only the
next settlement's proof or your own re-execution closes it.
