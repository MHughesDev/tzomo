# External Systems Deep-Dive (Round 2 — the broader canon)

Date: 2026-07-25. Method: each system audited against the ten-dimension rubric (A performance
techniques, B API/ABI discipline, C testing culture, D benchmark honesty, E packaging/
embeddability, F adoption mechanics, G governance/sustainability, H monetization precedent,
I failure modes to refuse, J transferable architecture per Tzomo layer). Per system: lessons,
refusals, and amendment candidates (consolidated in candidate-amendments.md). Status: stable
engineering knowledge; items that may have moved recently are tagged CHECK for the radar.

---

## Storage engines

### RocksDB — dimensions A, J (chunked base's closest cousin)
The production LSM engine. Lessons: (1) **compaction is a scheduling problem, not a
subroutine** — RocksDB's history is largely a history of compaction pacing: rate limiters,
write stalls when compaction falls behind, explicit stall/slowdown thresholds. Our chunked
base (ADR-0008) will hit the identical dynamics at Domain-Test-3 ingest rates. (2) **Write
amplification is a first-class metric** they report and tune against — ours should be a
benchmark row, not a footnote. (3) Leveled vs. tiered compaction is a documented tradeoff
(read amp vs. write amp) — our per-chunk compaction needs the same explicit policy choice.
(4) Column families ≈ our property columns sharing one WAL — validating DB-on-top (ADR-0009).
Refusals: RocksDB's configuration surface (hundreds of knobs) is the canonical cautionary
tale — tuning complexity became its adoption tax. Tzomo policy: opinionated defaults,
introspectable knobs, no knob required for the common case.

### LMDB — dimensions A, B, I (the honest counterpoint to audit 19)
Single-writer mmap B-tree, copy-on-write pages, zero-copy reads, crash-safe by construction
(never overwrites live pages — the same immutability-buys-safety logic as our chunk
manifests). Lesson: audit 19's "mmap considered harmful" verdict is about *write-heavy and
random-fault* paths; LMDB proves the **read-mostly embedded mmap path can be world-class**
when pages are immutable and access is through a real structure. This *refines* our ruling:
the convenience mmap path over frozen chunks is not merely tolerated, it can be excellent —
because our chunks are immutable, exactly LMDB's precondition. Refusal: LMDB's rigid
map-size preallocation UX friction.

### ClickHouse — dimensions A, D, F
Vectorized columnar execution at block granularity (~64K rows/block: big enough to amortize
dispatch, small enough for cache) — a concrete calibration point for our binned-pass block
sizing. Benchmark marketing: **ClickBench** — a public, reproducible, multi-competitor
benchmark that competitors submit to — is the strongest existing proof that "the benchmark
site is the marketing site" (SOW §14.1's thesis, validated at company scale). Refusal:
their early benchmark-culture reputation for favorable configurations — our §14.1 fairness
policy exists precisely to never earn that reputation.

### SQLite (deeper pass) — dimensions B, C, E, G
Beyond the prior study-plan notes: (1) **The amalgamation** — the entire library shipped as
one .c file + one .h — is arguably the single greatest embeddability device in software
history; a decade of adoption comes from "add two files to your build." (2) Testing: aviation-
grade — 100% MC/DC coverage, multiple independent harnesses, tests-to-code ratio near 600:1;
unreachable solo but the *posture* transfers: release-blocking suites, fuzz-hours as a
published metric. (3) Long-term-support pledge (format supported to 2050) as an adoption
asset. (4) Governance: open-source but not open-contribution — a legitimate solo posture that
resolves the support-burden risk honestly. Refusal: none; SQLite is the north star.

## Formats

### Apache Arrow / Parquet — dimensions B, E, G
Lesson: **spec-first, multi-implementation** — the format is a normative document independent
of any codebase, which is why the ecosystem trusts it. Our mmap-native format (§14.5) is
single-implementation, but writing its spec *as if* others will implement it (they may) is
cheap discipline that forces precision. Parquet adds: explicit format versioning with feature
flags, and the lesson that footer-metadata design determines random-access economics.

### FlatBuffers / Cap'n Proto — dimensions A, B
Zero-copy access disciplines for on-disk/wire structures: offset-based access, alignment
rules as part of the schema, **schema evolution rules stated from v1** (add-only fields,
deprecation not deletion). Directly applicable to the chunk format: evolution rules are part
of the v1 spec, not a v2 retrofit. Cap'n Proto's "infinitely faster" honesty joke (zero
encode step) is also a model of truthful benchmark framing.

### Lucene — dimensions A, B, J
The original immutable-segment + background-merge architecture (our chunked base's oldest
ancestor, predating LSM terminology). Two transferable policies: (1) **merge policies as
pluggable strategy objects** (size-tiered etc.) — our compaction should be similarly
policy-pluggable; (2) **index back-compat window: current major reads previous major** — a
concrete, shippable answer for §14.5's compatibility promise (read N-1, migrate-on-compact).

## Numerics

### FFTW — dimensions A, B, J (the deepest single lesson of this round)
The planner/wisdom architecture: FFTW doesn't ship one FFT — it ships a *space* of algorithm
compositions plus a **planner** that measures the actual machine and a **wisdom** cache that
persists the plans. This is precisely the shape of our unresolved "v1 selects representations
explicitly; automatic later" (§8): representation choice × ordering choice × kernel variant ×
block size is a plan space; measurement beats modeling; persisted wisdom amortizes planning.
Amendment candidate: name the component now (planner + wisdom in tzomo-runtime), ship
explicit-mode v1 with the planner as the Phase-3 automation path. Also: FFTW's dual license
(GPL + commercial) funded it for decades — not our model, but the precedent that solver-class
software sustains on licensing is relevant to Phase-6 optionality.

### BLIS / OpenBLAS — dimensions A, J
BLIS's insight: the entire BLAS reduces to a handful of **microkernels** (tiny, register-level,
arch-specific) wrapped in portable macro-loops for blocking/packing. This is the mature
version of our audit-13 escape-hatch design: VLA-portable outer machinery + enumerated
fixed-width microkernels, formalized. OpenBLAS adds runtime microarch dispatch (detect CPU,
select kernel set at load) — which our blessed-instantiation policy (ADR-0013) should adopt
for the specialized set. Refusal: OpenBLAS's threading model conflicts with host applications
(the classic "who owns the threads" bug class) — reinforcing our explicit scheduler-handle
rule (no hidden global pool, ever).

### Eigen — dimensions B, I
Expression templates give beautiful APIs and are also the canonical compile-time-cost and
error-message horror story. Lesson for our C++ surface: ergonomics matter enormously to C++
adoption (Eigen won on API despite competition); refusal: template metaprogramming depth
that produces 400-line diagnostics — our concepts-first C++23 style and ADR-0013 erasure
policy are the guardrails, and error-message quality (§14.11) applies to *compile-time*
errors too.

### PETSc — dimensions B, G, I
The solver-ecosystem precedent: runtime-composable solver/preconditioner options serving a
40-year scientific user base. Lesson: options-database introspection (every knob queryable)
for our determinism/config surface. Refusal: the "everything configurable, nothing default"
UX and MPI-everywhere assumption — embedded-first is our identity, not an afterthought.

## Parallel runtimes

### oneTBB — dimensions A, B, J
Work-stealing done industrially. The transferable core: **composability discipline** — TBB
arenas exist because libraries that spawn their own worlds of threads destroy host
applications; a library-grade runtime must accept external arenas, bound its concurrency, and
never oversubscribe. This is our workspace/scheduler-handle rule (audit 5, §7) elevated to a
contract: *Tzomo embedded in an app with its own pool must be a polite citizen* — amendment
candidate: a written composability contract in the C ABI (caller-provided scheduler,
max-concurrency cap, no TLS surprises).

### Taskflow — dimensions B, F
Modern C++ task-graph API whose adoption came substantially from **API ergonomics + paper
trail** (published at conferences, benchmarked publicly) — the academic-engagement loop
(§15.5) demonstrated in our own language/decade.

## SIMD

### Highway — dimensions B, E
The production vector-length-agnostic SIMD layer (Google): tag-dispatch API, per-target
compilation with runtime dispatch, exactly the VLA-core model of our v0.5 decision. The
adopt-vs-build ADR should lean **adopt (vendored)** unless the prior-art read finds a
disqualifier: it is battle-tested across the exact ISA matrix we target (AVX-512/NEON/SVE,
RVV in progress — CHECK current RVV status), and principle 3 beats principle 9's
zero-dependency instinct here since vendoring preserves dependency-freedom for consumers.

## Compression

### zstd — dimensions B, D, F, H
The modern adoption masterclass: (1) **API tiering** — one-line simple API, then contexts,
then advanced parameters; each tier complete, nobody forced upward. Directly transferable to
our C ABI shape. (2) The **level dial** (including negative levels) — one integer controlling
a documented speed/ratio curve — is the UX model for our determinism/speed and quality/effort
knobs. (3) Dictionary training as a distinct feature created a new use-case category —
analogous to our wisdom/planner artifacts. (4) Benchmark tables that include competitors'
wins built the trust that drove adoption (principle 11 validated). (5) Meta stewardship of a
single-author project (Collet) is a Phase-6 arc precedent (hired, project thrives).

### LZ4 — dimensions A, F
Simplicity as the moat: one job, tiny surface, unbeatable at it. The scope-discipline
precedent for individual Tzomo modules — `sketch` or a codec should feel like LZ4, not like
a platform.

## Query engines

### Velox — dimensions B, J
Meta's composable execution-kernel library: not a database — a **library of vectorized
components that other systems embed** (Presto, Spark). Validates our layered catalog thesis
and sharpens it: `blas`/`algo`/`part` should each be embeddable *without the platform* —
kernel-granularity adoption is a real channel (a DB vendor embedding just our BFS/PageRank
kernels is a win, not a leak). Amendment candidate: state standalone-usability of compute
modules as a design requirement.

### Polars — dimensions F, I (the nearest adoption analog)
Rust dataframe engine that beat pandas on performance and won massive adoption — the closest
existing run of our Phase-1 play. Critical nuance: **Polars deliberately broke pandas API
compatibility** (clean expressions API) and won anyway, but slowly and with a migration tax;
the ecosystem *then* built compatibility shims. Our NetworkX shim strategy is the inverse
bet (compatibility first). Both work; the lesson is to be *loud and deliberate* about the
parity boundary (which §7.11/§6.8 already mandate) and to pair the shim with a native API
that is *better*, not just faster — the shim converts users, the native API keeps them.

## Adoption playbooks

### Astral (uv / ruff) — dimensions D, F (the modern playbook, step by step)
The definitive recent proof of Tzomo's Phase-1 shape: take a beloved-but-slow Python-world
tool, rewrite the core in a fast native language, ship **drop-in compatibility + a benchmark
chart + a migration story**, win the ecosystem in months. Extractable mechanics: (1) the
compatibility promise was explicit and versioned; (2) benchmarks were reproducible and
front-and-center; (3) speed was marketed in *user time* ("10–100× faster") not architecture
terms; (4) relentless issue-response velocity in the first year converted skeptics.
Refusal/watch: Astral is VC-funded — the mechanics transfer, the funding model does not
(ours is §15.1); their eventual monetization pressure is a radar item, not a template.

## Solo and small-team precedents

### curl — dimensions B, C, G, H
Three decades, one lead maintainer. Transferable: (1) the **support policy as a document**
(what's supported, what's not, how to report security issues) pre-empts the support-burden
risk; (2) security process maturity (CVE handling, bounty via sponsors) as reputation
infrastructure; (3) funding: support contracts + sponsorships sustaining a maintainer
full-time — the living proof of §15.1's tertiary layer; (4) "I decide" governance stated
kindly but firmly — the model for §15.4's permission-to-say-no.

### zlib — dimensions B, E
The eternal ABI: one C interface, unchanged for 30 years, inside effectively every device on
earth. The standard of boring-interface discipline our C ABI (tzm_) aspires to. Also the
cautionary CVE lesson: ubiquity makes every parser bug a world event — §14.7's posture is
not optional at the adoption levels we intend.

### stb libraries — dimension E
Single-header distribution as philosophy: for a class of consumers, "copy one file" beats
every package manager. Pairs with SQLite's amalgamation into one amendment candidate: an
**amalgamated build target** (tzomo.h + tzomo.c or single-header core subset) for trivial
embedding — extremely cheap to produce from CMake, disproportionate adoption payoff.

### Ninja — dimension I (scope discipline)
Did one thing (fast builds from generated files), refused everything else (no configure
language), and became universal infrastructure *because* of the refusals. The per-module
scope model — and the living argument for our §6 exclusions.

### fmt — dimensions B, F
Modern C++ API quality driving adoption hard enough to be standardized (std::format). Lessons:
compile-time checking as UX, header-only option lowering trial friction, and the
maintainer-to-standard arc as a career precedent for C++ library authors.

## Cautionary tales (dimension I concentrated)

### RethinkDB postmortem — the most important negative result in the file
A technically excellent database, engineering-led, beloved — dead. The postmortem's own
diagnosis: optimized for correctness/elegance while the market bought on a different axis
(MongoDB's worse-but-marketed developer experience and use-case timing); no sharp wedge;
VC clock forced outcomes. Transfers: (1) our no-VC slow-burn model *removes the clock* —
the single biggest structural difference; (2) technical superiority must be **experienced in
minutes** (the shim, the amalgamation, the benchmark site are exactly that delivery
mechanism); (3) one sharp wedge per phase (§11 already does this — the discipline is to
refuse a second simultaneous wedge).

### Redis / Elastic / Terraform license sagas — CHECK current states
Three rug-pulls (permissive→restrictive) that each spawned forks (Valkey, OpenSearch,
OpenTofu) and burned community trust; Redis later partially reversed (AGPL option — CHECK).
Tzomo's structural position: MIT + DCO (no CLA) means **we cannot rug-pull even if tempted**
— contributors' copyright makes relicensing impractical. Amendment candidate: say this out
loud — a one-paragraph license-permanence statement converts our legal structure into an
adoption asset, aimed exactly at the enterprises these sagas made wary.

### ZeroMQ — dimensions G, I
Brilliant API, governance chaos (founder exit, forks: nanomsg/nng). Lesson: the bus-factor
answer isn't only code legibility (§15.3) — it's a stated succession intent (§15.4 has it;
keep it).

## Distribution-layer notes (Phase 5 shelf)

### NCCL — dimension J
Collective-communication design: topology detection, ring/tree algorithm selection by message
size, stream-ordered semantics. Shelved to Phase 5 with one present-day implication already
in ADR-0010: the device model's collectives vocabulary should mirror NCCL's (all-reduce,
all-gather, reduce-scatter, broadcast) so the plugin maps cleanly.

---

## Cross-cutting synthesis

1. **The immutable-segment pattern is universal** (Lucene, LSM/RocksDB, LMDB's COW, Parquet
   row groups, our chunks) — we are on the main road of storage engineering; steal its
   accumulated policy vocabulary (merge policies, stall budgets, compat windows) rather than
   rediscovering it.
2. **Adoption is a delivery-time problem**: SQLite's amalgamation, zstd's one-liner, uv's
   drop-in, stb's copy-a-file — every durable win shortened time-to-first-success to minutes.
   Tzomo's versions: the shim, the amalgamation target, the one-line pip install, the
   benchmark site.
3. **Composability is a contract, not a vibe** (TBB arenas, OpenBLAS's threading bugs):
   libraries that assume they own the machine get evicted from serious applications.
4. **Measurement beats modeling** (FFTW, OpenBLAS dispatch): the planner/wisdom pattern is
   the endgame for our representation matrix.
5. **Honesty compounds** (zstd's tables, ClickBench, Cap'n Proto's joke) and its absence
   compounds too (early ClickHouse reputation) — principle 11 is commercially load-bearing.
6. **The clock kills more projects than the code** (RethinkDB vs. curl/SQLite): our funding
   structure is a design decision, and it is already made correctly.
