# SPEC-0002. The architecture of tally

Date: 2026-10-22

This specification states how tally is built: its build, its packages, the
interface of the package `count`, its parser, its reading of a file and its check
of UTF-8. It serves the requirements of [PRD-0001](../prd/PRD-0001-tally.md) and the
behaviour of [SPEC-0001](SPEC-0001-tally.md), and it derives from the same facts. It
builds nothing.

Seven decisions were filtered against the requirements, the facts, the invariants
of [`guardrails.md`](../guardrails.md) and ADR-0001 to ADR-0009. Five have one
survivor: each is decided here and recorded as an ADR under
[ADR-0010](../adr/0010-record-product-architecture-decisions.md). Two have more than
one survivor: each has a section "Decision pending the bet" that lists its survivors
and decides nothing.

| Decision | Survivors | State | Record |
| -------- | --------- | ----- | ------ |
| Build tool | go-build-only | Decided | [ADR-0011](../adr/0011-build-with-go-build-only.md) |
| Interface of `count` | three-functions-teed-reader | Decided | [ADR-0012](../adr/0012-count-with-three-functions-over-a-teed-reader.md) |
| Flag parsing | hand-written-parser | Decided | [ADR-0013](../adr/0013-parse-flags-by-hand.md) |
| Read strategy | stream-fixed-buffer | Decided | [ADR-0014](../adr/0014-stream-through-a-fixed-buffer.md) |
| UTF-8 validation | validate-in-stream-print-after-eof | Decided | [ADR-0015](../adr/0015-validate-utf8-in-the-stream.md) |
| Module layout | root-main-count-subpackage, cmd-tally-count-subpackage | Pending the bet | [Module layout](#module-layout) |
| White-space class | unicode-isspace, unicode-white-space-table | Pending the bet | [White space](#white-space) |

## Build

tally is built with the Go toolchain's `go build` only. `go.mod` has no `require`
directive; there is no Makefile and no other build tool; every import is a package of
the Go standard library. (NFR-018, NFR-019, NFR-020;
[ADR-0011](../adr/0011-build-with-go-build-only.md))

## The package `count`

The package `count` exports three functions:

```go
func Lines(r io.Reader) (int64, error)
func Words(r io.Reader) (int64, error)
func Chars(r io.Reader) (int64, error)
```

`Lines` counts by REQ-009, `Words` by REQ-010 and `Chars` by REQ-011 of
SPEC-0001. Each reads `r` to its end through a fixed buffer, keeps only its count and
the state that spans two reads, and returns an error when a read fails or when the
text is not valid UTF-8. (NFR-021;
[ADR-0012](../adr/0012-count-with-three-functions-over-a-teed-reader.md),
[ADR-0014](../adr/0014-stream-through-a-fixed-buffer.md),
[ADR-0015](../adr/0015-validate-utf8-in-the-stream.md))

## The command

The package `main` does these steps:

1. **Parse the arguments** by hand, from left to right. Before the first file name:
   `-` is a file name; an argument that starts with `-` and has one character or more
   after it is a flag group of `l`, `w` and `c`, and any other character in it is an
   unknown flag (usage message on standard error, nothing on standard output, exit
   code 2); any other argument is a file name. After the first file name: an
   argument that does not start with `-`, and `-` itself, is a file name; an argument
   that starts with `-` and has one character or more after it is not decided, and
   the code task neither tests nor implements it until a fact answers it. With no
   flag, all three counts are selected. (REQ-002 to REQ-007;
   [ADR-0013](../adr/0013-parse-flags-by-hand.md))
2. **Open each source** in the order of the names, or standard input when there is
   no name. A name that cannot be opened or read, a directory included, gives a
   message with the name and the reason on standard error and no row. (REQ-004,
   REQ-008, REQ-014, REQ-015)
3. **Read the source once** through a fixed buffer, and tee the stream to the
   functions of the selected counts. (REQ-001;
   [ADR-0012](../adr/0012-count-with-three-functions-over-a-teed-reader.md),
   [ADR-0014](../adr/0014-stream-through-a-fixed-buffer.md))
4. **Print the row after the end of the source**, only when every selected function
   returned with no error: the selected counts in the order lines, words,
   characters, then the name, separated by one space; the row of standard input has
   no name. Only then add the counts to the total. On the UTF-8 error, print a
   message with the name on standard error, no row and nothing in the total.
   (REQ-012, REQ-016; [ADR-0015](../adr/0015-validate-utf8-in-the-stream.md))
5. **Print the total row** `total` when there are two names or more, then exit with
   1 when any source gave an error, else 0. (REQ-013, REQ-014, REQ-017)

## Module layout

One Go module holds the package `main` of the command and the package `count`,
separate from it (NFR-021). The single-package layout failed NFR-021 and Inv-7. Two
layouts survive the filter.

### Decision pending the bet

The survivors, in no order:

- **root-main-count-subpackage** — the package `main` sits at the root of the module,
  and the package `count` in the directory `count/`.
- **cmd-tally-count-subpackage** — the package `main` sits in the directory
  `cmd/tally/`, and the package `count` in the directory `count/`.

This specification decides nothing between them. The bet decides; its decision then
becomes an ADR, and this section states the decided layout.

## White space

`Words` splits on every code point with the Unicode property `White_Space`, not only
the ASCII space, tab and line feed (REQ-010). The ASCII-only set failed REQ-010. Two
classes of the Go standard library survive the filter.

### Decision pending the bet

The survivors, in no order:

- **unicode-isspace** — the test `unicode.IsSpace(r)`.
- **unicode-white-space-table** — the test `unicode.Is(unicode.White_Space, r)`.

This specification decides nothing between them. The bet decides; its decision then
becomes an ADR, and this section states the decided class.

## Open question

- After the first file name, an argument that starts with `-` and has one character
  or more after it, such as `-l` in `tally file -l`: no fact says whether it is a
  flag, a file name or wrong usage. It goes to the idea owner on the issue
  ([ADR-0013](../adr/0013-parse-flags-by-hand.md)).
