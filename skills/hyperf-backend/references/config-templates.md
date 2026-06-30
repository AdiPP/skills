# Hyperf Backend — Config Templates

---

## databases.php

```php
return [
    'default' => [
        'driver'    => env('DB_DRIVER', 'pgsql-swoole'),
        'host'      => env('DB_HOST', 'localhost'),
        'port'      => env('DB_PORT', 5432),
        'database'  => env('DB_DATABASE', '{project_name}db'),
        'username'  => env('DB_USERNAME', 'root'),
        'password'  => env('DB_PASSWORD', ''),
        'charset'   => env('DB_CHARSET', 'utf8mb4'),
        'collation' => env('DB_COLLATION', 'utf8mb4_unicode_ci'),
        'prefix'    => env('DB_PREFIX', '{service}_'),
        'schema'    => env('DB_SCHEMA', '{service}'),
        'pool' => [
            'min_connections' => env('DB_MIN_CONNECTIONS', 0),
            'max_connections' => env('DB_MAX_CONNECTIONS', 10),
            'connect_timeout' => (float) env('DB_CONNECT_TIMEOUT', 10.0),
            'wait_timeout'    => (float) env('DB_WAIT_TIMEOUT', 3.0),
            'heartbeat'       => -1,
            'max_idle_time'   => (float) env('DB_MAX_IDLE_TIME', 60),
        ],
    ],
];
```

---

## orm.php

```php
return [
    'persistent' => [
        'class'       => \Menumbing\Orm\Persistent\DatabasePersistent::class,
        'middlewares' => [
            \Menumbing\Orm\Persistent\Middleware\EnableDatabaseTransaction::class,
            \Menumbing\Orm\Persistent\Middleware\DispatchEvent::class,
            \Menumbing\Orm\Persistent\Middleware\ReleasePendingDomainEvent::class,
        ],
    ],
    'cache' => [
        'enabled' => env('ORM_CACHE_ENABLED', false),
        'store'   => env('ORM_CACHE_STORE', 'default'),
        'ttl'     => env('ORM_CACHE_TTL', 60 * 60 * 24),
    ],
];
```

---

## cache.php

```php
return [
    'default' => [
        'driver'  => Hyperf\Cache\Driver\RedisDriver::class,
        'packer'  => Hyperf\Codec\Packer\PhpSerializerPacker::class,
        'prefix'  => env('CACHE_PREFIX', '{project_name}:'),
    ],
];
```

---

## async_queue.php (AMQP / RabbitMQ)

Channel = bounded context name. Queue follows `{channel}.{action}.{service}-service`. Routing key follows `{channel}.{action}`. Set `use_delayed_exchange` to `false` to avoid requiring the RabbitMQ delayed-message plugin.

```php
<?php

declare(strict_types=1);

use Hyperf\Amqp\Message\Type;
use Hyperf\AsyncQueue\Driver\Amqp\AmqpDriverAdapter;
use Hyperf\AsyncQueue\Driver\SyncDriver;

return [
    'pools' => [
        'default' => [
            'driver'                => AmqpDriverAdapter::class,
            'auto_register_process' => true,
            'channel'               => '{domain}',
            'retry_seconds'         => [1, 5, 10, 30],
            'handle_timeout'        => 10,
            'processes'             => 1,
            'concurrent'            => ['limit' => 10],
            'max_messages'          => 0,
            'amqp' => [
                'pool'                 => 'default',
                'exchange_type'        => Type::DIRECT,
                'reroute_failed'       => false,
                'use_delayed_exchange' => false,
                'queue_durable'        => true,
                'queue_auto_delete'    => false,
                'exchange_auto_delete' => false,
                'queue_arguments'      => [],
                'queue'                => '{domain}.default.{service}-service',
                'routing_key'          => '{domain}.default',
            ],
        ],
        'test' => [
            'driver' => SyncDriver::class,
        ],
    ],
    'failed' => null,
    'debug' => [
        'before'      => false,
        'after'       => false,
        'failed'      => false,
        'retry'       => false,
        'before_push' => false,
        'after_push'  => false,
        'push_failed' => false,
    ],
];
```

CLI helpers:

```bash
php bin/hyperf.php queue:info [pool]
php bin/hyperf.php queue:reload <id>
php bin/hyperf.php queue:reload-all --pool=...
php bin/hyperf.php queue:flush --pool=...
```

---

## event_stream.php (Kafka)

```php
return [
    'group_name' => env('APP_NAME', '{project-name}-{service}'),
    'debug'      => true,
    'drivers' => [
        'default' => [
            'driver'  => \Menumbing\EventStream\Driver\Kafka\KafkaStream::class,
            'id'      => \Menumbing\EventStream\Driver\Kafka\DefaultKafkaId::class,
            'options' => [
                'pool'      => 'event-stream',
                'min_bytes' => 512,
                'max_wait'  => 300,
            ],
        ],
    ],
    'consumer' => [
        'processes'  => [],
        'block_for'   => 1,
        'retry_after' => 60,
        'concurrent'  => [],
    ],
    'serialization' => [
        'serializer' => 'default',
        'format'     => 'json',
    ],
    'skip_exceptions' => [
        \LogicException::class,
        \InvalidArgumentException::class,
        \DomainException::class,
    ],
];
```

---

## http_client.php (menumbing/http-client)

```php
<?php

declare(strict_types=1);

use Menumbing\Tracer\Constant\HttpClientTrace;
use function Hyperf\Support\env;

return [
    'defaults' => [
        'timeout' => 60,
        'headers' => [],
    ],
    'handler_factory' => [
        \Menumbing\HttpClient\Factory\GuzzleHttpClientHandlerFactory::class,
        [
            'min_connections' => 1,
            'max_connections' => 30,
            'wait_timeout'    => 3.0,
            'max_idle_time'   => 60,
        ],
    ],
    'middlewares' => [
        'retry' => [\Hyperf\Guzzle\RetryMiddleware::class, ['retries' => 1, 'delay' => 10]],
    ],
    'config_namespaces' => [],
    'http_clients' => [
        '{upstream}_api_client' => [
            'base_uri'    => env('{UPSTREAM}_BASE_URI', 'https://api.example.id'),
            'http_errors' => false,
            'timeout'     => 60,
            'headers'     => [
                'Accept' => 'application/json',
            ],
            'client_name' => '{upstream}-service',
            HttpClientTrace::TRACE_CAPTURE_ALL => true,
            'middlewares' => [
                'error' => [{ProjectName}\{Domain}\Middleware\HttpClient\{Upstream}Client\ErrorHandlerMiddleware::class],
                'retry' => [\Hyperf\Guzzle\RetryMiddleware::class, ['retries' => 0, 'delay' => 0]],
            ],
        ],
    ],
];
```

---

## opentracing.php (menumbing/tracer)

Toggle individual subsystems via env flags. Default driver is Zipkin; switch to Jaeger via `TRACER_DRIVER=jaeger`.

```php
<?php

declare(strict_types=1);

use Zipkin\Samplers\BinarySampler;
use function Hyperf\Support\env;

return [
    'default' => env('TRACER_DRIVER', 'zipkin'),
    'enable' => [
        'coroutine'             => (bool) env('TRACER_ENABLE_COROUTINE', false),
        'db_query'              => (bool) env('TRACER_ENABLE_DB', false),
        'exception'             => (bool) env('TRACER_ENABLE_EXCEPTION', false),
        'http_client'           => (bool) env('TRACER_ENABLE_HTTP_CLIENT', false),
        'method'                => (bool) env('TRACER_ENABLE_METHOD', false),
        'redis_call'            => (bool) env('TRACER_ENABLE_REDIS', false),
        'request_body'          => (bool) env('TRACER_ENABLE_REQUEST_BODY', false),
        'response_body'         => (bool) env('TRACER_ENABLE_RESPONSE_BODY', false),
        'event_stream_producer' => (bool) env('TRACER_ENABLE_EVENT_STREAM_PRODUCER', false),
        'event_stream_consumer' => (bool) env('TRACER_ENABLE_EVENT_STREAM_CONSUMER', false),
        'event_stream_payload'  => (bool) env('TRACER_ENABLE_EVENT_STREAM_PAYLOAD', false),
        'async_push'            => (bool) env('TRACER_ENABLE_ASYNC_PUSH', false),
        'async_consumer'        => (bool) env('TRACER_ENABLE_ASYNC_CONSUMER', false),
        'async_payload'         => (bool) env('TRACER_ENABLE_ASYNC_PAYLOAD', false),
        'ignore_exceptions'     => [],
        'trace_methods'         => [],
    ],
    'excluded_routes' => ['/health*'],
    'tracer' => [
        'zipkin' => [
            'driver'    => \Menumbing\Tracer\Adapter\ZipkinTracerFactory::class,
            'app'       => [
                'name' => env('APP_NAME', '{project-name}-{service}'),
                'ipv4' => null, 'ipv6' => null, 'port' => null,
            ],
            'reporter'  => env('ZIPKIN_REPORTER', 'http'),
            'reporters' => [
                'http' => [
                    'class'       => \Zipkin\Reporters\Http::class,
                    'constructor' => ['options' => [
                        'endpoint_url' => env('ZIPKIN_ENDPOINT_URL', 'http://localhost:9411/api/v2/spans'),
                        'timeout'      => (int) env('ZIPKIN_TIMEOUT', 1),
                    ]],
                ],
                'kafka' => [
                    'class'       => \Hyperf\Tracer\Adapter\Reporter\Kafka::class,
                    'constructor' => ['options' => [
                        'topic'             => env('ZIPKIN_KAFKA_TOPIC', 'zipkin'),
                        'bootstrap_servers' => env('ZIPKIN_KAFKA_BOOTSTRAP_SERVERS', '127.0.0.1:9092'),
                        'acks'              => (int) env('ZIPKIN_KAFKA_ACKS', -1),
                    ]],
                ],
                'noop'  => ['class' => \Zipkin\Reporters\Noop::class],
            ],
            'sampler' => BinarySampler::createAsAlwaysSample(),
            'trace_id_128bits' => false,
        ],
        'jaeger' => [
            'driver'  => \Hyperf\Tracer\Adapter\JaegerTracerFactory::class,
            'name'    => env('APP_NAME', '{project-name}-{service}'),
            'options' => [
                'local_agent' => [
                    'reporting_host' => env('JAEGER_REPORTING_HOST', 'localhost'),
                    'reporting_port' => env('JAEGER_REPORTING_PORT', 5775),
                ],
            ],
        ],
    ],
    'tags' => [
        'http_client' => ['http.url' => 'http.url', 'http.method' => 'http.method', 'http.status_code' => 'http.status_code'],
        'redis'       => ['arguments' => 'arguments', 'result' => 'result'],
        'db'          => ['db.query' => 'db.query', 'db.statement' => 'db.statement', 'db.query_time' => 'db.query_time'],
        'exception'   => ['class' => 'exception.class', 'code' => 'exception.code', 'message' => 'exception.message', 'stack_trace' => 'exception.stack_trace'],
        'request'     => ['path' => 'request.path', 'uri' => 'request.uri', 'method' => 'request.method', 'header' => 'request.header'],
        'response'    => ['status_code' => 'response.status_code'],
        'coroutine'   => ['id' => 'coroutine.id'],
    ],
];
```

---

## auth.php (menumbing/auth)

Pick guards based on whether the service owns the user/client aggregate (model providers) or only validates tokens issued by another service (stateless providers).

```php
<?php

declare(strict_types=1);

return [
    'default' => [
        'guard'     => 'oauth2',
        'passwords' => 'users',
    ],
    'guards' => [
        'oauth2' => [
            'driver'   => \Menumbing\OAuth2\ResourceServer\Guard\OAuth2Guard::class,
            'provider' => 'stateless_user',
            'options'  => [
                'token_validator' => env('OAUTH2_TOKEN_VALIDATOR', 'stateless'),
            ],
        ],
        'client' => [
            'driver'   => \Menumbing\OAuth2\ResourceServer\Guard\OAuth2Guard::class,
            'provider' => 'stateless_client',
            'options'  => [
                'token_validator' => env('OAUTH2_TOKEN_VALIDATOR', 'stateless'),
            ],
        ],
    ],
    'providers' => [
        'stateless_user' => [
            'driver' => \Menumbing\OAuth2\ResourceServer\Provider\User\StatelessUserProvider::class,
        ],
        'stateless_client' => [
            'driver' => \Menumbing\OAuth2\ResourceServer\Provider\Client\StatelessClientProvider::class,
        ],
        // 'users' => [
        //     'driver'  => \HyperfExtension\Auth\UserProviders\ModelUserProvider::class,
        //     'options' => [
        //         'model'       => {ProjectName}\{Domain}\Model\User::class,
        //         'hash_driver' => 'bcrypt',
        //     ],
        // ],
        // 'clients' => [
        //     'driver'  => \HyperfExtension\Auth\UserProviders\ModelUserProvider::class,
        //     'options' => [
        //         'model'       => {ProjectName}\{Domain}\Model\Client::class,
        //         'hash_driver' => 'bcrypt',
        //     ],
        // ],
    ],
    'passwords' => [
        'users' => [
            'driver'   => \HyperfExtension\Auth\Passwords\DatabaseTokenRepository::class,
            'provider' => 'users',
            'options'  => [
                'connection'  => null,
                'table'       => 'password_resets',
                'expire'      => 3600,
                'throttle'    => 60,
                'hash_driver' => null,
            ],
        ],
    ],
    'password_timeout' => 10800,
    'policies'         => [],
];
```

---

## oauth2_resource_server.php (menumbing/oauth2-resource-server)

```php
<?php

declare(strict_types=1);

use Menumbing\OAuth2\ResourceServer\Driver;
use function Hyperf\Support\env;

return [
    'public_key' => env('OAUTH2_PUBLIC_KEY'),
    'cookie' => [
        'name' => 'oauth2_token',
    ],
    'token_validators' => [
        'stateless' => [
            'driver' => Driver\TokenValidator\StatelessTokenValidator::class,
        ],
        'api' => [
            'driver'  => Driver\TokenValidator\ApiTokenValidator::class,
            'options' => [
                'http_client' => 'oauth2',
            ],
        ],
    ],
];
```

---

## middlewares.php

```php
<?php

declare(strict_types=1);

return [
    'http' => [
        \Gokure\HyperfCors\CorsMiddleware::class,
        \Menumbing\Tracer\Middleware\TraceMiddleware::class,
        \Hyperf\Validation\Middleware\ValidationMiddleware::class,
    ],
];
```

---

## health_check.php (menumbing/health-check)

Kubernetes liveness + readiness probes. **MANDATORY** in every service.

```php
<?php

declare(strict_types=1);

return [
    'checks' => [
        'liveness' => [
            'memory_consumption' => [
                'checker' => 'memory',
                'options' => [
                    'max_memory' => 1000, // MB
                ],
            ],
        ],
        'readiness' => [
            'database' => [
                'checker' => 'db',
                'options' => [
                    'connection' => 'default',
                ],
            ],
            'redis' => [
                'checker' => 'redis',
                'options' => [
                    'pool' => 'cache',
                ],
            ],
            // Add 'amqp' checker if using async queue
        ],
    ],
    'route' => [
        'server' => 'http',
        'paths' => [
            'liveness'  => '/health/liveness',
            'readiness' => '/health/readiness',
        ],
    ],
    'checkers' => [
        \Menumbing\HealthCheck\Checker\DbChecker::class,
        \Menumbing\HealthCheck\Checker\MemoryChecker::class,
        \Menumbing\HealthCheck\Checker\RedisChecker::class,
        // \Menumbing\HealthCheck\Checker\AmqpChecker::class, // if using AMQP
    ],
];
```

---

## authentik.php (custom Auth Guard config)

```php
<?php

declare(strict_types=1);

use function Hyperf\Support\env;

return [
    'jwks_url'       => env('AUTHENTIK_JWKS_URL', ''),
    'jwks_cache_ttl' => (int) env('AUTHENTIK_JWKS_CACHE_TTL', 3600),
];
```

---

## signature.php (menumbing/signature)

```php
<?php

declare(strict_types=1);

return [
    'client_repository_factory' => \Menumbing\Signature\Factory\OrmClientRepositoryFactory::class,
];
```

---

## crontab.php (hyperf/crontab)

```php
<?php

declare(strict_types=1);

use Hyperf\Crontab\Schedule;
use {ProjectName}\{Domain}\Command\{CommandClass};

return [
    'enable'  => true,
    'crontab' => [
        Schedule::command({CommandClass}::COMMAND_SIGNATURE)
            ->setName({CommandClass}::COMMAND_NAME)
            ->setRule('* * * * *')  // every minute
            ->isOnOneServer(),
    ],
];
```

---

## serializer.php (menumbing/serializer + Symfony)

Used for Kafka event denormalization and API response serialization.

```php
<?php

declare(strict_types=1);

use Symfony\Component\Serializer\Encoder;
use Symfony\Component\Serializer\Normalizer;

return [
    'default' => [
        'normalizers' => [
            Normalizer\ObjectNormalizer::class,
            Normalizer\DateTimeNormalizer::class,
            Normalizer\DateTimeZoneNormalizer::class,
            Normalizer\DateIntervalNormalizer::class,
            Normalizer\JsonSerializableNormalizer::class,
            Normalizer\ArrayDenormalizer::class,
        ],
        'encoders' => [
            Encoder\JsonEncoder::class,
        ],
    ],
];
```

---

## cors.php (gokure/hyperf-cors)

```php
<?php

declare(strict_types=1);

return [
    'paths'                 => ['*'],
    'allowed_methods'       => ['*'],
    'allowed_origins'       => ['*'],
    'allowed_origins_patterns' => [],
    'allowed_headers'       => ['*'],
    'exposed_headers'       => ['Trace-Id'],
    'max_age'               => 0,
    'supports_credentials'  => false,
];
```

---

## server.php (Swoole HTTP server)

```php
<?php

declare(strict_types=1);

use Hyperf\Server\Event;
use Hyperf\Server\Server;
use Swoole\Constant;
use function Hyperf\Support\env;

return [
    'mode'    => SWOOLE_BASE,
    'servers' => [
        [
            'name'      => 'http',
            'type'      => Server::SERVER_HTTP,
            'host'      => '0.0.0.0',
            'port'      => (int) env('HTTP_SERVER_PORT', 8000),
            'sock_type' => SWOOLE_SOCK_TCP,
            'callbacks' => [
                Event::ON_REQUEST => [Hyperf\HttpServer\Server::class, 'onRequest'],
            ],
            'options' => [
                'enable_request_lifecycle' => false,
            ],
        ],
    ],
    'settings' => [
        Constant::OPTION_ENABLE_COROUTINE   => true,
        Constant::OPTION_WORKER_NUM         => swoole_cpu_num(),
        Constant::OPTION_PID_FILE           => BASE_PATH . '/runtime/hyperf.pid',
        Constant::OPTION_OPEN_TCP_NODELAY   => true,
        Constant::OPTION_MAX_COROUTINE      => 100000,
        Constant::OPTION_OPEN_HTTP2_PROTOCOL => true,
        Constant::OPTION_MAX_REQUEST        => 100000,
        Constant::OPTION_SOCKET_BUFFER_SIZE => 2 * 1024 * 1024,
        Constant::OPTION_BUFFER_OUTPUT_SIZE => 2 * 1024 * 1024,
    ],
    'callbacks' => [
        Event::ON_WORKER_START => [Hyperf\Framework\Bootstrap\WorkerStartCallback::class, 'onWorkerStart'],
        Event::ON_PIPE_MESSAGE => [Hyperf\Framework\Bootstrap\PipeMessageCallback::class, 'onPipeMessage'],
        Event::ON_WORKER_EXIT  => [Hyperf\Framework\Bootstrap\WorkerExitCallback::class, 'onWorkerExit'],
    ],
];
```

---

## exceptions.php (handler chain)

Priority order matters — first matching handler wins.

```php
<?php

declare(strict_types=1);

return [
    'handler' => [
        'http' => [
            \Menumbing\Tracer\Exception\Handler\RespondTraceIdExceptionHandler::class,
            \Menumbing\OAuth2\ResourceServer\Exception\Handler\OAuth2ServerExceptionHandler::class,
            {ProjectName}\{Domain}\Exception\Handler\AppExceptionHandler::class,
        ],
    ],
];
```
