# ADR-0001: Embedded core, orchestrated scale-out

## Status
Accepted (2026-07-25, SOW v0.7)

## Context and Problem Statement
One architecture must serve laptop embedding and data-center clusters without making single-node the degenerate case.

## Decision Drivers
- Adoption physics: ~99% of users are single-node forever
- Solo testability for years before distribution exists
- Principles 1 (extreme scale) and 9 (embeddable) must not conflict

## Considered Options
- Embedded core + orchestration layer running one engine per node/GPU (DuckDB/CombBLAS/NCCL pattern)
- Distributed-native with single-node as special case (Hadoop/Spark pattern)

## Decision Outcome
Chosen option: **Embedded core + orchestration layer**.

Winners in adjacent spaces all chose it; Spark-shape is why Spark loses on one machine. Scale-up lives in the core, scale-out in orchestration.

### Consequences
- Good: adoption, testability, principle reconciliation
- Bad: kernel interfaces must be partition-aware-capable from day one even though v1 runs whole graphs
