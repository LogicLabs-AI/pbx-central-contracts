# Research & Design Decisions: Multitenant PBX Foundation

**Branch**: `001-pbx-multitenant-foundation` | **Date**: 2026-01-19

## Summary

This document captures the technical decisions for the PBX foundation, focusing on Rust/FreeSWITCH integration, Multi-tenancy architecture, and API security.

## 1. Rust <-> FreeSWITCH Integration

**Goal**: Establish efficient Control (Commands) and Configuration (XML) channels between Axum backend and FreeSWITCH.

### Decision: `eslrs` (or equivalent Tokio-native impl) for ESL; `mod_xml_curl` for Config

**Rationale**: 
- **Control**: `mod_event_socket` is the standard control plane. Using a Tokio-based library (`eslrs`) ensures the backend handles high concurrency of events without blocking threads, aligning with Axum's architecture.
- **Config**: `mod_xml_curl` is the only viable way to serve dynamic dialplans from a database.
- **Data Flow**:
  - *Inbound Call* -> FS -> `mod_xml_curl` -> Rust (Axum) -> Render Template from DB -> Return XML
  - *API Command (e.g. Hangup)* -> Rust -> ESL Connection -> FS -> Execute

**Alternatives Considered**:
- *Legacy XML-RPC*: Rejected due to blocking nature and poor performance.
- *Static Files with Reload*: Rejected as it violates the dynamic tenant creation requirement (requires FS reload).

## 2. Multi-tenancy Architecture

**Goal**: Strict data isolation per user story requirements (Security).

### Decision: PostgreSQL Schema-per-Tenant

**Implementation Strategy**:
- **Global Schema** (`public`): Stores `tenants`, `users` (system-wide), `system_settings`.
- **Tenant Schemas** (`tenant_<uuid>`): Stores `extensions`, `cdrs`, `queues`, `media_files`.
- **Middleware Logic**:
  1. Extract `tenant_id` from URL path.
  2. Verify user has access to this tenant.
  3. Lease DB connection.
  4. Exec `SET search_path TO tenant_<uuid>, public;`
  5. Pass connection to handler.

**Rationale**:
- **Security**: Native database-level isolation prevents accidental data leakage in complex queries.
- **Backup/Restore**: Easy to dump a single tenant's schema for portability.
- **Performance**: Smaller index sizes per tenant compared to one giant table with `tenant_id` columns.

**Alternatives Considered**:
- *Discriminator Column (`where tenant_id = ?`)*: Rejected due to risk of developer error (forgetting the where clause) leading to data leaks.

## 3. Web & API Security context

**Goal**: Stateless, secure access control.

### Decision: JWT + Explicit Path Context

**Key Claims**:
- `sub`: User UUID
- `role`: global_admin | tenant_admin | agent
- `tid`: Tenant UUID (if scoped to one tenant)

**Context Resolution**:
- **Super Admin**: Can access any `/tenants/{id}/...`.
- **Tenant Admin**: Can only access `/tenants/{their_id}/...`. Middleware checks `Path.id == JWT.tid`.

## 4. XML Templating Strategy

**Goal**: Maintainable, dynamic XML generation.

### Decision: Tera Templates

**Structure**:
```text
src/templates/
  ├── dialplan/
  │   ├── inbound_context.xml.tera
  │   └── outbound_context.xml.tera
  ├── directory/
  │   └── user_profile.xml.tera
  └── ivr/
      └── menu.xml.tera
```

**Rationale**: 
- Tera is modeled after Jinja2, widely used in Rust, and safe (auto-escaping).
- Separating templates from code keeps Rust logic clean.

## 5. Media Handling

**Goal**: Simple, performant storage.

### Decision: Local Filesystem with Nginx/Axum Serving

**Path Structure**:
`/var/lib/pbx/tenants/{tenant_uuid}/{category}/{filename}`
Categories: `recordings`, `music_on_hold`, `voicemail`.

**Rationale**:
- "Native" constraint rules out S3/MinIO for MVP.
- OS-level permissions can further secure these directories.
- Axum can stream these files efficiently or delegate to a reverse proxy (Nginx) in production.
