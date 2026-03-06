# Git Setup — .gitignore, Pre-commit Hooks, and PHP CS Fixer

This prompt configures version control, pre-commit quality gates, and code style enforcement.

**Prerequisites:** Complete [2-BACKEND.md](2-BACKEND.md) first. Requires `git` and `pre-commit` installed on the system.

---

## Step 1: Initialize Git Repository

```bash
git init
git branch -M main
```

---

## Step 2: Create .gitignore

Create `.gitignore` at the project root:

```gitignore
###> symfony/framework-bundle ###
/.env.local
/.env.local.php
/.env.*.local
/config/secrets/prod/prod.decrypt.private.php
/public/bundles/
/var/
/vendor/
###< symfony/framework-bundle ###

###> phpunit/phpunit ###
/phpunit.xml
###< phpunit/phpunit ###

# PHP CS Fixer cache
.php-cs-fixer.cache

# IDE
.idea/
.vscode/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Composer
composer.phar

# Node (if used)
node_modules/
npm-debug.log

# Secrets baseline (can be committed if desired — review first)
# .secrets.baseline
```

---

## Step 3: Create PHP CS Fixer Config

Create `.php-cs-fixer.dist.php` at the project root:

```php
<?php

$finder = PhpCsFixer\Finder::create()
    ->in(__DIR__.'/src')
    ->exclude('var');

return (new PhpCsFixer\Config())
    ->setRules([
        '@Symfony' => true,
        '@PSR12' => true,
        'array_syntax' => ['syntax' => 'short'],
        'ordered_imports' => ['sort_algorithm' => 'alpha'],
        'no_unused_imports' => true,
        'trailing_comma_in_multiline' => true,
        'phpdoc_order' => true,
    ])
    ->setFinder($finder)
    ->setCacheFile(__DIR__.'/.php-cs-fixer.cache');
```

Install PHP CS Fixer as a dev dependency:

```bash
composer require --dev friendsofphp/php-cs-fixer
```

Test it runs correctly:

```bash
vendor/bin/php-cs-fixer fix --config=.php-cs-fixer.dist.php --dry-run --diff
```

---

## Step 4: Create Pre-commit Configuration

Create `.pre-commit-config.yaml` at the project root:

```yaml
repos:
    - repo: https://github.com/pre-commit/pre-commit-hooks
      rev: v4.6.0
      hooks:
          - id: trailing-whitespace
          - id: end-of-file-fixer
          - id: check-yaml
          - id: check-added-large-files
            args: [--maxkb=500]

    - repo: https://github.com/Yelp/detect-secrets
      rev: v1.5.0
      hooks:
          - id: detect-secrets
            args: [--baseline, .secrets.baseline]

    - repo: https://github.com/gitleaks/gitleaks
      rev: v8.18.4
      hooks:
          - id: gitleaks
            args: [--config, .gitleaks.toml]

    - repo: local
      hooks:
          - id: php-cs-fixer
            name: PHP CS Fixer
            entry: vendor/bin/php-cs-fixer fix --config=.php-cs-fixer.dist.php --diff --dry-run
            language: system
            files: \.php$
            pass_filenames: false

          - id: php-lint
            name: PHP Syntax Check
            entry: bash -c 'for f in "$@"; do php -l "$f" || exit 1; done'
            language: system
            files: \.php$
```

---

## Step 5: Create Gitleaks Config

Create `.gitleaks.toml` at the project root:

```toml
[extend]
useDefault = true

[allowlist]
description = "Project-specific allowlist"
regexes = []
paths = [
    ".secrets.baseline",
    "docs/",
    "tests/",
]
```

---

## Step 6: Initialize detect-secrets Baseline

Run once to create the baseline file (scan current repo for any existing secrets patterns):

```bash
detect-secrets scan > .secrets.baseline
```

Review `.secrets.baseline` and commit it:

```bash
git add .secrets.baseline
git commit -m "chore: add detect-secrets baseline"
```

---

## Step 7: Install Pre-commit Hooks

```bash
pip install pre-commit   # or: brew install pre-commit
pre-commit install
pre-commit run --all-files   # dry run to verify everything passes
```

---

## Step 8: Initial Commit

```bash
git add .
git commit -m "initial project setup"
```

---

## Git Workflow Summary for Developers

Add this section to `docs/README.md` under a "Git Workflow" heading:

````markdown
## Git Workflow

### Branching

- All new features go in a dedicated branch:
    ```bash
    git checkout -b feature/<feature-name>
    ```
````

- Bug fixes: `fix/<short-description>`
- Hotfixes: `hotfix/<short-description>`
- Merge to `main` only when the feature is complete, tested, and reviewed.

### Pre-commit Hooks

Every commit automatically runs:

| Hook                    | What it checks                         |
| ----------------------- | -------------------------------------- |
| trailing-whitespace     | Removes trailing whitespace            |
| end-of-file-fixer       | Ensures files end with a newline       |
| check-yaml              | Validates YAML syntax                  |
| check-added-large-files | Blocks files > 500 KB                  |
| detect-secrets          | Scans for API keys, tokens, passwords  |
| gitleaks                | Deep secrets scanning (OWASP patterns) |
| php-cs-fixer            | Enforces PSR-12 + Symfony code style   |
| php-lint                | PHP syntax check on changed files      |

If a hook fails, the commit is blocked. Fix the issue and try again.

### Useful Commands

```bash
# Run all pre-commit hooks manually
pre-commit run --all-files

# Fix PHP code style
vendor/bin/php-cs-fixer fix --config=.php-cs-fixer.dist.php

# Skip hooks in emergency (use sparingly)
git commit --no-verify -m "message"
```
