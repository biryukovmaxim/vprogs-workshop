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
permission to *construct* a transaction; only the L1's rules decide
whether it holds, once proved.

## The read side: indexes

Actions go in; how does your UI know the current board, the user's
balance, the open games? Replaying proofs is not something a frontend
wants to do. So the operator runs a **DA/index layer**: a service that
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
board, post signed actions. All the exotic machinery from chapters 3–6 is
behind two habits: *sign locally, read the index*.

## Who pays for what

Users pay ordinary Kaspa fees for their own lane actions and deposits;
each action rides a normal transaction funded from the user's own UTXOs.
The operator pays the settlement transactions' fees and the proving
compute. tt itself charges nothing inside the program today; an in-program
fee model (debiting accounts to fund the operator) is a battery a real
deployment would place, not something the shape forces or forbids. Which
also answers the spam question, by half: anyone can publish garbage
actions to the lane, but garbage pays its own L1 fees, and the program is
free to reject it at near-zero execution cost. The other half is honest
too: garbage bytes still ride the batch into the proving pipeline, so
volume spam burns operator proving cycles until the fee battery is
placed. Both costs are real; only one is priced today. And the economics
are deliberately unanswered in tt: running the stack is pure cost, which
means the "someone will resume it" liveness story currently rests on
enthusiasm. Pricing actions to fund the operator is exactly what that
battery is for.

## Indexes are convenience, not authority

Here is the part worth internalizing, because it is the difference between
this and trusting a backend. If the index lies to you, shows you a board
that isn't real, a balance that isn't yours, what have you lost? A
moment of confusion. The money is not in the index. The money is in the
proof-chained state on Kaspa, and every exit goes through the permission
tree that the settlement chain itself commits. The index can be wrong,
hostile, or down; it cannot steal. A skeptical app could verify state
digests against L1 and even verify proofs itself; the endpoints are a
performance optimization over truth, not the truth's gatekeeper.

A lying index has one real weapon left, and it isn't theft: it can waste
your time. Your wallet builds actions from what it can see, so garbage
state produces actions that fail at proof time, an annoyance, and an
argument for the skeptical path: everything needed to reconstruct state is
public on L1, the node software is open, and running your own instance
re-executes the same deterministic path over the same public data. A
purpose-built light client (a small program that checks only the pieces
it cares about) doesn't ship today; re-execution does. The data structure
makes the middle path obvious, too: state is a sparse Merkle tree and the
settled digest is public, so the index could hand your wallet a short
inclusion proof for your account, checkable against L1 with no full node
at all. tt doesn't serve one yet; nothing about the shape prevents it.

That inversion, reads untrusted and writes self-certifying, is what makes
the app layer refreshingly boring. Boring is what you want at the top of a
stack like this.
