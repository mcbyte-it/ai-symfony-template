# New Project Setup

This prompt bootstraps a new Symfony project from scratch. Execute this file first, then run the numbered sub-files in order.

---

## Step 0: Read and Validate 0-PROJECT_DESCRIPTION.md

**Before doing anything else**, read the file `0-PROJECT_DESCRIPTION.md` in the same folder as this file.

Then check whether it still contains sample/placeholder data by looking for any of these indicators:

- Project name is still `InvoiceFlow`
- Short description mentions "invoicing" or "freelancers"
- The Project Brief section describes "clients, services, invoices, payments"
- The "Pre-Answered Setup Questions" table still references `InvoiceFlow`
- Any section contains the exact phrase "fictional invoicing application"

**If any of the above are true**, stop immediately and display this message to the user:

---

> **Setup cannot continue.**
>
> `0-PROJECT_DESCRIPTION.md` still contains the sample invoicing app data and has not been filled in for your project.
>
> Please open `docs/template/0-PROJECT_DESCRIPTION.md`, replace all sections with your own project details, and then run this prompt again.
>
> Pay special attention to:
>
> - **Project Identity** table (name, description, domain, deployment)
> - **Project Brief** and **Functional Requirements**
> - **Technical Requirements** (RBAC, testing, CI decisions)
> - **Pre-Answered Setup Questions** table (answers used by all sub-prompts)

---

Do not proceed past this point until the file has been updated.

---

## Step 1: Load Project Information from 0-PROJECT_DESCRIPTION.md

Read all answers from `0-PROJECT_DESCRIPTION.md`. Do not ask the user for information that is already defined there. Extract and store the following values for use in all subsequent steps:

| Variable              | Source in 0-PROJECT_DESCRIPTION.md                                      |
| --------------------- | ----------------------------------------------------------------------- |
| `PROJECT_NAME`        | Project Identity table → "Project name"                                 |
| `PROJECT_DESCRIPTION` | Project Identity table → "Short description"                            |
| `PROJECT_PURPOSE`     | Project Identity table → "Primary domain"                               |
| `USE_GIT`             | Project Identity table → "Git version control"                          |
| `DEPLOYMENT_TYPE`     | Project Identity table → "Deployment type"                              |
| `TABLE_PREFIX`        | Technical Requirements → Backend → "Table prefix"                       |
| `USE_RBAC`            | Technical Requirements → Backend → "RBAC"                               |
| `USE_I18N`            | Technical Requirements → Internationalisation → "Multilanguage"         |
| `I18N_LANGUAGES`      | Technical Requirements → Internationalisation → "Languages"             |
| `USE_RTL`             | Technical Requirements → Internationalisation → "RTL support"           |
| `FRONTEND_FRAMEWORK`  | Technical Requirements → Frontend → "Sample selection for this project" |
| `USE_TESTING`         | Technical Requirements → Testing → first line                           |
| `USE_CI`              | Technical Requirements → CI/CD → first line                             |
| `USE_SONARQUBE`       | Technical Requirements → CI/CD → "SonarQube"                            |

If any of the above values are missing or still show sample data for that specific field, stop and tell the user exactly which field in `0-PROJECT_DESCRIPTION.md` needs to be filled in.

Confirm to the user what was loaded before continuing:

> **Project loaded from 0-PROJECT_DESCRIPTION.md:**
>
> - Name: `[PROJECT_NAME]`
> - Description: `[PROJECT_DESCRIPTION]`
> - Git: `[USE_GIT]` | Deployment: `[DEPLOYMENT_TYPE]`
> - Table prefix: `[TABLE_PREFIX]` | RBAC: `[USE_RBAC]`
> - i18n: `[USE_I18N]` | Languages: `[I18N_LANGUAGES]` | RTL: `[USE_RTL]`
> - Frontend framework: `[FRONTEND_FRAMEWORK]`
> - Testing: `[USE_TESTING]` | CI: `[USE_CI]` | SonarQube: `[USE_SONARQUBE]`
>
> Proceeding with setup...

---

## Step 2: Create CLAUDE.md

Create `CLAUDE.md` at the project root with the following content. Use the values loaded from `0-PROJECT_DESCRIPTION.md` in Step 1 — replace all `[PLACEHOLDER]` tokens with the actual project values.

```markdown
# Claude Code Project Rules

## Project

**Name:** [PROJECT_NAME]
**Description:** [PROJECT_DESCRIPTION]
**Purpose:** [PROJECT_PURPOSE]

## Documentation Requirements

- Update relevant documentation in `/docs` when modifying features
- Keep `docs/README.md` in sync with new capabilities
- Maintain changelog entries in `docs/CHANGELOG.md`

## 1. Context & Memory Management

- **End-of-Session Routine:** Before ending a task, update `memory/ActiveContext.md`. Use the auto memory system at `memory/`.
- **File Management:** Do not read entire folders. Use Glob first to list files, then target specific files.
- **Large Files:** If a file is >300 lines, summarize its structure before editing.
- **Prompt History:** Ensure `memory/prompt-history.md` exists; create if missing. After each task, append a brief entry with date and summary of changes made.

## 2. Coding Standards / Tech Stack

- **Tech Stack:**
    - **Symfony 7.4**
    - **PHP 8.4**
    - **Doctrine DBAL 4.4.x** and **Doctrine ORM 3.x**
    - **MariaDB 12.2**
    - **Bootstrap 5** for CSS/layout
    - **Font Awesome 7.x** for iconography
    - **Modern UI design** across all user-facing pages
- **Standard:** PSR-12 for all PHP code
- **Errors:** If a terminal command fails twice, STOP and analyze the logs before trying a third time.

## 3. Communication

- Be concise. Don't explain basic code unless asked.
- In **Plan Mode** (EnterPlanMode), focus on architecture and design decisions — explore the codebase, present options, and wait for approval before writing code.
- In **implementation** (after plan approval or for simple tasks), focus on clean, direct execution.

## 4. Phase Gates — MANDATORY

After completing **each phase** in `plan-progress.md`:

1. Mark all completed checkboxes with `[x]` in `plan-progress.md`
2. Print a short summary: what was done, any files created or modified, any decisions made
3. **STOP. Do not start the next phase.**
4. Wait for the user to explicitly say to continue (e.g. "continue", "next", "proceed")

Never chain phases together automatically. One phase at a time, always.

## 5. Feature Development Workflow

- When a new feature is requested and the project uses Git:
    1. Create a new branch: `git checkout -b feature/<feature-name>`
    2. Develop everything in that branch
    3. Only merge to `main` when the feature is complete and tested

## Frontend Rules

- Always use **Bootstrap 5** for HTML forms, layout, and styling.
- Use **Font Awesome 7.x** icons inside buttons for actions.
- Use consistent Bootstrap classes (`btn btn-primary`, `btn btn-danger`, etc.) with icons:
    - Edit → `<i class="fa-solid fa-pen-to-square"></i>`
    - Delete → `<i class="fa-solid fa-trash"></i>`
    - View → `<i class="fa-solid fa-eye"></i>`
- Ensure forms and buttons are accessible and responsive.

## Testing (Only if the user want to implement testing: [USE_TESTING])

- Create PHPUnit unit test files for every service, command, and Twig extension created.
- Place unit tests under `tests/Unit/` mirroring the `src/` structure.
- Run `php bin/phpunit` to verify tests pass.

## Security

### Sensitive Files — DO NOT read or modify:

- `.env` files
- `*/config/secrets.*`
- `*/*.pem`
- Any file containing API keys, tokens, or credentials

### Security Practices

- Never commit sensitive files
- Use environment variables for secrets
- Keep credentials out of logs and output
- Always use CSRF protection on forms
- Sanitize and validate all user input
- Use parameterized queries — never raw SQL interpolation
```

---

## Step 3: Create Memory and History Files

Create the following files:

### `memory/ActiveContext.md`

```markdown
# Active Context

## Current Task

_None — project just initialized._

## Recent Changes

_None yet._

## Open Questions

_None._
```

### `memory/prompt-history.md`

```markdown
# Prompt History

## [DATE] — Project Initialization

- Created CLAUDE.md, memory files, and initial project structure.
- Ran sub-prompts: 1-NEW_PROJECT, 2-BACKEND, 3-GIT, 4-FRONTEND, 5-TESTING, 6-CI, 7-PRETTIER.
```

---

## Step 4: Create docs/CHANGELOG.md

```markdown
# Changelog

## [Unreleased]

### Added

- Initial project setup
- Symfony 7.4 backend with MariaDB 12.2
- User authentication with secure password hashing
- Git pre-commit hooks (secrets detection, PHP CS Fixer)
```

---

## Step 5: Create docs/README.md

Create `docs/README.md` with the following content. Use `PROJECT_NAME` loaded from `0-PROJECT_DESCRIPTION.md`. Also incorporate the **Folder Structure** section from `0-PROJECT_DESCRIPTION.md` if it has been customized (i.e. it differs from the default invoicing sample).

```markdown
# [PROJECT_NAME] — Developer Guide

## Tech Stack

- **Framework:** Symfony 7.4
- **PHP:** 8.4
- **DB access:** Doctrine DBAL 4.4.x + Doctrine ORM 3.x
- **Database:** MariaDB 12.2
- **Templating:** Twig 3
- **UI:** Bootstrap 5
- **Icons:** Font Awesome 7.x
- **Testing:** PHPUnit + PHPStan
- **Code Style:** PSR-12 (enforced by PHP CS Fixer)

## Project Structure
```

src/
Controller/ # One controller per feature area
Entity/ # Doctrine ORM entities
Repository/ # Query logic (ORM + DBAL for complex queries)
Form/ # Symfony Form types
Service/ # Business logic services
Command/ # Symfony console commands
Twig/ # Custom Twig extensions
templates/ # Twig templates, organized by feature
migrations/ # Doctrine migrations
config/ # Symfony configuration
public/ # Web root
tests/Unit/ # PHPUnit unit tests (mirrors src/)
docs/ # Project documentation
memory/ # Claude Code session memory

```

## Development Setup

1. Copy `.env` to `.env.local` and fill in `DATABASE_URL` and `APP_SECRET`
2. Run `composer install`
3. Run `php bin/console doctrine:database:create`
4. Run `php bin/console doctrine:migrations:migrate`
5. Run `php bin/console app:create-user` to create the first user
6. Start the development server: `symfony server:start` or configure Apache

## Coding Standards

- PHP 8.4, PSR-12
- All PHP code is auto-formatted by PHP CS Fixer on commit (via pre-commit hook)
- Run manually: `vendor/bin/php-cs-fixer fix --config=.php-cs-fixer.dist.php`

## Testing

- Run all unit tests: `php bin/phpunit tests/Unit --no-coverage`
- Run static analysis: `vendor/bin/phpstan analyse`
- Lint Twig templates: `php bin/console lint:twig templates/`

## Git Workflow

- Feature branches: `git checkout -b feature/<name>`
- Pre-commit hooks run automatically on every commit (see 3-GIT.md)
- Merge to `main` only when feature is complete and tests pass
```

---

## Step 6: Execute Sub-Prompts in Order

After completing Steps 1–5, execute the following sub-prompts in sequence. For each sub-prompt, pass the pre-loaded answers from `0-PROJECT_DESCRIPTION.md` — sub-prompts must **not** re-ask questions that are already answered there.

1. **[2-BACKEND.md](2-BACKEND.md)** — Use `TABLE_PREFIX`, `USE_RBAC`, `USE_I18N`, `I18N_LANGUAGES`, and `USE_RTL` from loaded data. Do not ask again.
2. **[3-GIT.md](3-GIT.md)** — Configure Git, .gitignore, and pre-commit hooks.
3. **[4-FRONTEND.md](4-FRONTEND.md)** — Use `FRONTEND_FRAMEWORK`, `USE_I18N`, and `USE_RTL` from loaded data. Set up the chosen UI framework and RTL support automatically if required. Do not ask again.
4. **[5-TESTING.md](5-TESTING.md)** — Use `USE_TESTING` from loaded data. If `yes`, proceed automatically without asking.
5. **[6-CI.md](6-CI.md)** — Use `USE_CI` and `USE_SONARQUBE` from loaded data. If `yes`, proceed automatically without asking.
6. **[7-PRETTIER.md](7-PRETTIER.md)** — Install and configure Prettier.

After each sub-prompt completes, update `memory/prompt-history.md` with a brief entry.

---

## Step 7: Generate plan-progress.md

Create (or overwrite) the file `plan-progress.md` inside `docs/`. This is the living progress tracker for the actual project being built.

### Rules for generating this file

- **Infrastructure-first ordering is mandatory.** All setup phases (skeleton, config, git, frontend base, testing setup, CI, prettier) must appear and be completed before any feature/code-generation phases are listed.
- Only include phases that apply — omit phases for features the user did not choose (e.g. if `USE_TESTING = No`, omit the Testing phase entirely; if `USE_I18N = No`, omit the i18n phase)
- Every step must have a `[ ]` checkbox
- Steps already completed during this setup session must be marked `[x]`
- Each group must have a clear heading and a short one-line description of what the phase delivers
- **Feature phases (entities, repositories, controllers, templates) must NOT appear until after all infrastructure phases.** Add a clear separator comment in the file to mark where feature planning begins.
- Do not generate feature phase content during the setup session — leave a placeholder block and instruct the user to add feature phases after the infrastructure is verified and committed.

### Required phase structure

Use this as the skeleton and adapt content to the project loaded from `0-PROJECT_DESCRIPTION.md`:

```markdown
# [PROJECT_NAME] — Plan Progress

_Generated by 1-NEW_PROJECT.md on [DATE]. Update checkboxes as features are implemented._

---

## Phase 1: Project Initialization

_Goal: working repo with CLAUDE.md, memory files, and docs._

- [x] Create CLAUDE.md with project metadata, tech stack, and coding rules
- [x] Create memory/ActiveContext.md
- [x] Create memory/prompt-history.md
- [x] Create docs/CHANGELOG.md
- [x] Create docs/README.md with tech stack and project structure

---

## Phase 2: Backend Skeleton — Symfony + Database

_Goal: running Symfony app connected to MariaDB with prefixed tables, authentication, and base security._

- [x] Initialize Symfony 7.4 skeleton project
- [x] Install doctrine ORM, DBAL, migrations, security, rate-limiter, twig, apache-pack
- [x] Configure .env with DATABASE_URL and TABLE_PREFIX=[TABLE_PREFIX]
- [x] Create PrefixedNamingStrategy and configure doctrine.yaml
- [x] Create User entity (username, email, password, realName, roles, isActive, failedAttempts, lockedUntil, mustChangePw, createdAt, updatedAt)
- [x] Create RememberMeToken entity
- [x] Generate and run initial migration — verify TABLE_PREFIX applied
- [x] Configure security.yaml (argon2id, form login + CSRF, remember-me 7 days, access control)
- [x] Configure rate limiter (5 attempts / 15 minutes on login)
- [x] Create SecurityController (login + logout)
- [x] Create login Twig template (Bootstrap 5, CSRF, remember-me checkbox)
- [x] Create app:create-user command
- [ ] Run security checklist (CSRF, argon2id, rate limiter, parameterized queries, .env in .gitignore)

---

## Phase 3: Version Control

_Goal: git repo with quality gates enforced on every commit._

- [x] Initialize git repository and set default branch to main
- [x] Create .gitignore (Symfony + IDE + OS)
- [x] Install PHP CS Fixer and create .php-cs-fixer.dist.php
- [x] Create .pre-commit-config.yaml (secrets detection, gitleaks, php-cs-fixer, php-lint)
- [x] Create .gitleaks.toml
- [x] Run detect-secrets scan and commit .secrets.baseline
- [x] Install pre-commit hooks and run --all-files to verify
- [x] Make initial commit

---

## Phase 4: Frontend Base

_Goal: base layout and UI framework in place; all pages inherit from a single base template._

- [x] Install and configure [FRONTEND_FRAMEWORK]
- [x] Create base.html.twig with layout, navbar, sidebar, and flash messages
- [x] Create dashboard placeholder route and template
- [ ] Verify login page renders correctly with chosen UI framework
- [ ] Verify layout is responsive on mobile

---

## Phase 5: Code Quality — Prettier

_Goal: consistent formatting enforced for JS/CSS/Twig assets._

- [x] Install Prettier and create .prettierrc
- [x] Configure .prettierignore
- [ ] Run Prettier on all frontend assets and verify output

---

<!-- ============================================================ -->
<!-- INFRASTRUCTURE COMPLETE — verify and commit before proceeding -->
<!-- Do not start feature phases until all phases above are [x]   -->
<!-- ============================================================ -->

---

# --- FEATURE PHASES — Add one block per Functional Requirement ---

# For each major feature area from 0-PROJECT_DESCRIPTION.md,
# copy the Phase N template below and fill in the details.
# Do NOT start implementing these until the infrastructure phases above are done and committed.

---

## Phase N: [Feature Name from Functional Requirements]

_Goal: [one sentence describing what this feature delivers to the user]_

- [ ] Create [FeatureName] entity with fields: [list key fields]
- [ ] Generate and run migration
- [ ] Create [FeatureName]Repository (ORM for CRUD, DBAL for complex queries if needed)
- [ ] Create [FeatureName]Type form
- [ ] Create [FeatureName]Service (business logic)
- [ ] Create [FeatureName]Controller (index, show, new, edit, delete routes)
- [ ] Create Twig templates (index, show, \_form partial)
- [ ] Write unit tests in tests/Unit/Service/[FeatureName]ServiceTest.php
- [ ] Write unit tests in tests/Unit/Repository/[FeatureName]RepositoryTest.php (if applicable)
```

### Additional conditional phases — include only if the flag is set

**Include if `USE_I18N = Yes`:**

```markdown
## Phase X: Internationalisation

_Goal: all UI strings translatable; locale switchable via URL prefix._

- [x] Install symfony/translation
- [x] Configure translation.yaml with default locale and fallback
- [x] Configure URL prefix routing in routes.yaml
- [x] Create LocaleController with app_switch_locale route
- [x] Create starter translations/messages.en.yaml
- [x] Create starter translations/messages.[LOCALE].yaml for each non-default language
- [ ] Replace all hard-coded UI strings with {{ 'key'|trans }} in templates
- [ ] Verify translation keys exist for all languages before each release
```

**Include if `USE_RTL = Yes`:**

```markdown
## Phase X: RTL Support

_Goal: layout renders correctly in right-to-left languages._

- [x] Set lang and dir attributes dynamically on <html> element
- [x] Serve RTL stylesheet (Bootstrap RTL / AdminLTE RTL) when locale is RTL
- [x] Add locale switcher to navbar
- [ ] Audit all templates for physical CSS properties (margin-left etc.) — replace with logical equivalents
- [ ] Test Arabic layout on all major pages: tables, forms, buttons, modals
- [ ] Add Arabic font (Noto Sans Arabic) with correct line-height
```

**Include if `USE_TESTING = Yes`:**

```markdown
## Phase X: Testing and Static Analysis

_Goal: full test suite passing; PHPStan at level 5 with 0 errors._

- [x] Install PHPUnit 11.x and PHPStan 2.x with Symfony + Doctrine extensions
- [x] Create phpunit.xml.dist
- [x] Create phpstan.neon (level 5)
- [x] Create tests/object-manager.php for PHPStan Doctrine integration
- [x] Create tests/Unit/ folder structure
- [ ] Achieve >80% coverage on all Service classes
- [ ] PHPStan level 5 — 0 errors
- [ ] All Twig templates pass lint:twig
- [ ] DI container passes lint:container
```

**Include if `USE_CI = Yes`:**

```markdown
## Phase X: CI Pipeline

_Goal: every push to main triggers automated quality checks._

- [x] Create .gitea/workflows/ci.yml
- [x] Configure Gitea repository secrets (SONAR_TOKEN, SONAR_HOST_URL)
- [ ] Push to repository and verify pipeline is green end-to-end
- [ ] Confirm coverage report uploaded to SonarQube (if enabled)
```

### Placement of conditional phases

Insert conditional phases (i18n, RTL, Testing, CI) **between Phase 5 and the infrastructure-complete separator**, in this order: i18n → RTL → Testing → CI. Do not place them after the feature placeholder.

### After generating the file

- Mark all steps that were already completed during this session as `[x]`
- Leave all future feature steps as `[ ]`
- **Do not generate or fill in any feature Phase N blocks.** Leave the placeholder template as-is.
- Tell the user:

> **Setup infrastructure is complete.**
>
> Your `plan-progress.md` has been created. The infrastructure phases (skeleton, git, frontend base, quality tools) are listed and tracked.
>
> **Next step:** Open `plan-progress.md` and add one Phase block per feature listed in your Functional Requirements. When you are ready to start the first feature phase, tell me which phase to begin and I will implement it — one phase at a time, stopping for your review after each one.
