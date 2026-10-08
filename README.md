# Order History, Stage 4: Test Data That Survives Four Workers

⌨️ **Stage 4 of 4.** Steps, page object, DTOs — done. The last thing the suite needs is data, and this is where the model is going to fail, on purpose, in front of you.

Every value Claude generates will look fine on its own. Put four workers on the same dataset at once and some of those values collide. Your job: predict where, prove it, then make the model generate data that *can't* collide — not by asking nicely, but by changing how the data is built.

**You'll need:** a fork of `order-history-stage-4` · `constraints/parallel-run.md` · `conventions/parallel-data-checklist.md` · `prompts/04-test-data.md` · your `tests/steps/`, `src/pages/`, `src/components/`, `src/dto/` copied in from the three earlier forks · a new Claude conversation.


### 1. Read the constraints as facts

Open `constraints/parallel-run.md`: the suite, how it splits across four workers, and the database rules. Not suggestions — the rules the collisions come from.

In `review-notes.md › Stage 4 › Before generating`: one line listing the data items that are **isolated** (no two tests can interfere through them), then one line each for the items that are **shared** — and why.

### 2. Let Claude draft it — the naive way

New conversation. Paste `prompts/04-test-data.md` with the constraints file in the slot. It asks for "realistic and consistent" data. Don't improve it. Send.

Save the exchange, untouched, as `claude-runs/04a-test-data-naive.md`.

### 3. Find the collisions

Lay the draft out worker by worker. Every collision goes in `Stage 4 › Collisions found`:

*item · which workers · what breaks at runtime, and why*

Expect at least two. If you see none, reread the constraints — a value that's valid on its own is exactly what a collision looks like.

*Fewer than two in your draft?* Then also review `samples/checkout-parallel-scenario.md` — a different suite with four seeded collisions — and list them under `Stage 4 › Control pass` the same way.

### 4. Generate again — engineered

Change the construction, not the wording. Write a prompt that builds uniqueness in: a worker prefix on every account (`w1-`, `w2-`…), a per-run suffix on anything the database must not have seen, one dedicated zero-order user per worker, the shared fixture read-only, and database-generated values left for the test to *read*, never pre-specified. Send it; save as `claude-runs/04b-test-data-engineered.md`.

In `Stage 4 › Scheme`, 3–5 lines: which technique closes which collision from step 3.

### 5. Deliver

`data/order-history.data.json` — the engineered dataset, checked by you against the constraints once more. Every collision from step 3 must be impossible in it by construction. Statuses from your `OrderStatus`; test names from your Stage 1 steps.

### 6. One sentence

`Stage 4 › Hardest call`: why doesn't "make every value unique" in the prompt fix this — and what does?

### Submit

Commit, push, submit the repo link. Last stage: the reviewer reads `data/`, both `claude-runs/04*` files and your Stage 4 notes — and checks the repo holds together: test names in the data match the steps, statuses match the DTO, every step's Action has a method.

### ✅ Before you submit

- `claude-runs/04a-…naive.md` and `04b-…engineered.md` — exact prompts, full responses, untouched
- Notes › Before generating — isolated items in one line; each shared item with a reason
- Notes › Collisions found — 2+ collisions: item · workers · what breaks and why (Control pass only if your draft gave fewer than two)
- Notes › Scheme — which technique closes which collision
- `data/order-history.data.json` — no shared mutable value across workers; no pre-specified DB-generated values; statuses from `OrderStatus`; test names from Stage 1
- Notes › Hardest call — the structural gap, and the fix that doesn't depend on wording
