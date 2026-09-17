---
name: twelve-factor
description: >
  Think in The Twelve-Factor App whenever the work is a deployable app or SaaS: one codebase, declared deps, env/secrets config, backing services as attached resources, build/release/run, stateless processes, port bind, process concurrency, disposability, dev/prod parity, logs as stdout, one-off admin processes. Use when designing, implementing, reviewing, refactoring, or introspecting an app/SaaS/service — new app, feature, Dockerfile, compose, k8s, Procfile, config/secrets, bind port, backing service, process model, logs, deploy, parity, migrate/console, legacy — or when the user says "12factor", "twelve-factor", "12-factor", or /twelve-factor. Do not use for libraries, CLIs with no deploy story, static UI, commit-message orchestration, or talk-only.
---

# Twelve-Factor

How a *service* is built so it can be configured and run the same way in every environment. This is how to think about an app, not a scorecard. Use it on new work and on existing code. Do not add infra the work does not need.

Canonical source: [The Twelve-Factor App](https://12factor.net/). Fetch a factor page only when the one-liner below is not enough.

`/twelve-factor` forces all twelve principles even when the agent would skip it.

## When

The work is a deployable app or SaaS: design, implement, review, refactor, or read legacy. Surfaces include config/secrets, container/orchestrator/Procfile, listen port, attached DB/queue/cache, build vs run, process model, logs, deploy, env parity, one-off migrate/console.

Not: a library, an undeployed CLI, static UI, commit-message orchestration, talk-only.

## Relevant factors

A new service uses **all twelve**. A narrower ask uses every factor that surface actually needs — as many as that is.

## Principles

I **[Codebase](https://12factor.net/codebase)** — one repo per app. Environments are deploys of that repo, not forks.
II **[Dependencies](https://12factor.net/dependencies)** — declare and pin what the app uses. No implicit system packages at runtime. Do not add a package only to look explicit.
III **[Config](https://12factor.net/config)** — secrets and values that *vary by environment* live in the environment (or a secret store). Constants that never vary stay in code. Nothing secret in the repo or image.
IV **[Backing services](https://12factor.net/backing-services)** — DB, queue, cache, mail are attached resources, reached by config, not by a code fork per environment.
V **[Build, release, run](https://12factor.net/build-release-run)** — build is immutable. A release is that build plus config. The run stage does not mutate the build.
VI **[Processes](https://12factor.net/processes)** — share-nothing. Session and durable state live in a backing service, not in local disk or process memory.
VII **[Port binding](https://12factor.net/port-binding)** — the app is the server; it binds a port. Do not assume an injected Apache/nginx app server unless the platform already is that.
VIII **[Concurrency](https://12factor.net/concurrency)** — scale by process types, not by a bigger in-process god object.
IX **[Disposability](https://12factor.net/disposability)** — fast start, graceful stop, crash-safe. No work that is only safe if this process lives forever.
X **[Dev/prod parity](https://12factor.net/dev-prod-parity)** — same *kinds* of backing services locally and in prod. Name gaps; do not paper them with “it works on my machine.”
XI **[Logs](https://12factor.net/logs)** — unbuffered stdout/stderr event stream. The app does not rotate or ship files.
XII **[Admin processes](https://12factor.net/admin-processes)** — migrate/console/one-shot are one-off processes against a release, same codebase and config as the app. Do not invent a script tree the design does not already need.
