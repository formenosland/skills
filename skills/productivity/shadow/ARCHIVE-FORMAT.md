# Archive

One file per ledger row: `archive/<id>.md`. Sanitize the id for a filename. The ledger stores that path.

```markdown
# <title or first line>

- id: <platform id>
- source: <source name>
- url: <canonical url>
- published_at: <ISO-8601>
- domains: <domain>, <domain>
- summary: <a few sentences from this item alone>

<verbatim text>
```

The body is the post, the transcript, or the article. Write the text the read returned. The summary and the domains come from this item alone. A short post uses its own text as the summary. A later fetch that changes the hash rewrites this file and updates the ledger hash and the index lines for that id. The id stays.

Done when the file's id, source, and url match its ledger row.
