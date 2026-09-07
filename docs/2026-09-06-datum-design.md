# DATUM conformance pack — design

2026-09-06. Status: **PROPOSED, nothing built.** Governs how `DATUM.md`
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
vendor/datum/check.mjs
vendor/datum/test.mjs
vendor/datum/fixtures/**
vendor/datum/reasons.md
vendor/datum/PIN            datum commit sha + "sha256  path" per vendored file, LF-normalized
```

and a CI step that runs, in order and failing on any non-zero exit:

```
node vendor/datum/test.mjs                    # the pack's own controls
node vendor/datum/check.mjs <rows-dir>        # the repo's emitted rows
```

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
