# ADR-0003: Edge identity: positional by default, stable opt-in

## Status
Accepted (2026-07-25, SOW v0.7)

## Context and Problem Statement
Analytics wants zero-overhead CSR-positional edge indexing; the DB and dynamic layers need stable edge identity for transactions.

## Decision Drivers
- Analytics hot path must pay nothing for generality
- DB transactional semantics require stable IDs

## Considered Options
- Always-stable IDs
- Always-positional
- Positional default + opt-in permutation map

## Decision Outcome
Chosen option: **CSR-positional by default; opt-in stable IDs via permutation map**.

The hot path never pays; the DB opts in.

### Consequences
- Good: both constituencies served
- Bad: two indexing modes to document and test
