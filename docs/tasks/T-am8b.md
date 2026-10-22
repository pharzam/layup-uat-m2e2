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
the survivors of their filter, with the revised requirements draft and its facts: the
problem statement (F-0003, rendered at the first bet), the marker answers
([F-0002](../facts/F-0002-marker-answers.md)) and the idea owner's answers Q-001 to
Q-014. A decision with one survivor is decided; a decision with more than one
survivor is written as "Decision pending the bet" and decided nothing. No Go code,
no MoSCoW or Phase cell and no facts record is written. No new dependency.

## Plan

1. Resolve the record-placement condition of the plan review first:
   [ADR-0010](../adr/0010-record-product-architecture-decisions.md) puts the
   product's architecture decisions in `docs/adr/`, and the README's section "What
   belongs in this directory" says so.
2. Write one ADR for each decision with one survivor:
   [ADR-0011](../adr/0011-build-with-go-build-only.md) (go-build-only),
   [ADR-0012](../adr/0012-count-with-three-functions-over-a-teed-reader.md)
   (three-functions-teed-reader),
   [ADR-0013](../adr/0013-parse-flags-by-hand.md) (hand-written-parser),
   [ADR-0014](../adr/0014-stream-through-a-fixed-buffer.md) (stream-fixed-buffer) and
   [ADR-0015](../adr/0015-validate-utf8-in-the-stream.md)
   (validate-in-stream-print-after-eof).
3. Write [SPEC-0002](../spec/SPEC-0002-tally-architecture.md): the decided
   architecture, and a section "Decision pending the bet" for the module layout
   (root-main-count-subpackage, cmd-tally-count-subpackage) and for the white-space
   class (unicode-isspace, unicode-white-space-table).
4. Update every document the change leaves stale: the ADR index and numbering, the
   ADR column of the PRD's traceability matrix and its change log, the project status
   and the `spec/` row in [onboarding](../onboarding-for-engineers.md), and the
   glossary entry Specification.

### Plan revision

The reviewed plan selected cmd-tally-count-subpackage for the module layout and
unicode-isspace for the white-space class, with an ADR for each. The step
instruction of this run, from the operator, revises that scope: a decision with more
than one survivor is left to the bet, and no option among its survivors is chosen.
Both decisions have two survivors, so this change writes them as "Decision pending
the bet" and writes no ADR for them; the bet's decision becomes their ADR. This
revision narrows the reviewed scope and adds nothing to it; it is recorded here for
the plan reviewer's confirmation on the issue.

## Definition of Done

- [x] Each decision with one survivor is decided in SPEC-0002 and recorded in one
  ADR, which names the rejected options and the constraint each one failed.
- [x] Each decision with more than one survivor has a section "Decision pending the
  bet" that lists its survivors and decides nothing.
- [x] No option that the filter removed is written as decided.
- [x] The record-placement condition of the plan review is resolved by ADR-0010.
- [x] The flag parser gives one instruction for each argument position: the rules
  hold before the first file name, and a flag-like argument after the first file name
  is an open question for the idea owner that the code task neither tests nor
  implements until a fact answers it.
- [x] No document is left stale; no MoSCoW or Phase cell and no file under
  `docs/facts/` is written.

## Open questions

- After the first file name, is an argument such as `-l` in `tally file -l` a flag, a
  file name or wrong usage? No fact answers it; it goes to the idea owner on the
  issue (ADR-0013, SPEC-0002).

## Evidence

- `sh docs/adr/adr-lint.sh` reports OK, with the one warning for ADR-0009 that the
  tree had before this change.
- `sh docs/links/link-lint.sh` reports OK.
- `sh docs/prd/prd-lint.sh` fails only with the empty MoSCoW and Phase cells, as
  before this change, until the first bet fills them.
- The second attempt fixes the findings of the first: the R13 finding by limiting
  the parser's rules to the positions the facts state, in ADR-0013 and SPEC-0002
  alike; the R12 finding by the plan revision above, under the step instruction.

## Verdict

Pending the review rounds.
