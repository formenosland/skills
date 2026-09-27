# LEDGER.md

One row per stored item. Append. A row already present for the same source and id stays as it is.

```markdown
# Ledger

| id | source | url | published_at | fetched_at | domains | hash | archive |
| --- | --- | --- | --- | --- | --- | --- | --- |
| <platform id> | <source name> | <canonical url> | <ISO-8601> | <ISO-8601> | <domain> <domain> | <content hash> | archive/<file>.md |
```

- **id** is the platform id: post id, video id, or feed guid.
- **source** matches the name in `sources.md`.
- **domains** matches the domains on the archive file. Retrieval uses [INDEX-FORMAT.md](INDEX-FORMAT.md), so the summary stays out of this table.
- **hash** is a hash of the stored text, so a later fetch can show the body changed.
- **archive** is the path of that item's file, relative to the shadow directory.

The ledger is a catalog. Search `index/<domain>.md` to choose items. Read this table to check a cursor, a hash, or that a path exists.

A summary, a search answer, or a row missing an id, a time, a URL, or text is not a row.

## Gap

When a page does not come back, add a line under the table, not a row:

```markdown
Gap on <source name> at <ISO-8601>: <what failed>. Cursor left at <id or none>.
```

The cursor in `sources.md` moves to the newest id that has a row. Items older than a gap stay unstored until a later refresh fetches them.

Done when every archive file has one row and every row points at a file that exists.
