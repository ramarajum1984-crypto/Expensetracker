# Spec 01 — Database Setup

## 1. Overview

This step replaces the stub in `database/db.py` with a working SQLite implementation. It creates the database file, defines the `users` and `expenses` tables, and seeds development data.

It is the foundational step of Spendly: authentication (registration, login/logout), the profile page, and all expense tracking features (add, edit, delete, filter) read from and write to these tables. No later step can be built until this one is complete.

## 2. Depends on

- None. This is the first step and has no prerequisites.

## 3. Routes

- No new routes are added.
- All existing placeholder routes in `app.py` (`/`, `/register`, `/login`, `/logout`, `/profile`, `/expenses/add`, `/expenses/<id>/edit`, `/expenses/<id>/delete`) remain unchanged.

## 4. Database Schema

### Table A: `users`

| Column          | SQLite Type | Constraints                          |
|-----------------|-------------|--------------------------------------|
| `id`            | INTEGER     | PRIMARY KEY, AUTOINCREMENT           |
| `name`          | TEXT        | NOT NULL                             |
| `email`         | TEXT        | NOT NULL, UNIQUE                     |
| `password_hash` | TEXT        | NOT NULL                             |
| `created_at`    | TEXT        | NOT NULL, DEFAULT CURRENT_TIMESTAMP  |

### Table B: `expenses`

| Column        | SQLite Type | Constraints                                                        |
|---------------|-------------|--------------------------------------------------------------------|
| `id`          | INTEGER     | PRIMARY KEY, AUTOINCREMENT                                         |
| `user_id`     | INTEGER     | NOT NULL, FOREIGN KEY → `users(id)` ON DELETE CASCADE              |
| `amount`      | REAL        | NOT NULL                                                           |
| `category`    | TEXT        | NOT NULL (must be one of the fixed categories in section 10)       |
| `date`        | TEXT        | NOT NULL, format **YYYY-MM-DD**                                    |
| `description` | TEXT        | Nullable                                                           |
| `created_at`  | TEXT        | NOT NULL, DEFAULT CURRENT_TIMESTAMP                                |

## 5. Functions to Implement (`database/db.py`)

| Function    | Responsibility |
|-------------|----------------|
| `get_db()`  | Opens `spendly.db` in the project root, sets `row_factory = sqlite3.Row`, executes `PRAGMA foreign_keys = ON`, and returns the connection. |
| `init_db()` | Creates both tables using `CREATE TABLE IF NOT EXISTS`. Safe to call multiple times. |
| `seed_db()` | Checks whether the `users` table already has rows; if yes, returns early. Otherwise inserts one demo user (name: `Demo User`, email: `demo@spendly.com`, password: `demo123` hashed via werkzeug) and 8 sample expenses linked to that user, covering all 7 categories, with dates spread across the current month. |

## 6. Changes to `app.py`

- Import `get_db`, `init_db`, `seed_db` from `database.db`.
- Call `init_db()` then `seed_db()` inside `app.app_context()` at startup (module level, after the app is created).
- The database must be fully initialised before any route is served.

## 7. Files to Change

- `database/db.py`
- `app.py`

## 8. Files to Create

- None.

## 9. Dependencies

- No new pip packages.
- Uses `sqlite3` (standard library) and `werkzeug.security` (already in `requirements.txt`).

## 10. Categories (Fixed List)

- Food
- Transport
- Bills
- Health
- Entertainment
- Shopping
- Other

These are the only valid category values.

## 11. Rules for Implementation

- No ORM, no SQLAlchemy.
- Parameterized queries only — never use string formatting to build SQL.
- `PRAGMA foreign_keys = ON` on every connection.
- Store `amount` as REAL, not INTEGER.
- Hash passwords with `generate_password_hash` from `werkzeug.security`.
- `seed_db()` must be idempotent (safe to call multiple times).
- Dates must always be stored as YYYY-MM-DD.

## 12. Expected Behavior

- `get_db()` returns a live connection to `spendly.db` whose rows are accessible by column name and which enforces foreign keys.
- `init_db()` creates the database file (if missing) and both tables; repeated calls make no changes and raise no errors.
- `seed_db()` on an empty database inserts exactly 1 user and 8 expenses; on a non-empty database it inserts nothing.
- The database enforces: unique emails, non-null required columns, and that every expense references an existing user.

## 13. Error Handling Expectations

- Inserting a user with a duplicate email raises `sqlite3.IntegrityError` (UNIQUE constraint failed).
- Inserting an expense with a non-existent `user_id` raises `sqlite3.IntegrityError` (FOREIGN KEY constraint failed).
- Invalid queries are not silently swallowed — they raise clear `sqlite3` errors for debugging.
- Connections are closed after use, including when an error occurs.

## 14. Definition of Done

- [ ] Database file is created on app startup
- [ ] Both tables exist with correct schema and constraints
- [ ] Demo user exists with a hashed password
- [ ] 8 sample expenses exist across all 7 categories
- [ ] No duplicate seed data on repeated runs
- [ ] App starts without errors
- [ ] Foreign key enforcement works
- [ ] All queries use parameterized SQL
