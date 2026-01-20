# Implementation Plan: Multitenant PBX Foundation

**Branch**: `001-pbx-multitenant-foundation` | **Date**: 2026-01-19 | **Spec**: [specs/001-pbx-multitenant-foundation/spec.md](../spec.md)
**Input**: Feature specification from `specs/001-pbx-multitenant-foundation/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command. See `.specify/templates/commands/plan.md` for the execution workflow.

## Summary

This feature establishes the foundational multi-tenant PBX system using FreeSWITCH, Rust, and React. It implements strict tenant isolation using PostgreSQL Schemas, decoupling web users from SIP extensions. It includes the core "Super Admin" creation flow, secure SIP credential generation, and basic call center queue management via `mod_callcenter` with static agent tiering.

## Technical Context

**Language/Version**: Rust (Latest Stable), TypeScript (5.x)
**Primary Dependencies**: 
- Backend: `Axum` (Web), `SQLx` (DB), `Tera` (Templates), `To Tokio` (Async), `uuid`, `jsonwebtoken`, `tracing`
- Frontend: `React`, `Vite`, `TailwindCSS`, `TanStack Query`, `SIP.js`
- Telephony: `FreeSWITCH` (Native install), `mod_xml_curl`, `mod_callcenter`, `mod_event_socket`
**Storage**: PostgreSQL (Schema-per-tenant), Local Filesystem (Audio/Recordings)
**Testing**: `cargo test` (Backend Unit/Int), `Vitest` (Frontend), Manual/Scripted SIP E2E (SIP.js scripts)
**Target Platform**: Linux (Debian/Ubuntu) - Native Service
**Project Type**: Web Application (Monorepo-style with `pbx-server` and `pbx-web` directories)
**Performance Goals**: Call setup < 200ms, API response < 100ms
**Constraints**: No Docker/Containers. Strict schema isolation.
**Scale/Scope**: Foundation for future modules. Initial scope: ~2 Tenants, ~10 extensions/queues for validation.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- [x] **I. Code Quality**: Rust ensures memory safety; TypeScript ensures frontend type safety. Standard linters (`cargo clippy`, `eslint`) will be used.
- [x] **II. Testing Standards**: Plan includes Backend Unit tests and E2E SIP validation.
- [x] **III. UX Consistency**: TailwindCSS and React ensure a consistent, premium design system foundation.
- [x] **IV. Performance**: Native Rust + FreeSWITCH avoids virtualization overhead; File-system serving for media is efficient.

## Project Structure

### Documentation (this feature)

```text
specs/001-pbx-multitenant-foundation/
├── plan.md              # This file
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
├── contracts/           # Phase 1 output
└── tasks.md             # Phase 2 output
```

### Source Code (repository root)

```text
pbx-server/
├── src/
│   ├── api/             # Axum handlers (REST + XML_CURL)
│   ├── db/              # SQLx models and schema management
│   ├── esl/             # FreeSWITCH Inbound/Outbound control
│   ├── services/        # Business logic (User, Tenant, Call)
│   ├── templates/       # Tera templates for XML dialplans
│   └── main.rs          # Entry point and config
└── tests/

pbx-web/
├── src/
│   ├── components/      # Reusable UI elements
│   ├── pages/           # Admin Dashboard pages
│   ├── hooks/           # API and SIP hooks
│   └── lib/             # API client and utils
└── tests/
```

**Structure Decision**: Selected a polyrepo-style structure within the monorepo root (`pbx-server`, `pbx-web`) to clearly separate the Rust backend from the React frontend, as required by **TC-006**.

## Complexity Tracking

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| Schema-per-tenant | Strict data isolation requirement | Row-level security (discriminator column) is error-prone and harder to backup/restore per tenant. |
| XML_CURL (Dynamic Config) | Dynamic behavior changes without reload | Static XML files result in downtime/reloads for every change. |
**Branch**: `[###-feature-name]` | **Date**: [DATE] | **Spec**: [link]
**Input**: Feature specification from `/specs/[###-feature-name]/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command. See `.specify/templates/commands/plan.md` for the execution workflow.

## Summary

[Extract from feature spec: primary requirement + technical approach from research]

## Technical Context

<!--
  ACTION REQUIRED: Replace the content in this section with the technical details
  for the project. The structure here is presented in advisory capacity to guide
  the iteration process.
-->

**Language/Version**: [e.g., Python 3.11, Swift 5.9, Rust 1.75 or NEEDS CLARIFICATION]  
**Primary Dependencies**: [e.g., FastAPI, UIKit, LLVM or NEEDS CLARIFICATION]  
**Storage**: [if applicable, e.g., PostgreSQL, CoreData, files or N/A]  
**Testing**: [e.g., pytest, XCTest, cargo test or NEEDS CLARIFICATION]  
**Target Platform**: [e.g., Linux server, iOS 15+, WASM or NEEDS CLARIFICATION]
**Project Type**: [single/web/mobile - determines source structure]  
**Performance Goals**: [domain-specific, e.g., 1000 req/s, 10k lines/sec, 60 fps or NEEDS CLARIFICATION]  
**Constraints**: [domain-specific, e.g., <200ms p95, <100MB memory, offline-capable or NEEDS CLARIFICATION]  
**Scale/Scope**: [domain-specific, e.g., 10k users, 1M LOC, 50 screens or NEEDS CLARIFICATION]

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

[Gates determined based on constitution file]

## Project Structure

### Documentation (this feature)

```text
specs/[###-feature]/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)
<!--
  ACTION REQUIRED: Replace the placeholder tree below with the concrete layout
  for this feature. Delete unused options and expand the chosen structure with
  real paths (e.g., apps/admin, packages/something). The delivered plan must
  not include Option labels.
-->

```text
# [REMOVE IF UNUSED] Option 1: Single project (DEFAULT)
src/
├── models/
├── services/
├── cli/
└── lib/

tests/
├── contract/
├── integration/
└── unit/

# [REMOVE IF UNUSED] Option 2: Web application (when "frontend" + "backend" detected)
backend/
├── src/
│   ├── models/
│   ├── services/
│   └── api/
└── tests/

frontend/
├── src/
│   ├── components/
│   ├── pages/
│   └── services/
└── tests/

# [REMOVE IF UNUSED] Option 3: Mobile + API (when "iOS/Android" detected)
api/
└── [same as backend above]

ios/ or android/
└── [platform-specific structure: feature modules, UI flows, platform tests]
```

**Structure Decision**: [Document the selected structure and reference the real
directories captured above]

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| [e.g., 4th project] | [current need] | [why 3 projects insufficient] |
| [e.g., Repository pattern] | [specific problem] | [why direct DB access insufficient] |
