# Building an app on it

Everything so far has been the machine. This chapter is the view from the
other side: you want to build an app on a vprogs rollup. What do you get?

The pattern is one sentence: **signed actions in, indexed state out.**

## The write side

Your app's users hold keys. Everything a user does is a signed action,
and building one is pure, local computation: take the action, encode it
with the program's wire format, sign it with the user's key. tt compiles
its guest wire library to WebAssembly so the browser can do exactly this:
the same encoders the zkVM verifies, running in the page, producing signed
actions with zero servers involved. The wallet never asks anyone's
permission to *construct* a transaction; the L1 carries it, and whether
it holds is decided at proof time by the program's own rules.

## The read side: indexes

Actions go in; how does your UI read the current board, the user's
balance, the open games? Replaying proofs is too heavy for a frontend.
So the operator runs a **DA/index layer**: a service that
follows the program's execution and serves queries over the resulting
state. tt's node exposes exactly the API a tic-tac-toe app needs:

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
above are hand-written. All the exotic machinery from chapters 3 to 6 is
behind two habits: *sign locally, read the index*.

## Who pays for what

Users pay ordinary Kaspa fees for their own lane actions and deposits;
each action rides a normal transaction funded from the user's own UTXOs,
and an exit claim is likewise the claimant's own transaction, its fee
carried by one of the claimer's own coins (the appendix has the shape).
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
stalled lane tip, chapter 6's stall, visible rather than silent. Running
the stack is pure cost in tt, so the "someone will resume it" story
depends on enthusiasm until that battery is placed.

## Indexes are convenience, not authority

This section is the difference between an index and a trusted backend.
If the index lies to you, shows you a board
that isn't real, a balance that isn't yours, what have you lost? A
moment of confusion. The money is not in the index. The money is in the
proof-chained state on Kaspa, and every exit goes through the permission
tree that the settlement chain itself commits. The index can be wrong,
hostile, or down; it cannot steal. A skeptical app could verify state
digests against L1 and even verify proofs itself; the endpoints are a
performance optimization, not the source of truth.

A lying index cannot steal; it can only waste your time.
Your wallet builds actions from what it can see, so garbage
state produces actions that fail at proof time, an annoyance, and an
argument for the skeptical path: everything needed to reconstruct state is
public on L1, the node software is open, and running your own instance
re-executes the same deterministic path over the same public data. A
purpose-built light client (a small program that verifies only the data
it needs) doesn't ship today; re-execution does. The data structure
makes the middle path obvious, too: state is a sparse Merkle tree and the
settled digest is public, so the index could hand your wallet a short
inclusion proof for your account, checkable against L1 with no full node
at all. tt doesn't serve one yet; nothing about the shape prevents it.

That inversion, reads untrusted and writes self-certifying, is what keeps
the app layer simple.
