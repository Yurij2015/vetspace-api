---
type: Architecture Spec
title: "Multi-Tenancy — Hybrid Isolation Model"
description: "How tenant data is isolated: a plan-dependent hybrid where enterprise tenants receive a dedicated PostgreSQL database and all other plans receive a PostgreSQL schema inside the central database, managed by stancl/tenancy v3 with UUID tenant identifiers."
tags: [architecture, tenancy, postgresql]
status: stable
sources:
  - id: tenants-screenshot
    resource: docs://screenshots/admin-tenants.png
domain: architecture.tenancy
last_verified_at: 2026-09-19
---

# Multi-Tenancy — Hybrid Isolation Model

## Domain
`architecture.tenancy`

## Overview

Tenant isolation is **not uniform** — it is selected per membership plan by a custom PostgreSQL manager. This keeps marginal-cost-per-tenant low for the free tier while offering strong isolation to enterprise customers.

---

## 1. Isolation modes

| Tenant plan | Isolation unit | Provisioned by |
|---|---|---|
| `enterprise` | dedicated PostgreSQL **database** `tenant{uuid}` | database-level manager |
| all others (incl. `null` → `basic`) | PostgreSQL **schema** `tenant{uuid}` inside the central DB | schema-level manager |

Consequences:

- A missing `tenant{uuid}` entry in `pg_database` means the tenant is schema-isolated — not a provisioning failure.
- Both kinds coexist: legacy tenants provisioned before the hybrid manager remain separate databases.
- Schema isolation relies on `search_path` scoping on the tenant connection; cross-tenant leakage is prevented at the connection level, not the row level.

## 2. Identification & lifecycle

- Tenants are identified by **domain**; tenant IDs are **UUIDs**.
- Creation runs a queued pipeline: `CreateDatabase → MigrateDatabase → SeedDatabase → CreateMainBranch` (roles/permissions seeded automatically).
- The `tenant` DB connection is created and purged dynamically by the tenancy package — code must never assume it persists across tenancy boundaries.

## 3. Dual-context execution

Code runs in either **tenant context** (default connection swapped to `tenant`, models hit tenant storage) or **central context**. `tenancy()->central(...)` performs a temporary switch for central-model access and reinitializes the tenant afterwards. Listeners/jobs that touch both contexts must be designed carefully around this boundary — queued execution loses the ambient tenant unless explicitly carried in the job payload.

## 4. Middleware ordering

On tenant routes, tenancy initialization executes **before** Sanctum authentication (enforced via Laravel's middleware-priority list, not the written route order). This is load-bearing: bearer tokens minted in a tenant context are looked up in that tenant's `personal_access_tokens` table.
