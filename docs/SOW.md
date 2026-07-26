# Statement of Work — v0.1 (FROZEN DRAFT)
## Project name: **Tzomo**

**Date:** 2026-07-25
**Owner:** Solo developer
**Status:** v0.1 frozen. Amendments require a change-log entry (§18).

---

## 1. Purpose & Vision

Build the most complete graph computation platform ever created: a ground-up, modern C++
family of SDKs spanning **laptop to data center**, **simple graphs to higher-order topology**,
and **static snapshots to unbounded temporal streams**.

The organizing thesis is **embedded-core, orchestrated scale-out**: a zero-dependency,
in-process, embeddable engine is the foundation; distribution is a separate layer that
orchestrates many embedded engines. Single-node is never the degenerate case.

**Driver:** engineering obsession and demonstrable performance supremacy — not monetization.
Career optionality (hire, acquisition, consulting, later commercial entity) is treated as a
byproduct of the platform being visibly the best engineering in its space, never as a
design input.

**Scale of endeavor:** decade-class. This is an accepted property of the plan, not a defect.

---

## 1B. Landscape Assumption Register

Every external-world claim the plan depends on, status-flagged. Re-verified annually by the
technology radar (§14.14). Status: VERIFIED (checked 2026-07), HIGH CONFIDENCE (stable fact),
PROJECTION (hedged by design, not by prediction).

**Competitive:** Kuzu archived Oct 2025 (Apple acqui-hire) — VERIFIED, but the vacuum is now
*contested*: LadybugDB (funded, enterprise support, pivoting to Arrow/DuckDB "graph lakehouse"),
a Vela multi-writer fork, and ArcadeDB marketing into the gap. Revision: the Phase-4 DB decision
is made against the 2029 field, not the 2025 vacuum — which §11 already prescribes. NetworkX
backend-dispatch machinery (entry-points, can_run/should_run, convert-and-cache) — VERIFIED,
and it strengthens the official-backend strategy. Graphviz aging, ARPACK-era code under SciPy —
HIGH CONFIDENCE (claim stays precisely worded: no *production C++* Laplacian solver; research
Julia code exists). Mt-KaHyPar is MIT — VERIFIED (sequential KaHyPar is GPLv3); Domain Test 6's
"must build from scratch" is retracted (see §17 item 9 ruling).

**Ecosystem:** Free-threaded Python officially supported (non-default) since 3.14, ecosystem
migrating; C-extension compatibility is the gating factor — VERIFIED; our bindings must be
explicitly FT-marked. ISO GQL published 2024, first native implementations appearing, Cypher
remains the de facto daily language — VERIFIED. Arrow as interchange lingua franca — HIGH
CONFIDENCE, corroborated by competitors converging on it. Hardware trajectory (SVE/RVV, CXL
tiers, P/E cores, coherent GPU memory) — PROJECTION, hedged by the v0.5 design decisions.
EU CRA obligations reach adopters ~Dec 2027 — HIGH CONFIDENCE.

**Market:** Solo-infra reputation converts to career/exit optionality — VERIFIED by the
founding example itself (Kuzu team's acqui-hire is the Phase-6 thesis demonstrated). Grant
programs exist for this class — HIGH CONFIDENCE; cycles/eligibility checked at application
(§15.1). Agent-driven graph demand — VERIFIED indirectly: competitors ship MCP servers and
position for "agentic AI" today, meaning §14.13 is table stakes forming, not speculation.

**Deltas absorbed (2026-07):** (1) contested-not-empty DB space — plan unchanged, framing
updated; (2) nx-cugraph occupies part of the "fast NetworkX" story — Phase 1 positioning
sharpened in §11; (3) MCP servers are table stakes by Phase 3, no longer a differentiator.

---

## 2. Naming & License

**License: MIT.** Confirmed. Maximum adoption; the moat is execution velocity, not legal text.
*Accepted risk:* MIT carries no explicit patent grant, which matters in patent-dense adjacent
domains (EDA, codecs). Logged in the risk register (§16) rather than reopened.

**Name: Tzomo. CONFIRMED.**

Coined from the Hebrew *tzomet* (צוֹמֶת) — a junction, an intersection, a crossroads; the
everyday Hebrew word for a network node. Shortened deliberately: the full word is carried by an
Israeli political party (tsomet.co.il) and by the Zomet Institute, so the clipped form keeps the
meaning and the sound while leaving both the political association and their search results
behind.

Pronunciation: *TSOH-moh* (initial "ts" as in *cats*; *ZOH-moh* is an acceptable anglicization).

Rejected candidates:
- **Spectra** — taken, direct collision: an established header-only C++ sparse eigensolver
  ("Sparse Eigenvalue Computation Toolkit as a Redesigned ARPACK", MPL2, spectralib.org) —
  precisely the component this platform intends to replace.
- **Spektral** — taken (Python GNN library on Keras/TF2).
- **Osmium** — taken: libosmium/pyosmium, an established C++ library for OpenStreetMap data.
- **Fiedler** — GitHub handle unavailable; math-eponym direction abandoned by preference.
- **Tzomet** — namespace free, but search results dominated by an unrelated political party.

**Conventions:** umbrella naming in the Apache-Arrow/Boost style — `tzomo-core`, `tzomo-blas`,
`tzomo-layout`, … ; C++ namespace `tzomo::`; C ABI prefix `tzm_`; Python package `tzomo`.

**Verification status (2026-07-25):** web search confirms **no software collisions** — the name
surfaces only a Chabad niggun, an art studio, and a music track. Software namespace clean.
Remaining: reserve GitHub org and PyPI name in week 1 (PyPI names are reserved by publishing a
minimal release, not by request); confirm crates.io/npm/conda-forge and run a word-mark search
before first public release.

---

## 3. Guiding Principles

1. **Extreme scale is the design center.** Billions of edges on one machine is the *baseline*
   workload. Every structure is evaluated at 10B edges. 2× memory overhead is a bug.
2. **Modern to the bone.** C++23 (concepts, ranges, `std::expected`); portable SIMD with
   AVX-512/NEON/SVE specializations; io_uring; NUMA-awareness; GPU as first-class backend.
   Modern in *knowledge* too: implement the 2015–2025 literature, not textbook versions.
3. **Reuse through layering, not copy-paste.** One parallel runtime, one traversal engine, one
   partitioner, one transport. Modules may not grow private copies of core capabilities.
4. **Speed is the product.** Every kernel benchmarked against the best-known competitor from
   day one; regressions block merges. "Fast enough" means *winning*. Profiling infrastructure
   is core code.
5. **Safety without paying for it.** ASan/UBSan/TSan and fuzzing always in CI; debug builds
   validate everything; release builds check at API boundaries, never in hot loops; ownership
   semantics that make use-after-free impossible through the public API; checksummed on-disk
   formats.
6. **Obsession-driven, not market-driven.** Features ship because the engineering is right.
   Long detours for small wins are legitimate work — that is the structural advantage over
   funded teams.
7. **Verifiable correctness.** Every kernel has an oracle: brute-force reference, property-based
   tests, cross-checks against NetworkX/igraph/graph-tool. Wrong-but-fast is unrecoverable.
8. **Determinism by option.** Bit-exact reproducible mode and fast mode, explicitly selected and
   documented per kernel. Non-negotiable for floating-point spectral code.
9. **Embeddable anywhere.** Zero mandatory dependencies in the core, no global state, pluggable
   allocators and thread pools, builds everywhere including WASM.
10. **Boring interfaces, radical internals.** Innovate ruthlessly inside kernels; keep APIs
    conservative, stable, documented. Documentation is an engineering standard.
11. **Measured honesty.** Publish losses alongside wins. Trusted numbers compound forever.

**Tiebreakers (frozen):**
- Performance wins *inside* kernels; safety is enforced at boundaries.
- Modernity wins over legacy-toolchain compatibility. Built for the next decade.

---

## 4. Data Model

**Universal core object:** a directed, weighted, typed multigraph with temporal edges and
columnar properties — expressed on an **incidence-structure foundation** (vertices,
hyperedges-as-vertex-sets with optional ordering/orientation, boundary/coboundary operators).

**Three arity tiers (frozen, §7.4):**
- **Pairwise** — dominant fast path (CSR family). Must pay *nothing* for the platform's
  generality: an unweighted static simple graph runs BFS exactly as fast as a bespoke engine.
- **Hypergraph** — native storage *and* native kernels, including partitioning.
- **Simplicial** — Hodge Laplacian and higher-order spectral analysis in scope. Persistent
  homology / TDA is a separately-scoped optional module (§6, item 10).

**Constrained views (all in scope, all zero-cost where applicable):** undirected (symmetric
view), simple (dedup constraint), unweighted (weight column absent), static (time column
absent), bipartite/k-partite (type constraint), DAGs and trees, signed weights, mixed direction.

**Also in scope:** probabilistic/uncertain graphs; temporal graphs (timestamps/intervals as
data, time-respecting paths, temporal centrality); streaming graphs (unbounded ingest with
bounded memory).

**Structure classes detected and dispatched on:** sparse/dense, power-law, small-world vs.
high-diameter, planar/near-planar, DAG/tree, community structure, bounded-width.

---

## 5. Scope — IN

### Layer 0 — Systems foundations
Arena/slab allocators over a **memory-tier abstraction** (enumerable tiers with
bandwidth/latency attributes and per-structure placement hints; NUMA is the two-tier special
case, HBM/CXL-attached/pooled memory are further tiers — the out-of-core gradient extended
upward); work-stealing parallel runtime with a **heterogeneity-aware scheduler** (per-core
capability metadata — frequency class, cache topology — so hub-vertex mega-tasks are not
scheduled onto efficiency cores); portable SIMD layer whose **core model is vector-length-
agnostic** (SVE/RVV-style), with fixed-width AVX-512/NEON as specializations — not the reverse,
plus an enumerated set of width-specialized escape hatches for the compute-bound kernel set
(intersection, decode, dense micro-kernels, ingest primitives); **the two signature runtime
primitives** (Perf Audit series): *binned/blocked passes* (seven jobs: contention privatization,
cache blocking, false-sharing elimination, permutation application, radix sort, scatter-to-
sequential conversion, wire batching) and support for *freeze-time vertex reordering* (six
jobs — see Layer 1); batched software-prefetch utilities (kernels own their loops; hot paths
consume contiguous spans, never element iterators); a contention toolkit (privatized
accumulators, hub-aware striping; pull-mode as the default answer to write contention);
hierarchical NUMA-aware barriers; explicit huge-page support (THP/madvise policy, hugetlbfs
for benchmarking); per-thread **workspace objects** (preallocated reusable scratch, cache-line
aligned, surfaced through the C ABI so repeated small-graph calls amortize to zero allocation);
compressed-integer primitives (Roaring, Elias-Fano, delta encoding); **communication
abstraction** (no-op → NCCL/MPI/RDMA backends, interface defined early, backends implemented
late); pluggable everything; zero mandatory dependencies; io_uring-based out-of-core I/O.

### Layer 1 — Representations
Immutable CSR/CSC (dual out/in); the **chunked immutable base** (§7.8): vertex-range- and
time-partitioned immutable chunks under a manifest, plus a mutable delta overlay (per-vertex
has-delta bitmap gating sorted-run merge) with per-chunk background compaction — one storage
architecture serving mutation, streaming retention (admit/retire chunks), temporal windows
(whole-chunk concatenation + boundary slivers), MVCC snapshots (a snapshot is a manifest), and
O_DIRECT I/O units (chunks are 2MB-aligned); compressed (WebGraph-style; on compressible graph
classes the *preferred* form for scan-dominated workloads, since 8–16× byte reduction beats
decode cost on bandwidth-bound kernels); temporal (time-columned edges, window materialization
when a window will see 2–3+ kernel passes, direct time-filtered traversal otherwise);
out-of-core (io_uring + O_DIRECT double-buffered chunk pipeline for write/ingest and
random-fault-prone paths; the **mmap read path over frozen chunks is a supported, documented
mode** — CA-10: immutable chunks satisfy LMDB's precondition, under which mmap reads are
excellent); partition-aware shard form; locally-dense/hybrid
block support; **freeze-time vertex reordering** as a standard on-by-default optimization pass
(pluggable orderings; six jobs: cache locality, hub identification, skew splitting, branch
predictability, compression ratio, ordering–encoding co-selection — free of user-visible
consequences because internal IDs are already divorced from external IDs, §7.2); explicit
conversion between all representations, every conversion O(m) sequential except time-reordering
(O(m log m) external sort — documented, and ingest preserves rough time order to avoid it);
permutations applied via binned two-pass scatters.

### Layer 2 — Compute kernels
**Standalone-usability requirement (CA-8, Velox model):** compute modules (`blas`, `algo`,
`part`, `sketch`) are each embeddable *without the platform* — kernel-granularity adoption is
a first-class channel, enforced by a CI link-test per module.
GraphBLAS-style sparse linear algebra (SpMV, SpGEMM, semirings, masking); frontier/traversal
engine (direction-optimizing); **sparse eigensolvers** (Lanczos, LOBPCG, Jacobi-Davidson — the
ARPACK/Spectra replacement); **Laplacian solvers and spectral sparsification** (core thesis,
§7.6); **construction/ingest kernel family** (parallel sort, hash, dedup, external-memory
build) as first-class, benchmarked work.

### Layer 3 — Algorithms
Traversals, centrality, components, shortest paths (incl. delta-stepping), max-flow/min-cut,
k-cores, triangle/motif counting, community detection (Leiden, spectral, multilevel), matching
and alignment, subgraph isomorphism — each with incremental/streaming variants where
achievable; structure-class detection with specialized dispatch.

### Layer 4 — Applied engines
- **Graph *and* hypergraph partitioning** — Mt-KaHyPar (MIT) adopted as baseline; in-house
  differentiating layer on top (spectral, incremental, layout/distribution-integrated). See
  catalog entry and §17 item 9 ruling.
- **Layout engine** — Graphviz replacement: multilevel force-directed, spectral drawing, stress
  majorization, edge bundling, incremental relayout, dot-compatible CLI (common-path).
- **GPU visualization renderer** — millions of elements, LOD, picking/interaction APIs.
- **Streaming engine** — incremental analytics, change/anomaly detection, retention/window/
  eviction lifecycle.
- **Matching / entity-resolution toolkit.**
- **Graph-ML plumbing** — neighborhood samplers, minibatching, embeddings, DLPack/PyTorch/ONNX
  export, GPU-resident structures, and **subgraph tokenization/serialization** for
  transformer-style and foundation-model consumers. Plumbing only; models stay out.
- **Approximate/sketching toolkit** — HLL, LSH, sampling, sparsifiers.
- **External-ID dictionary component** — two-phase design (audit 17): sharded mutable hash
  during ingest (tens of millions of ops/s/core, shards near-linearly), frozen
  **minimal-perfect-hash form** (~3 bits/key + packed strings) after freeze, mirroring the
  building→frozen epoch pattern; Arrow dictionary arrays for string properties. Memory is the
  binding constraint, not throughput — at 1B unique keys the mutable dictionary can exceed the
  graph itself. Engineered, not incidental.
- **Embedded graph database** — persistence, WAL, snapshots, transactions, query layer over an
  adopted standard (GQL / openCypher family).

### Layer 5 — Scale-out
Multi-GPU single-node (intermediate proving ground); multi-node CPU **and** GPU clusters;
data-center deployable. **Three execution modes over one partitioner and one transport:**
(a) linear-algebra-parallel (primary spine, 2D-partitioned SpMV/SpGEMM),
(b) vertex-centric BSP (convenience layer compiled onto the same substrate),
(c) asynchronous/priority-driven (required for high-diameter and latency-sensitive workloads).
v1 semantics: gang-scheduled, fail-stop, deterministically restartable.

### Layer 6 — Interfaces
Stable C ABI (handle-based, status codes, versioned option structs, no exceptions across the
boundary); Python bindings via nanobind/pybind11 over the C++ layer with a NetworkX-compatible
shim (common-path parity, published incompatibility list); WASM builds; Arrow interop; common
format I/O (GraphML, GML, edge lists, Parquet, mmap-native).

### Standing infrastructure
Public benchmark harness (GAP, LDBC) covering **small (10³), medium (10⁶), and large (10⁹)
graphs plus construction time**, with regression CI and published losses; correctness oracles;
sanitizers and fuzzing in CI; determinism modes; technical writing as a continuous deliverable;
`PRINCIPLES.md`.

---

## 6. Scope — OUT (auditable boundaries)

For each: what is out, what stays in that resembles it, why, and the reconsideration trigger.

1. **GUI applications.**
   *Out:* desktop apps, Electron/Qt shells, visual graph editors, a Gephi-competitor
   application, maintained notebook UIs.
   *In:* the rendering engine; interaction primitives (picking, pan/zoom, selection APIs);
   WASM demos on the project site; unsupported `/examples` viewers.
   *Why:* different discipline, endless polish treadmill, no advancement of the performance
   thesis. You ship the engine that makes great graph GUIs possible.
   *Reconsider:* never.

2. **Hosted services.**
   *Out:* SaaS, managed cloud, hosted benchmarking, any endpoint with uptime obligations or
   on-call.
   *In:* software others deploy in their own data centers; container images and deployment
   artifacts; a static project website.
   *Why:* operations is a permanent solo tax; principle 6 forbids revenue-driven features.
   This is a deferral of optionality, not a renunciation.
   *Reconsider:* only with a second maintainer or a separate commercial entity.

3. **Pure-theory research without implementation payoff.**
   *Out:* theorem-proving as a goal, open-conjecture chasing, papers whose results never reach
   the codebase.
   *In:* implementing frontier theory; papers and talks about those implementations; small
   lemmas forced by an implementation.
   *Why:* the specific failure mode of a mathematically-inclined obsessive.
   *Audit test:* does this work produce repo code within a quarter?
   *Reconsider:* bounded theory work is in-scope when an implementation is genuinely blocked.

4. **Novel query-language invention.**
   *Out:* designing a language, syntax, or semantics; proprietary extensions to an adopted
   standard.
   *In:* implementing GQL/openCypher; programmatic query-builder APIs (an API is not a
   language); internal algebra/IR.
   *Why:* decade-scale sinkhole with network-effect adoption dynamics; ISO GQL exists.
   *Reconsider:* if higher-order structures prove inexpressible — answer is a minimal
   documented extension, never a new language.

5. **Non-graph generality.**
   *Out:* becoming a dataframe library, tensor framework, general sparse-LA product, generic
   distributed-compute framework, or workflow engine.
   *In:* Arrow interop; the sparse-LA core (it exists *for* graphs — the boundary is API
   surface and positioning, not code); property operations sufficient for graph workloads.
   *Audit test:* does the feature make sense to someone with no graph? If yes, suspicious.
   *Reconsider:* never as strategy; accidentally-excellent components may graduate to
   standalone libraries without the platform pivoting.

6. **Official bindings beyond C ABI / Python / WASM.**
   *Out:* Rust, Java, Go, R, Julia, C#, Node bindings authored and supported in-house.
   *In:* a C ABI explicitly designed for others to bind; linking community bindings from the
   docs; fixing C-ABI deficiencies those authors report.
   *Why:* each binding is a permanent maintenance surface with its own ecosystem norms.
   *Reconsider:* bless/adopt a community binding if one demonstrably gates adoption.

7. **Windows/mobile as tier-1 performance targets.**
   *Out:* MSVC codegen chasing, Windows-specific perf paths, mobile OS support, Windows CI as
   a merge blocker.
   *In:* **Tier 1** Linux x86-64 and Linux ARM (performance-tuned, benchmark targets);
   **Tier 1.5** macOS (CI-tested, correctness-supported, performance-untuned — chosen
   deliberately because Phase-1 Python users are heavily on Macs); **Tier 2** Windows and
   **RISC-V Linux** (build, pass tests, no perf promises — RVV falls out of the length-agnostic
   SIMD core); WASM as its own tier, with ambition raised from demos to a supported compute
   target (WASM64 + threads) by Phase 3.
   *Reconsider:* only if GPU vendor realities force it.

8. **Bug-for-bug incumbent compatibility.**
   *Out:* Graphviz dot quirks beyond the common path, NetworkX edge-case semantics, full
   RDF/SPARQL compliance, Neo4j wire-protocol emulation, float-exact output matching.
   *In:* correct common-format I/O; the NetworkX shim at documented ~95% parity with a
   published differences list; incumbents used as *test oracles*.
   *Why:* compatibility tails are asymptotic; a differences doc converts the gap from bug
   reports into documentation.
   *Reconsider:* case-by-case if one quirk blocks a major integration. Never as policy.

9. **Cluster elasticity and fault recovery beyond fail-stop (v1 boundary).**
   *Out (v1):* automatic node-failure recovery, checkpoint-restart machinery, elastic
   scale-up/down, Kubernetes operators, multi-tenancy, deep scheduler integration.
   *In:* fail-stop semantics; deterministic restartability; cheap manual checkpoint hooks;
   deployment artifacts; running as a gang job under Slurm/K8s.
   *Why:* fault tolerance is the hardest half of distributed systems; MPI/HPC/ML-training
   prove fail-stop is production-legitimate.
   *Reconsider:* if job durations make rerun-on-failure painful, add checkpoint-restart —
   before anything elastic, ever.

10. **Persistent homology / full TDA library.** *(New — from Domain Test 7.)*
    Higher-order **spectral** work (Hodge Laplacians, simplicial spectra) is IN. Boundary-matrix
    reduction, filtrations, Vietoris-Rips construction, and persistence diagrams are a distinct
    kernel family with combinatorial blow-up risk. Explicitly deferred to an optional module,
    decided deliberately rather than by drift.

11. **SPARQL and the RDF stack.** *(Now explicit.)* RDF triples map onto typed edges and are
    ingestible; SPARQL engines, reasoners, and OWL semantics are out.

12. **Bitemporal modeling outside the DB layer.** Valid-time/transaction-time duality, if ever
    supported, belongs to the DB module. The analytics core carries one time dimension.

13. **Preprocessing-based routing methods (contraction hierarchies et al.) as a v1
    commitment.** *(From Domain Test 9.)* Delta-stepping and async execution cover routing
    adequately for v1; CH/CRP are a separate paradigm, revisitable later.

---

## 7. Frozen Architectural Decisions

**7.1 Embedded-core, orchestrated scale-out.**
The core is a zero-dependency, in-process, embeddable library. Distribution runs one engine
instance per node/GPU with a coordination layer above it. Scale-*up* (out-of-core, NUMA,
multi-GPU-on-one-box) lives in the core; scale-*out* lives in orchestration. Rationale:
matches DuckDB/CombBLAS/NCCL, preserves adoption physics (99% of users are single-node
forever), reconciles principles 1 and 9, and keeps the project solo-testable for years.

**7.2 Vertex identity and integer widths.**
Dense internal IDs, 0..n-1. **Compile-time template parameter; both 32- and 64-bit variants
shipped in the binary; runtime-selected at graph creation** (<4.29B vertices → 32-bit) with
explicit override. **Vertex-ID width and degree/offset width are independent parameters**
(forced by de Bruijn graphs: 64-bit IDs, tiny degrees). **Distributed mode uses partition-local
32-bit IDs regardless of global size.** External IDs never enter kernels; they live in the
dictionary component (§5, Layer 4).

**7.3 Edge identity.**
CSR-positional by default (zero overhead for analytics). **Opt-in stable edge IDs** via a
permutation map, required by the dynamic layer and by the DB's transactional semantics. The
analytics hot path never pays for stability.

**7.4 Higher-order model.** Incidence-structure foundation with three arity tiers (see §4).
Pairwise kernels never touch incidence structures; conversion is explicit.

**7.5 Property and type system.** Columnar, external to topology, **Arrow-typed** (do not
invent a type system). Three levels: schemaless (exploration, NetworkX shim), schema-declared
(DB and production, enforced at ingest), compile-time-bound (C++ users bind columns to static
types for kernel-speed access). Vertex/edge *types* are a dense categorical column that storage
and dispatch may partition on.

**7.6 Laplacian solvers are core thesis, not a Phase-3 curiosity.**
Near-linear-time Laplacian solving and spectral sparsification ship in Layer 2 alongside the
eigensolver. Rationale: largest theory-practice gap in the field; feeds the in-house
partitioner, layout, sparsification, max-flow, effective resistance, FEM and connectomics
domains simultaneously — the most cross-cutting component in the platform.
*Risk-managed as:* the **eigensolver is the committed deliverable**; the Laplacian solver is
the stretch flagship within the same module.

**7.7 Mutability contract.** Explicit epochs (building / frozen). An epoch/snapshot is a
**chunk manifest** (see 7.8) — immutable, cheap to hold, and the mechanism the DB uses for
MVCC snapshots.

**7.8 Chunked immutable base + delta overlay.** *(Restructured by the performance-audit
series — the largest single finding.)* The base is not one monolithic immutable CSR but a set
of **immutable vertex-range- and time-partitioned chunks under a manifest**, each 2MB-aligned.
Mutation lands in a small delta overlay (per-vertex has-delta bitmap, cache-resident at 100M+
vertices, gating a sorted-run merge on read); background compaction rewrites *individual
chunks*, LSM-style, bounding write amplification and pause times at high ingest rates
(monolithic-base compaction would rewrite the full graph per cycle — unacceptable at Domain
Test 3 rates). The same architecture yields: streaming retention/eviction as chunk
admit/retire (Domain Test 10), temporal windows as whole-chunk concatenation plus boundary
slivers (audit 16), snapshots as manifests (7.7), and O_DIRECT out-of-core units (audit 19).
Replaces the earlier "PMA/blocked structure" and "single immutable base" designs.

**7.9 DB-on-top, never DB-inside.** The analytics core has zero transaction awareness. The DB
module owns durability and hands frozen immutable snapshots to the core. Transactions produce
epochs; analytics consumes epochs.

**7.10 GPU support is a dynamically-loaded plugin, defined against an abstract device model.**
The core never links CUDA/ROCm; a CPU-only build stays dependency-free. The plugin *interface*
is specified in terms of an abstract device model — memory spaces, streams, kernel launch,
collectives — with CUDA as the first implementation, not the definition (HIP/SYCL/Metal remain
implementable). The device model represents **shared/coherent memory** (unified-memory APUs,
Grace-Hopper/MI300A-class), not only discrete copy-based devices. Resolves the conflict between
principle 9 and the GPU backend. Affects the build system from day one.

**7.11 Small-graph fast path is architecture, not optimization — with a stated budget.**
Total per-call overhead through the Python binding: **≤2–5μs** (the igraph/graph-tool class,
which is the real small-graph competition — NetworkX itself pays per-edge interpreter cost and
is easy to beat). Enforced by: zero-copy NumPy result views over Tzomo-owned memory (capsule
ownership, never copied), workspace reuse through the C ABI (audit 5), and a CI benchmark that
is literally a 500-node call in a loop. The NetworkX shim requires an explicit **conversion-
caching design** — convert once, attach the frozen graph to the NetworkX object, invalidate by
version-stamp on mutation (documented in the differences list) — because naive per-call O(m)
conversion would silently destroy the small-graph story (Domain Test 4). A genuinely
single-threaded,
allocation-light execution path with parallelism thresholds, present from Phase 0. Forced by
Domain Test 4 (see §9).

**7.12 Execution modes.** One partitioner, one transport, three faces: LA-parallel (spine),
vertex-centric BSP (sugar), asynchronous/priority-driven. The async mode is a **single-node
runtime capability arriving early** (delta-stepping is its first client, Phase 1 territory),
not merely a distributed face: audit 8 quantified that sparse-frontier supersteps on
high-diameter graphs (road networks, diameter ~4,000) spend 50%+ of wall time in barriers.
The structure-class detector estimates diameter (double-BFS sweep) so kernels select sync
vs. async policy automatically. On GPU the same problem reappears as kernel-launch overhead;
answered there by persistent kernels / device-side frontier loops (audit 21).

---

**7.13 Instantiation policy.** *(Closes open item: blessed precompiled set.)* **Structural
parameters (representation family, ID/offset width) are compile-time templates; value-type
parameters are type-erased by default** — audit 25 established that in bandwidth-bound kernels
a runtime column pointer + stride + type tag costs nothing measurable — **specialized only in
the enumerated compute-bound kernel set** (sorted-neighbor intersection, compressed-format
decode, dense micro-kernels, ingest primitives, LOBPCG internals). This collapses the
instantiation product from thousands to ~(5 representations × 2 widths) structural variants
plus a handful of typed hot kernels: binary in the tens of MB, sane compile times, no icache
thrash. Extern-template instantiation units for build parallelism; icache-pressure and HITM
perf-counter checks in CI (§14.2).

---

## 8. Workload × Representation Matrix

| Workload | Representation |
|---|---|
| Batch analytics | Immutable CSR (32-bit where possible) |
| High mutation / streaming ingest | Base CSR + delta overlay + compaction |
| Larger-than-RAM | mmap CSR + io_uring out-of-core partitioned passes |
| Temporal window queries | Time-columned edges → window materialization to CSR |
| Streaming analytics | Delta overlay + incremental algorithm state + retention policy |
| Huge static in-RAM | WebGraph-style compressed, decode-on-traverse |
| ML sampling | GPU-resident CSR + precomputed alias tables |
| Construction-dominated (e.g. de Bruijn) | External-memory build pipeline → frozen CSR |
| Locally dense (FEM, connectomics) | Hybrid block / dense-block-aware sparse LA |
| Distributed | 2D-partitioned shards, each internally CSR, partition-local IDs |
| Hypergraph (e.g. netlists) | Native incidence storage + hypergraph partitioner |

Selection is owned by a named component (CA-2, ADR-0020): the **planner** — which *measures*
representation/ordering/kernel/block choices on the actual machine, FFTW-style — with a
persisted **wisdom** cache amortizing planning across runs. v1 ships explicit mode plus the
planner interface; measurement-driven automation lands Phase 3. Starting block-size heuristic
~64K elements (CA-13), tuned by the planner thereafter.

---

## 9. Domain Pressure-Test Record

Ten domains chosen to span scale, construction cost, mutation rate, diameter, arity, typing,
latency, hardware, and unboundedness. Findings that changed the design are marked ⚠.

1. **Web/social analytics** (100B+ edges, power-law, batch) — holds. Exercises the 4.29B ID
   boundary; motivates partition-local IDs.
2. **Genome assembly / de Bruijn** (construction-dominated, arity ≤4) — ⚠ construction is 80%
   of runtime and was under-scoped as "io". Promoted to a first-class kernel family. Proved ID
   width and degree width are independent knobs.
3. **Financial fraud / transactions** (100k edges/sec, temporal, concurrent query+analytics) —
   ⚠ per-epoch CSR rebuild is catastrophic write amplification. Forced the base+delta overlay
   design (7.8). DB-on-top and epochs-as-MVCC otherwise validated.
4. **Code / build / dependency graphs** (10³–10⁵ nodes, DAG, latency-critical) — ⚠ **most
   important finding.** A billion-edge architecture will lose to NetworkX on a 500-node graph
   via thread-pool wakeups, allocation, and dispatch overhead. The Phase-1 launch claim dies on
   the first small-graph benchmark posted by a skeptic. Forced 7.11 and the small/medium/large
   benchmark requirement.
5. **Knowledge graphs / GraphRAG** (heterogeneous, string IDs, query-heavy) — holds, but the
   external-ID dictionary is often the largest memory consumer and the ingest bottleneck.
   Promoted from "optional layer" to an engineered named component.
6. **EDA netlists / place-and-route** (arity in the thousands, partitioning-dominated) — holds
   structurally; validates native hyperedges. ⚠ legal blocker as originally found — *revised
   v0.7:* Mt-KaHyPar verified MIT (sequential KaHyPar remains GPLv3), so the blocker applies
   only to the sequential/hMETIS lineage; the from-scratch mandate is retracted in favor of
   adopt-plus-differentiate (§17 item 9).
7. **Scientific meshes / FEM / connectomics** (simplicial, Laplacian-heavy) — splits. Hodge
   Laplacians run through the same sparse-LA machinery (elegant, in scope); persistent homology
   is a different kernel family (now §6.10). Also forced hybrid dense-block support.
8. **Graph ML training** (sampling-dominated, GPU-resident) — holds. ⚠ exposed the
   zero-dependency/CUDA conflict; resolved by 7.10.
9. **Road networks / routing** (diameter in the thousands, latency-sensitive) — ⚠ BSP collapses
   (thousands of supersteps). Forced the third, asynchronous execution mode (7.12).
10. **Cybersecurity provenance** (unbounded streams, retention limits) — ⚠ nothing specified
    what happens when the stream never ends. Streaming is a memory-bounded *lifecycle*
    (retention, TTL, window eviction, deleting compaction), not merely incremental algorithms.

---

## 9B. Performance-Audit Record

Twenty-five audits across six batches (memory & core; parallelism; SIMD; representations;
I/O & scale-out; boundaries & meta), applying hardware physics to every frozen design element.
Score: 19 amend, 6 hold-with-refinement, zero frozen decisions broken, one substantially
restructured (§7.8, the chunked base — the series' largest finding). Full analyses live in the
project ADRs; the five governing results:

1. **The bandwidth gradient governs everything:** 3TB/s HBM → 500GB/s DRAM → 64GB/s PCIe →
   25–50GB/s network → 7–14GB/s NVMe. Kernels sit at 1–2% of compute peak (≈0.15 flops/byte);
   the platform's scale-out story is the discipline of algorithm design down this gradient.
   Consequences: bytes-per-edge as the design metric, SIMD redirected away from traversal
   loops, type erasure free in the hot path, GPU doctrine resident-first, distributed pitch
   capacity-not-speedup.
2. **Two signature primitives crowned.** Freeze-time vertex reordering (six jobs) and
   binned/blocked passes (seven jobs) — when four unrelated failure modes converge on the same
   primitive set, that set is where the platform's performance identity lives.
3. **The chunked immutable base unified five concerns** — mutation, streaming retention,
   temporal windows, MVCC snapshots, O_DIRECT I/O — previously treated as separate designs
   (§7.8). Principle 3 enforced by physics.
4. **The partitioner promoted a third time** — now the determinant of distributed viability,
   with a quantitative quality target.
5. **Measured honesty acquired engineering content:** transfer-inclusive GPU numbers, scaling
   claims only from NVSwitch-class hardware, %-of-STREAM reporting, CI-enforced small-graph
   budget (§14.1).

---

## 10. Deliverable Catalog

**Core:** `core` (representations, properties, conversion), `runtime` (scheduler, SIMD,
transport abstraction), `io` (formats, ingest), `dict` (external-ID mapping),
`build` (construction kernels).

**Compute:** `blas` (GraphBLAS-style), `eigen` (sparse eigensolvers), `laplace` (Laplacian
solvers, sparsification), `algo` (algorithm battery), `part` (graph + hypergraph partitioning —
strategy revised v0.7 after license verification: **Mt-KaHyPar is MIT** (the sequential KaHyPar
is GPLv3), retracting Domain Test 6's build-from-scratch blocker. Ruling: **adopt/vendor
Mt-KaHyPar as the baseline engine; in-house work becomes the differentiating layer** — spectral
initial partitioning via `laplace`, streaming/incremental repartitioning, and native integration
with layout and the distributed substrate. The partitioner remains the determinant of
distributed viability (audit 23: partition quality moves communication volume 5–20×; quality
target — comm volume vs. random baseline), but Phase 2 no longer carries an unplanned
from-scratch build), `community`, `stream`, `sketch`.

**Applied:** `layout`, `viz`, `match`, `ml`, `db`.

**Scale-out:** `dist` (partitioning, collectives, three execution modes), `gpu` (plugin).

**Interfaces:** `capi`, `python` (+ NetworkX shim as a separate package), `wasm`.

**Infrastructure:** `bench`, `PRINCIPLES.md`, differences/compatibility docs, project site.

---

## 11. Phasing

Hard rule across all phases: **no new module begins before the previous one beats its
incumbent.** Sprawl, not difficulty, is the failure mode at this scope. Corollary (CA-17,
RethinkDB postmortem): **one sharp wedge per phase** — a second simultaneous wedge is refused
even when tempting; technical superiority must be *experienced in minutes* (shim,
amalgamation, benchmark site are the delivery mechanisms).

- **Phase 0 — Foundations (0–6 mo).** `core`, `runtime`, `io`, `build`, `bench`. Billion-edge
  load; GAP-suite kernels beating igraph/NetworKit; small-graph fast path proven. Quiet.
- **Phase 1 — First public win (6–12 mo).** `algo`, `eigen`, `capi`, `python` + NetworkX shim,
  **registered as an official NetworkX dispatch backend** (decided v0.7 — the backend machinery's
  convert-and-cache and can_run/should_run hooks solve half of audit 24's shim design, and
  registration is the distribution channel). Positioning sharpened against nx-cugraph (Delta 2):
  **CPU-native, no GPU required, embedded anywhere, billions of edges on one machine — and
  competitive at 500 nodes, where GPU backends structurally lose to transfer overhead.** Launch
  claim verified on small graphs before publication. **Launch mechanics per the Astral playbook
  (CA-12):** versioned compatibility promise published, reproducible benchmark page live,
  migration guide written, first-90-days issue-response-velocity commitment; the shim converts
  users, the **native API keeps them** — native-API ergonomics get explicit Phase-1 design
  attention. SECURITY.md and SUPPORT.md ship with the launch (CA-11). Success: external
  issues, first outside citation.
- **Phase 2 — Showcase (yr 1–2).** `layout` + `viz` + `part`. Visual, self-marketing
  superiority over Graphviz. Success: downstream integrations exist.
- **Phase 3 — Depth (yr 2–3).** `stream`, `laplace` flagship, `blas` maturity, `sketch`,
  `community`, higher-order tiers. Converts "faster known things" into "capabilities with no
  incumbent." Talks and papers.
- **Phase 4 — Persistence (yr 3–4).** `db` on a proven analytics core with an existing user
  base — categorically stronger than building it cold.
- **Phase 5 — Scale-out (yr 4–6).** Multi-GPU single node first (partitioning and communication
  without network/failure complexity), then multi-node CPU, then multi-node GPU.
- **Phase 6+ — Harvest.** Reputation-driven optionality: hire, consulting, commercial entity,
  or acquisition. No path is chosen in advance; all are downstream of the same thing.

**Phase 0 ninety-day plan.**
- Days 1–14: name verification and namespace reservation; repo creation; build system and
  toolchain; CI with sanitizers; `PRINCIPLES.md`; `decisions/` seeded with ADRs for the twelve
  frozen decisions in §7; benchmark harness skeleton with GAP dataset fetching; license/patent
  audit process established (§14.8).
- Days 1–30 (in parallel): **prior-art study** (§14.3) — source-level reading of the incumbent
  ecosystem, written up internally.
- Days 15–45: `core` v0 (CSR builder, 32/64 instantiation, property columns, GraphML/edge-list
  ingest); first kernel (BFS) exercising the runtime; **single-threaded allocation-light fast
  path** (a day-45 deliverable, not an optimization).
- Days 46–75: PageRank, connected components, triangle counting; second representation and the
  conversion layer.
- Days 76–90: honest benchmarking against NetworKit/igraph/graph-tool at 10³/10⁶/10⁹ scale
  **including construction time**; publish nothing; fix what loses; write the internal
  postmortem deciding whether Layer-0 foundations hold or need rework before Phase 1 builds on
  them.

---

## 12. Success Criteria

- **Phase 0:** GAP/LDBC wins at all three scales; construction competitive; no small-graph
  regression versus NetworkX.
- **Phase 1:** external users filing issues; first citation or third-party write-up.
- **Phase 2:** named downstream projects using the layout engine.
- **Phase 3:** at least one capability with no incumbent equivalent (streaming/incremental or
  production Laplacian solving).
- **Phase 5:** deployed multi-node run at data-center scale by someone other than the author.
- **Long-run:** maintainer reputation sufficient that hire/acquire/consult optionality exists
  without being pursued.

---

## 13. Constraints & Assumptions

- Solo developer, full-time equivalent, 2+ years runway, no income pressure.
- C++23 toolchain; Linux x86-64 and ARM as tier-1.
- **Hardware: ~$10–15k initial single machine, selected bandwidth-first.** Audit 1 inverts the
  usual spec priorities: **memory channels over core count** (a 12-channel EPYC beats a
  higher-clocked 8-channel part — cores are not the scarce resource, bandwidth is);
  **512GB–1TB RAM** (billion-edge graphs in RAM is the daily loop); **striped multi-NVMe**
  (out-of-core kernels are disk-bound 40–70×, so aggregate storage bandwidth is the out-of-core
  ceiling; audit 19); and **two** prosumer GPUs (4090-class) — two, not one, because the
  multi-GPU path cannot be developed on a single device, with the documented epistemics that
  this box validates correctness, never scaling claims (§14.1). Multi-node hardware deferred
  entirely; cloud spot instances for cluster testing when Phase 5 arrives.
- MIT license; permissive-only dependencies (a live constraint — it is what forces the in-house
  partitioner).

---

## 14. Engineering Process

Principles 4, 7, and 11 make measurement and correctness the entire credibility story. This
section specifies how that is actually done, because an unspecified benchmark claim is an
attackable one.

**14.1 Benchmark methodology.**
- **Dedicated bare-metal benchmarking machine**, reserved and otherwise idle. Cloud and shared
  CI runners are too noisy to publish from; results from them are for regression detection
  only, never for public claims. **DECIDED (v0.7):** during Phase 0–1 development, the primary
  workstation under a controlled protocol (pinned governor, isolated, no concurrent load)
  serves for development and regression numbers; **a second, cheaper dedicated unit is acquired
  before any Phase-1 claim is published** — published numbers never come from the daily-driver.
- **Statistical discipline:** minimum run counts, medians and interquartile ranges (not means),
  reported variance, warm/cold cache states declared, CPU frequency governor and turbo state
  pinned and documented.
- **Pinned everything:** competitor library versions, compiler versions and flags, dataset
  versions and checksums, OS and kernel version — all recorded with each published result.
- **Reproduction scripts published** alongside every claim, so a skeptic can rerun it.
- **Fairness policy, written and public:** competitors are configured as their own
  documentation recommends, tuned in good faith, and given their best representation. Where a
  competitor wins, the result is published (principle 11).
- **Scale coverage mandatory:** every headline claim is verified at 10³, 10⁶, and 10⁹ vertices,
  and includes construction/ingest time, not only kernel time (Domain Tests 2 and 4).
- **Bandwidth-native metrics:** kernels are designed, documented, and benchmarked in
  **bytes-per-edge**, and published results report achieved bandwidth as a percentage of the
  machine's measured STREAM — GFLOPs are meaningless for a platform whose kernels sit at 1–2%
  of compute peak (audit 1).
- **GPU honesty:** every published GPU number states whether host↔device transfer is included;
  resident and non-resident cases are reported separately (audit 21). The two-consumer-GPU dev
  box (no NVLink; peer transfers at PCIe speed) validates correctness and communication design
  but **cannot produce publishable multi-GPU scaling claims** — those require NVSwitch-class
  rented hardware (audit 22). Distributed results are framed as **capacity/weak scaling**, not
  fixed-graph speedup curves, because the bandwidth gradient makes communication-dominance at
  O(10) nodes a matter of physics, not engineering (audit 23).
- **Small-graph budget in CI:** the ≤2–5μs per-call overhead (§7.11) is enforced by a
  continuously-run benchmark; construction throughput targets (edges/sec, GB/s per format) are
  first-class benchmark rows (audit 20); **write amplification** under sustained ingest is a
  first-class row alongside bytes-per-edge (CA-4, RocksDB precedent), with compaction stall
  budgets stated and tested.

**14.2 Testing and correctness.** Per-kernel oracles (brute-force reference implementations);
property-based testing; differential testing against NetworkX/igraph/graph-tool; synthetic
generators for structured inputs (R-MAT, Kronecker, LFR, planar, high-diameter); ASan/UBSan/TSan
in CI; continuous fuzzing of all parsers and format loaders; determinism-mode verification as an
explicit test class; releases gate on full oracle suites, with **cumulative fuzz-hours and
test-suite size published in release notes** (CA-15, SQLite posture); an `alignas(64)` alignment-annotation convention with perf-counter (HITM /
frontend-stall) CI checks for false sharing and icache pressure (audits 10, 25).

**14.3 Prior-art study (Phase 0 deliverable).** Systematic source-level study of NetworKit,
graph-tool, igraph, SNAP, SuiteSparse:GraphBLAS, Ligra, Galois, GraphIt, cuGraph, CombBLAS,
KaHyPar/METIS, and LadybugDB — what each got right, where each is slow, and why. Written up
internally. Weeks of work that prevents years of reinvention, and it is also the raw material
for the benchmark suite and the differences documentation.

**14.4 Build, packaging, distribution.** Build system — **DECIDED (v0.7): CMake** (ADR-0013).
Rationale: consumers must be able to vendor via FetchContent/find_package; vcpkg and conda-forge
integration is native; every alternative (Meson, Bazel) costs embedder friction that violates
principle 9. Revisitable only by superseding ADR.
Distribution targets: source, vcpkg, conda-forge, manylinux wheels via cibuildwheel, an
**amalgamated build target** (`tzomo.h`/`tzomo.c`, CMake-generated — the SQLite/stb
"add two files to your build" device; CA-1, ADR-0019), and distro packaging left to distros. Cross-compilation supported; binary size tracked as a metric;
reproducible builds as a goal; **SLSA-style build provenance and sigstore-signed artifacts**
from the first public release.

**14.5 On-disk format versioning.** The mmap-native format is as permanent as the ABI once users
have files. Requires from v1: magic number, explicit format version field, **2MB alignment for
all mappable sections** (audit 3: huge-page mapping is a compatibility property of the format,
not a runtime optimization — a 40GB edge array under 4K pages TLB-thrashes by design),
**a read-N−1 compatibility window** (current major reads previous major's files; migration on
compaction; CA-5, Lucene model), schema-evolution rules stated in the v1 spec (add-only,
deprecate-don't-delete, alignment invariants; FlatBuffers/Cap'n Proto discipline), the spec
written **spec-first as if others will implement it** (Arrow discipline), documented
endianness policy, checksums, a stated forward/backward compatibility promise, and a migration
path for breaking changes.

**14.6 Numerical policy.** Precision strategy (fp32/fp64/mixed) declared per kernel; overflow
and saturation behavior specified; NaN/infinity handling documented; cross-architecture
reproducibility policy (x86 and ARM vector reductions differ) stated explicitly, alongside the
determinism modes of principle 8.

**14.7 Security posture.** Parsers are the attack surface; all format loaders are assumed to be
fed hostile input. Requires: a written threat model, continuous parser fuzzing, a published
security disclosure policy and contact, signed releases, and an SBOM once binaries are
distributed. Infrastructure receives CVEs; the process exists before the first report.
**CRA-readiness is named explicitly:** the EU Cyber Resilience Act reaches open-source stewards
and, more forcefully, this platform's commercial adopters — who will require SBOM, disclosure
process, update policy, and provenance from upstream. Having those artifacts simply already
there is an adoption edge with exactly the serious organizations the platform wants.

**14.8 License and patent discipline (standing process).** Every dependency *and every
algorithm* is checked for licensing and patent encumbrance before implementation, with a written
record. This is not a one-time audit — it is what surfaced the hypergraph-partitioner blocker
(Domain Test 6). Benchmark datasets carry licenses too and are checked on the same basis.

**14.9 Decision records.** The twelve frozen decisions in §7 and every subsequent significant
decision are recorded as ADRs in **MADR 4.0** format at `decisions/NNNN-short-title.md`,
co-located with the code. Each records context, decision drivers, options *considered*, outcome,
and consequences — one decision per ADR, status kept current, superseded records linked rather
than deleted. Rationale: in year four the reasoning must be recoverable, and the decision log is
what makes a decade-scale solo project legible to a future contributor, employer, or acquirer.

**14.10 Observability and configuration.** Logging, tracing, and profiling hooks are core
features, not tooling afterthoughts (principle 4). Explicitly: no telemetry, ever — the library
never phones home. Configuration and autotuning policy defined, with all tuning knobs
introspectable and overridable, including a PETSc-style enumerate-all-options capability
(CA-14); autotuning is owned by the planner/wisdom component (§8, ADR-0020).

**14.11 API discipline.** Written internal style guide; a single error taxonomy shared across
C++, C ABI, and Python layers; documented deprecation policy; **API tiering** (CA-7, zstd
model): the C ABI ships three complete tiers — one-line simple calls, context/workspace
calls, advanced-parameter calls — nobody forced upward, with single-dial conventions for
speed/quality/effort knobs; a written **composability contract** (CA-3, ADR-0021):
caller-provided scheduler honored everywhere, max-concurrency cap, no TLS surprises, no
global pools — Tzomo embedded in a host with its own runtime is a polite citizen; and a
documentation standard
treated as a merge requirement (principle 10). Error messages are written to be actionable by
both human and non-human readers: state what was wrong *and* what would have been valid.
Design-review checklist items from the audit series: every parallel kernel states its **skew
story** (edge-based work splitting via CSR-offset binary search is the default; audit 7) and
its contention answer (pull-mode default; audit 6); SIMD effort follows the audited priority
order — decode, intersection, dense micro-kernels, ingest first; traversal loops last, because
they are bandwidth/latency-bound and vectorization buys little there (audit 11); branch
doctrine is *sort first, branchless second, measure always* (audit 12).

**14.12 Safety model (published document).** The §3.5 stance made explicit and public: what is
safe by construction through the public API, what the sanitizer/fuzzing regime covers, and what
remains trusted. Written for the memory-safety-regulated decade this project will live in.
Corollary: first-class community **Rust bindings are a strategic asset** under §6.6's
adopt-and-bless clause — "Rust-callable with a documented safety contract" is how C++
infrastructure stays legitimate under regulatory pressure.

**14.13 Agent legibility.** A growing share of invocations over the platform's life will come
from AI agents acting on a user's behalf; software that is legible to agents is
disproportionately adopted. Deliverables: (a) a deterministic CLI with structured JSON output
for every operation; (b) machine-readable API descriptions and an `llms.txt`-style
documentation surface; (c) a thin **MCP server over the C ABI** as a Phase 3+ deliverable —
software users run themselves, consistent with §6.2. Python bindings are designed **GIL-free
from day one** (free-threaded CPython): no hidden global state at the binding layer, so
multiple Python threads can drive concurrent kernels — a Phase 1 differentiator no incumbent
has.

**14.14 Annual technology radar.** Future-proofing is a practice, not a 2026 list. A standing
one-day review each year assesses hardware (accelerators, PIM, CXL fabric, vector ISAs),
language (C++26 reflection — likely transformative for the property/type binding layer;
`std::execution` vs. the in-house scheduler; `std::simd`), and ecosystem (free-threaded Python
adoption, WASM64, graph foundation models, agent usage modes, regulation) against the frozen
decisions — producing ADR updates or explicit "no change" records. Standing watch list, not
built until they win: processing-in-memory and sparse accelerators (the semiring-kernel
interface is already the shape such a backend would plug into), TEE/confidential computing,
quantum graph algorithms. Explicitly rejected: blockchain/decentralized anything.

---

## 15. Project Sustainability

**15.1 Funding path — DECIDED (v0.7): grants-first, layered.** §13 records 2+ years of runway;
§11 runs into year 6 and beyond. The gap is closed by a layered strategy, all compatible with
principle 6: **primary — open-source grant applications during Phase 0** (NLnet, Sovereign Tech
Fund, CZI EOSS; NSF POSE if eligible — cycles run months, so applications begin while runway is
long); **secondary — GitHub Sponsors** passive from Phase 1 visibility; **tertiary — time-boxed
consulting** at high rates from Phase 2 reputation onward, capped so it never exceeds one day a
week; **documented fallback — paid role with reduced velocity**, the phase plan stretching
~3× with the project continuing nights and weekends. Reviewed at every phase boundary.
The option set considered:
- **Open-source grants** — NLnet, Sovereign Tech Fund, NSF POSE, CZI EOSS (the scientific-
  computing and eigensolver angle is a genuine fit). Non-dilutive, no product obligations.
- **GitHub Sponsors / individual sponsorship** — slow to accumulate, meaningful only after
  Phase 1 visibility.
- **Academic affiliation** — visiting researcher or affiliated-scholar arrangement; publishable
  frontier work (§7.6) makes this plausible and would also serve §15.5.
- **Consulting at a low duty cycle** — high hourly rate, strictly time-boxed, from Phase 2
  reputation onward.
- **Paid role with reduced project velocity** — the honest fallback: the project continues
  nights and weekends at perhaps a third the pace, and the phase plan stretches accordingly.
Deciding this late converts it from a strategy into an emergency.

**15.2 Kill criteria and off-ramps.** A decade-scale plan without stated abandonment conditions
is how year seven gets spent on something abandoned internally in year four. Written triggers:
- **Phase 0:** if the foundations cannot beat NetworKit/igraph on the GAP quintet at 10⁶ and
  10⁹ scale after the ninety-day postmortem and one corrective iteration, the Layer-0 design is
  wrong — rework or stop, do not build Phase 1 on it.
- **Phase 1:** if the NetworkX-compatible launch produces no external users, issues, or
  third-party writeups within six months, the adoption thesis is wrong — reconsider the
  interface strategy before building further.
- **Phase 2:** if no downstream project adopts the layout engine within a year of release, the
  "visible flagship" bet failed; the platform continues but the marketing thesis changes.
- **Phase 5:** if distribution has not begun by the time single-node work is complete and
  runway is spent, cut it — an excellent single-node platform is a complete success on its own.
- **Any phase:** if the work stops being the thing you want to do, that is a sufficient reason
  to stop. Principle 6 cuts both ways.

**15.3 Personal sustainability.** The risk register's "phase independence" is not a sufficient
mitigation for ten years of solo work. Concretely:
- **Isolation is the actual failure mode**, more than difficulty or motivation. Deliberate
  countermeasures: conference attendance from Phase 1, a small correspondence circle of people
  working on adjacent systems, and public build-logs that create low-cost interaction.
- **Sustainable rhythm:** defined working hours, real weekends, and periodic full stops.
  Ten-year projects are not won by sprinting.
- **Knowledge capture:** the ADR log (§14.9), design notes, and written postmortems exist partly
  so the project is not solely resident in one person's head — this is bus-factor insurance and
  onboarding material simultaneously.
- **Health as a project dependency**, stated plainly because at this timescale it is one.

**15.4 Community and governance.** Contribution terms (DCO preferred over CLA for a
permissively-licensed solo project); a code of conduct; an issue-triage policy with explicit
permission to say no (curl's model); a documented scope-refusal stance pointing at §6;
maintainer succession intent stated before it is needed; a published **license-permanence
statement** (CA-6): MIT forever, DCO makes relicensing structurally impractical, no CLA will
ever be introduced — the legal structure stated as an adoption asset for enterprises burned by
the Redis/Elastic/Terraform relicensing sagas; and **SECURITY.md + SUPPORT.md shipped at
Phase 1 launch, not later** (CA-11, curl/zlib precedent) — reporting process, disclosure
timeline, what is and isn't supported, response expectations, the polite no. Note the asymmetry: adoption success creates support burden,
which is itself a sustainability risk (§16).

**15.5 Academic engagement.** Target venues where this work would be cited and where the
frontier components (§7.6) are publishable: SIGMOD, VLDB, SC, PPoPP, ALENEX, and the graph-
drawing venues for the layout engine. Purpose is citation-durability and credibility, not
career points — and it doubles as the isolation countermeasure in §15.3.

**15.6 Trademark posture — DECIDED (v0.7):** unregistered mark with a published usage policy
(the standard posture for a permissively-licensed project); registration deferred until and
unless a commercial entity forms, at which point it is that entity's first legal task.

---

## 16. Risk Register

| Risk | Severity | Mitigation |
|---|---|---|
| **Runway (2+ yr) is shorter than the plan (6+ yr)** | Critical | §15.1 funding path — an open decision that must be closed early, not at exhaustion |
| Sprawl at decade scope | Critical | No-new-module-before-victory rule; this SOW as the arbiter |
| Isolation over a decade of solo work | High | §15.3 — conferences from Phase 1, correspondence circle, public build-logs |
| Adoption success creates unbounded support burden | Medium | §15.4 triage policy and explicit permission to decline; §6 as the stated scope defense |
| Fixed-width SIMD or discrete-GPU assumptions require rewrite as SVE/RVV and unified-memory hardware mainstream | High | Length-agnostic SIMD core and coherent-capable device model frozen at Layer 0 (v0.5) |
| Unspecified benchmark methodology makes claims attackable | Medium | §14.1 — dedicated bare-metal machine, pinned versions, published reproduction scripts, published losses |
| Hostile input via format parsers (CVE exposure) | Medium | §14.7 — threat model, continuous parser fuzzing, disclosure policy before first report |
| Small-graph regression kills Phase-1 launch | Critical | 7.11 fast path; three-scale benchmarks from day one; verify before publishing |
| Incorrect results damage benchmark credibility | Critical | Principle 7 oracles; sanitizers/fuzzing; publish losses |
| Solo bus factor / burnout over 10 years | High | Phase independence; each phase independently valuable and citable |
| Template instantiation combinatorics explode compile time and binary size | ~~High~~ Resolved | §7.13 policy: structural-only templates, type erasure free in bandwidth-bound kernels (audit 25) |
| Chunked-base architecture (§7.8) is more complex than a monolithic CSR and is now load-bearing for five subsystems | Medium | It is also the *only* design that survived audits 15/16/19 and Domain Tests 3/10; complexity is bought once, in Layer 1, behind the GraphView concept |
| Distributed systems are a different discipline | High | Fail-stop only; multi-GPU-single-node as proving ground; simple execution models |
| In-house partitioner is unplanned critical-path work | Medium | Scheduled in Phase 2 alongside layout, which needs it anyway |
| Laplacian solver is research-grade and may fail | Medium | Eigensolver is the committed deliverable; Laplacian is the stretch flagship |
| Higher-order native model has thin prior art | Medium | Pairwise fast path is unaffected; hypergraph tier validated by EDA domain |
| MIT lacks a patent grant in patent-dense adjacent domains | Low–Medium | Accepted risk, logged; avoid patent-encumbered algorithm choices where alternatives exist |
| Incumbents add speed (NetworkX backend dispatch) | Low | It is also an adoption channel — become a registered backend |

---

## 17. Open-Item Register — ALL CLOSED (v0.7)

1. **Tzomo namespace** — verified clean by web search (no software collisions). Remaining work
   is week-1 keyboard tasks: reserve GitHub org + PyPI (minimal release), confirm
   crates/npm/conda-forge, word-mark search before first release.
2. **Funding path** — DECIDED: grants-first layered strategy (§15.1). Applications begin in
   Phase 0.
3. **Build system** — DECIDED: CMake (§14.4, ADR-0013).
4. **Benchmark machine** — DECIDED: controlled-protocol primary box for development; dedicated
   second unit acquired before any published claim (§14.1).
5. **GQL vs. openCypher** — DECIDED: **openCypher-compatible surface, GQL-aligned core.** The
   query surface ships openCypher (the language graph users — including Kuzu refugees — write
   daily), while the internal semantics/IR align with ISO GQL so standard conformance is an
   additive layer at Phase 4, not a rewrite. Landscape check: GQL published 2024, native
   implementations only now appearing; Cypher remains the de facto standard.
6. **Instantiation set** — CLOSED by §7.13 (v0.6).
7. **Streaming retention/eviction** — DECIDED: chunk-lifecycle retention per §7.8 — policy
   objects (time-TTL, count-based, predicate-based); eviction = chunk retirement; point deletes
   = delta tombstones merged at compaction. Detail ADR at Phase 3.
8. **NetworkX official backend registration** — DECIDED: yes; strategically central (Delta 2);
   Phase 1 deliverable (§11).
9. **Hypergraph partitioner** — DECIDED, revised by license verification: **adopt Mt-KaHyPar
   (MIT) as baseline; build the differentiating layer in-house** (spectral initial partitioning
   via `laplace`, incremental repartitioning, layout/distribution integration). Domain Test 6's
   from-scratch mandate retracted; sequential KaHyPar (GPLv3) and hMETIS remain unusable.
10. **Docs/site/cadence** — DECIDED: docs-as-code in the repo, static site generator, no CMS.
    Internal notes only during quiet Phase 0; public build-log begins at Phase 1 launch,
    roughly monthly plus milestone posts. The blog is the platform's sales team (§11).
11. **Trademark** — DECIDED: unregistered mark + usage policy (§15.6).
12. **Pressure-test exclusions (§6.10–6.13)** — RATIFIED by owner (v0.7): persistent
    homology/TDA out (optional module later), SPARQL/RDF stack out, bitemporal confined to the
    DB layer, contraction hierarchies out of v1. The exclusions now carry explicit rulings, not
    just recommendations.

No owner decisions remain outstanding. Future open items enter through the change log and the
annual radar (§14.14), not by accretion.

---

## 18. Change Log

- **v0.8 (2026-07-25)** — Deep-dive round 2 (24 external systems) ratified in full: all 17
  candidate amendments applied. Amalgamated build target (CA-1, ADR-0019); planner/wisdom
  component named (CA-2, ADR-0020); composability contract (CA-3, ADR-0021); compaction
  vocabulary + write-amp benchmark row (CA-4); read-N−1 format window + spec-first evolution
  rules (CA-5); license-permanence statement (CA-6); API tiering (CA-7); standalone-usable
  compute modules (CA-8); Highway adoption ADR (CA-9, ADR-0022); mmap-over-frozen-chunks
  rehabilitated (CA-10); SECURITY/SUPPORT at launch (CA-11); Astral launch mechanics (CA-12);
  block-size seed (CA-13); options enumeration (CA-14); fuzz-hours publication (CA-15); NCCL
  vocabulary note (CA-16, ADR-0010); wedge discipline (CA-17). Seven refusals recorded in
  research/deep-dive-round2.md remain binding.
- **v0.7 (2026-07-25)** — Landscape verification + total open-item closure. §1B Landscape
  Assumption Register added (Kuzu/LadybugDB contested-space delta, NetworkX dispatch verified,
  free-threaded Python verified, ISO GQL verified, agent-demand corroborated). Tzomo namespace
  verified clean. Mt-KaHyPar verified MIT → partitioner strategy revised to
  adopt-plus-differentiate (Domain Test 6 blocker retracted). Phase 1 repositioned against
  nx-cugraph; official NetworkX backend registration decided. Funding path decided
  (grants-first layered). Build system (CMake), benchmark-machine protocol, trademark posture,
  docs cadence, streaming retention, and openCypher-surface/GQL-core all decided. §6.10–6.13
  ratified by owner. §17 rewritten: **zero open items.**
- **v0.6 (2026-07-25)** — Performance-audit series (25 audits, 6 batches) folded in. §7.8
  restructured: chunked immutable base (vertex-range + time partitioned, manifest snapshots,
  per-chunk compaction) unifying mutation/streaming/temporal/MVCC/out-of-core. §7.11 gains the
  ≤2–5μs binding budget and shim conversion-caching. §7.12: async mode promoted to early
  single-node capability. §7.13 added (instantiation policy; closes open item 6). Layer 0/1
  rewritten: crowned primitives (reordering — six jobs; binned passes — seven), workspace
  objects, huge pages, prefetch/contention toolkits, io_uring out-of-core, compressed-as-
  preferred on compressible classes, two-phase dictionary. §14.1: bytes-per-edge and
  %-of-STREAM metrics, GPU/scaling honesty rules, small-graph CI budget. §14.5: 2MB format
  alignment. §13: bandwidth-first hardware spec (channels over cores, striped NVMe). §9B audit
  record added. Risk register updated (template risk resolved; chunk-complexity risk added).
- **v0.5 (2026-07-25)** — Future-readiness audit folded in. Layer 0: memory-tier abstraction
  (NUMA as special case), heterogeneity-aware scheduler, vector-length-agnostic SIMD core.
  §7.10 rewritten: abstract device model, coherent/unified memory represented, CUDA as first
  implementation not definition. Tiers: RISC-V added at tier 2; WASM ambition raised to
  supported compute target by Phase 3. `ml` module gains subgraph tokenization (plumbing only).
  §14 gains: provenance/signing (14.4), CRA-readiness (14.7), published safety model + Rust-
  bindings posture (14.12), agent legibility incl. JSON CLI, MCP server, GIL-free Python
  bindings (14.13), annual technology radar with watch list (14.14). Risk register: hardware-
  assumption retrofit risk added.
- **v0.4 (2026-07-25)** — Coverage audit. Added §14 Engineering Process (benchmark methodology,
  testing, prior-art study, build/packaging, on-disk format versioning, numerical policy,
  security posture, license/patent discipline, MADR decision records, observability, API
  discipline) and §15 Project Sustainability (funding path, kill criteria, personal
  sustainability, community/governance, academic engagement, trademark). Risk register gained
  six entries, led by the runway-versus-plan gap. Phase 0 gained prior-art study, ADR seeding,
  and the license-audit process. Open items expanded to twelve; sections renumbered.
- **v0.3 (2026-07-25)** — Name changed to **Tzomo** (coined from Hebrew *tzomet*, junction).
  Conventions updated: `tzomo::`, `tzm_`, `tzomo-*` modules. Rejected-candidate record extended
  (Osmium, Fiedler, Tzomet) with reasons.
- **v0.2 (2026-07-25)** — Name confirmed: **Fiedler**. Naming conventions fixed (`fiedler::`,
  `fdl_`, `fiedler-*` modules). Open item 1 downgraded from a blocking decision to namespace
  reservation; new open item 8 added covering the four exclusions introduced by the pressure
  test rather than by explicit owner ruling.
- **v0.1 (2026-07-25)** — Initial freeze. Consolidates: vision and eleven principles; universal
  data model with three arity tiers; full in-scope layer stack; thirteen auditable exclusions;
  twelve frozen architectural decisions; workload×representation matrix; ten-domain pressure
  test and its seven forced amendments (construction kernels, base+delta overlay, small-graph
  fast path, ID dictionary component, in-house partitioner, async execution mode, streaming
  lifecycle); phasing with a ninety-day Phase-0 plan; hardware budget; risk register.
  Name unresolved — "Spectra" found to collide with an existing C++ sparse eigensolver.
