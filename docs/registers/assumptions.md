# Landscape Assumption Register

Source: SOW §1B (v0.7). Re-verified annually by the technology radar (SOW §14.14).

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
