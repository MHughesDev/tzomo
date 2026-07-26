# ADR-0021: Composability contract

## Status
Accepted (2026-07-25, SOW v0.8, CA-3)

## Context and Problem Statement
Libraries that assume they own the machine get evicted from serious applications (OpenBLAS threading-conflict bug class; oneTBB grew arenas for exactly this reason). Tzomo is embedded infrastructure and must be a polite citizen inside hosts with their own runtimes.

## Decision Outcome
A written, public contract in the C ABI: caller-provided scheduler handles honored on every parallel path; max-concurrency cap; no thread-local surprises; no global pools; workspace objects make all scratch state explicit (audit 5).

### Consequences
- Good: embeddability in applications, services, and other runtimes without pathological interaction; testable (CI runs kernels under a 1-thread external arena).
- Bad: no convenience global parallelism — the simple-tier API must construct a default scheduler explicitly and document it.
