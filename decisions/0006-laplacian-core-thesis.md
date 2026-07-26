# ADR-0006: Laplacian solvers as core thesis

## Status
Accepted (2026-07-25, SOW v0.7)

## Context and Problem Statement
Near-linear Laplacian solving is the largest theory-practice gap in the field; no production C++ implementation exists (research Julia code does).

## Decision Drivers
- Feeds partitioner, layout, sparsification, max-flow, FEM domains simultaneously
- Identity claim no benchmark can match
- Research-grade risk

## Considered Options
- Phase-3 curiosity
- Core Layer-2 deliverable with risk split

## Decision Outcome
Chosen option: **Ship in Layer 2 alongside the eigensolver; eigensolver is the committed deliverable, Laplacian solver the stretch flagship**.

Most cross-cutting component in the platform; risk contained by the commit/stretch split.

### Consequences
- Good: deepest moat candidate
- Bad: real failure probability, accepted
