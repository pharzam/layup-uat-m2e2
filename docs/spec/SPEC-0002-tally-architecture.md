# SPEC-0002. The architecture of tally

Date: 2026-10-23

This specification states how tally is built: its parts, how a run flows through
them, and which part meets each requirement of
[PRD-0001](../prd/PRD-0001-tally.md). The behaviour each part must have is in
[SPEC-0001](SPEC-0001-tally.md); this document does not repeat it. It derives from
the same facts — the problem statement (F-0003, rendered at the first bet), the
marker answers ([F-0002](../facts/F-0002-marker-answers.md)) and the idea owner's
answers Q-001 to Q-014 in [F-0001](../facts/F-0001-setup-answers.md) — and from
the options tables of seven decisions, filtered against those requirements, the
invariants of [`guardrails.md`](../guardrails.md) and ADR-0001 to ADR-0009.

Five decisions kept one surviving option each; they are decided here and recorded
as ADR-0011 to ADR-0015. Two decisions kept more than one surviving option; each
has a section "Decision pending the bet" below, which lists its survivors and
decides nothing. It builds nothing: no Go code exists yet.

## The decisions

| Decision | Survivors of the filter | State | Record |
| -------- | ----------------------- | ----- | ------ |
| Build tool | `go-build-only` | Decided | [ADR-0011](../adr/0011-build-with-go-build-only.md) |
| Flag parsing | `hand-written-parser` | Decided | [ADR-0012](../adr/0012-parse-flags-by-hand.md) |
| Read strategy | `stream-fixed-buffer` | Decided | [ADR-0013](../adr/0013-stream-through-a-fixed-buffer.md) |
| UTF-8 validation | `validate-in-stream-print-after-eof` | Decided | [ADR-0014](../adr/0014-validate-utf8-in-the-stream.md) |
| Counting interface | `three-functions-teed-reader` | Decided | [ADR-0015](../adr/0015-count-with-three-functions-over-a-teed-reader.md) |
| Module layout | `root-main-count-subpackage`, `cmd-tally-count-subpackage` | Pending the bet | [below](#the-module-layout) |
| White space test | `unicode-isspace`, `unicode-white-space-table` | Pending the bet | [below](#the-white-space-test) |

Why the product decisions are ADRs is [ADR-0010](../adr/0010-record-product-decisions-as-adrs.md).

## The parts

- **One Go module**, with no `require` directive and only standard-library imports
  (NFR-018, NFR-019).
- **The package `main`** of the command. It holds the command-line parser, opens
  each source, reads it through one buffer, validates its encoding, tees its bytes
  to the counting functions, prints the rows and the total, writes the messages on
  standard error and sets the exit code.
- **The package `count`**, separate from the package `main` (NFR-021). It holds the
  three exported functions `Lines`, `Words` and `Chars`, each over an `io.Reader`,
  and nothing that opens a file, prints or exits. The coverage target of 90 percent
  applies to it only.

Where the two packages sit in the module is pending the bet; see
[The module layout](#the-module-layout). Both
survivors keep `count` a separate package of the same module.

## Build

`go build` builds the binary, with no other build tool
([ADR-0011](../adr/0011-build-with-go-build-only.md), NFR-020). The package path
given to `go build` follows from the module layout, which is pending the bet.

## The command-line parser

The package `main` parses its arguments by hand, in order, with the rules of
[ADR-0012](../adr/0012-parse-flags-by-hand.md): before the first file name, `-`
followed by one or more of `l`, `w` and `c` selects those counts, and any other
argument of two or more characters that starts with `-` is a usage error (exit
code 2, nothing counted, nothing on standard output); `-` alone is a file name;
every other argument is a file name. No flag selects all three counts. The parser
returns the selected counts and the list of names; it opens nothing.

What an argument that starts with `-` means after a file name, and what `--`
means, is not in the facts and is not decided here (see ADR-0012).

## Reading a source

A source is a named file, or standard input when the list of names is empty. The
command opens each named file in order. A file that cannot be opened or read, a
directory included, gives a message on standard error with its name and the
reason, no row, and the exit code 1 at the end; the next file is still read
(REQ-014, REQ-015).

Each source is read once, from start to end, through one buffer of fixed size
allocated once and reused ([ADR-0013](../adr/0013-stream-through-a-fixed-buffer.md)).
The size of the buffer is set by the implementation and checked by the peak-heap
test. No source is read whole into memory.

## Validating the encoding

During the same single read, the command checks each buffer for valid UTF-8 with
the package `unicode/utf8`, carrying an incomplete sequence at the end of one read
into the next. An incomplete sequence at the end of the source is invalid
([ADR-0014](../adr/0014-validate-utf8-in-the-stream.md)).

When the source ends valid, the command prints its row and adds its counts to the
total. When it is invalid, the command prints a message on standard error with its
name, prints no row, adds nothing to the total and records the exit code 1; rows
already printed stay, and the next file is still read (REQ-016).

## The package count

The command tees the bytes of each read to one reader for each selected count, and
each reader feeds one of `Lines`, `Words` and `Chars`
([ADR-0015](../adr/0015-count-with-three-functions-over-a-teed-reader.md)). Each
function follows its rule in SPEC-0001: `Lines` counts line feeds and adds one when
the text is not empty and does not end with a line feed (REQ-009); `Words` counts
the runs of code points that are not white space (REQ-010); `Chars` counts code
points (REQ-011). Each function keeps its state across reads, so a code point or a
word split between two reads is counted once.

The test that decides whether a code point is white space is pending the bet; see
[The white space test](#the-white-space-test). Both
survivors use the Unicode tables of the Go standard library, not a hand-written
set.

## The flow of one run

1. Parse the arguments. On a usage error, print the usage message on standard
   error and exit with 2.
2. For each source in order: open it, read it through the buffer while validating
   it and teeing it to the selected counting functions, then print its row or its
   message as above.
3. With two named files or more, print the total row: the sum of each selected
   count over the files that gave a row, then the name `total` (REQ-013).
4. Exit with 1 if any source gave a message, otherwise with 0 (REQ-017).

A row is the selected counts in the order lines, words, characters, then the name,
each separated from the next by one space; the row of standard input ends after
its last count (REQ-008, REQ-012).

## The module layout

### Decision pending the bet

Where the package `main` and the package `count` sit in the one module. The filter kept two survivors, and this section chooses neither:

- `root-main-count-subpackage` — the package `main` at the root of the module, the
  package `count` in a subdirectory of it.
- `cmd-tally-count-subpackage` — the package `main` in the directory `cmd/tally`,
  the package `count` in a subdirectory of the module.

The filter failed the third option, `single-main-package`, on NFR-021 and Inv-7,
because it puts the counting code in the package `main`.

## The white space test

### Decision pending the bet

How `Words` decides that a code point has the Unicode
property `White_Space` (REQ-010). The filter kept two survivors, and this section
chooses neither:

- `unicode-isspace` — the function `unicode.IsSpace` of the Go standard library.
- `unicode-white-space-table` — the range table `unicode.White_Space`, tested with
  `unicode.Is`.

The filter failed the third option, `ascii-whitespace-set`, on REQ-010, because it
misses the white space outside ASCII, such as U+00A0 and U+3000.

## REQ-001 — Count each named file and print a total

The package `main` reads each named source and prints its row; step 3 of the flow
prints the total. The counts come from the package `count`.

## REQ-002 — The command line

The command-line parser (ADR-0012).

## REQ-003 — Combined flags

The command-line parser, rule 1 of ADR-0012.

## REQ-004 — The name `-` is a file

The command-line parser, rule 3 of ADR-0012; the file is then opened like any
other named file.

## REQ-005 — Usage errors

The command-line parser, rule 2 of ADR-0012, and step 1 of the flow.

## REQ-006 — The flags `-l`, `-w` and `-c`

The command-line parser selects the counts; the command tees the bytes only to the
selected functions and prints only their counts.

## REQ-007 — No flag selects all three counts

The command-line parser, rule 5 of ADR-0012.

## REQ-008 — Standard input

The package `main` reads standard input as the one source when no name is given,
once, through the same buffer (ADR-0013, ADR-0015); its row has no name.

## REQ-009 — What a line is

The function `Lines` of the package `count`.

## REQ-010 — What a word is

The function `Words` of the package `count`, with the white space test pending the
bet.

## REQ-011 — What a character is

The function `Chars` of the package `count`.

## REQ-012 — The shape of a row

The package `main` formats each row in the fixed order lines, words, characters.

## REQ-013 — The total row

The package `main`, step 3 of the flow.

## REQ-014 — A file that cannot be read

The package `main`, under [Reading a source](#reading-a-source).

## REQ-015 — A directory

The package `main`, under [Reading a source](#reading-a-source): a directory fails
to be read like any unreadable file.

## REQ-016 — A file that is not valid UTF-8

The package `main`, under [Validating the encoding](#validating-the-encoding)
(ADR-0014).

## REQ-017 — Success exits with 0

The package `main`, step 4 of the flow.

## NFR-018 — Go

Every part is Go.

## NFR-019 — The standard library only

The module has no `require` directive; the parser, the buffer, the validation and
the tee use the standard library only.

## NFR-020 — The Go toolchain

ADR-0011.

## NFR-021 — The package `count`

The package `count`, separate from the package `main` in both survivors of the
module layout (ADR-0015).

## NFR-022 — Only lines, words and characters

The package `count` has three functions and the parser three flags; there is no
other count.

## NFR-023 — No archives, directories or non-text files

The command opens a named path as one file and never recurses; a source that is not
valid UTF-8 is rejected under ADR-0014.

## NFR-024 — A command-line tool only

The package `main` is the only entry point; it opens no listener and loads no
plug-in.
