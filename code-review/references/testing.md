# Testing

Tests that pretend to test something but don't are worse than no tests — they create the illusion of coverage.

## What each test type proves

- **Unit** — pure function / tightly-scoped class method, no I/O. Fast, deterministic, tests _logic_. If it starts a database, it's not a unit test.
- **Integration** — real seams between your own components, real I/O where the seams are. For a backend, that means a real database (Testcontainers-spun Postgres), real HTTP, real serialization. Mock only true external boundaries.
- **E2E** — whole system from outside, usually via UI. Slow. High-signal when passing. Keep targeted.

Names don't matter; what the test _proves_ matters.

## Integration tests must not mock their own integration

The most common anti-pattern: an "integration test" that mocks the database layer it's integrating with.

```ts
// ❌ mocks the repo the route uses — not an integration test
const mockRepo = { create: jest.fn().mockResolvedValue({ id: '1' }) };
app.use(usersRouter(mockRepo));
await request(app).post('/users').send({...});
expect(mockRepo.create).toHaveBeenCalled();
```

What this proves: the route called `repo.create`. It does not prove the SQL is correct, that unique constraints behave as expected, that transactions roll back, that the response shape matches what the DB returns, that foreign keys work.

**Mock only true external boundaries:**

- ✅ Third-party HTTP APIs (Stripe, Twilio) — can't control their responses, don't want their load.
- ✅ Payment processing, email, SMS — real-world side effects with cost.
- ✅ Time (`Date.now`, `setTimeout`).
- ❌ Your own database layer in an integration test.
- ❌ Your own business services in a test meant to prove they integrate.

`jest.mock('@fincura/db')` or any first-party query module in an API/route test is a **blocker**, not a nit. `CODING_STANDARDS.md` Testing is the author contract.

## Tests typing conventions

- **`any` in test files** — when narrowing mocks; elsewhere it's a blocker.
- **`@ts-ignore` in test files** — when ignoring a type error that doesn't affect the test; elsewhere it's a blocker.
- **`@ts-expect-error` in test files** — when expecting a type error that doesn't affect the test; elsewhere it's a blocker.

## Test structure mirrors code structure

If `users.service.test.ts` ends up asserting about users _and_ consents because the services are tangled, that's a signal the services are tangled. Difficulty in testing is telling you about the design.

Nested `describe`s group by method/scenario. A file with one `describe` and 50 flat `it` calls is missing structure — failures are harder to triage.

## What to test

- **Happy path** — at least one test per public function / endpoint.
- **Branches** — every `if` / `switch` / error path. Untested branch = dead code (delete) or untested risk (cover).
- **Boundaries** — empty, single-element, max-size, null/undefined if allowed.
- **Failure modes** — DB down, third-party timeout, malformed input. What does the caller see? Is state consistent?
- **The second time** — re-consent, retry, re-upload, reconnect. Most state bugs live here.
- **Concurrent case** — if called twice in parallel, does outcome make sense?

## What NOT to test

- Framework code (Express routing, the ORM's query builder).
- Third-party libraries (`dayjs` formats dates correctly).
- Private implementation details (couples tests to implementation; breaks on every refactor).
- Getters/setters in languages that have them (tests the compiler).

## Assertions

- One logical assertion per test, or one logical flow.
- Assert on the shape/values you care about. Equality on large structures breaks on unrelated changes.
- Don't assert error messages verbatim unless the message is part of the public contract.

## Fixtures & factories

- **Factories over fixtures** — `createUser({ role: 'admin' })` beats a `fixtures/user.json` every test fights with.
- **Each test owns its data.** No shared state between tests. Truncate / transaction-rollback / per-test schema.
- **Name fixtures for their role in the test** — `adminUser`, `suspendedAccount`, `unverifiedEmail`. Not `user1`, `user2`.

## Flakiness

A flaky test is worse than a failing one. Sources:

- **Time** — `Date.now`, intervals. Mock via fake timers. Never `await new Promise(r => setTimeout(r, 100))` as a wait — fix the wait with a real event.
- **Order dependence** — passes in isolation, fails after another test. Shared state.
- **Real network** — mock the boundary.
- **Unseeded randomness** — seed the RNG or use property-based testing deliberately.
- **Async race** — missing `await`; TypeScript's `no-floating-promises` catches most.

A test that's flaky on first introduction → don't merge.

## Coverage

- A floor, not a target. 80% with meaningless assertions is worse than 60% with sharp ones.
- Use coverage to find _uncovered_ code, not to congratulate yourself on a number.
- Don't write tests to hit a threshold. Tests encode invariants; threshold-chasing produces weak tests.

## Database tests

- **Run against the real engine** — Postgres for Postgres. SQLite-as-Postgres lies about array types, JSONB, window functions, `ON CONFLICT`, constraint behavior.
- **Testcontainers or equivalent** — real Postgres in Docker per test run.
- **Transaction-per-test** for isolation — begin at test start, rollback at end. Parallel-safe.
- **Migrations run in test setup** — test schema must match production.

## Authorization tests

Authorization bugs are a leading incident class. For every protected endpoint:

- Authenticated + authorized → success.
- Authenticated + unauthorized → 403, **no side effects**, no leaked data in error.
- Unauthenticated → 401.
- **Cross-tenant** — authenticated as tenant A, accessing tenant B's resource → must fail.
- **Sibling row** — unit B / other office / other row of the same type that must stay hidden. Deny-all `[]` or “unit A is present” does not prove isolation. A test that would still pass if the visibility filter were dropped is worse than no test.

The cross-tenant test is the one people forget. The sibling-row test is the one people fake.

## Contract tests

When two services communicate, contract tests pin expectations so consumer and producer can't silently drift. Consumer-driven (Pact), schema-driven (OpenAPI / protobuf), or shared-types (monorepo, published package).

If a diff changes a shared interface, what proves the consumer still works? Name the mechanism in the PR, or it's a gap.
