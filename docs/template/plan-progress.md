# Project Setup Plan — Progress Tracker

This file tracks the progress of bootstrapping a new Symfony project using the template files in this folder.

**Instructions for Claude:** Update this file as each step is completed. Change `[ ]` to `[x]` when a step is done. Do not skip steps — complete them in order unless noted otherwise.

---

## Phase 1: Project Initialization

- [ ] Ask user for project name, description, purpose, Git preference, and deployment path
- [ ] Create `CLAUDE.md` at project root with project metadata, tech stack, coding standards, and feature branching rules
- [ ] Create `memory/ActiveContext.md`
- [ ] Create `memory/prompt-history.md` and add initial entry
- [ ] Create `docs/CHANGELOG.md` with initial unreleased section
- [ ] Create `docs/README.md` with project overview, tech stack, structure, and workflow guide

---

## Phase 2: Backend — Symfony + Database

- [ ] Initialize Symfony 7.4 project (`composer create-project symfony/skeleton`)
- [ ] Install required Symfony packages (twig, security, rate-limiter, apache-pack, maker-bundle, doctrine ORM + DBAL + migrations)
- [ ] Configure `.env` with `DATABASE_URL` and `TABLE_PREFIX`
- [ ] Create `PrefixedNamingStrategy` class and register in `services.yaml`
- [ ] Configure `doctrine.yaml` with custom naming strategy
- [ ] Create `User` entity with all required fields (username, email, password hash, real name, roles, is_active, must_change_pw, failed_attempts, locked_until, created_at, updated_at)
- [ ] Create `RememberMeToken` entity
- [ ] Ask user about RBAC / RLS preference
- [ ] If RBAC: create `Role` entity and user-role join table
- [ ] Generate and run Doctrine migrations (verify TABLE_PREFIX applied)
- [ ] Configure `security.yaml` (argon2id hashing, form login with CSRF, remember-me 7 days, route access control)
- [ ] Configure rate limiter (`login_limiter`: 5 attempts / 15 minutes)
- [ ] Create `SecurityController` (login + logout routes)
- [ ] Create login Twig template with Bootstrap 5 + CSRF token + remember-me checkbox
- [ ] Create `CreateUserCommand` (`app:create-user`) with interactive prompts and option arguments
- [ ] Run security checklist (CSRF, argon2id, rate limiter, parameterized queries, TABLE_PREFIX, .env in .gitignore)

---

## Phase 3: Git Configuration

- [ ] Initialize Git repository (`git init`, set branch to `main`)
- [ ] Create `.gitignore` (Symfony defaults + IDE + OS + cache files)
- [ ] Install PHP CS Fixer (`composer require --dev friendsofphp/php-cs-fixer`)
- [ ] Create `.php-cs-fixer.dist.php` with PSR-12 + Symfony ruleset
- [ ] Create `.pre-commit-config.yaml` with hooks: trailing-whitespace, end-of-file-fixer, check-yaml, check-added-large-files, detect-secrets, gitleaks, php-cs-fixer, php-lint
- [ ] Create `.gitleaks.toml`
- [ ] Run `detect-secrets scan > .secrets.baseline` and commit baseline
- [ ] Install pre-commit (`pip install pre-commit && pre-commit install`)
- [ ] Run `pre-commit run --all-files` and confirm all hooks pass
- [ ] Make initial commit (`git add . && git commit -m "chore: initial project setup"`)
- [ ] Update `docs/README.md` with Git workflow section (branching, hooks table, useful commands)

---

## Phase 4: Frontend Baseline

- [ ] Create `templates/base.html.twig` with Bootstrap 5 CDN, Font Awesome 7, nav, flash messages, and block structure
- [ ] Create `public/css/app.css` with minimal custom overrides
- [ ] Decide: CDN vs self-hosted for Bootstrap and Font Awesome
- [ ] Implement dashboard route and template (`app_dashboard`)
- [ ] Create `templates/macros.html.twig` with project-specific macros (if needed)
- [ ] **[USER ACTION REQUIRED]** Customize navigation structure for the project
- [ ] **[USER ACTION REQUIRED]** Define color scheme and visual design

---

## Phase 5: Testing (Optional — user must confirm)

- [ ] Ask user if testing should be set up
- [ ] Install PHPUnit 11.x and PHPStan 2.x with Symfony + Doctrine extensions
- [ ] Create `phpunit.xml.dist`
- [ ] Create `phpstan.neon` (level 5)
- [ ] Create `tests/object-manager.php` for PHPStan Doctrine integration
- [ ] Create folder structure: `tests/Unit/{Service,Controller,Twig,Command,Repository}/`
- [ ] Create `tests/Unit/ExampleTest.php` to verify test runner works
- [ ] Run `php bin/phpunit tests/Unit --no-coverage` — confirm passes
- [ ] Run `vendor/bin/phpstan analyse` — confirm 0 errors
- [ ] Update `CLAUDE.md` with testing requirements (test files for every service/command/extension)

---

## Phase 6: CI Workflow

- [ ] Create `.gitea/workflows/` directory
- [ ] Create `.gitea/workflows/ci.yml` with: checkout, PHP 8.4 setup, Composer cache, DB migrations (if needed), Twig lint, container lint, PHPStan, PHPUnit with coverage, SonarQube scan (optional)
- [ ] If SonarQube: create `sonar-project.properties`
- [ ] Configure Gitea repository secrets: `SONAR_TOKEN`, `SONAR_HOST_URL`
- [ ] Push to repository and verify CI pipeline passes

---

## Phase 7: Prettier

- [ ] Initialize npm (`npm init -y`)
- [ ] Install Prettier (`npm install --save-dev prettier`)
- [ ] Create `.prettierrc` (printWidth 120, tabWidth 4, no tabs, semis, double quotes, Twig override)
- [ ] Create `.prettierignore` (exclude vendor, node_modules, var, public/build, migrations, *.min.*)
- [ ] Create `.vscode/settings.json` (Prettier as default formatter, format on save; PHP CS Fixer for PHP files)
- [ ] Create `.vscode/extensions.json` with recommended extensions
- [ ] Add `format` and `format:check` scripts to `package.json`
- [ ] Run `npm run format:check` — confirm no errors
- [ ] Run `npm run format` if formatting fixes needed

---

## Phase 8: Final Verification

- [ ] Run full test suite: `php bin/phpunit tests/Unit --no-coverage`
- [ ] Run static analysis: `vendor/bin/phpstan analyse`
- [ ] Run Twig lint: `php bin/console lint:twig templates/`
- [ ] Run container lint: `php bin/console lint:container`
- [ ] Run Prettier check: `npm run format:check`
- [ ] Run pre-commit hooks: `pre-commit run --all-files`
- [ ] Test user creation: `php bin/console app:create-user`
- [ ] Test login at `/login` with created user
- [ ] Push to repository and confirm CI pipeline is green
- [ ] Update `memory/prompt-history.md` with final summary
- [ ] Update `memory/ActiveContext.md` to reflect project is initialized

---

## Notes

_Add any project-specific notes, decisions, or deviations from the template here._
