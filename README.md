# FleetPro — Premium UI & AI edition

An improved version of the supplied Django/React rental project, with separate customer and employee experiences, vehicle recommendations, and a consent-based AI concierge.

Start here. Read **docs/PROJECT_REVIEW.md** for the original drawbacks, implemented fixes, remaining limitations, and improvement priorities. The latest hardening update adds verified gateway settlement, branch isolation, private KYC storage, encrypted backups and deployment gates. Read **docs/PRODUCTION_READINESS.md** for the distinction between implemented safeguards and live checks still awaiting credentials/server access.

## Run locally

Use Python 3.12 and Node 20 or later. The local profile uses SQLite, an in-memory cache, synchronous tasks, and console email so you can evaluate without PostgreSQL, Redis or email credentials. It is for development only.

From this `fleetpro` directory:

```bash
python -m venv .venv
# macOS/Linux
source .venv/bin/activate
# Windows PowerShell: .venv\Scripts\Activate.ps1
python -m pip install -r requirements-local.txt
python manage.py migrate --settings=config.local_settings
python manage.py seed_data --settings=config.local_settings
python manage.py runserver --settings=config.local_settings
```

Open a second terminal:

```bash
cd frontend
npm ci
npm run dev
```

Open **http://localhost:3000**. The frontend proxies `/api` to Django on port 8000. After login, your account role selects the correct experience.

Demo accounts created by `seed_data`:

| Experience | Email | Password |
| --- | --- | --- |
| Customer | arjun.reddy@gmail.com | Customer@123! |
| Employee/admin | admin@fleetpro.in | Admin@123! |
| Super administrator | superadmin@fleetpro.in | FleetPro@2025! |

Use these only for local sample data. The database and credentials are generated locally; no populated database or real secrets are included. Existing SQL files are historical reference material: use the Django migrations as the schema authority, not both SQL and Django initialization.

## Try the customer flow

1. Sign in as the customer. Explore the collection or choose city, daily budget, seats, transmission and fuel.
2. Choose future pickup/return times, then **Find my match**. Category is a ranking preference; city, seats, fuel, transmission and daily budget filter eligibility.
3. Review **Why this is a match**. Scores express content preference fit, not an ML probability.
4. Open a vehicle, review rental/tax/deposit estimates, and request a reservation. The server rechecks dates and calculates the final amount; the request starts pending.
5. Open **My journeys** to inspect your bookings. Cancellation is supported for pending/confirmed requests; refunds are handled separately by staff.
6. Open **Concierge** to ask about rental requirements or your own bookings.

The sample fleet includes cars and motorcycles. Vehicle image uploads appear on the cards. When an image is missing, the app uses clearly described collection illustrations; they are not photographs of the listed vehicle.

## Enable both AI features

Create a backend `.env` alongside `manage.py`, or supply these environment variables:

```dotenv
FLEETPRO_AI_API_KEY=your_server_side_openai_api_key
FLEETPRO_AI_MODEL=gpt-4.1-mini
```

Restart Django. Keep keys out of frontend environment variables and source control. Configure an available Responses API model for your account if you use another model.

- **AI recommendations:** describe your journey and enable **Use AI to order matches**, then submit. The server first filters current inventory and dates, then optionally orders the top 12 eligible candidates using the description. Invented/duplicate candidate IDs are discarded. The displayed fit score retains its deterministic meaning.
- **AI chatbot:** enable the consent checkbox in Concierge. General rental questions use the server-side OpenAI Responses API. Own-booking queries are answered locally with a user-scoped query; policy questions use local guidance.
- Without a key or consent, content-based recommendations and local rental guidance remain usable. Provider timeouts or invalid responses fall back to local behavior and display the mode.
- External AI receives the submitted trip description or conversational input plus allowlisted catalog facts. Do not enter passwords, payment details or identity documents. `store=false` is sent to the provider; this is not a promise about all provider retention policies.

No live API key was supplied during validation, so external generation and ranking were tested with mocked provider responses and failures, not a billable live call.

## Validate

```bash
python -m pip install -r requirements-test.txt
python manage.py check --settings=config.local_settings
python -m pytest tests --ds=config.local_settings -q
cd frontend
npm run build
```

Validation details are recorded in **docs/VALIDATION.md**. The latest local suite passed 75 tests on Django 5.2.18; four PostgreSQL-only race/trigger tests are explicitly pending execution against PostgreSQL. Migrations and frontend production build also passed.

## Production deployment and verification

Read **docs/DEPLOYMENT_GUIDE.md** for the Docker/Caddy deployment, secure configuration and verification commands. Read **docs/PRODUCTION_READINESS.md** for implementation status and remaining live verification.

New functionality:

- Razorpay orders and signed webhooks; append-only rental/deposit capture/refund ledger.
- Local idempotency keys, bounded partial refunds and authenticated reconciliation.
- Branch-scoped employee APIs, analytics and AI catalog facts; super-admin-only unscoped administration.
- Private KYC/inspection/insurance storage, authenticated downloads, file validation and fail-closed malware scanning in production.
- Real PostgreSQL race tests and Docker/HTTPS/restore CI gates.
- Encrypted database/media backups, worker heartbeat, health checks, metrics and alert templates.
- Read-only live integration probes for provider credentials, SMTP, SMS account, private storage and AI.

Local development needs the operating system's libmagic library (`libmagic1` on Debian/Ubuntu; use the corresponding package on other systems). Private uploads accept PDF, PNG and JPEG up to 10 MiB. Production requires a working ClamAV service; scans cannot be bypassed by a user request.

The archive includes no real credentials and does not claim an unexecuted production deployment succeeded. Use the configuration template and run the documented live gates on your server before opening the application to real transactions.
