# tally

*A small Go command that counts the lines, words and characters of text files.*

**What this is.** This repository, `pharzam/layup-uat-m2e2`, holds **tally**: a
command, written in Go with the standard library only, that reads the files a person
names in a terminal and prints the number of lines, words and characters of each,
and a total. It is built under an engineering discipline taken from Armature — a
quality gate, guardrails, ADRs, a glossary, a customer-facts convention, and a task
backlog — which the team keeps in step with tally and grows over time.

```
tally [-l] [-w] [-c] [file...]
```

With no flag, tally prints all three counts; `-l`, `-w` and `-c` select lines, words
or characters. With no file name, it reads standard input. The full behaviour is in
the [problem statement](docs/facts/problem-statement-brief.md) and the idea owner's
[answers](docs/facts/F-0001-setup-answers.md).

## Start here

1. **[`docs/onboarding-for-engineers.md`](docs/onboarding-for-engineers.md)** — read this first
   (~30 min). It states the [Problem statement](docs/onboarding-for-engineers.md#1-problem-statement)
   and teaches the project's vocabulary.
2. **[`docs/engineering-discipline.md`](docs/engineering-discipline.md)** — how we work: the
   quality gate every substantive task passes, plus branches, tests, reviews, and
   ADRs. Read before your first commit.
3. **[`AGENTS.md`](AGENTS.md)** — if you are a coding agent, start here instead. It
   summarises [`docs/engineering-discipline.md`](docs/engineering-discipline.md)
   and [`docs/issue-workflow.md`](docs/issue-workflow.md), and indexes the rest.
   The long documents stay authoritative.

## Where things stand

The repository is set up and holds no Go code yet. The facts are recorded under
[`docs/facts/`](docs/facts/); the requirements, the Go module and its tests are the
work ahead, tracked in [`docs/tasks/backlog.md`](docs/tasks/backlog.md).

## What's inside

| Piece | What it holds |
|-------|---------------|
| [`AGENTS.md`](AGENTS.md) | The agent entry point: the quality gate, the checks, a pointer to the R1–R13 rules, and which document is authoritative for each — in under 1,500 words. |
| [`CLAUDE.md`](CLAUDE.md) | One line, `@AGENTS.md`, so Claude Code loads the same guide. No second copy to drift. |
| [`docs/agents/`](docs/agents/) | What the entry points are and what they may not become ([ADR-0004](docs/adr/0004-ship-agent-entry-points.md)). |
| [`docs/onboarding-for-engineers.md`](docs/onboarding-for-engineers.md) | The first door: the problem statement and a crash course in counting text. |
| [`docs/engineering-discipline.md`](docs/engineering-discipline.md) | The quality gate, the reusable solution-selection standard, and every working practice. |
| [`docs/issue-workflow.md`](docs/issue-workflow.md) | The issue-first workflow (R1–R13): the ticket policy the gate assumes. |
| [`docs/glossary.md`](docs/glossary.md) | The shared vocabulary the other docs assume, tally's terms included. |
| [`docs/guardrails.md`](docs/guardrails.md) | Known pitfalls, pre-registered pass/fail rules, and validation. |
| [`docs/adr/`](docs/adr/) | Architecture Decision Records that constitute the project — the *why* behind structural choices — plus [`adr-lint.sh`](docs/adr/adr-lint.sh), the discipline test that keeps them honest. |
| [`docs/facts/`](docs/facts/) | The idea owner's facts, kept as immutable evidence, and the citation convention that derives requirements from them. |
| [`docs/prd/`](docs/prd/) | Product Requirements Documents derived from the facts, plus [`prd-lint.sh`](docs/prd/prd-lint.sh), the discipline test that keeps them honest. |
| [`docs/tests/`](docs/tests/) | The testing conventions — the test levels, a pattern per level, the security, scaling, and Definition-of-Done checklists, and test-to-requirement traceability — plus [`run-discipline-tests.sh`](docs/tests/run-discipline-tests.sh), which tests the discipline linters against fixtures. |
| [`tests/`](tests/) | The repo-root home of tally's product tests. Empty until the Go code exists, kept in git by a `.gitkeep`. |
| [`docs/tasks/`](docs/tasks/) | The task index — [`backlog.md`](docs/tasks/backlog.md) and [`completed.md`](docs/tasks/completed.md). |
| [`docs/setup/`](docs/setup/) | The record of how this repository was set up: [`armature.pin`](docs/setup/armature.pin) names the exact Armature commit the discipline was copied from. |
| [`.githooks/`](.githooks/) | Git hooks that enforce the cheap gate locally — a commit-message check and a pre-commit runner. Install with `sh .githooks/install.sh`. |
| [`.gitattributes`](.gitattributes) | Keeps the scripts and hooks at line-feed endings, without which none of them runs on a Windows checkout, and pins the handful of fixtures whose Windows endings *are* the assertion. |
| [`docs/ci/`](docs/ci/) | Opt-in CI workflow files (GitHub Actions and GitLab CI) that run the same gate on every PR. The ready-as-is subset already runs under `.github/workflows/`. |
| [`docs/templates/`](docs/templates/) | Inert GitHub/GitLab issue and PR forms that embody the issue-first workflow. Inert until copied into place. |

## The baseline and its pin

The discipline in this repository was copied once from Armature, at the commit named
in [`docs/setup/armature.pin`](docs/setup/armature.pin); it is a one-time copy, not a
dependency, and the repository keeps no Armature history or remote. Some documents
still hold angle-quote markers for values the team has not settled yet — the test
runner, the evidence store, the task-ID scheme and the worktree directory; they
are listed in
[How this project tailors the discipline](docs/engineering-discipline.md#how-this-project-tailors-the-discipline).

## Branches

`main` is the default branch, and every change lands on it through a pull request.
The branch `layup-records` holds the records of the LAYUP delivery that set up and
builds this repository; it is kept apart from `main` so those records never mix
with tally's own history.

## Turning on enforcement

Install the git hooks once per clone by running `sh .githooks/install.sh` (it pins
`core.hooksPath` to the relative `.githooks`). The
[ADR](docs/adr/adr-lint.sh) and [PRD](docs/prd/prd-lint.sh) linters, the
[link linter](docs/links/link-lint.sh) and their
[fixture self-tests](docs/tests/run-discipline-tests.sh) run green out of the box;
the Go lint, test and security steps are set once the Go module exists.
