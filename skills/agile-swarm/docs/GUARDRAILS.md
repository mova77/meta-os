# Guardrails — why each one exists

Hard-won from a real multi-agent run. Each rule traces to a concrete failure it prevents.

## Branch & build
- **One story → one branch → one PR.** A mixed branch can't be split after merge. An incidental fix discovered mid-flight gets its *own* branch/issue, not a piggyback (or explicit PO sign-off).
- **Gate runs `clean verify`, FOREGROUND, time-bounded.**
  - *`clean`*: git branch switches do **not** wipe per-module build output. Stale compiled classes from another branch get picked up and fail (or pass!) wrongly. Always build from source.
  - *Foreground + bounded*: in a connection-bound (remote-control) session, a backgrounded build whose completion notification is lost on disconnect leaves the session idle until a human returns. Run gates in the foreground with a timeout so each turn is self-contained; offload genuinely long jobs to CI.
- **Stay current with main**: pull fresh main per session and before each story; `git fetch` periodically; rebase in-flight branches when main moves — catch drift early, not at a red gate.
- **Worktree isolation per lead** keeps agents off the human's primary checkout (no IDE collisions) and off each other's files. **But auto-isolation can SILENTLY FAIL** — a lead may land on the primary checkout instead of a worktree (observed: 3 of 4 leads isolated, the 4th didn't, twice). Defenses:
  - Every lead's brief makes step 1 **verify isolation**: `git rev-parse --show-toplevel` + `git worktree list` — if it's the primary (or the human's) checkout, **STOP and report**, run no git command.
  - **Never `git add -A` / `git add .`** — stage explicit paths only. An un-isolated lead running `git add -A` on the primary sweeps sibling worktrees (as gitlinks) and tool dirs (e.g. `.diffblue/`) into the commit. **Gitignore the worktree root and tool dirs** as belt-and-suspenders.
  - Orchestrator: after spawning, **`git worktree list` to confirm each lead got one**; if a lane's isolation didn't take, **pre-create a manual worktree** and point that lead at it rather than respawning blindly.

## Compiler / toolchain traps (language-specific, generalize the habit)
- Don't re-pin a dependency the imported platform BOM already manages — a local override shadows it and drifts.
- Some flags are mutually exclusive (e.g. `--release` vs `--add-exports` on javac). Know the per-module wiring and centralize it once (e.g. a repo-root JVM config) rather than per-module gymnastics.
- A "warnings-as-errors" gate exposes latent rot (deprecations) only on a *clean* build — fix it as a separate tracked chore, don't bundle.

## Tracker hygiene
- **Pre-set the issue/backlog state inside the code PR**; never open a standalone PR just to flip status.
- **Reconcile is read-only by default**; the designated source of truth wins. Only missing-either-side / status / estimate drift is real — label drift is informational noise.
- **PO owns sprint-close and product-affecting calls.** The orchestrator proposes and briefs; it does not decide scope, merges, or closure.

## Review
- **Every PR gets an INDEPENDENT review before the PO merges.** The lane lead self-reviews before opening, but a separate reviewer agent (not the author) catches what the author missed. Wire it into the monitor: when a new lane PR appears with no review yet, spawn a one-shot `reviewer-<N>` that runs `gh pr diff N`, reviews for correctness + security (auth/crypto/input) + project conventions, and posts `gh pr review N` (--approve / --request-changes / --comment). The reviewer **never merges** — the PO owns the merge; the review is a signal.
- Don't double-review: skip a PR that already has a review from `reviewer-<N>`.

## Agent discipline
- Every lead brief carries the **10-minute no-progress rule**: if no verifiable progress for 10 min, STOP and report — never retry silently. The orchestrator does liveness checks on the same cadence.
- **Don't poll** after spawning — leads message back / complete automatically.
- Leads run on the capable model and consume the real budget — scale lane count to budget; don't fan out beyond what's affordable.

## Decision discipline
- Use a **single structured multi-select** to get PO scope approval before spawning.
- Reserve questions for genuinely PO-owned forks (scope, risky/irreversible mechanism choices). Otherwise pick the sensible default and proceed.
