# How this repo works

This repo builds **tinytodo** spec-first. Three roles:

- **The owner (human)** describes the product and tests what gets built.
- **The planner (Claude in conversation)** turns that into the spec in `PLAN.md`.
- **The build routine (a scheduled Claude run)** implements the spec, one chunk at a time.

## PLAN.md decides what gets built

`PLAN.md` is the single source of truth. Code is only written to satisfy a chunk in `PLAN.md`.
If something isn't in `PLAN.md`, don't build it. Its Ground rules (language, folders, test
command, "all tests must pass") apply to every change.

## When the owner is talking about the product (planning mode)

- **Only update `PLAN.md`.** Do not write or change product code or tests.
- New, well-defined work becomes a **new chunk**: give it the next number, add it to the
  Status board as `TODO` with its dependencies, and write it in the same shape as the others
  (`**Status:** TODO`, goal, `- [ ]` checklist, `**Acceptance:**` line).
- **Never edit chunks that are `DONE`.** If done behaviour needs to change, add a new chunk that changes it.
- Chunks that are `TODO` may be refined if the owner asks.
- Vague or undecided ideas go under **Open questions / ideas**, not into chunks.
- Commit `PLAN.md` changes with a message starting `plan:`.

## When running as the build routine (build mode)

Build **exactly one chunk per run**:

1. Read `PLAN.md`. Pick the lowest-numbered chunk with status `TODO` whose dependencies are all `DONE`.
   If an `IN PROGRESS` chunk exists from an earlier run, finish that one instead.
   If nothing is eligible, add a log entry saying so and stop.
2. Set that chunk to `IN PROGRESS` (in both the Status board and the chunk's `**Status:**` line).
3. Implement only that chunk's checklist, following the Ground rules. Tick each `- [ ]` → `- [x]` as it's done.
4. Run the exact test command from Ground rules. All tests must pass.
5. Finish by updating `PLAN.md` so it stays accurate:
   - All tests pass and the checklist is complete → status `DONE`.
   - Stuck (failing tests you can't fix, unclear spec, missing dependency) → status `BLOCKED`,
     with a one-line reason under the chunk's Status line. Don't guess at the spec.
   - Add one entry to the Progress log, between `<!-- LOG START -->` and `<!-- LOG END -->`,
     newest at the bottom: `- YYYY-MM-DD — Chunk N — DONE|BLOCKED — what was built, test result (e.g. 12 tests OK)`.
6. Don't rewrite other chunks. If you spot a new need, add it under Open questions / ideas.
7. Commit code and `PLAN.md` together with a message like `build: chunk N — <name>`, and push.

Status values are exactly: `TODO`, `IN PROGRESS`, `DONE`, `BLOCKED`.
