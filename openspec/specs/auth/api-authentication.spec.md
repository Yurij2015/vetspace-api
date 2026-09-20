---
type: Behaviour Spec
title: "Auth — Token Model Across Two Routing Contexts"
description: "Sanctum-based API authentication spanning a central domain (owner accounts) and tenant domains (clinic staff), including middleware ordering guarantees, session management endpoints, and the shared frontend key required on all requests."
tags: [auth, sanctum, tenancy, security]
status: stable
sources:
  - id: api-overview-screenshot
    resource: docs://screenshots/api-docs-overview.png
domain: auth.tokens
last_verified_at: 2026-09-19
---

# Auth — Token Model Across Two Routing Contexts

## Domain
`auth.tokens`

## Overview

Two account populations authenticate against the same API: **owners** (central `users` table) and **clinic staff** (tenant-scoped credentials). Both use Laravel Sanctum bearer tokens, but tokens resolve against different databases depending on the routing context.

---

## 1. Two login surfaces

- **Central** (`/api/register-main`, `/login-main`, `/logout-main`) — owner accounts; `-main` suffix prevents route-name collisions with tenant auth.
- **Tenant** (`/api/register`, `/login`, `/logout` on clinic domains) — clinic staff; tokens are minted into and validated against the **tenant** database's `personal_access_tokens` table.

## 2. Middleware guarantee

Tenancy initialization runs **before** `auth:sanctum` on tenant routes (enforced via global middleware priority, not route-file order). Consequence: a token minted while tenancy was active is found correctly on later tenant-domain requests. The written route ordering deliberately does not reflect execution order.

## 3. Request prerequisites

- Every API request requires `X-Frontend-Key` — a shared secret between the first-party frontend and the API, validated by middleware before any auth logic.
- Authenticated requests then carry `Authorization: Bearer <token>`.

## 4. Account surface

- Profile management: `GET/PUT /api/me`, password change, avatar upload, onboarding flag.
- Session visibility: `GET /api/me/sessions` lists active sessions; `DELETE /api/me/sessions` revokes them.
- Account lifecycle: email verification endpoints, password reset, and `DELETE /api/me` account deletion.
- OAuth: Google sign-in via Socialite (account linking for existing emails, auto-provisioning for new users).
