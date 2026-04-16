---
name: connect-database
description: Use before writing any code that reads from or writes to a database, to discover the connection method and inspect the schema safely
---

# Connect Database

## Overview

Blind database work causes data loss and wasted sessions. Read-first, write-second.

**Core principle:** Understand the connection, then the schema, before touching any data.

## When to Use

Use this skill at the start of any task that involves a database, including:

- Writing a query or migration
- Adding a new table or column
- Debugging a database-related error
- Seeding or resetting data
- Connecting a new service to an existing database

## The Three Phases

### Phase 1: Discover the Connection

**Find how the project connects to its database before writing a single line.**

1. **Locate the connection string or config**
   - Check `.env`, `.env.example`, `config/`, `database.yml`, `settings.py`, `knexfile.js`, `prisma/schema.prisma`, or equivalent
   - Look for environment variables: `DATABASE_URL`, `DB_HOST`, `DB_NAME`, `DB_USER`
   - Check `docker-compose.yml` for service definitions

2. **Identify the database engine**
   - PostgreSQL, MySQL/MariaDB, SQLite, MongoDB, Redis, etc.
   - Each has different tools and syntax — confirm before proceeding

3. **Confirm connectivity**
   - For relational DBs: run a simple `SELECT 1` or equivalent to verify the connection works
   - For SQLite: verify the `.db` file path exists
   - For containerized DBs: confirm the container is running (`docker ps`)

4. **Check for a migration tool**
   - Alembic, Flyway, Liquibase, Rails migrations, Prisma migrate, Knex migrations, etc.
   - All schema changes MUST go through the project's migration tool, not raw DDL

**Stop here if** you cannot establish a connection. Diagnose the failure before continuing.

### Phase 2: Understand the Schema

**Read the schema before writing queries or migrations.**

1. **List existing tables / collections**
   ```sql
   -- PostgreSQL
   \dt
   -- MySQL
   SHOW TABLES;
   -- SQLite
   .tables
   ```

2. **Inspect relevant table structure**
   ```sql
   -- PostgreSQL / MySQL
   DESCRIBE table_name;
   -- or
   \d table_name
   -- SQLite
   .schema table_name
   ```

3. **Check existing indexes and constraints**
   - Unique constraints determine what can be upserted vs. inserted
   - Foreign keys determine deletion order
   - Indexes affect query plans — avoid accidental full-table scans

4. **Review existing migrations**
   - Read the most recent migrations to understand the intended evolution
   - Don't duplicate work already done in a pending migration

### Phase 3: Write and Verify Safely

1. **Use parameterised queries — always**
   - Never interpolate user input directly into SQL
   - Wrong: `f"SELECT * FROM users WHERE id = {user_id}"`
   - Right: `cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))`

2. **Wrap mutations in a transaction**
   - A single commit point makes rollback straightforward if something fails
   - Test the rollback path before declaring success

3. **Run queries in read mode first**
   - For any destructive operation, run the `SELECT` equivalent first to confirm the affected rows
   - Example: before `DELETE FROM orders WHERE status = 'pending'`, run `SELECT count(*) FROM orders WHERE status = 'pending'` and show your human partner the count

4. **Verify the result**
   - After an insert or update, query back to confirm the data is what you expected
   - Don't assume success from a zero-error return code alone

## Red Flags — Stop and Reassess

| Situation | Action |
|-----------|--------|
| Connection string contains production credentials | Stop. Ask your human partner before proceeding. |
| No migration tool found | Do not apply raw DDL. Find or create a migration file. |
| Schema differs from what you expected | Re-read the schema. Don't guess. |
| Destructive query affects more rows than expected | Rollback. Verify the WHERE clause. |
| You cannot reproduce the connection locally | Don't write against a shared or production DB. |

## Quick Reference

| Phase | Goal | Done When |
|-------|------|-----------|
| 1. Discover | Know engine, credentials, tooling | `SELECT 1` succeeds |
| 2. Schema | Know tables, columns, constraints | No surprises in structure |
| 3. Write | Safe, parameterised, verified | Query returns expected data |
