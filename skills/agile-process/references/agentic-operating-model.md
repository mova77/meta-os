# Agentic operating model — Claude Code in the Scrum loop

How the ceremonies are *executed* with Claude Code. Sub-agents are spawned via the
`Agent` / Task tool (batch independent work in one message to run in parallel; long
jobs go to the background); skills are invoked as `/{skill}`; durable automation
lives in `.claude/settings.json` hooks (cataloged in `scripts-and-hooks.md`). For a
full parallel sprint run, use the **`agile-swarm`** skill (multi-lane worktree leads).

- **Refinement & spikes** — fan out research with the `Explore` and `general-purpose`
  agents; `/deep-research` for external sources, `/graphify` for codebase questions;
  resolve real unknowns with `AskUserQuestion`. A spike's output is a
  `docs/spikes/{SCOPE}-SPIKE-…` note, not code.
- **Architecture** — the **design swarm** (see `new-service.md`): perspective
  sub-agents → consensus → ADR + spike + swarm trace.
- **Sprint planning** — `AskUserQuestion` for scope/sequencing; pick stories with no
  open `dependencies`; record in `sprint-history.md` + Jira. **Pre-plan for
  parallelism**: prefer a *diverse basket* of **independent** stories across distinct
  projects / vertical slices (one infra · one backend service · one frontend/SPA)
  over a dependent chain inside one epic — independent slices let N swarms run in N
  worktrees with no cross-blocking or shared-`.git` collisions (the SMS-swarm +
  MPS-swarm parallel win). Lead proposes the basket; PO approves scope.
- **Implementation** — enter **plan mode** (or the `Plan` agent) for any non-trivial
  story, then `java-architect` (design) → `java-coder` (impl) → `java-tester`
  (JUnit5/Mockito, always after a class) → `java-reasoner` (debug failures);
  `java-microservices` for Quarkus/reactive/Kafka work. Keep `domain`/`api`
  framework-free.
- **Definition of Done (before PR)** — `mvn verify` green (the **Pre-push gate**);
  `/code-review`, plus `/security-review` for anything touching auth/crypto/external
  input; `/verify` or `/run` to confirm real behaviour; `/simplify` for cleanup.
  Then open the PR vs `main` (draft if a review is pending) and transition the story
  to **IN REVIEW**.
- **Review stewardship (IN REVIEW)** — after opening a PR, `subscribe_pr_activity` to
  watch CI + review comments and autofix/respond; drive Jira to **DONE** on merge and
  mirror into `backlog.json`.
- **Long-running work** — run builds/CI/load tests as background `Bash` tasks or a
  `Monitor`; never block on `sleep`. `/loop` for recurring polling. **Bound every long
  task** (the `agent-long-task-timeout` hook enforces it): wrap long-runners in
  `timeout` (300s default; `mvn verify` ≈10% over the last build time) or set the Bash
  `timeout` field. Every spawned agent's brief carries the **no-progress rule**: if it
  makes no verifiable progress for **10 min**, STOP and report — never retry silently;
  the Lead does liveness checks on the same cadence and brings down stalled (costly)
  idle agents. **On ANY timeout, surface it to the Lead and the PO** — escalate, don't
  retry.
- **Local-bounded sessions (`/remote-control`) → run long work FOREGROUND, never park
  a turn on a background task.** A local-bounded session only executes while connected.
  If you start a `run_in_background` task (e.g. `mvn verify`) and end the turn waiting
  for its completion notification, a dropped connection orphans the process, the
  notification is lost, and the session sits **idle until the human reconnects** (this
  cost ~2h once). So in such sessions: run gates/builds in the **foreground with a
  bounded Bash `timeout`** so each turn completes in-turn (a ~12-min foreground
  `mvn verify` is fine); do **not** end a turn waiting on a background task; offload
  genuinely long or connection-independent work to **CI** (push → let the pipeline
  run) or a remote/cloud agent. Background `Bash`/`Monitor` is for sessions that can
  actually receive the wake-up (persistent/cloud), not local-bounded ones. On
  reconnect after a stall, check for orphaned `mvn` and `mvn clean` if two builds may
  have shared `target/`.
- **Stay current with `main`.** At **every new session** and **every new story**,
  `git pull` a fresh `main` before branching, and `git fetch` **periodically** during
  long work so you notice mid-flight pushes from other sessions/agents early (parallel
  swarms + the GitLab mirror mean `main` moves under you). Branch a story off the
  *latest* `main`, and if `main` advances while a branch is in flight, rebase onto it
  rather than discovering the drift at a red gate.
