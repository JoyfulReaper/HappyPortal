# HappyPortal Architecture

## Status

Initial architecture plan. This document describes intended boundaries before the application implementation begins.

## Architectural goals

HappyPortal should be easy to understand, easy to deploy, easy to back up, and difficult to accidentally over-engineer.

The initial system should be a modular monolith:

```text
Browser
  |
  v
HappyPortal ASP.NET Core application
  |
  +--> EF Core --> SQLite
  |
  +--> IFileStorage --> local persistent storage initially
  |
  +--> IPaymentProvider --> hosted external checkout later
  |
  +--> email provider
```

One application process is enough.

## Suggested solution shape

Start small:

```text
HappyPortal.sln

src/
  HappyPortal/

tests/
  HappyPortal.Tests/

docs/
  ARCHITECTURE.md
```

Do not create projects such as Domain/Application/Infrastructure merely to imitate a template.

Split projects only when a boundary has concrete benefits.

## Web UI

Prefer a server-rendered ASP.NET Core UI, likely Razor Pages.

HappyPortal is primarily:

- forms;
- tables;
- detail screens;
- authentication;
- file upload/download;
- invoice/message workflows.

A SPA is not required for those jobs.

Minimal JavaScript is fine where it improves usability.

## Persistence

Use EF Core with SQLite initially.

SQLite is appropriate for the expected workload and keeps deployment/backup simple.

Important rules:

- enable foreign keys;
- use migrations committed to source control;
- use integer minor units for money;
- use currency codes explicitly;
- prefer UTC timestamps;
- avoid database blobs for uploaded files;
- use safe SQLite backup behavior rather than copying an actively written database blindly.

If a future workload genuinely requires PostgreSQL or another database, the application can migrate then.

## Authentication and authorization

Use ASP.NET Core Identity.

Initial role:

- Administrator

Future role:

- Client

Potential later roles:

- Support
- Staff

Do not add a custom `IsAdmin` boolean when the framework already supports roles and policies.

Client access must be scoped through membership/authorization. A URL containing another client's numeric ID must never be sufficient to access that client's data.

Authorization requires integration tests.

## Client membership

Model authenticated users separately from clients.

Conceptually:

```text
ApplicationUser
      |
      v
ClientMembership
      |
      v
Client
```

MVP UI may support one client user, but the schema should not assume one forever.

## Communication model

Keep internal and client-visible communication distinct.

```text
Client
  +--> InternalNotes        admin only
  +--> Conversations
          +--> Messages     client-visible
```

Support tickets may later reuse message/file primitives rather than duplicating them.

## File-storage boundary

Use an interface such as:

```csharp
public interface IFileStorage
{
    Task<StoredFile> SaveAsync(...);
    Task<Stream> OpenReadAsync(...);
    Task DeleteAsync(...);
}
```

The exact API should be designed during implementation rather than copied literally from this example.

Initial implementation can use a Docker persistent volume/local filesystem.

Future implementations could use:

- Cloudflare R2;
- Backblaze B2;
- S3-compatible storage.

HappyPortal stores metadata and authorization relationships, not file bytes in SQLite.

## Billing boundary

HappyPortal owns invoices and payment records.

External providers process money.

```text
HappyPortal Invoice
       |
       v
IPaymentProvider
       |
       v
Hosted checkout
       |
       v
Provider webhook
       |
       v
HappyPortal Payment
```

### Payment provider requirements

A provider adapter should eventually support the minimum necessary operations, likely:

- create hosted checkout/session;
- provide redirect URL;
- validate/process webhook;
- return stable provider IDs/status.

Do not leak a provider's SDK models throughout the domain.

Do not build a huge provider abstraction before implementing the first provider.

### Webhooks

Payment webhooks must be:

- signature-validated;
- idempotent;
- bounded in accepted payload size;
- logged without secrets/card data;
- safe to retry.

A webhook should never trust a client-supplied invoice/payment status.

## Invoice ownership

HappyPortal is the invoice system of record.

Providers do not define invoice identity.

Issued invoices should preserve financial history.

Drafts may change.

Issued invoices should eventually use explicit void/credit/adjustment mechanisms rather than silent historical mutation.

## Email

Keep outbound email behind a small application service.

Initial uses:

- invoice delivery;
- message notification;
- password/account flows where appropriate;
- later renewal/ticket notifications.

Email should point clients back to canonical authenticated portal pages.

Do not build a general mail client.

## Service lifecycle

The Service model requires further design before implementation.

The architecture should expect manual state transitions first and automation later.

A future provisioning boundary may exist, but should not be created until there is an actual service to automate.

## Support tickets

Support tickets are a post-MVP roadmap feature.

They should reuse existing identity/client/message/file concepts where practical.

Do not create a separate standalone ticketing subsystem unless reuse becomes awkward.

## Accounting

Accounting/bookkeeping is deliberately outside the initial architecture.

Clean invoice/payment records should make either future direction possible:

1. add bookkeeping/expenses/accounting later; or
2. integrate/export to dedicated accounting software or hand it to a professional.

Do not introduce ledger abstractions in the MVP merely to prepare for a hypothetical future.

## Dashboard/read models

The admin dashboard can initially query the primary database directly through application query services.

Useful views include:

- client count;
- active services;
- upcoming renewals;
- open/overdue invoices;
- recent payments;
- recent communication/files;
- pending renewal/cancellation requests;
- later open support tickets.

No analytics warehouse is needed.

## Security boundary

HappyPortal will contain private client/business information.

Required before production use:

- HTTPS;
- secure cookies;
- CSRF protection;
- login lockout/rate limits;
- no public registration initially;
- persisted Data Protection keys;
- secrets outside source control;
- production-safe error handling;
- authorization checks/tests;
- upload size/type limits;
- safe generated download responses;
- webhook signature verification;
- encrypted offsite backups;
- restore testing.

Do not store raw card/bank credentials.

## Observability

Keep it modest.

Useful signals:

- application health;
- login failures/lockouts;
- invoice-send failures;
- payment webhook failures;
- storage failures;
- background job failures if/when background work exists.

Do not put client message bodies, file contents, secrets, or unnecessary PII into telemetry.

Mission Control integration can come later if useful.

## Deployment

Expected initial deployment:

```text
portal.kgivler.com
        |
        v
TLS/reverse proxy or Cloudflare
        |
        v
HappyPortal container
        |
        +--> persistent /data
        +--> persisted Data Protection keys
        +--> external secrets/config
```

The actual host remains undecided.

## Backup and restore

Back up:

- consistent SQLite database snapshot;
- uploaded/shared file storage;
- Data Protection keys;
- non-secret recovery configuration/documentation.

Secrets should be restorable through a documented secret-management process without being committed to Git.

A backup plan is incomplete until a restore has been tested.

## Evolution rule

Add architecture only in response to a demonstrated requirement.

Examples:

- PostgreSQL when SQLite is actually limiting;
- object storage when local file storage is actually limiting;
- background worker when work genuinely needs to leave request scope;
- queue/message bus when durable asynchronous integration actually exists;
- separate service when independent deployment/scaling/security boundaries justify it.

Until then, keep HappyPortal boring.
