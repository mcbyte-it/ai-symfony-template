# Symfony Project Template for AI Agents

A collection of structured prompt documents that guide an AI agent (such as Claude Code) through initializing a complete, production-ready Symfony 7.4 project from scratch.

## What This Is

This template provides a step-by-step setup system covering:

- Symfony 7.4 backend with Doctrine ORM/DBAL and MariaDB
- User authentication, RBAC, and security hardening
- Frontend scaffolding (AdminLTE, Bootstrap 5, or Tailwind/DaisyUI)
- Internationalisation and RTL support
- Git repository setup with pre-commit quality hooks
- PHPUnit + PHPStan testing configuration
- Gitea Actions CI pipeline with optional SonarQube integration
- Prettier code formatting

The AI agent reads your project description once and uses those answers across all setup steps — you are never asked the same question twice.

---

## How to Use

### Step 1 — Fill in your project description

Open [docs/template/0-PROJECT_DESCRIPTION.md](docs/template/0-PROJECT_DESCRIPTION.md) and replace **all** sections with your own project details:

- **Project Identity** — name, short description, primary domain, deployment type
- **Project Brief** — what the application does and who it is for
- **Core Entities** — the main data models your app manages
- **Key Workflows** — the primary user journeys
- **Functional Requirements** — must-have MVP features and nice-to-haves
- **Technical Requirements** — table prefix, RBAC roles, i18n languages, frontend framework choice, testing and CI flags
- **Pre-Answered Setup Questions** — fill the answer column for every row so sub-prompts do not ask again
- **Folder Structure** — adjust to match your expected `src/` layout

> Do not leave any section showing the sample "InvoiceFlow" invoicing app data. The setup will refuse to continue if sample data is detected.

### Step 2 — Run the setup prompt

Once `0-PROJECT_DESCRIPTION.md` is filled in, tell your AI agent:

> Execute the file `docs/template/1-NEW_PROJECT.md`

The agent will read your project description, confirm the loaded values, and then run all setup sub-prompts in sequence to generate a fully scaffolded project.

---

## Template Files

| File | Purpose |
|---|---|
| `docs/template/0-PROJECT_DESCRIPTION.md` | Your project specification — fill this in first |
| `docs/template/1-NEW_PROJECT.md` | Main entry point — runs all sub-prompts in order |
| `docs/template/2-BACKEND.md` | Symfony backend, Doctrine, entities, auth, security |
| `docs/template/3-GIT.md` | Git repo, .gitignore, pre-commit hooks |
| `docs/template/4-FRONTEND.md` | UI framework, RTL support, base templates |
| `docs/template/5-TESTING.md` | PHPUnit, PHPStan, test folder structure |
| `docs/template/6-CI.md` | Gitea Actions CI pipeline, SonarQube |
| `docs/template/7-PRETTIER.md` | Prettier for Twig, HTML, CSS, and JS |

---

## Output

After the agent completes setup you will have:

- A fully configured Symfony project with all chosen dependencies installed
- `CLAUDE.md` at the project root with coding rules and tech stack context
- `docs/plan-progress.md` — a living checklist of all remaining feature work to implement
- `memory/` files for session continuity across AI conversations
