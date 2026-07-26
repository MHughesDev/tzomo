# ADR-0012: Execution modes: LA spine, BSP sugar, async early

## Status
Accepted (2026-07-25, SOW v0.7)

## Context and Problem Statement
One distributed model does not fit: high-diameter graphs collapse BSP (audit 8: 50%+ wall time in barriers on road networks); LA-parallel has the best scaling theory; vertex-centric is the ergonomic face.

## Decision Drivers
- One partitioner, one transport, multiple programming faces
- Async needed single-node (delta-stepping), not just distributed
- GPU launch overhead is the same problem (persistent kernels)

## Considered Options
- BSP only
- LA only
- Three modes over shared substrate

## Decision Outcome
Chosen option: **LA-parallel spine; vertex-centric BSP compiled onto it; asynchronous/priority-driven mode arriving early as a single-node capability**.

Structure-class detector estimates diameter (double-BFS) and selects policy automatically.

### Consequences
- Good: covers the workload space honestly
- Bad: three modes to test; async is the hard one
