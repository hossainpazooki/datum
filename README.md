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
```

A governed repo vendors `conformance/` (plus a copy of
`schema/gate-verdict.v1.json` beside `check.mjs`) and a `PIN` written by
`check.mjs --write-pin <datum-sha>`; its CI runs `test.mjs`, then
`check.mjs <rows> --verify-pin`.

## Where things are

`DATUM.md` the rules, the row's invariants, and what changed since v1 · `schema/` the row ·
`conformance/` checker, controls, fixtures · `docs/` design and research
notes · dated state lives in `STATUS.md`, not here.
