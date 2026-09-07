# Agent Skills

Collection of skills to use for real engineering work.

Some are original, others are forked or adapted from other sources. Credits provided when due.

These skills are designed to be small, easy to adapt, and composable. They work with any model. Hack around with them, make them your own, and enjoy.

## Quickstart (30-second setup)

1. Run the skills.sh installer:

```bash
npx skills@latest add formenosland/skills
```

2. Pick the skills you want, and which coding agents you want to install them on.

3. That's it — you're ready to go.

## How To Use A Skill

Skills come in two flavors, split on one axis — who can invoke them.

- **User-invoked** skills are reachable only when you type them (e.g. `/terse`). Their job is to orchestrate.
- **Model-invoked** skills can be invoked by you _or_ reached for automatically by the agent when the task fits. They hold the reusable discipline.

A user-invoked skill may invoke model-invoked skills, but never another user-invoked one.

To trigger a skill manually, type its slash command (e.g. `/terse`). Otherwise, describe what you want and the agent will reach for the right skill on its own.

## Reference

| Skill | Invocation | Description | Credits |
| --- | --- | --- | --- |
| [terse](./terse/SKILL.md) | `/terse` | Ultra-compressed communication mode. Strips filler, drops articles and pleasantries, and keeps technical terms exact — while leaving code, commits, and PR descriptions untouched. | Inspired by [caveman](https://github.com/JuliusBrussee/caveman) but compressed and without the bloat |
| [conventional-commit](./conventional-commit/SKILL.md) | `/conventional-commit` | How the agent writes git commits: Conventional Commits 1.0.0 message shape, types, breaking changes, and a safe commit workflow. | [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/) |

## Contributing

Found a bug or want to add a skill? Open an issue or a pull request. These skills are meant to be forked and adapted, so make them your own.
