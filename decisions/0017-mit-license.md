# ADR-0017: MIT license

## Status
Accepted (2026-07-25, SOW v0.7)

## Context and Problem Statement
Maximum-adoption permissive licensing for an infrastructure platform whose moat is execution velocity.

## Decision Drivers
- Adoption physics
- Solo project: legal simplicity
- Accepted risk: no explicit patent grant (logged in risk register)

## Considered Options
- MIT
- Apache-2.0 (patent grant)
- MPL-2.0

## Decision Outcome
Chosen option: **MIT**.

Owner decision; patent-grant gap accepted and logged rather than reopened. Forces permissive-only dependencies (which surfaced the partitioner issue and its resolution).

### Consequences
- Good: simplest possible adoption story
- Bad: no patent grant in patent-dense adjacent domains (EDA, codecs)
