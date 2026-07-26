# Design Deepening — §4 Data Model: the GraphView surface and incidence tiers

**Status:** proposed for v0.9. Deepens SOW §4 toward implementable specificity.
**Ratifies into:** ADR-0023 (the new decision), a §4 amendment, and a v0.9 change-log entry.
**Governs / is governed by:** ADR-0002 (IDs), ADR-0003 (edge identity), ADR-0004
(incidence tiers), ADR-0005 (properties), ADR-0007 (epochs), ADR-0008 (chunked base),
ADR-0020 (planner), ADR-0021 (composability), SOW §7.13 (instantiation policy),
§7.11 (small-graph budget).

This document takes the one abstraction the risk register and ADR-0008 both name as
load-bearing — *"complexity bought once, in Layer 1, behind the GraphView concept"* —
and makes it concrete enough to build: what the concept is, what it requires, how
representations satisfy it, how kernels program against it, and where the sharp edges are.

The single new architectural commitment is stated up front, then justified, pressure-tested,
and specified. Everything downstream (kernels, the planner's plan-space, the C ABI's graph
handle, the freeze/compaction boundary) is a consequence of getting this surface right.

---

## 0. The decision in one paragraph

**GraphView is a C++23 *concept*, not an abstract base class.** Kernels are templates
constrained on it, monomorphized per representation × ID-width, so neighbor access is a direct
load with no per-access virtual dispatch — the only way to honor "an unweighted static simple
graph runs BFS exactly as fast as a bespoke engine" (§4) under the bandwidth budget (audit 1).
Representation *capabilities* that differ — single-span vs. multi-run neighbor storage, presence
of a CSC dual, random adjacency probe, stable edge IDs, weights, time — are expressed as a
**lattice of concept refinements**, and each kernel states the *exact* refinement it needs. The
runtime choice of representation (ADR-0020's planner) is resolved by **one type-erased dispatch
at the call boundary** (`AnyGraph`), monomorphic within. Pairwise and incidence (hypergraph /
simplicial) are **disjoint** concepts joined only by an explicit `project_to_pairwise`
conversion — which is how "pairwise kernels never touch incidence structures" (§7.4) becomes a
*compile error*, not a code-review convention.

---

## 1. Why a concept and not an `IGraph` base class

The tempting design — a polymorphic `IGraph` with `virtual span<VId> neighbors(VId) const` —
is what most graph libraries do and it is wrong here for one measurable reason:

- Traversal is latency-bound (audit 11). BFS over a billion-edge graph issues ~m neighbor-list
  fetches. A virtual call per fetch adds an indirect branch the predictor cannot cover across
  representations, plus it defeats inlining of the inner loop into the frontier scan. On a
  power-law graph the per-fetch cost is already ~one cache miss; doubling the *fixed* overhead
  per vertex is exactly the "2× overhead is a bug" (principle 1) failure at the hottest point.

A concept moves the dispatch from *per-access* to *per-call*:

- **Inner loop:** `template<GraphView G> void bfs(const G&, ...)` is compiled once per concrete
  `G`. Neighbor access is a direct member call the compiler inlines and vectorizes. Zero
  representation-dispatch cost inside the loop.
- **Boundary:** the planner/ABI hold an `AnyGraph` — a tagged union over the ~5 representation
  families × 2 widths (§7.13). A kernel invoked on an `AnyGraph` does **one** switch on the tag
  and calls the matching monomorphic instantiation. One predictable branch per *call*, well
  inside the §7.11 2–5 µs budget (which is per-call, not per-edge).

This is OpenBLAS's runtime-microarch-dispatch pattern (ADR-0020 cites it) applied at
representation granularity, and it is the only structure consistent with **both** §7.13
(structural parameters are compile-time templates; the instantiation product is bounded and
enumerable) **and** ADR-0020 (the planner selects representation at runtime).

**Consequence for instantiation count (checks out against §7.13):** a kernel instantiates only
for the representations that satisfy its required refinement. Intersection-heavy kernels
(triangle counting) require `ContiguousGraphView` and instantiate for ~2 representations, not
all 5. The lattice *shrinks* the instantiation product rather than growing it — a point in favor
against the §7.13 binary-size budget, not a cost against it.

---

## 2. The concept lattice

Concepts are refinements: an arrow `A → B` means "B refines A" (every `B` is an `A`, plus more).
Kernels constrain on the *weakest* concept they can. Representations satisfy the *strongest* they
can. The planner's legal moves for a kernel are exactly the representations at or above the
kernel's required node.

```
                         GraphView                      (pairwise base: sizes, degree,
                             │                           neighbor iteration as runs)
        ┌────────────┬───────┼────────────┬─────────────────┐
        ▼            ▼       ▼            ▼                 ▼
 Bidirectional   Weighted  Simple    Contiguous        Temporal
   GraphView     GraphView GraphView  GraphView         GraphView
 (CSC dual:     (aligned  (dedup    (single sorted     (time column
  in_neighbors) weight    guarantee) run per vertex)    per edge)
        │        spans)       │            │                 │
        └──────────────┬──────┴─────┬──────┘                 │
                       ▼            ▼                         │
                  Indexed      StableEdge                     │
                  GraphView    GraphView                      │
               (O(1)/O(log d)  (positional→stable            │
                adjacency      permutation map,               │
                probe)         ADR-0003 opt-in)               │
                                                              ▼
                                                    (temporal refinements
                                                     compose orthogonally
                                                     with all of the above)

  ── disjoint hierarchy, joined only by explicit conversion ──

     IncidenceView  ──project_to_pairwise (O(m), clique/star)──▶  GraphView
         │
         ▼
     SimplicialView   (graded by dimension k; oriented; exposes ∂_k as a sparse matrix)
```

### 2.1 `GraphView` — the pairwise base

The minimal surface every pairwise kernel can assume. Associated types carry the compile-time
identity widths (ADR-0002); the two widths are **independent** (de Bruijn: 64-bit `vertex_id`,
32-bit `degree_type`).

```cpp
template<class G>
concept GraphView = requires(const G g, typename G::vertex_id u) {
    typename G::vertex_id;      // dense internal id, 0..n-1  (ADR-0002, width is compile-time)
    typename G::degree_type;    // per-vertex degree / offset width, INDEPENDENT of vertex_id
    typename G::size_type;      // edge count / global offset width

    { g.num_vertices() } -> std::same_as<typename G::vertex_id>;
    { g.num_edges()    } -> std::same_as<typename G::size_type>;
    { g.out_degree(u)  } -> std::same_as<typename G::degree_type>;

    // Neighbor iteration is a RANGE OF SORTED RUNS, not a single span. For the frozen CSR
    // fast path this range has exactly one element (a span) and degenerates to zero cost.
    // For the base+delta overlay (ADR-0008) it has up to two (base run + delta run), merged
    // lazily by the consumer. This is the load-bearing choice — see §3.
    { g.out_neighbors(u) } -> neighbor_runs_of<typename G::vertex_id>;
};
```

`neighbor_runs_of<VId>` is a range whose elements are `std::span<const VId>`, each individually
sorted ascending, tombstone-filtered. The base guarantee is **not** global sortedness across runs
and **not** contiguity — those are refinements (§2.4). Every run is a non-owning view into
representation-owned memory (capsule ownership, §7.11); the base concept never copies.

Kernels that only read degrees and stream neighbors — BFS, PageRank (push and pull without CSC),
connected components, k-core, degree-sequence work — need nothing beyond this node, and therefore
run **directly on a live, mutating graph** without a freeze.

### 2.2 `BidirectionalGraphView` — the CSC dual

```cpp
template<class G>
concept BidirectionalGraphView = GraphView<G> && requires(const G g, typename G::vertex_id u) {
    { g.in_degree(u)    } -> std::same_as<typename G::degree_type>;
    { g.in_neighbors(u) } -> neighbor_runs_of<typename G::vertex_id>;
};
```

Required by **direction-optimizing BFS** (the pull step scans in-neighbors of the unvisited) and
by **pull-mode PageRank** (audit 6's default contention answer). An undirected graph satisfies it
trivially (in == out, one storage). A directed graph satisfies it only if the CSC was built
(O(m)); if not, a kernel requiring it either triggers the build or the ABI returns
`TZM_ERR_NEEDS_CSC` (§7 error mapping). The compile-time constraint means a bidirectional kernel
*cannot even be instantiated* on an out-only representation — the missing capability is caught at
build time, not at 3 a.m. on a road network.

### 2.3 `WeightedGraphView` — weights as an aligned column, on the hot path

Weight is the one property that lives *with* topology, because SpMV, delta-stepping, and
shortest paths touch it in the inner loop. It is a compile-time-typed `weight_type` (one of the
enumerated specialized hot kernels, §7.13), exposed as a span aligned index-for-index with the
neighbor run:

```cpp
template<class G>
concept WeightedGraphView = GraphView<G> && requires(const G g, typename G::vertex_id u) {
    typename G::weight_type;                       // compile-time; SpMV specializes on it
    { g.out_weights(u) } -> weight_runs_of<typename G::weight_type>;   // aligned with out_neighbors(u)
};
```

All **other** properties (Arrow columns, §7.5) are *not* on the view. They are passed to kernels
as separate type-erased `PropertyView` handles (runtime pointer + stride + Arrow type tag — free
in bandwidth-bound kernels, audit 25). The split is deliberate and is the concrete meaning of
"columnar, external to topology" (ADR-0005): **weight is topological-hot and compile-time-typed;
everything else is columnar-cold and type-erased.** State this in the ABI so binding authors know
which surface a datum lives on.

### 2.4 `ContiguousGraphView` — the single-sorted-run refinement (the freeze boundary)

```cpp
template<class G>
concept ContiguousGraphView = GraphView<G> && requires(const G g, typename G::vertex_id u) {
    // Exactly one sorted, tombstone-free span per vertex. The refinement that makes galloping
    // intersection, binary-search probe, and SIMD decode legal.
    { g.out_neighbor_span(u) } -> std::same_as<std::span<const typename G::vertex_id>>;
};
```

This is where the chunked base+delta architecture (ADR-0008) becomes *visible in the type
system*. A frozen CSR, a WebGraph-decoded buffer, and an mmap'd frozen chunk all satisfy it. A
**live base+delta graph does not** — its neighbors are split across ≤2 runs and may carry
tombstones. A kernel that needs contiguity (triangle counting, subgraph iso's candidate
intersection, any SIMD set operation) is `template<ContiguousGraphView G>`; to run it on a live
graph you must first `freeze()` → which returns an immutable `Csr` view over the current manifest,
materializing hot chunks and compacting away tombstones. **The cost of the mutable representation
is thus paid explicitly, at a named call, by exactly the kernels that cannot tolerate it** — and
never by BFS/PageRank, which stay on the base concept. This is the single most important
consequence of choosing "runs" (not "span") as the base contract; §3 defends it.

### 2.5 `IndexedGraphView`, `SimpleGraphView`, `StableEdgeGraphView`, `TemporalGraphView`

- **`IndexedGraphView`** (refines `ContiguousGraphView`): adds `has_edge(u,v)` and
  `edge_position(u,v)` in O(log d) (binary search on the sorted span) or O(1) (auxiliary hash for
  dense-block regions). Required by subgraph isomorphism's probe phase and by any join-shaped
  kernel. Intersection-based triangle counting needs only `ContiguousGraphView` (galloping merge),
  not `IndexedGraphView` — keep the distinction so the planner doesn't over-constrain.
- **`SimpleGraphView`** (refines `GraphView`): asserts the no-multi-edge / no-self-loop dedup
  constraint (§4 constrained views). A tag-only refinement — no new members, just a compile-time
  promise kernels can rely on (triangle counting's combinatorics assume simple; the shim can
  request it). Multigraphs do **not** satisfy it; a kernel needing simple either demands it or
  runs a dedup pass.
- **`StableEdgeGraphView`** (refines `GraphView`): `edge_id(u, local_pos) -> stable_edge_id` via
  the opt-in permutation map (ADR-0003). Analytics never requires it; the DB and dynamic layer do.
  Positional edge identity is implicit in the base concept (offset + local index); stability is
  the paid-for refinement.
- **`TemporalGraphView`** (refines `GraphView`): `edge_time(u, local_pos)` and time-filtered
  neighbor runs. Composes orthogonally with the others (a graph can be Bidirectional + Weighted +
  Temporal). Time-respecting traversal and temporal centrality constrain on it. Window
  materialization (SOW §8) produces a plain `ContiguousGraphView` from a `TemporalGraphView` when
  a window will see 2–3+ passes.

### 2.6 `IncidenceView` and `SimplicialView` — the higher-order tiers

Disjoint from `GraphView`. This disjointness is the enforcement mechanism for §7.4.

```cpp
template<class H>
concept IncidenceView = requires(const H h, typename H::vertex_id v, typename H::hedge_id e) {
    typename H::vertex_id;
    typename H::hedge_id;
    { h.num_vertices()   } -> std::same_as<typename H::vertex_id>;
    { h.num_hyperedges() } -> std::same_as<typename H::hedge_id>;
    { h.vertices_of(e)   } -> std::same_as<std::span<const typename H::vertex_id>>;  // incidence
    { h.hyperedges_of(v) } -> std::same_as<std::span<const typename H::hedge_id>>;   // dual incidence
    { h.is_ordered()     } -> std::same_as<bool>;   // sequence vs set (§4 "optional ordering")
    { h.is_oriented()    } -> std::same_as<bool>;   // signed incidence available (§4 "orientation")
};

template<class S>
concept SimplicialView = IncidenceView<S> && requires(const S s, int k) {
    { s.max_dimension()  } -> std::same_as<int>;
    { s.num_simplices(k) } -> std::same_as<typename S::size_type>;
    // Boundary operator ∂_k as a sparse matrix: rows = (k-1)-simplices, cols = k-simplices,
    // entries in {-1,+1} by orientation. Returned as a SpView the `blas`/`laplace` modules
    // consume directly — the Hodge Laplacian L_k = ∂_k^T ∂_k + ∂_{k+1} ∂_{k+1}^T is DERIVED,
    // never stored. This is the concrete meaning of "runs through the same sparse-LA machinery."
    { s.boundary(k) } -> SparseMatrixView;
};
```

**The enforcement, made concrete:** a pairwise kernel is `template<GraphView G>`. A `Hypergraph`
satisfies `IncidenceView`, not `GraphView`. Calling `bfs(hypergraph)` is a **compile error** whose
`static_assert` message names `project_to_pairwise`. To traverse a hypergraph pairwise you write
`bfs(project_to_pairwise(h, ExpansionPolicy::Clique))` — an explicit, O(m), documented conversion.
There is no path by which incidence storage silently enters a pairwise inner loop. A genuine
pairwise graph is *never* stored as incidence (§7.4: pairwise is the CSR family); a 2-uniform
hypergraph is a degenerate encoding, and requiring the explicit projection for it is correct, not
pedantic.

---

## 3. Defending "runs, not span" as the base contract

This is the subtle, load-bearing call, so it gets its own section.

The naive base contract is `out_neighbors(u) -> span<const VId>` — one contiguous span. It is
what a CSR-only design would pick. It is wrong as the *base* because it forces the chunked
base+delta representation (ADR-0008 — load-bearing for five subsystems) into one of two bad
outcomes on every neighbor access:

1. **Merge-copy per access:** synthesize a single span by copying base ∪ delta into scratch. That
   is an allocation-and-copy on the hottest path of a mutating graph — the exact write-side of the
   §7.8 write-amplification problem, re-introduced on the read side.
2. **Break the concept:** have the mutable graph *not* satisfy `GraphView`, forcing a freeze
   before *any* kernel, including BFS/PageRank that don't need sortedness or contiguity at all.

"Runs" as the base, "single span" as a refinement, avoids both. BFS/PageRank/CC/k-core consume
the ≤2-run form directly at zero copy (they don't care about order). Only the kernels that truly
require a single sorted run (intersection, probe, SIMD) demand `ContiguousGraphView`, and they pay
the freeze once, explicitly, visibly. The type system routes the cost to precisely the callers who
incur it and shields everyone else. This is "boring interfaces, radical internals" (principle 10)
and "reuse through layering" (principle 3) doing real work.

**Cost of the choice, stated honestly:** kernels written against the base concept must iterate a
range of runs, not a raw pointer. For the frozen fast path this is a one-element range the compiler
collapses (verified as a zero-cost acceptance gate, §6). For the ≤2-run path it is a two-iteration
outer loop. A kernel author who wants raw-pointer simplicity opts into `ContiguousGraphView` and
accepts the freeze precondition. Nobody is forced up the lattice; nobody is forced to pay.

---

## 4. Identity, epochs, and what never crosses the surface

- **Internal IDs only.** No external ID, string, or dictionary key ever crosses the GraphView
  surface (§7.2). Translation is the `dict` component's job, at the ABI boundary. Invariant
  enforced by construction: the concept's associated types are integer widths; there is no string
  type in the surface.
- **A view is over a frozen epoch** (ADR-0007). `GraphView` accessors are all `const`. Mutation
  goes through the builder or the delta-overlay API, never the view. A snapshot is a manifest; a
  `Csr` view is a manifest materialized/pinned. Holding a view pins the epoch cheaply.
- **Single address space.** `GraphView` is inherently one node / one shard. A distributed graph is
  a *collection* of local `GraphView`s plus a transport (§7.1), orchestrated by `dist` — never a
  `GraphView` subtype. Partition-local 32-bit IDs (ADR-0002) live inside each shard's view. This
  keeps "embedded-core, orchestrated scale-out" true at the type level: distribution *composes*
  views, it does not *specialize* one.

---

## 5. Mapping the C++ concept lattice onto the C ABI

The ABI cannot express concepts, so the compile-time lattice is reflected at runtime as a
**capability bitmask** on the opaque graph handle:

```c
typedef struct tzm_graph tzm_graph;                 /* opaque; wraps an AnyGraph               */

typedef uint32_t tzm_caps;                          /* reflection of the concept lattice        */
#define TZM_CAP_BIDIRECTIONAL  (1u << 0)            /* has CSC dual                             */
#define TZM_CAP_WEIGHTED       (1u << 1)
#define TZM_CAP_CONTIGUOUS     (1u << 2)            /* single sorted run per vertex (frozen)    */
#define TZM_CAP_INDEXED        (1u << 3)
#define TZM_CAP_SIMPLE         (1u << 4)
#define TZM_CAP_STABLE_EDGE    (1u << 5)
#define TZM_CAP_TEMPORAL       (1u << 6)

tzm_caps   tzm_graph_caps(const tzm_graph* g);
tzm_status tzm_graph_freeze(tzm_graph* g, tzm_graph** out_frozen);   /* → gains CONTIGUOUS       */
```

A kernel entry point checks the mask against its requirement and, if unmet, returns an
**actionable** status per the shared error taxonomy (§14.11) — e.g. `TZM_ERR_NEEDS_CONTIGUOUS`
("this kernel requires a frozen graph; call `tzm_graph_freeze`") or `TZM_ERR_NEEDS_CSC`. The
mask is the C-visible shadow of the compile-time lattice: C++ callers get the error at build time,
C callers get it as a named status with the remedy in the message. Binding authors (§6.6) bind
against the mask + status contract, which is stable ABI.

---

## 6. Acceptance criteria (CI-enforceable)

1. **Zero-cost pairwise gate.** BFS and PageRank on `Csr<u32,u32>` *through* the base `GraphView`
   concept match a hand-written bespoke CSR loop within measurement noise (median ± IQR, §14.1).
   Regression blocks merge (principle 4). This is the operational definition of "pays *nothing*
   for the platform's generality" (§4).
2. **No per-access dispatch.** A perf-counter check (indirect-branch rate inside the traversal
   loop ≈ 0) proves the concept monomorphized and did not degrade to virtual dispatch. Wired into
   the §14.2 HITM/frontend-stall CI harness.
3. **Single-branch call boundary.** The `AnyGraph` dispatch adds ≤1 predictable branch per call;
   the 500-node-in-a-loop benchmark (§7.11) stays inside the 2–5 µs budget with the dispatch in
   place.
4. **Compile-time incidence firewall.** A negative compile test: instantiating a pairwise kernel
   on an `IncidenceView` must fail to compile with the `project_to_pairwise` diagnostic. Run as a
   "must-not-compile" CI case.
5. **Freeze-boundary correctness.** A `ContiguousGraphView` kernel on a base+delta graph without a
   preceding freeze returns `TZM_ERR_NEEDS_CONTIGUOUS` (C ABI) / fails to compile (C++), never
   silently reads an unsorted or tombstoned span.
6. **Instantiation-count budget.** The extern-template unit count stays within the §7.13 envelope
   (~5 representations × 2 widths structural, plus enumerated typed hot kernels); a CI check
   asserts the lattice did not multiply instantiations beyond the budget.

---

## 7. Edge-case register

| # | Case | Resolution |
|---|------|-----------|
| E1 | Degree-0 vertex | Empty run range, never null. Kernels handle empty spans naturally. |
| E2 | Self-loop | Present in the neighbor run; each kernel documents inclusion (triangle counting excludes; PageRank includes). |
| E3 | Multi-edge | Duplicate entries in the run; `SimpleGraphView` kernels require prior dedup. |
| E4 | Hub vertex (skew) | Edge-based work splitting via CSR-offset binary search (audit 7) operates on the run(s); works identically through the view. |
| E5 | Base-empty, delta-nonempty vertex | Run range yields only the delta run. |
| E6 | Tombstoned edges in delta | Base concept filters tombstones during run iteration (a per-access cost); `ContiguousGraphView` is available only post-compaction, where tombstones are gone. |
| E7 | Out-of-core mmap fault | Still `ContiguousGraphView` (single span); access is disk-bound. The *planner* prices the latency; the *concept* is unchanged. Clean separation of correctness from cost. |
| E8 | Directed graph, no CSC built | Does not satisfy `BidirectionalGraphView`. Kernel requiring it either triggers an O(m) CSC build or returns `TZM_ERR_NEEDS_CSC`. |
| E9 | 2-uniform hypergraph | Satisfies `IncidenceView`, not `GraphView`. Pairwise use requires explicit `project_to_pairwise`. Correct, not pedantic — real pairwise graphs are never stored as incidence. |
| E10 | Empty graph (n=0) | All accessors valid; `num_vertices()==0`; kernels no-op. A property-test seed. |

---

## 8. How this feeds the planner (ADR-0020)

The refinement lattice **is** the planner's legal-move generator. For a given kernel, the planner
enumerates only representations that satisfy the kernel's required concept node, then measures
among them (FFTW-style) and persists the winner in wisdom. Concretely:

- Triangle counting requires `ContiguousGraphView` → the planner considers {frozen CSR, WebGraph-
  compressed-decode, mmap-frozen}, and *never* proposes running it on a live delta overlay
  (which would be a type error anyway). The plan-space is pre-pruned by the type system.
- BFS requires only `GraphView` → the planner may keep it on the live graph, avoiding a freeze
  the kernel doesn't need.
- Direction-optimizing BFS requires `BidirectionalGraphView` → the planner's move set includes
  "build CSC then run" with the O(m) build priced into the plan.

This is a genuine unification, not a coincidence: the same lattice that gives kernels their
compile-time guarantees gives the planner its runtime search space. Worth stating as an explicit
consequence in ADR-0020's eventual Phase-3 detail ADR.

---

## 9. Open sub-questions (recommendations, for ratification)

1. **Neighbor-run cardinality bound.** Recommend fixing the base+delta run count at **≤2** (one
   base, one delta) and forbidding a live graph from accumulating N delta runs — the delta overlay
   compacts into the base rather than growing a run stack, so the outer loop in base-concept
   kernels is a fixed 2-iteration, never data-dependent. *Ratify: yes* — keeps the base concept's
   iteration cost O(1)-bounded and predictable.
2. **Should `SimpleGraphView` be a tag or carry a runtime-checked invariant?** Recommend
   **compile-time tag only** (zero-cost promise), with an optional debug-build `assert` pass that
   validates the dedup constraint (principle 5: debug validates everything, release checks at
   boundaries). *Ratify: yes.*
3. **`project_to_pairwise` expansion policies.** Recommend shipping **clique** and **star**
   expansion in v1 (the two standard hypergraph→graph reductions), each documented with its
   edge-count blow-up, and leaving weighted/lossy expansions to later. *Ratify: yes.*
4. **Where does window materialization live in the lattice?** Recommend `TemporalGraphView →
   (materialize window) → ContiguousGraphView` as a first-class transition the planner owns,
   parallel to `freeze()`. *Ratify: yes.*

---

## 10. Proposed amendments (the delta)

- **New ADR-0023** — "GraphView is a concept lattice with a runtime-tagged dispatch boundary" —
  recording the §0 decision, the concept-vs-vtable choice, the runs-not-span base contract, and
  the incidence-disjointness firewall.
- **SOW §4 amendment** — add a short "GraphView surface" subsection pointing at this design note
  and ADR-0023, stating the base/refinement split and the compile-time incidence firewall as
  ratified data-model properties.
- **v0.9 change-log entry** in §18.
- **Downstream note** for the eventual ADR-0020 Phase-3 detail: the lattice is the planner's
  plan-space generator (§8 above).

No frozen decision is moved; no §6 exclusion is touched. This deepens §4/§7.4/§7.13 and is
additive.
