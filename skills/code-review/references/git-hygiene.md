# Git hygiene

This reference is **out of scope** for a normal code review. Read it only when the user asked to review commits, the PR, or history. Do not flag squash, branch names, commit messages, or PR body unless they did.

Good history is a gift to everyone who will later `git blame`, bisect, revert, or cherry-pick.

## Commit messages

```
<type>(<scope>): <subject, imperative, ≤50 chars>

<Body. Wrap at 72. Explain WHY. The diff shows WHAT.>

<Footer. Refs: ABC-123. Fixes: #42. BREAKING CHANGE: ...>
```

- **Imperative subject** — "Add caching", not "Added" / "Adds" / "Caching added".
- **No period** on subject; capitalize.
- **50/72** — soft 50 on subject, hard 72 on body wrap.
- **Why, not what.** "Fix bug" is not a message. "Fix token refresh race by holding lock across network call" is.
- **No "WIP" / "fixing stuff" / "more" / "review feedback"** in `main` history. Fine on a branch you'll squash; unacceptable in shared history.

### Conventional Commits (if used)

Type drives release tooling — a breaking change typed `fix:` ships as a patch and consumers break in prod. Using the wrong type is not cosmetic.

- `feat:` — MINOR bump
- `fix:` — PATCH bump
- `refactor:` / `perf:` / `docs:` / `style:` / `test:` / `build:` / `ci:` / `chore:` — no bump
- `BREAKING CHANGE:` in footer, or `!` after type — MAJOR bump

## Commit atomicity

One logical change per commit. Tests:

- **Reversible alone** — revert without also reverting unrelated work.
- **Buildable + testable at every commit.** `git bisect` is worthless otherwise.
- **Single subject line.** "and" in the subject → two commits mashed together.

Common violations:

- "Fix typo and refactor auth" → two commits.
- "Add migration, implement handler, update UI" → three. The migration ships on its own, provable alone.
- "Address review feedback" with 14 unrelated fixes → squash into the commits they belong to (interactive rebase).

### When to squash on merge

- **Squash** when branch history is noisy ("WIP", "fix typo", "respond to review") and cleaning via rebase costs more than the narrative is worth.
- **Preserve** when the branch is already a clean series telling a story.

Pick one policy per repo. What's never fine: merging 25 "WIP" commits into `main`.

## Branch naming

- `<type>/<description>` or `<ticket>/<description>` — stay consistent with the repo.
- Lowercase kebab-case.
- No personal names in long-lived branches.
- Short — the name appears in many places.

## Branch lifecycle

- Branch from `main` at current tip.
- Rebase on `main` regularly; long-lived branches without rebase = pain deferred.
- **Rebase, don't merge, _into_ feature branches.** Merge commits from `main` into a feature clutter the PR diff with unrelated changes.
- Delete after merge. Most hosts auto-delete — turn it on.

## Rebasing safely

- **Never rebase shared history.** Solo-owned branches only.
- `push --force-with-lease`, never `--force`. `--force-with-lease` refuses the push if the remote was updated since your last fetch.
- Interactive rebase to clean up before merging — on your own branch only.

## Don't commit these

- Generated files (`dist/`, `build/`, compiled artifacts).
- Dependency directories (`node_modules/`, `vendor/`, `venv/`).
- **Secrets** (`.env`, API keys, credentials, private keys). If committed, assume public — rotate immediately. `git filter-repo` removes from history, but pushed ≈ compromised.
- Local config (`.vscode/settings.json`, `.idea/`, `.DS_Store`, `Thumbs.db`).
- Large binaries. Use LFS or out-of-repo storage.
- Scratch work — commented-out experiments, `console.log` debugging.

Any of these in a diff → blocker.

## Merge strategy — one per repo

- **Merge commit** — preserves every commit, adds a merge. Full history, easy to revert the PR as a unit.
- **Rebase-and-merge** — linear history, no merge commits. Harder to revert "the PR" (it's N commits now); each commit is first-class.
- **Squash-and-merge** — one commit on `main`. Cleanest, loses granularity.

Enforce via host settings. Mixed strategies → messy history.

## Revert, don't rewrite

Commits in `main` pulled by anyone stay there. Bug fixes = new commits.

- `git revert <sha>` → safe, new commit that undoes changes.
- **Force-pushing `main` is a blocker-level mistake.** Breaks every clone, breaks CI, loses history.

## PR descriptions

Not optional. Required content:

- **What this changes** — one sentence.
- **Why** — link to ticket / issue / customer report.
- **How** — enough architecture to orient the reviewer before they look at code.
- **Risk** — what could break; blast radius if it does.
- **Testing** — what's tested, what isn't, how to test locally.
- **Rollout** — feature flag, dark launch, migration sequencing, rollback plan (if non-trivial).
- **Screenshots / recordings** — for UI changes.

"Fixes the thing" is not a description. Push back — the review can't be thorough without context, and asking is faster than reconstructing from the diff.

## Tags & releases

- Tags are immutable. Never move. Fix a release → cut a new one.
- Semver on externally-consumed artifacts.
- Changelog at every release, ideally auto-derived from conventional commits.

## CI

- Pre-commit: format, lint, type-check, fast tests. Slow = disabled.
- Pre-push: fuller suite if under ~30s.
- **CI is truth.** Don't trust local green.
- Required checks on `main`: format, lint, type-check, test, build. Nothing merges without them.
- Red CI = don't merge, even for "it's just flaky" — fix or quarantine the flake.
