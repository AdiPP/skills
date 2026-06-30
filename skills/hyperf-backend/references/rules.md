# Hyperf Backend — Rules & Requirements

---

## Required Packages (composer.json)

```json
"require": {
    "hyperf/hyperf": "~3.1.0",
    "hyperf/cache": "~3.1.0",
    "hyperf/crontab": "^3.1",
    "hyperf/guzzle": "~3.1.0",
    "hyperf/redis": "~3.1.0",
    "hyperf/tracer": "~3.1.0",
    "friendsofhyperf/cache": "^3.1",
    "menumbing/orm": "^1.0",
    "menumbing/event-stream": "^1.0",
    "menumbing/resource": "^1.0",
    "menumbing/graceful-process": "^1.0",
    "menumbing/health-check": "^1.0",
    "menumbing/async-queue": "^1.0",
    "menumbing/http-client": "^1.0",
    "menumbing/signature": "^1.0",
    "menumbing/tracer": "^1.0",
    "menumbing/auth": "^1.0",
    "menumbing/oauth2-resource-server": "^1.0"
}
```

---

## Graceful Process (MANDATORY)

Every Hyperf backend service **must** implement `menumbing/graceful-process` to ensure safe shutdown in Docker/Kubernetes environments.

### config/autoload/graceful_process.php

```php
<?php

declare(strict_types=1);

use function Hyperf\Support\env;

return [
    'timeout' => (int) env('GRACEFUL_PROCESS_TIMEOUT', 300),
    'max_wait_time' => (int) env('GRACEFUL_PROCESS_MAX_WAIT_TIME', 30),
];
```

Without this package, Swoole will abruptly close connections on SIGTERM, causing in-flight requests to fail.

---

## Rules

1. Always use `declare(strict_types=1);`
2. UUID v7 via `Str::orderedUuid()->toString()` for all primary keys
3. Use PHP 8.4 enums for status fields
4. snake_case for DB columns, camelCase for PHP properties
5. All API endpoints prefixed with `/v1`
6. Use `readonly class` for Actions and DTOs
7. Use `final class` for Services
8. Soft delete for operational data (`$table->softDeletes()`)
9. Always add `$table->datetimes()` (created_at, updated_at)
10. Cross-service communication only via Kafka events or API calls
11. No cross-schema joins in service code
12. Single transaction per service only
13. **MANDATORY** `menumbing/graceful-process` in every service
14. Cache access goes through `Hyperf\Cache\CacheManager` (`$this->cacheManager->getDriver()`), not direct `Hyperf\Redis\Redis`
15. Cache keys generated via dedicated key class (e.g., `{Name}Key::generate()`), prefixed with `{project_name}:`
16. Async queue: prefer `#[AsyncQueueMessage]` on `ListenerInterface::process()`. Use `Job` class only when there is no triggering event
17. Async queue channel = bounded context (`{domain}`); queue follows `{domain}.{action}.{service}-service`; routing key follows `{domain}.{action}`
18. Tune `maxAttempts` per use case: `1` for non-idempotent, `3` for typical idempotent work, `5` for flaky external dependencies
19. Default AMQP config uses `use_delayed_exchange: false` (TTL+DLQ) so no RabbitMQ plugin is required
20. HTTP client: never instantiate Guzzle directly and never call `$container->define()` for named clients. Declare each upstream as an entry under `http_client.http_clients` — `RegisterHttpClientListener` auto-registers the array key as a container id; consume it via `$container->get('{key}')` inside a Factory or via `#[HttpClient('{key}')]` property injection
21. Each upstream gets its own `HttpClient/{Upstream}Client/` namespace + `Factory/{Upstream}HttpClientFactory.php` + binding in `dependencies.php` (the wrapper class only — the named Guzzle client is auto-registered). Credentials/identifiers live in a per-upstream `config/autoload/{upstream}.php` (env-backed); Factory reads them via `ConfigInterface::get('{upstream}.{key}')`, NEVER `env()` inside Factory or Client
22. OAuth2 client_credentials tokens MUST be cached with a thundering-herd lock; expire `expires_in - 60s` to avoid edge expiry
23. Per-client `client_name` and `HttpClientTrace::TRACE_CAPTURE_ALL => true` are required so spans carry meaningful identifiers
24. Tracing toggles via `TRACER_ENABLE_*` env flags. Always exclude `/health*` and similar liveness routes via `excluded_routes`
25. Auth: every public Action MUST declare `#[Auth(guards: ...)]`. Use `oauth2` for user tokens, `client` for service-to-service tokens
26. Use stateless providers (`stateless_user`, `stateless_client`) when the service does NOT own the user/client aggregate; use model providers only in the Identity / aggregate-owner service
27. Per-endpoint authorization is expressed via routing `options: ['scope' => '...']`, NOT inside the action body
28. Inject the authenticated principal with `#[AuthUser(for: 'user'|'client', guards: [...])]` rather than fetching from the request manually
29. Scoped relation variants (e.g., `successful{Related}s()`, `enabled{Related}s()`) MUST chain `where(...)` on the base relation — never duplicate the relation definition
30. Default token validator is `stateless` (local JWT). Switch to `api` only when revocation/introspection must be synchronous
31. Each model relation lives in its own trait under `src/Relation/{Related}/`. Trait naming MUST encode the kind: `BelongsTo{Related}Trait`, `HasOne{Related}Trait`, `HasMany{Related}sTrait`, `BelongsToMany{Related}sTrait`. `MorphTo` is the only relation declared inline on the polymorphic owner model (not extracted to a trait)
32. **MANDATORY** `menumbing/health-check` in every service for Kubernetes liveness + readiness probes
33. Domain exceptions implement `ErrorInterface` (`getError()` + `getHint()`); use `ErrorResource` for JSON error responses
34. Consumed events use abstract base classes for event families sharing the same payload shape; concrete events add `#[ConsumedEvent]` annotation
35. Use `Finder` for read-only lookups — always `readonly class`, always `findOrFail` (never nullable returns)
36. Test directory is `test/` (not `tests/`); use Pest 2.x with `TestCase` base class; SQLite in-memory for database tests
