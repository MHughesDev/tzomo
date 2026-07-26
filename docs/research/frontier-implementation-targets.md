# Frontier Implementation Targets (theory–practice gap)

The research moat: production implementations of results that have none. Committed vs. stretch
per SOW §7.6; all are publishable engineering (SOW §15.5 venues).

1. **Near-linear Laplacian solvers** (Spielman–Teng lineage; practical descendants: KOSZ,
   approximate Cholesky). No production C++ implementation exists; research Julia code
   (Laplacians.jl) does. Stretch flagship of the `laplace` module; the eigensolver is the
   committed deliverable beside it.
2. **Spectral sparsification** (Batson–Spielman–Srivastava lineage; effective-resistance
   sampling). Compress any graph to O(n log n) edges preserving the spectrum. Feeds
   partitioning, solvers, and `sketch`.
3. **Dynamic graph algorithms** — incremental connectivity, shortest paths, PageRank
   maintenance under updates. Theory far ahead of usable implementations; this is the
   `stream` module's thesis and the chunked-base architecture's payoff.
4. **Practical descendants of almost-linear max-flow** (Chen et al. 2022). Impractical as
   stated; any practical fragment is famous work. Watch-and-attempt, no commitment.
5. **Structure-aware dispatch** — treewidth/twin-width/planarity detection driving specialized
   kernels. Parameterized-algorithms theory made production.
6. **Higher-order spectral** — Hodge Laplacians on simplicial tiers through the same SpMV
   machinery; production-quality native hypergraph kernels (with Mt-KaHyPar as partition
   baseline).

Each target gets an ADR when work begins, an oracle before optimization, and a paper/talk when
it lands (SIGMOD/VLDB/SC/PPoPP/ALENEX; graph-drawing venues for layout).
