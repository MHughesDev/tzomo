# ADR-0024: On-disk format — container of immutable chunk files + manifest

## Status
Accepted (2026-07-26, SOW v0.9)

Realizes ADR-0007 (epochs = manifests), ADR-0008 (chunked base), and ADR-0023 (the on-disk form
is the persisted `ContiguousGraphView`) on disk. Full spec: `docs/design/14-on-disk-format.md`.

## Context and Problem Statement
§14.5: "the mmap-native format is as permanent as the ABI once users have files." A permanent
format must be specified spec-first (Arrow discipline) before any user has bytes. ADR-0008 already
fixed the storage architecture (immutable 2 MB-aligned chunks under a manifest); this ADR fixes how
that lands on disk — the container model, versioning, endianness, integrity, and the scope boundary
against the mutable/transactional layers.

## Decision Drivers
- Chunk-granular lifecycle (retire, compact, snapshot, O_DIRECT) from ADR-0008 — a monolithic file
  defeats all of it (audit 15 write amplification).
- mmap zero-copy (CA-10/§7.11) must be the only read path — no byte-swap, no decode-on-map for the
  plain form.
- Huge-page mapping is a compatibility property of the bytes, not a runtime choice (audit 3).
- Parsers assume hostile input (§14.7); integrity must be checkable without trusting the file.
- The analytics core has zero transaction awareness (ADR-0009) — the format must stop at the frozen
  form.

## Considered Options
- **Monolithic single file** with internal segments — simplest to ship, but per-chunk compaction
  rewrites the whole file and retirement can't unlink; reintroduces write amplification.
- **Container of immutable chunk files + atomically-swapped manifest** — chosen.
- **Embed the delta/WAL in the format** — rejected: couples the analytics format to transactional
  concerns ADR-0009 assigns to `db`/`stream`; unbounds a format meant to be permanent.

## Decision Outcome
Chosen: **a graph is a directory (or object-store prefix) of immutable, self-describing, 2 MB-
aligned `.tzc` chunk files plus a small `MANIFEST` superblock pointing at an append-only manifest
generation; the manifest is the epoch/snapshot and is installed by a single atomic rename.** The
format encodes **only the frozen `ContiguousGraphView` form** — sorted, tombstone-free neighbor
runs — so mapping a chunk *is* the inverse of `freeze()`. Supporting calls, each recorded in the
design note: canonical **little-endian only** in v1 (big-endian rejected with a named error rather
than swapped, to preserve zero-copy); **xxHash3 per-section/per-header integrity** verified at the
API boundary (not in hot loops), with **authenticity detached** (SLSA/sigstore sidecar) so signing
never disturbs alignment; a **read-N−1 compatibility window** with **migrate-on-compaction** and
per-chunk version dispatch (mixed-generation chunks may coexist transiently); **add-only, deprecate-
don't-delete** schema evolution with permanent section-kind numbers and within-major alignment
invariants; property columns stored as **Arrow IPC** buffers (ADR-0005). v1 **fully specifies the
pairwise tier** and **reserves** `chunk_type` tags + unknown-section tolerance for the hypergraph
and simplicial tiers, so adding them later is a minor (non-breaking) bump.

### Consequences
- Good: chunk-granular retire/compact/snapshot/O_DIRECT all fall out; one atomic rename is the
  crash-safe commit point; the on-disk↔in-memory identity (`freeze()` writes it, mapping it is
  `freeze()`) keeps the two surfaces from drifting; huge-page mapping works for any consumer in any
  language; the frozen-form-only boundary keeps a permanent format bounded.
- Bad: many small-to-large files instead of one (directory hygiene, more fds); ≤2 MB padding per
  chunk; the mutable/WAL story lives elsewhere and must be kept coherent with this at the `stream`/
  `db` seam; big-endian platforms are unsupported by decision.
- Follow-ups: the higher-order (`chunk_type` 1/2) section catalogs are a later deepening; the
  dictionary artifact (`dict.tzd`, C4.7) and provenance sidecar get their own short specs.
