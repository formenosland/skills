# API design

The API is a contract; once shipped it's expensive to change. HTTP verbs have meaning — honor it.

## HTTP semantics

- **GET is safe and idempotent.** No writes, no upserts, no "reconciliation" on read. A GET that writes breaks caching, retries, monitoring probes, link previews, browser prefetch. Move writes to POST on a dedicated endpoint, a background job, or a webhook. A `GET` with side effects in a diff is a **blocker**, except last-activity on an authenticated session and a documented session-heal path the product already accepted. A new write on GET that is not that path still is.
- **POST** — neither safe nor idempotent. Resource creation, side-effectful actions.
- **PUT** — idempotent full replacement.
- **PATCH** — partial update, usually non-idempotent.
- **DELETE** — idempotent. Deleting an already-deleted resource still ends "deleted"; 204 or 404, no error state.

## URLs name resources, not actions

No verbs in URLs:

- ❌ `POST /users/:id/cancel` → ✅ `DELETE /users/:id/subscription`, or `POST /users/:id/cancellation` (cancellation as a resource).
- ❌ `POST /consents/:id/revoke` → ✅ `POST /consent-revocations`.
- ❌ `POST /orders/:id/refund` → ✅ `POST /orders/:id/refunds`.

When an action doesn't map to CRUD, model it as a first-class noun (`/transfers`, `/refunds`, `/approvals`).

## One ID space per path slot

If `/users/:id` sometimes takes a UUID and sometimes a slug, the router guesses. Guessing fails.

- One ID shape per slot. Either internal IDs everywhere or slugs everywhere — not both.
- Support multiple lookups via distinct paths (`/orgs/id/:id` vs `/orgs/slug/:slug`) or a query parameter naming the lookup type.
- Third-party IDs get their own namespaced path (`/integrations/stripe/customers/:stripe_customer_id`).

## Request validation

- **Parse into a typed shape at the edge** (Zod / valibot / io-ts). Handler receives the parsed result; `req.body` is not read directly.
- **`as` on `req.body` in a handler → validation layer is wrong.** Not missing: wrong. Blocker.
- **Reject with structured error** — which field, what was expected. 400, not 500.
- **Validate query and path params too**, not just bodies.
- **Size limits** on strings, arrays, bodies. Unbounded input is a DoS vector.

## Response shapes

- **Consistent envelope** across the API — either objects direct, or `{ data, meta }`. Pick one.
- **Uniform error shape** — JSON:API `errors[]` with **required** machine-readable
  `code` (`domain.reason`), plus `title`, complete user-facing `detail`, and
  `status`. See `.agents/skills/api-errors/SKILL.md`. No stack traces.
- **Dates** are ISO 8601 with zone. Not Unix timestamps, not wall-clock without zone.
- **IDs are strings**, even if numeric today — JS `Number` loses precision above 2^53.
- **Pagination metadata explicit**: `{ items, nextCursor }` or `{ items, page, size, total }`.

## Authorization in the pipeline

- **Middleware-level, uniformly.** Declared at route definition, auditable from one place.
- **Defense in depth.** Admin-only routes still verify tenant scope on the target resource. Cross-tenant access from an admin context is a common bug.
- **Resource check before load.** Don't spend compute fetching, then reject.
- **Role names from one source.** Backend, frontend, config reference the same enum. `admin` vs `administrator` across layers is a drift bug waiting to fire.

## Idempotency

For POSTs that create resources or trigger side effects (payments, notifications, external API calls), support an `Idempotency-Key` header. First request executes and records; subsequent requests with same key return the recorded result. Without this, a client retry after a network blip double-charges, double-sends, double-creates.

## Rate limiting

- Every public endpoint has limits, especially unauthenticated ones.
- Return 429 with `Retry-After`.
- Per-key (authenticated) beats per-IP. Combine for unauthenticated.

## Pagination

- Sane default page size (20–50). Cap maximum (no 10k-per-page requests).
- Prefer cursor over offset for anything paged deep.
- Document the sort order; deterministic across concurrent writes (tiebreaker on ID).

## Webhooks (outgoing)

- Sign payloads (HMAC). Consumers verify authenticity.
- Include `event_id` for consumer dedup.
- Include timestamp to reject stale replays.
- Retry with exponential backoff → eventually dead-letter.
- At-least-once delivery is the realistic contract — make consumers idempotent on event ID.

## Webhooks (incoming, from third parties)

- **Verify signature first**, before any work. Reject unsigned / mis-signed with 401.
- **Return 2xx fast.** Heavy work → enqueue, process async. Providers retry on non-2xx including timeouts.
- **Idempotency on provider's event ID.** Same event delivered twice → processed once.
- Structured logging of everything about the request.

## Feature cohesion

- **One feature, one module.** When routes/services/DTOs for one capability scatter across folders with different conventions, bugs follow.
- **One source per value within a request.** If `tenantId` is in both `req.session` and `req.params`, pick one per handler; mixing both is inconsistency by construction.
