# Learnings index

Pointers only; one fact per dated entry. A wrong entry is superseded by a new
entry with a `kills:` reference, never edited.

- [2026-09-10 -- the vendored self-test only ran at home](2026-09-10-vendored-self-test-only-ran-at-home.md) -- the pack shipped with green CI while its own self-test crashed once vendored (schema read from a path that exists only here); CI that runs the pack only at home cannot detect this class.
- [2026-09-10 -- Go reserves the vendor directory](2026-09-10-go-reserves-the-vendor-directory.md) -- a top-level `vendor/` makes every Go build in the module fail with `inconsistent vendoring`, so the vendoring contract is the file set, the pin and the CI order, never the path.
- [2026-09-10 -- a checker that selects its input can be fed past](2026-09-10-a-checker-that-selects-its-input-can-be-fed-past.md) -- a row hidden behind an upper-case extension was never read, and a status literal used as an object key was never scanned; a fixture corpus tests the judging, not the selecting.
