# Building an app on it

Everything so far has been the machine. This chapter is the view from the
other side: you want to build an app on a vprogs rollup. What do you get?

The pattern is one sentence: **signed actions in, indexed state out.**

## The write side

Your app's users hold keys. Everything a user does is a signed action —
and building one is pure, local computation: take the action, encode it
with the program's wire format, sign it with the user's key. tt compiles
its guest wire library to WebAssembly so the browser can do exactly this:
the same encoders the zkVM verifies, running in the page, producing signed
actions with zero servers involved. The wallet never asks anyone's
permission to *construct* a transaction — only the L1's rules decide
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
| `GET /api/config` | program config (stakes, timers, fee parameters) |
| `GET /api/games?status=…` | game list, filtered and paginated |
| `GET /api/games/:id` | one game — board, players, stake, status |
| `GET /api/accounts/:id` | one account — balance, lock |
| `GET /api/exits` | exit entitlements waiting to be claimed on L1 |

The web app is then a perfectly ordinary frontend: fetch state, render a
board, post signed actions. All the exotic machinery from chapters 3–6 is
behind two habits — *sign locally, read the index*.

## Indexes are convenience, not authority

Here is the part worth internalizing, because it is the difference between
this and trusting a backend. If the index lies to you — shows you a board
that isn't real, a balance that isn't yours — what have you lost? A
moment of confusion. The money is not in the index. The money is in the
proof-chained state on Kaspa, and every exit goes through the permission
tree that the settlement chain itself commits. The index can be wrong,
hostile, or down; it cannot steal. A skeptical app could verify state
digests against L1 and even verify proofs itself — the endpoints are a
performance optimization over truth, not the truth's gatekeeper.

That inversion — reads untrusted, writes self-certifying — is what makes
the app layer refreshingly boring. Boring is what you want at the top of a
stack like this.
