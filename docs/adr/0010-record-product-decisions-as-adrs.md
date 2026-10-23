# 0010. Record tally's product decisions as ADRs

Date: 2026-10-23

## Status

Accepted

## Context

ADR-0001 to ADR-0009 record how a project is run, and the
[ADR index](README.md) used to admit only records generic enough to carry onto any
domain. The architecture of tally now has decisions of its own: how it is built,
how it parses its command line, how it reads a file, where it checks the encoding
and how the package `count` is shaped. The
[engineering discipline](../engineering-discipline.md#architecture-decision-records)
calls a decision about structure, non-functional characteristics, dependencies,
interfaces or construction techniques architecturally significant, and sends it to
`docs/adr/`. The two texts gave such a decision no single home.

## Decision

We will record each architecturally significant decision about tally in
`docs/adr/`, as a **product record**, in the same sequence and the same format as
the discipline records, starting at ADR-0011. A product record cites the
requirement it serves in [PRD-0001](../prd/PRD-0001-tally.md), the architecture in
[SPEC-0002](../spec/SPEC-0002-tally-architecture.md) and, where it needs one, a
fact under [`facts/`](../facts/). It never cites an issue or a pull request. The
[ADR index](README.md) states the two kinds and the citations each may make.

A decision whose options table holds more than one surviving option is not
recorded here until one option is chosen: SPEC-0002 lists its survivors under the
heading "Decision pending the bet", and the record is written when the choice is
made.

We reject two alternatives:

- **A separate directory for product decisions.** It would start a second numbered
  sequence beside this one, which [ADR-0001](0001-record-architecture-decisions.md)
  does not provide for, and `adr-lint` would not read it.
- **Product decisions in the specification only.** A specification states what is
  built; it does not keep the rejected options and the consequences, so a later
  reader could reopen a closed question.

## Consequences

- One sequence holds every decision; `adr-lint` checks the product records like the
  others.
- A product record is specific to tally and does not carry onto another domain;
  the index says which records are which.
- A product record links into `prd/`, `spec/` and `facts/`, so moving those
  directories breaks its links, and `link-lint` reports it.
