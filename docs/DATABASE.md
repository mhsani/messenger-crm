# Messenger CRM — Database Architecture v1.0

## Tenant root

`organizations` is the tenant root. Organization-owned records carry `organization_id` and must be authorized against the active organization.

## Core tables

- `organizations`
- `users`
- `roles`
- `permissions`
- `role_permissions`
- `user_roles`
- `teams`
- `team_members`
- `facebook_pages`
- `facebook_connections`
- `customers`
- `customer_identities`
- `conversations`
- `conversation_participants`
- `messages`
- `message_attachments`
- `tags`
- `customer_tags`
- `conversation_tags`
- `conversation_assignments`
- `internal_notes`
- `saved_replies`
- `automations`
- `tasks`
- `notifications`
- `webhook_events`
- `api_logs`
- `audit_logs`

## Provider isolation

The CRM tables use generic fields such as `provider`, `external_id`, `type`, and `metadata`. Meta-specific structures remain in integration tables or JSON metadata. Do not add Meta-specific columns to core CRM tables unless there is a demonstrated domain-level requirement.

## Important invariants

- UUIDs are used for public identifiers; numeric primary keys may be used internally for efficient joins.
- Provider external IDs are not treated as internal primary keys.
- Messages are append-oriented and should preserve provider IDs for idempotency.
- Webhook events are retained for replay and debugging.
- Access tokens are encrypted at rest and never exposed through API resources.
- High-volume tables receive explicit composite indexes based on actual query patterns.
- Soft deletes are used only where recovery/history is meaningful.

## Message model

`messages.type` is a string rather than a restrictive database enum. Unknown provider message types can therefore be persisted as `unknown` or a provider-specific type without a schema migration.

## Webhook model

`webhook_events` stores:

- provider
- provider event ID
- event type/version
- external object ID
- payload
- headers (with sensitive values redacted)
- receipt/processing timestamps
- status
- attempt count
- error message

This supports idempotency, replay and future provider-format migration.

## Relationships

```text
Organization
├── Users ──< Team Members >── Teams
├── Facebook Pages ──< Facebook Connections
├── Customers ──< Customer Identities
│            └──< Customer Tags >── Tags
├── Conversations
│   ├──< Participants
│   ├──< Messages ──< Attachments
│   ├──< Assignments
│   ├──< Internal Notes
│   └──< Conversation Tags >── Tags
├── Saved Replies
├── Automations
├── Tasks
├── Notifications
├── Webhook Events
├── API Logs
└── Audit Logs
```

## Migration policy

Migrations must be additive and reversible where practical. Destructive changes require a separate migration and an explicit data-migration plan. Database contracts are versioned and treated as application interfaces.