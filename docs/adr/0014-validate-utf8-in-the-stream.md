# 0014. Validate UTF-8 in the stream, print after the end

Date: 2026-10-23

## Status

Accepted

## Context

A file that is not valid UTF-8 gets a message on standard error with its name, no
row, nothing added to the total, and the exit code 1 at the end; the other files
are still counted and the rows already printed stay. The check happens while the
file is read, before its row would be printed
([REQ-016 in PRD-0001](../prd/PRD-0001-tally.md); F-0003#21, Q-003 in
[F-0001](../facts/F-0001-setup-answers.md)). The options table of UTF-8 validation
held three options. The filter failed `print-row-while-reading` on Inv-3 in
[`guardrails.md`](../guardrails.md), because a row printed before the end of the
file can belong to a file found invalid later, and failed
`replace-invalid-with-fffd` on NFR-023, F-0003#11 and Inv-3, because replacing an
invalid byte counts a file that is not text. `validate-in-stream-print-after-eof`
passed every constraint.

## Decision

We will check that each source is valid UTF-8 during the same single read that
counts it, as each buffer arrives, with the help of the package `unicode/utf8`.
An incomplete sequence at the end of one read is carried to the next read; an
incomplete sequence at the end of the source is invalid. tally prints the row of a
source, and adds its counts to the total, only after it has read the source to its
end and found it valid. When the source is invalid, tally prints the message on
standard error, prints no row, adds nothing to the total, records the exit code 1
and goes on with the next file. The architecture is in
[SPEC-0002](../spec/SPEC-0002-tally-architecture.md#validating-the-encoding).

We reject `print-row-while-reading`, which fails Inv-3, and
`replace-invalid-with-fffd`, which fails NFR-023.

## Consequences

- No row and no part of the total ever comes from an invalid file, wherever the
  invalid byte sits in it.
- Validation adds no second read of the file and no copy of it in memory, so it
  keeps the bound of [ADR-0013](0013-stream-through-a-fixed-buffer.md).
- No count of an invalid source is printed, so tally may stop reading it at its
  first invalid byte; the message, the missing row and the exit code are the same
  either way.
