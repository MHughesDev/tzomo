# ADR-0011: Small-graph fast path with a stated binding budget

## Status
Accepted (2026-07-25, SOW v0.7)

## Context and Problem Statement
Most NetworkX users have small graphs; a billion-edge architecture loses to igraph-class per-call overhead at 500 nodes, and the Phase-1 launch dies on the first small-graph benchmark posted by a skeptic (Domain Test 4).

## Decision Drivers
- <=2-5us total per-call overhead (the igraph/graph-tool class)
- Zero-copy results; workspace reuse (audit 5)
- NetworkX shim must not convert per call (audit 24)

## Considered Options
- Optimize later
- Architectural fast path with CI-enforced budget

## Decision Outcome
Chosen option: **Single-threaded allocation-light path with parallelism thresholds from Phase 0; zero-copy NumPy views; shim conversion-caching (convert once, version-stamp invalidation); CI benchmark that is literally a 500-node call in a loop**.

The budget is architecture, not a tuning pass; NetworkX's official backend machinery (can_run/should_run, convert-and-cache) carries half the design.

### Consequences
- Good: closes the launch-killing gap GPU backends structurally cannot
- Bad: two execution paths to keep correct
