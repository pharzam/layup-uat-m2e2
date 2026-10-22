# 0014. Stream each file through a fixed buffer

Date: 2026-10-22

## Status

Accepted

## Context

This is a decision of the product, under
[ADR-0010](0010-record-product-architecture-decisions.md). The success metrics of
[PRD-0001](../prd/PRD-0001-tally.md) §8 hold a file of 1 GiB to at most 64 MiB of
peak heap, sampled during the count, and a file of 10 MiB to at most 2 seconds for
the whole process. Two options for reading a file were filtered: reading the whole
file into memory, and streaming it through a buffer of fixed size. Reading the whole
file fails the peak-heap metric (R-026 of the filter): a file of 1 GiB needs at least
1 GiB of heap. Streaming passes every constraint.

## Decision

We will read every source, a named file or standard input, as a stream through a
buffer of fixed size. The size is a constant of the code, independent of the size of
the file, and far below 64 MiB. The counting functions of
[ADR-0012](0012-count-with-three-functions-over-a-teed-reader.md) keep only their
counts and the state that spans two reads: the bytes of a UTF-8 sequence that a read
splits, and whether the last code point was white space or a line feed.

We reject reading the whole file, which breaks the peak-heap metric on a large file.

## Consequences

- The heap does not grow with the size of the file.
- A UTF-8 sequence can straddle two reads. The pitfall "A UTF-8 sequence split
  across reads" in [`guardrails.md`](../guardrails.md) §2 names the test: the same
  text through a reader that returns one byte at a time gives the same counts.
- The buffer size is not a frozen number; the code task picks it and the time and
  heap tests check it.
