# The words you need

Blockchain writing uses a small vocabulary. Here is all of it, in plain
words; the rest of the book assumes these. Entries marked * simplify
where the full rule would weigh more than the book needs; each links the
appendix that carries the complete version.

- **Kaspa**: the proof-of-work network this book runs on; its coin is
  KAS. What matters here is its blockdag and its UTXO model, both below.
- **Blockdag***: the shape of Kaspa's history: a web, not a line. Where
  a classic chain discards parallel blocks, a Kaspa block names several
  parents, so blocks mined at nearly the same time all count, and
  consensus orders the web into one agreed order of transactions,
  roughly one block per second. The machine consumes that order; the
  shallow reorg churn the web leaves behind is why the confirmation
  window exists (the Reorg entry below). The
  [appendix](appendix-kaspa-depth.md) has the ordering in full.
- **Sompi**: the smallest unit of KAS; one KAS is 100,000,000 sompi.
- <a id="daa-score"></a>**DAA score***: a depth counter every block carries, counting blocks
  in its past. It only moves forward, so the program reads it as its
  clock: deadlines are differences in DAA score, not wall-clock time
  ([The transaction vocabulary](transactions.md)). Kaspa also paces mining difficulty and emission by it.
  Its sibling, the **blue score**, is the counter Kaspa counts
  confirmations in; the program sees it only as block context
  ([The transaction vocabulary](transactions.md)). The [appendix](appendix-kaspa-depth.md) separates the
  two exactly.
- **L1**: "layer 1", the Kaspa network itself. The layer that holds the
  funds.
- **L2**: "layer 2": a system that does its work off the L1 while depending
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
  on-chain state is hard here ([Based rollup on Kaspa](based-rollup.md)).
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
- **Settlement**: the transaction that commits one new state digest of
  the program's off-chain (L2) state to Kaspa, spends the output only
  a valid settlement can spend, and chains to the settlement before
  it. [Chapter 4](transactions.md) is about it.
- **State digest**: a short fingerprint of the program's whole off-chain
  state: one 32-byte number that changes whenever the state does.
  Settlements commit it on L1. [Chapter 4](transactions.md) builds it.
- <a id="covenant-id"></a>**Covenant id**: the 32-byte identity of one program instance: its deposit
  address, its lane, and the exact rule-set version it proves, all
  bundled into one name. The book often says *the covenant* for the
  instance this id names.
- **Guest**: the program's own code, running inside the proving machine
  (the zkVM), as opposed to the framework around it.
- **Image id**: the cryptographic hash of one guest program binary.
  Pinning image ids fixes the exact code and proof stack an instance
  runs (chapters [4](transactions.md) and [7](zkvm.md)).
- **Lane**: the program's public inbox: a labeled stream of ordinary Kaspa
  transactions carrying users' signed actions. Miners mine them like any
  other payment; there is no gatekeeper to refuse an entry.
- **Mempool**: the set of transactions announced to the network but not
  yet included in a block.
- **Execution**: the guest program doing its work: reading confirmed
  L1 data, checking and applying each action, crediting deposits, and
  moving the state from one root to the next. It runs off-chain inside
  the zkVM; the proof is what makes its result trustworthy ([How it all chains](chaining.md)).
- **Witness**: the confirmed L1 data execution reads: lane entries,
  deposits, block context. [Chapter 5](chaining.md) builds the pipeline around it.
- **Journal**: the fixed-format record inside each proof: the state
  before, the state after, how far the lane had been read, which L1
  blocks execution saw, and the deposit and exit commitments where the
  step carried any. Used from [How it all chains](chaining.md) on.
- **Proof, receipt**: a few kilobytes of mathematics that convince anyone,
  without re-running the program, that a claimed execution really happened.
- **Runtime**: the layer of code that checks and applies each action.
  [Chapter 8](solana.md) is about who owns it.
- **Reorg (reorganization)**: now and then the network briefly agrees on
  one block order, then switches to another; the switched-away blocks
  "vanish". Shallow churn like this is normal and expected; deeply buried
  blocks essentially never reorganize, which is why the machine waits out
  a confirmation window before trusting fresh blocks ([How it all chains](chaining.md)).
- **Confirmation window**: the number of blocks of depth the machine
  waits before treating an L1 block as final; widened adaptively when the
  network looks reorg-prone ([How it all chains](chaining.md)).
- <a id="liveness"></a>**Liveness**: the guarantee that someone keeps
  executing, proving, and settling, so actions and exits keep processing.
  Safety says no one can steal;
  liveness says the machine does not stop. [Chapter 6](machinery.md) owns it.
- <a id="data-availability"></a>**Data availability**: the guarantee that you can fetch the full record
  of what was published, yourself, from the network, rather than
  trusting someone's summary of it.
- **Mass**: the weight of a transaction that fees are priced on: the
  larger of compute mass (verification work) and storage mass (the
  unspent-set growth the transaction leaves behind; it rises when value
  is split into many small outputs). The
  [appendix](appendix-kaspa-depth.md) has the formula and what nodes
  store.
- **Dust**: outputs too small to be worth spending. The network's
  minimum-relay rules floor how small an output may be, which limits how
  far the deposit pile can be split.
