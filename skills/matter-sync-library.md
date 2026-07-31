---
name: Incrementally sync a Matter library
description: Efficiently mirror a Matter library locally using cursor pagination and updated_since delta sync, plus reading-session stats.
api: openapi/matter-openapi-original.yml
operations: [listItems, listReadingSessions]
generated: '2026-07-20'
method: generated
---

# Incrementally sync a Matter library

Keep a local mirror of the library up to date without re-fetching everything. Base URL `https://api.getmatter.com/public/v1`, `Authorization: Bearer mat_...`.

## First full sync

1. Record the current UTC timestamp as your checkpoint **before** you start.
2. `GET /v1/items?limit=100` (`listItems`). Append `results` to your store.
3. If `has_more` is `true`, repeat with `cursor=<next_cursor>` until `has_more` is `false`.

## Incremental sync (every run after)

1. Record a new checkpoint timestamp first.
2. `GET /v1/items?updated_since=<last_checkpoint>&order=updated&limit=100` (`listItems`) and page via `cursor` until `has_more` is `false`.
3. Upsert each returned item into your store by `id`. `updated_at` advances on any change (status, reading progress, favorite, tags, annotations, re-extraction), so this captures all deltas.
4. Save the new checkpoint only after the run completes successfully (so a mid-run failure is retried safely).

## Local ordering

Every item carries `library_position` (queue/archive manual order, higher = nearer top) and `inbox_position` (feed order, lower = nearer top). Sort locally with these instead of re-requesting in position order.

## Reading stats

`GET /v1/reading_sessions?since=<iso8601>` (`listReadingSessions`) returns `{date, seconds_read}` sessions (newest first, cursor-paginated). Aggregate client-side for streaks or daily totals.

## Rules

- Prefer `limit=100` and `updated_since` to stay within the read limit (120/min). Respect `Retry-After` on `429`.
- Treat responses as forward-compatible: ignore unknown fields (see `lifecycle/matter-lifecycle.yml`).
