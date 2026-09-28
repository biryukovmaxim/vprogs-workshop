# The words you need

Blockchain writing leans on a small vocabulary. Here is all of it, in plain
words; the rest of the book assumes these.

- **Kaspa**: the proof-of-work network this book runs on; its coin is
  KAS. What matters here is its blockdag and its UTXO model, both below.
- **Blockdag**: the shape of Kaspa's history. On most chains each block
  names one parent, so the history is a single line; on Kaspa a block may
  name several parents, and the history is a web. Consensus rules weave
  that web into one agreed order of transactions, roughly a block per
  second, and the weaving is why Kaspa is fast and why shallow reorg
  churn (the Reorg entry below) is routine weather rather than an
  emergency.
- **Sompi**: the smallest unit of KAS; one KAS is 100,000,000 sompi.
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
  on-chain state is hard here (chapter 3).
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
- **Settlement**: the transaction that commits one new state digest to
  Kaspa and chains to the settlement before it. Chapter 4 is about it.
- **Covenant id**: the 32-byte identity of one program instance: its deposit
  address, its lane, and the exact rule-set version it proves, all
  bundled into one name.
- **Guest**: the program's own code, running inside the proving machine
  (the zkVM), as opposed to the framework around it.
- **Lane**: the program's public inbox: a labeled stream of ordinary Kaspa
  transactions carrying users' signed actions. Miners mine them like any
  other payment; there is no gatekeeper to refuse an entry.
- **Mempool**: the network's shared waiting room: transactions that have
  been announced but not yet included in a block.
- **Journal**: the fixed-format record inside each proof: the state
  before, the state after, how far the lane had been read, and which L1
  blocks the execution saw. Chapter 5 leans on it.
- **Proof, receipt**: a few kilobytes of mathematics that convince anyone,
  without re-running the program, that a claimed execution really happened.
- **Runtime**: the layer of code that checks and applies each action; the
  rules of the house. Chapter 8 is about who owns it.
- **Reorg (reorganization)**: now and then the network briefly agrees on
  one block order, then switches to another; the switched-away blocks
  "vanish". Shallow churn like this is routine and expected; deeply buried
  blocks essentially never reorganize, and "essentially" is why the
  machine also waits out a confirmation window before trusting fresh
  blocks. How deep is deep enough is a deployment choice, and the machine
  widens its window when the network looks reorg-prone. Money that moved
  long ago is as safe as Kaspa itself.
- **Data availability**: the guarantee that you can fetch the full record
  of what was published, yourself, from the network, not just trust
  someone's summary of it.
