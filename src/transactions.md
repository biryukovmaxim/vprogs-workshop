# The transaction vocabulary

Start where an L1 observer stands. Kaspa sees four kinds of transaction
from this machine, and none looks special on-chain: they are ordinary
transactions with particular scripts and payloads. The program's own
world, the L2, is what those bytes mean once interpreted. This chapter
walks both views: what lands on Kaspa, then what the program makes of it.

- a **settlement** spends the covenant's continuation output and carries a
  new state digest plus pointers in its script data
- a **lane entry** is an ordinary transaction tagged for the program's
  subnetwork, carrying a signed action
- a **deposit** is an ordinary payment to the program's deposit address
- an **exit claim** spends the permission output and pays a user out

Two are plain Kaspa usage (lane entries, deposits); two are the machine's
own script shapes (settlements, claims). Learn the four and every chapter
after this is just consequences.

## The state transition (settlement)

The settlement is the tx that matters: it is the only thing that advances the
program's authoritative state, and it does so on Kaspa itself. A settlement
attests three things at once:

- a **state digest**: the program's new state root. The full state (every
  account, every game, every balance) lives off-chain, served by the
  operator's DA; what Kaspa holds is one 32-byte root, a fingerprint of
  all of it at one moment. The structure behind the fingerprint is a
  *sparse Merkle tree*, and it exists for exactly this split: one root can
  commit to an entire off-chain world, and, because the tree is sparse
  (every possible position exists), proving that one account was inside
  takes a short branch of hashes and reveals nothing else. Change one
  sompi somewhere and the digest is a completely different number. The
  proof's central claim is always of the form "state root X became state
  root Y by executing the rules correctly".
- a **lane tip**: how far execution had read the program's action lane
  (next section) when the snapshot was taken: "I have processed every
  published action up to here."
- a **block proof point**: the last L1 block whose data execution
  consumed: "and the L1 world I saw was real up to this block."

Read a settlement as a snapshot claim, not a switch. It says: at block Y,
the state digest was D. The proof inside shows how the digest got there,
root by root, from block X to block Y, where X is the previous
settlement's proof point; the windows are contiguous, each bundle's
journal continuing exactly where the last one ended, so no block range
goes unwitnessed. The settlement transaction itself
lands at least one block after Y, and sometimes later. Each settlement
pins one more provable snapshot; nothing starts applying "from now on".

These three ride directly in the settlement transaction's script data, and
the settlement's outputs chain to the next settlement: output 0 is a P2SH
continuation that only the *next* valid settlement can spend. So on L1
itself there grows a single unbroken chain of settlements, each one
inheriting its predecessor's covenant and committing the next state digest.
That chain *is* the program's history; you can walk it on any Kaspa
explorer.

## The lane (subnetwork) and its key

User actions can't just be whispered to the operator, because then nobody
could prove what was submitted, when, or in what order. Instead, each
program instance owns a **lane**: an L1 subnetwork where its users' actions
are published as ordinary Kaspa transactions. The entries are sequenced by
the node's own commitment machinery (KIP-21): Kaspa's consensus itself
orders the lane and can prove, up to any block, exactly what it
contained. So the lane has a single
well-defined head, the **lane tip**, and the L1 node itself can prove what
the lane contained up to any block.

What is a lane, physically? Ordinary Kaspa transactions. A lane entry is a
real transaction, paying a real Kaspa fee from the user's own funds, mined
by the network's miners like any payment. The subnetwork is a label the
node's consensus tracks: it gossips, orders, and accounts for lane traffic
alongside ordinary payments, and (this is the load-bearing part) the node
itself will hand anyone a cryptographic proof of what the lane contained up
to any confirmed block. There is no lane operator to refuse an entry; the
front door is the Kaspa mempool.

The lane is the program's front door and its data availability in one:
publish there and the operator *must* eventually see your action; prove
from there and a verifier *knows* nothing was left out. The lane is
identified by a **lane key**, the hash of its subnetwork id, and every
proof names the lane it settles.

## User action txs

Everything a user does inside the program is a signed action: in tt that's
transferring balance, rotating your lock, depositing, withdrawing, creating
a game, joining a game, placing a mark, forfeiting an expired turn (clocks
inside the program are L1 block timestamps, carried in the batch metadata
the proof commits, so "expired" is a chain fact, not the operator's watch).
An action carries its author's authorization (more on locks and signers
below) and is published to the lane. What makes an action *valid* (whose
signature, which state it may touch, how much stake a game locks, what
happens when your turn timer expires) is not L1 law. It is the program's
own logic, checked inside the proof. The L1 neither knows nor cares what a
"game" is; it only carries the action to the machine that does.

## Deposits

A deposit is how value enters: an ordinary L1 output, paid to the program's
**deposit address** (derived from the *covenant id*, the 32-byte identity
of this program instance, formally introduced at the end of the chapter),
and credited to a user by the program's rules. In tt the deposit
*is* the action: the Kaspa transaction you publish to the lane carries your
signed deposit action (naming the account to credit, and your lock if the
account is new) and, in the same transaction, an output paying the deposit
address. The proof checks that the output exists, pays the covenant-derived
script, and credits exactly the account your signature named. Nobody can
steer your deposit to a different account, and the landed coins sit at a
script that only proven exits can unlock, never at an operator's key.

The shape is fixed (an L1 output, recognized by the program, proven into
the state), but the *address policy* is the guest's choice (the *guest* is
the program's own code running inside the zkVM; chapter 6). tt's deposit
policy derives one shared deposit address from the covenant; the framework
comment that ships alongside it notes a per-user address scheme as an
equally valid choice. Same battery, different placement.

## Exits and the permission tree

An exit is how value leaves: the program debits a user and emits an
entitlement to withdraw on L1. The entitlements accumulate in the
**permission tree**, an accumulator (a Merkle structure that answers one
question: is this leaf in?) whose leaves are
"(L1 script, amount)" pairs: who may claim how much, by L1 lock type. A
settlement whose bundle emitted exits carries the tree's current commitment
in a dedicated P2SH output (a bundle with no exits settles without one),
and a user claims by spending from it on L1: a permission spend that names
the covenant and proves its leaf. The claim is a self-updating UTXO: your
spend proves your leaf against the committed root, pays you, and must
re-commit the accumulator with your amount deducted for
everyone still waiting. Paying you works like making change: your claim
transaction pulls in whole coins from the program's deposit pile (only
this script shape can unlock them; your wallet picks which), the script
checks on-chain that what was pulled covers your amount, you receive your
leaf's value, and any leftover is re-locked at the same script in the same
transaction, for the people still in line. The same exit cannot be claimed
twice over: the UTXO you spent is gone, and the new commitment no longer
contains your leaf.

Claims serialize through that one UTXO: heavy exit traffic queues, and two
claims racing in the mempool conflict like any double-spend. One wins;
the other's transaction can no longer confirm (its input is gone), and the
wallet rebuilds it against the new commitment once it sees the winner.
And newer
settlements commit the accumulator *after* deducting every claim the
machine has watched land on L1, so an entitlement already claimed against
an older commitment is simply absent from the newer one; commitments
cannot be replayed either. A malformed claim fails its own validation; it
cannot jam anyone behind it.

Like the deposit policy, the permission tree is a ready-made part: the
framework ships the accumulator, the L1 commitment format, and the claim
flow. A program with different needs could, in principle, ship its own exit
design; the settlement shape would not change.

## Shape versus rules

| Fixed by the rollup shape | Chosen by the program (batteries included) |
|---|---|
| Actions are signed and published to a lane | What the actions *are* (tt: `CreateGame`, `PlaceMark`, …) |
| State is a digest; settlements chain digests on L1 | What lives in the state (resources, balances, boards) |
| Value enters by L1 deposit, leaves by proven exit | The deposit address policy |
| Settlements prove execution against L1 blocks | The exit mechanism (permission tree is the standard part) |
|  | Locks, unlockers, signers: how users authorize |

That last row deserves its own sentence: **locks, unlockers, and signers are
guest preferences too**. The framework provides common implementations: a
lock names who may act (tt uses pubkey locks), a signer proves control of
it, an unlocker pairs them. A program can compose or replace them.
Every battery above is real, shipped code in vprogs today; "replaceable"
means the *shape* doesn't depend on which one you use.

One term is left, and it names the whole thing: a **covenant id**: the
32-byte identity of one program instance. Deposits pay into it, settlements
chain within it, exits reference it, and at bootstrap it is pinned together
with the guest-ELF image ids the program runs; the covenant is "this
program, these exact rules, this instance". The pinned rule-set is public
on-chain: the covenant's script hash is computed *from* the pinned image
ids, so anyone can take a claimed rule-set, recompute the hash, and check
it against the address before depositing. The tooling for that check is a
script, not a website, today. In tt's live deployment the covenant is
literally a constant the operator funds. That constant is the settlement
chain's first link: at bootstrap the operator funds an initial output
locked by the covenant's script at its genesis state, and every
settlement descends from it.

With the vocabulary in hand: how do proofs tie all four tx types together
across multiple L1 blocks?
