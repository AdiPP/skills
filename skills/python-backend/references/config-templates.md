# Python Backend — Config Templates

Full configuration templates for all files referenced in SKILL.md.

---

## pyproject.toml

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
    "multidict>=6,<7",
    "opentelemetry-instrumentation-asyncpg==0.41b0",
    "opentelemetry-instrumentation-redis==0.41b0",
    "psycopg2-binary>=2.9.11",
    "pydantic>=2.12.5,<3",
    "redis~=4.5.1",
    "sentry-sdk>=1.0.0,<2",
    "sqlalchemy>=2.0,<3.0",
    "sqlalchemy-utils>=0.42.1",
    "uuid7>=0.1.0,<1",
    "uvloop>=0.15.2,<1 ; sys_platform != 'win32'",
    "yarl>=1,<=1.12.1",
    "aiokafka>=0.12,<1",
]

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

## Settings

### settings/base.py

```python
from pathlib import Path

from environs import Env


env = Env()

STAND = env.str("STAND", default="local")
BASE_PATH = Path.cwd().absolute()

if STAND == "local":
    env.read_env(path=str(BASE_PATH / ".env"))
```

### settings/app.py

```python
from .base import STAND, env


class AppConfig:
    HOST = env.str("HOST", default="0.0.0.0")
    PORT = env.int("PORT", default=8001)
    CLIENT_MAX_SIZE = env.int("CLIENT_MAX_SIZE", default=1024 * 1024 * 50)
    BASE_URL = env.str("APP_BASE_URL", default="/")
    ENVIRONMENT = env.str("ENVIRONMENT", default="development")

    DOC = {
        "title": env.str("APP_DOC_TITLE", default="{project-name}"),
        "version": env.str("APP_DOC_VERSION", default="0.0.1"),
        "route": env.str("APP_DOC_URL", default="/apidocs.json"),
        "schemes": env.list(
            "APP_DOC_SCHEMES",
            subcast=str,
            default=["http"] if STAND == "local" else ["https"],
        ),
    }

    CORS = {
        "origin_whitelist": tuple(env.list("APP_ORIGIN_WHITELIST", default=["*"])),
        "allow_headers": tuple(env.list("APP_ALLOW_HEADERS", default=["*"])),
        "expose_headers": tuple(env.list("APP_EXPOSE_HEADERS", default=["*"])),
        "allow_credentials": env.bool("APP_ALLOW_CREDENTIALS", default=False),
        "max_age": env.int("APP_MAX_AGE", default=0),
    }

    COUNTRY_CODE = env.str("COUNTRY_CODE", default="ID")
    LIVENESS_RESOURCE_TIMEOUT = env.int("LIVENESS_RESOURCE_TIMEOUT", default=5)
    ALLOWED_DATA_MODIFICATION_ROLES = env.list("ALLOWED_DATA_MODIFICATION_ROLES", default=[])
```

### settings/db.py

```python
from .base import BASE_PATH, env
from sqlalchemy.engine import URL


class DbConfig:
    CONNECTION_SETTINGS = {
        "dsn": env.str("DB_URL", default=None),
        "database": env.str("DB_DATABASE", default="{project-name}"),
        "host": env.str("DB_HOST", default="localhost"),
        "port": env.int("DB_PORT", default=5432),
        "user": env.str("DB_USER", default="postgres"),
        "password": env.str("DB_PASSWORD", default=""),
        "min_size": env.int("DB_POOL_MIN_SIZE", default=1),
        "max_size": env.int("DB_POOL_MAX_SIZE", default=2),
        "max_inactive_connection_lifetime": env.float(
            "DB_POOL_MAX_INACTIVE_CONNECTION_LIFETIME", default=300
        ),
        "timeout": env.int("DB_TIMEOUT", default=60),
        "statement_cache_size": env.int("DB_STATEMENT_CACHE_SIZE", default=1024),
    }
    DEFAULT_LIMIT = env.int("DB_DEFAULT_LIMIT", default=10)
    MAX_LIMIT = env.int("DB_MAX_LIMIT", default=100)
    MAX_BIG_INT = 2**63 - 1
    DB_SCHEMA = env.str("DB_SCHEMA", default="public")
    PATH_TO_MIGRATIONS = BASE_PATH / "migrations"

    @classmethod
    def get_database_url(cls) -> URL:
        if cls.CONNECTION_SETTINGS["dsn"]:
            return cls.CONNECTION_SETTINGS["dsn"]

        return URL.create(
            drivername="postgresql+asyncpg",
            username=cls.CONNECTION_SETTINGS["user"],
            password=cls.CONNECTION_SETTINGS["password"],
            host=cls.CONNECTION_SETTINGS["host"],
            port=cls.CONNECTION_SETTINGS["port"],
            database=cls.CONNECTION_SETTINGS["database"],
        )

    @classmethod
    def get_engine_kwargs(cls) -> dict:
        min_size = cls.CONNECTION_SETTINGS["min_size"]
        max_size = cls.CONNECTION_SETTINGS["max_size"]

        return {
            "pool_size": min_size,
            "max_overflow": max(max_size - min_size, 0),
            "pool_timeout": cls.CONNECTION_SETTINGS["timeout"],
            "pool_recycle": cls.CONNECTION_SETTINGS["max_inactive_connection_lifetime"],
            "pool_pre_ping": True,
            "connect_args": {
                "timeout": cls.CONNECTION_SETTINGS["timeout"],
                "statement_cache_size": cls.CONNECTION_SETTINGS["statement_cache_size"],
                "server_settings": {
                    "search_path": cls.DB_SCHEMA,
                },
            },
        }
```

### settings/kafka.py

```python
from .base import env


class KafkaConfig:
    bootstrap_servers: list[str] = env.list(
        "KAFKA_BOOTSTRAP_SERVERS", default=["localhost:19092"]
    )
    security_protocol: str = env.str("KAFKA_SECURITY_PROTOCOL", default="SASL_PLAINTEXT")
    sasl_mechanism: str = env.str("KAFKA_SASL_MECHANISM", default="SCRAM-SHA-256")
    sasl_username: str = env.str("KAFKA_SASL_USERNAME", default="admin")
    sasl_password: str = env.str("KAFKA_SASL_PASSWORD", default="password")
    group_id: str = env.str("KAFKA_GROUP_ID", default="{project-name}")
    wait_ms: int = env.int("KAFKA_WAIT_MS", default=3000)
    enable_auto_commit: bool = env.bool("KAFKA_ENABLE_AUTO_COMMIT", default=False)
```

### settings/event_stream.py

```python
from .base import env


class EventStreamConfig:
    driver: str = env.str("EVENT_STREAM_DRIVER", default="kafka")
    # Additional driver-specific config
```

### settings/auth.py

```python
from .base import env


class ClientsAuthConfig:
    SERVICE_URL = env.str("CLIENTS_AUTH_URL", default="http://clients-auth")
    PUBLIC_KEY_ENDPOINT = env.str("AUTH_PUBLIC_KEY_ENDPOINT", default="/public_key")
    AUTH_HEADER_NAME = "X-AUTH-TOKEN"
    PAYLOAD_KEY = "CLIENT_AUTH_PAYLOAD"
    PUBLIC_KEY_NAME = "CLIENT_AUTH_PUBLIC_KEY"


class InternalAuthConfig:
    SERVICE_URL = env.str("INTERNAL_AUTH_URL", default="http://internal-auth")
    PUBLIC_KEY_ENDPOINT = env.str(
        "INTERNAL_AUTH_PUBLIC_KEY_ENDPOINT", default="/public_key"
    )
    USERNAME = env.str("INTERNAL_AUTH_USERNAME", default="")
    PASSWORD = env.str("INTERNAL_AUTH_PASSWORD", default="")
    REFRESH_TIMEOUT = env.int("INTERNAL_AUTH_REFRESH_TIMEOUT", default=60 * 55)
    HEADER_NAME = env.str("INTERNAL_AUTH_HEADER_NAME", default="Authorization")
    SECURITY_TOKEN_KEY = "INTERNAL_AUTH_ACCESS_TOKEN"
    SECURITY_TOKEN_REFRESH_KEY = "INTERNAL_AUTH_REFRESH_TOKEN"
    PAYLOAD_KEY = "INTERNAL_AUTH_PAYLOAD"
    PUBLIC_KEY_NAME = "INTERNAL_AUTH_PUBLIC_KEY"
    ADDITIONAL_HEADERS = env.list(
        "INTERNAL_AUTH_ADDITIONAL_HEADERS",
        subcast=str,
        default="X-Authorization",
    )


class ExternalAuthConfig:
    HEADER_NAME = env.str("EXTERNAL_SERVICE_AUTH_HEADER_NAME", default="X-ApiKey")
```

### settings/http_client.py

```python
from .base import env


class HttpClientConfig:
    """Base config for upstream HTTP clients."""
    base_url: str = ""
    timeout: int = env.int("HTTP_CLIENT_TIMEOUT", default=30)
    max_connections: int = env.int("HTTP_CLIENT_MAX_CONNECTIONS", default=10)


class UpstreamServiceConfig:
    BASE_URL = env.str("UPSTREAM_SERVICE_URL", default="http://service:8000")
    TIMEOUT = env.int("UPSTREAM_SERVICE_TIMEOUT", default=30)
```

### settings/logs.py

```python
from .base import env


class LogsConfig:
    LEVEL = env.str("LOG_LEVEL", default="DEBUG")
    FORMAT = env.str(
        "LOGS_FORMAT",
        default="%(asctime)s %(levelname)-8s [%(name)s] %(message)s",
    )

    LOGGING = {
        "version": 1,
        "disable_existing_loggers": True,
        "formatters": {
            "default": {
                "format": FORMAT,
            },
        },
        "handlers": {
            "default": {
                "level": LEVEL,
                "formatter": "default",
                "class": "logging.StreamHandler",
                "stream": "ext://sys.stdout",
            },
        },
        "loggers": {
            "": {
                "handlers": ["default"],
                "level": LEVEL,
                "propagate": False,
            },
            "aiohttp": {
                "level": "WARNING",
            },
            "aiokafka": {
                "level": "WARNING",
            },
            "sqlalchemy": {
                "level": "WARNING",
            },
        },
    }
```

### settings/telemetry.py

```python
from .base import env


class TelemetryConfig:
    PROMETHEUS_REGISTRY_APP_KEY = "prometheus_registry"
    ENABLE_METRICS = env.bool("ENABLE_METRICS", default=False)
```

### settings/__init__.py

```python
from .app import AppConfig
from .auth import ClientsAuthConfig, ExternalAuthConfig, InternalAuthConfig
from .db import DbConfig
from .event_stream import EventStreamConfig
from .kafka import KafkaConfig
from .http_client import HttpClientConfig, UpstreamServiceConfig
from .logs import LogsConfig
from .telemetry import TelemetryConfig

__all__ = [
    "AppConfig",
    "ClientsAuthConfig",
    "ExternalAuthConfig",
    "InternalAuthConfig",
    "DbConfig",
    "EventStreamConfig",
    "KafkaConfig",
    "HttpClientConfig",
    "UpstreamServiceConfig",
    "LogsConfig",
    "TelemetryConfig",
]
```

---

## Web Resources

### web/resources.py

```python
import json
import logging
from typing import Any, AsyncGenerator

from aiokafka import AIOKafkaProducer
from database import set_session_factory
from sqlalchemy.ext.asyncio import (
    AsyncEngine,
    AsyncSession,
    create_async_engine,
    async_sessionmaker,
)

from settings import DbConfig, KafkaConfig

logger = logging.getLogger(__name__)


async def db_engine() -> AsyncGenerator[AsyncEngine, Any]:
    engine = create_async_engine(
        DbConfig.get_database_url(),
        echo=False,
        **DbConfig.get_engine_kwargs(),
    )
    logger.debug("DB Connection established")
    try:
        yield engine
    finally:
        await engine.dispose()
        logger.debug("DB Connection closed")


def session_factory(engine: AsyncEngine) -> async_sessionmaker[AsyncSession]:
    session_maker = async_sessionmaker(
        bind=engine,
        expire_on_commit=False,
        class_=AsyncSession,
    )
    set_session_factory(session_maker)
    return session_maker


async def kafka_producer():
    producer = AIOKafkaProducer(
        bootstrap_servers=KafkaConfig.bootstrap_servers,
        security_protocol=KafkaConfig.security_protocol,
        sasl_mechanism=KafkaConfig.sasl_mechanism,
        sasl_plain_username=KafkaConfig.sasl_username,
        sasl_plain_password=KafkaConfig.sasl_password,
        value_serializer=lambda v: json.dumps(v).encode("utf-8"),
        key_serializer=lambda k: k.encode("utf-8") if k else None,
        acks="all",
    )
    await producer.start()
    logger.debug("Kafka Producer started")
    try:
        yield producer
    finally:
        await producer.stop()
        logger.debug("Kafka Producer stopped")


async def upstream_http_session():
    """Example upstream HTTP client session resource.

    One per upstream service — copy and rename for each.
    """
    from aiohttp import ClientSession, ClientTimeout, TCPConnector

    connector = TCPConnector(limit=10)
    session = ClientSession(
        connector=connector,
        timeout=ClientTimeout(total=30),
    )
    logger.debug("Upstream HTTP session established")
    try:
        yield session
    finally:
        await session.close()
        logger.debug("Upstream HTTP session closed")
```

---

## Web Container

### web/container.py

```python
from dependency_injector import containers, providers

from containers.auth import Auth
from containers.core import Core
from containers.http_client import HttpClients
from containers.repositories import Repositories
from containers.services import Services


class Container(containers.DeclarativeContainer):
    wiring_config = containers.WiringConfiguration(
        modules=[
            "web.middleware",
            "web.api.default",
            "web.api.{domain}",
            "app.commands",
            "app.listeners",
        ]
    )

    core: Core = providers.Container(Core)
    repositories: Repositories = providers.Container(Repositories, core=core)
    http_clients: HttpClients = providers.Container(HttpClients, core=core)
    services: Services = providers.Container(
        Services,
        core=core,
        repositories=repositories,
        http_clients=http_clients,
    )
```

---

## Web App Factory

### web/create_app.py

```python
import logging

from aiohttp import web
from aiohttp_ext.app import AppBuilder
from tracing.http import tracing_middleware

from settings import AppConfig
from settings.tracing import TracingConfig
from web.initializers import app_init
from .container import Container
from .routes import routes

logger = logging.getLogger(__name__)


class App(AppBuilder):
    on_start = app_init.all
    doc_settings = AppConfig.DOC
    cors_settings = AppConfig.CORS
    base_url = AppConfig.BASE_URL
    routes = routes
    middlewares = [
        tracing_middleware(TracingConfig()),
    ]


async def create_app() -> web.Application:
    app = App.create(client_max_size=AppConfig.CLIENT_MAX_SIZE)
    container = Container()
    app.container = container

    async def on_startup(app):
        logger.debug("Starting resources...")

    async def on_cleanup(app):
        logger.debug("Shutting down resources...")

    app.on_startup.append(on_startup)
    app.on_cleanup.append(on_cleanup)

    return app
```

---

## Manage.py

```python
# manage.py
import asyncio
import logging.config

import click
import uvloop
from aiohttp import web

import settings
from helpers.settings_reader import SettingsReader
from settings import AppConfig, LogsConfig
from web.container import Container
from web.create_app import create_app, create_worker_app

AppConfig = settings.AppConfig
DbConfig = settings.DbConfig
LogsConfig = settings.LogsConfig


@click.group()
def cli():
    """Init event loop, logging config etc."""
    logging.config.dictConfig(LogsConfig.LOGGING)
    uvloop.install()


@cli.command(short_help="start web")
def start():
    """Start REST API application."""
    web_app = create_app()
    web.run_app(web_app, port=AppConfig.PORT, access_log_class=access_logger)


@cli.command(short_help="start worker")
def start_worker():
    """Start worker app (event consumer) with healthcheck APIs."""
    web_app = create_worker_app()
    web.run_app(
        web_app, host=AppConfig.HOST, port=AppConfig.PORT, access_log_class=access_logger
    )


@cli.command(short_help="show application settings")
def show_settings():
    """Print to console application settings."""
    click.echo("Application settings:\n")
    SettingsReader(settings_module=settings).print_settings(print_func=click.echo)


@cli.command(short_help="run database seeders")
@click.argument("name", required=False)
def seed(name: str | None):
    async def main():
        from seeders.runner import run_seeders

        await run_seeders(name)

    asyncio.run(main())


@cli.command(short_help="list routes")
def list_routes():
    """List all registered routes."""
    from web.routes import routes

    for route in routes:
        print(route.method, route.path)


if __name__ == "__main__":
    cli()
```

---

## Dockerfile

```dockerfile
FROM python:3.12-slim AS base

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1

WORKDIR /app

# Install system dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends libpq-dev gcc && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements/production.txt .
RUN pip install -r production.txt

# Copy application
COPY . .

# --- API image ---
FROM base AS api
EXPOSE 8000
CMD ["python", "manage.py", "start"]

# --- Worker image ---
FROM base AS worker
EXPOSE 8000
CMD ["python", "manage.py", "start-worker"]
```

---

## Alembic

### alembic.ini

```ini
[alembic]
script_location = alembic
sqlalchemy.url = driver://user:pass@localhost/dbname

[loggers]
keys = root,sqlalchemy,alembic

[handlers]
keys = console

[formatters]
keys = generic

[logger_root]
level = WARN
handlers = console

[logger_sqlalchemy]
level = WARN
handlers =
qualname = sqlalchemy.engine

[logger_alembic]
level = INFO
handlers =
qualname = alembic

[handler_console]
class = StreamHandler
args = (sys.stderr,)
level = NOTSET
formatter = generic

[formatter_generic]
format = %(levelname)-5.5s [%(name)s] %(message)s
datefmt = %H:%M:%S
```

### alembic/env.py

```python
import asyncio
from logging.config import fileConfig

from alembic import context
from sqlalchemy import engine_from_config, pool
from sqlalchemy.engine import Connection
from sqlalchemy.ext.asyncio import AsyncEngine

from app.models.common import Base
from settings import DbConfig

config = context.config
if config.config_file_name is not None:
    fileConfig(config.config_file_name)

target_metadata = Base.metadata


def run_migrations_offline() -> None:
    url = config.get_main_option("sqlalchemy.url")
    context.configure(
        url=url,
        target_metadata=target_metadata,
        literal_binds=True,
        dialect_opts={"paramstyle": "named"},
        version_table_schema=DbConfig.DB_SCHEMA,
        include_schemas=False,
    )
    with context.begin_transaction():
        context.run_migrations()


def do_run_migrations(connection: Connection) -> None:
    context.configure(
        connection=connection,
        target_metadata=target_metadata,
        version_table_schema=DbConfig.DB_SCHEMA,
        include_schemas=False,
    )
    with context.begin_transaction():
        context.run_migrations()


async def run_migrations_online() -> None:
    connectable = AsyncEngine(
        engine_from_config(
            config.get_section(config.config_ini_section, {}),
            prefix="sqlalchemy.",
            poolclass=pool.NullPool,
            future=True,
        )
    )
    async with connectable.connect() as connection:
        await connection.run_sync(do_run_migrations)
    await connectable.dispose()


if context.is_offline_mode():
    run_migrations_offline()
else:
    asyncio.run(run_migrations_online())
```

### alembic/script.py.mako

```mako
"""${message}

Revision ID: ${up_revision}
Revises: ${down_revision | comma,n}
Create Date: ${create_date}
"""
from alembic import op
import sqlalchemy as sa
from sqlalchemy.dialects import postgresql
${imports if imports else ""}

# revision identifiers, used by Alembic.
revision = ${repr(up_revision)}
down_revision = ${repr(down_revision)}
branch_labels = ${repr(branch_labels)}
depends_on = ${repr(depends_on)}


def upgrade() -> None:
    ${upgrades if upgrades else "pass"}


def downgrade() -> None:
    ${downgrades if downgrades else "pass"}
```

---

## Makefile

```makefile
DEV_DEPS = requirements/develop.txt
PROD_DEPS = requirements/production.txt
PYTEST_CMD = python -m pytest
PATH_TO = .

upgrade-pip:
	@pip install --upgrade pip setuptools

install-prod: upgrade-pip
	@pip install -Ur $(PROD_DEPS)

install: upgrade-pip
	@pip install -Ur $(DEV_DEPS)

test:
	$(PYTEST_CMD) $(PATH_TO) --disable-warnings

lint:
	@flake8

format:
	@autopep8 -ir $(PATH_TO)
	@isort $(PATH_TO)
	@autoflake -ir --expand-star-imports --remove-duplicate --remove-unused \
		--remove-unused-variables --exclude __init__.py --exit-zero-even-if-changed $(PATH_TO)
	@add-trailing-comma --py36-plus --exit-zero-even-if-changed $(PATH_TO)

clean:
	@rm -rf `find . -name __pycache__`
	@rm -f `find . -type f -name '*.py[co]'`
	@rm -f `find . -type f -name '*~' `
	@rm -f `find . -type d -name '.pytest_cache' `
	@rm -f .coverage
	@rm -rf htmlcov
```

---

## .env.example

```bash
# App
HOST=0.0.0.0
PORT=8000
ENVIRONMENT=development
APP_BASE_URL=/
APP_DOC_TITLE={project-name}
APP_DOC_VERSION=0.0.1

# Database
DB_URL=postgresql://postgres:postgres@localhost:5432/{project-name}
DB_POOL_MIN_SIZE=1
DB_POOL_MAX_SIZE=2
DB_SCHEMA=public

# Kafka
KAFKA_BOOTSTRAP_SERVERS=localhost:19092
KAFKA_GROUP_ID={project-name}
KAFKA_SECURITY_PROTOCOL=SASL_PLAINTEXT
KAFKA_SASL_MECHANISM=SCRAM-SHA-256
KAFKA_SASL_USERNAME=admin
KAFKA_SASL_PASSWORD=password

# Auth
CLIENTS_AUTH_URL=http://clients-auth
INTERNAL_AUTH_URL=http://internal-auth

# Logging
LOG_LEVEL=DEBUG

# Sentry
SENTRY_DSN=
```

---

## docker-compose.yml

```yaml
version: "3.8"

services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: {project-name}
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

  kafka:
    image: bitnami/kafka:3.6
    environment:
      KAFKA_CFG_NODE_ID: 1
      KAFKA_CFG_PROCESS_ROLES: broker,controller
      KAFKA_CFG_CONTROLLER_QUORUM_VOTERS: 1@localhost:9093
      KAFKA_CFG_LISTENERS: PLAINTEXT://:19092,CONTROLLER://:9093
      KAFKA_CFG_ADVERTISED_LISTENERS: PLAINTEXT://localhost:19092
      KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,CONTROLLER:PLAINTEXT
      KAFKA_CFG_CONTROLLER_LISTENER_NAMES: CONTROLLER
    ports:
      - "19092:19092"

volumes:
  pgdata:
```

---

## docker-compose.local.yml

```yaml
version: "3.8"

services:
  app:
    build:
      context: .
      target: api
    ports:
      - "${PORT:-8000}:8000"
    env_file:
      - .env
    depends_on:
      - postgres
      - kafka
```

---

## .pre-commit-config.yaml

```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: check-added-large-files
      - id: check-merge-conflict
      - id: check-yaml
      - id: end-of-file-fixer
      - id: trailing-whitespace

  - repo: https://github.com/pycqa/isort
    rev: 5.13.2
    hooks:
      - id: isort

  - repo: https://github.com/pycqa/flake8
    rev: 7.0.0
    hooks:
      - id: flake8

  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.8.0
    hooks:
      - id: mypy
        additional_dependencies: [types-all]
```

---

## pytest.ini

```ini
[pytest]
asyncio_mode = auto
testpaths = tests
python_files = test_*.py
filterwarnings =
    ignore::DeprecationWarning
    ignore::PendingDeprecationWarning
```

---

## setup.cfg

```ini
[flake8]
max-line-length = 100
extend-ignore = E203, W503
exclude = .git,__pycache__,migrations,venv,.venv

[mypy]
python_version = 3.12
ignore_missing_imports = True
warn_unused_configs = True
```

---

## .gitignore

```gitignore
__pycache__/
*.py[cod]
*.egg-info/
dist/
build/
.venv/
venv/
.env
*.db
.pytest_cache/
.coverage
htmlcov/
.mypy_cache/
*.log
.DS_Store
```
