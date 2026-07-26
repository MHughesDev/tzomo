# ADR-0013: Instantiation policy: structural templates, value-type erasure

## Status
Accepted (2026-07-25, SOW v0.7)

## Context and Problem Statement
Naive template product (representation x widths x property types x kernels) yields thousands of instantiations: multi-hundred-MB binaries, hour compiles, icache thrash.

## Decision Drivers
- Audit 25: in bandwidth-bound kernels type erasure costs nothing measurable
- Compute-bound set is small and enumerated (audit 11)

## Considered Options
- Full specialization
- Full erasure
- Structural compile-time + value-type erasure with enumerated specializations

## Decision Outcome
Chosen option: **Representation family and ID/offset width are compile-time; value types type-erased by default, specialized only in intersection/decode/dense/ingest/LOBPCG kernels**.

Collapses to ~(5 representations x 2 widths) structural variants plus a handful of typed hot kernels.

### Consequences
- Good: tens-of-MB binary, sane compiles
- Bad: the erased/specialized boundary must be policed in review
