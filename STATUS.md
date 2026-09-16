# STATUS

State of record for DATUM. The README defers to this file.

- **2026-09-06** — Repository created. `DATUM.md` moved in from
  `~/dev/briefs/` and revised to v2 in the same day, folding in the
  operator's rulings: DATUM governs baseline, traverse and meridian;
  BASELINE is not a catalog; row schema v1 (`schema`, `gate_sha`,
  `unevaluable_reason`, optional `rows`, multi-twin crediting); STATUS
  house rule; rules 7 and 8 reconciled. Design for the conformance pack
  written (`docs/2026-09-06-datum-design.md`). Precedent research note
  written from a verified run (`docs/2026-09-06-conformance-suite-precedents.md`).
  **Nothing executable exists**: no schema file, no checker, no fixtures,
  no CI. No governed repo has vendored anything. Every claim in the design
  is a proposal.

- **2026-09-09** — **Conformance pack built.** `schema/gate-verdict.v1.json`,
  `conformance/check.mjs`, `conformance/test.mjs`, `conformance/reasons.md`,
  13 positive and 46 negative fixtures, an empty real-row binding, CI on
  Node 20 and 24 (`.github/workflows/conformance.yml`). `node
  conformance/test.mjs --mutate` is green locally on Node 24: every PASS
  fixture accepted, every FAIL fixture refused for its stated reason, no
  refusal outside `reasons.md`, every reason expected by a fixture, and
  each of the 13 rules detected disabled by at least one fixture. An
  adversarial pass over the green build found two ways a bad row passed
  (a rows-directory filter that skipped files by extension, and a status
  literal smuggled in as an object key); both were fixed test-first the
  same day and are fixtures now. Measured
  against MERIDIAN's 18 rows at meridian `a087ab2`: exit 1 with five
  refusals per row, all in the two families the design planned for
  (missing `schema`/`gate_sha`/`gate_worktree`, unknown
  `parallax_sha`/`parallax_worktree`); after the planned rename plus
  `schema` key in a scratch copy, exit 0 with seven surfaces CLAIMABLE and
  twin counts matching MERIDIAN's `claimability.py`. Deviations from the
  design are in its §10. **Not yet:** CI has not run (nothing pushed); no
  governed repo has vendored the pack; `fixtures/real/` is empty. Nothing
  below the markers is typed by hand.
  Corrected later the same day, after `48dc48c` was pushed and CI ran
  green on Node 20 and 24: MERIDIAN's first vendoring found that the
  self-test read the schema from `../schema/`, a path that exists only
  here, so it crashed when vendored; fixed (`test.mjs` now finds the schema
  beside itself first), design §10. MERIDIAN also cannot use `vendor/`
  (Go reserves it); it vendors under `gates/datum/`. The fix is the only
  change to the pack since `48dc48c`; the self-test line below is unchanged.

- **2026-09-14** — Coalescing round: `--write-pin` now writes `PIN` itself.
  It takes an atomic temp-file-then-rename beside `check.mjs`, prints only
  `ok pin written` (never the pin body, so a shell redirect can no longer
  produce an empty `PIN`), and refuses (exit 2, prior `PIN` byte-unchanged,
  no temp file left) on a malformed sha or a content shape that fails its
  own line-count guard (new export `checkPinShape`, exercised test-first
  with crafted input rather than by weakening the guard). Controls added to
  `test.mjs` first; run against the pre-change `check.mjs` from a scratch
  copy they failed on a missing export (red), then passed once implemented
  (green). `fixtures/real/` is no longer empty: MERIDIAN's first live row
  and one of its two twins (`wrong_feed_served_as_base`), copied byte-for-byte
  from `gates/out/` at meridian `97d815c9d95b9bac7a85f69b1cc66f6e5310f45e`
  after confirming `gate_worktree: "clean"` and `gate_sha` on all three rows
  present there, hash-bound in `fixtures/real/SOURCE.md`. Flipping one byte
  of a copied row in a scratch copy made `test.mjs` fail naming that file,
  confirming the hash binding actually bites. CI
  (`.github/workflows/conformance.yml`) gained a step that runs the
  self-test from a freshly assembled vendored-shape temp copy (`cp -r
  conformance/.` plus the schema file into a `mktemp -d`, then `node
  test.mjs --mutate` there), so a pack that only passes at home is caught
  in CI, not just at the next adoption. Design §10 addendum records the
  same. Deviation: the two-twin set at the source commit means one twin
  (`hash_field_mislabeled`) is not bound; only one is required by the
  layout and the other was not needed to demonstrate the binding.

- **2026-09-14** — Pack CLI hardening (P1-P5), same coalescing round.
  `check.mjs` now accepts exactly one positional argument and rejects any
  flag outside `--json`/`--verify-pin`/`--write-pin` (exit 2, a usage line
  to stderr) instead of silently taking the first positional and ignoring
  the rest; every entry of a rows directory must be a regular file, so a
  subdirectory, a symlink, or a Windows junction is named and refused
  rather than skipped down to fewer rows (P1). `check.mjs --verify-pin`
  given alone, with no rows directory, is now a complete invocation --
  `ok pin verified` and exit 0 on a clean vendored tree, or exit 2 listing
  problems -- so CI can verify the pin before the vendored `test.mjs` runs
  at all, not only before the rows check after it (P2). `writePinAtomic`
  takes an injectable fs-like object and wraps the write-then-rename in a
  try/catch: the measured Windows defect (`node check.mjs --write-pin <sha>
  > PIN` throws `EPERM` on rename, uncaught, exit 1, a stray `.PIN.tmp.*`
  file, a truncated empty `PIN`) is now a clean refusal, exit 2, with the
  temp file removed; a `.PIN.tmp.*` already present beside `check.mjs` (a
  prior run's leftover) now refuses outright before writing anything,
  rather than racing it with a new temp file (P3). The entry-point gate now
  compares `realpathSync` of `import.meta.url` and `process.argv[1]`
  instead of the literal strings, so `check.mjs` reached through a symlink
  or a junction actually runs `main()` -- measured before the fix: exit 0,
  no output, through a Windows junction over a refusable rows directory
  (P4). `test.mjs` itself now refuses a `fixtures/real/<repo>/` directory
  that does not hold exactly one live and one twin row, short or over
  (P5). Every new behaviour has a control in `test.mjs`, each run against
  the pre-change `check.mjs` first and captured failing (18 failures,
  including a live, uncaught `EPERM` crash reproducing the Windows defect
  for real through a subprocess) before the fix made them pass. The new
  refusal strings are documented as prose under "Not refusals" in
  `reasons.md`, the same convention the prior round used for
  `--write-pin`'s own refusals, since none of them are verdicts on a row;
  the vocabulary count (17) is unchanged. `RULE_NAMES` and the row/set rule
  tables are unchanged, so the generated block below is unchanged. P6-P9
  (the `--expect` flag and its per-check crediting) are explicitly out of
  scope for this round and untouched.

- **2026-09-14** — Per-check crediting (P8, ruling R3) and the expected set
  (P7, ruling R4), same coalescing round. `credit()` now derives PARTIAL,
  never CLAIMABLE, for a (surface, lane) whose live row reports a check no
  twin RED as planted sets nonzero, and names those checks (`--json`:
  `"unfalsified": [...]` on the element; text: `; unfalsified: ...`); the
  exit code is unchanged by it. The rule lives in a separate crediting
  table (`CREDIT_RULE_NAMES`), and `--mutate` covers it through a new
  fixture key, `expect_credit`, which every set-level PASS fixture now
  carries. `check.mjs <rows> --expect <file>` refuses an unexpected cell,
  an expected cell with no rows, and a differing twin mutation set (three
  new rules in the refusal table, three new reasons); a malformed or
  unreadable expect file, or `--expect` misused, exits 2. Fixtures gain
  `expect_file`. New fixtures: positive 60-63, negative 64-67; positive 12
  and 13 gain `expect_credit` with no change of status. Controls were added
  to `test.mjs` first and run against the unchanged `check.mjs`: 37
  failures, exit 1; green after. Measured over a scratch copy of MERIDIAN's
  18 rows at meridian `97d815c`: 2 of 7 CLAIMABLE (P5, P7), the round-1
  prediction; P1-P4 and P6 PARTIAL. MERIDIAN's `claimability.py` still says
  7/7, so its agreement step will fail at its next re-vendor until its own
  derivation follows the ruling. The bound real P7 pair still conforms and
  derives PARTIAL (unfalsified `snapshot_rehash_matches_claimed`; the twin
  that plants it is not the bound one). **Not yet:** nothing committed;
  MERIDIAN not re-vendored; P6 and P9 are the next builder's.

- **2026-09-14** — Vendored-set wording (P9, ruling R1) and the adoption
  docs (P6), closing the coalescing round. A control added to `test.mjs`
  scans everything vendored — this directory plus
  `schema/gate-verdict.v1.json` — for the private repo's own name: it may
  appear exactly once, in `check.mjs`'s header, spelled `<name>, a private
  governing text`; elsewhere it is refused unless it is the schema
  identifier literal or the PIN format's own first line (both stay as
  machine fields). Run first against the unchanged tree: 20 failures,
  naming every offending line, including one the task's own file list had
  not named (`schema/gate-verdict.v1.json`'s `$comment`, edited anyway —
  logged as a deviation from the literal ownership list, since P9 named it
  explicitly and the control cannot go green without it). The comments,
  `reasons.md` prose, a fixture `case` string, `fixtures/real/SOURCE.md`
  prose, two internal temp-directory prefixes, two internal vendored-copy
  folder names, and the `STATUS.md` marker pair (`pack:status:*`, renamed
  from a form that itself spelled the name) were all reworded or renamed;
  none of the fixture `expect_reason` strings needed to change, since no
  refusal reason's own text was among the offending lines. The control's
  own source builds the name from character codes rather than spelling it,
  so that vendoring `test.mjs` into a public repo does not reintroduce the
  very mention it exists to police; verified green both here and from a
  scratch copy in the vendored-shape layout (schema flattened beside
  `check.mjs`), and verified to actually fire in that layout by injecting
  the name into a scratch copy's schema file and watching it get refused.
  `README.md` gained an "Adopting the pack" section: the order (archive a
  pushed commit, never a working tree; `--write-pin` run without a shell
  redirect, the measured Windows defect explained; `test.mjs --mutate` in
  the vendored copy; rows emitted into a directory cleared first;
  `check.mjs <rows> --verify-pin --expect <file>`; the governed repo's own
  second derivation checked for agreement) and what the pack does not
  check (several conditions an emitter folds into one `checks` count; a
  comparator exercised only by the governed repo's own unit tests and never
  by a twin the pack's real-row fixtures bind; anything outside the rows
  handed to one invocation). `.github/workflows/conformance.yml` needed no
  change: both its existing steps already run `test.mjs` unconditionally,
  which now includes the wording control regardless of `--mutate`.

- **2026-09-14** — Pack fixes after the round-2 refutation (ruling R7 and
  P3, P5, P6, P8, P9, P10), same coalescing round. **The round-2 claim
  "green end to end" was false:** the wording control scanned the
  generated `PIN` and refused its first line, so `test.mjs --mutate` exited
  1 in every vendored copy holding a `PIN` — the shape governed repos ship —
  while CI and every builder ran the vendored self-test without one. Also
  refuted and now fixed: `--write-pin` took its sha from anywhere in argv
  and ignored everything else (writing a `PIN`, exit 0, beside a rows
  directory or an unknown flag); the real-row walk counted only lower-case
  `.json` files, so an empty repo directory, a README-only directory and an
  upper-case `LIVE.JSON` passed; the wording control could not see a local
  path, a rule number or a commit sha without the name on the line;
  `fixtures/real/SOURCE.md` carried a local path and `test.mjs` a
  design-doc section citation; and the README's expect example (one p7 twin)
  is refused against meridian's rows. Every new control went into
  `test.mjs` first and was run against the unchanged checker, walk and
  scan: 53 failures, exit 1. Then: the control leaves `PIN` out; a new
  control builds `gates/conformance/` in a temp directory, writes a real
  `PIN`, runs `--verify-pin` alone and `test.mjs --mutate` there (each exit
  0), and a planted-defects copy must be refused for each defect by name;
  `--write-pin` accepts only its sha (eight argv shapes controlled), its
  leftover scan cannot throw, and a temp file it cannot remove is named;
  every file in a real-row repo directory counts; the wording control also
  refuses a local path, a rule number, a ruling or contract id (added
  beyond P9's list, logged as a deviation), a section-sign citation, and a
  commit sha of this repo's history — the last read with git in this
  repo's layout, failing closed on a missing or shallow history, and
  printed as not evaluable in a vendored copy; an expect file with a
  duplicate JSON key is unevaluable, exit 2 (row files are not yet:
  undecided, listed in the README). A group with an UNEVALUABLE row, live
  included, derives UNEVALUABLE and is not credited per check: this already
  held, so there was no red to capture on the live tree; positive fixtures
  68 and 69 pin it, and two scratch mutants of `credit()` each turned the
  matching fixture red. The comment saying `--write-pin` made an empty
  `PIN` impossible under a shell redirect was false (the shell truncates
  first) and is corrected; `--verify-pin` refusing an empty `PIN` is now
  controlled. CI checks out full history and its vendored-shape step now
  writes a `PIN` with the workflow's commit, verifies it alone, then runs
  the self-test (rehearsed locally in Git Bash: exit 0). README and the
  design doc use `gates/conformance/` (§5 amended, addendum "vendoring path
  and pack fixes"); `DATUM.md` has a dated amendment. Green after, on
  Windows, Node 24.13.0: `ok conformance: 19 positive, 50 negative, 2
  real, 20 reasons, 16 rules mutated, 1 crediting rule mutated`; and, the
  first run anyone has made off Windows, on Linux (WSL2 Ubuntu, Node
  20.20.2, checksum-verified download), the same line for `--mutate` and
  `--check-status` in this repo's layout (git history readable, 9 commits)
  and for the CI vendored-shape sequence, each exit 0. That Linux run read
  the working tree through the Windows mount, not a GitHub runner, and
  Node 24 on Linux was not run. The
  README's seven-cell expect file exits 0 over a scratch copy of meridian's
  18 rows (2 of 7 CLAIMABLE, unchanged). `fixtures/real/meridian/*.json`
  bytes unchanged and still identical to meridian's `gates/out/`.
  **Not yet:** nothing committed or pushed, so CI has not run any of this;
  no governed repo re-vendored; meridian still vendors under `gates/datum/`
  at the round-1 pin.

- **2026-09-14** — Pack fixes after the round-3 refutation (P11 (a)-(g),
  decision D2), same coalescing round. **Three round-3 claims were false:**
  the `PIN` exclusion let an authored `PIN` through the wording scan in both
  layouts (in this repo's layout nothing verified it at all); file names
  were never scanned, and `--write-pin` copied them into `PIN`; and "catches
  a local path, a rule number, a section-sign citation and a commit sha"
  held only for the probe spellings (an MSYS or Windows user path, `rule
  #7`, `rule-7`, `R 3`, `P-9`, the `&sect;` entity, a sha after `_` or a
  letter, and a UTF-16 file all passed). Also refuted: the schema literal
  and PIN template were allowed in markdown prose, and fixture directories
  dropped a stray file or a subdirectory holding a case silently. Every new
  control went into `test.mjs` first and was run against the unchanged
  checker and scan: `conformance: 61 failure(s)`, exit 1, in this repo's
  layout. The duplicate-key row control was then sharpened (the planted row
  replaces the conforming live row) and re-run from a frozen copy against
  the unchanged checker: both cases `exit 0, expected 2`, the row accepted
  silently. Built, each against its named control: (a) fixture
  directories refuse a non-`.json` file and a subdirectory, and `fixtures/`
  refuses anything but its three directories; (b) a `PIN` in this repo's
  layout is refused and scanned, and in a vendored copy `PIN` is left out
  only after `verifyPin()` accepts it, which now also refuses a line naming
  a file outside the pack or a repeated line; (c) file and directory names
  are scanned, with the accidental spellings above caught, plus drive
  letters, home and profile variables, roman numerals of two letters or
  more, numeric entities and escapes of the section sign, and control and
  invisible characters; (d) the machine forms are allowed only in `.mjs` and
  `.json`; (e) a vendored file that is not valid UTF-8 is refused; (f) a row
  file with a duplicate key at any depth exits 2 naming file and key, and
  case files and real rows with one are refused; (g) the README documents a
  CI-order check that matches the seven-line vendored-shape step as
  consecutive lines. Also: `fixtures/real/` with no governed repo directory
  is refused by name; the count control and planted copy no longer name
  meridian; `--write-pin` with an empty or flag value is a usage error.
  A line-by-line audit of what the rewrite removed found one narrowing:
  the new rule-number pattern had lost the old one's case-insensitivity
  (`RuLeS 4` passed). A probe for it failed first (`not caught: got []`),
  and the pattern now reads the word in any case, roman numerals upper-case
  only.
  Green after, Windows, Node 24.13.0: `node conformance/test.mjs --mutate`
  and `--check-status` both `ok conformance: 19 positive, 50 negative, 2
  real, 20 reasons, 16 rules mutated, 1 crediting rule mutated`, exit 0; the
  CI vendored-shape step, run verbatim with the workflow's sha set to
  `df306e6`, printed `ok pin written`, `ok pin verified`, the not-evaluable
  note and the same line, exit 0. Round-3 probes replayed: every one is
  refused by name, the authored `PIN` in this repo's layout included. The
  README CI-order check passes this tree's workflow and fails, exit 1, on
  `df306e6`'s workflow, round 3's violating workflow, and comment-only,
  reordered and blank-line copies. Not caught, and said in the control's
  comment: a single roman letter, and a sha abbreviated below seven
  characters. `fixtures/real/meridian/*.json` bytes unchanged. **Not yet:**
  nothing committed or pushed, so CI has not run any of this; not run off
  Windows this round (WSL has no `node` on its path); no governed repo
  re-vendored.

- **2026-09-15** — The self-test's layout decision, after the adversarial
  pass over the entry above. **The entry above's claims that a `PIN` in this
  repo's layout is refused and that the commit-sha sub-check runs here did
  not survive one planted file:** `test.mjs` read the layout from the schema
  under `../schema/` with none beside `check.mjs`, so a stray copy of the
  schema beside `check.mjs` in a full-history checkout read as a vendored
  copy. There a `PIN` written by `--write-pin` with HEAD's sha, plus HEAD's
  abbreviated sha appended to `reasons.md`, passed. Reproduced in a copy of
  this repo with its `.git`: `node conformance/test.mjs --mutate` printed `ok
  conformance: 19 positive, 50 negative, 2 real, 20 reasons, 16 rules
  mutated, 1 crediting rule mutated`, exit 0. The nested self-test also
  exited 0 with the schema moved beside `check.mjs` rather than copied. The
  layout is now read from the schema at `../schema/`, or, with that file
  gone, from git: a work tree whose top holds this directory and whose index
  tracks `schema/gate-verdict.v1.json`. A schema beside `check.mjs` is not
  consulted, and in this layout it is refused by name (`check.mjs` would
  read it before the real one). A new control copies this repo with its
  history and plants the schema copy, a verifying `PIN` and HEAD's sha,
  once as a stray copy and once with the schema moved. The re-run self-test
  must exit 1, name all three, and print no not-evaluable note. In the same
  work tree, a clean vendored copy under `gates/` must still exit 0 with that
  note. Red first, with the control added and the old decision in place:
  `conformance: 10 failure(s)`, exit 1. With the `../schema/` signal alone:
  `conformance: 5 failure(s)`, all in the moved variant. Green after, both
  signals, on Windows with Node 24.13.0: the same `ok conformance` line as
  above, exit 0. A scratch mutant without the top-of-work-tree check fails
  the vendored-copy sub-control (`exit 1, expected 0`). Replayed by hand on
  the fixed tree, the stray-copy attack exits 1 naming the schema copy, the
  `PIN` and `reasons.md:158`. The CI vendored-shape step, run verbatim,
  exits 0 and prints a second note, `note: layout: ...`, saying the new
  control is not run in a vendored copy. `check.mjs` and `reasons.md` are
  byte-unchanged, and the generated block below re-renders identically.
  `README.md` gains one "Known limits" section: the former "What the pack
  does not check" list, the minor findings of that same pass (left unfixed
  by ruling), and the items earlier builders declined, each re-probed
  against this tree. **Not yet:** nothing committed or pushed, so CI has not
  run; not run off Windows (WSL has git but no `node` on its path); no
  governed repo re-vendored.

- **2026-09-15, later** — **The entry above's claim that no planted file can
  turn this repo's layout toward vendored did not survive the next
  adversarial pass:** with the schema moved beside `check.mjs`, a `.git`
  file planted inside `conformance/` stopped `git -C conformance` at the
  plant, so git never saw the enclosing repository; the `PIN` was accepted,
  HEAD's sha printed the note, and `test.mjs --mutate` exited 0. Git is now
  asked from the pack's parent directory, and a `.git` entry inside the pack
  directory is refused by name in either layout. The layout-plant control
  gains a third variant (the moved schema plus a planted `.git`) that must
  exit 1 naming the `.git`, the schema copy, the `PIN` and the sha. Red
  first, with the variant added and the old decision in place:
  `conformance: 5 failure(s)`, exit 1, all five in the new variant. Green
  after, Windows, Node 24.13.0: `node conformance/test.mjs --mutate` and
  `--check-status` exit 0 with the same `ok conformance` line as above, and
  the CI vendored-shape step, run verbatim with `GITHUB_SHA` set to HEAD,
  exits 0. By hand, the plant's nested self-test exits 1 naming the `.git`,
  the schema copy, the `PIN` and `reasons.md:158`; two scratch mutants (git
  asked from the pack directory again; the `.git` refusal removed) each make
  the new variant fail. Still open, and listed in "Known limits": with the
  schema moved, deleting `.git/index` makes the checkout read as vendored
  (nested self-test exit 0 with the note). **Not yet:** nothing committed
  or pushed; not run off Windows; no governed repo re-vendored.
- **2026-09-15, twin definition** — **Amendment: where a twin plants its
  defect (ruling R9).** Operator ruling, picked from three options
  ("Gates-local twin binary", the same defect behind a build tag in
  production source, "Accept PARTIAL for P2"): a twin may plant its one
  defect in a sibling of the component under test that the gate runs in the
  twin's place, not only in the input. Occasion: MERIDIAN P2's
  `fresh_process_identical` (two fresh-process replays of one feed by the
  production binary) has no data twin, because the fold is pure and the
  feed is its only input, and no environment twin, because the binary reads
  only its flags; under R3 the property could never leave PARTIAL.
  Precedent already in MERIDIAN: the P7 Readers. Recorded in `DATUM.md` as
  an amendment to rule 3. No row invariant, refusal reason or pack rule
  changes; `conformance/` is untouched and the pack MERIDIAN vendors stays
  the one at `a0eb130`. Design-round evidence, measured on a scratch copy
  of meridian `df4cf0f` with the composed twin set (the twin binary
  `gates/p2nondet` sets `fresh_process_identical` and `pinned_hash_match`
  nonzero with `chain_verifies` 0): `sh gates/run.sh` exit 0, last lines
  `ok lane1 claimable=6/7` and `ok conformance pack agrees: claimable=6`
  over 27 rows; a control dropping the seven new twin rows restores the
  2/7 derivation with the same unfalsified lists as at `df4cf0f`. The one
  property still PARTIAL is P3, on `positions_match_manifest` and
  `unevaluable_match_manifest`, which no corporate-action defect can move
  on a feed whose position and unevaluable sets are constant across its
  viewpoints; the operator ruled "Accept PARTIAL for P3" rather than
  extend the base feed or drop the two checks (dropping them would lose
  their V1 and V2 coverage: P3 evaluates 15 for each, P1 and P4 evaluate
  5). **Not yet:** the MERIDIAN change is not built in its repo; this entry
  records the ruling and the design-round measurement only.

<!-- pack:status:begin -->
generated by `node conformance/test.mjs --write-status` from a green run; CI compares with `--check-status`

| pack | value |
|---|---|
| self-test | 19 positive, 50 negative, 2 real, 20 reasons, 16 rules mutated, 1 crediting rule mutated |
| fixtures | 19 positive, 50 negative, 2 real (meridian) |
| refusal reasons | 20 |
| rules | 16: schema, unevaluable_reason_iff, planted_iff, evaluated_keys, evaluated_zero, live_violations_red, twin_zero_not_red, twin_plant_match, status_literal, set_one_live, set_twin_unique, set_live_red_human, set_twin_green_human, expect_unexpected_cell, expect_missing_cell, expect_twin_set |
| crediting rules | 1: credit_live_checks_falsified |
<!-- pack:status:end -->
