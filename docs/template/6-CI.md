# CI Workflows — Gitea Actions

This prompt generates a Gitea Actions CI workflow for the project.

**Prerequisites:** Complete [5-TESTING.md](5-TESTING.md) first.

---

## Step 1: Create Workflow Directory

```bash
mkdir -p .gitea/workflows
```

---

## Step 2: Create the CI Workflow

Create `.gitea/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  qa:
    name: PHP QA
    runs-on: ubuntu-latest

    services:
      mariadb:
        image: mariadb:12.2
        env:
          MARIADB_ROOT_PASSWORD: root
          MARIADB_DATABASE: app_test
          MARIADB_USER: app
          MARIADB_PASSWORD: app
        ports:
          - 3306:3306
        options: >-
          --health-cmd="healthcheck.sh --connect --innodb_initialized"
          --health-interval=10s
          --health-timeout=5s
          --health-retries=5

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.4'
          extensions: intl, mbstring, pdo_mysql, xml, curl, zip
          coverage: xdebug

      - name: Cache Composer dependencies
        uses: actions/cache@v4
        with:
          path: vendor
          key: composer-${{ hashFiles('composer.lock') }}
          restore-keys: composer-

      - name: Install dependencies
        run: composer install --no-interaction --prefer-dist --optimize-autoloader

      - name: Copy .env for CI
        run: |
          cp .env .env.test.local
          echo "DATABASE_URL=mysql://app:app@127.0.0.1:3306/app_test?serverVersion=mariadb-12.2.0&charset=utf8mb4" >> .env.test.local
          echo "TABLE_PREFIX=app_" >> .env.test.local

      - name: Run database migrations
        run: php bin/console doctrine:migrations:migrate --no-interaction --env=test

      - name: Lint Twig templates
        run: php bin/console lint:twig templates/ --env=dev

      - name: Lint DI container
        run: php bin/console lint:container --env=dev

      - name: PHPStan static analysis
        run: |
          php bin/console cache:warmup --env=dev
          vendor/bin/phpstan analyse --memory-limit=256M --no-progress

      - name: Unit tests with coverage
        run: php bin/phpunit tests/Unit --coverage-text --colors=never

      - name: SonarQube scan
        uses: SonarSource/sonarqube-scan-action@master
        if: ${{ vars.SONAR_HOST_URL != '' }}
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: ${{ vars.SONAR_HOST_URL }}
```

---

## Step 3: Create SonarQube Config (Optional)

If the project uses SonarQube, create `sonar-project.properties` at the project root:

```properties
sonar.projectKey=your_project_key
sonar.projectName=Your Project Name
sonar.projectVersion=1.0

sonar.sources=src
sonar.tests=tests
sonar.language=php
sonar.php.coverage.reportPaths=var/coverage/clover.xml
sonar.php.tests.reportPath=var/test-results.xml
sonar.exclusions=vendor/**,var/**,public/build/**
```

For coverage reporting update the PHPUnit command to generate Clover XML:

```bash
php bin/phpunit tests/Unit --coverage-clover var/coverage/clover.xml
```

---

## Step 4: Configure Gitea Repository Secrets

In the Gitea repository settings, add:

| Secret / Variable | Description |
|---|---|
| `SONAR_TOKEN` | SonarQube authentication token (secret) |
| `SONAR_HOST_URL` | SonarQube server URL, e.g. `https://sonar.example.com` (variable) |

If SonarQube is not used, the scan step is automatically skipped (it checks `vars.SONAR_HOST_URL`).

---

## Step 5: Verify the Workflow

After pushing to the repository, the workflow should:

1. Check out the code
2. Install PHP 8.4 with required extensions
3. Cache and restore Composer dependencies
4. Set up a MariaDB 12.2 test database
5. Run Doctrine migrations on the test DB
6. Lint Twig templates
7. Lint the DI container
8. Run PHPStan static analysis (level 5)
9. Run PHPUnit unit tests with coverage output
10. Run SonarQube scan (if configured)

---

## Notes

- The `shivammathur/setup-php@v2` action is compatible with Gitea Act runners and GitHub Actions.
- If your Gitea instance uses a self-hosted runner without internet access, replace CDN-based action references with pinned versions from a local mirror.
- Coverage is enabled via `xdebug` in the PHP setup step. For faster runs on large test suites, switch to `pcov` which is faster than Xdebug for coverage only.
- The MariaDB service container is only needed if you have integration tests that require a real database. Pure unit tests do not need it — remove the `services` block if not needed.

---

## Alternative: Minimal CI (Unit Tests Only, No Database)

If the project only has unit tests with mocked dependencies (no DB integration tests), use this simplified workflow:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  qa:
    name: PHP QA
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.4'
          extensions: intl, mbstring, pdo_mysql, xml, curl, zip
          coverage: none

      - name: Cache Composer dependencies
        uses: actions/cache@v4
        with:
          path: vendor
          key: composer-${{ hashFiles('composer.lock') }}
          restore-keys: composer-

      - name: Install dependencies
        run: composer install --no-interaction --prefer-dist --optimize-autoloader

      - name: Lint Twig templates
        run: php bin/console lint:twig templates/ --env=dev

      - name: Lint DI container
        run: php bin/console lint:container --env=dev

      - name: PHPStan static analysis
        run: |
          php bin/console cache:warmup --env=dev
          vendor/bin/phpstan analyse --memory-limit=256M --no-progress

      - name: Unit tests
        run: php bin/phpunit tests/Unit --no-coverage
```
