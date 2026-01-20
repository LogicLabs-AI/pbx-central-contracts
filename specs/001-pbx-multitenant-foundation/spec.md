# Feature Specification: Multitenant PBX Foundation

**Feature Branch**: `001-pbx-multitenant-foundation`  
**Created**: 2026-01-19  
**Status**: Approved  
**Input**: User description: "Objetivo Estabelecer a fundação PBX multitenant..."

## Clarifications

### Session 2026-01-19
- Q: Deployment preference (Docker vs Native)? → A: No Docker. Infrastructure is already ready in the cloud; system must rely on native/VM-based deployment.
- Q: Target Production OS? → A: Linux (Debian/Ubuntu).
- Q: Database Multi-tenancy Strategy? → A: Postgres Schemas (Separate Schema per Tenant).
- Q: XML Handler Strategy? → A: Tera Templates (Server-Side Rendering).
- Q: API Spec Source of Truth? → A: Single `contracts/openapi.yaml` (merged).
- Q: FreeSWITCH Handler Parsing? → A: Strict Struct Binding.
- Q: PBX Engine and Database? → A: FreeSWITCH + PostgreSQL.
- Q: Backend Language & Framework? → A: Rust (Axum) + Swagger (OpenAPI).
- Q: Frontend Technology? → A: React + Vite + TailwindCSS.
- Q: Directory Structure? → A: Polyrepo style at root (`/pbx-server`, `/pbx-web`).
- Q: Web Authentication? → A: JWT (Stateless) for API access.
- Q: Database Interaction Library? → A: SQLx.
- Q: IVR/Call Logic Strategy? → A: Dynamic XML (mod_xml_curl).
- Q: Frontend State Management? → A: React Context (Built-in).
- Q: High Availability Scope? → A: Single Node Foundation (MVP).
- Q: Audio Storage? → A: Local Filesystem.
- Q: Queue Strategy? → A: Native `mod_callcenter` (XML/DB Config).
- Q: Queue Attributes? → A: Static membership (admin-assigned tiers).
- Q: Audio Conversion? → A: FFmpeg (CLI wrapper).
- Q: Eavesdrop Method? → A: Feature Code (*88 + Ext) via Dialplan.
- Q: Web User <-> SIP Extension Strategy? → A: 1:1 Optional (User can have 0-1 Ext; Ext can have 0-1 User).
- Q: Tenant Config Strategy? → A: Columns on Tenant Table (Start with defaults: Timezone, Locale).
- Q: API Tenant Context Strategy? → A: Path Parameter (e.g., `/tenants/{id}/...`) with Middleware validation.
- Q: Queue Membership Strategy? → A: Tiers on Extensions (Direct assignment in DB, mapped to mod_callcenter).
- Q: Backend <-> FS Control Strategy? → A: ESL (Event Socket Library) via TCP (Inbound/Outbound as needed).

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Super Admin & Tenant Setup (Priority: P1)

A Super Admin sets up the platform structure by creating independent tenants (organizations) for customers.

**Why this priority**: foundational step; without tenants, no other resources can exist in a multitenant system.

**Independent Test**: Can be tested via API/Admin Panel by creating records and verifying DB state.

**Acceptance Scenarios**:

1. **Given** a logged-in Super Admin, **When** they create a new Tenant with domain "company-a.pbx.io", **Then** the tenant is created and active in the system.
2. **Given** two created tenants, **When** the Super Admin views the tenant list, **Then** both are visible and distinct.

---

### User Story 2 - User & Secure Extension Provisioning (Priority: P1)

An Admin creates users and assigns them SIP extensions. The system automatically creates secure SIP credentials to prevent brute-force attacks.

**Why this priority**: Security against SIP scanners is a critical requirement (GoTo standard).

**Independent Test**: Create an extension and inspect the generated credentials. Try to register.

**Acceptance Scenarios**:

1. **Given** a Tenant Admin, **When** they create an extension "1000", **Then** the system generates a random UUID as the `auth-user` and a strong random password, NOT using "1000" for authentication.
2. **Given** the generated credentials, **When** a SIP client attempts to register using the UUID/Password, **Then** registration is successful.
3. **Given** an attacker, **When** they try to register using "1000" as the username, **Then** authentication fails.

---

### User Story 3 - Core Calling & Isolation (Priority: P1)

Users in the same tenant can call each other, but cannot dial extensions in other tenants internally.

**Why this priority**: Core value proposition of a PBX; Isolation is critical for multitenancy Security.

**Independent Test**: Two phones registered in Tenant A, one in Tenant B. Attempt calls.

**Acceptance Scenarios**:

1. **Given** Ext 1000 and Ext 1001 in Tenant A both registered, **When** 1000 dials 1001, **Then** the call connects with 2-way audio.
2. **Given** Ext 1000 in Tenant A and Ext 1000 in Tenant B (same number, different tenant), **When** Tenant A's user dials 1000, **Then** it rings their own voicemail or fails busy (does NOT ring Tenant B).

---

### User Story 4 - Call Center Basic Features (Priority: P2)

Admin configures queues with custom Music on Hold, and Agents receive calls from the queue.

**Why this priority**: Essential feature set for business use beyond simple calling.

**Independent Test**: Call a configured queue DID/Extension.

**Acceptance Scenarios**:

1. **Given** a queue configured with a custom uploaded MP3 as MOH, **When** a caller enters the queue, **Then** they hear the converted custom audio.
2. **Given** a queue with strategy "Ring-All", **When** a call enters, **Then** all available logged-in agents ring simultaneously.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST support hierarchical multi-tenancy using **PostgreSQL Schemas** (Schema-per-Tenant) to ensure strict data isolation.
- **FR-002**: System MUST decouple web user authentication (email/password) from SIP extension authentication.
- **FR-003**: System MUST provide a "Super Admin" role capable of accessing and managing all tenants and their resources globally.
- **FR-004**: System MUST automatically generate cryptographically secure, random credentials (UUID username, strong password) for SIP authentication, distinct from the display extension number.
- **FR-005**: System MUST dynamically authenticate all SIP registration and call attempts against the central database (no static user configuration files).
- **FR-006**: System MUST enforce strict dialplan context isolation, preventing internal dialing between extensions of different tenants.
- FR-007: System MUST support ACD Queues using **FreeSWITCH mod_callcenter**, configured via XML/DB, with static member assignment managed by Administrators (no dynamic login/logout for MVP).
- FR-008: System MUST allow upload of audio files (MP3/WAV) per tenant and automatically convert them to the required format (8kHz/16kHz WAV mono) using **FFmpeg** (CLI execution).
- **FR-009**: System MUST support dynamic generation of IVR (Interactive Voice Response) menus based on database configuration.
- **FR-010**: System MUST support enabling call recording on a per-extension or per-queue basis, saving files with tenant metadata.
- FR-011: System MUST provide an eavesdrop capability via **Feature Code** (e.g., `*88` followed by extension), allowing authorized administrators to listen in on active calls.
- **FR-012**: System MUST use **FreeSWITCH** as the core telephony engine, utilizing **mod_xml_curl** to fetch dynamic XML dialplans/IVRs from the Rust backend and **ESL (Event Socket Library) via TCP** for real-time control and event monitoring.
- **FR-013**: System MUST use **PostgreSQL** as the primary persistent data store for tenants, users, extensions, and CDRs (Call Detail Records).

### Key Entities *(include if feature involves data)*

- **Tenant**: Represents a customer organization. Attributes: Name, SIP Domain, UUID, Timezone (e.g., 'UTC'), Locale (e.g., 'en_US').
- **User**: Represents a human login to the admin panel. Attributes: Email, Password Hash, Role (Super/Tenant/Agent). Relationship: Can be associated with 0 or 1 Extension (1:1 Optional).
- **Extension**: Represents a SIP endpoint. Attributes: Extension Number (Display), Auth Username (UUID), Auth Password, Context, Associated User. Relationship: Can be associated with 0 or 1 User (1:1 Optional).
- **Queue**: Represents a call distribution point. Attributes: Name, Strategy, Music-on-Hold path. Relationship: Has Many Extensions (Members) with a Tier level.

### Technical Constraints

- **TC-001**: System MUST NOT use Docker or containerization. Deployment target is pre-existing cloud infrastructure running **Linux (Debian/Ubuntu)**.
- **TC-002**: Application MUST be capable of running as a standard system service (implementation details pending stack selection).
- **TC-003**: Backend services MUST be implemented in **Rust** using the **Axum** framework and **Tera** for XML templating.
- **TC-004**: API MUST be fully documented using **Swagger/OpenAPI** (v3.0+) and exposed via the backend.
- **TC-005**: Frontend MUST be built using **React** (TypeScript), **Vite**, and **TailwindCSS** to achieve a modern, premium aesthetic.
- **TC-006**: Project Structure MUST use top-level directories: `pbx-server` (Rust source) and `pbx-web` (Frontend source).
- **TC-007**: API Authentication MUST utilize **JWT (JSON Web Tokens)** for stateless web user session management. Tenant context MUST be explicit in URL paths (e.g., `/api/v1/tenants/{tenant_id}/resource`) and validated by middleware.
- **TC-008**: Media Storage MUST use the **Local Filesystem** (e.g., `/var/lib/pbx`) for simplicity, avoiding external object storage dependencies for this MVP.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Super Admin can create 2 fully isolated tenants and provision users in both within 5 minutes.
- **SC-002**: WebRTC client (SIP.js) registers successfully (200 OK) using generated UUID credentials.
- **SC-003**: Internal calls establish bidirectional audio (Opus codec confirmed) with < 200ms setup time.
- **SC-004**: Cross-tenant calls to identical extension numbers result in failure or local-only routing (0% leakage).
- **SC-005**: Uploaded MP3 audio files are successfully converted and playable as Music on Hold within 60 seconds of upload.
- **SC-006**: Call recordings are persisted to storage and accessible via API/Admin panel immediately after call termination.

### Edge Cases

- **EC-001**: Creation of duplicate extension numbers within the SAME tenant must be blocked.
- **EC-002**: Creation of duplicate extension numbers in DIFFERENT tenants must be allowed.
- **EC-003**: Authenticated user trying to access resources of another tenant via API (IDOR check) must be denied.
- **EC-004**: System behavior when audio conversion service fails (fallback to default tone/MOH).
