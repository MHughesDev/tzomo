# External Systems Research Log

Everything researched during project inception (2026-07), organized by relationship to Tzomo.
Status tags: VERIFIED (web-checked 2026-07-25), HIGH CONFIDENCE (stable knowledge), CHECK
(verify during prior-art study). This log feeds the §14.3 prior-art study and the §14.14
annual radar.

---

## 1. Incumbents Tzomo intends to beat

| System | What it is | Why it matters | Status |
|---|---|---|---|
| **NetworkX** | Pure-Python graph library, the field's default; millions of users; 100–1000× slower than native | Phase-1 adoption lever via the compatibility shim + official dispatch backend registration. Backend machinery (entry-points, can_run/should_run, convert-and-cache) VERIFIED and load-bearing for our design | VERIFIED |
| **igraph** | C core + Python/R bindings; the serious-user default | The real small-graph competition (~1–10μs/call class); differential-testing oracle | HIGH CONFIDENCE |
| **graph-tool** | C++/Boost + Python; one academic's output; strong performance | Proof one obsessive can build the class of thing we're building; oracle; benchmark competitor | HIGH CONFIDENCE |
| **NetworKit** | C++ parallel graph analytics, academic; our closest analog in scope | Primary benchmark competitor at Phase 0–1; feasibility proof (years-of-one-person scale) | HIGH CONFIDENCE |
| **SNAP (Stanford)** | C++ network analysis + dataset repository | Benchmark competitor; SNAP datasets for the harness | HIGH CONFIDENCE |
| **Graphviz** | 1990s AT&T layout engine; universal invisible plumbing (Doxygen, LLVM/GCC dumps, Terraform graph, PlantUML, pprof) | Phase-2 flagship target: dot-compatible CLI, multilevel/spectral layout at scales where dot chokes | HIGH CONFIDENCE |
| **ARPACK (+ Spectra)** | 1990s Fortran77 sparse eigensolver under scipy.sparse.linalg.eigs, MATLAB eigs, Octave; Spectra is the modern C++ header-only redesign (MPL2) | The `eigen` module replaces this lineage. Spectra also cost us the first project name | VERIFIED (Spectra identity/license) |
| **Gephi** | Java desktop graph-viz app, aging | Motivates `layout`+`viz` engines (we ship engines, not the GUI — SOW §6.1) | HIGH CONFIDENCE |

## 2. Systems to learn from (prior-art study subjects)

| System | Extract |
|---|---|
| **SuiteSparse:GraphBLAS** | Semiring kernel design, masking, the GraphBLAS spec in practice; what a one-maintainer LA core looks like |
| **Ligra** | Frontier abstraction, direction-optimizing traversal — the push/pull switch our audit 6 doctrine generalizes |
| **Galois** | Asynchronous/priority-driven parallel runtime — the prior art for our third execution mode |
| **GraphIt** | Scheduling-language separation of algorithm from optimization — informs kernel/doctrine separation |
| **CombBLAS** | 2D-partitioned distributed sparse LA — the spine of our Layer-5 model |
| **cuGraph / nx-cugraph** | GPU graph kernels; the zero-code-change NetworkX acceleration story we must position against (CPU-native, small-graph, no-GPU) | 
| **WebGraph (LAW)** | Compression formats (gap/Elias-Fano encoding), ordering–compression interaction — audit 14's foundation |
| **Mt-KaHyPar** | ADOPTED as partitioning baseline (ADR-0015). MIT VERIFIED; sequential KaHyPar GPLv3, hMETIS non-free. Study its coarsening/refinement before writing the spectral layer |
| **METIS/Scotch/Zoltan** | Partitioning lineage; license-check each before any code reading (SOW §14.8 discipline) |
| **DuckDB** | Embedded-analytics physics: build/packaging, extension model, out-of-core, community shape — the platform's strategic template |
| **SQLite** | ABI-stability discipline, testing culture, formats-as-contracts |
| **mold linker** | Solo-performance-project → reputation → company arc; benchmark-driven marketing |
| **simdjson / Hyperscan** | SIMD parsing techniques for `io`/`build` ingest kernels (audit 11 priority list) |
| **rr / Pernosco** | (From idea exploration) record-replay engineering culture; not on Tzomo's path but cited in career-arc precedents |
| **Laplacians.jl (Spielman group)** | The research-grade Laplacian solver code that exists; our claim is precisely "no production C++ implementation" — study before building `laplace` |
| **Apache DataSketches** | Sketch/approximate structure reference for `sketch` module |
| **GAP Benchmark Suite** | The benchmark harness's core: reference implementations, datasets, methodology (also the source of propagation blocking) |
| **LDBC (Graphalytics, SNB)** | Benchmark suites for analytics and, later, the DB layer; competitors already market LDBC wins |

## 3. Competitive landscape (the contested Kuzu vacuum) — all VERIFIED 2026-07-25

- **Kuzu**: archived Oct 2025 after Apple acqui-hire of Kùzu Inc. The founding premise of the
  adjacent DB exploration, and the Phase-6 career-thesis demonstrated (team hired for exactly
  this expertise).
- **LadybugDB**: community-successor fork by Ladybug Memory; pivoting from pure embedded DB to
  "graph lakehouse" (DuckDB-storage interop, Arrow/Parquet, object stores); enterprise-support
  positioning; multi-label support added post-fork; WASM bindings; markets to "agentic AI in
  regulated industries" with MCP server + OpenAPI SDKs.
- **Vela-Engineering/kuzu**: fork adding concurrent multi-writer support for multi-agent
  context graphs.
- **ArcadeDB**: multi-model DB marketing aggressively into the gap; claims LDBC Graphalytics
  wins over Kuzu; ships a built-in MCP server.
- **Implication (SOW §1B Delta 1)**: the embedded-graph-DB slot is contested-not-empty; Tzomo's
  Phase-4 DB decision is made against the 2029 field from a proven analytics core, and its
  thesis (analytics/algorithms-first) is what none of these players has.

## 4. Ecosystem facts the plan depends on — VERIFIED 2026-07-25

- **NetworkX dispatch**: third-party backends via entry-point registration; existing backends
  include graphblas, cugraph, parallel; dispatcher supports can_run/should_run and
  convert-and-cache. Registration decided (ADR referenced in SOW §11 Phase 1).
- **Free-threaded Python**: officially supported (non-default) since 3.14 per PEP 779;
  ecosystem migrating; a non-FT-marked C extension re-enables the GIL at import — our bindings
  must be explicitly FT-marked (SOW §14.13).
- **ISO GQL**: ISO/IEC 39075:2024 published April 2024 — first new ISO database language since
  SQL (1987); first native implementations appearing (NebulaGraph Enterprise); Cypher remains
  the de facto daily language. Drives ADR-0016.
- **EU CRA**: obligations reach open-source stewards and (more forcefully) commercial adopters
  ~Dec 2027 — SBOM/disclosure/provenance artifacts are an adoption edge (SOW §14.7).

## 5. Frontier / theory-practice gap (see frontier-implementation-targets.md)

Laplacian solvers (Spielman–Teng lineage), spectral sparsification (BSS), the 2022
almost-linear max-flow result (Chen et al.), dynamic graph algorithms, practical parameterized
algorithms (treewidth/twin-width dispatch). The platform's research moat: first serious
implementations of results that have none.

## 6. Naming research record

Rejected with reasons: Spectra (C++ eigensolver, direct collision), Spektral/Spektra (GNN
library), Osmium (libosmium/pyosmium, OSM C++), Fiedler (handle taken; eponym direction
abandoned), Tzomet (namespace free but search dominated by an Israeli political party;
Zomet Institute adjacent). **Tzomo** verified clean for software (2026-07-25).
