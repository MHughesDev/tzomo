# ADR-0020: Planner + wisdom component

## Status
Accepted (2026-07-25, SOW v0.8, CA-2)

## Context and Problem Statement
The workload×representation matrix (SOW §8) needs a selection mechanism. FFTW proved that measuring the actual machine beats modeling it, and that persisting measured plans ("wisdom") amortizes planning cost. OpenBLAS's runtime microarch dispatch is the same principle at kernel granularity.

## Decision Outcome
A named `tzomo-runtime` component: the **planner** measures representation/ordering/kernel/block choices on the actual machine; the **wisdom** cache persists plans across runs. v1 ships explicit mode plus the planner interface; measurement-driven automation lands in Phase 3. Starting block-size heuristic ~64K elements (ClickHouse calibration, CA-13). Autotuning knobs remain introspectable (SOW §14.10).

### Consequences
- Good: converts the selection problem from documentation into infrastructure; the endgame for automatic representation choice has a name and an interface from day one.
- Bad: plan caches are state — versioned, invalidated on hardware/library change; determinism modes must account for planned-vs-fixed execution.
