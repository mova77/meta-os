---
type: index
tags: [os, projects, registry]
---
# Project Registry

One node per repo the OS covers. A node **points at** its repo (path, purpose, stack,
entry points) — it never duplicates the code. This is the meta-OS's map of the estate.

| Project | Node | Purpose | Stack |
|---------|------|---------|-------|
| isohub | [[isohub]] | isohub.space — Space Open Platform (mission, telemetry, orbital) | Java · Maven · Quarkus |
| vectis | [[vectis]] | Reactive agile process-tracking system | (app) |
| tessera | [[tessera]] | Standalone OIDC / OAuth 2.0 authorization server | Java · Maven |
| io.space.infra | [[io.space.infra]] | Terraform + ArgoCD GitOps for isohub.space | Terraform · ArgoCD |
| isohub-assets | [[isohub-assets]] | Brand & platform assets for isohub | assets |
| Herakles | [[herakles]] | AAA agentic animation / research programme + vault | research |
| scrum | [[scrum]] | Process harness — backlog mirrors, agile skills (`mova77/SCRUM`) | process |

## Also on disk (not primary)

`io.space`, `io.space.platform`, `compass`, `claudio-realestateinvestment`, `poliba-tesi`,
`davinci`, `design`, `archives`. Add a node when one becomes actively governed by the OS.

## Adding a project

Copy [[templates/project|the project template]] to `projects/<name>.md`, fill it, and add a
row above. Keep the node in sync with reality — it's the OS's ground truth for "what is this
repo and how do I enter it."
