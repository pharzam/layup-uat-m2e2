# 0015. Count with three functions over a teed reader

Date: 2026-10-23

## Status

Accepted

## Context

The package `count` is a package of the same Go module as the command, separate
from its package `main`, and holds the functions that count the lines, the words
and the characters of a reader; the coverage target of 90 percent applies to it
only ([NFR-021 in PRD-0001](../prd/PRD-0001-tally.md); F-0003#25, Q-010 in
[F-0001](../facts/F-0001-setup-answers.md)). With no file name tally reads standard
input (REQ-008), which can be read only once. The options table of the counting
interface held three options. The filter failed `single-combined-function` on
NFR-021, which asks for functions that count the lines, the words and the
characters, and failed `three-functions-reread-source` on REQ-008, the draft's R-008, because standard
input cannot be read a second or third time. `three-functions-teed-reader` passed
every constraint.

## Decision

We will give the package `count` three exported functions, `Lines`, `Words` and
`Chars`. Each takes an `io.Reader` and returns its count of lines, words or
characters by the rules of REQ-009, REQ-010 and REQ-011. The command reads each
source once, through the buffer of
[ADR-0013](0013-stream-through-a-fixed-buffer.md), and tees the bytes it reads to
one reader for each selected function, using only types of the standard library
(for example `io.Pipe`, `io.MultiWriter` and `io.TeeReader`). The architecture is
in [SPEC-0002](../spec/SPEC-0002-tally-architecture.md#the-package-count).

We reject `single-combined-function`, which fails NFR-021, and
`three-functions-reread-source`, which fails REQ-008.

## Consequences

- Each function has one job and can be unit-tested alone on a `strings.Reader`,
  which keeps the 90 percent coverage of the package `count` cheap to reach.
- A source is read once, so standard input is counted like a file.
- The tee makes the functions read in step. If they run in separate goroutines,
  the command must wait for all of them and must not leak one when another
  returns an error; the implementation task tests that case.
- The exact signatures, including the integer type and an error result, are left
  to the test-first implementation; this record fixes the three names, the
  `io.Reader` parameter and the single read.
