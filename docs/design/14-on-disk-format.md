# Design Deepening — §14.5 On-disk format ("tzomo-frozen" v1)

**Status:** proposed for v0.9. Deepens SOW §14.5 to an implementable spec skeleton.
**Ratifies into:** ADR-0024, a §14.5 amendment, a v0.9 change-log entry.
**Governs / is governed by:** ADR-0007 (epochs = manifests), ADR-0008 (chunked base),
ADR-0023 (the on-disk form *is* the realization of `ContiguousGraphView`), ADR-0002 (ID widths),
ADR-0003 (edge identity), ADR-0005 (Arrow properties), ADR-0009 (DB-on-top), §14.4 (provenance),
§14.7 (hostile-input parsers).

§14.5 opens: *"the mmap-native format is as permanent as the ABI once users have files."* That
permanence is the reason to write it spec-first — *as if a third party will implement a reader*
(Arrow discipline) — before a single byte is committed to a user's disk. This document is that
spec skeleton: the container model, exact superblock and manifest byte layouts, the section table,
the alignment and checksum rules, the version-dispatch algorithm, and a reader conformance
checklist.

The one-line decision (ADR-0024): **the on-disk form is a container of immutable, 2 MB-aligned,
self-describing chunk files plus a small atomically-swapped manifest; it encodes only the frozen
`ContiguousGraphView` form; canonical little-endian; a read-N−1 compatibility window with
migrate-on-compaction; v1 fully specifies the pairwise tier and reserves chunk-type tags for the
higher-order tiers.**

---

## 1. Why a container of chunk files, not one monolithic file

ADR-0008 made the base a set of immutable, vertex-range- and time-partitioned chunks under a
manifest, precisely so that five operations stay cheap: mutation (delta + per-chunk compaction),
streaming retention (chunk admit/retire), temporal windows (whole-chunk concatenation), MVCC
snapshots (a manifest), and O_DIRECT I/O (2 MB-aligned units). A monolithic file defeats every one
of them — retiring a chunk would rewrite the file; compacting one chunk would rewrite the whole
graph (the write-amplification finding, audit 15).

So the **chunk is the unit of storage, addressing, and lifecycle**, and the natural on-disk
realization is:

- **one file per chunk** (immutable once fsync'd; retirement = unlink; compaction = write-new +
  manifest swap), and
- **one manifest** naming the live set — itself the epoch/snapshot (ADR-0007).

A chunk file is independently openable, mmap-able, checksummable, and O_DIRECT-readable. This is
the LMDB precondition CA-10 relies on (immutability makes mmap reads safe) realized at file
granularity.

> **Scope boundary (from ADR-0009).** This format is the *frozen analytics* form. The mutable
> delta overlay and any write-ahead log are **not** in this spec — they belong to the `stream`/`db`
> layers, which persist their own append-only logs and hand the analytics core frozen manifests.
> The analytics core reads manifests + chunks; it never reads a WAL. Keeping the format to the
> frozen form is what keeps it bounded and permanent.

---

## 2. Container layout

```
mygraph.tzomo/                     ← a directory (or object-store prefix); the "graph"
├── MANIFEST                       ← current epoch pointer (superblock) — tiny, atomically swapped
├── manifest-000017.mf             ← epoch 17 manifest (append-only generations; MANIFEST points here)
├── manifest-000018.mf             ← epoch 18 (a later snapshot; both retained while referenced)
├── chunk-0000000a.tzc             ← immutable chunk files (.tzc), 2 MB-aligned internally
├── chunk-0000000b.tzc
├── ...
├── dict.tzd                       ← frozen minimal-perfect-hash external-ID dictionary (C4.7), optional
└── PROVENANCE                     ← detached SLSA provenance + sigstore signature refs (§14.4), optional
```

- `MANIFEST` is a fixed-size superblock holding a magic, format version, and the filename +
  checksum of the *current* manifest generation. Epoch switch = write the new `manifest-NNNNNN.mf`,
  fsync, then atomically rewrite `MANIFEST` (write-temp + rename) — a single rename is the commit
  point, giving crash-atomic snapshot installation without a lock.
- A **snapshot / MVCC read** pins a manifest generation; its chunks are immutable, so a reader holds
  a consistent view with no coordination while compaction writes new chunks and later manifests.
- Object-store deployments map the same structure to a key prefix; `rename`-atomicity becomes a
  conditional-put on `MANIFEST`.

---

## 3. Byte layouts

All multi-byte integers **little-endian** (§8). All offsets are from the start of the containing
file. `u8/u16/u32/u64` are unsigned; `i8` etc. signed.

### 3.1 Chunk superblock (`.tzc`, first 4 KiB)

| Offset | Type | Field | Notes |
|-------:|------|-------|-------|
| 0x00 | u8[8] | `magic` | ASCII `"TZMOCHNK"` |
| 0x08 | u16 | `fmt_major` | breaking version; readers accept `{K, K−1}` (§7) |
| 0x0A | u16 | `fmt_minor` | additive version; unknown-minor readers ignore unknown sections |
| 0x0C | u32 | `flags` | see §3.2 |
| 0x10 | u64 | `v_begin` | first global-internal vertex id in this chunk (inclusive) |
| 0x18 | u64 | `v_end` | last + 1 (exclusive); `v_end−v_begin` = local vertex count |
| 0x20 | u64 | `edge_count` | edges stored in this chunk |
| 0x28 | u64 | `t_begin` | min edge timestamp, or `0` if not temporal |
| 0x30 | u64 | `t_end` | max edge timestamp + 1, or `0` |
| 0x38 | u64 | `generation` | the epoch that wrote this chunk (lineage; migration ordering) |
| 0x40 | u16 | `section_count` | number of entries in the section table |
| 0x42 | u16 | `chunk_type` | `0`=pairwise-CSR, `1`=hypergraph-incidence, `2`=simplicial. v1 specifies `0`; `1/2` reserved |
| 0x44 | u32 | `header_crc` | xxHash3-32 of bytes `[0x00, 0x44)` |
| 0x48 | Section[] | `section_table` | `section_count` entries, 24 B each (§3.3) |
| … | u8[] | pad | zero-pad to the 2 MB boundary; first data section starts at 0x200000 |

### 3.2 `flags` bitfield

| Bit | Name | Meaning |
|----:|------|---------|
| 0 | `BIG_ENDIAN` | set ⇒ file is big-endian. v1 readers **reject** with `TZM_ERR_ENDIAN` (§8) |
| 1 | `ID_WIDTH_64` | vertex-id width: set ⇒ u64, clear ⇒ u32 (ADR-0002) |
| 2 | `OFF_WIDTH_64` | degree/offset width: set ⇒ u64, clear ⇒ u32 — **independent** of bit 1 (de Bruijn) |
| 3 | `WEIGHTED` | a weights section is present |
| 4 | `TEMPORAL` | an edge-time section is present |
| 5 | `STABLE_EDGE` | a positional→stable-edge permutation section is present (ADR-0003) |
| 6 | `SIMPLE` | topology satisfies the no-multi-edge/no-self-loop constraint (`SimpleGraphView`) |
| 7 | `COMPRESSED` | neighbor section uses the WebGraph-style codec, not plain arrays |
| 8 | `HAS_CSC` | an in-neighbor (CSC) mirror section is present (`BidirectionalGraphView`) |
| 9–31 | — | reserved, must be 0 |

The low byte mirrors the runtime **capability bitmask** of ADR-0023 §5 (`tzm_graph_caps`): the
on-disk flags and the in-memory capabilities are the same lattice, one persisted and one live.

### 3.3 Section-table entry (24 B)

| Offset | Type | Field |
|-------:|------|-------|
| +0x00 | u32 | `kind` (see §3.4) |
| +0x04 | u32 | `crc` — xxHash3-32 of the section's bytes |
| +0x08 | u64 | `file_offset` — 2 MB-aligned |
| +0x10 | u64 | `byte_length` |

A reader iterates the table, maps/checks the sections it recognizes, and **skips unknown
`kind`s** — the forward-compatibility mechanism (a K reader tolerates a K same-major file that
added sections).

### 3.4 Section kinds (pairwise/`chunk_type=0`)

| `kind` | Name | Contents |
|-------:|------|----------|
| 1 | `OFFSETS` | `(v_end−v_begin)+1` offsets (width per `OFF_WIDTH_64`); CSR row pointers into `NEIGHBORS` |
| 2 | `NEIGHBORS` | `edge_count` neighbor ids (width per `ID_WIDTH_64`), each vertex's run **sorted ascending, tombstone-free** — this is what makes a mapped chunk a `ContiguousGraphView` (ADR-0023) |
| 3 | `WEIGHTS` | `edge_count` weights, index-aligned with `NEIGHBORS`; element type in the section's first 4 B (Arrow type id) |
| 4 | `EDGE_TIME` | `edge_count` timestamps (u64), index-aligned with `NEIGHBORS` |
| 5 | `CSC_OFFSETS` / 6 `CSC_NEIGHBORS` | the in-neighbor mirror (present iff `HAS_CSC`) |
| 7 | `EDGE_PERM` | positional→stable-edge-id map (present iff `STABLE_EDGE`) |
| 8 | `VPROP` | vertex property columns: an Arrow IPC record-batch covering `[v_begin, v_end)` |
| 9 | `EPROP` | edge property columns: an Arrow IPC record-batch covering this chunk's edges |
| 10 | `COMPRESSED_ADJ` | WebGraph-style compressed adjacency (present instead of 1+2 iff `COMPRESSED`) + its decode index |

Sections 8/9 are **Arrow IPC** buffers verbatim (ADR-0005: don't invent a type system; Arrow is
the interchange), so property columns are zero-copy into pyarrow/pandas and interop is free.

### 3.5 Manifest (`manifest-NNNNNN.mf`)

| Offset | Type | Field | Notes |
|-------:|------|-------|-------|
| 0x00 | u8[8] | `magic` | `"TZMOMNFS"` |
| 0x08 | u16 | `fmt_major` | / u16 `fmt_minor` at 0x0A |
| 0x0C | u64 | `generation` | monotonic epoch id |
| 0x14 | u64 | `num_vertices` (n) | graph-global |
| 0x1C | u64 | `num_edges` (m) | graph-global |
| 0x24 | u32 | `graph_flags` | directed/undirected, arity tier, global capability bits (§3.2 superset) |
| 0x28 | u32 | `chunk_count` |
| 0x2C | u64 | `dict_ref` | offset to the dictionary-artifact descriptor, or 0 |
| 0x34 | u32 | `manifest_crc` | xxHash3-32 of the manifest body |
| 0x38 | ChunkRef[] | `chunks` | `chunk_count` entries (§3.6) |

### 3.6 Chunk reference (in the manifest)

`{ chunk_id: u64, filename: pstring, v_begin: u64, v_end: u64, t_begin: u64, t_end: u64,
edge_count: u64, fmt_major: u16, flags: u32, file_crc: u32 }`

The manifest records each chunk's `fmt_major` so a reader **dispatches version handling
per-chunk** — a graph may transiently hold mixed-generation chunks during a migration (§7). Chunks
are non-overlapping and sorted by `v_begin` (binary-search vertex → chunk); temporal graphs may
carry multiple chunks per vertex range distinguished by `t_*` (whole-chunk window concatenation,
audit 16).

---

## 4. Alignment (audit 3) — a format property, not a runtime tweak

Every large mappable section (`kind` 1–10) begins at a **2 MB-aligned** file offset, and chunk
files are padded so a section can be mapped with `MADV_HUGEPAGE` / `MAP_HUGETLB`. A 40 GB neighbor
array under 4 KB pages TLB-thrashes *by design*; 2 MB huge pages are the fix, and because the
padding is baked into the bytes, the property holds for anyone who mmaps the file, in any language,
without knowing our runtime. The superblock + section table live in the first 2 MB; data starts at
`0x200000`. This costs ≤2 MB per chunk of padding — negligible against multi-GB chunks, and chunks
are sized (default target ~256 MB–2 GB of edges) so padding is well under 1%.

---

## 5. Integrity and authenticity (§14.7)

- **Corruption detection:** xxHash3-32 per section + per header/manifest (non-cryptographic, fast).
  Parsers assume hostile input (§14.7), so a reader **verifies the section CRC on first map / at
  the API boundary** — not in hot loops (principle 5). `open()` verifies the superblock and section
  table; each data section is verified on first touch or via an explicit `verify()`. A CRC mismatch
  is `TZM_ERR_CORRUPT` with the chunk id and section kind named (§14.11 actionable errors).
- **Authenticity** is *detached*: SLSA provenance + a sigstore signature over the manifest and
  chunk digests live in `PROVENANCE` (§14.4), not inside the mappable bytes — so signing never
  disturbs alignment or zero-copy, and an unsigned graph is byte-identical to a signed one minus
  the sidecar. Verification is a separate, optional step at load.
- **Bounds discipline:** every offset/length in the superblock and section table is validated
  against the file size before any map; a malformed section table cannot induce an out-of-bounds
  map. This is the first fuzz target (§14.2).

---

## 6. Reader algorithm (conformance)

```
open(graph_dir):
  sb = read(MANIFEST, 4KiB); check magic, fmt_major ∈ {K, K−1}, header_crc
  mf = read(sb.current_manifest); check magic, manifest_crc, fmt_major
  for ref in mf.chunks:
      require ref.fmt_major ∈ {K, K−1}           # else TZM_ERR_VERSION (name the chunk)
  return a GraphHandle over mf  (lazy; chunks mapped on demand)

neighbors(handle, u):                              # the GraphView hot path
  chunk = binary_search(mf.chunks, by v_begin, containing u)
  if chunk not mapped: mmap it (MADV_HUGEPAGE); verify OFFSETS+NEIGHBORS CRC once
  if chunk.flags.BIG_ENDIAN:  fail TZM_ERR_ENDIAN
  if chunk.flags.COMPRESSED:  return decode_run(chunk, u)       # → ContiguousGraphView after decode
  else:                       lo = OFFSETS[u−v_begin]; hi = OFFSETS[u−v_begin+1]
                              return span(NEIGHBORS[lo:hi])     # zero-copy, sorted, single run
```

A frozen chunk yields exactly one sorted, tombstone-free span per vertex → the mapped graph
satisfies `ContiguousGraphView` (ADR-0023), which is the on-disk↔in-memory identity the whole
design turns on: **`freeze()` writes this format, and mapping this format is `freeze()`'s inverse.**

---

## 7. Versioning, evolution, migration (CA-5, Lucene model)

- **`major.minor`.** *Minor* bumps are **additive only**: new optional sections, new flag bits, new
  chunk-type tags. A reader of the same major ignores unknown sections/flags (§3.3) — old code reads
  new files losslessly for what it understands.
- **Read-N−1 window.** A reader of major `K` reads `K` and `K−1`, never `K+1` (refuse with
  `TZM_ERR_VERSION`). Writers always emit current `K`.
- **Migrate-on-compaction.** A `K−1` chunk is rewritten as `K` when it is next compacted; no
  flag-day rewrite. The manifest's per-chunk `fmt_major` lets `K` and `K−1` chunks coexist during
  the transition. A standalone `tzomo migrate` performs cold, whole-graph conversion when a user
  wants to drop `K−1` support proactively.
- **Schema-evolution rules (stated in the v1 spec, FlatBuffers/Cap'n Proto discipline):**
  add-only; deprecate-don't-delete (a retired field keeps its slot, marked reserved, until a major
  bump reclaims it); **alignment invariants never change within a major**; section `kind` numbers
  are permanent (never reused for a different meaning).
- **Compatibility promise (published):** "current major reads its own and the previous major's
  files; minor versions are forward- and backward-compatible within a major; breaking changes bump
  the major, are announced, and are migrated on compaction or by the migrate tool — your files are
  never stranded."

---

## 8. Endianness — canonical little-endian in v1 (a real decision)

x86-64 and ARM (both tier-1, §13) are little-endian; WASM is little-endian; RISC-V is little-endian
in practice. **v1 stores canonical little-endian only.** A big-endian file (flag bit 0) is
**rejected on read** with `TZM_ERR_ENDIAN`, not transparently byte-swapped — because a swap path
would force a copy and destroy the mmap zero-copy property that CA-10/§7.11 depend on, to serve a
platform tier the project does not target. If a big-endian tier ever appears (none is planned), the
answer is an offline convert tool, never an in-hot-path swap. This keeps the one, fast, zero-copy
read path the *only* read path. Recorded in ADR-0024.

---

## 9. Scope for v1, and what is reserved

- **Fully specified in v1:** the pairwise tier (`chunk_type=0`) — the format above is complete for
  it, including weighted, temporal, CSC, stable-edge, compressed, and Arrow property variants.
- **Reserved, forward-compatible:** `chunk_type=1` (hypergraph incidence) and `2` (simplicial
  boundary operators) get their section-kind catalogs when those tiers harden (Phase 3). Because
  `chunk_type` and unknown `kind`s are already in the superblock/section-table, adding them is a
  *minor* bump — no break. The higher-order on-disk layout is a follow-on deepening (`design/NN`),
  not a v1 obligation.
- **Out of this format (by ADR-0009):** the mutable delta overlay, tombstone log, and WAL — the
  `stream`/`db` layers' persisted append-only logs, referenced by but not defined here.

---

## 10. Acceptance criteria (CI-enforceable)

1. **Round-trip identity.** `write(freeze(G))` then `open()` reproduces every kernel's output
   bit-for-bit (BFS, PageRank, triangle count) versus in-RAM `G`. Cross-checked in CI.
2. **mmap zero-copy.** Opening a chunk and iterating neighbors performs **no allocation and no
   copy** for the plain (non-compressed) path — asserted by an allocator hook in the test.
3. **Huge-page mapping.** Sections are 2 MB-aligned; a test maps a ≥2 GB neighbor section with
   `MADV_HUGEPAGE` and asserts the alignment invariant holds for every section in a generated chunk.
4. **Fuzz the reader.** `open()` + section mapping is a continuous fuzz target (§14.2); no crafted
   file induces an OOB read, over-long map, or UB — it must fail with a named `TZM_ERR_*`.
5. **Version dispatch.** A synthetic `K−1` chunk in a `K` manifest reads correctly; a `K+1` chunk
   is refused with `TZM_ERR_VERSION` naming the chunk.
6. **Corruption caught at the boundary.** A single flipped byte in any section is detected by CRC on
   first map and reported as `TZM_ERR_CORRUPT` with chunk id + section kind — never silently read.
7. **Atomic epoch swap.** A crash injected between chunk fsync and `MANIFEST` rename leaves the
   previous epoch fully readable (recovery test).

---

## 11. Proposed amendments (the delta)

- **New ADR-0024** — the on-disk format decision (§0): container-of-chunks + manifest; frozen/
  contiguous form only; canonical little-endian; read-N−1 with migrate-on-compaction; v1 pairwise
  complete, higher-order reserved.
- **SOW §14.5 amendment** — replace the requirements list's forward-reference with a pointer to
  this spec + ADR-0024, and state the container model and the "frozen-form-only" boundary as
  ratified.
- **System register** — X.2 On-disk format moves **K → S**; C6.4 Format I/O and L0.10 out-of-core
  gain their defining reference.
- **v0.9 change-log entry.**

No frozen decision moved; no §6 exclusion touched. Deepens §14.5 and realizes ADR-0007/0008/0023
on disk.
