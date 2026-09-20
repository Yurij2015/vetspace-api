---
type: Behaviour Spec
title: "Admin Panel — Platform Operations Surface"
description: "The Filament v5 admin panel: resource coverage for tenants, users, plans, orders, catalog and content management, dashboard metrics widgets, and role-based access control on a dedicated guard."
tags: [admin, filament, rbac]
status: stable
sources:
  - id: dashboard-screenshot
    resource: docs://screenshots/admin-dashboard.png
  - id: clinic-catalog-screenshot
    resource: docs://screenshots/admin-clinic-catalog.png
domain: admin.panel
last_verified_at: 2026-09-19
---

# Admin Panel — Platform Operations Surface

## Domain
`admin.panel`

## Overview

A Filament v5 panel at `/admin` (central domain only) gives platform operators full control over tenants, billing, catalog and content. Access uses a dedicated `filament` guard with role/permission management from the access-control package.

---

## 1. Resource coverage

| Area | Resources |
|---|---|
| Tenancy | Tenants, Domains, Users |
| Billing | Membership Plans (with Stripe sync), Orders |
| Catalog | Clinic Catalog (import/export, draft/publish workflow, bulk status actions), Free Clinic Bindings, Reviews, FAQ |
| CRM content | Landing Features, Landing Settings, Social Networks (footer) |
| Locations | Countries, Cities |
| Animals | Species, Breeds |
| Administration | Admin Users, Roles, Permissions |

## 2. Dashboard widgets

- Active subscriptions count and current MRR from active subscriptions.
- New orders (7d), total tenants, tenants without a linked user, total domains.
- Health check: tenants missing storage (schema/database) — surfaces provisioning gaps.
- Trend charts: active orders and new tenants over 30 days.

## 3. Operational details

- Full i18n: panel locale switcher (en/uk/pl) plus translatable resource fields via spatie-translatable.
- Sidebar badge counts per resource for at-a-glance volumes.
- Database notifications enabled; collapsible sidebar; sticky table headers on dense lists.
- Real-time broadcast wiring is conditionally injected when echo config is present.
