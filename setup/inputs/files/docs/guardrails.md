# Guardrails — the known pitfalls, and the numbers you must not move

This document is the target of gate step 2, "Honor the guardrails", in
[`engineering-discipline.md`](engineering-discipline.md). It holds the pitfalls a
task author on tally must take into account **before** writing code, so the team
does not re-derive a known trap every time.

It merges two kinds of guardrail that many projects keep in separate files —
**decision gates** (pre-registered pass/fail rules) and **validation** (how you
check you are not fooling yourself). tally keeps them together; the rule is that
both exist and both are read before work starts. The rules came from Armature, the
baseline pinned in [`setup/armature.pin`](setup/armature.pin); the invariants and the
numbers are tally's own, and each cites the fact it comes from.

## In plain terms

It is easy to build a counter that gives the right answer on every file anyone
tries and the wrong one on an empty file, a file with no final line feed, a
no-break space or a 1 GiB file; the defense is to write the expected counts and the
pass/fail numbers down, from the facts, before the code exists.

## 1. Pre-registered decisions — or the goalposts move

A decision rule chosen **after** seeing the result is a fitted parameter, not a
rule. Write the pass/fail numbers first, somewhere they cannot be quietly edited.

- **What must be pre-registered:** the three quality targets of tally — the time
  for a file of 10 MiB, the peak heap for a file of 1 GiB, and the statement
  coverage of the package `count` — and the expected output of every behaviour
  test (F-0001#15).
- **Where the numbers freeze:** in the facts — the
  [problem statement](facts/problem-statement-brief.md) and
  [`F-0001`](facts/F-0001-setup-answers.md) — and, once it exists, in the PRD that
  derives from them. A number is moved only by a new fact from the idea owner,
  never by an edit to make a run pass.
- **The bands, not a single line:** prefer **Pass / Investigate / Fail** to a
  single pass line on a noisy measure. The time target is noisy; define
  "Investigate" before you look — one re-run on the reference computer, with its
  scope fixed in advance; landing there twice counts as Fail.

### The frozen numbers

| Measure | Pass | Investigate | Fail | Source |
|---------|------|-------------|------|--------|
| Elapsed time of the whole process, start-up included, for 10 MiB of ASCII text with lines of about 80 characters, on 4 cores with a solid-state disk and a warm file cache | at most 2.0 s | above 2.0 s on one run only | above 2.0 s twice | F-0001#9 |
| Peak heap (`HeapAlloc`, sampled every 10 ms during the count) for a file of 1 GiB | at most 64 MiB | — | above 64 MiB | F-0001#10 |
| Statement coverage of the package `count` (`go test -cover`) | at least 90 % | — | below 90 % | F-0001#11 |

### Invariants

Each invariant is a behaviour tally must keep. Its `Check` line names the mechanism
that guards it today; `no check yet` is the honest value until a Go test exists.

- **Inv-1** — Counts always print in the fixed order lines, words, characters,
  whatever the order of the flags; combined flags such as `-lw` mean `-l -w`
  (F-0001#2, F-0001#3).
  Check: no check yet
- **Inv-2** — An unknown flag or wrong usage prints a usage message on standard
  error, counts nothing, and exits with the code 2; the name `-` is a file named
  `-`, never standard input (F-0001#3).
  Check: no check yet
- **Inv-3** — A file that is not valid UTF-8 gets no row and adds nothing to the
  total; a message on standard error names it, rows already printed stay, the
  other files are still counted, and the exit code is 1 (F-0001#4).
  Check: no check yet
- **Inv-4** — A file that cannot be read, a directory included, gives a message on
  standard error with its name and the reason, adds no row, and makes the exit code
  1 while the other files are still counted (F-0001#8).
  Check: no check yet
- **Inv-5** — An empty file counts zero lines, zero words and zero characters; a
  line feed alone counts as one line; a carriage return, a line feed and a byte
  order mark each count as one character (F-0001#6, F-0001#7).
  Check: no check yet
- **Inv-6** — The row for standard input has no name: it ends after the counts
  (F-0001#5).
  Check: no check yet
- **Inv-7** — The counting code lives in the package `count`, separate from the
  package `main`, and that package keeps at least 90 percent statement coverage
  (F-0001#11).
  Check: no check yet
- **Inv-8** — Every in-tree link and heading anchor of the documents resolves, so a
  reader who follows a citation to a fact lands on it.
  Check: docs/links/link-lint.sh (ci:link-lint)

## 2. Known pitfalls — the traps specific to this domain

The failure modes that hurt text counters. For each: the trap, why it is silent,
and the check that catches it.

- ❌ **The last line and the empty file.** Counting line feeds gives one line too
  few for a file with no final line feed; counting "line feeds plus one" gives one
  line for an empty file. Both look right on every everyday file. The check: unit
  tests for `""`, `"\n"`, `"a"` and `"a\n"`, written first (F-0001#6).
- ❌ **ASCII-only white space.** Splitting words on space, tab and line feed alone
  misses the other `White_Space` characters, such as the no-break space, so two
  words count as one. It is silent because test files are usually ASCII. The
  check: a unit test with each `White_Space` code point between two words, using
  the Unicode test of the Go standard library, not a hand-written set.
- ❌ **A UTF-8 sequence split across reads.** tally must stream to stay under 64 MiB
  of heap, and a multi-byte sequence can straddle two chunks; a counter that
  decodes each chunk on its own reports a valid file as invalid or counts a code
  point twice. It is silent because small test files fit in one chunk. The check:
  a test that feeds the same text through a reader that returns one byte at a time
  and compares the counts with a single read (F-0001#10).
- ❌ **Reading the whole file.** Reading a file into memory before counting meets
  every functional test and breaks the heap limit only on a large file. The check:
  the 1 GiB peak-heap test, sampling during the count rather than reading the heap
  once at the end, since a read after counting can miss the peak (F-0001#10).
- ❌ **A row printed before validation fails.** If a file's row is printed while it
  is still being read, an invalid byte late in the file leaves a row that should
  not exist and a total that includes it. The check: a test with an invalid byte at
  the end of a file, asserting no row and an unchanged total (F-0001#4).
- ❌ **A time test that measures less than the process.** A benchmark of the
  counting function alone leaves out process start-up, which the 2-second target
  includes. The check: the time test runs the built command as a separate process
  and measures from start to exit (F-0001#9).

### Writing a lesson back (project-wide — keep this)

A trap caught once should not be re-derived by the next task, so a lesson does not
stay on the issue that learned it. When a task ends, its author asks whether the task
taught a trap the next reader could hit; if it did, the lesson is written **here** as a
new `❌` pitfall — the trap, why it is silent, and the check that catches it — in the
same pull request. Gate step 7 asks the question, so the rule is applied rather than
merely written (see [Keeping documentation current](engineering-discipline.md#keeping-documentation-current)).

This is the one **cross-task** reach the baseline adds on purpose.
[R6](issue-workflow.md#r6--agent-to-agent-communication-through-the-issue) and
[R7](issue-workflow.md#r7--decision-transparency-on-every-action) already keep the
coordination and the reasoning on the issue, and
[Honesty and evidence](engineering-discipline.md#honesty-and-evidence) already reports a
failure as a failure — but each is scoped to *one* issue thread. A lesson on issue #N is
discoverable only by someone who reads #N; §2 is where it reaches issue #N+1.

**The filter — or §2 grows until nobody reads it.** Write back only a trap that would
**catch the next reader**: a silent failure mode, a check that looked green for the
wrong reason, a footgun in the baseline or the domain. Do **not** write back a one-off with
no general lesson, a restatement of a rule that already lives elsewhere, or the
blow-by-blow of the task — those belong to the issue thread and the commit history.
Volume is the failure mode here, not absence: a pitfall list nobody finishes reading
guards nothing.

### Gate pitfalls (project-wide — keep these)

The gate is only as real as the thing that runs it. These traps let it report
success without having done its job.

- ❌ **An absolute `core.hooksPath`.** Worktrees **share** `.git/config`, so an
  absolute path binds every worktree to one checkout's hooks. A check added on a
  branch then does not run on that branch's own commits: the hook reports success
  having run something other than what the branch says it runs. It is silent
  because the hook still runs, still passes, and still uses the *right* files —
  only the *set of checks* comes from elsewhere. **The check:** install with a
  **relative** path, `git config core.hooksPath .githooks`, which git resolves per
  working tree. The `pre-commit` hook's own block 0 then refuses to run when the
  resolved hooks directory lies outside the tree being committed to, and
  [`.githooks/tests/provenance-check.sh`](../.githooks/tests/provenance-check.sh)
  proves it against real worktrees, reporting how many cases it ran rather than a
  count written down here to go stale.
  **A relative path escapes just as surely:** `../elsewhere/.githooks` is as
  foreign as any absolute one, so the check resolves the value instead of trusting
  that relative means local.
  **No path at all is the same trap:** with `core.hooksPath` unset git falls back
  to `.git/hooks`, which a linked worktree reaches through the shared *common* git
  directory. So block 0 judges the resolved directory whatever set it, and names
  the source it actually found — telling an operator to fix a setting they never
  set is its own dishonest report.
  **Bound on the damage:** CI invokes each check script directly and never through
  `core.hooksPath`, so this costs a local round trip, not a landed bug — a
  developer-experience gap, not an open gate.
- ❌ **A check that cannot fail.** A grep whose pattern also matches its own error
  message, a fixture harness that compares only exit codes, a coverage floor that
  counts zero as success. The check: for every assertion, make it fail on purpose
  once and read the reason — a green nobody attacked is not evidence.
- ❌ **A check that runs but does not block.** CI is green, the pull request
  merges, and nothing connects the two: no check is required on the default
  branch, so the green was a run result, not a merge control, and a red would
  have merged the same way. It is silent because the run result looks identical
  either way. The check:
  [make the checks required](ci/README.md#make-the-checks-required) on the
  default branch, and take the branch API read as the evidence — not the green run.

- ❌ **A check the change supplies is not a control.** CI checks out the pull
  request's own head and then runs the check from that checkout, so **the script
  that judges the change comes from the change**. Measured: a branch that replaces
  `docs/links/link-lint.sh` with `exit 0` passes that job — and replacing
  `docs/tests/run-discipline-tests.sh` as well turns **every** required job green
  over a dead link in the tree. Gutting a linter alone does not, because the
  fixture harness asserts exit codes and 19 `bad-*` cases stop failing; the runner
  is the single point.
  **The trap inside the remedy:** each script roots its scan at its own directory
  (`dirname $0`), so running the default branch's copy *where it sits* lints the
  wrong tree and reports OK. That was measured too, while building the fix.
  **The check:** every job in [`ci.yml`](../.github/workflows/ci.yml) restores the
  check scripts from the default branch **in place** before running them, so the
  branch's copy is never the judge.
  **Bound on the damage, and it is not nil:** a `pull_request` event runs the
  workflow as the branch has it, so a branch that edits `ci.yml` removes the
  restore step — closed only by review of `.github/**`, which wants a `CODEOWNERS`
  entry and a second human the forge knows about. Restoring also stops a bypass,
  not a merge: once a weakened check lands it *is* the default branch's copy. And
  a change that *improves* a check is judged by the older copy, so it lands in two
  steps. These checks are a control against forgetting, not against an operator
  who edits the check.

### Testing pitfalls (project-wide — keep these)

These traps are not domain-specific: they hurt every project's test suite, so the
baseline ships them filled. Keep them, and add your own above.

- ❌ **Testing after the code.** A test written to fit code that already "works"
  tends to encode the code's bugs as expected behaviour. The check: write the test
  first and watch it fail for the right reason
  ([strict TDD](engineering-discipline.md#requirements-traceability)).
- ❌ **Tests that depend on external state.** A test that reads a shared database, a
  live network, the wall clock, or another test's leftovers passes or fails for
  reasons unrelated to the code. The check: isolate and control every dependency,
  with a fresh fixture per run — see
  [`tests/scaling-checklist.md`](tests/scaling-checklist.md).
- ❌ **Tests that pass for the wrong reason.** A test that asserts nothing, asserts
  the wrong thing, or never actually exercises the path reports a safety that is not
  there — worse than no test. The check: confirm the test fails when the behaviour
  is broken; the red step is the proof.
- ❌ **Stale tests after a requirement changes.** When a requirement changes but its
  test does not, the suite now guards the old behaviour and blocks the new. The
  check: the [old-tests conflict rule](engineering-discipline.md#testing) — fix the
  code, update the requirement with a written reason, or retire the test; never
  weaken a passing old test.
- ❌ **Tests that slow down as the project grows.** A suite that creeps past the
  hook's patience gets skipped, and a skipped gate is no gate. The check: keep the
  cheap levels fast and cheap-first, push slow ones to CI, and bound each with
  the test timeout of [`tests/test-levels.md`](tests/test-levels.md) — see [`tests/scaling-checklist.md`](tests/scaling-checklist.md).

### Reference-sweep pitfalls (project-wide — keep these)

A change that edits references or a rule's wording across the tree has three silent
failure modes worth keeping.

- ❌ **A blanket find-and-replace over a renamed record's citations.** When a record
  moves or a directory is renumbered, the same bare token can name *different*
  records in two places — a bare `ADR-0005` is the living `docs/adr/` record to one
  reader and the archived `docs/decisions/` one to another, because the two sequences
  once shared numbers. A global replace of the token silently rewrites the citations
  you must **not** touch alongside the ones you must; and the reverse — a citation the
  sweep's pattern never matched (a compound like `ADR-0003/0005`, a token in a code
  span or a `.sh`/`.yml` comment, one split across a line break) — is silently *left*
  pointing at the wrong record. It is silent because **no linter catches it**:
  `link-lint` checks only that a *link* resolves, and a bare textual mention resolves
  to nothing, so a citation that now sends a reader to the wrong record still passes
  every check. **The check:** classify each occurrence by its **link target**, not its
  token — a link into `../adr/` is the living record and stays, a link into
  `../decisions/` is the archive and is rewritten — and read every *bare* mention by
  hand, in every token shape, since it carries no path to classify it. A
  pre-registered grep that must finish returning only the intended survivors (the
  mapping table and deliberate historical prose) is the closest thing to a gate; run
  it against the whole tree, not only the files you expected to touch.
- ❌ **A repoint that orphans a bare back-reference.** Repointing a citation can
  strand a *different* reference that named the target only through it. A comment
  reading `section 6 says …` leaned on a nearby `D-0003 section 6` for its antecedent;
  repoint every `D-0003 section 6` and the bare `section 6` is left pointing at a
  structure only the deleted record holds — wrong on this project's tree, and sharing
  **no token** with the thing you renamed. It is silent because a grep keyed on the
  obvious token (`D-000N`) cannot match a bare `section 6`, so the pre-registered
  check goes green over the survivor. **The check:** grep for the *shapes* a reference
  takes, not only the token — a bare `section N`, a `§`, a pronoun (`that section`,
  `the record`) whose antecedent you removed — and read the neighbourhood of every
  citation you changed, not the citation alone.
- ❌ **Editing a rule whose decision record is archived.** A rule lives in two places
  — its operative statement in a living doc, and the immutable decision record that
  first set it under `docs/decisions/`. Change the living one and the archived one
  still asserts the old, and **no check compares them** (`adr-lint` never reads
  `docs/decisions/`; `link-lint` checks resolution, not agreement). You cannot rewrite
  the immutable body to match; discharge the divergence with a `Status`-line
  **amendment pointer** on the archived record. **The check:** grep the whole tree —
  archive and forge templates included — for the old wording, and reconcile each living
  mirror or point each immutable one; a dated log entry recording history stays.
- ❌ **A hand-mirrored count or check-set that no linter guards.** The set of discipline
  linters — and how many there are — is spelled out by hand across many living
  docs, among them [`engineering-discipline.md`](engineering-discipline.md), this file,
  [`tests/test-levels.md`](tests/test-levels.md), the two `tests/README.md` files,
  [`ci/README.md`](ci/README.md) and
  [`.githooks/README.md`](../.githooks/README.md). Add or remove a check and every one
  can go stale, and a **removed** check leaves its name behind as a linter that no longer
  exists — a `link-lint` run stays green, because it resolves a *link*, not a claim. It is
  silent because the sentence still reads well and the count still looks deliberate: the
  baseline once said `three`, `four` and `five` at once, and named an `agent-entry` linter that
  had been cut. **The check:** when you add or remove a discipline check, grep the whole
  tree for the check-set enumeration — the old name and each spelled count — and reconcile
  every living mirror in the same change; the immutable ADR and archived decision copies
  stay as history.

## 3. Validation — how you check you are not fooling yourself

A result is **untrusted** until it passes the checks below, and the pass is a
recorded event, not a memory. Order the checks cheap-first, so a failure stops the
expensive ones.

| # | Check | Pass condition | Cost |
|---|-------|----------------|------|
| 1 | The discipline linters and the Go formatter and vet over the tree | every check prints OK, with no finding | seconds |
| 2 | The unit and integration tests of tally, with statement coverage of the package `count` | every test passes and coverage is at least 90 percent | under a minute |
| 3 | The quality-target tests: 10 MiB elapsed time and 1 GiB peak heap | both inside the Pass band of the frozen numbers | minutes |

Notes on how to read a failure: check 1 catches broken links, malformed records
and code that does not compile cleanly, and is cheap enough for the commit hook;
check 2 catches a wrong count or a wrong exit code, and runs in the hook as a fast
subset and in CI in full; check 3 catches a streaming or performance defect, runs
in CI, and runs again on the reference computer before a change that touches
reading or counting lands.

**The automated gate is this validation layer, mechanized.** The cheap, always-on
checks — the [discipline linters](engineering-discipline.md#testing) the baseline
ships (ADR, PRD and link) and their
[fixture self-tests](engineering-discipline.md#testing), the
[test levels](engineering-discipline.md#testing), lint, a security
scan, and the [commit-format](engineering-discipline.md#commit-messages)
check — run in the [`pre-commit` hook](engineering-discipline.md#git-hooks) for
fast local feedback and in [CI](engineering-discipline.md#continuous-integration-opt-in)
as the authority. Treat those checks as pre-registered pass/fail rules under
section 1: they predate any single result and are not edited to make a change
pass. Wire the "cheap enough to wire into CI" checks from the table above into
both layers.

## 4. Mechanics

- **Frozen rules do not get edited.** Changing a guardrail after it is set means a
  new version with a written reason, the old one preserved. Legitimate reasons
  exist (a bug in the measure); silent edits do not.
- **This document holds the structure; the facts hold the frozen values.** The
  numbers above mirror [`F-0001`](facts/F-0001-setup-answers.md) and the
  [problem statement](facts/problem-statement-brief.md); where they differ, the
  facts win and this file is corrected in the same change.

## Sources

- [The problem statement](facts/problem-statement-brief.md) — behaviour rules 1 to 7
  and the quality targets.
- [`F-0001`](facts/F-0001-setup-answers.md) — the idea owner's answers on flags,
  invalid input, empty lines, the reference computer and the peak heap.
- The Unicode `White_Space` property, as the Go standard library exposes it.
