# Smell baseline (judgement calls)

Fowler, *Refactoring* ch.3. Repo docs (`CODING_STANDARDS.md`, domain refs)
override. Tooling-enforced style is skipped. Default **nit** unless the smell
is a concrete failure (then should-fix or blocker). Never one finding per hunk.

| Smell | What | Typical fix |
| --- | --- | --- |
| Mysterious Name | Name hides what it does or holds | Rename; if no honest name, the design is murky |
| Duplicated Code | Same logic shape in more than one hunk | Extract the shared shape |
| Feature Envy | Reaches into another object's data more than its own | Move the method onto that data |
| Data Clumps | Same few fields travel together | Bundle into a type |
| Primitive Obsession | Primitive standing in for a domain concept | Small dedicated type |
| Repeated Switches | Same cascade on the same type in several places | Polymorphism or one shared map |
| Shotgun Surgery | One change edits many scattered files | Gather what changes together |
| Divergent Change | One module edited for unrelated reasons | Split by reason |
| Speculative Generality | Hooks the spec does not need | Delete / inline |
| Message Chains | Long `a.b().c().d()` the caller should not know | Hide the walk |
| Middle Man | Mostly delegates | Call the real target |
| Refused Bequest | Implementer ignores most of what it inherits | Composition, not inheritance |
