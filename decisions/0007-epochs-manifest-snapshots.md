# ADR-0007: Mutability epochs; snapshots are chunk manifests

## Status
Accepted (2026-07-25, SOW v0.7)

## Context and Problem Statement
Kernels need immutability; mutation and MVCC need cheap snapshots.

## Decision Drivers
- No locks in the analytics core
- DB snapshot semantics

## Considered Options
- Locking concurrent structure
- Building/frozen epochs with manifest snapshots

## Decision Outcome
Chosen option: **Explicit building/frozen epochs; an epoch/snapshot is a chunk manifest (see ADR-0008)**.

MVCC-shaped by construction; the DB inherits snapshots for free.

### Consequences
- Good: lock-free analytics
- Bad: no concurrent-mutation-during-analytics promise in v1 (stream module's later problem)
