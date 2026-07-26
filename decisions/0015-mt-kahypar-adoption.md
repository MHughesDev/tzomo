# ADR-0015: Partitioning: adopt Mt-KaHyPar, differentiate on top

## Status
Accepted (2026-07-25, SOW v0.7)

## Context and Problem Statement
Domain Test 6 originally mandated a from-scratch MIT-clean partitioner (believing the KaHyPar family GPL); verification (2026-07-25) shows Mt-KaHyPar is MIT while sequential KaHyPar is GPLv3.

## Decision Drivers
- License compatibility (MIT platform)
- Partition quality determines distributed viability (audit 23: 5-20x communication volume)
- Phase 2 should not carry an unplanned multi-month build

## Considered Options
- Build from scratch
- Adopt Mt-KaHyPar as baseline + in-house differentiating layer

## Decision Outcome
Chosen option: **Adopt/vendor Mt-KaHyPar (MIT) as the baseline engine; build the differentiating layer in-house: spectral initial partitioning via the laplace module, streaming/incremental repartitioning, native layout/distribution integration**.

Inherit a decade of Karlsruhe engineering; spend obsession-hours on the spectral layer only we would build. hMETIS and sequential KaHyPar remain unusable.

### Consequences
- Good: months of critical-path work deleted
- Bad: an external dependency in a core-adjacent role; vendored and license-audited per SOW 14.8
