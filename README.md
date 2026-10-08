# Order History, Stage 4: Test Data That Survives Four Workers

⌨️ **Stage 4 of 4.** Your steps, page object and DTOs are ready. The last thing the suite needs is data — and this is the stage where the model is going to fail, on purpose, in front of you.

Every value Claude generates will be valid on its own. Put four workers on the same dataset at the same time, and some of those values will collide. Your job is to predict where, prove it, and then make the model generate data that can't collide — not by asking nicely, but by changing how the data is built.

**You'll need:** a fork of `order-history-stage-4` · `constraints/parallel-run.md` · `conventions/parallel-data-checklist.md` · `prompts/04-test-data.md` · `samples/checkout-*` · your files from the three earlier forks, copied in: `tests/steps/`, `src/pages/`, `src/components/`, `src/dto/` · a new Claude conversation.


### 1. Read the constraints as facts

Open `constraints/parallel-run.md`: the order-history suite, how it is split across four workers, and the database and API rules — which fields are unique, which are single-use, which values the database generates, which fixture is shared. These are not suggestions. They are the rules the collisions come from.

In `review-notes.md › Stage 4 › Before generating`, list every data item the suite needs and classify each one **isolated** or **shared** — one line of why per item. The question for each: *if two tests use this at the same time, can they interfere?*

### 2. Let Claude draft it — the naive way

New conversation. Paste `prompts/04-test-data.md` with the constraints file and the test list in the slots. The prompt asks for "realistic and consistent" data. Do not improve it. Send.

Save the exchange, untouched, as `claude-runs/04a-test-data-naive.md`.

### 3. Find the collisions

Lay the draft out worker by worker. For every shared item, find where two workers touch it. Each collision goes in `Stage 4 › Collisions found`:

*item · tests and workers · what fails at runtime (the error, and whether it's every run or only sometimes) · risk factor (duplicate values / shared state / timing · hidden dependency) · why the model produced it (no runtime state / no cross-call memory / no worker coordination)*

Expect at least two. If you found none, re-read the constraints: a value that is valid in isolation is exactly what a collision looks like.

### 4. Generate again — engineered

Now change the generation, not the wording. In the same or a new conversation, write a prompt that builds uniqueness into the scheme: a worker-index namespace on every account (`w1-`, `w2-`…), a per-run suffix on anything the database must not have seen, one dedicated user per worker for the empty-state test, the shared fixture used read-only, and the database-generated values left for the test to *read*, never pre-specified. Save it as `claude-runs/04b-test-data-engineered.md`.

In `Stage 4 › Scheme`, state the scheme in 3–5 lines: which technique closes which collision from step 3.

### 5. Deliver

`data/order-history.data.json` — the engineered dataset, checked by you against the constraints once more. Every collision from step 3 must be impossible in it, by construction. Statuses use the exact values of your Stage 3 `OrderStatus`; the tests named match your Stage 1 steps.

### 6. Control pass

`samples/checkout-parallel-scenario.md` is a different suite with four seeded collisions — one per risk factor. Find them; list them in `Stage 4 › Control pass` with the same five parts. No regeneration.

### 7. One sentence

`Stage 4 › Hardest call`: why does "make every value unique" in the prompt not fix this — and what does?

### Submit

Commit, push, submit the repo link. This is the last stage: the reviewer reads `data/`, both `claude-runs/04*` files, your Stage 4 notes — and checks that the repository holds together: names in the data match the steps, statuses match the DTO, every step's Action has a method.

### ✅ Before you submit

- `claude-runs/04a-…naive.md` and `04b-…engineered.md` — exact prompts, full responses, untouched
- Notes › Before generating — every data item classified isolated/shared with a reason
- Notes › Collisions found — 2+ collisions, each with item · workers · runtime failure · risk factor · mechanism
- Notes › Scheme — which technique closes which collision
- `data/order-history.data.json` — no shared mutable value across workers; no pre-specified DB-generated values; statuses from `OrderStatus`; test names from Stage 1
- Notes › Control pass — four collisions in the checkout sample, one per risk factor
- Notes › Hardest call — the structural gap, and the fix that doesn't depend on wording
