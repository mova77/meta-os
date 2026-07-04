---
type: system
tags: [os, system, process, scrum]
---
# Scrum Harness

The estate's agile process. The operative procedure lives in the [[skills/agile-process/SKILL|agile-process]] skill;
this node just anchors it into the OS and records the invariants.

## Source of truth

```
Jira (project <SPACE>) → scrum/<space>/backlog.json (mirror) → agile-process skill → OS invariants
```

On any conflict, **the higher source wins** — the tracker is authoritative. Platform code
PRs do **not** edit `scrum/**`. The instance's process-repo project node holds the details
(tracker site, space key, repo owner).

## What the harness covers (via [[skills/agile-process/SKILL|agile-process]])

- Backlog refinement + `sync.py` reconciliation (backlog.json ↔ Jira).
- Ceremonies: new/implement story, sprint open/close, daily retro/standup.
- Status transitions, one-story-one-branch, draft-PR discipline.
- New-service design swarm + 5-epic skeleton.

## For a full parallel sprint

Use [[systems/swarm-harness]] + the [[skills/agile-swarm/SKILL|agile-swarm]] skill — dependency-free items become
independent worktree-isolated lanes.

## Gotchas

- `backlog.json` never stores acceptance criteria → judge Definition-of-Ready from **Jira**,
  not the mirror.
- Reconcile treats only missing-either-side / status / estimate drift as real; label drift
  is noise.
