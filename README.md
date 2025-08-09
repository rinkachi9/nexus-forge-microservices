# NexusForge

A production-grade, portfolio-friendly **.NET 8 microservices reference system** that demonstrates modern architecture patterns: Clean Architecture per service, event-driven communication (Kafka), outbox/inbox, sagas, background workers, scheduled jobs, API gateway, observability, and secure auth with Keycloak.

> Goal: Ship a realistic, end-to-end system that actually **does something** while showcasing best practices you can reuse in real projects.

---

## Table of Contents

* [Why this project?](#why-this-project)
* [High-level architecture](#high-level-architecture)
* [Domain used](#domain-used)
* [Tech stack](#tech-stack)
* [Service catalog](#service-catalog)
* [Repository layout](#repository-layout)
* [Key guidelines & conventions](#key-guidelines--conventions)
* [Development setup](#development-setup)
* [Local run (Docker)](#local-run-docker)
* [Observability](#observability)
* [Testing strategy](#testing-strategy)
* [Security & auth](#security--auth)
* [Data & persistence](#data--persistence)
* [Messaging & integration](#messaging--integration)
* [Background processing](#background-processing)
* [CI/CD](#cicd)
* [Coding standards](#coding-standards)
* [Roadmap](#roadmap)

---

## Why this project?

* **Practical**: Implements business flows end-to-end (HTTP → domain → DB → events → consumers → projections).
* **Didactic**: Each microservice is a small Clean Architecture slice with Domain, Application, Infrastructure, and API layers.
* **Portable**: Runs locally via Docker Compose; cloud-ready for Kubernetes.
* **Opinionated**: Uses proven defaults (EF Core + Postgres, Kafka, Quartz.NET, YARP, Serilog, OpenTelemetry).

## High-level architecture

* **API Gateway**: YARP forwarding to internal services; central auth/headers.
* **Microservices**: Independent deployables with their own databases.
* **Event-driven**: Kafka for integration and domain event propagation.
* **Transactional Outbox**: Ensures reliable event publishing.
* **Sagas/Process Managers**: Coordinate multi-service workflows.
* **Workers & Jobs**: .NET Worker Services for async tasks; Quartz for schedules.
* **Observability**: OpenTelemetry (traces, metrics, logs) → OTLP → Prometheus/Grafana/Jaeger.

## Domain used

To keep it concrete and slightly challenging, the demo implements a lightweight **Order & Inventory** flow for a small marketplace:

* **Catalog**: Products, categories, pricing.
* **Orders**: Create orders, reserve stock, capture payment (simulated), fulfill.
* **Inventory**: Stock reservations, adjustments.
* **Payments**: Mock payment gateway simulation and idempotency.
* **Notifications**: Email-like outbox + template rendering (console in dev).

> Replaceable: You can swap this domain with your own while keeping the architecture.

## Tech stack

**Runtime**: .NET 8 (C# 12)
**API**: ASP.NET Core Minimal APIs / Controllers + FluentValidation
**Auth**: Keycloak (OpenID Connect, JWT, roles/scopes)
**DB**: PostgreSQL + EF Core, JSONB where sensible
**Cache**: Redis (caching, locks, idempotency keys)
**Messaging**: Kafka (Confluent.Kafka), schema via JSON/Avro (configurable)
**Gateway**: YARP
**Background**: .NET Workers + Quartz.NET
**Observability**: OpenTelemetry (OTLP), Serilog → Console + Seq (dev), Prometheus + Grafana, Jaeger
**Testing**: NUnit, FluentAssertions, Moq, Testcontainers for integration
**Infra**: Docker Compose (local), Kubernetes Helm charts (k8s/), Terraform stubs

## Service catalog

* **api-gateway** (YARP): Single entry point, routing, auth enforcement.
* **svc-catalog**: Products & pricing. Publishes `ProductPriceChanged`, `StockAdjusted` (projection helper).
* **svc-orders**: Order placement, state machine; publishes `OrderPlaced`, `OrderPaid`, `OrderShipped`.
* **svc-inventory**: Stock, reservations; consumes orders, publishes `StockReserved`, `StockRejected`.
* **svc-payments**: Simulated payments with idempotency; publishes `PaymentCaptured`, `PaymentFailed`.
* **svc-notifications**: Listens for significant events, sends emails (console/dev), templates.
* **wrk-ops**: Background ops (sagas timeouts, retries, cleanups, projections rebuilds).

Each service is independently deployable with its own DB schema and migrations.

## Repository layout

```
NexusForge/
├─ src/
│  ├─ api-gateway/                        # YARP gateway
│  ├─ svc-catalog/
│  │  ├─ Catalog.Api/                     # Presentation (controllers/minimal APIs)
│  │  ├─ Catalog.Application/             # Use cases, CQRS handlers, validators
│  │  ├─ Catalog.Domain/                  # Entities, VOs, domain events, aggregates
│  │  ├─ Catalog.Infrastructure/          # EF Core, DBAL, repositories, outbox
│  │  └─ Catalog.Contracts/               # Public contracts (DTOs, events)
│  ├─ svc-orders/
│  │  ├─ Orders.Api/
│  │  ├─ Orders.Application/
│  │  ├─ Orders.Domain/
│  │  ├─ Orders.Infrastructure/
│  │  └─ Orders.Contracts/
│  ├─ svc-inventory/
│  │  ├─ Inventory.Api/
│  │  ├─ Inventory.Application/
│  │  ├─ Inventory.Domain/
│  │  ├─ Inventory.Infrastructure/
│  │  └─ Inventory.Contracts/
│  ├─ svc-payments/
│  │  ├─ Payments.Api/
│  │  ├─ Payments.Application/
│  │  ├─ Payments.Domain/
│  │  ├─ Payments.Infrastructure/
│  │  └─ Payments.Contracts/
│  ├─ svc-notifications/
│  │  ├─ Notifications.Api/
│  │  ├─ Notifications.Application/
│  │  ├─ Notifications.Domain/
│  │  ├─ Notifications.Infrastructure/
│  │  └─ Notifications.Contracts/
│  └─ wrk-ops/                            # .NET Worker + Quartz jobs, consumers
│     ├─ Ops.Worker/
│     └─ Ops.Contracts/
│
├─ building-blocks/                       # Reusable libs (no business logic!)
│  ├─ BuildingBlocks.Messaging/           # Kafka abstractions, consumers, producers
│  ├─ BuildingBlocks.Outbox/              # Outbox helpers, EF interceptors
│  ├─ BuildingBlocks.Observability/       # OTel setup, Serilog sinks
│  ├─ BuildingBlocks.Web/                 # ProblemDetails, exception middleware
│  ├─ BuildingBlocks.Security/            # Auth extensions, Keycloak helpers
│  └─ BuildingBlocks.Testing/             # Test factories, fixtures, containers
│
├─ deploy/
│  ├─ compose/                            # docker-compose for local dev
│  │  ├─ docker-compose.yml
│  │  └─ *.env
│  ├─ k8s/                                # Helm charts & manifests
│  └─ terraform/                          # Cloud infra (optional)
│
├─ ops/
│  ├─ grafana/                            # Dashboards
│  ├─ prometheus/                         # Scrape configs
│  ├─ jaeger/
│  └─ seq/
│
├─ docs/                                  # Architecture decision records, diagrams
│  ├─ adr/
│  ├─ architecture.md
│  └─ sequence-diagrams/
│
├─ tests/
│  ├─ Catalog.Tests.Unit/
│  ├─ Catalog.Tests.Integration/
│  ├─ Orders.Tests.Unit/
│  └─ ...
│
├─ .editorconfig
├─ .gitattributes
├─ .gitignore
└─ README.md                               # this file
```

## Key guidelines & conventions

**Clean Architecture per service**

* **Domain**: Entities, value objects, domain events, specs; no infrastructure.
* **Application**: Use cases (CQRS), validators, orchestrations; no EF Core types.
* **Infrastructure (DBAL)**: EF Core, repositories, outbox, Kafka producers/consumers.
* **API**: Controllers/Minimal API; mapping, OpenAPI, authentication/authorization.

**Microservices rules**

* **Own your data**: One DB per service; no cross-DB joins.
* **Integration via events** (Kafka). Contracts live in `*.Contracts` and are versioned.
* **Outbox** in writers; **Inbox** (idempotency) in consumers.
* **Schema evolution**: Backward-compatible events; deprecate via versioned topics.
* **Sagas**: Long-running workflows in Orders/Ops using timeouts and compensations.

**Other conventions**

* **Idempotency**: Keys stored in Redis; gate payments & order submission.
* **Validation**: FluentValidation + ProblemDetails.
* **Errors**: RFC 7807 ProblemDetails everywhere.
* **Observability**: Trace all incoming HTTP and Kafka messages; correlate with `traceparent`.
* **Config**: `IOptions<T>` + `appsettings.*.json` + env overrides.
* **Testing**: Keep fast unit tests; integration via Testcontainers (Postgres/Kafka/Keycloak).

## Development setup

1. **Requirements**

    * Docker & Docker Compose
    * .NET 8 SDK
    * Node (for some dev tooling) – optional
2. **Restore & build**

   ```bash
   dotnet build
   ```
3. **Spin up infrastructure**

   ```bash
   docker compose -f deploy/compose/docker-compose.yml up -d
   ```
4. **Run services** (dev)

   ```bash
   dotnet run --project src/api-gateway
   dotnet run --project src/svc-catalog/Catalog.Api
   dotnet run --project src/svc-orders/Orders.Api
   # ... etc
   ```

## Local run (Docker)

* Each service has a `Dockerfile` (distroless base in Release).
* Compose brings up Postgres, Kafka, Redis, Keycloak, Grafana, Prometheus, Jaeger, Seq, and all services.
* Make targets (optional): `make up`, `make down`, `make logs`.

## Observability

* **Tracing**: OTel SDK + ASP.NET/Kafka instrumentation → Jaeger.
* **Metrics**: Prometheus scraping; Grafana dashboards in `ops/grafana/`.
* **Logs**: Serilog → Console + Seq (dev). Correlated by traceId/spanId.

## Testing strategy

* **Unit**: Pure domain and application logic.
* **Integration**: EF Core (real Postgres via Testcontainers), Kafka producers/consumers, gateway routing.
* **Contract**: Verify published/consumed events match `Contracts` via schema tests.
* **End-to-end (optional)**: Compose profile that runs headless workflows.

## Security & auth

* **Keycloak** OIDC with JWT bearer on services; roles/scopes per endpoint.
* Gateway enforces auth; internal services re-validate and authorize.
* Service-to-service: client credentials flow; short-lived tokens.

## Data & persistence

* **PostgreSQL** per service; migrations per `Infrastructure` project.
* Prefer **aggregate boundaries** that avoid cross-service transactions.
* Use **Outbox** table and EF Core interceptors to capture domain events and publish reliably.

## Messaging & integration

* **Kafka** topics per event type and version (e.g., `orders.v1.events`).
* Consumers are **idempotent** and store processed offsets/ids.
* **Dead-letter** strategy for poison messages + retries with exponential backoff.

## Background processing

* **.NET Workers** host consumers, projections, and jobs.
* **Quartz.NET** for schedules (e.g., nightly reconciliations, saga timeouts).
* **Resilience**: Polly for retries/bulkhead/timeouts; circuit-breakers on external calls.

## CI/CD

* **GitHub Actions** (examples provided):

    * PR: build, unit + integration tests, container build, Trivy scan.
    * Main: version bump (GitVersion), push images, deploy to k8s (optional).
* **Security**: `dotnet list package --vulnerable` gate; `dotnet format` check.

## Coding standards

* `.editorconfig` with C# analyzers, nullable enabled, warnings-as-errors in CI.
* Directory.Packages.props for central package management.

## Roadmap

* [ ] Implement first cohesive flow: Create Order → Reserve Stock → Capture Payment → Ship → Notify.
* [ ] Add projections/read models for order status in `svc-orders`.
* [ ] Introduce inbox pattern for consumers.
* [ ] Add YARP rate limiting & request throttling.
* [ ] Add Avro schemas & schema registry option.
* [ ] Add Helm charts per service.
* [ ] Add blue/green and canary samples in k8s.
* [ ] Add API examples & Postman collection.

---

### Notes on DBAL vs Domain

* Keep **DBAL in Infrastructure** (EF Core, mapping, migrations). Domain remains persistence-agnostic.
* Use repositories for **aggregate roots** only; prefer direct query handlers (via Dapper or EF Core compiled queries) for read models.
* Avoid sharing Infrastructure across services—share **building blocks** only (pure helpers, abstractions, OTel setup), never DB schemas.

### Naming & namespaces

* Root namespace: `NexusForge` (e.g., `NexusForge.Orders.Domain`).
* Contracts are versioned and explicitly referenced across services.

---

> **Tip**: Treat this repo as a pattern library. Copy slices into real projects or extend the domain here for your portfolio demos.
