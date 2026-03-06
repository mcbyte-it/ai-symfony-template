# Backend Setup — Symfony 7.4 on PHP 8.4

This prompt sets up the Symfony backend, database, security, and user management.

**Prerequisites:** Complete [1-NEW_PROJECT.md](1-NEW_PROJECT.md) first.

---

## Step 1: Initialize Symfony Project

Run the following commands to create the Symfony 7.4 project:

```bash
composer create-project symfony/skeleton:"7.4.*" _tmp
(shopt -s dotglob; mv _tmp/* .)
rm -rf _tmp
```

Then install required packages:

```bash
composer require symfony/flex
composer require symfony/twig-bundle
composer require symfony/security-bundle
composer require symfony/rate-limiter
composer require symfony/apache-pack
composer require symfony/maker-bundle --dev
composer require doctrine/doctrine-bundle
composer require doctrine/doctrine-migrations-bundle
composer require doctrine/orm:"^3.0"
composer require doctrine/dbal:"^4.4"
composer require symfony/profiler-pack --dev
```

The `profiler-pack` installs `WebProfilerBundle`, `DebugBundle`, and `MonologBundle`. Symfony Flex automatically registers them only for the `dev` environment — no manual bundle configuration is required.

Verify installed versions match:
- `doctrine/orm` >= 3.0
- `doctrine/dbal` >= 4.4
- `symfony/*` = 7.4.x
- PHP runtime = 8.4

---

## Step 1b: Internationalisation (i18n) — Only if USE_I18N = Yes

Read `USE_I18N`, `I18N_LANGUAGES`, and `USE_RTL` from `0-PROJECT_DESCRIPTION.md`. If `USE_I18N` is **No**, skip this entire step.

### Install Symfony Translation Packages

```bash
composer require symfony/translation
```

### Configure the Translation Component

Edit `config/packages/translation.yaml`:

```yaml
framework:
    default_locale: en
    translator:
        default_path: '%kernel.project_dir%/translations'
        fallbacks:
            - en
        paths:
            - '%kernel.project_dir%/translations'
```

### Configure Available Locales

In `config/services.yaml`, define the supported locales as a parameter:

```yaml
parameters:
    app.supported_locales: ['en', 'ar']   # add/remove based on I18N_LANGUAGES
```

### Enable URL Prefix Locale Switching

Edit `config/routes.yaml` to wrap all routes under a locale prefix:

```yaml
controllers:
    resource:
        path: ../src/Controller/
        namespace: App\Controller
    type: attribute
    prefix: /{_locale}
    requirements:
        _locale: en|ar   # match app.supported_locales
    defaults:
        _locale: en
```

### Create a Locale Switcher Controller

Create `src/Controller/LocaleController.php`:

```php
<?php

declare(strict_types=1);

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\RedirectResponse;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Routing\Attribute\Route;

class LocaleController extends AbstractController
{
    /** @param list<string> $supportedLocales */
    public function __construct(
        private readonly array $supportedLocales,
    ) {}

    #[Route('/locale/{locale}', name: 'app_switch_locale')]
    public function switchLocale(string $locale, Request $request): RedirectResponse
    {
        if (!in_array($locale, $this->supportedLocales, true)) {
            $locale = 'en';
        }

        $request->getSession()->set('_locale', $locale);
        $referer = $request->headers->get('referer', $this->generateUrl('app_dashboard'));

        return $this->redirect($referer);
    }
}
```

Register the supported locales in `config/services.yaml`:

```yaml
App\Controller\LocaleController:
    arguments:
        $supportedLocales: '%app.supported_locales%'
```

### Create Translation Files

Create the folder structure for translation files:

```
translations/
  messages.en.yaml
  messages.ar.yaml
```

Starter `translations/messages.en.yaml`:

```yaml
nav:
    dashboard: Dashboard
    logout: Logout

action:
    save: Save
    edit: Edit
    delete: Delete
    cancel: Cancel

confirm:
    delete: Are you sure you want to delete this item?

greeting:
    user: Welcome, %name%!
```

Starter `translations/messages.ar.yaml`:

```yaml
nav:
    dashboard: لوحة التحكم
    logout: تسجيل الخروج

action:
    save: حفظ
    edit: تعديل
    delete: حذف
    cancel: إلغاء

confirm:
    delete: هل أنت متأكد أنك تريد حذف هذا العنصر؟

greeting:
    user: مرحباً، %name%!
```

Add one YAML file per language defined in `I18N_LANGUAGES`. The file name format is `messages.[locale].yaml`.

### Use Translations in Twig

In all Twig templates, use the `trans` filter or tag instead of raw strings:

```twig
{# Filter syntax #}
{{ 'nav.dashboard'|trans }}

{# Tag syntax with parameters #}
{% trans with {'%name%': user.username} %}welcome.message{% endtrans %}
```

### RTL Support Note

RTL stylesheet switching is handled in `4-FRONTEND.md`. The backend only needs to expose the current locale to Twig (it does this automatically via `app.request.locale`).

---

## Step 2: Configure MariaDB with Table Prefix

### Environment

In `.env`, set the database URL (user fills in credentials):

```dotenv
DATABASE_URL="mysql://db_user:db_pass@127.0.0.1:3306/db_name?serverVersion=mariadb-12.2.0&charset=utf8mb4"
TABLE_PREFIX=app_
```

### Doctrine Configuration

Edit `config/packages/doctrine.yaml` to apply the table prefix globally using a custom naming strategy:

Create `src/Doctrine/PrefixedNamingStrategy.php`:

```php
<?php

declare(strict_types=1);

namespace App\Doctrine;

use Doctrine\ORM\Mapping\UnderscoreNamingStrategy;

class PrefixedNamingStrategy extends UnderscoreNamingStrategy
{
    public function __construct(private readonly string $prefix) {}

    public function classToTableName(string $className): string
    {
        return $this->prefix . parent::classToTableName($className);
    }
}
```

Register it in `config/services.yaml`:

```yaml
services:
    App\Doctrine\PrefixedNamingStrategy:
        arguments:
            $prefix: '%env(TABLE_PREFIX)%'

    doctrine.orm.naming_strategy:
        alias: App\Doctrine\PrefixedNamingStrategy
```

Update `config/packages/doctrine.yaml`:

```yaml
doctrine:
    dbal:
        url: '%env(resolve:DATABASE_URL)%'
    orm:
        auto_generate_proxy_classes: true
        naming_strategy: App\Doctrine\PrefixedNamingStrategy
        auto_mapping: true
        mappings:
            App:
                type: attribute
                is_bundle: false
                dir: '%kernel.project_dir%/src/Entity'
                prefix: 'App\Entity'
                alias: App
```

---

## Step 3: Create the Users Table via Migration

### Create the User Entity

Create `src/Entity/User.php`:

```php
<?php

declare(strict_types=1);

namespace App\Entity;

use App\Repository\UserRepository;
use DateTimeImmutable;
use Doctrine\ORM\Mapping as ORM;
use Symfony\Component\Security\Core\User\PasswordAuthenticatedUserInterface;
use Symfony\Component\Security\Core\User\UserInterface;

#[ORM\Entity(repositoryClass: UserRepository::class)]
#[ORM\HasLifecycleCallbacks]
class User implements UserInterface, PasswordAuthenticatedUserInterface
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column(type: 'integer')]
    private ?int $id = null;

    #[ORM\Column(type: 'string', length: 180, unique: true)]
    private string $username;

    #[ORM\Column(type: 'string', length: 255, unique: true)]
    private string $email;

    #[ORM\Column(type: 'string')]
    private string $password;

    #[ORM\Column(type: 'string', length: 255)]
    private string $realName;

    #[ORM\Column(type: 'json')]
    private array $roles = [];

    #[ORM\Column(type: 'boolean')]
    private bool $isActive = true;

    #[ORM\Column(type: 'boolean')]
    private bool $mustChangePw = false;

    #[ORM\Column(type: 'smallint')]
    private int $failedAttempts = 0;

    #[ORM\Column(type: 'datetime_immutable', nullable: true)]
    private ?DateTimeImmutable $lockedUntil = null;

    #[ORM\Column(type: 'datetime_immutable')]
    private DateTimeImmutable $createdAt;

    #[ORM\Column(type: 'datetime_immutable')]
    private DateTimeImmutable $updatedAt;

    #[ORM\PrePersist]
    public function onPrePersist(): void
    {
        $this->createdAt = new DateTimeImmutable();
        $this->updatedAt = new DateTimeImmutable();
    }

    #[ORM\PreUpdate]
    public function onPreUpdate(): void
    {
        $this->updatedAt = new DateTimeImmutable();
    }

    public function getId(): ?int { return $this->id; }

    public function getUsername(): string { return $this->username; }
    public function setUsername(string $username): static { $this->username = $username; return $this; }

    public function getEmail(): string { return $this->email; }
    public function setEmail(string $email): static { $this->email = $email; return $this; }

    public function getUserIdentifier(): string { return $this->username; }

    public function getPassword(): string { return $this->password; }
    public function setPassword(string $password): static { $this->password = $password; return $this; }

    public function getRealName(): string { return $this->realName; }
    public function setRealName(string $realName): static { $this->realName = $realName; return $this; }

    public function getRoles(): array
    {
        $roles = $this->roles;
        $roles[] = 'ROLE_USER';
        return array_unique($roles);
    }
    public function setRoles(array $roles): static { $this->roles = $roles; return $this; }

    public function isActive(): bool { return $this->isActive; }
    public function setIsActive(bool $isActive): static { $this->isActive = $isActive; return $this; }

    public function isMustChangePw(): bool { return $this->mustChangePw; }
    public function setMustChangePw(bool $mustChangePw): static { $this->mustChangePw = $mustChangePw; return $this; }

    public function getFailedAttempts(): int { return $this->failedAttempts; }
    public function setFailedAttempts(int $failedAttempts): static { $this->failedAttempts = $failedAttempts; return $this; }

    public function getLockedUntil(): ?DateTimeImmutable { return $this->lockedUntil; }
    public function setLockedUntil(?DateTimeImmutable $lockedUntil): static { $this->lockedUntil = $lockedUntil; return $this; }

    public function getCreatedAt(): DateTimeImmutable { return $this->createdAt; }
    public function getUpdatedAt(): DateTimeImmutable { return $this->updatedAt; }

    public function eraseCredentials(): void {}
}
```

### Create the RememberMeToken Entity

Create `src/Entity/RememberMeToken.php`:

```php
<?php

declare(strict_types=1);

namespace App\Entity;

use DateTimeImmutable;
use Doctrine\ORM\Mapping as ORM;

#[ORM\Entity]
class RememberMeToken
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column(type: 'integer')]
    private ?int $id = null;

    #[ORM\Column(type: 'string', length: 180)]
    private string $username;

    #[ORM\Column(type: 'string', length: 255, unique: true)]
    private string $value;

    #[ORM\Column(type: 'datetime_immutable')]
    private DateTimeImmutable $lastUsed;

    public function getId(): ?int { return $this->id; }
    public function getUsername(): string { return $this->username; }
    public function setUsername(string $username): static { $this->username = $username; return $this; }
    public function getValue(): string { return $this->value; }
    public function setValue(string $value): static { $this->value = $value; return $this; }
    public function getLastUsed(): DateTimeImmutable { return $this->lastUsed; }
    public function setLastUsed(DateTimeImmutable $lastUsed): static { $this->lastUsed = $lastUsed; return $this; }
}
```

### Generate and Run Migration

```bash
php bin/console doctrine:migrations:diff
php bin/console doctrine:migrations:migrate
```

Verify the generated migration SQL uses the correct table prefix (e.g. `app_user`, `app_remember_me_token`).

---

## Step 4: Ask About RBAC / RLS

Ask the user:

```
Do you want Role-Based Access Control (RBAC) or Row-Level Security (RLS)?
- RBAC: A separate `roles` table with role-to-user assignments (recommended for multi-role systems)
- RLS: Data-level filtering per user (advanced, database-side)
- Neither: Roles stored directly on the User entity (already configured above)

Answer: RBAC / RLS / Neither
```

### If RBAC is chosen:

Create `src/Entity/Role.php`:

```php
<?php

declare(strict_types=1);

namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;

#[ORM\Entity]
class Role
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column(type: 'integer')]
    private ?int $id = null;

    #[ORM\Column(type: 'string', length: 100, unique: true)]
    private string $name; // e.g. ROLE_ADMIN

    #[ORM\Column(type: 'string', length: 255)]
    private string $label; // e.g. Administrator

    public function getId(): ?int { return $this->id; }
    public function getName(): string { return $this->name; }
    public function setName(string $name): static { $this->name = $name; return $this; }
    public function getLabel(): string { return $this->label; }
    public function setLabel(string $label): static { $this->label = $label; return $this; }
}
```

Create a `user_roles` join table via migration. Run `doctrine:migrations:diff` and `doctrine:migrations:migrate` again.

---

## Step 5: Configure Symfony Security

Edit `config/packages/security.yaml`:

```yaml
security:
    password_hashers:
        App\Entity\User:
            algorithm: argon2id
            memory_cost: 65536
            time_cost: 4
            threads: 1

    providers:
        app_user_provider:
            entity:
                class: App\Entity\User
                property: username

    firewalls:
        dev:
            pattern: ^/(_(profiler|wdt)|css|images|js)/
            security: false

        main:
            lazy: true
            provider: app_user_provider
            form_login:
                login_path: app_login
                check_path: app_login
                enable_csrf: true
                default_target_path: app_dashboard
            logout:
                path: app_logout
                target: app_login
            remember_me:
                secret: '%kernel.secret%'
                lifetime: 604800   # 7 days
                path: /

    access_control:
        - { path: ^/login, roles: PUBLIC_ACCESS }
        - { path: ^/, roles: ROLE_USER }
```

---

## Step 6: Configure Rate Limiter

Edit `config/packages/rate_limiter.yaml`:

```yaml
framework:
    rate_limiter:
        login_limiter:
            policy: 'fixed_window'
            limit: 5
            interval: '15 minutes'
```

Apply the rate limiter in the login controller (see below).

---

## Step 7: Create Authentication Controller

Create `src/Controller/SecurityController.php`:

```php
<?php

declare(strict_types=1);

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;
use Symfony\Component\Security\Http\Authentication\AuthenticationUtils;

class SecurityController extends AbstractController
{
    #[Route('/login', name: 'app_login')]
    public function login(AuthenticationUtils $authenticationUtils): Response
    {
        if ($this->getUser()) {
            return $this->redirectToRoute('app_dashboard');
        }

        return $this->render('security/login.html.twig', [
            'last_username' => $authenticationUtils->getLastUsername(),
            'error' => $authenticationUtils->getLastAuthenticationError(),
        ]);
    }

    #[Route('/logout', name: 'app_logout')]
    public function logout(): never
    {
        throw new \LogicException('This method should never be reached.');
    }
}
```

Create `templates/security/login.html.twig` with a CSRF-protected Bootstrap 5 login form including a remember-me checkbox.

---

## Step 8: Create User Creation Command

Create `src/Command/CreateUserCommand.php`:

```php
<?php

declare(strict_types=1);

namespace App\Command;

use App\Entity\User;
use Doctrine\ORM\EntityManagerInterface;
use Symfony\Component\Console\Attribute\AsCommand;
use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Input\InputOption;
use Symfony\Component\Console\Output\OutputInterface;
use Symfony\Component\Console\Style\SymfonyStyle;
use Symfony\Component\PasswordHasher\Hasher\UserPasswordHasherInterface;

#[AsCommand(name: 'app:create-user', description: 'Create a new application user')]
class CreateUserCommand extends Command
{
    public function __construct(
        private readonly EntityManagerInterface $em,
        private readonly UserPasswordHasherInterface $hasher,
    ) {
        parent::__construct();
    }

    protected function configure(): void
    {
        $this
            ->addOption('username', null, InputOption::VALUE_OPTIONAL, 'Username')
            ->addOption('email', null, InputOption::VALUE_OPTIONAL, 'Email address')
            ->addOption('real-name', null, InputOption::VALUE_OPTIONAL, 'Real name')
            ->addOption('role', null, InputOption::VALUE_OPTIONAL, 'Role (e.g. ROLE_ADMIN)', 'ROLE_USER')
            ->addOption('must-change-pw', null, InputOption::VALUE_NONE, 'Force password change on first login');
    }

    protected function execute(InputInterface $input, OutputInterface $output): int
    {
        $io = new SymfonyStyle($input, $output);

        $username = $input->getOption('username') ?? $io->ask('Username');
        $email = $input->getOption('email') ?? $io->ask('Email');
        $realName = $input->getOption('real-name') ?? $io->ask('Real name');
        $plainPassword = $io->askHidden('Password (input hidden)');
        $role = $input->getOption('role') ?? $io->ask('Role', 'ROLE_USER');

        if (!$username || !$email || !$plainPassword) {
            $io->error('Username, email, and password are required.');
            return Command::FAILURE;
        }

        $user = new User();
        $user->setUsername($username);
        $user->setEmail($email);
        $user->setRealName($realName ?? $username);
        $user->setRoles([$role]);
        $user->setPassword($this->hasher->hashPassword($user, $plainPassword));
        $user->setMustChangePw($input->getOption('must-change-pw'));

        $this->em->persist($user);
        $this->em->flush();

        $io->success(sprintf('User "%s" created successfully.', $username));
        return Command::SUCCESS;
    }
}
```

Run: `php bin/console app:create-user`

---

## Security Checklist

Before finishing, verify:

- [ ] CSRF protection enabled on login form (`enable_csrf: true` in security.yaml)
- [ ] Password hashing uses `argon2id` with hardened memory/time settings
- [ ] Rate limiter applied to login endpoint
- [ ] All routes require `ROLE_USER` except `/login`
- [ ] No SQL string interpolation anywhere — use parameterized queries only
- [ ] `TABLE_PREFIX` env variable applied to all table names via naming strategy
- [ ] `.env` is in `.gitignore`; only `.env.example` is committed
