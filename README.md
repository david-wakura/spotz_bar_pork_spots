A Django web application for running a bar and pork-spot business. It covers sales, stock, staff, property, expenses, and day-to-day operations in one place. Built for a small team where the Director needs to see everything and staff need to do their jobs without friction.

What this system does
Point of sale. Ring up sales at the bar and other outlets. Handles cash, M-Pesa (both STK push and manual), and card payments. Splits payment across methods if a customer pays partly in cash and partly by phone. Snapshots product names at the time of sale so historical receipts always read correctly even if the product is later renamed.

Stock control. Track consumables (beer, pork, ingredients) per operating point. Every movement — sale, receipt, transfer, wastage, production — writes an immutable ledger row. Balances are derived from the ledger, never edited directly. Low-stock alerts fire automatically when a balance crosses its minimum.

Internal transfers. Move stock between operating points (e.g. butchery to kitchen). A sender confirms what they're shipping; the recipient confirms what they received. Small variances are auto-accepted; larger ones require a reason and are recorded as wastage. Transfers auto-complete after 30 minutes if the recipient never responds, so stock never sits in limbo.

Point operations. Each operating point (BAR, BUTCHERY, CHOMA, KITCHEN) has its own page showing live state: open or closed today, staff deployed, sales at a glance, stock on hand, what's below minimum, and recent movements.

Business day. Each trading day opens and closes. Staff can open their own point when they arrive and close it when they leave. The global day auto-closes when the last point closes. Staff are auto-deployed to their home point when a day opens, and the Director can override for coverage.

Expenses. Record business expenses with categories (Utilities, Security, Personnel, Operations, Entertainment, Licenses, Other). Every expense starts as pending and the Director approves or rejects. Recurring templates (DSTV, Wi-Fi, rent) generate pending expenses automatically when they come due.

Property and assets. Track durable assets (furniture, electronics, equipment). Assets can be relocated between areas with a request → approve → receive workflow. Condition reports flag issues for the Director to resolve.

Reports. Daily closing (cash expected in drawer, sales by payment method, wastage, stock by point), daily sales by point, projections vs actuals, wastage list, and asset movements.

Sales projections. The system projects expected daily sales per point using a seasonal moving average — it looks at the last six same-weekdays of sales, trims outliers, and weights recent weeks higher. The Director sees projected vs actual, with variance percentages colour-coded. Any projection can be manually overridden for special occasions (a concert, a holiday, a power cut).

Who uses it
Director — sees everything, approves expenses and asset relocations, opens/closes business days, manages staff and operating points, reviews all reports.

Attendants (bar, butchery, choma, kitchen) — ring sales, request and receive stock transfers, record wastage, open and close their own point, submit expenses.

Access is scoped: staff see their own point's data; the Director sees all.

Technology
Django 5.2 — Python web framework

SQLite for development (production can use PostgreSQL or SQLite depending on hosting)

No JavaScript framework — plain HTML, CSS, and small inline scripts

Celery was removed; scheduled work runs via cron and Django management commands

Requirements
Python 3.13

Dependencies pinned in requirements.txt

Running locally
bash
# Clone the repository
git clone https://github.com/david-wakura/spotz_bar_pork_spots.git
cd spotz_bar_pork_spots

# Create and activate a virtual environment
python -m venv .venv
# Windows:
.venv\Scripts\activate
# macOS/Linux:
source .venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Run migrations
python manage.py migrate

# Seed the default expense categories
python manage.py seed_expense_categories

# Create a superuser (Director account)
python manage.py createsuperuser

# Start the dev server
python manage.py runserver
Visit http://127.0.0.1:8000 and log in with the superuser you created.

Management commands
Command	Purpose
python manage.py seed_expense_categories	Create the standard expense category tree (7 roots, 23 children)
python manage.py compute_daily_projections	Compute sales projections for the next 7 days
python manage.py compute_daily_projections --days-back 30	Backfill projections for the last 30 days
python manage.py generate_due_recurring_expenses	Create pending expenses from recurring templates that have come due
python manage.py send_low_stock_reminders	Send reminders for active low-stock alerts and expire unacknowledged transfers
Scheduled jobs (production)
Two commands should run on a schedule (cron or a task runner):

compute_daily_projections — daily, to keep projections current

send_low_stock_reminders — hourly or as needed, to sweep low-stock alerts and auto-complete expired transfer requests

Project structure
text
accounts/     Users, staff, operating points, roles
audit/        Immutable audit log
business/     Business days, holidays, staff deployment, suppliers
catalog/      Products, categories, units of measure
config/       Django settings and URL routing
expenses/     Expense categories, expenses, recurring templates
property/     Assets, relocations, condition reports
sales/        Point of sale, sales, stock ledger, M-Pesa, projections
stock/        Stock operations, transfers, recipes, production batches
templates/    Base template and shared UI
Data model highlights
Immutable ledger. Stock movements and completed sales cannot be edited or deleted — corrections are made by new movements, not by mutating history.

Audit trail. Every state-changing action records who did it, when, and what changed.

Point-scoped access. Staff see only what they need. The Director sees everything.

Soft deletes. Categories and products are deactivated, not deleted, so historical records stay intact.

Deployment
The system is deployed on PythonAnywhere. Local deployment instructions:

Push code to GitHub

Clone on the server

Set DEBUG = False and update ALLOWED_HOSTS in settings.py

Run migrate and collectstatic

Configure static file serving

Point the WSGI file at config.settings

Reload the web app

License
Private business software. All rights reserved.
