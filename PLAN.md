# tinytodo — PLAN

> The living spec and single source of truth. If it isn't in this file, it doesn't get built.

## Product vision

`tinytodo` is a tiny command-line to-do list for one person. You can add a task, list tasks,
mark one done, and delete one. Tasks are saved in a local JSON file so they survive between runs.
No accounts, no sync, no dependencies — just Python 3.

Example session:

```text
$ python3 -m tinytodo add "Buy milk"
Added 1: Buy milk
$ python3 -m tinytodo add "Call the dentist"
Added 2: Call the dentist
$ python3 -m tinytodo list
[ ] 1  Buy milk
[ ] 2  Call the dentist
$ python3 -m tinytodo done 1
Done 1: Buy milk
$ python3 -m tinytodo list
[x] 1  Buy milk
[ ] 2  Call the dentist
$ python3 -m tinytodo delete 2
Deleted 2: Call the dentist
$ python3 -m tinytodo done 99
Error: no task with id 99
```

## Ground rules (apply to every chunk)

- **Language:** Python 3 (3.8+), **standard library only**. No third-party packages, no `requirements.txt`.
- **Folders:**
  - `tinytodo/` — the package (`__init__.py`, `storage.py`, `tasks.py`, `cli.py`, `__main__.py`).
  - `tests/` — `unittest` tests, one file per module (`test_storage.py`, `test_tasks.py`, `test_cli.py`), plus `tests/__init__.py`.
- **Data file:** `tinytodo.json` in the current working directory, unless the `TINYTODO_FILE`
  environment variable gives another path. Format:
  `{"next_id": 3, "tasks": [{"id": 1, "title": "Buy milk", "done": false}]}`.
  Ids are positive integers and are never reused, even after a delete.
- **Tests never touch the real data file:** they use `tempfile` directories / `TINYTODO_FILE`.
- **Exact test command** (run from the repo root):

  ```bash
  python3 -m unittest discover -s tests -v
  ```

- **All tests must pass** before a chunk can be marked DONE. A chunk with failing tests stays IN PROGRESS or becomes BLOCKED.
- Keep it small: no features beyond what the current chunk asks for.

## Status board

| # | Chunk | Status | Depends on |
|---|-------|--------|------------|
| 1 | Storage: load and save the JSON file | TODO | — |
| 2 | Task operations: add, list, done, delete | TODO | 1 |
| 3 | Command-line interface | TODO | 2 |

## Chunks

### Chunk 1 — Storage: load and save the JSON file

**Status:** TODO

**Goal:** A small storage module that reads and writes the task file safely, so later chunks never deal with JSON directly.

- [ ] Create `tinytodo/__init__.py` and `tests/__init__.py`.
- [ ] In `tinytodo/storage.py`, add `data_path()` returning the path from `TINYTODO_FILE` or `./tinytodo.json`.
- [ ] Add `load(path=None)` that returns `{"next_id": 1, "tasks": []}` when the file does not exist.
- [ ] Add `save(data, path=None)` that writes pretty-printed JSON (indent 2) by writing a temp file and replacing the original, so a crash can't leave a half-written file.
- [ ] If the file exists but is not valid JSON, raise a clear `StorageError` (don't silently wipe the user's tasks).
- [ ] Write `tests/test_storage.py`: missing file, save-then-load round trip, `TINYTODO_FILE` override, corrupt file raises `StorageError`.

**Acceptance:** Running the test command shows all storage tests passing; and in a Python shell, `save({"next_id": 2, "tasks": [{"id": 1, "title": "x", "done": False}]})` creates a readable `tinytodo.json` that `load()` returns unchanged.

### Chunk 2 — Task operations: add, list, done, delete

**Status:** TODO

**Goal:** The four to-do operations as plain functions on top of storage, with no printing or argument parsing.

- [ ] In `tinytodo/tasks.py`, add `add_task(title)` → returns the new task; strips whitespace; rejects an empty title with `ValueError`; uses and increments `next_id`.
- [ ] Add `list_tasks()` → returns all tasks in id order.
- [ ] Add `mark_done(task_id)` → sets `done` to `true` and returns the task; marking an already-done task is fine (no error).
- [ ] Add `delete_task(task_id)` → removes the task and returns it; ids are not reused afterwards.
- [ ] Unknown ids raise `TaskNotFound` (a custom exception) in `mark_done` and `delete_task`.
- [ ] Every operation that changes data saves it via `storage.save`.
- [ ] Write `tests/test_tasks.py` covering each function, empty title, unknown id, and "delete then add gets a fresh id".

**Acceptance:** Test command passes; in a Python shell, `add_task("Buy milk")`, `mark_done(1)`, then `list_tasks()` shows the task with `done: True`, and `tinytodo.json` on disk matches.

### Chunk 3 — Command-line interface

**Status:** TODO

**Goal:** A `python3 -m tinytodo` command that matches the example session in the Product vision exactly.

- [ ] In `tinytodo/cli.py`, add `main(argv=None)` using `argparse` with subcommands `add <title...>`, `list`, `done <id>`, `delete <id>`; return an exit code.
- [ ] `add` joins all remaining words into the title, so quotes are optional.
- [ ] Output formats exactly as in the vision: `Added N: title`, `[ ] N  title` / `[x] N  title`, `Done N: title`, `Deleted N: title`.
- [ ] `list` with no tasks prints `No tasks yet.`
- [ ] Errors (unknown id, empty title, corrupt file) print `Error: ...` to stderr and exit with code 1; bad usage is argparse's normal exit code 2.
- [ ] Add `tinytodo/__main__.py` that calls `sys.exit(main())`.
- [ ] Write `tests/test_cli.py` that calls `main([...])` with captured stdout/stderr and a temp `TINYTODO_FILE`, covering every subcommand and the error cases.
- [ ] Add a short `README.md` with usage copied from the example session.

**Acceptance:** In an empty folder, typing the example session from the Product vision produces the same output line for line, and the test command passes.

## Open questions / ideas

- Should `list` hide done tasks by default (with `--all` to show them)?
- An `undo`/`reopen` command to un-mark a done task?
- Editing a task's title (`edit <id> <new title>`)?
- Due dates or priorities — probably out of scope for "tiny".
- Installing as a real `tinytodo` command (a `pyproject.toml` entry point) instead of `python3 -m tinytodo`.

## Progress log

<!-- LOG START -->
<!-- LOG END -->
