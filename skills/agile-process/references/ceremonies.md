# Ceremonies & story workflows

> **Backlog moved to `mova77/scrum`:** steps below that say "add to / edit / commit
> `backlog.json`" now happen in that private repo (a Jira-derived mirror), **not** folded
> into io.space code PRs. A code PR records state in **Jira only**; the mirror is reconciled
> separately with `sync.py --apply backlog`.

The step-by-step playbooks. Mandatory **invariants** (the must/never rules) are
summarised always-on in `.claude/CLAUDE.md`; the full procedure is here.

## Definition of Ready / Done
- **DoR** — clear acceptance criteria, value understood, dependencies identified,
  effort estimated (story points), no open blockers.
- **DoD** — all ACs met; code reviewed + merged; tests pass; build/deploy pipelines
  green; docs updated; no known defects; **backlog updated** (Jira + `backlog.json`).

## New story
Add to `backlog.json` (`epic`, `sprint`, `storyPoints`, `dependencies`, `status`) →
`createJiraIssue` (clean summary, acceptance criteria, `customfield_10016`) → record
the returned `jiraId` → for each dependency `createIssueLink` type **"Blocks"**.

## Implement a story
Pick one with **no open `dependencies`**, status `PLANNED`/`REFINED`, in the
current/next sprint → transition to **In Progress** **and set its Sprint to the
current active sprint** (mirror `sprint` in `backlog.json`) → branch
`{projectKey}/{issueKey}-{short-summary}` (e.g. `IOS/IOS-58-orekit`) → implement,
commit, push, open PR vs `main` → transition to **IN REVIEW**.

### Sprint-on-transition rule (mandatory)
A story's Sprint always reflects when it was actually worked:
- **→ In Progress:** set Sprint to the **current active sprint** — a story can never
  be In Progress with no/old sprint.
- **→ Done:** ensure the **current active sprint is also on** the story. The Jira
  Sprint field (`customfield_10020`) is **multi-valued** — *add* the current sprint,
  do **not** replace the In-Progress one, so a story that spanned sprints shows both
  the **first** (started) and **last** (finished) sprint. Same-sprint start/finish →
  the field just holds that one.

Set `customfield_10020` (active sprint's numeric id(s)) and mirror `sprint` in
`backlog.json` in the **same move** as each status transition.

### One story → one branch (mandatory, all work — not just infra)
A branch carries exactly **one** story's scope. Do **not** bundle a second story's
work — or an incidental fix/enabler/refactor discovered mid-flight — onto an
in-flight feature branch: give it its own `{projectKey}/{issueKey}-…` branch, or get
explicit PO OK before piggybacking. Once a mixed branch is merged it can't be split
cleanly. When a new concern surfaces mid-implementation, **file it** (New story) and
branch it separately.

### Status transitions
`getTransitionsForJiraIssue` → `transitionJiraIssue`. Drive Jira to `IN REVIEW` on
PR open, `DONE` on merge; mirror into `backlog.json`. When a Story goes
`IN PROGRESS`, its parent Epic should be active too — cascade the Epic to
`IN PROGRESS` in both Jira and `backlog.json`.

### Draft a PR only while a review agent is reviewing it (mandatory)
The draft flag exists to stop a PR being merged **before a background review *agent*
has finished** — it's a hold for *automated* review, so the PO doesn't merge 
mid-review. It is **not** an always-on default.
- **A review agent is (or will be) reviewing this PR** → open it as a **DRAFT**
  (`gh pr create --draft`); a draft can't be merged. Mark ready (`gh pr ready <n>`)
  when the agent's review lands green. (#277 merged before `reviewer-277` reported and
  shipped 2 must-fix defects to main — drafting prevents exactly that.)
- **The PO is the sole reviewer, no review agent running** → **open it ready
  (non-draft)**; there's nothing to wait on — the PO reviews and merges at will, so a
  draft would just be friction.
- Merging is the PO's decision — don't self-merge a PR unless explicitly told to.

Either way, `gh pr create` must be **non-interactive + bounded** (inline
`--body`/`--fill` + a ~15s timeout) — enforced by `gh-pr-create-guard.sh`.

## Open a sprint
Append a planning stub for `IOS-S{n}` to `scrum/sprint-history.md` (copy the S8
template: header TBDs, goal, candidate stories table, don'ts, close-out checklist) →
transition Jira sprint to **Active** → update `backlog.json` sprint status to
`IN PROGRESS`. Pre-plan a **diverse basket of independent stories** across distinct
projects/vertical slices (one infra · one backend · one frontend) so N swarms run in
N worktrees with no cross-blocking. Lead proposes the basket; **PO approves scope**.

## Close a sprint  *(PO-only — `po-only-sprint-close.sh` blocks otherwise)*
Fill in the `sprint-history.md` stub for `IOS-S{n}` — velocity, SP/day, increment
narrative, retro highlights absorbed from `scrum/retros/` — then tick the close-out
checklist (updates Performance Analysis table + Cumulative Platform Progress).
Append the `IOS-S{n+1}` planning stub immediately so the doc is always one sprint
ahead. Sprint **closure** is the Product Owner's exclusive authority (bypass the
hook only with explicit PO approval: `PO_APPROVED_SPRINT_CLOSE` /
`CLAUDE_PO_SPRINT_CLOSE=1`).

## Daily retro / standup (rolling cadence)
The daily notes in `scrum/daily/` are a **rolling log** — each retro covers *"since
the last retro through now"*, not a fixed calendar window, so it never overlaps or
leaves a gap. Don't rediscover this each day:
- **Name** the file `scrum/daily/YYYY-MM-DD-{retro,standup}.md` by the **date it's
  written**. A retro written today about yesterday's work is dated *today*; if the
  day's earlier note was only a morning snapshot, the next note picks up where it
  stopped (state the explicit window in the header, e.g. "since the 06-20 ~07:30Z
  retro through ~01:26 local 06-21") rather than editing the old one — daily notes
  are **never deleted/clobbered**.
- **Reconstruct from ground truth**, not memory: `git log` across **all active
  repos** (`io.space`, `io.space.infra`, `tessera`) for the window
  (`--since`/the prior retro's commit), the merged-PR list, and the matching
  standup/plan note — then map commits → `IOS-` keys.
- **Sections** (a retro carries all; a standup is the forward-looking subset):
  **What landed** (merged to `main`; table keyed by `IOS-NN` + PR + state) ·
  **Backlog & refinement** (new/refined/split issues, dep-graph reconciles) ·
  **Process** (rules banked into CLAUDE.md/this skill) · **Impediments & incidents**
  (what blocked/broke, with root cause) · **Discoveries** (insights, decisions
  surfaced) · **Retro — keep/change** (went-well / banked-lessons / **watch items
  carried** forward). A *standup* instead leads with *Since last note / Today's
  basket / lanes*.
- **Cadence vs sprint docs**: daily notes are ephemeral and **absorbed into
  `sprint-history.md` at sprint close** (service-milestone highlights into
  `scrum/retros/`). A docs-only daily note may go straight to `main` (or ride the
  active story branch) — don't open a dedicated PR for it. Carry unresolved **watch
  items** into the next note until they close.

## Commit rule
Any backlog change syncs Jira **and** `backlog.json`. **Don't open a dedicated PR
just to change issue state** — embed the `backlog.json` status update in the related
code PR (a feature PR may also carry catch-up `DONE` flips for already-merged
issues). Standalone backlog *additions* (new issues, refinements) may go directly to
`main`. Code changes go via the story branch + PR.
