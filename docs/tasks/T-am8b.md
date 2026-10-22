# T-am8b — Write the architecture of tally

| Field | Value |
| ----- | ----- |
| Task | `T-am8b` |
| Issue | #6 |
| Kind | architecture |
| Size | large |
| Requirements | REQ-001 to REQ-017, NFR-018 to NFR-024 |
| Opened | 2026-10-22 |

## Scope

Write the architecture of tally from the options tables of its seven decisions and
the survivors of their constraint filter, with the revised draft of the
requirements and its facts. A decision with one survivor is written as decided, in
an ADR; a decision with more than one survivor is written under "Decision pending
the bet" and decides nothing. No Go code, MoSCoW or Phase cell, or facts record is
written. No new dependency.

## Plan

1. Resolve the record-placement conflict that the plan review named as its
   condition: write [ADR-0010](../adr/0010-hold-product-architecture-decisions-here.md)
   and amend "What belongs in this directory" in [`adr/README.md`](../adr/README.md)
   so that tally's own architecture decisions have an explicit home there.
2. Write one ADR for each decision with one survivor:
   [ADR-0011](../adr/0011-build-with-go-build-only.md) (`go-build-only`),
   [ADR-0012](../adr/0012-count-through-three-functions-over-a-teed-reader.md)
   (`three-functions-teed-reader`),
   [ADR-0013](../adr/0013-parse-flags-by-hand.md) (`hand-written-parser`),
   [ADR-0014](../adr/0014-stream-through-a-fixed-buffer.md) (`stream-fixed-buffer`),
   [ADR-0015](../adr/0015-validate-utf-8-in-the-stream-and-print-after-eof.md)
   (`validate-in-stream-print-after-eof`); index each in `adr/README.md`.
3. Write [SPEC-0002](../spec/SPEC-0002-architecture.md): the components, the flow
   of one run, and a "Decision pending the bet" section for module layout
   (`root-main-count-subpackage`, `cmd-tally-count-subpackage`) and for the
   white-space class (`unicode-isspace`, `unicode-white-space-table`).
4. Update every document the change leaves stale: the project status in
   [onboarding](../onboarding-for-engineers.md).

The plan review approved the plan with one condition (step 1), architecture and
scope only. The plan named a choice for module layout and the white-space class;
this step chooses neither, since each has two survivors and the choice belongs to
the bet. ADR numbers therefore run 0010 to 0015, not 0010 to 0016. No build test is
required at this step.

## Definition of Done

- [x] The record-placement conflict is resolved by ADR-0010 and the amended
  `adr/README.md`.
- [x] Each decision with one survivor has one accepted ADR, indexed, naming the
  rejected options and the constraints they failed.
- [x] Each decision with more than one survivor has a "Decision pending the bet"
  section in SPEC-0002 that lists its survivors and chooses none.
- [x] No option that failed the filter is written as chosen.
- [x] No Go code, MoSCoW or Phase cell, or file under `docs/facts/` is written.
- [x] No document is left stale, except a rule file this step may not change.

## Evidence

- `sh docs/adr/adr-lint.sh` reports OK, with one warning that was there before
  this change: nothing outside `adr/` links ADR-0009.
- `sh docs/links/link-lint.sh` reports OK, 825 links resolved;
  `sh docs/tests/run-discipline-tests.sh` reports 81 passed, 0 failed;
  `git diff --check` reports nothing.
- `docs/engineering-discipline.md` ("Architecture Decision Records") still reads
  that `adr/` holds records that constitute a project; it is a rule file of this
  step and is left as it is. ADR-0010 reads product records as part of that
  constitution.

## Verdict

Pending the review rounds.
