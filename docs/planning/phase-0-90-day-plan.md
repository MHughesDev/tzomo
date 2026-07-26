# Phasing & Phase-0 Ninety-Day Plan

Source: SOW §11 (v0.7).

Hard rule across all phases: **no new module begins before the previous one beats its
incumbent.** Sprawl, not difficulty, is the failure mode at this scope.

- **Phase 0 — Foundations (0–6 mo).** `core`, `runtime`, `io`, `build`, `bench`. Billion-edge
  load; GAP-suite kernels beating igraph/NetworKit; small-graph fast path proven. Quiet.
- **Phase 1 — First public win (6–12 mo).** `algo`, `eigen`, `capi`, `python` + NetworkX shim,
  **registered as an official NetworkX dispatch backend** (decided v0.7 — the backend machinery's
  convert-and-cache and can_run/should_run hooks solve half of audit 24's shim design, and
  registration is the distribution channel). Positioning sharpened against nx-cugraph (Delta 2):
  **CPU-native, no GPU required, embedded anywhere, billions of edges on one machine — and
  competitive at 500 nodes, where GPU backends structurally lose to transfer overhead.** Launch
  claim verified on small graphs before publication. Success: external issues, first outside
  citation.
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
