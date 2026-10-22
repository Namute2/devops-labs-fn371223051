# 0002. Database for Lab 4

**Status / Статус:** Proposed

## Context / Контекст

Notes API keeps notes in memory by default, so they are lost on every restart. In Lab 4 the service needs persistent storage. The app already reads a `DATABASE_URL` variable, so the choice of database decides what the connection string looks like and what we have to run next to the app (for example in a container).

## Options / Варианти

**Option 1: PostgreSQL**

- Pros: a real database server that is close to what production services use; runs as a separate container; the `/ready` endpoint can check a real connection; works with the `DATABASE_URL` format the app already expects (`postgresql://notes:secret@db:5432/notes`).
- Cons: one more service to run, configure and keep healthy; needs a password and a volume for the data.

**Option 2: SQLite**

- Pros: no extra service; the data is a single file; very simple to set up.
- Cons: the file lives inside the app's container unless a volume is mounted; not suited to several app instances writing at once; does not teach running a database as a separate service.

**Option 3: Keep notes in memory**

- Pros: nothing to set up.
- Cons: notes are lost on restart, so there is nothing for Lab 4 to demonstrate.

## Decision / Решение

We use PostgreSQL, started as a separate container, and pass its address to the app through `DATABASE_URL`. The in-memory mode stays as the default when `DATABASE_URL` is not set.

## Consequences / Последствия

- We need a PostgreSQL container, a volume for its data, and a secret for the password (not committed to the repository).
- `/ready` returns 503 when the database cannot be reached, so deployments must wait for the database to start.
- Tests can keep using the in-memory mode, so they do not need a database.
- Backups and upgrades of the database become our responsibility.