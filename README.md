# Support Ticket Tracker

An end-to-end ticket-tracking tool built with Python and SQLite: a 3-table schema, validated create/update/close logic, SQL reports, logging, and a pytest suite.

## Schema
```
customers (id, name, email UNIQUE, created_at)
   └── tickets (id, customer_id FK, title, description, priority, status, created_at, updated_at, closed_at)
          └── ticket_events (id, ticket_id FK, event_type, old_value, new_value, note, created_at)  -- audit trail
```
`CHECK` constraints enforce valid priorities/statuses; foreign keys are enabled.

## Validation rules
- Title required; priority in `low | medium | high | critical`; customer must exist
- Status flow: `open → in_progress → resolved` (and back to `in_progress`); no skipping steps
- Closing is a separate action that **requires a resolution note**; closed tickets are immutable
- Every change writes a row to `ticket_events`; multi-step writes run in a transaction

## Usage
```bash
python -m tracker add-customer "Asha" asha@example.com
python -m tracker create 1 "Login fails on Safari" -d "500 after submit" -p high
python -m tracker update 1 -s in_progress -n "Investigating"
python -m tracker close 1 "Fixed cookie bug in v1.2"
python -m tracker list -s open
python -m tracker history 1
python -m tracker report
python -m tracker -v create 1 "Verbose run logs to console"
```
Use `--db path.db` to choose the database file (default `tickets.db`).

## Reports (SQL)
Status summary · open tickets by priority · tickets per customer · average resolution time (hours) · per-ticket activity history.

## Tests
```bash
pip install -r requirements.txt
pytest -q
```
