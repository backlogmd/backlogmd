# Protocol Changelog

**About:** The current protocol version is always the one in `PROTOCOL.md`. Breaking changes (e.g. directory renames, format changes) create a new major version: add an entry below, a migration section, and when releasing a new major version, preserve the previous protocol in `protocols/` (e.g. `protocols/v1.md`). Non-breaking clarifications or examples can be noted here without a new major version.

---

## Version 1

- **Summary:** Initial version. Feature folders under `features/`, roadmap in `backlog.md`, task files with status and acceptance criteria, archive for completed work.
- **Changes:**
  - Directory layout: `backlog.md`, `features/<feature-slug>/` with `index.md` and task files, `.archive/` for completed features and archived feature folders.
  - Roadmap format: features with Status, Feature (link to folder), Description.
  - Feature format: `features/<feature-slug>/index.md` with Status, Goal, and Tasks table.
  - Task format: `features/<feature-slug>/<NNN>-<task-slug>.md` with Status, Priority, Owner, Feature link, Description, Acceptance Criteria; optional Depends on / Blocks.
  - Archive: completed features appended to `.archive/backlog.md`; archived feature folders moved to `.archive/features/<slug>/`.
  - Limits: max 10 open features; only `open` features accept new tasks.
  - Versioning: protocol version stated at top of `PROTOCOL.md` (**Version:** 1).
