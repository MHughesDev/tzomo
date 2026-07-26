# Week-1 Checklist

Everything is decided; this is execution only.

**Day 1–2 — claims and shell**
- [ ] Reserve GitHub org (fallback handles: tzomo-graph, tzomolib if bare taken) and create `tzomo` repo (private)
- [ ] Publish minimal `tzomo` 0.0.1a0 to PyPI (name reservation via release — README + link only)
- [ ] Check crates.io / npm / conda-forge handles; register a domain (tzomo.dev or tzomo.org)
- [ ] Run USPTO/EUIPO word-mark search (posture: unregistered mark, ADR-0017/§15.6 — search is due diligence only)

**Day 2–4 — toolchain and CI**
- [ ] CMake skeleton (ADR-0014): core library target, FetchContent-consumable, C++23
- [ ] CI: Linux x86-64 + ARM builds; ASan/UBSan/TSan jobs; clang-format/tidy config
- [ ] `decisions/` seeded with ADR-0001..0018 (this package's decisions/ directory drops in as-is)
- [ ] PRINCIPLES.md committed (this package's copy)

**Day 3–7 — infrastructure and funding**
- [ ] Benchmark harness skeleton: GAP dataset fetch + checksum pinning; small/medium/large tiers stubbed
- [ ] License-audit process file (SOW §14.8): dependency + algorithm check log started (first entries: Mt-KaHyPar MIT verified 2026-07-25)
- [ ] Begin NLnet application draft; check Sovereign Tech Fund and CZI EOSS current cycles (§15.1 — cycles run months, start now)
- [ ] Hardware order per §13 spec (channels-first EPYC/Threadripper, 512GB–1TB, striped NVMe, 2× 4090-class)

**Parallel, days 1–30**
- [ ] Prior-art study per research/prior-art-study-plan.md
