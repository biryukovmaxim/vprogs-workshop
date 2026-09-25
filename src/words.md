# The words you need

Blockchain writing leans on a small vocabulary. Here is all of it, in plain
words; the rest of the book assumes these.

- **Kaspa**: the proof-of-work network this book runs on; its coin is
  KAS. What matters here is its blockdag and its UTXO model, both below.
- **L1**: "layer 1", the Kaspa network itself. The thing that actually
  holds the money.
- **L2**: "layer 2": a system that does its work off the L1 while leaning
  on the L1 for what must be trusted: ordering, data availability, and
  settlement. A *rollup* is the common L2 shape: execute off-chain, then
  prove or commit the results back on-chain. The machine in this book is
  one.
- **KIP**: Kaspa Improvement Proposal: the process by which the network
  proposes, reviews, and activates protocol changes.
- **Output (UTXO)**: a piece of KAS at a lock. Every Kaspa transaction
  consumes earlier outputs (as its inputs) and creates new outputs; one
  that no transaction has consumed yet is an *unspent transaction
  output*. "Your money" is the set of UTXOs your key can unlock. And an
  output lives in exactly one transaction: once spent it is gone, and no
  other transaction can reference it. That one-way rule is why shared
  on-chain state is hard here (chapter 2).
- **SPK, P2PK, P2SH**: the locking half of an output is its *SPK*
  (script public key), a small program stored inside the output. A later
  transaction spends that output by supplying input data that satisfies
  the SPK. *P2PK*: the SPK demands a signature from one specific public
  key, and the address is derived from that key. *P2SH*: the SPK stores
  only the hash of a script; the spender reveals the script, shows its
  hash matches, and the revealed script then runs and must succeed. P2SH
  is how this machine puts its own rules onto Kaspa: the program is
  committed from the moment the output exists, but only seen at spend
  time.
- **Covenant**: the 32-byte identity of one program instance: its deposit
  address, its action lane, and the exact rule-set version it proves, all
  bundled into one name.
- **Guest**: the program's own code, running inside the proving machine
  (the zkVM), as opposed to the framework around it.
- **Lane**: the program's public inbox: a labeled stream of ordinary Kaspa
  transactions carrying users' signed actions. Miners mine them like any
  other payment; there is no gatekeeper to refuse an entry.
- **Journal**: the fixed-format record inside each proof: the previous and
  new state roots, the lane tips, and the L1 context the execution saw.
  The settlement script hashes exactly these bytes into the digest the
  receipt commits.
- **Proof, receipt**: a few kilobytes of mathematics that convince anyone,
  without re-running the program, that a claimed execution really happened.
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
