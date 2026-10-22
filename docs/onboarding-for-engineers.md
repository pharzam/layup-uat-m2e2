# Onboarding for engineers

> **Who this is for.** Software engineers joining tally who do **not** necessarily
> know how a text counter is specified. You know how to build software. This
> document teaches you the project's terminology and the few facts that shape every
> decision, then hands you off to the deeper docs.
>
> **Read this first.** Every other document is written for someone already fluent
> in the terminology. This one gets you to that point. Budget half an hour.

## 1. Problem statement

A person who works in a terminal wants to know how long a text file is. **tally** is
a small command, written in Go with the standard library only, that reads the files
the person names and prints the number of lines, words and characters of each, and a
total. It is the project's whole product: there is no graphical interface, no
network service and no plug-in system.

The idea owner's own words are kept as evidence in the
[problem statement](facts/problem-statement-brief.md), and the answers they gave to
the open questions of that statement are in
[`F-0001`](facts/F-0001-setup-answers.md). Those two files are the source of every
requirement; when this page and they disagree, they win.

The question the project answers is simple to say and easy to get subtly wrong: how
do you count lines, words and characters of UTF-8 text exactly, for files of any
size, without reading a whole file into memory? Nothing is built yet: the
repository holds the engineering discipline and the facts, and the Go code is the
work ahead.

## 2. Crash course: the domain

The terms below build on each other, in the order you need them. The same
vocabulary is in [glossary.md](glossary.md) as a table.

### 2.1 Code point and UTF-8

Text in a file is a sequence of bytes. **UTF-8** encodes each Unicode **code
point** in one to four bytes. tally counts code points, not bytes: the letter `é`
is two bytes in UTF-8 and one character for tally.

    bytes:       63 61 66 C3 A9 0A
    code points: c  a  f  é     LF   -> 5 characters

A file whose bytes are not valid UTF-8 is, for tally, a file that is not text
(F-0001#8).

### 2.2 Character

A **character** is one code point. Every code point counts, with no exception: the
line feed at the end of a line is a character, a carriage return before it is a
character, and a byte order mark at the start of a file is a character that tally
does not strip (F-0001#6, F-0001#7).

### 2.3 Line

A **line** is a run of characters that ends with a line feed. The last line of a
file counts too when the file does not end with a line feed. A line feed alone is
an empty line and counts as one line; an empty file has zero lines (F-0001#6).

    "a\nb\n"  -> 2 lines
    "a\nb"    -> 2 lines (the last line has no line feed)
    "\n"      -> 1 line
    ""        -> 0 lines

### 2.4 Word

A **word** is a run of characters that are not white space, where white space is
any character with the Unicode property `White_Space` — not only the ASCII space,
tab and line feed, but also, for example, the no-break space.

### 2.5 Row and total

The output has one **row** for each file: the counts the flags select, then the
file name, each separated from the next by one space. With two files or more, a
last row named `total` adds them up. The counts always come in the fixed order
lines, words, characters, whatever the order of the flags (F-0001#2). The row for
standard input has no name: it ends after the counts (F-0001#5).

    $ tally a.txt b.txt
    2 5 21 a.txt
    10 40 233 b.txt
    12 45 254 total

## 3. What the system actually does

tally is one pipeline, run once for each input:

    parse flags -> for each file: open -> read in chunks -> validate UTF-8 and count
                -> print the row -> add to total
                -> print total (two files or more) -> exit code

- **Flags.** `-l`, `-w` and `-c` select lines, words and characters; with no flag,
  all three. Combined flags such as `-lw` mean the same as `-l -w`. The name `-` is
  a file named `-`, not standard input. An unknown flag prints a usage message on
  standard error, counts nothing, and ends with the exit code 2 (F-0001#3).
- **Input.** With no file name, tally reads standard input.
- **Counting.** The counting lives in a separate Go package, `count`, in the same
  module as the `main` package of the command; it counts the lines, words and
  characters of a reader (F-0001#11).
- **Errors.** A file that cannot be read — a directory included — gives a message
  on standard error with its name and the reason. A file that is not valid UTF-8
  gives a message with its name; it gets no row and adds nothing to the total, and
  rows already printed for earlier files stay. In both cases the other files are
  still counted and the exit code is 1 (F-0001#4, F-0001#8). With no error, the
  exit code is 0.

## 4. Why it is hard

Three numbers make tally harder than it looks (F-0001#15):

- **Speed.** A file of 10 MiB is counted in at most 2 seconds of elapsed time for
  the whole process, start-up included, on a computer with 4 cores and a
  solid-state disk, with the file already in the cache of the operating system
  (F-0001#9).
- **Memory.** A file of 1 GiB is counted with at most 64 MiB of **peak** heap —
  sampled every 10 milliseconds during the count, not read once at the end
  (F-0001#10). A 1 GiB file does not fit in that limit, so the obvious approach,
  reading the whole file and then counting, does not work: tally must stream.
- **Coverage.** The package `count` has at least 90 percent statement coverage
  (F-0001#11).

Streaming brings its own trap: a multi-byte UTF-8 sequence can be split across two
reads, and a counter that validates each chunk on its own reports a valid file as
invalid, or counts one code point twice.

## 5. How this project works

### The one cultural thing to understand

We are more afraid of a count that is quietly wrong than of being slow to ship. A
counter that is off by one on the last line, or treats a no-break space as part of a
word, looks right on every everyday file and fails only on the file nobody tried.
So the edge case is written down as a failing test first, and the facts — not
anyone's memory of what the idea owner meant — settle the expected output.

Consequences you will meet immediately, and which are not negotiable:

- Every behaviour change starts from a failing test, and a confirmed test is never
  weakened to make new code pass — see the
  [quality gate](engineering-discipline.md#working-a-task-under-the-quality-gate).
- Every requirement cites the fact it comes from, and a question the facts do not
  answer goes back to the idea owner on the issue instead of being guessed — see
  [`guardrails.md`](guardrails.md).

### Repo layout

| Path | What |
|------|------|
| [`AGENTS.md`](../AGENTS.md) | The agent entry point — the gate in brief and a pointer to the R1–R13 rules, in one short file. [`CLAUDE.md`](../CLAUDE.md) imports it for Claude Code. |
| [`engineering-discipline.md`](engineering-discipline.md) | **How we work**: the quality gate, solution selection, branches, worktrees, commits, tests, reviews, and ADRs. Read before your first commit. |
| [`glossary.md`](glossary.md) | The shared vocabulary. Skim it; come back constantly. |
| [`facts/`](facts/) | Facts collected from the idea owner, stored as-is as immutable evidence. Derived requirements cite them by `F-NNNN` ID. |
| [`prd/`](prd/) | Product Requirements Documents, derived from the facts; each `REQ`/`NFR` cites an `F-NNNN` fact. |
| [`spec/`](spec/) | The specification of tally: one section per `REQ`/`NFR` of the PRD, saying how the command and the package `count` behave; and the architecture of tally, saying how they are built. |
| [`issue-workflow.md`](issue-workflow.md) | The issue-first rules (R1–R13): the ticket policy the gate assumes. |
| [`tasks/backlog.md`](tasks/backlog.md) | What to work on next. |
| [`tests/`](../tests/) | Where tally's product tests go once there is Go code. |

### What to read next, in order

0. [`AGENTS.md`](../AGENTS.md) — the one-page summary, if you are a coding agent
   (or a human who wants the shape before the detail).
1. [`engineering-discipline.md`](engineering-discipline.md) — how we work.
2. [`issue-workflow.md`](issue-workflow.md) — the issue-first rules the gate assumes.
3. [`glossary.md`](glossary.md) — skim, then reference.
4. [`guardrails.md`](guardrails.md) — the pitfalls and the frozen numbers.
5. [`tasks/backlog.md`](tasks/backlog.md) — what needs doing.

### Where the project stands

The repository is set up: the discipline is in place, the problem statement and the
idea owner's answers are recorded as facts, and the delivery has its success
criteria (F-0001#15). The requirements are derived: [PRD-0001](prd/PRD-0001-tally.md)
states them as REQ-001 to REQ-017 and NFR-018 to NFR-024, and
[SPEC-0001](spec/SPEC-0001-tally.md) specifies each one; their MoSCoW priorities and
phases are set at the first bet. [SPEC-0002](spec/SPEC-0002-tally-architecture.md)
states the architecture: ADR-0011 to ADR-0015 decide the build, the interface of the
package `count`, the parser, the streaming read and the check of UTF-8, and the module
layout and the white-space class wait for the bet. No Go code exists yet. The next step is to write the
first failing test for the package `count`, from the specification.
