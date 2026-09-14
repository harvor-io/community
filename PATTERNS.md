# Harvor Code Patterns

This document describes the code-level patterns Harvor codebases follow by
default. Where [`FEATURES.md`](FEATURES.md) describes what a service should do
and [`CONTRIBUTING.md`](CONTRIBUTING.md) sets Go and React as the standard
languages, this document describes how code within those languages should be
structured so that services stay consistent, testable, and able to support the
range of backing technologies self-hosters run.

These are defaults, not mandates — see the [Exceptions](#exceptions) note at
the end.

---

## Adapter pattern for external dependencies

Anything a service depends on outside its own process — a database, cache,
object store, queue/broker, or third-party API — sits behind an interface
owned by the code that uses it, not by the dependency's own SDK. Callers
depend only on that interface; a concrete implementation satisfies it.

Self-hosters run Harvor against many different backing technologies (Postgres
or MySQL, Redis or an in-memory cache, S3 or GCS or local disk). Coding
directly against a specific client's concrete types spreads that dependency
throughout the codebase and makes swapping or testing painful. An adapter
interface keeps a swap — or a test double — confined to a single
implementation.

### Package layout

The interface and any shared/internal types live in the package named after
the thing being adapted. Each concrete implementation gets its own
subpackage, named after the implementation, containing the code that
satisfies the interface:

```text
<thing>/<thing>.go               # interface + internal types
<thing>/<implementation>/<thing>.go   # concrete implementation of the interface
```

For example, a cache adapter with a Redis-backed implementation:

```text
cache/
├── cache.go        # Cache interface, Item type, ErrMiss, etc.
└── redis/
    └── cache.go     # redis.Cache — implements cache.Cache using go-redis
```

Callers import `cache` and depend only on `cache.Cache`. Only `main`, or
wherever the service wires its dependencies together, imports the concrete
`cache/redis` package.

## Repository pattern for persistence

The repository pattern is the adapter pattern applied to storage. A
`Repository` interface expresses the operations a service needs in domain
terms (e.g. `GetEvent(ctx, id) (*Event, error)`), not in SQL or ORM terms.
Concrete repositories (Postgres, etc.) implement that interface.

Repository methods return domain types, not database rows or ORM models, so
callers are never coupled to the storage schema's representation.

## Service layer pattern

Business logic lives in a service layer — not in HTTP handlers, and not in
repositories.

- **Handlers** parse and validate the transport-level request, call a service
  method, and translate the result (including errors) into a response. They
  contain no business logic.
- **Services** contain business logic and orchestration. They call one or
  more repositories/adapters and know nothing about HTTP, gRPC, or any other
  transport.
- **Repositories/adapters** talk to the outside world and know nothing about
  business rules.

This keeps business logic testable without standing up a transport layer, and
reusable across transports (HTTP, gRPC, a CLI) without duplicating it.

## Errors

Repositories and services return **typed errors**, not raw `fmt.Errorf`
strings or driver-specific errors, so calling code can check for them
programmatically (`errors.Is` / `errors.As`) instead of matching on error
text.

For example, a repository defines `ErrNotFound`; a service propagates or
wraps it; the HTTP handler maps `ErrNotFound` to `404`, `ErrConflict` to
`409`, and so on. This keeps the mapping from domain error to transport
status code in one place — the handler — and keeps repositories and services
transport-agnostic.

Wrap errors with context as they cross layers (`fmt.Errorf("...: %w", err)`)
so the typed error underneath stays reachable via `errors.Is`/`errors.As`,
and failures stay diagnosable from logs.

## REST API conventions

- **Use the correct HTTP verb for the operation:**
  - `GET` — read, no side effects, safe to cache.
  - `POST` — create a resource, or trigger a non-idempotent action.
  - `PUT` — full replace of a resource; the caller supplies the complete
    representation.
  - `PATCH` — partial update; the caller supplies only the fields that are
    changing.
  - `DELETE` — remove a resource.

  Don't use `PUT` for a partial update, or `POST` where `PUT`/`PATCH` is
  correct.

- **URLs are plural, resource-oriented nouns:** `/events`, `/events/:id` —
  not `/event` or `/getEvents`.

- **kebab-case for multi-word resource names:** `/event-types`, not
  `/eventTypes` or `/event_types`.

- **Nest at most one level deep.** Once you have a resource's own id, that id
  is sufficient to address anything below it directly — don't keep nesting
  parents on top of it. `GET /events/:id/attendees` is fine; `GET
  /events/:id/attendees/:attendeeId/comments` is not — once you have the
  attendee id, address its comments directly (`GET
  /attendees/:attendeeId/comments`).

- **JSON response bodies contain only the resource(s) being returned** — no
  pagination info, request IDs, or other metadata mixed into the payload.
  Metadata belongs in response headers (e.g. cursor/link headers for
  pagination, `X-Request-Id`).

- **Error responses use the correct HTTP status code and include a
  human-readable message.** See [`FEATURES.md`](FEATURES.md) for the full
  error envelope shape services should follow.

## Standard packages (Go)

Unless a repository has a documented reason to differ, Harvor Go services
use:

| Purpose | Package |
| --- | --- |
| HTTP routing | [`go-chi/chi`](https://github.com/go-chi/chi) |
| Request/response rendering | [`go-chi/render`](https://github.com/go-chi/render) |
| IDs | [`gofrs/uuid`](https://github.com/gofrs/uuid) (UUIDv7, for sortable, time-ordered identifiers) |
| CLI entrypoints | [`urfave/cli`](https://github.com/urfave/cli) |

Standard choices for logging, configuration, database access, migrations,
and testing are not yet documented here — until they are, follow the
convention already established in the repository you're contributing to, and
raise it in a Discussion if you think it should become the Harvor-wide
default.

## Exceptions

These patterns are defaults, not mandates. There are legitimate reasons to
deviate — a constraint a pattern doesn't serve well, a case where the
indirection isn't earning its cost. If you want to diverge from something
described here, explain why in your issue, discussion, or pull request; as
with the [technology stack](CONTRIBUTING.md#technology-standards),
exceptions are considered case by case rather than assumed.

---

**Infrastructure you can trust.**
