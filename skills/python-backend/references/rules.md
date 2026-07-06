# Python Backend — Rules & Requirements

---

## Required Packages (pyproject.toml)

```toml
[project]
name = "{project-name}"
version = "0.1.0"
description = ""
readme = "README.md"
requires-python = ">=3.12"
dependencies = [
    "aiohttp>=3.9,<4",
    "aiocache>=0.11.1,<1",
    "aioprometheus>=21.9.1,<22",
    "alembic>=1.17.2",
    "asyncpg>=0.31.0",
    "click>=7.1,<8",
    "dependency-injector>=4.48.3",
    "environs>=9.3.2,<10",
    "marshmallow>=3,<4",
    "opentelemetry-instrumentation-asyncpg==0.41b0",
    "opentelemetry-instrumentation-redis==0.41b0",
    "psycopg2-binary>=2.9.11",
    "pydantic>=2.12.5,<3",
    "redis~=4.5.1",
    "sentry-sdk>=1.0.0,<2",
    "sqlalchemy>=2.0,<3.0",
    "sqlalchemy-utils>=0.42.1",
    "uuid7>=0.1.0,<1",
    "uvloop>=0.15.2,<1",
    "yarl>=1,<=1.12.1",
]
```

Dev dependencies:

```toml
[dependency-groups]
dev = [
    "pytest==7.4.1",
    "pytest-aiohttp==1.0.5",
    "pytest-asyncio==0.21.0",
    "pytest-cov>=7.0.0",
    "flake8>=7,<8",
    "mypy<1",
    "isort>=5,<6",
    "pre-commit>=2.10.1,<3",
    "add-trailing-comma>=2.4.0,<3",
    "autoflake>=2.2.1,<3",
    "autopep8>=1,<3",
    "freezegun>=1,<2",
]
```

---

## Rules

1. Python 3.12+ with full type hints on every function signature — never omit type annotations.

2. UUID v7 via `uuid_extensions.uuid7` for all primary keys. Do not use auto-increment integers or UUID v4.

3. Use `StrEnum` / `IntEnum` from Python standard library for all status and type fields. Never use string literals or boolean flags for multi-state fields.

4. Use `@dataclass(frozen=True)` for all DTOs and events. Immutability prevents accidental mutation across service boundaries.

5. All API endpoints must be prefixed with `/v1` — never omit version prefix.

6. snake_case for DB column names, Python attributes, file names, and directory names.

7. Model classes inherit `Base, UuidMixin, TimestampMixin` in that exact order. `Base` must always be the first parent.

8. Every model defines a `fillable` class-level set of attribute names that are allowed for mass assignment via `fill(**kwargs)`. Never mass-assign directly — use `fill()` or explicit property setters.

9. Every model provides a `to_dict()` method for serialization using `json_safe()` to handle UUIDs, datetimes, and other non-serializable types.

10. Use `initialize()` static factory method or direct constructor `cls(**data)` for creating model instances. Prefer `initialize()` when side effects (defaults, computed fields) are needed.

11. Repository classes extend the `Repository` base class from the database library. Never call session operations directly in services or views.

12. Repository `find_or_fail()` methods raise `NotFoundError` on missing entities — never return `None`. For nullable lookups, use `find_one_by_{field}()` which returns `Optional[{Entity}]`.

13. Service classes use noun + suffix naming (`{Entity}{Suffix}`) — not verb-first or `Service`-suffixed names when a more specific suffix exists.

14. Each service class should have exactly one public method in most cases (single responsibility). Compose multiple services rather than creating a monolithic service.

15. Use `@inject` + `Provide[Container.services.xxx]` for DI in views and event handlers. Never instantiate services manually.

16. Use `@db_transaction` decorator on service methods that need transactional guarantees. Handlers that coordinate multiple service calls should NOT use `@db_transaction`.

17. When passing entities between transaction scopes, use entity IDs and re-fetch in the new scope. Never pass detached SQLAlchemy entities between transactions.

18. API views extend `JSONView` (from the aiohttp extensions library). Define `schema_body`, `schema_path`, `schema_query` as class attributes for input validation.

19. Use marshmallow schemas for request validation. Define endpoint-specific schemas in `web/api/{domain}/schemas.py`.

20. Use `@auth` / `@require_auth` / `@get_auth` decorators for endpoint protection. Never manually parse JWT tokens in views.

21. Event definitions use `@produced_event(stream='...', name='...')` decorator on a frozen dataclass extending `BaseEvent`. Never emit events without a typed event class.

22. Event listeners use `@event_listener(EventClass)` decorator and extend `EventHandler`. Keep handlers thin — delegate business logic to service layer.

23. Use `event_dispatcher.publish(event)` to emit events, not `dispatch()`. Events must be published only after all critical operations have successfully completed.

24. Custom exceptions extend `ClientError` (from aiohttp extensions). Name descriptively with `Exception` suffix (e.g., `EntityNotFoundException`) and include relevant context in the message.

25. Validation errors should be raised via schema validation or explicit `ValidationError` — never let invalid data reach the service layer.

26. Settings classes are static classes with `env.*` calls from `environs`. Never use `os.getenv()` directly. Never hardcode environment-dependent values.

27. Database configuration uses asyncpg driver with SQLAlchemy 2.0 async API. Pool settings (min/max size, timeout, recycle) must be env-configurable.

28. Resources (DB engine, Kafka producer, HTTP sessions) are async generators managed via `cleanup_ctx` or resource providers. Never create connections as module-level globals.

29. Healthcheck endpoints (`/health/readiness`, `/health/liveness`) are mandatory for every service. Readiness checks database connectivity; liveness is a simple HTTP 200.

30. Event stream consumers run in a separate worker process — never in the API process. Use `manage.py start-worker` for workers.

31. Seeders inherit from the base seeder class and are registered in `database_seeder.py`. Run in dependency order.

32. Tests use `@pytest.mark.asyncio` for async test functions. Mock repositories and external dependencies — never hit real databases or Kafka in tests.

33. Test fixtures mock `session_factory` and use `AsyncMock` for async repository methods. Use `MagicMock` for synchronous mocks.

34. Container `wiring_config` must include both `web.api.*` modules and `app.listeners` module for proper DI resolution.

35. Alembic migrations use async environment with PostgreSQL. Always set `include_schemas=False` in `env.py` to prevent accidentally dropping tables from other microservice schemas.

36. Each migration should focus on a single schema change. Create separate migration files for each table in dependency order.

37. Add indexes for frequently queried columns. Naming convention: `ix_{table_name}_{column_name}`.

38. Cross-service communication must only happen via Kafka events or HTTP API calls. Never share databases between services.

39. HTTP client sessions are defined as resources in `web/resources.py` — one per upstream service. Never create `ClientSession` inline in service code.

40. Tracing middleware is mandatory for API and event stream paths. Configure via settings and env flags.

41. Multiple event listeners can be registered for the same event, but each listener class handles one event type.

42. Prefer service composition over inheritance. Inject dependencies via constructor, not via inheritance.

43. CLI commands auto-register via `__init_subclass__` mechanism — ensure each command is importable from `app/commands/`.

44. Use `uv` for package management (`uv sync`, `uv add`, `uv lock`). Do not use `pip` directly.
