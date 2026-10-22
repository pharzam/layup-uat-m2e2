# 0012. Count with three functions over a teed reader

Date: 2026-10-22

## Status

Accepted

## Context

This is a decision of the product, under
[ADR-0010](0010-record-product-architecture-decisions.md). NFR-021 of
[PRD-0001](../prd/PRD-0001-tally.md) requires a package `count`, separate from the
package `main`, that holds the functions that count the lines, the words and the
characters of a reader. REQ-008 requires tally to count standard input, which can be
read once only. Three options for the interface of `count` were filtered:

- one combined function that returns the three counts — it fails NFR-021, which
  asks for the functions that count the lines, the words and the characters;
- three functions, each of which reads the source again — it fails REQ-008 (R-008
  of the filter), because standard input cannot be read a second time;
- three functions over a teed reader — it passes every constraint.

## Decision

We will export three functions from the package `count`, one for each count:

```go
func Lines(r io.Reader) (int64, error)
func Words(r io.Reader) (int64, error)
func Chars(r io.Reader) (int64, error)
```

Each function reads `r` to its end and applies one rule of
[SPEC-0001](../spec/SPEC-0001-tally.md): `Lines` REQ-009, `Words` REQ-010 and
`Chars` REQ-011. Each function returns an error when a read fails or when the text
is not valid UTF-8 ([ADR-0015](0015-validate-utf8-in-the-stream.md)).

The command reads each source once, a named file or standard input. It tees that one
stream to the functions of the counts that the flags select, so each selected
function reads the same bytes, and no source is opened or read twice.

We reject one combined function, which NFR-021 does not allow, and three functions
that each read the source again, which cannot count standard input.

## Consequences

- Each rule has its own function and its own unit tests, which the coverage target
  of the package `count` measures.
- The tee makes the command run up to three readers over one stream. How the tee is
  built (for example `io.Pipe` with `io.MultiWriter`) is an implementation choice of
  the code task, inside the standard library.
- With one flag, the tee has one reader; with no flag, it has three.
