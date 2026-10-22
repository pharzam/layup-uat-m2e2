# T-yzst — Write the specification of tally

| Field | Value |
| ----- | ----- |
| Task | `T-yzst` |
| Issue | #3 |
| Kind | specification |
| Size | small |
| Requirements | REQ-001 to REQ-017, NFR-018 to NFR-024 |
| Opened | 2026-10-22 |

## Scope

Write the requirements of tally and their specification from the revised draft of
the requirements and its facts: the problem statement (F-0003, rendered at the
first bet), the marker answers ([F-0002](../facts/F-0002-marker-answers.md)) and the
idea owner's answers Q-001 to Q-014. The interfaces are specified only, not built:
the command line `tally [-l] [-w] [-c] [file...]` with the exit codes 0, 1 and 2,
and a Go package `count` whose functions count the lines, words and characters of
an `io.Reader`. No new dependency.

## Plan

1. Write [PRD-0001](../prd/PRD-0001-tally.md): one row per requirement ID in §6 and
   §7, an acceptance criterion and a traceability row for each, the success
   metrics in §8 and the phases in §9. Keep the IDs of the draft; leave MoSCoW and
   Phase empty for the first bet.
2. Write [SPEC-0001](../spec/SPEC-0001-tally.md): one section per requirement ID.
3. Update every document the change leaves stale: the project status in
   [onboarding](../onboarding-for-engineers.md), the PRD index in
   [`prd/README.md`](../prd/README.md), and a gate pitfall in
   [`guardrails.md`](../guardrails.md) for the empty priority cells.

The plan review approved the plan (architecture and scope only). No build test is
required at this step.

## Definition of Done

- [x] Each of the 24 requirement IDs has one requirement row, one acceptance row and
  one traceability row in PRD-0001, citing its facts and F-0002.
- [x] Each success-criterion row of the draft has one success-metric row.
- [x] Each requirement ID has one non-empty section in SPEC-0001.
- [x] No document is left stale; no file under `docs/facts/` is written.

## Evidence

- `sh docs/prd/prd-lint.sh` fails only with `MoSCoW "" not in Must|Should|Could|Won't`
  and `has no phase value`, for each of the 24 requirements, as intended until the
  first bet fills those cells. It reports no other failure.
- The second attempt also fixes the finding of the first: the onboarding status no
  longer says that no PRD exists.

## Verdict

Pending the review rounds.
