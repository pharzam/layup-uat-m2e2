# tally — specification

The behaviour of each requirement of [PRD-0001](../prd/PRD-0001-tally.md), written
from the problem statement (F-0003) and the idea owner's answers (Q-001 to Q-010).
Every section names its requirement ID; the PRD holds the statement, its facts and
its acceptance criterion.

## Interfaces

The command line is `tally [-l] [-w] [-c] [file...]`, with the exit codes 0 (no
error), 1 (a file could not be read or is not valid UTF-8) and 2 (wrong usage). A
Go package `count`, separate from the package `main` of the command, holds the
functions that count the lines, the words and the characters of an `io.Reader`.
This specification states them only; it builds nothing.

## REQ-001 — Count each named file and a total

tally reads each file named on the command line and prints, for each, its number
of lines, words and characters, followed by a total (see REQ-013 for when the total
row appears).

## REQ-002 — Command-line form

The command is `tally [-l] [-w] [-c] [file...]`. It accepts zero, one or more file
names, each with or without flags, without a usage error.

## REQ-003 — Combined flags

Flags may be combined in one argument: `-lw` means the same as `-l -w`, and any
combination of the letters l, w and c is read the same way.

## REQ-004 — The name - is a file

The argument `-` is not standard input. tally opens a file named `-` and prints
its row ending with the name `-`; when no such file exists, it is an unreadable
file under REQ-014 and the exit code is 1.

## REQ-005 — Usage errors

An unknown flag, such as `-x`, or any other wrong usage prints a usage message on
standard error, prints nothing on standard output, counts nothing and exits with
the code 2.

## REQ-006 — Flags select counts

`-l` selects the line count, `-w` the word count and `-c` the character count. Only
the selected counts are printed.

## REQ-007 — Default counts

With no flag, tally prints all three counts: lines, words and characters, in that
order.

## REQ-008 — Standard input

With no file name, tally reads standard input and prints one row of counts with no
name after them: the row ends after the last count.

## REQ-009 — What a line is

A line is a run of characters ended by a line feed. A last line not ended by a line
feed still counts. A line feed alone ends an empty line, which counts as one line.
A carriage return before a line feed is part of the line and counts as a character.
An empty file has 0 lines, 0 words and 0 characters. So `"a\nb\n"`, `"a\nb"` and
`"\n\n"` each count 2 lines, and `"a\r\nb\r\n"` counts 2 lines and 6 characters.

## REQ-010 — What a word is

A word is a run of characters none of which is white space, where white space is
any character with the Unicode property `White_Space` — space, tab, line feed,
no-break space (U+00A0) and ideographic space (U+3000) among them.

## REQ-011 — What a character is

A character is one Unicode code point of the UTF-8 text, whatever its byte length:
`"aé€😀"` counts 4. The line feed counts as a character. A byte order mark at the
start of a file counts as one character and is not stripped.

## REQ-012 — Output rows

Each file gives one row: the selected counts, then the file name, each value
separated from the next by one space. Rows appear in the order the files are
given. The counts are always in the order lines, words, characters, whatever the
order of the flags: `tally -c -l file` and `tally -l -c file` both print
`<lines> <characters> file`.

## REQ-013 — Total row

With two or more files, the last row is the sum of each selected count followed by
the name `total`. With one file, or with standard input, no total row is printed.

## REQ-014 — Unreadable file

A file that cannot be read gives a message on standard error with the file's name
and the reason. The remaining files are still counted, and the exit code is 1.

## REQ-015 — Directory

A directory named on the command line is an unreadable file under REQ-014: a
message on standard error with its name and the reason, no row, and the exit code
1, while the other files are still counted.

## REQ-016 — File that is not valid UTF-8

A file that is not valid UTF-8 gives a message on standard error with its name, and
the exit code is 1 at the end. The check happens while the file is read, before
its row would be printed: no row is printed for it and it adds nothing to the
total. The other files are still counted, and rows already printed for earlier
files stay.

## REQ-017 — Success exit code

When no error occurs, over named files or over standard input, tally exits with
the code 0.

## NFR-018 — Written in Go

All product source code is Go.

## NFR-019 — Standard library only

The product uses the Go standard library only: `go.mod` declares no `require`
directive and every import is a standard-library package.

## NFR-020 — Built with the Go toolchain

`go build` produces the tally binary with no other build tool.

## NFR-021 — Package count

The counting code lives in a package `count` of the same Go module, separate from
the package `main` of the command. It exports the functions that count the lines,
the words and the characters of an `io.Reader`, and the command calls them. The
coverage target of 90 percent of statements applies to this package only.

## NFR-022 — Only lines, words and characters

tally offers no flag or output for bytes, paragraphs, sentences or any count other
than lines, words and characters.

## NFR-023 — Text files only

tally offers no option to read archives, recurse into directories or decode
formats that are not text. A file is not text exactly when it is not valid UTF-8,
and it is then rejected under REQ-016.

## NFR-024 — Command line only

The product is a command-line tool only: it has no graphical interface, opens no
network listener and loads no plug-ins.
