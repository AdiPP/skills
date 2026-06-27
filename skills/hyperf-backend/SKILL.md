---
name: hyperf-backend
description: Generate Hyperf microservice backend code with DDD, Clean Architecture, menumbing packages, and Kafka event streaming. Use when creating services, Models, Repositories, Actions, or DTOs.
---

# Hyperf Backend Skill

Generic skill for generating Hyperf-based microservice backend code aligned with DDD + Clean Architecture, using `menumbing/*` packages and Kafka event streaming.

## Placeholder Convention

When applying this skill, replace:

| Placeholder | Format | Example |
|-------------|--------|---------|
| `{ProjectName}` | PascalCase — used in PHP namespaces | `MagangHub`, `Saridana`, `Naker` |
| `{project-name}` | kebab-case — used in DB names, Kafka group, Docker names | `maganghub`, `saridana`, `naker` |
| `{project_name}` | snake_case — used in DB schema, prefix, env vars | `maganghub`, `saridana` |
| `{Domain}` | PascalCase — bounded context name | `Identity`, `Vacancy`, `Payment` |
| `{service}` | kebab-case — service identifier | `vacancy`, `auth`, `loan` |
| `{Name}` / `{Entity}` | PascalCase — entity / aggregate name | `Program`, `User` |
| `{Action}` | PascalCase verb — use case action | `Create`, `Update`, `Verify`, `Approve`, `Reject`, `Cancel`, `Search`, `Rotate`, `Submit` |
| `{Suffix}` | PascalCase role for Service classes | `Creator`, `Updater`, `Verifier`, `Rotator`, `Checker`, `Manager`, `Resolver`, `Approver` |

## Project Init

Create new service using:

```bash
composer create-project hyperf/hyperf-skeleton:dev-master
```

Namespace convention: `{ProjectName}\{Domain}\` (e.g., `{ProjectName}\Identity`, `{ProjectName}\Vacancy`, `{ProjectName}\Common`)

## Folder Structure

```
config/
├── autoload/
│   ├── auth.php
│   ├── async_queue.php
│   ├── cache.php
│   ├── databases.php
│   ├── dependencies.php
│   ├── event_stream.php
│   ├── exceptions.php
│   ├── graceful_process.php
│   ├── http_client.php
│   ├── middlewares.php
│   ├── oauth2_resource_server.php
│   ├── opentracing.php
│   ├── orm.php
│   ├── redis.php
│   ├── resource.php
│   ├── {upstream}.php       # One per outbound upstream (credentials/identifiers, env-backed)
│   └── ...
├── config.php
├── container.php
└── routes.php
migrations/
seeders/
src/
├── Action/              # Use Case Handlers (Application Layer) with routing annotations
├── Cache/               # Cache key generator classes
├── Command/             # CLI commands (sync, scheduler, batch)
├── Constant/            # Enums and constants
├── Controller/          # Rare, for complex auth flows
├── DTO/                 # Data Transfer Objects (per module)
├── Event/               # Domain Events with ProducedEvent annotation
├── Exception/           # Custom exceptions + Handler/
├── Factory/             # DI factories (e.g., HttpClient construction)
├── HttpClient/          # Outbound HTTP clients per upstream (one folder per service)
├── Job/                 # Async queue jobs (when extending Hyperf\AsyncQueue\Job)
├── Listener/            # Event listeners (also async queue handlers via #[AsyncQueueMessage])
├── Middleware/          # HTTP middlewares (server-side and HttpClient/ middlewares)
├── Model/               # Eloquent-like models (Menumbing ORM)
├── Query/               # Query objects (rare)
├── Relation/            # Relation traits for models
├── Repository/          # Repository classes
├── Request/             # Form validation requests
├── Resource/            # API response resources
└── Service/             # Domain services (business logic)
```

---

## Key Patterns — Quick Reference

For full code templates, see [references/patterns.md](references/patterns.md).

| Pattern | Naming | Location |
|---------|--------|----------|
| Action (Use Case Handler) | `{Action}{Name}Action` — invoked via `__invoke()` | `src/Action/` |
| Model | `{Name}` — extends `Menumbing\Orm\Model`, UUID v7 primary key | `src/Model/` |
| Repository | `{Name}Repository` — annotated with `#[AsRepository]` | `src/Repository/` |
| Domain Service | `{Name}{Suffix}` — `final class`, one method, `#[Transactional]` | `src/Service/{Name}/` |
| DTO | `{Action}{Name}DTO` — `readonly` properties, one per use case | `src/DTO/` |
| Domain Event | `{Entity}{Action}` — `#[ProducedEvent]` annotation, Kafka stream | `src/Event/` |
| Event Listener | `{Name}Listener` — `#[Listener]`, processes domain events | `src/Listener/` |
| Form Request | `{Action}{Name}Request` — validation rules | `src/Request/` |
| Migration | UUID primary key, `datetimes()`, `softDeletes()` | `migrations/` |
| Cache Key | `{Name}Key::generate()` — prefixed with `{project_name}:` | `src/Cache/` |

### Suffixes (Domain Service roles)

| Suffix | Role | Example |
|--------|------|---------|
| `Creator` | Creates a new aggregate | `UserCreator` |
| `Updater` | Updates an existing aggregate | `ProgramUpdater` |
| `Verifier` | Validates / verifies state | `OTPVerifier` |
| `Approver` / `Rejecter` | Workflow transitions | `ProposalApprover`, `PositionRejecter` |
| `Rotator` | Regenerates / rotates value | `ClientSecretRotator` |
| `Checker` | Uniqueness / preconditions | `UserUniquenessChecker` |
| `Resolver` | Read with caching / lookup | `PermissionResolver` |
| `Manager` | Multi-purpose for one aggregate | `UserPinManager` |
| `Invalidator` / `Requester` | Domain-specific verbs | `OTPInvalidator`, `OTPRequester` |

### Async Queue

- Prefer `#[AsyncQueueMessage]` on `ListenerInterface::process()` (Pattern A)
- Use `Job` class only when no triggering event (Pattern B)
- Tune `maxAttempts`: `1` non-idempotent, `3` typical, `5` flaky external

### HTTP Client

- Never instantiate Guzzle directly; declare upstreams in `http_client.php` — auto-registered by listener
- Each upstream: `HttpClient/{Upstream}Client/` + `Factory/{Upstream}HttpClientFactory.php` + `dependencies.php` binding
- Credentials in `config/autoload/{upstream}.php` (env-backed); Factory reads via `ConfigInterface::get()`, never `env()`
- Token caching with thundering-herd lock: `expire expires_in - 60s`

### Authentication

- Every public Action MUST declare `#[Auth(guards: ...)]`
- `oauth2` for user tokens, `client` for service-to-service
- Stateless providers when service does NOT own user/client aggregate
- Per-endpoint scope via `options: ['scope' => '...']` in routing
- Inject principal with `#[AuthUser(for: 'user'|'client')]`

### Relation Trait

Each relation lives in its own trait under `src/Relation/{Related}/`:
- `BelongsTo{Related}Trait` — singular: `{related}()`
- `HasOne{Related}Trait` — singular: `{related}()`
- `HasMany{Related}sTrait` — plural: `{related}s()`
- `BelongsToMany{Related}sTrait` — plural: `{related}s()`
- `MorphTo` — inline on polymorphic owner model (not a trait)

---

## Config Templates

See [references/config-templates.md](references/config-templates.md) for full templates:
- `databases.php`, `orm.php`, `cache.php`
- `async_queue.php` (AMQP/RabbitMQ)
- `event_stream.php` (Kafka)
- `http_client.php` (menumbing/http-client)
- `opentracing.php` (menumbing/tracer)
- `auth.php`, `oauth2_resource_server.php`
- `middlewares.php`

---

## Rules & Requirements

See [references/rules.md](references/rules.md) for the full 31 rules and `composer.json` requirements.

Key rules:
- Always `declare(strict_types=1);`
- UUID v7 via `Str::orderedUuid()->toString()` for primary keys
- PHP 8.4 enums for status fields
- `readonly class` for Actions and DTOs; `final class` for Services
- All API endpoints prefixed with `/v1`
- Cross-service communication only via Kafka events or API calls
- **MANDATORY** `menumbing/graceful-process` in every service
