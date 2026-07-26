# Risk Register

Source: SOW §16 (v0.7). Reviewed at every phase boundary.

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
