# HappyPortal

HappyPortal is a source-visible client portal and small-business operations application built by Kyle Givler.

It is intended to serve two purposes:

1. run the client-facing and administrative side of Kyle's own development/hosting work at `portal.kgivler.com`; and
2. demonstrate the kind of custom software Kyle can design, build, deploy, and maintain for clients.

HappyPortal is not meant to become a generic SaaS billing platform for every possible business. It starts as a focused client/service portal and grows only when real workflows justify it.

## Planned capabilities

HappyPortal is planned to cover:

- clients and client contacts;
- internal notes and client-visible communication;
- services, service dates, renewal state, and cancellation requests;
- invoices and invoice line items;
- manual and hosted payment flows;
- client login and self-service views;
- two-way file sharing;
- an administrative dashboard with useful business/client/service statistics;
- support tickets;
- later bookkeeping/accounting and expense tracking;
- later customer self-registration;
- later automation around renewals, provisioning, and service lifecycle.

Not all of those belong in the MVP. See [PORTAL_PLAN.md](PORTAL_PLAN.md) for the staged plan.

## Product direction

The portal is intentionally useful as both an internal business tool and a customer-facing application.

The public-facing development-services site lives separately at `dev.kgivler.com`. HappyPortal is the authenticated application behind `portal.kgivler.com`.

The project should remain boring where boring is good:

- ASP.NET Core / .NET
- server-rendered UI
- SQLite initially
- ASP.NET Core Identity
- Docker deployment
- simple interfaces around payment and file-storage providers
- no distributed-systems cosplay without an actual requirement

## Commercial availability

HappyPortal is also an example of software that can be commissioned from Kyle.

If you want a portal like this, a customized version, a white-glove deployment, integration work, hosting, maintenance, or a commercial license to HappyPortal itself, contact Kyle through the links on [kgivler.com](https://kgivler.com/).

Commercial licenses may be offered separately and may include rights not granted by the public source-visible license.

## License

**HappyPortal is source-visible, not open source.**

The source is published for inspection, security review, architectural transparency, education about the software itself, and evaluation of Kyle's work.

The public repository does **not** grant permission to run, deploy, modify, redistribute, host, or use the software except as expressly permitted by the [LICENSE](LICENSE).

Future versions of HappyPortal may be released under different license terms. A change to the license for a future release does not retroactively change rights already granted for a version you lawfully received under an earlier license.

## Status

Early planning / pre-MVP.

The current design is documented in:

- [PORTAL_PLAN.md](PORTAL_PLAN.md)
- [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)
- [AGENTS.md](AGENTS.md)

The service model and several later-stage workflows are intentionally not fully specified yet and should be discussed before implementation.
