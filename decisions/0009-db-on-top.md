# ADR-0009: DB-on-top, never DB-inside

## Status
Accepted (2026-07-25, SOW v0.7)

## Context and Problem Statement
The analytics core must stay a pure compute engine; the DB needs durability and transactions.

## Decision Drivers
- Zero transaction awareness in kernels
- Epochs are already MVCC-shaped

## Considered Options
- Integrated storage engine
- DB layered over analytics core

## Decision Outcome
Chosen option: **DB module owns WAL/durability and hands frozen manifest snapshots to the analytics core**.

Transactions produce epochs; analytics consumes epochs; neither knows the other's internals.

### Consequences
- Good: core stays embeddable and lock-free
- Bad: DB write path must round-trip through the delta/compaction machinery
