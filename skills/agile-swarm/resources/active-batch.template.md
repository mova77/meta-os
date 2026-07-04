# Active swarm batch — template

Persist this where it survives across sessions (a memory file / project notes). Tick items as PRs merge; when ALL lanes are green, assemble the next batch from the remaining ready items and rewrite this file.

---
**Active batch — <sprint>, opened <date>.** <N> independent lanes, each one lead in its own worktree (disjoint codebases → no collision). Tick on merge; roll to the next batch when all done.

### B1 · <title>  (`<codebase>`)
- [ ] <KEY> — <summary> (<priority>)
- [ ] <KEY> — <summary>

### B2 · <title>  (`<codebase>`)
- [ ] <KEY> — <summary>

### B3 · <title>  (`<codebase>`)
- [ ] <KEY> — <summary>

### Bench (ready, not in this batch)
- <KEY/lane> — <why deferred>
- <items blocked by stale/legacy dep keys — verify before pulling in>

**Don'ts:** cross-service stories (platform-wide spines, broad adoption) are NOT in parallel lanes — they collide with the per-service lanes; run solo.
---
