---
name: shadow
description: Build or consult a shadow, a compiled procedure of how a person thinks, kept current from their sources.
disable-model-invocation: true
argument-hint: "Person to shadow, or a slug to refresh or consult"
---

# Shadow

The person lives in a shadow directory. This skill writes that directory and, on a later invocation, reads it back. A **source** is one place they publish. A **ledger** row is one real item from a source. The **index** finds items by domain. The **procedure** in `PROFILE.md` is how they think across the archive. A domain procedure is how they think in one subject. The archive grows without being loaded whole.

The name comes from the shadows in the Silo series: an understudy who has absorbed how someone does the job.

## Shadow directory

Resolve the directory in this order:

1. The path the user gave.
2. `./shadows/<slug>/` when the current directory is a project (a git repo, or `./shadows/` already exists).
3. `$HERMES_HOME/shadows/<slug>/` when `HERMES_HOME` is set and the current directory is not a project.
4. `./shadows/<slug>/` in the current directory.

The slug is the name the user gives this person, in lowercase, words joined by hyphens. A handle on X, YouTube, or anywhere else belongs on that source, not in the slug. People use different handles per platform. When the user has only given handles, ask what name the shadow goes by and use that. A directory that already exists for that slug is that shadow. Done when one directory is chosen and you have not created a second copy.

## Branch

- **Create** when that directory does not exist.
- **Refresh** when the user asks to update a shadow, or the prompt is `refresh shadow <slug>`.
- **Consult** when the user asks the shadow a question.

One branch per run.

## Create

Ask for the name the shadow goes by, and for every source the user can reach, until both are named. A source is any place a read can return their items. X, YouTube, and Substack are examples. Write the directory using [SOURCES-FORMAT.md](SOURCES-FORMAT.md), ingest every reachable source using [LEDGER-FORMAT.md](LEDGER-FORMAT.md), [ARCHIVE-FORMAT.md](ARCHIVE-FORMAT.md), and [INDEX-FORMAT.md](INDEX-FORMAT.md), then write `PROFILE.md` using [PROFILE-FORMAT.md](PROFILE-FORMAT.md).

Done when every reachable source has a cursor, every blocked source says which read tool is missing, every stored item has index lines, and `PROFILE.md` contains every procedure section plus claims that cite ledger ids.

## Refresh

Read `sources.md`. For each source, fetch items newer than its cursor. Append the ledger, the archive, and the index for those items. Fold those items into `PROFILE.md` and into the domain procedures they touch, using [PROFILE-FORMAT.md](PROFILE-FORMAT.md).

Done when every source has either a cursor moved to the newest stored id or a recorded gap, every new item has index lines, and every claim in `PROFILE.md` cites an id that exists in `LEDGER.md`.

On Hermes, when the user asks for a schedule, register a cron job with the `cronjob` tool. Attach this skill. Set the prompt to `refresh shadow <slug>`. The job stays silent when nothing new arrives. Cron runs only while the Hermes gateway is running. In a project, refresh runs when the user invokes it.

## Consult

Load `PROFILE.md`. Load `domains/<name>.md` when the question sits in that domain. Search `index/<name>.md` for lines that match the question, using [INDEX-FORMAT.md](INDEX-FORMAT.md). Open the archive files for those ids. Leave the rest of the ledger and the archive on disk.

Done when every claim in the answer carries a ledger id or is marked as extrapolation.

## Ingest

Create and refresh share this rule. Use a read that returns items from that source, paged until the stored id. Which read depends on the source. `xurl` read or the raw user-posts endpoint for X, a channel's new videos and their transcripts for YouTube, and feed entries and their bodies for Substack are examples of that pattern. `x_search` returns a summary, so it never supplies a row. Any other source the harness can read follows the same pattern.

A row is written when the read returned an id, a time, a URL, and the text. With the row, write the archive summary and the index lines for that item. Credentials stay in the harness. A missing read tool blocks that source. The cursor advances only over items stored. Record a gap for a page that did not come back.
