# What the conformance suites actually do

Research note, 2026-09-06. Status: **verified against primary sources** for
everything in §2–§5; **unresearched** for everything in §7. Written for the
DATUM design (a governing text plus an executable conformance pack that three
repositories vendor at a pinned commit) and kept as raw material for a later
essay. Every claim below names the document it was read from. Claims that a
skeptic refuted are kept, in §6, because what a first reading gets wrong is
part of the record.

## 0. The question

DATUM proposes to govern a shared machine-readable row across independent
repositories by shipping the schema with a corpus of positive and negative
cases, each carrying its expected outcome and, for negatives, the expected
reason for rejection; each governed repository vendors the corpus at a pinned
commit and runs it in CI over its own emitted rows. Before building that, the
question was whether it is an instance of something already named, and what
the established suites do on each axis:

1. how the fixture corpora are structured, and whether negative cases carry
   an expected **reason** or only a reject bit;
2. how implementations consume a suite, and whether any suite pins its
   consumers by content hash;
3. how a **claim** of conformance is gated;
4. how per-implementation reports are structured;
5. whether there is a name for the triple of governing text, executable
   corpus and per-implementation report.

## 1. Method

A five-angle web research workflow: one search agent per angle, fetch of the
top sources, extraction of falsifiable claims, then three independent
adversarial verifiers per claim, each instructed to refute, with two
refutations killing a claim. Nine merged claims survived out of eighteen
extracted; the refuted ones are in §6. Fetch agents ran on a small model, the
verifiers on a mid-tier model, the synthesis on the session model. One angle
(WebAssembly and compiler test frameworks, §4) was left open by the workflow
and was fetched directly from primary sources by the controlling session
afterwards. Run identifier `wf_81c3cb4c-0b8`; 103 agents; about 5.7 million
tokens.

Method caveat for the essay: three-vote refutation establishes that a claim
survived three attempts to break it against the cited page on the day it was
fetched. It does not establish that the page is the current normative text,
or that nothing else on the site contradicts it.

## 2. Corpus structure: every case carries its outcome, none carries a reason

The five corpus suites converge on one pattern. Each case records what a
conforming implementation must produce. For negative cases the record is a
bit, or at most an error **class**. No surveyed suite records the reason an
input must be rejected.

**Test262** (ECMAScript). A negative test's frontmatter is a YAML dictionary
with exactly two keys: `phase`, one of `parse`, `resolution`, `runtime`, and
`type`, the name of the error constructor. The test fails if no exception is
thrown, if the constructor name differs, or if the error occurs at a
different phase. There is no third, explanatory field. Read from
`INTERPRETING.md` in the tc39/test262 repository. Vote 3–0.

**JSON Schema Test Suite.** The test-object schema requires `description`,
`data` and `valid`, permits only those plus `comment`, and sets
`additionalProperties: false`. A negative case is therefore `valid: false`
plus, at most, an unstructured human comment. The README's own Known
Limitations section states the structural consequence: the suite "cannot test
against any behavior which a schema is incapable of representing, even if the
behavior is mandated by the specification", with URI normalization as the
worked example. Read from the README and `test-schema.json`. Vote 3–0.

**toml-test.** Valid cases pair a `.toml` file with a JSON encoding of the
expected parse and require a zero exit plus matching output. Invalid cases are
a bare `.toml` file with no expected-outcome artifact at all; the decoder
passes by returning a non-zero exit code. Read from the toml-test README.
Vote 2–1, the only split vote in the run; see §6 for the adjacent claims that
were refuted.

**CommonMark.** The cases are embedded in `spec.txt` as fenced example blocks
of markdown source and expected HTML, each with a section name and example
number. An external harness, `test/spec_tests.py`, takes an arbitrary
implementation executable and diffs its output against the embedded HTML,
filtering by section pattern or example number and reporting the example
number, line range and section on failure. Every case is a positive
input-to-output pair. There are no negative cases in the reject-with-reason
sense. Read from the commonmark-spec README and `spec_tests.py`. Vote 3–0.

**Web Platform Tests.** The corpus and the expectations are separated. The
corpus is indexed by an auto-generated manifest of test type, references and
timeout, derived from filenames and contents with no manual registration.
Expectations live in per-test metadata files owned by each implementation,
with fields such as `expected`, `disabled`, `fuzzy` and
`implementation-status`, and a conditional syntax for platform-specific
values. Expectation values are status enumerations, not reasons. The
metadata is regenerated programmatically from a run report
(`wptreport.json` applied by `mach wpt-update`). Read from the wptrunner
expectation docs and the Firefox web-platform docs. Vote 3–0.

The pattern across all five: **the corpus states what must happen; the
implementation's own failure text is never part of the contract.**

## 3. Consumption: submodule, subtree, harness, or vendored copy with a bot

- **JSON Schema Test Suite** recommends cloning `main` as a git submodule or
  git subtree and declares `main` always stable. No tag or release pinning is
  offered as an alternative. In practice python-jsonschema vendors it as a
  git subtree with repeated squash-pull commits and no `.gitmodules`; a 2022
  request for automated update pull requests was never fulfilled. Read from
  the README and issue #549, with the consumer's commit history checked by a
  verifier. Vote 3–0.
- **CommonMark and toml-test** are external harnesses that take an
  implementation binary. The implementation vendors nothing; it exposes an
  executable.
- **WPT in Gecko** is a vendored copy under `testing/web-platform/tests`,
  kept in two-way sync by a bot that opens upstream pull requests for
  Firefox-originated changes and pulls upstream changes into the tree. The
  corpus and the expectation metadata are version-controlled together in the
  implementation's repository. Read from the Firefox docs and mozilla/wpt-sync.
  Vote 3–0.

**No surveyed suite pins its consumers by content hash.** Consumers pin the
suite by commit, or not at all. Hashing the vendored files is something DATUM
would add, not borrow.

## 4. The expected-reason precedent is in compiler testing, not conformance suites

The workflow left open whether any widely used suite records an expected
failure reason. Three primary sources, fetched directly afterwards, show that
the practice exists and is standard in one tradition: tests of compilers and
validators, where the diagnostic is itself the product under test.

- **WebAssembly spec test scripts.** The assertion forms `assert_malformed`,
  `assert_invalid`, `assert_trap` and `assert_exhaustion` each take a failure
  string. The interpreter README states: "The failure string in assertions
  exists for documentation purposes. The reference interpreter itself checks
  that the string is a prefix of the actual error message it generates."
  Read from `interpreter/README.md` in WebAssembly/spec.
- **rustc UI tests.** `//~ ERROR` annotations associate an error level and
  message with a source line; the message is matched as a substring ("you
  don't have to write out the entire message"). Errors and warnings "are
  required to be exhaustively covered by line annotations", so a diagnostic
  on a line with no annotation fails the test. Read from the rustc dev guide,
  UI tests chapter.
- **GCC DejaGnu.** `dg-error`, `dg-warning`, `dg-message` and `dg-bogus` take
  a regexp; "if the text of that message is not matched by regexp then the
  check fails". Diagnostics not handled by any directive fail the test unless
  `dg-excess-errors` declares them expected. Read from the GCC internals
  manual, Test Directives.

Two halves, each with precedent: **the expected reason is matched loosely**
(prefix, substring, regexp), and **an unexpected reason is itself a failure**
(rustc's exhaustive annotation, GCC's excess errors). Neither half appears in
any of the five conformance corpora in §2.

## 5. Claim gating: Jakarta EE's TCK process is the one hashed, public regime

The Jakarta EE TCK process, version 1.4.2 as fetched on 2026-09-06, is the
only surveyed regime that binds a conformance claim to a content-identified
suite and to public evidence:

- a certification request must reference the TCK version, its SHA-256
  fingerprint and its download URL;
- TCK binaries are GPG-signed by the Specification Committee so consumers can
  verify authenticity;
- the results summary must be publicly visible with no password or sign-up,
  and must include the total number of tests run and passed.

Certification is described as on your honor, but the public-summary
requirement is a hard MUST for being acknowledged as compatible and using the
brand. Vendors including Oracle, Payara, GlassFish and Tomcat publish
unauthenticated result pages. Read from jakarta.ee, TCK Process. Vote 3–0.
The approval workflow itself (lazy consensus, veto rules) was **refuted** as
stated and is not established here.

## 6. What a first reading got wrong

Kept because a blog about verification should show the misses.

- "Test262 records a binary pass/fail." Refuted: it records phase plus
  constructor name, an error class.
- "CommonMark's cases are stored as JSON inside `spec.txt`." Refuted 0–3:
  JSON is the dump and npm export format, not the in-spec format.
- "toml-test is distributed only as a versioned binary, not a corpus."
  Refuted. Its distribution model and exact invalid-case metadata are less
  settled than the surviving 2–1 claim suggests.
- "WPT expectations are lists of multiple statuses." Refuted on precision.
- "Jakarta approval is 14-day lazy consensus with any-committer veto."
  Refuted 0–3.
- The controlling session's own earlier claim, before this run, that the
  Java TCK was "the closest model" for the claim rule: partly right. It is
  the closest for **gating**, and it is not a corpus model at all.

## 7. Not covered

Nothing survived on these, so they are unresearched rather than absent:

- CNCF Certified Kubernetes conformance: `[Conformance]` test tagging,
  results submitted by pull request, the badge, and whether a hash-pinned
  suite version is required.
- OCI distribution-spec conformance tests and their report format.
- W3C implementation reports: per-normative-requirement rows, "untested" or
  "not implemented" marking, generation from EARL results, and whether any
  practice maps each RFC 2119 MUST to a test identifier.
- Whether "TCK", "compatibility kit" or "tests are the spec" is used as a
  term of art outside Java.
- Any published critique arguing that negative cases should carry an expected
  diagnostic.

## 8. Implications for DATUM

1. **There is no canonical name to cite.** DATUM's pack is a composition:
   fixture corpus with in-case expected outcomes (all five suites), expected
   failure reason matched loosely (WebAssembly, rustc, GCC), unexpected
   failure reason as a failure (rustc, GCC), commit-pinned vendored copy
   (JSON Schema consumers, Gecko), and a claim gated on a hashed suite plus a
   public run count (Jakarta). Say that, rather than borrowing a name that
   does not fit.
2. **Keep the fixtures JSON with the outcome inside each case.** That is the
   one thing every corpus suite does, and it keeps the reference checker
   from being the only possible runner.
3. **Negative fixtures carry `expect_reason`, matched as a prefix.** The
   checker fails a negative fixture it rejects for a different reason, and
   fails any run that emits a reason no fixture expects. Both halves have
   precedent, and neither is in the conformance-suite tradition, so the
   design should cite the compiler-test tradition for them.
4. **The pin file is DATUM's addition.** No surveyed suite hashes what its
   consumers vendored. Document it as such, and pair it with the Jakarta
   shape: a claim of "governed by DATUM at commit X" is valid only while the
   repository's CI runs the pack at that commit and the run count is public.
5. **The per-implementation report borrowing is unverified.** The W3C
   implementation-report shape assumed in the design (rows per requirement,
   generated from the run, untested marked as such) did not survive to a
   cited claim. Either research it in a second round or present it in the
   design as the author's proposal, not a precedent.
6. **The JSON Schema limitation applies to DATUM.** A pack that expresses
   its assertions as a schema cannot test behaviour the schema cannot
   represent. DATUM's rules 2, 4, 7, 8, 9, 10, 11 and 12 are of that kind,
   and the governing text should keep saying which rules the pack enforces
   and which are prose.

## Sources

- tc39/test262, `INTERPRETING.md`
- json-schema-org/JSON-Schema-Test-Suite, `README.md`, `test-schema.json`,
  issue #549
- toml-lang/toml-test, `README.md`
- commonmark/commonmark-spec, `README.md`, `test/spec_tests.py`
- web-platform-tests.org, wptrunner expectation docs;
  firefox-source-docs.mozilla.org, web-platform; mozilla/wpt-sync
- jakarta.ee, Specification Committee, TCK Process 1.4.2
- WebAssembly/spec, `interpreter/README.md`
- rustc-dev-guide.rust-lang.org, UI tests
- gcc.gnu.org, GCC Internals, Test Directives

Raw run output with per-claim evidence and vote counts: the session
scratchpad file `conformance-research-2026-09-06.md`, not committed.
