# 0010. Hold product architecture decisions here

Date: 2026-10-22

## Status

Accepted

## Context

Two documents disagreed about where a decision on tally's own structure belongs.
[`engineering-discipline.md`](../engineering-discipline.md#architecture-decision-records)
directs every architecturally significant decision — structure, non-functional
characteristics, dependencies, interfaces, construction techniques — to an ADR in
this directory. The [index of this directory](README.md) restricted it to records
"generic enough to carry onto any domain", which shuts out a decision about how
tally builds, reads and counts. Under
[R10](../issue-workflow.md#r10--sync-with-governance) a disagreement between two
documents is a defect to fix, and a conflict between two instructions stops work
until a decision record resolves it.

## Decision

We will hold tally's own architecture decisions in this directory, in the single
ADR sequence, beside the constitutive records. Such a record:

- decides one architecturally significant question about tally, from the options
  that survived the constraint filter of its decision;
- may cite tally's requirements and facts — [`prd/`](../prd/), [`spec/`](../spec/),
  [`facts/`](../facts/) and [`guardrails.md`](../guardrails.md) — besides the
  sibling constitutive documents;
- still references no issue or pull request of this repository, and still never
  links into the archive the index names.

A decision whose filter left more than one option is not recorded as an ADR until
one option is chosen; the architecture
([SPEC-0002](../spec/SPEC-0002-architecture.md)) lists its survivors under
"Decision pending the bet".

We reject a separate directory for product decisions: the discipline document
sends architecturally significant decisions here, so a second home would make the
two disagree again.

## Consequences

- The index of this directory says that it holds both kinds of record.
- A reader finds every decision about tally in one numbered sequence, and the
  linter checks every one of them.
- A product record does not carry onto another domain; a copy of this baseline
  for another project drops the product records and keeps the constitutive ones.
