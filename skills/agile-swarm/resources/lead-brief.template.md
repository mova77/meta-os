# Lead brief template

Fill the `<…>` placeholders. Spawn each lead `run_in_background` with worktree isolation; spawn ALL leads in one message so they share one team.

---
You are lead **<lead-name>** in a <N>-lead swarm (peers: <other-lead-names>; orchestrator: "lead"). Lane **<lane title>**, sprint <sprint>. Repo `<repo path>` — you are in an **isolated git worktree; never touch the PO's primary checkout**. Branch every story off the **latest** origin/main.

SCOPE — your codebase only: `<paths the lead may edit>`. (If part of the work lives in another repo, name it and how to reach it.)

STORIES (own branch + PR each, in priority order):
1. **<KEY-1>** (<priority>) — <one-line goal + the acceptance criteria that matter>.
2. **<KEY-2>** — <…>.
   (Lead story already transitioned to In-Progress by the orchestrator; you transition the rest as you start them.)

CONVENTIONS (read the repo's CLAUDE.md + relevant ADRs first): <architecture pattern, logging convention, typed-quantities/units rule, multi-tenancy/security rule, the project's validation approach, etc.>.

GUARDRAILS:
- One story = one branch (`<prefix>/<KEY>-<slug>`) + one PR vs main. Do NOT bundle a second story or an incidental fix — file it and branch it separately.
- Before each push run the gate `<mvn clean verify / npm build+lint+typecheck>` in the **FOREGROUND with a bounded timeout** (set the Bash timeout field). Never end a turn parked on a background build.
- Run `/code-review` (and `/security-review` if it touches auth/crypto/external input) on the diff before opening the PR.
- Pre-set the tracker/backlog state in the code PR; transition the issue → In-Review on PR open.

COMMS: `SendMessage` progress + PR links to "lead" after each PR; coordinate with peers by name only if genuinely needed. **NO-PROGRESS RULE: if you make no verifiable progress for 10 minutes, STOP and report to lead — never retry silently.**

Start with <first story>.
---
