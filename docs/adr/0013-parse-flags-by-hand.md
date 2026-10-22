# 0013. Parse the flags by hand

Date: 2026-10-22

## Status

Accepted

## Context

This is a decision of the product, under
[ADR-0010](0010-record-product-architecture-decisions.md). The command line is
`tally [-l] [-w] [-c] [file...]` (REQ-002 of [PRD-0001](../prd/PRD-0001-tally.md)).
Combined flags such as `-lw` mean the separate flags (REQ-003); the argument `-` is a
file named `-`, never standard input (REQ-004); an unknown flag or wrong usage prints
a usage message on standard error and exits with 2 (REQ-005). Three options were
filtered:

- the standard library's `flag` package — it fails REQ-003 (R-003 of the filter),
  because it reads `-lw` as one flag named `lw`, and Inv-1 of
  [`guardrails.md`](../guardrails.md);
- a third-party getopt package — it fails NFR-019 and its fact (F-2 of the filter),
  because it is a dependency outside the standard library;
- a parser written by hand in the package `main` — it passes every constraint.

## Decision

We will parse the arguments with a parser written by hand in the package `main`. It
reads the arguments from left to right and decides each argument **before the first
file name** by these rules:

1. The argument `-` is a file name, and it is the first file name.
2. An argument that starts with `-` and has one character or more after it is a flag
   group. Each character after the `-` must be `l`, `w` or `c`, and selects the line,
   word or character count. Any other character, `-` included, is an unknown flag:
   tally prints the usage message on standard error, counts nothing, prints nothing
   on standard output and exits with 2.
3. Any other argument is a file name, and it is the first file name.

**After the first file name**, an argument that does not start with `-`, and the
argument `-` itself, is a file name. An argument that starts with `-` and has one
character or more after it, such as `-l` in `tally file -l`, is **not decided**: no
fact states whether it is a flag, a file name or wrong usage. This record decides
nothing for it. The question goes to the idea owner on the issue, and the code task
writes no test and no behaviour for that position until a fact answers it.

The selected counts are a set: the order and the repetition of the flags do not
change the output, which always lists lines, words and characters in that order
(REQ-012, Inv-1). With no flag, all three counts are selected (REQ-007).

We reject the `flag` package, which cannot read combined flags, and a getopt
package, which adds a third-party dependency.

## Consequences

- The parser is small and has its own unit tests: `-lw`, `-l -w`, `-wl`, `-x`, `--`
  and `-` before a file name.
- The code task for the parser carries one open question, the position after the
  first file name, and closes it only with a fact.
- The usage message's wording is not a fact; the code task takes it from the
  synopsis of REQ-002.
