# Adversarial Audit Record

The SOW survived four adversarial passes: workloads (domain tests), coverage (v0.4), time (v0.5 future-readiness), physics (v0.6, 25 performance audits). This file preserves the two structured records.

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

