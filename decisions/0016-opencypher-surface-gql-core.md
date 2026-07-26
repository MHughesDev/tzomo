# ADR-0016: Query language: openCypher surface, GQL-aligned core

## Status
Accepted (2026-07-25, SOW v0.7)

## Context and Problem Statement
The DB layer must adopt a standard, not invent one; ISO GQL published 2024 but native implementations are only now appearing while Cypher remains what graph users write daily.

## Decision Drivers
- Kuzu-refugee continuity (Cypher dialect)
- ISO GQL is the long-run standard
- Migration must be additive, not a rewrite

## Considered Options
- GQL-first
- openCypher-first with GQL-aligned internals

## Decision Outcome
Chosen option: **openCypher-compatible query surface; internal semantics/IR aligned with ISO GQL so conformance is an additive Phase-4+ layer**.

Ride the de facto standard now, the de jure standard later, with one engine.

### Consequences
- Good: serves today's users and tomorrow's standard
- Bad: two dialects to document eventually
