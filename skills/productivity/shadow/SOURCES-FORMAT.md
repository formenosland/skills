# sources.md

One entry per source. The cursor is the newest item id stored for that source. `last_run_at` is when this source was last fetched.

```markdown
# Sources

## <name>

- kind: <short label for this platform>
- locator: <handle, url, or other id on that platform>
- cursor: <newest stored id, or none>
- last_run_at: <ISO-8601, or none>
- status: ready | blocked
- blocked: <missing read tool, or omit when ready>
```

The heading name identifies this source inside the shadow. It can be the handle on that platform. It is not the shadow slug. The name is stable. The ledger uses it in the source column.

`kind` is a short label for the platform (`x`, `youtube`, `reddit`, `blog`). It is not a fixed list. Any place a read can return items is a source.

## Fetch

Read commands only. The command returns items. Page until the stored cursor, then stop. Pick the read the harness has for that kind.

Examples of the same pattern: `xurl` read or the raw user-posts endpoint for X; videos newer than `cursor` and their transcripts for YouTube; feed entries newer than `cursor` and their bodies for Substack. Another kind uses whatever read returns that platform's items.

Set `status: blocked` and name the missing read tool when that command is not available. Leave `cursor` unchanged. A blocked source is skipped on later refreshes until the tool is present.

Set `status: ready` after a fetch that returned, including a fetch that found nothing new. Update `last_run_at` on every fetch that returned. Update `cursor` only when a new row was stored, to that row's id.

Done when every source the user named has an entry, and every ready source has a `last_run_at`.
