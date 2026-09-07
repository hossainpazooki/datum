# DATUM

Date: 2026-09-06, v2. Status: **governing** for three data-platform
repositories — **baseline**, **traverse**, **meridian** — by operator ruling
of this date. PARALLAX is not governed but conforms as the emitter of
BASELINE's rows. VANTAGE is out of scope.

This is one reader's statement of the discipline as built in BASELINE,
MERIDIAN, PARALLAX and VANTAGE, written before TRAVERSE is built so that
TRAVERSE carries the same logic. Where a rule is read from committed code or
a committed design it says so; where it is inference it says so. Nothing
here is a claim about any repo's current state; STATUS.md in each repo is.
A rule becomes normative for a repo when that repo pins the conformance pack
(rule 4); until then the repo's STATUS.md is the record and this text is
the target.

DATUM is a composition of borrowed mechanics, not an instance of a named
model. No surveyed conformance regime governs a shared schema across
independent repositories by a vendored, hash-pinned corpus; the pieces it
borrows, and the one it adds, are named where they appear and traced in
`docs/2026-09-06-conformance-suite-precedents.md`.

## Scope: a discipline, not a catalog

The shared thing is the discipline and the row, never a page. BASELINE is
not a catalog; MERIDIAN's cells do not register into it; no repo's rows are
copied into another's ledger. Basis, measured 2026-09-06: BASELINE's checker
refuses all 18 MERIDIAN rows (a PARALLAX check name is bound into its `rows`
rule, and it cannot hold a second twin per surface and lane), and its page
builder refuses a second surface by design. This supersedes MERIDIAN design
§7 ("MERIDIAN's cells land as BASELINE catalog rows") and the first open
item of MERIDIAN's 2026-09-03 closing handoff; both need a dated amendment
in MERIDIAN.

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

In a row the outcome is the exact enum `GREEN | RED | UNEVALUABLE`, and the
reason is its own field, `unevaluable_reason`, required when the outcome is
UNEVALUABLE and forbidden otherwise. A reason folded into the outcome string
is rejected by the checkers that exist.

Read from: MERIDIAN property 4 (missing price is a durable unevaluable
record), BASELINE denaming sweep (missing list is UNEVALUABLE, never clean),
TRAVERSE design (instance type with no floor is UNEVALUABLE, not FAIL).
Enum form read from BASELINE `scripts/lib/ledger.mjs`.

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
  expectation, key for key over the union of both key sets, and refuses the
  credit on any mismatch — a planted check never computed, and a computed
  violation never planted, are both refusals;
- twins that plant different defects are distinguishable by their planted
  block, so a duplicated twin cannot count twice;
- a property with several twins is credited only when every one of them is
  red as planted; one red twin does not credit a property that plants three
  defects.

A negative control names its reason. The checker is not credited if it
refuses for another reason, and a refusal nobody planted is itself a
failure. Neither half is found in the conformance corpora surveyed (Test262,
JSON Schema Test Suite, toml-test, CommonMark, WPT record an outcome or an
error class, never a reason); both are standard in compiler testing
(WebAssembly's `assert_malformed` prefix match; rustc's exhaustive `//~
ERROR` annotations; GCC's `dg-error` and excess-errors failure).

Read from: BASELINE README crediting rule and `scripts/lib/ledger.mjs`
(union-of-keys plant match); MERIDIAN `gates/claimability.py` (twins
grouped by `planted.mutation`, every twin must independently check out;
BASELINE's `groupCells` cannot hold two twins, so the multi-twin rule is
MERIDIAN's); MERIDIAN design ("if a property can't get a twin, it isn't a
property, it's a hope").

### 4. The checker has its own negative controls, and says what it cannot check

The thing that checks the rows is itself a gate, so it needs its own twins:
fixtures that must make the checker fail, each carrying the reason it must
fail for. A checker that has only ever said yes has proven nothing. These
run in CI before anything is built or published.

A checker whose assertions are expressed as a schema can test only what the
schema can represent, even where the specification mandates more. So the
checker states what it does not check, and "governed" is never read as
"machine-checked" for rules the checker cannot reach.

Applied to DATUM itself: the conformance pack under `conformance/` — a
schema for the row, a reference checker, the checker's own controls, and a
fixture corpus in which every case carries its expected outcome and every
negative case its expected reason — enforces rules 1, 3, 5 and 6. Rules 2,
7, 8, 9, 10, 11 and 12, and this rule's own prose, are not enforced by the
pack, and each governed repo's DATUM section says so rule by rule. Design:
`docs/2026-09-06-datum-design.md`.

Read from: BASELINE `scripts/test-ledger.mjs` (positive and negative
controls for `check-ledger.mjs`), MERIDIAN `gates/importpin.py
--self-test`. The limitation is read from the JSON Schema Test Suite
README, which states it of itself.

### 5. Status is derived, never authored

The claimability table, status block, or status column is computed from
rows at build time. A status literal found in a row, or a status typed into
the page, fails the build. The committed rendering is compared against a
fresh rendering in CI; any drift fails.

House rule for STATUS.md, all governed repos: the dated narrative record is
hand-written and corrected in place (rule 12); the claimability table or
status block is generated between markers in the same file and compared
against a fresh rendering by `--check`. Both live in one STATUS.md.

Vocabulary where the siblings agree: CLAIMABLE, PARTIAL, UNCLAIMED,
UNEVALUABLE for a lane; GREEN and RED for a cell.

Read from: BASELINE README and `scripts/build.mjs --check` (the page is
generated; BASELINE's STATUS.md is hand-written, contrary to what v1 of
this text said), MERIDIAN STATUS.md (hand-written record, derived
claimability table).

### 6. Identity is content plus code plus worktree

A row is bound to three things: a content hash of what was gated, with the
basis of that hash written next to it (what bytes, what canonical order,
what library version); the commit SHA of the emitter, in the field
`gate_sha`; and whether that emitter's worktree was clean, in
`gate_worktree`. A row from a dirty worktree is evidence about code nobody
else can check out. Hashes are taken over normalised bytes when the consumer
is newline-insensitive, because a pin stricter than its own parser only ever
produces false alarms.

The field names are emitter-neutral on purpose. Both existing emitters
write the literal key `parallax_sha` — MERIDIAN's holding a MERIDIAN commit
— and both change to `gate_sha`; BASELINE's rows are re-emitted at the new
key, never edited.

Read from: BASELINE verdict rows (content_hash, content_hash_basis, and the
sha and worktree fields under their old names), BASELINE SOURCE.md
(LF-normalised hashing, caught in CI 2026-08-31).

### 7. Nothing is a claim until the row sits on a pushed SHA

A verdict in a working tree is a draft. It becomes evidence when the commit
that contains it is on the remote, and the code it names is reachable from
that commit. Gates that depend on other gates (teardown, cost backfill, lane
parity) are UNEVALUABLE until the rows they depend on are pushed.

For runs that can be regenerated (rule 8's second case), the pushed SHA is
the code and fixtures, and the CI run that regenerated the rows is where
the rows live; a regenerable row is a claim when that run is green on a
pushed commit.

A conformance claim is a claim like any other: a repo may state "governed
by DATUM at `<sha>`" only while its CI runs the pinned pack green, in
public, on a pushed commit. Borrowed from the Jakarta EE TCK process, the
one surveyed regime that binds a claim to a hashed suite and a public run
count.

Read from: TRAVERSE design header and teardown gate, MERIDIAN design header,
MERIDIAN STATUS.md correction history (a paragraph that said "not pushed"
was corrected in place when it became false, never deleted). Claim rule
read from jakarta.ee, TCK Process 1.4.2.

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

Anything hand-copied across a repository boundary is hash-bound, rows and
the pack alike: a governed repo vendors the conformance pack at a pinned
DATUM commit with a sha256 per vendored file, and CI recomputes them. This
is DATUM's addition; the suites surveyed pin by commit or track a branch,
and none identifies its consumers' copies by content.

Read from: BASELINE ledger/SOURCE.md (committed rows, hash-bound); MERIDIAN
STATUS.md (gates/out gitignored, regenerated every run). Pack binding is
this text's own rule; see the design.

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

This governs records: rows, STATUS entries, learnings, handoffs. A
governing text is not a record. It is revised, its history is git, and each
revision carries a section naming what changed and why — this file's own
practice, below. Applying the record rule to a governing text produces a
document whose header is false and whose truth is in an appendix.

Read from: MERIDIAN STATUS.md 2026-09-01 and 2026-09-03 corrections.

## The row

The row's field names live in one place, `schema/gate-verdict.v1.json`, and
nowhere in prose. A table of fields typed into this text drifted from both
checkers before anyone ran it, which is rule 5's lesson applied to
documentation. What the text owes the reader is the invariants a row must
satisfy, all of which the pack enforces:

- it names its kind, its schema version, its surface, its lane, and its
  cell (rules 1, 3);
- a twin carries its planted expectation and a live row carries none
  (rule 3);
- its outcome is the exact three-value enum, with the reason in its own
  field (rule 2);
- every check it reports has an evaluated denominator, and a denominator
  of zero forces UNEVALUABLE (rule 2);
- it is bound to content, code and worktree under emitter-neutral names
  (rule 6);
- `rows` is optional in the shared row — a repo may require it and bind it
  to a named check in its own checker, as BASELINE does; `lane` is any
  integer from 1 whose meaning the repo's design defines; `runner` is any
  non-empty string, with `local` and `ci` conventional;
- repo-specific measurements (throughput, latency, cost, dataset revision
  and licence) go under `metrics` and never displace a field above.

Field names are BASELINE's except where v1 of this text was wrong about
them (rule 6); MERIDIAN adopted BASELINE's on day one, so both emitters
change together, by the same diff.

## Changes from v1

v1 is the 2026-09-06 brief plus nine appended amendments, on `a5299a5`.
v2 folds the amendments into the body and makes the changes below.

- Status: proposed → governing, with the three-repo scope and PARALLAX's
  emitter-only role. Ruling recorded in the header.
- "Not a catalog" added as a scope section, with the measured basis and
  the MERIDIAN texts it supersedes.
- Rule 2: exact enum plus `unevaluable_reason`; the `UNEVALUABLE:<reason>`
  form is withdrawn (TRAVERSE's design uses it and changes).
- Rule 3: union-of-keys mechanic stated; multi-twin crediting added and
  attributed to MERIDIAN; the named-reason and unplanned-refusal clauses
  added with their compiler-test precedents.
- Rule 4: "says what it cannot check" added; the pack and the list of
  which rules it enforces added.
- Rule 5: house rule for STATUS.md decided; the false statement that
  BASELINE's STATUS.md is generated corrected.
- Rule 6: `gate_sha` / `gate_worktree` replace the `<emitter>_sha`
  placeholder no emitter implemented.
- Rule 7: reconciled with rule 8 for regenerable runs; the conformance
  claim clause added with its Jakarta precedent.
- Rule 8: the vendored pack brought under the hash-binding rule, marked as
  DATUM's addition.
- Rule 12: records distinguished from governing texts; this section is the
  consequence.
- The row table removed in favour of invariants and the schema file.
- The TRAVERSE gap list moved to the traverse repo
  (`docs/2026-09-06-datum-gaps.md`), where a per-repo report belongs under
  rule 1. One gap added there: the outcome form under rule 2.
- "What this brief does not decide" removed: the catalog and STATUS
  questions are decided above; floor provenance is TRAVERSE's open item.
