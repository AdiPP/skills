# Hyperf Backend — Code Patterns

Full code templates for all patterns referenced in SKILL.md.

---

## Action (Use Case Handler)

Naming: `{Action}{Name}Action` — single-purpose class invoked via `__invoke()`.

```php
<?php

declare(strict_types=1);

namespace {ProjectName}\{Domain}\Action;

use Hyperf\HttpServer\Annotation\Controller;
use Hyperf\HttpServer\Annotation\PostMapping;
use HyperfExtension\Auth\Annotations\Auth;
use Menumbing\Resource\Annotation\WithResource;
use {ProjectName}\{Domain}\DTO\{Action}{Name}DTO;
use {ProjectName}\{Domain}\Request\{Action}{Name}Request;
use {ProjectName}\{Domain}\Service\{Name}\{Name}{Suffix};

#[Controller]
readonly class {Action}{Name}Action
{
    public function __construct(
        private {Name}{Suffix} $service,
    ) {}

    #[PostMapping(path: '/v1/{resource}')]
    #[Auth(guards: ['oauth2'])]
    #[WithResource(statusCode: 201)]
    public function __invoke({Action}{Name}Request $request): mixed
    {
        $payload = $request->validated();

        return $this->service->{action}(
            new {Action}{Name}DTO(...$payload),
        );
    }
}
```

---

## Model

```php
<?php

declare(strict_types=1);

namespace {ProjectName}\{Domain}\Model;

use Hyperf\Database\Model\Concerns\HasUuids;
use Hyperf\Stringable\Str;
use Menumbing\Orm\Contract\HasDomainEventInterface;
use Menumbing\Orm\Model;
use Menumbing\Orm\Trait\HasDomainEvent;

/**
 * @property string $id
 * @property string $name
 * @property Status $status
 * @property \Carbon\Carbon $createdAt
 * @property \Carbon\Carbon $updatedAt
 */
class {Name} extends Model implements HasDomainEventInterface
{
    use HasUuids, HasDomainEvent;

    protected ?string $table = '{table_name}';

    protected array $casts = [
        'status' => Status::class,
    ];

    public static function initialize(/* params */): static
    {
        $static = new static();
        $static->id = Str::orderedUuid()->toString();
        // set properties...

        return $static;
    }

    public function patch(/* fields */): void
    {
        $this->name = $name;
        $this->status = $status;
        // partial update — never resets unmentioned fields
    }
}
```

---

## Finder

Naming: `{Entity}Finder` — read-only lookup layer between Service and Repository. Always `readonly class`, always `findOrFail` (never nullable).

```php
<?php

declare(strict_types=1);

namespace {ProjectName}\{Domain}\Finder;

use Hyperf\Database\Model\ModelNotFoundException;
use Hyperf\Di\Annotation\Inject;
use Menumbing\Orm\Model;
use {ProjectName}\{Domain}\Model\{Name};
use {ProjectName}\{Domain}\Repository\{Name}Repository;

readonly class {Name}Finder
{
    #[Inject]
    protected {Name}Repository $repository;

    public function findOrFail(string $id): Model | {Name}
    {
        if (null !== $model = $this->repository->findById($id)) {
            return $model;
        }

        throw new ModelNotFoundException(
            sprintf('{Name} with id \'%s\' not found.', $id)
        );
    }

    public function findBy{Field}OrFail(string $value): Model | {Name}
    {
        if (null !== $model = $this->repository->findOneBy(['{field}' => $value])) {
            return $model;
        }

        throw new ModelNotFoundException(
            sprintf('{Name} with {field} \'%s\' not found.', $value)
        );
    }
}
```

---

## Consumed Event (Inbound Kafka)

Service consumes external Kafka events via `#[ConsumedEvent]`. Use abstract base classes for event families sharing the same payload shape.

### Abstract Base Event

```php
<?php

declare(strict_types=1);

namespace {ProjectName}\{Domain}\Event\{SubDomain}\{EventFamily};

readonly abstract class Abstract{EventFamily}Event
{
    public function __construct(
        public string $platformId,
        public array  $entity,
    ) {}

    abstract public function getStatus(): string;
}
```

### Concrete Consumed Event

```php
<?php

declare(strict_types=1);

namespace {ProjectName}\{Domain}\Event\{SubDomain}\{EventFamily};

use Menumbing\EventStream\Annotation\ConsumedEvent;

#[ConsumedEvent(
    stream: '{sub-domain}.{entity}-events',
    name  : '{sub-domain}.{entity}.{action}',
)]
readonly class {Entity}{Action} extends Abstract{EventFamily}Event
{
}
```

### Consumed Event Resolver

Utility to reflect `#[ConsumedEvent]` metadata at runtime:

```php
<?php

declare(strict_types=1);

namespace {ProjectName}\{Domain}\Shared\EventStream;

use Menumbing\EventStream\Annotation\ConsumedEvent;
use ReflectionClass;

class ConsumedEventResolver
{
    public static function getMetadata(string $class): ?ConsumedEvent
    {
        $reflection = new ReflectionClass($class);
        $attributes = $reflection->getAttributes(ConsumedEvent::class);

        return $attributes[0]?->newInstance();
    }

    public static function getName(string $class): ?string
    {
        return self::getMetadata($class)?->name;
    }

    public static function getStream(string $class): ?string
    {
        return self::getMetadata($class)?->stream;
    }
}
```

### Consumed Event Listener

```php
<?php

declare(strict_types=1);

namespace {ProjectName}\{Domain}\Listener\{SubDomain};

use Hyperf\Di\Annotation\Inject;
use Hyperf\Event\Annotation\Listener;
use Hyperf\Event\Contract\ListenerInterface;
use Psr\Container\ContainerInterface;
use {ProjectName}\{Domain}\Event\{SubDomain}\{EventFamily}\{Entity}{Action};
use {ProjectName}\{Domain}\Finder\{Entity}Finder;
use {ProjectName}\{Domain}\Repository\{Entity}Repository;
use {ProjectName}\{Domain}\Service\{Name}\{Name}{Suffix};

#[Listener]
class {Entity}{Action}Listener implements ListenerInterface
{
    #[Inject]
    protected ContainerInterface $container;

    public function listen(): array
    {
        return [{Entity}{Action}::class];
    }

    public function process(object $event): void
    {
        if (!$event instanceof {Entity}{Action}) {
            return;
        }

        $service = $this->container->get({Name}{Suffix}::class);
        $finder = $this->container->get({Entity}Finder::class);
        $repository = $this->container->get({Entity}Repository::class);

        // Find client/entity by platform_id from event, return early if null
        // Create domain object via service
        // Map payload from event to DTO
    }

    private function mapPayload({Entity}{Action} $event): array
    {
        return [
            'id'     => $event->entity['id'],
            'status' => $event->getStatus(),
            // map fields from event payload
        ];
    }
}
```

---

## Error Handling

### ErrorInterface

Domain exceptions implement this interface for structured error responses:

```php
<?php

declare(strict_types=1);

namespace {ProjectName}\{Domain}\Exception;

use BackedEnum;
use Throwable;

interface ErrorInterface extends Throwable
{
    public function getError(): string|BackedEnum;
    public function getHint(): ?string;
}
```

### Domain Exception

```php
<?php

declare(strict_types=1);

namespace {ProjectName}\{Domain}\Exception\{SubDomain};

use {ProjectName}\{Domain}\Constant\ErrorCode;
use {ProjectName}\{Domain}\Exception\ErrorInterface;

class {Entity}NotFoundException extends \RuntimeException implements ErrorInterface
{
    public function __construct(string $identifier)
    {
        parent::__construct(sprintf('{Name} with id \'%s\' not found.', $identifier));
    }

    public function getError(): string|BackedEnum
    {
        return ErrorCode::NOT_FOUND;
    }

    public function getHint(): ?string
    {
        return 'Check the identifier and try again.';
    }
}
```

### ErrorResource

Renders RFC-compliant JSON error responses:

```php
<?php

declare(strict_types=1);

namespace {ProjectName}\{Domain}\Resource;

use Hyperf\Resource\Json\JsonResource;
use {ProjectName}\{Domain}\Constant\ErrorCode;
use {ProjectName}\{Domain}\Exception\ErrorInterface;
use BackedEnum;
use Throwable;

class ErrorResource extends JsonResource
{
    public ?string $wrap = null;

    public function __construct(ErrorInterface|Throwable $resource)
    {
        parent::__construct($resource);
    }

    public function toArray(): array
    {
        $code = $this->getStatusCode();

        return array_filter([
            'error'               => $this->getError($this->resource),
            'error_description'   => $this->resource->getMessage(),
            'code'                => $code,
            'hint'                => $this->resource instanceof ErrorInterface ? $this->resource->getHint() : null,
            'debug'               => $this->getDebugInfo(),
        ]);
    }

    public function getStatusCode(): int
    {
        return match (true) {
            $this->resource instanceof \Hyperf\Validation\ValidationException => 422,
            $this->resource instanceof \HyperfExtension\Auth\Exceptions\AuthenticationException => 401,
            $this->resource instanceof \HyperfExtension\Auth\Exceptions\AuthorizationException => 403,
            $this->resource instanceof \Hyperf\HttpMessage\Exception\HttpException => $this->resource->getStatusCode(),
            default => 500,
        };
    }

    protected function getError(ErrorInterface|Throwable $error): string|BackedEnum
    {
        return match (true) {
            $error instanceof ErrorInterface => $error->getError(),
            default => ErrorCode::INTERNAL_ERROR,
        };
    }
}
```

### AppExceptionHandler

```php
<?php

declare(strict_types=1);

namespace {ProjectName}\{Domain}\Exception\Handler;

use Hyperf\ExceptionHandler\ExceptionHandler;
use {ProjectName}\{Domain}\Resource\ErrorResource;
use Swow\Psr7\Message\ResponsePlusInterface;
use Throwable;

class AppExceptionHandler extends ExceptionHandler
{
    public function handle(Throwable $throwable, ResponsePlusInterface $response)
    {
        $this->stopPropagation();

        $resource = new ErrorResource($throwable);

        return $resource
            ->toResponse()
            ->withStatus($resource->getStatusCode());
    }

    public function isValid(Throwable $throwable): bool
    {
        return true;
    }
}
```

Exception handler chain (priority order in `exceptions.php`):
1. `RespondTraceIdExceptionHandler` — attaches trace ID to response
2. `OAuth2ServerExceptionHandler` — OAuth2-specific errors
3. `AppExceptionHandler` — catch-all for all remaining throwables

---

## Custom Auth Guard (e.g., Authentik JWKS)

When the service authenticates against an external IdP with JWKS:

### Guard

```php
<?php

declare(strict_types=1);

namespace {ProjectName}\{Domain}\Auth\{Provider};

use Firebase\JWT\JWT;
use Firebase\JWT\JWK;
use Hyperf\HttpServer\Contract\RequestInterface;
use HyperfExtension\Auth\Contracts\AuthenticatableInterface;
use HyperfExtension\Auth\Contracts\GuardInterface;
use HyperfExtension\Auth\GenericUser;

class {Provider}Guard implements GuardInterface
{
    private ?AuthenticatableInterface $user = null;

    public function __construct(
        protected RequestInterface $request,
        protected {Provider}JwksFetcher $jwksFetcher,
    ) {}

    public function check(): bool { return $this->user() !== null; }
    public function guest(): bool { return $this->user() === null; }
    public function id(): mixed { return $this->user()?->getAuthIdentifier(); }

    public function user(): ?AuthenticatableInterface
    {
        if ($this->user !== null) {
            return $this->user;
        }

        $token = $this->extractBearerToken();
        if ($token === null) {
            return null;
        }

        $claims = $this->validateAndParseClaims($token);
        $this->user = new GenericUser($claims);

        return $this->user;
    }

    public function validate(array $credentials = []): bool { return $this->user() !== null; }
    public function setUser(AuthenticatableInterface $user): static { $this->user = $user; return $this; }

    private function extractBearerToken(): ?string { /* extract from Authorization header */ }
    private function validateAndParseClaims(string $rawToken): array { /* JWT decode via JWKS */ }
}
```

### JWKS Fetcher (with caching)

```php
<?php

declare(strict_types=1);

namespace {ProjectName}\{Domain}\Auth\{Provider};

use FriendsOfHyperf\Cache\Facade\Cache;
use GuzzleHttp\ClientInterface;

class {Provider}JwksFetcher
{
    private const CACHE_KEY = '{provider}_jwks';

    public function __construct(
        protected ClientInterface $httpClient,
        protected string $jwksUrl,
        protected int $cacheTtl = 3600,
    ) {}

    public function getKey(string $kid): array
    {
        $jwks = $this->fetchCached();
        foreach ($jwks['keys'] ?? [] as $key) {
            if (($key['kid'] ?? null) === $kid) {
                return $key;
            }
        }
        // Key not found — refresh cache once (key rotation)
        $jwks = $this->fetch();
        foreach ($jwks['keys'] ?? [] as $key) {
            if (($key['kid'] ?? null) === $kid) {
                return $key;
            }
        }
        throw new \RuntimeException("JWK not found for kid: {$kid}");
    }

    private function fetchCached(): array
    {
        return Cache::store('memory')->remember(self::CACHE_KEY, $this->cacheTtl, fn () => $this->fetch());
    }

    private function fetch(): array { /* HTTP GET + cache set */ }
}
```

### Factory

```php
<?php

declare(strict_types=1);

namespace {ProjectName}\{Domain}\Factory;

use GuzzleHttp\Client;
use Hyperf\Contract\ConfigInterface;
use Psr\Container\ContainerInterface;
use {ProjectName}\{Domain}\Auth\{Provider}\{Provider}Guard;
use {ProjectName}\{Domain}\Auth\{Provider}\{Provider}JwksFetcher;

class {Provider}GuardFactory
{
    public function __invoke(ContainerInterface $container): {Provider}Guard
    {
        $config = $container->get(ConfigInterface::class);

        $jwksFetcher = new {Provider}JwksFetcher(
            httpClient: new Client(),
            jwksUrl   : $config->get('{provider lowercase}.jwks_url'),
            cacheTtl  : $config->get('{provider lowercase}.jwks_cache_ttl', 3600),
        );

        return new {Provider}Guard(
            request    : $container->get(RequestInterface::class),
            jwksFetcher: $jwksFetcher,
        );
    }
}
```

Wiring in `dependencies.php`:
```php
return [
    {ProjectName}\{Domain}\Auth\{Provider}\{Provider}Guard::class
        => {ProjectName}\{Domain}\Factory\{Provider}GuardFactory::class,
];
```

---

## JsonResourceStrategy Override

Exclude `Throwable` from default resource serialization so exceptions route to `ErrorResource` instead:

```php
<?php

declare(strict_types=1);

namespace {ProjectName}\{Domain}\Resource;

use Menumbing\Resource\Strategy\JsonResourceStrategy as MenumbingJsonResourceStrategy;
use Throwable;

class JsonResourceStrategy extends MenumbingJsonResourceStrategy
{
    public function supports(mixed $resource): bool
    {
        return !$this->getOrigin($resource) instanceof Throwable;
    }
}
```

---

## Console Command

Naming: `{Action}{Entity}Console` — thin wrapper over `HyperfCommand` for interactive or scripted CLI operations.

```php
<?php

declare(strict_types=1);

namespace {ProjectName}\{Domain}\Console;

use Hyperf\Command\Annotation\Command;
use Hyperf\Command\Command as HyperfCommand;
use Hyperf\Di\Annotation\Inject;
use Symfony\Component\Console\Input\InputArgument;

#[Command]
class {Action}{Entity}Console extends HyperfCommand
{
    protected ?string $name = '{entity}:{action}';

    #[Inject]
    protected {Entity}Repository $repository;

    public function handle(): void
    {
        $id = $this->input->getArgument('id');
        // perform action...
        $this->output->success('Done.');
    }

    protected function getArguments(): array
    {
        return [
            ['id', InputArgument::REQUIRED, 'The id of the {entity}'],
        ];
    }
}
```

---

## Repository

```php
<?php

declare(strict_types=1);

namespace {ProjectName}\{Domain}\Repository;

use Menumbing\Orm\Annotation\AsRepository;
use Menumbing\Orm\Repository;
use {ProjectName}\{Domain}\Model\{Name};

#[AsRepository(modelClass: {Name}::class)]
class {Name}Repository extends Repository
{
    public function findByXxx(string $value): ?{Name}
    {
        return $this->newQuery()
            ->where('xxx', $value)
            ->first();
    }
}
```

---

## Domain Service

Naming: `{Name}{Suffix}` where `{Suffix}` describes the role.

| Suffix | Role | Example |
|--------|------|---------|
| `Creator` | Creates a new aggregate | `UserCreator`, `ProgramCreator` |
| `Updater` | Updates an existing aggregate | `ProgramUpdater` |
| `Verifier` | Validates / verifies state | `OTPVerifier`, `CompanyVerifier` |
| `Approver` / `Rejecter` | Workflow transitions | `ProposalApprover`, `PositionRejecter` |
| `Rotator` | Regenerates / rotates value | `ClientSecretRotator` |
| `Checker` | Uniqueness / preconditions | `UserUniquenessChecker` |
| `Resolver` | Read with caching / lookup | `PermissionResolver` |
| `Manager` | Multi-purpose for one aggregate | `UserPinManager` |
| `Invalidator` / `Requester` | Domain-specific verbs | `OTPInvalidator`, `OTPRequester` |

One method per service is preferred (single responsibility); the method name is the lowercase verb (`create`, `update`, `verify`, `approve`, `rotate`, etc.).

```php
<?php

declare(strict_types=1);

namespace {ProjectName}\{Domain}\Service\{Name};

use Hyperf\DbConnection\Annotation\Transactional;

final class {Name}{Suffix}
{
    public function __construct(
        private readonly {Name}Repository $repository,
    ) {}

    #[Transactional]
    public function {action}({Action}{Name}DTO $dto): {Name}
    {
        $entity = {Name}::initialize(/* from DTO */);

        return $this->repository->save($entity);
    }
}
```

---

## DTO

Naming: `{Action}{Name}DTO` — verb-first describes intent. One DTO per use case, immutable (`readonly` properties).

```php
<?php

declare(strict_types=1);

namespace {ProjectName}\{Domain}\DTO;

class {Action}{Name}DTO
{
    public function __construct(
        public readonly string $field1,
        public readonly string $field2,
    ) {}
}
```

---

## Domain Event (Kafka)

```php
<?php

declare(strict_types=1);

namespace {ProjectName}\{Domain}\Event;

use Menumbing\EventStream\Annotation\ProducedEvent;

#[ProducedEvent(
    stream: '{service}-events',
    name  : '{service}.{entity}-{action}',
)]
readonly class {Entity}{Action}
{
    public function __construct(
        public array $payload,
    ) {}

    public static function createFrom{Entity}({Entity} $entity): self
    {
        return new self(payload: $entity->toArray());
    }
}
```

---

## Event Listener

```php
<?php

declare(strict_types=1);

namespace {ProjectName}\{Domain}\Listener;

use Hyperf\Di\Annotation\Inject;
use Hyperf\Event\Annotation\Listener;
use Hyperf\Event\Contract\ListenerInterface;
use Psr\Container\ContainerInterface;

#[Listener]
class {Name}Listener implements ListenerInterface
{
    #[Inject]
    protected ContainerInterface $container;

    public function listen(): array
    {
        return [{SourceEvent}::class];
    }

    public function process(object $event): void
    {
        if (!$event instanceof {SourceEvent}) {
            return;
        }

        $service = $this->container->get({Service}::class);
        // Process...
    }
}
```

---

## Async Queue

Async queue offloads work from the main request lifecycle. The skill follows `menumbing/async-queue` (enhanced wrapper over `hyperf/async-queue`) with **AMQP (RabbitMQ)** as the default driver.

Prefer the **Listener + `#[AsyncQueueMessage]`** pattern for event-triggered background work.

### Pattern A — Listener with `#[AsyncQueueMessage]` (preferred)

```php
<?php

declare(strict_types=1);

namespace {ProjectName}\{Domain}\Listener;

use Hyperf\AsyncQueue\Annotation\AsyncQueueMessage;
use Hyperf\Event\Annotation\Listener;
use Hyperf\Event\Contract\ListenerInterface;
use Psr\Container\ContainerInterface;
use {ProjectName}\{Domain}\Event\{SourceEvent};
use {ProjectName}\{Domain}\Service\{Name}\{Name}{Suffix};

#[Listener]
class {Action}{Name}Listener implements ListenerInterface
{
    public function __construct(protected ContainerInterface $container) {}

    public function listen(): array
    {
        return [{SourceEvent}::class];
    }

    #[AsyncQueueMessage(maxAttempts: 5)]
    public function process(object $event): void
    {
        if (!$event instanceof {SourceEvent}) {
            return;
        }

        $service = $this->container->get({Name}{Suffix}::class);
        // Background work — heavy I/O, external API call, file operations, etc.
    }
}
```

Tune `maxAttempts` per use case:
- `1` — non-idempotent or already-guarded operations
- `3` — typical idempotent work (notifications, email)
- `5` — flaky external services that recover

### Pattern B — Job class (when no triggering event)

```php
<?php

declare(strict_types=1);

namespace {ProjectName}\{Domain}\Job;

use Hyperf\AsyncQueue\Job;

class {Action}{Name}Job extends Job
{
    protected int $maxAttempts = 3;

    public function __construct(
        protected string $entityId,
    ) {}

    public function handle(): void
    {
        // Resolve dependencies via ApplicationContext, then perform work.
    }

    public function fail(\Throwable $e): void
    {
        // Optional: called after all retries exhausted.
    }
}
```

Dispatch:

```php
use function Hyperf\AsyncQueue\dispatch;

dispatch(new {Action}{Name}Job($entityId));                 // immediate
dispatch(new {Action}{Name}Job($entityId), delay: 60);      // delayed 60s
dispatch(new {Action}{Name}Job($entityId), pool: 'mailer'); // specific pool
```

---

## HTTP Client

Outbound HTTP integrations follow `menumbing/http-client` (config-driven Guzzle wrapper) plus `menumbing/tracer` for automatic distributed tracing.

### Layout

```
src/
├── HttpClient/
│   └── {Upstream}Client/
│       └── {Upstream}HttpClient.php
├── Factory/
│   └── {Upstream}HttpClientFactory.php
└── Middleware/
    └── HttpClient/
        └── {Upstream}Client/
            └── ErrorHandlerMiddleware.php
```

### Client Class

The wrapper consumes the auto-registered named Guzzle client by container id (the array key from `http_client.php`). Credentials/identifiers are typed constructor params from a per-upstream config file, never `env()`.

```php
<?php

declare(strict_types=1);

namespace {ProjectName}\{Domain}\HttpClient\{Upstream}Client;

use GuzzleHttp\ClientInterface;
use GuzzleHttp\RequestOptions;
use Psr\Http\Message\ResponseInterface;

class {Upstream}HttpClient
{
    public function __construct(
        protected ClientInterface $client,
        protected string          $clientId,
        protected string          $clientSecret,
    ) {}

    public function get{Resource}(string $id): array
    {
        $response = $this->client->get(
            sprintf('/v1/{resource}/%s', $id),
            [
                RequestOptions::HEADERS => [
                    'X-Client-Id'   => $this->clientId,
                    'Authorization' => sprintf('Bearer %s', $this->auth->getAccessToken()),
                ],
            ],
        );

        return $this->decode($response);
    }

    private function decode(ResponseInterface $response): array
    {
        return json_decode($response->getBody()->getContents(), true);
    }
}
```

### Per-upstream Config File (`config/autoload/{upstream}.php`)

```php
<?php

declare(strict_types=1);

use function Hyperf\Support\env;

return [
    'client_id'     => env('{UPSTREAM}_CLIENT_ID'),
    'client_secret' => env('{UPSTREAM}_CLIENT_SECRET'),
];
```

### Factory — Variant A (upstream with its own credentials)

```php
<?php

declare(strict_types=1);

namespace {ProjectName}\{Domain}\Factory;

use Hyperf\Contract\ConfigInterface;
use Psr\Container\ContainerInterface;
use {ProjectName}\{Domain}\HttpClient\{Upstream}Client\{Upstream}HttpClient;

class {Upstream}HttpClientFactory
{
    public function __invoke(ContainerInterface $container, array $parameters = []): {Upstream}HttpClient
    {
        $config = $container->get(ConfigInterface::class);

        return new {Upstream}HttpClient(
            client      : $container->get('{upstream}_api_client'),
            clientId    : $config->get('{upstream}.client_id'),
            clientSecret: $config->get('{upstream}.client_secret'),
        );
    }
}
```

### Factory — Variant B (upstream that only needs a Bearer minted by another client)

```php
class {Upstream}HttpClientFactory
{
    public function __invoke(ContainerInterface $container, array $parameters = []): {Upstream}HttpClient
    {
        return new {Upstream}HttpClient(
            client        : $container->get('{upstream}_api_client'),
            authHttpClient: $container->get(AuthHttpClient::class),
        );
    }
}
```

### Wiring (`config/autoload/dependencies.php`)

Only the wrapper class is bound here. The underlying Guzzle client is auto-registered by the listener.

```php
return [
    {ProjectName}\{Domain}\HttpClient\{Upstream}Client\{Upstream}HttpClient::class
        => {ProjectName}\{Domain}\Factory\{Upstream}HttpClientFactory::class,
];
```

### Alternative: `#[HttpClient]` property injection (no Factory)

When an upstream needs no extra credentials and you only want a thin domain wrapper, use the `Menumbing\HttpClient\Annotation\HttpClient` property attribute.

```php
use GuzzleHttp\ClientInterface;
use Menumbing\HttpClient\Annotation\HttpClient;

final class {Name}Service
{
    #[HttpClient('{upstream}_api_client')]
    protected ClientInterface $client;

    public function call(): array
    {
        $response = $this->client->get('/v1/{resource}');
        return json_decode($response->getBody()->getContents(), true);
    }
}
```

Use this only for trivial cases; prefer the Factory + wrapper class pattern when the upstream needs typed config, response decoding, or shared error handling.

### Token Caching (Bearer auth upstream)

For OAuth2 client_credentials, cache the token with a thundering-herd lock — never request a token per call.

```php
use FriendsOfHyperf\Cache\Facade\Cache;
use function FriendsOfHyperf\Lock\lock;

public function getAccessToken(): string
{
    $cacheKey = '{upstream}_oauth_access_token';

    if ($token = Cache::get($cacheKey)) {
        return $token;
    }

    return lock($cacheKey, 10)->block(5, function () use ($cacheKey) {
        if ($cached = Cache::get($cacheKey)) {
            return $cached;
        }

        $token = $this->fetchAccessToken();
        Cache::set($cacheKey, $token['access_token'], ($token['expires_in'] ?? 3600) - 60);

        return $token['access_token'];
    });
}
```

---

## Authentication & Authorization

Two guard archetypes:

| Guard | Token subject | Provider | Use case |
|-------|---------------|----------|----------|
| `oauth2` (or `oauth2_user`) | End user | `stateless_user` or `users` | User-facing endpoints |
| `client` (or `oauth2_client`) | Machine / first-party service | `stateless_client` or `clients` | M2M endpoints |

### Annotations on Action

```php
// User endpoint with scope
#[PostMapping(
    path: '/v1/oauth2/me/pin-set',
    options: ['scope' => 'pin:set_verified'],
)]
#[Auth(guards: 'oauth2')]
#[AuthUser(for: 'user')]
#[WithResource]
public function set(?User $user, {Action}{Name}Request $request): mixed { /* ... */ }

// Service-to-service endpoint
#[PostMapping(path: '/v1/{resource}')]
#[Auth(guards: ['client'])]
#[WithResource(statusCode: 201)]
public function __invoke({Action}{Name}Request $request): mixed { /* ... */ }
```

---

## Relation Trait

Each relation lives in its own trait under `src/Relation/{Related}/`.

### BelongsTo

```php
<?php

declare(strict_types=1);

namespace {ProjectName}\{Domain}\Relation\{Related};

use Menumbing\Orm\Relation\BelongsTo;
use Menumbing\Orm\Trait\HasRelations;
use {ProjectName}\{Domain}\Model\{Related};

/**
 * @property-read {Related} ${related}
 * @property string         ${related}Id
 */
trait BelongsTo{Related}Trait
{
    use HasRelations;

    public function {related}(): BelongsTo
    {
        return $this->belongsTo({Related}::class);
    }
}
```

### HasOne

```php
<?php

declare(strict_types=1);

namespace {ProjectName}\{Domain}\Relation\{Related};

use Menumbing\Orm\Relation\HasOne;
use Menumbing\Orm\Trait\HasRelations;
use {ProjectName}\{Domain}\Model\{Related};

/**
 * @property-read {Related} ${related}
 */
trait HasOne{Related}Trait
{
    use HasRelations;

    public function {related}(): HasOne
    {
        return $this->hasOne({Related}::class);
    }
}
```

### HasMany (with scoped variants)

```php
<?php

declare(strict_types=1);

namespace {ProjectName}\{Domain}\Relation\{Related};

use Hyperf\Database\Model\Collection;
use Menumbing\Orm\Relation\HasMany;
use Menumbing\Orm\Trait\HasRelations;
use {ProjectName}\{Domain}\Constant\{Related}\{Related}Status;
use {ProjectName}\{Domain}\Model\{Related};

/**
 * @property-read Collection<int,{Related}> ${related}s
 * @property-read Collection<int,{Related}> $successful{Related}s
 */
trait HasMany{Related}sTrait
{
    use HasRelations;

    public function {related}s(): HasMany
    {
        return $this->hasMany({Related}::class);
    }

    public function successful{Related}s(): HasMany
    {
        return $this->{related}s()
            ->where('status', {Related}Status::SUCCEEDED);
    }
}
```

### BelongsToMany (with abstract base + scoped views)

```php
<?php

declare(strict_types=1);

namespace {ProjectName}\{Domain}\Relation\{Related};

use Hyperf\Database\Model\Collection;
use Menumbing\Orm\Relation\BelongsToMany;
use Menumbing\Orm\Trait\HasRelations;
use {ProjectName}\{Domain}\Model\{Related};

/**
 * @property Collection<int,{Related}> ${related}s
 * @property Collection<int,{Related}> $enabled{Related}s
 */
trait BelongsToMany{Related}sTrait
{
    use HasRelations;

    abstract public function {related}s(): BelongsToMany;

    public function enabled{Related}s(): BelongsToMany
    {
        return $this->{related}s()
            ->where('{pivot_table}.enabled', true)
            ->where('{related}s.enabled', true);
    }
}
```

Consuming model implements the abstract method:

```php
public function {related}s(): BelongsToMany
{
    return $this->belongsToMany({Related}::class, '{pivot_table}');
}
```

### MorphTo (polymorphic owner — inline on model, not in trait)

```php
<?php

declare(strict_types=1);

namespace {ProjectName}\{Domain}\Model;

use Menumbing\Orm\Model;
use Menumbing\Orm\Relation\MorphTo;
use Menumbing\Orm\Trait\HasRelations;

class {OwnerName} extends Model
{
    use HasRelations;

    public function {morph_name}(): MorphTo
    {
        return $this->morphTo();
    }
}
```

---

## Form Request

Naming: `{Action}{Name}Request` — mirrors the corresponding Action.

```php
<?php

declare(strict_types=1);

namespace {ProjectName}\{Domain}\Request;

use Hyperf\Validation\Request\FormRequest;

class {Action}{Name}Request extends FormRequest
{
    public function rules(): array
    {
        return [
            'field1' => 'required|string|max:255',
            'field2' => 'required|uuid',
        ];
    }

    public function authorize(): bool
    {
        return true;
    }
}
```

---

## Migration

```php
<?php

declare(strict_types=1);

use Hyperf\Database\Migrations\Migration;
use Hyperf\Database\Schema\Blueprint;
use Hyperf\Database\Schema\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('{table_name}', function (Blueprint $table) {
            $table->uuid('id')->primary();
            // columns...
            $table->string('status', 30)->index();
            $table->datetimes();
            $table->softDeletes();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('{table_name}');
    }
};
```

---

## Cache Key Class (PSR-16 + Hyperf CacheManager)

```php
<?php

declare(strict_types=1);

namespace {ProjectName}\{Domain}\Cache;

final class {Name}Key
{
    public static function generate(string $userId): string
    {
        return sprintf('{project_name}:{domain}:{name}:%s', $userId);
    }
}
```

---

## Service Using Cache

```php
<?php

declare(strict_types=1);

namespace {ProjectName}\{Domain}\Service\{Name};

use Hyperf\Cache\CacheManager;
use {ProjectName}\{Domain}\Cache\{Name}Key;

final readonly class {Name}Resolver
{
    public function __construct(
        private CacheManager $cacheManager,
        private {Name}Repository $repository,
    ) {}

    public function resolve(string $userId): ?{Name}
    {
        $cache = $this->cacheManager->getDriver();
        $key   = {Name}Key::generate($userId);

        $cached = $cache->get($key);
        if ($cached !== null) {
            return {Name}::fromArray($cached);
        }

        $entity = $this->repository->findByUserId($userId);
        if ($entity !== null) {
            $cache->set($key, $entity->toArray(), ttl: 300);
        }

        return $entity;
    }
}
```
