# Worked example — from a fact to a test

One concrete walk down the baseline's traceability line, so the abstract chain in
[`traceability-template.md`](traceability-template.md) has a filled example beside
it. It follows a single requirement from the customer's words all the way to the
test that proves it, using only the baseline's conventions and a real line of tally.

    fact (F-0001#6) → requirement (REQ-001) → guardrail → ADR → task (T-ab12) → test

This example needs no architecturally-significant decision, so the **ADR** link is
empty (`—`, as the [row at the end](#5-the-traceability-row-the-line-written-down)
shows); every other link is filled.

> **How to tailor this file.** This example uses one real fact of tally — the
> empty-file answer of the idea owner — but the requirement ID and the task ID
> `T-ab12` are illustrative until the PRD and the task exist. Swap them for the
> real IDs once they are written, or simply delete
> this file. It names no test runner: the test's command stays the `‹unit test
> command›` placeholder throughout. Remove this note if you keep and tailor the
> file.

## In plain terms

> Nothing in tally gets tested by accident. Every test traces back through
> a written requirement to something the customer actually said. This page shows
> that trace once, in full, so the pattern is concrete: a customer sentence
> becomes a requirement, the requirement inherits a known pitfall, and a test
> proves the pitfall is handled.

## 1. The fact (Layer 1 — the customer's words)

A [facts record](../facts/) stores the customer's words unchanged, numbered so a
requirement can cite one line. In
[`facts/F-0001-setup-answers.md`](../facts/F-0001-setup-answers.md), line 6 holds the
idea owner's answer on empty input, which reads in part:

    6. `Q-005` A line feed alone ends an empty line, which counts as one line. An empty file has zero lines, zero words and zero characters. ...

This is `F-0001#6` — the sixth fact in facts document `F-0001`. It is immutable
evidence: it is not rewritten, only cited.

## 2. The requirement (Layer 2 — derived, and it cites the fact)

A [PRD](../prd/) requirement is written *from* that fact and cites it. In the
PRD's `§6` functional-requirements table:

    | REQ | Statement | MoSCoW | Phase | Facts |
    | --- | --------- | ------ | ----- | ----- |
    | REQ-001 | tally counts an empty file as zero lines, zero words and zero characters. | Must | 1 | F-0001#6 |

`REQ-001` is a stable ID, assigned once. Because it cites `F-0001#6`, a reader can
always trace the requirement back to the customer's exact words —
[`prd-lint.sh`](../prd/prd-lint.sh) checks that the cited fact resolves.

## 3. The guardrail (the known pitfall it inherits)

An off-by-one on the last line is a classic trap of line counting, so it belongs in
[`guardrails.md`](../guardrails.md) as a known pitfall:

    - ❌ A line counter that counts one line for an empty file, or misses the last line when the file does not end with a line feed.

The requirement now carries that guardrail: the test must prove the empty file gives
*zero* lines, not merely that a normal file gives the right count.

## 4. The test (strict TDD — red, then green)

The requirement is proved at the [unit level](test-levels.md) — one component, in
isolation — following the [`template-unit.md`](template-unit.md) pattern and the
strict [red-then-green](../engineering-discipline.md#requirements-traceability)
order:

1. **Red.** Write the test first: count an empty reader with the package `count`,
   assert zero lines, zero words and zero characters. Run `‹unit test command›`;
   watch it **fail** against code that still counts one line for no input — failing
   for the *right* reason.
2. **Green.** Correct the line rule, run `‹unit test command›` again, watch it
   **pass**.
3. **Freeze.** Once a fresh context confirms it
   ([R9](../issue-workflow.md#r9--test-freeze-after-confirmation)), the test is
   frozen: not weakened later to make new code pass.

The test lives under `‹test directory›` and is tagged `unit` so it can run alone.

## 5. The traceability row (the line, written down)

Finally the test earns a row in the
[traceability table](traceability-template.md), closing the loop from fact to
test:

    | Test ID | Level | Covers | Fact | Guardrail | ADR | Task | Status |
    |---------|-------|--------|------|-----------|-----|------|--------|
    | count-empty-file | unit | REQ-001 | F-0001#6 | guardrails.md §2 | — | T-ab12 | frozen |

Read left to right, the row is the whole story: the idea owner said it (`F-0001#6`),
it became a requirement (`REQ-001`), it inherited a pitfall (`guardrails.md §2`), a
task delivered it (`T-ab12`), and a frozen unit test proves it. A requirement with
no such row is a requirement with no proof — which the
[DoD checklist](dod-checklist.md) treats as not done.
