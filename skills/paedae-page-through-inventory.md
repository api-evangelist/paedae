---
name: paedae-page-through-inventory
description: >-
  Page safely through a Gimbal organization's places and beacons, using the
  documented page/per_page parameters and the resource-specific total headers,
  without accidentally pulling the entire estate in one response.
api: Gimbal Manager REST API
base_url: https://manager.gimbal.com/api
operations:
  - listPlaces
  - listBeacons
  - getPlace
  - getBeacon
generated: '2026-08-13'
method: generated
source: >-
  https://docs.gimbal.com/rest.html (pagination and filtering sections) —
  operationIds verified against openapi/paedae-places-api-openapi.yml and
  openapi/paedae-beacons-api-openapi.yml
---

# Page through Gimbal places and beacons

Every request carries the organization server API key from Gimbal Manager
(Organizations), in the non-standard `Token token=` form:

```
Authorization: Token token=<organization_server_api_key>
Content-Type: application/json
```

## 1. Always send pagination parameters

Gimbal's pagination is **opt-in**. If you send neither `page` nor `per_page`,
`listPlaces` and `listBeacons` return the **entire collection** in one
response. Never call these operations bare.

```
GET /v2/places?page=1&per_page=50
GET /beacons?page=1&per_page=50
```

Defaults, per the reference: sending `per_page` alone implies `page=1`; sending
`page` alone implies `per_page=50`.

## 2. Read the totals from the response headers, not the body

The counts live in headers, and the total header name is **resource-specific**:

| Header | listPlaces | listBeacons |
|---|---|---|
| total records | `Total-Places` | `Total-Beacons` |
| total pages | `Total-Pages` | `Total-Pages` |
| current page | `Current-Page` | `Current-Page` |

`Total-Pages` and `Current-Page` are **not sent when the filter matches nothing**
— treat their absence as an empty result set, not as an error. `Total-Places` /
`Total-Beacons` is `0` in that case.

Loop while `Current-Page < Total-Pages`, incrementing `page`.

## 3. Narrow with filters before paging

- `name=<value>` — partial match on the record name.
- `attribute_key{n}` / `attribute_value{n}` — custom attribute filters. A key
  and value sharing the same trailing number must **both** match:
  `?attribute_key1=hello&attribute_value1=world` returns only records whose
  attribute `hello` equals `world`. A key match alone does not qualify.

## 4. Fetch detail per record

- `getPlace` — `GET /v2/places/{placeId}`
- `getBeacon` — `GET /beacons/{factoryId}` (beacons are keyed by factory ID;
  "beacon" and "transmitter" are the same thing in Gimbal's vocabulary)

## Failure handling

There are no rate limits published and no `429` in the documented status set —
so there is also no `Retry-After` to obey. Pace yourself conservatively.

Documented statuses and Gimbal error codes:

| HTTP | meaning | Gimbal code |
|---|---|---|
| 401 | Unauthorized — bad or missing `Token token=` header | — |
| 404 | Resource Not Found | 6000 |
| 422 | Unprocessable Entity | 6001 Invalid |
| 500 | Unexpected Error | 9000 |

Beacon-specific: `6002` (beacon belongs to another user) and `6003` (beacon
belongs to another organization) mean the record exists but is outside your
organization — do not retry, and do not treat it as a transient failure.

No idempotency key is available, so a retried write is a second write. Reads
here are safe to retry; writes in the sibling skills are not.
