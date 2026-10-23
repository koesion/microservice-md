# MICROSERVICE.md

**A lightweight, AI-ready documentation standard for microservices.**

[![Spec Version](https://img.shields.io/badge/spec-1.1.0-blue)](SPEC.md)
[![License: Apache 2.0](https://img.shields.io/badge/license-Apache%202.0-green)](LICENSE)

MICROSERVICE.md is a single Markdown file — `MICROSERVICE.md`, checked into
the root of a service's repo — that documents what a microservice is
responsible for, how it talks to other systems, what it owns, how it fails,
and how it's operated. It's designed to be read by humans (renders cleanly
on GitHub) and parsed by tools and AI coding agents (fixed, unambiguous
structure).

```markdown
# checkout-service

> Owns the shopping cart lifecycle and coordinates checkout across payment, inventory, and shipping.

**Bounded Context:** Order Management
**Team:** Commerce Platform · #commerce-platform (Slack)

---

## Responsibilities
- Owns the lifecycle of a shopping cart from creation to checkout
- Orchestrates the checkout saga across payment, inventory, and shipping

## Interfaces
### REST API
| Method | Path | Description |
| ------ | ---- | ------------ |
| POST | /carts/{id}/checkout | Start the checkout saga |
...
```

See the [full example](examples/MICROSERVICE.example.md) for a file that
exercises every section.

## Why

Service documentation usually lives in one of three bad places: a wiki page
nobody updates, scattered code comments, or nowhere. MICROSERVICE.md gives
every service exactly one canonical, versioned, human- and machine-readable
document — and because it's just Markdown with a fixed grammar, it's:

- **Diffable** — changes show up in code review like any other file.
- **Round-trippable** — tools can generate it from structured data and
  re-parse hand-edits back into that data without losing information.
- **AI-ready** — a coding agent can read one file and know a service's
  contracts, dependencies, SLOs, and runbook, instead of reverse-engineering
  them from source.

## The spec

The full specification — every section, its exact grammar, and every
enumerated value — is in **[SPEC.md](SPEC.md)**.

## Status

**v1.1.0.** The format is stable. Changes are tracked in
[CHANGELOG.md](CHANGELOG.md) and follow semantic versioning: new optional
sections are minor-version additions; anything that would break parsing of
an existing valid document is a major version.

## Reference implementation

[Microservice.MD](https://microservice.md) is a hosted wizard that
generates, edits, and keeps a `MICROSERVICE.md` in sync with structured
service data using exactly this grammar.

## Contributing

Issues and PRs proposing new optional sections, clarifications, or
corrections to the spec are welcome. Because MICROSERVICE.md files are
meant to be forward-compatible, changes that would break parsing of
existing valid documents need a major version bump and a clear migration
note.

## License

Apache License 2.0 — see [LICENSE](LICENSE). See [NOTICE](NOTICE) for
attribution.
