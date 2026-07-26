# ADR-0019: Amalgamated build target

## Status
Accepted (2026-07-25, SOW v0.8, CA-1)

## Context and Problem Statement
Embeddability drives adoption; the shortest time-to-first-success known is SQLite's amalgamation and the stb single-header pattern: add one or two files to any build, no package manager, no CMake required of the consumer.

## Decision Outcome
CMake generates an amalgamated `tzomo.h`/`tzomo.c` (and a single-header core subset) as a standard release artifact alongside normal packaging.

### Consequences
- Good: disproportionate adoption payoff for near-zero maintenance cost once the generator exists.
- Bad: the amalgamation generator is one more release artifact to test; include-order discipline required in source.
