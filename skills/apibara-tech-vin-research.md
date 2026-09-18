---
name: apibara-vin-auction-research
description: >-
  Look up a single vehicle by VIN or slug_vin and retrieve its retained Copart +
  IAAI auction history, then optionally its related vehicles and shipping.
api: Apibara Vehicle Auction Data API
operations:
  - getVehicleDetails
  - getVehicleHistory
  - getRelatedVehicles
  - getAuctionToPortShipping
method: generated
source: >-
  Grounded in openapi/apibara-tech-openapi.json and the provider Arazzo workflow
  arazzo/apibara-tech-vin-research.arazzo.yaml
---

# VIN auction research

Resolve a vehicle and pull its auction history.

## Steps

1. Call `getVehicleDetails` — `GET /vehicles/{slugVin}` — with header
   `X-API-Key: <key>`. Accepts a VIN or a slug_vin. On `404` the vehicle is not
   supported/indexed.
2. Call `getVehicleHistory` — `GET /vehicles/{slugVin}/history` — for the
   retained auction/sale history of that vehicle.
3. (Optional) `getRelatedVehicles` — `GET /vehicles/{slugVin}/related` — for
   comparable lots; `getAuctionToPortShipping` — `GET /shipping/auction-to-port`
   with `vin`/`lot_number` + `port` — for export shipping estimates.

## Rules

- Retained auction history is NOT complete ownership or accident history; do not
  present it as a full title/accident record.
- Preserve source attribution (Copart vs IAAI via `platform`).
- Errors use `{ ok:false, status, message, errors }` (not RFC 9457). A `4xx` is
  a client/data condition, not a service outage.
