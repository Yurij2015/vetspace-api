---
type: Behaviour Spec
title: "Billing — Membership Plans, Checkout & Stripe Webhooks"
description: "Subscription billing for clinic owners: a plan catalogue synced to Stripe, order/checkout flow via Cashier, webhook-driven state, and plan-based feature limits enforced across the tenant surface."
tags: [billing, stripe, subscriptions]
status: stable
sources:
  - id: plans-screenshot
    resource: docs://screenshots/admin-membership-plans.png
  - id: orders-screenshot
    resource: docs://screenshots/admin-orders.png
domain: billing.subscriptions
last_verified_at: 2026-09-19
---

# Billing — Membership Plans, Checkout & Stripe Webhooks

## Domain
`billing.subscriptions`

## Overview

Clinic owners subscribe to membership plans (Free / Starter / Clinic / Network tiers, monthly & yearly intervals). Plans are managed in the admin panel, synced to Stripe products/prices, and purchased through a checkout flow backed by Laravel Cashier.

---

## 1. Plan catalogue

- Plans are platform-managed records (price, currency, interval, status) with a **Sync with Stripe** admin action that creates/updates the corresponding Stripe product and price.
- The active plan of a tenant decides its **tenancy isolation mode** (`enterprise` → dedicated database; others → schema) at provisioning time — billing and infrastructure are directly coupled.

## 2. Orders & checkout

- `POST /api/orders/checkout` and `POST /api/orders/checkout/complete` implement the purchase flow on the central domain.
- Orders persist locally (order number, amount, status, plan, subscription reference) and are administrable in the panel.
- Subscription state is owned by Stripe and mirrored via Cashier; local records are projections.

## 3. Webhooks

- A dedicated Stripe webhook endpoint (signature-verified) applies subscription lifecycle events — activation, renewal, cancellation, payment failure — to local state.
- Plan limits (e.g. branch/doctor counts, feature flags) are enforced on the tenant surface; webhook-driven downgrades take effect without manual intervention.

## 4. Admin surface

- Filament resources for plans and orders; dashboard widgets expose revenue from active subscriptions, new orders (7d), and active-order trends (30d).
