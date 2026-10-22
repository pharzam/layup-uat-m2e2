# 0012. Count through three functions over a teed reader

Date: 2026-10-22

## Status

Accepted

## Context

The package `count` exports the functions that count the lines, the words and the
characters of an `io.Reader`, and the command calls them
([NFR-021](../spec/SPEC-0001-tally.md#nfr-021--the-package-count)). Standard input
is read when no file is named
([REQ-008](../spec/SPEC-0001-tally.md#req-008--standard-input)), and standard input
can be read only once. Three options were filtered: `three-functions-teed-reader`,
`three-functions-reread-source` and `single-combined-function`. Only
`three-functions-teed-reader` passed every constraint.

## Decision

We will export three functions from the package `count` — `Lines`, `Words` and
`Chars` — each of which counts one quantity of an `io.Reader`, by the rules of
REQ-009, REQ-010 and REQ-011. The command reads its source once and tees the same
bytes to the reader of each selected function, so every source, standard input
included, is read a single time.

We reject `three-functions-reread-source`, which opens the source again for each
function: it fails REQ-008, since standard input cannot be read twice. We reject
`single-combined-function`, one function that returns all three counts: it fails
NFR-021, which asks for the functions that count the lines, the words and the
characters.

## Consequences

- Each function is tested alone against a reader, which serves the coverage target
  of the package `count`.
- The command, not the package `count`, owns the tee and the single read of the
  source.
- A function that is not selected by a flag need not be fed.
