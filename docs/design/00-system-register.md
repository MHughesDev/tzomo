# System Definition Register

**Status:** living map, opened at v0.9. The completeness instrument for "is every system
clearly defined?"

This register enumerates **every system in the platform** — the §10 module catalog *plus* the
cross-cutting systems that are real but live inside modules (the scheduler, the SIMD layer, the
on-disk format, the structure-class detector, …). For each it records responsibility, phase,
dependencies, the sources that currently define it, and — the point of the document — a **definition
status**: how far the system is from an implementable surface.

It is deliberately *not* a place to re-decide anything. It measures the definition frontier and
orders the deepening work. Deepening a system means producing a `docs/design/NN-*.md` note (the
GraphView note, `04-graphview-and-incidence-tiers.md`, is the template) and ratifying the delta
via change log + ADR.

## Definition-status rubric

| Level | Meaning | Test |
|-------|---------|------|
| **S — Specified** | Implementable surface exists: interfaces, invariants, acceptance criteria, edge cases. | A competent engineer could start building from the doc without re-deciding architecture. |
| **D — Decided** | Architecture frozen by an ADR or a binding SOW clause; no interface-level spec yet. | The *what* and *why* are settled; the *how* (types, contracts, tests) is not written. |
| **K — Sketched** | Scope described in prose; no ADR-level decision and no interface. | You know what it's for; key design calls remain open. |
| **N — Named** | Appears in the catalog / scope list only. | A name and a one-line responsibility, nothing more. |

**Honest headline (v0.9):** of ~35 systems, **2 are Specified** (GraphView/§4 → C1.0; on-disk
format/§14.5 → X.2), ~15 Decided, ~11 Sketched, ~7 Named. The platform is *architecturally* dense
(24 ADRs, zero open items) and *implementationally* thin — exactly right for a project at day-1 of
Phase 0. The register's job is to convert that from an impression into a tracked frontier.

---

## Layer 0 — Systems foundations (module: `runtime`, `core`)

| # | System | Resp. | Phase | Depends on | Defined by | Status | Gap to close |
|---|--------|-------|-------|-----------|-----------|:------:|--------------|
| L0.1 | **Memory-tier abstraction** | Arena/slab allocators over enumerable tiers (HBM/DRAM/CXL/NVMe) with per-structure placement hints; NUMA as 2-tier case | 0 | — | §5 L0 | **K** | Tier enumeration API; placement-hint interface; allocator concept; NUMA-node binding contract |
| L0.2 | **Parallel runtime + heterogeneity-aware scheduler** | Work-stealing pool; per-core capability metadata; hub-task placement; hierarchical NUMA barriers | 0 | L0.1 | §5 L0; ADR-0021 (composability) | **K** | Scheduler handle interface (the thing ADR-0021 says is honored everywhere); task-graph API; skew/steal policy; barrier contract |
| L0.3 | **Portable SIMD layer** | Length-agnostic core (SVE/RVV model), fixed-width specializations, width-specialized escape hatches | 0 | — | ADR-0022 (Highway) | **D** | Which kernels get escape hatches (enumerated set exists); the length-agnostic wrapper surface; determinism interaction (§14.6) |
| L0.4 | **Signature primitive: binned/blocked passes** | Seven jobs (privatization, cache blocking, false-sharing elim, permutation apply, radix sort, scatter-to-sequential, wire batching) | 0 | L0.2 | §5 L0/L1; audit series | **K** | The reusable pass abstraction — one API the seven jobs share; block-size hook to the planner (ADR-0020) |
| L0.5 | **Signature primitive: freeze-time vertex reordering** | Six jobs (locality, hub id, skew split, branch predictability, compression, ordering–encoding co-select); on-by-default | 0–1 | L0.4, C1.x | §5 L1; §7.2 | **K** | Pluggable-ordering interface; the co-selection contract with representations; acceptance (must be free of user-visible consequence) |
| L0.6 | **Prefetch + contention toolkit** | Batched software prefetch; privatized accumulators; hub-aware striping; pull-mode default | 0 | L0.2 | §5 L0; audit 6 | **K** | The contention-answer API every parallel kernel states (§14.11 checklist) |
| L0.7 | **Workspace objects** | Preallocated cache-aligned scratch, surfaced through the C ABI; zero-alloc small-graph repeat calls | 0 | L0.1 | §5 L0; ADR-0011; audit 5 | **D** | Workspace lifecycle/ownership API; how kernels declare their scratch shape |
| L0.8 | **Compressed-integer primitives** | Roaring, Elias-Fano, delta encoding | 0 | — | §5 L0 | **N** | Which structures use which; the codec interface |
| L0.9 | **Communication abstraction** | no-op → NCCL/MPI/RDMA backends; interface early, backends late | 0 (iface) / 5 (impl) | L0.2 | §5 L0; ADR-0010 note (CA-16) | **K** | The transport concept (collectives, point-to-point); the no-op default; how `dist` and `gpu` plug in |
| L0.10 | **io_uring out-of-core I/O** | O_DIRECT double-buffered chunk pipeline; mmap-over-frozen-chunks read mode | 0 | L0.1, C1.2 | §5 L1; ADR-0008; CA-10 | **D** | The chunk-pipeline API; fault-handling contract; ties to the on-disk format (X.2) |

---

## Layer 1 — Representations (module: `core`)

| # | System | Resp. | Phase | Depends on | Defined by | Status | Gap to close |
|---|--------|-------|-------|-----------|-----------|:------:|--------------|
| C1.0 | **GraphView surface + incidence tiers** | The concept lattice all kernels program against; pairwise/incidence firewall | 0 | L0.* | **ADR-0023; design/04** | **S** | ✅ Specified. Follow-ups: ratify the four sub-questions in design/04 §9 |
| C1.1 | **Immutable CSR/CSC** | Dual out/in; the pairwise fast path; `ContiguousGraphView` | 0 | C1.0 | §5 L1; ADR-0002/0003 | **D** | Builder API; CSC-build trigger; freeze() semantics (named in C1.0, unspecified) |
| C1.2 | **Chunked base + delta overlay** | Vertex/time-partitioned immutable chunks under a manifest; has-delta bitmap; per-chunk compaction | 0–1 | C1.1, X.2 | **ADR-0007, ADR-0008**; §7.8 | **D** | Manifest schema; compaction strategy interface (CA-4); tombstone merge; the ≤2-run guarantee (design/04 §9.1) — **strong candidate for next deepening** |
| C1.3 | **Compressed (WebGraph-style)** | 8–16× byte reduction; decode-on-traverse; preferred on compressible classes | 0–3 | C1.1, L0.8 | §5 L1 | **K** | Codec choice; the decode→`ContiguousGraphView` path; when the planner prefers it |
| C1.4 | **Temporal representation** | Time-columned edges; window materialization; `TemporalGraphView` | 0–3 | C1.1 | §5 L1; §8 | **K** | Time-column layout; window-materialization transition (design/04 §9.4); time-respecting traversal contract |
| C1.5 | **Conversion layer** | Explicit O(m) conversion between all representations; permutations via binned scatters | 0 | C1.1–C1.4, L0.4 | §5 L1 | **K** | The conversion matrix (which pairs, which cost); the `project_to_pairwise` expansions (design/04 §9.3) |
| C1.6 | **Property / type system** | Columnar, Arrow-typed, three binding levels; type columns dispatched on | 0 | C1.0 | **ADR-0005** | **D** | The three access paths (schemaless/declared/compile-bound); PropertyView type-erased surface (named in design/04 §2.3) |

---

## Layer 2 — Compute kernels

| # | System (module) | Resp. | Phase | Depends on | Defined by | Status | Gap to close |
|---|-----------------|-------|-------|-----------|-----------|:------:|--------------|
| C2.1 | **`blas`** (GraphBLAS-style) | SpMV, SpGEMM, semirings, masking | 0–3 | C1.*, L0.3 | §5 L2; CA-8 standalone | **N** | Semiring interface; masking model; the standalone link-test contract |
| C2.2 | **`eigen`** (sparse eigensolvers) | Lanczos, LOBPCG, Jacobi-Davidson; ARPACK/Spectra replacement | 1 | C2.1 | §5 L2; §7.6 (committed deliverable) | **N** | Solver API; convergence/restart policy; determinism mode (principle 8 non-negotiable here); oracle design |
| C2.3 | **`laplace`** (Laplacian solvers + sparsification) | Near-linear solve; spectral sparsification; the cross-cutting flagship | 3 | C2.1, C2.2 | **ADR-0006**; §7.6 | **D** | **Algorithm selection (KOSZ vs approximate-Cholesky) + oracle** — a named candidate deepening; stretch-vs-committed line already drawn |
| C2.4 | **Construction/ingest kernels** (`build`) | Parallel sort, hash, dedup, external-memory build; first-class benchmarked work | 0 | L0.4, C1.1 | §5 L2; Domain Test 2 | **K** | The ingest pipeline stages; external-memory build contract; throughput acceptance rows (§14.1) |
| C2.5 | **Frontier/traversal engine** | Direction-optimizing BFS substrate | 0 | C1.0 (Bidirectional), L0.2 | §5 L2 | **K** | The push/pull switch policy; frontier data structure; async-mode hook (ADR-0012) |

---

## Layer 3 — Algorithms (module: `algo`, `community`)

| # | System | Resp. | Phase | Depends on | Defined by | Status | Gap to close |
|---|--------|-------|-------|-----------|-----------|:------:|--------------|
| C3.1 | **`algo` battery** | Traversals, centrality, components, shortest paths (delta-stepping), max-flow/min-cut, k-core, triangle/motif, matching, subgraph iso | 1 | C2.*, C1.0 | §5 L3 | **N** | Per-algorithm concept requirement (which lattice node — the register in design/04 §2 makes this mechanical); incremental variants; oracle per kernel |
| C3.2 | **`community`** | Leiden, spectral, multilevel | 3 | C2.3, C3.1 | §5 L3 | **N** | Algorithm set; quality metric; determinism |
| C3.3 | **`detect`** (structure-class detector) | Detects sparse/dense, power-law, small-world/high-diameter, planar, DAG/tree, community, bounded-width; drives dispatch; diameter via double-BFS | 0–1 | C2.5 | §4; §7.12; **§10 (v0.9)** | **K** | Now catalogued as `detect` (G1 resolved v0.9). Interface still to specify: what it measures, what dispatches on it, its tie to the planner (X.1) |

---

## Layer 4 — Applied engines

| # | System (module) | Resp. | Phase | Depends on | Defined by | Status | Gap to close |
|---|-----------------|-------|-------|-----------|-----------|:------:|--------------|
| C4.1 | **`part`** (graph + hypergraph partitioning) | Mt-KaHyPar baseline; in-house spectral/incremental/layout-integrated layer; determinant of distributed viability | 2 | C2.3, C1.0 (Incidence) | **ADR-0015**; §17.9 | **D** | The differentiating-layer interface; quality target wiring (comm-volume vs random); incremental repartition API |
| C4.2 | **`layout`** | Graphviz replacement: multilevel force-directed, spectral drawing, stress majorization, edge bundling, incremental relayout, dot-compatible CLI | 2 | C2.3, C4.1 | §5 L4 | **K** | Algorithm set + the dot-compat common-path scope; incremental-relayout contract |
| C4.3 | **`viz`** (GPU renderer) | Millions of elements, LOD, picking/interaction APIs | 2 | C4.2, C6.2 | §5 L4 | **N** | Rendering pipeline; interaction API surface; the §6.1 GUI boundary line |
| C4.4 | **`stream`** | Incremental analytics; change/anomaly detection; retention/window/eviction lifecycle | 3 | C1.2, C1.4 | §5 L4; §7.8; open-item 7 | **D** | Retention-policy objects (time-TTL/count/predicate); incremental-algorithm state model; the deleting-compaction path |
| C4.5 | **`match`** (entity resolution) | Matching / alignment toolkit | 3+ | C3.1 | §5 L4 | **N** | Scope; blocking/scoring model |
| C4.6 | **`ml`** (graph-ML plumbing) | Samplers, minibatching, embeddings export (DLPack/PyTorch/ONNX), GPU-resident structures, subgraph tokenization | 3+ | C1.0, C6.2 | §5 L4 (plumbing only) | **K** | Sampler interfaces; export format contracts; the plumbing/model boundary |
| C4.7 | **`dict`** (external-ID) | Two-phase: sharded mutable hash → frozen minimal-perfect-hash; Arrow dictionary arrays | 0–1 | L0.1 | §5 L4; audit 17 | **D** | The two-phase API; freeze transition; memory-vs-throughput acceptance |
| C4.8 | **`db`** (embedded graph DB) | Persistence, WAL, snapshots, transactions, openCypher-surface/GQL-core query layer | 4 | C1.2, C4.x | **ADR-0009, ADR-0016** | **D** | *Owned by a separate design track.* Here only as a consumer of epochs/manifests — the consumer contract is the part to keep coherent |

---

## Layer 5 — Scale-out (module: `dist`, `gpu`)

| # | System | Resp. | Phase | Depends on | Defined by | Status | Gap to close |
|---|--------|-------|-------|-----------|-----------|:------:|--------------|
| C5.1 | **`dist`** | One partitioner + one transport; three execution modes (LA-parallel spine, BSP sugar, async); gang-scheduled fail-stop deterministically restartable | 5 | C4.1, L0.9 | **ADR-0001, ADR-0012** | **D** | The three-face API over the substrate; restart/checkpoint hook; capacity-scaling acceptance framing (§14.1) |
| C5.2 | **`gpu`** (plugin) | Dynamically-loaded; abstract device model (memory spaces, streams, launch, collectives); coherent-memory-capable; CUDA first impl | 5 (design from day 0) | L0.9 | **ADR-0010** | **D** | The device-model interface; resident-first doctrine; transfer-inclusive acceptance (§14.1) |

---

## Layer 6 — Interfaces (module: `capi`, `python`, `wasm`)

| # | System | Resp. | Phase | Depends on | Defined by | Status | Gap to close |
|---|--------|-------|-------|-----------|-----------|:------:|--------------|
| C6.1 | **`capi`** (stable C ABI) | Handle-based, status codes, versioned option structs, three API tiers, capability bitmask, composability contract | 1 | all core | ADR-0011, ADR-0021; §14.11; design/04 §5 | **D** | The error taxonomy (§14.11); the option-struct versioning rule; the capability-bitmask enum (partially drafted in design/04 §5) |
| C6.2 | **`python`** (+ NetworkX shim) | nanobind/pybind11; GIL-free from day one; NetworkX dispatch backend; conversion-caching shim | 1 | C6.1 | ADR-0011; §7.11; §14.13; §11 | **D** | The shim conversion-cache design (invalidate-by-version-stamp); the FT-marking; the differences list |
| C6.3 | **`wasm`** | WASM builds → supported compute target (WASM64 + threads) by Phase 3 | 1–3 | C6.1 | §5 L6; §6.7 | **K** | The threading model under WASM; which kernels are in the WASM tier |
| C6.4 | **Format I/O** | GraphML, GML, edge lists, Parquet, mmap-native; hostile-input fuzzed | 0 | C1.1, X.2 | §5 L6; §14.7 | **K** | Parser threat model; the mmap-native reader (= the on-disk format, X.2) |

---

## Cross-cutting systems (not single modules)

| # | System | Resp. | Phase | Defined by | Status | Gap to close |
|---|--------|-------|-------|-----------|:------:|--------------|
| X.1 | **Planner + wisdom** | Measures representation/ordering/kernel/block choices; persisted wisdom cache; the plan-space is the concept lattice (design/04 §8) | 3 (auto) / 0 (iface) | **ADR-0020**; §8 | **D** | The planner interface + plan-space definition — a named candidate deepening; the wisdom cache format + invalidation |
| X.2 | **On-disk format** | mmap-native format: container of immutable 2MB-aligned `.tzc` chunks + atomically-swapped manifest; frozen `ContiguousGraphView` form only; read-N−1; Arrow-IPC properties | 0 | **ADR-0024; design/14**; §14.5; ADR-0008 | **S** | ✅ Specified (spec skeleton). Follow-ups: higher-order (`chunk_type` 1/2) section catalogs; the `dict.tzd` and provenance-sidecar sub-specs |
| X.3 | **Error taxonomy** | One error model across C++/C ABI/Python; actionable messages | 1 | §14.11 | **K** | The enum; the "what was wrong + what would be valid" convention; the capability-unmet statuses (design/04 §5) |
| X.4 | **Determinism infrastructure** | Bit-exact + fast modes, per-kernel selected; cross-arch reduction policy | 0 | Principle 8; §14.6 | **K** | The mode-selection API; the reproducible-reduction primitives; how the planner interacts with fixed-vs-planned execution |
| X.5 | **Benchmark harness** (`bench`) | GAP/LDBC at 10³/10⁶/10⁹ + construction; regression CI; bytes-per-edge & %-STREAM; small-graph & write-amp rows; published losses | 0 | §14.1 | **D** | The harness architecture; the metric-reporting schema; the reproduction-script generator |
| X.6 | **Correctness/oracle harness** | Per-kernel brute-force oracles; property-based tests; differential vs NetworkX/igraph/graph-tool; synthetic generators; fuzzing | 0 | Principle 7; §14.2 | **K** | The oracle interface every kernel implements; the generator suite; the fuzzing targets |

---

## Gaps and recommendations

**G1 — RESOLVED (v0.9).** The structure-class detector was uncatalogued; ratified and added to the
§10 catalog as **`detect`** (Compute group), described as existing in-scope work made legible, not
new scope. Its *interface* remains to be specified (status K) — a later deepening.

**G2 — Standalone-module contract (CA-8) is asserted but unspecified.** `blas`, `algo`, `part`,
`sketch` must each embed without the platform, enforced by a CI link-test. No system defines what
that boundary *is* (which headers, which allocator seam, which subset of `core` they may touch).
**Recommendation: specify the CA-8 boundary once**, as a cross-cutting note, before those modules
harden — otherwise the link-test is un-writable.

**G3 — The "every parallel kernel states its skew story + contention answer" checklist (§14.11) has
no home.** It is a design-review rule with no interface. **Recommendation: fold it into the kernel
authoring contract** when C2.5/the traversal engine is deepened (it is the natural first client).

**G4 — Determinism (X.4) is a principle with no system.** Principle 8 calls it non-negotiable for
spectral code (C2.2/C2.3), yet no interface exists. It should be specified *before* `eigen`, since
retrofitting bit-exactness is far harder than designing for it.

### Recommended deepening order

Ordered by *leverage × Phase-0 proximity* — deepen what everything else sits on, earliest-phase
first. Each entry is a `docs/design/NN-*.md` note + change-log + ADR where a real decision falls out.

1. ~~**X.2 On-disk format skeleton**~~ — ✅ **done (v0.9, ADR-0024, design/14).** Container of
   immutable 2MB-aligned chunk files + manifest; realizes the `freeze()`/`ContiguousGraphView`
   boundary on disk.
2. **C1.2 Chunked base + delta** — *now the top open item.* The second-most load-bearing
   representation; ADR-0008 is decided but the manifest schema, compaction-strategy interface, and
   ≤2-run guarantee are unspecified. Pairs naturally with the just-landed X.2 (they share the
   manifest).
3. **L0.2 Scheduler interface** — ADR-0021 says a caller scheduler is "honored everywhere," but the
   handle it's honored *through* is undefined. Every parallel kernel needs it; Phase 0.
4. **X.1 Planner interface + plan-space** — now that the lattice defines the plan-space (design/04
   §8), the interface is tractable; unblocks the §8 automation story.
5. **C2.3 `laplace` algorithm selection + oracle** — the flagship's central open technical choice
   (KOSZ vs approximate-Cholesky); higher-risk, Phase 3, but the research-vs-committed line matters
   early for grant framing (§15.1).

Then the Named-only kernels (`blas`, `eigen`, `algo`) as their phases approach, each deepened
against the now-fixed C1.0/X.2/L0.2 spine.

---

## How to read this register over time

- A system moves **N → K → D → S** as work lands. The header count is the project's definition
  frontier at a glance.
- Every **S** points at its `docs/design/NN-*.md`. Every **D** points at its ADR. Anything **K/N**
  is a to-do with a known home.
- New systems enter here first (name + status **N**), never by silent accretion — same discipline
  as the SOW change log and the annual radar (§14.14).
