---
name: code-review
description: Review pull requests, commits, branches, diffs, or patches against team standards. Use when the user asks to review code, a PR, a commit, a branch, or a diff; asks what's wrong with an implementation; or wants code scrutinized before merging — even if they don't say "review". Prefer this over ad-hoc feedback whenever code changes are being evaluated.
---

# Code Review

This thread is the **orchestrator**. It talks to the user, merges verified leads, and writes the plan and the closing summary. It does **not** implement large applies or re-read the whole diff in-thread. Review, apply, and fallout run in workers. `discuss` / `explain` / questions stay here, short, without re-emitting the plan.

Assert about the _code_, never the author. When shown you're wrong, retract.

R1 is the completeness pass. After apply, **fallout** checks the patch — not a second first-principles review. Cap is R1 + F1 + F2 (F2 only if F1 found blockers). No F3 review.

## Voice (user-facing)

Write like Slack to a teammate. Lead with what a user, operator, or later engineer hits; then `file:line` (or N sites); then the fix in verbs.

Bad: `SELECT-then-UPDATE is a race at the persistence seam.`
Good: `Two retries can both read the old row and one overwrites the other. Use UPDATE … RETURNING.`

Do not use process words in user-facing text: residue, fingerprint, seam, landmines, fan-out, apply grammar, nested, R2/R3, F1/F2, orchestrator. No sandwich, no “What’s good.”

## Loop

**R1 — plan, no edits.** Pin the diff. Spawn review workers (or review inline if small). Verify citations. Cluster. Emit the plan. Wait for the user's selection. This is the only human checkpoint for what to fix.

**Apply.** Spawn apply workers. Do not implement a large apply in this thread.

**F1 — fallout** after selected R1 items land. Not a clone of R1.

**F2** only if F1 found blockers, after those blockers are applied. Blockers/regressions only.

A question **pauses** the current round; it does not consume a review slot.

If R1 is LGTM with nothing to apply, stop after the plan.

IDs (`B1`, `S1`, …) are for `apply B1`. They do not appear in the final summary or in questions to the user.

**Subsequent R1** (user asks again on the same branch): honor prior decisions, fingerprints, and the standards/smells already judged. Report only **new** failure modes. Do not dump the same drift bundle.

## Orchestrator vs workers

**This thread:** pin diff, resolve spec, spawn, targeted `Read` to verify cites (not the whole diff), merge/cluster, user conversation, spawn apply/fallout, plan + summary.

**Stay here:** `discuss` / `explain` / answers. No edits.

**Review workers (R1):** one per applicable domain when fan-out; else this thread reviews inline. Read-only. Leads only. **Do not paste this SKILL.md.** Paste: anti-recursion, six questions, that domain’s reference path, smells path, severity, prior decisions, diff command, “uncapped leads — parent clusters.”

**Spec worker:** only if a spec exists and the diff is large. Else spec inline or skip.

**Apply workers:** one worker, or one per independent `slice` in parallel. Brief = selected items (IDs, fix, files, test command, locked product rules), apply protocol, indexes/migrations rules. They edit, add a revert-failing test, run the narrow check. Return ~10 lines: applied, open, tests. Logs stay in the child.

**Fallout workers:** after apply. Brief = fallout scope below, not R1 coverage. Return findings in the lead shape. Parent verifies, then apply workers land auto-applicable items.

**Anti-recursion (every child):** do not invoke the code-review skill; do not spawn reviewers; do not talk to the user.

**Small-diff exception:** fewer than 3 routing refs **and** fewer than 8 files — orchestrator may review and apply inline. Spawn overhead can exceed the work.

**Gate** (full tests + typecheck + lint) runs in the last apply/fallout worker. This thread gets pass/fail. A failing check from our edit is a regression: the apply worker fixes it. Do not claim Ready if the gate fails.

## Methodology (R1)

Read in **domain order, not file order**: schema → queries → domain logic → API → UI → tests → plumbing.

Six questions on every non-trivial diff:

1. **What does this destroy?** — UPDATE, DELETE, NULL'd timestamp. Audit/legal/debug loss?
2. **Who else reads this?** — New column on a hot table: which unrelated consumers now carry it?
3. **What happens the second time?** — Re-consent, re-upload, retry, reconnect.
4. **Where are the two of the same?** — Two cache keys, two RBAC styles, two ID spaces.
5. **Could this have been one statement?** — SELECT-then-UPDATE is a race.
6. **If this fails halfway, what state am I in?** — Multi-step writes without a transaction.

Trace one full read and one full write: DB → repo → service → route → client → cache key → component → render.

### Document the why

A surprising invariant with no comment at the owner (query, route, middleware) is a should-fix. Unknowable from the diff → Question. The apply for a Question answer **is** that comment (plus any code they asked for).

Surprising: global vs unit scope, fail-open, two ID spaces, capability vs visibility. Examples: `claim_actions` keyed by `payer_claim_control_number` so a canonical remittance can switch; `GET /organization-units` listing every office for members on purpose.

### Spec

Resolve once, then stop:

1. This thread / the user's ask
2. PR body if reviewing a PR
3. Commit trailers (`#123`, `Closes`) if cheap
4. A path the user passed

None → headline `Spec: none`. Do not invent requirements. Do not spawn a spec worker.

If present: missing / partial, scope creep, implemented-wrong; quote the spec line. Fold into the same Blockers / Should-fix / Questions by severity. No separate Spec section.

### Coding standards and smells

Read [CODING_STANDARDS.md](../../../CODING_STANDARDS.md) on every R1. Convention-only drift is **one** should-fix with site bullets, not one ID per site. Drift that is also unsound, persistence, GET-with-side-effects, or security is a blocker.

Smell baseline: [references/smells.md](references/smells.md). Repo docs win. Judgement calls. Default nit unless a concrete failure.

## Routing

Read the matching reference **before** finalizing R1 (workers read their own; orchestrator does not re-read all of them after):

| Touches                                           | Read                                                                                                               |
| ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| Every diff                                        | `../../../CODING_STANDARDS.md`                                                                                     |
| Schema, migrations, constraints, indexes          | `references/domain-modeling.md`                                                                                    |
| SQL, transactions, locks, triggers, ORM queries   | `references/persistence.md`                                                                                        |
| Auth, secrets, PII, consent, uploads, crypto      | `references/security.md`                                                                                           |
| Routes, controllers, validation, RBAC, pagination | `references/api-design.md`                                                                                         |
| ApiError / JSON:API errors / client error display | `../api-errors/SKILL.md`                                                                                           |
| Components, state, query cache, forms, a11y       | `references/frontend.md`                                                                                           |
| Test files, mocks, fixtures, CI                   | `references/testing.md`                                                                                            |
| Commits, branch names, PR structure               | skip unless the user asked (`references/git-hygiene.md` only then)                                                 |
| R1 review briefs                                  | `references/smells.md`                                                                                             |

## R1 coverage

Count routing rows that apply (including `CODING_STANDARDS.md`). Skip git unless asked.

- **Inline:** fewer than 3 references **and** fewer than 8 files.
- **Fan-out:** 3+ references **or** 8+ files. Parallel read-only `generalPurpose` workers, one per applicable domain (schema, persistence, API, frontend, tests — skip unused). Skip git unless asked.

Review worker must: answer the six questions in its domain; trace one read and one write across its boundary; honor prior-loop decisions; flag missing why as should-fix or Question; return `None` only after that trace.

Lead shape:

```
[lead] <short-name>
 where:  path/to/file.ts:47
 defect: <what breaks>
 fix:    <proposed change>
 severity: blocker | should-fix | nit | question
```

### Merge (orchestrator)

Leads are **unverified**. Open the cited region, confirm the failure, drop unsupported / pre-existing / unreachable / restated prior decisions. Then **cluster**:

- Same root cause, many sites → **one ID**, list the sites
- Convention-only `CODING_STANDARDS.md` drift → **at most one** should-fix (`Standards drift (N sites)` + bullets)
- Smells → cluster; default nit

**Must drop:** nits on a large or subsequent pass; git hygiene unless asked; documented/user-accepted invariants restated; paths no current caller can hit unless this diff opened them; pre-existing in files the diff did not touch; unverified leads.

**May not omit** a verified new blocker or a verified distinct should-fix **root cause**.

Plan is **severity-first** (all blockers together). Never group the plan by domain. `slice:` on each item is for `apply api` etc.

Offer slice tokens when **3+ slices have should-fix, or more than ~8 should-fix IDs** (after clustering). Slices: `schema` (schema + persistence), `api` (API + security), `frontend`, `tests`.

## Plan format — exact

Nothing outside this structure. No preamble.

```
## Review plan — <PR title / branch / commit range>

<One-line judgment. Spec: <none | one-line source>.>

N blockers · M should-fix · K nits · Q questions

Scan:
- [B1] <one line>
- [Q1] <one line>
(blockers and questions only)

Reply with one of:
  `apply all`                every item
  `apply all blockers`       skip should-fix and nits
  `apply schema`             that slice (when offered)
  `apply api`                …
  `apply frontend`           …
  `apply tests`              …
  `apply B1, B3, S2`         by ID
  `skip B2, N1`              everything except these
  `discuss B2`               explore first
  `explain S1`               more detail

### Blockers

[B1] Two retries can wipe archived consents
  users.ts:412 — UPDATE nulls consent_archived_at before insert
  Fix: insert a new row; never NULL history
  slice: schema

### Should-fix

[S1] ...

### Nits

(omit this section on large or subsequent passes)

### Questions

[Q1] <what the diff cannot answer>
```

Omit empty sections. Omit Scan if there are no blockers and no questions.

## Severity

- **Blocker** — data loss, security hole, likely-production race, user-visible broken flow, GET with side effects (except last-activity on an authenticated session and a documented session-heal path already accepted), type unsoundness masking a real bug, retry/second-use break, schema that costs 10× more to undo after ship. A new GET write that is not those exceptions is still a blocker.
- **Should-fix** — real maintenance cost, missing tests for non-trivial branches, unclear errors, convention drift, misnamed fields, multiple migration folders on one feature branch (squash to one before merge), surprising invariant with no why.
- **Nit** — formatting, minor naming, redundant comments, style, isolated smells.

Unsure? _If this ships badly, is the fix a 5-line patch or a multi-week migration?_ Latter → blocker.

## Apply protocol (apply workers)

1. **Order:** schema → queries → domain logic → API → UI → tests.
2. **Per item:** edit; add or extend a test that **fails if this patch is reverted**; AuthZ tests need a **sibling row** (unit B / other office) that must stay hidden — deny-all `[]` is not enough; run that test + narrowest typecheck/lint, not the full suite per item. Cannot write the test → Open. Check failed because of our edit → fix the regression. Ask only on a product-rule conflict.
3. Return applied/open in English for the orchestrator summary. Fallout finds: prefix `Found after apply:` at summary time.

`discuss` / `explain`: this thread, no edits, no full plan re-emit.

Question answered: lock it, ensure the why is written in code, then apply when the user says (or include in the current apply). Chat is not documentation.

## Fallout (F1, F2)

Do **not** re-review the original 77-file surface. Do **not** only look at patched files.

**Look at:** patched lines and their tests; direct callers of changed functions/types; **consumer trace only if the apply changed a contract** (schema/column, route payload, cache key, AuthZ): who else reads that table / key / endpoint.

**May report:** regressions from our edits; six-questions failures on those paths; missing sibling-row tests for AuthZ we just added.

**Must drop:** new naming/Fowler/standards hunts; missing-why on untouched code; nits; “while I was here”; pre-existing unless this apply opened the path.

**F1, no blockers** (and no apply-introduced six-questions failure): **Ready.** Mechanical should-fix from F1 may still be applied; **do not spawn F2.** Should-fix-only F1 is not another review.

**F1 found blockers:** apply them, then **F2 is required**, tighter: only those blocker patches, tests, callers, consumer trace if *those* fixes changed a contract. F2 reports **blockers and regressions only**.

**F2 found blockers:** apply them, narrow tests + gate, **stop. No F3.** Last-apply risk is the revert-failing test + gate, not another reviewer.

Same-thread “I re-read my own patch” is not fallout — spawn workers. Briefs must carry prior-loop decisions and fingerprints.

Internal IDs (`F1-B1`, …). Orchestrator never shows them to the user.

```
### Findings
[F1-B1] <short-name>
 where:  path/to/file.ts:47
 defect: <what breaks>
 fix:    <proposed change>
 auto:   yes | ask
 fingerprint: <stable short id>
```

### Fingerprints

Same defect already applied → drop (stagnation if a fix was already attempted). New failure mode on patched lines → new fingerprint. Example: `inbox-stats-uuid-or-pccn` is not `inbox-stats-pccn-cross-unit`.

### Auto-apply (fallout)

Without asking: regressions from our edits; unambiguous blockers; missing/weak indexes for queries this diff introduces; F1 mechanical should-fix that is a six-questions miss on patched paths (not a new standards hunt).

Must ask: schema/API/product/security tradeoffs not in approved R1 items (except indexes); two fixes that **change product behavior** (if both preserve the documented invariant, pick the smaller); Questions.

Must not apply: git hygiene unless asked; restated prior decisions; nits; new Fowler/standards lists.

## Indexes and migrations

Missing/wrong indexes for this diff’s queries, FKs, or filters: auto-apply. Broader DDL (new tables, column semantics, constraint policy): **ask** unless already in approved R1 items.

- Feature branch with a WIP migration: **fold into that folder**. Do not add a second directory vs merge-base. Hand-merge custom SQL.
- Branch has no migration yet: `make db-migrate`.
- Reviewing `main` (or a shipped commit): **new** migration. Never edit a shipped migration.

## Stop conditions

Do not start another **review** when:

- F1 found no blockers (Ready path)
- F2 finished (no F3)
- A question is open
- Same fingerprint survived a fix attempt
- Cap: R1 + F1 + optional F2

The cap does not stop apply of remaining clear blockers. Then gate. Open = questions, user skips, stagnation.

**Ready** = F1 clean of blockers **or** F2 ran, its blockers were applied, gate green. Not “zero should-fix forever.” Questions still open → Not ready.

## Final summary — exact

Only this. English. No IDs. No round ledger.

```
## Review complete — <title>

<One-line judgment.>

Loop: <what happened in one clause>
Gate: tests + typecheck + lint pass|fail

### Applied
- <what was wrong → what we did. Fallout lines start with "Found after apply:">

### Open
- <unresolved and why: question / skip / stalled / gate>
  (omit if nothing)

### Skipped
- <only what the user skipped at R1>
  (omit if none)
```

Example:

```
## Review complete — consent re-enroll

Not ready: need a call on archived consents. Gate green.

Loop: R1 applied, fallout caught a race on the history fix
Gate: tests + typecheck + lint pass

### Applied
- Re-enroll no longer clears consent history
- GET /widgets no longer writes
- Found after apply: consent write raced after the history fix; now one statement

### Open
- Should archived consents stay readable after re-enroll? Fixing that would guess a product rule.
```

## When the diff isn't provided

Ask once:

> "Paste the diff, attach the patch, or give me the branch / PR URL. For a branch, I'll need the diff against its base or repo access."

Then wait. No generic checklist.
