# Security

Security bugs don't just break the feature — they break the company. Every boundary where untrusted input enters the system needs scrutiny.

## Credentials & secrets

- **Don't store plaintext credentials in a generic, broadly-read table.** If a third-party token or API key must be persisted, encrypt at the column level (KMS / `pgcrypto`). If the encrypted value and the decryption key live on the same host with no separation, that's obfuscation, not encryption.
- **Prefer not persisting at all.** Re-fetch short-lived tokens on demand.
- **Secrets ≠ config.** Secrets come from a secret manager at runtime, not a repo file, not the Docker image.
- **Rotation is part of the design.** If rotation requires a redeploy, the design is brittle.
- **Blast radius.** Different envs / services use different secrets.

## Sensitive columns & read paths

When a table carries sensitive data:

- **No `SELECT *`** on tables with sensitive columns, ever. Every caller names columns.
- **Distinct repositories for distinct views** (redacted list / full owner / full admin). One method returning "the row" and expecting callers to filter is how leaks happen.
- **Deserialization hooks cost.** A model that decrypts on deserialize pays on every instantiation, including timing-observable paths.

## Consent — legally binding

A consent record must survive restores, schema changes, and hostile subpoenas. Requirements:

- **Immutable / append-only.** Never UPDATE, never NULL, never delete.
- **Versioned with content hash** — label + `sha256:` of the exact text agreed to. Labels drift; hashes don't.
- **Server timestamp**, not client.
- **Actor identified** — user ID, plus org if acting on behalf.
- **Context captured** — IP, UA, request ID. Informational, not authoritative (client-controllable).
- **Binding to artifact** — the text at version V must be recoverable from the hash.

Any of these silently overwritable → consent is worthless.

## AuthN / AuthZ

- **Separate concerns.** AuthN = "who"; AuthZ = "what can you do."
- **Per-request authorization**, not security-by-obscurity. Unguessable IDs are _in addition to_ AuthZ checks, not instead of them.
- **Defense in depth.** Admin-only does not mean tenant-agnostic. Admin routes must still verify the target resource belongs to the admin's scope. Two independent checks, both required.
- **AuthZ through middleware, uniformly.** Inline `if (!isAdmin) return 403` at the top of a handler bypasses audit/logging/testability of the middleware layer. Route-level policy declared in one place.
- **No role claims from the client.** A JWT payload signed by the server is trust. An `X-Role` header is not.
- **Resource-level checks happen before any work.** Don't load + render + then reject.

## Sessions & tokens

- JWTs are stateless; no revocation before expiry without a server-side denylist (which makes them stateful anyway). Don't reach for JWTs reflexively — opaque session IDs backed by a store are often better.
- Short-lived access tokens + longer refresh tokens; refresh rotates on use; revocation server-side.
- No secrets in JWT payload. It's base64, not encryption.

## Uploads

- **Validate by content (magic bytes), not MIME or extension** — both are client-controllable.
- **Cap size at the edge** (proxy / load balancer), not in application code.
- **Server-side key generation.** Never let the client name the storage object.
- **Scan for malware** if other users / admins will download.

## Signed URLs

- **Never hand raw storage URLs or keys to the client.**
- **Sign at access time, not at upload time.** Lazy signing puts revocation and AuthZ at the moment of access.
- **Narrow expiry.** 15 minutes usually suffices; 7-day URLs leak.
- **Bind to requester** (IP, identity) when the provider supports it.

## Trust boundaries

- `req.ip` behind a proxy is a lie unless the framework trusts the proxy. Express: `app.set('trust proxy', <specific>)`. If not configured, every user appears to come from the proxy.
- Don't trust `X-Forwarded-*` on untrusted ingress — spoofable end-to-end if no proxy strips them.
- User agent and referrer are informational only.

## Injection

- **SQL** — parameterized queries always. Verify every raw query in the diff uses bound params.
- **Shell** — `execFile`/`spawn` with arg arrays; never `exec` with concatenated strings.
- **HTML** — `dangerouslySetInnerHTML` / `v-html` / `bypassSecurityTrust*` defeat framework escaping. Each instance needs justification + DOMPurify-style pre-sanitization.
- **Templates / LDAP / NoSQL / command** — same principle: structured queries with bound params.

## Logging

- **Never log secrets** (passwords, tokens, keys, PII, session cookies, consent content). The log aggregator becomes a juicier target than the DB.
- **Redact at the boundary**, not via "remember not to log this."
- **Structured logging** for searchability and redactability. `logger.info({ userId, action })` beats string interpolation.
- Log retention ≠ GDPR-exempt. If the user is deleted, are their PII log entries?

## Error messages

- Say "no" without saying "here's why." Internal errors logged with correlation ID; external response generic.
- **Don't distinguish "no such user" vs "wrong password" on login.** One hint per guess is reconnaissance.

## Rate limiting & abuse

- Public endpoints, especially login / signup / password reset / token exchange / anything triggering email or SMS.
- 429 with `Retry-After`; not 500, not silent drops.
- Per-account limits for authenticated endpoints (per-IP is weak on mobile NATs / VPNs). Combine where possible.

## Crypto

- **Don't invent.** Vetted library's high-level API (libsodium, AES-GCM, ChaCha20-Poly1305).
- **Authenticated encryption only.** AES-CBC alone → footgun.
- **Nonce/IV uniqueness** per key. Reuse leaks plaintext.
- **Passwords** use argon2id / scrypt / bcrypt. SHA-256 is wrong — the point is slowness.
- **Constant-time comparison** for secret comparison (`crypto.timingSafeEqual`, `hmac.compare_digest`). Not `===`.

## Dependencies

- Pin versions. `^1.2.3` admits supply-chain risk.
- `npm audit` / `pip-audit` / `govulncheck` as a CI gate.
- A new dependency in the diff is a review topic — what it does, who maintains it, recent activity, transitive surface.

## PII / data protection

- **Data minimization** is a legal principle. Don't collect unused fields; delete what you've stopped using.
- **Deletion means deletion** for GDPR/CCPA. Soft-delete is not deletion. Know the hard-delete strategy.
- **Residency.** EU user data staying in EU means DB replicas, backups, and logs too.
