---
type: project
name: scrum
path: /Users/mova/code/scrum
repo: mova77/SCRUM
stack: [process]
tags: [project, process, scrum]
---
# scrum

**The process harness.** Private `mova77/SCRUM` repo holding the Jira-derived backlog
mirrors (`scrum/<space>/backlog.json`), sprint history, changelogs, and the process hooks.

- **Path:** `/Users/mova/code/scrum`
- **Holds:** `backlog.json` mirrors, `sync.py` (Jira reconciliation), sprint docs, and
  `.claude/hooks` (backlog-json-guard, po-only-sprint-close, validate-backlog).
- **Authority:** Jira (project **IOS**) → `backlog.json` → [[skills/agile-process/SKILL|agile-process]] → OS invariants.

## Governs
[[isohub]] primarily; the ceremonies/playbook apply across the estate.

## Notes
- **`scrum/.claude/skills` is retired (2026-07-04).** The library was moved to
  [[skills/_index|this OS's skills/]] (single copy) and is discovered machine-wide via
  `~/.claude/skills` symlinks. The IOS-specific `agile-process` variant was generalized in
  the move (`project <space>`).
- backlog.json never stores acceptance criteria — judge Definition-of-Ready from Jira, not
  the mirror.
- Wiki notes: `#project/process`.
