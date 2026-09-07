---
name: conventional-commit
description: >
  Write git commit messages that follow Conventional Commits 1.0.0. Use when the user asks to commit, create a commit, write a commit message, or mentions conventional commits, commitlint, or SemVer-style commit types (feat, fix, BREAKING CHANGE).
---

# Conventional Commit

Control how the agent writes git commits. Spec: [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/).

## When to commit

- Commit only when the user explicitly asks.
- Do not push unless the user asks.
- Do not amend unless the user asks and the amend is safe (HEAD is yours, not pushed).
- Never skip hooks (`--no-verify`) unless the user asks.
- Never update git config. Never use interactive flags (`-i`).
- Never bypass GPG signing (`--no-gpg-sign`, `commit.gpgsign=false`, or equivalent). See **GPG signing**.
- Do not commit secrets (`.env`, credentials, keys). Warn if asked to.

## Gather context (parallel)

Run together:

1. `git status` — staged, unstaged, untracked
2. `git diff` and `git diff --staged` — what actually changed
3. `git log -8 --oneline` — match this repo's type/scope style when one exists

Stage only files that belong in the commit. If nothing should be committed, stop. Do not create an empty commit.

If changes mix unrelated intents (feature + unrelated fix + docs), prefer multiple commits. One type per commit.

## Message format

```
<type>[optional scope][optional !]: <description>

[optional body]

[optional footer(s)]
```

Rules:

1. Type is a noun, then optional `(scope)`, optional `!`, then `: ` and description. Required.
2. Description is a short summary of **why/what**, imperative, no trailing period. Follows the colon and space immediately.
3. Body (if needed) starts one blank line after the description. Free-form; may be multiple paragraphs. Use body for motivation, not a file list.
4. Footers start one blank line after the body (or after the description if there is no body). Token + `: ` or ` #` + value. Tokens use `-` instead of spaces (`Reviewed-by`, `Refs`).
5. Casing of types is not case-sensitive; **use lowercase types**. `BREAKING CHANGE` in a footer **must** be uppercase. `BREAKING-CHANGE` is synonymous with `BREAKING CHANGE`.

## Types

| Type | When | SemVer |
| --- | --- | --- |
| `feat` | New user-facing capability | MINOR |
| `fix` | Bug fix | PATCH |
| `docs` | Documentation only | — |
| `style` | Formatting, no behavior change | — |
| `refactor` | Restructure without behavior change | — |
| `perf` | Performance improvement | — |
| `test` | Add or correct tests | — |
| `build` | Build system or dependencies | — |
| `ci` | CI configuration | — |
| `chore` | Maintenance that does not fit above | — |
| `revert` | Reverts a previous commit | — |

`feat` and `fix` are defined by the spec. Other types are allowed; prefer the table above unless the repo already uses a different set (then match the repo).

Scope is optional: a noun for the area of the change, in parentheses, e.g. `feat(parser):`.

## Breaking changes

A breaking API change is a MAJOR bump. Mark it in **either or both**:

- `!` immediately before `:`, e.g. `feat(api)!: drop support for Node 6`
- Footer: `BREAKING CHANGE: <description>` (uppercase)

If `!` is used, the description may stand as the breaking-change summary; the footer is still allowed.

A breaking change may appear on any type, not only `feat`.

## Reverts

```
revert: <description of what is being undone>

Refs: <sha>[, <sha>...]
```

## Commit command

Pass the message via HEREDOC (preserves formatting):

```bash
git commit -m "$(cat <<'EOF'
type(scope): description

Optional body.

Optional-Footer: value
EOF
)"
```

After commit, run `git status` to confirm success. If a hook rejects the commit, fix the issue and create a **new** commit. Do not amend a failed commit.

## GPG signing

If this repo or the user's git config enables signing (`commit.gpgsign`, `gpg.format`, a signing key, ssh/gpg program), keep signing on. Do not pass `--no-gpg-sign` or change git config to force an unsigned commit.

Signing often needs a **user action the agent cannot complete**: PIN, passphrase prompt, or touching a hardware key (YubiKey, etc.). If `git commit` is waiting on GPG/agent/pinentry:

1. Stop. Tell the user signing is waiting on them.
2. Say what to do (touch the key, unlock the agent, complete pinentry).
3. Wait for them to confirm, then retry the **same** signed commit. Do not proceed as if it succeeded.

If signing is configured and the commit **fails** (timeout, `gpg failed to sign`, no secret key, pinentry error, cancelled touch):

- **Hard fail.** Do not retry with `--no-gpg-sign`, `--no-verify`, or any other bypass.
- Report the error. Resolve it with the user (key available, agent running, they retry the signed commit).
- Only skip signing if the user **explicitly** asks after the failure.

## Examples

```
feat: allow provided config object to extend other configs

BREAKING CHANGE: `extends` key in config file is now used for extending other config files
```

```
feat(api)!: send an email to the customer when a product is shipped
```

```
fix: prevent racing of requests

Introduce a request id and a reference to latest request. Dismiss
incoming responses other than from latest request.

Refs: #123
```

```
docs: correct spelling of CHANGELOG
```

## Anti-patterns

- Subject that is not `type: description` (missing type or missing colon-space).
- `Added X` / `Adds X` instead of imperative `add X`.
- Dumping `git status` file lists into the subject.
- One commit that is both `feat` and `fix` when they can be split.
- Using a type not in the spec/table when a table type fits (`feet` instead of `feat`).
- Lowercase `breaking change:` footer token.
- Retrying a GPG/signing failure with `--no-gpg-sign` or unsigned config to "just land the commit".
