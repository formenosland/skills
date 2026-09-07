# Domain modeling

Schema mistakes outlast the code and are expensive to undo in production. Every column name is a contract future readers will trust.

## Distinct concepts → distinct tables

Don't conflate two concepts just because columns rhyme or they share a foreign key. Tests for "are these the same concept":

- **Lifecycle** — same creation, transitions, retention?
- **Actors** — same principal writes them?
- **Audit** — reconstructed together or separately?
- **Read patterns** — consumers want them together, or incidentally?

"Generic" tables (`user_data`, `entity_attributes`, `settings`, `metadata`) are suspect — they accumulate concepts and grow a `type` column gating the rest. Flag when a diff adds one.

## Events vs state

Legal / audit / forensic value → **append-only event log**. Never `UPDATE` it, never `NULL` its timestamps, never delete. Current state is _derived_.

Applies to: consent grants & withdrawals, ToS acceptances, approvals, signatures, payment authorizations, admin overrides, suspensions, permission changes, status transitions on regulated entities.

Anti-pattern: `consents(user_id, version, accepted_at)` with an UPDATE on re-consent. Destroys history. Correct shape:

```
consent_events(id, user_id, type, version, content_hash,
               action, occurred_at, actor_id, ip, user_agent)
```

Content hash binds the consent to the exact text agreed to — version labels drift; hashes don't.

## Status history

Every row with a `status` column almost always needs a `status_changes` log. The column is a denormalization for query convenience; the log is truth. **Never `NULL` a timestamp to mean "this didn't happen anymore"** — `cancelled_at = NULL` on reinstatement destroys the cancellation.

## Cardinality reflects reality

`UNIQUE` encodes claims about the world. If the world allows recurrence, the constraint is a bug pretending to be a guarantee. Ask: _can this legitimately happen twice?_

- Re-consent after version bump → `UNIQUE(user_id, type, version)`, not `UNIQUE(user_id, type)`.
- Multiple bank accounts per user → not `UNIQUE(user_id)` on `bank_links`.
- Multiple locations per org → not `UNIQUE(org_id)` on `addresses`.

Composite uniqueness over 3+ columns is a smell — usually the real identity wasn't modeled and the key is accidental.

## Column naming

- `*_url` → full resolvable URL, never a path or storage key.
- `*_key` → opaque object-store key, never a URL.
- `*_id` → ID. External IDs: `external_*_id` / `provider_*_id`. Don't mix internal and external in one column.
- `*_at` → timestamp with timezone. `*_on` → date.
- `*_count` → non-negative integer.
- Booleans: `is_*`, `has_*`, `can_*`. Not `status: bool`, not `flag`.

If a `document_url` column holds S3 keys, rename in the migration that fixes the data — don't let the lie propagate another release.

## Types and defaults

- Money: `NUMERIC(p,s)` or integer minor units. `FLOAT`/`DOUBLE` for money is a bug.
- Strings: `TEXT` or a `CHECK` that reflects a real rule. `VARCHAR(255)` is MySQL folklore.
- Enums: prefer `CHECK` constraint over native `ENUM` — enums resist evolution, and values end up scattered (UI, API) anyway. Pick one source and derive.
- `NOT NULL` and defaults are contracts. A default of `false`/`0` is a business rule in DDL — confirm it matches intent.

## Indexes

- Every FK needs an index (most engines don't auto-create).
- Unused indexes are write overhead — each new index needs a concrete query.
- Partial indexes (`WHERE status = 'active'`) for hot subsets of cold tables.
- `EXPLAIN` in the PR description is cheap evidence of intent.
- A review **may add missing indexes and optimize/improve indexing and related DB** when the diff introduces a filter/join/FK path that needs it. Auto-apply; do not ask; do not defer to a later performance pass.

## Migrations

- **Backward-compatible by default.** Drop/rename is unsafe while old code runs. Expand-and-contract: add new column → dual-write → migrate reads → drop old column in a later deploy.
- Naive `ALTER TABLE` on a large hot table locks for hours; require online-migration tooling (`pg_repack`, `gh-ost`).
- Every migration needs a rollback path. "Restore from backup" counts only if someone has confirmed the runbook.
- **Shipped migrations are append-only.** Edit one that's already on `main` / already run in production → blocker.
- **One migration per feature branch.** Reviewing a WIP branch: **do not add a new migration folder** — update the branch’s existing WIP `migration.sql` (and `schema.prisma`). If the branch has no migration yet, create one (`make db-migrate`). Reviewing `main`: a new migration is mandatory. Never edit a shipped migration.
- Data migrations are separate from schema migrations. Schema DDL should be idempotent; data changes are resumable jobs.

## Denormalization & triggers

Denormalization acceptable when (a) a measured read need justifies it, (b) **one writer** owns the derived column, (c) there's a reconciliation / rebuild plan.

**If the DB owns a derivation (trigger, generated column, view), application code must not also write that column.** Two writers = silent drift. Flag any app-level write to a trigger-derived column as a blocker.
