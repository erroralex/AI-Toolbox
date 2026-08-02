---
name: sql-migrations
description: Use when creating or changing database schema in a project with versioned migrations (Flyway or Liquibase) — writing a new migration, fixing "Migration checksum mismatch" or "Detected resolved migration not applied to database" / "applied migration not resolved locally" errors, baselining an existing database, testing migrations, or planning zero-downtime schema changes (expand/contract).
metadata:
  author: custom (this starter kit)
  version: "1.0"
---

# SQL migrations (Flyway-first, Spring Boot)

Schema is code: every change is a new, immutable, versioned migration file that runs
identically everywhere. The database's actual state is whatever the migration history
says — never what someone clicked in a GUI.

## Golden rules

1. **Never edit an applied migration.** Applied anywhere — including a teammate's
   machine or CI. Fix mistakes with a *new* migration.
2. **One concern per migration**, named for it: `V7__add_index_orders_customer_id.sql`.
3. **Migrations don't depend on application code.** Plain SQL preferred; they must run
   on an empty database from V1 to head.
4. **`ddl-auto: validate`** (or `none`) in every environment. Hibernate never creates
   or updates schema outside throwaway local experiments.
5. Data backfills are migrations too, but keep them separate from DDL and make them
   re-runnable in batches if the table is large.

## Flyway with Spring Boot

Files live in `src/main/resources/db/migration/`:

```
V1__create_users.sql
V2__create_orders.sql
V3__add_index_orders_customer_id.sql
R__order_summary_view.sql      -- repeatable: reruns when its content changes
```

- Versioned `V<version>__<description>.sql` run once, in version order.
- Repeatable `R__` run after versioned ones whenever their checksum changes — use for
  views, functions, procedures (things that are safe to re-create).
- Spring Boot auto-runs Flyway on startup when it's on the classpath. Useful toggles
  in `application.yml`:

```yaml
spring:
  flyway:
    locations: classpath:db/migration
    # baseline-on-migrate: true   # only for adopting an EXISTING database, see below
  jpa:
    hibernate:
      ddl-auto: validate
```

## Error recovery

| Error | Cause | Fix |
|---|---|---|
| `Migration checksum mismatch for V5` | An applied file was edited | Revert the edit; put the change in a new migration. If the edit was cosmetic and intentional: `flyway repair` recalculates checksums — team-wide decision, not a reflex |
| `Detected applied migration not resolved locally: V6` | Your branch lacks a file the DB has | Pull/merge — someone else's migration ran first. Never delete their history row |
| `Detected resolved migration not applied to database: V4` (with V5 applied) | Out-of-order merge: your V4 landed after V5 ran | Renumber yours to head (V6), or enable `out-of-order: true` if the team explicitly allows it |
| `FlywayValidateException` on startup after a failed migration (Postgres is transactional DDL; MySQL is not) | Partially applied migration | Postgres: the failed migration rolled back — fix the SQL, restart. MySQL: manually undo the partial DDL, `flyway repair` to clear the failed row, rerun |

There is no down-migration in this workflow. **Roll forward:** write a new migration
that reverses the mistake. "Undo" scripts that are never tested are worse than nothing.

## Adopting an existing database (baseline)

For a database that predates Flyway: generate `V1__baseline.sql` from the current
schema (`pg_dump --schema-only` / `mysqldump --no-data`), then start the history with
`baseline-on-migrate: true` and `baseline-version: 1` so existing environments skip V1
and fresh ones build from it. Remove `baseline-on-migrate` once all environments have
a history table.

## Zero-downtime changes (expand/contract)

Old and new application versions run side by side during deploy — schema must support
both. Split breaking changes into phases, each its own migration + release:

| Change | Expand (release N) | Contract (release N+1, after old code is gone) |
|---|---|---|
| Rename column | Add new column; app writes both, reads new with fallback; backfill | Drop old column |
| Add NOT NULL column | Add nullable with default; backfill; app always writes it | Add NOT NULL constraint (Postgres: `NOT VALID` then `VALIDATE` to avoid long locks) |
| Drop column | App stops reading/writing it | Drop it |
| Change type | Add new column of new type, dual-write, backfill | Swap reads, drop old |

Large-table cautions: Postgres — adding a column with a *volatile* default or
validating constraints rewrites/locks; use `CONCURRENTLY` for index creation (outside
a transaction, so `flyway.executeInTransaction=false` for that script). MySQL — prefer
`ALGORITHM=INPLACE`; for huge tables use gh-ost/pt-online-schema-change. SQLite — no
real `ALTER`: create-new → copy → drop-old → rename, inside a transaction.

## Testing migrations

- Integration tests run the real engine via Testcontainers; Flyway migrates from V1
  on an empty container every run — that *is* the migration test.
- Don't substitute H2: it accepts SQL your production engine rejects and vice versa.
- A backfill migration on a big table gets a test with realistic row counts before it
  goes anywhere near production.

## Liquibase notes (if the project uses it instead)

Same golden rules. Changelog formats: prefer SQL (`--liquibase formatted sql`) or YAML;
each changeset gets immutable `id` + `author`; `logicalFilePath` keeps renames from
breaking history; `validCheckSum` is the escape hatch equivalent to `flyway repair` —
same caution applies.
