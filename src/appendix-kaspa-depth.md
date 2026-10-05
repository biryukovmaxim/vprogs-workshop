# Appendix: Kaspa details behind the simplifications

The glossary simplifies three Kaspa mechanisms: how the blockdag's web
becomes one order, what the two depth counters count, and what storage
mass charges for. This page carries the full versions. Nothing here
changes the design; the machine reads the same L1 either way.

## From web to one order

Every Kaspa block names one *selected parent*, the block it builds on,
plus any further parents it merges. The consensus rule (GHOSTDAG) sorts
each block's mergeset into *blue* blocks, the ones well connected to
the selected-parent chain, and *red* ones, which stay in history but
order outside the blue set. The split orders blocks; it drops no
transactions: a red block's transactions ride the selected-chain block
that absorbs it, as the [settlement appendix](appendix-settlements.md#what-counts-as-one-block)'s rule says. Walking
selected parents from a tip back to
genesis gives the *selected chain*, the line the rest of the book
executes along. A block on that line absorbs its mergeset's
transactions into the same execution step; the
[settlement appendix](appendix-settlements.md) spells out what counts
as one block.

## Blue score: how deep is this block

A block's blue score is its selected parent's blue score plus the blue
blocks in its mergeset. It is the blockdag's version of block height,
and it is the counter Kaspa counts confirmations in: depth is measured
as blue-score distance from the including block toward the current tip.
Blue score only moves forward; a reorg replaces a stretch of the order,
it does not rewind the counter.

## DAA score: how much work is in the past

A block's DAA score (difficulty adjustment algorithm) is its selected
parent's DAA score plus the mergeset blocks that are not far behind
the chain: blue blocks and recent red ones. Deeply red blocks, mined
well behind the selected chain, are excluded, so a flood of badly
connected blocks cannot inflate the counter. The network reads mining
difficulty from a window of blocks chosen by DAA score, and the
emission schedule, including the reward halvings, is defined over DAA
score. Both counters ride every block header, so both only move
forward; that is what lets the program treat DAA score as a clock
([chapter 4](transactions.md)). The per-block context the program reads inside each proof
window, timestamp, DAA score, blue score, is committed by the chain
itself ([KIP-21](https://github.com/kaspanet/kips/blob/master/kip-0021.md)).

## Storage mass: paying for the unspent set

A full node keeps three things. Headers and the blockdag structure
between them: forever. The unspent transaction output set, one entry
per output created and not yet spent: forever, it is the state every
node must consult to validate. Full block data, transactions with
their bodies: only for a bounded recent window on ordinary (pruned)
nodes; archival nodes keep everything.

That asymmetry is why outputs cost. A transaction's mass, the weight
its fee is priced on, is the larger of two terms
([KIP-9](https://github.com/kaspanet/kips/blob/master/kip-0009.md)):
compute mass, covering verification work, and storage mass, covering
the unspent-set growth the transaction leaves behind. Storage mass
rises when a transaction splits value into many small outputs and is
offset when it consolidates small inputs: dust is expensive to create
and cheap to sweep. In the relaxed form, the storage term is a
constant times the positive part of the sum of reciprocals of output
values minus the sum of reciprocals of input values; the constant and
the integer rounding rules live in the KIP. The rule exists because a
dust attack once used
Kaspa's throughput to grow every full node's unspent set permanently;
minimum-relay rules (the glossary's dust entry) floor how small an
output may be, and storage mass prices the splitting itself. Both
limits meet in this book's deposit pile: an exit claim that splits the
pile can divide it only as far as relay rules and storage mass allow.
