# AGENTS.md

## What HappyPortal is

HappyPortal is a source-visible proprietary client/service portal built by Kyle Givler.

It serves two purposes:

1. operate Kyle's own client, service, billing, communication, and file-sharing workflows; and
2. demonstrate and potentially sell/license the kind of custom software Kyle builds.

Production hostname is planned as `portal.kgivler.com`.

The public development-services site is separate at `dev.kgivler.com`.

Read these before making significant changes:

- `PORTAL_PLAN.md`
- `docs/ARCHITECTURE.md`
- `LICENSE`

## Development branch

Active development should happen on `dev` once that branch exists.

Do not assume `main` is the active development branch.

## Core engineering rule

Prefer the simplest implementation that satisfies the current phase.

Do not introduce:

- microservices;
- message queues;
- distributed architecture;
- CQRS/event sourcing;
- generic repository/unit-of-work wrappers around EF Core;
- plugin systems;
- separate SPA/frontend and backend applications;
- deep Clean Architecture project trees;
- abstractions without an actual second implementation or near-term boundary

unless a concrete requirement justifies them.

This project should be a boring modular monolith for as long as that remains the right engineering decision.

## Current technical direction

Preferred initial stack:

- .NET / ASP.NET Core
- Razor Pages or similarly simple server-rendered UI
- EF Core
- SQLite
- ASP.NET Core Identity
- Docker

Expected replaceable boundaries:

- payment provider
- file storage
- outbound email

Keep those abstractions narrow. Do not design for ten hypothetical vendors before implementing one.

## Domain rules

### Clients

Clients may eventually have multiple authenticated users.

Do not encode a permanent one-user-per-client assumption.

Use a membership relationship when client authentication is implemented.

### Notes vs messages

Internal notes are admin-only.

Messages/conversations are client-visible.

Do not accidentally expose internal notes through client APIs/pages/timeline queries.

### Services

The Service model is intentionally not finalized.

Before implementing a large service hierarchy or complex schema, revisit the design in `PORTAL_PLAN.md`.

Manual lifecycle actions come before automated provisioning.

### Invoices

HappyPortal is planned to be the authoritative invoice system.

- drafts can be edited;
- issued financial history should not be silently rewritten;
- invoices should have line items;
- payments are separate records;
- money should use integer minor units plus currency code.

Do not make an external payment provider the source of truth for invoice identity.

### Payments

Payment providers must sit behind a narrow application interface.

HappyPortal must never store raw card credentials.

Hosted checkout and signed/idempotent webhook processing are preferred.

### Files

Both admin and clients will be able to share files.

Store file metadata in the database and bytes behind a storage abstraction.

Do not put arbitrary uploaded file blobs into SQLite.

Every upload/download path requires authorization.

### Support tickets

Support tickets are on the roadmap but are not required for the first MVP.

Prefer reusing conversation/message/file primitives where clean.

### Accounting

Bookkeeping/accounting and expense tracking are later possibilities, not MVP requirements.

Do not prematurely turn invoice/payment models into a half-built general ledger.

## Authentication and authorization

Use ASP.NET Core Identity and roles/policies.

Do not add a custom `IsAdmin` flag merely to avoid using framework roles.

Initial production use is single-admin.

Public customer registration is a later roadmap item.

Authorization boundaries are high-risk and require tests.

A client must never gain access to another client's:

- messages;
- files;
- services;
- invoices;
- payments;
- tickets;
- notes.

Internal notes are never client-visible.

## Data and history

Prefer archive/cancel/void state transitions over destructive deletion of meaningful business history.

Use:

- UTC timestamps;
- foreign keys;
- migrations in source control;
- explicit currency;
- integer minor units for money.

Be careful with SQLite concurrency and backup behavior.

## Security

Before storing real client data:

- HTTPS must be enforced at the deployment boundary;
- secure cookie settings must be correct;
- CSRF protections must remain enabled;
- login protections/lockout must exist;
- secrets must be external to Git;
- Data Protection keys must persist;
- file uploads must be bounded and validated;
- payment webhooks must be signature-validated and idempotent;
- production errors must not leak internals;
- backups must be encrypted/offsite;
- restore testing must have succeeded.

Avoid logging sensitive client content.

## Testing priorities

Prefer tests around behavior with real business/security consequences:

- authorization boundaries;
- invoice state transitions;
- payment idempotency;
- client membership;
- file ownership/access;
- internal-note visibility;
- service renewal/cancellation requests;
- provider/webhook failure behavior.

Do not chase meaningless coverage percentages.

## UI

The portal should be conventional, professional, and boring in a good way.

This is not the terminal-style `kgivler.com` homepage.

Favor:

- readable forms;
- clear tables;
- obvious status labels;
- responsive layouts;
- accessible controls;
- clear admin/client separation.

Avoid unnecessary JavaScript and UI framework churn.

## Commercial/source-visible constraints

HappyPortal is not open source.

Do not replace the custom LICENSE with MIT, BSD, GPL, Apache, or another open-source license without an explicit decision from Kyle.

Do not copy third-party code into this repository unless its license is compatible with HappyPortal's proprietary distribution model.

Keep third-party notices/licenses where required.

The source-visible license may change for future releases, but changes do not automatically rewrite licenses attached to earlier lawfully obtained versions.

## Scope discipline

Before adding a feature, ask whether it belongs to:

- MVP;
- near-term roadmap;
- later roadmap;
- another product entirely.

Things explicitly not required for the initial implementation include:

- full accounting;
- expense tracking;
- automatic provisioning;
- customer self-registration;
- support tickets;
- complex RBAC;
- multi-company SaaS tenancy;
- mobile app;
- generic public API;
- tax engine;
- bank sync.

Some are planned later. "Planned later" is not permission to scaffold them today.

## Documentation

When architectural/product decisions change, update the relevant documentation.

Prefer keeping the plans honest over preserving stale prose.

Do not claim a feature exists merely because it is on the roadmap.
