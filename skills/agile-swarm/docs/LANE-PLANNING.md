# Lane planning

How to turn a sprint into independent, parallel-safe lanes.

## Inputs
Pull the current sprint's **non-done** items (exclude epics/containers) with: `key, summary, status, priority, story-points, parent, dependencies`. Cross-reference dependencies from the backlog mirror.

## Filter to "ready"
- **Dependency-ready**: all blockers are Done (or NO-GO). 
- **Watch for stale/legacy dependency keys** (e.g. pre-migration project keys). A "blocked" flag pointing at a legacy key is often noise — verify whether it maps to already-done work before excluding (or excluding it from the swarm) the item.
- Prefer **PLANNED / REFINED** items with clean acceptance criteria.

## Group into lanes (the key step)
**One lane = one distinct codebase / service.** Heuristics:
- Items touching the **same module** → same lane (one lead, sequential — they'd collide as separate lanes).
- A natural diverse lane: *one backend service · another backend service · the frontend · infra/docs*. Distinct file trees → safe parallel worktrees.
- Put the **root/unblocking** story first in a lane (e.g. the shell scaffold that unblocks the sibling layouts; the identity story that unblocks adoption).
- Keep **cross-service stories out of parallel lanes** — anything that edits many services at once (platform-wide spines, cross-cutting adoption) will collide with the per-service lanes. Run those solo, before or after.

## Order & size
- Order lanes and within-lane items by **priority** (Highest → High → Medium).
- Note the SP per lane; a lane is a multi-story sequence (branch + PR each).
- Park clean-but-lower-priority candidates on a **bench** for the next batch.

## Present for approval
Show a table: lane → codebase → ready items → SP → priority/dep notes, plus an explicit **independence/collision-safety** statement. Recommend a first parallel set. Get PO approval via a single multi-select before spawning — this is a scope decision the PO owns.

## Example shape
| Lane | Codebase (isolated) | Ready items | SP | Notes |
|---|---|---|---|---|
| B1 · <service-A> foundation | `svc-a/*` + launcher | A1→A2→A3 | ~15 | self-contained new service |
| B2 · <service-B> feature | `svc-b/*` | B1 (High) → B2 | ~13 | unblocks downstream |
| B3 · frontend core | `web/*` | root scaffold → siblings | ~18 | structure-only pass |
| B4 · infra quick-wins | infra repo · archetype · docs | highest-priority infra + small items | ~10 | mixed, one lead sequential |
