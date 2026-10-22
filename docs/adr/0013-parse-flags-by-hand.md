# 0013. Parse flags by hand

Date: 2026-10-22

## Status

Accepted

## Context

The command line is `tally [-l] [-w] [-c] [file...]`
([REQ-002](../spec/SPEC-0001-tally.md#req-002--the-command-line)). Flags combine in
one argument, so `-lw` means `-l -w`
([REQ-003](../spec/SPEC-0001-tally.md#req-003--combined-flags)); the argument `-`
names a file called `-`
([REQ-004](../spec/SPEC-0001-tally.md#req-004--the-name---is-a-file)); wrong usage
exits with 2 ([REQ-005](../spec/SPEC-0001-tally.md#req-005--usage-errors)). Three
options were filtered: `hand-written-parser`, `stdlib-flag-package` and
`third-party-getopt`. Only `hand-written-parser` passed every constraint.

## Decision

We will parse the arguments with a parser written in tally's package `main`. An
argument that starts with `-` and holds more than that one character is a flag
group: each of its characters after the `-` must be `l`, `w` or `c`, and any other
character is a usage error. The argument `-` alone is a file name, and so is every
argument that does not start with `-`.

We reject `stdlib-flag-package`, the standard library's package `flag`: it does
not read `-lw` as `-l -w`, so it fails REQ-003 and Inv-1. We reject
`third-party-getopt`: it adds a third-party dependency, which fails NFR-019 and the
fact behind it (F-2).

## Consequences

- The parser is small and in tally's own code, so its tests cover every rule of
  REQ-002 to REQ-005.
- Any rule not stated by the facts — for example an argument after a file name
  that starts with `-` — is a question for the idea owner, not a choice of the
  parser.
