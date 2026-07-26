# Tzomo — Project Inception Package

Everything decided before the first commit. Generated 2026-07-25 from the inception
conversation; the SOW is the master document, at v0.8 with **zero open items**.

## Contents

- **SOW.md** — Statement of Work v0.8: vision, data model, full scope in/out, 13 frozen
  architectural decisions, workload matrix, domain pressure-test + 25-performance-audit
  records, deliverable catalog, phasing with 90-day plan, engineering process, sustainability,
  risk register, closed open-item register, change log (v0.1→v0.7).
- **PRINCIPLES.md** — the eleven principles + two tiebreakers (extracted; commit as-is).
- **decisions/** — 18 seed ADRs (MADR format), the §7 frozen decisions plus license, name,
  build system, partitioner, and query-language rulings. Drop into the repo unchanged
  (SOW §14.9 discipline: co-located with code, one decision per record).
- **research/**
  - `external-systems-log.md` — every incumbent, competitor, and ecosystem fact researched,
    status-flagged (VERIFIED / HIGH CONFIDENCE / CHECK), incl. the contested-Kuzu-vacuum
    landscape and the naming record.
  - `prior-art-study-plan.md` — the §14.3 four-week source-reading program with per-system
    extraction questions.
  - `frontier-implementation-targets.md` — the theory-practice-gap targets (`laplace` et al.).
  - `deep-dive-round2.md` — 24 systems from the broader canon (storage engines, numerics,
    runtimes, adoption playbooks, cautionary tales) audited against a ten-dimension rubric,
    with a cross-cutting synthesis.
  - `candidate-amendments.md` — 17 proposed SOW/ADR amendments harvested from the deep dive,
    tiered by value, **ratified in full** (applied as SOW v0.8; ADR-0019..0022).
- **registers/**
  - `assumptions.md` — landscape assumption register (§1B), re-verified annually (§14.14).
  - `risks.md` — risk register (§16).
  - `audit-record.md` — domain pressure tests + performance-audit series verdicts.
- **planning/**
  - `phase-0-90-day-plan.md` — phasing + the ninety-day plan.
  - `week-1-checklist.md` — execution-only; start here.
  - `funding-applications.md` — grants-first strategy with targets and prep notes.

## How to use

Week 1: follow `planning/week-1-checklist.md`. Copy PRINCIPLES.md and `decisions/` into the
repo on day 1. Everything else is reference; the SOW is the arbiter when scope questions arise
(§6 is written to be auditable). New decisions get new ADRs; the SOW changes only via its
change log.
