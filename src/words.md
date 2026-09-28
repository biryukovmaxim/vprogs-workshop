# The words you need

Blockchain writing leans on a small vocabulary. Here is all of it, in plain
words; the rest of the book assumes these.

- **Kaspa**: the proof-of-work network this book runs on; its coin is
  KAS. What matters here is its blockdag and its UTXO model, both below.
- **Blockdag**: the shape of Kaspa's history. On most chains each block
  names one parent, so the history is a single line; on Kaspa a block may
  name several parents, and the history is a web. Consensus rules order
  that web into one agreed sequence of transactions, roughly one block
  per second. That ordering is why Kaspa is fast, and why shallow reorg
  churn (the Reorg entry below) is normal and expected.
- **Sompi**: the smallest unit of KAS; one KAS is 100,000,000 sompi.
- **DAA score, blue score**: the chain's per-block depth counters
  (difficulty-adjusted and DAG depth); the program reads them as its
  clock (chapter 4).
- **L1**: "layer 1", the Kaspa network itself. The layer that holds the
  funds.
- **L2**: "layer 2": a system that does its work off the L1 while leaning
  on the L1 for what must be trusted: ordering, data availability, and
  settlement. A *rollup* is the common L2 shape: execute off-chain, then
  prove or commit the results back on-chain. The machine in this book is
  one.
- **KIP**: Kaspa Improvement Proposal: the process by which the network
  proposes, reviews, and activates protocol changes. The proposals
  themselves live in the
  [KIP repository](https://github.com/kaspanet/kips).
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
- **State digest**: a short fingerprint of the program's whole off-chain
  state: one 32-byte number that changes whenever the state does.
  Settlements commit it on L1. Chapter 4 builds it.
- **Covenant id**: the 32-byte identity of one program instance: its deposit
  address, its lane, and the exact rule-set version it proves, all
  bundled into one name. The book often says *the covenant* for the
  instance this id names.
- **Guest**: the program's own code, running inside the proving machine
  (the zkVM), as opposed to the framework around it.
- **Image id**: the cryptographic hash of one guest program binary.
  Pinning image ids fixes the exact code and proof stack an instance
  runs (chapters 4 and 7).
- **Lane**: the program's public inbox: a labeled stream of ordinary Kaspa
  transactions carrying users' signed actions. Miners mine them like any
  other payment; there is no gatekeeper to refuse an entry.
- **Mempool**: the set of transactions announced to the network but not
  yet included in a block.
- **Witness**: confirmed L1 data (lane entries, deposits, block context)
  fed to execution; chapter 5 builds the pipeline around it.
- **Journal**: the fixed-format record inside each proof: the state
  before, the state after, how far the lane had been read, and which L1
  blocks the execution saw. Used from chapter 5 on.
- **Proof, receipt**: a few kilobytes of mathematics that convince anyone,
  without re-running the program, that a claimed execution really happened.
- **Runtime**: the layer of code that checks and applies each action.
  Chapter 8 is about who owns it.
- **Reorg (reorganization)**: now and then the network briefly agrees on
  one block order, then switches to another; the switched-away blocks
  "vanish". Shallow churn like this is normal and expected; deeply buried
  blocks essentially never reorganize, which is why the machine waits out
  a confirmation window before trusting fresh blocks (chapter 5).
- **Confirmation window**: the number of blocks of depth the machine
  waits before treating an L1 block as final; widened adaptively when the
  network looks reorg-prone (chapter 5).
- **Liveness**: the guarantee that things keep moving: someone keeps
  executing, proving, and settling. Safety says no one can steal;
  liveness says the machine does not stop. Chapter 6 owns it.
- **Data availability**: the guarantee that you can fetch the full record
  of what was published, yourself, from the network, not just trust
  someone's summary of it.
- **Dust**: outputs too small to be worth spending. The network's
  minimum-relay rules floor how small an output may be, which limits how
  far the deposit pile can be split.
