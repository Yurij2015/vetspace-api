---
type: Architecture Spec
title: "AI Module — Orchestrated Analysis via Go Microservice"
description: "AI-assisted analysis split across a PHP orchestration module that manages tenancy context and analysis records, and a Go microservice that performs provider calls (Gemini by default) with per-tenant prompt overrides."
tags: [ai, microservice, gemini]
status: stable
sources:
  - id: readme
    resource: docs://README.md
domain: ai.analysis
last_verified_at: 2026-09-19
---

# AI Module — Orchestrated Analysis via Go Microservice

## Domain
`ai.analysis`

## Overview

AI analysis is deliberately split in two: a PHP module inside the monolith owns orchestration (tenant context, persistence, API surface), while a small Go service owns provider communication. This keeps provider SDK churn and retry/timeout policy out of the request-critical PHP path.

---

## 1. Components

- **PHP module** — initializes the correct tenant context, creates an `AiAnalysis` record, delegates inference to the Go service, stores results.
- **Go microservice** (`ai-hub`, port 8080 in the compose network) — accepts analysis requests, calls the configured provider (`AI_PROVIDER` / `AI_MODEL`, Gemini default), returns structured output.

## 2. Entry points

- `POST /api/ai/analyze` (central) — generic analysis; accepts `tenant_id`, `data`, optional `prompt_key` and polymorphic `analyzable_*` references.
- `POST /api/visits/{id}/analyze` (tenant) — analysis bound to a specific visit record.

## 3. Prompt management

- Prompts are config-defined with `{{field}}` interpolation for record data.
- Per-tenant overrides live in a tenant `ai_prompts` table — clinics can tune wording without code changes or deployments.
