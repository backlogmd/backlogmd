# Protocol

This document is the single source of truth for the `.backlogmd/` system — a markdown-based agile/kanban workflow for agentic development. Agents must read this file before interacting with the backlog.

## Directory Structure

```
.backlogmd/
├── PROTOCOL.md              # This file — rules, formats, conventions
├── backlog.md               # Active features (todo + in-progress), ordered by priority
├── sprints/                 # Max 10 open sprints
│   ├── <sprint-slug>/
│   │   ├── index.md         # Sprint metadata + task summary table
│   │   ├── 001-task-slug.md
│   │   ├── 002-task-slug.md
│   │   └── ...
│   └── ...
└── .archive/                # Cold storage — read-only, skipped by parsers
    ├── backlog.md            # Completed features
    └── sprints/              # Archived sprint folders (moved intact)
        └── <sprint-slug>/
```

All paths in this document and throughout the system are relative within `.backlogmd/`.

## Roadmap Format

File: `backlog.md`

```md
# Roadmap

## Features

### <NNN> - <Feature Name>
- **Status:** <status>
- **Sprint:** [<sprint name>](sprints/<sprint-slug>/index.md)
- **Description:** <one-line summary>
```

- `backlog.md` only contains active features (`todo` and `in-progress`). Completed features are moved to `.archive/backlog.md`.
- Features are ordered by priority number (`001` = highest).
- Priority numbers are zero-padded to three digits and unique within the roadmap.
- Each feature links to its sprint folder. Features without a sprint use `—` as the value.
- Valid feature statuses: `todo`, `in-progress`, `done`.
- A feature's status is derived from its tasks:
  - All tasks `done` → feature is `done`
  - Any task `in-progress`, `ready-to-review`, or `ready-to-test` → feature is `in-progress`
  - Otherwise → `todo`

## Sprint Format

File: `sprints/<sprint-slug>/index.md`

```md
# Sprint: <Sprint Name>

- **Status:** <status>
- **Goal:** <one-line goal>

## Tasks

| # | Task | Status | Owner | Depends on |
|---|------|--------|-------|------------|
| 001 | [Task name](001-task-slug.md) | <status> | <owner> | — |
```

- Sprint slug is lowercase kebab-case.
- Valid sprint statuses: `open`, `archived`.
- Only `open` sprints accept new tasks.
- The task table must stay in sync with the individual task files.

## Task Format

File: `sprints/<sprint-slug>/<NNN>-<task-slug>.md`

```md
# <Task Name>

- **Status:** <status>
- **Priority:** <NNN>
- **Owner:** <owner>
- **Feature:** [<Feature Name>](../../backlog.md#NNN---feature-name-slug)
- **Depends on:** <linked task list or `—`>
- **Blocks:** <linked task list or `—`>

## Description

<detailed description>

## Acceptance Criteria

- [ ] <criterion>
```

- Task filenames use the pattern `<NNN>-<slug>.md` where `NNN` is a zero-padded priority number.
- Priority numbers are unique within a sprint.
- Valid task statuses, in order: `todo` → `in-progress` → `ready-to-review` → `ready-to-test` → `done`.
- Owner is a handle (e.g. `@claude`) or `—` when unassigned.
- The Feature field links back to the parent feature in `backlog.md`.
- Acceptance criteria use markdown checkboxes.
- `Depends on` and `Blocks` are optional. Omit them from task files and the sprint table column when the project doesn't need dependency tracking. When used, `Depends on` lists tasks that must be `done` before this task can start (markdown links: `[001 - Task name](001-task-slug.md)`). `Blocks` is the inverse. Use `—` for tasks with no dependencies when the column/field is present.

## Archive

Directory: `.archive/`

- The archive is **cold storage** — agents and parsers must skip `.archive/` during normal operations.
- `.archive/backlog.md` uses the same format as `backlog.md`. Completed features are appended to it preserving their original priority number.
- `.archive/sprints/` contains archived sprint folders, moved intact from `sprints/`.
- Archive contents are **read-only**. Agents never modify archived items, only move things into the archive.

## Limits

- **Max 10 open sprints** in `sprints/` at any time. A new sprint cannot be created if 10 open sprints already exist.
- When a sprint is archived, its entire folder is moved from `sprints/<slug>/` to `.archive/sprints/<slug>/`.
- When a feature reaches `done` status, it is removed from `backlog.md` and appended to `.archive/backlog.md`.
- Both archival actions (sprint move + feature cleanup) happen together when a sprint is archived.

## Conventions

- All paths are relative within `.backlogmd/`.
- Priority numbers are unique per scope (features in roadmap, tasks within a sprint).
- Owner is optional; use `—` when unassigned.
- Sprint `index.md` task table must stay in sync with individual task files.
- Agents must read `PROTOCOL.md` before interacting with the backlog.
- No YAML frontmatter — the format is pure markdown throughout.

## Workflow Rules

1. Tasks move forward through statuses, never backward.
2. Only `open` sprints accept new tasks.
3. A feature's status is derived from its tasks (see Roadmap Format above).
4. When all tasks in a sprint are `done`, the sprint may be archived. Archiving moves the sprint folder to `.archive/sprints/` and moves any newly `done` features to `.archive/backlog.md`.
5. A task cannot move to `in-progress` unless all of its dependencies are `done`.
6. Circular dependencies are not allowed.
