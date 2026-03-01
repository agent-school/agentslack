# AGENTS.md — AgentSlack

Guidance for agentic coding agents working in this repository.

---

## Project Overview

AgentSlack is a multi-tenant B2B SaaS orchestration platform that coordinates specialized AI agents
to execute parallel, dependency-aware workflows. The backend is a **NestJS (TypeScript)** control
plane backed by **PostgreSQL** (via Prisma) and **Redis/BullMQ**, with **Discord** as the primary
IO surface. Frontend is intentionally deferred from MVP scope.

Architecture document: `_bmad-output/planning-artifacts/architecture.md`  
PRD: `_bmad-output/planning-artifacts/prd.md`

---

## Repository Layout

```
agentslack/
├── src/
│   ├── main.ts
│   ├── app.module.ts
│   ├── config/          # Runtime config, validated at boot
│   ├── common/          # Guards, interceptors, filters, errors, types
│   ├── modules/         # Feature modules (one folder per domain)
│   │   ├── auth/
│   │   ├── workspaces/
│   │   ├── discord-gateway/
│   │   ├── runs/
│   │   ├── tasks/
│   │   ├── orchestrator/
│   │   ├── queue/
│   │   ├── workers/
│   │   ├── events/
│   │   └── health/
│   └── infra/           # Prisma + Redis service adapters
├── prisma/
│   ├── schema.prisma
│   └── migrations/
├── test/
│   ├── e2e/
│   ├── integration/
│   └── fixtures/
├── _bmad-output/        # Planning + implementation artifacts (do not edit)
└── _bmad/               # BMAD agent config (do not edit)
```

---

## Build / Dev Commands

> The NestJS app is initialized with:
> `npx @nestjs/cli@latest new agentslack --package-manager pnpm`

| Purpose | Command |
|---------|---------|
| Install deps | `pnpm install` |
| Dev server (hot reload) | `pnpm run start:dev` |
| Worker process | `pnpm run start:worker` |
| Production build | `pnpm run build` |
| Type check only | `pnpm run build --noEmit` (or `tsc --noEmit`) |
| Lint | `pnpm run lint` |
| Lint + auto-fix | `pnpm run lint --fix` |
| Format | `pnpm run format` |

---

## Test Commands

| Purpose | Command |
|---------|---------|
| All unit tests | `pnpm run test` |
| Single test file | `pnpm run test -- --testPathPattern=<file>` |
| Single test by name | `pnpm run test -- -t "test name"` |
| Watch mode | `pnpm run test:watch` |
| Coverage | `pnpm run test:cov` |
| E2E tests | `pnpm run test:e2e` |
| Single e2e file | `pnpm run test:e2e -- --testPathPattern=<file>` |

Unit tests are co-located with source as `*.spec.ts`.  
E2E and integration tests live in `/test`.

---

## Database / Migrations

```bash
# Apply migrations (dev)
pnpm exec prisma migrate dev --name <description>

# Apply migrations (CI/prod)
pnpm exec prisma migrate deploy

# Regenerate Prisma client
pnpm exec prisma generate

# Open Prisma Studio
pnpm exec prisma studio
```

---

## Code Style Guidelines

### Language & Tooling

- **TypeScript** strict mode throughout; no `any` without explicit justification in a comment.
- **Node.js 24 LTS**, **pnpm 10.x** package manager.
- **NestJS 11.x** conventions — always use DI, modules, and lifecycle hooks idiomatically.
- Linting: ESLint with NestJS/TypeScript rules. Formatting: Prettier.

### Naming Conventions

| Context | Convention | Example |
|---------|------------|---------|
| Classes / types / interfaces | `PascalCase` | `RunOrchestratorService` |
| Files | `kebab-case` | `run-orchestrator.service.ts` |
| Functions / variables | `camelCase` | `getRunById` |
| Constants | `UPPER_SNAKE_CASE` | `MAX_RETRY_COUNT` |
| DB tables | `snake_case`, plural | `workflow_runs`, `task_events` |
| DB columns | `snake_case` | `workspace_id`, `created_at` |
| DB foreign keys | `<entity>_id` | `run_id`, `task_id` |
| DB enum values | `UPPER_SNAKE_CASE` in code | stored as lowercase strings in DB |
| API route params | `:camelCaseParam` | `/runs/:runId` |
| API query params | `camelCase` | `?workspaceId=...` |
| API JSON body/response fields | `camelCase` | `{ "workspaceId": "..." }` |
| Event type names | `domain.entity.action` | `run.task.created` |

### Imports

- Use absolute imports via TypeScript path aliases (configure in `tsconfig.json`).
- Group imports: NestJS/framework → third-party → internal modules → relative.
- No barrel `index.ts` re-exports unless the module has more than 5 exports.

### Formatting

- 2-space indentation, single quotes, trailing commas, 100-char line limit.
- Prettier enforces all of the above — do not override per-file.

### Types

- Prefer `interface` for object shapes, `type` for unions/intersections.
- All DTOs must use `class-validator` decorators; validate at controller boundary.
- Map DB `snake_case` fields to API `camelCase` in a dedicated mapper/service layer —
  never expose raw Prisma types directly in controllers.
- Dates: ISO-8601 UTC strings only (`2026-03-01T18:00:00Z`). Never pass `Date` objects
  across API or event boundaries.

### Error Handling

- Define typed domain errors extending a base `DomainError` in `src/common/errors/`.
- All unhandled exceptions must be caught by the global exception filter
  (`src/common/filters/global-exception.filter.ts`).
- Error response envelope: `{ "error": { "code": "STRING_CODE", "message": "...", "details": {} } }`.
- Always log machine-readable context with errors: `workspaceId`, `runId`, `taskId`, `eventId`.
- User-facing Discord error messages stay short; full diagnostics go to internal logs only.

### API Response Shape

```ts
// Success
{ "data": <payload>, "meta": { /* optional pagination/meta */ } }

// Error
{ "error": { "code": "RUN_NOT_FOUND", "message": "Run not found.", "details": {} } }
```

Never mix response shapes within an endpoint family.

---

## Architecture Constraints (Enforce Always)

1. **Orchestrator is the single write authority** for run/task state transitions. Workers and
   Discord gateway must never mutate run status directly — always go through
   `OrchestratorService`.

2. **Discord gateway handles Discord IO only.** It maps inbound messages to workspace context
   and delegates to orchestrator. It does not own any canonical state.

3. **Events are append-only.** The `EventLog` is a traceability record; never update or delete
   event rows.

4. **Secrets via environment variables only.** No secrets in prompts, event payloads, or logs.

5. **Redis is ephemeral.** Postgres is the source of truth. Redis/BullMQ is queue/cache only.

6. **Event envelope is required** for all emitted events:
   `eventId`, `ts`, `workspaceId`, `runId`, `taskId?`, `source`, `type`, `payload`.

---

## Module Boundaries

- `discord-gateway` — Discord IO only
- `orchestrator` — routing policy + state transitions (single authority)
- `runs` / `tasks` — domain CRUD and lifecycle reads
- `workers` — job execution and event emission
- `events` — append-only event log and publishing
- `common` — shared guards, interceptors, filters, errors, utils (no feature logic)
- `infra` — Prisma and Redis service adapters only

Do not add a generic `helpers/` or `utils/` dumping ground. Every shared utility needs clear
ownership in `src/common` or the relevant feature module.

---

## Testing Standards

- Unit tests co-located: `src/modules/runs/runs.service.spec.ts`
- Mock all external dependencies (Prisma, BullMQ, Discord) in unit tests.
- Integration/e2e tests in `/test`; use real DB via test Postgres instance.
- Aim for service-layer unit test coverage on all orchestrator state transitions.
- Test file naming: `<subject>.spec.ts` (unit), `<subject>.e2e-spec.ts` (e2e).
