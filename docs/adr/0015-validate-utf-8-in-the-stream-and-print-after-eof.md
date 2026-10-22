# 0015. Validate UTF-8 in the stream and print after EOF

Date: 2026-10-22

## Status

Accepted

## Context

A file that is not valid UTF-8 gets no row and adds nothing to the total; a message
on standard error names it, rows already printed stay, the other files are still
counted, and the exit code is 1
([REQ-016](../spec/SPEC-0001-tally.md#req-016--a-file-that-is-not-valid-utf-8),
Inv-3 of [`guardrails.md`](../guardrails.md)). The check happens while the file is
read, before its row would be printed (Q-003). Three options were filtered:
`validate-in-stream-print-after-eof`, `print-row-while-reading` and
`replace-invalid-with-fffd`. Only `validate-in-stream-print-after-eof` passed every
constraint.

## Decision

We will check the encoding of each chunk as it streams through the buffer of
[ADR-0014](0014-stream-through-a-fixed-buffer.md), carrying an incomplete sequence
at the end of a chunk into the next one. tally prints the row of a source only
after it reaches the end of that source with no invalid sequence found. At the
first invalid sequence it stops reading the source, prints the message on standard
error, prints no row for it, adds nothing to the total, and goes on with the next
source.

We reject `print-row-while-reading`, which prints counts before the end of the
file: an invalid byte late in the file would leave a row already printed, which
fails Inv-3. We reject `replace-invalid-with-fffd`, which counts an invalid byte as
U+FFFD: it counts a file that is not text, which fails NFR-023, F-11 and Inv-3.

## Consequences

- A row on standard output is always the row of a whole, valid source.
- The counts of a source are held until its end, which costs a few counters, not
  the source itself.
- An incomplete sequence at the very end of a source is invalid UTF-8.
