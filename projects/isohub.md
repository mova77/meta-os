---
type: project
name: isohub
path: /Users/mova/code/isohub
repo: isohub-space/isohub
stack: [java, maven, quarkus, docker]
tags: [project, platform]
---
# isohub

**isohub.space — Space Open Platform.** Open-source, cloud-native SaaS for space
operations: orbital mechanics, spacecraft simulation, mission planning, telemetry, and
reference data.

- **Path:** `/Users/mova/code/isohub`
- **Stack:** Java · Maven (multi-module: `apps/`, `contracts/`, `bom/`) · Quarkus · Docker Compose
- **Entry points:** `README.md`, `Makefile`, `dev.sh`, `docker-compose.yml`, `apps/`, `contracts/`
- **Has its own:** `CLAUDE.md`, `SCRUM.md`, `.claude/`, `.claude-flow/`, `graphify-out/`, `.obsidian/`

## Governance
- Process: [[scrum]] harness — Jira project **IOS** → `backlog.json` mirror → [[skills/agile-process/SKILL|agile-process]].
- Infra: [[io.space.infra]] provisions its cloud + GitOps.
- Assets: [[isohub-assets]].

## Notes
- Run graph queries against `isohub/graphify-out/` via [[skills/graphify/SKILL|graphify]].
- Related links: [[memory/wiki/_index|wiki]] notes tagged `#project/isohub`.
