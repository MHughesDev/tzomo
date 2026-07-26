# ADR-0004: Higher-order model: incidence foundation, three arity tiers

## Status
Accepted (2026-07-25, SOW v0.7)

## Context and Problem Statement
Hypergraphs (EDA netlists) and simplicial complexes (FEM/TDA) are in scope, but 99% of usage is pairwise and must pay nothing.

## Decision Drivers
- Native hyperedges validated by EDA domain
- Hodge Laplacians run through the same sparse-LA machinery
- Pairwise speed is non-negotiable

## Considered Options
- Encodings only (bipartite expansion)
- Native-everything single structure
- Incidence foundation with pairwise fast path

## Decision Outcome
Chosen option: **Incidence-structure foundation (vertices, hyperedges-as-sets, boundary operators); pairwise multigraph as specialized storage/kernels; simplicial = spectral in scope, persistence excluded**.

Generality at the model level, zero cost at the pairwise level; pairwise kernels never touch incidence structures.

### Consequences
- Good: world-first production ambition with protected common case
- Bad: thin prior art for the native tiers (risk register)
