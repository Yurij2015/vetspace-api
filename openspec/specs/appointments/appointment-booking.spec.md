---
type: Behaviour Spec
title: "Appointments — Booking, Availability & Lifecycle"
description: "Appointment scheduling on tenant domains: public and authenticated booking, slot availability computation, status lifecycle, schedule blocks, and the real-time events emitted on creation."
tags: [appointments, scheduling, tenancy]
status: stable
sources:
  - id: api-endpoints-screenshot
    resource: docs://screenshots/api-docs-endpoints.png
domain: appointments.booking
last_verified_at: 2026-09-19
---

# Appointments — Booking, Availability & Lifecycle

## Domain
`appointments.booking`

## Overview

Appointments are tenant-scoped resources representing bookings between a pet owner and a clinic branch/doctor. The system supports both authenticated clinic-cabinet booking and unauthenticated public booking, plus non-booking schedule blocks.

---

## 1. Endpoint surface (tenant domain)

| Endpoint | Purpose |
|---|---|
| `POST /api/appointments` | Create appointment (authenticated) |
| `POST /api/public/appointments` | Public booking — no auth required |
| `GET /api/appointments/available-slots` | Free slots for a branch/doctor/date |
| `GET /api/appointments/public-available-slots` | Same, for the public booking page |
| `PUT /api/appointments/{id}` | Update appointment |
| `PATCH /api/appointments/{id}/status` | Status transition |
| `GET /api/appointments/stats` | Counts grouped by status |
| `POST /api/appointments/block` | Create a schedule block (surgery/break/vacation/other) |
| `GET /api/calendar/events` | Calendar feed of appointments + blocks |

## 2. Key behaviours

- **Slot availability** is computed from branch working hours, doctor schedules, and existing appointments — returned as concrete start/end times the frontend renders directly.
- **Public booking** creates an appointment without an account: the client identity is captured as fields on the appointment row (`owner_name`, `pet_name`, `email`, phone) rather than a user FK.
- **Schedule blocks** share the `appointments` table via a `type` discriminator (`appointment` vs `surgery`/`break`/`vacation`/`other`). They occupy slots but are excluded from booking notifications and statistics semantics that assume real bookings.
- **Status lifecycle** — `draft → confirmed → completed`, with `no_show`/`cancelled` exits. Transitions go through a dedicated endpoint that fires client-facing notifications on the transition into `confirmed`.
- **Real-time fan-out** — creation broadcasts a `ShouldBroadcast` event (`toOthers()`) over WebSockets so other cabinet sessions see new bookings live; a synchronous listener fans out email/Telegram notifications.
- **Branch timezone** — times are stored/displayed in the branch's local timezone, resolved through the appointment's branch relationship.
