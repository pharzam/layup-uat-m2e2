# 0011. Build with go build only

Date: 2026-10-22

## Status

Accepted

## Context

tally is written in Go with the standard library only
([NFR-018](../spec/SPEC-0001-tally.md#nfr-018--go),
[NFR-019](../spec/SPEC-0001-tally.md#nfr-019--the-standard-library-only)), and
`go build` builds the binary with no other build tool
([NFR-020](../spec/SPEC-0001-tally.md#nfr-020--the-go-toolchain)). Two options were
filtered against the requirements, the facts, the invariants of
[`guardrails.md`](../guardrails.md) and the ADRs 0001 to 0009: `go-build-only` and
`make-wrapper`. Only `go-build-only` passed every constraint.

## Decision

We will build tally with the Go toolchain alone: `go build` builds the binary, and
no Makefile, script or other build tool stands between the developer and it.

We reject `make-wrapper`, a Makefile around the Go commands: it fails NFR-020 and
the fact behind it (F-3), since it makes a second build tool necessary.

## Consequences

- Building needs only an installed Go toolchain.
- The Go build, lint and test commands of each test level are still to be named in
  [`tests/test-levels.md`](../tests/test-levels.md); this record names no command
  beyond `go build`.
