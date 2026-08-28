# Messenger CRM

A provider-independent, multi-tenant SaaS CRM for managing Facebook Messenger conversations, with an architecture designed to absorb future Meta API and webhook changes with minimal impact on the core application.

## Architecture status

- Database Architecture v1.0 — frozen
- Application Architecture v1.0 — frozen
- Provider Abstraction v1.0 — frozen
- Meta Integration Strategy v1.0 — frozen

## Planned stack

- Laravel 13
- PHP 8.3+
- React 19 + TypeScript
- Inertia 3
- Vite
- MySQL 8+
- Redis
- Laravel Reverb
- Laravel queues
- GitHub Actions

Laravel 13 is the current framework target for this project. The official Laravel React starter kit provides React, TypeScript, Inertia, authentication, settings, Tailwind v4 and shadcn/ui scaffolding; we will use that foundation and keep our domain architecture independent of the starter UI choices.

## Core principle

Meta is an external provider, not the architecture of the CRM.

All Meta-specific webhook parsing, Graph API calls, version handling, capabilities and transformations must remain behind `app/Integrations/Meta` and the provider contracts.

## Repository workflow

Changes should be made in small, reviewable commits. Database contracts and integration contracts must be treated as versioned interfaces.

## Initial setup

See [docs/BOOTSTRAP.md](docs/BOOTSTRAP.md).

## Architecture

See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Database design

See [docs/DATABASE.md](docs/DATABASE.md).

## License

Proprietary. All rights reserved unless a separate license is provided.