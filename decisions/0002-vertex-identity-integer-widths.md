# ADR-0002: Vertex identity and integer widths

## Status
Accepted (2026-07-25, SOW v0.7)

## Context and Problem Statement
Kernels need dense integer IDs; graphs range from thousands to tens of billions of vertices; bandwidth-bound kernels gain ~1.5-2x from halving ID width.

## Decision Drivers
- Bandwidth is the scarce resource (audit 1)
- De Bruijn graphs force 64-bit IDs with tiny degrees (Domain Test 2)
- Distribution needs shard-local compactness

## Considered Options
- Runtime-only width (friendly, slow)
- Compile-time only 64-bit (wasteful)
- Compile-time parameter, both shipped, runtime-selected

## Decision Outcome
Chosen option: **Compile-time template parameter, both variants shipped, runtime-selected at graph creation**.

Zero hot-path cost, bindings users never think about it. Vertex-ID width and degree/offset width are independent parameters. Distributed mode uses partition-local 32-bit IDs regardless of global size.

### Consequences
- Good: cheapest performance win in the design
- Bad: instantiation surface doubled (bounded by ADR-0013)
