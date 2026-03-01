---
stepsCompleted:
  - step-01-validate-prerequisites
  - step-02-design-epics
  - step-03-create-stories
  - step-04-final-validation
inputDocuments:
  - /home/alex/projects/agentslack/_bmad-output/planning-artifacts/prd.md
  - /home/alex/projects/agentslack/_bmad-output/planning-artifacts/architecture.md
---

# agentslack - Epic Breakdown

## Overview

This document provides the complete epic and story breakdown for agentslack, decomposing the requirements from the PRD, UX Design if it exists, and Architecture requirements into implementable stories.

## Requirements Inventory

### Functional Requirements

FR1: SWE can create a new goal with task description and context.
FR2: SWE can edit an existing goal before execution.
FR3: SWE can attach supporting context and instructions to a goal.
FR4: SWE can set goal priority to influence execution order.
FR5: SWE can submit a goal for orchestration processing.
FR6: System can decompose a goal into executable task units.
FR7: SWE can review decomposition output before full execution.
FR8: SWE can adjust decomposition boundaries by splitting, merging, or reordering tasks.
FR9: System can represent task dependencies and execution prerequisites.
FR10: System can identify tasks eligible for parallel execution.
FR11: System can classify tasks as human-owned or agent-owned.
FR12: SWE can override task ownership decisions.
FR13: System can route agent-owned tasks to appropriate specialized roles.
FR14: System can hold execution when required human inputs are missing.
FR15: Owner can define available specialized agent roles in a workspace.
FR16: Owner can assign role constraints that limit agent scope of work.
FR17: System can detect out-of-role agent output during execution.
FR18: System can flag and surface guardrail violations to users.
FR19: SWE can pause or reroute affected branches after guardrail violations.
FR20: System can execute multiple eligible tasks in parallel.
FR21: System can enforce dependency order for non-parallel tasks.
FR22: System can pause, resume, and cancel workflow runs.
FR23: System can detect branch conflicts before downstream continuation.
FR24: SWE can resolve branch conflicts and continue execution.
FR25: System can support partial rerun of failed or corrected branches.
FR26: Users can view workflow run status across all branches.
FR27: Users can view task-level states (queued, running, blocked, completed, failed).
FR28: Users can view handoff history between agents and tasks.
FR29: System can provide run history for troubleshooting and audit-friendly review.
FR30: Users can view failure reasons and impacted branches.
FR31: System can support multi-tenant workspaces scoped to teams or organizations.
FR32: System can isolate data and runs between tenants.
FR33: Owner can invite and manage workspace members.
FR34: System can support role-based access (Owner, Member, optional Viewer).
FR35: System can restrict actions based on workspace role permissions.
FR36: Users can trigger a workflow run from Discord.
FR37: System can post workflow status updates to Discord channels or threads.
FR38: System can post agent outputs and answers into Discord conversation context.
FR39: System can associate Discord-triggered runs with workspace context.
FR40: Users can receive failure and attention-needed notifications in Discord.
FR41: External clients can trigger workflow runs via API.
FR42: External clients can query run status and branch progress via API.
FR43: External clients can retrieve final run outcomes via API.
FR44: System can authorize API actions within workspace boundaries.
FR45: Product can operate fully without billing or subscription implementation in MVP.
FR46: Product can retain placeholder plan metadata for future monetization phases.

### NonFunctional Requirements

NFR1: System should acknowledge workflow triggers from UI/API/Discord within 3 seconds for at least 95% of requests.
NFR2: System should update workflow and branch status visibility within 5 seconds of state change for at least 95% of events.
NFR3: System should support at least 3 concurrent parallel branches per run under hackathon MVP conditions.
NFR4: System should enforce workspace-level tenant isolation for runs and artifacts.
NFR5: System should enforce role-based authorization for Owner/Member/Viewer actions.
NFR6: System should use encrypted transport for client, API, and Discord communications.
NFR7: System should preserve workflow state consistency across pause/resume/retry operations.
NFR8: System should support branch-level recovery without requiring full run restart.
NFR9: System should retain run history and failure reasons for troubleshooting.
NFR10: Discord integration should deliver status notifications with at least 90% delivery success in normal demo conditions.
NFR11: API trigger/status failures should not corrupt internal orchestration state.
NFR12: Core workflow controls should support keyboard-only navigation.
NFR13: Core status displays should provide readable contrast and clear state labels.

### Additional Requirements

- Starter template required: NestJS CLI standard starter. First implementation story should initialize with `npx @nestjs/cli@latest new agentslack --package-manager pnpm`.
- Runtime and tooling baseline: Node.js 24 LTS and pnpm 10.x.
- Persistence architecture: PostgreSQL as source of truth with Prisma migrations; Redis and BullMQ for async job execution.
- Discord integration architecture: Discord is required IO channel but not state authority.
- API architecture: REST endpoints for run creation, status retrieval, task tree, and human decision inputs.
- Security model: workspace-scoped JWT auth, role checks, strict Discord-to-workspace mapping.
- State management: orchestrator as single authority for run and task transitions.
- Eventing and observability: standardized event envelope, append-only event logs, correlation IDs, structured logs.
- Reliability needs: idempotency, dedupe behavior, retry semantics, branch conflict handling and recovery support.
- Infrastructure/deployment baseline: Railway app service plus worker service.
- Data safety constraint: avoid sensitive production data in demo paths and logs.
- UX-specific requirements: none extracted because no UX design document was provided.

### FR Coverage Map

FR1: Epic 1 - Goal creation for orchestrated runs
FR2: Epic 2 - Goal editing in iterative and recovery scenarios
FR3: Epic 1 - Context and instruction attachment for execution
FR4: Epic 1 - Goal prioritization for run order
FR5: Epic 1 - Goal submission to orchestration
FR6: Epic 1 - Goal decomposition into executable tasks
FR7: Epic 2 - Decomposition review before continuation
FR8: Epic 2 - Decomposition adjustment during refinement
FR9: Epic 1 - Task dependency representation
FR10: Epic 1 - Parallel-eligible task detection
FR11: Epic 1 - Human vs agent task classification
FR12: Epic 1 - Ownership override support
FR13: Epic 1 - Role-based routing to specialized agents
FR14: Epic 1 - Execution holds for missing human input
FR15: Epic 2 - Specialized role definitions
FR16: Epic 2 - Role constraint configuration
FR17: Epic 2 - Out-of-role output detection
FR18: Epic 2 - Guardrail violation surfacing
FR19: Epic 2 - Branch pause and reroute controls
FR20: Epic 1 - Parallel task execution engine
FR21: Epic 1 - Dependency-order enforcement
FR22: Epic 2 - Run pause, resume, cancel controls
FR23: Epic 2 - Branch conflict detection
FR24: Epic 2 - Conflict resolution continuation
FR25: Epic 2 - Partial rerun support
FR26: Epic 1 - Run status visibility during execution
FR27: Epic 1 - Task-level state visibility
FR28: Epic 2 - Handoff history visibility
FR29: Epic 2 - Run history for troubleshooting and audit
FR30: Epic 2 - Failure reason and impact visibility
FR31: Epic 2 - Multi-tenant workspace model
FR32: Epic 2 - Tenant data and run isolation
FR33: Epic 2 - Workspace member management
FR34: Epic 2 - Owner, Member, Viewer role model
FR35: Epic 2 - Permission-restricted actions
FR36: Epic 1 - Discord-triggered workflow runs
FR37: Epic 1 - Discord workflow status updates
FR38: Epic 1 - Discord agent output delivery
FR39: Epic 1 - Discord run-to-workspace association
FR40: Epic 2 - Discord failure and attention notifications
FR41: Epic 2 - API workflow trigger
FR42: Epic 2 - API run and branch status queries
FR43: Epic 2 - API final outcomes retrieval
FR44: Epic 2 - Workspace-scoped API authorization
FR45: Epic 2 - MVP operation without billing
FR46: Epic 2 - Future monetization metadata placeholder

## Epic List

### Epic 1: Discord-to-Execution Happy Path
Users can trigger a workflow from Discord, submit goal context, and see the system decompose and execute tasks in parallel with live status.
**FRs covered:** FR1, FR3, FR4, FR5, FR6, FR9, FR10, FR11, FR12, FR13, FR14, FR20, FR21, FR26, FR27, FR36, FR37, FR38, FR39

### Epic 2: Minimal Recovery and Programmatic Run Visibility
Users can handle execution interruptions with basic run controls, see failure context and history, and access run status via API.
**FRs covered:** FR2, FR7, FR8, FR15, FR16, FR17, FR18, FR19, FR22, FR23, FR24, FR25, FR28, FR29, FR30, FR31, FR32, FR33, FR34, FR35, FR40, FR41, FR42, FR43, FR44, FR45, FR46

<!-- Repeat for each epic in epics_list (N = 1, 2, 3...) -->

## Epic {{N}}: {{epic_title_N}}

{{epic_goal_N}}

<!-- Repeat for each story (M = 1, 2, 3...) within epic N -->

### Story {{N}}.{{M}}: {{story_title_N_M}}

As a {{user_type}},
I want {{capability}},
So that {{value_benefit}}.

**Acceptance Criteria:**

<!-- for each AC on this story -->

**Given** {{precondition}}
**When** {{action}}
**Then** {{expected_outcome}}
**And** {{additional_criteria}}

<!-- End story repeat -->

## Epic 1: Discord-to-Execution Happy Path

Users can trigger a workflow from Discord, submit goal context, and see the system decompose and execute tasks in parallel with live status.

### Story 1.1: Initialize Core Orchestration Service

As a platform engineer,
I want to initialize the NestJS service with core run and task modules,
So that workflow orchestration can be implemented quickly and consistently.

**Acceptance Criteria:**

**Given** a new backend repo state
**When** I run the approved starter initialization and scaffold core modules
**Then** the service starts successfully with configuration for API, Discord ingress, and queue worker boundaries
**And** baseline run and task entities required for happy-path execution are available

### Story 1.2: Trigger Run from Discord with Workspace Mapping

As an SWE using Discord,
I want to start a workflow run from Discord and have it mapped to the correct workspace,
So that execution begins from my chat context without cross-workspace leakage.

**Acceptance Criteria:**

**Given** a mapped Discord user and workspace context
**When** I issue a valid workflow trigger command
**Then** the system creates a run in the mapped workspace and returns an acknowledgement
**And** unmapped or invalid mapping attempts are rejected safely

### Story 1.3: Submit Goal and Generate Dependency-Aware Task Plan

As an SWE,
I want my submitted goal to be decomposed into tasks with dependencies and ownership routing,
So that execution can proceed in the correct order with parallelism where possible.

**Acceptance Criteria:**

**Given** a submitted goal with context and priority
**When** planning runs
**Then** the system creates task units with dependency relationships and marks parallel eligibility
**And** each task has ownership classification with override support before execution

### Story 1.4: Execute Parallel Tasks and Publish Discord Status

As an SWE,
I want eligible tasks to execute in parallel and status to update in Discord,
So that I can observe progress and outcomes in real time.

**Acceptance Criteria:**

**Given** a planned run with executable tasks
**When** execution begins
**Then** dependency-free tasks run in parallel and dependency-blocked tasks wait correctly
**And** run-level and task-level status plus agent outputs are posted to the associated Discord thread or context

## Epic 2: Minimal Recovery and Programmatic Run Visibility

Users can handle execution interruptions with basic run controls, see failure context and history, and access run status via API.

### Story 2.1: Expose Workspace-Scoped Run Status API

As an external integrator,
I want to trigger runs and fetch run and task status through API,
So that orchestration can be monitored and integrated programmatically.

**Acceptance Criteria:**

**Given** an authorized workspace-scoped API client
**When** it calls trigger and status endpoints
**Then** the system creates or returns run and task data within workspace boundaries
**And** unauthorized or cross-workspace requests are rejected with consistent error responses

### Story 2.2: Provide Minimal Run Controls and Failure Diagnostics

As an SWE,
I want to pause, resume, and cancel runs and view failure reasons with impacted branches,
So that I can recover quickly when execution goes wrong.

**Acceptance Criteria:**

**Given** an active or failed run
**When** I invoke pause, resume, or cancel and request diagnostics
**Then** the run transitions consistently and diagnostics include failure reason and impacted scope
**And** status updates and attention-needed notifications are emitted to Discord

### Story 2.3: Support Conflict Handling and Partial Rerun

As an SWE,
I want branch conflicts detected and to rerun only affected parts,
So that I avoid full restart and preserve completed work.

**Acceptance Criteria:**

**Given** a run with conflicting branch outcomes or failed branches
**When** conflict detection and rerun actions are applied
**Then** conflicts are surfaced before downstream continuation
**And** partial rerun re-executes only selected affected branches while preserving unaffected results

### Story 2.4: Establish Minimal Role, Tenancy, and Commercial Boundary Baseline

As a workspace owner,
I want basic member roles, tenant isolation, and no-billing MVP operation,
So that the platform is usable safely in PoC scope without monetization overhead.

**Acceptance Criteria:**

**Given** a multi-tenant workspace environment
**When** members are assigned Owner, Member, and Viewer permissions and runs execute
**Then** role restrictions and tenant isolation are enforced for run and API actions
**And** the product operates fully without billing while retaining placeholder metadata for future plans
