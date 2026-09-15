# DATUM

The governing discipline for a family of data-platform repositories:
**baseline**, **traverse**, **meridian**. A data platform is credited by
what its gates have shown, never by what its author says, and a gate counts
only after it has been seen to fail.

`DATUM.md` states the discipline as twelve rules and the verdict row the
rules rest on. The conformance pack under `conformance/` is the part of
DATUM that can refuse: a schema for the row, a reference checker, the
checker's own negative controls, and a fixture corpus in which every case
carries its expected outcome and every negative case its expected reason.
Governed repos vendor the pack at a pinned commit and run it in CI over
their own emitted rows.

## Verify

```
node conformance/test.mjs --mutate   # the pack's own controls; each rule disabled in turn must be noticed
node conformance/check.mjs <dir>     # any directory of GATE_VERDICT rows: exit 0 conform, 1 refused, 2 unevaluable
node conformance/check.mjs <dir> --expect <file>   # also refuse a cell or twin set other than the expected one
```

`test.mjs` also builds the shape a governed repo ships (a copy under
`gates/conformance/` holding a `PIN` that `--write-pin` wrote), runs
`--verify-pin` alone and the self-test there, and re-runs the self-test in
copies with defects planted, one in each layout, requiring each defect to be
refused by name. Run here, in this repo's layout, its wording control reads
the full git history (a shallow clone fails it); CI checks out with
`fetch-depth: 0`. A `PIN` never sits in this repo's layout: one here is
refused, and so is a schema copy beside `check.mjs`. The layout is read
from `schema/gate-verdict.v1.json` one level up, or from git (a work tree
whose top holds `conformance/` and whose index tracks that schema), never
from a schema beside `check.mjs`; a control copies this repo with its
history, plants a schema beside `check.mjs`, a `PIN` that verifies and
HEAD's sha, and requires each to be refused. Fixture directories hold
`.json` cases and nothing else, and the wording control reads file and
directory names as well as file contents.

Crediting is per check: a (surface, lane) is CLAIMABLE only when every
check its live row reports has been set nonzero by some twin RED as
planted; otherwise it is PARTIAL and `--json` names the `unfalsified`
checks. A group with any UNEVALUABLE row, its live row included, derives
UNEVALUABLE and is not credited per check at all. The expect file is
`{"cells":[{"surface","lane","twins":[...]}]}` and never asserts a status.
Both are described in `conformance/reasons.md`.

## Adopting the pack

Every governed repo vendors the pack into **`gates/conformance/`**, the
same path everywhere (a Go module cannot use `vendor/`, the path first
proposed). The vendoring contract is the file set, the pin and the CI
order. A governed repo takes the pack in this order:

1. **Fetch a pushed commit, never a working tree** — a pin names a commit
   (see "Locked decisions" in `docs/2026-09-06-datum-design.md`). From a
   clone of this repo, into the governed repo's `gates/conformance/`,
   emptied first (a file left over from an earlier pin reads as unlisted):
   ```
   git archive <pushed datum sha> conformance schema/gate-verdict.v1.json \
     | tar -x -C <tmp>
   rm -rf <governed>/gates/conformance && mkdir -p <governed>/gates/conformance
   cp -r <tmp>/conformance/. <governed>/gates/conformance/
   cp <tmp>/schema/gate-verdict.v1.json <governed>/gates/conformance/
   ```
   The schema file goes beside `check.mjs`.
2. **`node gates/conformance/check.mjs --write-pin <that same sha>` —
   exactly that, and without a redirect.** Run it once, when vendoring. It
   writes `PIN` itself, atomically, and prints only `ok pin written`.
   `--write-pin` takes nothing but the sha: any other argument or flag
   beside it is a usage error (exit 2) that writes nothing. Never redirect
   its output (`> PIN`): the shell truncates `PIN` before node starts, so
   the file ends empty whatever `check.mjs` does, and on Windows the rename
   onto the redirect target fails as well (`--write-pin failed: EPERM`, exit
   2, its temp file removed; measured 2026-09-14, `STATUS.md`). Never
   hand-edit `PIN` either — `--verify-pin` checks it byte-for-byte and
   refuses an empty one.
3. `node gates/conformance/check.mjs --verify-pin` with no rows directory,
   **before** any vendored code runs: exit 0 printing `ok pin verified`, or
   exit 2 listing what is missing, unlisted or altered, and any `PIN` line
   that names something other than one of the pack's own files, once.
4. `node gates/conformance/test.mjs --mutate` in the vendored copy, with its
   `PIN` beside it — the shape the governed repo actually ships — so a pack
   that only conforms to itself at home is caught before anything
   downstream trusts it. In a vendored copy the wording control prints
   `note: wording: the commit-sha sub-check was not evaluable here (...)`:
   that one sub-check needs this repo's history and runs in this repo's CI
   instead, as does the control that plants files in a full-history copy
   (`note: layout: ...`). The notes are expected; the exit code is still 0
   only when every other control holds. The self-test leaves `PIN` out of
   its wording scan only when `--verify-pin` accepts it; a `PIN` that was
   edited, written by hand or left over from another pin fails the
   self-test and is scanned as authored text.
5. Emit the governed repo's own `GATE_VERDICT` rows into a directory
   **cleared first** — a stale row from a prior run reads identically to a
   live one.
6. `node gates/conformance/check.mjs <rows-dir> --verify-pin --expect <file>`
   — `--verify-pin` here confirms the rows-checking run is still against
   the pinned pack, not only the earlier `test.mjs` run; `--expect` names
   every `(surface, lane)` cell the rows must hold and the exact twin
   mutations each must carry, so a surface the emitter silently dropped, or
   a twin silently missing from a lane, is refused rather than passing
   quietly with fewer rows than intended. The expect file for meridian's 18
   rows at its commit `97d815c`:
   ```json
   {"cells": [
     {"surface": "meridian-lane1-p1", "lane": 1, "twins": ["naive_fold_no_dedupe"]},
     {"surface": "meridian-lane1-p2", "lane": 1, "twins": ["mutate_reorder_tamper"]},
     {"surface": "meridian-lane1-p3", "lane": 1, "twins": ["leak_amended_terms_at_V2"]},
     {"surface": "meridian-lane1-p4", "lane": 1, "twins": ["silent_zero_and_stale_carry_forward", "valuation_omitted_without_declaring_unevaluable"]},
     {"surface": "meridian-lane1-p5", "lane": 1, "twins": ["cost_basis_drift"]},
     {"surface": "meridian-lane1-p6", "lane": 1, "twins": ["fill_qty_plus_one", "invented_untraded_position", "price_plus_one"]},
     {"surface": "meridian-lane1-p7", "lane": 1, "twins": ["hash_field_mislabeled", "wrong_feed_served_as_base"]}
   ]}
   ```
   Every cell is listed, each with every twin mutation its rows carry
   (order irrelevant): a cell left out is refused as unexpected, a twin left
   out as a differing set. The file is strict: an unknown key, a `status`,
   or the same key twice in one object makes it unevaluable (exit 2). It
   never asserts a status — CLAIMABLE vs. PARTIAL stays derived.
7. Run the governed repo's own second derivation of status alongside the
   pack's `--json` output, and fail CI when they disagree — status is
   derived, never authored, and a repo that only trusts its own derivation
   has vendored a file set, not a discipline.

`check.mjs` takes exactly one positional argument; an extra one, an
unrecognised flag, or a rows-directory entry that is not a regular file (a
subdirectory, a symlink, a junction) is refused, exit 2, never silently
skipped. A row file carrying the same key twice in one object, at any
depth, is unevaluable too (exit 2, `duplicate key "<key>"`): a JSON parser
would keep one value and drop the other without a word. `--write-pin <sha>`
is a form of its own and takes nothing else.

### Checking a pinned commit's CI order

Before pinning a commit, a governed repo can confirm from that commit's own
workflow that its CI built the vendored shape and pinned it before running
the self-test there. Match the exact vendored-shape step, as consecutive
lines, not the order in which words first appear in the file: a word-order
check passes a workflow whose `--write-pin` and `--verify-pin` sit in a
repo-layout step or in a comment while the vendored copy runs its self-test
with no `PIN`. From Git Bash, with a clone of this repo:

```sh
cat > step.txt <<'EOF'
d="$(mktemp -d)/gates/conformance"
mkdir -p "$d"
cp -r conformance/. "$d"
cp schema/gate-verdict.v1.json "$d/"
node "$d/check.mjs" --write-pin "$GITHUB_SHA"
node "$d/check.mjs" --verify-pin
node "$d/test.mjs" --mutate
EOF
git -C <clone> show <sha>:.github/workflows/conformance.yml \
  | sed -e 's/^[[:space:]]*//' -e 's/[[:space:]]*$//' \
  | awk 'NR == FNR { want[++n] = $0; next } { got[++m] = $0 }
         END { for (i = 1; i + n - 1 <= m; i++) { ok = 1
                 for (j = 1; j <= n; j++) if (got[i + j - 1] != want[j]) { ok = 0; break }
                 if (ok) { print "ok the vendored-shape step is present, in order"; exit 0 } }
               print "FAIL the vendored-shape step is not present as consecutive lines"; exit 1 }' step.txt -
```

Leading and trailing whitespace is ignored; nothing else is, so a blank or
comment line inside the step, a reordering, or a missing line fails it. The
text shows the step exists at that commit; that it passed is shown only by
that commit's CI run (`gh run list --commit <sha> --workflow
conformance.yml`), and a green run is not evidence of the order on its own.
If this repo's workflow step changes, this block changes with it in the
same commit.

## Known limits

What the pack does not check, and what its controls are known to miss.

- **Legs folded into one count.** A `checks` key is a single integer. If an
  emitter's own check logic sums several distinct conditions ("legs") into
  one key before reporting it, the pack sees only the total — it cannot
  tell a count of 3 from one leg firing three times apart from three legs
  firing once each. An emitter that folds legs together loses that
  distinction to every downstream reader, the pack included.
- **A comparator exercised only by unit tests.** The pack conforms the
  *shape* of a row and the live/twin/crediting relationship between rows;
  it has no way to know whether the comparator that produced a check's
  count is itself correct. A comparator validated only by the governed
  repo's own unit tests, and never exercised by a twin mutation bound into
  the pack's real-row fixtures, is invisible to the pack — it can only
  credit what a twin has been seen to falsify (per-check crediting refuses
  exactly this blind spot for the twins the pack does see; it cannot reach
  past the rows to the comparator's own test suite).
- **Anything outside the rows.** The pack's entire universe is the rows
  handed to one invocation of `check.mjs`. It cannot detect rows that were
  regenerated, cherry-picked, or excluded before being pointed at it; it
  never reads the emitter's source, the data the emitter ran over, or
  anything about the pipeline that produced the rows. Conformance of the
  rows is not conformance of the system that emitted them.
- **A rows directory that is itself a link.** `check.mjs <dir>` follows a
  junction or symlink given as the rows directory and checks what it points
  at; only the entries inside must be regular files.
- **A row file that is not valid UTF-8.** `check.mjs` reads it lossily: a
  `0xFF` byte inside `unevaluable_reason` conforms, exit 0. Only the
  vendored set is held to strict UTF-8.
- **A group derived UNEVALUABLE exits 0.** Crediting never changes the exit
  code, so a conforming set whose group derives UNEVALUABLE exits 0.
- **A directory link under the pack.** A junction or directory symlink under
  `conformance/` crashes `test.mjs` with `EISDIR`, exit 1, before any named
  failure is printed; in a vendored copy `--write-pin` exits 2 with a bare
  `EISDIR` naming no entry.
- **Wording spellings the scan misses.** A rule number as `rule (7)`,
  `rule [7]`, `rule number 7` or `the 7th rule`, with an en dash or a
  non-breaking hyphen as separator, or with a soft hyphen inside the word; an
  id as `ruling 3`, `R#3`, or R and digits joined by an en dash or a no-break
  space; a home path as `$USERPROFILE/...`, `${env:USERPROFILE}\...`,
  `%HOME%`, `~<user>/...`, `/root/...` or `\\wsl.localhost\...`; `&#167`
  with no semicolon; a soft hyphen inside the name or a sha; a sha split
  across two lines within its first seven characters, or anywhere inside a
  hex run of 64 characters or more.
- **A single roman letter after the rule word.** `Rule V` is not caught: two
  or more upper-case numerals are required, so that `the rules I wrote` is
  not refused.
- **A sha abbreviated below seven characters.** Not caught: six hex
  characters match ordinary words and numbers.
- **A section cited in words, or a lower-case id.** `section 4` and `per r3`
  are not caught: a section is caught only as the section sign, its entities
  or its escapes, and an id only in upper case.
- **The commit-sha sub-check in a vendored copy.** It prints a note instead
  of checking: the pack's history is not there, and the `PIN`'s own sha is
  not used as a one-commit history.
- **A copy with no repository around it.** A copy with no `.git` and no
  `../schema/`, and with the schema beside `check.mjs`, is a vendored copy by
  every fact left: with HEAD's sha planted, the self-test there prints the
  note instead of refusing the sha.
- **A checkout whose git index was deleted.** With
  `schema/gate-verdict.v1.json` moved beside `check.mjs` and `.git/index`
  deleted, git no longer reports the schema as tracked, so the pack's own
  checkout reads as a vendored copy and a planted sha prints the note. A
  `.git` planted inside `conformance/` does not do this: git is asked from
  the parent directory, and a `.git` entry in the pack directory is refused
  by name.
- **The git-index condition is not pinned.** Replacing it with `true` keeps
  the self-test green; without it the layout decision only gets stricter.
- **A source tree without `schema/gate-verdict.v1.json`.** The layout-plant
  control copies that file, so in such a tree it throws `ENOENT` and the run
  exits 1 before printing its named failures.
- **`--write-pin` in this repo's layout.** `check.mjs --write-pin` writes a
  `PIN` here too; only the self-test refuses it.
- **The CI-order check proves text, not a run.** The check under "Checking a
  pinned commit's CI order" passes a workflow whose vendored-shape step has
  `if: false` or `continue-on-error: true`, runs under `set +e`, or has its
  `- name:` line commented out.
- **Linux with Node 24, locally.** The local runs recorded in `STATUS.md` are
  Windows with Node 24 and Linux with Node 20; only the CI matrix runs Linux
  with Node 24.
- **`STATUS.md` is a dated log.** An entry whose claim a later entry found
  false is corrected in the later entry, not rewritten.

## Where things are

`DATUM.md` the rules, the row's invariants, and what changed since v1 · `schema/` the row ·
`conformance/` checker, controls, fixtures · `docs/` design and research
notes · dated state lives in `STATUS.md`, not here.
