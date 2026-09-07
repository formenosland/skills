---
name: five-whys
description: >
  Use when the problem is unstated or the ask is a desire, imperative, or prescription: "I need", "I want", "I wish", "we should", "we must", "make it", "can you", "please add", "just add", "just fix", "implement", "support", "enable", "allow users to", "use X for this", a feature, a fix, or a ready-made solution; also "five whys", "root cause", "XY problem", or /five-whys. Do not use when the user already named who hurts, what fails, and the constraint; do not use for tool/process instructions to the agent (commit, push, terse); do not use for exhaustive design interviews (grilling).
---

# Five Whys

Cheap causal gate on desires, imperatives, and prescriptions. One chain of *why*, then stop. Not a design-tree interview.

`/five-whys` forces this skill even when the agent would skip it.

## When

Apply when the **job or pain is missing** and the ask is a desire, an order, or a solution. The surface wording does not have to mention a widget.

**Desire (need/want hides a job)**

- "I need a dashboard"
- "I want users to export CSV"
- "I wish we had dark mode"
- "We should notify on save"
- "We must support SSO"

**Imperative (do the thing)**

- "Add a toast on save"
- "Make it faster"
- "Can you add a settings page"
- "Please implement retries"
- "Allow users to share a link"

**Prescription (solution instead of cause)**

- "The list is slow, add a cache"
- "Use a mutex"
- "Just fix it with a debounce"
- "New flag to hide the button"

The first *why* on a desire is *why do you need that* (job), not *how do we build it*.

Skip when the user already named **who**, **what breaks**, and **the constraint** — implement.

Skip when the imperative is **instructions to the agent**, not product work: commit, push, apply a skill, change tone, run tests they already scoped.

Skip exhaustive planning. If the idea needs every branch resolved, say so and stop. Do not become `/grill-me`.

## Budget

At most **five** causal steps with the user. Prefer **three**. Zero user questions if the repo already answers.

Each step must be a *why this* (cause or job), never *how should we build it*.

## Anticipate (before any question)

Do this **silently in tools**, then speak. Do not interview first.

1. Ground the ask in the project: README, similar features, the touched area, recent git, issues/TODOs, tests, obvious constraints (auth, data model, existing UI).
2. Form a **hypothesis** (keep it internal until you steer):
   - **Why they asked** — likely job, pain, or incident behind the wording.
   - **Whether it is needed** — already exists, half-exists, wrong layer, XY for a bug, or a real gap.
   - **Better direction** — smallest change that matches the likely cause, or "don't do this."
3. Steer in the first message: one-line restatement of the ask, then the hypothesis and what the repo shows. Recommend a direction. Ask *why* only for what you still cannot know.

If the repo shows the capability already, the ask is redundant, or a different cause is obvious — say that and wait. Do not start building the prescription.

If you cannot inspect a repo (no codebase, idea-only), skip file reads; still hypothesize from the conversation, then ask.

## Loop

1. Restate the ask as a proposed solution (one line) **and** the hypothesis from Anticipate.
2. Ask *why that solution* / *why that pain* only where the hypothesis is still open — one question, or a short batch whose answers do not depend on each other.
3. Repeat on the new cause until you hit a **job-to-be-done** or a **root cause** you can act on.
4. Exit. Do not walk sibling design branches.

Stop early when:

- The problem is named and the proposed solution matches it.
- The proposed solution does **not** match the cause — say that and wait.
- Further *why* is ungrillable (needs a prototype, a metric, a user). Name the gap; do not guess.

## After the chain

Output, then act:

```
Problem: <who / what fails / why it matters>
Cause: <root cause or job, one line>
Ask vs cause: match | mismatch
Next: implement | stop (mismatch or need evidence) | grill (scope is a product/architecture tree, not a cause)
```

- **match** → implement the smallest change that addresses the cause. The original ask is a hint, not a spec.
- **mismatch** → do not implement the prescription. Offer the cause-shaped alternative; wait.
- **grill** → only if remaining uncertainty is *which product/design branches*, not *why this exists*. One sentence pointing at grilling; do not run a grilling session inside this skill.

## Anti-patterns

- More than five *why*s, or a forty-question interview.
- Asking the user before reading the repo, or asking what `git log`, the failing test, or the stack already shows.
- Dumping a research report. Steer in a few lines; keep the rest in the hypothesis.
- "How should the UI look?" / "Redux or Context?" as a why-step.
- Implementing the original widget after the chain showed a different cause.
- Auto-firing on every feature or fix when the problem is already clear.
- Treating "I need you to commit / push / speak terse" as a product *why*.
