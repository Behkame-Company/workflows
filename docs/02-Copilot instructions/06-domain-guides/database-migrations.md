# Domain Guide: Database & Migrations

> Instruction patterns for schema changes, Prisma, and migration safety.

---

## Sample: database.instructions.md

```yaml
---
applyTo: "prisma/**"
---
```

```markdown
# Database Instructions

## Schema Changes
- All schema changes go through `prisma/schema.prisma`
- After modifying schema, run: `pnpm db:migrate` to create a migration
- New columns on existing tables MUST be nullable OR have a default value
- Never delete columns — mark as deprecated with a comment first
- Never rename columns directly — create new, migrate data, then deprecate old

## Migrations
- Migration files in `prisma/migrations/` are immutable — NEVER edit them
- To fix a bad migration: create a new migration that corrects it
- Migration names should be descriptive: `add_user_email_verification`
- Always review the generated SQL before applying

## Naming Conventions
- Table names: PascalCase in Prisma, snake_case in DB (`@@map`)
- Column names: camelCase in Prisma, snake_case in DB (`@map`)
- Relation fields: named after the related model (`user`, `orders`)
- Enum values: SCREAMING_SNAKE_CASE

## Indexes
- Always add indexes on foreign key columns
- Add indexes on columns used in WHERE clauses frequently
- Composite indexes: most selective column first
- Unique constraints: use `@@unique` for business-rule uniqueness

## Seeding
- Seed data in `prisma/seed.ts`
- Seed must be idempotent (safe to run multiple times)
- Use `upsert` instead of `create` for seed data

## Queries
- Use Prisma Client for ALL database access
- Never write raw SQL (except for complex aggregations via `$queryRaw`)
- Always `select` only needed fields (don't fetch entire rows)
- Use `include` for relations, but limit depth to 2 levels

## Do Not
- Never modify existing migration files
- Never drop tables or columns without a deprecation period
- Never bypass Prisma for direct database access
- Never store passwords or tokens in plain text
- Never use `findFirst` when you mean `findUnique`
```

---

## Migration Safety Checklist

| Check | Why |
|-------|-----|
| New columns are nullable/defaulted | Prevents failures on existing rows |
| No column deletions | Preserves backward compatibility |
| No column renames | Prevents app errors during deployment |
| Indexes on foreign keys | Prevents slow JOIN queries |
| Migration is reversible | Enables rollback if needed |
| Seed data updated | New schema elements have test data |

---

## Migration Prompt File

```markdown
---
name: create-migration
description: Safe database schema change workflow
agent: agent
tools: [search, edit, run]
---
# Safe Database Migration

1. Describe the schema change needed
2. Modify `prisma/schema.prisma`
3. Run `pnpm db:migrate` — review generated SQL
4. Update seed data if new tables/columns added
5. Run `pnpm test` to verify nothing breaks
6. If adding a column to existing table: MUST be nullable or have default

Safety rules:
- Never modify existing migration files
- Never drop or rename existing columns
- Always add indexes for foreign keys
```
