# Build sprint scraper

- **Status:** todo
- **Priority:** 002
- **Owner:** —
- **Feature:** [Markdown Parser](../../backlog.md#002---markdown-parser)

## Description

Using the sprint paths extracted from the backlog, read each sprint's `index.md` and task files. Parse sprint metadata (status, goal) and all task entries from the table. Accept a `limit` parameter to cap the number of sprints scraped.

## Acceptance Criteria

- [ ] Reads sprint `index.md` and extracts status, goal, and task table
- [ ] Parses individual task files for full task details
- [ ] `limit` parameter controls max number of sprints processed
- [ ] Defaults to all sprints when no limit is provided
- [ ] Returns structured sprint and task data as JSON
