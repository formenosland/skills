# Agent Skills

Collection of skills to use for real engineering work.

Some are original, others are forked or adapted from other sources. Credits provided when due.

These skills are designed to be small, easy to adapt, and composable. They work with any model. Hack around with them, make them your own, and enjoy.

## Quickstart (30-second setup)

1. Install [skillsync](https://github.com/formenosland/skillsync):

```sh
brew install formenosland/tap/skillsync
```

```

2. Link your agents and add this catalog:

```sh
skillsync init
skillsync add formenosland/skills
```

3. That's it — you're ready to go.

## How To Use A Skill

Skills come in two flavors, split on one axis — who can invoke them.

- **User-invoked** skills are reachable only when you type them (e.g. `/terse`). Their job is to orchestrate.
- **Model-invoked** skills can be invoked by you _or_ reached for automatically by the agent when the task fits. They hold the reusable discipline.

A user-invoked skill may invoke model-invoked skills, but never another user-invoked one.

To trigger a skill manually, type its slash command (e.g. `/terse`). Otherwise, describe what you want and the agent will reach for the right skill on its own.

## Reference

### Engineering

| Skill | Invocation | Description | Credits |
| --- | --- | --- | --- |
| [code-review](./skills/engineering/code-review/SKILL.md) | `/code-review` | Orchestrated review of a PR, commit, branch, or diff against team standards. Model-invoked when you ask for a review; `/code-review` forces it. | - |
| [conventional-commit](./skills/engineering/conventional-commit/SKILL.md) | `/conventional-commit` | How the agent writes git commits: Conventional Commits 1.0.0 message shape, types, breaking changes, and a safe commit workflow. | [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/) |
| [lean](./skills/engineering/lean/SKILL.md) | `/lean` | Smallest working implementation: YAGNI, reuse, stdlib, native, existing deps, then the minimum. Model-invoked on implement/scaffold/new-dep work; `/lean` forces it. | Adapted from [ponytail](https://github.com/DietrichGebert/ponytail) (ladder only; no plugin, modes, or extra skills) |

### Product

| Skill | Invocation | Description | Credits |
| --- | --- | --- | --- |
| [five-whys](./skills/product/five-whys/SKILL.md) | `/five-whys` | Cheap causal gate: find the job or root cause behind a desire, imperative, or prescription ("I need", "I want", "add X", "use Y") before implementing. Model-invoked; `/five-whys` forces it. | Toyota-style 5 Whys; complementary to [grill-me](https://www.aihero.dev/skills-grill-me) by Matt Pocock |

### Productivity

| Skill | Invocation | Description | Credits |
| --- | --- | --- | --- |
| [terse](./skills/productivity/terse/SKILL.md) | `/terse` | Ultra-compressed communication mode. Strips filler, drops articles and pleasantries, and keeps technical terms exact — while leaving code, commits, and PR descriptions untouched. | Inspired by [caveman](https://github.com/JuliusBrussee/caveman) but compressed and without the bloat |

## Contributing

Found a bug or want to add a skill? Open an issue or a pull request. These skills are meant to be forked and adapted, so make them your own.
