# Software Bill of Materials (SBOM)

This SBOM provides a high-level inventory of all dependencies, libraries, and components used in the project, along with
their purposes. It ensures transparency, security tracking, and compliance.

## Project Overview

* **Project Name**: NexusForge
* **Description**: A .NET 8 microservices-based reference system demonstrating best practices for APIs, workers, jobs,
  messaging, security, and observability.
* **Primary Language**: C#
* **Framework**: .NET 8 (LTS)
* **Containerization**: Docker
* **Orchestration**: Docker Compose / Kubernetes (Helm)
* **Auth**: Keycloak (OIDC, JWT)
* **Messaging**: Kafka
* **Databases**: PostgreSQL (per service), Redis (cache, locks, idempotency)
* **Observability**: OpenTelemetry, Prometheus, Grafana, Jaeger, Seq

## Core Runtime

| Component          | Version       | Purpose                                                |
|--------------------|---------------|--------------------------------------------------------|
| .NET SDK / Runtime | 8.0.x         | Main runtime for all services and workers              |
| ASP.NET Core       | 8.0.x         | Web API framework for building HTTP endpoints          |
| Docker Engine      | Latest stable | Container runtime for local and production deployments |
| Docker Compose     | Latest stable | Local orchestration of services and dependencies       |

## Core Libraries & Packages

| Package                                      | Version | Purpose                                                          |
|----------------------------------------------|---------|------------------------------------------------------------------|
| Microsoft.AspNetCore.OpenApi                 | 8.0.8   | OpenAPI/Swagger integration                                      |
| Swashbuckle.AspNetCore                       | 6.7.3   | Swagger UI & OpenAPI generation                                  |
| FluentValidation.AspNetCore                  | 11.9.0  | Model and request validation                                     |
| Npgsql.EntityFrameworkCore.PostgreSQL        | 8.0.4   | EF Core provider for PostgreSQL                                  |
| Microsoft.EntityFrameworkCore.Design         | 8.0.8   | EF Core migrations and tooling                                   |
| Microsoft.EntityFrameworkCore.Relational     | 8.0.8   | EF Core relational DB features                                   |
| Confluent.Kafka                              | 2.5.0   | Kafka client for .NET                                            |
| Serilog.AspNetCore                           | 8.0.1   | Structured logging integration                                   |
| Serilog.Sinks.Console                        | 5.0.1   | Console log sink for Serilog                                     |
| OpenTelemetry.Exporter.OpenTelemetryProtocol | 1.9.0   | OTLP exporter for telemetry data                                 |
| OpenTelemetry.Extensions.Hosting             | 1.9.0   | OpenTelemetry integration for .NET hosting                       |
| OpenTelemetry.Instrumentation.AspNetCore     | 1.9.0   | Auto-instrumentation for ASP.NET Core                            |
| OpenTelemetry.Instrumentation.Http           | 1.9.0   | Auto-instrumentation for HTTP client calls                       |
| Quartz                                       | 3.9.0   | Job scheduling for background workers                            |
| Polly                                        | 8.4.0   | Resilience policies (retry, circuit breaker, timeout)            |
| Mapster                                      | 7.4.0   | Object mapping                                                   |
| StackExchange.Redis                          | 2.8.24  | Redis client library                                             |
| NUnit                                        | 4.1.0   | Unit testing framework                                           |
| Moq                                          | 4.20.72 | Mocking framework for tests                                      |
| FluentAssertions                             | 6.12.0  | Assertion library for tests                                      |
| Testcontainers                               | 3.10.0  | Run dependencies (Postgres, Kafka, etc.) in containers for tests |

## Infrastructure Components

| Component    | Version       | Purpose                                    |
|--------------|---------------|--------------------------------------------|
| PostgreSQL   | Latest stable | Relational database for each microservice  |
| Redis        | Latest stable | Cache, distributed lock, idempotency store |
| Apache Kafka | Latest stable | Event streaming platform                   |
| Keycloak     | Latest stable | Identity and access management             |
| Prometheus   | Latest stable | Metrics collection                         |
| Grafana      | Latest stable | Metrics visualization and dashboards       |
| Jaeger       | Latest stable | Distributed tracing visualization          |
| Seq          | Latest stable | Centralized log management                 |

## Security Considerations

* All dependencies should be scanned for vulnerabilities using `dotnet list package --vulnerable` and container image
  scanners (e.g., Trivy).
* Keycloak and API Gateway enforce authentication/authorization.
* Dependencies are pinned to specific versions to reduce supply chain risk.

## Update & Maintenance Strategy

* Review and update NuGet packages monthly.
* Run `dotnet list package --outdated` to identify updates.
* Rebuild Docker images with latest base images regularly.
* Review SBOM after each release.
