---
name: lean
description: >
  Smallest working code: YAGNI, reuse this repo, stdlib, native platform, already-installed deps, then one line, then the minimum. Use when implementing, adding a feature, fixing a bug, scaffolding, adding a dependency, wrapping a library, or when the change might overbuild. Also comments, JSDoc, or unsolicited markdown in the tree; "lean", "yagni", "don't over-engineer", or /lean. Do not use for talk-style-only requests, interviews about whether to build, or commit/PR/review orchestration.
---

# Lean

Ship the first solution that holds. Not sloppy: read the change, then write less.

`/lean` forces this skill even when the agent would skip it.

Implement the ask. Do not refuse it or turn it into a product interview. Do not ship a larger design and mention a smaller one as an aside.

## Ladder

Stop at the first rung that holds. Climb only after you have read the task and the code it touches, and traced the real flow.

1. Does this need to exist? Speculative = skip, one line.
2. Already in this repo? Reuse it.
3. Stdlib does it? Use it.
4. Native platform covers it? Use it (`<input type="date">` over a picker lib, CSS over JS, DB constraint over app code).
5. Already-installed dependency solves it? Use it. Do not add a new one for a few lines.
6. Can it be one line? One line.
7. Only then: the minimum that works.

Two rungs work → take the higher one. Smallest diff in the wrong place is a second bug, not lean. Next rung is a new package, new folder, or a second public API → you skipped a higher rung.

Bug fix = root cause. Grep every caller of the function you touch. One guard in the shared function beats a guard per caller.

## Rules

- No unrequested abstractions: no interface with one implementation, no factory for one product, no config for a value that never changes.
- No boilerplate "for later".
- Deletion over addition. Boring over clever. Fewest files.
- Two stdlib options, same size → the one that is correct on edge cases.
- User asks for the full version after a skip → build it, no re-arguing.
- Duplicate twice; extract on the third copy. A new helper for two similar blocks is not lean. Exception: one shared bug-fix path (above).
- Match the simplest existing pattern that already works for this seam, not the most layered neighboring file.
- Only files the task needs. No drive-by refactors, extra params, flags, retries, metrics, or "while here" comments.
- No new Service/Manager/Helper/Util/Adapter/interface unless this repo already uses that unit for this seam.

## Prose in the tree

A comment at the owner names a surprising *invariant* in **one line** (dual-read, fail-open, two ID spaces). Names, types, tests, and git carry what changed. An existing group label (`// skipped outcomes (noop)`) stays one line; it is not a license to paragraph every variant. Markdown, JSDoc, and module banners only when asked or this seam already uses that form (public API, OpenAPI, skill files). Requested docs, CONTEXT.md, and ADRs in scope stay.

## Do not cut

Input validation at trust boundaries, error handling that prevents data loss, security, accessibility basics, calibration real hardware needs, anything explicitly requested. A one-line invariant at the owner stays.

Never lean about understanding. Trace the flow first.

Non-trivial logic (branch, loop, parser, money/security path) leaves **one** runnable check: smallest thing that fails if the logic breaks. No test framework or fixture pile unless asked. Trivial one-liners need no test.

## Output

Code first. Then at most one line: `skipped: [X], add when [Y].`

Unrequested essays are complexity as prose. A report or walkthrough they asked for is not.

Examples:

- "Add a cache for these API responses." → `@lru_cache(maxsize=1000)` on the fetch. `skipped: custom cache class, add when lru_cache measurably falls short.`
- "Add a date picker." → `<input type="date">`. `skipped: picker lib, add when native input is not enough.`
- "These two blocks are similar, extract a helper." → leave both. `skipped: helper, add when a third copy appears.`
- Nearby `FooService` + interface, feature needs a call. → call the function this seam already uses. `skipped: sibling service, add when this seam already is that layer.`
- Dual-read enum member. → one line at the owner: reason `posted_partial` is dual-read; new writes use outcome `posted_partial` plus a real reason. `skipped: changelog comments and sibling-code lists.`
