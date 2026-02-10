---
name: backlogmd
description: Define a new feature, break it into tasks, and add it to the backlog.
argument-hint: <feature description>
allowed-tools: Read, Write, Edit, Glob, Bash(mkdir *), WebFetch
---

# Backlog Feature Creator

You are an agent that helps the user define a new feature, break it into tasks, and write all the necessary files in the `.backlogmd/` system.

## Step 1: Read the protocol and current backlog

- Fetch the canonical protocol from `https://raw.githubusercontent.com/belugalab/backlogmd/main/.backlogmd/PROTOCOL.md` to understand all file formats, naming conventions, and rules. If the local `.backlogmd/PROTOCOL.md` exists, prefer the remote version as the source of truth.
- Check the protocol **Version** at the top of the document. This skill supports **Protocol v1** only. If the version is greater than 1, warn the user that the skill may be outdated and suggest they use an updated skill or follow the protocol manually.
- Read `.backlogmd/backlog.md` to determine the next available feature priority number (zero-padded to three digits, e.g. `002`).

## Step 2: Propose the feature and tasks

Based on `$ARGUMENTS`, propose:

1. **Feature name** — short, descriptive title
2. **One-line description** — what this feature delivers
3. **Tasks** — break the feature into concrete implementation tasks. For each task propose:
   - Task name
   - Short description (2-3 sentences)
   - Acceptance criteria (as checkbox items)

Present the full proposal to the user in a readable format and **ask for confirmation or edits** before proceeding. Do not write any files yet.

## Step 3: Feature placement

After the user confirms the feature and tasks:

1. Scan `.backlogmd/features/` for existing feature folders.
2. Read each feature's `index.md` and check its **Status** field.
3. Collect all features with status `open`.

Then:

- **If open features exist:** List them and ask the user which feature to add the tasks to, or whether to create a new feature folder.
- **If no open features exist:** Ask the user for a new feature name and a one-line feature goal.
- **If 10 open features already exist:** A new feature folder cannot be created. The user must archive an existing feature first, or add tasks to an existing open feature.

## Step 4: Write all files

Once the user has confirmed everything:

### 4a. Append feature to `backlog.md`

Add a new feature entry at the end of the `## Features` section following this exact format:

```
### <NNN> - <Feature Name>
- **Status:** todo
- **Feature:** [<feature name>](features/<feature-slug>/index.md)
- **Description:** <one-line summary>
```

### 4b. Create feature folder (if new feature)

If the user chose to create a new feature folder:

1. Create the directory `.backlogmd/features/<feature-slug>/`
2. Create `.backlogmd/features/<feature-slug>/index.md` with this format:

```
# Feature: <Feature Name>

- **Status:** open
- **Goal:** <one-line goal>

## Tasks

| # | Task | Status | Owner |
|---|------|--------|-------|
```

### 4c. Create task files

For each task, create `.backlogmd/features/<feature-slug>/<NNN>-<task-slug>.md`:

```
# <Task Name>

- **Status:** todo
- **Priority:** <NNN>
- **Owner:** —
- **Feature:** [<Feature Name>](../../backlog.md#NNN---feature-name-slug)

## Description

<detailed description>

## Acceptance Criteria

- [ ] <criterion>
```

- Task numbers are zero-padded to three digits and sequential within the feature (check existing tasks to find the next number).
- Task slugs are lowercase kebab-case derived from the task name.
- The Feature link anchor must be lowercase, with spaces replaced by `-` and the pattern `NNN---feature-name`.

### 4d. Update feature task table

Append a row for each new task to the `## Tasks` table in the feature's `index.md`:

```
| <NNN> | [<Task name>](<NNN>-<task-slug>.md) | todo | — |
```

## Rules

- This skill targets **Protocol v1**. Respect the versioning rules in `PROTOCOL.md`.
- Follow the formats in `PROTOCOL.md` exactly — no YAML frontmatter, pure markdown.
- All paths are relative within `.backlogmd/`.
- Never overwrite existing features or tasks — only append.
- Always confirm with the user before writing files.
- Max 10 open features in `features/`. If the limit is reached, the user must archive a feature or use an existing one.
- The `.archive/` directory is read-only cold storage. Never modify its contents, only move items into it.
