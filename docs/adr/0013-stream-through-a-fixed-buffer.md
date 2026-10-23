# 0013. Stream each file through a fixed buffer

Date: 2026-10-23

## Status

Accepted

## Context

tally must count a file of 1 GiB with at most 64 MiB of peak heap, sampled during
the count (the peak-heap metric in §8 of [PRD-0001](../prd/PRD-0001-tally.md);
F-0003#24, Q-009 in [F-0001](../facts/F-0001-setup-answers.md)). The options table of the read strategy
held two options. The filter failed `read-whole-file` on R-026, the draft's ID of
that metric, because a file read whole into memory takes at least its own size of
heap. `stream-fixed-buffer` passed every constraint.

## Decision

We will read each file, and standard input, in order from start to end through one
buffer of fixed size, allocated once for the source and reused for each read. The
heap that a count uses does not grow with the size of the file. The size of the
buffer is not fixed by this record; the implementation sets it, and the peak-heap
test checks it. The architecture is in
[SPEC-0002](../spec/SPEC-0002-tally-architecture.md#reading-a-source).

We reject `read-whole-file`: it fails the peak-heap metric (R-026).

## Consequences

- The peak heap is bounded by the buffer and by the counters, not by the file.
- A multi-byte UTF-8 sequence can be split between two reads. The counting and
  the validation must carry an incomplete sequence from one read to the next; the
  pitfall "A UTF-8 sequence split across reads" in
  [`guardrails.md`](../guardrails.md#2-known-pitfalls--the-traps-specific-to-this-domain)
  names the test that catches it.
- A word can also be split between two reads, so the word counter keeps whether
  the last code point of a read was white space.
