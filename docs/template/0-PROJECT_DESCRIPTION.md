# Project Description

> **IMPORTANT: MODIFY THIS FILE BEFORE RUNNING ANY OTHER TEMPLATE PROMPTS.**
>
> This file contains sample data for a fictional invoicing application. Replace every section below with your own project details. The answers you provide here will be used automatically by `1-NEW_PROJECT.md` and all sub-prompts — you will not be asked the same questions twice.
>
> When you are done editing, run `1-NEW_PROJECT.md` to start the project setup.

---

## Project Identity

| Field | Value |
|---|---|
| **Project name** | InvoiceFlow |
| **Short description** | A self-hosted invoicing and billing management app for freelancers and small agencies |
| **Primary domain** | Business operations — client management, service catalog, invoice lifecycle, payment tracking |
| **Deployment type** | Sub-folder under Apache (e.g. `/invoiceflow`) |
| **Git version control** | Yes |

---

## Project Brief

InvoiceFlow is a multi-user web application for managing the full invoicing lifecycle. The system allows users to manage clients, define billable services, generate invoices, and track payment status — all from a single self-hosted dashboard.

The target user is a freelancer or small agency owner who needs a lightweight alternative to SaaS invoicing tools, with full control over their data.

### Core Entities

- **Clients** — companies or individuals who receive invoices. Each client has a name, contact details, billing address, VAT number, and currency preference.
- **Services / Line Items** — a catalog of billable services with a default unit price and unit label (hours, days, items, etc.).
- **Invoices** — issued to a client, containing one or more line items. Each invoice has a status lifecycle: `draft` → `sent` → `paid` / `overdue` / `cancelled`.
- **Payments** — partial or full payment records linked to an invoice.
- **Users** — staff members who create and manage invoices. Multiple users share the same account base.

### Key Workflows

1. Create a client → define their services → generate an invoice → send it → mark as paid.
2. View overdue invoices and send reminders.
3. Generate monthly/quarterly revenue reports by client or service.
4. Export invoice as PDF.

---

## Functional Requirements

### Must Have (MVP)

- [ ] Multi-user authentication with role-based access (admin, billing staff)
- [ ] Client management (CRUD with contact details, billing address, currency)
- [ ] Service catalog (CRUD — name, unit price, unit label, tax rate)
- [ ] Invoice creation with dynamic line items (quantity × unit price + tax)
- [ ] Invoice status lifecycle: draft, sent, paid, overdue, cancelled
- [ ] Payment recording (partial and full payments)
- [ ] Invoice PDF export
- [ ] Dashboard with key metrics (outstanding total, overdue count, recent payments)
- [ ] Reporting: revenue by client, revenue by month, overdue aging report
- [ ] Audit log: who created/modified each invoice and when

### Nice to Have (Post-MVP)

- [ ] Client portal (read-only view for clients to see their invoices)
- [ ] Recurring invoices (auto-generate monthly invoices for retainer clients)
- [ ] Email sending (send invoice PDF via SMTP)
- [ ] Multi-currency support with exchange rate snapshots
- [ ] Custom invoice number format (prefix + sequence)
- [ ] Tax configuration per country/region

### Out of Scope

- Payment gateway integration (Stripe, PayPal) — invoices are tracked manually
- Time tracking / timesheet integration
- Inventory management

---

## Technical Requirements

### Backend

- Symfony 7.4 on PHP 8.4
- Doctrine ORM 3.x for entity CRUD; Doctrine DBAL 4.4 for reporting queries
- MariaDB 12.2
- Table prefix: `inv_`
- PDF export: `dompdf/dompdf` or `knplabs/knp-snappy` (to be decided)
- RBAC: **Yes** — two roles: `ROLE_ADMIN` and `ROLE_BILLING`

### Frontend

> **Choose one of the following UI frameworks and delete the others. The sample uses AdminLTE v4.**

**Option A — AdminLTE v4.0rc (recommended for admin/dashboard apps):**
- AdminLTE 4.0rc (built on Bootstrap 5) via npm or CDN
- Font Awesome 7 (CDN or self-hosted)
- Chart.js for dashboard charts
- No frontend build tool needed — vanilla JS only

**Option B — Bootstrap 5 (minimal, general-purpose):**
- Bootstrap 5.x (CDN or npm)
- Font Awesome 7 (CDN or self-hosted)
- Vanilla JS only

**Option C — Tailwind CSS + DaisyUI (utility-first, component library):**
- Tailwind CSS 4.x via npm + Vite
- DaisyUI 5.x plugin for pre-built components
- Font Awesome 7 (CDN or self-hosted)
- Requires a frontend build step (Vite or Webpack Encore)

---
**Sample selection for this project: AdminLTE v4.0rc**

- AdminLTE 4.0rc (CDN)
- Font Awesome 7 (CDN)
- Chart.js for dashboard revenue charts
- No frontend build tool needed — vanilla JS only
- Professional admin dashboard aesthetic (sidebar navigation, dark header, card-based layout)

### Internationalisation

- **Multilanguage required:** Yes
- **Languages:** English (default), Arabic
- **RTL support required:** Yes — Arabic is a right-to-left language
- **Symfony translation component:** Yes (`symfony/translation`)
- **Translation file format:** YAML (`.yaml`) stored in `translations/`
- **Locale switching:** URL prefix strategy (e.g. `/en/dashboard`, `/ar/dashboard`)
- **RTL CSS approach:** Bootstrap 5 RTL build (`bootstrap.rtl.min.css`) served when `locale = ar`; AdminLTE also ships an RTL stylesheet

### Testing

- **Yes** — testing is required
- PHPUnit 11.x for unit tests covering all services, repositories, and Twig extensions
- PHPStan level 5 for static analysis
- Twig template linting on every CI run

### CI/CD

- **Yes** — Gitea Actions CI required
- Pipeline: checkout → PHP 8.4 setup → install deps → DB migrations → Twig lint → container lint → PHPStan → PHPUnit with coverage → SonarQube scan
- SonarQube: **Yes** — server URL and token to be added as Gitea secrets
- No MariaDB service needed for CI (unit tests only — no integration tests)

### Code Quality

- PHP CS Fixer with PSR-12 + Symfony ruleset (pre-commit)
- Prettier for Twig/HTML/CSS/JS formatting
- detect-secrets + gitleaks in pre-commit hooks

---

## Architecture Notes

- Follow the DBAL/ORM split pattern: ORM for CRUD, DBAL for complex reporting queries (JOINs, aggregations)
- Invoice number generation must be atomic (use DB sequence or locking to avoid duplicates)
- PDF export should be triggered by a Symfony controller action and streamed as a download
- All money values stored as integers (cents) to avoid floating point rounding errors
- Currency formatting done in Twig with a custom filter
- Audit log table records `user_id`, `entity_type`, `entity_id`, `action`, `created_at` — append-only, no updates

---

## Pre-Answered Setup Questions

The questions asked by `1-NEW_PROJECT.md` and sub-prompts are answered here. Claude should read this section and use these answers directly — **do not ask the user again**.

| Prompt file | Question | Answer |
|---|---|---|
| 1-NEW_PROJECT.md | Project name | `InvoiceFlow` |
| 1-NEW_PROJECT.md | Short description | Self-hosted invoicing and billing management for freelancers and small agencies |
| 1-NEW_PROJECT.md | Primary purpose / domain | Business operations — client/invoice/payment management |
| 1-NEW_PROJECT.md | Git version control? | **Yes** |
| 1-NEW_PROJECT.md | Sub-folder or root deployment? | Sub-folder: `/invoiceflow` |
| 2-BACKEND.md | TABLE_PREFIX | `inv_` |
| 2-BACKEND.md | RBAC or RLS? | **RBAC** — roles: `ROLE_ADMIN`, `ROLE_BILLING` |
| 2-BACKEND.md | Multilanguage / i18n needed? | **Yes** — English (default) + Arabic |
| 2-BACKEND.md | RTL support needed? | **Yes** — Arabic is RTL |
| 2-BACKEND.md | Locale switching strategy? | URL prefix — e.g. `/en/`, `/ar/` |
| 4-FRONTEND.md | UI framework choice? | **AdminLTE v4.0rc** (Bootstrap 5 base) |
| 4-FRONTEND.md | RTL CSS approach? | Bootstrap RTL build + AdminLTE RTL stylesheet when locale = `ar` |
| 5-TESTING.md | Set up testing? | **Yes** — PHPUnit 11 + PHPStan level 5 |
| 6-CI.md | CI workflow needed? | **Yes** — Gitea Actions |
| 6-CI.md | SonarQube integration? | **Yes** — add `SONAR_TOKEN` and `SONAR_HOST_URL` to Gitea secrets |
| 6-CI.md | MariaDB service in CI? | **No** — unit tests only, no DB integration tests |

---

## Folder Structure (Expected)

```
src/
  Controller/
    DashboardController.php
    ClientController.php
    ServiceController.php
    InvoiceController.php
    PaymentController.php
    ReportController.php
    SecurityController.php
  Entity/
    User.php
    Role.php
    Client.php
    Service.php
    Invoice.php
    InvoiceLineItem.php
    Payment.php
    AuditLog.php
  Repository/
    ClientRepository.php
    InvoiceRepository.php       # DBAL for reporting queries
    PaymentRepository.php
  Form/
    ClientType.php
    ServiceType.php
    InvoiceType.php
    PaymentType.php
  Service/
    InvoiceNumberGenerator.php
    InvoicePdfExporter.php
    RevenueReportService.php
    AuditLogService.php
  Twig/
    AppExtension.php            # money filter, status badge, etc.
templates/
  base.html.twig
  dashboard/
    index.html.twig
  client/
  invoice/
  payment/
  report/
  security/
    login.html.twig
tests/
  Unit/
    Service/
    Twig/
    Repository/
```

---

## Notes and Decisions Log

_Add any decisions made during development here. This section is for the team, not for Claude._

- [DATE] Decision: store money as integer cents, format in Twig. Rationale: avoid float rounding.
- [DATE] Decision: PDF export via dompdf (pure PHP, no system dependency). Rationale: simpler deployment.
