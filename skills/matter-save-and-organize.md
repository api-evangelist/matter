---
name: Save and organize an article in Matter
description: Save a URL to the Matter library, wait for extraction, tag it, and file it in the queue or archive.
api: openapi/matter-openapi-original.yml
operations: [createItem, getItem, addTagToItem, updateItem]
generated: '2026-07-20'
method: generated
---

# Save and organize an article in Matter

Use the Matter API (`https://api.getmatter.com/public/v1`) to capture a URL and file it. Every request needs `Authorization: Bearer mat_...` and an active Matter Pro subscription.

## Steps

1. **Save the URL** — `POST /v1/items` (`createItem`) with body `{"url": "<url>", "status": "queue"}`. You get an `item` back with an `id` (`itm_...`).
   - Saving is idempotent by URL: if the URL is already saved, the existing item returns with HTTP `200` instead of `201`.
   - `save` has its own rate-limit tier (10/min).
2. **Wait for extraction if needed** — check `processing_status`. If it is `processing`, poll `GET /v1/items/{id}` (`getItem`) every ~5s until it is `completed` (typically 20–60s; ~40% complete instantly). `failed` means the source was unreachable/unsupported.
3. **Tag it** — `POST /v1/items/{item_id}/tags` (`addTagToItem`) with `{"name": "essays"}`. Tag names are case-insensitive and the tag is created if it does not exist.
4. **File it** — `PATCH /v1/items/{id}` (`updateItem`) with `{"status": "archive"}` or `{"is_favorite": true}` as desired.

## Rules

- Handle errors from the `{ "error": { "code", "message", "field" } }` envelope (see `errors/matter-problem-types.yml`). A `422 unprocessable` on save usually means an invalid `url` (check `field`).
- On `429`, back off using the `Retry-After` header — never retry in a tight loop.
- `403 forbidden` / `pro_required` means the account lacks Matter Pro.
