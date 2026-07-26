# Prior-Art Study Plan (SOW §14.3 — Phase 0, days 1–30)

Source-level reading of the incumbent ecosystem, written up internally before Layer-0 code
hardens. Per system, answer: (a) what did they get right that we adopt, (b) where are they slow
and why, (c) what API/design mistake do we refuse to repeat, (d) license status of anything we
might learn *implementation* from (SOW §14.8 discipline — clean-room where needed).

## Reading order (dependency-driven)

**Week 1 — the performance canon**
1. GAP Benchmark Suite reference code — benchmark methodology, propagation blocking, direction-optimizing BFS.
2. Ligra — frontier abstraction; the push/pull machinery our contention doctrine generalizes.
3. NetworKit — the closest scope analog; where a one-professor C++ platform put its complexity.

**Week 2 — linear algebra and compression**
4. SuiteSparse:GraphBLAS — semiring kernels, masking, spec-vs-practice gaps.
5. CombBLAS — 2D partitioning, distributed SpMV; the Layer-5 spine's prior art.
6. WebGraph — encodings, ordering–compression interaction (feeds `core` compressed representation).

**Week 3 — runtime and structures**
7. Galois — async/priority runtime (third execution mode's prior art).
8. GraphIt — algorithm/schedule separation.
9. Mt-KaHyPar — coarsening/refinement architecture of our adopted baseline (ADR-0015); identify the seam where the spectral layer attaches.

**Week 4 — the ecosystem faces**
10. igraph + graph-tool binding layers — per-call overhead anatomy (audit 24 budget calibration).
11. nx-cugraph — backend registration mechanics, can_run/should_run usage, conversion caching; and its small-graph weakness (our positioning).
12. DuckDB — extension/packaging/out-of-core patterns; SQLite — ABI and testing culture (skim, principles-level).
13. Laplacians.jl — what research-grade Laplacian solving looks like before we design `laplace`.

**Deliverable:** one internal write-up per system (a page is enough), plus a consolidated
"design refusals" list — mistakes we are explicitly not repeating — checked into the repo
alongside the ADRs.
