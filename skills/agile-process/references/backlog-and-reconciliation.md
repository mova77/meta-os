# Backlog model & reconciliation

> **Moved:** the per-space backlog mirrors now live in the dedicated **private repo
> `mova77/SCRUM`** (Jira-derived). Reconcile there with `python3 scripts/sync.py
> --space <key> --apply backlog`. Code PRs in **io.space no longer edit `scrum/**`** —
> they record state in Jira only. Paths below are relative to the SCRUM repo.

Source of truth and data shapes for the IOS backlog, plus the full `sync.py`
reconciliation procedure. The one-line invariants live always-on in
`.claude/CLAUDE.md`; this is the detail.

## Source of truth
- **Jira** project **IOS** is authoritative. Site `iospace.atlassian.net`,
  cloudId `9cc74dd5-1b4f-4ad3-acf2-a809863597db`, projectId `10299`.
- `scrum/<space>/backlog.json` is a **1:1 local mirror** of every IOS issue. Keep both in
  sync; on any conflict, **Jira wins**.
- Jira connector tools: `mcp__*Atlassian_Rovo__*` (the MCP server id varies per
  session — match on the suffix, e.g. `…getJiraIssue`, `…transitionJiraIssue`).
- Issue types: Epic · Story · Enabler · Spike · Feature · Bug · Requirement ·
  Documentation · Open Decision.

## System entry points (umbrella epics)
When researching/planning a system, **start from its umbrella epic** (the
program-level rollup linking that system's child epics via "Relates to"). Ask
**graphify** for the entry point first; fall back to the `system-entry-points.md`
roster in the `mova77/SCRUM` repo. Roster is reproducible via
`jql = project = IOS AND issuetype = Epic AND labels = umbrella`. A new service's
5-epic skeleton gets an `umbrella`-labelled epic.

| Sys | Umbrella | Sys | Umbrella | Sys | Umbrella |
|--|--|--|--|--|--|
| MPS | IOS-434 | SLR | IOS-439 | AGE | IOS-441 |
| SMS | IOS-432 | XTR | IOS-440 | ALR | IOS-524 |
| SFD | IOS-433 | INF | IOS-436 | SMC | IOS-536 |
| MSOT | IOS-435 | SPA | IOS-437 | TEN | IOS-548 |
| CLK | IOS-438 | TTC | IOS-602 | | |

## `backlog.json` shape
`{ project, sprints[], epics[], stories[] }`
- **`jiraId`** (e.g. `IOS-184`) is the **sole identifier**. No legacy `id` field.
  No `jiraUrl` — derive it as `…/browse/{jiraId}`.
- `epics[]`: `project, jiraId, title, status, labels` (+ `description`, `color`).
- `stories[]` (also Feature/Spike/Requirement/etc.): `project, jiraId, title,
  status, epic, labels` (+ `type` when not a plain Story, `storyPoints`, `sprint`,
  `usecase`, `acceptanceCriteria`, `dependencies`, `actors`, …).
- `epic` and `dependencies` reference other issues **by `jiraId`**.
- **Story points** → Jira field `customfield_10016`.
- **Sprint** → Jira field `customfield_10020` (multi-valued — see the
  sprint-on-transition rule in `ceremonies.md`).
- `dependencies` map to Jira **"Blocks"** links: blocker = `inwardIssue`, blocked =
  `outwardIssue`.

### Editing `backlog.json` — surgical only (never round-trip)
The file is **not** stored in plain `json.dumps(indent=2, ensure_ascii=False)` form,
so loading it and re-dumping (or `jq '…' > file`) silently rewrites *every* entry to
a different-but-equivalent serialization. A one-field change then shows up as a
~1000-line diff that conflicts with every concurrent branch (this actually happened —
a single IOS-666 status flip produced a 1020-line diff).
- **Small change (status, story points, a field):** edit the **exact line** — the
  Edit tool, or `sed -i '<line>s/"IN REVIEW"/"DONE"/' scrum/<space>/backlog.json`. Then
  **`git diff --stat scrum/<space>/backlog.json`** to confirm only the intended lines moved
  (a status flip ≈ 2 lines).
- **Programmatic / bulk change:** use **`scripts/sync.py`** — it is the only
  writer that emits the file's canonical format (`_write_backlog`: 2-space indent,
  `ensure_ascii=False`, trailing newline).
- **Never** hand-roll a `json.load`/`json.dump` script over the whole file for a
  small edit. If you genuinely need a large intentional rewrite, prefix the commit
  with `CLAUDE_BACKLOG_BULK=1`.
- Enforced at commit time by `backlog-json-guard.sh` (see `scripts-and-hooks.md`):
  it blocks an oversized or invalid-JSON staged `backlog.json` diff.

### Labels (free-form, advisory)
Labels mirror Jira's cleaned label set. The project key is present (conventionally
`labels[0]`); otherwise labels track Jira minus noise — drop sprint tags
(`sprint-6`), leaked workflow statuses (`refined`), and legacy per-issue/epic/
decision codes (`SMS-033`, `E-05`, `MSOT-E1`, `ADR-INF-01-OD1`, i.e. ALL-CAPS codes
containing a digit, allowlisting topical ones like `M2M`/`3D`). Role/topical tags
(`frontend-engineer`, `observability`, …) are kept. **Label differences vs Jira are
advisory only — never counted as drift.**

### Status vocabulary (canonical)
`TO DO`, `PLANNED`, `REFINED`, `IN PROGRESS`, `IN REVIEW`, `DONE`. Map Jira
`To Do/Planned/…/Done` to these.

Plus one **deliberate terminal status `NO GO`** (a real Jira workflow transition the
PO created): work/decision **killed** — abandoned on purpose with the lesson
recorded and accepted. It is *not* drift and must **not** be "fixed" to a canonical
status; treat it like `DONE` for exclusion from active scope but keep it for
traceability (e.g. IOS-164). `sync.py` / `validate-backlog.sh` accept `NO GO` as
valid, not flagged.

## Reconciliation — `scripts/sync.py`
Diffs backlog↔Jira (missing either side, status/point/label drift, **and dependency
"Blocks"-link drift**). **Read-only by default** — reports and exits ≠0 on drift;
run it before/after backlog edits. Treat only **missing-either-side, status, and
story-point** drift as real (labels are advisory).

`scripts/sync_check.py` is a thin alias forwarding to `sync.py` (no args = the
read-only check).

### Dependency-link reconciliation (IOS-685)
`sync.py` diffs the dependency graph: backlog `A.dependencies=[B]` ⇔ Jira "Blocks"
link `B blocks A` (inwardIssue=B, outwardIssue=A). It reports edges present on one
side only (real drift), flagging *dangling* edges whose endpoint doesn't exist
(legacy `IN-*`/`INF-*` codes in `dependencies[]`). Checked only when the Jira side
carried `issuelinks` (always true live; a lean offline export skips it). The MPS
sprint-planner reads backlog `dependencies[]`, so keep this graph reconciled or the
schedule is unreliable.

### Apply modes (writes are dry-run previews unless `--yes`)
- `--set-status KEY=STATUS …` / `--set-desc KEY=@file|text …` with
  `--to {jira,backlog,both}` — targeted, repeatable.
- `--apply backlog` mirrors Jira→backlog (status/SP + adds missing issues).
- `--apply jira` pushes backlog status→Jira for status-drifted issues; for deps,
  creates missing Jira "Blocks" links from backlog (verifying direction on the first
  create). `--apply backlog` mirrors Jira-only links into `dependencies[]`.
- **Jira-wins still governs**: prefer `--apply backlog`.

### Procedure
1. Export IOS issues — either set `JIRA_EMAIL`+`JIRA_API_TOKEN` (live mode), or
   build an offline export via the connector (no creds needed):
   - Call `searchJiraIssuesUsingJql` with `cloudId`,
     `jql = "project = IOS ORDER BY key ASC"`,
     `fields = ["status","issuetype","labels","parent","summary","customfield_10016"]`,
     `maxResults = 100`, `responseContentFormat = "markdown"`. ~5 pages (IOS ≈ 484
     issues); paginate by passing `.issues.pageInfo.endCursor` as `nextPageToken`
     until `hasNextPage=false`.
   - Each page exceeds the tool token cap and is **auto-saved to a file** (path in
     the error). Don't read it into context — reshape each saved page with `jq`
     straight to disk:
     ```
     jq -c '[.issues.nodes[] | {key, fields: {status:.fields.status.name,
       issuetype:.fields.issuetype.name, labels:.fields.labels,
       parent:.fields.parent.key, summary:.fields.summary,
       customfield_10016:.fields.customfield_10016}}]' "$PAGE" > /tmp/jira_pN.json
     ```
     (read `endCursor`/`hasNextPage` from the same file with `jq`).
   - Merge pages: `jq -s 'add' /tmp/jira_p*.json > /tmp/jira_export.json`. The count
     must equal the backlog item count.
2. `python3 scripts/sync.py --jira /tmp/jira_export.json` (or no `--jira` for
   live). Exit ≠0 means drift; reconcile by editing whichever side is wrong (Jira
   wins for state, backlog mirrors new issues), then re-run until clean. To
   auto-apply the backlog side: `sync.py --apply backlog` (preview) then `--yes`.

## Scrum document roles
| File | Role | Cadence |
|------|------|---------|
| `scrum/<space>/backlog.json` | 1:1 Jira mirror — source of truth | Every story change |
| `scrum/sprint-history.md` | Living sprint record — planning seed → close-out | Sprint open + close |
| `scrum/retros/` | Per-service retrospectives — highlights absorbed into sprint-history.md | Service milestones |
| `scrum/daily/` | Ephemeral refinement/planning/retro notes — absorbed at sprint close | As needed; never deleted |
| `scrum/archive/` | Superseded docs — traceability only | Never edited |
| `scripts/sync.py` | Jira↔backlog reconcile — read-only by default | Edit deliberately |
