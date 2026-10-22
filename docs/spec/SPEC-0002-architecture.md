# SPEC-0002. The architecture of tally

Date: 2026-10-22

This document states how tally is built to meet [PRD-0001](../prd/PRD-0001-tally.md)
as specified by [SPEC-0001](SPEC-0001-tally.md). It derives from seven decisions,
each filtered against the requirements REQ-001 to REQ-017 and NFR-018 to NFR-024,
the facts, the invariants Inv-1 to Inv-8 of [`guardrails.md`](../guardrails.md)
and the ADRs 0001 to 0009. A decision with one surviving option is decided here and
recorded as an ADR; a decision with more than one survivor is listed under
"Decision pending the bet" and decides nothing. It describes structure only; it
builds nothing.

## The decisions

| Decision | Survivors | State | Record |
| -------- | --------- | ----- | ------ |
| Build tool | `go-build-only` | Decided | [ADR-0011](../adr/0011-build-with-go-build-only.md) |
| Count API | `three-functions-teed-reader` | Decided | [ADR-0012](../adr/0012-count-through-three-functions-over-a-teed-reader.md) |
| Flag parsing | `hand-written-parser` | Decided | [ADR-0013](../adr/0013-parse-flags-by-hand.md) |
| Read strategy | `stream-fixed-buffer` | Decided | [ADR-0014](../adr/0014-stream-through-a-fixed-buffer.md) |
| UTF-8 validation | `validate-in-stream-print-after-eof` | Decided | [ADR-0015](../adr/0015-validate-utf-8-in-the-stream-and-print-after-eof.md) |
| Module layout | `root-main-count-subpackage`, `cmd-tally-count-subpackage` | Pending the bet | [below](#module-layout) |
| White-space class | `unicode-isspace`, `unicode-white-space-table` | Pending the bet | [below](#white-space-class) |

Where these records sit is itself decided by
[ADR-0010](../adr/0010-hold-product-architecture-decisions-here.md).

## Components

- **One Go module**, built by `go build` alone, with no `require` directive and
  only standard-library imports (NFR-018, NFR-019, NFR-020; ADR-0011).
- **The package `main`** — the command. It parses the arguments, opens each
  source, reads it once through a fixed buffer, checks its encoding, feeds the
  counting functions, prints the rows and the total, writes the messages on
  standard error and sets the exit code. Where it sits in the module is pending
  (see [Module layout](#module-layout)).
- **The package `count`** — the counting code, separate from the package `main`
  (NFR-021, Inv-7). It exports `Lines`, `Words` and `Chars`, each of which counts
  one quantity of an `io.Reader` (ADR-0012). It holds no flag, output or file
  handling, and it is the only package under the 90 percent coverage target.

## The flow of one run

1. **Parse.** The hand-written parser (ADR-0013) reads the arguments: a group such
   as `-lw` selects counts, `-` alone and every argument not starting with `-` is a
   file name, and any other character in a flag group is a usage error — a usage
   message on standard error, nothing on standard output, exit code 2 (REQ-002 to
   REQ-006). With no flag, all three counts are selected (REQ-007).
2. **Open.** For each name, in command-line order, the command opens the file;
   with no name, the source is standard input (REQ-008). A file that cannot be
   opened or read, a directory included, gives a message with its name and the
   reason, no row, and a final exit code of 1 (REQ-014, REQ-015).
3. **Stream.** The source is read in chunks through a buffer of fixed size
   (ADR-0014). Each chunk is checked for valid UTF-8, an incomplete sequence at a
   chunk's end carried into the next (ADR-0015), and teed to the reader of each
   selected counting function, so the source is read once (ADR-0012).
4. **Row.** At the end of a valid source, the command prints its row: the selected
   counts in the order lines, words, characters, then the name, separated by one
   space; the row for standard input has no name (REQ-008, REQ-012, Inv-1, Inv-6).
   At an invalid sequence, it stops reading the source, prints a message with the
   name, and prints no row (REQ-016, Inv-3).
5. **Total.** With two named files or more, the last row holds the sum of each
   selected count over the files that gave a row, then `total` (REQ-013).
6. **Exit.** 0 when no error occurred, 1 when a file was unreadable or invalid, 2
   on a usage error (REQ-017).

The counting rules — a line ends with a line feed and a last unterminated line
counts (REQ-009), a word is a run of non-white-space code points (REQ-010), a
character is one code point, a byte order mark included (REQ-011) — live in the
package `count`. Each function carries its state across chunk boundaries.

## Module layout

### Decision pending the bet

Two options survived the filter; this document chooses neither.

- **`root-main-count-subpackage`** — the package `main` of the command at the
  root of the module, and the package `count` in a subdirectory `count/`.
- **`cmd-tally-count-subpackage`** — the package `main` of the command in the
  directory `cmd/tally/`, and the package `count` in a subdirectory `count/`.

Both keep the package `count` separate from the package `main` in one module
(NFR-021, Inv-7). The option `single-main-package` failed NFR-021 and Inv-7.

## White-space class

### Decision pending the bet

Two options survived the filter; this document chooses neither.

- **`unicode-isspace`** — the function `Words` tests a code point for white space
  with the standard library's `unicode.IsSpace`.
- **`unicode-white-space-table`** — the function `Words` tests a code point for
  white space against the standard library's range table `unicode.White_Space`.

Both test the Unicode property `White_Space` that REQ-010 names. The option
`ascii-whitespace-set` failed REQ-010.

## Requirements covered

| Requirement | Where it is met |
| ----------- | --------------- |
| REQ-001, REQ-012, REQ-013 | The package `main`: rows in command-line order, then the total (steps 4–5). |
| REQ-002 to REQ-007 | The hand-written parser (ADR-0013, step 1). |
| REQ-008 | Standard input as the single source, read once (ADR-0012, steps 2–4). |
| REQ-009, REQ-011 | `Lines` and `Chars` in the package `count`. |
| REQ-010 | `Words` in the package `count`; its white-space test is pending the bet. |
| REQ-014, REQ-015 | The open step of the package `main` (step 2). |
| REQ-016, NFR-023 | The validation in the stream, row after the end (ADR-0015). |
| REQ-017 | The exit step (step 6). |
| NFR-018, NFR-019, NFR-020 | One Go module, standard library only, `go build` (ADR-0011). |
| NFR-021 | The package `count`; its place in the module is pending the bet. |
| NFR-022, NFR-024 | Only three counts, one command, no listener and no plug-in. |
| R-026 (peak heap) | The fixed buffer (ADR-0014). |
