# Candidate Amendments from Deep-Dive Round 2

Status: **RATIFIED IN FULL (2026-07-25, owner) — applied as SOW v0.8**; ADR-0019..0022 cut; ADR-0008/0010 extended. Each maps to a SOW section or ADR and states
its source system(s). Ratify, modify, or reject; accepted items enter via SOW change-log
(→ v0.8) and new/updated ADRs.

## Tier 1 — recommend adopting (high value, low cost)

**CA-1. Amalgamated build target.** (SQLite amalgamation + stb) Ship `tzomo.h`/`tzomo.c`
(or single-header core subset) generated from CMake, alongside normal packaging. "Add two
files to your build" is the strongest embeddability device known. → SOW §14.4, Layer 6;
new ADR.

**CA-2. Planner + wisdom component, named now.** (FFTW; OpenBLAS dispatch) Upgrade §8's
"v1 selects explicitly; automatic later" into a named `tzomo-runtime` component: a planner
that *measures* representation/ordering/kernel/block choices on the actual machine and a
persisted wisdom cache. v1 ships explicit mode + the interface; Phase 3 ships measurement.
→ SOW §8, §5 Layer 0; new ADR.

**CA-3. Composability contract in the C ABI.** (oneTBB arenas; OpenBLAS threading bugs)
Written guarantee: caller-provided scheduler honored everywhere, max-concurrency cap, no
thread-local surprises, no global pools, polite-citizen behavior when embedded in hosts with
their own runtimes. Elevates the audit-5 workspace rule to a public contract. → SOW §14.11,
ADR-0011 extension or new ADR.

**CA-4. Compaction policy vocabulary + write-amplification as a benchmark row.** (RocksDB,
Lucene) Chunk compaction gets: pluggable merge policies, explicit stall/slowdown budgets at
ingest saturation, and write-amplification reported in §14.1 alongside bytes-per-edge.
→ ADR-0008 extension, SOW §14.1.

**CA-5. Format compatibility window: read N−1.** (Lucene; Parquet feature flags) §14.5's
"stated compatibility promise" gets its concrete answer: current major reads previous major's
files; migration happens on compaction; evolution rules (add-only, deprecate-don't-delete,
alignment invariants) are part of the v1 spec, written spec-first as if others will
implement. → SOW §14.5; chunk-format ADR when authored.

**CA-6. License-permanence statement.** (Redis/Elastic/Terraform sagas) One public paragraph:
MIT forever, DCO means relicensing is structurally impractical, no CLA will ever be
introduced. Converts legal structure into an adoption asset aimed at saga-burned enterprises.
→ SOW §15.4; COMMUNITY/GOVERNANCE doc at Phase 1.

**CA-7. API tiering doctrine.** (zstd) The C ABI ships three complete tiers: one-line simple
calls, context/workspace calls, advanced-parameter calls — nobody forced upward. Plus the
zstd-style single-dial convention for speed/quality/effort knobs where applicable.
→ SOW §14.11.

## Tier 2 — recommend adopting (moderate cost, clear payoff)

**CA-8. Standalone-usability requirement for compute modules.** (Velox) `blas`, `algo`,
`part`, `sketch` each embeddable without the platform; kernel-granularity adoption is a
first-class channel. Mostly enforced by existing layering — this makes it a stated
requirement with a CI link-test. → SOW §5, §12 success criteria.

**CA-9. Adopt Highway (vendored) for the SIMD layer.** (Highway) Resolves the audit-13
build-vs-adopt ADR toward adopt: battle-tested across our exact ISA matrix, VLA-native;
vendoring preserves consumers' dependency-freedom. Condition: prior-art read confirms no
disqualifier; CHECK RVV target maturity. → New ADR closing the open ADR question.

**CA-10. mmap read path rehabilitated for frozen chunks.** (LMDB) Refine audit 19's verdict:
immutable chunks satisfy LMDB's precondition, so the convenience mmap path over *frozen*
data is a supported, documented mode (write/ingest paths remain io_uring). → SOW §5 Layer 1
wording; ADR-0008 note.

**CA-11. Support-policy and security-process documents at Phase 1, not later.** (curl, zlib)
SECURITY.md (reporting, disclosure timeline), SUPPORT.md (what is and isn't supported,
response expectations, the polite no). Pre-empts the support-burden risk the moment adoption
starts. → SOW §14.7, §15.4 scheduling.

**CA-12. Launch mechanics checklist from the Astral playbook.** (uv/ruff; Polars) Phase-1
launch requires: versioned compatibility promise published, reproducible benchmark page live,
migration guide written, and a first-90-days issue-response-velocity commitment (the
skeptic-conversion window). Also: the shim converts users, the native API keeps them —
native-API ergonomics get explicit design attention in Phase 1, not just the shim.
→ SOW §11 Phase 1.

## Tier 3 — worth logging, apply opportunistically

**CA-13. Block-size calibration point.** (ClickHouse) ~64K-element processing blocks as the
starting heuristic for binned-pass sizing; planner (CA-2) tunes from there. → runtime detail.

**CA-14. Options introspection.** (PETSc) Every knob queryable at runtime (already implied
by §14.10 — this adds the PETSc-style enumerate-all-options capability). → SOW §14.10.

**CA-15. Fuzz-hours and test-ratio as published metrics.** (SQLite) Release notes state
cumulative fuzz time and suite size; releases gate on full oracle suites. Posture, not
MC/DC. → SOW §14.2.

**CA-16. NCCL vocabulary in the device model.** (NCCL) Already consistent with ADR-0010;
recorded so the plugin interface names collectives identically (all-reduce, all-gather,
reduce-scatter, broadcast). → ADR-0010 note, Phase 5 shelf.

**CA-17. Wedge discipline restated.** (RethinkDB postmortem) One sharp wedge per phase;
a second simultaneous wedge is refused even when tempting. §11's structure already encodes
this — the postmortem is the citation that keeps it honest. → SOW §11 preamble note.

## Explicit refusals harvested (no SOW change; recorded to stay refused)

- RocksDB's hundred-knob configuration surface (opinionated defaults instead).
- Eigen-depth template metaprogramming and its diagnostics (concepts + erasure instead).
- OpenBLAS-style hidden global threading (scheduler handles, always).
- PETSc's everything-configurable UX and MPI-everywhere assumption.
- Polars' break-compatibility-first sequencing (we shim first; parity boundary stated loudly).
- Astral's funding model (mechanics yes, VC clock no).
- Any benchmark configuration that would earn early-ClickHouse's reputation.
