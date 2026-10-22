# 0010. Record the product's architecture decisions in this directory

Date: 2026-10-22

## Status

Accepted

## Context

[ADR-0001](0001-record-architecture-decisions.md) records the decision to keep
architecture decisions as ADRs, and
[`engineering-discipline.md`](../engineering-discipline.md#architecture-decision-records)
sends every decision that affects structure, dependencies, interfaces or
construction techniques to this directory. The directory's README, in its section
"What belongs in this directory", said that the directory holds only records
generic enough to carry onto any domain. A decision about how the product itself is
built — its build command, its packages, its parser, its reading of a file — is
architecturally significant and is not generic. The two texts gave two homes, or
none, for such a decision.

## Decision

We will record the architecture decisions of the product in this directory, in the
same sequence and the same format as the constitutive records. Each such record
says in its title or its first paragraph that it is a decision of the product. It
keeps the two written rules of the README: it references no issue or pull request
of this repository, and it never links into the closed archive. It cites the
requirements and the specification it serves by their IDs and by links to
[`prd/`](../prd/) and [`spec/`](../spec/).

We reject a second directory for product decisions: the discipline names one ADR
directory, and its linter reads only this one, so a second directory would hold
records that no check reads. We reject keeping the product decisions on the issue
only: a fresh context then loses them when the issue closes.

## Consequences

- The README's section "What belongs in this directory" now names the two kinds of
  record that this directory holds.
- A copy of this repository's discipline onto another project carries the product
  records with it; the copy removes them, as it removes the facts and the PRD.
- A decision that one survivor of the filter settles becomes a record here. A
  decision with more than one survivor gets no record until the bet decides it.
