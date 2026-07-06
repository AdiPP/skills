# Python Backend — Code Patterns

Full code templates for all patterns referenced in SKILL.md.

---

## API View (Endpoint Handler)

Naming: `{Action}{Entity}` — single-purpose view class extending `JSONView` with marshmallow validation.

```python
# web/api/{domain}/views.py
from aiohttp.abc import StreamResponse
from dependency_injector.wiring import Provide, inject
from aiohttp_ext.views import JSONView
from aiohttp_ext.views.types import RequestParams
from database import db_transaction
from auth import auth, require_auth, get_auth

from app.dto.{domain} import {Action}{Entity}DTO
from app.enums import {Entity}Status
from app.models import {Entity}
from app.repositories import {Entity}Repository
from app.services.{entity} import {Entity}{Suffix}
from web.container import Container

from .schemas import {Action}{Entity}RequestSchema, {Action}{Entity}PathSchema


class {Action}{Entity}(JSONView):
    schema_body = {Action}{Entity}RequestSchema()
    schema_path = {Action}{Entity}PathSchema()

    @inject
    async def post_response(
        self,
        params: RequestParams,
        service: {Entity}{Suffix} = Provide[Container.services.{entity}_{suffix}],
    ) -> StreamResponse:
        payload = params.body
        dto = {Action}{Entity}DTO(**payload)

        entity = await service.{action}(dto)

        return entity.to_dict()
```

---

## Schema (Marshmallow)

Request and response schemas per endpoint.

```python
# web/api/{domain}/schemas.py
from marshmallow import EXCLUDE, Schema, ValidationError, fields, validate, validates_schema
from marshmallow.validate import OneOf


class {Entity}BaseSchema(Schema):
    id = fields.UUID(dump_only=True)
    name = fields.String()
    status = fields.String()
    created_at = fields.String(dump_only=True)
    updated_at = fields.String(dump_only=True)


class {Entity}ResourceResponseSchema({Entity}BaseSchema):
    class Meta:
        unknown = EXCLUDE


# --- {Action} {Entity} ---

class {Action}{Entity}RequestSchema(Schema):
    name = fields.String(required=True, validate=validate.Length(min=1, max=100))
    # additional fields...


class {Action}{Entity}PathSchema(Schema):
    {entity} = fields.UUID(required=True)
```

---

## Model (SQLAlchemy)

Naming: `{Entity}` — extends `Base, UuidMixin, TimestampMixin`.

```python
# app/models/{entity}s.py
from typing import TYPE_CHECKING

from sqlalchemy import ForeignKey, String, Text
from sqlalchemy.dialects.postgresql import UUID
from sqlalchemy.orm import Mapped, mapped_column, relationship

from .common import Base, TimestampMixin, UuidMixin


if TYPE_CHECKING:
    from app.models.related import RelatedEntity


class {Entity}(Base, UuidMixin, TimestampMixin):
    __tablename__ = "{table_name}"

    fillable = {
        "name",
        "status",
        # field names that can be mass-assigned via fill()
    }

    name: Mapped[str] = mapped_column(String(100), nullable=False)
    status: Mapped[str] = mapped_column(String(50), nullable=False, default="draft")
    description: Mapped[str | None] = mapped_column(Text, nullable=True)

    # Foreign key relationships
    related_id: Mapped[UUID | None] = mapped_column(
        UUID(as_uuid=True),
        ForeignKey("related_table.id", ondelete="SET NULL"),
        nullable=True,
    )
    related: Mapped["RelatedEntity"] = relationship(back_populates="{entity}s")

    @classmethod
    def initialize(cls, **kwargs) -> "{Entity}":
        """Factory method for creating a new instance with required setup."""
        entity = cls()
        for key, value in kwargs.items():
            setattr(entity, key, value)
        return entity

    def patch(self, **kwargs) -> None:
        """Partial update — never resets unmentioned fields."""
        allowed_fields = self.fillable.intersection(kwargs.keys())
        for key in allowed_fields:
            setattr(self, key, kwargs[key])
```

### Common Mixins

```python
# app/models/common.py
from datetime import datetime

from sqlalchemy import DateTime, MetaData, func, inspect
from sqlalchemy.dialects.postgresql import UUID, TIMESTAMP
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column
from uuid_extensions import uuid7

from helpers.json_serializer import json_safe
from settings import DbConfig

meta = MetaData(schema=DbConfig.DB_SCHEMA)


class UuidMixin:
    id: Mapped[UUID] = mapped_column(
        UUID(as_uuid=True),
        primary_key=True,
        default=uuid7,
    )

    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        if self.id is None:
            self.id = uuid7()


class TimestampMixin:
    created_at: Mapped[datetime] = mapped_column(
        TIMESTAMP(precision=0),
        server_default=func.now(),
        default=datetime.utcnow,
        nullable=False,
    )

    updated_at: Mapped[datetime] = mapped_column(
        TIMESTAMP(precision=0),
        server_default=func.now(),
        default=datetime.utcnow,
        onupdate=datetime.utcnow,
        nullable=False,
    )


class Base(DeclarativeBase):
    metadata = meta
    fillable: set[str] = set()

    def fill(self, **kwargs):
        """Mass-assign only fillable attributes."""
        mapper = inspect(self.__class__)
        columns = {c.key for c in mapper.attrs}
        allowed_keys = self.fillable.intersection(columns)

        for key, value in kwargs.items():
            if key in allowed_keys:
                setattr(self, key, value)
        return self

    def to_dict(self) -> dict:
        """Convert model instance to a plain dict."""
        serialized = {}
        for column in self.__table__.columns:
            value = getattr(self, column.name)
            serialized[column.name] = json_safe(value)
        return serialized
```

---

## Repository

Naming: `{Entity}Repository` — extends `Repository` from the database library.

```python
# app/repositories/{entity}s.py
from uuid import UUID

from aiohttp_ext.errors import NotFoundError
from database import Repository
from sqlalchemy import select
from sqlalchemy.orm import joinedload, selectinload

from app.models import {Entity}


class {Entity}Repository(Repository):
    entity = {Entity}

    async def save(self, entity: {Entity}) -> {Entity}:
        async with self.session() as session:
            session.add(entity)
            return entity

    async def find_or_fail(self, entity_id: str | UUID) -> {Entity}:
        async with self.session() as session:
            stmt = select(self.entity).where(self.entity.id == entity_id)
            result = await session.execute(stmt)
            entity = result.scalar_one_or_none()

            if entity is None:
                raise NotFoundError(
                    f"{Entity.__name__} with id '{entity_id}' not found"
                )
            return entity

    async def find_one_by_{field}(self, value: str) -> {Entity} | None:
        async with self.session() as session:
            stmt = select(self.entity).where(self.entity.{field} == value)
            result = await session.execute(stmt)
            return result.scalar_one_or_none()

    async def find_all(self, limit: int = 10, offset: int = 0) -> list[{Entity}]:
        async with self.session() as session:
            stmt = select(self.entity).limit(limit).offset(offset)
            result = await session.execute(stmt)
            return list(result.scalars().all())

    def _apply_relations(self, stmt, relations: list[str] | None = None):
        """Apply eager loading options to a select statement."""
        if not relations:
            return stmt
        options = []
        for relation in relations:
            if relation == "related_entity":
                options.append(joinedload(getattr(self.entity, relation)))
        return stmt.options(*options)
```

---

## Domain Service

Naming: `{Entity}{Suffix}` — single-responsibility, noun + role.

```python
# app/services/{entity}/{entity}_{suffix}.py
from uuid import UUID
from uuid_extensions import uuid7

from app.dto import {Action}{Entity}DTO
from app.enums import {Entity}Status
from app.models import {Entity}
from app.repositories import {Entity}Repository


class {Entity}{Suffix}:
    def __init__(self, {entity}_repository: {Entity}Repository):
        self.{entity}_repo = {entity}_repository

    async def {action}(self, dto: {Action}{Entity}DTO) -> {Entity}:
        entity = {Entity}.initialize(
            name=dto.name,
            status={Entity}Status.DRAFT,
        )
        entity = await self.{entity}_repo.save(entity)
        return entity
```

### Service with Transaction Decorator

```python
# app/services/{entity}/{entity}_{suffix}.py
from database import db_transaction
from aiohttp_ext.errors import ValidationError

from app.dto import {Action}{Entity}DTO
from app.enums import {Entity}Status
from app.events import {Domain}{Action}
from event_stream import EventDispatcher
from app.models import {Entity}
from app.repositories import {Entity}Repository


class {Entity}{Suffix}:
    def __init__(
        self,
        {entity}_repository: {Entity}Repository,
        event_dispatcher: EventDispatcher,
    ):
        self.{entity}_repo = {entity}_repository
        self.event_dispatcher = event_dispatcher

    @db_transaction
    async def {action}(self, dto: {Action}{Entity}DTO) -> {Entity}:
        entity = await self.{entity}_repo.find_or_fail(dto.entity_id)

        if entity.status != {Entity}Status.DRAFT:
            raise ValidationError("Can only {action} entities in draft status")

        entity.status = {Entity}Status.ACTIVE
        entity = await self.{entity}_repo.save(entity)

        await self.event_dispatcher.publish(
            {Domain}{Action}(
                entity_id=str(entity.id),
                status=entity.status,
            )
        )

        return entity
```

---

## DTO (Data Transfer Object)

Naming: `{Action}{Entity}DTO` — frozen dataclass.

```python
# app/dto/{domain}.py
from dataclasses import dataclass
from uuid import UUID


@dataclass(frozen=True)
class {Action}{Entity}DTO:
    name: str
    entity_id: UUID | None = None
    status: str | None = None
```

---

## Produced Event

Naming: `{Domain}{Action}` — frozen dataclass with `@produced_event` decorator.

```python
# app/events/{domain}.py
from dataclasses import dataclass

from event_stream import BaseEvent, produced_event


@produced_event(
    stream='{stream_name}',
    name='{event_name}',
)
@dataclass(frozen=True)
class {Domain}{Action}(BaseEvent):
    entity_id: str
    status: str
```

---

## Consumed Event (Listener)

Naming: `{SubDomain}{EventName}Handler` — decorated with `@event_listener`, extends `EventHandler`.

```python
# app/listeners/{domain}.py
from dependency_injector.wiring import Provide, inject
from event_stream import EventHandler, event_listener

from app.events.other_service import OtherServiceEvent
from app.repositories import {Entity}Repository
from app.services.{entity} import {Entity}{Suffix}
from web.container import Container


@event_listener(OtherServiceEvent)
class {SubDomain}{EventName}Handler(EventHandler):
    @inject
    async def handle(
        self,
        event: OtherServiceEvent | dict,
        {entity}_{suffix}: {Entity}{Suffix} = Provide[Container.services.{entity}_{suffix}],
    ):
        await {entity}_{suffix}.{action}(
            entity_id=event.entity_id,
            status=event.status,
        )
```

### Listener for Raw Events (Unmapped)

```python
from event_stream import EventHandler, event_listener_raw, RawEvent


@event_listener_raw(RawEvent(name="other.service.event", stream="other-service-stream"))
class {SubDomain}{EventName}Handler(EventHandler):
    @inject
    async def handle(
        self,
        event: dict,
        # inject dependencies...
    ):
        payload = event if isinstance(event, dict) else {}
        # handle raw event payload
```

---

## Enum

Naming: Descriptive name with snake_case values.

```python
# app/enums/{entity}_status.py
from enum import StrEnum


class {Entity}Status(StrEnum):
    DRAFT = "draft"
    ACTIVE = "active"
    SUSPENDED = "suspended"
    DELETED = "deleted"
```

```python
# app/enums/__init__.py
from .{entity}_status import *
```

---

## Exception

Naming: `{Entity}{Error}Exception` — extends `ClientError`.

```python
# app/exceptions/{entity}_exceptions.py
from aiohttp_ext.errors import ClientError


class {Entity}NotFoundException(ClientError):
    def __init__(self, entity_id: str):
        super().__init__(f"{Entity.__name__} with id '{entity_id}' not found")


class Invalid{Entity}StateException(ClientError):
    def __init__(self, current_status: str, expected_status: str):
        super().__init__(
            f"Invalid {Entity.__name__} status '{current_status}', expected '{expected_status}'"
        )
```

---

## HTTP Client

Per-upstream service client.

```python
# app/client/{upstream}_client/{upstream}_client.py
from typing import Any

from aiohttp import ClientSession, ClientTimeout


class {Upstream}Client:
    def __init__(self, session: ClientSession, base_url: str):
        self._session = session
        self._base_url = base_url

    async def get_{entity}(self, entity_id: str) -> dict[str, Any]:
        url = f"{self._base_url}/v1/{entity}s/{entity_id}"
        async with self._session.get(url) as resp:
            resp.raise_for_status()
            return await resp.json()

    async def post_{action}(self, payload: dict) -> dict[str, Any]:
        url = f"{self._base_url}/v1/{endpoint}"
        async with self._session.post(url, json=payload) as resp:
            resp.raise_for_status()
            return await resp.json()
```

---

## CLI Command

Auto-registered commands via `__init_subclass__`.

```python
# app/commands/{action}_command.py
import click
from click import Command

from app.services.{entity} import {Entity}{Suffix}
from web.container import Container


class {Action}Command(Command):
    def __init__(self):
        super().__init__(name="{action}-{entity}")

    def invoke(self, ctx: click.Context) -> None:
        import asyncio
        container = Container()
        service = container.services.{entity}_{suffix}()
        asyncio.run(service.{action}(...))
```

---

## Seeder

```python
# seeders/{entity}_seeder.py
from seeders.base import BaseSeeder


class {Entity}Seeder(BaseSeeder):
    def __init__(self):
        self.data = [
            {"name": "Item 1", "status": "active"},
            {"name": "Item 2", "status": "active"},
        ]

    async def run(self):
        from app.models import {Entity}
        for item in self.data:
            entity = {Entity}.initialize(**item)
            await self.save(entity)
```

---

## Alembic Migration (Async)

```python
# alembic/versions/XXXX_create_{table}_table.py
"""create {table_name} table

Revision ID: xxxx
Revises: yyyy
Create Date: 2026-01-01 00:00:00.000000
"""
from alembic import op
import sqlalchemy as sa
from sqlalchemy.dialects import postgresql
from uuid_extensions import uuid7

revision = 'xxxx'
down_revision = 'yyyy'
branch_labels = None
depends_on = None


def upgrade() -> None:
    op.create_table(
        '{table_name}',
        sa.Column('id', postgresql.UUID(as_uuid=True), primary_key=True, default=uuid7),
        sa.Column('name', sa.String(100), nullable=False),
        sa.Column('status', sa.String(50), nullable=False, server_default='draft'),
        sa.Column('created_at', sa.TIMESTAMP(timezone=False), server_default=sa.func.now(), nullable=False),
        sa.Column('updated_at', sa.TIMESTAMP(timezone=False), server_default=sa.func.now(), nullable=False),
        schema='{schema}',
    )
    op.create_index(
        'ix_{table_name}_name',
        '{table_name}',
        ['name'],
        schema='{schema}',
    )


def downgrade() -> None:
    op.drop_index('ix_{table_name}_name', table_name='{table_name}', schema='{schema}')
    op.drop_table('{table_name}', schema='{schema}')
```

---

## Resource Lifecycle (Web Resources)

Async generators for DB engine, Kafka producer, HTTP sessions — managed via `cleanup_ctx`.

```python
# web/resources.py
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
```

---

## DI Container Wiring

```python
# containers/core.py
from dependency_injector import containers, providers

from app.event_stream.dispatcher import EventDispatcher
from app.event_stream.handler import HandlerRegistry
from web.resources import db_engine, session_factory, kafka_producer


class Core(containers.DeclarativeContainer):
    engine = providers.Resource(db_engine)
    session_factory = providers.Singleton(session_factory, engine=engine)
    handler_registry = providers.Singleton(HandlerRegistry)
    kafka_producer = providers.Resource(kafka_producer)
    event_dispatcher = providers.Singleton(EventDispatcher, producer=kafka_producer)
```

```python
# containers/repositories.py
from dependency_injector import containers, providers

from app.repositories import {Entity}Repository
from containers.core import Core


class Repositories(containers.DeclarativeContainer):
    core: Core = providers.DependenciesContainer()
    session_factory_service = core.session_factory

    {entity}_repository = providers.Singleton(
        {Entity}Repository,
        session_factory=session_factory_service,
    )
```

```python
# containers/services.py
from dependency_injector import containers, providers

from app.services.{entity} import {Entity}{Suffix}
from containers.core import Core
from containers.http_client import HttpClients
from containers.repositories import Repositories


class Services(containers.DeclarativeContainer):
    core: Core = providers.DependenciesContainer()
    repositories: Repositories = providers.DependenciesContainer()
    http_clients: HttpClients = providers.DependenciesContainer()

    {entity}_{suffix} = providers.Singleton(
        {Entity}{Suffix},
        {entity}_repository=repositories.{entity}_repository,
    )
```

```python
# web/container.py
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

## App Factory

```python
# web/create_app.py
import logging

from aiohttp import web
from aiohttp_ext.app import AppBuilder
from tracing.http import tracing_middleware

from settings import AppConfig
from settings.tracing import TracingConfig
from web.container import Container
from web.initializers import app_init
from web.routes import routes

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


async def create_app(app: Application) -> web.Application:
    web_app = App.create(client_max_size=AppConfig.CLIENT_MAX_SIZE)
    web_app.container = app.container

    async def on_startup(app):
        logger.debug("Starting resources...")

    async def on_cleanup(app):
        logger.debug("Shutting down resources...")

    web_app.on_startup.append(on_startup)
    web_app.on_cleanup.append(on_cleanup)

    return web_app
```

---

## Routes

```python
# web/routes.py
from aiohttp import web

from .api import default, {domain}


default_routes = [
    web.get('/health/readiness', default.Readiness),
    web.get('/health/liveness', default.Liveness),
]

{domain}_routes = [
    web.post('/v1/{entity}s', {domain}.{Action}{Entity}),
    web.get('/v1/{entity}s/{{entity}}', {domain}.Get{Entity}),
    web.patch('/v1/{entity}s/{{entity}}', {domain}.Update{Entity}),
    web.delete('/v1/{entity}s/{{entity}}', {domain}.Delete{Entity}),
]

routes = (
    *default_routes,
    *{domain}_routes,
)
```

---

## Manage.py (CLI Entry Point)

```python
# manage.py
import asyncio
import logging.config

import click
from aiohttp import web
from click import Command
from settings import AppConfig, DbConfig, LogsConfig
from web.container import Container
from web.create_app import create_app
from helpers.settings_reader import SettingsReader
import settings
import uvloop


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


@cli.command(short_help="start workers")
def start_workers():
    """Start worker app (event consumer)."""
    web_app = create_worker_app()
    web.run_app(web_app, port=AppConfig.PORT, access_log_class=access_logger)


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

## App Initializers (Healthcheck + Event Consumer)

```python
# web/initializers.py
from functools import cached_property
from logging import getLogger
from typing import Any

from aiohttp_ext.clients.health_check import HealthCheck
from settings import constants, AppConfig
from web.health_check import PgPoolHealthCheck
from app.event_stream.consumer import init_event_consumer
from aiohttp_ext.clients.worker import WorkerInit

logger = getLogger(__name__)


class InitApp:
    @cached_property
    def event_consumer(self):
        return WorkerInit(
            init_callback=init_event_consumer,
            app_attribute_name="EVENT_CONSUMER",
        )

    @cached_property
    def healthcheck(self):
        resources = [PgPoolHealthCheck()]
        return HealthCheck(
            resources=resources,
            app_attribute_name=constants.HEALTH_CHECK,
        )

    @property
    def all(self):
        inits: list[Any] = [self.healthcheck]
        if AppConfig.ENVIRONMENT == "staging":
            inits.append(self.event_consumer)
        return tuple(inits)


app_init = InitApp()
```

---

## Event Stream Consumer (Kafka)

```python
# app/event_stream/consumer.py
import logging

import ujson
from aiokafka import AIOKafkaConsumer, ConsumerRecord
from kafka_client import KafkaConsumer

from app.event_stream.handler import HandlerRegistry
from settings.kafka import KafkaConfig

logger = logging.getLogger(__name__)


class EventConsumer(KafkaConsumer):
    async def process_msg(self, record: ConsumerRecord) -> None:
        try:
            payload = ujson.loads(record.value)
            event_name: str = payload["type"]
            data: dict = payload["data"]

            app = self["app"]
            container = app.container
            registry: HandlerRegistry = container.core.handler_registry()

            entry = registry.get_entry(event_name)
            if not entry:
                logger.warning(f"No handler for event '{event_name}'")
                return

            handler = registry.get_handler(event_name)
            if entry.is_typed:
                event = entry.event_cls.from_dict(data)
            else:
                event = data

            await handler.handle(event)
            await self._consumer.commit()
        except Exception:
            logger.exception("Error processing Kafka message", extra={"record": record})


async def init_event_consumer(app):
    registry: HandlerRegistry = app.container.core.handler_registry()
    streams = registry.event_streams()

    if not streams:
        raise RuntimeError("No event streams to consume")

    consumer = EventConsumer(
        consumer=AIOKafkaConsumer(
            *streams,
            group_id=KafkaConfig.group_id,
            bootstrap_servers=KafkaConfig.bootstrap_servers,
            fetch_max_wait_ms=KafkaConfig.wait_ms,
            security_protocol=KafkaConfig.security_protocol,
            sasl_mechanism=KafkaConfig.sasl_mechanism,
            sasl_plain_username=KafkaConfig.sasl_username,
            sasl_plain_password=KafkaConfig.sasl_password,
        ),
    )
    consumer["app"] = app
    return consumer
```

---

## Event Stream Dispatcher

```python
# app/event_stream/dispatcher.py
from aiokafka import AIOKafkaProducer
from event_stream import BaseEvent
from uuid_extensions import uuid7


class EventDispatcher:
    def __init__(self, producer: AIOKafkaProducer):
        self.producer = producer

    async def publish(self, event: BaseEvent):
        await self.producer.send_and_wait(
            topic=event.event_stream,
            value={
                "stream": event.event_stream,
                "type": event.event_name,
                "data": event.to_dict(),
                "context": {},
                "id": None,
            },
            key=str(uuid7()),
        )
```

---

## Event Handler Registry

```python
# app/event_stream/handler.py
from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import Optional, Type

from event_stream import BaseEvent


class EventHandler(ABC):
    @abstractmethod
    async def handle(self, event: BaseEvent | dict):
        pass


@dataclass(frozen=True)
class HandlerEntry:
    handler_cls: Type[EventHandler]
    stream: str
    event_name: str
    event_cls: Optional[Type[BaseEvent]] = None
    eager: bool = True

    @property
    def is_typed(self) -> bool:
        return self.event_cls is not None

    @staticmethod
    def from_event_class(
        *,
        handler_cls: Type[EventHandler],
        event_cls: Type[BaseEvent],
    ) -> "HandlerEntry":
        return HandlerEntry(
            handler_cls=handler_cls,
            stream=event_cls.event_stream,
            event_name=event_cls.event_name,
            event_cls=event_cls,
        )

    @staticmethod
    def from_raw(
        *,
        handler_cls: Type[EventHandler],
        event_name: str,
        stream: str,
    ) -> "HandlerEntry":
        return HandlerEntry(
            handler_cls=handler_cls,
            stream=stream,
            event_name=event_name,
            event_cls=None,
        )


class HandlerRegistry:
    def __init__(self) -> None:
        self._entries: dict[str, HandlerEntry] = {}
        self._instances: dict[str, EventHandler] = {}

    def register(self, entry: HandlerEntry) -> None:
        self._entries[entry.event_name] = entry
        if entry.eager:
            self._instances[entry.event_name] = entry.handler_cls()

    def get_entry(self, event_name: str) -> HandlerEntry | None:
        return self._entries.get(event_name)

    def get_handler(self, event_name: str) -> EventHandler:
        if event_name not in self._instances:
            self._instances[event_name] = self._entries[event_name].handler_cls()
        return self._instances[event_name]

    def event_streams(self) -> set[str]:
        return {entry.stream for entry in self._entries.values()}
```

---

## Event Base Class

```python
# app/event_stream/event.py
from dataclasses import dataclass
from typing import ClassVar, Type


@dataclass(frozen=True)
class BaseEvent:
    event_stream: ClassVar[str] = ""
    event_name: ClassVar[str] = ""

    def to_dict(self) -> dict:
        return self.__dict__

    @classmethod
    def from_dict(cls, payload: dict):
        return cls(**payload)


EVENT_REGISTRY: dict[str, type[BaseEvent]] = {}


def event_message(*, stream: str, name: str):
    def decorator(cls: Type[BaseEvent]):
        if not issubclass(cls, BaseEvent):
            raise TypeError("event_message can only decorate BaseEvent subclasses")
        cls.event_stream = stream
        cls.event_name = name
        if name in EVENT_REGISTRY:
            raise RuntimeError(f"Event '{name}' already registered")
        EVENT_REGISTRY[name] = cls
        return cls
    return decorator
```

---

## Error Resource (Error Response)

```python
# app/errors/errors.py
class ErrorResource:
    """Structured error response."""

    def __init__(self, error: str, description: str, code: int = 400):
        self.error = error
        self.error_description = description
        self.code = code

    def to_dict(self) -> dict:
        return {
            "error": self.error,
            "error_description": self.error_description,
            "code": self.code,
        }
```

---

## Test Example

```python
# tests/test_api/test_create_{entity}.py
from contextlib import asynccontextmanager
from unittest.mock import AsyncMock, MagicMock, patch
from uuid import uuid4

import pytest
from dependency_injector import providers

from app.models import {Entity}
from app.services.{entity} import {Entity}{Suffix}


@pytest.fixture(autouse=True)
def mock_session_factory():
    mock_session = AsyncMock()
    with patch("app.repositories.{entity}s.session_ctx.get") as mock_get:
        mock_get.return_value = None
        yield


@pytest.fixture
def auth_headers():
    return {"Authorization": "Bearer test-token"}


def _base_payload(**overrides) -> dict:
    defaults = {
        "name": "Test Entity",
    }
    return {**defaults, **overrides}


@pytest.mark.asyncio
async def test_create_{entity}_success(app, app_client, auth_headers):
    payload = _base_payload()
    resp = await app_client.post("/v1/{entity}s", json=payload, headers=auth_headers)
    assert resp.status == 200
```
