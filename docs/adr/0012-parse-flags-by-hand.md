# 0012. Parse the flags by hand

Date: 2026-10-23

## Status

Accepted

## Context

The command is `tally [-l] [-w] [-c] [file...]`. Combined flags such as `-lw` mean
`-l -w`, the name `-` is a file named `-`, and an unknown flag or wrong usage
prints a usage message on standard error, counts nothing and exits with 2
([REQ-002 to REQ-006](../prd/PRD-0001-tally.md); F-0003#6, Q-002 in
[F-0001](../facts/F-0001-setup-answers.md)). The options table of flag parsing held
three options. The filter failed `stdlib-flag-package` on REQ-003, the draft's R-003 (the package
`flag` does not accept a combined `-lw`) and on Inv-1 in
[`guardrails.md`](../guardrails.md), and failed `third-party-getopt` on NFR-019
and F-0003#2 (the standard library only). `hand-written-parser` passed every
constraint.

## Decision

We will parse the command line with a parser written in the package `main`, using
the standard library only. The parser reads the arguments in order, and its rules
are:

1. An argument that is a `-` followed by one or more of the letters `l`, `w` and
   `c`, and that comes before every file name, selects each count it names: `-lw`
   selects the same counts as `-l -w`.
2. An argument that starts with `-`, has more than one character and comes before
   every file name, but holds a letter other than `l`, `w` and `c`, is a usage
   error: tally prints a usage message on standard error, prints nothing on
   standard output, counts nothing and exits with 2.
3. The argument `-` is a file name, never standard input.
4. Every other argument is a file name, in the order given.
5. With no flag, all three counts are selected. Whatever the order of the flags,
   a row prints the selected counts in the order lines, words, characters.

The architecture is in
[SPEC-0002](../spec/SPEC-0002-tally-architecture.md#the-command-line-parser).

We reject `stdlib-flag-package`, which fails REQ-003, and `third-party-getopt`, which
fails NFR-019.

## Consequences

- Combined flags, the literal `-` and the exit code 2 are under tally's own code
  and its own unit tests.
- The facts do not say what an argument that starts with `-` means when it comes
  after a file name (for example `tally file -l`), nor what `--` means. This record
  decides neither case. The implementation task asks the idea owner on its issue
  before it writes code for them.
- The parser is small, but it is product code, so its tests are written first like
  any other.
