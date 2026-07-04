---
type: system
tags: [os, system, memory]
---
# Memory Layer

The knowledge discipline. Full folder detail in [[memory/_index]]; this is the *why* and
the *rules*.

## raw → wiki → output

- **raw/** — capture everything, structure nothing. Disposable. Never authoritative.
- **wiki/** — the durable, linked, indexed reference. Only **promoted** notes live here.
- **output/** — finished deliverables, each linking back to the wiki notes it drew from.

## Promotion is the whole game

An LLM-wiki is only as trustworthy as its promotion discipline. To promote a `raw/` note:

1. Clean it — remove scratch, keep the signal.
2. Add front-matter (`type:`, `tags:`) and a clear title.
3. Link it into the graph with `[[wikilinks]]` (to projects, skills, other wiki notes).
4. Move it to the right `wiki/` topic folder; update that folder's `_index.md`.

Run [[skills/graphify/SKILL|graphify]] over `wiki/` periodically to surface orphans and gaps — orphans are
usually un-promoted knowledge or missing links.

## Federation by symlink, not absorption

Project-local vaults keep existing and stay colocated with their code: the isohub docs
vault (`isohub/isohub` — ADRs, architecture, runbooks) and [[herakles]]-vault. They join
this OS's Obsidian graph through symlinks in [[vaults/_index|vaults/]] — one unified graph,
zero file moves, each vault still owned and committed by its own repo.

Federation is for **navigation**; promotion is for **canon**: cross-project-durable
knowledge still gets promoted into this `wiki/`. Single source of truth per fact.

## Relationship to agent memory

Runtime agent memory (ReasoningBank / AgentDB, the `.swarm/` stores) is *operational* — it
learns trajectories and patterns. This vault is *editorial* — human-curated knowledge.
Promote insights worth keeping from the former into the latter; don't conflate them.
