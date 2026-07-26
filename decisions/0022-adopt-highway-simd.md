# ADR-0022: Adopt Highway (vendored) as the SIMD layer

## Status
Accepted (2026-07-25, SOW v0.8, CA-9) — conditional: prior-art read (Phase 0) may veto with cause; RVV target maturity is a CHECK item.

## Context and Problem Statement
The v0.5 decision fixed a vector-length-agnostic SIMD core model. Building that layer in-house duplicates Google Highway: battle-tested VLA API, per-target compilation with runtime dispatch, covering exactly our ISA matrix (AVX-512/NEON/SVE; RVV in progress).

## Considered Options
- Build a thin in-house VLA layer (principle 9 purity)
- Adopt Highway, vendored (principle 3 reuse)

## Decision Outcome
Adopt Highway, **vendored** into the tree so consumers see zero added dependencies. Width-specialized escape hatches (audit 13) sit beside it for the enumerated compute-bound kernels (BLIS microkernel pattern).

### Consequences
- Good: years of ISA-portability engineering inherited; the escape-hatch design already assumed fixed-width islands.
- Bad: an external codebase in a core role — pinned, license-audited (Apache-2.0: compatible), tracked by the annual radar.
