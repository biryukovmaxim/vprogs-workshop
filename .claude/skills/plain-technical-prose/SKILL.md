---
name: plain-technical-prose
description: Use when writing or editing prose in the vprogs-workshop book (src/*.md chapters, glossary, README, SUMMARY blurbs) - drafting sections, headings, openings, transitions, summaries, or applying wording review feedback.
---

# Plain technical prose (workshop book)

This book is a technical guide, not a bestseller. Readers extract mechanisms.
Core rule: **name the structure, do not stage a scene.** Every sentence states
a mechanism, a fact, or an honest caveat, in plain technical English.

## The recipe

What the prose is:

- Short declarative sentences, one mechanism each. Split chained clauses.
- Things are named by their technical role: operator, lane, settlement,
  proof, deposit. A new coinage is allowed only when it names structure
  literally: "the shape of Kaspa's history", "the history is a web", "the
  rollup fixes the shape; the program picks the rules". Reject any term that
  imports an image from another domain (vault, weather, landlord, waiting
  room, bedrock, walls, hinges).
- Litmus for any new term: could it appear in a protocol spec without
  quotes? Yes = keep. No = cut.
- Headings name the technical content ("The two limits, and the extensions
  that change them", "Shape versus rules"), never a metaphor.
- Parallel constructions only when both sides are literal: "safety says no
  one can steal; liveness says the machine does not stop" is fine.
  "Liveness is a job" is not.
- End a section on a fact or a caveat, not an aphorism.
- Keep honest caveats, stated plainly ("funds are locked, not lost").
- Demonstration framing: the book shows what is possible on Kaspa. Kaspa
  is the given; never argue "why Kaspa".
- Accuracy over rhetoric: phrase it so the true mechanism is the claim
  (not "no DA layer" when the lane on Kaspa provides data availability).

## Established vocabulary

Reuse; never coin synonyms:

the shape of Kaspa's history, the history is a web, blockdag, shape versus
rules, battery (a shipped replaceable framework part), lane (the program's
public inbox), covenant id, state digest, user action, deposit, exit,
permission tree, witness, journal, guest, image id, reorg, confirmation
window, liveness, data availability, dust.

Full definitions live in `src/words.md`. If a concept needs a new name, add
it there once, then use that one word everywhere.

## Hard rules

- No em-dash or en-dash anywhere in prose. Use commas, colons, semicolons,
  periods, parentheses. (`--` inside mermaid edge labels is diagram syntax,
  not prose.)
- No anthropomorphic framing of systems: state the behavior, not a mood.
- Verify before finishing an edit:
  `grep -rn '—\|–' src/ README.md` returns nothing.

## Real fixes (the pattern)

| Was | Fixed to |
|---|---|
| Heading: "The walls, and the hinges" | "The two limits, and the extensions that change them" |
| "Kaspa is the bedrock; the rollup is the building on top of it" | State the dependency plainly: the rollup settles on Kaspa, and Kaspa holds the funds |
| Heading: "The vault that needs an operator" | Name the topic: "Safety and liveness" |
| "What the chain cannot supply is motion" | "What the chain cannot supply is liveness" |
| "liveness is a job someone must keep doing" | "liveness says the machine does not stop" |

## Red flags (rewrite on sight)

- A metaphor introduced in a heading
- An abstract noun where a glossary term exists
- "not just X, but Y" intensifier pairs
- An aphoristic closing line
- A sentence that would fit on a conference slide
