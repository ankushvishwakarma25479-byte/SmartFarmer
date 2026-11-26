# Smart Farmer Procurement Management System

> An automated, queue-free, and transparent crop procurement management platform designed for farmers, mandis (procurement centres), and agricultural departments.

---

## Overview

The **Smart Farmer Procurement Management System** eliminates chaotic day-long mandi waiting times, guarantees transparent weighbridge quality inspection, and provides live fair queue tracking for farmers. Built to run with zero paid external services and no complex JavaScript build steps.

### Highlights
- **100% Free Stack**: Django 5, Bootstrap 5 (CDN), HTMX (CDN), Chart.js (CDN), SQLite for local development (PostgreSQL-ready).
- **Trilingual Accessibility**: Fully localized in **English**, **हिन्दी (Hindi)**, and **ਪੰਜਾਬੀ (Punjabi)** with session language switcher.
- **Role-Based Workflows**:
  - **Farmers**: OTP authentication, crop declarations, 1-hour slot reservation, automated FIFO waitlist promotion, QR arrival pass, live queue tracker.
  - **Centre Staff**: Desk dashboard, masked phone search, check-in validation, large atomic "Call Next" queue caller, weighbridge gross/tare weighing, moisture grading, and assisted walk-in booking.
  - **Administrators**: Centre & slot scheduling, emergency day/slot closures with automated farmer notification and rescheduling, queue policy configuration, simulated SMS audit log, immutable security audit trail with CSV export, and Chart.js analytics reports.
- **Zero-Cost Automation**: Dedicated management command `run_scheduled_jobs` runs appointment reminders, waitlist offer expiry, no-show sweeps, and SMS retries in a single synchronous pass for free external cron triggers.

---

## Quickstart & Setup

### 1. Prerequisites
- Python 3.11+
- Git

### 2. Environment Setup

```powershell
# Clone the repository
git clone <repo-url>
cd "Djnago Project"

# Create and activate Python virtual environment
python -m venv .venv
.\.venv\Scripts\Activate.ps1   # Windows PowerShell
# source .venv/bin/activate     # macOS / Linux

# Install dependencies
pip install -r requirements.txt
```

### 3. Environment Configuration
The project uses `django-environ`. A local `.env` is pre-configured for zero-setup SQLite development:

```ini
DJANGO_SECRET_KEY=django-insecure-smartfarmer-local-development-secret-key-391823
DJANGO_DEBUG=True
DJANGO_ALLOWED_HOSTS=localhost,127.0.0.1,[::1]
DJANGO_ENV=dev

# Zero-setup local SQLite database:
DATABASE_URL=sqlite:///db.sqlite3

# Optional: PostgreSQL in Docker or production
# DATABASE_URL=postgresql://smartfarmer_user:smartfarmer_password@localhost:5432/smartfarmer_db
```

### 4. Database Migrations & Translations

```powershell
# Apply database schema
python manage.py migrate

# Compile gettext translations (English, Hindi, Punjabi)
python scripts/compile_translations.py
```

### 5. Seed Demo Data

```powershell
# Seeds Ludhiana procurement centre, crops (Paddy & Wheat), seasons, slots, and demo accounts
python manage.py seed_demo
```

### 6. Run Development Server

```powershell
python manage.py runserver
```
Visit `http://localhost:8000/` in your browser.

---

## Demo Accounts & Role Access

Log in at `http://localhost:8000/login/`:

| Role | Mobile / Identifier | Authentication Flow | Dashboard URL | Capabilities |
| :--- | :--- | :--- | :--- | :--- |
| **Farmer** | Any 10-digit number (e.g. `9876543210` or `9876512345`) | Mobile OTP. In `DEBUG=True`, code appears on screen (e.g. `123456`). | `/dashboard/farmer/` | Submit crops, book slots, view waitlist, download QR pass, live queue tracking. |
| **Staff Member** | `9876500001` | Phone OTP (or Staff Tab) | `/staff/` | Assigned to Ludhiana Mandi. Check-in, Call Next, gross/tare weighbridge entry, complete procurement. |
| **Administrator** | `9876500002` | Phone OTP (or Staff Tab) | `/dashboard/admin/` | System KPIs, manage centres & slots, queue policy, audit logs, SMS logs, Chart.js analytics & CSV export. |

---

## Running the Demo Queue Simulator

To test the entire end-to-end procurement workflow (creating farmers, reserving slots, checking in at gate, calling next in queue, weighing grain, and recording payments), run the built-in simulator:

```powershell
# Standard run (4 sequential steps, 0.5s pause)
python manage.py simulate_queue

# Custom steps and delay
python manage.py simulate_queue --steps=5 --delay=1.0

# Target a specific procurement centre
python manage.py simulate_queue --centre=1
```

### What the Simulator Demonstrates:
1. **Infrastructure**: Initializes active centre, operating counters, and current season.
2. **Farmers & Bookings**: Concurrently creates farmers and books consecutive 1-hour slots with gapless token generation.
3. **Check-In**: Simulates arrival at mandi reception during check-in window and activates live queue tracking.
4. **Staff Queue Desk**: Calls next eligible farmer in queue order.
5. **Weighbridge & Inspection**: Records gross and tare weights, tests moisture, assigns quality grade, and completes procurement.

---

## Scheduled Maintenance Jobs (`run_scheduled_jobs`)

For production setups without persistent paid Celery Beat workers, the `run_scheduled_jobs` command executes all periodic procurement background maintenance tasks in a single pass:

```powershell
# Run all scheduled maintenance tasks
python manage.py run_scheduled_jobs

# Run only a specific task
python manage.py run_scheduled_jobs --only reminders   # Day-before SMS appointment reminders
python manage.py run_scheduled_jobs --only waitlist    # Expire unanswered offers & promote next farmer
python manage.py run_scheduled_jobs --only no-show     # Mark missed slots NO_SHOW & release slot capacity
python manage.py run_scheduled_jobs --only retries     # Retry pending/failed SMS notifications
```

### Free External Scheduler Integration

You can trigger `run_scheduled_jobs` periodically using any free cron service:

#### 1. Linux Crontab / Systemd Timer
Run every 15 minutes:
```bash
*/15 * * * * cd /home/ubuntu/smartfarmer && /home/ubuntu/smartfarmer/.venv/bin/python manage.py run_scheduled_jobs >> /var/log/smartfarmer_cron.log 2>&1
```

#### 2. Free Webhook Trigger (cron-job.org / EasyCron)
Configure a free scheduled HTTP GET every 10–15 minutes hitting your application health check or trigger URL:
```
https://your-smartfarmer-domain.com/api/health/
```

#### 3. GitHub Actions (Free Scheduled Workflow)
Create `.github/workflows/scheduled_jobs.yml`:
```yaml
name: Scheduled Procurement Jobs
on:
  schedule:
    - cron: '*/30 * * * *'  # Runs every 30 minutes
  workflow_dispatch:

jobs:
  maintenance:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      - name: Run Maintenance Jobs
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
        run: |
          pip install -r requirements.txt
          python manage.py run_scheduled_jobs
```

#### 4. Windows Task Scheduler
1. Open **Task Scheduler** > **Create Basic Task**.
2. Trigger: **Daily** (repeat every 1 hour).
3. Action: **Start a program**:
   - Program: `C:\path\to\project\.venv\Scripts\python.exe`
   - Arguments: `manage.py run_scheduled_jobs`
   - Start in: `C:\path\to\project\`

---

## Responsive Phone Layouts (360px Mobile)

The frontend is optimized for low-end Android mobile devices and 360px viewport widths:
- Touch-friendly buttons (`min-height: 44px`).
- Responsive typography (`clamp` font sizes and fluid badge paddings).
- Horizontal scrolling for weighbridge tables (`.table-responsive`).
- 2x2 dashboard metric grids on mobile screens for live queue tracking (`Token Served`, `My Token`, `Farmers Ahead`, `Status`).
- Zero JavaScript build step — native Bootstrap 5 and HTMX.

---

## Running the Automated Test Suite

The test suite runs with `pytest` and uses an isolated SQLite test database.

```powershell
# Run all tests (199 passing tests)
pytest

# Run tests with verbose output
pytest -v

# Run only scheduled jobs tests
pytest tests/test_scheduled_jobs.py -v

# Run admin portal tests
pytest tests/test_admin_portal.py -v

# Run farmer pages tests
pytest tests/test_farmer_pages.py -v
```

---

## Project Structure

```text
smartfarmer/
├── accounts/           # User authentication, OTP service, farmer & staff profiles
├── centres/            # Mandis, crops, seasons, slots management, seed demo command
├── bookings/           # Crop submissions, slot bookings, FIFO waitlist, QR pass generator
├── queueing/           # Queue tokens, live state calculation, staff console, audit logging
├── procurement/        # Weighbridge inspection, gross/tare weights, moisture grading, payments
├── notifications/      # Multilingual SMS templates, simulated SMS gateway, scheduled jobs command
├── reports/            # Procurement reports, Chart.js data models, CSV export, admin web views
├── scripts/            # Translation compilation tools (compile_translations.py)
├── templates/          # Responsive Bootstrap 5 + HTMX templates
│   ├── accounts/       # Login, OTP verification, register, profile completion
│   ├── admin/          # Admin dashboard, centres, slots, queue policy, logs, reports
│   ├── farmer/         # Farmer dashboard, crop declarations, slot booking, queue tracker, QR pass
│   ├── public/         # Landing page, about, how it works, centre explorer
│   └── staff/          # Staff weighbridge desk console & HTMX partials
└── tests/              # 199 automated pytest tests covering all apps and role permissions
```

---

## License

This project is open-source and intended for agricultural research, university projects, and transparent public procurement operations.
