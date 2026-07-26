# Tzomo — Principles

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
