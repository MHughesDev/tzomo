# ADR-0023: GraphView is a concept lattice with a runtime-tagged dispatch boundary

## Status
Accepted (2026-07-26, SOW v0.9)

Deepens ADR-0004 (incidence tiers) and ADR-0002/0003/0005 into an implementable surface.
Full design: `docs/design/04-graphview-and-incidence-tiers.md`.

## Context and Problem Statement
The risk register and ADR-0008 both name GraphView as the abstraction behind which the
chunked-base complexity is "bought once, in Layer 1." Nothing yet specified what GraphView *is*.
Kernels must be written once and run across five representation families (frozen CSR, base+delta
overlay, WebGraph-compressed, mmap out-of-core, distributed shard) at two ID widths, while an
unweighted static simple graph runs BFS exactly as fast as a bespoke engine (SOW §4) under a
bandwidth budget where traversal is latency-bound (audit 1, 11). The runtime representation choice
belongs to the planner (ADR-0020); the instantiation product must stay bounded (§7.13); pairwise
kernels must never touch incidence structures (§7.4).

## Decision Drivers
- Per-access virtual dispatch is unaffordable on the traversal hot path (audit 11).
- The chunked base+delta representation (ADR-0008) splits a vertex's neighbors across ≤2 runs and
  carries tombstones — a single-contiguous-span contract would force a merge-copy per access.
- §7.13 bounds instantiation; ADR-0020 selects representation at runtime — both must hold.
- §7.4 requires that "pairwise never touches incidence" be enforced, not merely intended.

## Considered Options
- **Polymorphic `IGraph` base class** — one runtime type, virtual neighbor access. Simple ABI,
  but a virtual call per neighbor fetch defeats inlining and adds an unpredictable indirect branch
  at the hottest point; violates the zero-cost pairwise requirement.
- **Single-span base contract** (`out_neighbors -> span`) — clean for CSR, but forces base+delta
  into a per-access merge-copy or excludes the live mutable graph from satisfying the concept at
  all (freezing even BFS, which needs neither order nor contiguity).
- **Concept lattice with runs-as-base and a runtime-tagged dispatch boundary** — chosen.

## Decision Outcome
Chosen option: **GraphView is a C++23 concept, not a base class.** Kernels are
`template<GraphView G>`, monomorphized per representation × ID-width; neighbor access is a direct
inlined load with no per-access dispatch. Representation capabilities that differ — CSC dual,
weights, single-sorted-run contiguity, adjacency probe, dedup, stable edge IDs, time — are
expressed as a **lattice of concept refinements**, and each kernel constrains on the weakest node
it needs. The **base neighbor contract is a range of ≤2 sorted runs, not a single span**; single
contiguous span is the `ContiguousGraphView` refinement, satisfied by frozen representations and
reached from a live graph only via an explicit `freeze()` — so the mutable representation's cost
is paid at a named call by exactly the kernels (intersection, probe, SIMD) that require it, and
never by BFS/PageRank. The runtime representation choice is **one type-erased dispatch at the call
boundary** (`AnyGraph`), monomorphic within — reconciling concepts with ADR-0020 and §7.13.
Pairwise (`GraphView`) and incidence (`IncidenceView`/`SimplicialView`) are **disjoint** concepts
joined only by an explicit O(m) `project_to_pairwise` conversion, making "pairwise never touches
incidence" a compile error. The simplicial tier exposes the boundary operator ∂_k as a sparse
matrix `blas`/`laplace` consume directly; the Hodge Laplacian is derived, never stored. The C ABI
reflects the compile-time lattice as a runtime **capability bitmask** on the opaque graph handle,
with unmet requirements returned as actionable named statuses (`TZM_ERR_NEEDS_CONTIGUOUS`,
`TZM_ERR_NEEDS_CSC`).

### Consequences
- Good: zero per-access dispatch; the mutable-representation cost is routed by the type system to
  only the callers who incur it; the incidence firewall is compile-enforced; the refinement
  lattice *shrinks* the instantiation product (kernels instantiate only for representations that
  satisfy their requirement) and simultaneously serves as the planner's plan-space generator
  (ADR-0020); freeze / CSC-build / window-materialization boundaries become visible in the type
  system rather than implicit.
- Bad: base-concept kernels iterate a range of runs, not a raw pointer (one-element and
  compiler-collapsed on the frozen fast path, two-iteration on base+delta); the C++ concept lattice
  must be mirrored as an ABI capability bitmask kept in sync; more concepts to document and test
  than a single `IGraph` would need.
- Follow-ups ratified in the design note §9: neighbor-run cardinality bounded at ≤2 (delta compacts
  rather than stacking runs); `SimpleGraphView` is a compile-time tag with a debug-only validation
  pass; `project_to_pairwise` ships clique and star expansion in v1; window materialization is a
  first-class `TemporalGraphView → ContiguousGraphView` transition the planner owns.
