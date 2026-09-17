---
name: lean
description: >
  Smallest working code: YAGNI, reuse this repo, stdlib, native platform, already-installed deps, then one line, then the minimum. Use when implementing, adding a feature, fixing a bug, scaffolding, adding a dependency, wrapping a library, or when the change might overbuild. Also "lean", "yagni", "don't over-engineer", or /lean. Do not use for talk-style-only requests, interviews about whether to build, or commit/PR/review orchestration.
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

Two rungs work → take the higher one. Smallest diff in the wrong place is a second bug, not lean.

Bug fix = root cause. Grep every caller of the function you touch. One guard in the shared function beats a guard per caller.

## Rules

- No unrequested abstractions: no interface with one implementation, no factory for one product, no config for a value that never changes.
- No boilerplate "for later".
- Deletion over addition. Boring over clever. Fewest files.
- Two stdlib options, same size → the one that is correct on edge cases.
- User asks for the full version after a skip → build it, no re-arguing.

## Do not cut

Input validation at trust boundaries, error handling that prevents data loss, security, accessibility basics, calibration real hardware needs, anything explicitly requested.

Never lean about understanding. Trace the flow first.

Non-trivial logic (branch, loop, parser, money/security path) leaves **one** runnable check: smallest thing that fails if the logic breaks. No test framework or fixture pile unless asked. Trivial one-liners need no test.

## Output

Code first. Then at most one line: `skipped: [X], add when [Y].`

Unrequested essays are complexity as prose. A report or walkthrough they asked for is not.

Example: "Add a cache for these API responses."

- `@lru_cache(maxsize=1000)` on the fetch. `skipped: custom cache class, add when lru_cache measurably falls short.`
