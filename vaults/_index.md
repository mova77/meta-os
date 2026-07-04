---
type: index
tags: [os, vaults, federation]
---
# Federated Vaults

Symlinks that pull the project vaults into this Obsidian graph **without moving their
files** — each stays owned by, and colocated with, its repo.

| Vault | Symlink target | Owner repo | Conventions |
|-------|----------------|------------|-------------|
| `vaults/isohub` | `../../isohub/isohub` | [[isohub]] | ADRs, architecture, runbooks, sprints — docs-as-code, 144+ notes |
| `vaults/herakles` | `../../Herakles/herakles-vault` | [[herakles]] | docs/lore/sources/spikes/systems, numbered prefixes |

## Rules

- **Edit federated notes in their own conventions**, not this OS's — they belong to their repo.
- Their content is committed in *their* repos; this repo only commits the symlinks.
- On a machine without those checkouts, the symlinks dangle — harmless; Obsidian just
  shows them empty.
- Cross-project-durable knowledge still gets **promoted** into [[memory/wiki/_index|memory/wiki/]]
  — federation is for navigation, promotion is for canon.
