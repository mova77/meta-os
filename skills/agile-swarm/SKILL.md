---
name: "Multi-Lane Dev Pipeline"
description: "Plan and run a sprint as parallel multi-lane agent swarms: group dependency-free backlog items into independent vertical-slice lanes across distinct codebases, spawn worktree-isolated leads in one team, persist the batch across sessions, and enforce engineering-discipline gates (one-story-one-branch, foreground clean-verify, tracker↔backlog sync, review/security before PR, PO-owned merges and sprint-close). Use when orchestrating multiple stories in parallel, planning a sprint swarm, running multi-agent backlog execution, or whenever asked to 'spawn a swarm', run 'multi-lane' / 'parallel lanes', or 'burn' through a sprint."
---

# Agile Swarm Orchestration

Turn a sprint backlog into **N independent parallel lanes**, each a lead agent in its own git worktree, coordinated as one team — with the discipline gates that keep a fast multi-agent run from corrupting the build or the tracker. Distilled from a real platform run.

**Pipeline model.** Each lane is an independent *development pipeline*: a story flows through fixed stages — branch → implement → clean-verify → review → PR → merge — and N lanes run concurrently like the parallel pipelines of a superscalar RISC CPU. Disjoint codebases keep the lanes **hazard-free** (no shared-file 'data hazards'); cross-cutting work that would touch many lanes is the 'hazard' you serialize.

## What this skill does
1. **Plans lanes** — picks dependency-free, priority-ordered items and groups them into *independent vertical slices* (distinct codebases → worktree-safe, no collisions).
2. **Persists the batch** — writes the active lanes to a memory file so the run survives across sessions and rolls to the next batch when complete.
3. **Spawns leads** — one background lead per lane, all in one team (peer `SendMessage`), each in an isolated worktree.
4. **Enforces guardrails** — the non-negotiable gates (see `docs/GUARDRAILS.md`).
5. **Auto-reviews & stewards** — spawns an independent reviewer on each new PR, relays PRs, ticks the batch on merge, rolls the next set.

## The core principle
**Parallelism is safe only when lanes don't touch the same files.** Group by *distinct codebase/service* (one backend service · another service · the frontend · infra), never by dependent chains inside one module. N disjoint trees → N worktrees → zero `.git`/file collisions. If two candidate items share a module, they belong in the **same** lane (one lead, sequential), not two.

---

## Workflow

### 1 · Pre-flight (before transitioning or spawning anything)
- **Pull a fresh main** and `git fetch`; branch every lane off the *latest* main.
- **Reconcile tracker ↔ backlog** (read-only check; the source of truth wins). Treat only missing-either-side / status / estimate drift as real — label drift is noise.
- **Refresh the code graph / recon** so leads query current structure (note any stale areas and point leads at the live source there).
- **Ready the coordination layer** (swarm topology / shared memory), if used.

### 2 · Plan the lanes  → see `docs/LANE-PLANNING.md`
Pull the current sprint's non-done items with **status, priority, dependencies**. Keep only the **dependency-ready** ones (watch for stale/legacy dependency keys — verify before trusting "blocked"). Group ready items into lanes, one per distinct codebase, ordered by priority. **Present the proposal and get PO scope approval** (use a single multi-select question) — never auto-spawn.

### 3 · Persist the batch  → template in `resources/active-batch.template.md`
Write the approved lanes to a durable memory/notes file with per-item checkboxes. Tick items as PRs merge; when **all** lanes are green, assemble the **next** batch from the remaining ready items and rewrite the file.

### 4 · Spawn the leads  → brief template in `resources/lead-brief.template.md`
Spawn **all** leads in **one message** (→ one team; peers addressable by name), each `run_in_background` with **worktree isolation**. Transition each lane's lead story to In-Progress first. Every brief's **first step is to verify isolation** (`git rev-parse --show-toplevel` / `git worktree list`) and STOP if it's on a shared/primary checkout, and to **never `git add -A`** (explicit paths only). Brief also carries: scope (its codebase only), conventions, guardrails, comms, and the **10-minute no-progress STOP-and-report** rule. **After spawning, run `git worktree list` to confirm each lead actually got a worktree — auto-isolation can silently fail (a lead lands on the primary); if a lane's didn't, pre-create a manual worktree and point that lead at it.** Then **STOP and let them work — never poll**; they message back and complete automatically.

### 4.5 · Auto-review each new PR
Wire an independent reviewer into the monitor: when a new lane PR appears with no review yet, spawn a one-shot `reviewer-<N>` (NOT the author) that runs `gh pr diff N`, reviews for correctness + security + conventions, and posts `gh pr review N` (`--approve` / `--request-changes` / `--comment`). It **never merges** — the PO does. Skip PRs already reviewed.

### 5 · Steward
Relay each PR to the PO as it lands; tick the batch memory on merge; flip tracker states (In-Review on PR open, Done on merge) per the project's convention. When the batch finishes, roll to the next.

---

## Guardrails (the part that makes it safe)
Full list + rationale in **`docs/GUARDRAILS.md`**. The essentials:
- **One story → one branch → one PR.** Never bundle a second story (or an incidental fix found mid-flight) onto a lane's branch — file it and branch it separately.
- **Gate = `clean verify` (or equivalent), run FOREGROUND and time-bounded.** `clean` because branch switches don't wipe build output → stale cross-branch artifacts cause spurious failures. Never end a turn parked on a background build in a connection-bound session.
- **Pre-set the tracker/backlog state in the code PR** — don't open a standalone PR just to change issue state.
- **Independent review on every PR before the PO merges** — a `reviewer-<N>` agent (not the author) posts findings via `gh pr review`; security review for auth/crypto/external input. The PO owns the merge.
- **Verify each lead's worktree isolation** before it acts (auto-isolation can silently fail → lead on the primary); never `git add -A`; gitignore the worktree root + tool dirs.
- **PO owns merges and sprint-close.** The orchestrator proposes scope and briefs; it does not decide product-affecting calls.
- **Stay current with main** — rebase in-flight lanes when main advances; don't discover drift at a red gate.

## When NOT to use this
A single story, a 1–2 line fix, or dependent work that all lives in one module — just do it directly (one lead). The swarm pays off only with **3+ genuinely independent slices**.

## Cost note
N parallel leads each run full gate cycles — token- and time-heavy. Scale the lane count to the budget; offer to start with the top-priority 2 and add lanes once they're moving.

See also: `docs/GUARDRAILS.md` · `docs/LANE-PLANNING.md` · `resources/lead-brief.template.md` · `resources/active-batch.template.md`.
