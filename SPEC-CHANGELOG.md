# Spec Changelog

**About:** The current spec version is always the one in `SPEC.md`. The spec follows [Semantic Versioning](https://semver.org/) (`MAJOR.MINOR.PATCH`). Major bumps for breaking changes, minor for backward-compatible additions, patch for clarifications. When releasing a new major version, preserve the previous spec in `specs/` (e.g. `specs/v1.md`).

---

## 2.0.0

- **Summary:** Simplified protocol. Renamed `items/` to `work/`, simplified backlog to a plain slug list, replaced item task tables with bullet lists, introduced HTML comment sections and code block metadata in task files, reduced task statuses to four, removed Owner/Blocks/Goal/Item-backlink fields.
- **Breaking changes:**
  - Directory renamed: `items/` → `work/`.
  - `backlog.md` format changed from structured markdown (headers, Type, Status, Item link, Description) to a bullet list of markdown links to each item's `index.md`.
  - Item `index.md` changed from metadata + task table to a bullet list of task links.
  - Task file format changed from plain markdown fields to HTML comment sections (`<!-- METADATA -->`, `<!-- DESCRIPTION -->`, `<!-- ACCEPTANCE CRITERIA -->`) with a fenced code block for metadata.
  - Type is now embedded in the item slug (e.g. `001-chore-project-foundation`) instead of a separate field.
  - Task statuses reduced: `todo → in-progress → ready-to-review → ready-to-test → done` replaced by `open → in-progress → done` (plus `block`).
  - Removed fields: Owner, Blocks, Item backlink, Goal.
  - Removed derived status logic from `backlog.md` — backlog is now a plain list with no status.
  - Archive simplified: `.archive/backlog.md` and `.archive/items/` replaced by `.archive/` containing item folders directly.

## 1.0.0

- **Summary:** Initial version. Item folders under `items/`, roadmap in `backlog.md`, task files with status and acceptance criteria, archive for completed work. Items support typed work: feature, bugfix, refactor, chore.
- **Changes:**
  - Directory layout: `backlog.md`, `items/<item-slug>/` with `index.md` and task files, `.archive/` for completed items and archived item folders.
  - Roadmap format: items with Type, Status, Item (link to folder), Description.
  - Item format: `items/<item-slug>/index.md` with Type, Status, Goal, and Tasks table.
  - Task format: `items/<item-slug>/<NNN>-<task-slug>.md` with Status, Priority, Owner, Item link, Description, Acceptance Criteria; optional Depends on / Blocks.
  - Type field: `feature`, `bugfix`, `refactor`, `chore` — extensible per project.
  - Derived status logic: all done → done; any in-progress/review/test → in-progress; mix of done and todo → in-progress; all todo → todo.
  - Archive: completed items appended to `.archive/backlog.md`; archived item folders moved to `.archive/items/<slug>/`.
  - Limits: max 10 open items; only `open` items accept new tasks.
  - Versioning: semver, protocol version stated at top of `SPEC.md` (**Version:** 1.0.0).
