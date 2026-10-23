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
the survivors of their filter, with the revised requirements draft and its facts.
A decision with one survivor is decided and recorded as an ADR; a decision with
more than one survivor gets a section "Decision pending the bet" that lists its
survivors and decides nothing. No Go code, no MoSCoW or Phase cell and no facts
record is written. No new dependency.

## Plan

1. Resolve where tally's product decisions live (the condition of the plan review,
   R10): [ADR-0010](../adr/0010-record-product-decisions-as-adrs.md) puts them in
   `docs/adr/` as product records, and [`adr/README.md`](../adr/README.md) states
   the two kinds of record and lets a product record cite `prd/`, `spec/` and
   `facts/`.
2. Record the five decisions with one survivor each:
   [ADR-0011](../adr/0011-build-with-go-build-only.md) (`go-build-only`),
   [ADR-0012](../adr/0012-parse-flags-by-hand.md) (`hand-written-parser`),
   [ADR-0013](../adr/0013-stream-through-a-fixed-buffer.md) (`stream-fixed-buffer`),
   [ADR-0014](../adr/0014-validate-utf8-in-the-stream.md)
   (`validate-in-stream-print-after-eof`) and
   [ADR-0015](../adr/0015-count-with-three-functions-over-a-teed-reader.md)
   (`three-functions-teed-reader`).
3. Write [SPEC-0002](../spec/SPEC-0002-tally-architecture.md): the parts, the flow
   of a run, one section per requirement ID, and one section "Decision pending the
   bet" for each of the module layout and the white space test.
4. Update every document the change leaves stale: the ADR index and its next
   number, and the project status and repo layout in
   [onboarding](../onboarding-for-engineers.md).

### Revision of the plan on the issue

The plan on issue #6 selected `cmd-tally-count-subpackage` for the module layout
and `unicode-isspace` for the white space test, and named seven ADRs, 0010 to
0016. The instruction of this step of the run, written by the operator, says:
"For each decision that has more than one survivor, write a section with the
heading 'Decision pending the bet' that lists its survivors and decides nothing.
Never choose among survivors." Both decisions have two survivors, so this change
does not carry those two selections: it lists the survivors of each and leaves the
choice to the bet, and it writes six ADRs, 0010 to 0015. The two selections and
their reasons stay on the issue as input to the bet. The review approved the plan's
architecture and scope with one condition, record placement, which step 1 meets;
the narrower scope removes two decisions and adds none. This revision is posted on
the issue with this change, for the plan review to confirm; until it does, the last
box of the Definition of Done stays open.

## Definition of Done

- [x] Each decision with one survivor is written as decided, in SPEC-0002 and in one
  ADR, with its rejected options and the constraint each failed.
- [x] Each decision with more than one survivor has a section "Decision pending the
  bet" in SPEC-0002 that lists its survivors and decides nothing.
- [x] No option that is not a survivor is written as a choice.
- [x] The record-placement condition of the plan review is resolved in ADR-0010 and
  `adr/README.md`, with no rule left that both forbids and requires a product
  record's citations.
- [x] Each of the 24 requirement IDs has one section in SPEC-0002.
- [x] No Go code, MoSCoW or Phase cell, or facts record is written.
- [ ] The revision of the plan is confirmed on issue #6.

## Evidence

- `sh docs/adr/adr-lint.sh` reports `adr-lint: OK`, with one warning that
  existed before this change (nothing outside `adr/` links ADR-0009).
- `sh docs/links/link-lint.sh` reports `link-lint: OK`;
  `sh docs/tests/run-discipline-tests.sh` reports 81 passed, 0 failed;
  `git diff --check` is clean.
- `sh docs/prd/prd-lint.sh` still fails only on the empty MoSCoW and Phase cells of
  PRD-0001, which the first bet fills; this change does not touch them.
- The facts do not say what an argument that starts with `-` means after a file
  name, nor what `--` means; ADR-0012 decides neither, and the implementation task
  asks the idea owner.
- [`engineering-discipline.md`](../engineering-discipline.md#architecture-decision-records)
  still says `adr/` holds only records that constitute a project. It is a rule file
  of this run and is not changed here; ADR-0010 reads a product record as one that
  constitutes this project.

## Verdict

Pending the review rounds.
