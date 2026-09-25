# The transaction vocabulary

The whole machine — deposits, games, withdrawals, settlements — is built from
four kinds of transaction. Two live on the L1, one lives in the lane, and one
is the settlement itself. Learn these four and every chapter after this is
just consequences.

## The state transition (settlement)

The settlement is the tx that matters: it is the only thing that advances the
program's authoritative state, and it does so on Kaspa itself. A settlement
attests three things at once:

- a **state digest** — the program's new state root. Inside every program
  state lives in a sparse Merkle tree, and the digest is its 32-byte root:
  a cryptographic fingerprint of *every account, every game, every balance*,
  at one moment. Change one satoshi somewhere and the digest is a completely
  different number. The proof's central claim is always of the form "state
  root X became state root Y by executing the rules correctly".
- a **lane tip** — the head of the program's action lane (next section) at
  the moment of settlement: "I have processed everything up to here."
- a **block proof point** — the L1 block the whole claim is proven against:
  "and the L1 world I saw was real up to this block."

These three ride directly in the settlement transaction's script data, and
the settlement's outputs chain to the next settlement: output 0 is a P2SH
continuation that only the *next* valid settlement can spend. So on L1
itself there grows a single unbroken chain of settlements — each one
inheriting its predecessor's covenant and committing the next state digest.
That chain *is* the program's history; you can walk it on any Kaspa
explorer.

## The lane (subnetwork) and its key

User actions can't just be whispered to the operator — then nobody could
prove what was submitted, when, or in what order. Instead, each program
instance owns a **lane**: an L1 subnetwork where its users' actions are
published as ordinary Kaspa transactions. Every entry in the lane chains to
the one before it, so the lane has a single well-defined head — the **lane
tip** — and the L1 node itself can prove what the lane contained up to any
block.

The lane is the program's front door and its data availability in one: publish
there and the operator *must* eventually see your action; prove from there
and a verifier *knows* nothing was left out. The lane is identified by a
**lane key** — the hash of its subnetwork id — and every proof names the lane
it settles.

## User action txs

Everything a user does inside the program is a signed action: in tt that's
transferring balance, rotating your lock, depositing, withdrawing, creating a
game, joining a game, placing a mark, forfeiting an expired turn. An action
carries its author's authorization (more on locks and signers below) and is
published to the lane. What makes an action *valid* — whose signature, which
state it may touch, how much stake a game locks, what happens when your turn
timer expires — is not L1 law. It is the program's own logic, checked inside
the proof. The L1 neither knows nor cares what a "game" is; it only carries
the action to the machine that does.

## Deposits

A deposit is how value enters: an ordinary L1 output, paid to the program's
deposit address, credited to a user by the program's rules. The shape is
fixed — an L1 output, recognized by the program, proven into the state —
but the *address policy* is the guest's choice. tt's deposit policy derives
one shared deposit address from the covenant; the framework comment that
ships alongside it notes a per-user address scheme as an equally valid
choice. Same battery, different placement.

## Exits and the permission tree

An exit is how value leaves: the program debits a user and emits an
entitlement to withdraw on L1. The entitlements accumulate in the
**permission tree** — a Merkle accumulator whose leaves are
"(L1 script, amount)" pairs: who may claim how much, by L1 lock type. Each
settlement carries the tree's current commitment in a dedicated P2SH output,
and a user claims by spending from it on L1 — a permission spend that names
the covenant and proves its leaf.

Like the deposit policy, the permission tree is a ready-made part: the
framework ships the accumulator, the L1 commitment format, and the claim
flow. A program with different needs could, in principle, ship its own exit
design — the settlement shape would not change.

## Shape versus rules

| Fixed by the rollup shape | Chosen by the program (batteries included) |
|---|---|
| Actions are signed and published to a lane | What the actions *are* (tt: `CreateGame`, `PlaceMark`, …) |
| State is a digest; settlements chain digests on L1 | What lives in the state (resources, balances, boards) |
| Value enters by L1 deposit, leaves by proven exit | The deposit address policy |
| Settlements prove execution against L1 blocks | The exit mechanism (permission tree is the standard part) |
| — | Locks, unlockers, signers: how users authorize |

That last row deserves its own sentence: **locks, unlockers, and signers are
guest preferences too**. The framework provides common implementations — a
lock names who may act (tt uses pubkey locks), a signer proves control of
it, an unlocker pairs them — and a program can compose or replace them.
Every battery above is real, shipped code in vprogs today; "replaceable"
means the *shape* doesn't depend on which one you use.

One term is left, and it names the whole thing: a **covenant id** — the
32-byte identity of one program instance. Deposits pay into it, settlements
chain within it, exits reference it, and at bootstrap it is pinned together
with the guest-ELF image ids the program runs — the covenant is "this
program, these exact rules, this instance". In tt's live deployment it is
literally a constant the operator funds.

With the vocabulary in hand: how do proofs tie all four tx types together
across multiple L1 blocks?
