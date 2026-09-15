# DATUM conformance pack — design

2026-09-06. Status: **PROPOSED, nothing built** when written; **built
2026-09-09**, with the deviations recorded in §10. Governs how `DATUM.md`
binds the three repos it governs (baseline, traverse, meridian). Approach A
of three considered (a `datum` repo with an executable pack; vendored text
only; BASELINE's checker as reference implementation) — chosen by the
operator on 2026-09-06. Rulings folded in: `gate_sha`, generated block in
STATUS.md, Node as the pack runtime, JSON-only fixtures with in-case
outcomes. Precedent research and its limits:
`2026-09-06-conformance-suite-precedents.md`; where this design borrows,
it says from whom, and where it adds, it says so.

## 1. What the pack is

A governing text is a set of claims about how repos behave. Rule 4 of
DATUM says a checker without negative controls has proven nothing; applied
to DATUM itself, a governing text without an executable pack has governed
nothing. The pack is the part of DATUM that can refuse.

```
datum/
  DATUM.md                              governing text, v2, with its changes-from-v1 section
  schema/gate-verdict.v1.json           the row, as JSON Schema (specification)
  conformance/check.mjs                 reference checker: rows dir in; exit 0, or FAIL lines
  conformance/test.mjs                  the checker's own controls (rule 4 applied to DATUM)
  conformance/fixtures/positive/*.json  conforming rows; outcome inside each case
  conformance/fixtures/negative/*.json  one defect each; expected FAIL reason inside each case
  conformance/fixtures/real/<repo>/     one real row per governed repo, pinned by hash
  STATUS.md  README.md  docs/
```

No dependencies. Node 20 or later. `check.mjs` is the enforcement;
`schema/` is the specification it implements; `test.mjs` proves the two
agree on every fixture. Borrowed shape: every corpus suite surveyed keeps
the expected outcome inside each case (Test262, JSON Schema Test Suite,
toml-test, CommonMark, WPT). Added: the expected failure reason, which none
of those carry and which compiler test frameworks do (WebAssembly's
`assert_malformed` prefix match; rustc's exhaustive `//~ ERROR`; GCC's
`dg-error` plus excess-errors failure).

## 2. The row, schema v1

`schema/gate-verdict.v1.json`. Required unless marked optional.

| field | type | rule |
|---|---|---|
| `schema` | string | exactly `datum/gate-verdict/1` |
| `kind` | string | exactly `GATE_VERDICT` |
| `surface` | string | `^[a-z0-9][a-z0-9-]*$`; meaning per repo |
| `lane` | integer ≥ 1 | meaning per repo, defined in its design |
| `cell` | string | `live` or `twin` |
| `result` | string | `GREEN`, `RED`, `UNEVALUABLE` — exact |
| `unevaluable_reason` | string, non-empty | required iff `result` is `UNEVALUABLE`; forbidden otherwise |
| `checks` | object | check name → integer ≥ 0; at least one key |
| `evaluated` | object | same key set as `checks`; integer ≥ 0 each |
| `planted` | object | required iff `cell` is `twin`; forbidden on `live`. `{mutation: string, mutated_rows: integer ≥ 0, expected_violations: object}` |
| `scope` | string | what subset, in words |
| `params` | object | the inputs that select the run |
| `content_hash` | string | `^sha256:[0-9a-f]{64}$` |
| `content_hash_basis` | string | what bytes, what order, what library |
| `gate_sha` | string | `^[0-9a-f]{40}$` — commit of the emitter |
| `gate_worktree` | string | `clean` or `dirty` |
| `ran_at` | string | ISO-8601 UTC, `Z` suffix |
| `runner` | string, non-empty | `local`, `ci`, or an instance id |
| `rows` | integer ≥ 0, optional | a repo may require it and bind it to a named check |
| `metrics` | object, optional | repo-specific measurements; never displaces a field above |

Additional properties are refused. A row that fails any rule is refused
with a named reason (§4).

Cross-field rules the schema cannot express and `check.mjs` enforces:

- `evaluated[k] === 0` for any `k` forces `result: UNEVALUABLE`.
- `cell: live` with any non-zero `checks` value must be `RED`, never
  `GREEN`; `cell: twin` with all-zero `checks` must not be `RED`.
- A `twin` with `result: RED` must have `checks` deep-equal to
  `planted.expected_violations` over the union of both key sets.

Changes from what BASELINE and MERIDIAN emit today, each fixing a drift
measured on 2026-09-06: `schema` added; `gate_sha`/`gate_worktree` replace
the literal `parallax_sha`/`parallax_worktree` both emitters write;
`unevaluable_reason` replaces the `UNEVALUABLE:<reason>` form; `rows`
optional; `lane` unbounded above; `runner` unbounded.

## 3. Crediting across rows

The pack checks single rows and, given a directory, the set:

- at most one `live` row per (`surface`, `lane`);
- `twin` rows unique per (`surface`, `lane`, `planted.mutation`);
- a (`surface`, `lane`) is **CLAIMABLE** when its live row is `GREEN` and
  every twin is `RED` as planted; **PARTIAL** when exactly one cell kind is
  present; **UNCLAIMED** when none; **UNEVALUABLE** when any present row
  is. Live `RED` or twin `GREEN` is refused, not derived — a real surface
  failing or a plant the gate missed needs a human.
- the literals `CLAIMABLE`, `PARTIAL`, `UNCLAIMED` may not appear as any
  string value in any row (rule 5).

This is MERIDIAN's rule (`gates/claimability.py`), generalized. BASELINE's
single-twin rule is the special case with one twin.

## 4. Fixtures and the checker's own controls

Every fixture is one JSON file whose top level is the case, not the row:

```json
{
  "case": "twin red for the wrong reason",
  "expect": "FAIL",
  "expect_reason": "twin RED does not match the plant",
  "row": { ... }
}
```

`expect` is `PASS` or `FAIL`. `expect_reason` is required on `FAIL` cases
and forbidden on `PASS` cases. `check.mjs` emits one line per refusal,
`FAIL <file>: <reason>`, from a fixed reason vocabulary listed in
`conformance/reasons.md` and mirrored as constants in `check.mjs`.

`test.mjs` runs `check.mjs` over every fixture and holds three things:

1. every `PASS` fixture is accepted;
2. every `FAIL` fixture is refused, and the emitted reason **starts with**
   `expect_reason` (prefix match, WebAssembly's rule);
3. no refusal is emitted that no fixture expected (rustc and GCC's rule:
   an unexpected diagnostic is itself a failure).

A negative fixture that the checker rejects for a different reason fails
`test.mjs`. That is the twin rule applied to the pack: rejection is not
enough; rejection for exactly the planted reason is.

`fixtures/real/<repo>/` holds one live and one twin row copied from each
governed repo at adoption, bound by LF-normalized sha256 in
`fixtures/real/SOURCE.md`. They are `PASS` fixtures. When a repo's emitter
changes, the real fixture is re-copied at the new commit; it is never
edited.

## 5. How a repo is governed

A governed repo carries:

```
gates/conformance/check.mjs
gates/conformance/test.mjs
gates/conformance/fixtures/**
gates/conformance/reasons.md
gates/conformance/gate-verdict.v1.json
gates/conformance/PIN      datum commit sha + "sha256  path" per vendored file, LF-normalized
```

and a CI step that runs, in order and failing on any non-zero exit:

```
node gates/conformance/check.mjs --verify-pin                              # the vendored bytes are the pinned ones
node gates/conformance/test.mjs --mutate                                   # the pack's own controls
node gates/conformance/check.mjs <rows-dir> --verify-pin --expect <file>   # the repo's emitted rows
```

(Path, file set and order as amended on 2026-09-14; see the §10 addendum
"vendoring path and pack fixes" of that date. As first written, this
section named `vendor/datum/` and two commands.)

`<rows-dir>` is the repo's own: `ledger/verdicts/` for baseline,
`gates/out/` after `sh gates/run.sh` for meridian, the committed rows
directory for traverse. A pin-verification step recomputes each vendored
file's sha256 against `PIN` so a locally edited pack cannot pass as the
pinned one. Pinning by content hash is DATUM's addition: no surveyed suite
does it (JSON Schema consumers pin by commit or track `main`; Gecko vendors
WPT via a sync bot).

The repo's README carries a **DATUM** section: the pinned commit, the two
commands, and a rule-by-rule table. For rules 1, 3, 5 and 6 the table's
cell is the pack's result on the last CI run. For the eight prose rules it
is authored (`carried`, `partial`, `missing`) and labelled authored. A
generated per-rule report replacing the authored cells is a **proposal**,
not a precedent: the W3C implementation-report practice it was modelled on
did not survive to a cited claim in the research note.

Claim rule, borrowed from the Jakarta EE TCK process (hashed suite, public
run count): a repo may state "governed by DATUM at `<sha>`" only while its
CI runs the pinned pack green on a pushed commit.

## 6. Rollout

1. **datum** commit zero: `DATUM.md` v2, this design, the
   research note, README, STATUS. Then the pack: schema, `check.mjs`,
   `test.mjs`, synthetic fixtures. CI runs `test.mjs`. `fixtures/real/`
   stays empty until step 2 produces conforming rows.
2. **meridian**: `verdict.go` writes `schema`, `gate_sha`, `gate_worktree`;
   `claimability.py` reads the new keys; vendor the pack; CI step; README
   section; dated amendment to design §7 and a STATUS entry (DATUM's
   "not a catalog" scope section). Copy one live and one twin row into `fixtures/real/meridian/`.
3. **baseline**: build `superseded_by` first, since re-emitting the two
   rows otherwise 404s their published URLs (learning
   `2026-08-31-ledger-has-no-supersession`). Then generalize
   `scripts/lib/ledger.mjs` (`gate_sha`, `unevaluable_reason`, `rows`
   bound to `no_future_accepted` stays as BASELINE's own rule), rename in
   PARALLAX's `verdict_row()`, re-emit both rows against the gold surface
   (operator step: needs the surface), re-bind `SOURCE.md`, vendor the
   pack, CI step, README section. Copy the two rows into
   `fixtures/real/baseline/`.
4. **traverse**: born conforming. Amend its design for rule 3's mechanics
   (`cell`, `planted`, GREEN/RED, `unevaluable_reason`) before any code.

Order within a repo: pack vendored and `test.mjs` green in CI **before**
the emitter changes, so the first `check.mjs` run over real rows is a
measured red, not an assumed one.

## 7. Error handling

`check.mjs` exit codes: `0` all rows conform; `1` one or more refusals,
each on its own `FAIL` line; `2` unevaluable — rows directory missing or
empty, a file that is not JSON, or a `PIN` mismatch when run with
`--verify-pin`. Exit 2 is never coerced into 0 (DATUM rule 2). `test.mjs`
exits `1` on any of its three conditions and prints which fixture and
which condition.

## 8. Testing the pack itself

- `test.mjs` is the gate; CI runs it on every push to datum.
- A mutation self-test: `test.mjs --mutate` flips one rule in an in-memory
  copy of `check.mjs`'s rule table and asserts that at least one negative
  fixture stops being refused. A pack whose fixtures cannot detect a
  disabled rule has an unfalsified rule (MERIDIAN's invariant: a check
  whose every twin plants 0 is unfalsified).
- The schema file is validated against the fixtures with a minimal
  in-repo JSON Schema subset validator (no external validator; the subset
  used is `type`, `enum`, `pattern`, `required`, `properties`,
  `additionalProperties`, `minimum`, `minProperties`). If a fixture needs
  a keyword outside that subset, the design changes, not the validator.

## 9. Not built, and not decided

- The generated per-rule implementation report (§5) — proposal.
- Signing of the pack or of rows.
- A `superseded_by` field in the shared row. BASELINE needs supersession
  for its published URLs; whether it is a row field or a ledger-level
  pointer is BASELINE's decision and is not in schema v1.
- Whether `fixtures/real/` rows are updated by a bot or by hand. By hand
  until there is a third governed repo.
- Floor provenance for TRAVERSE (DATUM rule 10).

## 10. Built 2026-09-09, and what moved from §1–§8

Status of the sections above: **built** as written except where this
section says otherwise. Built: `schema/gate-verdict.v1.json`;
`conformance/check.mjs` (subset validator, nine row rules, four set rules,
crediting, `--json`, `--verify-pin`, `--write-pin`, exit codes 0/1/2);
`conformance/test.mjs` (the three conditions, vocabulary coverage,
real-row hash binding, CLI exit-code and PIN exercises, `--mutate`,
`--write-status` / `--check-status`); `conformance/reasons.md`; 13
positive and 46 negative fixtures; `fixtures/real/SOURCE.md` (empty
binding); CI on Node 20 and 24. Measured at build: `test.mjs --mutate`
green, 17 reasons, 13 rules each detected disabled by at least one
fixture; over MERIDIAN's 18 rows at `a087ab2`, `check.mjs` exits 1 with
five refusals per row, all in the two planned families (three missing
required fields `schema`, `gate_sha`, `gate_worktree`; two unknown fields
`parallax_sha`, `parallax_worktree`), and after the planned rename plus
`schema` key in a scratch copy it exits 0 with all seven surfaces
CLAIMABLE and twin counts 1/1/1/2/1/3/2, matching MERIDIAN's own
`claimability.py`.

Deviations from the sections above, each made at build and each
checkable in the code:

- **§2**: `schema` is an enum, not a pattern, so an unknown version refuses
  with `must be one of` like the other exact literals. A negative count
  in `checks` is a schema refusal only; the live-with-violations rule
  counts positives, so a fixture plants one defect, not two.
- **§3**: `UNCLAIMED` is never derived from a directory (no rows, no
  group). It is a status only a repo's own STATUS.md can carry for a lane
  it has not run. The status-literal rule reads object keys as well as
  string values: an adversarial pass on the first green build smuggled
  `CLAIMABLE` in as a `params` key and was not refused; it is now.
- **§7**: every regular file in the rows directory is a row. The first
  build filtered on a lowercase `.json` extension and silently skipped the
  rest; the same adversarial pass hid a live-RED row as `BAD.JSON` and got
  exit 0. Now a file that does not parse as JSON makes the directory
  unevaluable (exit 2), and nothing is skipped by name.
- **§4**: a fixture may carry `rows` (an array) instead of `row` for
  set-level cases; the case wrapper is otherwise as designed. `reasons.md`
  also names the rule-table entry behind each reason, since that is the
  unit `--mutate` disables.
- **§5**: the vendored set gains a copy of `schema/gate-verdict.v1.json`
  beside `check.mjs` (the checker loads it from there, falling back to
  `../schema/` inside this repo), and `check.mjs --write-pin <datum-sha>`
  writes `PIN` over every file in the vendored directory. `--verify-pin`
  refuses a listed file that is missing or altered and an unlisted file
  that is present.
- **§8**: the mutation self-test does not edit a copy of `check.mjs`; the
  rule table is exported and `checkRows` takes a set of disabled rules.
  Same property, no file rewriting. A second condition was added: with
  every rule disabled, every negative fixture must be accepted, so no
  refusal can live outside the table.
- **DATUM's own STATUS.md**: the reserved generated block is rendered by
  `test.mjs --write-status` (fixture and rule counts from a green run) and
  compared in CI by `--check-status` (rule 5 applied to this repo).

Found at the first adoption (MERIDIAN, 2026-09-09), after the pack was
pushed:

- **§5, location**: a Go module cannot hold the pack under `vendor/` — the
  toolchain treats that directory as module vendoring and refuses to build
  without `vendor/modules.txt` ("inconsistent vendoring", `go vet ./...`
  exit 1). A governed repo places the pack where its toolchain allows;
  MERIDIAN used `gates/datum/`. The contract is the file set, the `PIN`,
  and the CI order, not the path. (Superseded on 2026-09-14: every
  governed repo now uses `gates/conformance/`; see that date's addendum
  "vendoring path and pack fixes".)
- **§8, the self-test when vendored**: `test.mjs` copied the schema from
  `../schema/`, which exists only inside this repo, so the vendored
  self-test crashed on its PIN exercise. It now takes the schema from
  beside itself first. A pack whose self-test only runs at home was never
  a vendored pack; this one line is the difference.

### Addendum, 2026-09-14 (coalescing round, D1-D3)

- **D1, `--write-pin` writes the file itself**: previously the CLI printed
  the pin body to stdout and left the caller to redirect it into `PIN`
  (`test.mjs`'s own PIN exercise did exactly that). A failed run could
  therefore leave an empty `PIN` behind via `> PIN` truncating before node
  ever wrote anything. `--write-pin` now calls the new `writePinAtomic`,
  which writes a temp file beside `check.mjs` and renames it onto `PIN`,
  and prints only `ok pin written`. Content-building is unchanged (same
  walk, same LF-normalized sha256 over every non-`PIN` file); the only new
  thing is a shape guard, `checkPinShape(lines, fileCount)`, asserting line
  1 is `datum <sha>` and the line count is the file count plus one. Given
  how the lines are built this guard cannot actually fire from real input,
  so it is exercised in `test.mjs` by calling it directly with crafted
  lines/fileCount rather than by weakening the guard itself — the same
  test-the-guard-not-the-code-path discipline as the mutation self-test in
  §8. Both new refusal strings are documented in `reasons.md` as prose
  under "Not refusals" rather than as vocabulary bullets, since they are
  refusals about `--write-pin`'s own construction, not verdicts on a row,
  and adding them as bullets would have required a row-level fixture that
  doesn't exist for them.
- **D2, first real rows**: `fixtures/real/` held no rows since the pack was
  built; it now holds MERIDIAN's first live row and one of its two twins
  (`wrong_feed_served_as_base`), copied byte-for-byte from `gates/out/` at
  meridian `97d815c9d95b9bac7a85f69b1cc66f6e5310f45e` after confirming
  `gate_worktree: "clean"` and a matching `gate_sha` on all three rows
  present there. The second twin present at that commit
  (`hash_field_mislabeled`) is not bound; the real-row layout calls for one
  live and one twin per governed repo, not every twin a surface happens to
  have run. Flipping one byte of the bound live row in a scratch copy made
  `test.mjs` fail naming that exact file with a sha256 mismatch against
  `SOURCE.md`, confirming the hash binding is load-bearing and not
  decorative.
- **D3, CI catches a non-vendored-shape regression**: the self-test step
  in CI ran `conformance/test.mjs` in place, which is not the shape a
  governed repo actually runs (found the hard way at MERIDIAN's first
  adoption, above). CI now also assembles a fresh temp copy
  (`conformance/.` plus the schema file) and runs `test.mjs --mutate`
  there, so a regression of that class is caught here rather than at the
  next adopter.

### Addendum, 2026-09-14 (pack CLI hardening, P1-P5)

- **P1, the CLI's own argument and entry shape**: `check.mjs` took the
  first non-flag argument as the rows directory and silently ignored an
  extra positional or an unrecognised flag; a rows-directory entry that was
  not a regular file (a subdirectory, a symlink, a Windows junction) was
  filtered out of the file list rather than refused, so a directory holding
  only such entries read as "holds no rows" instead of naming what was
  actually there. Both are now usage-shaped exit-2 refusals: more than one
  positional or a flag outside `--json`/`--verify-pin`/`--write-pin` prints
  a usage line to stderr; a non-regular-file entry prints `unevaluable:
  <name> is not a regular file`, naming it, before the file list is even
  built. Measured before the fix, in a scratch copy: `check.mjs rows
  rows` and `check.mjs rows --verify-pn` both exited 0 as if the extra
  argument were not there; a rows directory with one valid row plus one
  subdirectory exited 0 as if the subdirectory were absent.
- **P2, `--verify-pin` alone**: previously `--verify-pin` was only ever
  combined with a rows directory, so a CI step that wanted to confirm the
  pin before running anything else had no standalone form to call; giving
  `--verify-pin` with no positional fell through to the usual "no rows
  directory" usage error, exit 2, even when the vendored tree was clean.
  `--verify-pin` given alone is now a complete invocation in its own right:
  `ok pin verified`, exit 0, on a clean tree; the existing problem list to
  stderr, exit 2, otherwise. The adoption order this enables (verify pin,
  then `test.mjs`, then `check.mjs <rows> --verify-pin`) is P6's, not
  built here; this addendum only supplies the CLI form it depends on.
- **P3, `--write-pin`'s own write is not yet atomic against real failure**:
  `writePinAtomic` built and validated the PIN content, then called
  `writeFileSync`/`renameSync` with no `try`/`catch` around either. The
  measured Windows defect: `node check.mjs --write-pin <sha> > PIN` makes
  the shell's own redirect target the rename destination, so the rename
  throws `EPERM`, uncaught -- Node's default uncaught-exception exit code
  (1, not 2), a stack trace on stderr, a temp file left beside `check.mjs`,
  and (from the shell's own truncating `>`, independent of anything
  `check.mjs` does) an empty `PIN`. Reproduced for real in a vendored-shape
  temp copy before the fix; the same command after the fix exits 2 with
  `--write-pin failed: EPERM` and leaves no temp file. The write and the
  rename now run inside a `try`/`catch` that removes this run's own temp
  file on any failure and reports `--write-pin failed: <code>`; a
  `.PIN.tmp.*` already present beside `check.mjs` is treated as a prior
  run's leftover and refuses the new run outright, before anything is
  written, rather than leaving two temp files to race. `writePinAtomic`
  takes the fs calls it uses (`writeFileSync`, `renameSync`, `unlinkSync`,
  `readdirSync`) as an injectable second argument, defaulted to the real
  ones, so the rename-failure path -- unreachable via a real POSIX rename
  onto an open file, which ordinarily succeeds -- can be forced and
  asserted on any CI platform, not only where the Windows fault reproduces.
- **P4, an entry point reached through a link**: the guard that decides
  whether `check.mjs` was invoked directly (`main()` runs) or only
  `import`ed (it does not) compared `import.meta.url` against
  `process.argv[1]` as literal strings. ESM module resolution realpaths
  `import.meta.url` by default, but `process.argv[1]` is left exactly as
  typed; reached through a symlink or a Windows junction the two strings
  differ even though they name the same file, so `main()` silently never
  ran -- measured before the fix: exit 0, no output, invoking `check.mjs`
  through a junction over a rows directory that should have been refused.
  The guard now compares `realpathSync` of both sides (falling back to the
  literal comparison if either path cannot be resolved, rather than
  throwing), so a vendored pack reached through a link a governed repo's
  own tooling creates still runs.
- **P5, the real-fixture layout is asserted, not just documented**: §4
  states the layout as one live and one twin row per governed repo, but
  nothing checked it; a `fixtures/real/<repo>/` directory short a cell, or
  carrying a second one, passed silently as long as every row present
  conformed. `test.mjs` now refuses a repo directory whose live and twin
  counts are not each exactly one, naming the repo and the counts found.
  Exercised by copying the pack into a vendored-shape scratch directory,
  removing MERIDIAN's twin row from the copy, and re-running the copy's own
  `test.mjs` against itself -- proof the control travels with the vendored
  pack rather than living only in this repo's own run.
- **Discipline note**: every behaviour above has a control in `test.mjs`
  that was run against the pre-change `check.mjs` first and captured
  failing (18 failures across P1-P4, including the `EPERM` stack trace
  itself in the captured output) before the corresponding fix made it
  pass. None of the new refusal strings extend `reasons.md`'s vocabulary
  bullets; they are prose under "Not refusals", the same convention D1
  used, since none is a verdict on a row. `RULE_NAMES` (13) and the
  reason count (17) are unchanged, so DATUM's own generated STATUS.md
  block did not need re-rendering. P6-P9 (`--expect` and its per-check
  crediting) are out of scope here and untouched.

### Addendum, 2026-09-14 (per-check crediting P8, expected set P7)

Operator rulings R3 (refuse credit for an unfalsified check) and R4 (add
`--expect` in the same pack move). §3 is amended as follows; its text above
stands as the record of what it said before.

- **§3, CLAIMABLE is per check**: §3 made a (`surface`, `lane`) CLAIMABLE
  when its live row is GREEN and every twin is RED as planted, which credits
  per gate: a check every twin plants 0 on was credited as though it had
  been seen to fail. §8 already named that state unfalsified for the pack's
  own rules; nothing applied it to a gate's checks. Now a group is
  CLAIMABLE only when, in addition, every check key the live row reports is
  set nonzero in the checks of at least one twin RED as planted; otherwise
  it derives PARTIAL, `--json` adds `"unfalsified": [<keys>]` (sorted) to
  its element, and the text line names the keys. A live row with no twin
  lists all of its checks. UNEVALUABLE still dominates and carries no list.
  A twin's nonzero check under a name the live row does not carry credits
  nothing (fixture `62-set-partial-twin-checks-disjoint`, the shape of
  MERIDIAN's P2). This is a crediting rule, not a refusal: the exit code of
  a conforming set is 0 either way, and the `--json` array stays flat.
- **§8, how the crediting rule is mutated**: the refusal rule table was the
  wrong home for it, since `--mutate` requires a negative fixture to stop
  being refused and a crediting rule refuses nothing. It lives in its own
  table (`CREDIT_RULES`, exported as `CREDIT_RULE_NAMES`, with `credit()`
  taking a disabled set like `checkRows()`). Its mutation unit is a new
  optional fixture key, `expect_credit` (the exact `--json` array a PASS
  case must derive, required on every set-level PASS case, forbidden on a
  FAIL case): `test.mjs --mutate` disables each crediting rule in turn and
  fails unless some PASS case's derived crediting stops matching its
  `expect_credit`. Breaking the rule's body in a scratch copy, rather than
  disabling it, also turns the plain self-test red on those fixtures.
- **§3, the expected set (`--expect <file>`)**: the file is
  `{"cells":[{"surface":"<s>","lane":<n>,"twins":["<mutation>",...]}]}`.
  Three new refusal rules join the rule table (so `--mutate` covers them
  like any other): `expect_unexpected_cell`, `expect_missing_cell`,
  `expect_twin_set`, reasons `unexpected cell`, `expected cell has no rows`,
  `twin mutation set differs for`. A cell is present when any row names it,
  refused or not, so a refused row does not also read as missing. The file
  is strict: an unknown key at either level (a cell's `status` included,
  because the expected set never asserts one), a duplicate cell, a repeated
  mutation, a lane below 1, or a surface outside the schema's pattern is
  malformed, exit 2, as is an unreadable or non-JSON file. `--expect` with
  no value, twice, or without a rows directory is a usage error, exit 2.
- **§4, fixture shape**: a case may carry `expect_file` (the content of an
  `--expect` file, applied to its rows, refusals reported against
  `<fixture>#expect`) and `expect_credit` (above). The key is not `expect`,
  which already holds PASS/FAIL. Positive set fixtures 12 and 13 gain
  `expect_credit` deliberately; neither's status changes under R3 (12's two
  twins together cover both live checks and stay CLAIMABLE; 13's two
  live-only groups were already PARTIAL and now also list their checks as
  unfalsified). No existing fixture's expected outcome changed.
- **Measured**: over a scratch copy of MERIDIAN's 18 rows (`gates/out/` at
  meridian `97d815c`), the pack exits 0 and derives 2 of 7 CLAIMABLE (P5,
  P7); P1, P2, P3, P4 and P6 derive PARTIAL, with unfalsified checks
  matching the round-1 survey exactly. MERIDIAN's own `claimability.py`
  still says 7/7, so its CLAIMABLE-count agreement step fails at the next
  re-vendor until its second derivation adopts the same rule; that is the
  ruling's expected consequence, not a pack defect. The bound real pair in
  `fixtures/real/meridian/` (P7 live plus the `wrong_feed_served_as_base`
  twin) still conforms and derives PARTIAL, unfalsified
  `snapshot_rehash_matches_claimed`: that twin plants 0 on it, and the twin
  that sets it nonzero (`hash_field_mislabeled`) is not bound.
- **P9, vendored-set wording (ruling R1)**: added a control to `test.mjs`
  scanning the vendored file set (`conformance/` plus
  `schema/gate-verdict.v1.json`) for the private repo's own name. It may
  appear exactly once, in `check.mjs`'s header, spelled `<name>, a private
  governing text`; elsewhere it is refused unless it is the schema
  identifier literal (`<name>/gate-verdict/<n>`, the row's own `schema`
  field, unchanged by design) or the PIN format's own first line -- its
  construction, its regex source, or its backtick-quoted description alike
  (the PIN line stays a machine field). Run first against the unchanged
  tree: 20 failures, each naming a file:line. Cleaned in place: header and
  inline comments in `check.mjs` and `test.mjs`, a `reasons.md` bullet, a
  fixture `case` string, `fixtures/real/SOURCE.md` prose, two internal
  temp-directory prefixes and two internal vendored-copy folder names (both
  cosmetic, not part of any contract), the `STATUS.md` generated-block
  marker pair (renamed in lockstep in both `test.mjs` and `STATUS.md`, since
  it is a matched pair, not free text), and `schema/gate-verdict.v1.json`'s
  own `$comment` (outside this task's literal file-ownership list but named
  explicitly by the ruling; edited anyway, logged as a deviation -- the
  control cannot go green without it). No refusal reason's own text
  changed, so no fixture's `expect_reason` needed a matching edit. The
  control's own source builds the name from character codes rather than
  spelling it: a control that could only find the name by containing the
  name would reintroduce the very mention it exists to police the moment
  `test.mjs` is vendored into a public repo. Verified green in this repo's
  own layout and in the CI vendored-shape layout (schema flattened beside
  `check.mjs`), and verified to actually fire in the latter by injecting
  the name into a scratch copy's schema file and watching it get refused,
  not by construction (a check that only ever finds itself already clean
  proves nothing about a genuinely broken input).
- **P6, adoption docs**: `README.md` gained "Adopting the pack" (the order
  -- archive a pushed commit, `--write-pin` run without a shell redirect and
  why, `test.mjs --mutate` in the vendored copy, rows emitted into a
  cleared directory, `check.mjs <rows> --verify-pin --expect <file>`, the
  governed repo's own second derivation checked for agreement) and "What
  the pack does not check" (conditions an emitter folds into one `checks`
  count; a comparator exercised only by the governed repo's own unit tests
  and never by a twin the pack's real-row fixtures bind; anything outside
  the rows handed to one invocation). No change to
  `.github/workflows/conformance.yml`: its two existing steps both invoke
  `test.mjs` unconditionally, which now runs the wording control regardless
  of `--mutate`, so nothing needed a new step.

### Addendum, 2026-09-14 (vendoring path and pack fixes)

Ruling R7, and the adversarial pass over the round-2 pack, which refuted
the pack's "green end to end" claim. Every behaviour change below started
from a control run and captured failing -- one run against the unchanged
`check.mjs`, real-row walk and wording scan: 53 failures, exit 1 -- and
then passed. The crediting item is the exception, stated there.

- **§5, path (R7)**: every governed repo vendors the pack into
  `gates/conformance/`. §5 now names that path, the schema file beside
  `check.mjs`, and the three-command CI order (the pin verified alone, the
  self-test, the rows with `--verify-pin --expect`). The MERIDIAN finding
  above keeps `gates/datum/` as the record of that adoption, marked
  superseded.
- **The PIN in a vendored copy (the blocker)**: the wording control
  scanned the generated `PIN`, whose first line is the machine field ruling
  R1 keeps, and refused it, so `test.mjs --mutate` exited 1 in every
  vendored copy holding a `PIN` -- the shape every governed repo ships.
  Neither CI nor any builder had run the self-test with a `PIN` present.
  The control now leaves `PIN` out (nobody authors it; `--verify-pin`
  fixes its shape). `test.mjs` gained a control that builds
  `gates/conformance/` in a temp directory, writes a real `PIN` with
  `--write-pin`, runs `--verify-pin` alone and then `test.mjs --mutate`
  there, each required to exit 0. CI runs the same sequence as a step,
  pinned to the workflow's own commit (P10). A copy started by the
  self-test does not start further copies of itself (an environment flag
  marks it nested, and it prints that it is).
- **§8, `--write-pin` (P3)**: it took its sha from anywhere in argv and
  ignored the rest, writing a `PIN` with exit 0 beside a rows directory,
  an unknown flag or extra positionals. It is now a form of its own,
  exactly `--write-pin <sha>`; anything else is a usage error that writes
  nothing (eight argv shapes under control). The leftover-temp scan could
  throw on a directory-listing error; it is now a refusal carrying the
  code. When both the rename and the clean-up unlink fail, the temp file
  stays and refuses every later run, so the refusal now names it.
- **`> PIN`**: the round-2 comment "a shell redirect can no longer produce
  an empty PIN" was false: the shell truncates the file before node starts.
  Comments, `reasons.md` and the README now say so, and a control pins that
  `--verify-pin` refuses an empty `PIN`.
- **§4, real-row layout (P5)**: the walk counted only lower-case `.json`
  files, so an empty repo directory, a directory holding only a README,
  and an upper-case `LIVE.JSON` all passed. Every regular file in a repo
  directory now counts and must be a `.json` row (any case) bound in
  `SOURCE.md`; anything else at the top level, a nested directory, or a
  link is refused. Controlled by one planted-defects copy that also carries
  a bound upper-case twin, counted as a second twin.
- **Wording sub-checks (P9)**: independently of the name, the control now
  refuses a local path, a rule number, a ruling or contract id, a
  section-sign citation, and a commit sha of this repo's own history (a
  7-to-40 hex run that begins a commit `git rev-list --all` lists). The
  last runs only in this repo's own layout, where an unreadable or shallow
  history fails the control (CI now checks out with `fetch-depth: 0`); in
  a vendored copy it prints that it was not evaluable there. The prose
  spelling of the PIN's first line (the name followed by `<sha>`) is no
  longer an allowed form: prose describes that line in words. Cleaned: the
  local path in `fixtures/real/SOURCE.md`, a design-doc section citation
  and contract ids in `test.mjs` comments, control labels and temp
  directory names, and the prose PIN spelling in `check.mjs` and
  `reasons.md`. Probes run in process against a synthetic history, and
  against the real history with HEAD's abbreviated sha.
- **Deviation, the id sub-check**: refusing a ruling or contract id is not
  in P9's list. It was added because the public wording rule names ids and
  the vendored bytes carried them. It matches an upper-case R, P or D plus
  digits only when not preceded by a word character or a hyphen, so a
  fixture surface such as `Example-P1` is not refused; lower-case labels
  are not caught and were renamed by hand.
- **§3, crediting (P8)**: a group with any UNEVALUABLE row, the live row
  included, derives UNEVALUABLE and no crediting rule runs on it;
  per-check crediting applies only to a GREEN live with conforming twins.
  This already held, so no control could be captured failing against the
  live tree; `credit()` now skips the crediting rules for such a group
  instead of computing and discarding them (no output change). Two
  positive fixtures pin it (68: a live UNEVALUABLE beside a twin covering
  one check; 69: a live GREEN, one RED twin, one UNEVALUABLE twin), and
  two scratch mutants of `credit()` -- one ignoring a live UNEVALUABLE, one
  ignoring a twin UNEVALUABLE -- each turned exactly the matching fixture
  red.
- **§7, duplicate keys in an expect file**: refused, exit 2,
  `duplicate key "<key>"`, at any object level, keys compared after
  unescaping. `JSON.parse` keeps the last of two equal keys without a word;
  an expect file exists to refuse the silent loss of an authored cell, so
  accepting last-wins would reintroduce exactly what it is for. Row files
  keep `JSON.parse` behaviour for now: whether a duplicate key in a row is a
  refusal (a new reason) or unevaluable is a row-contract decision not
  taken here, and the README lists it under what the pack does not check.
- **Comment fix**: `check.mjs` said `--mutate` needs a negative fixture for
  every rule. The refusal tables are mutated through negative fixtures;
  the crediting table through positive fixtures' `expect_credit`.
- **P6, README**: the adoption steps use `gates/conformance/`, the expect
  example is meridian's full seven-cell file (p4 two twins, p6 three, p7
  two), checked to exit 0 over a scratch copy of its 18 rows (the round-2
  example, one p7 twin, is refused against those rows).

### Addendum, 2026-09-14 (P11 and the round-3 refutations)

The adversarial pass over the round-3 pack refuted three of its wording
claims: an authored `PIN` escaped the wording scan in both layouts, file
names were never scanned (and a `PIN` carries every file name), and the
sub-checks missed ordinary accidental spellings. It also found fixture
directories dropping files silently. Every behaviour change below started as
a control added to `test.mjs` and run against the unchanged checker and scan:
61 failures, exit 1 (this repo's layout). The duplicate-key row control was
then sharpened so the planted row replaces the conforming live row, and
re-run from a frozen copy against the unchanged checker: both cases exit 0,
the row silently accepted (58 failures in that copy, vendored layout).

- **§4, fixture layout (P11 a)**: `fixtures/positive/` and
  `fixtures/negative/` refuse a file whose name does not end in `.json` and a
  subdirectory, by name, as `fixtures/real/` already did; an upper-case
  `.JSON` is read, not skipped. Beyond P11: anything at the top of
  `fixtures/` other than the three directories is refused, a case file or a
  real row carrying the same key twice is refused, and `fixtures/real/`
  holding no governed repo directory is refused by name rather than through
  the count control's setup. The count control and the planted-defects copy
  now take whichever governed repo sorts first instead of naming meridian,
  and the count control no longer runs inside a nested copy.
- **The PIN (P11 b)**: in this repo's layout a file named `PIN` anywhere
  under `conformance/` is refused and scanned. In a vendored copy `PIN` is
  left out of the scan only after `verifyPin()` accepts it; otherwise it is
  refused and scanned. `verifyPin()` was tightened to make that exclusion
  sound: a line naming a file outside the pack (through `..`), a second
  spelling of a listed file, or a repeated line no longer verifies, so a
  verified `PIN` holds only its machine first line and the pack's own file
  names.
- **Wording (P11 c)**: file and directory names in the vendored set carry
  the same sub-checks as lines. Local paths now include MSYS and Windows user
  directories (slash or backslash, escaped or not), a drive letter, a dev
  directory segment with no trailing slash, and home or profile variables. A
  rule number may follow a hash, hyphen, colon, dot or underscore, or be two
  or more roman numerals. An id may carry one space, hyphen, dot or
  underscore. The section sign is caught as its named and numeric HTML
  entities and its escapes. The commit check reads every maximal hex run of
  7 to 63 characters, in any case, and flags one holding a commit's first
  seven characters, so a sha after an underscore or a letter, or between hex
  letters, is caught; a 64-character run is a sha256 digest and is not read.
  Beyond P11: control and invisible format characters are refused. Not
  caught, and stated in the control's comment: a single roman letter after
  the rule word, and an abbreviation under seven characters (ordinary words
  and numbers match six hex characters). Every removed line of the old
  scan was audited against its replacement: each new sub-check matches
  everything the old one did, except the rule word, whose case-insensitivity
  had been lost; a mixed-case probe failed first, and it is restored.
- **Prose (P11 d)**: the schema literal, the PIN template and its regex
  source are allowed only in `.mjs` and `.json` files. Wrong-version schema
  literals in negative controls and in `test.mjs` stay, as code or data.
- **Encoding (P11 e)**: a vendored file that is not valid UTF-8 is refused.
  UTF-16 without a byte-order mark decodes as UTF-8 with a NUL between
  letters, and is refused as a control character.
- **§7, duplicate keys in a row file (P11 f, ruling D2)**: a row file
  carrying the same key twice in one object, at any depth, exits 2, naming
  the file and the key, never a FAIL line. This closes the open question in
  the previous addendum; the README no longer lists it under what the pack
  does not check.
- **§5, CI order (P11 g)**: the README documents a check a governed repo can
  run against a pinned commit's workflow that matches the seven lines of the
  vendored-shape step as consecutive lines. It passed this tree's workflow
  and failed, each exit 1, on the workflow at `df306e6`, round 3's violating
  workflow (a vendored self-test with no `PIN` after a repo-layout
  `--write-pin`), a comment-only copy, a reordered copy and a copy with a
  blank line inside the step.
- **`--write-pin`**: a second `--write-pin` or an empty string in the sha's
  place is a usage error, not a sha that fails its shape.

### Addendum, 2026-09-15 (the layout decision)

The adversarial pass over the P11 build found one major. `test.mjs` decided
this repo's layout by a guess: the schema under `../schema/` and none beside
`check.mjs`. One stray copy of `gate-verdict.v1.json` beside `check.mjs`, in
a checkout with its full history, made the layout read as vendored. Two
things switched off with it: the refusal of a `PIN` in this layout, and the
commit-sha sub-check, which became a note. A `PIN` written by `--write-pin`
with HEAD's sha, plus HEAD's abbreviated sha appended to `reasons.md`, then
passed the full self-test. Reproduced before any change, in a copy of this
repo with its `.git`: `test.mjs --mutate` exited 0. The nested self-test
also exited 0 with the schema moved out of `schema/` rather than copied.

- **The decision**: this is the pack's own repo layout when
  `../schema/gate-verdict.v1.json` exists. With that file gone from the
  working tree, it still is when git reports a work tree whose top holds this
  directory, with `check.mjs` in it, and whose index tracks
  `schema/gate-verdict.v1.json`. A schema beside `check.mjs` is no longer
  consulted. The two readings are not symmetric: the vendored one is the
  lenient one, and planted files can now only move a copy toward the pack's
  own layout, where they cost a false alarm, never a pass.
- **The schema beside `check.mjs`**: in this layout it is refused by name.
  `check.mjs` reads a schema beside itself before `../schema/`, so a copy
  there would silently stand in for the real one.
- **The control**: `test.mjs` copies this repo, `.git` included, and plants
  three things: a schema beside `check.mjs`, HEAD's abbreviated sha in
  `reasons.md`, and a `PIN` written by `--write-pin` with HEAD's sha, which
  verifies. It does this twice, once as a stray copy and once with the schema
  moved out of `schema/`. The self-test re-run in each copy must exit 1, name
  all three, and print no not-evaluable note. In the stray-copy tree it also
  places a clean vendored copy under `gates/` with its own `PIN`, which must
  exit 0 and print the note, so a vendored copy inside some git work tree is
  not read as the pack's own repo. The control needs this repo's history, so
  it runs only where that history was read. A vendored copy's non-nested run
  prints `note: layout: ...` saying so.
- **Evidence**: with the control added and the old decision in place,
  `conformance: 10 failure(s)`, exit 1, five per variant. With the
  `../schema/` signal alone, `conformance: 5 failure(s)`, all in the
  moved-schema variant, which is why the git signal exists. With both, exit
  0 and `ok conformance: 19 positive, 50 negative, 2 real, 20 reasons, 16
  rules mutated, 1 crediting rule mutated`. A scratch mutant without the
  top-of-work-tree check failed the vendored-copy sub-control (`exit 1,
  expected 0`). By-hand replay on the fixed tree: the stray copy, non-nested
  `--mutate`, exited 1 naming the schema copy, the `PIN` and `reasons.md:158`;
  the moved schema, nested, exited 1 naming the same three. The CI
  vendored-shape step, run verbatim, exited 0. Windows, Node 24.13.0 only.
- **Not changed, and listed in the README's "Known limits"**: `check.mjs
  --write-pin` still writes a `PIN` in this repo's layout, and only the
  self-test refuses it there. A copy of the pack with no `.git` and no
  `../schema/`, with the schema beside `check.mjs`, is a vendored copy by
  every fact left: with HEAD's sha planted, its nested self-test exited 0
  with the note. The README section also lists the minor findings of the
  same pass and the items earlier builders declined, each re-probed.

### Addendum, 2026-09-15 (a `.git` planted in the pack directory)

The adversarial pass over the layout decision above found one more plant.
With the schema moved beside `check.mjs`, a `.git` file placed inside
`conformance/` made `git -C conformance` stop at the planted entry. Git never
saw the enclosing repository, so the layout read as vendored again: the
`PIN` was accepted, HEAD's sha printed the note, and `test.mjs --mutate`
exited 0.

- **The fix**: git is asked from the pack's parent directory, and a `.git`
  entry inside the pack directory is refused by name in either layout.
- **The control**: the layout plant runs a third variant, the moved schema
  with a `.git` planted in the pack, and requires exit 1 naming the `.git`,
  the schema copy, the `PIN` and the sha.
- **Evidence**: with the variant added and the old decision in place,
  `conformance: 5 failure(s)`, exit 1, all five in the new variant. After the
  fix, `test.mjs --mutate` exits 0 with `ok conformance: 19 positive, 50
  negative, 2 real, 20 reasons, 16 rules mutated, 1 crediting rule mutated`.
  By hand, the same plant's nested self-test exits 1 naming the `.git`, the
  schema copy, the `PIN` and `reasons.md:158`. Two scratch mutants, git asked
  from the pack directory again and the `.git` refusal removed, each make
  the new variant fail. Windows, Node 24.13.0 only.
- **Still a limit** (README "Known limits"): the claim above that planted
  files cost "a false alarm, never a pass" holds for planted files only.
  With the schema moved beside `check.mjs`, deleting `.git/index` makes the
  checkout read as vendored: measured, the nested self-test exits 0 with the
  note.
