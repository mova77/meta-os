# Scripts & hooks catalog

Every script and hook that automates the IOS process, with what it enforces and how
to bypass it. Hooks are wired in `.claude/settings.json` → version-controlled
`.claude/hooks/*.sh` (manage with `/update-config` or `/hooks`). They make the gates
**deterministic** instead of relying on an agent to remember them.

## Scripts (`scripts/` + `.claude/workflows/`)

| Script | What it does | Mode |
|--------|--------------|------|
| `scripts/sync.py` | Jira↔backlog reconcile — status/SP/label + dependency "Blocks"-link drift. Full usage in `backlog-and-reconciliation.md`. | Read-only by default; writes behind `--apply`/`--set-*` + `--yes` |
| `scripts/sync_check.py` | Thin alias → `sync.py` (no args = read-only check). Called by the `sprint-reconcile` workflow. | Read-only |
| `scripts/prune_ghost_sprints.py` | Prune **ghost sprints** — empty sprints lingering in Jira's Agile API but invisible in the board UI (e.g. duplicate `IOS Sprint 8`/id 565), which pollute `futureSprints()`/`openSprints()` JQL. Skips ACTIVE and non-empty sprints. Same `JIRA_EMAIL`/`JIRA_API_TOKEN` auth as `sync.py`. | Read-only dry-run; deletes only with `--delete` (+ `--yes`) |
| `scripts/miro_sync.py` | One-way idempotent sync of `backlog.json` → a Miro planning board (Services × {INFRA/BACKEND/FRONTEND} matrix; cards recolour by rolled-up status). Backlog is source of truth; board is a regenerated view. | Write (to Miro only) |
| `.claude/workflows/sprint-reconcile.js` | Workflow wrapper: snapshot Jira (live creds or MCP export) → `sync_check.py --json` → structured drift report. Run at session start, sprint open/close, before/after backlog edits. | Read-only (reports; never writes) |

## Hooks (`.claude/hooks/`, wired in `settings.json`)

| Hook | Event | Enforces | Bypass |
|------|-------|----------|--------|
| `pre-push-verify.sh` | PreToolUse·Bash | Runs `mvn clean verify` before a `git push` **only when Java/POM files changed**; blocks a red build (exit 2). `clean` is deliberate — branch switches don't wipe `target/`, so stale cross-branch test classes would cause spurious failures. **Do not** add `-Djava.util.logging.manager` to this Maven JVM (breaks Quarkus augmentation; already set per test-fork in the root-pom surefire config). Skips docs/backlog-only pushes. | `CLAUDE_SKIP_VERIFY=1` |
| `validate-backlog.sh` | PostToolUse·Edit\|Write | On a `scrum/<space>/backlog.json` edit: hard-fails only on invalid JSON; flags duplicate/unknown `jiraId`, non-canonical status, and unresolved `IOS-` refs as advisories to reconcile via `sync.py`. Accepts `NO GO`. | — (advisory) |
| `backlog-json-guard.sh` | PreToolUse·Bash | Before a `git commit` that stages `scrum/<space>/backlog.json`: blocks (exit 2) if the staged file is **invalid JSON** or the staged diff exceeds ~80 changed lines — the signature of a **whole-file re-serialize** (`json.load`→`json.dump`/`jq`), which the file's non-canonical formatting turns into a 1000-line, conflict-prone diff. Forces surgical edits (Edit tool / `sed -i '<line>s/…/…/'`) or `sync.py` for bulk. Threshold via `CLAUDE_BACKLOG_DIFF_THRESHOLD`. | `CLAUDE_BACKLOG_BULK=1` |
| `po-only-sprint-close.sh` | PreToolUse·Bash | Blocks any command flipping a sprint to `CLOSED` — sprint closure is the PO's exclusive authority. | `PO_APPROVED_SPRINT_CLOSE` marker / `CLAUDE_PO_SPRINT_CLOSE=1` |
| `agent-long-task-timeout.sh` | PreToolUse·Bash | Blocks an unbounded long-runner (`mvn`/`gradle`/`npm`/`docker build`/`curl`…) or `while/until/--retry` poll loop unless bounded by a `timeout` wrapper or the Bash `timeout` field (300s default; `mvn verify` ≈10% over last build). Pairs with the soft 10-min no-progress STOP-and-report rule. | `NO_TIMEOUT_GUARD` marker / `CLAUDE_SKIP_TIMEOUT_GUARD=1` |
| `gh-pr-create-guard.sh` | PreToolUse·Bash | Blocks `gh pr create` unless it is **non-interactive** (inline `--body`/`--body-file`/`-F`/`--fill*`) **and bounded** (~15s `timeout`/Bash `timeout` field) — a bodiless `gh pr create` opens `$EDITOR` and hangs forever, stalling a swarm lead. | `NO_GH_PR_GUARD` marker / `CLAUDE_SKIP_GH_PR_GUARD=1` |
| `story-pickup-preflight.sh` | UserPromptSubmit | On a "pick up / tackle a story" prompt, injects a MANDATORY pre-flight: (1) ensure the **ruflo swarm** is ready + start the ruflo agents locally, (2) **sync graphify**, (3) run `scripts/sync.py` — before transitioning the story or spawning the swarm. | — |
| `provision-jdk.sh` | SessionStart | Auto-provisions **JDK 25** in the remote/web env (IOS-622) so the build toolchain matches. | — |
| `session-start.sh` | SessionStart | Prints a toolchain/readiness summary (JDK, Maven, Orekit data, build command, backlog source-of-truth) into context. | — |

> Two inline (non-file) PreToolUse hooks in `settings.json` also enforce the
> **graphify-first** rule: before `grep`/`find`/raw file reads when
> `graphify-out/graph.json` exists, run `graphify query` first.

> The Story→Epic progress cascade (when a Story goes `IN PROGRESS`, its parent Epic
> should too) is now an agent responsibility documented in `ceremonies.md` →
> *Status transitions*, not a hook. An earlier `epic-progress-cascade.sh` advisory
> hook was removed (it was never wired, and the agent — which holds the Jira MCP
> tools — keeps both sides in sync directly).
