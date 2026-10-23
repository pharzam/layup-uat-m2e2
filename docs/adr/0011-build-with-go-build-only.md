# 0011. Build with go build only

Date: 2026-10-23

## Status

Accepted

## Context

tally is written in Go with the standard library only and is built with the Go
toolchain ([NFR-018 to NFR-020](../prd/PRD-0001-tally.md)). The options table of
the build tool held two options: `go-build-only` and `make-wrapper`. The filter
failed `make-wrapper` on NFR-020 and on the fact F-0003#3 ("built with the Go
toolchain"), because a makefile adds a build tool that is not the Go toolchain.
`go-build-only` passed every constraint.

## Decision

We will build tally with the Go toolchain alone: `go build` builds the binary, and
no makefile, script or other build tool is part of the build. The architecture is
in [SPEC-0002](../spec/SPEC-0002-tally-architecture.md#build).

We reject `make-wrapper`: it fails NFR-020.

## Consequences

- A person with the Go toolchain builds tally with one command and no other tool.
- The exact package path that `go build` names depends on the module layout, which
  is pending the bet in SPEC-0002; this record does not fix it.
- The Go build, lint and test commands of each test level are still to be named in
  `docs/tests/test-levels.md`; this record does not name them.
