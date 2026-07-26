# ADR-0008: Chunked immutable base + delta overlay

## Status
Accepted (2026-07-25, SOW v0.7)

## Context and Problem Statement
Monolithic immutable CSR forces full-graph rewrites per compaction at streaming ingest rates (catastrophic write amplification); five separate concerns (mutation, retention, temporal windows, snapshots, out-of-core I/O) each demanded a storage answer.

## Decision Drivers
- Audit 15: write amplification at Domain Test 3 rates
- Audit 16: temporal windows
- Domain Test 10: retention
- Audit 19: O_DIRECT units
- Audit 3: 2MB alignment

## Considered Options
- Monolithic base + delta
- PMA in-place dynamic structure
- Chunked immutable base (vertex-range + time partitioned) + delta overlay + per-chunk compaction

## Decision Outcome
Chosen option: **Chunked immutable base under a manifest; per-vertex has-delta bitmap gating sorted-run merge; LSM-style per-chunk compaction**.

One architecture serves all five concerns. The largest single finding of the performance-audit series.

Extension (CA-4/CA-5/CA-10, v0.8): compaction merge policies are pluggable strategy objects with explicit stall/slowdown budgets (RocksDB/Lucene vocabulary); write amplification is a first-class benchmark row; chunk format carries a read-N−1 compatibility window with migrate-on-compact; the mmap read path over frozen chunks is a supported mode (LMDB precondition satisfied by immutability).

### Consequences
- Good: unification (principle 3 enforced by physics)
- Bad: more complex than monolithic CSR; complexity bought once in Layer 1 behind the GraphView concept
