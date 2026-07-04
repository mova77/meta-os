# Architecting & seeding a new service

The design swarm (for a new service or major capability) and the 5-epic skeleton
that every platform microservice is seeded with. Default to **design-only**; seed the
skeleton into Jira + `backlog.json` only when explicitly authorised.

## Architect a new service or major capability (design swarm)
Before seeding a skeleton, run a multi-agent design swarm to produce the
architecture, exploiting Claude Code sub-agents.

1. **Convene perspectives** — spawn specialised sub-agents in parallel (`Agent` /
   Task tool), each given the *same* ground truth (the real platform seams —
   MSOT · TTC · AGE · SMC · MPS · SLR · SFD · CLK · XTR — and the constraints:
   Quarkus 3 · Java 25 · hexagonal · Mutiny · Kafka · JSR-385 · multi-tenant +
   `BaselineId`) plus one distinct lens (e.g. domain/hexagon · real-time pipeline ·
   inference/forecast · agentic/consensus · integration & external ports). Ground
   the swarm in existing ADRs/spikes first — never invent services that exist.
2. **Exchange & converge** — R1: each agent emits an independent position paper
   (parallel). R2: distribute every agent's position to all the others with the
   identified conflicts + a proposed resolution each, and have each vote
   RATIFY/OBJECT and converge. R3: the orchestrator adjudicates the residuals. (If
   `SendMessage` is unavailable, re-spawn each agent with the others' positions
   injected — the cross-pollination, not session continuity, is what matters.)
3. **Record** three artifacts: `docs/adr/ADR-{SCOPE}-NN-…` (house style, decisions
   `D1..Dn`, Open Decisions → Jira); a buildable design
   `docs/spikes/{SCOPE}-SPIKE-S0-…` (modules, sealed domain model, pipeline, ports,
   Kafka event/topic map, the `{SCOPE}` 5-epic skeleton, SLOs, risks); and the
   consensus trace `docs/swarm/{SCOPE}-design-swarm-consensus.md`. Index the ADR in
   `docs/adr/README.md`.
4. **Default to design-only**; seed the skeleton only when explicitly authorised —
   then follow **New microservice** below.

## New microservice — the 5-epic skeleton
Prepopulate the project backlog (Jira + `backlog.json`) so every service aligns and
never drifts. For prefix `{PRJ}` (e.g. `PLD`):

- `{PRJ}-E1` **Platform Foundation** — scaffold the **four-module in-service hexagon**
  (`{prj}-domain`, `-api`, `-persistence`, `-rest`) plus the centralized
  composition-root deployable `launchers/{service}-svc` (per **ADR-INF-04**, <SPACE>-386
  — *no* in-service `*-launcher`; one `api` → many launchers → many topologies, also
  wired into `-Pmonolith`), ArchUnit layering rules (`DomainPurityTest` in `-domain`,
  `HexagonalArchitectureTest` in the launcher), initial container/deploy profile.
- `{PRJ}-E2` **Domain Model & Types** — pure immutable domain structures (data
  products, calibration, …); physical quantities via JSR-385 (`javax.measure`).
- `{PRJ}-E3` **Ingestion & API** — contract-first OpenAPI 3.1 schemas; search
  endpoints and incoming streaming/packet buffers.
- `{PRJ}-E4` **Persistence** — Hibernate Reactive entities indexing processed files
  / metadata catalogs for fast query.
- `{PRJ}-E5` **Processing Stream & Observability** — Mutiny pipelines
  (decompression/filtering/…) with Micrometer metrics named
  `{service}.{subsystem}.{metric}`.

Each new service's umbrella epic gets the `umbrella` label (so it shows up in the
system-entry-points roster).

## Mandatory tenancy-adoption story
Also create the standing story *Adopt multi-tenancy isolation + tenant-claim
propagation in {PRJ}* under epic **<SPACE>-42** ([INF] SaaS Multi-tenancy & Data Plane;
umbrella **<SPACE>-548**), blocked by **<SPACE>-365** (Postgres RLS) + **<SPACE>-366**
(Keycloak/X-Tenant-Id propagation). It is **not optional** (see the multi-tenancy
engineering convention in CLAUDE.md). Mirror the <SPACE>-403–407 / <SPACE>-542–547 pattern
(labels `INF, {PRJ}, multi-tenancy, identity, rls, security`).
