---
type: Behaviour Spec
title: "Notifications — Channels, Recipients & Delivery Guarantees"
description: "How the platform delivers notifications: email via Mailgun and Telegram via a custom channel, per-user channel preferences, email-verification gates, and the queue-based isolation that keeps delivery failures out of user-facing requests."
tags: [notifications, email, telegram, queues]
status: stable
sources:
  - id: readme
    resource: docs://README.md
domain: notifications.delivery
last_verified_at: 2026-09-19
---

# Notifications — Channels, Recipients & Delivery Guarantees

## Domain
`notifications.delivery`

## Overview

Notifications serve two recipient populations — clinic staff (central `User` accounts) and booking clients (email addresses captured on appointments) — over two channels: email (Mailgun) and Telegram (custom channel via the Bot API).

---

## 1. Channels

- **Email** — Symfony Mailer via Mailgun transport; localized message bodies (uk/en/pl).
- **Telegram** — custom `TelegramChannel` resolved from `via()`; a single `TelegramBotService` is the only place that calls the Bot API, no-ops without a configured token, and logs failures with context.

## 2. Recipient & preference model

- Clinic users carry two independent opt-ins: `notify_email` (gated additionally by **verified email** at both toggle time and send time) and `notify_telegram` + `telegram_chat_id`.
- Booking clients receive transactional email unconditionally when an address is present — these are lifecycle notifications, not preference-gated marketing.
- Distinct lifecycle events get distinct notifications: *booking received* (at creation) vs *appointment confirmed* (on status transition) — different wording, different trigger points.

## 3. Delivery isolation

- Notification classes are **queued** — `notify()` serializes and pushes a job; actual transport happens on Horizon workers. A mail-provider failure surfaces as a failed job with retries, never as a failed HTTP request.
- The event **listener** that orchestrates delivery runs **synchronously** in the firing request — a deliberate constraint: it resolves tenant models *and* central users via `tenancy()->central()`, and executing that inside a queued job risks cross-connection serialization issues documented in the platform's specs.

## 4. Administrative surface

- Per-user preference endpoints: `PUT /api/profile/notifications` (email opt-in; enabling requires a verified address) and `POST /api/telegram/notifications`.
- Telegram binding flow: `connect → test → disconnect` endpoints on the central domain.
