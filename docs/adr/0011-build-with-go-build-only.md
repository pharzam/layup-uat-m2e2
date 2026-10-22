# 0011. Build tally with go build only

Date: 2026-10-22

## Status

Accepted

## Context

This is a decision of the product, under
[ADR-0010](0010-record-product-architecture-decisions.md). tally is written in Go,
uses the standard library only and is built with the Go toolchain: NFR-018, NFR-019
and NFR-020 of [PRD-0001](../prd/PRD-0001-tally.md). Two options for the build were
filtered against the requirements, the facts, the invariants of
[`guardrails.md`](../guardrails.md) and ADR-0001 to ADR-0009: the Go toolchain's
`go build` alone, and a make wrapper around it. The make wrapper fails NFR-020 and
the fact behind it (F-3 of the filter), because it adds a build tool that the Go
toolchain does not hold. `go build` alone passes every constraint.

## Decision

We will build tally with the Go toolchain's `go build` only. The repository has a
`go.mod` with no `require` directive, and no Makefile, build script or other build
tool. Every import is a package of the Go standard library.

We reject a make wrapper: it adds a second build tool, which NFR-020 excludes.

## Consequences

- One command builds the binary. The package path that `go build` names depends on
  the module layout, which the bet decides (see
  [SPEC-0002](../spec/SPEC-0002-tally-architecture.md#module-layout)).
- The Go version line of `go.mod` and the module path are not settled here; the
  task that writes `go.mod` takes them from a fact or asks the idea owner.
- The build, lint and test commands of each test level are still the markers of
  [`tests/test-levels.md`](../tests/test-levels.md); this record names no command
  for them.
