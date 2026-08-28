# Messenger CRM — Application Architecture v1.0

## Principles

1. Meta is an external provider, not the CRM architecture.
2. Core domain objects are provider-independent.
3. All Meta API/webhook behavior is isolated behind integration contracts.
4. Incoming webhook payloads are persisted before asynchronous processing.
5. Unknown provider events must be safely stored and ignored, never crash the webhook endpoint.
6. Our public API is versioned independently from Meta's Graph API.
7. Provider capabilities and API versions are explicit.
8. Every organization-owned record is tenant-scoped.
9. Outbound messaging always uses a provider contract.
10. Provider changes should normally require changes only under `app/Integrations/<Provider>` and associated contract fixtures/tests.

## Runtime flow

```text
Meta Webhook
  -> Webhook Controller
  -> webhook_events
  -> Queue: webhooks
  -> Provider Parser / Normalizer
  -> Internal Domain Event
  -> Domain Handler
  -> MySQL
  -> Domain Event Broadcast
  -> Laravel Reverb
  -> React/Inertia UI
```

## Outbound message flow

```text
React
  -> API v1
  -> SendMessage application service
  -> MessagingProvider contract
  -> MetaMessengerProvider
  -> Meta Graph Client
  -> Meta API
  -> Persist message result
  -> Broadcast
```

## Proposed application tree

```text
app/
├── Domain/
│   ├── Organizations/
│   ├── Users/
│   ├── Teams/
│   ├── Customers/
│   ├── Conversations/
│   ├── Messages/
│   ├── Attachments/
│   ├── Tags/
│   ├── Assignments/
│   ├── Notes/
│   ├── SavedReplies/
│   ├── Automations/
│   ├── Tasks/
│   ├── Notifications/
│   └── Analytics/
├── Integrations/
│   ├── Contracts/
│   └── Meta/
│       ├── Graph/
│       ├── Messenger/
│       ├── OAuth/
│       ├── Webhooks/
│       ├── DTOs/
│       ├── Mappers/
│       ├── Transformers/
│       ├── Exceptions/
│       ├── Versions/
│       └── Capabilities/
├── Application/
│   ├── Commands/
│   ├── Queries/
│   ├── Services/
│   └── Jobs/
├── Infrastructure/
│   ├── Database/
│   ├── Cache/
│   ├── Queue/
│   ├── Storage/
│   └── Broadcasting/
├── Http/
│   ├── Controllers/
│   ├── Requests/
│   ├── Resources/
│   └── Middleware/
├── Events/
├── Listeners/
├── Policies/
└── Providers/
```

## Provider contract

The core application depends on interfaces such as:

- `MessagingProvider`
- `WebhookProvider`
- `ContactProvider` (future)
- `ProviderCapabilities`

Meta implements these contracts. The domain must never instantiate or call a Meta SDK/client directly.

## Internal normalized events

Initial event vocabulary:

- `IncomingMessageEvent`
- `OutgoingMessageEvent`
- `MessageDeliveredEvent`
- `MessageReadEvent`
- `MessageReactionEvent`
- `ConversationUpdatedEvent`
- `AttachmentReceivedEvent`
- `UnknownProviderEvent`

## Queue boundaries

- `webhooks`: receipt, validation, parsing, normalization
- `messaging`: outbound provider calls and message delivery work
- `automation`: rules, tags, assignments and follow-ups
- `notifications`: user notifications
- `analytics`: aggregation and reporting work
- `ai`: AI processing; must never block core message receipt
- `maintenance`: cleanup, token checks, replay and scheduled jobs

## API

The application API starts at `/api/v1/`. Meta's Graph API version is independent and is configured inside the Meta integration.

## Real-time

Laravel Reverb is the real-time transport. Authorization is required for organization, team, user and conversation channels.

## Testing strategy

- Unit tests for domain services and parsers.
- Feature tests for HTTP endpoints.
- Integration tests for queues and persistence.
- Contract tests using stored provider webhook fixtures.
- Regression fixtures for every supported Meta event shape.
- Idempotency tests for duplicate webhooks.
- Failure/retry tests for provider rate limits and token failures.

## Change policy

Changes to the core domain require architecture review. Provider-specific changes should remain isolated under the relevant integration. Meta version upgrades must include fixture and contract-test updates before activation.