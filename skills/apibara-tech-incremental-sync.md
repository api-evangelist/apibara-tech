---
name: apibara-incremental-vehicle-sync
description: >-
  Keep a local mirror of Copart & IAAI auction records current by pulling only
  what changed since the last run, using Apibara's updated_within_minutes change
  window and cursor pagination.
api: Apibara Vehicle Auction Data API
operations:
  - listAuctionVehicles
method: generated
source: >-
  Grounded in openapi/apibara-tech-openapi.json and the provider Arazzo workflow
  arazzo/apibara-tech-incremental-sync.arazzo.yaml
---

# Incremental vehicle-auction sync

Fetch a window of recently updated auction records and page through it.

## Steps

1. Call `listAuctionVehicles` — `GET /vehicles` — with header `X-API-Key: <key>`
   and query `updated_within_minutes=<N>` (1..525600). Use a value slightly
   larger than your scheduling interval so windows overlap.
2. Read `data[]` and upsert each vehicle by `vin`/`lot_number` (idempotent — one
   vehicle can have multiple auction records over time).
3. If `meta.next_cursor` is present, call `listAuctionVehicles` again with the
   same filters plus `cursor=<meta.next_cursor>`. Repeat until `next_cursor` is
   null.

## Rules

- Respect rate limits: on HTTP `429`, honor `Retry-After`; watch
  `X-Subscription-Remaining` for monthly quota.
- `per_page` maximum is plan-dependent (20/30/40); the enforced value is
  returned in `X-Per-Page-Limit` — do not assume one universal maximum.
- Null/missing source fields are expected and do not prove real-world absence.
- Source-data freshness (up to ~15 min) is separate from HTTP latency.
