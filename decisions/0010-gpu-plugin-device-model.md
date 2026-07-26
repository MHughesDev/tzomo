# ADR-0010: GPU as dynamically-loaded plugin over an abstract device model

## Status
Accepted (2026-07-25, SOW v0.7)

## Context and Problem Statement
Zero-dependency core (principle 9) conflicts with linking CUDA; ten-year life spans vendor shifts and coherent-memory hardware.

## Decision Drivers
- CPU-only build must stay dependency-free
- HIP/SYCL/Metal must remain implementable
- Grace-Hopper/MI300A-class coherent memory changes algorithm choice (audit 21)

## Considered Options
- Link CUDA in core
- Compile-time GPU variant
- Dynamic plugin over abstract device model

## Decision Outcome
Chosen option: **Plugin interface defined against an abstract device model (memory spaces, streams, launch, collectives) representing shared/coherent memory; CUDA is the first implementation, not the definition**.

Resolves the principle conflict and future-proofs vendor and memory-model shifts.

Note (CA-16, v0.8): the device model's collectives vocabulary mirrors NCCL (all-reduce, all-gather, reduce-scatter, broadcast) so the CUDA plugin maps cleanly and alternatives inherit a known shape.

### Consequences
- Good: dependency-free core; vendor-neutral
- Bad: an abstraction layer to maintain; affects build system from day one
