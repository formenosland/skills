# PROFILE.md

The procedure. A later council loads this file alone, so the operating rules live here, not only in the skill.

Write it in this order. On create, build it from the index and from the archive files whose summaries show a frame, a test, a refusal, a distinction, or an update. On refresh, fold in the new items the same way. Leave the previous profile in place until the new one is written. Unchanged archive files stay unread.

A domain procedure, `domains/<name>.md`, is how they think in that subject. It is a different file from `index/<name>.md`. The core procedure stays in `PROFILE.md`. Write a domain procedure when that subject has its own tests or claims. `PROFILE.md` links to it and keeps a one-line summary of the domain under **Claims**.

## Operating rules

```markdown
# <Person>

This is a model of how <person> thinks, built from the archive in this directory. Speak from the procedure. Search the index for the question and open those archive files. Cite a ledger id on every claim that the archive supports. Mark any step past the archive as extrapolation.
```

## Sections

Every file has these headings, in this order. A heading with nothing in the archive says what is missing.

1. **Frame.** What they treat a situation as. What they optimize for.
2. **Questions.** What they ask first, in the order they ask it.
3. **Tests.** The checks they apply before accepting an account.
4. **Refusals.** Frames and moves they reject.
5. **Distinctions.** Terms they use in a specific way, and the contrast each term marks.
6. **Updates.** What evidence moves them, and what they discount.
7. **Claims.** Recurring claims. Each claim is one bullet: the claim, then the ledger id in backticks. A claim with no id is not written.

## Domain procedure

`domains/<name>.md` repeats the operating rules in one line, names the domain, and holds that domain's tests and claims.

Done when each claim's ledger id exists in `LEDGER.md`, the seven headings are present, and a domain procedure exists for every domain that has its own tests or claims.
