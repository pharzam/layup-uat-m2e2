# PRD-0001. tally, a command that counts the lines, words and characters of text files

Date: 2026-10-22

## 1. Header & "In plain terms"

| Field        | Value |
| ------------ | ----- |
| PRD ID       | `PRD-0001` |
| Title        | tally, a command that counts the lines, words and characters of text files |
| Status       | `Draft` |
| Date         | 2026-10-22 |
| Author       | Task T-yzst (issue 3), an agent session |
| Derives from | F-0003 (the problem statement), the idea owner's answers Q-001 to Q-014, and F-0002 |

### In plain terms

> tally is a small command for people who work in a terminal: you name some text
> files and it tells you how many lines, words and characters each one holds, and
> the sum of them all.

## 2. Problem & evidence

A person who works in a terminal wants to know how long a text file is
(F-0003#4). The product `tally` reads the files that the person names and prints
the number of lines, words and characters of each, and a total (F-0003#5). It is
written in Go, with the standard library only, and built with the Go toolchain
(F-0003#1, F-0003#2, F-0003#3). The behaviour rules of the statement (F-0003#13 to
F-0003#22) and the idea owner's answers (Q-001 to Q-010) fix what counts as a line,
a word and a character, the shape of the output and the exit codes.

## 3. Goals & non-goals

**Goals.** Count the lines, words and characters of each named file, or of
standard input, and print them with a total; report an unreadable or invalid file
without stopping the count of the others; meet the time, memory and coverage
targets of §8.

**Non-goals.** Counts other than lines, words and characters, such as bytes,
paragraphs or sentences (NFR-022); reading archives, recursing into directories or
decoding files that are not text, where a file is not text exactly when it is not
valid UTF-8 (NFR-023); a graphical interface, a network service or a plug-in system
(NFR-024).

## 4. Personas & jobs-to-be-done

**Terminal user.** A person who works in a terminal and wants to know how long a
text file is (F-0003#4).

## 5. User stories

- As a terminal user, I name one or more files so that I see the lines, words and characters of each and a total (`F-0003#5`).
- As a terminal user, I pipe text into tally so that I count standard input (`F-0003#9`).
- As a terminal user, I pass -l, -w or -c so that I see only the count I need (`F-0003#7`).

## 6. Functional requirements

| REQ | Statement | MoSCoW | Phase | Facts |
| --- | --------- | ------ | ----- | ----- |
| REQ-001 | The command tally shall read each file the user names and print its number of lines, words and characters, and a total. |  |  | F-0003#5 F-0002 |
| REQ-002 | The command shall be invoked as `tally [-l] [-w] [-c] [file...]` and accept zero or more file names. |  |  | F-0003#6 F-0002 |
| REQ-003 | Combined flags such as -lw shall be accepted and mean the same as the separate flags -l -w. |  |  | F-0003#6 Q-002 F-0002 |
| REQ-004 | The name - shall not denote standard input: tally shall treat it as a file named -. |  |  | F-0003#6 Q-002 F-0002 |
| REQ-005 | An unknown flag or wrong usage shall print a usage message on standard error, count nothing, and end with the exit code 2. |  |  | F-0003#6 Q-002 F-0002 |
| REQ-006 | The flag -l shall select the line count only, -w the word count only and -c the character count only. |  |  | F-0003#7 F-0002 |
| REQ-007 | When no flag is given, tally shall print all three counts: lines, words and characters. |  |  | F-0003#8 F-0002 |
| REQ-008 | When no file name is given, tally shall read standard input and count it, and the row for standard input shall have no name: it ends after the counts. |  |  | F-0003#9 Q-004 F-0002 |
| REQ-009 | A line shall be counted as a run of characters that ends with a line feed, and the last line of a file shall count when the file does not end with a line feed; a line feed alone ends an empty line, which counts as one line, and a carriage return before the line feed is part of the line and counts as a character. |  |  | F-0003#13 F-0003#14 Q-005 F-0002 |
| REQ-010 | A word shall be counted as a run of characters that are not white space, where white space is any character with the Unicode property White_Space. |  |  | F-0003#15 F-0002 |
| REQ-011 | A character shall be counted as one Unicode code point of the UTF-8 text; the line feed counts as a character, and a byte order mark at the start of a file counts as one character and is not stripped. |  |  | F-0003#16 Q-006 F-0002 |
| REQ-012 | The output shall have one row for each file, holding the counts the flags select followed by the file name, each value separated from the next by one space, and the counts shall always appear in the fixed order lines, words, characters, whatever the order of the flags on the command line. |  |  | F-0003#17 Q-001 F-0002 |
| REQ-013 | With two files or more, the output shall end with a row named total that holds the sum of each selected count. |  |  | F-0003#18 F-0002 |
| REQ-014 | A file that cannot be read shall produce a message on standard error naming the file and the reason, the exit code shall be 1, and the remaining files shall still be counted. |  |  | F-0003#19 F-0003#20 F-0002 |
| REQ-015 | A directory named on the command line shall be treated as an unreadable file: tally shall print a message on standard error with its name and the reason, add no row for it, and end with the exit code 1. |  |  | F-0003#11 F-0003#19 Q-007 F-0002 |
| REQ-016 | A file that is not valid UTF-8 shall produce a message on standard error naming the file, and the exit code shall be 1 at the end; no row shall be printed for that file and it shall add nothing to the total, the other files shall still be counted, and rows already printed for earlier files shall stay. |  |  | F-0003#21 Q-003 F-0002 |
| REQ-017 | When no error occurs, tally shall exit with code 0. |  |  | F-0003#22 F-0002 |

| Requirement | Acceptance criterion |
| ----------- | -------------------- |
| `REQ-001` | Given two named text files, tally prints the line, word and character counts of each file and a total row. |
| `REQ-002` | Invoking tally with zero, one and three file names, each with and without flags, is accepted and produces output without a usage error. |
| `REQ-003` | For a known file, `tally -lw file` prints exactly the same output as `tally -l -w file`. |
| `REQ-004` | In a directory holding a file named "-", `tally -` prints that file's counts followed by the name "-" and does not read standard input; with no file named "-", it reports an unreadable file and exits with code 1. |
| `REQ-005` | `tally -x file` prints a usage message on standard error, prints nothing on standard output, and exits with code 2. |
| `REQ-006` | For a known file, `tally -l` prints only the line count, `tally -w` only the word count and `tally -c` only the character count, each followed by the file name. |
| `REQ-007` | For a known file, `tally file` prints the line, word and character counts, in that order, followed by the file name. |
| `REQ-008` | Piping a known text into `tally` with no file name prints one row holding the correct line, word and character counts of that text, separated by single spaces, with nothing after the last count. |
| `REQ-009` | A file "a\nb\n" counts 2 lines, a file "a\nb" counts 2 lines, a file "\n\n" counts 2 lines, a file "a\r\nb\r\n" counts 2 lines and 6 characters, and an empty file counts 0 lines, 0 words and 0 characters. |
| `REQ-010` | A text separated by space, tab, line feed, no-break space (U+00A0) and ideographic space (U+3000) counts one word for each non-white-space run between them. |
| `REQ-011` | A file holding "aé€😀" (1, 2, 3 and 4 bytes per code point) counts 4 characters, a file "ab\n" counts 3 characters, and a file holding a byte order mark (EF BB BF) followed by "a" counts 2 characters. |
| `REQ-012` | For two files and the flags -l -w, each output row is exactly "<lines> <words> <name>" with single spaces, one row per file in the order given; `tally -c -l file` and `tally -l -c file` both print "<lines> <characters> file". |
| `REQ-013` | With two files, the last row is "<sums> total"; with one file, no total row is printed. |
| `REQ-014` | Given a missing file between two readable files, standard error holds a message with the missing file's name and the reason, the two readable files are counted on standard output, and the exit code is 1. |
| `REQ-015` | Given a directory between two readable files, standard error holds a message with the directory's name and the reason, no row is printed for the directory, the two readable files are counted, and the exit code is 1. |
| `REQ-016` | Given a valid file, a file holding the byte 0xFF, and another valid file, standard output holds the rows of the two valid files and a total that is their sum only, no row names the invalid file, standard error holds a message with the invalid file's name, and the exit code is 1. |
| `REQ-017` | A run over readable, valid UTF-8 files and a run over standard input both exit with code 0. |

## 7. Non-functional requirements

| REQ | Statement | MoSCoW | Phase | Facts |
| --- | --------- | ------ | ----- | ----- |
| NFR-018 | The product shall be written in Go. |  |  | F-0003#1 F-0002 |
| NFR-019 | The product shall use the Go standard library only, with no third-party dependency. |  |  | F-0003#2 F-0002 |
| NFR-020 | The product shall be built with the Go toolchain. |  |  | F-0003#3 F-0002 |
| NFR-021 | The counting code shall live in a package count of the same Go module, separate from the package main of the command, and that package shall hold the functions that count the lines, the words and the characters of a reader. |  |  | F-0003#25 Q-010 F-0002 |
| NFR-022 | The product shall count only lines, words and characters; other counts, such as bytes, paragraphs or sentences, are out of scope. |  |  | F-0003#10 F-0002 |
| NFR-023 | Reading archives, directories or files that are not text is out of scope, where a file is not text exactly when it is not valid UTF-8. |  |  | F-0003#11 Q-007 F-0002 |
| NFR-024 | The product shall be a command-line tool only, with no graphical interface, network service or plug-in system. |  |  | F-0003#12 F-0002 |

| Requirement | Acceptance criterion |
| ----------- | -------------------- |
| `NFR-018` | All product source code is Go. |
| `NFR-019` | go.mod declares no require directive and every import is a standard-library package. |
| `NFR-020` | `go build` produces the tally binary with no other build tool. |
| `NFR-021` | The module holds a package count, separate from package main, that exports the functions counting lines, words and characters of an io.Reader, and the command calls them. |
| `NFR-022` | tally offers no flag or output for bytes, paragraphs, sentences or any count other than lines, words and characters. |
| `NFR-023` | tally offers no option to read archives, recurse into directories or decode non-text formats, and a file is rejected as not text exactly when it is not valid UTF-8. |
| `NFR-024` | The product has no graphical interface, opens no network listener and loads no plug-ins. |

## 8. Success metrics

| Metric | Fact |
| ------ | ---- |
| The elapsed time of the whole tally process, from its start to its exit, to count a file of 10 MiB of ASCII text with lines of about 80 characters, already in the cache of the operating system: At most 2 seconds on any computer with 4 cores and a solid-state disk, measured by a test that generates a file of 10 MiB of ASCII text with lines of about 80 characters, reads it once to bring it into the cache of the operating system, then runs tally on it and measures the elapsed time of the whole process including start-up. | F-0003#23 Q-008 |
| The peak heap memory used during the count of a file of 1 GiB: At most 64 MiB of heap, measured by a test that counts a generated file of 1 GiB while sampling runtime.ReadMemStats HeapAlloc every 10 milliseconds and takes the largest sampled value. | F-0003#24 Q-009 |
| The test coverage of the package count: At least 90 percent of the statements of the package count only, as reported by `go test -cover`. | F-0003#25 Q-010 |
| The delivery's success criteria: tally prints the lines, words and characters of each named file and a total, as the statement's behaviour rules say. | Q-014 |
| The delivery's success criteria: a file that cannot be read or is not valid UTF-8 gives a message and the exit code 1 while the other files are still counted. | Q-014 |
| The delivery's success criteria: a 10 MiB file is counted in at most 2 seconds and a 1 GiB file with at most 64 MiB of peak heap. | Q-014 |
| The delivery's success criteria: the package count has at least 90 percent statement coverage. | Q-014 |

## 9. Rollout & phases

| Phase | Name | Ships |
| ----- | ---- | ----- |
| 1 | First release | The command tally with the flags -l, -w and -c, standard input, the total row and the exit codes 0, 1 and 2, and the package count that counts the lines, words and characters of a reader. |

## 10. Risks

- The time and memory targets of §8 depend on reading a file as a stream rather than whole; see [`../guardrails.md`](../guardrails.md) for the pre-registered pass and fail rules.
- The invalid-UTF-8 rule (REQ-016) drops a file's row after the file is read, so the counts of a file must not be printed or added to the total before its whole text is checked.

## 11. Open questions & assumptions

No open question stops this PRD. The MoSCoW priority and the phase of each
requirement are set at the first bet.

## 12. Requirements traceability matrix

One row per requirement in §6 / §7.

| REQ | Facts | Guardrail | ADR | Task | Test |
| --- | ----- | --------- | --- | ---- | ---- |
| REQ-001 | F-0003#5 F-0002 | — | — | — | — |
| REQ-002 | F-0003#6 F-0002 | — | — | — | — |
| REQ-003 | F-0003#6 Q-002 F-0002 | — | — | — | — |
| REQ-004 | F-0003#6 Q-002 F-0002 | — | — | — | — |
| REQ-005 | F-0003#6 Q-002 F-0002 | — | — | — | — |
| REQ-006 | F-0003#7 F-0002 | — | — | — | — |
| REQ-007 | F-0003#8 F-0002 | — | — | — | — |
| REQ-008 | F-0003#9 Q-004 F-0002 | — | — | — | — |
| REQ-009 | F-0003#13 F-0003#14 Q-005 F-0002 | — | — | — | — |
| REQ-010 | F-0003#15 F-0002 | — | — | — | — |
| REQ-011 | F-0003#16 Q-006 F-0002 | — | — | — | — |
| REQ-012 | F-0003#17 Q-001 F-0002 | — | — | — | — |
| REQ-013 | F-0003#18 F-0002 | — | — | — | — |
| REQ-014 | F-0003#19 F-0003#20 F-0002 | — | — | — | — |
| REQ-015 | F-0003#11 F-0003#19 Q-007 F-0002 | — | — | — | — |
| REQ-016 | F-0003#21 Q-003 F-0002 | — | — | — | — |
| REQ-017 | F-0003#22 F-0002 | — | — | — | — |
| NFR-018 | F-0003#1 F-0002 | — | — | — | — |
| NFR-019 | F-0003#2 F-0002 | — | — | — | — |
| NFR-020 | F-0003#3 F-0002 | — | — | — | — |
| NFR-021 | F-0003#25 Q-010 F-0002 | — | — | — | — |
| NFR-022 | F-0003#10 F-0002 | — | — | — | — |
| NFR-023 | F-0003#11 Q-007 F-0002 | — | — | — | — |
| NFR-024 | F-0003#12 F-0002 | — | — | — | — |

## 13. Change log

| Date       | Change | Requirement(s) affected |
| ---------- | ------ | ----------------------- |
| 2026-10-22 | First draft, from the revised draft of task T-yzst. | REQ-001 to REQ-017, NFR-018 to NFR-024 |
