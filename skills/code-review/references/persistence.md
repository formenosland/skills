# Persistence

Persistence bugs are mostly concurrency bugs. They don't show up in unit tests; they show up on the second concurrent request in production.

## Fetch-then-act is a race

`SELECT` then `UPDATE` on the same row: between them, anyone can delete, update, or sibling-create. Replace with `UPDATE … RETURNING` (or `INSERT`/`DELETE` analogs); zero rows → 404. One round trip, no race.

When you see a `SELECT` immediately followed by an UPDATE/DELETE keyed on the same row, flag it.

## Transactions — what they do and don't do

Wrap in a transaction when: writes cross tables, step N reads step N−1's write, or a mid-sequence failure leaves a nonsensical state.

**Transactions don't atomify external I/O.** A DB tx wrapping a write + an HTTP call is a lie — the HTTP call fires regardless of commit. Cross-system consistency = outbox pattern: write intent to a local outbox row in the tx; async worker delivers with retries and idempotency.

**Long transactions hold locks.** A tx spanning an HTTP call, a user prompt, or a file upload is too long. Scope tightly to the atomic writes.

Default isolation (`READ COMMITTED` in Postgres/MySQL) does not prevent lost updates on SELECT-then-UPDATE patterns. Use `SELECT … FOR UPDATE`, optimistic concurrency with a version column, or (preferred) restructure to one statement.

## Triggers vs application code — one owner

If a trigger, generated column, or view owns a derivation, application code must not also write the derived value. Two writers → drift → silent bugs. Any app write to a trigger-derived column is a blocker.

## SELECT * payload creep

Every `SELECT *` picks up columns added later. When a diff adds a sensitive column (encrypted token, PII, audit blob), audit every `SELECT *` path against that table — those paths now carry the column whether callers want it or not. Hot-path queries name their columns.

## N+1 (especially lazy-load)

`await` inside a loop over DB rows where the inner call touches a relation. Silent in test suites; brutal at scale. ORMs with lazy-loaded properties make this invisible in the application code — the loop looks clean, but each property access fires a query.

Fix with a join, `WHERE IN (...)`, or a batched dataloader.

## Pagination

- **Deterministic ordering or rows skip/duplicate across pages.** `ORDER BY created_at` alone is not enough when timestamps tie; add a tiebreaker (`ORDER BY created_at DESC, id DESC`).
- **Offset pagination** scans and discards for deep pages. `LIMIT 20 OFFSET 10000` is 10,020 rows read. Use keyset/cursor pagination for anything users can page deep into.
- **Cursor tokens are opaque** from the client's perspective. Sign or encode so callers can't tamper to scan outside their scope.

## External pagination loops

Consuming a paginated third-party API: need a **leash**.

- Max-pages cap (abort + alert if exceeded).
- Timeout / `AbortSignal`.
- Exponential backoff with jitter on failure.
- Resumable from last-processed cursor.

Trusting a remote `nextPageToken` to terminate is a production hang waiting for a bad response.

## Soft delete (breaks three things at once)

Every query now needs `WHERE deleted_at IS NULL` — miss it once, leak deleted rows. Enforce via repository layer or a view, not convention.

`UNIQUE(email)` now blocks re-signup after delete. Use `UNIQUE(email) WHERE deleted_at IS NULL` or move deleted rows to an archive table.

Foreign keys now reference "deleted" rows. Decide: cascade-soft-delete, block the delete, or accept the orphan — and document which.

A diff adding `deleted_at` without addressing all three is a blocker.

## Time

Store UTC with zone (`TIMESTAMPTZ`). Wall-clock without zone is a bug. Dates without times (birthday, due date) use `DATE`, not `TIMESTAMP` at midnight — midnight-in-UTC flips across the dateline.
