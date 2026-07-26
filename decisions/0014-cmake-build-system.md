# ADR-0014: CMake as the build system

## Status
Accepted (2026-07-25, SOW v0.7)

## Context and Problem Statement
Consumers must vendor the library (FetchContent/find_package); vcpkg and conda-forge integration must be native.

## Decision Drivers
- Embedder friction violates principle 9
- Ecosystem ubiquity for C++ libraries

## Considered Options
- CMake
- Meson
- Bazel

## Decision Outcome
Chosen option: **CMake**.

Every alternative costs embedder friction; CMake is the lingua franca whatever its aesthetics.

### Consequences
- Good: zero-friction embedding and packaging
- Bad: CMake is CMake
