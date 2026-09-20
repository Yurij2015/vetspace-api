---
type: Architecture Spec
title: "System Overview — VetSpace Platform Backend"
description: "High-level topology of the platform: a Laravel monolith serving two API routing contexts (central + tenant), a Go AI microservice, a Filament admin panel, and the surrounding infrastructure services."
tags: [architecture, overview]
status: stable
sources:
  - id: readme
    resource: docs://README.md
domain: architecture.overview
last_verified_at: 2026-09-19
---

# System Overview — VetSpace Platform Backend

## Domain
`architecture.overview`

## Overview

VetSpace is a multi-tenant SaaS for veterinary clinics. The backend is a Laravel monolith that serves three audiences — clinic staff on tenant domains, pet owners on the central domain, and platform administrators in a Filament panel — plus a Go microservice for AI provider calls.

---

## 1. Components

| Component | Tech | Role |
|---|---|---|
| Core API | Laravel 13 / PHP 8.5 | REST API for both routing contexts, GraphQL endpoint, webhooks |
| Tenant API | same app, domain-scoped routes | Clinic cabinet: branches, doctors, services, appointments, calendar, reviews |
| Admin panel | Filament v5 | Platform operations: tenants, plans, orders, catalog, access control |
| AI microservice | Go | Provider calls (Gemini default), invoked by the PHP orchestrator over HTTP |
| Queue | Redis + Horizon | Queued notifications, jobs, tenant provisioning pipeline |
| Broadcasting | Pusher protocol | Real-time appointment/calendar events |
| Mail | Mailgun | Transactional notifications |
| Payments | Stripe (Cashier) | Subscription plans, checkout, webhooks |

## 2. Request surfaces

- **Central domain** (`/api/*`) — owner/registration auth (`*-main` suffixed routes to avoid name collisions), pets, clinic catalog, orders, profile, AI analysis entry point.
- **Tenant domains** (`/api/*` on `{tenant}.{domain}`) — all clinic-cabinet operations behind `InitializeTenancyByDomain`, which is re-prioritised to run *before* `auth:sanctum` so bearer tokens resolve against the tenant connection.
- **Admin** (`/admin`) — Filament panel on the central domain only, separate `filament` guard.
- **GraphQL** — Lighthouse endpoint for richer queries.
- **Public booking** — unauthenticated appointment endpoints on tenant domains (slot listing + booking).

## 3. Tenant provisioning

Tenant creation runs a job pipeline: `CreateDatabase → MigrateDatabase → SeedDatabase → CreateMainBranch`. Isolation type is decided by plan at provisioning time (see `architecture/multi-tenancy.spec.md`).

## 4. Security posture

- All API requests require an `X-Frontend-Key` header (shared frontend secret) in addition to Sanctum tokens.
- Stripe webhooks verified by signature; Telegram bot handled via webhook + service layer.
- Destructive DB commands prohibited in production at the provider level.
