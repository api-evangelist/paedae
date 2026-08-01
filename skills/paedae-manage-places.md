---
name: paedae-manage-places
description: >-
  Manage Gimbal places (geofences and beacons) and the beacons within them via
  the Gimbal Manager REST API.
api: Gimbal REST API
base_url: https://manager.gimbal.com/api
auth: 'Authorization: Token token=<organization_server_api_key>'
operations:
  - listPlaces
  - createPlace
  - getPlace
  - updatePlace
  - listBeacons
  - activateBeacon
  - getBeacon
---

# Manage Gimbal Places and Beacons

Use this skill to inventory and manage location assets on the Gimbal proximity
platform. All requests and responses are `application/json` and authenticate
with the organization server API key header
`Authorization: Token token=<organization_server_api_key>`.

## Steps

1. **List existing places** — `GET /v2/places` (`listPlaces`). Supports
   filtering and pagination for large accounts.
2. **Inspect a place** — `GET /v2/places/{placeId}` (`getPlace`) to see its
   geofence and associated beacons.
3. **Create a place** — `POST /v2/places` (`createPlace`) with a geofence
   and/or beacons. On validation failure the API returns HTTP 422 with a Gimbal
   error code (6001 Invalid).
4. **List beacons** — `GET /beacons` (`listBeacons`), or fetch one by factory
   ID with `GET /beacons/{factoryId}` (`getBeacon`).
5. **Activate a beacon** — `POST /beacons` (`activateBeacon`). If the beacon is
   already registered elsewhere you will get error 6002 (belongs to another
   user) or 6003 (belongs to another organization).

## Error handling

- 401 Unauthorized — check the `Authorization: Token token=` header.
- 404 / code 6000 — the referenced place or beacon does not exist.
- 422 / code 6001 — payload failed validation.

See `errors/paedae-problem-types.yml` and `conventions/paedae-conventions.yml`.
