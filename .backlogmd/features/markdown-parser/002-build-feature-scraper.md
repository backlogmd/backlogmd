# Build feature scraper

- **Status:** todo
- **Priority:** 002
- **Owner:** —
- **Feature:** [Markdown Parser](../../backlog.md#002---markdown-parser)

## Description

Using the feature paths extracted from the backlog, read each feature's `index.md` and task files. Parse feature metadata (status, goal) and all task entries from the table. Accept a `limit` parameter to cap the number of features scraped.

## Acceptance Criteria

- [ ] Reads feature `index.md` and extracts status, goal, and task table
- [ ] Parses individual task files for full task details
- [ ] `limit` parameter controls max number of features processed
- [ ] Defaults to all features when no limit is provided
- [ ] Returns structured feature and task data as JSON
