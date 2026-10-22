# 0015. Validate UTF-8 in the stream and print the row after the end of the file

Date: 2026-10-22

## Status

Accepted

## Context

This is a decision of the product, under
[ADR-0010](0010-record-product-architecture-decisions.md). REQ-016 of
[PRD-0001](../prd/PRD-0001-tally.md) says that a file that is not valid UTF-8 gets a
message on standard error, no row and nothing in the total, and the exit code 1 at
the end; the check happens while the file is read, and rows already printed for
earlier files stay. NFR-023 says that a file is not text exactly when it is not valid
UTF-8. Three options were filtered:

- print the row while the file is still being read — it fails Inv-3 of
  [`guardrails.md`](../guardrails.md), because an invalid byte late in the file
  leaves a row that must not exist;
- replace each invalid sequence with U+FFFD and count on — it fails NFR-023, its
  fact (F-11 of the filter) and Inv-3, because it counts a file that is not text;
- validate in the stream and print the row only after the end of the file — it
  passes every constraint.

## Decision

We will validate the UTF-8 of each source while the counting functions of
[ADR-0012](0012-count-with-three-functions-over-a-teed-reader.md) read it, in the
stream of [ADR-0014](0014-stream-through-a-fixed-buffer.md). Each counting function
returns an error at the first invalid sequence, and at the end of the stream when it
ends inside a sequence. The command prints the row of a file only after every
selected function has reached the end of the file with no error, and only then adds
the counts to the total. When a function returns the UTF-8 error, the command prints
a message with the file name on standard error, prints no row, adds nothing to the
total, goes on with the next file, and exits with 1 at the end.

We reject printing while reading, which can leave a row for an invalid file, and
replacing invalid sequences, which counts a file that is not text.

## Consequences

- Rows of earlier files stay on standard output; the row of the invalid file never
  appears.
- The pitfall "A row printed before validation fails" in
  [`guardrails.md`](../guardrails.md) §2 names the test: an invalid byte at the end
  of a file gives no row and an unchanged total.
- A byte order mark at the start of a file is valid UTF-8 and counts as one
  character (REQ-011).
