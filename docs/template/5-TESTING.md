# Testing Setup — PHPUnit and PHPStan

This prompt configures the project's testing infrastructure.

**Prerequisites:** Complete [2-BACKEND.md](2-BACKEND.md) first.

---

## Step 1: Ask the User

Before doing anything, ask:

```
Do you want to set up automated testing for this project?

This will install:
  - PHPUnit 11.x — unit and integration testing
  - PHPStan level 5 — static analysis

It will also:
  - Create a phpunit.xml.dist config
  - Create a phpstan.neon config
  - Create the tests/Unit/ folder structure
  - Update CLAUDE.md to require unit tests for all created code

Answer: yes / no
```

If the user answers **no**, skip all remaining steps in this file.

---

## Step 2: Install Testing Dependencies

```bash
composer require --dev phpunit/phpunit:"^11.5"
composer require --dev phpstan/phpstan:"^2.0"
composer require --dev phpstan/extension-installer
composer require --dev phpstan/phpstan-symfony
composer require --dev phpstan/phpstan-doctrine
```

---

## Step 3: Create PHPUnit Configuration

Create `phpunit.xml.dist` at the project root:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="vendor/phpunit/phpunit/phpunit.xsd"
         bootstrap="vendor/autoload.php"
         cacheDirectory=".phpunit.cache"
         executionOrder="depends,defects"
         requireCoverageMetadata="false"
         beStrictAboutCoverageMetadata="false">

    <testsuites>
        <testsuite name="Unit">
            <directory>tests/Unit</directory>
        </testsuite>
    </testsuites>

    <source>
        <include>
            <directory>src</directory>
        </include>
    </source>

    <coverage>
        <report>
            <html outputDirectory="var/coverage"/>
            <text outputFile="php://stdout" showUncoveredFiles="false"/>
        </report>
    </coverage>
</phpunit>
```

---

## Step 4: Create PHPStan Configuration

Create `phpstan.neon` at the project root:

```neon
parameters:
    level: 5
    paths:
        - src
    symfony:
        containerXmlPath: var/cache/dev/App_KernelDevDebugContainer.xml
    doctrine:
        objectManagerLoader: tests/object-manager.php
    ignoreErrors: []
    excludePaths:
        - src/Kernel.php

includes:
    - vendor/phpstan/phpstan-symfony/extension.neon
    - vendor/phpstan/phpstan-doctrine/extension.neon
```

Create `tests/object-manager.php` for PHPStan Doctrine integration:

```php
<?php

require_once __DIR__ . '/../vendor/autoload.php';

$kernel = new App\Kernel('dev', true);
$kernel->boot();

return $kernel->getContainer()->get('doctrine')->getManager();
```

---

## Step 5: Create Test Folder Structure

```bash
mkdir -p tests/Unit/Service
mkdir -p tests/Unit/Controller
mkdir -p tests/Unit/Twig
mkdir -p tests/Unit/Command
mkdir -p tests/Unit/Repository
```

Create `tests/Unit/.gitkeep` in each empty directory (or add a sample test — see below).

---

## Step 6: Create a Sample Unit Test

Create `tests/Unit/ExampleTest.php` to verify the test suite runs:

```php
<?php

declare(strict_types=1);

namespace App\Tests\Unit;

use PHPUnit\Framework\TestCase;

class ExampleTest extends TestCase
{
    public function testTrueIsTrue(): void
    {
        $this->assertTrue(true);
    }
}
```

Run the test suite:

```bash
php bin/phpunit tests/Unit --no-coverage
```

Run static analysis:

```bash
# Warm up the Symfony container first (needed for PHPStan Symfony extension)
php bin/console cache:warmup --env=dev

vendor/bin/phpstan analyse --memory-limit=256M --no-progress
```

Both commands should pass with no errors.

---

## Step 7: Update CLAUDE.md

Add the following rule to `CLAUDE.md` under the **Testing** section:

```markdown
## Testing Requirements

- Every new service, command, Twig extension, or repository must have a corresponding unit test file.
- Test files go in `tests/Unit/` mirroring the `src/` directory structure:
  - `src/Service/FooService.php` → `tests/Unit/Service/FooServiceTest.php`
  - `src/Command/FooCommand.php` → `tests/Unit/Command/FooCommandTest.php`
  - `src/Twig/AppExtension.php` → `tests/Unit/Twig/AppExtensionTest.php`
- Unit tests must extend `PHPUnit\Framework\TestCase`
- Mock all external dependencies (database, HTTP clients, etc.)
- Run tests before marking any task as complete: `php bin/phpunit tests/Unit --no-coverage`
- Run static analysis: `vendor/bin/phpstan analyse --memory-limit=256M --no-progress`
```

---

## Testing Conventions

### Unit Test Template

Use this as the starting point for every new test class:

```php
<?php

declare(strict_types=1);

namespace App\Tests\Unit\Service;

use App\Service\MyService;
use PHPUnit\Framework\TestCase;

class MyServiceTest extends TestCase
{
    private MyService $service;

    protected function setUp(): void
    {
        // Initialize the service under test with mocked dependencies
        $this->service = new MyService(/* mock dependencies */);
    }

    public function testSomeBehavior(): void
    {
        // Arrange
        $input = 'some input';

        // Act
        $result = $this->service->process($input);

        // Assert
        $this->assertSame('expected output', $result);
    }
}
```

### Mocking Dependencies

```php
// Mock a repository
$repo = $this->createMock(UserRepository::class);
$repo->method('find')->willReturn(new User());

// Mock a service
$mailer = $this->createMock(MailerInterface::class);
$mailer->expects($this->once())->method('send');
```

---

## Useful Commands

```bash
# Run all unit tests
php bin/phpunit tests/Unit --no-coverage

# Run with coverage report (HTML at var/coverage/)
php bin/phpunit tests/Unit --coverage-html var/coverage

# Run a single test file
php bin/phpunit tests/Unit/Service/MyServiceTest.php

# Run static analysis
vendor/bin/phpstan analyse --memory-limit=256M

# Lint Twig templates
php bin/console lint:twig templates/

# Lint Symfony DI container
php bin/console lint:container
```
