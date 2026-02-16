# Specification

**Version:** 4.0.0

Single source of truth for the `.backlogmd/` system — markdown-based agile/kanban for agentic development. Agents must read this file before interacting with the backlog.

## Directory Structure

```
.backlogmd/
├── backlog.md
├── work/
│   ├── <item-id>-<slug>/
│   │   ├── index.md
│   │   ├── 001-task-slug.md
│   │   ├── 002-task-slug.md
│   │   └── ...
│   └── ...
└── .archive/
    └── <YYYY>/<MM>/<item-id>-<slug>/
```

All paths in this document and throughout the system are relative within `.backlogmd/`.

## IDs and Naming

- **Item IDs** (`item-id`): Zero-padded integers, minimum 3 digits (e.g., `001`, `012`, `999`, `1000`). Unique across the backlog.
- **Task IDs** (`tid`): Zero-padded integers, minimum 3 digits. Unique per item.
- **Slugs**: Lowercase kebab-case. An optional Conventional Commit type may follow the ID as the first slug segment (e.g., `001-chore-project-foundation`).
- **Priority** (`p`): Integer, unique per item. **Lower number = higher priority.**

## Backlog Format (`backlog.md`)

- Bullet list of markdown links to each item's `index.md`, sorted by `item-id` ascending.
- Format: `- [<item-id>-<slug>](work/<item-id>-<slug>/index.md)`
- Completed items are removed from `backlog.md` and their folder is moved to `.archive/`.

### Example

```md
- [001-chore-project-foundation](work/001-chore-project-foundation/index.md)
- [002-ci-initialize-github-actions](work/002-ci-initialize-github-actions/index.md)
```

## Item Format (`work/<item-id>-<slug>/index.md`)

- Bullet list of relative links to task files within the same directory.
- Sorted by task id (`tid`) ascending.
- Format: `- [<tid>-<task-slug>](<tid>-<task-slug>.md)`

### Example

```md
- [001-setup-repo](001-setup-repo.md)
- [002-init-ci](002-init-ci.md)
```

## Task Format (`work/<item-id>-<slug>/<tid>-<task-slug>.md`)

````md
<!-- METADATA -->

```yaml
t: Add login form
s: plan # plan | open | reserved | ip | review | block | done
p: 10 # priority within item (lower = higher priority)
dep: ["work/002-ci-initialize-github-actions/001-ci-cd-setup.md"] # optional: paths (relative to .backlogmd/) to tasks that must be done before this task can start
a: "" # assignee/agent id; empty string if unassigned
h: false # true if human review required before done
expiresAt: null # ISO 8601 timestamp for reservation expiry, or null
```

<!-- DESCRIPTION -->

## Description

<detailed description>

<!-- ACCEPTANCE -->

## Acceptance criteria

- [ ] <criterion>
````

### Dependencies

- **`dep`** lists paths to **other tasks** that must be **done** before this task can move to `ip`. Each entry is a path relative to `.backlogmd/`: `work/<item-id>-<slug>/<tid>-<task-slug>.md` (e.g. `work/002-ci-initialize-github-actions/001-ci-cd-setup.md`). Dependencies may be in the same item or in another item (cross-item). Agents must wait for each referenced work/task to be complete (`s: done`) before starting this task.

### Rules

- Filenames: `<tid>-<task-slug>.md`; `tid` zero-padded, unique per item.
- Status codes:
  - `plan` — groomed/draft, not ready for agents to pick up.
  - `open` — ready to be claimed by an agent.
  - `reserved` — claimed, awaiting start. Requires `a` (non-empty).
  - `ip` — in progress. Requires `a` (non-empty).
  - `review` — awaiting human approval. Set when `h: true` and work is complete. Requires `a`.
  - `block` — blocked by an external dependency. `a` is preserved.
  - `done` — complete. `a` is cleared.
- **`dep`**: See **Dependencies** above. No self-reference; no duplicates; no cycles (backlog must remain a DAG).
- Three HTML comment markers only: `<!-- METADATA -->`, `<!-- DESCRIPTION -->`, `<!-- ACCEPTANCE -->`.
- Acceptance criteria use markdown checkboxes.
- No YAML frontmatter outside the fenced code block; keep metadata lines ≤ 120 chars.

## Claim & Human-in-the-Loop Protocol

### Write ordering

File-based systems cannot provide true atomicity. To minimize inconsistency:

- Task files are the authoritative source for status, assignment, and content. Write the task file, then update `index.md` if the set of tasks or their order changed.

1. **Write the task file first**, then `index.md` (if affected).
2. If a write fails midway, the task file is the single source of truth. Re-read it and repair `index.md` if needed (regenerate links from task files on disk).

### Claiming a task

1. Agent reads `backlog.md` and task files to find tasks with `s: open`. To claim one, set `s: reserved`, `a: <agent-id>`, and optionally `expiresAt` (ISO 8601, short horizon — e.g., 30 min). Update the task file, then `index.md` only if needed.
2. Only `open` tasks may be claimed. `plan` tasks must first be promoted to `open` by a human or authorized agent.
3. `plan` cannot transition directly to `reserved`; it must go through `open` before claiming.

### Starting work

4. Set `s: ip` (keep `a`). A task cannot move to `ip` until **every** dependency in its `dep` list is done: each `dep` entry is a path `work/<item-id>-<slug>/<tid>-<task-slug>.md`; read that task file and require `s === "done"` for each. Agents block on the full work/task (each referenced task) being done before starting.

### Completing work

5. If `h: false`: set `s: done` and clear `a`.
6. If `h: true`: agent MUST set `s: review` and **stop**. Direct `ip → done` is **invalid** when `h: true`. Only a human (or authorized role) may move `review → done`.

### Releasing a claim

7. To unclaim, set `s: open` and clear `a` and `expiresAt`.

### Expiry

8. If `expiresAt` is non-null and in the past, another agent may reclaim by setting `s: reserved`, overwriting `a` with its own ID, and setting a fresh `expiresAt`.
9. `expiresAt` applies only in `reserved` and `ip`; ignore it while a task is in `review`.

### Blocking

10. Set `s: block` when an external dependency prevents progress. `a` remains set. Valid transitions out of `block`: back to `ip` (when unblocked) or to `open` (if releasing the claim).

## Archive

- Cold storage; agents skip `.archive/`.
- An item is archived **only when every task in the item has `s: done`**.
- Archive procedure (execute in order):
  1. Move the item folder to `.archive/<YYYY>/<MM>/<item-id>-<slug>/`.
  2. Remove the item's entry from `backlog.md`.

## Limits

- Max 50 open items (count entries in `backlog.md`).
- Recommended max 20 tasks per item. Items with more should be split into a new item.
- Archival steps must be executed in the prescribed order (folder move → backlog removal).

## Workflow Rules

### Status flow

```
plan ──→ open ──→ reserved ──→ ip ──→ done           (h: false)
                                  ──→ review ──→ done (h: true)

Any active state ──→ block ──→ ip or open
```

- `plan → open`: promotion (human or authorized agent only).
- `open → reserved`: claim (any agent).
- `reserved → ip`: start work (assigned agent).
- `ip → done`: completion (only valid when `h: false`).
- `ip → review → done`: completion with human gate (required when `h: true`).
- `block`: reachable from `reserved`, `ip`, or `review`. Returns to `ip` (unblocked) or `open` (released).
- A task cannot move to `ip` until every referenced task in `dep` (by path `work/.../<tid>-<task-slug>.md`) has `s: done`. Agents wait for each dependency (that work/task) to be fully done before starting.
- No circular dependencies. The set of all tasks and their `dep` paths must form a DAG. Agents adding `dep` entries must verify no cycle is introduced across the backlog.
- Task edits update the task file and, if needed, `index.md`.
- When all tasks in an item are `done`, archive the item.

## Reconciliation

Task files and `backlog.md` are the source of truth. There is no separate index file to reconcile with.

- **`index.md` link list**: Regenerate from the task files present on disk, sorted by `tid`.
- If `index.md` is missing links for existing tasks, regenerate it; if it has links to missing files, remove those links.
- If a task is `done`, `a` MUST be empty in the task file; clear it during any repair.

Agents should repair `index.md` on read if a mismatch is detected, and log a warning.

## Conventions

- All paths are relative within `.backlogmd/`.
- Priority numbers are unique per item (lower = higher priority).
- `index.md` must stay in sync with task files on disk.
- No YAML frontmatter outside the fenced code block.
- Keep metadata lines ≤ 120 chars.
- Avoid inline HTML outside the three comment markers.

## Versioning

- Semantic Versioning (`MAJOR.MINOR.PATCH`).
- This file is 4.0.0. Prior specs live in `specs/`.
- See `SPEC-CHANGELOG.md` for history and migrations.
- Agents may reject if the spec version in this file is unsupported.
