# VetSpace API — Platform Overview

> Backend of a multi-tenant SaaS platform for veterinary clinics: appointment scheduling, clinic CRM, pet-owner portal, subscriptions, AI-assisted analysis, and a full admin panel.
>
> **The source code is private.** This repository is a public technical overview — architecture, domain specifications written in the project's own OpenSpec format, and screenshots of the API documentation and admin panel.
>
> **Staging:** [API documentation (Swagger UI)](https://vet.digispace.pro/api/documentation) · [Admin panel](https://vet.digispace.pro/admin) *(requires credentials)*
>
> **Production:** [vetcard.pro](https://vetcard.pro) · [vetspace.pro](https://vetspace.pro) — the platform is live in production.
>
> **Companion repositories:** [vetspace-frontend](https://github.com/Yurij2015/vetspace-frontend) (clinic CRM panel & marketing site) · [vetspace-vetcard](https://github.com/Yurij2015/vetspace-vetcard) (public clinic portal & booking)

![Admin dashboard](docs/screenshots/admin-dashboard.png)

---

## What it does

VetSpace is a SaaS platform where every veterinary clinic gets its own subdomain and fully isolated data, while pet owners live in a shared central space:

- **Clinic cabinet (tenant domains)** — branches, rooms, doctors, services, appointment scheduling with slot availability, calendar events, reviews, public (unauthenticated) booking pages.
- **Owner portal (central domain)** — registration/auth, pets, orders & checkout, VetCard bindings to clinics, profile, notification preferences, active sessions.
- **Platform admin** — Filament panel for tenants, domains, users, membership plans, orders, clinic catalog, landing page content, roles & permissions.
- **AI module** — PHP orchestrator + a Go microservice that calls the configured AI provider (Gemini by default) for visit/data analysis, with per-tenant prompt overrides.

## Architecture

```mermaid
flowchart LR
    subgraph clients["Clients"]
        FE["Clinic cabinet SPA<br/>(tenant domain)"]
        PO["Owner portal SPA<br/>(central domain)"]
        ADM["Platform admin"]
    end

    subgraph api["Laravel 13 monolith"]
        TA["Tenant API<br/>domain → tenancy init → Sanctum"]
        CA["Central API<br/>owners, pets, orders, catalog"]
        FIL["Filament v5 panel<br/>separate guard + RBAC"]
        GQL["GraphQL<br/>(Lighthouse)"]
    end

    subgraph data["Data layer"]
        PG[("PostgreSQL 18<br/>central DB + per-tenant<br/>schema or database")]
        RD[("Redis<br/>Horizon queues, cache")]
        S3[("S3 / MinIO")]
    end

    subgraph ext["External services"]
        ST["Stripe<br/>subscriptions + webhooks"]
        MG["Mailgun"]
        TG["Telegram Bot API"]
        PU["Pusher<br/>broadcasting"]
    end

    subgraph ai["AI module"]
        ORCH["PHP orchestrator<br/>tenant context + records"]
        GO["Go microservice"]
        GEM["Gemini (configurable)"]
    end

    FE --> TA --> PG
    PO --> CA --> PG
    ADM --> FIL
    CA --> GQL
    api --> RD
    api --> S3
    CA --> ST
    api --> MG
    api --> TG
    TA --> PU
    CA --> ORCH --> GO --> GEM
```

### Highlights

- **Hybrid multi-tenancy** ([stancl/tenancy](https://tenancyforlaravel.com/) v3): tenant isolation is decided *per plan* — `enterprise` tenants get a dedicated PostgreSQL **database**, everyone else gets a PostgreSQL **schema** inside the central DB. Tenant IDs are UUIDs.
- **Two routing contexts in parallel**: central API (`/api` on the main domain) and tenant API (`/api` on each clinic's domain), with domain-based tenancy middleware reordered to run *before* Sanctum auth so tokens resolve against the tenant DB.
- **Event-driven notifications**: booking events fan out to email (Mailgun) and Telegram via queued notifications with per-user channel preferences and email-verification gates.
- **Real-time updates**: appointment and calendar events broadcast over WebSockets (Pusher protocol).
- **Spec-driven development**: behavioural contracts are maintained as OpenSpec specs (OKF frontmatter, domain-grouped) — see [`openspec/specs/`](openspec/specs/) in this repo for the public-facing set.
- **Documentation layers**: product truth in PRDs, behaviour truth in OpenSpec, API truth in generated Swagger/OpenAPI, operational truth in runbooks.

## Tech stack

| Layer | Technology |
|---|---|
| Framework | Laravel 13, PHP 8.5 |
| Database | PostgreSQL 18 — hybrid DB/schema per-tenant isolation |
| Cache / Queue | Redis (Predis), Laravel Horizon |
| Admin panel | Filament v5 (+ access control, translatable fields) |
| API docs | OpenAPI 3.0 via L5-Swagger (~140 REST routes) |
| GraphQL | Lighthouse |
| Auth | Laravel Sanctum, Socialite (OAuth) |
| Payments | Laravel Cashier (Stripe subscriptions, webhooks) |
| Broadcasting | Pusher |
| Mail | Mailgun (Symfony mailer) |
| AI provider | Go microservice → Gemini (configurable) |
| Storage | S3 / MinIO |
| Observability | Sentry, Laravel Telescope, Pulse |
| Testing | Pest 4 / PHPUnit 12 — 1100+ tests (unit / integration / feature) |

## Screenshots

### OpenAPI documentation (Swagger UI)

![API docs overview](docs/screenshots/api-docs-overview.png)

![API docs — Appointments endpoints](docs/screenshots/api-docs-endpoints.png)

### Filament admin panel

| | |
|---|---|
| ![Tenants](docs/screenshots/admin-tenants.png) | ![Users](docs/screenshots/admin-users.png) |
| ![Clinic catalog](docs/screenshots/admin-clinic-catalog.png) | ![Membership plans](docs/screenshots/admin-membership-plans.png) |
| ![Orders](docs/screenshots/admin-orders.png) | ![Login](docs/screenshots/admin-login.png) |

## Repository layout

```
openspec/specs/       Public-facing behavioural specs (OpenSpec / OKF format)
  architecture/       System overview, multi-tenancy boundaries
  appointments/       Booking, availability, lifecycle
  notifications/      Email/Telegram delivery & preferences
  billing/            Stripe subscriptions & webhooks
  auth/               API authentication model
  ai/                 AI analysis module
  admin/              Admin panel surface
docs/screenshots/     Swagger UI + Filament admin screenshots
```

## Status

Portfolio / documentation repository. Application source code, infrastructure, and product documents are private.

---

## Explore & contact

- **[Browse the live API documentation](https://vet.digispace.pro/api/documentation)** — full OpenAPI surface with try-it-out
- **[Read the domain specs](openspec/specs/)** — behavioural contracts in the project's OpenSpec format
- **[Production sites](https://vetcard.pro)** — vetcard.pro · vetspace.pro
- **Companion repos:** [vetspace-frontend](https://github.com/Yurij2015/vetspace-frontend) · [vetspace-vetcard](https://github.com/Yurij2015/vetspace-vetcard)

Built and maintained by **[Yurii Mokryi](https://yuriimokryi.vercel.app/)** — product owner & lead developer.

[Portfolio](https://yuriimokryi.vercel.app/) · [LinkedIn](https://www.linkedin.com/in/yurii-mokryi/) · [Telegram](https://t.me/YuriiMokryi) · [GitHub](https://github.com/Yurij2015)
