# Handoff -- conformance pack built, and survived its first adoption

2026-09-10. Newest commit this brief describes: **`d4f8dbf`**
(`fix: vendored self-test finds the schema beside itself`, main, level with
origin, clean tree). Pick-up measures drift from here. Cross-repo anchor:
meridian at **`cdee383`**, the first governed repo, whose CI is **red** for
a reason that belongs to this repo's tooling (see Open / next item 1).

This repo is private. Governed repos are not: a public repo may name DATUM
once as a private governing text and vendor the pack, and must cite only
evidence its own readers can check.

## Current state

- **built** -- the row, as a JSON Schema in the eight-keyword subset the
  design allows: `schema/gate-verdict.v1.json`.
  re-verify: `node -e 'const s=require("./schema/gate-verdict.v1.json");console.log(s.required.length,Object.keys(s.properties).length)'` -> `16 20` (16 required of 20 fields; `unevaluable_reason`, `planted`, `rows`, `metrics` are the conditional four)
- **built** -- the reference checker `conformance/check.mjs`: a subset
  validator over that schema, nine row rules and four set rules in an
  exported table, crediting per (surface, lane), `--json`,
  `--verify-pin`, `--write-pin`, exit codes 0 conform / 1 refused /
  2 unevaluable. Node builtins only.
  re-verify: `node -e 'import("./conformance/check.mjs").then(m=>console.log(m.RULE_NAMES.length,m.RULE_NAMES.join(",")))'` -> `13 schema,unevaluable_reason_iff,...,set_twin_green_human`
- **built** -- the checker's own controls `conformance/test.mjs`: every
  PASS fixture accepted, every FAIL fixture refused with a reason that
  starts with its `expect_reason`, no refusal outside `reasons.md`, no
  vocabulary entry that no fixture expects, real-row hash binding, CLI
  exit-code and PIN exercises, `--mutate`, `--write-status` /
  `--check-status`.
  re-verify: `node conformance/test.mjs --mutate` -> `ok conformance: 13 positive, 46 negative, 0 real, 17 reasons, 13 rules mutated`
- **built** -- the self-test runs from a vendored-shape copy, not only from
  this tree. This is the `d4f8dbf` fix; before it the vendored self-test
  died with `ENOENT`. Learnings:
  `docs/learnings/2026-09-10-vendored-self-test-only-ran-at-home.md`.
  re-verify: `d=$(mktemp -d) && cp -r conformance/. "$d" && cp schema/gate-verdict.v1.json "$d/" && node "$d/test.mjs" --mutate` -> the same `ok conformance:` line
- **built** -- CI on Node 20 and 24, running the self-test with `--mutate`
  and the STATUS freshness check.
  re-verify: `gh run list --repo hossainpazooki/datum --limit 1 --json headSha,conclusion --jq '.[0]|"\(.headSha[0:7]) \(.conclusion)"'` -> `d4f8dbf success`
- **built** -- STATUS.md's block is generated from a green run and compared
  in CI; nothing between the markers is typed.
  re-verify: `node conformance/test.mjs --check-status` -> `ok STATUS.md generated block is fresh`
- **built, once, for real** -- the vendoring contract has been exercised by
  a governed repo: meridian vendors the pack at `gates/datum/`, byte for
  byte identical to this tree at `d4f8dbf`, and its gate runs the
  self-test before its gates and the checker over its rows after them.
  re-verify: `for f in $(cd ~/dev/meridian/gates/datum && find . -type f ! -name PIN | sed 's|^\./||'); do s=$([ "$f" = gate-verdict.v1.json ] && echo schema/$f || echo conformance/$f); git show "d4f8dbf:$s" | diff -q - ~/dev/meridian/gates/datum/$f >/dev/null || echo "DIFFERS $f"; done; echo done` -> `done` with no DIFFERS line
- **measured, elsewhere** -- the pack's first contact with real rows was
  red for exactly the reasons the design predicted, then green after the
  emitter moved: meridian's 18 rows refused five ways per row (missing
  `schema`, `gate_sha`, `gate_worktree`; unknown `parallax_sha`,
  `parallax_worktree`), then accepted with seven surfaces CLAIMABLE.
  re-verify: `sh ~/dev/meridian/gates/run.sh >/dev/null 2>&1; node ~/dev/meridian/gates/datum/check.mjs ~/dev/meridian/gates/out | tail -1` -> `ok 18 rows conform, 7 surfaces`
- **not started** -- `conformance/fixtures/real/` is empty; no governed
  repo's rows are bound here yet. `SOURCE.md` exists with an empty hash
  list and `test.mjs` enforces the binding when rows appear.
  re-verify: `ls conformance/fixtures/real/` -> `SOURCE.md` only
- **not started** -- baseline and traverse. Neither has vendored anything;
  baseline still emits the pre-DATUM key names and needs supersession
  before its published rows can be re-emitted.
- **proposed, not built** -- `--write-pin` writing the pin file itself
  instead of streaming to a shell redirect. See Open / next item 1.

## Locked decisions

1. **Approach A: a repo with an executable pack**, over vendored text
   only or BASELINE's checker as the reference. Operator, 2026-09-06.
   Reason: a governing text without something that can refuse has
   governed nothing (rule 4 applied to itself).
2. **A fixture carries its expected outcome AND, when negative, its
   expected reason**, matched by prefix. Reason: rejection is not enough;
   rejection for exactly the planted reason is -- the twin rule applied to
   the checker. Borrowed from compiler test suites, which is where the
   named-reason practice exists; the conformance corpora surveyed record
   an outcome or an error class only.
3. **An unexpected refusal is itself a failure.** Reason: a checker that
   refuses for a reason nobody planted is not more careful, it is
   uncontrolled. `test.mjs` condition 3.
4. **The rule table is exported and `--mutate` disables each entry in
   turn**, requiring some negative fixture to stop being refused; and with
   every rule disabled, every negative fixture must be accepted. Reason:
   a rule no fixture can detect disabled is unfalsified, and a refusal
   outside the table is a rule nobody is testing.
5. **Rows are build output and stay uncommitted**, except one live and one
   twin per governed repo under `fixtures/real/`, hash-bound and re-copied
   on an emitter change, never edited. Reason: the hand-copy seam is
   unverified by construction, so it is bounded by a hash.
6. **The row's identity fields are emitter-neutral** (`gate_sha`,
   `gate_worktree`), the outcome is the exact three-value enum with the
   reason in its own field, and `schema` names the version. Operator,
   2026-09-06. Reason: both existing emitters wrote a key named for one
   project, one of them holding another project's commit.
7. **The vendoring contract is the file set, the pin and the CI order --
   never the path.** Added 2026-09-09 at the first adoption: Go reserves
   `vendor/`, which the design had named. Learnings:
   `docs/learnings/2026-09-10-go-reserves-the-vendor-directory.md`.
8. **Only the operator writes git history.** Emit commit commands,
   grouped by repo, one concern per commit.

## Reuse map

- `conformance/check.mjs` -- `ROW_RULES` and `SET_RULES` are the whole
  checker; `checkRows(entries, {disabled})` is the entry point both the
  CLI and `test.mjs` use. `credit()` derives the status vocabulary.
  `writePin()` / `verifyPin()` hash over LF-normalized bytes.
- `conformance/test.mjs` -- the three conditions, then vocabulary
  coverage, then the real-row binding, then CLI and PIN exercises, then
  `--mutate`. Add a new control next to the exercise it resembles.
- `conformance/reasons.md` -- the refusal vocabulary, each entry naming
  the rule-table unit behind it. A new refusal string must land here or
  `test.mjs` fails.
- `conformance/fixtures/` -- 13 positive, 46 negative, one case per file,
  `rows` (array) instead of `row` for set-level cases.
- `docs/2026-09-06-datum-design.md` section 10 -- what moved between the
  proposal and the build, including the two adoption findings.
- `~/dev/meridian/gates/` -- the only worked example of adoption:
  `run.sh` for the CI order, `gates/datum/PIN` for the pin shape,
  `claimability.py --render/--check` for a repo-side generated block that
  agrees with the pack.

## Invariants

- **The self-test must be exercised from a vendored-shape copy.** CI runs
  it at home; that arrangement cannot see a portability defect, and one
  shipped. Until CI does both, run the temp-copy line before any release.
- **Never edit a row or a fixture to make a checker pass.** Fix the
  emitter or the rule and re-copy.
- **Every rule needs a fixture that notices it disabled**, and every
  reason in `reasons.md` needs a fixture that expects it.
- **A pack that has only ever said yes has proven nothing** -- the first
  run against a governed repo's real rows must be a measured red, and a
  first run that is green means the checker was never pointed at the rows.
- **Exit 2 is never coerced into 0.** Unevaluable halts: no rows
  directory, a file that is not JSON, a pin that does not verify.
- **A governed repo may state it is governed only while its CI runs the
  pinned pack green on a pushed commit.** meridian does not currently
  qualify; see below.
- **Only the operator writes git history.**

## Open / next

1. **The pin is a footgun, and it has already fired.** `--write-pin`
   streams to stdout for a shell redirect, so the redirect truncates the
   target before node runs and any failure leaves a 0-byte pin that looks
   written. meridian committed and pushed exactly that, and its CI is red
   at `--verify-pin` (run 34410348602 on `cdee383`). The refusal is
   correct -- fail-closed worked, nothing was credited -- but the write
   path invites it. Fix: have `--write-pin` write `PIN` itself,
   atomically, and refuse to emit a pin whose line count is not the
   vendored file count plus one. Blocker: changing `check.mjs` moves every
   pin, so it lands together with a re-vendor in meridian; do not ship it
   alone. meridian's side is already fixed locally by writing the pin
   under a guard -- see its 2026-09-10 handoff entry.
2. **Bind meridian's real rows.** One live and one twin row (P7 is the
   most informative) copied into `conformance/fixtures/real/meridian/`,
   listed in `SOURCE.md` with LF-normalized sha256, STATUS block
   re-rendered. Blocker: rows must come from a clean tree at a pushed sha,
   so meridian's pin fix must land and its CI go green first -- today's
   rows carry `gate_worktree: dirty`.
3. **baseline.** Build `superseded_by` before re-emitting, since its two
   rows are published at URLs that would 404; then generalize its checker,
   rename in PARALLAX's emitter, re-emit against the gold surface (an
   operator step -- it needs the surface), re-bind `SOURCE.md`, vendor.
4. **traverse.** Born conforming: amend its design for the cell, plant,
   enum and reason fields before any code exists.
