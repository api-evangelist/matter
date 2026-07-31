---
name: Search Matter and read an item with its highlights
description: Run a full-text search across a Matter library, open the top result as markdown, and pull its annotations.
api: openapi/matter-openapi-original.yml
operations: [search, getItem, listAnnotations]
generated: '2026-07-20'
method: generated
---

# Search Matter and read an item with its highlights

Find content in the library and read it. Base URL `https://api.getmatter.com/public/v1`, `Authorization: Bearer mat_...`.

## Steps

1. **Search** — `GET /v1/search?query=<q>&type=items` (`search`). `query` needs ≥2 chars. Supported operators: `"exact phrase"`, `-exclude`, `by:author`, `site:domain`, `title:word`. Optionally add `status=queue,archive` to restrict to saved content. Results come back grouped: `results.items` is a cursor-paginated list ranked by relevance.
   - `search` has its own rate-limit tier (30/min).
   - A result item may have `status: null` when it is content that is not saved in the user's library.
2. **Open the article body** — `GET /v1/items/{id}?include=markdown` (`getItem`) returns the parsed article body in the `markdown` field. Markdown requests count against the `markdown` tier (20/min) as well as read.
3. **Pull highlights** — `GET /v1/items/{item_id}/annotations` (`listAnnotations`) returns the highlights (`text`) and notes (`note`) on the item, cursor-paginated.

## Rules

- Page every list via `has_more` + `cursor` (see `conventions/matter-conventions.yml`).
- Errors use the `{ "error": { "code", "message", "field" } }` envelope; `404 not_found` means the id is wrong or not owned by the account.
- Respect `Retry-After` on `429`.
