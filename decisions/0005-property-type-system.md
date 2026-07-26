# ADR-0005: Property/type system: columnar, Arrow-typed, schema-optional

## Status
Accepted (2026-07-25, SOW v0.7)

## Context and Problem Statement
Properties must serve schemaless exploration, schema-enforced production/DB, and compile-time-bound C++ kernels without three systems.

## Decision Drivers
- Zero-copy Python/data-ecosystem interop
- Do not invent a type system
- Kernel-speed property access for C++ users

## Considered Options
- Interleaved with topology
- Custom type system
- Columnar external Arrow-typed, three binding levels

## Decision Outcome
Chosen option: **Columnar, external to topology, Arrow-typed; three levels (schemaless / schema-declared / compile-time-bound); vertex-edge types as dense categorical columns**.

Arrow is the lingua franca (independently corroborated by competitors converging on it).

### Consequences
- Good: interop for free; C++26 reflection can later delete binding boilerplate
- Bad: three access paths to keep coherent
