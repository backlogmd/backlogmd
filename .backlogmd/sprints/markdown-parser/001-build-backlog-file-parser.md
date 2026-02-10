# Build backlog file parser

- **Status:** todo
- **Priority:** 001
- **Owner:** —
- **Feature:** [Markdown Parser](../../backlog.md#002---markdown-parser)

## Description

Create a parser that reads `backlog.md` and extracts all feature entries with their metadata (status, sprint link, description). Return structured JSON data for each feature.

## Acceptance Criteria

- [ ] Reads and parses `backlog.md`
- [ ] Extracts feature name, status, sprint path, and description for each entry
- [ ] Returns structured data (array/list of feature objects)
- [ ] Handles empty backlog gracefully
