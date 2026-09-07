# DATUM

Date: 2026-09-06
Status: Proposed. This is one reader's statement of the discipline as built in
BASELINE, MERIDIAN, PARALLAX and VANTAGE, written before TRAVERSE is built so
that TRAVERSE carries the same logic. Where a rule is read from committed code
or a committed design it says so; where it is my inference it says so. Nothing
here is a claim about any repo's current state; STATUS.md in each repo is.

**2026-09-06, later.** Status changed from Proposed to **governing** for
baseline, traverse and meridian by operator ruling; see Amendments at the
end of this file. The body above is kept as written.

## One sentence

A data platform is credited by what its gates have shown, never by what its
author says, and a gate counts only after it has been seen to fail.

## The logic, in order

Each rule below depends on the one before it. Drop one and the ones after it
stop meaning anything.

### 1. The record is a row, not a sentence

Every claim the repo makes resolves to a machine-checkable row: a verdict
emitted by a gate run, carrying the identity of the code that ran it and the
content it ran over. Prose points at rows. Prose never carries a count, a
status, or a number that a row does not carry first.

Read from: BASELINE README ("nothing on the page may claim more than the
rows"), MERIDIAN README ("this README never carries counts").

### 2. A gate has three outcomes

PASS, FAIL, and UNEVALUABLE with a stated reason. Unevaluable is not a soft
pass and not a soft fail; it halts the thing that depends on it. A missing
floor, a missing price, a quota not granted, a denylist file that is absent:
all of these are the third outcome, never coerced into the first two.

Read from: MERIDIAN property 4 (missing price is a durable unevaluable
record), BASELINE denaming sweep (missing list is UNEVALUABLE, never clean),
TRAVERSE design (instance type with no floor is UNEVALUABLE, not FAIL).

### 3. A gate that has never gone red is not a gate

Every gate ships with a twin: a copy of the input with exactly one planted
defect. The gate is credited only when the live input passes AND the twin
fails FOR EXACTLY THE PLANTED REASON. Failing for some other reason is not a
red twin, it is a broken gate. Twins are written before the stage they test
is allowed to pass.

The mechanics that make this checkable, not just stated:

- the verdict row names its cell (live or twin);
- a twin row carries the planted expectation (which checks, how many
  violations, which mutation);
- a checker compares the row's actual violations against the planted
  expectation, key for key, and refuses the credit on any mismatch;
- two twins that plant different defects are distinguishable by their
  planted block, so a duplicated twin cannot count twice.

Read from: BASELINE README crediting rule, MERIDIAN gates/claimability.py
(re-derives red-as-planted from the row's own contents), MERIDIAN design
("if a property can't get a twin, it isn't a property, it's a hope").

### 4. The checker has its own negative controls

The thing that checks the rows is itself a gate, so it needs its own twins:
fixtures that must make the checker fail. A checker that has only ever said
yes has proven nothing. These run in CI before anything is built or published.

Read from: BASELINE scripts/test-ledger.mjs (positive and negative controls
for check-ledger.mjs), MERIDIAN gates/importpin.py --self-test.

### 5. Status is derived, never authored

The claimability table, STATUS page, or status column is computed from rows
at build time. A status literal found in a row, or a status typed into the
page, fails the build. The committed rendering is compared against a fresh
rendering in CI; any drift fails.

Vocabulary where the siblings agree: CLAIMABLE, PARTIAL, UNCLAIMED,
UNEVALUABLE for a lane; GREEN and RED for a cell.

Read from: BASELINE README and scripts/build.mjs --check. Note: MERIDIAN
keeps a hand-written dated STATUS.md as its state of record and derives only
the claimability table; TRAVERSE's design follows BASELINE (STATUS.md
generated, never hand-written).

### 6. Identity is content plus code plus worktree

A row is bound to three things: a content hash of what was gated, with the
basis of that hash written next to it (what bytes, what canonical order,
what library version); the commit SHA of the emitter; and whether that
emitter's worktree was clean. A row from a dirty worktree is evidence about
code nobody else can check out. Hashes are taken over normalised bytes when
the consumer is newline-insensitive, because a pin stricter than its own
parser only ever produces false alarms.

Read from: BASELINE verdict rows (content_hash, content_hash_basis,
parallax_sha, parallax_worktree), BASELINE SOURCE.md (LF-normalised
hashing, caught in CI 2026-08-31).

### 7. Nothing is a claim until the row sits on a pushed SHA

A verdict in a working tree is a draft. It becomes evidence when the
commit that contains it is on the remote, and the code it names is reachable
from that commit. Gates that depend on other gates (teardown, cost backfill,
lane parity) are UNEVALUABLE until the rows they depend on are pushed.

Read from: TRAVERSE design header and teardown gate, MERIDIAN design header,
MERIDIAN STATUS.md correction history (a paragraph that said "not pushed"
was corrected in place when it became false, never deleted).

### 8. Rows that cannot be regenerated cross a hand-copy seam, and the seam is guarded

Some runs cannot be repeated in CI: a rented GPU, a 440k-row vendor read, a
run on a box that no longer exists. Their rows are committed, not
regenerated. The seam where a human copies a row into the repo is
unverified by construction, so it is bounded: a source file binds every
committed row to its sha256 at copy time, the checker recomputes each hash,
and any unlisted, missing, or altered file fails the build. The replay
command is written next to the binding.

Where runs CAN be regenerated, rows are build output and are not committed;
the record is the state-of-record file and CI regenerates the rows on every
push.

Read from: BASELINE ledger/SOURCE.md (committed rows, hash-bound); MERIDIAN
STATUS.md (gates/out gitignored, regenerated every run). TRAVERSE is the
former case and its design does not yet name the binding file or the
checker (inference from the design doc, 2026-09-05).

### 9. Effects are probed, never read from the action's own report

A deploy that says it deployed, a destroy that says it destroyed, a run that
prints ok: each is a claim about the action. The effect is verified by
probing the resulting state with a probe that can tell presence from
absence. Terraform state is an index, not a decision; the teardown verdict
comes from an inventory query by tag that must return empty, and it is run
a second time later to catch stragglers. A probe that would return the same
answer whether or not the effect happened is vacuous and earns nothing.

Read from: TRAVERSE design teardown gate; MERIDIAN vacuity-guard learning
(2026-09-01); rigor verify-the-effect skill.

### 10. Thresholds are declared with their provenance

A floor, a tolerance, a margin is an authored number, and an authored number
is a status in disguise. Each one is declared in one file, keyed by the
condition it applies to (instance type, lane, fixture), with where it came
from written beside it. A gate over a floor proves the run cleared a number
somebody chose; the twin proves the gate can go red; neither proves the
floor is right. Say so.

Inference from rule 5 applied to TRAVERSE's gates/floors.json; BASELINE
forbids authored status literals but has no floors, so this rule has no
committed precedent yet.

### 11. Scope walls stand before code

The README names what the repo does not do, what it will not claim, and
what would be a lie if the slug or the prose implied it. A claim ceiling
lists what may be claimed after the last planned run and everything outside
it is unclaimed by construction. Precise verbs everywhere: we validate, we
use as a baseline, we propose, future extension.

Read from: MERIDIAN README scope walls ("no production claim", "no
performance vocabulary"), TRAVERSE design claim ceiling.

### 12. Corrections are appended in place, dated, never erased

When a row or a paragraph becomes false, the correction is written next to
it with the date and what was true when the original was written. History
of the record is part of the record.

Read from: MERIDIAN STATUS.md 2026-09-01 and 2026-09-03 corrections.

## The row, as the siblings shape it

The minimal verdict row that satisfies rules 1, 3, 6 and 7. Field names are
BASELINE's; MERIDIAN adopted them on day one so registration is a copy, not
a translation.

    kind            GATE_VERDICT
    surface | stage what was gated
    lane            which lane of the build
    cell            live | twin
    result          GREEN | RED | UNEVALUABLE:<reason>
    checks          { check_name: violation_count }
    evaluated       { check_name: rows_evaluated }
    planted         twin only: { mutation, mutated_rows, expected_violations: {...} }
    scope           what subset, in words
    params          the inputs that select the run
    content_hash    sha256:...
    content_hash_basis  what bytes, what order, what library version
    <emitter>_sha   commit of the code that emitted the row
    <emitter>_worktree  clean | dirty
    ran_at          UTC timestamp
    runner          local | ci | <instance>

Repo-specific measurements (throughput, latency, cost per thousand, price
per hour used, dataset revision and licence) go under a metrics block and do
not displace the fields above.

## Where TRAVERSE's 2026-09-05 design stands against this

Verified against the design doc, not against any code (none exists).

- Carries rules 2, 3 (the statement), 5, 7, 9, 11 as written.
- Rule 3 mechanics: missing. No cell field, no planted block, PASS/FAIL
  instead of GREEN/RED, "a twin fixture that must fail" rather than "fails
  for exactly the planted reason".
- Rule 4: partially. Gates have twin fixtures; the status builder's own
  negative controls are not named.
- Rule 6: partially. git_sha present; no worktree flag, no content hash
  basis (manifest shas serve as content identity, basis unstated).
- Rule 8: the binding file and recomputing checker are not named.
- Rule 10: floors.json exists; provenance of floors unstated.
- Rule 12: no statement; adopt MERIDIAN's practice.
- Runtime twins (tc throttle, cgroup io.max) are Linux-only and need the
  rented box, so they are not CI-reproducible; the siblings' twins all run
  in CI. Day 3's fresh-clone CPU reproduction covers fixture twins only.

## What this brief does not decide

Whether TRAVERSE registers into BASELINE's catalog (different domain; the
row schema costs nothing to adopt either way). Whether floors are set from
a prior public benchmark or from the first run's own numbers, which would
make the first run's PASS circular. Whether MERIDIAN's hand-written
STATUS.md or BASELINE's generated one is the house rule; the two coexist
today and this brief records both.

---

# Amendments — 2026-09-06 (governing)

The text above is kept as written on 2026-09-06 morning. Each amendment
below is dated and names what it corrects; per rule 12 nothing above is
erased. Where an amendment and the text disagree, the amendment governs.

1. **Status: governing, not proposed.** Operator ruling, 2026-09-06: DATUM
   is the governing document for three data-platform repositories —
   **baseline, traverse, meridian**. PARALLAX is not governed but conforms
   as the emitter of BASELINE's rows. VANTAGE is out of scope. A rule
   becomes normative for a repo when that repo pins the conformance pack
   (amendment 7); until then the repo's own STATUS.md is the record and
   this document is the target.

2. **BASELINE is not the catalog.** Operator ruling, 2026-09-06: MERIDIAN's
   cells do not register into BASELINE, and no repo's rows are copied into
   another's ledger. The shared thing is the discipline and the row, not a
   page. This supersedes MERIDIAN design §7 ("MERIDIAN's cells land as
   BASELINE catalog rows") and the first "Open / next" item of MERIDIAN's
   2026-09-03 closing handoff; both need a dated amendment in MERIDIAN.
   Basis for the ruling, measured the same day: BASELINE's checker refuses
   all 18 MERIDIAN rows (`rows must equal evaluated.no_future_accepted`, a
   PARALLAX check name baked into the checker; and a second twin per
   surface/lane, which `groupCells` cannot hold), and `build.mjs` refuses a
   second surface by design.

3. **The row, corrected.** The table under "The row, as the siblings shape
   it" says field names are BASELINE's and that MERIDIAN adopted them. True
   of fifteen keys, false of the identity keys, and the table itself
   diverges from both checkers. Row schema v1 (design doc
   `docs/2026-09-06-datum-design.md`) fixes each measured drift:
   - `schema: "datum/gate-verdict/1"` — new required key.
   - `gate_sha`, `gate_worktree` replace the `<emitter>_sha` placeholder.
     No emitter implemented the placeholder: PARALLAX and MERIDIAN both
     write the literal key `parallax_sha`, MERIDIAN's holding a MERIDIAN
     commit. Both emitters change; BASELINE's two rows are re-emitted, never
     edited.
   - `result` is the exact enum `GREEN | RED | UNEVALUABLE`. The reason
     moves to a separate `unevaluable_reason`, required when the result is
     UNEVALUABLE and forbidden otherwise. The `UNEVALUABLE:<reason>` form in
     the table is rejected by BASELINE's checker today and appears in
     TRAVERSE's design; TRAVERSE changes.
   - `rows` is optional in the shared row. A repo may require it and bind
     it to a named check in its own checker, as BASELINE does.
   - `lane` is an integer from 1 whose meaning each repo defines in its
     design. `runner` is any non-empty string; `local` and `ci` are the
     conventional values.
   - Multi-twin crediting is the shared rule, taken from MERIDIAN: at most
     one live row per surface and lane; twins unique by `planted.mutation`;
     credit requires the live row GREEN and every twin RED with `checks`
     equal to `planted.expected_violations` over the union of keys and at
     least one non-zero.

4. **Rule 5, corrected and decided.** The text says TRAVERSE follows
   BASELINE in generating STATUS.md. BASELINE's STATUS.md is hand-written;
   only its page is generated. House rule, operator ruling 2026-09-06: the
   dated narrative record is hand-written and corrected in place (rule 12);
   the claimability table or status block is generated between markers in
   the same STATUS.md and compared against a fresh rendering by `--check`.
   Applies to all three governed repos.

5. **Rules 7 and 8, reconciled.** For runs that can be regenerated, the
   pushed SHA is the code and fixtures, and the CI run that regenerated the
   rows is where the rows live. A regenerable row is a claim when that run
   is green on a pushed commit. Rule 7's "the row sits on a pushed SHA"
   applies unchanged to hand-copied rows (rule 8's first case).

6. **Rule 3's mechanics, attributed.** The fourth mechanic (twins
   distinguished by their planted block, so a duplicate cannot count twice)
   is read from MERIDIAN `gates/claimability.py`, which groups twins by
   `planted.mutation`. BASELINE's `groupCells` cannot hold two twins at all.
   The checker's own negative controls (rule 4) are read from BASELINE
   `scripts/test-ledger.mjs` and MERIDIAN `gates/importpin.py --self-test`.

7. **Rule 4 applied to DATUM: the conformance pack.** DATUM ships a JSON
   Schema for the row, a reference checker, the checker's own negative
   controls, and a fixture corpus in which every case carries its expected
   outcome and every negative case its expected failure reason. Each
   governed repo vendors the pack at a pinned DATUM commit, hashes the
   vendored files in a pin file, and runs the pack in CI over its own
   emitted rows. The pack enforces rules 1, 3, 5 and 6. Rules 2, 4, 7, 8,
   9, 10, 11 and 12 are prose, and each repo's DATUM section says so rule by
   rule. A repo may say "governed by DATUM at commit X" only while its CI
   runs the pack at X green. Design: `docs/2026-09-06-datum-design.md`.
   Precedents and their limits: `docs/2026-09-06-conformance-suite-precedents.md`.

8. **TRAVERSE's gap list, one addition.** Its design uses
   `UNEVALUABLE:quota-not-granted`; under amendment 3 that is
   `result: UNEVALUABLE` plus `unevaluable_reason`. The other gaps stand as
   listed.

9. **"What this brief does not decide", updated.** The catalog question is
   decided (amendment 2). The STATUS house rule is decided (amendment 4).
   Floor provenance for TRAVERSE remains open.
