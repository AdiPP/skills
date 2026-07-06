---
name: python-backend
description: Generate Python aiohttp microservice backend code with Clean Architecture, SQLAlchemy, Kafka event streaming, dependency injection, and marshmallow schemas. Use when creating services, models, repositories, views, or event handlers.
---

# Python Backend Skill

Generic skill for generating aiohttp-based microservice backend code aligned with Clean Architecture, using SQLAlchemy 2.0 async, Kafka event streaming, and `dependency-injector` for IoC.

## Placeholder Convention

When applying this skill, replace:

| Placeholder | Format | Example |
|-------------|--------|---------|
| `{ProjectName}` | PascalCase — used in application/doc title | `Blog`, `Shop`, `CMS` |
| `{project-name}` | kebab-case — used in Kafka group, Docker names | `blog-api`, `shop-api` |
| `{Domain}` | PascalCase — bounded context name | `Employer`, `OpenAccount`, `Withdrawal` |
| `{Entity}` | PascalCase — entity / aggregate name | `Employer`, `Platform`, `DocumentTemplate` |
| `{Action}` | PascalCase verb — use case action | `Create`, `Update`, `Approve`, `Reject`, `Submit`, `Suspend`, `Rotate`, `Inquire`, `Delete`, `Verify`, `Confirm` |
| `{Suffix}` | PascalCase role for Service classes | `Creator`, `Updater`, `Registrar`, `Rotator`, `Checker`, `Generator`, `Inquirer`, `Processor`, `Factory` |
| `{stream_name}` | kebab-case — Kafka/event stream topic | `user-events`, `order-events` |
| `{event_name}` | dot-notation — event type identifier | `user.created`, `order.placed` |
| `{schema}` | snake_case — DB schema name | `public`, `app`, `inventory` |

## Project Init

Create a new service from the template:

```bash
uv init --python 3.12 {project-name}
cd {project-name}
```

Or copy from the existing boilerplate repository.

## Folder Structure

```
{project-name}/
├── app/                        # Business logic layer
│   ├── client/                 # Outbound HTTP clients (one per upstream service)
│   ├── commands/               # CLI command classes
│   ├── contracts/              # Abstract interfaces / protocols
│   ├── dto/                    # Data Transfer Objects (dataclasses)
│   ├── enums/                  # StrEnum/IntEnum definitions
│   ├── events/                 # Produced event definitions
│   ├── exceptions/             # Custom exception classes
│   ├── listeners/              # Consumed event handlers (Kafka listeners)
│   ├── models/                 # SQLAlchemy ORM models
│   ├── repositories/           # Repository classes
│   ├── services/               # Domain services (one subfolder per domain)
│   └── utils/                  # Shared utilities
├── containers/                 # DI sub-containers
│   ├── core.py                 # Core resources (DB engine, Kafka producer)
│   ├── repositories.py         # Repository wiring
│   ├── services.py             # Service wiring
│   ├── http_client.py          # HTTP client wiring
│   ├── queue.py                # Queue wiring
│   └── auth.py                 # Auth wiring
├── helpers/                    # Pure utility functions
│   ├── misc.py
│   ├── str.py
│   ├── json_serializer.py
│   └── settings_reader.py
├── migrations/                 # Alembic migration scripts
├── scripts/                    # One-off CLI scripts
├── seeders/                    # Database seeders
├── settings/                   # Configuration (env-based classes)
│   ├── __init__.py             # Exports all configs
│   ├── base.py                 # Base env setup
│   ├── app.py                  # App-level config
│   ├── db.py                   # Database config
│   ├── kafka.py                # Kafka config
│   ├── auth.py                 # Auth config
│   ├── http_client.py          # Outbound HTTP client configs
│   ├── event_stream.py         # Event stream config
│   ├── logs.py                 # Logging config
│   ├── telemetry.py            # OpenTelemetry config
│   └── ...                     # Per-upstream config files
├── tests/
│   ├── conftest.py             # Pytest fixtures
│   ├── config.py               # Test configuration
│   ├── test_api/               # API endpoint tests
│   ├── test_services/          # Service/repository tests
│   └── test_jobs/              # Queue job tests
├── web/                        # Web framework layer
│   ├── api/                    # Endpoint views + marshmallow schemas
│   │   ├── default/            # Healthcheck endpoints
│   │   ├── {domain1}/          # Domain-specific endpoints
│   │   └── ...
│   ├── container.py            # Top-level DI container
│   ├── create_app.py           # Web app factory
│   ├── create_worker_app.py    # Worker app factory (events only)
│   ├── routes.py               # Route definitions
│   ├── resources.py            # Async resource lifecycle generators
│   ├── initializers.py         # App bootstrap (healthcheck, event consumer)
│   ├── middleware.py            # HTTP middlewares
│   └── health_check.py         # Health check implementations
├── manage.py                   # CLI entry point (click)
├── pyproject.toml              # Project metadata + dependencies
├── Makefile                    # Dev commands
├── Dockerfile                  # Production image
├── alembic.ini                 # Alembic config
├── alembic/
│   ├── env.py                  # Alembic environment
│   └── script.py.mako          # Migration template
├── .env.example                # Environment variables template
├── docker-compose.yml          # Local dev services
└── docker-compose.local.yml    # Local overrides
```

---

## Key Patterns — Quick Reference

For full code templates, see [references/patterns.md](references/patterns.md).

| Pattern | Naming | Location |
|---------|--------|----------|
| API View | `{Action}{Entity}` — extends `JSONView`, `schema_body` / `schema_path` | `web/api/{domain}/views.py` |
| Schema | `{Action}{Entity}RequestSchema` / `{Action}{Entity}PathSchema` — marshmallow | `web/api/{domain}/schemas.py` |
| Model | `{Entity}` — extends `Base, UuidMixin, TimestampMixin` | `app/models/` |
| Repository | `{Entity}Repository` — extends `Repository` | `app/repositories/` |
| Domain Service | `{Entity}{Suffix}` — `final` class, single public method | `app/services/{entity}/{entity}_{suffix}.py` |
| DTO | `{Action}{Entity}DTO` — `@dataclass(frozen=True)` | `app/dto/` |
| Event (produced) | `{Domain}{Action}` — `@produced_event`, `@dataclass(frozen=True)` | `app/events/` |
| Event Listener | `{SubDomain}{EventName}Handler` — `@event_listener`, extends `EventHandler` | `app/listeners/` |
| Enum | `{Entity}Status`, `{Entity}Type` — `StrEnum` | `app/enums/` |
| Exception | `{Entity}{Error}Exception` — extends `ClientError` | `app/exceptions/` |
| Migration | UUID v7 primary key, `datetimes()`, indexes | `alembic/versions/` |
| HTTP Client | `{Upstream}Client` — per-upstream namespace | `app/client/{upstream}_client/` |
| Settings | `{Subsystem}Config` — static class with `env.*` | `settings/` |
| CLI Command | `{Action}Command` — auto-registered via `__init_subclass__` | `app/commands/` |
| Seeder | `{Entity}Seeder` — extends base seeder | `seeders/` |

### Suffixes (Domain Service roles)

| Suffix | Role | Example |
|--------|------|---------|
| `Creator` | Creates a new aggregate | `EmployerCreator` |
| `Updater` | Updates an existing aggregate | `CompanyUpdater` |
| `Registrar` | Registers an entity | `CustomerRegistrar`, `UserRegistrar` |
| `Rotator` | Regenerates / rotates value | `PlatformSecretRotator` |
| `Checker` | Uniqueness / preconditions | `UserUniquenessChecker` |
| `Deleter` | Removes an entity | `EmployerDeleter` |
| `Reviewer` | Review workflow step | `EmployerReviewer` |
| `Submitter` | Submit workflow step | `EmployerSubmitter` |
| `Generator` | Generates documents / files | `EmployerDocumentGenerator` |
| `Processor` | Processes files / data | `EmployeeApplicationFileProcessor`, `RiskAssessmentProcessor` |
| `Factory` | Constructs complex aggregates | `DocumentTemplateFactory`, `CommissionerFactory` |
| `Inquirer` | Read-only inquiry / lookup | `EmployeeApplicationAccountInquirer` |
| `Service` | Multi-purpose per aggregate | `BillingService`, `PaymentService`, `WebhookService` |
| `Confirmer` | Confirms / cancels operations | `WithdrawalConfirmer` |

### Transaction Management

- Use `@db_transaction` decorator on service methods for proper transaction scope
- Handlers should NOT have `@db_transaction` if calling separate service methods that each need their own transaction
- When passing entities between transactions, use IDs and re-fetch in the new transaction to avoid detached entity errors
- The `Repository` base class reuses an existing transaction context if present; otherwise it owns the session lifecycle + commit

### Event Streaming

- **Produced events**: decorate a frozen dataclass with `@produced_event(stream='{stream_name}', name='{event_name}')`
- **Consumed events**: decorate a handler class with `@event_listener(EventClass)` — must extend `EventHandler`
- Handlers are auto-registered via the global registry on import
- Use `event_dispatcher.publish(event)` to emit events, never `dispatch()`
- Event payloads from Kafka use camelCase; event classes use snake_case — handle mapping in `from_dict()`
- `event_streams()` returns the set of all subscribed topics for consumer initialization

### HTTP Client

- Never instantiate `aiohttp.ClientSession` directly; declare resources in `web/resources.py`
- Each upstream service gets an HTTP session resource + config in `settings/http_client.py`
- Client sessions managed as async generators via `cleanup_ctx`
- Token caching with thundering-herd lock pattern for OAuth2 client credentials

### Authentication

- Use `@auth` / `@require_auth` / `@get_auth` decorators from the auth library
- Three auth schemes: client tokens, internal service tokens, external API keys
- Auth config per scheme in `settings/auth.py`
- Token validation via JWT public key endpoints
- Inject authenticated payload into request via middleware

### Error Handling

- Custom exceptions extend `ClientError` from the aiohttp extensions
- Descriptive exception names ending with `Exception` (e.g., `NotificationNotFoundException`)
- Include relevant context in exception message (entity IDs)
- Validation errors raised via `ValidationError` (marshmallow or custom)
- `NotFoundError` for missing entities in repositories

### Testing

- **Framework**: pytest with `pytest-aiohttp` and `pytest-asyncio`
- **Pattern**: mock-heavy API tests — mock `session_factory`, mock repository returns
- **Fixtures**: `app` (web.Application), `app_client` (TestClient with headers), `db_pool`
- **Conventions**: `@pytest.mark.asyncio`, async def test functions
- **Structure**: `test_api/` (endpoint tests), `test_services/` (service/integration), `test_jobs/` (queue jobs)
- **Mocks**: `AsyncMock` for async calls, `MagicMock` for sync, `patch` context managers
- **Auth headers**: fixture helpers for generating JWT tokens with test secrets

### Seeders

Database seeders run via `python manage.py seed`:
- Each entity gets its own seeder class extending base seeder
- Registered in `database_seeder.py`
- Support for reference data seeding (cities, postal codes, banks, etc.)
- Run in dependency order

---

## Config Templates

See [references/config-templates.md](references/config-templates.md) for full templates:
- `pyproject.toml` (dependencies, build system)
- `settings/base.py` (env setup)
- `settings/app.py` (app config)
- `settings/db.py` (async PostgreSQL connection)
- `settings/kafka.py` (event stream config)
- `settings/auth.py` (auth scheme configs)
- `settings/http_client.py` (outbound HTTP client configs)
- `settings/logs.py` (logging config)
- `settings/telemetry.py` (OpenTelemetry config)
- `web/resources.py` (DB engine, Kafka producer, HTTP sessions)
- `web/container.py` (DI container wiring)
- `web/create_app.py` (web app factory)
- `manage.py` (CLI entry point)
- `Dockerfile` (multi-stage build)
- `alembic/env.py` (async migration environment)
- `Makefile` (dev commands)
- `.env.example`

See [references/rules.md](references/rules.md) for the full rules list and requirements.

Key rules:
- Python 3.12+ with type hints everywhere
- UUID v7 via `uuid_extensions.uuid7` for all primary keys
- `StrEnum` / `IntEnum` for status and type fields
- `@dataclass(frozen=True)` for DTOs and events
- All API endpoints prefixed with `/v1`
- snake_case for DB columns and Python attributes
- Models use `fillable` set + `fill(**kwargs)` for mass assignment
- Only domain exceptions should raise; validation errors handled by schemas
- Use Finder for read-only lookups via repository; not nullable returns
- Event handlers must be thin — delegate business logic to service layer
