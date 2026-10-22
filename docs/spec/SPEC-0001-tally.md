# SPEC-0001. The specification of tally

Date: 2026-10-22

This specification states how tally behaves, one section per requirement of
[PRD-0001](../prd/PRD-0001-tally.md). It derives from the same facts: the problem
statement (F-0003, rendered at the first bet), the marker answers
([F-0002](../facts/F-0002-marker-answers.md)) and the idea owner's answers Q-001 to
Q-014. It specifies the interfaces only; it builds nothing.

## Interfaces

- **The command line.** `tally [-l] [-w] [-c] [file...]`. The exit code is 0 when
  no error occurs, 1 when a named file cannot be read or is not valid UTF-8, and 2
  on an unknown flag or wrong usage.
- **The package `count`.** A package of the same Go module as the command,
  separate from its package `main`. Its exported functions count the lines, the
  words and the characters of an `io.Reader`. The command calls them.

## Terms

- **Line feed** — the code point U+000A.
- **White space** — a code point with the Unicode property `White_Space`.
- **Row** — one line of standard output: the selected counts, then a name.

## REQ-001 — Count each named file and print a total

tally reads each file that the user names and prints its number of lines, words
and characters. With two files or more it also prints a total row (see REQ-013).
Test: two named text files give one row each and a total row. (F-0003#5)

## REQ-002 — The command line

The command is `tally [-l] [-w] [-c] [file...]`. It accepts zero, one or more file
names, with or without flags. With zero names it reads standard input (REQ-008).
Test: zero, one and three names, each with and without flags, give output and no
usage error. (F-0003#6)

## REQ-003 — Combined flags

Flags may be combined in one argument: `-lw` means `-l -w`, and any combination of
`l`, `w` and `c` after one `-` is allowed. Test: `tally -lw file` and
`tally -l -w file` print the same output. (F-0003#6, Q-002)

## REQ-004 — The name `-` is a file

The argument `-` names a file called `-`. It never means standard input. Test: in
a directory with a file `-`, `tally -` prints its counts and the name `-`; with no
such file, tally reports an unreadable file (REQ-014) and exits with 1.
(F-0003#6, Q-002)

## REQ-005 — Usage errors

An unknown flag, or any other wrong usage, prints a usage message on standard
error. tally then counts nothing, prints nothing on standard output and exits with
2. Test: `tally -x file`. (F-0003#6, Q-002)

## REQ-006 — The flags `-l`, `-w` and `-c`

`-l` selects the line count, `-w` the word count and `-c` the character count. A
row holds only the selected counts. Test: each flag alone on a known file prints
only its count, then the file name. (F-0003#7)

## REQ-007 — No flag selects all three counts

With no flag, a row holds the line, word and character counts, in that order, then
the name. Test: `tally file`. (F-0003#8)

## REQ-008 — Standard input

With no file name, tally reads standard input and prints one row of its counts.
This row has no name: it ends after the last count, with no trailing space. Test:
pipe a known text into `tally`. (F-0003#9, Q-004)

## REQ-009 — What a line is

A line is a run of characters that ends with a line feed. The last line counts when
the file does not end with a line feed. A line feed alone ends an empty line, which
counts as one line. A carriage return before a line feed is part of the line and
counts as a character. An empty file has 0 lines, 0 words and 0 characters. So the
line count is the number of line feeds, plus one when the text is not empty and
does not end with a line feed. Test: `a\nb\n` and `a\nb` give 2 lines; `\n\n`
gives 2 lines; `a\r\nb\r\n` gives 2 lines and 6 characters. (F-0003#13, F-0003#14,
Q-005)

## REQ-010 — What a word is

A word is a run of characters that are not white space. White space is every code
point with the Unicode property `White_Space`, not only the ASCII space, tab and
line feed. Test: words separated by space, tab, line feed, U+00A0 and U+3000 count
one word per run. (F-0003#15)

## REQ-011 — What a character is

A character is one Unicode code point of the UTF-8 text, whatever its length in
bytes. The line feed is a character. A byte order mark at the start of a file is
one character, and tally does not strip it. Test: `aé€😀` gives 4 characters;
`ab\n` gives 3; the bytes `EF BB BF` then `a` give 2. (F-0003#16, Q-006)

## REQ-012 — The shape of a row

Each file has one row: the selected counts, then the file name, each value
separated from the next by one space. The counts are always in the order lines,
words, characters, whatever the order of the flags. The rows follow the order of
the names on the command line. Test: `tally -l -w a b` prints `<lines> <words> a`
then `<lines> <words> b`; `tally -c -l file` and `tally -l -c file` both print
`<lines> <characters> file`. (F-0003#17, Q-001)

## REQ-013 — The total row

With two files or more, the last row holds the sum of each selected count, then the
name `total`. A file that gives an error (REQ-014, REQ-015, REQ-016) adds nothing
to it. With one file, there is no total row. Test: two files end with
`<sums> total`; one file has no total row. (F-0003#18)

## REQ-014 — A file that cannot be read

When a named file cannot be read, tally prints a message on standard error with
the file name and the reason, prints no row for it, and goes on with the next file.
At the end it exits with 1. Test: a missing file between two readable files.
(F-0003#19, F-0003#20)

## REQ-015 — A directory

A directory named on the command line is an unreadable file under REQ-014: a
message on standard error with its name and the reason, no row, and the exit code
1. tally does not read the files inside it. Test: a directory between two readable
files. (F-0003#11, F-0003#19, Q-007)

## REQ-016 — A file that is not valid UTF-8

tally checks the encoding while it reads a file, before it prints the file's row.
When the file is not valid UTF-8, tally prints a message on standard error with the
file name, prints no row for it, adds nothing to the total, and goes on with the
next file. Rows already printed for earlier files stay. At the end it exits with 1.
Test: a valid file, a file holding the byte `0xFF`, and a valid file.
(F-0003#21, Q-003)

## REQ-017 — Success exits with 0

When no error occurs, tally exits with 0, for named files and for standard input.
Test: a run over valid files and a run over standard input. (F-0003#22)

## NFR-018 — Go

All product source code is Go. (F-0003#1)

## NFR-019 — The standard library only

The module has no third-party dependency: `go.mod` has no `require` directive, and
every import is a package of the Go standard library. (F-0003#2)

## NFR-020 — The Go toolchain

`go build` builds the tally binary. No other build tool is needed. (F-0003#3)

## NFR-021 — The package `count`

The counting code is the package `count` of the same Go module, separate from the
package `main` of the command. It exports the functions that count the lines, the
words and the characters of an `io.Reader`, by the rules of REQ-009, REQ-010 and
REQ-011. The command calls these functions. The coverage target of 90 percent of the statements applies to
this package only. (F-0003#25, Q-010)

## NFR-022 — Only lines, words and characters

tally counts lines, words and characters only. It has no flag and no output for
bytes, paragraphs, sentences or any other count. (F-0003#10)

## NFR-023 — No archives, directories or non-text files

tally does not read archives, does not recurse into directories and does not
decode formats that are not text. A file is not text exactly when it is not valid
UTF-8, and REQ-016 rejects it. (F-0003#11, Q-007)

## NFR-024 — A command-line tool only

tally has no graphical interface, opens no network listener and loads no plug-in.
(F-0003#12)
