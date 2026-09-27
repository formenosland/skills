# Index

The retrieval layer. One file per domain: `index/<domain>.md`. A domain is a subject the person publishes on, named in lowercase words joined by hyphens (`macro`, `crypto`). Reuse a domain name already in `index/` when the item belongs there.

The index is not the procedure. `domains/<domain>.md` is how they think in that subject. `index/<domain>.md` is how you find the posts.

One line per item. An item that sits in several domains gets one line in each of those files, with the same id.

```markdown
# <domain>

- <published_at> `<id>` <one sentence from this item alone>
```

The sentence says what the item is about. A short post uses its own text as that sentence. Search this file for the question. Open the archive file for a matching id. Leave the rest of the index on disk.

Done when every new item appears as one line in each domain it was given, and each line's id has an archive file.
