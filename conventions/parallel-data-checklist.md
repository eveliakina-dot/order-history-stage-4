# Parallel test-data checklist

Compiled from the lesson "Why AI-Generated Test Data Fails Under Concurrency".

## The one question
For every data item: *if two tests use this at the same time, can they interfere?* → **isolated** (no) or **shared** (yes).

## Four risk factors and their CI signatures
| Risk factor | What it looks like | CI signature |
|---|---|---|
| Duplicate values | the same unique field on two workers | consistent failure (409/422), every run |
| Shared state | a single-use or mutable resource used by two workers | ordering-dependent — whoever runs first wins |
| Timing | a race between workers on the same resource | intermittent |
| Hidden dependency | a test expects a value another test or the system produces | wrong precondition; often 100% failure |

## Why the model produces them
No runtime state · no cross-call memory · no worker coordination. Instructions ("make values unique") cannot close a structural gap.

## Countermeasures — build uniqueness into construction
- Worker-index namespace on every account/value (`w1-…`, `w2-…`).
- Per-run suffix on anything the database must not have seen before.
- One dedicated fixture per worker for tests that need a specific state (e.g. a user with zero orders).
- Shared fixtures used read-only; any test that writes gets its own.
- Values the system generates (IDs, order numbers, tracking numbers) are read at runtime, never pre-specified.
