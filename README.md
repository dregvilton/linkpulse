# LinkPulse

A self-hosted smart URL shortener and routing engine.

Early development: iteration 1 provides the Laravel foundation and standard authentication only.

## Requirements

- PHP 8.5 with Laravel's required extensions and `pdo_pgsql`.
- Composer 2.
- Node.js 24 LTS (24.11+) and npm; Node 22.18+ is also supported. Node 23 is unsupported. `.nvmrc` selects Node 24.
- A running PostgreSQL 14+ server and its command-line tools.

## Local setup

```sh
git clone https://github.com/dregvilton/linkpulse.git
cd linkpulse
composer install
cp .env.example .env
php artisan key:generate
```

With PostgreSQL running, create a local login role and two databases using a PostgreSQL administrator account:

```sh
createuser --pwprompt linkpulse
createdb --owner=linkpulse linkpulse
createdb --owner=linkpulse linkpulse_testing
```

Choose your own password at the prompt. If your PostgreSQL installation requires it, supply the administrator's `--host`, `--port` and `--username` to these commands.

Set the matching values in your untracked `.env`:

```dotenv
DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=linkpulse
DB_USERNAME=linkpulse
DB_PASSWORD=your-local-password
```

Laravel reads these environment values through `config/database.php`. Keep credentials in `.env`, never in Git. After changing settings, run `php artisan config:clear` if configuration was cached.

```sh
php artisan migrate
npm ci
npm run build
composer run dev
```

Open http://localhost:8000. Registration, login and `/dashboard` use the official Livewire starter kit. `composer run dev` starts the PHP server and Vite with asset hot reload. You can also run `php artisan serve` and `npm run dev` in separate terminals. `npm run build` produces assets for serving without Vite.

Sessions and cache use PostgreSQL. Queued work runs synchronously; no queue worker or Redis server is needed. After registering, open the email verification link written to `storage/logs/laravel.log` (`MAIL_MAILER=log`) to access the dashboard.

## Tests

The standard starter-kit tests run against PostgreSQL, including registration, login and dashboard access:

```sh
php artisan test
composer test
```

`composer test` also checks formatting with Pint and types with Larastan. `phpunit.xml` forces PostgreSQL and the dedicated `linkpulse_testing` database, which the test suite resets. Never point tests at a database containing data you need. Connection credentials come from `.env`; use an untracked `.env.testing` if the test server or role differs. No manual test migrations are needed: Laravel's `RefreshDatabase` trait applies them.

GitHub Actions installs the locked dependencies, builds assets and runs `composer ci:check` with its own PostgreSQL service.
