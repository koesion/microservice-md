# Changelog

All notable changes to the MICROSERVICE.md specification are documented in
this file. This project follows [Semantic Versioning](https://semver.org/):
a new optional section or field is a **minor** release; a change that would
break parsing of an existing, valid document is a **major** release.

## [1.1.0] - 2026-10-22

- New optional `**Catalog ID:**` metadata line (§5.1). It links a
  MICROSERVICE.md to the service's record in a service catalog, so tools
  and AI coding agents in the repository can find that record. Documents
  without it are unchanged.

## [1.0.0] - 2026-08-10

Initial public release of the specification.

- Document structure: header/metadata, `Responsibilities`, `Interfaces`
  (REST API, GraphQL Schema, gRPC Proto, Published/Consumed Events),
  `Data & Dependencies` (Owned Data, Sync/Async Dependencies), `Operations`
  (SLO/SLA, Security, Deployment), `Observability` (Logs, Metrics,
  Distributed Tracing, Dashboards, Alerts), `Architecture Patterns` (DDD
  Aggregates, API Gateway, BFF, CQRS, Sagas, Materialized Views),
  `Runbook`, `Architecture Decision Records`, `Context Pack`.
- Enumerated values for HTTP methods, data store types, dependency
  criticality/type, resilience patterns, deployment platforms, secret
  stores, BFF channels, CQRS roles, saga types, and ADR statuses.
- Extensibility rules for custom sections and forward compatibility.
