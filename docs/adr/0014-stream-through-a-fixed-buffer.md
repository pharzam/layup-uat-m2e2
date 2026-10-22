# 0014. Stream through a fixed buffer

Date: 2026-10-22

## Status

Accepted

## Context

A file of 1 GiB must be counted with at most 64 MiB of peak heap, measured by
sampling `runtime.ReadMemStats` `HeapAlloc` every 10 milliseconds
([PRD-0001 §8](../prd/PRD-0001-tally.md#8-success-metrics); R-026 of the draft).
Two options were filtered: `stream-fixed-buffer` and `read-whole-file`. Only
`stream-fixed-buffer` passed every constraint.

## Decision

We will read every source — a named file or standard input — in chunks through a
buffer of fixed size, allocated once per source, and count each chunk as it
arrives. The heap held by a count does not grow with the size of the source.

We reject `read-whole-file`, which reads the source into memory before counting:
a file of 1 GiB would take 1 GiB of heap, which fails R-026.

## Consequences

- The peak heap is bounded by the buffer and the counters, whatever the file size.
- A UTF-8 sequence or a run of white space can cross the end of a chunk, so each
  counter carries its state from one chunk to the next.
- The size of the buffer is an implementation constant, not fixed by this record.
