# Harvor Service Standards

Harvor provides open-source, self-hostable building blocks for the infrastructure
modern applications need. For those building blocks to be trustworthy, every
Harvor service should aim for the same baseline of quality regardless of what it
does.

This document is the high-level, cross-cutting feature list that each Harvor
service strives to implement. It is a target to design and review against, not a
gate that every service must fully clear before its first release. Individual
services may track their progress against this list in their own repositories.

The goals, in short:

- **Dependable** — behaves predictably under load and failure, loses no data, and
  upgrades without downtime.
- **Easy to use** — a developer can stand it up, understand it, and integrate
  with it quickly, with good defaults and clear errors.
- **Observable** — its health, behavior, and failures are visible from the
  outside using open standards.
- **Production ready** — it can be deployed, operated, secured, and upgraded by a
  small team using automation.

---

## 1. APIs and interfaces

- **RESTful HTTP APIs** using standard conventions: correct verbs and status
  codes, resource-oriented URLs, JSON request/response bodies, `Content-Type`
  negotiation.
- **Consistent envelope and error format** across all services — a single
  documented shape for errors (e.g. RFC 9457 / `application/problem+json`) with
  stable machine-readable error codes and human-readable detail.
- **Idempotency on writes** — mutating endpoints accept an idempotency key and
  guarantee exactly-once effect on retry; naturally idempotent verbs (`PUT`,
  `DELETE`) behave accordingly.
- **Optimistic concurrency control** via `ETag` / `If-Match` on updates.
- **Consistent pagination, filtering, sorting, and field selection** — cursor
  based pagination by default; documented limits.
- **API versioning and compatibility policy** — explicit versioning scheme,
  additive-change guarantees, a published deprecation and sunset policy.
- **Published, accurate API specification** — OpenAPI (and Protobuf/gRPC or
  AsyncAPI where relevant) generated from or verified against the implementation.
- **Generated SDKs / client libraries** for major languages, plus a CLI for
  common operations.
- **Rate limiting and quotas** with standard response headers
  (`RateLimit-*`, `Retry-After`).
- **Asynchronous and bulk operations** — long-running work exposed as jobs with
  status; batch endpoints where they prevent N+1 call patterns.
- **Events and webhooks** — outbound events published on a standard schema
  (e.g. CloudEvents), webhooks signed, retried with backoff, and replayable.
- **Backwards and forwards compatible payloads** — unknown fields tolerated,
  enums extensible.

## 2. Dependability and resilience

- **Stateless service processes** — all durable state lives in backing stores;
  any instance can serve any request; scale horizontally.
- **Health endpoints** — separate liveness, readiness, and startup checks;
  readiness reflects real dependency health.
- **Graceful startup and shutdown** — drain in-flight requests, stop accepting
  new work, release leases and connections cleanly; tolerate dependencies coming
  up in any order.
- **Resilient outbound calls** — timeouts on everything, retries with exponential
  backoff and jitter, circuit breakers, and bounded connection pools.
- **Backpressure and load shedding** — concurrency limits, request queue bounds,
  and early rejection with `429`/`503` instead of collapse.
- **Retry-safe processing** — at-least-once semantics with deduplication; the
  transactional outbox pattern for reliable event publication.
- **Zero-downtime schema migrations** — versioned, expand/contract style,
  reversible, safe to run while old and new code run concurrently.
- **Zero-downtime deploys** — rolling, blue/green, or canary; old and new
  versions interoperate during rollout.
- **Data durability** — documented backup strategy, automated backups, regularly
  tested restores, point-in-time recovery where the datastore supports it.
- **Defined RTO/RPO and a disaster-recovery runbook**; multi-AZ deployment
  supported.
- **No unbounded work** — every query, list, loop, and buffer has a limit.
- **Failure testing** — dependency-failure and chaos scenarios exercised in CI or
  staging.

## 3. Observability

- **OpenTelemetry as the instrumentation standard** for traces, metrics, and
  logs, exported via OTLP and configurable to any collector.
- **Distributed tracing** across service boundaries with W3C Trace Context
  propagation; spans for inbound requests, outbound calls, and queue processing.
- **Structured logs** (JSON) with severity levels, correlation/trace IDs, and no
  secrets or PII; log verbosity configurable at runtime.
- **Metrics** following RED (rate, errors, duration) for interfaces and USE
  (utilization, saturation, errors) for resources; Prometheus-compatible
  scrape endpoint in addition to OTLP.
- **Correlation** — a single request/trace ID ties together logs, metrics
  exemplars, and traces.
- **Service metadata endpoint** — version, build, commit, and effective
  (non-secret) configuration.
- **Audit logging** for security- and tenant-relevant actions, separable from
  operational logs.
- **Shipped operational assets** — reference dashboards, SLO definitions, and
  alert rules (including burn-rate alerts) delivered as code alongside the
  service, with runbooks linked from alerts.

## 4. Configuration and provisioning (configuration as code)

- **Twelve-factor configuration** — environment variables and/or mounted config
  files; no environment-specific values baked into images.
- **Declarative deployment artifacts maintained in-tree** — an official Helm
  chart and/or Kubernetes manifests, a Terraform module, and a Docker Compose
  file for local and small deployments.
- **Secure, working defaults** — the service starts and runs correctly with
  minimal configuration; every option is documented with its default.
- **Config validation at startup** — fail fast with a clear message on invalid or
  missing required configuration; never start half-configured.
- **Secrets from a secrets manager** — integration with Kubernetes Secrets,
  Vault, and cloud secret stores; secrets never logged or echoed by the metadata
  endpoint.
- **Automated provisioning of dependencies** — schema creation and migration on
  startup or via a documented job; required buckets, topics, and roles created
  by the provided IaC.
- **Reproducible, pinned builds** — locked dependencies, deterministic image
  builds.

## 5. Packaging and deployment

- **OCI container images** — minimal/distroless base, multi-arch (amd64/arm64),
  runs as non-root with a read-only root filesystem and dropped capabilities.
- **Supply-chain integrity** — signed images (e.g. cosign), published SBOM, and
  build provenance/attestation (SLSA-aligned).
- **Kubernetes-friendly** — resource requests/limits, PodDisruptionBudgets,
  HorizontalPodAutoscaler support, NetworkPolicies, and topology spread in the
  provided manifests; an operator/CRDs only where they add real value.
- **Small-footprint deployment** — runs on a single node via Compose for
  self-hosters who don't need Kubernetes.
- **Versioned releases** — semantic versioning, a maintained changelog, signed
  release artifacts with checksums, and a stated support/compatibility window.
- **Tested upgrades and rollbacks** — every supported version transition is
  exercised, including migration rollback.

## 6. Security and compliance

- **Encryption in transit** (TLS) for all external and inter-service traffic;
  optional mTLS between services.
- **Encryption at rest** for datastores and backups, using pluggable keys.
- **Standard authentication and authorization** — OIDC/OAuth 2.0, API keys, and
  service identities; least-privilege RBAC/ABAC with scoped tokens.
- **Strict input validation and output encoding**; protection against the OWASP
  Top 10.
- **Automated security scanning in CI** — dependency, container, secret, SAST,
  and (where feasible) DAST scans.
- **Vulnerability response policy** — a security disclosure process and a
  committed remediation SLA.
- **Multi-tenancy isolation** where the service is multi-tenant — enforced at the
  data layer, not just the API.
- **Data governance** — documented retention, export, and deletion (including
  right-to-erasure); PII identified and minimized.

## 7. Data management and portability

- **Explicit, documented data model** and clear ownership of each store.
- **Bundled migration tooling** — forward and backward migrations runnable as a
  standalone command.
- **No lock-in** — documented, scriptable backup/restore and full data
  import/export in open formats.
- **Referential integrity and validation** enforced close to the data.
- **Pluggable backends** where practical (storage, queue, database) behind stable
  interfaces, using standard protocols (S3 API, AMQP/Kafka, SQL).

## 8. Developer experience

- **One-command local startup** — `docker compose up` (or equivalent) yields a
  working instance with seed data.
- **Fast quickstart** — a new user reaches a working "hello world" in minutes,
  following a single guide.
- **Complete documentation** — concepts, task-oriented how-tos, full API
  reference, self-hosting/operations guide, and reference architectures.
- **Actionable errors** — every error has a stable code, a plain-language
  explanation, and a suggested remediation; a searchable error catalog.
- **Sizing and performance guidance** — documented throughput, latency, and
  resource characteristics with a capacity-planning guide.
- **Feature flags** for progressive delivery of risky changes.
- **Demo/sandbox mode** for evaluation without external dependencies.

## 9. Quality and release engineering

- **Layered automated tests** — unit, integration, contract (API and event),
  and end-to-end — run on every change.
- **Contract tests** guard API and event-schema compatibility against published
  specs.
- **Load and soak tests** for the primary paths, run before release.
- **Automated release pipeline** — build, sign, publish, and deploy with canary
  analysis and automatic rollback triggers.
- **Compatibility matrix testing** across supported datastore and dependency
  versions.

---

## Per-service readiness checklist

A compact view of the above for design reviews and release sign-off:

- [ ] REST API follows shared conventions and error format
- [ ] Writes are idempotent; concurrency is controlled with ETags
- [ ] OpenAPI/AsyncAPI spec published and verified in CI
- [ ] SDKs and CLI available for common operations
- [ ] Stateless; scales horizontally; liveness/readiness/startup probes
- [ ] Graceful shutdown; timeouts, retries, and circuit breakers on all I/O
- [ ] Backpressure / load shedding under overload
- [ ] Zero-downtime, reversible migrations; zero-downtime deploys
- [ ] Automated, restore-tested backups; documented RTO/RPO and DR runbook
- [ ] OpenTelemetry traces, metrics, and logs; Prometheus endpoint
- [ ] Structured logs with trace correlation; no secrets/PII
- [ ] Dashboards, SLOs, and alerts shipped as code with runbooks
- [ ] Audit log for security-relevant actions
- [ ] 12-factor config; validated at startup; secure defaults
- [ ] Helm chart, Terraform module, and Compose file maintained in-tree
- [ ] Secrets sourced from a secrets manager
- [ ] Multi-arch, non-root, distroless container; signed; SBOM; provenance
- [ ] TLS everywhere; standard authn/authz; least-privilege RBAC
- [ ] Dependency/container/secret/SAST scans in CI; disclosure policy
- [ ] Tenant isolation enforced at the data layer (if multi-tenant)
- [ ] Data export/import and documented restore; bundled migration tool
- [ ] `docker compose up` gives a working instance with seed data
- [ ] Docs: concepts, how-tos, API reference, operations guide
- [ ] Actionable errors with stable codes and remediation
- [ ] Semantic versioning, changelog, signed artifacts, tested upgrades/rollbacks
- [ ] Unit, integration, contract, e2e, and load tests in CI
