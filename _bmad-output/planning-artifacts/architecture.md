---
stepsCompleted:
  - 1
  - 2
  - 3
  - 4
  - 5
  - 6
  - 7
  - 8
inputDocuments:
  - /home/alex/projects/agentslack/_bmad-output/planning-artifacts/prd.md
  - /home/alex/projects/agentslack/docs/TODO.md
workflowType: 'architecture'
lastStep: 8
status: 'complete'
completedAt: '2026-03-01T14:02:25-05:00'
project_name: 'agentslack'
user_name: 'AgentSlack Team'
date: '2026-03-01T13:52:45-05:00'
---

# Architecture Decision Document

_This document builds collaboratively through step-by-step discovery. Sections are appended as we work through each architectural decision together._

## Project Context Analysis

### Requirements Overview

**Functional Requirements:**
The PRD is treated as the authoritative product direction. Architecturally, the system must support goal intake, task decomposition, human-vs-agent routing, specialized role-constrained agents, dependency-aware parallel execution, and workflow status visibility. Recovery controls for drift/conflict and a minimal trigger/status integration surface are part of scope, with Discord positioned as an interaction channel rather than source of truth.

**Non-Functional Requirements:**
Architecture is driven by guardrail enforcement, workflow observability, auditable execution, and reliable branch progression with conflict handling. Performance should make parallel execution visibly responsive for users. Compliance requirements are minimal for MVP, but privacy hygiene (no sensitive production data in demo paths) remains expected.

**Scale & Complexity:**
Project complexity is medium, with primary risk in orchestration correctness rather than interface breadth. The architecture should optimize for deterministic behavior, clear boundaries, and low coordination overhead.

- Primary domain: AI workflow orchestration platform (full-stack, backend/control-plane heavy)
- Complexity level: medium (hackathon implementation scope)
- Estimated architectural components: 8-10 core components

### Technical Constraints & Dependencies

- PRD is authoritative for direction, even if some sections are still in progress.
- Architecture should preserve extensibility where PRD detail is still emerging.
- Hackathon execution requires strict scope discipline and a deterministic happy path.
- Discord integration is required for this implementation phase while keeping Discord as the IO surface, with canonical workflow state in internal persistence and event systems.
- Core reliability needs include stable task/run state transitions, dedupe/idempotency behavior, and clear retry semantics.
- Risky capabilities (e.g., computer-use actions) should remain isolated and permission-scoped.

### Cross-Cutting Concerns Identified

- Agent role boundaries and policy enforcement
- Context continuity across decomposition and handoffs
- Event traceability and run-level observability
- Conflict detection and branch reconciliation
- Idempotent updates and recovery workflows
- Human override/approval for ambiguous or high-risk actions
- Scope governance aligned to hackathon outcomes

## Starter Template Evaluation

### Primary Technology Domain

API/Backend orchestration platform with required Discord integration, based on project requirements analysis.

### Starter Options Considered

1. NestJS starter (CLI-generated)
- Strong fit for modular control-plane architecture
- Native structure for DI, modules, lifecycle, and testability
- Clean path to Discord gateway module, orchestration module, worker module

2. Next.js full-stack starter
- Great for UI-heavy products, but adds frontend surface area early
- For this hackathon, increases setup and decision overhead without improving core orchestration proof quickly

3. Ad hoc Express/Fastify skeleton
- Minimal initial setup, but missing guardrail-friendly conventions and structure
- Higher architecture drift risk as features are added quickly

### Selected Starter: NestJS (CLI Standard Starter)

**Rationale for Selection:**
NestJS provides the best speed-to-clarity tradeoff for this project's required control-plane behavior. It supports strict module boundaries (Gateway, Router/Orchestrator, Jobs, Workers, Tools), aligns with required Discord integration, and avoids premature frontend complexity for hackathon execution.

**Initialization Command:**

```bash
npx @nestjs/cli@latest new agentslack --package-manager pnpm
```

**Architectural Decisions Provided by Starter:**

**Language & Runtime:**
TypeScript-first Node.js backend foundation with Nest module structure and dependency injection patterns.

**Styling Solution:**
Not applicable for backend starter (UI intentionally deferred for hackathon scope).

**Build Tooling:**
Nest CLI build/start workflow with standard TypeScript compilation and project scripts.

**Testing Framework:**
Default unit and e2e test scaffolding included by Nest starter.

**Code Organization:**
Module-oriented structure suitable for:
- `discord-gateway` (Discord ingestion/replies)
- `orchestrator` (routing + task graph)
- `jobs` (queue producers/consumers)
- `workers` (agent execution adapters)
- `run-state` (persistent workflow/task state)

**Development Experience:**
Fast local setup, hot-reload dev mode, predictable architecture conventions, and maintainable expansion path under hackathon constraints.

**Version Check Snapshot (web-verified):**
- NestJS core: 11.1.14
- discord.js: 14.25.0
- BullMQ: 5.70.1
- Prisma: 7.4.2

**Note:** Project initialization using this command should be the first implementation story.

## Core Architectural Decisions

### Decision Priority Analysis

**Critical Decisions (Block Implementation):**
1. Discord-first ingress via single bot gateway (required by PRD update).
2. Single backend control plane using NestJS modules (no microservices split in hackathon).
3. Postgres as source of truth for workspaces/runs/tasks/events.
4. Redis + BullMQ for long-running and retryable agent jobs.
5. Canonical internal task state machine (Discord is IO, not state authority).

**Important Decisions (Shape Architecture):**
1. REST API for run/task operations and status retrieval.
2. Minimal auth model for hackathon (workspace token/JWT with scoped roles).
3. Structured event envelope persisted for traceability and replay-friendly debugging.
4. Worker isolation boundary for tool execution (OpenCode executor as separate worker path).

**Deferred Decisions (Post-MVP):**
1. Advanced RBAC granularity beyond Owner/Member/Viewer.
2. Full frontend application and design system.
3. Multi-region scaling, sharding, and high-availability tuning.
4. Rich Discord automations (beyond command intake + thread status loop).

### Data Architecture

- **Primary database:** PostgreSQL 18.x (latest patch from managed provider)
- **ORM/migrations:** Prisma 7.4.2 (`prisma` + `@prisma/client`)
- **Core entities:** Workspace, User, Conversation, Run, Task, TaskDependency, AgentAssignment, EventLog
- **Validation:** DTO validation at API boundary + schema-level constraints in DB
- **Caching/queue:** Redis 8.x + BullMQ 5.70.1
- **Migration strategy:** forward-only Prisma migrations; no destructive migration flow during hackathon

### Authentication & Security

- **Auth method:** lightweight JWT (`@nestjs/jwt` 11.0.2) with workspace-scoped claims
- **Authorization:** role checks at controller/service boundaries (Owner/Member/Viewer)
- **Discord security:** map Discord user/guild/channel to internal workspace context explicitly
- **API protection:** bearer token on non-public endpoints, request validation, basic rate limits
- **Secrets:** environment variables only; no secrets in prompts/events/log payloads

### API & Communication Patterns

- **API style:** REST (simple, fast to implement, easy demo tooling)
- **Primary endpoints:** create run, get run status, get task tree, post human decision, health
- **Error handling:** normalized error envelope with stable machine-readable codes
- **Rate limiting:** basic guardrails on ingress and command endpoints
- **Internal communication:** queue-driven job dispatch + persisted event logs
- **Discord integration model:** command intake + status replies + thread-per-run updates

### Frontend Architecture

- **Hackathon scope choice:** no dedicated frontend required
- **Primary interface:** Discord + minimal API inspection
- **Optional fallback UI:** deferred unless needed for demo polish
- **State management:** not applicable in MVP due to no full web app in initial scope

### Infrastructure & Deployment

- **Runtime:** Node.js 24 LTS
- **Package manager:** pnpm 10.x
- **Deployment:** Railway (single project with app + Postgres + Redis)
- **Process model:** one API/control-plane service + one worker process
- **Environment strategy:** `.env` per environment with strict variable naming
- **Observability:** structured JSON logs + run/task/event correlation IDs
- **CI/CD:** minimal pipeline (lint/test/build) plus deploy hook

### Decision Impact Analysis

**Implementation Sequence:**
1. Scaffold NestJS project and module boundaries.
2. Add DB schema + migrations for run/task/event state.
3. Add Discord gateway ingress and workspace mapping.
4. Add BullMQ worker pipeline and task lifecycle transitions.
5. Add API status surfaces and thread update loop.
6. Add guardrails (role enforcement, validation, retry/idempotency checks).

**Cross-Component Dependencies:**
- Discord gateway depends on workspace/run/task state APIs.
- Worker execution depends on queue + persisted task state transitions.
- Status visibility depends on event log consistency and correlation IDs.
- Guardrails affect all layers (ingress validation, orchestrator policy, worker permissions).

## Implementation Patterns & Consistency Rules

### Pattern Categories Defined

**Critical Conflict Points Identified:**
12 high-risk areas where AI agents could make incompatible choices if unspecified.

### Naming Patterns

**Database Naming Conventions:**
- Tables: `snake_case`, plural (`workspaces`, `workflow_runs`, `task_events`)
- Columns: `snake_case` (`workspace_id`, `created_at`, `updated_at`)
- Foreign keys: `<entity>_id` (`run_id`, `task_id`, `agent_id`)
- Enum values: `UPPER_SNAKE_CASE` in code, stored as lowercase strings where practical

**API Naming Conventions:**
- Endpoints: plural nouns, kebab-free paths (`/runs`, `/tasks`, `/workspaces/:workspaceId/runs`)
- Route params: `:camelCaseParam` in framework definitions, serialized as path segments
- Query params: `camelCase` (`status`, `workspaceId`, `includeEvents`)
- Headers: standard HTTP headers + `x-workspace-id` for internal multi-tenant routing

**Code Naming Conventions:**
- Classes/types: `PascalCase` (`RunOrchestratorService`)
- Files: `kebab-case` (`run-orchestrator.service.ts`)
- Functions/variables: `camelCase`
- Constants: `UPPER_SNAKE_CASE`

### Structure Patterns

**Project Organization:**
- Organize by feature module, not by technical layer only:
  - `discord-gateway/`
  - `orchestrator/`
  - `runs/`
  - `tasks/`
  - `workers/`
  - `events/`
- Keep DTOs, controllers, services, and tests inside each feature module.
- Co-locate tests as `*.spec.ts`; e2e stays in `/test`.

**File Structure Patterns:**
- Config in `src/config`
- Shared primitives in `src/common` (guards, interceptors, errors, utils)
- No generic `helpers` dumping ground; every shared utility needs clear ownership

### Format Patterns

**API Response Formats:**
- Success:
  - `{ "data": <payload>, "meta": { ...optional } }`
- Error:
  - `{ "error": { "code": "STRING_CODE", "message": "Human-readable", "details": {} } }`
- No mixed response shapes per endpoint family.

**Data Exchange Formats:**
- JSON field naming: `camelCase` at API boundary
- Internally map DB `snake_case` <-> API `camelCase` through service/mapper layer
- Dates: ISO-8601 UTC strings only (`2026-03-01T18:00:00Z`)
- Booleans: strict `true/false`

### Communication Patterns

**Event System Patterns:**
- Event naming: `domain.entity.action` (e.g., `run.task.created`, `run.status.updated`)
- Event envelope required:
  - `eventId`, `ts`, `workspaceId`, `runId`, `taskId?`, `source`, `type`, `payload`
- Event payloads are append-only compatible; avoid breaking field removals in MVP

**State Management Patterns:**
- Run states (example): `created -> planning -> running -> blocked -> completed|failed`
- Task states (example): `pending -> ready -> running -> retrying -> done|failed|cancelled`
- All transitions via orchestrator service (single write authority)

### Process Patterns

**Error Handling Patterns:**
- Use typed domain errors + global exception filter
- Always log machine-readable context (`workspaceId`, `runId`, `taskId`, `eventId`)
- User-facing Discord errors stay short; full diagnostics remain internal logs

**Loading/Progress Patterns:**
- Every long task emits progress events
- Discord updates are throttled and idempotent (edit existing status message/thread post where possible)
- No silent background work; each job must be traceable from run timeline

### Enforcement Guidelines

**All AI Agents MUST:**
- Follow naming conventions exactly (no ad hoc table/endpoint/file naming)
- Emit/consume the standard event envelope
- Use orchestrator-managed state transitions only (no direct status mutation bypass)

**Pattern Enforcement:**
- CI checks: lint + typecheck + test + schema drift check
- PR checklist includes "pattern compliance"
- Violations logged in `architecture.md` change notes and fixed before merge

### Pattern Examples

**Good Examples:**
- Endpoint: `POST /workspaces/:workspaceId/runs`
- Event type: `run.task.created`
- DB columns: `workspace_id`, `created_at`
- File: `discord-status-updater.worker.ts`

**Anti-Patterns:**
- Mixed naming (`user_id` in API JSON, `userId` elsewhere without mapping)
- Event names like `TaskCreated` in one module and `task.created` in another
- Direct worker writes to run status bypassing orchestrator
- Inconsistent error payloads by controller

## Project Structure & Boundaries

### Complete Project Directory Structure

```text
agentslack/
├── README.md
├── package.json
├── pnpm-lock.yaml
├── nest-cli.json
├── tsconfig.json
├── tsconfig.build.json
├── .env.example
├── .gitignore
├── docker-compose.yml
├── prisma/
│   ├── schema.prisma
│   └── migrations/
├── src/
│   ├── main.ts
│   ├── app.module.ts
│   ├── config/
│   │   ├── env.validation.ts
│   │   ├── app.config.ts
│   │   ├── discord.config.ts
│   │   ├── db.config.ts
│   │   └── queue.config.ts
│   ├── common/
│   │   ├── dto/
│   │   ├── errors/
│   │   ├── filters/
│   │   │   └── global-exception.filter.ts
│   │   ├── guards/
│   │   │   ├── jwt-auth.guard.ts
│   │   │   └── workspace.guard.ts
│   │   ├── interceptors/
│   │   │   └── request-context.interceptor.ts
│   │   ├── logging/
│   │   │   └── logger.service.ts
│   │   └── types/
│   ├── modules/
│   │   ├── auth/
│   │   │   ├── auth.module.ts
│   │   │   ├── auth.service.ts
│   │   │   ├── jwt.strategy.ts
│   │   │   └── dto/
│   │   ├── workspaces/
│   │   │   ├── workspaces.module.ts
│   │   │   ├── workspaces.service.ts
│   │   │   ├── workspaces.controller.ts
│   │   │   └── dto/
│   │   ├── discord-gateway/
│   │   │   ├── discord-gateway.module.ts
│   │   │   ├── discord-client.service.ts
│   │   │   ├── discord-command.handler.ts
│   │   │   ├── discord-thread-updater.service.ts
│   │   │   └── mappers/
│   │   ├── runs/
│   │   │   ├── runs.module.ts
│   │   │   ├── runs.controller.ts
│   │   │   ├── runs.service.ts
│   │   │   ├── run-state.machine.ts
│   │   │   └── dto/
│   │   ├── tasks/
│   │   │   ├── tasks.module.ts
│   │   │   ├── tasks.service.ts
│   │   │   ├── task-dependency.service.ts
│   │   │   └── dto/
│   │   ├── orchestrator/
│   │   │   ├── orchestrator.module.ts
│   │   │   ├── orchestrator.service.ts
│   │   │   ├── routing-policy.service.ts
│   │   │   └── transition-policy.service.ts
│   │   ├── queue/
│   │   │   ├── queue.module.ts
│   │   │   ├── queue.producer.ts
│   │   │   └── queue.constants.ts
│   │   ├── workers/
│   │   │   ├── workers.module.ts
│   │   │   ├── run-executor.worker.ts
│   │   │   ├── opencode-executor.worker.ts
│   │   │   └── retry-policy.ts
│   │   ├── events/
│   │   │   ├── events.module.ts
│   │   │   ├── event-log.service.ts
│   │   │   ├── event-envelope.ts
│   │   │   └── event-publisher.service.ts
│   │   └── health/
│   │       ├── health.module.ts
│   │       └── health.controller.ts
│   └── infra/
│       ├── prisma/
│       │   └── prisma.service.ts
│       └── redis/
│           └── redis.service.ts
├── test/
│   ├── e2e/
│   ├── integration/
│   └── fixtures/
└── .github/
    └── workflows/
        ├── ci.yml
        └── deploy.yml
```

### Architectural Boundaries

**API Boundaries:**
- Public MVP endpoints:
  - `POST /workspaces/:workspaceId/runs`
  - `GET /workspaces/:workspaceId/runs/:runId`
  - `GET /workspaces/:workspaceId/runs/:runId/tasks`
  - `POST /workspaces/:workspaceId/runs/:runId/decisions`
  - `GET /health`
- Discord ingress enters only through `discord-gateway` module, never directly mutating DB state.

**Component Boundaries:**
- `discord-gateway` handles Discord IO only.
- `orchestrator` is single authority for run/task transitions.
- `workers` execute jobs/tools and report events.
- `events` persists and publishes envelopes.
- `runs/tasks` own domain CRUD and lifecycle reads.

**Service Boundaries:**
- Controllers -> service layer -> infra adapters.
- No worker writes directly to run final state without orchestrator validation.
- Queue producer accepts only orchestrator-issued jobs.

**Data Boundaries:**
- Postgres is source of truth.
- Redis is ephemeral queue/cache only.
- Event log is append-only record for traceability and recovery.

### Requirements to Structure Mapping

**Feature Mapping:**
- Goal intake/task manager -> `modules/runs`, `modules/tasks`, `modules/orchestrator`
- Human vs agent routing -> `modules/orchestrator/routing-policy.service.ts`
- Specialized subagents -> `modules/workers`, `modules/orchestrator`
- Parallel execution -> `modules/queue`, `modules/workers`, `modules/tasks`
- Status visibility -> `modules/events`, `modules/discord-gateway`, `modules/runs`

**Cross-Cutting Concerns:**
- Guardrails -> `common/guards`, `modules/orchestrator/transition-policy.service.ts`
- Auditability -> `modules/events/event-log.service.ts`
- Auth -> `modules/auth`, `common/guards/jwt-auth.guard.ts`

### Integration Points

**Internal Communication:**
- Discord command -> orchestrator -> task creation -> queue enqueue -> worker execution -> event log -> Discord thread update.

**External Integrations:**
- Discord API via `discord.js`
- OpenCode execution through `opencode-executor.worker.ts`
- Railway managed Postgres/Redis

**Data Flow:**
1. Inbound Discord command mapped to workspace.
2. Run + tasks persisted in Postgres.
3. Runnable tasks queued in Redis/BullMQ.
4. Workers execute and emit structured events.
5. Orchestrator applies transitions.
6. Discord status thread updated from event stream.

### File Organization Patterns

**Configuration Files:**
- Runtime config under `src/config`, validated at boot.
- `.env.example` is canonical env contract.

**Source Organization:**
- Feature-first under `src/modules`.
- Shared framework concerns under `src/common`.
- Vendor/resource adapters under `src/infra`.

**Test Organization:**
- Unit tests co-located (`*.spec.ts`) + centralized integration/e2e under `/test`.

**Asset Organization:**
- Minimal for backend MVP; no heavy static asset strategy in phase 1.

### Development Workflow Integration

**Development Server Structure:**
- One NestJS app process for API + gateway.
- One worker process for queue consumers.

**Build Process Structure:**
- Standard Nest build compiles app and workers from shared codebase.

**Deployment Structure:**
- Railway deploys:
  - service A: app/api + Discord gateway
  - service B: worker
  - managed Postgres + Redis

## Architecture Validation Results

### Coherence Validation ✅

**Decision Compatibility:**
All selected technologies are compatible and reinforce the same architecture style: NestJS control plane, Discord ingress, Postgres system-of-record, and Redis/BullMQ async execution. No decision conflicts detected.

**Pattern Consistency:**
Naming, response formats, event envelope, state transitions, and error handling patterns are internally consistent and support multi-agent implementation without drift.

**Structure Alignment:**
Project structure directly reflects architectural boundaries and supports required integrations, especially Discord-first IO with internal orchestration authority.

### Requirements Coverage Validation ✅

**Epic/Feature Coverage:**
No formal epic/story file was provided, so validation is mapped against PRD feature categories and journey requirements.

**Functional Requirements Coverage:**
All major PRD functional areas are covered architecturally:
- Goal intake/task manager
- Goal decomposition and routing
- Specialized agent execution
- Dependency-aware parallel runs
- Status visibility and run traceability
- Discord integration path

**Non-Functional Requirements Coverage:**
NFRs are covered through:
- Guardrails and role boundaries
- Event logging and observability
- Retry/idempotency patterns
- Branch conflict/recovery strategy
- Scope controls for hackathon reliability

### Implementation Readiness Validation ✅

**Decision Completeness:**
Critical decisions are documented with validated version baselines and rationale.

**Structure Completeness:**
Directory tree and module boundaries are concrete and implementation-ready.

**Pattern Completeness:**
Conflict-prone areas are addressed with enforceable conventions and examples.

### Gap Analysis Results

**Critical Gaps:**
- None identified.

**Important Gaps (non-blocking):**
- OpenCode executor mode finalization (`GitHub Actions` vs `self-hosted`) left to implementation planning.
- Production-grade hardening (advanced RBAC, HA/scaling) intentionally deferred by design.

**Nice-to-Have Gaps:**
- Optional minimal web dashboard for non-Discord stakeholders.
- Additional automated architecture conformance checks in CI.

### Validation Issues Addressed

- Resolved PRD alignment concern by treating PRD as authoritative direction while allowing incomplete sections.
- Updated integration stance to required Discord support while preserving internal system-of-record principles.

### Architecture Completeness Checklist

**✅ Requirements Analysis**
- [x] Project context thoroughly analyzed
- [x] Scale and complexity assessed
- [x] Technical constraints identified
- [x] Cross-cutting concerns mapped

**✅ Architectural Decisions**
- [x] Critical decisions documented with versions
- [x] Technology stack fully specified
- [x] Integration patterns defined
- [x] Performance considerations addressed for MVP

**✅ Implementation Patterns**
- [x] Naming conventions established
- [x] Structure patterns defined
- [x] Communication patterns specified
- [x] Process patterns documented

**✅ Project Structure**
- [x] Complete directory structure defined
- [x] Component boundaries established
- [x] Integration points mapped
- [x] Requirements-to-structure mapping complete

### Architecture Readiness Assessment

**Overall Status:** READY FOR IMPLEMENTATION

**Confidence Level:** High (hackathon scope)

**Key Strengths:**
- Clear control-plane architecture with explicit boundaries
- Discord integration included without making Discord the state authority
- Strong consistency rules for multi-agent implementation
- Lean scope with deterministic implementation path

**Areas for Future Enhancement:**
- richer RBAC model
- enhanced observability dashboards
- stronger deployment resilience and scaling controls

### Implementation Handoff

**AI Agent Guidelines:**
- Follow architectural decisions exactly as documented
- Apply implementation patterns consistently
- Respect module boundaries and orchestrator authority
- Use the event envelope and state machine conventions for all workflow changes

**First Implementation Priority:**
Initialize the backend with:
`npx @nestjs/cli@latest new agentslack --package-manager pnpm`
then scaffold module boundaries from the approved structure.
