# Building an app on it

Everything so far has been the machine. This chapter is the view from the
other side: you want to build an app on a vprogs rollup. What do you get?

The pattern is one sentence: **signed actions in, indexed state out.**

## The write side

Your app's users hold keys. Everything a user does is a signed action,
and building one is pure, local computation: take the action, encode it
with the program's wire format, sign it with the user's key. tt compiles
its [guest wire library to WebAssembly](https://github.com/biryukovmaxim/vprog-tictactoe/blob/fe6b0e8/encoder-wasm/src/lib.rs) so the browser can do exactly this:
the same encoders the zkVM verifies, running in the page, producing signed
actions with zero servers involved. The wallet never asks anyone's
permission to *construct* a transaction; the L1 carries it, and whether
it holds is decided at proof time by the program's own rules.

## The read side: indexes

Actions go in; how does your UI read the current board, the user's
balance, the open games? Replaying proofs is too heavy for a frontend.
So the operator runs a **DA/index layer**: a service that
follows the program's execution and serves queries over the resulting
state. [tt's node](https://github.com/biryukovmaxim/vprog-tictactoe/blob/fe6b0e8/node/src/da.rs) exposes exactly the API a tic-tac-toe app needs:

| Endpoint | What the app reads |
|---|---|
| `GET /api/state` | program state summary (current digest, settlement info) |
| `GET /api/config` | program config (stakes, turn timer, withdrawal minimum) |
| `GET /api/games?status=…` | game list, filtered and paginated |
| `GET /api/games/:id` | one game: board, players, stake, status |
| `GET /api/accounts/:id` | one account: balance, lock |
| `GET /api/exits` | exit entitlements waiting to be claimed on L1 |

The web app is then a perfectly ordinary frontend: fetch state, render a
board, post signed actions. There is no Anchor-style IDL or generated
client yet: the WASM wire library is the client SDK, and the endpoints
above are hand-written. All the exotic machinery from [Based rollup on Kaspa](based-rollup.md) through [The machinery](machinery.md) is
behind two habits: *sign locally, read the index*.

## Who pays for what

Users pay ordinary Kaspa fees for their own lane actions and deposits;
each action rides a normal transaction funded from the user's own UTXOs,
and an exit claim is likewise an ordinary transaction, built by
whoever claims (the user, a subsidized operator, a claims service),
its fee carried by one of the builder's own coins (the [settlement appendix](appendix-settlements.md#an-exit-claim-up-close) has
the shape).
The operator pays the settlement transactions' fees and the proving
compute. tt itself charges nothing inside the program today; an in-program
fee model (debiting accounts to fund the operator) is a battery a real
deployment would place, not something the shape forces or forbids.

Spam has a partial answer: anyone can publish garbage actions to the
lane, but garbage pays its own L1 fees, and the program rejects it at
near-zero execution cost. The rest of the answer is cost: garbage bytes
still ride the batch into the proving pipeline, so volume spam burns
operator proving cycles until the fee battery is placed.

There is no pre-proof filter, deliberately: whoever filters decides what
counts as garbage, and the lane's promise is that inclusion is not
anyone's decision. Skipping an entry cannot hide; it shows up as a
stalled lane tip, [The machinery](machinery.md)'s stall, visible rather than silent. Running
the stack is pure cost in tt, so the "someone will resume it" story
depends on enthusiasm until that battery is placed.

## Indexes are convenience, not authority

This section is the difference between an index and a trusted backend.
The money is not in the index. The money is in the proof-chained state
on Kaspa, and every exit goes through the permission tree that the
settlement chain itself commits. The index can be wrong, hostile, or
down; it cannot move funds itself. What it can do is induce you to
sign: your wallet builds actions from what it can see, so a lying read
costs you exactly what the resulting action can lose under the
program's rules. For tt that bound is tight: a fake board produces a
move that fails at proof time, or at worst loses one game's stake. For
an app whose actions trade on state, a swap against reserves the index
misreported, the induced action is valid, executes against the true
state, and the loss is yours. The defense is two-sided. The program's
half: actions should carry their own bounds, a swap its minimum
received, every action its deadline in DAA score, so an action built
on bad data fails at proof time instead of executing a loss. The
reader's half: treat the index as a claim and keep a way to check it,
which is the ladder below. The endpoints are a performance
optimization, not the source of truth.

## How far can you verify a read?

First, what an account is: a resource id plus its state bytes. The
whole state is one sparse Merkle tree, the id is the key, the leaf
commits the id and the hash of the bytes, and the settled state digest
on L1 is the root.
Every read verification walks the same chain of anchors, and how far
you can walk it depends on what you run:

| What you run or trust | What you verify | Against what |
|---|---|---|
| Nothing (the operator's index, the default app path) | Display only; the board it shows is a claim | Nothing today; the next settlement publishes the digest, but checking an account against it still needs the path that does not ship (below) |
| An L1 node, your own or an explorer's | The digest chain: each settlement names the digest it moved to, and Kaspa consensus checked the proof binding it | L1 itself |
| Your own L2 node in execution mode (replay without proving) | The full state: it replays the public lane and takes nothing from the operator | L1, re-executed |

An indexer is a view over one of these nodes; it inherits whatever that
node is worth trusting. Your own indexer against the operator's node
buys nicer queries, not independence. One more check the shape allows:
the settlement's receipt is public data on L1 and verifies in
milliseconds against the pinned image ids ([The zkVM, briefly](zkvm.md)); consensus runs
that check on every settlement, so running it yourself matters only if
you do not process the chain yourself.

In the game's terms: the index shows your opponent's move as a new
board. Checking it means hashing that board, checking the hash is the
leaf at the game's resource id, and checking the resulting root against
the settled digest. That middle step is the one the shape allows but no
API serves today: tt hands out account bytes, not Merkle paths, while
the proving host (the prover's side outside the zkVM) loads exactly
those paths for every account a batch
touches. The [state tree appendix](appendix-state-tree.md) has the
walk, and where the bytes behind the leaves live. A purpose-built light
client (a small program that verifies only the data it needs) doesn't
ship today either; re-execution does.

One gap no endpoint closes, only time or re-execution: state between
settlements. The last settlement pins a digest; everything after it, up
to the lane tip the operator claims, is unproved. Either wait for the
next settlement, whose proof must explain the gap, or replay the suffix
yourself.

That inversion, reads untrusted and writes self-certifying, is what
keeps the app layer simple.
