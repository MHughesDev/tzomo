# ADR-0018: Project name: Tzomo

## Status
Accepted (2026-07-25, SOW v0.7)

## Context and Problem Statement
Four prior candidates collided (Spectra = the C++ eigensolver we replace; Osmium = OSM C++ library; Fiedler = handle taken + direction abandoned; Tzomet = Israeli political party dominates search).

## Decision Drivers
- Software-namespace cleanliness verified by search (2026-07-25: only a niggun, an art studio, a music track)
- Smooth, short, not a person's name
- Meaning: coined from Hebrew tzomet, junction - the word for a network node

## Considered Options
- Tzomet
- Zometh
- Tzoma
- Tzomo

## Decision Outcome
Chosen option: **Tzomo (TSOH-moh; ZOH-moh acceptable). Conventions: tzomo:: namespace, tzm_ C prefix, tzomo-* modules, tzomo on PyPI**.

Keeps the junction root and sound while escaping the political search space; verified clean for software.

### Consequences
- Good: distinctive, meaningful, clean
- Bad: initial ts- digraph will be anglicized (accepted)
