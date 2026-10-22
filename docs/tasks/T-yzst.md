# T-yzst — Specify tally

| Field | Value |
| ----- | ----- |
| Task | T-yzst |
| Issue | #3 |
| Kind | specification |
| Requirements | REQ-001 to REQ-017, NFR-018 to NFR-024 |
| Opened | 2026-10-22 |

## Summary

Write tally's product requirements document and specification from the revised
draft, the problem statement (F-0003) and the idea owner's answers (Q-001 to
Q-014). The work specifies; it builds no Go code.

## Plan

- **Size:** small.
- **Open questions:** none.
- **New interfaces (specified only, not built):** the command line
  `tally [-l] [-w] [-c] [file...]` with the exit codes 0, 1 and 2, and a Go
  package `count` whose functions count the lines, words and characters of an
  `io.Reader`.
- **New dependencies:** none.
- **Tests:** none at this step; each acceptance criterion in the PRD is the test a
  later build task writes first.

The plan review approved the plan (architecture and scope only).

## Outputs

- [`docs/prd/PRD-0001-tally.md`](../prd/PRD-0001-tally.md) — the requirements, their
  facts and acceptance criteria, the success metrics, the phases and the
  traceability matrix. The MoSCoW and Phase cells stay empty until the first bet.
- [`docs/spec/tally.md`](../spec/tally.md) — one section for each requirement ID.

## Acceptance criteria

- [x] One PRD row, one acceptance row and one matrix row for each of REQ-001 to REQ-017 and NFR-018 to NFR-024, with the IDs of the draft.
- [x] Each Facts cell cites the facts the row covers and F-0002.
- [x] One success-metric row for each success criterion of the draft.
- [x] One specification section for each requirement ID.
- [x] Nothing written under `docs/facts/`.

## Verdict

Pending review.
