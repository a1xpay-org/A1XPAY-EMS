# A1XPAY EMS — Employee Management System

A role-based **Employee Management System** built for **A1XPAY PVT LTD**. It covers day-to-day HR operations (attendance, leave, holidays, offer letters), **location-verified check-in/out**, a flexible **roles & permissions** layer, **client/project revenue tracking**, and **digital business cards + ID cards with QR verification** — all in one admin panel.

> Built with Laravel 12, plain Blade views and a custom design system (no front-end build step required).

---

## Features

### People & HR operations
- **Employees** — create / edit / deactivate, bulk-set cost per day, employee profile with city & address.
- **Attendance** — employee self check-in / check-out with a mandatory work note; HR/Admin manual entry; late detection and working-hours calculation.
- **Leave management** — employees apply, **HR approves → Super Admin gives final approval** (or rejects).
- **Holidays** — yearly holiday calendar, visible to every role.
- **Offer letters** — generate, download as PDF and send by email.
- **Reports** — Attendance, Leave, Employee and Revenue reports with filters and **CSV / PDF export**.

### Attendance intelligence
- **IP-based location verification** — the check-in/out IP is resolved to a city and compared with the employee's registered city. A confirmed mismatch is blocked; if the location cannot be resolved (private IP, API down) the check-in is allowed and stored as *Unverified* so attendance never breaks.
- **Activity status** — a lightweight tab-visibility heartbeat shows who is currently Active / Inactive.

### Roles & permissions
- **Super Admin** has full access; everyone else gets access from a **custom Role** with a **read / write toggle per tab** (write always implies read).
- Admin users are assigned one role; a Super-Admin flag overrides the role.
- A **JSON login API** (`POST /api/login`) returns the user, `is_super_admin` and the full permissions object.
- Sidebar and routes honour the permissions (`tab:<name>,read|write` middleware).

### Revenue tracking
- **Clients** and **Projects** (fixed monthly retainer or hourly billing) with employee assignment.
- A single `RevenueService` calculates **Revenue, Cost, Profit and Margin %** — reused by the Dashboard cards, the Revenue report, the per-employee **Performance** page (plus a self-service "My Performance") and the **Profitability Report** (client-level and project-level).

### Digital Business Card & ID Card
- Every employee gets a **permanent, unguessable public link + QR code** for their business card; HR decides (tick / untick) which details are public. Hidden fields are never sent to the page.
- A separate **ID card** (front and back, printable) with its **own QR and verification link**. The verification page shows *Activation pending* until HR activates the card, and then only a minimal fixed set of details (name, employee ID, designation, department, issue date).
- Employees can only **view** their own cards; HR/Admin (or any role granted the *ID & Digital Cards* tab) manage everyone's.

---

## Tech stack

| Layer | Technology |
|---|---|
| Backend | PHP 8.2+, Laravel 12 |
| Database | MySQL / MariaDB |
| Auth & permissions | Laravel auth + [spatie/laravel-permission](https://github.com/spatie/laravel-permission) + custom tab-level read/write roles |
| PDF | [barryvdh/laravel-dompdf](https://github.com/barryvdh/laravel-dompdf) |
| Front-end | Blade templates, custom CSS design system, vanilla JS |
| Icons / charts / QR | [Lucide](https://lucide.dev), [Chart.js](https://www.chartjs.org), [qrcode-generator](https://github.com/kazuhikoarase/qrcode-generator) (all via CDN) |
| IP geolocation | [ip-api.com](https://ip-api.com) (free tier, no key) |

---

## Getting started

### Requirements
- PHP **8.2+** with the usual Laravel extensions (`mbstring`, `openssl`, `pdo_mysql`, `gd`, `xml`, `curl`)
- Composer 2
- MySQL / MariaDB
- Internet access for the CDN libraries and IP geolocation

### Install

```bash
git clone <your-repo-url> a1xpay-ems
cd a1xpay-ems

composer install
cp .env.example .env
php artisan key:generate
```

Create an empty database, put its credentials in `.env` (`DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD`), then:

```bash
php artisan migrate
php artisan db:seed      # creates roles, permissions and the first Super Admin
php artisan serve
```

The seeder creates the Super Admin `admin@a1xpay.com` and **prints a random password once in the terminal — save it immediately** (it is not shown again). Change it after the first login.

> Run `db:seed` only on a fresh database. It uses `firstOrCreate`, so on an existing install it will not change the admin's password.

### Key `.env` settings

| Key | Why it matters |
|---|---|
| `APP_URL` | **Must be the real public URL in production.** QR codes encode links built from it. |
| `DB_*` | Database connection |
| `MAIL_*` | Needed for credential e-mails and offer-letter e-mails |
| `APP_ENV=production`, `APP_DEBUG=false` | Required on a live server |

If the app runs behind a reverse proxy / Cloudflare, configure Laravel's trusted proxies so the real client IP reaches the location check.

---

## Project structure

```
app/
  Http/Controllers/     # Attendance, Leave, Employee, Report, Revenue, Cards, Roles ...
  Http/Middleware/      # EnsureTabAccess (tab:cards,write)
  Models/               # User, AdminRole, Attendance, Leave, Client, Project ...
  Services/             # RevenueService, IpGeolocationService, AttendanceService, ReportService
config/
  admin_tabs.php        # the tabs that can be given read/write permission
  card_fields.php       # fields a user may show on the public business card
  company.php           # company name, support e-mail, website, logo, services
database/migrations/    # schema (roles, location tracking, revenue, cards ...)
resources/views/        # Blade views (layouts, per-role pages, digital-card, cards ...)
public/css/app.css      # the design system
public/images/          # company logo
routes/web.php          # all routes (public card/verify routes + role-prefixed groups)
```

### Public routes

| Route | Purpose |
|---|---|
| `GET /card/{token}` | Public business card (opened by the card QR) |
| `GET /verify/{token}` | ID-card identity verification (opened by the ID QR) |
| `POST /api/login` | JSON login returning permissions |

Tokens are 40-character random strings, the pages are rate-limited and sent with `noindex`.

---

## Deployment checklist

```bash
php artisan migrate --force
php artisan optimize:clear
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

Take a database backup (`mysqldump`) before migrating, set the real `APP_URL`, and make sure `public/images/a1xpay-logo.png` is uploaded (the logo is used on cards and the sidebar). No `npm run build` step is needed.

---

## Security notes

- `.env` is git-ignored — never commit credentials.
- Public card/verify pages expose only what HR ticked (or a fixed minimal set for ID verification) and return the same 404 for unknown tokens and deactivated users.
- Route access is enforced on the server (role middleware and tab-permission middleware), not just by hiding menu items.

---

## License

Private project of **A1XPAY PVT LTD**. All rights reserved.
