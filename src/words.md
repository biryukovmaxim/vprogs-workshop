# The words you need

Blockchain writing leans on a small vocabulary. Here is all of it, in plain
words; the rest of the book assumes these.

- **Kaspa**: the L1 this book runs on. What matters here is its blockdag
  and its UTXO model, both below.
- **L1**: "layer 1", the Kaspa network itself. The thing that actually
  holds the money.
- **Output (UTXO)**: a piece of KAS sitting at a lock. A Kaspa payment
  does not edit a number in a database; it spends existing outputs and
  creates new ones. "Your money" is the set of outputs only you can spend.
  And an output lives in exactly one transaction: once spent it is gone,
  and no other transaction can reference it. That one-way rule is why
  shared on-chain state is hard here (chapter 2).
- **Lock, script, P2SH**: the condition under which an output may be
  spent. *Pay to public key*: whoever proves control of a secret key.
  *Pay to script hash*: whoever satisfies a small program whose
  fingerprint is baked into the address. P2SH is how this machine writes
  its own rules onto Kaspa.
- **Covenant**: the 32-byte identity of one program instance: its deposit
  address, its action lane, and the exact rule-set version it proves, all
  bundled into one name.
- **Guest**: the program's own code, running inside the proving machine
  (the zkVM), as opposed to the framework around it.
- **Lane**: the program's public inbox: a labeled stream of ordinary Kaspa
  transactions carrying users' signed actions. Miners mine them like any
  other payment; there is no gatekeeper to refuse an entry.
- **Witness**: the confirmed Kaspa facts the machine feeds to the program:
  which actions landed, which deposits paid, what the blocks said. Not a
  courtroom; a data feed.
- **Proof, receipt**: a few kilobytes of mathematics that convince anyone,
  without re-running the program, that a claimed execution really happened.
- **Merkle tree**: a fingerprint of fingerprints. Leaves are hashed in
  pairs up to one root, so a short list of branch-hashes proves a leaf
  belongs under that root. *Sparse* means every possible position exists,
  so proving "your account" never requires revealing anyone else's. An
  *accumulator* is the same idea trimmed to one question: "is this leaf
  in?"
- **Runtime**: the layer of code that checks and applies each action; the
  rules of the house. Chapter 7 is about who owns it.
- **Reorg (reorganization)**: now and then the network briefly agrees on
  one block order, then switches to another; the switched-away blocks
  "vanish". Shallow churn like this is routine and expected; deeply buried
  blocks essentially never reorganize, and "essentially" is why the
  machine also waits out a confirmation window before trusting fresh
  blocks. Money that moved long ago is as safe as Kaspa itself.
- **Data availability**: the guarantee that you can fetch the full record
  of what was published, yourself, from the network, not just trust
  someone's summary of it.
