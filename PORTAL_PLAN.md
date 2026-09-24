# HappyPortal Plan

## Purpose

HappyPortal is Kyle Givler's client/service portal and a commercial software showcase.

It is intended to:

- manage Kyle's real clients, services, invoices, payments, communication, and shared files;
- give clients a useful authenticated place to see and act on their relationship with Kyle;
- provide an administrative overview of the business;
- serve as a public source-visible example of the kind of custom .NET software Kyle can build;
- be licensable/customizable for customers who want a similar portal or a white-glove implementation.

The goal is not to build a generic enterprise CRM, accounting suite, or hosting control panel before there is a real need.

Production hostname: `portal.kgivler.com`.

The public development-services site remains separate at `dev.kgivler.com`.

## Product principles

1. Start with Kyle's actual workflows.
2. Prefer boring, understandable architecture.
3. Keep financial history auditable.
4. Keep payment providers replaceable.
5. Keep file storage replaceable.
6. Do not store raw payment-card data.
7. Favor explicit lifecycle/history over destructive deletion.
8. Build customer automation only after the manual workflow is understood.
9. Keep future multi-user clients possible without exposing the complexity in the MVP.
10. Treat the source-visible repository as both engineering transparency and a commercial portfolio/product demonstration.

## Roles and users

### MVP

- one Administrator: Kyle;
- client accounts are created manually;
- each client effectively has one visible login initially;
- no public registration.

### Model for later

The domain should support multiple users belonging to one client through a membership relationship rather than hard-coding a single user onto a client.

Likely roles eventually include:

- Administrator
- Client

Future staff/support roles may be added if there is a real use case.

Use ASP.NET Core Identity roles/policies rather than a custom `IsAdmin` flag.

## Core concepts

### Client

Represents a customer or customer organization.

Likely fields:

- ID
- display/legal name
- primary contact information
- status
- notes/summary
- created/updated timestamps

Do not over-model contacts in the MVP, but keep the model compatible with multiple client users later.

### Client membership

Associates an authenticated user with a client.

MVP may expose only one member per client while preserving the ability to support multiple users later.

### Internal note

Administrative note visible only to Kyle/admin users.

Examples:

- preferences;
- deployment notes;
- commercial context;
- reminders;
- decisions from off-portal conversations.

Internal notes are distinct from client-visible messages.

### Conversation and message

Client-visible communication inside HappyPortal.

Both Kyle and clients can participate.

Email notifications may notify a user that a message exists and link back to the portal. HappyPortal should not become a general-purpose email client.

A client timeline may later combine messages with other events without forcing every event type into one database table.

### File

Both Kyle and clients must eventually be able to upload/share files.

The file metadata belongs to HappyPortal; file bytes should sit behind an `IFileStorage` abstraction so local storage can be used initially and object storage can be added later.

Files need:

- owning client;
- uploader;
- display/original filename;
- content type;
- size;
- storage key;
- created timestamp;
- visibility/access rules;
- optional relationship to a conversation, invoice, service, or ticket later.

File uploads require conservative size/type limits and authorization checks.

### Service

Represents something Kyle provides to a client.

Examples may include:

- website development;
- hosting;
- maintenance;
- custom software;
- .NET/backend/API work;
- automation/integrations;
- Docker/VPS/self-hosting work;
- debugging/code review;
- consulting/mentoring;
- DNS or infrastructure work;
- other bespoke services.

The exact service schema should be discussed before implementation.

The model should support:

- active/inactive/cancelled lifecycle;
- start date;
- renewal/end date;
- billing relationship;
- human-readable description;
- notes;
- renewal/cancellation requests.

Avoid a deep class hierarchy for every possible service type unless real requirements demand one.

### Renewal/cancellation

MVP behavior is manual:

1. client requests renewal or cancellation;
2. request is recorded;
3. Kyle reviews it;
4. Kyle performs any infrastructure/business action;
5. service state is updated.

Later this may become automated behind explicit policies and provisioning integrations.

### Invoice

HappyPortal should be the authoritative invoice system unless future legal/accounting requirements force a different boundary.

HappyPortal owns:

- invoice number;
- client;
- line items;
- issue/due dates;
- currency;
- subtotal/total;
- invoice status;
- history;
- relationship to payments.

Draft invoices may be edited.

Once issued, financial contents should not be silently rewritten. Corrections should eventually use voiding/adjustments/credits rather than erasing history.

Initial statuses:

- Draft
- Open
- Paid
- Void

### Invoice line item

Invoices should support line items from the beginning rather than one opaque invoice amount.

Likely fields:

- description;
- quantity;
- unit amount in minor currency units;
- computed line total;
- optional service reference.

Tax/discount handling is deliberately deferred until requirements are understood.

### Payment

Payments are first-class records and do not collapse into an `Invoice.IsPaid` flag.

Likely fields:

- client;
- optional invoice;
- amount in minor currency units;
- currency;
- paid timestamp;
- method/provider;
- external transaction/reference ID;
- notes.

Manual payments are supported from the start.

### Payment provider

Hosted/online payment flows live behind an application interface such as `IPaymentProvider`.

The provider is replaceable.

HappyPortal should:

- create or request a hosted checkout/payment session;
- redirect the client to the provider;
- validate signed provider webhooks/callbacks;
- record resulting payment data;
- update invoice state idempotently.

HappyPortal should not receive or store raw card credentials.

Provider selection is deliberately undecided.

### Support tickets

Support tickets are on the roadmap but are not required for the first MVP.

They should eventually support:

- client-created tickets;
- admin-created tickets;
- status and priority;
- threaded messages;
- file attachments;
- optional relationship to a service;
- history/timestamps.

Do not build a separate communication engine for tickets if the existing conversation/message model can be reused cleanly.

### Activity/timeline

A client detail page should eventually provide a useful chronological view combining important events such as:

- messages;
- invoice issue/payment;
- file sharing;
- service creation/renewal/cancellation;
- support-ticket activity;
- admin-visible events where appropriate.

This is primarily a UX/query concept initially. Do not create an event-sourcing system merely to render a timeline.

## Admin experience

Kyle needs an administrative view of:

- clients;
- active/inactive services;
- upcoming renewals;
- open/overdue invoices;
- recent payments;
- renewal/cancellation requests;
- recent client communication;
- recently shared files;
- later support tickets;
- basic revenue/service/client counts.

The first dashboard should answer "what needs my attention?" before trying to become a business-intelligence product.

## Client experience

Clients should eventually be able to:

- log in;
- see their services and relevant dates;
- see invoices and payment history;
- pay an invoice through a hosted payment flow;
- request renewal/cancellation;
- read and send messages;
- upload/download shared files;
- see their relevant communication/history;
- later create and manage support tickets;
- later manage multiple users if enabled.

The UI should be conventional, clear, and professional.

## Invoice delivery

HappyPortal should eventually send invoice email.

An initial invoice email can contain:

- invoice number;
- client name;
- total;
- due date;
- a clean HTML representation/summary;
- a secure link to the canonical invoice page in HappyPortal.

PDF generation is optional and can come later.

## MVP

The first useful version should focus on an internal/admin workflow plus a small but real client experience.

### MVP candidate scope

- ASP.NET Core Identity;
- Administrator role;
- manually created client accounts;
- client records;
- internal client notes;
- services;
- invoices and line items;
- manual payment recording;
- admin dashboard;
- client login;
- client service view;
- client invoice/payment view;
- client-visible conversations/messages;
- two-way file sharing;
- manual renewal/cancellation requests;
- invoice email/link flow;
- basic audit timestamps/history.

Hosted payment checkout may land in the MVP if it remains small, but it is acceptable as the first post-MVP feature.

## Post-MVP roadmap

### Near-term

- hosted online payment provider;
- signed webhook processing;
- renewal reminders;
- stronger activity/timeline view;
- richer dashboard stats;
- support tickets;
- multiple users per client;
- more useful email notifications;
- file-storage provider beyond local disk;
- invoice PDF generation if useful.

### Later

- automatic recurring invoices;
- automatic renewals;
- provisioning/service automation;
- customer self-registration;
- staff/support roles;
- bookkeeping/accounting;
- expense tracking;
- vendor records;
- exports/integration with external accounting software;
- taxes/discounts/credits where required;
- operational integration with Mission Control where that creates real value;
- optional APIs/integrations.

Accounting may be built into HappyPortal later or abandoned in favor of dedicated accounting software/professional bookkeeping. The portal should keep invoice/payment history clean enough to support either outcome.

## Explicit non-goals for the initial implementation

Do not build these just because they sound adjacent:

- general-purpose CRM;
- full double-entry accounting;
- expense tracking;
- tax engine;
- payroll;
- bank synchronization;
- arbitrary workflow engine;
- multi-company SaaS tenancy;
- automatic infrastructure provisioning;
- automatic service suspension;
- domain registrar;
- email hosting;
- broad public API;
- mobile app;
- live chat;
- social/OAuth login;
- complex RBAC;
- generic plugin framework;
- message bus;
- microservices;
- separate SPA frontend.

## Technical direction

Initial preference:

- .NET / ASP.NET Core
- Razor Pages or similarly boring server-rendered UI
- EF Core
- SQLite
- ASP.NET Core Identity
- Docker
- HTTPS at the edge/reverse proxy
- local file storage behind `IFileStorage` initially
- payment flow behind `IPaymentProvider`
- server-side email abstraction/provider
- unit/integration tests around authorization, billing state, and provider boundaries

Do not split the application into multiple deployable services until there is a concrete operational reason.

## Data and financial rules

- Store money as integer minor units plus currency code.
- Prefer UTC for timestamps; render user-facing local dates/times appropriately.
- Use foreign keys.
- Preserve issued invoice/payment history.
- Archive/cancel records instead of casually deleting business history.
- Do not log secrets, payment credentials, private file contents, or sensitive message bodies unnecessarily.
- Use explicit authorization checks on every client-owned resource.

## Deployment

Production target:

`portal.kgivler.com`

Likely deployment:

`Internet -> HTTPS/reverse proxy or Cloudflare -> HappyPortal container -> SQLite/data volume`

Exact host is deliberately undecided.

Production configuration and secrets must remain outside the repository.

Persist ASP.NET Core Data Protection keys.

## Backup requirements

Before real client data is trusted to HappyPortal:

- database backup must be consistent with SQLite;
- file storage must be backed up;
- Data Protection keys and required recovery configuration must be included;
- secrets must be documented for recovery without committing them to Git;
- encrypted offsite backup should exist;
- at least one real restore test must succeed.

## Source-visible / commercial model

HappyPortal is source-visible proprietary software, not open source.

The public code exists to:

- demonstrate implementation quality;
- make architecture/security practices inspectable;
- provide a concrete example of work Kyle can sell;
- allow prospective customers to evaluate HappyPortal as a licensable/customizable product.

Separate commercial licenses may grant permission to run, customize, deploy, host, redistribute, or otherwise use HappyPortal.

Commercial work may also include white-glove setup, customization, integrations, hosting, maintenance, and support.

The public license may change for future releases. License changes for future versions do not automatically rewrite the license attached to copies of earlier versions already lawfully obtained.

## Decisions still intentionally open

Discuss before implementation where relevant:

- exact Service schema and lifecycle;
- exact MVP cut line;
- payment provider;
- email provider;
- file-storage limits and allowed file types;
- invoice numbering format;
- overdue behavior;
- taxes/discounts/credits;
- whether client messages should support email reply ingestion;
- deployment host;
- retention rules;
- support-ticket workflow;
- customer self-registration requirements;
- bookkeeping/accounting direction.
